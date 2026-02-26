---
Title: I got tired of Googling pandas methods, so I built this
Description: 
Author: Ali Aldahmani
Date: 2026-02-26T22:11:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>I built a VS Code extension that shows Python cheat sheets right next to your code.</p>

<p>So I got tired of switching between my editor and browser every time I forgot a <code>pandas</code> method or a <code>numpy</code> function. You know the drill, you're in the zone, writing code, and then you have to stop, open a new tab, search "pandas groupby example", scroll past 3 Stack Overflow answers... and by the time you get back to your code, you've lost your train of thought.</p>

<p>So I built something to fix that.</p>

<p>DevLens is a VS Code extension that opens a cheat sheet panel right beside your code. It covers HTML, CSS, Tailwind, NumPy, Pandas, Matplotlib, Seaborn, and Scikit-learn (for now).</p>

<p>The part I'm most happy with is the auto-detection. When you open a .py file, it scans your imports and automatically switches to the right library. Open a file with import pandas as pd, and it's already showing you pandas snippets. No clicking, no selecting.</p>

<p>Every snippet has two buttons: Insert drops it directly at your cursor, and Copy puts it on your clipboard. Small thing, but it saves a surprising amount of time.</p>

<p>What's next:</p>

<p>More Python libraries (requests, os, datetime...)<br>
JavaScript and TypeScript support<br>
Java support<br>
Install the library button directly from the panel</p>

<p>It's open source, still early, and I'd love feedback from other developers.</p>

<p>👉 GitHub: <a href="https://github.com/Ali-Aldahmani/devlens" rel="noopener noreferrer">https://github.com/Ali-Aldahmani/devlens</a></p>

<p>Would love to hear what libraries you'd want added first. Drop a comment! 🙌</p>

