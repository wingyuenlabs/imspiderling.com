---
Title: Making sure your PDF compliance and privacy requirements are met with JoyDoc
Description: 
Author: John Pagley
Date: 2025-09-17T21:25:06.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>TL;DR:</strong> <br>
Picking a random PDF tool can quietly break your compliance. Cloud-hosted PDF builders often route data through outside servers, change terms without warning, skimp on encryption, and lack the audit logs regulators expect—putting you at risk with GDPR, HIPAA, CCPA, and more. The safer path is to keep control: self-host Joyfill’s Form Builder, model documents as JSON with JoyDoc, collect inputs via your own HTML forms, then generate PDFs on your own infrastructure (e.g., with the JoyDoc Exporter + Puppeteer). This setup lets Legal get clear custody and audit trails, Engineering keep a simple, flexible workflow, and Product avoid surprise data exposure—all while meeting strict privacy promises to users. In short: own your data flow, prove who accessed what and when, and ship PDFs without handing sensitive info to third parties.</p>
</blockquote>

<p>Many companies underestimate how something as simple as choosing a PDF form builder or a library can affect compliance. When multiple teams are involved, priorities diverge: Management focuses on cost, speed, and scalability; Legal focuses on regulatory obligations and liability; Design and Engineering prioritize user experience, functionality, and integration.</p>

<p>Yet, it's easy to overlook how a seemingly minor detail such as where and how a third-party PDF filler or a PDF builder stores and processes user data can undermine all other priorities by introducing serious compliance violations.</p>

<h2>
  
  
  Common Risks in Everyday PDF Workflows
</h2>

<p>Because many PDF tool vendors operate as cloud-managed services, confidential data may pass through external servers or even jurisdictions where data and processing regulations may conflict with GDPR, NEN, HIPPA, CCPA, and dozens of other regional regulations your company has promised to uphold.</p>

<p>A trend that's got most companies that care about data privacy on their toes is how a PDF form creator vendor for example, updates its terms of service and privacy policy without adequately notifying its customers. So, what seemed like a compliant solution yesterday might expose your organization to significant regulatory penalties tomorrow.</p>

<p>Some PDF processors use outdated encryption methods or fail to maintain end-to-end encryption throughout the document lifecycle, creating vulnerabilities that could result in compliance violations if sensitive data is compromised.</p>

<p>Compliance frameworks often require comprehensive audit trails showing who accessed what documents when and from where. Many PDF tools lack the detailed logging capabilities necessary to satisfy these requirements.</p>

<p>To properly manage such risks, an organization must proactively identify and mitigate both common and less obvious compliance risks as soon as they decide to collect, process, or share confidential data. The moment user data enters your system, you are responsible for its security, access control, and for maintaining the privacy guarantees you’ve made to both your users and regulatory bodies of jurisdictions where they live.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2ahngqtngjs3nh6mplxk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2ahngqtngjs3nh6mplxk.png" alt="common compliance pitfalls with PDF tools" width="800" height="446"></a></p>

<h2>
  
  
  How JoyDoc and Joyfill’s Self-Hosted Approach Facilitate Compliance
</h2>

<p>Organizations with strict compliance requirements can’t afford to risk unauthorized access to PDF documents or user data collected through PDF forms hosted by third-party PDF services with opaque handling policies. That’s why many teams are turning to self-hosted PDF solutions built on tools like <a href="https://joyfill.io" rel="noopener noreferrer">Joyfill's</a> web and mobile components.</p>

<p>Although Joyfill offers a managed service, its tools are also designed to accommodate compliance requirements through integration paths where you retain full control of your data.</p>

<p>Such an integration strategy is made possible by <a href="https://docs.joyfill.io/docs/joydoc-usage" rel="noopener noreferrer">JoyDoc</a>, a JSON schema for transferring data between Joyfill platform solutions such as the mobile/web UI components, and the PDF form builder and PDF filler SDK.</p>

<p>Because JoyDoc represents a PDF document as JSON, organizations could avoid compliance issues using the following PDF generation workflow:</p>

<ol>
<li>Self-hosting Joyfill’s PDF Form Builder SDK.</li>
<li>Creating an empty but reusable JoyDoc PDF document using the self-hosted form builder.</li>
<li>Collecting user input through HTML forms.</li>
<li>Generating actual PDF documents by populating reusable JoyDocs from such user input. This can be done in your own infrastructure using Joyfill’s PDF Exporter SDK and a headless browser like <a href="https://pptr.dev/" rel="noopener noreferrer">Puppeteer</a>.</li>
</ol>

<h3>
  
  
  Step 1: Self-host Joyfill’s Form Builder SDK
</h3>

<p>Joyfill’s <a href="https://www.npmjs.com/package/@joyfill/components" rel="noopener noreferrer">Form Builder SDK</a> offers a  simple drag-and-drop interface for authoring a PDF document’s fields in JoyDoc format. Self-hosting the SDK allows you to:</p>

<ol>
<li>Control where your data is stored and how it’s processed.</li>
<li>Reuse the created document across your applications by updating its fields from user data collected through HTML forms.</li>
</ol>

<p>For example, a simple patient portal that allows admins to create reusable PDF templates with JoyDoc, patients to schedule appointments through HTML forms, and the system to generate  PDFs from each appointment’s details might look as follows:</p>

<ol>
<li>Create a new folder named <code>patient-portal</code> and open it in your preferred code editor.</li>
<li>
<p>Run the following command in the code editor’s integrated terminal to configure a new NodeJS project:<br>
</p>
<pre class="highlight shell"><code>npm init <span class="nt">-y</span>
</code></pre>

</li>
<li>
<p>Install the project’s dependencies by running the following commands:<br>
</p>
<pre class="highlight shell"><code>npm <span class="nb">install </span>express lowdb puppeteer
</code></pre>

</li>
</ol>

<ul>
<li>    <code>express</code> exposes the web application to your computer’s network.</li>
<li>    <code>lowdb</code> stores data in a single JSON file.</li>
<li>    <code>puppeteer</code> will use JoyDoc’s exporter to generate PDFs in a 
headless browser.</li>
</ul>

