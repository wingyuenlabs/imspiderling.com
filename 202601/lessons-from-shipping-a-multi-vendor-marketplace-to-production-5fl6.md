---
Title: Lessons from Shipping a Multi-Vendor Marketplace to Production
Description: 
Author: Arbythecoder
Date: 2026-01-27T21:54:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>I recently shipped a multi-vendor marketplace platform with vendor subdomains, payment processing, and file uploads. During development and deployment, I ran into several issues that weren't obvious from tutorials or documentation. Here's what happened and how I solved them.</p>

<h2>
  
  
  The Setup
</h2>

<p>Frontend on Vercel, backend on Fly.io, vendors get their own subdomains. The architecture worked well, but production revealed edge cases I hadn't considered during local development.</p>

<h3>
  
  
  1. CORS Configuration for Wildcard Subdomains
</h3>

<p><strong>What Happened:</strong></p>

<p>Vendor subdomains like <code>vendorname.example.com</code> were getting blocked when making API requests:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Access to fetch at 'https://api-backend.fly.dev/api/vendor/products' from origin 'https://vendorname.example.com' has been blocked by CORS policy.
</code></pre>

</div>



<p>The issue was that my initial CORS setup only whitelisted the main domain, not the subdomain pattern.</p>

<p><strong>The Fix:</strong></p>

<p>Updated backend CORS to whitelist both the main domain and wildcard subdomains:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://example.com
https://*.example.com
</code></pre>

</div>



<p>Made sure <code>credentials: true</code> was set along with the necessary methods and headers.</p>

<p><strong>Takeaway:</strong></p>

<p>Dynamic subdomains need explicit CORS patterns. This is common in multi-tenant applications but easy to miss until you test in production with real subdomains.</p>

<h3>
  
  
  2. Data Structure Mismatch in Image Uploads
</h3>

<p><strong>What Happened:</strong></p>

<p>The backend schema expected <code>images: [String]</code>, but during testing I noticed the frontend was sending:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">[{</span> <span class="na">url</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span> <span class="na">publicId</span><span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span> <span class="na">isPrimary</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}]</span>
</code></pre>

</div>



<p>This caused MongoDB CastErrors that broke product creation.</p>

<p><strong>The Fix:</strong></p>

<p>Normalized the data on the frontend before sending:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">submitData</span><span class="p">.</span><span class="nx">images</span> <span class="o">=</span> <span class="nx">imageUrls</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">img</span> <span class="o">=&gt;</span> <span class="nx">img</span><span class="p">.</span><span class="nx">url</span><span class="p">).</span><span class="nf">filter</span><span class="p">(</span><span class="nb">Boolean</span><span class="p">);</span>
</code></pre>

</div>



<p>The backend image upload endpoint now returns <code>string[]</code> for database insertion while keeping richer metadata for UI state management.</p>

<p><strong>Takeaway:</strong></p>

<p>When building APIs, I've learned to validate data shape at both ends. Frontend state often needs more metadata than the database requires, so mapping before submission prevents type errors.</p>

<h3>
  
  
  3. API Response Inconsistency
</h3>

<p><strong>What Happened:</strong></p>

<p>After creating vendor profiles, the database showed correct values but the API response was returning hardcoded defaults:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"canReceiveOrders"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"isPublic"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This created confusion in the UI, showing stores as inactive when they were actually live.</p>

<p><strong>The Fix:</strong></p>

<p>Changed the response to reflect actual database state:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">201</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
  <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Vendor profile created successfully! Your store is now live.</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">data</span><span class="p">:</span> <span class="p">{</span>
    <span class="nx">vendor</span><span class="p">,</span>
    <span class="na">approvalStatus</span><span class="p">:</span> <span class="nx">vendor</span><span class="p">.</span><span class="nx">approvalStatus</span><span class="p">,</span>
    <span class="na">canReceiveOrders</span><span class="p">:</span> <span class="nx">vendor</span><span class="p">.</span><span class="nx">isVerified</span><span class="p">,</span>
    <span class="na">isPublic</span><span class="p">:</span> <span class="nx">vendor</span><span class="p">.</span><span class="nx">isPublic</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Takeaway:</strong></p>

<p>Response payloads should always reflect database state, not assumptions. This seems obvious but during rapid iteration it's easy to return static values and forget to update them.</p>

<h3>
  
  
  4. Payment Webhook Security Gap
</h3>

<p><strong>What Happened:</strong></p>

<p>While reviewing the payment flow, I realized the webhook endpoint was accepting POST requests without verifying they actually came from Stripe. This is a common attack vector.</p>

