---
Title: I Built an Honest AI Agent to Fight Hallucinations. Here's How It Works.
Description: 
Author: Pavel
Date: 2025-08-14T21:43:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hello, everyone!</p>

<p>We've all been amazed by Large Language Models. They can generate code, poetry, and coherent essays seemingly out of thin air. But I’ve always been bothered by their dark side: a tendency to "hallucinate." Ask a question about an obscure topic, and a model might confidently invent a non-existent term or "fact."</p>

<p>I decided to tackle this problem head-on and came to a simple conclusion: <strong>the process is more important than size, and honesty is more important than omniscience.</strong></p>

<p>Instead of relying on a giant model that <em>knows everything</em>, I created <code>WeblookAI</code>—an autonomous agent that starts by knowing <em>nothing</em>. Its only power lies in a clever algorithm I designed, which allows it to find, analyze, and synthesize information, all while remaining intellectually honest.</p>

<p>This post is the story of how I built it. I believe this project will be especially interesting for learners, as it clearly demonstrates how you can achieve impressive results with minimal resources. And yes, I asked my local neural network what it thinks of the agent—it's a fan ;)</p>

<h4>
  
  
  The Problem: A Passive "Know-it-all" vs. My Active "Researcher"
</h4>

<p>The standard approach to AI is to build a colossal library of knowledge and force a model to memorize it. My approach is inverted. I didn't want to build another librarian; I wanted to build a detective with nothing but a notebook, a magnifying glass, and endless curiosity.</p>

<p>I built <code>WeblookAI</code> on three core principles that make it unique:</p>

<p><strong>1. Intellectual Honesty — My Guiding Principle</strong></p>

<p>This isn't just a feature; it's the foundation. I designed the agent's main advantage to be its ability to <strong>minimize hallucinations</strong>. I achieved this through a strict, final-step prompt: "Answer the question based <em>only</em> on the provided facts. If a direct answer isn't available, say so."</p>

<p>The results are fantastic. When I asked it about "Wise JSON," it didn't invent a new framework. Instead, it honestly reported that it found separate data on "WISE" and "JSON." It just so happens that Wise JSON is another project of mine, but the internet doesn't know much about it yet—and my agent correctly reflected that reality.</p>

<p><strong>2. My Homegrown Web Agent — The "Magnifying Glass and Notebook"</strong></p>

<p>Instead of paying for search APIs, I built a unique data-gathering engine from scratch using <code>Puppeteer</code>. It doesn't just "search"; it conducts a genuine, almost human-like investigation I programmed it to perform:</p>

<ul>
<li>  <strong>Fault Tolerance:</strong> If one "fishing spot" (Brave Search) is empty, the agent doesn't give up. It automatically moves to the next one (DuckDuckGo, StartPage).</li>
<li>  <strong>"Human-like" Emulation:</strong> I made the agent present itself to websites as a standard Chrome browser on Windows, allowing it to bypass basic anti-bot defenses.</li>
<li>  <strong>"Smart" Filtering:</strong> I abandoned fragile CSS selectors that break with every site redesign. Instead, I programmed my agent to grab <em>all</em> links on a page and apply its own heuristic algorithm: it discards ads and junk using a domain blacklist, analyzes URLs for "usefulness," and assigns a "relevance score" to each link.</li>
</ul>

<p><strong>3. 100% Local and Autonomous — My Commitment to Independence</strong></p>

<ul>
<li>  <strong>A Local Brain:</strong> The entire "thinking" process—from planning the search to synthesizing the final answer—is handled by a language model (like <code>llama3:8b</code>) that I run on my own machine via <strong>Ollama</strong>.</li>
<li>  <strong>Privacy:</strong> Your queries and the data my agent finds never leave your computer.</li>
<li>  <strong>Zero Cost:</strong> The project I built requires no API keys, no subscriptions, and is completely independent of the whims of cloud services.</li>
</ul>

<h4>
  
  
  How I Made It Work in Practice
</h4>

<p>I broke down the entire process into three crystal-clear steps:</p>

<ol>
<li> <strong>DECOMPOSE:</strong> The local LLM receives the user's question and breaks it down into 3-4 simple search queries, creating a research plan.</li>
<li> <strong>INVESTIGATE:</strong> My <code>WebAgent</code> on <code>Puppeteer</code> heads into the field—methodically cycling through search engines, filtering links, visiting websites, and extracting clean, core text content.</li>
<li> <strong>SYNTHESIZE:</strong> The local LLM receives the original question and the collected "evidence" with one ironclad rule I gave it: do not invent.</li>
</ol>

<h4>
  
  
  Who I Built This Project For
</h4>

<p>First and foremost, for learners. For anyone who wants to understand that AI is not just about giant models but also about elegant, effective algorithms. I've open-sourced it on GitHub so you can download it, run it with a single command, and see firsthand how you can achieve impressive results by making a small model "think" the right way.</p>




<p><strong>⚠️ Important Notice:</strong><br>
This project and its web scraping agent are intended <strong>for educational purposes only</strong>. Please be respectful of the websites the agent visits. You are solely responsible for your actions and any use of this tool.</p>

<p>You can find the entire project on my GitHub:<br>
<strong><a href="https://github.com/Xzdes/weblookai" rel="noopener noreferrer">https://github.com/Xzdes/weblookai</a></strong></p>

<p>Thanks for reading! I hope this story inspires you to start your own experiments. Feel free to ask questions in the comments.</p>

