---
Title: The TypeScript Features I Wish I'd Learned Sooner
Description: 
Author: Tarun Moorjani
Date: 2026-01-08T21:52:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been writing TypeScript for five years, and I'm still having "wait, it can do THAT?" moments. Not the fancy type gymnastics that make you feel clever, but the features that genuinely make everyday coding easier, faster, and less frustrating.</p>

<p>These aren't obscure tricks. They're practical features that have been sitting in my editor this whole time, waiting for me to discover them. Each one made me think: "Where has this been all my life?"</p>

<p>If you're like me—comfortable with TypeScript basics but sensing there's more—this is for you.</p>

<h2>
  
  
  1. The <code>satisfies</code> Operator: Type Checking Without Losing Precision
</h2>

<p>For years, I wrestled with a frustrating trade-off: I could either get type safety OR preserve literal types, but not both.</p>

<h3>
  
  
  The Old Way: Choose Your Pain
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="na">retries</span><span class="p">:</span> <span class="mi">3</span>
<span class="p">};</span>
<span class="c1">// Problem: TypeScript infers `apiUrl: string` instead of the literal</span>
<span class="c1">// If I typo the URL, no autocomplete helps me fix it</span>
</code></pre>

</div>



<p>Okay, let's add a type annotation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">timeout</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">retries</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">config</span><span class="p">:</span> <span class="nx">Config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="na">retries</span><span class="p">:</span> <span class="mi">3</span>
<span class="p">};</span>
<span class="c1">// Problem: Still just `string` and `number`, literal types are gone</span>
</code></pre>

</div>



<p>What about <code>as const</code>?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="na">retries</span><span class="p">:</span> <span class="mi">3</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>
<span class="c1">// Problem: Now everything is readonly, and I have no type checking</span>
<span class="c1">// I could add a typo field and TypeScript won't complain</span>
</code></pre>

</div>



<h3>
  
  
  The New Way: <code>satisfies</code> to the Rescue
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">timeout</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">retries</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="na">retries</span><span class="p">:</span> <span class="mi">3</span>
<span class="p">}</span> <span class="nx">satisfies</span> <span class="nx">Config</span><span class="p">;</span>

<span class="c1">// Magic! TypeScript checks the structure matches Config,</span>
<span class="c1">// but keeps the literal types:</span>
<span class="c1">// - apiUrl is "https://api.example.com", not string</span>
<span class="c1">// - timeout is 5000, not number</span>
</code></pre>

</div>



<p>Now I get both type safety AND precise inference:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Type error! Extra property</span>
<span class="kd">const</span> <span class="nx">badConfig</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">timeout</span><span class="p">:</span> <span class="mi">5000</span><span class="p">,</span>
  <span class="na">retries</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>
  <span class="na">debugMode</span><span class="p">:</span> <span class="kc">true</span>  <span class="c1">// Error: Object literal may only specify known properties</span>
<span class="p">}</span> <span class="nx">satisfies</span> <span class="nx">Config</span><span class="p">;</span>

<span class="c1">// But I still have literals for autocomplete and refactoring</span>
<span class="k">if </span><span class="p">(</span><span class="nx">config</span><span class="p">.</span><span class="nx">apiUrl</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>  <span class="c1">// TypeScript knows this is always true</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Real-World Example: Route Definitions
</h3>

<p>This pattern shines for configuration objects:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Route</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">path</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GET</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">PUT</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">DELETE</span><span class="dl">"</span><span class="p">;</span>
  <span class="nl">handler</span><span class="p">:</span> <span class="p">(</span><span class="na">req</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="kr">any</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">routes</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">getUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/users/:id</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GET</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">handler</span><span class="p">:</span> <span class="p">(</span><span class="nx">req</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">userId</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">.</span><span class="nx">id</span> <span class="p">})</span>
  <span class="p">},</span>
  <span class="na">createUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/users</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">handler</span><span class="p">:</span> <span class="p">(</span><span class="nx">req</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">user</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span> <span class="p">})</span>
  <span class="p">}</span>
<span class="p">}</span> <span class="nx">satisfies</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">Route</span><span class="o">&gt;</span><span class="p">;</span>

<span class="c1">// Now you can do this with full autocomplete:</span>
<span class="kd">type</span> <span class="nx">RouteName</span> <span class="o">=</span> <span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">routes</span><span class="p">;</span>  <span class="c1">// "getUser" | "createUser"</span>
<span class="kd">const</span> <span class="nx">getUserPath</span> <span class="o">=</span> <span class="nx">routes</span><span class="p">.</span><span class="nx">getUser</span><span class="p">.</span><span class="nx">path</span><span class="p">;</span>  <span class="c1">// Type is "/users/:id", not string!</span>
</code></pre>

</div>



<p><strong>When to use it:</strong> Anytime you want to validate an object's structure without widening its types. Especially for configs, route definitions, and lookup tables.</p>

<h2>
  
  
  2. Const Assertions: The Feature I Ignored for Too Long
</h2>

<p>I saw <code>as const</code> in code and thought "that's just readonly, right?" Wrong. It's one of the most powerful features in TypeScript.</p>

