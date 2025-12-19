---
Title: Atlas Search score details (the BM25 calculation)
Description: 
Author: Franck Pachot
Date: 2025-12-19T22:05:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>With <a class="mentioned-user" href="https://dev.to/james_blackwoodsewell_58">@james_blackwoodsewell_58</a> we were comparing the BM25 text search scores between MongoDB Atlas (Lucene), ElasticSearch (Lucene) and ParadeDB (using Tantivy)  which provide the same ordering but MongoDB Atlas shows constantly a lower score by a factor of 2.2: </p>


<div class="crayons-card c-embed text-styles text-styles--secondary">
    <div class="c-embed__content">
        <div class="c-embed__cover">
          <a href="https://www.linkedin.com/pulse/revisiting-text-search-mongodb-bm25-tf-idf-postgresql-james-o4w0e" class="c-link align-middle" rel="noopener noreferrer">
            <img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmedia.licdn.com%2Fdms%2Fimage%2Fv2%2FD4E12AQENaZnC9BmkXA%2Farticle-cover_image-shrink_720_1280%2FB4EZs04UTtKkAI-%2F0%2F1766118738158%3Fe%3D2147483647%26v%3Dbeta%26t%3D4q2D4JKjMzWmYKa2dRd7XoRLZtovPwGhiWG7Qu3slZU" height="719" class="m-0" width="1280">
          </a>
        </div>
      <div class="c-embed__body">
        <h2 class="fs-xl lh-tight">
          <a href="https://www.linkedin.com/pulse/revisiting-text-search-mongodb-bm25-tf-idf-postgresql-james-o4w0e" rel="noopener noreferrer" class="c-link">
            Revisiting "Text Search With MongoDB (BM25 TF-IDF) and PostgreSQL"
          </a>
        </h2>
          <p class="truncate-at-3">
            Back in October Franck Pachot from MongoDB (love your work) wrote a post comparing text search in MongoDB and PostgreSQL (with both the built-in tsvector and ParadeDB's pg_search extension). I'm not going to recap his whole post, but basically Mongo seemed to behave exactly how it should returning B
          </p>
        <div class="color-secondary fs-s flex items-center">
            <img alt="favicon" class="c-embed__favicon m-0 mr-2 radius-0" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fstatic.licdn.com%2Faero-v1%2Fsc%2Fh%2Fal2o9zrvru7aqj8e1x2rzsrca" width="64" height="64">
          linkedin.com
        </div>
      </div>
    </div>
</div>


<p>It was the occasion for me to look at the <a href="https://www.mongodb.com/docs/atlas/atlas-search/score/get-details?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=scoreDetails" rel="noopener noreferrer">score details</a> which gives the calculation details for the score.</p>

<h2>
  
  
  Test case
</h2>

<p>I've built the same test case as in my <a href="https://www.linkedin.com/posts/franckpachot_text-search-with-mongodb-and-postgresql-activity-7374907160146747392-UISF/" rel="noopener noreferrer">previous blog</a>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">db</span><span class="p">.</span><span class="nx">articles</span><span class="p">.</span><span class="nf">drop</span><span class="p">();</span>
<span class="nx">db</span><span class="p">.</span><span class="nx">articles</span><span class="p">.</span><span class="nf">deleteMany</span><span class="p">({});</span>
<span class="nx">db</span><span class="p">.</span><span class="nx">articles</span><span class="p">.</span><span class="nf">insertMany</span><span class="p">([</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍏 🍌 🍊</span><span class="dl">"</span> <span class="p">},</span>                <span class="c1">// short, 1 🍏</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍌 🍊</span><span class="dl">"</span> <span class="p">},</span>                <span class="c1">// short, 1 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍌 🍊 🍎</span><span class="dl">"</span> <span class="p">},</span>             <span class="c1">// larger, 2 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍌 🍊 🍊 🍊</span><span class="dl">"</span> <span class="p">},</span>          <span class="c1">// larger, 1 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍌 🍊 🌴 🫐 🍈 🍇 🌰</span><span class="dl">"</span> <span class="p">},</span>  <span class="c1">// large, 1 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍎 🍎 🍎 🍎 🍎</span><span class="dl">"</span> <span class="p">},</span>       <span class="c1">// large, 6 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍌</span><span class="dl">"</span> <span class="p">},</span>                 <span class="c1">// very short, 1 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍌 🍊 🌴 🫐 🍈 🍇 🌰 🍎</span><span class="dl">"</span> <span class="p">},</span>  <span class="c1">// large, 1 🍎</span>
 <span class="p">{</span> <span class="na">description</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">🍎 🍎 🍌 🍌 🍌</span><span class="dl">"</span> <span class="p">},</span>          <span class="c1">// shorter, 2 🍎</span>
<span class="p">]);</span>
<span class="nx">db</span><span class="p">.</span><span class="nx">articles</span><span class="p">.</span><span class="nf">createSearchIndex</span><span class="p">(</span><span class="dl">"</span><span class="s2">default</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">{</span> <span class="na">mappings</span><span class="p">:</span> <span class="p">{</span> <span class="na">dynamic</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  Score with details
</h2>

<p>I ran the same query, adding <code>scoreDetails: true</code> to the search stage, and <code>scoreDetails: { $meta: "searchScoreDetails" }  }</code> to the projection stage:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>
<span class="nx">db</span><span class="p">.</span><span class="nx">articles</span><span class="p">.</span><span class="nf">aggregate</span><span class="p">([</span>
  <span class="p">{</span>
    <span class="na">$search</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">text</span><span class="p">:</span> <span class="p">{</span>  <span class="na">query</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">🍎</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">🍏</span><span class="dl">"</span><span class="p">],</span>  <span class="na">path</span><span class="p">:</span> <span class="dl">"</span><span class="s2">description</span><span class="dl">"</span>  <span class="p">},</span>
      <span class="na">index</span><span class="p">:</span> <span class="dl">"</span><span class="s2">default</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">scoreDetails</span><span class="p">:</span> <span class="kc">true</span>
    <span class="p">}</span>
  <span class="p">},</span>
  <span class="p">{</span>  <span class="na">$project</span><span class="p">:</span> <span class="p">{</span>  
        <span class="na">_id</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>  <span class="na">description</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>  
        <span class="na">score</span><span class="p">:</span> <span class="p">{</span> <span class="na">$meta</span><span class="p">:</span> <span class="dl">"</span><span class="s2">searchScore</span><span class="dl">"</span> <span class="p">},</span>  
        <span class="na">scoreDetails</span><span class="p">:</span> <span class="p">{</span> <span class="na">$meta</span><span class="p">:</span> <span class="dl">"</span><span class="s2">searchScoreDetails</span><span class="dl">"</span> <span class="p">}</span>  <span class="p">}</span>  <span class="p">},</span>
  <span class="p">{</span> <span class="na">$sort</span><span class="p">:</span> <span class="p">{</span> <span class="na">score</span><span class="p">:</span> <span class="o">-</span><span class="mi">1</span> <span class="p">}</span> <span class="p">}</span>  <span class="p">,</span>
  <span class="p">{</span> <span class="na">$limit</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span>
<span class="p">])</span>
</code></pre>

