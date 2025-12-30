---
Title: avro-phonetic-go: Avro-style Banglish to বাংলা transliteration in Go
Description: 
Author: Mehedi Hasan Sajib
Date: 2025-12-29T21:23:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you work with Bengali users, you already know the problem.</p>

<p>People type Bangla using Latin letters. Not carefully. Not consistently.<br>
Yet they expect your application to understand them.</p>

<p>The Avro Phonetic keyboard showed that this problem can be solved using<br>
a grammar-driven approach: pattern matching combined with local context rules.</p>

<p>This article introduces <strong>avro-phonetic-go</strong>, a Go library that implements<br>
an Avro-style phonetic transliteration engine with a clean, production-oriented API.</p>
<h2>
  
  
  Design goals
</h2>

<p>The library was built with the following principles:</p>

<ul>
<li>Grammar-driven, not hardcoded</li>
<li>Deterministic output using longest-match scanning</li>
<li>Explicit rule evaluation (prefix and suffix constraints)</li>
<li>No magic heuristics in strict mode</li>
<li>Optional BD mode for modern Bangladeshi typing shortcuts</li>
</ul>

<p>The overall idea is credited to the Avro Phonetic keyboard.<br>
The internal design of this library was strongly influenced by<br>
the PHP reference implementation <a href="https://github.com/imerfanahmed/avro-php" rel="noopener noreferrer">https://github.com/imerfanahmed/avro-php</a></p>
<h2>
  
  
  Quick example
</h2>

<p>Strict mode:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">avrophonetic</span><span class="o">.</span><span class="n">To</span><span class="p">(</span><span class="s">"ami bangla gan gai"</span><span class="p">))</span>
<span class="c">// আমি বাংলা গান গাই</span>
</code></pre>

</div>



<p>BD mode (opt-in):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">avrophonetic</span><span class="o">.</span><span class="n">ToBD</span><span class="p">(</span><span class="s">"tmi valo"</span><span class="p">))</span>
<span class="c">// তুমি ভালো</span>
</code></pre>

</div>



<h2>
  
  
  How it works
</h2>

<p>At a high level, the engine works in four steps:</p>

<ol>
<li>Load a grammar consisting of patterns and rules</li>
<li>Build a trie for fast longest-match lookup</li>
<li>Scan the input left to right</li>
<li>Validate candidate patterns using local context rules</li>
</ol>

<p>Rules are evaluated only against immediate neighbors.<br>
There is no backtracking, which keeps the algorithm predictable and fast.</p>
<h2>
  
  
  Strict mode vs BD mode
</h2>

<p>Strict mode is intended to behave as a clean Avro-style baseline.<br>
If you want exact behavior, use strict mode only.</p>

<p>BD mode layers a small set of additional patterns on top of strict grammar.<br>
These patterns capture real-world Bangladeshi typing habits<br>
without polluting the base grammar.</p>

<p>This separation keeps the engine usable for both compatibility<br>
and user-experience–oriented applications.</p>
<h2>
  
  
  Custom grammar support
</h2>

<p>The engine is fully grammar-driven.</p>

<p>You can load a complete grammar JSON file to reach parity with the grammar you prefer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">g</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">avrophonetic</span><span class="o">.</span><span class="n">FromGrammarFile</span><span class="p">(</span><span class="s">"grammar.json"</span><span class="p">)</span>
<span class="n">a</span> <span class="o">:=</span> <span class="n">avrophonetic</span><span class="o">.</span><span class="n">New</span><span class="p">(</span><span class="n">avrophonetic</span><span class="o">.</span><span class="n">WithGrammar</span><span class="p">(</span><span class="n">g</span><span class="p">))</span>
<span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">a</span><span class="o">.</span><span class="n">Parse</span><span class="p">(</span><span class="s">"ami"</span><span class="p">))</span>
</code></pre>

</div>



<p>This makes the library suitable for search indexing, chat processing,<br>
form normalization, and NLP pipelines.</p>

<h2>
  
  
  Credits
</h2>

<ul>
<li>Avro Phonetic keyboard: original idea and grammar concept</li>
<li>PHP reference implementation: <a href="https://github.com/imerfanahmed/avro-php" rel="noopener noreferrer">https://github.com/imerfanahmed/avro-php</a>
</li>
</ul>

<p>This project is an independent Go implementation and is not affiliated with the original Avro project.</p>

<h2>
  
  
  Closing thoughts
</h2>

<p>Transliteration is a small feature with outsized impact.</p>

<p>If your product handles Bengali user input,<br>
doing this well is not optional anymore.</p>

<p>Repository: <a href="https://github.com/mhshajib/avro-phonetic-go" rel="noopener noreferrer">https://github.com/mhshajib/avro-phonetic-go</a><br>
Documentation and examples are available in the README.</p>

