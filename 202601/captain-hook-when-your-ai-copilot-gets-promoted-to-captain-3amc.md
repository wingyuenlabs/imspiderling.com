---
Title: Captain Hook: When Your AI Copilot Gets Promoted to Captain!
Description: 
Author: Mátyás Kuti-Kreszács
Date: 2026-01-29T22:00:54.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>Ahoy, matey! 🏴‍☠️ Let me tell you a tale about <strong>Captain Hook</strong> – not the villain from Neverland, but the hero of your git repository!</p>

<p>You see, GitHub Copilot usually sits beside you as a friendly <strong>copilot</strong>, offering helpful suggestions. But what if your copilot got <strong>promoted to captain</strong> and started giving you <em>orders</em>? What if it didn't just suggest – it <em>enforced</em>? </p>

<p>That's exactly what I built: an intelligent git pre-commit <strong>hook</strong> that acts like a <strong>captain</strong>, commanding your code to ship-shape quality before it sets sail into your repository!</p>

<h3>
  
  
  🎣 The Hook That Captains Your Code
</h3>

<p><strong>Captain Hook</strong> is a git pre-commit hook with a personality problem (in the best way). It:</p>

<ul>
<li>
<strong>📋 Linting</strong> (ruff check) - <em>"Arr! Walk the plank, unused imports!"</em>
</li>
<li>
<strong>🎨 Formatting</strong> (ruff format) - <em>"Straighten those lines, ye scurvy code!"</em>
</li>
<li>
<strong>🔎 Type Checking</strong> (mypy) - <em>"Where be yer type hints, sailor?"</em>
</li>
<li>
<strong>🧪 Tests</strong> (pytest) - <em>"No code leaves port without passing inspection!"</em>
</li>
</ul>

<h3>
  
  
  ⚓ From Copilot to Captain
</h3>

<p>Here's the twist that makes this project special:</p>

<p><strong>Normal Copilot</strong>: "Hey, maybe you could fix this import?" 😊</p>

<p><strong>Captain Hook</strong>: "HALT! Unused import detected! Removing it NOW!" 💪</p>

<p><strong>Normal Copilot</strong>: "Consider adding type hints here..." 🤔</p>

<p><strong>Captain Hook</strong>: "NO TYPE HINTS? Calling in backup! <em>blows whistle</em> GitHub Copilot CLI, report for duty!" 🎺</p>

<p>When Captain Hook encounters something it can't auto-fix (type errors, test failures), it doesn't just block your commit and walk away. Oh no! It calls in the <strong>GitHub Copilot CLI</strong> – think of it as calling in the <em>Admiral</em> for strategic advice!</p>

<h3>
  
  
  🏴‍☠️ Why "Captain Hook"?
</h3>

<p>Because this git <strong>hook</strong> acts like a <strong>captain</strong>:</p>

<ul>
<li>
<strong>Hook</strong> (git pre-commit mechanism) → Technical term</li>
<li>
<strong>Captain</strong> (commands and enforces) → How it behaves</li>
<li>
<strong>Captain Hook</strong> → The perfect nautical pun! 🎣</li>
</ul>

<p>Plus, just like the infamous Captain Hook, this tool:</p>

<ul>
<li>✅ Never lets Peter Pan (bad code) escape</li>
<li>✅ Has a trusty sidekick (GitHub Copilot CLI)</li>
<li>✅ Commands the crew (QA tools) with authority</li>
<li>✅ Protects the ship (your repository) fiercely</li>
</ul>

<h3>
  
  
  🚢 What It Means to Me
</h3>

<p>As developers, we often treat CI/CD and code quality tools as annoying gatekeepers – the grumpy pirates who say "NO!" to everything. But what if they were more like a wise captain who:</p>

<ul>
<li>Fixes what can be fixed automatically (80% of issues)</li>
<li>Teaches you how to fix complex problems (using AI)</li>
<li>Keeps the ship running smoothly (maintains quality)</li>
<li>Actually makes your life <em>easier</em>, not harder</li>
</ul>

<p><strong>Captain Hook</strong> represents a shift from "code quality as punishment" to "code quality as a helpful mentor with a sense of humor and a pirate hat."</p>

<h2>
  
  
  Demo
</h2>

