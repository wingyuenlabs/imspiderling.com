---
Title: The Secret Life of Python: The Pickle Jar
Description: 
Author: Aaron Rose
Date: 2026-02-28T21:45:01.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Why 'Cannot Pickle' happens: The limits of Python serialization</em></p>




<p>🎧 Audio Edition: Prefer to listen? Check out the expanded AI podcast version of this deep dive on <a href="https://youtu.be/1NACztyvRRE" rel="noopener noreferrer">YouTube</a>.</p>

<p>📺 Video Edition: Prefer to watch? Check out the 7-minute visual explainer on <a href="https://youtu.be/PTpGJapgaJ4" rel="noopener noreferrer">YouTube</a>.</p>




<p>Timothy was looking at the <code>TypeError</code> from his previous attempt to copy a database connection. "Margaret, I understand now why I can't copy a live connection, but I'm curious about the error message itself. Why did Python say it couldn't 'pickle' the object? What does that actually mean?"</p>

<p>Margaret walked over to the whiteboard and drew a simple diagram of a Python object—a dictionary containing a string and an integer. "It's a great question, Timothy. 'Pickling' is the term Python uses for <strong>Serialization</strong>."</p>

<h2>
  
  
  The Byte Stream
</h2>

<p>"When you want to save an object to a file, send it over a network, or even make a <code>deepcopy</code>, Python has to translate that complex object into a flat series of ones and zeros," Margaret explained kindly. "We call this a <strong>Byte Stream</strong>."</p>

<p>She drew an arrow from the object to a long line of hex digits: <code>\x80\x04\x95\x11...</code></p>

<p>"The <code>pickle</code> module looks at your object and creates a set of instructions," Margaret continued. "It records the type of the object, its attributes, and the data it holds. When you want the object back, Python reads those instructions and 'unpickles' it—reconstructing a brand-new object in memory that looks exactly like the original."</p>

<h2>
  
  
  The OS Boundary
</h2>

<p>Timothy pointed to the <code>_thread.lock</code> error on his screen. "So when I tried to <code>deepcopy</code> the database connection, <code>pickle</code> was trying to turn the connection into these instructions?"</p>

<p>"Exactly," Margaret nodded. "But <code>pickle</code> can only record data that lives entirely inside Python's memory. A database connection involves a 'file descriptor'—a specific number assigned by the Operating System (OS) to a live network socket. That number only means something to the OS while the connection is active."</p>

<p>She drew a brick wall on the board between "Python Memory" and the "Operating System."</p>

<p>"If Python tried to 'pickle' that socket number and save it to a file, and then you tried to 'unpickle' it an hour later, that number would be meaningless. The OS might have given that socket to a completely different program by then. Because <code>pickle</code> can't guarantee a working result across that boundary, it refuses to even try. That's why you saw the error."</p>

<h2>
  
  
  Seeing the Bytes
</h2>

<p>"Can I see what an object looks like when it's pickled?" Timothy asked.</p>

<p>"You certainly can," Margaret said. She wrote a small snippet on the board:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pickle</span>

<span class="n">data</span> <span class="o">=</span> <span class="p">{</span><span class="sh">"</span><span class="s">matches</span><span class="sh">"</span><span class="p">:</span> <span class="mi">10</span><span class="p">,</span> <span class="sh">"</span><span class="s">active</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span><span class="p">}</span>
<span class="n">pickled_data</span> <span class="o">=</span> <span class="n">pickle</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="n">pickled_data</span><span class="p">)</span>

</code></pre>

</div>



<p>"That <code>pickled_data</code> is the raw 'blueprint' of your data. You can save it or send it, and as long as the other side has Python, they can turn it back into a dictionary using <code>pickle.loads()</code>."</p>

<p>"Is <code>pickle</code> the only way to do this?" Timothy asked.</p>

<p>"Not at all," Margaret replied. "Pickle is very powerful because it handles almost any Python object, but it's Python-specific. If you wanted to send this data to a program written in JavaScript or Go, you would use a format like <strong>JSON</strong>. JSON is a 'universal language' for data, though it can only handle basic types like strings, numbers, and lists."</p>

<h2>
  
  
  A Word of Caution
</h2>

<p>Margaret's tone became a bit more serious. "There is one golden rule with pickling, Timothy: <strong>Never unpickle data from a source you don't trust.</strong> Because <code>pickle</code> reconstructs objects by following instructions, a malicious person could craft a 'pickle' that tells Python to execute harmful code during the unpickling process. It’s a powerful tool, but it requires a secure environment."</p>




<h2>
  
  
  Margaret’s Cheat Sheet: Serialization (Pickling)
</h2>

<ul>
<li>
<strong>The Process:</strong> Pickling (Serialization) converts a Python object into a byte stream. Unpickling (Deserialization) turns it back into an object.</li>
<li>
<strong>The Boundary:</strong> You cannot pickle objects tied to the Operating System, like file handles, network sockets, or database connections.</li>
<li>
<strong>The Alternatives:</strong> Use <strong>Pickle</strong> for Python-to-Python storage; use <strong>JSON</strong> for cross-language APIs.</li>
<li>
<strong>The Security Rule:</strong> Only <code>pickle.loads()</code> data you created yourself or received from a verified, secure source.</li>
<li>
<strong>Advanced Tip:</strong> For custom control, objects can define <code>__reduce__</code> to tell Python exactly how they should be serialized.</li>
</ul>




<p><em>Aaron Rose is a software engineer and technology writer at <a href="https://www.tech-reader.blog" rel="noopener noreferrer">tech-reader.blog</a>. For explainer videos and podcasts, check out <a href="https://www.youtube.com/@tech-reader2007" rel="noopener noreferrer">Tech-Reader YouTube channel</a>.</em></p>

