---
Title: When Small Method Choices Cascade Into Big Performance Wins
Description: 
Author: Paul Keen
Date: 2025-09-03T21:45:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>Three months ago, I spent an embarrassing amount of time optimizing a complex Redis caching layer - tweaking expiration strategies, adding compression, even experimenting with different serialization formats. The performance improvements were modest at best. Then, almost accidentally, I discovered that a single line of string processing code was consuming 40% of our CPU cycles.</p>

<p>The culprit? Using <code>gsub</code> instead of <code>tr</code> for simple character replacement.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># What we had (processing thousands of slugs per request)</span>
<span class="n">slug</span><span class="p">.</span><span class="nf">gsub</span><span class="p">(</span><span class="s1">'-'</span><span class="p">,</span> <span class="s1">' '</span><span class="p">)</span>

<span class="c1"># The simple fix that changed everything</span>
<span class="n">slug</span><span class="p">.</span><span class="nf">tr</span><span class="p">(</span><span class="s1">'-'</span><span class="p">,</span> <span class="s1">' '</span><span class="p">)</span>
</code></pre>

</div>



<p>Here's what that looked like in our benchmarks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="nb">require</span> <span class="s1">'benchmark/ips'</span>

<span class="no">SLUG</span> <span class="o">=</span> <span class="s1">'high-performance-web-apps'</span>

<span class="no">Benchmark</span><span class="p">.</span><span class="nf">ips</span> <span class="k">do</span> <span class="o">|</span><span class="n">x</span><span class="o">|</span>
  <span class="n">x</span><span class="p">.</span><span class="nf">report</span><span class="p">(</span><span class="s1">'gsub'</span><span class="p">)</span> <span class="p">{</span> <span class="no">SLUG</span><span class="p">.</span><span class="nf">gsub</span><span class="p">(</span><span class="s1">'-'</span><span class="p">,</span> <span class="s1">' '</span><span class="p">)</span> <span class="p">}</span>
  <span class="n">x</span><span class="p">.</span><span class="nf">report</span><span class="p">(</span><span class="s1">'tr'</span><span class="p">)</span>   <span class="p">{</span> <span class="no">SLUG</span><span class="p">.</span><span class="nf">tr</span><span class="p">(</span><span class="s1">'-'</span><span class="p">,</span> <span class="s1">' '</span><span class="p">)</span> <span class="p">}</span>
  <span class="n">x</span><span class="p">.</span><span class="nf">compare!</span>
<span class="k">end</span>

<span class="c1"># String#tr: 2,121,629.8 i/s</span>
<span class="c1"># String#gsub: 474,970.5 i/s - 4.90x slower</span>
</code></pre>

</div>



<p>The difference isn't subtle - we're talking about a 5x performance gap. When you're processing thousands of these conversions per request, that compounds fast.</p>

<p>This experience taught me something uncomfortable: sometimes the biggest performance wins hide in the smallest details. While I was architecting elaborate caching strategies, the real bottleneck was a basic method choice.</p>

<h2>
  
  
  The count/length/size confusion that keeps biting people
</h2>

<p>Rails developers run into this trap constantly, and I've done it myself more times than I care to admit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># This will load every single record into memory</span>
<span class="n">users</span><span class="p">.</span><span class="nf">length</span>  <span class="c1"># SELECT * FROM users (!!!)</span>

<span class="c1"># This hits the database with COUNT(*)</span>
<span class="n">users</span><span class="p">.</span><span class="nf">count</span>   <span class="c1"># SELECT COUNT(*) FROM users</span>

<span class="c1"># This tries to be smart about it</span>
<span class="n">users</span><span class="p">.</span><span class="nf">size</span>    <span class="c1"># Uses COUNT(*) if not loaded, length if already in memory</span>
</code></pre>

</div>



<p>I watched a colleague's dashboard go from snappy to timing out because they assumed <code>.length</code> and <code>.count</code> were interchangeable. The users table had grown to 800,000 records. That <code>.length</code> call suddenly meant loading nearly a million ActiveRecord objects just to count them.</p>

<p>The frustrating part? This knowledge exists in every Rails developer's head somewhere. But under deadline pressure, these details slip through code review.</p>

<h2>
  
  
  N+1 queries have evolved (and gotten sneakier)
</h2>

