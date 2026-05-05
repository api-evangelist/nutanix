---
title: "Nutanix v4 API Batches: A Technical Use Case"
url: "https://www.nutanix.dev/2026/04/13/nutanix-v4-api-batches-a-technical-use-case/"
date: "Mon, 13 Apr 2026 14:00:00 +0000"
author: "Chris Rasmussen"
feed_url: "https://www.nutanix.dev/feed/"
---
<h2 class="wp-block-heading">Introduction</h2>



<p>In July 2024, we published a two-part series covering the technical usage of Nutanix v4 API Batch operations. That series focused specifically on <a href="https://www.nutanix.dev/2024/07/08/nutanix-v4-apis-batch-operations-part-1-create-modify/" rel="noreferrer noopener" target="_blank">CREATE and MODIFY</a> operations in the first part, and batch <a href="https://www.nutanix.dev/2024/07/10/nutanix-v4-api-batch-operations-part-2-action/" rel="noreferrer noopener" target="_blank">ACTIONS</a> in the second part, both using the Nutanix v4 Python SDK.</p>



<p>In today&#8217;s article we&#8217;ll look at the &#8220;why&#8221;, as well as a potential real-world use case for the Nutanix v4 batch APIs provided by the <code class="">prism</code> namespace. To achieve this, we&#8217;ll demonstrate that use case with a practical, REST API example.</p>



<p>If you are new to the Nutanix v4 APIs, see the <a href="https://www.nutanix.dev/nutanix-api-user-guide/" rel="noreferrer noopener" target="_blank">Nutanix v4 API User Guide</a> for getting started info.</p>



<h2 class="wp-block-heading">Why?</h2>



<p>To begin, consider the following scenario.</p>



<ol class="wp-block-list">
<li>A request is received that requires the creation of 10 virtual machines. For the purposes of this article, assume the following:
<ul class="wp-block-list">
<li>2 of the new virtual machines are destined to be database servers</li>



<li>8 of the new virtual machines are destined to be web servers</li>
</ul>
</li>



<li>The approaches outlined here apply even when the request is significantly larger</li>
</ol>



<p>Note: We&#8217;ll focus only on the deployment of the virtual machine infrastructure only and not the deployment of the database or web server software.</p>



<h3 class="wp-block-heading">Option 1: The &#8220;old&#8221; way</h3>



<p>Depending on the infrastructure system or API versions in use, it may be possible to achieve the desired using 10 separate API requests.  This requires the following steps:</p>



<ul class="wp-block-list">
<li>Construction of 10 individual REST API payloads.</li>



<li>Submission of 10 individual REST API requests.</li>



<li>A minimum of 10 individual REST API requests to check on the status of each entity create request, although this assumes each related task is only checked once.</li>



<li>In total, a minimum of 30 individual API requests to deploy 10 new virtual machines.</li>
</ul>



<p>While this will achieve the desired result, it will generate additional complexity and the submission of additional network traffic in the form of individual requests.  In some environments, the minimization of unnecessary network traffic is a key requirement; 10 individual requests may not meet this requirement.  This can be especially important in remotely-managed or high-latency networks.</p>



<h4 class="wp-block-heading">Repetitive Requests</h4>



<p>For each VM created here, the payload would need to be similar to this example; the trimmed section would contain environment specific details such as storage configuration, network configuration (etc).</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;name&quot;: &quot;db1&quot;,
    &quot;description&quot;: &quot;batchvm_db{vm_number}&quot;,
    &quot;cluster&quot;: {
        &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
    }
    ...
}</code></pre>



<p>At first glance this looks like a simple payload &#8211; and it is! However, it would need to be modified and sent as 10 individual requests for this approach to work. Because batching of API requests is a direct answer to the inefficiencies of repeated requests, we can do better.</p>



<h3 class="wp-block-heading">Option 2: The &#8220;new&#8221; way</h3>



<p>Batch processing can alleviate the complex task of managing and monitoring 10 individual requests by doing exactly as the name suggests: grouping the requests into batches i.e. smaller, more manageable chunks that can be easily monitored, up to 500 entities at a time.</p>



