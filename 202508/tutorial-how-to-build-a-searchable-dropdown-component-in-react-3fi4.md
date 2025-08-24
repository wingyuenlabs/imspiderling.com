---
Title: 🧑‍🏫 Tutorial: How to Build a Searchable Dropdown Component in React
Description: 
Author: Md Shahjalal
Date: 2025-08-24T18:21:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>In this comprehensive tutorial, you'll learn how to build a <strong>fully accessible, debounced, searchable dropdown component</strong> in React using hooks, TypeScript, and best practices. </p>

<p>This component is perfect for autocompleting users, products, or any data fetched from an API.</p>

<p>We'll walk through each part of the code step-by-step, explain what it does, and why it’s structured that way.</p>

<h2>
  
  
  ✅ Final Features
</h2>

<ul>
<li>🔍 <strong>Search-as-you-type</strong> with debounce</li>
<li>⏱️ <strong>Debounced input</strong> to avoid excessive API calls</li>
<li>🌐 <strong>Async data fetching</strong>
</li>
<li>📱 <strong>Accessible</strong> (ARIA attributes, keyboard navigation)</li>
<li>⌨️ <strong>Keyboard navigation</strong> (arrow keys, Enter, Escape)</li>
<li>❌ <strong>Click outside to close</strong>
</li>
<li>🎨 <strong>Customizable rendering</strong>
</li>
<li>🛑 <strong>Error handling and loading states</strong>
</li>
<li>💬 <strong>No results / empty state</strong>
</li>
<li>📏 <strong>Controlled minimum query length</strong>
</li>
</ul>




<h2>
  
  
  🛠️ Prerequisites
</h2>

<p>Before starting, ensure you have:</p>

<ul>
<li>Basic knowledge of <strong>React</strong> and <strong>TypeScript</strong>
</li>
<li>A React project set uu</li>
<li>Some familiarity with React hooks (<code>useState</code>, <code>useEffect</code>, <code>useCallback</code>, <code>useMemo</code>, <code>useRef</code>)</li>
</ul>

<h2>
  
  
  📁 Step 1: Create the Component File
</h2>

<p>Create a new file: <code>SearchableDropdown.tsx</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>src/components/SearchableDropdown.tsx
</code></pre>

</div>



<h2>
  
  
  🧩 Step 2: Import Dependencies
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="nx">React</span><span class="p">,</span> <span class="p">{</span>
  <span class="nx">useState</span><span class="p">,</span>
  <span class="nx">useEffect</span><span class="p">,</span>
  <span class="nx">useRef</span><span class="p">,</span>
  <span class="nx">useCallback</span><span class="p">,</span>
  <span class="nx">useMemo</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>
</code></pre>

</div>



<p>We’re using several React hooks:</p>

<ul>
<li>
<code>useState</code>: manage component state</li>
<li>
<code>useEffect</code>: side effects (fetching, event listeners)</li>
<li>
<code>useRef</code>: access DOM elements</li>
<li>
<code>useCallback</code>: memoize functions</li>
<li>
<code>useMemo</code>: memoize expensive computations</li>
</ul>

<h2>
  
  
  🔁 Step 3: Create the <code>useDebounce</code> Hook
</h2>

<p>This custom hook delays updates to the search query so we don’t call the API on every keystroke.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">useDebounce</span> <span class="o">=</span> <span class="p">(</span><span class="nx">value</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">delay</span><span class="p">:</span> <span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">debouncedValue</span><span class="p">,</span> <span class="nx">setDebouncedValue</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="nx">value</span><span class="p">);</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nf">setDebouncedValue</span><span class="p">(</span><span class="nx">value</span><span class="p">),</span> <span class="nx">delay</span><span class="p">);</span>
    <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">clearTimeout</span><span class="p">(</span><span class="nx">handler</span><span class="p">);</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">value</span><span class="p">,</span> <span class="nx">delay</span><span class="p">]);</span>

  <span class="k">return</span> <span class="nx">debouncedValue</span><span class="p">;</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  🔍 How It Works:
</h3>

<ul>
<li>Every time <code>value</code> changes, restart a 300ms timer.</li>
<li>Only after 300ms without changes will <code>debouncedValue</code> update.</li>
<li>Prevents spamming the API during fast typing.</li>
</ul>

<blockquote>
<p>💡 Tip: You can reuse this hook anywhere you need debouncing!</p>
</blockquote>




