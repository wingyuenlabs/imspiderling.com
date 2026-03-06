---
Title: I got tired of clunky font menus, so I built a blazing-fast local font explorer (Zero Tracking)
Description: 
Author: Fady Helmey
Date: 2026-03-06T21:28:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Problem: The "Font Menu" Fatigue<br>
If you are a designer or a frontend developer, you know the pain. You have hundreds of fonts installed on your machine, but previewing and comparing them means scrolling through tiny, slow, and clunky dropdown menus in your design software. Testing fonts one by one wastes time and completely breaks the creative flow.<br>
I wanted a single, focused workflow to explore my typography without the mess. So, I built TypeLens.<br>
What is TypeLens?<br>
<a href="https://type-lens-app.vercel.app" rel="noopener noreferrer">TypeLens</a> is an experimental typography exploration tool that helps designers and developers visualize, compare, and manage their locally installed fonts in a fast, minimal interface—right in the browser.<br>
Key Features I Focused On:<br>
Wordmark Mode: Type your text once and instantly preview it across every single font installed on your system. No more guessing how your logo or heading will look.<br>
Side-by-Side Comparison: You can lock up to 4 fonts, sync their size and alignment, and clearly observe the structural differences side by side.<br>
Smart Filtering: Filter your local fonts by category, weight, or style to quickly narrow down large libraries.<br>
100% Privacy (Zero Tracking): This was crucial for me. TypeLens uses the Local Font Access API with fully local processing. No font files are uploaded, there is no tracking, and absolutely no data leaves your machine.<br>
Under the Hood &amp; A Recent Fix<br>
Building this required ensuring that the browser accurately reflects what's actually installed on the OS.<br>
In fact, I just pushed a major update: The font filtering issue is now fixed ✅<br>
Previously, browsers could sometimes simulate font weights if the actual weight wasn't installed. Now, TypeLens reads the real installed font weights correctly. The Bold, Italic, and Bold Italic filters only show true variants, and display fonts are handled and labeled properly.<br>
Try It Out!<br>
TypeLens is completely free and open to use. I built this to solve my own itch, but I’d love for the design and dev community to try it out.<br>
🔗 Check it out here: <a href="https://type-lens-app.vercel.app&lt;br&gt;%0A![%20](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ievfgms9dyxwemb4g5wq.png)&lt;br&gt;%0A![%20](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9d8li5ps6l91xi6buzud.png)&lt;br&gt;%0A![%20](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y0ndkanlwzhi73dpzgi6.png)" rel="noopener noreferrer">TypeLens</a><br>
I’d love your feedback! If you notice any bugs or have feature requests, let me know in the comments. </p>

