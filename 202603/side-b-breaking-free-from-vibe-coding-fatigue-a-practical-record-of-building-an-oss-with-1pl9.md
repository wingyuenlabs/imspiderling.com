---
Title: [Side B] Breaking Free from Vibe Coding Fatigue: A Practical Record of Building an OSS with 'Spec-First AI Development'
Description: 
Author: D
Date: 2026-03-25T22:11:30.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>From the Author:</strong><br>
<strong>D-MemFS</strong> was featured in <a href="https://www.pythonweekly.com/p/python-weekly-issue-737-march-19-2026" rel="noopener noreferrer"><strong>Python Weekly Issue #737</strong></a> (March 19, 2026) under <em>Interesting Projects, Tools and Libraries</em>. Being picked up by one of the most widely-read Python newsletters confirmed that in-memory I/O bottlenecks and memory management are truly universal challenges for developers everywhere. This series is my response to that interest.</p>
</blockquote>

<h3>
  
  
  🧭 About this Series: The Two Sides of Development
</h3>

<p>To provide a complete picture of this project, I’ve split each update into two perspectives:</p>

<ul>
<li>
<strong>Side A (Practical / from Qiita):</strong> Implementation details, benchmarks, and technical solutions.</li>
<li>
<strong>Side B (Philosophy / from Zenn):</strong> The development war stories, AI-collaboration, and design decisions.</li>
</ul>

<p><em>Looking for the other side of this chapter?</em></p>

<ul>
<li>
<strong>[Side A] Building a RAM Disk on Windows without Admin Privileges — Python I/O Acceleration Techniques</strong>: <a href="https://dev.to/d_9d93cd53/side-a-building-a-ram-disk-on-windows-without-admin-privileges-python-io-acceleration-4omd">https://dev.to/d_9d93cd53/side-a-building-a-ram-disk-on-windows-without-admin-privileges-python-io-acceleration-4omd</a>
</li>
</ul>




<h2>
  
  
  Are You Realizing the Limits of "Vibe Coding"?
</h2>

<p>Having AI write code for us has become the norm. Throw a prompt at it, and it returns plausible code. It runs. The tests pass. It's convenient.</p>

<p>However, if you continue this way of "having AI generate code based on a vague vibe"—often called <strong>Vibe Coding</strong>—you will inevitably hit a wall at some point.</p>

<ul>
<li>You don't understand the underlying principles of the code the AI generated.</li>
<li>Trying to fix a bug creates another bug.</li>
<li>You want to refactor, but have no standard for what must be protected.</li>
<li>In code reviews, you can't answer "Why is it done this way?"</li>
</ul>

<p>This is <strong>Vibe Coding Fatigue</strong>. By continually accepting AI's output, the codebase eventually slips out of your control.</p>

<p>So, what should you do?</p>

<p>My answer was: "<strong>Write the design document before the code.</strong>" Inspired by the idea "Write the specifications first," which I had heard somewhere last year, I arbitrarily called this "<strong>Spec-First AI Development</strong>." Before letting the AI write any code, we thoroughly iron out the specifications and design documents.</p>

<h2>
  
  
  Why "Design Documents Before Code"?
</h2>

<p>When you suddenly let an AI write code, the following problems arise:</p>

<p><strong>Problem 1: Reviewing AI-generated code is too difficult for humans.</strong></p>

<p>AI yields hundreds of lines of code in an instant. But the human reviewing it must read and understand every single line. AI code frequently "works but the intention is unreadable." This is because the choices of variables and logic don't reflect human design intent.</p>

<p><strong>On the other hand, reviewing a design document is overwhelmingly easier.</strong> If it says, "Throw an exception before writing if the quota is exceeded," you can judge the validity of that statement as plain language. What's more, you can even have the AI itself review the design document.</p>

<p><strong>Problem 2: Code alone cannot guarantee "Design Philosophy."</strong></p>

<p>Code states "what to do," but not "why it does so." When adding a new feature, AI might suggest an implementation that violates the existing design philosophy. If the philosophy is explicitly stated in the design document, you can instruct the AI to "think of a method based on the design philosophy." The design document becomes a <strong>guardrail</strong>.</p>

<p>For D-MemFS, I explicitly stated design philosophies like "Zero External Dependencies," "Safety First," and "Binary-Exclusive FS" in the design document. Even if AI suggested using a convenient external library, I could reject it with a single phrase: "It goes against the design philosophy."</p>

