---
Title: drop_down_menu.js Module in WebForms Core
Description: 
Author: Elanat Framework
Date: 2026-02-09T22:09:56.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What is WebForms Core?
</h2>

<p><a href="https://github.com/webforms-core" rel="noopener noreferrer">WebForms Core</a> is a modern technology from <a href="https://elanat.net" rel="noopener noreferrer">Elanat</a>, introduced in 2024, that is actually a new multi-paradigm in web development. This technology is a two-way protocol that establishes automatic communication between a server-side class called WebForms (available for various programming languages ​​such as C#, PHP, Python, and Java) and a client-side JavaScript library called WebFormsJS. Its main goal is to manage HTML elements and manipulate the DOM directly from the server, in a way that largely eliminates the need for extensive front-end development and writing complex client-side code. In this model, instead of sending large amounts of data or the entire HTML structure, the server sends concise commands in the form of INI format to the client for precise and targeted updates, which leads to reduced bandwidth consumption, near-zero latency, and high server scalability. This approach addresses the common challenges of front-end framework complexity, high dependencies, and large node_modules, and encourages developers to focus more on server-side logic.</p>

<h2>
  
  
  How to use WebForms Core technology?
</h2>

<p>Two steps are required:<br>
1- <strong>On the client side</strong>: Add the WebFormsJS script to your HTML page.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"module"</span> <span class="na">src=</span><span class="s">"/script/web-forms.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
</code></pre>

</div>



<p>Get the WebFormsJS script from the following link:<br>
<a href="https://github.com/webforms-core/Web_forms/blob/elanat_framework/web-forms.js" rel="noopener noreferrer">https://github.com/webforms-core/Web_forms/blob/elanat_framework/web-forms.js</a></p>

<p>2- <strong>On the server side</strong>: Import the WebForms class for your programming language.<br>
Get the WebForms class associated with the server programming language from the following link:<br>
<a href="https://github.com/webforms-core/Web_forms_classes" rel="noopener noreferrer">https://github.com/webforms-core/Web_forms_classes</a></p>
<h2>
  
  
  Server-side JavaScript Module Loading Feature
</h2>

<p>One of the standout features of WebForms Core is the ability to <strong>dynamically load JavaScript modules from the server</strong>. This feature revolutionizes the way client code is managed and executed and offers several benefits:</p>
<h3>
  
  
  Benefits of server-side JS module loading:
</h3>

<ol>
<li>
<strong>Intelligent Lazy Loading</strong>: JavaScript modules are loaded only when needed</li>
<li>
<strong>Better memory management</strong>: Prevents unnecessary code loading in the browser</li>
<li>
<strong>Increased performance</strong>: Reduces initial page load time</li>
<li>
<strong>Higher security</strong>: Better control over code executed on the client side</li>
<li>
<strong>Maintainability</strong>: Centralized module management on the server</li>
</ol>
<h3>
  
  
  How it works:
</h3>

<p>WebForms Core loads JavaScript files from the specified path using the <code>LoadModule()</code> method and then makes specific functions of that module available. This mechanism allows for dynamic interaction between the server and the client.</p>
<h2>
  
  
  Introducing the example: DropDown Menu module
</h2>
<h3>
  
  
  Exploring the <code>drop_down_menu.js</code> module
</h3>

<p>The dropdown menu module is a practical example of WebForms Core capabilities that demonstrates how complex UI components can be created by working together on the server and client. This module is available at the following GitHub address:</p>

<p><strong>Module address</strong>: <a href="https://github.com/webforms-core/Web_forms/blob/elanat_framework/module/menu/drop_down_menu.js" rel="noopener noreferrer">https://github.com/webforms-core/Web_forms/blob/elanat_framework/module/menu/drop_down_menu.js</a></p>
<h3>
  
  
  Main methods of this module:
</h3>

<ol>
<li>
<strong><code>ddm_AddItem(text, url, level)</code></strong>: To add new items to the menu</li>
</ol>

<ul>
<li>The <code>level</code> parameter specifies the hierarchical structure of the menu</li>
</ul>

<ol>
<li><p><strong><code>ddm_ResetMenu()</code></strong>: To clear all menu items and reset it</p></li>
<li><p><strong><code>ddm_Render(className, appendMenuCSS)</code></strong>: To render the menu and generate HTML</p></li>
</ol>

<ul>
<li>Automatically injects the required CSS</li>
</ul>

<ol>
<li><p><strong><code>ddm_GetMenuCSS(className)</code></strong>: To get the menu CSS styles</p></li>
<li><p><strong><code>ddm_InjectMenuCSS(className)</code></strong>: To inject styles into the head section Page</p></li>
</ol>
<h3>
  
  
  Example of use in server (C#):
</h3>

<p>This example was created using the Elanat team's backend framework, <a href="https://github.com/elanatframework/Code_behind" rel="noopener noreferrer">CodeBehind</a>. You can use WebForms Core technology in all popular web programming languages.</p>

<blockquote>
<p>Note: WebForms Core technology and the CodeBehind framework are similar in naming to Microsoft's former WebForms; however, they are both powerful, modern systems that offer advanced functionality and operate differently from Microsoft's WebForms.</p>
</blockquote>

<p><strong>Project Root</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>├── WebForms.cs          // WebForms Class in Server
├── Controller.cs
├── wwwroot
│   ├── page.aspx
│   └── script
│       ├── web-forms.js // WebFormsJS in Client
│       └── module
│           └── drop_down_menu.js
</code></pre>

</div>



<p><strong>View (page.aspx)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>@page
@controller ModuleDropDownMenuController
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
<span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;title&gt;</span>DropDown Menu Module<span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"module"</span> <span class="na">src=</span><span class="s">"/script/web-forms.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
<span class="nt">&lt;/head&gt;</span>
<span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;h1&gt;</span>WebForms Core - DropDown Menu Module<span class="nt">&lt;/h1&gt;</span>
    <span class="nt">&lt;button</span> <span class="na">id=</span><span class="s">"Button1"</span><span class="nt">&gt;</span>Click me!<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;p</span> <span class="na">id=</span><span class="s">"pTag"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/p&gt;</span>
<span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span> 
</code></pre>

</div>



<p><strong>Server Response (Controller.cs)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">CodeBehind</span><span class="p">;</span>

<span class="k">public</span> <span class="k">partial</span> <span class="k">class</span> <span class="nc">ModuleDropDownMenuController</span> <span class="p">:</span> <span class="n">CodeBehindController</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>       
        <span class="k">if</span> <span class="p">(</span><span class="n">context</span><span class="p">.</span><span class="n">Request</span><span class="p">.</span><span class="n">Query</span><span class="p">.</span><span class="nf">ContainsKey</span><span class="p">(</span><span class="s">"action"</span><span class="p">))</span>
        <span class="p">{</span>
            <span class="nf">Button1_OnClick</span><span class="p">(</span><span class="n">context</span><span class="p">);</span>
            <span class="k">return</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="n">WebForms</span> <span class="n">form</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">WebForms</span><span class="p">();</span>

        <span class="n">form</span><span class="p">.</span><span class="nf">SetGetEvent</span><span class="p">(</span><span class="s">"Button1"</span><span class="p">,</span> <span class="n">HtmlEvent</span><span class="p">.</span><span class="n">OnClick</span><span class="p">,</span> <span class="s">"?action"</span><span class="p">);</span>

        <span class="nf">Control</span><span class="p">(</span><span class="n">form</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">private</span> <span class="k">void</span> <span class="nf">Button1_OnClick</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">WebForms</span> <span class="n">form</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">WebForms</span><span class="p">();</span>

        <span class="n">form</span><span class="p">.</span><span class="nf">LoadModule</span><span class="p">(</span><span class="s">"/script/module/drop_down_menu.js"</span><span class="p">,</span> <span class="p">[</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="s">"ddm_Render"</span><span class="p">]);</span>

        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Home"</span><span class="p">,</span> <span class="s">"/"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Product"</span><span class="p">,</span> <span class="s">"#"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Laptop"</span><span class="p">,</span> <span class="s">"/laptops"</span><span class="p">,</span> <span class="s">"-"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Gaming"</span><span class="p">,</span> <span class="s">"/laptops/gaming"</span><span class="p">,</span> <span class="s">"--"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Office"</span><span class="p">,</span> <span class="s">"/laptops/office"</span><span class="p">,</span> <span class="s">"--"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Mobile"</span><span class="p">,</span> <span class="s">"/mobiles"</span><span class="p">,</span> <span class="s">"-"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Budget"</span><span class="p">,</span> <span class="s">"/mobiles/budget"</span><span class="p">,</span> <span class="s">"--"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Midrange"</span><span class="p">,</span> <span class="s">"/mobiles/midrange"</span><span class="p">,</span> <span class="s">"--"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Flagship"</span><span class="p">,</span> <span class="s">"/mobiles/flagship"</span><span class="p">,</span> <span class="s">"--"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Contact"</span><span class="p">,</span> <span class="s">"/contact"</span><span class="p">]);</span>
        <span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"About"</span><span class="p">,</span> <span class="s">"/about"</span><span class="p">]);</span>

        <span class="n">form</span><span class="p">.</span><span class="nf">SetText</span><span class="p">(</span><span class="s">"pTag"</span><span class="p">,</span> <span class="n">Fetch</span><span class="p">.</span><span class="nf">ModuleMethod</span><span class="p">(</span><span class="s">"ddm_Render"</span><span class="p">));</span>

        <span class="nf">Control</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="k">true</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Page request
