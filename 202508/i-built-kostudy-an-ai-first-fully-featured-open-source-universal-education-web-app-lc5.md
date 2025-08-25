---
Title: I built Kostudy - An AI-first, fully-featured, open source, universal education web app
Description: 
Author: youssef-imlyhen
Date: 2025-08-25T19:13:04.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Kostudy: Reimagining AI-Powered Learning with React, TypeScript, and the Google Gemini API
</h1>

<h2>
  
  
  Live Demo
</h2>

<ul>
<li>Watch on YouTube: <a href="https://youtu.be/VsXnGYEopW0" rel="noopener noreferrer">https://youtu.be/VsXnGYEopW0</a>
</li>
<li>Try it at <a href="https://kostudy.netlify.app/" rel="noopener noreferrer">https://kostudy.netlify.app/</a>
</li>
<li>Code: <a href="https://github.com/youssef-imlyhen/kostudy" rel="noopener noreferrer">https://github.com/youssef-imlyhen/kostudy</a>
</li>
</ul>

<p>Most of today's learning tools fall short. They either hook you with cheap gamification to sell more ads, or they slap a chatbot onto a decade-old quiz and call it an "AI education app." You deserve better.</p>

<p>That's why I built <strong>Kostudy</strong> – an AI-first, feature-rich, open-source learning application that leverages modern web technologies and the Google Gemini API to deliver an intelligent, context-aware, and engaging learning experience.</p>

<p>In this post, we'll dive deep into Kostudy's architecture, explore its unique AI-powered features, and see how it's redefining what a learning app should be in the age of AI.</p>

<h2>
  
  
  The Problem with Current Learning Apps
</h2>

<p>Before we dive into Kostudy's solutions, let's address a fundamental question: Why do most learning apps fail to deliver real value?</p>

<p>Traditional learning apps suffer from several critical issues:</p>

<ol>
<li>
<strong>Superficial AI Integration</strong>: Many apps simply add a chatbot to an old quiz engine and call it "AI-powered"</li>
<li>
<strong>One-Size-Fits-All Approach</strong>: They don't adapt to individual learning needs or contexts</li>
<li>
<strong>Poor Content Quality</strong>: Questions are often generic and not tailored to specific learning materials</li>
<li>
<strong>Limited Interactivity</strong>: Most apps offer basic quizzes with little engagement beyond multiple choice</li>
<li>
<strong>No Context Awareness</strong>: They can't leverage your own materials (videos, PDFs, notes) for personalized learning</li>
</ol>

<p>These limitations create a gap between what learners need and what's available. Kostudy was designed to bridge this gap.</p>

<h2>
  
  
  Introducing Kostudy: A New Approach to Learning
</h2>

<p>Kostudy is an open-source learning application built with React, TypeScript, and Tailwind CSS. It leverages the Google Gemini API to provide AI-powered features that transform how we learn. But what makes Kostudy different?</p>

<p>Unlike traditional learning apps, Kostudy puts AI at the center of the learning experience. It's not just an add-on feature – it's the foundation that enables:</p>

<ul>
<li>Context-aware question generation from your own materials</li>
<li>Interactive learning adventures that adapt to your progress</li>
<li>AI-powered app generation for custom learning experiences</li>
<li>Real-time voice conversations with an AI tutor</li>
<li>And much more</li>
</ul>

<p>Let's dive into the technical architecture that makes this possible.</p>

<h2>
  
  
  Technical Architecture: Building a Modern Learning Platform
</h2>

<h3>
  
  
  Core Technologies
</h3>

<p>Kostudy is built on a modern stack that prioritizes performance, developer experience, and extensibility:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"dependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"@google/genai"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^1.10.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"react"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^18.3.1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"react-dom"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^18.3.1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"react-router-dom"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^7.1.3"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"react-hook-form"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^7.60.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"react-icons"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^5.5.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"react-markdown"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^10.1.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@headlessui/react"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^2.2.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@heroicons/react"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^2.2.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"howler"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^2.2.4"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"papaparse"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^5.5.3"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"uuid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^11.0"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"devDependencies"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"typescript"</span><span class="p">:</span><span class="w"> </span><span class="s2">"~5.6.2"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"vite"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^6.0.5"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"tailwindcss"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^3.4.17"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"daisyui"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^4.12.23"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"@vitejs/plugin-react"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^4.3.4"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"vite-plugin-pwa"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^1.0.2"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Component Architecture
</h3>

