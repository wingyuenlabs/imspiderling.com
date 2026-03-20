---
Title: steamworks-ffi-node: A Steamworks SDK Library for JavaScript Game Frameworks
Description: 
Author: Artur Khutak
Date: 2026-03-20T21:25:07.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  How It Started
</h2>

<p>I was building an Electron app for release on Steam. Everything went smoothly until I needed to add Steam integrations: achievements, leaderboards, cloud saves, and Steam Overlay support.</p>

<p>The first thing I found were two existing libraries: <a href="https://github.com/greenheartgames/greenworks" rel="noopener noreferrer">greenworks</a> and <a href="https://github.com/ceifa/steamworks.js" rel="noopener noreferrer">steamworks.js</a>. I tried both, but both had significant drawbacks.</p>

<p><strong>greenworks</strong> disappointed me because of missing Steam integrations, the need to compile the correct library version, and its focus on NW.js. </p>

<p><strong>steamworks.js</strong> had poor maintenance, lacked documentation, and had Steam Overlay issues — it only worked on Windows.</p>

<p>With the rise of AI coding agents, I decided to build my own library. That's how <a href="https://github.com/ArtyProf/steamworks-ffi-node" rel="noopener noreferrer">steamworks-ffi-node</a> was born — a Node.js wrapper for Steamworks SDK that uses FFI instead of native C++ compilation.</p>




<h2>
  
  
  The Problem: Steam SDK and JavaScript Don't Mix Well
</h2>

<p>Valve provides Steamworks SDK as a set of C++ headers and dynamic libraries (<em>steam_api64.dll</em>, <em>libsteam_api.so</em>, <em>libsteam_api.dylib</em>). To call these functions from<br>
Node.js, there are two classic approaches:</p>

<ol>
<li>
<strong>Native Node.js addon in C++ (Node-API / NAN)</strong> — compiled separately for each platform and Node version.</li>
<li>
<strong>FFI (Foreign Function Interface)</strong> — call library functions directly from JavaScript, with no compilation step.</li>
</ol>

<p><strong>greenworks</strong> and <strong>steamworks.js</strong> chose the first approach. <strong>steamworks-ffi-node</strong> chose the second.</p>


<h2>
  
  
  Overview of Existing Libraries
</h2>

<p><strong>greenworks</strong></p>

<p>greenworks is the oldest library in this space, originally developed by Greenheart Games for their game Game Dev Tycoon, later open-sourced.</p>

<p><strong>Approach</strong>: Native C++ addon built with NAN (Native Abstractions for Node.js).</p>

<p><strong>Issues I encountered:</strong></p>

<ul>
<li>Maintained on a <em>best-effort basis</em>
</li>
<li>No TypeScript typings</li>
<li>Supports Steamworks SDK v1.62 but parts of the API remain uncovered</li>
<li>High focus on NW.js</li>
<li>Last release: v0.22.0 (September 2025), 61 open issues</li>
</ul>

<p><strong>steamworks.js</strong></p>

<p>steamworks.js is a more modern alternative, built with Rust via NAPI-RS.</p>

<p><strong>What's good:</strong> No manual compilation needed, TypeScript typings included, modern API design.</p>

<p><strong>Limitations I hit:</strong></p>

<ul>
<li>Tied to specific Node.js versions via NAPI ABI</li>
<li>Significantly limited API coverage</li>
<li>Maintainer activity currently absent</li>
<li>Last release: v0.4.0 (two years ago on npm), 52 open issues</li>
</ul>


<h2>
  
  
  The Technical Decision: Why FFI Over Native Addons
</h2>

<p><strong>How Koffi Works</strong></p>

