---
Title: I Built an AI Pipeline for Books, Here's the Architecture
Description: 
Author: Mykyta Chernenko
Date: 2026-02-22T21:52:13.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  We Treated Book Generation as a Compiler Pipeline. Here's What We Learned From 50K Books.
</h1>

<p>Most AI writing tools are chat wrappers. Paste a prompt, get text, copy into Google Docs, repeat. For a full book that's hundreds of round trips, and you lose all context between them.</p>

<p>I've spent 3 years in the AI + publishing space. Published books myself, built a reading platform (NanoReads, 130+ books, 341K readers), talked to hundreds of authors. The same complaints kept coming up: AI loses track of what happened 10 chapters ago, every chapter sounds different, dialogue is flat, and the output is full of "Moreover," and "Furthermore," and "It's worth noting that."</p>

<p>These aren't model quality problems. After generating 50K+ books on our platform (<a href="https://aiwritebook.com" rel="noopener noreferrer">AIWriteBook</a>), we're pretty confident the bottleneck is the specification pipeline, not the language model.</p>

<h2>
  
  
  The architecture
</h2>

<p>We treat book creation as a multi-stage compilation pipeline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Book Metadata -&gt; Character Graph -&gt; Chapter Outlines -&gt; Chapter Content
     |               |                  |                  |
  (schema)       (schema)           (schema)          (streaming)
</code></pre>

</div>



<p>Each stage produces schema-constrained structured output that feeds the next stage. Nothing is freeform until the final prose generation.</p>

<h3>
  
  
  Stage 1: Book metadata
</h3>

<p>User provides title + description. AI generates structured details:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"The Dragon's Reluctant Mate"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"genres"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Fantasy"</span><span class="p">,</span><span class="w"> </span><span class="s2">"Romance"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"tone"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"dark"</span><span class="p">,</span><span class="w"> </span><span class="s2">"romantic"</span><span class="p">,</span><span class="w"> </span><span class="s2">"suspenseful"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"style"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"dialogue-heavy"</span><span class="p">,</span><span class="w"> </span><span class="s2">"fast-paced"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"target_audience"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Adult fantasy romance readers"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"plot_techniques"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"enemies-to-lovers"</span><span class="p">,</span><span class="w"> </span><span class="s2">"slow-burn"</span><span class="p">,</span><span class="w"> </span><span class="s2">"foreshadowing"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"writing_style"</span><span class="p">:</span><span class="w"> </span><span class="s2">"..."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Everything downstream uses this as context. Tone, style, audience are constraints, not suggestions.</p>

<h3>
  
  
  Stage 2: Character graph
</h3>

<p>Each character is a structured node with voice, motivation, arc, internal conflict. The important bit: when generating a chapter, we only pass the characters present in that chapter. The model gets their specific voice patterns, current arc position, relationship dynamics with the other characters in the scene.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Kira Ashvane"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"protagonist"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"voice"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Sharp, clipped sentences. Uses sarcasm as defense."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"motivation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Prove she doesn't need the dragon clan's protection"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"internal_conflict"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Craves belonging but fears vulnerability"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"arc"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Isolation -&gt; reluctant alliance -&gt; trust -&gt; sacrifice"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This is why dialogue doesn't all sound the same. The model has explicit voice specs per character instead of trying to infer it from nothing.</p>

<h3>
  
  
  Stage 3: Chapter outlines
</h3>

<p>This turned out to be the most important stage. Each chapter gets a spec:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"chapter_number"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
  </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"The Binding Ceremony"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"events"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Kira is forced to attend the bonding ritual"</span><span class="p">,</span><span class="w"> </span><span class="s2">"..."</span><span class="p">],</span><span class="w">
  </span><span class="nl">"locations"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Dragon temple, obsidian halls lit by bioluminescent moss"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"twists"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"The ritual reveals Kira has dormant dragon magic"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"character_interactions"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"characters"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Kira"</span><span class="p">,</span><span class="w"> </span><span class="s2">"Draethor"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"dynamic"</span><span class="p">:</span><span class="w"> </span><span class="s2">"hostile tension with undercurrent of curiosity"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"word_count"</span><span class="p">:</span><span class="w"> </span><span class="mi">2800</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>We ran an internal comparison: same book concept, same voice training, one group used the default generated outline, the other spent time customizing it.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Default outline</th>
<th>Customized outline</th>
</tr>
</thead>
<tbody>
<tr>
<td>Export rate</td>
<td>16%</td>
<td>34%</td>
</tr>
<tr>
<td>Satisfaction</td>
<td>3.4/5</td>
<td>4.3/5</td>
</tr>
<tr>
<td>Regenerations/chapter</td>
<td>1.8</td>
<td>0.7</td>
</tr>
<tr>
<td>Completion rate</td>
<td>41%</td>
<td>72%</td>
</tr>
</tbody>
</table></div>