<p>For our example, this requires building a single payload for the database servers and a single payload for the web servers.  Then, instead of sending 2 requests for the database servers and 8 requests for the web servers, we can build and send a single batch request that encapsulates all 10 requests for the 10 required servers.</p>



<p>Note: All batch operations are managed by the <code class="">prism</code> v4 API namespace.  The operations <em>within</em> the batch request will define which APIs are used for each individual request; in this case, create VM from the <code class="">vmm</code> namespace.</p>



<h2 class="wp-block-heading">The Solution</h2>



<h3 class="wp-block-heading">Assumptions</h3>



<p>This article is not intended as a getting started guide for the Nutanix v4 APIs. As such, the demo cluster <code class="">extId</code>: <code class="">577251cb-351c-4768-a869-2766fafc3289</code> is already available.  In a production scenario this would likely be obtained by using the <code class="">clustermgmt</code> v4 API namespace to get the details of the VM host cluster.</p>



<h3 class="wp-block-heading">Payload metadata</h3>



<p>Prism batch requests consist of two main parts:</p>



<ol class="wp-block-list">
<li>Metadata: the per-batch metadata e.g. the type of request, API request path for each subtask and what to do if an error occurs.  The <code class="">metadata</code> object also contains action-specific details if the batch action acts on existing entities.  This could be updating a VM or adding a VM to a category.</li>



<li>Data: the per-entity data e.g. for creating a VM this would include the VM name, description and spec properties such as CPU configuration, RAM, storage and network details.</li>
</ol>



<p>Here&#8217;s an example of the request metadata <strong>object</strong> for our 10 VM request.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;metadata&quot;: {
        &quot;action&quot;: &quot;CREATE&quot;,
        &quot;name&quot;: &quot;10 VMs created with Nutanix v4.2 APIs&quot;,
        &quot;uri&quot;: &quot;/api/vmm/v4.2/ahv/config/vms&quot;,
        &quot;shouldStopOnError&quot;: false,
        &quot;chunkSize&quot;: 1
    },</code></pre>



<p>This metadata clearly shows properties matching those described above:</p>



<ul class="wp-block-list">
<li><code class="">action</code>: <strong>Create</strong> an entity/entities.</li>



<li><code class="">name</code>: The <strong>name</strong> of the batch request.</li>



<li><code class="">uri</code>: The URI/<strong>endpoint</strong> to use for each subtask; in this example the path is the <code class="">vmm</code> namespace&#8217;s VM create endpoint.</li>



<li><code class="">shouldStopOnError</code>: For our example, we do NOT want to stop the entire batch process if one of the VM create requests should fail. If there are VM dependencies that would cause problems later, this would be set to <code class="">true</code></li>



<li><code class="">chunkSize</code>: How many VMs to create at a time.  Note <code class="">chunkSize</code> is not intended as a performance enhancement.</li>
</ul>



<h3 class="wp-block-heading">Payload data</h3>



<p>With the request metadata built, we can move on to building the <code class="">data</code> part of the request.  The <code class="">data</code> <strong>list</strong> specifies the spec for each affected entity &#8211; new VMs in our example.</p>



<p>How the user builds the <code class="">data</code> list will depend on the requirements.  This could be a script loop or an imported JSON file.  For our example, let&#8217;s assume the VM specs are as follows:</p>



<ul class="wp-block-list">
<li>Database VMs
<ul class="wp-block-list">
<li><code class="">name</code>: <code class="">db{vm_number}</code> i.e. <code class="">db1</code> and <code class="">db2</code></li>



<li><code class="">description</code>: <code class="">batchvm_db{vm_number}</code></li>



<li>vRAM in GiB: <code class="">1</code></li>



<li>All other specs are left as default</li>
</ul>
</li>



<li>Web server VMs:
<ul class="wp-block-list">
<li><code class="">name</code>: <code class="">web{vm_number}</code> i.e. <code class="">web1</code> &#8230; <code class="">web8</code></li>



<li><code class="">description</code>: <code class="">batchvm_web{vm_number}</code></li>



<li>vRAM in GiB: <code class="">1</code></li>