<h3>
  
  
  What It Actually Does
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Without as const</span>
<span class="kd">const</span> <span class="nx">colors</span> <span class="o">=</span> <span class="p">[</span><span class="dl">"</span><span class="s2">red</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">green</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">blue</span><span class="dl">"</span><span class="p">];</span>
<span class="c1">// Type: string[]</span>

<span class="c1">// With as const</span>
<span class="kd">const</span> <span class="nx">colors</span> <span class="o">=</span> <span class="p">[</span><span class="dl">"</span><span class="s2">red</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">green</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">blue</span><span class="dl">"</span><span class="p">]</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>
<span class="c1">// Type: readonly ["red", "green", "blue"]</span>
</code></pre>

</div>



<p>"So what?" Past me would ask. "It's just readonly."</p>

<p>But watch what happens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Color</span> <span class="o">=</span> <span class="k">typeof</span> <span class="nx">colors</span><span class="p">[</span><span class="kr">number</span><span class="p">];</span>
<span class="c1">// Type: "red" | "green" | "blue"</span>

<span class="c1">// Now you have a union type derived from your data!</span>
<span class="kd">function</span> <span class="nf">setColor</span><span class="p">(</span><span class="nx">color</span><span class="p">:</span> <span class="nx">Color</span><span class="p">)</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>

<span class="nf">setColor</span><span class="p">(</span><span class="dl">"</span><span class="s2">red</span><span class="dl">"</span><span class="p">);</span>     <span class="c1">// ✓</span>
<span class="nf">setColor</span><span class="p">(</span><span class="dl">"</span><span class="s2">purple</span><span class="dl">"</span><span class="p">);</span>  <span class="c1">// ✗ Type error!</span>
</code></pre>

</div>



<h3>
  
  
  The Pattern That Changed Everything
</h3>

<p>Before <code>as const</code>, I wrote types twice—once in code, once in types:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// The data</span>
<span class="kd">const</span> <span class="nx">STATUS_CODES</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">OK</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span>
  <span class="na">NOT_FOUND</span><span class="p">:</span> <span class="mi">404</span><span class="p">,</span>
  <span class="na">SERVER_ERROR</span><span class="p">:</span> <span class="mi">500</span>
<span class="p">};</span>

<span class="c1">// Manually maintaining a matching type</span>
<span class="kd">type</span> <span class="nx">StatusCode</span> <span class="o">=</span> <span class="mi">200</span> <span class="o">|</span> <span class="mi">404</span> <span class="o">|</span> <span class="mi">500</span><span class="p">;</span>
</code></pre>

</div>



<p>This is maintenance hell. Add a status code? Update two places.</p>

<p>With <code>as const</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">STATUS_CODES</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">OK</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span>
  <span class="na">NOT_FOUND</span><span class="p">:</span> <span class="mi">404</span><span class="p">,</span>
  <span class="na">SERVER_ERROR</span><span class="p">:</span> <span class="mi">500</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">StatusCode</span> <span class="o">=</span> <span class="k">typeof</span> <span class="nx">STATUS_CODES</span><span class="p">[</span><span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">STATUS_CODES</span><span class="p">];</span>
<span class="c1">// Type: 200 | 404 | 500</span>
<span class="c1">// Single source of truth!</span>
</code></pre>

</div>



<h3>
  
  
  My Favorite Use Case: Type-Safe Constants
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">PERMISSIONS</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">READ</span><span class="p">:</span> <span class="dl">"</span><span class="s2">read</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">WRITE</span><span class="p">:</span> <span class="dl">"</span><span class="s2">write</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">DELETE</span><span class="p">:</span> <span class="dl">"</span><span class="s2">delete</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">ADMIN</span><span class="p">:</span> <span class="dl">"</span><span class="s2">admin</span><span class="dl">"</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">Permission</span> <span class="o">=</span> <span class="k">typeof</span> <span class="nx">PERMISSIONS</span><span class="p">[</span><span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">PERMISSIONS</span><span class="p">];</span>
<span class="c1">// Type: "read" | "write" | "delete" | "admin"</span>

