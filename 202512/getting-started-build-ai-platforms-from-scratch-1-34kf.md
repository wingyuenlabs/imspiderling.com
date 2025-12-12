---
Title: Getting Started - Build AI Platforms From Scratch #1
Description: 
Author: Nick Goldstein
Date: 2025-12-12T21:20:34.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Build AI Platforms from Scratch
</h1>

<p>Learn to build powerful AI-powered applications and platforms from scratch. This course focuses on design, architecture, and engineering, NOT coding.</p>

<p><strong>📺 <a href="https://nicholasmgoldstein.com/course/build-ai-platforms/module/1" rel="noopener noreferrer">View this module with video &amp; slides</a></strong></p>

<h2>
  
  
  Before We Get Started
</h2>

<p>Please consider the following before proceeding:</p>

<ul>
<li>
<strong>NOTE:</strong> This is NOT a coding course. This series will strictly focus on the design, architecture, and engineering choices involved in building an AI platform/app.</li>
<li>Despite this not being a coding course, we will provide a Github repo with some common coding patterns that you will likely utilize at some point in the process of building an AI platform.</li>
<li>This course is for beginners who want to take advantage of LLMs, and other AI technologies in their projects. The content will be geared more toward ambitious and complex projects that want to make use of things like multi-layered AI architectures, data transformations through LLMs, agents, ambient intelligence, etc., rather than something like a simple chatbot.</li>
</ul>

<h2>
  
  
  Key Terms &amp; Definitions
</h2>

<p>Important concepts and terminology used throughout this course:</p>

<p><strong>LLM (Large Language Model)</strong><br><br>
A type of AI trained on massive amounts of text that can understand and generate human language.</p>

<p><strong>Agents</strong><br><br>
AI systems that can take actions and make decisions autonomously to accomplish goals, not just answer questions.</p>

<p><strong>AI Architecture</strong><br><br>
How you structure and organize the different components of your AI system to work together.</p>

<p><strong>Data Transformation Through LLMs</strong><br><br>
Using AI to convert information from one format to another—like turning messy user input into structured data.</p>

<p><strong>Prompt Engineering</strong><br><br>
The practice of writing instructions to AI models to get reliable, useful outputs.</p>

<p><strong>Tech Stack</strong><br><br>
The collection of technologies and tools you use to build your platform—programming languages, frameworks, databases, etc.</p>

<p><strong>IDE (Integrated Development Environment)</strong><br><br>
The software where you write your code.</p>

<p><strong>MToks (Million Tokens)</strong><br><br>
Standard unit for measuring large-scale API usage. 1 MTok = 1,000,000 tokens.</p>

<p><strong>APIs (Application Programming Interfaces)</strong><br><br>
How your code communicates with external services like AI providers.</p>

<p><strong>Industry Benchmarks</strong><br><br>
Standard metrics and performance expectations for AI systems in production.</p>

<p><strong>Input/Output (in LLM APIs)</strong><br><br>
Input = what you send to the AI. Output = what it generates back. Output typically costs more.</p>

<p><strong>Context Windows</strong><br><br>
The total amount of text an AI model can process at once, including your instructions and its response.</p>

<p><strong>Tokens</strong><br><br>
The basic units AI models use to read and generate text. Roughly 1 token = 0.75 words.</p>

<p><strong>AI Hallucination</strong><br><br>
When AI generates plausible-sounding information that's actually incorrect or made up.</p>

<p><strong>Systems Thinking</strong><br><br>
Designing platforms as interconnected parts rather than isolated features, focusing on how components interact.</p>

<h2>
  
  
  Course Specs
</h2>

<p>Broad agenda of what you'll receive in future modules:</p>

<ul>
<li>A comprehensive understanding of what building an AI platform entails</li>
<li>Actionable steps to develop your own AI-powered application</li>
<li>Real-world examples of AI platforms and how they were built</li>
<li>Prompt Engineering for complex projects (Modularity, Inputs/Outputs, Preventing Bad Behavior, etc.)</li>
<li>Considerations regarding design and user experience</li>
<li>Multi-layered AI workflows and data transformations</li>
<li>Tracking AI usage and limiting tokens while maintaining accurate responses</li>
</ul>

<h2>
  
  
  Tech Stack
</h2>

<p>The tools and technologies I use in my projects most frequently:</p>

<ul>
<li>
<strong>Frontend (Web):</strong> Typescript/Vite &amp; Tailwind</li>
<li>
<strong>Mobile:</strong> Flutter</li>
<li>
<strong>Backend:</strong> Go (Gin)</li>
<li>
<strong>DB:</strong> PostgreSQL, GORM</li>
<li>
<strong>LLM API:</strong> Anthropic, OpenAI (Sometimes - mostly as backup), Mistral (For simple tasks)</li>
<li>
<strong>TTS API:</strong> OpenAI TTS</li>
<li>
<strong>IDE:</strong> Cursor</li>
<li>
<strong>Cloud Hosting:</strong> Digital Ocean</li>
<li>
<strong>Image and Video Hosting:</strong> Cloudinary</li>
</ul>

