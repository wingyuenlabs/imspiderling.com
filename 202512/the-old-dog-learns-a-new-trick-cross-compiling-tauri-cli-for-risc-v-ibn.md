---
Title: The Old Dog Learns a New Trick: Cross-Compiling Tauri CLI for RISC-V
Description: 
Author: Bruno Verachten
Date: 2025-12-23T22:01:34.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Photo by <a href="https://unsplash.com/@dariuszpiosik" rel="noopener noreferrer">Dariusz Piosik</a> on Unsplash</em></p>

<h2>
  
  
  Previously, on "Bruno Fights RISC-V"
</h2>

<p>If you missed the first episode (<a href="https://dev.to/gounthar/adding-risc-v-support-to-armbian-imager-a-tale-of-qemu-tauri-and-deja-vu-18nl">available here</a>), here's the recap: I tried adding RISC-V builds to Armbian Imager (a Tauri app), hit a wall with 6+ hour QEMU emulation times, and realized the real fix was upstream. Tauri CLI ships pre-built binaries for x64, ARM64, macOS, Windows... but not RISC-V.</p>

<p>My plan was simple: use my physical Banana Pi F3 as a self-hosted GitHub runner. Native RISC-V compilation. No emulation overhead. I'd done this before with Docker builds. Easy.</p>

<p>Famous last words. Again.</p>

<h3>
  
  
  <strong>🚀 The Self-Hosted Runner Approach</strong>
</h3>

<p>I wasn't walking into this blind. I already had working RISC-V runners from my Docker-for-RISC-V project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">jobs</span><span class="pi">:</span>
  <span class="na">build</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">self-hosted</span><span class="pi">,</span> <span class="nv">riscv64</span><span class="pi">]</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">cargo build --release</span>
</code></pre>

</div>



<p>Two machines at home (192.168.1.185 and 192.168.1.36), both Banana Pi F3 boards, both registered as GitHub runners. They've been churning out Docker engine builds for months. Adding Tauri CLI to the mix seemed natural.</p>

<p>And you know what? <strong>It worked.</strong></p>

<h2>
  
  
  The Working Solution (That Got Replaced)
</h2>

<h3>
  
  
  <strong>✅ First Blood</strong>
</h3>

<p>The self-hosted runner approach actually succeeded. Full workflow run, actual binary produced:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Build time</td>
<td><strong>1h 2m 28s</strong></td>
</tr>
<tr>
<td>Binary</td>
<td>ELF 64-bit RISC-V, RVC, double-float ABI</td>
</tr>
<tr>
<td>Size</td>
<td>16 MB (stripped)</td>
</tr>
<tr>
<td>Version</td>
<td>tauri-cli 2.9.6</td>
</tr>
</tbody>
</table></div>

<p>Proof: <a href="https://github.com/gounthar/tauri/releases/tag/tauri-cli-v2.9.6" rel="noopener noreferrer">actual release with actual binary</a>.</p>

<p>I was ready to submit the PR. Self-hosted runners, conditional logic to skip them if not available, fallback to QEMU for the brave souls with 6 hours to spare. A complete solution.</p>

<p>Then FabianLars left a comment on my PR.</p>

<h3>
  
  
  <strong>🤔 "Have you considered cross?"</strong>
</h3>

<p>I stared at the screen. Cross? Cross-compilation? I'd just spent an hour in the previous session explaining why cross-compilation was a nightmare for WebKit-dependent projects. Sysroots, symlink hell, version mismatches. I'd tried it. I'd abandoned it.</p>

<p>But FabianLars wasn't talking about manual cross-compilation. He was talking about <a href="https://github.com/cross-rs/cross" rel="noopener noreferrer">cross-rs</a>.</p>

<blockquote>
<p>cross should work pretty well for the tauri cli</p>

<p>— FabianLars, GitHub PR comment</p>
</blockquote>

<p>I had never heard of cross-rs.</p>

<p>Let me say that again, because it bears repeating: I've been doing ARM32 cross-compilation since 2013. I've set up sysroots. I've dealt with qemu-user-static. I've maintained unofficial Node.js builds for exotic architectures. I've built Docker images on physical RISC-V hardware. <strong>Twelve years</strong> of this.</p>

