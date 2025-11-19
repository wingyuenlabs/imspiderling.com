---
Title: Testing LLM Prompts in Production Pipelines: A Practical Approach
Description: 
Author: Stuart Palmer
Date: 2025-11-19T22:04:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the past few months, I’ve been working on integrating a number of LLM-based features throughout our product; things like content generation, intelligent recommendations, and agentic task flows. As these features matured, one question kept coming up: how do we actually test this?</p>

<p>Traditional unit testing works beautifully for deterministic code. You write a test, assert an expected output, and move on. But LLMs don’t behave that way; run the same prompt twice and you'll get two different responses. So how do you test something that never gives you the same answer?</p>

<h2>
  
  
  Where We Started: Mocking Our Way Forward
</h2>

<p>Our in-house LLM service already had strong test coverage, so at the application layer we treated it like any other dependency: we mocked it. Our unit tests would stub out the LLM responses and focus on testing the code around them: error handling, data transformations, business logic.</p>

<p>This approach <em>did</em> have value. It gave us confidence that our integration code was solid, let us test edge cases without eating model compute time, and kept our test suite fast. For early development, it was exactly what we needed.</p>

<p>But there was an elephant in the room; we weren't testing the prompts themselves. We had no way to catch when a prompt change degraded output quality, shifted tone, or broke RAG accuracy. We were testing everything except the thing that mattered the most.</p>

<h2>
  
  
  What Mocks Can't Catch
</h2>

<p>As our LLM features moved into production, the gaps became obvious. We had no way to catch things like:</p>

<ul>
<li>
<strong>Prompt regressions</strong> (changes that subtly degraded output quality)</li>
<li>
<strong>Behavioural requirements</strong> (whether outputs met specific criteria like tone, structure, or completeness)</li>
<li>
<strong>RAG faithfulness</strong> (whether our Retrieval-Augmented Generation features accurately represented the knowledge they retrieved, or were hallucinating despite having correct context)</li>
</ul>

<p>These weren’t just theoretical risk. In one case, I updated a function that generates performance recommendations for technical specialists. I added some extra metrics; a straightforward improvement, or so I thought. The feature still ran fine, but the tone had shifted. What was previously clear, actionable technical advice became overly dramatic and superficial. </p>

<p>Our mocked tests passed with no issues, but the actual output had regressed in a way that would frustrate users.</p>

<p>We needed a way to test the prompts themselves.</p>

<h2>
  
  
  The Solution: Prompt Integration Testing
</h2>

<p>To fill the gap, we added a new pipeline stage specifically for testing LLM prompts, focusing on integration testing that evaluates <em>real model outputs</em> against defined requirements. We use <a href="https://www.promptfoo.dev/" rel="noopener noreferrer">PromptFoo</a>, a framework for LLM testing, and integrated it with our existing Jest test suite.</p>

<p>Our tests fall into two main categories:</p>

<h3>
  
  
  1. Quality/Subjective
</h3>

<p>These test behavioural characteristics like tone, structure, completeness and safety. We’re not looking for exact textural matches, instead we define expectations such as:</p>

<ul>
<li>"Response is professional and contains no profanity"</li>
<li>"Answer is structured with headings and bullet points"</li>
<li>"Response does not include personal opinions or speculation"</li>
</ul>

<p>These tests catch regressions where functionality still works but quality silently drops.</p>

<h3>
  
  
  2. Faithfulness/Objective
</h3>

<p>For our RAG-based features, we need to verify that outputs faithfully reflect the retrieved context. These tests check whether the model is grounding its output correctly:</p>

<ul>
<li>"If the knowledge base lists opening hours as 11am-3pm for the holiday, does the response reflect that?"</li>
<li>"When the documentation specifies version 2.4.1, does the answer cite the correct version?"</li>
<li>"Does the summary include all critical warnings from the source material?"</li>
</ul>

<p>This catches hallucinations early and ensures users aren’t seeing confidently incorrect information.</p>

<h2>
  
  
  Implementation: Making It Feel Like Normal Testing
</h2>

<p>The key to adoption was making prompt tests feel like writing any other test. We built a Jest wrapper around PromptFoo that lets developers write LLM tests using familiar patterns.</p>

