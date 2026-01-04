---
Title: 10 hours with Gas Town (out of a possible 48)
Description: 
Author: Mike Lady
Date: 2026-01-04T22:03:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>I am now paying $200/mo for Claude Code and that is a steal, or: how programming is ruined for me.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0bfrvxd2e5iwbko3beyf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0bfrvxd2e5iwbko3beyf.png" alt=" " width="800" height="427"></a><br>
Figure 1: What Piloting Gas Town Feels like, based on “<a href="https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04?utm_source=www.enterprisevibecode.com&amp;utm_medium=newsletter&amp;utm_campaign=10-hours-with-gas-town-out-of-a-possible-48&amp;_bhlid=da1401aba4e82f7cf7d0f19706a6c6e614bb5910" rel="noopener noreferrer">Figure 1: Welcome to Gas Town</a>”</p>

<p>Steve Yegge <a href="https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04?utm_source=www.enterprisevibecode.com&amp;utm_medium=newsletter&amp;utm_campaign=10-hours-with-gas-town-out-of-a-possible-48&amp;_bhlid=da1401aba4e82f7cf7d0f19706a6c6e614bb5910" rel="noopener noreferrer">dropped this article on us on the 1st of the year</a>. It is, at the very minimum, a wild introduction to his new coding agent orchestration framework. 48 hours later, I am now paying $200/month for Claude code. 💸 💸 💸 </p>

<p>He teased that this framework was coming out sometime in December, but I honestly had forgotten about it until I got the email notification from Medium for his blog. As soon as I saw it in my inbox, I was hooked. Rereading it again after using it in 10 of the last 48 hours, what he describes in the intro makes much more sense.</p>

<p>I will try to simplify, summarize, and just put in different words the most important bits from my experience going from newbie only generating 5 PRs in my first three hours to creating 36 PRs in my last four hours of using it.</p>

<p>Of course, don't take my word for it. PRs can be of various sizes and shapes. Lines of code is the worst metric to grade software developers on. Maybe to give a better sense, I was working on three to four different modules of my system (and Gas Town itself) at the same time! Within those modules, there were multiple workers (“Polecats”) sometimes working in parallel, depending on the Beads (lightweight agent issue tracker) that got planned.</p>

<p>"I gotta feeling / That tonight's gonna be a good night”</p>

<p>The Black Eyed Peas<br>
The Feeling<br>
Let me lead with how using Gas Town feels. As humans, our best tools become thoughtless extensions of ourselves. We put our shoes on at the beginning of the day, walk around all day, and forget that they are even on. We drive our cars, zone out, and become one with the flow of traffic. We pull out the pizza roller or ice cream scooper on Friday nights after a long work week, and they “just work” to deliver deliciousness and comfort.</p>

<p>Gas Town feels like these tools. It feels like all of the friction has been taken out of managing multiple agents. Dare I say all of the friction has been taken out of programming! The wind is fully in the sails of developing software.</p>

<p>Gas Town completely empties your queue of things that you want to do. You can just offload any little task you can think of into the orchestrator agent (aka “The Mayor”) and it figures out the planning, implementing, testing, and merging of all that work for you.</p>

<p>The focus is throughput: creation and correction at the speed of thought.</p>

<p>Steve Yegge<br>
You will run out of things to ask it to do and feel embarrassed. Your creativity and ability to track work in parallel will be challenged. The steady, patient, ever present hum of “The Refinery” merging your work into your project repo (or “rig” in gt parlance) main branches will leave you in awe.</p>

<p>You will be more excited to accomplish whatever you want with Vibe Coding in any language that you want. The only limit is how much you’re willing to spend on Claude Code tokens.</p>

<p>The Reality<br>
You will become bloodthirsty for tokens. There are not enough tokens in the world to satisfy all of the things you want your agents to do now that the pain of managing them has been dulled to a sufficient degree.</p>

<p>Yes, “dulled”. There is still some pain, but not nearly as bad as it was with manually resizing terminals and cursing your lack of screen real estate. You also won’t have to worry as much about bringing Agent context up to speed in project, managing git worktrees, nor caring about how close an Agent is to the end of its context window. That sharp pain is now replaced with a new, easier kind of friction that is much more tolerable.</p>

<p>The friction, aka, your job is to keep your Agents busy and unblocked. Or as I said on my <a href="https://www.youtube.com/live/d7KK_SbkuuY?si=l22xfBN1xiG-WV0O&amp;utm_source=www.enterprisevibecode.com&amp;utm_medium=newsletter&amp;utm_campaign=10-hours-with-gas-town-out-of-a-possible-48&amp;_bhlid=4f2f3a89fd7cfb69d2a5d2bb3bcd47db9747b73c" rel="noopener noreferrer">most recent livestream</a>, “keep your Tamagotchi alive”. We are all now managers of agents, as Steve Yegge and Gene Kim have mentioned this before in their book, Vibe Coding, but my experience with Gas Town really nails the point home.</p>

<p>You are now the manager of all the agents that you can keep track of and fit in your Claude Code subscription. Your management “span of control” is directly correlated to your attention span and memory. People are working on visualizations for the ongoing units of work called “Convoys”, but as of the first version you have to run a specific command over time to be reminded of all the work that is still outstanding or potentially even unaccounted for.</p>

<p>Author’s Note: While writing this newsletter, I <a href="https://github.com/steveyegge/gastown/pull/65?utm_source=www.enterprisevibecode.com&amp;utm_medium=newsletter&amp;utm_campaign=10-hours-with-gas-town-out-of-a-possible-48&amp;_bhlid=4fbf8698cb3271a00e2d3c9927da70dcd929acb7" rel="noopener noreferrer">became an official contributor to gastown</a> from my instance running in the background. It added an HTML report for Convoys while the preceding paragraph was being written about them. This is the power of Gas Town.</p>

