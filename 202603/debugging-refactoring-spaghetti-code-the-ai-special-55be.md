---
Title: Debugging & Refactoring Spaghetti Code - The A.I Special 🤖
Description: 
Author: Brandon Lozano
Date: 2026-03-03T21:33:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>Given the fact that people have varying and mostly static opinions on Artificial Intelligence, I feel like I must start by stating my own. The following is merely opinion based on my experience and overall understanding of the world and technology.</p>

<p><strong>Short &amp; Sweet - My Opinion on A.I</strong></p>

<p>I think every engineer should utilize A.I to some extent: writing docs, debugging, comparing technologies, and writing code are all good use cases. However, it is not, nor will it be, the end all and be all for great software. We do not have the compute nor the data for A.I to reach Artificial General Intelligence (AGI) - where A.I becomes human-like. The people who are selling the dream of A.I doing everything while we hang back like the people in WALL-E are directly benefiting from the stock surges caused by this speculation. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg481uzp17hjgdry26del.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg481uzp17hjgdry26del.png" alt="WALL-E People In Chairs" width="800" height="337"></a></p>

<h2>
  
  
  Problem Statement
</h2>

<p>Once upon a time, not too long ago I was handed over a project for Data APIs that were responsible for collecting and organizing all sales data for the company. The APIs had a ~80% success rate, not great for sales data that would mold and shift marketing and sales tactics. </p>

<p>On top of the bug infestation, the application was hosted in PipeDream which basically provides an env for your code to run, and handles things like gateways. Awesome for viability testing, but not great for real world applications. There was essentially no version control, config access, or proper way to manage dependencies.</p>

<h2>
  
  
  Why All Of The Issues?
</h2>

<p>All of the architecture, code, and schema was built while heavily relying on A.I, and when I say heavily rely I mean, A.I drove - the old employee hit yes. </p>

<p>Overall there were around 50 data points that needed to be tracked, to get them there were countless <code>if else</code> statements, nested for loops, and API calls for each individual order instead of batch calls.</p>

<p>The project was the utter representation of spaghetti code - but I honestly learned more about APIs, debugging, databases, and clean code during this project than any other single project.</p>

<h2>
  
  
  To Gain An Understanding
</h2>

<p>Plainly looking at the code and digesting the limited documentation did not provide enough context to give me any true indication as to what data points were required and where they came from. </p>

<p>I at least had a general understanding that sales data originated in Shopify, we were utilizing HubSpot to associate sales to team members, PostgreSQL was our combined source of truth, and the whole thing relied on PipeDream APIs. I started from the DB viewpoint to understand how data should be represented (if the pipeline ran successfully).</p>

<p>Below is the example schema for the deals table, this is the main entry point for tracking all sales<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">IF</span> <span class="k">NOT</span> <span class="k">EXISTS</span> <span class="n">DEALS</span> <span class="p">(</span>
  <span class="n">deal_number</span> <span class="nb">INT</span><span class="p">,</span>
  <span class="n">price</span> <span class="nb">INT</span><span class="p">,</span>
  <span class="n">disc_price</span> <span class="nb">INT</span><span class="p">,</span>
  <span class="n">shipped</span> <span class="nb">BOOLEAN</span><span class="p">,</span>
  <span class="p">...</span>
<span class="p">);</span>
</code></pre>

</div>



<p>Once I thoroughly understood data requirements, I began to use pen and paper to map out data points, their origin source, and data type that was preferred. This allowed me to make very rough sketches of the data pipeline. To start, the entire system was of a very ambiguous state - no beginning and no end.</p>

<h2>
  
  
  Console Line Debugging
</h2>

<p>A real advantage here was that Pipedream saved each event with its full request payload for 30 days, meaning I could replay the exact same request through the broken pipeline repeatedly while making fixes. This made isolating failures significantly faster than traditional debugging — rather than waiting for real events to reproduce a bug, I could trigger it on demand.</p>

