---
title: "Migrating AOS persistent metadata to Flatbuffers with Zero Logic Changes"
url: "https://www.nutanix.dev/2026/04/15/migrating-aos-persistent-metadata-to-flatbuffers-with-zero-logic-changes/"
date: "Wed, 15 Apr 2026 14:00:00 +0000"
author: "Chris Rasmussen"
feed_url: "https://www.nutanix.dev/feed/"
---
<p>If you work on a large-scale distributed storage system long enough, you may eventually hit a wall where your data serialization format becomes a big CPU bottleneck.</p>



<p>For some time in the Nutanix Core Data path, our metadata was persistently stored and communicated using Google’s Protocol Buffers (Protobuf). Protobuf is mature, strongly typed, and excellent for many use cases. But, as our clusters scaled, metadata became more complex and, as we pushed millions of IOPS, profiling revealed a glaring issue: the CPU overhead and memory allocations required to serialize and deserialize these complex Protobufs were artificially capping our throughput. Every cycle spent unpacking metadata was a cycle stolen from serving user I/O.</p>



<h2 class="wp-block-heading">The Problem</h2>



<p>To understand why Protobuf choked our CPU, you have to visualize what deserialization actually does. Protobuf is beautifully compressed on disk. But, to read it, the CPU must parse every tag, allocate C++ objects on the heap, copy strings into new memory locations, and align pointers. A dense 14KB payload on disk explodes into a 45KB web of fragmented C++ objects in memory.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69629" height="679" src="https://www.nutanix.dev/wp-content/uploads/2026/04/Amod-Jaltade-Flatbuffers-Fig-1_-Overview-of-Protocol-Buffers-internals-v2-1024x679.png" width="1024" /><figcaption class="wp-element-caption">Figure 1: Overview of Protocol Buffers internals</figcaption></figure>
</div>


<p>&nbsp;Flatbuffers take the exact opposite approach. The data is written to disk already padded, aligned, and mapped with offsets. When we load it into memory, there is no decoding. We simply cast a pointer to the start of the byte array and read the data directly. We traded disk density for zero-copy memory access.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69630" height="748" src="https://www.nutanix.dev/wp-content/uploads/2026/04/Amod-Jaltade-Flatbuffers-Fig-2_-Overview-of-Flatbuffers-internals-v2-1024x748.png" width="1024" /><figcaption class="wp-element-caption">Figure 2: Overview of Flatbuffers internals.</figcaption></figure>
</div>


<p>After careful study, we chose to move to <strong>Flatbuffers</strong>. By trading a larger payload footprint for massive CPU and memory efficiency, we could achieve a completely flat, near-zero deserialization time regardless of metadata complexity. For typical storage systems, where reads drastically outnumber writes &#8211; TPC-E, the most recent transactional TPC benchmark specification, has a ~9:1 R:W ratio. Therefore, eliminating this deserialization penalty in the read-path is absolutely critical.</p>



<h3 class="wp-block-heading">Constraints</h3>



<p>But migrating a live, hyper-scale storage system presents two massive, seemingly impossible constraints:</p>



<ol class="wp-block-list">
<li><strong>The Data Constraint:</strong> We had hundreds of gigabytes of existing metadata sitting persistently on disks in Protobuf format. A cluster-wide, blocking background job to rewrite all of this to Flatbuffers would destroy foreground performance and risk downtime.</li>



<li><strong>The Developer Constraint:</strong> Flatbuffers are fundamentally different to construct than Protobufs. Asking our engineers to rewrite thousands of lines of complex state-machine logic just to populate Flatbuffers wasn&#8217;t feasible.</li>
</ol>



<p>In this post, I&#8217;ll walk through how we engineered a transparent migration by building a system that was completely bilingual on disk &#8211; reading legacy Protobufs but writing new Flatbuffers &#8211; and how we modified the compiler itself to allow clients to adopt the new format simply by updating class names, with zero changes to their underlying code flow or business logic.</p>



<h2 class="wp-block-heading">The Solution: Bridging the API Gap at Compile Time</h2>



<p>To understand why a transparent wrapper was necessary, you have to look at how fundamentally different the construction paradigms are between Protobuf and Flatbuffers.</p>



<p>With Protobuf, the generated classes act like standard mutable objects. A developer can instantiate an object, pass it around different functions, and set fields in a completely arbitrary order. Flatbuffers, however, are built &#8220;bottom-up.&#8221; You must create child objects (like strings or nested tables) <em>before</em> you create the parent object that references them. Forcing our engineering teams to rewrite thousands of call sites to strictly follow a bottom-up construction order would have been a massive and arduous exercise.</p>