<ol>
<li>Change the value of the <code>type</code> field in the project’s <code>package.json</code> file to make it compatible with <code>lowdb</code> which is going to be used as a database.</li>
<li>
<p>Create an <code>index.js</code> file in the project and copy the following code to it:<br>
</p>
<pre class="highlight jsx"><code><span class="c1">// index.js</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">JSONFilePreset</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">lowdb/node</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">express</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">express</span><span class="dl">'</span><span class="p">;</span>

<span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// Create a data store for JoyDoc and appointments storage</span>
  <span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="k">await</span> <span class="nc">JSONFilePreset</span><span class="p">(</span><span class="dl">'</span><span class="s1">db.json</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">joydoc</span><span class="p">:</span> <span class="p">[],</span>
    <span class="na">appointment</span><span class="p">:</span> <span class="p">[],</span>
  <span class="p">});</span>

  <span class="c1">// Configure an express application</span>
  <span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">static</span><span class="p">(</span><span class="dl">'</span><span class="s1">static</span><span class="dl">'</span><span class="p">));</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">urlencoded</span><span class="p">({</span> <span class="na">extended</span><span class="p">:</span> <span class="kc">false</span> <span class="p">}));</span>

  <span class="c1">// Retrieve all JoyDoc</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">documents</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">doc</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">doc</span><span class="p">.</span><span class="nx">name</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">200</span><span class="p">).</span><span class="nf">json</span><span class="p">(</span><span class="nx">documents</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="c1">// Create a reusable JoyDoc</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">doc</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">savedDoc</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">find</span><span class="p">((</span><span class="nx">d</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">d</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="nx">doc</span><span class="p">.</span><span class="nx">name</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">savedDoc</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">res</span>
        <span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">A document with the same name already exists</span><span class="dl">'</span> <span class="p">});</span>

      <span class="k">return</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c1">// Save new document</span>
    <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">doc</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">write</span><span class="p">();</span>

    <span class="nx">savedDoc</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">find</span><span class="p">((</span><span class="nx">d</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">d</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="nx">doc</span><span class="p">.</span><span class="nx">name</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">201</span><span class="p">).</span><span class="nf">json</span><span class="p">(</span><span class="nx">savedDoc</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="c1">// Retrieve a JoyDoc for editing</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc/:name</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="nx">name</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">params</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">savedDoc</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">find</span><span class="p">((</span><span class="nx">d</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">d</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="nx">name</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">savedDoc</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">404</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Not found</span><span class="dl">'</span> <span class="p">});</span>

      <span class="k">return</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">200</span><span class="p">).</span><span class="nf">json</span><span class="p">(</span><span class="nx">savedDoc</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="c1">// Update an existing JoyDoc as it's edited</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">put</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">update</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">docIndex</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">findIndex</span><span class="p">((</span><span class="nx">d</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">d</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="nx">update</span><span class="p">.</span><span class="nx">name</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">docIndex</span> <span class="o">===</span> <span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">404</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Not found</span><span class="dl">'</span> <span class="p">});</span>
      <span class="k">return</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">[</span><span class="nx">docIndex</span><span class="p">]</span> <span class="o">=</span> <span class="nx">update</span><span class="p">;</span>
    <span class="nx">db</span><span class="p">.</span><span class="nf">write</span><span class="p">();</span>

    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">200</span><span class="p">).</span><span class="nf">end</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="c1">// Schedule a new appointment</span>
  <span class="nx">app</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">'</span><span class="s1">/appointment</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">appointment</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">);</span>
    <span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">write</span><span class="p">();</span>

    <span class="kd">const</span> <span class="p">[</span><span class="nx">date</span><span class="p">,</span> <span class="nx">time</span><span class="p">]</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">time</span><span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="dl">'</span><span class="s1">T</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">201</span><span class="p">).</span><span class="nf">send</span><span class="p">(</span>
      <span class="s2">`&lt;body style="min-height: 90vh; display: flex; justify-content: center; align-items: center;"&gt;
          &lt;p style="font-size: 2rem;"&gt;You successfully scheduled an appointment for </span><span class="p">${</span><span class="nx">time</span><span class="p">}</span><span class="s2"> on </span><span class="p">${</span><span class="nx">date</span><span class="p">}</span><span class="s2">.&lt;/p&gt;
        &lt;/body&gt;
        `</span>
    <span class="p">);</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">port</span> <span class="o">=</span> <span class="mi">3000</span><span class="p">;</span>

  <span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">port</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Patient portal is listening on port </span><span class="p">${</span><span class="nx">port</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">})();</span>
</code></pre>

</li>
<li><p>Create a <code>static</code> folder in the project.</p></li>
<li>
<p>Add an <code>index.html</code> file to the just created <code>static</code> folder and copy the following HTML to it:<br>
</p>
<pre class="highlight html"><code><span class="c">&lt;!-- static/index.html --&gt;</span>
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>