</h2>

<p>When the "page.aspx" page is requested by the user's browser, a new instance of the controller named "ModuleDropDownMenuController" is created and the <code>PageLoad</code> method is called.</p>

<p>In this method, the presence of the "action" query in the request is first checked, if it is present, the "Button1_OnClick" method is executed.</p>

<p>Next, a new instance of the WebForms class is created and a click event is requested from the server for the HTML button with id "Button1" and the query "?action" is assigned.</p>

<p>Calling the <code>Control</code> method causes the following HTML comment (containing the WebForms commands) to be added to the "page.aspx" page.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>...
<span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
<span class="c">&lt;!--[web-forms]
EgButton1=onclick|?load--&gt;</span>
</code></pre>

</div>



<h2>
  
  
  <code>Button1_OnClick</code> method
</h2>

<h3>
  
  
  Purpose of the function
</h3>

<p>This function handles the click event on the <code>Button1</code> button and creates a drop-down menu and displays it on the screen.</p>

<h3>
  
  
  Steps to execute the function:
</h3>

<h4>
  
  
  1. <strong>Create a WebForms object</strong>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">WebForms</span> <span class="n">form</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">WebForms</span><span class="p">();</span>
</code></pre>

</div>



<p>A new instance of the WebForms class is created that handles the communication with the client side.</p>