<h2>
  
  
  📦 Step 4: Define the Component Props
</h2>

<p>We define a generic interface for flexibility.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kr">interface</span> <span class="nx">SearchableDropdownProps</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="na">fetchOptions</span><span class="p">:</span> <span class="p">(</span><span class="na">query</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">T</span><span class="p">[]</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="nl">displayValue</span><span class="p">:</span> <span class="p">(</span><span class="na">item</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">renderOption</span><span class="p">?:</span> <span class="p">(</span><span class="na">item</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">React</span><span class="p">.</span><span class="nx">ReactNode</span><span class="p">;</span>
  <span class="nl">placeholder</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">minQueryLength</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  🔎 Explanation:
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Prop</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>fetchOptions</code></td>
<td>Async function to fetch data based on query</td>
</tr>
<tr>
<td><code>displayValue</code></td>
<td>Function to extract display string from item</td>
</tr>
<tr>
<td>
<code>renderOption</code> <em>(optional)</em>
</td>
<td>Custom JSX for each dropdown option</td>
</tr>
<tr>
<td>
<code>placeholder</code> <em>(optional)</em>
</td>
<td>Input placeholder text</td>
</tr>
<tr>
<td>
<code>minQueryLength</code> <em>(optional)</em>
</td>
<td>Minimum characters before search starts</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  🎯 Step 5: Forward Ref &amp; Generic Typing
</h2>

<p>We use <code>React.forwardRef</code> so parent components can access the <code>&lt;input&gt;</code> element (e.g., for focusing).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">SearchableDropdown</span> <span class="o">=</span> <span class="nx">React</span><span class="p">.</span><span class="nx">forwardRef</span><span class="o">&lt;</span>
  <span class="nx">HTMLInputElement</span><span class="p">,</span>
  <span class="nx">SearchableDropdownProps</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span>
<span class="o">&gt;</span><span class="p">(</span>
  <span class="p">(</span>
    <span class="p">{</span>
      <span class="nx">fetchOptions</span><span class="p">,</span>
      <span class="nx">displayValue</span><span class="p">,</span>
      <span class="nx">renderOption</span><span class="p">,</span>
      <span class="nx">placeholder</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">Search...</span><span class="dl">'</span><span class="p">,</span>
      <span class="nx">minQueryLength</span> <span class="o">=</span> <span class="mi">2</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="nx">ref</span>
  <span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// component logic here</span>
  <span class="p">}</span>
<span class="p">);</span>
</code></pre>

</div>



<blockquote>
<p>🔒 Note: We use <code>any</code> temporarily. For full type safety, we’ll improve this later.</p>
</blockquote>

<h2>
  
  
  🧠 Step 6: Manage Component State
</h2>

<p>Inside the component, define all necessary state:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="p">[</span><span class="nx">query</span><span class="p">,</span> <span class="nx">setQuery</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">''</span><span class="p">);</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">options</span><span class="p">,</span> <span class="nx">setOptions</span><span class="p">]</span> <span class="o">=</span> <span class="nx">useState</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span><span class="p">([]);</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">loading</span><span class="p">,</span> <span class="nx">setLoading</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">error</span><span class="p">,</span> <span class="nx">setError</span><span class="p">]</span> <span class="o">=</span> <span class="nx">useState</span><span class="o">&lt;</span><span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">showDropdown</span><span class="p">,</span> <span class="nx">setShowDropdown</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">focusedIndex</span><span class="p">,</span> <span class="nx">setFocusedIndex</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">dropdownRef</span> <span class="o">=</span> <span class="nx">useRef</span><span class="o">&lt;</span><span class="nx">HTMLDivElement</span><span class="o">&gt;</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
</code></pre>

</div>



<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>State</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>query</code></td>
<td>Current input value</td>
</tr>
<tr>
<td><code>options</code></td>
<td>Fetched results</td>
</tr>
<tr>
<td><code>loading</code></td>
<td>Show loading indicator</td>
</tr>
<tr>
<td><code>error</code></td>
<td>Handle fetch errors</td>
</tr>
<tr>
<td><code>showDropdown</code></td>
<td>Toggle dropdown visibility</td>
</tr>
<tr>
<td><code>focusedIndex</code></td>
<td>Track keyboard-highlighted option</td>
</tr>
<tr>
<td><code>dropdownRef</code></td>
<td>Detect clicks outside</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  ⏳ Step 7: Debounce the Query
</h2>

<p>Use the custom hook to debounce the search:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">debouncedQuery</span> <span class="o">=</span> <span class="nf">useDebounce</span><span class="p">(</span><span class="nx">query</span><span class="p">,</span> <span class="mi">300</span><span class="p">);</span>
</code></pre>

</div>



<p>Now <code>debouncedQuery</code> only updates 300ms after user stops typing.</p>

<h2>
  
  
  🌐 Step 8: Fetch Options on Debounced Query
</h2>

<p>Use <code>useEffect</code> to trigger API calls when <code>debouncedQuery</code> changes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">debouncedQuery</span><span class="p">.</span><span class="nf">trim</span><span class="p">().</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="nx">minQueryLength</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">setOptions</span><span class="p">([]);</span>
    <span class="nf">setShowDropdown</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="nf">setFocusedIndex</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">fetchData</span> <span class="o">=</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">setLoading</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="nf">setError</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
    <span class="k">try</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchOptions</span><span class="p">(</span><span class="nx">debouncedQuery</span><span class="p">.</span><span class="nf">trim</span><span class="p">());</span>
      <span class="nf">setOptions</span><span class="p">(</span><span class="nx">results</span> <span class="o">||</span> <span class="p">[]);</span>
      <span class="nf">setShowDropdown</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="na">err</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">err</span><span class="p">);</span>
      <span class="nf">setError</span><span class="p">(</span><span class="nx">err</span><span class="p">.</span><span class="nx">message</span> <span class="o">||</span> <span class="dl">'</span><span class="s1">Failed to load options</span><span class="dl">'</span><span class="p">);</span>
      <span class="nf">setOptions</span><span class="p">([]);</span>
    <span class="p">}</span> <span class="k">finally</span> <span class="p">{</span>
      <span class="nf">setLoading</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
      <span class="nf">setFocusedIndex</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">};</span>

  <span class="nf">fetchData</span><span class="p">();</span>
<span class="p">},</span> <span class="p">[</span><span class="nx">debouncedQuery</span><span class="p">,</span> <span class="nx">fetchOptions</span><span class="p">,</span> <span class="nx">minQueryLength</span><span class="p">]);</span>
</code></pre>

