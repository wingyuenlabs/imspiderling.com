---
Title: 🧪 Unit Testing Tools in WebForms Core 2
Description: 
Author: Elanat Framework
Date: 2025-11-06T21:37:19.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://github.com/webforms-core" rel="noopener noreferrer">WebForms Core</a> technology consists of a client-side library called WebFormsJS and a server-side class called WebForms and the connection between the two. In this technology, you only need to add the WebFormsJS library in the head section of HTML and do the development in the WebForms class on the server in a command-oriented way. So the WebFormsJS library only plays a communication role and you do not develop it! The connection between the two is automatic and has been tested in advance and the probability of errors is very low; However, we at <a href="https://elanat.net" rel="noopener noreferrer">Elanat</a> have added the unit testing tool in version 2 of WebForms Core technology.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftyvlj524n320xwikdlrx.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftyvlj524n320xwikdlrx.jpg" alt="WebForms Core Unit Testing" width="800" height="533"></a></p>

<p>Unit testing is a critical part of modern web development, ensuring that components behave as expected and changes don’t introduce regressions. Before release of <strong>WebForms Core 2</strong>, we introduce a lightweight and powerful unit testing toolset has been introduced to help developers validate their HTML output directly in the browser. At the heart of this system is the <code>AssertEqual</code> function — a deep comparison utility for DOM elements.</p>

<p>Unlike traditional test frameworks, WebForms Core 2 introduces a <strong>built-in testing utility</strong> that runs directly in your WebForms Core environment without needing additional dependencies.</p>




<h2>
  
  
  🔧 What Is <code>AssertEqual</code>?
</h2>

<p><code>AssertEqual(InputPlace, Tag)</code> is a server method in WebForms class, designed to compare two DOM elements — one actual and one expected — and determine whether they are structurally and visually equivalent. It supports both HTML strings and DOM nodes as input, making it flexible for various testing scenarios.</p>

<p>It is recommended to use the <code>AssertEqualByOutputPlace</code> function. This function targets an existing tag instead of the full tag string.</p>

<p>Structure:<br>
<code>AssertEqualByOutputPlace(InputPlace, OutputPlace)</code></p>


<h2>
  
  
  🧠 How It Works
</h2>

<p>The function performs a multi-layered comparison:</p>

<ul>
<li>
<strong>Tag Name Matching</strong>: Ensures both elements use the same HTML tag.</li>
<li>
<strong>Attribute Comparison</strong>: Checks for missing, mismatched, or unexpected attributes.</li>
<li>
<strong>Class List Verification</strong>: Compares class names regardless of order.</li>
<li>
<strong>Inline Style Matching</strong>: Verifies that inline styles are identical.</li>
<li>
<strong>Form Value Validation</strong>: For form elements (<code>input</code>, <code>textarea</code>, <code>select</code>), it compares values and checked states.</li>
<li>
<strong>Child Node Recursion</strong>: Performs a deep comparison of child nodes, including text content and nested elements.</li>
</ul>

<p>All differences are logged using <code>console.info</code> and <code>console.warn</code>, providing clear feedback without interrupting execution.</p>


<h2>
  
  
  ✅ Key Advantages
</h2>

<p>Here’s why <code>AssertEqual</code> stands out as a unit testing tool:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Benefit</th>
</tr>
</thead>
<tbody>
<tr>
<td>🔍 Deep DOM Comparison</td>
<td>Validates entire structure, including children and text nodes</td>
</tr>
<tr>
<td>📝 Form Support</td>
<td>Handles values and states of form elements</td>
</tr>
<tr>
<td>🎨 Attribute &amp; Style Checks</td>
<td>Compares attributes, classes, and inline styles</td>
</tr>
<tr>
<td>📋 Clear Reporting</td>
<td>Logs readable messages for each mismatch</td>
</tr>
<tr>
<td>🧠 Simple API</td>
<td>Returns <code>true</code> or <code>false</code> for easy integration</td>
</tr>
<tr>
<td>⚙️ Minimalistic Design</td>
<td>No extra parameters or configuration needed</td>
</tr>
<tr>
<td>🔧 Extensible</td>
<td>Can be expanded to support events, computed styles, etc.</td>
</tr>
</tbody>
</table></div>


<h2>
  
  
  🚀 Getting Started
</h2>

<p>To use <code>AssertEqual</code>, simply pass the actual DOM element and the expected HTML string or node:</p>

