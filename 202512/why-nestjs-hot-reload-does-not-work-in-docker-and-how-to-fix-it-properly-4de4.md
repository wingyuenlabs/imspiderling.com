---
Title: Why NestJS Hot Reload Does Not Work in Docker and How to Fix It Properly
Description: 
Author: Stephen Akugbe
Date: 2025-12-22T21:51:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you are running NestJS inside Docker and your code changes are not reflecting, you are not alone. This is one of the most common pain points developers hit when combining NestJS, Docker, and macOS or Windows.</p>

<p>You save a file.<br>
Nothing happens.<br>
You rebuild the container.<br>
Still confused.</p>

<p><em>Sidenote: I have actually fixed this exact problem before on another project. But when I hit it again on a new NestJS setup, I realized I could not remember all the details. That was the moment I decided to document it properly this time.</em></p>

<p>At some point, you install webpack because someone on the internet said it fixes the issue.</p>

<p>This article explains what is really going on, why Docker behaves this way, when webpack is actually needed, and what the correct setup looks like for NestJS development.</p>

<h2>
  
  
  The core misunderstanding
</h2>

<p>Docker does not watch your files.</p>

<p>Docker only sees what exists inside the container. Whether changes reflect or not depends entirely on how files get into the container and how NestJS is running.</p>

<p>There are two very different worlds that often get mixed up.</p>

<ul>
<li>Development mode</li>
<li>Production mode</li>
</ul>

<p>If you confuse the two, hot reload will never work.</p>

<h2>
  
  
  How NestJS hot reload works without Docker
</h2>

<p>When you run NestJS locally using</p>

<p><code>npm run start:dev</code></p>

<p>The Nest CLI uses a file watcher. When a file changes, the process restarts automatically.</p>

<p>No Webpack is involved here.<br>
No HMR.<br>
Just process a restart.</p>

<p>This works because your filesystem events are available directly to Node.</p>

<h2>
  
  
  What changes when you introduce Docker
</h2>

<p>Once NestJS runs inside a container, file watching depends on three things.</p>

<ol>
<li>How files get into the container</li>
<li>How NestJS is started</li>
<li>Whether filesystem events propagate correctly</li>
</ol>

<p>If any of these is wrong, hot reload breaks.</p>

<h2>
  
  
  The only setup that works reliably for development
</h2>

<p>For NestJS changes to reflect instantly inside Docker, all of the following must be true.</p>

<p>Your source code is mounted into the container using volumes<br>
NestJS runs in watch mode using start dev<br>
File watching works inside Docker</p>

<p>If even one is missing, you will be rebuilding images unnecessarily.</p>

<p>Volumes are non-negotiable in development</p>

<p>If your Dockerfile copies files like this</p>

<p><code>COPY . .</code></p>

<p>Then your container has a snapshot of your code. Docker has no idea when you edit files locally.</p>

<p>That setup is correct for production but wrong for development.</p>

<p>In development, your compose file must mount the project into the container.</p>

<p>Example conceptually:</p>

<p>Local files<br>
Shared into the container<br>
Changes appear instantly</p>

<p>This is exactly what your compose file already does.</p>

<p><code>.:/usr/src/app</code></p>

<p>That line is the backbone of hot reload.</p>

<p>Why node_modules must be excluded</p>

<p>This line is subtle but important.</p>

<p><code>/usr/src/app/node_modules</code></p>

<p>It prevents your local node_modules from overwriting the container dependencies. Without it, you get native module issues and random crashes.</p>

<p>This is a best practice, and you got it right.</p>

<p>start dev is not optional</p>

<p>NestJS will only watch files if you explicitly run it in watch mode.</p>

<p>This means your command must be</p>

<p><code>npm run start:dev</code></p>

<p>and not</p>

<p><code>node dist/main.js</code><br>
or<br>
<code>nest start</code></p>

<p>The moment you run compiled JavaScript, hot reload is gone. That is expected behavior.</p>

<h2>
  
  
  The macOS and Windows file watching problem
</h2>

<p>Even with volumes and start dev, many developers still see no reload.</p>

<p>This is not a NestJS bug.</p>

<p>On macOS and Windows, Docker Desktop runs Linux containers inside a virtual machine. Native filesystem events often do not propagate correctly into the container.</p>

<p>That is why NestJS does not detect changes.</p>

<p>The fix is polling.</p>

<p>These environment variables force Node-based watchers to actively check for changes instead of relying on filesystem events.</p>

<p><code>CHOKIDAR_USEPOLLING=true<br>
WATCHPACK_POLLING=true</code></p>

<p>This is not a hack. It is a known workaround and widely used.</p>

<p>Once enabled, NestJS reliably restarts when files change.</p>

<h2>
  
  
  The webpack confusion explained
</h2>

<p>This is where most people go wrong.</p>

<p>NestJS supports two different reload strategies.</p>

<ol>
<li>Process restart using the Nest CLI watcher</li>
<li>Hot Module Replacement using webpack</li>
</ol>

<p>Webpack is only required for HMR.</p>

<p>If you never enabled HMR, you don't need Webpack at all.</p>

<p>So why do people install it?</p>

<p>Because they accidentally enable HMR.</p>

<p>If you see any of these, Webpack becomes mandatory.</p>

<ol>
<li>
<code>module.hot</code> in main.ts</li>
<li>
<code>webpack: true</code> in nest cli config</li>
<li>Custom HMR setup</li>
</ol>

<p>At that point, NestJS expects Webpack and Webpack CLI to exist.</p>

<p>Many developers install Webpack, thinking it fixes Docker reload issues, when in reality, polling was the missing piece.</p>

<p>When webpack actually makes sense</p>

<p>Webpack HMR can be useful if:</p>

<ol>
<li>Your application startup time is slow</li>
<li>You want instant reload without restarting the process</li>
<li>You know you enabled HMR intentionally</li>
</ol>

<p>But it adds complexity and is not required for most backend APIs.</p>

<p>For many projects, a simple restart on change is stable and good enough.</p>

<p>Development versus production mental model</p>

<p>This simple rule will save you hours.</p>

<p>In development:</p>

<ul>
<li>Use volumes</li>
<li>Run start dev</li>
<li>Enable polling</li>
<li>Never rebuild</li>
</ul>

<p>In production:</p>

<ul>
<li>Do not use volumes</li>
<li>Copy files during build</li>
<li>Run compiled JavaScript</li>
<li>Rebuild on every change</li>
</ul>

<p>If you mix these two worlds, things break.</p>

<p>Final thoughts:</p>

<ul>
<li>Docker is not broken.</li>
<li>NestJS is not broken.</li>
<li>Hot reload is not magic.</li>
</ul>

<p>Once you understand how files enter the container and how NestJS watches them, everything becomes predictable.</p>

<p>If you ever find yourself rebuilding Docker images just to see a console log update, something in your setup is wrong.</p>

<p>And if you installed Webpack just to make file watching work, you probably did not need it.</p>

<p>If this article saved you time, chances are it will save someone else hours of frustration too.</p>

