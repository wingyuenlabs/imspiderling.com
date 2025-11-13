---
Title: Inline editing with custom elements in Rails
Description: 
Author: Rails Designer
Date: 2025-11-13T21:15:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>This article was originally published on <a href="https://railsdesigner.com/custom-element-inline-edit/" rel="noopener noreferrer">Rails Designer</a></p>




<p>How would you tackle this feature in a typical Hotwired Rails app: a HTML-element (like this <code>h1</code>) gets editable on click and when focus is removed, the record is updated.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm4locf8u6omea86hb2vc.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm4locf8u6omea86hb2vc.gif" alt="Inline editing demo" width="1000" height="712"></a></p>

<p>How about I tell you, it is done using just this HTML:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight erb"><code><span class="nt">&lt;editable-content</span> <span class="na">url=</span><span class="s">"</span><span class="cp">&lt;%=</span> <span class="n">post_path</span><span class="p">(</span><span class="vi">@post</span><span class="p">)</span> <span class="cp">%&gt;</span><span class="s">"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;h1</span> <span class="na">name=</span><span class="s">"post[title]"</span><span class="nt">&gt;</span>
    <span class="cp">&lt;%=</span> <span class="vi">@post</span><span class="p">.</span><span class="nf">title</span> <span class="cp">%&gt;</span>
  <span class="nt">&lt;/h1&gt;</span>
<span class="nt">&lt;/editable-content&gt;</span>

<span class="nt">&lt;div&gt;</span>
  <span class="cp">&lt;%=</span> <span class="n">simple_format</span> <span class="vi">@post</span><span class="p">.</span><span class="nf">content</span> <span class="cp">%&gt;</span>
<span class="nt">&lt;/div&gt;</span>
</code></pre>

</div>



<p>Yes! 🤯 It is made possible using a custom element: <code>editable-content</code>. It is part of a little experimentation phase of custom elements I am currently in.</p>

<p>In this article I want to show how you can create such a custom element. As always the <a href="https://github.com/rails-designer-repos/editable-custom-element" rel="noopener noreferrer">code can be found here</a>.</p>

<h2>
  
  
  What are custom elements?
</h2>

<p>But first: what are custom elements? If you are familiar in this place on the web, you have seen them mentioned before. If not: custom elements are part of the Web Components standard. They let you define your own HTML tags with custom behavior. The beauty is that they are just HTML. You can style them with CSS, use them in your templates and they work without any framework. They have lifecycle callbacks like <code>connectedCallback</code> (when the element is added to the page) and <code>disconnectedCallback</code> (when removed) that make them perfect for encapsulating interactive behavior. Similar to Stimulus controllers. An article that does a deep(er) dive into them is scheduled. 🤫</p>

<p>For this inline editing feature, the custom element handles everything: detecting clicks, creating input fields, saving changes and restoring the display. All wrapped in a clean HTML tag.</p>

<h2>
  
  
  Setting up the custom element
</h2>

<p>Start by creating the JavaScript file for the custom element:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// app/javascript/components/editable_content.js</span>
<span class="kd">class</span> <span class="nc">EditableContent</span> <span class="kd">extends</span> <span class="nc">HTMLElement</span> <span class="p">{</span>
  <span class="nf">connectedCallback</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">click</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">edit</span><span class="p">(</span><span class="nx">event</span><span class="p">));</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">customElements</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="dl">"</span><span class="s2">editable-content</span><span class="dl">"</span><span class="p">,</span> <span class="nx">EditableContent</span><span class="p">);</span>
</code></pre>

</div>



<p>The <code>connectedCallback</code> runs when the element is added to the page. It sets up a click listener that triggers the editing flow. The <code>customElements.define</code> call registers the new tag name with the browser.</p>

<p>Import it in your application JavaScript:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code> import "@hotwired/turbo-rails"
 import "controllers"
<span class="gi">+
+import "components/editable_content"
</span></code></pre>

</div>



<p>And configure importmap to find the new directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code> pin_all_from "app/javascript/controllers", under: "controllers"
<span class="gi">+
+pin_all_from "app/javascript/components", under: "components"
</span></code></pre>

</div>



<p>Now you can use <code>&lt;editable-content&gt;</code> tags in your views. 🥳</p>

<h2>
  
  
  Making elements editable
</h2>

<p>The <code>#edit</code> method handles the click event and determines what should become editable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="err">#</span><span class="nf">edit</span><span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">target</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">editable</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">target</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">target</span> <span class="o">||</span> <span class="nx">target</span> <span class="o">===</span> <span class="k">this</span> <span class="o">||</span> <span class="o">!</span><span class="nx">target</span><span class="p">.</span><span class="nf">hasAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">))</span> <span class="k">return</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">field</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">create</span><span class="p">(</span><span class="nx">target</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">wrapper</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">wrap</span><span class="p">(</span><span class="nx">field</span><span class="p">);</span>
  <span class="nx">wrapper</span><span class="p">.</span><span class="nx">original</span> <span class="o">=</span> <span class="nx">target</span><span class="p">;</span>

  <span class="nx">target</span><span class="p">.</span><span class="nf">replaceWith</span><span class="p">(</span><span class="nx">wrapper</span><span class="p">);</span>
  <span class="nx">field</span><span class="p">.</span><span class="nf">focus</span><span class="p">();</span>
  <span class="nx">field</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">blur</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">save</span><span class="p">(</span><span class="nx">wrapper</span><span class="p">,</span> <span class="nx">field</span><span class="p">));</span>
