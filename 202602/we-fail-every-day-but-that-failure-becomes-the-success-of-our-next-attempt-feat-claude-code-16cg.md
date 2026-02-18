---
Title: We fail every day, but that failure becomes the success of our next attempt (feat. Claude Code)
Description: 
Author: elbanic
Date: 2026-02-18T21:50:32.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Dev Sentinel: Recording Failure So We Learn From It
</h2>

<blockquote>
<p>"We fail every day, but that failure becomes the success of our next attempt."</p>
</blockquote>

<h3>
  
  
  Failure Repeats
</h3>

<p>Not long ago, a production incident occurred in a service I operate.</p>

<p>An SQS-triggered Lambda handler processed a single event that spawned an excessive number of DynamoDB transactions in parallel. Eventually, the Lambda hit its timeout limit.</p>

<p>It was a reminder that Promise-based parallelism is not always optimal. IO-bound workloads sometimes require concurrency control or even sequential execution. The code appeared logically sound, yet it failed under real-world resource constraints.</p>

<p>I had a similar experience in a toy project.</p>

<p>A memory-based persistence layer worked perfectly in a local environment, but after deploying to a cloud-based Docker setup, consistency tests failed. The root cause was a stateful design in an environment where container instances could change per request.</p>

<p>Both incidents taught more than simple bug fixes.</p>

<blockquote>
<p>Code may be logically correct, but if it ignores the constraints of its execution environment, the system will eventually break.</p>
</blockquote>

<p>These lessons clearly help in future projects.</p>

<h3>
  
  
  Does Every Failure Lead to Success?
</h3>

<p>We fail every day. We encounter bugs, make incorrect assumptions, and struggle with performance issues.</p>

<p>Becoming a senior engineer often means accumulating these experiences until they turn into intuition. Operational incidents, scaling failures, and flawed architectural decisions gradually build a sense that "something feels risky."</p>

<p>However, in reality, not every failure translates into future success. Many lessons are never recorded, never revisited, and similar mistakes reappear in different contexts.</p>

<p>The problem is not failure itself. The problem is that failure disappears.</p>

<h3>
  
  
  Dev Sentinel: Detecting and Preserving Failure
</h3>

<p>Dev Sentinel started from this observation. The core idea is simple: Detect failure cycles, record them, and surface them when a similar situation arises again.</p>

<p>I define failure as a cycle: Attempt → Frustration → (Abandonment | Resolution)</p>

<p>Within this cycle, I designed two concepts.</p>

<h4>
  
  
  1. Experience Capture
</h4>

<p>Detect frustration, then track whether the attempt ended in abandonment or resolution, and store it as experience. This is not just logging. It structures the context: what the problem was, where the difficulty emerged, and how it concluded.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc25vrpc3qfim6au7ofmy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc25vrpc3qfim6au7ofmy.png" alt="Experience Capture process showing how Dev Sentinel detects and records failure cycles" width="800" height="591"></a></p>

<h4>
  
  
  2. Active Recall
</h4>

<p>When similar frustration is detected again, surface relevant past experiences. The goal is not to solve the problem automatically, but to remind the developer: "You have seen something like this before."</p>

<p>To implement this, Dev Sentinel leverages Claude Code's UserPromptSubmit and Stop hooks.</p>

<ul>
<li>During UserPromptSubmit, it detects signals of frustration (phrasing, repeated attempts, context shifts, etc.).</li>
<li>During Stop, it determines whether the failure cycle concluded and stores the structured experience.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F77wmg1domms38yahy38l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F77wmg1domms38yahy38l.png" alt="Active Recall mechanism showing how Dev Sentinel surfaces relevant past experiences when similar patterns are detected" width="800" height="373"></a></p>

<h3>
  
  
  What If Dev Sentinel Had Existed Then?
</h3>

<p>Looking back at the earlier incidents:</p>

<ul>
<li>Lambda timeout caused by excessive parallelism</li>
<li>Stateful design failure in a containerized environment</li>
</ul>

<p>Both were not merely implementation mistakes. They shared a pattern: underestimating environmental constraints.</p>

<p>If Dev Sentinel had existed, then in a later project:</p>

<ul>
<li>When designing high-concurrency processing logic</li>
<li>When introducing in-memory caching casually</li>
</ul>

<p>It could have surfaced past experiences as a reminder.</p>

<p>Not to provide the answer, but to ask: "Have you seen this pattern fail before?"</p>

<h3>
  
  
  Between Organizational Runbooks and Personal Memory
</h3>

<p>Many organizations reduce failure through runbooks, SOPs, wikis, and knowledge bases. These are highly effective. However, organizational knowledge does not always align perfectly with individual context.</p>

<p>Dev Sentinel takes the opposite direction:</p>

<ul>
<li>Capture individual failure experiences first</li>
<li>Accumulate them over time</li>
<li>Return them to the individual when relevant</li>
</ul>

<p>The expectation is that accumulated personal experience leads to better design judgment, and that judgment ultimately influences projects and organizations.</p>

<h3>
  
  
  Closing Thoughts
</h3>

<p>We fail every day. But failure does not automatically become an asset. Failures that are not remembered are repeated.</p>

<p>Dev Sentinel is not a grand AI system. It is a small mechanism to prevent failure from quietly fading away. Tonight feels like a good time to reflect on what today's lesson was.</p>

<p>git repo: <a href="https://github.com/elbanic/dev-sentinel" rel="noopener noreferrer">https://github.com/elbanic/dev-sentinel</a></p>

