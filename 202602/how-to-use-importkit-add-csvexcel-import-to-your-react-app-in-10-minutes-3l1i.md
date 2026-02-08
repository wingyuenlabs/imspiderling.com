---
Title: How to Use ImportKit: Add CSV/Excel Import to Your React App in 10 Minutes
Description: 
Author: Gissur Runarsson
Date: 2026-02-08T21:37:10.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever spent weeks building CSV import functionality only to realize you still need Excel support, better error handling, and a column mapping UI that doesn't confuse users? ImportKit solves this with a drop-in React component that handles everything from file parsing to AI-powered column matching. Here's how to integrate it in one afternoon.</p>

<h1>
  
  
  How to Use ImportKit: Add CSV/Excel Import to Your React App in 10 Minutes
</h1>

<p>Building CSV import functionality from scratch takes weeks. You need file parsing, column mapping UI, validation logic, error handling, and Excel support. ImportKit gives you all of this as a React component you can drop into your app in one afternoon.</p>

<p>This guide walks through integrating ImportKit for a common scenario: letting users import contact lists into a CRM application.</p>

<h2>
  
  
  Prerequisites
</h2>

<p>You need:</p>

<ul>
<li>A React 18.x or React 19.x application</li>
<li>An ImportKit account (sign up at importkit.app)</li>
<li>10 minutes</li>
</ul>

<p>ImportKit supports both React 18 and React 19 as peer dependencies, so it works with any modern React setup including Next.js applications.</p>

<h2>
  
  
  Step 1: Install the Package
</h2>

<p>ImportKit is published as <code>@importkit/react</code> on npm. The package is MIT licensed and includes TypeScript type definitions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @importkit/react
</code></pre>

</div>



<p>The package provides dual ESM/CJS builds via tsup, so it works with both modern and legacy bundler configurations.</p>

<h2>
  
  
  Step 2: Get Your API Key
</h2>

<p>Log into the ImportKit dashboard and navigate to the API Keys section. Generate a new key for your application. The widget requires an API key for authentication and usage tracking. Keys are scoped per user and can be revoked from the dashboard.</p>

<p>The free tier gives you 500 rows per month, which resets automatically on the 1st via Supabase pg_cron. Paid tiers offer 5,000 rows (Starter), 50,000 rows (Pro), or unlimited (Enterprise).</p>

<h2>
  
  
  Step 3: Define Your Data Schema
</h2>

<p>ImportKit uses a field configuration to define what data you expect. For a contact import, you might have:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">contactFields</span> <span class="o">=</span> <span class="p">[</span>
  <span class="p">{</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Email Address</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">email</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">required</span><span class="p">:</span> <span class="kc">true</span>
  <span class="p">},</span>
  <span class="p">{</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">firstName</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">'</span><span class="s1">First Name</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">required</span><span class="p">:</span> <span class="kc">true</span>
  <span class="p">},</span>
  <span class="p">{</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">lastName</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Last Name</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">required</span><span class="p">:</span> <span class="kc">false</span>
  <span class="p">},</span>
  <span class="p">{</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">status</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Status</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">enum</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">enum</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">values</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">Active</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Inactive</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Pending</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">hints</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">enabled,live</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">disabled,off</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">waiting,review</span><span class="dl">'</span><span class="p">]</span>
    <span class="p">}</span>
  <span class="p">},</span>
  <span class="p">{</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">age</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">label</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Age</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">number</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">validate</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span>
        <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">min</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">value</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
        <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Age must be positive</span><span class="dl">'</span>
      <span class="p">}</span>
    <span class="p">]</span>
  <span class="p">}</span>
<span class="p">]</span>
</code></pre>

</div>



<p>ImportKit supports five field types: text, email, number, date, and enum. Each type can have custom validation rules including email format checking, numeric ranges, string length constraints, and regex patterns.</p>

<p>The <code>enum</code> field type is particularly powerful. It uses a 6-step matching cascade to handle messy real-world data: exact match, case-insensitive match, customer-specific learned mappings, global learned mappings, developer-provided hints, and finally AI semantic matching. This means "Administrator" automatically maps to "Admin" and "enabled" maps to "Active" using the hints you provide.</p>

