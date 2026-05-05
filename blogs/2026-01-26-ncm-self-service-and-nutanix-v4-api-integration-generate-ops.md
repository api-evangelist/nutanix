---
title: "NCM Self-Service and Nutanix v4 API Integration: Generate Ops Mgmt report using NCM Self-Service"
url: "https://www.nutanix.dev/2026/01/26/ncm-self-service-and-nutanix-v4-api-integration-generate-ops-mgmt-report-using-ncm-self-service/"
date: "Mon, 26 Jan 2026 15:00:00 +0000"
author: "Chris Rasmussen"
feed_url: "https://www.nutanix.dev/feed/"
---
<h2 class="wp-block-heading">Introduction</h2>



<p>Nutanix Intelligent Operations allows the generation of detailed reports.  These reports can be configured to include a number of fields or data points, resulting in timely information being sent to the people that need it.  When combined with NCM Self-Service Playbooks and NCM Self-Service Runbooks, a powerful, automated reporting system can be created, reducing the need for manual interaction or report creation.  In today&#8217;s quick demo we&#8217;ll use NCM Self-Service Playbooks to react to a specific event and, when that event occurs, execute an NCM Self-Service Runbook that will generate an Intelligent Operations report.  To demonstrate how an automated environment could integrate this approach into custom applications, the runbook will generate the report using the Nutanix v4 APIs.</p>



<p>Let&#8217;s get started.  Note: All images throughout this article can be clicked to view full size.</p>



<h2 class="wp-block-heading">Playbooks vs Runbooks</h2>



<p>Before looking at the demo, let&#8217;s take a quick look at the difference between an NCM Intelligent Operations Playbook and an NCM Self-Service Runbook.</p>



<h3 class="wp-block-heading">Runbooks</h3>



<p>An NCM Self-Service Runbook is a collection of tasks and procedures that be run across various endpoints.  For example, this demo&#8217;s runbook runs on two different endpoints:</p>



<ol class="wp-block-list">
<li>A Linux virtual machine that has been configured specifically for the purposes of running Python scripts</li>



<li>Prism Central, as the endpoint for Nutanix v4 API requests</li>
</ol>



<p>Runbooks are intended to be run either via the Prism Central UI or through some other intentional mechanism. Runbooks cannot be run on a scheduled basis unless combined with an NCM Intelligent Operations playbook. Runbook task types at the time of writing are available on the <a href="https://portal.nutanix.com/page/documents/details?targetId=Self-Service-Admin-Operations-Guide-v4_3_0:nuc-runbook-overview-c.html" rel="noreferrer noopener" target="_blank">Runbooks in Self-Service</a> portal page.</p>



<h3 class="wp-block-heading">Playbooks</h3>



<p>NCM Intelligent Operations Playbooks, whilst they can be run manually, enable powerful, event-triggered automations.  Triggers can include events such as a VM being created, a cluster event such a host entering maintenance mode or an alert.</p>



<p>The playbook used in this demo reacts to the &#8220;Created VM&#8221; event trigger and will execute every time a new VM is created.</p>



<p>Playbook actions at the time of writing are available on the <a href="https://portal.nutanix.com/page/documents/details?targetId=Intelligent-Operations-Guide-vpc_7_5:mul-automation-management-pc-c.html" rel="noreferrer noopener" target="_blank">Task Automation &#8211; Playbooks</a> portal page. </p>



<h2 class="wp-block-heading">Scenario</h2>



<p>Our demo environment has an existing Intelligent Operations report configuration named <strong>cr-report-config</strong>. The specific report configuration won&#8217;t impact the outcome of this demo, although you can download the demo&#8217;s report configuration from the <a href="https://github.com/nutanixdev/playbooks/tree/master/ncm_self_service_integration" rel="noreferrer noopener" target="_blank">NutanixDev GitHub</a>.</p>



<h3 class="wp-block-heading">Required Outcomes</h3>



<p>At the end of this demo, we will have an NCM Self-Service Playbook that responds to the creation of a VM.  The playbook will execute an NCM Self-Service runbook which, in turn, generates an Intelligent Operations report using the Nutanix v4 <strong>opsmgmt</strong> APIs.</p>



