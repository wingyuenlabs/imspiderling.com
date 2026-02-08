---
Title: Design-Driven Firebase in Flutter: Building Custom In-App Messaging with SwiftUI & Compose
Description: 
Author: Salah Nahed
Date: 2026-02-08T20:53:54.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🆕 <a href="https://medium.com/@salahamassi/design-driven-firebase-in-flutter-building-custom-in-app-messaging-with-swiftui-compose-ff2b7875d8a7" rel="noopener noreferrer">Design-Driven Firebase in Flutter: Building Custom In-App Messaging with SwiftUI &amp; Compose</a>
</h2>

<p>In-app messaging plays a critical role in how users interact with modern apps — from promoting offers to driving engagement and retention.<br>
<strong>Published on:</strong> April 11, 2025 · ⏱️ 10 min read</p>


<h2>
  
  
  Android Ain’t Easy: Crafting Custom FIAM with Compose
</h2>
<h3>
  
  
  Setting Up Jetpack Compose: The First Brick in the Wall 🧱
</h3>

<p>Before diving into the custom renderer, we first needed to enable Jetpack Compose in our Flutter project’s native Android layer. This meant updating the <code>build.gradle</code> file with the proper buildFeatures, setting the <code>kotlinCompilerExtensionVersion</code>, and pulling in all the right dependencies.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="nf">buildFeatures</span> <span class="p">{</span>
    <span class="n">compose</span> <span class="k">true</span>
<span class="p">}</span>

