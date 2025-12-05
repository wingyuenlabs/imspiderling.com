---
Title: How I Reduced My React Native 0.81 APK Size by 50% (Without Breaking Anything)
Description: 
Author: Ajmal Hasan
Date: 2025-12-05T21:38:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>A large APK size can slow down downloads, increase install drop-offs, and impact user experience — especially in regions with slower internet connections.</p>

<p>In this post, I'll walk you through the <strong>exact steps</strong> I used to reduce a production React Native 0.81 app's APK size by ~50%, while maintaining full functionality and security.</p>




<h2>
  
  
  📊 The Results
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Before</th>
<th>After</th>
<th>Reduction</th>
</tr>
</thead>
<tbody>
<tr>
<td>~85 MB</td>
<td>~42 MB</td>
<td><strong>~50%</strong></td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  🛠️ Step 1: Build Only for arm64-v8a Architecture
</h2>

<p>Most optimization guides tell you to split APKs by CPU architecture. But here's the thing: <strong>if your minSdkVersion is 29+ (Android 10+), you only need arm64-v8a</strong>.</p>

<p>Why? Because:</p>

<ul>
<li>✅ All Android 10+ devices support 64-bit (arm64-v8a)</li>
<li>✅ Google Play Store requires 64-bit support</li>
<li>✅ 95%+ of active Android devices use arm64-v8a</li>
<li>❌ armeabi-v7a is for 32-bit devices (pre-2014)</li>
<li>❌ x86/x86_64 is for Intel devices (&lt;1% market share)</li>
</ul>

<h3>
  
  
  Implementation
</h3>

<p>In your <code>android/app/build.gradle</code>:</p>

<p>// Enable separate builds per CPU architecture<br>
def enableSeparateBuildPerCPUArchitecture = true</p>

<p>android {<br>
    // ... existing config ...</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>splits {
    abi {
        reset()
        enable enableSeparateBuildPerCPUArchitecture
        universalApk false  // No universal APK needed
        include "arm64-v8a"  // Only modern 64-bit devices
    }
}
</code></pre>

</div>

<p>}💡 <strong>Pro tip</strong>: Set <code>universalApk false</code> to skip building a fat universal APK — it's unnecessary when targeting a single architecture.</p>




<h2>
  
  
  🌍 Step 2: Keep Only Necessary Languages
</h2>

<p>Your app probably only supports a few languages, but by default, Android bundles resources for <strong>ALL</strong> languages from your dependencies (like Google Play Services).</p>

<h3>
  
  
  Implementation
</h3>

<p>In <code>android/app/build.gradle</code>, inside <code>defaultConfig</code>:</p>

<p>defaultConfig {<br>
    // ... existing config ...</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Keep only necessary languages to reduce APK size
resConfigs "ar", "en", "fr"  // Add your supported languages
</code></pre>

</div>

<p>}This single line can remove <strong>several megabytes</strong> of unused language resources!</p>




<h2>
  
  
  📦 Step 3: Enable Resource Shrinking &amp; PNG Compression
</h2>

<p>Enable these in your release build type:</p>

<p>buildTypes {<br>
    release {<br>
        signingConfig signingConfigs.release<br>
        debuggable false<br>
        minifyEnabled true          // Code shrinking (already enabled)<br>
        shrinkResources true        // Remove unused resources<br>
        crunchPngs true             // Extra PNG compression<br>
        zipAlignEnabled true        // Memory optimization<br>
        proguardFiles getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro"<br>
    }<br>
}### What Each Does:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Option</th>
<th>Benefit</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>minifyEnabled true</code></td>
<td>Removes unused code, obfuscates names</td>
</tr>
<tr>
<td><code>shrinkResources true</code></td>
<td>Removes unused drawables, layouts, etc.</td>
</tr>
<tr>
<td><code>crunchPngs true</code></td>
<td>Additional PNG compression</td>
</tr>
<tr>
<td><code>zipAlignEnabled true</code></td>
<td>Optimizes APK for memory-mapped reading</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  🔐 Step 4: Optimize ProGuard Rules
</h2>

