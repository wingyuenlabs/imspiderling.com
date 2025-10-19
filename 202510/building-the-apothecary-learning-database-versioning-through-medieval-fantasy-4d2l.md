---
Title: Building The Apothecary: Learning Database Versioning Through Medieval Fantasy
Description: 
Author: RAMI GHARBI
Date: 2025-10-19T21:18:54.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Picture this: You're about to deploy a major pricing update to your e-commerce site. Your hands hover over the keyboard. One wrong migration and you could corrupt production data. Sound familiar?</p>

<p>I built <strong>The Apothecary</strong> to explore a different approach: treating databases like Git repositories. what if, to test a new promotion, you could branch your database? Go back to any point in time? Test multiple pricing strategies with no risks? it sure sounds like a dream come true.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyn4cblhsaxgy5y5mte6p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyn4cblhsaxgy5y5mte6p.png" alt="18 medieval products with dynamic pricing and promotions" width="800" height="434"></a></p>

<p>This isn't theoretical. whithin two days, I built a fully functional e-commerce demo site using <strong>Guepard</strong>, a database versioning system that brings Git-like workflows to PostgreSQL. Here's what i discovered about database versioning, why it is important, and how it flips in our head the way we think about data.</p>




<h2>
  
  
  The Problem: Database Changes Are Scary
</h2>

<p>Let's be honest: sporatic database changes are a developer's nightmare. Unlike code, where Git gives us branches, commits, and instant rollbacks, databases feel... permanent. Make a mistake and you'll find yourself struggling with SQL scripts at 2 AM trying to restore from backups.</p>

<p>Traditional workflows look like this:</p>

<ol>
<li>Write migration scripts</li>
<li>Test in staging (hope it matches production)</li>
<li>Deploy during low-traffic hours</li>
<li>Hold your breath and pray</li>
<li>If something breaks, scramble to write rollback SQL</li>
</ol>

<p>We've all heard the stories: a misplaced decimal in a pricing update, a migration that corrupts data, a schema change that accidently breaks production. When disaster strikes, rolling back means struggle, manual SQL queries, data recovery from backups, and hours of stress. The current tools we rely on make database mistakes feel permanent and recovery feel arduous.</p>

<p>What if databases could work more like Git? That's the question Guepard answers.</p>




<h2>
  
  
  The Solution: Git for Databases
</h2>

<p>Guepard introduces three core concepts borrowed from Git:</p>

<h3>
  
  
  1. Branches
</h3>

<p>Create isolated database clones in seconds. Each branch has its own data and schema. That is perfect for testing promotions or new features without messing with production.</p>

<h3>
  
  
  2. Commits (Snapshots)
</h3>

<p>Save the current state of your database with a message. Unlike backups, commits are instant and lightweight - think the equivalent of <code>git commit</code> but for data.</p>

<h3>
  
  
  3. Checkout
</h3>

<p>Jump to any branch or commit instantly. Made a mistake? Checkout the previous commit. Want to compare strategies? Checkout different branches side-by-side.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Traditional approach - risky!</span>
<span class="k">await</span> <span class="nx">prisma</span><span class="p">.</span><span class="nx">item</span><span class="p">.</span><span class="nf">updateMany</span><span class="p">({</span>
  <span class="na">data</span><span class="p">:</span> <span class="p">{</span> <span class="na">price</span><span class="p">:</span> <span class="p">{</span> <span class="na">multiply</span><span class="p">:</span> <span class="mi">10</span> <span class="p">}</span> <span class="p">}</span> <span class="c1">// OOPS! Should be "add"</span>
<span class="p">})</span>
<span class="c1">// Now you need backup recovery...</span>

<span class="c1">// With Guepard - safe!</span>
<span class="c1">// 1. Create branch for testing</span>
<span class="c1">// 2. Test the change</span>
<span class="c1">// 3. If wrong, just delete the branch</span>
<span class="c1">// 4. If right, merge to main</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63ufj4avuyhtyqbt1sza.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63ufj4avuyhtyqbt1sza.png" alt="Visual branch history - just like Git, but for your database" width="800" height="421"></a></p>

<p>For The Apothecary, I built four real-world scenarios to demonstrate these concepts and guepard capabilities.</p>




<h2>
  
  
  Four Real-World Scenarios
</h2>

<h3>
  
  
  Scenario 1: Testing a Promotion Without Fear
</h3>

<p><strong>The Use Case</strong>: Marketing wizards want to test a "15% off ingredients" promotion called "Lunar Blessing."</p>

<p>In a traditional setup, I'd see myself either:</p>

<ul>
<li>Test in staging (which never perfectly matches production)</li>
<li>Deploy to production and cross my fingers</li>
<li>Set up complex feature flags</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvmcwdhf7hykdjuwapsw4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvmcwdhf7hykdjuwapsw4.png" alt="Dragon's Breath Essence showing 15% discount on the lunar-blessing-test branch" width="360" height="643"></a></p>

