---
Title: How to Convert Telegram Chat Export JSON into CSV
Description: 
Author: Ignas
Date: 2026-02-10T21:27:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>Telegram Desktop allows you to export chats, but the export format is JSON.<br>
If you’ve ever tried to open that JSON in Excel or Google Sheets, you know it’s basically unusable.</p>

<p>I ran into this problem when I needed to work with a Telegram chat outside of Telegram<br>
(for analysis, archiving, and basic filtering), and the raw export just didn’t work.</p>


<h2>
  
  
  Why Telegram exports are a problem
</h2>

<p>Telegram JSON exports have a few issues:</p>

<ul>
<li>Messages can be nested and formatted inconsistently</li>
<li>Emojis and multiline messages often break spreadsheets</li>
<li>System messages are mixed with real messages</li>
<li>Large exports can crash Excel entirely</li>
</ul>

<p>In practice, this makes Telegram exports hard to use for anything other than storage.</p>


<h2>
  
  
  The basic idea of a solution
</h2>

<p>At a high level, the solution is simple:</p>

<ol>
<li>Load the exported <code>result.json</code>
</li>
<li>Iterate through the <code>messages</code> array</li>
<li>Normalize each message into a flat structure</li>
<li>Write everything to a properly quoted CSV</li>
</ol>

<p>The tricky part is handling edge cases like:</p>

<ul>
<li>multiline text</li>
<li>emojis</li>
<li>media messages</li>
<li>system/service messages</li>
</ul>


<h2>
  
  
  Minimal example
</h2>

<p>Here is a minimal example showing the general approach:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">csv</span>

<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">result.json</span><span class="sh">"</span><span class="p">,</span> <span class="n">encoding</span><span class="o">=</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
    <span class="n">data</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>

<span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">chat.csv</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">w</span><span class="sh">"</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">""</span><span class="p">,</span> <span class="n">encoding</span><span class="o">=</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">)</span> <span class="k">as</span> <span class="n">out</span><span class="p">:</span>
    <span class="n">writer</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nf">writer</span><span class="p">(</span><span class="n">out</span><span class="p">)</span>
    <span class="n">writer</span><span class="p">.</span><span class="nf">writerow</span><span class="p">([</span><span class="sh">"</span><span class="s">date</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">sender</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">])</span>

    <span class="k">for</span> <span class="n">msg</span> <span class="ow">in</span> <span class="n">data</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">messages</span><span class="sh">"</span><span class="p">,</span> <span class="p">[]):</span>
        <span class="n">date</span> <span class="o">=</span> <span class="n">msg</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">date</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>
        <span class="n">sender</span> <span class="o">=</span> <span class="n">msg</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">from</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Unknown</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">text</span> <span class="o">=</span> <span class="n">msg</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>

        <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">text</span><span class="p">,</span> <span class="nb">list</span><span class="p">):</span>
            <span class="n">text</span> <span class="o">=</span> <span class="sh">""</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span>
                <span class="n">t</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span> <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="nb">dict</span><span class="p">)</span> <span class="k">else</span> <span class="n">t</span>
                <span class="k">for</span> <span class="n">t</span> <span class="ow">in</span> <span class="n">text</span>
            <span class="p">)</span>

        <span class="n">writer</span><span class="p">.</span><span class="nf">writerow</span><span class="p">([</span><span class="n">date</span><span class="p">,</span> <span class="n">sender</span><span class="p">,</span> <span class="n">text</span><span class="p">])</span>
<span class="c1">##
</span></code></pre>

</div>



<p>This demonstrates the idea, but a production-ready solution needs to handle many more edge cases.<br>
A practical solution<br>
To avoid rewriting this logic every time, I packaged the full solution into a small offline Python CLI that converts Telegram Desktop exports into clean CSV files.<br>
It handles emojis, multiline messages, system messages, and large exports, and produces a CSV that opens correctly in Excel and Google Sheets.<br>
If you want the ready-to-use version, it’s available here: <a href="https://momentumguard.gumroad.com/l/ivyxn%EF%BF%BD" rel="noopener noreferrer">https://momentumguard.gumroad.com/l/ivyxn�</a></p>