<p>Everyone knows about the classic N+1 problem:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="n">posts</span> <span class="o">=</span> <span class="no">Post</span><span class="p">.</span><span class="nf">all</span>
<span class="n">posts</span><span class="p">.</span><span class="nf">each</span> <span class="k">do</span> <span class="o">|</span><span class="n">post</span><span class="o">|</span>
  <span class="nb">puts</span> <span class="n">post</span><span class="p">.</span><span class="nf">comments</span><span class="p">.</span><span class="nf">count</span>  <span class="c1"># N database hits</span>
<span class="k">end</span>
</code></pre>

</div>



<p>But modern applications create more sophisticated variants. Last year I debugged a GraphQL API where the query pattern looked innocent enough, but nested resolvers were creating exponential query growth. Not N+1, but N×M×P queries spiraling into thousands of database hits.</p>

<p>The textbook solution is <code>includes</code>, but that brings its own problems:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Seems right, but can be a memory bomb</span>
<span class="no">Post</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="ss">:comments</span><span class="p">,</span> <span class="ss">comments: </span><span class="p">[</span><span class="ss">:author</span><span class="p">,</span> <span class="ss">:reactions</span><span class="p">])</span>

<span class="c1"># Better for large datasets - be selective</span>
<span class="no">Post</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="ss">:comments</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">where</span><span class="p">(</span><span class="ss">comments: </span><span class="p">{</span> <span class="ss">status: </span><span class="s1">'approved'</span> <span class="p">})</span>
    <span class="p">.</span><span class="nf">limit</span><span class="p">(</span><span class="mi">20</span><span class="p">)</span>
</code></pre>

</div>



<p>Here's something the Rails guides don't emphasize enough: <code>includes</code> can actually make things worse for large datasets. When you're joining tables with hundreds of thousands of rows, that LEFT OUTER JOIN might generate a cartesian product that exhausts your memory before Rails even starts instantiating objects.</p>

<p>For read-heavy endpoints, I've started reaching for PostgreSQL's JSON aggregation instead:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Skip ActiveRecord entirely for heavy read operations</span>
<span class="k">SELECT</span> <span class="n">categories</span><span class="p">.</span><span class="o">*</span><span class="p">,</span> 
       <span class="n">json_agg</span><span class="p">(</span>
         <span class="n">json_build_object</span><span class="p">(</span>
           <span class="s1">'id'</span><span class="p">,</span> <span class="n">items</span><span class="p">.</span><span class="n">id</span><span class="p">,</span>
           <span class="s1">'name'</span><span class="p">,</span> <span class="n">items</span><span class="p">.</span><span class="n">name</span><span class="p">,</span>
           <span class="s1">'price'</span><span class="p">,</span> <span class="n">items</span><span class="p">.</span><span class="n">price</span>
         <span class="p">)</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">items</span><span class="p">.</span><span class="n">name</span>
       <span class="p">)</span> <span class="k">as</span> <span class="n">items_json</span>
<span class="k">FROM</span> <span class="n">categories</span>
<span class="k">LEFT</span> <span class="k">JOIN</span> <span class="n">items</span> <span class="k">ON</span> <span class="n">items</span><span class="p">.</span><span class="n">category_id</span> <span class="o">=</span> <span class="n">categories</span><span class="p">.</span><span class="n">id</span>
<span class="k">GROUP</span> <span class="k">BY</span> <span class="n">categories</span><span class="p">.</span><span class="n">id</span><span class="p">;</span>
</code></pre>

</div>



<p>You lose ActiveRecord's convenience, but gain 50-90% reduction in memory usage. Sometimes that trade-off makes sense.</p>

<h2>
  
  
  Database indexes that actually match your queries
</h2>

<p>Adding indexes feels straightforward until you start looking at your actual query patterns. Many developers create single-column indexes without considering how their WHERE clauses work together.</p>

<p>Here's a real example from an e-commerce platform:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- What most people start with</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_orders_user_id</span> <span class="k">ON</span> <span class="n">orders</span><span class="p">(</span><span class="n">user_id</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_orders_created_at</span> <span class="k">ON</span> <span class="n">orders</span><span class="p">(</span><span class="n">created_at</span><span class="p">);</span>

<span class="c1">-- What actually matches the query pattern</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_orders_user_date_status</span> 
  <span class="k">ON</span> <span class="n">orders</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">created_at</span> <span class="k">DESC</span><span class="p">,</span> <span class="n">status</span><span class="p">)</span>
  <span class="n">INCLUDE</span> <span class="p">(</span><span class="n">total_amount</span><span class="p">,</span> <span class="n">shipping_address_id</span><span class="p">);</span>
