---
Title: Debug a NestJS application in WebStorm
Description: 
Author: Pantelis Papasavvas
Date: 2025-11-10T22:02:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>NestJS is a <strong>progressive framework</strong> that helps developers build <strong>scalable, maintainable, and testable</strong> backend applications using modern <strong>JavaScript</strong> and <strong>TypeScript</strong>.</p>

<p>It's built on top of <strong>Express</strong>, but optionally you can configure it to use <strong>Fastify</strong> as well, and embraces concepts like <strong>dependency injection</strong>, <strong>modularity</strong>, and <strong>decorators</strong>, making backend development cleaner and more predictable.</p>

<p>But even with a powerful framework like NestJS, there is one skill every developer needs — and that is <strong>debugging</strong>.</p>




<h2>
  
  
  🧩 Why debugging matters
</h2>

<p>As developers, we face many unexpected issues daily — from logical bugs to missing async calls.<br><br>
Adding <code>console.log()</code> everywhere might work for quick fixes, but it's far from efficient.</p>

<p>This is where <strong>real debugging</strong> comes into play.</p>

<p>Setting up proper debugging allows you to:</p>

<ul>
<li>Pause the code execution where you want
</li>
<li>Inspect variables in real time
</li>
<li>Step through the logic line by line
</li>
</ul>

<p>However, if you try to set a breakpoint in a NestJS project, you’ll notice that the execution doesn't stop where you expect.</p>

<p>So let’s fix that.</p>




<h2>
  
  
  ⚙️ Setting up the Debug Configuration in WebStorm
</h2>

<p>To debug a NestJS app in <strong>WebStorm</strong>, we need to add a <strong>Run/Debug configuration</strong>.</p>




<h4>
  
  
  🔹 Open the configuration window
</h4>

<p>In the top menu, go to:<br><br>
<strong>Run → Edit Configurations</strong></p>

<p>This will open the configuration window — something like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F211rd0planxum3nvkvol.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F211rd0planxum3nvkvol.png" alt=" " width="800" height="714"></a></p>




<h4>
  
  
  🔹 Create a new Node.js configuration
</h4>

<p>Press the <strong>➕ (plus)</strong> button in the top-left corner and select <strong>Node.js</strong>.<br><br>
Then give your configuration a name like <code>NestJS Debug</code> or something that makes sense to you.</p>




<h4>
  
  
  🔹 Configure the Node.js settings
</h4>

<p>Here’s what each field means and what to fill in:</p>

<ul>
<li><p><strong>Node Interpreter</strong> → This is the Node version used by your project.<br><br>
Choose <strong>Project node (<code>some path</code>)</strong> — it’s usually preselected.  </p></li>
<li><p><strong>Working Directory</strong> → The root folder of your NestJS project (where your <code>nest-cli.json</code> lives).  </p></li>
<li><p><strong>File</strong> → The path to the Nest CLI file. Normally, you will find it at:<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  node_modules/@nestjs/cli/bin/nest.js
</code></pre>

</div>



<p>This is the executable that runs your application.  </p>

<ul>
<li>
<strong>Application Parameters</strong> → These are the arguments that tell Nest CLI what to do.
You can type:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  start --debug --watch
</code></pre>

</div>






<h4>
  
  
  🔹 Understanding the parameters
</h4>

<p>Let’s break down what these parameters actually do:</p>

<ul>
<li>
<code>start</code>: Tells NestJS to start the application.
</li>
<li>
<code>--debug</code>: Enables debugging mode, allowing WebStorm to attach breakpoints.
</li>
<li>
<code>--watch</code>: Enables live reload — every time you save a file, NestJS restarts automatically.
</li>
</ul>

<p>So, in simple terms, we are telling Nest CLI:  </p>

<blockquote>
<p>“Start my app in debug mode and keep watching for code changes.”</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmk272vbw5t8cbea5oxzu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmk272vbw5t8cbea5oxzu.png" alt=" " width="800" height="720"></a></p>




<h4>
  
  
  🔹 Save and start debugging
</h4>

<p>Now everything is set<br>
Press <strong>OK</strong> to save your configuration, set your breakpoints, and hit the <strong>Debug</strong> button.</p>

<p>WebStorm will launch the application in debug mode, and you’ll be able to pause the execution exactly where you want it.</p>




<h2>
  
  
  🧠 Conclusion
</h2>

<p>Debugging is not just about fixing bugs — it’s about understanding how your code actually works.</p>

<p>By applying this setup, you’ll spend less time guessing and more time improving your code.</p>

<p>If you found this helpful, leave a ❤️ or a comment — it helps other developers discover it too.</p>

<p><strong>Happy debugging! ⚡</strong></p>

