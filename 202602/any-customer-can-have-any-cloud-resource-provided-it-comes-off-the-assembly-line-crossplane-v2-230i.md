---
Title: Any Customer Can Have Any Cloud Resource, Provided It Comes Off the Assembly Line: Crossplane v2 and the SIPOC Factory Floor
Description: 
Author: Willem van Heemstra
Date: 2026-02-22T21:19:38.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>How Crossplane v2 orchestrates your cloud infrastructure like Henry Ford orchestrated the Model T — with stunning efficiency, total standardisation, and a philosophy that any customer can have any colour, as long as it’s black. Or in our case: any resource, as long as it’s defined in a Composition.</em></p>

<h2>
  
  
  Introduction
</h2>

<p>It’s 1913. Highland Park, Michigan. Henry Ford is pacing the factory floor, twirling his moustache, and muttering something darkly about chassis frames not moving fast enough. Then, in a moment of industrial genius that would define the twentieth century, he invents the moving assembly line. Workers stay put. The work comes to them. Every step is standardised. Every output is predictable. Every Model T rolls off the end looking exactly like the last one.</p>

<p>The output was so standardised, in fact, that Ford famously told customers they could have any colour they liked — as long as it was black. Not because he lacked imagination. Because standardisation was <em>the point</em>.</p>

<p>Fast-forward 111 years. You’re on a platform engineering team. Application developers are pinging you on Slack at 2pm (and 2am, and on a Friday afternoon when you were <em>this close</em> to starting the weekend). “Can I have an Azure Resource Group?” “Can I have an AKS cluster?” “Can I have an Azure Subscription?” You’re the factory. They’re the customers. And right now, your production line is a shared Google Doc, a Jira ticket nobody reads, and a mild sense of existential dread.</p>

<p>Enter <strong>Crossplane v2</strong>.</p>

<p>And Henry Ford, back from the dead and extremely opinionated about YAML.</p>




<h2>
  
  
  The SIPOC Mental Model (Straight Off the Factory Floor)
</h2>

<p>Before we look at a single line of YAML, let’s borrow a concept from the world of Lean process improvement: <strong>SIPOC</strong>.</p>

<p><strong>S</strong>upplier → <strong>I</strong>nput → <strong>P</strong>rocess → <strong>O</strong>utput → <strong>C</strong>onsumer</p>

<p>Ford used this pattern at Highland Park — he just didn’t call it that. He was too busy being revolutionary and paying his workers five dollars a day, which was either enlightened capitalism or a clever scheme to ensure they could afford to buy the cars they built. Possibly both.</p>

<p>Here’s how his factory maps to Crossplane v2:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>SIPOC Stage</th>
<th>Ford’s Factory (1913)</th>
<th>Crossplane v2 (Now)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Supplier</strong></td>
<td>Steel mills, rubber plantations, glass factories</td>
<td>Providers, Composition Functions</td>
</tr>
<tr>
<td><strong>Input</strong></td>
<td>Raw materials: steel, rubber, glass, dreams</td>
<td>XRD schema + XR instance</td>
</tr>
<tr>
<td><strong>Process</strong></td>
<td>The moving assembly line itself</td>
<td>Composition (Pipeline mode)</td>
</tr>
<tr>
<td><strong>Output</strong></td>
<td>A gleaming Model T in any colour you like (black)</td>
<td>A managed cloud resource in Azure</td>
</tr>
<tr>
<td><strong>Consumer</strong></td>
<td>A proud American buying their first automobile</td>
<td>Your app team, self-serving infra via GitOps</td>
</tr>
</tbody>
</table></div>

<p>“Gentlemen,” Ford announces to the assembled platform engineers, most of whom are wearing hoodies and looking slightly alarmed, “you have been hand-crafting automobiles. One. At. A. Time. Bespoke. Artisanal. <em>Slow.</em>”</p>

<p>He gestures at the chaos of tickets, Slack messages, and manually-clicked Azure portals.</p>

<p>“I see your problem. You have no assembly line.”</p>

<p>He’s right. And Crossplane v2 is about to fix that.</p>




<h2>
  
  
  The Supplier: Raw Materials Come From Somewhere
</h2>

<p>Ford didn’t grow his own rubber or smelt his own steel. He had <em>suppliers</em> — raw material producers who fed the factory with exactly what it needed to build cars. Without them, the assembly line grinds to a halt and everyone goes home disappointed.</p>

<p>In Crossplane v2, your <strong>Suppliers</strong> are:</p>

<ol>
<li>
<strong>Providers</strong> — the connections to cloud APIs (Azure, AWS, GCP). They’re your steel mills: they provide the raw managed resource types your platform will compose.</li>
<li>
<strong>Composition Functions</strong> — the specialised tools that know <em>how</em> to shape those raw materials. They’re your rubber vulcanisers and glass cutters.</li>
</ol>

<p>Let’s install ours. Ford would call this “building the supply chain.” We call it “applying some YAML.”</p>

<h3>
  
  
  Installing the Azure Provider (Your Steel Mill)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># provider-azure.yaml</span>
<span class="c1"># The steel mill — provides every Azure resource type we could want</span>
<span class="c1"># (and several we definitely don't, but they come along for the ride)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pkg.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Provider</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">provider-azure-resources</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">package</span><span class="pi">:</span> <span class="s">xpkg.upbound.io/upbound/provider-azure-resources:v1</span>
  <span class="c1"># Ford didn't question where the steel came from. We don't question the package registry.</span>
  <span class="na">runtimeConfigRef</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">provider-azure-runtime</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># provider-config.yaml</span>
<span class="c1"># The supplier contract — credentials and which Azure subscription to use</span>
<span class="c1"># Ford had to sign contracts with Carnegie Steel. We sign in with a Service Principal.</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">azure.upbound.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">ProviderConfig</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">azure-provider-config</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">credentials</span><span class="pi">:</span>
    <span class="na">source</span><span class="pi">:</span> <span class="s">Secret</span>
    <span class="na">secretRef</span><span class="pi">:</span>
      <span class="na">namespace</span><span class="pi">:</span> <span class="s">crossplane-system</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">azure-credentials</span>
      <span class="na">key</span><span class="pi">:</span> <span class="s">credentials</span>
      <span class="c1"># Keep this secret safer than Ford kept his assembly line patents</span>
</code></pre>

</div>



<h3>
  
  
  Installing Composition Functions (Your Specialist Tools)
</h3>

<p>Ford’s factory had specialist workers who knew exactly how to attach a chassis, fit a windshield, or bolt on an engine. In Crossplane v2, <strong>Functions</strong> are those specialists — they know how to transform your declarative intent into actual managed resources.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># functions/function-patch-and-transform.yaml</span>
<span class="c1"># The chassis-fitting specialist</span>
<span class="c1"># Knows how to take XR fields and patch them onto managed resources</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pkg.crossplane.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Function</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">package</span><span class="pi">:</span> <span class="s">xpkg.upbound.io/crossplane-contrib/function-patch-and-transform:v0.8.0</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># functions/function-go-templating.yaml</span>
<span class="c1"># The body-shop specialist — when you need more flexibility than simple patches</span>
<span class="c1"># Supports Go templates for complex transformations</span>
<span class="c1"># Ford's equivalent: the worker who could improvise when the part didn't quite fit</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pkg.crossplane.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Function</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">function-go-templating</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">package</span><span class="pi">:</span> <span class="s">xpkg.upbound.io/crossplane-contrib/function-go-templating:v0.7.0</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># functions/function-auto-ready.yaml</span>
<span class="c1"># The quality inspector at the end of the line</span>
<span class="c1"># Marks the XR as READY when all composed resources are healthy</span>
<span class="c1"># Ford had a man check every car before it left the factory. This is that man.</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pkg.crossplane.io/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Function</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">function-auto-ready</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">package</span><span class="pi">:</span> <span class="s">xpkg.upbound.io/crossplane-contrib/function-auto-ready:v0.3.0</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Apply everything in the right order</span>
<span class="c"># (Ford learned the hard way: steel before bolts, bolts before engine)</span>
kubectl apply <span class="nt">-f</span> providers/
kubectl apply <span class="nt">-f</span> functions/

<span class="c"># Check suppliers are healthy</span>
kubectl get providers
kubectl get functions

<span class="c"># Expected output — what a healthy supply chain looks like:</span>
<span class="c"># NAME                         INSTALLED   HEALTHY   PACKAGE                                                    AGE</span>
<span class="c"># provider-azure-resources     True        True      xpkg.upbound.io/upbound/provider-azure-resources:v1       2m</span>
<span class="c">#</span>
<span class="c"># NAME                                INSTALLED   HEALTHY   AGE</span>
<span class="c"># function-patch-and-transform        True        True      90s</span>
<span class="c"># function-go-templating              True        True      90s</span>
<span class="c"># function-auto-ready                 True        True      90s</span>
</code></pre>

</div>



<p>Ford’s supply chain lesson: <em>know where your raw materials come from, and make sure they’re reliable.</em> A supplier who delivers inconsistently is worse than no supplier at all.</p>




