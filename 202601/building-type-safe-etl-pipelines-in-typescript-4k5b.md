---
Title: Building Type-Safe ETL Pipelines in TypeScript
Description: 
Author: Paul H
Date: 2026-01-24T21:42:13.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>A practical guide to implementing Extract, Transform, Load workflows with full type safety</strong></p>




<h2>
  
  
  Introduction
</h2>

<p>ETL (Extract, Transform, Load) is a fundamental pattern in data engineering. Whether you're syncing CRM data to a warehouse, migrating between databases, or integrating third-party APIs, ETL pipelines are everywhere. But building them in a type-safe, maintainable way has traditionally been challenging.</p>

<p>In this article, we'll explore how to implement ETL pipelines in TypeScript using OpenETL, a modern framework that brings type safety and abstraction to data integration.</p>

<h2>
  
  
  What is ETL?
</h2>

<p>ETL consists of three phases:</p>

<ol>
<li>
<strong>Extract</strong>: Pull data from a source (database, API, file)</li>
<li>
<strong>Transform</strong>: Clean, reshape, or enrich the data</li>
<li>
<strong>Load</strong>: Write the data to a destination
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────┐    ┌───────────┐    ┌─────────┐
│ Extract │ ── │ Transform │ ── │  Load   │
└─────────┘    └───────────┘    └─────────┘
    │                │               │
    ▼                ▼               ▼
 Database         Rename           Data
   API            Filter         Warehouse
   File           Merge             API
</code></pre>

</div>



<h2>
  
  
  The Challenge with Traditional ETL
</h2>

<p>Traditional ETL implementations often suffer from:</p>

<ul>
<li>
<strong>Tight coupling</strong>: Source-specific code mixed with business logic</li>
<li>
<strong>No type safety</strong>: Runtime errors from schema mismatches</li>
<li>
<strong>Code duplication</strong>: Similar patterns repeated for each integration</li>
<li>
<strong>Hard to test</strong>: Database connections embedded in business logic</li>
</ul>

<h2>
  
  
  The Adapter Pattern Solution
</h2>

