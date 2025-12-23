---
Title: Testing shadcn/ui components with TWD
Description: 
Author: Kevin Julián Martínez Escobar
Date: 2025-12-23T21:56:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’ve put together a small documentation site that shows <strong>how to test shadcn/ui components using TWD</strong>, built on top of <strong>Testing Library</strong>.</p>

<p>If you already use Testing Library, this means you can keep the same queries and assertions while gaining the benefits of Testing While Developing: running tests directly inside your real application as you build UI.</p>

<p>This approach helps:</p>

<ul>
<li>Validate real shadcn components in real screens</li>
<li>Turn manual UI checks into repeatable tests</li>
<li>Keep tests close to real user interactions</li>
<li>Reuse Testing Library knowledge and patterns</li>
</ul>

<p>You can find the docs and examples here: <a href="https://brikev.github.io/twd-shadcn/" rel="noopener noreferrer">https://brikev.github.io/twd-shadcn/</a></p>

<p>The goal is not to replace existing testing tools, but to make UI testing part of the <strong>everyday development workflow</strong>.</p>




<h2>
  
  
  Example: testing a Collapsible component
</h2>

<p>Given a simple shadcn/ui <code>Collapsible</code> component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Button</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@/components/ui/button</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span>
  <span class="nx">Collapsible</span><span class="p">,</span>
  <span class="nx">CollapsibleContent</span><span class="p">,</span>
  <span class="nx">CollapsibleTrigger</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@/components/ui/collapsible</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">function</span> <span class="nf">CollapsibleDemo</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="p">&lt;</span><span class="nc">Collapsible</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">CollapsibleTrigger</span> <span class="na">asChild</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nc">Button</span> <span class="na">variant</span><span class="p">=</span><span class="s">"outline"</span><span class="p">&gt;</span>Toggle content<span class="p">&lt;/</span><span class="nc">Button</span><span class="p">&gt;</span>
      <span class="p">&lt;/</span><span class="nc">CollapsibleTrigger</span><span class="p">&gt;</span>
      <span class="p">&lt;</span><span class="nc">CollapsibleContent</span><span class="p">&gt;</span>
        <span class="p">&lt;</span><span class="nt">div</span> <span class="na">className</span><span class="p">=</span><span class="s">"mt-4 p-4 border rounded-md"</span><span class="p">&gt;</span>
          Here is some hidden content
        <span class="p">&lt;/</span><span class="nt">div</span><span class="p">&gt;</span>
      <span class="p">&lt;/</span><span class="nc">CollapsibleContent</span><span class="p">&gt;</span>
    <span class="p">&lt;/</span><span class="nc">Collapsible</span><span class="p">&gt;</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>With TWD, the test focuses on real user behavior and uses familiar Testing Library-style queries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">twd</span><span class="p">,</span> <span class="nx">screenDom</span><span class="p">,</span> <span class="nx">userEvent</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">twd-js</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">describe</span><span class="p">,</span> <span class="nx">it</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">twd-js/runner</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Collapsible Component</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">toggles content visibility on trigger click</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">twd</span><span class="p">.</span><span class="nf">visit</span><span class="p">(</span><span class="dl">'</span><span class="s1">/collapsible</span><span class="dl">'</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">toggleButton</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">screenDom</span><span class="p">.</span><span class="nf">findByRole</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Toggle content</span><span class="dl">'</span> <span class="p">});</span>
    <span class="nx">twd</span><span class="p">.</span><span class="nf">should</span><span class="p">(</span><span class="nx">toggleButton</span><span class="p">,</span> <span class="dl">'</span><span class="s1">be.visible</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Initially collapsed (content not in DOM)</span>
    <span class="kd">let</span> <span class="nx">content</span> <span class="o">=</span> <span class="nx">screenDom</span><span class="p">.</span><span class="nf">queryByText</span><span class="p">(</span><span class="dl">'</span><span class="s1">Here is some hidden content</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">content</span><span class="p">).</span><span class="nf">eql</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>

    <span class="c1">// Open</span>
    <span class="k">await</span> <span class="nx">userEvent</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="nx">toggleButton</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">opened</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">screenDom</span><span class="p">.</span><span class="nf">findByText</span><span class="p">(</span><span class="dl">'</span><span class="s1">Here is some hidden content</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">twd</span><span class="p">.</span><span class="nf">should</span><span class="p">(</span><span class="nx">opened</span><span class="p">,</span> <span class="dl">'</span><span class="s1">be.visible</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Close</span>
    <span class="k">await</span> <span class="nx">userEvent</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="nx">toggleButton</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">closed</span> <span class="o">=</span> <span class="nx">screenDom</span><span class="p">.</span><span class="nf">queryByText</span><span class="p">(</span><span class="dl">'</span><span class="s1">Here is some hidden content</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">closed</span><span class="p">).</span><span class="nf">eql</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p>The test runs inside the real application, validating the component exactly as a user would interact with it.</p>