<h2>
  
  
  The Input: Defining What You’re Building
</h2>

<p>At Highland Park, Ford’s engineers didn’t just throw steel at workers and hope for the best. They had <strong>blueprints</strong> — precise specifications for every part. The blueprint told the factory: “We are building a Model T. It requires these inputs. Nothing else.”</p>

<p>In Crossplane v2, your <strong>Input</strong> has two layers:</p>

<ol>
<li>
<strong>The XRD (CompositeResourceDefinition)</strong> — the blueprint. It defines what parameters consumers can pass in. This is the platform team’s <em>contract</em> with the world.</li>
<li>
<strong>The XR (Composite Resource)</strong> — the actual order. An instance of that blueprint, submitted by a consumer, specifying their particular values.</li>
</ol>

<blockquote>
<p><strong>Crossplane v2 key change</strong>: XRDs now use <code>apiVersion: apiextensions.crossplane.io/v2</code>. The <code>scope</code> field defaults to <code>Namespaced</code>. Claims (<code>claimNames</code>) are <strong>not supported</strong> in v2 — XRs are consumed directly. This is Ford removing the middle-man dealership and letting customers order directly from the factory.</p>
</blockquote>

<h3>
  
  
  The XRD: Your Blueprint (The Factory Specification)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/azure-resource-group/xrd-01.yaml</span>
<span class="c1">#</span>
<span class="c1"># Henry Ford's equivalent: the engineering specification document.</span>
<span class="c1"># "A Model T shall require: one engine, four wheels, one chassis.</span>
<span class="c1">#  Colour options: black."</span>
<span class="c1">#</span>
<span class="c1"># Our equivalent:</span>
<span class="c1"># "An Azure Resource Group shall require: one name, one location.</span>
<span class="c1">#  Colour options: whatever tags the consumer provides."</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v2</span>          <span class="c1"># v2 — not v1. This matters.</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">CompositeResourceDefinition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="c1"># The XRD name is &lt;plural&gt;.&lt;group&gt; — Crossplane is strict about this</span>
  <span class="c1"># Like Ford labelling every blueprint "MODEL-T-SPEC-001" not "Car Thing"</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xazureresourcegroups.platform.example.com</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">scope</span><span class="pi">:</span> <span class="s">Namespaced</span>                                  <span class="c1"># v2 default: Namespaced. Explicit is better.</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">platform.example.com</span>                        <span class="c1"># Your platform's API group</span>
  <span class="na">names</span><span class="pi">:</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureResourceGroup</span>                        <span class="c1"># The 'X' prefix = Composite Resource convention</span>
    <span class="na">plural</span><span class="pi">:</span> <span class="s">xazureresourcegroups</span>
  <span class="c1"># claimNames: intentionally absent</span>
  <span class="c1"># Claims are not supported in apiextensions.crossplane.io/v2</span>
  <span class="c1"># Ford would say: "Order from the factory directly. No dealerships."</span>

  <span class="na">versions</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">v1alpha1</span>
      <span class="na">served</span><span class="pi">:</span> <span class="kc">true</span>         <span class="c1"># This version accepts requests</span>
      <span class="na">referenceable</span><span class="pi">:</span> <span class="kc">true</span>  <span class="c1"># Compositions reference this version</span>
      <span class="na">schema</span><span class="pi">:</span>
        <span class="na">openAPIV3Schema</span><span class="pi">:</span>
          <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
          <span class="na">properties</span><span class="pi">:</span>
            <span class="na">spec</span><span class="pi">:</span>
              <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
              <span class="na">properties</span><span class="pi">:</span>
                <span class="na">parameters</span><span class="pi">:</span>
                  <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
                  <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">The</span><span class="nv"> </span><span class="s">order</span><span class="nv"> </span><span class="s">form.</span><span class="nv"> </span><span class="s">What</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">consumer</span><span class="nv"> </span><span class="s">fills</span><span class="nv"> </span><span class="s">in."</span>
                  <span class="na">required</span><span class="pi">:</span>
                    <span class="pi">-</span> <span class="s">resourceGroupName</span>
                    <span class="pi">-</span> <span class="s">location</span>
                  <span class="na">properties</span><span class="pi">:</span>
                    <span class="na">resourceGroupName</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Name</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">Azure</span><span class="nv"> </span><span class="s">Resource</span><span class="nv"> </span><span class="s">Group"</span>
                      <span class="c1"># Ford: "What do you want on the nameplate?"</span>
                    <span class="na">location</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Azure</span><span class="nv"> </span><span class="s">region</span><span class="nv"> </span><span class="s">where</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">Resource</span><span class="nv"> </span><span class="s">Group</span><span class="nv"> </span><span class="s">will</span><span class="nv"> </span><span class="s">live"</span>
                      <span class="na">enum</span><span class="pi">:</span>
                        <span class="pi">-</span> <span class="s">westeurope</span>
                        <span class="pi">-</span> <span class="s">northeurope</span>
                        <span class="pi">-</span> <span class="s">eastus</span>
                        <span class="pi">-</span> <span class="s">eastus2</span>
                      <span class="c1"># Ford: "Which factory depot should we deliver to?"</span>
                    <span class="na">environment</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Target</span><span class="nv"> </span><span class="s">environment</span><span class="nv"> </span><span class="s">—</span><span class="nv"> </span><span class="s">affects</span><span class="nv"> </span><span class="s">tagging</span><span class="nv"> </span><span class="s">and</span><span class="nv"> </span><span class="s">policies"</span>
                      <span class="na">enum</span><span class="pi">:</span>
                        <span class="pi">-</span> <span class="s">dev</span>
                        <span class="pi">-</span> <span class="s">test</span>
                        <span class="pi">-</span> <span class="s">prod</span>
                      <span class="na">default</span><span class="pi">:</span> <span class="s">dev</span>
                      <span class="c1"># Ford: "Standard model or deluxe? (We make them the same anyway)"</span>
                    <span class="na">costCenter</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Cost</span><span class="nv"> </span><span class="s">center</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">billing</span><span class="nv"> </span><span class="s">tags"</span>
                      <span class="c1"># Ford: "Who's paying? Sign here."</span>
            <span class="na">status</span><span class="pi">:</span>
              <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
              <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">What</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">factory</span><span class="nv"> </span><span class="s">reports</span><span class="nv"> </span><span class="s">back</span><span class="nv"> </span><span class="s">after</span><span class="nv"> </span><span class="s">building"</span>
              <span class="na">properties</span><span class="pi">:</span>
                <span class="na">resourceGroupId</span><span class="pi">:</span>
                  <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                  <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">The</span><span class="nv"> </span><span class="s">Azure</span><span class="nv"> </span><span class="s">Resource</span><span class="nv"> </span><span class="s">ID</span><span class="nv"> </span><span class="s">of</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">provisioned</span><span class="nv"> </span><span class="s">Resource</span><span class="nv"> </span><span class="s">Group"</span>
                  <span class="c1"># Ford: "Your car's serial number — proof it was built"</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Apply the XRD — register the blueprint with the factory</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-resource-group/xrd-01.yaml

<span class="c"># Crossplane now creates a new Kubernetes CRD for XAzureResourceGroup</span>
<span class="c"># The factory has accepted the engineering specification</span>
kubectl get xrd xazureresourcegroups.platform.example.com

<span class="c"># Output:</span>
<span class="c"># NAME                                          ESTABLISHED   OFFERED   AGE</span>
<span class="c"># xazureresourcegroups.platform.example.com     True          True      30s</span>
</code></pre>

</div>



<h3>
  
  
  The XR: Placing an Order at the Factory Counter
</h3>

<p>Now a consumer can place an order. In Crossplane v2, this is an XR submitted directly into their namespace.</p>

<blockquote>
<p><strong>Crossplane v2 key change</strong>: All Crossplane machinery (<code>compositionRef</code>, <code>compositionRevisionRef</code>, <code>resourceRefs</code>) now lives under <code>spec.crossplane</code>. This keeps the consumer-facing <code>spec.parameters</code> clean — they don’t need to see the factory’s internal workings, just like a Model T buyer didn’t care about the assembly line choreography, only that the car arrived.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/azure-resource-group/xr-01.yaml</span>
<span class="c1">#</span>
<span class="c1"># This is the customer's order form.</span>
<span class="c1"># Ford: "Name, delivery address, standard or deluxe? Sign here."</span>
<span class="c1"># Us: "resourceGroupName, location, environment? Apply here."</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureResourceGroup</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">team-atlas-dev-rg</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">team-atlas</span>                              <span class="c1"># v2: namespace-scoped, no Claims needed</span>
  <span class="c1"># Ford: "Order placed at the Highland Park counter by Team Atlas"</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="c1"># Consumer-facing parameters — clean, no Crossplane noise</span>
  <span class="c1"># This is what the developer sees and fills in</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">resourceGroupName</span><span class="pi">:</span> <span class="s">atlas-dev-resources</span>
    <span class="na">location</span><span class="pi">:</span> <span class="s">westeurope</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">dev</span>
    <span class="na">costCenter</span><span class="pi">:</span> <span class="s2">"</span><span class="s">CC-ATLAS-001"</span>

  <span class="c1"># Crossplane v2: all machinery under spec.crossplane</span>
  <span class="c1"># The consumer doesn't need to understand this, just like</span>
  <span class="c1"># a Model T buyer didn't need to understand the production schedule</span>
  <span class="na">crossplane</span><span class="pi">:</span>
    <span class="na">compositionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">xazureresourcegroup-azure-v1alpha1</span>
      <span class="c1"># "Please build this using the standard azure composition"</span>
      <span class="c1"># Ford: "Standard Model T, please. Not the experimental one."</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Submit the order to the factory</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-resource-group/xr-01.yaml

