---
Title: Why I Built a Pure Python Library for Legacy Office Files (And Why RAG Pipelines Need One)
Description: 
Author: Tobias Horsmann
Date: 2025-12-25T21:30:17.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Why I Built a Pure Python Library for Legacy Office Files (And Why RAG Pipelines Need One)
</h1>

<p>If you're building RAG pipelines or document ingestion for LLM agents, you've probably solved the easy part already. Modern Office files? No problem. <code>python-docx</code>, <code>openpyxl</code>, <code>python-pptx</code> — pick your library, extract your text, move on.</p>

<p>Then someone points your pipeline at an enterprise SharePoint.</p>

<h2>
  
  
  The Legacy File Problem
</h2>

<p>Enterprise SharePoints are digital archaeology sites. Marketing uploaded PowerPoints in 2008. Legal has Word documents from 2005. Finance runs on Excel files that predate most of your team's careers.</p>

<p>These aren't edge cases. In my experience, legacy <code>.doc</code>, <code>.xls</code>, and <code>.ppt</code> files make up a significant chunk of any long-running enterprise document store. And if you're building a system that needs to ingest "all the documents," you can't just skip them.</p>

<h2>
  
  
  Why Existing Solutions Didn't Work for Me
</h2>

<p>I needed to process these files in AWS Lambda functions for a RAG pipeline. My options were:</p>

<p><strong>LibreOffice</strong></p>

<p>The standard answer. Install LibreOffice, run it headless, convert files to text. It works, but it adds over 1GB to your container image. Lambda has a 250MB limit for deployment packages (10GB with container images, but still). Plus, configuring headless LibreOffice is its own adventure.</p>

<p><strong>Apache Tika</strong></p>

<p>Solid tool, widely used. But it requires a Java runtime and typically runs as a separate server. That's another service to deploy, monitor, and secure. For a document extraction step in a pipeline, it felt like overkill.</p>

<p><strong>Subprocess calls to command-line tools</strong></p>

<p>Various tools exist that you can shell out to. But subprocess calls are a security concern, they break in restricted environments, and they make your code platform-dependent.</p>

<p>I wanted something simpler: a Python library I could <code>pip install</code> and call.</p>

<h2>
  
  
  Building sharepoint-to-text
</h2>

<p>So I built <a href="https://github.com/Horsmann/sharepoint-to-text" rel="noopener noreferrer">sharepoint-to-text</a>.</p>

<p>The core idea: parse both legacy Office binary formats (OLE2) and modern XML-based formats (OOXML) directly in Python. No external dependencies. No subprocess calls. Just text extraction.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">sharepoint2text</span>

<span class="c1"># Works the same for legacy or modern files
</span><span class="n">result</span> <span class="o">=</span> <span class="n">sharepoint2text</span><span class="p">.</span><span class="nf">read_file</span><span class="p">(</span><span class="sh">"</span><span class="s">ancient_report.doc</span><span class="sh">"</span><span class="p">)</span>
<span class="n">result</span> <span class="o">=</span> <span class="n">sharepoint2text</span><span class="p">.</span><span class="nf">read_file</span><span class="p">(</span><span class="sh">"</span><span class="s">modern_report.docx</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p>It handles:</p>

<ul>
<li>Legacy formats: <code>.doc</code>, <code>.xls</code>, <code>.ppt</code>
</li>
<li>Modern formats: <code>.docx</code>, <code>.xlsx</code>, <code>.pptx</code>
</li>
<li>Plus <code>.pdf</code> and plain text</li>
</ul>

<p>One interface, no conditional logic, no format detection boilerplate in your code.</p>

<h2>
  
  
  Why This Matters for RAG and LLM Agents
</h2>

<p>If you're building document ingestion for RAG, you're probably dealing with heterogeneous input. Users upload files. Pipelines crawl document stores. You can't control what formats show up.</p>

<p>The typical approach is a cascade of if-statements and multiple libraries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># The ugly version
</span><span class="k">if</span> <span class="n">path</span><span class="p">.</span><span class="nf">endswith</span><span class="p">(</span><span class="sh">'</span><span class="s">.docx</span><span class="sh">'</span><span class="p">):</span>
    <span class="n">text</span> <span class="o">=</span> <span class="nf">extract_with_python_docx</span><span class="p">(</span><span class="n">path</span><span class="p">)</span>
<span class="k">elif</span> <span class="n">path</span><span class="p">.</span><span class="nf">endswith</span><span class="p">(</span><span class="sh">'</span><span class="s">.doc</span><span class="sh">'</span><span class="p">):</span>
    <span class="n">text</span> <span class="o">=</span> <span class="nf">extract_with_libreoffice</span><span class="p">(</span><span class="n">path</span><span class="p">)</span>  <span class="c1"># hope it's installed
</span><span class="k">elif</span> <span class="n">path</span><span class="p">.</span><span class="nf">endswith</span><span class="p">(</span><span class="sh">'</span><span class="s">.xlsx</span><span class="sh">'</span><span class="p">):</span>
    <span class="n">text</span> <span class="o">=</span> <span class="nf">extract_with_openpyxl</span><span class="p">(</span><span class="n">path</span><span class="p">)</span>
<span class="c1"># ... and so on
</span></code></pre>

</div>



<p>With sharepoint-to-text, it's just:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">sharepoint2text</span>

<span class="n">result</span> <span class="o">=</span> <span class="n">sharepoint2text</span><span class="p">.</span><span class="nf">read_file</span><span class="p">(</span><span class="n">path</span><span class="p">)</span>
</code></pre>

</div>



<p>The library figures out the format and handles it appropriately.</p>

<h2>
  
  
  Deployment Benefits
</h2>

<p>Because it's pure Python with no system dependencies:</p>

<ul>
<li>
<strong>Container images stay small</strong> — no LibreOffice bloat</li>
<li>
<strong>Serverless-friendly</strong> — works in Lambda, Cloud Functions, Azure Functions</li>
<li>
<strong>No security concerns</strong> — no subprocess calls, no shell execution</li>
<li>
<strong>Cross-platform</strong> — Windows, macOS, Linux, whatever</li>
</ul>

<h2>
  
  
  When You Might Need This
</h2>

<ul>
<li>You're building RAG pipelines against enterprise document stores</li>
<li>Your LLM agent needs to process user-uploaded files of unknown vintage</li>
<li>You're deploying to serverless with size constraints</li>
<li>Your security team doesn't allow subprocess execution</li>
<li>You're tired of maintaining LibreOffice in containers</li>
</ul>

<h2>
  
  
  Try It Out
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>sharepoint-to-text
</code></pre>

</div>



<p>GitHub: <a href="https://github.com/Horsmann/sharepoint-to-text" rel="noopener noreferrer">https://github.com/Horsmann/sharepoint-to-text</a></p>

<p>I'd appreciate feedback, especially if you hit edge cases with specific file types. Legacy Office formats are notoriously inconsistent, and real-world files are the best test suite.</p>