<p>🔗 <strong>Live Repository</strong>: <a href="https://github.com/thec0dewriter/captain_hook.git" rel="noopener noreferrer">https://github.com/thec0dewriter/captain_hook.git</a></p>

<h3>
  
  
  🗺️ Project Repository Structure:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>captain_hook/
├── src/captain_hook/         # The treasure (Python library code)
├── tests/                    # Quality inspection (100% coverage)
├── hooks/                    # The Captain's quarters! ⚓
│   ├── pre-commit           # Captain Hook himself
│   ├── install.sh           # Recruit the captain
│   ├── demo.sh              # Training exercises
│   └── README.md            # Captain's orders
├── submission.md            # This very tale!
├── HOOK_SUMMARY.md          # The Captain's log
└── README.md                # Ship's manifest
</code></pre>

</div>



<h3>
  
  
  ⚓ Captain Hook in Action:
</h3>

<p><strong>Example 1: The Captain Takes Command</strong> 💪</p>

<p>Picture this: You try to commit some messy code...<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>git commit <span class="nt">-m</span> <span class="s2">"Add new feature"</span>

🔍 Running pre-commit QA checks...

📋 Step 1/4: Linting <span class="o">(</span>ruff check<span class="o">)</span>
✗ Linting failed
  - Found unused imports: os, sys, json
  - Missing spaces around operators

⚓ <span class="s2">"AVAST, YE SCURVY CODE!"</span> - Captain Hook

Attempting auto-fix with ruff...
✓ Auto-fixed linting issues
✓ Unused imports walked the plank!
✓ Spaces restored to their rightful place!
Staging auto-fixed files...

🎨 Step 2/4: Format checking <span class="o">(</span>ruff format<span class="o">)</span>
✗ Format check failed

⚓ <span class="s2">"This code be messier than a pirate's beard!"</span>

Auto-formatting files...
✓ Files formatted
✓ Your code now looks shipshape!
Staging formatted files...

🔎 Step 3/4: Type checking <span class="o">(</span>mypy<span class="o">)</span>
✓ Type checking passed

⚓ <span class="s2">"Good! Proper type hints, as any respectable sailor knows!"</span>

🧪 Step 4/4: Running tests <span class="o">(</span>pytest<span class="o">)</span>
✓ Tests passed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ All pre-commit checks PASSED
⚓ <span class="s2">"Permission granted! Your code may set sail!"</span>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
</code></pre>

</div>



<p><strong>Example 2: When the Captain Calls for Backup</strong> 🎺</p>

<p>Sometimes even captains need help from the admiral...<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>git commit <span class="nt">-m</span> <span class="s2">"Add calculator"</span>

🔍 Running pre-commit QA checks...

📋 Step 1/4: Linting <span class="o">(</span>ruff check<span class="o">)</span>
✓ Linting passed

🎨 Step 2/4: Format checking <span class="o">(</span>ruff format<span class="o">)</span>
✓ Format check passed

🔎 Step 3/4: Type checking <span class="o">(</span>mypy<span class="o">)</span>
✗ Type checking failed
src/calculator.py:4: error: Function is missing a <span class="nb">type </span>annotation

⚓ <span class="s2">"Blimey! Missing type hints be beyond me fixing powers!"</span>
⚓ <span class="s2">"Calling in the ADMIRAL for strategic advice!"</span>

⚠️ Attempting auto-fix with GitHub Copilot CLI...

🎺 <span class="k">*</span>Admiral GitHub Copilot CLI reports <span class="k">for </span>duty<span class="k">*</span>

💡 Copilot suggests:
   <span class="s2">"Add type annotations to function parameters and return type:

    def calculate(a: float, b: float) -&gt; float:
        return a + b"</span>

⚓ <span class="s2">"There ye have it! Follow the Admiral's orders, sailor!"</span>

Tips:
  - Apply the Admiral<span class="s1">'s wisdom (Copilot'</span>s suggestion<span class="o">)</span>
  - Run <span class="s1">'make check'</span> to verify
  - Try committing again
</code></pre>

</div>



<h3>
  
  
  🏴‍☠️ Real Voyage Testing
</h3>

