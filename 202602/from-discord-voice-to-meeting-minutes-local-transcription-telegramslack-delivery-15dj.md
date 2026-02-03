---
Title: From Discord Voice to Meeting Minutes: Local Transcription + Telegram/Slack Delivery
Description: 
Author: Paul
Date: 2026-02-03T21:12:27.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  discord2sum — meeting minutes for Discord voice, without the busywork
</h1>

<p>Discord voice calls are where decisions get made — and also where decisions get lost.</p>

<p>Someone says “let’s do it this way”, someone else agrees, a couple of tasks are assigned… and then the call ends. A day later you remember <em>that</em> you agreed on something, but not the exact details, who volunteered for what, or what the next step was.</p>

<p>That’s the problem <strong>discord2sum</strong> solves.</p>

<p>Repo: <a href="https://github.com/toxed/discord2sum" rel="noopener noreferrer">https://github.com/toxed/discord2sum</a></p>

<p>It’s a small Discord bot that quietly turns voice conversations into something you can actually refer to later: a short, structured summary delivered to your chat.</p>




<h2>
  
  
  What the bot feels like in daily use
</h2>

<p>You hop into a voice channel with your team.</p>

<p>When people start talking, the bot joins in the background. It doesn’t interrupt and doesn’t try to be clever in real time. It just listens.</p>

<p>When the channel becomes empty, you get a message with “minutes”:</p>

<ul>
<li>what the call was about (a concise summary)</li>
<li>what was decided</li>
<li>what’s next (action items)</li>
<li>who spoke / who was present (where possible)</li>
</ul>

<p>You end up with a lightweight paper trail that’s easy to scan, easy to search, and easy to forward.</p>




<h2>
  
  
  Privacy-first by design
</h2>

<p>A lot of call‑summary products work by uploading your audio somewhere and letting a cloud service do the transcription.</p>

<p><strong>discord2sum is built around local processing</strong>:</p>

<ul>
<li>audio is recorded and processed on your own machine/server</li>
<li>speech-to-text runs locally (Whisper-based)</li>
<li>transcripts can be stored locally with retention limits</li>
</ul>

<p>For the summary step, you can choose what you’re comfortable with:</p>

<ul>
<li>use a hosted LLM (OpenAI) for higher-quality summaries, or</li>
<li>use a local LLM endpoint (e.g. Ollama) for an all‑local pipeline</li>
</ul>




<h2>
  
  
  Where the summary can go
</h2>

<p>Telegram is the default because it’s simple and fast.</p>

<p>But the bot can also deliver the same summary to:</p>

<ul>
<li>
<strong>Slack</strong> (via Incoming Webhooks)</li>
<li>
<strong>any HTTP webhook</strong> (as JSON), if you want to push minutes into your own tooling</li>
</ul>

<p>That makes it easy to route “call outcomes” into whatever system your team already uses.</p>




<h2>
  
  
  Why I like this approach
</h2>

<p>It’s intentionally small and opinionated.</p>

<p>No calendar integration. No meeting invites. No complex UI.</p>

<p>Just one job: <strong>when a Discord voice call ends, send a clear summary</strong>.</p>

<p>That’s enough to stop decisions from disappearing — without adding process or forcing people to change how they talk.</p>