<p><strong>Problem 3: Context Explosion.</strong></p>

<p>Basic design documents, detailed design documents, test design documents—if you separate documents by phase or purpose, <strong>you don't have to make the implementing AI read all the documents.</strong> When implementing tests, you just need to hand over the test design document. The AI's context window is finite. If you flood it with irrelevant information, the accuracy of the crucial parts drops.</p>

<p><strong>And the biggest advantage: Human understanding advances.</strong></p>

<p>Through the process of bouncing ideas off the AI to refine the design document, the human's system comprehension deepens. "What should this class do?", "Where does this error occur?", "Is this operation thread-safe?"—you begin to hold the answers to these questions in your own mind. Consequently, the code output by the AI becomes remarkably easier to read. You can read it because you understand it.</p>

<h2>
  
  
  Using Browser AIs as Sparring Partners
</h2>

<p>Let me talk about what I actually did to create the design documents.</p>

<p>The first thing I did was <strong>brush up on the idea</strong>. I uploaded the source file of the memory FS feature I had embedded in my personal app to both Gemini and ChatGPT. A sparring session started: "If I were to carve this out as a library, what kind of design would be best?"</p>

<p>I summarized the sparring contents into Markdown and handed it to another AI. Spurred on by that, we sparred some more. Of course, I relentlessly interjected with my own questions.</p>

<p>Once the ideas were somewhat aligned, I had one of the AIs write the design document. From here on, they naturally split into the <strong>role of modifying the design document</strong> and the <strong>role of reviewing</strong>.</p>

<p>:::details [Instruction to Gemini]<br>
Based on the following ideas and requirements,<br>
please write a design document for a Python in-memory file system library.<br>
Requirements: Quota management, Hierarchical directories, Thread-safe, No external dependencies<br>
[Paste the Markdown of ideas]<br>
:::</p>

<p>Gemini outputs a design document (v1). I throw that straight into ChatGPT.</p>

<p>:::details [Instruction to ChatGPT]<br>
Please critique the following design document.<br>
List problems, omissions, and design inconsistencies as much as possible.<br>
[Paste Design Document v1]<br>
:::</p>

<p>ChatGPT lists the problems. I hand that over to Gemini to let it fix them. Then I evaluate it with ChatGPT again. I repeated this back-and-forth. Sometimes I hit Gemini's Pro usage limits.</p>

<p><strong>And I myself also relentlessly kept putting in my two cents.</strong></p>

<p>"Why does this API look like this?", "Are these error types sufficient?", "What happens during concurrent writes?", "When is the quota calculated?" If a vague answer returned, I kept pressing for "more specifics." I didn't let "something roughly like this" slide.</p>

<p>There is one thing to note. If you keep sparring over technical details, <strong>the context fills up rapidly, and the AI's responses gradually get weird.</strong> A dark descent, so to speak. Before that happens, you have the AI generate a handover message and switch to a new chat. I did this "chat switching" numerous times.</p>
<h2>
  
  
  Claude Opus Overturned Everything
</h2>

<p>When the design document had somewhat come together (around v7-v8), I asked Claude Opus to review it.</p>

<p>What came back was a <strong>massive amount of feedback</strong>.</p>

<p>Problems I couldn't catch in sparring with Gemini and ChatGPT surfaced one after another. Issues with lock granularity, naive rollback design upon quota excess, omissions in path normalization rules, latent race conditions in the asynchronous wrapper design...</p>

<p>It was a moment I realized, "This is fundamentally much harder than I thought."</p>

<p>From here, we repeated: modify → Opus review → modify → Sparring → modify... and the design document eventually reached <strong>v13</strong>. Note that I started implementation around v11 or v12.</p>
<h2>
  
  
  Never Bend the Design Philosophy
</h2>

<p>Along the way, the topic of "Wouldn't it be convenient to add this feature too?" came up many times. Every single time, I returned to this question:</p>

<p><strong>"Does this align with the original design philosophy?"</strong></p>

<p>The design philosophy for D-MemFS was set from the beginning:</p>

<ol>
<li>
<strong>Zero External Dependencies</strong> (stdlib only)</li>
<li>
<strong>Safety First</strong> (No dangerous operations provided)</li>
<li><strong>Not just a buffer, but a real FS with a hierarchical structure</strong></li>
<li><strong>Direct extraction support for Archives (ZIP)</strong></li>
</ol>

<p>Any proposal conflicting with these principles, no matter how convenient it seemed, was entirely dropped.</p>

