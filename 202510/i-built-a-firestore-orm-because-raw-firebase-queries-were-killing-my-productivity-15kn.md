---
Title: I Built a Firestore ORM Because Raw Firebase Queries Were Killing My Productivity
Description: 
Author: HBFL3Xx
Date: 2025-10-16T22:04:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>Honestly, I hit a wall. After spending years building backends with Firestore, I was spending more time writing boilerplate than actually shipping features. Every collection meant writing the same CRUD patterns again, managing soft deletes manually, dealing with validation scattered everywhere, and constantly running into composite index errors at 2 AM.<br>
I looked for solutions, but the existing Firestore ORMs either felt abandoned or were too opinionated for real-world work. So I built FirestoreORM—basically the tool I wish existed when I started.</p>

<p><strong>What's Different Here</strong><br>
The core philosophy is: get out of your way when you need to move fast, but catch mistakes before they hit production.<br>
Type-safe by default. Everything's fully typed with TypeScript inference. No more guessing what fields exist or what type a query returns.</p>

<p><strong>Validation that actually works</strong>. Drop a Zod schema in, and it automatically validates every create and update before it touches Firestore. Invalid data gets caught instantly with clear error messages.</p>

<p><strong>Soft deletes built-in</strong>. Your data isn't actually gone—just marked as deleted. Recovery is one method call away. Queries automatically exclude deleted docs unless you ask otherwise. Audit trails, accidental deletion recovery, compliance—it's all there.</p>

<p><strong>Lifecycle hooks that feel natural</strong>. Need to send an email after someone signs up? Log changes for auditing? Run cleanup on deletion? Just hook into beforeCreate, afterCreate, etc. No messy callback hell.</p>

<p><strong>Query builder that doesn't suck</strong>. Filtering, sorting, pagination, aggregations—all chainable and intuitive. Cursor-based pagination by default (way cheaper than offset pagination on large datasets).</p>

<p><strong>Transactions done right</strong>. Critical operations like money transfers or inventory updates just work atomically. The ORM handles the complexity.</p>

<p><strong>Real Code, Real Speed</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Set up once
const userRepo = FirestoreRepository.withSchema&lt;User&gt;(
  db,
  'users',
  userSchema
);

// Create (auto-validated)
const user = await userRepo.create({
  name: 'Alice',
  email: 'alice@example.com'
});

// Query (fully typed)
const activeUsers = await userRepo.query()
  .where('status', '==', 'active')
  .where('age', '&gt;', 18)
  .orderBy('createdAt', 'desc')
  .paginate(20);

// Soft delete (recoverable)
await userRepo.softDelete(user.id);
await userRepo.restore(user.id); // instant recovery
</code></pre>

</div>



<p>No ceremony. Just readable, fast code.</p>

<p><strong>Framework Agnostic</strong><br>
Whether you're in Express, NestJS, Fastify, Next.js, or just a plain Node.js script, it works. No vendor lock-in—it's all built on the Firebase Admin SDK you're already using.</p>

<p><strong>What You Get</strong></p>

<ul>
<li>Full TypeScript support with intelligent type inference</li>
<li>Zod schema validation baked in</li>
<li>Soft deletes that actually save time</li>
<li>Lifecycle hooks for clean side effects</li>
<li>Query builder with pagination, aggregation, streaming</li>
<li>Transaction support for critical operations</li>
<li>Subcollection support</li>
<li>Real-time listeners with onSnapshot</li>
<li>Built-in error handling</li>
</ul>

<p>Check it out: <strong>github.com/HBFLEX/spacelabs-firestoreorm</strong><br>
         <strong><a href="https://www.npmjs.com/package/@spacelabstech/firestoreorm" rel="noopener noreferrer">https://www.npmjs.com/package/@spacelabstech/firestoreorm</a></strong></p>

<p>Installation is a one-liner:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm install @spacelabstech/firestoreorm firebase-admin zod
</code></pre>

</div>



<p>The docs have everything—real-world examples with e-commerce orders, multi-tenant apps, social feeds, performance cost breakdowns, best practices. I spent a lot of time on this because I wanted to ship something actually useful, not just another half-baked library.</p>

<p>This is my first open-source library project, so I'm genuinely interested in feedback. If you've dealt with Firestore pain, try it out and let me know what you think. Issues, suggestions, contributions—all welcome.</p>

<p>Happy coding.</p>

