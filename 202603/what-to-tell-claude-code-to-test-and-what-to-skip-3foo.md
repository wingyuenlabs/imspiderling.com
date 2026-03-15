---
Title: What to Tell Claude Code to Test (and What to Skip)
Description: 
Author: Ramon
Date: 2026-03-15T21:02:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you're using Claude Code to build apps, you've probably noticed it loves writing tests. Ask it to build a feature and it'll offer to test it. Ask it to fix a bug and it'll suggest adding coverage. Left to its own devices, it will happily generate hundreds of tests for your application.</p>

<p>Most of them won't matter.</p>

<p>Here's the filter I use before writing any test: <strong>if this breaks silently, what happens?</strong></p>

<p>If the answer is "nothing for a while" or "a user gets a slightly wrong result," skip it. If the answer is "data gets corrupted," "someone gets charged incorrectly," or "the failure is invisible until it's too late," write the test.</p>

<p>That's the whole framework. Everything below is just applying it.</p>

<p>One important caveat: this is a strategy for the build phase. If you're shipping an MVP or a side project and you're the only person working on it, this is the right approach. Once you have a team, long-lived systems, and daily deploys, your testing philosophy needs to evolve. But that's a different post.</p>




<h2>
  
  
  What Claude Code will test if you don't guide it
</h2>

<p>Claude Code defaults to thoroughness. It will test that your database queries return data, that your UI components render, that your API routes respond with 200, that your forms submit, that your auth redirects work.</p>

<p>These tests aren't wrong. They're just low value at the start. A broken UI component is visible the moment you open the browser. A failing form submit takes five seconds of manual testing to catch. You don't need automation for things that announce themselves.</p>

<p>What you need automation for are the things that fail quietly.</p>




<h2>
  
  
  The areas worth testing
</h2>

<h3>
  
  
  Business logic with edge cases
</h3>

<p>Any function that makes a decision based on data is worth testing. Not the happy path, which you'll notice when it breaks. The edge cases are what get you.</p>

<p>In a monitoring tool I built called <a href="https://pulsemon.dev" rel="noopener noreferrer">PulseMon</a>, the core checker function determines whether a monitor is late based on the last ping timestamp, the expected interval, and the grace period. It's about 20 lines of pure logic with 9 tests covering scenarios like: what if the last ping arrived exactly at the deadline, what if there's no ping at all, what if the grace period is zero.</p>

<p>That function runs every 60 seconds for every user. If it's wrong, monitors either never alert or alert constantly. Neither failure is obvious until users complain. That's the definition of something worth testing.</p>

<p><strong>Tell Claude Code:</strong> <em>"Write tests for the core business logic only. Focus on edge cases, not the happy path."</em></p>




<h3>
  
  
  External integrations that handle money or critical data
</h3>

<p>Stripe webhooks. Payment processing. Anything where a bug means someone gets charged twice, doesn't get charged at all, or loses access to something they paid for.</p>

<p>These are worth testing because the failure modes are severe and the bugs are subtle. A wrong status code, a missing field, an event type you didn't handle. These don't throw obvious errors. They silently do the wrong thing.</p>

<p>For PulseMon's Stripe webhook handler there are 10 tests covering: subscription created, updated, deleted, payment failed, and an invalid signature. That last one matters because without it, anyone can send fake webhook events to your endpoint.</p>

<p><strong>Tell Claude Code:</strong> <em>"Test the Stripe webhook handler. Cover subscription created, updated, deleted, and invalid signature. Mock the Stripe library at the module boundary."</em></p>

<p>Unit tests cover your logic. Before going live, run one real test with the Stripe CLI to verify the raw body handling and your webhook secret are configured correctly. That catches the two bugs tests can't.</p>




<h3>
  
  
  Authorisation boundaries
</h3>

<p>Not "can a user log in" since that's visible immediately if it breaks. The subtle version: can user A access user B's data?</p>

