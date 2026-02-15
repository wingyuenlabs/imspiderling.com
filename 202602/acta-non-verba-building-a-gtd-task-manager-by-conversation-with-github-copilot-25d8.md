---
Title: Acta Non Verba: Building a GTD Task Manager by Conversation with GitHub Copilot
Description: 
Author: Alin Pisica
Date: 2026-02-15T21:40:28.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>A while ago I read David Allen's Getting Things Done it and it kind of changed the way I do things.<br>
Seeing this challenge I felt that it is a perfect opportunity of combining the pleasure with work, and I created Acta.</p>

<p><strong>Acta</strong> is a command-line task manager built around the <a href="https://gettingthingsdone.com/" rel="noopener noreferrer">Getting Things Done (GTD)</a> methodology combined with my current needs (I use an external tool at the moment for tracking tasks, but I end up finding myself pretty often in the need of adding a new task right from the terminal, while I work - it removes friction as much as possible, and keeping my in my flow). The name comes from the Latin phrase <em>"Acta non verba"</em> - actions, not words - which felt fitting for a tool designed to get things off your mind and into motion.</p>

<p>The workflow mirrors the GTD capture-process-do loop:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4o3r34l8kzm12ono1vl4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4o3r34l8kzm12ono1vl4.png" alt=" " width="800" height="304"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CAPTURE → PROCESS → DO
   ↓         ↓       ↓
Inbox    Actionable  Work
</code></pre>

</div>



<ol>
<li>
<strong>Capture</strong> - Dump a task into the inbox instantly, with optional <code>@context</code> tags (e.g., <code>@project</code>, <code>@article</code>, <code>@research</code>)</li>
<li>
<strong>Process</strong> - Review the inbox: assign a due date and priority, or mark as done/remove</li>
<li>
<strong>Do</strong> - Work through tasks in priority order: overdue first, then today's, then upcoming</li>
<li>
<strong>List</strong> - Filter tasks by context, keyword, regex, or field (e.g., <code>priority:high</code>)</li>
<li>
<strong>Stats</strong> - Analytics dashboard: completion rate, context breakdown, overdue count, priority distribution</li>
</ol>

<p>The app is built with <strong>.NET 10</strong>, <strong>Spectre.Console</strong> for the terminal UI, and compiled as a <strong>NativeAOT</strong> binary - a single, fast executable with no runtime dependency, cross-compiled for Linux, Windows, and macOS.</p>

<p>Didn't write a console app since university, and for a long time I wanted to try out Spectre.Console touches crept in. The interactive menu greets you with a FigletText banner reading <em>"Acta, non verba"</em> and I tried to integrate colors as much as I can, without making it disturbing.</p>

<p>What makes this project meaningful to me is that I didn't write most of the code. I <em>guided</em> it out of GitHub Copilot, one natural-language prompt at a time. It was a fun time and a perfect example of Copilot CLI's functionalities: it created a tool that I needed, based on my needs and requirements, with as little input as possible from my side.</p>




<h2>
  
  
  Demo
</h2>

<p><strong>Project Repository</strong>: <a href="https://github.com/alinpisica/acta-non-verba-gdt-cli" rel="noopener noreferrer">Acta - GitHub</a></p>

<p>Uploaded builds for specific distros under the <code>releases/</code> folder.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0uobxohax6qjev9gjb0f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0uobxohax6qjev9gjb0f.png" alt=" " width="800" height="640"></a></p>

<h3>
  
  
  The GTD loop in action
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Capture tasks anywhere, any time</span>
acta capture <span class="s2">"Renew car insurance @admin"</span>
acta capture <span class="s2">"Review PR #42 @work"</span>
acta capture <span class="s2">"Buy coffee beans @shopping"</span>

<span class="c"># Process the inbox (assign dates, priorities, remove noise)</span>
acta process

<span class="c"># Work through today's tasks</span>
acta <span class="k">do</span>

<span class="c"># Focus on a specific context</span>
acta <span class="k">do</span> @work

<span class="c"># Search and filter</span>
acta list <span class="s2">"@work priority:high"</span>
acta list <span class="s2">"/renew|review/"</span>      <span class="c"># regex</span>

