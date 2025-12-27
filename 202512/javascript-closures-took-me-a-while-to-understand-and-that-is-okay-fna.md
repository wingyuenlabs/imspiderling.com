---
Title: JavaScript Closures Took Me a While to Understand and That Is Okay
Description: 
Author: Ibim
Date: 2025-12-27T21:19:16.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>A reflection on closures, memory, and behaviour</strong></p>

<p>For a long time, I told myself I understood JavaScript closures.</p>

<p>I could repeat the definition. I knew they were related to scope. I had even used them in projects without thinking too much about it. But if I am being honest, none of that meant they truly clicked.<br>
And that is okay.</p>

<p>Closures are one of those ideas in JavaScript that do not reward memorisation. They reward time, exposure, and the uncomfortable moments where code behaves consistently while completely violating your expectations.</p>

<p>For me, that moment came in the form of a bug.</p>

<p><strong>The bug that forced me to actually understand closures</strong></p>

<p>The moment closures finally made sense did not come from documentation. It came from chasing behaviour that felt wrong, even though it was technically correct.</p>

<p>I was looping over a list and attaching handlers. Nothing clever. Nothing experimental. Just everyday JavaScript.</p>

<p>But every handler behaved as if it had forgotten which item it belonged to.</p>

<p>I assumed I had made a mistake. Then I assumed some shared state was being overwritten. I added logs. I rewrote the function. The behaviour did not change.</p>

<p>Every handler was seeing the same value.</p>

<p>A simplified version of the problem looked like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>for (var i = 0; i &lt; 3; i++) {
  setTimeout(() =&gt; {
    console.log(i);
  }, 1000);
}
</code></pre>

</div>



<p>I expected to see 0, 1, and 2.</p>

<p>What I saw was 3, printed three times.</p>

<p>That was the moment I realised I was not dealing with a logic error. I was dealing with a misunderstanding of how JavaScript remembers things.</p>

<p>Each callback was not getting its own copy of the value. They were all holding onto the same variable. By the time the function actually ran, that variable had already changed.</p>

<p>Once I saw that, the behaviour stopped being mysterious. It was predictable. It was even reasonable.</p>

<p>The bug disappeared the moment I stopped asking why JavaScript was doing this and started asking what the function was still connected to.</p>

<p>That was the point where closures stopped being a definition and became a mental model.</p>

<p><strong>Closures are about environments, not syntax</strong></p>

<p>Closures are often introduced as a feature of functions. That framing is convenient, but it is incomplete.</p>

<p>Closures are not something you turn on by writing functions a certain way. They emerge from how JavaScript manages environments over time.</p>

<p>When a function is created, it is associated with the environment in which it was defined. That environment contains bindings, not frozen values. When the function later executes, it resolves those bindings by walking outward through its lexical environments.</p>

<p>A closure exists when that environment remains accessible beyond the lifetime of the original function call.</p>

<p><strong>Why closures persist after functions return</strong></p>

<p>A common misconception is that closures store values. They do not.</p>

<p>What persists is reachability.</p>

<p>If a function still has access to a variable, that variable remains reachable. If it remains reachable, it stays alive in memory. Garbage collection does not care where a variable was declared. It only cares whether something can still reach it.</p>

<p>Once you see closures as a consequence of reachability rather than a special feature, many confusing behaviours stop feeling arbitrary.</p>

<p><strong>Closures explain bugs before they explain features</strong></p>

<p>Most developers encounter closures through bugs long before they use them deliberately.</p>

<p>Asynchronous callbacks, event handlers, and timers all expose the same underlying behaviour. Closures capture variables, not moments in time. Multiple functions can close over the same binding. Delayed execution simply makes this visible.</p>

<p>Understanding this shifts closures from being an interview topic to being a practical debugging tool.</p>

<p><strong>Closures are foundational to encapsulation</strong></p>

<p>Before modern class syntax and private fields, closures were the primary way JavaScript achieved encapsulation.</p>

<p>Even now, they offer something classes cannot always provide cleanly. Privacy without exposure.</p>

<p>A variable that exists only inside a function scope and is accessed only through returned functions is fundamentally inaccessible from the outside. No keyword is required. The structure enforces it naturally.</p>

<p>Closures support intentional design by keeping state local and controlled.</p>

<p><strong>Closures and functional composition</strong></p>

<p>Closures become especially powerful when combined with higher order functions.</p>

<p>Functions that return functions, partially apply arguments, or generate specialised behaviour all rely on closures. This is not an advanced trick. It is a natural extension of how environments work.</p>

<p>Once closures stop feeling mysterious, patterns like currying and composition start to feel obvious rather than clever.</p>

<p><strong>The real cost of closures</strong></p>

<p>Closures are often blamed for performance problems, but that concern is usually misplaced.</p>

<p>The real cost of closures is responsibility.</p>

<p>If you close over large objects, DOM nodes, or long lived references unintentionally, you extend their lifetime. That can lead to memory retention that is difficult to spot.</p>

<p>This is not a reason to avoid closures. It is a reason to understand them well enough to use them deliberately.</p>

<p><strong>A more accurate mental model</strong></p>

<p>After spending time with closures in real code, the most accurate mental model I have found is this:</p>

<p>A closure is not something you create.<br>
A closure is something that remains.</p>

<p>It is the result of:</p>

<ul>
<li>lexical scoping</li>
<li>name resolution</li>
<li>reference reachability</li>
<li>delayed or repeated execution</li>
</ul>

<p>When these conditions align, a closure exists.</p>

<p><strong>Why it is okay that closures take time</strong></p>

<p>Closures sit below the surface of JavaScript. You can write code for years without naming them explicitly. That does not mean you are not using them. It means the language is doing its job.</p>

<p>Understanding closures deeply requires encountering unexpected behaviour, tracing execution mentally, and thinking about memory rather than just output.</p>

<p>That is not beginner work.</p>

<p>If closures took you a while to understand, that is not a failure. It is part of becoming fluent.</p>

<p>They took me a while too.</p>

<p>And that is okay.</p>

<p><strong>Further reading</strong></p>

<p>The ideas in this article were shaped over time by reading and experience. These resources were especially helpful:</p>

<ul>
<li><p>JavaScript from Beginner to Professional. For practical reinforcement of function scope and nested behaviour. </p></li>
<li><p>Eloquent JavaScript (4th Edition) by Marijn Haverbeke. For building intuition around nested scope and environments.</p></li>
<li><p>Clean Code in JavaScript by James Padolsey. For connecting closures to encapsulation, maintainability, and intentional design.</p></li>
</ul>

