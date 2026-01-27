---
Title: I Didn't Like Other Workout Tracking Apps, So I Built My Own.
Description: 
Author: Dominic Magnifico
Date: 2026-01-27T21:37:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been lifting for years. And I've tried every workout app out there. Strong. Hevy. FitNotes. Ladder. They all fall short in the same predictable way.</p>

<p>You're 30 seconds into your rest period, sweaty, trying to log a set, and the app wants you to:</p>

<ol>
<li>Tap "Add Set"</li>
<li>Tap the weight field</li>
<li>Wait for the keyboard</li>
<li>Type the weight</li>
<li>Dismiss the keyboard</li>
<li>Tap the reps field</li>
<li>Wait for the keyboard AGAIN</li>
<li>Type the reps</li>
<li>Dismiss the keyboard AGAIN</li>
<li>Scroll to find the "Save" button</li>
<li>Tap "Save"</li>
</ol>

<p>By this point your rest timer expired 45 seconds ago and you're late for your next set.</p>

<h2>
  
  
  The Problem Nobody Was Solving
</h2>

<p>Most workout apps treat logging like you're sitting at a desk with a cup of coffee and unlimited time.</p>

<p>Meanwhile, in the real world, you've got maybe 60-90 seconds between sets.</p>

<p>So we built <a href="https://opentrainer.app" rel="noopener noreferrer">OpenTrainer</a>. Two taps per set.</p>

<h2>
  
  
  The Tech Stack (And Why We Chose It)
</h2>

<p>Here's what we're running:</p>

<ul>
<li>Next.js 16</li>
<li>Convex</li>
<li>Clerk</li>
<li>OpenRouter</li>
<li>shadcn</li>
<li>Vercel</li>
</ul>

<h3>
  
  
  Why Convex?
</h3>

<p>Convex is awesome for this use case.</p>

<p>Traditional workout apps use REST APIs. You tap "Log Set" → HTTP request → server processes → database write → HTTP response → UI updates. If you're on sketchy gym WiFi, that's 2-3 seconds of loading spinner hell.</p>

<p>Convex gives us real-time sync with zero setup. The mutation fires, the optimistic update shows instantly, and the actual database write happens in the background. If it fails, it rolls back automatically. If you go offline mid-workout, it queues the mutations and syncs when you're back online.</p>

<p>We have optimistic updates working with <strong>seven lines of code</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">logSet</span> <span class="o">=</span> <span class="nf">useConvexMutation</span><span class="p">(</span><span class="nx">api</span><span class="p">.</span><span class="nx">workouts</span><span class="p">.</span><span class="nx">addLiftingEntry</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">handleLogSet</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">logSet</span><span class="p">({</span>
    <span class="nx">workoutId</span><span class="p">,</span>
    <span class="na">exerciseName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Bench Press</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">clientId</span><span class="p">:</span> <span class="nf">generateClientId</span><span class="p">(),</span> <span class="c1">// Deduplication</span>
    <span class="na">kind</span><span class="p">:</span> <span class="dl">"</span><span class="s2">lifting</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">lifting</span><span class="p">:</span> <span class="p">{</span> <span class="na">setNumber</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">reps</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span> <span class="na">weight</span><span class="p">:</span> <span class="mi">225</span><span class="p">,</span> <span class="na">unit</span><span class="p">:</span> <span class="dl">"</span><span class="s2">lb</span><span class="dl">"</span> <span class="p">}</span>
  <span class="p">});</span>
  <span class="c1">// UI updates instantly. Done.</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Convex handles:</p>

<ul>
<li>Optimistic UI updates</li>
<li>Deduplication via <code>clientId</code>
</li>
<li>Offline queueing</li>
<li>Conflict resolution</li>
<li>Real-time sync across devices</li>
</ul>

<h3>
  
  
  The Schema
</h3>

