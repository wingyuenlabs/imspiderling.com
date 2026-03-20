---
Title: Fuck Around and Find Out: How a $6.99 Phishing Scam Got 1.2 Million Fake Victims
Description: 
Author: Mr. 0x1
Date: 2026-03-20T22:09:31.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>A forensic teardown and hostile response to the Smishing Triad's Lighthouse phishing kit.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkk9kk1t9da9tsz77zipz.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkk9kk1t9da9tsz77zipz.gif" alt="fuck-around-and-find-out" width="480" height="322"></a></p>




<h2>
  
  
  The Hustle
</h2>

<p>It started, as these things always do, with a text message.</p>

<blockquote>
<p><strong>California DMV Final Notice:</strong> Enforcement will begin on March 21.<br>
Our system indicates you have an unresolved traffic violation.<br>
Per relevant regulations, if payment is not completed by March 20, 2026, the following will apply:</p>

<ol>
<li>Update your record in the DMV system</li>
<li>Suspend your vehicle registration starting March 21</li>
<li>Temporarily restrict driving privileges for 30 days</li>
<li>Refer your account for further processing with an additional administrative fee</li>
<li>Possible legal follow-up and impact to your credit record</li>
</ol>

<p>Please settle this promptly to avoid service restrictions and additional procedures.<br>
Link: <code>https://ca.gov-axb.cfd/dmv</code></p>
</blockquote>

<p>Classic. Urgency. Authority. Escalating consequences. A $6.99 "traffic violation" — just low enough that a panicked person might pay it without thinking. The link even starts with <code>ca.gov-</code> to give it that government feel, like the digital equivalent of a trench coat and a fake badge.</p>

<p>They sent this to the wrong phone.</p>




<h2>
  
  
  The Forensics
</h2>

<p><strong>Twenty-two hours.</strong> That's the entire lifespan of this operation from domain registration to active phishing campaign. Let's appreciate the speedrun:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Timestamp</th>
<th>Event</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>2026-03-19 14:10</strong></td>
<td>Domain <code>gov-axb.cfd</code> registered via Gname.com (Singapore)</td>
</tr>
<tr>
<td><strong>2026-03-20 ~12:17</strong></td>
<td>Let's Encrypt SSL certificate issued for <code>ca.gov-axb.cfd</code>
</td>
</tr>
<tr>
<td><strong>2026-03-20 ~13:00</strong></td>
<td>Smishing messages begin arriving</td>
</tr>
</tbody>
</table></div>

<p>The domain was registered on a <code>.cfd</code> TLD — that's "Clothing Fashion Design," in case you were wondering what haute couture has to do with the California DMV. It costs about $0.98 to register. The attackers configured <strong>wildcard DNS</strong> pointing everything at <code>47.89.135.90</code> — an Alibaba Cloud server — so <code>ca.gov-axb.cfd</code>, <code>tx.gov-axb.cfd</code>, <code>fl.gov-axb.cfd</code>, and literally any subdomain you can dream up all resolve to the same phishing kit. Seven confirmed state-targeted subdomains on a single domain. Assembly line phishing.</p>

<p>The nameservers (<code>share-dns.com</code>, <code>share-dns.net</code>) are a Chinese DNS provider. The registrar abuse contact is in Singapore. The hosting is Alibaba Cloud's US allocation. It's a jurisdictional lasagna designed to make takedowns as painful as possible.</p>




<h2>
  
  
  Meet the Kit: Lighthouse
</h2>

<p>The phishing page itself is a slick California DMV impersonation — official logo, official color scheme, footer links to real <code>.ca.gov</code> pages. It's built on <strong>Vue.js 3</strong>, bundled with <strong>Vite</strong>, communicating over <strong>Socket.IO</strong> with <strong>AES-128-CBC encrypted WebSocket messages</strong>. This isn't some amateur HTML form. This is a commercial product.</p>