</div>



<h3>
  
  
  ✅ Logic Flow:
</h3>

<ol>
<li>If query too short → clear results and hide dropdown</li>
<li>Otherwise → fetch results</li>
<li>Handle success/error</li>
<li>Always reset focused index</li>
</ol>

<blockquote>
<p>⚠️ Dependencies include <code>fetchOptions</code> and <code>minQueryLength</code> to avoid stale closures.</p>
</blockquote>

<h2>
  
  
  🖱️ Step 9: Close Dropdown on Outside Click
</h2>

<p>Add a click-outside listener:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">handleClickOutside</span> <span class="o">=</span> <span class="p">(</span><span class="na">e</span><span class="p">:</span> <span class="nx">MouseEvent</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span>
      <span class="nx">dropdownRef</span><span class="p">.</span><span class="nx">current</span> <span class="o">&amp;&amp;</span>
      <span class="o">!</span><span class="nx">dropdownRef</span><span class="p">.</span><span class="nx">current</span><span class="p">.</span><span class="nf">contains</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span> <span class="k">as</span> <span class="nx">Node</span><span class="p">)</span>
    <span class="p">)</span> <span class="p">{</span>
      <span class="nf">setShowDropdown</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">};</span>
  <span class="nb">document</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">mousedown</span><span class="dl">'</span><span class="p">,</span> <span class="nx">handleClickOutside</span><span class="p">);</span>
  <span class="k">return </span><span class="p">()</span> <span class="o">=&gt;</span>
    <span class="nb">document</span><span class="p">.</span><span class="nf">removeEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">mousedown</span><span class="dl">'</span><span class="p">,</span> <span class="nx">handleClickOutside</span><span class="p">);</span>
<span class="p">},</span> <span class="p">[]);</span>
</code></pre>

</div>



<p>This ensures dropdown closes when clicking elsewhere.</p>

<h2>
  
  
  ✅ Step 10: Handle Option Selection
</h2>

