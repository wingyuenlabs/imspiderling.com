---
Title: How can I bind OLSRT to PHP?
Description: 
Author: Javad
Date: 2025-11-01T21:46:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey Dev Community!<br><br>
After my last blog about <a href="https://dev.to/javadinteger/how-can-i-bind-olsrt-to-nodejs-352m">binding OLSRT to Node.js</a>, I’m back again—this time with a new challenge: <strong>How can I bind OLSRT to PHP?</strong>  </p>

<p>If you’ve ever wanted to bring async/event‑driven power into PHP, this post is for you. Let’s dive in together!</p>


<h2>
  
  
  🔧 Prerequisites
</h2>

<p>Before we start, make sure you have:</p>

<ul>
<li>
<strong>Knowledge:</strong> C programming and basic PHP internals (Zend API).</li>
<li>
<strong>Tools:</strong>

<ul>
<li>On <strong>Windows</strong>: PHP SDK, Visual Studio (MSVC), Git.</li>
<li>On <strong>Linux/macOS</strong>: <code>php-dev</code>, <code>phpize</code>, GCC/Clang, Autotools, Git.</li>
</ul>
</li>
<li>
<strong>OLSRT repo cloned:</strong>
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  git clone https://github.com/OverLab-Group/OLSRT
</code></pre>

</div>



<h2>
  
  
  📂 Project Setup
</h2>

<p>Inside your project root, create:</p>

<ul>
<li>
<code>php_olsrt.h</code> — header file</li>
<li>
<code>php_olsrt.c</code> — main binding code</li>
<li>
<code>config.m4</code> (Linux/macOS) or <code>config.w32</code> (Windows) — build config</li>
<li>
<code>test.php</code> — to test the extension</li>
</ul>

<p>Also add include directories to compiler: <code>OLSRT/src</code>, <code>OLSRT/includes</code>, and Root Project.</p>


<h2>
  
  
  📝 php_olsrt.h
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cp">#ifndef PHP_OLSRT_H
#define PHP_OLSRT_H
</span>
<span class="k">extern</span> <span class="n">zend_module_entry</span> <span class="n">olsrt_module_entry</span><span class="p">;</span>
<span class="cp">#define phpext_olsrt_ptr &amp;olsrt_module_entry
</span>
<span class="cp">#define PHP_OLSRT_EXTNAME "olsrt"
#define PHP_OLSRT_VERSION "0.1.0"
</span>
<span class="n">PHP_FUNCTION</span><span class="p">(</span><span class="n">olsrt_hello</span><span class="p">);</span>
<span class="n">PHP_FUNCTION</span><span class="p">(</span><span class="n">olsrt_version</span><span class="p">);</span>

<span class="cp">#endif </span><span class="cm">/* PHP_OLSRT_H */</span><span class="cp">
</span></code></pre>

</div>



<h2>
  
  
  📝 php_olsrt.c
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cp">#include</span> <span class="cpf">"php.h"</span><span class="cp">
#include</span> <span class="cpf">"php_ini.h"</span><span class="cp">
#include</span> <span class="cpf">"ext/standard/info.h"</span><span class="cp">
#include</span> <span class="cpf">"php_olsrt.h"</span><span class="cp">
</span><span class="c1">// #include "olsrt.h" // Uncomment when OLSRT headers are ready</span>

<span class="n">PHP_FUNCTION</span><span class="p">(</span><span class="n">olsrt_hello</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">php_printf</span><span class="p">(</span><span class="s">"[OLSRT] Hello from PHP binding!</span><span class="se">\\</span><span class="s">n"</span><span class="p">);</span>
    <span class="n">RETURN_TRUE</span><span class="p">;</span>
<span class="p">}</span>

