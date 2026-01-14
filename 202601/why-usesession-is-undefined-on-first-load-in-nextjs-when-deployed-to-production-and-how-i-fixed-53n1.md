---
Title: Why useSession() Is undefined on First Load in Next.js When Deployed to Production (and how i fixed it)
Description: 
Author: Athashri Keny
Date: 2026-01-14T21:03:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>When I deployed my SaaS (Next.js app) to production, I noticed something strange. On the first load, my console showed session undefined, but when I refreshed the page (for example /dashboard), everything worked perfectly. Even more confusing, this never happened on localhost—it only happened in production. At first, I thought my auth token was broken, but it turned out it wasn’t. This blog explains what was actually happening, why this bug appears only in production, and how to fix it properly.</p>

<p>The setup I was using was very common: Next.js (App Router), NextAuth, useSession() on the client, and a protected dashboard page.</p>

<p>he problem appeared on the initial page load. I had the following code:<br>
<code><br>
const { data: session } = useSession();<br>
console.log(session);</code></p>

<p>The output was undefined. However, after refreshing the page, the output became { user: {...}, expires: "..." }. This raised the main question: why is the session undefined on first load but works after refresh?</p>

<p>The real reason is that useSession() is asynchronous. This was the key thing I misunderstood at first. When the page renders, React renders components immediately, and at the same time useSession() starts fetching the session. At that moment, the session is not available yet, so console.log(session) prints session = undefined with status = "loading". This is expected behavior.</p>

<p>This issue does not usually appear on localhost because of speed differences. On localhost, the app is slower, so the session often resolves early before you notice the problem. In production, the app loads much faster, so the render happens before the session is ready, which exposes the issue.</p>

<p>The correct fix on the client side starts with always checking the session status:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const { data: session, status } = useSession();

if (status === "loading") {
  return null;
}

if (!session) {
  redirect("/login");
}

</code></pre>

</div>



<p>The second important fix is guarding your useEffect so it does not run before the session exists:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>useEffect(() =&gt; {
  if (!session) return;

  fetchBills(session.user.id);
}, [session]);

</code></pre>

</div>



<p>The big lesson I learned from this is that rendering happens before async data is ready, so you should always assume undefined at least once. This does not apply only to useSession(), but also to API calls, database fetches, React Query, and any asynchronous data in React.</p>

<p>Production bugs don’t mean you’re bad—they mean you’re learning real-world development.</p>

<p>Thanks for reading. If this helped you, feel free to share or connect with me.<br>
Athashri Keny</p>

