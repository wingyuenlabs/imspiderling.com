---
Title: Building with AI: A Deep Dive Into My Gemini-Powered Projects
Description: 
Author: Michael Odhiambo
Date: 2025-12-07T21:38:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the year, I’ve been experimenting with AI APIs, search engines, and creative automation — using Gemini, Hugging Face, GitHub integrations, and Google Programmable Search to build practical apps that solve real problems.<br>
What started as small experiments grew into a collection of fully functional tools: storytelling apps, retrieval-only assistants, virtual try-on platforms, and multi-category AI quiz systems.</p>

<p>This is a breakdown of each project, the problems they solve, and the technologies behind them.</p>




<h2>
  
  
  <strong>1. StoryLoom</strong>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F245c0u3p8ebcfx0n225b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F245c0u3p8ebcfx0n225b.png" alt="Story loom page" width="800" height="556"></a></p>

<p><strong>StoryLoom</strong> is a storytelling platform designed for kids, teens, and anyone who loves creative narratives. The user chooses a theme, age range, and prompt, and the system generates:</p>

<ul>
<li>A complete, original story</li>
<li>A matching AI-generated cover image</li>
<li>A comprehension quiz</li>
<li>Vocabulary flashcards</li>
<li>Translations into multiple languages</li>
<li>A read-aloud mode with customizable voices</li>
</ul>

<h3>
  
  
  <strong>Tech Behind the App</strong>
</h3>

<p><strong>Frontend:</strong> React<br>
<strong>Backend:</strong> Python Flask<br>
<strong>AI Providers:</strong></p>

<ul>
<li>
<strong>Primary:</strong> Gemini 2.0 Flash</li>
<li>
<strong>Fallback:</strong> Hugging Face (Mixtral-8x7B)</li>
<li>
<strong>Images:</strong> Hugging Face Image API</li>
</ul>

<p>The core idea is reliability. If Gemini is available, the app uses it. If Gemini fails or isn’t configured, it falls back automatically to Hugging Face — no interruptions, no manual switching.</p>

<p><strong>Why fallback matters:</strong></p>

<ul>
<li>Works even with free-tier API limits</li>
<li>Eliminates downtime</li>
<li>Avoids single-provider dependency</li>
</ul>

<p>This project taught me a lot about multi-provider orchestration, structured prompting, and synchronizing text + image pipelines.</p>




<h2>
  
  
  <strong>2. QuizBaze</strong>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsdb502de7x8jvieza131.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsdb502de7x8jvieza131.png" alt="Quiz baze landing page" width="800" height="465"></a></p>

<p><strong>QuizBaze</strong> was born from exam week pressure, trying to answer Quizlet flashcard questions quickly without flipping through dozens of sets manually.</p>

<p>This app doesn’t “generate” answers.<br>
It <strong>only retrieves</strong> from Quizlet flashcards, guaranteeing zero hallucination.</p>

<h3>
  
  
  <strong>How It Works</strong>
</h3>

<ol>
<li>User enters a flashcard question</li>
<li>The system uses <strong>Google Custom Search Engine</strong> to find matching Quizlet pages</li>
<li>It scrapes only the answer portion</li>
<li>If no match exists, it returns “Not found.”</li>
</ol>

<h3>
  
  
  <strong>What Makes It Unique</strong>
</h3>

<ul>
<li>🔒 Retrieval-locked (no AI guessing)</li>
<li>⚡ Fast, direct answers</li>
<li>🎯 Exact matches from Quizlet only</li>
<li>🚫 Zero hallucinations</li>
</ul>

<p>This project highlights how powerful “search + extraction” systems can be even without model training.</p>




<h2>
  
  
  <strong>3. StyleAI Studio</strong>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu9gdwcsfluoydincn3em.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu9gdwcsfluoydincn3em.png" alt="Style AI landing page" width="800" height="467"></a></p>

<p>This is the most ambitious app in the list.<br>
StyleAI Studio reimagines how users interact with fashion — for both individuals and businesses.</p>

