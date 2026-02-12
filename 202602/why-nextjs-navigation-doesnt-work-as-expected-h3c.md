---
Title: Why Next.js Navigation Doesn’t Work as Expected
Description: 
Author: Atena / Full stack developer
Date: 2026-02-12T21:44:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>We ran into an issue where clicking a menu item sometimes did not navigate to the expected page.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Firzc3cwhfdnlm5e78qe4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Firzc3cwhfdnlm5e78qe4.png" alt=" " width="800" height="59"></a></p>

<p>The URL in the address bar would change correctly, but the content on the screen would not update. If we kept clicking the menu, navigation would occasionally work.<strong>The issue did not appear in local development, but only in the production environment</strong>.</p>

<p>It took some time to figure out the cause and solution, I would like to share what we learned.</p>

<h2>
  
  
  Reason
</h2>

<h3>
  
  
  Next.js Sends an RSC Request When a <code>&lt;Link&gt;</code> Is Clicked
</h3>

<p>In the App Router, clicking a <code>&lt;Link&gt;</code> component triggers a client-side navigation instead of a full page reload. To render the next page, Next.js sends an internal request related to React Server Components (RSC) to fetch the server-rendered data required for the transition.</p>

<p>It allows the page to update without reloading the entire document.</p>

<h3>
  
  
  What We Found in the Network Tab
</h3>

<p>While inspecting the Network tab during navigation, we noticed the following response headers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>x-middleware-rewrite: /rate-limit-error?retryAfter=1&amp;_rsc=xxxx
x-nextjs-rewritten-path: /rate-limit-error
</code></pre>

</div>



<p><code>x-middleware-rewrite</code>:<br>
The middleware rewrote the internal RSC request to the <code>rate-limit error</code> route.</p>

<p><code>x-nextjs-rewritten-path</code>:<br>
Next.js ultimately processed the request as <code>/rate-limit-error</code>.</p>

<p>This means that when an internal RSC request was triggered, the middleware rewrote it to the rate-limit error route, and Next.js ultimately processed the request as /rate-limit-error.</p>
<h3>
  
  
  Flow of the Issue
</h3>

<ol>
<li>The user clicks a <code>&lt;Link&gt;</code>.</li>
<li>Next.js sends an internal RSC request to fetch data for the next page.</li>
<li>The rate-limiting middleware incorrectly applies rate limiting to this internal request.</li>
<li>The middleware rewrites the request to /rate-limit-error.</li>
<li>The browser URL remains correct, but the fetched content comes from a different route.</li>
<li>This mismatch causes the page to appear broken or not update as expected.</li>
</ol>

<p>This mismatch causes the page to appear broken or not update as expected.</p>
<h3>
  
  
  Let's fix
</h3>
<h4>
  
  
  Before
</h4>

<p>Our rate-limiting middleware was too broad and could apply to internal Next.js requests. As a result, RSC requests triggered by  navigation were sometimes rate-limited and rewritten to <code>/rate-limit-error</code>, causing the URL and rendered content to mismatch.<br>
Before (problem)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Rate limit applied too broadly
const shouldRateLimit = true; ←too broad

// Incomplete RSC detection
const hasRscParam = requestUrl.includes('_rsc=');
const hasRscHeader = request.headers.get('rsc') === '1';
const isRSCRequest = hasRscParam || hasRscHeader;

// Internal requests were not excluded
if (shouldRateLimit &amp;&amp; isRateLimited) {
  return NextResponse.rewrite(
    new URL('/rate-limit-error', request.url)
  );
}
</code></pre>

</div>



<p>Problem:<br>
Internal RSC requests triggered by  navigation could be rate-limited and rewritten.</p>
<h4>
  
  
  After
</h4>

<p>We restricted rate limiting to API routes and non-GET requests only, and explicitly excluded internal Next.js requests (RSC/router requests). This prevented the middleware from rewriting navigation-related requests and restored stable client-side routing.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Rate limit only APIs and non-GET requests
const isApiRoute = pathname.startsWith('/api/');
const isNonGetRequest = request.method !== 'GET';
const shouldRateLimit = isApiRoute || isNonGetRequest;

// Accurate RSC detection
const hasRscParam = requestUrl.includes('_rsc=');
const hasRscHeader = request.headers.get('rsc') === '1';
const hasRscAccept =
  request.headers.get('accept')?.includes('text/x-component');
const isRSCRequest = hasRscParam || hasRscHeader || hasRscAccept;

// Exclude internal navigation requests
if (shouldRateLimit &amp;&amp; !isRSCRequest &amp;&amp; isRateLimited) {
  return NextResponse.rewrite(
    new URL('/rate-limit-error', request.url)
  );
}
</code></pre>

</div>



<p>Exclude internal RSC requests from rate limiting and apply rate limits only to API and non-GET requests.</p>

<h2>
  
  
  To avoid similar issues in the future
</h2>

<ul>
<li>Apply rate limiting only to API routes and non-GET requests.</li>
<li>Explicitly exclude internal Next.js requests such as RSC and router-related requests from middleware logic.</li>
<li>Be cautious when using rewrites in middleware, especially with client-side navigation.</li>
<li>When navigation behaves inconsistently, inspect the Network tab and response headers to check for unexpected rewrites.</li>
</ul>

<p>Keeping these points in mind helps ensure stable navigation and avoids hard-to-debug issues in production.</p>

