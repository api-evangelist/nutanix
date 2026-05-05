---
title: "Part 2 – Scale From Zero Node Pools in NKP: Testing and Validating the Setup"
url: "https://www.nutanix.dev/2026/04/22/part-1-scale-from-zero-node-pools-in-nkp-how-it-works-and-how-to-configure-it-2/"
date: "Wed, 22 Apr 2026 14:00:00 +0000"
author: "Jose Gomez"
feed_url: "https://www.nutanix.dev/feed/"
---
<h2 class="wp-block-heading">Introduction</h2>



<p>In <strong><a href="https://www.nutanix.dev/2026/04/15/part-1-scale-from-zero-node-pools-in-nkp-how-it-works-and-how-to-configure-it/" rel="noreferrer noopener" target="_blank">Part 1 of this series</a></strong>, we explained how <strong>Scale From Zero works in NKP</strong>, including how Cluster Autoscaler evaluates empty node pools and how to configure the required capacity annotations.</p>



<p>If you missed it, it’s worth reading <strong><a href="https://www.nutanix.dev/2026/04/15/part-1-scale-from-zero-node-pools-in-nkp-how-it-works-and-how-to-configure-it/" rel="noreferrer noopener" target="_blank">Part 1 first</a></strong>, since it covers the concepts and configuration that enable scaling from zero nodes.</p>



<p>In this post, we focus on <strong>testing and validation</strong>. We will trigger real scaling scenarios, observe how the autoscaler reacts when node pools start at zero, and verify that workloads correctly cause new nodes to be provisioned.</p>



<p>By the end, you’ll have a repeatable approach for confirming that your <strong>Scale From Zero configuration works as expected in practice</strong>.</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">Deploy a Workload That Targets the Scale from Zero Pool</h2>



<p>To see the behavior of Cluster Autoscaler with a scale from zero node pool, we are going to test this in three steps:</p>



<ul class="wp-block-list">
<li>Force an unscheduled workload. By only setting the label and not the taint</li>



<li>Add the taint to trigger the worker node creation (from 0 → 1 node replica)</li>



<li>Delete the workload to decommission the worker (from 1 → 0 node replica)</li>
</ul>



<h3 class="wp-block-heading"><strong>Unscheduled workload (label only)</strong></h3>



<p>The workload must have the label <code class="">node-role.kubernetes.io/worker</code> defined. Otherwise, without the label, it will be scheduled to the default NKP node pool, <code class="">md-0</code>.&nbsp;</p>



<p>The following manifest is a single replica deployment asking for a node with the label <code class="">node-role.kubernetes.io/worker: &lt;your_nodepool_name_value&gt;&nbsp;</code></p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p><strong>Note</strong>: you must complete the steps in part 1 before running the following command, otherwise the environment variable <code class="">$NODEPOOL_NAME</code> won&#8217;t be set</p>
</blockquote>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl apply -f -&lt;&lt;EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: burst-workload
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: burst
  template:
    metadata:
      labels:
        app: burst
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: $NODEPOOL_NAME
      containers:
      - name: app
        image: ghcr.io/traefik/whoami:v1.11
        resources:
          requests:
            cpu: &quot;1&quot;
            memory: &quot;512Mi&quot;
EOF</code></pre>



<p>When this <code class="">Deployment</code> is created:</p>



<ul class="wp-block-list">
<li>Pod is unschedulable</li>



<li>Autoscaler will check <code class="">MachineDeployments</code> that meet the criteria</li>



<li>Autoscaler won’t scale from <code class="">0 → 1 </code>because no matches</li>



<li>No node is provisioned</li>



<li>Pod remains unscheduled</li>
</ul>



<p>Check the status of the pod to confirm it is <code class="">Pending</code>:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get pod -l app=burst --namespace default</code></pre>



<p>You should see a similar output:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">NAME                              READY   STATUS    RESTARTS   AGE
burst-workload-6f4c8ffd7c-jcxdm   1/1     Pending   0          53s</code></pre>



<p>You can also check <code class="">MachineDeployment</code> to confirm that no nodes have been created yet.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get machinedeployment --namespace $NAMESPACE</code></pre>



<h3 class="wp-block-heading">Scheduled workload (label &amp; taint required)</h3>



<p>Let’s update the <em>Deployment</em> to match the tolerations to the taint.&nbsp;</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl apply -f -&lt;&lt;EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: burst-workload
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: burst
  template:
    metadata:
      labels:
        app: burst
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: $NODEPOOL_NAME
      tolerations:
      - key: &quot;dedicated&quot;
        operator: &quot;Equal&quot;
        value: &quot;${NODEPOOL_NAME}&quot;
        effect: &quot;NoSchedule&quot;
      containers:
      - name: app
        image: ghcr.io/traefik/whoami:v1.11
        resources:
          requests:
            cpu: &quot;1&quot;
            memory: &quot;512Mi&quot;
EOF</code></pre>



<p>When the Deployment is updated:</p>



<ul class="wp-block-list">
<li>Pod is still unscheduled</li>



