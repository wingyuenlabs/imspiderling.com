---
Title: How I Built a Multi-Agent Content Automation System With Claude
Description: 
Author: AI Operator
Date: 2026-03-19T21:49:54.000Z
Robots: noindex,nofollow
Template: index
---
<p>A few months ago I was using Claude like most people: one-off prompts, copy-paste results, repeat. It worked fine for individual tasks, but there was no memory, no continuity, and no compounding value. Every session started from scratch.</p>

<p>That changed when I started building what I now call a multi-agent content automation system. It runs my entire content operation: SEO articles, Reddit/forum engagement, keyword research, image generation, WordPress publishing, and lead capture. The whole thing runs mostly autonomously with me approving outputs and steering direction.</p>

<p>This article covers the architecture, the key design decisions, and the actual tech stack with real costs.</p>




<h2>
  
  
  The Core Problem With One-Off AI Prompts
</h2>

<p>When you use an LLM through a single prompt, you get a one-shot output with no context from previous work, no awareness of your existing systems, and no ability to take action beyond generating text.</p>

<p>The shift that made everything click for me was treating Claude not as a chatbot but as a <strong>runtime</strong> for executing structured workflows.</p>

<p>The architecture I landed on has three layers:</p>

<ol>
<li>
<strong>Skills</strong> - reusable workflow modules (like functions in code)</li>
<li>
<strong>State files</strong> - persistent JSON that carries context between sessions</li>
<li>
<strong>Execution scripts</strong> - Python scripts that do the heavy lifting (API calls, file I/O, browser automation)</li>
</ol>




<h2>
  
  
  Skills as Workflow Modules
</h2>

<p>A "skill" in my system is a markdown file with YAML frontmatter that instructs the AI agent how to perform a specific multi-step task. Think of it like a function definition.</p>

<p>Here's a simplified example of what a skill definition looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">rr-article-writer</span>
<span class="na">description</span><span class="pi">:</span> <span class="s">Write SEO-optimised articles for the content site</span>
<span class="nn">---</span>
</code></pre>

</div>



<p>The body of the file contains step-by-step instructions, decision rules, output formats, and references to external scripts. When I invoke a skill, the agent reads those instructions and executes them with full context of the current session.</p>

<p>This means I can have a <code>reddit-growth</code> skill that knows: check account warmup status, find threads posted within the last 30 days, write value-first replies in a colloquial tone, never post affiliate links until account trust is established. All of that logic lives in the skill file, not in my head.</p>

<p>The skills I've built so far cover:</p>

<ul>
<li>
<strong>Content pipeline</strong>: ideation, research, writing, auditing, image generation, publishing</li>
<li>
<strong>Community growth</strong>: Reddit account warmup, subreddit building, multilingual forum outreach</li>
<li>
<strong>SEO</strong>: Google Search Console + DataForSEO keyword analysis, on-page audits</li>
<li>
<strong>Revenue</strong>: email capture, lead magnet delivery, affiliate link management</li>
<li>
<strong>Utilities</strong>: cold email campaigns, market research, invoice extraction</li>
</ul>




<h2>
  
  
  State Files Keep Everything in Sync
</h2>

<p>This is the piece most people miss when they try to build something like this.</p>

<p>Each agent maintains state in JSON files. A few examples from my setup:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">agents/schedule.json</span><span class="w"> </span><span class="err">-</span><span class="w"> </span><span class="err">task</span><span class="w"> </span><span class="err">cadences</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"tasks"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Reddit Engagement"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"skill"</span><span class="p">:</span><span class="w"> </span><span class="s2">"reddit-growth"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"cadence_days"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="nl">"last_run"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-18"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RoboRhythms Article"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"skill"</span><span class="p">:</span><span class="w"> </span><span class="s2">"reddit-article-pipeline"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"cadence_days"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
      </span><span class="nl">"last_run"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-03-17"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">agents/account_state.json</span><span class="w"> </span><span class="err">-</span><span class="w"> </span><span class="err">Reddit</span><span class="w"> </span><span class="err">warmup</span><span class="w"> </span><span class="err">tracking</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"username"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ThatRandomApe"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"warmup_start"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-01-20"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"post_count"</span><span class="p">:</span><span class="w"> </span><span class="mi">47</span><span class="p">,</span><span class="w">
  </span><span class="nl">"karma"</span><span class="p">:</span><span class="w"> </span><span class="mi">89</span><span class="p">,</span><span class="w">
  </span><span class="nl">"phase"</span><span class="p">:</span><span class="w"> </span><span class="s2">"building_trust"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>At the start of each session, a briefing script reads these files and surfaces what's due today, what's overdue, and what account states look like. I open Claude, see the briefing, and can run the day's tasks with a few commands.</p>




<h2>
  
  
  The Content Pipeline End to End
</h2>

<p>Here's how a full article goes from idea to published:</p>

<p><strong>Step 1: Idea sourcing</strong><br>
The <code>reddit-article-pipeline</code> skill scans target subreddits for trending threads, identifies content gaps, and proposes article topics based on search volume data from DataForSEO.</p>

<p><strong>Step 2: Research</strong><br>
The <code>rr-notebooklm-prep</code> skill fetches the top 5 competitor articles for the chosen topic, uploads them to NotebookLM via MCP, and extracts a structured research brief: key facts, content gaps, angle ideas.</p>

<p><strong>Step 3: Writing</strong><br>
The <code>rr-article-writer</code> skill takes the research brief and writes a full article (2500-4000 words) optimised for the target keyword. It follows strict style rules: no em dashes, first-person voice, numbered lists for comparisons, working examples for every major claim.</p>

