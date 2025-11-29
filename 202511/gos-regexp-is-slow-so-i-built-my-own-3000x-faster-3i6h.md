---
Title: Go's Regexp is Slow. So I Built My Own - up to 3000x Faster
Description: 
Author: Andrey Kolkov
Date: 2025-11-29T21:27:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been writing Go for years. Love the language. But there's one thing that always bothered me: <strong>regex performance</strong>.</p>

<p>Recently I was processing large text files (logs, traces, datasets) and kept hitting the same wall - <code>regexp.Find()</code> consuming 70-80% of CPU time. Not on complex patterns. On simple stuff like <code>.*error.*connection.*</code>.</p>

<p>So I did what any reasonable developer would do: spent 6 months building <a href="https://github.com/coregx/coregex" rel="noopener noreferrer">coregex</a>, a drop-in replacement for Go's regexp that's <strong>3-3000x faster</strong> while keeping the same O(n) guarantees.</p>

<p>Here's how and why.</p>




<h2>
  
  
  The Problem
</h2>

<p>Let's be direct: <strong>Go's regexp is not optimized for performance</strong>.</p>

<p>It uses Thompson's NFA exclusively - great for correctness (no ReDoS, guaranteed O(n)), terrible for speed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Searching for "error" in 1MB file</span>
<span class="n">re</span> <span class="o">:=</span> <span class="n">regexp</span><span class="o">.</span><span class="n">MustCompile</span><span class="p">(</span><span class="s">`.*error.*`</span><span class="p">)</span>
<span class="n">start</span> <span class="o">:=</span> <span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span>
<span class="n">re</span><span class="o">.</span><span class="n">Find</span><span class="p">(</span><span class="n">data</span><span class="p">)</span> <span class="c">// 27ms</span>
</code></pre>

</div>



<p><strong>Why so slow?</strong></p>

<ol>
<li>No SIMD - processes bytes sequentially</li>
<li>No prefilters - scans entire input even when pattern clearly won't match</li>
<li>Single engine - uses same algorithm for all patterns</li>
<li>Allocation overhead in hot paths</li>
</ol>

<p>Compare with Rust's regex crate on same pattern: <strong>21µs</strong>. That's <strong>1,285x faster</strong>.</p>

<p>The gap is real. And fixable.</p>




<h2>
  
  
  The Rust Rabbit Hole
</h2>

<p>I started digging into how Rust achieves this. Turns out, they use <strong>multi-engine architecture</strong>:</p>

<ol>
<li>
<strong>Literal extraction</strong>: Pull out fixed strings from pattern (<code>.*error.*</code> → literal "error")</li>
<li>
<strong>SIMD prefilter</strong>: Use AVX2 to find candidates (12x faster byte search)</li>
<li>
<strong>Strategy selection</strong>: Pick optimal engine per pattern

<ul>
<li>DFA for simple patterns</li>
<li>NFA for complex patterns</li>
<li>Reverse search for suffix patterns</li>
<li>
<strong>Specialized engines</strong> for common cases</li>
</ul>
</li>
<li>
<strong>Zero allocations</strong>: Object pools, preallocated buffers</li>
</ol>

<p>Go has none of this. <code>regexp</code> package is ~10 years old, hasn't changed much.</p>

<p>Could I bring this to Go? Challenge accepted.</p>




<h2>
  
  
  Building coregex - Month by Month
</h2>

<h3>
  
  
  Month 1-2: SIMD Primitives
</h3>

<p>Started with the foundation. Go's <code>bytes.IndexByte</code> is okay. AVX2 is better.</p>

<p>Wrote assembly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Find byte in slice using AVX2 (32 bytes parallel)
TEXT ·memchr(SB), NOSPLIT, $0-40
    MOVQ    haystack+0(FP), DI
    MOVQ    len+8(FP), CX
    MOVBQZX needle+24(FP), AX

    // Broadcast needle to YMM0
    VMOVD   AX, X0
    VPBROADCASTB X0, Y0

loop:
    VMOVDQU (DI), Y1       // Load 32 bytes
    VPCMPEQB Y0, Y1, Y2    // Compare (parallel)
    VPMOVMSKB Y2, AX       // Extract mask
    TESTL   AX, AX
    JNZ     found
    // ... continue
</code></pre>

</div>



<p><strong>Benchmark</strong> (finding 'e' in 64KB):</p>

<ul>
<li>stdlib: 8,367 ns</li>
<li>coregex: 679 ns</li>
<li><strong>12.3x faster</strong></li>
</ul>

<p>Then <code>memmem</code> (substring search), then <code>teddy</code> (multi-pattern SIMD).</p>

