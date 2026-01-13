---
Title: Virtual 3D Museum - Three.js
Description: 
Author: nbmbnbm
Date: 2026-01-13T22:09:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>So, I was shitcanned recently and said to myself: "Hey, why not actually learn something new and interesting for once?"</p>

<p>Three.js has been high on my list for a long time. I tried to make a pinball game a couple of years back, failed miserably, and never quite forgot about it. This time, I wanted to see if I could turn Wikipedia entries into something more visual and "walkable"</p>

<p>The result is a Virtual 3D Museum. It’s a 3D environment where the "exhibits" are pulled dynamically from the Wikipedia API, and gallery rooms are populated with that info on the fly</p>

<p><iframe height="600" src="https://codepen.io/notbigmuzzy/embed/dPXOmoq?height=600&amp;default-tab=result&amp;embed-version=2">
</iframe>
</p>

<p><strong>The Tech:</strong></p>

<ul>
<li>Three.js: Handles the spatial layout and rendering.</li>
<li>Vanilla JS: No frameworks. I wanted to keep it lightweight and see how far I could get with just the basics (spoiler: it can go really far).</li>
<li>Wikipedia API: The source of all the data.</li>
</ul>

<p><strong>The Details:</strong></p>

<ul>
<li>Multi-language support: You can toggle between different language versions of Wikipedia.</li>
<li>Persistence: It uses basic local storage, so it remembers your session when you come back.</li>
<li>Environment: I went for a "faux 3D" look—enough to feel immersive without being heavy on the hardware. I know it doesn't look "well" yet; I don't know how to use Blender properly, and I'm still learning exactly how far you can push a browser before the stuttering ruins the experience.</li>
</ul>

<p>It’s been a fun experiment in data visualization and spatial UI. If you’re into Three.js or just want to see a weird way to read about history or universe, check it out ;)</p>

<p>Source Code: GitHub - <a href="https://github.com/notbigmuzzy/linkwalk" rel="noopener noreferrer">https://github.com/notbigmuzzy/linkwalk</a> <br>
Live Demo: Full version with languages and persistence - <a href="https://notbigmuzzy.github.io/linkwalk/" rel="noopener noreferrer">https://notbigmuzzy.github.io/linkwalk/</a></p>