<p>OpenETL solves these problems with the adapter pattern. Each data source implements a common interface, allowing you to swap sources without changing your pipeline logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// The adapter interface - all sources implement this</span>
<span class="kr">interface</span> <span class="nx">AdapterInstance</span> <span class="p">{</span>
  <span class="nf">connect</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nf">disconnect</span><span class="p">():</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nf">download</span><span class="p">(</span><span class="nx">options</span><span class="p">:</span> <span class="nx">PageOptions</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="p">}</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nf">upload</span><span class="p">(</span><span class="nx">data</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This abstraction means your pipeline code doesn't care whether data comes from PostgreSQL, MongoDB, or a REST API.</p>

<h2>
  
  
  Getting Started
</h2>

<h3>
  
  
  Installation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>openetl
npm <span class="nb">install</span> @openetl/postgresql  <span class="c"># or any adapter you need</span>
</code></pre>

</div>



<h3>
  
  
  Basic Pipeline
</h3>

<p>Here's a simple pipeline that downloads data from PostgreSQL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Orchestrator</span><span class="p">,</span> <span class="nx">Vault</span><span class="p">,</span> <span class="nx">Pipeline</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">openetl</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">postgresql</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@openetl/postgresql</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// 1. Define credentials in a vault</span>
<span class="kd">const</span> <span class="nx">vault</span><span class="p">:</span> <span class="nx">Vault</span> <span class="o">=</span> <span class="p">{</span>
  <span class="dl">'</span><span class="s1">my-database</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">my-database</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">basic</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">credentials</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">host</span><span class="p">:</span> <span class="dl">'</span><span class="s1">localhost</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">port</span><span class="p">:</span> <span class="dl">'</span><span class="s1">5432</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">database</span><span class="p">:</span> <span class="dl">'</span><span class="s1">myapp</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">username</span><span class="p">:</span> <span class="dl">'</span><span class="s1">user</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">password</span><span class="p">:</span> <span class="dl">'</span><span class="s1">secret</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="c1">// 2. Register adapters</span>
<span class="kd">const</span> <span class="nx">adapters</span> <span class="o">=</span> <span class="p">{</span> <span class="nx">postgresql</span> <span class="p">};</span>

<span class="c1">// 3. Define the pipeline</span>
<span class="kd">const</span> <span class="nx">pipeline</span><span class="p">:</span> <span class="nx">Pipeline</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">export-users</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">source</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">source</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">postgresql</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">table_query</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">credential_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">my-database</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">fields</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">name</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">created_at</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">config</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">schema</span><span class="p">:</span> <span class="dl">'</span><span class="s1">public</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">table</span><span class="p">:</span> <span class="dl">'</span><span class="s1">users</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="c1">// 4. Execute</span>
<span class="kd">const</span> <span class="nx">etl</span> <span class="o">=</span> <span class="nc">Orchestrator</span><span class="p">(</span><span class="nx">vault</span><span class="p">,</span> <span class="nx">adapters</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">etl</span><span class="p">.</span><span class="nf">runPipeline</span><span class="p">(</span><span class="nx">pipeline</span><span class="p">);</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Exported </span><span class="p">${</span><span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> users`</span><span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  Working with Multiple Data Sources
</h2>

<p>The power of abstraction becomes clear when working with multiple sources. The same pipeline structure works regardless of the source:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">postgresql</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@openetl/postgresql</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">mysql</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@openetl/mysql</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">mongodb</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@openetl/mongodb</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">hubspot</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@openetl/hubspot</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">adapters</span> <span class="o">=</span> <span class="p">{</span> <span class="nx">postgresql</span><span class="p">,</span> <span class="nx">mysql</span><span class="p">,</span> <span class="nx">mongodb</span><span class="p">,</span> <span class="nx">hubspot</span> <span class="p">};</span>

<span class="c1">// PostgreSQL source</span>
<span class="kd">const</span> <span class="nx">postgresSource</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">postgresql</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">table_query</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">config</span><span class="p">:</span> <span class="p">{</span> <span class="na">schema</span><span class="p">:</span> <span class="dl">'</span><span class="s1">public</span><span class="dl">'</span><span class="p">,</span> <span class="na">table</span><span class="p">:</span> <span class="dl">'</span><span class="s1">customers</span><span class="dl">'</span> <span class="p">},</span>
<span class="p">};</span>

<span class="c1">// MySQL source - same structure, different adapter</span>
<span class="kd">const</span> <span class="nx">mysqlSource</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mysql</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">table_query</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">config</span><span class="p">:</span> <span class="p">{</span> <span class="na">database</span><span class="p">:</span> <span class="dl">'</span><span class="s1">sales</span><span class="dl">'</span><span class="p">,</span> <span class="na">table</span><span class="p">:</span> <span class="dl">'</span><span class="s1">customers</span><span class="dl">'</span> <span class="p">},</span>
<span class="p">};</span>

<span class="c1">// MongoDB source - still the same pattern</span>
<span class="kd">const</span> <span class="nx">mongoSource</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">mongodb</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">collection_query</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">config</span><span class="p">:</span> <span class="p">{</span> <span class="na">table</span><span class="p">:</span> <span class="dl">'</span><span class="s1">customers</span><span class="dl">'</span> <span class="p">},</span>
<span class="p">};</span>

<span class="c1">// HubSpot API - REST API, same interface</span>
<span class="kd">const</span> <span class="nx">hubspotSource</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">hubspot</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">contacts</span><span class="dl">'</span><span class="p">,</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  Data Transformations
</h2>

