---
Title: New Features for Better State Control in WebForms Core 2
Description: 
Author: Elanat Framework
Date: 2025-11-12T22:10:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>As you know, version 2 of WebForms Core technology will be released by <a href="https://elanat.net" rel="noopener noreferrer">Elanat</a> soon. In the past days, we have given an extensive description of the revolutionary features and capabilities of version 2, and in the next few days we will reveal other new features.</p>

<p>WebForms Core is a server-based technology, yet it offers revolutionary offline capabilities.</p>

<p>Think of this workflow:</p>

<ul>
<li>The server defines the interaction patterns</li>
<li>The client caches the JavaScript commands and calls</li>
<li>The user interacts with the application completely offline</li>
<li>The JavaScript runs locally without contacting the server</li>
<li>Synchronization occurs when the connection is resumed</li>
</ul>

<blockquote>
<p>This solves the problem of PWA complexity that plagues traditional SPA development.</p>
</blockquote>

<h2>
  
  
  Better state control
</h2>

<p>Version 2 of <a href="https://elanat.net/page_content/web_forms_core" rel="noopener noreferrer">WebForms Core</a> introduces a Checksum mechanism and SHA256 algorithm to enhance data integrity and security. This feature enables the server to verify whether an Action Control has been executed by comparing checksum or hash values generated on the client side.</p>

<p>Client-side functions like <code>SetHash</code> and <code>SetCheckSum</code> compute and store the hash in a global array, while the server uses <code>GetCheckSum</code> (WebForms class) and <code>HasHash</code> (Fetch class) to validate the hash and make logical decisions. New PostBack settings—such as <code>PostBackOptions.SendChecksum</code> and <code>PostBackOptions.ChecksumName</code>—support transmitting checksums during form submissions.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1wx742x265tvwss3sdh0.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1wx742x265tvwss3sdh0.jpg" alt="WebForms Core State Control" width="800" height="754"></a></p>

<p>Additional methods for cache and session management have also been added, allowing efficient data storage and retrieval.</p>

<h2>
  
  
  New Hash and Checsum features for status control
</h2>

<h3>
  
  
  Sending checksum of POST data
</h3>

<p>A new feature has been added in version 2 that calculates and sends the checksum when sending Post data. Enabling this option allows you to automatically calculate the checksum of the data when form data is sent to the server.</p>

<p>New settings in WebFormsJS<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Security</span>
<span class="p">...</span>
<span class="nx">WebFormsOptions</span><span class="p">.</span><span class="nx">SendChecksum</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
<span class="nx">WebFormsOptions</span><span class="p">.</span><span class="nx">ChecksumName</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">checksum</span><span class="dl">"</span><span class="p">;</span>
</code></pre>

</div>



<p>The above settings determine that:</p>

<ul>
<li>
<code>WebFormsOptions.SendChecksum = false</code>: This option specifies that the checksum is not sent when the form is submitted.</li>
<li>
<code>WebFormsOptions.ChecksumName = "checksum"</code>: The name of the checksum field in the form data is set to "checksum".</li>
</ul>

<p>After the data is sent, we can calculate the checksum values ​​on the server.</p>

<p>Server code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">CodeBehind</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Text.RegularExpressions</span><span class="p">;</span>

<span class="k">public</span> <span class="k">partial</span> <span class="k">class</span> <span class="nc">ChecksumController</span> <span class="p">:</span> <span class="n">CodeBehindController</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">requestForm</span> <span class="p">=</span> <span class="n">context</span><span class="p">.</span><span class="n">Request</span><span class="p">.</span><span class="n">Form</span><span class="p">;</span>

        <span class="kt">string</span> <span class="n">checksum</span> <span class="p">=</span> <span class="n">requestForm</span><span class="p">[</span><span class="s">"checksum"</span><span class="p">];</span>
        <span class="kt">string</span> <span class="n">requestBody</span> <span class="p">=</span> <span class="kt">string</span><span class="p">.</span><span class="nf">Join</span><span class="p">(</span><span class="s">"&amp;"</span><span class="p">,</span> <span class="n">requestForm</span><span class="p">.</span><span class="nf">Select</span><span class="p">(</span><span class="n">x</span> <span class="p">=&gt;</span> <span class="s">$"</span><span class="p">{</span><span class="n">x</span><span class="p">.</span><span class="n">Key</span><span class="p">}</span><span class="s">=</span><span class="p">{</span><span class="n">x</span><span class="p">.</span><span class="n">Value</span><span class="p">}</span><span class="s">"</span><span class="p">));</span>

        <span class="kt">string</span> <span class="n">requestBodyWithoutChecksum</span> <span class="p">=</span> <span class="n">Regex</span><span class="p">.</span><span class="nf">Replace</span><span class="p">(</span><span class="n">requestBody</span><span class="p">,</span> <span class="s">"&amp;?checksum=[^&amp;]*"</span><span class="p">,</span> <span class="s">""</span><span class="p">);</span>

        <span class="n">WebForms</span> <span class="n">form</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">WebForms</span><span class="p">();</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">form</span><span class="p">.</span><span class="nf">ChecksumCalculation</span><span class="p">(</span><span class="n">requestBodyWithoutChecksum</span><span class="p">)</span> <span class="p">==</span> <span class="n">checksum</span><span class="p">)</span>
            <span class="n">form</span><span class="p">.</span><span class="nf">Message</span><span class="p">(</span><span class="s">"Checksum value is the same"</span><span class="p">);</span>

        <span class="nf">Write</span><span class="p">(</span><span class="n">form</span><span class="p">.</span><span class="nf">Response</span><span class="p">());</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The Controller class above does the following in order:</p>