<p><a href="https://koffi.dev/" rel="noopener noreferrer">Koffi</a> is an FFI library for Node.js that lets you call functions from native dynamic libraries directly from JavaScript — no C++ code required.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">koffi</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">koffi</span><span class="dl">'</span><span class="p">;</span>
<span class="c1">// Load the Steam API library</span>
<span class="kd">const</span> <span class="nx">lib</span> <span class="o">=</span> <span class="nx">koffi</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="dl">'</span><span class="s1">steam_api64</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// Declare a function signature</span>
<span class="kd">const</span> <span class="nx">SteamAPI_Init</span> <span class="o">=</span> <span class="nx">lib</span><span class="p">.</span><span class="nf">func</span><span class="p">(</span><span class="dl">'</span><span class="s1">bool SteamAPI_Init()</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// Call it directly from JavaScript!</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nc">SteamAPI_Init</span><span class="p">();</span>
</code></pre>

</div>



<p><strong>No Compilation on Install</strong></p>

<p>The FFI approach means the library is pure JavaScript/TypeScript that installs as a regular npm package. No <em>node-gyp</em>, no Visual Studio Build Tools, no Xcode.</p>

<p><strong>Node.js Version Independence</strong></p>

<p>Native addons are tied to the ABI of a specific V8/Node version. Upgrading Electron forces recompilation or waiting for a new release. Koffi FFI calls <em>.so</em>/<em>.dll</em> directly. The Steam binary ABI doesn't change. Upgrading Electron doesn't break the integration.</p>

<p><strong>Easy to Contribute</strong></p>

<p>The entire library is written in TypeScript, so there's no separate compilation step. Adding a new Steamworks function is straightforward.</p>




<h2>
  
  
  Tricky Technical Challenges
</h2>

<p><strong>The Struct Return ABI Problem on Linux</strong></p>

<p>This was the most interesting bug I fixed in version 0.9.3. The Steam Input API returns structures directly from functions.</p>

<p>On <strong>Windows</strong> (MSVC x64 ABI), small structs are returned via registers RAX/RDX. On <strong>Linux</strong> (System V x86_64 ABI), small structs up to 16 bytes return in RAX:RDX registers. Large structs (like InputMotionData_t at 40 bytes) return via a hidden pointer passed as the <strong>first</strong> argument.</p>

<p>My incorrect code passed the buffer as the <strong>second</strong> argument — the function wrote 40 bytes into ISteamInput , causing memory corruption and segfaults.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// INCORRECT — worked only on Windows</span>
<span class="kd">const</span> <span class="nx">buf</span> <span class="o">=</span> <span class="nx">Buffer</span><span class="p">.</span><span class="nf">alloc</span><span class="p">(</span><span class="mi">8</span><span class="p">);</span>
<span class="k">this</span><span class="p">.</span><span class="nx">steamLib</span><span class="p">.</span><span class="nf">func</span><span class="p">(</span><span class="dl">'</span><span class="s1">SteamAPI_ISteamInput_GetDigitalActionData</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">void</span><span class="dl">'</span><span class="p">,</span> 
  <span class="p">[</span><span class="dl">'</span><span class="s1">void*</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">void*</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">uint64</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">uint64</span><span class="dl">'</span><span class="p">])(</span><span class="nx">iface</span><span class="p">,</span> <span class="nx">buf</span><span class="p">,</span> <span class="nx">handle</span><span class="p">,</span> <span class="nx">actionHandle</span><span class="p">);</span>
</code></pre>

</div>



<p>The fix: declare structs through Koffi and use the correct return type. Koffi automatically selects the correct calling convention depending on platform and struct size.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// CORRECT: let Koffi handle the ABI</span>
<span class="kd">const</span> <span class="nx">InputMotionData_t</span> <span class="o">=</span> <span class="nx">koffi</span><span class="p">.</span><span class="nf">struct</span><span class="p">(</span><span class="dl">'</span><span class="s1">InputMotionData_t</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
<span class="na">rotQuatX</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">rotQuatY</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">rotQuatZ</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">rotQuatW</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">posAccelX</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">posAccelY</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">posAccelZ</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">rotVelX</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">rotVelY</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="na">rotVelZ</span><span class="p">:</span> <span class="dl">'</span><span class="s1">float</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>
<span class="kd">const</span> <span class="nx">GetMotionData</span> <span class="o">=</span> <span class="nx">lib</span><span class="p">.</span><span class="nf">func</span><span class="p">(</span>
<span class="dl">'</span><span class="s1">InputMotionData_t SteamAPI_ISteamInput_GetMotionData(void*, uint64)</span><span class="dl">'</span>
<span class="p">);</span>
</code></pre>

</div>



<p><strong>Native Steam Overlay for Electron</strong></p>

<p>The standard Steam Overlay (Shift+Tab) doesn't work in Electron games out of the box — Steam doesn't know how to inject its rendering into Chromium.</p>

<p>I implemented native C++ modules for each platform:</p>

<ul>
<li>
<strong>macOS</strong>: Metal rendering via <em>MTKView</em> and <em>CAMetalLayer</em>
</li>
<li>
<strong>Windows</strong>: OpenGL via Win32 window hooks</li>
<li>
<strong>Linux/SteamOS</strong>: OpenGL 3.3 with GLX (tested on Steam Deck Desktop Mode)</li>
</ul>

<p>This is the only part requiring native compilation, but prebuilt binaries ship via npm.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5c6wbs8bvyo1otsujni1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5c6wbs8bvyo1otsujni1.png" alt=" " width="800" height="926"></a></p>




<h2>
  
  
  Library Architecture
</h2>

<p>Each manager is responsible for one section of the Steamworks API, making it easy to add new APIs without modifying existing code.</p>

<p>Key API modules currently supported:</p>

<ul>
<li>
<strong>Achievements</strong> — 100% coverage (all 20 functions)</li>
<li>
<strong>Stats</strong> — 100% coverage (all 14 functions)</li>
<li>
<strong>Leaderboards</strong> — 100% coverage (all 7 functions)</li>
<li>
<strong>Friends</strong> — 22 complete social functions</li>
<li>
<strong>Rich Presence</strong> — 6 functions for custom status display</li>
<li>
<strong>Cloud Storage</strong> — Full file management support</li>
<li>
<strong>Steam Input</strong> — Controller/gamepad support</li>
<li>
<strong>Steam Overlay</strong> — Native overlay for Electron apps</li>
</ul>




<h2>
  
  
  Usage Example
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">SteamworksSDK</span><span class="p">,</span> <span class="p">{</span>
  <span class="nx">LeaderboardSortMethod</span><span class="p">,</span>
  <span class="nx">LeaderboardDisplayType</span><span class="p">,</span>
  <span class="nx">LeaderboardUploadScoreMethod</span><span class="p">,</span>
  <span class="nx">LeaderboardDataRequest</span><span class="p">,</span>
  <span class="nx">EFriendFlags</span><span class="p">,</span>
  <span class="nx">EUGCQuery</span><span class="p">,</span>
  <span class="nx">EUGCMatchingUGCType</span><span class="p">,</span>
  <span class="nx">EItemState</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">steamworks-ffi-node</span><span class="dl">"</span><span class="p">;</span>

<span class="c1">// Helper to auto-start callback polling</span>
<span class="kd">function</span> <span class="nf">startCallbackPolling</span><span class="p">(</span><span class="nx">steam</span><span class="p">:</span> <span class="nx">SteamworksSDK</span><span class="p">,</span> <span class="nx">interval</span><span class="p">:</span> <span class="nx">number</span> <span class="o">=</span> <span class="mi">1000</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">steam</span><span class="p">.</span><span class="nf">runCallbacks</span><span class="p">();</span>
  <span class="p">},</span> <span class="nx">interval</span><span class="p">);</span>
<span class="p">}</span>

