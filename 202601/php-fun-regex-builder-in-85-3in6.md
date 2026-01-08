---
Title: PHP fun: regex builder in 8.5
Description: 
Author: david duymelinck
Date: 2026-01-08T22:06:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>I just saw following post.</p>


<div class="ltag__link">
  <a href="/a7mdfre7at" class="ltag__link__link">
    <div class="ltag__link__pic">
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F2443278%2Ff053cb1b-880d-48e2-8a09-b41ab5bb47a8.png" alt="a7mdfre7at">
    </div>
  </a>
  <a href="https://dev.to/a7mdfre7at/stop-writing-unreadable-regex-build-patterns-the-fluent-way-in-c-413p" class="ltag__link__link">
    <div class="ltag__link__content">
      <h2>Stop Writing Unreadable Regex - Build Patterns the Fluent Way in C#</h2>
      <h3>Ahmad Al-Freihat ・ Jan 8</h3>
      <div class="ltag__link__taglist">
        <span class="ltag__link__tag">#csharp</span>
        <span class="ltag__link__tag">#regex</span>
        <span class="ltag__link__tag">#dotnet</span>
        <span class="ltag__link__tag">#opensource</span>
      </div>
    </div>
  </a>
</div>


<p>And the first thing I thought was, this has pipe operator written all over it.</p>

<h2>
  
  
  Why?
</h2>

<p>The main reason for me is this is a builder that creates a text so why should it need to instantiate an object?</p>

<p>The second reason is that the build-in PHP regex methods should be used, instead of wrapping then in an object method.<br>
<code>duplicates.IsMatch('test')</code> versus <code>preg_match($pattern, 'test')</code>.</p>
<h2>
  
  
  How will it work?
</h2>

<p>I'm not going to create the whole library, I'm going to highlight a few of the possibilities of the C# library in their pipe operator form.</p>

<p>The start is simply an empty string, but it can be a string with a delimiter or even a delimiter function.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$pattern</span> <span class="o">=</span> <span class="s1">''</span> <span class="o">|&gt;</span> <span class="nf">anyCharacter</span><span class="p">(</span><span class="mf">...</span><span class="p">);</span> <span class="c1">// regex: .*</span>

<span class="c1">// or</span>

<span class="nv">$pattern</span> <span class="o">=</span> <span class="s1">'/'</span> <span class="o">|&gt;</span> <span class="nf">anyCharacter</span><span class="p">(</span><span class="mf">...</span><span class="p">);</span>

<span class="c1">// or </span>

<span class="nv">$pattern</span> <span class="o">=</span> <span class="nf">delimiter</span><span class="p">()</span> <span class="o">|&gt;</span> <span class="nf">anyCharacter</span><span class="p">(</span><span class="mf">...</span><span class="p">);</span>
</code></pre>

</div>



<p>The library uses class constants, <code>Pattern.With.LowercaseLetter</code>, to add known character patterns. This can be a backed enum in PHP, and the <code>anyCharacter</code> function can become <code>any</code> with an argument.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="n">enum</span> <span class="nc">CharacterPattern</span><span class="o">:</span> <span class="n">string</span>
<span class="p">{</span>
   <span class="k">case</span> <span class="nc">Any</span> <span class="o">=</span> <span class="s1">'.'</span><span class="p">;</span>
   <span class="k">case</span> <span class="nc">LowercaseLetter</span> <span class="o">=</span> <span class="s1">'[a-z]'</span><span class="p">;</span>
   <span class="k">case</span> <span class="nc">Word</span> <span class="o">=</span> <span class="s1">'\w'</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">function</span> <span class="n">any</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$pattern</span><span class="p">,</span> <span class="kt">CharacterPattern</span><span class="o">|</span><span class="n">string</span> <span class="nv">$add</span> <span class="o">=</span> <span class="nc">CharacterPattern</span><span class="o">::</span><span class="nc">Any</span><span class="p">):</span> <span class="kt">string</span>
<span class="p">{</span>
  <span class="nv">$addPattern</span> <span class="o">=</span> <span class="nv">$add</span> <span class="k">instanceof</span> <span class="nc">CharacterPattern</span> <span class="o">?</span> <span class="nv">$add</span><span class="o">-&gt;</span><span class="n">value</span><span class="p">;</span>

  <span class="k">return</span> <span class="s2">"</span><span class="nv">$pattern$addPattern</span><span class="s2">*"</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// examples</span>

<span class="nv">$pattern</span> <span class="o">=</span> <span class="s1">''</span> <span class="o">|&gt;</span> <span class="p">(</span><span class="k">fn</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">any</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">));</span>
<span class="nv">$pattern</span> <span class="o">=</span> <span class="s1">''</span> <span class="o">|&gt;</span> <span class="p">(</span><span class="k">fn</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">any</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">,</span> <span class="nc">CharacterPattern</span><span class="o">::</span><span class="nc">LowercaseLetter</span><span class="p">));</span>
<span class="nv">$pattern</span> <span class="o">=</span> <span class="s1">''</span> <span class="o">|&gt;</span> <span class="p">(</span><span class="k">fn</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">any</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">,</span> <span class="s1">'[sunday|monday]'</span><span class="p">));</span>
</code></pre>

