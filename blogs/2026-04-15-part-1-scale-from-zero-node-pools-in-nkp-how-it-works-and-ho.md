---
title: "Part 1 – Scale From Zero Node Pools in NKP: How It Works and How to Configure It"
url: "https://www.nutanix.dev/2026/04/15/part-1-scale-from-zero-node-pools-in-nkp-how-it-works-and-how-to-configure-it/"
date: "Wed, 15 Apr 2026 14:00:00 +0000"
author: "Jose Gomez"
feed_url: "https://www.nutanix.dev/feed/"
---
<h2 class="wp-block-heading">Introduction</h2>



<p>Kubernetes clusters are often overprovisioned, with dedicated node pools created for batch workloads, data processing, CI pipelines, GPU jobs, seasonal traffic, or specialized application tiers. These pools offer isolation and flexibility, but they often remain underutilized for extended periods, quietly accumulating infrastructure costs without delivering value.</p>



<p>The Nutanix Kubernetes Platform (NKP) supports <strong>scaling node pools from zero</strong>. Instead of keeping capacity provisioned by default, NKP allows node pools to be defined with a minimum size of zero. When a workload demands infrastructure resources, NKP automatically provisions additional capacity. This shifts the cluster from static, pre-allocated infrastructure to elastic capacity, aligning operational costs directly with actual workload demand.</p>



<p>This post is <strong>Part 1 of a two-part series on Scale From Zero in NKP</strong>. Here we explain how it works and how to configure it. In <strong><a href="https://www.nutanix.dev/2026/04/22/part-1-scale-from-zero-node-pools-in-nkp-how-it-works-and-how-to-configure-it-2/" rel="noreferrer noopener" target="_blank">Part 2</a></strong>, we’ll focus on testing and validating the setup.</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">How Scale From Zero Works in NKP</h2>



<p>NKP leverages upstream <a href="https://cluster-api.sigs.k8s.io/tasks/automated-machine-management/autoscaling" rel="noreferrer noopener" target="_blank">Cluster Autoscaler on Cluster API</a>. Autoscaling from zero is an opt-in enhancement that infrastructure providers can implement. The method described in this blog is agnostic to the infrastructure provider, meaning it should work with any provider included in NKP.</p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p><strong>Disclaimer</strong>: This method has only been tested with the Cluster API provider for Nutanix AHV (CAPX)</p>
</blockquote>



<p>At a high level, the flow is:</p>



<ol class="wp-block-list">
<li>You configure:
<ul class="wp-block-list">
<li>A node pool with a minimum of zero and a maximum size.</li>



<li>You add extra configuration to the generated MachineDeployment:
<ol class="wp-block-list">
<li>Capacity annotations.</li>



<li>Labels.</li>



<li><a href="https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/" rel="noreferrer noopener" target="_blank">Taints</a> (anti-affinity).</li>
</ol>
</li>
</ul>
</li>



<li>When you deploy a workload matching the node pool labels and taints, Cluster Autoscaler:
<ul class="wp-block-list">
<li>Uses capacity annotations to simulate node resources, since no real nodes exist yet.</li>



<li>Validates that the pod would fit</li>



<li>Scales the <code class="">MachineDeployment</code></li>



<li>Triggers node creation</li>
</ul>
</li>



<li>When workloads disappear:
<ul class="wp-block-list">
<li>Nodes become “unneeded.”</li>



<li>After cooldown timers:
<ol class="wp-block-list">
<li>Machines are drained and deleted</li>



<li>Replicas return to 0</li>
</ol>
</li>
</ul>
</li>
</ol>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">Step-by-Step: Creating a Scale From Zero Pool in NKP</h2>



<h3 class="wp-block-heading">Prerequisites</h3>



<ul class="wp-block-list">
<li>NKP cluster (any edition) running on Nutanix AHV</li>



<li>NKP CLI</li>



