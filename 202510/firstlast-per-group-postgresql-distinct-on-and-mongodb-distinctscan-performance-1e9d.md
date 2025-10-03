---
Title: First/Last per Group: PostgreSQL DISTINCT ON and MongoDB DISTINCT_SCAN Performance
Description: 
Author: Franck Pachot
Date: 2025-10-03T22:08:35.000Z
Robots: noindex,nofollow
Template: index
---
<p>On Stack Overflow, the <a href="https://stackoverflow.com/questions/tagged/postgresql?tab=Frequent" rel="noopener noreferrer">most frequent question for PostgreSQL</a> is: "<em><strong>Select first row in each GROUP BY group?</strong></em>". I've written about it previously, presenting multiple alternatives and execution plans: <a href="https://www.yugabyte.com/blog/select-first-row-group-by-postgresql/" rel="noopener noreferrer">How to Select the First Row of Each Set of Grouped Rows Using GROUP BY</a>.<br>
Solving this problem in a way that's both straightforward and high-performing can be challenging with SQL databases. However, MongoDB's aggregation provides a simple syntax and an efficient execution plan. When you use <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/first/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$first</a> or <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/last/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$last</a> alongside <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/sort/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$sort</a> and <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$group</a>, MongoDB can perform an efficient <strong>loose index scan</strong>, which is similar to an <strong>index skip scan</strong>, reading only what is necessary for the result.</p>
<h2>
  
  
  PostgreSQL
</h2>

<p>With PostgreSQL, the <strong>DISTINCT ON ... ORDER BY</strong> syntax is the easiest for the developer, but not the best for performance.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">create</span> <span class="k">table</span> <span class="n">demo</span> <span class="p">(</span>
 <span class="k">primary</span> <span class="k">key</span> <span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">,</span> <span class="k">c</span><span class="p">),</span>
  <span class="n">a</span> <span class="nb">int</span><span class="p">,</span> <span class="n">b</span> <span class="n">timestamptz</span><span class="p">,</span>
  <span class="k">c</span> <span class="nb">float</span><span class="p">,</span>
  <span class="n">d</span> <span class="nb">text</span>
<span class="p">);</span>
<span class="c1">-- insert a hundred thousand rows</span>
<span class="k">insert</span> <span class="k">into</span> <span class="n">demo</span>
 <span class="k">select</span> 
  <span class="n">a</span><span class="p">,</span>
  <span class="n">now</span><span class="p">()</span> <span class="k">as</span> <span class="n">b</span><span class="p">,</span>
  <span class="n">random</span><span class="p">()</span> <span class="k">as</span> <span class="k">c</span><span class="p">,</span>
  <span class="n">repeat</span><span class="p">(</span><span class="s1">'x'</span><span class="p">,</span><span class="mi">5</span><span class="p">)</span> <span class="k">as</span> <span class="n">d</span>
 <span class="k">from</span> <span class="n">generate_series</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span><span class="mi">5</span><span class="p">)</span> <span class="n">a</span>
    <span class="p">,</span> <span class="n">generate_series</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span><span class="mi">20000</span><span class="p">)</span> <span class="k">c</span>
 <span class="c1">-- ignore bad luck random;</span>
 <span class="k">on</span> <span class="n">conflict</span> <span class="k">do</span> <span class="k">nothing</span>
<span class="p">;</span>
<span class="c1">-- run 10 more times (now() will be different):</span>
<span class="err">\</span><span class="n">watch</span> <span class="k">count</span><span class="o">=</span><span class="mi">9</span>
<span class="k">vacuum</span> <span class="k">analyze</span> <span class="n">demo</span><span class="p">;</span>
</code></pre>

</div>



<p>In PostgreSQL 18, all rows are read, but most are eliminated so that only one row remains per group:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">explain</span> <span class="p">(</span><span class="n">buffers</span><span class="p">,</span> <span class="k">analyze</span><span class="p">,</span> <span class="k">verbose</span><span class="p">,</span> <span class="n">costs</span> <span class="k">off</span><span class="p">)</span>
<span class="k">select</span>
 <span class="k">distinct</span> <span class="k">on</span> <span class="p">(</span><span class="n">b</span><span class="p">)</span> <span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">,</span> <span class="k">c</span><span class="p">,</span> <span class="n">d</span>
