---
Title: Adversarial Planning for Spec Driven Development
Description: 
Author: marcosomma
Date: 2026-02-12T21:39:13.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p>I have always loved one idea in machine learning. The idea that you can sharpen a model by forcing it to face a challenger. You can call it adversarial training, red teaming, or constructive hostility. The name matters less than the mechanism. You introduce pressure. You int<br>
roduce disagreement. You force the system to earn its confidence.</p>
</blockquote>

<p>For years I kept that concept in a mental drawer labeled “cool, but academic.” Then it become a core concept within the Orka-reasoning development but lately my attention is shifting toward code agent workflows and how all happen. Not the marketing version. The real version, where you sit down to ship software, and you realize that a helpful model is not the same thing as a rigorous model. Helpful is easy. Rigorous is costly.</p>

<p>This article is about how I tried to transplant an adversarial dynamic into Spec Driven Development sessions. Not as theater. Not as an AI debate club. As an engineering tool. It worked. It also nearly became a token-burning trap. That tradeoff is the point.</p>

<h2>
  
  
  What Spec Driven Development means here
</h2>

<p>Spec Driven Development, or SDD, is a workflow where the spec is not documentation. The spec is the product of the thinking phase. It becomes the contract you implement against. You write it before code changes. You review it like you would review code. You use it to force scope, constraints, interfaces, and acceptance criteria into something explicit.</p>

<p>The point is not to be verbose. The point is to move ambiguity upstream, when it is still cheap. The spec becomes the unit of alignment, review, and iteration. Code is the execution of that spec, not the place where you discover what the spec should have been.</p>

<h2>
  
  
  The problem with a trustable planner
</h2>

<p>If you use an LLM as a planning companion, you know the feeling. You came with your idea. You debate a bit. And then it gives you a plan. The plan is detailed. It is readable. It sounds plausible. It often includes little snippets that look like they belong in your codebase, even when they do not. It is confident. It is fast.</p>

<p>And that is exactly the problem.</p>

<p>A planner model has incentives you did not explicitly set. Its default incentive is to be useful to you in the moment. It wants to reduce friction. It wants to keep you engaged. It wants to produce something that reads like progress.</p>

<p>So it will fill gaps with assumptions. If your own initial plan is fluffy. It will smooth rough edges. It will complete the pattern of what a good plan should look like. It will also happily unlock future possibilities, because possibilities are cheap to generate and expensive to invalidate.</p>

<p>When you are deep in a product, that behavior is dangerous. Not because the model is malicious. Because it is compliant. It will often accept your framing even if your framing is wrong. It will not push hard unless you force it to.</p>

<p>This is the failure mode I kept hitting. I would craft a plan with the planner. I would feel momentum. Then I would start implementation and discover that the plan was under-specified in the only places that matter.</p>

<p>Interfaces were vague. Invariants were missing. Acceptance criteria were soft. The plan assumed the architecture could absorb a change without showing how. It assumed the code was more modular than it actually was. It assumed integration would be straightforward.</p>

<p>In other words, it was a nice plan. It was not a plan that survived contact with a real codebase.</p>

<h2>
  
  
  Why adversarial dynamics work in ML
</h2>

<p>Adversarial training is interesting because it makes weakness visible. You do not improve a system by praising it. You improve it by exposing it to inputs that exploit its blind spots. You force it to fail in ways that are informative.</p>

<p>In a GAN, the generator learns because the discriminator is not polite. The discriminator does not care about your feelings. It cares about whether the output holds up under scrutiny. That pressure creates signal.</p>

<p>In engineering, we already do this. Code review is adversarial when it is healthy. Testing is adversarial by definition. Security review is adversarial. Load testing is adversarial. Even a good product manager is adversarial at the right moments.</p>

<p>But planning often is not. Planning often becomes a social process. People nod. People optimize for alignment. People avoid being the blocker. Under time pressure, that tendency gets amplified.</p>

<p>If you bring an LLM into planning and you let it be the agreeable teammate, you amplify the most comfortable version of planning. You pay tokens to make yourself feel certain.</p>

<p>That is not what I wanted. I wanted the planning stage to contain more of the pain, so implementation contains less.</p>

<h2>
  
  
  The translation to SDD: Planner plus Architect
</h2>

