---
Title: Lo viejo funciona, Juan! A monolithic is enough, almost ever.
Description: 
Author: coco
Date: 2025-12-20T21:41:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>It is a common saying: ‘You will need a MVP ready as fast as you can.’ <br>
Because you need to test it, to know if there are any users who validates your idea and see if it solves anyone’s problem. However, as a developers, we are always designing a huge architecture for no more than 100 actual users.</p>

<blockquote>
<p>I need a landing page with a contact form to input their email and user’s name.</p>
</blockquote>

<p>Great!</p>

<p><code>npm create vite@latest</code></p>

<p>I will develop a ReactJS project with router, so any case the user writes the URL wrong, I can redirect them to the homepage again. I will split the code into many components. One for the header, the other is the footer, a Main-content wrapping the different sections. <br>
I will need a React Hook Form. For isolating component re-renders, to improve the site’s performance, of course.<br>
To style all this, I will use Tailwind and @gsap/react for animations, so I will be able to finish it faster.</p>

<p>Then, I also need an API in NodeJS to send the email.</p>

<ol>
<li>A POST route for the form, </li>
<li>GET route for checking the API’s health.</li>
<li>GET default route for index.</li>
</ol>

<p>I was thinking about using express-xss-sanitizer middleware for security reasons, and Jest for unit testing. <br>
You will have it in two or three weeks.</p>

<p>But we are forgetting the KISS. Where is the love? </p>

<p>KISS rule is Keep it simple s…</p>

<p>Instead of all these files, routes, packages, and repositories. What if we develop a simple HTML with CSS. You can use Tailwind, and GSAP, loading with a CDN if it makes the development faster. But you truly don’t need React for a simple landing page. <br>
You could even try Astro first, or just Vite. But I do believe that a simple HTML file is enough.<br>
You will need a backend to send the email; it is mandatory. In this case, you could look for premade tools to send emails, or serverless such as Lambda in AWS. However, you can do it with a simple PHP file. If you don’t know PHP language, you could ask ChatGPT or GitHub Copilot to make one for you.<br>
With two simple files (one HTML another in PHP), plus a CSS file, you will have the landing running.</p>

<p>Is it the old way to do it? Yes, but it is easier and faster. You will have time to make your micro-service architecture any time later.</p>

<p>The main goal of a developer is to fix any problem as fast as possible. If you can make it cheaper, better :D</p>

<p>You can also do it with no-code apps like Wix, Webflow, Carrd, etc. I know that. However, this post is for developers, not for entrepreneurs. If you are not a developer, you should definitely go to no-code apps. They will make your life easier. </p>

<p>However, what I am trying to debate here, is that we (developers) always think about the first rule: DRY (Don’t Repeat Yourself), but we always forget about the second KISS rule. It is important to balance both.</p>

<p>:D</p>

