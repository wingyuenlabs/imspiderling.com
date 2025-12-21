---
Title: Functional testing: the boring basics that catch real bugs
Description: 
Author: Kelina Cowell
Date: 2025-12-21T21:49:54.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>Portfolio version (canonical, with full context and styling):<br><br>
<a href="https://kelinacowellqa.github.io/Manual-QA-Portfolio-Kelina-Cowell/articles/functional-testing.html" rel="noopener noreferrer">https://kelinacowellqa.github.io/Manual-QA-Portfolio-Kelina-Cowell/articles/functional-testing.html</a></p>
</blockquote>

<h2>
  
  
  TL;DR
</h2>

<ul>
<li>
<strong>What it is:</strong> a functional testing workflow for timeboxed solo passes.</li>
<li>
<strong>Backing project:</strong> <em>Battletoads</em> on PC (Game Pass), one-week pass (27 Oct to 1 Nov 2025).</li>
<li>
<strong>Approach:</strong> validate start-to-control and Pause/Resume first, then expand where risk appears (often input and focus).</li>
<li>
<strong>Outputs:</strong> pass/fail outcomes, reproducible defect reports, and short evidence recordings supporting each finding.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frfa4vnswepnmkg5dmspw.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frfa4vnswepnmkg5dmspw.webp" alt="Functional testing workflow diagram for a timeboxed manual QA pass: start to first control, timebox, mixed input ownership, and outputs (defects, context notes, bug reports, evidence)." width="800" height="533"></a></p>

<p><em>Functional testing workflow used during a one-week Battletoads (PC) pass.</em></p>

<h2>
  
  
  Functional testing workflow context and scope
</h2>

<p>This article is grounded in a self-directed portfolio pass on <strong>Battletoads (PC, Game Pass)</strong>, build <code>1.1F.42718</code>, run in a one-week timebox.</p>

<p>Test focus was <strong>core functional flows</strong> and <strong>mixed input ownership</strong> (controller plus keyboard), with short evidence clips captured for reproducibility.</p>

<h2>
  
  
  What functional testing is (in practice)
</h2>

<p>For me, functional testing is simple: does the product do what it claims to do, end-to-end, without excuses or interpretation?</p>

<p>I validate core flows, confirm expected behaviour, and write issues so a developer can reproduce them without guessing.</p>

<p>The mistake is treating functional testing as “easy” and therefore less valuable. It is the foundation. If the foundation is cracked, everything built on top of it fails in more complicated ways.</p>

<h3>
  
  
  Functional testing outputs: pass/fail results with evidence
</h3>

<p>Clear outcomes: pass or fail, backed by evidence and reproducible steps. Not vibes. Not opinions.</p>

<h2>
  
  
  The two flows I verify first
</h2>

<p>1) <strong>Start to first control</strong><br><br>
The first minute determines whether the product feels broken. If “New Game” does not reliably get you playing, nothing else matters.<br><br>
In <em>Battletoads</em>, I validate this from Title into Level 1 and through the first arena transition before expanding scope.</p>

<p>2) <strong>Pause and Resume</strong><br><br>
Pause stresses state, focus, input context, UI navigation and overlays. If Pause is unstable, you get a stream of defects that look random but are not.<br><br>
In <em>Battletoads (PC)</em>, this surfaced early as keyboard and controller routing issues around Pause and Join In.</p>

<h2>
  
  
  Input ownership testing: controller and keyboard hand-off
</h2>

<p>Mixed input is a feature, not an edge case. When a controller is connected and a keyboard is used, behaviour must remain predictable.</p>

<ul>
<li>Pause must open consistently.</li>
<li>Navigation must respect the active input method.</li>
<li>Confirm and back must not silently stop responding.</li>
<li>Hand-off must not route to the wrong UI or disable input.</li>
</ul>

<p>I treat keyboard and controller hand-off as a dedicated test area because it produces high-impact, easily reproducible defects. In this Battletoads pass, mixed input could misroute actions (for example, <strong>Resume opening Join In</strong>) and temporarily break controller response.</p>

<h3>
  
  
  Common pattern: mixed input causes menu focus bugs
</h3>

<p>Controller connected + keyboard input + menu open = focus bugs. Easy to reproduce. Easy to prove. Easy to fix once isolated.</p>

<h2>
  
  
  Bug evidence: what I capture and why
</h2>

<p>I favour short video clips (10 to 30 seconds) and only use screenshots when they add clarity. The goal is to make the defect obvious without forcing someone to scrub a long recording.</p>

<ul>
<li>
<strong>Video:</strong> shows timing, input, and incorrect outcome together.</li>
<li>
<strong>Screenshot:</strong> supports UI state, text, or configuration.</li>
<li>
<strong>Environment:</strong> platform, build/version, input device, display mode.</li>
</ul>

