---
Title: From Prototype to Production: How Promptfoo and Vitest Made podcast-it Reliable
Description: 
Author: logarithmicspirals
Date: 2025-09-13T21:56:43.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>In my previous article, <a href="https://logarithmicspirals.com/blog/from-idea-to-audio-building-podcast-it-cloudflare-worker/" rel="noopener noreferrer">From Idea to Audio: Building the podcast-it Cloudflare Worker</a>, I detailed how I had created a simple Cloudflare Worker which could create podcast scripts and audio from a source blog post.</p>

<p>While I was not sure if I would continue working on the project, I made some breakthroughs in my understanding of how to build a serverless app powered by an LLM. One of the key pieces I was missing was LLM evaluations (evals). With evals, I was able to massively improve my development speed and feel much more confident in the progress of the project. Being confident in the quality of the podcasts gave me the energy I needed to keep going with the project.</p>

<h2>
  
  
  Why LLM Evals Matter
</h2>

<p>When you’re building traditional software, testing usually means making sure your code behaves correctly: does this function return the right value, does this API respond in time, does this component render as expected? But with generative AI apps, there’s a new dimension — the quality of the model’s output itself. Even if your API endpoints all work perfectly, the app can still “fail” if the generated script sounds robotic, skips key information, or produces inconsistent episode structures.</p>

<p>That’s where LLM evals come in. Instead of only testing the code around the model, evals test the behavior of the model inside your app. They let you measure whether the text produced is useful, accurate, and consistent across many runs — which is critical if you want to move from a fun prototype to something people can actually trust.</p>

<p>One way I brought the power of evals into this project is through <a href="https://www.promptfoo.dev/docs/intro/" rel="noopener noreferrer">promptfoo</a>. With promptfoo, I was able to quickly get started creating eval tests in my TypeScript environment. A test might check, for example:</p>

<ul>
<li>Does every generated script include an introduction and conclusion?</li>
<li>Does the host’s name appear in the dialogue?</li>
<li>Is the transcript free of obvious hallucinations (like citing a source that isn’t in the blog post)?</li>
</ul>

<p>To get started with promptfoo, I installed it like so:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> promptfoo
</code></pre>

</div>



<p>After setting up and configuring promptfoo, I was able to create a script called <code>"evals"</code> in my <code>package.json</code> and run it as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run evals
</code></pre>

</div>



<p>The evals script looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm run export-schemas <span class="o">&amp;&amp;</span> npx dotenvx run <span class="nt">-f</span> .dev.vars <span class="nt">--</span> sh <span class="nt">-c</span> <span class="s1">'cd evals &amp;&amp; PROMPTFOO_DISABLE_TELEMETRY=1 promptfoo eval'</span>
</code></pre>

</div>



<p>Here's what it does:</p>

<ol>
<li>A custom script is used to convert TypeScript types from my source code into JSON schema files within the <code>evals/</code> directory.</li>
<li>The <a href="https://dotenvx.com" rel="noopener noreferrer">dotenvx</a> package is used to read the environment variables of <code>.dev.vars</code> into memory so that promptfoo has access to them. Promptfoo expects a <code>.env</code> file, but the Worker expects a <code>.dev.vars</code> file locally. This step bridges that gap.</li>
<li>The script switches to the <code>evals/</code> directory and then runs promptfoo using the config file in that directory.</li>
</ol>

<p>Currently, I have evals which confirm the following:</p>

<ol>
<li>The expected hosts are used, and no other hallucinated participants are present.</li>
<li>The show title is mentioned in the script.</li>
<li>The correct JSON schema is used.</li>
<li>LLM graders are giving passing scores to the scripts.</li>
<li>The content of the script is relevant to the post.</li>
<li>The script passes OpenAI's moderation criteria.</li>
</ol>

<h2>
  
  
  The Role of Integration Tests in Gen AI Apps
</h2>

<p>While the LLM evals tell me if the content is good, integration tests tell me if the system is behaving correctly. Most of the issues I ran into weren’t “the model produced a weird sentence.” They were the boring but important things: a request that didn’t persist correctly, an episode that never left processing, or an endpoint that returned the wrong status code.</p>

<p>To catch these, I wrote integration tests with <a href="https://vitest.dev" rel="noopener noreferrer">Vitest</a>. Instead of just mocking everything, the tests spin up a temporary database and exercise the actual REST endpoints. That means I can simulate creating an episode, check that it moves through the lifecycle, and verify that the audio and metadata end up where they belong.</p>

<p>To install Vitest in my project, I ran the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">--save-dev</span> vitest
</code></pre>

</div>