<p>With Guepard, I created a branch called <code>lunar-blessing-test</code> in the web dashboard, checked out, and ran my seed script:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Dashboard: Create branch "lunar-blessing-test" → Checkout</span>
npm run seed:lunar-blessing
npm run dev
</code></pre>

</div>



<p>Within seconds, I had in my disposition a complete copy of production data with the promotion active. I found myself being able to freely test the pricing, logic, verify how the UI was rendered, and test orders without messing with the main branch. When done, i could easily checkout to the main branch. Zero risk.</p>

<h3>
  
  
  Scenario 2: Instant Recovery from Disasters
</h3>

<p><strong>The Use Case</strong>: The apothecary has been cursed with a 10x pricing disaster! Let's revert it safely.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu54rxf8pm3sq6ou00pza.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu54rxf8pm3sq6ou00pza.png" alt="The cursed-batch seed multiplies all prices by 10" width="800" height="351"></a></p>

<p>I created a commit called "before-price-update", then ran the cursed batch script that multiplies all prices by 10:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// seed-cursed-batch.ts</span>
<span class="k">await</span> <span class="nx">prisma</span><span class="p">.</span><span class="nx">item</span><span class="p">.</span><span class="nf">updateMany</span><span class="p">({</span>
  <span class="na">data</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">price</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">multiply</span><span class="p">:</span> <span class="mi">10</span>  <span class="c1">// Simulating a pricing disaster</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">})</span>
</code></pre>

</div>



<p>In a real-life scenario, this would be catastrophic:<br>
Prices exploded and everyone is sweating profusely. With Guepard? I opened the dashboard, found the "before-price-update" commit, clicked "Checkout." Three clicks, five seconds and magically the database was restored perfectly.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4vimnu9t1v4uww5ewyvs.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4vimnu9t1v4uww5ewyvs.gif" alt="Restoring from commit in the Guepard dashboard" width="426" height="240"></a></p>
<h3>
  
  
  Scenario 3: Parallel Testing
</h3>

<p><strong>The Use Case</strong>: Test two discount strategies simultaneously: 20% vs 25% off potions.</p>

<p>I created two branches: <code>solstice-20-percent</code> and <code>solstice-25-percent</code>. By checking out each branch, I could run the app and see different data states. In a real application, one would process test orders on both cases and compare metrics to deduce the best strategy empirically.</p>

<p>This is impossible with traditional databases unless you maintain separate servers. Guepard made it trivial.</p>
<h3>
  
  
  Scenario 4: Schema Evolution Without Downtime
</h3>

<p><strong>The Use Case</strong>: Add a customer reviews feature (new <code>Review</code> model) without disrupting the main site.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvrpe13a1nkr752sh33k0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvrpe13a1nkr752sh33k0.png" alt="Review system tested in isolation" width="800" height="334"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// schema-with-reviews.prisma (new model)</span>
<span class="nx">model</span> <span class="nx">Review</span> <span class="p">{</span>
  <span class="nx">id</span>        <span class="nb">String</span>   <span class="p">@</span><span class="nd">id</span> <span class="p">@</span><span class="nd">default</span><span class="p">(</span><span class="nf">cuid</span><span class="p">())</span>
  <span class="nx">itemId</span>    <span class="nb">String</span>
  <span class="nx">rating</span>    <span class="nx">Int</span>      <span class="c1">// 1-5 stars</span>
  <span class="nx">comment</span>   <span class="nb">String</span>
  <span class="nx">author</span>    <span class="nb">String</span>
  <span class="nx">item</span>      <span class="nx">Item</span>     <span class="p">@</span><span class="nd">relation</span><span class="p">(</span><span class="nx">fields</span><span class="p">:</span> <span class="p">[</span><span class="nx">itemId</span><span class="p">],</span> <span class="nx">references</span><span class="p">:</span> <span class="p">[</span><span class="nx">id</span><span class="p">])</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I created a <code>reviews-feature</code> branch, switched my Prisma schema, pushed the migration, seeded 20 fantasy character reviews, checked everything worked as inteded. As easy as that. what if it had broken something? no harm done.</p>




<h2>
  
  
  Technical Choices &amp; Architecture
</h2>

<p>I built The Apothecary with <strong>Next.js 14</strong> using the App Router, <strong>TypeScript</strong> for type safety, and <strong>Prisma</strong> as the ORM. The UI uses <strong>TailwindCSS</strong> for that medieval aesthetic - dark backgrounds, gold accents, serif fonts that feel ancient.</p>

<p>For state management, I chose <strong>Zustand</strong> to handle the shopping cart. No need for Redux complexity.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// store/cart.ts - Simple cart management with Zustand</span>
<span class="k">export</span> <span class="kd">const</span> <span class="nx">useCartStore</span> <span class="o">=</span> <span class="nx">create</span><span class="o">&lt;</span><span class="nx">CartState</span><span class="o">&gt;</span><span class="p">((</span><span class="kd">set</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="na">items</span><span class="p">:</span> <span class="p">[],</span>
  <span class="na">addItem</span><span class="p">:</span> <span class="p">(</span><span class="nx">item</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
    <span class="na">items</span><span class="p">:</span> <span class="p">[...</span><span class="nx">state</span><span class="p">.</span><span class="nx">items</span><span class="p">,</span> <span class="nx">item</span><span class="p">]</span>
  <span class="p">})),</span>
  <span class="na">removeItem</span><span class="p">:</span> <span class="p">(</span><span class="nx">id</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">set</span><span class="p">((</span><span class="nx">state</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">({</span>
    <span class="na">items</span><span class="p">:</span> <span class="nx">state</span><span class="p">.</span><span class="nx">items</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">i</span> <span class="o">=&gt;</span> <span class="nx">i</span><span class="p">.</span><span class="nx">id</span> <span class="o">!==</span> <span class="nx">id</span><span class="p">)</span>
  <span class="p">})),</span>
<span class="p">}))</span>
</code></pre>

