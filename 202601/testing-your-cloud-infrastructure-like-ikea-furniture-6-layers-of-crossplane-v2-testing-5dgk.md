---
Title: Testing Your Cloud Infrastructure Like IKEA Furniture: 6 Layers of Crossplane v2 Testing (PostgreSQL Example)
Description: 
Author: Willem van Heemstra
Date: 2026-01-31T21:41:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever assembled an IKEA BILLY bookshelf? You open the box, lay out the pieces, follow the steps, tighten everything, and then do the most important part: give it a careful shake before trusting it with a shelf full of books.</p>

<p>Testing Crossplane compositions is the same idea—except your “bookshelf” is cloud infrastructure, and your “Allen key” is <code>kubectl</code>.</p>

<p>This article refactors the testing story into <strong>six testing layers</strong> (0–5), and uses a single prime target throughout: a <strong>PostgreSQL database</strong> composition (Azure PostgreSQL Flexible Server + Database) built with Crossplane v2 pipeline mode.</p>

<h2>
  
  
  Why test your infrastructure? (aka “don’t skip the instructions”)
</h2>

<p>The failure mode is familiar:</p>

<ul>
<li>You changed “one small thing” in a composition</li>
<li>The rendered YAML looks plausible</li>
<li>The cluster accepts it</li>
<li>The cloud provider doesn’t</li>
<li>Cleanup leaves something behind</li>
</ul>

<p>A layered strategy gives you fast feedback early, and high confidence later—without paying the cost of running full end-to-end tests for every tiny change.</p>

<h2>
  
  
  The IKEA metaphor (mapping)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>IKEA furniture</th>
<th>Crossplane v2</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Instruction manual</strong></td>
<td>XRD (CompositeResourceDefinition)</td>
</tr>
<tr>
<td><strong>Assembly steps</strong></td>
<td>Composition (pipeline mode) + Functions</td>
</tr>
<tr>
<td><strong>Individual pieces</strong></td>
<td>Managed Resources (e.g., <code>ResourceGroup</code>, <code>FlexibleServer</code>, <code>FlexibleServerDatabase</code>)</td>
</tr>
<tr>
<td><strong>Assembled furniture</strong></td>
<td>Composite Resource (XR)</td>
</tr>
<tr>
<td><strong>Quality checks</strong></td>
<td>Layered test suite (0–5)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  The six testing layers (0–5)
</h2>

<p>Here’s the structure we’ll follow (adapted from the platform’s testing strategy docs):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Name</th>
<th>Primary intent</th>
<th>Typical tools</th>
</tr>
</thead>
<tbody>
<tr>
<td>0</td>
<td>Local composition rendering</td>
<td>Validate XRD + Composition logic without a cluster</td>
<td><code>crossplane render</code></td>
</tr>
<tr>
<td>1</td>
<td>Cluster health + provider validation</td>
<td>Ensure Crossplane stack is stable; providers/functions Healthy</td>
<td>
<code>kubectl</code>, health scripts, optional Uptest</td>
</tr>
<tr>
<td>2</td>
<td>Visual inspection &amp; relationship debugging</td>
<td>Understand XR → managed resources graph and conditions</td>
<td>Crossview</td>
</tr>
<tr>
<td>3</td>
<td>In-cluster E2E tests</td>
<td>Validate reconciliation behavior and lifecycle</td>
<td>KUTTL</td>
</tr>
<tr>
<td>4</td>
<td>Cloud-side verification</td>
<td>Confirm real Azure resources match intent</td>
<td>Azure CLI</td>
</tr>
<tr>
<td>5</td>
<td>GitOps deployment &amp; monitoring</td>
<td>Continuous reconciliation, drift detection, ops visibility</td>
<td>Flux + Headlamp</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Our “flat-pack” example: PostgreSQL as a platform API
</h2>

<p>We’ll build a small Crossplane API package that gives platform consumers a namespaced XR:</p>

<ul>
<li>XR kind: <code>XPostgreSQLDatabase</code>
</li>
<li>Composed resources: <code>ResourceGroup</code> + <code>FlexibleServer</code> + <code>FlexibleServerDatabase</code>
</li>
</ul>