<p>Kostudy follows a modular component architecture that makes it easy to extend and maintain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>src/
├── components/          # Reusable UI components
├── context/             # React context providers
├── data/                # Static data and question banks
├── hooks/               # Custom React hooks
├── localization/        # Multi-language support
├── sagalearn/           # Interactive learning adventure engine
├── screens/             # Top-level screen components
├── services/            # Business logic and API integrations
├── types/               # TypeScript type definitions
└── utils/               # Utility functions
</code></pre>

</div>



<h3>
  
  
  State Management
</h3>

<p>Kostudy uses React's Context API for state management, which provides a clean and predictable way to manage global state:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/context/QuizContext.tsx</span>
<span class="k">export</span> <span class="kr">interface</span> <span class="nx">QuizContextType</span> <span class="p">{</span>
  <span class="nl">currentQuiz</span><span class="p">:</span> <span class="p">{</span>
    <span class="nx">category</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">difficulty</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">progress</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">score</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">currentLevel</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
    <span class="nl">playAllMode</span><span class="p">?:</span> <span class="nx">boolean</span><span class="p">;</span>
    <span class="nl">currentStreak</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">maxStreak</span><span class="p">?:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="p">};</span>
  <span class="nl">setCurrentQuiz</span><span class="p">:</span> <span class="p">(</span><span class="nx">quiz</span><span class="p">:</span> <span class="nx">QuizContextType</span><span class="p">[</span><span class="dl">'</span><span class="s1">currentQuiz</span><span class="dl">'</span><span class="p">])</span> <span class="o">=&gt;</span> <span class="k">void</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">QuizContext</span> <span class="o">=</span> <span class="nx">createContext</span><span class="o">&lt;</span><span class="nx">QuizContextType</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
</code></pre>

</div>



<p>This approach eliminates the need for external state management libraries while keeping the codebase clean and maintainable.</p>

<h2>
  
  
  AI-Powered Features: The Heart of Kostudy
</h2>

<h3>
  
  
  1. AI Question Generation with Rich Media Context
</h3>

<p>One of Kostudy's standout features is its ability to generate context-aware questions from various media types. This is powered by the Google Gemini API and supports:</p>

<ul>
<li>YouTube videos</li>
<li>PDF documents</li>
<li>Audio files</li>
<li>Images</li>
<li>Web articles</li>
</ul>

<p>The implementation uses a sophisticated media processing pipeline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/utils/mediaUtils.ts</span>
<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">mediaContextToParts</span><span class="p">(</span>
  <span class="nx">context</span><span class="p">:</span> <span class="nx">MediaContext</span><span class="p">,</span>
  <span class="nx">genAI</span><span class="p">:</span> <span class="nx">GoogleGenAI</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">Part</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="na">parts</span><span class="p">:</span> <span class="nx">Part</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="k">switch </span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="kd">type</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">youtube</span><span class="dl">'</span><span class="p">:</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">url</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">parts</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nf">createYouTubePart</span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">url</span><span class="p">));</span>
      <span class="p">}</span>
      <span class="k">break</span><span class="p">;</span>

    <span class="k">case</span> <span class="dl">'</span><span class="s1">image</span><span class="dl">'</span><span class="p">:</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">video</span><span class="dl">'</span><span class="p">:</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">audio</span><span class="dl">'</span><span class="p">:</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">pdf</span><span class="dl">'</span><span class="p">:</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">file</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">try</span> <span class="p">{</span>
          <span class="c1">// For large files, use file upload API</span>
          <span class="k">if </span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">file</span><span class="p">.</span><span class="nx">size</span> <span class="o">&gt;</span> <span class="mi">20</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">*</span> <span class="mi">1024</span> <span class="o">||</span> <span class="nx">context</span><span class="p">.</span><span class="kd">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">pdf</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
            <span class="kd">const</span> <span class="nx">filePart</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">uploadFileToGemini</span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">file</span><span class="p">,</span> <span class="nx">genAI</span><span class="p">);</span>
            <span class="nx">parts</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">filePart</span><span class="p">);</span>
          <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
            <span class="c1">// For smaller files, use inline data</span>
            <span class="kd">const</span> <span class="nx">filePart</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fileToGenerativePart</span><span class="p">(</span><span class="nx">context</span><span class="p">.</span><span class="nx">file</span><span class="p">);</span>
            <span class="nx">parts</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">filePart</span><span class="p">);</span>
          <span class="p">}</span>
        <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error processing media file:</span><span class="dl">'</span><span class="p">,</span> <span class="nx">error</span><span class="p">);</span>
          <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
        <span class="p">}</span>
      <span class="p">}</span>
      <span class="k">break</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">parts</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This approach allows Kostudy to handle files up to 100MB and provides appropriate processing based on file type and size.</p>

