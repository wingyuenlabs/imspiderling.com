---
Title: Turning a 2013 Dell E6540 into a Dedicated TV Media Controller
Description: 
Author: Anand Rathnas
Date: 2026-01-11T21:31:46.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>TL;DR</strong>: I spent 12+ hours fighting Ubuntu 24.04's layered squashfs, hidden welcome wizards, and Chrome sandbox crashes to create a turnkey live USB for my old Dell E6540 laptop. Here's everything I learned so you don't have to.</p>

<h2>
  
  
  The Setup
</h2>

<p>I have a Dell Latitude E6540 (circa 2013) collecting dust. I also have a TV that needs a dedicated browser for streaming. The plan: create a custom Ubuntu live USB that boots directly into Chrome, no setup wizards, no login screens, just plug and play.</p>

<p>Hardware:</p>

<ul>
<li>Dell Latitude E6540 (i7, 8GB RAM)</li>
<li>
<a href="https://a.co/d/2OPBlTy" rel="noopener noreferrer">Logitech K400 Plus Wireless Keyboard</a> - perfect for couch browsing</li>
<li>Any USB drive (8GB+)</li>
</ul>

<h2>
  
  
  The Journey (aka What Went Wrong)
</h2>

<h3>
  
  
  1. Ubuntu 24.04's Layered Squashfs
</h3>

<p><strong>The Problem</strong>: Ubuntu 24.04 doesn't use a single <code>filesystem.squashfs</code> anymore. It uses a layered system:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>casper/
├── minimal.squashfs           # Base layer
├── minimal.standard.squashfs  # Standard additions
└── minimal.standard.live.squashfs  # Live environment customizations
</code></pre>

</div>



<p><strong>What I Tried</strong>: Merging all layers into one <code>filesystem.squashfs</code>.</p>

<p><strong>What Happened</strong>: Boot failure - "File system layers are missing"</p>

<p><strong>The Fix</strong>: Keep ALL layers intact. Only modify the specific layer containing what you need to change. Casper expects the layered structure.</p>

<p><strong>Reference</strong>: <a href="https://manpages.ubuntu.com/manpages/noble/man7/casper.7.html" rel="noopener noreferrer">Ubuntu Casper Manual</a></p>

<h3>
  
  
  2. Finding gnome-initial-setup
</h3>

<p><strong>The Problem</strong>: The GNOME initial setup wizard kept appearing on boot. I needed to remove it.</p>

<p><strong>What I Tried</strong>: Searching and removing from the top layer only.</p>

<p><strong>What Happened</strong>: Still appeared. The binary wasn't in the top layer.</p>

<p><strong>The Fix</strong>: Search ALL layers using <code>unsquashfs -l</code> (list mode - fast!) instead of extracting:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">for </span>layer <span class="k">in </span>minimal minimal.standard minimal.standard.live<span class="p">;</span> <span class="k">do
  </span><span class="nv">FOUND</span><span class="o">=</span><span class="si">$(</span>unsquashfs <span class="nt">-l</span> <span class="s2">"</span><span class="nv">$CASPER_DIR</span><span class="s2">/</span><span class="k">${</span><span class="nv">layer</span><span class="k">}</span><span class="s2">.squashfs"</span> 2&gt;/dev/null | <span class="se">\</span>
    <span class="nb">grep</span> <span class="nt">-E</span> <span class="s2">"gnome-initial-setup"</span> <span class="o">||</span> <span class="nb">true</span><span class="si">)</span>
  <span class="k">if</span> <span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$FOUND</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"Found in </span><span class="k">${</span><span class="nv">layer</span><span class="k">}</span><span class="s2">.squashfs!"</span>
  <span class="k">fi
done</span>
</code></pre>

</div>



<p><strong>Lesson</strong>: <code>gnome-initial-setup</code> lives in <code>minimal.squashfs</code> (the base layer).</p>

<h3>
  
  
  3. The "Welcome to Ubuntu" Imposter
</h3>

<p><strong>The Problem</strong>: After removing gnome-initial-setup, a "Welcome to Ubuntu" wizard STILL appeared.</p>

<p><strong>What I Tried</strong>: Removing more gnome-initial-setup files, masking systemd services, creating done files.</p>

<p><strong>What Happened</strong>: Still appeared. Different icon, different behavior.</p>

<p><strong>The Fix</strong>: It's NOT gnome-initial-setup. It's <code>ubuntu-desktop-bootstrap</code> - a completely different snap package!<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Search for the REAL culprit</span>
unsquashfs <span class="nt">-l</span> layer.squashfs | <span class="nb">grep</span> <span class="nt">-E</span> <span class="s2">"ubuntu-desktop-bootstrap|desktop-bootstrap"</span>
</code></pre>

</div>



<p><strong>Lesson</strong>: Don't assume. Check the actual desktop shortcut or process name.</p>

<h3>
  
  
  4. Auto-Login Configuration
</h3>

<p><strong>The Problem</strong>: System landed on login screen instead of auto-logging in.</p>

