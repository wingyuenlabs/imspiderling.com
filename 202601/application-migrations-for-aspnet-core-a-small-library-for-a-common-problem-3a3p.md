---
Title: Application Migrations for ASP.NET Core: A Small Library for a Common Problem
Description: 
Author: Sébastien Sougnez
Date: 2026-01-14T21:19:03.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Application Migrations for ASP.NET Core: A Small Library for a Common Problem
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fcdn-images-1.medium.com%2Fmax%2F7600%2F0%2AltKanbgMM9Oj9xlu" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fcdn-images-1.medium.com%2Fmax%2F7600%2F0%2AltKanbgMM9Oj9xlu" alt="Photo by [AbsolutVision](https://unsplash.com/@codzilla_swiss?utm_source=medium&amp;utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&amp;utm_medium=referral)" width="800" height="400"></a></p>

<h2>
  
  
  Managing Application Updates in ASP.NET Core
</h2>

<p>You know how it goes. You build an app, you deploy it, everything’s great. Then you need to push an update. Maybe you’re adding a new feature that requires some initial data. Or you need to transform existing records because you changed how something works. Or you just need to create a folder structure on first run.</p>

<p>If you’ve been doing this for a while, you probably have your own hacky solutions. A boolean flag in the database that says “did we run the v2 setup?”. Some code that checks if a file exists before creating default config. A bunch of <em>if</em> statements scattered across your <em>Program.cs</em>.</p>

<p>I’ve written all of these. They work, until they don’t.</p>

<p>The real issue is that we don’t have a proper way to version our application’s state — not the database schema, but the application itself.</p>

<h2>
  
  
  EF Core Migrations Don’t Cover Everything
</h2>

<p>EF Core migrations exist and they’re great for what they do: managing database schema changes. Need a new table? Migration. New column? Migration. Index? Migration.</p>

<p>But EF Core migrations run <strong>before</strong> your application code. They deal with structure, not data. They don’t handle things like:</p>

<ul>
<li><p>Seeding initial data after a schema change</p></li>
<li><p>Sending a one-time notification when a feature goes live</p></li>
<li><p>Creating file system structures</p></li>
<li><p>Running cleanup tasks when upgrading from v1 to v2</p></li>
<li><p>Transforming existing data when you change how something is stored</p></li>
</ul>

<p>You could try cramming this stuff into EF migrations using raw SQL, but honestly, that’s a mess. No dependency injection, no access to your services, no proper C# code. And good luck testing any of it.</p>

<h2>
  
  
  A Small Library for Application Migrations
</h2>

<p>I wrote a library to handle this. The idea is simple: give your application the same versioning semantics that your database schema already has.</p>

<p>Here’s what a migration looks like:<br>
</p>
<div class="ltag_gist-liquid-tag">
  
</div>
<br>
Notice a few things:

<ol>
<li>
<strong>Constructor injection works</strong>: Your migrations are full citizens of the DI container.</li>
<li>
<strong>Versions are explicit:</strong> No date-based naming, no magic strings. Just <em>System.Version</em>.</li>
<li>
<strong>The *FirstTime *flag</strong>: This tells you if this version has ever been registered before.</li>
</ol>

<p>That last one is particularly useful. During development, the current version gets re-executed on every startup so you can iterate quickly. But the <code>FirstTime</code> flag lets you guard operations that should genuinely only happen once.</p>
<h2>
  
  
  The Lifecycle Hooks
</h2>

<p>Sometimes you need more control. Maybe you’re doing a complex data transformation where you need to capture data <strong>before</strong> a schema change and apply it <strong>after</strong>. The library gives you hooks for this:<br>
</p>
<div class="ltag_gist-liquid-tag">
  
</div>
<br>
The cache dictionary is shared with your migrations, so you can read that captured data in your <em>UpAsync()</em> method and transform it however you need.
<h2>
  
  
  Setting It Up
</h2>

<p>The setup is pretty minimal:<br>
</p>
<div class="ltag_gist-liquid-tag">
  
</div>
<br>
When you configure a DbContext, the library automatically runs <em>Database.MigrateAsync()</em> before executing your application migrations. So you don’t need to call it yourself — EF Core schema changes are applied first, then your application migrations run.

<p>You do need to implement the storage part yourself — where do you want to track which versions have been applied? Most people use a database table, but you could use a file, Redis, whatever makes sense for your setup.</p>

<p>Here’s a basic database implementation:<br>
</p>
<div class="ltag_gist-liquid-tag">
  
</div>


<h2>
  
  
  Multi-Server Deployments
</h2>

<p>If you’re running multiple instances of your app (load balancing, replicas, etc.), you probably don’t want all of them trying to run migrations at the same time. The engine has a <code>ShouldRun</code> property you can override to control this.</p>

<p>A common pattern is to designate one instance as the “master” via configuration:<br>
</p>
<div class="ltag_gist-liquid-tag">
  
</div>
<br>
Then in your deployment, only the primary instance gets <em>Migrations:IsMaster = true</em>. The others will skip migrations entirely and start normally. This way you avoid race conditions and duplicate executions without needing distributed locks.

<h2>
  
  
  Real-World Use Cases
</h2>

<p>Here are some scenarios where this actually helps:</p>

<p><strong>Feature rollout with data seeding</strong>: You’re adding a new “Categories” feature. The EF migration creates the table, but you need to populate it with default categories and assign existing products. One migration handles both the data creation and the assignments.</p>

<p><strong>Configuration evolution</strong>: Your app used to store settings one way, now it stores them differently. Write a migration that reads the old format and writes the new one.</p>

<p><strong>Environment setup</strong>: First deployment to a new server? Create required directories, generate default config files, set up whatever your app needs to run.</p>

<p><strong>Notification on upgrade</strong>: Want to email admins when a major version is deployed? Put it in a migration, guarded by <code>FirstTime</code>.</p>

<h2>
  
  
  Why No Down() Method?
</h2>

<p>There’s no <em>Down()</em> method for rollbacks. In my experience, rollback methods are rarely tested and often broken when you actually need them. I find it cleaner to write a new forward migration that undoes whatever you need to undo. Version 1.1 messed something up? Version 1.2 fixes it.</p>

<h2>
  
  
  <strong>Getting It</strong>
</h2>

<p>The package is on NuGet:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>dotnet add package AreaProg.AspNetCore.Migrations
</code></pre>

</div>

<p>It targets .NET 6, 8, 9, and 10. Source is on GitHub if you want to poke around: <a href="https://github.com/ssougnez/aspnet-migrations" rel="noopener noreferrer">https://github.com/ssougnez/aspnet-migrations</a></p>

<p>— —</p>

<p>This isn’t revolutionary stuff. It’s just filling a gap that I kept running into project after project. If you’ve ever written a “run once on first deploy” hack, maybe give it a try.</p>

<p>Questions? Feedback? I’d love to hear how others have been solving this problem.</p>