<p>Since I was working with the Cloudflare Workers framework, I was able to configure my tests to have a temporary D1 database and R2 bucket. The same migration scripts used to run the app are applied before the tests are run. For context, Cloudflare Workers have a feature where developers can version their database changes using migration scripts. Having the test database configuration match the real database configuration is very easy with this feature. Here's what my <code>vitest.config.ts</code> file looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="nx">path</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">node:path</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span>
    <span class="nx">defineWorkersProject</span><span class="p">,</span>
    <span class="nx">readD1Migrations</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@cloudflare/vitest-pool-workers/config</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="nf">defineWorkersProject</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">migrationsPath</span> <span class="o">=</span> <span class="nx">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nx">__dirname</span><span class="p">,</span> <span class="dl">"</span><span class="s2">migrations</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">migrations</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">readD1Migrations</span><span class="p">(</span><span class="nx">migrationsPath</span><span class="p">);</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="na">test</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">setupFiles</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">./tests/apply-migrations.ts</span><span class="dl">"</span><span class="p">],</span>
            <span class="na">poolOptions</span><span class="p">:</span> <span class="p">{</span>
                <span class="na">workers</span><span class="p">:</span> <span class="p">{</span>
                    <span class="na">miniflare</span><span class="p">:</span> <span class="p">{</span>
                        <span class="na">compatibilityDate</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2025-08-03</span><span class="dl">"</span><span class="p">,</span>
                        <span class="na">d1Databases</span><span class="p">:</span> <span class="p">{</span> <span class="na">DB</span><span class="p">:</span> <span class="dl">"</span><span class="s2">:memory:</span><span class="dl">"</span> <span class="p">},</span>
                        <span class="na">d1Persist</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
                        <span class="na">bindings</span><span class="p">:</span> <span class="p">{</span> <span class="na">TEST_MIGRATIONS</span><span class="p">:</span> <span class="nx">migrations</span> <span class="p">},</span>
                        <span class="na">r2Buckets</span><span class="p">:</span> <span class="p">[</span><span class="dl">"</span><span class="s2">podcasts</span><span class="dl">"</span><span class="p">]</span>
                    <span class="p">},</span>
                <span class="p">},</span>
            <span class="p">},</span>
        <span class="p">},</span>
    <span class="p">};</span>
<span class="p">});</span>
</code></pre>

</div>



<p>With these integration tests, I am able to confirm things like:</p>

<ul>
<li>Episode creation and deletion work as expected.</li>
<li>The right status code is returned when a request to make the same episode twice is made.</li>
<li>Audio files are uploaded to the right location.</li>
</ul>

<p>Integration tests are also useful for making sure changes made to the codebase don't break anything. As such, I also added <a href="https://github.com/typicode/husky" rel="noopener noreferrer">Husky</a> to the project so that I could run the tests every time I tried to push code from my laptop to GitHub.</p>

<p>My Husky script, <code>.husky/pre-push</code>, is pretty simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/sh</span>

npm run prepush
</code></pre>

</div>



<p>The actual meat of this prepush hook is the <code>"prepush"</code> script in <code>package.json</code> which looks like <code>npm run typecheck &amp;&amp; npm test &amp;&amp; npm run evals</code>. In English, this script does the following:</p>

<ol>
<li>Runs type checks on the project to look for developer errors.</li>
<li>Runs all available tests with Vitest.</li>
<li>Runs the evals script to confirm the models still behave as expected.</li>
</ol>

<h2>
  
  
  Iterating Faster with Promptfoo + Vitest
</h2>

<p>The biggest unlock came when I started running evals and integration tests together in my development loop. Promptfoo gave me immediate feedback on content quality — whether scripts still had an intro and conclusion, whether the host’s name appeared consistently, and whether hallucinations crept in. At the same time, Vitest confirmed that the APIs, database migrations, and storage flows all worked as expected.</p>

<p>That combination gave me confidence to move faster. I could tweak prompts, refactor code, or adjust infrastructure without constantly worrying about breaking something. If an eval failed, I knew it was a content issue. If a test failed, I knew it was a system issue. Together, they formed a feedback loop that made iteration smoother, reduced surprises, and made shipping updates feel much less risky.</p>

<h2>
  
  
  Lessons Learned
</h2>

<ul>
<li>
<strong>Schemas eliminate drift.</strong> Converting TypeScript types to JSON schemas kept evals aligned with the source code and cut down on manual setup.</li>
<li>
<strong>Evals catch subtle regressions.</strong> Automated checks surfaced issues—like the host’s name quietly disappearing—that slipped past manual review.</li>
<li>
<strong>Tests and evals reinforce each other.</strong> Tests safeguard the system (routes, state, storage) while evals safeguard the output (tone, structure, relevance). Together, they build confidence.</li>
</ul>

<h2>
  
  
  What’s Next
</h2>

<p>Here's what is coming up for podcast-it:</p>

<ol>
<li>Integration into my blog. I will be generating podcast episodes for all my existing blog posts.</li>
<li>Enriching scripts with relevant content from the web.</li>
<li>Adding script editing through a micro frontend.</li>
<li>Investigating model fine tuning to see if longer scripts can be generated without losing quality.</li>
</ol>

<h2>
  
  
  Conclusion
</h2>

<p>Moving podcast-it from “it works” to “it’s reliable” wasn’t about shipping new features—it was about building confidence. Promptfoo gave me a way to measure output quality, integration tests kept the system stable, and unit tests still catch the small stuff early. Together, they create a safety net that makes iterating on a generative AI app sustainable. If you’re building with LLMs, set up that loop early—you’ll avoid a lot of pain later.</p>

