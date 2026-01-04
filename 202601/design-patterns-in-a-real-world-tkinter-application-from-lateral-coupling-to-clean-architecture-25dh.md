---
Title: Design Patterns in a Real-World Tkinter Application: From Lateral Coupling to Clean Architecture
Description: 
Author: giuseppe costanzi
Date: 2026-01-04T22:02:03.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>What you can build with Python's standard library</em></p>




<h2>
  
  
  Beyond "Hello World"
</h2>

<p>Python, Tkinter, SQLite. The "batteries included" stack.</p>

<p>Most tutorials stop at a contact list or a todo app. But this stack can do <strong>much more</strong> — production applications solving real problems, with clean architecture and proper design patterns.</p>

<p>This article shows how. No external GUI frameworks, no ORMs, no heavyweight dependencies. Just the standard library, good patterns, and real-world requirements.</p>

<p>I'll walk you through <a href="https://github.com/1966bc/inventarium" rel="noopener noreferrer">Inventarium</a>, a laboratory inventory management system running in production at a hospital. Along the way, we'll cover:</p>

<ul>
<li>
<strong>Singleton</strong> (two different implementations!)</li>
<li>
<strong>Observer</strong> (decoupled view communication)</li>
<li>
<strong>Registry</strong> (Toplevel window management)</li>
<li><strong>Template Method</strong></li>
<li><strong>Mixin Architecture</strong></li>
</ul>

<p>All with real code, real problems, and real solutions.</p>




<h2>
  
  
  The Problem: Lateral Coupling
</h2>

<p>Picture this: You have a <code>Warehouse</code> window showing stock levels. You have a <code>Delivery</code> window where you record incoming goods. When a delivery is saved, the warehouse view needs to refresh.</p>

<p>The naive solution:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># In delivery.py
</span><span class="k">def</span> <span class="nf">on_save</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
    <span class="n">self</span><span class="p">.</span><span class="nf">save_to_database</span><span class="p">()</span>

    <span class="c1"># Lateral coupling: Delivery KNOWS about Warehouse
</span>    <span class="k">if</span> <span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">:</span>
        <span class="n">warehouse</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">[</span><span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span><span class="p">]</span>
        <span class="n">warehouse</span><span class="p">.</span><span class="nf">refresh_batches</span><span class="p">()</span>
</code></pre>

</div>



<p>This works. But now add:</p>

<ul>
<li>
<code>Expiring</code> view (shows batches about to expire)</li>
<li>
<code>Stocks</code> view (shows stock levels)</li>
<li>
<code>Requests</code> view (shows pending orders)</li>
</ul>

<p>Suddenly, <code>Delivery</code> needs to know about <strong>all of them</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># This is getting ugly...
</span><span class="k">def</span> <span class="nf">on_save</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
    <span class="n">self</span><span class="p">.</span><span class="nf">save_to_database</span><span class="p">()</span>

    <span class="k">if</span> <span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">:</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">[</span><span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span><span class="p">].</span><span class="nf">refresh_batches</span><span class="p">()</span>
    <span class="k">if</span> <span class="sh">"</span><span class="s">expiring</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">:</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">[</span><span class="sh">"</span><span class="s">expiring</span><span class="sh">"</span><span class="p">].</span><span class="nf">refresh</span><span class="p">()</span>
    <span class="k">if</span> <span class="sh">"</span><span class="s">stocks</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">:</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">[</span><span class="sh">"</span><span class="s">stocks</span><span class="sh">"</span><span class="p">].</span><span class="nf">refresh</span><span class="p">()</span>
    <span class="c1"># And on and on...
</span></code></pre>

</div>



<p>This is <strong>lateral coupling</strong> — modules that should be independent are now tangled together. Adding a new view means modifying every module that might affect it.</p>




<h2>
  
  
  Solution: The Observer Pattern
</h2>

<p>The fix is elegant: instead of "push" (delivery tells everyone), use "pull" (interested parties subscribe).</p>

<h3>
  
  
  The Subject (Engine)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">DBMS</span><span class="p">,</span> <span class="n">Controller</span><span class="p">,</span> <span class="n">Tools</span><span class="p">,</span> <span class="n">Launcher</span><span class="p">,</span> <span class="n">metaclass</span><span class="o">=</span><span class="n">_EngineMeta</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Main orchestrator with event system.</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">database</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">autocommit</span><span class="p">:</span> <span class="nb">bool</span> <span class="o">=</span> <span class="bp">True</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">database</span><span class="o">=</span><span class="n">database</span><span class="p">,</span> <span class="n">autocommit</span><span class="o">=</span><span class="n">autocommit</span><span class="p">)</span>

        <span class="c1"># Event system: event_name -&gt; [callbacks]
