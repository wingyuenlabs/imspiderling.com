---
Title: Inside ChatGPT: Deconstructing "Attention Is All You Need" (Part 1)
Description: 
Author: Tuntufye Mwakalasya 
Date: 2025-11-21T21:49:32.000Z
Robots: noindex,nofollow
Template: index
---
<p>To understand how modern Large Language Models (LLMs) like ChatGPT work, we must first understand the architecture that changed everything: the Transformer. Before we dive into the complex layers, we need to establish why we moved away from previous methods and how the model initially processes language.</p>

<h3>
  
  
  1. The Predecessor: Recurrent Neural Networks (RNNs) and Their Limitations
</h3>

<p>Before the "Attention Is All You Need" paper, the standard for processing sequential data (like text) was the Recurrent Neural Network (RNN).In an RNN, data is processed sequentially. We give the network an initial state (State 0) along with an input x1 to produce an output y1 and a hidden state. This hidden state is passed forward to the next step, allowing the network to "remember" previous inputs.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnuo5bxbcze354rq8ki51.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnuo5bxbcze354rq8ki51.png" alt=" " width="800" height="436"></a></p>

<h1>
  
  
  The Vanishing Gradient Problem
</h1>

<p>While intuitive, RNNs suffer from severe limitations, specifically slow computation for long sequences and the <strong>vanishing</strong> or <strong>exploding gradient problem</strong>.</p>

<p>To understand this, let's look at calculus, specifically the Chain Rule.</p>

<p>If we have a composite function 

</p>
<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">F(x)=(f∘g)(x)F(x) = (f \circ g)(x) </span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord mathnormal">F</span><span class="mopen">(</span><span class="mord mathnormal">x</span><span class="mclose">)</span><span class="mspace"></span><span class="mrel">=</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mopen">(</span><span class="mord mathnormal">f</span><span class="mspace"></span><span class="mbin">∘</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord mathnormal">g</span><span class="mclose">)</span><span class="mopen">(</span><span class="mord mathnormal">x</span><span class="mclose">)</span></span></span></span></span>
</div>
, the derivative is:


<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">F′(x)=f′(g(x))⋅g′(x)
F'(x) = f'(g(x)) \cdot g'(x)
</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord"><span class="mord mathnormal">F</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">′</span></span></span></span></span></span></span></span></span><span class="mopen">(</span><span class="mord mathnormal">x</span><span class="mclose">)</span><span class="mspace"></span><span class="mrel">=</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord"><span class="mord mathnormal">f</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">′</span></span></span></span></span></span></span></span></span><span class="mopen">(</span><span class="mord mathnormal">g</span><span class="mopen">(</span><span class="mord mathnormal">x</span><span class="mclose">))</span><span class="mspace"></span><span class="mbin">⋅</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord"><span class="mord mathnormal">g</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">′</span></span></span></span></span></span></span></span></span><span class="mopen">(</span><span class="mord mathnormal">x</span><span class="mclose">)</span></span></span></span></span>
</div>


<p>In a deep neural network, backpropagation involves multiplying gradients layer by layer (like the chain rule). If we have many layers, we are essentially multiplying many numbers together.</p>

<p>Imagine multiplying fractions like:</p>


<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">12×12×12×⋯
\frac{1}{2} \times \frac{1}{2} \times \frac{1}{2} \times \cdots
</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord">2</span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace"></span><span class="mbin">×</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord">2</span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace"></span><span class="mbin">×</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord">2</span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace"></span><span class="mbin">×</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="minner">⋯</span></span></span></span></span>
</div>


<p>As the number of layers (or time steps) increases, this number becomes infinitesimally small ("vanishes") or massively large ("explodes"). This makes it incredibly difficult for the model to access or learn from information that appeared early in a long sequence.</p>

<h2>
  
  
  2. The Transformer Architecture
</h2>

<p>The Transformer abandons recurrence entirely, relying instead on an <strong>Encoder-Decoder</strong> architecture. It processes the entire sequence at once, which solves the speed and long-term dependency issues of RNNs.</p>

<h3>
  
  
  The Input Matrix
</h3>

<p>Let's look at how data enters the model.</p>

<p>If we have an input sentence of length 6 (Sequence Length) and a model dimension (
</p>
<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">dmodeld_{model}</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord"><span class="mord mathnormal">d</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">m</span><span class="mord mathnormal mtight">o</span><span class="mord mathnormal mtight">d</span><span class="mord mathnormal mtight">e</span><span class="mord mathnormal mtight">l</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span></span></span></span>
</div>
) of 512, our input is a matrix of size 
<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">(6,512)(6, 512)</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mopen">(</span><span class="mord">6</span><span class="mpunct">,</span><span class="mspace"></span><span class="mord">512</span><span class="mclose">)</span></span></span></span></span>
</div>
.

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frq1og6xg2j7sugx3yobr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frq1og6xg2j7sugx3yobr.png" alt=" " width="800" height="436"></a></p>

