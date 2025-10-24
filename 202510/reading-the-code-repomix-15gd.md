---
Title: Reading the code: Repomix
Description: 
Author: Kyle Homen
Date: 2025-10-24T21:51:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>After spending some time looking at the [Repomix] tool and looking for features to implement in my own [project], I found that there was a checkbox for removing comments from the final output. This seems like a useful feature to have, so I got to reading the code to find out how it was implemented. I decided to use AI to give me an overview of the code, then dived deeper on my own.</p>

<h2>
  
  
  What does the feature do?
</h2>

<p>The comment removal feature strips comments from source code files when packing a repository. For example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// This is a comment
function hello() {
  /* Another comment */
  return "Hello!";
}
</code></pre>

</div>



<p>becomes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function hello() {
  return "Hello!";
}
</code></pre>

</div>



<p>This reduces the token count when feeding code into LLMs, so it's definitely a useful feature to implement.</p>

<h2>
  
  
  How it works
</h2>

<p><code>removeComments</code> is first defined in <a href="https://github.com/yamadashy/repomix/blob/main/src/config/configSchema.ts" rel="noopener noreferrer"><code>configSchema.ts</code></a> as a boolean.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">removeComments</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">boolean</span><span class="p">().</span><span class="nf">optional</span><span class="p">(),</span>
</code></pre>

</div>



<p>The config gets merged in <a href="https://github.com/yamadashy/repomix/blob/main/src/config/configLoad.ts" rel="noopener noreferrer"><code>configLoad.ts</code></a>, where it checks for arguments and overwrites by precedence. This is similar to how I implemented the TOML config in a previous lab for one of my lab partners. CLI argument (<code>--remove-comments</code>) overwrites the config (<code>repomix.config.json</code>), which overwrites the default value.</p>

<p>I managed to stumble upon the <a href="https://deepwiki.com/yamadashy/repomix" rel="noopener noreferrer">project wiki</a>, which gives a lot of valuable insight, such as the CLI's core components:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9l0hgamdhxh35h1rmlq2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9l0hgamdhxh35h1rmlq2.png" alt="Core compononents" width="698" height="388"></a></p>

<p>Here it mentions how the CLI uses Commander.js for argument parsing, and supports over 30 command-line options.</p>

<p>Also, they seem to use worker-based processing via <a href="https://github.com/tinylibs/tinypool" rel="noopener noreferrer"><code>Tinypool</code></a> to enable parallel processing. I'm not too familiar with parallel processing so I don't fully understand this, but that's a neat bit of optimization even for something that is seemingly fast enough.</p>

<p>Moving on, I found that files are processed in <a href="https://github.com/yamadashy/repomix/blob/main/src/core/file/fileProcessContent.ts" rel="noopener noreferrer"><code>fileProcessContent.ts</code></a>, which is where it checks if comments are to be removed and calls the manipulator.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>  <span class="k">if </span><span class="p">(</span><span class="nx">manipulator</span> <span class="o">&amp;&amp;</span> <span class="nx">config</span><span class="p">.</span><span class="nx">output</span><span class="p">.</span><span class="nx">removeComments</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">processedContent</span> <span class="o">=</span> <span class="nx">manipulator</span><span class="p">.</span><span class="nf">removeComments</span><span class="p">(</span><span class="nx">processedContent</span><span class="p">);</span>
  <span class="p">}</span>
</code></pre>

</div>



<p>And it is here, in <a href="https://github.com/yamadashy/repomix/blob/main/src/core/file/fileManipulate.ts" rel="noopener noreferrer"><code>fileManipulate.ts</code></a>, where the various types of manipulators exist for each different file type.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>  <span class="nf">removeComments</span><span class="p">(</span><span class="nx">content</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">removeDocStrings</span><span class="p">(</span><span class="nx">content</span><span class="p">);</span>
    <span class="nx">result</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">removeHashComments</span><span class="p">(</span><span class="nx">result</span><span class="p">);</span>
    <span class="k">return</span> <span class="nf">rtrimLines</span><span class="p">(</span><span class="nx">result</span><span class="p">);</span>
  <span class="p">}</span>
</code></pre>

</div>



<p>Where, each manipulator defines it's own way to remove comments and doc strings.</p>