</span>        <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">subscribe</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">event</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">callback</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Register a callback for an event.</span><span class="sh">"""</span>
        <span class="k">if</span> <span class="n">event</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">[</span><span class="n">event</span><span class="p">]</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">if</span> <span class="n">callback</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">[</span><span class="n">event</span><span class="p">]:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">[</span><span class="n">event</span><span class="p">].</span><span class="nf">append</span><span class="p">(</span><span class="n">callback</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">unsubscribe</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">event</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">callback</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Remove a callback from an event.</span><span class="sh">"""</span>
        <span class="k">if</span> <span class="n">event</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">:</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">[</span><span class="n">event</span><span class="p">].</span><span class="nf">remove</span><span class="p">(</span><span class="n">callback</span><span class="p">)</span>
            <span class="k">except</span> <span class="nb">ValueError</span><span class="p">:</span>
                <span class="k">pass</span>

    <span class="k">def</span> <span class="nf">notify</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">event</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="bp">None</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="bp">None</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Notify all subscribers of an event.</span><span class="sh">"""</span>
        <span class="k">for</span> <span class="n">callback</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">_subscribers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">event</span><span class="p">,</span> <span class="p">[]):</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="nf">callback</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>
            <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
                <span class="k">pass</span>  <span class="c1"># Subscriber might be dead, ignore
</span></code></pre>

</div>



<h3>
  
  
  The Publisher (Delivery)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># In delivery.py - clean and simple
</span><span class="k">def</span> <span class="nf">on_save</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
    <span class="n">self</span><span class="p">.</span><span class="nf">save_to_database</span><span class="p">()</span>

    <span class="c1"># Delivery doesn't know WHO listens, just WHAT happened