<p>ProGuard (or R8) is powerful but can break your app if not configured correctly. Here's a comprehensive, battle-tested configuration for React Native 0.81:</p>

<h3>
  
  
  <code>android/app/proguard-rules.pro</code>
</h3>

<p>guard</p>

<h1>
  
  
  ========== STANDARD OPTIMIZATIONS ==========
</h1>

<p>-optimizationpasses 3<br>
-allowaccessmodification<br>
-overloadaggressively<br>
-repackageclasses ''<br>
-dontusemixedcaseclassnames<br>
-dontpreverify</p>

<h1>
  
  
  Remove debug logging in production (keeps error logs)
</h1>

<p>-assumenosideeffects class android.util.Log {<br>
    public static *** d(...);<br>
    public static *** v(...);<br>
    public static *** i(...);<br>
    public static *** w(...);<br>
}</p>

<h1>
  
  
  Keep crash reporting info
</h1>

<p>-keepattributes SourceFile,LineNumberTable<br>
-renamesourcefileattribute SourceFile<br>
-keepattributes Signature, <em>Annotation</em></p>

<h1>
  
  
  ========== REACT NATIVE CORE ==========
</h1>

<p>-keep class com.facebook.hermes.** { <em>; }<br>
-keep class com.facebook.hermes.unicode.</em>* { <em>; }<br>
-keep class com.facebook.react.</em>* { <em>; }<br>
-keep class com.facebook.jni.</em>* { <em>; }<br>
-keep class com.facebook.soloader.</em>* { *; }</p>

<h1>
  
  
  Bridge &amp; Native Modules
</h1>

<p>-keep public class * extends com.facebook.react.bridge.NativeModule { <em>; }<br>
-keep class com.facebook.react.bridge.</em>* { *; }<br>
-keepclassmembers class * { <br>
    <a class="mentioned-user" href="https://dev.to/com">@com</a>.facebook.react.bridge.ReactMethod *; <br>
}<br>
-keepclassmembers class * {<br>
    <a class="mentioned-user" href="https://dev.to/com">@com</a>.facebook.proguard.annotations.DoNotStrip *;<br>
}</p>

<h1>
  
  
  UI Manager &amp; View System
</h1>

<p>-keep <a class="mentioned-user" href="https://dev.to/com">@com</a>.facebook.react.uimanager.annotations.ReactProp class * { <em>; }<br>
-keep <a class="mentioned-user" href="https://dev.to/com">@com</a>.facebook.react.uimanager.annotations.ReactPropGroup class * { *; }<br>
-keep class * extends com.facebook.react.uimanager.ViewManager { *; }<br>
-keep class com.facebook.react.uimanager.</em>* { *; }</p>

<h1>
  
  
  TurboModules (New Architecture)
</h1>

<p>-keep class com.facebook.react.turbomodule.** { <em>; }<br>
-keep class com.facebook.react.animated.</em>* { <em>; }<br>
-keep class com.facebook.react.common.</em>* { *; }</p>

<h1>
  
  
  ========== COMMON LIBRARIES ==========
</h1>

<h1>
  
  
  Reanimated
</h1>

<p>-keep class com.swmansion.reanimated.** { <em>; }<br>
-keep class com.swmansion.</em>* { <em>; }<br>
-dontwarn com.swmansion.</em>*</p>

<h1>
  
  
  SVG
</h1>

<p>-keep public class com.horcrux.svg.** { *; }</p>

<h1>
  
  
  Fast Image
</h1>

<p>-keep class com.dylanvann.fastimage.** { *; }</p>

<h1>
  
  
  Firebase
</h1>

<p>-keep class com.google.firebase.** { <em>; }<br>
-dontwarn com.google.firebase.</em>*</p>

<h1>
  
  
  OkHttp
</h1>

