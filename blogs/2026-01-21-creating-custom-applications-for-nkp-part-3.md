---
title: "Creating Custom Applications for NKP – Part 3"
url: "https://www.nutanix.dev/2026/01/21/creating-custom-applications-for-nkp-part-3/"
date: "Wed, 21 Jan 2026 15:00:00 +0000"
author: "Mark Dastmalchi-Round"
feed_url: "https://www.nutanix.dev/feed/"
---
<p>Previously in this series, we <a href="https://www.nutanix.dev/2026/01/07/creating-custom-applications-for-nkp-part-1/" rel="noreferrer noopener" target="_blank">built a sample application</a> and <a href="https://www.nutanix.dev/2026/01/14/creating-custom-applications-for-nkp-part-2/" rel="noreferrer noopener" target="_blank">packaged it for deployment</a> across Kubernetes clusters. We integrated it into the NKP application catalog so it could be deployed and managed the same as any other platform application. In this article, we&#8217;ll wrap things up by looking at configuration management and upgrades, rolling out an update with a new feature along the way.</p>



<h2 class="wp-block-heading">Configure the Application</h2>



<p>As we saw in <a href="https://www.nutanix.dev/2026/01/07/creating-custom-applications-for-nkp-part-1/" rel="noreferrer noopener" target="_blank">part 1</a>, it’s possible to set Helm values to customize the application configuration.&nbsp;</p>



<p>As Helmify created a few values and parameters for us automatically, we can make use of them and use NKP to ensure consistent configuration across all your clusters.&nbsp;</p>



<p>For example, to again scale our deployment up to 3 replicas, return to the workspace applications view, and select “Edit” for the Hello World application:</p>



<p>Type the following content into the code editor:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">hello:
  replicas: 3 </code></pre>



<p>This is the same format as the <code class="">values.yaml</code> file we looked at when creating the Helm chart. Your screen should now look like this:</p>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-69477" height="302" src="https://www.nutanix.dev/wp-content/uploads/2026/01/image-8-1024x302.png" width="1024" /></figure>



<p>You should then click the blue “Save” button at the bottom of the screen.&nbsp;</p>



<p>Soon, you’ll see the configuration deployed, the <code class="">HelmRelease</code> move into an “upgraded” state, and the deployment will scale up to have 3 pods running:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl get pods -A -l app=hello

NAMESPACE                     NAME                                  READY   STATUS    RESTARTS   AGE
kommander-default-workspace   hello-world-hello-nkp-hello-bd6njgd   1/1     Running   0          7m21s
kommander-default-workspace   hello-world-hello-nkp-hello-bd75nm6   1/1     Running   0          17m
kommander-default-workspace   hello-world-hello-nkp-hello-bd8thz5   1/1     Running   0          7m21s</code></pre>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>Tip &#8211; You can also override any of the default values from the chart’s values.yaml file here &#8211; take a look at the file and see what else you can configure!</p>
</blockquote>



<h2 class="wp-block-heading">A Look Behind the Scenes</h2>



<p>So far we&#8217;ve been using the browser UI and CLI tooling, but here&#8217;s another area that NKP really shines &#8211; everything can be managed as infrastructure-as-code. Regardless of the frontend you are using, all this is represented using standard Kubernetes resources like Custom Resource Definitions (CRDs) and ConfigMaps.&nbsp;&nbsp;</p>



<p>For example, if you look at your Management Cluster, you’ll see there is a ConfigMap created in your chosen workspace namespace named <code class="">hello-world-config-overrides</code>. We can examine this and see that the contents are the configuration values for our application we entered above. For example:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl -n kommander-default-workspace \
  get configmap hello-world-config-overrides \
  -o jsonpath=&#039;{ .data }&#039;</code></pre>



<p>Which shows:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-json">{&quot;values.yaml&quot;:&quot;hello:\n  replicas: 3 &quot;}</code></pre>



<p>The application is deployed by an AppDeployment CRD in the same namespace called <code class="">hello-world</code>.If you examine this, you’ll see a simple format that defines which application should be deployed to which clusters, and also references the above ConfigMap as a source of configuration. It’ll look something like this:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">apiVersion: apps.kommander.d2iq.io/v1alpha3
kind: AppDeployment
metadata:
  finalizers:
  - kommander.mesosphere.io/appdeployment
  name: hello-world
  namespace: kommander-default-workspace
