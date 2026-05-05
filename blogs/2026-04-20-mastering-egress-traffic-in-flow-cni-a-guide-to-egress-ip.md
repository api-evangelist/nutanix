---
title: "Mastering Egress Traffic in Flow CNI: A Guide to Egress IP"
url: "https://www.nutanix.dev/2026/04/20/mastering-egress-traffic-in-flow-cni-a-guide-to-egress-ip/"
date: "Mon, 20 Apr 2026 14:00:00 +0000"
author: "Chris Rasmussen"
feed_url: "https://www.nutanix.dev/feed/"
---
<p>Managing outbound network traffic from your Kubernetes clusters is just as critical as handling inbound connections. Whether you need to ensure your pods communicate with external services using a consistent, predictable IP address, or you&#8217;re building robust multi-tenant environments, Nutanix <a href="https://www.nutanix.com/tech-center/blog/introducing-flow-cni" rel="noreferrer noopener" target="_blank">Flow CNI</a> has you covered!</p>



<p>In this post, we&#8217;ll dive into the details of configuring <strong>Egress IP</strong> on <a href="https://www.nutanix.com/products/kubernetes-management-platform" rel="noreferrer noopener" target="_blank">Nutanix Kubernetes Platform</a> (NKP) with Flow CNI to give you ultimate control over Kubernetes cluster&#8217;s outbound traffic.</p>



<h2 class="wp-block-heading">Why use Egress?</h2>



<p>By default, outbound traffic uses Source NAT (SNAT) to mask a pod’s internal IP with the host node’s IP. If a pod moves to a different node, its source IP changes. While this works for basic scenarios, many enterprise use cases require specific external IP addresses for whitelisting on external firewalls, databases, or third-party APIs.</p>



<p>This is where Flow CNI&#8217;s Egress capabilities come into play, offering two powerful approaches:</p>



<ol class="wp-block-list">
<li>Egress IP: Assigns specific IP addresses to outbound traffic from a targeted group of pods or namespaces.</li>



<li>Egress Service: Leverages LoadBalancer services (like MetalLB) to provide highly available egress paths.</li>
</ol>



<p>Let&#8217;s look at how to implement Egress IP. We will discuss implementing Egress Service in another blog.</p>



<h2 class="wp-block-heading">Why Flow CNI?</h2>



<p>In Kubernetes, a Container Network Interface (CNI) plugin is what enables pods to communicate over the network. Flow CNI is Nutanix’s CNI implementation that allows Kubernetes clusters to participate in Flow Virtual Networking, meaning Kubernetes pods can exist within the same VPC networking context as VMs.</p>



<p>With Flow CNI:</p>



<ul class="wp-block-list">
<li>Kubernetes pods receive network connectivity that’s consistent with other VPC subnets.</li>



<li>Pod traffic can route seamlessly to VMs within the same VPC and other pods across different clusters managed by Prism Central.</li>



<li>Standard Kubernetes networking models extend into the Nutanix SDN fabric.</li>
</ul>



<h2 class="wp-block-heading">Pre-Requisites</h2>



<ul class="wp-block-list">
<li>Minimum Required Versions</li>
</ul>



<figure class="wp-block-table"><table class="has-fixed-layout"><tbody><tr><td>AOS version</td><td>7.5</td></tr><tr><td>PC version (X-Large)</td><td>7.5</td></tr><tr><td>Network Controller version</td><td>7.0.0</td></tr><tr><td>Flow CNI version</td><td>1.0.0</td></tr><tr><td>NKP version</td><td>2.17.1</td></tr></tbody></table></figure>



<ul class="wp-block-list">
<li>This article assumes that
<ul class="wp-block-list">
<li>An NKP cluster is <a href="https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide-v7_0_0%3Aear-flow-cni-prepare-kubernetes-cluster-nkp-pc-t.html" rel="noreferrer noopener" target="_blank">created with FlowCNI installed</a> on the workload cluster.</li>



<li>Tcpdump is installed on the egress node for verification.</li>



<li><a href="https://github.com/ovn-kubernetes/ovn-kubernetes/blob/de6b3bb229031840d2870945c800c6f492df7c6d/dist/templates/k8s.ovn.org_egressips.yaml.j2" rel="noreferrer noopener" target="_blank">Egress IP CRD</a>’s (Custom Resource Definitions) are installed.</li>
</ul>
</li>
</ul>



<h3 class="wp-block-heading">Assigning labels to Egress Nodes</h3>