<p>It's called <strong>Lighthouse</strong> — a phishing-as-a-service (PhaaS) platform operated by the <strong>Smishing Triad</strong>, a Chinese cybercrime operation that Google literally filed a federal lawsuit against in November 2025. The operator, attributed to someone going by <strong>Wang Duo Yu</strong>, runs a platform serving <strong>~300 "front desk staff"</strong> worldwide, targeting <strong>316+ brands across 74 countries</strong> through an estimated <strong>194,000+ malicious domains</strong>.</p>

<p>The kit has a real-time operator panel. When a victim lands on the page, the operator sees their session appear in real time. As the victim fills in their name, address, credit card — the data streams live to the operator's dashboard. They can push the victim through verification steps in real time: <em>"Enter the code we just sent to your phone."</em> <em>"Now enter your card PIN."</em> It's live, interactive social engineering at industrial scale.</p>

<p>The victim flow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Landing Page → Contact Info Form → Credit Card Form → Verification Codes → 💀
</code></pre>

</div>



<p>Every field submission, every page navigation, every keystroke-level interaction — all wrapped in AES encryption and shuttled over WebSocket. The kit even has <strong>anti-debugging</strong> protections in its JavaScript, string obfuscation tables, and <strong>mobile User-Agent gating</strong> (it won't even load if you're not on a phone browser).</p>

<p>They thought of everything.</p>

<p>Almost.</p>




<h2>
  
  
  The Reverse Engineering
</h2>

<p>The kit's JavaScript is two files totaling ~570KB of heavily obfuscated code. Here's what that looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">xe</span><span class="p">(){</span><span class="kd">const</span> <span class="nx">t</span><span class="o">=</span><span class="p">[</span><span class="dl">"</span><span class="s2">YWdl</span><span class="dl">"</span><span class="p">,</span><span class="dl">"</span><span class="s2">message</span><span class="dl">"</span><span class="p">,</span><span class="dl">"</span><span class="s2">Y2hhbm</span><span class="dl">"</span><span class="p">,</span><span class="dl">"</span><span class="s2">Zmllb</span><span class="dl">"</span><span class="p">,</span><span class="dl">"</span><span class="s2">c3VibW</span><span class="dl">"</span><span class="p">,</span><span class="dl">"</span><span class="s2">bm90a...</span><span class="dl">"</span><span class="p">];</span>
<span class="c1">// ... 260,000 more characters of this</span>
</code></pre>

</div>



<p>Every meaningful string — event names, API paths, encryption keys — is routed through a deobfuscation function (<code>be()</code>) that indexes into a shuffled, Base64-encoded string table. The Socket.IO connection path isn't <code>/socket.io/</code> like normal. It's been moved. The encryption key and IV aren't in plaintext. Nothing is where it should be.</p>

<p><strong>Step 1: Find the Socket.IO Path</strong></p>

<p>After failing to statically decode the obfuscation (the string table uses a self-modifying shuffle — it rearranges itself during initialization), I pivoted to empirical observation. Loaded the page in a headless browser with a mobile User-Agent, captured the WebSocket traffic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>wss://ca.gov-axb.cfd:443/console/?uuid=&lt;uuid&gt;&amp;EIO=4&amp;transport=websocket
</code></pre>

</div>



<p>The Socket.IO path is <code>/console/</code>. Not <code>/socket.io/</code>. Not <code>/ws/</code>. Not anything in the standard playbook.</p>

<p><strong>Step 2: Crack the Encryption</strong></p>

<p>All WebSocket messages are encrypted. Client sends <code>42["message","&lt;base64&gt;"]</code>, server responds the same way. The actual event data is AES-128-CBC encrypted inside that base64 blob.</p>

