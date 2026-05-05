---
title: "Avoid These Kubernetes Anti-Patterns: A Guide for Virtualization Professionals"
url: "https://www.nutanix.dev/2026/04/01/avoid-these-kubernetes-anti-patterns-a-guide-for-virtualization-professionals/"
date: "Wed, 01 Apr 2026 14:00:00 +0000"
author: "Jose Gomez"
feed_url: "https://www.nutanix.dev/feed/"
---
<p>If you’re a virtualization or infrastructure administrator moving into Kubernetes, the biggest challenge isn’t learning new tools; it’s treating Kubernetes like a form of virtualization using VM-style operations.</p>


<div class="wp-block-image">
<figure class="alignleft size-full is-resized"><img alt="Nutanix Kubernetes Platform - Certified Kubernetes" class="wp-image-69595" height="806" src="https://www.nutanix.dev/wp-content/uploads/2026/03/certified_kubernetes_color-1.png" style="width: 92px; height: auto;" width="596" /></figure>
</div>


<p><a href="https://www.nutanix.com/products/kubernetes-management-platform" rel="noreferrer noopener" target="_blank">Nutanix Kubernetes Platform (NKP)</a> is a CNCF-certified, 100% upstream-compatible Kubernetes platform designed to help platform teams transition from VM-style operations to fully automated, declarative platform engineering. NKP bundles lifecycle management, GitOps, disaster recovery, storage, networking, security, and multi-cluster operations into a single supported distribution, eliminating the need to assemble and operate a large DIY Kubernetes toolchain.</p>



<p>These practices are often referred to as <strong>Day-2 Kubernetes operations</strong>, <strong>Kubernetes lifecycle management</strong>, and <strong>platform engineering best practices</strong>.</p>



<p>Before we dive into the anti-patterns, here are the most important mindset shifts to keep in mind.</p>



<ul class="wp-block-list">
<li>Kubernetes nodes are <strong>immutable and replaceable</strong>, not assets to restore.</li>



<li>GitOps should be the <strong>primary deployment model</strong>, not manual <code class="">kubectl</code> usage.</li>



<li>Production Kubernetes requires <strong>multi-cluster disaster recovery</strong>, not stretched clusters.</li>



<li>Enterprise Kubernetes platforms should include <strong>RBAC, PKI, storage, and lifecycle automation</strong> out of the box.</li>
</ul>



<p>Let’s walk through the anti-patterns we encounter most often when VM operations meet Kubernetes.</p>



<h2 class="wp-block-heading"><strong>At a Glance: The Mindset Shift</strong></h2>



<figure class="wp-block-table"><table class="has-fixed-layout"><tbody><tr><td><strong>Anti-Pattern</strong></td><td><strong>Risk Introduced</strong></td><td><strong>NKP Capability</strong></td><td><strong>Operational Outcome</strong></td></tr><tr><td>VM snapshots for nodes</td><td>Configuration drift and failed restores</td><td>Cluster API lifecycle</td><td>Nodes replaced automatically</td></tr><tr><td>SSHing into nodes</td><td>Drift and non-reproducible fixes</td><td>Declarative config with CAPI</td><td>Consistent fleet configuration</td></tr><tr><td>“kubectl apply” deployments</td><td>No audit trail</td><td>Built-in Flux CD GitOps</td><td>Self-healing deployments</td></tr><tr><td>Static node IPs</td><td>Scale-out and upgrade failures</td><td>Dynamic IPAM and load balancing</td><td>Safe scaling and upgrades</td></tr><tr><td>Stretched clusters</td><td>etcd quorum risk</td><td>Multi-cluster DR with NDK</td><td>Site-level resiliency</td></tr><tr><td>“cluster-admin” everywhere</td><td>High blast radius</td><td>RBAC roles + IdP integration</td><td>Secure multi-tenancy</td></tr><tr><td>Manual certificates</td><td>Expiry outages</td><td>Auto-renewal with cert-manager</td><td>Automated PKI lifecycle</td></tr><tr><td>Local node storage</td><td>Data loss</td><td>Nutanix CSI + NDK</td><td>Durable persistent storage</td></tr></tbody></table></figure>