</code></pre>

</div>



<p>That <code>INCLUDE</code> clause creates a covering index - PostgreSQL can satisfy the entire query without touching the main table. For their most common dashboard query, this cut I/O operations by 73%.</p>

<p>But here's where it gets interesting: PostgreSQL's query planner is cost-based, not rule-based. It looks at your data distribution to choose between available indexes. I've seen perfectly good indexes get ignored because the table statistics were stale.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Force a statistics update after major data changes</span>
<span class="k">ANALYZE</span> <span class="n">orders</span><span class="p">;</span>

<span class="c1">-- For skewed data distributions, increase sample size</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">orders</span> <span class="k">ALTER</span> <span class="k">COLUMN</span> <span class="n">user_id</span> <span class="k">SET</span> <span class="k">STATISTICS</span> <span class="mi">1000</span><span class="p">;</span>
</code></pre>

</div>



<p>That last line can be a game-changer. The default statistics target is 100 samples, but if you have columns where some values are much more common than others (like user_id where power users have thousands of orders), increasing this helps the planner make better choices.</p>

<h2>
  
  
  Caching strategies that don't blow up in production
</h2>

<p>After debugging enough cache-related outages, I've learned that caching isn't really about storing data temporarily. It's about managing state synchronization across distributed systems while keeping things roughly consistent.</p>

<p>The approach that's worked best for me involves multiple layers with different consistency guarantees:</p>

<p><strong>Browser caching for immutable assets:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Webpack config for long-term caching</span>
<span class="nx">output</span><span class="p">:</span> <span class="p">{</span>
  <span class="nl">filename</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[name].[contenthash].js</span><span class="dl">'</span><span class="p">,</span>
  <span class="nx">chunkFilename</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id].[contenthash].chunk.js</span><span class="dl">'</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Redis for application state:</strong><br>
