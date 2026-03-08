---
Title: Every Readability Formula Explained (with JavaScript Examples)
Description: 
Author: ckmtools
Date: 2026-03-08T21:45:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>Readability formulas have been around since the 1940s. The U.S. military commissioned the first one to make training manuals less confusing. Newspapers adopted them to keep articles accessible. Today, content teams at organizations like gov.uk and the NYT use them to measure whether their writing actually reaches people.</p>

<p>But if you search for "readability formulas javascript," you get scattered npm packages — each implementing one formula — and blog posts from 2018. No single guide covers all the formulas with working code you can install and run.</p>

<p>This article covers all 8 major readability formulas, explains the math behind each one, and shows how to compute them in JavaScript with <a href="https://www.npmjs.com/package/textlens" rel="noopener noreferrer">textlens</a>.</p>

<h2>
  
  
  Setup
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>textlens
</code></pre>

</div>



<p>textlens is a zero-dependency TypeScript toolkit. It works in Node.js 16+ and ships ESM and CommonJS with full type definitions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">readability</span><span class="p">,</span> <span class="nx">statistics</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">textlens</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// or: import { readability, statistics } from 'textlens';</span>
</code></pre>

</div>



<p>Every example below uses the same sample text:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="s2">`The quick brown fox jumps over the lazy dog.
This sentence contains several common English words.
Reading difficulty depends on word length and sentence structure.
Shorter sentences with familiar words score as more readable.`</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  The 8 Readability Formulas
</h2>

<h3>
  
  
  1. Flesch Reading Ease
</h3>

<p>The most widely known formula. Created by Rudolf Flesch in 1948. Returns a score from 0 to 100 — higher means easier to read.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>score = 206.835 - 1.015 × (words / sentences) - 84.6 × (syllables / words)
</code></pre>

</div>



<p>Two factors drive the score: average sentence length and average syllables per word. Short sentences with short words score high. Academic prose with long sentences and polysyllabic words scores low.</p>

<p><strong>Score interpretation:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Score</th>
<th>Difficulty</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>90-100</td>
<td>Very easy</td>
<td>Children's books</td>
</tr>
<tr>
<td>60-70</td>
<td>Standard</td>
<td>Newspaper articles</td>
</tr>
<tr>
<td>30-50</td>
<td>Difficult</td>
<td>Academic papers</td>
</tr>
<tr>
<td>0-30</td>
<td>Very difficult</td>
<td>Legal documents</td>
</tr>
</tbody>
</table></div>

<p><strong>With textlens:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">scores</span> <span class="o">=</span> <span class="nf">readability</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">fleschReadingEase</span><span class="p">.</span><span class="nx">score</span><span class="p">);</span>
<span class="c1">// 72.4</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">fleschReadingEase</span><span class="p">.</span><span class="nx">interpretation</span><span class="p">);</span>
<span class="c1">// "Fairly easy to read"</span>
</code></pre>

</div>



<h3>
  
  
  2. Flesch-Kincaid Grade Level
</h3>

<p>Same inputs as Flesch Reading Ease, but the coefficients are rearranged to output a U.S. grade level instead of a 0-100 score. Created by J. Peter Kincaid for the U.S. Navy in 1975.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>grade = 0.39 × (words / sentences) + 11.8 × (syllables / words) - 15.59
</code></pre>

</div>



<p>A result of 8.0 means an eighth grader can understand the text. Most successful blog posts land between grades 6 and 9.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">fleschKincaidGrade</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 5.6</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">fleschKincaidGrade</span><span class="p">.</span><span class="nx">interpretation</span><span class="p">);</span>
<span class="c1">// "6th grade"</span>
</code></pre>

</div>



<h3>
  
  
  3. Gunning Fog Index
</h3>

<p>Created by Robert Gunning, a newspaper consultant, in 1952. The Fog Index specifically penalizes "complex words" — words with 3 or more syllables. This makes it stricter than Flesch-Kincaid for jargon-heavy text.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>grade = 0.4 × ((words / sentences) + 100 × (complex_words / words))
</code></pre>

</div>



<p>Where <code>complex_words</code> = words with 3+ syllables (excluding proper nouns, common suffixes, and compound words).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">gunningFogIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 8.1</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">gunningFogIndex</span><span class="p">.</span><span class="nx">interpretation</span><span class="p">);</span>
<span class="c1">// "8th grade"</span>
</code></pre>

</div>



<p>If your Fog Index is above 12, you're writing at a level that most readers will struggle with. Cut the jargon.</p>

<h3>
  
  
  4. Coleman-Liau Index
</h3>

<p>Unlike the previous formulas, Coleman-Liau uses <strong>character count</strong> instead of syllable count. This matters because syllable counting is inherently imprecise (is "fire" one syllable or two?), while character counting is deterministic.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>grade = 0.0588 × L - 0.296 × S - 15.8
</code></pre>

</div>



<p>Where <code>L</code> = average number of letters per 100 words, <code>S</code> = average number of sentences per 100 words.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">colemanLiauIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 7.8</span>
</code></pre>

</div>



<p>Coleman-Liau is a good choice when you need consistent, reproducible scores across different implementations.</p>

<h3>
  
  
  5. SMOG Index
</h3>

<p>"Simple Measure of Gobbledygook." Created by G. Harry McLaughlin in 1969. SMOG focuses specifically on polysyllabic words (3+ syllables) and is considered one of the most accurate grade-level predictors.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>grade = 1.0430 × √(polysyllabic_words × (30 / sentences)) + 3.1291
</code></pre>

</div>



<p>SMOG requires at least 30 sentences for statistical validity. For shorter texts, most implementations (including textlens) use a normalized version.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">smogIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 7.2</span>
</code></pre>

