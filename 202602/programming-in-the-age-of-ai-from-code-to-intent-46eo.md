---
Title: Programming in the Age of AI: From Code to Intent
Description: 
Author: Roberto B.
Date: 2026-02-13T22:07:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>I was building an AI agent in PHP, wiring up tools, feeding it project documentation, defining how it should interact with the codebase, when a thought stopped me mid-keystroke.</p>

<p>I was writing instructions for an AI, in a language designed for humans, using conventions invented so human brains could follow the logic.</p>

<p>And I thought: why are we still doing this?</p>

<p>Programming languages were designed for us humans:</p>

<ul>
<li>Clear instructions <code>if</code>, <code>while</code>, <code>for</code>.</li>
<li>Indentation for our eyes.</li>
<li>Mnemonics for our memory.</li>
<li>Clean APIs so we can reason about behavior.</li>
<li>Design patterns so we don’t forget how systems are structured.</li>
</ul>

<p>But now AI is writing the code.</p>

<p>AI doesn’t need mnemonics. It doesn’t get confused by nested brackets. It doesn’t get fatigued by verbosity. It can hold an entire codebase in context.</p>

<p>So the question feels inevitable: "Should we create a programming language optimized for AI instead of for humans?"<br>
I think the answer is yes.</p>

<p>But not in the way most people expect.</p>
<h2>
  
  
  The Pattern We Keep Repeating
</h2>

<p>Look at the history of how we talk to machines:</p>

<ul>
<li>
<strong>1950s</strong>: Humans write machine code. Raw numbers. Only a handful of experts can do it.</li>
<li>
<strong>1960s</strong>: Assembly arrives. Still low-level, but now we see <code>MOV</code> and <code>ADD</code> instead of opcodes.</li>
<li>
<strong>1980s–90s</strong>: C, Java, Python. We stop thinking about registers and memory addresses. We write <code>for user in users</code> and let the compiler handle the rest.</li>
<li>
<strong>Today</strong>: We write PHP, Python, TypeScript. AI assists us. But we still write code line by line.</li>
</ul>

<p>Notice the pattern?</p>

<p>Each generation moves humans further from the machine and closer to pure intent.</p>

<p>Nobody writes assembly today. Not because it stopped working, it’s still there under everything you use. We just stopped reading it. It became an intermediate representation.</p>

<blockquote>
<p>Code is about to become the next assembly.</p>
</blockquote>
<h2>
  
  
  The Three Layers of the Future
</h2>

<p>Here’s what I think is emerging.</p>
<h3>
  
  
  Layer 1: Humans Write Intent
</h3>

<p>I expect we will not have code, but we will have specifications. Constraints. Business rules. Performance requirements. Security guarantees.</p>

<p>Instead of:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">if</span> <span class="p">(</span><span class="nv">$user</span><span class="o">-&gt;</span><span class="nf">isAuthenticated</span><span class="p">())</span> <span class="p">{</span> <span class="mf">...</span> <span class="p">}</span>
</code></pre>

</div>



<p>We define something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Authenticated users can save preferences.
Maximum 100 preferences per user.
Response time under 200ms.
Backwards compatible with the v2 API.
</code></pre>

</div>



<p>This looks less like code and more like a contract.</p>

<p>Programming becomes the act of defining <em>what must be true</em>, not how to make it true.</p>

<h3>
  
  
  Layer 2: AI Generates Code
</h3>

<p>PHP, Python, Rust, they don’t disappear.<br>
The ecosystems are too valuable. The runtimes, the libraries, the decades of optimization. AI writes in these languages because that’s where the infrastructure lives.</p>

<p>But humans read this code less and less.</p>

<p>Just like you don’t read the assembly output of your C compiler, you won’t read every line of AI-generated PHP.</p>

<p>Code becomes an implementation detail.</p>

<h3>
  
  
  Layer 3: Machines Verify Correctness
</h3>

<p>This is the critical piece, if humans stop reviewing every line, how do we trust the output?<br>
The answer isn’t “trust the AI more.”</p>

<p>It’s verification.</p>

<ul>
<li>Strong type systems</li>
<li>Contract checking</li>
<li>Property-based testing</li>
<li>Static analysis</li>
<li>Formal verification tools</li>
<li>Automated proofs that the implementation matches the specification</li>
</ul>