The pattern I keep coming back to handles the cache stampede problem - when your cache expires during a traffic spike and every request triggers an expensive recomputation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">CacheManager</span>
  <span class="k">def</span> <span class="nf">fetch_with_grace</span><span class="p">(</span><span class="n">key</span><span class="p">,</span> <span class="ss">ttl: </span><span class="mi">3600</span><span class="p">,</span> <span class="ss">grace_period: </span><span class="mi">60</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">block</span><span class="p">)</span>
    <span class="n">value</span><span class="p">,</span> <span class="n">remaining_ttl</span> <span class="o">=</span> <span class="n">redis</span><span class="p">.</span><span class="nf">multi</span> <span class="k">do</span> <span class="o">|</span><span class="n">r</span><span class="o">|</span>
      <span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">key</span><span class="p">)</span>
      <span class="n">r</span><span class="p">.</span><span class="nf">ttl</span><span class="p">(</span><span class="n">key</span><span class="p">)</span>
    <span class="k">end</span>

    <span class="c1"># Serve stale content while recomputing in background</span>
    <span class="k">if</span> <span class="n">value</span> <span class="o">&amp;&amp;</span> <span class="n">remaining_ttl</span> <span class="o">&lt;</span> <span class="n">grace_period</span>
      <span class="no">Thread</span><span class="p">.</span><span class="nf">new</span> <span class="p">{</span> <span class="n">recompute_and_store</span><span class="p">(</span><span class="n">key</span><span class="p">,</span> <span class="n">ttl</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">block</span><span class="p">)</span> <span class="p">}</span>
      <span class="k">return</span> <span class="no">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="n">value</span><span class="p">)</span>
    <span class="k">end</span>

    <span class="k">return</span> <span class="no">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="n">value</span><span class="p">)</span> <span class="k">if</span> <span class="n">value</span>

    <span class="c1"># First miss - compute with distributed lock</span>
    <span class="n">redis</span><span class="p">.</span><span class="nf">with_lock</span><span class="p">(</span><span class="s2">"lock:</span><span class="si">#{</span><span class="n">key</span><span class="si">}</span><span class="s2">"</span><span class="p">,</span> <span class="ss">timeout: </span><span class="mi">10</span><span class="p">)</span> <span class="k">do</span>
      <span class="n">recompute_and_store</span><span class="p">(</span><span class="n">key</span><span class="p">,</span> <span class="n">ttl</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">block</span><span class="p">)</span>
    <span class="k">end</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>This pattern has prevented numerous outages. Instead of having 100 concurrent requests all trying to regenerate the same expensive data, one request does the work while others get slightly stale content.</p>

<p><strong>Fragment caching that actually works:</strong><br>
Rails' Russian Doll caching is underused, probably because it requires thinking about cache dependencies upfront:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight erb"><code><span class="cp">&lt;%</span> <span class="n">cache</span> <span class="p">[</span><span class="s1">'v1'</span><span class="p">,</span> <span class="vi">@product</span><span class="p">,</span> <span class="vi">@current_user</span><span class="o">&amp;</span><span class="p">.</span><span class="nf">cache_key</span><span class="p">]</span> <span class="k">do</span> <span class="cp">%&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"product"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h2&gt;</span><span class="cp">&lt;%=</span> <span class="vi">@product</span><span class="p">.</span><span class="nf">name</span> <span class="cp">%&gt;</span><span class="nt">&lt;/h2&gt;</span>

    <span class="cp">&lt;%</span> <span class="n">cache</span> <span class="p">[</span><span class="vi">@product</span><span class="p">,</span> <span class="s1">'pricing'</span><span class="p">]</span> <span class="k">do</span> <span class="cp">%&gt;</span>
      <span class="cp">&lt;%=</span> <span class="n">render</span> <span class="s1">'pricing_info'</span><span class="p">,</span> <span class="ss">product: </span><span class="vi">@product</span> <span class="cp">%&gt;</span>
    <span class="cp">&lt;%</span> <span class="k">end</span> <span class="cp">%&gt;</span>

    <span class="cp">&lt;%</span> <span class="vi">@product</span><span class="p">.</span><span class="nf">variants</span><span class="p">.</span><span class="nf">each</span> <span class="k">do</span> <span class="o">|</span><span class="n">variant</span><span class="o">|</span> <span class="cp">%&gt;</span>
      <span class="cp">&lt;%</span> <span class="n">cache</span> <span class="n">variant</span> <span class="k">do</span> <span class="cp">%&gt;</span>
        <span class="cp">&lt;%=</span> <span class="n">render</span> <span class="s1">'variant'</span><span class="p">,</span> <span class="ss">variant: </span><span class="n">variant</span> <span class="cp">%&gt;</span>
      <span class="cp">&lt;%</span> <span class="k">end</span> <span class="cp">%&gt;</span>
    <span class="cp">&lt;%</span> <span class="k">end</span> <span class="cp">%&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
<span class="cp">&lt;%</span> <span class="k">end</span> <span class="cp">%&gt;</span>
</code></pre>

</div>



<p>When a single variant updates, only that fragment invalidates. For a product with 50 variants, that's 2% cache invalidation instead of 100%. The <code>'v1'</code> version string lets you bust all fragments when you deploy template changes - without it, you'll serve mixed HTML from different code versions.</p>

<h2>
  
  
  Making CI/CD faster (the productivity multiplier everyone ignores)
</h2>

<p>Every minute you shave off CI is a minute returned to every developer, multiple times per day. The compound effect adds up to hours per week per person.</p>

<p><strong>Docker layer optimization:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">node:18-alpine</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>

<span class="c"># System dependencies change rarely - cache these layers</span>
<span class="k">RUN </span>apk add <span class="nt">--no-cache</span> python3 make g++

<span class="c"># Dependencies change occasionally</span>
<span class="k">WORKDIR</span><span class="s"> /app</span>
<span class="k">COPY</span><span class="s"> package*.json ./</span>
<span class="k">RUN </span>npm ci <span class="nt">--only</span><span class="o">=</span>production

<span class="c"># App code changes frequently - put this last</span>
<span class="k">COPY</span><span class="s"> . .</span>
<span class="k">RUN </span>npm run build
</code></pre>

</div>



<p>But the real win is using BuildKit's cache mounts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">RUN </span><span class="nt">--mount</span><span class="o">=</span><span class="nb">type</span><span class="o">=</span>cache,target<span class="o">=</span>/root/.npm <span class="se">\
</span>    npm ci <span class="nt">--only</span><span class="o">=</span>production
</code></pre>

</div>



<p>This persists npm's cache between builds. Three-minute dependency installs become 20-second cache hits.</p>

<p><strong>Parallel test execution:</strong><br>
The trick isn't just splitting tests - it's ensuring they don't interfere:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># config/database.yml</span>
<span class="ss">test:
  database: </span><span class="n">myapp_test</span><span class="o">&lt;</span><span class="sx">%= ENV['TEST_ENV_NUMBER'] %&gt;

# Each process gets its own schema
$ parallel_rspec spec/
</span></code></pre>

</div>



<p>We went from 45-minute test suites to 12 minutes with four parallel processes.</p>

<p><strong>Dealing with flaky tests:</strong><br>
Rather than letting flaky tests poison your entire suite, quarantine them:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// jest.config.js</span>
<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">projects</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="na">displayName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">stable</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">testMatch</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">&lt;rootDir&gt;/src/**/*.test.js</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">testPathIgnorePatterns</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">&lt;rootDir&gt;/src/**/*.flaky.test.js</span><span class="dl">'</span><span class="p">]</span>
    <span class="p">},</span>
    <span class="p">{</span>
      <span class="na">displayName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">quarantine</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">testMatch</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">&lt;rootDir&gt;/src/**/*.flaky.test.js</span><span class="dl">'</span><span class="p">],</span>
      <span class="na">retries</span><span class="p">:</span> <span class="mi">3</span>
    <span class="p">}</span>
  <span class="p">]</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Flaky tests run separately with retries. They don't block deployments, but they also don't get ignored.</p>