<p>First we create a custom Jest Matcher:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// addPromptMatchers.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">assertions</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">promptFoo</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="p">{</span> <span class="nx">matchesLlmRubric</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">assertions</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">addPromptMatchers</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">expect</span><span class="p">.</span><span class="nf">extend</span><span class="p">({</span>
    <span class="k">async</span> <span class="nf">toSatisfyLlmRule</span><span class="p">(</span>
      <span class="na">actual</span><span class="p">:</span> <span class="kr">string</span><span class="p">[],</span>
      <span class="na">expected</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
      <span class="nx">passRateThreshold</span> <span class="o">=</span> <span class="mf">0.95</span>
    <span class="p">)</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span>
        <span class="nx">actual</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span>
          <span class="k">async </span><span class="p">(</span><span class="nx">llmOutput</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">await</span> <span class="nf">matchesLlmRubric</span><span class="p">(</span><span class="nx">expected</span><span class="p">,</span> <span class="nx">llmOutput</span><span class="p">)</span>
        <span class="p">)</span>
      <span class="p">);</span>

      <span class="kd">const</span> <span class="nx">passCount</span> <span class="o">=</span> <span class="nx">results</span><span class="p">.</span><span class="nf">filter</span><span class="p">(({</span> <span class="nx">pass</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="nx">pass</span><span class="p">).</span><span class="nx">length</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">passRate</span> <span class="o">=</span> <span class="nx">passCount</span> <span class="o">/</span> <span class="nx">actual</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">passRate</span> <span class="o">&gt;=</span> <span class="nx">passRateThreshold</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="p">{</span>
          <span class="na">message</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="dl">'</span><span class="s1">passed</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">pass</span><span class="p">:</span> <span class="kc">true</span>
        <span class="p">};</span>
      <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="k">return</span> <span class="p">{</span>
          <span class="na">message</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span>
            <span class="s2">`Prompt Eval Failed: </span><span class="p">${</span><span class="nx">passRate</span> <span class="o">*</span> <span class="mi">100</span><span class="p">}</span><span class="s2">% pass rate (needed </span><span class="p">${</span>
              <span class="nx">passRateThreshold</span> <span class="o">*</span> <span class="mi">100</span>
            <span class="p">}</span><span class="s2">% to pass test)`</span><span class="p">,</span>
          <span class="na">pass</span><span class="p">:</span> <span class="kc">false</span>
        <span class="p">};</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">});</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Then, we add this to the Jest startup config, and create a separate Jest config for prompt-specific test runs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">//jest.setupAfter.ts</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">addPromptMatchers</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./addPromptMatchers.ts</span><span class="dl">'</span>

<span class="nf">addPromptMatchers</span><span class="p">();</span>

<span class="c1">//jest.promptEval.config.ts</span>
<span class="k">export</span> <span class="k">default</span> <span class="p">{</span>
  <span class="na">setupFilesAfterEnv</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">./jest.setupAfter.ts</span><span class="dl">'</span><span class="p">],</span>
  <span class="c1">// notice the glob pattern matches files with the suffix .test.prompt.ts</span>
  <span class="na">testMatch</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">**/?(*.)+(spec|test).?(prompt).?([mc])[jt]s?(x)</span><span class="dl">'</span><span class="p">]</span> 
  <span class="c1">// Other Jest config</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Then when you plumb it into your pipeline, remember to invoke an additional Jest stage with the additional config file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// package.json</span>
<span class="p">{</span>
  <span class="dl">"</span><span class="s2">scripts</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
    <span class="dl">"</span><span class="s2">tests:prompt</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">jest --config=jest.promptEval.config.ts</span><span class="dl">"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>To generate tests which utilise this and run within the dedicated pipeline, we use a modified file naming convention (<code>.test.prompt.ts</code> vs <code>.test.ts</code>) to make the distinction clear. Developers who know how to write unit tests can write prompt tests.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// LLM prompt test file</span>
<span class="c1">// myFeature.test.prompt.ts</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">recommendation prompt</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should provide professional, actionable advice</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">promptResults</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span>
      <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">({</span> <span class="na">length</span><span class="p">:</span> <span class="mi">20</span> <span class="p">}).</span><span class="nf">map</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">await</span> <span class="nf">runPrompt</span><span class="p">(</span><span class="nx">promptUnderTest</span><span class="p">);</span>
      <span class="p">})</span>
    <span class="p">);</span>

    <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">promptResults</span><span class="p">).</span><span class="nf">toSatisfyLlmRule</span><span class="p">(</span>
      <span class="dl">'</span><span class="s1">response is professional and contains clear, actionable recommendations</span><span class="dl">'</span>
    <span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>

</code></pre>

</div>



<h2>
  
  
  Handling Non-Determinism
</h2>

<p>The biggest challenge with LLM testing is non-determinism. Run the same prompt twice and you'll get different outputs. Our approach: embrace it.</p>

<p>Instead of expecting identical results, we run each test multiple times and measure the success rate. By default, we require a 95% success rate; if a test passes 19 out of 20 runs, it passes overall. This threshold is configurable per test, which has sparked interesting code-review discussions about what ‘consistent enough’ means for different features.</p>

<p>This is actually a question I'm still thinking about: what success rate thresholds make sense for LLM testing? Is 95% too strict? Too lenient? Does it depend on the use case? (I suspect it does, but I'm curious what others have landed on.)</p>