<p>Define a stable callback with <code>useCallback</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">handleSelect</span> <span class="o">=</span> <span class="nf">useCallback</span><span class="p">(</span>
  <span class="p">(</span><span class="nx">item</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">setQuery</span><span class="p">(</span><span class="nf">displayValue</span><span class="p">(</span><span class="nx">item</span><span class="p">));</span>
    <span class="nf">setShowDropdown</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="nf">setOptions</span><span class="p">([]);</span>
    <span class="nf">setFocusedIndex</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">);</span>
  <span class="p">},</span>
  <span class="p">[</span><span class="nx">displayValue</span><span class="p">]</span>
<span class="p">);</span>
</code></pre>

</div>



<p>Sets the input to selected item and hides the list.</p>

<h2>
  
  
  ⌨️ Step 11: Keyboard Navigation
</h2>

<p>Handle arrow keys, Enter, and Escape:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">handleKeyDown</span> <span class="o">=</span> <span class="p">(</span><span class="nx">e</span><span class="p">:</span> <span class="nx">React</span><span class="p">.</span><span class="nx">KeyboardEvent</span><span class="o">&lt;</span><span class="nx">HTMLInputElement</span><span class="o">&gt;</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">showDropdown</span> <span class="o">||</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

  <span class="k">switch </span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">key</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">ArrowDown</span><span class="dl">'</span><span class="p">:</span>
      <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
      <span class="nf">setFocusedIndex</span><span class="p">(</span><span class="nx">prev</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="nx">prev</span> <span class="o">+</span> <span class="mi">1</span><span class="p">)</span> <span class="o">%</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
      <span class="k">break</span><span class="p">;</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">ArrowUp</span><span class="dl">'</span><span class="p">:</span>
      <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
      <span class="nf">setFocusedIndex</span><span class="p">(</span><span class="nx">prev</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="nx">prev</span> <span class="o">-</span> <span class="mi">1</span> <span class="o">+</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span><span class="p">)</span> <span class="o">%</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
      <span class="k">break</span><span class="p">;</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">Enter</span><span class="dl">'</span><span class="p">:</span>
      <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">focusedIndex</span> <span class="o">&gt;=</span> <span class="mi">0</span><span class="p">)</span> <span class="nf">handleSelect</span><span class="p">(</span><span class="nx">options</span><span class="p">[</span><span class="nx">focusedIndex</span><span class="p">]);</span>
      <span class="k">break</span><span class="p">;</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">Escape</span><span class="dl">'</span><span class="p">:</span>
      <span class="nf">setShowDropdown</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
      <span class="k">break</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<ul>
<li>
<code>ArrowDown</code> / <code>ArrowUp</code>: Cycle through options</li>
<li>
<code>Enter</code>: Select highlighted option</li>
<li>
<code>Escape</code>: Close dropdown</li>
</ul>

<blockquote>
<p><code>% options.length</code> enables circular navigation.</p>
</blockquote>

<h2>
  
  
  🧾 Step 12: Render Options Efficiently
</h2>

