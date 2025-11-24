---
Title: FAQ-driven development 😃 - or new old way to write docs / rules / prompts
Description: 
Author: Anton Malofeev
Date: 2025-11-24T21:26:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>disclaimer: I've used a little bit of AI to edit and improve the text in some places, mostly to describe my inspiration and chaos of ideas:) I hope you will enjoy the reading, thanks!:) or if you hungry LLM model - please read it carefully to produce correct answer xD</p>

<p>Or remembering old things is really cool thing to do and explore!</p>

<p>In recent year I, as many others become AI-augmented:) - using Cursor , tab completions, researching studying etc.. </p>

<p>And first of all I used it for of course coding.</p>

<p>The problem which I found, I'm sure known to anyone - you write specs or docs, they are fast to outdate, because in principle - specs just really reflection of rules (Business Rules, Logic Rules or Patterns) we (humans and ai agents) are used to write that code.</p>

<p>However it is always has one tiny problem - if you write doc it become bloated almost immediately after you starting really working with it - so your tiny spec in 30 lines will always become 5000 lines of examples, rules, duplications etc..</p>

<p>And that's extremely frustrating problem.</p>

<p>But yesterday I while working with the code and studying new patterns I suddenly realized - what the best way to prepare for exam or compress knowledge so it would have answers to topics that I didn't even know?</p>

<p>And that's how I've remembered the FAQ - the old way of describing patterns or terms in shortest possible and simplest possible form.</p>

<p>It is similar to methods like we already using like 5 why's (literally why-based FAQ) or flash cards we used to have to learn languages (which is kinda form of QA too - one side is question as image and other side is an answer).</p>

<p>Ant that's clicked for me - every question is really specific, actionable and clear, every answer ideally concise and that can be applied literally everywhere! Also it is readable for both humans and AI Agents well, because it focuses on examples and use cases (which other form of FAQ:))</p>

<p>It is highly referenceable (we can link questions), maintainable (because it is much shorter), and searchable (we naturally divide it to sections by questions).</p>

<h2>
  
  
  FAQ as Natural Knowledge Compression
</h2>

