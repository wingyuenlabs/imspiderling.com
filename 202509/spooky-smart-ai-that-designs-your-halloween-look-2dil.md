---
Title: Spooky Smart AI That Designs Your Halloween Look
Description: 
Author: robot254
Date: 2025-09-14T21:51:31.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwo12k19n2mcf7ozzpsja.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwo12k19n2mcf7ozzpsja.png" alt=" " width="800" height="445"></a></p>

<p><em>This is a submission for the <a href="https://dev.to/challenges/google-ai-studio-2025-09-03">Google AI Studio Multimodal Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>I built the AI Halloween Costume Generator, a comprehensive web application designed to solve the age-old problem of choosing the perfect Halloween costume. It acts as a creative partner, transforming vague ideas or even simple images into complete, ready-to-make costume guides.</p>

<h3>
  
  
  The app provides a multi-faceted approach to inspiration:
</h3>

<p>Search: Users can type in any theme or idea (e.g., "costumes for my dog," "spooky sci-fi ideas") and receive five distinct, fully-detailed costume concepts to compare and choose from.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3lx6ct9vlucwodz6brp7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3lx6ct9vlucwodz6brp7.png" alt="Image Ideas" width="765" height="635"></a></p>

<p>Generate from Image: Users can upload a photo of an object, a person, or a pet, and the AI will generate a unique costume concept based on the visual input.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp7zbohg0g59do2nypyfs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp7zbohg0g59do2nypyfs.png" alt="Generate from Images" width="694" height="516"></a></p>

<p>Surprise Me!: For users who are completely stumped, a "Surprise Me!" button generates a totally random and creative idea out of the blue.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg13kbvxv766enhw1h3i3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg13kbvxv766enhw1h3i3.png" alt=" " width="639" height="626"></a></p>

<p>Once an idea is selected, the app provides a complete DIY guide, including a list of materials, an estimated cost and difficulty level, and most importantly, detailed step-by-step instructions with custom-generated, additive illustrations that show the costume coming together.</p>

<h2>
  
  
  Demo
</h2>

<p><strong>To try out:</strong> <a href="https://halloween-costume-generator-610288702971.us-west1.run.app/" rel="noopener noreferrer">Cloud Run Demo link</a></p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/t5FiUM5jlTc">
</iframe>
</p>

<h2>
  
  
  How I Used Google AI Studio
</h2>

<p>I leveraged a suite of multimodal models from the Gemini API to power the application's core features, orchestrating them to create a seamless user experience.</p>

<p>gemini-2.5-flash: This was the primary model for all text and structured data generation. I used it with a strictly defined responseSchema to ensure the AI's output was always in a predictable JSON format. This model was responsible for:</p>

<p>Generating the costume's name, description, materials, and detailed text instructions.</p>

<p>Powering the search feature by creating five distinct costume concepts from a single prompt.</p>

<p>Handling the conversational "Refine" feature, where it would modify a costume based on follow-up user input.</p>

<p><strong>imagen-4.0-generate-001:</strong> This powerful image generation model was used to create the crucial first image for each set of instructions, establishing the visual foundation for the step-by-step guide.</p>

<p><strong>gemini-2.5-flash-image-preview:</strong> This versatile image editing model was the key to creating the app's most unique feature. It was used to generate all subsequent instruction images by taking the previous step's image as input and adding the new details described in the current step's text.</p>

<h2>
  
  
  Multimodal Features
</h2>

<p>The app is built around two core multimodal functionalities that create a rich and intuitive user experience.</p>

<p><strong>Vision Understanding: Image to Costume Idea</strong></p>

<p>The ability for a user to upload an image and receive a relevant costume idea is a powerful multimodal feature. It goes beyond simple text prompts by allowing for visual context. A user can upload a picture of their pet, a favorite object, or a friend, and the AI can creatively interpret that visual data to generate a highly personalized and often unexpected costume concept. This makes the brainstorming process more personal and engaging.</p>

<p><strong>Additive Image Generation: A Cohesive Visual Guide</strong></p>

<p>The app's standout feature is its ability to create a set of instruction images that build upon one another. Instead of generating a new, disconnected image for each step, the system uses an iterative, multimodal process:</p>

<ul>
<li>Step 1: Generate a base image from a text prompt.</li>
<li>Step 2+: Feed the image from the previous step plus the text for the current step into the image editing model (gemini-2.5-flash-image-preview).</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fda4cnmky8ct892vvv6e8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fda4cnmky8ct892vvv6e8.png" alt="Image steps" width="625" height="637"></a></p>

<p>This creates a coherent visual narrative, allowing the user to literally watch the costume come together from one image to the next. This significantly enhances the user experience by making the instructions far easier to understand and follow compared to a series of isolated diagrams. It transforms the app from a simple idea generator into a true step-by-step visual crafting guide.</p>