<p>OpenETL includes 12 built-in transformation types. Transformations are applied in sequence during the pipeline execution:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">pipeline</span><span class="p">:</span> <span class="nx">Pipeline</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">transform-contacts</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">source</span><span class="p">:</span> <span class="p">{</span>
    <span class="c1">// ... source config</span>
    <span class="na">transform</span><span class="p">:</span> <span class="p">[</span>
      <span class="c1">// Combine first and last name</span>
      <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">concat</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">options</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">properties</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">first_name</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">last_name</span><span class="dl">'</span><span class="p">],</span>
          <span class="na">glue</span><span class="p">:</span> <span class="dl">'</span><span class="s1"> </span><span class="dl">'</span><span class="p">,</span>
          <span class="na">to</span><span class="p">:</span> <span class="dl">'</span><span class="s1">full_name</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">},</span>
      <span class="c1">// Normalize email to lowercase</span>
      <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">lowercase</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">options</span><span class="p">:</span> <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">},</span>
      <span class="c1">// Remove whitespace</span>
      <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">trim</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">options</span><span class="p">:</span> <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">full_name</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">},</span>
      <span class="c1">// Extract domain from email</span>
      <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">extract</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">options</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">pattern</span><span class="p">:</span> <span class="dl">'</span><span class="s1">@(.+)$</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">to</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email_domain</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">},</span>
    <span class="p">],</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Available Transformations
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>concat</code></td>
<td>Combine multiple fields</td>
<td>
<code>['first', 'last']</code> → <code>'full_name'</code>
</td>
</tr>
<tr>
<td><code>renameKey</code></td>
<td>Copy/rename a field</td>
<td>
<code>'old_field'</code> → <code>'new_field'</code>
</td>
</tr>
<tr>
<td><code>uppercase</code></td>
<td>Convert to uppercase</td>
<td>
<code>'hello'</code> → <code>'HELLO'</code>
</td>
</tr>
<tr>
<td><code>lowercase</code></td>
<td>Convert to lowercase</td>
<td>
<code>'HELLO'</code> → <code>'hello'</code>
</td>
</tr>
<tr>
<td><code>trim</code></td>
<td>Remove whitespace</td>
<td>
<code>' text '</code> → <code>'text'</code>
</td>
</tr>
<tr>
<td><code>split</code></td>
<td>Split into array</td>
<td>
<code>'a,b,c'</code> → <code>['a','b','c']</code>
</td>
</tr>
<tr>
<td><code>replace</code></td>
<td>Regex replacement</td>
<td>
<code>'foo'</code> → <code>'bar'</code>
</td>
</tr>
<tr>
<td><code>addPrefix</code></td>
<td>Add prefix</td>
<td>
<code>'123'</code> → <code>'ID-123'</code>
</td>
</tr>
<tr>
<td><code>addSuffix</code></td>
<td>Add suffix</td>
<td>
<code>'file'</code> → <code>'file.txt'</code>
</td>
</tr>
<tr>
<td><code>toNumber</code></td>
<td>Parse as number</td>
<td>
<code>'42'</code> → <code>42</code>
</td>
</tr>
<tr>
<td><code>extract</code></td>
<td>Extract substring</td>
<td>
<code>'user@example.com'</code> → <code>'example.com'</code>
</td>
</tr>
<tr>
<td><code>mergeObjects</code></td>
<td>Combine fields into object</td>
<td>
<code>{a, b}</code> → <code>{merged: {a, b}}</code>
</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Filtering and Sorting
</h2>

<p>Apply filters and sorting at the source level for efficient data retrieval:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">pipeline</span><span class="p">:</span> <span class="nx">Pipeline</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">active-premium-users</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">source</span><span class="p">:</span> <span class="p">{</span>
    <span class="c1">// ...</span>
    <span class="na">filters</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">status</span><span class="dl">'</span><span class="p">,</span> <span class="na">operator</span><span class="p">:</span> <span class="dl">'</span><span class="s1">=</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">active</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">plan</span><span class="dl">'</span><span class="p">,</span> <span class="na">operator</span><span class="p">:</span> <span class="dl">'</span><span class="s1">=</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">premium</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">created_at</span><span class="dl">'</span><span class="p">,</span> <span class="na">operator</span><span class="p">:</span> <span class="dl">'</span><span class="s1">&gt;=</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">2024-01-01</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">],</span>
    <span class="na">sort</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">created_at</span><span class="dl">'</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">desc</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">name</span><span class="dl">'</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">asc</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">],</span>
    <span class="na">limit</span><span class="p">:</span> <span class="mi">1000</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Filter Groups
</h3>

<p>For complex conditions, use filter groups with AND/OR logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">filters</span><span class="p">:</span> <span class="p">[</span>
  <span class="p">{</span>
    <span class="na">op</span><span class="p">:</span> <span class="dl">'</span><span class="s1">OR</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">filters</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">status</span><span class="dl">'</span><span class="p">,</span> <span class="na">operator</span><span class="p">:</span> <span class="dl">'</span><span class="s1">=</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">active</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">status</span><span class="dl">'</span><span class="p">,</span> <span class="na">operator</span><span class="p">:</span> <span class="dl">'</span><span class="s1">=</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">pending</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">],</span>
  <span class="p">},</span>
  <span class="p">{</span> <span class="na">field</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email_verified</span><span class="dl">'</span><span class="p">,</span> <span class="na">operator</span><span class="p">:</span> <span class="dl">'</span><span class="s1">=</span><span class="dl">'</span><span class="p">,</span> <span class="na">value</span><span class="p">:</span> <span class="dl">'</span><span class="s1">true</span><span class="dl">'</span> <span class="p">},</span>
<span class="p">]</span>
<span class="c1">// SQL: WHERE (status = 'active' OR status = 'pending') AND email_verified = true</span>
</code></pre>

