---
Title: Import Your AI Memory into ContextForge: Claude Code, ChatGPT, and Knowledge Graph
Description: 
Author: Alfredo Izquierdo
Date: 2026-03-09T22:06:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>You've been building knowledge with your AI assistant for months — project decisions, API patterns, debugging notes. But that knowledge is scattered across <code>MEMORY.md</code> files, ChatGPT exports, and Knowledge Graph JSONL dumps.</p>

<p>Today, ContextForge lets you import all of it in seconds.</p>

<h2>
  
  
  The Problem: Scattered AI Memory
</h2>

<p>If you use AI coding assistants, your knowledge ends up in different places:</p>

<ul>
<li>
<strong>Claude Code</strong> saves memory in <code>~/.claude/projects/*/memory/MEMORY.md</code>
</li>
<li>
<strong>MCP Knowledge Graph</strong> server stores entities in <code>memory.jsonl</code>
</li>
<li>
<strong>ChatGPT</strong> lets you export conversations as <code>conversations.json</code>
</li>
<li>
<strong>Random Markdown files</strong> with notes you copied manually</li>
</ul>

<p>None of these tools talk to each other. Switch from one to another, and you start from zero.</p>

<h2>
  
  
  One Import, All Your Knowledge
</h2>

<p>ContextForge now supports importing from all major AI memory formats:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Format</th>
<th>File Type</th>
<th>What Gets Imported</th>
</tr>
</thead>
<tbody>
<tr>
<td>Claude Code</td>
<td><code>.md</code></td>
<td>Each <code>##</code> section becomes a knowledge item</td>
</tr>
<tr>
<td>MCP Knowledge Graph</td>
<td><code>.jsonl</code></td>
<td>Entities become searchable knowledge items</td>
</tr>
<tr>
<td>ChatGPT</td>
<td><code>.json</code></td>
<td>Assistant responses extracted automatically</td>
</tr>
<tr>
<td>Plain Markdown</td>
<td><code>.md</code></td>
<td>Sections split by headings</td>
</tr>
<tr>
<td>ContextForge JSON</td>
<td><code>.json</code></td>
<td>Full backup/restore between spaces</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  How It Works
</h2>

<h3>
  
  
  From the Dashboard (No Code Required)
</h3>

<ol>
<li>
<strong>Open any Space</strong> in your dashboard</li>
<li>Click the <strong>"Import"</strong> button next to "Add Item"</li>
<li>
<strong>Select your format</strong> or just upload a file (auto-detected)</li>
<li>
<strong>Preview</strong> the items that will be imported</li>
<li>Click <strong>"Import N Items"</strong> — done</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkk5bkptoqz662jmykbw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkk5bkptoqz662jmykbw.png" alt=" " width="800" height="619"></a><br>
Duplicates are automatically detected and skipped using SHA-256 hashing. You won't get repeated items.</p>
<h3>
  
  
  From Claude or Cursor (Via MCP)
</h3>

<p>You can also import directly from your AI assistant:</p>

<blockquote>
<p>"Import my MEMORY.md file into the backend-docs space"</p>
</blockquote>

<p>Or use the MCP tool directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">memory_import(</span><span class="p">{</span><span class="w">
  </span><span class="nl">"format"</span><span class="p">:</span><span class="w"> </span><span class="s2">"claude_memory"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"data"</span><span class="p">:</span><span class="w"> </span><span class="s2">"## Auth Flow</span><span class="se">\n</span><span class="s2">- JWT tokens expire in 1 hour</span><span class="se">\n</span><span class="s2">..."</span><span class="w">
</span><span class="p">}</span><span class="err">)</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Supported Formats in Detail
</h2>

<h3>
  
  
  Claude Code (MEMORY.md)
</h3>

<p>Claude Code stores persistent memory in Markdown files at <code>~/.claude/projects/*/memory/MEMORY.md</code>. Each <code>##</code> heading becomes a separate knowledge item.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gh"># My Project Memory</span>

