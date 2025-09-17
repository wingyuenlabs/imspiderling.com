---
Title: Wolverine + Marten: My story and subjective take
Description: 
Author: Tran Manh Hung
Date: 2025-09-17T21:38:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>If you're a .NET developer who's heard about <a href="https://wolverinefx.net/" rel="noopener noreferrer">Wolverine</a> and <a href="http://martendb.io/" rel="noopener noreferrer">Marten</a>but haven't yet taken the plunge, this article might be for you.</p>

<p>I come from what you could call the "old dog" camp—spending most of my career with EF Core, Dapper, and the classic Controller-based approach. (Yes, I still write controllers instead of going all-in with minimal APIs—though I've dabbled with them in a few isolated scenarios.)</p>

<p>What follows is my personal story: moving from skepticism to cautious optimism as I started exploring Wolverine and Marten. Along the way I'll highlight the mistakes I made, the lessons I learned, and why these tools might (or might not) be worth your attention.</p>

<p>And just to be clear—this is a subjective take. And I am in still in the learning phase. So take it with a grain of salt. Don't be another AI that will keep glazing me that I am totally right. Feel free to fix any errors i made.</p>

<p>Alright lets start...</p>

<h2>
  
  
  The Traditional .NET Web API Way
</h2>

<p>Most of us .NET developers are comfortable with the controller-based approach:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="p">[</span><span class="n">ApiController</span><span class="p">]</span>
<span class="p">[</span><span class="nf">Route</span><span class="p">(</span><span class="s">"api/[controller]"</span><span class="p">)]</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">ProductsController</span> <span class="p">:</span> <span class="n">ControllerBase</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">IProductService</span> <span class="n">_productService</span><span class="p">;</span>

    <span class="p">[</span><span class="n">HttpPost</span><span class="p">]</span>  
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">ActionResult</span><span class="p">&gt;</span> <span class="nf">CreateProduct</span><span class="p">(</span><span class="n">CreateProductRequest</span> <span class="n">request</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">result</span> <span class="p">=</span> <span class="k">await</span> <span class="n">_productService</span><span class="p">.</span><span class="nf">CreateProductAsync</span><span class="p">(</span><span class="n">request</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">Ok</span><span class="p">(</span><span class="n">result</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It's familiar, well-documented, and works. So why change?</p>

<h2>
  
  
  What Are Wolverine and Marten?
</h2>

<p><strong>Marten</strong> is a .NET document database and event store built on PostgreSQL. Think of it as a sophisticated ORM that treats PostgreSQL as both a relational and document database.</p>

<p><strong>Wolverine</strong> is a messaging framework that handles command/query processing, with built-in support for Marten's event sourcing capabilities.</p>

<p>Together, they form a stack for building event-driven, DDD-focused applications.</p>

<h2>
  
  
  The Half-Baked Approach (Don't Do This)
</h2>

<p>When I inherited my current project, it was using Wolverine purely as a mediator pattern replacement:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="c1">// This is the WRONG way - just using Wolverine as MediatR</span>
<span class="k">public</span> <span class="k">class</span> <span class="nc">CreateProductHandler</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">ProductResponse</span><span class="p">&gt;</span> <span class="nf">Handle</span><span class="p">(</span><span class="n">CreateProductCommand</span> <span class="n">command</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="c1">// Traditional service calls, no events, no aggregates</span>
        <span class="c1">// Just replacing MediatR with Wolverine</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>This is the worst of both worlds.</strong> You get:</p>

<ul>
<li>All the learning curve of a new framework</li>
<li>None of the actual benefits</li>
<li>More ceremony than traditional controllers</li>
<li>Confused team members</li>
</ul>

<p>If you're going to use Wolverine, don't just use it as a fancy mediator. That's like buying Škoda car to use as a bicycle rack.</p>

<h2>
  
  
  Embrace the Full Stack
</h2>

<p>After struggle and intensive documentation reading, I realized the power comes from embracing the entire paradigm:</p>

<h3>
  
  
  1. Aggregate-Driven Design
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">Product</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="n">Name</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">decimal</span> <span class="n">Price</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="k">static</span> <span class="n">Product</span> <span class="nf">Create</span><span class="p">(</span><span class="kt">string</span> <span class="n">name</span><span class="p">,</span> <span class="kt">decimal</span> <span class="n">price</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">product</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">Product</span><span class="p">();</span>
        <span class="n">product</span><span class="p">.</span><span class="nf">Apply</span><span class="p">(</span><span class="k">new</span> <span class="nf">ProductCreated</span><span class="p">(</span><span class="n">Guid</span><span class="p">.</span><span class="nf">NewGuid</span><span class="p">(),</span> <span class="n">name</span><span class="p">,</span> <span class="n">price</span><span class="p">));</span>
        <span class="k">return</span> <span class="n">product</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">Apply</span><span class="p">(</span><span class="n">ProductCreated</span> <span class="n">@event</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">Id</span> <span class="p">=</span> <span class="n">@event</span><span class="p">.</span><span class="n">ProductId</span><span class="p">;</span>
        <span class="n">Name</span> <span class="p">=</span> <span class="n">@event</span><span class="p">.</span><span class="n">Name</span><span class="p">;</span>
        <span class="n">Price</span> <span class="p">=</span> <span class="n">@event</span><span class="p">.</span><span class="n">Price</span><span class="p">;</span>
    <span class="p">}</span>   
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. Command Handlers That Work With Aggregates
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">ProductHandlers</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">static</span> <span class="n">ProductCreated</span> <span class="nf">Handle</span><span class="p">(</span><span class="n">CreateProduct</span> <span class="n">command</span><span class="p">,</span> <span class="n">IDocumentSession</span> <span class="n">session</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">product</span> <span class="p">=</span> <span class="n">Product</span><span class="p">.</span><span class="nf">Create</span><span class="p">(</span><span class="n">command</span><span class="p">.</span><span class="n">Name</span><span class="p">,</span> <span class="n">command</span><span class="p">.</span><span class="n">Price</span><span class="p">);</span>
        <span class="n">session</span><span class="p">.</span><span class="nf">Store</span><span class="p">(</span><span class="n">product</span><span class="p">);</span>

        <span class="k">return</span> <span class="k">new</span> <span class="nf">ProductCreated</span><span class="p">(</span><span class="n">product</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">command</span><span class="p">.</span><span class="n">Name</span><span class="p">,</span> <span class="n">command</span><span class="p">.</span><span class="n">Price</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Or even better you can not use command at all<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">ProductHandlers</span>
<span class="p">{</span>
    <span class="p">[</span><span class="n">AggregateHandler</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">WolverinePost</span><span class="p">(</span><span class="s">"/product"</span><span class="p">),</span> <span class="n">EmptyResponse</span><span class="p">]</span>
    <span class="k">public</span> <span class="k">static</span> <span class="n">ProductCreated</span> <span class="nf">CreateProduct</span><span class="p">(</span><span class="n">CreateProduct</span> <span class="n">command</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">product</span> <span class="p">=</span> <span class="n">Product</span><span class="p">.</span><span class="nf">Create</span><span class="p">(</span><span class="n">command</span><span class="p">.</span><span class="n">Name</span><span class="p">,</span> <span class="n">command</span><span class="p">.</span><span class="n">Price</span><span class="p">);</span>        
        <span class="k">return</span> <span class="k">new</span> <span class="nf">ProductCreated</span><span class="p">(</span><span class="n">product</span><span class="p">.</span><span class="n">Id</span><span class="p">,</span> <span class="n">command</span><span class="p">.</span><span class="n">Name</span><span class="p">,</span> <span class="n">command</span><span class="p">.</span><span class="n">Price</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  3. Event-Driven Projections
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">class</span> <span class="nc">ProductProjection</span> <span class="p">:</span> <span class="n">SingleStreamProjection</span><span class="p">&lt;</span><span class="n">ProductSummary</span><span class="p">&gt;</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="n">ProductSummary</span> <span class="nf">Create</span><span class="p">(</span><span class="n">ProductCreated</span> <span class="n">created</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="k">new</span> <span class="n">ProductSummary</span>
        <span class="p">{</span>
            <span class="n">Id</span> <span class="p">=</span> <span class="n">created</span><span class="p">.</span><span class="n">ProductId</span><span class="p">,</span>
            <span class="n">Name</span> <span class="p">=</span> <span class="n">created</span><span class="p">.</span><span class="n">Name</span><span class="p">,</span>
            <span class="n">Price</span> <span class="p">=</span> <span class="n">created</span><span class="p">.</span><span class="n">Price</span>
        <span class="p">};</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  The Transformation Results
</h2>

<h3>
  
  
  Code Volume Reduction
</h3>

<p>By my totally subjective eye i think codebase shrunk by about 30%. No more:</p>

<ul>
<li>Separate DTOs for every operation</li>
<li>Mapping between entities and DTOs</li>
<li>Complex repository patterns</li>
<li>Manual change tracking</li>
</ul>

<h2>
  
  
  Ok so main points
</h2>

<h3>
  
  
  1. Start Small
</h3>

<p>Don't rewrite everything at once. Pick a bounded context and migrate gradually.</p>

<h3>
  
  
  2. Invest in Learning
</h3>

<p>Don't half-ass it. Read the documentation thoroughly. Understand DDD principles. Your team needs to really understand the concepts, not just copy-paste code.</p>

<h3>
  
  
  3. Embrace Events
</h3>

<p>Stop thinking in CRUD terms. Think in business events. "ProductCreated," "PriceChanged," "ProductDiscontinued."</p>

<h3>
  
  
  4. Use Projections Wisely
</h3>

<p>Create read models that match your UI needs. Don't try to make one projection serve all purposes.</p>

<h3>
  
  
  5. PostgreSQL over SQL Server
</h3>

<p>Remember, you're not just using a document database - you have all of PostgreSQL's power available for complex queries. Although this may not be up to date anymore (<a href="https://learn.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server?view=sql-server-ver17" rel="noopener noreferrer">https://learn.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server?view=sql-server-ver17</a>)</p>

<h2>
  
  
  Sample Project Structure
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/Domain
  /Products
    Product.cs (Aggregate)
    ProductEvents.cs 
/Handlers
  ProductHandlers.cs
/Projections
  ProductProjections.cs
Program.cs 
</code></pre>

</div>



<h2>
  
  
  My take
</h2>

<p>After about two to three weeks of learning (and I'm still learning), I've noticed that even my junior developers are coding faster and delivering features more smoothly. The initial skepticism I had slowly gave way to optimism as my concerns were put to the test.</p>

<p>At first, I thought using Marten was just reinventing the wheel. Why not simply stick with Entity Framework and MediatR? But as I dug deeper, I realized that while EF is excellent at what it does, Marten's hybrid approach of documents and relational data solves problems that EF tends to complicate. Event sourcing, projections, and complex querying become much more natural to work with, instead of feeling bolted on.</p>

<p>Another doubt I had was the risk of relying on a non-Microsoft library. What if the maintainers abandon it? To my surprise, I found that the project is actively maintained, backed by a strong community, and growing steadily. And even in the worst case, the data remains safe in PostgreSQL, and the design patterns I've learned are transferable to other systems.</p>

<p>Finally, I worried that the learning curve would be too steep for my team, especially for junior developers. I assumed they'd get lost in the concepts. But once they grasped the fundamentals of aggregates and events, the patterns began to feel intuitive. With the framework handling much of the heavy lifting, the adoption was far smoother than I expected.</p>

<h2>
  
  
  Summary
</h2>

<p>Is Wolverine + Marten for everyone? No. </p>

<p>Is it worth considering if you're building event-driven, domain-rich applications? Ye</p>

<p>The key is commitment. Don't use Wolverine as a drop-in MediatR replacement. Embrace the aggregate approach, think in events, and leverage PostgreSQL's strengths. The payoff is </p>

<p>My advice: Build a small proof of concept, read the documentation religiously, and be prepared for a mindset shift.</p>

<p><strong>Message to the Wolverine/Marten authors</strong>: If you notice unusual traffic spikes in your documentation in the last few weeks, im so sorry.</p>

