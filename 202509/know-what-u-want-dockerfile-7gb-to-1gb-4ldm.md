---
Title: Know what you want - Dockerfile 7GB to 1GB
Description: 
Author: Mohamed Roshan
Date: 2025-09-15T21:25:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>Encountered the classic big image issue when I built a docker image with multi staging.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg7o5tp08w09hy0gc0qtk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg7o5tp08w09hy0gc0qtk.png" alt="dockerfile" width="800" height="422"></a></p>

<p>Coming to what i was trying to achieve, I had this idea of hosting openai CLIP model for fun on gke so for that i was containerizing the following dependencies and app file, my requirement initially was<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>torch
torchvision
ftfy
regex
tqdm
fastapi
uvicorn
pillow
numpy&lt;2
python-multipart

</code></pre>

</div>



<p>A lil bit of context here, the idea is to have the CLIP model that runs on CPU and doesn't need any CUDA/GPU support. So this <code>requirement.txt</code> installs the torch related packages with default GPU/CUDA which is not required and this made the image size to be 7gb </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flbq0j07vidkshv3sjnpd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flbq0j07vidkshv3sjnpd.png" alt="clip image" width="800" height="27"></a></p>

<p>So like any other person I digged into it cos i was expecting the size to be around 2gb max. I found about the unwanted CUDA/GPU stuff being added while installing the torch packages. </p>

<p>So I updated the <code>requirement.txt</code> and mentioned that I just want CPU support and that just did it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>torch==2.2.0+cpu
torchvision==0.17.0+cpu
-f https://download.pytorch.org/whl/cpu/torch_stable.html
ftfy
regex
tqdm
fastapi
uvicorn
pillow
numpy&lt;2
python-multipart
</code></pre>

</div>



<p>now the image size got drastically reduced to ~1GB from `~7GB</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbx6unhh7deap9s14rto1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbx6unhh7deap9s14rto1.png" alt="clip image optimised" width="762" height="97"></a></p>

<p>and if you want you can further drill it down by removing the tests and example directories from the site-package and that will probably save u 50mb ~ 60mb</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2gzb6eb296xhxxo3u0za.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2gzb6eb296xhxxo3u0za.png" alt="dockerfile optimised" width="800" height="335"></a></p>

<p>and now the size went from 1.07GB to 1.01GB</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzve7yknl2zabzmiw5kmm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzve7yknl2zabzmiw5kmm.png" alt="clip image light" width="800" height="67"></a></p>

<p>In fact you can go aggressive and might even reduce more if you're individually copying the packages from <code>site-packages</code></p>