<p>In any multi-user app, the query that fetches data scoped to the current user is the one worth testing. The bug that leaks one user's data to another is catastrophic and invisible. You won't catch it in manual testing because you're always logged in as the same user.</p>

<p><strong>Tell Claude Code:</strong> <em>"Write tests that verify a user cannot access another user's resources. Mock the database and test that all queries include the userId filter."</em></p>




<h3>
  
  
  Anything that runs on a schedule without human oversight
</h3>

<p>Cron jobs, background workers, scheduled cleaners. If nobody is watching them run, you need tests for the logic inside. Not integration tests that actually fire the job, but unit tests that cover what the job decides.</p>

<p>If your cleanup job deletes records older than 30 days, test that it deletes the right records and leaves the wrong ones alone. These jobs run at 3am and nobody checks them.</p>




<h2>
  
  
  What to tell Claude Code to skip
</h2>

<h3>
  
  
  UI tests during early development
</h3>

<p>Skipping UI tests is not a blanket recommendation. It's a prioritisation call for when you're moving fast.</p>

<p>During the initial build, a broken button is visible the moment you look at the screen. Testing it at that stage slows you down without adding much. But once your core conversion paths exist, things like signup, onboarding, and checkout, they are worth protecting. A global CSS change or a Tailwind config update can silently hide a button on mobile and you won't catch it manually every time.</p>

<p>The practical version: skip UI tests while you're building, add them for your most critical flows once they're stable.</p>

<p><strong>Tell Claude Code:</strong> <em>"Skip UI component tests for now. We'll add coverage for critical conversion paths once they're stable."</em></p>




<h3>
  
  
  Simple CRUD operations
</h3>

<p>Creating, reading, updating, and deleting records doesn't need tests if it's just calling an ORM method. The ORM is already tested. Your thin wrapper around it doesn't need coverage.</p>

<p>The exception is CRUD that enforces business rules. A create operation that checks plan limits before inserting is worth testing. A create operation that just calls <code>db.insert()</code> is not.</p>




<h3>
  
  
  Auth configuration, but not auth logic
</h3>

<p>Libraries like Auth.js are tested by their maintainers. Whether your sign-in redirect fires correctly doesn't need coverage. You'll know immediately if it breaks.</p>

<p>What is worth testing is the authorisation logic you write yourself: middleware that checks roles, functions that decide what a user can see, session handling that scopes data correctly. Those are yours to own, and that's where leaks happen.</p>

<p><strong>Tell Claude Code:</strong> <em>"Skip testing that the auth redirect fires. Do test any custom middleware, role checks, or session scoping logic we've written."</em></p>




<h2>
  
  
  A prompt that actually works
</h2>

<p>When starting a new feature, give Claude Code this framing before asking it to write tests:</p>

<blockquote>
<p>"We only write tests for high-stakes areas where a silent failure would cause real damage. This means: core business logic with edge cases, external integrations that handle payments or critical data, and authorisation boundaries that prevent data leaks. Skip UI tests during initial build, basic CRUD, and standard auth library configuration. For each test you write, add a comment at the top of the test explaining in one sentence why a silent bug here would be a serious problem. If you can't write that sentence, the test shouldn't exist."</p>
</blockquote>

<p>The comment becomes permanent documentation for why the test was written. Six months from now, when you're wondering whether you can delete it, the answer is right there.</p>




<h2>
  
  
  Why the filter matters more than the framework
</h2>

<p>Coverage targets and exhaustive test suites are a different conversation to the one most solo developers building with AI assistance need to have right now. The more immediate problem is Claude Code generating 80 tests when 15 would have been enough, most of them testing things that would have been obviously broken on first look.</p>

<p>Start with the filter. Test the things that fail silently and cause real damage. Build coverage from there as your product matures.</p>

<p>Write tests for those. Skip everything else. Tell Claude Code exactly that and it'll spend its time on the things that actually matter.</p>