<p>The Captain was battle-tested with intentionally broken code:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Challenge</th>
<th>Captain's Response</th>
<th>Outcome</th>
</tr>
</thead>
<tbody>
<tr>
<td>Unused imports (3x)</td>
<td>"Walk the plank!"</td>
<td>✅ Auto-removed</td>
</tr>
<tr>
<td>Bad spacing <code>x,y</code>
</td>
<td>"Mind yer spaces!"</td>
<td>✅ Fixed to <code>x, y</code>
</td>
</tr>
<tr>
<td>Messy formatting</td>
<td>"Swab the deck!"</td>
<td>✅ Auto-formatted</td>
</tr>
<tr>
<td>Missing type hints</td>
<td>"Call the Admiral!"</td>
<td>✅ Copilot suggested fix</td>
</tr>
<tr>
<td>Code ready to commit</td>
<td>"Set sail!"</td>
<td>✅ Smooth sailing</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  🗺️ Quick Start: Recruit Your Captain
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Clone the ship</span>
git clone https://github.com/thec0dewriter/captain_hook.git
<span class="nb">cd </span>captain_hook

<span class="c"># Stock the supplies (install dependencies)</span>
make <span class="nb">install</span>

<span class="c"># Recruit Captain Hook to your crew</span>
make install-hooks

<span class="c"># Make a change and try to commit</span>
<span class="nb">echo</span> <span class="s2">"import os"</span> <span class="o">&gt;&gt;</span> src/captain_hook/calculator.py
git add <span class="nb">.</span>
git commit <span class="nt">-m</span> <span class="s2">"Test the Captain"</span>

<span class="c"># Watch Captain Hook spring into action! 🎣</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiqshb4v0ai3n50l7mko6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiqshb4v0ai3n50l7mko6.png" alt="Working screenshot" width="800" height="836"></a></p>

<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<h3>
  
  
  🎭 The Plot Twist: From Copilot to Captain
</h3>

<p>When I started this project, I thought I was building a simple pre-commit hook. But something magical happened: <strong>the copilot became the captain</strong>.</p>

<p>Here's how GitHub Copilot CLI transformed this project:</p>

<h3>
  
  
  1️⃣ <strong>The Captain's Wisdom: Intelligent Error Resolution</strong>
</h3>

<p><strong>Before Copilot CLI:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Error: Function missing type annotation
*stares at screen*
*Googles "python type hints"*
*reads docs for 10 minutes*
*tries something*
*fails*
*repeats*
</code></pre>

</div>



<p><strong>With Captain Hook + Copilot CLI:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>⚓ "Type hints be missing! Calling the Admiral!"
💡 Copilot: "Add type hints like this: def foo(x: int) -&gt; int:"
✅ "Brilliant! Fixed in 30 seconds!"
</code></pre>

</div>



<p>It's like having a <strong>ship's navigator</strong> who actually knows where you're going instead of just pointing at the horizon!</p>

<h3>
  
  
  2️⃣ <strong>Building the Captain: Meta-Development</strong>
</h3>

<p>The beautiful irony? I used GitHub Copilot CLI to build a tool that <em>uses</em> GitHub Copilot CLI!</p>

<p><strong>Inception moment:</strong></p>

<ul>
<li>Used Copilot to write the hook</li>
<li>The hook calls Copilot CLI for help</li>
<li>It's copilots all the way down! 🎭</li>
</ul>

<p>Copilot helped me:</p>

<ul>
<li>Scaffold the entire git hook structure</li>
<li>Write robust bash error handling</li>
<li>Design the captain personality (yes, the pirate puns!)</li>
<li>Generate comprehensive documentation</li>
<li>Create engaging demo scenarios</li>
</ul>

<h3>
  
  
  3️⃣ <strong>The Admiral's Strategy: Learning Through Collaboration</strong>
</h3>

<p>Working with Copilot CLI felt less like "using a tool" and more like <strong>pair programming with a pirate crew</strong>:</p>

<ul>
<li>
<strong>The Captain</strong> (my hook) → Commands and enforces</li>
<li>
<strong>The Admiral</strong> (Copilot CLI) → Provides strategic advice</li>
<li>
<strong>The Crew</strong> (ruff, mypy, pytest) → Do the actual work</li>
<li>
<strong>Me</strong> (the developer) → Enjoys smooth sailing! ⛵</li>
</ul>

<p>Best part? When Captain Hook calls Copilot CLI, it's not just showing error messages – it's providing <strong>contextual wisdom</strong> based on the actual code and error.</p>