<li>Autoscaler scales <code class="">MachineDeployment</code> from <code class="">0 → 1</code> (it might take a few minutes)
<ul class="wp-block-list">
<li>In Prism Central, you can check for incoming tasks of a new virtual machine creation</li>
</ul>
</li>



<li>Node is provisioned</li>



<li>Pod schedules</li>
</ul>



<p>Check back the <code class="">MachineDeployment</code> to confirm it is <code class="">ScalingUp</code>.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get machinedeployment --namespace $NAMESPACE</code></pre>


<div class="wp-block-image">
<figure class="aligncenter size-full"><img alt="" class="wp-image-69717" height="53" src="https://www.nutanix.dev/wp-content/uploads/2026/04/image-3.png" width="759" /></figure>
</div>


<p>Once the worker node is ready, the pod should be in <code class="">Running</code> status.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get pod -l app=burst --namespace default</code></pre>



<p>You should see a similar output:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">NAME                             READY   STATUS    RESTARTS   AGE
burst-workload-bf6997679-42lm5   1/1     Running   0          3m52s</code></pre>



<h3 class="wp-block-heading">Scaling back to zero</h3>



<p>The last step would be to scale the <code class="">Deployment</code> down to zero replicas or completely delete it. As a best practice, we’ll keep the <code class="">Deployment</code> and just set the number of replicas to zero.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl scale --replicas=0 deployment/burst-workload --namespacec default</code></pre>



<p>Confirm that the <code class="">Deployment</code> shows <code class="">0/0</code> pods.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get deployment burst-workload --namespacec default</code></pre>



<p>When the <code class="">Deployment</code> is scaled down to zero:</p>



<ul class="wp-block-list">
<li>Node becomes unneeded</li>



<li>After cooldown timers (default: 10 minutes)</li>



<li>Machine is drained</li>



<li>Replicas return to 0</li>
</ul>



<p>After 10 minutes, check the <code class="">MachineDeployment</code> to confirm the node was deleted. If it isn’t deleted, check that you don’t have any workloads with labels and taints that request this node pool.</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">Important Production Considerations</h2>



<h3 class="wp-block-heading">Autoscaler Timers</h3>



<p>Scale-down is not immediate. Expect delays controlled by flags in Cluster Autoscaler, such as:</p>



<ul class="wp-block-list">
<li><code class="">--scale-down-delay-after-add</code> (default: 10 minutes)</li>



<li>&#8211;<code class="">-scale-down-unneeded-time</code> (default: 10 minutes)</li>
</ul>



<p>This is intentional and prevents aggressive churn. NKP uses the default values.&nbsp;</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h3 class="wp-block-heading">MachineHealthCheck (MHC) Interaction</h3>



<p>If <code class="">MachineHealthCheck</code> is enabled:</p>



<ul class="wp-block-list">
<li>Ensure <code class="">nodeStartupTimeout</code> is not too aggressive (default: 10 minutes)</li>



<li>Avoid remediation loops for ephemeral pools</li>



<li>Consider excluding scale-from-zero pools from <code class="">MHC</code> if necessary</li>
</ul>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h3 class="wp-block-heading">DaemonSets Do Not Block Scale-Down</h3>



<p>System <code class="">DaemonSets</code> (CNI, CSI, etc.) are ignored during scale-down evaluation. They will not prevent a node from being removed.</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">When NOT to Use Scale From Zero</h2>



<p>Scale from zero node pools is not a disaster recovery strategy.</p>



<p>In a DR architecture, the best practice is to:</p>



<ul class="wp-block-list">
<li>Maintain at least one active instance</li>



<li>Continuously validate data replication</li>



<li>Route a portion of production or synthetic traffic</li>



<li>Exercise networking and storage paths</li>
</ul>



<p>If your DR site runs at zero capacity and you fail over during an incident, you risk discovering at the worst possible moment:</p>



<ul class="wp-block-list">
<li>Storage attachment issues</li>



<li>Secret replication failures</li>



<li>Certificate expiration</li>



<li>Network misconfiguration</li>



<li>Image pull failures</li>
</ul>



<p>Scale-from-zero is ideal for:</p>



<ul class="wp-block-list">
<li>Batch jobs</li>



<li>On-demand compute</li>



<li>CI pipelines</li>



<li>Ephemeral workloads</li>



<li>GPU burst capacity</li>
</ul>



<p>It is not a substitute for a warm DR environment.</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">Final Thoughts</h2>



<p>Scale from zero node pools is one of the most effective infrastructure optimizations available in Kubernetes today. It allows you to:</p>



<ul class="wp-block-list">
<li>Align infrastructure cost with real demand</li>



<li>Eliminate idle compute waste</li>



<li>Maintain clean workload isolation</li>



<li>Deliver reactive platform engineering</li>
</ul>



<p>Used correctly, it reduces both cost and operational sprawl. Used carelessly, it can introduce reliability risk in critical systems.</p>



<p>The key is understanding where elasticity provides value — and where readiness must remain constant.</p>