<h3>
  
  
  2. AI Chat Assistant with Context Awareness
</h3>

<p>Kostudy's chat assistant goes beyond simple Q&amp;A by maintaining context from your learning materials:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/components/AIGenerationTab.tsx</span>
<span class="kd">const</span> <span class="nx">fullPrompt</span> <span class="o">=</span> <span class="s2">`
  </span><span class="p">${</span><span class="nx">mediaPromptPrefix</span><span class="p">}</span><span class="s2">generate a list of quiz questions in JSON format.
  The user's text will specify the topic and the number of questions to generate.
  Each question object must have the following structure:
  {
    "id": "a-unique-uuid",
    "category": "A relevant category based on the content, with ' (AI Gen)' appended",
    "question": "The question text",
    "type": "multiple-choice", // or "true-false" or "fill-in-the-blank"
    "difficulty": "medium", // or "easy" or "hard"
    "explanation": "A brief explanation of why the answer is correct. Supports markdown formatting."
  }
`</span><span class="p">;</span>
</code></pre>

</div>



<h3>
  
  
  3. Interactive SagaLearn: Choose-Your-Own-Adventure Learning
</h3>

<p>SagaLearn transforms learning into an adventure with branching narratives that adapt to your choices and performance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/screens/SagaLearnScreen.tsx</span>
<span class="kd">const</span> <span class="nx">convertToSagaQuestion</span> <span class="o">=</span> <span class="p">(</span><span class="nx">question</span><span class="p">:</span> <span class="nx">MainAppQuestion</span><span class="p">):</span> <span class="nx">SagaQuestion</span> <span class="o">|</span> <span class="kc">null</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">answer</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">wrongOptions</span><span class="p">:</span> <span class="kr">string</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="k">switch </span><span class="p">(</span><span class="nx">question</span><span class="p">.</span><span class="kd">type</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">case</span> <span class="dl">'</span><span class="s1">multiple-choice</span><span class="dl">'</span><span class="p">:</span>
      <span class="k">if </span><span class="p">(</span><span class="k">typeof</span> <span class="nx">question</span><span class="p">.</span><span class="nx">correctAnswer</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">answer</span> <span class="o">=</span> <span class="nx">question</span><span class="p">.</span><span class="nx">correctAnswer</span><span class="p">;</span>
        <span class="nx">wrongOptions</span> <span class="o">=</span> <span class="nx">question</span><span class="p">.</span><span class="nx">options</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">opt</span> <span class="o">=&gt;</span> <span class="nx">opt</span> <span class="o">!==</span> <span class="nx">question</span><span class="p">.</span><span class="nx">correctAnswer</span><span class="p">);</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="c1">// Handle multi-select</span>
        <span class="nx">answer</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="nf">isArray</span><span class="p">(</span><span class="nx">question</span><span class="p">.</span><span class="nx">correctAnswer</span><span class="p">)</span> <span class="p">?</span> <span class="nx">question</span><span class="p">.</span><span class="nx">correctAnswer</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="p">:</span> <span class="dl">''</span><span class="p">;</span>
        <span class="nx">wrongOptions</span> <span class="o">=</span> <span class="nx">question</span><span class="p">.</span><span class="nx">options</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">opt</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">question</span><span class="p">.</span><span class="nx">correctAnswer</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">opt</span><span class="p">));</span>
      <span class="p">}</span>
      <span class="k">break</span><span class="p">;</span>
    <span class="c1">// ... other question types</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">topic</span><span class="p">:</span> <span class="nx">question</span><span class="p">.</span><span class="nx">category</span><span class="p">,</span>
    <span class="na">question</span><span class="p">:</span> <span class="nx">question</span><span class="p">.</span><span class="nx">question</span><span class="p">,</span>
    <span class="nx">answer</span><span class="p">,</span>
    <span class="nx">wrongOptions</span>
  <span class="p">};</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  4. AI App Studio: Generate Custom Learning Apps
