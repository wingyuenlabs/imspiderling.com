---
Title: Building MoodMatch: An AI Agent That Understands Your Emotions
Description: 
Author: Olaitan34 
Date: 2025-11-03T21:44:49.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>MoodMatch is an AI-powered agent that analyzes your emotional state and provides personalized recommendations for music, movies, and books. Built for the HNG Stage 3 Backend Task, it demonstrates the power of the A2A (Agent-to-Agent) protocol and AI integration.</p>

<h2>
  
  
  What MoodMatch Does
</h2>

<ul>
<li>Analyzes user messages to detect emotions</li>
<li>Supports 52 different mood categories</li>
<li>Recommends music from Spotify</li>
<li>Suggests movies from TMDB</li>
<li>Recommends books from Google Books</li>
<li>Provides direct, clickable links to all recommendations</li>
</ul>

<h2>
  
  
  Technical Stack
</h2>

<ul>
<li>
<strong>Backend</strong>: Python + FastAPI</li>
<li>
<strong>AI</strong>: Google Gemini 2.5 Flash for mood analysis</li>
<li>
<strong>APIs</strong>: Spotify, TMDB, Google Books</li>
<li>
<strong>Protocol</strong>: A2A (Agent-to-Agent) with JSON-RPC 2.0</li>
<li>
<strong>Deployment</strong>: Leapcell (serverless platform)</li>
<li>
<strong>Integration</strong>: Telex.im messaging platform</li>
</ul>

<h2>
  
  
  Key Features
</h2>

<ol>
<li>
<strong>Smart Mood Detection</strong>: Even if you say "I need money," it understands you're stressed</li>
<li>
<strong>52 Mood Categories</strong>: From happy to bittersweet, covering complex emotions</li>
<li>
<strong>Multi-source Recommendations</strong>: Music, movies, and books in one response</li>
<li>
<strong>Context-Aware</strong>: Considers time of day and emotion intensity</li>
<li>
<strong>Direct Links</strong>: Click and enjoy your recommendations immediately</li>
</ol>

<h2>
  
  
  Technical Challenges &amp; Solutions
</h2>

<h3>
  
  
  Challenge 1: A2A Protocol Learning Curve
</h3>

<p>The A2A protocol uses JSON-RPC 2.0, which was new to me. Solution: Studied the spec, implemented proper request/response handling, and tested with Postman.</p>

<h3>
  
  
  Challenge 2: Mood Detection Accuracy
</h3>

<p>Getting AI to map free-form text to specific moods was tricky. Solution: Used Gemini 2.5 Flash with structured output and fuzzy matching for fallback.</p>

