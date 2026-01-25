---
Title: Building a CLI Tool with Cognee: Lessons from 5 Epics
Description: 
Author: Armel BOBDA
Date: 2026-01-25T21:27:28.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs0v53p32k9cfp9vplqb7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs0v53p32k9cfp9vplqb7.png" alt="Building a CLI Tool with Cognee: Lessons from 5 Epics" width="800" height="436"></a></p>

<p>I just finished building <a href="https://github.com/armelhbobdad/sentinel" rel="noopener noreferrer">Sentinel</a>, a CLI tool that uses Cognee to detect energy conflicts in personal schedules. Five development epics. 860+ tests. Four critical bugs were found and fixed.</p>

<p>Along the way, I learned a lot about working with Cognee that isn't in the documentation. This article shares those lessons so you can avoid my mistakes.</p>




<h2>
  
  
  What I Built
</h2>

<p>Sentinel analyses schedule text and builds a knowledge graph to find "energy collisions"—situations where a draining activity (dinner with a difficult relative) precedes a demanding one (important presentation).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>sentinel <span class="nb">paste</span> &lt; schedule.txt
✓ Extracted 7 entities
Found 6 relationships.
✓ Graph saved to ~/.local/share/sentinel/graph.db

<span class="nv">$ </span>sentinel check
⚠️  COLLISION DETECTED                    Confidence: 85%

<span class="o">[</span>Aunt Susan] <span class="nt">--DRAINS--</span><span class="o">&gt;</span> <span class="o">(</span>drained<span class="o">)</span>
                            |
                     CONFLICTS_WITH
                            |
                      <span class="o">(</span>focused<span class="o">)</span> &lt;<span class="nt">--REQUIRES--</span> <span class="o">[</span>Strategy Presentation]
</code></pre>

</div>



<p>The tool uses Cognee for entity extraction and relationship building, then applies custom collision detection logic on top.</p>

<p>Here's what I learned.</p>




<h2>
  
  
  Lesson 1: Use CYPHER, Not GRAPH_COMPLETION
</h2>

<p>This one cost me hours of debugging.</p>

<p><strong>The mistake:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># DON'T DO THIS for graph extraction
</span><span class="n">results</span> <span class="o">=</span> <span class="k">await</span> <span class="n">cognee</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span>
    <span class="n">SearchType</span><span class="p">.</span><span class="n">GRAPH_COMPLETION</span><span class="p">,</span>
    <span class="n">query_text</span><span class="o">=</span><span class="sh">"</span><span class="s">*</span><span class="sh">"</span>
<span class="p">)</span>
</code></pre>

</div>



<p>My unit tests (with mocked Cognee) passed. Production extracted zero entities.</p>

<p><strong>The problem:</strong> <code>GRAPH_COMPLETION</code> returns LLM-generated prose, not structured graph data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"The schedule contains a dinner event with Aunt Susan on Sunday,
which is described as emotionally draining..."
</code></pre>

</div>



<p>Useful for chat interfaces. Useless for graph algorithms.</p>

<p><strong>The fix:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">cognee.api.v1.search</span> <span class="kn">import</span> <span class="n">SearchType</span>

<span class="c1"># Get nodes
</span><span class="n">node_results</span> <span class="o">=</span> <span class="k">await</span> <span class="n">cognee</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span>
    <span class="n">query_text</span><span class="o">=</span><span class="sh">"</span><span class="s">MATCH (n) RETURN n</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">query_type</span><span class="o">=</span><span class="n">SearchType</span><span class="p">.</span><span class="n">CYPHER</span><span class="p">,</span>
<span class="p">)</span>

<span class="c1"># Get edges
</span><span class="n">edge_results</span> <span class="o">=</span> <span class="k">await</span> <span class="n">cognee</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span>
    <span class="n">query_text</span><span class="o">=</span><span class="sh">"</span><span class="s">MATCH (a)-[r]-&gt;(b) RETURN a, r, b</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">query_type</span><span class="o">=</span><span class="n">SearchType</span><span class="p">.</span><span class="n">CYPHER</span><span class="p">,</span>
<span class="p">)</span>
</code></pre>