</h3>

<p>Perhaps the most innovative feature is the AI App Studio, which can generate complete learning applications from simple prompts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/services/enhancedAIService.ts</span>
<span class="k">private</span> <span class="nf">buildEnhancedSystemPrompt</span><span class="p">():</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="k">return</span> <span class="s2">`You are an expert full-stack developer, educational technologist, and UX designer with deep expertise in creating engaging, interactive web applications.

  CORE MISSION:
  Create exceptional, self-contained HTML applications that are not just functional, but truly engaging and educational.

  CRITICAL REQUIREMENTS:
  1. Generate ONLY a complete HTML file with embedded CSS and JavaScript
  2. The application must be fully functional and self-contained
  3. Use cutting-edge web technologies (HTML5, CSS3, ES2023+ JavaScript)
  4. Implement responsive design with mobile-first approach
  5. Include smooth animations, transitions, and micro-interactions
  6. Ensure full accessibility (WCAG 2.1 AA compliance)
  7. Add comprehensive error handling and user feedback
  8. Create beautiful, modern UI with attention to visual hierarchy
  9. Implement progressive enhancement principles
  10. Include performance optimizations
  `</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Core Quiz Features: Solid Foundation for Learning
</h2>

<h3>
  
  
  Flexible Question Types
</h3>

