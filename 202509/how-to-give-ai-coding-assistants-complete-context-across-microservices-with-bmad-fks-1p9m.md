---
Title: How to Give AI Coding Assistants Complete Context Across Microservices with BMAD FKS
Description: 
Author: vishalmysore
Date: 2025-09-16T21:35:04.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building Context with BMAD Federated Knowledge System
</h1>

<h2>
  
  
  What is BMAD?
</h2>

<p>BMAD stands for "Breakthrough Method for Agile AI-Driven Development" - a powerful agentic framework that transforms domains through specialized AI expertise. At its core, BMAD is a four-phased context engineering approach to product development that sets the foundation for agentic product development by clarifying all connected layers of the process.</p>

<p>Code is <a href="https://github.com/vishalmysore/bmad-federated-knowledge" rel="noopener noreferrer">Here</a></p>

<h2>
  
  
  The Challenge of Context in Microservices
</h2>

<p>Modern software systems often employ distributed microservice architectures, creating significant challenges for developers, operators, and AI assistants trying to understand the complete system. Consider a healthcare application with separate services:</p>

<ul>
<li>Patient records management service</li>
<li>Medicine supply service </li>
<li>Billing service</li>
<li>Appointment scheduling service</li>
</ul>

<p>Each service has its own repository, data sources, APIs, and dependencies, making it difficult to maintain a unified understanding of the system especially for an AI dev agent as they work based on single repostirty and limted context</p>

<h2>
  
  
  BMAD Federated Knowledge: The Solution
</h2>

<p>The BMAD Federated Knowledge System (FKS) solves this challenge by creating a unified context layer that aggregates knowledge from multiple sources. This system enables:</p>

<ul>
<li>
<strong>Cross-service understanding</strong>: Gain insights across all your microservices</li>
<li>
<strong>Comprehensive context</strong>: Combine code, documentation, databases, and external sources</li>
<li>
<strong>AI-ready information</strong>: Format knowledge in ways that LLMs can efficiently consume</li>
</ul>

<h2>
  
  
  Configuration Setup
</h2>

<p>To use the BMAD Federated Knowledge System, you first need to initialize it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed init
</code></pre>

</div>



