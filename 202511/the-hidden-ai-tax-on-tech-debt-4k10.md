---
Title: The Hidden AI Tax on Tech Debt
Description: 
Author: Volodymyr Yepishev
Date: 2025-11-17T21:44:28.000Z
Robots: noindex,nofollow
Template: index
---
<p>TL/DR: the larger your files are, the more you will likely pay for tokens using AI</p>

<p>Sometimes files bloat: someone got lazy and did not separate concerns, someone was rushed to release the feature, someone silently quit or could care less. It is impossible to catalogue all the possible reasons. There are different views on the file sizes in the codebase, especially when it comes to frontend development, with some developers considering fine grained small files with single export, as was the case with one of the Angular's older style guides, or loose suggestions as it was once in React: see what works for you, you can start by putting everything in one file, or something like that. Both are valid approaches and usually developers go by their preferences.</p>

<p>Yet, it is the end of 2025 and everyone and their mother are prompt egineers shipping features daily, while it is often that the velocity of the development is paid for by the quality of the end product and the state of the codebase itself. This often produces bloated files, which is no wonder, but if we are using AI to develop, is the file bloat actually a problem? Certainly it used to be a problem in the old days for the clean code purists, but what of now?</p>

<p>This has lead me to wonder if AI struggles with files as they grow larger. Intuitively it has to, I even asked some AIs if increased file sizes would be more costly to maintain using AI coding agents, the answer I got back was that I was absolutely right (like always when asking AI anything). I needed a way to test it. I found my old repo with a practice project in React, where I had followed one export per file rule and kept everything separated to an extreme: 89 files in total, 1450 lines, with the largest file counting as few as 136 lines. Copying the repo and merging all files into more bulky ones by module I produced a project with the same code, same functionality, but higher code density, down to 17 files, with the largest file sitting with 448 lines. Another copy would take it to the extreme: all the functional code in a single of 1204 lines. I wish I had something larger to test on, but these would have to suffice.</p>

<p>For testing I further generated 80 prompts in 4 different categories: <code>Code Reading &amp; Display</code>, <code>Code Analysis</code>, <code>Information Queries</code> and <code>Code Generation</code> (very primitive, i.e. add comment or add a field to an interface). The methodology for research was rather simple:</p>

<ul>
<li>open folder with small/large/humongous file(s) with Cursor/Grok;</li>
<li>submit prompts one by one, each in new chat window at regular interval;</li>
<li>
<del>waste all money on tokens</del> compare token usage/price;</li>
<li>repeat 5 times.</li>
</ul>

<p>Submitting prompts at regular intervals in that quantity was an impossible endeavor for me, so a <code>Python</code> script was crafted to take care of it and log results with time. A prompt every 2 minutes, 3 repos, 5 runs per each repo to collect the data. It would take a while. Also, every folder would need to be copied from a pristine source into a new path, to prevent the agent from re-using previous knowledge of it.</p>

<p>The initial displayed some interesting results:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>363,431.00</td>
<td>5,651,776.00</td>
<td>48,670.00</td>
<td>6,063,877.00</td>
<td>0.308</td>
</tr>
<tr>
<td>large</td>
<td>641,014.00</td>
<td>4,732,480.00</td>
<td>42,424.00</td>
<td>5,415,918.00</td>
<td>0.338</td>
</tr>
<tr>
<td>humongous</td>
<td>817,980.00</td>
<td>4,771,392.00</td>
<td>44,903.00</td>
<td>5,634,275.00</td>
<td>0.424</td>
</tr>
</tbody>
</table></div>

<p>The repos with smaller files had better cache read numbers and cheaper use as the result. The next run produced similar results, though not identical (probably because the non-deterministic nature of AI, temperature and etc):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>478,906.00</td>
<td>6,445,127.00</td>
<td>49,960.00</td>
<td>6,973,993.00</td>
<td>0.364</td>
</tr>
<tr>
<td>large</td>
<td>700,265.00</td>
<td>4,968,008.00</td>
<td>44,018.00</td>
<td>5,712,291.00</td>
<td>0.372</td>
</tr>
<tr>
<td>humongous</td>
<td>1,000,671.00</td>
<td>4,846,604.00</td>
<td>45,552.00</td>
<td>5,892,827.00</td>
<td>0.482</td>
</tr>
</tbody>
</table></div>

