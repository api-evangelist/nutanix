---
title: "Nutanix v4 SDK: API Version Negotiation in Prism Central 7.5"
url: "https://www.nutanix.dev/2026/01/23/nutanix-v4-sdk-api-version-negotiation-in-prism-central-7-5/"
date: "Fri, 23 Jan 2026 15:00:00 +0000"
author: "Chris Rasmussen"
feed_url: "https://www.nutanix.dev/feed/"
---
<h2 class="wp-block-heading">Introduction</h2>



<p>In today&#8217;s quick article we&#8217;re going to cover a new Nutanix v4 SDK and Prism Central <strong>7.5</strong> feature specifically aimed at providing &#8220;Version Negotiation&#8221;.  This feature does not need to be manually enabled; it is enabled by default.</p>



<h2 class="wp-block-heading">Scenario</h2>



<p>In today&#8217;s mock scenario our initial configuration is as follows.</p>



<ul class="wp-block-list">
<li>SDK client uses the Nutanix v4.0.1 Python SDK; this article will use the <code class="">vmm</code> Virtual Machine Management namespace</li>



<li>The API endpoint is Prism Central version 7.3.1</li>
</ul>



<p>In this environment, a basic request would be as follows.</p>



<ul class="wp-block-list">
<li>Client uses the appropriate SDK function to request a list of VMs i.e. <code class="">ntnx_vmm_py_client.api.VmApi.list_vms()</code></li>



<li>The SDK sends the request to <em>/api/vmm/<strong>v4.0</strong>/vms</em></li>



<li>Because the SDK and Prism Central versions are compatible, the server responds with a list of VMs</li>
</ul>



<p>However, if the client upgrades to the latest SDK version <em>without</em> upgrading Prism Central, the exact same SDK function will fail with an HTTP 404 NOT FOUND error. The failure is caused by the new SDK version sending requests to an API endpoint that is not yet supported by Prism Central. In this scenario, the configuration is now as follows:</p>



<ul class="wp-block-list">
<li>SDK client version 4.2.1 (upgraded)</li>



<li>Prism Central version 7.3.1 (unchanged)</li>
</ul>



<p>In this environment, the request would be as follows:</p>



<ul class="wp-block-list">
<li>Client uses the same SDK function to request a list of VMs i.e. <code class="">ntnx_vmm_py_client.api.VmApi.list_vms()</code></li>



<li>The SDK, because it has been upgraded, sends the request to <em>/api/vmm/<strong>v4.2</strong>/vms</em></li>



<li>Because the SDK and Prism Central versions are <strong>not</strong> compatible, the request fails with an HTTP 404 NOT FOUND error</li>
</ul>



<p>The process of version negotiation is specifically designed to prevent this type of error.</p>



<h2 class="wp-block-heading">What is Version Negotiation?</h2>



<p>Version negotiation is a process that enables an SDK client such as the Nutanix v4 Python SDK to communicate with a server providing an API such as Prism Central and, at the same time, ensure the SDK uses API endpoints compatible with the deployed version of Prism Central.</p>



<p>Note: This feature applies to all Nutanix v4 SDKs &#8211; Python, Java, JavaScript and Go.  Python has been used as an example.</p>



<p>With Version Negotiation available, the improved request process is as follows.</p>



<ul class="wp-block-list">
<li>Client uses an SDK function or method to request a list of VMs</li>



<li>The SDK, because it has been upgraded, sends the request to <em>/api/vmm/<strong>v4.2</strong>/vms</em></li>



<li>Prism Central 7.5 (upgraded) and the client SDK agree on a mutually supported API version</li>



<li>The agreed API version is then used to service the request, ensuring the response is as expected and, most importantly, does not result in an HTTP 404 NOT FOUND error.</li>
</ul>



<h2 class="wp-block-heading">Examples</h2>



<p>Using the Nutanix v4 Python SDK, let&#8217;s test what we&#8217;ve learnt so far. Note this quick demo is being run on a Linux workstation.</p>



<h3 class="wp-block-heading">Setup</h3>



<p>To begin, a Python virtual environment has been created and activated.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-bash"># create the environment
python -m venv venv
# activate
. venv/bin/activate</code></pre>



<h3 class="wp-block-heading">Test 1: Expected Success</h3>



<p>Environment:</p>



<ul class="wp-block-list">
<li>Nutanix v4 Python SDK <strong>v4.0.1</strong></li>



<li>Prism Central <strong>7.3.1</strong></li>
</ul>



<ol class="wp-block-list">
<li>Install specific version dependencies.</li>
</ol>