<p><strong>Step 4: Audit</strong><br>
The <code>rr-auditor</code> skill reviews the draft against a checklist: banned words, missing sections, title length, meta description, internal link opportunities. It fixes what it can automatically.</p>

<p><strong>Step 5: Images</strong><br>
The <code>rr-image-generator</code> skill generates 3 blog images using fal.ai's FLUX.1 model via their REST API. Prompts are visual metaphors only (FLUX.1 cannot render legible text, so never include words in image prompts).</p>

<p><strong>Step 6: Publish</strong><br>
The <code>rr-publisher</code> skill uploads the article to WordPress via REST API, attaches images, sets the author and category, saves as draft, and sends an email with the edit link.</p>

<p>The whole pipeline takes maybe 25 minutes of wall-clock time with 5-10 minutes of my attention.</p>




<h2>
  
  
  Community Growth: The Long Game
</h2>

<p>Content alone doesn't get traffic. You need distribution.</p>

<p>My Reddit growth system uses Firefox browser automation via Playwright MCP to post replies manually (no Reddit API). This is intentional: the API adds friction and rate limits, and browser automation gives me more flexibility for account warmup.</p>

<p>The warmup strategy:</p>

<ul>
<li>Phase 1 (weeks 1-4): value-only replies, no links, target threads under 30 days old</li>
<li>Phase 2 (weeks 5-8): soft mentions of the site by name, still no direct links</li>
<li>Phase 3 (8+ weeks, 50+ karma): can include affiliate/site links in appropriate context</li>
</ul>

<p>For forums, I have a multilingual system covering English, Spanish, Portuguese, French, German, and Dutch forums. The <code>forum-growth</code> skill has two modes: RESEARCH (scans forums, drafts replies, builds a queue) and POSTING (executes the queue via browser automation, generates a PDF report).</p>




<h2>
  
  
  SEO Automation With Real Data
</h2>

<p>The SEO layer uses two data sources:</p>

<p><strong>Google Search Console</strong> via a custom Python MCP server. This surfaces real impressions, clicks, CTR, and position data per URL and per query.</p>

<p><strong>DataForSEO</strong> for competitor SERP analysis, keyword difficulty, search volume, and on-page audit data.</p>

<p>The <code>rr-seo-auditor</code> skill cross-references both: it pulls GSC data to find pages losing clicks, then uses DataForSEO to check if competitors have overtaken those positions and why. Output is a prioritized action plan ranked by estimated traffic impact.</p>




<h2>
  
  
  The Tech Stack and Real Costs
</h2>

<p>Here's what the system actually runs on and what it costs per month:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tool</th>
<th>Purpose</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>Claude Pro</td>
<td>Core AI runtime</td>
<td>$20/mo</td>
</tr>
<tr>
<td>fal.ai</td>
<td>Image generation</td>
<td>~$5/mo (usage-based)</td>
</tr>
<tr>
<td>DataForSEO</td>
<td>SERP + keyword data</td>
<td>~$30/mo (usage-based)</td>
</tr>
<tr>
<td>ElevenLabs</td>
<td>TTS for video</td>
<td>$5/mo (starter)</td>
</tr>
<tr>
<td>Perplexity</td>
<td>Real-time web search</td>
<td>$20/mo</td>
</tr>
<tr>
<td>WordPress hosting</td>
<td>Content site</td>
<td>$15/mo</td>
</tr>
<tr>
<td>Systeme.io</td>
<td>Email list + automation</td>
<td>Free tier</td>
</tr>
</tbody>
</table></div>

<p>Total: roughly $95/month to run a full content, SEO, and community operation that would otherwise require a team.</p>

<p>The return on that $95 comes from affiliate commissions when articles rank and convert. On a good month the margin is 10-20x.</p>




<h2>
  
  
  What I Got Wrong Early On
</h2>

<p>A few mistakes worth learning from:</p>

<p><strong>Mistake 1: No state files.</strong> Early version had no memory between sessions. Every task restarted from scratch and I was re-explaining context constantly. State files fixed this.</p>

<p><strong>Mistake 2: Prompts that were too generic.</strong> Skill files need to be specific. "Write a good article" produces generic output. "Write a 2500-word comparison article with a numbered features table, at least 3 worked examples, and a summary box at the end" produces something publishable.</p>

<p><strong>Mistake 3: Skipping the audit step.</strong> I used to publish articles straight from the writer. The auditor catches em dashes, missing sections, off-brand tone, and title length issues every single time. It's worth the extra minute.</p>

<p><strong>Mistake 4: Over-automating too fast.</strong> Start with one pipeline working end-to-end. Get the content pipeline solid before adding community growth. Get community growth working before adding SEO automation. Trying to build everything at once means nothing works reliably.</p>




<h2>
  
  
  What's Next
</h2>

<p>A few things I'm building toward:</p>

<ul>
<li>WhatsApp bot for mobile idea capture (bridge already built, in testing)</li>
<li>YouTube pipeline: ElevenLabs narration + Kling video generation + ffmpeg assembly</li>
<li>Automated cold outreach tied to the market research suite</li>
</ul>

<p>The architecture scales well because each skill is independent. Adding a new capability means writing a new skill file and a Python script, not rebuilding anything.</p>




<h2>
  
  
  Go Deeper
</h2>

<p>If you want the full system documented: architecture diagrams, skill templates, state file structures, the complete prompt system, and the exact tools with setup instructions, I packaged everything into a 35-page guide.</p>

<p><strong><a href="https://payhip.com/b/mAic1" rel="noopener noreferrer">The AI Agent Automation Blueprint 2026</a></strong> covers exactly what's described here, with enough detail to replicate the system from scratch.</p>

<p>Happy to answer questions in the comments about any specific part of the architecture.</p>

