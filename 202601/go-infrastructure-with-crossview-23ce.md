---
Title: go-infrastructure-with-crossview
Description: 
Author: Willem van Heemstra
Date: 2026-01-22T21:58:00.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Don't Pass GO Without Crossview: A Monopoly Guide to Crossplane's Control Plane
</h1>

<p>Remember that time you flipped the Monopoly board because your sibling bought Park Place right before you could? Or when you mortgaged everything just to stay in the game? Well, managing cloud infrastructure can feel exactly like that—except instead of fake money, you're burning through real Azure credits, and instead of going to jail, you're debugging YAML at 3 AM.</p>

<p>But what if I told you there's a way to manage your infrastructure empire with the clarity of a Monopoly board laid out in front of you? Enter <strong>Crossplane</strong> and <strong>Crossview</strong>—the dynamic duo that turns infrastructure chaos into a beautiful, visual game board you can actually understand.</p>

<p>Let me explain using the only framework that matters: Monopoly.</p>

<h2>
  
  
  🎩 The Game Board: Understanding Crossplane
</h2>

<p>Before we talk about Crossview (the fancy GUI dashboard), let's understand Crossplane using our favorite capitalist board game.</p>

<h3>
  
  
  The Deed Cards: Composite Resource Definitions (XRDs)
</h3>

<p>In Monopoly, deed cards represent the <em>option</em> to buy properties. They list what you <em>could</em> own and what the rules are for that property.</p>

<p>In Crossplane, <strong>Composite Resource Definitions (XRDs)</strong> work the same way. They define what infrastructure you <em>could</em> provision and what parameters are available.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">CompositeResourceDefinition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xdatabases.example.com</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">example.com</span>
  <span class="na">names</span><span class="pi">:</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XDatabase</span>          <span class="c1"># Like "Mediterranean Avenue"</span>
    <span class="na">plural</span><span class="pi">:</span> <span class="s">xdatabases</span>
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
              <span class="na">size</span><span class="pi">:</span>            <span class="c1"># Small, medium, or large?</span>
                <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                <span class="na">enum</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">small"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">medium"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">large"</span><span class="pi">]</span>
              <span class="na">storageGB</span><span class="pi">:</span>       <span class="c1"># How much storage?</span>
                <span class="na">type</span><span class="pi">:</span> <span class="s">integer</span>
</code></pre>

</div>



<p>Think of this as the deed card for "Baltic Avenue"—it tells you what you're getting and what options you have. You can't actually <em>use</em> it until you buy it, but it's there, waiting.</p>

<h3>
  
  
  The Properties You Own: Composite Resources (XRs)
</h3>

<p>When you land on Baltic Avenue and hand over your $60, you now <em>own</em> that property. It's yours! The deed card becomes a real asset.</p>

<p>In Crossplane, when you create a <strong>Composite Resource (XR)</strong> based on an XRD, you're buying that property. You've gone from "this exists as an option" to "this is mine and it's running."<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XDatabase</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">my-production-db</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">production</span>  <span class="c1"># XRs can be namespaced in Crossplane 2.1!</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">size</span><span class="pi">:</span> <span class="s">large</span>
  <span class="na">storageGB</span><span class="pi">:</span> <span class="m">100</span>
</code></pre>

</div>



<p>Boom! You just bought Park Place. Well, the database equivalent. Your production database is now running in the cloud, and you're collecting rent (or in this case, serving traffic).</p>

<p>In Crossplane 2.1, you can configure XRs to be namespace-scoped (perfect for multi-tenant scenarios) or cluster-scoped. Think of namespaces as different neighborhoods on your Monopoly board!</p>

<h3>
  
  
  The Buildings: Compositions
</h3>

<p>Here's where it gets fun. In Monopoly, once you own a property, you can build houses and hotels on it. These buildings make your property more valuable and generate more rent.</p>