<p>From the obfuscated source, I could see the key and IV were assembled from string fragments:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">We</span> <span class="o">=</span> <span class="nc">Be</span><span class="p">(</span><span class="mi">226</span><span class="p">)</span> <span class="o">+</span> <span class="nf">ye</span><span class="p">(</span><span class="mi">250</span><span class="p">)</span> <span class="o">+</span> <span class="nf">ze</span><span class="p">(</span><span class="mi">254</span><span class="p">)</span> <span class="o">+</span> <span class="dl">"</span><span class="s2">LDND</span><span class="dl">"</span><span class="p">;</span>   <span class="c1">// AES key</span>
<span class="kd">const</span> <span class="nx">Se</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">OJGJHHFMJEJB</span><span class="dl">"</span> <span class="o">+</span> <span class="nf">we</span><span class="p">(</span><span class="mi">259</span><span class="p">);</span>                <span class="c1">// AES IV</span>
<span class="nx">Ne</span> <span class="o">=</span> <span class="nx">CryptoJS</span><span class="p">.</span><span class="nx">enc</span><span class="p">.</span><span class="nx">Utf8</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">We</span><span class="p">);</span>                    <span class="c1">// → WordArray</span>
<span class="nx">Te</span> <span class="o">=</span> <span class="nx">CryptoJS</span><span class="p">.</span><span class="nx">enc</span><span class="p">.</span><span class="nx">Utf8</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">Se</span><span class="p">);</span>                    <span class="c1">// → WordArray</span>
</code></pre>

</div>



<p>The <code>Be()</code>, <code>ye()</code>, <code>ze()</code>, <code>we()</code> functions are all aliases for the same deobfuscator, returning short string fragments. I couldn't decode them statically — the shuffle function defeats that approach.</p>

