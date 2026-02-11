---
Title: Building My First AI Agent
Description: 
Author: Martin Cartledge
Date: 2026-02-11T21:44:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>AI agents have never been more prominent in today's technology sector, and I think their momentum will not be slowing down anytime soon. They have become an integral piece in the workflow of technology knowledge workers across the globe, and they have done so in a matter of several months.</p>

<p>But how do these work? If you have used any of the leading agents today, you might have come to the conclusion that they share strengths and pitfalls alike.</p>

<p>I recently came across an article by <a href="https://ampcode.com/notes/how-to-build-an-agent" rel="noopener noreferrer">Thorsten Ball</a>, a long-time favorite writer of mine, that provides a roadmap for building your own AI agent.</p>

<p>I jumped at this tutorial for many reasons: I want to expand my knowledge of these tools, I don't get to use Go in my 9-5 but love the language mechanics, and I knew Thorsten is a source I can trust. He has a great way of explaining dense, archaic concepts simply.</p>

<p>I opened up the webpage and off I went. Here is my journey.</p>

<h2>
  
  
  Immediate Takeaway
</h2>

<p>By looking at the length of the post, I felt, <strong>there has to be more to this, <em>right?</em></strong> Surely a coding agent was much more involved than what I was seeing. I soon discovered that was not the case.</p>

<p>In the following sections I will go over my experience at a high level - do not expect a mirco view of this process, Thorsten does a great job of that! This post is mainly meant to serve as my overall experience and takeaways after building an agent.</p>

<h2>
  
  
  Building GoldenEye
</h2>

<h3>
  
  
  The bedrock
</h3>

<p>After creating project files and the backbone of any Go project, <strong>main.go</strong>, I used the Anthropic SDK to create an <code>Agent</code> type and a <code>NewAgent</code> function that accepts a few parameters: an Anthropic Client and a <code>getUserMessage</code> function. This new type and function are wired up in the <strong>main.go</strong> file.</p>

<p>Next, after adding my Anthropic API key, I created a <code>run</code> function that accepted a few parameters: <code>context</code> and a <code>conversation</code> array.</p>

<p>Once my key was wired up along with the <code>run</code> function, I was able to run Claude in my terminal—so cool! It greeted me with: <code>"Chat with Boris, slug head"</code> (a nod to the James Bond film <em>GoldenEye</em>, namely the N64 game, IYKYK).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv7njoozv3bneg1votdzk.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv7njoozv3bneg1votdzk.gif" alt="setting-up-the-llm" width="800" height="398"></a></p>

<h3>
  
  
  Reading, viewing, and editing
</h3>

<p>Simply creating a wrapper application that can run a LLM locally is cool on its own, but to create a proper coding agent, this application needed to do a few standard things: reading files within a working directory, listing files within that directory, and possibly the most important, <em>editing</em> files within a working directory.</p>

<p>This phase of the tutorial melted my brain a little bit, but after adding the respective types and executable functions, I found that these actions followed similar paths: they needed a strict schema to delare the precise input and output, they needed to marshal and unmarshal JSON (essentially packing and unpacking JSON in the application), and they needed to register their tool's namespace (read, write, list) in the main executable function in the go program.</p>

<h4>
  
  
  Listing and reading
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy5czg957k3n1pgw6qx1c.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy5czg957k3n1pgw6qx1c.gif" alt="listing-and-editing-files" width="720" height="359"></a></p>

<h4>
  
  
  Editing
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fca7hfvb3pyk5a9zst56j.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fca7hfvb3pyk5a9zst56j.gif" alt="editing-files" width="600" height="275"></a></p>

<h4>
  
  
  I am invincible!
</h4>

<p>With these three action pillars integrated, GoldenEye could perform similarly to the agent I use at work and at home!</p>

<p>I still feel like I need to process this piece a bit more. There is so much happening, but at the same time, little happening?</p>

<p>I give full credit to Thorsten due to this feeling. He showcases a simple path to getting an agent to work, showcases the patterns when integrating actions within an agent, and presents the final product in a no-nonsense manner.</p>

<h2>
  
  
  What I Learned
</h2>

<p>This experience reinforced something crucial: <strong>context is KING</strong>. The tool continually showed that being as clear as possible with actions—like <code>read_file({path: "main.go"})</code>—only galvanized the emphasis on providing context to increase the chances the agent acts the way you want it to.</p>

<p>More importantly, it demystified what agents actually are. The act of wiring up a model and running it against your code on a local machine is straightforward. What separates companies in this space isn't some secret sauce in the agent architecture—<strong>it's the presentation and the model itself.</strong></p>

<p>It's how context is handled throughout an ecosystem. It's keeping input and output performance at the tip of high-end for the user.</p>

<p>Companies can shine if they provide a sleek, intuitive UI and a performative, relevant, and dense model pool for their consumers. The agent loop? That's almost a solved problem. The real differentiator is the experience and the intelligence of the underlying model.</p>

<h2>
  
  
  What's Next
</h2>

<p>Now that I understand how agents work at this level, I want to go deeper. The next layer of abstraction—<strong>the models that power these agents</strong>—is rich in mystery and vast in expansive knowledge. That's where I want to dive next.</p>

<p>Building GoldenEye showed me what's possible with relatively little code. Now I want to understand what makes the intelligence behind it tick.</p>

<h2>
  
  
  Resources
</h2>

<ul>
<li>
<a href="https://ampcode.com/notes/how-to-build-an-agent" rel="noopener noreferrer">How to Build an Agent by Thorsten Ball</a> - The tutorial that inspired this</li>
<li><a href="https://docs.anthropic.com/" rel="noopener noreferrer">Anthropic API Documentation</a></li>
<li>
<a href="https://github.com/martincartledge/goldeneye" rel="noopener noreferrer">GoldenEye repository</a> - My implementation</li>
</ul>

