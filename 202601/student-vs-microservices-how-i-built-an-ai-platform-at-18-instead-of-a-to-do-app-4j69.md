---
Title: Student vs. Microservices: How I Built an AI Platform at 18 Instead of a To-Do App
Description: 
Author: Bogdan Tytysh
Date: 2026-01-25T21:39:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hi Dev.to! I'm Bogdan, and I just turned 18. While many start their coding journey with a simple To-Do list, I decided to go deep into the rabbit hole of microservices for my project, <strong>SkillSwapAI</strong>.</p>

<p>I’ve spent the last few months architecting a system that isn't just a monolithic block. Here is my current stack:</p>

<ul>
<li>
<strong>Backend</strong>: NestJS handles the core logic and user management.</li>
<li>
<strong>AI Engine</strong>: A FastAPI service in Python that generates study plans using OpenAI.</li>
<li>
<strong>Cloud Infrastructure</strong>: I’m using AWS Lambda, S3 for storage, and CloudFront for content delivery.</li>
<li>
<strong>Database</strong>: PostgreSQL with Prisma ORM.</li>
</ul>

<p><strong>Architecure:</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80akh88lbqpbrjuqh055.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80akh88lbqpbrjuqh055.png" alt=" " width="800" height="927"></a></p>

<p>The biggest challenge wasn't the code itself, but making these services talk to each other efficiently via HTTP. I’m currently refactoring parts of it to LangChain to make the AI features even smarter.</p>

<p>I’m starting my first official job as a Full Stack JS Developer on January 28th, but this project is where I truly learned how to be an engineer. Check out my full architectural breakdown on <a href="https://medium.com/@bogdantytysh5/student-vs-microservices-how-i-built-a-full-stack-ai-platform-instead-of-another-todo-app-ccf056e5d713" rel="noopener noreferrer"><strong>Medium</strong></a>"</p>

