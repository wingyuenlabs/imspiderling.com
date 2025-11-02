---
Title: The Siren's Call of Async/Await: A Journey Beyond `try/catch`
Description: 
Author: Alex Aslam
Date: 2025-11-02T22:00:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>There's a certain artistry to writing asynchronous code in modern Node.js. We've journeyed far from the callback pyramids of old, navigated the limbo of <code>Promise.then().catch()</code> chains, and finally found what felt like the promised land: <code>async/await</code>.</p>

<p>It reads like synchronous code. It's beautiful. It's clean. We can almost forget we're dealing with the non-blocking, event-driven heart of Node.</p>

<p>But this beauty is a siren's call. It tempts us to ignore the jagged rocks of reality lurking just beneath the surface—the rocks of <strong>errors</strong>.</p>

<p>My fellow senior developer, let's take a journey. Let's not treat this as a lecture, but as a critique of an artwork we're all composing: our codebase.</p>

<h3>
  
  
  Act I: The Illusion of Control
</h3>

<p>When we first embraced <code>async/await</code>, we were diligent. We built sturdy, if a little clumsy, fortresses around our promises.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// The Diligent Novice's Art</span>
<span class="kd">const</span> <span class="nx">fetchUserData</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">userId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">User</span><span class="p">.</span><span class="nf">findById</span><span class="p">(</span><span class="nx">userId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">posts</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Post</span><span class="p">.</span><span class="nf">find</span><span class="p">({</span> <span class="na">author</span><span class="p">:</span> <span class="nx">userId</span> <span class="p">});</span>
    <span class="kd">const</span> <span class="nx">avatar</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">uploadAvatar</span><span class="p">(</span><span class="nx">user</span><span class="p">.</span><span class="nx">avatarUrl</span><span class="p">);</span> <span class="c1">// A potential failure point</span>
    <span class="k">return</span> <span class="p">{</span> <span class="nx">user</span><span class="p">,</span> <span class="nx">posts</span><span class="p">,</span> <span class="nx">avatar</span> <span class="p">};</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Oh no! Something went wrong:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
    <span class="c1">// But what *specifically* went wrong? And what do we do now?</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>This code has good intentions. It's defensive. But it's a blunt instrument. An error in <code>User.findById</code> is treated with the same brush as an error in <code>uploadAvatar</code>. The user gets a generic "Something went wrong," and we, the developers, get a cryptic log line. Our application state is a mystery. Did the user get created without posts? Was the avatar upload half-complete?</p>

<p>This is the first stage of our journey: recognizing that a single, monolithic <code>try/catch</code> is like using a single color to paint an entire landscape. It captures the scene but loses all the detail.</p>

<h3>
  
  
  Act II: The Descent into Silence
</h3>

<p>Then, we get clever. We think, "I'm a senior dev. I know this function is mostly safe. That <code>thirdPartyAPI.call()</code> is robust, I don't need to clutter the code."</p>

<p>So we write this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// The Siren's Song of "Clean Code"</span>
<span class="kd">const</span> <span class="nx">processOrder</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">orderId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Order</span><span class="p">.</span><span class="nf">findById</span><span class="p">(</span><span class="nx">orderId</span><span class="p">);</span> <span class="c1">// No try/catch</span>
  <span class="nx">order</span><span class="p">.</span><span class="nx">status</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">processing</span><span class="dl">'</span><span class="p">;</span>
  <span class="k">await</span> <span class="nx">order</span><span class="p">.</span><span class="nf">save</span><span class="p">();</span> <span class="c1">// No try/catch</span>

  <span class="c1">// This is the bomb that never ticks... until it does.</span>
  <span class="k">await</span> <span class="nx">thirdPartyShippingAPI</span><span class="p">.</span><span class="nf">notify</span><span class="p">(</span><span class="nx">order</span><span class="p">);</span> <span class="c1">// No try/catch</span>

  <span class="k">await</span> <span class="nx">EmailService</span><span class="p">.</span><span class="nf">sendConfirmation</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">userEmail</span><span class="p">,</span> <span class="nx">order</span><span class="p">);</span> <span class="c1">// No try/catch</span>
  <span class="nx">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="s2">`Order </span><span class="p">${</span><span class="nx">orderId</span><span class="p">}</span><span class="s2"> processed successfully`</span><span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<p>From a distance, this code is a masterpiece of minimalism. It's pure logic, unburdened by the messiness of reality. But it's a house of cards.</p>

<p>When <code>thirdPartyShippingAPI.notify()</code> fails—and it will—what happens?</p>

<ul>
<li>The promise rejects.</li>
<li>The <code>processOrder</code> function throws.</li>
<li><strong>The error bubbles up to the event loop.</strong></li>
<li>If unhandled, it crashes your Node.js process.</li>
</ul>

<p>A single, transient network blip in a non-critical service can now take down your entire application. This isn't clean code; it's a denial of the distributed, unreliable nature of the systems we build. We've painted over the cracks in the canvas, hoping no one will notice.</p>

<h3>
  
  
  Interlude: The Art of Intentional Failure
</h3>

<p>A true master doesn't ignore failure; they incorporate it into their design. Error handling isn't a defensive chore; it's a core feature of your application's logic. It's the shadows in a painting that give the light meaning.</p>

<p>We need to move from <em>ignoring</em> errors to <em>orchestrating</em> our response to them.</p>

<h3>
  
  
  Act III: The Master's Palette - Composition and Strategy
</h3>

<p>As senior developers, our goal isn't to just prevent crashes. It's to build systems that are <strong>resilient</strong> and <strong>observable</strong>. This requires a more nuanced palette of techniques.</p>

<h4>
  
  
  Strategy 1: The Functional Wrapper (<code>handle</code>)
</h4>

<p>Instead of repetitive <code>try/catch</code> blocks, we can compose our async functions with a wrapper that structures the outcome.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// A stroke of functional genius</span>
<span class="kd">const</span> <span class="nx">handle</span> <span class="o">=</span> <span class="p">(</span><span class="nx">promise</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">promise</span><span class="p">.</span><span class="nf">then</span><span class="p">((</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="kc">null</span><span class="p">,</span> <span class="nx">data</span><span class="p">]).</span><span class="k">catch</span><span class="p">((</span><span class="nx">error</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">[</span><span class="nx">error</span><span class="p">,</span> <span class="kc">null</span><span class="p">]);</span>

<span class="kd">const</span> <span class="nx">updateUserProfile</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">userId</span><span class="p">,</span> <span class="nx">updates</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="nx">error</span><span class="p">,</span> <span class="nx">user</span><span class="p">,</span> <span class="nx">avatar</span><span class="p">;</span>

  <span class="p">[</span><span class="nx">error</span><span class="p">,</span> <span class="nx">user</span><span class="p">]</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">handle</span><span class="p">(</span><span class="nx">User</span><span class="p">.</span><span class="nf">findByIdAndUpdate</span><span class="p">(</span><span class="nx">userId</span><span class="p">,</span> <span class="nx">updates</span><span class="p">,</span> <span class="p">{</span> <span class="na">new</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}));</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nf">logError</span><span class="p">(</span><span class="dl">'</span><span class="s1">DB_UPDATE_FAILED</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">,</span> <span class="p">{</span> <span class="nx">userId</span> <span class="p">});</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">ApplicationError</span><span class="p">(</span><span class="dl">'</span><span class="s1">Failed to update user profile</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">updates</span><span class="p">.</span><span class="nx">avatarUrl</span><span class="p">)</span> <span class="p">{</span>
    <span class="p">[</span><span class="nx">error</span><span class="p">,</span> <span class="nx">avatar</span><span class="p">]</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">handle</span><span class="p">(</span><span class="nx">ImageProcessor</span><span class="p">.</span><span class="nf">uploadAndResize</span><span class="p">(</span><span class="nx">updates</span><span class="p">.</span><span class="nx">avatarUrl</span><span class="p">));</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// We failed to upload the avatar, but the user was updated!</span>
      <span class="c1">// This is a partial failure. How do we handle it?</span>
      <span class="k">await</span> <span class="nf">logError</span><span class="p">(</span><span class="dl">'</span><span class="s1">AVATAR_UPLOAD_FAILED</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">,</span> <span class="p">{</span> <span class="nx">userId</span> <span class="p">});</span>
      <span class="k">await</span> <span class="nx">EmailService</span><span class="p">.</span><span class="nf">sendAlert</span><span class="p">(</span><span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span> <span class="dl">'</span><span class="s1">Your profile was updated, but we failed to process your new avatar.</span><span class="dl">'</span><span class="p">);</span>
      <span class="c1">// We do NOT throw. The core operation was successful.</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">user</span><span class="p">,</span> <span class="nx">avatar</span> <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>



<p>This approach gives us fine-grained control. We can handle errors where they occur, make decisions about partial success, and never lose context.</p>

<h4>
  
  
  Strategy 2: The <code>catch</code> Method - A Deliberate Brushstroke
</h4>

<p>Sometimes, you can recover from an error immediately. In such cases, attaching a <code>.catch()</code> to a non-critical promise is an elegant solution.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">fulfillOrder</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">order</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Critical: Must not fail.</span>
  <span class="k">await</span> <span class="nx">InventoryService</span><span class="p">.</span><span class="nf">decrementStock</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">lineItems</span><span class="p">);</span>

  <span class="c1">// Non-critical: Should not block fulfillment.</span>
  <span class="nx">Analytics</span><span class="p">.</span><span class="nf">track</span><span class="p">(</span><span class="dl">'</span><span class="s1">order_fulfilled</span><span class="dl">'</span><span class="p">,</span> <span class="nx">order</span><span class="p">)</span>
    <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">analyticsError</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// We log it for later investigation, but the order continues.</span>
      <span class="nx">logger</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="dl">'</span><span class="s1">Analytics flush failed</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">error</span><span class="p">:</span> <span class="nx">analyticsError</span><span class="p">,</span> <span class="na">orderId</span><span class="p">:</span> <span class="nx">order</span><span class="p">.</span><span class="nx">id</span> <span class="p">});</span>
    <span class="p">});</span>

  <span class="c1">// Also non-critical: Fire-and-forget email.</span>
  <span class="nx">EmailService</span><span class="p">.</span><span class="nf">sendReceipt</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">userEmail</span><span class="p">,</span> <span class="nx">order</span><span class="p">)</span>
    <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">emailError</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">logger</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="dl">'</span><span class="s1">Receipt email failed</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">error</span><span class="p">:</span> <span class="nx">emailError</span> <span class="p">}));</span>

  <span class="c1">// The main function fulfills regardless of the non-critical tasks.</span>
  <span class="k">await</span> <span class="nf">markOrderAsFulfilled</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<p>This is intentional. We are declaring, "This operation is best-effort. Its failure should not impact the core journey."</p>

<h4>
  
  
  Strategy 3: The Architectural Frame - Express Error-Handling Middleware
</h4>

<p>At the boundaries of your application, you need a frame to catch everything that slips through. In an Express.js app, this is your final safety net.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// The final, unifying frame for your artwork</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">err</span><span class="p">,</span> <span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Log the error with a structured logger</span>
  <span class="nx">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Unhandled Error</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">message</span><span class="p">:</span> <span class="nx">err</span><span class="p">.</span><span class="nx">message</span><span class="p">,</span>
    <span class="na">stack</span><span class="p">:</span> <span class="nx">err</span><span class="p">.</span><span class="nx">stack</span><span class="p">,</span>
    <span class="na">path</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">path</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">method</span><span class="p">,</span>
    <span class="na">userId</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">user</span><span class="p">?.</span><span class="nx">id</span>
  <span class="p">});</span>

  <span class="c1">// Check if it's an error we expect</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">err</span> <span class="k">instanceof</span> <span class="nx">ValidationError</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Invalid input</span><span class="dl">'</span><span class="p">,</span> <span class="na">details</span><span class="p">:</span> <span class="nx">err</span><span class="p">.</span><span class="nx">details</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">err</span> <span class="k">instanceof</span> <span class="nx">AuthenticationError</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">401</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Please log in</span><span class="dl">'</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Default: Don't leak internals!</span>
  <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">An internal server error occurred</span><span class="dl">'</span> <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p>This middleware is the gallery guard. It ensures that no unhandled error, no matter how deep in your async call stack, ever brings down the server or returns an unformatted stack trace to the user.</p>

<h3>
  
  
  The Finished Masterpiece
</h3>

<p>Ignoring error handling in <code>async/await</code> is like an artist ignoring the laws of perspective. You might create something that looks interesting for a moment, but it will never stand the test of time.</p>

<p>The true masterpiece—the codebase that is resilient, observable, and maintainable—is one where error handling is an integral part of the composition. It's not an afterthought; it's the careful application of shadow and light, the strategic use of a functional wrapper, the deliberate suppression of a non-critical task, and the strong, reliable frame of a top-level handler.</p>

<p>So, the next time you write <code>await</code>, pause. Ask yourself: "What is the story of failure here? And how shall my code tell it?"</p>

<p>Happy coding, artist.</p>

