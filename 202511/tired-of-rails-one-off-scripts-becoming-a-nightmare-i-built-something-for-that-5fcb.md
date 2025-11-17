---
Title: Tired of Rails one-off scripts becoming a nightmare? I built something for that.
Description: 
Author: Abd El-latif
Date: 2025-11-17T21:51:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>You know the drill. You need to run a script once to fix some data, update user preferences, or migrate something. You write a rake task, run it, and then... did it actually run? Did it work? If something breaks halfway through, how do you know where to restart?</p>

<p>I got tired of this cycle, so I built <strong>script_tracker</strong> - a Ruby gem that treats your one-off scripts like migrations (but better).</p>

<p><strong>What it does:</strong></p>

<ul>
<li>Tracks which scripts have run (no more "did I run this already?")</li>
<li>Wraps everything in transactions (rollback on failure)
</li>
<li>Built-in progress logging ("Processing 1,247 of 10,000 users...")</li>
<li>Batch processing helpers (because memory matters)</li>
<li>Timeout support (no more runaway scripts)</li>
<li>Simple rake commands to manage everything</li>
</ul>

<p><strong>Before:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Some random rake task</span>
<span class="c1"># Did this run? Who knows!</span>
<span class="n">rake</span> <span class="n">data</span><span class="ss">:fix_user_preferences</span>
</code></pre>

</div>



<p><strong>After:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Clean, tracked, logged</span>
<span class="n">rake</span> <span class="n">scripts</span><span class="ss">:create</span><span class="p">[</span><span class="s2">"fix user preferences"</span><span class="p">]</span> 
<span class="n">rake</span> <span class="n">scripts</span><span class="ss">:run</span>
<span class="n">rake</span> <span class="n">scripts</span><span class="ss">:status</span>  <span class="c1"># See what ran and when</span>
</code></pre>

</div>



<p>The best part? If your script fails halfway through, you know exactly where, and you can handle retries properly.</p>

<p><strong>Why I built this:</strong><br>
Data scripts have burned every Rails dev. When you run something in production, it fails silently, or worse, it runs twice and corrupts the data. I wanted the same confidence we have with migrations, but for one-off scripts.</p>

<p><strong>Real talk:</strong> This started as internal tooling at my company. We had too many “wait, did that script run?” conversations. Now our data migrations are as reliable as our schema migrations.</p>

<p>The gem is open source and ready to use. Would love feedback from fellow Rails developers who’ve felt this pain.</p>

<p>Check it out: <a href="https://github.com/a-abdellatif98/script_tracker" rel="noopener noreferrer">https://github.com/a-abdellatif98/script_tracker</a></p>

<p>What’s your biggest one-off script horror story? I bet this would have prevented it.</p>

