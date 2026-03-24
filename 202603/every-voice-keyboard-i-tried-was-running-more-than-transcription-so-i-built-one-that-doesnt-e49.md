---
Title: Every Voice Keyboard I Tried Was Running More Than Transcription. So I Built One That Doesn't.
Description: 
Author: Ondrej Machala
Date: 2026-03-24T21:48:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>I dictate a lot. Messages to family, notes to myself, things I'd rather say than type. At some point I got curious about what was actually happening when I used a voice keyboard app.</p>

<p>Most of them bundle analytics SDKs, crash reporters, attribution libraries. Services that have nothing to do with transcription but run in the background every time you open the keyboard. Crash reports can include fragments of text from memory. Analytics events capture metadata about what you were doing. None of it is the transcription service you signed up for.</p>

<p>I didn't want that. I wanted a keyboard that did one thing: turn my voice into text. Nothing running in the background. Nothing phoning home except the transcription itself, and only when I ask it to.</p>

<p>So I built Diction.</p>

<h2>
  
  
  Three modes, all of them clean
</h2>

<p><strong>On-device:</strong> transcription runs locally on your iPhone. Nothing leaves the device. No network request at all.</p>

<p><strong>Self-hosted:</strong> audio goes to your own server. You control what runs there. The server code is on GitHub if you want to read it.</p>

<p><strong>Diction One:</strong> my cloud, same pipeline. Audio processed in real time, not stored.</p>

<p>Pick whichever fits. Switch any time. The keyboard is identical across all three.</p>

<h2>
  
  
  Why it matters
</h2>

<p>Voice input is personal in a way that typing isn't. You speak more naturally than you write. You say things you wouldn't bother typing. That's the whole point of dictation.</p>

<p>It also means a voice keyboard that runs background analytics has access to something more candid than most apps ever see.</p>

<p>Diction doesn't do that. Transcription, nothing else.</p>

<p><a href="https://apps.apple.com/app/id6759807364" rel="noopener noreferrer">App Store</a> | <a href="https://github.com/omachala/diction" rel="noopener noreferrer">GitHub</a></p>