</div>



<p><strong>Takeaway:</strong> If you need structured graph data for programmatic use, always use <code>SearchType.CYPHER</code> with explicit Cypher queries.</p>




<h2>
  
  
  Lesson 2: Cognee Results Are Deeply Nested
</h2>

<p>When you get Cypher results back, don't expect a flat list of nodes.</p>

<p><strong>Actual structure:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">results</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">{</span>
        <span class="sh">'</span><span class="s">search_result</span><span class="sh">'</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">[</span>
                <span class="p">[</span><span class="n">node1_data</span><span class="p">],</span>  <span class="c1"># &lt;-- Your actual node is here
</span>                <span class="p">[</span><span class="n">node2_data</span><span class="p">],</span>
                <span class="bp">...</span>
            <span class="p">]</span>
        <span class="p">]</span>
    <span class="p">}</span>
<span class="p">]</span>
</code></pre>

</div>



<p><strong>Access pattern:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">extract_nodes</span><span class="p">(</span><span class="n">results</span><span class="p">):</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">results</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">[]</span>

    <span class="n">nodes</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="n">search_result</span> <span class="o">=</span> <span class="n">results</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">search_result</span><span class="sh">'</span><span class="p">,</span> <span class="p">[])</span>

    <span class="k">if</span> <span class="n">search_result</span><span class="p">:</span>
        <span class="n">node_list</span> <span class="o">=</span> <span class="n">search_result</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>  <span class="c1"># First level unwrap
</span>        <span class="k">for</span> <span class="n">node_wrapper</span> <span class="ow">in</span> <span class="n">node_list</span><span class="p">:</span>
            <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">node_wrapper</span><span class="p">,</span> <span class="nb">list</span><span class="p">)</span> <span class="ow">and</span> <span class="n">node_wrapper</span><span class="p">:</span>
                <span class="n">node</span> <span class="o">=</span> <span class="n">node_wrapper</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>  <span class="c1"># Second level unwrap
</span>                <span class="n">nodes</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">node</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">nodes</span>
</code></pre>

</div>



<p><strong>Takeaway:</strong> Write robust extraction helpers and test them against real Cognee output, not mocks.</p>




<h2>
  
  
  Lesson 3: Filter to Entity Nodes Only
</h2>

<p>Cognee's graph contains multiple node types. Not all of them are what you want.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Node Type</th>
<th>What It Is</th>
<th>Keep?</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>Entity</code></td>
<td>Actual entities from your text</td>
<td>✅ Yes</td>
</tr>
<tr>
<td><code>DocumentChunk</code></td>
<td>Text segments</td>
<td>❌ No</td>
</tr>
<tr>
<td><code>EntityType</code></td>
<td>Category definitions</td>
<td>❌ No</td>
</tr>
<tr>
<td><code>TextDocument</code></td>
<td>Source document metadata</td>
<td>❌ No</td>
</tr>
<tr>
<td><code>TextSummary</code></td>
<td>LLM-generated summaries</td>
<td>❌ No</td>
</tr>
</tbody>
</table></div>

<p><strong>Filter pattern:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">extract_entities</span><span class="p">(</span><span class="n">nodes</span><span class="p">):</span>
    <span class="k">return</span> <span class="p">[</span>
        <span class="n">node</span> <span class="k">for</span> <span class="n">node</span> <span class="ow">in</span> <span class="n">nodes</span>
        <span class="k">if</span> <span class="n">node</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">type</span><span class="sh">'</span><span class="p">)</span> <span class="o">==</span> <span class="sh">'</span><span class="s">Entity</span><span class="sh">'</span>
    <span class="p">]</span>
</code></pre>

</div>



<p>Without this filter, your graph will be cluttered with infrastructure nodes that aren't useful for domain logic.</p>




<h2>
  
  
  Lesson 4: Properties Are JSON Strings
</h2>

<p>Cognee returns node properties as JSON strings, not Python dicts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># What you get
</span><span class="n">node</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">'</span><span class="s">id</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">abc-123</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">Aunt Susan</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">type</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">Entity</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">properties</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">{</span><span class="sh">"</span><span class="s">description</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">Family member</span><span class="sh">"</span><span class="s">, </span><span class="sh">"</span><span class="s">entity_type</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">PERSON</span><span class="sh">"</span><span class="s">}</span><span class="sh">'</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Parse them:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>

