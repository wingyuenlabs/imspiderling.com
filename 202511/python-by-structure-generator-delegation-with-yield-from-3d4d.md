---
Title: Python by Structure: Generator Delegation with Yield From
Description: 
Author: Aaron Rose
Date: 2025-11-16T21:10:32.000Z
Robots: noindex,nofollow
Template: index
---
<p>Timothy was studying a function that traversed a nested directory structure, and his forehead was creased with confusion. "Margaret, I'm trying to understand this code. It has <code>yield from</code> in it, and I can't figure out what it's doing differently than just <code>yield</code>."</p>

<p>Margaret looked over his shoulder at the screen. "Ah, generator delegation. This is one of Python's more elegant features, but it's subtle. Let me show you why it exists and what the structure reveals."</p>

<p>Timothy's code was attempting to flatten a nested list structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">flatten</span><span class="p">(</span><span class="n">items</span><span class="p">):</span>
    <span class="n">result</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">items</span><span class="p">:</span>
        <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">item</span><span class="p">,</span> <span class="nb">list</span><span class="p">):</span>
            <span class="k">for</span> <span class="n">sub_item</span> <span class="ow">in</span> <span class="nf">flatten</span><span class="p">(</span><span class="n">item</span><span class="p">):</span>
                <span class="n">result</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">sub_item</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="n">result</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">item</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">result</span>
</code></pre>

</div>



<p>"This works," Timothy said, "but I saw someone use <code>yield from</code> to do the same thing in fewer lines. I tried to rewrite it but I'm not sure I understand the structure."</p>

<h2>
  
  
  The Problem: Manual Iteration Over Generators
</h2>

<p>"Your version builds a list," Margaret observed. "That means you're storing everything in memory at once. Watch what happens when we use generators and <code>yield from</code> instead."</p>

<p>She rewrote the function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">flatten</span><span class="p">(</span><span class="n">items</span><span class="p">):</span>
    <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">items</span><span class="p">:</span>
        <span class="k">if</span> <span class="nf">isinstance</span><span class="p">(</span><span class="n">item</span><span class="p">,</span> <span class="nb">list</span><span class="p">):</span>
            <span class="k">yield</span> <span class="k">from</span> <span class="nf">flatten</span><span class="p">(</span><span class="n">item</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">yield</span> <span class="n">item</span>
</code></pre>

</div>



<p>"Wait," Timothy blinked. "That's... much shorter. But what's <code>yield from</code> actually doing?"</p>

<p>"Let me show you the structure."</p>

<p><strong>Tree View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>flatten(items)
    For item in items
        If isinstance(item, list)
        ├── (yield from flatten(item))
        └── Else
            (yield item)
</code></pre>

</div>



<p><strong>English View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Function flatten(items):
  For each item in items:
    If isinstance(item, list):
      Evaluate (yield from flatten(item)).
    else:
      Evaluate (yield item).
</code></pre>

</div>



<p>Timothy studied the structure carefully. "So <code>yield from</code> is... delegating to another generator?"</p>

<p>"Exactly," Margaret said. "When you write <code>yield from flatten(item)</code>, you're telling Python: 'I'm delegating iteration to this other generator. Whatever it yields, yield that directly to my caller.'"</p>

<h2>
  
  
  Understanding the Pattern
</h2>

<p>Margaret pulled out another example to make the concept clearer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">read_files</span><span class="p">(</span><span class="n">filenames</span><span class="p">):</span>
    <span class="k">for</span> <span class="n">filename</span> <span class="ow">in</span> <span class="n">filenames</span><span class="p">:</span>
        <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">filename</span><span class="p">,</span> <span class="sh">'</span><span class="s">r</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="k">yield</span> <span class="k">from</span> <span class="n">f</span>
</code></pre>

</div>



<p><strong>Tree View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>read_files(filenames)
    For filename in filenames
        With open(filename, 'r') as f
            (yield from f)
</code></pre>

</div>



<p><strong>English View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Function read_files(filenames):
  For each filename in filenames:
    With open(filename, 'r') as f:
      Evaluate (yield from f).
</code></pre>

</div>



<p>"See?" Margaret pointed at the structure. "The <code>yield from f</code> means 'delegate iteration to the file object. Each line it yields, I yield.'"</p>

<p>Timothy's eyes widened. "So instead of writing a loop that yields each line individually, <code>yield from</code> just... connects the two generators together?"</p>

<p>"That's a perfect way to think about it," Margaret confirmed. "It's generator plumbing. You're connecting one generator's output directly to another's input."</p>

<p>She showed him what the manual version would look like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">read_files_manual</span><span class="p">(</span><span class="n">filenames</span><span class="p">):</span>
    <span class="k">for</span> <span class="n">filename</span> <span class="ow">in</span> <span class="n">filenames</span><span class="p">:</span>
        <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">filename</span><span class="p">,</span> <span class="sh">'</span><span class="s">r</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">f</span><span class="p">:</span>
                <span class="k">yield</span> <span class="n">line</span>
</code></pre>

</div>



<p><strong>Tree View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>read_files_manual(filenames)
    For filename in filenames
        With open(filename, 'r') as f
            For line in f
                (yield line)
</code></pre>

</div>



<p><strong>English View:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Function read_files_manual(filenames):
  For each filename in filenames:
    With open(filename, 'r') as f:
      For each line in f:
        Evaluate (yield line).
</code></pre>

</div>



<p>"Now look at the difference in structure," Margaret said. "The manual version has three levels of nesting. The <code>yield from</code> version has two. When you're delegating to another generator, <code>yield from</code> expresses that delegation directly."</p>

<p>Timothy compared the two structures side by side. "So <code>yield from</code> is saying 'I trust this other generator to handle iteration. Just pass through whatever it yields.'"</p>

<p>"Precisely. And it's not just about fewer lines of code - it's about expressing intent. When you see <code>yield from</code>, you know immediately that this generator is delegating responsibility to another generator."</p>

<p>"Does it work with any iterable?" Timothy asked.</p>

<p>"Yes. You can <code>yield from</code> any iterable - lists, tuples, generators, files, even other objects that implement the iterator protocol. Python handles the delegation."</p>

<p>Timothy nodded slowly, then looked back at his nested list flattener. "So in my flatten function, when I hit a nested list, I'm recursively delegating to another flatten generator, and it yields all its items directly to my caller?"</p>

<p>"Exactly. The structure shows it clearly - you're not manually iterating and re-yielding. You're delegating the entire iteration to the recursive call."</p>

<p>Margaret leaned back. "Generator delegation is powerful because it lets you compose generators like building blocks. Each generator does one thing, and <code>yield from</code> connects them together structurally."</p>

<p>Timothy saved his refactored code, already thinking about other places where manual iteration loops could become clean delegation. "First I learned about <code>else</code> clauses I didn't know existed. Now I'm learning about delegation I didn't know existed. Python keeps surprising me."</p>

<p>"The structure is always there," Margaret smiled. "You just needed to learn how to see it."</p>




<p><strong>Explore Python structure yourself:</strong> Download the <a href="https://www.pacificw.com" rel="noopener noreferrer">Python Structure Viewer</a> - a free tool that shows code structure in tree and plain English views. Works offline, no installation required.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyabctbp423ti1q3bs4t1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyabctbp423ti1q3bs4t1.png" alt="Python Structure Viewer" width="800" height="500"></a></p>




<p><em>Aaron Rose is a software engineer and technology writer at <a href="https://www.tech-reader.blog" rel="noopener noreferrer">tech-reader.blog</a> and the author of <a href="https://amazon.com/author/aaron.rose" rel="noopener noreferrer">Think Like a Genius</a>.</em></p>

