---
Title: Building a JSON Formatter: Zero-Click Auto-Format and the Clipboard API Trap
Description: 
Author: tommy
Date: 2026-03-06T22:01:57.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  When an API response comes back as a single line, what do you reach for?
</h2>

<p>Open jsonformatter.org, paste, click "Format."</p>

<p>Three steps I was repeating dozens of times a week. Then in November 2025, watchTowr Labs revealed that <strong>jsonformatter.org and codebeautify.org had leaked over 80,000 AWS keys and API secrets</strong> -- because every piece of JSON you paste gets sent to their servers. The act of formatting JSON online was itself a security risk.</p>

<p>So I built a tool where <strong>you copy JSON, open a tab, and it's already formatted. Zero server transmission.</strong></p>

<blockquote>
<p><strong>Try it now</strong> -- copy any JSON to your clipboard and open this link:<br>
<a href="https://json.puremark.app" rel="noopener noreferrer">https://json.puremark.app</a></p>
</blockquote>

<p>This article documents the traps I hit along the way: <strong>Clipboard API quirks, syntax highlighting without external libraries, and Tailwind's purge problem with innerHTML.</strong> If you're building browser-based developer tools, this might save you some debugging hours.</p>

<h2>
  
  
  Tech Stack
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>React 19 + TypeScript + Vite 6 + Tailwind CSS v4 + vite-plugin-pwa
</code></pre>

</div>



<p>A simple SPA -- no state management library needed. <code>useState</code> + <code>useCallback</code> was more than enough.</p>

<h2>
  
  
  1. Auto-Reading the Clipboard -- Clipboard API Behavior
</h2>

<h3>
  
  
  Basic Implementation
</h3>

<p>On page load, read the clipboard. If the content looks like JSON, auto-format it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">readClipboard</span> <span class="o">=</span> <span class="nf">useCallback</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nb">navigator</span><span class="p">.</span><span class="nx">clipboard</span><span class="p">.</span><span class="nf">readText</span><span class="p">().</span><span class="nf">then</span><span class="p">((</span><span class="nx">text</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">text</span><span class="p">.</span><span class="nf">trim</span><span class="p">())</span> <span class="k">return</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">t</span> <span class="o">=</span> <span class="nx">text</span><span class="p">.</span><span class="nf">trimStart</span><span class="p">();</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">{</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">[</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">"</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nf">applyInput</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}).</span><span class="k">catch</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{});</span>
<span class="p">},</span> <span class="p">[</span><span class="nx">applyInput</span><span class="p">]);</span>

<span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">readClipboard</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">onVisible</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">visibilityState</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span> <span class="nf">readClipboard</span><span class="p">();</span>
  <span class="p">};</span>
  <span class="nb">document</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">visibilitychange</span><span class="dl">'</span><span class="p">,</span> <span class="nx">onVisible</span><span class="p">);</span>
  <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">document</span><span class="p">.</span><span class="nf">removeEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">visibilitychange</span><span class="dl">'</span><span class="p">,</span> <span class="nx">onVisible</span><span class="p">);</span>
<span class="p">},</span> <span class="p">[</span><span class="nx">readClipboard</span><span class="p">]);</span>
</code></pre>

</div>



<p>Using <code>visibilitychange</code> means the clipboard is re-read every time the tab regains focus. Copy JSON in another tab, switch back, and it's already formatted.</p>

<h3>
  
  
  Gotcha #1: Don't gate auto-detection on <code>JSON.parse</code> success
</h3>

<p>My first implementation only auto-applied text that successfully parsed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Bad: first implementation</span>
<span class="kd">const</span> <span class="nx">r</span> <span class="o">=</span> <span class="nf">processJson</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
<span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">r</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="nf">applyInput</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
</code></pre>

</div>



<p>This meant <strong>JSON with syntax errors</strong> (trailing commas, mismatched quotes, etc.) was silently ignored. From the user's perspective: "I copied it, but nothing happened."</p>

