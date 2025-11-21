---
Title: Introducing Nano Banana Pro: Complete Developer Tutorial
Description: 
Author: Guillaume Vernade
Date: 2025-11-21T21:30:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>You loved <a href="https://dev.to/googleai/how-to-build-with-nano-banana-complete-developer-tutorial-646">Nano-Banana</a>? Created figurine images of all your friends and ghost face behind all your foes? Here now comes the not-so-nano "<a href="https://deepmind.google/models/gemini-image/pro/" rel="noopener noreferrer">Gemini 3 Pro Image</a>" model, that you will all prefer calling <strong>Nano Banana Pro</strong>!</p>

<p>While the Flash model (Nano Banana) brought speed and affordability, the Pro version introduces "thinking" capabilities, search grounding, and high-fidelity 4K output. It's time to go bananas with complex creative tasks!</p>

<p>This guide will walk you through the advanced features of Nano Banana Pro using the <a href="https://ai.google.dev/" rel="noopener noreferrer">Gemini Developer API</a>.</p>

<p>This guide will cover:</p>

<ol>
<li> Using Nano Banana Pro in Google AI Studio</li>
<li> Project setup</li>
<li> Initialize the Client</li>
<li> Basic Generation (The Classics)</li>
<li> The "Thinking" Process</li>
<li> Search Grounding</li>
<li> High-Resolution 4K Generation</li>
<li> Multilingual Capabilities</li>
<li> Advanced Image Mixing</li>
<li>Pro-Exclusive Demos</li>
</ol>

<blockquote>
<p>Note: for an interactive version of this post, checkout the <a href="https://colab.sandbox.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_Started_Nano_Banana.ipynb" rel="noopener noreferrer">python cookbook</a> or the AI Studio's <a href="https://ai.studio/apps/bundled/get_started_image_out?fullscreenApplet=true" rel="noopener noreferrer">Javascript Notebook</a>.</p>
</blockquote>

<h2>
  
  
  1) Using Nano Banana Pro in Google AI Studio
</h2>

<p>While end-users can access Nano Banana Pro in the <a href="https://gemini.google.com/" rel="noopener noreferrer">Gemini app</a>, the best environment for developers to prototype and test prompts is <a href="https://aistudio.google.com/banana-pro" rel="noopener noreferrer">Google AI Studio</a>. AI Studio is a playground to experiment with all available AI models before writing any code, and it's also the entry point for building with the Gemini API.</p>

<p>You can use Nano Banana Pro within AI Studio. To get started, go to <a href="https://aistudio.google.com/banana-pro" rel="noopener noreferrer">aistudio.google.com</a>, sign in with your Google account, and select <strong>Nano Banana Pro</strong> (Gemini 3 Pro Image) from the model picker.</p>

<p>Contrary to Nano-Banana, the pro version <strong>doesn't have a free tier</strong>, which means you need to select an API key with billing enabled (see "project setup" section below).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fidqeymcwrtjlk416q2ag.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fidqeymcwrtjlk416q2ag.png" alt="Get started with Nano Banana Pro on AI Studio" width="800" height="456"></a></p>

<blockquote>
<p><strong>Tip</strong>: You can also vibe code Nano Banana web apps directly in AI Studio at <a href="//ai.studio/apps">ai.studio/apps</a>, or explore the code and remix one of the <a href="https://aistudio.google.com/apps?source=showcase&amp;showcaseTag=nano-banana" rel="noopener noreferrer">existing apps</a>.</p>
</blockquote>

<h2>
  
  
  2) Project setup
</h2>

<p>To follow this guide, you will need the following:</p>

<ul>
<li>  An API key from <a href="https://aistudio.google.com/" rel="noopener noreferrer">Google AI Studio</a>.</li>
<li>  Billing set up for your project.</li>
<li>  The Google Gen AI SDK for <a href="https://github.com/googleapis/python-genai" rel="noopener noreferrer">Python</a> or <a href="https://github.com/googleapis/js-genai" rel="noopener noreferrer">JavaScript/TypeScript</a>.</li>
</ul>

<p>If you already are a hardcore Gemini API user with all of that, great! just skip this section and move to the next one. Otherwise, here's how to get started:</p>

<h3>
  
  
  Step A: Get your API Key
</h3>

<p>When you first log in on AI Studio, a Google Cloud project and an API key should be automatically created.</p>

<p>Open the <a href="https://aistudio.google.com/api-keys" rel="noopener noreferrer">API key management screen</a> and click on the "copy" icon to copy your API key.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkwglctk5jiunm7gdi207.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkwglctk5jiunm7gdi207.png" alt="Copy your API key" width="800" height="431"></a></p>