<p>And I had never heard of cross-rs.</p>

<h2>
  
  
  What Is Cross-rs?
</h2>

<h3>
  
  
  <strong>📦 The Tool I Should Have Known About</strong>
</h3>

<p>Cross is a "zero setup" cross compilation tool for Rust. Instead of manually setting up toolchains and sysroots, it uses pre-built Docker containers with everything already configured.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install it</span>
cargo <span class="nb">install </span>cross

<span class="c"># Use it exactly like cargo</span>
cross build <span class="nt">--target</span> riscv64gc-unknown-linux-gnu <span class="nt">--release</span>
</code></pre>

</div>



<p>That's it. No sysroot configuration. No toolchain setup. No symlink archaeology. It just... works.</p>

<p>The magic is Docker containers maintained by the cross-rs team. For RISC-V, you get:</p>

<ul>
<li>The GNU toolchain (<code>riscv64-unknown-linux-gnu-gcc</code>)</li>
<li>Proper sysroot with glibc</li>
<li>QEMU for running test binaries</li>
<li>All the standard Rust cross-compilation targets</li>
</ul>

<p>When I ran <code>cross build</code> for the first time, I expected it to fail spectacularly. WebKit dependencies! Linker errors! Missing symbols! The usual cross-compilation dance of despair.</p>

<p>Instead:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ time cross build --manifest-path ./crates/tauri-cli/Cargo.toml \
    --target riscv64gc-unknown-linux-gnu --profile release-size-optimized

   Compiling tauri-cli v2.9.6
    Finished `release-size-optimized` profile [optimized] target(s) in 4m 27s

real    4m27.235s
</code></pre>

</div>



<p>Four minutes and twenty-seven seconds.</p>

<h3>
  
  
  <strong>📊 The Numbers Don't Lie</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Method</th>
<th>Time</th>
<th>Relative Speed</th>
</tr>
</thead>
<tbody>
<tr>
<td>QEMU emulation (Docker buildx)</td>
<td>6+ hours (killed)</td>
<td>Baseline of pain</td>
</tr>
<tr>
<td>Native RISC-V (Banana Pi F3)</td>
<td>63 minutes</td>
<td>~6x faster than QEMU</td>
</tr>
<tr>
<td>Cross-rs on x64</td>
<td><strong>4m 27s</strong></td>
<td><strong>~90x faster than QEMU</strong></td>
</tr>
</tbody>
</table></div>

<p>I sat there looking at my terminal. Sixty-three minutes of native compilation. Four and a half minutes with cross. Fourteen times faster. On the same code. Same binary output.</p>

<h3>
  
  
  <strong>🧠 Why Cross Works Here</strong>
</h3>

<p>Here's what I got wrong in my previous article: I assumed Tauri CLI had the same WebKit dependencies as Tauri apps.</p>

<p>It doesn't.</p>

<p>Tauri CLI is a <strong>build tool</strong>. It orchestrates the build process, invokes cargo, bundles assets. It doesn't link against WebKit2GTK. That's what the <strong>app</strong> does at runtime. The CLI is just orchestration code.</p>

<p>No WebKit dependency means no sysroot nightmare. No sysroot nightmare means cross-compilation is trivial.</p>

<p>I spent an hour in my previous session explaining why cross-compilation was impossible. Turns out I was solving the wrong problem.</p>

<h2>
  
  
  The New Implementation
</h2>

<h3>
  
  
  <strong>🔧 Simpler Is Better</strong>
</h3>

