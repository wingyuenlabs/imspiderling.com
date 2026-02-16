---
Title: EF Core HasQueryFilter II
Description: 
Author: Karen Payne
Date: 2026-02-16T21:26:05.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>EF Core has the HasQueryFilter extension method, which is a LINQ query predicate applied to EF Core entity models. This is especially useful in scenarios requiring soft deletion. A developer may need to view data without a filter, which is done using <a href="https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.ignorequeryfilters?view=efcore-10.0" rel="noopener noreferrer">IgnoreQueryFilters</a> extension method.</p>

<p><a href="https://github.com/karenpayneoregon/ef-code-8-samples/tree/master/HasQueryFilterConditionalSample" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">Source code</a>
</p>

<h2>
  
  
  Learn how
</h2>

<ul>
<li>Filter on properties rather than soft deletes and archiving, see the <a href="https://dev.to/karenpayneoregon/ef-core-global-query-filters-1dal">following</a> for soft deletes.</li>
<li>Control filters using settings from appsettings.json
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"ContextOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"UseAuditInterceptor"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"CustomersOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"UseQueryFilter"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
      </span><span class="nl">"CountryCode"</span><span class="p">:</span><span class="w"> </span><span class="mi">20</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"CategoryOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"UseQueryFilter"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Id"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Example 1
</h2>

<p>Using a customer table in a database, filter by country code.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpohpnr5t2stsy8w13lbb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpohpnr5t2stsy8w13lbb.png" alt="Customers and Countries table and relations" width="800" height="457"></a></p>

<p>We can hard-code a country code in the OnModelCreating method of the dbContext.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Customer</span><span class="p">&gt;()</span>
    <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="n">c</span> <span class="p">=&gt;</span>
        <span class="n">EF</span><span class="p">.</span><span class="n">Property</span><span class="p">&lt;</span><span class="kt">int</span><span class="p">?&gt;(</span><span class="n">c</span><span class="p">,</span>
            <span class="k">nameof</span><span class="p">(</span><span class="n">Models</span><span class="p">.</span><span class="n">Customer</span><span class="p">.</span><span class="n">CountryIdentifier</span><span class="p">))</span> <span class="p">==</span> <span class="m">20</span><span class="p">);</span> <span class="c1">// 20 is the CountryIdentifier for USA in Northwind database</span>
</code></pre>

</div>



<p>Or use a bool and int properties from appsettings.json.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnfng72vzbqsoyaq7h3d7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnfng72vzbqsoyaq7h3d7.png" alt="settings for Customer model in appsettings.json" width="509" height="307"></a></p>

<p>Below <strong>HasQueryFilter</strong> is wrapped with an if condition to use HasQueryFilter or not.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">if</span> <span class="p">(</span><span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CustomerOptions</span><span class="p">.</span><span class="n">UseQueryFilter</span><span class="p">)</span>
<span class="p">{</span>

    <span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Customer</span><span class="p">&gt;()</span>
        <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="n">c</span> <span class="p">=&gt;</span>
            <span class="n">EF</span><span class="p">.</span><span class="n">Property</span><span class="p">&lt;</span><span class="kt">int</span><span class="p">?&gt;(</span><span class="n">c</span><span class="p">,</span>
                <span class="k">nameof</span><span class="p">(</span><span class="n">Models</span><span class="p">.</span><span class="n">Customer</span><span class="p">.</span><span class="n">CountryIdentifier</span><span class="p">))</span> <span class="p">==</span>
            <span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CustomerOptions</span><span class="p">.</span><span class="n">CountryCode</span><span class="p">);</span>
<span class="p">}</span>



<span class="k">public</span> <span class="k">class</span> <span class="nc">CustomerOptions</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">bool</span> <span class="n">UseQueryFilter</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="kt">int</span> <span class="n">CountryCode</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>