<span class="c"># Watch Crossplane pick it up and start building</span>
kubectl get xazureresourcegroup team-atlas-dev-rg <span class="nt">-n</span> team-atlas

<span class="c"># Output — the factory floor status board:</span>
<span class="c"># NAME                  SYNCED   READY   COMPOSITION                              AGE</span>
<span class="c"># team-atlas-dev-rg     True     True    xazureresourcegroup-azure-v1alpha1       45s</span>
</code></pre>

</div>



<p>Ford’s Input lesson: <em>the blueprint (XRD) defines what can be ordered; the order form (XR) is the actual request.</em> Separate them clearly, and the factory can process thousands of orders without confusion.</p>




<h2>
  
  
  The Process: The Moving Assembly Line
</h2>

<p>Here’s where the magic happens. Ford’s assembly line didn’t build a car from a single huge machine — it was a <strong>pipeline of specialised stations</strong>, each doing one job perfectly before passing the work to the next.</p>

<p>Crossplane v2 <strong>Compositions</strong> work identically. A <code>mode: Pipeline</code> Composition is a sequence of Function steps, each transforming the XR’s desired state a little further, until the final output is a fully specified set of managed resources.</p>

<blockquote>
<p><strong>Crossplane v2 note</strong>: Compositions still use <code>apiVersion: apiextensions.crossplane.io/v1</code>. Only the XRD moved to v2. Compositions remain on v1 — this is a common gotcha, like expecting the assembly line to be redesigned just because you updated the order form.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Ford's assembly line stations:
  Station 1: Fit the engine                          → Step 1: function-patch-and-transform
  Station 2: Attach the chassis                      → Step 2: function-go-templating (if needed)
  Station 3: Fit the body                            → Step 3: more patches
  Station 4: Final inspection                        → Step N: function-auto-ready

Each station receives the work-in-progress, does its job, passes it on.
Each Function receives the observed/desired state, transforms it, passes it on.
</code></pre>

</div>



<h3>
  
  
  The Composition: Your Assembly Line
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/azure-resource-group/composition-01.yaml</span>
<span class="c1">#</span>
<span class="c1"># This is the assembly line itself.</span>
<span class="c1"># Ford designed every station, every movement, every tool placement.</span>
<span class="c1"># We define every pipeline step, every patch, every transformation.</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>          <span class="c1"># Compositions stay on v1. Don't forget this.</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xazureresourcegroup-azure-v1alpha1</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">provider</span><span class="pi">:</span> <span class="s">azure</span>
    <span class="na">product</span><span class="pi">:</span> <span class="s">resource-group</span>
    <span class="c1"># Ford labelled every assembly line variant. We label ours.</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="c1"># Which XR type does this assembly line handle?</span>
  <span class="c1"># Ford: "This line builds Model Ts. Not Model As. Model Ts."</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureResourceGroup</span>

  <span class="c1"># Crossplane v2: Pipeline mode is the way forward</span>
  <span class="c1"># The old inline 'resources' mode is to Pipeline mode</span>
  <span class="c1"># what horse-drawn wagons were to Ford's assembly line</span>
  <span class="na">mode</span><span class="pi">:</span> <span class="s">Pipeline</span>

  <span class="na">pipeline</span><span class="pi">:</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="c1"># ASSEMBLY STATION 1: Patch and Transform</span>
    <span class="c1"># Ford's engine-fitting station: take the raw XR fields and</span>
    <span class="c1"># stamp them onto the managed resource blueprint</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">patch-and-transform</span>
      <span class="na">functionRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
      <span class="na">input</span><span class="pi">:</span>
        <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pt.fn.crossplane.io/v1beta1</span>
        <span class="na">kind</span><span class="pi">:</span> <span class="s">Resources</span>
        <span class="na">resources</span><span class="pi">:</span>
          <span class="c1"># The actual managed resource we're building</span>
          <span class="c1"># Ford: "This is the car body. Everything gets stamped onto it."</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">azure-resource-group</span>
            <span class="na">base</span><span class="pi">:</span>
              <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">azure.upbound.io/v1beta1</span>
              <span class="na">kind</span><span class="pi">:</span> <span class="s">ResourceGroup</span>
              <span class="na">spec</span><span class="pi">:</span>
                <span class="na">forProvider</span><span class="pi">:</span>
                  <span class="na">location</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>   <span class="c1"># Will be stamped in from XR below</span>
                  <span class="na">tags</span><span class="pi">:</span>
                    <span class="na">ManagedBy</span><span class="pi">:</span> <span class="s">crossplane</span>
                    <span class="c1"># Additional tags stamped in from XR parameters</span>
                <span class="na">providerConfigRef</span><span class="pi">:</span>
                  <span class="na">name</span><span class="pi">:</span> <span class="s">azure-provider-config</span>

            <span class="na">patches</span><span class="pi">:</span>
              <span class="c1"># ── Stamp the location from XR parameter onto the Managed Resource ──</span>
              <span class="c1"># Ford Station 1A: "Fit the engine from the engine rack"</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.location</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.location</span>
                <span class="c1"># Read from XR spec.parameters.location</span>
                <span class="c1"># Write to the ResourceGroup's spec.forProvider.location</span>

              <span class="c1"># ── Stamp the resource group name as the external-name annotation ──</span>
              <span class="c1"># This controls the actual Azure resource name</span>
              <span class="c1"># Ford Station 1B: "Stamp the serial number on the chassis"</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.resourceGroupName</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">metadata.annotations[crossplane.io/external-name]</span>

              <span class="c1"># ── Stamp the environment tag ──</span>
              <span class="c1"># Ford Station 1C: "Apply the colour" (it's still black, but tagged)</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.environment</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.tags.Environment</span>

              <span class="c1"># ── Stamp the cost center tag ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.costCenter</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.tags.CostCenter</span>

              <span class="c1"># ── Combine name and environment into a useful name tag ──</span>
              <span class="c1"># Ford: "Print 'Model T - Standard' on the dashboard"</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">CombineFromComposite</span>
                <span class="na">combine</span><span class="pi">:</span>
                  <span class="na">variables</span><span class="pi">:</span>
                    <span class="pi">-</span> <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.resourceGroupName</span>
                    <span class="pi">-</span> <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.environment</span>
                  <span class="na">strategy</span><span class="pi">:</span> <span class="s">string</span>
                  <span class="na">string</span><span class="pi">:</span>
                    <span class="na">fmt</span><span class="pi">:</span> <span class="s2">"</span><span class="s">%s-%s"</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.tags.Name</span>

              <span class="c1"># ── Write the Azure Resource ID back to the XR status ──</span>
              <span class="c1"># Ford: "Update the order tracking system: car is built, serial = X"</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">ToCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">status.atProvider.id</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">status.resourceGroupId</span>
                <span class="na">policy</span><span class="pi">:</span>
                  <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">Optional</span>
                  <span class="c1"># Optional: the ID only exists after Azure creates the resource</span>
                  <span class="c1"># Like a car's serial number — it exists after manufacture, not before</span>

    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="c1"># ASSEMBLY STATION 2: Final Quality Inspection</span>
    <span class="c1"># Ford's end-of-line inspector who checked every car before it left</span>
    <span class="c1"># This function marks the XR as READY only when all managed</span>
    <span class="c1"># resources are healthy — your digital quality control department</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">auto-ready</span>
      <span class="na">functionRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">function-auto-ready</span>
      <span class="c1"># No input needed — this function inspects all resources from previous steps</span>
      <span class="c1"># Ford: "The inspector checks everything. If it passes, it ships."</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install the assembly line</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-resource-group/composition-01.yaml

<span class="c"># Check it's registered and healthy</span>
kubectl get composition xazureresourcegroup-azure-v1alpha1

<span class="c"># Output:</span>
<span class="c"># NAME                                    XR-KIND                GRPS                        AGE</span>
<span class="c"># xazureresourcegroup-azure-v1alpha1      XAzureResourceGroup    platform.example.com         10s</span>

<span class="c"># Ford would check: "Is the assembly line running? Are all stations staffed?"</span>
kubectl get functions
<span class="c"># NAME                              INSTALLED   HEALTHY   AGE</span>
<span class="c"># function-patch-and-transform      True        True      5m</span>
<span class="c"># function-go-templating            True        True      5m</span>
<span class="c"># function-auto-ready               True        True      5m</span>
<span class="c"># </span>
<span class="c"># Good. All stations are staffed. Production can begin.</span>
</code></pre>