<span class="c"># Analytics at a glance</span>
acta stats
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1wwf4ur7p0b8qouhjrki.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1wwf4ur7p0b8qouhjrki.png" alt=" " width="800" height="481"></a></p>

<h3>
  
  
  Interactive mode
</h3>

<p>Run <code>acta</code> with no arguments to get the full interactive menu:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  ___        _
 / _ \      | |
/ /_\ \ ___| |_ __ _   _ __   ___  _ __    __   _____ _ __| |__   __ _
|  _  |/ __| __/ _` | | '_ \ / _ \| '_ \   \ \ / / _ \ '__| '_ \ / _` |
| | | | (__| || (_| |_| | | | (_) | | | |   \ V /  __/ |  | |_) | (_| |
\_| |_/\___|\__\__,_(_)_| |_|\___/|_| |_|    \_/ \___|_|  |_.__/ \__,_|

── Productivity for the weak-willed ──────────────────────────────────────

What do you feel like doing?
&gt; Capture
  Process
  List
  Do
  Stats
  Help
  Exit
</code></pre>

</div>



<h3>
  
  
  What the task list looks like
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> #   Pri  Title                          Context        Due         Status
 ─   ───  ─────────────────────────────  ─────────────  ──────────  ──────
 1   ❗❗ Review PR #42                   @work         2026-02-15  →
 2   ❗   Write article for Dev.To        @article      2026-02-20  →
 3   ·    Upload content on Git           @dev          -           ○ ⏸2h
</code></pre>

</div>



<p><em>(Add a screenshot or recording before publishing)</em></p>




<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<p>What was fun? Well, everything.</p>

<p>I felt like I was the project manager, product owner and architect, while Copilot acted like an army of developers. I wrote the requirements in plain English; Copilot wrote the code, spotted its own bugs, refactored its own architecture, and generated its own documentation. What follows is the story of how that actually played out across three phases.</p>

<h3>
  
  
  Phase 1 - Building from scratch, prompt by prompt
</h3>

<p>I started with a simple instruction:</p>

<blockquote>
<p><em>"Create a new .NET 9 console project named Acta, add Spectre.Console package, and enable NativeAOT in the csproj file. The project will be a CLI-based tasks management based on 'Getting Things Done' idea."</em></p>
</blockquote>

<p>Copilot scaffolded the project correctly. After, I directed him in how the model should look, how data should be stored and how it should process the data. It went smoothly, with minimal corrections. I did not ask for his decision making, I made the calls, and only directed towards him as simply as possible, giving it plenty of room to juggle. I prompted it everythime to ask for clarification questions, if needed.</p>

<p>From there I fed it one feature at a time. Each prompt described the <em>behavior</em>, not the implementation:</p>

<ul>
<li><em>"Create a StorageService that reads and writes a list of TaskItem to a JSON file in the local AppData folder using System.Text.Json source generation."</em></li>
<li><em>"Populate the Capture choice. The user should be asked for the task's input text. He can provide context by specifying elements with '@' inside the task."</em></li>
<li><em>"Allow the user to also directly capture tasks using <code>acta capture "Insert task text here"</code> directly from CLI, without running the executable."</em></li>
<li><em>"Create the 'list' action. It should list elements based on filters or contexts. <code>@home</code> shows home tasks, <code>make bed @home</code> shows tasks matching 'make bed' with context <a class="mentioned-user" href="https://dev.to/home">@home</a>. Multiple contexts, multiple words, all lowercase."</em></li>
</ul>

<p>The <code>StorageService</code> prompt is worth highlighting. Copilot's first attempt failed to compile. More interesting: after the build error, it didn't just fix the error - it scanned the rest of the code for null-safety warnings it hadn't been asked to fix, and resolved them proactively. That was the moment I realized this wasn't just autocomplete.</p>

<p>By the end of Phase 1 the app had:</p>

<ul>
<li>capture (inline and interactive)</li>
<li>process (inbox review with date assignment)</li>
<li>list (context and keyword filtering)</li>
<li>(task queue with overdue → today → upcoming priority order).</li>
</ul>

<p>Everything was working and I had my "mini-tool" ready to go into my dev enviroment and day-to-day tasks.</p>

<h3>
  
  
  Phase 2 - Copilot refactors itself
</h3>

<p>As expected, from the initial phase, everything was in one place: commands, display logic, filtering, menu handling. I asked Copilot to look at it and generate summaries for everything that was existing.<br>
I identified the problems (tight coupling, mixed concerns, poor testability) based on his explanations (not leaving room for interpretation), proposed a structure (Commands / Services / Models / Utilities), and listed him the file structure I wanted - along with what would stay the same.</p>

<p>Then it did the refactoring:</p>

<ul>
<li>
<code>Program.cs</code> shrank from 354 lines to 54</li>
<li>9 new source files were created across 4 directories</li>
<li>
<code>TaskFilterService</code>, <code>TaskDisplayService</code>, <code>CommandDispatcher</code>, and <code>ContextParser</code> emerged as focused, single-responsibility classes</li>
<li>Every existing command worked identically after the change - zero breaking changes</li>
</ul>

<h3>
  
  
  Phase 3 - Copilot plans its own roadmap, then implements it
</h3>

<p>After the refactoring, went on the classical path of a software product: improve it. I asked him to produce a <code>nextsteps.md</code>: a prioritized list of missing features and UI improvements, each with impact assessment, effort estimate, and a specific list of files to modify. It even laid out a 5-phase implementation roadmap. I did not suggest anything, I wanted to test its limitations and see how far it could go. It amazed me. Did it create something revolutionary, that did not exist until now? No, of course, we all know how it works (right? right? it can't invent new things... YET). Did it create a list of features that improved the product, and it reduced my time of defining and describing the features? Yes.</p>

<p>I picked some of them and asked for implementations (I updated the .md file wiht my own changes).</p>

<ul>
<li>
<strong>Advanced search</strong>: regex syntax (<code>/pattern/</code>), fuzzy matching (<code>oml</code> → <code>Complete</code>), field-specific filters (<code>priority:high</code>, <code>context:@work</code>), exclusion with <code>-</code>
</li>
<li>
<strong>Task priority levels</strong>: Critical / High / Medium / Low / None, with visual indicators and color coding throughout the UI</li>
<li>
<strong>Snooze</strong>: defer tasks by 1h/2h/4h/tomorrow/next week/custom, with automatic filtering and a ⏸ indicator showing remaining time</li>
<li>
<strong>Analytics dashboard</strong>: completion rate, context distribution, priority breakdown, overdue counts - displayed as a formatted multi-table panel via Spectre.Console</li>
</ul>

<p>In the same pass it added colored status indicators, a task detail view, keyboard shortcut help, and context autocomplete infrastructure. 5 new services, 1 new command, 11 modified files - all building cleanly with zero warnings. In the meantime, I made a coffee.</p>

<h3>
  
  
  What I actually learned
</h3>

<p><strong>The prompt is the design doc.</strong> The quality of what Copilot produced was directly proportional to how precisely I described the <em>user experience</em> I wanted. Vague prompts produced vague code. "Allow me to filter by context" gave me something workable; "I can use <code>make bed @home</code> and it should show all elements containing 'make bed' with the <a class="mentioned-user" href="https://dev.to/home">@home</a> context, searched in any word order, case-insensitive" gave me exactly what I needed.</p>

<p><strong>It catches its own mistakes - if you give it feedback.</strong> The failed StorageService build was a normal failure mode. What wasn't normal was watching Copilot read the build output, identify the root cause, fix it, <em>and then</em> scan for related issues it hadn't been asked about.</p>

<p><strong>The developer becomes the curator.</strong> My job shifted from writing code to knowing what to ask for, recognizing when the output was right, and deciding when to push back. The code I <em>guided</em> into existence is better-structured and more thoroughly documented than most code I write from scratch, simply because Copilot's defaults lean toward clean architecture. All code was reviewed and checked by me. I do not think that any code written by or with an AI tool should end up in a production-ready environment without the manual review of a developer.</p>

<p>Fun fact: Copilot helped me review this, rephrase some weirdly mentioned items and corrected some grammar mistakes. It does more than coding, if used correctly. It does more than coding, if used under supervision. It can be an amazing tool, but no hammer has ever replaced the blacksmith.</p>