<p>I insisted on zero external dependencies because I had seen the failure of pyfilesystem2. It stopped working due to a single change in setuptools. Relying on a library means adopting that library's risks wholesale. By using only the stdlib, it runs as long as Python runs.</p>

<p>"Safety First" concretely manifests in decisions like "Not providing operations to wipe the entire file system" and "Rejecting operations that could result in path traversal attacks by default." It's better not to possess convenient but dangerous features from the start.</p>
<h2>
  
  
  Looking Back, This Was SDD
</h2>

<p>To summarize, this is what I did:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Spar ideas with AI and write design docs (use multiple AIs)
2. Human thoroughly reviews the design docs
3. Generate code ONLY after the design doc is solidified
4. If a problem is found, fix the design doc first
5. Only after fixing the design doc, fix the code
</code></pre>

</div>



<p>I learned recently that this is precisely the method known in the industry as <strong>SDD (Specification Driven Development)</strong>. While I casually labeled it "Spec-First AI Development" earlier, it seems it already had a proper name. While Vibe Coding is "letting AI write code by vibe," SDD is "controlling AI through specifications." What I was doing was practicing this SDD in collaboration with AI.</p>

<p>The critical points are <strong>4</strong> and <strong>5</strong>. When a problem is found during the implementation phase, it's easy to just fix the code first. But doing so causes the design document and the code to drift apart. The design document turns into "a dream we wrote down at the start," misaligned with reality.</p>

<p>Therefore, when a problem is found, return to the design document first. Fix the design document saying, "This specification should be changed like this," and then fix the code according to that modification. The design document is permanently the "source of truth," and the code follows it. Maintaining this sequence was vital.</p>

<p>By doing so, the evaluation criteria for the code written by AI transforms from "Does it run?" to "Does it correctly implement the intentions of the design document?" I believe this is the framework for "using AI while trusting it."</p>

<p>In my personal opinion, I even feel that <strong>the design document is more important than the code</strong>. As long as the design document is correct, the code can be rewritten infinitely. Whether you let an AI write it or write it by hand, having the standard of a design document allows you to evaluate "if it's correct." Conversely, without a design doc, the standard itself to judge whether the code is correct doesn't exist.</p>

<h2>
  
  
  "It Got Bigger Than I Expected"
</h2>

<p>To be formal about it, I initially envisioned something much smaller. My intention was just to "tidy up the prototype a bit and turn it into a library."</p>

<p>But the more I wrote the design document, the more things I realized I had to consider. If I seriously consider thread safety, I realize I need RW locks. By implementing RW locks, I naturally want to check if it runs properly in GIL-free Python (<code>PYTHON_GIL=0</code>). To check that, I need tests, and writing tests exposes the flaws in the design...</p>

<p>Before I knew it, what emerged was a library running purely on the standard library, supporting hierarchical directories, equipped with quota management, RW-locked, harboring an asynchronous wrapper, supporting free-threaded Python, with <strong>369 tests and 97% coverage</strong>.</p>

<p>In v0.3.0, even a <strong>Memory Guard</strong> was introduced. While Hard Quotas manage the "budget within the virtual FS," the Memory Guard checks the host machine's physical memory and rejects writes proactively if space cannot be secured. Given that an OS will execute an OOM kill before reaching the quota if the set quota exceeds the machine's free memory, this feature became a logical necessity. Having written in the design document that "the quota will absolutely not be exceeded," I could no longer ignore the possibility of death <em>outside</em> the quota — this too is a consequence of "Spec-First." (The technical details will be discussed in the 3rd article.)</p>

<p><strong>The scary part of being Spec-First might be that if you try to do it right, you actually end up creating something truly solid.</strong></p>




<h3>
  
  
  🔗 Links &amp; Resources
</h3>

<ul>
<li>
<strong>GitHub</strong>: <a href="https://github.com/nightmarewalker/D-MemFS" rel="noopener noreferrer">https://github.com/nightmarewalker/D-MemFS</a>
</li>
<li>
<strong>Original Japanese Article</strong>: <a href="https://zenn.dev/nightwalk/articles/5a8fc5de770995" rel="noopener noreferrer">バイブコーディング消耗からの脱却：「スペックファーストAI開発」でOSSを作った実践記録 ― D-MemFS 開発戦記２</a>
</li>
</ul>

<p><strong>If you find this project interesting, a ⭐ on GitHub would be the best way to support my work!</strong></p>

