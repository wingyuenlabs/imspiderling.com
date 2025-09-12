---
Title: Probabilistic Reasoning (Bite-size Article)
Description: 
Author: koshirok096
Date: 2025-09-12T21:46:27.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Introduction
</h1>

<p>In the real world, not everything can be determined with a simple “yes” or “no.”  </p>

<ul>
<li>A medical test came back positive — but does that really mean the person is sick? There’s still a chance they aren’t.
</li>
<li>An incoming email — is it spam, and with what probability?
</li>
<li>While driving, is that dark object in front of the car a person or just a garbage bag?
</li>
</ul>

<p>In these situations where we can’t be 100% certain, <strong>Probabilistic Reasoning</strong> is the idea of handling possibilities by assigning numbers (probabilities) to them.  </p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9rwj9bly8nxmk9jmkaij.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9rwj9bly8nxmk9jmkaij.png" alt=" " width="800" height="533"></a></p>

<h1>
  
  
  A Dog’s Bark and a Burglar
</h1>

<p><strong>Probabilistic Reasoning</strong> means “reasoning about uncertainty using probability.” To illustrate this, let’s look at a simple example.  </p>

<p>In a certain town, the probability of a burglar coming is <strong>1%</strong>. If a burglar comes, your dog will bark <strong>90% of the time</strong>. However, even when there is no burglar, your dog still barks <strong>5% of the time</strong> at night.  </p>

<p>Given these conditions, we want to answer: <em>“If the dog barks at night, what is the probability that a burglar is actually present?”</em><br><br>
Using Probabilistic Reasoning (via Bayes’ theorem), we can write a short JavaScript example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Probabilistic reasoning with a dog and a burglar (Bayesian update)</span>

<span class="kd">function</span> <span class="nf">posteriorThief</span><span class="p">(</span><span class="nx">priorThief</span><span class="p">,</span> <span class="nx">sensitivity</span><span class="p">,</span> <span class="nx">specificity</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">falseAlarm</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">-</span> <span class="nx">specificity</span><span class="p">;</span>
  <span class="c1">// Bayes' theorem</span>
  <span class="k">return </span><span class="p">(</span><span class="nx">sensitivity</span> <span class="o">*</span> <span class="nx">priorThief</span><span class="p">)</span> <span class="o">/</span>
         <span class="p">(</span><span class="nx">sensitivity</span> <span class="o">*</span> <span class="nx">priorThief</span> <span class="o">+</span> <span class="nx">falseAlarm</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="nx">priorThief</span><span class="p">));</span>
<span class="p">}</span>

<span class="c1">// Prior probability: a burglar comes 1% of the time</span>
<span class="kd">const</span> <span class="nx">priorThief</span> <span class="o">=</span> <span class="mf">0.01</span><span class="p">;</span>
<span class="c1">// Sensitivity: if a burglar comes, the dog barks 90% of the time</span>
<span class="kd">const</span> <span class="nx">sensitivity</span> <span class="o">=</span> <span class="mf">0.90</span><span class="p">;</span>
<span class="c1">// Specificity: if there is no burglar, the dog stays quiet 95% of the time</span>
<span class="c1">// (meaning a 5% false alarm rate)</span>
<span class="kd">const</span> <span class="nx">specificity</span> <span class="o">=</span> <span class="mf">0.95</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">posteriorThief</span><span class="p">(</span><span class="nx">priorThief</span><span class="p">,</span> <span class="nx">sensitivity</span><span class="p">,</span> <span class="nx">specificity</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Probability that a burglar is present when the dog barks =</span><span class="dl">"</span><span class="p">,</span>
            <span class="p">(</span><span class="nx">result</span> <span class="o">*</span> <span class="mi">100</span><span class="p">).</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">1</span><span class="p">),</span> <span class="dl">"</span><span class="s2">%</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>



<p>The result will be:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">Probability</span> <span class="nx">that</span> <span class="nx">a</span> <span class="nx">burglar</span> <span class="nx">is</span> <span class="nx">present</span> <span class="nx">when</span> <span class="nx">the</span> <span class="nx">dog</span> <span class="nx">barks</span> <span class="o">=</span> <span class="mf">15.4</span> <span class="o">%</span>
</code></pre>

</div>






<p>So, “the dog barked = there’s a burglar” does not hold true.<br>
In fact, the probability of a burglar actually being there is only about 15%. Because the prior probability of a burglary (1%) is so low, the posterior probability doesn’t rise very high, even with the evidence of barking.</p>

<p>This is the essence of Probabilistic Reasoning: updating our beliefs under uncertainty using probabilities.</p>




<p>Other familiar applications include:</p>

<ul>
<li><p>Spam detection: “This email is 95% likely to be spam.”</p></li>
<li><p>Weather forecasts: “There’s a 70% chance of rain tomorrow.”</p></li>
<li><p>Autonomous driving: “There’s a 70% chance that the object ahead is a person — stop the car.”</p></li>
</ul>

<p>In our daily lives, we constantly benefit from such probability-based reasoning.</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4k5dsod929ok87qno0s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4k5dsod929ok87qno0s.png" alt=" " width="800" height="533"></a></p>

<h1>
  
  
  Conclusion
</h1>

<p>Probabilistic Reasoning, and the ideas derived from it, are applied in countless aspects of everyday life.<br>
In particular, in hot fields like AI technology and data science, these concepts are used to enable rational decisions such as: “We don’t know for sure, but the likelihood is about this much.”</p>

<p>Thank you for reading.</p>