<h2>
  
  
  Performance budgets as guardrails
</h2>

<p>Teams that maintain good performance over time have embedded performance constraints into their development process:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"bundles"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"main.js"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"maxSize"</span><span class="p">:</span><span class="w"> </span><span class="s2">"300 KB"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"warning"</span><span class="p">:</span><span class="w"> </span><span class="s2">"250 KB"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"metrics"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"firstContentfulPaint"</span><span class="p">:</span><span class="w"> </span><span class="mi">1500</span><span class="p">,</span><span class="w">
    </span><span class="nl">"largestContentfulPaint"</span><span class="p">:</span><span class="w"> </span><span class="mi">2500</span><span class="p">,</span><span class="w">
    </span><span class="nl">"totalBlockingTime"</span><span class="p">:</span><span class="w"> </span><span class="mi">200</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Every pull request runs against these budgets. Exceed them and the build fails. Want to add that 50KB analytics library? Fine, but what are you removing to stay under budget?</p>

<h2>
  
  
  Some numbers from actual systems
</h2>

<p><strong>E-commerce platform (last year):</strong></p>

<ul>
<li>Switched from individual Redis keys to hash data structures</li>
<li>Added covering indexes for product catalog queries
</li>
<li>Implemented fragment caching with 5-minute TTL</li>
<li>
<strong>Result:</strong> 47ms → 12ms 95th percentile response time</li>
</ul>

<p><strong>B2B SaaS dashboard:</strong></p>

<ul>
<li>Replaced nested GraphQL N+1 patterns with DataLoader</li>
<li>Set up database connection pooling via PgBouncer</li>
<li>Moved CI from Jenkins to GitHub Actions with build caching</li>
<li>
<strong>Result:</strong> 3.2s → 340ms API response time, 45-minute → 8-minute builds</li>
</ul>

<h2>
  
  
  Technologies worth watching
</h2>

<p><strong>eBPF for production observability:</strong><br>
Being able to trace performance issues in production with near-zero overhead changes how you debug problems. No more "works fine in staging" mysteries.</p>

<p><strong>WASM at the edge:</strong><br>
For compute-intensive operations, running WebAssembly modules at edge locations gives you 95% of native performance with 50x faster cold starts than containers.</p>

<p><strong>Ruby's YJIT compiler:</strong><br>
If you're on Ruby 3.2+, enabling YJIT can improve performance by 15-40% with a one-line configuration change.</p>

<h2>
  
  
  Where to start
</h2>

<p>If this resonates but feels overwhelming, pick one area:</p>

<p><strong>Week 1:</strong> Add basic monitoring. Get 95th percentile response times on a dashboard you actually look at.</p>

<p><strong>Week 2:</strong> Find your worst N+1 query and fix it. Use something like rack-mini-profiler to spot them.</p>

<p><strong>Week 3:</strong> Add one performance test to CI - just your most critical user path.</p>

<p><strong>Week 4:</strong> Set up performance budgets for your main JavaScript bundle.</p>

<p>Performance work compounds. Minor improvements stack up over time, but only if you measure consistently and fix regressions quickly.</p>

<p>The teams that build fast applications treat performance as a feature, not an afterthought. They measure continuously, optimize incrementally, and never assume their systems will stay fast without attention.</p>

<p>Start small, measure everything, and remember that the most impactful optimizations often hide in the details everyone takes for granted.</p>