</div>






<h2>
  
  
  A More Complex Assembly Line: Azure Subscription
</h2>

<p>Ford didn’t just build one model. The Model T was a simple product, but his factory principles applied equally to complex multi-component vehicles. Let’s see SIPOC applied to something more involved: an <strong>Azure Subscription</strong>, which involves multiple composed resources.</p>

<h3>
  
  
  The XRD Blueprint for an Azure Subscription
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/azure-subscription/xrd-01.yaml</span>
<span class="c1">#</span>
<span class="c1"># A more complex order form.</span>
<span class="c1"># Ford: "Sir, you're ordering a touring car with extra luggage capacity.</span>
<span class="c1">#        That requires additional specifications."</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v2</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">CompositeResourceDefinition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xazuresubscriptions.platform.example.com</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">scope</span><span class="pi">:</span> <span class="s">Namespaced</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">platform.example.com</span>
  <span class="na">names</span><span class="pi">:</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureSubscription</span>
    <span class="na">plural</span><span class="pi">:</span> <span class="s">xazuresubscriptions</span>
  <span class="na">versions</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">v1alpha1</span>
      <span class="na">served</span><span class="pi">:</span> <span class="kc">true</span>
      <span class="na">referenceable</span><span class="pi">:</span> <span class="kc">true</span>
      <span class="na">schema</span><span class="pi">:</span>
        <span class="na">openAPIV3Schema</span><span class="pi">:</span>
          <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
          <span class="na">properties</span><span class="pi">:</span>
            <span class="na">spec</span><span class="pi">:</span>
              <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
              <span class="na">properties</span><span class="pi">:</span>
                <span class="na">parameters</span><span class="pi">:</span>
                  <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
                  <span class="na">required</span><span class="pi">:</span>
                    <span class="pi">-</span> <span class="s">subscriptionDisplayName</span>
                    <span class="pi">-</span> <span class="s">billingAccountName</span>
                    <span class="pi">-</span> <span class="s">enrollmentAccountName</span>
                    <span class="pi">-</span> <span class="s">managementGroupId</span>
                  <span class="na">properties</span><span class="pi">:</span>
                    <span class="na">subscriptionDisplayName</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Display</span><span class="nv"> </span><span class="s">name</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">Azure</span><span class="nv"> </span><span class="s">Subscription"</span>
                      <span class="na">maxLength</span><span class="pi">:</span> <span class="m">64</span>
                      <span class="c1"># Ford: "What name goes on the registration document?"</span>

                    <span class="na">billingAccountName</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Enterprise</span><span class="nv"> </span><span class="s">Agreement</span><span class="nv"> </span><span class="s">billing</span><span class="nv"> </span><span class="s">account</span><span class="nv"> </span><span class="s">name"</span>
                      <span class="c1"># Ford: "Which corporate account is paying for this order?"</span>

                    <span class="na">enrollmentAccountName</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">EA</span><span class="nv"> </span><span class="s">enrollment</span><span class="nv"> </span><span class="s">account</span><span class="nv"> </span><span class="s">to</span><span class="nv"> </span><span class="s">create</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">subscription</span><span class="nv"> </span><span class="s">under"</span>
                      <span class="c1"># Ford: "Which dealer are we registered through?"</span>

                    <span class="na">managementGroupId</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Management</span><span class="nv"> </span><span class="s">Group</span><span class="nv"> </span><span class="s">to</span><span class="nv"> </span><span class="s">place</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">subscription</span><span class="nv"> </span><span class="s">into</span><span class="nv"> </span><span class="s">after</span><span class="nv"> </span><span class="s">creation"</span>
                      <span class="c1"># Ford: "Which factory district does this car go to after handover?"</span>

                    <span class="na">environment</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Subscription</span><span class="nv"> </span><span class="s">purpose</span><span class="nv"> </span><span class="s">—</span><span class="nv"> </span><span class="s">drives</span><span class="nv"> </span><span class="s">policy</span><span class="nv"> </span><span class="s">assignment"</span>
                      <span class="na">enum</span><span class="pi">:</span>
                        <span class="pi">-</span> <span class="s">sandbox</span>
                        <span class="pi">-</span> <span class="s">dev</span>
                        <span class="pi">-</span> <span class="s">test</span>
                        <span class="pi">-</span> <span class="s">prod</span>
                      <span class="na">default</span><span class="pi">:</span> <span class="s">sandbox</span>
                      <span class="c1"># Ford: "Standard, commercial, or racing spec?"</span>

                    <span class="na">workload</span><span class="pi">:</span>
                      <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                      <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Azure</span><span class="nv"> </span><span class="s">workload</span><span class="nv"> </span><span class="s">type"</span>
                      <span class="na">enum</span><span class="pi">:</span>
                        <span class="pi">-</span> <span class="s">Production</span>
                        <span class="pi">-</span> <span class="s">DevTest</span>
                      <span class="na">default</span><span class="pi">:</span> <span class="s">DevTest</span>
                      <span class="c1"># Ford: "Light duty or heavy haul?"</span>

            <span class="na">status</span><span class="pi">:</span>
              <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
              <span class="na">properties</span><span class="pi">:</span>
                <span class="na">subscriptionId</span><span class="pi">:</span>
                  <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                  <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Provisioned</span><span class="nv"> </span><span class="s">Azure</span><span class="nv"> </span><span class="s">Subscription</span><span class="nv"> </span><span class="s">GUID"</span>
                  <span class="c1"># Ford: "Your Model T serial number"</span>
                <span class="na">subscriptionState</span><span class="pi">:</span>
                  <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                  <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Current</span><span class="nv"> </span><span class="s">state</span><span class="nv"> </span><span class="s">of</span><span class="nv"> </span><span class="s">the</span><span class="nv"> </span><span class="s">subscription"</span>
                  <span class="c1"># Ford: "On the line / finished / shipped"</span>
</code></pre>

</div>



<h3>
  
  
  The XR: Placing the Subscription Order
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/azure-subscription/xr-01.yaml</span>
<span class="c1">#</span>
<span class="c1"># Team Atlas wants a production subscription.</span>
<span class="c1"># They fill in the order form. The factory does the rest.</span>
<span class="c1"># Ford: "Revolutionary concept."</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureSubscription</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">atlas-prod-subscription</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">team-atlas</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">subscriptionDisplayName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Atlas</span><span class="nv"> </span><span class="s">Production</span><span class="nv"> </span><span class="s">—</span><span class="nv"> </span><span class="s">Managed</span><span class="nv"> </span><span class="s">by</span><span class="nv"> </span><span class="s">Platform"</span>
    <span class="na">billingAccountName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">contoso-ea-billing"</span>
    <span class="na">enrollmentAccountName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">platform-enrollment"</span>
    <span class="na">managementGroupId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">mg-production"</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">prod</span>
    <span class="na">workload</span><span class="pi">:</span> <span class="s">Production</span>

  <span class="na">crossplane</span><span class="pi">:</span>
    <span class="na">compositionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">xazuresubscription-azure-v1alpha1</span>
    <span class="c1"># Crossplane v2: machinery under spec.crossplane, parameters stay clean</span>
    <span class="c1"># The consumer sees a simple form. The factory sees the pipeline.</span>
</code></pre>

</div>