<p>Here’s the shape of the “instruction manual” (XRD) from the demo (trimmed):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/postgresql-databases/xrd.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v2</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">CompositeResourceDefinition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xpostgresqldatabases.database.example.io</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">database.example.io</span>
  <span class="na">names</span><span class="pi">:</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLDatabase</span>
    <span class="na">plural</span><span class="pi">:</span> <span class="s">xpostgresqldatabases</span>
  <span class="na">scope</span><span class="pi">:</span> <span class="s">Namespaced</span>
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
                <span class="na">properties</span><span class="pi">:</span>
                  <span class="na">location</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">westeurope</span> <span class="pi">}</span>
                  <span class="na">resourceGroupName</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span> <span class="pi">}</span>
                  <span class="na">databaseName</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">appdb</span> <span class="pi">}</span>
                  <span class="na">adminUsername</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">pgadmin</span> <span class="pi">}</span>
                  <span class="na">adminPasswordSecretName</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">postgres-admin-password</span> <span class="pi">}</span>
                  <span class="na">adminPasswordSecretKey</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">password</span> <span class="pi">}</span>
                  <span class="na">postgresVersion</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="s2">"</span><span class="s">16"</span> <span class="pi">}</span>
                  <span class="na">skuName</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">B_Standard_B1ms</span> <span class="pi">}</span>
                  <span class="na">storageMb</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">integer</span><span class="pi">,</span> <span class="nv">default</span><span class="pi">:</span> <span class="nv">32768</span> <span class="pi">}</span>
                <span class="na">required</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">resourceGroupName</span><span class="pi">]</span>
          <span class="na">status</span><span class="pi">:</span>
            <span class="na">type</span><span class="pi">:</span> <span class="s">object</span>
            <span class="na">properties</span><span class="pi">:</span>
              <span class="na">serverName</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span> <span class="pi">}</span>
              <span class="na">databaseName</span><span class="pi">:</span> <span class="pi">{</span> <span class="nv">type</span><span class="pi">:</span> <span class="nv">string</span> <span class="pi">}</span>
</code></pre>

</div>



<p>And here’s the “assembly steps” (Composition) in pipeline mode—showing the three composed resources and how we push useful IDs up into XR status (trimmed):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/postgresql-databases/composition.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xpostgresqldatabases.database.example.io</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">provider</span><span class="pi">:</span> <span class="s">azure</span>
    <span class="na">type</span><span class="pi">:</span> <span class="s">standard</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.io/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLDatabase</span>
  <span class="na">mode</span><span class="pi">:</span> <span class="s">Pipeline</span>
  <span class="na">pipeline</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">patch-and-transform</span>
    <span class="na">functionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
    <span class="na">input</span><span class="pi">:</span>
      <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pt.fn.crossplane.io/v1beta1</span>
      <span class="na">kind</span><span class="pi">:</span> <span class="s">Resources</span>
      <span class="na">resources</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">resourcegroup</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">azure.m.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">ResourceGroup</span>
        <span class="c1"># patches: location, external-name, tags, etc.</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">flexibleserver</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">dbforpostgresql.azure.m.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">FlexibleServer</span>
        <span class="na">patches</span><span class="pi">:</span>
          <span class="c1"># ... name transforms to satisfy Azure constraints ...</span>
          <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">ToCompositeFieldPath</span>
            <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">metadata.annotations[crossplane.io/external-name]</span>
            <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">status.serverName</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">flexibleserverdatabase</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">dbforpostgresql.azure.m.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">FlexibleServerDatabase</span>
        <span class="na">patches</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
            <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.databaseName</span>
            <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">metadata.annotations[crossplane.io/external-name]</span>
          <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">ToCompositeFieldPath</span>
            <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">metadata.annotations[crossplane.io/external-name]</span>
            <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">status.databaseName</span>
  <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">auto-ready</span>
    <span class="na">functionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">function-auto-ready</span>
</code></pre>

</div>



<p>In the repo, the canonical paths used in the demo are:</p>

<ul>
<li><code>apis/v1alpha1/postgresql-databases/xrd.yaml</code></li>
<li><code>apis/v1alpha1/postgresql-databases/composition.yaml</code></li>
<li><code>apis/v1alpha1/postgresql-databases/examples/basic.yaml</code></li>
<li>
<code>tests/e2e/postgresql-databases/basic/</code> (KUTTL)</li>
</ul>

<h2>
  
  
  Layer 0 — Local composition rendering (unbox the parts)
</h2>

<p>Before you touch a cluster, validate that your “instruction manual + steps” actually produce the right parts.</p>