</div>



<h3>
  
  
  6. Automated Readability Index (ARI)
</h3>

<p>Originally designed for real-time readability monitoring on typewriters. Uses characters per word and words per sentence — no syllable counting needed.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>grade = 4.71 × (characters / words) + 0.5 × (words / sentences) - 21.43
</code></pre>

</div>



<p>ARI tends to give slightly higher grade levels than other formulas because it counts characters rather than syllables. A word like "through" (7 characters, 1 syllable) gets penalized more by ARI than by Flesch.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">automatedReadabilityIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 4.8</span>
</code></pre>

</div>



<h3>
  
  
  7. Dale-Chall Readability Score
</h3>

<p>Takes a completely different approach. Instead of measuring word length or syllables, Dale-Chall compares your text against a list of 3,000 words that 80% of fourth graders can understand. Words not on the list are "difficult."</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>score = 0.1579 × (difficult_words / words × 100) + 0.0496 × (words / sentences)
if (difficult_words / words &gt; 0.05) score += 3.6365
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">daleChallScore</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 6.4</span>
</code></pre>

</div>



<p>Dale-Chall is especially useful for content aimed at general audiences. If more than 5% of your words aren't on the familiar-word list, the formula adds a penalty.</p>

<h3>
  
  
  8. Linsear Write Formula
</h3>

<p>Developed for the U.S. Air Force to assess technical manual readability. Classifies words as "easy" (2 or fewer syllables) or "hard" (3+ syllables), then weights them differently.</p>

<p><strong>The math:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>raw = (easy_words × 1 + hard_words × 3) / sentences
grade = raw &gt; 20 ? raw / 2 : (raw - 1) / 2
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">linsearWriteFormula</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="c1">// 5.5</span>
</code></pre>

</div>



<h2>
  
  
  The Consensus Grade: One Number to Rule Them All
</h2>

<p>Each formula has biases. Flesch-Kincaid over-penalizes syllables. Coleman-Liau over-penalizes long words with few syllables. Dale-Chall depends on a static word list from the 1940s.</p>

<p>textlens computes all 8 formulas and averages the 7 grade-level results (excluding Flesch Reading Ease, which uses a different scale) into a single <code>consensusGrade</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">scores</span> <span class="o">=</span> <span class="nf">readability</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>

<span class="c1">// Individual formulas</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">fleschReadingEase</span><span class="p">.</span><span class="nx">score</span><span class="p">);</span>  <span class="c1">// 0-100 scale</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">fleschKincaidGrade</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span> <span class="c1">// US grade level</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">gunningFogIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">colemanLiauIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">smogIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">automatedReadabilityIndex</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">daleChallScore</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">linsearWriteFormula</span><span class="p">.</span><span class="nx">grade</span><span class="p">);</span>

<span class="c1">// Weighted average of all grade-level formulas</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">scores</span><span class="p">.</span><span class="nx">consensusGrade</span><span class="p">);</span> <span class="c1">// 6.5</span>
</code></pre>

</div>



<p>The consensus grade smooths out the biases of individual formulas. One number, backed by seven formulas.</p>

<h2>
  
  
  What Grade Level Should You Target?
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Content type</th>
<th>Target grade</th>
<th>Flesch Ease</th>
</tr>
</thead>
<tbody>
<tr>
<td>Blog posts</td>
<td>6-8</td>
<td>60-70</td>
</tr>
<tr>
<td>Technical docs</td>
<td>8-12</td>
<td>40-60</td>
</tr>
<tr>
<td>API reference</td>
<td>10-14</td>
<td>30-50</td>
</tr>
<tr>
<td>Academic papers</td>
<td>14+</td>
<td>0-30</td>
</tr>
</tbody>
</table></div>

<p>The average American reads at an 8th-grade level. If your consensus grade is above 12, consider:</p>

<ul>
<li>Splitting long sentences</li>
<li>Replacing multisyllabic words with shorter alternatives</li>
<li>Breaking complex ideas into multiple sentences</li>
</ul>

<h2>
  
  
  CLI: Quick Check Without Writing Code
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx textlens article.md
</code></pre>

</div>



<p>For JSON output you can pipe to other tools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx textlens article.md <span class="nt">--json</span> | jq <span class="s1">'.readability.consensusGrade'</span>
</code></pre>

</div>



<p>For a specific formula:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx textlens article.md <span class="nt">--formula</span> flesch-reading-ease
</code></pre>

</div>



<h2>
  
  
  Why One Package Instead of Eight
</h2>

<p>Before textlens, computing all these formulas meant installing separate packages: <code>flesch</code>, <code>flesch-kincaid</code>, <code>gunning-fog</code>, <code>coleman-liau</code>, <code>automated-readability</code>, <code>smog-formula</code>, <code>dale-chall-formula</code>. That's 7 dependencies for 7 formulas, each with different APIs and maintainers.</p>

<p>textlens gives you all 8 formulas in a single, zero-dependency package with a consistent API. One install, one import, one function call.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>textlens
</code></pre>

</div>



<ul>
<li><a href="https://www.npmjs.com/package/textlens" rel="noopener noreferrer">npm: textlens</a></li>
<li><a href="https://github.com/ckmtools/textlens" rel="noopener noreferrer">GitHub: ckmtools/textlens</a></li>
<li><a href="https://ckmtools.dev/textlens/" rel="noopener noreferrer">Docs: ckmtools.dev/textlens</a></li>
</ul>

<p>If you find it useful, a star on GitHub helps others discover it.</p>

<p><em>This is part of the <a href="https://dev.to/ckmtools/series/textlens">textlens series</a> — a set of tutorials on text analysis in JavaScript and TypeScript.</em></p>