<h3>
  
  
  The Composition: A More Complex Assembly Line
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/azure-subscription/composition-01.yaml</span>
<span class="c1">#</span>
<span class="c1"># The subscription assembly line is more involved.</span>
<span class="c1"># Ford didn't just bolt four wheels on a frame for his touring car —</span>
<span class="c1"># he had extra stations, extra specialists, extra quality checks.</span>
<span class="c1"># We follow the same principle.</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>          <span class="c1"># Compositions: still v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xazuresubscription-azure-v1alpha1</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">provider</span><span class="pi">:</span> <span class="s">azure</span>
    <span class="na">product</span><span class="pi">:</span> <span class="s">subscription</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureSubscription</span>

  <span class="na">mode</span><span class="pi">:</span> <span class="s">Pipeline</span>  <span class="c1"># Always Pipeline in v2-era Crossplane. The assembly line is the way.</span>

  <span class="na">pipeline</span><span class="pi">:</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="c1"># STATION 1: Create the Azure Subscription</span>
    <span class="c1"># Ford: "The main chassis station — everything else attaches to this"</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">create-subscription</span>
      <span class="na">functionRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
      <span class="na">input</span><span class="pi">:</span>
        <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pt.fn.crossplane.io/v1beta1</span>
        <span class="na">kind</span><span class="pi">:</span> <span class="s">Resources</span>
        <span class="na">resources</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">azure-subscription</span>
            <span class="na">base</span><span class="pi">:</span>
              <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">billing.azure.upbound.io/v1beta1</span>
              <span class="na">kind</span><span class="pi">:</span> <span class="s">Subscription</span>
              <span class="na">spec</span><span class="pi">:</span>
                <span class="na">forProvider</span><span class="pi">:</span>
                  <span class="na">subscriptionName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>       <span class="c1"># Patched from XR</span>
                  <span class="na">billingAccountName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>     <span class="c1"># Patched from XR</span>
                  <span class="na">enrollmentAccountName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>  <span class="c1"># Patched from XR</span>
                  <span class="na">workload</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>              <span class="c1"># Patched from XR</span>
                <span class="na">providerConfigRef</span><span class="pi">:</span>
                  <span class="na">name</span><span class="pi">:</span> <span class="s">azure-provider-config</span>

            <span class="na">patches</span><span class="pi">:</span>
              <span class="c1"># ── Subscription display name ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.subscriptionDisplayName</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.subscriptionName</span>

              <span class="c1"># ── Billing account ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.billingAccountName</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.billingAccountName</span>

              <span class="c1"># ── Enrollment account ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.enrollmentAccountName</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.enrollmentAccountName</span>

              <span class="c1"># ── Workload type ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.workload</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.workload</span>

              <span class="c1"># ── Write subscription ID back to XR status ──</span>
              <span class="c1"># "The chassis number is stamped. Update the tracking board."</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">ToCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">status.atProvider.subscriptionId</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">status.subscriptionId</span>
                <span class="na">policy</span><span class="pi">:</span>
                  <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">Optional</span>

              <span class="c1"># ── Write subscription state back to XR status ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">ToCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">status.atProvider.state</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">status.subscriptionState</span>
                <span class="na">policy</span><span class="pi">:</span>
                  <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">Optional</span>

    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="c1"># STATION 2: Place subscription into Management Group</span>
    <span class="c1"># Ford: "Route the finished car to the correct depot for distribution"</span>
    <span class="c1"># This station waits for the subscription ID from Station 1</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">assign-management-group</span>
      <span class="na">functionRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
      <span class="na">input</span><span class="pi">:</span>
        <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pt.fn.crossplane.io/v1beta1</span>
        <span class="na">kind</span><span class="pi">:</span> <span class="s">Resources</span>
        <span class="na">resources</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">management-group-subscription</span>
            <span class="na">base</span><span class="pi">:</span>
              <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">management.azure.upbound.io/v1beta1</span>
              <span class="na">kind</span><span class="pi">:</span> <span class="s">ManagementGroupSubscriptionAssociation</span>
              <span class="na">spec</span><span class="pi">:</span>
                <span class="na">forProvider</span><span class="pi">:</span>
                  <span class="na">managementGroupId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>     <span class="c1"># Patched from XR</span>
                  <span class="na">subscriptionId</span><span class="pi">:</span> <span class="s2">"</span><span class="s">"</span>        <span class="c1"># Patched from XR status (written by Station 1)</span>
                <span class="na">providerConfigRef</span><span class="pi">:</span>
                  <span class="na">name</span><span class="pi">:</span> <span class="s">azure-provider-config</span>

            <span class="na">patches</span><span class="pi">:</span>
              <span class="c1"># ── Management group from XR parameters ──</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.managementGroupId</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.managementGroupId</span>

              <span class="c1"># ── Subscription ID from XR status (written by Station 1) ──</span>
              <span class="c1"># Ford: "Take the chassis number from the previous station's stamp"</span>
              <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
                <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">status.subscriptionId</span>
                <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.subscriptionId</span>
                <span class="na">policy</span><span class="pi">:</span>
                  <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">Required</span>
                  <span class="c1"># Required: Station 2 won't proceed until Station 1 provides the ID</span>
                  <span class="c1"># Ford: "Don't attach the body before the chassis is finished"</span>

    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="c1"># STATION 3: Final Inspection</span>
    <span class="c1"># Ford: "Sign it off. If it passes, ship it."</span>
    <span class="c1"># ─────────────────────────────────────────────────────────────────</span>
    <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">auto-ready</span>
      <span class="na">functionRef</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">function-auto-ready</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Apply the subscription XRD and Composition</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-subscription/xrd-01.yaml
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-subscription/composition-01.yaml

<span class="c"># Place the order</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-subscription/xr-01.yaml

<span class="c"># Watch the assembly line process it</span>
kubectl get xazuresubscription atlas-prod-subscription <span class="nt">-n</span> team-atlas <span class="nt">-w</span>

<span class="c"># Output — watching the factory floor:</span>
<span class="c"># NAME                       SYNCED   READY   AGE</span>
<span class="c"># atlas-prod-subscription    True     False   5s    ← Building...</span>
<span class="c"># atlas-prod-subscription    True     False   15s   ← Station 1 working...</span>
<span class="c"># atlas-prod-subscription    True     False   30s   ← Station 2 working...</span>
<span class="c"># atlas-prod-subscription    True     True    45s   ← Shipped! Ready for use.</span>

<span class="c"># Inspect what the factory built (the Output)</span>
kubectl get managed | <span class="nb">grep </span>atlas

<span class="c"># Ford's equivalent: walking the factory floor and counting completed cars</span>
</code></pre>

</div>






<h2>
  
  
  The Output: What Rolls Off the Line
</h2>

<p>Ford’s assembly line produced <strong>one output</strong>: a Model T, ready to drive. Every single one identical. Predictable. Reliable.</p>

<p>Crossplane’s Pipeline produces <strong>managed resources</strong> — actual Azure infrastructure objects, reconciled continuously to match the desired state. Unlike Ford’s cars, which required no further attention once sold (until they broke down in a ditch), Crossplane’s managed resources are <em>continuously reconciled</em>. The factory floor never truly stops — it monitors every car it ever built and fixes deviations automatically.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Inspect the actual Azure resources that were created</span>
<span class="c"># These are the "cars" that rolled off the assembly line</span>

kubectl get resourcegroup
<span class="c"># NAME                    READY   SYNCED   EXTERNAL-NAME          AGE</span>
<span class="c"># atlas-dev-resources     True    True     atlas-dev-resources    2m</span>

kubectl get subscription
<span class="c"># NAME                         READY   SYNCED   EXTERNAL-NAME                          AGE</span>
<span class="c"># atlas-prod-subscription-x9k   True    True     a1b2c3d4-xxxx-xxxx-xxxx-xxxxxxxxxxxx   3m</span>

<span class="c"># Describe a managed resource to see the full factory output</span>
kubectl describe resourcegroup atlas-dev-resources

<span class="c"># Key things to look for in the output:</span>
<span class="c">#</span>
<span class="c"># Status.Conditions:</span>
<span class="c">#   Type: Ready       Status: True   ← Car passed quality inspection</span>
<span class="c">#   Type: Synced      Status: True   ← Factory monitoring confirms it matches spec</span>
<span class="c">#</span>
<span class="c"># Status.AtProvider:</span>
<span class="c">#   Id: /subscriptions/.../resourceGroups/atlas-dev-resources  ← Azure resource ID</span>
<span class="c">#   ProvisioningState: Succeeded    ← Azure confirms it exists</span>
<span class="c">#</span>
<span class="c"># This is Ford's quality sticker: "Built at Highland Park. Inspected. Approved."</span>
</code></pre>

</div>



<h3>
  
  
  Reconciliation: The Factory Never Sleeps
</h3>

<p>Here’s where Crossplane beats Ford’s factory: if someone <em>drives the car into a wall</em> (deletes a resource via the Azure portal), Crossplane fixes it automatically.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Simulate configuration drift — someone manually changes the resource group tags in Azure</span>
<span class="c"># Ford's equivalent: a disgruntled worker repainting a car red on the factory floor</span>

<span class="c"># Crossplane reconciles within seconds (default: every 10 minutes, </span>
<span class="c"># but triggered immediately when it detects drift)</span>
<span class="c"># The "car" is repainted black again, automatically</span>
<span class="c"># Ford would approve — he was very particular about the colour</span>

kubectl get events <span class="nt">-n</span> team-atlas | <span class="nb">grep </span>atlas-dev-rg
<span class="c"># LAST SEEN   TYPE      REASON                    OBJECT                        MESSAGE</span>
<span class="c"># 30s         Normal    ReconcileSuccess           xazureresourcegroup/atlas-dev-rg   Successfully reconciled resource group</span>
<span class="c"># 15s         Warning   ReconcileError             xazureresourcegroup/atlas-dev-rg   Observed drift, correcting...</span>
<span class="c"># 5s          Normal    ReconcileSuccess           xazureresourcegroup/atlas-dev-rg   Drift corrected. Good service resumed.</span>
</code></pre>

</div>



<p>Ford’s Output lesson: <em>the assembly line doesn’t stop at delivery. It monitors. It corrects. Every car stays a Model T, not a red Chevrolet someone converted it into over the weekend.</em></p>




<h2>
  
  
  The Consumer: Self-Service at the Showroom
</h2>

<p>Ford’s master stroke wasn’t just the assembly line. It was making the <strong>car affordable enough that the workers who built it could buy one</strong>. He democratised the automobile. Before Ford, cars were hand-crafted luxury items for the wealthy. After Ford, they were something everyone could access.</p>

<p>This is exactly what Crossplane v2 does for cloud infrastructure. Before Crossplane, developers needed a platform engineer to provision every resource — an artisan craftsman, expensive and slow. After Crossplane, developers can self-serve from a catalogue of approved, compliant, standardised resources. The factory floor is open to all customers.</p>

