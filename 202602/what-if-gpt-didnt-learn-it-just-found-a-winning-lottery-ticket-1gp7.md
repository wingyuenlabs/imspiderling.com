---
Title: What If GPT Didn’t “Learn”, It Just Found a Winning Lottery Ticket?
Description: 
Author: Maulik Sompura
Date: 2026-02-19T21:49:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>I used to say it confidently:</p>

<p>“The model learned this.”</p>

<p>It felt obvious. We initialize weights. We run gradient descent. We minimize loss. The network learns.</p>

<p>End of story.</p>

<p>But then I came across a research paper that genuinely disturbed that simple narrative.</p>

<p>While reading late one night, I stumbled upon a paper by Jonathan Frankle and Michael Carbin. At first, it looked like just another pruning paper. But the core claim made me stop and reread it twice.</p>

<p>It suggested something radical:</p>

<p>What if neural networks don’t build intelligence from scratch during training?</p>

<p>What if, hidden inside a randomly initialized network, there already exists a smaller subnetwork that is capable of solving the task and training merely discovers it?</p>

<p>That idea is known as the Lottery Ticket Hypothesis.</p>

<p>And if it’s even partially true, then gradient descent isn’t constructing intelligence.</p>

<p>It’s searching for a winning ticket inside structured randomness.</p>

<p>Before we go further, let’s unpack what that actually means.</p>

<h2>
  
  
  The Idea That Quietly Shook Deep Learning
</h2>

<p>In 2018, <strong>Jonathan Frankle</strong> and <strong>Michael Carbin</strong> proposed something almost heretical:</p>

<blockquote>
<p>Inside every randomly initialized neural network, there exists a smaller subnetwork that is already capable of learning the task.</p>
</blockquote>

<p>They called it the <strong>Lottery Ticket Hypothesis (LTH)</strong>.</p>

<p>The claim?</p>

<p>A huge neural network contains a sparse “winning ticket” —<br>
a subnetwork that, when trained alone (starting from the original initialization), matches the full network’s performance.</p>


<h2>
  
  
  Why “Lottery Ticket”?
</h2>

<p>Imagine you randomly initialize a huge neural network.</p>

<p>Most weights are useless.</p>

<p>But hidden inside that random initialization is a rare configuration of connections that is already aligned with the task.</p>

<p>Training doesn’t create intelligence from scratch.<br>
It discovers and amplifies that lucky subnetwork.</p>

<p>Like buying lottery tickets, most are worthless, but a few are already winners.</p>


<h2>
  
  
  The Experiment (Simplified)
</h2>

<p>Researchers performed a surprisingly simple experiment:</p>

<ol>
<li>Train a full network to convergence.</li>
<li>Prune (remove) the smallest-magnitude weights.</li>
<li>Reset the remaining weights back to their original random initialization.</li>
<li>Retrain only the pruned subnetwork.</li>
</ol>

<p>Shockingly:</p>

<p>The pruned network trained just as well.</p>

<p>Sometimes even faster.</p>

<p>This means the full network wasn’t necessary.<br>
The winning subnetwork was already present at initialization.</p>


<h2>
  
  
  Why This Is Mind-Blowing
</h2>

<p>It challenges the intuitive story:</p>

<p>❌ “Training builds intelligence.”</p>

<p>Instead, it suggests:</p>

<p>✅ “Initialization already contains many potential intelligent subnetworks.”</p>

<p>Training becomes less about constructing intelligence and more about <strong>searching for one good structure hidden inside randomness.</strong></p>

<p>That shifts deep learning from:</p>

<p>Optimization theory<br>
to<br>
Combinatorial search through structured randomness.</p>


<h2>
  
  
  The Mathematical Angle
</h2>

<p>Let a neural network be:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>f(x; θ)
</code></pre>

</div>



<p>where:</p>

<ul>
<li>θ ∈ ℝᵈ</li>
<li>d could be millions or billions.</li>
</ul>

<p>LTH says:</p>

<p>There exists a binary mask m ∈ {0,1}ᵈ such that:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>f(x; m ⊙ θ₀)
</code></pre>

</div>



<p>trained alone performs as well as the full model.</p>

<p>Where:</p>

<ul>
<li>θ₀ = original random initialization</li>
<li>⊙ = elementwise multiplication</li>
</ul>

<p>So the “winning ticket” is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>m ⊙ θ₀
</code></pre>

</div>



<p>The crucial detail:</p>

<p>If you randomly reinitialize after pruning, performance drops.</p>

<p>That means the specific random draw of θ₀ contained structure.</p>

<p>Randomness wasn’t neutral noise.<br>
It already encoded useful geometry.</p>


