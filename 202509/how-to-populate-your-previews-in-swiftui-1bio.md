---
Title: How to populate your #Previews in SwiftUI
Description: 
Author: Stoian Dan
Date: 2025-09-05T21:23:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Apple created the <code>#Preview</code> macro to more easily preview your SwiftUI views in Xcode:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="cp">#Preview {</span>
    <span class="kt">BookView</span><span class="p">()</span>
<span class="p">}</span>
</code></pre>

</div>



<p>You can even write some handy code in a <code>#Preview</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="cp">#Preview {</span>
    <span class="k">let</span> <span class="nv">books</span> <span class="o">=</span> <span class="p">[</span><span class="s">"Eros&amp;Agape, by Anders Nygren"</span><span class="p">,</span> <span class="s">"Commentary to Galatians, by Martin Luther"</span><span class="p">]</span>

    <span class="kt">BookView</span><span class="p">(</span><span class="nv">books</span><span class="p">:</span> <span class="n">books</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>However, what do you do when you want to <em>cache</em> the data needed for preview initialization or, another use case, if you need to use <strong>async</strong> code to get that data in the first place?</p>

<p>While initially I saw approaches like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="cp">#Preview {</span>
    <span class="kd">@Previewable</span> <span class="kd">@State</span> <span class="k">var</span> <span class="nv">books</span><span class="p">:</span> <span class="p">[</span><span class="kt">Book</span><span class="p">]</span>  <span class="o">=</span> <span class="p">[]</span>

    <span class="k">if</span> <span class="n">books</span><span class="o">.</span><span class="n">isEmpty</span> <span class="p">{</span>
        <span class="kt">ProgressView</span><span class="p">()</span>
          <span class="o">.</span><span class="n">task</span> <span class="p">{</span>
             <span class="n">books</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">myAsyncFunc</span><span class="p">()</span>
          <span class="p">}</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="kt">BookView</span><span class="p">(</span><span class="nv">books</span><span class="p">:</span> <span class="n">books</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This was a <em>hacky</em> way to do it and it also didn't provide caching. So here's the more Apple way — that I wish Apple would advertise more , as they're documentation doesn't really provide examples in general 😩 — to do it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">struct</span> <span class="kt">BookProvider</span><span class="p">:</span> <span class="kt">PreviewModifier</span> <span class="p">{</span>
    <span class="kd">static</span> <span class="kd">func</span> <span class="nf">makeSharedContext</span><span class="p">()</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="p">[</span><span class="kt">Book</span><span class="p">]</span> <span class="p">{</span>
             <span class="k">return</span> <span class="k">await</span> <span class="nf">myAsyncBookGenerator</span><span class="p">()</span>
    <span class="p">}</span>


    <span class="kd">func</span> <span class="nf">body</span><span class="p">(</span><span class="nv">content</span><span class="p">:</span> <span class="kt">Content</span><span class="p">,</span> <span class="nv">context</span><span class="p">:</span> <span class="p">[</span><span class="kt">Book</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="kd">some</span> <span class="kt">View</span> <span class="p">{</span>
        <span class="kt">BookView</span><span class="p">(</span><span class="nv">books</span><span class="p">:</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In essence, you can use <code>makeSharedContext</code> to generate <em>whatever</em> type you want – by default it's <code>Void</code>, so you can also entirely omit  this function – then, via the <code>body</code> method, you get your content (whatever the #Preview provides), your <code>context</code> (whatever <code>makeSharedContext</code> provides); finally you have your chance to return whatever the newly modified view! That's it!</p>

<p>To apply the modifier, just:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="cp">#Preview(traits: .modifier(BookProvider())) {</span>
    <span class="kt">BookView</span><span class="p">()</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I'm just wondering whether or not Apple could have used generics (probably not? because you can't really do that in protocols…) to not have that <code>Content</code> type be a <em>type erased</em> view.</p>

<p>P.S.<br>
I'll admit my use-case is a bit strange because I don't really make use of the <code>content</code> variable – I can't, because it's typed erased – I just directly return a view no matter what it's being thrown by the client #Preview.</p>

