---
Title: Reactive Programming and Observables with RxJS
Description: 
Author: Rebekah van Wyk
Date: 2025-09-08T22:03:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>According to the <a href="https://rxjs.dev/" rel="noopener noreferrer">RxJS docs</a>, RxJS, or <em>Reactive Extensions for JavaScript</em>, is a library "for <strong>reactive programming</strong> using <strong>Observables</strong>, to make it easier to compose <strong>asynchronous</strong> or callback-based code”. In this article, we’ll break down the core concepts behind reactive programming and how RxJS makes it easier to work with asynchronous data.</p>

<h2>
  
  
  Reactive Programming
</h2>

<p>To understand what reactive programming is, we should first understand <strong>data streams</strong>. A <strong>data stream</strong> is a flow of potentially infinite data, where the data can arrive at various points in time. </p>

<p>Visually, it might look something like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd3lqhirslhhp8kuqxg5p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd3lqhirslhhp8kuqxg5p.png" alt="Visual representation of a data stream" width="800" height="152"></a></p>

<p>where the dotted line represents time, and our nodes A, B, and C represent data points arriving at different points in time. </p>

<p><strong>Reactive programming</strong> is a paradigm that focuses on data streams and <strong>change propagation,</strong> meaning that when new data arrives, any part of our system that relies on this data automatically reacts and changes accordingly.</p>

<p>RxJS helps us to achieve this through <strong>Observables</strong>.</p>

<h2>
  
  
  What are Observables?
</h2>

<p>Put simply, Observables are data producers that represent a data stream. They emit notifications that we can consume. </p>

<p>There are three types of notifications that can be emitted from an Observable:</p>

<ul>
<li>
<strong>Next</strong> notifications deliver a value emitted by the Observable</li>
<li>
<strong>Error</strong> notifications indicate that something has gone wrong during the Observable's execution</li>
<li>
<strong>Complete</strong> notifications indicate that the Observable has successfully finished its execution</li>
</ul>

<p>All of these notifications are useless to us, however, if we don't have the relevant Observers and Subscriptions in place to effectively consume these notifications.</p>

<h2>
  
  
  Observers and Subscriptions
</h2>

<p><strong>Observers</strong> define how each type of notification from an Observable is handled. </p>

<p>A <strong>subscription</strong> is what actually connects an Observable to an observer. Without a subscription, the Observable would remain idle, and its logic would never run.</p>

<p>The subscribe() method of RxJS creates a subscription, and the unsubscribe() method ends a subscription. Something to note is that once a notification of type <strong>error</strong> or <strong>complete</strong> is emitted from an Observable, the subscription is automatically ended, and nothing further will be emitted by the Observable.</p>

<p>Overall, the relationship between Observables, Observers, and Subscriptions, looks something like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fngvxxygrk2i1xv81yz3e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fngvxxygrk2i1xv81yz3e.png" alt="Relationship between Observables, Observers, and Subscriptions" width="800" height="500"></a></p>

<h3>
  
  
  Why is it important to unsubscribe?
</h3>

<p>In Angular, components can live for a long time. If you forget to unsubscribe from long-running streams, you risk memory leaks, performance issues, and in the worst case scenario, your app could even crash. </p>

