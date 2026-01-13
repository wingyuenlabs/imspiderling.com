---
Title: Focus-Roast: Unleashing AI-Powered Shame to Conquer Procrastination
Description: 
Author: Shashank Chakraborty
Date: 2026-01-13T22:06:19.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Focus-Roast: Unleashing AI-Powered Shame to Conquer Procrastination
</h1>

<p>Tired of politely ignored site blockers? We've all been there. Most productivity tools offer gentle nudges or a serene blank screen when you stray, making it all too easy to bypass them and dive back into the rabbit hole of distractions.</p>

<p>Enter <strong>Focus-Roast</strong>: a novel, open-source approach to productivity that leverages artificial intelligence to ruthlessly bully you into staying on task. This isn't your grandma's site blocker; it's an uncompromising digital drill sergeant designed to make procrastination an uncomfortably public and embarrassing experience.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fvia.placeholder.com%2F800x400%3Ftext%3DInsert%2BScreenshot%2Bof%2BFocus-Roast%2BRed%2BScreen%2Bwith%2BRoast" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fvia.placeholder.com%2F800x400%3Ftext%3DInsert%2BScreenshot%2Bof%2BFocus-Roast%2BRed%2BScreen%2Bwith%2BRoast" alt="Focus-Roast Demo" width="800" height="400"></a><br>
<em>Figure 1: A mock-up of Focus-Roast in action, displaying a context-aware roast.</em></p>
<h2>
  
  
  😈 Core Functionality &amp; Architecture
</h2>

<p>Focus-Roast operates as a two-part system: a Python backend handles the AI heavy lifting, and a Chrome extension acts as the vigilant gatekeeper, intercepting your browsing habits. This architecture ensures your browsing data remains entirely local and private.</p>
<h3>
  
  
  Key Features:
</h3>

<ul>
<li>  <strong>Context-Aware Roasts via Gemini AI:</strong> At its heart, Focus-Roast integrates with <strong>Google's Gemini AI</strong>. When you attempt to access a blacklisted site, the extension sends your predefined goal (e.g., "Study Calculus") and the offending URL (e.g., "instagram.com") to the backend. The AI then generates a savage, personalized insult tailored to your specific transgression.</li>
<li>  <strong>Audio Guilt 🔊 (Text-to-Speech):</strong> Not only does the roast appear on screen, but it's also immediately converted to speech and played aloud. Imagine trying to surreptitiously scroll through social media in a quiet office or library while Focus-Roast declares your "oxygen thief" status.</li>
<li>  <strong>Gallery of Disappointment (Visual Deterrence):</strong> To amplify the guilt, the system displays random GIFs featuring famously disappointed figures (e.g., Gordon Ramsay, characters from <em>The Office</em>), providing a constant visual reminder of your failure to focus.</li>
<li>  <strong>The "Walk of Shame" 🚶‍♂️ (Cognitive Friction):</strong> There's no quick "Close" button. To regain access to the distracting site, you are forced to manually type the phrase: <em>"I am weak and lazy"</em>. This deliberate act of self-admission creates significant cognitive friction, making you consciously confront your procrastination.</li>
<li>  <strong>Shame Rank System 📉 (Gamified Accountability):</strong> Focus-Roast keeps a persistent tally of your failures, categorizing your "shame rank":

<ul>
<li>  <strong>0-2 attempts:</strong> Safe... for now.</li>
<li>  <strong>3-5 attempts:</strong> Certified Clown 🤡</li>
<li>  <strong>10+ attempts:</strong> Oxygen Thief 💀</li>
</ul>
</li>
</ul>
<h2>
  
  
  🛠️ Local Setup: Running Your Personal Roaster
</h2>

<p>Focus-Roast is designed to run entirely locally, giving you full control over your data and eliminating any third-party server dependencies.</p>
<h3>
  
  
  1. The Brain (Backend) 🧠
</h3>

<p>The backend is built with Python, leveraging <code>uvicorn</code> and <code>FastAPI</code> to serve the AI roasting logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Clone the repository</span>
git clone https://github.com/YOUR_USERNAME/focus-roast.git
<span class="nb">cd </span>focus-roast/backend

<span class="c"># Install Python dependencies</span>
pip <span class="nb">install</span> <span class="nt">-r</span> requirements.txt

<span class="c"># Configure your Gemini API Key</span>
<span class="c"># You'll need a free API key from Google AI Studio (aistudio.google.com).</span>
<span class="c"># Create a .env file in the 'backend' directory with your key:</span>
<span class="nb">echo</span> <span class="s2">"GEMINI_API_KEY=AIzaSy..."</span> <span class="o">&gt;</span> .env

<span class="c"># Start the roasting server</span>
<span class="c"># The --reload flag enables live reloading during development.</span>
uvicorn main:app <span class="nt">--reload</span>
</code></pre>

</div>



<p>Ensure your <code>GEMINI_API_KEY</code> is correctly set in the <code>.env</code> file. This key is crucial for the AI roast generation.</p>

<h3>
  
  
  2. The Trap (Chrome Extension) 🌐
</h3>

<p>The frontend is a standard Chrome extension that monitors your browsing and communicates with your local backend.</p>

<ol>
<li> <strong>Open Chrome Extensions Manager:</strong> Navigate to <code>chrome://extensions</code> in your Chrome browser.</li>
<li> <strong>Enable Developer Mode:</strong> Toggle the "Developer mode" switch, usually found in the top-right corner.</li>
<li> <strong>Load Unpacked Extension:</strong> Click the "Load unpacked" button.</li>
<li> <strong>Select Extension Directory:</strong> Browse to and select the <code>focus-roast/extension</code> folder from your cloned repository.</li>
</ol>

<p>The Focus-Roast icon should now appear in your browser's toolbar.</p>

<h2>
  
  
  🚀 How to Play (and Get Roasted)
</h2>

<ol>
<li> <strong>Click the Extension Icon:</strong> Locate and click the Focus-Roast icon in your Chrome toolbar.</li>
<li> <strong>Enter Your Goal:</strong> In the pop-up, clearly state your current productivity goal (e.g., "Finish my Resume," "Prepare for API interview").</li>
<li> <strong>Click "Lock In 🔒":</strong> This activates Focus-Roast.</li>
<li> <strong>Test Your Willpower:</strong> Attempt to open a distracting site like Twitter, Instagram, Reddit, or YouTube.</li>
<li> <strong>Brace for Impact:</strong> Experience the full force of AI-powered disappointment.</li>
</ol>

<h2>
  
  
  Why Local? Privacy and Control
</h2>

<p>One of the primary design decisions for Focus-Roast was to ensure complete privacy. By running the backend locally, your browsing habits and AI interactions never leave your machine. There are no external servers logging your activity, providing peace of mind alongside unparalleled productivity enforcement.</p>

<h2>
  
  
  License
</h2>

<p>This project is released under the <strong>MIT License</strong>. Feel free to fork it, modify it, and contribute to making it even more potent (or, dare I say, even meaner). Your pull requests for new roast categories, improved AI prompts, or additional shame mechanisms are highly encouraged!</p>




<p><strong>Tags:</strong> #productivity #ai #chrome-extension #python #gemini-api #fastapi #open-source #tutorial</p>

<p>🔗 <a href="https://github.com/Shashank0701-byte/roast-me" rel="noopener noreferrer">https://github.com/Shashank0701-byte/roast-me</a></p>

