---
Title: Stop Leaking API Keys in your AI Agent Logs: A Go Sidecar Approach
Description: 
Author: Ilya Ploskovitov
Date: 2026-02-05T21:41:24.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Stop Leaking API Keys in your AI Agent Logs: A Go Sidecar Approach
</h1>

<p><strong>Subtitle:</strong> The Hidden Privacy Leak in your AI Agents (and why your LLM "Audit Logs" are a GDPR Nightmare)</p>




<h2>
  
  
  1. The Problem
</h2>

<p>Everyone is building AI agents right now. Whether you're using LangChain, AutoGPT, or custom loops, you are almost certainly logging their work. You keep traces of every step to debug reasoning loops or monitor costs.</p>

<p><strong>Here lies the pain:</strong> Everything goes into these logs. User prompts, model responses, and raw JSON payloads from APIs.</p>

<h3>
  
  
  The Visualization
</h3>

<p>Imagine this scenario:</p>

<p><strong>Input Log:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"user"</span><span class="p">:</span><span class="w"> </span><span class="s2">"aragossa"</span><span class="p">,</span><span class="w"> </span><span class="nl">"prompt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"My key is sk-live-123456"</span><span class="p">}</span><span class="w"> 
</span></code></pre>

</div>



<p><strong>What your Logging System Saved:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"user"</span><span class="p">:</span><span class="w"> </span><span class="s2">"aragossa"</span><span class="p">,</span><span class="w"> </span><span class="nl">"prompt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"My key is sk-live-123456"</span><span class="p">}</span><span class="w"> 
</span></code></pre>

</div>



<p>If a user pastes their password, API key, or PII into the chat, or if an API returns a sensitive internal token, that data is now permanently etched into your Elasticsearch, Datadog, or S3 bucket.</p>

<p><strong>The Consequence:</strong> Your fine-tuning dataset is now "poisoned" with real user data. This is a massive GDPR violation and a ticking security time bomb. You can't just "delete" it if you don't know where it is.</p>

<h2>
  
  
  2. The Gap: Why usual methods fail
</h2>

<ul>
<li>  <strong>Regex?</strong> Good luck maintaining a regex list for every possible API key format, session token, and PII variation in existence. It’s a game of whack-a-mole you will lose.</li>
<li>  <strong>ML-based protection?</strong> Too slow. If your agent operates in real-time, you cannot afford a 500ms roundtrip to a BERT model just to sanitize specific log lines. PII scans often become the bottleneck.</li>
<li>  <strong>Python-native logic?</strong> Processing massive text streams in an interpreted language (like Python) adds significant CPU overhead per log line compared to a compiled Go binary. In high-throughput pipes, this latency adds up fast.</li>
<li>
<p><strong>Existing Observability Tools?</strong><br>
Systems like Fluent Bit, Datadog, or OpenTelemetry already offer redaction and PII masking, usually via pattern rules and regex. For many workloads that’s perfectly fine. The trade‑off is that these pipelines are not optimized for AI‑agent traces: they either run late in the pipeline (after logs have already left the pod) or rely on configuration‑heavy pattern catalogs that are hard to keep up‑to‑date in a world of ever‑changing API keys and internal tokens.</p>

<p>Static scanners like <strong>TruffleHog</strong> shine for repositories and CI, where you scan code at rest. They’re not meant to sit inline on a hot log stream and make sub‑millisecond decisions on every line.</p>

<p>Where <strong>PII‑Shield</strong> is different is not in “inventing redaction”, but in the combination of techniques tailored for AI agents: entropy + bigram signals for unknown secrets, <strong>deterministic HMAC</strong> instead of <code>***</code> for referential integrity, and deep JSON traversal to keep your log schemas intact while still scrubbing sensitive values.</p>
</li>
</ul>

<h2>
  
  
  3. The Implementation: Enter PII-Shield
</h2>

<p>Meet <strong>PII-Shield</strong>. It’s a lightweight sidecar written in Go that sits right next to your agent.</p>

<h3>
  
  
  Killer Feature #1: Entropy-based Detection
</h3>

<p>We don't just search for "password=". We look for <strong>chaos</strong>.<br>
API keys and authentication tokens naturally have high "entropy" (randomness/complexity). Normal human speech has low entropy. By calculating the mathematical complexity of strings, we can flag 64-character hex strings or base64 blobs without knowing their specific format.</p>
<h3>
  
  
  Killer Feature #2: Deterministic HMAC
</h3>

<p>This is the feature that caught attention on Hacker News.<br>
We don't just replace secrets with <code>***</code>. We turn <code>secret123</code> into <code>[HIDDEN:a1b2c3]</code>.</p>