<span class="p">}</span>
</code></pre>

</div>



<p>First it finds the actual editable element using <code>#editable</code>. This method walks up the DOM tree until it finds a direct child of the custom element. This means you can click anywhere inside an element (like in the middle of a heading) and the whole element becomes editable.</p>

<p>The method checks for a <code>name</code> attribute. This is required because it tells the server which field to update. Without it, the element is not editable.</p>

<p>Then it creates an input field, wraps it and replaces the original element. The wrapper stores a reference to the original element so it can be restored later. Finally it focuses the field and sets up a blur listener to save changes when the user clicks away.</p>

<h2>
  
  
  Creating the input field
</h2>

<p>The <code>#create</code> method builds the appropriate input element:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="err">#</span><span class="nf">create</span><span class="p">(</span><span class="nx">element</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">field</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nf">createElement</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">isMultiline</span><span class="p">(</span><span class="nx">element</span><span class="p">)</span> <span class="p">?</span> <span class="dl">"</span><span class="s2">textarea</span><span class="dl">"</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">input</span><span class="dl">"</span><span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nf">isMultiline</span><span class="p">(</span><span class="nx">element</span><span class="p">))</span> <span class="nx">field</span><span class="p">.</span><span class="nx">type</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">;</span>

  <span class="nx">field</span><span class="p">.</span><span class="nx">className</span> <span class="o">=</span> <span class="nx">element</span><span class="p">.</span><span class="nx">className</span><span class="p">;</span>
  <span class="nx">field</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="nx">element</span><span class="p">.</span><span class="nx">textContent</span><span class="p">.</span><span class="nf">trim</span><span class="p">();</span>
  <span class="nx">field</span><span class="p">.</span><span class="nx">name</span> <span class="o">=</span> <span class="nx">element</span><span class="p">.</span><span class="nf">getAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">);</span>

  <span class="nx">field</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">click</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">event</span><span class="p">.</span><span class="nf">stopPropagation</span><span class="p">());</span>

  <span class="k">return</span> <span class="nx">field</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It decides between a textarea and input based on the element type. Block-level elements like paragraphs get textareas. Inline elements like headings get text inputs. The field inherits the original element's classes so your CSS styling carries over. The click listener prevents the field from triggering another edit when clicked.</p>

<p>The <code>#isMultiline</code> helper checks the tag name:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="err">#</span><span class="nf">isMultiline</span><span class="p">(</span><span class="nx">element</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">[</span><span class="dl">"</span><span class="s2">p</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">div</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">blockquote</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">pre</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">article</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">section</span><span class="dl">"</span><span class="p">].</span><span class="nf">includes</span><span class="p">(</span><span class="nx">element</span><span class="p">.</span><span class="nx">tagName</span><span class="p">.</span><span class="nf">toLowerCase</span><span class="p">());</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Saving changes
</h2>

<p>When the field loses focus, the <code>#save</code> method sends the update to the server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">async</span> <span class="err">#</span><span class="nf">save</span><span class="p">(</span><span class="nx">wrapper</span><span class="p">,</span> <span class="nx">field</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">formData</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">FormData</span><span class="p">();</span>
  <span class="nx">formData</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="nx">field</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span> <span class="nx">field</span><span class="p">.</span><span class="nx">value</span><span class="p">);</span>

  <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">url</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">PATCH</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">X-CSRF-Token</span><span class="dl">"</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="err">#</span><span class="nx">csrfToken</span> <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">formData</span>
  <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">response</span><span class="p">.</span><span class="nx">ok</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">display</span> <span class="o">=</span> <span class="nx">wrapper</span><span class="p">.</span><span class="nx">original</span><span class="p">;</span>

  <span class="nx">display</span><span class="p">.</span><span class="nx">textContent</span> <span class="o">=</span> <span class="nx">field</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
  <span class="nx">wrapper</span><span class="p">.</span><span class="nf">replaceWith</span><span class="p">(</span><span class="nx">display</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It builds a FormData object with the field name and value (to matches Rails' parameter format). The fetch request includes the CSRF token for security. If the save succeeds, it updates the original element with the new value and swaps it back in place.</p>

<h2>
  
  
  Limitations and extensions
</h2>

<p>This implementation keeps things simple. It only handles text content without new lines. For rich text you would need a different approach. But for titles, labels and short descriptions it works great.</p>

<p>You can make fields more clearly editable with CSS. Add a hover effect or an edit icon to signal interactivity. For feedback on successful saves, you could add flash notifications or a subtle CSS animation on the field. The custom element makes it easy to extend with these features.</p>

<p>The beauty of custom elements is that they are just HTML. You can nest them, style them and combine them with other components. They work with Turbo, Stimulus and any other JavaScript you have. And because they use standard browser APIs, they are fast and reliable.</p>

<p>How do you like this approach?</p>