<p>Each row represents a word, and the columns (length 512) represent that word as a vector. You might ask: <strong>Why 512 dimensions?</strong></p>

<p>We need high-dimensional space to capture:</p>

<ol>
<li>
<strong>Semantic Meaning</strong>: What the word actually means.</li>
<li>
<strong>Syntactic Role</strong>: Is it a noun, verb, or adjective?</li>
<li>
<strong>Relationships</strong>: How it relates to other words (e.g., "King" vs "Queen").</li>
<li>
<strong>Context</strong>: Multiple contexts the word can appear in.</li>
</ol>
<h3>
  
  
  Input Embedding
</h3>

<p>Computers don't understand strings; they understand numbers. We take our original sentence:</p>

<p><strong>"Your cat is a lovely cat"</strong></p>

<p>First, we map these to Input IDs (their position in the vocabulary):<br>
We then map these IDs into a vector of size 512. Note that these vectors are <strong>not fixed</strong>; they are learned parameters that change during training to better represent the word's meaning.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2jatm5ou1q877p6xv9wr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2jatm5ou1q877p6xv9wr.png" alt=" " width="800" height="436"></a></p>
<h2>
  
  
  3. Positional Encoding
</h2>

<p>Since the Transformer processes words in parallel (not sequentially like an RNN), it has no inherent concept of "order." It doesn't know that "Your" comes before "cat." We must inject this information manually using <strong>Positional Encodings</strong>.</p>

<p>We want the model to treat words that appear close to each other as "close" mathematically. To do this, we use trigonometric functions because they naturally represent continuous patterns that the model can easily learn to extrapolate.</p>

<p>We add this positional vector to our embedding vector. The formula used in the paper is:</p>

<ul>
<li>For even positions (
<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">2i2i</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord">2</span><span class="mord mathnormal">i</span></span></span></span></span>
</div>
):</li>
</ul>


<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">PE(pos,2i)=sin⁡(pos100002i/dmodel)
PE(pos, 2i) = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord mathnormal">PE</span><span class="mopen">(</span><span class="mord mathnormal">p</span><span class="mord mathnormal">os</span><span class="mpunct">,</span><span class="mspace"></span><span class="mord">2</span><span class="mord mathnormal">i</span><span class="mclose">)</span><span class="mspace"></span><span class="mrel">=</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mop">sin</span><span class="mspace"></span><span class="minner"><span class="mopen delimcenter"><span class="delimsizing size2">(</span></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord">1000</span><span class="mord"><span class="mord">0</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">2</span><span class="mord mathnormal mtight">i</span><span class="mord mtight">/</span><span class="mord mtight"><span class="mord mathnormal mtight">d</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size3 size1 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">m</span><span class="mord mathnormal mtight">o</span><span class="mord mathnormal mtight">d</span><span class="mord mathnormal mtight">e</span><span class="mord mathnormal mtight">l</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord mathnormal">p</span><span class="mord mathnormal">os</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mclose delimcenter"><span class="delimsizing size2">)</span></span></span></span></span></span></span>
</div>


<ul>
<li>For odd positions (
<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">2i+12i+1</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord">2</span><span class="mord mathnormal">i</span><span class="mspace"></span><span class="mbin">+</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord">1</span></span></span></span></span>
</div>
):</li>
</ul>


<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">PE(pos,2i+1)=cos⁡(pos100002i/dmodel)
PE(pos, 2i+1) = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord mathnormal">PE</span><span class="mopen">(</span><span class="mord mathnormal">p</span><span class="mord mathnormal">os</span><span class="mpunct">,</span><span class="mspace"></span><span class="mord">2</span><span class="mord mathnormal">i</span><span class="mspace"></span><span class="mbin">+</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord">1</span><span class="mclose">)</span><span class="mspace"></span><span class="mrel">=</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mop">cos</span><span class="mspace"></span><span class="minner"><span class="mopen delimcenter"><span class="delimsizing size2">(</span></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord">1000</span><span class="mord"><span class="mord">0</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">2</span><span class="mord mathnormal mtight">i</span><span class="mord mtight">/</span><span class="mord mtight"><span class="mord mathnormal mtight">d</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size3 size1 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">m</span><span class="mord mathnormal mtight">o</span><span class="mord mathnormal mtight">d</span><span class="mord mathnormal mtight">e</span><span class="mord mathnormal mtight">l</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord mathnormal">p</span><span class="mord mathnormal">os</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mclose delimcenter"><span class="delimsizing size2">)</span></span></span></span></span></span></span>
</div>


<p>This ensures that every position has a unique encoding that is consistent across training and inference.</p>

<h2>
  
  
  4. Self-Attention: The Core Mechanism
</h2>

<p>This is the "magic" of the architecture. Self-attention allows the model to relate words to each other within the same sentence. It determines how much "focus" the word "lovely" should have on the word "cat."</p>

<p>The formula for Scaled Dot-Product Attention is:</p>