</div>



<p>The database schema is quite simple: Items, Promotions, Orders. Later I added Reviews as a way to successfully demonstrate schema evolution. Each model uses Prisma's type-safe queries and no raw SQL was needed.</p>




<h2>
  
  
  Key Learnings
</h2>

<h3>
  
  
  1. Database Branches Change Everything
</h3>

<p>The shift I noticed in my approach is astonishing. Once you can branch your database, you feel as if a heavy burden was lifted off your back. Experiments become cheap. Testing becomes thorough. Mistakes become trivial.</p>

<h3>
  
  
  2. Recovery Shouldn't Be Stressful
</h3>

<p>Traditional backup/restore is quite the Sisyphean task. It is slow and stress-inducing. Guepard's commit system is quite the game changer: instant, confident, reversible.</p>

<h3>
  
  
  3. Staging Never Matches Production
</h3>

<p>This project, even with its small scope, already proves that staging environments drift from production. Testing directly on a production branch alleviates confusion and ensures compatibility. It is a huge time saver.</p>

<h3>
  
  
  4. Schema Migrations Need Isolation
</h3>

<p>Adding the Reviews feature in an isolated branch meant I could test to my heart's content: schema changes, data seeding, UI updates without laying a finger on main. This is how schema changes should work.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2a9gjdo5yrspcyv4rq07.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2a9gjdo5yrspcyv4rq07.png" alt="Multiple database environments running in parallel" width="800" height="249"></a></p>




<h2>
  
  
  What's Next?
</h2>

<p>This project enlightened me on the importance of database versionning. It is a huge shsift from how we approach data. Although Guepard is still in its early days, the concept is already proven. I can't express how excited i am to see ambitious tools like this mature. One can only imagine the endless possibilities such as data branches in CI/CD pipelines and automated testing on production-clone branches</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nub7jcrnxonk3cr5kx4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nub7jcrnxonk3cr5kx4.png" alt="The Apothecary: Where medieval fantasy meets modern database practices" width="800" height="423"></a><br>
If you're building something with databases(who isn't?). I wholeheartedly encourage you to explore versioning tools like Guepard. Start simple, try projects like this. Nothing beats the peace of minds that comes neatly packaged with it.</p>




<h2>
  
  
  Try It Yourself
</h2>

<ul>
<li>
<strong>GitHub Repository</strong>: <a href="https://github.com/ramigharbi/guepard_demo/" rel="noopener noreferrer">https://github.com/ramigharbi/guepard_demo/</a>
</li>
<li>
<strong>Live Demo</strong>: <a href="https://youtu.be/WiIZls-ug-8?si=SfQ1AwcArWKK_H9a" rel="noopener noreferrer">https://youtu.be/WiIZls-ug-8?si=SfQ1AwcArWKK_H9a</a>
</li>
<li>
<strong>Guepard</strong>: <a href="https://guepard.run" rel="noopener noreferrer">https://guepard.run</a>
</li>
</ul>

<p>The entire project is open source. Feel free to clone it, break it and fix it as easily with a quick checkout. That's the beauty of versioned databases.</p>




<h2>
  
  
  Final Thoughts: The Paradigm Shift
</h2>

<p>Building The Apothecary struck me deep: our approach to databases is archaic! We've easily accepted the inherent risk and irreversibility of data even  though it is a limitation of our tools, not a property of databases themselves. Database versioning addresses the elephant in the room: Given the ability to branch, commit and rollback as easily as code, the entire development approach transforms. The fear of testing is alleviated. Experimentation becomes accessible. Recovery becomes instant. More than a niche feature, it is rather a new perspective on data infrastructure</p>




<p><em>What database challenges have you faced? Have you tried database versioning tools? Share your experiences in the comments!</em></p>