<h2 class="wp-block-heading">Prerequisites</h2>



<p>To follow along with this demo in your own environment, the following assumptions will be made.</p>



<ul class="wp-block-list">
<li>You have already created a Nutanix Prism Central Intelligent Operations report configuration
<ul class="wp-block-list">
<li>However, because this demo is intended to demonstrate the <em>execution</em> of a report configuration using NCM Self-Service, Intelligent Ops and REST APIs, the report configuration shown is for demo purposes only.</li>



<li>The report configuration used in this demo can be downloaded from the <a href="https://github.com/nutanixdev/playbooks/tree/master/ncm_self_service_integration" rel="noreferrer noopener" target="_blank">NutanixDev GitHub</a>.</li>
</ul>
</li>



<li>You have deployed a Linux-based VM that has been configured as an NCM Self-Service Python endpoint.  For detailed instructions on creating a Python endpoint, see the <a href="https://portal.nutanix.com/page/documents/details?targetId=Self-Service-Admin-Operations-Guide-v4_2_1:nuc-endpoints-overview-c.html" rel="noreferrer noopener" target="_blank">NCM Self-Service Administration and Operations Guide</a>.
<ul class="wp-block-list">
<li>The Python endpoint will need the <strong>requests</strong> and <strong>urllib3</strong> modules available in the virtual environment.</li>



<li>An example endpoint is shown in the following screenshot:</li>
</ul>
</li>
</ul>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69370" height="300" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-12-287x300.png" width="287" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Example NCM Self-Service endpoint for running Python scripts</figcaption></figure>
</div>


<ul class="wp-block-list">
<li>You have created an NCM Self-Service HTTP endpoint for your Prism Central instance
<ul class="wp-block-list">
<li>An example endpoint is shown in the following screenshot:</li>
</ul>
</li>
</ul>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69369" height="300" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-11-201x300.png" width="201" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Example NCM Self-Service endpoint for Prism Central</figcaption></figure>
</div>


<ul class="wp-block-list">
<li>You have <a href="https://github.com/nutanixdev/playbooks/tree/master/ncm_self_service_integration" rel="noreferrer noopener" target="_blank">downloaded</a> the<strong> cr-generate-report</strong> runbook used throughout this demo and imported it into your Prism Central instance.</li>



<li>Optionally, you can also <a href="https://github.com/nutanixdev/playbooks/tree/master/ncm_self_service_integration" rel="noreferrer noopener" target="_blank">download</a> the <strong>cr-demo-playbook</strong> playbook used in the final section of this demo.</li>
</ul>



<h2 class="wp-block-heading">Demo Environment</h2>



<p>The demo environment used throughout this article is as follows:</p>



<ul class="wp-block-list">
<li>Prism Central 7.5</li>



<li>AOS 7.5</li>



<li>NCM Self-Service 4.3.0</li>



<li>v4 API <code class="">opsmgmt</code> namespace v4.0</li>
</ul>



<h2 class="wp-block-heading">API Requests</h2>



<p>In order for this demo to succeed, we&#8217;ll need to send various requests using the Nutanix v4 APIs.  Specifically, the <code class="">opsmgmt</code> (Ops Management) v4.0 namespace will handle all the demo requests, including requesting existing configuration details and generating the final report.</p>



<p>Before getting started, let&#8217;s take a look at how a Prism Central Intelligent Operations report can be generated using a manual API request.  We&#8217;ll use Postman here, but the request itself will be identical regardless of the API prototyping tool used.</p>



<h3 class="wp-block-heading">Obtain Report Configuration extId</h3>



<p>Since we already know the report configuration is named <strong>cr-report-config</strong>, we can use Odata filters with our v4 API request.  The filter will request only those report configurations matching the required name.</p>



<ul class="wp-block-list">
<li>Request URL: <strong>https://{{pc_ip}}:9440/api/opsmgmt/v4.0/config/report-configs?$filter=name eq &#8216;cr-report-config&#8217;</strong></li>



