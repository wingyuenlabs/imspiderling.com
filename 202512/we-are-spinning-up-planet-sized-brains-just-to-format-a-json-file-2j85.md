---
Title: We are spinning up planet-sized brains just to format a JSON file
Description: 
Author: Alexander Ertli
Date: 2025-12-01T21:37:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>🤖 🤖 🫧 🪡 Are we watching a rerun of 1987?<br><br>
(Yes, and it’s going to hurt exactly the same way)</p>

<h2>
  
  
  The God Model Fallacy: Why the AI Future Looks Like 1987
</h2>

<p>We are spinning up planet-sized brains just to format a JSON file. That's the God Model Fallacy in a nutshell. </p>

<p><strong>We’re in the Uncanny Valley:</strong> 90% on benchmarks, but the system still feels dumb in real life, and nothing truly works without heavy prompt massaging or multiple turns.</p>

<p>I spent the last 8 months (and most of my savings) architecting my own GenAI stack to learn if we engineers still have a future or should all quickly become founders for LLM-wrappers. </p>

<p><strong>Here is what I think now:</strong><br>
We are repeating the Lisp Machine collapse, frame by frame.</p>

<h3>
  
  
  1987 in two sentences
</h3>

<p>$100k specialized Lisp Machines were sold as the only way to run “real AI” (expert systems).<br><br>
Then normal Sun workstations did the same job for $20 k → every Lisp Machine company went to zero literally overnight.</p>

<h3>
  
  
  2025 direct translation
</h3>

<p>God Models (GPT-5, Claude Opus, Grok-4) = Lisp Machines<br><br>
Nvidia H200 racks = Symbolics boxes<br>
DeepSeek-R1, Qwen-2.5, Llama-405B + LoRAs = the Sun workstations that are already good enough</p>

<h3>
  
  
  The actual future (it’s Unix, not bigger brains)
</h3>

<ul>
<li>Tiny router (1–3 B) → picks the lane
</li>
<li>Retriever → grabs context
</li>
<li>Specialist (7–70 B) → does the work
</li>
<li>Synthesizer → makes it pretty
</li>
</ul>

<p>Whole chain = ~1/100th the cost of one 405 B call<br>
Runs locally on a 2027 phone. No data center required.</p>

<h3>
  
  
  The part nobody wants to talk about: Integration Tax
</h3>

<p>Monolith world<br><br>
→ expensive tokens, easy code</p>

<p>Chained world<br><br>
→ almost free tokens, engineering hell (routing, fallbacks, latency, observability, race conditions)</p>

<p>Result?<br><br>
It will be to complex to build open-ended user-facing LLM applications like chat-bots.</p>

<h3>
  
  
  The Great Sobering (coming 2026–2027)
</h3>

<p>The day a random 3–8 B open-weight model on an M5 / Snapdragon casually does 95 % of what we currently pay $500k+/month for is the day the entire frontier-model funding circus collapses.</p>

<p>To the AGI maximalists: over-investing in a beautiful idea is the hardest thing to quit.</p>

<p>To the builders reading this:<br><br>
The magic is dying.<br><br>
Real engineering is finally allowed to start.</p>

<p>That’s actually good news.</p>

<p>Which boring, high-ROI workflow do you think survives the Integration Tax?</p>

