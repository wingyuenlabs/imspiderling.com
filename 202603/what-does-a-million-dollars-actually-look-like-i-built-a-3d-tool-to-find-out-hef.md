---
Title: What does a million dollars actually look like? I built a 3D tool to find out.
Description: 
Author: Alp Yalay
Date: 2026-03-06T21:21:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>A few months ago, my co-founder Moris asked me a question that I couldn't answer: "If someone handed you a million dollars right now, how much space would it actually take up?"</p>

<p>We started digging and realized that every "money" tool on the internet is basically a table of numbers. Numbers are great for spreadsheets, but they're terrible at showing scale. You can't feel the difference between $10,000 and $100,000 until you see the physical volume.</p>

<p>So we spent the last few months building Money Visualiser.</p>

<p>It’s a WebGL (WebGPU in progress) sandbox where you type in an amount, pick two currencies, and see them materialize as 3D bill stacks. We pulled the exact dimensions for 82 different banknotes from central bank specs. For example, if you're looking at a stack of Japanese Yen, you’ll see they’re wider and taller than US Dollars, just like in real life.</p>

<p>Technical bits for the curious:</p>

<p>Stack: Next.js 16 (App Router), Three.js, React Three Fiber.<br>
Physics: We use bank-standard "straps" and "bricks" so the stacking logic is actually accurate to how money is stored.<br>
Performance: Rendering 300k+ individual bills at 60fps was... fun.<br>
Live Data: Exchange rates refresh every 30s with a 5-tier fallback system.<br>
I personally love the 'Aircraft Carrier' environment—it’s the only place big enough to show what a trillion dollars actually looks like without clipping through the walls.</p>

<p>Check it out at moneyvisualiser.com. No signup, no catch, just a project to make abstract numbers feel a bit more real. I'd love to hear what other devs think about the Drei/R3F setup we used.</p>