<li>Method: <strong>GET</strong></li>



<li>Headers: <strong>Content-Type: application/json</strong></li>



<li>Payload: None required</li>
</ul>



<p>Looking at the response, we can see the report is available in the report configuration list.  For the following steps, the report&#8217;s <strong>extId</strong> must be noted down.  In an automated environment, this <strong>extId</strong> would be saved as a variable.  This example shows the report configuration is <strong>127b2178-2af5-4c27-51ec-6ebde983eebe</strong>.</p>



<p>Note: The full response has been trimmed for readability.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    ...
    &quot;data&quot;: [
        {
            ...
            &quot;extId&quot;: &quot;127b2178-2af5-4c27-51ec-6ebde983eebe&quot;,
            ...
            &quot;name&quot;: &quot;cr-report-config&quot;,
            &quot;sections&quot;: [
                ...
            ]
        }
    ],
    &quot;metadata&quot;: {
        ...
        &quot;totalAvailableResults&quot;: 1
    }
}</code></pre>



<h3 class="wp-block-heading">Generate Report</h3>



<p>With the report configuration&#8217;s <strong>extId</strong> available and noted down, the generate report request will be built as follows.</p>



<ul class="wp-block-list">
<li>Request URL: <strong>https://{{pc_ip}}:9440/api/opsmgmt/v4.0/config/reports</strong></li>



<li>Method: <strong>POST</strong></li>



<li>Headers:
<ul class="wp-block-list">
<li><strong>Content-Type: application/json</strong></li>



<li><strong>Ntnx-Request-Id: &lt;UUID&gt;</strong></li>
</ul>
</li>



<li>Payload:</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;name&quot;: &quot;ncm_selfservice_generated_report&quot;,
    &quot;configExtId&quot;: &quot;127b2178-2af5-4c27-51ec-6ebde983eebe&quot;,
    &quot;isPersistent&quot;: false,
    &quot;startTime&quot;: &quot;2025-10-01T00:00:00Z&quot;,
    &quot;endTime&quot;: &quot;2025-10-02T00:00:00Z&quot;,
    &quot;recipientFormats&quot;: [
        &quot;PDF&quot;
    ],
    &quot;recipients&quot;: [
        {
            &quot;emailAddress&quot;: &quot;<span class="apbct-email-encoder" title="This contact has been encoded by Anti-Spam by CleanTalk. Click to decode. To finish the decoding make sure that JavaScript is enabled in your browser.">no<span class="apbct-blur">******</span>@<span class="apbct-blur">**</span>me.com</span>&quot;,
            &quot;recipientName&quot;: &quot;Demo Recipient&quot;
        }
    ]
}</code></pre>



<p>When this request is sent, the report configuration <strong>cr-report-config</strong> will be used as the basis for a report sent to <strong><span class="apbct-email-encoder" title="This contact has been encoded by Anti-Spam by CleanTalk. Click to decode. To finish the decoding make sure that JavaScript is enabled in your browser.">no<span class="apbct-blur">******</span>@<span class="apbct-blur">**</span>me.com</span></strong>.  The recipient details need to be changed to match appropriate settings for your environment.</p>



<h2 class="wp-block-heading">NCM Self-Service Runbook</h2>



<p>Since the requirement for this demo is to manage this entire process with a NCM Self-Service, let&#8217;s look at the runbook that will handle the report generation.</p>



<p>To simplify this demo, the runbook has been made available for download.  If you have not already done so, download the runbook now and import it into your Prism Central instance.</p>



<h3 class="wp-block-heading">Runbook Flow</h3>



<p>The final runbook will look like this:</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69357" height="294" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-300x294.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Final runbook flow showing all steps</figcaption></figure>
</div>


<p>Here are the steps the runbook will follows:</p>



<ol class="wp-block-list">
<li>Request a matching report configuration.  If a matching config is not found, enter the <strong>False</strong> branch and run a simple Python script indicating the negative result.  If a matching runbook is found, enter the <strong>True</strong> branch.</li>



