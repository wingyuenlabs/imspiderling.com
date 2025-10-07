---
Title: Who wants to build an admin UI?
Description: 
Author: aaronblondeau
Date: 2025-10-07T21:48:27.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://pocketbase.io/" rel="noopener noreferrer">PocketBase</a> is my favorite tool for rapidly prototyping app ideas. One of the main things that I love about it is the extremely capable admin UI it comes with. Aside from asking non-technical users to edit JSON columns you can get really far with it.</p>

<p><strong>Having an admin UI for free saves enormous amounts of development time and lets me focus on other more MVP critical areas.</strong></p>

<p>I discovered <a href="https://kottster.app/" rel="noopener noreferrer">Kottster</a> a few weeks ago and decided to try it on one of my non-PocketBase projects. However, I abandoned it pretty quickly because at the time you had to login with their hosted authentication service and there was no way to completely self host it. They must have heard my loud sigh of exasperation because they recently announced full-self hosting support : (<a href="https://kottster.app/blog/kottster-is-now-fully-self-hosted)%5Bhttps://kottster.app/blog/kottster-is-now-fully-self-hosted%5D" rel="noopener noreferrer">https://kottster.app/blog/kottster-is-now-fully-self-hosted)[https://kottster.app/blog/kottster-is-now-fully-self-hosted]</a></p>

<p>Huge props to the Kottster team for recognizing that as a roadblock to adoption and addressing it!</p>

<p>Like the PocketBase admin UI, Kottster gets you really far out of the box. You can view and edit the tables in your database and you can also create dashboards for displaying query results.</p>

<p>Kottster operates in a neat way where as you create dashboards or table views in your instance it creates the code for them within your project's directory. This allows you to easily customize them as well as keep them in a code repo.</p>

<p>Kottster also allows you to create custom admin pages where you are set loose with a react component for the frontend page content, and a server side controller for the backend.</p>

<p>Here is what some of the server side code looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">app</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@/_server/app</span><span class="dl">'</span><span class="p">;</span>

<span class="cm">/*
 * Custom server procedures for your page
 * 
 * These functions run on the server and can be called from your React components
 * using callProcedure('procedureName', input)
 * 
 * Learn more: https://kottster.app/docs/custom-pages/api
 */</span>

<span class="kd">const</span> <span class="nx">controller</span> <span class="o">=</span> <span class="nx">app</span><span class="p">.</span><span class="nf">defineCustomController</span><span class="p">({</span>
  <span class="c1">// Define your procedures here</span>
  <span class="c1">// For example:</span>
  <span class="na">getMessage</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="na">input</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="s2">`Hello, </span><span class="p">${</span><span class="nx">input</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">!`</span> <span class="p">};</span>
  <span class="p">},</span>
  <span class="na">getSetting</span><span class="p">:</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// .env files work just fine</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">value</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">WIDGET_SETTING</span> <span class="p">}</span>
  <span class="p">},</span>
  <span class="na">getWidgets</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="na">input</span><span class="p">:</span> <span class="p">{</span> <span class="na">page</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="na">perPage</span><span class="p">:</span> <span class="kr">number</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// I had to reverse engineer things to create this code,</span>
    <span class="c1">// but direct access to the db via knex is available!</span>

    <span class="c1">// Another cool note is that the input and output types of</span>
    <span class="c1">// procedures are available in the frontend react component!</span>

    <span class="kd">const</span> <span class="nx">pg</span> <span class="o">=</span> <span class="nx">app</span><span class="p">.</span><span class="nx">dataSources</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>
    <span class="kd">const</span> <span class="nx">client</span> <span class="o">=</span> <span class="nx">pg</span><span class="p">.</span><span class="nx">adapter</span><span class="p">.</span><span class="nx">client</span>
    <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">widgets</span><span class="dl">'</span><span class="p">).</span><span class="nf">select</span><span class="p">(</span><span class="dl">'</span><span class="s1">*</span><span class="dl">'</span><span class="p">).</span><span class="nf">limit</span><span class="p">(</span><span class="nx">input</span><span class="p">.</span><span class="nx">perPage</span><span class="p">)</span>
    <span class="kd">const</span> <span class="nx">totalResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">client</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">widgets</span><span class="dl">'</span><span class="p">).</span><span class="nf">count</span><span class="p">(</span><span class="dl">'</span><span class="s1">*</span><span class="dl">'</span><span class="p">)</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">widgets</span><span class="p">:</span> <span class="nx">results</span> <span class="k">as</span> <span class="p">{</span>
      <span class="na">id</span><span class="p">:</span> <span class="kr">string</span>
      <span class="na">name</span><span class="p">:</span> <span class="kr">string</span>
      <span class="na">description</span><span class="p">:</span> <span class="kr">string</span>
      <span class="c1">// A json col...</span>
      <span class="na">options</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">label</span><span class="p">:</span> <span class="kr">string</span>
        <span class="na">value</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kr">number</span> <span class="o">|</span> <span class="nx">boolean</span>
      <span class="p">}[]</span>
      <span class="na">created</span><span class="p">:</span> <span class="nb">Date</span>
      <span class="na">updated</span><span class="p">:</span> <span class="nb">Date</span>
    <span class="p">}[],</span> <span class="na">totalWidgets</span><span class="p">:</span> <span class="nx">totalResult</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">count</span> <span class="p">}</span>
  <span class="p">}</span>
<span class="p">});</span>

<span class="k">export</span> <span class="k">default</span> <span class="nx">controller</span><span class="p">;</span>
<span class="k">export</span> <span class="kd">type</span> <span class="nx">Procedures</span> <span class="o">=</span> <span class="k">typeof</span> <span class="nx">controller</span><span class="p">.</span><span class="nx">procedures</span><span class="p">;</span>
</code></pre>

</div>



<p>Calling a backend procedure from the frontend React component is really easy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">widgets</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">callProcedure</span><span class="p">(</span><span class="dl">'</span><span class="s1">getWidgets</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">page</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">perPage</span><span class="p">:</span> <span class="mi">20</span> <span class="p">})</span>
</code></pre>

</div>



<p>My only blocker for using both Kottster and PocketBase more widely is a lack of ability to customize the editor for specific columns in the database. In my case I have a json column that needs to match a specific schema. For both Kottster and PocketBase these columns are edited as text and don't provide a way for me to drop in my own editor. I did create a <a href="https://github.com/kottster/kottster/issues/97" rel="noopener noreferrer">feature request</a> for this for the Kottster team though!</p>

<p>I'd highly recommend you give Kottster a try. It seems like a fairly young project but is on a good trajectory and solves a problem that I think a lot of devs have.</p>