<p>Example XR used for rendering (trimmed):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># apis/v1alpha1/postgresql-databases/examples/basic.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.io/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLDatabase</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">render-postgres-example</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">crossplane</span><span class="pi">:</span>
    <span class="na">compositionSelector</span><span class="pi">:</span>
      <span class="na">matchLabels</span><span class="pi">:</span>
        <span class="na">provider</span><span class="pi">:</span> <span class="s">azure</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">standard</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">location</span><span class="pi">:</span> <span class="s">westeurope</span>
    <span class="na">resourceGroupName</span><span class="pi">:</span> <span class="s">crossplane-e2e-test-rg</span>
    <span class="na">databaseName</span><span class="pi">:</span> <span class="s">appdb</span>
    <span class="na">adminUsername</span><span class="pi">:</span> <span class="s">pgadmin</span>
    <span class="na">adminPasswordSecretName</span><span class="pi">:</span> <span class="s">postgres-admin-password</span>
    <span class="na">adminPasswordSecretKey</span><span class="pi">:</span> <span class="s">password</span>
    <span class="na">postgresVersion</span><span class="pi">:</span> <span class="s2">"</span><span class="s">16"</span>
    <span class="na">skuName</span><span class="pi">:</span> <span class="s">B_Standard_B1ms</span>
    <span class="na">storageMb</span><span class="pi">:</span> <span class="m">32768</span>
</code></pre>

</div>



<p>Minimal render example (from the demo):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>crossplane render <span class="se">\</span>
  apis/v1alpha1/postgresql-databases/xrd.yaml <span class="se">\</span>
  apis/v1alpha1/postgresql-databases/composition.yaml <span class="se">\</span>
  apis/v1alpha1/postgresql-databases/examples/basic.yaml <span class="se">\</span>
  <span class="nt">--include-function-results</span> <span class="se">\</span>
  <span class="o">&gt;</span> rendered-output.yaml
</code></pre>

</div>



<p>This is where you catch:</p>

<ul>
<li>wrong patch paths</li>
<li>schema mismatches between XR parameters and Composition expectations</li>
<li>naming/transform issues (e.g., lowercasing and sanitizing Azure names)</li>
</ul>

<p>If you maintain multiple APIs, treat examples as contracts and render them all (the demo includes a <code>scripts/render-all.sh</code> pattern that’s suitable for pre-commit and CI).</p>

<h2>
  
  
  Layer 1 — Cluster validation &amp; health (check your workshop is stable)
</h2>

<p>Even a perfect render can fail if the workshop is broken:</p>

<ul>
<li>Crossplane core isn’t stable</li>
<li>providers/functions aren’t Healthy</li>
<li>webhooks are timing out (common on local clusters under load)</li>
<li>
<code>ProviderConfig</code> credentials are misconfigured</li>
</ul>

<p>The demo uses a pre-test health script (<code>scripts/check-crossplane-health.sh</code>) to gate everything else. The quality bar is simple: <strong>stable Crossplane pods, Healthy providers/functions, and reliable webhook behavior</strong>.</p>

<p>Optional (but powerful): run <strong>Uptest</strong> as a fast provider/credential smoke test (think “verify the screwdriver works” before you build the whole bookshelf).</p>

<h2>
  
  
  Layer 2 — Crossview visual inspection (use the exploded diagram)
</h2>

<p>When something is off, you want the “exploded view” that shows how everything connects:</p>

<ul>
<li>did your XR select the intended Composition?</li>
<li>which managed resources were created?</li>
<li>which condition/event explains why the XR isn’t Ready?</li>
</ul>

<p>Crossview is great here because it visualizes the XR → composed resources graph. Use it as the interactive debugger between Layers 1 and 3.</p>

<h2>
  
  
  Layer 3 — In-cluster E2E with KUTTL (the shake test)
</h2>

<p>Now we let Kubernetes do the real assembly: create the XR, watch reconciliation, assert readiness, and ensure cleanup works.</p>

<p>The demo’s PostgreSQL test case builds up like this:</p>

<ul>
<li>
<strong>00</strong>: create a password <code>Secret</code> + create the XR</li>
<li>
<strong>00 assert</strong>: wait for XR <code>Synced=True</code> and <code>Ready=True</code>
</li>
<li>
<strong>01 assert</strong>: wait for composed managed resources to become <code>Ready=True</code>
</li>
<li>
<strong>01 verify (Azure)</strong>: query Azure to ensure the server + database exist</li>
<li>
<strong>02 delete</strong>: delete the XR</li>
<li>
<strong>02 assert</strong>: confirm XR is gone</li>
</ul>

