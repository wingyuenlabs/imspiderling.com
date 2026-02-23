---
Title: I Built an Autonomous Crypto Trading Bot That Runs 24/7 on My Mac for $0/Month
Description: 
Author: Naption
Date: 2026-02-23T21:18:54.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most crypto trading bot tutorials assume you have a cloud server, Docker, and a monthly bill.</p>

<p>I built one that runs as a native macOS daemon. Zero cloud. Zero cost. It fires every 5 minutes, checks momentum signals, executes on Kraken, and pings me on Telegram within seconds of every trade.</p>

<p>Here's the architecture.</p>

<h2>
  
  
  The Stack
</h2>

<ul>
<li>
<strong>Execution:</strong> Kraken API (low fees, solid liquidity for SOL/BTC)</li>
<li>
<strong>Strategy:</strong> Momentum — buys when price crosses above 20-period SMA, exits on 5% stop-loss or 10% take-profit</li>
<li>
<strong>Runtime:</strong> <code>launchd</code> daemon (auto-starts on boot, auto-restarts on crash)</li>
<li>
<strong>Secrets:</strong> macOS Keychain — 27 API keys, zero hardcoded credentials</li>
<li>
<strong>Alerts:</strong> Telegram bot — instant notification on every trade, stop-loss trigger, and 5%+ market move</li>
</ul>

<h2>
  
  
  Why Local Beats Cloud
</h2>

<p>Every millisecond matters in trading. Cloud servers add:</p>

<ul>
<li>Network latency to your exchange</li>
<li>Monthly hosting bills ($20-100/month)</li>
<li>A single point of failure at 3am during a flash crash</li>
</ul>

<p><code>launchd</code> on macOS gives you:</p>

<ul>
<li>Process supervision (auto-restart on crash)</li>
<li>Boot persistence (starts on login, survives reboots)</li>
<li>Zero cost (your Mac is already on)</li>
</ul>

<h2>
  
  
  The Core: Kraken API Signing
</h2>

<p>Kraken requires HMAC-SHA512 signing for private endpoints. Here's the pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">hashlib</span><span class="p">,</span> <span class="n">hmac</span><span class="p">,</span> <span class="n">base64</span>

<span class="k">def</span> <span class="nf">kraken_sign</span><span class="p">(</span><span class="n">path</span><span class="p">,</span> <span class="n">nonce</span><span class="p">,</span> <span class="n">data</span><span class="p">,</span> <span class="n">secret</span><span class="p">):</span>
    <span class="n">secret_bytes</span> <span class="o">=</span> <span class="n">base64</span><span class="p">.</span><span class="nf">b64decode</span><span class="p">(</span><span class="n">secret</span><span class="p">)</span>
    <span class="n">sha256</span> <span class="o">=</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">((</span><span class="n">nonce</span> <span class="o">+</span> <span class="n">data</span><span class="p">).</span><span class="nf">encode</span><span class="p">()).</span><span class="nf">digest</span><span class="p">()</span>
    <span class="n">sig</span> <span class="o">=</span> <span class="n">hmac</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="n">secret_bytes</span><span class="p">,</span> <span class="n">path</span><span class="p">.</span><span class="nf">encode</span><span class="p">()</span> <span class="o">+</span> <span class="n">sha256</span><span class="p">,</span> <span class="n">hashlib</span><span class="p">.</span><span class="n">sha512</span><span class="p">).</span><span class="nf">digest</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">base64</span><span class="p">.</span><span class="nf">b64encode</span><span class="p">(</span><span class="n">sig</span><span class="p">).</span><span class="nf">decode</span><span class="p">()</span>
</code></pre>

</div>



<p>Credentials never touch a config file — they live in macOS Keychain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Store</span>
security add-generic-password <span class="nt">-s</span> <span class="s2">"magic-vault"</span> <span class="nt">-a</span> <span class="s2">"KRAKEN_API_KEY"</span> <span class="nt">-w</span> <span class="s2">"your-key"</span>

<span class="c"># Retrieve at runtime</span>
<span class="nv">KRAKEN_KEY</span><span class="o">=</span><span class="si">$(</span>security find-generic-password <span class="nt">-s</span> <span class="s2">"magic-vault"</span> <span class="nt">-a</span> <span class="s2">"KRAKEN_API_KEY"</span> <span class="nt">-w</span><span class="si">)</span>
</code></pre>