<span class="kd">const</span> <span class="nx">USER_ROLES</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">guest</span><span class="p">:</span> <span class="p">[</span><span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">READ</span><span class="p">],</span>
  <span class="na">user</span><span class="p">:</span> <span class="p">[</span><span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">READ</span><span class="p">,</span> <span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">WRITE</span><span class="p">],</span>
  <span class="na">admin</span><span class="p">:</span> <span class="p">[</span><span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">READ</span><span class="p">,</span> <span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">WRITE</span><span class="p">,</span> <span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">DELETE</span><span class="p">,</span> <span class="nx">PERMISSIONS</span><span class="p">.</span><span class="nx">ADMIN</span><span class="p">]</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">Role</span> <span class="o">=</span> <span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">USER_ROLES</span><span class="p">;</span>
<span class="c1">// Type: "guest" | "user" | "admin"</span>

<span class="kd">function</span> <span class="nf">hasPermission</span><span class="p">(</span><span class="nx">role</span><span class="p">:</span> <span class="nx">Role</span><span class="p">,</span> <span class="nx">permission</span><span class="p">:</span> <span class="nx">Permission</span><span class="p">):</span> <span class="nx">boolean</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">USER_ROLES</span><span class="p">[</span><span class="nx">role</span><span class="p">].</span><span class="nf">includes</span><span class="p">(</span><span class="nx">permission</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  The Tuple Trick
</h3>

<p>This one blew my mind:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Without as const</span>
<span class="kd">const</span> <span class="nx">point</span> <span class="o">=</span> <span class="p">[</span><span class="mi">10</span><span class="p">,</span> <span class="mi">20</span><span class="p">];</span>
<span class="c1">// Type: number[] - could be any length!</span>

<span class="c1">// With as const</span>
<span class="kd">const</span> <span class="nx">point</span> <span class="o">=</span> <span class="p">[</span><span class="mi">10</span><span class="p">,</span> <span class="mi">20</span><span class="p">]</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>
<span class="c1">// Type: readonly [10, 20] - exactly 2 elements with exact values</span>

<span class="kd">function</span> <span class="nf">useCoordinates</span><span class="o">&lt;</span><span class="nx">T</span> <span class="kd">extends</span> <span class="k">readonly</span> <span class="p">[</span><span class="kr">number</span><span class="p">,</span> <span class="kr">number</span><span class="p">]</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">coords</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">x</span><span class="p">,</span> <span class="nx">y</span><span class="p">]</span> <span class="o">=</span> <span class="nx">coords</span><span class="p">;</span>
  <span class="c1">// TypeScript knows there are exactly 2 elements</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>When to use it:</strong> Whenever you have data that should become a type. Status codes, permissions, configuration values, lookup tables.</p>

<h2>
  
  
  3. The <code>infer</code> Keyword: Extracting Types from Types
</h2>

<p>I avoided <code>infer</code> for the longest time because it looked scary. Turns out, it's just pattern matching for types.</p>

<h3>
  
  
  The Basic Idea
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">GetReturnType</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span> <span class="nx">T</span> <span class="nf">extends </span><span class="p">(...</span><span class="nx">args</span><span class="p">:</span> <span class="kr">any</span><span class="p">[])</span> <span class="o">=&gt;</span> <span class="nx">infer</span> <span class="nx">R</span> <span class="p">?</span> <span class="nx">R</span> <span class="p">:</span> <span class="nx">never</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">getUser</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">John</span><span class="dl">"</span> <span class="p">};</span>
<span class="p">}</span>

<span class="kd">type</span> <span class="nx">User</span> <span class="o">=</span> <span class="nx">GetReturnType</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">getUser</span><span class="o">&gt;</span><span class="p">;</span>
<span class="c1">// Type: { id: number; name: string; }</span>
</code></pre>

</div>



<p>That <code>infer R</code> is saying: "If T is a function, extract its return type and call it R."</p>

<h3>
  
  
  Where This Actually Helps: Unwrapping Promises
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Unwrap</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span> <span class="nx">T</span> <span class="kd">extends</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">infer</span> <span class="nx">U</span><span class="o">&gt;</span> <span class="p">?</span> <span class="nx">U</span> <span class="p">:</span> <span class="nx">T</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchUser</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">John</span><span class="dl">"</span> <span class="p">};</span>
<span class="p">}</span>

<span class="kd">type</span> <span class="nx">User</span> <span class="o">=</span> <span class="nx">Unwrap</span><span class="o">&lt;</span><span class="nb">ReturnType</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">fetchUser</span><span class="o">&gt;&gt;</span><span class="p">;</span>
<span class="c1">// Type: { id: number; name: string; }</span>
<span class="c1">// Not Promise&lt;{ id: number; name: string; }&gt;!</span>
</code></pre>

</div>



<h3>
  
  
  My Most-Used Pattern: Deep Property Access
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">PropType</span><span class="o">&lt;</span><span class="nx">T</span><span class="p">,</span> <span class="nx">Path</span> <span class="kd">extends</span> <span class="kr">string</span><span class="o">&gt;</span> <span class="o">=</span> 
  <span class="nx">Path</span> <span class="kd">extends</span> <span class="kr">keyof</span> <span class="nx">T</span> 
    <span class="p">?</span> <span class="nx">T</span><span class="p">[</span><span class="nx">Path</span><span class="p">]</span>
    <span class="p">:</span> <span class="nx">Path</span> <span class="kd">extends</span> <span class="s2">`</span><span class="p">${</span><span class="nx">infer</span> <span class="nx">K</span><span class="p">}</span><span class="s2">.</span><span class="p">${</span><span class="nx">infer</span> <span class="nx">Rest</span><span class="p">}</span><span class="s2">`</span>
      <span class="p">?</span> <span class="nx">K</span> <span class="kd">extends</span> <span class="kr">keyof</span> <span class="nx">T</span>
        <span class="p">?</span> <span class="nx">PropType</span><span class="o">&lt;</span><span class="nx">T</span><span class="p">[</span><span class="nx">K</span><span class="p">],</span> <span class="nx">Rest</span><span class="o">&gt;</span>
        <span class="p">:</span> <span class="nx">never</span>
      <span class="p">:</span> <span class="nx">never</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">User</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">profile</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">settings</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">theme</span><span class="p">:</span> <span class="dl">"</span><span class="s2">light</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">dark</span><span class="dl">"</span><span class="p">;</span>
    <span class="p">};</span>
  <span class="p">};</span>
