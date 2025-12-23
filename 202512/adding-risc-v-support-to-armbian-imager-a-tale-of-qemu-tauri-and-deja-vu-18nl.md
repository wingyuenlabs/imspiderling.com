---
Title: Adding RISC-V Support to Armbian Imager: A Tale of QEMU, Tauri, and Deja Vu
Description: 
Author: Bruno Verachten
Date: 2025-12-23T21:58:25.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Photo by <a href="https://unsplash.com/@catauggie" rel="noopener noreferrer">SnapSaga</a> on Unsplash</em></p>

<h2>
  
  
  The Setup
</h2>

<p>Armbian Imager is a Tauri 2 application: React frontend, Rust backend, builds for Linux (x64, ARM64), macOS (both architectures), and Windows (both architectures). A proper multi-platform desktop app that actually works, which is rarer than you'd think.</p>

<p>The build workflow already handled six platform combinations through GitHub Actions. Adding a seventh (Linux RISC-V 64-bit) seemed straightforward. After all, I'd done this dance before with ARM32 back in the dark ages (2013-2014, when docker-compose on Raspberry Pi was considered experimental black magic).</p>

<p>Famous last words.</p>

<h3>
  
  
  The Murphy's Law Setup
</h3>

<p>Here's the thing: whenever you think "this should be straightforward," the universe takes that as a personal challenge. I should've known better. The gray hairs didn't appear from successful, uneventful builds.</p>

<h2>
  
  
  Why RISC-V, Why Now
</h2>

<p>Armbian supports RISC-V boards. The Banana Pi F3 runs Armbian. Various Pine64 and StarFive boards run Armbian. The Framework 13 laptop has a RISC-V mainboard option (DC-Roma, because apparently laptop mainboards are modular now, what a time to be alive). The ecosystem is growing.</p>

<p>But if you're on a RISC-V system and want to flash an Armbian image to an SD card, you currently need to use <code>dd</code> like it's 1995. The Imager app doesn't have a RISC-V build.</p>

<p>I figured I'd fix that.</p>

<p>Sounds appealing or strange enough to get you intrigued?</p>

<h2>
  
  
  The Research Phase
</h2>

<p>First step: figure out what the existing build workflow does. The <code>.github/workflows/build.yml</code> file tells the story:</p>

<ul>
<li>Linux x64 builds on <code>ubuntu-24.04</code> runners</li>
<li>Linux ARM64 builds on <code>ubuntu-24.04-arm</code> runners (GitHub has native ARM runners now, progress!)</li>
<li>macOS and Windows have their own native runners</li>
</ul>

<p>For RISC-V, GitHub doesn't offer native runners yet. There's <a href="https://cloud-v.co/github-riscv-runner" rel="noopener noreferrer">Cloud-V</a> which provides RISC-V GitHub runners, but Armbian's workflow isn't set up for external runners. For this contribution, emulation was the path of least resistance.</p>

<p>Docker + QEMU it is. (Spoiler alert: this is where things get interesting.)</p>

<h3>
  
  
  The WebKit2GTK Archaeology
</h3>

<p>Tauri apps on Linux need WebKit2GTK. This is the system webview that renders the UI, basically the browser engine that makes your Rust backend look pretty. On x64 and ARM64, it's readily available in Debian bookworm and trixie.</p>

<p>On RISC-V? I checked the <a href="https://tracker.debian.org/pkg/webkit2gtk" rel="noopener noreferrer">Debian package tracker</a>, because I'm a glutton for disappointment.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>webkit2gtk in bookworm: not available for riscv64
webkit2gtk in trixie: available for riscv64
</code></pre>

</div>



<p>Good news: Debian trixie became stable in 2025, and it has the packages we need. The timing worked out: RISC-V users on the current stable release get WebKit2GTK out of the box.</p>

<h3>
  
  
  The NodeSource Curveball
</h3>

<p>The frontend build needs Node.js. The standard approach in CI is to use NodeSource's distribution:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://deb.nodesource.com/setup_20.x | bash -
apt-get <span class="nb">install</span> <span class="nt">-y</span> nodejs
</code></pre>

</div>



<p>I checked NodeSource's supported architectures: amd64, arm64, armhf. No riscv64.</p>

<p>Of course not. Why would there be?</p>

<p>Here's where things get simple (yes, really): use Debian's native nodejs package instead. It's version 18 instead of 20, but that's close enough for a Vite build. Sometimes the boring solution is the right solution.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>apt-get <span class="nb">install</span> <span class="nt">-y</span> nodejs npm
</code></pre>

</div>



