---
Title: How I Fixed an SSI-Breaking Bug in NGINX Gateway Fabric
Description: 
Author: Michael Uanikehi
Date: 2026-03-23T21:47:24.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Introduction</strong></p>

<p>This bug found me, not the other way around.</p>

<p>I was in the middle of migrating my team's infrastructure from NGINX Ingress Controller to NGINX Gateway Fabric when SSI (Server-Side Includes) stopped working. Subrequests were hitting the wrong backend paths, and pages that relied on SSI includes were silently broken. I could have worked around it, added a flag, patched a config, moved on. Instead, I decided to find where it was actually coming from and fix it at the source.</p>

<p>This is the story of that bug, what caused it, and the one-line condition that fixed it.</p>

<p><strong>Background: What is NGINX Gateway Fabric?</strong></p>

<p>NGINX Gateway Fabric is a Kubernetes Gateway API implementation backed by NGINX. It translates Kubernetes <code>HTTPRoute</code> resources into NGINX configuration, handling routing, load balancing, and traffic management. It's the next-generation replacement for NGINX Ingress Controller, built around the Kubernetes Gateway API spec.</p>

<p>When processing HTTP routes, it generates <code>proxy_pass</code> directives in NGINX location blocks to forward traffic to upstream backends.</p>

<p><strong>The Migration That Surfaced the Bug</strong></p>

<p>During the migration from NGINX Ingress Controller to NGINX Gateway Fabric, one of our services used SSI to compose pages from multiple backend responses a pattern like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;h1&gt;</span>SSI Test Page<span class="nt">&lt;/h1&gt;</span>
<span class="c">&lt;!--# include virtual="/include.html" --&gt;</span>
</code></pre>

</div>



<p>On NGINX Ingress Controller this worked fine. After switching to NGINX Gateway Fabric, the SSI includes were broken. After some investigation, the generated <code>proxy_pass</code> directive was the culprit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">proxy_pass</span> <span class="s">http://my-backend</span><span class="nv">$request_uri</span><span class="p">;</span>
</code></pre>

</div>



<p><code>$request_uri</code> in NGINX always holds the <strong>original</strong> client request URI, it never changes, even during internal subrequests. So when SSI triggered a subrequest to <code>/include.html</code>, the <code>proxy_pass</code> directive would still forward <code>$request_uri</code> (e.g. <code>/</code>) to the backend completely ignoring the subrequest's intended path.</p>

<p><strong>Understanding the Two Location Types</strong></p>

<p>NGINX Gateway Fabric uses two types of location blocks:</p>

<p><strong>External locations</strong> — match the original client request directly. Here, <code>$uri</code> is already the correctly processed URI. Using <code>$request_uri</code> is redundant and harmful.</p>

<p><strong>Internal locations</strong> — used for NJS-driven HTTP matching. When a request comes in, NJS evaluates the matching rules and calls <code>r.internalRedirect(match.redirectPath + args)</code>, which changes <code>$uri</code> to an internal path like <code>/@rule0-route0</code>. Without <code>$request_uri</code>, NGINX would forward that internal path to the backend — so <code>$request_uri</code> is needed here to restore the original client URI.</p>

<p>The bug was that the code made no distinction between these two cases.</p>

<p><strong>The Fix</strong></p>

<p>In <code>internal/controller/nginx/config/servers.go</code>, the <code>createProxyPass</code> function was changed from:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Before: applies to ALL non-gRPC locations</span>
<span class="k">if</span> <span class="o">!</span><span class="n">grpc</span> <span class="p">{</span>
    <span class="k">if</span> <span class="n">filter</span> <span class="o">==</span> <span class="no">nil</span> <span class="o">||</span> <span class="n">filter</span><span class="o">.</span><span class="n">Path</span> <span class="o">==</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">requestURI</span> <span class="o">=</span> <span class="s">"$request_uri"</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>To:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// After: only applies to INTERNAL locations</span>
<span class="k">if</span> <span class="o">!</span><span class="n">grpc</span> <span class="o">&amp;&amp;</span> <span class="n">locationType</span> <span class="o">==</span> <span class="n">http</span><span class="o">.</span><span class="n">InternalLocationType</span> <span class="p">{</span>
    <span class="k">if</span> <span class="n">filter</span> <span class="o">==</span> <span class="no">nil</span> <span class="o">||</span> <span class="n">filter</span><span class="o">.</span><span class="n">Path</span> <span class="o">==</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">requestURI</span> <span class="o">=</span> <span class="s">"$request_uri"</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This means:</p>

<ul>
<li>
<strong>External locations</strong> generate: <code>proxy_pass http://my-backend;</code>
</li>
<li>
<strong>Internal locations</strong> still generate: <code>proxy_pass http://my-backend$request_uri;</code>
</li>
</ul>

<p><strong>Verifying the Fix</strong></p>

<p>I deployed the fix to a local <code>kind</code> cluster with an SSI-enabled backend. Before the fix, the SSI include failed the subrequest hit the wrong path. After the fix, the response correctly returned:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;h1&gt;</span>SSI Test Page<span class="nt">&lt;/h1&gt;</span>
<span class="nt">&lt;p&gt;</span>This content was included via SSI!<span class="nt">&lt;/p&gt;</span>
</code></pre>

</div>



<p>The generated NGINX config confirmed the clean <code>proxy_pass</code> directive with no <code>$request_uri</code> for the external location.</p>

<p><strong>Takeaways</strong></p>

<ul>
<li>Real-world migrations are great bug finders. Moving from NGINX Ingress Controller to NGINX Gateway Fabric exposed a behavioral difference that tests hadn't caught.</li>
<li>NGINX's <code>$request_uri</code> is immutable across the entire request lifecycle, including subrequests. it always reflects the <em>original</em> client URI.</li>
<li>Blindly appending <code>$request_uri</code> to <code>proxy_pass</code> can interfere with NGINX features that rely on internal subrequests (SSI, auth subrequests, etc.).</li>
<li>When you hit a bug in open source, you can just fix it. The maintainers were responsive and collaborative throughout the review process.</li>
</ul>

<p><strong>PR:</strong> <a href="https://github.com/nginx/nginx-gateway-fabric/pull/4935" rel="noopener noreferrer">https://github.com/nginx/nginx-gateway-fabric/pull/4935</a></p>