<p>The humongoes file was getting beaten by its smaller counterparts, it was obviously more expensive to maintain, even though it offered exactly same functionality and same code. It was then that I decided the margin could be even more dramatic if the file size increases even more, though I did not have any code to bloat it with, so I merely polluted it with useless comments to ridiculous <code>3615</code> lines, which is almost 3 times larger that its original volume.</p>

<p>The results of the next run were absolutely buffling to me: although small and large files were still better than the single humongous, the bloated version which had almost 3 times the volume of the original, did outperform it!</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>337,567.00</td>
<td>5,612,288.00</td>
<td>48,050.00</td>
<td>5,997,905.00</td>
<td>0.312</td>
</tr>
<tr>
<td>large</td>
<td>606,161.00</td>
<td>4,459,264.00</td>
<td>40,769.00</td>
<td>5,106,194.00</td>
<td>0.318</td>
</tr>
<tr>
<td>humongous</td>
<td>1,024,393.00</td>
<td>5,148,457.00</td>
<td>45,304.00</td>
<td>6,218,154.00</td>
<td>0.448</td>
</tr>
<tr>
<td>humongous-bloated</td>
<td>336,670.00</td>
<td>6,513,728.00</td>
<td>53,460.00</td>
<td>6,903,858.00</td>
<td>0.337</td>
</tr>
</tbody>
</table></div>

<p>I had no reasonable explanation why this happened. My expectation had been the bloated file would tank, yet with cache reads it outperformed every other repo. Without any idea how LLMs treat files in codebases under the hood, I started thinking that after certain size it is somehow split and treated as multiple.</p>

<p>ChatGPT, being LLM, had a different explanation:</p>

<blockquote>
<p><em>The heavily bloated file performed unexpectedly well because most of its added content consisted of repetitive comments, which produced extremely stable and highly cacheable token prefixes—allowing the model to reuse far more of the long-context cache than any of the smaller, more diverse codebases.</em></p>
</blockquote>

<p>I had no expertise to confirm whether it is true or not, so I continued testing. The next run actually revealed the bloated file to be the best:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>415,485.00</td>
<td>6,425,600.00</td>
<td>49,693.00</td>
<td>6,890,778.00</td>
<td>0.328</td>
</tr>
<tr>
<td>large</td>
<td>626,846.00</td>
<td>4,460,160.00</td>
<td>39,830.00</td>
<td>5,126,836.00</td>
<td>0.331</td>
</tr>
<tr>
<td>humongous</td>
<td>891,457.00</td>
<td>5,117,184.00</td>
<td>43,634.00</td>
<td>6,052,275.00</td>
<td>0.427</td>
</tr>
<tr>
<td>humongous-bloated</td>
<td>348,328.00</td>
<td>6,393,536.00</td>
<td>52,378.00</td>
<td>6,794,242.00</td>
<td>0.320</td>
</tr>
</tbody>
</table></div>

<p>Inspecting the prompts, code generation was only one of the categories, 20 prompts out of 80, the other three being for reading. It was the time to calculate the average per category based on 5 runs for small + large + humongous and 3 runs form the humongous-bloated.</p>

<p>Here are the results</p>

<h2>
  
  
  CODE READING &amp; DISPLAY (PROMPTS 1-20)
</h2>

<p>The bloated file basically gives same speed as the repo with a bunch of small ones.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>71,709.40</td>
<td>646,280.20</td>
<td>4,261.00</td>
<td>722,250.60</td>
<td>0.034</td>
</tr>
<tr>
<td>large</td>
<td>135,992.80</td>
<td>796,562.80</td>
<td>4,647.00</td>
<td>937,202.60</td>
<td>0.051</td>
</tr>
<tr>
<td>humongous</td>
<td>221,114.60</td>
<td>783,213.80</td>
<td>4,396.20</td>
<td>1,008,724.60</td>
<td>0.070</td>
</tr>
<tr>
<td>humongous-bloated</td>
<td>55,467.33</td>
<td>769,216.00</td>
<td>4,856.67</td>
<td>829,540.00</td>
<td>0.033</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  CODE ANALYSIS (PROMPTS 21-40)
</h2>