<span class="p">};</span>

<span class="kd">type</span> <span class="nx">Theme</span> <span class="o">=</span> <span class="nx">PropType</span><span class="o">&lt;</span><span class="nx">User</span><span class="p">,</span> <span class="dl">"</span><span class="s2">profile.settings.theme</span><span class="dl">"</span><span class="o">&gt;</span><span class="p">;</span>
<span class="c1">// Type: "light" | "dark"</span>
</code></pre>

</div>



<p>Okay, that one's getting fancy. But the point is: <code>infer</code> lets you extract types from other types. Once you understand the pattern, you'll start seeing uses everywhere.</p>

<p><strong>When to use it:</strong> When you're writing utility types that need to extract or transform parts of other types. Start with the built-in utilities (<code>ReturnType</code>, <code>Parameters</code>) to see the pattern.</p>

<h2>
  
  
  4. Template Literal Types: String Manipulation at the Type Level
</h2>

<p>I thought TypeScript was all about runtime JavaScript. Then I discovered you can manipulate strings in the type system.</p>

<h3>
  
  
  Basic Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Greeting</span> <span class="o">=</span> <span class="s2">`Hello, </span><span class="p">${</span><span class="kr">string</span><span class="p">}</span><span class="s2">!`</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">valid</span><span class="p">:</span> <span class="nx">Greeting</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">Hello, World!</span><span class="dl">"</span><span class="p">;</span>     <span class="c1">// ✓</span>
<span class="kd">const</span> <span class="nx">invalid</span><span class="p">:</span> <span class="nx">Greeting</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">Hi there!</span><span class="dl">"</span><span class="p">;</span>       <span class="c1">// ✗ Type error</span>
</code></pre>

</div>