<p>I kept the planner. I did not replace it. The planner is good at structure. It is good at decomposing a vague goal into sequential work. It is good at producing a spec you can follow. It is good at holding context across iterations.</p>

<p>But I introduced a second role. I call it the Architect. The job is simple. Challenge the plan as if you are the most annoying senior engineer in the room, with one constraint. The criticism must be grounded. It must point to specific failure modes. It must force explicit decisions.</p>

<p>The Architect pushes on the places where the planner tends to glide over reality. It asks what the boundary of the change really is. It asks what breaks if you do it, and what breaks if you do not. It pressures you to name the coupling you are creating and the coupling you are relying on. It attacks the parts of the spec that sound confident but are not falsifiable.</p>

<p>This role is unpleasant. It is supposed to be unpleasant. It is also productive, if you keep it under control.</p>

<p>The immediate effect was obvious. Specs became harder to write. My initial drafts got rejected more often. I had to define outcomes in tighter language. I had to stop relying on vibes and start writing constraints.</p>

<p>The less obvious effect was more important. I started noticing the difference between a plan that sounds implementable and a plan that is falsifiable.</p>

<p>A falsifiable plan is one where you can point at a step and say: if this condition does not hold, the step is wrong. If the step is wrong, we know why. We can adjust.</p>

<p>A non-falsifiable plan is one where every step is elastic. You can always reinterpret it. You can always claim partial success. It is planning as comfort.</p>

<p>The Architect hates comfort. That is the point.</p>

<h2>
  
  
  What the Architect actually improved
</h2>

<p>It did not make my system magically correct. It made my system explicit.</p>

<p>It reduced scope creep because it forced me to define what done means in terms of observable outcomes. It reduced hidden coupling because it forced me to identify which pieces of the system now move together. It reduced abstraction drift because it forced me to state which module owns which responsibility. It improved testability because it pushed me to name the failure cases the system must catch and the layer that must catch them. It also lowered integration fantasies by making me draw the dependency edges in plain language.</p>

<p>This matters because most planning failures are not about missing steps. They are about missing friction. You only discover friction when someone tries to break your plan.</p>

<p>A planner rarely tries to break your plan. An Architect lives for it.</p>

<h2>
  
  
  The mental model: controlled adversarial pressure
</h2>

<p>At some point I realized the dynamic I was building was not adversarial planning. It was controlled adversarial pressure.</p>

<p>Pressure is good when it produces signal. Pressure is bad when it produces noise.</p>

<p>The Architect can easily produce noise. It can challenge everything. It can question the existence of the feature. It can spiral into meta debates. It can do the classic senior engineer move of turning every change into a referendum on architecture.</p>

<p>That is why this approach can become dangerous. It is not just about tokens. It is about cognitive load. Too much adversarial pressure makes you doubt everything. You stop shipping. You start ruminating. You start optimizing a plan instead of building the thing.</p>

<p>So the key is control. You want the Architect to challenge the plan in a bounded way, then you move on.</p>

<p>The only sustainable use is somewhere in the middle. You let it break your plan until the breakage becomes repetitive. When the criticism starts looping, it is done. That loop is your stop signal.</p>

<h2>
  
  
  How the infinite loop happens
</h2>

<p>I learned this the hard way. The Architect is very good at finding the next critique, even when the plan is already good enough, even when the remaining critiques are marginal.</p>

<p>There are two reasons.</p>

<p>First, LLMs are generative machines. They can always produce another objection. The space of objections is large. Many objections are plausible. Plausible is not the same as important.</p>

<p>Second, adversarial roles reward themselves. When the Architect produces a clever critique, it feels like progress. It feels like rigor. It feels like you are doing serious engineering. You can get addicted to that feeling, especially if you already equate doubt with intelligence.</p>

<p>So you need stop conditions that are not emotional. You need boundaries that are mechanical.</p>

<p>Time is a boundary. Token budget is a boundary. The best boundary is value.</p>

<p>The question is: does this criticism point to a concrete failure mode that is likely in this codebase, in this release, under these constraints. If yes, incorporate it. If no, write it down as a future consideration and move on.</p>

<p>That discipline sounds simple. It is not. It requires you to accept that you will ship with risk. It requires you to prefer explicit risk over imagined safety.</p>

<h2>
  
  
  Why this helps SDD specifically
</h2>