<p>Use <code>useMemo</code> to prevent unnecessary re-renders:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">renderedOptions</span> <span class="o">=</span> <span class="nf">useMemo</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">loading</span><span class="p">)</span>
    <span class="k">return</span> <span class="p">&lt;</span><span class="nt">li</span> <span class="na">className</span><span class="p">=</span><span class="s">'p-2 text-sm text-gray-500 italic'</span><span class="p">&gt;</span>Loading...<span class="p">&lt;/</span><span class="nt">li</span><span class="p">&gt;;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="k">return</span> <span class="p">&lt;</span><span class="nt">li</span> <span class="na">className</span><span class="p">=</span><span class="s">'p-2 text-sm text-red-500'</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">error</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">li</span><span class="p">&gt;;</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">loading</span> <span class="o">&amp;&amp;</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span> <span class="o">&amp;&amp;</span> <span class="nx">query</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;=</span> <span class="nx">minQueryLength</span><span class="p">)</span>
    <span class="k">return</span> <span class="p">&lt;</span><span class="nt">li</span> <span class="na">className</span><span class="p">=</span><span class="s">'p-2 text-sm text-gray-500'</span><span class="p">&gt;</span>No results found<span class="p">&lt;/</span><span class="nt">li</span><span class="p">&gt;;</span>

  <span class="k">return</span> <span class="nx">options</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">option</span><span class="p">,</span> <span class="nx">index</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">isSelected</span> <span class="o">=</span> <span class="nx">index</span> <span class="o">===</span> <span class="nx">focusedIndex</span><span class="p">;</span>
    <span class="k">return </span><span class="p">(</span>
      <span class="p">&lt;</span><span class="nt">li</span>
        <span class="na">key</span><span class="p">=</span><span class="si">{</span><span class="nx">option</span><span class="p">.</span><span class="nx">id</span> <span class="o">||</span> <span class="nx">index</span><span class="si">}</span>
        <span class="na">role</span><span class="p">=</span><span class="s">'option'</span>
        <span class="na">aria-selected</span><span class="p">=</span><span class="si">{</span><span class="nx">isSelected</span><span class="si">}</span>
        <span class="na">onClick</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">handleSelect</span><span class="p">(</span><span class="nx">option</span><span class="p">)</span><span class="si">}</span>
        <span class="na">onMouseEnter</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">setFocusedIndex</span><span class="p">(</span><span class="nx">index</span><span class="p">)</span><span class="si">}</span>
        <span class="na">className</span><span class="p">=</span><span class="si">{</span><span class="s2">`p-2 cursor-pointer hover:bg-gray-100 </span><span class="p">${</span>
          <span class="nx">isSelected</span> <span class="p">?</span> <span class="dl">'</span><span class="s1">bg-blue-50</span><span class="dl">'</span> <span class="p">:</span> <span class="dl">''</span>
        <span class="p">}</span><span class="s2"> </span><span class="p">${</span><span class="nx">index</span> <span class="o">&lt;</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="mi">1</span> <span class="p">?</span> <span class="dl">'</span><span class="s1">border-b border-gray-100</span><span class="dl">'</span> <span class="p">:</span> <span class="dl">''</span><span class="p">}</span><span class="s2">`</span><span class="si">}</span>
      <span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">renderOption</span> <span class="p">?</span> <span class="nf">renderOption</span><span class="p">(</span><span class="nx">option</span><span class="p">)</span> <span class="p">:</span> <span class="nf">displayValue</span><span class="p">(</span><span class="nx">option</span><span class="p">)</span><span class="si">}</span>
        <span class="si">{</span><span class="nx">option</span><span class="p">.</span><span class="nx">email</span> <span class="o">&amp;&amp;</span> <span class="p">(</span>
          <span class="p">&lt;</span><span class="nt">span</span> <span class="na">className</span><span class="p">=</span><span class="s">'text-gray-500 text-xs'</span><span class="p">&gt;</span> (<span class="si">{</span><span class="nx">option</span><span class="p">.</span><span class="nx">email</span><span class="si">}</span>)<span class="p">&lt;/</span><span class="nt">span</span><span class="p">&gt;</span>
        <span class="p">)</span><span class="si">}</span>
      <span class="p">&lt;/</span><span class="nt">li</span><span class="p">&gt;</span>
    <span class="p">);</span>
  <span class="p">});</span>
<span class="p">},</span> <span class="p">[</span>
  <span class="nx">loading</span><span class="p">,</span>
  <span class="nx">error</span><span class="p">,</span>
  <span class="nx">options</span><span class="p">,</span>
  <span class="nx">focusedIndex</span><span class="p">,</span>
  <span class="nx">query</span><span class="p">,</span>
  <span class="nx">renderOption</span><span class="p">,</span>
  <span class="nx">displayValue</span><span class="p">,</span>
  <span class="nx">handleSelect</span><span class="p">,</span>
  <span class="nx">minQueryLength</span><span class="p">,</span>
<span class="p">]);</span>
</code></pre>

</div>



<h3>
  
  
  🎨 Notes:
</h3>

<ul>
<li>Uses ARIA roles for accessibility</li>
<li>Highlights hovered/focused item</li>
<li>Falls back to <code>index</code> if no <code>id</code>
</li>
<li>Shows email if available (could be removed or generalized)</li>
</ul>

<h2>
  
  
  🖼️ Step 13: Render the UI
</h2>