<li>yq (version 4 &#8211; <a href="https://github.com/mikefarah/yq?tab=readme-ov-file#install" rel="noreferrer noopener" target="_blank">install guide</a>)</li>
</ul>



<h3 class="wp-block-heading">Step 1 — Create a Node Pool</h3>



<p>Let’s start by creating a <code class="">.env</code> file with all the required variables and values for the different steps you’ll be walking through.</p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p><strong>Note</strong>: All the commands must be executed against the NKP Management cluster, which hosts the Cluster API resources.</p>
</blockquote>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session"># .env file

# If you don&#039;t know the cluster name run: kubectl get cluster -A
# NAMESPACE   NAME          CLUSTERCLASS          PHASE         AGE   VERSION
# default     nkp-2-nodes   nkp-nutanix-v2.17.0   Provisioned   34h   v1.34.1
export CLUSTER=&lt;cluster_name&gt;

# Namespace for your NKP cluster
export NAMESPACE=&lt;cluster_namespace&gt;

# Nutanix AHV cluster name
export NUTANIX_CLUSTER=&lt;prism_element_cluster_name&gt;

# Name for the node pool. Ex.: scale-from-zero
export NODEPOOL_NAME=&lt;nodepool_name&gt;

# Infrastructure resources
export NODEPOOL_VCPU=8
export NODEPOOL_MEMORY=32
export NODEPOOL_SUBNET=&lt;subnet_name&gt;

# Must match the name with the image already uploaded in Prism Central. Ex.: nkp-rocky-9.6-release-cis-1.34.1-20251206060914.qcow2
export NODEPOOL_VM_IMAGE=&lt;vm_image&gt;

# Generated manifest file
export FILE=$CLUSTER-scale-from-zero-nodepool.yaml</code></pre>



<p>With the environment variables file ready, let’s generate an updated <code class="">Cluster</code> manifest that includes the new node pool. We are doing a <code class="">--dry-run</code> because we need to make some tweaks to the manifest before we can apply it.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">source .env

nkp create nodepool nutanix $NODEPOOL_NAME \
  --namespace $NAMESPACE \
  --cluster-name $CLUSTER \
  --prism-element-cluster $NUTANIX_CLUSTER \
  --vcpus $NODEPOOL_VCPU \
  --memory $NODEPOOL_MEMORY \
  --subnets $NODEPOOL_SUBNET \
  --vm-image $NODEPOOL_VM_IMAGE \
  --replicas 0 \
  --dry-run \
  --output yaml &gt; $FILE</code></pre>



<p>If you take a look at the generated manifest file, you’ll see at the bottom of it the addition of the new node pool.</p>


<div class="wp-block-image">
<figure class="aligncenter size-full"><img alt="" class="wp-image-69658" height="598" src="https://www.nutanix.dev/wp-content/uploads/2026/04/image-1.png" width="633" /></figure>
</div>


<h3 class="wp-block-heading">Step 2 — Add Extra Configuration</h3>



<p>Before applying it, we must add the following information to our <code class="">MachineDeployment</code> (new node pool):</p>



<ul class="wp-block-list">
<li>Max size configuration</li>



<li>Autoscaler capacity annotations</li>



<li>Label</li>



<li>Taint</li>
</ul>



<p>These are required for scale-from-zero to work correctly.</p>



<p>To ensure the changes are applied correctly, we’ll be using the following <code class="">yq</code> snippet. It will:</p>



<ul class="wp-block-list">
<li>Filter by the new <code class="">MachineDeployment</code></li>



<li>Add the following capacity annotations (values will be different in your case):
<ul class="wp-block-list">
<li><code class="">capacity.cluster-autoscaler.kubernetes.io/cpu: 8</code></li>



<li><code class="">capacity.cluster-autoscaler.kubernetes.io/memory: 32G</code></li>



<li><code class="">capacity.cluster-autoscaler.kubernetes.io/labels: node-role.kubernetes.io/worker=demo</code></li>



<li><code class="">capacity.cluster-autoscaler.kubernetes.io/taints: dedicated=demo:NoSchedule</code></li>
</ul>
</li>



<li>Update the annotation <code class="">cluster.x-k8s.io/cluster-api-autoscaler-node-group-max-size</code></li>



<li>Add a metadata label to the <code class="">MachineDeployment</code> with the API <code class="">node-role.kubernetes.io</code> to ensure it propagates to the nodes when deployed</li>



<li>Add the taint <code class="">dedicated=demo:NoSchedule</code> to the <code class="">workerConfig</code> variables</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">source .env

yq -i &#039;
  (.spec.topology.workers.machineDeployments[]
    | select(.name == env(NODEPOOL_NAME))
  ) |= (
    # --- autoscaler annotations ---
    .metadata.annotations |= (
      . + {
        &quot;capacity.cluster-autoscaler.kubernetes.io/cpu&quot;: strenv(NODEPOOL_VCPU),
        &quot;capacity.cluster-autoscaler.kubernetes.io/memory&quot;: &quot;\(env(NODEPOOL_MEMORY))G&quot;,
        &quot;capacity.cluster-autoscaler.kubernetes.io/labels&quot;: &quot;node-role.kubernetes.io/worker=\(env(NODEPOOL_NAME))&quot;,
        &quot;capacity.cluster-autoscaler.kubernetes.io/taints&quot;: &quot;dedicated=\(env(NODEPOOL_NAME)):NoSchedule&quot;
      }
      | .[&quot;cluster.x-k8s.io/cluster-api-autoscaler-node-group-max-size&quot;] = strenv(NODEPOOL_MAX_SIZE)
    )
    |
    # --- labels ---
    .metadata.labels |= (
      . + {
        &quot;node-role.kubernetes.io/worker&quot;: env(NODEPOOL_NAME)
      }
    )
    |
    # --- persistent taints ---
    (.variables.overrides[]
      | select(.name == &quot;workerConfig&quot;)
      | .value.taints
    ) = [
      {
        &quot;effect&quot;: &quot;NoSchedule&quot;,
        &quot;key&quot;: &quot;dedicated&quot;,
        &quot;value&quot;: env(NODEPOOL_NAME)
      }
    ]
  )
&#039; &quot;$FILE&quot;</code></pre>



<p>Before applying the new changes, confirm that the manifest file has the correct annotations, label, and taint.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">cat $FILE</code></pre>


<div class="wp-block-image">
<figure class="aligncenter size-full"><img alt="" class="wp-image-69657" height="751" src="https://www.nutanix.dev/wp-content/uploads/2026/04/image-2.png" width="759" /></figure>
</div>


<p>If everything looks alright, now it’s time to apply the manifest.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl apply -f $FILE</code></pre>



<p>If you check the <code class="">MachineDeployments</code>, you’ll see the new node pool created with no values.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get machinedeployment --namespace $NAMESPACE</code></pre>


<div class="wp-block-image">
<figure class="aligncenter size-full"><img alt="" class="wp-image-69656" height="53" src="https://www.nutanix.dev/wp-content/uploads/2026/04/image.png" width="755" /></figure>
</div>


<h4 class="wp-block-heading">Why Capacity Annotations Matter</h4>



<p>When replicas = 0, no nodes exist. Autoscaler must simulate scheduling against a <strong>virtual template node</strong>. Without CPU and memory capacity annotations, scale-from-zero will fail.</p>



<hr class="wp-block-separator has-alpha-channel-opacity" />



<h2 class="wp-block-heading">Final Thoughts</h2>



<p>Scale from zero node pools is one of the most effective infrastructure optimizations available in Kubernetes today. When implemented correctly, they allow platforms to align infrastructure with real workload demand and eliminate idle compute resources.</p>



<p>However, this capability depends on understanding how the autoscaler evaluates node capacity, how scaling from zero works, and how to correctly configure node pools so the scheduler can make safe scaling decisions.</p>



<p>With the fundamentals and configuration in place, the next step is to validate that scaling behaves as expected in real-world scenarios.</p>



<p>In <strong><a href="https://www.nutanix.dev/2026/04/22/part-1-scale-from-zero-node-pools-in-nkp-how-it-works-and-how-to-configure-it-2/" rel="noreferrer noopener" target="_blank">Part 2</a></strong>, we will explain how to test scale from zero behavior and verify that workloads correctly trigger node creation.</p>



<p></p>