<span class="nf">composeOptions</span> <span class="p">{</span>
    <span class="n">kotlinCompilerExtensionVersion</span> <span class="p">=</span> <span class="s">"1.4.8"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Then we added the essential Compose libraries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="c1">// Compose BOM</span>
<span class="n">implementation</span> <span class="nf">platform</span><span class="p">(</span><span class="s">"androidx.compose:compose-bom:2025.02.00"</span><span class="p">)</span>

<span class="c1">// Core UI</span>
<span class="n">implementation</span> <span class="s">"androidx.compose.ui:ui"</span>
<span class="n">implementation</span> <span class="s">"androidx.compose.ui:ui-tooling-preview"</span>
<span class="n">implementation</span> <span class="s">"androidx.compose.material3:material3"</span>

<span class="c1">// Runtime</span>
<span class="n">implementation</span> <span class="s">"androidx.activity:activity-compose:1.10.1"</span>

<span class="c1">// For debugging previews</span>
<span class="n">debugImplementation</span> <span class="s">"androidx.compose.ui:ui-tooling"</span>
</code></pre>

</div>



<h3>
  
  
  🧨 Why version <code>1.4.8</code>?
</h3>

<p>Because if you go any higher while using Kotlin <code>1.8.22</code> (which Flutter ships with), Compose will politely scream at you. We initially tried newer versions, only to be met with this charming message:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>This version of the Compose Compiler requires Kotlin version 1.9.23 but you are using 1.8.22 which is not known to be compatible…
</code></pre>

</div>



<p>📌 Check the <a href="https://developer.android.com/jetpack/androidx/releases/compose-kotlin" rel="noopener noreferrer">Compose-Kotlin compatibility table</a> for more details.</p>

<p>Then it hits you with the legendary line:</p>

<blockquote>
<p>"…or suppressKotlinVersionCompatibilityCheck but don’t say I didn’t warn you!"</p>
</blockquote>

<p>Yeah. Compose isn’t messing around 😅.</p>

<p>So we locked it down to 1.4.8, which plays nicely with the Kotlin version Flutter currently ships.</p>




<h3>
  
  
  Replacing the Default: Building Our Custom In-App Renderer
</h3>

<p>Firebase tells us: implement <code>FirebaseInAppMessagingDisplay</code> and register it. That’s all. No examples, no docs for Compose. Just vibes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="nc">FirebaseInAppMessaging</span><span class="p">.</span><span class="nf">getInstance</span><span class="p">().</span><span class="nf">setMessageDisplayComponent</span><span class="p">(</span><span class="nc">NologyInAppMessageDisplay</span><span class="p">(</span><span class="n">context</span><span class="p">))</span>
</code></pre>

</div>



<p>Let's walk through the custom implementation we built — clean, robust, and fully integrated with your Compose UI.</p>

<h3>
  
  
  🧠 The Heart of the Custom FIAM: <code>NologyInAppMessageDisplay</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="kd">class</span> <span class="nc">NologyInAppMessageDisplay</span><span class="p">(</span><span class="k">private</span> <span class="kd">val</span> <span class="py">context</span><span class="p">:</span> <span class="nc">Context</span><span class="p">)</span> <span class="p">:</span> <span class="nc">FirebaseInAppMessagingDisplay</span> <span class="p">{</span>
    <span class="k">override</span> <span class="k">fun</span> <span class="nf">displayMessage</span><span class="p">(</span>
        <span class="n">inAppMessage</span><span class="p">:</span> <span class="nc">InAppMessage</span><span class="p">,</span>
        <span class="n">callbacks</span><span class="p">:</span> <span class="nc">FirebaseInAppMessagingDisplayCallbacks</span>
    <span class="p">)</span> <span class="p">{</span>
        <span class="c1">// Implementation logic</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>✅ <strong>What This Class Does:</strong></p>

<ul>
<li>Confirms message type: Only supports <code>ModalMessage</code>.</li>
<li>Loads remote image manually (unlike iOS).</li>
<li>Extracts button styles and parses <code>appData</code> for optional buttons.</li>
<li>Wraps the Composable inside a <code>DialogFragment</code>.</li>
<li>Handles actions and calls <code>messageClicked()</code> / <code>messageDismissed()</code>.</li>
</ul>




<h2>
  
  
  🔍 Platform Differences: iOS vs Android
</h2>

<h3>
  
  
  📸 Image Handling
</h3>

<ul>
<li>
<strong>iOS:</strong> Uses <code>imageRawData</code> directly, no network call required.</li>
<li>
<strong>Android:</strong> Must manually download image from <code>imageUrl</code>.</li>
</ul>

<h3>
  
  
  🔗 Deep Linking
</h3>

<ul>
<li>
<strong>iOS:</strong> Firebase automatically opens links.</li>
<li>
<strong>Android:</strong> Must manually launch an <code>Intent.ACTION_VIEW</code> with the URL.</li>
</ul>




<h2>
  
  
  🧪 How to Test Custom In-App Messages
</h2>

<p>Firebase caches campaigns for up to 24 hours — but test mode lets you bypass that using a device’s <strong>FID (Firebase Installation ID)</strong>.</p>

<h3>
  
  
  Step 1: Get the FID
</h3>

<p><strong>On Android:</strong><br>
Look for this in Logcat:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>I/FIAM.Headless: Starting InAppMessaging runtime with Installation ID &lt;YOUR_ID&gt;
</code></pre>

</div>



<p><strong>On iOS:</strong><br>
Go to Xcode → Product &gt; Scheme &gt; Edit Scheme…<br>
Add argument: <code>-FIRDebugEnabled</code><br>
Then find in logs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[Firebase/InAppMessaging] Starting InAppMessaging runtime with Firebase Installation ID &lt;YOUR_ID&gt;
</code></pre>

</div>



<h3>
  
  
  Step 2: Send a Test Message
</h3>

<ol>
<li>Go to Firebase → In-App Messaging → New Campaign</li>
<li>Select <strong>Modal</strong> layout (⚠️ Only one supported by custom renderer)</li>
<li>Add <strong>custom data</strong>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"secondaryTitle"</span><span class="p">:</span><span class="w"> </span><span class="s2">"SKIP"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"secondaryTextColor"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#000000"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"secondaryBackgroundColor"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#C9C9C9"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<ol>
<li>Click <strong>Test on your device</strong>
</li>
<li>Paste the FID</li>
<li>Reopen app — message should appear instantly 💥</li>
</ol>




<h2>
  
  
  🔗 About Deep Links?
</h2>

<p>Setting up proper deep link support is a big deal — and deserves its own article.</p>

<p>I’ll cover everything in a future piece:</p>

<ul>
<li>App links &amp; Universal Links</li>
<li>Domain verification</li>
<li>JSON files for each platform</li>
<li>Mapping paths into Flutter navigation</li>
</ul>

<p>If that sounds helpful — drop a comment and I’ll prioritize it!</p>




<h2>
  
  
  ✅ Wrapping Up
</h2>

<p>Building custom layouts for Firebase In-App Messaging isn’t just about styling — it’s about creating immersive, native, analytics-driven experiences that match your app’s personality and performance goals.</p>

<p>From deep link support to analytics tracking, we stayed close to Firebase’s native protocols while still delivering a UI that feels ours.</p>




<h2>
  
  
  🙋‍♂️ Who’s Behind This?
</h2>

<p>I’m <strong>Salah Nahed</strong>, a Flutter &amp; mobile craftsman passionate about building beautiful cross-platform experiences.</p>

<ul>
<li>📦 Creator of <a href="https://github.com/salahamassi" rel="noopener noreferrer">Bond</a>
</li>
<li>💼 Available for consulting &amp; freelancing</li>
<li>🧠 Follow me for more dev stories</li>
</ul>

<p><strong>Let’s connect:</strong></p>

<ul>
<li>🔗 <a href="https://www.linkedin.com/in/salahamassi" rel="noopener noreferrer">LinkedIn</a>
</li>
<li>🐦 <a href="https://twitter.com/salahamassi" rel="noopener noreferrer">Twitter/X</a>
</li>
<li>🐙 <a href="https://github.com/salahamassi" rel="noopener noreferrer">GitHub</a>
</li>
<li>📸 <a href="https://instagram.com/salahamassi" rel="noopener noreferrer">Instagram</a>
</li>
</ul>




<p>Thanks for reading — and see you in the next deep dive ✌️</p>