spec:
  appRef:
    kind: App
    name: hello-world-0.1.0
  clusterSelector:
    matchExpressions:
    - key: kommander.d2iq.io/cluster-name
      operator: In
      values:
      - workload-1
  configOverrides:
    name: hello-world-config-overrides</code></pre>



<p>You can see it contains:</p>



<ul class="wp-block-list">
<li>The <code class="">appRef</code> which specifies <em>what</em> to deploy</li>



<li>The <code class="">clusterSelector</code> which selects <em>where</em> to deploy it</li>



<li>And the <code class="">configOverrides</code> which govern <em>how</em> it is configured.&nbsp;</li>
</ul>



<p>Simple! If you want to configure your application deployments and configuration in a declarative fashion, you’d simply create the Kubernetes resources as above. This makes NKP easy to manage at scale, because you&#8217;re working directly with the Kubernetes API and there are no proprietary interfaces or special protocols to learn.&nbsp;</p>



<h2 class="wp-block-heading">Updating the Application</h2>



<p>You may have noticed that a lot of the NKP platform applications have their own dashboards and links to them from the cluster Applications tab.</p>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-69480" height="447" src="https://www.nutanix.dev/wp-content/uploads/2026/01/image-9-1024x447.png" width="1024" /></figure>



<p>Wouldn’t it be nice if we could add one for our application? Well, we can &#8211; let’s do this and roll out a new version of our application (0.2.0) while we’re at it.</p>



<h2 class="wp-block-heading">Adding the Dashboard</h2>



<p>As with most things in NKP, customizing or automating the platform is as easy as creating a Kubernetes object. In this case, we can use a simple ConfigMap object with appropriate annotations to create a dashboard link for our application. We can bundle this into our Helm chart so it gets deployed along with the rest of our application manifests.</p>



<p>Create a new file at <code class="">charts/hello-nkp/templates/hello-dashboard.yaml</code>, and copy the following content into it:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">---
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world-dashboard-info
  labels:
    kommander.d2iq.io/application: hello-world
data:
  dashboardLink: &quot;/hello&quot;
  name: Hello World
  version: 0.2.0</code></pre>



<h2 class="wp-block-heading">Creating a new Helm Chart</h2>



<p>Change the <code class="">version</code> and <code class="">appVersion</code> metadata parameters to read <code class="">0.2.0</code> in <code class="">charts/hello-nkp/Chart.yaml</code>. Then you can package and push again:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">helm package ./charts/hello-nkp
helm push ./hello-nkp-0.2.0.tgz oci://registry.example.com/helm</code></pre>



<h2 class="wp-block-heading">Updating the Application Catalog</h2>



<p>Update the OCIRepository resource in applications/hello-world/0.1.0/helmrelease/helmrelease.yaml file to use the new package tag 0.2.0:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-yaml">apiVersion: source.toolkit.fluxcd.io/v1
kind: OCIRepository
metadata:
  name: ${releaseName}-chart-source
  namespace: ${releaseNamespace}
spec:
  interval: 6h0m0s
  ref:
    tag: 0.2.0
  url: oci://registry.example.com/helm/hello-nkp</code></pre>



<p>And finally rename the directory to &#8220;bump&#8221; the application version:</p>



<pre class="wp-block-prismatic-blocks"><code class="">mv applications/hello-world/0.1.0 applications/hello-world/0.2.0</code></pre>



<p>Now we can bundle the catalog repository and push the OCI package again:</p>



<pre class="wp-block-prismatic-blocks"><code class="">nkp validate catalog-repository
nkp create catalog-bundle --output-file hello-world-0.2.0.tar
nkp push bundle --bundle hello-world-0.2.0.tar --to-registry registry.example.com/library</code></pre>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>Note &#8211; you can have multiple applications, and multiple versions of an application stored within a catalog bundle. You can even create an air-gapped distribution which includes container images and OCI artifacts for distribution to environments without Internet access! For more information, see the built in CLI help or review the NKP documentation.</p>
</blockquote>



<h2 class="wp-block-heading">Upgrading the Application</h2>



<p>Now we update the application catalog to point to version 0.2.0. You will need to target the Management Cluster kubectl context again, and run the following:</p>