<span class="c1">// Initialize Steam connection</span>
<span class="kd">const</span> <span class="nx">steam</span> <span class="o">=</span> <span class="nx">SteamworksSDK</span><span class="p">.</span><span class="nf">getInstance</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">initialized</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nf">init</span><span class="p">({</span> <span class="na">appId</span><span class="p">:</span> <span class="mi">480</span> <span class="p">});</span> <span class="c1">// Your Steam App ID</span>

<span class="k">if </span><span class="p">(</span><span class="nx">initialized</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Start callback polling automatically (required for async operations)</span>
  <span class="kd">const</span> <span class="nx">callbackInterval</span> <span class="o">=</span> <span class="nf">startCallbackPolling</span><span class="p">(</span><span class="nx">steam</span><span class="p">,</span> <span class="mi">1000</span><span class="p">);</span>

  <span class="c1">// Get current Steam language for localization</span>
  <span class="kd">const</span> <span class="nx">language</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nf">getCurrentGameLanguage</span><span class="p">();</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Steam language:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">language</span><span class="p">);</span> <span class="c1">// e.g., 'english', 'french', 'german'</span>

  <span class="c1">// Get achievements from Steam servers</span>
  <span class="kd">const</span> <span class="nx">achievements</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">achievements</span><span class="p">.</span><span class="nf">getAllAchievements</span><span class="p">();</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Steam achievements:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">achievements</span><span class="p">);</span>

  <span class="c1">// Unlock achievement (permanent in Steam!)</span>
  <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">achievements</span><span class="p">.</span><span class="nf">unlockAchievement</span><span class="p">(</span><span class="dl">"</span><span class="s2">ACH_WIN_ONE_GAME</span><span class="dl">"</span><span class="p">);</span>

  <span class="c1">// Check unlock status from Steam</span>
  <span class="kd">const</span> <span class="nx">isUnlocked</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">achievements</span><span class="p">.</span><span class="nf">isAchievementUnlocked</span><span class="p">(</span>
    <span class="dl">"</span><span class="s2">ACH_WIN_ONE_GAME</span><span class="dl">"</span>
  <span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Achievement unlocked:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">isUnlocked</span><span class="p">);</span>

  <span class="c1">// Track user statistics</span>
  <span class="kd">const</span> <span class="nx">kills</span> <span class="o">=</span> <span class="p">(</span><span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">stats</span><span class="p">.</span><span class="nf">getStatInt</span><span class="p">(</span><span class="dl">"</span><span class="s2">total_kills</span><span class="dl">"</span><span class="p">))</span> <span class="o">||</span> <span class="mi">0</span><span class="p">;</span>
  <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">stats</span><span class="p">.</span><span class="nf">setStatInt</span><span class="p">(</span><span class="dl">"</span><span class="s2">total_kills</span><span class="dl">"</span><span class="p">,</span> <span class="nx">kills</span> <span class="o">+</span> <span class="mi">1</span><span class="p">);</span>

  <span class="c1">// Get global statistics</span>
  <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">stats</span><span class="p">.</span><span class="nf">requestGlobalStats</span><span class="p">(</span><span class="mi">7</span><span class="p">);</span>
  <span class="k">await</span> <span class="k">new</span> <span class="nc">Promise</span><span class="p">((</span><span class="nx">resolve</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">setTimeout</span><span class="p">(</span><span class="nx">resolve</span><span class="p">,</span> <span class="mi">2000</span><span class="p">));</span>
  <span class="nx">steam</span><span class="p">.</span><span class="nf">runCallbacks</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">globalKills</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">stats</span><span class="p">.</span><span class="nf">getGlobalStatInt</span><span class="p">(</span><span class="dl">"</span><span class="s2">global.total_kills</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Total kills worldwide:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">globalKills</span><span class="p">);</span>

  <span class="c1">// Work with leaderboards</span>
  <span class="kd">const</span> <span class="nx">leaderboard</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">leaderboards</span><span class="p">.</span><span class="nf">findOrCreateLeaderboard</span><span class="p">(</span>
    <span class="dl">"</span><span class="s2">HighScores</span><span class="dl">"</span><span class="p">,</span>
    <span class="nx">LeaderboardSortMethod</span><span class="p">.</span><span class="nx">Descending</span><span class="p">,</span>
    <span class="nx">LeaderboardDisplayType</span><span class="p">.</span><span class="nx">Numeric</span>
  <span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">leaderboard</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Upload score</span>
    <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">leaderboards</span><span class="p">.</span><span class="nf">uploadLeaderboardScore</span><span class="p">(</span>
      <span class="nx">leaderboard</span><span class="p">.</span><span class="nx">handle</span><span class="p">,</span>
      <span class="mi">1000</span><span class="p">,</span>
      <span class="nx">LeaderboardUploadScoreMethod</span><span class="p">.</span><span class="nx">KeepBest</span>
    <span class="p">);</span>

    <span class="c1">// Download top 10 scores</span>
    <span class="kd">const</span> <span class="nx">topScores</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">leaderboards</span><span class="p">.</span><span class="nf">downloadLeaderboardEntries</span><span class="p">(</span>
      <span class="nx">leaderboard</span><span class="p">.</span><span class="nx">handle</span><span class="p">,</span>
      <span class="nx">LeaderboardDataRequest</span><span class="p">.</span><span class="nx">Global</span><span class="p">,</span>
      <span class="mi">1</span><span class="p">,</span>
      <span class="mi">10</span>
    <span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Top 10 scores:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">topScores</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Access friends and social features</span>
  <span class="kd">const</span> <span class="nx">personaName</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getPersonaName</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">friendCount</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendCount</span><span class="p">(</span><span class="nx">EFriendFlags</span><span class="p">.</span><span class="nx">All</span><span class="p">);</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">personaName</span><span class="p">}</span><span class="s2"> has </span><span class="p">${</span><span class="nx">friendCount</span><span class="p">}</span><span class="s2"> friends`</span><span class="p">);</span>

  <span class="c1">// Get all friends with details</span>
  <span class="kd">const</span> <span class="nx">allFriends</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getAllFriends</span><span class="p">(</span><span class="nx">EFriendFlags</span><span class="p">.</span><span class="nx">All</span><span class="p">);</span>
  <span class="nx">allFriends</span><span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">5</span><span class="p">).</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">friend</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">name</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendPersonaName</span><span class="p">(</span><span class="nx">friend</span><span class="p">.</span><span class="nx">steamId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">state</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendPersonaState</span><span class="p">(</span><span class="nx">friend</span><span class="p">.</span><span class="nx">steamId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">level</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendSteamLevel</span><span class="p">(</span><span class="nx">friend</span><span class="p">.</span><span class="nx">steamId</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">name</span><span class="p">}</span><span class="s2">: Level </span><span class="p">${</span><span class="nx">level</span><span class="p">}</span><span class="s2">, Status: </span><span class="p">${</span><span class="nx">state</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="c1">// Get avatar handles</span>
    <span class="kd">const</span> <span class="nx">smallAvatar</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getSmallFriendAvatar</span><span class="p">(</span><span class="nx">friend</span><span class="p">.</span><span class="nx">steamId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">mediumAvatar</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getMediumFriendAvatar</span><span class="p">(</span><span class="nx">friend</span><span class="p">.</span><span class="nx">steamId</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">smallAvatar</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
        <span class="s2">`  Avatar handles: small=</span><span class="p">${</span><span class="nx">smallAvatar</span><span class="p">}</span><span class="s2">, medium=</span><span class="p">${</span><span class="nx">mediumAvatar</span><span class="p">}</span><span class="s2">`</span>
      <span class="p">);</span>
    <span class="p">}</span>

    <span class="c1">// Check if playing a game</span>
    <span class="kd">const</span> <span class="nx">gameInfo</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendGamePlayed</span><span class="p">(</span><span class="nx">friend</span><span class="p">.</span><span class="nx">steamId</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">gameInfo</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  Playing: App </span><span class="p">${</span><span class="nx">gameInfo</span><span class="p">.</span><span class="nx">gameId</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">});</span>

  <span class="c1">// Check friend groups (tags)</span>
  <span class="kd">const</span> <span class="nx">groupCount</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendsGroupCount</span><span class="p">();</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">groupCount</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">groupId</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendsGroupIDByIndex</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">groupName</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendsGroupName</span><span class="p">(</span><span class="nx">groupId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">members</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendsGroupMembersList</span><span class="p">(</span><span class="nx">groupId</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Group "</span><span class="p">${</span><span class="nx">groupName</span><span class="p">}</span><span class="s2">" has </span><span class="p">${</span><span class="nx">members</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> members`</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Check recently played with</span>
  <span class="kd">const</span> <span class="nx">coplayCount</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getCoplayFriendCount</span><span class="p">();</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">coplayCount</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">recentPlayer</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getCoplayFriend</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">playerName</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendPersonaName</span><span class="p">(</span><span class="nx">recentPlayer</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">coplayTime</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">friends</span><span class="p">.</span><span class="nf">getFriendCoplayTime</span><span class="p">(</span><span class="nx">recentPlayer</span><span class="p">);</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Recently played with </span><span class="p">${</span><span class="nx">playerName</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Set rich presence for custom status</span>
  <span class="nx">steam</span><span class="p">.</span><span class="nx">richPresence</span><span class="p">.</span><span class="nf">setRichPresence</span><span class="p">(</span><span class="dl">"</span><span class="s2">status</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">In Main Menu</span><span class="dl">"</span><span class="p">);</span>
  <span class="nx">steam</span><span class="p">.</span><span class="nx">richPresence</span><span class="p">.</span><span class="nf">setRichPresence</span><span class="p">(</span><span class="dl">"</span><span class="s2">connect</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">+connect server:27015</span><span class="dl">"</span><span class="p">);</span>

  <span class="c1">// Open Steam overlay</span>
  <span class="nx">steam</span><span class="p">.</span><span class="nx">overlay</span><span class="p">.</span><span class="nf">activateGameOverlay</span><span class="p">(</span><span class="dl">"</span><span class="s2">Friends</span><span class="dl">"</span><span class="p">);</span> <span class="c1">// Open friends list</span>
  <span class="nx">steam</span><span class="p">.</span><span class="nx">overlay</span><span class="p">.</span><span class="nf">activateGameOverlayToWebPage</span><span class="p">(</span><span class="dl">"</span><span class="s2">https://example.com/wiki</span><span class="dl">"</span><span class="p">);</span> <span class="c1">// Open wiki</span>

  <span class="c1">// Steam Cloud storage operations</span>
  <span class="kd">const</span> <span class="nx">saveData</span> <span class="o">=</span> <span class="p">{</span> <span class="na">level</span><span class="p">:</span> <span class="mi">5</span><span class="p">,</span> <span class="na">score</span><span class="p">:</span> <span class="mi">1000</span><span class="p">,</span> <span class="na">inventory</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">sword</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">shield</span><span class="dl">"</span><span class="p">]</span> <span class="p">};</span>
  <span class="kd">const</span> <span class="nx">buffer</span> <span class="o">=</span> <span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">saveData</span><span class="p">));</span>

  <span class="c1">// Write save file to Steam Cloud</span>
  <span class="kd">const</span> <span class="nx">written</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">cloud</span><span class="p">.</span><span class="nf">fileWrite</span><span class="p">(</span><span class="dl">"</span><span class="s2">savegame.json</span><span class="dl">"</span><span class="p">,</span> <span class="nx">buffer</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">written</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">✅ Save uploaded to Steam Cloud</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Check cloud quota</span>
  <span class="kd">const</span> <span class="nx">quota</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">cloud</span><span class="p">.</span><span class="nf">getQuota</span><span class="p">();</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
    <span class="s2">`Cloud storage: </span><span class="p">${</span><span class="nx">quota</span><span class="p">.</span><span class="nx">usedBytes</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span>
      <span class="nx">quota</span><span class="p">.</span><span class="nx">totalBytes</span>
    <span class="p">}</span><span class="s2"> bytes (</span><span class="p">${</span><span class="nx">quota</span><span class="p">.</span><span class="nx">percentUsed</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">)}</span><span class="s2">%)`</span>
  <span class="p">);</span>

  <span class="c1">// Read save file from Steam Cloud</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">steam</span><span class="p">.</span><span class="nx">cloud</span><span class="p">.</span><span class="nf">fileExists</span><span class="p">(</span><span class="dl">"</span><span class="s2">savegame.json</span><span class="dl">"</span><span class="p">))</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">cloud</span><span class="p">.</span><span class="nf">fileRead</span><span class="p">(</span><span class="dl">"</span><span class="s2">savegame.json</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">success</span> <span class="o">&amp;&amp;</span> <span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">loadedSave</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nf">toString</span><span class="p">());</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
        <span class="s2">`Loaded save: Level </span><span class="p">${</span><span class="nx">loadedSave</span><span class="p">.</span><span class="nx">level</span><span class="p">}</span><span class="s2">, Score </span><span class="p">${</span><span class="nx">loadedSave</span><span class="p">.</span><span class="nx">score</span><span class="p">}</span><span class="s2">`</span>
      <span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="c1">// List all cloud files</span>
  <span class="kd">const</span> <span class="nx">cloudFiles</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">cloud</span><span class="p">.</span><span class="nf">getAllFiles</span><span class="p">();</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Steam Cloud contains </span><span class="p">${</span><span class="nx">cloudFiles</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> files:`</span><span class="p">);</span>
  <span class="nx">cloudFiles</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">file</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">kb</span> <span class="o">=</span> <span class="p">(</span><span class="nx">file</span><span class="p">.</span><span class="nx">size</span> <span class="o">/</span> <span class="mi">1024</span><span class="p">).</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">2</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">status</span> <span class="o">=</span> <span class="nx">file</span><span class="p">.</span><span class="nx">persisted</span> <span class="p">?</span> <span class="dl">"</span><span class="s2">☁️</span><span class="dl">"</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">⏳</span><span class="dl">"</span><span class="p">;</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">status</span><span class="p">}</span><span class="s2"> </span><span class="p">${</span><span class="nx">file</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2"> - </span><span class="p">${</span><span class="nx">kb</span><span class="p">}</span><span class="s2"> KB`</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="c1">// Steam Workshop operations</span>
  <span class="c1">// Subscribe to a Workshop item</span>
  <span class="kd">const</span> <span class="nx">subscribeResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">subscribeItem</span><span class="p">(</span><span class="mi">123456789</span><span class="nx">n</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">subscribeResult</span><span class="p">.</span><span class="nx">success</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">✅ Subscribed to Workshop item</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Get all subscribed items</span>
  <span class="kd">const</span> <span class="nx">subscribedItems</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">getSubscribedItems</span><span class="p">();</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Subscribed to </span><span class="p">${</span><span class="nx">subscribedItems</span><span class="p">.</span><span class="nx">length</span><span class="p">}</span><span class="s2"> Workshop items`</span><span class="p">);</span>

  <span class="c1">// Query Workshop items with text search</span>
  <span class="kd">const</span> <span class="nx">query</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">createQueryAllUGCRequest</span><span class="p">(</span>
    <span class="nx">EUGCQuery</span><span class="p">.</span><span class="nx">RankedByTextSearch</span><span class="p">,</span>
    <span class="nx">EUGCMatchingUGCType</span><span class="p">.</span><span class="nx">Items</span><span class="p">,</span>
    <span class="mi">480</span><span class="p">,</span> <span class="c1">// Creator App ID</span>
    <span class="mi">480</span><span class="p">,</span> <span class="c1">// Consumer App ID</span>
    <span class="mi">1</span> <span class="c1">// Page 1</span>
  <span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">query</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Set search text to filter results</span>
    <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">setSearchText</span><span class="p">(</span><span class="nx">query</span><span class="p">,</span> <span class="dl">"</span><span class="s2">map</span><span class="dl">"</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">queryResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">sendQueryUGCRequest</span><span class="p">(</span><span class="nx">query</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">queryResult</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
        <span class="s2">`Found </span><span class="p">${</span><span class="nx">queryResult</span><span class="p">.</span><span class="nx">numResults</span><span class="p">}</span><span class="s2"> Workshop items matching "map"`</span>
      <span class="p">);</span>

      <span class="c1">// Get details for each item</span>
      <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">queryResult</span><span class="p">.</span><span class="nx">numResults</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">details</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">getQueryUGCResult</span><span class="p">(</span><span class="nx">query</span><span class="p">,</span> <span class="nx">i</span><span class="p">);</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">details</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`📦 </span><span class="p">${</span><span class="nx">details</span><span class="p">.</span><span class="nx">title</span><span class="p">}</span><span class="s2"> by </span><span class="p">${</span><span class="nx">details</span><span class="p">.</span><span class="nx">steamIDOwner</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
          <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
            <span class="s2">`   Score: </span><span class="p">${</span><span class="nx">details</span><span class="p">.</span><span class="nx">score</span><span class="p">}</span><span class="s2">, Downloads: </span><span class="p">${</span><span class="nx">details</span><span class="p">.</span><span class="nx">numUniqueSubscriptions</span><span class="p">}</span><span class="s2">`</span>
          <span class="p">);</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span>
    <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">releaseQueryUGCRequest</span><span class="p">(</span><span class="nx">query</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Check download progress for subscribed items</span>
  <span class="nx">subscribedItems</span><span class="p">.</span><span class="nf">forEach</span><span class="p">((</span><span class="nx">itemId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">state</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">getItemState</span><span class="p">(</span><span class="nx">itemId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">stateFlags</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">state</span> <span class="o">&amp;</span> <span class="nx">EItemState</span><span class="p">.</span><span class="nx">Subscribed</span><span class="p">)</span> <span class="nx">stateFlags</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="dl">"</span><span class="s2">Subscribed</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">state</span> <span class="o">&amp;</span> <span class="nx">EItemState</span><span class="p">.</span><span class="nx">NeedsUpdate</span><span class="p">)</span> <span class="nx">stateFlags</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="dl">"</span><span class="s2">Needs Update</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">state</span> <span class="o">&amp;</span> <span class="nx">EItemState</span><span class="p">.</span><span class="nx">Installed</span><span class="p">)</span> <span class="nx">stateFlags</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="dl">"</span><span class="s2">Installed</span><span class="dl">"</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">state</span> <span class="o">&amp;</span> <span class="nx">EItemState</span><span class="p">.</span><span class="nx">Downloading</span><span class="p">)</span> <span class="nx">stateFlags</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="dl">"</span><span class="s2">Downloading</span><span class="dl">"</span><span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Item </span><span class="p">${</span><span class="nx">itemId</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">stateFlags</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="dl">"</span><span class="s2">, </span><span class="dl">"</span><span class="p">)}</span><span class="s2">`</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">state</span> <span class="o">&amp;</span> <span class="nx">EItemState</span><span class="p">.</span><span class="nx">Downloading</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// If downloading</span>
      <span class="kd">const</span> <span class="nx">progress</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">getItemDownloadInfo</span><span class="p">(</span><span class="nx">itemId</span><span class="p">);</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">progress</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">percent</span> <span class="o">=</span> <span class="p">((</span><span class="nx">progress</span><span class="p">.</span><span class="nx">downloaded</span> <span class="o">/</span> <span class="nx">progress</span><span class="p">.</span><span class="nx">total</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">).</span><span class="nf">toFixed</span><span class="p">(</span>
          <span class="mi">1</span>
        <span class="p">);</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span>
          <span class="s2">`  Download: </span><span class="p">${</span><span class="nx">percent</span><span class="p">}</span><span class="s2">% (</span><span class="p">${</span><span class="nx">progress</span><span class="p">.</span><span class="nx">downloaded</span><span class="p">}</span><span class="s2">/</span><span class="p">${</span><span class="nx">progress</span><span class="p">.</span><span class="nx">total</span><span class="p">}</span><span class="s2"> bytes)`</span>
        <span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">state</span> <span class="o">&amp;</span> <span class="nx">EItemState</span><span class="p">.</span><span class="nx">Installed</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// If installed</span>
      <span class="kd">const</span> <span class="nx">info</span> <span class="o">=</span> <span class="nx">steam</span><span class="p">.</span><span class="nx">workshop</span><span class="p">.</span><span class="nf">getItemInstallInfo</span><span class="p">(</span><span class="nx">itemId</span><span class="p">);</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">info</span><span class="p">.</span><span class="nx">success</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`  Installed at: </span><span class="p">${</span><span class="nx">info</span><span class="p">.</span><span class="nx">folder</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">}</span>