<span class="k">public</span> <span class="k">sealed</span> <span class="k">class</span> <span class="nc">ContextSettings</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">static</span> <span class="k">readonly</span> <span class="n">Lazy</span><span class="p">&lt;</span><span class="n">ContextSettings</span><span class="p">&gt;</span> <span class="n">Lazy</span> <span class="p">=</span> <span class="k">new</span> <span class="n">Lazy</span><span class="p">&lt;</span><span class="n">ContextSettings</span><span class="p">&gt;(()</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nf">ContextSettings</span><span class="p">());</span>

    <span class="k">public</span> <span class="k">static</span> <span class="n">ContextSettings</span> <span class="n">Instance</span> <span class="p">=&gt;</span> <span class="n">ContextSettings</span><span class="p">.</span><span class="n">Lazy</span><span class="p">.</span><span class="n">Value</span><span class="p">;</span>

    <span class="k">public</span> <span class="n">CustomerOptions</span> <span class="n">CustomerOptions</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>

      <span class="k">private</span> <span class="nf">ContextSettings</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="c1">// Config is set up in the project file to bind the configuration</span>
        <span class="c1">// to the ContextOptions class, so we can directly retrieve it here.</span>
        <span class="n">ContextOptions</span> <span class="n">options</span> <span class="p">=</span> <span class="n">Config</span><span class="p">.</span><span class="n">Configuration</span><span class="p">.</span><span class="nf">JsonRoot</span><span class="p">()</span>
            <span class="p">.</span><span class="nf">GetSection</span><span class="p">(</span><span class="k">nameof</span><span class="p">(</span><span class="n">ContextOptions</span><span class="p">))</span>
            <span class="p">.</span><span class="n">Get</span><span class="p">&lt;</span><span class="n">ContextOptions</span><span class="p">&gt;()!;</span>

        <span class="n">CustomerOptions</span> <span class="p">=</span> <span class="n">options</span><span class="p">.</span><span class="n">CustomersOptions</span><span class="p">;</span> 
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Example 2
</h2>

<p>Uses the same logic as in example 1, this time to use HasQueryFilter for filtering on a <a href="https://github.com/karenpayneoregon/ef-code-8-samples/blob/897b080f9dca3e7618397f49a963e8faa00348a9/HasQueryFilterConditionalSample/Models/Category.cs?plain=1#L8C1-L8C30" rel="noopener noreferrer">category</a> for <a href="https://github.com/karenpayneoregon/ef-code-8-samples/blob/897b080f9dca3e7618397f49a963e8faa00348a9/HasQueryFilterConditionalSample/Models/Product.cs?plain=1#L8C1-L8C29" rel="noopener noreferrer">products</a>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">if</span> <span class="p">(</span><span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">UseQueryFilter</span><span class="p">)</span>
<span class="p">{</span>

    <span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Category</span><span class="p">&gt;()</span>
        <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="n">c</span> <span class="p">=&gt;</span>
            <span class="n">EF</span><span class="p">.</span><span class="n">Property</span><span class="p">&lt;</span><span class="kt">int</span><span class="p">?&gt;(</span><span class="n">c</span><span class="p">,</span>
                <span class="k">nameof</span><span class="p">(</span><span class="n">Models</span><span class="p">.</span><span class="n">Category</span><span class="p">.</span><span class="n">CategoryId</span><span class="p">))</span> <span class="p">==</span>
            <span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>

<span class="p">}</span>
</code></pre>

</div>



<p>Another option, perform a contains for multiple category identifiers.</p>

