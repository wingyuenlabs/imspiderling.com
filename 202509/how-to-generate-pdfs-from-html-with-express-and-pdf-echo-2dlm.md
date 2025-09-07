---
Title: How to Generate PDFs from HTML with Express and PDF Echo
Description: 
Author: Tomás Duclos
Date: 2025-09-07T20:33:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>One of the most common tasks in web applications is generating PDF documents: invoices, reports, tickets, receipts, and more.</p>

<p>Doing this from scratch can be tedious, but with HTML + CSS and a templating engine like Handlebars, the task becomes much easier. And if we add PDF Echo, we can seamlessly transform those HTML documents into ready-to-use PDFs for your users.</p>

<p>In this post, I’ll show you how to build a small project with Node.js + Express + Handlebars to generate an Invoice and convert it to PDF with <a href="https://pdfecho.com" rel="noopener noreferrer">PDF Echo</a>.</p>

<h1>
  
  
  1. Create the Node.js project
</h1>

<p>First, initialize a new project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>invoice-pdf
<span class="nb">cd </span>invoice-pdf
npm init <span class="nt">-y</span>
npm <span class="nb">install</span> <span class="nt">-E</span> express express-handlebars
</code></pre>

</div>



<h1>
  
  
  2. Configuring <code>package.json</code>
</h1>

<p>Before writing any code, let’s update your package.json to make sure Node.js runs in ESM (ECMAScript Modules) mode and that you have handy scripts for development and production.</p>

<p>In your <code>package.json</code>, add:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"module"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"scripts"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node --env-file=.env src/index.js"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"dev"</span><span class="p">:</span><span class="w"> </span><span class="s2">"node --watch --env-file=.env src/index.js"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<ul>
<li>
<code>"type"</code>: <code>"module"</code> allows us to use modern <code>import</code> / <code>export</code> syntax.</li>
<li>The <code>"start"</code> script runs the app normally.</li>
<li>The <code>"dev"</code> script runs the app in watch mode so it reloads automatically when you make changes.</li>
</ul>

<h1>
  
  
  3. Configure Express and Handlebars
</h1>

<p>In <code>src/app.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express</span><span class="dl">'</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">engine</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express-handlebars</span><span class="dl">'</span>
<span class="k">import</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">node:path</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">()</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">engine</span><span class="p">(</span><span class="dl">'</span><span class="s1">hbs</span><span class="dl">'</span><span class="p">,</span> <span class="nf">engine</span><span class="p">({</span> <span class="na">defaultLayout</span><span class="p">:</span> <span class="kc">false</span> <span class="p">}))</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="dl">'</span><span class="s1">view engine</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">hbs</span><span class="dl">'</span><span class="p">)</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="dl">'</span><span class="s1">views</span><span class="dl">'</span><span class="p">,</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="dl">'</span><span class="s1">src</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">views</span><span class="dl">'</span><span class="p">))</span>

<span class="k">export</span> <span class="p">{</span> <span class="nx">app</span> <span class="p">}</span>
</code></pre>

</div>



