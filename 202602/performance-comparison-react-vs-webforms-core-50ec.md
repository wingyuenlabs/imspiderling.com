---
Title: Performance Comparison: React vs WebForms Core
Description: 
Author: Elanat Framework
Date: 2026-02-21T22:11:34.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Focus on Network Requests, Bandwidth Consumption, and Client Execution Model
</h2>

<p>In modern web architectures, performance is not only about rendering speed. A critical factor is the <strong>communication pattern with the server and the volume of transferred data</strong>.</p>

<p>This article compares <strong>React</strong> and <strong><a href="https://github.com/webforms-core" rel="noopener noreferrer">WebForms Core</a> (WFC)</strong> strictly from a performance and network perspective, focusing on:</p>

<ul>
<li>Initial request count</li>
<li>Secondary request count</li>
<li>Initial download size</li>
<li>Bandwidth consumption during interactions</li>
<li>Client execution model</li>
<li>React’s need for build and JSX</li>
<li>HTML structure handling in WFC</li>
</ul>

<p>This is not a general pros/cons comparison — it is a technical, architectural performance analysis.</p>




<h1>
  
  
  1️⃣ Initial Load Phase
</h1>

<h2>
  
  
  React
</h2>

<p>In a typical React application, the following occurs:</p>

<ol>
<li>Download of JavaScript bundle (often hundreds of KB to multiple MBs)</li>
<li>Download of React runtime</li>
<li>Execution of hydration or mounting</li>
<li>DOM generation through JavaScript</li>
</ol>

<p>Even in optimized projects:</p>

<ul>
<li>One initial HTML request</li>
<li>Multiple JS requests</li>
<li>CSS and other assets</li>
</ul>

<p>As a result:</p>

<ul>
<li>📦 Initial payload is usually larger</li>
<li>🌐 Higher number of initial requests</li>
<li>⚙ Initial rendering depends on JavaScript execution</li>
</ul>




<h2>
  
  
  WebForms Core
</h2>

<p>In WFC:</p>

<ul>
<li>A standard HTML page is sent from the server</li>
<li>Optionally, a lightweight WebFormsJS file</li>
<li>No heavy bundle</li>
<li>No mandatory build pipeline</li>
</ul>

<p>In the simplest case:</p>

<ul>
<li>One HTML request</li>
<li>One small JS request</li>
</ul>

<p>Result:</p>

<ul>
<li>📦 Smaller initial payload</li>
<li>🌐 Fewer initial requests</li>
<li>⚙ Page contains usable HTML even before JS executes</li>
</ul>




<h1>
  
  
  2️⃣ User Interactions
</h1>

<p>This is where architectural differences become more significant.</p>




<h2>
  
  
  Scenario A: Pure Client-Side Interaction
</h2>

<h3>
  
  
  WFC Example (KeyUp without server call)
</h3>

