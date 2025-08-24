---
Title: Build social features fast with mongoose-reactions — a tiny, production-ready Mongoose plugin for reactions
Description: 
Author: Ali nazari
Date: 2025-08-24T20:15:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>mongoose-reactions is a lightweight, TypeScript-first Mongoose plugin that adds Laravel-like polymorphic reactions (likes, loves, laughs, custom strings) to any model. </p>

<p>This post shows why you’ll want it in your stack, how it works, quickstart examples, scaling tips, and how you can help the project grow (⭐ <a href="https://github.com/Silent-Watcher/mongoose-reactions" rel="noopener noreferrer">GitHub</a>!).</p>

<h2>
  
  
  What this solves and why you should care
</h2>

<p>If you build social features (posts, comments, photos, products with likes), you need a fast, scalable, and developer-friendly way to attach reactions to any model.</p>

<p>mongoose-reactions:</p>

<ul>
<li>Stores reactions in a single, indexed collection for efficient counts and queries.</li>
<li>Works polymorphically (one reactions collection — many models).</li>
<li>Is TypeScript-ready and tested for common concurrency edge cases.</li>
<li>Provides easy statics &amp; instance helpers: react, unreact, toggleReaction, getReactionCounts, getUserReactions.</li>
<li>Plays nicely with transactions and bulk aggregation queries.</li>
</ul>

<p>If you want to ship social features fast, avoid custom ad-hoc arrays in your documents, and keep your code clean and maintainable — this plugin is for you.</p>

<h2>
  
  
  Why not just store reactions on the parent document?
</h2>

<p>Many early-stage apps put <code>likes</code> as an array on Post documents. That looks simple but quickly becomes painful:</p>

<ul>
<li>Large arrays grow MongoDB documents to awkward sizes and harm performance.</li>
<li>Counting and listing across many posts (or across models) becomes expensive.</li>
<li>Deleting a user’s reactions (GDPR) requires scanning many documents.</li>
<li>Cross-model analytics (top-reacted posts vs comments) is hard.</li>
</ul>

<p>By moving reactions into a small, indexed, dedicated collection you get:</p>

<ul>
<li>O(1) writes (append-only docs), efficient aggregation for counts, and simpler migrations.</li>
<li>Single source of truth for reactions and easier features (undo, toggle, per-user lists).</li>
<li>Ability to add adapters later (Redis for counters, Prisma adapter, etc.).</li>
</ul>

<h2>
  
  
  Features at a glance
</h2>

<ul>
<li>
<strong>Polymorphic</strong>: store reactions for Post, Comment, Photo, etc., in one collection.</li>
<li>
<strong>Configurable</strong>: single-reaction-per-user (default) or multi-reaction-per-user mode.</li>
<li>
<strong>Validation</strong>: whitelist reaction types (e.g., ['like','love','haha']) with case-insensitive normalization.</li>
<li>
<strong>Transaction-friendly</strong>: write operations can accept a session.</li>
<li>
<strong>Bulk helpers</strong>: get counts for many reactables in one aggregation.</li>
<li>
<strong>GDPR helpers</strong>: utilities to delete all reactions for a user or a reactable.</li>
<li>TypeScript types + Vitest testing scaffolding.</li>
</ul>

<h2>
  
  
  Quickstart — install &amp; use
</h2>

<p><strong>Install:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>mongoose mongoose-reactions
<span class="c"># or</span>
pnpm add mongoose mongoose-reactions
</code></pre>

</div>



<p><strong>Basic usage:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">mongoose</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">mongoose</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">reactionsPlugin</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">mongoose-reactions</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">PostSchema</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nc">Schema</span><span class="p">({</span> <span class="na">title</span><span class="p">:</span> <span class="nb">String</span> <span class="p">});</span>