<span class="k">def</span> <span class="nf">parse_properties</span><span class="p">(</span><span class="n">node</span><span class="p">):</span>
    <span class="n">props</span> <span class="o">=</span> <span class="n">node</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">properties</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">{}</span><span class="sh">'</span><span class="p">)</span>
    <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">props</span><span class="p">,</span> <span class="nb">str</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">props</span><span class="p">)</span>
        <span class="k">except</span> <span class="n">json</span><span class="p">.</span><span class="n">JSONDecodeError</span><span class="p">:</span>
            <span class="k">return</span> <span class="p">{}</span>
    <span class="k">return</span> <span class="n">props</span> <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">props</span><span class="p">,</span> <span class="nb">dict</span><span class="p">)</span> <span class="k">else</span> <span class="p">{}</span>
</code></pre>

</div>






<h2>
  
  
  Lesson 5: The LLM Will Generate Unexpected Relation Types
</h2>

<p>This was my biggest surprise. I expected Cognee to use consistent relation type names. Instead:</p>

<p><strong>What I expected:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>DRAINS, REQUIRES, INVOLVES, SCHEDULED_AT
</code></pre>

</div>



<p><strong>What I got (sampling from multiple runs):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>drains, depletes, exhausts, causes_fatigue,
emotionally_draining, negatively_impacts,
is_emotionally_draining, energy_draining,
leads_to_exhaustion, causes_exhaustion...
</code></pre>

</div>



<p>Eleven variations for one concept. Per run.</p>

<p><strong>Why this happens:</strong> Cognee's LLM extraction has no ontology constraints. The model generates semantically correct but lexically variable relation names.</p>

<p><strong>The fix:</strong> Build a normalisation layer. I wrote a 3-tier matching system:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Tier 1: Exact match dictionary (85+ entries)
</span><span class="n">RELATION_MAP</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">drains</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">depletes</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">exhausts</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># ...
</span><span class="p">}</span>

<span class="c1"># Tier 2: Keyword matching (stems)
</span><span class="n">KEYWORDS</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">DRAINS</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="sh">"</span><span class="s">drain</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">exhaust</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">deplet</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">fatigue</span><span class="sh">"</span><span class="p">],</span>
    <span class="c1"># ...
</span><span class="p">}</span>

<span class="c1"># Tier 3: Fuzzy matching (RapidFuzz)
</span><span class="kn">from</span> <span class="n">rapidfuzz</span> <span class="kn">import</span> <span class="n">fuzz</span><span class="p">,</span> <span class="n">process</span>
<span class="c1"># Match against candidate phrases
</span></code></pre>

</div>



<p><strong>Takeaway:</strong> Don't assume LLM output will be consistent. Build robust normalisation for any categorical data coming from Cognee.</p>

<blockquote>
<p><em>I wrote a full deep-dive on this pattern: <a href="https://dev.to/armelhbobdad/taming-llm-output-chaos-a-3-tier-normalisation-pattern-41jm">Taming LLM Output Chaos: A 3-Tier Normalisation Pattern</a></em></p>
</blockquote>




<h2>
  
  
  Lesson 6: Custom Prompts Change Everything
</h2>

<p>Cognee's <code>cognify()</code> function accepts a <code>custom_prompt</code> parameter. This was the key to getting domain-specific relationships.</p>

<p><strong>Default behavior:</strong></p>

<ul>
<li>Generic entity extraction</li>
<li>Relations like <code>involves</code>, <code>about</code>, <code>scheduled_at</code>
</li>
<li>No energy-domain relationships (<code>DRAINS</code>, <code>REQUIRES</code>)</li>
</ul>

<p><strong>With custom prompt:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">EXTRACTION_PROMPT</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
You are extracting a PERSONAL ENERGY knowledge graph.