<ul class="wp-block-list">
<li>While you <em>can</em> apply these directly using kubectl label node &#8230;, <strong>this is not recommended for production!</strong> If a node dies and is replaced by the cluster autoscaler, the manual labels will be lost.
<ul class="wp-block-list">
<li><strong>The Best Practice:</strong> Apply these labels directly to your NKP MachineDeployment!
<ul class="wp-block-list">
<li>You can find your machine deployments by running:</li>
</ul>
</li>
</ul>
</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">kubectl --kubeconfig &lt;nkp-mgmt-cluster.conf&gt; get machinedeployment -A</code></pre>



<ul class="wp-block-list">
<li>Then, edit the specific deployment to inject the labels under the template.metadata.labels hierarchy:</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash"># Output from: kubectl edit machinedeployment ...
...
    template:
      metadata:
        labels:
          cluster.x-k8s.io/cluster-name: nkpworkloadcluster2
          topology.cluster.x-k8s.io/deployment-name: md-0
          # Add your egress labels here!
          node.cluster.x-k8s.io/egress-assignable: &quot;&quot;
...</code></pre>



<ul class="wp-block-list">
<li>By placing the labels here, NKP ensures that any new nodes spun up by this MachineDeployment will automatically inherit the necessary egress configurations.</li>
</ul>



<h2 class="wp-block-heading">What is Egress IP?</h2>



<p>The Egress IP feature enables a cluster administrator to ensure that the traffic from one or more pods in one or more namespaces has a consistent source IP address for services outside the cluster network.</p>



<p><strong><em>Note- </em></strong>East-West traffic (including pod -&gt; node IP) is excluded from Egress IP.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img alt="" class="wp-image-69744" height="564" src="https://www.nutanix.dev/wp-content/uploads/2026/04/image-4-1024x564.png" width="1024" /></figure>
</div>


<h3 class="wp-block-heading">Creating the Egress IP Resource</h3>



