---
Title: Build a UV Exposure Tracker in 15 Minutes with JavaScript
Description: 
Author: 200 OK
Date: 2026-03-10T22:10:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>Skin cancer is the most common cancer worldwide, yet most apps still ignore UV data. Whether you're building a health tracker, a weather app, or a fitness platform — real-time UV index data with personalized safe exposure times makes your app genuinely useful.</p>

<p>In this tutorial, I'll show you how to build a UV exposure tracker using vanilla JavaScript and the <a href="https://rapidapi.com/twohundredok-twohundredok-default/api/uv-index-sun-exposure" rel="noopener noreferrer">UV Index &amp; Sun Exposure API</a> on RapidAPI.</p>

<h2>
  
  
  What We're Building
</h2>

<p>A tracker that shows:</p>

<ul>
<li>Current UV index with color-coded risk level</li>
<li>Health advice (sunscreen, shade, protective clothing)</li>
<li>Safe exposure time personalized by Fitzpatrick skin type</li>
<li>Vitamin D synthesis estimate</li>
<li>3-day hourly UV forecast chart</li>
</ul>

<p><strong>No frameworks.</strong> Just HTML, CSS, and vanilla JavaScript.</p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>A free RapidAPI account (<a href="https://rapidapi.com/auth/sign-up" rel="noopener noreferrer">sign up here</a>)</li>
<li>Subscribe to the <a href="https://rapidapi.com/twohundredok-twohundredok-default/api/uv-index-sun-exposure" rel="noopener noreferrer">UV Index &amp; Sun Exposure API</a> (free tier — 100 requests/day)</li>
<li>A text editor and browser</li>
</ul>

<h2>
  
  
  Step 1: Set Up the HTML
</h2>

<p>Create an <code>index.html</code> file:</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
html
&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
  &lt;meta charset="UTF-8"&gt;
  &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
  &lt;title&gt;UV Exposure Tracker&lt;/title&gt;
  &lt;style&gt;
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #0f172a; color: #e2e8f0; min-height: 100vh; padding: 2rem; }
    .container { max-width: 640px; margin: 0 auto; }
    h1 { font-size: 1.5rem; margin-bottom: 1.5rem; text-align: center; }

    .controls { display: flex; gap: 0.5rem; margin-bottom: 1.5rem; flex-wrap: wrap; }
    .controls input, .controls select, .controls button { padding: 0.6rem 1rem; border-radius: 8px; border: 1px solid #334155; background: #1e293b; color: #e2e8f0; font-size: 0.9rem; }
    .controls input { flex: 1; min-width: 80px; }
    .controls button { background: #f59e0b; color: #0f172a; font-weight: 600; border: none; cursor: pointer; }
    .controls button:hover { background: #d97706; }

    .uv-card { background: #1e293b; border-radius: 12px; padding: 1.5rem; margin-bottom: 1rem; }
    .uv-main { display: flex; align-items: center; gap: 1.5rem; margin-bottom: 1rem; }
    .uv-circle { width: 100px; height: 100px; border-radius: 50%; display: flex; flex-direction: column; align-items: center; justify-content: center; font-weight: 700; }
    .uv-value { font-size: 2rem; line-height: 1; }
    .uv-label { font-size: 0.75rem; margin-top: 4px; text-transform: uppercase; }
    .uv-details h3 { font-size: 1.1rem; margin-bottom: 0.25rem; }
    .uv-details p { font-size: 0.85rem; color: #94a3b8; line-height: 1.5; }

    .exposure-card { background: #1e293b; border-radius: 12px; padding: 1.5rem; margin-bottom: 1rem; }
    .exposure-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; margin-top: 0.75rem; }
    .exposure-stat { background: #0f172a; border-radius: 8px; padding: 1rem; text-align: center; }
    .exposure-stat .value { font-size: 1.5rem; font-weight: 700; color: #f59e0b; }
    .exposure-stat .label { font-size: 0.75rem; color: #94a3b8; margin-top: 0.25rem; }

    .forecast-card { background: #1e293b; border-radius: 12px; padding: 1.5rem; }
    .forecast-bars { display: flex; align-items: flex-end; gap: 4px; height: 120px; margin-top: 1rem; }
    .forecast-bar { flex: 1; border-radius: 4px 4px 0 0; position: relative; min-width: 8px; }
    .forecast-bar .time { position: absolute; bottom: -18px; left: 50%; transform: translateX(-50%); font-size: 0.6rem; color: #64748b; white-space: nowrap; }

    .hidden { display: none; }
    .loading { text-align: center; padding: 2rem; color: #64748b; }
  &lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;div class="container"&gt;
    &lt;h1&gt;UV Exposure Tracker&lt;/h1&gt;
    &lt;div class="controls"&gt;
      &lt;input type="text" id="lat" placeholder="Latitude" value="40.7128"&gt;
      &lt;input type="text" id="lon" placeholder="Longitude" value="-74.0060"&gt;
      &lt;select id="skinType"&gt;
        &lt;option value="1"&gt;Type I — Very fair&lt;/option&gt;
        &lt;option value="2" selected&gt;Type II — Fair&lt;/option&gt;
        &lt;option value="3"&gt;Type III — Medium&lt;/option&gt;
        &lt;option value="4"&gt;Type IV — Olive&lt;/option&gt;
        &lt;option value="5"&gt;Type V — Brown&lt;/option&gt;
        &lt;option value="6"&gt;Type VI — Very dark&lt;/option&gt;
      &lt;/select&gt;
      &lt;button id="checkBtn"&gt;Check UV&lt;/button&gt;
    &lt;/div&gt;
    &lt;div id="results" class="hidden"&gt;&lt;/div&gt;
    &lt;div id="loading" class="loading hidden"&gt;Loading UV data...&lt;/div&gt;
  &lt;/div&gt;
  &lt;script src="app.js"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

</div>

