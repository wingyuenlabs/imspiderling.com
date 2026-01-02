---
Title: I built a Chrome extension that saves me from copy-pasting text to ChatGPT every time
Description: 
Author: Faxziah
Date: 2026-01-02T21:48:46.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  How I Stopped Wasting Time Copy-Pasting Text to ChatGPT
</h1>

<p>If you work with text and use ChatGPT, DeepSeek, or other AI tools — you know this pain.</p>

<p>I dealt with it until I snapped and built a Chrome extension that solves it in one click. Been using it for a couple months now and loving the time saved: <a href="https://chromewebstore.google.com/detail/ai-text-tools-summarize-r/lafblpkfbhdbcehcehdlkkhiniijpcmh" rel="noopener noreferrer">AI Text Tools on Chrome Web Store</a>.</p>

<h2>
  
  
  How I Got Here
</h2>

<p>I work with text every day. Reading articles in English — need translation. Writing an email to a client — want to check my wording. Studying documentation — need a quick summary. AI is everywhere.</p>

<p>And every single time it's the same routine:</p>

<ol>
<li>Select text</li>
<li>Copy</li>
<li>Open ChatGPT</li>
<li>Paste</li>
<li>Write a prompt</li>
<li>Wait</li>
<li>Copy the response</li>
<li>Switch back to the page</li>
</ol>

<p>Sound familiar?</p>

<p>At some point I caught myself thinking: I'm spending more time copying than actually working. It was annoying.</p>

<h2>
  
  
  The Solution Seemed Obvious
</h2>

<p>Select text → click a button → get the result. Right on the page. No tab switching.</p>

<p>And... gradually the extension was ready.</p>

<h2>
  
  
  What AI Text Tools Does
</h2>

<p>Select any text on a page — a menu with actions appears:</p>

<p><strong>1. Summarize</strong> — compresses a wall of text to its essence. Great for long articles and documentation.</p>

<p><strong>2. Rephrase</strong> — rewrites text in different words. I use it for emails when I want to sound more formal or casual.</p>

<p><strong>3. Translate</strong> — translation to many languages. Works both ways.</p>

<p><strong>4. Discuss</strong> — opens an AI chat right on the page. You can ask questions about the selected text.</p>

<p><strong>5. Highlight Key Paragraphs</strong> — if you're worried about AI hallucinations and want to read the text yourself, but without all the fluff — perfect feature.</p>

<p>Everything runs through Google's Gemini API. You just need an API key (free).</p>

<h2>
  
  
  What It Looks Like
</h2>

<p>Say I'm reading an article in a foreign language. I select a paragraph, a modal with a menu appears, I click Translate — in a second I see the translation right below the text. I don't leave the page, don't lose context.</p>

<p>Or I receive a long email. Select it, hit Summarize — get the gist in three sentences. Then decide if it's worth reading in full.</p>

<p>These little things actually save quite a bit of time and energy each day. Under the hood, it uses Gemini models (LLM from Google), accessed via API.</p>

<h2>
  
  
  Why Gemini and Not ChatGPT?
</h2>

<p>The main reason — free API.</p>

<p>OpenAI's ChatGPT doesn't have a free tier. Google does. For most tasks it's more than enough. So I managed to keep the extension completely free, as originally planned.</p>

<p>Also, this meant I didn't need to connect to any external server. Your computer talks directly to Gemini via API, so the extension doesn't collect or store any user data.</p>

<p>Plus, Gemini's responses are generally just as good as ChatGPT's.</p>

<h2>
  
  
  What's Next
</h2>

<p>The extension is free, but I really want to add:</p>

<ul>
<li>
<strong>Support for other models</strong> (DeepSeek, ChatGPT, Claude, etc.) — users could add their own API keys, keeping the extension server-free</li>
<li>
<strong>Custom actions</strong> — so everyone can create their own prompts, like "Shorten to 280 characters for a tweet"</li>
<li>
<strong>Keyboard shortcuts</strong> — for even faster AI access</li>
</ul>

<p>If you want to try it: <a href="https://chromewebstore.google.com/detail/ai-text-tools-summarize-r/lafblpkfbhdbcehcehdlkkhiniijpcmh" rel="noopener noreferrer">AI Text Tools on Chrome Web Store</a></p>

<p>I'd love your feedback in the comments. What are your most common text tasks that could be automated?</p>

<p>Install, try it out, and leave a review!</p>

<p>Thanks for reading.</p>

