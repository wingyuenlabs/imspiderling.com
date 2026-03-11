---
Title: swift 6, screencapturekit, and why my app worked in xcode but not as a .app
Description: 
Author: KimSejun
Date: 2026-03-11T22:06:00.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Swift 6, ScreenCaptureKit, and why my app worked in Xcode but not as a .app
</h1>

<p>I created this post for the purposes of entering the Gemini Live Agent Challenge. I'm building <a href="https://github.com/Two-Weeks-Team/vibeCat" rel="noopener noreferrer">VibeCat</a>, a desktop AI companion that watches your screen and talks to you.</p>

<p>The backend was done. Nine agents, WebSocket proxy, Gemini Live API integration — all working. Time to build the macOS client. Swift 6. SwiftUI. ScreenCaptureKit. How hard could it be?</p>

<p>Three days. Three days of things silently not working, with zero error messages.</p>

<h2>
  
  
  the screen capture that captured nothing
</h2>

<p>VibeCat needs to see your screen to be useful. The VisionAgent on the backend analyzes screenshots to detect errors, notice you're stuck, or see tests pass. So the client needs ScreenCaptureKit.</p>

<p>The code itself is clean:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@MainActor</span>
<span class="kd">final</span> <span class="kd">class</span> <span class="kt">ScreenCaptureService</span> <span class="p">{</span>
    <span class="kd">func</span> <span class="nf">captureAroundCursor</span><span class="p">()</span> <span class="k">async</span> <span class="o">-&gt;</span> <span class="kt">CaptureResult</span> <span class="p">{</span>
        <span class="k">do</span> <span class="p">{</span>
            <span class="k">let</span> <span class="nv">image</span> <span class="o">=</span> <span class="k">try</span> <span class="k">await</span> <span class="nf">performCapture</span><span class="p">(</span><span class="nv">fullWindow</span><span class="p">:</span> <span class="kc">false</span><span class="p">)</span>
            <span class="k">if</span> <span class="o">!</span><span class="kt">ImageDiffer</span><span class="o">.</span><span class="nf">hasSignificantChange</span><span class="p">(</span><span class="nv">from</span><span class="p">:</span> <span class="n">lastImage</span><span class="p">,</span> <span class="nv">to</span><span class="p">:</span> <span class="n">image</span><span class="p">)</span> <span class="p">{</span>
                <span class="k">return</span> <span class="o">.</span><span class="n">unchanged</span>
            <span class="p">}</span>
            <span class="n">lastImage</span> <span class="o">=</span> <span class="n">image</span>
            <span class="k">return</span> <span class="o">.</span><span class="nf">captured</span><span class="p">(</span><span class="n">image</span><span class="p">)</span>
        <span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
            <span class="k">return</span> <span class="o">.</span><span class="nf">unavailable</span><span class="p">(</span><span class="n">error</span><span class="o">.</span><span class="n">localizedDescription</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="kd">private</span> <span class="kd">func</span> <span class="nf">performCapture</span><span class="p">(</span><span class="nv">fullWindow</span><span class="p">:</span> <span class="kt">Bool</span><span class="p">)</span> <span class="k">async</span> <span class="k">throws</span> <span class="o">-&gt;</span> <span class="kt">CGImage</span> <span class="p">{</span>
        <span class="k">let</span> <span class="nv">content</span> <span class="o">=</span> <span class="k">try</span> <span class="k">await</span> <span class="kt">SCShareableContent</span><span class="o">.</span><span class="nf">excludingDesktopWindows</span><span class="p">(</span>
            <span class="kc">false</span><span class="p">,</span> <span class="nv">onScreenWindowsOnly</span><span class="p">:</span> <span class="kc">true</span>
        <span class="p">)</span>
        <span class="k">guard</span> <span class="k">let</span> <span class="nv">display</span> <span class="o">=</span> <span class="n">content</span><span class="o">.</span><span class="n">displays</span><span class="o">.</span><span class="n">first</span> <span class="k">else</span> <span class="p">{</span>
            <span class="k">throw</span> <span class="kt">CaptureError</span><span class="o">.</span><span class="n">noDisplay</span>
        <span class="p">}</span>

        <span class="c1">// Exclude VibeCat's own windows</span>
        <span class="k">let</span> <span class="nv">excludedApps</span> <span class="o">=</span> <span class="n">content</span><span class="o">.</span><span class="n">applications</span><span class="o">.</span><span class="n">filter</span> <span class="p">{</span> <span class="n">app</span> <span class="k">in</span>
            <span class="n">app</span><span class="o">.</span><span class="n">bundleIdentifier</span> <span class="o">==</span> <span class="kt">Bundle</span><span class="o">.</span><span class="n">main</span><span class="o">.</span><span class="n">bundleIdentifier</span>
        <span class="p">}</span>

        <span class="k">let</span> <span class="nv">filter</span> <span class="o">=</span> <span class="kt">SCContentFilter</span><span class="p">(</span>
            <span class="nv">display</span><span class="p">:</span> <span class="n">display</span><span class="p">,</span>
            <span class="nv">excludingApplications</span><span class="p">:</span> <span class="n">excludedApps</span><span class="p">,</span>
            <span class="nv">exceptingWindows</span><span class="p">:</span> <span class="p">[]</span>
        <span class="p">)</span>
        <span class="k">let</span> <span class="nv">config</span> <span class="o">=</span> <span class="kt">SCStreamConfiguration</span><span class="p">()</span>
        <span class="n">config</span><span class="o">.</span><span class="n">width</span> <span class="o">=</span> <span class="mi">1280</span>
        <span class="n">config</span><span class="o">.</span><span class="n">height</span> <span class="o">=</span> <span class="mi">720</span>
        <span class="n">config</span><span class="o">.</span><span class="n">pixelFormat</span> <span class="o">=</span> <span class="n">kCVPixelFormatType_32BGRA</span>
        <span class="n">config</span><span class="o">.</span><span class="n">showsCursor</span> <span class="o">=</span> <span class="kc">false</span>

        <span class="k">return</span> <span class="k">try</span> <span class="k">await</span> <span class="kt">SCScreenshotManager</span><span class="o">.</span><span class="nf">captureImage</span><span class="p">(</span>
            <span class="nv">contentFilter</span><span class="p">:</span> <span class="n">filter</span><span class="p">,</span> <span class="nv">configuration</span><span class="p">:</span> <span class="n">config</span>
        <span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Ran it in Xcode. Screen capture worked perfectly. Built a <code>.app</code> bundle with <code>swift build</code>. Ran it. Screen capture silently returned nothing. No error. No crash. Just... nothing.</p>

<p>The entitlement. The <code>com.apple.security.screen-recording</code> entitlement was in the Xcode project but wasn't getting embedded in the SPM-built binary. macOS doesn't throw an error when you try to capture without the entitlement — ScreenCaptureKit just quietly returns empty content. You get an empty <code>displays</code> array and no indication why.</p>

<p>I added it to <code>VibeCat.entitlements</code> and passed it via codesign:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>codesign <span class="nt">--force</span> <span class="nt">--entitlements</span> VibeCat/VibeCat.entitlements <span class="se">\</span>
  <span class="nt">--sign</span> - .build/release/VibeCat
</code></pre>

</div>



<p>First lesson: ScreenCaptureKit fails silently. If your capture returns nothing, check your entitlements before you check your code.</p>

<h2>
  
  
  the image differ — because you don't send every frame
</h2>

<p>The companion captures your screen periodically, but you don't want to send every single frame to the backend. If your screen hasn't changed, there's nothing new to analyze. So I built a pixel-level change detector:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">public</span> <span class="kd">enum</span> <span class="kt">ImageDiffer</span> <span class="p">{</span>
    <span class="kd">private</span> <span class="kd">static</span> <span class="k">let</span> <span class="nv">thumbnailSize</span> <span class="o">=</span> <span class="mi">32</span>

    <span class="kd">public</span> <span class="kd">static</span> <span class="kd">func</span> <span class="nf">hasSignificantChange</span><span class="p">(</span>
        <span class="n">from</span> <span class="nv">previous</span><span class="p">:</span> <span class="kt">CGImage</span><span class="p">?,</span>
        <span class="n">to</span> <span class="nv">current</span><span class="p">:</span> <span class="kt">CGImage</span><span class="p">,</span>
        <span class="nv">threshold</span><span class="p">:</span> <span class="kt">Double</span> <span class="o">=</span> <span class="mf">0.05</span>
    <span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">Bool</span> <span class="p">{</span>
        <span class="k">guard</span> <span class="k">let</span> <span class="nv">previous</span> <span class="k">else</span> <span class="p">{</span> <span class="k">return</span> <span class="kc">true</span> <span class="p">}</span>
        <span class="k">guard</span> <span class="k">let</span> <span class="nv">prevThumb</span> <span class="o">=</span> <span class="nf">thumbnail</span><span class="p">(</span><span class="n">previous</span><span class="p">),</span>
              <span class="k">let</span> <span class="nv">currThumb</span> <span class="o">=</span> <span class="nf">thumbnail</span><span class="p">(</span><span class="n">current</span><span class="p">)</span> <span class="k">else</span> <span class="p">{</span> <span class="k">return</span> <span class="kc">true</span> <span class="p">}</span>
        <span class="k">let</span> <span class="nv">diff</span> <span class="o">=</span> <span class="nf">pixelDiff</span><span class="p">(</span><span class="n">prevThumb</span><span class="p">,</span> <span class="n">currThumb</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">diff</span> <span class="o">&gt;</span> <span class="n">threshold</span>
    <span class="p">}</span>

    <span class="kd">private</span> <span class="kd">static</span> <span class="kd">func</span> <span class="nf">pixelDiff</span><span class="p">(</span><span class="n">_</span> <span class="nv">a</span><span class="p">:</span> <span class="p">[</span><span class="kt">UInt8</span><span class="p">],</span> <span class="n">_</span> <span class="nv">b</span><span class="p">:</span> <span class="p">[</span><span class="kt">UInt8</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="kt">Double</span> <span class="p">{</span>
        <span class="k">guard</span> <span class="n">a</span><span class="o">.</span><span class="n">count</span> <span class="o">==</span> <span class="n">b</span><span class="o">.</span><span class="n">count</span><span class="p">,</span> <span class="o">!</span><span class="n">a</span><span class="o">.</span><span class="n">isEmpty</span> <span class="k">else</span> <span class="p">{</span> <span class="k">return</span> <span class="mf">1.0</span> <span class="p">}</span>
        <span class="k">var</span> <span class="nv">total</span><span class="p">:</span> <span class="kt">Double</span> <span class="o">=</span> <span class="mi">0</span>
        <span class="k">for</span> <span class="n">i</span> <span class="k">in</span> <span class="nf">stride</span><span class="p">(</span><span class="nv">from</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="nv">to</span><span class="p">:</span> <span class="n">a</span><span class="o">.</span><span class="n">count</span><span class="p">,</span> <span class="nv">by</span><span class="p">:</span> <span class="mi">4</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">let</span> <span class="nv">dr</span> <span class="o">=</span> <span class="kt">Double</span><span class="p">(</span><span class="n">a</span><span class="p">[</span><span class="n">i</span><span class="p">])</span> <span class="o">-</span> <span class="kt">Double</span><span class="p">(</span><span class="n">b</span><span class="p">[</span><span class="n">i</span><span class="p">])</span>
            <span class="k">let</span> <span class="nv">dg</span> <span class="o">=</span> <span class="kt">Double</span><span class="p">(</span><span class="n">a</span><span class="p">[</span><span class="n">i</span><span class="o">+</span><span class="mi">1</span><span class="p">])</span> <span class="o">-</span> <span class="kt">Double</span><span class="p">(</span><span class="n">b</span><span class="p">[</span><span class="n">i</span><span class="o">+</span><span class="mi">1</span><span class="p">])</span>
            <span class="k">let</span> <span class="nv">db</span> <span class="o">=</span> <span class="kt">Double</span><span class="p">(</span><span class="n">a</span><span class="p">[</span><span class="n">i</span><span class="o">+</span><span class="mi">2</span><span class="p">])</span> <span class="o">-</span> <span class="kt">Double</span><span class="p">(</span><span class="n">b</span><span class="p">[</span><span class="n">i</span><span class="o">+</span><span class="mi">2</span><span class="p">])</span>
            <span class="n">total</span> <span class="o">+=</span> <span class="nf">sqrt</span><span class="p">(</span><span class="n">dr</span><span class="o">*</span><span class="n">dr</span> <span class="o">+</span> <span class="n">dg</span><span class="o">*</span><span class="n">dg</span> <span class="o">+</span> <span class="n">db</span><span class="o">*</span><span class="n">db</span><span class="p">)</span> <span class="o">/</span> <span class="p">(</span><span class="mf">255.0</span> <span class="o">*</span> <span class="nf">sqrt</span><span class="p">(</span><span class="mf">3.0</span><span class="p">))</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="n">total</span> <span class="o">/</span> <span class="kt">Double</span><span class="p">(</span><span class="n">a</span><span class="o">.</span><span class="n">count</span> <span class="o">/</span> <span class="mi">4</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It's a static method on an <code>enum</code> (no cases — just a namespace for functions). Downscale both images to 32×32, compute Euclidean distance in RGB space per pixel, average across all pixels. If the difference exceeds 5%, it's a "significant change" worth sending.</p>

<p>Why <code>enum</code> instead of <code>struct</code>? Because a struct can be accidentally instantiated. An enum with no cases is pure namespace — you can't create an instance of <code>ImageDiffer</code>. It's a Swift pattern for grouping static utility functions.</p>

<h2>
  
  
  Bundle.main.resourcePath — the Xcode lie
</h2>

<p>This one hurt. In SpriteAnimator, I needed to load PNG sprite frames from <code>Assets/Sprites/cat/</code>. First attempt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">let</span> <span class="nv">path</span> <span class="o">=</span> <span class="kt">Bundle</span><span class="o">.</span><span class="n">main</span><span class="o">.</span><span class="n">resourcePath</span><span class="o">!</span> <span class="o">+</span> <span class="s">"/Assets/Sprites/</span><span class="se">\(</span><span class="n">char</span><span class="se">)</span><span class="s">"</span>
</code></pre>

</div>



<p>Works perfectly in Xcode. The <code>!</code> force-unwrap succeeds. Files are found. Sprites animate.</p>

<p>Run the same binary outside Xcode? <code>Bundle.main.resourcePath</code> is nil. Force-unwrap crashes. Silent death.</p>

<p>The issue: when Xcode runs your app, <code>Bundle.main</code> points to your project directory structure where everything is available. When you build with SPM and run the <code>.app</code> independently, <code>Bundle.main.resourcePath</code> often returns nil because resources aren't in the expected bundle location.</p>

<p>The fix was a <code>findRepoRoot()</code> function that walks up from both the working directory and the bundle URL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">private</span> <span class="kd">func</span> <span class="nf">findRepoRoot</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="kt">URL</span> <span class="p">{</span>
    <span class="c1">// Try working directory first</span>
    <span class="k">var</span> <span class="nv">url</span> <span class="o">=</span> <span class="kt">URL</span><span class="p">(</span><span class="nv">fileURLWithPath</span><span class="p">:</span> <span class="kt">FileManager</span><span class="o">.</span><span class="k">default</span><span class="o">.</span><span class="n">currentDirectoryPath</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">_</span> <span class="k">in</span> <span class="mi">0</span><span class="o">..&lt;</span><span class="mi">6</span> <span class="p">{</span>
        <span class="k">if</span> <span class="kt">FileManager</span><span class="o">.</span><span class="k">default</span><span class="o">.</span><span class="nf">fileExists</span><span class="p">(</span>
            <span class="nv">atPath</span><span class="p">:</span> <span class="n">url</span><span class="o">.</span><span class="nf">appendingPathComponent</span><span class="p">(</span><span class="s">"Assets/Sprites"</span><span class="p">)</span><span class="o">.</span><span class="n">path</span>
        <span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">url</span>
        <span class="p">}</span>
        <span class="n">url</span> <span class="o">=</span> <span class="n">url</span><span class="o">.</span><span class="nf">deletingLastPathComponent</span><span class="p">()</span>
    <span class="p">}</span>
    <span class="c1">// Fallback: walk up from bundle URL</span>
    <span class="k">var</span> <span class="nv">bundleURL</span> <span class="o">=</span> <span class="kt">Bundle</span><span class="o">.</span><span class="n">main</span><span class="o">.</span><span class="n">bundleURL</span>
    <span class="k">for</span> <span class="n">_</span> <span class="k">in</span> <span class="mi">0</span><span class="o">..&lt;</span><span class="mi">6</span> <span class="p">{</span>
        <span class="k">if</span> <span class="kt">FileManager</span><span class="o">.</span><span class="k">default</span><span class="o">.</span><span class="nf">fileExists</span><span class="p">(</span>
            <span class="nv">atPath</span><span class="p">:</span> <span class="n">bundleURL</span><span class="o">.</span><span class="nf">appendingPathComponent</span><span class="p">(</span><span class="s">"Assets/Sprites"</span><span class="p">)</span><span class="o">.</span><span class="n">path</span>
        <span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span> <span class="n">bundleURL</span>
        <span class="p">}</span>
        <span class="n">bundleURL</span> <span class="o">=</span> <span class="n">bundleURL</span><span class="o">.</span><span class="nf">deletingLastPathComponent</span><span class="p">()</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="kt">URL</span><span class="p">(</span><span class="nv">fileURLWithPath</span><span class="p">:</span> <span class="kt">FileManager</span><span class="o">.</span><span class="k">default</span><span class="o">.</span><span class="n">currentDirectoryPath</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It's not pretty. But it works in Xcode, in a standalone <code>.app</code>, and when running from the terminal in the repo root. I used the same pattern in <code>BackgroundMusicPlayer</code> for finding <code>Assets/Music/</code>.</p>

<h2>
  
  
  the NSWindow.isVisible trap
</h2>

<p>Swift 6 strict concurrency plus AppKit is a minefield. Here's one that's particularly evil: <code>NSWindow</code> has a built-in property called <code>isVisible</code>. If you define your own stored property with the same name in a subclass or extension, Swift doesn't warn you — it just breaks.</p>

<p>I had:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">class</span> <span class="kt">CompanionPanel</span><span class="p">:</span> <span class="kt">NSPanel</span> <span class="p">{</span>
    <span class="k">var</span> <span class="nv">isVisible</span><span class="p">:</span> <span class="kt">Bool</span> <span class="o">=</span> <span class="kc">false</span>  <span class="c1">// ← shadows NSWindow.isVisible</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This compiles. It even seems to work at first. But <code>NSWindow.isVisible</code> is a computed property tied to the window server. My stored property hid it. Window visibility checks started returning wrong values. The panel would appear/disappear at random.</p>

<p>The fix was just a rename:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="k">var</span> <span class="nv">hudVisible</span><span class="p">:</span> <span class="kt">Bool</span> <span class="o">=</span> <span class="kc">false</span>
</code></pre>

</div>



<p>No warning from the compiler. No runtime error. Just subtle incorrectness that took hours to track down.</p>

<h2>
  
  
  @MainActor everywhere
</h2>

<p>Swift 6 requires <code>@MainActor</code> on anything that touches AppKit. In Swift 5 you could get away with updating UI from background threads — the app would work until it didn't. Swift 6 is strict: if a class touches <code>NSWindow</code>, <code>NSImage</code>, or any AppKit type, it must be <code>@MainActor</code>.</p>

<p>Every service class in VibeCat is <code>@MainActor</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@MainActor</span>
<span class="kd">final</span> <span class="kd">class</span> <span class="kt">ScreenCaptureService</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>

<span class="kd">@MainActor</span>
<span class="kd">final</span> <span class="kd">class</span> <span class="kt">SpriteAnimator</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>

<span class="kd">@MainActor</span>
<span class="kd">final</span> <span class="kd">class</span> <span class="kt">BackgroundMusicPlayer</span> <span class="p">{</span> <span class="o">...</span> <span class="p">}</span>
</code></pre>

</div>



<p>But <code>Timer</code> callbacks aren't <code>@MainActor</code> by default. So this pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kt">Timer</span><span class="o">.</span><span class="nf">scheduledTimer</span><span class="p">(</span><span class="nv">withTimeInterval</span><span class="p">:</span> <span class="mf">0.12</span><span class="p">,</span> <span class="nv">repeats</span><span class="p">:</span> <span class="kc">true</span><span class="p">)</span> <span class="p">{</span> <span class="n">_</span> <span class="k">in</span>
    <span class="k">self</span><span class="o">.</span><span class="nf">advanceFrame</span><span class="p">()</span>  <span class="c1">// ❌ Not on MainActor</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Has to become:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kt">Timer</span><span class="o">.</span><span class="nf">scheduledTimer</span><span class="p">(</span><span class="nv">withTimeInterval</span><span class="p">:</span> <span class="mf">0.12</span><span class="p">,</span> <span class="nv">repeats</span><span class="p">:</span> <span class="kc">true</span><span class="p">)</span> <span class="p">{</span> <span class="p">[</span><span class="k">weak</span> <span class="k">self</span><span class="p">]</span> <span class="n">_</span> <span class="k">in</span>
    <span class="kt">Task</span> <span class="p">{</span> <span class="kd">@MainActor</span> <span class="p">[</span><span class="k">weak</span> <span class="k">self</span><span class="p">]</span> <span class="k">in</span>
        <span class="k">self</span><span class="p">?</span><span class="o">.</span><span class="nf">advanceFrame</span><span class="p">()</span>  <span class="c1">// ✅ MainActor</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Every Timer. Every callback. Every closure that touches UI. Wrap it in <code>Task { @MainActor in }</code>. Swift 6 is safer, but the migration tax is real.</p>

<h2>
  
  
  the client is deliberately dumb
</h2>

<p>One design principle I'm proud of: the macOS client is deliberately dumb. It captures screens, plays audio, animates sprites, and shuttles data to the backend. It makes zero AI decisions.</p>

<p>When the client captures a screenshot, it doesn't analyze it — it sends the raw image to the backend's <code>/analyze</code> endpoint. When the backend says "set character to surprised," the client just changes the sprite state. When the backend says "play this audio," the client plays it.</p>

<p>This is a challenge requirement (all AI through backend), but it's also good architecture. The client is ~1,970 lines of Swift. The backend is ~2,900 lines of Go. If I need to change how VibeCat responds to errors, I never touch the client.</p>

<p>The smartest thing the client does is the <code>ImageDiffer</code> — and even that is just an optimization to avoid sending unchanged frames, not an AI decision.</p>

<h2>
  
  
  what I'd do differently
</h2>

<p>If I were starting over:</p>

<ol>
<li><p><strong>Test outside Xcode from day one.</strong> Every feature should be verified as a standalone <code>.app</code>, not just in the Xcode debug session. The silent failures cost me a full day.</p></li>
<li><p><strong>Use a resource bundle properly.</strong> The <code>findRepoRoot()</code> hack works, but it's fragile. A proper SPM resource bundle with <code>Bundle.module</code> would be cleaner.</p></li>
<li><p><strong>Start with Swift 6 strict concurrency enabled.</strong> I started with Swift 5 mode and migrated. The migration was painful — dozens of <code>@MainActor</code> annotations and callback wraps. Starting strict would have caught these at write-time instead of all-at-once.</p></li>
</ol>

<p>But it works. The cat sees your screen. The sprites animate. The music plays. And the client stays dumb enough to let the backend do the thinking.</p>

<p>The moment I ran the codesigned <code>.app</code> outside Xcode for the first time — double-clicked it from Finder, no debugger, no safety net — and the cat appeared on my desktop, captured my screen, and waved at me? That was the best moment of this entire project. Three days of silent failures, for ten seconds of a pixel cat saying hello.</p>




<p><em>Building VibeCat for the <a href="https://geminiliveagentchallenge.devpost.com/" rel="noopener noreferrer">Gemini Live Agent Challenge</a>. Source: <a href="https://github.com/Two-Weeks-Team/vibeCat" rel="noopener noreferrer">github.com/Two-Weeks-Team/vibeCat</a></em></p>

