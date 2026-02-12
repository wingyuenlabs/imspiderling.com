---
Title: Faster Server Startup in Meteor 3.4 with Deferrables
Description: 
Author: Gabriel Grubba
Date: 2026-02-12T22:01:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>As you might have seen in the <a href="https://dev.to/meteor/meteor-34-is-out-rspack-integration-4x-faster-builds-8x-smaller-bundles-and-extended-bundler-51ia">3.4 release blog post</a>, Meteor has received a <strong>HUGE</strong> update with a lot of focus on the developer experience. One of the new additions was the new <code>deferrable</code> functions, which we will dive into today in this blog post.</p>

<h2>
  
  
  Why were those functions added?
</h2>

<p>While seeing teams update to 3.4, we saw a recurring problem: build times have gotten faster with rspack, but the developers were not feeling it because they had a bottleneck at the <strong>startup</strong></p>

<p>With that in mind, we did our investigation, and we saw that those apps that were not feeling this increase in productivity were the ones that had async or costly operations being done at the startup. You can think of requesting data from external APIs, waiting for sidekick services to get started, etc.</p>

<p>For production, this makes sense — you don’t want your app to start without those dependencies ready. You might not need all that. With that in mind, we made this addition to our api.</p>

<h2>
  
  
  New API
</h2>

<p>The added functions were: <a href="https://docs.meteor.com/api/meteor.html#Meteor-deferrable" rel="noopener noreferrer"><code>deferrable</code></a>, <a href="https://docs.meteor.com/api/meteor.html#Meteor-deferDev" rel="noopener noreferrer"><code>deferDev</code></a>, <a href="https://docs.meteor.com/api/meteor.html#Meteor-deferProd" rel="noopener noreferrer"><code>deferProd</code></a>. I'll start with the first.</p>

<blockquote>
<p>⚠️<br>
If a function is not deferred for the current environment, it runs immediately and behaves exactly like a normal function call (including return values).</p>
</blockquote>

<h3>
  
  
  Meteor.deferrable
</h3>

<p>In the docs, we can see it in action:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Meteor</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">meteor/meteor</span><span class="dl">"</span><span class="p">;</span>

<span class="nx">Meteor</span><span class="p">.</span><span class="nf">startup</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">await</span> <span class="nx">Meteor</span><span class="p">.</span><span class="nf">deferrable</span><span class="p">(</span><span class="nx">connectToExternalDB</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">on</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">development</span><span class="dl">"</span><span class="p">],</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p>This means that, <code>on</code> development, this connectToExternalDB function will run deferred (scheduled to run after the initial startup completes, instead of blocking it).</p>

<p>You can define <code>test</code>, <code>development</code>, and <code>production</code> – for testing only environments, this can be very nice! </p>

<p>Conceptually, deferrable functions let Meteor finish booting the app first, and then run non-critical async work afterward. The app becomes usable sooner, even if some background initialization is still happening.</p>

<h4>
  
  
  Meteor.deferDev
</h4>

<p>deferDev is an abstraction on top of <code>deferrable</code> with the options for development and testing environments.</p>

<p>This is probably the one that will be used the most. You should use this one for flagging those functions that <em>can</em> run later in development mode; Consistently using it will make your overall startup and rebuild faster. </p>

<p>In our hypothetical <a href="https://github.com/Grubba27/modern-meteor-example/pull/1" rel="noopener noreferrer">PR</a> example, using <code>deferDev</code> was saving our startup time in around two seconds!</p>

<p>And to use it is <em>very</em> simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  const connectToExternalExpensiveService = async () =&gt; {
    const sleep = (ms) =&gt; new Promise(resolve =&gt; setTimeout(resolve, ms));
    console.log("Connecting to external expensive service...");
    await sleep(Math.random() * 2000 + 1000); // simulate expensive connection
    console.log("Connected to external expensive service!");
  }

  await Meteor.deferDev(connectToExternalExpensiveService);
</code></pre>

</div>



<h4>
  
  
  Meteor.deferProd
</h4>

<p>deferProd is an abstraction on top of <code>deferrable</code> with the options for production.</p>

