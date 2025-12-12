---
Title: Setting Up Your Platform Prereqs - Build AI Platforms From Scratch #2
Description: 
Author: Nick Goldstein
Date: 2025-12-12T21:25:44.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Setting Up Your Platform Prerequisites
</h1>

<p>Developing the framework to iterate on your platforms, selecting a tech &amp; productivity stack.</p>

<p><strong>📺 <a href="https://nicholasmgoldstein.com/course/build-ai-platforms/module/2" rel="noopener noreferrer">View this module with video &amp; slides</a></strong></p>




<h2>
  
  
  Quick Reminder...
</h2>

<p>For people who may have forgotten:</p>

<ul>
<li>This course isn't for coding. However, there will be a code repo linked in the External Resources section that has common code patterns that you will likely need for calling third-party LLM APIs, parsing responses, counting tokens and assessing cost, etc.</li>
<li>If you want a more focused learning environment, take the course on nicholasmgoldstein.com/courses. This way you won't be tempted to jump to a recommended video or divert your attention.</li>
<li>This is a free course, but if you want to support me, you can like/subscribe or check out some of my own AI platforms like Emstrata @ emstrata.com.</li>
<li>I'm fully open to your input. If there's topics you'd like covered more thoroughly or anything you're unclear on, feel free to comment.</li>
</ul>

<h2>
  
  
  Picking an LLM
</h2>

<p>Making your choice early on prevents paralysis later.</p>

<p><strong>LLM Strengths and Learning:</strong> Different LLMs have varying strengths and textual voices. The best way to learn them is by working with them and iterating on system prompts, rather than solely relying on benchmarks.</p>

<p><strong>Recommended LLMs and Usage:</strong> My personal favorite is Claude/Anthropic's third-party API (specifically versions 3.7 to 4.0), but ChatGPT/OpenAI and Gemini/Google are also great options. These platforms typically require a credit card for usage, and pricing information can be found on their respective websites.</p>

<p><strong>Multi-LLM Strategy:</strong> Consider using multiple LLMs for different types of tasks. For instance, Claude 4.0 is excellent for heavier analysis and rule-following, while Mistral or other smaller models work well for simpler tasks. This strategy can save money while maintaining effectiveness.</p>

<p><strong>Additional Tools:</strong> A benefit of using OpenAI or Google over Anthropic is that their platforms often include other built-in tools such as Text-to-Speech (TTS), transcription, and image generation.</p>

<h2>
  
  
  The Choice is Yours
</h2>

<p>Below are some links to some of the API Workbenches mentioned:</p>

<ul>
<li>
<strong>Anthropic</strong> - <a href="https://console.anthropic.com" rel="noopener noreferrer">console.anthropic.com</a>
</li>
<li>
<strong>OpenAI</strong> - <a href="https://platform.openai.com" rel="noopener noreferrer">platform.openai.com</a>
</li>
<li>
<strong>Google</strong> - <a href="https://ai.google.dev" rel="noopener noreferrer">ai.google.dev</a>
</li>
<li>
<strong>Mistral</strong> - <a href="https://console.mistral.ai" rel="noopener noreferrer">console.mistral.ai</a>
</li>
</ul>

<h2>
  
  
  Anatomy of an AI Request
</h2>

<p>How building in the Workbench looks:</p>

<p><strong>Prompt Library System:</strong> Most workbenches have a feature to save prompts for testing. You'll need a naming system for high volume usage.</p>

<p><strong>Model Selection:</strong> Click on the model name (e.g., 'claude-sonnet-4-5...') to switch between different Anthropic models or other providers.</p>

<p><strong>System Prompt:</strong> The static instruction sent to the LLM. This fundamentally defines what the prompt does and sets the behavior and context for the AI.</p>

<p><strong>User Request:</strong> The term 'User' can be misleading for non-chatbot systems. This refers to the particular data sent for a specific instance or request.</p>

<p><strong>Expected Response:</strong> This is an LLM response from the API, which you can test and iterate on before implementing in your application.</p>

<h2>
  
  
  Intermediate Techniques
</h2>

<p>System Prompt Variables &amp; Message Pairs:</p>

<p><strong>System Prompt Variables:</strong> Wrap camelCase strings in <code>{{...}}</code> to setup a variable within your system prompt. This could be used to customize the prompt for specific purposes or to further modularize prompts.</p>

<p><strong>Message Pairs:</strong> The most common use-case is for chat history, but fundamentally adds context to this specific request. This allows you to maintain conversation context across multiple interactions.</p>