<span class="n">PHP_FUNCTION</span><span class="p">(</span><span class="n">olsrt_version</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">RETURN_STRING</span><span class="p">(</span><span class="s">"OLSRT PHP binding 0.1.0 (stub)"</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">static</span> <span class="k">const</span> <span class="n">zend_function_entry</span> <span class="n">olsrt_functions</span><span class="p">[]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="n">PHP_FE</span><span class="p">(</span><span class="n">olsrt_hello</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">)</span>
    <span class="n">PHP_FE</span><span class="p">(</span><span class="n">olsrt_version</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">)</span>
    <span class="n">PHP_FE_END</span>
<span class="p">};</span>

<span class="n">PHP_MINIT_FUNCTION</span><span class="p">(</span><span class="n">olsrt</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="n">SUCCESS</span><span class="p">;</span> <span class="p">}</span>
<span class="n">PHP_MSHUTDOWN_FUNCTION</span><span class="p">(</span><span class="n">olsrt</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="n">SUCCESS</span><span class="p">;</span> <span class="p">}</span>
<span class="n">PHP_RINIT_FUNCTION</span><span class="p">(</span><span class="n">olsrt</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="n">SUCCESS</span><span class="p">;</span> <span class="p">}</span>
<span class="n">PHP_RSHUTDOWN_FUNCTION</span><span class="p">(</span><span class="n">olsrt</span><span class="p">)</span> <span class="p">{</span> <span class="k">return</span> <span class="n">SUCCESS</span><span class="p">;</span> <span class="p">}</span>

<span class="n">PHP_MINFO_FUNCTION</span><span class="p">(</span><span class="n">olsrt</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">php_info_print_table_start</span><span class="p">();</span>
    <span class="n">php_info_print_table_header</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s">"OLSRT support"</span><span class="p">,</span> <span class="s">"enabled"</span><span class="p">);</span>
    <span class="n">php_info_print_table_row</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="s">"Version"</span><span class="p">,</span> <span class="n">PHP_OLSRT_VERSION</span><span class="p">);</span>
    <span class="n">php_info_print_table_end</span><span class="p">();</span>
<span class="p">}</span>

<span class="n">zend_module_entry</span> <span class="n">olsrt_module_entry</span> <span class="o">=</span> <span class="p">{</span>
    <span class="n">STANDARD_MODULE_HEADER</span><span class="p">,</span>
    <span class="n">PHP_OLSRT_EXTNAME</span><span class="p">,</span>
    <span class="n">olsrt_functions</span><span class="p">,</span>
    <span class="n">PHP_MINIT</span><span class="p">(</span><span class="n">olsrt</span><span class="p">),</span>
    <span class="n">PHP_MSHUTDOWN</span><span class="p">(</span><span class="n">olsrt</span><span class="p">),</span>
    <span class="n">PHP_RINIT</span><span class="p">(</span><span class="n">olsrt</span><span class="p">),</span>
    <span class="n">PHP_RSHUTDOWN</span><span class="p">(</span><span class="n">olsrt</span><span class="p">),</span>
    <span class="n">PHP_MINFO</span><span class="p">(</span><span class="n">olsrt</span><span class="p">),</span>
    <span class="n">PHP_OLSRT_VERSION</span><span class="p">,</span>
    <span class="n">STANDARD_MODULE_PROPERTIES</span>
<span class="p">};</span>

<span class="cp">#ifdef COMPILE_DL_OLSRT
</span><span class="n">ZEND_GET_MODULE</span><span class="p">(</span><span class="n">olsrt</span><span class="p">)</span>
<span class="cp">#endif
</span></code></pre>

</div>



<h2>
  
  
  ⚙️ config.m4 (Linux/macOS)
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PHP_ARG_ENABLE(olsrt, whether to enable OLSRT support,
[  --enable-olsrt   Enable OLSRT support])

if test "$PHP_OLSRT" != "no"; then
  PHP_NEW_EXTENSION(olsrt, php_olsrt.c, $ext_shared)
  PHP_ADD_INCLUDE(/absolute/path/to/OLSRT/includes)
fi
</code></pre>

</div>



<h2>
  
  
  ⚙️ config.w32 (Windows)
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nc">ARG_ENABLE</span><span class="p">(</span><span class="dl">"</span><span class="s2">olsrt</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Enable OLSRT support</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">no</span><span class="dl">"</span><span class="p">);</span>