<p>The fix was a simple heuristic based on the first character:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Good: fixed version</span>
<span class="kd">const</span> <span class="nx">t</span> <span class="o">=</span> <span class="nx">text</span><span class="p">.</span><span class="nf">trimStart</span><span class="p">();</span>
<span class="k">if </span><span class="p">(</span><span class="nx">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">{</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">[</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">"</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
  <span class="nf">applyInput</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>  <span class="c1">// Apply even broken JSON -&gt; show error message</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Text starting with <code>{</code>, <code>[</code>, or <code>"</code> is very likely JSON. URLs and plain text almost never trigger false positives. Broken JSON gets applied and displayed with a line-numbered error message.</p>

<blockquote>
<p><strong>Lesson learned</strong>: If you gate auto-detection on "is the data valid?", you block users who want to <em>fix</em> invalid data.</p>
</blockquote>

<h3>
  
  
  Gotcha #2: The Clipboard API "shouldn't work" on page load -- but it does
</h3>

<p><code>navigator.clipboard.readText()</code> is supposed to require a user gesture (click, etc.) or it throws a permission error. MDN says so.</p>

<p>But in Chrome, <strong>calling it on page load triggers the permission prompt, and if the user grants it, the read succeeds.</strong> This was the behavior I'd been relying on from the start.</p>

<p>At one point I removed the on-mount call because "the spec says it shouldn't work." Auto-read broke completely.</p>

<blockquote>
<p><strong>Lesson learned</strong>: Don't delete working code because "the spec says it shouldn't work." If you must, test on real devices first.</p>
</blockquote>

<h3>
  
  
  Fallback: Paste Event
</h3>

<p>For environments where the Clipboard API isn't available, a <code>paste</code> event listener provides a fallback. <code>Ctrl+V</code> / <code>Cmd+V</code> works regardless of browser permissions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">onPaste</span> <span class="o">=</span> <span class="p">(</span><span class="na">e</span><span class="p">:</span> <span class="nx">ClipboardEvent</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">((</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span> <span class="k">as</span> <span class="nx">HTMLElement</span><span class="p">)?.</span><span class="nx">tagName</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">TEXTAREA</span><span class="dl">'</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">text</span> <span class="o">=</span> <span class="nx">e</span><span class="p">.</span><span class="nx">clipboardData</span><span class="p">?.</span><span class="nf">getData</span><span class="p">(</span><span class="dl">'</span><span class="s1">text</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">text</span><span class="p">?.</span><span class="nf">trim</span><span class="p">())</span> <span class="nf">applyInput</span><span class="p">(</span><span class="nx">text</span><span class="p">);</span>
  <span class="p">};</span>
  <span class="nb">document</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">paste</span><span class="dl">'</span><span class="p">,</span> <span class="nx">onPaste</span><span class="p">);</span>
  <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nb">document</span><span class="p">.</span><span class="nf">removeEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">paste</span><span class="dl">'</span><span class="p">,</span> <span class="nx">onPaste</span><span class="p">);</span>
<span class="p">},</span> <span class="p">[</span><span class="nx">applyInput</span><span class="p">]);</span>
</code></pre>

</div>



<p>Paste events inside the <code>TEXTAREA</code> are handled by React's <code>onChange</code>, so the document-level listener skips them to avoid duplication.</p>

<h2>
  
  
  2. Syntax Highlighting -- No External Libraries
</h2>

<p>highlight.js or Prism.js would make this easy, but they add tens of KBs to the bundle. For JSON-only highlighting, a custom implementation was more than sufficient.</p>

<h3>
  
  
  One Regex to Classify All Tokens
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">content</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span>
  <span class="sr">/</span><span class="se">(</span><span class="sr">"</span><span class="se">(?:\\</span><span class="sr">.|</span><span class="se">[^</span><span class="sr">"</span><span class="se">\\])</span><span class="sr">*"</span><span class="se">)\s</span><span class="sr">*:|</span><span class="se">(</span><span class="sr">"</span><span class="se">(?:\\</span><span class="sr">.|</span><span class="se">[^</span><span class="sr">"</span><span class="se">\\])</span><span class="sr">*"</span><span class="se">)</span><span class="sr">|</span><span class="se">(</span><span class="sr">true|false|null</span><span class="se">)</span><span class="sr">|</span><span class="se">(</span><span class="sr">-</span><span class="se">?\d</span><span class="sr">+</span><span class="se">(?:\.\d</span><span class="sr">+</span><span class="se">)?(?:[</span><span class="sr">eE</span><span class="se">][</span><span class="sr">+-</span><span class="se">]?\d</span><span class="sr">+</span><span class="se">)?)</span><span class="sr">|</span><span class="se">([</span><span class="sr">{}</span><span class="se">\[\]])</span><span class="sr">|</span><span class="se">([</span><span class="sr">,:</span><span class="se">])</span><span class="sr">/g</span><span class="p">,</span>
  <span class="p">(</span><span class="nx">match</span><span class="p">,</span> <span class="nx">key</span><span class="p">,</span> <span class="nx">str</span><span class="p">,</span> <span class="nx">keyword</span><span class="p">,</span> <span class="nx">num</span><span class="p">,</span> <span class="nx">bracket</span><span class="p">,</span> <span class="nx">punct</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">key</span><span class="p">)</span> <span class="k">return</span> <span class="s2">`&lt;span class="text-sky-300"&gt;</span><span class="p">${</span><span class="nf">escHtml</span><span class="p">(</span><span class="nx">key</span><span class="p">)}</span><span class="s2">&lt;/span&gt;:`</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">str</span><span class="p">)</span> <span class="k">return</span> <span class="s2">`&lt;span class="text-emerald-400"&gt;</span><span class="p">${</span><span class="nf">escHtml</span><span class="p">(</span><span class="nx">str</span><span class="p">)}</span><span class="s2">&lt;/span&gt;`</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">keyword</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">true</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">keyword</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">false</span><span class="dl">'</span><span class="p">)</span>
      <span class="k">return</span> <span class="s2">`&lt;span class="text-orange-400"&gt;</span><span class="p">${</span><span class="nx">keyword</span><span class="p">}</span><span class="s2">&lt;/span&gt;`</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">keyword</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">null</span><span class="dl">'</span><span class="p">)</span>
      <span class="k">return</span> <span class="s2">`&lt;span class="text-zinc-500"&gt;null&lt;/span&gt;`</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">num</span><span class="p">)</span> <span class="k">return</span> <span class="s2">`&lt;span class="text-blue-400"&gt;</span><span class="p">${</span><span class="nx">num</span><span class="p">}</span><span class="s2">&lt;/span&gt;`</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">bracket</span><span class="p">)</span> <span class="p">{</span> <span class="cm">/* depth-based coloring */</span> <span class="p">}</span>
    <span class="c1">// ...</span>
  <span class="p">}</span>
