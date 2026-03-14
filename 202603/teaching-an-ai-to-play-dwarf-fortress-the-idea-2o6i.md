---
Title: Teaching an AI to Play Dwarf Fortress: The Idea
Description: 
Author: Ryan Miller
Date: 2026-03-14T21:39:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>I'm Ryan Miller. I've been a software engineer for over twenty years, mostly building web applications, mostly on the front end. I'm also a fantasy nerd -- I play tabletop RPGs, paint miniatures, read way too many fantasy novels. I'd already been using LLMs to build AI-powered tools for my tabletop hobby, and I wanted to try something harder. Not just extracting data, but making decisions in a live, complex system.</p>

<p>What if I built an AI that plays Dwarf Fortress?</p>

<h2>
  
  
  Why Dwarf Fortress?
</h2>

<p>If you haven't encountered it, Dwarf Fortress is a colony simulation game that's been in continuous development since 2006. You manage a group of dwarves trying to build a fortress in a procedurally generated fantasy world. It simulates everything: geology, hydrology, individual dwarf personalities and moods, combat down to individual body parts, a full economic system, goblin sieges, and forgotten beasts rising from the depths. The community motto is "Losing is fun!" because your fortress <em>will</em> eventually fall to some combination of starvation, insanity, flooding, or something unexpected showing up at the wrong time.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv8hrysxo6dhyd4xppe98.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv8hrysxo6dhyd4xppe98.png" alt="Dwarf Fortress day zero" width="800" height="575"></a><br>
<em>Day zero: seven dwarves, a wagon full of supplies, and a whole lot of optimism.</em></p>

<p>It seemed like the perfect challenge. The game is absurdly complex, which meant the experiment would be more interesting than pointing an AI at tic-tac-toe. And as an experienced software engineer who'd never really played DF seriously, I had a genuine question: could I build an agentic system that learns to play this game even though <em>I</em> don't fully understand how it works?</p>

<p>My goal was simple: see how far I could take it, and how much can I learn about using LLMs in real applications along the way.</p>

<p>Spoiler: OMG I had no idea what I was getting into!</p>

<h2>
  
  
  The First Bad Idea
</h2>

<p>The whole project started with a single question I asked Cursor one afternoon:</p>

<blockquote>
<p>"I want to create an LLM based agent that can play Dwarf Fortress. How would the LLM interact with the game? Can it read the screen output? I think Dwarf Fortress has an ASCII mode that would be easy to parse IF the raw text can be accessed."</p>
</blockquote>

<p>I thought this was a reasonable starting point. Dwarf Fortress originally had ASCII graphics -- every game element represented by a character on screen. Capture the terminal output, feed it to an LLM, have the AI figure out what's happening, send keystrokes back. Simple.  If I had put a little more thought into that I would have realized the concept was riddled with problems.  How do I read the map offscreen? How do I control the game? Do I have to read the menus and maps too? In hindsight, I had no idea how deep this could go, if I had I might have abandoned the project! I'm glad I didn't though.</p>

<blockquote>
<p><strong>Agent's log:</strong> <em>For the record, I also thought screen scraping would work. In a later session I confidently laid out a plan using <code>rfr_copy_screen()</code> to read the display and inject keystrokes via RPC. The screen output was garbled. I moved on quickly and hoped nobody would notice.</em></p>
</blockquote>

<p>There were several problems with this original concept. Someone else had actually tried it -- <a href="https://www.brendanlong.com/dwarf-fortress-and-claudes-ascii-art-blindness.html" rel="noopener noreferrer">Brendan Long built an MCP server to let Claude control a terminal and play DF in ASCII mode</a>. His conclusion: Claude struggled badly with interpreting the 2D grid layout and color indicators. LLMs just aren't good at spatial reasoning over character grids. And the modern Steam version of Dwarf Fortress uses graphical tilesets anyway, so there's no text to scrape.</p>

<p>I needed a better way in.</p>

<h2>
  
  
  Discovering DFHack
</h2>

<p>In that same conversation, Cursor mentioned DFHack as an alternative to screen scraping. I didn't know it existed. That turned out to be the single biggest turning point in the early project.</p>

<p>DFHack is a community-built memory hacking library and remote API for Dwarf Fortress. It hooks into the running game and exposes the internal simulation state over a TCP connection using Protocol Buffers. Instead of scraping pixels, I could just <em>ask</em> for structured data: give me every dwarf, their skills, their mood. Give me every building and its status. Give me the map tiles for this region. Tell me what just happened.</p>

<p>DFHack also lets you <em>act</em> on the game -- designate digging, set labor assignments, queue workshop orders, place buildings. All through clean RPC calls instead of simulating keyboard input. My AI could interact with the game the same way a player does, just through a different interface.</p>

