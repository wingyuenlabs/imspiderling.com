---
Title: 🔫Why You Suck at Structuring Data And How to Finally Get Better 💪
Description: 
Author: Cédric Pierre
Date: 2025-12-04T21:42:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Let’s be honest.<br><br>
Most developers — frontend and backend — suck at structuring data.</p>

<p>Not because they’re stupid.<br><br>
Not because they’re lazy.<br><br>
But because the industry taught them to think in <strong>screens</strong>, not <strong>systems</strong>.</p>

<p>Once you understand this, everything starts to make sense.</p>

<p>Let’s break the taboo.</p>


<h2>
  
  
  1. You think in UI, not in domain
</h2>

<p>Most devs start from a Figma screen or a quick whiteboard sketch — but rarely from actual business logic.</p>

<p>They ask:</p>

<blockquote>
<p>“What JSON do I need for this response?”</p>
</blockquote>

<p>instead of:</p>

<blockquote>
<p>“What does the underlying domain really look like?”</p>
</blockquote>

<p>So the API becomes a <strong>reflection of the UI</strong>, not of the business.</p>

<p>Your tables, models, and routes start to smell like:</p>

<ul>
<li><code>subtitle</code></li>
<li><code>isCollapsed</code></li>
<li><code>displayOrder</code></li>
<li><code>object.with.20.levels.of.depth</code></li>
</ul>

<p>That’s not data modeling.<br><br>
That’s serializing your component tree.</p>


<h2>
  
  
  2. You merge unrelated concepts “because it’s easier”
</h2>

<p>You know this one:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"user"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="err">...</span><span class="p">},</span><span class="w">
  </span><span class="nl">"orders"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="err">...</span><span class="p">],</span><span class="w">
  </span><span class="nl">"permissions"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="err">...</span><span class="p">],</span><span class="w">
  </span><span class="nl">"stats"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="err">...</span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Congratulations.<br><br>
You've created a <strong>blob API</strong> — a JSON smoothie.</p>

<p>It works for one screen.<br><br>
It dies everywhere else.</p>




<h2>
  
  
  3. You don’t separate stable vs unstable structures
</h2>

<p>UIs change weekly.<br><br>
Business rules change yearly.</p>

<p>If your API breaks every time someone moves a card in Figma or a customer requests a “tiny change”…</p>

<p>You didn’t build an API.<br><br>
You built a <strong>UI transport layer</strong>.</p>

<p>A good data model must survive:</p>

<ul>
<li>redesigns
</li>
<li>mobile apps
</li>
<li>partner integrations
</li>
<li>analytics
</li>
<li>AI
</li>
<li>new features
</li>
<li>new markets
</li>
</ul>

<p>If it doesn’t survive these → it was never a real data model.</p>




<h2>
  
  
  4. You over-aggregate because you’re afraid of making 4 requests
</h2>

<p>Some developers believe:</p>

<blockquote>
<p>“We need one endpoint for the entire page.”</p>
</blockquote>

<p><strong>No, you don't.</strong></p>

<p>Making multiple async calls is:</p>

<ul>
<li>cleaner
</li>
<li>more predictable
</li>
<li>easier to cache
</li>
<li>easier to invalidate
</li>
<li>far more flexible
</li>
</ul>

<p>Aggregation is a tool.<br><br>
Not a default.</p>




<h2>
  
  
  5. You don’t understand entities vs projections
</h2>

<p>This is the root cause of 90% of bad APIs.</p>

<ul>
<li>
<strong>Entities</strong> = the truth of your business
</li>
<li>
<strong>Projections</strong> = reshaped, convenient data for a specific use-case
</li>
</ul>

<p>If you mix them, you get:</p>

<ul>
<li>UI fields leaking into persistence
</li>
<li>domain rules encoded inside components
</li>
<li>APIs that break with every design update
</li>
<li>models nobody understands anymore
</li>
</ul>

<p>Good developers keep these two worlds separate.<br><br>
Great developers <strong>document</strong> the projection boundaries.</p>




<h2>
  
  
  6. You never ask the hard questions
</h2>

<p>The best data architects ask:</p>

