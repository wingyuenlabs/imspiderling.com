---
Title: Type-Safe CustomEvents: Better Messaging with Native APIs
Description: 
Author: Andrew Bone
Date: 2026-02-28T21:49:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>The native <code>EventTarget</code> is a hidden gem for internal messaging. It is built-in, fast and stays out of your way. However, the standard <code>CustomEvent</code> interface is a bit of a "black box" for TypeScript. Usually, you end up casting <code>e as CustomEvent&lt;MyData&gt;</code> every time you want to access your data.</p>

<p>We can do better. By wrapping the target and the event creation, we can have a completely type-safe event bus with zero runtime overhead.</p>

<h2>
  
  
  The Problem
</h2>

<p>When you dispatch a <code>CustomEvent</code>, the data is tucked away in the <code>detail</code> property. Out of the box, <code>addEventListener</code> has no idea what that detail contains, forcing you to manually type your listeners.</p>

<h2>
  
  
  The Solution: TypedEventTarget
</h2>

<p>We can use a generic map to link event names to their specific <code>CustomEvent</code> payloads.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">EventListener</span><span class="o">&lt;</span><span class="nx">E</span> <span class="kd">extends</span> <span class="nx">Event</span><span class="o">&gt;</span> <span class="o">=</span> <span class="p">(</span><span class="nx">evt</span><span class="p">:</span> <span class="nx">E</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">EventListenerObject</span><span class="o">&lt;</span><span class="nx">E</span> <span class="kd">extends</span> <span class="nx">Event</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="nf">handleEvent</span><span class="p">(</span><span class="na">evt</span><span class="p">:</span> <span class="nx">CustomEvent</span><span class="o">&lt;</span><span class="nx">E</span><span class="o">&gt;</span><span class="p">):</span> <span class="k">void</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// The type of our listener receives the CustomEvent with our specific data</span>
<span class="kd">type</span> <span class="nx">TEL</span><span class="o">&lt;</span><span class="nx">E</span><span class="o">&gt;</span> <span class="o">=</span> <span class="nx">EventListener</span><span class="o">&lt;</span><span class="nx">CustomEvent</span><span class="o">&lt;</span><span class="nx">E</span><span class="o">&gt;&gt;</span> <span class="o">|</span> <span class="nx">EventListenerObject</span><span class="o">&lt;</span><span class="nx">CustomEvent</span><span class="o">&lt;</span><span class="nx">E</span><span class="o">&gt;&gt;</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">TypedEventTarget</span><span class="o">&lt;</span><span class="nx">M</span> <span class="kd">extends</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">unknown</span><span class="o">&gt;&gt;</span> <span class="p">{</span>
  <span class="k">private</span> <span class="k">readonly</span> <span class="nx">target</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">EventTarget</span><span class="p">();</span>

  <span class="nx">addEventListener</span><span class="o">&lt;</span><span class="nx">K</span> <span class="kd">extends</span> <span class="kr">keyof</span> <span class="nx">M</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="na">type</span><span class="p">:</span> <span class="nx">K</span> <span class="o">&amp;</span> <span class="kr">string</span><span class="p">,</span>
    <span class="na">listener</span><span class="p">:</span> <span class="nx">TEL</span><span class="o">&lt;</span><span class="nx">M</span><span class="p">[</span><span class="nx">K</span><span class="p">]</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="nx">options</span><span class="p">?:</span> <span class="nx">boolean</span> <span class="o">|</span> <span class="nx">AddEventListenerOptions</span><span class="p">,</span>
  <span class="p">)</span> <span class="p">{</span>
    <span class="c1">// We cast to EventListenerOrEventListenerObject because the browser expects the base Event type</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="kd">type</span><span class="p">,</span> <span class="nx">listener</span> <span class="k">as</span> <span class="nx">EventListenerOrEventListenerObject</span><span class="p">,</span> <span class="nx">options</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="nx">removeEventListener</span><span class="o">&lt;</span><span class="nx">K</span> <span class="kd">extends</span> <span class="kr">keyof</span> <span class="nx">M</span><span class="o">&gt;</span><span class="p">(</span>
    <span class="na">type</span><span class="p">:</span> <span class="nx">K</span> <span class="o">&amp;</span> <span class="kr">string</span><span class="p">,</span>
    <span class="na">listener</span><span class="p">:</span> <span class="nx">TEL</span><span class="o">&lt;</span><span class="nx">M</span><span class="p">[</span><span class="nx">K</span><span class="p">]</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="nx">options</span><span class="p">?:</span> <span class="nx">boolean</span> <span class="o">|</span> <span class="nx">EventListenerOptions</span><span class="p">,</span>
  <span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nf">removeEventListener</span><span class="p">(</span><span class="kd">type</span><span class="p">,</span> <span class="nx">listener</span> <span class="k">as</span> <span class="nx">EventListenerOrEventListenerObject</span><span class="p">,</span> <span class="nx">options</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// A helper to ensure we always dispatch a properly formatted CustomEvent</span>
  <span class="nx">dispatchEvent</span><span class="o">&lt;</span><span class="nx">K</span> <span class="kd">extends</span> <span class="kr">keyof</span> <span class="nx">M</span><span class="o">&gt;</span><span class="p">(</span><span class="na">type</span><span class="p">:</span> <span class="nx">K</span><span class="p">,</span> <span class="p">...</span><span class="na">args</span><span class="p">:</span> <span class="nx">M</span><span class="p">[</span><span class="nx">K</span><span class="p">]</span> <span class="kd">extends</span> <span class="k">void</span> <span class="p">?</span> <span class="p">[</span><span class="nx">detail</span><span class="p">?:</span> <span class="kc">undefined</span><span class="p">]</span> <span class="p">:</span> <span class="p">[</span><span class="na">detail</span><span class="p">:</span> <span class="nx">M</span><span class="p">[</span><span class="nx">K</span><span class="p">]])</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="p">[</span><span class="nx">detail</span><span class="p">]</span> <span class="o">=</span> <span class="nx">args</span><span class="p">;</span>

    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">(</span><span class="k">new</span> <span class="nc">CustomEvent</span><span class="p">(</span><span class="nc">String</span><span class="p">(</span><span class="kd">type</span><span class="p">),</span> <span class="p">{</span> <span class="nx">detail</span> <span class="p">}));</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Of course this is only a base class, we'll need to extend it in order to use it with our own data.</p>

<h2>
  
  
  Real-world usage
</h2>

<p>Let's imagine a situation where we want to track an online shopping cart. We want to be able to make changes to the cart from anywhere in the site, but we don't want the UI or the total price to go out of sync.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TypedEventTarget</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./TypedEventTarget</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kr">interface</span> <span class="nx">CartItem</span> <span class="p">{</span>
  <span class="nl">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">price</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
<span class="p">}</span>

