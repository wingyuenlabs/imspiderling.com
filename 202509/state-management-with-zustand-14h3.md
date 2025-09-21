---
Title: 🏃‍♂️State management with Zustand!
Description: 
Author: Gustavo Isensee
Date: 2025-09-21T21:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Managing State in React with Zustand
</h2>

<p>State management in React can get pretty messy if you don’t pick the right tool. You start with <code>useState</code>, then you raise it up to <code>useContext</code>, then suddenly your app feels like it's participating in a circus and juggling props all over the place. That’s where <strong>Zustand</strong> strolls in, clean and minimal, like that one friend who doesn’t overcomplicate dinner plans.  </p>

<p>Let’s explore how to manage a <strong>list of users</strong> across multiple components using Zustand.</p>

<p> </p>

<h2>
  
  
  Why Zustand?
</h2>

<p>Zustand is a <strong>small but powerful</strong> state management library. Unlike Redux (which sometimes feels like filing taxes), Zustand has no boilerplate ceremony, just plain simple functions.  </p>

<p>Key benefits:  </p>

<ul>
<li>Tiny and fast 🏎️
</li>
<li>No reducers, actions, or ceremony 🎉
</li>
<li>Works great with React hooks 👌
</li>
</ul>

<p> </p>

<h2>
  
  
  Basic Requirements Recap
</h2>

<p>To run this little app, you just need:  </p>

<ul>
<li>React (v18+)
</li>
<li>Node &amp; npm installed
</li>
<li>
<code>zustand</code> package installed
</li>
</ul>

<p> </p>

<h2>
  
  
  Setting Up the Store
</h2>

<p>Let’s create a store for our users.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>zustand
</code></pre>

</div>



<p>Now, define a store:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// store/userStore.js</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">create</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zustand</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">useUserStore</span> <span class="o">=</span> <span class="nf">create</span><span class="p">((</span><span class="kd">set</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="na">users</span><span class="p">:</span> <span class="p">[],</span>

  <span class="na">addUser</span><span class="p">:</span> <span class="p">(</span><span class="nx">newUser</span><span class="p">)</span> <span class="o">=&gt;</span>
    <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
      <span class="na">users</span><span class="p">:</span> <span class="p">[...</span><span class="nx">state</span><span class="p">.</span><span class="nx">users</span><span class="p">,</span> <span class="nx">newUser</span><span class="p">],</span>
    <span class="p">})),</span>

  <span class="na">removeUser</span><span class="p">:</span> <span class="p">(</span><span class="nx">id</span><span class="p">)</span> <span class="o">=&gt;</span>
    <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
      <span class="na">users</span><span class="p">:</span> <span class="nx">state</span><span class="p">.</span><span class="nx">users</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">user</span> <span class="o">=&gt;</span> <span class="nx">user</span><span class="p">.</span><span class="nx">id</span> <span class="o">!==</span> <span class="nx">id</span><span class="p">),</span>
    <span class="p">})),</span>
<span class="p">}));</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">useUserStore</span><span class="p">;</span>
</code></pre>

</div>



<p>Here’s what’s happening:  </p>

<ul>
<li>
<code>users</code> holds our list of users.
</li>
<li>
<code>addUser</code> lets us push a new user.
</li>
<li>
<code>removeUser</code> helps us clean up users 👋.
</li>
</ul>

<p> </p>

<h2>
  
  
  Displaying the User List
</h2>