<p>The updated workflow replaces self-hosted runners with cross:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">build</span><span class="pi">:</span>
  <span class="na">runs-on</span><span class="pi">:</span> <span class="s">${{ matrix.config.os }}</span>
  <span class="na">strategy</span><span class="pi">:</span>
    <span class="na">matrix</span><span class="pi">:</span>
      <span class="na">config</span><span class="pi">:</span>
        <span class="c1"># ... existing x64, ARM64, macOS, Windows targets ...</span>

        <span class="pi">-</span> <span class="na">os</span><span class="pi">:</span> <span class="s">ubuntu-22.04</span>
          <span class="na">rust_target</span><span class="pi">:</span> <span class="s">riscv64gc-unknown-linux-gnu</span>
          <span class="na">ext</span><span class="pi">:</span> <span class="s1">'</span><span class="s">'</span>
          <span class="na">args</span><span class="pi">:</span> <span class="s1">'</span><span class="s">'</span>
          <span class="na">cross</span><span class="pi">:</span> <span class="kc">true</span>

  <span class="na">steps</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s1">'</span><span class="s">Setup</span><span class="nv"> </span><span class="s">Rust'</span>
      <span class="na">if</span><span class="pi">:</span> <span class="s">${{ !matrix.config.cross }}</span>
      <span class="na">uses</span><span class="pi">:</span> <span class="s">dtolnay/rust-toolchain@stable</span>
      <span class="na">with</span><span class="pi">:</span>
        <span class="na">targets</span><span class="pi">:</span> <span class="s">${{ matrix.config.rust_target }}</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install cross</span>
      <span class="na">if</span><span class="pi">:</span> <span class="s">${{ matrix.config.cross }}</span>
      <span class="na">uses</span><span class="pi">:</span> <span class="s">taiki-e/install-action@v2</span>
      <span class="na">with</span><span class="pi">:</span>
        <span class="na">tool</span><span class="pi">:</span> <span class="s">cross@0.2.5</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build CLI (cross)</span>
      <span class="na">if</span><span class="pi">:</span> <span class="s">${{ matrix.config.cross }}</span>
      <span class="na">run</span><span class="pi">:</span> <span class="s">cross build --manifest-path ./crates/tauri-cli/Cargo.toml \</span>
           <span class="s">--target ${{ matrix.config.rust_target }} \</span>
           <span class="s">--profile release-size-optimized</span>
</code></pre>

</div>



<p>Changes from the self-hosted runner approach:</p>

<ol>
<li>
<strong>Runs on GitHub's ubuntu-22.04</strong>, not external hardware</li>
<li>
<strong>Uses <code>cross@0.2.5</code></strong> (pinned version for reproducibility)</li>
<li>
<strong>Skips unnecessary steps</strong> (Rust cache, Linux dependencies - cross handles these)</li>
<li>
<strong>Different artifact path</strong> (<code>target/{target}/</code> instead of <code>target/</code>)</li>
</ol>

<p>No self-hosted runners to maintain. No physical hardware to keep online. No network connectivity issues. Just a matrix entry that happens to use cross instead of cargo.</p>

<h2>
  
  
  The Humility Lesson
</h2>

<h3>
  
  
  <strong>🎓 You Don't Know What You Don't Know</strong>
</h3>

<p>Here's the uncomfortable truth: I had <strong>opinions</strong> about cross-compilation. Strong ones. Formed over a decade of wrestling with toolchains and sysroots. Those opinions were... not wrong, exactly. Cross-compiling WebKit apps <strong>is</strong> a nightmare. Setting up manual sysroots <strong>is</strong> fragile.</p>

<p>But cross-rs exists. Has existed since 2016. Nine years of development. Excellent RISC-V support. And I'd never encountered it because I was busy doing things the hard way.</p>

<p>This is the curse of experience: you learn patterns that work, and you stop exploring alternatives. "I know how to do this" becomes "I know how this is done" becomes "this is how it's done." Except sometimes a tool comes along that makes your carefully accumulated knowledge... obsolete? Not quite. But certainly less essential.</p>

<p>The gray hairs from ARM32 era taught me <strong>a</strong> way. Not <strong>the</strong> way.</p>

<h3>
  
  
  <strong>🔁 The Pattern Repeats</strong>
</h3>

<p>2013: "Cross-compiling for ARM32 requires a full sysroot setup."<br>
2016: cross-rs releases, nobody tells me.<br>
2025: "Cross-compiling for RISC-V is impossible because WebKit."<br>
Also 2025: "Have you considered cross?"</p>

<p>I should probably set up an RSS feed for Rust tooling. Or just... ask people before assuming I know the answer.</p>

<h2>
  
  
  What's Next
</h2>

<h3>
  
  
  <strong>🎯 PR Status</strong>
</h3>