<p>So I intercepted the JavaScript <strong>in flight</strong>. Using Playwright's <code>route</code> API, I caught the JS file as it loaded, injected a single line that exposed the computed key values to the global scope, and read them out:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">await</span> <span class="nx">context</span><span class="p">.</span><span class="nf">route</span><span class="p">(</span><span class="dl">'</span><span class="s1">**/CPD9iRZf.js</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">route</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">route</span><span class="p">.</span><span class="nf">fetch</span><span class="p">();</span>
    <span class="kd">let</span> <span class="nx">body</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">text</span><span class="p">();</span>
    <span class="nx">body</span> <span class="o">=</span> <span class="nx">body</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span>
        <span class="sr">/Se="OJGJHHFMJEJB"</span><span class="se">\+\w</span><span class="sr">+</span><span class="se">\(\d</span><span class="sr">+</span><span class="se">\)</span><span class="sr">/</span><span class="p">,</span>
        <span class="p">(</span><span class="nx">match</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">match</span> <span class="o">+</span> <span class="dl">'</span><span class="s1">;window.__AES_KEY=We;window.__AES_IV=Se</span><span class="dl">'</span>
    <span class="p">);</span>
    <span class="k">await</span> <span class="nx">route</span><span class="p">.</span><span class="nf">fulfill</span><span class="p">({</span> <span class="nx">response</span><span class="p">,</span> <span class="nx">body</span> <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Key: <code>KIKOCCJCFILDLDND</code></strong><br>
<strong>IV: <code>OJGJHHFMJEJBDFAI</code></strong></p>

<p>Verified by decrypting a captured server message:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"data"</span><span class="p">:{</span><span class="nl">"money"</span><span class="p">:</span><span class="s2">""</span><span class="p">,</span><span class="nl">"successUrl"</span><span class="p">:</span><span class="s2">""</span><span class="p">,</span><span class="nl">"currencySymbol"</span><span class="p">:</span><span class="s2">""</span><span class="p">,</span><span class="nl">"products"</span><span class="p">:[]},</span><span class="nl">"event"</span><span class="p">:</span><span class="s2">"userSiteConfig"</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>And a captured client message:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"event"</span><span class="p">:</span><span class="s2">"changleField"</span><span class="p">,</span><span class="nl">"data"</span><span class="p">:{</span><span class="nl">"router"</span><span class="p">:</span><span class="s2">"首页"</span><span class="p">}}</span><span class="w">
</span></code></pre>

</div>



<p>That typo — <code>changleField</code> instead of <code>changeField</code> — is a known Lighthouse kit fingerprint. It's in every deployment. Somewhere, a developer fat-fingered it once, and now it's a forensic signature stamped across 194,000 domains.</p>

<p>Also: <code>首页</code> is Mandarin for "home page." The Chinese-language strings are <em>everywhere</em> in the source: <code>手机验证页</code> (phone verification page), <code>运通CVV验证页</code> (Amex CVV verification page), <code>加密异常!</code> (encryption exception!). For a kit that goes to great lengths to look like the California DMV, it sure does think in Mandarin.</p>




<h2>
  
  
  The Weapon
</h2>

<p>With the encryption cracked and the protocol reverse-engineered, it was time to build.</p>

<p>Not a Python script. Not a Node.js hack. <strong>Rust.</strong> Specifically: an async, multi-session, zero-copy WebSocket flood engine using Tokio, with native AES-128-CBC encryption, realistic fake data generation, and the exact Socket.IO protocol the kit expects.</p>

<p>Each session:</p>

<ol>
<li>Opens a WebSocket to <code>/console/</code> with a mobile User-Agent</li>
<li>Completes the Engine.IO handshake</li>
<li>Sends the Socket.IO connect packet</li>
<li>Responds to server heartbeats (ping <code>2</code> → pong <code>3</code>)</li>
<li>Encrypts and sends <strong>12 events</strong> mimicking a complete victim session:

<ul>
<li>Router navigation (<code>changleField</code> — yes, with the typo)</li>
<li>Full personal info (name, address, city, state, zip, phone, email)</li>
<li>Full credit card data (Luhn-valid card number, holder name, expiry, CVV)</li>
<li>Phone verification code</li>
<li>Email verification code</li>
<li>PIN code</li>
<li>Express CVV re-verification</li>
<li>Custom verification code</li>
</ul>
</li>
<li>Closes gracefully</li>
</ol>

<p>Every piece of fake data is realistic:</p>

<ul>
<li>
<strong>Names</strong> drawn from US census frequency tables</li>
<li>
<strong>Addresses</strong> with real California street patterns and cities</li>
<li>
<strong>ZIP codes</strong> matching California postal zones</li>
<li>
<strong>Phone numbers</strong> with correct California area codes</li>
<li>
<strong>Credit cards</strong> that pass Luhn validation (Visa, Mastercard, Amex, Discover prefixes)</li>
<li>
<strong>Email addresses</strong> with realistic username patterns at common providers</li>
</ul>

<p>From the server's perspective, each session is indistinguishable from a real victim clicking through the phishing flow.</p>




<h2>
  
  
  The Flood
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>╔══════════════════════════════════════════════════════════╗
║       LIGHTHOUSE FLOOD — Pure Rust WebSocket Engine     ║
║       AES-128-CBC | Direct Socket.IO | Max Throughput   ║
╚══════════════════════════════════════════════════════════╝
Target:       https://ca.gov-axb.cfd
WS Path:      /console/
Concurrency:  100
Count:        ∞
</code></pre>

</div>



<p>First test: 5 sessions, 3 concurrent. <strong>60 events, 0 errors.</strong> Proof of concept.</p>

<p>Second test: 200 sessions, 30 concurrent. <strong>2,400 events, 0 errors, 28 seconds.</strong> Warming up.</p>

<p>Third test: 1,000 sessions, 100 concurrent. <strong>12,000 events, 0 errors, 42 seconds.</strong> Now we're talking. ~24 complete fake victims per second.</p>

<p>Then we let it rip.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>╔══════════════════════════════════════════════════════════╗
║                     FLOOD COMPLETE                      ║
╠══════════════════════════════════════════════════════════╣
║  Submitted: 100700  |  Events: 1208984  |  Errors:    0 ║
╚══════════════════════════════════════════════════════════╝
</code></pre>

</div>



<p><strong>One hundred thousand fake victim sessions. 1.2 million encrypted events. Zero errors.</strong></p>

<p>One hour. One machine. One Rust binary. Their operator's dashboard — the same one they use to monitor real victims in real time — is now a firehose of:</p>

<ul>
<li>
<strong>Jennifer Wilson</strong> from San Jose, CA 95132, card ending in 8388</li>
<li>
<strong>Kevin Sanchez</strong> from Chula Vista, CA 92101, card ending in 6501</li>
<li>
<strong>Laura Martin</strong> from Santa Ana, CA 92701, card ending in 4787</li>
<li>... times one hundred thousand.</li>
</ul>

<p>Every single entry looks real. Luhn-valid card numbers that'll fail at the processor. California addresses that go to actual streets. Phone numbers that ring actual area codes. The operators can't tell which data is real and which is garbage. Their entire database is poisoned.</p>

<p>For every real victim they managed to phish today, there are now <strong>thousands</strong> of fake records burying them. Good luck finding the needle in that haystack.</p>




<h2>
  
  
  The Takeaway
</h2>

<p>This particular phishing operation is part of a <strong>massive, well-funded, commercially operated</strong> cybercrime ecosystem. The Smishing Triad runs Lighthouse as a <em>business</em> — subscription model, customer support, Telegram channels, the works. Google sued them. The FBI has received thousands of complaints. They're still operating.</p>

<p>Individual takedowns are whack-a-mole. The domain <code>gov-axb.cfd</code> will die, and tomorrow there'll be <code>gov-xyz.cfd</code> and <code>gov-abc.win</code> and a hundred more. The kit itself rotates, the infrastructure shifts, the TLDs change.</p>

<p>But today? Today, one specific operator's database is garbage. One specific campaign's ROI just cratered. One specific Lighthouse instance's real-time dashboard is scrolling fake Californians faster than any human could possibly triage.</p>

<p>They set up an encrypted, obfuscated, anti-debugging, mobile-gated phishing operation.</p>

<p>We reverse-engineered the encryption in an afternoon and flooded them with a million fake victims before dinner.</p>

<p><strong>Fuck around. Find out.</strong></p>




<h2>
  
  
  Technical Summary
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Detail</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Target</strong></td>
<td>
<code>ca.gov-axb.cfd</code> (California DMV phishing)</td>
</tr>
<tr>
<td><strong>Threat Actor</strong></td>
<td>Smishing Triad / Lighthouse PhaaS</td>
</tr>
<tr>
<td><strong>Kit Stack</strong></td>
<td>Vue.js 3 + Vite + Socket.IO + AES-128-CBC</td>
</tr>
<tr>
<td><strong>Socket.IO Path</strong></td>
<td>
<code>/console/</code> (non-standard)</td>
</tr>
<tr>
<td><strong>Encryption Key</strong></td>
<td>
<code>KIKOCCJCFILDLDND</code> (UTF-8)</td>
</tr>
<tr>
<td><strong>Encryption IV</strong></td>
<td>
<code>OJGJHHFMJEJBDFAI</code> (UTF-8)</td>
</tr>
<tr>
<td><strong>Kit Fingerprint</strong></td>
<td>
<code>changleField</code> typo in all event names</td>
</tr>
<tr>
<td><strong>Flood Tool</strong></td>
<td>Rust + Tokio + tungstenite (async WebSocket)</td>
</tr>
<tr>
<td><strong>Sessions Completed</strong></td>
<td>100,700+</td>
</tr>
<tr>
<td><strong>Events Sent</strong></td>
<td>1,208,984+</td>
</tr>
<tr>
<td><strong>Error Rate</strong></td>
<td>0.000%</td>
</tr>
<tr>
<td><strong>Throughput</strong></td>
<td>~24 complete victim sessions/second</td>
</tr>
<tr>
<td><strong>Duration</strong></td>
<td>~63 minutes</td>
</tr>
</tbody>
</table></div>




<p><em>Filed under: threat intelligence, active defense, phishing disruption, and not taking shit from scammers.</em></p>