<p>In <code>src/index.js</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">app</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./app.js</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">??</span> <span class="mi">4000</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="dl">'</span><span class="s1">0.0.0.0</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">error</span> <span class="o">!==</span> <span class="kc">undefined</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">error</span><span class="p">)</span>
    <span class="k">return</span>
  <span class="p">}</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`SV ON PORT: </span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">)</span>
<span class="p">})</span>
</code></pre>

</div>



<h1>
  
  
  4. Create an Invoice template with Handlebars
</h1>

<p>Now let’s design the invoice template.<br>
Create a file at <code>src/views/invoice.hbs</code> and paste the following code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight handlebars"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
<span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"UTF-8"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1.0"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;title&gt;</span>Invoice<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;style&gt;</span>
    <span class="o">*</span> <span class="p">{</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">box-sizing</span><span class="p">:</span> <span class="n">border-box</span>
    <span class="p">}</span>

    <span class="nt">body</span> <span class="p">{</span>
      <span class="nl">font-family</span><span class="p">:</span> <span class="n">system-ui</span><span class="p">,</span> <span class="n">-apple-system</span><span class="p">,</span> <span class="n">BlinkMacSystemFont</span><span class="p">,</span> <span class="s2">'Segoe UI'</span><span class="p">,</span> <span class="n">Roboto</span><span class="p">,</span> <span class="n">Oxygen</span><span class="p">,</span> <span class="n">Ubuntu</span><span class="p">,</span> <span class="n">Cantarell</span><span class="p">,</span> <span class="s2">'Open Sans'</span><span class="p">,</span> <span class="s2">'Helvetica Neue'</span><span class="p">,</span> <span class="nb">sans-serif</span>
    <span class="p">}</span>

    <span class="nc">.wave-top</span> <span class="p">{</span>
      <span class="nl">position</span><span class="p">:</span> <span class="nb">absolute</span><span class="p">;</span>
      <span class="nl">top</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">left</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">right</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">height</span><span class="p">:</span> <span class="m">12rem</span><span class="p">;</span>
      <span class="nl">background</span><span class="p">:</span> <span class="n">linear-gradient</span><span class="p">(</span><span class="n">to</span> <span class="nb">right</span><span class="p">,</span> <span class="m">#F69474</span><span class="p">,</span> <span class="m">#ED2762</span><span class="p">);</span>
      <span class="nl">clip-path</span><span class="p">:</span> <span class="n">ellipse</span><span class="p">(</span><span class="m">65%</span> <span class="m">100%</span> <span class="n">at</span> <span class="m">0%</span> <span class="m">0%</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="nt">main</span> <span class="p">{</span>
      <span class="nl">position</span><span class="p">:</span> <span class="nb">relative</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">2rem</span> <span class="m">4rem</span><span class="p">;</span>

      <span class="err">header</span> <span class="err">{</span>
        <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
        <span class="nl">justify-content</span><span class="p">:</span> <span class="n">space-between</span><span class="p">;</span>
        <span class="nl">margin-bottom</span><span class="p">:</span> <span class="m">4rem</span><span class="p">;</span>

        <span class="err">.hero</span> <span class="err">{</span>
          <span class="err">h1</span> <span class="err">{</span>
            <span class="nl">font-size</span><span class="p">:</span> <span class="m">3rem</span><span class="p">;</span>
            <span class="nl">text-transform</span><span class="p">:</span> <span class="nb">uppercase</span><span class="p">;</span>
            <span class="nl">margin-bottom</span><span class="p">:</span> <span class="m">0.5rem</span><span class="p">;</span>
            <span class="nl">color</span><span class="p">:</span> <span class="m">#FFFFFF</span><span class="p">;</span>
          <span class="p">}</span>
          <span class="nt">h2</span> <span class="p">{</span>
            <span class="nl">font-weight</span><span class="p">:</span> <span class="m">400</span><span class="p">;</span>
            <span class="nl">font-size</span><span class="p">:</span> <span class="m">1rem</span><span class="p">;</span>

            <span class="err">span</span> <span class="err">{</span>
              <span class="nl">font-weight</span><span class="p">:</span> <span class="m">600</span><span class="p">;</span>
              <span class="nl">text-transform</span><span class="p">:</span> <span class="nb">uppercase</span><span class="p">;</span>
            <span class="p">}</span>
          <span class="err">}</span>
        <span class="err">}</span>

        <span class="nc">.company</span> <span class="p">{</span>
          <span class="err">span</span> <span class="err">{</span>
            <span class="nl">font-size</span><span class="p">:</span> <span class="m">2rem</span><span class="p">;</span>
            <span class="nl">margin-bottom</span><span class="p">:</span> <span class="m">1rem</span><span class="p">;</span>
            <span class="nl">display</span><span class="p">:</span> <span class="nb">block</span><span class="p">;</span>
          <span class="p">}</span>

          <span class="nt">ul</span> <span class="p">{</span>
            <span class="nl">list-style</span><span class="p">:</span> <span class="nb">none</span><span class="p">;</span>
            <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
            <span class="nl">flex-direction</span><span class="p">:</span> <span class="n">column</span><span class="p">;</span>
            <span class="py">gap</span><span class="p">:</span> <span class="m">0.25rem</span><span class="p">;</span>

            <span class="err">li</span> <span class="err">{</span>
              <span class="nl">font-size</span><span class="p">:</span> <span class="m">0.75rem</span>
            <span class="p">}</span>
          <span class="err">}</span>
        <span class="err">}</span>
      <span class="err">}</span>

      <span class="nc">.overview</span> <span class="p">{</span>
        <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
        <span class="nl">justify-content</span><span class="p">:</span> <span class="n">space-between</span><span class="p">;</span>
        <span class="py">gap</span><span class="p">:</span> <span class="m">1rem</span><span class="p">;</span>
        <span class="nl">margin-bottom</span><span class="p">:</span> <span class="m">2rem</span><span class="p">;</span>

        <span class="err">.box</span> <span class="err">{</span>
          <span class="nl">background</span><span class="p">:</span> <span class="n">rgba</span><span class="p">(</span><span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0.1</span><span class="p">);</span>
          <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
          <span class="nl">height</span><span class="p">:</span> <span class="m">150px</span><span class="p">;</span>
          <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
          <span class="nl">flex-direction</span><span class="p">:</span> <span class="n">column</span><span class="p">;</span>
          <span class="nl">justify-content</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
          <span class="nl">align-items</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>

          <span class="err">h4</span> <span class="err">{</span>
            <span class="nl">font-size</span><span class="p">:</span> <span class="m">1rem</span><span class="p">;</span>
          <span class="p">}</span>

          <span class="nt">span</span> <span class="p">{</span>
            <span class="nl">font-size</span><span class="p">:</span> <span class="m">0.75rem</span><span class="p">;</span>
          <span class="p">}</span>
        <span class="err">}</span>
      <span class="err">}</span>

      <span class="nt">table</span> <span class="p">{</span>
        <span class="nl">border-spacing</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
        <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>

        <span class="err">thead</span> <span class="err">{</span>
          <span class="nl">background</span><span class="p">:</span> <span class="n">linear-gradient</span><span class="p">(</span><span class="n">to</span> <span class="nb">right</span><span class="p">,</span> <span class="m">#EE2762</span><span class="p">,</span> <span class="m">#F89675</span><span class="p">);</span>

          <span class="err">tr</span> <span class="err">{</span>
            <span class="err">th</span> <span class="err">{</span>
              <span class="nl">padding</span><span class="p">:</span> <span class="m">1rem</span> <span class="m">2rem</span><span class="p">;</span>
              <span class="nl">text-transform</span><span class="p">:</span> <span class="nb">uppercase</span><span class="p">;</span>
              <span class="nl">color</span><span class="p">:</span> <span class="m">#FFFFFF</span><span class="p">;</span>
              <span class="nl">text-align</span><span class="p">:</span> <span class="nb">left</span><span class="p">;</span>

              <span class="err">&amp;:first-child</span> <span class="err">{</span>
                <span class="nl">border-top-left-radius</span><span class="p">:</span> <span class="m">999px</span><span class="p">;</span>
                <span class="nl">border-bottom-left-radius</span><span class="p">:</span> <span class="m">999px</span><span class="p">;</span>
              <span class="p">}</span>

              <span class="o">&amp;</span><span class="nd">:last-child</span> <span class="p">{</span>
                <span class="nl">border-top-right-radius</span><span class="p">:</span> <span class="m">999px</span><span class="p">;</span>
                <span class="nl">border-bottom-right-radius</span><span class="p">:</span> <span class="m">999px</span><span class="p">;</span>
              <span class="p">}</span>
            <span class="err">}</span>
          <span class="err">}</span>
        <span class="err">}</span>

        <span class="nt">tbody</span> <span class="p">{</span>
          <span class="err">tr</span> <span class="err">{</span>
            <span class="err">td</span> <span class="err">{</span>
              <span class="nl">padding</span><span class="p">:</span> <span class="m">1rem</span> <span class="m">2rem</span><span class="p">;</span>
              <span class="nl">font-weight</span><span class="p">:</span> <span class="m">600</span><span class="p">;</span>
            <span class="p">}</span>
          <span class="err">}</span>
        <span class="err">}</span>
      <span class="err">}</span>

      <span class="nt">hr</span> <span class="p">{</span>
        <span class="nl">margin</span><span class="p">:</span> <span class="m">1rem</span> <span class="m">0</span><span class="p">;</span>
      <span class="p">}</span>

      <span class="nc">.summary</span> <span class="p">{</span>
        <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
        <span class="nl">flex-direction</span><span class="p">:</span> <span class="n">column</span><span class="p">;</span>
        <span class="nl">align-items</span><span class="p">:</span> <span class="n">flex-end</span><span class="p">;</span>
        <span class="py">gap</span><span class="p">:</span> <span class="m">1rem</span><span class="p">;</span>

        <span class="err">.row</span> <span class="err">{</span>
          <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
          <span class="nl">align-items</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
          <span class="nl">justify-content</span><span class="p">:</span> <span class="n">space-between</span><span class="p">;</span>
          <span class="nl">width</span><span class="p">:</span> <span class="m">40%</span><span class="p">;</span>
          <span class="nl">min-width</span><span class="p">:</span> <span class="m">2rem</span><span class="p">;</span>

          <span class="err">h3</span> <span class="err">{</span>
            <span class="nl">text-align</span><span class="p">:</span> <span class="nb">right</span><span class="p">;</span>
            <span class="nl">min-width</span><span class="p">:</span> <span class="m">10rem</span><span class="p">;</span>
            <span class="nl">width</span><span class="p">:</span> <span class="m">50%</span><span class="p">;</span>
            <span class="nl">font-weight</span><span class="p">:</span> <span class="m">600</span><span class="p">;</span>
          <span class="p">}</span>

          <span class="nt">span</span> <span class="p">{</span>
            <span class="nl">font-weight</span><span class="p">:</span> <span class="m">600</span><span class="p">;</span>

            <span class="err">&amp;.fill</span> <span class="err">{</span>
              <span class="nl">background</span><span class="p">:</span> <span class="n">linear-gradient</span><span class="p">(</span><span class="n">to</span> <span class="nb">right</span><span class="p">,</span> <span class="m">#EE2762</span><span class="p">,</span> <span class="m">#F89675</span><span class="p">);</span>
              <span class="nl">padding</span><span class="p">:</span> <span class="m">0.5rem</span><span class="p">;</span>
              <span class="nl">color</span><span class="p">:</span> <span class="m">#FFFFFF</span><span class="p">;</span>
            <span class="p">}</span>
          <span class="err">}</span>
        <span class="err">}</span>
      <span class="err">}</span>
    <span class="err">}</span>
  <span class="nt">&lt;/style&gt;</span>
<span class="nt">&lt;/head&gt;</span>
<span class="nt">&lt;body&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"wave-top"</span><span class="nt">&gt;&lt;/div&gt;</span>
  <span class="nt">&lt;main&gt;</span>
    <span class="nt">&lt;header&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"hero"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h1&gt;</span>Invoice<span class="nt">&lt;/h1&gt;</span>
        <span class="nt">&lt;h2&gt;</span>Invoice to: <span class="nt">&lt;span&gt;</span><span class="k">{{</span><span class="nv">customer_name</span><span class="k">}}</span><span class="nt">&lt;/span&gt;&lt;/h2&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"company"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;span&gt;</span>LOGO<span class="nt">&lt;/span&gt;</span>
        <span class="nt">&lt;ul&gt;</span>
          <span class="nt">&lt;li&gt;</span>Address: Your Street No, 223 NY USA<span class="nt">&lt;/li&gt;</span>
          <span class="nt">&lt;li&gt;</span>Phone: <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"tel:+1234567890"</span><span class="nt">&gt;</span>+123 456 7890<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
          <span class="nt">&lt;li&gt;</span>Email: <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"mailto:your@email.com"</span><span class="nt">&gt;</span>your@email.com<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
          <span class="nt">&lt;li&gt;</span>Website: <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"https://example.com"</span><span class="nt">&gt;</span>example.com<span class="nt">&lt;/a&gt;&lt;/li&gt;</span>
        <span class="nt">&lt;/ul&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/header&gt;</span>

    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"overview"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"box"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h4&gt;</span>Date<span class="nt">&lt;/h4&gt;</span>
        <span class="nt">&lt;span&gt;</span><span class="k">{{</span><span class="nv">created_at</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"box"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h4&gt;</span>Service NO.<span class="nt">&lt;/h4&gt;</span>
        <span class="nt">&lt;span&gt;</span><span class="k">{{</span><span class="nv">service_no</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"box"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h4&gt;</span>Account NO.<span class="nt">&lt;/h4&gt;</span>
        <span class="nt">&lt;span&gt;</span><span class="k">{{</span><span class="nv">account_no</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"box"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h4&gt;</span>Amount<span class="nt">&lt;/h4&gt;</span>
        <span class="nt">&lt;span&gt;</span>$<span class="k">{{</span><span class="nv">sum</span> <span class="nv">total_amount</span> <span class="p">(</span><span class="nv">calculateTax</span> <span class="nv">tax</span> <span class="nv">total_amount</span><span class="p">)</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>

    <span class="nt">&lt;table&gt;</span>
      <span class="nt">&lt;thead&gt;</span>
        <span class="nt">&lt;tr&gt;</span>
          <span class="nt">&lt;th&gt;</span>Item Description<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;th&gt;</span>Price<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;th&gt;</span>QTY<span class="nt">&lt;/th&gt;</span>
          <span class="nt">&lt;th&gt;</span>Total<span class="nt">&lt;/th&gt;</span>
        <span class="nt">&lt;/tr&gt;</span>
      <span class="nt">&lt;/thead&gt;</span>
      <span class="nt">&lt;tbody&gt;</span>
        <span class="k">{{#</span><span class="nn">each</span> <span class="nv">items</span><span class="k">}}</span>
          <span class="nt">&lt;tr&gt;</span>
            <span class="nt">&lt;td&gt;</span><span class="k">{{</span><span class="nv">name</span><span class="k">}}</span><span class="nt">&lt;/td&gt;</span>
            <span class="nt">&lt;td&gt;</span>$<span class="k">{{</span><span class="nv">unit_price</span><span class="k">}}</span><span class="nt">&lt;/td&gt;</span>
            <span class="nt">&lt;td&gt;</span><span class="k">{{</span><span class="nv">quantity</span><span class="k">}}</span><span class="nt">&lt;/td&gt;</span>
            <span class="nt">&lt;td&gt;</span>$<span class="k">{{</span><span class="nv">multiply</span> <span class="nv">unit_price</span> <span class="nv">quantity</span><span class="k">}}</span><span class="nt">&lt;/td&gt;</span>
          <span class="nt">&lt;/tr&gt;</span>
        <span class="k">{{/</span><span class="nn">each</span><span class="k">}}</span>
      <span class="nt">&lt;/tbody&gt;</span>
    <span class="nt">&lt;/table&gt;</span>

    <span class="nt">&lt;hr</span> <span class="nt">/&gt;</span>

    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"summary"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h3&gt;</span>Subtotal<span class="nt">&lt;/h3&gt;</span>
        <span class="nt">&lt;span&gt;</span>$<span class="k">{{</span><span class="nv">total_amount</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h3&gt;</span>VAT/TAX <span class="k">{{</span><span class="nv">tax</span><span class="k">}}</span>%<span class="nt">&lt;/h3&gt;</span>
        <span class="nt">&lt;span&gt;</span>$<span class="k">{{</span><span class="nv">calculateTax</span> <span class="nv">tax</span> <span class="nv">total_amount</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"row"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h3&gt;</span>Total Due<span class="nt">&lt;/h3&gt;</span>
        <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"fill"</span><span class="nt">&gt;</span>$<span class="k">{{</span><span class="nv">sum</span> <span class="nv">total_amount</span> <span class="p">(</span><span class="nv">calculateTax</span> <span class="nv">tax</span> <span class="nv">total_amount</span><span class="p">)</span><span class="k">}}</span><span class="nt">&lt;/span&gt;</span>
      <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/main&gt;</span>
<span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<p>Notice this template uses custom helpers:</p>

<ul>
<li>
<code>multiply</code> → to calculate line totals</li>
<li>
<code>calculateTax</code> → to apply tax percentage</li>
<li>
<code>sum</code> → to add subtotal + tax</li>
</ul>

<p>Before we can render this template, we need to register these helpers in our Express app.</p>

<p>In <code>src/app.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">app</span><span class="p">.</span><span class="nf">engine</span><span class="p">(</span><span class="dl">'</span><span class="s1">hbs</span><span class="dl">'</span><span class="p">,</span> <span class="nf">engine</span><span class="p">({</span>
  <span class="na">defaultLayout</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
  <span class="na">helpers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">sum</span><span class="p">:</span> <span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span> <span class="o">+</span> <span class="nx">b</span><span class="p">,</span>
    <span class="na">multiply</span><span class="p">:</span> <span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span> <span class="o">*</span> <span class="nx">b</span><span class="p">,</span>
    <span class="na">calculateTax</span><span class="p">:</span> <span class="p">(</span><span class="nx">tax</span><span class="p">,</span> <span class="nx">totalAmount</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="nx">totalAmount</span> <span class="o">*</span> <span class="p">(</span><span class="nx">tax</span> <span class="o">/</span> <span class="mi">100</span><span class="p">))</span>
  <span class="p">}</span>
<span class="p">}))</span>
</code></pre>

</div>



<p>Now Handlebars knows how to handle those operations, and the template will render correctly.</p>

<h1>
  
  
  4. Render the template in a route
</h1>

<p>Before integrating with <a href="https://pdfecho.com" rel="noopener noreferrer">PDF Echo</a>, let’s make sure that our Handlebars template is rendering correctly in the browser. This way, we can quickly spot any issues with the template or helpers before moving on.</p>

<p>Create a simple route in your Express server that compiles the template with some mock data and returns the rendered HTML:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/invoice</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">MOCKUP_DATA</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">inv_cmeyqpbks000008iccyi9fd20</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">customer_name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Joe Doe</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">account_no</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CUST-12312332</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">service_no</span><span class="p">:</span> <span class="dl">'</span><span class="s1">SRV-123123213</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">items</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Graphic Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">125</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">2</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Web Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">150</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Branding Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Brochure Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span>
    <span class="p">],</span>
    <span class="na">created_at</span><span class="p">:</span> <span class="dl">'</span><span class="s1">30-08-2025</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">tax</span><span class="p">:</span> <span class="mf">7.5</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">totalAmount</span> <span class="o">=</span> <span class="nx">MOCKUP_DATA</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">acc</span><span class="p">,</span> <span class="nx">value</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">acc</span> <span class="o">+</span> <span class="nx">value</span><span class="p">.</span><span class="nx">unit_price</span> <span class="o">*</span> <span class="nx">value</span><span class="p">.</span><span class="nx">quantity</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span>

  <span class="nx">res</span><span class="p">.</span><span class="nf">render</span><span class="p">(</span><span class="dl">'</span><span class="s1">invoice</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="p">...</span><span class="nx">MOCKUP_DATA</span><span class="p">,</span>
    <span class="na">total_amount</span><span class="p">:</span> <span class="nx">totalAmount</span>
  <span class="p">})</span>
<span class="p">})</span>
</code></pre>

</div>



<p>And visit <a href="http://localhost:4000/invoice" rel="noopener noreferrer">http://localhost:4000/invoice</a> in your browser.<br>
You should see the rendered invoice with your mock data 🎉.</p>

<p>👉 This step is optional but very useful. It helps you confirm that:</p>

<ul>
<li>Your Handlebars template compiles without errors.</li>
<li>Your custom helpers (from Step 3) are working correctly.</li>
<li>The HTML layout looks good before exporting it to PDF.</li>
</ul>

<p>Once everything looks fine in the browser, we’ll move on to connecting with PDF Echo to generate a real PDF.</p>
<h1>
  
  
  5. Integrate with PDF Echo to Generate a PDF
</h1>

<p>Now that we’ve verified that our template renders correctly in the browser, it’s time to generate a real PDF using <a href="https://pdfecho.com" rel="noopener noreferrer">PDF Echo</a>.</p>

<p><a href="https://pdfecho.com" rel="noopener noreferrer">PDF Echo</a> allows you to send HTML and receive a fully rendered PDF — no need to worry about Puppeteer, wkhtmltopdf, or other heavy engines. To use the API, you'll need an API key. You can get yours for free by signing up on our website and navigating to your dashboard. For a detailed guide, please refer to our <a href="https://docs.pdfecho.com/quickstart" rel="noopener noreferrer">Quickstart Documentation</a>.</p>

<p>Create a new route that sends the rendered HTML to PDF Echo and returns the PDF to the client:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/invoice/pdf</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">MOCKUP_DATA</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">inv_cmeyqpbks000008iccyi9fd20</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">customer_name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Joe Doe</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">account_no</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CUST-12312332</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">service_no</span><span class="p">:</span> <span class="dl">'</span><span class="s1">SRV-123123213</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">items</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Graphic Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">125</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">2</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Web Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">150</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Branding Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">1</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Brochure Design</span><span class="dl">'</span><span class="p">,</span> <span class="na">unit_price</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span> <span class="na">quantity</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span>
    <span class="p">],</span>
    <span class="na">created_at</span><span class="p">:</span> <span class="dl">'</span><span class="s1">30-08-2025</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">tax</span><span class="p">:</span> <span class="mf">7.5</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">totalAmount</span> <span class="o">=</span> <span class="nx">MOCKUP_DATA</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">reduce</span><span class="p">((</span><span class="nx">acc</span><span class="p">,</span> <span class="nx">value</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">acc</span> <span class="o">+</span> <span class="nx">value</span><span class="p">.</span><span class="nx">unit_price</span> <span class="o">*</span> <span class="nx">value</span><span class="p">.</span><span class="nx">quantity</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span>

  <span class="nx">res</span><span class="p">.</span><span class="nf">render</span><span class="p">(</span>
    <span class="dl">'</span><span class="s1">invoice</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">{</span>
      <span class="p">...</span><span class="nx">MOCKUP_DATA</span><span class="p">,</span>
      <span class="na">total_amount</span><span class="p">:</span> <span class="nx">totalAmount</span>
    <span class="p">},</span>
    <span class="k">async </span><span class="p">(</span><span class="nx">error</span><span class="p">,</span> <span class="nx">html</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">error</span> <span class="o">!==</span> <span class="kc">undefined</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="p">{</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">invalid_html</span><span class="dl">'</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">invalid_request_error</span><span class="dl">'</span> <span class="p">}</span> <span class="p">})</span>
      <span class="p">}</span>

      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">request</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://api.pdfecho.com</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
          <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
            <span class="na">source</span><span class="p">:</span> <span class="nx">html</span>
          <span class="p">}),</span>
          <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Basic </span><span class="p">${</span><span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">sk_test_****</span><span class="dl">'</span> <span class="o">+</span> <span class="dl">'</span><span class="s1">:</span><span class="dl">'</span><span class="p">).</span><span class="nf">toString</span><span class="p">(</span><span class="dl">'</span><span class="s1">base64</span><span class="dl">'</span><span class="p">)}</span><span class="s2">`</span><span class="p">,</span>
            <span class="dl">'</span><span class="s1">pe-test-mode</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">true</span><span class="dl">'</span>
          <span class="p">}</span>
        <span class="p">})</span>

        <span class="kd">const</span> <span class="nx">data</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">request</span><span class="p">.</span><span class="nf">arrayBuffer</span><span class="p">()</span>

        <span class="nx">res</span>
           <span class="p">.</span><span class="nf">contentType</span><span class="p">(</span><span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">application/pdf</span><span class="dl">'</span><span class="p">)</span>
           <span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">data</span><span class="p">))</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="p">{</span> <span class="na">code</span><span class="p">:</span> <span class="dl">'</span><span class="s1">server_internal_error</span><span class="dl">'</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">api_error</span><span class="dl">'</span> <span class="p">}</span> <span class="p">})</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">)</span>
<span class="p">})</span>
</code></pre>