<ol>
<li>Gets all the values ​​from the <strong>POST request</strong> (the form sent from the client) with <code>context.Request.Form</code>.</li>
<li>Stores the value of the <strong>checksum parameter</strong> separately.</li>
<li>Converts the entire form to a string like <code>"key1=value1&amp;key2=value2&amp;..."</code>.</li>
<li>Using <strong>Regex</strong>, removes the <code>checksum</code> parameter from this string.</li>
<li>Calls the <code>ChecksumCalculation()</code> method to calculate the actual checksum of the data.</li>
<li>If the calculated checksum is the same as the value sent, it outputs the message <code>"Checksum is the same"</code>.</li>
<li>Finally, it returns the output with <code>Write(form.Response())</code>.</li>
</ol>

<blockquote>
<p>Note: On the server, you must remove the "&amp;checksum=value" value and then compare the checksum result.</p>
</blockquote>

<h3>
  
  
  Verifying the previous state of the client
</h3>

<p>By using the new Hash or Checksum properties, the server can now determine whether the previous Action Control has been executed or not.</p>

<p><strong>How ​​it works:</strong></p>

<ul>
<li>Call the SetHash or SetCheckSum functions in the WebForms class on the server
(These commands calculate the Hash and CheckSum for the Action Controls in WebFormsJS and then add them to a global Hash array.)</li>
<li>Use the HasHash method in the Fetch auxiliary class on the server
(By calling the condition section, the existence of the hash values ​​is checked.)</li>
</ul>

<blockquote>
<p>Note: The "ChecksumCalculation" function is available in the WebForms class on the server, but you can only obtain SHA256 through the server programming language functions.</p>
</blockquote>

<ul>
<li><code>form.SetHash()</code></li>
<li><code>form.SetChecksum()</code></li>
</ul>

<p>Example:</p>

<p>Send data and save static Hash value<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">static</span> <span class="kt">string</span> <span class="n">StaticHashValue</span><span class="p">;</span>
<span class="p">...</span>
<span class="n">WebForms</span> <span class="n">form</span> <span class="p">=</span> <span class="k">new</span> <span class="nf">WebForms</span><span class="p">();</span>

<span class="n">form</span><span class="p">.</span><span class="nf">AddTag</span><span class="p">(</span><span class="s">"&lt;body&gt;"</span><span class="p">,</span> <span class="s">"h3"</span><span class="p">);</span>
<span class="n">form</span><span class="p">.</span><span class="nf">SetBackgroundColor</span><span class="p">(</span><span class="s">"&lt;h3&gt;"</span><span class="p">,</span> <span class="s">"blue"</span><span class="p">);</span>
<span class="n">form</span><span class="p">.</span><span class="nf">SetHash</span><span class="p">();</span>
<span class="n">StaticHashValue</span> <span class="p">=</span> <span class="nf">SHA256</span><span class="p">(</span><span class="n">form</span><span class="p">.</span><span class="nf">GetWebFormsData</span><span class="p">());</span>
</code></pre>

</div>



<p>We can also use Checksum instead of Hash.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="p">...</span>
<span class="n">form</span><span class="p">.</span><span class="nf">SetChecksum</span><span class="p">();</span>
<span class="n">StaticHashValue</span> <span class="p">=</span> <span class="n">form</span><span class="p">.</span><span class="nf">ChecksumCalculation</span><span class="p">(</span><span class="n">form</span><span class="p">.</span><span class="nf">GetWebFormsData</span><span class="p">());</span>
<span class="c1">// or StaticHashValue = form.GetChecksum();</span>
</code></pre>

</div>



