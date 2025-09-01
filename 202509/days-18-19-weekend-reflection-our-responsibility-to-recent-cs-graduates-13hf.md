---
Title: Days 18-19: Weekend Reflection - Our Responsibility to Recent CS Graduates
Description: 
Author: Clay Roach
Date: 2025-09-01T22:07:25.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Weekend of August 30-31, 2025
</h2>

<p>This weekend, as I took a much-needed break from the intensive coding of our 30-day challenge (spending time at Alki Beach with friends, some excellent crab and salmon fishing, and a great BBQ), I found myself reflecting on something that's been weighing on my mind for weeks.</p>

<h2>
  
  
  The Conversations That Changed My Perspective
</h2>

<p>Over the last few months, I've had several conversations with recent Computer Science graduates—some friends of my son, others children of friends my age—who are struggling to even get unpaid internship positions. With the advances in coding capabilities of LLMs, getting entry-level jobs has become nearly impossible for them.</p>

<p>But here's what hit me: the problem is really us.</p>

<h2>
  
  
  Our Collective Responsibility
</h2>

<p>We as engineers have encouraged the younger generation (myself included) to pick up CS because they will "always be employable." In retrospect, this is still decent advice, but I feel the onus is on us as more experienced engineers to give these graduates actual opportunities. This could be a huge boost not only to their own prospects but to the economy as a whole—if we can figure out how to create the right jobs for them.</p>

<p>Right now, it's clear they won't be as good at coding out of the gate as anyone with five, ten, or 20+ years of experience. However, I think those of us in senior positions are the historical equivalent of assembly-level coders.</p>

<h2>
  
  
  The Assembly Language Analogy
</h2>

<p>It wasn't all that long ago that we had to take a larger leap of faith that compilers could generate as good (or better) code as hand-written assembly. We now sit on codebases and the entire web built out of higher-level programming languages.</p>

<p>We don't need engineers to learn how to develop and compete with the equivalent of assembly code against AI. Rather, they need to be extremely adept at building coding agents and enhancing tools such that they follow best practice engineering principles while operating at a higher level.</p>

<p>This still means getting deep into the code—just like we did when examining compiled binary or bytecode to see how it translated into machine instructions. We still need foundational principles to be well understood, but <strong>this is exactly what is still taught in Computer Science classes!</strong></p>

<h2>
  
  
  The Calculator Parallel
</h2>

<p>I recently had a conversation about all of this with my son, Nemo, and he called out the parallel to calculators. Schoolchildren are given these amazing tools but often not given the ability to learn how to use them effectively. Yes, we need to know the fundamentals so we can think abstractly and gain all the benefits of mathematical education, but at some point, we can accelerate our learning by taking those fundamentals and applying them to tools that can propel our education even further.</p>

<h2>
  
  
  Our Collective Amnesia
</h2>

<p>For me and this project, I want to primarily prove out the ability to build an enterprise-grade application with superhuman capabilities (credit: <a href="https://youtube.com/watch?v=-HzgcbRXUK8&amp;t=6207" rel="noopener noreferrer">Lex Friedman &amp; Demis Hassabis Podcast</a>) and experiment with ideas and approaches I've learned over 25 years of building application monitoring and management tools.</p>

<p>However, I feel like we have strange collective amnesia. We fought for years—desperately—for H1B visas and offshore hiring trends for the last 30+ years, and now somehow we feel like "well, we have enough developers now!"</p>

<p>I think this is a crock of BS. We still desperately need engineers who can take the current set of tools to the next level.</p>

<p>No, I don't expect them to rattle off three different ways to implement bubblesort in an interview, because now I expect the LLM to be very good at that kind of thing. But I do expect them to understand when and why different algorithms matter, and how to architect systems that leverage AI capabilities effectively.</p>

<h2>
  
  
  Taking Action: A Practical Experiment
</h2>

<p>Practically, this means I'll be attempting to enlist a few recent graduates into this project to see how well we can work through making them superhuman LLM-based developers.</p>

<p>Yes, I'll still refer them to <em>Patterns of Enterprise Application Architecture</em> (Martin Fowler) and speak fondly of my early days learning Java because I couldn't figure out if I was a "scruffy" or "neat" kind of AI student in the late 90s. But I also expect this will provide a good learning foundation for them in whatever career they decide to pursue.</p>

<p>It's on us—older engineers—to help lay foundation work for the next generation, just as it was laid down for us.</p>

<h2>
  
  
  Weekend Progress: Small Steps Forward
</h2>

<p>Speaking of foundation work, even during this relaxed weekend, we made some meaningful progress on the observability platform. The topology visualization now features a force-directed graph implementation (ADR-013 Phase 1) that provides real-time service health monitoring. It's a small piece, but it demonstrates how AI-assisted development can maintain momentum even during downtime.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Example of the AI-generated topology service integration</span>
<span class="kd">const</span> <span class="nx">topologyData</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchServiceTopology</span><span class="p">()</span>
<span class="kd">const</span> <span class="nx">healthMetrics</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">analyzeServiceHealth</span><span class="p">(</span><span class="nx">topologyData</span><span class="p">)</span>
<span class="kd">const</span> <span class="nx">visualizationComponent</span> <span class="o">=</span> <span class="nf">generateTopologyChart</span><span class="p">(</span><span class="nx">healthMetrics</span><span class="p">)</span>
</code></pre>

</div>



<p>The visualization automatically adapts to service changes and highlights potential issues—exactly the kind of high-level, AI-assisted development that recent graduates could excel at with proper guidance.</p>

<h2>
  
  
  The Path Forward
</h2>

<p>As we head into Week 3 of our 30-day challenge, I'm energized not just by the technical progress but by the possibility of creating a new model for how experienced developers can mentor and integrate recent graduates into meaningful, high-impact work.</p>

<p>The future isn't about replacing human developers with AI—it's about creating superhuman developer teams where AI amplifies human creativity, problem-solving, and architectural thinking.</p>

<p>Let's get cracking!</p>




<p><em>This is part of the 30-Day AI-Native Observability Platform series. Follow along as we build a complete observability platform using AI-assisted development, while exploring how to create opportunities for the next generation of developers.</em></p>