<h3>
  
  
  Step B: Enable Billing
</h3>

<p>Since Nano Banana Pro doesn't have a free tier. You must enable billing on your Google Cloud project.</p>

<p>In the <a href="https://aistudio.google.com/projects" rel="noopener noreferrer">API key management screen</a>, click <strong>Set up billing</strong> next to your project and follow the on-screen instructions.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn7wi4ygjj4utb8lrntsl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn7wi4ygjj4utb8lrntsl.png" alt="Set up billing" width="800" height="435"></a></p>

<h4>
  
  
  How much does Nano Banana Pro cost?
</h4>

<p>Image generation with Nano Banana Pro is more expensive than the Flash version, especially for 4K images. At the time this post is published, a 1K or 2K image costs $0.134, while a 4K one costs $0.24 (plus the token cost of the input and the text output).</p>

<p>Check the <a href="https://ai.google.dev/gemini-api/docs/pricing#gemini-3-pro-image-preview" rel="noopener noreferrer">pricing</a> in the documentation for the latest details.</p>

<h3>
  
  
  Step C: Install the SDK
</h3>

<p>Choose the SDK for your preferred language.</p>

<p><strong>Python:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install</span> <span class="nt">-U</span> google-genai
<span class="c"># Install the Pillow library for image manipulation</span>
pip <span class="nb">install </span>Pillow
</code></pre>

</div>



<p><strong>JavaScript / TypeScript:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @google/genai
</code></pre>

</div>



<blockquote>
<p>The following examples use the Python SDK for demonstration. Equivalent code snippets to use Nano Banana in JavaScript are provided in this <strong><a href="https://ai.studio/apps/bundled/get_started_image_out?fullscreenApplet=true" rel="noopener noreferrer">JS Notebook</a></strong>.</p>
</blockquote>

<h2>
  
  
  3) Initialize the Client
</h2>

<p>To use the Pro model, you'll need to use the <strong><code>gemini-3-pro-image-preview</code></strong> model ID.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google</span> <span class="kn">import</span> <span class="n">genai</span>
<span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>

<span class="c1"># Initialize the client
</span><span class="n">client</span> <span class="o">=</span> <span class="n">genai</span><span class="p">.</span><span class="nc">Client</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="sh">"</span><span class="s">YOUR_API_KEY</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Set the model ID
</span><span class="n">PRO_MODEL_ID</span> <span class="o">=</span> <span class="sh">"</span><span class="s">gemini-3-pro-image-preview</span><span class="sh">"</span>
</code></pre>

</div>



<h2>
  
  
  4) Basic Generation (The Classics)
</h2>

<p>Before we get into the fancy stuff, let's look at a standard generation. You can control the output using <code>response_modalities</code> (to get text and images or only images) and <code>aspect_ratio</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">prompt</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Create a photorealistic image of a siamese cat with a green left eye and a blue right one</span><span class="sh">"</span>
<span class="n">aspect_ratio</span> <span class="o">=</span> <span class="sh">"</span><span class="s">16:9</span><span class="sh">"</span> <span class="c1"># "1:1","2:3","3:2","3:4","4:3","4:5","5:4","9:16","16:9" or "21:9"
</span>
<span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">PRO_MODEL_ID</span><span class="p">,</span>
    <span class="n">contents</span><span class="o">=</span><span class="n">prompt</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">response_modalities</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">Text</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Image</span><span class="sh">'</span><span class="p">],</span> <span class="c1"># Or just ['Image']
</span>        <span class="n">image_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">ImageConfig</span><span class="p">(</span>
            <span class="n">aspect_ratio</span><span class="o">=</span><span class="n">aspect_ratio</span><span class="p">,</span>
        <span class="p">)</span>
    <span class="p">)</span>
<span class="p">)</span>

<span class="c1"># Display the image
</span><span class="k">for</span> <span class="n">part</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">parts</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">image</span><span class="p">:</span><span class="o">=</span> <span class="n">part</span><span class="p">.</span><span class="nf">as_image</span><span class="p">():</span>
        <span class="n">image</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="sh">"</span><span class="s">cat.png</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqrqb18ptavkufaqav7o5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqrqb18ptavkufaqav7o5.png" alt="Siamese cat" width="800" height="800"></a></p>

