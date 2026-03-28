---
Title: Your AI Doesn't Need Screenshots. It Needs DevTools.
Description: 
Author: Emad Omar
Date: 2026-03-28T22:04:34.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Your AI Doesn't Need Screenshots. It Needs DevTools.
</h1>

<p>Most AI coding agents are still debugging web apps in the dumbest possible way.</p>

<p>They ask for a screenshot.</p>

<p>Meanwhile the real answer is usually sitting in the browser console, the Network tab, the request payload, or the response body.</p>

<p>That is not really an AI problem. It is a tooling problem.</p>

<p>I got tired of watching agents guess from screenshots while I had DevTools open right next to them, showing the exact reason something failed. So I built <a href="https://github.com/emadklenka/mare_browser_mcp" rel="noopener noreferrer"><code>mare-browser-mcp</code></a>, a browser MCP designed less like "remote control for a webpage" and more like "give the model the same debugging signals I actually use."</p>

<p>That changed the loop from this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AI writes code -&gt; I test it -&gt; I describe the bug -&gt; AI guesses a fix -&gt; repeat
</code></pre>

</div>



<p>to this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AI writes code -&gt; AI tests it -&gt; AI reads the failure -&gt; AI fixes it
</code></pre>

</div>



<p>That difference matters more than people think.</p>

<h2>
  
  
  The Screenshot Trap
</h2>

<p>Most browser MCPs start from the same assumption: if the model can see the page, it can debug the app.</p>

<p>That sounds reasonable until you try to debug anything real.</p>

<p>A screenshot does not tell the model:</p>

<ul>
<li>that login failed with a <code>401</code>
</li>
<li>that the frontend sent <code>user_email</code> instead of <code>email</code>
</li>
<li>that a button click triggered a JS exception</li>
<li>that the UI is fine but the API returned invalid JSON</li>
<li>that the missing data is actually below the fold in a scrollable grid</li>
<li>that the context menu only appears on right-click</li>
</ul>

<p>A screenshot is expensive, unstructured, and often the least useful artifact in the whole debugging flow.</p>

<p>Language models are good at reasoning over structured text. We keep handing them pixels and asking them to guess.</p>

<h2>
  
  
  What I Wanted Instead
</h2>

<p>I wanted a browser MCP that makes the model think more like a developer with DevTools open.</p>

<p>That means a few things:</p>

<ul>
<li>It should read console logs and page errors directly</li>
<li>It should see network requests, payloads, response bodies, and timing</li>
<li>It should batch actions instead of making one tiny tool call per click</li>
<li>It should query the DOM as structured data</li>
<li>It should handle hover, drag, right-click, and nested scroll containers</li>
<li>It should only use screenshots when the problem is actually visual</li>
</ul>

<p>That is the core idea behind <code>mare-browser-mcp</code>.</p>

<p>It is a lean MCP server built with Playwright and the MCP SDK, and the whole design is biased toward structured debugging data first.</p>

<h2>
  
  
  The One Tool I Wish More Browser MCPs Had
</h2>

<p>The most important tool in the server is <code>browser_debug</code>.</p>

<p>One call returns:</p>

<ul>
<li>current URL</li>
<li>page title</li>
<li>console logs</li>
<li>page errors</li>
<li>dialogs</li>
<li>network request history</li>
</ul>

<p>And the network history is the useful part. The server captures things like:</p>

<ul>
<li>method</li>
<li>URL</li>
<li>parsed query params</li>
<li>request headers</li>
<li>request body</li>
<li>status code</li>
<li>JSON response body</li>
<li>duration in milliseconds</li>
</ul>

<p>So instead of the model seeing "submit button clicked and page still looks wrong," it can see:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"method"</span><span class="p">:</span><span class="w"> </span><span class="s2">"POST"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://app.example.com/api/session"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"requestBody"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"user_email"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user@example.com"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"password"</span><span class="p">:</span><span class="w"> </span><span class="s2">"secret"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="mi">401</span><span class="p">,</span><span class="w">
  </span><span class="nl">"responseBody"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"error"</span><span class="p">:</span><span class="w"> </span><span class="s2">"missing field: email"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"duration_ms"</span><span class="p">:</span><span class="w"> </span><span class="mi">23</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>That is a completely different debugging experience.</p>

<p>At that point the model is not guessing anymore. It can actually reason from evidence.</p>

<p>There are a few guardrails in the implementation too:</p>