<p>SDD is already an attempt to move thinking earlier. You spend more effort defining the work before coding. That sounds obvious. It is not common.</p>

<p>Many teams code first, then retrofit clarity. Specs become documentation after the fact. Tests become a safety net after the mistakes.</p>

<p>SDD flips that. You try to make the spec the forcing function. The spec becomes the contract. The spec becomes the review surface. The spec becomes the artifact you can reason about without running the entire system in your head.</p>

<p>If your spec is weak, SDD collapses into bureaucracy. You get long documents that do not prevent failures. You get ceremonial approval. You get a spec that exists, but does not constrain the outcome.</p>

<p>The adversarial role helps because it forces the spec to earn its existence. It forces explicit interfaces. It forces explicit invariants. It forces explicit failure handling. It forces explicit success conditions. It makes the spec testable in a reasoning sense.</p>

<h2>
  
  
  Doubt as a tool, doubt as a poison
</h2>

<p>There is a psychological aspect here that I did not expect.</p>

<p>When you introduce an adversarial voice into planning, you introduce doubt. That can be healthy. It can also be corrosive.</p>

<p>Healthy doubt looks like this. You have a plan. You expose it to pressure. You find the weak points. You fix them. You ship with more confidence because your confidence is earned.</p>

<p>Corrosive doubt looks like this. You have a plan. You expose it to pressure. The pressure never ends. You start believing that every plan is fragile. You stop trusting your ability to decide. You keep rewriting the plan to reduce anxiety. You ship nothing.</p>

<p>The difference is not intelligence. The difference is boundaries.</p>

<p>In a team, boundaries are social. Someone ends the meeting. Someone says enough, we decide. Someone accepts risk explicitly.</p>

<p>In a solo workflow with agents, you need to manufacture that boundary. Otherwise the system will drift toward endless review because endless review feels safer than a decision.</p>

<p>If you are prone to overthinking, an adversarial agent can amplify that trait. It can turn careful into paralyzed. That is not a reason to avoid it. It is a reason to instrument it.</p>

<h2>
  
  
  What this is not
</h2>

<p>This is not asking an AI to argue with itself and then picking a side. That is entertainment. It can be useful for brainstorming. It is not a development methodology.</p>

<p>This is not letting the Architect design the system. That is just outsourcing. The Architect is a critic, not a creator.</p>

<p>This is not making the Architect mean. Mean is cheap. Precision is expensive. You want precision tied to concrete failure modes.</p>

<p>This is also not a replacement for real review. A human senior engineer with context will catch things an LLM will miss. The point here is to raise your baseline. The point is to catch the obvious architecture risks before you waste days implementing.</p>

<h2>
  
  
  The practical outcome
</h2>

<p>The measurable outcome for me was simple.</p>

<p>I rewrote fewer specs mid-implementation. I discovered fewer “we forgot that” moments. I spent less time refactoring because of missing boundaries. I argued less with my future self.</p>

<p>The spec still does not become perfect. The spec fails earlier, on paper, when failure is cheap. That is what adversarial pressure buys you.</p>

<h2>
  
  
  The simplest way to frame it
</h2>

<p>Your planner optimizes for completeness. Your Architect optimizes for survivability.</p>

<p>Completeness is about covering steps. Survivability is about covering reality.</p>

<p>A complete plan can still die on a hidden assumption. A survivable plan is one where assumptions are visible, bounded, and either validated or consciously accepted.</p>

<p>The adversarial role does not need to make you pessimistic. It needs to make you explicit. If it makes you pessimistic, you let it run too long.</p>

<h2>
  
  
  Sane engineer the disagreement
</h2>

<p>Good engineering requires disagreement. Not constant fighting. Not performative contrarianism. Real disagreement that targets risk.</p>

<p>In teams, disagreement is expensive socially. With agents, disagreement is expensive computationally. The price changes. The dynamics stay.</p>

<p>If you can engineer disagreement so that it is bounded, precise, and tied to concrete failure modes, you get a sharper process. You get better specs. You get fewer surprises.</p>

<p>If you cannot bound it, you get the worst of both worlds. You get more doubt and less shipping.</p>

<p>So adopt the adversarial phase, but treat it like a test suite. You run it to catch failures. You do not run it forever because you enjoy watching it fail.</p>

<p>Controlled adversarial pressure. Enough to sharpen. Not enough to cut.</p>