<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">Attention(Q,K,V)=softmax(QKTdk)V
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord text"><span class="mord">Attention</span></span><span class="mopen">(</span><span class="mord mathnormal">Q</span><span class="mpunct">,</span><span class="mspace"></span><span class="mord mathnormal">K</span><span class="mpunct">,</span><span class="mspace"></span><span class="mord mathnormal">V</span><span class="mclose">)</span><span class="mspace"></span><span class="mrel">=</span><span class="mspace"></span></span><span class="base"><span class="strut"></span><span class="mord text"><span class="mord">softmax</span></span><span class="mspace"></span><span class="minner"><span class="mopen delimcenter"><span class="delimsizing size3">(</span></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord sqrt"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span class="svg-align"><span class="pstrut"></span><span class="mord"><span class="mord"><span class="mord mathnormal">d</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span></span><span><span class="pstrut"></span><span class="hide-tail"></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord mathnormal">Q</span><span class="mord"><span class="mord mathnormal">K</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">T</span></span></span></span></span></span></span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mclose delimcenter"><span class="delimsizing size3">)</span></span></span><span class="mspace"></span><span class="mord mathnormal">V</span></span></span></span></span>
</div>


<p>Where:</p>

<ul>
<li>
<strong>Q</strong> (Query): What I am looking for.</li>
<li>
<strong>K</strong> (Key): What I contain.</li>
<li>
<strong>V</strong> (Value): The actual content I will pass along.</li>
</ul>

<h3>
  
  
  The Matrix Math
</h3>

<p>For a sequence length of 6 and dimension 512:</p>

<ol>
<li>We multiply <strong>Q</strong> (6 × 512) by <strong>K^T</strong> (512 × 6).</li>
<li>This results in a (6 × 6) matrix.</li>
<li>We apply the Softmax function. This turns the scores into probabilities (summing up to 1).</li>
</ol>

<p>This (6 × 6) matrix captures the interaction between every word and every other word. When we multiply this by <strong>V</strong>, we get a weighted sum of the values, where the weights are determined by the compatibility of the Query and Key.</p>

<p><strong>Key Benefits of Self-Attention:</strong></p>

<ul>
<li>
<strong>Permutation Invariant</strong>: It treats the sequence as a set of relationships rather than a strict list.</li>
<li>
<strong>Parameter Efficiency</strong>: Pure self-attention requires no learnable parameters (though the linear layers surrounding it do).</li>
<li>
<strong>Long-range Dependencies</strong>: Words at the start of a sentence can attend to words at the end just as easily as adjacent words.</li>
</ul>

<p><strong>Key Benefits of Self-Attention:</strong></p>

<ul>
<li>
<strong>Permutation Invariant</strong>: It treats the sequence as a set of relationships rather than a strict list.</li>
<li>
<strong>Parameter Efficiency</strong>: Pure self-attention requires no learnable parameters (though the linear layers surrounding it do).</li>
<li>
<strong>Long-range Dependencies</strong>: Words at the start of a sentence can attend to words at the end just as easily as adjacent words.</li>
</ul>

<h2>
  
  
  Summary &amp; Looking Ahead
</h2>

<p>We have successfully moved away from the sequential limitations of RNNs and embraced the parallel nature of Transformers. We've learned how to convert text into meaningful vector spaces, inject order using positional encoding, and, most importantly, derive the mathematical foundation of how words "pay attention" to each other using Queries, Keys, and Values.</p>

<p><strong>But there is a catch.</strong></p>

<p>The mechanism we just described, a single pass of 
</p>
<div class="katex-element">
  <span class="katex-display"><span class="katex"><span class="katex-mathml">softmax(QKTdk)V\text{softmax}(\frac{QK^T}{\sqrt{d_k}})V</span><span class="katex-html"><span class="base"><span class="strut"></span><span class="mord text"><span class="mord">softmax</span></span><span class="mopen">(</span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="mord"><span class="mord sqrt"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span class="svg-align"><span class="pstrut"></span><span class="mord"><span class="mord"><span class="mord mathnormal">d</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">k</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span></span><span><span class="pstrut"></span><span class="hide-tail"></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span></span></span><span><span class="pstrut"></span><span class="frac-line"></span></span><span><span class="pstrut"></span><span class="mord"><span class="mord mathnormal">Q</span><span class="mord"><span class="mord mathnormal">K</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist"><span><span class="pstrut"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">T</span></span></span></span></span></span></span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mclose">)</span><span class="mord mathnormal">V</span></span></span></span></span>
</div>
, is only capable of focusing on one type of relationship at a time. For example, it might focus heavily on syntactic relationships (such as subject-verb agreement) but completely miss semantic nuances (like sarcasm or references).

<p>Real-world language is too complex for a single "gaze." To build a model like ChatGPT, we need it to look at the sentence through multiple lenses simultaneously.</p>

<p>In Part 2, we will take the self-attention mechanism and clone it, creating <strong>Multi-Head Attention</strong>. We will then see how these attention scores are processed through <strong>Feed-Forward Networks</strong> to finally construct the complete Transformer block.</p>

<p>Stay tuned.</p>

