---
Title: I Gave My LLM a Promotion: Now It Delegates Its Own Work
Description: 
Author: Pavel
Date: 2025-08-15T20:41:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Large Language Models are powerful, but they're also resource-intensive. Every query, no matter how simple, consumes expensive computational cycles. I realized a huge chunk of my server costs was from the LLM repeatedly answering "hello," "thanks," and "ok."</p>

<p>These queries are a waste. They don't teach the model anything new. They don't require complex reasoning. They are pure resource drain.</p>

<p>My first thought was to filter them out on the client-side. But that creates a manual chore—I'd have to constantly update the list of simple phrases. That approach doesn't scale.</p>

<p>So, I flipped the problem on its head. What if the LLM could solve this problem itself?</p>

<p><strong>The core idea is this: I created a system where the LLM itself decides which queries are too simple for its attention and teaches a client-side helper to handle them in the future.</strong></p>

<h3>
  
  
  The Architecture: Self-Delegation
</h3>

<p>I built a project around this single concept, stripping away everything non-essential from a previous version. It has only two key parts:</p>

<ol>
<li> <strong>The Server-Side "Teacher" (The LLM):</strong> The main, powerful model. Its job is to handle complex tasks and—crucially—to identify low-value, repetitive queries.</li>
<li> <strong>The Client-Side "Gatekeeper" (The Helper):</strong> A tiny, zero-dependency JavaScript agent in the browser. It intercepts all user input and asks the LLM for help only when it encounters something it hasn't been taught to handle.</li>
</ol>

<h3>
  
  
  How the LLM Offloads Its Work
</h3>

<p>The first time a user sends a simple query like "thx", the Gatekeeper doesn't recognize it and forwards it to the LLM.</p>

<p>The LLM knows "thx" is simple. Instead of just sending back a text answer, it sends back a special JSON payload containing a direct order:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"userResponse"</span><span class="p">:</span><span class="w"> </span><span class="s2">"You're welcome!"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"learningInstruction"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"LEARN_SIMPLE_PHRASE"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"query"</span><span class="p">:</span><span class="w"> </span><span class="s2">"thx"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"response"</span><span class="p">:</span><span class="w"> </span><span class="s2">"You're welcome!"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This <code>learningInstruction</code> is the key. It’s the LLM telling its Gatekeeper:</p>

<blockquote>
<p><strong>"I've answered this for you once. Now learn it. From now on, you handle this query yourself. Do not send it to me again."</strong></p>
</blockquote>

<p>The Gatekeeper receives this command, saves the new rule to the browser's <code>localStorage</code>, and delivers the response to the user. The user sees nothing but a fast response.</p>

<p>But in the background, the system just became smarter and more efficient. The next time "thx" is sent, the Gatekeeper handles it instantly, and the server is never bothered.</p>

<p>The LLM is actively, automatically, and invisibly making its own job easier. It's not being trained <em>by</em> a human; it's training its own assistant to filter out the noise.</p>

<p>This project was an exercise in minimalism. I threw out a complex neural network library and other unnecessary features to focus solely on perfecting this self-delegation loop. The result is a lean, powerful system that demonstrates a smarter way to build AI applications.</p>

<p>We don't just need bigger models; we need smarter architectures where models can work together and optimize their own workflows.</p>

<p>Check out the full implementation on GitHub and see the self-delegation in action.</p>

<p><a href="https://github.com/Xzdes/slmnet-Hybrid" rel="noopener noreferrer">https://github.com/Xzdes/slmnet-Hybrid</a></p>