<p>Representative KUTTL steps (excerpts from <code>tests/e2e/postgresql-databases/basic/</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># 00-secret.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Secret</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">postgres-admin-password</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">type</span><span class="pi">:</span> <span class="s">Opaque</span>
<span class="na">stringData</span><span class="pi">:</span>
  <span class="c1"># Demo-only password for e2e tests (Azure enforces complexity rules).</span>
  <span class="na">password</span><span class="pi">:</span> <span class="s2">"</span><span class="s">P@ssw0rd1234!"</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># 00-xr-postgres.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.io/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLDatabase</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">test-postgres-e2e-001</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">default</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">crossplane</span><span class="pi">:</span>
    <span class="na">compositionSelector</span><span class="pi">:</span>
      <span class="na">matchLabels</span><span class="pi">:</span>
        <span class="na">provider</span><span class="pi">:</span> <span class="s">azure</span>
        <span class="na">type</span><span class="pi">:</span> <span class="s">standard</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">location</span><span class="pi">:</span> <span class="s">westeurope</span>
    <span class="na">resourceGroupName</span><span class="pi">:</span> <span class="s">crossplane-e2e-test-rg</span>
    <span class="na">databaseName</span><span class="pi">:</span> <span class="s">appdb</span>
    <span class="na">adminUsername</span><span class="pi">:</span> <span class="s">pgadmin</span>
    <span class="na">adminPasswordSecretName</span><span class="pi">:</span> <span class="s">postgres-admin-password</span>
    <span class="na">adminPasswordSecretKey</span><span class="pi">:</span> <span class="s">password</span>
    <span class="na">postgresVersion</span><span class="pi">:</span> <span class="s2">"</span><span class="s">16"</span>
    <span class="na">skuName</span><span class="pi">:</span> <span class="s">B_Standard_B1ms</span>
    <span class="na">storageMb</span><span class="pi">:</span> <span class="m">32768</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># 00-assert.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">kuttl.dev/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">TestAssert</span>
<span class="na">timeout</span><span class="pi">:</span> <span class="m">2400</span>
<span class="na">commands</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">kubectl wait -n default xpostgresqldatabase test-postgres-e2e-001 --for=condition=Synced --timeout=2400s</span>
    <span class="s">kubectl wait -n default xpostgresqldatabase test-postgres-e2e-001 --for=condition=Ready --timeout=2400s</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># 01-verify-azure.yaml (excerpt)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">kuttl.dev/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">TestAssert</span>
<span class="na">commands</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">SERVER_NAME=$(kubectl get -n default xpostgresqldatabase test-postgres-e2e-001 -o jsonpath='{.status.serverName}')</span>
    <span class="s">DB_NAME=$(kubectl get -n default xpostgresqldatabase test-postgres-e2e-001 -o jsonpath='{.status.databaseName}')</span>
    <span class="s">az postgres flexible-server show --resource-group crossplane-e2e-test-rg --name "$SERVER_NAME" --output none</span>
    <span class="s">az postgres flexible-server db show --resource-group crossplane-e2e-test-rg --server-name "$SERVER_NAME" --database-name "$DB_NAME" --output none</span>
</code></pre>

</div>



<p>KUTTL suite config (so you can run everything consistently):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">kuttl.dev/v1beta1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">TestSuite</span>
<span class="na">timeout</span><span class="pi">:</span> <span class="m">2400</span>
<span class="na">parallel</span><span class="pi">:</span> <span class="m">1</span>
<span class="na">startKIND</span><span class="pi">:</span> <span class="kc">false</span>
<span class="na">testDirs</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="s">./tests/e2e/postgresql-databases</span>
</code></pre>

</div>



<p>Run it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl kuttl <span class="nb">test</span> <span class="se">\</span>
  <span class="nt">--config</span> tests/e2e/kuttl-test.yaml <span class="se">\</span>
  <span class="nt">--timeout</span> 2400 <span class="se">\</span>
  <span class="nt">--start-kind</span><span class="o">=</span><span class="nb">false</span>
</code></pre>

</div>



<p>If your tests don’t include cleanup, they’re not end-to-end—they’re “create-to-end”.</p>

<h2>
  
  
  Layer 4 — Cloud-side verification (confirm it works in the real world)
</h2>

<p>Kubernetes conditions are necessary, but the cloud control plane is the source of truth.</p>

<p>The demo’s E2E suite uses Azure CLI checks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">SERVER_NAME</span><span class="o">=</span><span class="si">$(</span>kubectl get <span class="nt">-n</span> default xpostgresqldatabase test-postgres-e2e-001 <span class="se">\</span>
  <span class="nt">-o</span> <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.serverName}'</span><span class="si">)</span>

<span class="nv">DB_NAME</span><span class="o">=</span><span class="si">$(</span>kubectl get <span class="nt">-n</span> default xpostgresqldatabase test-postgres-e2e-001 <span class="se">\</span>
  <span class="nt">-o</span> <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.databaseName}'</span><span class="si">)</span>

az postgres flexible-server show <span class="se">\</span>
  <span class="nt">--resource-group</span> crossplane-e2e-test-rg <span class="se">\</span>
  <span class="nt">--name</span> <span class="s2">"</span><span class="nv">$SERVER_NAME</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">--output</span> none