</div>



<p>Here is the result:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">mdb</span><span class="o">&gt;</span> <span class="nx">db</span><span class="p">.</span><span class="nx">articles</span><span class="p">.</span><span class="nf">aggregate</span><span class="p">([</span>
<span class="p">...</span>   <span class="p">{</span>
<span class="p">...</span>     <span class="nx">$search</span><span class="p">:</span> <span class="p">{</span>
<span class="p">...</span>       <span class="nx">text</span><span class="p">:</span> <span class="p">{</span>  <span class="nl">query</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">🍎</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">🍏</span><span class="dl">"</span><span class="p">],</span>  <span class="nx">path</span><span class="p">:</span> <span class="dl">"</span><span class="s2">description</span><span class="dl">"</span>  <span class="p">},</span>
<span class="p">...</span>       <span class="nx">index</span><span class="p">:</span> <span class="dl">"</span><span class="s2">default</span><span class="dl">"</span><span class="p">,</span>
<span class="p">...</span>       <span class="nx">scoreDetails</span><span class="p">:</span> <span class="kc">true</span>
<span class="p">...</span>     <span class="p">}</span>
<span class="p">...</span>   <span class="p">},</span>
<span class="p">...</span>   <span class="p">{</span>  <span class="nl">$project</span><span class="p">:</span> <span class="p">{</span>  <span class="na">_id</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>  <span class="na">description</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>  <span class="na">score</span><span class="p">:</span> <span class="p">{</span> <span class="na">$meta</span><span class="p">:</span> <span class="dl">"</span><span class="s2">searchScore</span><span class="dl">"</span> <span class="p">},</span>  <span class="na">scoreDetails</span><span class="p">:</span> <span class="p">{</span> <span class="na">$meta</span><span class="p">:</span> <span class="dl">"</span><span class="s2">searchScoreDetails</span><span class="dl">"</span> <span class="p">}</span>  <span class="p">}</span>  <span class="p">},</span>
<span class="p">...</span>   <span class="p">{</span> <span class="nl">$sort</span><span class="p">:</span> <span class="p">{</span> <span class="na">score</span><span class="p">:</span> <span class="o">-</span><span class="mi">1</span> <span class="p">}</span> <span class="p">}</span>  <span class="p">,</span>
<span class="p">...</span>   <span class="p">{</span> <span class="nl">$limit</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span>
<span class="p">...</span> <span class="p">])</span>
<span class="p">[</span>
  <span class="p">{</span>
    <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">🍏 🍌 🍊</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">score</span><span class="p">:</span> <span class="mf">1.0242118835449219</span><span class="p">,</span>
    <span class="na">scoreDetails</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">value</span><span class="p">:</span> <span class="mf">1.0242118835449219</span><span class="p">,</span>
      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">sum of:</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">details</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">value</span><span class="p">:</span> <span class="mf">1.0242118835449219</span><span class="p">,</span>
          <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">$type:string/description:🍏 [BM25Similarity], result of:</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">details</span><span class="p">:</span> <span class="p">[</span>
            <span class="p">{</span>
              <span class="na">value</span><span class="p">:</span> <span class="mf">1.0242118835449219</span><span class="p">,</span>
              <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">score(freq=1.0), computed as boost * idf * tf from:</span><span class="dl">'</span><span class="p">,</span>
              <span class="na">details</span><span class="p">:</span> <span class="p">[</span>
                <span class="p">{</span>
                  <span class="na">value</span><span class="p">:</span> <span class="mf">1.8971199989318848</span><span class="p">,</span>
                  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:</span><span class="dl">'</span><span class="p">,</span>
                  <span class="na">details</span><span class="p">:</span> <span class="p">[</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">n, number of documents containing term</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">},</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mi">9</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">N, total number of documents with field</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">}</span>
                  <span class="p">]</span>
                <span class="p">},</span>
                <span class="p">{</span>
                  <span class="na">value</span><span class="p">:</span> <span class="mf">0.5398772954940796</span><span class="p">,</span>
                  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:</span><span class="dl">'</span><span class="p">,</span>
                  <span class="na">details</span><span class="p">:</span> <span class="p">[</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">freq, occurrences of term within document</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">},</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mf">1.2000000476837158</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">k1, term saturation parameter</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">},</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mf">0.75</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">b, length normalization parameter</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">},</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">dl, length of field</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">},</span>
                    <span class="p">{</span>
                      <span class="na">value</span><span class="p">:</span> <span class="mf">4.888888835906982</span><span class="p">,</span>
                      <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">avgdl, average length of field</span><span class="dl">'</span><span class="p">,</span>
                      <span class="na">details</span><span class="p">:</span> <span class="p">[]</span>
                    <span class="p">}</span>
                  <span class="p">]</span>
                <span class="p">}</span>
              <span class="p">]</span>
            <span class="p">}</span>
          <span class="p">]</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">]</span>
