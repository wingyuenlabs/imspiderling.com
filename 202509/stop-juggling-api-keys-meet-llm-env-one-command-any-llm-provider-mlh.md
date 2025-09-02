---
Title: Stop Juggling API Keys: Meet llm-env — One Command, Any LLM Provider
Description: 
Author: Sam Estrin
Date: 2025-09-02T21:30:11.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  TL;DR
</h2>

<p>If you bounce between multiple AI providers like OpenAI, Gemini, Groq, Cerebras, or local LLMs—and you want an OpenAI-compatible workflow—this tiny Bash environment helper is for you. It simplifies LLM provider switching, keeps your API keys organized, and boosts developer productivity.</p>

<p><a href="https://github.com/samestrin/llm-env" rel="noopener noreferrer"><code>llm-env</code></a> is a tiny Bash script that standardizes your Bash environment around the familiar OPENAI_* variables so OpenAI-compatible tools "just work" across providers.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Switch providers in one command</span>
llm-env <span class="nb">set </span>openai
llm-env <span class="nb">set </span>gemini
llm-env <span class="nb">set </span>groq
</code></pre>

</div>



<p>Result: Your existing AI tools (aider, llm, qwen-code, LiteLLM) immediately pick up the right API key, base URL, and model. No manual edits, no copy/paste, no restarts.</p>

<h2>
  
  
  The Problem (You May Have Felt This Today)
</h2>

<ul>
<li>Multiple providers, each with different endpoints and auth</li>
<li>OPENAI_* has become the de facto standard—but not every provider uses those names</li>
<li>You end up editing ~/.bashrc or ~/.zshrc over and over</li>
<li>Context switching kills flow, and small mistakes cause mysterious 401s/404s</li>
</ul>

<h2>
  
  
  A Developer Story
</h2>

<p>Sarah, an ML engineer at a fintech startup, prototypes using the Gemini free tier, uses Groq for CI speed, and ships with OpenAI in production. With <code>llm-env</code>, she changes providers with a single command and avoids configuration drift across environments. </p>

<p>The result: <strong>faster cycles</strong> and <strong>fewer “why is this failing?” moments</strong>.</p>

<h2>
  
  
  The Solution: <a href="https://github.com/samestrin/llm-env" rel="noopener noreferrer"><code>llm-env</code></a>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fylderczpgbos0fdzfn02.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fylderczpgbos0fdzfn02.png" alt="llm-env --help" width="800" height="408"></a></p>

<p>A single script that:</p>

<ol>
<li>Centralizes provider configuration in one place (~/.config/llm-env/llm-env.conf)</li>
<li>Normalizes every provider to OPENAI_* environment variables</li>
<li>Let's you switch providers instantly with <code>llm-env set &lt;provider&gt;</code>
</li>
<li>Includes a built-in connectivity test so you know your provider works</li>
</ol>

<h2>
  
  
  What Using It Feels Like
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwhjqyy55blmrc96s0oze.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwhjqyy55blmrc96s0oze.png" alt="Example llm-env Workflow" width="800" height="575"></a></p>

<p>Before vs. After:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Before (manual OPENAI_* exports)</span>
<span class="nb">export </span><span class="nv">OPENAI_API_KEY</span><span class="o">=</span><span class="s2">"sk-••••abcd"</span>
<span class="nb">export </span><span class="nv">OPENAI_BASE_URL</span><span class="o">=</span><span class="s2">"https://api.openai.com/v1"</span>
<span class="nb">export </span><span class="nv">OPENAI_MODEL</span><span class="o">=</span><span class="s2">"gpt-5"</span>
<span class="nb">source</span> ~/.bashrc  <span class="c"># reload to apply changes</span>

<span class="c"># After (one command)</span>
llm-env <span class="nb">set </span>openai  <span class="c"># sets OPENAI_API_KEY, OPENAI_BASE_URL, OPENAI_MODEL</span>
</code></pre>

</div>



<p>Common Commands:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span>llm-env list         <span class="c"># Browse configured providers</span>
<span class="nv">$ </span>llm-env <span class="nb">set </span>openai   <span class="c"># Switch instantly</span>
<span class="nv">$ </span>llm-env <span class="nb">test </span>openai  <span class="c"># Verify connectivity and permissions</span>
<span class="nv">$ </span>llm-env show         <span class="c"># See exactly what’s active</span>
</code></pre>

</div>



<h2>
  
  
  Installation (30 seconds)
</h2>

<p>Quickly install <code>llm-env</code> with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://raw.githubusercontent.com/samestrin/llm-env/main/install.sh | bash
</code></pre>

</div>



<p>Add your keys to your shell profile (examples):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">LLM_OPENAI_API_KEY</span><span class="o">=</span><span class="s2">"your_openai_key"</span>
<span class="nb">export </span><span class="nv">LLM_CEREBRAS_API_KEY</span><span class="o">=</span><span class="s2">"your_cerebras_key"</span>
<span class="nb">export </span><span class="nv">LLM_GROQ_API_KEY</span><span class="o">=</span><span class="s2">"your_groq_key"</span>
<span class="c"># ...add keys for the providers you use</span>
</code></pre>

</div>



<p>Start using <code>llm-env</code> right away:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>llm-env list
llm-env <span class="nb">set </span>openai
llm-env <span class="nb">test </span>openai
llm-env show
</code></pre>

</div>



