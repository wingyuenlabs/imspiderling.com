---
Title: Day 11 of #100DaysOfCode — Understanding State Management in React
Description: 
Author: M Saad Ahmad
Date: 2026-02-12T21:40:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>At this point, I had covered almost every essential React topic: state, hooks, list rendering, conditional rendering, and more. So for <strong>Day 11 of my #100DaysOfCode</strong>, the goal was to finally introduce myself to <strong>state management in React</strong>.</p>

<p>At first glance, state management may look like some completely new React topic, but in reality, it's all about <strong>how you structure your app</strong> and <strong>how you handle states across different components</strong>. Chances are, if you're building a React app, you’re already using some kind of state management technique, whether you realize it or not.</p>

<p>Let’s take a closer look at the basics.</p>




<h2>
  
  
  🧠 What Is State Management in React?
</h2>

<p>In a React app, you generally deal with two categories of state:</p>

<h3>
  
  
  <strong>1. Local State</strong>
</h3>

<p>A state that <strong>only one component needs</strong>, and no other part of the app depends on it.</p>

<h3>
  
  
  <strong>2. Global State</strong>
</h3>

<p>A state that <strong>multiple components</strong> or even the <strong>entire React app</strong> needs.</p>




<h2>
  
  
  📌 Example: Todo List App
</h2>

<p>Suppose you're building a todo list app. The user types into an input to add items.</p>

<ul>
<li>Instead of creating the search/input state inside <code>App.jsx</code>, it's better to create a dedicated <code>Search.jsx</code> component and use:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="kd">const</span> <span class="p">[</span><span class="nx">search</span><span class="p">,</span> <span class="nx">setSearch</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">""</span><span class="p">);</span>
</code></pre>

</div>



<p>This stays as <strong>local state</strong>, because only the search bar needs it.</p>

<ul>
<li>But the list of todo items is something the <strong>entire app interacts with</strong>. So creating:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="kd">const</span> <span class="p">[</span><span class="nx">items</span><span class="p">,</span> <span class="nx">setItems</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">([]);</span>
</code></pre>

</div>



<p>inside <code>App.jsx</code> makes sense, because it's <strong>global state</strong> for your app.</p>




<h2>
  
  
  🔼 Lifting State (But Not Always Global)
</h2>

<p>If several components rely on the same piece of state, you move that state into a common parent component and pass it down as props. This is called <strong>lifting state up</strong>.</p>

<p>But remember:</p>

<blockquote>
<p><strong>Lifting state ≠ making it global.</strong><br>
It just means you moved it to the nearest shared parent.</p>
</blockquote>




<h2>
  
  
  🛒 Example: Shopping Website
</h2>

<p>In a shopping app, you might have many different states such as search, items, filters, cart, wish list, etc. Structuring these states properly is the core of <strong>state management</strong>.</p>

<ul>
<li>Place state <strong>where it logically belongs</strong>
</li>
<li>Keep related logic close to the component that needs it</li>
<li>Avoid unnecessary states in <code>App.jsx</code>
</li>
<li>If passing props down multiple layers becomes “too much,” use the <strong>Context API</strong> to avoid prop drilling</li>
</ul>

<p>All of this <em>is</em> state management.</p>




<h2>
  
  
  📦 External Libraries: Zustand, Redux, Jotai, etc.
</h2>

<p>External state management libraries like <strong>Zustand</strong>, <strong>Redux</strong>, or <strong>Jotai</strong> exist to solve problems that appear in <strong>larger, more complex apps</strong>.</p>

<p>Use them when:</p>

<ul>
<li>The app becomes difficult to manage with just <code>useState</code> or <code>useReducer</code>
</li>
<li>Context API becomes messy or too nested (“Context Hell”)</li>
<li>You begin facing performance bottlenecks from unnecessary re-renders</li>
</ul>

<p>For many apps:</p>

<blockquote>
<p><strong>Context + useState is enough.</strong><br>
You don’t need Redux or Zustand unless your app truly needs them.</p>
</blockquote>




<h2>
  
  
  ❓ Should I Care Where I Create State?
</h2>

<p>For small apps, you don’t need to stress too much.<br>
But it <em>is</em> best practice to separate <strong>local</strong> and <strong>global</strong> state properly.</p>

<p>Care about where you create state if:</p>

<ul>
<li>You find <strong>duplicate states</strong>
</li>
<li>You notice <strong>prop drilling</strong>
</li>
<li>You see confusion about where a state “lives”</li>
<li>Components depend on data they shouldn’t have to know about</li>
</ul>




<h2>
  
  
  ❓ Should I Use the Context API?
</h2>

<p>Use it when:</p>

<ul>
<li>There is <strong>too much prop drilling</strong>
</li>
<li>Multiple unrelated components need the same global state</li>
</ul>

<p>If your app doesn’t suffer from this, you don’t need Context.</p>




<h2>
  
  
  ❓ So If These Techniques Work, Why Use Zustand or Redux?
</h2>

<p>Honestly?<br>
For many apps, you <strong>don’t need</strong> external libraries.</p>

<p>But when your React app grows, you may face:</p>

<ul>
<li>Performance issues</li>
<li>Deeply nested context providers</li>
<li>Re-renders caused by large global states</li>
<li>Difficulty scaling or organizing state logic</li>
</ul>

<p>That’s when Zustand or Redux becomes helpful.</p>




<h2>
  
  
  🎯 Final Thoughts
</h2>

<p>The idea is not to bloat your React app with Context, Zustand, Redux, React Router, and whatnot. Keep your React app <strong>simple first</strong>, and only scale the state management system when your app actually demands it.</p>

<p><strong>Day 11 was all about observing how to manage a React app properly, and as it grows.</strong></p>

<p>Happy coding!</p>