<p>Kostudy supports multiple question types with rich media support:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/types/question.ts</span>
<span class="k">export</span> <span class="kd">type</span> <span class="nx">QuestionType</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">multiple-choice</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">true-false</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">fill-in-the-blank</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kr">interface</span> <span class="nx">MultipleChoiceQuestion</span> <span class="kd">extends</span> <span class="nx">QuestionBase</span> <span class="p">{</span>
  <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">multiple-choice</span><span class="dl">'</span><span class="p">;</span>
  <span class="nl">options</span><span class="p">:</span> <span class="kr">string</span><span class="p">[];</span>
  <span class="nl">correctAnswer</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kr">string</span><span class="p">[];</span> <span class="c1">// Can be a single answer or multiple</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kr">interface</span> <span class="nx">TrueFalseQuestion</span> <span class="kd">extends</span> <span class="nx">QuestionBase</span> <span class="p">{</span>
  <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">true-false</span><span class="dl">'</span><span class="p">;</span>
 <span class="nl">correctAnswer</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kr">interface</span> <span class="nx">FillInTheBlankQuestion</span> <span class="kd">extends</span> <span class="nx">QuestionBase</span> <span class="p">{</span>
  <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">fill-in-the-blank</span><span class="dl">'</span><span class="p">;</span>
  <span class="nl">correctAnswer</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Customization and Theming
</h3>

<p>Kostudy's theming system is highly customizable through a central configuration file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/config.ts</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">config</span><span class="p">:</span> <span class="nx">AppConfig</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">appName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Kostudy</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">theme</span><span class="p">:</span> <span class="dl">'</span><span class="s1">light</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">primaryColor</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#58CC02</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">quizFeatures</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">enableMultipleChoice</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">enableTrueFalse</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">enableFillInTheBlank</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">enableMultiSelect</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">enableImageOptions</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">enableQuestionImage</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">},</span>
  <span class="na">aiFeatures</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">enableAIGeneration</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Internationalization
</h3>

<p>With support for 16+ languages, Kostudy makes learning accessible worldwide:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/context/LanguageContext.tsx</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">languages</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">en</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">English</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">fr</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Français</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">es</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Español</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">ar</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">العربية</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">rtl</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">de</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Deutsch</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">hi</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">हिन्दी</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">pt</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Português</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="na">zh</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">中文</span><span class="dl">'</span><span class="p">,</span> <span class="na">dir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ltr</span><span class="dl">'</span> <span class="p">},</span>
  <span class="c1">// ... 10 more languages</span>
<span class="p">}</span> <span class="k">as</span> <span class="kd">const</span><span class="p">;</span>
</code></pre>

</div>



<h2>
  
  
  Developer Experience: Building and Extending Kostudy
</h2>

<h3>
  
  
  Easy Setup and Development
</h3>

<p>Getting started with Kostudy is straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/youssef-imlyhen/kostudy.git
<span class="nb">cd </span>kostudy
npm <span class="nb">install
</span>npm run dev
</code></pre>

</div>



<p>The project uses Vite for fast development and hot module replacement, making the development experience smooth and efficient.</p>

<h3>
  
  
  Customization Through Configuration
</h3>

<p>Kostudy is designed to be easily customizable without requiring code changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/config.ts</span>
<span class="k">export</span> <span class="kr">interface</span> <span class="nx">AppConfig</span> <span class="p">{</span>
  <span class="nl">appName</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">theme</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">primaryColor</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">quizFeatures</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">enableMultipleChoice</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
    <span class="nl">enableTrueFalse</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
    <span class="nl">enableFillInTheBlank</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
    <span class="c1">// ... other features</span>
  <span class="p">};</span>
  <span class="nl">aiFeatures</span><span class="p">?:</span> <span class="p">{</span>
    <span class="nx">enableAIGeneration</span><span class="p">?:</span> <span class="nx">boolean</span><span class="p">;</span>
    <span class="nl">geminiApiKey</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="p">};</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Extensibility
</h3>

<p>The modular architecture makes it easy to add new features:</p>

<ol>
<li>
<strong>New Question Types</strong>: Extend the <code>Question</code> type and add corresponding display components</li>
<li>
<strong>New AI Features</strong>: Implement new services in the <code>services/</code> directory</li>
<li>
<strong>New UI Components</strong>: Add components to the <code>components/</code> directory</li>
<li>
<strong>New Learning Modes</strong>: Create new screens in the <code>screens/</code> directory</li>
</ol>

<h2>
  
  
  Deployment and Scaling
</h2>

<p>Kostudy is built as a static site, making it easy to deploy anywhere:</p>

<ul>
<li>
<strong>Netlify</strong>: One-click deployment with automatic builds</li>
<li>
<strong>Vercel</strong>: Seamless integration with GitHub</li>
<li>
<strong>GitHub Pages</strong>: Free hosting with GitHub Actions</li>
<li>
<strong>Self-hosted</strong>: Build and deploy to any static hosting provider
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Production build</span>
npm run build

<span class="c"># Preview locally</span>
npm run preview
</code></pre>

</div>



<h2>
  
  
  Performance and Optimization
</h2>

<p>Kostudy incorporates several performance optimizations:</p>

<ol>
<li>
<strong>Code Splitting</strong>: React.lazy and Suspense for route-based code splitting</li>
<li>
<strong>Image Optimization</strong>: Proper handling of media files with base64 encoding and file uploads</li>
<li>
<strong>Bundle Optimization</strong>: Vite's built-in optimizations for production builds</li>
<li>
<strong>Caching</strong>: localStorage for user progress and preferences</li>
<li>
<strong>Lazy Loading</strong>: Components and data loaded only when needed</li>
</ol>

<h2>
  
  
  Future Roadmap
</h2>

<p>Kostudy is under active development with exciting features planned:</p>

<ul>
<li>
<strong>More AI Providers</strong>: Support for local AI models (llama.cpp, Ollama, LM Studio)</li>
<li>
<strong>Spaced Repetition</strong>: Integration of learning science principles for long-term retention</li>
<li>
<strong>Enhanced Onboarding</strong>: Guided setup flows and templates for common use cases</li>
<li>
<strong>Expanded Import/Export</strong>: Additional formats and better validation</li>
<li>
<strong>Accessibility Improvements</strong>: Full a11y compliance and performance optimizations</li>
</ul>

<h2>
  
  
  Conclusion: The Future of AI-Powered Learning
</h2>

<p>Kostudy represents a significant step forward in educational technology. By putting AI at the center of the learning experience and providing rich context awareness, it offers a more personalized and engaging approach to learning than traditional apps.</p>

<p>Whether you're a student looking to enhance your study sessions, a teacher creating custom learning materials, or a developer interested in educational technology, Kostudy provides a powerful platform to explore new possibilities in AI-powered learning.</p>

<p>The project is open-source and welcomes contributions. If you're interested in helping shape the future of education technology, check out the repository on GitHub and consider contributing.</p>

<h3>
  
  
  Try Kostudy Today
</h3>

<p>Ready to transform your learning experience? Visit <a href="https://kostudy.netlify.app/" rel="noopener noreferrer">kostudy.netlify.app</a> to try the live demo, or check out the <a href="https://github.com/youssef-imlyhen/kostudy" rel="noopener noreferrer">GitHub repository</a> to run it locally or contribute to the project.</p>

<p>The future of learning is here – and it's powered by AI.</p>

