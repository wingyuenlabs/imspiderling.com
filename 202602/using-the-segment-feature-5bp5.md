---
Title: Using the Segment Feature
Description: 
Author: Elanat Framework
Date: 2026-02-27T21:28:20.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Segment</strong> is an attribute that can be applied to ASPX pages. When this feature is enabled, all path parts that appear after the aspx file path are treated as segments and routed to the same executing page. Segment is one of the innovative ideas of the <a href="https://elanat.net" rel="noopener noreferrer">Elanat</a> team. By enabling Segment, you gain full control over URL paths.</p>




<h2>
  
  
  Enabling Segment
</h2>

<h3>
  
  
  Razor Syntax
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight diff"><code>@page
<span class="gi">+@segment
</span>&lt;!DOCTYPE html&gt;
&lt;html&gt;
...
</code></pre>

</div>



<h3>
  
  
  Standard ASPX Syntax
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;</span><span class="err">%@</span> <span class="na">Page</span> <span class="na">Segment=</span><span class="s">"true"</span> <span class="err">%</span><span class="nt">&gt;</span>
<span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
...
</code></pre>

</div>






<h2>
  
  
  How Segment Works
</h2>

<p>If you enable Segment on the following path:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about.aspx
</code></pre>

</div>



<p>Any additional path parts after this address will be considered segments, and the executable file <code>/page/about.aspx</code> will still be executed.</p>

<h3>
  
  
  Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about.aspx/segment1/segment2/.../segmentN
</code></pre>

</div>



<p>If Segment is enabled on a file such as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about/Default.aspx
</code></pre>

</div>



<p>You can still access it using both of the following formats:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about/Default.aspx/segment1/segment2/.../segmentN
</code></pre>

</div>



<p>or<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about/segment1/segment2/.../segmentN
</code></pre>

</div>






<h2>
  
  
  Accessing Segment Values
</h2>

<p>You can access Segment values in all three layers: <strong>View, Controller, and Model</strong>.</p>

<p>Segments are zero-based indexed.</p>

<p>Given the URL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about/segment1/segment2/segment3
</code></pre>

</div>



<ul>
<li>
<code>Segment.GetValue(0)</code> returns <code>"segment1"</code>
</li>
<li>
<code>Segment.GetValue(1)</code> returns <code>"segment2"</code>
</li>
<li>
<code>Segment.GetValue(2)</code> returns <code>"segment3"</code>
</li>
</ul>

<p>In general:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Segment.GetValue(n)
</code></pre>

</div>



<p>returns the segment at index <code>n</code> (starting from 0).</p>




<h3>
  
  
  Example in View (Razor)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;b&gt;segment 1 is: @Segment.GetValue(0)&lt;/b&gt;
&lt;b&gt;segment 2 is: @Segment.GetValue(1)&lt;/b&gt;
</code></pre>

</div>



<h3>
  
  
  Example in View (Standard ASPX)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;b&gt;segment 1 is: &lt;%= Segment.GetValue(0) %&gt;&lt;/b&gt;
&lt;b&gt;segment 2 is: &lt;%= Segment.GetValue(1) %&gt;&lt;/b&gt;
</code></pre>

</div>



<h3>
  
  
  Example in Controller
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight diff"><code><span class="p">using CodeBehind;
</span><span class="err">
</span><span class="p">namespace YourProjectName
</span>{
    public partial class DefaultController : CodeBehindController
    {
        public void PageLoad(HttpContext context)
        {
<span class="gi">+           Write(Segment.GetValue(0));
+           Write(Segment.GetValue(1));
</span>        }
    }
}
</code></pre>

</div>



<p>By activating Segment, you no longer need to use query strings to pass parameters through the URL.</p>




<h2>
  
  
  Important Behavior
</h2>

<p>If you enable Segment on a specific path, ASPX view files located in its subdirectories will not be executed.</p>

<h3>
  
  
  Example
</h3>

<p>If Segment is enabled on:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about/Default.aspx
</code></pre>

</div>



<p>Then the following path will no longer be accessible:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/about/license/Default.aspx
</code></pre>

</div>






<h2>
  
  
  Checking Segment Existence
</h2>

<p>You can use the <code>Exist</code> method to check whether a segment value exists before accessing it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>@if (!Segment.Exist(0))
{
    <span class="nt">&lt;b&gt;</span>Value does not exist<span class="nt">&lt;/b&gt;</span>
}
</code></pre>

</div>






<h2>
  
  
  Segment in Default MVC Configuration
</h2>

<p>In the default MVC configuration, the Segment feature is enabled automatically in controller routing.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">CodeBehind</span><span class="p">;</span>

<span class="k">public</span> <span class="k">partial</span> <span class="k">class</span> <span class="nc">user</span> <span class="p">:</span> <span class="n">CodeBehindController</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">PageLoad</span><span class="p">(</span><span class="n">HttpContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>   
        <span class="nf">Write</span><span class="p">(</span><span class="n">Segment</span><span class="p">.</span><span class="nf">GetValue</span><span class="p">(</span><span class="m">0</span><span class="p">));</span> <span class="c1">// path: /user/{id}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Rewrite ASPX Path as Directory
</h2>

<p>One of the interesting ideas introduced by the Elanat team in the CodeBehind framework is the ability to rewrite an aspx file path as a directory name.</p>

<p>When this option is enabled, routes leading to an aspx file without the <code>.aspx</code> extension will be treated as directory paths.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[CodeBehind options]</span><span class="c">; do not change order
</span><span class="err">...</span>
<span class="py">rewrite_aspx_file_to_directory</span><span class="p">=</span><span class="s">true</span>
<span class="err">...</span>
</code></pre>

</div>



<p><strong>Note:</strong> You can safely enable this option because this rewrite does not introduce any additional processing load.</p>

<h3>
  
  
  Example
</h3>

<p>Access:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/contact.aspx
</code></pre>

</div>



<p>Using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/contact
</code></pre>

</div>



<p>And also:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/page/contact/segment1/segment2/.../segmentN
</code></pre>

</div>






<h2>
  
  
  Related links
</h2>

<p>CodeBehind on GitHub:<br>
<a href="https://github.com/elanatframework/Code_behind" rel="noopener noreferrer">https://github.com/elanatframework/Code_behind</a></p>

<p>CodeBehind in NuGet:<br>
<a href="https://www.nuget.org/packages/CodeBehind/" rel="noopener noreferrer">https://www.nuget.org/packages/CodeBehind/</a></p>

<p>CodeBehind page:<br>
<a href="https://elanat.net/page_content/code_behind" rel="noopener noreferrer">https://elanat.net/page_content/code_behind</a></p>