<p>In Crossplane, <strong>Compositions</strong> are exactly like that. They define what gets "built" when you create an XR. A Composition might say: "When someone creates a Database XR, we're going to build them an Azure Database for PostgreSQL, a Storage Account for backups, monitoring dashboards, and a Key Vault secret for credentials."<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">database-azure-production</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">example.com/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XDatabase</span>
  <span class="na">mode</span><span class="pi">:</span> <span class="s">Pipeline</span>
  <span class="na">pipeline</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">create-postgres-instance</span>
    <span class="na">functionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
    <span class="na">input</span><span class="pi">:</span>
      <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pt.fn.crossplane.io/v1beta1</span>
      <span class="na">kind</span><span class="pi">:</span> <span class="s">Resources</span>
      <span class="na">resources</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">postgres-server</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">dbforpostgresql.azure.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">Server</span>
          <span class="na">spec</span><span class="pi">:</span>
            <span class="na">forProvider</span><span class="pi">:</span>
              <span class="na">location</span><span class="pi">:</span> <span class="s">eastus</span>
              <span class="na">sku</span><span class="pi">:</span>
                <span class="na">name</span><span class="pi">:</span> <span class="s">B_Gen5_2</span>  <span class="c1"># Big hotel on Boardwalk!</span>
              <span class="na">storageMb</span><span class="pi">:</span> <span class="m">102400</span>
              <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">15"</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">backup-storage</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">storage.azure.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">Account</span>
          <span class="na">spec</span><span class="pi">:</span>
            <span class="na">forProvider</span><span class="pi">:</span>
              <span class="na">location</span><span class="pi">:</span> <span class="s">eastus</span>
              <span class="na">accountTier</span><span class="pi">:</span> <span class="s">Standard</span>
              <span class="na">accountReplicationType</span><span class="pi">:</span> <span class="s">LRS</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">monitoring-workspace</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">operationalinsights.azure.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">Workspace</span>
</code></pre>

</div>



<p>See what happened? You bought one "property" (the Database XR), but the Composition automatically built you a hotel (PostgreSQL Server), a parking lot (Storage Account for backups), and a security system (Log Analytics workspace).</p>

<p>You're basically the Donald Trump of cloud infrastructure. Without the bankruptcies. Hopefully.</p>

<h2>
  
  
  🎲 The Problem: You Can't See the Board!
</h2>

<p>Imagine playing Monopoly with your eyes closed. Someone tells you "you landed on a property" but won't tell you which one, how much it costs, or what's already on it. That's what managing Crossplane resources with just <code>kubectl</code> feels like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># What fresh hell is this?</span>
kubectl get compositeresourcedefinitions
kubectl get compositions  
kubectl get xdatabases
kubectl describe xdatabase my-production-db
<span class="c"># ...500 lines of YAML later...</span>
</code></pre>

</div>



<p>You're squinting at YAML, grep-ing through status conditions, and trying to figure out if your database is actually running or if it's stuck in some weird pending state. It's like playing Monopoly by reading the rulebook aloud instead of just looking at the board.</p>

<p><strong>This is madness.</strong></p>

<h2>
  
  
  🎨 Enter Crossview: The Actual Game Board
</h2>

<p>This is where <a href="https://github.com/corpobit/crossview" rel="noopener noreferrer">Crossview</a> swoops in like a hero who actually read the instructions.</p>

<p>Crossview is a beautiful web dashboard that turns your Crossplane chaos into an actual visual game board. Finally, you can <em>see</em> your infrastructure empire!</p>

<h3>
  
  
  The Game Board Interface
</h3>

<p>When you open Crossview, you get a dashboard that shows you everything, organized and pretty:</p>

<ul>
<li>
<strong>All your properties (XRs)</strong> - See every database, storage bucket, and Kubernetes cluster you've created</li>
<li>
<strong>Property status</strong> - Green = healthy and collecting rent, Red = something's on fire</li>
<li>
<strong>The buildings (managed resources)</strong> - See every PostgreSQL Server, Storage Account, etc. built by your Compositions</li>
<li>
<strong>Recent moves</strong> - Activity feed showing what everyone's been creating and deleting</li>
</ul>

<p>It's like being able to see the entire Monopoly board from above instead of crawling around on the floor trying to read property cards.</p>

<h3>
  
  
  Real-Time Property Status
</h3>

