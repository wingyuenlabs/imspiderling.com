---
Title: 🛠 Fixing `Undefined property: TwilioSmsMessage::$contentVariables` in Laravel (Twilio Notifications)
Description: 
Author: Ayowande Oluwatosin
Date: 2026-01-06T22:14:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>While integrating <strong>Twilio SMS notifications</strong> in a Laravel app using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="nl">"laravel-notification-channels/twilio"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^4.1"</span><span class="w">
</span></code></pre>

</div>



<p>I ran into this error:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Undefined property: NotificationChannels\Twilio\TwilioSmsMessage::$contentVariables
</code></pre>

</div>



<p>Even though my notification code was perfectly valid and only used <code>-&gt;content()</code>, Laravel kept throwing this exception.</p>

<p>This post explains <strong>why this happens</strong>, <strong>how I fixed it</strong>, and <strong>what to be careful about</strong>.</p>




<h2>
  
  
  🔍 The Problem
</h2>

<p>The error occurs when <strong>some internal logic (or another package)</strong> tries to access:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$contentVariables</span>
</code></pre>

</div>



<p>on <code>TwilioSmsMessage</code>.</p>

<p>However, in <strong>Twilio Notification Channel v4.1</strong>, the base class <code>TwilioMessage</code> <strong>does not define this property</strong>, causing PHP to throw an undefined property exception.</p>

<p>This is especially common if:</p>

<ul>
<li>You upgraded/downgraded packages</li>
<li>You previously experimented with Twilio Content Templates</li>
<li>A dependency expects <code>contentVariables</code> to exist</li>
</ul>




<h2>
  
  
  ✅ The Workaround That Fixed It
</h2>

<p>I fixed the issue by <strong>extending the base <code>TwilioMessage</code> class</strong> inside the package to explicitly define <code>contentVariables</code>.</p>

<p>📄 <strong>File edited:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>vendor/laravel-notification-channels/twilio/src/TwilioMessage.php
</code></pre>

</div>



<h3>
  
  
  ✏️ Updated <code>TwilioMessage</code> class
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>

<span class="kn">namespace</span> <span class="nn">NotificationChannels\Twilio</span><span class="p">;</span>

<span class="k">abstract</span> <span class="kd">class</span> <span class="nc">TwilioMessage</span>
<span class="p">{</span>
    <span class="cd">/**
     * The phone number the message should be sent from.
     */</span>
    <span class="k">public</span> <span class="kt">?string</span> <span class="nv">$from</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="k">public</span> <span class="kt">?string</span> <span class="nv">$statusCallback</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="k">public</span> <span class="kt">?string</span> <span class="nv">$statusCallbackMethod</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="cd">/**
     * Create a new message instance.
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">(</span>
        <span class="k">public</span> <span class="kt">string</span> <span class="nv">$content</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span>
        <span class="k">public</span> <span class="kt">string</span> <span class="nv">$contentVariables</span> <span class="o">=</span> <span class="s1">''</span>
    <span class="p">)</span> <span class="p">{</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Create a message object.
     */</span>
    <span class="k">public</span> <span class="k">static</span> <span class="k">function</span> <span class="n">create</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$content</span> <span class="o">=</span> <span class="s1">''</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nc">static</span><span class="p">(</span><span class="nv">$content</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Set the message content.
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">content</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$content</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">content</span> <span class="o">=</span> <span class="nv">$content</span><span class="p">;</span>

        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Set content variables (prevents undefined property errors).
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">contentVariables</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$content</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">content</span> <span class="o">=</span> <span class="nv">$content</span><span class="p">;</span>

        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Set the phone number the message should be sent from.
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">from</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$from</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">from</span> <span class="o">=</span> <span class="nv">$from</span><span class="p">;</span>

        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">getFrom</span><span class="p">():</span> <span class="kt">?string</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">from</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">statusCallback</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$statusCallback</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">statusCallback</span> <span class="o">=</span> <span class="nv">$statusCallback</span><span class="p">;</span>

        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">statusCallbackMethod</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$statusCallbackMethod</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">statusCallbackMethod</span> <span class="o">=</span> <span class="nv">$statusCallbackMethod</span><span class="p">;</span>

        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>After this change, the error disappeared completely.</p>




<h2>
  
  
  ⚠️ Important Warning
</h2>

<p><strong>Editing vendor files is NOT recommended for production.</strong></p>

<p>Why?</p>

<ul>
<li>Composer updates will overwrite the change</li>
<li>Other environments won’t have the fix</li>
<li>It hides the real compatibility issue</li>
</ul>




<h2>
  
  
  ✅ Better Long-Term Alternatives
</h2>

<p>If you hit this issue, consider:</p>

<ol>
<li><strong>Ensuring all Twilio-related packages are on compatible versions</strong></li>
<li><strong>Avoiding <code>contentVariables</code> when sending SMS</strong></li>
<li>
<strong>Using Twilio REST API directly</strong> if you need Content Templates (WhatsApp)</li>
<li>
<strong>Forking the package</strong> and maintaining your fix properly</li>
<li>
<strong>Opening a GitHub issue / PR</strong> on the package repo</li>
</ol>




<h2>
  
  
  🏁 Final Thoughts
</h2>

<p>If you’re working with:</p>

<ul>
<li>Laravel Notifications</li>
<li>Twilio SMS</li>
<li><code>laravel-notification-channels/twilio</code></li>
</ul>

<p>…and hit unexplained property errors, <strong>check the base message classes first</strong>.</p>

<p>Hope this saves someone hours of debugging 👋<br>
Happy coding 🚀</p>