<p>We use a discriminated union pattern for entries. One table, multiple exercise types:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// entries table</span>
<span class="p">{</span>
  <span class="nl">kind</span><span class="p">:</span> <span class="dl">"</span><span class="s2">lifting</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">cardio</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">mobility</span><span class="dl">"</span><span class="p">,</span>
  <span class="nx">lifting</span><span class="p">?:</span> <span class="p">{</span>
    <span class="na">setNumber</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span>
    <span class="na">reps</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span>
    <span class="na">weight</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span>
    <span class="na">unit</span><span class="p">:</span> <span class="dl">"</span><span class="s2">kg</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">lb</span><span class="dl">"</span><span class="p">,</span>
    <span class="nx">rpe</span><span class="p">?:</span> <span class="kr">number</span>
  <span class="p">},</span>
  <span class="nx">cardio</span><span class="p">?:</span> <span class="p">{</span>
    <span class="na">durationSeconds</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span>
    <span class="nx">distance</span><span class="p">?:</span> <span class="kr">number</span><span class="p">,</span>
    <span class="nx">intensity</span><span class="p">?:</span> <span class="kr">number</span>
  <span class="p">},</span>
  <span class="nx">mobility</span><span class="p">?:</span> <span class="p">{</span>
    <span class="na">holdSeconds</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span>
    <span class="nx">perSide</span><span class="p">?:</span> <span class="nx">boolean</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This beats having separate tables for <code>lifting_entries</code>, <code>cardio_entries</code>, <code>mobility_entries</code>. Querying a workout's full history is a single query. This way if we want to add a new exercise category we just extend the union.</p>

<h3>
  
  
  Next.js 16
</h3>

<p><strong>What worked:</strong></p>

<ul>
<li>Parallel routes made the bottom navigation clean</li>
<li>Metadata API makes SEO easier</li>
<li>Clerk's new <code>clerkMiddleware</code> (v5+) works perfectly with Next.js 16</li>
</ul>

<p><strong>What hurt:</strong></p>

<ul>
<li>The dev server randomly decides to recompile everything. Often.</li>
<li>Hot reload occasionally needs a manual refresh</li>
<li>Some Convex types needed manual casting (minor annoyance)</li>
</ul>

<p><strong>Auth Setup:</strong> Clerk + Convex requires a JWT template in the Clerk dashboard. Standard stuff, nothing custom:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// convex/auth.config.ts</span>
<span class="k">export</span> <span class="k">default</span> <span class="p">{</span>
  <span class="na">providers</span><span class="p">:</span> <span class="p">[{</span>
    <span class="na">domain</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">CLERK_JWT_ISSUER_DOMAIN</span><span class="o">!</span><span class="p">,</span>
    <span class="na">applicationID</span><span class="p">:</span> <span class="dl">"</span><span class="s2">convex</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">}],</span>
<span class="p">}</span>
</code></pre>

</div>



<p>You just need to grab your JWT issuer URL from Clerk and set it as an env var. That's it.</p>

<h2>
  
  
  The AI Layer
</h2>

<p>Here's where we diverged from every other fitness app.</p>

<p>Most apps with "AI" just slap ChatGPT on a prompt and call it a day. We wanted something smarter.</p>

<h3>
  
  
  Why OpenRouter Instead of Direct APIs
</h3>

<p>We use <strong>OpenRouter</strong> as our AI gateway. One API, multiple models. Right now we're using <strong>Gemini 3 Flash</strong> through OpenRouter's unified endpoint.</p>

<p>Why OpenRouter?</p>

<ul>
<li>
<strong>No vendor lock-in</strong> - Switch models with one line of code</li>
<li>
<strong>Cost optimization</strong> - OpenRouter shows pricing for every model</li>
<li>
<strong>Fallback logic</strong> - Can implement automatic failover to Claude/GPT if Gemini is down</li>
<li>
<strong>One API key</strong> - Don't need separate Google, Anthropic, OpenAI accounts
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">"</span><span class="s2">https://openrouter.ai/api/v1/chat/completions</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">method</span><span class="p">:</span> <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
    <span class="dl">"</span><span class="s2">Authorization</span><span class="dl">"</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">OPENROUTER_API_KEY</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">},</span>
  <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span>
    <span class="na">model</span><span class="p">:</span> <span class="dl">"</span><span class="s2">google/gemini-3-flash-preview</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">messages</span><span class="p">:</span> <span class="p">[</span>
      <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">system</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">systemPrompt</span> <span class="p">},</span>
      <span class="p">{</span> <span class="na">role</span><span class="p">:</span> <span class="dl">"</span><span class="s2">user</span><span class="dl">"</span><span class="p">,</span> <span class="na">content</span><span class="p">:</span> <span class="nx">userMessage</span> <span class="p">},</span>
    <span class="p">],</span>
  <span class="p">}),</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  The Routine Generation Problem
</h3>

<p>Ask GPT-4 to generate a workout routine and you get:</p>

