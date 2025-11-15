---
Title: Temporal State Coordination: A Timeline of a Timeline
Description: 
Author: Tia Lovelace
Date: 2025-11-15T21:02:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>Long, long ago, back when they were relatively new, I decided to have a go at writing a DOM animation library.</p>

<p>I called it 'Twixt'. Between, tween, betwixt... I know, right?</p>

<p>It didn't matter. It was only for personal use. The existing libraries left me unsatisfied in some way or other, and CSS <code>transition</code> was so new and incomplete it was still vendor-prefixed, if memory serves.</p>

<p>It followed principles that I appreciated more back then than I do today, of declarative code; clean, descriptive objects that tell the engine what I want to animate, and how. It suited me as a hobbyist, more interested in solving problems than finding and applying what exists. The original wheel was terrible.</p>

<p>As with any animation library, I'd give it the information it needs. Element, property, from, to, duration. Optional easing function. It did what I needed. Fade this in. Slide that there.</p>

<p>Sticking points and feature wishlists presented themselves as I used it. I added support for concurrent tweens, each with optional predelay, driven by one central, invisible timer. I added the option to provide a callback instead of an element and a property, so that I could tween more things on that same synchronising timer. I could tween state for canvas rendering, audio volume, anything, all perfectly in sync.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">twixt</span><span class="p">.</span><span class="nf">animate</span><span class="p">({</span>
    <span class="na">from</span><span class="p">:</span> <span class="nx">audio</span><span class="p">.</span><span class="nx">volume</span><span class="p">,</span>
    <span class="na">to</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span>
    <span class="na">duration</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span>
    <span class="na">callback</span><span class="p">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">v</span><span class="p">){</span>
        <span class="nx">audio</span><span class="p">.</span><span class="nx">volume</span> <span class="o">=</span> <span class="nx">v</span>
    <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>It was shaping up to be quite powerful, for its time. I used it for all sorts; particle effects, game engines, snazzy notifications, you name it. As I did so, as I was passing in elements for some effects, callbacks for others, something dawned on me.</p>

<p>It was just as expressive, just as verbose, and more cognitively streamlined to perform all effects, including CSS manipulation, through that callback option. Those callbacks were, after all, simpler expressions of what would be produced by the internal logic that would read those <code>element</code> and <code>property</code> arguments. The API could be simplified.</p>

<p>There were trade-offs, of course. Now I had to be generally conscious of what properties still needed the vendor prefix, but if I wanted to use CSS outside of Twixt - and I did - I ought to be conscious of those anyway, or there were libraries dedicated to solving that. I had to manually combine transform chains, but if I wanted to harness the power of an ordered transform chain I'd have to manually combine them anyway. It's easy enough to write a quick reusable function to combine them in whichever way we need. They were edge cases, and the callback-centric approach allowed me to solve them like any other edge case - by doing what it does, synchronising tweens, and letting me do the rest.</p>

<p>Off I went, animating this and that with the new streamlined API. As I used it to synchronise more and more complex systems, those feature wishlists grew more advanced. A particle system wanted discrete, disposable 'timelines' that come and go. A game engine begged for a way to sync tweens with its own time progression system. The foundation was there in its central timer, but these features would mean significant rethinking and refactoring.</p>