<h2>
  
  
  Step 4: Add the Widget to Your Component
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">ImportWidget</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@importkit/react</span><span class="dl">'</span>

<span class="kd">function</span> <span class="nf">ContactImportPage</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">handleComplete</span> <span class="o">=</span> <span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Imported contacts:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">data</span><span class="p">)</span>
    <span class="c1">// Send to your API</span>
    <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/contacts/bulk</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">data</span><span class="p">),</span>
      <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span> <span class="p">}</span>
    <span class="p">})</span>
  <span class="p">}</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">h1</span><span class="o">&gt;</span><span class="nx">Import</span> <span class="nx">Contacts</span><span class="o">&lt;</span><span class="sr">/h1</span><span class="err">&gt;
</span>      <span class="o">&lt;</span><span class="nx">ImportWidget</span>
        <span class="nx">apiKey</span><span class="o">=</span><span class="dl">"</span><span class="s2">your-api-key</span><span class="dl">"</span>
        <span class="nx">fields</span><span class="o">=</span><span class="p">{</span><span class="nx">contactFields</span><span class="p">}</span>
        <span class="nx">onComplete</span><span class="o">=</span><span class="p">{</span><span class="nx">handleComplete</span><span class="p">}</span>
      <span class="sr">/</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The widget handles everything: drag-and-drop file upload, parsing CSV and Excel files (.csv, .xlsx, .xls), column mapping, validation, and error correction. Users see a multi-step interface that guides them through the import process.</p>

<h2>
  
  
  Step 5: Let AI Handle Column Mapping
</h2>

<p>When a user uploads a file, ImportKit sends the CSV headers and your target fields to <code>/api/suggest-mapping</code>. This endpoint calls OpenAI gpt-4o-mini with temperature=0 for deterministic results. The AI returns structured JSON via the response_format parameter with confidence scores for each mapping suggestion.</p>

<p>If AI fails or is unavailable, the widget falls back to substring matching automatically. This means your imports work even if OpenAI is down. Users see immediate substring matching while the AI loads asynchronously in the background.</p>

<p>For example, if the CSV has columns "E-mail", "First", "Last", "Account Status", the AI maps:</p>

<ul>
<li>"E-mail" → email (95% confidence)</li>
<li>"First" → firstName (90% confidence)</li>
<li>"Last" → lastName (90% confidence)</li>
<li>"Account Status" → status (85% confidence)</li>
</ul>

<p>Users can override any mapping manually if the AI gets it wrong.</p>

<h2>
  
  
  Step 6: Validate Before Import
</h2>

<p>ImportKit validates every row against your rules in real-time. The validation shows inline errors per row in the preview step. Users can choose to import only valid rows or fix errors before importing.</p>

<p>Validation rules support email format checking, numeric ranges (min/max), string length constraints (minLength/maxLength), regex patterns, and enum value matching. Required fields block the entire import if missing.</p>

<p>This catches data quality issues before they enter your system and reduces support tickets from bad imports.</p>

<h2>
  
  
  Optional: Use Templates for Reusable Configs
</h2>

<p>If you have multiple import scenarios or want to let users manage imports themselves, save your field configuration as a template in the dashboard.</p>

<p>Templates are stored in Supabase's <code>import_templates</code> table and loaded via the widget's <code>templateId</code> prop:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="o">&lt;</span><span class="nx">ImportWidget</span>
  <span class="nx">apiKey</span><span class="o">=</span><span class="dl">"</span><span class="s2">your-api-key</span><span class="dl">"</span>
  <span class="nx">templateId</span><span class="o">=</span><span class="dl">"</span><span class="s2">template-uuid</span><span class="dl">"</span>
  <span class="nx">onComplete</span><span class="o">=</span><span class="p">{</span><span class="nx">handleComplete</span><span class="p">}</span>
<span class="sr">/</span><span class="err">&gt;
</span></code></pre>

</div>



<p>This enables consistent imports across users and sessions while reducing implementation time. Non-technical users can manage import configurations through the dashboard CRUD UI.</p>

<h2>
  
  
  Optional: Customize the Appearance
</h2>

