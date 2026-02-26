---
Title: Understanding LSTMs – Part 5: The Input Gate Explained
Description: 
Author: Rijul Rajesh
Date: 2026-02-26T21:33:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>In the <a href="https://dev.to/rijultp/understanding-lstms-part-4-how-lstm-decides-what-to-forget-c9o">previous article</a>, we have just went through the 2nd and 3rd components in LSTM, we will understand it further in this article.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmzdh24cvml0n0grnf4s4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmzdh24cvml0n0grnf4s4.png" alt=" " width="800" height="533"></a></p>

<p>Starting with the block furthest to the right, we multiply the short-term memory and the input by their respective weights:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flfknbo9p13cm9v7cllmm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flfknbo9p13cm9v7cllmm.png" alt=" " width="740" height="86"></a></p>

<p>This value, 2.03, becomes the input to the tanh activation function.</p>

<p>Now we plug 2.03 into the tanh function and obtain approximately 0.97.</p>

<p>The tanh activation function maps any input to a value between −1 and 1.</p>

<p>When the input to the LSTM is 1, after calculating the x-axis value, the tanh activation function produces an output close to 1.</p>

<p>In contrast, if the input to the LSTM were −10, then after calculating the x-axis value, the tanh activation function would produce an output close to −1.</p>

<p>So, based on the short-term memory and the input, we now have a potential memory of 0.97.</p>

<p>Next, the LSTM must decide how much of this potential memory to retain. This is done using the same method as before.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftp0708i958pcc5yzq25i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftp0708i958pcc5yzq25i.png" alt=" " width="582" height="77"></a></p>

<p>This value, 4.27, is the x-axis input to the sigmoid activation function.</p>

<p>When we plug this value into the sigmoid function, we obtain a y-axis value of approximately 1.0.</p>

<p>This means the entire potential long-term memory is retained, because multiplying it by 1 does not change it.</p>

<p>If the input were −10, then the percentage of potential memory to retain would be 0.</p>

<p>So, nothing would be added to the long-term memory.</p>

<p>Now, going back, we add 0.97 to the existing long-term memory:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F97afhm3mua87hyobxd2v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F97afhm3mua87hyobxd2v.png" alt=" " width="309" height="84"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjg5m00f02utnup7o3rpb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjg5m00f02utnup7o3rpb.png" alt=" " width="800" height="528"></a></p>

<p>This part is called the input gate.</p>

<p>Now we are ready to discuss the final stage of the LSTM, which we will explore in the next article.</p>

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