<p>Since we are now more managers and less developers, we get to think about big picture ideas more and invest in the infrastructure to enable those ideas to happen autonomously. Very much like technical or even product managers, we suddenly have to be all about systems and processes.</p>

<p>For me, as a Senior DevOps Engineer, this is basically a dream come true. I've always felt that given an intense enough pipeline, cranked up to 11, we can ensure quality to the degree that developers feel like maintaining it.</p>

<p>Now your developers don't feel.</p>

<p>They don't complain. They get “slung” beads (“slinging” is the technical term for assigning work), and immediately start on the work and do as they are told. As long as Tmux hits “Enter” on the Claude prompt and doesn't get stuck [1].</p>

<p>Basic Workflow<br>
After following the <a href="https://github.com/steveyegge/gastown?tab=readme-ov-file&amp;utm_source=www.enterprisevibecode.com&amp;utm_medium=newsletter&amp;utm_campaign=10-hours-with-gas-town-out-of-a-possible-48&amp;_bhlid=6d16017cfb234b74485b33b09b75e143e2aeff34#quick-start" rel="noopener noreferrer">Quick Start Guide</a>, you may think you’re going to run a bunch of gt commands all over the place. You’d be mistaken. Once you attach to the Mayor and are in its Claude TUI, you can ask something like the following phrases in plain english (these happen to be mine, but feel free to use your own):</p>

<p>Work Management</p>

<ul>
<li>"show convoys" / "show status"</li>
<li>"what's blocked" / "what's ready"</li>
<li>"check on [polecat]"</li>
</ul>

<p>Polecat Management</p>

<ul>
<li>when things get stuck: "nudge polecats" / "nudge all workers" / "nudge everything"</li>
<li>"restart all"</li>
<li>"clean up polecats"</li>
</ul>

<p>Work Dispatch</p>

<ul>
<li>"sling X (description of what you want done) to a polecat" - assign tasks to polecats</li>
<li>"create convoy for X (full description of work to be done)" - batch related work</li>
<li>"sling all available beads"</li>
</ul>

<p>PR/Code Work</p>

<ul>
<li>"please fix [PR URL]"</li>
<li>"run code review on [PR]"</li>
<li>"address comments on [PR]"</li>
<li>"merge all X PRs into one"</li>
</ul>

<p>Infrastructure</p>

<ul>
<li>"upgrade gastown"</li>
<li>"run dashboard"</li>
</ul>

<p>Planning</p>

<ul>
<li>"use shiny formula to plan X"</li>
<li>"copy formulas from X to Y"</li>
</ul>

<p>Meta/Documentation</p>

<ul>
<li>"how many PRs today?"</li>
<li>"give me a workflow document"</li>
<li>"what are the typical commands?"
You can “jump out” from the Claude terminal control to the tmux bottom status line with Ctrl+b</li>
</ul>

<p>Once you’re on the tmux bottom status line, you can view the various tmux instances with just s, (so the whole command is Ctrl+b lift up finger from control, type s) and then use your arrow keys to move up and down, and hit enter to jump into another Claude tmux instance (that has a different role). You can hit esc in the s window to go back to your current tmux window.</p>

<p>You can use Ctrl+d to detach from the tmux session to go back to your normal terminal.</p>

<p>Final Thoughts from the first 48 hours<br>
I feel like I'm staring into the future by using this now mostly voice-activated method of programming since I'm using Wispr Flow. Yet, it is here today and I never have to go back to any prior form of programming, save for some probably really weird stuff.</p>

<p>The magic in Gas Town is Claude managing other instances of Claude via tmux, along with other orchestration wizardry that is similar to Kubernetes. Hopefully soon in the near future, we can use other types of coding Agents to manage even more different Agents using this framework.</p>

<p>I think the dirty little secret that the frontier model companies don't want you to think about is that life is better for the end consumer if we have access to multiple models. We get different perspectives on reviewing plans, PR reviews, and even choosing between different generated implementations. I don't believe there's going to be one model to rule them all and that's fine by me.</p>

<p>We benefit from treating these agents as if they're peer developers to get their perspectives. We can use those perspectives to triangulate a viable solution space to implement. Just like how companies don’t just hire "the one best developer” (because there isn't), we have teams of developers with their specialties, strengths, and weaknesses. Our strengths add up to greater than the sum of their parts, while our systems and processes cover over the holes left by our weaknesses.</p>

<p>My thesis behind Enterprise Vibe Code</p>

<p>Enterprise Vibe Code<br>
Humans + Agents are a team within a team of other Humans + Agents. The end goal (at a job) is to produce more business value than what they pay you. It doesn't matter if a Human or an AI wrote the code. If we have a strong process that transforms the unreliable inputs on the left (humans aren’t infallible, remember) into a reliable output (another Yegge-ism), we will have a successful business.</p>

<p>Subscribe to the free weekly newsletter at <a href="https://www.enterprisevibecode.com/" rel="noopener noreferrer">https://www.enterprisevibecode.com/</a></p>

<p>[1] Which is depressingly often where you give The Mayor the command to Sling and then you get distracted and move on to a different Convoy. You remember you slung it 30 minutes later, and you ask the mayor to remind you which Polecat that work went to and then use tmux to manually check and find the prompt sitting there in the Claude terminal unsubmitted. Then you have to curse the Mayor out for not starting it. The same also applies when Claude agents don't get started up in YOLO mode and they get stuck on the first permission check.</p>

