---
Title: Why File systems are hard to debug
Description: 
Author: Anant Tyagi
Date: 2026-03-23T22:00:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’m building a file system from scratch. Not because I need one—but because debugging what I can’t see is guesswork.</p>

<p>Understanding this at the file system level is my first step toward kernel-level observability with eBPF.</p>

<p>Most file systems work fine—until they don’t.</p>

<p>When something slows down or behaves unexpectedly, you don’t really know why. You just see symptoms: high disk usage, latency spikes, random slowdowns.</p>

<p>The problem is simple. The file system is a black box.</p>

<p>You can monitor CPU. You can track memory. You can inspect processes.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq9x9bz4pkdjlm05gdcsc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq9x9bz4pkdjlm05gdcsc.png" alt=" " width="484" height="373"></a></p>

<p>But what actually happens inside the file system—between a read, a write, and the disk—is mostly invisible.</p>

<p>That’s where things break.</p>

<p>Debugging turns into guessing.</p>

<p>You don’t know:</p>

<ul>
<li>which file caused the issue</li>
<li>which process triggered it</li>
<li>where the latency actually happened</li>
</ul>

<p>And that’s not a tooling problem. It’s a visibility problem.</p>

<p>So instead of just studying file systems, I decided to build one.</p>

<p>Not for performance. Not for production.</p>

<p>But for visibility.</p>

<p>The goal is simple:</p>

<ul>
<li>track every operation</li>
<li>measure latency</li>
<li>connect file activity to what caused it</li>
</ul>

<p>Make the file system explain itself.</p>

<p>This is where I start.</p>

<p>I will continue this series and make a different low level useful tools.<br>
This is where I start.</p>

<p>This is part of a larger series where I’ll be building low-level system tools from scratch—step by step—as I work toward understanding how an operating system really comes together.</p>

<p>The file system is just the beginning.</p>

<p>In this series, I’ll explore:</p>

<ul>
<li>how data is stored and managed</li>
<li>how processes interact with the system</li>
<li>how system behavior can be observed and debugged</li>
<li>and how to make these internals visible instead of opaque</li>
</ul>

<p>The goal isn’t to build a production-ready OS.</p>

<p>The goal is to understand systems deeply—and make them observable.</p>

<p>Along the way, I’ll connect these ideas to kernel-level observability using eBPF.</p>

<p>Next: starting with the disk layer.</p>