<p>This is used for more one-off cases. We saw usage in this one for connecting to dev-only services and to overall services that will be used later in production, but were necessary to be there at startup when working locally.</p>

<p>A good example is warming a cache or connecting to an analytics pipeline that won’t be used during the first requests, but must still exist in production.</p>

<h4>
  
  
  Using deferrable with values
</h4>

<p>The last notable thing about this new API is that deferred functions, if they are not deferred they, work <em>exactly</em> as if they were not marked as deferred.</p>

<p>You can see this logging example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Before defer Dev</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">Meteor</span><span class="p">.</span><span class="nf">deferDev</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Connecting to some external service...</span><span class="dl">"</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">This will run later on local mode!</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">})</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">After defer Dev</span><span class="dl">"</span><span class="p">);</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Before defer Prod</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">Meteor</span><span class="p">.</span><span class="nf">deferProd</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Connecting to some service...</span><span class="dl">"</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">This will run later on prod mode!</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">})</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">After defer Prod</span><span class="dl">"</span><span class="p">);</span>

  <span class="c1">// in a regular meteor run</span>
  <span class="c1">// Before defer Dev</span>
  <span class="c1">// After defer Dev</span>
  <span class="c1">// Before defer Prod</span>
  <span class="c1">// Connecting to some service...</span>
  <span class="c1">// This will run later on prod mode!</span>
  <span class="c1">// After defer Prod</span>
  <span class="c1">// Connecting to some external service...</span>
  <span class="c1">// This will run later on local mode!</span>
</code></pre>

</div>



<p>Very cool, eh?</p>

<p>But <em>also</em> this API can return values so for example you can have this example in your code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>  <span class="kd">const</span> <span class="nx">value</span> <span class="o">=</span> <span class="nx">Meteor</span><span class="p">.</span><span class="nf">deferDev</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="dl">"</span><span class="s2">later_in_dev</span><span class="dl">"</span><span class="p">)</span> <span class="o">||</span> <span class="nx">Meteor</span><span class="p">.</span><span class="nf">deferProd</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="dl">"</span><span class="s2">later_in_prod</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">value is set to:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">value</span><span class="p">);</span>
</code></pre>

</div>



<p>If a function is not deferred for the current environment, it runs immediately and behaves exactly like a normal function call (including return values).</p>

<blockquote>
<p>⚠️<br>
You can see all the examples that I shared today in this <a href="https://github.com/Grubba27/modern-meteor-example/pull/1" rel="noopener noreferrer">PR</a></p>
</blockquote>

<h3>
  
  
  Migration &amp; Conclusion
</h3>

<p>To start getting gains with this new API, begin by wrapping your slowest startup work — external connections, async setup, or non-critical initialization — with <code>deferDev</code> and measure the impact. Most teams will see improvements immediately.</p>

<blockquote>
<p><a href="https://galaxycloud.app/" rel="noopener noreferrer">Galaxy team</a> saw these improvements first hand – their local environment, with these optimizations, got ~3x faster!</p>
</blockquote>

<p>You can think of these functions as saying: <code>this work is important — just not right now.</code></p>

<p>Treat <code>deferDev</code> as part of your startup hygiene: anything that doesn’t block correctness should be a candidate.</p>

<p>If you haven’t revisited your startup code since upgrading to 3.4, now is the perfect time — you may be sitting on seconds of free productivity without realizing it.</p>

<p>If you’re migrating to 3.4 and have questions, run into edge cases, or want to share how much startup time you shaved off using these APIs, join the conversation on the Meteor forums. We’d love to hear what worked for your app — and what didn’t.</p>




<p>We’re excited about what’s to come and can’t wait for you to join the Meteor renaissance!</p>

<p>For feedback, questions, or support, <a href="https://forums.meteor.com/" rel="noopener noreferrer">visit our forums</a> or join our <a href="https://discord.com/invite/3w7EKdpghq" rel="noopener noreferrer">Discord channel</a>.</p>

<p>Follow us on <a href="https://x.com/meteorjs" rel="noopener noreferrer">Twitter</a> and <a href="https://github.com/meteor" rel="noopener noreferrer">GitHub</a>.</p>