</div>



<p>Let's break down the most important parts of this code:</p>

<p><code>res.render()</code> Callback: Instead of sending the rendered HTML directly, we're using a three-argument version of <code>res.render()</code>. The third argument is a callback function that executes once the template is fully rendered. This function receives two parameters: <code>error</code> and <code>html</code>. The html parameter is a string containing our final, rendered HTML, which is exactly what we need for the next step.</p>

<p><code>fetch()</code> to the PDF Echo API:</p>

<ul>
<li><p><code>method: 'POST'</code>: We send the request using a POST method because we are sending data (the HTML) to the API.</p></li>
<li><p><code>body: JSON.stringify({ source: html })</code>: The API expects a JSON object in the request body. This object must have a key called <code>source</code>, and its value is the <code>html</code> string we got from the <code>res.render</code> callback.</p></li>
<li><p><code>Authorization</code> header: This header is essential for authenticating your request with your API Key. We use basic authentication, encoding our API key in Base64.</p></li>
<li><p><code>'pe-test-mode': 'true'</code>: This header tells the API to use its test mode, which is great for development and testing without using up your production credits.</p></li>
</ul>

<p>Then open <a href="http://localhost:4000/invoice/pdf" rel="noopener noreferrer">http://localhost:4000/invoice/pdf</a> in your browser.<br>
You should see the PDF download automatically with your rendered invoice. 🎉</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fve3n9h6gdqbeb4pw3t1b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fve3n9h6gdqbeb4pw3t1b.png" alt=" " width="800" height="900"></a></p>

<p>Repository: <a href="https://github.com/pdf-echo/example-invoice-nodejs-express-hbs" rel="noopener noreferrer">https://github.com/pdf-echo/example-invoice-nodejs-express-hbs</a></p>

<p>In this tutorial, we set up an Express server and integrated it with PDF Echo to generate PDF documents in real time.</p>

<p>From here, you can:</p>

<ul>
<li><p>Explore the fields of the <a href="https://docs.pdfecho.com/api-reference/endpoint/create-pdf" rel="noopener noreferrer"><code>/v1/pdf</code> endpoint</a> to see what other options you have when generating the PDF.</p></li>
<li><p>Configure a webhook to automatically receive the PDF without needing to poll.</p></li>
<li><p>Store PDFs in services like AWS S3 or Google Cloud Storage, and work directly with the public or signed URL.</p></li>
<li><p>Extend your templates to practical use cases such as invoices, purchase orders, reports, or contracts — fully automated.</p></li>
<li><p>With this infrastructure, you now have a flexible backend that can integrate into any app or business workflow.</p></li>
</ul>