<span class="gu">## Database Patterns</span>
<span class="p">-</span> Always use connection pooling
<span class="p">-</span> Migrations must be idempotent
<span class="p">-</span> Use RLS for row-level security

<span class="gu">## Auth Flow</span>
<span class="p">-</span> JWT tokens expire in 1 hour
<span class="p">-</span> Refresh tokens last 30 days
</code></pre>

</div>



<p>This file imports as <strong>2 items</strong>: "Database Patterns" and "Auth Flow".</p>

<h3>
  
  
  MCP Knowledge Graph (.jsonl)
</h3>

<p>The official <code>@modelcontextprotocol/server-memory</code> package stores entities in a JSONL file. Each entity becomes a knowledge item with its observations as content.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="s2">"entity"</span><span class="p">,</span><span class="nl">"name"</span><span class="p">:</span><span class="s2">"ContextForge"</span><span class="p">,</span><span class="nl">"entityType"</span><span class="p">:</span><span class="s2">"project"</span><span class="p">,</span><span class="nl">"observations"</span><span class="p">:[</span><span class="s2">"A SaaS for persistent AI memory"</span><span class="p">,</span><span class="s2">"Built with Supabase"</span><span class="p">]}</span><span class="w">
</span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="s2">"entity"</span><span class="p">,</span><span class="nl">"name"</span><span class="p">:</span><span class="s2">"Auth System"</span><span class="p">,</span><span class="nl">"entityType"</span><span class="p">:</span><span class="s2">"component"</span><span class="p">,</span><span class="nl">"observations"</span><span class="p">:[</span><span class="s2">"Uses JWT with 1h expiry"</span><span class="p">,</span><span class="s2">"Refresh tokens in httpOnly cookies"</span><span class="p">]}</span><span class="w">
</span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="s2">"relation"</span><span class="p">,</span><span class="nl">"from"</span><span class="p">:</span><span class="s2">"ContextForge"</span><span class="p">,</span><span class="nl">"to"</span><span class="p">:</span><span class="s2">"Auth System"</span><span class="p">,</span><span class="nl">"relationType"</span><span class="p">:</span><span class="s2">"contains"</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Entities are imported as knowledge items. Relations are preserved for future knowledge graph features.</p>

<h3>
  
  
  ChatGPT Export (conversations.json)
</h3>

<p>Go to <strong>ChatGPT Settings &gt; Data Controls &gt; Export Data</strong>. You'll receive a zip file containing <code>conversations.json</code>.</p>

<p>ContextForge extracts assistant responses from each conversation. Only meaningful content is imported — empty or system messages are skipped.</p>

<h2>
  
  
  Export Anytime
</h2>

<p>Your data is yours. Export from any space in JSON, Markdown, or CSV:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">memory_export(</span><span class="p">{</span><span class="w">
  </span><span class="nl">"format"</span><span class="p">:</span><span class="w"> </span><span class="s2">"json"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"space_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"your-space-id"</span><span class="w">
</span><span class="p">}</span><span class="err">)</span><span class="w">
</span></code></pre>

</div>



<p>Or just ask your AI assistant: <em>"Export my backend-docs space as Markdown."</em></p>

<h2>
  
  
  Get Started
</h2>

<ol>
<li>Sign up at <a href="https://contextforge.dev" rel="noopener noreferrer">contextforge.dev</a> (free tier available)</li>
<li>Install: <code>npm install -g contextforge-mcp</code>
</li>
<li>Open any space and click <strong>Import</strong>
</li>
<li>Upload your <code>MEMORY.md</code>, <code>memory.jsonl</code>, or <code>conversations.json</code>
</li>
</ol>

<p><strong>Stop losing knowledge when you switch tools. Bring everything into one place.</strong></p>




<p><em>Have questions about importing? Check the full docs at <a href="https://contextforge.dev/docs/import-export" rel="noopener noreferrer">contextforge.dev/docs/import-export</a> or reach out through the dashboard.</em></p>

