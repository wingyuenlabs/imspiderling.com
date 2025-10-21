---
Title: FastMCP + Claude Desktop: When Optional[X] Type Hints Break Validation
Description: 
Author: Scott Raisbeck
Date: 2025-10-21T21:57:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>Your MCP server works perfectly. Python tests all green. You deploy to staging, connect Claude Desktop, and immediately hit this error:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Input validation error: '[16]' is not valid under any of the given schemas
</code></pre>

</div>



<p>You try different formats. Arrays, integers, strings. All fail. Same cryptic message every time.</p>

<p>I spent two hours debugging this evening. Turns out there's a mismatch between how FastMCP's Python client handles optional parameters and what Claude Desktop sends over the wire.</p>

<h2>
  
  
  The Crime Scene
</h2>

<p>I was building an MCP server for a RAG system. Tried to create a document with a <code>project_id</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">create_document</span><span class="p">(</span>
    <span class="n">title</span><span class="o">=</span><span class="sh">"</span><span class="s">Test Document</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">content</span><span class="o">=</span><span class="sh">"</span><span class="s">Some content</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">project_id</span><span class="o">=</span><span class="mi">16</span>  <span class="c1"># Integer, seems reasonable
</span><span class="p">)</span>
</code></pre>

</div>



<p>Error. Tried the array format for <code>tags</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">create_code_artifact</span><span class="p">(</span>
    <span class="n">title</span><span class="o">=</span><span class="sh">"</span><span class="s">Test Code</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">code</span><span class="o">=</span><span class="sh">"</span><span class="s">print(</span><span class="sh">'</span><span class="s">hello</span><span class="sh">'</span><span class="s">)</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">tags</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">test</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">validation</span><span class="sh">"</span><span class="p">],</span>  <span class="c1"># Arrays work everywhere else
</span>    <span class="n">project_id</span><span class="o">=</span><span class="mi">16</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Both parameters failed. Same error, same confusion.</p>

<p>The weird part? Linking worked fine:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">link_document_to_project</span><span class="p">(</span><span class="n">document_id</span><span class="o">=</span><span class="mi">25</span><span class="p">,</span> <span class="n">project_id</span><span class="o">=</span><span class="mi">16</span><span class="p">)</span>  <span class="c1"># Success
</span></code></pre>

</div>



<p>So the backend accepted integers. The validation was happening earlier, at the FastMCP protocol layer before my application code even ran.</p>

<h2>
  
  
  The False Start
</h2>

<p>I dug through my input coercion logic, thinking the type handling was broken. It wasn't. Created test cases. Tried different serialization formats. The pattern emerged slowly: every <code>Optional[X]</code> parameter was failing.</p>

<p>Including one I hadn't tested yet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">mark_obsolete</span><span class="p">(</span><span class="n">memory_id</span><span class="o">=</span><span class="mi">176</span><span class="p">,</span> <span class="n">superseded_by</span><span class="o">=</span><span class="mi">178</span><span class="p">)</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Input validation error: '178' is not valid under any of the given schemas
</code></pre>

</div>



<p>Same error pattern across 13 different parameters in my tool definitions.</p>

<h2>
  
  
  The Pattern
</h2>

<p>I checked my tool signatures:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@mcp.tool</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">create_document</span><span class="p">(</span>
    <span class="n">title</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">content</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">project_id</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="nb">int</span><span class="p">]</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>  <span class="c1"># Fails from Claude Desktop
</span>    <span class="n">tags</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="n">List</span><span class="p">[</span><span class="nb">str</span><span class="p">]]</span> <span class="o">=</span> <span class="bp">None</span>   <span class="c1"># Also fails
</span><span class="p">):</span>
    <span class="bp">...</span>
</code></pre>

</div>



<p>Standard Python type hints. What everyone uses for optional parameters.</p>

<p>FastMCP generates different JSON schemas depending on how you declare optional parameters:</p>

<p><strong><code>Optional[List[str]]</code> generates:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"anyOf"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"array"</span><span class="p">,</span><span class="w"> </span><span class="nl">"items"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">}},</span><span class="w">
    </span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"null"</span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong><code>List[str] = None</code> generates:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"array"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"items"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The first format wasn't working with Claude Desktop. The second one was.</p>

<p>The FastMCP Python client? Handled both formats fine. That's why all my tests passed.</p>

<h2>
  
  
  What I Changed
</h2>

<p>Converted every optional parameter from <code>Optional[X]</code> to <code>X = None</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@mcp.tool</span><span class="p">()</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">create_document</span><span class="p">(</span>
    <span class="n">title</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">content</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">project_id</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>      <span class="c1"># Changed from Optional[int]
</span>    <span class="n">tags</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="nb">str</span><span class="p">]</span> <span class="o">=</span> <span class="bp">None</span>       <span class="c1"># Changed from Optional[List[str]]
</span><span class="p">):</span>
    <span class="bp">...</span>
</code></pre>

</div>



<p><strong>13 parameters across 3 files:</strong></p>

<p>memory_tools.py (5):</p>

<ul>
<li>importance_threshold</li>
<li>project_ids
</li>
<li>tags (update)</li>
<li>importance (update)</li>
<li>superseded_by</li>
</ul>

<p>document_tools.py (4):</p>

<ul>
<li>size_bytes</li>
<li>tags</li>
<li>project_id (create + update)</li>
</ul>

<p>code_artifact_tools.py (4):</p>

<ul>
<li>tags (create + list + update)</li>
<li>project_id (create + list + update)</li>
</ul>

<h2>
  
  
  Results
</h2>

<p>Rebuilt the container. Deployed to staging. Tested every parameter that had been failing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># All of these now work
</span><span class="nf">create_document</span><span class="p">(</span><span class="n">project_id</span><span class="o">=</span><span class="mi">16</span><span class="p">)</span>
<span class="nf">create_memory</span><span class="p">(</span><span class="n">project_ids</span><span class="o">=</span><span class="p">[</span><span class="mi">16</span><span class="p">])</span>
<span class="nf">create_code_artifact</span><span class="p">(</span><span class="n">tags</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">test</span><span class="sh">"</span><span class="p">],</span> <span class="n">project_id</span><span class="o">=</span><span class="mi">16</span><span class="p">)</span>
<span class="nf">mark_obsolete</span><span class="p">(</span><span class="n">superseded_by</span><span class="o">=</span><span class="mi">180</span><span class="p">)</span>
</code></pre>

</div>



<p>Green across the board.</p>

<h2>
  
  
  The Takeaway
</h2>

<p>Type hints matter at the protocol boundary. <code>Optional[X]</code> is semantically identical to <code>X = None</code> in Python, but i think FastMCP treats them differently when generating JSON schemas. Different MCP clients serialize parameters differently. That was my guess at least, well Claude Desktop's guess at it realised it was occurring on particular fields marked as Optional[], i sometimes have to pinch myself when the system I am working with tells me what the issue is with my integration logic, but this is the world we live in!</p>

<p>This is the kind of bug that's invisible in tests unless you're testing against the actual client. Integration tests with the FastMCP Python client pass. The failure only shows up when Claude Desktop connects (it may happen in other Agents but this was the one where it showed up for me - Claude Code couldn't reproduce it). </p>

<p>There might be other ways to fix this—maybe adjusting FastMCP's schema generation, or handling the <code>anyOf</code> schema differently on the client side. I just know changing the type hints worked for my case.</p>

<p>The fix itself? Five minutes once I found the pattern. The debugging? Two hours of confusion and multiple false starts, until your own software tells you what the problem is! </p>

<p>Dog-fooding works. Eventually.</p>

