---
Title: I Built an Open-Source CLI to Compare LLM API Costs in Your Terminal (npx, Zero Install)
Description: 
Author: followtayeeb
Date: 2026-03-08T21:49:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you've ever had to compare costs between GPT-4o, Claude Sonnet, and Gemini before committing to a model, you know the pain: browser tabs, manual math, outdated blog posts.</p>

<p>I built <strong>llm-costs</strong> — a zero-install CLI that does it instantly.</p>

<h2>
  
  
  The Problem
</h2>

<p>Every time I start a new project using LLMs, I go through the same ritual:</p>

<ul>
<li>Open Anthropic pricing page</li>
<li>Open OpenAI pricing page
</li>
<li>Open Google AI pricing page</li>
<li>Try to compare apples to oranges with different tokenizers</li>
<li>Do math in my head (or a spreadsheet)</li>
<li>Realize the blog post I was referencing is 6 months out of date</li>
</ul>

<p>There had to be a better way.</p>

<h2>
  
  
  Demo
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx llm-costs <span class="s2">"Build a REST API in Python"</span> <span class="nt">--compare</span>
</code></pre>

</div>



<p>This counts your prompt tokens using the actual tokenizer (tiktoken for OpenAI models, character-based estimation for others), then renders a comparison table across all major providers right in your terminal.</p>

<p>Output looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Model                    Input Cost    Output Cost   Total
─────────────────────────────────────────────────────────
deepseek-chat            $0.00003      $0.00008      $0.00011
gemini-flash-2.0         $0.00005      $0.00020      $0.00025
claude-haiku-3-5         $0.00020      $0.00100      $0.00120
gpt-4o-mini              $0.00027      $0.00108      $0.00135
claude-sonnet-4-5        $0.00150      $0.00750      $0.00900
gpt-4o                   $0.00375      $0.01500      $0.01875
</code></pre>

</div>



<h2>
  
  
  Features
</h2>

<ul>
<li>
<strong>Zero install</strong> — works with <code>npx</code> or <code>npm i -g llm-costs</code>
</li>
<li>
<strong>17 models, 6 providers</strong> — Anthropic, OpenAI, Google, DeepSeek, Mistral, Cohere</li>
<li>
<strong>Auto-updating prices</strong> — fetches fresh pricing weekly via GitHub Actions; client checks staleness locally (7-day TTL cache at <code>~/.llm-costs/pricing.json</code>)</li>
<li>
<strong>Batch processing</strong> — pipe a file of prompts, get cost totals: <code>llm-costs batch prompts.txt</code>
</li>
<li>
<strong>Budget guard</strong> — set a cost ceiling for CI/CD: <code>llm-costs guard --max 0.10</code>
</li>
<li>
<strong>Watch mode</strong> — live-refresh as you type your prompt</li>
<li>
<strong>MCP server mode</strong> — integrate with Claude Desktop or any MCP-compatible tool</li>
<li>
<strong>Price changelog</strong> — track when costs changed: <code>llm-costs changelog --since 30d</code>
</li>
<li>
<strong>Budget projections</strong> — <code>llm-costs budget --requests 10000</code>
</li>
</ul>

<h2>
  
  
  Auto-Updating Prices — The Key Feature
</h2>

<p>Most LLM pricing tools go stale within weeks. llm-costs solves this with a two-layer approach:</p>

<p><strong>Client-side:</strong> On each run, the CLI checks if your local <code>~/.llm-costs/pricing.json</code> is older than 7 days. If it is, it fetches fresh data from GitHub in the background (non-blocking, 5s timeout).</p>

<p><strong>Server-side:</strong> A GitHub Actions workflow runs every Monday morning, fetches prices from LiteLLM's aggregate JSON (which tracks provider pricing), diffs the result, and opens a PR if anything changed — including a markdown table showing exactly what went up or down.</p>

<p>This means the package is always reasonably fresh, and the repo stays transparent about price changes.</p>

<h2>
  
  
  Install
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># One-shot, no install:</span>
npx llm-costs <span class="s2">"your prompt here"</span>

<span class="c"># Or install globally:</span>
npm <span class="nb">install</span> <span class="nt">-g</span> llm-costs

<span class="c"># Compare across all models:</span>
npx llm-costs <span class="s2">"your prompt"</span> <span class="nt">--compare</span>

<span class="c"># Check a specific model:</span>
npx llm-costs <span class="s2">"your prompt"</span> <span class="nt">--model</span> claude-sonnet-4-5
</code></pre>

</div>



<h2>
  
  
  Why Open Source?
</h2>

<p>LLM pricing is genuinely confusing and changes frequently. I wanted something the community could maintain together — PRs to add new models, fix prices, or add providers are very welcome.</p>

<h2>
  
  
  Links
</h2>

<ul>
<li>
<strong>GitHub:</strong> <a href="https://github.com/followtayeeb/llm-costs" rel="noopener noreferrer">https://github.com/followtayeeb/llm-costs</a> ⭐ (stars appreciated!)</li>
<li>
<strong>npm:</strong> <a href="https://www.npmjs.com/package/llm-costs" rel="noopener noreferrer">https://www.npmjs.com/package/llm-costs</a>
</li>
</ul>

<p>Would love feedback in the comments — what models or features are you missing?</p>