<p>appsettings.json</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F48gkmcj14ebuvbi2sl67.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F48gkmcj14ebuvbi2sl67.png" alt="added an integer array" width="537" height="173"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">if</span> <span class="p">(</span><span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">UseQueryFilter</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">var</span> <span class="n">ids</span> <span class="p">=</span> <span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">Identifiers</span><span class="p">;</span>
    <span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Category</span><span class="p">&gt;()</span>
        <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="n">c</span> <span class="p">=&gt;</span>
            <span class="n">ids</span><span class="p">.</span><span class="nf">Contains</span><span class="p">(</span><span class="n">EF</span><span class="p">.</span><span class="n">Property</span><span class="p">&lt;</span><span class="kt">int</span><span class="p">&gt;(</span><span class="n">c</span><span class="p">,</span> <span class="k">nameof</span><span class="p">(</span><span class="n">Models</span><span class="p">.</span><span class="n">Category</span><span class="p">.</span><span class="n">CategoryId</span><span class="p">))));</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Sample code uses a conditional to try out both.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="cp">#if UseThis
</span>            <span class="k">if</span> <span class="p">(</span><span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">UseQueryFilter</span><span class="p">)</span>
            <span class="p">{</span>

                <span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Category</span><span class="p">&gt;()</span>
                    <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="n">c</span> <span class="p">=&gt;</span>
                        <span class="n">EF</span><span class="p">.</span><span class="n">Property</span><span class="p">&lt;</span><span class="kt">int</span><span class="p">?&gt;(</span><span class="n">c</span><span class="p">,</span>
                            <span class="k">nameof</span><span class="p">(</span><span class="n">Models</span><span class="p">.</span><span class="n">Category</span><span class="p">.</span><span class="n">CategoryId</span><span class="p">))</span> <span class="p">==</span>
                        <span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">Id</span><span class="p">);</span>

            <span class="p">}</span>
<span class="cp">#else
</span>            <span class="k">if</span> <span class="p">(</span><span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">UseQueryFilter</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="kt">var</span> <span class="n">ids</span> <span class="p">=</span> <span class="n">ContextSettings</span><span class="p">.</span><span class="n">Instance</span><span class="p">.</span><span class="n">CategoryOptions</span><span class="p">.</span><span class="n">Identifiers</span><span class="p">;</span>
                <span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Category</span><span class="p">&gt;()</span>
                    <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="n">c</span> <span class="p">=&gt;</span>
                        <span class="n">ids</span><span class="p">.</span><span class="nf">Contains</span><span class="p">(</span><span class="n">EF</span><span class="p">.</span><span class="n">Property</span><span class="p">&lt;</span><span class="kt">int</span><span class="p">&gt;(</span><span class="n">c</span><span class="p">,</span> <span class="k">nameof</span><span class="p">(</span><span class="n">Models</span><span class="p">.</span><span class="n">Category</span><span class="p">.</span><span class="n">CategoryId</span><span class="p">))));</span>
            <span class="p">}</span>
<span class="cp">#endif
</span></code></pre>

</div>



<h2>
  
  
  EF Core 10: Using multiple query filters
</h2>

<p>Introduces <a href="https://learn.microsoft.com/en-us/ef/core/querying/filters?tabs=ef10#using-multiple-query-filters" rel="noopener noreferrer">named query filters</a>, which allow you to manage each filter separately, including selectively disabling one but not the other.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">modelBuilder</span><span class="p">.</span><span class="n">Entity</span><span class="p">&lt;</span><span class="n">Blog</span><span class="p">&gt;()</span>
    <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="s">"SoftDeletionFilter"</span><span class="p">,</span> <span class="n">b</span> <span class="p">=&gt;</span> <span class="p">!</span><span class="n">b</span><span class="p">.</span><span class="n">IsDeleted</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">HasQueryFilter</span><span class="p">(</span><span class="s">"TenantFilter"</span><span class="p">,</span> <span class="n">b</span> <span class="p">=&gt;</span> <span class="n">b</span><span class="p">.</span><span class="n">TenantId</span> <span class="p">==</span> <span class="n">tenantId</span><span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  Summary
</h2>

<p>This article has provided some alternative uses of global filtering in EF Core beyond what most web examples offer, such as soft deletes and the ability to control filter enabling without recompiling the application.</p>