<p>This led to the penultimate incarnation: timeline.ts. A typed, composable, seekable set of tweens and timestamped events, packaged in an object with its own driving timer. The API was overhauled to meet its new purpose-identity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">tl</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Timeline</span><span class="p">();</span>
<span class="nx">tl</span><span class="p">.</span><span class="nf">tween</span><span class="p">(</span>
    <span class="mi">0</span><span class="p">,</span> <span class="c1">// start</span>
    <span class="mi">500</span><span class="p">,</span> <span class="c1">// duration</span>
    <span class="p">(</span><span class="nx">v</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">element</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">opacity</span> <span class="o">=</span> <span class="nx">v</span><span class="p">,</span> <span class="c1">// apply</span>
    <span class="mi">1</span><span class="p">,</span> <span class="c1">// from value</span>
    <span class="mi">0</span><span class="p">,</span> <span class="c1">// to value</span>
    <span class="nx">easers</span><span class="p">.</span><span class="nx">easeIn</span>
<span class="p">);</span>

<span class="nx">tl</span><span class="p">.</span><span class="nf">at</span><span class="p">(</span><span class="mi">500</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">element</span><span class="p">.</span><span class="nf">remove</span><span class="p">());</span>

<span class="c1">// tl.seek(400);</span>
<span class="nx">tl</span><span class="p">.</span><span class="nf">play</span><span class="p">();</span>
</code></pre>

</div>



<p>I no longer conceptualised the library as an animation system. Now it was a <strong>temporal state coordination engine</strong>. An intuitive means to synchronise complex patterns of state with a dimension of time. Not just time. <em>Input</em>. An easy-to-read, easy-to-write model for creating complex functions of <code>(n) =&gt; state</code>.</p>

<p>That conceptual shift opened doors for new ways of approaching the API's design, new ways of approaching temporal state composition. I still felt it could be more.</p>

<p>I credit RxJS for inspiration for Timeline's new emission transform model. RxJS itself wasn't quite suited, but it wasn't too far off. The model needed some tweaking to fit a synchronised state engine without undue complexity and overhead. I'd already been considering ways to introduce the concept of ranges to Timeline, but I'd struggled to imagine a model that would justify the fundamental rewrite it would entail until I realised a range could emit a progression, and that could be chained to emit a composed tween.</p>

<p>So I rewrote Timeline from scratch, as a <em>parametric transformation pipeline</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">tl</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Timeline</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">oneSecondIn</span> <span class="o">=</span> <span class="nx">tl</span><span class="p">.</span><span class="nf">point</span><span class="p">(</span><span class="mi">1000</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">firstFiveSeconds</span> <span class="o">=</span> <span class="nx">tl</span><span class="p">.</span><span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">5000</span><span class="p">);</span>

<span class="nx">oneSecondIn</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">One second has passed</span><span class="dl">"</span><span class="p">));</span>

<span class="c1">// naive use of progression emissions:</span>
<span class="nx">firstFiveSeconds</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(</span><span class="nx">v</span> <span class="o">=&gt;</span> 
        <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`We are </span><span class="p">${</span><span class="nx">v</span> <span class="o">*</span> <span class="mi">100</span><span class="p">}</span><span class="s2">% through the first five seconds`</span><span class="p">)</span>
    <span class="p">);</span>

<span class="c1">// apply transformation:</span>
<span class="kd">const</span> <span class="nx">percentEmitter</span> <span class="o">=</span> <span class="nx">firstFiveSeconds</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">progress</span> <span class="o">=&gt;</span> <span class="nx">progress</span> <span class="o">*</span> <span class="mi">100</span><span class="p">);</span>