<pre class="wp-block-prismatic-blocks"><code class="language-shell-session">kubectl patch \
  --type merge \
  -n kommander \
  ocirepository nkp-hello-world-hello-world \
  --patch &#039;{&quot;spec&quot;: {&quot;ref&quot;:{&quot;tag&quot;:&quot;0.2.0&quot;}}}&#039;</code></pre>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
<p>Note that the NKP CLI includes commands to work with these objects (such as <code class="">nkp edit ocirepository</code>), but I’m using a quick patch operation in these examples to make the instructions more succinct.</p>
</blockquote>



<p>Finally we upgrade the AppDeployment object across our chosen workspace:</p>



<pre class="wp-block-prismatic-blocks"><code class="">kubectl patch \
  --type merge \
  -n kommander-default-workspace \
  AppDeployment hello-world \
  --patch &#039;{&quot;spec&quot;:{&quot;appRef&quot;:{&quot;name&quot;:&quot;hello-world-0.2.0&quot;}}}&#039;</code></pre>



<p>This will trigger a Helm upgrade of our application across our selected workspace clusters. After a few moments, you’ll see our updated application complete with dashboard link appearing in the cluster’s Enabled Applications tab:</p>



<figure class="wp-block-image size-large"><img alt="" class="wp-image-69482" height="515" src="https://www.nutanix.dev/wp-content/uploads/2026/01/image-10-1024x515.png" width="1024" /></figure>



<h2 class="wp-block-heading">Conclusion</h2>



<p>And there we have it! We’ve taken a simple “Hello, World” application and transformed it into a fully-fledged NKP platform application.</p>



<p>Let’s recap what we’ve achieved:</p>



<ul class="wp-block-list">
<li>Created Kubernetes manifests and converted them into a Helm chart</li>



<li>Packaged and pushed our chart to an OCI registry</li>



<li>Built an NKP application catalog with proper metadata and versioning</li>



<li>Deployed our application across multiple clusters through the NKP interface</li>



<li>Added a custom dashboard link, just like the built-in platform applications</li>



<li>Demonstrated how to upgrade and manage versions seamlessly</li>
</ul>



<p>The real power here is that your custom applications now behave exactly like NKP’s built-in platform applications. They appear in the same catalog, use the same deployment mechanisms, and can be managed with the same consistent workflow across your entire Kubernetes estate &#8211; whether that’s on-prem, in the cloud, virtualized, or bare-metal.</p>



<p>By leveraging NKP’s application framework, you’re ensuring your platform services can be consistently configured, upgraded, and managed at scale. And because it’s all built on standard Kubernetes primitives and GitOps principles, you’re never locked in. Your platform, your applications, your way!</p>



<h2 class="wp-block-heading">Other Resources</h2>



<p>The following is a list of useful documentation and resources used while creating this article.</p>



<ul class="wp-block-list">
<li>NKP Documentation
<ul class="wp-block-list">
<li>Custom Application documentation: <a href="https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-custom-apps-c.html" rel="noreferrer noopener" target="_blank">https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-custom-apps-c.html</a> </li>



<li>Creating Catalog Bundles and Collections: <a href="https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-create-a-git-repo-t.html" rel="noreferrer noopener" target="_blank">https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-create-a-git-repo-t.html</a> </li>



<li>Application metadata spec: <a href="https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-workspace-app-metadata-c.html" rel="noreferrer noopener" target="_blank">https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-workspace-app-metadata-c.html</a> </li>



<li>Dashboard card spec: <a href="https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-custom-cluster-app-dashboard-cards-r.html" rel="noreferrer noopener" target="_blank">https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_16:top-custom-cluster-app-dashboard-cards-r.html</a> </li>
</ul>
</li>



<li>Third-party documentation
<ul class="wp-block-list">
<li>FluxCD Helm CRDs: <a href="https://fluxcd.io/flux/components/helm/helmreleases/" rel="noreferrer noopener" target="_blank">https://fluxcd.io/flux/components/helm/helmreleases/</a> </li>



<li>Helm Chart documentation: <a href="https://helm.sh/docs/topics/charts/" rel="noreferrer noopener" target="_blank">https://helm.sh/docs/topics/charts/</a> </li>
</ul>
</li>
</ul>



<p></p>