<li>Send an API request that returns the report config&#8217;s <strong>extId</strong>.  This could potentially be collapsed into the main decision branch but is split here for demo purposes.  The report config&#8217;s <strong>extId</strong> is returned from the response key&#8217;s <strong>$.data[0].extId</strong> field and saved in a variable named <strong>REPORT_CONFIG_EXTID</strong>.  This step also returns HTTP 200 (success) or 400/409 for failure/request not found.</li>



<li>Generates a random UUID using the Python standard library&#8217;s <strong>uuid</strong> module and saves it in a variable named <strong>REQUEST_ID</strong>.</li>



<li>Converts the report start and end dates into usable and formatted ISO-8601 that will work with the Nutanix v4 APIs.  These are returned as variables named <strong>REPORT_START_ISO8601</strong> and <strong>REPORT_END_ISO8601</strong>, respectively.</li>



<li>Lastly, sends a Nutanix v4 API request using the <strong>opsmgmt</strong> namespace.  This namespace provides report management functionality and is the same namespace used in previous steps.  The variables collected in previous steps are used as the various fields within the request&#8217;s POST payload.</li>
</ol>



<h3 class="wp-block-heading">Runbook Credentials</h3>



<p>The runbook requires two distinct credentials:</p>



<ol class="wp-block-list">
<li>Prism Central login with permissions to work with Intelligent Operations reports</li>



<li>Python endpoint credentials with permission to execute Python scripts within your endpoint</li>
</ol>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69358" height="201" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-1-300x201.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Runbook credentials required for successful execution.&nbsp; Note: Credentials can be basic authentication (username and password) or SSH key pair.&nbsp; This demo uses an SSH key pair credential.</figcaption></figure>
</div>


<h3 class="wp-block-heading">Runbook Input Variables</h3>



<p>To make this runbook as configurable as possible, a number of input variables have been specified.  This allows the user to supply input variables at runtime, if required:</p>



<ol class="wp-block-list">
<li>Report name: <strong>REPORT_NAME</strong></li>



<li>Recipient email address: <strong>RECIPIENT_EMAIL</strong></li>



<li>Recipient name: <strong>RECIPIENT_NAME</strong></li>



<li>Report configuration name: <strong>REPORT_CONFIG_NAME</strong></li>



<li>Report start and end times: <strong>REPORT_START</strong> and <strong>REPORT_END</strong></li>



<li>Prism Central IP address or FQDN: <strong>PC_IP</strong></li>
</ol>



<p>After uploading the sample runbook, make sure you edit these variables to match values appropriate for your environment.</p>



<h4 class="wp-block-heading">Important Note</h4>



<p>The runbook&#8217;s Python scripts use NCM Self-Service placeholders that are replaced by input variables at runtime.  For example, the downloadable sample runbook uses a Prism Central credential named &#8220;Prism Central Admin&#8221;.  Within the decision step, the script contains this step:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-python">username = &quot;@@{Prism Central Admin.username}@@&quot;</code></pre>



<p>This Python code creates a variable named <strong>username</strong> and sets that variable to the &#8220;Prism Central Admin&#8221; credential&#8217;s <strong>username</strong> field.</p>



<p>If you name your credentials and input variables something other than those used in the downloadable runbook, make sure you edit the placeholders in each runbook step.</p>



<h3 class="wp-block-heading">Runbook Output Variables</h3>



<p>Because this script contains return variables, we have also specified a number of Runbook Output Variables:</p>



<ul class="wp-block-list">
<li>Report end, formatted as ISO-8601: <strong>REPORT_END_ISO8601</strong></li>



<li>Report start, formatted as ISO-8601: <strong>REPORT_START_ISO8601</strong></li>



<li>Request ID, used for request idempotency: <strong>REQUEST_ID</strong></li>



<li>Report configuration <strong>extId</strong>: <strong>REPORT_CONFIG_EXTID</strong></li>
</ul>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69359" height="141" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-2-300x141.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Runbook output variables</figcaption></figure>
</div>


<h2 class="wp-block-heading">Intelligent Operations Playbook</h2>