**REQUIRED RELATIONSHIP TYPES** (use ONLY these):
- DRAINS: Activity depletes energy/focus
- REQUIRES: Activity needs energy/focus
- CONFLICTS_WITH: Energy state conflicts with requirement
- SCHEDULED_AT: Activity occurs at time
- INVOLVES: Activity includes person/thing

**COLLISION PATTERN** (create when applicable):
[draining_activity] --DRAINS--&gt; (energy_state) --CONFLICTS_WITH--&gt;
[requiring_activity] --REQUIRES--&gt; (resource)

**EXAMPLE**:
Input: </span><span class="sh">"</span><span class="s">Sunday: Draining dinner. Monday: Important presentation.</span><span class="sh">"</span><span class="s">
Graph:
- [dinner] --DRAINS--&gt; (emotional_energy)
- (emotional_energy) --CONFLICTS_WITH--&gt; [presentation]
- [presentation] --REQUIRES--&gt; (sharp_focus)
</span><span class="sh">"""</span>

<span class="k">await</span> <span class="n">cognee</span><span class="p">.</span><span class="nf">cognify</span><span class="p">(</span><span class="n">custom_prompt</span><span class="o">=</span><span class="n">EXTRACTION_PROMPT</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Results:</strong></p>

<ul>
<li>Before custom prompt: ~20% collision detection rate</li>
<li>After custom prompt: ~70% edge type accuracy (still needed normalisation)</li>
<li>After prompt + normalisation: 100% collision detection</li>
</ul>

<p><strong>Takeaway:</strong> Don't fight Cognee's defaults. Guide them with domain-specific prompts that include examples and explicit relationship ontologies.</p>




<h2>
  
  
  Lesson 7: Node IDs Vary Too (Semantic Consolidation)
</h2>

<p>Even with good prompts and relation normalisation, I had one more problem:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Run 1: [dinner] --DRAINS--&gt; (emotional_exhaustion)
Run 2: [dinner] --DRAINS--&gt; (low_energy)
Run 3: [dinner] --DRAINS--&gt; (drained_state)
</code></pre>

</div>



<p>Same concept, different node labels. My BFS collision detection couldn't find paths because it was doing exact string matching on node IDs.</p>

<p><strong>The fix:</strong> Semantic node consolidation using RapidFuzz:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">rapidfuzz</span> <span class="kn">import</span> <span class="n">fuzz</span>

<span class="k">def</span> <span class="nf">group_similar_nodes</span><span class="p">(</span><span class="n">nodes</span><span class="p">,</span> <span class="n">threshold</span><span class="o">=</span><span class="mi">70</span><span class="p">):</span>
    <span class="n">groups</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">node</span> <span class="ow">in</span> <span class="n">nodes</span><span class="p">:</span>
        <span class="n">merged</span> <span class="o">=</span> <span class="bp">False</span>
        <span class="k">for</span> <span class="n">group</span> <span class="ow">in</span> <span class="n">groups</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">fuzz</span><span class="p">.</span><span class="nc">WRatio</span><span class="p">(</span><span class="n">node</span><span class="p">.</span><span class="n">label</span><span class="p">,</span> <span class="n">group</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">label</span><span class="p">)</span> <span class="o">&gt;=</span> <span class="n">threshold</span><span class="p">:</span>
                <span class="n">group</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">node</span><span class="p">)</span>
                <span class="n">merged</span> <span class="o">=</span> <span class="bp">True</span>
                <span class="k">break</span>
        <span class="k">if</span> <span class="ow">not</span> <span class="n">merged</span><span class="p">:</span>
            <span class="n">groups</span><span class="p">.</span><span class="nf">append</span><span class="p">([</span><span class="n">node</span><span class="p">])</span>
    <span class="k">return</span> <span class="n">groups</span>

<span class="k">def</span> <span class="nf">consolidate</span><span class="p">(</span><span class="n">graph</span><span class="p">):</span>
    <span class="n">groups</span> <span class="o">=</span> <span class="nf">group_similar_nodes</span><span class="p">(</span><span class="n">graph</span><span class="p">.</span><span class="n">nodes</span><span class="p">)</span>
    <span class="c1"># Pick canonical representative, rewrite edge references
</span>    <span class="c1"># ...
</span></code></pre>

</div>



<p><strong>Takeaway:</strong> LLM variability affects both relation types AND node identity. Handle both.</p>




<h2>
  
  
  Lesson 8: Mocked Tests Will Lie to You
</h2>

<p>I had 178 tests passing. All green. Two critical bugs in production.</p>

<p><strong>Bug 1:</strong> <code>SearchType.GRAPH_COMPLETION</code> returned prose instead of graph data. My mock returned what I <em>expected</em> Cognee to return, not what it <em>actually</em> returns.</p>

<p><strong>Bug 2:</strong> Rich console interpreted <code>[node labels]</code> as style markup. My tests didn't render through the actual Rich console.</p>

<p><strong>The fix:</strong> Live API tests.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@pytest.mark.live</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">test_real_entity_extraction</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Verify actual Cognee behavior.</span><span class="sh">"""</span>
    <span class="n">engine</span> <span class="o">=</span> <span class="nc">CogneeEngine</span><span class="p">()</span>
    <span class="n">graph</span> <span class="o">=</span> <span class="k">await</span> <span class="n">engine</span><span class="p">.</span><span class="nf">ingest</span><span class="p">(</span><span class="sh">"</span><span class="s">Dinner with Aunt Susan on Sunday</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">assert</span> <span class="nf">len</span><span class="p">(</span><span class="n">graph</span><span class="p">.</span><span class="n">nodes</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">,</span> <span class="sh">"</span><span class="s">No entities extracted</span><span class="sh">"</span>
    <span class="n">labels</span> <span class="o">=</span> <span class="p">{</span><span class="n">n</span><span class="p">.</span><span class="n">label</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span> <span class="k">for</span> <span class="n">n</span> <span class="ow">in</span> <span class="n">graph</span><span class="p">.</span><span class="n">nodes</span><span class="p">}</span>
    <span class="k">assert</span> <span class="nf">any</span><span class="p">(</span><span class="sh">"</span><span class="s">susan</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">l</span> <span class="k">for</span> <span class="n">l</span> <span class="ow">in</span> <span class="n">labels</span><span class="p">)</span>
</code></pre>

</div>



<p>Run them manually before marking stories "done":<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Requires API key</span>
uv run pytest tests/live/ <span class="nt">-m</span> live <span class="nt">-v</span>

<span class="c"># Skip in CI</span>
uv run pytest <span class="nt">-m</span> <span class="s2">"not live"</span>
</code></pre>

</div>



<p><strong>Takeaway:</strong> For LLM integrations, unit tests with mocks are necessary but not sufficient. Add live API tests for critical paths.</p>




<h2>
  
  
  Lesson 9: Suppress Cognee's Logging (But Keep a Debug Mode)
</h2>

<p>Cognee produces verbose output during normal operation. Great for debugging, annoying for users.</p>

<p><strong>Solution:</strong> Lazy import with suppression:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">contextlib</span> <span class="kn">import</span> <span class="n">redirect_stdout</span><span class="p">,</span> <span class="n">redirect_stderr</span>
<span class="kn">from</span> <span class="n">io</span> <span class="kn">import</span> <span class="n">StringIO</span>

<span class="k">def</span> <span class="nf">get_engine</span><span class="p">():</span>
    <span class="k">with</span> <span class="nf">redirect_stdout</span><span class="p">(</span><span class="nc">StringIO</span><span class="p">()),</span> <span class="nf">redirect_stderr</span><span class="p">(</span><span class="nc">StringIO</span><span class="p">()):</span>
        <span class="kn">import</span> <span class="n">warnings</span>
        <span class="k">with</span> <span class="n">warnings</span><span class="p">.</span><span class="nf">catch_warnings</span><span class="p">():</span>
            <span class="n">warnings</span><span class="p">.</span><span class="nf">simplefilter</span><span class="p">(</span><span class="sh">"</span><span class="s">ignore</span><span class="sh">"</span><span class="p">)</span>
            <span class="kn">from</span> <span class="n">sentinel.core.engine</span> <span class="kn">import</span> <span class="n">CogneeEngine</span>
    <span class="k">return</span> <span class="nc">CogneeEngine</span><span class="p">()</span>
</code></pre>

</div>



<p><strong>But keep a debug flag:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@click.option</span><span class="p">(</span><span class="sh">'</span><span class="s">--debug</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">-d</span><span class="sh">'</span><span class="p">,</span> <span class="n">is_flag</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">main</span><span class="p">(</span><span class="n">debug</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">debug</span><span class="p">:</span>
        <span class="n">engine</span> <span class="o">=</span> <span class="nc">CogneeEngine</span><span class="p">()</span>  <span class="c1"># Normal import, verbose
</span>    <span class="k">else</span><span class="p">:</span>
        <span class="n">engine</span> <span class="o">=</span> <span class="nf">get_engine</span><span class="p">()</span>  <span class="c1"># Suppressed
</span></code></pre>

</div>






<h2>
  
  
  The Journey: 5 Epics in Numbers
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Development epics</td>
<td>5</td>
</tr>
<tr>
<td>Stories completed</td>
<td>37</td>
</tr>
<tr>
<td>Tests written</td>
<td>860+</td>
</tr>
<tr>
<td>Critical bugs found</td>
<td>4</td>
</tr>
<tr>
<td>Relation type mappings</td>
<td>85+</td>
</tr>
<tr>
<td>Collision detection rate</td>
<td>15% → 100%</td>
</tr>
</tbody>
</table></div>

<p><strong>The architecture that worked:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User Input
    ↓
Cognee Extraction (custom prompt)
    ↓
3-Tier Relation Mapping (exact → keyword → fuzzy)
    ↓
Semantic Node Consolidation (RapidFuzz grouping)
    ↓
BFS Collision Detection
    ↓
Rich Terminal Output
</code></pre>

</div>



<p>Each layer handles a different source of LLM variability.</p>




<h2>
  
  
  Key Takeaways for Cognee Users
</h2>

<ol>
<li><p><strong>Use <code>SearchType.CYPHER</code></strong> for structured graph data, not <code>GRAPH_COMPLETION</code>.</p></li>
<li><p><strong>Expect nested results.</strong> Write robust extraction helpers.</p></li>
<li><p><strong>Filter to Entity nodes.</strong> Cognee returns infrastructure nodes too.</p></li>
<li><p><strong>Parse JSON properties.</strong> They come as strings.</p></li>
<li><p><strong>Normalise relation types.</strong> The LLM will surprise you.</p></li>
<li><p><strong>Use custom prompts.</strong> Domain-specific ontologies need explicit guidance.</p></li>
<li><p><strong>Consolidate semantically equivalent nodes.</strong> IDs vary like relation types.</p></li>
<li><p><strong>Add live API tests.</strong> Mocks don't catch integration bugs.</p></li>
<li><p><strong>Suppress verbose logging.</strong> But keep a debug mode.</p></li>
</ol>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<a href="https://github.com/armelhbobdad/sentinel" rel="noopener noreferrer">Sentinel on GitHub</a> - The complete implementation</li>
<li>
<a href="https://docs.cognee.ai" rel="noopener noreferrer">Cognee Documentation</a> - Official docs</li>
<li>
<a href="https://github.com/rapidfuzz/RapidFuzz" rel="noopener noreferrer">RapidFuzz</a> - Fuzzy string matching library</li>
</ul>

<h2>
  
  
  Related Articles
</h2>

<ul>
<li>
<a href="https://medium.com/@armelhbobdad/why-your-calendar-app-misses-the-real-conflicts-39f980bb1791" rel="noopener noreferrer">Why Your Calendar App Misses the Real Conflicts</a> - The "why" behind Sentinel</li>
<li>
<a href="https://dev.to/armelhbobdad/taming-llm-output-chaos-a-3-tier-normalisation-pattern-41jm">Taming LLM Output Chaos: A 3-Tier Normalization Pattern</a> - Deep-dive on handling LLM variability</li>
</ul>




<p><em>Built for the <a href="https://discord.com/channels/1120795297094832337/1317073613446185074" rel="noopener noreferrer">Cognee Mini Challenge 2026 - January Edition</a>. Happy building!</em></p>

