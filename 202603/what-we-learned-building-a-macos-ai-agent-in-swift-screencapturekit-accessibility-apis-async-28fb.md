---
Title: What We Learned Building a macOS AI Agent in Swift (ScreenCaptureKit, Accessibility APIs, Async Pipelines)
Description: 
Author: Matthew Diakonov
Date: 2026-03-17T21:41:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>We have been building <a href="https://github.com/m13v/fazm" rel="noopener noreferrer">Fazm</a> for about six months - a macOS desktop agent that uses voice input to control your computer. Along the way we ran into a lot of Swift-specific challenges that we did not see documented anywhere. Here is what we learned.</p>

<h2>
  
  
  ScreenCaptureKit for Real-Time Screen Capture
</h2>

<p>The first problem is capturing what is on the screen. We needed something fast enough for real-time use but lightweight enough to run continuously without killing battery life.</p>

<p>ScreenCaptureKit was the answer. Apple introduced it in macOS 12.3 as a modern replacement for the older CGWindowListCreateImage approach. The key advantages for an AI agent:</p>

<ul>
<li>
<strong>Per-window filtering.</strong> You can capture specific windows instead of the entire screen, which means less data to process and better privacy.</li>
<li>
<strong>Hardware-accelerated.</strong> The capture pipeline runs on the GPU, so CPU overhead stays minimal even at high frame rates.</li>
<li>
<strong>CMSampleBuffer output.</strong> You get raw pixel buffers that you can feed directly to vision models without image format conversion overhead.</li>
</ul>

<p>The tricky part was getting the update frequency right. Too slow and the agent misses UI state changes. Too fast and you waste compute processing identical frames. We settled on adaptive capture - high frequency during active automation, low frequency during idle monitoring.</p>

<h2>
  
  
  Accessibility APIs Beat Screenshots Every Time
</h2>

<p>Early on we tried the screenshot-and-OCR approach that most AI computer agents use. Take a picture of the screen, send it to a vision model, get back coordinates of where to click. It works in demos. It breaks in production.</p>

<p>The problems are fundamental:</p>

<ul>
<li>
<strong>Resolution dependence.</strong> A button that is 40px wide on a Retina display looks different at every scale factor.</li>
<li>
<strong>Visual ambiguity.</strong> Two buttons that look identical to a vision model might do completely different things.</li>
<li>
<strong>Fragility.</strong> Every UI update, theme change, or notification banner shifts pixel coordinates.</li>
</ul>

<p>macOS accessibility APIs solve all of this. Instead of guessing what a UI element is based on its appearance, you query it directly by role, label, and position in the element hierarchy. A "Save" button is always <code>AXButton</code> with <code>AXTitle: "Save"</code>, regardless of what it looks like.</p>

<p>The accessibility tree gives you:</p>

<ul>
<li>Element roles (button, text field, menu item)</li>
<li>Labels and values</li>
<li>Hierarchical relationships (this button is inside this toolbar inside this window)</li>
<li>Actionable operations (press, set value, select)</li>
</ul>

<p>This is the same interface screen readers use, which means it is designed to be stable across visual changes. When an app redesigns their UI, the accessibility tree usually stays the same. We wrote a detailed comparison of this approach versus screenshot-based vision in <a href="https://fazm.ai/blog/how-ai-agents-see-your-screen-dom-vs-screenshots" rel="noopener noreferrer">how AI agents see your screen</a>.</p>

<h2>
  
  
  Swift Concurrency for the Pipeline
</h2>

<p>A desktop AI agent has a natural pipeline: capture screen, process with LLM, execute actions, verify result. Each stage has different latency characteristics and failure modes.</p>

<p>Swift's structured concurrency turned out to be a great fit:</p>

<ul>
<li>
<strong>AsyncStream</strong> for the capture pipeline - frames flow continuously from ScreenCaptureKit.</li>
<li>
<strong>Task groups</strong> for parallel action execution - clicking a button and verifying the result can happen concurrently.</li>
<li>
<strong>Actor isolation</strong> for state management - the agent's understanding of the current screen state needs to be thread-safe.</li>
</ul>

<p>The biggest lesson was to not fight the concurrency model. Early versions tried to use completion handlers and dispatch queues, and the code was a mess. Rewriting to async/await made the pipeline logic readable and the error handling straightforward.</p>

<h2>
  
  
  Voice Input with WhisperKit
</h2>

<p>For voice control, we use WhisperKit running locally on Apple Silicon. The key decision was local vs cloud transcription. Cloud gives you better accuracy. Local gives you zero latency and complete privacy.</p>

<p>For a desktop agent, privacy wins. The agent already sees everything on your screen - sending audio to a cloud service on top of that was a non-starter. WhisperKit on an M1 gives us usable transcription speed with good-enough accuracy for command input.</p>

<h2>
  
  
  The Architecture Summary
</h2>

<p>The full pipeline looks like:</p>

<ol>
<li>
<strong>WhisperKit</strong> captures and transcribes voice input locally</li>
<li>
<strong>ScreenCaptureKit</strong> grabs the current screen state</li>
<li>
<strong>Accessibility APIs</strong> build a structured representation of the UI</li>
<li>
<strong>LLM</strong> (Claude or local model via Ollama) plans the actions</li>
<li>
<strong>Accessibility APIs</strong> execute the actions (click, type, select)</li>
<li>
<strong>ScreenCaptureKit</strong> verifies the result</li>
</ol>

<p>Each step is an async operation in a structured concurrency pipeline. Failures at any stage cancel downstream work cleanly.</p>




<p><em>Fazm is <a href="https://github.com/m13v/fazm" rel="noopener noreferrer">open source on GitHub</a> - MIT licensed, written entirely in Swift/SwiftUI.</em></p>