<span class="k">if </span><span class="p">(</span><span class="nx">PHP_OLSRT</span> <span class="o">!=</span> <span class="dl">"</span><span class="s2">no</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">var</span> <span class="nx">OLSRT_INC</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">C:</span><span class="se">\\</span><span class="s2">path</span><span class="se">\\</span><span class="s2">to</span><span class="se">\\</span><span class="s2">OLSRT</span><span class="se">\\</span><span class="s2">includes</span><span class="dl">"</span><span class="p">;</span>
    <span class="nc">ADD_SOURCES</span><span class="p">(</span><span class="nx">configure_module_dirname</span><span class="p">,</span> <span class="dl">"</span><span class="s2">php_olsrt.c</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">olsrt</span><span class="dl">"</span><span class="p">);</span>
    <span class="nc">ADD_INCLUDE</span><span class="p">(</span><span class="nx">OLSRT_INC</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  ▶️ Build &amp; Install
</h2>
<h3>
  
  
  Linux/macOS
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>phpize
./configure <span class="nt">--enable-olsrt</span>
make
<span class="nb">sudo </span>make <span class="nb">install</span>
</code></pre>

</div>


<p>Add to <code>php.ini</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>extension=olsrt.so
</code></pre>

</div>



<h3>
  
  
  Windows
</h3>

<ul>
<li>Run <code>buildconf.bat</code>
</li>
<li><code>configure.js --enable-olsrt</code></li>
<li>Build with MSVC → output <code>php_olsrt.dll</code>
</li>
<li>Copy to <code>ext\</code> and add to <code>php.ini</code>:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  extension=php_olsrt.dll
</code></pre>

</div>






<h2>
  
  
  🧪 test.php
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="cp">&lt;?php</span>
<span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="nb">extension_loaded</span><span class="p">(</span><span class="s1">'olsrt'</span><span class="p">))</span> <span class="p">{</span>
    <span class="o">@</span><span class="nb">dl</span><span class="p">(</span><span class="s1">'olsrt.'</span> <span class="mf">.</span> <span class="p">(</span><span class="kc">PHP_SHLIB_SUFFIX</span> <span class="o">??</span> <span class="s1">'so'</span><span class="p">));</span>
<span class="p">}</span>

<span class="nf">olsrt_hello</span><span class="p">();</span>
<span class="k">echo</span> <span class="s2">"Version: "</span><span class="p">,</span> <span class="nf">olsrt_version</span><span class="p">(),</span> <span class="kc">PHP_EOL</span><span class="p">;</span>
</code></pre>

</div>



<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>php test.php
</code></pre>

</div>






<h2>
  
  
  🎯 Conclusion
</h2>

<p>And that’s it! You’ve just built a PHP extension that binds to OLSRT. Right now it’s a skeleton with <code>olsrt_hello</code> and <code>olsrt_version</code>, but the structure is ready for you to plug in <strong>Futures, Actors, Event Loop, Streams, and WebSockets</strong> from OLSRT.  </p>

<p>This is the foundation—next steps could be:</p>

<ul>
<li>Wrapping OLSRT Futures into PHP classes</li>
<li>Adding async/await‑like APIs</li>
<li>Building a real demo (e.g. chat server)</li>
</ul>




<h2>
  
  
  👋 Final words
</h2>

<p>Thanks for following along! I hope this guide helps you bring the async/event‑driven magic of OLSRT into PHP.<br><br>
If you try this out, let me know what you build—I’d love to see it.  </p>

<p>Until next time, have a great OLSRT day! 🚀</p>