<h2 class="wp-block-heading">Anti-Pattern 1 — Backing Up Nodes Instead of Protecting State</h2>



<p><strong>The Anti-Pattern:</strong> Treating a Kubernetes node like a traditional VM by taking snapshots and backups. In virtualization, a failed server is &#8220;restored&#8221; from a snapshot. In Kubernetes, nodes are <strong>cattle, not pets</strong>. The node itself is a disposable resource; its &#8220;soul&#8221; lives in <strong>etcd and persistent storage</strong>, not the local OS disk. Restoring node snapshots is not recommended, as it can introduce configuration drift and potentially destabilize the cluster.</p>



<p><strong>Do this instead</strong></p>



<p>Protect what actually matters:</p>



<ul class="wp-block-list">
<li><strong>The cluster state:</strong> Back up your <strong>etcd</strong> database using <strong>Velero</strong> (included with NKP).</li>



<li><strong>The application data:</strong> Use <strong><a href="https://www.nutanix.com/products/data-services-for-kubernetes" rel="noreferrer noopener" target="_blank">Nutanix Data Services for Kubernetes (NDK)</a></strong> to provide enterprise-grade storage-layer replication for your persistent volumes.</li>
</ul>



<p>For your nodes<strong>,</strong> <strong>Nutanix Kubernetes Platform (NKP)</strong> utilizes <strong>Cluster API (CAPI)</strong> to manage the machine lifecycle. Instead of &#8220;restoring&#8221; an old snapshot, NKP uses a declarative manifest to define the desired state. If a node fails, CAPI simply deploys a fresh copy based on your defined OS image template.</p>



<p>The following snippet is an excerpt that describes the topology of an NKP cluster.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">[...]
topology:
  controlPlane:
    replicas: 3
  variables:
    - name: clusterConfig
      value:
        nutanix:
          machineDetails:
            image:
              name: nkp-ubuntu-24.04-release-cis-gpu-1.34.1
            memorySize: 16Gi
            systemDiskSize: 80Gi
            vcpuSockets: 4
[...]</code></pre>



<h2 class="wp-block-heading">Anti-Pattern 2 — Manual SSH for Fixes Instead of Declarative Automation</h2>



<p><strong>The Anti-Pattern:</strong> Logging into nodes to manually &#8220;fix&#8221; configurations or restart runtimes. Manual intervention creates <strong>configuration drift</strong>. Your manual fixes won&#8217;t be applied to new nodes created during a scale-out or upgrade.</p>