<p><strong>Input Log:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"user"</span><span class="p">:</span><span class="w"> </span><span class="s2">"aragossa"</span><span class="p">,</span><span class="w"> </span><span class="nl">"prompt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"My key is sk-live-123456"</span><span class="p">}</span><span class="w"> 
</span></code></pre>

</div>



<p><strong>PII-Shield Output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="nl">"user"</span><span class="p">:</span><span class="w"> </span><span class="s2">"aragossa"</span><span class="p">,</span><span class="w"> </span><span class="nl">"prompt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"My key is [HIDDEN:8f2a1b]"</span><span class="p">}</span><span class="w"> 
</span></code></pre>

</div>



<p><strong>Why?</strong><br>
This is a deterministic HMAC (Hash-based Message Authentication Code).</p>

<ul>
<li>  It allows you to <strong>trace</strong> a specific user or session across multiple log lines without knowing who they are.</li>
<li>  It preserves <strong>Referential Integrity</strong> for debugging. You can see that "Session A" failed 5 times, but you validly cannot see the Session ID itself.</li>
</ul>
<h3>
  
  
  Killer Feature #3: Statistical Adaptive Threshold
</h3>

<p>PII-Shield doesn't just use a hardcoded number. It learns the "baseline noise" of your logs. By calculating the mean and standard deviation ($2\sigma$), it automatically adjusts the sensitivity to your specific environment.</p>
<h2>
  
  
  4. The Logic: Sidecar Architecture
</h2>

<p>The architecture is dead simple, leveraging the power of Kubernetes sidecars or UNIX pipes.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2a293mf3qpmfv2r502b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2a293mf3qpmfv2r502b.png" alt=" " width="800" height="442"></a></p>

<p>Your agent simply writes logs to stdout. PII-Shield intercepts the stream, scans it in real-time with near-zero overhead, sanitizes it, and passes it forward.</p>
<h2>
  
  
  5. The Technical Meat
</h2>

<p>Why Go? Because we need raw speed and no dependencies.</p>
<h3>
  
  
  1. Entropy &amp; Bigrams (The Math)
</h3>

<p>Here is how we calculate the "Chaos" (Entropy) of a token in <a href="https://github.com/aragossa/pii-shield/blob/main/pkg/scanner/scanner.go" rel="noopener noreferrer">scanner.go</a>. We use a combination of Shannon Entropy, Character Class Bonuses, and English Bigram analysis.</p>

