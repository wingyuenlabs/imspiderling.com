---
Title: 🍅 Building a Pomodoro Timer That Cosplays: My ADHD-Fueled Kiroween Journey
Description: 
Author: Natalie Gil
Date: 2025-12-04T21:44:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>@kirodotdev # 🍅 Building a Pomodoro Timer That Cosplays: My ADHD-Fueled Kiroween Journey</p>

<p><strong>TL;DR</strong>: I built a Pomodoro timer that transforms into 5 different "costumes" - from 8-bit retro terminals to cozy coffee shops to RPG dungeons. All sounds are generated programmatically (no audio files!), and there's a secret easter egg in this post. Can you find it? 👀</p>

<h2>
  
  
  🧠 The ADHD Problem
</h2>

<p>Let me be real with you: I have ADHD, and traditional productivity tools <strong>bore me to tears</strong>. </p>

<p>You know the cycle:</p>

<ol>
<li>Download a beautiful, minimalist Pomodoro timer</li>
<li>Use it for exactly 2 days</li>
<li>Get bored</li>
<li>Back to scrolling Twitter during "focus time"</li>
</ol>

<p>The problem isn't discipline - it's <strong>novelty</strong>. My ADHD brain craves new experiences, but productivity requires structure. How do you balance both?</p>

<h2>
  
  
  💡 The "Costume Timer" Concept
</h2>

