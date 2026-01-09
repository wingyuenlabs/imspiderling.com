---
Title: Vibe Coding vs AI-Driven Development: The Contracts Problem (and GS-TDD)
Description: 
Author: Dennis Schmock
Date: 2026-01-09T22:08:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most AI-assisted coding right now is just… vibes.</p>

<p>You ask for a feature, you get a plausible diff, it compiles, your dopamine spikes, and two days later production starts doing interpretive dance.</p>

<p>That failure mode has a name: <strong>vibe coding</strong>.</p>

<p>What I’m aiming for instead is <strong>AI-driven development</strong>. Whether you call it AI-Assisted Engineering or (as we say in my native Denmark) <em>AI-drevet udvikling</em>, the goal is the same: a workflow where AI helps you design and implement software every day, while humans keep full accountability for correctness, security, and operations.</p>

<p>The difference isn’t “which model” you use. It’s <strong>contracts</strong>.</p>

<h2>
  
  
  Vibe coding is a trust problem
</h2>

<p>Vibe coding is what happens when we treat generated code like it’s “probably fine” because:</p>

<ul>
<li>it <em>looks</em> reasonable</li>
<li>it uses familiar patterns</li>
<li>it passes a quick smoke test</li>
<li>it’s late and we want to go home</li>
</ul>

<p>AI is great at producing plausible code. But plausible code is not the same as correct code.</p>

<p>If your workflow is “generate → ship”, you’re not doing AI-driven development. You’re doing probabilistic deployment.</p>

<h2>
  
  
  AI-driven development is a contract problem
</h2>

<p>The reliable version is simple, but it requires discipline:</p>

<ol>
<li>
<strong>Define the contract</strong> (Humans)</li>
<li>Let AI work <strong>inside</strong> that contract (Machine)</li>
<li>Verify the result like an adult (Humans + CI)</li>
</ol>

<p>The “contract” can be acceptance tests, unit tests, invariants, schemas, types, or all of the above. But tests are the most universal contract because they encode behavior <em>and</em> they fail loudly.</p>

<h2>
  
  
  The boring loop: Red → Gold → Refactor (GS-TDD)
</h2>

<p>To operationalize this, I use a variation of TDD I call <strong>Gold Standard TDD (GS-TDD)</strong>. It evolves the classic Red–Green–Refactor loop into <strong>Red–Gold–Refactor</strong>.</p>

<ol>
<li>
<strong>Red</strong>: Write (or generate) a failing test suite that defines behavior. Ideally BDD-style so intent and edge cases are explicit.</li>
<li>
<p><strong>Gold</strong>: Instruct the AI to produce a <strong>Gold Standard implementation</strong> on the first pass, which should be <em>production-oriented by design</em><br>
That means that we have:    </p>

<ul>
<li>Security-aware defaults</li>
<li>Maintainable structure</li>
<li>Solid error handling and boundaries</li>
<li>Boring, standard architecture</li>
<li>no mvp, no prototyping</li>
</ul>
</li>
<li><p><strong>Refactor</strong>: Improve the internals safely while keeping behavior unchanged.</p></li>
</ol>

<p><strong>The key shift:</strong> GS-TDD replaces “minimal Green” with “Gold Standard”, because AI allows us to skip the boilerplate phase. Your tests (the contract) keep the AI honest, and the "Gold" prompt keeps the architecture clean.</p>

<p><em>For a deeper dive into the methodology, check out my research note on <a href="https://www.boringreliability.dev/research/gs-tdd" rel="noopener noreferrer">GS-TDD</a>.</em></p>

<h2>
  
  
  A tiny example (The Workflow)
</h2>

<p>Say you’re adding a rule: “Users can’t create more than 3 API keys”.</p>

<p><strong>Red</strong>: Write the contract first.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">it</span><span class="p">(</span><span class="dl">"</span><span class="s2">limits API keys per user</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">seedUser</span><span class="p">();</span>
  <span class="k">await</span> <span class="nf">createKey</span><span class="p">(</span><span class="nx">user</span><span class="p">);</span>
  <span class="k">await</span> <span class="nf">createKey</span><span class="p">(</span><span class="nx">user</span><span class="p">);</span>
  <span class="k">await</span> <span class="nf">createKey</span><span class="p">(</span><span class="nx">user</span><span class="p">);</span>

  <span class="c1">// This defines the contract: behavior AND error shape</span>
  <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nf">createKey</span><span class="p">(</span><span class="nx">user</span><span class="p">)).</span><span class="nx">rejects</span><span class="p">.</span><span class="nf">toThrow</span><span class="p">(</span><span class="dl">"</span><span class="s2">API key limit reached</span><span class="dl">"</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Now the test suite is the boss.</p>

<p><strong>Gold</strong>: Prompt the AI for a production-oriented first pass <em>under constraints</em>. You don't just say "make it pass". You say:</p>

<blockquote>
<p>"Implement this feature. Enforce the limit transactionally to avoid race conditions, use our standard domain error types, and add a structured log line when the limit is hit."</p>
</blockquote>

<p>The “Gold” code might still fail a test or two on the first run — that’s fine. You debug inside the contract until it’s green.</p>

<p><strong>Refactor</strong>: Once behavior is proven, you clean up naming and file structure.</p>

<p>AI can help in all steps — but <strong>the contract decides what’s allowed</strong>.</p>

<h2>
  
  
  The Boring AI Checklist (8 rules I actually follow)
</h2>

<ol>
<li>
<strong>Define the contract</strong>: Tests describe what must remain true.</li>
<li>
<strong>Verify, don’t trust</strong>: Run tests, lint, and build locally.</li>
<li>
<strong>Prefer small diffs</strong>: AI hallucinates more in large contexts. Keep changes reviewable.</li>
<li>
<strong>Keep prompts in the diff</strong>: The “why” belongs in PRs/docs, not in your chat history.</li>
<li>
<strong>Review like it’s human code</strong>: Check naming, invariants, and boundaries.</li>
<li>
<strong>Guard risky edges</strong>: Auth, data loss, security headers, rate limits.</li>
<li>
<strong>Watch production</strong>: Logs/metrics/alerts confirm the change behaved.</li>
<li>
<strong>Rollback is a feature</strong>: Know how you undo the change before you ship.</li>
</ol>

<p>That’s it. No mysticism. Just boring verification at high speed.</p>

<h2>
  
  
  “But isn’t this just TDD?”
</h2>

<p>Sort of — but with a twist in motivation.</p>

<p>Classic TDD helps humans think clearly and design well. With AI in the loop, tests also become <strong>safety rails</strong> that stop plausible nonsense from sliding into main.</p>

<p>If your AI workflow doesn’t have contracts, your delivery pipeline is basically a slot machine that sometimes pays out.</p>

<h2>
  
  
  Read the full series
</h2>

<p>I’m documenting this entire methodology (and the concept of "Wards") on my site.</p>

<ul>
<li>
<strong>The Hub</strong>: <a href="https://www.boringreliability.dev/ai-driven-development" rel="noopener noreferrer">AI-driven development / AI-drevet udvikling</a>
</li>
<li>
<strong>The Core Idea</strong>: <a href="https://www.boringreliability.dev/blog/ward-2-red-gold-refactor" rel="noopener noreferrer">Ward #2: Red-Gold-Refactor</a>
</li>
</ul>

<p>The future of coding is boring: small diffs, explicit contracts, and fewer production surprises.</p>

<p>Which is exactly the kind of future worth shipping.</p>

