---
Title: The Pitfalls of Reading User Input in C: a Story About scanf and Stdin
Description: 
Author: Dmitrii Doroshev
Date: 2025-12-11T21:40:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>I recently had to write a piece of C code that takes some input from <code>stdin</code>, ignores the newline, discards whatever exceeds the buffer, and does this repeatedly in a loop.</p>

<p>Knowing something about <code>scanf</code> syntax (<a href="https://man7.org/linux/man-pages/man3/fscanf.3p.html" rel="noopener noreferrer">man</a>) I came up with this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cp">#include</span> <span class="cpf">&lt;stdio.h&gt;</span><span class="cp">
</span>
<span class="kt">void</span> <span class="nf">take_input</span><span class="p">(</span><span class="kt">void</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">char</span> <span class="n">buf</span><span class="p">[</span><span class="mi">20</span><span class="p">];</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">"&gt; "</span><span class="p">);</span>
    <span class="n">scanf</span><span class="p">(</span><span class="s">"%19[^</span><span class="se">\n</span><span class="s">]"</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">"input=`%s`</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
<span class="p">}</span>

<span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">void</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">for</span> <span class="p">(</span><span class="kt">int</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="mi">5</span><span class="p">;</span> <span class="n">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">take_input</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p><em>(Note: The original code used an infinite loop, but a simple <code>for</code> is enough to demonstrate the behavior.)</em></p>

<p>When I ran it, the result was surprising:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ gcc -o main main.c
$ ./main
&gt; hello world↵
input=`hello world`
&gt; input=`hello world`
&gt; input=`hello world`
&gt; input=`hello world`
&gt; input=`hello world`
$ █
</code></pre>

</div>



<p>It consumed the string once and printed the same value 5 times. Why?</p>

<h2>
  
  
  1. The Stack (Ghost Data)
</h2>

<p>Stack behavior explains the first confusing part. Although <code>char buf[20]</code> is a locally scoped variable, in this particular case, each invocation of <code>take_input()</code> ends up using the exact same stack memory address.</p>

<p>Printint the address makes this undeniable:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cm">/* ... */</span>
<span class="n">printf</span><span class="p">(</span><span class="s">"address=%p, input=`%s`</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="p">(</span><span class="kt">void</span><span class="o">*</span><span class="p">)</span><span class="n">buf</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
<span class="cm">/* ... */</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ ./main
&gt; hello world
address=0x7ffcb96c49d0, input=`hello world`
&gt; address=0x7ffcb96c49d0, input=`hello world`
&gt; address=0x7ffcb96c49d0, input=`hello world`
&lt;...&gt;
</code></pre>

</div>



<p>Because the buffer wasn't initialized, and because subsequent <code>scanf</code> calls failed to overwrite it, the old content (the "ghost data") remained unchanged. Initializing the buffer with zeroes fixes the specific issue:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cm">/* ... */</span>
<span class="kt">char</span> <span class="n">buf</span><span class="p">[</span><span class="mi">20</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="mi">0</span><span class="p">};</span>
<span class="cm">/* ... */</span>
</code></pre>

</div>



<p>Now the buffer resets each time, but <code>scanf</code> still behaves oddly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ ./main
&gt; hello world
address=0x7ffea5c2d840, input=`hello world`
&gt; address=0x7ffea5c2d840, input=``
&gt; address=0x7ffea5c2d840, input=``
&lt;...&gt;
</code></pre>

</div>



<h2>
  
  
  2. Scanf, Scansets and Stdin
</h2>

<p>The real enemy here is the scanset <code>%19[^\n]</code>. It reads up to 19 characters (<code>len(buf) - 1</code>) that are <em>not</em> a newline, but it <strong>does not consume the newline itself</strong>.</p>

<p>Let's open <code>gdb</code> and inspect <code>stdin</code> to see the wreckage:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ gcc -g3 -O0 main.c -o main
$ gdb main
(gdb) break 8
Breakpoint 1 at 0x11d3: file main.c, line 8.
(gdb) run
&lt;...&gt;
&gt; hello world↵
&lt;...&gt;
(gdb) p *stdin
$1 = {
    _flags = -72539512,
    _IO_read_ptr = 0x55555555972b "\n",
    _IO_read_end = 0x55555555972c "",
    _IO_read_base = 0x555555559720 "hello world\n",
    &lt;...&gt;
}
</code></pre>

</div>



<p><code>stdin-&gt;_IO_read_ptr</code>, the current position of input, still points to the newline. It was not consumed by <code>scanf</code>; it is still sitting in <code>stdin</code>, waiting to be digested.</p>

<p>When the loop runs again, <code>scanf("%19[^\n]", buf)</code> sees the newline immediately, matches zero characters, and aborts. The buffer stays empty (or zeroed), and the loop spins.</p>

<p>Unlike Python's expression <code>input()[:19]</code>, which consumes the newline and truncates the string cleanly, C's <code>scanf</code> cannot express that behavior using a single scanset. I found three workable solutions.</p>

<h3>
  
  
  Solution A: The "Double Tap" (Extra <code>scanf</code>)
</h3>

<p>We can use extra <code>scanf</code> calls to force the consumption of the remainder of the line and the newline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="kt">void</span> <span class="nf">take_input</span><span class="p">(</span><span class="kt">void</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">char</span> <span class="n">buf</span><span class="p">[</span><span class="mi">20</span><span class="p">];</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">"&gt; "</span><span class="p">);</span>
    <span class="n">scanf</span><span class="p">(</span><span class="s">"%19[^</span><span class="se">\n</span><span class="s">]"</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
    <span class="cm">/* discard the rest of line, if it's &gt;19 chars */</span>
    <span class="n">scanf</span><span class="p">(</span><span class="s">"%*[^</span><span class="se">\n</span><span class="s">]"</span><span class="p">);</span>
    <span class="cm">/* discard the newline */</span>
    <span class="n">scanf</span><span class="p">(</span><span class="s">"%*c"</span><span class="p">);</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">"input=`%s`</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It looks messy, but it works reliably for whitespace, short inputs, and long truncated inputs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ ./main
&gt; hey
input=`hey`
&gt; hello world
input=`hello world`
&gt; a b c d e f g h i j k l m o p q r s t u v w x y z
input=`a b c d e f g h i j`
&gt; ␣␣␣␣␣    /* 5 whitespaces */
input=`     `
&gt; abcdefghijklmnpoqrstuvwxyz
input=`abcdefghijklmnpoqrs`
</code></pre>

</div>



<h3>
  
  
  Solution B: <code>fgets</code> (The Standard Way)
</h3>

<p>A more predictable and explicit approach is <code>fgets</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cp">#include</span> <span class="cpf">&lt;string.h&gt;</span><span class="cp">
</span>
<span class="kt">void</span> <span class="nf">take_input_2</span><span class="p">(</span><span class="kt">void</span><span class="p">)</span>
<span class="p">{</span>
    <span class="kt">char</span> <span class="n">buf</span><span class="p">[</span><span class="mi">20</span><span class="p">];</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">"&gt; "</span><span class="p">);</span>

    <span class="k">if</span> <span class="p">(</span><span class="n">fgets</span><span class="p">(</span><span class="n">buf</span><span class="p">,</span> <span class="k">sizeof</span><span class="p">(</span><span class="n">buf</span><span class="p">),</span> <span class="n">stdin</span><span class="p">))</span> <span class="p">{</span>
        <span class="kt">char</span> <span class="o">*</span><span class="n">newline_ptr</span> <span class="o">=</span> <span class="n">strchr</span><span class="p">(</span><span class="n">buf</span><span class="p">,</span> <span class="sc">'\n'</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">newline_ptr</span><span class="p">)</span> <span class="p">{</span>
            <span class="cm">/* replace \n with \0 to trim it */</span>
            <span class="o">*</span><span class="n">newline_ptr</span> <span class="o">=</span> <span class="sc">'\0'</span><span class="p">;</span>
        <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
            <span class="cm">/* no newline found, meaning the input
               was truncated; we must consume
               the rest of the line from stdin */</span>
            <span class="kt">int</span> <span class="n">c</span><span class="p">;</span>
            <span class="k">while</span> <span class="p">((</span><span class="n">c</span> <span class="o">=</span> <span class="n">getchar</span><span class="p">())</span> <span class="o">!=</span> <span class="sc">'\n'</span> <span class="o">&amp;&amp;</span> <span class="n">c</span> <span class="o">!=</span> <span class="n">EOF</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
    <span class="n">printf</span><span class="p">(</span><span class="s">"input=`%s`</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">buf</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Solution C: <code>getline</code> (The Heap Way)
</h3>

<p>There is a handy <code>getline</code> function (<a href="https://man7.org/linux/man-pages/man3/getline.3.html" rel="noopener noreferrer">man</a>), which is not in ISO C but exists in POSIX. The difference is that <code>getline</code> scans the whole line and allocates the buffer on the heap. You don't have to care about buffer boundaries anymore, but you do need to manually <code>free</code> the memory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight c"><code><span class="cp">#include</span> <span class="cpf">&lt;stdio.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;string.h&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;stdlib.h&gt;</span><span class="cp">
</span>
<span class="kt">void</span> <span class="nf">take_input_3</span><span class="p">(</span><span class="kt">void</span><span class="p">)</span>
<span class="p">{</span>
    <span class="cm">/* initialize the pointer with NULL */</span>
    <span class="kt">char</span> <span class="o">*</span><span class="n">line</span> <span class="o">=</span> <span class="nb">NULL</span><span class="p">;</span>
    <span class="cm">/* getline will return the updated capacity of `line`
       - in practice it grows dynamically */</span>
    <span class="kt">size_t</span> <span class="n">cap</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
    <span class="kt">ssize_t</span> <span class="n">n</span><span class="p">;</span>

    <span class="n">printf</span><span class="p">(</span><span class="s">"&gt; "</span><span class="p">);</span>

    <span class="cm">/* n contains the actual length of the input line,
       or -1 on failure or EOF */</span>
    <span class="n">n</span> <span class="o">=</span> <span class="n">getline</span><span class="p">(</span><span class="o">&amp;</span><span class="n">line</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">cap</span><span class="p">,</span> <span class="n">stdin</span><span class="p">);</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">n</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
        <span class="cm">/* remove newline */</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">line</span><span class="p">[</span><span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span> <span class="o">==</span> <span class="sc">'\n'</span><span class="p">)</span> <span class="p">{</span>
            <span class="n">line</span><span class="p">[</span><span class="n">n</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span> <span class="o">=</span> <span class="sc">'\0'</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="n">printf</span><span class="p">(</span><span class="s">"input=`%s`</span><span class="se">\n</span><span class="s">"</span><span class="p">,</span> <span class="n">line</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="n">free</span><span class="p">(</span><span class="n">line</span><span class="p">);</span>  <span class="cm">/* NB: always free the heap memory */</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Outcomes
</h2>

<p>In the end, the exercise became a small reminder of how surprisingly tricky input handling in C can be. <code>scanf</code> looks convenient until you hit edge cases around whitespace and newlines. At that point, "convenience" becomes a liability.</p>

<p>The takeaway is simple: know your tools. If you need predictable, portable, line-oriented input with truncation handling, <code>fgets</code> is almost always the better choice. If you have the luxury of POSIX and heap allocation, <code>getline</code> offers freedom. But <code>scanf</code>? Save that for when you know exactly what the input looks like.</p>