<p>So far, we&#8217;ve got an NCM Self-Service runbook that takes pre-entered input from the user, checks for the existence of a specific report configuration and, if that configuration exists, generates an Intelligent Operations report based on that configuration.  This runbook can be executed at any time.</p>



<p>However, this demo requires the runbook to be executed when an event is triggered: VM creation.  In a production environment the report would likely generate different specific details, although the downloadable examples will be sufficient for this demo.</p>



<p>Let&#8217;s take a look at our playbook.</p>



<h3 class="wp-block-heading">Playbook Creation</h3>



<p>Our demo has been configured as an <strong>Event triggered</strong> playbook during the playbook creation step. This enables the playbook to run when a specific event is triggered; in this case, whenever a VM is created.</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69361" height="164" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-3-300x164.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Selecting &#8220;Event&#8221; trigger during playbook creation</figcaption></figure>
</div>


<p>The type of event can be specified&nbsp;after the event trigger is selected. Nutanix Intelligent Operations provides over 40 different triggers, from cluster and node activity to failover and maintenance events. Our demo playbook will only be triggered when a VM is created. As you can see in the following screenshot, the VM creation trigger has been left intentionally broad but could also be configured to only trigger when VMs are created within a specific category.</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69362" height="109" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-4-300x109.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Selecting &#8220;Created VM&#8221; as the playbook event trigger</figcaption></figure>
</div>


<h3 class="wp-block-heading">Playbook Actions</h3>



<p>With the event trigger specified, it is now time to select what will happen when the event is triggered.  Nutanix Intelligent Operations provides a large selection of actions, from alert and warning acknowledgment through to virtual hardware actions such as eject CD-ROM or power off VM.  In our demo, we want to execute an existing NCM Self-Service Runbook.  After clicking the <strong>Add Action</strong> option, we can select <strong>Execute a Self Service Runbook</strong> from the list of available actions:</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69363" height="150" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-5-300x150.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Selecting &#8220;Execute a Self Service Runbook&#8221; as the playbook action</figcaption></figure>
</div>


<p>Our demo uses a non-marketplace runbook, so after selecting our project, we must change the <strong>Runbook Type</strong> from <strong>Marketplace</strong> to <strong>Non-Marketplace</strong> before selecting the existing runbook.</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69365" height="173" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-7-300x173.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Project and runbook selection for the playbook action</figcaption></figure>
</div>


<p>Although we won&#8217;t use the option in this demo, the runbook can be set to <strong>Stop</strong> if action fails.</p>



<p>Scrolling down within the action settings, the previously specified runbook <strong>Input Variables</strong> can be set.  Under normal circumstances these would be set once here and used during  subsequent runbook executions.  If you are following this demo in your own environment, make sure you set this appropriately before continuing.</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69366" height="198" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-8-300x198.png" width="300" /><button class="lightbox-trigger" type="button">
			<svg fill="none" height="12" viewBox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg">
				<path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff">
			</svg>
		</button><figcaption class="wp-element-caption">Option to specify input variables for runbook executions</figcaption></figure>
</div>


<p>When all settings have been specified, the playbook can be <strong>Saved and Closed</strong>.</p>



<p>Note: Intelligent Operations playbooks are <strong>DISABLED</strong> by default, unless they are explicitly enabled when being saved.</p>


<div class="wp-block-image">
<figure class="aligncenter size-medium wp-lightbox-container"><img alt="" class="wp-image-69367" height="173" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-9-300x173.png" width="300" /><button
			class="lightbox-trigger"
			type="button"
			aria-haspopup="dialog"
			aria-label="Enlarge"0
			data-wp-init="callbacks.initTriggerButton"
			data-wp-on--click="actions.showLightbox"
			data-wp-style--right="state.imageButtonRight"
			data-wp-style--top="state.imageButtonTop"
		>
			<svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="none" viewBox="0 0 12 12">
				<path fill="#fff" d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" />
			</svg>
		</button><figcaption class="wp-element-caption">Clicking &#8220;Save &amp; Close&#8221; and setting the playbook status to &#8220;Enabled&#8221;</figcaption></figure>
</div>


<h2 class="wp-block-heading">Testing the Automation</h2>