<h3>
  
  
  Month 3-4: Strategy Selection
</h3>

<p>Not all patterns are equal. Suffix patterns need reverse search. Inner literal patterns need bidirectional search.</p>

<p>Built meta-engine:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="n">selectStrategy</span><span class="p">(</span><span class="n">pattern</span> <span class="o">*</span><span class="n">syntax</span><span class="o">.</span><span class="n">Regexp</span><span class="p">)</span> <span class="n">Strategy</span> <span class="p">{</span>
    <span class="c">// Extract literals</span>
    <span class="n">prefix</span> <span class="o">:=</span> <span class="n">extractPrefix</span><span class="p">(</span><span class="n">pattern</span><span class="p">)</span>
    <span class="n">suffix</span> <span class="o">:=</span> <span class="n">extractSuffix</span><span class="p">(</span><span class="n">pattern</span><span class="p">)</span>
    <span class="n">inner</span> <span class="o">:=</span> <span class="n">extractInner</span><span class="p">(</span><span class="n">pattern</span><span class="p">)</span>

    <span class="c">// Choose optimal strategy</span>
    <span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">suffix</span><span class="p">)</span> <span class="o">&gt;=</span> <span class="m">3</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">UseReverseSuffix</span>  <span class="c">// 1000x for .*\.txt</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">inner</span><span class="p">)</span> <span class="o">&gt;=</span> <span class="m">3</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">UseReverseInner</span>   <span class="c">// 3000x for .*error.*</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">prefix</span><span class="p">)</span> <span class="o">&gt;=</span> <span class="m">3</span> <span class="p">{</span>
        <span class="k">return</span> <span class="n">UseDFA</span>            <span class="c">// 5-10x for prefix.*</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="n">UseAdaptive</span>           <span class="c">// Try DFA, fallback NFA</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Each pattern gets the <strong>right tool for the job</strong>.</p>

<h3>
  
  
  Month 5: ReverseInner - The Killer Optimization
</h3>

<p>This is where it got interesting.</p>

<p><strong>Problem</strong>: Pattern <code>.*error.*connection.*</code> on large file.</p>

<p><strong>stdlib approach</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>for each position:
    try to match .*error.*connection.*
    (scan entire input, backtracking, slow)
</code></pre>

</div>



<p><strong>coregex approach</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Find "connection" with SIMD → 200ns (not 27ms!)
2. From "connection", scan backward for "error" → fast DFA
3. From "connection", scan forward for .* → fast DFA
4. First match = done (leftmost-first semantics)
</code></pre>

</div>



<p><strong>Benchmark</strong> (250KB file):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>stdlib:  12.6ms
coregex: 4µs
speedup: 3,154x
</code></pre>

</div>



<p>Not 3x. Not 30x. <strong>Three thousand times faster.</strong></p>

<h3>
  
  
  Month 6: Zero Allocations
</h3>

<p>Hot paths must not allocate. Profiled with <code>pprof</code>, eliminated every allocation:</p>

<ul>
<li>Object pooling for DFA states</li>
<li>Preallocated buffers</li>
<li>Careful escape analysis</li>
<li>Stack-only data structures</li>
</ul>

<p>Result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>BenchmarkIsMatch-8  1000000  1.2 µs/op  0 B/op  0 allocs/op
</code></pre>

</div>



<p><strong>Zero.</strong></p>




<h2>
  
  
  How It Actually Works
</h2>

<p>Let's trace <code>.*error.*connection.*</code> step by step.</p>

<p><strong>Input</strong>: 250KB log file with "connection" appearing 5 times.</p>

<p><strong>stdlib (NFA)</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Position 0:    Try .*error.*connection.* → fail
Position 1:    Try .*error.*connection.* → fail
...
Position 250000: Try .*error.*connection.* → fail
Time: 12.6ms
</code></pre>

</div>



<p><strong>coregex (ReverseInner)</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SIMD scan: Find "connection" → positions [1245, 5678, ...]
At position 1245:
  Reverse DFA: Scan backward, find "error" at 1230 ✓
  Forward DFA: Scan forward, match .* ✓
  Return [1230, 1260]
Time: 4µs
</code></pre>

</div>



<p>We don't scan 250,000 positions. We check <strong>5 candidates</strong> (SIMD-found literals). That's why it's 3000x faster.</p>




<h2>
  
  
  The Architecture
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────┐
│     Meta-Engine         │
│  (Strategy Selection)   │
└────────┬────────────────┘
         │
    ┌────┴────┐
    │Prefilter│──► memchr (AVX2)
    └────┬────┘──► memmem (SIMD)
         │     ──► teddy (multi-pattern)
         │
┌────────┴──────────────┐
│  ┌────┐  ┌────┐  ┌───┐│
│  │DFA │  │NFA │  │Rev││
│  └────┘  └────┘  └───┘│
└───────────────────────┘
</code></pre>

</div>



<p>Simple idea: <strong>Use specialized algorithms for specialized patterns</strong>.</p>




<h2>
  
  
  Real Benchmarks
</h2>

<p>Here's what matters - patterns you actually use:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Pattern</th>
<th>Size</th>
<th>stdlib</th>
<th>coregex</th>
<th>Speedup</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>.*\.txt$</code></td>
<td>1MB</td>
<td>27ms</td>
<td>21µs</td>
<td><strong>1,314x</strong></td>
</tr>
<tr>
<td><code>.*error.*</code></td>
<td>250KB</td>
<td>12.6ms</td>
<td>4µs</td>
<td><strong>3,154x</strong></td>
</tr>
<tr>
<td><code>(?i)error</code></td>
<td>32KB</td>
<td>1.23ms</td>
<td>4.7µs</td>
<td><strong>263x</strong></td>
</tr>
<tr>
<td><code>\w+@\w+\.\w+</code></td>
<td>1KB</td>
<td>688ns</td>
<td>196ns</td>
<td><strong>3.5x</strong></td>
</tr>
</tbody>
</table></div>

<p>Even simple patterns are faster.</p>




<h2>
  
  
  API - Drop-In Replacement
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Change one line</span>
<span class="c">// import "regexp"</span>
<span class="k">import</span> <span class="s">"github.com/coregx/coregex"</span>

<span class="n">re</span> <span class="o">:=</span> <span class="n">coregex</span><span class="o">.</span><span class="n">MustCompile</span><span class="p">(</span><span class="s">`.*error.*`</span><span class="p">)</span>
<span class="n">re</span><span class="o">.</span><span class="n">Find</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>    <span class="c">// 3000x faster, same API</span>
</code></pre>

</div>



<p>Full compatibility:</p>

<ul>
<li>✅ <code>Find</code>, <code>FindAll</code>, <code>Match</code>, <code>MatchString</code>
</li>
<li>✅ Capture groups with <code>FindSubmatch</code>
</li>
<li>✅ Named captures with <code>SubexpNames()</code>
</li>
<li>✅ <code>Replace</code>, <code>ReplaceAll</code>, <code>Split</code>
</li>
<li>✅ Unicode support via <code>regexp/syntax</code>
</li>
</ul>

<p>Only difference: <strong>no backreferences</strong> (they require backtracking → exponential worst-case).</p>




<h2>
  
  
  When to Use It
</h2>

<p><strong>Use coregex if:</strong></p>

<ul>
<li>Regex is a bottleneck (profile first!)</li>
<li>You have patterns with wildcards (<code>.*error.*</code>)</li>
<li>You do case-insensitive matching at scale</li>
<li>You parse logs, traces, or large text files</li>
<li>Performance matters</li>
</ul>

<p><strong>Stick with stdlib if:</strong></p>

<ul>
<li>Regex is not your bottleneck</li>
<li>You need maximum API stability (coregex is v0.8, pre-1.0)</li>
<li>You want zero dependencies</li>
</ul>

<p><strong>Performance by pattern type:</strong></p>

<ul>
<li>Suffix patterns (<code>.*\.txt</code>): <strong>1000-1500x</strong>
</li>
<li>Inner patterns (<code>.*error.*</code>): <strong>2000-3000x</strong>
</li>
<li>Case-insensitive: <strong>100-300x</strong>
</li>
<li>Simple patterns: <strong>2-10x</strong>
</li>
</ul>




<h2>
  
  
  Part of CoreGX
</h2>

<p>coregex is the first release from <a href="https://github.com/coregx" rel="noopener noreferrer">CoreGX</a> - an org I created for high-quality Go libraries.</p>

<p><strong>Mission</strong>: Build production-grade tools that should exist but don't.</p>

<p><strong>Why?</strong> Go stdlib prioritizes simplicity and correctness. Great for most cases. But sometimes you need <strong>performance without sacrificing correctness</strong>. That's CoreGX.</p>

<p>Future projects:</p>

<ul>
<li>JSON parser with SIMD</li>
<li>Advanced concurrency primitives</li>
<li>Zero-copy data structures</li>
</ul>

<p>All MIT. All production-ready. All built for real use.</p>




<h2>
  
  
  Trying It Out
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go get github.com/coregx/coregex@v0.8.0
</code></pre>

</div>



<p>Run benchmarks on <strong>your</strong> patterns. If it's faster, great. If not, file an issue - I want to know why.</p>

<p><strong>Project stats:</strong></p>

<ul>
<li>88.3% test coverage</li>
<li>Zero known bugs</li>
<li>18.5K lines of code</li>
<li>6 months development</li>
<li>MIT licensed</li>
</ul>

<p><strong>Roadmap:</strong></p>

<ul>
<li>v0.9.0 (Q1 2026): Look-around assertions</li>
<li>v1.0.0 (Q2 2026): API stability guarantee</li>
<li>v1.1.0+: ARM NEON SIMD</li>
</ul>




<h2>
  
  
  Technical Deep Dive: SIMD Assembly
</h2>

<p>If you're into this stuff, here's how AVX2 memchr works:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Find byte 'e' in haystack (processes 32 bytes in parallel)
VPBROADCASTB needle, YMM0    // Broadcast to 256-bit register

loop:
    VMOVDQU (haystack), YMM1  // Load 32 bytes (unaligned)
    VPCMPEQB YMM0, YMM1, YMM2 // Compare all 32 at once
    VPMOVMSKB YMM2, RAX       // Extract bitmask
    TEST RAX, RAX             // Any matches?
    JNZ found                 // Jump if found
    ADD haystack, 32          // Next 32 bytes
    JMP loop

found:
    TZCNT RAX, RAX            // Find first set bit
    ADD result, haystack
</code></pre>

</div>



<p>This is why it's 12x faster than Go's byte-by-byte approach.</p>




<h2>
  
  
  Lessons Learned
</h2>

<ol>
<li>
<strong>Profile before optimizing</strong> - but don't stop at profiling</li>
<li>
<strong>SIMD is accessible</strong> - Go's asm syntax is surprisingly clean</li>
<li>
<strong>Algorithm matters more than micro-opts</strong> - ReverseInner is 3000x not because of clever asm, but because of smart strategy</li>
<li>
<strong>Zero allocs is achievable</strong> - requires discipline but pays off</li>
<li>
<strong>Compatibility is hard</strong> - matching stdlib behavior has edge cases</li>
</ol>




<h2>
  
  
  Open Source &amp; Contributions
</h2>

<p>Source: <a href="https://github.com/coregx/coregex" rel="noopener noreferrer">github.com/coregx/coregex</a></p>

<p>PRs welcome for:</p>

<ul>
<li>ARM NEON implementation</li>
<li>Additional SIMD primitives</li>
<li>Test coverage improvements</li>
<li>Documentation</li>
</ul>

<p>Not welcome:</p>

<ul>
<li>"Rewrite in Rust" (this is a Go library)</li>
<li>Breaking API changes pre-1.0</li>
<li>Features without benchmarks</li>
</ul>




<h2>
  
  
  Why I Built This
</h2>

<p>Honest answer: <strong>I wanted to learn</strong>.</p>

<p>How do modern regex engines work? Can I replicate Rust's performance in Go? What are the limits of Go's runtime?</p>

<p>Turned out the answer was: <strong>Yes, you can</strong>, and <strong>it's actually not that hard</strong> if you use the right algorithms.</p>

<p>Also, I needed this for my own projects. I maintain <a href="https://github.com/kolkov/racedetector" rel="noopener noreferrer">racedetector</a> (pure-Go race detection) and other tools that parse a lot of text. This will help those projects too once v1.0 lands.</p>




<h2>
  
  
  The Bottom Line
</h2>

<p>Go's regexp is fine for most use cases. But if regex is your bottleneck, you have options now.</p>

<p>coregex isn't magic - it's just <strong>better algorithms + SIMD</strong>. The techniques are well-known (Rust uses them, RE2 uses parts of them). I just brought them to Go.</p>

<p>If you're spending significant CPU time in regex, benchmark coregex. If it helps, use it. If you find bugs or slowness, report them.</p>

<p>Links:</p>

<ul>
<li><a href="https://github.com/coregx/coregex" rel="noopener noreferrer">GitHub</a></li>
<li><a href="https://github.com/kolkov" rel="noopener noreferrer">My profile</a></li>
<li><a href="https://github.com/coregx/coregex/discussions" rel="noopener noreferrer">Discussions</a></li>
</ul>

<p>Star if useful. File issues if broken. PRs if you want to contribute.</p>




<p><em>P.S.</em> The most fun part? Writing AVX2 assembly at 2am and having it actually work. Second most fun? Seeing 3000x in benchmark output and thinking "that can't be right" (it was).</p>

<p><em>Built by <a href="https://github.com/kolkov" rel="noopener noreferrer">@kolkov</a> as part of <a href="https://github.com/coregx" rel="noopener noreferrer">CoreGX</a> - Production Go libraries.</em></p>