</div>



<h2>
  
  
  Source to Target Pipelines
</h2>

<p>Move data between systems by defining both source and target:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">pipeline</span><span class="p">:</span> <span class="nx">Pipeline</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">sync-crm-to-warehouse</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">source</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">hubspot-source</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">hubspot</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">contacts</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">credential_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">hubspot-api</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">fields</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">firstname</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">lastname</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">company</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">pagination</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">cursor</span><span class="dl">'</span><span class="p">,</span> <span class="na">itemsPerPage</span><span class="p">:</span> <span class="mi">100</span> <span class="p">},</span>
  <span class="p">},</span>
  <span class="na">target</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">postgres-target</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">adapter_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">postgresql</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">endpoint_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">table_insert</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">credential_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">warehouse-db</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">fields</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">first_name</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">last_name</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">company</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">config</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">schema</span><span class="p">:</span> <span class="dl">'</span><span class="s1">analytics</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">table</span><span class="p">:</span> <span class="dl">'</span><span class="s1">crm_contacts</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">etl</span><span class="p">.</span><span class="nf">runPipeline</span><span class="p">(</span><span class="nx">pipeline</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Synced </span><span class="p">${</span><span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> contacts to warehouse`</span><span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  Pagination Strategies
</h2>

<p>Different APIs use different pagination methods. OpenETL supports three strategies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Offset-based (SQL databases)</span>
<span class="nx">pagination</span><span class="p">:</span> <span class="p">{</span> <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">offset</span><span class="dl">'</span><span class="p">,</span> <span class="nx">itemsPerPage</span><span class="p">:</span> <span class="mi">100</span> <span class="p">}</span>

<span class="c1">// Cursor-based (modern APIs like HubSpot, Stripe)</span>
<span class="nl">pagination</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">cursor</span><span class="dl">'</span><span class="p">,</span> <span class="na">itemsPerPage</span><span class="p">:</span> <span class="mi">100</span> <span class="p">}</span>

<span class="c1">// Page-based (traditional REST APIs)</span>
<span class="nl">pagination</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">page</span><span class="dl">'</span><span class="p">,</span> <span class="na">itemsPerPage</span><span class="p">:</span> <span class="mi">50</span> <span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Error Handling
</h2>

<p>Configure retry behavior for resilient pipelines:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">pipeline</span><span class="p">:</span> <span class="nx">Pipeline</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">resilient-sync</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">source</span><span class="p">:</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">},</span>
  <span class="na">error_handling</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">max_retries</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>
    <span class="na">retry_interval</span><span class="p">:</span> <span class="mi">1000</span><span class="p">,</span>  <span class="c1">// ms between retries</span>
    <span class="na">fail_on_error</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>   <span class="c1">// stop on first error</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<p>OpenETL uses exponential backoff with jitter to prevent thundering herd problems when retrying.</p>

<h2>
  
  
  Pipeline Validation
</h2>

<p>Validate pipelines before execution to catch configuration errors early:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">validatePipeline</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">openetl</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">validatePipeline</span><span class="p">(</span><span class="nx">pipeline</span><span class="p">,</span> <span class="nx">adapters</span><span class="p">,</span> <span class="nx">vault</span><span class="p">);</span>

<span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">result</span><span class="p">.</span><span class="nx">valid</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Pipeline validation failed:</span><span class="dl">'</span><span class="p">);</span>
  <span class="nx">result</span><span class="p">.</span><span class="nx">errors</span><span class="p">.</span><span class="nf">forEach</span><span class="p">(</span><span class="nx">err</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="s2">`  - </span><span class="p">${</span><span class="nx">err</span><span class="p">}</span><span class="s2">`</span><span class="p">));</span>
<span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
  <span class="c1">// Safe to run</span>
  <span class="k">await</span> <span class="nx">etl</span><span class="p">.</span><span class="nf">runPipeline</span><span class="p">(</span><span class="nx">pipeline</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Building Custom Adapters
</h2>

<p>Create adapters for any data source by implementing the adapter interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Adapter</span><span class="p">,</span> <span class="nx">AdapterInstance</span><span class="p">,</span> <span class="nx">Connector</span><span class="p">,</span> <span class="nx">AuthConfig</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">openetl</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">MyCustomAdapter</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">my-adapter</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">My Custom Adapter</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">http</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">action</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">download</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">upload</span><span class="dl">'</span><span class="p">],</span>
  <span class="na">credential_type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">api_key</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">base_url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://api.example.com</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">endpoints</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">users</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Fetch users</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">supported_actions</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">download</span><span class="dl">'</span><span class="p">],</span>
    <span class="p">},</span>
  <span class="p">],</span>