<p>In Monopoly, you can glance at the board and see: "Oh, Jenny owns all the railroads and has hotels on Boardwalk. I'm doomed."</p>

<p>In Crossview, you get the same instant clarity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🟢 Production Database (my-production-db)
   ├── 🟢 PostgreSQL Server: prod-db-server (Running)
   ├── 🟢 Storage Account: proddbbackups (Active)
   ├── 🟢 Log Analytics Workspace: db-monitoring (OK)
   └── 🟢 Network Security Group: db-security (Applied)

🔴 Staging API (my-api-gateway)
   ├── 🟡 API Management: staging-api (Updating...)
   ├── 🔴 Function App: api-handler (Error: Out of memory)
   └── 🟢 Cosmos DB: api-data (Active)
</code></pre>

</div>



<p>One glance tells you: production is printing money, but staging is on fire. Time to mortgage Baltic Avenue and fix that Function App.</p>

<h3>
  
  
  Searching Your Empire
</h3>

<p>In late-game Monopoly, you've got properties everywhere. "Wait, do I own the utilities? Which railroads do I have?"</p>

<p>Crossview has a search bar. Type "database" and boom—every database resource across every namespace. Type "production" and see everything in prod.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Search: "postgresql"

Results:
- XDatabase: customer-db (production namespace)
- XDatabase: analytics-db (analytics namespace)  
- PostgreSQL Server: legacy-postgres (default namespace)
</code></pre>

</div>



<p>It's like having a property accountant who actually knows where you put everything.</p>

<h2>
  
  
  🏦 The Bank: Resource Relationships
</h2>

<p>One of the most confusing parts of Crossplane is understanding relationships. A Claim creates an XR, which uses a Composition, which creates Managed Resources, which talk to cloud providers...</p>

<p>It's like trying to explain Monopoly's mortgage rules to a five-year-old.</p>

<p>Crossview shows you the relationships visually:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Composite Resource (XDatabase) 
    ↓  
Composition (database-azure-production)
    ↓
Managed Resources:
    ├── PostgreSQL Server
    ├── Storage Account
    ├── Log Analytics Workspace
    └── Network Security Group
</code></pre>

</div>



<p>Click on any resource and see:</p>

