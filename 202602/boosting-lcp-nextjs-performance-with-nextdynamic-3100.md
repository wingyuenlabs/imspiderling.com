---
Title: Boosting LCP: Next.js Performance with next/dynamic
Description: 
Author: Atena / Full stack developer
Date: 2026-02-12T21:37:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every frontend engineer knows the pain. Your team keeps requesting you to add new features to the website, and suddenly your LCP score takes a nose dive.</p>

<p>This is exactly where <code>next/dynamic</code> saves the day. I’ve used this method to rescue performance scores. </p>

<h2>
  
  
  1. What is <code>next/dynamic</code>?
</h2>

<p>It’s a tool for <strong>Lazy Loading your React components</strong>. When you do a standard import like import Header from <code>'./Header'</code>, that component gets bundled into the main JavaScript file. This means the browser has to download, parse, and execute that code before the user can really interact with the page. This kills your LCP score. <br>
<code>next/dynamic</code> lets you tell the bundler: "Don't load this yet. Wait until we actually need it." It splits the code into a separate "chunks" file.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80g2ip0wn04yzbnsbhqa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80g2ip0wn04yzbnsbhqa.png" alt="Explanation of next/dynamic" width="800" height="436"></a></p>
<h2>
  
  
  2. Two Strategies for Using It
</h2>

<p>There are two main ways I use <code>next/dynamic</code>, depending on what I'm trying to fix.</p>
<h3>
  
  
  Strategy 1: Optimizing for Timing (Fixing LCP)
</h3>

<p>This is for things the user doesn't see immediately, like heavy components or a section way down the page. By lazy loading these, you let the browser focus 100% of its energy on rendering the Hero section. This makes the perceived load time much faster.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const HeavyComponent = dynamic(() =&gt; import('../components/Heavy'),
 {
  loading: () =&gt; &lt;p&gt;Loading...&lt;/p&gt;,
 })
</code></pre>

</div>



<h3>
  
  
  Strategy 2: Optimizing for Location (Server vs. Browser)
</h3>

<p>Sometimes, you have code that simply cannot run on the server. Maybe it uses window, document, or a library. By using the <code>ssr: false</code> option, you force the component to be rendered entirely on the client side.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const MapComponent = dynamic(() =&gt; import('../components/Map'),
 {
  ssr: false, // Skip the server entirely!
 })
</code></pre>

</div>



<h2>
  
  
  3. <code>next/dynamic</code> vs. <code>'use client'</code>
</h2>

<p>This is where I got confused. If I import a component with <code>ssr: true</code>, but that component has <code>'use client'</code>, does that make it CSR?</p>

<p>The answer is No. It is still SSR first.<br>
To clear up this confusion, we need to correct our mental model.</p>

<h3>
  
  
  Method vs. Declaration
</h3>

<p>We must distinguish between the Method and the Declaration. </p>

<ul>
<li>SSR/CSR is a Method. It describes when and where the rendering happens.</li>
<li>
<code>'use client'</code> is a Declaration. It signals what features (like hooks or event listeners) are being used.</li>
</ul>

<p>It can be confusing that adding <code>'use client'</code> opts out of SSR. This is wrong. In the Next.js App Router, even if you mark a component with <code>'use client'</code>, it is still rendered on the server initially.</p>

<p>Then how does the  <code>'use client'</code> work with SSR/ CSR?</p>

<ul>
<li><p>SSR with <code>'use client'</code><br>
The server pre-renders the HTML Shell (the tags like <code>&lt;button&gt;</code> inside the server box).  The <code>'use client'</code> acts as <strong>Hydration</strong>  to make the existing button interactive after it arrives in the browser. The HTML is born on the Server. The user sees the content immediately.</p></li>
<li><p>CSR with <code>'use client'</code><br>
The server skips rendering and sends an Empty Shell. The server skips rendering and sends an Empty Shell. Here, the JavaScript bundle must build the entire UI from scratch inside the <strong>Browser</strong>. The user waits for the JS to load before seeing any content.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbc7a54h8lrnu393bdvui.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbc7a54h8lrnu393bdvui.png" alt="Explanation of CSR vs SSR with use cleint" width="800" height="436"></a></p>

<p>Performance is all about control.You need to know what is rendering, where it's rendering, and when.</p>

<p>Want to improve LCP?<br>
→ Use next/dynamic to split up your heavy JS bundles.</p>

<p>Fixing a "window is undefined" error?<br>
→ Use ssr: false to skip the server execution entirely.</p>

<p>Need a button to click?<br>
→ Use 'use client' to add interactivity (while keeping SSR).</p>

<p>But remember, with<code>'use client'</code>, the HTML is still born on the server. Learning this distinction is the key to building Next.js apps that feel instant.</p>

