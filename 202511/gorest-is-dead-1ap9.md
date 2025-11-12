---
Title: GoREST is dead...
Description: 
Author: Nicolas Bonnici
Date: 2025-11-12T21:31:11.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  This Is the End
</h1>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxtp30uwz9udncgq7qjry.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxtp30uwz9udncgq7qjry.webp" alt="Jim Morrison from The Doors band, singing this is the end" width="500" height="368"></a></p>

<p>Recently, <a href="https://github.com/nicolasbonnici/gorest" rel="noopener noreferrer">I shared a project skeleton</a> and <a href="https://dev.to/nicolasbonnici/from-a-database-to-a-rest-api-3cgf">a post</a> that allows you to easily turn almost any relational database schema into a REST API.</p>

<p>It took me about a month to develop the first release, just a few hours at night when the kids were asleep and during weekends, since I’m currently working full-time.</p>

<p>The feedback was mixed. Some people liked it, others simply hated it. At some point on Reddit, people even downvoted the project without explaining why. Fortunately, some benevolent users also took the time to tell me what they liked or didn’t about the project.</p>




<h2>
  
  
  The Need and the Vision Behind the First Release
</h2>

<p>My initial goal was to solve a problem I had: reusing an existing, functional relational data model and its data from a monolithic application, and turning it into a <strong>headless architecture</strong>, where the backend is a REST API, and the resources are consumed by both a web app and an admin portal using Nuxt UI and Flutter respectively.</p>

<p>After benchmarking the first release candidate, I was impressed by the performance and decided to share the code under an MIT license for anyone facing the same challenge.</p>

<p>I’m not a Go expert. I recently learned the basics through <a href="https://grok.com/" rel="noopener noreferrer">Grok</a>, and this was my first real project involving more advanced use cases. I managed to build something genuinely useful, and overall, the performance was quite decent, so why not share it?</p>




<h2>
  
  
  So, Why Kill It So Fast?
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbcvsraiess514w1m4ifr.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbcvsraiess514w1m4ifr.webp" alt="Scary movie meme, I see dead people" width="480" height="206"></a></p>

<p>Sometimes, programming languages are more than just ways to give instructions to a machine. They come with implicit rules, conventions, and structural philosophies. Go is no exception.</p>

<p>The first and most important issue with GoREST 0.1 wasn’t specific to Go: <strong>separation of concerns</strong>. Nobody wants to mix their business logic into a skeleton project unless that skeleton is meant to evolve directly into their final product.</p>

<p>Then there’s <strong>maintainability</strong>. Having a boilerplate that requires regular updates quickly becomes a burden and makes long-term maintenance painful.</p>

<p>And finally, <strong>flexibility</strong>. I wanted to offer multiple ways to use and implement GoREST, either as a library integrated into an existing project or as the foundation of a brand-new one. There are thousands of ways to code business logic, and I don’t want to impose just one.</p>

<p>So, there’s no reason to maintain this skeleton anymore.<br><br>
Let’s cut off its head (no headless architecture related nerdy joke intended, literally).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpokyjy5itcxx8fok2y9y.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpokyjy5itcxx8fok2y9y.webp" alt="Conan The Barbarian movie scene" width="319" height="177"></a></p>




<h2>
  
  
  RIP GoREST 0.1.0
</h2>

<p>Long live <strong><a href="https://github.com/nicolasbonnici/gorest/tree/0.2.0" rel="noopener noreferrer">GoREST 0.2.0</a></strong>!  </p>

<p>Yup, from the fresh ashes of GoREST 0.1 comes a complete rewrite as a <strong>library</strong> and <strong>toolbox</strong>. It aims to help you quickly scaffold your business objects and write your business logic with ease.</p>

<p>A new release marks a new chapter for GoREST. Yes, I know this is a total breaking change and should be a major version, but since no one is using it yet except me, and we’re still far from version 1.0.0, there’s no harm in making it a minor upgrade.</p>

<p>In my opinion, <a href="https://github.com/nicolasbonnici/gorest/tree/0.2.0" rel="noopener noreferrer">this new 0.2.0 release</a> is a healthy foundation to iterate from. It keeps the same features and benefits as the previous version but is now decoupled into a library of Go modules and useful commands.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fege78ipvxwmc6b06tbqs.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fege78ipvxwmc6b06tbqs.webp" alt="Frankenstein movie scene, it's alive" width="245" height="240"></a></p>




<h2>
  
  
  Road to 1.0.0
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqkoods35x2wv5fen7b1i.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqkoods35x2wv5fen7b1i.webp" alt="An old Amiga racing game called Lotus Turbo Challenge" width="641" height="402"></a></p>

<p>Here’s what I want to accomplish before releasing the first stable, long-term support version:</p>

<ul>
<li>Better performance
</li>
<li>A fully modular middleware layer with install/uninstall hooks
</li>
<li>100% test coverage (currently around 80%)
</li>
<li>gRPC support
</li>
<li>Support for more database engines
</li>
<li>Scaffolded clients
</li>
<li>Some kickass new features I haven’t even thought of yet...</li>
</ul>




<h2>
  
  
  Contribute
</h2>

<p>Wanna join the adventure and contribute to the project?  </p>

<p>All kinds of contributions are welcome, even just ideas or suggestions.</p>

<p>Read the <a href="https://github.com/nicolasbonnici/gorest/blob/trunk/CONTRIBUTING.md" rel="noopener noreferrer">contributing section</a>.</p>

<p>You can <a href="https://github.com/nicolasbonnici/gorest/fork" rel="noopener noreferrer">fork it</a> and/or <a href="https://github.com/nicolasbonnici/gorest/stargazers" rel="noopener noreferrer">star the project</a> if you like it.</p>

<p>Thanks for reading.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsz7ab4q2empepsuks9b2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsz7ab4q2empepsuks9b2.png" alt="Until next time, Skeletor meme" width="257" height="196"></a></p>

