---
Title: We Didn’t “Align” — We Argued (and Shipped a Better System)
Description: 
Author: Art light
Date: 2026-01-11T21:50:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>Software projects rarely fail because someone can’t write code.<br>
They fail because people quietly disagree, assumptions go unchallenged, and everyone is “aligned” right up until production is on fire.</p>

<p>This is a story about a project where collaboration didn’t look pretty — but it worked.</p>
<h2>
  
  
  The Setup: Parallel Work, Parallel Universes
</h2>

<p>We were building a product with a fairly standard setup:</p>

<ul>
<li>Frontend: React + TypeScript</li>
<li>Backend: REST API</li>
<li>Multiple engineers working in parallel</li>
<li>Tight deadlines (the fun kind)</li>
</ul>

<p>Everything looked fine… until it didn’t.</p>

<p>The frontend assumed fields that the backend hadn’t implemented yet.<br>
The backend refactored response shapes without realizing the frontend relied on them.<br>
Types existed — just not in the same universe.</p>

<p>Nothing was broken enough to stop development, but bugs kept appearing late.<br>
The worst kind. The “why didn’t we catch this earlier?” kind.</p>
<h2>
  
  
  The First Real Collaboration Moment (aka The Argument)
</h2>

<p>At some point, someone asked the dangerous question:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>“Why don’t we just define the API first?”
</code></pre>

</div>



<p>This triggered:</p>

<ul>
<li>A backend engineer worrying about losing flexibility</li>
<li>A frontend engineer worried about slowing down</li>
<li>Someone else saying “we can just fix it later” (classic)
Instead of pretending we agreed, we argued it out.</li>
</ul>

<p>And that was the turning point.</p>

<h2>
  
  
  The Decision: API First (But Practically)
</h2>

<p>We didn’t adopt API-first as a buzzword.<br>
We adopted it as a collaboration contract.</p>

<p>What we actually did:</p>

<ul>
<li>Defined OpenAPI specs early</li>
<li>Generated types for the frontend</li>
<li>Treated breaking API changes as actual breaking changes, not “oops”</li>
</ul>

<p>Trade-offs we accepted:</p>

<ul>
<li>Backend lost some freedom to “just rename stuff”</li>
<li>Specs required discipline and review</li>
<li>Changes took slightly longer upfront</li>
</ul>

<p>What we gained:</p>

<ul>
<li>Predictable frontend types</li>
<li>Fewer late surprises</li>
<li>Much calmer PR reviews</li>
</ul>
<h2>
  
  
  The Unexpected Benefit: Better Conversations
</h2>

<p>The biggest win wasn’t technical — it was human.</p>

<p>Instead of:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>“Why is this field missing?”
</code></pre>

</div>



<p>We started asking:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>“Should this field even exist?”
</code></pre>

</div>



<p>Specs forced us to:</p>

<ul>
<li>Name things clearly</li>
<li>Argue about responsibility</li>
<li>Make decisions visible</li>
</ul>

<p>Design reviews became real design reviews — not post-mortems.</p>

<h2>
  
  
  Mistakes We Made (So You Don’t Romanticize This)
</h2>

<p>Let’s be honest — it wasn’t smooth:</p>

<ul>
<li>We over-specified early on and had to loosen things later</li>
<li>One spec PR was longer than some novels</li>
<li>We once broke the API and forgot to regenerate types (yes, that hurt)</li>
</ul>

<p>Collaboration doesn’t remove mistakes.<br>
It just makes them cheaper and earlier.</p>
<h2>
  
  
  Humor Break: The “One Small Change” Lie
</h2>

<p>Every team has this moment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>“It’s just a small backend change.”
</code></pre>

</div>



<p>Famous last words.</p>

<p>With shared specs, that sentence became:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>“It’s a small change… but it touches 12 endpoints.”
</code></pre>

</div>



<p>Which is exactly the kind of honesty you want before merging.</p>

<h2>
  
  
  What Senior Collaboration Actually Looks Like
</h2>

<p>From this experience, here’s what I now believe:</p>

<ul>
<li>Collaboration is surfacing disagreement early</li>
<li>Silence is more dangerous than conflict</li>
<li>Tools don’t fix teams — they expose them</li>
<li>Clear contracts reduce ego-driven decisions</li>
</ul>

<p>Senior engineers don’t avoid friction.<br>
They manage it productively.</p>

<h2>
  
  
  Final Takeaway
</h2>

<p>Good collaboration isn’t about being nice.<br>
It’s about being clear, honest, and occasionally uncomfortable — together.</p>

<p>When teams stop optimizing for speed today and start optimizing for trust tomorrow, the codebase follows.</p>

<p>And yes — sometimes that starts with an argument.</p>