<ul class="wp-block-list">
<li>For Egress IP, the underlying worker nodes need specific labels so OVN-Kubernetes knows where to route the traffic (<a href="http://k8s.ovn.org/egress-assignable=">k8s.ovn.org/egress-assignable=</a>&#8220;&#8221;).<br /><strong><em>Note- </em></strong><em>This label is automatically propagated by the virtue of labelling machine deployment.</em>
<ul class="wp-block-list">
<li>Using the below Egress IP yaml, it will create egress ip on NKP nodes.</li>
</ul>
</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">apiVersion: k8s.ovn.org/v1
kind: EgressIP
metadata:
  name: egress-ip-vpc1
spec:
  egressIPs:
    - 10.x.x.99
    - 10.x.x.180
  namespaceSelector:
    matchLabels:
      kubernetes.io/metadata.name: vpc-egress
  podSelector:
    matchLabels:
      app: egress-label-vpc1</code></pre>



<h3 class="wp-block-heading">Validating the status of the Nodes</h3>



<ul class="wp-block-list">
<li>Ensure that all the nodes are up and running</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">#kubectl get nodes -o wide
NAME                                STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION                 CONTAINER-RUNTIME
nkp-wrkld2-cj6xz-4flq8              Ready    control-plane   14d   v1.34.3   10.x.x.83   &lt;none&gt;        Rocky Linux 9.6 (Blue Onyx)   5.14.0-570.18.1.el9_6.x86_64   containerd://1.7.29-d2iq.1
nkp-wrkld2-cj6xz-zsjhv              Ready    &lt;none&gt;          14d   v1.34.3   10.x.x.71   &lt;none&gt;        Rocky Linux 9.6 (Blue Onyx)   5.14.0-570.18.1.el9_6.x86_64   containerd://1.7.29-d2iq.1
nkp-wrkld2-md-0-fj7kp-k68zs-bpx7s   Ready    &lt;none&gt;          14d   v1.34.3   10.x.x.81   &lt;none&gt;        Rocky Linux 9.6 (Blue Onyx)   5.14.0-570.18.1.el9_6.x86_64   containerd://1.7.29-d2iq.1</code></pre>



<h2 class="wp-block-heading">Testing Egress IP</h2>



<ul class="wp-block-list">
<li>Create a VPC with a VM/endpoint that is hosting the application service that is outside of this VPC.</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">root@localhost:~# kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
                            &quot;--service-cluster-ip-range=10.x.x.x/12&quot;,

root@localhost:~# kubectl cluster-info dump | grep -m 1 cluster-cidr
                            &quot;--cluster-cidr=192.168.0.0/16&quot;,

root@localhost:~#kubectl get nodes -o jsonpath=&#039;{.items[*].spec.podCIDR}{&quot;\\n&quot;}&#039;
192.168.5.0/24 192.168.3.0/24 192.168.1.0/24

#&lt;atlas&gt; network.list
Virtual network name  Virtual network UUID

vpc-egress            7a3be63b-##########################
vpc-egress2           ###################################

&lt;atlas&gt; network.get 7a3be63b-##########################
vpc-egress {
  federation_uuid: &quot;###################################&quot;
  kubernetes_cluster_list {
    cluster_uuid: &quot;###################################&quot;
    gateway_nodes_selector {
    }
    namespace_selector {
      match_label_list {
        key: &quot;vpc&quot;
        value: &quot;vpc-egress&quot;
      }
    }
    pod_cidr: &quot;30.0.0.0/16/24&quot;
    transit_ip_address: &quot;169.254.3.2&quot;
  }
  logical_timestamp: 0
  mac_binding_aging_threshold: &quot;14400&quot;
  name: &quot;vpc-egress&quot;
  project_id: &quot;00000000-0000-0000-0000-000000000000&quot;
  supported_multiple_external_subnet_type: &quot;kOnlyNoNat&quot;
  transit_ip_address: &quot;169.254.3.1&quot;
  uuid: &quot;###################################&quot;
  vpc_scope: &quot;kVMsAndContainers&quot;
  vpc_type: &quot;kRegular&quot;
}</code></pre>



<ul class="wp-block-list">
<li>Create a pod and pass the same label “egress-label-vpc1” at the time of pod creation</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pod-ubuntu
  namespace: vpc-egress
spec:
  serviceName: pod-ubuntu
  replicas: 1
  selector:
    matchLabels:
      app: egress-label-vpc1
  template:
    metadata:
      labels:
        app: egress-label-vpc1
    spec:
      containers:
      - name: ssh-client
        image: ubuntu:20.04
        command:
          - /bin/bash
          - -c
        args:
          - |
            apt-get update &amp;&amp; \
            apt-get install -y sshpass openssh-client iproute2 net-tools &amp;&amp; \
            tail -f /dev/null
</code></pre>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">#kubectl get pods -o wide -n vpc-egress | grep pod-ubuntu
pod-ubuntu-0                 1/1     Running   0               43h   30.0.0.3   nkp-wrkld2-cj6xz-zsjhv              &lt;none&gt;           &lt;none&gt;</code></pre>



<ul class="wp-block-list">
<li>Validate that the egress node is set for “egress-assignable”</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">root@localhost:~# kubectl describe node nkp-wrkld2-cj6xz-4flq8 | grep egress
                    k8s.ovn.org/egress-assignable=
                    k8s.ovn.org/bridge-egress-ips: [&quot;10.x.x.99&quot;]</code></pre>



<ul class="wp-block-list">
<li>Send traffic from the client pod towards an endpoint that resides outside the cluster and you will notice that the source IP address would be consistent and be the Egress Nodes IP address that we have configured above ending with 10.x.x.99.
<ul class="wp-block-list">
<li>In this case 1.1.1.60 is an endpoint that resides outside the Kubernetes cluster.</li>



<li>Traffic originating from pod.</li>
</ul>
</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">#root@pod-ubuntu-0:/# ip a
1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if18: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1400 qdisc noqueue state UP group default
    link/ether 0a:58:1e:00:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 30.0.0.3/24 brd 30.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::858:1eff:fe00:3/64 scope link
       valid_lft forever preferred_lft forever


#root@pod-ubuntu-0:/# ping 1.1.1.60
PING 1.1.1.60 (1.1.1.60) 56(84) bytes of data.
64 bytes from 1.1.1.60: icmp_seq=1 ttl=51 time=22.1 ms
64 bytes from 1.1.1.60: icmp_seq=2 ttl=51 time=17.0 ms
64 bytes from 1.1.1.60: icmp_seq=3 ttl=51 time=11.7 ms
64 bytes from 1.1.1.60: icmp_seq=4 ttl=51 time=11.8 ms
64 bytes from 1.1.1.60: icmp_seq=5 ttl=51 time=11.5 ms
64 bytes from 1.1.1.60: icmp_seq=6 ttl=51 time=11.7 ms
^C
--- 1.1.1.60 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5008ms
rtt min/avg/max/mdev = 11.517/14.288/22.094/3.989 ms


#root@pod-ubuntu-0:/# curl ifconfig.me
192.x.x.98</code></pre>



<ul class="wp-block-list">
<li>Traffic captured on the egress node shows the source IP address as the egress node IP.</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">[root@nkp-wrkld2-cj6xz-4flq8 /]# tcpdump -nnnvi eth0 icmp
dropped privs to tcpdump
tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
08:07:44.114688 IP (tos 0x0, ttl 62, id 62649, offset 0, flags [DF], proto ICMP (1), length 84)
    10.x.x.99 &gt; 1.1.1.60: ICMP echo request, id 58017, seq 1, length 64
08:07:44.126593 IP (tos 0x0, ttl 53, id 21082, offset 0, flags [none], proto ICMP (1), length 84)
    1.1.1.60 &gt; 10.x.x.99: ICMP echo reply, id 58017, seq 1, length 64
08:07:45.116205 IP (tos 0x0, ttl 62, id 62953, offset 0, flags [DF], proto ICMP (1), length 84)
    
[root@nkp-wrkld2-cj6xz-4flq8 /]# tcpdump -nnnvi eth0 port 80

08:12:30.138175 IP (tos 0x0, ttl 62, id 35342, offset 0, flags [DF], proto TCP (6), length 127)
    10.x.x.99.45185 &gt; 34.160.111.145.80: Flags [P.], cksum 0xfbfa (correct), seq 1:76, ack 1, win 32, options [nop,nop,TS val 4131398484 ecr 2298361677], length 75: HTTP, length: 75
	GET / HTTP/1.1
	Host: ifconfig.me
	User-Agent: curl/7.68.0
	Accept: */*

08:12:30.148409 IP (tos 0x0, ttl 121, id 44024, offset 0, flags [none], proto TCP (6), length 52)
    34.160.111.145.80 &gt; 10.x.x.99.45185: Flags [.], cksum 0x64f6 (correct), ack 76, win 1050, options [nop,nop,TS val 2298361693 ecr 4131398484], length 0
08:12:30.189158 IP (tos 0x0, ttl 121, id 44025, offset 0, flags [none], proto TCP (6), length 217)
    34.160.111.145.80 &gt; 10.x.x.99.45185: Flags [P.], cksum 0x055b (correct), seq 1:166, ack 76, win 1050, options [nop,nop,TS val 2298361734 ecr 4131398484], length 165: HTTP, length: 165
	HTTP/1.1 200 OK
	Content-Length: 14
	access-control-allow-origin: *
	content-type: text/plain
	date: Wed, 01 Apr 2026 08:12:30 GMT
	via: 1.1 google</code></pre>



<ul class="wp-block-list">
<li>Digging further we can see that nat(src=10.x.x.99) is the action which will do src nat to egress ip based on pkt_mark.</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">root@nkp-wrkld2-cj6xz-4flq8:/# ovs-ofctl dump-flows breth0 | grep 10.x.x.99
 cookie=0xdeff105, duration=161371.465s, table=0, n_packets=21311, n_bytes=1559651, idle_age=249, hard_age=65534, priority=105,pkt_mark=0xc350,ip,in_port=2,dl_src=50:6b:8d:de:2d:a9 actions=ct(commit,zone=64000,nat(src=10.x.x.99),exec(load:0x4-&gt;NXM_NX_CT_MARK[])),output:1
 cookie=0xdeff105, duration=161371.465s, table=0, n_packets=0, n_bytes=0, idle_age=65534, hard_age=65534, priority=105,pkt_mark=0xc350,ip,in_port=3,dl_src=50:6b:8d:de:2d:a9 actions=ct(commit,zone=64000,nat(src=10.x.x.99),exec(load:0x5-&gt;NXM_NX_CT_MARK[])),output:1</code></pre>



<h2 class="wp-block-heading">How is the traffic encapsulated from the worker node to the egress node?&nbsp;</h2>



<ul class="wp-block-list">
<li>The traffic is encapsulated using GENEVE from the worker node to the egress node.</li>



<li>10.x.x.71 is the worker node on which the test pod is running whereas 10.x.x.83 is the egress node.</li>
</ul>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">#kubectl get nodes -o wide
NAME                                STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION                 CONTAINER-RUNTIME
nkp-wrkld2-cj6xz-4flq8              Ready    control-plane   14d   v1.34.3   10.x.x.83   &lt;none&gt;        Rocky Linux 9.6 (Blue Onyx)   5.14.0-570.18.1.el9_6.x86_64   containerd://1.7.29-d2iq.1
nkp-wrkld2-cj6xz-zsjhv              Ready    &lt;none&gt;          14d   v1.34.3   10.x.x.71   &lt;none&gt;        Rocky Linux 9.6 (Blue Onyx)   5.14.0-570.18.1.el9_6.x86_64   containerd://1.7.29-d2iq.1
nkp-wrkld2-md-0-fj7kp-k68zs-bpx7s   Ready    &lt;none&gt;          14d   v1.34.3   10.x.x.81   &lt;none&gt;        Rocky Linux 9.6 (Blue Onyx)   5.14.0-570.18.1.el9_6.x86_64   containerd://1.7.29-d2iq.1</code></pre>



<ul class="wp-block-list">
<li>Here below we can see GENEVE encapsulation being used for the communication amongst the worker and egress node with traffic being sent from the test pod towards the service external to the cluster.</li>
</ul>



<p><strong><em>Note</em></strong><em>&#8211; UDP port 6081 is the </em><em>default port used for Geneve (Generic Network Virtualization Encapsulation) tunnel traffic</em><em>, enabling communication between pods on different nodes.</em></p>



<pre class="wp-block-prismatic-blocks"><code class="language-bash">[root@nkp-wrkld2-cj6xz-4flq8 /]# tcpdump -lnnvvei breth0 udp and port 6081 | grep 1.1.1.60 -B 3
dropped privs to tcpdump
tcpdump: listening on breth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes

10:19:11.715637 50:6b:8d:b2:65:f9 &gt; 50:6b:8d:de:2d:a9, ethertype IPv4 (0x0800), length 156: (tos 0x0, ttl 64, id 7566, offset 0, flags [DF], proto UDP (17), length 142)
    10.x.x.71.17684 &gt; 10.x.x.83.6081: [bad udp cksum 0x554b -&gt; 0x7d88!] Geneve, Flags [C], vni 0x2, proto TEB (0x6558), options [class Open Virtual Networking (OVN) (0x102) type 0x80(C) len 8 data 00010006]
	0a:58:64:41:00:01 &gt; 0a:58:64:41:00:06, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 63, id 31954, offset 0, flags [DF], proto ICMP (1), length 84)
    30.0.0.3 &gt; 1.1.1.60: ICMP echo request, id 9, seq 31, length 64
10:19:11.727740 50:6b:8d:de:2d:a9 &gt; 50:6b:8d:b2:65:f9, ethertype IPv4 (0x0800), length 156: (tos 0x0, ttl 64, id 11716, offset 0, flags [DF], proto UDP (17), length 142)
    10.x.x.83.58232 &gt; 10.x.x.71.6081: [bad udp cksum 0x554b -&gt; 0x6ca5!] Geneve, Flags [C], vni 0x1, proto TEB (0x6558), options [class Open Virtual Networking (OVN) (0x102) type 0x80(C) len 8 data 0001000a]
	0a:58:1e:00:00:01 &gt; 0a:58:1e:00:00:03, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 51, id 48813, offset 0, flags [none], proto ICMP (1), length 84)
    1.1.1.60 &gt; 30.0.0.3: ICMP echo reply, id 9, seq 31, length 64</code></pre>



<h2 class="wp-block-heading">Conclusion</h2>



<p>Flow CNI and OVN-Kubernetes make it incredibly simple to implement advanced traffic routing scenarios. With Egress IP’s you can ensure your Kubernetes workloads integrate seamlessly and securely with your broader enterprise network architecture.</p>



<h2 class="wp-block-heading">References</h2>



<ul class="wp-block-list">
<li><a href="https://www.nutanix.com/tech-center/blog/introducing-flow-cni" rel="noreferrer noopener" target="_blank">Flow CNI</a></li>



<li><a href="https://ovn-kubernetes.io/features/cluster-egress-controls/egress-ip/" rel="noreferrer noopener" target="_blank">Egress IP in OVN-Kubernetes</a></li>



<li><a href="https://ovn-kubernetes.io/api-reference/egress-ip-api-spec/" rel="noreferrer noopener" target="_blank">Egress IP API Spec</a></li>



<li><a href="https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide-v7_0_0:ear-flow-cni-prerequisites-pc-r.html" rel="noreferrer noopener" target="_blank">Try out Flow CNI</a></li>
</ul>



<p></p>
