---
Title: Announcing CourseShelf: Rotten Tomatoes for Online Learning
Description: 
Author: Daniel Bergholz
Date: 2026-01-21T22:10:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>Back in 2019, I created <a href="https://techschool.dev" rel="noopener noreferrer">TechSchool</a>, an open-source platform to help people find free programming courses. The idea was simple: fight back against predatory coding bootcamps that charge thousands of dollars for content that's often worse than what's available for free on YouTube.</p>

<p>TechSchool worked. People found great courses. The community grew. But over time, I started noticing something.</p>

<h2>
  
  
  The limitations of TechSchool
</h2>

<p>TechSchool had two big problems:</p>

<p><strong>One</strong>, it was limited to tech. My friend who wanted to learn guitar couldn't use it. My mom who wanted to learn cooking couldn't use it. The core idea of community-curated courses was valuable, but I was artificially limiting who could benefit from it.</p>

<p><strong>Two</strong>, adding new courses required opening a Pull Request on GitHub. Great for developers, terrible for everyone else. Most people who want to learn don't know what Git is, and they shouldn't have to.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk4rrcopfkefk3a2jhln0.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk4rrcopfkefk3a2jhln0.gif" alt="Thinking hard" width="500" height="345"></a></p>

<p>I kept thinking: what if we could build something bigger? A place where anyone could discover quality courses in <strong>any</strong> subject, reviewed by real learners instead of marketing departments?</p>

<h2>
  
  
  Enter CourseShelf
</h2>

<p>CourseShelf is the spiritual successor to TechSchool. Think of it as <strong>Rotten Tomatoes for online learning</strong>.</p>

<p>Here's the core idea: when you're looking for a movie to watch, you check Rotten Tomatoes or IMDb. You trust the crowd more than the trailer. Why should finding a good course be any different?</p>

<p>Right now, if you want to learn something new, you either:</p>

<ul>
<li>Trust the platform's algorithm (which prioritizes engagement, not quality)</li>
<li>Trust sponsored reviews (which are basically ads)</li>
<li>Hope you get lucky</li>
</ul>

<p>CourseShelf fixes this by letting real learners review and rate courses. No fake reviews. No sponsored content. Just honest opinions from people who actually completed the course.</p>

<p>Website: <a href="https://thecourseshelf.com" rel="noopener noreferrer">https://thecourseshelf.com</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F31tjmfirk4k8rs5g1pa9.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F31tjmfirk4k8rs5g1pa9.gif" alt="Let's go" width="500" height="500"></a></p>

<h2>
  
  
  What you can do on CourseShelf
</h2>

<p>Here's a quick overview of the features:</p>

<ul>
<li>
<strong>Discover courses</strong> across any subject, not just tech</li>
<li>
<strong>Read and write reviews</strong> with star ratings</li>
<li>
<strong>Save courses to your library</strong> to track what you want to learn</li>
<li>
<strong>Create playlists</strong> to curate learning paths for yourself or others</li>
<li>
<strong>Follow other learners</strong> to see what they're learning and recommending</li>
<li>
<strong>Verify your channel</strong> if you're a course creator on YouTube</li>
</ul>

<p>Right now, the platform supports <strong>YouTube courses</strong> and <strong>books</strong>. We have plans to add support for other platforms like Coursera, Skillshare, and Domestika in the future.</p>

<h2>
  
  
  For course creators
</h2>

<p>If you're a content creator, CourseShelf has something special for you. You can verify your YouTube channel and get a <strong>verified badge</strong> on your profile. This helps learners identify official creators and gives you credibility.</p>

<p>Verified creators also get access to analytics: see how many people are viewing your courses, what they're saying in reviews, and how you compare to others in your category.</p>

<p>I created TechSchool because I was frustrated that my free courses had fewer views than paid garbage. CourseShelf is my attempt to fix the discoverability problem for all creators who are putting out quality content.</p>

<h2>
  
  
  The tech stack
</h2>

<p>I built CourseShelf using <strong>Elixir and Phoenix</strong> with <strong>React and TypeScript</strong> on the frontend via Inertia.js. If you've read my blog post about <a href="https://dev.to/danielbergholz/from-nextjs-to-rails-then-elixir-my-journey-through-reactjs-burnout-h8d">leaving React for Rails and then Elixir</a>, you know this is the stack I fell in love with.</p>

<p>Phoenix gives me the productivity of Rails with the performance and reliability of the Erlang VM. Inertia.js lets me use React without the complexity of building a separate API. It's the best of both worlds.</p>

<h2>
  
  
  Disclaimer
</h2>

<p>CourseShelf is currently in <strong>beta</strong>. We're actively adding features, fixing bugs, and improving the experience based on feedback. If you find something broken or have a suggestion, please let me know!</p>

<p>Some features are limited during beta, but the core experience of discovering and reviewing courses is fully functional.</p>

<h2>
  
  
  Join the community
</h2>

<p>I built TechSchool because I believe tech education should be accessible to everyone. I'm building CourseShelf because I believe <strong>all</strong> education should be accessible to everyone.</p>

<p>If you're a self-taught learner, a course creator, or just someone who's tired of wasting money on bad courses, give CourseShelf a try. And if you know a great course that deserves more attention, add it to the platform!</p>

<p>Let's build the largest community-curated database of online courses together. We are all gonna make it 🔥</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftqek6gbi1lugsas1ryft.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftqek6gbi1lugsas1ryft.gif" alt="We're all gonna make it" width="500" height="270"></a></p>

<h2>
  
  
  Thank you
</h2>

<p>If you reached the end of this post, thank you for your time. Building in public is scary, but seeing people use something I created makes it worth it.</p>

<p>Have questions? Want to chat? Find me on Twitter <a href="https://twitter.com/danielbergholz" rel="noopener noreferrer">@danielbergholz</a> or drop me an email.</p>

<p>See you on CourseShelf!</p>