<h3>
  
  
  <strong>For Personal Users</strong>
</h3>

<ul>
<li>Upload a photo of yourself</li>
<li>Upload clothing items</li>
<li>Generate a realistic try-on image</li>
<li>Manage your digital wardrobe</li>
<li>Auto-save your outfit history to your Google Drive</li>
</ul>

<h3>
  
  
  <strong>For Business Users</strong>
</h3>

<ul>
<li>Upload mannequins + product photos</li>
<li>Add a description</li>
<li>Generate AI-powered product catalog images</li>
<li>Organize all business assets in a dedicated Google Drive folder</li>
</ul>

<h3>
  
  
  <strong>Tech Used</strong>
</h3>

<ul>
<li>Gemini (for generation + transformations)</li>
<li>Google Drive API (personal + business storage)</li>
<li>Node/React (presentation layer)</li>
<li>Python backend (for image workflows)</li>
</ul>

<p>This project pushed the limits of file handling, secure user asset storage, and realistic image manipulation.</p>




<h2>
  
  
  <strong>4. QuizMate</strong>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F42izewi67pjhaa8ymn8x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F42izewi67pjhaa8ymn8x.png" alt="quizmate landing page" width="800" height="470"></a></p>

<p><strong>QuizMate</strong> is a study companion that mixes retrieval, summarization, question generation, and flashcards into a single workflow. It turns any piece of content into structured learning material.</p>

<p>It uses:</p>

<ul>
<li>Gemini for summarization + quiz generation</li>
<li>Optional search integrations for external material</li>
</ul>

<p>A more flexible, personalized version of Quizlet + ChatGPT in one app.</p>




<h2>
  
  
  <strong>5. Quiz Master</strong>
</h2>

<p>Quiz Master is an interactive quiz platform built with React (frontend) and Python FastAPI (backend). It combines traditional trivia APIs with Gemini-generated questions.</p>

<h3>
  
  
  <strong>Features</strong>
</h3>

<ul>
<li>
<strong>Multiple categories:</strong> animals, sports, food, geography, trivia, etc.</li>
<li>
<p><strong>Two modes:</strong></p>

<ul>
<li>
<em>Facts Mode:</em> standard Q&amp;A</li>
<li>
<em>Identify Mode:</em> image-based guessing</li>
</ul>


</li>

<li><p><strong>AI-enhanced questions:</strong> Gemini generates variations, difficulty levels, and new questions</p></li>

<li><p><strong>Real images:</strong> pulled from public APIs (e.g., TheMealDB, country flag APIs, animal APIs)</p></li>

<li><p><strong>Scoring, review, and clean UI</strong></p></li>

</ul>

<h3>
  
  
  <strong>Tech Stack</strong>
</h3>

<ul>
<li>React</li>
<li>FastAPI</li>
<li>Gemini API</li>
<li>Third-party data sources</li>
</ul>

<p>This app demonstrates how AI + external APIs + structured content can blend into a smooth quiz experience.</p>




<h2>
  
  
  <strong>Why I Built These Projects</strong>
</h2>

<p>Each app solved a real need:</p>

<ul>
<li>
<strong>StoryLoom</strong> → creative storytelling and learning tools</li>
<li>
<strong>QuizBaze</strong> → fast retrieval during tight deadlines</li>
<li>
<strong>StyleAI Studio</strong> → a full AI-powered fashion workflow</li>
<li>
<strong>QuizMate</strong> → personalized study automation</li>
<li>
<strong>Quiz Master</strong> → fun but educational quiz generation</li>
</ul>

<p>They also helped me explore:</p>

<ul>
<li>Multi-provider AI systems</li>
<li>Retrieval-based AI (RAG without embedding models)</li>
<li>Vision + text generation</li>
<li>Search engine automation</li>
<li>Secure file storage with Google Drive</li>
<li>Python/Flask, FastAPI, React full-stack architectures</li>
</ul>