<p>We needed a drop-in replacement interface that mirrored the Protobuf API perfectly, but manually writing and maintaining these wrapper classes for thousands of changing schemas was out of the question.</p>



<h3 class="wp-block-heading">Hacking the Compiler: Auto-Generating the Bilingual Interface</h3>



<p>To overcome this hurdle, we modified the Flatbuffer compiler (<code class="">flatc</code>) itself to do the heavy lifting for us. We extended <code class="">flatc</code> so that during the build process, it ingests both the legacy <code class="">.proto</code> files and the new <code class="">.fbs</code> (Flatbuffer) schema files. Using both schemas, our modified compiler auto-generates a unified, bilingual interface class.</p>



<p>To the client code, this auto-generated interface exposes the exact same setter and getter methods they were already using (e.g., <code class="">set_timestamp()</code>, <code class="">mutable_child_node()</code>). The only code change required by the client teams was a simple search-and-replace to update the class namespace they were instantiating. The business logic and control flow remained 100% untouched.</p>



<p>Under the hood, this generated interface acts as a transparent translation layer. It handles the state accumulation required to bridge the gap between Protobuf&#8217;s random-order mutations and Flatbuffer&#8217;s strict bottom-up construction. When reading from the disk or the network, the interface layer dynamically routes access to either the legacy Protobuf object or the zero-copy Flatbuffer, completely transparent to the client.</p>



<p>Because this was all handled at compile time, developers simply continued writing schemas as usual. The build system automatically provided the bilingual classes that could read and write legacy Protobufs or Flatbuffers.</p>



<h3 class="wp-block-heading">The &#8220;Lazy&#8221; On-Disk Migration</h3>



<p>Solving the developer experience with an auto-generated compiler was only half the battle. The second, arguably more terrifying constraint was the sheer gravity of our existing data.</p>



<p>At Nutanix scale, our storage layer manages hundreds of gigabytes of persistent metadata. Historically, all of this was written to disk as Protocol Buffers. If we had attempted a traditional migration, forcing the cluster into a read-only state while a massive background job parsed and rewrote billions of metadata entries into Flatbuffers,we would have tanked foreground I/O performance and caused unacceptable downtime for our customers.</p>



<p>We needed a way to upgrade the airplane while it was flying. Enter our strategy for a &#8220;lazy,&#8221; mixed-state metadata migration.</p>



<h4 class="wp-block-heading">The Bilingual Disk</h4>



<p>Instead of forcing a mass conversion, we designed our storage layer to become completely bilingual at the storage layer. We updated our read/write paths so that the system could natively handle a disk where some metadata entities were legacy Protobufs, and others were newly minted Flatbuffers.</p>



<p>Here is how the lifecycle worked in practice:</p>



<ul class="wp-block-list">
<li>The Read Path: When our storage layer needed to read an entity&#8217;s metadata from disk, it inspected the payload header. If it detected a legacy Protobuf, our auto-generated interface dynamically parsed it into memory. If it detected a Flatbuffer, it achieved the zero-copy read directly. To the higher-level logic, both structures looked identical.</li>



<li>The Write Path: The actual migration happened invisibly during standard I/O operations. If an entity was read as a Protobuf, modified by the system, and then flushed back to disk, the interface always serialized the updated state as a new Flatbuffer.</li>
</ul>



<h4 class="wp-block-heading">Hot Data Migrates Itself</h4>



<p>The beauty of this architecture was its organic efficiency.</p>



<p>For hot data, where the metadata is actively being accessed and modified, naturally and rapidly converted itself to Flatbuffers simply by existing in the normal I/O path. For cold data (snapshots or archived metadata) sitting untouched remains safely stored as Protobufs with zero overhead. There was no need to waste precious CPU cycles or disk bandwidth rewriting cold data that wasn&#8217;t being actively queried.</p>



<p>With this, the cluster comfortably existed in a mixed state. We achieved a massive architectural shift with zero downtime, zero blocking background jobs, and a natural, phased transition that prioritized our most critical, latency-sensitive data first.</p>



<h4 class="wp-block-heading">Under the Hood: Storage and Routing Mechanics</h4>



<p>Abstracting the serialization layer of a live database sounds great in theory, but the actual implementation required solving complex memory and routing challenges. To make this project work, we had to build a system that dynamically routed operations to the correct backend engine.</p>