<h3>
  
  
  The Consumer Experience: Ordering a Car at the Showroom
</h3>

<p>In the old world:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Developer: "I need a Resource Group in West Europe for my dev environment"
Platform Engineer: (opens Azure Portal, clicks 47 things, forgets a tag, has to redo it)
Developer: (waits two days)
Platform Engineer: "Here it is. Also I misconfigured the IAM, sorry."
Developer: (cries quietly)
</code></pre>

</div>



<p>In the Crossplane v2 world:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># examples/team-payments-dev-rg.yaml</span>
<span class="c1"># A developer on the payments team ordering infrastructure</span>
<span class="c1"># Like walking into a Ford showroom with a completed order form</span>
<span class="c1">#</span>
<span class="c1"># They don't know (or need to know) about:</span>
<span class="c1"># - How Azure Resource Groups are actually created via ARM</span>
<span class="c1"># - Which service principal has which permissions</span>
<span class="c1"># - Which Composition pipeline runs behind the scenes</span>
<span class="c1"># - That there's a function-auto-ready checking everything is healthy</span>
<span class="c1">#</span>
<span class="c1"># They fill in: name, location, environment, cost centre.</span>
<span class="c1"># The factory handles the rest.</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">platform.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XAzureResourceGroup</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">payments-dev-rg</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">team-payments</span>          <span class="c1"># Namespaced in their own team namespace</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">resourceGroupName</span><span class="pi">:</span> <span class="s">payments-dev-resources</span>
    <span class="na">location</span><span class="pi">:</span> <span class="s">westeurope</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">dev</span>
    <span class="na">costCenter</span><span class="pi">:</span> <span class="s2">"</span><span class="s">CC-PAYMENTS-007"</span>
  <span class="na">crossplane</span><span class="pi">:</span>
    <span class="na">compositionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">xazureresourcegroup-azure-v1alpha1</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Developer applies this themselves, in their own namespace</span>
kubectl apply <span class="nt">-f</span> team-payments-dev-rg.yaml <span class="nt">-n</span> team-payments

<span class="c"># They can check their own order status without bothering the platform team</span>
kubectl get xazureresourcegroup <span class="nt">-n</span> team-payments

<span class="c"># Output:</span>
<span class="c"># NAME               SYNCED   READY   AGE</span>
<span class="c"># payments-dev-rg    True     True    60s</span>
<span class="c">#</span>
<span class="c"># "Your Model T is ready for collection, sir."</span>
<span class="c"># "Cheers, Ford."</span>

<span class="c"># They can see status details too</span>
kubectl describe xazureresourcegroup payments-dev-rg <span class="nt">-n</span> team-payments

<span class="c"># Status:</span>
<span class="c">#   Resource Group Id: /subscriptions/.../resourceGroups/payments-dev-resources</span>
<span class="c">#   Conditions:</span>
<span class="c">#     Ready:  True</span>
<span class="c">#     Synced: True</span>
<span class="c">#</span>
<span class="c"># Ford: "There's your serial number. She's all yours."</span>
</code></pre>

</div>



<h3>
  
  
  Multiple Teams, Multiple Orders, One Factory
</h3>

<p>The beauty of the assembly line is <strong>scale</strong>. Ford didn’t build one car at a time. He ran the line continuously, serving many customers simultaneously.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Multiple teams ordering simultaneously</span>
<span class="c"># Crossplane reconciles them all concurrently</span>

kubectl apply <span class="nt">-f</span> team-atlas-rg.yaml <span class="nt">-n</span> team-atlas
kubectl apply <span class="nt">-f</span> team-payments-rg.yaml <span class="nt">-n</span> team-payments
kubectl apply <span class="nt">-f</span> team-logistics-rg.yaml <span class="nt">-n</span> team-logistics
kubectl apply <span class="nt">-f</span> team-fraud-detection-rg.yaml <span class="nt">-n</span> team-fraud

<span class="c"># All four resource groups provisioned in parallel</span>
<span class="c"># Ford's equivalent: four different Model Ts simultaneously on the line</span>
<span class="c"># Each independent, each to spec, each delivered promptly</span>

kubectl get xazureresourcegroup <span class="nt">--all-namespaces</span>

<span class="c"># NAMESPACE        NAME                      SYNCED   READY   AGE</span>
<span class="c"># team-atlas       atlas-dev-rg              True     True    2m</span>
<span class="c"># team-payments    payments-dev-rg           True     True    90s</span>
<span class="c"># team-logistics   logistics-test-rg         True     True    75s</span>
<span class="c"># team-fraud       fraud-detection-prod-rg   True     True    60s</span>
<span class="c">#</span>
<span class="c"># Four cars. Four customers. One assembly line.</span>
<span class="c"># Ford would nod approvingly.</span>
</code></pre>

</div>



<h3>
  
  
  The Consumer’s RBAC: Not Everyone Gets the Keys
</h3>

<p>Ford sold to paying customers, not to any random person who wandered into the factory. Crossplane consumers are similarly scoped — RBAC ensures each team can only order (and see) their own resources.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Platform team applies this once — like setting up the showroom access policies</span>
<span class="c1"># developers in team-payments can only touch their namespace</span>

<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Role</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">crossplane-consumer</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">team-payments</span>
  <span class="c1"># Ford: "Customer pass for the Payments team showroom counter"</span>
<span class="na">rules</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">apiGroups</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">platform.example.com"</span><span class="pi">]</span>
    <span class="na">resources</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">xazureresourcegroups"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">xazuresubscriptions"</span><span class="pi">]</span>
    <span class="na">verbs</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">get"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">list"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">watch"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">create"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">update"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">patch"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">delete"</span><span class="pi">]</span>
    <span class="c1"># They can order, modify, and cancel their own resources</span>
    <span class="c1"># They cannot interfere with other teams' orders</span>
  <span class="pi">-</span> <span class="na">apiGroups</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">platform.example.com"</span><span class="pi">]</span>
    <span class="na">resources</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">xazureresourcegroups/status"</span><span class="pi">]</span>
    <span class="na">verbs</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">get"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">list"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">watch"</span><span class="pi">]</span>
    <span class="c1"># They can check the status board for their own orders</span>

<span class="nn">---</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">RoleBinding</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">team-payments-consumers</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">team-payments</span>
<span class="na">subjects</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">kind</span><span class="pi">:</span> <span class="s">Group</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">team-payments</span>       <span class="c1"># Azure AD group for the payments team</span>
    <span class="na">apiGroup</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io</span>
<span class="na">roleRef</span><span class="pi">:</span>
  <span class="na">kind</span><span class="pi">:</span> <span class="s">Role</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">crossplane-consumer</span>
  <span class="na">apiGroup</span><span class="pi">:</span> <span class="s">rbac.authorization.k8s.io</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># From a developer's perspective on the payments team:</span>

<span class="c"># ✅ This works — ordering from their own showroom counter</span>
kubectl apply <span class="nt">-f</span> payments-dev-rg.yaml <span class="nt">-n</span> team-payments

<span class="c"># ❌ This doesn't — they can't order from another team's counter</span>
kubectl apply <span class="nt">-f</span> payments-dev-rg.yaml <span class="nt">-n</span> team-atlas
<span class="c"># Error: rolebindings.rbac.authorization.k8s.io is forbidden</span>

<span class="c"># Ford: "Your pass is for Highland Park only, not the Detroit branch."</span>
</code></pre>

</div>






<h2>
  
  
  The Full Factory Floor: Putting It All Together
</h2>

<p>Let’s step back and see the complete SIPOC assembly line in one view. This is Ford’s factory, rendered in YAML.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>╔══════════════════════════════════════════════════════════════════════════════╗
║                    THE CROSSPLANE v2 ASSEMBLY LINE                          ║
╚══════════════════════════════════════════════════════════════════════════════╝

  SUPPLIER              INPUT                 PROCESS               OUTPUT           CONSUMER
  ─────────────         ─────────────         ─────────────         ─────────────    ─────────────
  Provider        ──▶  XRD              ──▶  Composition     ──▶  Managed      ──▶  App Team
  (Azure API)          (Blueprint)           (Assembly Line)       Resource          (self-serves
                                                                   (The Car)         in their
  Functions       ──▶  XR               ──▶  Pipeline Steps  ──▶  Azure API    ──▶  namespace)
  (Specialists)        (Order Form)          (Stations)            (Real cloud
                                                                   resource)
  apiVersion:          apiVersion:           apiVersion:           Managed by
  pkg.crossplane       apiextensions         apiextensions         Provider
  .io/v1beta1          .crossplane           .crossplane           Reconciled
  kind: Function       .io/v2                .io/v1                continuously
                       kind: CRD             kind:
                                             Composition
                        ↓
                       XR submitted
                       (apiVersion:
                       platform.
                       example.com/v1alpha1)
                       namespace-scoped
                       spec.crossplane.*
                       (v2 pattern)
</code></pre>

</div>



<h3>
  
  
  Quick Reference: Getting the API Versions Right
</h3>

