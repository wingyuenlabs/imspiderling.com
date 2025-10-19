---
Title: 🗂️ Clprolf Directory Explorer — When Breadth-First Becomes Intuitive
Description: 
Author: Charles Koffler
Date: 2025-10-19T21:24:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Everyone knows that exploring directories can quickly turn into a messy technical exercise:<br>
loops, recursion, stacks, file filters… and code that loses all readability.</p>

<p>With <strong>Clprolf</strong>, clarity is built-in.<br>
You don’t just write <em>code</em> — you design <strong>agents</strong>, <strong>workers</strong>, and <strong>models</strong> that reflect what really happens.<br>
Let’s see how a simple <em>directory explorer</em> can become a beautifully structured program.</p>


<h3>
  
  
  💡 1. The Concept
</h3>

<p>We want to:</p>

<ul>
<li>explore all subdirectories of a given folder,</li>
<li>in <strong>breadth-first</strong> order (level by level),</li>
<li>assign each directory a hierarchical ID,</li>
<li>and display the results cleanly.</li>
</ul>

<p>Instead of mixing logic, display, and data handling,<br>
Clprolf invites us to split them into <strong>roles</strong>.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Declension</th>
<th>Responsibility</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>Launcher</code></td>
<td><code>@Worker_agent</code></td>
<td>Starts the exploration</td>
</tr>
<tr>
<td><code>DirectoryExplorerImpl</code></td>
<td><code>@Agent</code></td>
<td>Performs the exploration</td>
</tr>
<tr>
<td><code>DirectoryExplorerWorkerImpl</code></td>
<td><code>@Worker_agent</code></td>
<td>Displays results</td>
</tr>
<tr>
<td><code>Directory</code></td>
<td><code>@Model</code></td>
<td>Represents one directory node</td>
</tr>
<tr>
<td>
<code>DirectoryExplorer</code> / <code>DirectoryExplorerWorker</code>
</td>
<td><code>@Version_inh</code></td>
<td>Define contracts between roles</td>
</tr>
</tbody>
</table></div>


<h3>
  
  
  ⚙️ 2. The Launcher
</h3>