<p>Let's have a look at a very simple <strong>code example</strong> of an Observable in action.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">execute</span><span class="p">()</span> <span class="p">{</span>
  <span class="c1">// create an Observable</span>
  <span class="kd">const</span> <span class="nx">observable$</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">((</span><span class="nx">subscriber</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">subscriber</span><span class="p">.</span><span class="nf">next</span><span class="p">(</span><span class="dl">'</span><span class="s1">First emission</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">subscriber</span><span class="p">.</span><span class="nf">next</span><span class="p">(</span><span class="dl">'</span><span class="s1">Second emission</span><span class="dl">'</span><span class="p">),</span> <span class="mi">2000</span><span class="p">);</span>
    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">subscriber</span><span class="p">.</span><span class="nf">complete</span><span class="p">(),</span> <span class="mi">3000</span><span class="p">);</span> <span class="c1">// indicates that we're done and unsubscribes</span>
    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">subscriber</span><span class="p">.</span><span class="nf">next</span><span class="p">(</span><span class="dl">'</span><span class="s1">Final emission</span><span class="dl">'</span><span class="p">),</span> <span class="mi">4000</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="c1">// define an Observer</span>
  <span class="kd">const</span> <span class="nx">observer</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">next</span><span class="p">:</span> <span class="p">(</span><span class="na">value</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Received: </span><span class="dl">'</span><span class="p">,</span> <span class="nx">value</span><span class="p">),</span>
    <span class="na">error</span><span class="p">:</span> <span class="p">(</span><span class="na">err</span><span class="p">:</span> <span class="kr">any</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error: </span><span class="dl">'</span><span class="p">,</span> <span class="nx">err</span><span class="p">),</span>
    <span class="na">complete</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Done</span><span class="dl">'</span><span class="p">),</span>
  <span class="p">};</span>

  <span class="c1">// subscribe to start receiving values</span>
  <span class="c1">// pass our Observer to our subscribe function</span>
  <span class="nx">observable$</span><span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">observer</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In this example, the Observer has indeed defined how notifications from the Observable will be handled; relevant messages will be logged to the console. We'd see the following logged to the console when this execute() method is run:</p>

<ul>
<li>the string "Received: First emission" synchronously</li>
<li>the string "Received: Second emission" after 2 seconds</li>
<li>the string "Done" after 3 seconds</li>
</ul>

<p>Then, despite there being another next notification defined, we'd see nothing further in the console due to the <strong>complete</strong> notification ending the subscription to our Observable. In a more practical example, the number or timing of emissions from our Observable may be unknown, and we'd need to unsubscribe more explicitly. You can learn more about ways to do that in Angular <a href="https://dev.to/hassantayyab/unsubscribing-from-observables-in-angular-a-comprehensive-guide-jf8">here</a>.</p>

<p>While this example is a valid way to work with Observables, in practice we tend to make use of functions in RxJS called <strong>operators</strong> to work with Observables more cleanly and easily.</p>

<h2>
  
  
  RxJS Operators
</h2>

<p>As we've just mentioned, RxJS operators are simply functions defined by RxJS which make it easier to work with Observables. There are various types of operators, as listed <a href="https://rxjs.dev/guide/operators#categories-of-operators" rel="noopener noreferrer">here on the RxJS docs</a>. We would typically make use of these operators to do things like transform, filter, and combine streams of data easily. Let's look at two important categories of RxJS operators: creation operators and pipeable operators.</p>

<h3>
  
  
  Creation operators
</h3>

<p>Creation operators create new Observables with pre-defined behaviour, or by joining other Observables. They can be called as standalone functions.</p>

<p>Some popular creation operators are of(), combineLatest(), and merge(). We'll just have a brief look at the of() operator.</p>

<h4>
  
  
  of()
</h4>

<p>The of() operator returns an Observable that simply emits the provided arguments <strong>synchronously</strong> and then completes.</p>

<p>Code snippet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">of</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">value</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">value</span><span class="p">))</span>
</code></pre>

</div>



<p>Logs:<br>
1<br>
2<br>
3</p>
<h3>
  
  
  Pipeable operators
</h3>

<p>Pipeable operators start with an Observable as input, transform or update the emissions of that Observable in some way, and then return the result as a new Observable. These operators are used with RxJS's <strong>pipe()</strong> method.</p>

<p>Let's have a look at a couple of the most commonly used pipeable operators, namely map() and filter().</p>
<h4>
  
  
  map()
</h4>

<p>The map() operator applies a function to each value emitted from the Observable. </p>

<p>Code snippet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">of</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
  <span class="nf">map</span><span class="p">(</span><span class="nx">num</span> <span class="o">=&gt;</span> <span class="nx">num</span> <span class="o">*</span> <span class="mi">10</span><span class="p">)</span>
<span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">value</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">value</span><span class="p">));</span>
</code></pre>

</div>



<p>Logs:<br>
10<br>
20<br>
30</p>
<h4>
  
  
  filter()
</h4>

<p>The filter() operator only passes values that match a specified predicate.</p>

<p>Code snippet:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">of</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
  <span class="nf">filter</span><span class="p">(</span><span class="nx">num</span> <span class="o">=&gt;</span> <span class="nx">num</span> <span class="o">%</span> <span class="mi">2</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span>
<span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">value</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">value</span><span class="p">));</span>
</code></pre>

</div>



<p>Logs:<br>
2<br>
4</p>

<p>The beauty of the pipe() function is that it allows us to chain operators together to perform powerful operations. For example, we could combine the <strong>map</strong> and <strong>filter</strong> operators in the following way:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">of</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
  <span class="nf">map</span><span class="p">(</span><span class="nx">num</span> <span class="o">=&gt;</span> <span class="nx">num</span> <span class="o">*</span> <span class="mi">10</span><span class="p">),</span>     
  <span class="nf">filter</span><span class="p">(</span><span class="nx">num</span> <span class="o">=&gt;</span> <span class="nx">num</span> <span class="o">&gt;</span> <span class="mi">20</span><span class="p">)</span>
<span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">value</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">value</span><span class="p">));</span>
</code></pre>

</div>



<p>Logs: <br>
30<br>
40 <br>
50</p>

<blockquote>
<p><strong>Note:</strong> <a href="https://rxjs.dev/guide/operators#marble-diagrams" rel="noopener noreferrer">Marble diagrams</a> are a great visual tool to help us understand how each RxJS operator works. <a href="https://rxmarbles.com/" rel="noopener noreferrer">RxMarbles</a> is a website with very useful interactive marble diagrams that can be used to help deepen your understanding of various RxJS Operators. I suggest having a look to further your understanding of the above examples.</p>
</blockquote>

<h2>
  
  
  Real-world applications of Observables
</h2>

<p>Observables are most valuable in cases where code is asynchronous or event-driven in nature. Examples of such scenarios include:</p>

<ul>
<li>Handling HTTP requests </li>
<li>Managing UI events such as clicks, scrolls, and key presses</li>
<li>Working with State Management libraries such as NgRx</li>
</ul>

<p>Observables provide a clean way to handle data as it arrives, making it much easier to manage these kinds of scenarios.</p>

<h2>
  
  
  Conclusion
</h2>

<p>This introduction to RxJS Observables highlighted how they provide a powerful way to handle asynchronous and event-driven data. We discussed how combining Observables with operators enables you to transform, filter, and combine data streams with ease, making reactive programming more intuitive and achievable. I hope this post provided some clarity around what Observables are, how they work, and why they are useful to us. Thanks for reading!</p>

