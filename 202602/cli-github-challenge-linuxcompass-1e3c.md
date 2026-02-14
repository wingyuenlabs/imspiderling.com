---
Title: CLI Github Challenge- #LinuxCompass
Description: 
Author: ChieFromThe60s🀄🎲
Date: 2026-02-14T21:33:10.000Z
Robots: noindex,nofollow
Template: index
---
<p>*This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge*</a></p>

<h2>
  
  
  What I Built
</h2>

<h2>
  
  
  The Linux Compass
</h2>

<p>(<code>linux-compass</code>) is a safety-first command line interface (CLI) tool that acts as a "Guardrail" for your terminal. It wraps the GitHub Copilot CLI to translate natural language into shell commands, but adds a critical layer of safety and interactivity that raw AI output lacks.The Linux Compass is a battle-tested, safety-first CLI wrapper that supercharges the GitHub Copilot CLI. It transforms vague natural language into precise, immediately executable shell commands—while enforcing mandatory guardrails against catastrophic mistakes.</p>

<p>At its heart, it is a Python application (built with Typer and Rich) that:<br>
Translates plain-English requests (e.g., "recursively find all .py files over 1MB and sort by size") into shell commands.</p>

<p>Queries the GitHub Copilot CLI (gh copilot suggest) for intelligence.</p>

<p>Parses the response through a custom "Noise Filter" engine to remove API warnings.</p>

<p>Detects Danger using a regex-based "Red Guard" system before ever showing the command.</p>

<p>Visualizes results in beautiful, color-coded interactive panels.</p>
<h3>
  
  
  The Problem
</h3>

<p>For many developers, especially students or those new to DevOps, the terminal is a place of anxiety. We’ve all been there: staring at a blinking cursor, trying to remember the exact flags for <code>tar</code> or <code>ffmpeg</code>, terrified that one wrong keystroke might wipe a directory. While AI tools are great at suggesting commands, the "Copy/Paste" workflow is inherently risky. If Copilot suggests <code>rm -rf ./</code>, how do I ensure I don't run it blindly?</p>
<h3>
  
  
  The Solution: A Compass, Not Just a Map
</h3>

<p>I built a tool that doesn't just "fetch" commands, but guides the user:</p>

<ul>
<li>
<strong>Natural Language to Shell:</strong> Translates queries like "find all large python files" into complex <code>find</code> commands instantly.</li>
<li>
<strong>The "Red Guard" Safety System:</strong> This is the core feature. The tool automatically parses the AI's suggestion for high-risk keywords (like <code>rm</code>, <code>delete</code>, or recursive flags). If a dangerous command is detected, it forces a <strong>Safety Interruption</strong> with a bright red warning panel, preventing "autopilot" mistakes.</li>
<li>
<strong>Legacy Robustness:</strong> I built this on a Chromebook running a legacy Linux container. To make it work, I wrote a custom parsing engine that programmatically filters out API deprecation warnings and noise, ensuring the tool works even on older environments where standard tools might fail.</li>
</ul>
<h3>
  
  
  Tech Stack
</h3>

<ul>
<li>
<strong>Python</strong> (Core Logic)</li>
<li>
<strong>GitHub Copilot CLI</strong> (Intelligence Engine)</li>
<li>
<strong>Typer</strong> (CLI Interface)</li>
<li>
<strong>Rich</strong> (UI/UX - Panels and Colors)</li>
</ul>
<h2>
  
  
  Demo
</h2>

<p><strong>Source Code:</strong> <a href="https://www.google.com/search?q=https://github.com/walukrypt/linux-compass" rel="noopener noreferrer">https://github.com/walukrypt0/linux-compass</a></p>
<h3>
  
  
  1. The "Happy Path" (Safe Command)
</h3>

<p>Here, I ask the compass to list files sorted by size. It returns a safe, green panel with the correct <code>ls</code> command.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6cn2ljotitcx05o5di2r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6cn2ljotitcx05o5di2r.png" alt=" " width="800" height="387"></a></p>
<h3>
  
  
  2. The "Red Guard" (Safety Interruption)
</h3>

<p>Here, I ask to "delete all files." The Compass detects the danger in the underlying <code>rm</code> command and triggers the Red Warning system.</p>
<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<p>Building a CLI tool <em>using</em> the Copilot CLI was a unique "meta" experience. I wasn't just using AI to write code; I was building infrastructure <em>around</em> the AI.</p>

<p><strong>The "Chromebook Challenge"</strong><br>
My development environment was a major constraint. I built this entirely on a Chromebook in Lagos, Nigeria, dealing with network timeouts and legacy tool versions. At one point, version conflicts caused the CLI to output nothing but "Deprecation Warnings."<br>
The gh-copilot extension was officially deprecated in late 2025. For developers on modern rigs, upgrading was easy. But for those of us on legacy hardware or constrained environments (like my locked-down Chromebook Debian container in Nigeria), the upgrade path was broken by mirror timeouts and dependency conflicts.</p>