<h2>
  
  
  Pre‑Configured for the Modern AI Stack
</h2>

<p><code>llm-env</code> ships with 20 popular providers ready to go and works with <strong>any OpenAI‑compatible API</strong>. You can easily add your own providers by editing a single config file.</p>

<ul>
<li>Cloud providers (OpenAI, Groq, Gemini, Cerebras, xAI, and more)</li>
<li>OpenRouter presets (including free options)</li>
<li>Self‑hosted setups (Ollama, LM Studio, vLLM)</li>
</ul>

<h2>
  
  
  Why Standardize on OPENAI_*?
</h2>

<p>Most AI tools already expect these variables:</p>

<ul>
<li>OPENAI_API_KEY</li>
<li>OPENAI_BASE_URL</li>
<li>OPENAI_MODEL</li>
</ul>

<p><code>llm-env</code> embraces that reality. It updates those variables for you—correctly—no matter the provider. Your tools stay unchanged; your provider becomes a one‑line decision.</p>

<h2>
  
  
  Security First
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbxtph6cnpn9wxjvhxdnf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbxtph6cnpn9wxjvhxdnf.png" alt="llm-env show (demonstrating masked keys)" width="800" height="344"></a></p>

<p><em>Keys are masked in output (e.g., ••••15x0) to keep secrets safe on screen and in screenshots.</em></p>

<p>Security is a top priority:</p>

<ul>
<li>Keys live in environment variables—never written to config files</li>
<li>Outputs are masked (e.g., ••••abcd) — see the llm-env show output for an example</li>
<li>Switching is local; nothing is sent over the network except your own API calls during tests</li>
</ul>

<h3>
  
  
  Why Bash?
</h3>

<p>I wrote <code>llm-env</code> in Bash so it runs anywhere Bash runs—macOS, Linux, containers, CI—without asking you to install Python or Node first. It’s intentionally compatible with older shells and includes shims for pre-4.0 behavior.</p>

<ul>
<li>Works out-of-the-box on macOS’s default Bash 3.2 and modern Bash 5.x installations; Linux distros with Bash 4.0+ are covered as well.</li>
<li>Backwards-compatible layer for older shells ensures features like associative arrays “just work,” even on Bash 3.2.</li>
<li>Verified by an automated test matrix across Bash 3.2, 4.0+, and 5.x on macOS and Linux (see README → Testing).</li>
</ul>

<h2>
  
  
  Advanced Workflows (Examples)
</h2>

<p>Cost‑optimized development:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>llm-env <span class="nb">set </span>gemini     <span class="c"># take advantage of Gemini's free tier</span>
<span class="c"># ... iterate quickly</span>
llm-env <span class="nb">set </span>openai     <span class="c"># switch to OpenAI for final runs</span>
</code></pre>

</div>



<p>Provider‑specific optimization:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Code generation and debugging</span>
llm-env <span class="nb">set </span>deepseek
<span class="c"># Generate functions, fix bugs, code reviews</span>

<span class="c"># Real-time applications requiring speed</span>
llm-env <span class="nb">set </span>groq
<span class="c"># Chat interfaces, live demos, rapid prototyping</span>

<span class="c"># Complex analysis and reasoning tasks</span>
llm-env <span class="nb">set </span>openai
<span class="c"># Strategic planning, research synthesis, complex problem-solving</span>
</code></pre>

</div>



<p>Environment‑aware deployment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># dev → staging → prod with different providers</span>
llm-env <span class="nb">set </span>cerebras
llm-env <span class="nb">set </span>openrouter2
llm-env <span class="nb">set </span>openai
</code></pre>

</div>



<h2>
  
  
  Try It
</h2>

<h3>
  
  
  Install (Quick)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-fsSL</span> https://raw.githubusercontent.com/samestrin/llm-env/main/install.sh | bash

<span class="c"># Configure your OpenAI key</span>
<span class="nb">echo</span> <span class="s1">'export LLM_OPENAI_API_KEY="your_key"'</span> <span class="o">&gt;&gt;</span> ~/.bashrc

<span class="c"># Switch in one line</span>
llm-env <span class="nb">set </span>openai
</code></pre>

</div>



<h3>
  
  
  Install
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>
curl <span class="nt">-fsSL</span> https://raw.githubusercontent.com/samestrin/llm-env/main/install.sh | bash
llm-env config init
llm-env config edit  <span class="c"># Configure your API keys variables here</span>
llm-env <span class="nb">set </span>openai   <span class="c"># Now you're ready to go!</span>
</code></pre>

</div>



<p>Repository: <a href="https://github.com/samestrin/llm-env" rel="noopener noreferrer">https://github.com/samestrin/llm-env</a> (<a href="https://github.com/samestrin/llm-env/tree/main/docs" rel="noopener noreferrer">docs</a>) (<a href="https://github.com/samestrin/llm-env/tree/main/tests" rel="noopener noreferrer">tests</a>)</p>

<p><strong>Question for the community</strong>: What's your biggest pain point when working with multiple LLM providers? How do you currently manage API keys and environment switching?</p>

<p>Drop a comment below—I'd love to hear about your workflow and how <code>llm-env</code> might fit in.</p>

<p>⭐ <strong><a href="https://github.com/samestrin/llm-env" rel="noopener noreferrer">Star the repo</a></strong> if this solves a problem you've been facing. The more developers who adopt standardized tooling, the better the entire ecosystem becomes.</p>

