---
Title: Contributing to streamplace
Description: 
Author: ElshadHu
Date: 2025-12-13T21:42:58.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  How I found the Project
</h3>

<p>Nowadays I've been reading and writing Go code regularly, and my Go journey started with <a href="https://go.dev/tour/welcome/1" rel="noopener noreferrer">A Tour of Go</a>. While building my <a href="https://github.com/ElshadHu/http-server-ts" rel="noopener noreferrer">http-server-ts</a>, I watched <a href="https://www.youtube.com/watch?v=FknTw9bJsXM&amp;t=248s" rel="noopener noreferrer">this video</a> where Primeagen builds an HTTP server in Go, which showed me how Go handles network programming. After getting confident with the language specifics, I found <a href="https://github.com/streamplace/streamplace" rel="noopener noreferrer">streamplace</a>. I tackled that <a href="https://github.com/streamplace/streamplace/issues/765" rel="noopener noreferrer">issue</a> and successfully made a <a href="https://github.com/streamplace/streamplace/pull/768" rel="noopener noreferrer">PR</a> that got merged. I'll give you the details below.</p>

<h3>
  
  
  The Issue Description
</h3>

<p>When the firehose is unavailable, the stream had error-prone code that would crash the loop. After getting familiar with the codebase (without even setting it up, thanks to my C++ background), I commented on the issue saying I'd like to work on it. I mentioned that the issue was in the <code>StartLabelerFirehose()</code> function in <code>labeler_firehose.go</code>, and I was planning to replace the crash with a backoff mechanism. I got nice feedback from the maintainer saying my approach sounded reasonable and that it would be better to add a metric to <code>spmetrics.go</code>. Basically, my conversation with the maintainer reduced the back and forth in my pull request, which I recommend to everybody that  communicate beforehand when tackling an issue.</p>

<h3>
  
  
  Set Up
</h3>

<p>Even though the code structure was pretty niche, setting up this project was time-consuming because of handling dependencies. I spent way too much time on this, which I should have done a long time ago if I wanted to contribute regularly. I dealt with <code>sqlite3</code> version issues when following the documentation.</p>

<h3>
  
  
  The Solution
</h3>

<p>I fixed the issue by adding a <code>streamplace_labeler_firehoses_connected</code> metric in <code>spmetrics.go</code> to track labeler connections:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">var</span> <span class="n">LabelerFirehosesConnected</span> <span class="o">=</span> <span class="n">promauto</span><span class="o">.</span><span class="n">NewGaugeVec</span><span class="p">(</span><span class="n">prometheus</span><span class="o">.</span><span class="n">GaugeOpts</span><span class="p">{</span>
    <span class="n">Name</span><span class="o">:</span> <span class="s">"streamplace_labeler_firehoses_connected"</span><span class="p">,</span>
    <span class="n">Help</span><span class="o">:</span> <span class="s">"number of currently connected labeler firehoses"</span><span class="p">,</span>
<span class="p">},</span> <span class="p">[]</span><span class="kt">string</span><span class="p">{</span><span class="s">"labeler"</span><span class="p">})</span>
</code></pre>

</div>



<p>Then I imported <code>spmetrics.go</code> and changed the log level at the retry threshold from <code>Error</code> to <code>Warn</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">log</span><span class="o">.</span><span class="n">Warn</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="s">"firehose failed 3 times within a minute, backing off"</span><span class="p">,</span> <span class="s">"err"</span><span class="p">,</span> <span class="n">err</span><span class="p">)</span>
<span class="n">time</span><span class="o">.</span><span class="n">Sleep</span><span class="p">(</span><span class="m">5</span> <span class="o">*</span> <span class="n">time</span><span class="o">.</span><span class="n">Second</span><span class="p">)</span>
<span class="n">retryCount</span> <span class="o">=</span> <span class="m">0</span>
<span class="n">retryWindow</span> <span class="o">=</span> <span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span>
</code></pre>

</div>



<p>When fixing this bug, I made a mistake that I corrected before even committing. I was unintentionally creating a race condition by changing <code>cancel()</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">defer</span> <span class="k">func</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">spmetrics</span><span class="o">.</span><span class="n">LabelerFirehosesConnected</span><span class="o">.</span><span class="n">WithLabelValues</span><span class="p">(</span><span class="n">did</span><span class="p">)</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="m">0</span><span class="p">)</span>
    <span class="n">cancel</span><span class="p">()</span>
<span class="p">}()</span>
</code></pre>

</div>



<p>This was problematic because the <code>defer</code> won't execute until the function returns, but I also had <code>defer cancel()</code> which would be called in the reverse order of declaration. This was creating confusion. When it comes to critical-performance places in the code, it's always better to be skeptical. For that reason, I didn't touch <code>cancel()</code> and it stayed as <code>defer cancel()</code>.</p>

<h3>
  
  
  Testing and Suggestion from Maintainer
</h3>

<p>I tested the change locally using Bluesky's official moderation labeler. I made sure that the metric reports 1 while the labeler firehose is connected. I also tested the retry path and saw that the code backs off for 5 seconds after repeated failures instead of crashing.</p>

<p>But I didn't consider more than one labeler firehose by writing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">spmetrics</span><span class="o">.</span><span class="n">LabelerFirehosesConnected</span><span class="o">.</span><span class="n">WithLabelValues</span><span class="p">(</span><span class="n">did</span><span class="p">)</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="m">1</span><span class="p">)</span>
<span class="k">defer</span> <span class="n">spmetrics</span><span class="o">.</span><span class="n">LabelerFirehosesConnected</span><span class="o">.</span><span class="n">WithLabelValues</span><span class="p">(</span><span class="n">did</span><span class="p">)</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="m">0</span><span class="p">)</span>
</code></pre>

</div>



<p>Via the suggestion from the maintainer, I changed to <code>Inc</code> and <code>Dec</code> instead of <code>Set</code> for connecting more than one labeler firehose:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code>    <span class="n">spmetrics</span><span class="o">.</span><span class="n">LabelerFirehosesConnected</span><span class="o">.</span><span class="n">WithLabelValues</span><span class="p">(</span><span class="n">did</span><span class="p">)</span><span class="o">.</span><span class="n">Inc</span><span class="p">()</span>
    <span class="k">defer</span> <span class="n">spmetrics</span><span class="o">.</span><span class="n">LabelerFirehosesConnected</span><span class="o">.</span><span class="n">WithLabelValues</span><span class="p">(</span><span class="n">did</span><span class="p">)</span><span class="o">.</span><span class="n">Dec</span><span class="p">()</span>
</code></pre>

</div>



<p>Eventually my pull request got merged.</p>

<h3>
  
  
  Gained Knowledge
</h3>

<p>For the first time, I dealt with a well-structured Go codebase. I learned about race conditions and reminded myself how important small details are. Also, as I mentioned before, it's better to keep code changes precise and stay in touch with maintainers. I'll keep going and continue contributing to this codebase.</p>