<h5 class="wp-block-heading">The Magic Header Routing</h5>



<p>How does a storage node instantly know if it&#8217;s reading a legacy Protobuf or a new Flatbuffer without taking a deserialization penalty?</p>



<p>We implemented a lightweight, unified <code class="">MessageHeader</code> struct that prepended every payload on disk. When our storage layer reads a payload, it simply peeks at the header&#8217;s magic bytes. If it sees <code class="">0xFEEEFEEE</code>, the system routes the payload to the Protobuf parser. If it sees <code class="">0xACEDFEED</code>, it bypasses parsing entirely and routes the payload to the zero-copy Flatbuffer engine. This routing happens in microseconds, completely transparent to the higher-level application logic.</p>



<h5 class="wp-block-heading">The Tri-State Machine</h5>



<p>Under the hood, our auto-generated interface classes didn&#8217;t just handle two states; they managed a strict tri-state machine represented by a unified <code class="">BaseMessage</code> class:</p>



<ol class="wp-block-list">
<li><strong><code class="">kProto</code>:</strong> The payload was read as a legacy Protobuf and instantiated into memory.</li>



<li><strong><code class="">kFlatbufferParsed</code>:</strong> The payload was read as a new Flatbuffer. The interface class simply held a raw pointer directly to the underlying bytes (zero-copy), acting as a read-only view.</li>



<li><strong><code class="">kFlatbuffer</code> (The Staging State):</strong> The state used when a developer is actively mutating or constructing a new message from scratch.</li>
</ol>



<h2 class="wp-block-heading">The Developer Experience: Defeating the &#8220;Bottom-Up&#8221; Constraint</h2>



<p>Flatbuffers are incredibly fast, but they demand a strict &#8220;bottom-up&#8221; construction. You cannot build a parent object until all of its children (strings, vectors, nested messages) have been created and their offsets collected.</p>



<p>In a system like Nutanix, an I/O operation is a complex, multi-stage state machine. We rarely have all the metadata available at the start of an operation. Instead, we populate pieces of the metadata incrementally as they are calculated or deduced across different stages of the pipeline.</p>



<p>Let&#8217;s look at a sanitized example from our internal test suite to see why native Flatbuffers would have broken this paradigm.</p>



<p>To set the stage, here is a simplified look at the data structures we are building. The schema involves a root message (<code class="">Message1</code>) containing scalar fields, vectors of strings, and nested child messages (<code class="">Message2</code>).</p>



<p>Here is how the legacy <strong>Protobuf</strong> definition looks:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-cpp">message Message2Proto {
  repeated int32 rep_int = 1;
  optional int32 opt_int2 = 3;
  optional col.Color color = 5 [default = Red];
}

message Message_1Proto {
  optional int32 opt_int = 1;
  repeated string rep_string = 2;
  repeated int32 rep_int = 3;
  optional Message2Proto opt_msg = 5;
  repeated Message2Proto rep_msg = 6;
}</code></pre>



<p>And here is the corresponding <strong>Flatbuffer</strong> definition we needed to migrate to:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-cpp">table Message2 {
  rep_int : [int] (id: 0);
  opt_int2 : int (id: 2);
  color : _nutanix._test._col.Color = Red (id: 4);
}

table Message1 {
  opt_int : int (id: 0);
  rep_string : [string] (id: 1);
  rep_int : [int] (id: 2);
  opt_msg : Message2 (id: 4);
  rep_msg : [Message2] (id: 5);
}
root_type Message1;</code></pre>



<h3 class="wp-block-heading">The Protobuf Way&nbsp;</h3>



<p>With <strong>Protobuf</strong>, the construction is highly intuitive and perfectly suited for a state machine. You can pass a mutable object around and set fields or append to lists as the data becomes available, in any random order:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-cpp">// Standard Protobuf Construction: Piecemeal &amp; Intuitive
Message_1Proto m1_p;
// We can set scalar fields natively
m1_p.set_opt_int(48);

// We can append to repeated fields randomly
m1_p.add_rep_int(4);
m1_p.add_rep_string(&quot;Hello&quot;);

// We can instantiate nested messages and set their fields piecemeal
auto m2 = m1_p.mutable_opt_msg();
m2-&gt;set_opt_int2(444);

// Now set a field in the parent message.
m1_p.add_rep_int(8);
// And another one in the child.
m2-&gt;set_color(col::Color::Green);</code></pre>



