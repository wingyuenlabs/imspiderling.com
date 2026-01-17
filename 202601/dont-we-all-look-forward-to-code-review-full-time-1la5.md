---
Title: Don't we all look forward to code-review full time?
Description: 
Author: naugtur
Date: 2026-01-17T21:15:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>I remember the time when getting your tests to pass and finishing up a pull request was not even half the calendar time it took to ship code because it had to be reviewed. And getting a review was not trivial. Somehow everyone wanted to do other things and convincing them to review a pull request was part of doing the work.</p>

<p>Wait, that was yesterday!</p>

<p>Now with coding agents producing up to an order of magnitude more code it's not a problem though, because we've suddenly fell in love with the process of reviewing code. Right? Right...?</p>

<p>I mean, isn't it dreadful that you not only have to do code review for your peers but also for what you're making now too?</p>

<p>But why is reviewing code not sparking the same joy as writing it? </p>

<p>There's many reasons, but I want to focus on one. The friction in building the mental model.</p>

<p>The process of understanding how code works is really one of building a mental model that represents the logic encoded in the program. Mental models are structures. Building a structure "from the ground up" is much easier than floating bits in space until it's rigid, if we take a physical metaphor. And that's my point - reading the code of a finished changeset is much less convenient for creating a mental model than experiencing various fragments of code going together in a reasonable order.</p>

<p>Which brings me to my point - AI coding agents would be much better if they gave us the experience of a pair programmer instead of an experience of an intern turning in every result as if it was a take-home assignment.</p>

<p>So imagine giving the agent a prompt or a spec and instead of getting a non-stop flow of tokens appear in the project files, being told a relatable story of how the logic flows through the change.</p>

<p>Code appearing in a logical order, with live spoken commentary. You don't have to read 2 places at once. Human modalities (reading and listening) are utilized optimally.</p>

<blockquote>
<ul>
<li>Let's start by creating the function with its arguments and defining input types.</li>
<li>Now we're adding validation same as the function above</li>
<li>Since that's reusable, let's extract it to the utils file</li>
<li>We're now implementing the functionality split into 3 steps. </li>
<li>Step 1 will call out to the implementation over there... </li>
</ul>
</blockquote>

<p>That sort of presentation of the code would not only match the natural process in which the human builds the mental model, it would also eliminate the context switching and distractions creeping in while the agent is working.</p>

<p>It should also scale to orchestrating multiple agents, where the work progresses while the leading agent schedules the work, controls the results and enqueues the walkthrough/explanation for the human in the loop.</p>

<p>And then the obvious next step is to handle instant feedback and discussion, like a peer programmer would.</p>

<p>I hope someone build this  :D</p>