<span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"UTF-8"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1.0"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;title&gt;</span>Patient Portal<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;style&gt;</span>
    <span class="nt">body</span> <span class="p">{</span>
      <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">min-height</span><span class="p">:</span> <span class="m">100vh</span><span class="p">;</span>
      <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
      <span class="nl">justify-content</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
      <span class="nl">align-items</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
      <span class="py">gap</span><span class="p">:</span> <span class="m">7.5vmin</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nt">a</span> <span class="p">{</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">2rem</span><span class="p">;</span>
      <span class="nl">border</span><span class="p">:</span> <span class="m">1px</span> <span class="nb">solid</span> <span class="m">#ddd</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">1em</span><span class="p">;</span>
      <span class="nl">border-radius</span><span class="p">:</span> <span class="m">.5em</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="nt">&lt;/style&gt;</span>
<span class="nt">&lt;/head&gt;</span>

<span class="nt">&lt;body&gt;</span>
  <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"/document.html"</span> <span class="na">class=</span><span class="s">"cta"</span><span class="nt">&gt;</span>Manage Documents<span class="nt">&lt;/a&gt;</span>
  <span class="nt">&lt;a</span> <span class="na">href=</span><span class="s">"/appointment.html"</span> <span class="na">class=</span><span class="s">"cta"</span><span class="nt">&gt;</span>Schedule Appointment<span class="nt">&lt;/a&gt;</span>
<span class="nt">&lt;/body&gt;</span>

<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
<li>
<p>Start the web server by running the following command:<br>
</p>
<pre class="highlight shell"><code>node index.js
</code></pre>

</li>
<li><p>Visit <a href="http://localhost:3000" rel="noopener noreferrer">http://localhost:3000</a> in your browser. You should see the following web page:</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzer7b843xe7z367qs91p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzer7b843xe7z367qs91p.png" alt="pdf web page preview" width="800" height="305"></a></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>The app currently offers the options of managing existing JoyDocs or scheduling an appointment. Let's make a page to create and edit JoyDocs below.
</code></pre>

</div>

<ol>
<li>
<p>Add a <code>document.html</code> file to the <code>static</code> folder and save the following markup to it:<br>
</p>
<pre class="highlight html"><code><span class="c">&lt;!-- static/document.html --&gt;</span>
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>

<span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"UTF-8"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1.0"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;title&gt;</span>Manage Documents<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;style&gt;</span>
    <span class="nc">.action-form</span> <span class="p">{</span>
      <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
      <span class="nl">justify-content</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
      <span class="nl">align-items</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
      <span class="nl">min-height</span><span class="p">:</span> <span class="m">100vh</span><span class="p">;</span>
      <span class="py">gap</span><span class="p">:</span> <span class="m">2em</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.field-group</span> <span class="p">{</span>
      <span class="nl">border</span><span class="p">:</span> <span class="m">1px</span> <span class="nb">solid</span> <span class="m">#ddd</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">1em</span><span class="p">;</span>
      <span class="nl">flex-basis</span><span class="p">:</span> <span class="m">400px</span><span class="p">;</span>
      <span class="nl">max-width</span><span class="p">:</span> <span class="m">400px</span><span class="p">;</span>
      <span class="nl">height</span><span class="p">:</span> <span class="m">240px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.field-group</span><span class="o">&gt;</span><span class="nc">.field-label</span> <span class="p">{</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">1.75em</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.hint</span> <span class="p">{</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">0.75em</span><span class="p">;</span>
      <span class="nl">margin-top</span><span class="p">:</span> <span class="m">2px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nf">#old-document</span><span class="o">,</span>
    <span class="nf">#new-document</span><span class="o">,</span>
    <span class="nf">#create-new</span> <span class="p">{</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">1.25rem</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nf">#old-document</span><span class="o">,</span>
    <span class="nf">#new-document</span><span class="o">,</span>
    <span class="nf">#create-new</span><span class="o">,</span>
    <span class="nc">.field-group</span><span class="o">&gt;</span><span class="nc">.field-label</span> <span class="p">{</span>
      <span class="nl">font-family</span><span class="p">:</span> <span class="n">system-ui</span><span class="p">,</span> <span class="n">-apple-system</span><span class="p">,</span> <span class="n">BlinkMacSystemFont</span><span class="p">,</span> <span class="s2">'Segoe UI'</span><span class="p">,</span> <span class="n">Roboto</span><span class="p">,</span>
        <span class="n">Oxygen</span><span class="p">,</span> <span class="n">Ubuntu</span><span class="p">,</span> <span class="n">Cantarell</span><span class="p">,</span> <span class="s2">'Open Sans'</span><span class="p">,</span> <span class="s2">'Helvetica Neue'</span><span class="p">,</span> <span class="nb">sans-serif</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nf">#old-document</span> <span class="p">{</span>
      <span class="nl">flex</span><span class="p">:</span> <span class="m">1</span><span class="p">;</span>
      <span class="nl">cursor</span><span class="p">:</span> <span class="nb">pointer</span><span class="p">;</span>
      <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0.8em</span><span class="p">;</span>
      <span class="nl">border-radius</span><span class="p">:</span> <span class="m">0.25em</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nf">#old-document</span><span class="o">,</span>
    <span class="nd">::picker</span><span class="o">(</span><span class="nt">select</span><span class="o">)</span> <span class="p">{</span>
      <span class="nl">appearance</span><span class="p">:</span> <span class="n">base-select</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nf">#new-document</span> <span class="p">{</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0.8em</span><span class="p">;</span>
      <span class="nl">display</span><span class="p">:</span> <span class="nb">block</span><span class="p">;</span>
      <span class="nl">max-width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
      <span class="nl">border-radius</span><span class="p">:</span> <span class="m">0.25em</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nf">#create-new</span> <span class="p">{</span>
      <span class="nl">display</span><span class="p">:</span> <span class="nb">block</span><span class="p">;</span>
      <span class="nl">border</span><span class="p">:</span> <span class="nb">none</span><span class="p">;</span>
      <span class="nl">border-radius</span><span class="p">:</span> <span class="m">0.25em</span><span class="p">;</span>
      <span class="nl">background-color</span><span class="p">:</span> <span class="no">green</span><span class="p">;</span>
      <span class="nl">color</span><span class="p">:</span> <span class="no">white</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0.8em</span> <span class="m">1em</span><span class="p">;</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">1em</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="nt">&lt;/style&gt;</span>
<span class="nt">&lt;/head&gt;</span>

<span class="nt">&lt;body&gt;</span>
  <span class="nt">&lt;form</span> <span class="na">action=</span><span class="s">""</span> <span class="na">class=</span><span class="s">"action-form"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;p</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Edit existing document<span class="nt">&lt;/p&gt;</span>
      <span class="nt">&lt;select</span> <span class="na">id=</span><span class="s">"old-document"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;option</span> <span class="na">value=</span><span class="s">""</span> <span class="na">selected</span> <span class="na">class=</span><span class="s">"document-option"</span><span class="nt">&gt;</span>Select a document<span class="nt">&lt;/option&gt;</span>
      <span class="nt">&lt;/select&gt;</span>
    <span class="nt">&lt;/div&gt;</span>

    <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;p</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Create new document<span class="nt">&lt;/p&gt;</span>
      <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">id=</span><span class="s">"new-document"</span> <span class="na">placeholder=</span><span class="s">"Document name"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;p</span> <span class="na">class=</span><span class="s">"hint"</span><span class="nt">&gt;</span>E.g. <span class="nt">&lt;i&gt;</span>Patient Appointment<span class="nt">&lt;/i&gt;&lt;/p&gt;</span>
      <span class="nt">&lt;button</span> <span class="na">id=</span><span class="s">"create-new"</span><span class="nt">&gt;</span>Create<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;/form&gt;</span>

  <span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"joyfill"</span><span class="nt">&gt;&lt;/div&gt;</span>

  <span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"application/javascript"</span>
    <span class="na">src=</span><span class="s">"https://cdn.jsdelivr.net/npm/@joyfill/components@latest/dist/joyfill.min.js"</span><span class="nt">&gt;&lt;/script&gt;</span>

  <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"/document.js"</span><span class="nt">&gt;&lt;/script&gt;</span>

  <span class="nt">&lt;script&gt;</span>
    <span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">let</span> <span class="nx">docNames</span> <span class="o">=</span> <span class="p">[];</span>

      <span class="c1">// Retrieve existing documents</span>
      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
          <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span> <span class="p">},</span>
        <span class="p">});</span>

        <span class="nx">docNames</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">e</span><span class="p">);</span>

        <span class="nf">alert</span><span class="p">(</span>
          <span class="s2">`Failed to retrieve list of existing documents: </span><span class="p">${</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">message</span><span class="p">)}</span><span class="s2">`</span>
        <span class="p">);</span>
      <span class="p">}</span>

      <span class="c1">// Add the names of existing documents to the select field to the UI</span>
      <span class="kd">let</span> <span class="nx">docList</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#old-document</span><span class="dl">'</span><span class="p">);</span>

      <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">name</span> <span class="k">of</span> <span class="nx">docNames</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">option</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="dl">'</span><span class="s1">option</span><span class="dl">'</span><span class="p">);</span>

        <span class="nx">option</span><span class="p">.</span><span class="nx">textContent</span> <span class="o">=</span> <span class="nx">name</span><span class="p">;</span>
        <span class="nx">option</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="nx">name</span><span class="p">;</span>

        <span class="nx">docList</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nx">option</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="c1">//  Handle new document creation</span>
      <span class="nb">document</span>
        <span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#create-new</span><span class="dl">'</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">click</span><span class="dl">'</span><span class="p">,</span> <span class="nx">createNewDocument</span><span class="p">);</span>

      <span class="c1">// Handle document selection</span>
      <span class="nb">document</span>
        <span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#old-document</span><span class="dl">'</span><span class="p">)</span>
        <span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">change</span><span class="dl">'</span><span class="p">,</span> <span class="nx">loadExisting</span><span class="p">);</span>
    <span class="p">})();</span>

    <span class="c1">// Create a new document</span>
    <span class="k">async</span> <span class="kd">function</span> <span class="nf">createNewDocument</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>

      <span class="kd">let</span> <span class="nx">name</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#new-document</span><span class="dl">'</span><span class="p">).</span><span class="nx">value</span><span class="p">;</span>
      <span class="kd">let</span> <span class="nx">blankDocument</span> <span class="o">=</span> <span class="nx">Joyfill</span><span class="p">.</span><span class="nf">getDefaultDocument</span><span class="p">();</span>

      <span class="nx">blankDocument</span><span class="p">.</span><span class="nx">name</span> <span class="o">=</span> <span class="nx">name</span><span class="p">;</span>

      <span class="c1">// Save the document to the store</span>
      <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
        <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">blankDocument</span><span class="p">),</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">});</span>

      <span class="kd">const</span> <span class="nx">doc</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">doc</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">msg</span> <span class="o">=</span> <span class="s2">`Failed to save </span><span class="p">${</span><span class="nx">blankDocument</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">doc</span><span class="p">.</span><span class="nx">error</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>

        <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">msg</span><span class="p">);</span>
        <span class="nf">alert</span><span class="p">(</span><span class="nx">msg</span><span class="p">);</span>

        <span class="k">return</span><span class="p">;</span>
      <span class="p">}</span>

      <span class="nf">initFormBuilder</span><span class="p">(</span><span class="nx">doc</span><span class="p">);</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Saved </span><span class="p">${</span><span class="nx">doc</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="kd">function</span> <span class="nf">initFormBuilder</span><span class="p">(</span><span class="nx">doc</span><span class="p">)</span> <span class="p">{</span>
      <span class="c1">// Partially reset layout</span>
      <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">.action-form</span><span class="dl">'</span><span class="p">).</span><span class="nx">style</span><span class="p">.</span><span class="nx">display</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">none</span><span class="dl">'</span><span class="p">;</span>

      <span class="nx">Joyfill</span><span class="p">.</span><span class="nc">JoyDoc</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">'</span><span class="s1">joyfill</span><span class="dl">'</span><span class="p">),</span> <span class="p">{</span>
        <span class="nx">doc</span><span class="p">,</span>
        <span class="na">mode</span><span class="p">:</span> <span class="dl">'</span><span class="s1">edit</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">onChange</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">changelogs</span><span class="p">,</span> <span class="nx">updatedDoc</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="c1">// Log document changes to the console.</span>
          <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">changelogs</span><span class="p">);</span>

          <span class="c1">// Update document</span>
          <span class="k">try</span> <span class="p">{</span>
            <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">/joydoc</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
              <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">PUT</span><span class="dl">'</span><span class="p">,</span>
              <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">updatedDoc</span><span class="p">),</span>
              <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
                <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
              <span class="p">},</span>
            <span class="p">});</span>

            <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Successfully updated </span><span class="p">${</span><span class="nx">updatedDoc</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
          <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">e</span><span class="p">);</span>
            <span class="nf">alert</span><span class="p">(</span><span class="s2">`Failed to update </span><span class="p">${</span><span class="nx">updatedDoc</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">: </span><span class="p">${</span><span class="nx">e</span><span class="p">.</span><span class="nx">message</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
          <span class="p">}</span>
        <span class="p">},</span>
      <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="kd">function</span> <span class="nf">loadExisting</span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>

      <span class="kd">const</span> <span class="nx">name</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#old-document</span><span class="dl">'</span><span class="p">).</span><span class="nx">value</span><span class="p">;</span>

      <span class="k">try</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="s2">`/joydoc/</span><span class="p">${</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="p">{</span>
          <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span> <span class="p">},</span>
        <span class="p">});</span>

        <span class="kd">const</span> <span class="nx">doc</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

        <span class="nf">initFormBuilder</span><span class="p">(</span><span class="nx">doc</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">e</span><span class="p">);</span>

        <span class="nf">alert</span><span class="p">(</span>
          <span class="s2">`Failed retrieve list of existing documents: </span><span class="p">${</span><span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">message</span><span class="p">)}</span><span class="s2">`</span>
        <span class="p">);</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="nt">&lt;/script&gt;</span>