<h3>
  
  
  Where It Clicks: CSS-in-JS
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">CSSUnit</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">px</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">em</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">rem</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">%</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">Size</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="kr">number</span><span class="p">}${</span><span class="nx">CSSUnit</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">setWidth</span><span class="p">(</span><span class="nx">width</span><span class="p">:</span> <span class="nx">Size</span><span class="p">)</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>

<span class="nf">setWidth</span><span class="p">(</span><span class="dl">"</span><span class="s2">100px</span><span class="dl">"</span><span class="p">);</span>   <span class="c1">// ✓</span>
<span class="nf">setWidth</span><span class="p">(</span><span class="dl">"</span><span class="s2">2em</span><span class="dl">"</span><span class="p">);</span>     <span class="c1">// ✓</span>
<span class="nf">setWidth</span><span class="p">(</span><span class="dl">"</span><span class="s2">50%</span><span class="dl">"</span><span class="p">);</span>     <span class="c1">// ✓</span>
<span class="nf">setWidth</span><span class="p">(</span><span class="dl">"</span><span class="s2">100</span><span class="dl">"</span><span class="p">);</span>     <span class="c1">// ✗ Type error!</span>
<span class="nf">setWidth</span><span class="p">(</span><span class="dl">"</span><span class="s2">100pt</span><span class="dl">"</span><span class="p">);</span>   <span class="c1">// ✗ Type error!</span>
</code></pre>

</div>



<h3>
  
  
  The Pattern I Use Most: Event Names
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Entity</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">post</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">comment</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">Action</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">create</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">update</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">delete</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">EventName</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">Entity</span><span class="p">}</span><span class="s2">:</span><span class="p">${</span><span class="nx">Action</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>

<span class="c1">// Type: "user:create" | "user:update" | "user:delete" | </span>
<span class="c1">//       "post:create" | "post:update" | "post:delete" |</span>
<span class="c1">//       "comment:create" | "comment:update" | "comment:delete"</span>

<span class="kd">const</span> <span class="nx">eventHandlers</span><span class="p">:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="nx">EventName</span><span class="p">,</span> <span class="p">(</span><span class="nx">data</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{</span>
  <span class="dl">"</span><span class="s2">user:create</span><span class="dl">"</span><span class="p">:</span> <span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">},</span>
  <span class="dl">"</span><span class="s2">user:update</span><span class="dl">"</span><span class="p">:</span> <span class="p">(</span><span class="nx">data</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">},</span>
  <span class="c1">// TypeScript ensures I handle all combinations!</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Auto-generating Type-Safe APIs
</h3>

<p>This is where it gets powerful:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">HTTPMethod</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">get</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">post</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">put</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">delete</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">Endpoint</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">users</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">posts</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">comments</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">APIMethod</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">HTTPMethod</span><span class="p">}${</span><span class="nb">Capitalize</span><span class="o">&lt;</span><span class="nx">Endpoint</span><span class="o">&gt;</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>

<span class="c1">// Type: "getUsers" | "postUsers" | "putUsers" | "deleteUsers" |</span>
<span class="c1">//       "getPosts" | "postPosts" | "putPosts" | "deletePosts" | ...</span>

<span class="kd">type</span> <span class="nx">API</span> <span class="o">=</span> <span class="p">{</span>
  <span class="p">[</span><span class="nx">K</span> <span class="k">in</span> <span class="nx">APIMethod</span><span class="p">]:</span> <span class="p">(</span><span class="na">params</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">api</span><span class="p">:</span> <span class="nx">API</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">getUsers</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">params</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">},</span>
  <span class="na">postUsers</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">params</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">},</span>
  <span class="c1">// etc.</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>When to use it:</strong> When you have multiple enums that combine (HTTP methods + routes, entities + actions, etc.). Also great for CSS values, file paths, and any domain with structured strings.</p>

<h2>
  
  
  5. Namespace Tricks: Organization Without the Overhead
</h2>

<p>I used to think namespaces were deprecated. Turns out, they're perfect for organizing related types and values.</p>

<h3>
  
  
  The Problem: Type Clutter
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// In a large codebase, this gets messy fast</span>
<span class="kd">type</span> <span class="nx">UserRole</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">admin</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">guest</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">UserPermissions</span> <span class="o">=</span> <span class="kr">string</span><span class="p">[];</span>
<span class="kd">type</span> <span class="nx">UserProfile</span> <span class="o">=</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">};</span>
<span class="kd">type</span> <span class="nx">UserSettings</span> <span class="o">=</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">};</span>
<span class="kd">const</span> <span class="nx">UserDefaults</span> <span class="o">=</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">};</span>
<span class="kd">function</span> <span class="nf">validateUser</span><span class="p">()</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  The Solution: Namespaces as Containers
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">namespace</span> <span class="nx">User</span> <span class="p">{</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Role</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">admin</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">guest</span><span class="dl">"</span><span class="p">;</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Permissions</span> <span class="o">=</span> <span class="kr">string</span><span class="p">[];</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Profile</span> <span class="o">=</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">};</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Settings</span> <span class="o">=</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">};</span>

  <span class="k">export</span> <span class="kd">const</span> <span class="nx">Defaults</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">guest</span><span class="dl">"</span> <span class="k">as</span> <span class="nx">Role</span><span class="p">,</span>
    <span class="na">permissions</span><span class="p">:</span> <span class="p">[]</span>
  <span class="p">};</span>

  <span class="k">export</span> <span class="kd">function</span> <span class="nf">validate</span><span class="p">(</span><span class="nx">user</span><span class="p">:</span> <span class="nx">Profile</span><span class="p">):</span> <span class="nx">boolean</span> <span class="p">{</span>
    <span class="c1">// ...</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="kd">function</span> <span class="nf">updateUser</span><span class="p">(</span><span class="nx">role</span><span class="p">:</span> <span class="nx">User</span><span class="p">.</span><span class="nx">Role</span><span class="p">,</span> <span class="nx">settings</span><span class="p">:</span> <span class="nx">User</span><span class="p">.</span><span class="nx">Settings</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Mix Types and Values
</h3>

<p>This is the killer feature: namespaces can contain both types and runtime values:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">namespace</span> <span class="nx">API</span> <span class="p">{</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Response</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">data</span><span class="p">:</span> <span class="nx">T</span><span class="p">;</span>
    <span class="nl">status</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">message</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">};</span>

  <span class="k">export</span> <span class="kd">const</span> <span class="nx">BaseURL</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">https://api.example.com</span><span class="dl">"</span><span class="p">;</span>

  <span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">request</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">endpoint</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Response</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">BaseURL</span><span class="p">}${</span><span class="nx">endpoint</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Everything is organized under one name</span>
<span class="kd">const</span> <span class="nx">response</span><span class="p">:</span> <span class="nx">API</span><span class="p">.</span><span class="nx">Response</span><span class="o">&lt;</span><span class="nx">User</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">API</span><span class="p">.</span><span class="nf">request</span><span class="p">(</span><span class="dl">"</span><span class="s2">/users/1</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Pattern: Domain-Driven Organization
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">namespace</span> <span class="nx">Auth</span> <span class="p">{</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Credentials</span> <span class="o">=</span> <span class="p">{</span> <span class="na">username</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">password</span><span class="p">:</span> <span class="kr">string</span> <span class="p">};</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Token</span> <span class="o">=</span> <span class="p">{</span> <span class="na">value</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">expiresAt</span><span class="p">:</span> <span class="nb">Date</span> <span class="p">};</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Session</span> <span class="o">=</span> <span class="p">{</span> <span class="na">user</span><span class="p">:</span> <span class="nx">User</span><span class="p">;</span> <span class="nl">token</span><span class="p">:</span> <span class="nx">Token</span> <span class="p">};</span>

  <span class="k">export</span> <span class="kd">const</span> <span class="nx">TokenKey</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">auth_token</span><span class="dl">"</span><span class="p">;</span>

  <span class="k">export</span> <span class="kd">function</span> <span class="nf">login</span><span class="p">(</span><span class="nx">creds</span><span class="p">:</span> <span class="nx">Credentials</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Session</span><span class="o">&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>
  <span class="k">export</span> <span class="kd">function</span> <span class="nf">logout</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>
  <span class="k">export</span> <span class="kd">function</span> <span class="nf">isAuthenticated</span><span class="p">():</span> <span class="nx">boolean</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>
<span class="p">}</span>

<span class="k">namespace</span> <span class="nx">Payment</span> <span class="p">{</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Method</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">card</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">paypal</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">crypto</span><span class="dl">"</span><span class="p">;</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Transaction</span> <span class="o">=</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">amount</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> <span class="nl">method</span><span class="p">:</span> <span class="nx">Method</span> <span class="p">};</span>
  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Receipt</span> <span class="o">=</span> <span class="p">{</span> <span class="na">transaction</span><span class="p">:</span> <span class="nx">Transaction</span><span class="p">;</span> <span class="nl">timestamp</span><span class="p">:</span> <span class="nb">Date</span> <span class="p">};</span>

  <span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">process</span><span class="p">(</span><span class="nx">amount</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">method</span><span class="p">:</span> <span class="nx">Method</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Receipt</span><span class="o">&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>
  <span class="k">export</span> <span class="kd">function</span> <span class="nf">refund</span><span class="p">(</span><span class="nx">transactionId</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span> <span class="cm">/* ... */</span> <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Clear, organized, discoverable</span>
<span class="nx">Auth</span><span class="p">.</span><span class="nf">login</span><span class="p">({</span> <span class="na">username</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">password</span><span class="p">:</span> <span class="dl">"</span><span class="s2">pass</span><span class="dl">"</span> <span class="p">});</span>
<span class="nx">Payment</span><span class="p">.</span><span class="nf">process</span><span class="p">(</span><span class="mi">100</span><span class="p">,</span> <span class="dl">"</span><span class="s2">card</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>When to use it:</strong> When you have related types and functions that belong together. Great for API clients, domain logic, and utility libraries. Just don't overuse—they're for organization, not encapsulation.</p>

<h2>
  
  
  6. The <code>satisfies</code> + <code>as const</code> Combo: Type Safety Nirvana
</h2>

<p>Here's where it all comes together. Combine <code>satisfies</code> and <code>as const</code> for the ultimate type safety:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Theme</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">colors</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">primary</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">secondary</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">};</span>
  <span class="nl">spacing</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">small</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">medium</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">large</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="p">};</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">theme</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">colors</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">primary</span><span class="p">:</span> <span class="dl">"</span><span class="s2">#007bff</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">secondary</span><span class="p">:</span> <span class="dl">"</span><span class="s2">#6c757d</span><span class="dl">"</span>
  <span class="p">},</span>
  <span class="na">spacing</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">small</span><span class="p">:</span> <span class="mi">4</span><span class="p">,</span>
    <span class="na">medium</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span>
    <span class="na">large</span><span class="p">:</span> <span class="mi">16</span>
  <span class="p">}</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span> <span class="nx">satisfies</span> <span class="nx">Theme</span><span class="p">;</span>

<span class="c1">// You get:</span>
<span class="c1">// 1. Type checking (satisfies Theme)</span>
<span class="c1">// 2. Literal types (as const)</span>
<span class="c1">// 3. Readonly protection (as const)</span>

<span class="kd">type</span> <span class="nx">PrimaryColor</span> <span class="o">=</span> <span class="k">typeof</span> <span class="nx">theme</span><span class="p">.</span><span class="nx">colors</span><span class="p">.</span><span class="nx">primary</span><span class="p">;</span>  <span class="c1">// "#007bff", not string</span>
<span class="kd">type</span> <span class="nx">SpacingSize</span> <span class="o">=</span> <span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">theme</span><span class="p">.</span><span class="nx">spacing</span><span class="p">;</span>    <span class="c1">// "small" | "medium" | "large"</span>
</code></pre>

</div>



<h3>
  
  
  Real Example: Feature Flags
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">FeatureFlag</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">enabled</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
  <span class="nl">rolloutPercentage</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">features</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">newDashboard</span><span class="p">:</span> <span class="p">{</span> <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">rolloutPercentage</span><span class="p">:</span> <span class="mi">50</span> <span class="p">},</span>
  <span class="na">advancedSearch</span><span class="p">:</span> <span class="p">{</span> <span class="na">enabled</span><span class="p">:</span> <span class="kc">false</span> <span class="p">},</span>
  <span class="na">darkMode</span><span class="p">:</span> <span class="p">{</span> <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span> <span class="nx">satisfies</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">FeatureFlag</span><span class="o">&gt;</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">FeatureName</span> <span class="o">=</span> <span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">features</span><span class="p">;</span>  <span class="c1">// "newDashboard" | "advancedSearch" | "darkMode"</span>

<span class="kd">function</span> <span class="nf">isFeatureEnabled</span><span class="p">(</span><span class="nx">name</span><span class="p">:</span> <span class="nx">FeatureName</span><span class="p">):</span> <span class="nx">boolean</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">features</span><span class="p">[</span><span class="nx">name</span><span class="p">].</span><span class="nx">enabled</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// Full autocomplete, type safety, and single source of truth!</span>
</code></pre>

</div>



<h2>
  
  
  7. Utility Types You're Probably Not Using
</h2>

<p>TypeScript ships with powerful utility types. I ignored them for years, writing my own versions of things that already existed.</p>

<h3>
  
  
  <code>Awaited&lt;T&gt;</code>: Unwrap Promises (Without <code>infer</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">User</span> <span class="o">=</span> <span class="nx">Awaited</span><span class="o">&lt;</span><span class="nb">ReturnType</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">fetchUser</span><span class="o">&gt;&gt;</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">fetchUser</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">John</span><span class="dl">"</span> <span class="p">};</span>
<span class="p">}</span>

<span class="c1">// User is { id: number; name: string }, not Promise&lt;...&gt;</span>
</code></pre>

</div>



<h3>
  
  
  <code>NonNullable&lt;T&gt;</code>: Filter Out null and undefined
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">MaybeString</span> <span class="o">=</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">;</span>
<span class="kd">type</span> <span class="nx">DefinitelyString</span> <span class="o">=</span> <span class="nb">NonNullable</span><span class="o">&lt;</span><span class="nx">MaybeString</span><span class="o">&gt;</span><span class="p">;</span>  <span class="c1">// string</span>

<span class="kd">function</span> <span class="nf">processValue</span><span class="p">(</span><span class="nx">value</span><span class="p">:</span> <span class="nx">MaybeString</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">value</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// TypeScript narrows to string here</span>
    <span class="kd">const</span> <span class="nx">upper</span><span class="p">:</span> <span class="nx">DefinitelyString</span> <span class="o">=</span> <span class="nx">value</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  <code>Extract&lt;T, U&gt;</code> and <code>Exclude&lt;T, U&gt;</code>: Union Type Filtering
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Status</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">pending</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">approved</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">rejected</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">cancelled</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">ActiveStatus</span> <span class="o">=</span> <span class="nb">Exclude</span><span class="o">&lt;</span><span class="nx">Status</span><span class="p">,</span> <span class="dl">"</span><span class="s2">rejected</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">cancelled</span><span class="dl">"</span><span class="o">&gt;</span><span class="p">;</span>
<span class="c1">// Type: "pending" | "approved"</span>

<span class="kd">type</span> <span class="nx">FinalStatus</span> <span class="o">=</span> <span class="nb">Extract</span><span class="o">&lt;</span><span class="nx">Status</span><span class="p">,</span> <span class="dl">"</span><span class="s2">approved</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">rejected</span><span class="dl">"</span><span class="o">&gt;</span><span class="p">;</span>
<span class="c1">// Type: "approved" | "rejected"</span>
</code></pre>

</div>



<h3>
  
  
  <code>Parameters&lt;T&gt;</code> and <code>ConstructorParameters&lt;T&gt;</code>: Function Introspection
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">createUser</span><span class="p">(</span><span class="nx">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">age</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">admin</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span> <span class="nx">name</span><span class="p">,</span> <span class="nx">age</span><span class="p">,</span> <span class="nx">role</span> <span class="p">};</span>
<span class="p">}</span>

<span class="kd">type</span> <span class="nx">CreateUserParams</span> <span class="o">=</span> <span class="nb">Parameters</span><span class="o">&lt;</span><span class="k">typeof</span> <span class="nx">createUser</span><span class="o">&gt;</span><span class="p">;</span>
<span class="c1">// Type: [name: string, age: number, role: "admin" | "user"]</span>

<span class="c1">// Now you can use the same types elsewhere:</span>
<span class="kd">function</span> <span class="nf">logUserCreation</span><span class="p">(...</span><span class="nx">args</span><span class="p">:</span> <span class="nx">CreateUserParams</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Creating user with:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">args</span><span class="p">);</span>
  <span class="nf">createUser</span><span class="p">(...</span><span class="nx">args</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  8. The tsconfig.json Settings That Actually Matter
</h2>

<p>I used to cargo-cult my tsconfig from project to project. Here are the settings I wish I'd understood sooner:</p>

<h3>
  
  
  <code>strict: true</code> (Just Do It)
</h3>

<p>Stop fighting this. Turn it on from day one. The short-term pain is worth it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"compilerOptions"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"strict"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  <code>noUncheckedIndexedAccess</code>: Prevent the Classic Bug
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Without this setting:</span>
<span class="kd">const</span> <span class="nx">dict</span><span class="p">:</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="kr">string</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{};</span>
<span class="kd">const</span> <span class="nx">value</span> <span class="o">=</span> <span class="nx">dict</span><span class="p">[</span><span class="dl">"</span><span class="s2">nonexistent</span><span class="dl">"</span><span class="p">];</span>  <span class="c1">// Type: string (LIES!)</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">value</span><span class="p">.</span><span class="nf">toUpperCase</span><span class="p">());</span>   <span class="c1">// Runtime error!</span>

<span class="c1">// With "noUncheckedIndexedAccess": true</span>
<span class="kd">const</span> <span class="nx">value</span> <span class="o">=</span> <span class="nx">dict</span><span class="p">[</span><span class="dl">"</span><span class="s2">nonexistent</span><span class="dl">"</span><span class="p">];</span>  <span class="c1">// Type: string | undefined</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">value</span><span class="p">.</span><span class="nf">toUpperCase</span><span class="p">());</span>   <span class="c1">// Type error - must check for undefined</span>
</code></pre>

</div>



<h3>
  
  
  <code>exactOptionalPropertyTypes</code>: Optional vs Undefined
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">User</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">age</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>  <span class="c1">// Can be undefined or missing</span>
<span class="p">};</span>

<span class="c1">// Without exactOptionalPropertyTypes:</span>
<span class="kd">const</span> <span class="nx">user1</span><span class="p">:</span> <span class="nx">User</span> <span class="o">=</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">John</span><span class="dl">"</span><span class="p">,</span> <span class="na">age</span><span class="p">:</span> <span class="kc">undefined</span> <span class="p">};</span>  <span class="c1">// Allowed</span>

<span class="c1">// With exactOptionalPropertyTypes:</span>
<span class="kd">const</span> <span class="nx">user2</span><span class="p">:</span> <span class="nx">User</span> <span class="o">=</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">John</span><span class="dl">"</span><span class="p">,</span> <span class="na">age</span><span class="p">:</span> <span class="kc">undefined</span> <span class="p">};</span>  <span class="c1">// Error!</span>
<span class="kd">const</span> <span class="nx">user3</span><span class="p">:</span> <span class="nx">User</span> <span class="o">=</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">John</span><span class="dl">"</span> <span class="p">};</span>                   <span class="c1">// This is the only way</span>
</code></pre>

</div>



<h3>
  
  
  <code>noPropertyAccessFromIndexSignature</code>: Catch Typos
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">Config</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">apiUrl</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">[</span><span class="na">key</span><span class="p">:</span> <span class="kr">string</span><span class="p">]:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">config</span><span class="p">:</span> <span class="nx">Config</span> <span class="o">=</span> <span class="p">{</span> <span class="na">apiUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://api.com</span><span class="dl">"</span> <span class="p">};</span>

<span class="c1">// Without this setting:</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">config</span><span class="p">.</span><span class="nx">apiUrll</span><span class="p">);</span>  <span class="c1">// No error, returns undefined</span>

<span class="c1">// With "noPropertyAccessFromIndexSignature": true</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">config</span><span class="p">.</span><span class="nx">apiUrll</span><span class="p">);</span>  <span class="c1">// Type error - must use bracket notation</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">config</span><span class="p">[</span><span class="dl">"</span><span class="s2">apiUrll</span><span class="dl">"</span><span class="p">]);</span>  <span class="c1">// OK (you're explicitly opting into dynamic access)</span>
</code></pre>

</div>



<h2>
  
  
  What I'd Tell My Past Self
</h2>

<ol>
<li><p><strong>Learn <code>satisfies</code> first.</strong> It solves the most common pain point: wanting both validation and precise types.</p></li>
<li><p><strong>Use <code>as const</code> liberally.</strong> Any time you have data that should become a type, slap <code>as const</code> on it.</p></li>
<li><p><strong>Don't fear advanced types.</strong> Start with <code>ReturnType</code> and <code>typeof</code>, then explore from there. The pattern repeats.</p></li>
<li><p><strong>Namespaces aren't dead.</strong> They're perfect for organizing related types and functions.</p></li>
<li><p><strong>Read the tsconfig docs.</strong> Seriously. Five minutes of reading could prevent hours of debugging.</p></li>
<li><p><strong>The type system is your friend.</strong> If the types feel wrong, your code might be wrong. Listen to them.</p></li>
</ol>

<h2>
  
  
  The Compounding Effect
</h2>

<p>Each of these features individually is nice. Together, they transform how you write TypeScript:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Everything we learned, combined:</span>
<span class="k">namespace</span> <span class="nx">API</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">ENDPOINTS</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">users</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/users</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">posts</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/posts</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">comments</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/comments</span><span class="dl">"</span>
  <span class="p">}</span> <span class="k">as</span> <span class="kd">const</span> <span class="nx">satisfies</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="s2">`/</span><span class="p">${</span><span class="kr">string</span><span class="p">}</span><span class="s2">`</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="kd">type</span> <span class="nx">Endpoint</span> <span class="o">=</span> <span class="kr">keyof</span> <span class="k">typeof</span> <span class="nx">ENDPOINTS</span><span class="p">;</span>

  <span class="k">export</span> <span class="kd">type</span> <span class="nx">Response</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">data</span><span class="p">:</span> <span class="nx">T</span><span class="p">;</span>
    <span class="nl">status</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">error</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">};</span>

  <span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">get</span><span class="o">&lt;</span><span class="nx">E</span> <span class="kd">extends</span> <span class="nx">Endpoint</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="nx">endpoint</span><span class="p">:</span> <span class="nx">E</span>
  <span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Response</span><span class="o">&lt;</span><span class="nx">unknown</span><span class="o">&gt;&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">url</span> <span class="o">=</span> <span class="nx">ENDPOINTS</span><span class="p">[</span><span class="nx">endpoint</span><span class="p">];</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="nx">url</span><span class="p">);</span>
    <span class="k">return</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Type-safe, autocomplete everywhere, single source of truth</span>
<span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">API</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">users</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<h2>
  
  
  Your Turn
</h2>

<p>These features were hiding in plain sight in my editor for years. Each one I discovered made me think: "I could have saved so much time if I'd known this earlier."</p>

<p>You don't need to learn them all at once. Pick one that solves a pain point you have today. Use it for a week. Then come back and grab another.</p>

<p>Your code will thank you. Your team will thank you. And most importantly, future you will thank you.</p>




<p><em>What TypeScript features changed the game for you? I'd love to hear what you wish you'd learned sooner.</em></p>