<span class="p">);</span>
</code></pre>

</div>



<p>The key insight for distinguishing keys from string values: <strong>keys are followed by <code>:</code></strong>. The regex capture group <code>("...")\s*:</code> separates them cleanly.</p>

<h3>
  
  
  Depth-Based Bracket Coloring
</h3>

<p>To improve readability of deeply nested JSON, <code>{}</code> and <code>[]</code> get colored by nesting depth, cycling through 5 colors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">BRACKET_COLORS</span> <span class="o">=</span> <span class="p">[</span>
  <span class="dl">'</span><span class="s1">text-yellow-300</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">text-pink-400</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">text-blue-400</span><span class="dl">'</span><span class="p">,</span>
  <span class="dl">'</span><span class="s1">text-green-400</span><span class="dl">'</span><span class="p">,</span>  <span class="dl">'</span><span class="s1">text-orange-400</span><span class="dl">'</span><span class="p">,</span>
<span class="p">];</span>

<span class="c1">// Opening bracket: use color at current depth, then depth++</span>
<span class="c1">// Closing bracket: depth-- then use color at that depth</span>
</code></pre>

</div>



<h3>
  
  
  Indent Guides (Vertical Lines)
</h3>

<p>Vertical lines at each indent level make nesting depth visually obvious:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">indentSpaces</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">guideHtml</span> <span class="o">+=</span> <span class="s2">`&lt;span style="display:inline-block;width:1ch;position:relative"&gt;
    &lt;span style="position:absolute;top:0;bottom:0;left:0.45ch;
      border-left:1px solid #71717a"&gt;&lt;/span&gt; &lt;/span&gt;`</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Notice the <strong>inline styles</strong> here. That's important -- explained in the next section.</p>

<h2>
  
  
  3. Tailwind CSS + dangerouslySetInnerHTML: The Purge Problem
</h2>

<p>The syntax-highlighted HTML is injected via <code>dangerouslySetInnerHTML</code>. If you use Tailwind classes in that HTML, <strong>they get purged at build time and have no effect</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// Bad: doesn't work after build</span>
<span class="s2">`&lt;span class="border-l border-zinc-500"&gt;...&lt;/span&gt;`</span>
</code></pre>

</div>



<p>Tailwind v4 detects classes by scanning source files, but it can't find classes inside strings passed to <code>dangerouslySetInnerHTML</code>.</p>

<p>The fix is straightforward: use <strong>inline styles</strong> for injected HTML.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// Good: inline styles always apply</span>
<span class="s2">`&lt;span style="border-left:1px solid #71717a"&gt;...&lt;/span&gt;`</span>
</code></pre>

</div>



<p>The syntax highlighting color classes (<code>text-sky-300</code>, etc.) survive the purge because they're also used in the JSX return. Only the indent guide lines needed inline styles.</p>

<blockquote>
<p><strong>Lesson learned</strong>: If you're using Tailwind classes in <code>dangerouslySetInnerHTML</code> content, either add them to your <code>safelist</code> or use inline styles.</p>