<p><strong>The Fix</strong>: Configure GDM properly in the squashfs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$SQUASH_DIR</span><span class="s2">/etc/gdm3"</span>
<span class="nb">cat</span> <span class="o">&gt;</span> <span class="s2">"</span><span class="nv">$SQUASH_DIR</span><span class="s2">/etc/gdm3/custom.conf"</span> <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
[daemon]
InitialSetupEnable=false
AutomaticLoginEnable=true
AutomaticLogin=ubuntu
</span><span class="no">EOF
</span></code></pre>

</div>



<h3>
  
  
  5. Chrome Crashes on Live USB
</h3>

<p><strong>The Problem</strong>: Chrome installed fine but crashed immediately on launch. Error reporter appeared.</p>

<p><strong>What I Tried</strong>: Adding <code>--disable-gpu</code> (wrong assumption about nomodeset).</p>

<p><strong>The Real Problem</strong>: Chrome's sandbox doesn't play nice with overlayfs (used by live USB environments).</p>

<p><strong>The Fix</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>google-chrome-stable <span class="se">\</span>
  <span class="nt">--no-first-run</span> <span class="se">\</span>
  <span class="nt">--no-default-browser-check</span> <span class="se">\</span>
  <span class="nt">--disable-session-crashed-bubble</span> <span class="se">\</span>
  <span class="nt">--no-sandbox</span> <span class="se">\</span>
  <span class="s2">"https://your-url-here"</span>
</code></pre>

</div>



<p><strong>Note</strong>: <code>--no-sandbox</code> reduces security but is necessary for live USB environments.</p>

<h3>
  
  
  6. <code>pactl: command not found</code>
</h3>

<p><strong>The Problem</strong>: Script failed with <code>pactl: command not found</code> on Ubuntu 24.04 live environment.</p>

<p><strong>Why</strong>: Ubuntu 24.04 uses PipeWire, and <code>pactl</code> (PulseAudio control) may not be available in the minimal live environment.</p>

<p><strong>The Fix</strong>: Check before using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">if </span><span class="nb">command</span> <span class="nt">-v</span> pactl &amp;&gt;/dev/null<span class="p">;</span> <span class="k">then
  </span><span class="nv">HDMI_SINK</span><span class="o">=</span><span class="si">$(</span>pactl list short sinks | <span class="nb">grep</span> <span class="nt">-i</span> hdmi | <span class="nb">head</span> <span class="nt">-1</span> | <span class="nb">awk</span> <span class="s1">'{print $2}'</span><span class="si">)</span>
  <span class="o">[</span> <span class="nt">-n</span> <span class="s2">"</span><span class="nv">$HDMI_SINK</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> pactl set-default-sink <span class="s2">"</span><span class="nv">$HDMI_SINK</span><span class="s2">"</span>
<span class="k">else
  </span><span class="nb">echo</span> <span class="s2">"Warning: pactl not available"</span>
<span class="k">fi</span>
</code></pre>

</div>



<h3>
  
  
  7. Slow USB Write on macOS
</h3>

<p><strong>The Problem</strong>: Writing 6GB ISO took 1085 seconds (~18 minutes).</p>

<p><strong>The Fix</strong>: Use raw device <code>/dev/rdisk</code> instead of <code>/dev/disk</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Slow (~18 min for 6GB)</span>
<span class="nb">sudo dd </span><span class="k">if</span><span class="o">=</span>ubuntu.iso <span class="nv">of</span><span class="o">=</span>/dev/disk4 <span class="nv">bs</span><span class="o">=</span>4m

<span class="c"># Fast (~2 min for 6GB) - use rdisk</span>
<span class="nv">RAW_DEVICE</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"/dev/disk4"</span> | <span class="nb">sed</span> <span class="s1">'s|/dev/disk|/dev/rdisk|'</span><span class="si">)</span>
<span class="nb">sudo dd </span><span class="k">if</span><span class="o">=</span>ubuntu.iso <span class="nv">of</span><span class="o">=</span><span class="nv">$RAW_DEVICE</span> <span class="nv">bs</span><span class="o">=</span>4m <span class="nv">status</span><span class="o">=</span>progress
</code></pre>

</div>



<p><strong>Why</strong>: <code>/dev/rdisk</code> bypasses macOS's buffer cache, giving ~10x faster writes.</p>

<h3>
  
  
  8. EFI Partition Extraction
</h3>

<p><strong>The Problem</strong>: Custom ISO wouldn't boot on UEFI systems.</p>

<p><strong>The Fix</strong>: Extract and preserve the original EFI partition:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Get EFI partition info from original ISO</span>
<span class="nv">EFI_INFO</span><span class="o">=</span><span class="si">$(</span>xorriso <span class="nt">-indev</span> <span class="s2">"</span><span class="nv">$ISO_IN</span><span class="s2">"</span> <span class="nt">-report_el_torito</span> as_mkisofs 2&gt;&amp;1 | <span class="se">\</span>
  <span class="nb">grep</span> <span class="nt">-A1</span> <span class="s2">"append_partition 2"</span><span class="si">)</span>