<ul>
<li>Generic "bro split" garbage</li>
<li>Exercises you don't have equipment for</li>
<li>Zero consideration for your actual goals</li>
<li>Hallucinated exercise names ("reverse cable overhead skull-tricep extension")</li>
</ul>

<p>We built an <strong>equipment audit</strong> during onboarding. User checks off what they have access to. Planet Fitness member? Cool, we know you have Smith machines and dumbbells, not barbells. Home gym? Tell us what you've got.</p>

<p>Then we pass that to Gemini with a structured prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">routinePrompt</span> <span class="o">=</span> <span class="s2">`
Generate a </span><span class="p">${</span><span class="nx">goal</span><span class="p">}</span><span class="s2"> program for </span><span class="p">${</span><span class="nx">experienceLevel</span><span class="p">}</span><span class="s2"> lifter.

EQUIPMENT AVAILABLE:
</span><span class="p">${</span><span class="nx">equipment</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="dl">"</span><span class="s2">, </span><span class="dl">"</span><span class="p">)}</span><span class="s2">

CONSTRAINTS:
- </span><span class="p">${</span><span class="nx">weeklyAvailability</span><span class="p">}</span><span class="s2"> days per week
- </span><span class="p">${</span><span class="nx">sessionDuration</span><span class="p">}</span><span class="s2"> minutes per session
- Must use ONLY the equipment listed above
- Use standard exercise names (no made-up movements)