<span class="nt">&lt;/body&gt;</span>

<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
</ol>

<h3>
  
  
  Step 2: Create a Resusable JoyDoc
</h3>

<ol>
<li>Click the <code>Manage Documents</code> link on <a href="http://localhost:3000" rel="noopener noreferrer">http://localhost:3000</a>. It should display the options shown in the image below. Create a new document named <code>Patient Appointment</code> as shown below. </li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9hiov8qje72kpse1qh7r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9hiov8qje72kpse1qh7r.png" alt="create new joydoc" width="800" height="309"></a></p>

<ol>
<li>Add fields to the document as demonstrated in the following image. For each field, set an appropriate title, identifier, etc.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7l5d1m3q1qyc8g08970j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7l5d1m3q1qyc8g08970j.png" alt="patient appointment form preview" width="800" height="344"></a></p>

<h3>
  
  
  Step 3: Collect User Input from HTML Forms
</h3>

<ol>
<li>
<p>Back in your code editor, add an <code>appointment.html</code> file to the <code>static</code> folder and copy the following markup into it:<br>
</p>
<pre class="highlight html"><code><span class="c">&lt;!-- static/appointment.html --&gt;</span>
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>

<span class="nt">&lt;head&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"UTF-8"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1.0"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;title&gt;</span>Patient Appointment<span class="nt">&lt;/title&gt;</span>
  <span class="nt">&lt;style&gt;</span>
    <span class="nc">.appointment-wrapper</span> <span class="p">{</span>
      <span class="nl">background-color</span><span class="p">:</span> <span class="m">#ecedf3</span><span class="p">;</span>
      <span class="nl">min-height</span><span class="p">:</span> <span class="m">100vh</span><span class="p">;</span>
      <span class="nl">background-color</span><span class="p">:</span> <span class="m">#ecedf3</span><span class="p">;</span>
      <span class="nl">font-family</span><span class="p">:</span> <span class="n">system-ui</span><span class="p">,</span> <span class="n">-apple-system</span><span class="p">,</span> <span class="n">BlinkMacSystemFont</span><span class="p">,</span> <span class="s2">'Segoe UI'</span><span class="p">,</span> <span class="n">Roboto</span><span class="p">,</span>
        <span class="n">Oxygen</span><span class="p">,</span> <span class="n">Ubuntu</span><span class="p">,</span> <span class="n">Cantarell</span><span class="p">,</span> <span class="s2">'Open Sans'</span><span class="p">,</span> <span class="s2">'Helvetica Neue'</span><span class="p">,</span> <span class="nb">sans-serif</span><span class="p">;</span>
      <span class="nl">color</span><span class="p">:</span> <span class="m">#2c3345</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.appointment</span> <span class="p">{</span>
      <span class="nl">background-color</span><span class="p">:</span> <span class="m">#fff</span><span class="p">;</span>
      <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span> <span class="nb">auto</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">5vmin</span><span class="p">;</span>
      <span class="nl">max-width</span><span class="p">:</span> <span class="m">816px</span><span class="p">;</span>
      <span class="nl">min-height</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
      <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
      <span class="nl">justify-content</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
      <span class="nl">align-items</span><span class="p">:</span> <span class="nb">center</span><span class="p">;</span>
      <span class="nl">flex-wrap</span><span class="p">:</span> <span class="n">wrap</span><span class="p">;</span>
      <span class="py">row-gap</span><span class="p">:</span> <span class="m">24px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.title</span><span class="o">,</span>
    <span class="nc">.name</span> <span class="p">{</span>
      <span class="nl">flex</span><span class="p">:</span> <span class="m">0</span> <span class="m">0</span> <span class="m">100%</span><span class="p">;</span>
      <span class="nl">max-width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.name</span> <span class="p">{</span>
      <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
      <span class="nl">justify-content</span><span class="p">:</span> <span class="n">space-between</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.name</span><span class="o">&gt;</span><span class="nc">.field-control</span> <span class="p">{</span>
      <span class="nl">flex</span><span class="p">:</span> <span class="m">0</span> <span class="m">0</span> <span class="m">40%</span><span class="p">;</span>
      <span class="nl">max-width</span><span class="p">:</span> <span class="m">40%</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.field-label</span><span class="o">,</span>
    <span class="nt">input</span><span class="nc">.field</span> <span class="p">{</span>
      <span class="nl">display</span><span class="p">:</span> <span class="nb">block</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.field-label</span> <span class="p">{</span>
      <span class="nl">margin-bottom</span><span class="p">:</span> <span class="m">4px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nt">input</span><span class="nc">.field</span> <span class="p">{</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0.8em</span><span class="p">;</span>
      <span class="nl">width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.appointment</span><span class="o">&gt;</span><span class="nc">.field-group</span> <span class="p">{</span>
      <span class="nl">flex</span><span class="p">:</span> <span class="m">0</span> <span class="m">0</span> <span class="m">100%</span><span class="p">;</span>
      <span class="nl">max-width</span><span class="p">:</span> <span class="m">100%</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.field-hint</span> <span class="p">{</span>
      <span class="nl">margin-top</span><span class="p">:</span> <span class="m">4px</span><span class="p">;</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">14px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.contact-preference</span><span class="o">&gt;</span><span class="nt">label</span> <span class="p">{</span>
      <span class="nl">display</span><span class="p">:</span> <span class="n">inline-block</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.prefers-email-label</span> <span class="p">{</span>
      <span class="nl">margin-right</span><span class="p">:</span> <span class="m">24px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.field</span><span class="o">,</span>
    <span class="nc">.prefers-email</span><span class="o">,</span>
    <span class="nc">.prefers-phone</span><span class="o">,</span>
    <span class="nc">.contact-preference</span><span class="o">&gt;</span><span class="nt">label</span><span class="o">,</span>
    <span class="nc">.departments</span><span class="o">&gt;</span><span class="nt">ul</span><span class="o">&gt;</span><span class="nt">li</span> <span class="p">{</span>
      <span class="nl">border-radius</span><span class="p">:</span> <span class="m">0.5em</span><span class="p">;</span>
      <span class="nl">border</span><span class="p">:</span> <span class="m">1px</span> <span class="nb">solid</span> <span class="m">#333</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.departments</span><span class="o">&gt;</span><span class="nt">ul</span><span class="o">&gt;</span><span class="nt">li</span><span class="o">,</span>
    <span class="nc">.contact-preference</span> <span class="nt">label</span> <span class="p">{</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0.8em</span> <span class="m">1em</span><span class="p">;</span>
      <span class="nl">cursor</span><span class="p">:</span> <span class="nb">pointer</span><span class="p">;</span>
      <span class="nl">width</span><span class="p">:</span> <span class="m">220px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.departments</span> <span class="p">{</span>
      <span class="nl">display</span><span class="p">:</span> <span class="n">flex</span><span class="p">;</span>
      <span class="nl">justify-content</span><span class="p">:</span> <span class="n">flex-start</span><span class="p">;</span>
      <span class="py">gap</span><span class="p">:</span> <span class="m">24px</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.departments</span><span class="o">&gt;</span><span class="nt">ul</span> <span class="p">{</span>
      <span class="nl">list-style-type</span><span class="p">:</span> <span class="nb">none</span><span class="p">;</span>
      <span class="nl">flex</span><span class="p">:</span> <span class="m">0</span> <span class="m">0</span> <span class="m">30%</span><span class="p">;</span>
      <span class="nl">padding-left</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">padding-right</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nc">.schedule</span> <span class="p">{</span>
      <span class="nl">cursor</span><span class="p">:</span> <span class="nb">pointer</span><span class="p">;</span>
      <span class="nl">margin</span><span class="p">:</span> <span class="m">0px</span> <span class="nb">auto</span><span class="p">;</span>
      <span class="nl">display</span><span class="p">:</span> <span class="nb">block</span><span class="p">;</span>
      <span class="nl">background-color</span><span class="p">:</span> <span class="no">green</span><span class="p">;</span>
      <span class="nl">border</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="nl">border-radius</span><span class="p">:</span> <span class="m">0.25em</span><span class="p">;</span>
      <span class="nl">color</span><span class="p">:</span> <span class="no">white</span><span class="p">;</span>
      <span class="nl">padding</span><span class="p">:</span> <span class="m">0.5em</span> <span class="m">1em</span><span class="p">;</span>
      <span class="nl">font-size</span><span class="p">:</span> <span class="m">1.25rem</span><span class="p">;</span>
      <span class="nl">font-family</span><span class="p">:</span> <span class="n">system-ui</span><span class="p">,</span> <span class="n">-apple-system</span><span class="p">,</span> <span class="n">BlinkMacSystemFont</span><span class="p">,</span> <span class="s2">'Segoe UI'</span><span class="p">,</span> <span class="n">Roboto</span><span class="p">,</span>
        <span class="n">Oxygen</span><span class="p">,</span> <span class="n">Ubuntu</span><span class="p">,</span> <span class="n">Cantarell</span><span class="p">,</span> <span class="s2">'Open Sans'</span><span class="p">,</span> <span class="s2">'Helvetica Neue'</span><span class="p">,</span> <span class="nb">sans-serif</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="nt">&lt;/style&gt;</span>
<span class="nt">&lt;/head&gt;</span>

<span class="nt">&lt;body&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"appointment-wrapper"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;form</span> <span class="na">class=</span><span class="s">"appointment"</span> <span class="na">action=</span><span class="s">"/appointment"</span> <span class="na">method=</span><span class="s">"POST"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;h1</span> <span class="na">class=</span><span class="s">"title"</span><span class="nt">&gt;</span>Patient Appointment<span class="nt">&lt;/h1&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"name"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-control"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"firstname"</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>First name<span class="nt">&lt;/label&gt;</span>
          <span class="nt">&lt;input</span> <span class="na">value=</span><span class="s">"John"</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">name=</span><span class="s">"firstname"</span> <span class="na">class=</span><span class="s">"firstname field"</span> <span class="nt">/&gt;</span>
        <span class="nt">&lt;/div&gt;</span>

        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-control"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"lastname"</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Last name<span class="nt">&lt;/label&gt;</span>
          <span class="nt">&lt;input</span> <span class="na">value=</span><span class="s">"Doe"</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">name=</span><span class="s">"lastname"</span> <span class="na">class=</span><span class="s">"lastname field"</span> <span class="nt">/&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;label</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Email<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">value=</span><span class="s">"john@doe.com"</span> <span class="na">type=</span><span class="s">"email"</span> <span class="na">name=</span><span class="s">"email"</span> <span class="na">class=</span><span class="s">"email field"</span> <span class="nt">/&gt;</span>
        <span class="nt">&lt;p</span> <span class="na">class=</span><span class="s">"field-hint"</span><span class="nt">&gt;</span>example@example.com<span class="nt">&lt;/p&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;label</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Phone<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">value=</span><span class="s">"+17182222222"</span> <span class="na">type=</span><span class="s">"tel"</span> <span class="na">name=</span><span class="s">"phone"</span> <span class="na">class=</span><span class="s">"phone field"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;p</span> <span class="na">class=</span><span class="s">"field-hint"</span><span class="nt">&gt;</span>+17182222222<span class="nt">&lt;/p&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;label</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Contact preference<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"contact-preference"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;label</span> <span class="na">class=</span><span class="s">"prefers-email-label"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;input</span> <span class="na">checked</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"contactPreference"</span> <span class="na">class=</span><span class="s">"prefers-email"</span> <span class="na">value=</span><span class="s">"Email"</span><span class="nt">&gt;</span>
            Email
          <span class="nt">&lt;/label&gt;</span>

          <span class="nt">&lt;label&gt;</span>
            <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"contactPreference"</span> <span class="na">class=</span><span class="s">"prefers-phone"</span> <span class="na">value=</span><span class="s">"Phone"</span><span class="nt">&gt;</span>
            Phone
          <span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"department"</span><span class="nt">&gt;</span>Which department do you want to make an appointment with?<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"departments"</span><span class="nt">&gt;</span>
          <span class="nt">&lt;ul&gt;</span>
            <span class="nt">&lt;li&gt;&lt;input</span> <span class="na">checked</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"department"</span> <span class="na">value=</span><span class="s">"Pediatrics"</span><span class="nt">&gt;</span> Pediatrics<span class="nt">&lt;/li&gt;</span>
            <span class="nt">&lt;li&gt;&lt;input</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"department"</span> <span class="na">value=</span><span class="s">"General Surgery"</span><span class="nt">&gt;</span> General Surgery<span class="nt">&lt;/li&gt;</span>
          <span class="nt">&lt;/ul&gt;</span>

          <span class="nt">&lt;ul&gt;</span>
            <span class="nt">&lt;li&gt;&lt;input</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"department"</span> <span class="na">value=</span><span class="s">"Obstetrics &amp; Gynecology"</span><span class="nt">&gt;</span> Obstetrics <span class="ni">&amp;amp;</span> Gynecology<span class="nt">&lt;/li&gt;</span>
            <span class="nt">&lt;li&gt;&lt;input</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"department"</span> <span class="na">value=</span><span class="s">"Neurology"</span><span class="nt">&gt;</span> Neurology<span class="nt">&lt;/li&gt;</span>
          <span class="nt">&lt;/ul&gt;</span>

          <span class="nt">&lt;ul&gt;</span>
            <span class="nt">&lt;li&gt;&lt;input</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"department"</span> <span class="na">value=</span><span class="s">"Orthopedics"</span><span class="nt">&gt;</span> Orthopedics<span class="nt">&lt;/li&gt;</span>
            <span class="nt">&lt;li&gt;&lt;input</span> <span class="na">type=</span><span class="s">"radio"</span> <span class="na">name=</span><span class="s">"department"</span> <span class="na">value=</span><span class="s">"Oncology"</span><span class="nt">&gt;</span> Oncology<span class="nt">&lt;/li&gt;</span>
            <span class="nt">&lt;ul&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"field-group"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"time"</span> <span class="na">class=</span><span class="s">"field-label"</span><span class="nt">&gt;</span>Time<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">value=</span><span class="s">"2030-06-01T08:30"</span> <span class="na">type=</span><span class="s">"datetime-local"</span> <span class="na">name=</span><span class="s">"time"</span> <span class="na">class=</span><span class="s">"time field"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;/div&gt;</span>

      <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">class=</span><span class="s">"schedule"</span><span class="nt">&gt;</span>Schedule<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;/form&gt;</span>
  <span class="nt">&lt;/div&gt;</span>

<span class="nt">&lt;/body&gt;</span>

<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
<li><p>Visit <a href="http://localhost:3000/appointment.html" rel="noopener noreferrer">http://localhost:3000/appointment.html</a> in your browser and click the <code>Schedule</code> button at the bottom of the page:</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F70an4199dizgp7hgp54y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F70an4199dizgp7hgp54y.png" alt="scheule button on form preview" width="800" height="787"></a></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>This should save the appointment’s details to a `db.json` file created by `lowdb` earlier.
</code></pre>

</div>
<h3>
  
  
  Step 4: Generate PDFs from User Supplied Data
</h3>

<p>JoyDoc Exporter renders a populated JoyDoc to HTML in a browser context. However, you can take advantage of it outside a browser by adding a JavaScript module like the following to your project.<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight jsx"><code><span class="c1">// generate-pdf.js</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">JSONFilePreset</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">lowdb/node</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="nx">puppeteer</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">puppeteer</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">join</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">existsSync</span><span class="p">,</span> <span class="nx">mkdirSync</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">fs</span><span class="dl">'</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">generatePdfs</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="k">await</span> <span class="nc">JSONFilePreset</span><span class="p">(</span><span class="dl">'</span><span class="s1">db.json</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">joydoc</span><span class="p">:</span> <span class="p">[],</span> <span class="na">appointment</span><span class="p">:</span> <span class="p">[]</span> <span class="p">});</span>
  <span class="kd">const</span> <span class="nx">browser</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">puppeteer</span><span class="p">.</span><span class="nf">launch</span><span class="p">();</span>
  <span class="kd">const</span> <span class="nx">page</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">browser</span><span class="p">.</span><span class="nf">newPage</span><span class="p">();</span>

  <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">addScriptTag</span><span class="p">({</span>
    <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://cdn.jsdelivr.net/npm/@joyfill/components@latest/dist/joyfill.min.js</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="c1">// Get appointments JoyDoc from the database</span>
  <span class="kd">const</span> <span class="nx">joydoc</span> <span class="o">=</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">joydoc</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span>
    <span class="p">(</span><span class="nx">doc</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">doc</span><span class="p">.</span><span class="nx">name</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">Patient Appointment</span><span class="dl">'</span>
  <span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nf">existsSync</span><span class="p">(</span><span class="dl">'</span><span class="s1">appointments</span><span class="dl">'</span><span class="p">))</span> <span class="p">{</span>
    <span class="nf">mkdirSync</span><span class="p">(</span><span class="dl">'</span><span class="s1">appointments</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Render each appointment in a headless browser and export the page</span>
  <span class="c1">// and save it to a PDF file.</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">appointment</span> <span class="k">of</span> <span class="nx">db</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">appointment</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">appointmentJoyDoc</span> <span class="o">=</span> <span class="nf">populateDocument</span><span class="p">(</span><span class="nx">appointment</span><span class="p">,</span> <span class="nx">joydoc</span><span class="p">);</span>

    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Generating PDF for </span><span class="p">${</span><span class="nx">appointmentJoyDoc</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">((</span><span class="nx">appointmentJoyDoc</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">container</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="dl">'</span><span class="s1">div</span><span class="dl">'</span><span class="p">);</span>
      <span class="nx">container</span><span class="p">.</span><span class="nx">id</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">joyfill</span><span class="dl">'</span><span class="p">;</span>
      <span class="nb">document</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nx">container</span><span class="p">);</span>

      <span class="nb">document</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nf">appendChild</span><span class="p">(</span><span class="nx">container</span><span class="p">);</span>
      <span class="nx">Joyfill</span><span class="p">.</span><span class="nc">JoyDocExporter</span><span class="p">(</span><span class="nx">container</span><span class="p">,</span> <span class="p">{</span>
        <span class="na">doc</span><span class="p">:</span> <span class="nx">appointmentJoyDoc</span><span class="p">,</span>
        <span class="na">config</span><span class="p">:</span> <span class="p">{</span> <span class="na">page</span><span class="p">:</span> <span class="p">{</span> <span class="na">height</span><span class="p">:</span> <span class="mi">1056</span><span class="p">,</span> <span class="na">width</span><span class="p">:</span> <span class="mi">816</span><span class="p">,</span> <span class="na">padding</span><span class="p">:</span> <span class="mi">0</span> <span class="p">}</span> <span class="p">},</span>
      <span class="p">});</span>
    <span class="p">},</span> <span class="nx">appointmentJoyDoc</span><span class="p">);</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">pdf</span><span class="p">({</span>
      <span class="na">path</span><span class="p">:</span> <span class="nf">join</span><span class="p">(</span><span class="dl">'</span><span class="s1">appointments</span><span class="dl">'</span><span class="p">,</span> <span class="s2">`</span><span class="p">${</span><span class="nx">appointmentJoyDoc</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">.pdf`</span><span class="p">),</span>
    <span class="p">});</span>

    <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nb">document</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">#joyfill</span><span class="dl">'</span><span class="p">).</span><span class="nf">remove</span><span class="p">();</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">await</span> <span class="nx">browser</span><span class="p">.</span><span class="nf">close</span><span class="p">();</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">populateDocument</span><span class="p">(</span><span class="nx">appointment</span><span class="p">,</span> <span class="nx">joydoc</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// Include the patient's name in the document's title</span>
  <span class="kd">const</span> <span class="nx">filledDocument</span> <span class="o">=</span> <span class="p">{</span> <span class="p">...</span><span class="nx">joydoc</span> <span class="p">};</span>

  <span class="c1">// Set the document's name</span>
  <span class="nx">filledDocument</span><span class="p">.</span><span class="nx">name</span> <span class="o">=</span> <span class="s2">`</span><span class="p">${</span><span class="nx">filledDocument</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2"> - </span><span class="p">${</span><span class="nx">appointment</span><span class="p">.</span><span class="nx">firstname</span><span class="p">}</span><span class="s2"> </span><span class="p">${</span><span class="nx">appointment</span><span class="p">.</span><span class="nx">lastname</span><span class="p">}</span><span class="s2"> - </span><span class="p">${</span><span class="nx">appointment</span><span class="p">.</span><span class="nx">time</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>

  <span class="c1">// Set the document's title</span>
  <span class="nx">filledDocument</span><span class="p">.</span><span class="nx">fields</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">value</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">Patient Appointment</span><span class="dl">'</span><span class="p">;</span>

  <span class="c1">// Populate other fields of the document</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">field</span> <span class="k">of</span> <span class="nx">filledDocument</span><span class="p">.</span><span class="nx">fields</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">id</span> <span class="o">=</span> <span class="nx">field</span><span class="p">.</span><span class="nx">identifier</span><span class="p">;</span>

    <span class="k">if </span><span class="p">([</span><span class="dl">'</span><span class="s1">contactPreference</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">department</span><span class="dl">'</span><span class="p">].</span><span class="nf">includes</span><span class="p">(</span><span class="nx">id</span><span class="p">))</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">option</span> <span class="o">=</span> <span class="nx">field</span><span class="p">.</span><span class="nx">options</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span>
        <span class="p">(</span><span class="nx">option</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">option</span><span class="p">.</span><span class="nx">value</span><span class="p">.</span><span class="nf">trim</span><span class="p">()</span> <span class="o">===</span> <span class="nx">appointment</span><span class="p">[</span><span class="nx">id</span><span class="p">].</span><span class="nf">trim</span><span class="p">()</span>
      <span class="p">);</span>

      <span class="nx">field</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="p">[</span><span class="nx">option</span><span class="p">.</span><span class="nx">_id</span><span class="p">];</span>
      <span class="k">continue</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="nx">field</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="nx">appointment</span><span class="p">[</span><span class="nx">id</span><span class="p">];</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">filledDocument</span><span class="p">;</span>
<span class="p">}</span>

<span class="nf">generatePdfs</span><span class="p">()</span>
  <span class="p">.</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">PDFs saved to 'appointments' successfully</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">})</span>
  <span class="p">.</span><span class="k">catch</span><span class="p">((</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Failed to generate PDFs</span><span class="dl">'</span><span class="p">,</span> <span class="nx">e</span><span class="p">);</span>
  <span class="p">});</span>
</code></pre>

</div>



<p>With a bit of hardening and the right sanity checks in place, a setup like the one above, leverages Joyfill’s self-hosted tools and the JoyDoc schema to strike the balance most organizations struggle with. You get full control over where and how data is stored, clear audit trail for compliance teams, and the flexibility engineers need to keep workflows simple. In practice, this means meeting strict regulatory requirements without exposing sensitive information to third-party PDF services or sacrificing usability in the process.</p>

<p><strong>Need to build PDF capabilities inside your SaaS application?</strong> <a href="https://joyfill.io/developers" rel="noopener noreferrer">Joyfill</a> makes it easy for developers to natively build, embed, and customize form and PDF experiences inside their web and mobile apps.</p>