<ul>
<li>What is the source of truth?
</li>
<li>What entities actually exist?
</li>
<li>What invariants must never break?
</li>
<li>Would a mobile app use this?
</li>
<li>What happens if the UI changes completely?</li>
</ul>

<p>If you never ask these questions, you're not modeling — you're sketching.</p>




<h1>
  
  
  🔥 So… how do you get better?
</h1>

<p>Here’s the simple framework that instantly levels up your architecture.</p>




<h2>
  
  
  ✔️ 1. Model the domain before the API
</h2>

<p>If you can’t explain the business with boxes on a whiteboard → <strong>don’t write the endpoint.</strong></p>




<h2>
  
  
  ✔️ 2. Keep entities pure
</h2>

<p>No UI fields.<br><br>
No layout info.<br><br>
No Figma-driven naming.</p>

<p>If your database tables look like React components, stop everything and refactor.</p>

<p>If you chose a document database because it mirrors the UI, ask yourself:</p>

<blockquote>
<p>“What if there was no UI at all?”</p>
</blockquote>




<h2>
  
  
  ✔️ 3. Prefer multiple granular endpoints
</h2>

<p>Frontend frameworks are built for composition.<br><br>
Use them.</p>

<p>Let the UI combine the data.<br><br>
Don’t let the UI dictate the data.</p>




<h2>
  
  
  ✔️ 4. Use projections sparingly — and name them clearly
</h2>

<p>Examples:</p>

<ul>
<li>
<code>/orders</code> → entity
</li>
<li>
<code>/orders/overview</code> → projection
</li>
<li>
<code>/dashboard</code> → projection
</li>
<li>
<code>/users/:id/profile-view</code> → projection
</li>
</ul>

<p>Good naming avoids 80% of confusion.</p>




<h2>
  
  
  ✔️ 5. Design for multiple frontends
</h2>

<p>Ask yourself:</p>

<p>Would this structure still make sense if:</p>

<ul>
<li>I built a mobile app?
</li>
<li>A partner consumed the API?
</li>
<li>I fed this into an analytics pipeline?
</li>
<li>I replaced React tomorrow?</li>
</ul>

<p>If the answer is <strong>no</strong>, start over.</p>




<h2>
  
  
  ✔️ 6. Think in relationships and constraints
</h2>

<p>A real data model is not:</p>

<blockquote>
<p>“Objects with stuff inside.”</p>
</blockquote>

<p>It’s:</p>

<ul>
<li>entities
</li>
<li>references
</li>
<li>invariants
</li>
<li>constraints
</li>
<li>timelines
</li>
<li>state machines
</li>
</ul>

<p>This is where a developer becomes an architect.</p>




<h2>
  
  
  ✔️ 7. Simpler is better
</h2>

<p>Multiple small, meaningful entities are easier to maintain<br><br>
than one giant “god object.”</p>




<h2>
  
  
  ✔️ 8. Avoid deep nesting — prefer flat structures
</h2>

<p>Your API should deliver <strong>flat, normalized</strong> data.<br><br>
Let the UI handle grouping, filtering, nesting, aggregation.</p>

<p>Flat data = predictable data.</p>




<h2>
  
  
  ✔️ 9. Use correct naming — and be consistent
</h2>

<p>Clear naming saves lives.</p>

<p><code>UserGroupsResponse</code> is instantly understandable.</p>

<p>If your naming forces people to guess → it’s wrong.</p>




<h2>
  
  
  ✔️ 10. Anticipate change
</h2>

<p>Your model will grow.<br><br>
It will evolve.<br><br>
It will get more complex.</p>

<p>Design with flexibility in mind.<br><br>
Future-you will thank you.</p>




<h1>
  
  
  🎯 Final message
</h1>

<p>You don’t suck at structuring data because you’re bad.<br><br>
You suck because nobody ever taught you to think <strong>beyond the screen</strong>.</p>

<p>Once you learn to separate:</p>

<ul>
<li>domain vs projection
</li>
<li>entity vs layout
</li>
<li>truth vs convenience
</li>
</ul>

<p>…everything becomes easier.</p>

<p>Screens change every week.<br><br>
Good data structures last a decade.</p>

