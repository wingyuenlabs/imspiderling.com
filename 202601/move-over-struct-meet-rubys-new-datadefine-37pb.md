---
Title: Move Over, Struct: Meet Ruby's New Data.define
Description: 
Author: Zil Norvilis
Date: 2026-01-14T20:50:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>With the introduction of Ruby 3.2, <strong><code>Data.define</code> is now the preferred way to handle DTOs (Data Transfer Objects)</strong> in modern Ruby applications.</p>

<p>While <code>Struct</code> has been the go-to for years, it has several quirks that make it less-than-ideal for pure data transfer. <code>Data</code> was specifically designed to be a "Value Object" factory.</p>

<p>Here is a deep dive into <code>Data.define</code>, how it works, and how it compares to <code>Struct</code>.</p>




<h3>
  
  
  What is <code>Data.define</code>?
</h3>

<p><code>Data</code> is a class factory (similar to <code>Struct</code>) used to define simple, immutable objects.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Define a Data class</span>
<span class="no">Address</span> <span class="o">=</span> <span class="no">Data</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="ss">:city</span><span class="p">,</span> <span class="ss">:zip</span><span class="p">)</span>

<span class="c1"># Initialize it</span>
<span class="n">home</span> <span class="o">=</span> <span class="no">Address</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="ss">city: </span><span class="s2">"Vilnius"</span><span class="p">,</span> <span class="ss">zip: </span><span class="s2">"01100"</span><span class="p">)</span>

<span class="c1"># Access data</span>
<span class="n">home</span><span class="p">.</span><span class="nf">city</span> <span class="c1"># =&gt; "Vilnius"</span>
</code></pre>

</div>



<h3>
  
  
  Key Features of <code>Data.define</code>
</h3>

<h4>
  
  
  1. Immutable by Design
</h4>

<p>The biggest difference between <code>Data</code> and <code>Struct</code> is that <code>Data</code> objects are <strong>immutable</strong>. There are no setter methods.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Point</span> <span class="o">=</span> <span class="no">Data</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="ss">:x</span><span class="p">,</span> <span class="ss">:y</span><span class="p">)</span>
<span class="n">p1</span> <span class="o">=</span> <span class="no">Point</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="ss">x: </span><span class="mi">1</span><span class="p">,</span> <span class="ss">y: </span><span class="mi">2</span><span class="p">)</span>

<span class="n">p1</span><span class="p">.</span><span class="nf">x</span> <span class="o">=</span> <span class="mi">10</span> 
<span class="c1"># NoMethodError (undefined method `x=' for #&lt;data Point x=1, y=2&gt;)</span>
</code></pre>

</div>



<p>In a DTO context, immutability is a massive advantage. It ensures that data remains consistent as it is passed through different layers of your application.</p>

<h4>
  
  
  2. Flexible Initialization (Positional or Keywords)
</h4>

<p><code>Data</code> is smarter about how it accepts arguments. You can use positional arguments or keyword arguments out of the box.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Point</span> <span class="o">=</span> <span class="no">Data</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="ss">:x</span><span class="p">,</span> <span class="ss">:y</span><span class="p">)</span>

<span class="c1"># Both work:</span>
<span class="n">p1</span> <span class="o">=</span> <span class="no">Point</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
<span class="n">p2</span> <span class="o">=</span> <span class="no">Point</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="ss">x: </span><span class="mi">1</span><span class="p">,</span> <span class="ss">y: </span><span class="mi">2</span><span class="p">)</span>
</code></pre>

</div>



<p><em>Note: If you use keywords, they are checked. Passing a typo (e.g., <code>z: 3</code>) will raise an <code>ArgumentError</code>.</em></p>

<h4>
  
  
  3. Value Equality
</h4>

<p>Two different instances of a <code>Data</code> object are considered equal if their values are equal.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="n">p1</span> <span class="o">=</span> <span class="no">Point</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>
<span class="n">p2</span> <span class="o">=</span> <span class="no">Point</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">)</span>

<span class="n">p1</span> <span class="o">==</span> <span class="n">p2</span> <span class="c1"># =&gt; true</span>
<span class="n">p1</span><span class="p">.</span><span class="nf">eql?</span><span class="p">(</span><span class="n">p2</span><span class="p">)</span> <span class="c1"># =&gt; true</span>
</code></pre>

</div>



<h4>
  
  
  4. Pattern Matching Integration
</h4>