<pre class="wp-block-prismatic-blocks"><code class="language-bash"># install Nutanix VMM Python SDK, specifically version 4.0.1
pip install ntnx_vmm_py_client==4.0.1
# enter Python REPL
python</code></pre>



<ol class="wp-block-list" start="2">
<li>Run a quick test, noting the VM list is returned as expected. </li>
</ol>



<pre class="wp-block-prismatic-blocks"><code class="language-python"># import dependencies
import ntnx_vmm_py_client
from ntnx_vmm_py_client import Configuration
from ntnx_vmm_py_client import ApiClient

# disable SSL certificate warnings; not recommended in production
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# setup the configuration
config = Configuration()
config.host = &quot;10.0.0.10&quot;
config.port = &quot;9440&quot;
config.username = &quot;admin&quot;
config.password = &quot;nutanix/4u&quot;
config.verify_ssl = False
config.debug = False

client = ApiClient(configuration=config)
vmm_instance = ntnx_vmm_py_client.api.VmApi(api_client=client)

# list VMs
vm_list = vmm_instance.list_vms(async_req=False)</code></pre>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69385" height="548" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-13-1024x548.png" width="1024" /><figcaption class="wp-element-caption">Quick Python script to list VMs in a supported SDK + Prism Central environment</figcaption></figure>
</div>


<p>As expected, this returns a list of VMs from our Prism Central instance and the response object confirms we have a list of VMs:</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69386" height="89" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-14-1024x89.png" width="1024" /><figcaption class="wp-element-caption">14 VMs are available in our demo environment</figcaption></figure>
</div>


<h3 class="wp-block-heading">Test 2: Expected Failure</h3>



<p>Environment:</p>



<ul class="wp-block-list">
<li>Nutanix v4 Python SDK <strong>v4.2.1</strong> (upgraded)</li>



<li>Prism Central <strong>7.3.1</strong> (unchanged)</li>
</ul>



<ol class="wp-block-list">
<li>Exit the Python REPL with <code class="">exit()</code> and upgrade the <code class="">ntnx_vmm_py_client</code> SDK.</li>
</ol>



<pre class="wp-block-prismatic-blocks"><code class="">exit()
# Python VMM namespace version 4.2.1 is the latest at the time of writing
pip install ntnx_vmm_py_client==4.2.1
# re-enter the Python REPL
python</code></pre>



<ol class="wp-block-list" start="2">
<li>Run the same script as before without making any changes, observing that the exact same script now fails with HTTP 404 NOT FOUND:</li>
</ol>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69387" height="564" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-15-1024x564.png" width="1024" /><figcaption class="wp-element-caption">Expected failure due to SDK upgrade before Prism Central is upgraded to 7.5</figcaption></figure>
</div>


<p>However, note the error message also contains the following text:</p>



<pre class="wp-block-prismatic-blocks"><code class="">2025-12-23 05:38:43,628Z WARNING [MainThread:ntnx_vmm_py_client.api_client:983] Server version v4.1 is below minimum supported version v4.2. Version negotiation will not be performed.</code></pre>



<p>This is an indication that the new <strong>v4.2.1</strong> version of the Python SDK is version negotiation aware, but Prism Central does not yet support that feature.</p>



<h2 class="wp-block-heading">Current and future state</h2>



<p>Because Prism Central 7.5 and Python SDK v4.2.1 (the latest at the time of writing) are both version negotiation aware, it is logical these these two versions are compatible with each other.  SDK functions will function as normal.</p>



<p>However, if the SDK is upgraded to a later version at some point in the future <em>without</em> upgrading Prism Central at the same time, Prism Central will be able to use version negotiation to ensure the correct API version is consumed by the SDK.</p>



<p>This will remove the possibility of HTTP 404 NOT FOUND errors from occurring when using the Nutanix v4 SDKs and ensure a smoother, easier to debug experience when using the Nutanix v4 SDKs.</p>



<p>Thanks for reading.</p>



<h2 class="wp-block-heading">Related Resources</h2>



<ul class="wp-block-list">
<li><a href="https://www.nutanix.dev/api-reference-v4/" rel="noreferrer noopener" target="_blank">Nutanix v4 API Introduction</a></li>



<li><a href="https://www.nutanix.dev/nutanix-api-user-guide/" rel="noreferrer noopener" target="_blank">Nutanix v4 API User Guide</a></li>



<li><a href="https://developers.nutanix.com/" rel="noreferrer noopener" target="_blank">Nutanix v4 API Documentation</a></li>
</ul>