<p><strong>What is declarative automation?</strong> In the VM world, we often use <strong>imperative</strong> commands (e.g., <em>&#8220;Step 1: SSH in. Step 2: Update the config. Step 3: Restart.&#8221;</em>). <strong>Declarative automation</strong> flips this: you simply define the <strong>desired state</strong> (e.g., <em>&#8220;This node must have this specific OS version and memory.&#8221;</em>). The system (Kubernetes) then works continuously to ensure the current state always matches that definition. If a node drifts, the system automatically fixes it.</p>



<p><strong>Do this instead</strong></p>



<p>NKP treats nodes as declarative objects. If you need an OS-level change, update the NKP Cluster configuration. <strong>CAPI</strong> ensures every node in the fleet matches that spec automatically.</p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p><strong>Pro tip</strong>: For collecting logs for troubleshooting or opening a support case, use <code class="">nkp diagnose</code>.</p>
</blockquote>



<h2 class="wp-block-heading">Anti-Pattern 3 — CLI Push Instead of GitOps Pull</h2>



<p><strong>The Anti-Pattern:</strong> Using <code class="">kubectl apply -f </code>as your primary deployment method. Manual &#8220;pushes&#8221; create a single point of truth failure and leave no audit trail.</p>



<p><strong>What is GitOps?</strong> Think of GitOps as &#8220;Infrastructure as Code&#8221; but with an autopilot.</p>



<ol class="wp-block-list">
<li><strong>Code:</strong> You store your desired cluster state (YAML files) in a Git repository.</li>



<li><strong>Sync:</strong> A software agent in your cluster (e.g., Flux) continuously compares the <em>cluster&#8217;s actual state </em>with the <em>desired</em> state in Git.</li>



<li><strong>Action:</strong> If there is a difference (e.g., you commit a change or the cluster drifts), the agent automatically updates the cluster to match the Git state. No manual commands required.</li>
</ol>



<p><strong>Do this instead</strong></p>



<p>Version control everything. Let Git be your &#8220;Undo&#8221; button.</p>



<p>In NKP, Flux CD is delivered as a supported, preconfigured component, enabling immediate GitOps adoption without additional platform integration work.</p>



<p>The following YAML is an example of a <code class="">GitRepository</code> resource in NKP that Flux is “watching” and synchronizes the cluster to keep it up to date.</p>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: nkp-demo-catalog-applications
  namespace: kommander
spec:
  interval: 1m0s
  ref:
    branch: main
  timeout: 1m0s
  url: https://github.com/nutanixdev/nkp-demo-catalog-applications</code></pre>



<h2 class="wp-block-heading">Anti-Pattern 4 — Static IPs Instead of Dynamic IPAM</h2>



<p><strong>The Anti-Pattern:</strong> Manually configuring static IP addresses on the node OS. Hardcoding IPs prevents auto-scaling and self-healing.</p>



<p><strong>Why are static IPs a scaling and upgrade bottleneck?</strong> In a virtualized environment, a server might keep its IP for years. In Kubernetes, nodes are ephemeral and frequently cycled. Designing for static IPs introduces three major risks:</p>



<ul class="wp-block-list">
<li><strong>The scaling friction:</strong> Every time you want to scale out, you must manually &#8220;carve out&#8221; IPs from your subnet and hardcode them into your automation. This prevents the <strong>Cluster Autoscaler</strong> from working autonomously.</li>



<li><strong>The upgrade trap:</strong> During a cluster upgrade, NKP uses a <strong>rolling update</strong> strategy. Before CAPI decommissions an old node, it must deploy a brand-new node to prevent a capacity drop. If you are using static IPs, you must have additional addresses pre-reserved and ready for these new nodes, or the upgrade will fail for lack of available IPs.</li>



<li><strong>The blast radius:</strong> If a node fails, its replacement shouldn&#8217;t have to wait for a human to reassign a &#8220;fixed&#8221; address. Static designs turn a sub-second self-healing process into a manual ticket.</li>
</ul>



<p><strong>Do this instead</strong></p>



<p>Only use static IPs for the <strong>control plane VIP</strong> and your <strong>load balancer pool</strong> (managed via <strong>MetalLB</strong>, included in NKP).</p>



<p>Rely on <strong>DHCP</strong> or <strong>IPAM</strong> for your nodes. In NKP, you define an IP pool, and the platform handles the &#8220;plumbing.&#8221;</p>



<h2 class="wp-block-heading">Anti-Pattern 5 — Stretched Clusters Instead of Multi-Cluster DR</h2>



<p><strong>The Anti-Pattern:</strong> Stretching a single cluster across two sites for disaster avoidance. While this works for VMs, it creates a single point of failure (SPOF) in the <strong>etcd quorum</strong>. High latency between sites can take down your entire cluster API.</p>



<p>As a rule of thumb, Kubernetes clusters should remain <strong>single-site control planes</strong>. Site-level resilience should be achieved using <strong>multiple independent clusters and replicated data</strong>, not a stretched etcd quorum.</p>



<p><strong>Do this instead</strong></p>



<p>Detect site failures via <strong>global server load balancing</strong> (GSLB) and reroute traffic to a completely independent, healthy Kubernetes cluster.</p>



<p>While NKP components can be deployed across multiple failure domains, this often creates an SPOF if not well-architected. Instead, use a <strong>multi-cluster, replicated-data</strong> strategy. Deploy independent clusters at sites A and B. Use NDK for synchronous or asynchronous volume replication.</p>



<figure class="wp-block-embed is-type-video is-provider-youtube wp-block-embed-youtube wp-embed-aspect-16-9 wp-has-aspect-ratio"><div class="wp-block-embed__wrapper">

</div></figure>



<h2 class="wp-block-heading">Anti-Pattern 6 — cluster-admin Instead of RBAC</h2>



<p><strong>The Anti-Pattern:</strong> Using the high-privileged <code class="">cluster-admin</code> role for daily operations. An exposed admin token is far more dangerous than a stolen password; it can wipe your entire fleet of clusters in seconds.</p>



<p><strong>Do this instead</strong></p>



<p>Use Kubernetes roles to sandbox developers and automated processes, ensuring they see only what they need.</p>



<p>NKP <strong>Multi-Tenancy</strong> eliminates the complexity of manual Role-Based Access Control (RBAC) by providing a <strong>library of pre-defined common roles</strong>. Instead of writing complex security policies from scratch, you can apply curated permissions to <strong>Workspaces</strong> (for clusters) or <strong>Projects</strong> (for namespaces) directly from the UI.</p>



<p>NKP integrates with your existing identity providers (<strong>Active Directory, Okta, GitHub, …</strong>), allowing you to delegate admin power to specific project teams without ever handing over the &#8220;keys to the kingdom.&#8221;</p>



<h2 class="wp-block-heading">Anti-Pattern 7 — Self-Signed Certs Instead of Automated PKI</h2>



<p><strong>The Anti-Pattern:</strong> Manually generating self-signed certificates or ignoring expiration dates. Internal Kubernetes communication (API to Kubelet) relies on TLS. If a cert expires, the cluster goes &#8220;brain dead.&#8221;</p>



<p><strong>Do this instead</strong></p>



<p>Connect your cluster to a trusted CA (like your internal CA or Let&#8217;s Encrypt).</p>



<p>NKP includes <strong>cert-manager</strong>, the industry standard for certificate lifecycle management. It automatically requests, issues, and renews certificates before they expire.</p>



<h2 class="wp-block-heading">Anti-Pattern 8 — Local Disks Instead of Persistent Storage</h2>



<p><strong>The Anti-Pattern:</strong> Avoiding stateful apps because containers are &#8220;ephemeral&#8221; or writing data to local node disks. If a pod restarts in a new node, local data is lost.</p>



<p><strong>Do this instead</strong></p>



<p>Use <strong>Persistent Volume Claims (PVCs)</strong> backed by a storage backend and let the <code class="">StorageClass</code> handle disk provisioning.</p>



<p>NKP uses a high-performance <strong>CSI Driver</strong> to connect pods to Nutanix AOS storage. With <strong>NDK</strong>, your persistent volumes are not only durable but protected by enterprise-grade snapshots and replication.</p>



<h2 class="wp-block-heading"><strong>Frequently Asked Questions</strong></h2>



<p><strong>When is SSH acceptable?</strong><strong><br /></strong>Limit SSH to exceptional troubleshooting scenarios. Routine changes should be declarative.</p>



<p><strong>Does NKP include GitOps?</strong><strong><br /></strong>Yes, Flux CD is integrated and ready to use.</p>



<p><strong>Does NKP include DR and backups?</strong><strong><br /></strong>Yes, Velero for cluster state and NDK for persistent volumes.</p>



<p><strong>Do I need cluster-admin?</strong><strong><br /></strong>No, NKP provides RBAC roles integrated with enterprise identity providers.</p>



<p><strong>Can NKP run stateful apps?</strong><strong><br /></strong>Yes, with CSI and NDK providing durable storage.</p>



<p><strong>How are node failures handled?</strong><strong><br /></strong>Failed nodes are automatically recreated from the cluster template by Cluster API.</p>



<h2 class="wp-block-heading">Next Steps</h2>



<p>Ready to evaluate how an integrated Kubernetes platform simplifies Day-2 operations?</p>



<ul class="wp-block-list">
<li>Read: <a href="https://www.nutanix.com/library/ebooks/cloud-native-playbook-for-platform-engineers" rel="noreferrer noopener" target="_blank"><strong>A Cloud Native Playbook for Platform Engineers</strong></a></li>



<li>Try: <a href="https://cloud.nutanixtestdrive.com/login?source=one-platform&amp;type=nkp" rel="noreferrer noopener" target="_blank"><strong>Start a Nutanix Kubernetes Platform Test Drive</strong></a></li>



<li>Learn: <a href="https://www.youtube.com/watch?v=jT6N2c32ETE" rel="noreferrer noopener" target="_blank"><strong>Kubernetes Lifecycle Management with NKP and Cluster API</strong></a></li>
</ul>