</div>



<p>For the last example the library needed a method, <code>literal</code>, type hinting is more than enough.</p>

<p>To complete the quantifying pattern functions, there should be the <code>exact</code> and <code>atLeast</code> functions.</p>

<p>The <code>PositiveLookahead</code> method of the library can result in nested function calls as shown in <code>.PositiveLookahead(Pattern.With.Anything.Repeat.ZeroOrMore.Set(Pattern.With.Literal("!@#$%^&amp;*()_+-=")))</code>.<br>
Splitting the method in two functions allows the builder to remain flat.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nf">positiveLookaheadStart</span><span class="p">(</span><span class="n">string</span> <span class="n">pattern</span><span class="p">,</span> <span class="n">string</span> <span class="nv">$times</span> <span class="o">=</span> <span class="s1">''</span><span class="p">)</span>
<span class="p">{</span>
   <span class="k">return</span> <span class="s2">"</span><span class="nv">$pattern</span><span class="s2">(?=</span><span class="nv">$times</span><span class="s2">"</span><span class="p">;</span>
<span class="p">}</span>

<span class="nf">positiveLookaheadEnd</span><span class="p">(</span><span class="n">string</span> <span class="n">pattern</span><span class="p">)</span>
<span class="p">{</span>
  <span class="k">return</span> <span class="s2">"</span><span class="nv">$pattern</span><span class="s2">)"</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// example</span>

<span class="nv">$pattern</span> <span class="o">=</span> <span class="s1">''</span>
  <span class="o">|&gt;</span> <span class="p">(</span><span class="k">fn</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">positiveLookaheadStart</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">,</span> <span class="s1">'.*'</span><span class="p">))</span>
  <span class="o">|&gt;</span> <span class="p">(</span><span class="k">fn</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">any</span><span class="p">(</span><span class="nv">$pattern</span><span class="p">,</span> <span class="s1">'[sunday|monday]'</span><span class="p">))</span>
  <span class="o">|&gt;</span> <span class="nf">positiveLookaheadEnd</span><span class="p">(</span><span class="mf">...</span><span class="p">);</span>
</code></pre>

</div>



<p>Another example in the library of a method that should be split in a pipe operator library is the <code>NamedGroup</code> method. I would call the function <code>group</code> because beside naming the group it is also possible to create non-capturing groups.</p>

<p>The back referencing example might look great, but in regex a back reference is nothing more than <code>\1</code> or <code>\k&lt;name&gt;</code> depending on the use of unnamed or named groups. A function could be;<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">function</span> <span class="n">backReference</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$pattern</span><span class="p">,</span> <span class="kt">int</span><span class="o">|</span><span class="n">string</span> <span class="nv">$add</span><span class="p">)</span>
<span class="p">{</span>
   <span class="nv">$reference</span> <span class="o">=</span> <span class="nb">is_string</span><span class="p">(</span><span class="nv">$add</span><span class="p">)</span> <span class="o">?</span> <span class="s2">"k&lt;</span><span class="nv">$add</span><span class="s2">&gt;"</span> <span class="o">:</span> <span class="nv">$add</span><span class="p">;</span>

   <span class="k">return</span> <span class="s2">"</span><span class="nv">$pattern</span><span class="se">\\</span><span class="nv">$reference</span><span class="s2">"</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>The benefit of using the pipe operator is that there will be less code because the functions do one thing without side effects.</p>

<p>This also means less edge cases to test, because the regex building happens on the language level.</p>

<p>A minor negative point is that because the functions have more generic names it might not be as intuitive as a fluent API pattern. <br>
To alleviate that problem you could add your own functions, which is also another positive consequence of using the pipe operator. Extending a library becomes trivial. </p>

<p>Next time you think about the builder pattern, ask yourself if using the pipe operator might be a better fit.</p>

<p>PS: Don't use a regex builder in production, use the output for example from a pre-warm cache.</p>

