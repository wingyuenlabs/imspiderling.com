---
Title: Remove Image Backgrounds in the Browser - No Backend Needed 🚀
Description: 
Author: Maher Gallaoui
Date: 2025-11-28T21:16:47.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Removing backgrounds from images is a common need. but many solutions require uploading your image to a server. What if you could run it fully in the browser, with no backend, no API keys, and full privacy? That’s exactly what this project does.</p>

<p>Using <strong>Transformers.js</strong>, I built a background removal web app that runs client-side. The model runs locally in your browser, and the processed image never leaves your device.</p>

<p>In this post, I walk you through it step by step, from project setup to UI, and finally to background-removal logic.</p>

<p>💻 You can access the full source code on GitHub and try it yourself: <a href="https://github.com/gallaouim/bg-remover" rel="noopener noreferrer">bg-remover Repository</a></p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>Node.js (v16+ recommended)</li>
<li>npm or yarn</li>
<li>A modern browser (for WASM or WebGPU — depending on your config)</li>
</ul>

<h2>
  
  
  Initialise the project with Vite
</h2>

<p>To get started quickly, I used Vite to scaffold a minimal web project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm create vite@latest bg-remover -- --template vanilla
cd bg-remover
npm install
</code></pre>

</div>



<p>I’ll assume vanilla JS, HTML and CSS for simplicity.</p>

<p>After installing the dependencies, you can now start the server with<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm run dev
</code></pre>

</div>



<h2>
  
  
  2. Install Transformers.js
</h2>

<p>Transformers.js brings the power of Hugging Face,running segmentation, classification directly in the browser using ONNX runtime under the hood. So you have to add the NPM module with<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm install @huggingface/transformers

</code></pre>

</div>



<h2>
  
  
  3. Build the HTML &amp; UI
</h2>

<p>We want get into the details Here as it's actually a very simple UI that you can see here , you will find below the links to the html and style files</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftmo4hwjlftru0ydvf5y8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftmo4hwjlftru0ydvf5y8.png" alt=" " width="800" height="596"></a></p>

<p>HTML : <a href="https://github.com/gallaouim/bg-remover/blob/main/index.html" rel="noopener noreferrer">index.html</a> <br>
Style: <a href="https://github.com/gallaouim/bg-remover/blob/main/src/style.css" rel="noopener noreferrer">Style.css</a></p>
<h2>
  
  
  4. Main Logic: Load model, process image, remove background
</h2>
<h3>
  
  
  Step 1: Import Transformers.js modules
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo781c3xsylltxcv6us6w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo781c3xsylltxcv6us6w.png" alt=" " width="800" height="42"></a></p>

<ul>
<li>
<code>AutoProcessor</code> → handles preprocessing of your input image to the format the model expects.</li>
<li>
<code>AutoModel</code> → loads the actual model that will perform the task (background removal / segmentation).
Basically, the processor prepares the image, the model predicts the foreground mask.</li>
</ul>
<h3>
  
  
  Step 2: Initialize the Model and Processor
</h3>

<p>In this step, we load the pretrained background-removal model and its processor. Both are configurable, which means you can tweak preprocessing settings, input size, normalization, etc.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7fxk44oez350s8c76hs1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7fxk44oez350s8c76hs1.png" alt=" " width="800" height="458"></a></p>

<ul>
<li>
<code>model_type</code>: "custom" → allows you to load models that don’t include a standard config.json.</li>
<li>
<code>do_normalize / image_mean / image_std</code> → normalize pixel values for better model accuracy.</li>
<li>
<code>do_resize / size</code> → resize input images to the model’s expected dimensions.</li>
<li>
<code>do_rescale / rescale_factor</code> → scale pixel values from 0–255 to 0–1.</li>
<li>
<code>resample</code> → resizing algorithm</li>
</ul>

<p>Key idea: By changing these config values, you can adapt the preprocessing to different models or custom training data without changing the model code itself</p>
<h3>
  
  
  Step 3: Listen for image uploads
</h3>

<p>We attach a listener to the file input element <code>(&lt;input type="file"&gt;)</code>that triggers when the user selects an image. An Image object is created from the file, using <code>URL.createObjectURL(file)</code> to generate a temporary local URL without uploading anything. Once the image is loaded <code>(img.onload)</code>, we call <code>predict()</code> with the image to perform background removal.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fib9h6mm1ljjeznyp7678.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fib9h6mm1ljjeznyp7678.png" alt=" " width="800" height="427"></a></p>
<h3>
  
  
  Step 4: Remove the background
</h3>

<p>Once the user uploads an image, we can preprocess it, run the model, and apply the background mask:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0nre0o0qr65ejmmgtd80.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0nre0o0qr65ejmmgtd80.png" alt=" " width="800" height="761"></a></p>

<ol>
<li>Preprocessing: Converts the uploaded image into the format expected by the model.</li>
<li>Prediction: The model outputs a mask indicating the foreground.</li>
<li>Post-processing: The mask is resized to the original image size and applied as an alpha channel to remove the background.</li>
<li>Display: The final image is drawn on a <code>&lt;canvas&gt;</code>for the user to see.</li>
</ol>
<h3>
  
  
  5. Try It Yourself
</h3>

<p>Clone the repository and run it locally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git clone https://github.com/gallaouim/bg-remover.git
cd bg-remover
npm install
npm run dev
</code></pre>

</div>



<p>Upload an image and watch the background disappear directly in your browser!</p>

<h2>
  
  
  ✅ Summary
</h2>

<p>You’ve learned how to build a fully client-side background removal app using Transformers.js. The key steps are: load a model, preprocess the image, predict a mask, and render the final image on a canvas</p>