az postgres flexible-server db show <span class="se">\</span>
  <span class="nt">--resource-group</span> crossplane-e2e-test-rg <span class="se">\</span>
  <span class="nt">--server-name</span> <span class="s2">"</span><span class="nv">$SERVER_NAME</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">--database-name</span> <span class="s2">"</span><span class="nv">$DB_NAME</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">--output</span> none
</code></pre>

</div>



<p>This catches issues like:</p>

<ul>
<li>Azure name constraints</li>
<li>subscription provider registration gaps (e.g., <code>Microsoft.DBforPostgreSQL</code>)</li>
<li>resources that exist but don’t match intent (location/SKU/tags)</li>
</ul>

<h2>
  
  
  Layer 5 — GitOps with Flux + Headlamp (keep it assembled over time)
</h2>

<p>Layer 5 answers a different question: “Can we deliver and operate this platform continuously from Git?”</p>

<p>In the demo, Flux is configured to reconcile the Crossplane APIs from the repo:</p>

<ul>
<li>
<code>GitRepository</code>: <code>crossplane-configs</code> (namespace <code>flux-system</code>)</li>
<li>
<code>Kustomization</code>: <code>crossplane-apis</code> (namespace <code>flux-system</code>)</li>
</ul>

<p>Then you run an explicit “proof” test (Step 16.1 in the demo):</p>

<ul>
<li>
<strong>Option A (config-only, safest)</strong>: change a label in the PostgreSQL <code>Composition</code>, commit/push, confirm the label appears on the in-cluster <code>Composition</code>.</li>
<li>
<strong>Option B (proves Crossplane reconciliation)</strong>: add a tag to the composed <code>ResourceGroup</code> base in the Composition, then confirm:

<ul>
<li>the composed managed resource reflects it</li>
<li>optionally, Azure shows it</li>
</ul>


</li>

</ul>

<p>Reconciling on-demand:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flux reconcile <span class="nb">source </span>git crossplane-configs
flux reconcile kustomization crossplane-apis <span class="nt">--with-source</span>
</code></pre>

</div>



<p>Verifying the GitOps flow in-cluster:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Confirm your committed label is now on the Composition</span>
kubectl get composition xpostgresqldatabases.database.example.io <span class="se">\</span>
  <span class="nt">-o</span> <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.metadata.labels.gitops-test}'</span><span class="p">;</span> <span class="nb">echo</span>
</code></pre>

</div>



<p>If you did <strong>Option B</strong> (ResourceGroup tag change), verify Crossplane reconciliation (and optionally Azure):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl get <span class="nt">-n</span> default resourcegroups.azure.m.upbound.io <span class="se">\</span>
  <span class="nt">-l</span> crossplane.io/composite<span class="o">=</span>test-postgres-e2e-001 <span class="nt">-o</span> yaml

az group show <span class="nt">--name</span> crossplane-e2e-test-rg <span class="nt">--query</span> <span class="s2">"tags.gitopsTest"</span> <span class="nt">-o</span> tsv
</code></pre>

</div>



<p>Headlamp (with the Flux plugin) is the “ops dashboard” for this layer: it makes it obvious which Source/Kustomization is failing and why.</p>

<h2>
  
  
  The complete picture (full source)
</h2>

<p>All the code referenced here—including the PostgreSQL API package, KUTTL suites, helper scripts, and Flux structure—is in:</p>

<p><strong><a href="https://github.com/software-journey/crossplane-e2e-testing" rel="noopener noreferrer">https://github.com/software-journey/crossplane-e2e-testing</a></strong></p>

<h2>
  
  
  Key takeaways (your assembly summary sheet)
</h2>

<ul>
<li>
<strong>Layer 0 catches the most mistakes fastest</strong>: render before you reconcile.</li>
<li>
<strong>Layer 1 prevents noisy failures</strong>: don’t trust E2E results from an unhealthy cluster.</li>
<li>
<strong>Layer 2 shortens debugging</strong>: visualize the XR → managed resource graph.</li>
<li>
<strong>Layer 3 proves lifecycle correctness</strong>: create, assert, verify, delete.</li>
<li>
<strong>Layer 4 closes the loop</strong>: validate cloud reality, not just Kubernetes status.</li>
<li>
<strong>Layer 5 makes it operable</strong>: Git → Flux → Kubernetes → Crossplane, continuously.</li>
</ul>




<p><strong>About the Author</strong>: I'm Willem, a Cloud Engineer transitioning to platform engineering. I believe complex infrastructure concepts should be accessible to everyone—even if it means comparing them to Swedish furniture.</p>

