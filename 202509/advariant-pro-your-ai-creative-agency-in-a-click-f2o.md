---
Title: AdVariant Pro: Your AI Creative Agency in a Click
Description: 
Author: Debora Fernandes
Date: 2025-09-13T21:19:58.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/google-ai-studio-2025-09-03">Google AI Studio Multimodal Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>built AdVariant Pro, an AI-powered Marketing Strategist that transforms a simple product photo into a complete, ready-to-launch ad campaign. It solves one of the biggest challenges for marketers: creating high-quality, personalized campaign assets quickly and affordably.</p>

<p>Traditionally, creating a single ad requires a photographer, a copywriter, a strategist, and a designer. My applet consolidates these roles, allowing a user to upload a product image, define an audience, and receive a full campaign package in seconds.</p>

<p>With AdVariant Pro, a user can:</p>

<ul>
<li><p>Generate Complete Ad Scenes: Upload a product image (even with a plain background) and the app will intelligently place it into a newly generated, contextually relevant, and photorealistic scene tailored to a specific audience.</p></li>
<li><p>Create Audio Pitches: Instantly generate a compelling 15-second "elevator pitch" for the campaign, voiced by a high-quality AI from ElevenLabs, ready for social media videos.</p></li>
<li><p>Receive AI-Powered Creative Assistance: Get instant suggestions for catchy slogans or expand a brief audience description into a detailed marketing persona, all powered by Gemini.</p></li>
<li><p>Achieve Strategic Composition: The AI acts as an Art Director, analyzing the visual composition to strategically position the slogan for maximum impact and legibility.</p></li>
</ul>

<h2>
  
  
  Demo
</h2>

<p>You can try the live applet here:<br>
<a href="https://advariant-pro-gerador-de-campanhas-de-an-ncios-790832384921.us-west1.run.app/" rel="noopener noreferrer">https://advariant-pro-gerador-de-campanhas-de-an-ncios-790832384921.us-west1.run.app/</a></p>

<p>Here is a quick overview of the workflow:</p>

<ol>
<li>Upload Your Product &amp; Define Your Audience
The user starts by uploading their product image and describing the target customer. The interface is mobile-first and multilíngue (EN/PT).</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz3wv277mlaiwpbkz7w0h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz3wv277mlaiwpbkz7w0h.png" alt=" " width="505" height="822"></a></p>

<ol>
<li><p>Generate Your Campaign (Visual + Audio)<br>
With a single click, AdVariant Pro generates a complete ad scene and a 15-second audio pitch.</p></li>
<li><p>Hyper-Personalized Results<br>
This shows the power of the tool. Using the exact same uploaded product photo, AdVariant Pro created two completely different campaign packages (visuals and audio pitches) for two unique audiences.</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa9hdgxi5lp2zqi9v6st7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa9hdgxi5lp2zqi9v6st7.png" alt=" " width="800" height="340"></a></p>

<p>Video Demo: <a href="https://youtu.be/P2m-3_arVkU" rel="noopener noreferrer">https://youtu.be/P2m-3_arVkU</a><br>
<iframe width="710" height="399" src="https://www.youtube.com/embed/P2m-3_arVkU">
</iframe>
</p>

<h2>
  
  
  How I Used Google AI Studio
</h2>

<p>Google AI Studio was the command center for this project, enabling a seamless workflow from prompt engineering to final deployment.</p>

<ul>
<li><p>Rapid Prototyping: I used AI Studio to meticulously craft and test the multi-step prompts. The ability to iterate quickly on the "AI Art Director" logic was essential to achieving the high-quality scene generation and strategic slogan placement.</p></li>
<li><p>Seamless Deployment: The "Deploy to Cloud Run" feature is a game-changer. It handled the entire containerization and deployment pipeline, allowing me to focus on building features rather than managing infrastructure. This was critical for a fast-paced challenge.</p></li>
<li>
<p>Model Selection: I orchestrated multiple Gemini models to act as a specialized creative team:</p>

<ul>
<li>Gemini 1.5 Flash: Used as the central "brain" for all reasoning and text tasks: analyzing the user's text inputs, generating slogan suggestions, writing the script for the audio pitch, and acting as the main Art Director.</li>
<li>gemini-1.5-flash-image-preview: Used as the "AI Photographer" to generate the final, high-quality ad scene based on the detailed instructions from the Art Director.</li>
</ul>


</li>

</ul>

<h2>
  
  
  Multimodal Features
</h2>

<p>AdVariant Pro is built from the ground up on a sophisticated multimodal pipeline that mimics a real-world creative agency workflow.</p>

<ol>
<li>Core Feature: Image-to-Scene Generation (Image + Text → Image)
This is the heart of the applet. The user provides an image of their product, and Gemini performs a complex multimodal task:</li>
</ol>

<ul>
<li><p>It understands the subject of the uploaded image.</p></li>
<li><p>It imagines a new, contextually appropriate background scene based on the text description of the target audience.</p></li>
<li><p>It generates a new, cohesive image, seamlessly integrating the original product while matching lighting, shadows, perspective, and reflections to make it look completely natural.</p></li>
</ul>

<ol>
<li>Full Campaign Pipeline (Image + Text → Text → Audio)
The app creates more than just a picture; it creates a campaign. This involves a multi-step, cross-modality process orchestrated by Gemini:</li>
</ol>

<ul>
<li><p>Step A (Visuals): Gemini first generates the visual ad scene as described above.</p></li>
<li><p>Step B (Scriptwriting): Based on the product, the audience, and the visuals it just created, Gemini then writes a concise and persuasive 15-second "elevator pitch" script.</p></li>
<li><p>Step C (Voice Generation): This script is then passed to the ElevenLabs API to generate a high-quality, ready-to-use audio track.</p></li>
</ul>

<p>This process, which starts with an image and ends with a unique audio clip, demonstrates a powerful and practical application of multimodal AI to solve a real-world marketing challenge. It transforms the user's input into a rich set of campaign assets, not just a single output.</p>

<p>By: Debora Fernandes</p>