<p>The PR is submitted: <a href="https://github.com/tauri-apps/tauri/pull/14685" rel="noopener noreferrer">https://github.com/tauri-apps/tauri/pull/14685</a></p>

<p>Current state:</p>

<ul>
<li>[x] Cross-rs implementation complete</li>
<li>[x] Build time: ~4 minutes (down from 63 minutes native, 6+ hours QEMU)</li>
<li>[x] CodeRabbit feedback addressed</li>
<li>[ ] Awaiting maintainer review</li>
</ul>

<p>If merged, every Tauri CLI release will include a RISC-V binary. Anyone on a RISC-V system can <code>cargo install tauri-cli</code> and get a pre-built binary in seconds instead of compiling 600+ crates.</p>

<h3>
  
  
  <strong>🚀 The Bigger Picture</strong>
</h3>

<p>This matters beyond Tauri. The RISC-V ecosystem is at that inflection point I keep mentioning. Hardware exists. Kernels boot. Distributions ship packages. But every missing binary is a barrier.</p>

<p>Framework 13 with DC-ROMA RISC-V mainboard? You'll want GUI apps. Banana Pi F3 running Armbian? You'll want to flash images without <code>dd</code>. Pine64 boards? Same story.</p>

<p>Pre-built binaries are the difference between "works out of the box" and "come back in 6 hours." Cross-rs makes pre-built binaries trivial to produce.</p>

<p>I'll probably be using cross for everything RISC-V from now on. Assuming the binary doesn't need runtime system libraries. For those cases... well, there's always the Banana Pi.</p>

<h2>
  
  
  Lessons Learned
</h2>

<h3>
  
  
  <strong>💡 Takeaways</strong>
</h3>

<ul>
<li><p><strong>Ask before assuming</strong>. A decade of experience doesn't mean you've seen everything. The maintainer's one-line suggestion saved hours of CI time.</p></li>
<li><p><strong>Cross-rs exists</strong>. For pure Rust binaries without system library dependencies, it's the obvious choice. Zero setup, fast builds, maintained Docker images.</p></li>
<li><p><strong>Know your dependencies</strong>. I assumed Tauri CLI had the same requirements as Tauri apps. It doesn't. CLI is a build tool, not a WebKit consumer.</p></li>
<li><p><strong>Self-hosted runners still have their place</strong>. For projects that <strong>do</strong> need system libraries (like the actual Armbian Imager), native hardware remains the answer. But for build tools and pure-Rust code? Cross wins.</p></li>
<li><p><strong>Pin your versions</strong>. <code>cross@0.2.5</code> is reproducible. <code>cross</code> is not. CI is not the place for floating versions.</p></li>
<li><p><strong>The hard way isn't always the right way</strong>. Sometimes the clever solution you've perfected over years gets replaced by a tool that just... does it better.</p></li>
</ul>

<p>I've been building for exotic architectures since ARM32 was exotic. Gray hairs and all. And today I learned something new.</p>

<p>That's not embarrassing. That's the job.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<a href="https://github.com/cross-rs/cross" rel="noopener noreferrer">cross-rs</a>: Zero setup cross compilation for Rust</li>
<li>
<a href="https://github.com/tauri-apps/tauri/pull/14685" rel="noopener noreferrer">Tauri PR #14685</a>: The RISC-V CLI support PR</li>
<li>
<a href="https://github.com/gounthar/tauri/releases/tag/tauri-cli-v2.9.6" rel="noopener noreferrer">Proof of concept release</a>: Working RISC-V binary (self-hosted runner build)</li>
<li>
<a href="https://tauri.app/" rel="noopener noreferrer">Tauri</a>: The framework this is all about</li>
<li>
<a href="https://www.armbian.com/download/?arch=riscv64" rel="noopener noreferrer">Armbian RISC-V</a>: The boards that need this software</li>
<li>
<a href="https://wiki.banana-pi.org/Banana_Pi_BPI-F3" rel="noopener noreferrer">Banana Pi BPI-F3</a>: The little board that could (in 63 minutes)</li>
</ul>

<p>The code is submitted. The binary works. And I have a new tool in my belt.</p>

<p>Twelve years late, but who's counting?</p>