<p>Below is an example of what the code base looked like.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Rough Example of what we were dealing with</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">Client</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">pg</span><span class="dl">'</span><span class="p">);</span>
<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">EventHandler</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">fetchShopifyOrders</span> <span class="o">=</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SHOPIFY_URL</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">API_KEY</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SHOPIFY_API_KEY</span><span class="p">,</span>
      <span class="p">}</span>
    <span class="p">});</span>
    <span class="k">return</span> <span class="nx">response</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">fetchHubspotData</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">orderId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">HUBSPOT_URL</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">API_KEY</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">HUBSPOT_API_KEY</span><span class="p">,</span>
      <span class="p">},</span>
      <span class="na">body</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">orderNumber</span><span class="p">:</span> <span class="nx">orderId</span>
      <span class="p">}</span>
    <span class="p">});</span>
    <span class="k">return</span> <span class="nx">response</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Client</span><span class="p">({</span>
    <span class="na">user</span><span class="p">:</span> <span class="dl">'</span><span class="s1">your_user</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">host</span><span class="p">:</span> <span class="dl">'</span><span class="s1">localhost</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">database</span><span class="p">:</span> <span class="dl">'</span><span class="s1">your_db</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">password</span><span class="p">:</span> <span class="dl">'</span><span class="s1">your_password</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">port</span><span class="p">:</span> <span class="mi">5432</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">orders</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchShopifyOrders</span><span class="p">();</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">order</span> <span class="k">of</span> <span class="nx">orders</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">orderNumber</span> <span class="o">=</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">orderNumber</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">currency</span> <span class="o">=</span> <span class="dl">''</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">price</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">delivered</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">hubspotId</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">hubspotOwner</span><span class="p">;</span>
    <span class="c1">// image 30 more variable likes this </span>

    <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">country</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">us</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">currency</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">USD</span><span class="dl">'</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">country</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">ca</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">currency</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">CAD</span><span class="dl">'</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">country</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">au</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">currency</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">AUD</span><span class="dl">'</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">discountPrice</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">price</span> <span class="o">=</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">discountPrice</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="nx">price</span> <span class="o">=</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">price</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">deliveredStatus</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">delivered</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">delivered</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
      <span class="nx">delivered</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// and 30 more if else statements to assign variable values</span>

    <span class="kd">const</span> <span class="nx">hubspotData</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchHubspotData</span><span class="p">(</span><span class="nx">orderNumber</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">hubspotData</span><span class="p">.</span><span class="nx">dealOwner</span> <span class="o">!==</span> <span class="dl">''</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">hubspotOwner</span> <span class="o">=</span> <span class="nx">hubspotData</span><span class="p">.</span><span class="nx">dealOwner</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">hubspotData</span><span class="p">.</span><span class="nx">id</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">hubspotId</span> <span class="o">=</span> <span class="nx">hubspotData</span><span class="p">.</span><span class="nx">id</span>
    <span class="p">}</span>

    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">query</span> <span class="o">=</span> <span class="s2">`INSERT INTO DEALS (orderNumber, currency, price, delivered, hubspotId, hubspotOwner) VALUES (</span><span class="p">${</span><span class="nx">orderNumber</span><span class="p">}</span><span class="s2">, </span><span class="p">${</span><span class="nx">currency</span><span class="p">}</span><span class="s2">, </span><span class="p">${</span><span class="nx">price</span><span class="p">}</span><span class="s2">, </span><span class="p">${</span><span class="nx">delivered</span><span class="p">}</span><span class="s2">, </span><span class="p">${</span><span class="nx">hubspotId</span><span class="p">}</span><span class="s2">, </span><span class="p">${</span><span class="nx">hubspotOwner</span><span class="p">}</span><span class="s2">) RETURNING *`</span>
    <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="nx">query</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Issues with the above
</h3>

<ul>
<li>
<code>aysnc</code> typo would crash immediately at runtime</li>
<li>
<code>fetchShopifyOrders</code> called without <code>await</code> — iterating a Promise</li>
<li>
<code>client.connect()</code> called on every loop iteration, never released</li>
<li>Raw template literal SQL query is a textbook SQL injection vulnerability</li>
<li>Hardcoded DB credentials in the source code</li>
<li>No error handling anywhere — a single failure crashes the entire batch</li>
<li>Dozens of verbose <code>if/else</code> chains that could each be a single expression</li>
<li>Variables declared and mutated individually per order with no shared shape or structure</li>
</ul>

<h3>
  
  
  To The Refactor
</h3>

<p>After quite a bit of work and understanding, I was able to get to a place like this: </p>

<ul>
<li>Batch cleaned all order data first with a single <code>map</code> before any async work</li>
<li>Bulk fetched all HubSpot data in parallel with <code>Promise.all</code> instead of sequential awaits in a loop</li>
<li>Replaced verbose if/else chains with a <code>CURRENCY_MAP</code> lookup and ternary operators</li>
<li>Moved DB credentials to environment variables</li>
<li>Introduced a single pool connection reused across all inserts, released in a <code>finally</code> block</li>
<li>Parameterized SQL queries eliminating injection risk</li>
<li>
<code>RETURNING id</code> gives back insert IDs for debugging and reduces load for returning all data - which is not necessary</li>
<li>Wrapped everything in try/catch for graceful error handling
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Pool</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">pg</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">pool</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Pool</span><span class="p">({</span>
  <span class="na">user</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_USER</span><span class="p">,</span>
  <span class="na">host</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_HOST</span><span class="p">,</span>
  <span class="na">database</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_NAME</span><span class="p">,</span>
  <span class="na">password</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PASSWORD</span><span class="p">,</span>
  <span class="na">port</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">DB_PORT</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">CURRENCY_MAP</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">us</span><span class="p">:</span> <span class="dl">'</span><span class="s1">USD</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">ca</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CAD</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">au</span><span class="p">:</span> <span class="dl">'</span><span class="s1">AUD</span><span class="dl">'</span><span class="p">,</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">EventHandler</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">orders</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchShopifyOrders</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">cleanedOrders</span> <span class="o">=</span> <span class="nx">orders</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">order</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
      <span class="na">orderNumber</span><span class="p">:</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">orderNumber</span><span class="p">,</span>
      <span class="na">currency</span><span class="p">:</span> <span class="nx">CURRENCY_MAP</span><span class="p">[</span><span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">country</span><span class="p">]</span> <span class="o">??</span> <span class="dl">'</span><span class="s1">USD</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">price</span><span class="p">:</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">discountPrice</span> <span class="o">??</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">price</span><span class="p">,</span>
      <span class="na">delivered</span><span class="p">:</span> <span class="nx">order</span><span class="p">.</span><span class="nx">details</span><span class="p">.</span><span class="nx">deliveredStatus</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">delivered</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">hubspotId</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span>
      <span class="na">hubspotOwner</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span>
    <span class="p">}));</span>

    <span class="kd">const</span> <span class="nx">hubspotResults</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span>
      <span class="nx">cleanedOrders</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">order</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">fetchHubspotData</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">orderNumber</span><span class="p">))</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">enrichedOrders</span> <span class="o">=</span> <span class="nx">cleanedOrders</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">order</span><span class="p">,</span> <span class="nx">i</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
      <span class="p">...</span><span class="nx">order</span><span class="p">,</span>
      <span class="na">hubspotId</span><span class="p">:</span> <span class="nx">hubspotResults</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">id</span> <span class="o">??</span> <span class="kc">null</span><span class="p">,</span>
      <span class="na">hubspotOwner</span><span class="p">:</span> <span class="nx">hubspotResults</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">dealOwner</span> <span class="o">||</span> <span class="kc">null</span><span class="p">,</span>
    <span class="p">}));</span>

    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">pool</span><span class="p">.</span><span class="nf">connect</span><span class="p">();</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span>
        <span class="nx">enrichedOrders</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">order</span><span class="p">)</span> <span class="o">=&gt;</span>
          <span class="nx">client</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span>
            <span class="s2">`INSERT INTO DEALS (orderNumber, currency, price, delivered, hubspotId, hubspotOwner)
             VALUES ($1, $2, $3, $4, $5, $6)
             RETURNING id`</span><span class="p">,</span>
            <span class="p">[</span><span class="nx">order</span><span class="p">.</span><span class="nx">orderNumber</span><span class="p">,</span> <span class="nx">order</span><span class="p">.</span><span class="nx">currency</span><span class="p">,</span> <span class="nx">order</span><span class="p">.</span><span class="nx">price</span><span class="p">,</span> <span class="nx">order</span><span class="p">.</span><span class="nx">delivered</span><span class="p">,</span> <span class="nx">order</span><span class="p">.</span><span class="nx">hubspotId</span><span class="p">,</span> <span class="nx">order</span><span class="p">.</span><span class="nx">hubspotOwner</span><span class="p">]</span>
          <span class="p">)</span>
        <span class="p">)</span>
      <span class="p">);</span>

      <span class="k">return</span> <span class="nx">results</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">r</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">rows</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">id</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="nx">client</span><span class="p">.</span><span class="nf">release</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">EventHandler failed:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
    <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">fetchShopifyOrders</span> <span class="o">=</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SHOPIFY_URL</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="na">API_KEY</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SHOPIFY_API_KEY</span> <span class="p">},</span>
  <span class="p">});</span>
  <span class="k">return</span> <span class="nx">response</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">fetchHubspotData</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">orderId</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">HUBSPOT_URL</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="na">API_KEY</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">HUBSPOT_API_KEY</span> <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="p">{</span> <span class="na">orderNumber</span><span class="p">:</span> <span class="nx">orderId</span> <span class="p">},</span>
  <span class="p">});</span>
  <span class="k">return</span> <span class="nx">response</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<p>The above code examples are brief indications of what the code-base looked like, in reality there were five other scripts and 5 other DB tables that were being populated. Each of these scripts contained at least 400 lines of spaghetti code. All code followed the same patterns above and each had to be rebuilt from the same foundation up.</p>

<p>The irony wasn't lost on me that I used AI tools during the refactor — for documentation, sanity checking logic, and comparing approaches. The difference was that I was driving. Understanding the problem space deeply enough to catch what the AI got wrong, knowing when to trust it and when to question it, is the skill. The code doesn't lie, and an 80% success rate on sales data will always find its way back to you.</p>

