---
Title: Setting Up NestJS with Prisma (Without the Headache )
Description: 
Author: Xavier Carrera Gimbert
Date: 2025-11-18T21:20:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>Starting a project with NestJS and Prisma should be easy, right? They’re both popular technologies and a great match. There are even official articles <a href="https://docs.nestjs.com/recipes/prisma" rel="noopener noreferrer">(1)</a><a href="https://www.prisma.io/blog/nestjs-prisma-rest-api-7D056s1BmOL0" rel="noopener noreferrer">(2)</a> explaining how to integrate them.</p>

<p><strong>Wrong.</strong></p>

<p>For some reason, those articles are outdated. So here I am, writing this guide to save you a serious headache when setting up your project.</p>

<h2>
  
  
  Let's Get to It
</h2>

<blockquote>
<p>💡 I’ll be using <strong>pnpm</strong> for this project, but feel free to use any package manager you prefer.</p>
</blockquote>

<h3>
  
  
  1. Create the Project
</h3>

<p>You’ll be prompted to choose a package manager. Select <strong>pnpm</strong> if you want to follow this guide without changes. Otherwise, adapt as needed.</p>

<blockquote>
<p>💡 At the time of writing, the latest version of @nestjs/cli appears to have a bug, so I'm using an older version. This may have been resolved by the time you're reading.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm dlx @nestjs/cli@11.0.0 new nestjs-prisma-test
</code></pre>

</div>



<h3>
  
  
  2. Set Up PostgreSQL with Docker
</h3>

<p>Create a Docker file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">touch </span>docker-compose.yml
</code></pre>

</div>



<p>Add the following content:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># docker-compose.yml</span>

<span class="na">version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.8'</span>
<span class="na">services</span><span class="pi">:</span>
  <span class="na">postgres</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:13.5</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">always</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">POSTGRES_USER=myuser</span>
      <span class="pi">-</span> <span class="s">POSTGRES_PASSWORD=mypassword</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">postgres:/var/lib/postgresql/data</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s1">'</span><span class="s">5432:5432'</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">postgres</span><span class="pi">:</span>
</code></pre>

</div>



<p>Run the container and keep the terminal open:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker-compose up
</code></pre>

</div>



<h3>
  
  
  3. Install Prisma
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add prisma <span class="nt">-D</span>
</code></pre>

</div>



<p>Initialize Prisma:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm dlx prisma init
</code></pre>

</div>



<h3>
  
  
  4. Configure Environment Variables
</h3>

<p>Install <code>dotenv</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add dotenv
</code></pre>

</div>



<p>Update your config files to load environment variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// prisma.config.ts</span>
<span class="k">import</span> <span class="dl">'</span><span class="s1">dotenv/config</span><span class="dl">'</span><span class="p">;</span>
<span class="c1">// rest of the file...</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/main.ts</span>
<span class="k">import</span> <span class="dl">'</span><span class="s1">dotenv/config</span><span class="dl">'</span><span class="p">;</span>
<span class="c1">// rest of the file...</span>
</code></pre>

</div>



<p>Update your <code>.env</code> file (remove any comments):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>DATABASE_URL="postgres://myuser:mypassword@localhost:5432"
</code></pre>

</div>



<h3>
  
  
  5. Define Your Models
</h3>

<p>Edit <code>prisma/schema.prisma</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
</code></pre>

</div>



<p>Run the initial migration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm dlx prisma migrate dev <span class="nt">--name</span> <span class="s2">"init"</span>
</code></pre>

</div>



<h3>
  
  
  6. Install Prisma Client
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm add @prisma/client
</code></pre>

</div>



<blockquote>
<p>💡 You don’t need to run <code>generate</code> now - Prisma does it automatically when installing the client. But for future schema changes, run:</p>


<pre class="highlight shell"><code>pnpm dlx prisma generate
</code></pre>

</blockquote>

<h3>
  
  
  7. Create the Prisma Service
</h3>

<p>Note: We import from <code>/client</code>, not as shown in the official docs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/services/prisma.service.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span><span class="p">,</span> <span class="nx">OnModuleInit</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@nestjs/common</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">PrismaClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">generated/prisma/client</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">()</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">PrismaService</span> <span class="kd">extends</span> <span class="nc">PrismaClient</span> <span class="k">implements</span> <span class="nx">OnModuleInit</span> <span class="p">{</span>
  <span class="k">async</span> <span class="nf">onModuleInit</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">$connect</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  8. Register the Prisma Service
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/app.module.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">Module</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@nestjs/common</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">AppController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./app.controller</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">PrismaService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./services/prisma.service</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Module</span><span class="p">({</span>
  <span class="na">imports</span><span class="p">:</span> <span class="p">[],</span>
  <span class="na">controllers</span><span class="p">:</span> <span class="p">[</span><span class="nx">AppController</span><span class="p">],</span>
  <span class="na">providers</span><span class="p">:</span> <span class="p">[</span><span class="nx">PrismaService</span><span class="p">],</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">AppModule</span> <span class="p">{}</span>
</code></pre>

</div>



<h3>
  
  
  9. Use the Prisma Service
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// src/app.controller.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">Controller</span><span class="p">,</span> <span class="nx">Get</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@nestjs/common</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">PrismaService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./services/prisma.service</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Controller</span><span class="p">()</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">AppController</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="k">readonly</span> <span class="nx">prismaService</span><span class="p">:</span> <span class="nx">PrismaService</span><span class="p">)</span> <span class="p">{}</span>

  <span class="p">@</span><span class="nd">Get</span><span class="p">()</span>
  <span class="k">async</span> <span class="nf">getUsers</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">prismaService</span><span class="p">.</span><span class="nx">user</span><span class="p">.</span><span class="nf">findMany</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>💡 This is a minimal setup and not production-ready. In a real app, you shouldn’t call a service directly from the controller. But this is a Prisma + NestJS set up tutorial - so I don’t care.</p>
</blockquote>

<h3>
  
  
  10. Run the Server
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pnpm run start:dev
</code></pre>

</div>



<p>And that’s it! Populate your database and test the endpoint.</p>

<p>If you run into any issues, let me know in the comments. Hope you enjoyed the ride!</p>