<h4>
  
  
  2. <strong>Load the JavaScript module</strong>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">LoadModule</span><span class="p">(</span><span class="s">"/script/module/drop_down_menu.js"</span><span class="p">,</span> <span class="p">[</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="s">"ddm_Render"</span><span class="p">]);</span>
</code></pre>

</div>



<p>This command does two things:</p>

<ul>
<li>Loads the <code>drop_down_menu.js</code> file from the specified path on the client side</li>
<li>Makes the two functions <code>ddm_AddItem</code> and <code>ddm_Render</code> available from this module</li>
</ul>

<h4>
  
  
  3. <strong>Adding menu items</strong>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Home"</span><span class="p">,</span> <span class="s">"/"</span><span class="p">]);</span>
<span class="n">form</span><span class="p">.</span><span class="nf">CallModuleMethod</span><span class="p">(</span><span class="s">"ddm_AddItem"</span><span class="p">,</span> <span class="p">[</span><span class="s">"Product"</span><span class="p">,</span> <span class="s">"#"</span><span class="p">]);</span>
<span class="c1">// and more...</span>
</code></pre>

</div>



<p>Each of these commands adds an item to the menu. Parameters in order:</p>

<ol>
<li>Item text (e.g. "Home")</li>
<li>Link address (e.g. "/")</li>
<li>Level in hierarchy (optional)</li>
</ol>

