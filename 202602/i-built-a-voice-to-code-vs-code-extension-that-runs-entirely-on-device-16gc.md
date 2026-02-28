---
Title: I Built a Voice-to-Code VS Code Extension That Runs Entirely On-Device
Description: 
Author: Nate Archer
Date: 2026-02-28T21:50:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>Every AI coding assistant requires typing. GitHub Copilot, Continue, Kiro — they all expect you to type your prompts. But what if you could just talk?</p>

<p>That's why I built VoxPilot.</p>

<h2>
  
  
  The Problem
</h2>

<p>I spend a lot of time typing prompts like "refactor this function to use async/await with proper error handling and add unit tests." That's 15 seconds of typing for something I could say in 3 seconds.</p>

<p>For developers with RSI or carpal tunnel, the problem is worse. Typing isn't just slow — it's painful.</p>

<h2>
  
  
  The Solution
</h2>

<p>VoxPilot is a VS Code extension that captures your voice, transcribes it locally using Moonshine ASR, and sends the text to your coding assistant.</p>

<p>The key word is "locally." Your audio never leaves your machine. There are no API keys, no cloud calls, no telemetry. The ASR model is 27MB and runs via ONNX Runtime.</p>

<h2>
  
  
  How It Works
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Microphone → PCM Audio → Voice Activity Detection → Moonshine ASR → Text → VS Code Chat
</code></pre>

</div>



<ol>
<li><p><strong>Audio Capture</strong>: Native CLI tools (arecord on Linux, sox on macOS, ffmpeg on Windows) capture raw PCM audio at 16kHz.</p></li>
<li><p><strong>Voice Activity Detection</strong>: An energy-based VAD detects when you start and stop speaking. No need to press a button — just talk.</p></li>
<li><p><strong>Transcription</strong>: Moonshine's encoder-decoder architecture processes the audio through ONNX Runtime. The Tiny model (27MB) handles quick commands; the Base model (65MB) is better for longer dictation.</p></li>
<li><p><strong>Delivery</strong>: The transcript goes to VS Code's Chat API, targeting whatever participant you've configured (Copilot, Continue, etc.).</p></li>
</ol>

<h2>
  
  
  Privacy
</h2>

<p>This was non-negotiable. Voice data is sensitive. VoxPilot processes everything in-memory and never writes audio to disk or sends it over the network.</p>

<h2>
  
  
  Try It
</h2>

<ul>
<li>
<strong>Open VSX</strong>: <a href="https://open-vsx.org/extension/natearcher-ai/voxpilot" rel="noopener noreferrer">https://open-vsx.org/extension/natearcher-ai/voxpilot</a>
</li>
<li>
<strong>GitHub</strong>: <a href="https://github.com/natearcher-ai/voxpilot" rel="noopener noreferrer">https://github.com/natearcher-ai/voxpilot</a>
</li>
</ul>

<p>MIT licensed. PRs welcome. Star the repo if it's useful.</p>

