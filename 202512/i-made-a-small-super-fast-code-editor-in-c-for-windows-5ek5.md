---
Title: I made a small, super fast code editor in C for Windows
Description: 
Author: Giovanni Carlino
Date: 2025-12-13T21:36:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the past few months I’ve been working on a side project called Icarus: a Windows code editor written entirely in C.</p>

<p>The idea came from a pretty simple frustration: most modern editors feel heavy. They’re powerful, but they take time to start, pull in a lot of dependencies, and often feel like total HELL when working on huge codebases.</p>

<p>I wanted something that starts instantly, stays out of the way, and feels closer to the system it runs on.</p>

<p>From the start, I set a few clear constraints:</p>

<p>Instant startup (no splash screens, no background services)<br>
Very small footprint (single EXE)<br>
Portable (no installer, no dependencies)<br>
Native Windows UI (pure Win32, no Electron or web tech)</p>

<p>I chose C mostly for control and simplicity. There’s no runtime to ship, no GC pauses, and no hidden work happening behind the scenes. What the program does is what I wrote.</p>

<p>Icarus is intentionally minimal, but it already supports:</p>

<p>Syntax highlighting<br>
Multiple tabs and split views<br>
Project-based file navigation<br>
Fast text rendering and scrolling</p>

<p>There’s no plugin system, and no AI features. If I don’t personally need something for everyday coding, it probably doesn’t belong in the editor.</p>

<p>A few things stood out while building this:</p>

<p>Startup time matters more than people think. When an editor opens instantly, it changes how often you use it.<br>
Writing everything yourself forces better decisions. You can’t hide behind libraries.<br>
Minimalism isn’t about removing features — it’s about being intentional with the ones you keep.</p>

<p>I just released Icarus v1.0. It’s Windows-only and available as a portable executable. Right now I’m mainly looking for feedback, bug reports, and ideas that fit the lightweight philosophy.</p>

<p>If you enjoy small, fast tools or you’re curious about native Windows development, I’d love to hear your thoughts.</p>

<p>Project page: <a href="https://giovannicarlino.gumroad.com/l/icarus" rel="noopener noreferrer">Gumroad Link</a></p>

<p>Happy to answer questions about the implementation, C vs C++, or anything else.</p>

