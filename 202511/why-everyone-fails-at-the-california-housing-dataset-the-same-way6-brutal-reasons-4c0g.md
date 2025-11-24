---
Title: Why everyone fails at the California Housing dataset the same way(6 brutal reasons)
Description: 
Author: MohammadReza Mahdian
Date: 2025-11-24T21:41:21.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Why You’ll Never Build a Strong Model with the 9 Raw Features of the California Housing Dataset (6 Key Insights)
</h1>

<p>A 5-minute analysis of the California housing dataset</p>

<p>A while back, I reopened the famous California housing dataset and looked at it from scratch — no solutions, no notebooks, just the raw data and my own eyes.<br>
These are the 9 features we all know:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>'longitude',
'latitude',
'housing_median_age',
'total_rooms',
'total_bedrooms',
'population',
'households',
'median_income',
'ocean_proximity'
</code></pre>

</div>



<p>Here are the six things that actually matter (and that most people miss):</p>

<ol>
<li><p>Latitude and longitude alone tell you almost nothing. Any single coordinate has a massive spread in prices — you can’t rely on them as-is.</p></li>
<li><p>Median house age has the same issue: huge variance at every age level. By itself, it barely helps the model.</p></li>
<li><p>Total rooms and bedrooms obviously correlate with luxury, but they’re heavily correlated with each other. What actually matters is their ratio to each other and especially to the number of households.</p></li>
<li><p>Population and households follow very similar patterns — except in high-price areas where density drops (because wealthy households are fewer), in middle-class urban zones both rise together, and in very crowded areas, prices fall again despite high density.</p></li>
<li><p>Median income is extremely important, but if you don’t combine it properly with population or households, it can mislead you badly. A small block of very high earners can have higher total income than a much larger middle-income area — causing the model to overpredict prices.</p></li>
<li><p>Distance from the ocean (or ocean proximity) alone creates another huge circle of varying prices. It’s not sufficient on its own.</p></li>
</ol>

<p>if you just throw these 9 raw features into a model, you won’t get anything impressive.</p>

<p>But if you actually combine them intelligently and create a few meaningful new features using only these same columns, your model’s performance jumps significantly — almost instantly.</p>

<p>If you’d like, let me know in the comments and I’ll build and share a set of powerful new features derived purely from these 9 original ones.</p>

<p>If there’s enough interest, I’ll post the full version next week.</p>

<p>follow me:<br>
<a href="https://github.com/PyRz-Tech" rel="noopener noreferrer">GitHub</a></p>

