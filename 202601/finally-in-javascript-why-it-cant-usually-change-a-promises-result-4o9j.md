---
Title: finally() in JavaScript: Why It Can't (Usually) Change a Promise's Result
Description: 
Author: Aleksandra Dudkina
Date: 2026-01-04T21:49:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>A common misconception is that <code>finally()</code> behaves like <code>then()</code>.<br><br>
In reality, it doesn’t receive values, ignores return values, and only affects a promise in very specific cases. Let’s see why.</p>
<h1>
  
  
  <code>finally()</code>: Why It Can’t Change a Promise Result
</h1>
<h2>
  
  
  <code>finally()</code> doesn’t receive arguments
</h2>

<p>Let's start with a quick quiz to test your understanding:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">myPromise</span><span class="p">.</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">return</span> <span class="dl">"</span><span class="s2">Hi, Sasha!</span><span class="dl">"</span><span class="p">)</span>
<span class="p">.</span><span class="k">finally</span><span class="p">(</span><span class="nx">res</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">res</span> <span class="o">+</span> <span class="dl">"</span><span class="s2">Whats up?</span><span class="dl">"</span><span class="p">))</span> 

<span class="c1">//what will appear in console?</span>
</code></pre>

</div>



<p>If you think the answer is “Hi, Sasha!Whats up?” then keep reading 🙂</p>

<h2>
  
  
  How finally() works under the hood
</h2>

<p>The <code>finally</code> method doesn't have its own low-level implementation in the Promise specification.</p>

<p>Under the hood, it simply delegates to <code>then</code>, using the same callback for both success and failure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">promise</span><span class="p">.</span><span class="k">finally</span><span class="p">(</span><span class="nx">onDone</span><span class="p">);</span>
<span class="c1">// is roughly equivalent to</span>
<span class="nx">promise</span><span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">onDone</span><span class="p">,</span> <span class="nx">onDone</span><span class="p">);</span>
</code></pre>

</div>



<p>Because of this <code>onDone</code> <strong>does not receive any arguments (</strong>no resolved value, no rejection reason)</p>

<p>And what happens to the parameter value which wasn’t passed? Correct: it stays <strong>undefined</strong>.</p>

<p>Correct answer to the quiz is <strong>“undefinedWhats up?“</strong> as <code>res</code> value is <strong>undefined.</strong></p>

<h2>
  
  
  What happens to the Promise value after <code>finally()</code>?
</h2>

<p>Another quiz:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">myPromise</span><span class="p">.</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">return</span> <span class="dl">"</span><span class="s2">Hi, Sasha!</span><span class="dl">"</span><span class="p">)</span>
<span class="p">.</span><span class="k">finally</span><span class="p">(</span><span class="nx">res</span> <span class="o">=&gt;</span> <span class="k">return</span> <span class="dl">"</span><span class="s2">my new result</span><span class="dl">"</span><span class="p">)</span>
<span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">res</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">res</span><span class="p">))</span> <span class="c1">//what will be logged here?</span>
</code></pre>

</div>



<p>The answer is "Hi, Sasha!"</p>

<p>The value returned from <code>finally()</code> is simply <strong>ignored</strong>.</p>

<p>This is a key rule: <code>finally()</code> <strong><em>doesn’t change Promise’s state.</em></strong></p>

<p>It exists for <strong>side effects only</strong> — cleanup logic, metrics, etc.</p>

<h2>
  
  
  Why the returned value from <code>finally()</code> is ignored
</h2>

<ul>
<li><p><code>finally()</code> doesn’t receive the resolved value</p></li>
<li><p>it can’t pass a new value down the chain</p></li>
<li><p>the Promise continues with the value from the last <code>then()</code> or <code>catch()</code></p></li>
</ul>

<p>So this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">.</span><span class="k">finally</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="dl">"</span><span class="s2">new value</span><span class="dl">"</span><span class="p">)</span>
</code></pre>

</div>



<p>is effectively the same as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">.</span><span class="k">finally</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{})</span>
</code></pre>

</div>



<h3>
  
  
  Takeaways so far
</h3>

<ul>
<li><p>Since <code>finally()</code> doesn't get any arguments, any value given to it stays <strong>undefined</strong>.</p></li>
<li><p>The return value from a <code>finally()</code> callback is <strong>ignored</strong> and <strong>doesn't change the Promise's state</strong> or the next <code>then()</code> chain</p></li>
<li><p>The Promise keeps the value from the last <code>then()</code> or <code>catch()</code> before <code>finally()</code> was called.</p></li>
</ul>

<h2>
  
  
  Can <code>finally()</code> change the Promise state?
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="dl">"</span><span class="s2">OK</span><span class="dl">"</span><span class="p">)</span>
  <span class="p">.</span><span class="k">finally</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Something went wrong</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">})</span>
  <span class="p">.</span><span class="nf">then</span><span class="p">((</span><span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">res</span><span class="p">))</span>
  <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">err</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">err</span><span class="p">));</span>

<span class="c1">//what is the Promise state at this point?</span>
<span class="c1">//what will be logged? "OK" or "Something went wrong"?</span>
</code></pre>