<p>The entry point is as simple as it looks.<br>
It prepares the environment and delegates the job to the proper agent.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Worker_agent</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">Launcher</span> <span class="o">{</span>

    <span class="kd">public</span> <span class="kd">static</span> <span class="kt">void</span> <span class="nf">main</span><span class="o">(</span><span class="nc">String</span><span class="o">[]</span> <span class="n">args</span><span class="o">)</span> <span class="o">{</span>
        <span class="nd">@With_compat</span> <span class="nc">Path</span> <span class="n">path</span> <span class="o">=</span> <span class="nc">Paths</span><span class="o">.</span><span class="na">get</span><span class="o">(</span><span class="n">args</span><span class="o">.</span><span class="na">length</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="o">?</span> <span class="n">args</span><span class="o">[</span><span class="mi">0</span><span class="o">]</span> <span class="o">:</span> <span class="nc">System</span><span class="o">.</span><span class="na">getProperty</span><span class="o">(</span><span class="s">"user.home"</span><span class="o">));</span>
        <span class="k">try</span> <span class="o">{</span> <span class="n">path</span> <span class="o">=</span> <span class="n">path</span><span class="o">.</span><span class="na">toRealPath</span><span class="o">(</span><span class="nc">LinkOption</span><span class="o">.</span><span class="na">NOFOLLOW_LINKS</span><span class="o">);</span> <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">Exception</span> <span class="n">ignored</span><span class="o">)</span> <span class="o">{}</span>

        <span class="k">if</span> <span class="o">(!</span><span class="nc">Files</span><span class="o">.</span><span class="na">isDirectory</span><span class="o">(</span><span class="n">path</span><span class="o">))</span> <span class="o">{</span>
            <span class="nc">System</span><span class="o">.</span><span class="na">err</span><span class="o">.</span><span class="na">println</span><span class="o">(</span><span class="s">"Not a directory: "</span> <span class="o">+</span> <span class="n">path</span><span class="o">);</span>
            <span class="nc">System</span><span class="o">.</span><span class="na">exit</span><span class="o">(</span><span class="mi">1</span><span class="o">);</span>
        <span class="o">}</span>

        <span class="nd">@With_compat</span> <span class="nc">DirectoryExplorer</span> <span class="n">explorer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DirectoryExplorerImpl</span><span class="o">();</span>
        <span class="n">explorer</span><span class="o">.</span><span class="na">breadthFirstFolders</span><span class="o">(</span><span class="n">path</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>It’s clear who does what:<br>
this worker doesn’t explore — it simply <em>launches the agent</em>.</p>


<h3>
  
  
  🧭 3. The Agent — <code>DirectoryExplorerImpl</code>
</h3>

<p>Here lies the real exploration logic.<br>
The agent collaborates with a worker, manipulates a model, and manages a queue.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Agent</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">DirectoryExplorerImpl</span> <span class="kd">implements</span> <span class="nd">@Contracts</span> <span class="nc">DirectoryExplorer</span> <span class="o">{</span>

    <span class="kd">private</span> <span class="nd">@With_compat</span> <span class="nc">DirectoryExplorerWorker</span> <span class="n">worker</span><span class="o">;</span>

    <span class="kd">public</span> <span class="nf">DirectoryExplorerImpl</span><span class="o">()</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">worker</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DirectoryExplorerWorkerImpl</span><span class="o">();</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">breadthFirstFolders</span><span class="o">(</span><span class="nd">@With_compat</span> <span class="nc">Path</span> <span class="n">directoryPath</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">directoryPath</span> <span class="o">=</span> <span class="n">directoryPath</span><span class="o">.</span><span class="na">normalize</span><span class="o">().</span><span class="na">toAbsolutePath</span><span class="o">();</span>

        <span class="nd">@With_compat</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Directory</span><span class="o">&gt;</span> <span class="n">foldersList</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayList</span><span class="o">&lt;&gt;();</span>
        <span class="nd">@With_compat</span> <span class="nc">Queue</span><span class="o">&lt;</span><span class="nc">Directory</span><span class="o">&gt;</span> <span class="n">directoryToExplore</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LinkedList</span><span class="o">&lt;&gt;();</span>

        <span class="n">directoryToExplore</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="k">new</span> <span class="nc">Directory</span><span class="o">(</span><span class="n">directoryPath</span><span class="o">,</span> <span class="nc">List</span><span class="o">.</span><span class="na">of</span><span class="o">(</span><span class="mi">0</span><span class="o">)));</span>

        <span class="k">while</span> <span class="o">(!</span><span class="n">directoryToExplore</span><span class="o">.</span><span class="na">isEmpty</span><span class="o">())</span> <span class="o">{</span>
            <span class="nc">Directory</span> <span class="n">current</span> <span class="o">=</span> <span class="n">directoryToExplore</span><span class="o">.</span><span class="na">poll</span><span class="o">();</span>
            <span class="n">foldersList</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="n">current</span><span class="o">);</span>

            <span class="nc">File</span><span class="o">[]</span> <span class="n">files</span> <span class="o">=</span> <span class="n">current</span><span class="o">.</span><span class="na">getPath</span><span class="o">().</span><span class="na">toFile</span><span class="o">().</span><span class="na">listFiles</span><span class="o">();</span>
            <span class="k">if</span> <span class="o">(</span><span class="n">files</span> <span class="o">!=</span> <span class="kc">null</span><span class="o">)</span> <span class="o">{</span>
                <span class="kt">int</span> <span class="n">index</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
                <span class="k">for</span> <span class="o">(</span><span class="nc">File</span> <span class="n">file</span> <span class="o">:</span> <span class="n">files</span><span class="o">)</span> <span class="o">{</span>
                    <span class="k">if</span> <span class="o">(</span><span class="n">file</span><span class="o">.</span><span class="na">isDirectory</span><span class="o">())</span> <span class="o">{</span>
                        <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Integer</span><span class="o">&gt;</span> <span class="n">newId</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayList</span><span class="o">&lt;&gt;(</span><span class="n">current</span><span class="o">.</span><span class="na">getHierarchicalId</span><span class="o">());</span>
                        <span class="n">newId</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="n">index</span><span class="o">);</span>
                        <span class="n">directoryToExplore</span><span class="o">.</span><span class="na">add</span><span class="o">(</span><span class="k">new</span> <span class="nc">Directory</span><span class="o">(</span><span class="n">file</span><span class="o">.</span><span class="na">toPath</span><span class="o">().</span><span class="na">normalize</span><span class="o">().</span><span class="na">toAbsolutePath</span><span class="o">(),</span> <span class="n">newId</span><span class="o">));</span>
                        <span class="n">index</span><span class="o">++;</span>
                    <span class="o">}</span>
                <span class="o">}</span>
            <span class="o">}</span>
        <span class="o">}</span>

        <span class="n">worker</span><span class="o">.</span><span class="na">displayResult</span><span class="o">(</span><span class="n">foldersList</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Every part is crystal clear:</p>

<ul>
<li>The queue defines a <strong>breadth-first traversal</strong>.</li>
<li>Each directory receives its own <strong>hierarchical ID</strong>.</li>
<li>The <strong>worker</strong> takes care of presentation.</li>
</ul>

<p>No recursion, no confusion.</p>




<h3>
  
  
  👷 4. The Worker Agent
</h3>

<p>Responsible for showing the result, not for computing it.<br>
Again, we separate <em>doing</em> from <em>showing</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Worker_agent</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">DirectoryExplorerWorkerImpl</span> <span class="kd">implements</span> <span class="nd">@Contracts</span> <span class="nc">DirectoryExplorerWorker</span> <span class="o">{</span>

    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">displayResult</span><span class="o">(</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">Directory</span><span class="o">&gt;</span> <span class="n">foldersList</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">for</span> <span class="o">(</span><span class="nc">Directory</span> <span class="n">dir</span> <span class="o">:</span> <span class="n">foldersList</span><span class="o">)</span> <span class="o">{</span>
            <span class="nc">String</span> <span class="n">display</span> <span class="o">=</span> <span class="n">formatId</span><span class="o">(</span><span class="n">dir</span><span class="o">.</span><span class="na">getHierarchicalId</span><span class="o">())</span> <span class="o">+</span> <span class="s">" : "</span> <span class="o">+</span> <span class="n">dir</span><span class="o">.</span><span class="na">getPath</span><span class="o">();</span>
            <span class="nc">System</span><span class="o">.</span><span class="na">out</span><span class="o">.</span><span class="na">println</span><span class="o">(</span><span class="n">display</span><span class="o">);</span>
        <span class="o">}</span>
    <span class="o">}</span>

    <span class="kd">private</span> <span class="nc">String</span> <span class="nf">formatId</span><span class="o">(</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">Integer</span><span class="o">&gt;</span> <span class="n">id</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">return</span> <span class="s">"("</span> <span class="o">+</span> <span class="nc">String</span><span class="o">.</span><span class="na">join</span><span class="o">(</span><span class="s">", "</span><span class="o">,</span> <span class="n">id</span><span class="o">.</span><span class="na">stream</span><span class="o">().</span><span class="na">map</span><span class="o">(</span><span class="nl">String:</span><span class="o">:</span><span class="n">valueOf</span><span class="o">).</span><span class="na">toArray</span><span class="o">(</span><span class="nc">String</span><span class="o">[]::</span><span class="k">new</span><span class="o">))</span> <span class="o">+</span> <span class="s">")"</span><span class="o">;</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Simple, explicit, human-readable.</p>




<h3>
  
  
  📦 5. The Model
</h3>

<p>A <code>@Model</code> in Clprolf is always clear:<br>
it represents data, and nothing more.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Model</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">Directory</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="nc">Path</span> <span class="n">path</span><span class="o">;</span>
    <span class="kd">private</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Integer</span><span class="o">&gt;</span> <span class="n">hierarchicalId</span><span class="o">;</span>

    <span class="kd">public</span> <span class="nf">Directory</span><span class="o">(</span><span class="nc">Path</span> <span class="n">path</span><span class="o">,</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Integer</span><span class="o">&gt;</span> <span class="n">id</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">path</span> <span class="o">=</span> <span class="n">path</span><span class="o">;</span>
        <span class="k">this</span><span class="o">.</span><span class="na">hierarchicalId</span> <span class="o">=</span> <span class="n">id</span><span class="o">;</span>
    <span class="o">}</span>

    <span class="kd">public</span> <span class="nc">Path</span> <span class="nf">getPath</span><span class="o">()</span> <span class="o">{</span> <span class="k">return</span> <span class="n">path</span><span class="o">;</span> <span class="o">}</span>
    <span class="kd">public</span> <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Integer</span><span class="o">&gt;</span> <span class="nf">getHierarchicalId</span><span class="o">()</span> <span class="o">{</span> <span class="k">return</span> <span class="n">hierarchicalId</span><span class="o">;</span> <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>No logic, no side effects. Just structure.</p>




<h3>
  
  
  🤝 6. The Contracts
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Agent</span>
<span class="nd">@Version_inh</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">DirectoryExplorer</span> <span class="o">{</span>
    <span class="kt">void</span> <span class="nf">breadthFirstFolders</span><span class="o">(</span><span class="nc">Path</span> <span class="n">directoryPath</span><span class="o">);</span>
<span class="o">}</span>

<span class="nd">@Worker_agent</span>
<span class="nd">@Version_inh</span>
<span class="kd">public</span> <span class="kd">interface</span> <span class="nc">DirectoryExplorerWorker</span> <span class="o">{</span>
    <span class="kt">void</span> <span class="nf">displayResult</span><span class="o">(</span><span class="nc">List</span><span class="o">&lt;</span><span class="nc">Directory</span><span class="o">&gt;</span> <span class="n">foldersList</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Contracts make the collaboration explicit.<br>
No hidden dependencies, no tight coupling — just clear communication.</p>




<h3>
  
  
  🪶 7. Why it matters
</h3>

<p>Breadth-first exploration is only the example.<br>
What matters here is <strong>how naturally the architecture expresses itself</strong>:</p>

<ul>
<li>The launcher launches.</li>
<li>The agent explores.</li>
<li>The worker displays.</li>
<li>The model represents.</li>
<li>The contract binds.</li>
</ul>

<p>Clprolf doesn’t just help you code — it helps you <strong>think</strong>.<br>
The structure emerges from the intention.</p>




<h3>
  
  
  ✨ Final Thoughts
</h3>

<blockquote>
<p>In classical Java, you might have written a single class doing everything.</p>

<p>In Clprolf, each role finds its natural place.<br>
The result is simple, explicit, and readable — even for someone who never wrote Java before.</p>
</blockquote>

<p>Clprolf brings <strong>clarity back into architecture</strong>,<br>
and even the smallest utilities become examples of well-designed software.</p>




