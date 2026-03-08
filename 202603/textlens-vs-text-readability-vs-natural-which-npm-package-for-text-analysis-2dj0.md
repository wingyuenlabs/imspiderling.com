---
Title: textlens vs text-readability vs natural: Which npm Package for Text Analysis?
Description: 
Author: ckmtools
Date: 2026-03-08T22:00:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you need text analysis in a Node.js project, npm gives you dozens of options. Three packages cover the most common use cases: <strong>textlens</strong> (readability + sentiment + keywords), <strong>text-readability</strong> (readability formulas only), and <strong>natural</strong> (full NLP toolkit). Each takes a different approach, and the right choice depends on what you're building.</p>

<p>This is an honest comparison with benchmarks, feature coverage, and bundle sizes. I built textlens, so I'm biased — but I'll point out where the other packages are better choices.</p>

<h2>
  
  
  Feature Comparison
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>textlens</th>
<th>text-readability</th>
<th>natural</th>
</tr>
</thead>
<tbody>
<tr>
<td>Readability formulas</td>
<td>8 (all major)</td>
<td>7</td>
<td>0</td>
</tr>
<tr>
<td>Consensus grade</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Sentiment analysis</td>
<td>AFINN-165</td>
<td>No</td>
<td>Bayes + AFINN</td>
</tr>
<tr>
<td>Keyword extraction</td>
<td>TF-IDF</td>
<td>No</td>
<td>TF-IDF</td>
</tr>
<tr>
<td>Keyword density (n-grams)</td>
<td>Unigrams, bigrams, trigrams</td>
<td>No</td>
<td>N-grams</td>
</tr>
<tr>
<td>SEO scoring</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Text summarization</td>
<td>Extractive</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Reading time</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Tokenization</td>
<td>Basic (English)</td>
<td>Basic</td>
<td>Advanced (multilingual)</td>
</tr>
<tr>
<td>Stemming</td>
<td>No</td>
<td>No</td>
<td>Porter, Lancaster, etc.</td>
</tr>
<tr>
<td>Classification</td>
<td>No</td>
<td>No</td>
<td>Naive Bayes, logistic regression</td>
</tr>
<tr>
<td>Phonetics</td>
<td>No</td>
<td>No</td>
<td>Soundex, Metaphone, etc.</td>
</tr>
<tr>
<td>Dependencies</td>
<td>0</td>
<td>0</td>
<td>1 (webworker-threads, optional)</td>
</tr>
<tr>
<td>TypeScript</td>
<td>Native</td>
<td>No (DefinitelyTyped)</td>
<td>No (DefinitelyTyped)</td>
</tr>
<tr>
<td>CLI</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Bundle size (min)</td>
<td>~45KB</td>
<td>~8KB</td>
<td>~2MB+</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  When to Use Each Package
</h2>

<h3>
  
  
  Use textlens when:
</h3>

<p>You need <strong>content analysis</strong> — readability scoring, sentiment detection, keyword extraction, SEO scoring — and you want it all from one package with one API.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">analyze</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">textlens</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">analyze</span><span class="p">(</span><span class="nx">articleText</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">readability</span><span class="p">.</span><span class="nx">consensusGrade</span><span class="p">);</span>  <span class="c1">// 7.2</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">sentiment</span><span class="p">.</span><span class="nx">label</span><span class="p">);</span>              <span class="c1">// 'positive'</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">keywords</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">word</span><span class="p">);</span>             <span class="c1">// 'javascript'</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">readingTime</span><span class="p">.</span><span class="nx">minutes</span><span class="p">);</span>           <span class="c1">// 4</span>
</code></pre>

</div>



<p>Best for: blog post quality checks, content pipelines, documentation linting, SEO tools, content management systems.</p>

<h3>
  
  
  Use text-readability when:
</h3>

<p>You <strong>only</strong> need readability scores and nothing else. It's smaller (~8KB vs ~45KB) and does one thing.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">rs</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">text-readability</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">grade</span> <span class="o">=</span> <span class="nx">rs</span><span class="p">.</span><span class="nf">fleschKincaidGrade</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">ease</span> <span class="o">=</span> <span class="nx">rs</span><span class="p">.</span><span class="nf">fleschReadingEase</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
</code></pre>

</div>



<p>Best for: simple readability gates where bundle size matters and you don't need sentiment, keywords, or other analysis.</p>

<p><strong>Trade-off:</strong> No consensus grade (you pick one formula), no TypeScript types shipped with the package, no CLI.</p>

<h3>
  
  
  Use natural when:
</h3>