<p>-keep class okhttp3.** { <em>; }<br>
-keep interface okhttp3.</em>* { <em>; }<br>
-dontwarn okhttp3.</em>*<br>
-dontwarn okio.**### Why This Works:</p>

<ol>
<li>
<strong><code>-optimizationpasses 3</code></strong> — Multiple optimization passes for maximum shrinkage</li>
<li>
<strong>Log removal</strong> — Strips debug/verbose/info/warning logs (keeps errors for debugging)</li>
<li>
<strong><code>-keepattributes SourceFile,LineNumberTable</code></strong> — Preserves stack traces for crash reporting</li>
<li>
<strong>Comprehensive <code>-keep</code> rules</strong> — Protects all React Native internals and common libraries</li>
</ol>




<h2>
  
  
  ✅ Step 5: Verify Hermes is Enabled
</h2>

<p>Hermes is React Native's optimized JavaScript engine. It should be enabled by default in RN 0.81, but verify in your <code>android/gradle.properties</code>:</p>

<p>hermesEnabled=trueHermes provides:</p>

<ul>
<li>✅ Faster app startup</li>
<li>✅ Smaller JS bundle size</li>
<li>✅ Lower memory usage</li>
</ul>




<h2>
  
  
  🧪 Testing Your Build
</h2>

<p>After applying all optimizations:</p>

<h1>
  
  
  Clean previous builds
</h1>

<p>cd android &amp;&amp; ./gradlew clean</p>

<h1>
  
  
  Build release APK
</h1>

<p>./gradlew assembleRelease</p>

<h1>
  
  
  Check APK size
</h1>

<p>ls -lh app/build/outputs/apk/<em>/release/</em>.apk---</p>

<h2>
  
  
  📋 Complete Checklist
</h2>

<ul>
<li>[ ] Set <code>enableSeparateBuildPerCPUArchitecture = true</code>
</li>
<li>[ ] Include only <code>arm64-v8a</code> architecture (if minSdk 29+)</li>
<li>[ ] Set <code>universalApk false</code>
</li>
<li>[ ] Add <code>resConfigs</code> for your supported languages</li>
<li>[ ] Enable <code>shrinkResources true</code>
</li>
<li>[ ] Enable <code>crunchPngs true</code>
</li>
<li>[ ] Enable <code>zipAlignEnabled true</code>
</li>
<li>[ ] Add comprehensive ProGuard rules</li>
<li>[ ] Verify Hermes is enabled</li>
<li>[ ] Test on real devices before release!</li>
</ul>




<h2>
  
  
  🎯 Summary
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Optimization</th>
<th>APK Size Impact</th>
</tr>
</thead>
<tbody>
<tr>
<td>Single architecture (arm64-v8a)</td>
<td><strong>-30-40%</strong></td>
</tr>
<tr>
<td>Resource shrinking</td>
<td><strong>-5-10%</strong></td>
</tr>
<tr>
<td>Language filtering</td>
<td><strong>-2-5%</strong></td>
</tr>
<tr>
<td>ProGuard optimizations</td>
<td><strong>-5-10%</strong></td>
</tr>
<tr>
<td>PNG compression</td>
<td><strong>-1-3%</strong></td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  ⚠️ Important Notes
</h2>

<ol>
<li>
<strong>Always test on real devices</strong> after applying these optimizations</li>
<li>
<strong>Keep error logs</strong> — don't strip <code>Log.e()</code> for production debugging</li>
<li>
<strong>Firebase Crashlytics still works</strong> — we preserved <code>SourceFile,LineNumberTable</code>
</li>
<li>
<strong>Play Store accepts single-architecture APKs</strong> — they'll deliver the right one to users</li>
</ol>




<p>Hope this helps you ship a lighter, faster React Native app! 🚀</p>

<p>Got questions or improvements? Drop them in the comments! 👇</p>




<p><em>Found this useful? Follow me for more React Native optimization tips!</em></p>