<p>(Side note: if you need the latest Node.js LTS on RISC-V, I maintain <a href="https://github.com/gounthar/unofficial-builds/releases" rel="noopener noreferrer">unofficial builds</a> with an APT repo via GitHub Pages. We've got 24.12.0 ready to go. But for this build, Debian's package was sufficient.)</p>

<p>Two problems identified, two solutions found. Look at me being all productive and efficient. This never happens. I should've been more suspicious.</p>

<h2>
  
  
  The Implementation
</h2>

<p>I added a new job to the GitHub workflow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">build-linux-riscv64</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s">build-linux (riscv64gc-unknown-linux-gnu)</span>
  <span class="na">needs</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">create-release</span><span class="pi">]</span>
  <span class="na">if</span><span class="pi">:</span> <span class="s">${{ github.event_name == 'push' || inputs.build_linux_riscv64 }}</span>
  <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-24.04</span>
  <span class="na">steps</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up QEMU for RISC-V emulation</span>
      <span class="na">uses</span><span class="pi">:</span> <span class="s">docker/setup-qemu-action@v3</span>
      <span class="na">with</span><span class="pi">:</span>
        <span class="na">platforms</span><span class="pi">:</span> <span class="s">riscv64</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Set up Docker Buildx</span>
      <span class="na">uses</span><span class="pi">:</span> <span class="s">docker/setup-buildx-action@v3</span>

    <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Build in RISC-V container (Debian trixie)</span>
      <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
        <span class="s">docker run --rm --platform linux/riscv64 \</span>
          <span class="s">-v "$(pwd)":/app \</span>
          <span class="s">-w /app \</span>
          <span class="s">riscv64/debian:trixie \</span>
          <span class="s">bash -c '</span>
            <span class="s"># Install build dependencies</span>
            <span class="s">apt-get update</span>
            <span class="s">apt-get install -y \</span>
              <span class="s">curl build-essential pkg-config \</span>
              <span class="s">libwebkit2gtk-4.1-dev \</span>
              <span class="s">libayatana-appindicator3-dev \</span>
              <span class="s">librsvg2-dev patchelf libssl-dev libgtk-3-dev \</span>
              <span class="s">squashfs-tools xdg-utils file</span>

            <span class="s"># Node.js from Debian (NodeSource lacks RISC-V)</span>
            <span class="s">apt-get install -y nodejs npm</span>

            <span class="s"># Install Rust</span>
            <span class="s">curl --proto "=https" --tlsv1.2 -sSf https://sh.rustup.rs | \</span>
              <span class="s">sh -s -- -y</span>
            <span class="s">source "$HOME/.cargo/env"</span>

            <span class="s"># Build frontend</span>
            <span class="s">npm ci</span>
            <span class="s">npm run build</span>

            <span class="s"># Install Tauri CLI and build</span>
            <span class="s">cargo install tauri-cli --version "^2" --locked</span>
            <span class="s">cargo tauri build --bundles deb</span>
          <span class="s">'</span>
</code></pre>

</div>



<p>The key differences from the x64/ARM64 builds:</p>

<ol>
<li>Uses <code>riscv64/debian:trixie</code> instead of Ubuntu (because we need those WebKit packages)</li>
<li>Node.js comes from Debian packages, not NodeSource (because NodeSource said "nope")</li>
<li>Runs through QEMU user-mode emulation (this will be important later)</li>
<li>Only builds <code>.deb</code> packages (no AppImage; that's a story for another day, involving AppImage's aversion to exotic architectures)</li>
</ol>

<p>I also created a standalone script <code>scripts/build-riscv64.sh</code> for local builds, and added <code>--riscv64</code> to the <code>build-all.sh</code> orchestrator, because I like my tooling to be consistent.</p>

<p>Four commits later:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>aae7628 feat: Add RISC-V 64-bit build support
105f77f fix: Use Debian nodejs package for RISC-V builds
22499b1 feat: Add pre-built Docker image support for faster RISC-V builds
0eb2992 fix: Use Debian nodejs in build-all.sh for RISC-V
</code></pre>

</div>



<p>Time to test. What could possibly go wrong?</p>

<h2>
  
  
  The Wall
</h2>

<p>I kicked off a build. Docker pulled the RISC-V Debian image. QEMU started emulating. The apt packages installed. Rust downloaded. So far, so good.</p>

<p>Then:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cargo install tauri-cli --version "^2" --locked
</code></pre>

</div>



<p>And we wait.</p>

<p>And wait.</p>

<p>And wait some more.</p>

<h3>
  
  
  The Great Compilation Watch
