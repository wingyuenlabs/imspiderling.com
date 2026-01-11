---
Title: My Phone, Claude Code, and Me
Description: 
Author: sijiaoh
Date: 2026-01-11T21:54:57.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  I Want to Write Code on My Phone
</h2>

<p>On the train, while taking a walk, or during short waiting times—moments when you can’t open a laptop, yet your mind drifts back to code.<br>
That function. That bug. That unfinished feature.</p>

<p>But actually typing code on a phone?<br>
Entering symbols is pure hell. It’s just not practical.</p>

<p>At least, that’s what I used to think.</p>

<h2>
  
  
  After Meeting Coding Agents
</h2>

<p>Once I started using coding agents seriously, my perspective changed.</p>

<p>Natural language turns directly into code.<br>
“Refactor this function,” and it actually refactors it.<br>
“Add some tests,” and the tests appear.<br>
It can even run commands.</p>

<p>That’s when it clicked:<br>
if coding no longer fundamentally depends on keyboard input, then maybe a phone is enough.</p>

<h2>
  
  
  I Built Pockode
</h2>

<p>At first, I thought about putting my entire development environment in the cloud.<br>
But setup was annoying, performance was mediocre, and more importantly—<br>
I didn’t want to give up my familiar local environment.</p>

<p>So I built <a href="https://pockode.com" rel="noopener noreferrer">Pockode</a>.</p>

<p>It’s a tool that lets you control Claude Code running on your home computer from your phone.<br>
As long as Claude Code is installed locally, you start it with one command, scan a QR code, and you’re in.</p>

<p>No new environment to set up. No project migration.<br>
You keep developing exactly the way you always have.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F800dhy3858fo2752euwz.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F800dhy3858fo2752euwz.jpg" alt=" " width="420" height="800"></a></p>

<h2>
  
  
  I Hardly Open Neovim Anymore
</h2>

<p>When I first started developing Pockode, Neovim was still essential—<br>
mainly for checking <code>git diff</code>.</p>

<p>Later, I added diff viewing and file browsing directly into Pockode.<br>
I was developing Pockode using Pockode, originally just for debugging—but then I realized<br>
I no longer needed to open Neovim at all.</p>

<p>Now I only open an editor when I need full-text search or deep code reading.</p>

<p>Sometimes I fix bugs while walking.<br>
I give instructions to the AI on my phone, generate code, run tests.<br>
By the time I get home, the feature is already done.</p>

<h2>
  
  
  99% of the Code Is Written by AI
</h2>

<p>In Pockode, about 99% of the code comes from Claude Code.</p>

<p>But this isn’t “hands-off” development.<br>
I discuss design with it, review the output, and continuously adjust direction.</p>

<p>Honestly, my Go experience was barely beyond tutorials,<br>
and I hadn’t touched React in a long time.<br>
Yet I still managed to build a complete, usable product.</p>

<p>The reasons are simple:</p>

<ul>
<li>Go is highly readable—you can judge logic even if syntax details are fuzzy</li>
<li>AI proactively suggests appropriate libraries and implementations</li>
<li>It introduces approaches I wouldn’t have thought of myself</li>
</ul>

<p>If I had been developing alone, this project would probably have been abandoned before completion.<br>
I kept going because AI was running alongside me.</p>

<h2>
  
  
  Living With AI’s Weaknesses
</h2>

<p>AI isn't perfect.</p>

<p><strong>It can sprint in the wrong direction.</strong><br>
It keeps optimizing locally while forgetting the original goal.<br>
When that happens, you must intervene immediately and pull it back.</p>

<p><strong>It tends to apply quick fixes.</strong><br>
Instead of addressing root causes, it patches symptoms.<br>
Too many patches, and even simple features turn into tangled code.</p>

<p><strong>The worst case is “meaningless code.”</strong><br>
Variables that are defined but never used. Branches that are never reached.<br>
They look harmless, but in the next session they mislead the AI:<br>
“This must exist for a reason.”</p>

<p>And that’s how new misunderstandings are born.</p>

<p>To avoid this, there’s only one requirement:<br>
you must understand the overall structure.<br>
You can’t fully let go—but as long as you hold the steering wheel, AI is extremely powerful.</p>

<h2>
  
  
  Even When You Leave Your Desk, Development Doesn’t Stop
</h2>

<p>Coding agents have a practical problem: permission prompts.</p>

<p>Writing files. Running commands.<br>
Every confirmation can stop progress the moment you step away.</p>

<p>But auto-approving everything is dangerous.</p>

<p>With Pockode, this problem disappears.<br>
Even when you’re out, you can confirm permissions from your phone and keep the agent running.</p>

<p>I used to skip meals because I “didn’t want to break the flow.”<br>
That’s no longer the case.</p>

<p>While walking, on the train, or out and about,<br>
I can always check progress and intervene if needed.</p>

<p>Development is no longer tied to a place.</p>

<p>That’s the kind of life I want.</p>

<p>If this resonates with you,<br>
give <a href="https://pockode.com" rel="noopener noreferrer">Pockode</a> a try.</p>

<p>Original text: <a href="https://sijiaoh.com/en/posts/phone-claude-code-and-me/" rel="noopener noreferrer">https://sijiaoh.com/en/posts/phone-claude-code-and-me/</a></p>

