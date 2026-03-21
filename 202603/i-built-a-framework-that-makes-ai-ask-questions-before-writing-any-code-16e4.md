---
Title: I Built a Framework That Makes AI Ask Questions Before Writing Any Code
Description: 
Author: Muhammad Hamid Raza
Date: 2026-03-21T21:39:20.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  I Built a Framework That Makes AI Ask Questions Before Writing Any Code
</h1>

<h2>
  
  
  The Moment I Got Frustrated
</h2>

<p>I asked AI to build me an authentication system.</p>

<p>90 seconds later — 40 files generated.</p>

<p>3 months later — I had no idea how any of it worked.</p>

<p>I couldn't debug it. I couldn't extend it.<br>
I was completely dependent on AI forever.</p>

<p>Sound familiar?</p>


<h2>
  
  
  The Real Problem
</h2>

<p>AI coding tools are optimized for one thing: <strong>speed.</strong></p>

<p>You say "add authentication"<br>
AI immediately starts writing code.</p>

<p>But that's NOT how senior developers work.</p>

<p>A senior developer never opens their editor <br>
the moment they hear a feature request.</p>

<p>They:</p>

<ol>
<li>Ask questions — What exactly do you need?</li>
<li>Consider options — What are the trade-offs?</li>
<li>Surface edge cases — What could go wrong?</li>
<li>Plan architecture — How does it fit the system?</li>
<li>THEN write code — Only after all of the above</li>
</ol>

<p><strong>AI skips steps 1 through 4.</strong></p>


<h2>
  
  
  My Solution: Spec-Kit-CoLearn
</h2>

<p>I built <a href="https://github.com/MuhammadHamidRaza/spec-kit-colearn" rel="noopener noreferrer">Spec-Kit-CoLearn</a> — a free open-source <br>
framework that transforms any AI coding tool into <br>
your senior architect co-partner first, <br>
coding worker second.</p>


<h2>
  
  
  Two Modes — One Tool
</h2>
<h3>
  
  
  MODE 1: Senior Architect (Default)
</h3>

<p>When you start, AI enters Senior Architect Mode:</p>

<p>✅ Asks 3-6 discovery questions<br>
✅ Presents 2-3 design options with trade-offs<br>
✅ <strong>Teaches you concepts you don't know</strong><br>
✅ Surfaces at least 3 edge cases<br>
✅ Defines measurable success criteria<br>
❌ Does NOT write code in this mode</p>
<h3>
  
  
  MODE 2: Coding Worker
</h3>

<p>Only activates AFTER you say "tasks approved":</p>

<p>✅ Executes tasks one by one in order<br>
✅ Writes code matching spec exactly<br>
✅ Runs tests after each task<br>
✅ Stops and asks if scope changes</p>
<h3>
  
  
  Why This Order Matters
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>WRONG: Code first → Realize it's wrong → Rewrite → Waste days
RIGHT: Think first → Spec → Plan → Tasks → Code once, code right
</code></pre>

</div>



<h2>
  
  
  Teaching Mode — The Feature I'm Most Proud Of
</h2>

<p>This is what makes Spec-Kit-CoLearn unique.</p>

<p><strong>You say:</strong> "I want users to log in"</p>

<p><strong>AI doesn't write code. It teaches:</strong></p>

<blockquote>
<p>You mentioned authentication. Let me explain <br>
your options before we design anything.</p>

<p><strong>JWT</strong> is like a concert wristband.<br>
You show your ID once at entry.<br>
They give you a wristband.<br>
You show the wristband everywhere inside.<br>
No ID check again on every request.</p>

<p><strong>Sessions</strong> store user data on the server.<br>
More control. Instant logout possible.<br>
But harder to scale.</p>

<p><strong>OAuth</strong> lets users log in via Google or GitHub.<br>
Less friction. You don't manage passwords.<br>
But you depend on third-party.</p>

<p>Which fits your use case?</p>
</blockquote>

<p>Now you've learned AND made an informed decision.</p>


<h2>
  
  
  The Clarity Gate
</h2>

