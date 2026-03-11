---
Title: I built a tool that scans your Gmail and finds every subscription you forgot about
Description: 
Author: Lakshmi Sravya Vedantham
Date: 2026-03-11T21:57:09.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Your subscription data stays on your machine. Nothing leaves. No accounts, no cloud, no tracking.</strong></p>

<p>I was paying for things I completely forgot about.</p>

<p>Suno. Replicate. A Google One plan I stopped using months ago. Scattered across two Gmail accounts, quietly charging me every month.</p>

<p>So I built <strong>inboxscan</strong> — a CLI tool that connects to your Gmail via IMAP, scans for subscription-related emails, and tells you exactly what you are paying for. Everything runs locally. Your emails never leave your computer.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>inboxscan
inboxscan run <span class="nt">--email</span> you@gmail.com
</code></pre>

</div>



<p>Output looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>INBOXSCAN REPORT
════════════════════════════════════════════════════════════
Scanned: you@gmail.com
Found: 8 subscriptions  |  Total burn: $142.00/mo

  [ACTIVE]     Claude Pro             $20.00/mo      Mar 01
  [ACTIVE]     Spotify                $10.99/mo      Mar 02
  [ACTIVE]     Notion                 $16.00/mo      Feb 28
  [ACTIVE]     GitHub Pro              $4.00/mo      Mar 05
  [ACTIVE]     Figma                  $15.00/mo      Mar 04
  [DORMANT]    Audible                $14.95/mo      Nov 02
  [DORMANT]    Skillshare              $9.99/mo      Oct 15
  [DORMANT]    SomeService            $25.00/mo      Sep 27

════════════════════════════════════════════════════════════
Potential savings: $49.94/mo (cancel DORMANT subscriptions)
</code></pre>

</div>



<h2>
  
  
  Privacy first — always
</h2>

<ul>
<li>
<strong>Zero network calls</strong> beyond the Gmail IMAP fetch itself</li>
<li>
<strong>No credentials stored</strong> — app password used only during the scan, never written to disk</li>
<li>
<strong>Everything runs locally</strong> — parsing, detection, analysis, all on your machine</li>
<li>
<strong>No telemetry, no analytics, no phone-home</strong> — the tool has no idea who you are</li>
<li>Results cached in <code>~/.inboxscan/cache.db</code> — a local SQLite file, stays on your machine</li>
<li>Open source and MIT licensed — read every line at <a href="https://github.com/LakshmiSravyaVedantham/inboxscan" rel="noopener noreferrer">https://github.com/LakshmiSravyaVedantham/inboxscan</a>
</li>
</ul>

<p>This is the opposite of most subscription tracking apps that ask you to hand over your bank credentials or email login. inboxscan never stores your data or sends it anywhere.</p>

<h2>
  
  
  How it works
</h2>

<p>It connects to Gmail via IMAP, searches for subscription-related subjects (receipt, invoice, billing, renewal, etc.), parses each email for amounts and dates, and groups by sender domain.</p>

<p><strong>The clever part:</strong> it does not just check a hardcoded list of services. It detects any sender that has charged you 2+ times — so it catches services you have never heard of, not just Netflix and Spotify.</p>

<h2>
  
  
  Cancellation built in
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>inboxscan cancel audible
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Cancel Audible
────────────────────────────────────────
Cancellation page: https://www.audible.com/account/optout

Email template:
Subject: Cancel Audible Membership

Please cancel my Audible membership immediately and confirm cancellation.
</code></pre>

</div>



<h2>
  
  
  Multiple accounts
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>inboxscan run <span class="nt">--email</span> work@gmail.com <span class="nt">--email</span> personal@gmail.com
</code></pre>

</div>



<p>Scans both, shows which subscription came from which account, combined report.</p>

<h2>
  
  
  Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>inboxscan
</code></pre>

</div>



<p>You will need a Gmail app password (not your real password — a separate one you can revoke anytime):</p>

<ol>
<li>Go to myaccount.google.com/apppasswords</li>
<li>Create one named "inboxscan"</li>
<li>Use that 16-char password when prompted</li>
<li>Revoke it anytime from the same page</li>
</ol>

<p>The app password only grants IMAP read access. inboxscan uses it to fetch email subjects and bodies — nothing else.</p>

<h2>
  
  
  What I found on my own machine
</h2>

<p>Running it on my own accounts turned up services I had completely forgotten about — dormant charges adding up to real money every month. The tool paid for itself in the first scan.</p>




<p>Repo: <a href="https://github.com/LakshmiSravyaVedantham/inboxscan" rel="noopener noreferrer">https://github.com/LakshmiSravyaVedantham/inboxscan</a></p>

<p>PyPI: <a href="https://pypi.org/project/inboxscan/" rel="noopener noreferrer">https://pypi.org/project/inboxscan/</a></p>

<p>It is MIT licensed and open source. PRs welcome — especially for expanding the known services list and adding support for other email providers.</p>

