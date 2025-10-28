---
Title: Before You Learn Closures, Understand Context
Description: 
Author: Ghassen Faidi
Date: 2025-10-28T21:58:50.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkd6ags4vh1bpbshmbsma.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkd6ags4vh1bpbshmbsma.png" alt="painting of beautiful cherry blossoms" width="800" height="533"></a><br>
Names give meaning to our world, our feelings, and so our programs. and that's where our story shall begin.</p>

<p>Context is one of the most foundational concepts in programming and in JavaScript, and it is a building block that leads us to powerful ideas like lexical scope and closures<br>
It's worth noting from the start that in many programming languages, they use the term "environment" instead of context. it's the same thing, and both terms work well.</p>

<p>To understand context, we need to take a step back and talk about values, the ones we deal with in programs: integers, strings, etc. </p>

<p>At first, of course, values alone are useless. We need to combine them together to create expressions, but at the end of the day, all expressions will <em>evaluate</em> to a value. What does "evaluate" mean?<br>
It means we take an expression and, step by step, turn it into a value. We can think of a value as an expression that cannot be evaluated any further. For example, imagine you type <code>1 + 2</code> into your console and press enter. The interpreter (the thing that runs your code) will print <code>3</code>. If you type <code>3</code> and press enter, it will print <code>3</code> again: it cannot evaluate it any further. It's already a value. (Some programming languages call this an <em>atom</em>.)</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgpnegij28c4ee47vcpz0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgpnegij28c4ee47vcpz0.png" alt="Javascript's console" width="791" height="156"></a></p>

<p>Let's see more examples. An expression like <code>3 + 5</code> will evaluate to <code>8</code>, and <code>"hello " + "world"</code> will evaluate to <code>"hello world"</code>. Every operator (or function) has a set of rules. the so-called <em>evaluation rules</em>, or the <em>semantics</em> of the language.<br>
When we understand the evaluation rules, we understand how things work. JavaScript's documentation (<a href="https://tc39.es/ecma262/" rel="noopener noreferrer">ECMAScript specs</a>) contains those evaluation rules, and those who implement the interpreter make sure it meets those requirements.</p>

<p>Now, expressions are pretty good. We use them all day. But as programmers, we figured it's probably a good idea to give names to expressions. It's much easier! it's basically an "abstraction". Instead of thinking about the value 3.1415, I'll just think about the name we give to that value: PI. (That's why we say math is abstract, lots of symbols!)</p>

<p>When we give a name to a value, we use the term <em>binding</em>. That is, we say we bind x to the value 5, or we say x is bound to 5.</p>

<p>But wait a minute. Before, we just had expressions floating in the sea of bits and bytes, but now, giving names creates new challenges. What if I see "x" in my code... how does JavaScript know what value it's associated with? We need a place to store the name and the value associated with it. We need some sort of memory (a data structure) where we can store those <em>bindings</em>—that is, we store each name and the value it maps to.</p>

<p>We usually don't use the term "name" that much and we prefer other terms like "variable", but the term "name" is very abstract and it will be made clear why that matters a bit later.</p>

<p>Anyway, imagine you are a interpreter. You see an expression like <code>x + 1</code>. Well, you know the value 1, it's clear, but who or what the heck is x? that's why we need some sort of memory! the best that comes to mind is some sort of a hash map (i.e., a dictionary, something that has key-value pairs)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "x": 5
}
</code></pre>

</div>



<p>That's good enough for our mental model! Now whenever the code sees x, it can look at this memory and figure out what it evaluates to (that is, 5).</p>

<p>We call this data structure a <strong>context</strong>.</p>

<p>Well, the thing is, by creating this data structure, we made our language much more sophisticated. We have cool functionalities (also known as "constructs") like assignment!</p>

<p>Assignment is simple. It's basically a function that binds a variable name to a value:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">hobby</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">reading</span><span class="dl">"</span>
</code></pre>

</div>



<p>This will bind hobby to the value "reading".<br>
One question that comes to mind: why do we use the term "bind"? Isn't it just assignment? Why all this jargon?</p>

<p>The reason is that binding is more general than assignment. Binding means associating a name to a value. </p>

