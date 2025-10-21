---
Title: Mastering JSON the Go Way
Description: 
Author: Hugo Oliveira
Date: 2025-10-21T21:54:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>One of the things I love about Go is how practical it is when dealing with JSON.<br>
No complicated setup. No magic. Just clear, predictable code.</p>

<p>The <code>encoding/json</code> package provides <strong>two main ways</strong> to work with JSON:</p>

<p><strong>- In-memory (Marshal/Unmarshal)</strong><br>
<strong>- Stream-based (Encoder/Decoder)</strong></p>

<p>Let’s break it down.</p>

<p><strong>1. <code>Marshal</code> and <code>Unmarshal</code> — for small or simple data</strong></p>

<p>These are the most common functions when your JSON fits comfortably in memory.</p>

<p>▶️ <strong>Go → JSON</strong> (<code>Marshal</code>)</p>

<p>You convert a Go struct, map, or slice into JSON.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">User</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Name</span>  <span class="kt">string</span> <span class="s">`json:"name"`</span>
    <span class="n">Email</span> <span class="kt">string</span> <span class="s">`json:"email"`</span>
<span class="p">}</span>

<span class="n">user</span> <span class="o">:=</span> <span class="n">User</span><span class="p">{</span><span class="s">"Hugo"</span><span class="p">,</span> <span class="s">"hugo@example.com"</span><span class="p">}</span>
<span class="n">data</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">json</span><span class="o">.</span><span class="n">Marshal</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
<span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
    <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="n">err</span><span class="p">)</span>
<span class="p">}</span>
<span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="kt">string</span><span class="p">(</span><span class="n">data</span><span class="p">))</span>
</code></pre>

</div>



<p><strong>Output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"name"</span><span class="p">:</span><span class="s2">"Hugo"</span><span class="p">,</span><span class="nl">"email"</span><span class="p">:</span><span class="s2">"hugo@example.com"</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p><strong>Use <code>Marshal</code> when:</strong></p>

<ul>
<li>You already have a Go object in memory.</li>
<li>You want to send or save it as JSON (HTTP response, file, etc.).</li>
<li>The data is not too large (fits easily in memory).</li>
</ul>

<p>◀️<strong>JSON → Go</strong> (<code>Unmarshal</code>)</p>

<p>You convert JSON bytes into a Go struct or map.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">jsonData</span> <span class="o">:=</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="s">`{"name":"Hugo","email":"hugo@example.com"}`</span><span class="p">)</span>

<span class="k">var</span> <span class="n">user</span> <span class="n">User</span>
<span class="n">err</span> <span class="o">:=</span> <span class="n">json</span><span class="o">.</span><span class="n">Unmarshal</span><span class="p">(</span><span class="n">jsonData</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">user</span><span class="p">)</span>
<span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
    <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="n">err</span><span class="p">)</span>
<span class="p">}</span>

<span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">user</span><span class="o">.</span><span class="n">Name</span><span class="p">)</span>  <span class="c">// Hugo</span>
</code></pre>

</div>



<p><strong>Use <code>Unmarshal</code> when:</strong></p>

<ul>
<li>You receive JSON as a string or from a file and want to turn it into a Go type.</li>
<li>The JSON is fully available in memory.</li>
</ul>

<p>✅ <strong>Tip</strong>:<br>
Always pass a pointer to the destination (<code>&amp;user</code>), so Go can modify the variable.</p>

<p><strong>2. <code>Encoder</code> and <code>Decoder</code> — for streams or large data</strong></p>

<p>When working with <strong>large JSON files</strong> or <strong>network streams</strong>, loading everything into memory isn’t efficient.<br>
That’s where <code>Encoder</code> and <code>Decoder</code> come in — they work with <strong>streams</strong> (<code>io.Reader</code> and <code>io.Writer</code>).</p>

<p>▶️<strong>Go → JSON stream (<code>Encoder</code>)</strong><br>
You write JSON directly to an output, like a file or HTTP response.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">user</span> <span class="o">:=</span> <span class="n">User</span><span class="p">{</span><span class="s">"Hugo"</span><span class="p">,</span> <span class="s">"hugo@example.com"</span><span class="p">}</span>
<span class="n">file</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">os</span><span class="o">.</span><span class="n">Create</span><span class="p">(</span><span class="s">"user.json"</span><span class="p">)</span>
<span class="k">defer</span> <span class="n">file</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

<span class="n">encoder</span> <span class="o">:=</span> <span class="n">json</span><span class="o">.</span><span class="n">NewEncoder</span><span class="p">(</span><span class="n">file</span><span class="p">)</span>
<span class="n">encoder</span><span class="o">.</span><span class="n">SetIndent</span><span class="p">(</span><span class="s">""</span><span class="p">,</span> <span class="s">"  "</span><span class="p">)</span> <span class="c">// optional, for pretty-print</span>
<span class="n">encoder</span><span class="o">.</span><span class="n">Encode</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Use <code>Encoder</code> when:</strong></p>

<ul>
<li>You want to write JSON directly to a file, socket, or HTTP response.</li>
<li>You want to avoid keeping the full JSON in memory.</li>
<li>You’re writing continuous data (e.g., logs, API responses).</li>
</ul>

<p>◀️ <strong>JSON stream → Go (<code>Decoder</code>)</strong></p>

<p>You read JSON progressively from an input, like a file or HTTP request body.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">file</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">os</span><span class="o">.</span><span class="n">Open</span><span class="p">(</span><span class="s">"user.json"</span><span class="p">)</span>
<span class="k">defer</span> <span class="n">file</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

<span class="k">var</span> <span class="n">user</span> <span class="n">User</span>
<span class="n">decoder</span> <span class="o">:=</span> <span class="n">json</span><span class="o">.</span><span class="n">NewDecoder</span><span class="p">(</span><span class="n">file</span><span class="p">)</span>
<span class="n">decoder</span><span class="o">.</span><span class="n">Decode</span><span class="p">(</span><span class="o">&amp;</span><span class="n">user</span><span class="p">)</span>

<span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Use <code>Decoder</code> when:</strong></p>

<ul>
<li>You’re reading large JSON data.</li>
<li>The data is coming from a network or file stream.</li>
<li>You don’t want to load everything at once into memory.</li>
</ul>

<p><strong>Summary</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Direction</strong></th>
<th><strong>Data Type</strong></th>
<th><strong>Function</strong></th>
<th><strong>Best for</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>JSON → Go</td>
<td><code>[]byte</code></td>
<td><code>Unmarshal</code></td>
<td>Small data in memory</td>
</tr>
<tr>
<td>Go → JSON</td>
<td><code>[]byte</code></td>
<td><code>Marshal</code></td>
<td>Small data in memory</td>
</tr>
<tr>
<td>JSON → Go</td>
<td><code>io.Reader</code></td>
<td><code>Decoder</code></td>
<td>Large files or streams</td>
</tr>
<tr>
<td>Go → JSON</td>
<td><code>io.Writer</code></td>
<td><code>Encoder</code></td>
<td>Large files or streams</td>
</tr>
</tbody>
</table></div>

<p>Go’s <code>encoding/json</code> package is a perfect example of the language’s philosophy:<br>
<strong>simple, explicit, and reliable</strong>.</p>

<p>When you need something quick and small, use <code>Marshal</code> and <code>Unmarshal</code>.<br>
When you’re dealing with streams, files, or large payloads, use <code>Encoder</code> and <code>Decoder</code>.</p>

<p>You get both simplicity and performance, and you always know exactly what your code is doing.</p>

<p>That’s Go.</p>