<p>This creates a configuration file (by default at <code>./.bmad-fks-core/fks-core-config.yaml</code>) with the following structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">bmad_config</span><span class="pi">:</span>
  <span class="na">federated_settings</span><span class="pi">:</span>
    <span class="na">cache_root</span><span class="pi">:</span> <span class="s">./.bmad-fks-cache</span>
    <span class="na">max_cache_size</span><span class="pi">:</span> <span class="s">1GB</span>
    <span class="na">max_file_size</span><span class="pi">:</span> <span class="s">10MB</span>
    <span class="na">exclude_patterns</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">node_modules/**</span>
      <span class="pi">-</span> <span class="s">.git/**</span>
    <span class="na">token_safety_margin</span><span class="pi">:</span> <span class="m">10000</span>

  <span class="na">knowledge_sources</span><span class="pi">:</span>
    <span class="c1"># Your knowledge sources will be configured here</span>

  <span class="na">connections</span><span class="pi">:</span>
    <span class="c1"># Database connections will be defined here</span>
</code></pre>

</div>



<h2>
  
  
  Adding Knowledge Sources
</h2>

<p>BMAD Federated Knowledge System supports three types of knowledge sources: repositories, databases, and web sources. Here's how to add each type:</p>

<h3>
  
  
  1. Adding Repository Sources
</h3>

<p>Use the interactive command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed add-knowledge
</code></pre>

</div>



<p>When prompted, select "Repository" as the knowledge type and provide:</p>

<ul>
<li>Name for the repository</li>
<li>Git URL</li>
<li>Branch (defaults to main)</li>
<li>Local cache path (optional)</li>
<li>Description</li>
</ul>

<p>Or modify the config file directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">bmad_config</span><span class="pi">:</span>
  <span class="na">knowledge_sources</span><span class="pi">:</span>
    <span class="na">patient-service</span><span class="pi">:</span>
      <span class="na">type</span><span class="pi">:</span> <span class="s">repository</span>
      <span class="na">repo</span><span class="pi">:</span> <span class="s">https://github.com/healthcare-org/patient-service.git</span>
      <span class="na">branch</span><span class="pi">:</span> <span class="s">main</span>
      <span class="na">local_cache</span><span class="pi">:</span> <span class="s">./.bmad-fks-cache/patient-service</span>
      <span class="na">metadata</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Patient</span><span class="nv"> </span><span class="s">records</span><span class="nv"> </span><span class="s">management</span><span class="nv"> </span><span class="s">microservice"</span>
</code></pre>

</div>



<h3>
  
  
  2. Adding Database Sources
</h3>

<p>First, add a database connection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed add-connection
</code></pre>

</div>



<p>Select the database type (MongoDB, MySQL, PostgreSQL) and provide connection details.</p>

<p>Then, add a database knowledge source:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed add-knowledge
</code></pre>

</div>



<p>When prompted, select "Database" as the knowledge type and provide:</p>

<ul>
<li>Name for the database source</li>
<li>Select previously added connection</li>
<li>SQL query or MongoDB query</li>
<li>Description</li>
</ul>

<p>This creates a configuration like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">bmad_config</span><span class="pi">:</span>
  <span class="na">connections</span><span class="pi">:</span>
    <span class="na">mongo-patient-db</span><span class="pi">:</span>
      <span class="na">type</span><span class="pi">:</span> <span class="s">mongodb</span>
      <span class="na">connection_string</span><span class="pi">:</span> <span class="s">mongodb://username:password@hostname:27017/patients</span>

  <span class="na">knowledge_sources</span><span class="pi">:</span>
    <span class="na">patient-demographics</span><span class="pi">:</span>
      <span class="na">type</span><span class="pi">:</span> <span class="s">database</span>
      <span class="na">connection_ref</span><span class="pi">:</span> <span class="s">mongo-patient-db</span>
      <span class="na">query</span><span class="pi">:</span> <span class="s1">'</span><span class="s">{"collection":</span><span class="nv"> </span><span class="s">"patients",</span><span class="nv"> </span><span class="s">"filter":</span><span class="nv"> </span><span class="s">{"active":</span><span class="nv"> </span><span class="s">true}}'</span>
      <span class="na">metadata</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Active</span><span class="nv"> </span><span class="s">patient</span><span class="nv"> </span><span class="s">demographics</span><span class="nv"> </span><span class="s">information"</span>
</code></pre>

</div>



<p>To sync the database knowledge source and generate PDF/JSON files:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed sync-db <span class="nt">--all</span>
</code></pre>

</div>



<p>Or for a specific source:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed sync-db patient-demographics
</code></pre>

</div>



<p>Options:</p>

<ul>
<li>
<code>--json</code>: Save as JSON instead of PDF</li>
<li>
<code>--force</code>: Force sync even if recently synced</li>
<li>
<code>--mock</code>: Use mock data for testing (without actual DB connection)</li>
</ul>

<h3>
  
  
  3. Adding Web Sources
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed add-knowledge
</code></pre>

</div>



<p>When prompted, select "Web" as the knowledge type and provide:</p>

<ul>
<li>Name for the web source</li>
<li>URL to the web resource</li>
<li>Description</li>
</ul>

<p>This creates a configuration like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">bmad_config</span><span class="pi">:</span>
  <span class="na">knowledge_sources</span><span class="pi">:</span>
    <span class="na">api-documentation</span><span class="pi">:</span>
      <span class="na">type</span><span class="pi">:</span> <span class="s">web</span>
      <span class="na">url</span><span class="pi">:</span> <span class="s">https://healthcare-org.github.io/api-docs/</span>
      <span class="na">metadata</span><span class="pi">:</span>
        <span class="na">description</span><span class="pi">:</span> <span class="s2">"</span><span class="s">API</span><span class="nv"> </span><span class="s">documentation</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">all</span><span class="nv"> </span><span class="s">microservices"</span>
</code></pre>

</div>



<h2>
  
  
  Building Context with Multiple Knowledge Sources
</h2>

<p>The <code>build-context</code> command is the cornerstone of the BMAD Federated Knowledge System. It aggregates information from all configured knowledge sources:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed build-context
</code></pre>

</div>



<p>This command automatically:</p>

<ol>
<li>
<strong>Syncs repositories</strong>: Clones or updates all configured git repositories</li>
<li>
<strong>Syncs database sources</strong>: Connects to databases, executes queries, and saves results as PDF or JSON</li>
<li>
<strong>Syncs web sources</strong>: Downloads and processes content from configured web URLs</li>
<li>
<strong>Flattens repositories</strong>: Converts repository code into AI-optimized XML format</li>
<li>
<strong>Generates context.md</strong>: Creates a comprehensive map of all knowledge sources</li>
</ol>

<p>The process looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ bmad-fed build-context
✓ Repos synced: 4 successful, 0 failed
✓ Database sources sync completed.
✓ Web sources sync completed.

🔄 Flattening repo "patient-service" → ./.bmad-fks-cache/patient-service.xml
... flattening logs ...
✓ Flattened patient-service → ./.bmad-fks-cache/patient-service.xml

... more flattening logs for other repos ...

✓ Context built successfully for all repos!

📄 Context definition written to context.md
</code></pre>

</div>



<h2>
  
  
  Output: The Context Definition File
</h2>

<p>The generated <code>context.md</code> file maps all knowledge sources:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># Context Definition File</span>

This file maps each knowledge source to its file, along with descriptions.

<span class="gu">## Repository Sources</span>

<span class="gu">### patient-service</span>
<span class="p">-</span> <span class="gs">**File**</span>: ./.bmad-fks-cache/patient-service.xml
<span class="p">-</span> <span class="gs">**Repository**</span>: https://github.com/healthcare-org/patient-service.git
<span class="p">-</span> <span class="gs">**Description**</span>: Patient records management microservice

... more repository sources ...

<span class="gu">## Database Sources</span>

<span class="gu">### patient-demographics</span>
<span class="p">-</span> <span class="gs">**File**</span>: ./bmad-cache/db-knowledge/patient-demographics.pdf
<span class="p">-</span> <span class="gs">**Connection**</span>: mongo-patient-db
<span class="p">-</span> <span class="gs">**Query**</span>: {"collection": "patients", "filter": {"active": true}}
<span class="p">-</span> <span class="gs">**Description**</span>: Active patient demographics information

... more database sources ...

<span class="gu">## Web Sources</span>

<span class="gu">### api-documentation</span>
<span class="p">-</span> <span class="gs">**File**</span>: ./.bmad-fks-cache/api-documentation.pdf
<span class="p">-</span> <span class="gs">**URL**</span>: https://healthcare-org.github.io/api-docs/
<span class="p">-</span> <span class="gs">**Description**</span>: API documentation for all microservices

... more web sources ...
</code></pre>

</div>



<h2>
  
  
  Cache Structure
</h2>

<p>By default, the system uses these cache directories:</p>

<ul>
<li>Repository sources: <code>./.bmad-fks-cache/[repo-name]</code> and <code>./.bmad-fks-cache/[repo-name].xml</code>
</li>
<li>Database sources: <code>./bmad-cache/db-knowledge/[source-name].pdf</code> or <code>./bmad-cache/db-knowledge/[source-name].json</code>
</li>
<li>Web sources: <code>./.bmad-fks-cache/[source-name].pdf</code>
</li>
</ul>

<p>You can configure the cache root in the settings:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">bmad_config</span><span class="pi">:</span>
  <span class="na">federated_settings</span><span class="pi">:</span>
    <span class="na">cache_root</span><span class="pi">:</span> <span class="s">./your-custom-cache-directory</span>
</code></pre>

</div>



<h2>
  
  
  Using Built Context for Microservices Development
</h2>

<p>After running <code>build-context</code>, the unified context enables:</p>

<ol>
<li>
<strong>Enhanced AI assistance</strong>: Your IDE's AI assistant gains cross-service understanding</li>
<li>
<strong>Better design decisions</strong>: Make informed architectural choices with complete context</li>
<li>
<strong>Dependency awareness</strong>: Understand how services impact each other</li>
<li>
<strong>Streamlined onboarding</strong>: Help new team members understand the entire system</li>
</ol>

<h2>
  
  
  Advanced Commands
</h2>

<h3>
  
  
  Checking System Status
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed status
</code></pre>

</div>



<p>This displays the status of all configured knowledge sources, cache usage, and more.</p>

<h3>
  
  
  Rebuilding Specific Knowledge Sources
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed sync-source <span class="o">[</span>source-name]
</code></pre>

</div>



<h3>
  
  
  Managing Connections
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bmad-fed list-connections     <span class="c"># List all connections</span>
bmad-fed test-connection <span class="o">[</span>name]  <span class="c"># Test a specific connection</span>
bmad-fed remove-connection <span class="o">[</span>name]  <span class="c"># Remove a connection</span>
</code></pre>

</div>



<h2>
  
  
  Troubleshooting
</h2>

<p>Common issues and solutions:</p>

<ol>
<li>
<p><strong>Database connection errors</strong>: </p>

<ul>
<li>Check connection strings in config</li>
<li>Ensure database drivers are installed (<code>npm install mongodb mysql2 pg</code>)</li>
<li>Use <code>--mock</code> flag to test with mock data</li>
</ul>
</li>
<li>
<p><strong>Repository sync failures</strong>:</p>

<ul>
<li>Verify git credentials</li>
<li>Check network connectivity</li>
<li>Ensure repository URL is correct</li>
</ul>
</li>
<li>
<p><strong>Cache directory issues</strong>:</p>

<ul>
<li>Ensure write permissions for cache directories</li>
<li>Check if paths in config are consistent</li>
</ul>
</li>
</ol>

<h2>
  
  
  Getting Started
</h2>

<p>To build your first federated context:</p>

<ol>
<li>Install the BMAD Federated Knowledge System: <code>npm install -g bmad-federated-knowledge</code>
</li>
<li>Initialize the system: <code>bmad-fed init</code>
</li>
<li>Add knowledge sources: <code>bmad-fed add-knowledge</code>
</li>
<li>Build context: <code>bmad-fed build-context</code>
</li>
<li>Use the generated context with your preferred AI coding assistant</li>
</ol>

<p>By leveraging the BMAD Federated Knowledge System, your microservices development can achieve a new level of cohesion and insight, even as your system grows in complexity.</p>