</code></pre>

</div>



<p>So all is there. Here is the scoring breakdown for "🍏 🍌 🍊", which produced a score of <code>1.0242118835449219</code>.</p>

<h2>
  
  
  IDF calculation (inverse document frequency)
</h2>

<p>Search result:</p>

<ul>
<li>Number of documents containing the term: <code>n = 1</code>
</li>
<li>Total number of documents with this field: <code>N = 9</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="n">idf</span> <span class="o">=</span> <span class="n">log</span><span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="p">(</span><span class="n">N</span> <span class="o">-</span> <span class="n">n</span> <span class="o">+</span> <span class="mi">0</span><span class="p">.</span><span class="mi">5</span><span class="p">)</span> <span class="o">/</span> <span class="p">(</span><span class="n">n</span> <span class="o">+</span> <span class="mi">0</span><span class="p">.</span><span class="mi">5</span><span class="p">))</span>
    <span class="o">=</span> <span class="n">log</span><span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="p">(</span><span class="mi">9</span> <span class="o">-</span> <span class="mi">1</span> <span class="o">+</span> <span class="mi">0</span><span class="p">.</span><span class="mi">5</span><span class="p">)</span> <span class="o">/</span> <span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="mi">0</span><span class="p">.</span><span class="mi">5</span><span class="p">))</span>
    <span class="o">=</span> <span class="n">log</span><span class="p">(</span><span class="mi">6</span><span class="p">.</span><span class="mi">666666666666667</span><span class="p">)</span><span class="err">`</span>  
    <span class="err">≈</span> <span class="mi">1</span><span class="p">.</span><span class="mi">8971199989318848</span>
</code></pre>

</div>



<h2>
  
  
  TF calculation (term frequency)
</h2>

<p>Parameters are the Lucene defaults:</p>

<ul>
<li>Term saturation parameter: <code>k1 = 1.2000000476837158</code>
</li>
<li>Length normalization parameter: <code>b = 0.75</code>
</li>
</ul>

<p>Document field statistics:</p>

<ul>
<li>Average length of the field: <code>avgdl = 44 / 9 ≈ 4.888888835906982</code>
</li>
<li>Occurrences of the term in this document: <code>freq = 1</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="n">tf</span> <span class="o">=</span> <span class="n">freq</span> <span class="o">/</span> <span class="p">(</span><span class="n">freq</span> <span class="o">+</span> <span class="n">k1</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="n">b</span> <span class="o">+</span> <span class="n">b</span> <span class="o">*</span> <span class="n">dl</span> <span class="o">/</span> <span class="n">avgdl</span><span class="p">))</span> 
   <span class="o">=</span> <span class="mi">1</span> <span class="o">/</span> <span class="p">(</span><span class="mi">1</span> <span class="o">+</span> <span class="mi">1</span><span class="p">.</span><span class="mi">2000000476837158</span> <span class="err">×</span> <span class="p">(</span><span class="mi">0</span><span class="p">.</span><span class="mi">25</span> <span class="o">+</span> <span class="mi">0</span><span class="p">.</span><span class="mi">75</span> <span class="err">×</span> <span class="p">(</span><span class="mi">3</span> <span class="o">/</span> <span class="mi">4</span><span class="p">.</span><span class="mi">888888835906982</span><span class="p">)))</span> 
   <span class="err">≈</span> <span class="mi">0</span><span class="p">.</span><span class="mi">5398772954940796</span>
</code></pre>

</div>



<h2>
  
  
  Final score
</h2>

<p>Parameter: </p>

<ul>
<li>Boost: <code>1.0</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="n">score</span> <span class="o">=</span> <span class="n">boost</span> <span class="err">×</span> <span class="n">idf</span> <span class="err">×</span> <span class="n">tf</span> 
      <span class="o">=</span> <span class="mi">1</span><span class="p">.</span><span class="mi">0</span> <span class="err">×</span> <span class="mi">1</span><span class="p">.</span><span class="mi">8971199989318848</span> <span class="err">×</span> <span class="mi">0</span><span class="p">.</span><span class="mi">5398772954940796</span> 
      <span class="err">≈</span> <span class="mi">1</span><span class="p">.</span><span class="mo">024211</span><span class="mi">8835449219</span>
</code></pre>

</div>



<p>That confirms that Atlas Search uses the same scoring as Lucene <a href="https://github.com/apache/lucene/blob/releases/lucene/10.3.2/lucene/core/src/java/org/apache/lucene/search/similarities/BM25Similarity.java#L183" rel="noopener noreferrer">https://github.com/apache/lucene/blob/releases/lucene/10.3.2/lucene/core/src/java/org/apache/lucene/search/similarities/BM25Similarity.java#L183</a></p>

<h2>
  
  
  What about ElasticSearch and Tantivy
</h2>

<p>Eight years ago, Lucene removed the <code>(k1 + 1)</code> factor in <a href="https://issues.apache.org/jira/browse/LUCENE-8563" rel="noopener noreferrer">LUCENE-8563</a>. For <code>k1 = 1.2</code>, this change reduces the score by a factor of 2.2 from that version onward. Tantivy and Elasticsearch apparently still use the old formula, while Atlas Search uses the updated one, which explains the observed differences in scoring.</p>

<h2>
  
  
  Conclusion
</h2>

<p>MongoDB Atlas Search indexes use the same scoring mechanism as Lucene indexes. When you compare them with other text search engines that also use Lucene, you might notice a score difference by a factor of about 2.2. This doesn’t actually matter, because scores are used only for ordering results, and the relative ordering remains the same.</p>

<p>Text search scores can seem magical, but they are fully deterministic and based on open-source formulas. In MongoDB, you can include the <a href="https://www.mongodb.com/docs/atlas/atlas-search/score/get-details?utm_campaign=devrel&amp;utm_source=third-party-content&amp;utm_term=franck_pachot&amp;utm_medium=devto&amp;utm_content=scoreDetails" rel="noopener noreferrer">score details</a> option in a text search query to inspect all the parameters and formulas behind the score.</p>