<p>A mediocre model with a detailed outline beats a good model with a vague outline. This is the same lesson as software: garbage requirements produce garbage output regardless of how good the team is.</p>

<h3>
  
  
  Stage 4: Chapter generation
</h3>

<p>The only streaming stage. The model receives book metadata, relevant characters with voice specs, this chapter's outline, previous chapter summaries for continuity, and the author's writing style samples.</p>

<p>Two-model strategy: Gemini Flash for all structural work (fast, cheap, good at structured output), frontier model for actual prose.</p>

<h2>
  
  
  Voice training
</h2>

<p>Authors can upload 3-5 writing samples. We extract style features and use them as few-shot examples during generation.</p>

<p>The numbers from our data:</p>

<ul>
<li>2.4x higher export rate with voice training</li>
<li>41% fewer regeneration requests</li>
<li>67% less manual editing</li>
</ul>

<p>Fewer than 3 samples: marginal improvement. More than 5: diminishing returns. We were surprised how narrow the sweet spot is.</p>

<p>Without voice training, the output sounds like default GPT. Authors recognize it instantly and either abandon the book or spend hours rewriting. With voice training, most of the "AI slop" problem disappears. The model is capable of varied prose, it just needs examples to anchor on.</p>

<h2>
  
  
  Fiction vs. nonfiction are different pipelines
</h2>

<p>Fiction uses the character graph + plot continuity pipeline above.</p>

<p>Nonfiction is a separate architecture. Authors upload reference materials (research papers, coaching notes, blog posts, whatever their source material is). We extract content pieces and assign relevant ones to each chapter.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Reference Files -&gt; Content Extraction -&gt; Book Structure Selection
                                               |
                               Chapter Outlines (with assigned references)
                                               |
                               Chapter Content (with citations)
</code></pre>

</div>



<p>Nonfiction with reference materials vs. without:</p>

<ul>
<li>38% higher export rate</li>
<li>Satisfaction: 4.4/5 vs. 3.5/5</li>
</ul>

<p>When the model has specific data, named studies, real quotes to ground its writing, the output gets noticeably better. Without references it falls back on training data generalizations, and readers can feel the difference.</p>

<h2>
  
  
  Things we learned from 50K books
</h2>

<p><strong>Chapter length sweet spot is 2,000-3,500 words.</strong> Below that, chapters feel underdeveloped. Above 3,500, the model starts repeating itself with different phrasing, introducing tangents, padding with unnecessary description. Above 5,000, quality drops hard. If a chapter needs to be long, splitting it works better than generating one long one.</p>

<p><strong>Genre matters a lot.</strong> Romance has a 31% export rate. Literary fiction has 11%. Humor is 13%. Poetry is 9%. The pattern: AI does well with genres that have established conventions and lots of training data. It struggles with voice-dependent and creativity-dependent writing. Makes sense intuitively, but it was useful to see the numbers.</p>

<p><strong>Only 23% of generated books get exported for publishing.</strong> The ones that do share traits: 3.2x more time on outline editing, voice training enabled in 74% of cases, at least one manual edit in 89% of chapters. The books that make it to publish are iterated on, not one-click generated.</p>

<p><strong>Multilingual quality varies a lot.</strong> Spanish, French, German are close to English quality. Polish, Russian, Japanese, Korean are good but noticeably lower. Smaller languages are usable for drafts. This maps directly to training data volume. For authors writing in smaller languages, generating in English and translating works better than generating natively.</p>

<h2>
  
  
  Stack
</h2>

<ul>
<li>Frontend: Next.js, Tailwind, Supabase client</li>
<li>Backend: Supabase Edge Functions (Deno)</li>
<li>AI: Gemini Flash (structural), frontier models (prose)</li>
<li>30+ languages supported</li>
</ul>

<h2>
  
  
  Wrapping up
</h2>

<p>The main thing we took away from building this: the quality problem in AI-generated books is a specification problem, not a model problem. If you give the model a vague prompt and hit generate, you get slop. If you give it a detailed character graph, a structured outline, voice samples, and proper constraints, the output is genuinely good.</p>

<p>If you want to poke at it, there's a free tier that gives you a full 7-chapter book: <a href="https://aiwritebook.com" rel="noopener noreferrer">aiwritebook.com</a></p>

<p>Happy to answer questions about the architecture, the data, or anything about AI + publishing.</p>




<p><em>Tags: #ai #writing #books #showdev #webdev #productivity</em></p>

