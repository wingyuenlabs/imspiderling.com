---
Title: Why I built a "Zero-Backend" dev toolset with Next.js 16 and Tailwind v4
Description: 
Author: Artyom Davtyan
Date: 2026-02-19T21:37:52.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feb52vk0f33cbfz8egleo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feb52vk0f33cbfz8egleo.png" alt=" " width="800" height="393"></a></p>

<p>Let’s be real: most "<strong>free</strong>" online tools for developers suck.</p>

<p>You want to resize a quick image for a README or format a messy JSON config, and you're met with:</p>

<p>Shady ads everywhere.</p>

<p>"Please create an account to download."</p>

<p>The "Security Guilt"—waiting for a server to process your private data.</p>

<p>I got tired of this friction. I wanted something that felt like a native app but lived in my browser. So, I spent the last few weeks building noserver.app.</p>

<p>The Goal: <strong>100% Client-Side</strong><br>
The main challenge was: can I make heavy tasks (like image optimization or PDF manipulation) run entirely on the user's hardware?</p>

<p>Thanks to <strong>WebAssembly (WASM)</strong> and the Canvas API, the answer is a resounding yes.</p>

<p><strong>The Stack (The "Vibe" part)</strong><br>
I decided to go with the bleeding edge because... why not?</p>

<p><strong>Next.js 16</strong>: The App Router is finally feeling stable, and the hydration handling in 16 is a lifesaver for WASM-heavy sites.</p>

<p><strong>Tailwind v4</strong>: I switched to v4 early on, and the build speed is actually insane. No more bulky tailwind.config.js—everything is CSS-first. It made the styling phase of this project actually fun.</p>

<p><strong>Pica.js</strong>: For high-quality image resizing without that "browser blur."</p>

<p>What I learned<br>
Moving logic from the backend to the client isn't just about privacy; it's about latency. When you remove the "Upload -&gt; Process -&gt; Download" cycle, the UX feels instant.</p>

<p>However, managing WASM binaries in a Next.js environment is still a bit of a headache (matching the server/client chunks can be tricky).</p>

<p><strong>What’s next?</strong><br>
I just launched the first few modules (Image Resizer, DevShot, JSON tools), but I’m just getting started.</p>

<p>I have two questions for the Dev.to community:</p>

<p>What’s a "simple" tool you use that feels unnecessarily bloated or slow?</p>

<p>If you’ve worked with Tailwind v4, how are you handling complex animations? I’m still figuring out the best patterns there.</p>

<p>Check it out here: <a href="https://noserver.app" rel="noopener noreferrer">https://noserver.app</a> 🚀</p>

<p>I’d love some brutal feedback on the performance and the UI. No filters, just tell me what's broken.</p>