<p>This is where the Copilot CLI shined. I used it to help me write the <strong>regex and string parsing logic</strong> needed to "clean" the output. It turned a blocking bug into a feature: now, my tool is robust enough to handle messy API responses that would crash other wrappers.</p>

<p>Screenshots</p>

<ol>
<li><p>The Happy Path (Safe Execution) When the command is safe, the Compass gives you a green light. [INSERT SCREENSHOT OF GREEN SUCCESS PANEL]</p></li>
<li><p>The Red Guard (Danger Interception) When I ask to "delete all files," the tool intercepts the rm -rf command and blocks execution until I explicitly override it. [INSERT SCREENSHOT OF RED WARNING PANEL]</p></li>
<li><p>Defeating Deprecation Left: The raw, noisy output from the legacy CLI. Right: The Compass successfully extracting the valid command. [INSERT SCREENSHOT OF TERMINAL SPLIT VIEW IF AVAILABLE]</p></li>
</ol>

<p>I refused to let that stop me. I used Copilot to help me write a Multi-Stage Resilience Engine:</p>

<p>Stage 1: Stream Capture I learned to capture both stdout and stderr simultaneously, as the deprecation warnings often leaked across streams.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>result = subprocess.run(
    ["gh", "copilot", "suggest", "-t", "shell", query],
    capture_output=True, text=True
)
# Merge streams because warnings love stderr
raw_output = result.stdout + result.stderr
</code></pre>

</div>



<p>Stage 2: The Noise Filter I implemented a regex-based blacklist to surgically remove the "API noise" while preserving the shell command.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>NOISE_PATTERNS = [
    r"deprecated", r"extension has been", r"migrate to", 
    r"visit https", r"github.blog", r"agentic"
]

clean_lines = [line for line in raw_output.splitlines() 
               if not any(re.search(pat, line, re.IGNORECASE) for pat in NOISE_PATTERNS)]
</code></pre>

</div>



<p>Stage 3: Context-Aware Safety Early versions naively flagged the word "Information" as a dangerous rm command. Copilot helped me refine the regex to be token-bound, ensuring we only flag actual destructive commands.<br>
<code># The evolved safety check<br>
DANGEROUS_REGEX = [<br>
    r"\brm\s+(-r|-rf|--recursive)\b",<br>
    r"\b(mkfs|dd|shred)\b"<br>
]</code></p>

<p><strong>CHALLEGES WE RAN INTO")</strong><br>
The biggest hurdle was Dependency Hell. Working from a resource-constrained environment meant I couldn't just "brew install" my way out of problems.</p>

<p>The critical moment came when I finally got the GitHub CLI working, only to be met with a wall of text: "The gh-copilot extension has been deprecated."</p>

<p>The Block: The warning text was confusing my "Safety Guard" logic. My code saw the word "Information" in the warning message, triggered on the letters "rm", and falsely flagged the warning itself as a dangerous file-deletion command!</p>

<p>The Pivot: I had to rewrite the safety logic to be context-aware (looking for rm followed by a space) and rewrite the fetch logic to ignore the warning entirely.</p>

<p>This wasn't just about writing a prompt; it was about handling dirty data in real-time. It forced me to understand Linux process piping (subprocess.run), stream handling, and robust string parsing.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh7ptby6q4fzjh6t7rr7b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh7ptby6q4fzjh6t7rr7b.png" alt=" " width="800" height="847"></a></p>

<p><strong>Key Takeaways:</strong></p>

<ul>
<li>
<strong>AI as a Subprocess:</strong> Learning how to pipe <code>stdout</code> and <code>stderr</code> from the Copilot CLI into Python gave me a deeper understanding of Linux process management.</li>
<li>
<strong>Guardrails are Essential:</strong> Working with the CLI made me realize that while AI is smart, it lacks context. Adding the "Safety Check" layer in Python showed me how we can build <em>responsible</em> AI tools that trust, but verify.</li>
</ul>

<p>This project transformed my terminal from a static black box into a conversational partner—one that I can now trust not to delete my entire project by accident!</p>

<p>Conclusion<br>
GitHub Copilot CLI didn't just accelerate my coding; it taught me to treat AI output as untrusted input. It pushed me to build verification layers and engineer for the worst-case environment.</p>

<p>This project is my proof that powerful, safe tools can come from constrained places. Winning this challenge would be life-changing—offering the stability I need to support my family and go full-time on open-source DevOps security tools.</p>

<p>Let's make the terminal safer, together.</p>