<span class="c1">// Cleanup</span>
<span class="nf">clearInterval</span><span class="p">(</span><span class="nx">callbackInterval</span><span class="p">);</span>
<span class="nx">steam</span><span class="p">.</span><span class="nf">shutdown</span><span class="p">();</span>
</code></pre>

</div>






<h2>
  
  
  Comparison Table
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>greenworks</th>
<th>steamworks.js</th>
<th>steamworks-ffi-node</th>
</tr>
</thead>
<tbody>
<tr>
<td>Approach</td>
<td>C++ NAN addon</td>
<td>Rust NAPI-RS</td>
<td>Koffi FFI</td>
</tr>
<tr>
<td>Compilation needed</td>
<td>Yes</td>
<td>Precompiled</td>
<td>No</td>
</tr>
<tr>
<td>TypeScript</td>
<td>No</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Node version lock</td>
<td>Yes</td>
<td>Yes (NAPI ABI)</td>
<td>No</td>
</tr>
<tr>
<td>API coverage</td>
<td>Partial</td>
<td>Limited</td>
<td>Extensive</td>
</tr>
<tr>
<td>Steam Overlay</td>
<td>support only NW.js</td>
<td>Windows only (partial Linux) for Electron</td>
<td>Win, macOS, Linux for Electron</td>
</tr>
<tr>
<td>Active maintenance</td>
<td>Best-effort</td>
<td>Inactive</td>
<td>Active</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Games Already Using steamworks-ffi-node
</h2>