<p>Finally, return the JSX:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">return </span><span class="p">(</span>
  <span class="p">&lt;</span><span class="nt">div</span> <span class="na">className</span><span class="p">=</span><span class="s">'w-72 relative'</span> <span class="na">ref</span><span class="p">=</span><span class="si">{</span><span class="nx">dropdownRef</span><span class="si">}</span><span class="p">&gt;</span>
    <span class="p">&lt;</span><span class="nt">input</span>
      <span class="na">ref</span><span class="p">=</span><span class="si">{</span><span class="nx">ref</span><span class="si">}</span>
      <span class="na">type</span><span class="p">=</span><span class="s">'text'</span>
      <span class="na">placeholder</span><span class="p">=</span><span class="si">{</span><span class="nx">placeholder</span><span class="si">}</span>
      <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">query</span><span class="si">}</span>
      <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="nx">e</span> <span class="o">=&gt;</span> <span class="nf">setQuery</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)</span><span class="si">}</span>
      <span class="na">onFocus</span><span class="p">=</span><span class="si">{</span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">query</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;=</span> <span class="nx">minQueryLength</span> <span class="o">&amp;&amp;</span> <span class="nx">options</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span>
          <span class="nf">setShowDropdown</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
      <span class="p">}</span><span class="si">}</span>
      <span class="na">onKeyDown</span><span class="p">=</span><span class="si">{</span><span class="nx">handleKeyDown</span><span class="si">}</span>
      <span class="na">aria-autocomplete</span><span class="p">=</span><span class="s">'list'</span>
      <span class="na">aria-expanded</span><span class="p">=</span><span class="si">{</span><span class="nx">showDropdown</span><span class="si">}</span>
      <span class="na">aria-controls</span><span class="p">=</span><span class="s">'dropdown-listbox'</span>
      <span class="na">className</span><span class="p">=</span><span class="s">'w-full p-2 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-2 focus:ring-blue-400'</span>
    <span class="p">/&gt;</span>

    <span class="si">{</span><span class="nx">showDropdown</span> <span class="o">&amp;&amp;</span> <span class="p">(</span>
      <span class="p">&lt;</span><span class="nt">ul</span>
        <span class="na">id</span><span class="p">=</span><span class="s">'dropdown-listbox'</span>
        <span class="na">role</span><span class="p">=</span><span class="s">'listbox'</span>
        <span class="na">aria-label</span><span class="p">=</span><span class="s">'Search results'</span>
        <span class="na">className</span><span class="p">=</span><span class="s">'absolute top-11 left-0 right-0 bg-white border border-gray-300 rounded-md shadow-md max-h-48 overflow-y-auto z-10'</span>
      <span class="p">&gt;</span>
        <span class="si">{</span><span class="nx">renderedOptions</span><span class="si">}</span>
      <span class="p">&lt;/</span><span class="nt">ul</span><span class="p">&gt;</span>
    <span class="p">)</span><span class="si">}</span>
  <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  🔍 Accessibility Highlights:
</h3>

<ul>
<li>
<code>aria-autocomplete="list"</code>: indicates suggestions</li>
<li>
<code>aria-expanded</code>: shows if dropdown is open</li>
<li>
<code>aria-controls</code>: links input to listbox</li>
<li>
<code>role="listbox"</code> and <code>option"</code>: proper ARIA semantics</li>
</ul>

<blockquote>
<p>💡 The dropdown appears below the input (<code>top-11</code> ≈ input height + padding)</p>
</blockquote>

<h2>
  
  
  🏷️ Step 14: Set Display Name
</h2>

<p>Helpful for debugging in React DevTools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="nx">SearchableDropdown</span><span class="p">.</span><span class="nx">displayName</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">SearchableDropdown</span><span class="dl">'</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  🚀 Step 15: Export the Component
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">export</span> <span class="k">default</span> <span class="nx">SearchableDropdown</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  🧪 Step 16: Example Usage
</h2>