</h3>

<p>The tauri-cli crate has over 600 dependencies. Six. Hundred. Each one needs to compile. Under QEMU user-mode emulation, every single CPU instruction goes through a translation layer. A Rust build that takes 2 minutes on native hardware takes... well, let's just say it takes a while.</p>

<p>I went to make coffee. Came back. Still compiling.</p>

<p>I went to lunch. Came back. Still compiling.</p>

<p>I started questioning my life choices. The build was still compiling.</p>

<p>This is the ARM32 era all over again, and I'm having flashbacks.</p>

<h2>
  
  
  Flashback: 2013-2014
</h2>

<h3>
  
  
  The ARM32 PTSD
</h3>

<p>Picture this: around 2013, I was trying to get docker-compose working on ARM32. And RethinkDB. And everything else needed to run openSTF (Open Smartphone Test Farm) on Raspberry Pi hardware, because apparently I enjoy suffering.</p>

<p>Nothing worked. Every dependency was a new adventure in "why doesn't this architecture exist in their CI matrix?" "Just compile it from source" meant leaving your Pi running overnight and hoping it didn't thermal throttle itself into early retirement. Cross-compilation setups were fragile. One wrong symlink and you'd spend three hours debugging why <code>ld</code> couldn't find <code>libc.so.6</code>. Pre-built binaries were as rare as sensible variable names in legacy code.</p>

<p>We eventually got there. ARM support improved. Docker got multi-arch images. GitHub added ARM runners. The ecosystem matured. The gray hairs appeared.</p>

<p>Here's the thing: RISC-V is at that same inflection point now. The hardware exists. The kernels boot. The distributions have packages. But the tooling ecosystem hasn't caught up yet.</p>

<p>And I'm apparently volunteering to help it catch up. Because I learn nothing from experience.</p>

<h2>
  
  
  The Math
</h2>

<p>Let's be concrete about the problem, because misery loves documentation.</p>

<p>On a native x64 machine with cached dependencies, <code>cargo install tauri-cli</code> takes about 2 minutes. Fast enough to grab a coffee, check Slack, come back to a finished build.</p>

<p>Under QEMU user-mode emulation, that same operation takes... I didn't let it finish. After 3 hours, I killed the build because I value my sanity. Extrapolating from progress (and some very pessimistic napkin math), a complete build would take 6-8 hours.</p>

<p>For CI/CD, this is unusable. GitHub Actions has a 6-hour timeout per job. Even if it finished, waiting that long for every release is absurd. Imagine telling your team "yeah, the release will be ready in 8 hours, assuming nothing goes wrong." (Spoiler: something always goes wrong.)</p>

<h3>
  
  
  The Pre-built Image Strategy
</h3>

<p>My first optimization: build a Docker image with tauri-cli pre-installed. Suffer once, benefit forever (or until Tauri releases a new version, whichever comes first).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="s"> --platform=linux/riscv64 riscv64/debian:trixie</span>

<span class="c"># Install all build dependencies</span>
<span class="k">RUN </span>apt-get update <span class="o">&amp;&amp;</span> apt-get <span class="nb">install</span> <span class="nt">-y</span> <span class="se">\
</span>    curl build-essential pkg-config <span class="se">\
</span>    libwebkit2gtk-4.1-dev libayatana-appindicator3-dev <span class="se">\
</span>    librsvg2-dev patchelf libssl-dev libgtk-3-dev <span class="se">\
</span>    squashfs-tools xdg-utils file nodejs npm <span class="se">\
</span>    <span class="o">&amp;&amp;</span> <span class="nb">rm</span> <span class="nt">-rf</span> /var/lib/apt/lists/<span class="k">*</span>

<span class="c"># Install Rust</span>
<span class="k">RUN </span>curl <span class="nt">--proto</span> <span class="s1">'=https'</span> <span class="nt">--tlsv1</span>.2 <span class="nt">-sSf</span> https://sh.rustup.rs | <span class="se">\
</span>    sh <span class="nt">-s</span> <span class="nt">--</span> <span class="nt">-y</span>
<span class="k">ENV</span><span class="s"> PATH="/root/.cargo/bin:${PATH}"</span>

<span class="c"># Pre-install tauri-cli (this is the slow part)</span>
<span class="k">RUN </span>cargo <span class="nb">install </span>tauri-cli <span class="nt">--version</span> <span class="s2">"^2"</span> <span class="nt">--locked</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">CMD</span><span class="s"> ["bash"]</span>
</code></pre>

</div>



