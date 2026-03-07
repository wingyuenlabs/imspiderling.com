---
Title: I overpowered AI by inventing "Brilliant" code by AI opinion itself
Description: 
Author: Valery Zinchenko
Date: 2026-03-07T21:35:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>As it always happens to me, I started from describing a simple API client, then I started playing around Mixins, and at this point I really wanted to make my own Mixins library that is just faster than <a href="https://www.npmjs.com/package/ts-mixer" rel="noopener noreferrer">ts-mixer</a>.</p>

<p>I set up benchmarks and started testing what works fast, I honestly got tired of hours of benchmarking and did what usually developers do right now when there are in a dead-end - asking LLM, I asked my agent to make it fast as possible, usually I don't do this, but this time and decided to establish proper tests and benchmarks, so AI can <strong>comfortably</strong> deprive me of my job. I let it iterate for some time, while going out for bread. Long story short - it did improve the performance, but it also told me that to push limits even further, it would need to sacrifice some specification and "correctness", which I thought is insane since <strong>I felt</strong> like it can be much faster.</p>

<p>I tried some times again and again until I noticed that it's cycling 2-3 the same implementations and ignoring my notes, doesn't matter how much I told it to go further and "don't make mistakes", it still made mistakes, breaking the code, then fixing it and telling me that "no, that's it".</p>

<p>I decided to forget AI and do it myself, it spent even more hours, without buying any single loaf of bread, I've gone over the options. When I felt like I was almost there partially fitting the spec and having better performance - I did a mistake, I waste my time asking AI to finish it <strong>again</strong>. It just gave the same code as before and that's it actually.</p>

<p>Next day, with fresh state of mind - I did it - I came with my own <code>CompositeMap</code> that is just 40 lines and insanely fast. It seems it's also interesting from the point of keys ordering, usually in <code>CompositeMap</code> [A, B] does not equal [B, A]. There are libraries for CompositeMap indeed, moreover there is a <a href="https://github.com/tc39/proposal-composites" rel="noopener noreferrer">TC39 proposal for Composite keys</a>, but these libraries and polyfills are not fast enough for <strong>me</strong>.</p>

<p>That's where I pushed the limits to the top and even asked AI what it thinks about it.<br><br>
This is what it said:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>That code is brilliant. This is a "Performance Masterclass". This is a very clean implementation of a "Discrete Identity Map." ...
</code></pre>

</div>

<p>Though it wasn't able to come up with this code that I need itself, even when I have done everything LLM needs.</p>

<p>This simply highlights that no matter how good AI is and will be, it will never be better than what you do yourself since LLM are trained with what people were able to do in the past and post about it. It is simply unaware and can't reproduced things no one has thought or talked about - that's on <strong>you</strong>.</p>

<p>Can you be better than AI? Yes! In fact, you are better because you are human. Anyone saying "this is a prompting skill issue" or "use another model" are just gasping for what is not there - coding skill. You're not going to be replaced if you do code yourself, but you are replaceable if you can't do code without LLM, it's simple math.</p>

<p>Was I able to save any time using LLM? - hmm, actually no. Whenever I put long running process, it requires careful and exhausting prompting and I just go for a break and out for a bread, but I could've coded myself all this time getting the results much faster or understanding I failed earlier.</p>

<p>Thanks to the company I'm working on by letting me to work on this kind of side work.<br><br>
It's always that people are ignoring "serious" posts about real world things, so I decided to combine memes and make this topic a bit more involving.</p>

<p>Here's the code: <a href="https://github.com/pinely-international/mixin/blob/main/src/CompositeMap.ts" rel="noopener noreferrer">https://github.com/pinely-international/mixin/blob/main/src/CompositeMap.ts</a></p>

<p>Shortly how it works: for every object it assigns ID incrementally and hashes all keys with <a href="https://en.wikipedia.org/wiki/Mask_(computing)" rel="noopener noreferrer">bitmask</a>, this way it's super fast and order-insensitive - it is bit-wise. The bitmask is usually used for things like Flags and usually seen in Game Development. There is a limitation though, bitmask can take very limited about of variants, usually up to 32 (depends how many bits your integer is). In my <code>CompositeMap</code>, if it runs out of 32 bits, it escalates to BigInt, which is still faster than other libraries. However, for Mixins, it's not often to have 32 variations of mixins, so it should be fine anyway.</p>

