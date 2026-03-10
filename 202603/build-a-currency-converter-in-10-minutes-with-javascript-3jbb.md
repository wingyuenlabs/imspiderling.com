---
Title: Build a Currency Converter in 10 Minutes with JavaScript
Description: 
Author: 200 OK
Date: 2026-03-10T22:10:54.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever needed to add currency conversion to your app? Whether you're building an e-commerce platform, a travel app, or a finance dashboard — you'll need reliable exchange rates.</p>

<p>In this tutorial, I'll show you how to build a working currency converter in under 10 minutes using vanilla JavaScript and the <a href="https://rapidapi.com/twohundredok-twohundredok-default/api/currency-exchange-rates13" rel="noopener noreferrer">Currency Exchange Rates API</a> on RapidAPI.</p>

<h2>
  
  
  What We're Building
</h2>

<p>A simple web-based currency converter that:</p>

<ul>
<li>Converts between 31 major currencies</li>
<li>Shows real-time rates from the European Central Bank (ECB)</li>
<li>Updates instantly when you change the amount or currency</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/placeholder-screenshot.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/placeholder-screenshot.png" alt="Currency Converter Demo" width="800" height="400"></a></p>

<h2>
  
  
  Prerequisites
</h2>

<ul>
<li>Basic JavaScript knowledge</li>
<li>A free RapidAPI account (<a href="https://rapidapi.com/auth/sign-up" rel="noopener noreferrer">sign up here</a>)</li>
<li>A text editor and browser</li>
</ul>

<h2>
  
  
  Step 1: Get Your API Key (1 minute)
</h2>

<ol>
<li>Head to the <a href="https://rapidapi.com/twohundredok-twohundredok-default/api/currency-exchange-rates13" rel="noopener noreferrer">Currency Exchange Rates API</a> on RapidAPI</li>
<li>Click <strong>Subscribe to Test</strong> and select the <strong>Free plan</strong> (100 requests/day — plenty for development)</li>
<li>Copy your <code>X-RapidAPI-Key</code> from the code snippets section</li>
</ol>

<h2>
  
  
  Step 2: Build the HTML (3 minutes)
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
  &lt;title&gt;Currency Converter&lt;/title&gt;
  &lt;style&gt;
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      display: flex; justify-content: center; align-items: center;
      min-height: 100vh; background: #f0f2f5;
    }
    .converter {
      background: white; padding: 2rem; border-radius: 12px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1); width: 400px;
    }
    h1 { font-size: 1.5rem; margin-bottom: 1.5rem; text-align: center; }
    .field { margin-bottom: 1rem; }
    label { display: block; font-size: 0.875rem; color: #666; margin-bottom: 4px; }
    input, select {
      width: 100%; padding: 10px 12px; border: 1px solid #ddd;
      border-radius: 8px; font-size: 1rem;
    }
    .result {
      text-align: center; padding: 1.5rem; background: #f8f9fa;
      border-radius: 8px; margin-top: 1rem;
    }
    .result .amount { font-size: 2rem; font-weight: 700; color: #2563eb; }
    .result .rate { font-size: 0.875rem; color: #666; margin-top: 4px; }
    .swap-btn {
      display: block; margin: 0.5rem auto; background: none;
      border: 1px solid #ddd; border-radius: 50%; width: 36px;
      height: 36px; cursor: pointer; font-size: 1.2rem;
    }
    .swap-btn:hover { background: #f0f2f5; }
  &lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;div class="converter"&gt;
    &lt;h1&gt;💱 Currency Converter&lt;/h1&gt;
    &lt;div class="field"&gt;
      &lt;label&gt;Amount&lt;/label&gt;
      &lt;input type="number" id="amount" value="100" min="0" step="0.01"&gt;
    &lt;/div&gt;
    &lt;div class="field"&gt;
      &lt;label&gt;From&lt;/label&gt;
      &lt;select id="fromCurrency"&gt;&lt;/select&gt;
    &lt;/div&gt;
    &lt;button class="swap-btn" id="swap" title="Swap currencies"&gt;⇅&lt;/button&gt;
    &lt;div class="field"&gt;
      &lt;label&gt;To&lt;/label&gt;
      &lt;select id="toCurrency"&gt;&lt;/select&gt;
    &lt;/div&gt;
    &lt;div class="result"&gt;
      &lt;div class="amount" id="resultAmount"&gt;—&lt;/div&gt;
      &lt;div class="rate" id="resultRate"&gt;&lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
  &lt;script src="app.js"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

</div>