<span class="c1">// further transform:</span>
<span class="kd">const</span> <span class="nx">percentStringEmitter</span> <span class="o">=</span> <span class="nx">percentEmitter</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">floor</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">v</span> <span class="o">=&gt;</span> <span class="nx">v</span> <span class="o">+</span> <span class="dl">"</span><span class="s2">%</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// use it</span>
<span class="nx">percentStringEmitter</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">percent</span> <span class="o">=&gt;</span> <span class="s2">`Loading: </span><span class="p">${</span><span class="nx">percent</span><span class="p">}</span><span class="s2">%`</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(</span><span class="nx">msg</span> <span class="o">=&gt;</span> <span class="nb">document</span><span class="p">.</span><span class="nx">title</span> <span class="o">=</span> <span class="nx">msg</span><span class="p">);</span>
<span class="c1">// twice</span>
<span class="nx">percentStringEmitter</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(</span><span class="nx">v</span> <span class="o">=&gt;</span> <span class="nx">progressBar</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">width</span> <span class="o">=</span> <span class="nx">v</span><span class="p">);</span>

<span class="c1">// state isn't just visual:</span>
<span class="nx">tl</span><span class="p">.</span><span class="nf">range</span><span class="p">(</span><span class="mi">1000</span><span class="p">,</span> <span class="mi">500</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">ease</span><span class="p">(</span><span class="nx">easers</span><span class="p">.</span><span class="nx">easeInOut</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">tween</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">255</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(</span><span class="nx">v</span> <span class="o">=&gt;</span> <span class="nx">microcontroller</span><span class="p">.</span><span class="nf">setPWM</span><span class="p">(</span><span class="nx">v</span><span class="p">));</span>

<span class="kd">const</span> <span class="nx">twoSecondsIn</span> <span class="o">=</span> <span class="nx">oneSecondIn</span><span class="p">.</span><span class="nf">delta</span><span class="p">(</span><span class="mi">1000</span><span class="p">);</span>
<span class="nx">tl</span><span class="p">.</span><span class="nf">seek</span><span class="p">(</span><span class="nx">twoSecondsIn</span><span class="p">);</span>
<span class="nx">tl</span><span class="p">.</span><span class="nf">seek</span><span class="p">(</span><span class="nx">firstFiveSeconds</span><span class="p">.</span><span class="nx">end</span><span class="p">,</span> <span class="mi">3000</span><span class="p">,</span> <span class="dl">"</span><span class="s2">easeInOut</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<p>Ranges emit a value between 0 and 1 as 'time' passes through them. Each step in the transform chain is a distinct, immutable emitter, each listening to its parent and emitting a value of its own.</p>

<p>The <code>tween()</code> emitter will interpolate numbers, number arrays, tokens within strings and blendable objects, to keep your code as expressive as possible. Point events can include undo logic, for those fancy backward seeks. There are subtle touches to its logic that mean every seek gives a predictable, deterministic result.</p>

<p>I've found this to be a powerful mental and logical model for synchronising complex states over time, and I'm proud of where this long study of API design has led me.</p>

<p>I wrote Twixt because I wanted to include unique, performant, predictable animations in my web designs, but as I've used it and refined it I've found so many other things it can conveniently synchronise. Composed attack animations with specific sub-frame moments of infliction. XP metres that glow and swell as they fill. <a href="https://codepen.io/xtaltia/pen/jEWrXOg" rel="noopener noreferrer">Ridiculously easy spritesheet animation logic</a>. Real-time scheduling in backend logic. All with just a few lines of clean, clear code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// drive with real-time</span>
<span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">timeline</span><span class="p">.</span><span class="nf">seek</span><span class="p">(</span><span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()),</span> <span class="mi">1000</span><span class="p">);</span>

<span class="c1">// schedule maintenance</span>
<span class="nx">timeline</span>
    <span class="p">.</span><span class="nf">point</span><span class="p">(</span><span class="nx">someDate</span><span class="p">.</span><span class="nf">getTime</span><span class="p">())</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">downForMaintenance</span> <span class="o">=</span> <span class="kc">true</span><span class="p">);</span>

<span class="c1">// graduate config parameters over a period</span>
<span class="nx">timeline</span>
    <span class="p">.</span><span class="nf">range</span><span class="p">(</span><span class="nx">someDate</span><span class="p">.</span><span class="nf">getTime</span><span class="p">(),</span> <span class="mi">3600000</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">tween</span><span class="p">(</span><span class="mi">1000</span><span class="p">,</span> <span class="mi">50</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">apply</span><span class="p">(</span><span class="nx">v</span> <span class="o">=&gt;</span> <span class="nx">server</span><span class="p">.</span><span class="nx">rateLimit</span> <span class="o">=</span> <span class="nx">v</span><span class="p">);</span>
</code></pre>

</div>



<p>If this summary glance of Timeline strikes your interest, you can read more about it at <a href="https://www.npmjs.com/package/@xtia/timeline" rel="noopener noreferrer">npmjs - @xtia/timeline</a> or play around with the <a href="https://stackblitz.com/edit/timeline-string-tween?file=src%2Fmain.ts" rel="noopener noreferrer">string tweening demo at StackBlitz</a>. If I hear it's helped someone with a tricky synchronisation issue I'll go to bed happy.</p>

<p><iframe height="600" src="https://codepen.io/xtaltia/embed/JoGWzQV?height=600&amp;default-tab=result&amp;embed-version=2">
</iframe>
</p>