<p>Chat mode is also an option (it's actually what I would recommend for multi-turn editing). Check the 8th example, "Polyglot Banana", for an example.</p>

<h2>
  
  
  5) The "Thinking" Process (It's alive!)
</h2>

<p>Nano Banana Pro isn't just drawing; it's <em>thinking</em>. This means it can reason through your most complex, twisted prompts before generating an image. And the best part? You can peek into its brain!</p>

<p>To enable this, set <code>include_thoughts=True</code> in the <code>thinking_config</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">prompt</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Create an unusual but realistic image that might go viral</span><span class="sh">"</span>
<span class="n">aspect_ratio</span> <span class="o">=</span> <span class="sh">"</span><span class="s">16:9</span><span class="sh">"</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">PRO_MODEL_ID</span><span class="p">,</span>
    <span class="n">contents</span><span class="o">=</span><span class="n">prompt</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">response_modalities</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">Text</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Image</span><span class="sh">'</span><span class="p">],</span>
        <span class="n">image_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">ImageConfig</span><span class="p">(</span>
            <span class="n">aspect_ratio</span><span class="o">=</span><span class="n">aspect_ratio</span><span class="p">,</span>
        <span class="p">),</span>
        <span class="n">thinking_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">ThinkingConfig</span><span class="p">(</span>
            <span class="n">include_thoughts</span><span class="o">=</span><span class="bp">True</span> <span class="c1"># Enable thoughts
</span>        <span class="p">)</span>
    <span class="p">)</span>
<span class="p">)</span>

<span class="c1"># Display the image and thoughts
</span><span class="k">for</span> <span class="n">part</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">parts</span><span class="p">:</span>
  <span class="k">if</span> <span class="n">part</span><span class="p">.</span><span class="n">thought</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Thought: </span><span class="si">{</span><span class="n">part</span><span class="p">.</span><span class="n">text</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
  <span class="k">elif</span> <span class="n">image</span><span class="p">:</span><span class="o">=</span> <span class="n">part</span><span class="p">.</span><span class="nf">as_image</span><span class="p">():</span>
    <span class="n">image</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="sh">"</span><span class="s">viral.png</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmi6pn0b3tuqnm37o85n8.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmi6pn0b3tuqnm37o85n8.jpg" alt="Viral image" width="800" height="446"></a></p>

<p>This transparency helps you understand how the model interpreted your request. It's like having a conversation with your artist!</p>

<h2>
  
  
  6) Search Grounding (Real-time magic)
</h2>

<p>One of the most game-changing features is <strong>Search Grounding</strong>. Nano Banana Pro isn't stuck in the past; it can access real-time data from Google Search to generate accurate, up-to-date images. Want the weather? You got it.</p>

<p>For example, you can ask it to visualize the <em>current</em> weather forecast:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">prompt</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Visualize the current weather forecast for the next 5 days in Tokyo as a clean, modern weather chart. add a visual on what i should wear each day</span><span class="sh">"</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">PRO_MODEL_ID</span><span class="p">,</span>
    <span class="n">contents</span><span class="o">=</span><span class="n">prompt</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">response_modalities</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">Text</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Image</span><span class="sh">'</span><span class="p">],</span>
        <span class="n">image_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">ImageConfig</span><span class="p">(</span>
            <span class="n">aspect_ratio</span><span class="o">=</span><span class="sh">"</span><span class="s">16:9</span><span class="sh">"</span><span class="p">,</span>
        <span class="p">),</span>
        <span class="n">tools</span><span class="o">=</span><span class="p">[{</span><span class="sh">"</span><span class="s">google_search</span><span class="sh">"</span><span class="p">:</span> <span class="p">{}}]</span> <span class="c1"># Enable Google Search
</span>    <span class="p">)</span>
<span class="p">)</span>

<span class="c1"># Save the image
</span><span class="k">for</span> <span class="n">part</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">parts</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">image</span><span class="p">:</span><span class="o">=</span> <span class="n">part</span><span class="p">.</span><span class="nf">as_image</span><span class="p">():</span>
        <span class="n">image</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="sh">"</span><span class="s">weather.png</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Display sources (you must always do that)
</span><span class="nf">print</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">candidates</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">grounding_metadata</span><span class="p">.</span><span class="n">search_entry_point</span><span class="p">.</span><span class="n">rendered_content</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbgg57xg6mvtzk521zslw.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbgg57xg6mvtzk521zslw.jpg" alt="Weather in Tokyo" width="800" height="446"></a></p>

<h2>
  
  
  7) Go Big or Go Home: 4K Generation
</h2>