<span class="p">};</span>

<span class="kd">function</span> <span class="nf">myAdapter</span><span class="p">(</span><span class="nx">connector</span><span class="p">:</span> <span class="nx">Connector</span><span class="p">,</span> <span class="nx">auth</span><span class="p">:</span> <span class="nx">AuthConfig</span><span class="p">):</span> <span class="nx">AdapterInstance</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="k">async</span> <span class="nf">connect</span><span class="p">()</span> <span class="p">{</span>
      <span class="c1">// Validate connection</span>
    <span class="p">},</span>

    <span class="k">async</span> <span class="nf">disconnect</span><span class="p">()</span> <span class="p">{</span>
      <span class="c1">// Cleanup</span>
    <span class="p">},</span>

    <span class="k">async</span> <span class="nf">download</span><span class="p">({</span> <span class="nx">limit</span><span class="p">,</span> <span class="nx">offset</span> <span class="p">})</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span>
        <span class="s2">`https://api.example.com/users?limit=</span><span class="p">${</span><span class="nx">limit</span><span class="p">}</span><span class="s2">&amp;offset=</span><span class="p">${</span><span class="nx">offset</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="p">{</span> <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">Authorization</span><span class="dl">'</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">auth</span><span class="p">.</span><span class="nx">credentials</span><span class="p">.</span><span class="nx">api_key</span><span class="p">}</span><span class="s2">`</span> <span class="p">}</span> <span class="p">}</span>
      <span class="p">);</span>
      <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
      <span class="k">return</span> <span class="p">{</span> <span class="nx">data</span> <span class="p">};</span>
    <span class="p">},</span>

    <span class="k">async</span> <span class="nf">upload</span><span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://api.example.com/users</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
        <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">data</span><span class="p">),</span>
      <span class="p">});</span>
    <span class="p">},</span>
  <span class="p">};</span>
<span class="p">}</span>

<span class="k">export</span> <span class="p">{</span> <span class="nx">myAdapter</span><span class="p">,</span> <span class="nx">MyCustomAdapter</span> <span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  Type Safety Benefits
</h2>

<p>TypeScript provides compile-time checking for your pipelines:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Type error: 'invalid' is not a valid pagination type</span>
<span class="kd">const</span> <span class="nx">pipeline</span><span class="p">:</span> <span class="nx">Pipeline</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">source</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">pagination</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">invalid</span><span class="dl">'</span> <span class="p">},</span> <span class="c1">// ❌ TypeScript error</span>
  <span class="p">},</span>
<span class="p">};</span>

<span class="c1">// Type error: missing required field</span>
<span class="kd">const</span> <span class="nx">auth</span><span class="p">:</span> <span class="nx">BasicAuth</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">db</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">basic</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">credentials</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">username</span><span class="p">:</span> <span class="dl">'</span><span class="s1">user</span><span class="dl">'</span><span class="p">,</span>
    <span class="c1">// password: missing  // ❌ TypeScript error</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>OpenETL brings the benefits of TypeScript to ETL development:</p>

<ul>
<li>
<strong>Type safety</strong> catches configuration errors at compile time</li>
<li>
<strong>Adapter abstraction</strong> decouples pipelines from data sources</li>
<li>
<strong>Built-in transformations</strong> handle common data manipulation</li>
<li>
<strong>Flexible pagination</strong> works with any API pattern</li>
<li>
<strong>Error handling</strong> with exponential backoff for resilience</li>
</ul>

<p>By using these patterns, you can build maintainable, testable ETL pipelines that scale with your data integration needs.</p>

<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://github.com/openetl/openetl" rel="noopener noreferrer">OpenETL GitHub Repository</a></li>
<li><a href="//./openetl.md">API Documentation</a></li>
<li><a href="https://github.com/openetl/openetl/tree/main/adapters" rel="noopener noreferrer">Available Adapters</a></li>
<li><a href="https://jspreadsheet.com" rel="noopener noreferrer">JavaScript Spreadsheet Component</a></li>
<li><a href="https://calendarjs.com" rel="noopener noreferrer">JavaScript Calendar Component</a></li>
</ul>




<p><em>OpenETL is open source under the MIT license. Contributions welcome!</em></p>