<h3 class="wp-block-heading">The Native Flatbuffer Way&nbsp;</h3>



<p>If we had forced our feature teams to use the native <strong>Flatbuffer C++ API</strong>, that exact same logical flow would have been destroyed.</p>



<p>Because of the bottom-up requirement, developers cannot incrementally populate a parent message. They would be forced to manually manage vectors of offsets, temporarily cache calculated strings, and ensure every nested object (like <code class="">Message2</code>) is completely finalized before they even attempt to construct the parent (<code class="">Message1</code>):</p>



<pre class="wp-block-prismatic-blocks"><code class="language-cpp">// Native Flatbuffer Construction: Bottom-Up &amp; Rigid
flatbuffers::FlatBufferBuilder builder;
builder.ForceDefaults(true);

// 1. Child elements MUST be created and their offsets stored manually
std::vector&lt;flatbuffers::Offset&lt;flatbuffers::String&gt;&gt; str_offsets;
str_offsets.push_back(builder.CreateString(&quot;Hello&quot;));
std::vector&lt;int32_t&gt; rep_int = {4, 8};

// 2. Build the nested message using direct offsets
auto m2o = CreateMessage2Direct(builder, nullptr, nullptr, 444, nullptr, 
                                _nutanix::_test::_col::Color::Green);

// 3. Finally, you can build the parent message using the manual offsets
auto m1o = CreateMessage1Direct(builder, 48, &amp;str_offsets, &amp;rep_int, nullptr, m2o);

FinishMessage1Buffer(builder, m1o);</code></pre>



<p>Forcing this paradigm onto a complex, piecemeal state machine would require passing dozens of <code class="">flatbuffers::Offset</code> objects between functions, resulting in a massive, unreadable refactor across the entire storage layer codebase.</p>



<h3 class="wp-block-heading">The Solution (The Drop-In Replacement)</h3>



<p>We aimed to completely shield our engineers from this complexity. Using our auto-generated interface, developers wrote code that looked and behaved almost identically to the legacy Protobuf implementation.</p>



<p>Under the hood, our <code class="">BaseMessage</code> interface buffered the piecemeal state additions into memory efficient data-structures. Any elements which could be directly written to the underlying flatbuffer were pass-through. For complex data types (strings, vectors, nested messages), data was buffered so that the developer was free to mutate the message in whatever chaotic, random order their business logic dictated.</p>



<p>Here is how the exact same message is constructed using the interface class:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-cpp">// We instantiate the auto-generated bilingual interface class with type 
// kFlatbuffer to indicate that the underlying format is a flatbuffer.
TestMessage1 m1(arena, nutanix::MessageType::kFlatbuffer);

// The API looks and acts exactly like Protobuf!
m1.set_opt_int(48);
m1.add_rep_int(4);
m1.add_rep_string(&quot;Hello&quot;);

// Nested messages are handled transparently
auto m2 = m1.mutable_opt_msg();
m2-&gt;set_opt_int2(444);

m1.add_rep_int(8);
m2-&gt;set_color(col::Color::Green);

// The actual strict, bottom-up Flatbuffer packing happens here.
auto buffer = m1.Serialize();</code></pre>



<p>Only when the state machine completed its run and the system called Serialize() did we traverse the staging area and execute the complex Flatbuffer packing logic.</p>



<p>Zero logic changes. Zero manual offset management. 100% of the Flatbuffer performance.</p>



<p>To make this illusion as performant as possible, the wrapper uses a hybrid construction model. Fixed-size scalars and plain old data (POD) types are passed efficiently, but dynamic types like vectors and strings, which Flatbuffers require to be built out-of-order, are intercepted. When a developer mutates a string, it gets safely parked in an in-memory staging area. Only when <code class="">Serialize()</code> is called does the wrapper flush the staging area into the final Flatbuffer binary, perfectly aligning the offsets.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69632" height="700" src="https://www.nutanix.dev/wp-content/uploads/2026/04/Amod-Jaltade-Flatbuffers-Fig-3_-Efficient-construction-of-flatbuffers-v2-1024x700.png" width="1024" /><figcaption class="wp-element-caption">Figure 3: Efficient construction of flatbuffers</figcaption></figure>
</div>


<h2 class="wp-block-heading">Results and Takeaways: Trading Bytes for CPU Cycles</h2>



<p>Benchmarks revealed the classical systems trade-off: we were trading network and disk footprint for massive CPU and memory efficiency.</p>



