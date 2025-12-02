---
Title: I built a Zero-Code Data Analyst
Description: 
Author: prajjawal1707
Date: 2025-12-02T21:48:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Problem: I love Python for data analysis, but I hate writing matplotlib boilerplate just to see a simple trend. I also didn't want the overhead of a full React frontend for a simple dashboard.</p>

<p>The Solution: I built Morph-AI-Era, a monolithic app using:</p>

<p>FastAPI (Backend): Handles CSV parsing (Pandas) and AI Forecasting (Scikit-learn).</p>

<p>Vanilla JS + Plotly (Frontend): Fetches JSON and renders charts instantly.</p>

<p>Supabase: For Auth and Credits.</p>

<p>How it works:</p>

<p>User drags CSV.</p>

<p>FastAPI cleans NaN values and detects date columns.</p>

<p>Frontend renders interactive Scatter/Line charts via Plotly.</p>

<p>Bonus: I added a "Viral PDF" generator using html2canvas.</p>

<p>The Result: A dashboard that feels like a SPA but runs on a simple Python backend. It handles 100k+ rows in the browser without lag.</p>

<p>Try it out: I’m looking for feedback on the forecasting logic. You can test it here (10 Free Credits for new accounts): <a href="https://www.morph-ai-era.online" rel="noopener noreferrer">https://www.morph-ai-era.online</a></p>

<p>Let me know if you want to see the source code for the "Forecast" endpoint!</p>

<h1>
  
  
  python #fastapi #webdev #showdev
</h1>