<p>Need print-quality images? Nano Banana Pro supports 4K resolution. Because sometimes, bigger <em>is</em> better.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">prompt</span> <span class="o">=</span> <span class="sh">"</span><span class="s">A photo of an oak tree experiencing every season</span><span class="sh">"</span>
<span class="n">resolution</span> <span class="o">=</span> <span class="sh">"</span><span class="s">4K</span><span class="sh">"</span> <span class="c1"># Options: "1K", "2K", "4K", be careful lower case do not work.
</span>
<span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">PRO_MODEL_ID</span><span class="p">,</span>
    <span class="n">contents</span><span class="o">=</span><span class="n">prompt</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">response_modalities</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">Text</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Image</span><span class="sh">'</span><span class="p">],</span>
        <span class="n">image_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">ImageConfig</span><span class="p">(</span>
            <span class="n">aspect_ratio</span><span class="o">=</span><span class="sh">"</span><span class="s">1:1</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">image_size</span><span class="o">=</span><span class="n">resolution</span>
        <span class="p">)</span>
    <span class="p">)</span>
<span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7d7g9bjm3w1mtu5oa9pr.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7d7g9bjm3w1mtu5oa9pr.jpg" alt="Oak tree experiencing all seasons" width="800" height="800"></a></p>

<p><em>Note: 4K generation comes at a higher cost, so use it wisely!</em></p>

<h2>
  
  
  8) Polyglot Banana (Multilingual Capabilities)
</h2>

<p>The model can generate and even translate text within images across over a dozen languages. It's basically a universal translator for your eyes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Generate an infographic in Spanish
</span><span class="n">message</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Make an infographic explaining Einstein</span><span class="sh">'</span><span class="s">s theory of General Relativity suitable for a 6th grader in Spanish</span><span class="sh">"</span>

<span class="n">response</span> <span class="o">=</span> <span class="n">chat</span><span class="p">.</span><span class="nf">send_message</span><span class="p">(</span><span class="n">message</span><span class="p">,</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
        <span class="n">image_config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">ImageConfig</span><span class="p">(</span><span class="n">aspect_ratio</span><span class="o">=</span><span class="sh">"</span><span class="s">16:9</span><span class="sh">"</span><span class="p">)</span>
    <span class="p">)</span>
<span class="p">)</span>

<span class="c1"># Save the image
</span><span class="k">for</span> <span class="n">part</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">parts</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">image</span><span class="p">:</span><span class="o">=</span> <span class="n">part</span><span class="p">.</span><span class="nf">as_image</span><span class="p">():</span>
        <span class="n">image</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="sh">"</span><span class="s">relativity.png</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvw5sj8d1e33wdu0kdxbc.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvw5sj8d1e33wdu0kdxbc.jpg" alt="General Relativity in Spanish" width="800" height="446"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Translate it to Japanese
</span><span class="n">message</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Translate this infographic in Japanese, keeping everything else the same</span><span class="sh">"</span>
<span class="n">response</span> <span class="o">=</span> <span class="n">chat</span><span class="p">.</span><span class="nf">send_message</span><span class="p">(</span><span class="n">message</span><span class="p">)</span>

<span class="c1"># Save the image
</span><span class="k">for</span> <span class="n">part</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">parts</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">image</span><span class="p">:</span><span class="o">=</span> <span class="n">part</span><span class="p">.</span><span class="nf">as_image</span><span class="p">():</span>
        <span class="n">image</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="sh">"</span><span class="s">relativity_JP.png</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1z8v17jqcxi9rp4vafdo.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1z8v17jqcxi9rp4vafdo.jpg" alt="General Relativity in Japanese" width="800" height="446"></a></p>

<h2>
  
  
  9) Mix it up! (Advanced Image Mixing)
</h2>

<p>While the Flash model can mix up to 3 images, the Pro model can handle up to <strong>14 images</strong>! That's a whole party in one prompt. Perfect for creating complex collages or showing off your entire product line.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Mix multiple images
</span><span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">PRO_MODEL_ID</span><span class="p">,</span>
    <span class="n">contents</span><span class="o">=</span><span class="p">[</span>
        <span class="sh">"</span><span class="s">An office group photo of these people, they are making funny faces.</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">PIL</span><span class="p">.</span><span class="n">Image</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">'</span><span class="s">John.png</span><span class="sh">'</span><span class="p">),</span>
        <span class="n">PIL</span><span class="p">.</span><span class="n">Image</span><span class="p">.</span><span class="nf">open</span><span class="p">(</span><span class="sh">'</span><span class="s">Jane.png</span><span class="sh">'</span><span class="p">),</span>
        <span class="c1"># ... add up to 14 images
</span>    <span class="p">],</span>
<span class="p">)</span>

<span class="c1"># Save the image
</span><span class="k">for</span> <span class="n">part</span> <span class="ow">in</span> <span class="n">response</span><span class="p">.</span><span class="n">parts</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">image</span><span class="p">:</span><span class="o">=</span> <span class="n">part</span><span class="p">.</span><span class="nf">as_image</span><span class="p">():</span>
        <span class="n">image</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="sh">"</span><span class="s">group_picture.png</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5gpi2avw16veccv9leyv.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5gpi2avw16veccv9leyv.jpeg" alt=" " width="800" height="446"></a></p>