<p>ImportKit supports full theming to match your brand. You can customize colors, borders, fonts, and remove the "Powered by ImportKit" branding:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="o">&lt;</span><span class="nx">ImportWidget</span>
  <span class="nx">apiKey</span><span class="o">=</span><span class="dl">"</span><span class="s2">your-api-key</span><span class="dl">"</span>
  <span class="nx">fields</span><span class="o">=</span><span class="p">{</span><span class="nx">contactFields</span><span class="p">}</span>
  <span class="nx">onComplete</span><span class="o">=</span><span class="p">{</span><span class="nx">handleComplete</span><span class="p">}</span>
  <span class="nx">theme</span><span class="o">=</span><span class="p">{{</span>
    <span class="na">primaryColor</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#6366f1</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">borderRadius</span><span class="p">:</span> <span class="dl">'</span><span class="s1">12px</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">fontFamily</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Inter</span><span class="dl">'</span>
  <span class="p">}}</span>
  <span class="nx">showBranding</span><span class="o">=</span><span class="p">{</span><span class="kc">false</span><span class="p">}</span>
<span class="sr">/</span><span class="err">&gt;
</span></code></pre>

</div>



<p>The <code>showBranding</code> prop defaults to true. Setting it to false removes the "Powered by ImportKit" footer for a white-label experience.</p>

<h2>
  
  
  Optional: Set Up Webhook Delivery
</h2>

<p>If you're on the Pro or Enterprise tier, configure a webhook URL in the dashboard to receive import data automatically. ImportKit POSTs the data to your endpoint after each import completes.</p>

<p>Webhooks include an <code>X-ImportKit-Signature</code> header with an HMAC-SHA256 signature for verification. The secret is generated via <code>crypto.randomBytes(32)</code>. Recipients verify by computing HMAC of the request body.</p>

<p>Delivery retries 3 times with exponential backoff (1 second, 4 seconds delays) and a 10-second timeout per attempt. Each webhook payload includes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"event"</span><span class="p">:</span><span class="w"> </span><span class="s2">"import.completed"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"importId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"uuid"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"rowCount"</span><span class="p">:</span><span class="w"> </span><span class="mi">150</span><span class="p">,</span><span class="w">
  </span><span class="nl">"templateName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Contact Import"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2024-01-28T10:30:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This enables automated workflows without polling. If all retry attempts fail, the delivery is logged but does not block the import response. Webhooks are fire-and-forget from the user's perspective.</p>

<h2>
  
  
  What Happens Behind the Scenes
</h2>

<p>ImportKit uses PapaParse for CSV parsing and SheetJS (xlsx library) for Excel parsing. It automatically extracts headers and data from uploaded files. The widget reads the first sheet by default from Excel files and converts to JSON for mapping.</p>

<p>All authentication happens via API key in the request body, not cookies. This uses <code>supabaseAdmin</code> to bypass Row Level Security (RLS) for API key validation. The widget works from any domain without CORS issues since the API routes are configured with appropriate CORS headers.</p>

<p>Usage tracking happens through the <code>/api/track-import</code> endpoint, which checks <code>rows_used_this_month</code> against tier limits before recording an import. If you exceed your limit, you get an error until the automatic reset on the 1st.</p>

<h2>
  
  
  When to Use ImportKit vs Building It Yourself
</h2>

<p>ImportKit replaces weeks of development time with a single npm package. You avoid building parsing logic, validation UI, mapping interfaces, error handling, retry logic, and Excel support from scratch.</p>

<p>Compare this to DIY solutions:</p>

<ul>
<li>ImportKit provides AI-powered column matching vs manual substring matching</li>
<li>Built-in error handling and retry logic for webhooks vs implementing your own</li>
<li>Handles both CSV and Excel formats out of the box vs separate parsers</li>
<li>Maintained and updated by the ImportKit team vs your maintenance burden</li>
</ul>

<p>The widget is open source (MIT licensed) with code published at github.com/gthorr/importkit. The dashboard and backend are part of the hosted service.</p>




<p>For implementation questions, check the <a href="https://github.com/gthorr/importkit" rel="noopener noreferrer">GitHub repository</a> or the documentation at <a href="https://importkit.app" rel="noopener noreferrer">importkit.app</a>.</p>