<p>The AI-optimized “language” isn’t a new syntax for writing loops.</p>

<p>It’s a rigorous way to describe intent, and to mathematically or systematically verify that generated code satisfies it.</p>

<blockquote>
<p>The future of programming isn’t about generating code faster. It’s about proving code correct automatically.</p>
</blockquote>

<h2>
  
  
  Why This Matters Now
</h2>

<p>We’re in an awkward transition period.</p>

<p>AI writes code, but humans still review it line by line.<br>
That’s like reviewing assembly output in 1995. Technically possible. Not where the value is.</p>

<p>The developers who will thrive aren’t the ones who type the fastest.</p>

<p>They’re the ones who:</p>

<ul>
<li>Define precise specifications</li>
<li>Express constraints clearly</li>
<li>Build strong verification systems</li>
<li>Design architectures that are easy to validate</li>
</ul>

<p>The skill is shifting:</p>

<ul>
<li>From <em>“How do I implement this algorithm?”</em> → to <em>“How do I define what correct means?”</em>
</li>
<li>From <em>“Let me debug this code.”</em> → to <em>“Let me define constraints so bugs can’t exist.”</em>
</li>
<li>From writing instructions for machines → to writing intent for AI.</li>
</ul>

<h2>
  
  
  What This Means for Junior Developers (and People Starting Today)
</h2>

<p>There’s an uncomfortable question hidden in all this:</p>

<blockquote>
<p>If AI writes most of the code in the future, is learning programming still worth it?</p>
</blockquote>

<p>Yes. More than ever.</p>

<p>Understanding programming logic is not about memorizing syntax. It’s about understanding:</p>

<ul>
<li>Control flow</li>
<li>Data structures</li>
<li>State</li>
<li>Side effects</li>
<li>Performance trade-offs</li>
<li>System design</li>
<li>Failure modes</li>
</ul>

<p>If you understand how machines think, you can describe intent more precisely.</p>

<p>You know what the machine expects.<br>
You know what can go wrong.<br>
You know where ambiguity hides.</p>

<p>And that makes you dramatically more effective when collaborating with AI.<br>
The abstraction layer may rise. But understanding what’s underneath remains a superpower.</p>

<p>There’s another practical reason this matters today.</p>

<p>If the future depends on verification and coherence, then the habits we adopt now become critical:</p>

<ul>
<li>Use strict typing.</li>
<li>Write automated tests.</li>
<li>Use static analysis tools.</li>
<li>Enforce consistent architecture.</li>
<li>Keep code uniform and predictable.</li>
</ul>

<p>Why?</p>

<p>Because coherent, well-structured codebases are easier for AI to understand, extend, and reason about.</p>

<p>If tomorrow AI generates 100% of the code, consistency will reduce:</p>

<ul>
<li>Errors</li>
<li>Misunderstandings</li>
<li>Hidden edge cases</li>
<li>Bias introduced by unclear patterns</li>
</ul>

<p>Clean, strictly typed, well-tested systems are not just “good engineering.”<br>
They are AI-ready systems.</p>

<blockquote>
<p>Learning programming today is not obsolete. It’s training for a higher level of abstraction tomorrow.</p>
</blockquote>

<h2>
  
  
  The Real Singularity in Software
</h2>

<p>The singularity isn’t AI writing code.<br>
That’s already happening.<br>
The real shift is the moment when code becomes an intermediate representation that nobody reads, just like assembly before it.<br>
When that happens, “programming” won’t mean writing PHP or Python.</p>

<p>It will mean writing intent.</p>

<p>The language changes. The skill doesn’t.</p>

<p>It has always been about turning human intent into working machines.</p>

<p>We’re just removing one more layer of translation.</p>

<p>If you’re already using AI tools such as Copilot, Claude, Cursor, or others, have you already noticed the shift?</p>

<p>Less mental energy on syntax. More on defining what you actually want.</p>

<p>And the uncomfortable question:</p>

<p>When was the last time you read every line of AI-generated code before shipping it?</p>

<p>Are we ready to let code become the next assembly?</p>

<p>Or are we holding on because we’re not ready to trust what we can’t read?</p>