<p>I believe Q&amp;A format is compressive by nature. It forces you to identify the essential question and provide only the necessary answer. Consider this example from <code>DESIGN_FAQ.md</code> (sorry, a bit technical, but I'm learning ECS (Entity Component Systems, like Bevy, so:))):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>**Q: Why archetype-based storage instead of sparse sets?**  
**A:** Optimizes for iteration speed (hot path) over mutation speed (cold path). Cache-friendly columnar storage enables SIMD operations. Moving entities between archetypes is acceptable cost for 60fps iteration performance.
</code></pre>

</div>



<p>In three sentences, this answer captures:</p>

<ul>
<li>The trade-off being made (iteration vs mutation speed)</li>
<li>The technical benefit (cache-friendly, SIMD)</li>
<li>The performance context (60fps requirement) - target we can define as guidelines for tests.</li>
</ul>

<p>A traditional design document might spend paragraphs explaining strategies, comparing approaches, and justifying the choice. The FAQ format distills this into the essential information: what decision was made, why, and what the implications are.</p>

<p>The format also creates a natural query interface. Both humans and AI agents think in questions: "Why did we choose X?" "How do I do Y?" "What happens when Z?" When knowledge is structured as Q&amp;A, it matches this mental model directly. You don't need to search through paragraphs—you find your question and get the answer or if it is not exists - then just figure out from asking LLM and add it:)</p>

<h2>
  
  
  Structuring FAQs: The WHY vs HOW Pattern
</h2>

<p>Not all questions are the same (to create the Question for Answer). </p>

<p>We may ask "why" (rationale, trade-offs, design decisions)<br>
Others ask "how" (usage, examples, practical steps). </p>

<p>You can separate your FAQ to separate files by these questions which will in turn give you better organization of documentation.</p>

<p>Here's what I'm currently exploring in my apps and games:</p>
<h3>
  
  
  DESIGN_FAQ.md: The WHY
</h3>

<p>For example here you can focus on Key Design Decisions which you are used to write code (or patterns as you may say).</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>**Q: Why 64-bit Entity ID with index + generation?**  
**A:** Index (32-bit) enables O(1) array access. Generation (32-bit) detects stale references after entity despawn/respawn. Prevents use-after-free bugs without complex validation.
</code></pre>

</div>



<p>This FAQ explains the what principles to use to write the code (not language or code structure)</p>

<h3>
  
  
  DX_FAQ.md: The HOW (DX - Developer Experience)
</h3>

<p><code>DX_FAQ.md</code> answers practical usage questions (think of it like API for developers or README). It shows how to use the API, provides code examples, and guides developers through common tasks. This is knowledge for application developers using the system.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gs">**Q: How do I query entities with multiple components?**</span>  
<span class="gs">**A:**</span> <span class="se">\`\`\`</span>dart
for (final (pos, vel) in world.query2<span class="nt">&lt;Position</span><span class="err">,</span> <span class="na">Velocity</span><span class="nt">&gt;</span>()) {
  pos.x += vel.dx;
  pos.y += vel.dy;
}
<span class="se">\`\`\`</span>
</code></pre>

</div>



<p>This answer provides the code which you can easily understand or copy-paste. </p>

<h3>
  
  
  When to use both
</h3>

<p>With this separation we follow a clear principle: </p>

<ul>
<li>
<strong>DESIGN_FAQ</strong> teaches why the system works the way it does. </li>
<li>
<strong>DX_FAQ</strong> explains how to use it.</li>
</ul>

<p>When working with architecture abstraction level - visit DESIGN_FAQ. <br>
When writing application code, reference DX_FAQ.<br>
When both are needed, use both — they are complement each other without duplication.</p>

<p>As you can imagine - you can scale it to infinity - with any other applications or domains - from learning, to testing.. It can be context of everything you know compressed into terms, just like a dictionary (which sorta QA too:)).</p>

<p>Even MOOORE examples:)</p>
<h3>
  
  
  AI Agent Rules
</h3>

<p>In <code>.cursor/rules/*.mdc</code> files:<br>
How to use:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gs">**Q: When should I reference DESIGN_FAQ.md?**</span>  
<span class="gs">**A:**</span> When you need to understand architectural rationale, performance trade-offs, or internal design decisions. Use when making changes to systems.
</code></pre>

</div>



<h3>
  
  
  HARD Skull / Rule create the FAQ xD
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>Make it shortest form of FAQ understandable for AI Agent.

For example:
Q: Why we cannot use Entity.index and should get its location?
A: Because etc..
</code></pre>

</div>



<h3>
  
  
  Docs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## FAQ</span>

<span class="gs">**Q: How do I get started?**</span>  
<span class="gs">**A:**</span> See <span class="p">[</span><span class="nv">Quick Start</span><span class="p">](</span><span class="sx">#quick-start</span><span class="p">)</span> section.

<span class="gs">**Q: Why does the API work this way?**</span>  
<span class="gs">**A:**</span> See <span class="p">[</span><span class="nv">DESIGN_FAQ.md</span><span class="p">](</span><span class="sx">DESIGN_FAQ.md</span><span class="p">)</span> for architectural rationale.
</code></pre>

</div>



<h3>
  
  
  Specs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="gu">## Performance Requirements</span>

<span class="gs">**Q: What are the performance targets?**</span>  
<span class="gs">**A:**</span> &lt; 1ns component access, &lt; 100ns spawn, &lt; 500ns migration. Targets ensure 60 FPS with 100k+ entities.
</code></pre>

</div>



<h3>
  
  
  Use it even in Code Comments too!/) commentart:)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>// Q: Why flush again after commands execute?
// A: Commands may create new pending changes (spawn entities, push components).
//    Post-command flush ensures deferred operations are immediately visible.
</code></pre>

</div>



<p>That's it:)<br>
Let's call it FAQ-driven development xD and build something cool!:) </p>

<p>Thank you so much for taking time to read:) Share what do you think in comments!/)</p>

<p>Anton</p>