<h2>
  
  
  CI/CD Integration
</h2>

<p>We run these tests in a dedicated pipeline stage, separate from our regular unit tests. There’s a few reasons for that:</p>

<ul>
<li>
<strong>They're slow</strong> (Multiple runs of real LLM calls take time)</li>
<li>
<strong>They're expensive</strong> (Each test execution hits the LLM multiple times)</li>
<li>
<strong>They need a different rollout approach</strong> (This is new territory for most of the team)</li>
</ul>

<p>That last point is key. Currently, the stage is non-blocking; failures don’t prevent deployment. Instead, we report results to a Slack channel where the team can monitor performance and trends. This gives us space to build confidence in the solution gradually, expand test coverage, refine our success criteria, and get comfortable with non-deterministic testing before making it a hard gate.</p>

<h2>
  
  
  Trade-offs and Learnings
</h2>

<p>There are definite costs and consequences to this approach:</p>

<p><strong>Time</strong>: This pipeline stage is slower than our standard unit tests. Where regular tests complete in seconds, these can take minutes depending on the number of prompts and configured run counts.</p>

<p><strong>Cost</strong>: Running multiple iterations of real LLM calls adds up - token usage isn’t free. </p>

<p><strong>Wider Scope</strong>: These tests blur the line between technical and product decisions. Unlike traditional tests that verify logic, prompt tests encode expectations about quality, tone, and behaviour. This means their criteria probably shouldn't be defined by engineering alone - Product owners have a role to play. It reminds me of behaviour-driven development (BDD), where acceptance criteria come from the business rather than the code. "The response should be professional and helpful" is less a technical specification and more a product requirement that happens to be testable with LLMs.</p>

<p><strong>Flakiness</strong>: Even with high success rate thresholds, occasional failures still happen. We're still developing our understanding of when a failure indicates a real problem versus normal variance.</p>

<p>Despite these trade-offs, the benefits are clear. We've caught issues that would have shipped to production, and developers have more confidence making prompt changes knowing there's a safety net.</p>

<h2>
  
  
  Takeaways for Others
</h2>

<p>If you're considering adding prompt testing, here's what we’ve learned so far:</p>

<p><strong>Start small</strong>: Pick one or two critical prompts to test first. Don't try to achieve 100% coverage immediately. Focus on prompts where failures would have the highest impact.</p>

<p><strong>Involve Product Owners</strong>: Prompt test criteria are product decisions as much as technical ones. Bring Product Owners into conversations about how we should be validating the prompt output from the user perspective. They'll help you define rubrics that reflect real user needs rather than engineering assumptions.</p>

<p><strong>Make it easy to write tests</strong>: Whatever your implementation, minimise friction for developers. If writing a prompt test is significantly harder than writing a unit test, adoption will suffer.</p>

<p><strong>Accept imperfection</strong>: Probabilistic success rates feel odd at first, but they’re a more honest model of how LLMs behave.</p>

<p><strong>Balance cost versus confidence</strong>: You don't need to test every prompt on every commit. Be strategic about when tests run and how many iterations you perform.</p>

<h2>
  
  
  Conclusion
</h2>

<p>LLMs are now core to many B2B products, but traditional testing doesn’t fit their probabilistic nature. Mocking still has its place for testing integration code, but it can't tell you if your prompts truly deliver the intended outcome.</p>

<p>Prompt integration testing isn't perfect - it's slower, more expensive, and fuzzier than regular testing. But it fills a critical gap. It can catch regressions before they reach production, give developers confidence to refactor prompts, and forces us to articulate what "good" actually means for our AI features.</p>

<p>However you’re using AI, one principle holds true: if the product’s value depends on LLM outputs, you need to test those outputs directly.</p>