<p>Build this image once (accepting the 6+ hour wait like penance for your architectural choices), push it to a registry, then subsequent builds only need to compile the actual application, maybe 10-20 minutes under emulation. Still slow, but manageable.</p>

<blockquote>
<p><strong>Tip:</strong> If you're building locally, run <code>./scripts/build-riscv64.sh --build-image</code> once to create the pre-built image. Subsequent builds will skip tauri-cli compilation entirely. Your future self will thank you.</p>
</blockquote>

<p>This works for local development. For CI/CD, it requires maintaining a container registry with RISC-V images, rebuilding whenever Tauri releases a new version. Manageable, but inelegant. And I have opinions about inelegant solutions.</p>

<h3>
  
  
  The Cross-Compilation Dream (That Didn't Happen)
</h3>

<p>The proper solution is cross-compilation, right? Build on a fast x64 machine, target riscv64. No emulation overhead. Just pure, unadulterated compilation speed.</p>

<p>The problem: WebKit2GTK. Again. That dependency is following me around like a particularly persistent technical debt.</p>

<p>Tauri links against the system WebKit. To cross-compile, you need a RISC-V sysroot with all the WebKit headers and libraries. Setting that up is... non-trivial. You're essentially recreating a Debian trixie root filesystem for a different architecture, and WebKit pulls in half of GNOME as dependencies. Have you <em>seen</em> the dependency tree for a full GNOME stack? It's like a fractal of "why does this need that."</p>

<p>I spent an hour exploring this path. It's doable, but fragile. One apt update breaks your sysroot. Version mismatches between the cross-compilation toolchain and the target libraries. Symlink hell. Not worth the maintenance burden for a community contribution.</p>

<p>Sometimes you have to know when to fold.</p>

<h2>
  
  
  Going Upstream
</h2>

<h3>
  
  
  The Aha Moment
</h3>

<p>Here's where my thinking shifted.</p>

<p>The bottleneck isn't Armbian Imager. It isn't QEMU. It isn't Docker. The bottleneck is <code>cargo install tauri-cli</code>: compiling 600+ crates from source because there's no pre-built RISC-V binary.</p>

<p>Wait, what? Why isn't there a pre-built binary?</p>

<p>Tauri provides pre-built binaries for:</p>

<ul>
<li>Linux x64</li>
<li>Linux ARM64</li>
<li>macOS x64</li>
<li>macOS ARM64</li>
<li>Windows x64</li>
</ul>

<p>No RISC-V.</p>

<p>If Tauri's release workflow included RISC-V binaries, the entire Tauri ecosystem would benefit. Every single project trying to build for RISC-V would save those 6+ hours. Every developer who comes after me wouldn't have to rediscover this problem. Every CI pipeline wouldn't timeout waiting for Rust to compile half the internet.</p>

<p>So that's the plan. Fork tauri-cli, add RISC-V to the release matrix, get a PR upstream. Contribute to the root of the problem instead of working around it at the leaf.</p>

<p>This isn't about maintaining a fork long-term (I've already got enough side projects to feel guilty about). It's about making the whole ecosystem better. Rising tide lifts all boats, and all that.</p>

<h3>
  
  
  The Contribution Plan
</h3>

<p>Here's the thing: I've been on both sides of this. I've been the maintainer getting PRs for exotic architectures. I've been the contributor trying to convince maintainers that yes, people actually use this platform.</p>

<p>The Tauri team has been responsive to architecture additions before. They added ARM64 support. They care about multi-platform support; it's literally in their value proposition. Adding RISC-V to their CI matrix is a natural extension of what they're already doing.</p>

<p>The work isn't even that complicated:</p>

<ol>
<li>Add RISC-V to their GitHub Actions build matrix</li>
<li>Set up QEMU for the build (they already do this for other architectures)</li>
<li>Upload the resulting binary to their releases</li>
</ol>

<p>The hard part is the 6-hour compile time under QEMU, but there's an alternative: register a native RISC-V runner for the project. <a href="https://cloud-v.co/github-riscv-runner" rel="noopener noreferrer">Cloud-V</a> offers exactly that. Native compilation, no emulation penalty. What takes 6 hours under QEMU would take minutes on real hardware.</p>

<p>Either way, once it's in their CI, it stays current. Every Tauri version automatically gets RISC-V binaries.</p>

<p>I just need to prove it works, submit a clean PR, and make a compelling case. How hard could it be?</p>

<p>(I really need to stop asking that question.)</p>

<h2>
  
  
  Current Status
</h2>

<p>The RISC-V build support is implemented and committed on the <code>feature/riscv64-support</code> branch:</p>