<p>The <strong>Bigram Check</strong> is crucial: it penalizes strings that look like valid English (common letter pairs) and boosts score for "unnatural" strings. (Note: This is optimized for English but can be disabled or tuned via <code>PII_DISABLE_BIGRAM_CHECK</code> for other languages).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// From scanner.go</span>
<span class="k">func</span> <span class="n">CalculateComplexity</span><span class="p">(</span><span class="n">token</span> <span class="kt">string</span><span class="p">)</span> <span class="kt">float64</span> <span class="p">{</span>
    <span class="c">// 1. Shannon Entropy</span>
    <span class="n">entropy</span> <span class="o">:=</span> <span class="n">calculateShannon</span><span class="p">(</span><span class="n">token</span><span class="p">)</span> 

    <span class="c">// 2. Class Bonus (Upper, Lower, Digit, Symbols)</span>
    <span class="c">// bonus := float64(classes-1) * 0.5</span>
    <span class="n">bonus</span> <span class="o">:=</span> <span class="n">calculateClassBonus</span><span class="p">(</span><span class="n">token</span><span class="p">)</span> 

    <span class="c">// 3. Bigram Check (English Likelihood)</span>
    <span class="c">// Penalizes common English, boosts random noise</span>
    <span class="n">bigramScore</span> <span class="o">:=</span> <span class="n">calculateBigramAdjustment</span><span class="p">(</span><span class="n">token</span><span class="p">)</span> 

    <span class="k">return</span> <span class="n">entropy</span> <span class="o">+</span> <span class="n">bonus</span> <span class="o">+</span> <span class="n">bigramScore</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. False Positives? (Whitelists)
</h3>

<p>"Entropy is great, but won't it eat my Git Hashes or UUIDs?"<br>
PII-Shield includes built-in <strong>Whitelists</strong> (<a href="https://github.com/aragossa/pii-shield/blob/main/pkg/scanner/scanner.go#670-760" rel="noopener noreferrer">isSafe</a> function) for standard identifiers like UUIDs, IPv6 addresses, Git Commit hashes (SHA-1), and MongoDB ObjectIDs. This ensures your debugging data stays intact while secrets get redacted.</p>
<h3>
  
  
  3. Credit Card Detection (Luhn Algorithm)
</h3>

<p>Entropy isn't enough for credit cards, as numbers often have low randomness. PII-Shield includes a high-performance implementation of the <strong>Luhn Algorithm</strong> to scan for valid card checksums in the stream (<a href="https://github.com/aragossa/pii-shield/blob/main/pkg/scanner/scanner.go#813-883" rel="noopener noreferrer">FindLuhnSequences</a>).</p>
<h3>
  
  
  4. Deep JSON Inspection
</h3>

<p>For JSON logs, PII-Shield performs deep inspection (<a href="https://github.com/aragossa/pii-shield/blob/main/pkg/scanner/scanner.go#962-981" rel="noopener noreferrer">processJSONLine</a>), preserving the schema while redacting values. <br>
<em>Note: PII-Shield re-serializes JSON (using <code>json.Marshal</code>), which may change key order/sorting. It guarantees semantic integrity but is best used early in your pipeline, before any byte-sensitive steps (like signing or exact-diff comparisons).</em></p>
<h3>
  
  
  5. Deterministic Redaction
</h3>

<p>Here is the HMAC logic using a secure Salt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">redactWithHMAC</span><span class="p">(</span><span class="n">sensitiveData</span> <span class="kt">string</span><span class="p">)</span> <span class="kt">string</span> <span class="p">{</span>
    <span class="c">// CurrentConfig.Salt is loaded securely from env vars</span>
    <span class="n">mac</span> <span class="o">:=</span> <span class="n">hmac</span><span class="o">.</span><span class="n">New</span><span class="p">(</span><span class="n">sha256</span><span class="o">.</span><span class="n">New</span><span class="p">,</span> <span class="n">currentConfig</span><span class="o">.</span><span class="n">Salt</span><span class="p">)</span>
    <span class="n">mac</span><span class="o">.</span><span class="n">Write</span><span class="p">([]</span><span class="kt">byte</span><span class="p">(</span><span class="n">sensitiveData</span><span class="p">))</span>
    <span class="n">hash</span> <span class="o">:=</span> <span class="n">hex</span><span class="o">.</span><span class="n">EncodeToString</span><span class="p">(</span><span class="n">mac</span><span class="o">.</span><span class="n">Sum</span><span class="p">(</span><span class="no">nil</span><span class="p">))</span>
    <span class="c">// We only keep a short prefix for tracing identity</span>
    <span class="k">return</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">"[HIDDEN:%s]"</span><span class="p">,</span> <span class="n">hash</span><span class="p">[</span><span class="o">:</span><span class="m">6</span><span class="p">])</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The main loop (<a href="https://github.com/aragossa/pii-shield/blob/main/cmd/cleaner/main.go" rel="noopener noreferrer">cmd/cleaner/main.go</a>) is a highly efficient buffered reader:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">reader</span> <span class="o">:=</span> <span class="n">bufio</span><span class="o">.</span><span class="n">NewScanner</span><span class="p">(</span><span class="n">os</span><span class="o">.</span><span class="n">Stdin</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">reader</span><span class="o">.</span><span class="n">Scan</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">text</span> <span class="o">:=</span> <span class="n">reader</span><span class="o">.</span><span class="n">Text</span><span class="p">()</span>
        <span class="c">// Core logic: 0 allocations for safe lines</span>
        <span class="c">// Imports github.com/aragossa/pii-shield/pkg/scanner</span>
        <span class="n">cleaned</span> <span class="o">:=</span> <span class="n">scanner</span><span class="o">.</span><span class="n">ScanAndRedact</span><span class="p">(</span><span class="n">text</span><span class="p">)</span> 
        <span class="n">fmt</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="n">cleaned</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  A Note on Scope: High-Entropy vs. Natural Language
</h3>

<p>Let's be clear: PII-Shield is laser-focused on <strong>High-Entropy secrets</strong> (API keys, tokens, auth headers) and structural patterns (Credit Cards). It is <strong>not</strong> a magic NLP bullet for detecting names like "John Smith" or free-text addresses. For that, you should treat PII-Shield as a low-latency "first line of defense" for your infrastructure, potentially complemented by heavier offline NLP tools for semantic analysis.</p>

<h2>
  
  
  6. How to try it
</h2>

<p>I am looking for edge cases. If you are building AI agents, try running your trace logs through this and see what it catches (or misses).</p>

<p>You can run it locally with Docker:<br>
<code>docker run -i -e PII_SALT="mysalt" pii-shield &lt; logs.txt</code></p>

<ul>
<li>  <strong>GitHub</strong>: <a href="https://github.com/aragossa/pii-shield" rel="noopener noreferrer">https://github.com/aragossa/pii-shield</a>
</li>
</ul>

<p><strong>Why this matters:</strong><br>
Security often trails behind innovation. With the explosion of AI Agents, we are generating massive amounts of sensitive data in logs. PII-Shield is a "drop-in" safety net to ensure your innovation doesn't become a liability.</p>