<ul>
<li><a href="https://store.steampowered.com/app/4087020/" rel="noopener noreferrer">Bulletail</a></li>
<li><a href="https://store.steampowered.com/app/3715040/" rel="noopener noreferrer">DekaDuck</a></li>
<li><a href="https://store.steampowered.com/app/3905510/" rel="noopener noreferrer">Falling Face Fragments</a></li>
<li><a href="https://store.steampowered.com/app/4118640/" rel="noopener noreferrer">Idle Looter</a></li>
<li><a href="https://store.steampowered.com/app/4385780/" rel="noopener noreferrer">Maldhalla</a></li>
<li><a href="https://store.steampowered.com/app/4072610/" rel="noopener noreferrer">Realms of Conflict</a></li>
</ul>




<h2>
  
  
  Conclusion
</h2>

<p>I gained invaluable experience working with the FFI approach, Steamworks SDK, and solving challenges like the Steam Input ABI bug and cross-platform Steam Overlay.</p>

<p>If you're building a game on Electron or another JS framework and need Steam integration, you now have a choice between three approaches:</p>

<ul>
<li>
<strong>greenworks</strong> — for basic functionality if you don't mind C++ compiler dependencies</li>
<li>
<strong>steamworks.js</strong> — for limited API needs with a Rust implementation</li>
<li>
<strong>steamworks-ffi-node</strong> — for comprehensive API coverage, zero compilation, TypeScript support, and active development</li>
</ul>