<p><strong>How ​​to display hierarchy:</strong></p>

<ul>
<li>No parameter: Main item</li>
<li>
<code>"-"</code>: Level 1 submenu</li>
<li>
<code>"--"</code>: Level 2 submenu</li>
<li>
<code>"---"</code>: Level 3 submenu and so on</li>
</ul>

<h4>
  
  
  4. <strong>Rendering and displaying the menu</strong>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">SetText</span><span class="p">(</span><span class="s">"pTag"</span><span class="p">,</span> <span class="n">Fetch</span><span class="p">.</span><span class="nf">ModuleMethod</span><span class="p">(</span><span class="s">"ddm_Render"</span><span class="p">));</span>
</code></pre>

</div>



<p>This command:</p>

<ol>
<li>Calls the <code>ddm_Render</code> module function</li>
<li>Gets the HTML output of the menu</li>
<li>Replaces the content of the element with <code>id="pTag"</code> with the HTML code of the menu</li>
</ol>

<h4>
  
  
  5. <strong>Sending commands to the client</strong>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="nf">Control</span><span class="p">(</span><span class="n">form</span><span class="p">,</span> <span class="k">true</span><span class="p">);</span>
</code></pre>

</div>



<blockquote>
<p>Note: Setting the value "true" as the second argument to the 'Control' function will prevent View and Layout from being re-sent in the response.</p>
</blockquote>

<p>Sends all WebForms commands set so far to the client as an INI-formatted response.</p>

<h2>
  
  
  Final output in browser:
</h2>

<p>After clicking the button, the following commands are automatically sent from the server to the client in INI format.</p>

<p><strong>Action Controls</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[web-forms]
Ml=/script/module/drop_down_menu.js|ddm_AddItem|ddm_Render
lM=ddm_AddItem|Home|/
lM=ddm_AddItem|Product|#
lM=ddm_AddItem|Laptop|/laptops|-
lM=ddm_AddItem|Gaming|/laptops/gaming|--
lM=ddm_AddItem|Office|/laptops/office|--
lM=ddm_AddItem|Mobile|/mobiles|-
lM=ddm_AddItem|Budget|/mobiles/budget|--
lM=ddm_AddItem|Midrange|/mobiles/midrange|--
lM=ddm_AddItem|Flagship|/mobiles/flagship|--
lM=ddm_AddItem|Contact|/contact
lM=ddm_AddItem|About|/about
stpTag=@cMddm_Render
</code></pre>

</div>



<p><strong>Created hierarchical structure</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Home
Product
  - Laptop
    - Gaming
    - Office
  - Mobile
    - Budget
    - Midrange
    - Flagship
Contact
About
</code></pre>

</div>



<p><strong>Screenshot</strong></p>

<p>The following screenshot shows the rendered dropdown menu after clicking the button.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgviluxhmu8bs6uvjnh5l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgviluxhmu8bs6uvjnh5l.png" alt="DropDown Menu" width="686" height="579"></a></p>

<h2>
  
  
  Summary
</h2>

<p>WebForms Core offers a new approach to web application development by providing the ability to dynamically load JavaScript modules from the server. The example of the drop-down menu module is a good example of how this feature can be used to create interactive and dynamic components.</p>

<p>This architecture not only increases the performance of applications, but also simplifies their maintenance and development. By separating business logic on the server and intelligently managing client resources, WebForms Core provides an ideal solution for modern web projects.</p>

<p>The complete drop-down menu module, along with server and client code, is available from the GitHub repository and can be used as a template for developing other custom modules.</p>