<li>All other specs are left as default</li>
</ul>
</li>
</ul>



<p>Programmatically, it wouldn&#8217;t make much sense to manually type or create one large JSON for these specs. In a script or app, the payload would be built iteratively, that is, a loop that builds a single VM payload at each iteration and appends it to the <code class="">data</code> <strong>list</strong>.</p>



<p>This approach would produce a JSON payload data <strong>list</strong> similar to this example:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">&quot;payload&quot;: [
    {
        &quot;data&quot;: {
            &quot;name&quot;: &quot;db1&quot;,
            &quot;description&quot;: &quot;batchvm_db1&quot;,
            &quot;memory_size_bytes&quot;: 1073741824,
            &quot;cluster&quot;: {
                &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
            }
        }
    },
    ...
    {
        &quot;data&quot;: {
            &quot;name&quot;: &quot;web1&quot;,
            &quot;description&quot;: &quot;batchvm_web1&quot;,
            &quot;memory_size_bytes&quot;: 1073741824,
            &quot;cluster&quot;: {
                &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
            }
        }
    },
    ...
]</code></pre>



<h3 class="wp-block-heading">Send the request batch</h3>



<p>With the batch payload constructed, it is now a simple case of sending the request as follows.</p>



<ul class="wp-block-list">
<li>URL: <strong>https://{{pc_ip}}:9440/api/prism/v4.2/operations/$actions/batch</strong></li>



<li>Method: <strong>POST</strong></li>



<li>Payload: The complete JSON payload constructed so far; a copy of the complete payload will be in Appendix A at the end of this article.</li>
</ul>



<h2 class="wp-block-heading">So &#8230; why?</h2>



<p>Using legacy methods, every request&#8217;s response would need to be parsed, the task <code class="">extId</code> for that task extracted, and the <code class="">prism</code> namespace&#8217;s <code class="">tasksApi</code> used to gain visibility into the results of that task.  As discussed earlier, the &#8220;why&#8221; of batches is to increase efficiency and to gain a better insight into large collections of related requests.</p>



<p>The Nutanix v4 API batch operations avoid the need to process a potential multitude of individual requests, a process that can take time and comparatively manual effort.</p>



<h3 class="wp-block-heading">Batch Success</h3>



<p>Let&#8217;s look at the response from a successful batch request.  The main response contains a single task ID i.e. the batch task itself:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;data&quot;: {
        &quot;$objectType&quot;: &quot;prism.v4.config.TaskReference&quot;,
        &quot;$reserved&quot;: {
            &quot;$fv&quot;: &quot;v4.r2&quot;
        },
        &quot;extId&quot;: &quot;ZXJnb24=:3cade39c-6199-470f-5bf5-2cea80f6f3a2&quot;
    }
}</code></pre>



