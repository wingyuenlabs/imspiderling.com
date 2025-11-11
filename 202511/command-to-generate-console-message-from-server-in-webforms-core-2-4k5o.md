---
Title: Command to Generate Console Message from Server in WebForms Core 2
Description: 
Author: Elanat Framework
Date: 2025-11-11T21:23:42.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>In <strong>version 2 of Elanat’s <a href="https://elanat.net/page_content/web_forms_core" rel="noopener noreferrer">WebForms Core</a></strong> technology, a very useful feature has been introduced that allows developers to <strong>send messages from the server directly to the browser console</strong>.<br>
This feature is implemented through a new command called <code>ConsoleMessage</code>.</p>

<p>The ability to log messages from the server to the browser console is a powerful tool for <strong>debugging</strong>, <strong>monitoring client-side behavior</strong>, and <strong>enhancing the developer experience</strong>.</p>


<h2>
  
  
  The <code>ConsoleMessage</code> Command
</h2>

<p>The new function is defined as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="nf">ConsoleMessage</span><span class="p">(</span><span class="n">Text</span><span class="p">,</span> <span class="n">Type</span> <span class="p">=</span> <span class="s">"log"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Parameters:
</h3>

<ul>
<li><p><strong>Text</strong>:<br>
The text to be displayed in the browser console.</p></li>
<li><p><strong>Type (optional)</strong>:<br>
The type of message, which determines how it will appear (color and style) in the console.</p></li>
</ul>

<h4>
  
  
  Supported Types:
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>log</code></td>
<td>Regular message (default)</td>
</tr>
<tr>
<td><code>info</code></td>
<td>Informational message</td>
</tr>
<tr>
<td><code>warn</code></td>
<td>Warning message</td>
</tr>
<tr>
<td><code>error</code></td>
<td>Error message</td>
</tr>
<tr>
<td><code>debug</code></td>
<td>Debug information</td>
</tr>
<tr>
<td><code>trace</code></td>
<td>Execution trace</td>
</tr>
<tr>
<td><code>group</code></td>
<td>Starts a grouped section of console messages</td>
</tr>
<tr>
<td><code>groupend</code></td>
<td>Ends a group of messages</td>
</tr>
<tr>
<td><code>table</code></td>
<td>Displays data in a table format</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  The <code>ConsoleMessageAssert</code> Command
</h2>

<p>In addition to the main command, another related function is available:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="nf">ConsoleMessageAssert</span><span class="p">(</span><span class="n">Text</span><span class="p">,</span> <span class="n">Condition</span><span class="p">)</span>
</code></pre>

</div>



<p>This command displays a console message <strong>only when the specified condition is false</strong>.<br>
It works similarly to JavaScript’s <code>console.assert()</code> and is particularly useful for validating logic and verifying runtime conditions.</p>


<h2>
  
  
  Practical Examples
</h2>
<h3>
  
  
  1. Displaying a Simple Console Message
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="nf">ConsoleMessage</span><span class="p">(</span><span class="s">"Page loaded successfully"</span><span class="p">);</span>
</code></pre>

</div>


<p>Console Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Page loaded successfully
</code></pre>

</div>






<h3>
  
  
  2. Sending an Error Message from the Server
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="nf">ConsoleMessage</span><span class="p">(</span><span class="s">"Database connection failed"</span><span class="p">,</span> <span class="s">"error"</span><span class="p">);</span>
</code></pre>

</div>



<p>Console Output:<br>
⚠️ A red error message labeled <em>error</em> will appear.</p>




<h3>
  
  
  3. Conditional Message Assertion
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="nf">ConsoleMessageAssert</span><span class="p">(</span><span class="s">"Method not exist!"</span><span class="p">,</span> <span class="n">Fetch</span><span class="p">.</span><span class="nf">HasMethod</span><span class="p">(</span><span class="s">"myFunc"</span><span class="p">));</span>
</code></pre>

</div>



<p>If <code>userId</code> is null, the message will be displayed in the console.</p>




<h2>
  
  
  Importance of This Feature
</h2>

<ol>
<li><p><strong>Easier Client-Side Debugging:</strong><br>
Developers can send log messages directly from the server to the browser console, making it easier to monitor data flow and detect issues.</p></li>
<li><p><strong>Reduces Need for Third-Party Tools:</strong><br>
There’s no need for complex logging systems just to view runtime information in the browser.</p></li>
<li><p><strong>Enhances Server–Client Interaction:</strong><br>
This bridges the gap between server-side and client-side environments, bringing WebForms Core closer to modern web development patterns.</p></li>
<li><p><strong>Ideal for Development and Testing:</strong><br>
During module testing, developers can trace execution paths without affecting the end-user experience.</p></li>
</ol>




<h2>
  
  
  Conclusion
</h2>

<p>The new <code>ConsoleMessage</code> feature in <strong>WebForms Core 2</strong> represents an important step forward in modernizing the Elanat WebForms framework.<br>
By enabling direct console logging from the server, this capability greatly improves <strong>monitoring, testing, and debugging efficiency</strong>.</p>

<p>Alongside <code>ConsoleMessageAssert</code>, it provides developers with a precise toolset for validating logic and ensuring reliable program execution.</p>

<p>WebForms Core in GitHub:<br>
<a href="https://github.com/webforms-core" rel="noopener noreferrer">https://github.com/webforms-core</a></p>