<span class="c1">// This map defines exactly what data each event emits</span>
<span class="kd">type</span> <span class="nx">ShoppingCartEvents</span> <span class="o">=</span> <span class="p">{</span>
  <span class="dl">'</span><span class="s1">item-added</span><span class="dl">'</span><span class="p">:</span> <span class="nx">CartItem</span><span class="p">;</span>
  <span class="dl">'</span><span class="s1">item-removed</span><span class="dl">'</span><span class="p">:</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="kr">string</span> <span class="p">};</span>
  <span class="dl">'</span><span class="s1">cart-cleared</span><span class="dl">'</span><span class="p">:</span> <span class="k">void</span><span class="p">;</span>
<span class="p">};</span>

<span class="k">export</span> <span class="k">default</span> <span class="kd">class</span> <span class="nc">ShoppingCart</span> <span class="kd">extends</span> <span class="nc">TypedEventTarget</span><span class="o">&lt;</span><span class="nx">ShoppingCartEvents</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">private</span> <span class="na">_items</span><span class="p">:</span> <span class="nx">CartItem</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="nf">addItem</span><span class="p">(</span><span class="na">item</span><span class="p">:</span> <span class="nx">CartItem</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">item</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">(</span><span class="dl">'</span><span class="s1">item-added</span><span class="dl">'</span><span class="p">,</span> <span class="nx">item</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="nf">removeItem</span><span class="p">(</span><span class="na">id</span><span class="p">:</span> <span class="nx">CartItem</span><span class="p">[</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">])</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_items</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">_items</span><span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">item</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">item</span><span class="p">.</span><span class="nx">id</span> <span class="o">!==</span> <span class="nx">id</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">(</span><span class="dl">'</span><span class="s1">item-removed</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="nx">id</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">clear</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">_items</span> <span class="o">=</span> <span class="p">[];</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">dispatchEvent</span><span class="p">(</span><span class="dl">'</span><span class="s1">cart-cleared</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="kd">get</span> <span class="nf">items</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">_items</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="kd">get</span> <span class="nf">total</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">sum</span><span class="p">,</span> <span class="nx">item</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">sum</span> <span class="o">+</span> <span class="nx">item</span><span class="p">.</span><span class="nx">price</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now that we have a class to store our data, you can see we've added methods that dispatch events automatically. Because these events are strictly typed, we get full autocomplete and safety when listening to them:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">shoppingCartSingleton</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">item-added</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span><span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">item</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">detail</span><span class="p">;</span> <span class="c1">// types know we've got CartItem;</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">item</span><span class="p">)</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Demo
</h3>

<p>Here is a simple demo using the above class.</p>

<p><iframe src="https://codesandbox.io/embed/typedeventtarget-yvmyjj">
</iframe>
</p>

<h2>
  
  
  Signing-off
</h2>

<p>I've only really touched the surface of what's possible with this technique. We live in a world where there are many mainstream frameworks and each has a different way of doing things. I think we're on the cusp of seeing the return of the generalised singleton: one native JS/TS class that can be connected to the framework of your choice. But that's an opinion for another day and perhaps another post.</p>

<p>Thanks for reading! If you'd like to connect, here are my <a href="https://bsky.app/profile/link2twenty.bsky.social" rel="noopener noreferrer">BlueSky</a> and <a href="https://www.linkedin.com/in/andrew-bone-ba241b179/" rel="noopener noreferrer">LinkedIn</a> profiles. Come say hi 😊</p>