<p>Here’s how to use it in a parent component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="nx">SearchableDropdown</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./SearchableDropdown</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Example data type</span>
<span class="kr">interface</span> <span class="nx">User</span> <span class="p">{</span>
  <span class="nl">id</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">email</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="nx">App</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">fetchUsers</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="na">query</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">User</span><span class="p">[]</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`/api/users?q=</span><span class="p">${</span><span class="nf">encodeURIComponent</span><span class="p">(</span><span class="nx">query</span><span class="p">)}</span><span class="s2">`</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
  <span class="p">};</span>

  <span class="kd">const</span> <span class="nx">inputRef</span> <span class="o">=</span> <span class="nx">useRef</span><span class="o">&lt;</span><span class="nx">HTMLInputElement</span><span class="o">&gt;</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nt">div</span> <span class="na">className</span><span class="p">=</span><span class="s">"p-8"</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nt">h1</span><span class="p">&gt;</span>Search Users<span class="p">&lt;/</span><span class="nt">h1</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">SearchableDropdown</span>
        <span class="na">ref</span><span class="p">=</span><span class="si">{</span><span class="nx">inputRef</span><span class="si">}</span>
        <span class="na">fetchOptions</span><span class="p">=</span><span class="si">{</span><span class="nx">fetchUsers</span><span class="si">}</span>
        <span class="na">displayValue</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">user</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="si">}</span>
        <span class="na">renderOption</span><span class="p">=</span><span class="si">{</span><span class="p">(</span><span class="nx">user</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
          <span class="p">&lt;</span><span class="nt">div</span><span class="p">&gt;</span>
            <span class="p">&lt;</span><span class="nt">strong</span><span class="p">&gt;</span><span class="si">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">strong</span><span class="p">&gt;</span>
            <span class="p">&lt;</span><span class="nt">span</span> <span class="na">className</span><span class="p">=</span><span class="s">"text-gray-600"</span><span class="p">&gt;</span> — <span class="si">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="si">}</span><span class="p">&lt;/</span><span class="nt">span</span><span class="p">&gt;</span>
          <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
        <span class="p">)</span><span class="si">}</span>
        <span class="na">placeholder</span><span class="p">=</span><span class="s">"Search users..."</span>
        <span class="na">minQueryLength</span><span class="p">=</span><span class="si">{</span><span class="mi">2</span><span class="si">}</span>
      <span class="p">/&gt;</span>
    <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  ✅ Bonus: Improve Type Safety (Optional)
</h2>

<p>Replace <code>any</code> with proper generics:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">SearchableDropdown</span> <span class="o">=</span> <span class="nx">React</span><span class="p">.</span><span class="nx">forwardRef</span><span class="o">&lt;</span>
  <span class="nx">HTMLInputElement</span><span class="p">,</span>
  <span class="nx">SearchableDropdownProps</span><span class="o">&lt;</span><span class="nx">unknown</span><span class="o">&gt;</span>
<span class="o">&gt;</span><span class="p">(({</span> <span class="nx">fetchOptions</span><span class="p">,</span> <span class="nx">displayValue</span><span class="p">,</span> <span class="nx">renderOption</span><span class="p">,</span> <span class="nx">placeholder</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">Search...</span><span class="dl">'</span><span class="p">,</span> <span class="nx">minQueryLength</span> <span class="o">=</span> <span class="mi">2</span> <span class="p">},</span> <span class="nx">ref</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// ... same logic</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Or better yet, keep <code>T</code> generic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="kd">const</span> <span class="nx">SearchableDropdown</span> <span class="o">=</span> <span class="nx">React</span><span class="p">.</span><span class="nf">forwardRef</span><span class="p">(</span>
  <span class="p">&lt;</span><span class="nc">T</span> <span class="na">extends</span> <span class="na">unknown</span><span class="p">&gt;</span>(
    props: SearchableDropdownProps<span class="p">&lt;</span><span class="nc">T</span><span class="p">&gt;</span>,
    ref: React.ForwardedRef<span class="p">&lt;</span><span class="nc">HTMLInputElement</span><span class="p">&gt;</span>
  ) =&gt; <span class="si">{</span> <span class="p">...</span> <span class="si">}</span>
) as <span class="p">&lt;</span><span class="nc">T</span><span class="p">&gt;</span>(
  props: SearchableDropdownProps<span class="p">&lt;</span><span class="nc">T</span><span class="p">&gt;</span> <span class="err">&amp;</span> React.RefAttributes<span class="p">&lt;</span><span class="nc">HTMLInputElement</span><span class="p">&gt;</span>
) =&gt; JSX.Element;
</code></pre>

</div>



<p>But this requires advanced typing — stick with <code>any</code> or <code>unknown</code> unless you need strict typing.</p>

<p>You’ve now built a <strong>production-ready searchable dropdown</strong> with:</p>

<ul>
<li>Debouncing</li>
<li>Async loading</li>
<li>Full keyboard + mouse support</li>
<li>Error handling</li>
<li>Accessibility</li>
<li>Reusability</li>
</ul>

<p>It's modular, clean, and scalable — ideal for forms, user searches, product selectors, and more.</p>

<p>Happy coding! 💻✨</p>