OUTPUT FORMAT: JSON matching this schema...
`</span><span class="p">;</span>
</code></pre>

</div>



<p>This works <strong>way better</strong> than you'd expect. Gemini's actually pretty good at constraint satisfaction when you give it clear boundaries.</p>

<h3>
  
  
  The Training Load Calculation
</h3>

<p>Most apps calculate "volume" as <code>sets × reps × weight</code>. That's... fine. But it doesn't account for intensity.</p>

<p>We use a modified training load formula that factors in RPE (Rate of Perceived Exertion):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">calculateTrainingLoad</span><span class="p">(</span><span class="nx">entry</span><span class="p">:</span> <span class="nx">Entry</span><span class="p">):</span> <span class="kr">number</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">entry</span><span class="p">.</span><span class="nx">kind</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">lifting</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>

  <span class="kd">const</span> <span class="p">{</span> <span class="nx">reps</span> <span class="o">=</span> <span class="mi">0</span><span class="p">,</span> <span class="nx">weight</span> <span class="o">=</span> <span class="mi">0</span><span class="p">,</span> <span class="nx">rpe</span> <span class="o">=</span> <span class="mi">5</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">entry</span><span class="p">.</span><span class="nx">lifting</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">volume</span> <span class="o">=</span> <span class="nx">reps</span> <span class="o">*</span> <span class="nx">weight</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">intensityFactor</span> <span class="o">=</span> <span class="nx">rpe</span> <span class="o">/</span> <span class="mi">10</span><span class="p">;</span>

  <span class="k">return</span> <span class="nx">volume</span> <span class="o">*</span> <span class="nx">intensityFactor</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>A set of 5 reps at 225 lbs with RPE 9 (brutal) counts for more than 10 reps at 135 lbs with RPE 5 (warm-up). This gives us way better insights into actual training stress.</p>

<h2>
  
  
  The UI Philosophy
</h2>

<p>Here's the entire interaction model for logging a set:</p>

<ol>
<li>
<strong>Tap the weight number</strong> → Quick adjust (+5 lb) or type any value</li>
<li>
<strong>Tap "Log Set"</strong> → Done</li>
</ol>

<p>We use large tap targets (minimum 48px). Everything important is in thumb-reach on a phone held one-handed. The rest timer starts automatically after you log a set. Haptic feedback on every interaction so you get confirmation even with sweaty hands.</p>

<h3>
  
  
  The Stepper Component
</h3>

<p>Building good steppers for weight/rep adjustment was surprisingly tricky:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="p">&lt;</span><span class="nc">SetStepper</span>
  <span class="na">value</span><span class="p">=</span><span class="si">{</span><span class="nx">weight</span><span class="si">}</span>
  <span class="na">onChange</span><span class="p">=</span><span class="si">{</span><span class="nx">setWeight</span><span class="si">}</span>
  <span class="na">step</span><span class="p">=</span><span class="si">{</span><span class="mi">5</span><span class="si">}</span>  <span class="c1">// +/- 5 lbs per tap</span>
  <span class="na">min</span><span class="p">=</span><span class="si">{</span><span class="mi">0</span><span class="si">}</span>
  <span class="na">max</span><span class="p">=</span><span class="si">{</span><span class="mi">1000</span><span class="si">}</span>
  <span class="na">label</span><span class="p">=</span><span class="s">"Weight"</span>
  <span class="na">unit</span><span class="p">=</span><span class="s">"lb"</span>
  <span class="na">enableDirectInput</span>  <span class="c1">// Tap the number to type</span>
<span class="p">/&gt;</span>
</code></pre>

</div>



<p>We're using optimistic updates here too. The value updates in local state immediately, then fires a debounced mutation to Convex to persist it.</p>

<h2>
  
  
  The Stuff That Broke
</h2>

<h3>
  
  
  Problem #1: The Rest Timer Kept Getting Killed
</h3>

<p>Mobile browsers are aggressive about killing background JavaScript. We had users complaining the rest timer would stop if they switched to Spotify between sets.</p>

<p><strong>Solution:</strong> Web Workers + Service Workers + loud-ass notifications.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// rest-timer-worker.ts</span>
<span class="nb">self</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">message</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">e</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">action</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">start</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">interval</span> <span class="o">=</span> <span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">postMessage</span><span class="p">({</span> <span class="na">timeRemaining</span><span class="p">:</span> <span class="nf">getRemainingTime</span><span class="p">()</span> <span class="p">});</span>
    <span class="p">},</span> <span class="mi">1000</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Plus we request notification permissions so we can send an annoying alert when rest time is up. It works. Users hate it. But they also don't miss their rest periods anymore.</p>

<h3>
  
  
  Problem #2: Clerk JWT Validation Failing Randomly
</h3>

<p>Convex needs to validate Clerk JWTs. The issuer domain kept changing between <code>clerk.accounts.dev</code> and <code>accounts.dev</code> for a reason I'm still not clear on.</p>

<p>We set up a <code>.env</code> var for the issuer domain and made it configurable.</p>

<h3>
  
  
  Problem #3: The "My Phone Died Mid-Workout" Problem
</h3>

<p>Users would start a workout, log 10 sets, phone dies, come back and... everything's gone.</p>

<p>We added a recovery mechanism:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// On app load, check for incomplete workout</span>
<span class="kd">const</span> <span class="nx">checkForIncompleteWorkout</span> <span class="o">=</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">activeWorkout</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">getActiveWorkout</span><span class="p">(</span><span class="nx">userId</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">activeWorkout</span> <span class="o">&amp;&amp;</span> <span class="nx">activeWorkout</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">in_progress</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Show "Resume workout?" dialog</span>
    <span class="nf">showResumeDialog</span><span class="p">(</span><span class="nx">activeWorkout</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Now if your phone dies, the workout stays in "in_progress" state. When you open the app again, it asks if you want to resume. Crisis averted.</p>

<h2>
  
  
  What We'd Do Differently
</h2>

<h3>
  
  
  1. User Testing Sooner
</h3>

<p>We built the "perfect" UI in our heads, then watched actual users struggle with it.</p>

<h2>
  
  
  Try It
</h2>

<p>OpenTrainer is in alpha. It's free. No ads. No tracking pixels. Your data exports as JSON whenever you want.</p>

<p><strong><a href="https://opentrainer.app" rel="noopener noreferrer">Try it here</a></strong></p>

<p>Or clone the repo and run it yourself. It's Apache 2.0 licensed. Do whatever you want with it.</p>

<p><strong><a href="https://github.com/house-of-giants/opentrainer" rel="noopener noreferrer">GitHub: house-of-giants/opentrainer</a></strong></p>




<h2>
  
  
  TLDR
</h2>

<p>We built a workout tracker that doesn't suck. Two taps per set. Real-time sync. AI that actually knows your gym has a Smith machine, not a barbell. Next.js + Convex + Clerk + OpenRouter.</p>

<p>The hardest parts weren't the tech. They were:</p>

<ol>
<li>Making everything feel <strong>instant</strong> (optimistic updates everywhere)</li>
<li>Keeping timers alive on mobile (Web Workers + notifications)</li>
<li>Building UI that works with sweaty hands (big tap targets, haptic feedback)</li>
</ol>

<p>If you're building something similar, use Convex. Seriously. It's rad for real-time apps.</p>