<p><strong>NOTE:</strong> My personal preference is to avoid things like system prompt variables and instead systematize user requests. You'll see examples of how I do this in the Prompt Engineering module. This is just what makes more sense to me, but both approaches have their merits.</p>

<h2>
  
  
  Thoughts on Frontend/Backend
</h2>

<p>You will need to select which languages/frameworks you want to use:</p>

<p><strong>Security Warning:</strong> Do NOT expose your API keys on the frontend AT ALL. This is a massive security vulnerability. This is why third-party LLM API calls will be made on the backend. Remember, your bank accounts and/or cards are linked to these API keys. Set proper limits and don't expose yourself to bad actors.</p>

<p>I'll be using TypeScript/Vite for frontend, Flutter for mobile, and Go/Gin for backend in this tutorial. AI-Powered IDEs will be a necessity for the non-coders (even if you know how to code, they boost productivity significantly). Cursor is my go-to.</p>

<p>Again, code will not be a prominent factor in this course, but I'll give concrete steps for setting up a basic frontend and backend that you can use to iterate on for your project.</p>

<h2>
  
  
  Learn While Building
</h2>

<p>Don't let AI run rampant in your project:</p>

<ul>
<li>If you're new to coding, a non-coder, or just working with code patterns you're unfamiliar with, AI can help you learn.</li>
<li>Vibe-coding is overrated—learn what the AI is writing and why it works. This will help you with troubleshooting later and also let you know what is and isn't possible in your own projects. Don't fall for one-shot app-builder marketing; you should still know the fundamentals.</li>
<li>In Cursor, instruct your agent to "Add comments explaining what each line of code is doing in-depth. I'm new to coding and would like to understand what each line is doing." This simple instruction will help you immensely and also help the AI think critically about the code it's writing. Then it's on you; actually read the comments for better understanding of your system.</li>
</ul>

<h2>
  
  
  Frontend Setup
</h2>

<p>TypeScript/Vite Steps/Commands:</p>

<ul>
<li>If you need to revisit: Click into the code repo and open setup-instructions.txt</li>
<li><code>npm create vite@latest my-project-name -- --template vanilla-ts</code></li>
<li><code>cd my-project-name</code></li>
<li><code>npm install</code></li>
<li><code>npm run dev</code></li>
</ul>

<h2>
  
  
  Backend Setup
</h2>

<p>Go/Gin Steps/Commands:</p>

<ul>
<li>More instructions for Go: Click into the code repo and open setup-instructions.txt</li>
<li><code>mkdir my-project-name</code></li>
<li><code>cd my-project-name</code></li>
<li><code>go mod init github.com/yourusername/my-project-name</code></li>
<li><code>go get -u github.com/gin-gonic/gin</code></li>
</ul>

<h2>
  
  
  Database Setup
</h2>

<p>PostgreSQL &amp; GORM Steps/Commands:</p>

<ul>
<li>More instructions for Database setup: Click into the code repo and open setup-instructions.txt</li>
<li>Install PostgreSQL</li>
<li><code>createdb mydb</code></li>
<li>Then in your Go project:

<ul>
<li><code>go get -u gorm.io/gorm</code></li>
<li><code>go get -u gorm.io/driver/postgres</code></li>
</ul>


</li>

</ul>

<h2>
  
  
  Mobile App Setup
</h2>

<p>Flutter Steps/Commands:</p>

<ul>
<li><code>flutter create my_project_name</code></li>
<li><code>cd my_project_name</code></li>
<li><code>flutter run</code></li>
<li>(You'll need Flutter SDK installed first: <a href="https://docs.flutter.dev/get-started/install" rel="noopener noreferrer">https://docs.flutter.dev/get-started/install</a>)</li>
</ul>

<h2>
  
  
  The Main Takeaway
</h2>

<p>What's the point of all this setup?</p>

<p>If you're new to coding, you don't need to understand any of this code yet. The goal is to have a working environment—frontend, backend, database, mobile—so when you start building, everything's already there.</p>

<p>AI will be essential for the implementation. Cursor and other AI-powered IDEs can write the code. Your job is understanding what you're building, overseeing, troubleshooting, and architecting, not necessarily memorizing syntax, although you will learn a lot about coding throughout this process.</p>




<h2>
  
  
  Continue Learning
</h2>

<p><strong><a href="https://nicholasmgoldstein.com/course/build-ai-platforms/module/2" rel="noopener noreferrer">Watch Module 2 with video &amp; slides</a></strong></p>

<p><strong><a href="https://nicholasmgoldstein.com/courses" rel="noopener noreferrer">View full course</a></strong></p>

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