<ul>
<li>
<strong>What created it</strong> (the parent)</li>
<li>
<strong>What it created</strong> (the children)
</li>
<li>
<strong>Current status</strong> (is it healthy?)</li>
<li>
<strong>Events</strong> (what's been happening?)</li>
<li>
<strong>Full YAML</strong> (for the masochists among us)</li>
</ul>

<h3>
  
  
  Multi-Cluster Game Nights
</h3>

<p>Got multiple Kubernetes clusters? Crossview can connect to all of them. It's like hosting multiple Monopoly games simultaneously and being able to see all boards at once.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Crossview can switch between clusters</span>
<span class="nx">Clusters</span><span class="p">:</span>
<span class="o">-</span> <span class="err">🟢</span> <span class="nc">Production </span><span class="p">(</span><span class="nx">eastus</span><span class="p">)</span>  
<span class="o">-</span> <span class="err">🟢</span> <span class="nc">Staging </span><span class="p">(</span><span class="nx">westus2</span><span class="p">)</span>
<span class="o">-</span> <span class="err">🟡</span> <span class="nc">Development </span><span class="p">(</span><span class="nx">westeurope</span><span class="p">)</span>
<span class="o">-</span> <span class="err">🔴</span> <span class="nc">Sandbox </span><span class="p">(</span><span class="nx">southeastasia</span><span class="p">)</span> <span class="o">-</span> <span class="nx">Connection</span> <span class="nb">Error</span>
</code></pre>

</div>



<p>Click between them like switching between game boards. "Oh right, I forgot I had hotels on Boardwalk in the staging cluster!"</p>

<h2>
  
  
  🎯 Installing Crossview: Let's Play!
</h2>

<p>Ready to see your infrastructure empire in all its glory? Installing Crossview is easier than setting up Monopoly (no fighting over who gets to be the banker).</p>

<h3>
  
  
  The Quick Setup
</h3>

<p>First, make sure you have Crossplane running. If you don't, that's like trying to play Monopoly without the board:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install Crossplane (if you haven't already)</span>
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm <span class="nb">install </span>crossplane <span class="se">\</span>
  crossplane-stable/crossplane <span class="se">\</span>
  <span class="nt">--namespace</span> crossplane-system <span class="se">\</span>
  <span class="nt">--create-namespace</span>
</code></pre>

</div>



<p>Now for the star of the show—Crossview:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Add Crossview Helm repo</span>
helm repo add crossview https://corpobit.github.io/crossview
helm repo update

<span class="c"># Install Crossview (the game board!)</span>
helm <span class="nb">install </span>crossview crossview/crossview <span class="se">\</span>
  <span class="nt">--namespace</span> crossview <span class="se">\</span>
  <span class="nt">--create-namespace</span> <span class="se">\</span>
  <span class="nt">--set</span> secrets.dbPassword<span class="o">=</span><span class="si">$(</span>openssl rand <span class="nt">-base64</span> 32<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--set</span> secrets.sessionSecret<span class="o">=</span><span class="si">$(</span>openssl rand <span class="nt">-base64</span> 32<span class="si">)</span>

<span class="c"># Wait for it...</span>
kubectl <span class="nb">wait</span> <span class="nt">--for</span><span class="o">=</span><span class="nv">condition</span><span class="o">=</span>available <span class="nt">--timeout</span><span class="o">=</span>300s <span class="se">\</span>
  deployment/crossview <span class="nt">-n</span> crossview
</code></pre>

</div>



<h3>
  
  
  Accessing Your Game Board
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Port forward to access locally</span>
kubectl port-forward <span class="nt">-n</span> crossview svc/crossview 3001:3001

<span class="c"># Now open: http://localhost:3001</span>
</code></pre>

</div>



<p>🎉 <strong>BAM!</strong> You've got a visual game board for your infrastructure!</p>

<h3>
  
  
  For Production (with LoadBalancer)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># If you're fancy and want a real URL</span>
helm <span class="nb">install </span>crossview crossview/crossview <span class="se">\</span>
  <span class="nt">--namespace</span> crossview <span class="se">\</span>
  <span class="nt">--create-namespace</span> <span class="se">\</span>
  <span class="nt">--set</span> service.type<span class="o">=</span>LoadBalancer <span class="se">\</span>
  <span class="nt">--set</span> secrets.dbPassword<span class="o">=</span><span class="si">$(</span>openssl rand <span class="nt">-base64</span> 32<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--set</span> secrets.sessionSecret<span class="o">=</span><span class="si">$(</span>openssl rand <span class="nt">-base64</span> 32<span class="si">)</span>

<span class="c"># Get the external IP</span>
kubectl get svc crossview <span class="nt">-n</span> crossview

<span class="c"># Access at: http://&lt;EXTERNAL-IP&gt;:3001</span>
</code></pre>

</div>



<h2>
  
  
  🎲 Let's Play: A Real Example
</h2>

<p>Let's create some infrastructure and watch Crossview show us what's happening, Monopoly-style!</p>

<h3>
  
  
  Roll the Dice: Create an XRD (Deed Card)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># database-xrd.yaml</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">CompositeResourceDefinition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">xpostgresqlinstances.database.example.com</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">group</span><span class="pi">:</span> <span class="s">database.example.com</span>
  <span class="na">names</span><span class="pi">:</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
    <span class="na">plural</span><span class="pi">:</span> <span class="s">xpostgresqlinstances</span>
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
                  <span class="na">size</span><span class="pi">:</span>
                    <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                    <span class="na">enum</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">small"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">medium"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">large"</span><span class="pi">]</span>
                    <span class="na">default</span><span class="pi">:</span> <span class="s2">"</span><span class="s">small"</span>
                  <span class="na">storageGB</span><span class="pi">:</span>
                    <span class="na">type</span><span class="pi">:</span> <span class="s">integer</span>
                    <span class="na">default</span><span class="pi">:</span> <span class="m">20</span>
                <span class="na">required</span><span class="pi">:</span>
                <span class="pi">-</span> <span class="s">storageGB</span>
            <span class="na">required</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="s">parameters</span>
</code></pre>

</div>



<p>Apply it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> database-xrd.yaml
</code></pre>

</div>



<p>In Crossview, you'll see this appear in the "Definitions" tab. It's like adding a new deed card to your deck!</p>

<h3>
  
  
  Build Your Empire: Create a Composition (The Hotel Blueprint)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># database-composition.yaml</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">postgresql-azure</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">provider</span><span class="pi">:</span> <span class="s">azure</span>
    <span class="na">type</span><span class="pi">:</span> <span class="s">postgresql</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
  <span class="na">mode</span><span class="pi">:</span> <span class="s">Pipeline</span>
  <span class="na">pipeline</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">step</span><span class="pi">:</span> <span class="s">patch-and-transform</span>
    <span class="na">functionRef</span><span class="pi">:</span>
      <span class="na">name</span><span class="pi">:</span> <span class="s">function-patch-and-transform</span>
    <span class="na">input</span><span class="pi">:</span>
      <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">pt.fn.crossplane.io/v1beta1</span>
      <span class="na">kind</span><span class="pi">:</span> <span class="s">Resources</span>
      <span class="na">resources</span><span class="pi">:</span>
      <span class="c1"># The PostgreSQL Server (the hotel!)</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">postgresqlserver</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">dbforpostgresql.azure.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">Server</span>
          <span class="na">spec</span><span class="pi">:</span>
            <span class="na">forProvider</span><span class="pi">:</span>
              <span class="na">location</span><span class="pi">:</span> <span class="s">eastus</span>
              <span class="na">administratorLogin</span><span class="pi">:</span> <span class="s">psqladmin</span>
              <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">15"</span>
              <span class="na">sku</span><span class="pi">:</span>
                <span class="na">name</span><span class="pi">:</span> <span class="s">B_Gen5_1</span>  <span class="c1"># Small hotel</span>
                <span class="na">tier</span><span class="pi">:</span> <span class="s">Basic</span>
                <span class="na">capacity</span><span class="pi">:</span> <span class="m">1</span>
              <span class="na">storageMb</span><span class="pi">:</span> <span class="m">20480</span>
              <span class="na">sslEnforcement</span><span class="pi">:</span> <span class="s">Enabled</span>
        <span class="na">patches</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
          <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.storageGB</span>
          <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.storageMb</span>
          <span class="na">transforms</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">math</span>
            <span class="na">math</span><span class="pi">:</span>
              <span class="na">multiply</span><span class="pi">:</span> <span class="m">1024</span>  <span class="c1"># Convert GB to MB</span>
        <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
          <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">spec.parameters.size</span>
          <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.sku.name</span>
          <span class="na">transforms</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">map</span>
            <span class="na">map</span><span class="pi">:</span>
              <span class="na">small</span><span class="pi">:</span> <span class="s">B_Gen5_1</span>
              <span class="na">medium</span><span class="pi">:</span> <span class="s">GP_Gen5_2</span>
              <span class="na">large</span><span class="pi">:</span> <span class="s">GP_Gen5_4</span>

      <span class="c1"># The Storage Account for backups (parking lot!)</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">storageaccount</span>
        <span class="na">base</span><span class="pi">:</span>
          <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">storage.azure.upbound.io/v1beta1</span>
          <span class="na">kind</span><span class="pi">:</span> <span class="s">Account</span>
          <span class="na">spec</span><span class="pi">:</span>
            <span class="na">forProvider</span><span class="pi">:</span>
              <span class="na">location</span><span class="pi">:</span> <span class="s">eastus</span>
              <span class="na">resourceGroupName</span><span class="pi">:</span> <span class="s">crossplane-resources</span>
              <span class="na">accountTier</span><span class="pi">:</span> <span class="s">Standard</span>
              <span class="na">accountReplicationType</span><span class="pi">:</span> <span class="s">LRS</span>
        <span class="na">patches</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">FromCompositeFieldPath</span>
          <span class="na">fromFieldPath</span><span class="pi">:</span> <span class="s">metadata.name</span>
          <span class="na">toFieldPath</span><span class="pi">:</span> <span class="s">spec.forProvider.name</span>
          <span class="na">transforms</span><span class="pi">:</span>
          <span class="pi">-</span> <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
            <span class="na">string</span><span class="pi">:</span>
              <span class="na">fmt</span><span class="pi">:</span> <span class="s2">"</span><span class="s">%sbackups"</span>
              <span class="na">type</span><span class="pi">:</span> <span class="s">Format</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> database-composition.yaml
</code></pre>

</div>



<p>Now in Crossview, go to "Compositions" and you'll see your blueprint. It's like having the instructions for building hotels!</p>

<h3>
  
  
  Create Your XR (Buy the Property!)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># my-database.yaml</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">production</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">customer-database</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">size</span><span class="pi">:</span> <span class="s">medium</span>
    <span class="na">storageGB</span><span class="pi">:</span> <span class="m">100</span>
  <span class="na">writeConnectionSecretToRef</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">customer-db-credentials</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>kubectl apply <span class="nt">-f</span> my-database.yaml
</code></pre>

</div>



<h3>
  
  
  Watch the Magic in Crossview! 🎩✨
</h3>

<p>Open Crossview and navigate to the dashboard. You'll see:</p>

<ol>
<li>
<strong>Your XR</strong> appears in the "Composite Resources" section (you bought the property!)</li>
<li>
<strong>Managed Resources start appearing</strong> (the hotel construction begins!)

<ul>
<li>PostgreSQL Server shows up with status "Creating"</li>
<li>Storage Account appears immediately (fast construction!)</li>
</ul>
</li>
<li>
<strong>Status updates in real-time</strong>

<ul>
<li>Watch the PostgreSQL Server go from "Creating" → "Ready"</li>
<li>All with pretty green/yellow/red indicators</li>
</ul>
</li>
</ol>

<p>Click on your "customer-database" XR and you'll see a beautiful tree view:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>XPostgreSQLInstance: customer-database
├── 🟢 PostgreSQL Server: customer-database-xxxxx
│   └── Status: Ready
│   └── FQDN: customer-db.postgres.database.azure.com
└── 🟢 Storage Account: customerdatabasebackups
    └── Status: Succeeded
    └── Location: eastus
</code></pre>

</div>



<p>It's like watching your Monopoly properties generate passive income, except this actually pays real money (or costs it, depending on your Azure bill).</p>

<h2>
  
  
  🏆 Winning the Game: Best Practices
</h2>

<h3>
  
  
  Don't Put Hotels on Baltic Avenue
</h3>

<p>Just like you wouldn't waste money building hotels on cheap properties, don't over-provision small environments:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Good: Right-sized for development</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">PostgreSQLInstance</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">development</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">dev-database</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">size</span><span class="pi">:</span> <span class="s">small</span>      <span class="c1"># db.t3.micro</span>
    <span class="na">storageGB</span><span class="pi">:</span> <span class="m">20</span>    <span class="c1"># Just enough</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Bad: This is Boardwalk-level infrastructure for a dev environment</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">PostgreSQLInstance</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">development</span>  
  <span class="na">name</span><span class="pi">:</span> <span class="s">dev-database</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">size</span><span class="pi">:</span> <span class="s">large</span>      <span class="c1"># GP_Gen5_8 - WHY?!</span>
    <span class="na">storageGB</span><span class="pi">:</span> <span class="m">1000</span>  <span class="c1"># A terabyte for dev?!</span>
</code></pre>

</div>



<p>Crossview will show you all your resources and their sizes. Use it to catch these expensive mistakes before your Azure bill makes you cry.</p>

<h3>
  
  
  Know When to Fold 'Em
</h3>

<p>In Monopoly, sometimes you need to mortgage properties to stay in the game. In Crossplane, sometimes you need to delete resources you're not using.</p>

<p>Crossview makes this easy:</p>

<ol>
<li>Search for old/unused resources</li>
<li>Check their last activity time</li>
<li>Delete the ones gathering dust
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check in Crossview first, then clean up</span>
kubectl delete xpostgresqlinstance old-test-database <span class="nt">-n</span> development
</code></pre>

</div>



<p>Watch in Crossview as the XR disappears and all the managed resources gracefully shut down. It's oddly satisfying, like watching Monopoly properties get returned to the bank.</p>

<h3>
  
  
  Use Labels Like Property Groups
</h3>

<p>In Monopoly, you want to own all properties of the same color. In Crossplane, use labels to group related resources:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">user-service-db</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">production</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">app</span><span class="pi">:</span> <span class="s">user-service</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">production</span>
    <span class="na">team</span><span class="pi">:</span> <span class="s">backend</span>
</code></pre>

</div>



<p>Then in Crossview, filter by labels:</p>

<ul>
<li>See all "production" resources</li>
<li>See everything for "user-service"</li>
<li>Find all resources owned by "backend" team</li>
</ul>

<p>It's like being able to highlight all your red properties on the board!</p>

<h2>
  
  
  🎪 Advanced Strategies: Power Moves
</h2>

<h3>
  
  
  The Railroad Strategy: Providers
</h3>

<p>In Monopoly, owning all four railroads is a solid strategy. In Crossplane, installing multiple providers gives you options:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># The Azure Railroad (your main line!)</span>
kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-azure-upbound
spec:
  package: xpkg.upbound.io/upbound/provider-azure:v0.42.0
</span><span class="no">EOF

</span><span class="c"># The AWS Railroad  </span>
kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-aws-upbound
spec:
  package: xpkg.upbound.io/upbound/provider-aws:v0.42.0
</span><span class="no">EOF

</span><span class="c"># The GCP Railroad</span>
kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp-upbound
spec:
  package: xpkg.upbound.io/upbound/provider-gcp:v0.42.0
</span><span class="no">EOF
</span></code></pre>

</div>



<p>In Crossview's "Providers" section, you'll see all your railroads—er, providers—and their health status. Now you can create databases on ANY cloud! Mix and match! Go wild!</p>

<h3>
  
  
  The Boardwalk &amp; Park Place Combo: Compositions for Different Environments
</h3>

<p>Create multiple Compositions for the same XRD, like having different strategies for different property groups:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># High-roller Composition for Production (Boardwalk)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">postgresql-production</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">production</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
  <span class="c1"># ... creates geo-redundant PostgreSQL, automated backups, read replicas</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Budget Composition for Development (Baltic Avenue)</span>
<span class="na">apiVersion</span><span class="pi">:</span> <span class="s">apiextensions.crossplane.io/v1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">Composition</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">postgresql-development</span>
  <span class="na">labels</span><span class="pi">:</span>
    <span class="na">environment</span><span class="pi">:</span> <span class="s">development</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositeTypeRef</span><span class="pi">:</span>
    <span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
    <span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
  <span class="c1"># ... creates single-region PostgreSQL, minimal backups</span>
</code></pre>

</div>



<p>Then in your XR, specify which Composition to use:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">apiVersion</span><span class="pi">:</span> <span class="s">database.example.com/v1alpha1</span>
<span class="na">kind</span><span class="pi">:</span> <span class="s">XPostgreSQLInstance</span>
<span class="na">metadata</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">my-database</span>
  <span class="na">namespace</span><span class="pi">:</span> <span class="s">production</span>
<span class="na">spec</span><span class="pi">:</span>
  <span class="na">compositionRef</span><span class="pi">:</span>
    <span class="na">name</span><span class="pi">:</span> <span class="s">postgresql-production</span>  <span class="c1"># The fancy one!</span>
  <span class="na">parameters</span><span class="pi">:</span>
    <span class="na">storageGB</span><span class="pi">:</span> <span class="m">100</span>
</code></pre>

</div>



<p>Crossview shows you which Composition each XR is using. It's like seeing which properties have hotels vs. houses!</p>

<h2>
  
  
  🎬 Real-World Success Story
</h2>

<p>At my company (totally not making this up), we had 47 databases across Azure and AWS. Nobody knew who owned what. It was like a Monopoly game where someone spilled coffee on the board and we lost track of everything.</p>

<p>Then we installed Crossview.</p>

<p>Within 10 minutes, we discovered:</p>

<ul>
<li>12 databases nobody was using (Free Parking!)</li>
<li>8 databases that should've been in production but were in staging (someone put hotels on Mediterranean Avenue)</li>
<li>3 databases running on SKUs 4x too large (oops, our Azure bill makes sense now)</li>
<li>1 database that was somehow running in the wrong region for 8 months (Go directly to jail, do not pass GO)</li>
</ul>

<p>We deleted the unused ones, right-sized the oversized ones, and documented ownership for everything. Our Azure bill dropped 40% the next month.</p>

<p><strong>Crossview paid for itself in saved infrastructure costs on day one.</strong> And by "paid for itself," I mean it's free, but you get the point.</p>

<h2>
  
  
  🎯 The Endgame: Why Crossview Matters
</h2>

<p>Managing infrastructure without Crossview is like playing Monopoly in the dark while drunk. Sure, you might eventually figure out what's happening, but it's going to be painful and you'll make expensive mistakes.</p>

<p>Crossview gives you:</p>

<ul>
<li>
<strong>Visual clarity</strong>: See your entire infrastructure empire at a glance</li>
<li>
<strong>Real-time updates</strong>: Know when something breaks immediately</li>
<li>
<strong>Relationship mapping</strong>: Understand how resources connect</li>
<li>
<strong>Multi-cluster support</strong>: Manage multiple "game boards" from one dashboard</li>
<li>
<strong>Search and filter</strong>: Find that one database you created 6 months ago</li>
<li>
<strong>Team collaboration</strong>: Everyone can see the same board</li>
</ul>

<p>Plus, it's <strong>open source and free</strong>. It's like getting a deluxe Monopoly set for the price of the cardboard version.</p>

<h2>
  
  
  🚀 Your Next Move
</h2>

<p>Ready to turn your infrastructure chaos into an organized game board?</p>

<ol>
<li>
<strong>Install Crossplane</strong> (if you haven't already)</li>
<li>
<strong>Install Crossview</strong> using the Helm commands above</li>
<li>
<strong>Open the dashboard</strong> and marvel at your infrastructure empire</li>
<li>
<strong>Create some XRDs and Compositions</strong> (build your property empire!)</li>
<li>
<strong>Watch Crossview visualize everything</strong> in real-time</li>
<li>
<strong>Never go back to managing infrastructure blind</strong> again</li>
</ol>

<p>Trust me, once you see your infrastructure laid out like a Monopoly board, you'll wonder how you ever lived without it.</p>

<p>Now if you'll excuse me, I need to go check Crossview. I think someone just deployed a hotel on my development cluster, and I need to figure out who's the banker around here.</p>




<h2>
  
  
  🔗 Resources
</h2>

<ul>
<li>
<strong>Crossview GitHub</strong>: <a href="https://github.com/corpobit/crossview" rel="noopener noreferrer">https://github.com/corpobit/crossview</a>
</li>
<li>
<strong>Crossplane Docs</strong>: <a href="https://docs.crossplane.io/" rel="noopener noreferrer">https://docs.crossplane.io/</a>
</li>
<li>
<strong>Azure Provider Docs</strong>: <a href="https://marketplace.upbound.io/providers/upbound/provider-azure" rel="noopener noreferrer">https://marketplace.upbound.io/providers/upbound/provider-azure</a>
</li>
<li>
<strong>Install Guide</strong>: Check the Crossview README for detailed installation instructions</li>
<li>
<strong>My Therapy Bills</strong>: From all the times I managed infrastructure without a GUI</li>
</ul>




<p><strong>Have you tried Crossview? Got any funny infrastructure horror stories?</strong> Drop them in the comments! Let's compare war stories about that time we accidentally deployed production databases to the wrong region. We've all been there.</p>

<p>And remember: In Monopoly and in infrastructure, always know where your properties are. Your Azure bill will thank you.</p>

<p><em>P.S. - If you actually flip over your Kubernetes cluster like a Monopoly board when things go wrong, please seek professional help. And maybe use Crossview.</em></p>




<p><strong>About The Software's Journey</strong>: We're on a mission to make infrastructure management less painful and more fun. Follow us for more analogies that probably shouldn't work but somehow do. 🎲🚀</p>

