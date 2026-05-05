---
title: "Launch VM console outside Nutanix Prism UI"
url: "https://www.nutanix.dev/2026/05/01/vm-console-external-access/"
date: "Fri, 01 May 2026 14:00:00 +0000"
author: "Chris Rasmussen"
feed_url: "https://www.nutanix.dev/feed/"
---
<p>The Nutanix VM console was released in H1 2026. Since then, the Nutanix VM console has been primarily accessible through the Prism Central web interface. It is used as an option to debug and solve critical issues when RDP is not available. Customers have been increasingly asking for a way to open a VM console programmatically from automation pipelines, external portals, and custom tooling.&nbsp;</p>



<p>With the new <strong>v4 VM Console API</strong>, Nutanix now provides a secure, standards-based, token-driven mechanism for launching VM consoles outside of the Prism Central web interface.</p>



<p>The details on the API are available in the Nutanix API Reference under the namespace Virtual Machine Management (v4.2+) <a href="https://developers.nutanix.com/api-reference?namespace=vmm&amp;version=v4.2" rel="noreferrer noopener" target="_blank">here</a>.</p>



<p>Below, we will walk through:</p>



<ul class="wp-block-list">
<li>How the new v4 architecture works</li>



<li>What is different from the traditional console model</li>



<li>How to generate a console token</li>



<li>How to launch the VM console using your own client or application</li>
</ul>



<h2 class="wp-block-heading">Why a New VM Console API?</h2>



<p>Historically, launching a VM console was not available through public APIs. Instead it required a complex workaround using Prism Central to construct a WebSocket connection to the VM’s VNC endpoint through Acropolis and it relied on a number of internal mechanisms that were not designed for this use.&nbsp;</p>



<p>The new v4 VM console design introduces the following:</p>



<ul class="wp-block-list">
<li>A <strong>public, supported API</strong> for generating a VM console token</li>



<li>A <strong>WebSocket URI</strong> that the client can connect to directly</li>



<li>A <strong>JWT-based authorization model</strong></li>



<li>A secure proxy path between Nutanix services</li>
</ul>



<p><strong>This opens the door for automation, CI/CD access, custom dashboards, and third-party integrations.</strong></p>



<h2 class="wp-block-heading">Process Overview</h2>



<p>When choosing to setup a client to launch a VM console, the process can look like this:</p>



<h3 class="wp-block-heading">Client calls the v4 “Generate Console Token” API</h3>



<p>This is an async task API that returns:</p>



<ul class="wp-block-list">
<li>A <strong>WebSocket URI</strong></li>



<li>A <strong>JWT console token</strong><br /></li>
</ul>



<h3 class="wp-block-heading">Client opens a WebSocket connection to the URI</h3>



<p>This is a <strong>WSS</strong> (TLS-secured) WebSocket request.<br />The JWT token is passed as a query parameter.</p>



<h3 class="wp-block-heading">Nutanix infrastructure validates the token</h3>



<p>Prism Central authenticates the request using:</p>



<ul class="wp-block-list">
<li>Your Prism user session cookie</li>
</ul>



<p>Prism Central service validates the JW token:</p>



<ul class="wp-block-list">
<li>The JWT console token (contains VM UUID, user ID, expiration)</li>
</ul>



<h3 class="wp-block-heading">Connection is proxied to the VM</h3>



<p>The backend establishes a secure bidirectional tunnel from:</p>



<ul class="wp-block-list">
<li>Client → Prism Central → AHV Gateway → VM socket</li>
</ul>



<p>Once the handshake completes, the console behaves exactly as it would in the Prism web interface.</p>



<h2 class="wp-block-heading">Step-by-Step Guide</h2>



<p>Below is a simple workflow that can be used to launch a VM console using the new API.</p>



<p><strong>Permissions for </strong><strong><em>Generate_VM_Console_Token</em></strong><strong>are required.</strong> This can be provided through the Prism Central RBAC (role-based access control) settings. The current system defined roles which already include this permission today include: Super Admin, Prism Admin, Project Admin, Project Manager, Developer, Consumer, Operator, and Virtual Machine Operator.  </p>



<h3 class="wp-block-heading">Step 1 — Generate a VM Console Token</h3>



<p>Make a POST request to generate the console token.</p>



<pre class="wp-block-prismatic-blocks"><code class="">POST https://&lt;PC-IP&gt;:9440/api/vmm/v4.0/ahv/config/vms/&lt;VM_UUID&gt;/$actions/generate-console-token</code></pre>



<p>Include authentication headers (basic auth or session cookie).</p>



<p><strong>Task UUID </strong>will be returned.</p>



<p>Poll the <code class="">GenerateVmConsoleToken</code> API from above until it completes.</p>



<h3 class="wp-block-heading">Step 2 — Retrieve Token + WebSocket URI</h3>



<p>When the task finishes, the task completion details in response includes:</p>



<ul class="wp-block-list">
<li><strong>console_websocket_uri</strong></li>



<li><strong>console_token</strong> (JWT)</li>
</ul>



<p>Example:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
  &quot;WsUri&quot;: &quot;console/launch/&lt;VM_UUID&gt;&quot;,
  &quot;VmConsoleToken&quot;: &quot;&lt;jwt-token&gt;&quot;
}</code></pre>



<h3 class="wp-block-heading">Step 3 — Connect via WebSocket</h3>



<p>Your client should open the WebSocket URL similar to the following:</p>



<pre class="wp-block-prismatic-blocks"><code class="">wss://&lt;PC-IP&gt;:9440/console/launch/&lt;VM_UUID&gt;?VmConsoleToken=&lt;jwt-token&gt;</code></pre>



<p><strong>You must also include your session cookie</strong> (or bearer token) so the system can validate your identity.</p>



<p>Any NoVNC WebSocket-capable client will work. In addition to the NoVNC client, a Proxy client may also be required to ensure that it can establish the connection between the no VNC client and the Metropolis websocket server (WSS call).</p>



<p>Once connected, your tool can send/receive VNC frames exactly like Prism.</p>


<div class="wp-block-image">
<figure class="aligncenter size-full"><img alt="" class="wp-image-69763" height="540" src="https://www.nutanix.dev/wp-content/uploads/2026/04/image-5.png" width="960" /></figure>
</div>


<h2 class="wp-block-heading">Security Considerations</h2>



<p>The v4 model adds multiple layers of protection:</p>



<h3 class="wp-block-heading">JWT token</h3>



<p>Signed by a Nutanix-managed private key. Encodes:</p>



<ul class="wp-block-list">
<li>VM UUID</li>



<li>User UUID</li>



<li>Expiration (1 hour)</li>
</ul>



<h3 class="wp-block-heading">User authentication</h3>



<p>The caller must still be logged into Prism Central and have the correct permission to generate a VM console token.</p>



<h3 class="wp-block-heading">Per-VM connection limits</h3>



<p>AHV Gateway enforces a <strong>maximum of 32 connections per VM</strong>.</p>



<h3 class="wp-block-heading">Short-lived authorizations</h3>



<p>Tokens cannot be reused beyond expiration.</p>



<p>This design helps keep the console secure even when accessed outside Prism.</p>



<h2 class="wp-block-heading">Conclusion</h2>



<p>We have demonstrated how the generate-console-token api can be utilized by a client to authenticate and then launch the VM Console. This unlocks the potential to include VM Console in custom cloud management tools and use this tool in your preferred location. For example a Service Provider could integrate this direction into their self-service portal for tenants.</p>