<p>Before ANY spec is written, AI must pass <br>
a hard check — all 6 must pass:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✅ WHO is clear        — Who uses this feature?
✅ WHAT is clear       — What exactly does it do?
✅ SCOPE is defined    — What's IN and what's OUT?
✅ EDGE CASES covered  — At least 3 discussed
✅ SUCCESS measurable  — At least 2 measurable criteria
✅ YOU approved        — You explicitly said "approved"
</code></pre>

</div>



<p>If ANY item fails → more questions.<br>
No gate passed = no spec written.<br>
<strong>This alone saves weeks of rewrites.</strong></p>


<h2>
  
  
  The Learning Log
</h2>

<p>After every feature, AI generates a Learning Log:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## What You Learned:</span>

| Term         | Definition                              |
|--------------|-----------------------------------------|
| JWT          | Stateless token for API authentication  |
| OAuth 2.0    | Third-party login protocol              |
| Sessions     | Server-side user state storage          |
| Bcrypt       | Password hashing with built-in salt     |
| httpOnly     | Cookie JavaScript cannot read (XSS safe)|
</code></pre>

</div>



<p>You don't just ship code — <strong>you ship understanding.</strong></p>




<h2>
  
  
  Full Workflow
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. pip install spec-kit-colearn
2. spec-kit-colearn init my-project
3. Open any AI coding tool
4. Paste activation prompt
5. Tell AI what you want (vague is fine!)
6. AI asks questions → teaches concepts
7. You choose from 3 options
8. Say "approved"
9. AI creates spec → plan → tasks
10. Say "tasks approved"
11. AI switches to Mode 2 → codes task by task
12. Learning log generated ✅
</code></pre>

</div>






<h2>
  
  
  Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Python</span>
pip <span class="nb">install </span>spec-kit-colearn

<span class="c"># JavaScript / Node</span>
npm <span class="nb">install</span> <span class="nt">-g</span> @muhammad-hamid-raza/spec-kit-colearn

<span class="c"># Initialize project</span>
spec-kit-colearn init my-project
</code></pre>

</div>






<h2>
  
  
  Works With Every AI Tool
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>AI Tool</th>
<th>Support</th>
</tr>
</thead>
<tbody>
<tr>
<td>Claude Code</td>
<td>✅</td>
</tr>
<tr>
<td>Codex CLI</td>
<td>✅</td>
</tr>
<tr>
<td>Gemini CLI</td>
<td>✅</td>
</tr>
<tr>
<td>GitHub Copilot</td>
<td>✅</td>
</tr>
<tr>
<td>Cursor</td>
<td>✅</td>
</tr>
<tr>
<td>Windsurf</td>
<td>✅</td>
</tr>
<tr>
<td>Qwen CLI</td>
<td>✅</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Who Is This For?
</h2>

<p><strong>Non-developers</strong> — Have ideas but can't code?<br>
AI guides you step by step and teaches along the way.</p>

<p><strong>Junior developers</strong> — Learn architecture, not just syntax.<br>
Understand WHY before HOW.</p>

<p><strong>Senior developers</strong> — Structured workflow, clear docs,<br>
no more "just write it" chaos.</p>




<h2>
  
  
  Links
</h2>

<p>⭐ GitHub: <a href="https://github.com/MuhammadHamidRaza/spec-kit-colearn" rel="noopener noreferrer">https://github.com/MuhammadHamidRaza/spec-kit-colearn</a><br>
📦 PyPI: <a href="https://pypi.org/project/spec-kit-colearn" rel="noopener noreferrer">https://pypi.org/project/spec-kit-colearn</a><br>
📦 npm: <a href="https://npmjs.com/package/@muhammad-hamid-raza/spec-kit-colearn" rel="noopener noreferrer">https://npmjs.com/package/@muhammad-hamid-raza/spec-kit-colearn</a></p>

<p><strong>MIT license. Free forever. Contributions welcome.</strong></p>




<h2>
  
  
  Final Thought
</h2>

<p>AI should be your <strong>thinking partner.</strong><br>
Not just your code generator.</p>

<p>Build with clarity.<br>
Ship with confidence.<br>
Learn along the way.</p>




<p><em>Built with 🔥 from Pakistan 🇵🇰</em><br>
<em>Feedback welcome in the comments!</em></p>