<p>Server code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">void</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
<span class="p">{</span>       
    <span class="n">WebForms</span> <span class="n">form</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">WebForms</span><span class="p">();</span>

    <span class="n">form</span><span class="p">.</span><span class="nf">SetCommentEvent</span><span class="p">(</span><span class="s">"TextBox1"</span><span class="p">,</span> <span class="n">HtmlEvent</span><span class="p">.</span><span class="n">OnKeyUp</span><span class="p">,</span> <span class="s">"keyup"</span><span class="p">);</span>
    <span class="n">form</span><span class="p">.</span><span class="nf">StartIndex</span><span class="p">(</span><span class="s">"keyup"</span><span class="p">);</span>
    <span class="n">form</span><span class="p">.</span><span class="nf">SetBackgroundColor</span><span class="p">(</span><span class="s">"TextBox1"</span><span class="p">,</span> <span class="n">Fetch</span><span class="p">.</span><span class="nf">GetValue</span><span class="p">(</span><span class="s">"TextBox1"</span><span class="p">));</span>

    <span class="nf">Write</span><span class="p">(</span><span class="n">form</span><span class="p">.</span><span class="nf">ExportToHtmlComment</span><span class="p">());</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Server output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="c">&lt;!--[web-forms]
EbTextBox1=onkeyup|keyup|
#=keyup
bcTextBox1=@$vTextBox1--&gt;</span>
<span class="nt">&lt;input</span> <span class="na">id=</span><span class="s">"TextBox1"</span> <span class="na">type=</span><span class="s">"text"</span> <span class="nt">/&gt;</span>
</code></pre>

</div>



<p>In this case:</p>

<ul>
<li>No request is sent to the server</li>
<li>WebFormsJS parses the comment</li>
<li>The event is registered in the client</li>
<li>DOM is updated directly</li>
</ul>

<p>Execution model:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Event → Direct DOM manipulation
</code></pre>

</div>



<p>Requests: 0<br>
Secondary bandwidth: 0</p>


<h3>
  
  
  React Equivalent
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="kd">function</span> <span class="nf">App</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">color</span><span class="p">,</span> <span class="nx">setColor</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">input</span>
      <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">setColor</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)</span><span class="si">}</span>
      <span class="na">style</span><span class="p">=</span><span class="si">{</span><span class="p">{</span> <span class="na">backgroundColor</span><span class="p">:</span> <span class="nx">color</span> <span class="p">}</span><span class="si">}</span>
    <span class="p">/&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>


<p>Execution model:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Event → setState → Virtual DOM diff → Patch DOM
</code></pre>

</div>



<p>Requests: 0<br>
Secondary bandwidth: 0</p>

<p>However, internally React:</p>

<ul>
<li>Maintains state</li>
<li>Executes Virtual DOM diffing</li>
<li>Triggers re-render cycles</li>
</ul>

<p>WFC in this scenario:</p>

<ul>
<li>Automatic state with server state engine</li>
<li>No diff algorithm</li>
<li>Direct DOM update</li>
</ul>

<p>From a CPU perspective in simple cases, WFC introduces less abstraction overhead.</p>


<h2>
  
  
  Scenario B: Interaction with Server
</h2>
<h3>
  
  
  React
</h3>

<p>A typical form submission in React:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="nf">fetch</span><span class="p">(</span><span class="dl">"</span><span class="s2">/api/contact</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span> <span class="na">body</span><span class="p">:</span> <span class="nx">data</span> <span class="p">})</span>
</code></pre>

</div>



<p>Standard SPA pattern:</p>

<ul>
<li>AJAX request</li>
<li>JSON response</li>
<li>State update</li>
<li>Re-render</li>
</ul>

<p>In larger apps:</p>

<ul>
<li>Multiple API endpoints</li>
<li>Parallel requests</li>
<li>Polling or WebSocket connections</li>
</ul>

<p>React applications are generally API-driven, meaning frequent server communication.</p>




<h3>
  
  
  WebForms Core
</h3>

<p>WFC preserves the traditional form model:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>POST → server → INI response → DOM patch
</code></pre>

</div>



<p>The key distinction:</p>

<p>The server does not return full HTML.<br>
It returns compact command instructions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[web-forms]
sd(button)=1
nt&lt;form&gt;=h3
bc&lt;h3&gt;=green
st&lt;h3&gt;=Message
</code></pre>

</div>



<p>In this case:</p>

<ul>
<li>Only change instructions are sent</li>
<li>No full HTML re-render</li>
<li>No large JSON state tree</li>
</ul>

<p>Bandwidth consumption is lower than full page reloads, and typically lighter than API-heavy SPA responses.</p>




<h1>
  
  
  3️⃣ Request Patterns in Real Applications
</h1>

<h2>
  
  
  React (SPA Model)
</h2>

<p>Common behavior:</p>

<ul>
<li>Large bundle downloaded once</li>
<li>Then dozens or hundreds of API calls</li>
<li>Fully dependent on backend APIs</li>
</ul>

<p>Benefit: No full page reload<br>
Cost: Continuous API dependency</p>




<h2>
  
  
  WebForms Core
</h2>

<p>Two possible patterns:</p>

<ol>
<li>Pure client-side interaction (no server calls)</li>
<li>Lightweight roundtrip form-based interaction</li>
</ol>

<p>In many simple UI cases:</p>

<ul>
<li>No server request is required</li>
</ul>

<p>In data-driven cases:</p>

<ul>
<li>Fewer calls than API-heavy SPAs</li>
<li>Interaction is often form-based rather than endpoint-driven</li>
</ul>




<h1>
  
  
  4️⃣ Bandwidth Consumption
</h1>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Aspect</th>
<th>React</th>
<th>WebForms Core</th>
</tr>
</thead>
<tbody>
<tr>
<td>Initial download</td>
<td>Large (bundle)</td>
<td>Small</td>
</tr>
<tr>
<td>Client-only interaction</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<td>Server interaction</td>
<td>JSON + state updates</td>
<td>Compact instruction commands</td>
</tr>
<tr>
<td>API dependency</td>
<td>High</td>
<td>Optional</td>
</tr>
</tbody>
</table></div>




<h1>
  
  
  5️⃣ JSX and Build Requirement
</h1>

<h2>
  
  
  React
</h2>

<p>React is rarely used without a build process.</p>

<p>Why?</p>

<ul>
<li>JSX must be transpiled</li>
<li>Modules must be bundled</li>
<li>Optimizations must be applied</li>
</ul>

<p>In React:</p>

<ul>
<li>HTML is written inside JavaScript</li>
<li>UI structure depends on runtime</li>
<li>Without JS, nothing renders</li>
</ul>




<h2>
  
  
  WebForms Core
</h2>

<ul>
<li>HTML remains standard</li>
<li>No JSX</li>
<li>No mandatory build step</li>
<li>View is independent and inspectable</li>
</ul>

<p>Server logic remains separate from HTML markup.</p>




<h1>
  
  
  6️⃣ Development Model Perspective
</h1>

<p>React:</p>

<ul>
<li>UI = function(state)</li>
<li>State-driven</li>
<li>Re-render-driven</li>
<li>Data-centric architecture</li>
</ul>

<p>WebForms Core:</p>

<ul>
<li>UI = HTML</li>
<li>Behavior = commands</li>
<li>Event-driven</li>
<li>Command-based execution</li>
</ul>




<h1>
  
  
  7️⃣ Final Technical Summary
</h1>

<p>If the evaluation focuses strictly on network and bandwidth:</p>

<h3>
  
  
  React
</h3>

<ul>
<li>Larger initial payload</li>
<li>API-heavy interaction model</li>
<li>Client-side state engine</li>
</ul>

<h3>
  
  
  WebForms Core
</h3>

<ul>
<li>Smaller initial payload</li>
<li>Ability to operate without server calls</li>
<li>Compact instruction-based server responses</li>
<li>No diff engine required</li>
</ul>

<p><strong>Summary of Network &amp; Structural Behavior</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fha0l4vkmrlqxm8sugr2s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fha0l4vkmrlqxm8sugr2s.png" alt="React and WebForms Core Performance Comparison" width="800" height="584"></a></p>




<h1>
  
  
  Conclusion
</h1>

<p>React represents a full SPA architecture with a client-side state engine, requiring bundling, build pipelines, and extensive API usage.</p>

<p>WebForms Core represents a hybrid model that can:</p>

<ul>
<li>Operate fully client-side</li>
<li>Or communicate with minimal bandwidth using compact command instructions</li>
<li>Avoid complex client state engines</li>
<li>Eliminate mandatory build pipelines</li>
</ul>

<p>From a network and bandwidth perspective:</p>

<p>WebForms Core tends to be lighter in traditional, form-centric applications.<br>
React provides stronger data-flow control in large, API-driven applications.</p>