<ul>
<li>GitHub workflow job: ready, but will timeout in CI without optimizations (those pesky 6-hour limits)</li>
<li>Standalone build script: works locally with pre-built Docker image (tested, confirmed, actually functions)</li>
<li>Integration with build-all.sh: complete (because consistency matters)</li>
</ul>

<p>What's blocked:</p>

<ul>
<li>CI/CD builds: need either pre-built tauri-cli binaries or a hosted RISC-V builder image</li>
<li>Upstream contribution: need to explore Tauri's build infrastructure and submit that PR</li>
</ul>

<h2>
  
  
  What's Next
</h2>

<h3>
  
  
  The Roadmap
</h3>

<ol>
<li><p><strong>Short term</strong>: Push the branch, document the limitation honestly (because documentation that lies helps nobody), offer the pre-built Docker image approach as a workaround for anyone who wants RISC-V builds today.</p></li>
<li><p><strong>Medium term</strong>: Fork tauri-cli, experiment with adding RISC-V to their release workflow, prepare a PR that's actually mergeable (not just "hey I hacked this together and it technically works").</p></li>
<li><p><strong>Long term</strong>: Native RISC-V CI runners will eventually exist. GitHub, GitLab, someone will offer them. And when that happens, this entire problem disappears. But we're not there yet, and people want to use RISC-V now, so here we are.</p></li>
</ol>

<h2>
  
  
  Lessons Learned
</h2>

<h3>
  
  
  Takeaways and Tips for Future Archaeologists
</h3>

<ul>
<li><p><strong>Debian trixie is your friend for RISC-V</strong> - The testing distribution has packages that stable lacks. For bleeding-edge architectures, accept some instability in exchange for working software. It's a fair trade.</p></li>
<li><p><strong>NodeSource doesn't support everything</strong> - When the fancy installer doesn't work, fall back to distribution packages. They're usually good enough, and "good enough" ships.</p></li>
<li><p><strong>Emulation is slow, but it works</strong> - QEMU user-mode emulation lets you run foreign binaries on any host. The speed penalty is brutal for compilation, but acceptable for runtime testing. Know which problem you're solving.</p></li>
<li><p><strong>Sometimes the fix is upstream</strong> - When you hit a wall that affects the whole ecosystem, consider fixing the source rather than building elaborate workarounds. Be the change you want to see in the dependency tree.</p></li>
<li><p><strong>This has all happened before</strong> - ARM32 in 2013. ARM64 in 2016. RISC-V in 2025. The pattern repeats: hardware arrives, software catches up, early adopters suffer, then it all becomes normal. We're in the "early adopters suffer" phase.</p></li>
<li><p><strong>Pre-built images are your friend</strong> - One slow build beats a thousand slow builds. Cache aggressively, share generously, document thoroughly.</p></li>
<li><p><strong>Don't cross-compile WebKit unless you hate yourself</strong> - Some battles aren't worth fighting. Some dependency trees are better left untraversed.</p></li>
</ul>

<p>I've been here before. The gray hairs prove it. But that's also why I know it gets better. The pain is temporary. The infrastructure improvements are permanent. And somewhere, someday, a developer will <code>cargo install tauri-cli</code> on their RISC-V board and it'll just work in 30 seconds, and they'll never know about the 6-hour compile times we endured to make that possible.</p>

<p>That's the dream, anyway.</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li>
<a href="https://tauri.app/" rel="noopener noreferrer">Tauri documentation</a>: Official Tauri 2 docs (actually well-written, surprisingly)</li>
<li>
<a href="https://wiki.debian.org/RISC-V" rel="noopener noreferrer">Debian RISC-V wiki</a>: Porting status and bootstrap info</li>
<li>
<a href="https://www.qemu.org/docs/master/user/main.html" rel="noopener noreferrer">QEMU user-mode emulation</a>: How binfmt_misc magic works (read this if you want to understand the sorcery)</li>
<li>
<a href="https://www.armbian.com/download/?arch=riscv64" rel="noopener noreferrer">Armbian RISC-V boards</a>: Supported RISC-V hardware</li>
<li>
<a href="https://wiki.banana-pi.org/Banana_Pi_BPI-F3" rel="noopener noreferrer">Banana Pi BPI-F3</a>: SpacemiT K1 octa-core RISC-V board (actual hardware you can buy today)</li>
</ul>

<p>The code works. The build process works. It's just... slow. For now.</p>

<p>But we've been here before, and we know how this story ends. The ecosystem catches up. The tooling improves. The compile times get reasonable. The gray hairs multiply.</p>

<p>Let's make it happen.</p>