<ul>
<li>static assets are skipped</li>
<li>
<code>OPTIONS</code> preflights are ignored</li>
<li>bodies are capped at 4 KB</li>
<li>auth headers are masked</li>
<li>cookies are reduced to <code>"[present]"</code>
</li>
<li>failed requests are logged with the failure reason</li>
</ul>

<p>So the data stays useful without turning into junk.</p>

<h2>
  
  
  Batching Matters More Than It Sounds
</h2>

<p>Another thing that slows agents down is one-action-per-call design.</p>

<p>Click. Wait. Screenshot. Fill. Wait. Screenshot.</p>

<p>That is not just annoying. It makes the whole debugging loop slower and dumber.</p>

<p><code>browser_act</code> fixes that by accepting an ordered list of actions in one call:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">[</span><span class="w">
  </span><span class="p">{</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"fill"</span><span class="p">,</span><span class="w"> </span><span class="nl">"selector"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#email"</span><span class="p">,</span><span class="w"> </span><span class="nl">"value"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user@example.com"</span><span class="w"> </span><span class="p">},</span><span class="w">
  </span><span class="p">{</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"fill"</span><span class="p">,</span><span class="w"> </span><span class="nl">"selector"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#password"</span><span class="p">,</span><span class="w"> </span><span class="nl">"value"</span><span class="p">:</span><span class="w"> </span><span class="s2">"secret"</span><span class="w"> </span><span class="p">},</span><span class="w">
  </span><span class="p">{</span><span class="w"> </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"click"</span><span class="p">,</span><span class="w"> </span><span class="nl">"selector"</span><span class="p">:</span><span class="w"> </span><span class="s2">"button[type=submit]"</span><span class="w"> </span><span class="p">}</span><span class="w">
</span><span class="p">]</span><span class="w">
</span></code></pre>

</div>



<p>The current action set covers a lot of what real apps actually need:</p>

<ul>
<li><code>click</code></li>
<li><code>hover</code></li>
<li><code>drag</code></li>
<li><code>clicklink</code></li>
<li><code>fill</code></li>
<li><code>select</code></li>
<li><code>keypress</code></li>
<li><code>waitfor</code></li>
<li><code>scrollto</code></li>
<li><code>wait</code></li>
<li><code>clearconsole</code></li>
</ul>

<p>That means the model is not stuck the moment the UI uses hover states, resize handles, sortable grids, or context menus.</p>

<p>And yes, right-click works through <code>click</code> with <code>button: "right"</code>, which turns out to matter more often than you might expect.</p>

<h2>
  
  
  Structured DOM Queries Beat Giant Dumps
</h2>

<p>Another thing I did not want was a browser tool that just vomits the whole DOM back at the model.</p>

<p><code>browser_query</code> is useful because it lets the model ask narrower questions:</p>

<ul>
<li>How many rows are there?</li>
<li>Is this button visible?</li>
<li>What text do these badges have?</li>
<li>Is this input disabled?</li>
</ul>

<p>Examples:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w"> </span><span class="nl">"selector"</span><span class="p">:</span><span class="w"> </span><span class="s2">".ag-row"</span><span class="p">,</span><span class="w"> </span><span class="nl">"count_only"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w"> </span><span class="p">}</span><span class="w">
</span></code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w"> </span><span class="nl">"selector"</span><span class="p">:</span><span class="w"> </span><span class="s2">"button"</span><span class="p">,</span><span class="w"> </span><span class="nl">"all"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w"> </span><span class="nl">"visible_only"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w"> </span><span class="nl">"limit"</span><span class="p">:</span><span class="w"> </span><span class="mi">10</span><span class="w"> </span><span class="p">}</span><span class="w">
</span></code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w"> </span><span class="nl">"selector"</span><span class="p">:</span><span class="w"> </span><span class="s2">".badge"</span><span class="p">,</span><span class="w"> </span><span class="nl">"fields"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"text"</span><span class="p">,</span><span class="w"> </span><span class="s2">"className"</span><span class="p">,</span><span class="w"> </span><span class="s2">"visible"</span><span class="p">]</span><span class="w"> </span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The supported fields are deliberately practical:</p>

<ul>
<li><code>text</code></li>
<li><code>value</code></li>
<li><code>visible</code></li>
<li><code>disabled</code></li>
<li><code>className</code></li>
<li><code>href</code></li>
<li><code>innerHTML</code></li>
</ul>

<p>That keeps the output small enough to reason about and avoids wasting context on useless noise.</p>

<h2>
  
  
  Screenshots Are Still There. They Just Aren't First.
</h2>

<p>I did keep <code>browser_screenshot</code>.</p>

<p>But I made the tool description explicitly say it should be the last resort, not the default.</p>