<p>This trips people up more than anything else. Ford had one assembly line model. Crossplane v2 has two different API versions for two different object types:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># ┌─────────────────────────────────────────────────────────────────┐</span>
<span class="c1"># │  OBJECT TYPE                    │  API VERSION                  │</span>
<span class="c1"># ├─────────────────────────────────┼───────────────────────────────┤</span>
<span class="c1"># │  CompositeResourceDefinition    │  apiextensions.crossplane     │</span>
<span class="c1"># │  (XRD)                          │  .io/v2      ← v2!           │</span>
<span class="c1"># ├─────────────────────────────────┼───────────────────────────────┤</span>
<span class="c1"># │  Composition                    │  apiextensions.crossplane     │</span>
<span class="c1"># │                                 │  .io/v1      ← still v1!     │</span>
<span class="c1"># ├─────────────────────────────────┼───────────────────────────────┤</span>
<span class="c1"># │  Provider                       │  pkg.crossplane.io/v1         │</span>
<span class="c1"># │  Function                       │  pkg.crossplane.io/v1beta1    │</span>
<span class="c1"># ├─────────────────────────────────┼───────────────────────────────┤</span>
<span class="c1"># │  XR (Composite Resource)        │  your.group.com/v1alpha1      │</span>
<span class="c1"># │  (the actual order form)        │  (whatever you defined in XRD)│</span>
<span class="c1"># └─────────────────────────────────┴───────────────────────────────┘</span>
</code></pre>

</div>



<h3>
  
  
  Testing the Assembly Line Without Touching Azure
</h3>

<p>Ford had prototype workshops where engineers tested new assembly line configurations before rolling them out to production. Crossplane has <code>crossplane render</code> — a dry-run tool that simulates what your Composition will produce, without actually calling Azure.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Test the assembly line without production consequences</span>
<span class="c"># Ford: "Run it in the prototype shop first"</span>

crossplane render <span class="se">\</span>
  apis/v1alpha1/azure-resource-group/xr-01.yaml <span class="se">\</span>
  apis/v1alpha1/azure-resource-group/composition-01.yaml <span class="se">\</span>
  <span class="nt">--function-runtime-configs</span> functions/

<span class="c"># Output — what would be built:</span>
<span class="c"># ---</span>
<span class="c"># apiVersion: azure.upbound.io/v1beta1</span>
<span class="c"># kind: ResourceGroup</span>
<span class="c"># metadata:</span>
<span class="c">#   annotations:</span>
<span class="c">#     crossplane.io/composition-resource-name: azure-resource-group</span>
<span class="c">#     crossplane.io/external-name: atlas-dev-resources    ← Azure resource name</span>
<span class="c">#   name: team-atlas-dev-rg-xkp9m</span>
<span class="c"># spec:</span>
<span class="c">#   forProvider:</span>
<span class="c">#     location: westeurope                                 ← Patched from XR</span>
<span class="c">#     tags:</span>
<span class="c">#       CostCenter: CC-ATLAS-001                          ← Patched from XR</span>
<span class="c">#       Environment: dev                                  ← Patched from XR</span>
<span class="c">#       ManagedBy: crossplane</span>
<span class="c">#       Name: atlas-dev-resources-dev                     ← Combined patch</span>
<span class="c">#   providerConfigRef:</span>
<span class="c">#     name: azure-provider-config</span>
<span class="c">#</span>
<span class="c"># Ford: "Prototype approved. Start production."</span>
</code></pre>

</div>






<h2>
  
  
  Debugging the Assembly Line
</h2>

<p>Every factory has breakdowns. Ford’s workers had spanners. We have <code>kubectl describe</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Something isn't working. Let's diagnose.</span>
<span class="c"># Ford: "Which station is the hold-up?"</span>

kubectl get xazureresourcegroup atlas-dev-rg <span class="nt">-n</span> team-atlas

<span class="c"># SYNCED   READY</span>
<span class="c"># False    False   ← Something is wrong at one of the stations</span>

<span class="c"># Step 1: Check what the XR says</span>
kubectl describe xazureresourcegroup atlas-dev-rg <span class="nt">-n</span> team-atlas

<span class="c"># Look for the Events section at the bottom:</span>
<span class="c"># Events:</span>
<span class="c">#   Type      Reason             Message</span>
<span class="c">#   Warning   CannotObserveComposedResource   cannot get object: resource group "atlas-dev-resources" not found</span>
<span class="c">#</span>
<span class="c"># Ford: "Station 1 says the chassis hasn't arrived. Check the steel supplier."</span>

<span class="c"># Step 2: Check the managed resources the Composition created</span>
kubectl get managed | <span class="nb">grep </span>atlas

<span class="c"># NAME                        READY   SYNCED   EXTERNAL-NAME          AGE</span>
<span class="c"># atlas-dev-resources-xkp9m   False   False    atlas-dev-resources    30s</span>

kubectl describe resourcegroup atlas-dev-resources-xkp9m

<span class="c"># Look for:</span>
<span class="c"># Status.Conditions:</span>
<span class="c">#   Synced: False</span>
<span class="c">#   Last Transition Time: ...</span>
<span class="c">#   Message: cannot create ResourceGroup: authorization error...</span>
<span class="c">#</span>
<span class="c"># Ford: "The steel mill is refusing to deliver. Check the supplier contract (ProviderConfig credentials)."</span>

<span class="c"># Step 3: Check the Provider is healthy</span>
kubectl get providers
kubectl describe provider provider-azure-resources

<span class="c"># Step 4: Check Function logs (Station worker logs)</span>
kubectl get functions
kubectl logs <span class="nt">-n</span> crossplane-system <span class="nt">-l</span> pkg.crossplane.io/revision<span class="o">=</span><span class="k">function</span><span class="nt">-patch-and-transform</span>

<span class="c"># Ford: "Ask the station workers what happened. They saw everything."</span>
</code></pre>

</div>



<p>Common issues and their Ford analogies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Problem: Provider not HEALTHY
Ford: "The steel mill is on strike. No materials, no production."
Fix:  Check ProviderConfig credentials, Service Principal permissions

Problem: XRD not ESTABLISHED
Ford: "The engineering blueprint has errors. Production cannot start."
Fix:  Validate XRD schema — check for required/optional field mistakes

Problem: Composition not selecting correct XR
Ford: "The assembly line doesn't recognise this order form."
Fix:  Check compositeTypeRef matches your XRD group/kind exactly

Problem: Patch path not found
Ford: "Station worker reached for a part that wasn't on the rack."
Fix:  Verify fromFieldPath exists in XR; use policy.fromFieldPath: Optional if it might be absent

Problem: Managed resource stuck in SYNCED: False
Ford: "The car is half-built and nobody knows why it stopped."
Fix:  kubectl describe the managed resource; check status.conditions for Azure API errors
</code></pre>

</div>






<h2>
  
  
  The Repository Structure: Your Factory Blueprints Archive
</h2>

<p>Every Ford factory had a blueprint archive. Every Crossplane repository should have a clear structure. Here’s how we’ve laid ours out in <code>learning-crossplane-sipoc</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>learning-crossplane-sipoc/              ← The factory archive
│
├── apis/                               ← Engineering blueprints (XRDs, XRs, Compositions)
│   └── v1alpha1/
│       ├── azure-resource-group/       ← Model T (simple product)
│       │   ├── xrd-01.yaml             ← Blueprint v1: what can be ordered
│       │   ├── xr-01.yaml              ← Example order form
│       │   ├── composition-01.yaml     ← Assembly line for this product
│       │   └── README.md               ← SIPOC walkthrough for this product
│       │
│       ├── azure-subscription/         ← Touring car (complex product)
│       │   ├── xrd-01.yaml
│       │   ├── xr-01.yaml
│       │   ├── composition-01.yaml
│       │   └── README.md
│       │
│       └── azure-aks-cluster/          ← Racing car (advanced product)
│           ├── xrd-01.yaml
│           ├── xr-01.yaml
│           ├── composition-01.yaml
│           └── README.md
│
├── providers/                          ← Supplier contracts
│   ├── provider-azure.yaml             ← Steel mill contract
│   ├── provider-config.yaml            ← Supplier credentials
│   └── README.md
│
├── functions/                          ← Specialist tools
│   ├── function-patch-and-transform.yaml
│   ├── function-go-templating.yaml
│   ├── function-auto-ready.yaml
│   └── README.md
│
├── tests/                              ← Prototype workshop
│   ├── render/                         ← crossplane render dry-run outputs
│   │   ├── azure-resource-group/
│   │   │   └── expected-output.yaml    ← What we expect the line to produce
│   │   └── azure-subscription/
│   │       └── expected-output.yaml
│   └── README.md
│
├── examples/                           ← Showroom order forms
│   ├── team-atlas-dev-rg.yaml          ← Example customer orders
│   ├── team-payments-dev-rg.yaml
│   ├── atlas-prod-subscription.yaml
│   └── README.md
│
└── docs/                               ← Factory manuals
    ├── sipoc-explained.md              ← This mental model, in detail
    ├── crossplane-v2-changes.md        ← What changed from v1 (important!)
    ├── composition-functions.md        ← How the pipeline stations work
    ├── provider-setup.md               ← Getting the suppliers connected
    └── glossary.md                     ← Factory terminology