<p><em>Note: If you want very high fidelity for your characters, limit yourself to 5, which is already more than enough for a party night!</em></p>

<h2>
  
  
  10) Show off time! (Pro-Exclusive Demos)
</h2>

<p>Here are some examples of what's possible only with Nano Banana Pro. Prepare to be amazed:</p>

<h3>
  
  
  Personalized Pixel Art (Search Grounding)
</h3>

<p><em>Prompt: "Search the web then generate an image of isometric perspective, detailed pixel art that shows the career of Guillaume Vernade"</em></p>

<p>This uses search grounding to find specific information about a person and visualizes it in a specific style.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcor096x1iq8jlct5nvl1.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcor096x1iq8jlct5nvl1.jpg" alt="Guillaume Vernade career" width="800" height="446"></a></p>

<h3>
  
  
  Complex Text Integration
</h3>

<p><em>Prompt: "Show me an infographic about how sonnets work, using a sonnet about bananas written in it, along with a lengthy literary analysis of the poem. Good vintage aesthetics"</em></p>

<p>The model can generate coherent, lengthy text and integrate it perfectly into a complex layout.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp0xg8dnb2x567xtmzpl7.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp0xg8dnb2x567xtmzpl7.jpg" alt=" " width="800" height="446"></a></p>

<h3>
  
  
  High-Fidelity Mockups
</h3>

<p><em>Prompt: "A photo of a program for the Broadway show about TCG players on a nice theater seat, it's professional and well made, glossy, we can see the cover and a page showing a photo of the stage."</em></p>

<p>Create photorealistic mockups of print materials with accurate lighting and texture.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh0xb4n9fn0rvlt6s5ljj.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh0xb4n9fn0rvlt6s5ljj.jpg" alt=" " width="800" height="446"></a></p>

<h2>
  
  
  11) Best Practices and prompting tips for Nano Banana and Nano Banana Pro
</h2>

<p>To achieve the best results with the Nano Banana models, follow these prompting guidelines:</p>

<p><strong>Be Hyper-Specific:</strong> The more detail you provide about subjects, colors, lighting, and composition, the more control you have over the output.<br>
<strong>Provide Context and Intent:</strong> Explain the purpose or desired mood of the image. The model's understanding of context will influence its creative choices.<br>
<strong>Iterate and Refine:</strong> Don't expect perfection on the first try. Use the model's conversational ability to make incremental changes and refine your image.<br>
<strong>Use Step-by-Step Instructions:</strong> For complex scenes, break your prompt into a series of clear, sequential instructions.<br>
<strong>Use Positive Framing:</strong> Instead of negative prompts like "no cars," describe the desired scene positively: "an empty, deserted street with no signs of traffic."<br>
<strong>Control the Camera:</strong> Use photographic and cinematic terms to direct the composition, such as "wide-angle shot", "macro shot", or "low-angle perspective".<br>
<strong>Use search grounding to your advantage:</strong> When you know that you want the model to use real-time or real-world data, be very precise about it. "Search the web about the last Olympic Lyonnais's game and make an infographics" will work better than just "an infographics of the OL last games" (which should still work, but don't take chances).</p>

<p>For a deeper dive into best practices, check the <a href="https://ai.google.dev/gemini-api/docs/image-generation#prompt-guide" rel="noopener noreferrer">prompting guide</a> in the documentation and the <a href="https://developers.googleblog.com/en/how-to-prompt-gemini-2-5-flash-image-generation-for-the-best-results/" rel="noopener noreferrer">prompting best practices</a> for Nano Banana publish on the official blog.</p>

<h2>
  
  
  Wrap up
</h2>

<p>Nano Banana Pro (Gemini 3 Pro Image) opens up a new frontier for AI image generation. With its ability to think, search, and render in 4K, it's a tool for serious creators (and serious fun).</p>

<p>Ready to try it out? Head over to <a href="https://aistudio.google.com/" rel="noopener noreferrer">Google AI Studio</a>, try or customize our <a href="https://aistudio.google.com/apps?source=showcase&amp;showcaseTag=nano-banana" rel="noopener noreferrer">Apps</a> or check out the <a href="https://colab.sandbox.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_Started_Nano_Banana.ipynb" rel="noopener noreferrer">cookbook</a>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcng2t1e9qmky91on2k7h.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcng2t1e9qmky91on2k7h.jpeg" alt=" " width="800" height="446"></a></p>

