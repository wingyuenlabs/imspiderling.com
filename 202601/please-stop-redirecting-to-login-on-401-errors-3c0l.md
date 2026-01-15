---
Title: Please, Stop Redirecting to Login on 401 Errors 🛑
Description: 
Author: Ilya Ploskovitov
Date: 2026-01-15T21:57:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>You spend 15 minutes filling out a long configuration form. You get a Slack notification, switch tabs, reply to a colleague, and grab a coffee.</p>

<p>30 minutes later, you come back to the form and click <strong>"Save"</strong>.</p>

<p>The page flashes. The login screen appears.<strong>And your data is gone.</strong></p>

<p>This is the most annoying UX pattern in web development, and we need to stop doing it.</p>

<h3>
  
  
  The "Lazy" Pattern
</h3>

<p>Why does this happen? Usually, it's because the JWT (access token) expired, the backend returned a 401 Unauthorized, and the frontend code did exactly what the tutorials said to do:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Don't do this</span>
<span class="nx">axios</span><span class="p">.</span><span class="nx">interceptors</span><span class="p">.</span><span class="nx">response</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="kc">null</span><span class="p">,</span> <span class="nx">error</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">response</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="mi">401</span><span class="p">)</span> <span class="p">{</span>
    <span class="nb">window</span><span class="p">.</span><span class="nx">location</span><span class="p">.</span><span class="nx">href</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">;</span> <span class="c1">// RIP data 💀</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">reject</span><span class="p">(</span><span class="nx">error</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Developers often argue: <em>"But it's a security requirement! The session is dead!"</em></p>

<p>Yes, the session is dead. But that doesn't mean you have to kill the current page state.</p>

<h3>
  
  
  The Better Way (Resilience)
</h3>

<p>If a user is just reading a dashboard, a redirect is fine. But if they have unsaved input (forms, comments, settings), a redirect is a bug.</p>

<p>Here is how a robust app handles this:</p>

<ol>
<li><p><strong>Intercept:</strong> Catch the 401 error.</p></li>
<li><p><strong>Queue:</strong> Pause the failed request. Do not reload the page.</p></li>
<li><p><strong>Refresh:</strong> Try to get a new token in the background (using a refresh token) OR show a modal asking for the password again.</p></li>
<li><p><strong>Retry:</strong> Once authenticated, replay the original request with the new token.</p></li>
</ol>

<p>The user doesn't even notice. The form saves successfully.</p>

<h3>
  
  
  How to test this? (The hard part)
</h3>

<p>Implementing the "Silent Refresh" is tricky, but testing it is annoying.</p>

<p>Access tokens usually last 1 hour. You can't ask your QA team to "wait 60 minutes and then click Save" to verify the fix.</p>

<p>You need a way to trigger a 401 error <strong>exactly when you click the button</strong>, even if the token is valid.</p>

<h3>
  
  
  The "Chaos" Approach
</h3>

<p>Instead of waiting for the token to expire naturally, we can just delete it "mid-flight."</p>

<p>I use <strong>Playwright</strong> for this. We can intercept the outgoing request and strip the Authorization header before it hits the server.</p>

<p>This forces the backend to reject the request, triggering your app's recovery logic immediately.</p>

<p>Here is a Python/Playwright snippet I use to verify my apps are "expiry-proof":<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">test_chaos_silent_logout</span><span class="p">(</span><span class="n">page</span><span class="p">):</span>
    <span class="c1"># 1. Login and go to a form
</span>    <span class="n">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="sh">"</span><span class="s">/login</span><span class="sh">"</span><span class="p">)</span>
    <span class="c1"># ... perform login logic ...
</span>    <span class="n">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="sh">"</span><span class="s">/settings/profile</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># 2. Fill out data
</span>    <span class="n">page</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="sh">"</span><span class="s">#bio</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Important text I don</span><span class="sh">'</span><span class="s">t want to lose.</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># 3. CHAOS: Intercept the 'save' request
</span>    <span class="k">def</span> <span class="nf">kill_token</span><span class="p">(</span><span class="n">route</span><span class="p">):</span>
        <span class="n">headers</span> <span class="o">=</span> <span class="n">route</span><span class="p">.</span><span class="n">request</span><span class="p">.</span><span class="n">headers</span>
        <span class="c1"># We manually delete the token to simulate expiration
</span>        <span class="k">if</span> <span class="sh">"</span><span class="s">authorization</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">headers</span><span class="p">:</span>
            <span class="k">del</span> <span class="n">headers</span><span class="p">[</span><span class="sh">"</span><span class="s">authorization</span><span class="sh">"</span><span class="p">]</span>

        <span class="c1"># Send the "naked" request. Backend will throw 401.
</span>        <span class="n">route</span><span class="p">.</span><span class="nf">continue_</span><span class="p">(</span><span class="n">headers</span><span class="o">=</span><span class="n">headers</span><span class="p">)</span>

    <span class="c1"># Attach the interceptor
</span>    <span class="n">page</span><span class="p">.</span><span class="nf">route</span><span class="p">(</span><span class="sh">"</span><span class="s">**/api/profile/save</span><span class="sh">"</span><span class="p">,</span> <span class="n">kill_token</span><span class="p">)</span>

    <span class="c1"># 4. Click Save
</span>    <span class="n">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="sh">"</span><span class="s">#save-btn</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># 5. Check if we survived
</span>
    <span class="c1"># If the app is bad, we are now on /login
</span>    <span class="c1"># if page.url == "/login": fail()
</span>
    <span class="c1"># If the app is good, it refreshed the token and retried.
</span>    <span class="c1"># The text should still be there, and the save should succeed.
</span>    <span class="nf">expect</span><span class="p">(</span><span class="n">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="sh">"</span><span class="s">#bio</span><span class="sh">"</span><span class="p">)).</span><span class="nf">to_have_value</span><span class="p">(</span><span class="sh">"</span><span class="s">Important text I don</span><span class="sh">'</span><span class="s">t want to lose.</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">expect</span><span class="p">(</span><span class="n">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="sh">"</span><span class="s">.success-message</span><span class="sh">"</span><span class="p">)).</span><span class="nf">to_be_visible</span><span class="p">()</span>
</code></pre>

</div>



<h3>
  
  
  Summary
</h3>

<p>Network failures and expired tokens are facts of life. Your app should handle them without punishing the user.</p>

<p>If you want to build high-quality software, treat 401 Unauthorized as a recoverable error, not a fatal crash.</p>




<p><em>PS: If you need to test this on real mobile devices where you can't run Playwright scripts, you can use a <a href="https://chaos-proxy.debuggo.app/blog/silent-logout-auth-chaos" rel="noopener noreferrer">Chaos Proxy</a> to strip headers on the network level.</em></p>