<p>For instance, an import statement:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="k">import</span> <span class="nx">m</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./my-module</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">log</span><span class="p">,</span> <span class="nx">factorial</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./math.js</span><span class="dl">"</span>
</code></pre>

</div>



<p>Now, the name (or <em>identifier</em>) <code>m</code> is bound to the module object (or whatever the module exports). We can say <code>log</code> is bound to a function, but we don't say it's an assignment, because it isn't an assignment. We also have function bindings (i.e. function declaration), and we even have <code>this</code> binding (i.e., what value this is bound to when we call a function).<br>
In short, assignment <em>is</em> a form of binding, but not all bindings are assignments.</p>

<p>Anyway, enough discussion about terminology.<br>
We talked about contexts being some sort of memory that keeps track of bindings. In JavaScript, we have a global context, which is accessible from anywhere.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">friend</span><span class="dl">"</span>
<span class="kd">function</span> <span class="nf">hello</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">hello</span><span class="dl">"</span><span class="p">,</span> <span class="nx">x</span><span class="p">)</span>
<span class="p">}</span>
<span class="nf">hello</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span>
</code></pre>

</div>



<p>after executing the first two lines, the global environment would look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code> <span class="p">{</span>
    <span class="dl">"</span><span class="s2">x</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">friend</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">hello</span><span class="dl">"</span><span class="p">:</span> <span class="o">&lt;</span><span class="kd">function</span> <span class="nf">object</span><span class="o">&gt;</span>
 <span class="p">}</span>
</code></pre>

</div>



<p>functions are values, we can think of a function as a special object. <br>
now the interesting part:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">hello</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span>
</code></pre>

</div>



<p>the interpreter sees this line and it feels anxious, "hello", what on earth is that? we have also two parenthesis, certainly it's a function call, and there is an "x" there, well, let's start evaluating.<br>
it looks "hello" in the context and it finds it,<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;function object&gt;(x)
</code></pre>

</div>



<p>good, but I can't call the function with "x", we need to evaluate it first, we look it up in the global context, and evaluate it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;function object&gt;("friend")
</code></pre>

</div>



<p>now the function is called. and "hello friend" is printed.</p>

<p>But Soon we discovered the need for separate contexts for different places. For instance, inside a function body, you wouldn't want the variables defined there to be accessible outside. We don't want to clutter the global context with irrelevant variable names!</p>

<p>And so, we ended up with the concept of scope.</p>

<p>It's basically a set of rules each programming language defines, so that you as a programmer memorize them (and your compiler knows them by heart) so that you understand the code you read and can write code that makes sense. Basically, when the interpreter sees x + 1, scoping rules tell the compiler <em>where</em> to look to find the value of x (i.e., what x evaluates to)—that is, what context to look in and where not to look.</p>

<p>Takeaway: scope is simply a consequence of having contexts, which is a consequence of us programmers liking to give names. </p>

<p>And well, it's human nature to give names. Names make us express ourselves and give meaning to expressions :)</p>

<p>And so, really, scoping rules can get complicated, and every language can differ a bit, and that's okay! It's a human effort, and we can make mistakes, and it can get really complicated to the point that JavaScript has 3 or more ways to bind a variable!</p>

<p>Those terms are fundamental to understanding the essence of JavaScript. Closures and lexical scopes require articles of their own, but let's take one concept: lexical scoping.</p>

<p>Lexical scoping means a function body gets evaluated (we go through the lines of code and run them line by line) in the context where it was defined, not the context where it was called.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="mi">10</span>
<span class="kd">function</span> <span class="nf">test</span><span class="p">()</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span>
<span class="p">}</span>
<span class="kd">function</span> <span class="nf">main</span><span class="p">(</span><span class="nx">y</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">y</span><span class="p">)</span>
    <span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="mi">20</span>
    <span class="nf">test</span><span class="p">()</span>
<span class="p">}</span>
<span class="nf">main</span><span class="p">(</span><span class="dl">"</span><span class="s2">hello</span><span class="dl">"</span><span class="p">)</span>

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>hello
10
</code></pre>

</div>



<p>Let's trace through and think about context.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="mi">10</span>
</code></pre>

