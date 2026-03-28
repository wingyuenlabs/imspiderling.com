---
Title: Understanding Attention Mechanisms – Part 3: From Cosine Similarity to Dot Product
Description: 
Author: Rijul Rajesh
Date: 2026-03-28T21:55:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>In the <a href="https://dev.to/rijultp/understanding-attention-mechanisms-part-2-comparing-encoder-and-decoder-outputs-22ch">previous article</a>, we explored the comparison between encoder and decoder outputs. In this article, we will be checking the math on how the calculation is done, and how it can be further simplified.</p>

<p>The output values for the <strong>two LSTM cells in the encoder</strong> for the word <strong>"Let’s"</strong> are <strong>-0.76</strong> and <strong>0.75</strong>.</p>

<p>The output values from the <strong>two LSTM cells in the decoder</strong> for the <strong><code>&lt;EOS&gt;</code> token</strong> are <strong>0.91</strong> and <strong>0.38</strong>.</p>

<p>We can represent this as:</p>

<p><strong>A = Encoder</strong><br>
<strong>B = Decoder</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Cell #1     Cell #2
-0.76       0.75
 0.91       0.38
</code></pre>

</div>



<p>Now, we plug these values into the <strong>cosine similarity equation</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4scsocrbsp48d6cdoq61.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4scsocrbsp48d6cdoq61.png" alt=" " width="349" height="228"></a></p>

<p>This gives us a result of <strong>-0.39</strong>.</p>

<p>To simplify this further, a common approach is to <strong>compute only the numerator</strong>.</p>

<p>The denominator mainly <strong>scales the value between -1 and 1</strong>, so in some cases, we can ignore it for simplicity.</p>

<p>Since we are dealing with a <strong>fixed number of cells</strong>, this simplification works well. This is also known as the <strong>dot product</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx6oaj38y7mgeolaw27bz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx6oaj38y7mgeolaw27bz.png" alt=" " width="244" height="108"></a></p>

<p>When we calculate only the dot product, we get:</p>

<p><strong>(-0.76 × 0.91) + (0.75 × 0.38) = -0.41</strong></p>

<p>We will <strong>explore this further in the next article</strong>.</p>




<p><strong>Looking for an easier way to install tools, libraries, or entire repositories?</strong><br>
Try <strong>Installerpedia</strong>: a <strong>community-driven, structured installation platform</strong> that lets you install almost anything with <strong>minimal hassle</strong> and <strong>clear, reliable guidance</strong>.</p>

<p>Just run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ipm <span class="nb">install </span>repo-name
</code></pre>

</div>



<p>… and you’re done! 🚀</p>

<p><a href="https://hexmos.com/freedevtools/installerpedia" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm2s3mzj8pfcq94a1y4at.png" alt="Installerpedia Screenshot" width="800" height="336"></a></p>

<p>🔗 <a href="https://hexmos.com/freedevtools/installerpedia/" rel="noopener noreferrer"><strong>Explore Installerpedia here</strong></a></p>