<p>Before continuing with our first test, let&#8217;s take a quick look at what we&#8217;ve accomplished so far.</p>



<ol class="wp-block-list">
<li>Uploaded a sample runbook that looks for a specific Intelligent Operations report configuration, gets the report configuration&#8217;s unique <strong>extId</strong>, generates a UUID for use in the upcoming POST request and generates a report using the configuration, <strong>ONLY</strong> if the configuration was found.</li>



<li>Created a Nutanix Intelligent Operations playbook that watches for VMs to be created and, when the VM Created event is triggered, executes our NCM Self-Service runbook using previously-populated input variables.</li>
</ol>



<p>Now, if we create a VM using any supported method &#8211; Prism UI, API, SDK etc &#8211; the <strong>cr-demo-playbook</strong> will trigger the <strong>Created VM</strong> event and execute the NCM Self-Service <strong>cr-generate-report</strong> runbook.</p>



<p>The following screenshot confirms that when a VM called <strong>v</strong> was created, NCM Intelligent Operations triggered the event and executed the runbook.</p>


<div class="wp-block-image">
<figure data-wp-context="{&quot;imageId&quot;:&quot;69f7db210a924&quot;}" data-wp-interactive="core/image" data-wp-key="69f7db210a924" class="aligncenter size-medium wp-lightbox-container"><img loading="lazy" decoding="async" width="300" height="75" data-wp-class--hide="state.isContentHidden" data-wp-class--show="state.isContentVisible" data-wp-init="callbacks.setButtonStyles" data-wp-on--click="actions.showLightbox" data-wp-on--load="callbacks.setButtonStyles" data-wp-on-window--resize="callbacks.setButtonStyles" src="https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-300x75.png" alt="" class="wp-image-69368" srcset="https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-300x75.png 300w, https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-1024x257.png 1024w, https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-768x193.png 768w, https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-1536x386.png 1536w, https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-2048x514.png 2048w, https://www.nutanix.dev/wp-content/uploads/2025/12/image-10-1568x394.png 1568w" sizes="(max-width: 300px) 100vw, 300px" /><button
			class="lightbox-trigger"
			type="button"
			aria-haspopup="dialog"
			aria-label="Enlarge"1
			data-wp-init="callbacks.initTriggerButton"
			data-wp-on--click="actions.showLightbox"
			data-wp-style--right="state.imageButtonRight"
			data-wp-style--top="state.imageButtonTop"
		>
			<svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="none" viewBox="0 0 12 12">
				<path fill="#fff" d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" />
			</svg>
		</button><figcaption class="wp-element-caption">Confirmation of successful runbook execution from playbook event trigger</figcaption></figure>
</div>


<h2 class="wp-block-heading">Conclusion</h2>



<p>In this demo, we have seen how various Nutanix products and features can be cleanly integrated, resulting in a highly configurable, automated reporting setup:</p>



<ul class="wp-block-list">
<li>A Nutanix Intelligent Operations report configuration containing VM performance data</li>



<li>An NCM Self-Service runbook to generate a report based on the report configuration</li>



<li>A Nutanix Intelligent Operations playbook to watch for VM creation events which then executes the playbook</li>
</ul>



<h2 class="wp-block-heading">Related Resources</h2>



<ul class="wp-block-list">
<li><a href="https://portal.nutanix.com/page/documents/details?targetId=Self-Service-Admin-Operations-Guide-v4_2_1:nuc-endpoints-overview-c.html" target="_blank" rel="noreferrer noopener">NCM Self-Service Endpoints</a></li>



<li><a href="https://portal.nutanix.com/page/documents/details?targetId=Self-Service-Admin-Operations-Guide-v4_2_1:nuc-runbook-overview-c.html" target="_blank" rel="noreferrer noopener">NCM Self-Service Runbooks</a></li>



<li><a href="https://portal.nutanix.com/page/documents/details?targetId=Intelligent-Operations-Guide-vpc_7_5:mul-automation-management-pc-c.html" target="_blank" rel="noreferrer noopener">NCM Intelligent Operations Playbooks</a></li>
</ul>
