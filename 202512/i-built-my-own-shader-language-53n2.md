---
Title: I built my own Shader Language
Description: 
Author: Jannik Brozy
Date: 2025-12-28T21:49:28.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been writing shaders for a while now, and i keep running into the same couple of issues. They aren't deal-breakers, but they are annoying enough that I started wondering if there was a better way to handle them.</p>

<p>The biggest one is Coordinate Spaces.</p>

<p>In standard GLSL, a <code>vec4</code> is just a <code>vec4</code>. The compiler doesn't care if that vector represents a position in World Space, Model Space, or Clip Space. If you accidentally multiply a View-Space-Vector by a Model-Matrix, the compiler won't stop you. You just get a black screen or weird lighting, and then you spend 20 minutes debugging only to realize you mixed up your spaces.</p>

<p>So, I decided to start a side project called <a href="//github.com/jbrozy/cast">Cast</a>.<br>
It's a small language that compiles down to GLSL, written in C# using ANTLR4. My goal wasn't to replace GLSL, but to create a "wrapper" that enforces some safety rules before generating the final code.</p>

<p><strong>The Main Idea:</strong><br>
I wanted the coordinate system to be a part of the type itself. Instead of just writing <code>vec4</code>, in Cast you write <code>vec4&lt;World&gt;</code> or <code>vec4&lt;Model&gt;</code>.</p>

<p>This allows the compiler to check the math.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">let</span> <span class="n">modelPos</span> <span class="p">:</span> <span class="n">vec4</span><span class="o">&lt;</span><span class="n">Model</span><span class="o">&gt;</span> <span class="o">=</span> <span class="o">...</span><span class="p">;</span>
<span class="k">let</span> <span class="n">matrix</span><span class="p">:</span> <span class="n">mat4</span><span class="o">&lt;</span><span class="n">Model</span><span class="p">,</span> <span class="n">World</span><span class="o">&gt;</span> <span class="o">=</span> <span class="o">...</span><span class="p">;</span>

<span class="c1">// This works because of the types match</span>
<span class="k">let</span> <span class="n">worldPos</span> <span class="o">=</span> <span class="n">matrix</span> <span class="o">*</span> <span class="n">modelPos</span><span class="p">;</span>

<span class="c1">// This would throw a compiler error</span>
<span class="k">let</span> <span class="n">wrong</span> <span class="o">=</span> <span class="n">projectionMatrix</span> <span class="o">*</span> <span class="n">modelPos</span><span class="p">;</span>
</code></pre>

</div>



<p>It's basically just using a strong type system to prevent logical errors.</p>

<p><strong>Cleaning up the Syntax</strong><br>
The other thing that bothered me was reading nested math functions. Shader code often ends up looking like this: <code>max(pow(dot(N, L), 32.0), 0.0)</code></p>

<p>You have to read it from inside out. My idea was to read it from left to right, instead of inside out. The outcome would be <code>N.dot(L).pow(32.0).max(0.0)</code></p>

<p><strong>Adding new features</strong></p>

<p>While this is just syntactic sugar, i wanted to implement something that i remember from the language Go. It's called a Receiver Type and it goes like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">type</span> <span class="n">SomeStruct</span> <span class="k">struct</span> <span class="p">{</span> 
    <span class="n">width</span><span class="p">,</span> 
    <span class="n">height</span> <span class="kt">int</span> 
<span class="p">}</span>

<span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">SomeStruct</span><span class="p">)</span> <span class="n">SomeFunction</span><span class="p">()</span> <span class="p">{</span><span class="o">...</span><span class="p">}</span>
</code></pre>

</div>



<p>It's basically the same in Cast.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">struct</span> <span class="n">SomeStruct</span> <span class="p">{</span> <span class="n">x</span><span class="p">:</span> <span class="nb">float</span><span class="p">,</span> <span class="n">y</span><span class="p">:</span> <span class="nb">float</span> <span class="p">}</span>

<span class="k">fn</span> <span class="p">(</span><span class="n">SomeStruct</span><span class="p">)</span> <span class="nf">SomeFunction</span><span class="p">()</span> <span class="p">{</span><span class="o">...</span><span class="p">}</span>
</code></pre>

</div>



<p>Additionally, to keep the file a bit more structured i added in, out and uniform groups.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>uniform {
    ...
}

in {
    ...
}

out {
    ...
}
</code></pre>

</div>



<p>Cast is currently in the "Proof of Concept" stage. It is not production-ready. Many standard features are still missing, and the compiler might crash if you look at it the wrong way.</p>

<p>I am sharing this now because i think the concepts (explicit coordinates and so on) are worth discussing, even if the implementation is still young.</p>

<p>I'd love to hear your feedback on the syntax and the architecture! Feel free to check out the source code, open an issue, or just tell me why my approach to matrix multiplication is wrong.</p>

<p>You can find it here <a href="//github.com/jbrozy/cast">Cast</a>.</p>