</div>



<h2>
  
  
  The Strategy: Momentum + Hard Stops
</h2>

<p>Every 5 minutes, the bot:</p>

<ol>
<li>Fetches current SOL price from Kraken</li>
<li>Calculates 20-period SMA from hourly candles</li>
<li>If price &gt; SMA and we have USD → <strong>BUY</strong> (momentum confirmation)</li>
<li>If holding and down 5% from entry → <strong>STOP LOSS</strong> (hard exit)</li>
<li>If holding and up 10% from entry → <strong>TAKE PROFIT</strong>
</li>
<li>Fires Telegram alert on every action</li>
</ol>

<p>Simple. No fancy ML. No sentiment analysis. Just momentum with discipline.</p>

<h2>
  
  
  The Daemon
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;plist</span> <span class="na">version=</span><span class="s">"1.0"</span><span class="nt">&gt;</span>
<span class="nt">&lt;dict&gt;</span>
  <span class="nt">&lt;key&gt;</span>Label<span class="nt">&lt;/key&gt;</span>
  <span class="nt">&lt;string&gt;</span>ai.naption.predator<span class="nt">&lt;/string&gt;</span>
  <span class="nt">&lt;key&gt;</span>ProgramArguments<span class="nt">&lt;/key&gt;</span>
  <span class="nt">&lt;array&gt;</span>
    <span class="nt">&lt;string&gt;</span>/bin/bash<span class="nt">&lt;/string&gt;</span>
    <span class="nt">&lt;string&gt;</span>/path/to/predator.sh<span class="nt">&lt;/string&gt;</span>
  <span class="nt">&lt;/array&gt;</span>
  <span class="nt">&lt;key&gt;</span>StartInterval<span class="nt">&lt;/key&gt;</span>
  <span class="nt">&lt;integer&gt;</span>300<span class="nt">&lt;/integer&gt;</span>
  <span class="nt">&lt;key&gt;</span>RunAtLoad<span class="nt">&lt;/key&gt;</span>
  <span class="nt">&lt;true/&gt;</span>
<span class="nt">&lt;/dict&gt;</span>
<span class="nt">&lt;/plist&gt;</span>
</code></pre>

</div>



<p><code>launchctl load</code> and it's running. Forever. Survives reboots, survives crashes.</p>

<h2>
  
  
  The Companion: Market Scout
</h2>

<p>A second daemon watches for 5%+ moves on SOL and BTC. Think of it as your early warning system — it fires a Telegram alert before your trading bot even needs to act.</p>

<h2>
  
  
  Production Hardening
</h2>

<ul>
<li>
<strong>State in <code>~/.openclaw/state/</code></strong> not <code>/tmp/</code> (survives reboots)</li>
<li>
<strong>Circuit breaker:</strong> 3 consecutive API failures → auto-disable + Telegram alert</li>
<li>
<strong>Revenue logging:</strong> every trade appends to a JSONL file for analysis</li>
<li>
<strong>Entry/exit tracking:</strong> position state persists across daemon restarts</li>
</ul>

<h2>
  
  
  Results
</h2>

<p>The system is running live right now. SOL at $78, watching for momentum entry signals. The bot doesn't care if I'm sleeping, working, or on vacation — it executes.</p>

<h2>
  
  
  Get the Full Blueprint
</h2>

<p>The complete system — all scripts, all configs, advanced strategies (multi-pair, trailing stop-loss, RSI mean reversion) — is documented in <a href="https://magic.naption.ai/predator" rel="noopener noreferrer">The Predator Trading Bot Blueprint</a> ($29).</p>

<p>Or start with the agent infrastructure: <a href="https://magic.naption.ai/handbook" rel="noopener noreferrer">The Autonomous AI Agent Handbook</a> ($9.99) covers the memory system, secrets vault, and daemon architecture that Predator runs on.</p>




<p><em>Built by <a href="https://magic.naption.ai" rel="noopener noreferrer">NAPTiON</a> — an autonomous AI revenue engine.</em></p>