<span class="k">from</span> <span class="n">demo</span>
<span class="k">where</span> <span class="n">a</span><span class="o">=</span><span class="mi">1</span>
<span class="k">order</span> <span class="k">by</span> <span class="n">b</span><span class="p">,</span> <span class="k">c</span>
<span class="p">;</span>

                                             <span class="n">QUERY</span> <span class="n">PLAN</span>
<span class="c1">----------------------------------------------------------------------------------------------------</span>
 <span class="k">Unique</span> <span class="p">(</span><span class="n">actual</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">025</span><span class="p">..</span><span class="mi">94</span><span class="p">.</span><span class="mi">601</span> <span class="k">rows</span><span class="o">=</span><span class="mi">10</span><span class="p">.</span><span class="mi">00</span> <span class="n">loops</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
   <span class="k">Output</span><span class="p">:</span> <span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">,</span> <span class="k">c</span><span class="p">,</span> <span class="n">d</span>
   <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">199959</span>
   <span class="o">-&gt;</span>  <span class="k">Index</span> <span class="n">Scan</span> <span class="k">using</span> <span class="n">demo_pkey</span> <span class="k">on</span> <span class="k">public</span><span class="p">.</span><span class="n">demo</span> <span class="p">(</span><span class="n">actual</span> <span class="nb">time</span><span class="o">=</span><span class="mi">0</span><span class="p">.</span><span class="mi">024</span><span class="p">..</span><span class="mi">77</span><span class="p">.</span><span class="mi">263</span> <span class="k">rows</span><span class="o">=</span><span class="mi">200000</span><span class="p">.</span><span class="mi">00</span> <span class="n">loops</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
         <span class="k">Output</span><span class="p">:</span> <span class="n">a</span><span class="p">,</span> <span class="n">b</span><span class="p">,</span> <span class="k">c</span><span class="p">,</span> <span class="n">d</span>
         <span class="k">Index</span> <span class="n">Cond</span><span class="p">:</span> <span class="p">(</span><span class="n">demo</span><span class="p">.</span><span class="n">a</span> <span class="o">=</span> <span class="mi">1</span><span class="p">)</span>
         <span class="k">Index</span> <span class="n">Searches</span><span class="p">:</span> <span class="mi">1</span>
         <span class="n">Buffers</span><span class="p">:</span> <span class="n">shared</span> <span class="n">hit</span><span class="o">=</span><span class="mi">199959</span>
 <span class="n">Planning</span> <span class="nb">Time</span><span class="p">:</span> <span class="mi">0</span><span class="p">.</span><span class="mi">077</span> <span class="n">ms</span>
 <span class="n">Execution</span> <span class="nb">Time</span><span class="p">:</span> <span class="mi">94</span><span class="p">.</span><span class="mi">622</span> <span class="n">ms</span>
</code></pre>

</div>



<p>Although the DISTINCT ON ... ORDER BY syntax is straightforward, it is not efficient here: the number of rows processed (<code>rows=200,000.00</code>) and buffers read (<code>Buffers: shared hit=199,959</code>) is excessive compared to the result size (<code>rows=10.00</code>).</p>

<p>If you want to avoid unnecessary reads, you have to write a complex recursive CTE:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">with</span> <span class="k">recursive</span> <span class="n">skip_scan</span> <span class="k">as</span> <span class="p">(</span>
 <span class="p">(</span>
  <span class="c1">-- get the first row</span>
  <span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span>
  <span class="k">where</span> <span class="n">a</span><span class="o">=</span><span class="mi">1</span>
  <span class="k">order</span> <span class="k">by</span> <span class="n">b</span><span class="p">,</span><span class="k">c</span> <span class="k">limit</span> <span class="mi">1</span>
 <span class="p">)</span> <span class="k">union</span> <span class="k">all</span> <span class="p">(</span>
  <span class="c1">-- get the next row</span>
  <span class="k">select</span> <span class="n">demo</span><span class="p">.</span><span class="o">*</span>
  <span class="k">from</span> <span class="n">skip_scan</span> <span class="p">,</span> <span class="k">lateral</span><span class="p">(</span>
   <span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">demo</span>
   <span class="k">where</span> <span class="n">demo</span><span class="p">.</span><span class="n">a</span> <span class="o">=</span> <span class="n">skip_scan</span><span class="p">.</span><span class="n">a</span> <span class="k">and</span> <span class="n">demo</span><span class="p">.</span><span class="n">b</span> <span class="o">&gt;</span> <span class="n">skip_scan</span><span class="p">.</span><span class="n">b</span>
   <span class="k">order</span> <span class="k">by</span> <span class="n">b</span><span class="p">,</span><span class="k">c</span> <span class="k">limit</span> <span class="mi">1</span>
  <span class="p">)</span> <span class="n">demo</span>
 <span class="p">)</span>
<span class="p">)</span>
<span class="k">select</span> <span class="o">*</span> <span class="k">from</span> <span class="n">skip_scan</span>
<span class="p">;</span>
</code></pre>

</div>



<p>This simulates an index loose scan with nested loops, iterating from a recursive WITH clause.</p>

<h2>
  
  
  MongoDB aggregation
</h2>

<p>In MongoDB, using an aggregation pipeline makes it easy and efficient to get either the first or last document from each group.</p>

<p>I create a collection and fill it with similar data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Create collection and compound unique index to mimic PRIMARY KEY(a, b, c)</span>
<span class="nx">db</span><span class="p">.</span><span class="nx">demo</span><span class="p">.</span><span class="nf">drop</span><span class="p">();</span>
<span class="nx">db</span><span class="p">.</span><span class="nx">demo</span><span class="p">.</span><span class="nf">createIndex</span><span class="p">(</span>
  <span class="p">{</span> <span class="na">a</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">b</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">c</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">unique</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}</span>
<span class="p">);</span>

<span class="c1">// Function to insert bulk records</span>
<span class="kd">function</span> <span class="nf">insertDemoBatch</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">batch</span> <span class="o">=</span> <span class="p">[];</span>
  <span class="kd">const</span> <span class="nx">now</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">();</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">a</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span> <span class="nx">a</span> <span class="o">&lt;=</span> <span class="mi">5</span><span class="p">;</span> <span class="nx">a</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">j</span> <span class="o">=</span> <span class="mi">1</span><span class="p">;</span> <span class="nx">j</span> <span class="o">&lt;=</span> <span class="mi">20000</span><span class="p">;</span> <span class="nx">j</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">batch</span><span class="p">.</span><span class="nf">push</span><span class="p">({</span>
        <span class="na">a</span><span class="p">:</span> <span class="nx">a</span><span class="p">,</span>
        <span class="na">b</span><span class="p">:</span> <span class="nx">now</span><span class="p">,</span>                   <span class="c1">// similar to now()</span>
        <span class="na">c</span><span class="p">:</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">(),</span>         <span class="c1">// random float [0,1)</span>
        <span class="na">d</span><span class="p">:</span> <span class="dl">'</span><span class="s1">x</span><span class="dl">'</span><span class="p">.</span><span class="nf">repeat</span><span class="p">(</span><span class="mi">5</span><span class="p">)</span>          <span class="c1">// repeat string</span>
      <span class="p">});</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="nx">db</span><span class="p">.</span><span class="nx">demo</span><span class="p">.</span><span class="nf">insertMany</span><span class="p">(</span><span class="nx">batch</span><span class="p">,</span> <span class="p">{</span> <span class="na">ordered</span><span class="p">:</span> <span class="kc">false</span> <span class="p">});</span> <span class="c1">// ignore duplicates</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">print</span><span class="p">(</span><span class="s2">`Insert completed with some duplicates ignored: </span><span class="p">${</span><span class="nx">e</span><span class="p">.</span><span class="nx">writeErrors</span><span class="p">?.</span><span class="nx">length</span> <span class="o">??</span> <span class="mi">0</span><span class="p">}</span><span class="s2"> errors`</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Run 10 times — now() will be different each run</span>
<span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="mi">10</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="nf">insertDemoBatch</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Here is the aggregation that groups and keeps the first value of each group:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">db</span><span class="p">.</span><span class="nx">demo</span><span class="p">.</span><span class="nf">aggregate</span><span class="p">([</span>
  <span class="p">{</span> <span class="na">$match</span><span class="p">:</span> <span class="p">{</span> <span class="na">a</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span> <span class="p">},</span>         <span class="c1">// equivalent to WHERE a=1</span>
  <span class="p">{</span> <span class="na">$sort</span><span class="p">:</span> <span class="p">{</span> <span class="na">b</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">c</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span> <span class="p">},</span>    <span class="c1">// equivalent to ORDER BY b, c</span>
  <span class="p">{</span> <span class="na">$group</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">_id</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$b</span><span class="dl">"</span><span class="p">,</span>                 <span class="c1">// equivalent to DISTINCT ON (b)</span>
      <span class="na">a</span><span class="p">:</span> <span class="p">{</span> <span class="na">$first</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$a</span><span class="dl">"</span> <span class="p">},</span>       
      <span class="na">b</span><span class="p">:</span> <span class="p">{</span> <span class="na">$first</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$b</span><span class="dl">"</span> <span class="p">},</span>
      <span class="na">c</span><span class="p">:</span> <span class="p">{</span> <span class="na">$first</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$c</span><span class="dl">"</span> <span class="p">},</span>
      <span class="na">d</span><span class="p">:</span> <span class="p">{</span> <span class="na">$first</span><span class="p">:</span> <span class="dl">"</span><span class="s2">$d</span><span class="dl">"</span> <span class="p">}</span>      
  <span class="p">}},</span>
  <span class="p">{</span> <span class="na">$project</span><span class="p">:</span> <span class="p">{</span>                  <span class="c1">// equivalent to SELECT a, b, c, d</span>
      <span class="na">_id</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="na">a</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">b</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">c</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">d</span><span class="p">:</span> <span class="mi">1</span> 
  <span class="p">}},</span>
<span class="p">]).</span><span class="nf">explain</span><span class="p">(</span><span class="dl">"</span><span class="s2">executionStats</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<p>The execution plan is efficient, reading only one document per group (<code>totalDocsExamined: 10</code>) and seeking to the end of each group (<code>keysExamined: 11</code>) in the index scan:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">...</span>
        <span class="nx">executionStats</span><span class="p">:</span> <span class="p">{</span>                                                                                                                                                                            
          <span class="nl">executionSuccess</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
          <span class="nx">nReturned</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
          <span class="nx">executionTimeMillis</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
          <span class="nx">totalKeysExamined</span><span class="p">:</span> <span class="mi">11</span><span class="p">,</span>
          <span class="nx">totalDocsExamined</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
          <span class="nx">executionStages</span><span class="p">:</span> <span class="p">{</span>
            <span class="nl">isCached</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
            <span class="nx">stage</span><span class="p">:</span> <span class="dl">'</span><span class="s1">FETCH</span><span class="dl">'</span><span class="p">,</span>
            <span class="nx">nReturned</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
            <span class="nx">executionTimeMillisEstimate</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
            <span class="nx">works</span><span class="p">:</span> <span class="mi">11</span><span class="p">,</span>
            <span class="nx">advanced</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
            <span class="nx">needTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
            <span class="nx">needYield</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
            <span class="nx">saveState</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
            <span class="nx">restoreState</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
            <span class="nx">isEOF</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
            <span class="nx">docsExamined</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
            <span class="nx">alreadyHasObj</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
            <span class="nx">inputStage</span><span class="p">:</span> <span class="p">{</span>
              <span class="nl">stage</span><span class="p">:</span> <span class="dl">'</span><span class="s1">DISTINCT_SCAN</span><span class="dl">'</span><span class="p">,</span>
              <span class="nx">nReturned</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
              <span class="nx">executionTimeMillisEstimate</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
              <span class="nx">works</span><span class="p">:</span> <span class="mi">11</span><span class="p">,</span>
              <span class="nx">advanced</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span>
              <span class="nx">needTime</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
              <span class="nx">needYield</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
              <span class="nx">saveState</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
              <span class="nx">restoreState</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
              <span class="nx">isEOF</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
              <span class="nx">keyPattern</span><span class="p">:</span> <span class="p">{</span> <span class="nl">a</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="nx">b</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="nx">c</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
              <span class="nx">indexName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">a_1_b_1_c_1</span><span class="dl">'</span><span class="p">,</span>
              <span class="nx">isMultiKey</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
              <span class="nx">multiKeyPaths</span><span class="p">:</span> <span class="p">{</span> <span class="nl">a</span><span class="p">:</span> <span class="p">[],</span> <span class="nx">b</span><span class="p">:</span> <span class="p">[],</span> <span class="nx">c</span><span class="p">:</span> <span class="p">[]</span> <span class="p">},</span>
              <span class="nx">isUnique</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
              <span class="nx">isSparse</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
              <span class="nx">isPartial</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
              <span class="nx">indexVersion</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span>
              <span class="nx">direction</span><span class="p">:</span> <span class="dl">'</span><span class="s1">forward</span><span class="dl">'</span><span class="p">,</span>
              <span class="nx">indexBounds</span><span class="p">:</span> <span class="p">{</span>
                <span class="nl">a</span><span class="p">:</span> <span class="p">[</span> <span class="dl">'</span><span class="s1">[1, 1]</span><span class="dl">'</span> <span class="p">],</span>
                <span class="nx">b</span><span class="p">:</span> <span class="p">[</span> <span class="dl">'</span><span class="s1">[MinKey, MaxKey]</span><span class="dl">'</span> <span class="p">],</span>
                <span class="nx">c</span><span class="p">:</span> <span class="p">[</span> <span class="dl">'</span><span class="s1">[MinKey, MaxKey]</span><span class="dl">'</span> <span class="p">]</span>
              <span class="p">},</span>
              <span class="nx">keysExamined</span><span class="p">:</span> <span class="mi">11</span>
            <span class="p">}</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">},</span>
      <span class="nx">nReturned</span><span class="p">:</span> <span class="nc">Long</span><span class="p">(</span><span class="dl">'</span><span class="s1">10</span><span class="dl">'</span><span class="p">),</span>
<span class="p">...</span>
</code></pre>

</div>



<p>MongoDB uses <a href="https://github.com/mongodb/mongo/blob/fbebc5b7/src/mongo/db/exec/classic/distinct_scan.h#L107" rel="noopener noreferrer">DISTINCT_SCAN</a> in aggregation when the pipeline starts with a <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/sort/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$sort</a> and <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$group</a> using <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/first/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$first</a> or <a href="https://www.mongodb.com/docs/manual/reference/operator/aggregation/last/?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=sort-group-first" rel="noopener noreferrer">$last</a>. The planner checks for a matching index that's not multi-key and has the correct sort order, adjusting the scan direction if needed. If these <a href="https://github.com/mongodb/mongo/blob/fbebc5b7/src/mongo/db/pipeline/pipeline_d.cpp#L1117C1-L1117C27" rel="noopener noreferrer">conditions</a> are met, MongoDB rewrites the pipeline to use <code>DISTINCT_SCAN</code> and <code>$groupByDistinct</code>, optimizing by skipping to relevant index entries and retrieving only needed documents.</p>

<p>This pattern is common in real‑world queries such as:</p>

<ul>
<li>Latestor earliest measure for each metric in a time‑series database</li>
<li>Last contract with each supplier</li>
<li>Last purchase from each client</li>
<li>Most recent transaction for each account</li>
<li>Earliest login event for each user</li>
<li>Lowest‑paid employee in each department</li>
</ul>