<p><strong>NOTE:</strong> Architectural concepts apply regardless of stack.</p>

<h2>
  
  
  Structure of Course Topics
</h2>

<p>How we'll lay out easily digestible info:</p>

<ol>
<li>Introduction/definition of a common issue or a foundational concept</li>
<li>Real world story involving this topic in either one of my projects or a well-known platform elsewhere and its resolution</li>
<li>The decision making process breakdown and specifics that should inform build</li>
<li>Major takeaways from previous slides and how to apply this to your own project</li>
</ol>

<h2>
  
  
  My Projects
</h2>

<p>What I've built using the methods that I'm sharing in this course:</p>

<h3>
  
  
  Emstrata
</h3>

<p>A text-based emergent storycraft simulator where stories unfold like lived experiences. Participants make choices that shape narrative worlds, AI systems maintain continuity and consistency, and users can interact with the simulation—altering mid-flight, navigating AI-generated planes, making inquiries, and correcting errors.</p>

<h3>
  
  
  PLATO5 (In the middle of a redesign)
</h3>

<p>An AI-first social engine designed to generate real-world friendships, not screen time. It matches people based on personality (Big 5 traits), interests, and location, facilitates conversations through "Zen—an AI chat manager," and guides users toward planning actual meetups, with the goal of getting people off the app and into real life.</p>

<h2>
  
  
  Why Build AI Platforms
</h2>

<p>Novel applications of AI are best for this:</p>

<ul>
<li>When you're building something that doesn't exist yet, you need architecture designed specifically for that problem</li>
<li>Complex AI applications require precise control over how prompts are structured, how state is managed across multiple AI calls, and how data flows between different parts of the system</li>
<li>Building from scratch means you can optimize every layer for your specific use case—from how you call the AI to how you handle edge cases to how you manage costs at scale</li>
<li>The architecture choices you make define what's possible. Emstrata's multi-layer narrative system works because the entire platform is built around maintaining continuity and tracking story threads</li>
</ul>

<h2>
  
  
  The Cost Reality of AI Apps
</h2>

<p>How much an AI application will cost to run, test, and maintain:</p>

<ul>
<li>
<strong>AI API usage:</strong> Usage is measured in tokens (MToks). Different models and APIs have varying charges and strengths not always reflected in benchmarks.</li>
<li>
<strong>Input and Output costs:</strong> Input and output tokens often have different prices, with output tokens typically costing more. Applications generating large AI responses will incur higher costs, especially compared to those with larger context windows.</li>
<li>
<strong>Context window size:</strong> The size of the context window affects not only prices but also performance. It's important to manage the amount of tokens sent to the API and prefer standardized methods for controlling and organizing tokens into easily readable inputs.</li>
</ul>

<h2>
  
  
  Apps that Evolve
</h2>

<p>Building for the unpredictability of AI:</p>

<ul>
<li>AIs hallucinate. You likely already know this if you're considering building an AI platform, but consider that the scale of the hallucination and the downstream effects of them can expand exponentially with architecture and design that rests on the accuracy of AI responses. A lot of this course will cover this reality and demonstrate ways to hedge against these unfortunate issues or embrace them in interesting ways.</li>
<li>Luckily, many hallucinations can be stamped out, accounted for, or predicted in the course of designing your app. This will require persistence, trial and error, and a level of systems-thinking.</li>
<li>All in all, this is a feature, not a bug. The reason AIs hallucinate is the same reason that we will receive different responses to the same input and the AI can mimic human creative capacity.</li>
</ul>

<h2>
  
  
  External Resources
</h2>

<p><a href="https://github.com/goldsteinnicholas/build-ai-platforms-from-scratch" rel="noopener noreferrer">Repo w/ Common Code Patterns</a><br><br>
Common code patterns and examples from the course.</p>

<p><a href="https://nicholasmgoldstein.com/system-prompt-generator" rel="noopener noreferrer">System Prompt Generator</a><br><br>
A tool for generating effective system prompts for AI platforms.</p>

<p><a href="https://emstrata.com" rel="noopener noreferrer">Emstrata</a><br><br>
A platform for creating immersive narrative experiences using AI to generate emergent storylines.</p>

<p><a href="https://plato5.us" rel="noopener noreferrer">PLATO5</a><br><br>
A social engine designed to turn online connections into real-world friendships, with AI integration to facilitate conversations.</p>