<span class="nx">PostSchema</span><span class="p">.</span><span class="nf">plugin</span><span class="p">(</span><span class="nx">reactionsPlugin</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">reactionTypes</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">like</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">love</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">haha</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">angry</span><span class="dl">'</span><span class="p">],</span> <span class="c1">// optional whitelist</span>
  <span class="na">allowMultipleReactionsPerUser</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>          <span class="c1">// default</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">Post</span> <span class="o">=</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nf">model</span><span class="p">(</span><span class="dl">'</span><span class="s1">Post</span><span class="dl">'</span><span class="p">,</span> <span class="nx">PostSchema</span><span class="p">);</span>

<span class="c1">// react</span>
<span class="k">await</span> <span class="nx">Post</span><span class="p">.</span><span class="nf">react</span><span class="p">(</span><span class="nx">postId</span><span class="p">,</span> <span class="nx">userId</span><span class="p">,</span> <span class="dl">'</span><span class="s1">like</span><span class="dl">'</span><span class="p">);</span>               <span class="c1">// static call</span>
<span class="k">await</span> <span class="nx">postDoc</span><span class="p">.</span><span class="nf">react</span><span class="p">(</span><span class="nx">userId</span><span class="p">,</span> <span class="dl">'</span><span class="s1">love</span><span class="dl">'</span><span class="p">);</span>                    <span class="c1">// instance call</span>

<span class="c1">// counts (no need to pass model name; plugin infers it)</span>
<span class="kd">const</span> <span class="nx">counts</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Post</span><span class="p">.</span><span class="nf">getReactionCounts</span><span class="p">(</span><span class="nx">postId</span><span class="p">);</span>    <span class="c1">// -&gt; { like: 12, love: 3 }</span>
</code></pre>

</div>



<p><strong>Express route sample:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nx">express</span><span class="p">.</span><span class="nc">Router</span><span class="p">();</span>

<span class="nx">router</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/posts/:id/reactions</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">userId</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nx">_id</span><span class="p">;</span>            <span class="c1">// assume auth middleware</span>
    <span class="kd">const</span> <span class="nx">postId</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">.</span><span class="nx">id</span><span class="p">;</span>
    <span class="k">await</span> <span class="nx">Post</span><span class="p">.</span><span class="nf">react</span><span class="p">(</span><span class="nx">postId</span><span class="p">,</span> <span class="nx">userId</span><span class="p">,</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">reaction</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">counts</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Post</span><span class="p">.</span><span class="nf">getReactionCounts</span><span class="p">(</span><span class="nx">postId</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">ok</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="nx">counts</span> <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span> <span class="nf">next</span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  API (developer-friendly)
</h2>

<p>Core methods exposed on any model you apply the plugin to:</p>

<ul>
<li>
<strong><code>Model.react(reactableId, userId, reaction, meta?, { session? })</code></strong> - create/update reaction</li>
<li>
<strong><code>Model.unreact(reactableId, userId, reaction?, { session? })</code></strong>- remove reaction(s)</li>
<li>
<strong><code>Model.toggleReaction(reactableId, userId, reaction, meta?, { session? })</code></strong> - toggle behavior</li>
<li>
<strong><code>Model.getReactionCounts(reactableId)</code></strong>- <code>{ like: 10, love: 2 }</code>  (infers model)</li>
<li>
<strong><code>Model.getUserReactions(reactableId, userId)</code></strong> - <code>ReactionDoc[]</code> (useful when multiple allowed)</li>
<li>
<strong><code>Model.listReactors(reactableId , opts)</code></strong> -  list users who reacted with optional reaction filter, paginated </li>
</ul>

<p>Instance helpers map to the statics, so you can use <code>postDoc.react(userId, 'like')</code>.</p>

<h2>
  
  
  Polymorphism &amp; model names — the small but crucial detail
</h2>

<p>The plugin stores a <code>reactableModel</code> (we recommend the Mongoose <code>modelName</code> like 'Post') alongside <code>reactableId</code>.</p>

<p>This avoids collisions when two models share the same id value and makes <code>populate()</code> and permissions checks straightforward.</p>

<p>We infer <code>this.modelName</code> inside statics — so you don’t need to pass the model name every time. <code>Post.getReactionCounts(id)</code> is enough.</p>

<p><strong>If you like the plugin, please consider:</strong></p>

<p>⭐ Star the GitHub repo — stars help the project show up in searches and attract contributors.</p>

