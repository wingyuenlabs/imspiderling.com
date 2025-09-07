---
Title: Look & Learn: a Google AI Multimodal Challenge Entry
Description: 
Author: Frederik 👨‍💻➡️🌐 Creemers
Date: 2025-09-07T21:50:37.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/google-ai-studio-2025-09-03">Google AI Studio Multimodal Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>I created Look &amp; Learn. An app for language learners. The app will generate an image with some interesting scene on it. It will then ask you some questions about that image, in the language you're tryin to learn. For the beginner level, all the questions will be multiple choice. For intermediate and advanced levels, you'll have to type out your answers.</p>

<h2>
  
  
  Demo
</h2>

<p><a href="https://look-learn-174115661584.us-west1.run.app/" rel="noopener noreferrer">Try Look &amp; Learn here</a></p>

<h3>
  
  
  Screenshots:
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foex3r48mj3fr1ljjnab6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foex3r48mj3fr1ljjnab6.png" alt="Screenshot of the Look &amp; Learn start screen. Here, the user has options to select the language they speak, the language they want to learn, and their level of fluency. A button at the bottom lets them start the quiz." width="800" height="745"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1s4wtjd7y2g8nuw4fbcn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1s4wtjd7y2g8nuw4fbcn.png" alt="Image showing a multiple choice question where the user has picked the wrong answer. The wrong answer is highlighted in red, while the correct one is shown in green. At the bottom, an explanation in the user's native language is shown." width="800" height="521"></a></p>

<p>Screenshot from an intermediate level Dutch quiz:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy6uvum1x70c4fc2vcodz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy6uvum1x70c4fc2vcodz.png" alt="Screenshot showing a question where the user had to type in an answer. At the bottom, there's a message indicating that the user has answered correctly, but that there was a grammar issue about verb conjugation, and explaining it." width="800" height="520"></a></p>

<h2>
  
  
  How I Used Google AI Studio
</h2>

<p>I wanted to see how far I could take Google AI studio while touching the code by hand as little as possible. While I'm mostly skeptical of vibe coding, I thought this challenge was an interesting opportunity to give it a try. So, I mostly wrote prompts, and would give the model feedback in natural language.</p>

<h2>
  
  
  Multimodal Features
</h2>

<p>At the start of the quiz, the application either generates an interesting image using Imagen, or takes an existing one from Google Cloud Storage. Currently taking a stored one 80% of the time. It then uses Gemini-2.5-flash, to generate questions about the image, giving it the generated image and a prompt that includes guidelines for the questions, as well as the user's fluency level.</p>

<p>For multiple choice questions, there's a well-defined correct answer, so the application immediately gives the user feedback. For text entry questions, we again give the image to Gemini-2.5-flash, along with the question and the users answer, in order to get an evaluation of the correctness, as well as the user's use of vocabulary and grammar.</p>

<p>I'm also passing the image to Gemini-2.5-flash to generate an alt text for the image. The alt text should contain enough details to answer all the quiz questions. It's provided in the user's native language, so that they still have to go through the exercise of finding the correct answer in the description, and translating it. I've also tried to ensure that all elements that may appear in a different language have a matching <code>lang</code> attribute, so that screen readers read them out correctly.</p>