<p>Using the <code class="">prism</code> namespace&#8217;s tasks endpoints, the task details can be quickly retrieved and parsed; the full response has been trimmed for readability.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;data&quot;: {
        &quot;extId&quot;: &quot;ZXJnb24=:657959ff-b4ff-4b62-4a46-b2f14d24ed0b&quot;,
        &quot;operation&quot;: &quot;BATCH-CREATE&quot;,
        &quot;operationDescription&quot;: &quot;10 VMs created with Nutanix v4.2 APIs&quot;,
        &quot;createdTime&quot;: &quot;2026-03-19T05:30:08.911013Z&quot;,
        &quot;startedTime&quot;: &quot;2026-03-19T05:30:08.921785Z&quot;,
        &quot;completedTime&quot;: &quot;2026-03-19T05:30:20.56036Z&quot;,
        &quot;progressPercentage&quot;: 100,
        &quot;entitiesAffected&quot;: [
            {
                &quot;extId&quot;: &quot;5d8f74b9-ac76-4b1f-5a21-533d47ee4183&quot;,
                &quot;rel&quot;: &quot;vmm:ahv:config:vm&quot;,
                &quot;name&quot;: &quot;batch_vm_db1&quot;,
                &quot;$reserved&quot;: {
                    &quot;$fv&quot;: &quot;v4.r2&quot;
                },
                &quot;$objectType&quot;: &quot;prism.v4.config.EntityReference&quot;
            },
            ...
        ],
        &quot;subTasks&quot;: [
            {
                &quot;$reserved&quot;: {
                    &quot;$fv&quot;: &quot;v4.r2&quot;
                },
                &quot;$objectType&quot;: &quot;prism.v4.config.TaskReferenceInternal&quot;,
                &quot;extId&quot;: &quot;ZXJnb24=:3ea4ab04-0d4b-512e-9830-7cfeff99df44&quot;,
                &quot;href&quot;: &quot;https://10.0.0.1:9440/api/prism/v4.2/config/tasks/ZXJnb24=:3ea4ab04-0d4b-512e-9830-7cfeff99df44&quot;,
                &quot;rel&quot;: &quot;subtask&quot;
            },
            ...
        ],
        ...
        &quot;numberOfSubtasks&quot;: 10,
        &quot;numberOfEntitiesAffected&quot;: 10,
        ...
        &quot;status&quot;: &quot;SUCCEEDED&quot;,
        ...
    },
    ...
    &quot;metadata&quot;: {
        &quot;flags&quot;: [
            {
                ...
                &quot;name&quot;: &quot;hasError&quot;,
                &quot;value&quot;: false
            },
            ...
        ],
        ...
    }
}</code></pre>



<p>The key points to notice here are as follows.</p>



<ul class="wp-block-list">
<li>All 10 VMs have been created with a single request</li>



<li>All 10 VMs were successfully created, with related entity information available in the <code class="">entitiesAffected</code> list</li>



<li>For each of the 10 new VMs, the related VM creation details are available in the <code class="">subTasks</code> list</li>



<li>Within the <code class="">metadata</code> object, the <code class="">hasError</code> flag indicates there were no errors encountered during the parent batch request</li>



<li>If required, you can still parse the full response, get each individual task ID and request that task for more detailed per-entity information. For example, if we request the task relating to the <code class="">web2</code> VM, the response contains the following details (trimmed for readability):</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;data&quot;: {
        &quot;extId&quot;: &quot;ZXJnb24=:3ea4ab04-0d4b-512e-9830-7cfeff99df44&quot;,
        &quot;operation&quot;: &quot;CreateVm&quot;,
        &quot;operationDescription&quot;: &quot;Create VM&quot;,
        &quot;createdTime&quot;: &quot;2026-03-19T05:30:19.320126Z&quot;,
        &quot;startedTime&quot;: &quot;2026-03-19T05:30:19.327126Z&quot;,
        &quot;completedTime&quot;: &quot;2026-03-19T05:30:19.858228Z&quot;,
        &quot;progressPercentage&quot;: 100,
        &quot;entitiesAffected&quot;: [
            {
                &quot;extId&quot;: &quot;30465c06-517c-4acd-559d-5e5e9a9bf6db&quot;,
                &quot;rel&quot;: &quot;vmm:ahv:config:vm&quot;,
                &quot;name&quot;: &quot;web2&quot;,
                &quot;$reserved&quot;: {
                    &quot;$fv&quot;: &quot;v4.r2&quot;
                },
                &quot;$objectType&quot;: &quot;prism.v4.config.EntityReference&quot;
            }
        ],
        ...
        &quot;status&quot;: &quot;SUCCEEDED&quot;,
    }
    ...
    &quot;metadata&quot;: {
        &quot;flags&quot;: [
            {
                ...
                &quot;name: &quot;hasError&quot;,
                &quot;value&quot;: false
            },
            ...
        ]
    }
}</code></pre>



<h3 class="wp-block-heading">Batch Error</h3>



