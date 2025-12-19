---
Title: Zombie State in Pinia: A Silent Bug You Might Already Have
Description: 
Author: hichem ben chaabene
Date: 2025-12-19T22:02:32.000Z
Robots: noindex,nofollow
Template: index
---
<p>Pinia is one of the best state management libraries in the Vue ecosystem.<br>
It’s simple, type-safe, and feels natural with the Composition API.</p>

<p>But there’s a subtle problem many teams run into — often without realizing it.</p>

<p>That problem is <strong>zombie state</strong>.</p>
<h2>
  
  
  What Is Zombie State?
</h2>

<p>Zombie state is state that should no longer exist, but is still alive and affecting your app.</p>

<p>It usually comes from:</p>

<ul>
<li>A previous page</li>
<li>An old user flow</li>
<li>A completed form</li>
<li>A finished async request</li>
</ul>

<p>Yet it continues to:</p>

<ul>
<li>Prefill inputs</li>
<li>Trigger validation</li>
<li>Affect UI decisions</li>
<li>Cause "random" bugs</li>
</ul>

<p>It doesn’t crash your app.<br>
It just quietly causes confusion.</p>
<h2>
  
  
  Why Zombie State Happens in Pinia
</h2>

<p>Pinia stores are:</p>

<ul>
<li>Global</li>
<li>Long-lived</li>
<li>Singletons by default</li>
</ul>

<p>This means one important thing:</p>

<p><code>Navigation away doesn't reset store data</code></p>

<p>Unless you explicitly reset it, the state stays in memory — even when it no longer makes sense.</p>

<p>A Very Common Example<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// useWizardStore.ts</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">useWizardStore</span> <span class="o">=</span> <span class="nf">defineStore</span><span class="p">(</span><span class="dl">'</span><span class="s1">wizard</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">step</span> <span class="o">=</span> <span class="nf">ref</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>
  <span class="kd">const</span> <span class="nx">email</span> <span class="o">=</span> <span class="nf">ref</span><span class="p">(</span><span class="dl">'</span><span class="s1">old@email.com</span><span class="dl">'</span><span class="p">)</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">step</span><span class="p">,</span> <span class="nx">email</span> <span class="p">}</span>
<span class="p">})</span>
</code></pre>

</div>



<h2>
  
  
  What happens
</h2>

<p>1- user opens the wizard<br>
2- fills some steps<br>
3- navigates away<br>
4- comes back later</p>

<p>The wizard: </p>

<ul>
<li>Starts on step 2</li>
<li>Shows the old email</li>
<li>Fails validation unexpectedly</li>
</ul>

<p>The store never reset, that's the zombie state.</p>
<h2>
  
  
  Async Zombie State (Even Trickier)
</h2>

<p>Zombie state can also come from async code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">async</span> <span class="kd">function</span> <span class="nf">loadUser</span><span class="p">()</span> <span class="p">{</span>
  <span class="nx">user</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchUser</span><span class="p">()</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Scenario: <br>
1- User opens page A<br>
2- Request starts<br>
3- User navigates to page B<br>
4- Request from Page A finishes late</p>

<p>Result:<br>
Page B shows data from page A</p>

<p>This is zombie state caused by a late async response.</p>
<h2>
  
  
  How Zombie State Shows Up in Real Apps
</h2>

<p>You might recognize these symptoms:</p>

<ul>
<li>Forms already filled when they shouldn’t be</li>
<li>Validation errors on first render</li>
<li>Wrong data after navigation</li>
<li>UI behaving differently after back/forward</li>
<li>Bugs that disappear on refresh</li>
</ul>

<p>These are some of the hardest bugs to debug — because they depend on history, not just current state.</p>
<h2>
  
  
  The Key Question to Ask
</h2>

<p>Whenever you create a store, ask:</p>

<p><code>Who owns this state, and when should it be destroyed?</code></p>

<p>If the answer isn’t clear, zombie state is likely.</p>
<h2>
  
  
  How to Prevent Zombie State in Pinia
</h2>

<p>1- Add an explicit reset</p>

<p>The simplest and most effective solution<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">useWizardStore</span> <span class="o">=</span> <span class="nf">defineStore</span><span class="p">(</span><span class="dl">'</span><span class="s1">wizard</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">step</span> <span class="o">=</span> <span class="nf">ref</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
  <span class="kd">const</span> <span class="nx">email</span> <span class="o">=</span> <span class="nf">ref</span><span class="p">(</span><span class="dl">''</span><span class="p">)</span>

  <span class="kd">function</span> <span class="nf">reset</span><span class="p">()</span> <span class="p">{</span>
    <span class="nx">step</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="mi">1</span>
    <span class="nx">email</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="dl">''</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">step</span><span class="p">,</span> <span class="nx">email</span><span class="p">,</span> <span class="nx">reset</span> <span class="p">}</span>
<span class="p">})</span>
</code></pre>

</div>



<p>Use it when the flow ends<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">onUnmounted</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">store</span><span class="p">.</span><span class="nf">reset</span><span class="p">())</span>
</code></pre>

</div>



<p>Pinia doesn’t reset state for you — you have to be intentional.</p>

<p>2- Reset on context changes<br>
if state depends on route parameters, reset when they change.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">watch</span><span class="p">(</span>
  <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">route</span><span class="p">.</span><span class="nx">params</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
  <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">store</span><span class="p">.</span><span class="nf">reset</span><span class="p">()</span>
<span class="p">)</span>
</code></pre>

</div>



<p>This prevents old data from leaking into new contexts.</p>

<p>3- Guard Async Requests</p>

<p>A small guard can prevent async zombie state.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">let</span> <span class="nx">requestId</span> <span class="o">=</span> <span class="mi">0</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">load</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">id</span> <span class="o">=</span> <span class="o">++</span><span class="nx">requestId</span>
  <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchData</span><span class="p">()</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">id</span> <span class="o">!==</span> <span class="nx">requestId</span><span class="p">)</span> <span class="k">return</span>
  <span class="nx">state</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="nx">data</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Late responses are ignored instead of overwriting valid state.</p>

<p>4- Separate UI State from Domain State</p>

<p>UI state often becomes zombie state.</p>

<p><strong>Good candidates for stores</strong></p>

<p>-User data<br>
-Orders<br>
-Permissions</p>

<p><strong>Bad candidates</strong><br>
-Input values<br>
-Modal visibility<br>
-Hover or focus state</p>

<p>Keep short-lived UI state close to the component when possible.</p>

<h2>
  
  
  Zombie State vs Stale State
</h2>

<p>They’re not the same.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td>Stale state</td>
<td>Old but still valid</td>
</tr>
<tr>
<td>Zombie state</td>
<td>Invalid but still active</td>
</tr>
</tbody>
</table></div>

<p>Zombie state is dangerous because it looks correct — until it isn’t.</p>

<h2>
  
  
  A Simple Rule of Thumb
</h2>

<p><code>If state outlives its owner, it becomes a zombie.</code></p>

<p>Once you start thinking about state ownership and lifecycle, many “random” bugs suddenly make sense.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>Zombie state isn’t a Pinia problem.<br>
It’s a <strong>state lifecycle problem</strong>.</p>

<p>Pinia gives us powerful tools — but with that power comes responsibility:</p>

<ul>
<li>Define ownership</li>
<li>Control lifetime</li>
<li>Reset intentionally</li>
</ul>

<p>Doing this consistently leads to apps that are:</p>

<ul>
<li>Easier to reason about</li>
<li>Easier to debug</li>
<li>More predictable for users</li>
</ul>

<p>If this helps even one developer avoid a hard-to-trace production bug, it’s worth sharing.</p>