<p><code>Data</code> objects work perfectly with Ruby’s pattern matching (<code>case/in</code>).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">case</span> <span class="n">p1</span>
<span class="k">in</span> <span class="no">Point</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span> <span class="k">if</span> <span class="n">x</span> <span class="o">&gt;</span> <span class="mi">0</span>
  <span class="nb">puts</span> <span class="s2">"X is positive: </span><span class="si">#{</span><span class="n">x</span><span class="si">}</span><span class="s2">"</span>
<span class="k">end</span>
</code></pre>

</div>



<h4>
  
  
  5. Adding Methods
</h4>

<p>Just like <code>Struct</code>, you can pass a block to <code>Data.define</code> to add custom logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Money</span> <span class="o">=</span> <span class="no">Data</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="ss">:amount</span><span class="p">,</span> <span class="ss">:currency</span><span class="p">)</span> <span class="k">do</span>
  <span class="k">def</span> <span class="nf">to_s</span>
    <span class="s2">"</span><span class="si">#{</span><span class="n">amount</span><span class="si">}</span><span class="s2"> </span><span class="si">#{</span><span class="n">currency</span><span class="si">}</span><span class="s2">"</span>
  <span class="k">end</span>
<span class="k">end</span>

<span class="nb">puts</span> <span class="no">Money</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="mi">100</span><span class="p">,</span> <span class="s2">"USD"</span><span class="p">).</span><span class="nf">to_s</span> <span class="c1"># =&gt; "100 USD"</span>
</code></pre>

</div>






<h3>
  
  
  <code>Data</code> vs. <code>Struct</code>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>
<code>Data.define</code> (Ruby 3.2+)</th>
<th><code>Struct</code></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Mutability</strong></td>
<td>
<strong>Immutable</strong> (Read-only)</td>
<td>
<strong>Mutable</strong> (Read/Write)</td>
</tr>
<tr>
<td><strong>Initialization</strong></td>
<td>Positional OR Keywords</td>
<td>Positional (Keywords must be enabled)</td>
</tr>
<tr>
<td><strong>Enumerable</strong></td>
<td>No</td>
<td>Yes (has <code>.each</code>, <code>.map</code>, etc.)</td>
</tr>
<tr>
<td><strong>Intention</strong></td>
<td>Value Objects / DTOs</td>
<td>Lightweight "Property" classes</td>
</tr>
<tr>
<td><strong>Interface</strong></td>
<td>Minimal (Clean)</td>
<td>Large (Inherits many methods)</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  When to use <code>Struct</code> instead?
</h3>

<p>Even though <code>Data</code> is usually better for DTOs, <code>Struct</code> is still useful if:</p>

<ol>
<li>
<strong>You need mutability:</strong> If you are using the object as a temporary "scratchpad" where you update values frequently.</li>
<li>
<strong>You need Enumerable:</strong> If you want to call <code>.each</code> or <code>.select</code> directly on the object's attributes.</li>
<li>
<strong>Legacy Ruby:</strong> If your environment is running Ruby version 3.1 or older.</li>
</ol>

<h3>
  
  
  Summary: Why I use <code>Data.define</code> for DTOs
</h3>

<p>In a typical Rails or Dry-Ruby stack, DTOs are used to move data from a Service Object to a View or from an API response to a Model. </p>

<p><strong>I use <code>Data.define</code> because:</strong></p>

<ol>
<li>It prevents accidental side effects (someone changing a value halfway through a request).</li>
<li>The code is more "honest"—the object is just data.</li>
<li>The built-in support for keyword arguments makes the code much more readable than positional <code>Struct</code> arguments.</li>
</ol>

<p><strong>Example of a modern DTO:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">UserDTO</span> <span class="o">=</span> <span class="no">Data</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="ss">:id</span><span class="p">,</span> <span class="ss">:email</span><span class="p">,</span> <span class="ss">:role</span><span class="p">)</span>

<span class="c1"># In a controller or service</span>
<span class="n">user_data</span> <span class="o">=</span> <span class="no">UserDTO</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="ss">id: </span><span class="n">user</span><span class="p">.</span><span class="nf">id</span><span class="p">,</span> <span class="ss">email: </span><span class="n">user</span><span class="p">.</span><span class="nf">email</span><span class="p">,</span> <span class="ss">role: </span><span class="n">user</span><span class="p">.</span><span class="nf">role</span><span class="p">)</span>
</code></pre>

</div>