</div>



<p>At first glance, you might expect <code>"OK"</code>.</p>

<p>After all, we already know that <code>finally()</code> doesn’t change a Promise’s value, right?</p>

<p>But in this case, the output will be “Something went wrong” and the Promise ends up <strong>rejected</strong>.</p>

<h3>
  
  
  Why does this happen?
</h3>

<p>The important clarification is this:</p>

<blockquote>
<p><code>finally()</code> does not change a Promise’s state by default.</p>
</blockquote>

<p>However, if an error is thrown inside <code>finally()</code> — or if it returns a rejected Promise — that error <strong>overrides the previous state</strong>.</p>

<p>Here is another quick quiz:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="dl">"</span><span class="s2">OK</span><span class="dl">"</span><span class="p">)</span>
  <span class="p">.</span><span class="k">finally</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="dl">"</span><span class="s2">Something went wrong</span><span class="dl">"</span><span class="p">)</span>
  <span class="p">})</span>
  <span class="p">.</span><span class="nf">then</span><span class="p">((</span><span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">res</span><span class="p">))</span>
  <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">err</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">err</span><span class="p">));</span>

<span class="c1">//what is the Promise state at this point?</span>
<span class="c1">//what will be logged? "OK" or "Something went wrong"?</span>
</code></pre>

</div>



<p>We are not throwing any error here. However, we are still getting the <strong>"Something went wrong"</strong> here.</p>

<p>Another way to change a Promise’s state is to <strong>return a rejected Promise from</strong> <code>finally()</code>.</p>

<p>The takeaway we had previously is that the return value from a <code>finally()</code> callback is <strong>ignored, so why returning rejected Promise work?</strong></p>

<p>Let’s investigate what happens under the hood.</p>

<p>Remember how finally() works?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">promise</span><span class="p">.</span><span class="k">finally</span><span class="p">(</span><span class="nx">onDone</span><span class="p">);</span>
<span class="c1">// is roughly equivalent to</span>
<span class="nx">promise</span><span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">onDone</span><span class="p">,</span> <span class="nx">onDone</span><span class="p">);</span>
</code></pre>

</div>



<p>Knowing that, let’s investigate our case:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="dl">"</span><span class="s2">OK</span><span class="dl">"</span><span class="p">)</span>
  <span class="p">.</span><span class="k">finally</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="dl">"</span><span class="s2">Something went wrong</span><span class="dl">"</span><span class="p">)</span>
  <span class="p">})</span>
  <span class="p">.</span><span class="nf">then</span><span class="p">((</span><span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">res</span><span class="p">))</span>
  <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">err</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">err</span><span class="p">));</span>

<span class="c1">//is roughly the same as:</span>


<span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span><span class="dl">"</span><span class="s2">OK</span><span class="dl">"</span><span class="p">)</span>
<span class="c1">//this is our finally method</span>
<span class="p">.</span><span class="nf">then</span><span class="p">(</span>

<span class="c1">// first onDone callback</span>
  <span class="nx">value</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span>
    <span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="k">return</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="dl">"</span><span class="s2">Something went wrong</span><span class="dl">"</span><span class="p">);</span> <span class="p">})()</span>
  <span class="p">).</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">value</span><span class="p">),</span> <span class="c1">// if resolves, continue with the previous value</span>

<span class="c1">// second onDone callback</span>
  <span class="nx">reason</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">resolve</span><span class="p">(</span>
    <span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="k">return</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="dl">"</span><span class="s2">Something went wrong</span><span class="dl">"</span><span class="p">);</span> <span class="p">})()</span>
<span class="c1">//throwing error</span>
  <span class="p">).</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="k">throw</span> <span class="nx">reason</span><span class="p">;</span> <span class="p">})</span> <span class="c1">// if gets rejected, continue with rejection reason as value</span>
<span class="p">)</span>
<span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">res</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">res</span><span class="p">))</span> <span class="c1">//this is ignored </span>
<span class="p">.</span><span class="k">catch</span><span class="p">(</span><span class="nx">err</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">err</span><span class="p">));</span> <span class="c1">//this is called </span>
</code></pre>

</div>



<p>Remember that throwing error changes Promise’s state?</p>

<p>This is what is done behind the scenes when we return a rejected Promise.</p>

<p>Note that if you return a new <strong>resolved promise</strong> from <code>finally()</code>, <strong>nothing changes</strong>. The chain waits for it to resolve, and <code>then()</code> continues with the <strong>original value</strong>.</p>

<h2>
  
  
  Final takeaways
</h2>

<ul>
<li><p>Since <code>finally()</code> doesn't receive any arguments, any value provided to it remains <strong>undefined</strong>.</p></li>
<li><p>The return value from a <code>finally()</code> callback is <strong>ignored</strong> and does not alter the Promise's state or affect the subsequent <code>then()</code> chain.</p></li>
<li>
<p>The Promise retains the value from the last <code>then()</code> or <code>catch()</code> before <code>finally()</code> was invoked, unless:</p>

<ul>
<li>The method returns a rejected Promise.</li>
<li>An error is thrown within the method.</li>
</ul>


</li>

</ul>