<ul>
<li>First the form data is retrieved (<code>form.GetWebFormsData()</code>).</li>
<li>Then a Hash (SHA256) or Checksum is calculated and stored in a variable (<code>StaticHashValue</code>).</li>
<li>This value will be used later to check for changes.</li>
</ul>

<p>Check Hash similarity<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">IsTrue</span><span class="p">(</span><span class="n">Fetch</span><span class="p">.</span><span class="nf">HasHash</span><span class="p">(</span><span class="n">StaticHashValue</span><span class="p">));</span>
<span class="n">form</span><span class="p">.</span><span class="nf">Message</span><span class="p">(</span><span class="s">"Hash value is the same"</span><span class="p">);</span>
</code></pre>

</div>



<ul>
<li>
<code>Fetch.HasHash(StaticHashValue)</code> checks whether this Hash has already been registered.</li>
<li>If it is <code>true</code>, the data has not changed and the Action Control has already been executed.</li>
<li>Then an appropriate message can be displayed (<code>"Hash value is the same"</code>).</li>
</ul>

<h3>
  
  
  When Checksums Are Useful
</h3>

<ul>
<li>
<strong>Data Integrity:</strong> Ensures data received by the server matches what was sent by the client.</li>
<li>
<strong>Form Change Detection:</strong> Helps process only modified data.</li>
<li>
<strong>Duplicate Submission Prevention:</strong> Detects repeated submissions from user actions or browser behavior.</li>
<li>
<strong>Basic Security Layer:</strong> Provides lightweight protection against simple tampering or unauthorized changes.</li>
</ul>

<h3>
  
  
  When Checksums May Be Unnecessary
</h3>

<ul>
<li>HTTPS already secures data transmission.</li>
<li>Very simple forms with minimal fields.</li>
<li>No server-side validation or logic applied.</li>
</ul>

<h3>
  
  
  Rare Edge Cases to Consider
</h3>

<ul>
<li>Mismatch between UTF-8 (server) and UTF-16 (JavaScript <code>charCodeAt</code>) encoding.</li>
<li>Text containing characters outside the Basic Multilingual Plane (e.g., emojis) may be decoded inconsistently.</li>
</ul>




<h2>
  
  
  Added ability to add cache and session directly
</h2>

<p>In version 2, four new methods have been added to the WebForms class on the server that allow cache management:</p>

<ul>
<li><code>AddCacheValue(Value, CacheKey)</code></li>
<li><code>InsertCacheValue(Value, CacheKey)</code></li>
<li><code>AddSessionCacheValue(Value, CacheKey)</code></li>
<li><code>InsertSessionCacheValue(Value, CacheKey)</code></li>
</ul>

<p>Example<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">AddCacheValue</span><span class="p">(</span><span class="n">Fetch</span><span class="p">.</span><span class="nf">LoadUrl</span><span class="p">(</span><span class="s">"/api/category-list"</span><span class="p">),</span> <span class="s">"category-list"</span><span class="p">);</span>
</code></pre>

</div>



<p>The above code stores the list of categories in the key "category-list".</p>

<p>Using<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">form</span><span class="p">.</span><span class="nf">AddText</span><span class="p">(</span><span class="s">"&lt;p&gt;2"</span><span class="p">,</span> <span class="n">Fetch</span><span class="p">.</span><span class="nf">Cache</span><span class="p">(</span><span class="s">"category-list"</span><span class="p">));</span>
</code></pre>

</div>



<p>The above code shows how to use Cache.</p>

<blockquote>
<p>Note: If you store data in "SessionCache", you need to use the "Form.Saved" method to access its values.</p>
</blockquote>

<p>Difference between <strong>Add</strong> and <strong>Insert</strong> in WebForms Core technology:</p>

<ul>
<li>If the key does not already exist, both insert new data.</li>
<li>If the key already exists, the word "Add" at the beginning of the methods causes the data to be added, but the word "Insert" does not insert new data.</li>
</ul>




<h3>
  
  
  Feature Objectives
</h3>

<ul>
<li>
<strong>Detect Execution:</strong> Identify whether a form action was performed.</li>
<li>
<strong>Validate Changes:</strong> Skip processing if no form changes occurred.</li>
<li>
<strong>Prevent Redundancy:</strong> Avoid handling duplicate submissions.</li>
<li>
<strong>Ensure Integrity:</strong> Confirm data consistency between client and server.</li>
<li>
<strong>Enhance Security:</strong> Provide basic protection against tampering.</li>
</ul>

<p>Together, these features enable smarter, safer, and more efficient form handling in <a href="https://github.com/webforms-core" rel="noopener noreferrer">WebForms Core</a>.</p>