<span class="nv">INTERVAL</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$EFI_INFO</span><span class="s2">"</span> | <span class="nb">grep</span> <span class="nt">-oP</span> <span class="s1">'\d+d-\d+d'</span> | <span class="nb">head</span> <span class="nt">-1</span><span class="si">)</span>

<span class="c"># Extract it</span>
<span class="nv">START_SECTOR</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$INTERVAL</span><span class="s2">"</span> | <span class="nb">cut</span> <span class="nt">-d</span><span class="s1">'-'</span> <span class="nt">-f1</span> | <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">'d'</span><span class="si">)</span>
<span class="nv">END_SECTOR</span><span class="o">=</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$INTERVAL</span><span class="s2">"</span> | <span class="nb">cut</span> <span class="nt">-d</span><span class="s1">'-'</span> <span class="nt">-f2</span> | <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">'d'</span><span class="si">)</span>
<span class="nv">COUNT</span><span class="o">=</span><span class="k">$((</span>END_SECTOR <span class="o">-</span> START_SECTOR <span class="o">+</span> <span class="m">1</span><span class="k">))</span>
<span class="nb">dd </span><span class="k">if</span><span class="o">=</span><span class="s2">"</span><span class="nv">$ISO_IN</span><span class="s2">"</span> <span class="nv">of</span><span class="o">=</span><span class="s2">"</span><span class="nv">$EFI_IMG</span><span class="s2">"</span> <span class="nv">bs</span><span class="o">=</span>512 <span class="nv">skip</span><span class="o">=</span><span class="s2">"</span><span class="nv">$START_SECTOR</span><span class="s2">"</span> <span class="nv">count</span><span class="o">=</span><span class="s2">"</span><span class="nv">$COUNT</span><span class="s2">"</span>
</code></pre>

</div>



<h2>
  
  
  The Final Solution
</h2>

<p>I built a Docker-based tool that:</p>

<ol>
<li>Downloads Ubuntu 24.04 desktop ISO</li>
<li>Extracts and modifies the layered squashfs</li>
<li>Removes ALL welcome wizards (gnome-initial-setup, gnome-tour, ubuntu-desktop-bootstrap)</li>
<li>Configures auto-login</li>
<li>Pre-installs wallpapers, Chrome policies, dark mode</li>
<li>Rebuilds a bootable hybrid ISO (BIOS + UEFI)</li>
<li>Writes to USB with one command
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./make.sh  <span class="c"># Does everything: clean, build, write to USB</span>
</code></pre>

</div>



<h2>
  
  
  Use It Yourself
</h2>

<p>If you have a Dell E6540 (or similar older laptop) and want a dedicated TV/media controller:</p>

<p><strong>GitHub</strong>: <a href="https://github.com/tv6540/cubic2" rel="noopener noreferrer">github.com/tv6540/cubic2</a></p>

<p><strong>Requirements</strong>:</p>

<ul>
<li>Docker</li>
<li>macOS or Linux</li>
<li>USB drive (8GB+)</li>
<li>
<a href="https://a.co/d/2OPBlTy" rel="noopener noreferrer">Logitech K400 Plus</a> (recommended for couch use)</li>
</ul>

<p><strong>Usage</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/tv6540/cubic2
<span class="nb">cd </span>cubic2
./make.sh
</code></pre>

</div>



<p>The script will:</p>

<ol>
<li>Prompt for sudo (kept alive during build)</li>
<li>Show USB device picker</li>
<li>Confirm before erasing</li>
<li>Download Ubuntu ISO (cached for future builds)</li>
<li>Build custom ISO in Docker</li>
<li>Write to USB</li>
</ol>

<p>Boot the USB, and the setup script runs automatically to configure display, install Chrome, and you're ready to stream.</p>

<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li>
<strong>Ubuntu 24.04 uses layered squashfs</strong> - don't merge them, modify in place</li>
<li>
<strong><code>unsquashfs -l</code> is your friend</strong> - list files without extracting (fast!)</li>
<li>
<strong>ubuntu-desktop-bootstrap != gnome-initial-setup</strong> - check process names</li>
<li>
<strong>Chrome needs <code>--no-sandbox</code> on live USB</strong> - overlayfs breaks the sandbox</li>
<li>
<strong>Use <code>/dev/rdisk</code> on macOS</strong> - 10x faster USB writes</li>
<li>
<strong>Get all prompts upfront</strong> - nobody wants to wait 20 min then type "yes"</li>
</ol>

<h2>
  
  
  Hardware Recommendation
</h2>

<p>The <a href="https://a.co/d/2OPBlTy" rel="noopener noreferrer">Logitech K400 Plus</a> is perfect for this setup:</p>

<ul>
<li>Wireless with tiny USB receiver</li>
<li>Built-in trackpad</li>
<li>Media keys</li>
<li>Long battery life</li>
<li>Compact for couch use</li>
</ul>




<p><em>Built with mass frustration, mass trial-and-error, and mass caffeine.</em></p>