<p>Conversely, introducing an intentional error into the batch details returns a clear indication that something failed.  By setting the <code class="">shouldStopOnError</code> property to <code class="">false</code>, the entire batch will continue even if one or more of the VM creation subtasks fails.  For example, setting the cluster <code class="">extId</code> to an invalid value will respond as follows; this time we&#8217;ll just show the relevant parts of the response.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;data&quot;: {
        &quot;entitiesAffected&quot;: [
           ...
        ],
        ...
        &quot;errorMessages&quot;: [
            {
                &quot;$reserved&quot;: {
                    &quot;$fv&quot;: &quot;v4.r2&quot;
                },
                &quot;$objectType&quot;: &quot;prism.v4.error.AppMessage&quot;,
                &quot;message&quot;: &quot;Operation failed due to legacy error and &#039;legacyErrorMessage&#039; should be referred to for details&quot;,
                &quot;severity&quot;: &quot;ERROR&quot;,
                &quot;code&quot;: &quot;TSKS-20801&quot;,
                &quot;locale&quot;: &quot;en_US&quot;,
                &quot;errorGroup&quot;: &quot;LEGACY_ERROR&quot;
            }
        ],
        &quot;legacyErrorMessage&quot;: &quot;1 subtask has failed.&quot;,
        ...
        &quot;numberOfSubtasks&quot;: 10,
        &quot;numberOfEntitiesAffected&quot;: 9,
        ...
    ...
}</code></pre>



<p>Here we can see that of the 10 entities that were requested, 9 were &#8220;affected&#8221;, indicating that 1 of the requested entities could not be processed. The <code class="">entitiesAffected</code> list then be parsed to easily find out which entity could not be processed.</p>



<p>Important note: In the current release of the Nutanix v4 APIs the complete failed task details are not yet available via API. This is a known limitation, although task details can still be observed in the Prism UI.</p>



<h2 class="wp-block-heading">Conclusion</h2>



<p>As shown in this article, payload batching can be a more efficient way of handling large related entity operations compared to legacy approaches. Even though this article demonstrated the batch creation of &#8220;only&#8221; 10 new VMs, up to 500 entities can be batched at a time.  In this example, a single request initiated the batch operation and a single request obtained details of that batch operation.  That&#8217;s a total of only 2 requests, vs a potential total of 20 requests for legacy methods.</p>



<p>For Nutanix v4 REST API <code class="">prism</code> namespace batching operations, see <a href="https://developers.nutanix.com/api-reference?namespace=prism&amp;version=v4.2#tag/Batches" rel="noreferrer noopener" target="_blank">Batches</a> in the <a href="https://developers.nutanix.com/" rel="noreferrer noopener" target="_blank">Nutanix v4 API documentation</a>.</p>



<h2 class="wp-block-heading">Appendix A: Complete JSON Payload</h2>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{
    &quot;metadata&quot;: {
        &quot;action&quot;: &quot;CREATE&quot;,
        &quot;name&quot;: &quot;10 VMs created with Nutanix v4.2 APIs&quot;,
        &quot;uri&quot;: &quot;/api/vmm/v4.2/ahv/config/vms&quot;,
        &quot;shouldStopOnError&quot;: false,
        &quot;chunkSize&quot;: 1
    },
    &quot;payload&quot;: [
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;db1&quot;,
                &quot;description&quot;: &quot;batchvm_db1&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;db2&quot;,
                &quot;description&quot;: &quot;batchvm_db2&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web1&quot;,
                &quot;description&quot;: &quot;batchvm_web1&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web2&quot;,
                &quot;description&quot;: &quot;batchvm_web2&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web3&quot;,
                &quot;description&quot;: &quot;batchvm_web3&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web4&quot;,
                &quot;description&quot;: &quot;batchvm_web4&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web5&quot;,
                &quot;description&quot;: &quot;batchvm_web5&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web6&quot;,
                &quot;description&quot;: &quot;batchvm_web6&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web7&quot;,
                &quot;description&quot;: &quot;batchvm_web7&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web8&quot;,
                &quot;description&quot;: &quot;batchvm_web8&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        },
        {
            &quot;data&quot;: {
                &quot;name&quot;: &quot;web9&quot;,
                &quot;description&quot;: &quot;batchvm_web9&quot;,
                &quot;memory_size_bytes&quot;: 1073741824,
                &quot;cluster&quot;: {
                    &quot;extId&quot;: &quot;577251cb-351c-4768-a869-2766fafc3289&quot;
                }
            }
        }
    ]
}</code></pre>



<p></p>
