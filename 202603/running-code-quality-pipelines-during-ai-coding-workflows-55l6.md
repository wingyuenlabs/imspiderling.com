---
Title: Running code quality pipelines during AI coding workflows
Description: 
Author: Toni Antunovic
Date: 2026-03-12T21:23:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been experimenting a lot with AI-assisted coding tools like Claude Code and Cursor.</p>

<p>One thing I noticed is that code quality checks usually run only later in CI. Linting, type checks, tests, security scans, and coverage often happen after the code is already written.</p>

<p>That workflow works for humans, but it feels awkward when AI is generating code.</p>

<p>So I started experimenting with running the <strong>entire quality pipeline locally during development</strong> and exposing the results in a way that AI tools can use to iterate on fixes.</p>

<p>That experiment became a small project called LucidShark.</p>

<p><strong>What LucidShark does</strong></p>

<p>LucidShark is a <strong>local-first CLI code quality pipeline</strong> designed to work well with AI coding workflows.</p>

<p>Key ideas:</p>

<ul>
<li>Runs entirely from the <strong>CLI</strong>
</li>
<li>
<strong>Local-first</strong> (no SaaS or external service)</li>
<li>
<strong>Configuration as code</strong> via a repo config file</li>
<li>Integrates with <strong>Claude Code via MCP</strong>
</li>
<li>Generates a <strong>quality overview that can be committed to git</strong>
</li>
</ul>

<p>It orchestrates common quality checks such as:</p>

<ul>
<li>linting</li>
<li>type checking</li>
<li>tests</li>
<li>security scans</li>
<li>coverage</li>
</ul>

<h2>
  
  
  Example usage
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>lucidshark
lucidshark init
lucidshark scan
</code></pre>

</div>



<p>Language and tool support is still <strong>fairly limited</strong> right now, but it should work reasonably well for <strong>Python and Java</strong> projects.</p>

<h2>
  
  
  Why I built it
</h2>

<p>The main goal is to explore workflows where <strong>AI agents can read quality results and fix issues automatically</strong>, instead of developers running the checks manually later.</p>

<p>The project is still early and I'm mostly looking for feedback from people experimenting with AI coding workflows.</p>

<p>GitHub: <a href="https://github.com/toniantunovi/lucidshark" rel="noopener noreferrer">https://github.com/toniantunovi/lucidshark</a><br>
Docs: <a href="https://lucidshark.com" rel="noopener noreferrer">https://lucidshark.com</a></p>