<p>You need <strong>NLP capabilities</strong> beyond text analysis — tokenization with stemmers, Naive Bayes classification, phonetic matching, string distance algorithms, or multilingual support.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">natural</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">natural</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">tokenizer</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">natural</span><span class="p">.</span><span class="nc">WordTokenizer</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">classifier</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">natural</span><span class="p">.</span><span class="nc">BayesClassifier</span><span class="p">();</span>
<span class="nx">classifier</span><span class="p">.</span><span class="nf">addDocument</span><span class="p">(</span><span class="dl">'</span><span class="s1">great product</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">positive</span><span class="dl">'</span><span class="p">);</span>
<span class="nx">classifier</span><span class="p">.</span><span class="nf">addDocument</span><span class="p">(</span><span class="dl">'</span><span class="s1">terrible service</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">negative</span><span class="dl">'</span><span class="p">);</span>
<span class="nx">classifier</span><span class="p">.</span><span class="nf">train</span><span class="p">();</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">classifier</span><span class="p">.</span><span class="nf">classify</span><span class="p">(</span><span class="dl">'</span><span class="s1">good experience</span><span class="dl">'</span><span class="p">));</span>
</code></pre>

</div>



<p>Best for: chatbots, search engines, language processing pipelines, text classification, spell checking.</p>

<p><strong>Trade-off:</strong> Large bundle (~2MB+), no readability formulas at all, no built-in content scoring. It's a general NLP library, not a content analysis tool.</p>

<h2>
  
  
  Readability Formula Coverage
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Formula</th>
<th>textlens</th>
<th>text-readability</th>
<th>natural</th>
</tr>
</thead>
<tbody>
<tr>
<td>Flesch Reading Ease</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Flesch-Kincaid Grade</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Gunning Fog Index</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Coleman-Liau Index</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>SMOG Index</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Automated Readability Index</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Dale-Chall</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Linsear Write</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr>
<td>Consensus Grade</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
</tbody>
</table></div>

<p>textlens adds the Linsear Write formula and a consensus grade that averages all grade-level formulas into a single number.</p>

<h2>
  
  
  Sentiment Analysis Comparison
</h2>

<p>Both textlens and natural offer sentiment analysis, but with different approaches.</p>

<p><strong>textlens</strong> uses the AFINN-165 lexicon (~3,300 words with sentiment scores from -5 to +5). It returns a normalized score, a label, a confidence value, and lists of matched positive/negative words:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">sentiment</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">textlens</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">sentiment</span><span class="p">(</span><span class="dl">'</span><span class="s1">The product is great but the support is terrible.</span><span class="dl">'</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">label</span><span class="p">);</span>      <span class="c1">// 'neutral' (mixed)</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">positive</span><span class="p">);</span>   <span class="c1">// ['great']</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">negative</span><span class="p">);</span>   <span class="c1">// ['terrible']</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">confidence</span><span class="p">);</span> <span class="c1">// 0.18</span>
</code></pre>

</div>



<p><strong>natural</strong> offers both AFINN-based and Naive Bayes classification. The Bayes approach is trainable — you can feed it your own labeled data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">natural</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">natural</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">Analyzer</span> <span class="o">=</span> <span class="nx">natural</span><span class="p">.</span><span class="nx">SentimentAnalyzer</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">stemmer</span> <span class="o">=</span> <span class="nx">natural</span><span class="p">.</span><span class="nx">PorterStemmer</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">analyzer</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Analyzer</span><span class="p">(</span><span class="dl">'</span><span class="s1">English</span><span class="dl">'</span><span class="p">,</span> <span class="nx">stemmer</span><span class="p">,</span> <span class="dl">'</span><span class="s1">afinn</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">analyzer</span><span class="p">.</span><span class="nf">getSentiment</span><span class="p">([</span><span class="dl">'</span><span class="s1">great</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">but</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">terrible</span><span class="dl">'</span><span class="p">]);</span>
<span class="c1">// Returns a single number</span>
</code></pre>

</div>



<p><strong>Bottom line:</strong> textlens gives more structured output (label, confidence, word lists) out of the box. natural gives more flexibility if you want trainable classifiers.</p>

<h2>
  
  
  Keyword Extraction
</h2>

<p>Both textlens and natural support TF-IDF keyword extraction.</p>

<p><strong>textlens:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">keywords</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">textlens</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">kw</span> <span class="o">=</span> <span class="nf">keywords</span><span class="p">(</span><span class="nx">articleText</span><span class="p">,</span> <span class="p">{</span> <span class="na">topN</span><span class="p">:</span> <span class="mi">5</span> <span class="p">});</span>
<span class="c1">// [{ word: 'javascript', score: 4.2, count: 8, density: 2.1 }, ...]</span>
</code></pre>