<h2>
  
  
  Even Deeper Implications
</h2>
<h2>
  
  
  Overparameterization Might Be a Search Strategy
</h2>

<p>Why are large language models enormous?</p>

<p>Maybe not because they need every parameter.</p>

<p>Maybe because a larger network increases the probability that a good subnetwork exists.</p>

<p>If each subnetwork has a tiny probability p of being “trainable,” and you have N possible subnetworks, then the probability at least one works is roughly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1 − (1 − p)^N
</code></pre>

</div>



<p>As N grows, this rapidly approaches 1.</p>

<p>Bigger models might simply mean more lottery tickets.</p>




<h2>
  
  
  Random Initialization Is Not “Just Noise”
</h2>

<p>We usually think:</p>

<p>Random weights = meaningless chaos.</p>

<p>But in high dimensions, strange things happen:</p>

<ul>
<li>Concentration of measure</li>
<li>Emergent correlations</li>
<li>Structured spectral properties of random matrices</li>
</ul>

<p>Random high-dimensional systems often contain surprising regularities.</p>

<p>LTH suggests intelligence might emerge from those hidden regularities.</p>

<p>This connects to:</p>

<ul>
<li>Random matrix theory</li>
<li>High-dimensional probability</li>
<li>Sparse approximation theory</li>
</ul>




<h2>
  
  
  Compression and Pruning Make More Sense
</h2>

<p>LTH helps explain why:</p>

<ul>
<li>Pruned networks often retain performance</li>
<li>Sparse models can match dense ones</li>
<li>Quantized models still work</li>
</ul>

<p>Redundancy may not be inefficiency.</p>

<p>It may be a probabilistic discovery mechanism.</p>




<h2>
  
  
  LTH and Large Language Models
</h2>

<p>Now the dangerous thought.</p>

<p>Modern systems like GPT have billions of parameters.</p>

<p>What if:</p>

<ul>
<li>Only a fraction are truly necessary?</li>
<li>Reasoning lives in sparse subnetworks?</li>
<li>Scaling works because it increases the chance of containing rare reasoning-capable configurations?</li>
</ul>

<p>Instead of:</p>

<p>“Bigger models are smarter.”</p>

<p>It might be:</p>

<p>“Bigger models are more likely to contain a rare intelligent structure.”</p>

<p>That’s a radically different interpretation of scaling laws.</p>




<h2>
  
  
  But It Gets Stranger
</h2>

<p>Later research found that, for very large networks, resetting to <strong>early training weights</strong> works better than resetting to full initialization.</p>

<p>This led to ideas like:</p>

<ul>
<li>Early-bird tickets</li>
<li>Mode connectivity</li>
<li>Linear low-loss paths between solutions</li>
</ul>

<p>The loss landscape appears smoother and more connected than classical intuition suggests.</p>

<p>The geometry of deep learning is far stranger than we once believed.</p>




<h2>
  
  
  Open Questions
</h2>

<p>We still don’t fully understand:</p>

<ul>
<li>Why winning tickets exist</li>
<li>How large they must be</li>
<li>Whether the hypothesis scales cleanly to transformers</li>
<li>Whether reasoning is localized or distributed</li>
</ul>

<p>The theory is incomplete.</p>

<p>The implications are enormous.</p>




<h2>
  
  
  The Radical Interpretation
</h2>

<p>Some researchers speculate:</p>

<p>Deep networks behave like massive random feature ensembles.<br>
Training selects coherent sparse structures from that ensemble.</p>

<p>That comes dangerously close to saying:</p>

<p>Intelligence emerges from structured randomness.</p>

<p>Not from careful deterministic construction.</p>




<h2>
  
  
  The Truly Mind-Bending Part
</h2>

<p>If the Lottery Ticket Hypothesis holds at scale…</p>

<p>Then scaling laws might reflect:</p>

<p>Extreme value statistics in high dimensions.</p>

<p>GPT’s performance curves might be explainable through probability theory not just optimization.</p>

<p>That would connect large language models to:</p>

<ul>
<li>Statistical physics</li>
<li>Spin glass theory</li>
<li>Phase transitions</li>
</ul>

<p>And that’s when deep learning stops looking like engineering…</p>

<p>…and starts looking like high-dimensional statistical mechanics.</p>

<p>So next time someone says:</p>

<p>“GPT learned that.”</p>

<p>You might ask:</p>

<p>Did it learn?</p>

<p>Or did it find a winning ticket hidden inside randomness?</p>

<p>The Actual Research Paper You can find it here..<br>
<a href="https://arxiv.org/abs/1803.03635" rel="noopener noreferrer">https://arxiv.org/abs/1803.03635</a></p>