<p><strong>Testing with HTML string</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">AssertEqual</span><span class="p">(</span><span class="s">"myButton"</span><span class="p">,</span> <span class="s">@"&lt;button id="</span><span class="n">myButton</span><span class="s">" class="</span><span class="n">primary</span><span class="s">"&gt;Click Me&lt;/button&gt;"</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Testing with element</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">AssertEqualByOutputPlace</span><span class="p">(</span><span class="s">"myButton1"</span><span class="p">,</span> <span class="s">"myButton2"</span><span class="p">);</span>
</code></pre>

</div>



<p>This makes it ideal for testing dynamic rendering, form generation, and component output in WebForms Core applications.</p>




<h2>
  
  
  ℹ️ Big Test
</h2>

<p>HTML<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"actualElement"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;form</span> <span class="na">id=</span><span class="s">"mainForm"</span> <span class="na">class=</span><span class="s">"form-container"</span> <span class="na">style=</span><span class="s">"background: white; padding: 20px;"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h2</span> <span class="na">class=</span><span class="s">"form-title"</span><span class="nt">&gt;</span>User Registration<span class="nt">&lt;/h2&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">id=</span><span class="s">"username"</span> <span class="na">class=</span><span class="s">"form-input"</span> <span class="na">value=</span><span class="s">"john_doe"</span> <span class="na">required</span><span class="nt">&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"email"</span> <span class="na">id=</span><span class="s">"email"</span> <span class="na">class=</span><span class="s">"form-input"</span> <span class="na">value=</span><span class="s">"john@example.com"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"checkbox-group"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"checkbox"</span> <span class="na">id=</span><span class="s">"newsletter"</span> <span class="na">checked</span><span class="nt">&gt;</span>
            <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"newsletter"</span><span class="nt">&gt;</span>Subscribe to newsletter<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;select</span> <span class="na">id=</span><span class="s">"country"</span> <span class="na">class=</span><span class="s">"form-select"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;option</span> <span class="na">value=</span><span class="s">"us"</span><span class="nt">&gt;</span>United States<span class="nt">&lt;/option&gt;</span>
            <span class="nt">&lt;option</span> <span class="na">value=</span><span class="s">"uk"</span> <span class="na">selected</span><span class="nt">&gt;</span>United Kingdom<span class="nt">&lt;/option&gt;</span>
        <span class="nt">&lt;/select&gt;</span>
        <span class="nt">&lt;textarea</span> <span class="na">id=</span><span class="s">"bio"</span> <span class="na">class=</span><span class="s">"form-textarea"</span><span class="nt">&gt;</span>Hello, I'm John!<span class="nt">&lt;/textarea&gt;</span>
        <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"submit"</span> <span class="na">id=</span><span class="s">"submitBtn"</span> <span class="na">class=</span><span class="s">"btn-primary"</span> <span class="na">disabled</span><span class="nt">&gt;</span>Register<span class="nt">&lt;/button&gt;</span>
    <span class="nt">&lt;/form&gt;</span>
<span class="nt">&lt;/div&gt;</span>

<span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"differentElement"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;form</span> <span class="na">id=</span><span class="s">"mainForm"</span> <span class="na">class=</span><span class="s">"form-wrapper"</span> <span class="na">style=</span><span class="s">"background: #f5f5f5; padding: 15px;"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;h2</span> <span class="na">class=</span><span class="s">"page-title"</span><span class="nt">&gt;</span>User Profile<span class="nt">&lt;/h2&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"text"</span> <span class="na">id=</span><span class="s">"username"</span> <span class="na">class=</span><span class="s">"input-field"</span> <span class="na">value=</span><span class="s">"jane_smith"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"email"</span> <span class="na">id=</span><span class="s">"email"</span> <span class="na">class=</span><span class="s">"input-field"</span> <span class="na">value=</span><span class="s">"jane@test.com"</span> <span class="na">required</span><span class="nt">&gt;</span>
        <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"options-group"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"checkbox"</span> <span class="na">id=</span><span class="s">"newsletter"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;label</span> <span class="na">for=</span><span class="s">"newsletter"</span><span class="nt">&gt;</span>Get newsletter updates<span class="nt">&lt;/label&gt;</span>
        <span class="nt">&lt;/div&gt;</span>
        <span class="nt">&lt;select</span> <span class="na">id=</span><span class="s">"country"</span> <span class="na">class=</span><span class="s">"dropdown-select"</span><span class="nt">&gt;</span>
            <span class="nt">&lt;option</span> <span class="na">value=</span><span class="s">"us"</span><span class="nt">&gt;</span>United States<span class="nt">&lt;/option&gt;</span>
            <span class="nt">&lt;option</span> <span class="na">value=</span><span class="s">"ca"</span> <span class="na">selected</span><span class="nt">&gt;</span>Canada<span class="nt">&lt;/option&gt;</span>
        <span class="nt">&lt;/select&gt;</span>
        <span class="nt">&lt;textarea</span> <span class="na">id=</span><span class="s">"bio"</span> <span class="na">class=</span><span class="s">"text-area"</span><span class="nt">&gt;</span>This is Jane's bio<span class="nt">&lt;/textarea&gt;</span>
        <span class="nt">&lt;button</span> <span class="na">type=</span><span class="s">"button"</span> <span class="na">id=</span><span class="s">"saveBtn"</span> <span class="na">class=</span><span class="s">"btn-success"</span><span class="nt">&gt;</span>Save Changes<span class="nt">&lt;/button&gt;</span>
        <span class="nt">&lt;span</span> <span class="na">class=</span><span class="s">"help-text"</span><span class="nt">&gt;</span>All fields are required<span class="nt">&lt;/span&gt;</span>
    <span class="nt">&lt;/form&gt;</span>
