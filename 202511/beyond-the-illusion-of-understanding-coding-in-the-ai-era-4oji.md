---
Title: What I Learned From Vibe Coding an Entire App
Description: 
Author: Jordan Green
Date: 2025-11-07T22:09:32.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  The Hidden Cost of Vibe Coding: Managing "Knowledge Debt"
</h1>

<p>"Vibe coding"—and utilizing AI in software development broadly—is still a frontier we are all actively mapping. It is undeniably a double-edged sword. On one side, it offers unprecedented speed; features that once took weeks can now be spun up in an hour. It feels miraculously efficient at first.</p>

<p>But this speed is not free. It comes at the cost of adding a thick, seductive layer of abstraction between you and your code. The central challenge for anyone learning to code today isn't just <em>"how can I build this,"</em> but <em>"how can I build this, internalize the fundamentals, and avoid accruing a mountain of 'Knowledge Debt' that I'll have to pay back later?"</em></p>

<h3>
  
  
  The Spectrum of Understanding vs. The Illusion of Competence
</h3>

<p>Understanding code is not a binary state—it is a spectrum.</p>

<ul>
<li>
<strong>Level 1:</strong> You understand it enough to prompt the AI to build it.</li>
<li>
<strong>Level 2:</strong> You understand it enough to fix it when the AI inevitably hits a wall.</li>
<li>
<strong>Level 3:</strong> You understand it enough to build it from scratch.</li>
</ul>

<p>The danger of AI is that it's easy to mistake Level 2 for Level 3. AI can give you the illusion of learning. You might read its explanation and nod along, but if you are constantly forgetting concepts a day later, you only achieved a shallow understanding.</p>

<p>We must be extremely mindful of the depth we are targeting. If your goal is to truly master TypeScript, asking an AI to "refactor my JS to TS" is actively harmful to your learning process. You are robbing yourself of the struggle that cements the knowledge.</p>

<h3>
  
  
  The New Burden for Junior Engineers
</h3>

<p>Ironically, while AI makes coding "easier," it makes <em>learning</em> to code more complex. Pre-AI engineers only had to worry about learning the fundamentals. Post-AI engineers must balance two competing skill sets simultaneously:</p>

<ol>
<li> Learning how to effectively wield AI tools.</li>
<li> Learning the discipline to <em>not</em> use them when fundamentals matter.</li>
</ol>

<p>Knowing when to dive deep and when to skim has always been a vital developer skill—if you deep dive on everything, you never ship. But AI has supercharged the ability to skim, making the temptation to stay in the shallow end nearly irresistible.</p>

<h3>
  
  
  Case Study: Paying Interest on LinkUp BU
</h3>

<p>I learned this the hard way while building my app, LinkUp BU. Relying heavily on tools like Claude Code felt like a superpower initially. I was shipping faster than I ever thought possible. What would have likely taken a solo dev like me with no Flutter experience over a year to make was done in a matter of months.</p>

<p>But I was accumulating massive Knowledge Debt.</p>

<p>Eventually, the "interest payments" came due. Things started breaking in subtle ways the AI couldn't fundamentally understand or fix. I found myself having to refactor huge chunks of the app because the AI's initial architectural decisions were suboptimal—"dumb," even.</p>

<p>While I successfully built the app and it works flawlessly, I realized this AI-first approach had a hidden cost. If you’ve spent any time job hunting, you’ve seen postings for engineers with "strong opinions." I realized that by outsourcing all the hard architectural decisions to an AI, I was robbing myself of the struggle required to <em>form</em> those opinions.</p>

<p>My next phase isn't just building new features; it's going back to pay off that debt, because I want to be an engineer who understands the 'why,' not just the 'how.'</p>

<p>To be clear, building LinkUp BU wasn't a passive experience—I learned a significant amount about full-stack architecture because I still had to touch and debug every part of the app to actually ship it. But this type of AI-assisted learning often feels like cramming for a college final. You learn the material just deeply enough to get the grade (or ship the feature), but the moment you walk out of the exam room, the detailed information begins to evaporate. I retain the high-level mental models and could certainly re-learn any specific part faster than a beginner, but if asked to apply it immediately in a deep technical interview, I would need to dig back into the "textbook" first.</p>

<h3>
  
  
  Conclusion: The Satisfaction of Struggle
</h3>

<p>Coding with AI can sometimes feel hollow. It becomes a game of trying to trick a generic model into understanding your specific intent, rather than the deeply engaging (and rewarding) puzzle of architectural problem-solving.</p>

<p>There is no standard playbook yet for how to integrate these tools into a junior developer's workflow. We have to figure it out for ourselves. But one thing is clear: we must use AI intentionally, not just because it's the path of least resistance.</p>

