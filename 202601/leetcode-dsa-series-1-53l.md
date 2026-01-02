---
Title: LeetCode DSA Series #1
Description: 
Author: David Babalola
Date: 2026-01-01T21:18:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>It is a new year. Happy New Year!</p>

<p>On December 31, 2025, I saw a post on X (formerly Twitter) from someone who had solved LeetCode problems every day in 2025. </p>

<p><iframe class="tweet-embed" id="tweet-2006322352368279585-377" src="https://platform.twitter.com/embed/Tweet.html?id=2006322352368279585">
</iframe>

  // Detect dark theme
  var iframe = document.getElementById('tweet-2006322352368279585-377');
  if (document.body.className.includes('dark-theme')) {
    iframe.src = "https://platform.twitter.com/embed/Tweet.html?id=2006322352368279585&amp;theme=dark"
  }



</p>

<p>This has inspired me to begin my own challenge in 2026. I honestly do not know if I have the drive to see it through. I will see how it goes.</p>

<p>My submission for Day 1 is <strong><a href="https://leetcode.com/problems/number-of-1-bits/" rel="noopener noreferrer">191. Number of 1 Bits</a></strong></p>

<p>This is my solution<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">Solution</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">hammingWeight</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">n</span><span class="p">:</span> <span class="nb">int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">int</span><span class="p">:</span>
        <span class="n">n_binary</span>  <span class="o">=</span> <span class="sh">""</span>
        <span class="k">while</span> <span class="n">n</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
            <span class="n">remainder</span> <span class="o">=</span> <span class="n">n</span> <span class="o">%</span> <span class="mi">2</span>
            <span class="n">n</span> <span class="o">=</span> <span class="n">n</span> <span class="o">//</span> <span class="mi">2</span>
            <span class="n">n_binary</span> <span class="o">+=</span> <span class="nf">str</span><span class="p">(</span><span class="n">remainder</span><span class="p">)</span>
        <span class="n">n_binary</span> <span class="o">=</span> <span class="n">n_binary</span><span class="p">[::</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span>

        <span class="n">count</span> <span class="o">=</span> <span class="mi">0</span>
        <span class="k">for</span> <span class="n">char</span> <span class="ow">in</span> <span class="n">n_binary</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">char</span> <span class="o">==</span> <span class="sh">'</span><span class="s">1</span><span class="sh">'</span><span class="p">:</span>
                <span class="n">count</span> <span class="o">+=</span> <span class="mi">1</span>

        <span class="k">return</span> <span class="n">count</span>        
</code></pre>

</div>



<p>Both time and space complexity are O(Log N).</p>

<p>Is there a better way to do this? Please let me know. </p>