</div>



<p>That line is a let binding. It binds x (or simply, "assigns x") to the value 10.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">test</span><span class="p">()</span> <span class="p">{...}</span>
</code></pre>

</div>



<p>This is a function binding. It binds <code>test</code> to the function object (again, functions are values too!).</p>

<p>Right now, the environment looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>global context: {
    "x": 10
    "test": &lt;function object&gt;
}
</code></pre>

</div>



<p>One might wonder, what about the function body? there is a difference between a function binding (declaration) and a function call. We only evaluate the body when it's a function call.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{...}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>global context: {
    "x": 10
    "test": &lt;function object&gt;
    "main": &lt;function object&gt;
}
</code></pre>

</div>



<p>Now, finally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">main</span><span class="p">(</span><span class="dl">"</span><span class="s2">hello</span><span class="dl">"</span><span class="p">)</span>
</code></pre>

</div>



<p>The scoping rules say, whenever you call a function, a brand new context for that function will be created! It's empty at first:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>main's context: {}
</code></pre>

</div>



<p>The first thing we do is bind the parameters to the argument values. (As I said, the term "bind" is super useful: it can be used anywhere you want to associate a name to a value.)</p>

<p>In our case, we bind the parameter y to the argument value "hello". (Some people created this distinction for clarity, Parameters are the names we define in the function signature; arguments are the actual values we pass to it.)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>main's context: {
    "y": "hello"
}
</code></pre>

</div>



<p>The next line we want to evaluate is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">y</span><span class="p">)</span>
</code></pre>

</div>



<p>Okay, it's a function call, and the function expects an argument. But the JavaScript interpreter pauses for a moment. Again? Who is y? We can't call the function until we know y's value.</p>

<p>But rest assured, we already have a place where we stored the value: the function's context!</p>

<p><code>console.log(y)</code> evaluates to <code>console.log("hello")</code> and finally prints to the console.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">x</span> <span class="o">=</span> <span class="mi">20</span>
</code></pre>

</div>



<p>This line binds x to the value 20. But what x? Well, that's the wrong question. We don't care about that. We just have a value we want to give a name to. The right question is where: where do we want to store this binding? the answer is, the current environment!<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>global context: {
    "x": 10
    "test": &lt;function object&gt;
    "main: &lt;function object&gt;
}
main's context: {
    "y": "hello"
    "x": 20
}
</code></pre>

</div>



<p>Now, the final line!<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">test</span><span class="p">()</span>
</code></pre>

</div>



<p>It's a function call. As you can see, it takes no arguments. The scoping rules say we should create a new context where we store the bindings defined in that function:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>test's context: {}
</code></pre>

</div>



<p>It has only one line:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span>
</code></pre>

</div>



<p>Okay, we need to call the log function, but AGAIN, sadly we can't do it yet. we need to evaluate x. But where?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>test's context: {}
</code></pre>

</div>



<p>We don't find it in the function's context... where do we look next?</p>

<p>Now, the concept of <strong>chaining contexts</strong> appears. It says, if you don't find the binding you are looking for in the local function's context, look it up in the context  where you defined the function. That's called <strong>lexical scoping</strong>.</p>

<blockquote>
<p>lexical scoping: you always look up for variable names in the context where the function was <em>defined</em> , and not context where it was <em>called</em>.</p>
</blockquote>

<p>We defined the function in the global scope, so we look there:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>global context: {
    "x": 10
    "test": &lt;function object&gt;
    "main": &lt;function object&gt;
}
</code></pre>

</div>



<p>We find it, and we grab the value from there:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="mi">10</span><span class="p">)</span>
</code></pre>

</div>



<p>Now we can call the log function and print 10 on the screen.<br>
that's it for this example.</p>

<p>By understanding the concept of context, many things start to fall into place, and it gives meaning to many of JavaScript's quirks, like the difference between let and var, hoisting, closures, and much more.</p>

<p>It all started by someone who thought it's a good idea to give names to things in programs.</p>

<blockquote>
<p>They should call it—let me see—the White Way of Delight. Isn't that a nice imaginative name? Anne</p>
</blockquote>

