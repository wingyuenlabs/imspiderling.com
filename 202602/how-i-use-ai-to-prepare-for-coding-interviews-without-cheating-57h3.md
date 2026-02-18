---
Title: How I Use AI to Prepare for Coding Interviews (Without Cheating)
Description: 
Author: Matthew Hou
Date: 2026-02-18T21:46:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>There's a fine line between using AI to prepare for interviews and using AI to cheat in interviews. Here's how I use it for the former.</p>

<h2>
  
  
  The Rules I Set for Myself
</h2>

<ol>
<li>
<strong>AI for learning, never for live interviews</strong> — If I can't solve it without AI during prep, I don't know it well enough</li>
<li>
<strong>Understand every line</strong> — If AI generates code I can't explain, I rewrite it until I can</li>
<li>
<strong>Use AI as a tutor, not a crutch</strong> — "Explain why this is O(n log n)" &gt; "Solve this for me"</li>
</ol>

<h2>
  
  
  My AI-Assisted Interview Prep Workflow
</h2>

<h3>
  
  
  Phase 1: Pattern Recognition (Days 1-3)
</h3>

<p>Instead of grinding random LeetCode, I ask Claude/ChatGPT:</p>

<blockquote>
<p>"Give me 3 problems that use the sliding window pattern, ordered from easy to hard. Don't give me the solution — just the problem statement and a hint about which variant of sliding window to use."</p>
</blockquote>

<p>Then I solve them myself. If I get stuck for 20+ minutes, I ask for a <strong>hint</strong> (not the solution):</p>

<blockquote>
<p>"I'm trying to solve [problem]. I think I need a sliding window but I'm not sure how to handle the shrinking condition. Can you give me a conceptual hint without code?"</p>
</blockquote>

<h3>
  
  
  Phase 2: Mock Interviews with AI (Days 4-7)
</h3>

<p>I use AI as a mock interviewer:</p>

<blockquote>
<p>"You're a senior engineer at Google interviewing me. Ask me a medium-difficulty array problem. After I solve it, give me feedback on:</p>

<ol>
<li>Did I clarify requirements well enough?</li>
<li>Was my approach discussion clear?</li>
<li>Is my code clean and correct?</li>
<li>Did I analyze time/space complexity accurately?
Don't help me during the solve — only give feedback after."</li>
</ol>
</blockquote>

<p>This is surprisingly effective. The AI catches things friends might be too polite to mention.</p>

<h3>
  
  
  Phase 3: System Design Practice (Days 8-10)
</h3>

<blockquote>
<p>"Ask me to design [Twitter's home timeline / URL shortener / chat system]. Play the role of a senior interviewer who asks follow-up questions. Push me on scalability trade-offs."</p>
</blockquote>

<p>After my initial design, I ask:</p>

<blockquote>
<p>"What did I miss? What would a Staff Engineer include that I didn't?"</p>
</blockquote>

<h3>
  
  
  Phase 4: Behavioral Story Crafting (Days 11-14)
</h3>

<p>I write my STAR stories, then ask AI:</p>

<blockquote>
<p>"I'm preparing for a behavioral interview. Here's my answer to 'tell me about a time you disagreed with a teammate': [paste story]. <br>
Critique this: Is it specific enough? Are the metrics compelling? Does the 'Learning' section show growth? How would you improve it without changing the core facts?"</p>
</blockquote>

<h2>
  
  
  What AI Is Terrible At (For Interview Prep)
</h2>

<ol>
<li>
<strong>Judging your actual skill level</strong> — It'll always say "great job!" even when your solution is O(n³)</li>
<li>
<strong>Simulating real pressure</strong> — Real interviews are stressful. AI practice is low-stakes.</li>
<li>
<strong>Company-specific insights</strong> — AI doesn't know that Company X always asks graph problems in round 2</li>
<li>
<strong>Salary negotiation nuance</strong> — Every situation is different; generic advice can backfire</li>
</ol>

<h2>
  
  
  My Results
</h2>

<p>Using this workflow over 2 weeks of prep:</p>

<ul>
<li>Went from solving ~30% of medium LeetCode to ~75%</li>
<li>Behavioral answers went from rambling 5-minute stories to crisp 2-minute STAR-L responses</li>
<li>Got significantly more comfortable thinking out loud</li>
</ul>

<h2>
  
  
  The Prompts I Use Most
</h2>

<p>Here are my most-used prompts, copy-paste ready:</p>

<p><strong>For problem practice:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Give me a [easy/medium/hard] problem that uses [pattern]. 
Problem statement only — no hints, no solution.
</code></pre>

</div>



<p><strong>For code review:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Review my solution for [problem]. Rate it 1-10 on:
- Correctness
- Time complexity
- Code cleanliness
- Edge case handling
What would you change?
</code></pre>

</div>



<p><strong>For mock behavioral:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Ask me: "Tell me about a time you [scenario]"
After I answer, score me on: specificity, metrics, 
structure (STAR), and self-awareness. Be honest.
</code></pre>

</div>






<p>If you want a structured framework for all of this, I put together a few resources:</p>

<ul>
<li>
<a href="https://updatewave.gumroad.com/l/gfspw" rel="noopener noreferrer">Free ATS Resume Checklist</a> — Free, no catch</li>
<li>
<a href="https://updatewave.gumroad.com/l/ipwla" rel="noopener noreferrer">Coding Interview Cheatsheet</a> — All 8 patterns with code templates</li>
<li>
<a href="https://updatewave.gumroad.com/l/jpqrik" rel="noopener noreferrer">Full Interview Playbook</a> — Behavioral + coding + system design + negotiation</li>
</ul>

<p>What's your AI interview prep workflow? Anything I should add?</p>