<p>All of this -- <strong>syntax highlighting, indent guides, and bracket coloring</strong> -- implemented with zero external libraries. See it in action at <a href="https://json.puremark.app" rel="noopener noreferrer">json.puremark.app</a>.</p>
</blockquote>

<h2>
  
  
  4. Error Line Numbers -- Cross-Browser Differences
</h2>

<p>Extracting line and column numbers from <code>JSON.parse</code> errors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">msg</span> <span class="o">=</span> <span class="p">(</span><span class="nx">e</span> <span class="k">as</span> <span class="nb">Error</span><span class="p">).</span><span class="nx">message</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">lineMatch</span> <span class="o">=</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">match</span><span class="p">(</span><span class="sr">/line </span><span class="se">(\d</span><span class="sr">+</span><span class="se">)</span><span class="sr">/i</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">colMatch</span> <span class="o">=</span> <span class="nx">msg</span><span class="p">.</span><span class="nf">match</span><span class="p">(</span><span class="sr">/column </span><span class="se">(\d</span><span class="sr">+</span><span class="se">)</span><span class="sr">/i</span><span class="p">);</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>But <strong>error message formats differ across browsers</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Browser</th>
<th>Error message example</th>
<th>Line number</th>
</tr>
</thead>
<tbody>
<tr>
<td>Firefox</td>
<td><code>unexpected character at line 3 column 5</code></td>
<td>Extractable</td>
</tr>
<tr>
<td>Chrome</td>
<td><code>Unexpected token } in JSON at position 42</code></td>
<td><strong>Position only</strong></td>
</tr>
<tr>
<td>Safari</td>
<td><code>JSON Parse error: Expected '}'</code></td>
<td>None</td>
</tr>
</tbody>
</table></div>

<p>Firefox gives precise line and column numbers. Chrome only provides a character offset (<code>position</code>). For better Chrome support, you'd need to scan the input text and convert the position to a line number.</p>

<h2>
  
  
  Monorepo Structure
</h2>

<p>This JSON Formatter lives in the same monorepo as <a href="https://annotate.puremark.app" rel="noopener noreferrer">PureMark Annotate</a>, an image annotation tool:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>puremark/
  apps/
    annotate/         # Image annotation tool
    json-formatter/   # JSON Formatter (this article)
  packages/
    ui/               # Shared UI components (future)
  package.json        # npm workspaces
</code></pre>

</div>



<p>Each app gets its own Cloudflare Pages project with a targeted build command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># json-formatter build config
Build command: cd apps/json-formatter &amp;&amp; npm install &amp;&amp; npm run build
Build output: apps/json-formatter/dist
</code></pre>

</div>



<p>Domains are separated by subdomain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>puremark.app               -&gt; Landing page
annotate.puremark.app      -&gt; Image annotation tool
json.puremark.app          -&gt; JSON Formatter
</code></pre>

</div>



<h2>
  
  
  Summary
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Challenge</th>
<th>Solution</th>
</tr>
</thead>
<tbody>
<tr>
<td>Clipboard auto-read</td>
<td>Clipboard API (on mount + visibilitychange) + paste event fallback</td>
</tr>
<tr>
<td>JSON detection</td>
<td>First-character heuristic (<code>{</code>, <code>[</code>, <code>"</code>). Don't gate on <code>JSON.parse</code> success</td>
</tr>
<tr>
<td>Syntax highlighting</td>
<td>Single regex classifying all tokens. No external libraries</td>
</tr>
<tr>
<td>Tailwind + innerHTML</td>
<td>Use inline styles for injected HTML</td>
</tr>
<tr>
<td>Error line numbers</td>
<td>
<code>/line (\d+)/i</code> extraction. Chrome only gives position (room for improvement)</td>
</tr>
</tbody>
</table></div>

<p>When you build developer tools that solve your own pain, implementation decisions come fast. When you're unsure about a feature, ask: "Would I use this every day?" That keeps things focused.</p>

<p><strong>PureMark JSON Formatter</strong> -- copy JSON, open a tab, done. Zero server transmission.<br>
<a href="https://json.puremark.app" rel="noopener noreferrer">https://json.puremark.app</a></p>




<h2>
  
  
  References
</h2>

<ul>
<li><a href="https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/readText" rel="noopener noreferrer">Clipboard API - MDN Web Docs</a></li>
<li><a href="https://labs.watchtowr.com/" rel="noopener noreferrer">watchTowr Labs: JSON Formatter Security Leak (2025)</a></li>
<li><a href="https://annotate.puremark.app" rel="noopener noreferrer">PureMark Annotate -- Browser-based image annotation tool</a></li>
</ul>

