---
Title: JH Link: A PWA to Connect and Engage Our Local Youth Center
Description: 
Author: OLIVIER VANHESTE
Date: 2026-03-01T21:43:03.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/weekend-2026-02-28">DEV Weekend Challenge: Community</a></em></p>

<h2>
  
  
  The Community
</h2>

<p>This project is for my local youth center. While its main focus is on providing a vibrant hub for young people aged 10-19, the center maintains an open and welcoming atmosphere for the entire community. People of all ages are encouraged to drop in for a chat, grab a drink from the cafe, or even participate in activities. However, communication and engagement are often fragmented, relying on word-of-mouth and scattered social media posts. It can be hard for members to track activities, see who's attending, or feel a persistent sense of belonging when they aren't physically at the center. Administrators also spend significant time on manual tasks like check-ins and tracking participation.</p>

<h2>
  
  
  What I Built
</h2>

<p>To address these challenges, I built <strong>JH Link</strong>, a Progressive Web App (PWA) designed to be the digital heart of our youth center.</p>

<p>It's a mobile-first application that provides:</p>

<ul>
<li>  <strong>Activity Feed:</strong> A real-time list of upcoming events and activities that members can register for with a single tap.</li>
<li>  <strong>Member Profiles:</strong> A space for members to express themselves and see their engagement history.</li>
<li>  <strong>Digital Member Card:</strong> A QR-code-based virtual card for easy check-ins, replacing the need for physical cards.</li>
<li>  <strong>Gamification:</strong> A points and leveling system that rewards members for participating in activities, fostering friendly competition and sustained engagement.</li>
<li>  <strong>Admin Dashboard:</strong> A comprehensive backend for staff to manage activities, check in members, and oversee the points system.</li>
</ul>

<p>The goal is to make participation easier, more rewarding, and to strengthen the community bond, even when members are apart.</p>

<h2>
  
  
  Demo
</h2>

<p>You can try out a live demo of the application here: <a href="https://jh-link.vercel.app/login" rel="noopener noreferrer">https://jh-link.vercel.app/login</a></p>

<p>Here's a video walkthrough of the application:</p>

<p><a href="https://youtu.be/pvxCczZUtH4" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhyt77qvsfzji9b12j1o9.jpg" alt="JH Link Demo Video" width="480" height="360"></a></p>

<h2>
  
  
  Code
</h2>

<p>The full source code is available on GitHub:</p>

<p><a href="https://github.com/polcrosmol/jh-link" rel="noopener noreferrer">https://github.com/polcrosmol/jh-link</a></p>

<h2>
  
  
  How I Built It
</h2>

<p>I developed JH Link over the weekend, leveraging a modern, full-stack TypeScript approach.</p>

<ul>
<li>  <strong>Framework:</strong> <a href="https://nextjs.org/" rel="noopener noreferrer">Next.js</a> (using the App Router)</li>
<li>  <strong>Backend &amp; Database:</strong> <a href="https://supabase.com/" rel="noopener noreferrer">Supabase</a> for authentication, a PostgreSQL database with Row Level Security, and storage.</li>
<li>  <strong>Deployment:</strong> <a href="https://vercel.com/" rel="noopener noreferrer">Vercel</a>
</li>
<li>  <strong>Styling:</strong> <a href="https://tailwindcss.com/" rel="noopener noreferrer">Tailwind CSS</a>
</li>
<li>  <strong>UI/UX:</strong> The app is a fully Progressive Web App (PWA), installable on mobile devices for a native-like experience. It includes page transition animations and a navigation progress bar.</li>
<li>  <strong>Tooling:</strong> I used <code>bun</code> as the package manager and the entire development was done inside a VS Code Dev Container.</li>
</ul>

<p>A significant part of the development process involved working with an AI assistant. This co-development model was incredibly efficient. The AI handled much of the initial scaffolding, boilerplate code, and complex server actions, which allowed me to focus on the overall architecture, user experience, and critically reviewing the generated code to ensure its quality and correctness. We collaborated on everything from feature implementation to complex debugging of SQL policies and asynchronous JavaScript, enabling the creation of a feature-rich application in a very short amount of time.</p>