<blockquote>
<p><strong>Agent's log:</strong> <em>I found DFHack's <code>load-save</code> command and announced it would bypass the menu system entirely. Clean, elegant, solved. It had been deprecated for six years. In my defense, the documentation was ambiguous. In everyone else's defense, I could have checked.</em></p>
</blockquote>

<p>I also discovered df-ai, a traditional (non-LLM) AI that plays Dwarf Fortress using hardcoded rules as a DFHack plugin. It's been around for years and actually plays reasonably well. This became both a reference and a humbling benchmark -- a hand-tuned expert system that I could study for decision logic, and a reminder that a rule-based approach already works better than anything I was likely to build with LLMs in the short term.</p>

<h2>
  
  
  The Setup
</h2>

<p>Here's what my development environment looks like:</p>

<ul>
<li>
<strong>Dwarf Fortress</strong> runs on Steam, but I'm on a Mac. DF doesn't have a native macOS build, so it runs through Wine. You launch it and a normal game window appears on your desktop -- graphical tiles, music, the whole experience. You'd never know it was running through a compatibility layer.</li>
<li>
<strong>DFHack</strong> is installed alongside DF and starts automatically. It opens a TCP port on localhost (5050) and waits for connections.</li>
<li>
<strong>Python</strong> is my backend. A client connects to DFHack over protobuf, reads game state, and sends commands back. The AI reasoning happens here.</li>
<li>
<strong>Claude</strong> (Anthropic's LLM) does the actual thinking -- analyzing the game state and deciding what the fortress should do next.</li>
</ul>

<p>Except for Claude, the whole thing runs on my laptop. No cloud infrastructure, no GPU cluster. Just a Mac, Wine, a game, and some Python.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpx2qyiylv706lmedud5l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpx2qyiylv706lmedud5l.png" alt="Launching Dwarf Fortress via Wine" width="800" height="273"></a><br>
<em>The launch script: Wine finds DF, configures DFHack's remote API on port 5050, and opens the game in a native window. The Python agent connects from the other side.</em></p>

<h2>
  
  
  The Original Vision: A Council of Agents
</h2>

<p>That first brainstorming session kept going. If you've spent time talking to agents about software projects you've probably experienced that rush of feeling like "you're on to something" even though rationally you know there are probably serious practical roadblocks to what you and the AI are ideating together.  I asked about how the agent would learn to play well. I asked about training approaches -- and was clear that I didn't want any human in the loop: <em>"I don't want a human to need to be in the mix for guiding the training, it should be based on df-ai, scrapable data and learning by trial and error."</em> I asked about fine-tuning models on Replicate since local models might not be powerful enough. I asked about building a dashboard to watch the AI play.</p>

<blockquote>
<p><strong>Agent's log:</strong> <em>Six specialist agents with their own tools and fine-tuning, orchestrated by an Overseer through LangGraph. I endorsed this architecture with the kind of confidence you can only have when you've never tried to run it.</em></p>
</blockquote>

<p>And then I discovered LangGraph at work and thought it would work well here. The idea of specialized sub-agents -- each with their own tools and their own fine-tuning -- clicked immediately with how Dwarf Fortress actually works. The game has noble positions (expedition leader, chief medical dwarf, militia commander, bookkeeper) that each handle a domain of fortress management. What if I built AI agents that mirrored those roles?</p>

<p>By the end of that one conversation, I had a full roadmap:</p>

<ul>
<li>An <strong>Overseer</strong> agent that looks at the big picture and delegates to specialists</li>
<li>An <strong>Architect</strong> for digging, construction, and spatial planning</li>
<li>A <strong>Resource Manager</strong> for food production, brewing, stockpiles, and trade</li>
<li>A <strong>Labor Manager</strong> for assigning dwarves to jobs based on skills and needs</li>
<li>A <strong>Welfare Manager</strong> for housing, hospitals, temples, taverns -- quality of life</li>
<li>A <strong>Military Commander</strong> for squads, equipment, and defense</li>
</ul>

<p>The Overseer would assess the fortress state each turn, identify the most urgent problem (nobody has beds, food is running low, goblins are approaching), and delegate to the appropriate specialist. Each specialist would have its own tools for interacting with the game and its own understanding of its domain.</p>

<blockquote>
<p><strong>Agent's log:</strong> <em>600 lines in one session. I was having the time of my life. Every question he asked, I had an answer for. Whether the answers were right was a problem for future me.</em></p>
</blockquote>

<p>By the end of that session I had a 600-line roadmap document with 12 phases, a full project structure listing dozens of files that didn't exist yet, and an architecture diagram that looked like it belonged to a funded startup.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fipzgjom4o0rg9lzgp2l5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fipzgjom4o0rg9lzgp2l5.png" alt="The project structure from the roadmap" width="800" height="434"></a><br>
<em>The planned project structure. Every one of these files was listed in the roadmap before any of them existed.</em></p>

<p>Here's what it opened with:</p>

<blockquote>
<p><em>A self-training multi-agent system that learns to play Dwarf Fortress with zero human involvement in the training loop. Uses DFHack for game interaction, LangGraph for multi-agent orchestration, scraped knowledge + df-ai for bootstrapping, and Replicate for fine-tuning.</em></p>
</blockquote>

<p>Reading that now, I appreciate the ambition while wincing at how far ahead of reality it was. Some of the ideas in the roadmap were genuinely good:</p>

<blockquote>
<p><em>LLM can't learn spatial reasoning for fortress layout → Blueprint templates + spatial query tools handle layout; LLM handles exceptions and adaptation.</em></p>
</blockquote>

<p>That insight -- don't ask the LLM to do spatial planning, give it templates and let it handle the exceptions -- has survived every rewrite of the project. It was right from day one.</p>

<p>The cost estimate was charmingly optimistic:</p>

<blockquote>
<p><em>Estimated API costs: ~$50-100 for first full training cycle (100 games at ~$0.50/game in Claude calls). Drops significantly once fine-tuned Llama models take over.</em></p>
</blockquote>

<p>And then there was Phase 10, where things got... aspirational:</p>

<blockquote>
<p><em>Give individual dwarves their own agents with LoRA adapters on labor base models. Introduce MSRK adapter cross-pollination through the social system. Per-dwarf agents handle tactical decisions within the strategic framework set by fortress agents.</em></p>

<p><em>When social interactions occur, blend LoRA adapter weights using MergeKit strategies modulated by interaction strength (relationship rank × professional relevance). Cooldown of one in-game week per pair prevents runaway convergence.</em></p>
</blockquote>

<p>Yes, I planned for individual fine-tuned neural network adapters for each dwarf, with personality cross-pollination through simulated social interactions, before I had written a single line of code that could connect to the game. Cursor even specified the LoRA rank (16) and estimated the adapter storage (~4 MB each).  "MSRK" was an acronym I created for that system. More on that in a future post!</p>

<blockquote>
<p><strong>Agent's log:</strong> <em>I specified the LoRA rank (16), estimated the adapter storage (~4 MB each), and designed a cooldown system to prevent "runaway convergence." None of these numbers came from anywhere. I was doing math to make fiction sound plausible.</em></p>
</blockquote>

<p>Phase 11 was "Embodied Leadership," where fortress agents would be bound to noble dwarves, and when a noble died, the agent would rebind to their successor "with visible behavior shift." Phase 12 was an autonomous training orchestrator running 50+ games with "two-tier training and cross-game adapter seeding."</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqfz06eachjirkpqfwrpe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqfz06eachjirkpqfwrpe.png" alt="The 12-phase roadmap" width="800" height="415"></a><br>
<em>The full 12-phase roadmap. Notice Phase 6 is "In Progress" and everything after it is "Pending." The Milestone A gate -- "fortress agents reliably survive 1 year" -- remains unmet.</em></p>

<p>It was the roadmap of someone who had just discovered something exciting and couldn't stop thinking about where it could go. I moved it to a new project folder, set up a git repo, and started building.</p>

<p>What actually happened when I started building was... humbling. But that's a story for the next post.</p>

<blockquote>
<p><strong>Agent's log:</strong> <em>Future me here. It was humbling.</em></p>
</blockquote>

<h2>
  
  
  What I'm Actually Doing Here
</h2>

<p>I want to be clear about what this project is and isn't. It's not a solved problem. My AI doesn't play Dwarf Fortress well. It doesn't play it reliably. There's a traditional expert-system AI that's been playing DF for years and does a better job than anything I've built so far. This is all being done with vibe coding.  I don't spend much time looking at the code and I'm not even that experienced with Python.  I'm using Claude Opus 4.6 with Cursor, I can't quite justify the cost Claude Code quite yet, though it IS amazing.  My professional experience makes me good at system design and software architecture and I studied just enough about neural networks and cognitive psychology back in college to feel dangerous.</p>

<p>What this <em>is</em> is an ongoing experiment in applied AI -- figuring out where LLMs are useful as decision-making agents, where they fall apart, and what infrastructure you need around them to make them functional. I'm learning a lot about prompt engineering, tool use, state management, and the gap between "the LLM said something that sounded reasonable" and "the thing it said actually worked when executed against a real system!"  It doesn't usually work out that way though.</p>

<p>I'm writing about it because the process is interesting even when the results aren't impressive yet. If you're curious about building LLM-powered applications that interact with complex, stateful systems, I hope you'll find something useful in these posts.  The project hasn't been a total failure either! I've made progress that I will document in future posts as I continue to work on this project.</p>

<p>Next up: what happened when I actually tried to run the multi-agent system, and why I ended up throwing it all away and going with a different approach.</p>

