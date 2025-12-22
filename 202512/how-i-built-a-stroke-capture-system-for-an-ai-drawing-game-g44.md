---
Title: How I Built a Stroke Capture System for an AI Drawing Game
Description: 
Author: Adam
Date: 2025-12-22T21:31:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>Most drawing apps save the final image. I wanted to capture the whole journey.</p>

<p>For <a href="https://paintyourdate.io" rel="noopener noreferrer">Paint Your Partner</a>, I built a system that records every single brush stroke. Position, timing, pressure, colour, order. Everything.</p>

<p>Why? Because an AI is watching. And it needs to see the drawing happen, not just the end result.</p>

<h2>
  
  
  The Problem
</h2>

<p>I was building a couples drawing game. You draw your partner, they draw you, an AI commentates live.</p>

<p>But if the AI only sees the finished image, the commentary is boring. "Nice portrait." Cool. Thanks.</p>

<p>I wanted it to react as you draw. Spot the eyes appearing. Notice you're spending ages on the hair. Catch the moment you give up on the nose and scribble something abstract.</p>

<p>For that, I needed stroke data.</p>

<h2>
  
  
  What Gets Captured
</h2>

<p>Every stroke is an object. It contains:</p>

<ul>
<li>
<strong>Path data</strong> (array of x/y coordinates)</li>
<li><strong>Colour</strong></li>
<li><strong>Brush size</strong></li>
<li>
<strong>Timestamp</strong> (when the stroke started)</li>
<li>
<strong>Duration</strong> (how long it took)</li>
<li>
<strong>Order</strong> (which stroke number this was)</li>
</ul>

<p>Nothing clever here. Just capture everything and store it.</p>

<p>I've got over 1,000 drawings in the database now, each with complete stroke history.</p>

<h2>
  
  
  Replay
</h2>

<p>With full stroke data, I can replay any drawing as a video. Speed it up, slow it down, export as GIF or MP4.</p>

<p>This is useful for:</p>

<ul>
<li>Shareable content (watch someone draw their partner in 10 seconds)</li>
<li>AI training data (if I ever want to fine-tune)</li>
<li>Debugging (see exactly what a user drew and when)</li>
</ul>

<h2>
  
  
  Real-Time AI Commentary
</h2>

<p>The AI watches strokes arrive in real-time. It doesn't wait for you to finish.</p>

<p>As features appear on the canvas (eyes, hair, smile), it fires off short commentary. One or two sentences. Cheeky, not critical.</p>

<p><em>"Ooh those eyes are looking dreamy!"</em></p>

<p><em>"Someone's really capturing that smile!"</em></p>

<p>The stroke data tells the AI what's being drawn and how. Hesitation, confidence, chaos. It's all in the timing.</p>

<h2>
  
  
  The Engine
</h2>

<p>Paint Your Partner runs on the same engine as Artbitrator (my multiplayer drawing game, like Quick Draw but with AI judging). Same stroke capture. Same replay system. Different game modes.</p>

<p>Building the core engine once and spinning up different products on top has been worth the upfront effort.</p>

<h2>
  
  
  What's Next
</h2>

<p>Camera capture. Sync the players' webcam feeds with the stroke replay and export the whole thing as one video. TikTok-ready couples content.</p>

<p>Not built yet. But the stroke system makes it possible.</p>

<h2>
  
  
  Try It
</h2>

<p><a href="https://paintyourdate.io" rel="noopener noreferrer">paintyourdate.io</a></p>

<p>Draw your partner. Let the AI commentate. See how bad you really are at noses.</p>