</div>



<p><strong>natural:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">natural</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">natural</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">TfIdf</span> <span class="o">=</span> <span class="nx">natural</span><span class="p">.</span><span class="nx">TfIdf</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">tfidf</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TfIdf</span><span class="p">();</span>

<span class="nx">tfidf</span><span class="p">.</span><span class="nf">addDocument</span><span class="p">(</span><span class="nx">articleText</span><span class="p">);</span>
<span class="nx">tfidf</span><span class="p">.</span><span class="nf">listTerms</span><span class="p">(</span><span class="mi">0</span><span class="p">).</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>
</code></pre>

</div>



<p>textlens adds keyword density percentages and n-gram analysis (bigrams, trigrams) via the separate <code>density()</code> function. natural requires more setup but supports multi-document TF-IDF natively.</p>

<h2>
  
  
  Bundle Size and Dependencies
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Package</th>
<th>Install size</th>
<th>Dependencies</th>
<th>Min bundle</th>
</tr>
</thead>
<tbody>
<tr>
<td>textlens</td>
<td>~180KB</td>
<td>0</td>
<td>~45KB</td>
</tr>
<tr>
<td>text-readability</td>
<td>~45KB</td>
<td>0</td>
<td>~8KB</td>
</tr>
<tr>
<td>natural</td>
<td>~10MB+</td>
<td>Several</td>
<td>~2MB+</td>
</tr>
</tbody>
</table></div>

<p>If you're building a serverless function or a browser-side tool, size matters. text-readability is the smallest. textlens is a reasonable middle ground. natural is heavy.</p>

<h2>
  
  
  TypeScript Support
</h2>

<p>textlens ships native TypeScript types — no <code>@types</code> package needed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">readability</span><span class="p">,</span> <span class="nx">ReadabilityResult</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">textlens</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">result</span><span class="p">:</span> <span class="nx">ReadabilityResult</span> <span class="o">=</span> <span class="nf">readability</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
</code></pre>

</div>



<p>text-readability and natural have community-maintained types via DefinitelyTyped (<code>@types/text-readability</code>, <code>@types/natural</code>), which can lag behind releases.</p>

<h2>
  
  
  The Overlap Problem
</h2>

<p>Before textlens, building a content analysis pipeline meant installing multiple packages:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Old approach</span>
npm <span class="nb">install </span>text-readability  <span class="c"># readability scores</span>
npm <span class="nb">install </span>sentiment          <span class="c"># sentiment analysis</span>
npm <span class="nb">install </span>keyword-extractor  <span class="c"># keyword extraction</span>
npm <span class="nb">install </span>reading-time       <span class="c"># reading time</span>
</code></pre>

</div>



<p>Four packages, four APIs, four sets of documentation. textlens consolidates these into one package:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>textlens
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">analyze</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">textlens</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">analyze</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
<span class="c1">// result.readability, result.sentiment, result.keywords, result.readingTime</span>
</code></pre>

</div>



<h2>
  
  
  Which Should You Choose?
</h2>

<p><strong>Choose textlens if</strong> you want readability + sentiment + keywords + SEO scoring from a single zero-dependency package. It covers the "content analysis" use case end to end.</p>

<p><strong>Choose text-readability if</strong> you only need readability scores and want the smallest possible bundle.</p>

<p><strong>Choose natural if</strong> you need NLP capabilities like classification, stemming, phonetics, or multilingual tokenization. It's a different category of tool.</p>

<p>There's no wrong answer — it depends on what you're building. All three are MIT-licensed and actively maintained.</p>

<h2>
  
  
  Links
</h2>

<ul>
<li>
<a href="https://www.npmjs.com/package/textlens" rel="noopener noreferrer">textlens on npm</a> | <a href="https://github.com/ckmtools/textlens" rel="noopener noreferrer">GitHub</a>
</li>
<li><a href="https://www.npmjs.com/package/text-readability" rel="noopener noreferrer">text-readability on npm</a></li>
<li><a href="https://www.npmjs.com/package/natural" rel="noopener noreferrer">natural on npm</a></li>
</ul>

<p><strong>Disclosure:</strong> I built textlens. This comparison reflects my honest assessment, but read the other packages' docs and decide for yourself.</p>

<p><em>This is part of the <a href="https://dev.to/ckmtools/series/textlens">textlens series</a> — tutorials on text analysis in JavaScript and TypeScript.</em></p>