<span class="nt">&lt;/div&gt;</span>
</code></pre>

</div>



<p>Server code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">AssertEqualByOutputPlace</span><span class="p">(</span><span class="s">"actualElement"</span><span class="p">,</span> <span class="s">"differentElement"</span><span class="p">);</span>
</code></pre>

</div>



<p>Result in console<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>🔍 Start unit testing assert equal
ℹ️ [ASSERT FAIL] Class list mismatch: expected [form-wrapper] but got [form-container]
ℹ️ [ASSERT FAIL] Style mismatch: expected "background: #f5f5f5; padding: 15px;" but got "background: white; padding: 20px;"
ℹ️ [ASSERT FAIL] Text mismatch at child index 0: expected "User Profile" but got "User Registration"
ℹ️ [ASSERT FAIL] Attribute mismatch on "required": expected "undefined" but got ""
ℹ️ [ASSERT FAIL] Class list mismatch: expected [input-field] but got [form-input]
ℹ️ [ASSERT FAIL] Value mismatch in &lt;input&gt;: expected "jane_smith" but got "john_doe"
ℹ️ [ASSERT FAIL] Attribute mismatch on "required": expected "" but got "undefined"
ℹ️ [ASSERT FAIL] Class list mismatch: expected [input-field] but got [form-input]
ℹ️ [ASSERT FAIL] Value mismatch in &lt;input&gt;: expected "jane@test.com" but got "john@example.com"
ℹ️ [ASSERT FAIL] Class list mismatch: expected [options-group] but got [checkbox-group]
ℹ️ [ASSERT FAIL] Value mismatch in &lt;input&gt;: expected false but got true
ℹ️ [ASSERT FAIL] Text mismatch at child index 1: expected "Get newsletter updates" but got "Subscribe to newsletter"
ℹ️ [ASSERT FAIL] Class list mismatch: expected [dropdown-select] but got [form-select]
ℹ️ [ASSERT FAIL] Value mismatch in &lt;select&gt;: expected "ca" but got "uk"
ℹ️ [ASSERT FAIL] Class list mismatch: expected [text-area] but got [form-textarea]
ℹ️ [ASSERT FAIL] Text mismatch at child index 0: expected "This is Jane's bio" but got "Hello, I'm John!"
ℹ️ [ASSERT FAIL] Attribute mismatch on "type": expected "button" but got "submit"
ℹ️ [ASSERT FAIL] Class list mismatch: expected [btn-success] but got [btn-primary]
ℹ️ [ASSERT FAIL] Attribute mismatch on "disabled": expected "undefined" but got ""
ℹ️ [ASSERT FAIL] Text mismatch at child index 0: expected "Save Changes" but got "Register"
ℹ️ [ASSERT FAIL] Different number of child nodes: expected 8, got 7
⚠️ [ASSERT FAIL] Differences found
🔍 End unit testing assert equal
</code></pre>

</div>






<h2>
  
  
  🧩 Conclusion
</h2>

<p>WebForms Core 2 introduces a practical and elegant approach to unit testing with tools like <code>AssertEqual</code> and <code>AssertEqualByOutputPlace</code>. It empowers developers to write meaningful tests for their UI logic without relying on heavy frameworks. Whether you're validating a simple button or a complex form, this tool provides clarity, precision, and flexibility — all within a few lines of code.</p>