<p>That sounds minor, but it really changes how an agent behaves. If the tool surface itself tells the model:</p>

<ul>
<li>use <code>browser_debug</code> first</li>
<li>use <code>browser_query</code> second</li>
<li>use screenshots only for visual problems</li>
</ul>

<p>then the model starts spending its attention on data instead of appearance.</p>

<p>That is the behavior I wanted.</p>

<h2>
  
  
  Real Apps Need More Than Page Scroll
</h2>

<p>One of the easiest ways browser tools fall apart is scrolling.</p>

<p>Scrolling the page is not enough for modern apps. A lot of important UI is inside:</p>

<ul>
<li>data grid viewports</li>
<li>chat panels</li>
<li>sidebars</li>
<li>overflow containers</li>
<li>modals</li>
</ul>

<p><code>browser_scroll</code> can scroll the page, scroll an element into view, or scroll inside a specific container.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"container"</span><span class="p">:</span><span class="w"> </span><span class="s2">".ag-body-viewport"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"direction"</span><span class="p">:</span><span class="w"> </span><span class="s2">"down"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"pixels"</span><span class="p">:</span><span class="w"> </span><span class="mi">500</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>And because it returns scroll position data, the model can tell where it is instead of just flailing around.</p>

<h2>
  
  
  There Is Also an Escape Hatch
</h2>

<p>Even with all of that, fixed tools never cover every case.</p>

<p>So there is <code>browser_eval</code>, which runs JavaScript in the page context.</p>

<p>That is useful for things like:</p>

<ul>
<li>reading computed styles</li>
<li>inspecting app state on <code>window</code>
</li>
<li>appending input text without clearing it</li>
<li>simulating more custom interactions</li>
<li>calling <code>fetch()</code> directly</li>
<li>checking CSS visibility details</li>
</ul>

<p>I do not think the escape hatch should be the first thing the model reaches for. But it absolutely should exist.</p>

<h2>
  
  
  What This Looks Like in Practice
</h2>

<p>A realistic flow is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. browser_navigate({ url: "https://myapp.com", clear_logs: true })

2. browser_act({
     commands: [
       { action: "fill", selector: "#email", value: "user@example.com" },
       { action: "fill", selector: "#password", value: "secret" },
       { action: "click", selector: "button[type=submit]" }
     ]
   })

3. browser_wait_for_network({ url_pattern: "/api/session", method: "POST" })

4. browser_debug({ console_types: ["error"], last_n: 20 })
</code></pre>

</div>



<p>From there the model can see:</p>

<ul>
<li>whether the request fired</li>
<li>what payload got sent</li>
<li>what response came back</li>
<li>whether the browser threw an error</li>
<li>whether the failure is frontend, backend, or both</li>
</ul>

<p>That is the kind of loop where an agent becomes genuinely useful.</p>

<h2>
  
  
  Why I Think This Matters
</h2>

<p>I think a lot of people are measuring browser MCPs by the wrong thing.</p>

<p>The question is not "can the AI click buttons?"</p>

<p>The real question is: when something breaks, can the AI see the same evidence a good developer would look at?</p>

<p>If the answer is yes, the quality of debugging jumps fast.</p>

<p>If the answer is no, you end up with a very expensive screenshot viewer.</p>

<p>That is why I built <code>mare-browser-mcp</code> the way I did. Not to make agents better at looking at pages, but to make them better at understanding what happened in the browser.</p>

<p>Not screenshots first. Telemetry first.</p>

<h2>
  
  
  Try It
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/emadklenka/mare_browser_mcp
<span class="nb">cd </span>mare_browser_mcp
pnpm <span class="nb">install
</span>npx playwright <span class="nb">install </span>chromium
pnpm run setup
</code></pre>

</div>



<p>It also supports global install:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add <span class="nt">-g</span> mare-browser-mcp
npx playwright <span class="nb">install </span>chromium
</code></pre>

</div>



<p>The repo has setup instructions for Claude Code and OpenCode, and the current server exposes:</p>

<ul>
<li><code>browser_navigate</code></li>
<li><code>browser_act</code></li>
<li><code>browser_debug</code></li>
<li><code>browser_query</code></li>
<li><code>browser_screenshot</code></li>
<li><code>browser_eval</code></li>
<li><code>browser_scroll</code></li>
<li><code>browser_restart</code></li>
<li><code>browser_upload</code></li>
<li><code>browser_wait_for_network</code></li>
</ul>

<p>If you are building web apps with AI, that combination is much closer to giving your agent DevTools than giving it a screenshot and hoping for the best.</p>