<p>If the evidence cannot answer what was pressed, what happened, and what should have happened, it is not evidence.</p>

<h2>
  
  
  How I timebox a one-week pass
</h2>

<ul>
<li>
<strong>Day 1:</strong> Smoke testing and baseline flow validation.</li>
<li>
<strong>Days 2 to 4:</strong> Execute runs, expand where risk appears, log defects immediately.</li>
<li>
<strong>Days 5 to 6:</strong> Retest, tighten repro steps, confirm consistency.</li>
<li>
<strong>Day 7:</strong> Summarise outcomes and document learnings.</li>
</ul>

<p>Practical note: I start each session with a short baseline loop (load, gain control, Pause, resume) before deeper checks. It catches obvious breakage early and prevents wasted time.</p>

<h2>
  
  
  Testing oracles used for functional verification
</h2>

<ul>
<li>
<strong>In-game UI and outcomes:</strong> observable behaviour of core flows (control, progression, Pause/Resume).</li>
<li>
<strong>Controls menu bindings:</strong> used as an oracle for expected key behaviour (for example, Esc and Enter bindings).</li>
<li>
<strong>Consistency across repeated runs:</strong> behaviour confirmed via reruns to rule out one-off variance.</li>
</ul>

<h2>
  
  
  Examples from the Battletoads functional pass
</h2>

<h3>
  
  
  Example bug pattern: mixed input misroutes Pause and overlays
</h3>

<p>With a controller connected on PC, Pause opened and closed reliably via controller Start. Keyboard interaction on Pause could be ignored or misrouted into Join In, and in one observed case the controller became unresponsive until the overlay was closed.</p>

<p>Short evidence clips were captured to show input, timing, and outcome together.</p>

<h3>
  
  
  Micro-charter: mixed input ownership around Pause and overlays
</h3>

<ul>
<li>
<strong>Charter:</strong> Mixed input ownership around Pause and overlays (controller plus keyboard).</li>
<li>
<strong>Goal:</strong> Confirm predictable focus, navigation, and confirm/back actions under common PC setups.</li>
</ul>

<h2>
  
  
  Functional testing takeaways (flows, input, evidence)
</h2>

<ul>
<li>Functional testing finds high-impact defects early because it targets the systems everything else relies on.</li>
<li>Pause and overlays are reliable bug generators because they stress state and input routing.</li>
<li>On PC, mixed input should be treated as a primary scenario, not an edge case.</li>
<li>Short evidence clips reduce repro ambiguity and speed up triage.</li>
<li>Repeating the same steps is how “random” issues become diagnosable patterns.</li>
</ul>

<h2>
  
  
  Functional testing FAQ (manual QA)
</h2>

<h3>
  
  
  Is functional testing just basic or beginner testing?
</h3>

<p>No. Functional testing validates the core systems everything else depends on. When it’s done poorly, teams chase “random” bugs that are actually foundational failures.</p>

<h3>
  
  
  Does functional testing only check happy paths?
</h3>

<p>No. It starts with happy paths, but expands wherever risk appears. Input ownership, Pause/Resume, and state transitions are common failure points.</p>

<h3>
  
  
  How is functional testing different from regression testing?
</h3>

<p>Functional testing validates expected behaviour end-to-end. Regression testing verifies that previously working behaviour still holds after change. They overlap, but they answer different questions.</p>

<h3>
  
  
  Is functional testing still relevant if automation exists?
</h3>

<p>Yes. Automation relies on a correct understanding of expected behaviour. Functional testing establishes that baseline and finds issues automation often misses.</p>

<h3>
  
  
  What makes a functional bug report “good”?
</h3>

<p>Clear steps, a clear expected result, a clear actual result, and short evidence that shows input, timing, and outcome together.</p>




<h2>
  
  
  Evidence and case study links
</h2>

<ul>
<li><p>Battletoads functional case study (full artefacts and evidence):<br><br>
<a href="https://kelinacowellqa.github.io/Manual-QA-Portfolio-Kelina-Cowell/projects/battletoads/" rel="noopener noreferrer">https://kelinacowellqa.github.io/Manual-QA-Portfolio-Kelina-Cowell/projects/battletoads/</a></p></li>
<li><p>QA Chronicles Issue 1: Battletoads:<br><br>
<a href="https://kelinacowellqa.github.io/QA-Chronicles-Kelina-Cowell/issues/issue-01-battletoads.html" rel="noopener noreferrer">https://kelinacowellqa.github.io/QA-Chronicles-Kelina-Cowell/issues/issue-01-battletoads.html</a></p></li>
</ul>

<p>This dev.to post stays focused on the workflow. The case study links out to the workbook structure, runs, and evidence.</p>