<h3>
  
  
  4️⃣ <strong>Real-World Impact: The 80/20 Rule of Joy</strong>
</h3>

<p>Here's the magic formula Captain Hook delivers:</p>

<ul>
<li>
<strong>80% of problems</strong>: Auto-fixed instantly (bye bye, formatting fights!)</li>
<li>
<strong>20% of problems</strong>: Copilot CLI teaches you the solution</li>
<li>
<strong>100% of the time</strong>: You stay in flow state 🌊</li>
</ul>

<p><strong>Old workflow:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Write code → Commit → CI fails → Context switch → Google → Fix → Repeat
😤 Frustration level: High
</code></pre>

</div>



<p><strong>Captain Hook workflow:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Write code → Commit attempt → Captain fixes most issues → 
Copilot teaches remaining fixes → Commit succeeds
😊 Frustration level: "What frustration?"
</code></pre>

</div>



<h3>
  
  
  5️⃣ <strong>The Joke That Became Real</strong>
</h3>

<p>Started as: "Haha, what if git hooks had personality?"</p>

<p>Became: "Wait, this is actually the future of developer experience!"</p>

<p>The <strong>Captain Hook</strong> name started as a pun but revealed a deeper truth:</p>

<ul>
<li>Copilots <em>suggest</em> (helpful but passive)</li>
<li>Captains <em>command</em> (authoritative but caring)</li>
<li>The best tools do both! 🎯</li>
</ul>

<h3>
  
  
  🎬 Key Takeaway: It's About the Journey
</h3>

<p>GitHub Copilot CLI isn't just "AI that writes code faster." It's:</p>

<p>✨ <strong>A teaching tool</strong> that explains <em>why</em> fixes work<br>
🚢 <strong>A workflow enhancer</strong> that keeps you sailing smoothly<br><br>
🎣 <strong>A captain</strong> that catches problems before they cause shipwrecks<br>
💡 <strong>A collaboration partner</strong> that makes coding more fun</p>

<p><strong>Captain Hook proves that AI in development isn't about replacing developers – it's about promoting your helpful copilot to a wise captain who still lets you steer the ship!</strong></p>


<h3>
  
  
  🎯 Technologies Used:
</h3>

<ul>
<li>Python 3.9+ with UV package manager (fastest ship in the fleet!)</li>
<li>Ruff (linting &amp; formatting) (the ship's cleaning crew)</li>
<li>Mypy (type checking) (the quality inspector)</li>
<li>Pytest (testing with coverage) (safety drills)</li>
<li>Bash scripting (the ship's machinery)</li>
<li>Git hooks (the captain's perch)</li>
<li>
<strong>GitHub Copilot CLI</strong> (the wise admiral) 🎖️</li>
</ul>
<h3>
  
  
  📋 Current Status &amp; Future Voyages
</h3>

<p><strong>✅ What's Shipshape:</strong></p>

<ul>
<li>Intelligent pre-commit hook with auto-fix</li>
<li>GitHub Copilot CLI integration</li>
<li>100% test coverage (8 passing tests)</li>
<li>Comprehensive documentation</li>
<li>Real-world testing completed</li>
</ul>

<p><strong>🚧 Potential Future Adventures (TODOs):</strong></p>

<ul>
<li>[ ] Add more "captain personality" responses for different error types</li>
<li>[ ] Support for commit-msg hook (Captain checks your commit messages!)</li>
<li>[ ] Integration with more language ecosystems (TypeScript, Go, Rust)</li>
<li>[ ] "Parrot mode" - hook verbosity level (quiet parrot vs. chatty parrot 🦜)</li>
<li>[ ] Collectible "achievements" when you fix issues</li>
<li>[ ] Ship's log (detailed commit history with captain's commentary)</li>
</ul>

<p><strong>Try it yourself:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/thec0dewriter/captain_hook.git
<span class="nb">cd </span>captain_hook
make <span class="nb">install
</span>make install-hooks

<span class="c"># Let Captain Hook command your code to quality! ⚓</span>
</code></pre>

</div>






<p><strong>Built with ❤️ (and lots of pirate puns) for the GitHub Copilot CLI Challenge</strong></p>

<p><em>"Not all those who wander are lost, but all code that wanders needs a captain!"</em> 🎣⚓</p>