<p>Also if you will plan to use library or face any issues do not hesitate to drop a question or comment here or create a <a href="https://github.com/ArtyProf/steamworks-ffi-node/issues" rel="noopener noreferrer">Github issue</a>. Will be happy for any feedback from you!</p>

<p>If this was useful — drop a star on <a href="https://github.com/ArtyProf/steamworks-ffi-node" rel="noopener noreferrer">GitHub</a>!</p>

<p><strong>Links</strong>:</p>

<ul>
<li>Github: <a href="https://github.com/ArtyProf/steamworks-ffi-node" rel="noopener noreferrer">https://github.com/ArtyProf/steamworks-ffi-node</a>
</li>
<li>NPM: <a href="https://www.npmjs.com/package/steamworks-ffi-node" rel="noopener noreferrer">https://www.npmjs.com/package/steamworks-ffi-node</a>
</li>
<li>Docs: <a href="https://github.com/ArtyProf/steamworks-ffi-node/blob/main/docs/README.md" rel="noopener noreferrer">https://github.com/ArtyProf/steamworks-ffi-node/blob/main/docs/README.md</a>
</li>
<li>Steam Overlay integration details for Electron: <a href="https://github.com/ArtyProf/steamworks-ffi-node/blob/main/docs/STEAM_OVERLAY_INTEGRATION.md" rel="noopener noreferrer">https://github.com/ArtyProf/steamworks-ffi-node/blob/main/docs/STEAM_OVERLAY_INTEGRATION.md</a>
</li>
</ul>

