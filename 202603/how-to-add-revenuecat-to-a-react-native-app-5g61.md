---
Title: How to Add RevenueCat to a React Native App
Description: 
Author: Jay-F Nnadi
Date: 2026-03-08T21:47:20.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  How to Add RevenueCat to a React Native App
</h1>

<p>RevenueCat can remove a lot of subscription plumbing, but most teams still lose time on the first working integration. The fastest path is to wire a minimal client that can:</p>

<ul>
<li>initialize the SDK correctly</li>
<li>fetch the current offering</li>
<li>purchase the first available package</li>
<li>restore purchases</li>
<li>inspect active entitlements</li>
</ul>

<p>This walkthrough matches the runnable Luna demo in <code>projects/revenuecat-react-native-example</code>.</p>

<h2>
  
  
  1. Install the SDK
</h2>

<p>Use the React Native RevenueCat package in your app:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>react-native-purchases
</code></pre>

</div>



<p>If you are using Expo with native modules, make sure your project is configured for the native build flow required by the package.</p>

<h2>
  
  
  2. Add platform API keys
</h2>

<p>RevenueCat uses different public SDK keys per platform. In the Luna demo, the app reads:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>EXPO_PUBLIC_REVENUECAT_IOS_API_KEY=appl_public_sdk_key
EXPO_PUBLIC_REVENUECAT_ANDROID_API_KEY=goog_public_sdk_key
EXPO_PUBLIC_REVENUECAT_WEB_API_KEY=web_public_sdk_key
</code></pre>

</div>



<h2>
  
  
  3. Configure RevenueCat on app startup
</h2>

<p>Initialize the SDK once and immediately fetch offerings and customer info:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="nx">Purchases</span><span class="p">.</span><span class="nf">setLogLevel</span><span class="p">(</span><span class="nx">LOG_LEVEL</span><span class="p">.</span><span class="nx">DEBUG</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">Purchases</span><span class="p">.</span><span class="nf">configure</span><span class="p">({</span> <span class="nx">apiKey</span> <span class="p">});</span>
<span class="kd">const</span> <span class="nx">offerings</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Purchases</span><span class="p">.</span><span class="nf">getOfferings</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">info</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Purchases</span><span class="p">.</span><span class="nf">getCustomerInfo</span><span class="p">();</span>
</code></pre>

</div>



<p>That gives you the two most important building blocks:</p>

<ul>
<li>the current paywall/offering to display</li>
<li>the current entitlement state for the signed-in user</li>
</ul>

<h2>
  
  
  4. Purchase the current package
</h2>

<p>The simplest working purchase flow is to take the first package from the current offering:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Purchases</span><span class="p">.</span><span class="nf">purchasePackage</span><span class="p">(</span><span class="nx">offering</span><span class="p">.</span><span class="nx">availablePackages</span><span class="p">[</span><span class="mi">0</span><span class="p">]);</span>
<span class="nf">setCustomerInfo</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">customerInfo</span><span class="p">);</span>
</code></pre>

</div>



<p>In production, map packages to explicit product UI instead of assuming the first package is the one you want.</p>

<h2>
  
  
  5. Restore purchases
</h2>

<p>Every subscription UI should provide a restore path:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">info</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Purchases</span><span class="p">.</span><span class="nf">restorePurchases</span><span class="p">();</span>
<span class="nf">setCustomerInfo</span><span class="p">(</span><span class="nx">info</span><span class="p">);</span>
</code></pre>

</div>



<p>This is especially important when users change devices or reinstall the app.</p>

<h2>
  
  
  6. Inspect entitlements
</h2>

<p>Once customer info is available, active entitlements tell you what the user should be allowed to access:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">entitlements</span> <span class="o">=</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">customerInfo</span><span class="p">?.</span><span class="nx">entitlements</span><span class="p">.</span><span class="nx">active</span> <span class="o">??</span> <span class="p">{});</span>
</code></pre>

</div>



<p>That is the simplest way to drive premium UI state in a client demo.</p>

<h2>
  
  
  What developers learn from this demo
</h2>

<ul>
<li>how to initialize RevenueCat correctly in React Native</li>
<li>how to load offerings</li>
<li>how to trigger a purchase</li>
<li>how to restore purchases</li>
<li>how to inspect active entitlements</li>
</ul>

<h2>
  
  
  Why this matters
</h2>

<p>The gap for many mobile teams is not understanding RevenueCat conceptually. The gap is getting to the first working implementation fast. This example is intentionally narrow so you can get a real purchase loop working before adding account systems, paywall design, and backend coordination.</p>