</code></pre>

</div>






<h2>
  
  
  The Verdict: Ford Was Right
</h2>

<p>Henry Ford’s insight wasn’t just about efficiency. It was about <strong>democratisation through standardisation</strong>. Make the process repeatable, make the product consistent, make access universal.</p>

<p>Crossplane v2 is the same insight applied to cloud infrastructure:</p>

<ul>
<li>
<strong>Repeatable</strong>: the same Composition produces the same resources, every time</li>
<li>
<strong>Consistent</strong>: XRDs enforce the same schema, across every team</li>
<li>
<strong>Universal</strong>: any team can self-serve, in their own namespace, with appropriate RBAC</li>
</ul>

<p>The differences between 1913 and now are mostly cosmetic:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Ford’s Factory</th>
<th>Crossplane v2</th>
</tr>
</thead>
<tbody>
<tr>
<td>Moving assembly line</td>
<td>Pipeline Compositions</td>
</tr>
<tr>
<td>Engineering blueprint</td>
<td>CompositeResourceDefinition (v2)</td>
</tr>
<tr>
<td>Customer order form</td>
<td>Composite Resource (XR)</td>
</tr>
<tr>
<td>Specialised workers</td>
<td>Composition Functions</td>
</tr>
<tr>
<td>Supply chain</td>
<td>Providers</td>
</tr>
<tr>
<td>Quality inspector</td>
<td>function-auto-ready</td>
</tr>
<tr>
<td>Factory floor CCTV</td>
<td><code>kubectl get managed</code></td>
</tr>
<tr>
<td>Car serial number</td>
<td><code>status.atProvider.id</code></td>
</tr>
<tr>
<td>“Any colour, as long as it’s black”</td>
<td>“Any resource, as long as it’s in the Composition”</td>
</tr>
<tr>
<td>15 million Model Ts produced</td>
<td>Theoretical infinite managed resources (your cloud bill may vary)</td>
</tr>
</tbody>
</table></div>

<p>Ford paid his workers $5 a day so they could afford the cars they built. Platform teams should build platforms that developers <em>actually want to use</em> — self-service, fast, compliant, and boring in all the right ways.</p>

<p>If developers are bypassing your platform and clicking around the Azure Portal manually, you don’t have a Crossplane problem. You have a UX problem. The assembly line should be the path of least resistance.</p>

<p>Build the line. Standardise the product. Let the consumers self-serve.</p>

<p>And remember Ford’s final lesson: <strong>you don’t have to offer every colour. Black is fine. Consistency is the point.</strong></p>




<p><em>Somewhere in Highland Park, the ghost of Henry Ford examines a running Crossplane cluster, nods once, and says “Now THAT is an assembly line.” He then asks why everything is in YAML instead of stamped metal, and we explain it’s 2025. He does not look convinced. We show him the reconciliation loop. He looks slightly more convinced.</em></p>

<p><em>He asks about the colour options.</em></p>

<p><em>We say it’s whatever the XRD allows.</em></p>

<p><em>He nods. “Good.”</em></p>




<h2>
  
  
  Getting Started on Your Own Assembly Line
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step 1: Create a Kind cluster (your prototype factory floor)</span>
kind create cluster <span class="nt">--name</span> crossplane-factory

<span class="c"># Step 2: Install Crossplane (build the factory building)</span>
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm <span class="nb">install </span>crossplane crossplane-stable/crossplane <span class="se">\</span>
  <span class="nt">--namespace</span> crossplane-system <span class="se">\</span>
  <span class="nt">--create-namespace</span> <span class="se">\</span>
  <span class="nt">--version</span> <span class="s2">"&gt;= 2.0.0"</span>   <span class="c"># We want v2!</span>

<span class="c"># Step 3: Install providers and functions (sign the supplier contracts)</span>
kubectl apply <span class="nt">-f</span> providers/
kubectl apply <span class="nt">-f</span> functions/

<span class="c"># Step 4: Wait for suppliers to be ready</span>
kubectl get providers <span class="nt">--watch</span>
kubectl get functions <span class="nt">--watch</span>

<span class="c"># Step 5: Apply your XRD (register the blueprint)</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-resource-group/xrd-01.yaml

<span class="c"># Step 6: Apply your Composition (commission the assembly line)</span>
kubectl apply <span class="nt">-f</span> apis/v1alpha1/azure-resource-group/composition-01.yaml

<span class="c"># Step 7: Dry-run test (prototype workshop)</span>
crossplane render <span class="se">\</span>
  apis/v1alpha1/azure-resource-group/xr-01.yaml <span class="se">\</span>
  apis/v1alpha1/azure-resource-group/composition-01.yaml <span class="se">\</span>
  <span class="nt">--function-runtime-configs</span> functions/

<span class="c"># Step 8: Submit your first XR (place your first order)</span>
kubectl apply <span class="nt">-f</span> examples/team-atlas-dev-rg.yaml <span class="nt">-n</span> team-atlas

<span class="c"># Step 9: Watch it roll off the line</span>
kubectl get xazureresourcegroup <span class="nt">-n</span> team-atlas <span class="nt">--watch</span>

<span class="c"># Step 10: Admire the output</span>
kubectl get managed
<span class="c"># Ford: "Beautiful. Now build ten thousand more."</span>
</code></pre>

</div>






<h2>
  
  
  Further Reading
</h2>

<h3>
  
  
  Crossplane Documentation
</h3>

<ul>
<li>
<a href="https://docs.crossplane.io/latest/" rel="noopener noreferrer">Crossplane v2 Official Docs</a> — The factory manual, updated for the new model year</li>
<li>
<a href="https://docs.crossplane.io/latest/whats-new/" rel="noopener noreferrer">What’s New in v2</a> — The release notes Ford would have written, if Ford wrote release notes</li>
<li>
<a href="https://docs.crossplane.io/latest/composition/composite-resource-definitions/" rel="noopener noreferrer">Composite Resource Definitions</a> — Blueprint specification guide</li>
<li>
<a href="https://docs.crossplane.io/latest/concepts/composition-functions/" rel="noopener noreferrer">Composition Functions</a> — The assembly station worker manual</li>
<li>
<a href="https://marketplace.upbound.io/providers/upbound/provider-azure-resources/" rel="noopener noreferrer">Upbound Azure Provider</a> — Your steel mill catalogue</li>
</ul>

<h3>
  
  
  Historical Context (For Verisimilitude)
</h3>

<ul>
<li>
<a href="https://www.thehenryford.org/visit/henry-ford-museum/exhibits/made-in-america/" rel="noopener noreferrer">The Highland Park Ford Plant</a> — The original assembly line, now a museum</li>
<li>
<a href="https://en.wikipedia.org/wiki/SIPOC" rel="noopener noreferrer">The SIPOC Model</a> — Lean process improvement tool, predates Crossplane by about 80 years</li>
<li>
<a href="https://www.thehenryford.org/collections-and-research/digital-collections/artifact/210133" rel="noopener noreferrer">Ford’s Five Dollar Day</a> — Democratising access through standardisation</li>
</ul>

<h3>
  
  
  Repository
</h3>

<ul>
<li>
<a href="https://github.com/stallone/learning-crossplane-sipoc" rel="noopener noreferrer">learning-crossplane-sipoc</a> — All the code from this article, properly laid out in assembly-line order</li>
</ul>




<p><em>This article is part of the “Infrastructure-as-Code Adventures” series:</em></p>

<ul>
<li>
<strong>Crossplane Networking</strong>: Mind the Gap — The London Underground <em>(published)</em>
</li>
<li>
<strong>Crossplane SIPOC</strong>: The Ford Assembly Line — You are here, on the line</li>
<li>
<strong>Crossplane Environments</strong>: The Scaling Problem — Or: Why Ford Built More Than One Factory <em>(coming soon)</em>
</li>
<li>
<strong>Crossplane with Backstage</strong>: The Showroom Catalogue — Making consumers feel like customers <em>(coming soon)</em>
</li>
</ul>




<p><em>Estimated reading time: about as long as a Model T took to traverse an early twentieth-century road. Which is to say: longer than you’d expect, but you arrive feeling like you understood something important.</em></p>

<p><strong>Tags:</strong> #crossplane #kubernetes #platformengineering #devops #iac #sipoc #azure #crossplanev2</p>




<p><strong>Disclaimer:</strong> No actual Henry Fords were resurrected for the writing of this article. The Model T was eventually discontinued in 1927, replaced by the Model A, which had more colour options. Crossplane’s Compositions have no such limitation, though your organisation’s governance policies might.</p>

<p>Henry Ford’s actual views on labour relations, antisemitism, and various other matters are not endorsed by this article, the author, or the Crossplane project. We are here for the assembly line metaphor only. The man built a good assembly line. We will acknowledge that and stop there.</p>

<p><em>“Coming together is a beginning, staying together is progress, and working together is success.”</em> — Henry Ford, on teams that use GitOps properly.</p>