<p>Using representative scaled metadata payloads, here is what the relative performance revealed:</p>



<h3 class="wp-block-heading">The Massive Wins: CPU and Memory</h3>



<p>The primary goal of the migration was to eliminate the &#8220;serialization tax,&#8221; and the Flatbuffer architecture delivered spectacular reductions in overhead:</p>



<ul class="wp-block-list">
<li><strong>Zero-Cost Deserialization:</strong> As metadata entries grew in complexity, Protobuf deserialization time scaled linearly and steeply. In contrast, Flatbuffer deserialization time remained a completely flat line near zero, regardless of the entry size.</li>



<li><strong>Cumulative Speedup:</strong> Thanks to completely bypassing the parsing phase, the total API call time for batch lookups (where multiple metadata entries are looked up together) plummeted. At larger payload scales, Flatbuffers achieved a roughly <strong>60% reduction in total lookup latency</strong> compared to Protobufs.&nbsp; This is purely due to CPU savings.</li>



<li><strong>Reduced Memory Footprint:</strong> Unpacked Protobuf objects consume significant heap space. Flatbuffers proved to be much more memory-efficient when loaded into RAM, yielding a <strong>~40% reduction in memory consumption</strong> for the same data. This in turn, allows us to cache more metadata.</li>
</ul>



<h3 class="wp-block-heading">The Trade-Off: Payload Size and Network RPCs</h3>



<p>Flatbuffers achieve their zero-copy speed by storing data on disk and over the wire exactly as it is represented in memory, complete with padding and offsets.</p>



<ul class="wp-block-list">
<li><strong>The Disk/Wire Footprint:</strong> Protobuf is vastly superior at packing data densely. Our benchmarks showed that Flatbuffer payloads were roughly <strong>100% larger (double the size)</strong> on the wire compared to Protobuf.</li>



<li><strong>Network Transfer Times:</strong> Because Flatbuffer payloads are physically larger, the actual time spent pushing the bytes over the network during a batch lookup RPC call saw a <strong>~30% increase</strong>. However, the massive CPU time we saved by completely bypassing deserialization heavily outweighed this network penalty.</li>
</ul>



<h3 class="wp-block-heading">Additional Considerations &amp; Tooling</h3>



<p>Migrating a core serialization protocol across a massive codebase is notoriously difficult, but the success of the project came down to our investment in tooling and fallback mechanisms.</p>



<p>There are a few critical considerations that made this a truly seamless transition:</p>



<ol class="wp-block-list">
<li><strong>A Truly Zero-Touch Compiler Pipeline:</strong> We didn&#8217;t just write wrapper classes by hand; we modified the <code class="">flatc</code> compiler itself. The compiler takes the existing <code class="">.proto</code> metadata definitions as its input. During the build process, it automatically generates the corresponding <code class="">.fbs</code> file, the native Flatbuffer C++ code, and the bilingual Interface class. It is a completely automated pipeline.</li>



<li><strong>Graceful Handling of Legacy Metadata:</strong> We designed the system around &#8220;Magic Headers&#8221; , but terabytes of existing metadata on disk were written years before these headers existed. To handle this, our parsing logic includes a safe fallback. If the parser encounters an unrecognizable header (or no header at all), it gracefully falls back to treating the raw buffer as a legacy Protobuf. The system natively digests the old data and automatically attaches the new magic headers the next time that data is modified and written back to disk.</li>



<li><strong>A One-Line Code Change:</strong> Because the compiler auto-generates an interface that perfectly mimics the Protobuf setters and getters, and because the C++ staging area handles the complex Flatbuffer offset math, the feature developers had virtually no work to do. The <em>only</em> requirement for a feature team to adopt this massive architectural shift was a simple Find-and-Replace to update the class name being instantiated in their code.</li>
</ol>



<h2 class="wp-block-heading">Conclusion</h2>



<p>This project proved that in modern hyper-converged infrastructure, CPU cycles are often far more precious than payload bytes.</p>



<p>By taking the time to build a compiler-generated wrapper, a bilingual tri-state machine, and a defensive read-path, we migrated our core data path to zero-copy Flatbuffers with zero downtime. Most importantly, we protected our developer experience. Teams adopted a massive architectural shift simply by changing the class namespaces they instantiated, allowing product velocity to remain completely uninterrupted.</p>



<p>With rigorous systems engineering, and a little bit of compiler magic, you really can rebuild the airplane while it’s flying.</p>
