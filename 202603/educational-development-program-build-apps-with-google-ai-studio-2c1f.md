---
Title: Educational Development Program: Build Apps with Google AI Studio
Description: 
Author: Adriana
Date: 2026-03-04T21:53:14.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This post is my entry for the <a href="https://dev.to/deved/build-apps-with-google-ai-studio">Educational Development Program: Build Apps with Google AI Studio</a>.</em></p>

<h2>
  
  
  What I Developed
</h2>

<p>I developed <strong>MockupGen</strong>, an app that transforms amateur product photos into professional-looking mockups for e-commerce. For Shopify store managers or casual sellers looking to sell a product, it's difficult to achieve professional-looking photos without a photo studio. <br>
MockupGen solves this problem: you upload a product photo taken with your phone, select a background style (white studio, lifestyle scene, outdoor, desktop, or gradient), and the app takes care of the rest. I used Gemini to analyze the uploaded image and generate an optimized product description. Then, Image's background swap editing tool replaced it with a professional background, keeping the product intact and transforming everything around it.</p>

<p><strong>Key Instructions for Creating the Application:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
Create an application called MockupGen that transforms amateur product photos into professional e-commerce mockups. The user uploads a product photo and selects a background style from options such as: pristine white studio, lifestyle kitchen scene, outdoor natural environment, minimalist desktop, or gradient background. Use Gemini to analyze the uploaded image and generate an optimized product description. Then, use the background swap feature to replace the product photo's background with the selected professional setting. Display the original and transformed images side-by-side, with a download button for the result and a "Try Another Style" button to regenerate with a different background. The user interface should be clean, modern, and in dark mode.

</code></pre>

</div>



<h2>
  
  
  Relevant Aspects
</h2>

<p>After the initial generation, I iterated on the interface to add more background styles. During iteration, I realized that it didn't accurately reflect the product's proportions.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz1zpekv9db4q433hkjtb.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz1zpekv9db4q433hkjtb.webp" alt="Image1-before nanobanana" width="800" height="536"></a></p>

<p>To correct this, I applied this solution:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
I need to correct the product fidelity in the mockup results. Please make these changes:

1. Automatically detect and preserve the aspect ratio of the original image for the output.

2. When calling the Image API to swap the background, add explicit instructions to the message to preserve the exact colors, proportions, shape, and details of the original product. The product should look identical to the original; only the background should change.

3. Add a note to the message sent to Image: "Do not modify, recolor, reshape, or alter the product in any way. Only replace the surrounding background."

4. Before generating, use Gemini to describe the product in detail (exact colors, materials, dimensions, and distinctive features) and include that description in the Image message as a constraint to maintain product accuracy.

</code></pre>

</div>



<p>But it didn't work; the result remained the same. The solution was to switch to native Gemini for image generation. Gemini 3 Flash has image editing capabilities (Nano Banana) that accepts reference images and tends to better preserve the original.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
Switch from Imagen background swap to using Gemini's native image 
editing. Send the uploaded product photo as a reference image to 
Gemini with a prompt like: "Take this exact product [1] and place 
it in a [selected background style]. Keep the product exactly as 
it appears in the reference image — same colors, same proportions, 
same details. Only change the background environment. The product 
should look photorealistic and properly scaled in the new setting."

</code></pre>

</div>



<p>It´s a huge difference:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjeq3xxa31icl5oc1cbg6.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjeq3xxa31icl5oc1cbg6.PNG" alt="after nano" width="800" height="512"></a></p>

<h2>
  
  
  Demo
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxsqdcd6mqxa013eoqbba.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxsqdcd6mqxa013eoqbba.PNG" alt="ejemplo1" width="800" height="534"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs46d913z3nacdekm3ogj.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs46d913z3nacdekm3ogj.PNG" alt="ejemplo2" width="800" height="405"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F60jx40nrnpx8knpxqx6a.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F60jx40nrnpx8knpxqx6a.PNG" alt="ejemplo3" width="800" height="318"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7gi3sefz5263eckis8ox.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7gi3sefz5263eckis8ox.PNG" alt="ejemplo4" width="800" height="393"></a></p>

<p>👉 <a href="https://aistudio.google.com/apps/c05b2760-5d93-477c-beb1-63421f1952ee?showPreview=true&amp;showAssistant=true&amp;project=gen-lang-client-0109917735" rel="noopener noreferrer">Live MockupGen Test</a></p>

<h2>
  
  
  My Experience
</h2>

<p>The most valuable aspect of this program was understanding how to easily chain multiple AI models together. Getting them to work together taught me a lot about instruction engineering at every stage of the creation process.<br>
What surprised me most was the importance of the initial instruction when using the creation function and how to address functions that don't return a good result. By specifying the user flow (load, select style, transform, compare in parallel), the result was much better. This reinforced something we see everywhere and that is very important to understand: the quality of the instructions directly determines the quality of the results.<br>
An interesting challenge I encountered was product fidelity. My first version used Image background swapping, but it constantly altered the product's colors and proportions. After iterating, I switched to Gemini's native image editing (Nano Banana) with reference images, which preserved the original product much more faithfully. This taught me that choosing the right model for each task is just as important as writing good prompts. Sometimes, the solution isn't better instructions, but a better tool.<br>
A few months ago, I collaborated on the development of an e-commerce store, and I see great potential in tools like this. Product photography is one of the biggest hurdles for small retailers, and being able to prototype a solution like MockupGen in minutes with AI Studio is a glimpse into how we'll develop software in the future.</p>

<p>Thanks for the feedback! 🚀</p>