<p>Same amazing speed for the bloated monster, on par with larger files on the average.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>120,173.40</td>
<td>2,113,496.40</td>
<td>16,864.40</td>
<td>2,250,534.20</td>
<td>0.118</td>
</tr>
<tr>
<td>large</td>
<td>188,936.40</td>
<td>1,685,606.40</td>
<td>13,997.20</td>
<td>1,888,540.00</td>
<td>0.116</td>
</tr>
<tr>
<td>humongous</td>
<td>223,842.20</td>
<td>1,580,842.40</td>
<td>14,566.80</td>
<td>1,819,251.40</td>
<td>0.129</td>
</tr>
<tr>
<td>humongous-bloated</td>
<td>121,723.00</td>
<td>2,063,594.67</td>
<td>17,915.67</td>
<td>2,203,233.33</td>
<td>0.116</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  INFORMATION QUERIES (PROMPTS 41-60)
</h2>

<p>Here the bloated file takes a dive and small ones prevail.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>115,328.40</td>
<td>1,907,962.40</td>
<td>13,827.80</td>
<td>2,037,118.60</td>
<td>0.097</td>
</tr>
<tr>
<td>large</td>
<td>169,173.80</td>
<td>1,357,708.80</td>
<td>11,282.60</td>
<td>1,538,165.20</td>
<td>0.099</td>
</tr>
<tr>
<td>humongous</td>
<td>273,745.80</td>
<td>1,562,889.80</td>
<td>11,829.80</td>
<td>1,848,465.40</td>
<td>0.131</td>
</tr>
<tr>
<td>humongous-bloated</td>
<td>103,394.00</td>
<td>2,133,376.00</td>
<td>14,525.00</td>
<td>2,251,295.00</td>
<td>0.107</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Code Generation (Prompts 61-80)
</h2>

<p>Code generation the bigger files lose, though the large files outperform the super small ones.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Repo</th>
<th>Input (w/o Cache Write)</th>
<th>Cache Read</th>
<th>Output Tokens</th>
<th>Total Tokens</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>small</td>
<td>88,252.20</td>
<td>1,365,952.00</td>
<td>14,204.80</td>
<td>1,468,409.00</td>
<td>0.078</td>
</tr>
<tr>
<td>large</td>
<td>151,602.60</td>
<td>795,801.60</td>
<td>11,740.40</td>
<td>959,144.60</td>
<td>0.071</td>
</tr>
<tr>
<td>humongous</td>
<td>218,112.60</td>
<td>1,050,280.60</td>
<td>14,053.40</td>
<td>1,282,446.60</td>
<td>0.109</td>
</tr>
<tr>
<td>humongous-bloated</td>
<td>87,987.67</td>
<td>1,589,269.33</td>
<td>16,841.33</td>
<td>1,694,098.33</td>
<td>0.087</td>
</tr>
</tbody>
</table></div>

<p>Yet the overal totals still have the repo with smallest files the winner<br>
| Repo | Input (w/o Cache Write) | Cache Read | Output Tokens | Total Tokens | Cost |<br>
| small | 395,463.40 | 6,033,691.00 | 49,158.00 | 6,478,312.40 | 0.327 |<br>
| large | 648,632.20 | 4,643,444.80 | 41,759.40 | 5,333,836.40 | 0.340 |<br>
| humongous | 936,815.20 | 4,977,226.60 | 44,846.20 | 5,958,888.00 | 0.440 |<br>
| humongous-bloated | 368,572.00 | 6,555,456.00 | 54,138.67 | 6,978,166.67 | 0.343 |</p>

<p>So what is the conclusion here? <del>Bloat your file with comments to make it faster for AI</del> On the average smaller files are more cost effective, as they allow more cache reads when using AI agents, modify less cache. The larger you go, the more expensive it will be for any operation.</p>

<p>As usual, the <a href="https://github.com/Bwca/testing_cursor-token-usage-with-different-file-sizes" rel="noopener noreferrer">repo with code</a>.</p>