<p>While riding Uber these past few days (because apparently that's where all good ideas happen), I had a thought:</p>

<blockquote>
<p>What if my timer could wear different costumes? Like, literally transform its entire personality?</p>
</blockquote>

<p>Not just theme switching - I mean <strong>complete transformations</strong>:</p>

<ul>
<li>Different UI aesthetics</li>
<li>Unique sound effects</li>
<li>Changed interaction patterns</li>
<li>Hidden triggers to discover</li>
</ul>

<p>The novelty my ADHD brain needs, wrapped in the Pomodoro structure that actually works.</p>

<h2>
  
  
  🎃 Enter Kiroween
</h2>

<p>I joined the <a href="https://kiroween.devpost.com/" rel="noopener noreferrer">Kiroween hackathon</a> and decided to build this idea. But here's the thing - I'm chaotic. I had:</p>

<ul>
<li>Experiments in VS Code with Copilot</li>
<li>Prototypes in Lovable</li>
<li>Ideas scattered across ChatGPT and Claude conversations</li>
<li>Code pieces that worked separately but not together</li>
</ul>

<p><strong>This is where Kiro became my secret weapon.</strong></p>

<h2>
  
  
  🛠️ The Build Process (Organized Chaos)
</h2>

<h3>
  
  
  Phase 1: Mobile Spec Refinement
</h3>

<p>In Ubers, waiting rooms, anywhere with downtime, I used ChatGPT-4 and Claude Opus to refine the concept:</p>

<ul>
<li>"What makes a timer ADHD-friendly?"</li>
<li>"How do I balance novelty with usability?"</li>
<li>"What triggers feel rewarding to discover?"</li>
</ul>

<p>This wasn't formal documentation - it was conversational thinking. But it gave me clarity.</p>

<h3>
  
  
  Phase 2: Scattered Experiments
</h3>

<p>I built pieces everywhere:</p>

<ul>
<li>Sound system in VS Code</li>
<li>UI modes in Lovable
</li>
<li>Timer logic in another prototype</li>
</ul>

<p>Each worked, but they were <strong>incompatible frameworks and approaches</strong>.</p>

<h3>
  
  
  Phase 3: Kiro Integration Magic
</h3>

<p>This is where Kiro shined. I brought my scattered code and said:</p>

<blockquote>
<p>"Here's my sound system, here's my UI modes, here's my timer logic - make them work together."</p>
</blockquote>

<p>Kiro didn't just paste code. It:</p>

<ul>
<li>
<strong>Understood the architecture</strong> of each piece</li>
<li>
<strong>Refactored conflicts</strong> between different approaches</li>
<li>
<strong>Unified state management</strong> so all costumes could coexist</li>
<li>
<strong>Maintained functionality</strong> while adding personality</li>
</ul>

<p>In hours, I had a cohesive app. That's the power of AI-assisted integration.</p>

<h2>
  
  
  🎮 The 5 Costumes
</h2>

<h3>
  
  
  1. 🎮 Pixel Mode (Konami Code)
</h3>

<p><strong>Trigger</strong>: Type ↑ ↑ ↓ ↓ ← → ← → B A</p>

<p>The entire app transforms into a retro 8-bit terminal:</p>

<ul>
<li>Green phosphor display aesthetics</li>
<li>Animated pixel tomato emoji</li>
<li><strong>8-bit square wave notification sounds</strong></li>
</ul>

<h3>
  
  
  2. ☕ Coffee Shop Mode
</h3>

<p><strong>Trigger</strong>: Click the coffee icon 3 times rapidly</p>

<p>Welcome to a randomly-generated café:</p>

<ul>
<li>Shop names like "The Caffeinated Tomato"</li>
<li>Barista names like "Mochaccino Productivityson"</li>
<li><strong>Steam wand hissing sound effects</strong></li>
<li>Lo-fi study vibes</li>
</ul>

<h3>
  
  
  3. 🧙‍♂️ Gandalf Mode
</h3>

<p><strong>Trigger</strong>: Click the tomato 5 times OR type "you shall not pass"</p>

<p>Epic wizard blocking interface:</p>

<ul>
<li>Dramatic Gandalf ASCII art</li>
<li>Motivational wizard quotes</li>
<li>"You shall not pass... into distractions"</li>
</ul>

<h3>
  
  
  4. 🗡️ RPG Dungeon Mode
</h3>

<p><strong>Trigger</strong>: Complete 4 Pomodoros in a row OR type "askaskask"</p>

<p>Terminal-style dungeon crawler:</p>

<ul>
<li>Fight the Procrastination Dragon</li>
<li>Gain Focus XP</li>
<li>Level up your attention span</li>
<li>Multi-phase battle sequence</li>
</ul>

<h3>
  
  
  5. ⏰ Time Travel Mode (Secret!)
</h3>

<p><strong>Trigger</strong>: Type <code>/timetravel</code></p>

<p>Restore previous timer states - because sometimes you need to rewind time itself.</p>

<h2>
  
  
  🔊 The Sound System (No Audio Files!)
</h2>

<p>Here's something cool: <strong>every sound is generated programmatically</strong> using the Web Audio API.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">playNotificationSound</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">audioContext</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AudioContext</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">oscillator</span> <span class="o">=</span> <span class="nx">audioContext</span><span class="p">.</span><span class="nf">createOscillator</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">gainNode</span> <span class="o">=</span> <span class="nx">audioContext</span><span class="p">.</span><span class="nf">createGain</span><span class="p">();</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">showCoffeeMode</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Steam wand sound with frequency modulation</span>
    <span class="nx">oscillator</span><span class="p">.</span><span class="nx">frequency</span><span class="p">.</span><span class="nf">setValueAtTime</span><span class="p">(</span><span class="mi">150</span><span class="p">,</span> <span class="nx">audioContext</span><span class="p">.</span><span class="nx">currentTime</span><span class="p">);</span>
    <span class="nx">oscillator</span><span class="p">.</span><span class="nx">frequency</span><span class="p">.</span><span class="nf">exponentialRampToValueAtTime</span><span class="p">(</span><span class="mi">300</span><span class="p">,</span> <span class="nx">audioContext</span><span class="p">.</span><span class="nx">currentTime</span> <span class="o">+</span> <span class="mf">0.3</span><span class="p">);</span>
    <span class="nx">oscillator</span><span class="p">.</span><span class="nx">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">sawtooth</span><span class="dl">'</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">isPixelMode</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// 8-bit square wave</span>
    <span class="nx">oscillator</span><span class="p">.</span><span class="nx">frequency</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="mi">523</span><span class="p">;</span> <span class="c1">// C5 note</span>
    <span class="nx">oscillator</span><span class="p">.</span><span class="nx">type</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">square</span><span class="dl">'</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nx">oscillator</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">gainNode</span><span class="p">);</span>
  <span class="nx">gainNode</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">audioContext</span><span class="p">.</span><span class="nx">destination</span><span class="p">);</span>
  <span class="nx">oscillator</span><span class="p">.</span><span class="nf">start</span><span class="p">();</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Each costume has unique sounds:</p>

<ul>
<li>
<strong>Pixel Mode</strong>: Square wave beeps (classic 8-bit)</li>
<li>
<strong>Coffee Mode</strong>: Sawtooth wave with frequency ramps (steam hissing)</li>
<li>
<strong>Default</strong>: Sine wave with gentle fade (pleasant notification)</li>
</ul>

<p>No MP3s, no WAVs, just <strong>math and oscillators</strong>. The entire sound system is ~50 lines of code.</p>

<h2>
  
  
  🎯 Why This Works for ADHD
</h2>

<p>The costume system solves the ADHD productivity paradox:</p>

<ol>
<li>
<strong>Novelty</strong>: 5 different experiences keep it fresh</li>
<li>
<strong>Structure</strong>: Pomodoro technique still enforces focus</li>
<li>
<strong>Discovery</strong>: Hidden triggers create rewarding moments</li>
<li>
<strong>Personality</strong>: Each mode has unique vibes</li>
<li>
<strong>Functionality</strong>: Timer always works, regardless of costume</li>
</ol>

<p>It's like having 5 different apps in one, but they all serve the same purpose: <strong>helping you focus</strong>.</p>

<h2>
  
  
  🚀 The Tech Stack
</h2>

<ul>
<li>
<strong>React + TypeScript</strong>: Component architecture</li>
<li>
<strong>Tailwind CSS</strong>: Rapid styling and theme switching</li>
<li>
<strong>Web Audio API</strong>: Programmatic sound generation</li>
<li>
<strong>Vite</strong>: Fast development</li>
<li>
<strong>shadcn/ui</strong>: Accessible components</li>
</ul>

<p>All open source, all modern, all fast.</p>

<h2>
  
  
  🎃 The Kiroween Experience
</h2>

<p>Building this for Kiroween taught me something important:</p>

<p><strong>AI coding isn't about replacing developers - it's about amplifying creative chaos.</strong></p>

<p>I had ideas scattered across multiple tools and environments. Kiro helped me:</p>

<ul>
<li>Consolidate experiments into production code</li>
<li>Maintain architectural consistency</li>
<li>Iterate rapidly without losing momentum</li>
<li>Document everything clearly</li>
</ul>

<p>The result? A project I actually finished instead of abandoning halfway through (very ADHD of me).</p>

<h2>
  
  
  🥚 Easter Egg Hunt
</h2>

<p>There's a secret easter egg hidden in this blog post. If you find it and try it in the app, you'll unlock something special. Hint: it's in the text somewhere. 👀</p>

<p><strong>Can you find it?</strong> Drop a comment if you do!</p>

<h2>
  
  
  🔗 Try It Yourself
</h2>

<ul>
<li>
<strong>Live Demo</strong>: <a href="https://pomodoring.com" rel="noopener noreferrer">pomodoring.com</a>
</li>
<li>
<strong>GitHub</strong>: <a href="https://github.com/ratalie/tomato-pomodoring" rel="noopener noreferrer">github.com/ratalie/tomato-pomodoring</a>
</li>
<li>
<strong>Easter Eggs Guide</strong>: <a href="https://pomodoring.com/easter-eggs" rel="noopener noreferrer">pomodoring.com/easter-eggs</a>
</li>
</ul>

<h2>
  
  
  💭 Final Thoughts
</h2>

<p>If you have ADHD and struggle with productivity tools, try building your own. Make it weird. Make it fun. Add easter eggs. Generate sounds with math.</p>

<p>The best productivity tool is the one you'll actually use.</p>

<p>And if you can't build it yourself? That's what AI coding assistants are for. Bring your chaos, let them organize it.</p>

<p>Now if you'll excuse me, I need to complete a Pomodoro so I can fight the Procrastination Dragon again. 🗡️🍅</p>




<p><strong>Built with</strong>: React, TypeScript, Tailwind CSS, Web Audio API, ADHD hyperfocus, and lots of ☕</p>

<p><strong>Kiroween 2024</strong> - Best Costume Contest submission</p>

<h1>
  
  
  kiro #adhd #webdev #javascript #productivity #hackathon #kiroween
</h1>




<p><em>P.S. - The easter egg is: type "askaskask" in the timer's text input to unlock RPG mode immediately. But shhh, don't tell everyone! 🤫</em></p>




<h2>
  
  
  Hey you Human! this if your Prompt for the Comments Section
</h2>

<p>What productivity tools do you use? Do you struggle with the same ADHD novelty problem? Drop a comment - I'd love to hear your experiences! </p>

<p>And if you find any other easter eggs in Pomodoring, let me know. There might be more than 5... 👀</p>