</span>    <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">notify</span><span class="p">(</span><span class="sh">"</span><span class="s">stock_changed</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  The Subscribers (Warehouse, Expiring, etc.)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># In warehouse.py
</span><span class="k">class</span> <span class="nc">UI</span><span class="p">(</span><span class="n">ParentView</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">parent</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">parent</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">_reusing</span><span class="p">:</span>
            <span class="k">return</span>

        <span class="c1"># Subscribe to events we care about
</span>        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="sh">"</span><span class="s">stock_changed</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">on_stock_changed</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="sh">"</span><span class="s">batch_cancelled</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">on_stock_changed</span><span class="p">)</span>

        <span class="n">self</span><span class="p">.</span><span class="nf">init_ui</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">on_stock_changed</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">React to stock changes from any source.</span><span class="sh">"""</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">refresh_batches</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">on_cancel</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">evt</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="c1"># Clean up subscriptions before closing
</span>        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">unsubscribe</span><span class="p">(</span><span class="sh">"</span><span class="s">stock_changed</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">on_stock_changed</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">unsubscribe</span><span class="p">(</span><span class="sh">"</span><span class="s">batch_cancelled</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">on_stock_changed</span><span class="p">)</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">on_cancel</span><span class="p">()</span>
</code></pre>

</div>



<h3>
  
  
  The Events
</h3>

<p>Inventarium uses these events:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Event</th>
<th>Fired by</th>
<th>Subscribers</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>stock_changed</code></td>
<td>Delivery</td>
<td>Warehouse, Stocks</td>
</tr>
<tr>
<td><code>batch_cancelled</code></td>
<td>Expiring</td>
<td>Warehouse</td>
</tr>
<tr>
<td><code>category_changed</code></td>
<td>Category</td>
<td>Warehouse, Products</td>
</tr>
<tr>
<td><code>package_changed</code></td>
<td>Package</td>
<td>Warehouse</td>
</tr>
<tr>
<td><code>request_changed</code></td>
<td>Requests</td>
<td>Delivery</td>
</tr>
</tbody>
</table></div>

<p>Now adding a new view is trivial — just subscribe to the events you care about. No existing code needs to change.</p>




<h2>
  
  
  Singleton: Three Ways
</h2>

<p>Tkinter has a classic problem: users double-click menu items and create duplicate windows. Here are three approaches I use:</p>

<h3>
  
  
  1. Singleton via <code>__new__</code> (ParentView)
</h3>

<p>For main windows (Warehouse, Products, Suppliers), I use <code>__new__</code> to intercept instance creation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ParentView</span><span class="p">(</span><span class="n">tk</span><span class="p">.</span><span class="n">Toplevel</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Base class for main views with singleton behavior.</span><span class="sh">"""</span>

    <span class="n">_instance</span> <span class="o">=</span> <span class="bp">None</span>

    <span class="k">def</span> <span class="nf">__new__</span><span class="p">(</span><span class="n">cls</span><span class="p">,</span> <span class="n">parent</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="c1"># If instance exists and is alive, reuse it
</span>        <span class="k">if</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="k">if</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span><span class="p">.</span><span class="nf">winfo_exists</span><span class="p">():</span>
                    <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span><span class="p">.</span><span class="nf">lift</span><span class="p">()</span>
                    <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span><span class="p">.</span><span class="nf">focus_set</span><span class="p">()</span>
                    <span class="k">return</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span>
            <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
                <span class="k">pass</span>

        <span class="c1"># Create new instance
</span>        <span class="n">obj</span> <span class="o">=</span> <span class="nf">super</span><span class="p">().</span><span class="nf">__new__</span><span class="p">(</span><span class="n">cls</span><span class="p">)</span>
        <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span> <span class="o">=</span> <span class="n">obj</span>
        <span class="k">return</span> <span class="n">obj</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">parent</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="c1"># Guard against re-initialization
</span>        <span class="n">self</span><span class="p">.</span><span class="n">_reusing</span> <span class="o">=</span> <span class="nf">getattr</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="sh">"</span><span class="s">_is_init</span><span class="sh">"</span><span class="p">,</span> <span class="bp">False</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">_reusing</span><span class="p">:</span>
            <span class="k">return</span>

        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="n">name</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_is_init</span> <span class="o">=</span> <span class="bp">True</span>

        <span class="c1"># Anti-flash: hide during construction
</span>        <span class="n">self</span><span class="p">.</span><span class="nf">attributes</span><span class="p">(</span><span class="sh">"</span><span class="s">-alpha</span><span class="sh">"</span><span class="p">,</span> <span class="mf">0.0</span><span class="p">)</span>

        <span class="n">self</span><span class="p">.</span><span class="n">parent</span> <span class="o">=</span> <span class="n">parent</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">nametowidget</span><span class="p">(</span><span class="sh">"</span><span class="s">.</span><span class="sh">"</span><span class="p">).</span><span class="n">engine</span>

    <span class="k">def</span> <span class="nf">show</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Show window after construction.</span><span class="sh">"""</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">deiconify</span><span class="p">()</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">attributes</span><span class="p">(</span><span class="sh">"</span><span class="s">-alpha</span><span class="sh">"</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">on_cancel</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">evt</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Clean up singleton reference.</span><span class="sh">"""</span>
        <span class="nf">type</span><span class="p">(</span><span class="n">self</span><span class="p">).</span><span class="n">_instance</span> <span class="o">=</span> <span class="bp">None</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_is_init</span> <span class="o">=</span> <span class="bp">False</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">destroy</span><span class="p">()</span>
</code></pre>

</div>



<p>Usage is clean:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">UI</span><span class="p">(</span><span class="n">ParentView</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">parent</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">parent</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">warehouse</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">_reusing</span><span class="p">:</span>
            <span class="k">return</span>  <span class="c1"># Window already exists, nothing to do
</span>
        <span class="c1"># ... setup UI ...
</span>        <span class="n">self</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<h3>
  
  
  2. Registry Pattern (ChildView)
</h3>

<p>For dialogs (edit forms, detail views), I use a registry to prevent duplicate windows — we want one instance per dialog type, and if the user tries to open another, we close the existing one first:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ChildView</span><span class="p">(</span><span class="n">tk</span><span class="p">.</span><span class="n">Toplevel</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Base class for CRUD dialogs with registry.</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">parent</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">name</span><span class="o">=</span><span class="n">name</span><span class="p">)</span>

        <span class="n">self</span><span class="p">.</span><span class="n">parent</span> <span class="o">=</span> <span class="n">parent</span>
        <span class="n">self</span><span class="p">.</span><span class="n">engine</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">nametowidget</span><span class="p">(</span><span class="sh">"</span><span class="s">.</span><span class="sh">"</span><span class="p">).</span><span class="n">engine</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_dialog_name</span> <span class="o">=</span> <span class="n">name</span>

        <span class="c1"># Register in global registry to track open windows
</span>        <span class="k">if</span> <span class="n">name</span><span class="p">:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">[</span><span class="n">name</span><span class="p">]</span> <span class="o">=</span> <span class="n">self</span>

    <span class="k">def</span> <span class="nf">on_cancel</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">evt</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="c1"># Unregister on close
</span>        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">_dialog_name</span><span class="p">:</span>
            <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="n">dict_instances</span><span class="p">.</span><span class="nf">pop</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="n">_dialog_name</span><span class="p">,</span> <span class="bp">None</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">destroy</span><span class="p">()</span>
</code></pre>

</div>



<p>The caller checks the registry before opening:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">on_edit</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
    <span class="c1"># Close existing dialog if open
</span>    <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">close_instance</span><span class="p">(</span><span class="sh">"</span><span class="s">product</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Open new one
</span>    <span class="n">obj</span> <span class="o">=</span> <span class="n">product</span><span class="p">.</span><span class="nc">UI</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">index</span><span class="o">=</span><span class="n">product_id</span><span class="p">)</span>
    <span class="n">obj</span><span class="p">.</span><span class="nf">on_open</span><span class="p">(</span><span class="n">selected</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  3. Metaclass Singleton (Engine)
</h3>

<p>For the Engine class (the application's brain), I use a metaclass — the "proper" Python way:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">_EngineMeta</span><span class="p">(</span><span class="nb">type</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Metaclass ensuring single Engine instance.</span><span class="sh">"""</span>

    <span class="n">_instance</span> <span class="o">=</span> <span class="bp">None</span>

    <span class="k">def</span> <span class="nf">__call__</span><span class="p">(</span><span class="n">cls</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span> <span class="ow">is</span> <span class="bp">None</span><span class="p">:</span>
            <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span> <span class="o">=</span> <span class="nf">super</span><span class="p">().</span><span class="nf">__call__</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">cls</span><span class="p">.</span><span class="n">_instance</span>


<span class="k">class</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">DBMS</span><span class="p">,</span> <span class="n">Controller</span><span class="p">,</span> <span class="n">Tools</span><span class="p">,</span> <span class="n">Launcher</span><span class="p">,</span> <span class="n">metaclass</span><span class="o">=</span><span class="n">_EngineMeta</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Main orchestrator - singleton via metaclass.</span><span class="sh">"""</span>
    <span class="k">pass</span>
</code></pre>

</div>



<p>This intercepts creation <strong>before</strong> <code>__new__</code> and <code>__init__</code> — the cleanest possible Singleton.</p>

<blockquote>
<p><strong>Note:</strong> The metaclass is "educational" in Inventarium since Engine is only instantiated once in <code>App</code>. But it demonstrates the pattern and protects against accidents.</p>
</blockquote>




<h2>
  
  
  Template Method: ParentView and ChildView
</h2>

<p>Both base classes implement the <strong>Template Method</strong> pattern — they define the skeleton, subclasses fill in the details:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────┐
│              ParentView                 │
├─────────────────────────────────────────┤
│ __new__()      → Singleton logic        │
│ __init__()     → Anti-flash setup       │
│ show()         → Display window         │
│ on_cancel()    → Cleanup + destroy      │
├─────────────────────────────────────────┤
│ Subclass provides:                      │
│ • init_ui()    → Build the interface    │
│ • Event handlers                        │
└─────────────────────────────────────────┘
</code></pre>

</div>



<p>Every window follows the same lifecycle:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">UI</span><span class="p">(</span><span class="n">ParentView</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">parent</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">parent</span><span class="p">,</span> <span class="n">name</span><span class="o">=</span><span class="sh">"</span><span class="s">myview</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">self</span><span class="p">.</span><span class="n">_reusing</span><span class="p">:</span>
            <span class="k">return</span>

        <span class="n">self</span><span class="p">.</span><span class="nf">protocol</span><span class="p">(</span><span class="sh">"</span><span class="s">WM_DELETE_WINDOW</span><span class="sh">"</span><span class="p">,</span> <span class="n">self</span><span class="p">.</span><span class="n">on_cancel</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">init_ui</span><span class="p">()</span>           <span class="c1"># Subclass builds UI
</span>        <span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">center_window</span><span class="p">(</span><span class="n">self</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>              <span class="c1"># Template method
</span>
    <span class="k">def</span> <span class="nf">init_ui</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="c1"># Subclass-specific UI code
</span>        <span class="k">pass</span>

    <span class="k">def</span> <span class="nf">on_cancel</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">evt</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="c1"># Subclass cleanup...
</span>        <span class="nf">super</span><span class="p">().</span><span class="nf">on_cancel</span><span class="p">()</span>      <span class="c1"># Template method
</span></code></pre>

</div>






<h2>
  
  
  Mixin Architecture: The Engine
</h2>

<p>Engine combines four specialized mixins via multiple inheritance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">Engine</span><span class="p">(</span><span class="n">DBMS</span><span class="p">,</span> <span class="n">Controller</span><span class="p">,</span> <span class="n">Tools</span><span class="p">,</span> <span class="n">Launcher</span><span class="p">,</span> <span class="n">metaclass</span><span class="o">=</span><span class="n">_EngineMeta</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    Mixin Architecture (in MRO order):
        1. DBMS: Database connection and queries
        2. Controller: SQL builders and domain logic
        3. Tools: Shared utilities
        4. Launcher: Cross-platform file opener
    </span><span class="sh">"""</span>
    <span class="k">pass</span>
</code></pre>

</div>



<p>Each mixin has a single responsibility:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Mixin</th>
<th>Responsibility</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>DBMS</code></td>
<td>
<code>read()</code>, <code>write()</code>, <code>execute()</code>
</td>
</tr>
<tr>
<td><code>Controller</code></td>
<td>
<code>get_stock()</code>, <code>build_sql()</code>, domain queries</td>
</tr>
<tr>
<td><code>Tools</code></td>
<td>
<code>center_window()</code>, <code>set_icon()</code>, validation</td>
</tr>
<tr>
<td><code>Launcher</code></td>
<td>
<code>launch()</code> — open files with default app</td>
</tr>
</tbody>
</table></div>

<p>Views access everything through one interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># In any view
</span><span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">read</span><span class="p">(</span><span class="bp">True</span><span class="p">,</span> <span class="n">sql</span><span class="p">,</span> <span class="n">params</span><span class="p">)</span>      <span class="c1"># From DBMS
</span><span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">get_stock</span><span class="p">()</span>                   <span class="c1"># From Controller
</span><span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">center_window</span><span class="p">(</span><span class="n">self</span><span class="p">)</span>           <span class="c1"># From Tools
</span><span class="n">self</span><span class="p">.</span><span class="n">engine</span><span class="p">.</span><span class="nf">launch</span><span class="p">(</span><span class="n">filepath</span><span class="p">)</span>              <span class="c1"># From Launcher
</span></code></pre>

</div>






<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li><p><strong>Observer eliminates lateral coupling</strong> — publishers don't know subscribers, subscribers don't know publishers. Just events.</p></li>
<li><p><strong>Singleton has multiple valid implementations</strong> — <code>__new__</code> for simplicity, metaclass for purity, registry for context-sensitive uniqueness.</p></li>
<li><p><strong>Template Method creates consistency</strong> — every window follows the same lifecycle, reducing cognitive load.</p></li>
<li><p><strong>Mixins compose behavior</strong> — instead of deep inheritance hierarchies, combine focused capabilities.</p></li>
<li><p><strong>Real code teaches better than tutorials</strong> — patterns emerge from solving actual problems, not from contrived examples.</p></li>
</ol>




<h2>
  
  
  Try It Yourself
</h2>

<p>The full source is on GitHub: <strong><a href="https://github.com/1966bc/inventarium" rel="noopener noreferrer">github.com/1966bc/inventarium</a></strong></p>

<p>It's a working application managing real laboratory inventory — proof that Python's "batteries included" philosophy extends beyond tutorials to real-world, maintainable software.</p>

<p>Browse the code, steal the patterns, file issues if you find bugs.</p>

<p><strong>Python + Tkinter + SQLite + Design Patterns = Production-ready applications.</strong></p>




<p><em>Giuseppe Costanzi (<a href="https://github.com/1966bc" rel="noopener noreferrer">@1966bc</a>) is a biomedical laboratory technician at Sant'Andrea University Hospital, Mass Spectrometry Lab, in Rome. Inventarium is his solution for managing IVD product inventory.</em></p>

<p><em>This project was developed in collaboration with Claude (Anthropic). The AI pair-programming approach allowed focusing on architecture and functionality rather than getting bogged down in boilerplate code and syntax details — a workflow that proved transformative for development productivity.</em></p>