<p><strong>The Fix:</strong></p>

<p>Implemented signature verification:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">sig</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">headers</span><span class="p">[</span><span class="dl">'</span><span class="s1">stripe-signature</span><span class="dl">'</span><span class="p">];</span>
<span class="nx">event</span> <span class="o">=</span> <span class="nx">stripe</span><span class="p">.</span><span class="nx">webhooks</span><span class="p">.</span><span class="nf">constructEvent</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">,</span> <span class="nx">sig</span><span class="p">,</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">STRIPE_WEBHOOK_SECRET</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Takeaway:</strong></p>

<p>Payment webhooks are critical infrastructure. Signature verification should be implemented from day one, not added later. I've made it part of my initial setup checklist now.</p>

<h3>
  
  
  5. Missing Authorization on Product Updates
</h3>

<p><strong>What Happened:</strong></p>

<p>The route <code>/products/:id/stock</code> verified that a vendor was logged in, but didn't check if they owned the product. This meant any vendor could potentially modify another vendor's inventory.</p>

<p><strong>The Fix:</strong></p>

<p>Added ownership verification middleware:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">router</span><span class="p">.</span><span class="nf">patch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/products/:id/stock</span><span class="dl">'</span><span class="p">,</span> <span class="nx">protect</span><span class="p">,</span> <span class="nf">authorize</span><span class="p">(</span><span class="dl">'</span><span class="s1">vendor</span><span class="dl">'</span><span class="p">),</span> <span class="nx">attachVendor</span><span class="p">,</span> <span class="nf">validateMongoId</span><span class="p">(</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">),</span> <span class="nx">updateStock</span><span class="p">);</span>
</code></pre>

</div>



<p>The <code>attachVendor</code> middleware ensures requests only succeed if the product belongs to the authenticated vendor.</p>

<p><strong>Takeaway:</strong></p>

<p>Authentication confirms who you are, authorization confirms what you can access. For multi-tenant systems, every resource route needs ownership checks. IDOR vulnerabilities are common when building quickly.</p>




<h2>
  
  
  6. Deployment Cache Issues on Vercel
</h2>

<p><strong>What Happened:</strong></p>

<p>After pushing updates, the production site wasn't reflecting new code. Features that worked locally, including the fixes above, weren't showing up.</p>

<p><strong>The Fix:</strong></p>

<p>Forced a clean deployment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>vercel --prod --force
</code></pre>

</div>



<p>This bypassed caches and deployed fresh. Everything worked after that.</p>

<p><strong>Takeaway:</strong></p>

<p>Platform-specific quirks exist. Vercel, Netlify, and similar services cache aggressively. When behavior doesn't match your local environment, a forced rebuild often resolves it.</p>

<h2>
  
  
  Quick Reference
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Issue</th>
<th>Cause</th>
<th>Solution</th>
<th>Why It Matters</th>
</tr>
</thead>
<tbody>
<tr>
<td>CORS blocking subdomains</td>
<td>Wildcard pattern not configured</td>
<td>Whitelist <code>https://*.example.com</code>
</td>
<td>Common in multi-tenant apps</td>
</tr>
<tr>
<td>Image upload type error</td>
<td>Frontend/backend schema mismatch</td>
<td>Normalize data before sending</td>
<td>Prevents runtime errors</td>
</tr>
<tr>
<td>Response inconsistency</td>
<td>Hardcoded response values</td>
<td>Return actual DB state</td>
<td>Keeps UI in sync</td>
</tr>
<tr>
<td>Vendor IDOR vulnerability</td>
<td>Missing ownership check</td>
<td>Add authorization middleware</td>
<td>Critical for multi-tenant security</td>
</tr>
<tr>
<td>Webhook security gap</td>
<td>No signature verification</td>
<td>Verify Stripe signatures</td>
<td>Prevents payment fraud</td>
</tr>
<tr>
<td>Stale deployments</td>
<td>Build cache</td>
<td>Force clean deploy</td>
<td>Platform-specific behavior</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  What I'd Do Differently Next Time
</h2>

<ul>
<li>Set up CORS patterns early when working with subdomains, not after deployment.</li>
<li>Add ownership middleware to all resource routes from the start, not during security review.</li>
<li>Implement webhook verification before testing payment flows.</li>
<li>Document platform quirks as I encounter them for faster debugging later.</li>
</ul>

<p>These weren't beginner mistakes; they're the kind of issues that surface when shipping real applications with multiple user roles, payment processing, and complex domain setups. Documenting them helps me build faster next time and hopefully helps other developers avoid the same debugging sessions.</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>

</code></pre>

</div>