<p>We’ll need a component to <strong>show</strong> the users.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// components/UserList.js</span>
<span class="k">import</span> <span class="nx">React</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">useUserStore</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../store/userStore</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">UserList</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">users</span> <span class="o">=</span> <span class="nf">useUserStore</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">users</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">removeUser</span> <span class="o">=</span> <span class="nf">useUserStore</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">removeUser</span><span class="p">);</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">h2</span><span class="o">&gt;</span><span class="nx">User</span> <span class="nx">List</span><span class="o">&lt;</span><span class="sr">/h2</span><span class="err">&gt;
</span>      <span class="o">&lt;</span><span class="nx">ul</span><span class="o">&gt;</span>
        <span class="p">{</span><span class="nx">users</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">user</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span>
          <span class="o">&lt;</span><span class="nx">li</span> <span class="nx">key</span><span class="o">=</span><span class="p">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">}</span><span class="o">&gt;</span>
            <span class="p">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span> 
            <span class="o">&lt;</span><span class="nx">button</span> <span class="nx">onClick</span><span class="o">=</span><span class="p">{()</span> <span class="o">=&gt;</span> <span class="nf">removeUser</span><span class="p">(</span><span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">)}</span><span class="o">&gt;</span><span class="err">❌</span><span class="o">&lt;</span><span class="sr">/button</span><span class="err">&gt;
</span>          <span class="o">&lt;</span><span class="sr">/li</span><span class="err">&gt;
</span>        <span class="p">))}</span>
      <span class="o">&lt;</span><span class="sr">/ul</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">UserList</span><span class="p">;</span>
</code></pre>

</div>



<p>Here, we’re pulling users directly from the store with <code>useUserStore</code>. No prop drilling. No tears.  </p>

<p> </p>

<h2>
  
  
  Adding a New User
</h2>

<p>We also need a form (or a button) to add users.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// components/AddUser.js</span>
<span class="k">import</span> <span class="nx">React</span><span class="p">,</span> <span class="p">{</span> <span class="nx">useState</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">useUserStore</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../store/userStore</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">AddUser</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">name</span><span class="p">,</span> <span class="nx">setName</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="dl">''</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">addUser</span> <span class="o">=</span> <span class="nf">useUserStore</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">addUser</span><span class="p">);</span>

  <span class="kd">const</span> <span class="nx">handleSubmit</span> <span class="o">=</span> <span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">name</span><span class="p">.</span><span class="nf">trim</span><span class="p">())</span> <span class="k">return</span><span class="p">;</span>

    <span class="nf">addUser</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span> <span class="nx">name</span> <span class="p">});</span>
    <span class="nf">setName</span><span class="p">(</span><span class="dl">''</span><span class="p">);</span>
  <span class="p">};</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">form</span> <span class="nx">onSubmit</span><span class="o">=</span><span class="p">{</span><span class="nx">handleSubmit</span><span class="p">}</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">input</span>
        <span class="nx">value</span><span class="o">=</span><span class="p">{</span><span class="nx">name</span><span class="p">}</span>
        <span class="nx">onChange</span><span class="o">=</span><span class="p">{(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">setName</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">value</span><span class="p">)}</span>
        <span class="nx">placeholder</span><span class="o">=</span><span class="dl">"</span><span class="s2">Enter user name</span><span class="dl">"</span>
      <span class="o">/&gt;</span>
      <span class="o">&lt;</span><span class="nx">button</span> <span class="nx">type</span><span class="o">=</span><span class="dl">"</span><span class="s2">submit</span><span class="dl">"</span><span class="o">&gt;</span><span class="nx">Add</span> <span class="nx">User</span><span class="o">&lt;</span><span class="sr">/button</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/form</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">AddUser</span><span class="p">;</span>
</code></pre>

</div>



<p>Now you have a way to actually <em>add</em> new users.  </p>

<p> </p>

<h2>
  
  
  Putting It Together
</h2>

<p>Finally, use both components in your <code>App</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// App.js</span>
<span class="k">import</span> <span class="nx">React</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">react</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">AddUser</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./components/AddUser</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">UserList</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./components/UserList</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">App</span><span class="p">()</span> <span class="p">{</span>
  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">h1</span><span class="o">&gt;</span><span class="nx">Zustand</span> <span class="nx">User</span> <span class="nx">Management</span> <span class="err">🐻</span><span class="o">&lt;</span><span class="sr">/h1</span><span class="err">&gt;
</span>      <span class="o">&lt;</span><span class="nx">AddUser</span> <span class="o">/&gt;</span>
      <span class="o">&lt;</span><span class="nx">UserList</span> <span class="o">/&gt;</span>
    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">App</span><span class="p">;</span>
</code></pre>

</div>



<p> </p>

<h2>
  
  
  Final Thoughts
</h2>

<p>Zustand makes state management less painful. Instead of dealing with reducers and dispatchers (hello, Redux), you just <strong>call functions</strong>. Your code stays readable, and your state stays centralized.  </p>

