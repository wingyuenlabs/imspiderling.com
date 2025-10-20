---
Title: Proving ROI with Data-Driven AI Agent Experiments
Description: 
Author: Scarlett Attensil
Date: 2025-10-20T21:33:33.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Published October 9th, 2025</em></p>


<p>by Scarlett Attensil</p>

<h2>
  
  
  What You'll Learn in 5 Minutes (or Build in 30)
</h2>

<blockquote>
<p><strong>Key Findings from Our Experiments:</strong></p>

<ul>
<li>
<strong>Unexpected discovery</strong>: Free Mistral model is not only \$0 but also significantly faster than Claude Haiku</li>
<li>
<strong>Cost paradox revealed</strong>: "Free" security agent increased total system costs by forcing downstream agents to compensate</li>
<li>
<strong>Premium model failure</strong>: Claude Opus 4 performed 64% worse than GPT-4o despite costing 33% more</li>
<li>
<strong>Sample size reality</strong>: High-variance metrics (cost, feedback) require 5-10x more data than low-variance ones (latency)</li>
</ul>
</blockquote>

<h2>
  
  
  The Problem
</h2>

<p>Your CEO asks: <strong>"Is the new expensive AI model worth it?"</strong></p>

<p>Your finance team wonders: <strong>"Does the enhanced privacy justify the cost?"</strong></p>

<p>Without experiments, you're guessing. This tutorial shows you how to <strong>measure the truth</strong> and sometimes discover unanticipated gains.</p>

<h2>
  
  
  The Solution: Real Experiments, Real Answers
</h2>

<p>In 30 minutes, you'll run actual A/B tests that reveal:</p>

<p><strong>Will aggressive PII redaction hurt user satisfaction?</strong></p>

<p><strong>Is Claude Opus 4 worth 33% more than GPT-4o?</strong></p>

<p><em>Part 3 of 3 of the series: **Chaos to Clarity: Defensible AI Systems That Deliver on Your Goals</em>**</p>

<h2>
  
  
  Quick Start Options
</h2>

<h3>
  
  
  <strong>Option 1: Just Want the Concepts?</strong> (5 min read)
</h3>

<p>Skip to Understanding the Experiments to learn the methodology without running code.</p>

<h3>
  
  
  <strong>Option 2: Full Hands-On Tutorial</strong> (30 min)
</h3>

<p>Follow the complete guide to run your own experiments.</p>

<p>
  "<strong>Prerequisites for Hands-On Tutorial</strong>"
  <p><strong>Required from Previous Parts:</strong></p>

<ul>
<li>Active LaunchDarkly project completed from <a href="https://dev.to/tutorials/agents-langgraph">Part 1</a> &amp; <a href="https://dev.to/tutorials/multi-agent-mcp-targeting">Part 2</a>
</li>
<li>API keys: Anthropic, OpenAI, LaunchDarkly. <a href="http://app.launchdarkly.com/signup" rel="noopener noreferrer">Sign up for a free account here</a> and then <a href="https://dev.to/home/account/api-create">follow these instructions to get your API access token</a>.</li>
</ul>

<p><strong>Investment:</strong></p>

<ul>
<li>Time: ~30 minutes</li>
<li>Cost: \$25-35 default (\$5-10 with <code>--queries 50</code>)</li>
</ul>

<p>
  "<strong>Reduce Experiment Costs</strong>"
  </p>
<p>The default walk-through uses Claude Opus 4 (premium model) for testing. To reduce costs while still learning the experimentation patterns, you can modify <code>bootstrap/tutorial_3_experiment_variations.py</code> in your cloned repository to test with the free Mistral model instead:</p>

<p>In the <code>create_premium_model_variations</code> function, change:<br>
</p>

<pre class="highlight python"><code><span class="c1"># Original (expensive):
</span><span class="sh">"</span><span class="s">model</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">claude-opus-4-20250514</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">provider</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">anthropic</span><span class="sh">"</span>
<span class="p">}</span>

<span class="c1"># Change to (free Mistral):
</span><span class="sh">"</span><span class="s">model</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">mistral-small-latest</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">provider</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">mistral</span><span class="sh">"</span>
<span class="p">}</span>
</code></pre>



<p>This reduces the experiment cost by about $20 (you'll still have costs from the control group using GPT-4o and other agents in the system).</p>



</p>





<h2>
  
  
  How the Experiments Work
</h2>

<p><strong>The Setup</strong>: Your AI system will automatically test variations on simulated users, collecting real performance data that flows directly to LaunchDarkly for statistical analysis.</p>

<p><strong>The Process</strong>:</p>

<ol>
<li>
<strong>Traffic simulation</strong> generates queries from your actual knowledge base</li>
<li>
<strong>Each user</strong> gets randomly assigned to experiment variations</li>
<li>
<strong>AI responses</strong> are evaluated for quality and tracked for cost/speed</li>
<li>
<strong>LaunchDarkly</strong> calculates statistical significance automatically</li>
</ol>

<p><strong>Note</strong>: The two experiments can run independently. Each user can participate in both, but the results are analyzed separately.</p>

<p><strong>Experiment Methodology</strong>: Our supervisor agent routes PII queries to the security agent (then to support), while clean queries go directly to support. LaunchDarkly tracks metrics <strong>at the user level across all agents</strong>, revealing system-wide effects. </p>

<h2>
  
  
  Understanding Your Two Experiments
</h2>

<h3>
  
  
  <strong>Experiment 1: Security Agent Analysis</strong>
</h3>

<p><strong>Question</strong>: Does Strict Security (free Mistral model with aggressive PII redaction) improve performance without harming user experience or significantly increasing system costs?</p>

<p><strong>Variations</strong> (50% each):</p>

<ul>
<li>
<strong>Control</strong>: Basic Security (Claude Haiku, moderate PII redaction)</li>
<li>
<strong>Treatment</strong>: Strict Security (Mistral free, aggressive PII redaction)</li>
</ul>

<p><strong>Success Criteria</strong>:</p>

<ol>
<li>Positive feedback rate: stable or improving (not significantly worse)</li>
<li>Cost increase: ≤15% with ≥75% confidence</li>
<li>Latency increase: ≤3 seconds (don't significantly slow down)</li>
<li>Enhanced privacy protection delivered</li>
</ol>

<h3>
  
  
  <strong>Experiment 2: Premium Model Value Analysis</strong>
</h3>

<p><strong>Question</strong>: Does Claude Opus 4 justify its premium cost vs GPT-4o?</p>

<p><strong>Variations</strong> (50% each):</p>

<ul>
<li>
<strong>Control</strong>: GPT-4o with full tools (current version)</li>
<li>
<strong>Treatment</strong>: Claude Opus 4 with identical tools</li>
</ul>

<p><strong>Success Criteria (must meet 90% threshold)</strong>:</p>

<ul>
<li>≥15% positive feedback rate improvement by Claude Opus 4</li>
<li>Cost-value ratio ≥ 0.25 (positive feedback rate gain % ÷ cost increase %)</li>
</ul>

<h2>
  
  
  Setting Up Metrics and Experiments
</h2>

<h3>
  
  
  <strong>Step 1: Configure Metrics (5 minutes)</strong>
</h3>

<h4>
  
  
  <strong>Quick Metric Setup</strong>
</h4>

<p>Navigate to <strong>Metrics</strong> in LaunchDarkly and <a href="https://dev.to/home/metrics/create-metrics">create three custom metrics</a>:</p>

<div class="table-wrapper-paragraph"><table>
  <tr>
    <th>Metric Name</th>
    <th>Event Key</th>
    <th>Type</th>
    <th>What It Measures</th>
  </tr>
  <tr>
    <td><strong>p95_total_user_latency</strong></td>
    <td>$ld:ai:duration:total</td>
    <td>P95</td>
    <td>Response speed</td>
  </tr>
  <tr>
    <td><strong>average_total_user_tokens</strong></td>
    <td>$ld:ai:tokens:total</td>
    <td>Average</td>
    <td>Token usage</td>
  </tr>
  <tr>
    <td><strong>ai_cost_per_request</strong></td>
    <td>ai_cost_per_request</td>
    <td>Average</td>
    <td>Dollar cost</td>
  </tr>
  <tr>
    <td><strong>Positive Feedback</strong></td>
    <td>Built-in</td>
    <td>Rate</td>
    <td>Positive feedback rate</td>
  </tr>
  <tr>
    <td><strong>Negative Feedback</strong></td>
    <td>Built-in</td>
    <td>Rate</td>
    <td>User complaints</td>
  </tr>
</table></div>

<p>
  "<strong>See detailed setup for P95 Latency</strong>"
  <ol>
<li>Event key: <code>$ld:ai:duration:total</code>
</li>
<li>Type: Value/Size → Numeric, Aggregation: Sum</li>
<li>Definition: P95, value, user, sum, "lower is better"</li>
<li>Unit: <code>ms</code>, Name: <code>p95_total_user_latency</code>
</li>
</ol>

<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzu9wt47htwm9mjheg99e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzu9wt47htwm9mjheg99e.png" alt="P95 Setup" width="800" height="1213"></a><br>
</p>



<br>
</p>

<p>
  "<strong>View other metric configurations</strong>"
  <ul>
<li>
<strong>Tokens</strong>: Event key <code>$ld:ai:tokens:total</code>, Name: <code>average_total_user_tokens</code>, Average aggregation</li>
<li>
<strong>Cost</strong>: Event key <code>ai_cost_per_request</code>, Name: <code>ai_cost_per_request</code>, Average in dollars</li>
</ul>

<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwzjsrcfxvy53api9modn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwzjsrcfxvy53api9modn.png" alt="Tokens" width="800" height="1234"></a><br>
</p>

<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvft0jb7dzwxyvyuv9841.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvft0jb7dzwxyvyuv9841.png" alt="Cost" width="800" height="1226"></a><br>
</p>



<br>
</p>

<p>The cost tracking is implemented in <code>utils/cost_calculator.py</code>, which calculates actual dollar costs using the formula <code>(input_tokens × input_price + output_tokens × output_price) / 1M</code>. The system has pre-configured pricing for each model (as of October 2025): GPT-4o at \$2.50/\$10 per million tokens, Claude Opus 4 at \$15/\$75, and Claude Sonnet at \$3/\$15. When a request completes, the cost is immediately calculated and sent to LaunchDarkly as a custom event, enabling direct cost-per-user analysis in your experiments.</p>

<h3>
  
  
  <strong>Step 2: Create Experiment Variations</strong>
</h3>

<p>Create the experiment variations using the bootstrap script:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>uv run python bootstrap/tutorial_3_experiment_variations.py
</code></pre>

</div>



<p>This creates the <code>claude-opus-treatment</code> variation for the Premium Model Value experiment. To verify the script worked correctly, navigate to your <strong>support-model-config</strong> feature flag in LaunchDarkly - you should now see the <code>claude-opus-treatment</code> variation alongside your existing variations. The Security Agent Analysis experiment will use your existing baseline and enhanced variations. Both experiments use the existing <code>other-paid</code> configuration as their control group.</p>

<h3>
  
  
  <strong>Step 3: Configure Security Agent Experiment</strong>
</h3>

<p>
  "Click for details"
  <p>Navigate to <strong>AI Configs → security-agent</strong>. In the right navigation menu, click the plus (+) sign next to <strong>Experiments</strong> to create a new experiment</p>
<h4>
  
  
  <strong>Experiment Design</strong>
</h4>

<p><strong>Experiment type:</strong></p>

<ul>
<li>Keep <code>Feature change</code> selected (default)</li>
</ul>

<p><strong>Name:</strong> <code>Security Level</code></p>
<h4>
  
  
  <strong>Hypothesis and Metrics</strong>
</h4>

<p><strong>Hypothesis:</strong> <code>Enhanced security improves safety compliance without significantly harming positive feedback rates</code></p>

<p><strong>Randomize by:</strong> <code>user</code></p>

<p><strong>Metrics:</strong> Click "Select metrics or metric groups" and add:</p>

<ol>
<li>
<code>Positive feedback rate</code> → Select first to set as <strong>Primary</strong>
</li>
<li><code>Negative feedback rate</code></li>
<li><code>p95_total_user_latency</code></li>
<li><code>ai_cost_per_request</code></li>
</ol>
<h4>
  
  
  <strong>Audience Targeting</strong>
</h4>

<p><strong>Flag or AI Config</strong></p>

<ul>
<li>Click the dropdown and select <strong>security-agent</strong>
</li>
</ul>

<p><strong>Targeting rule:</strong></p>

<ul>
<li>Click the dropdown and select <strong>Rule 4</strong>
</li>
<li>This will configure: <code>If Context</code> → <code>is in Segment</code> → <code>Other Paid</code>
</li>
</ul>
<h4>
  
  
  <strong>Audience Allocation</strong>
</h4>

<p><strong>Variations served outside of this experiment:</strong></p>

<ul>
<li><code>Basic Security</code></li>
</ul>

<p><strong>Sample size:</strong> Set to <code>100%</code> of users in this experiment</p>

<p><strong>Variations split:</strong> Click "Edit" and configure:</p>

<p>Note: Before setting these percentages, scroll down to the <strong>Control</strong> field below and set <code>Basic Security</code> as the control variation first, otherwise you won't be able to allocate 50% traffic to it.</p>

<ul>
<li>
<code>pii-detector</code>: <code>0%</code>
</li>
<li>
<code>Basic Security</code>: <code>50%</code>
</li>
<li>
<code>Strict Security</code>: <code>50%</code>
</li>
</ul>

<p><strong>Control:</strong></p>

<ul>
<li><code>Basic Security</code></li>
</ul>
<h4>
  
  
  <strong>Statistical Approach and Success Criteria</strong>
</h4>

<p><strong>Statistical approach:</strong> <code>Bayesian</code><br>
<strong>Threshold:</strong> <code>90%</code></p>

<p>Click <strong>"Save"</strong><br>
Click <strong>"Start experiment"</strong> to launch.</p>

<p><strong>Note</strong>: You may see a "Health warning" indicator after starting the experiment. This is normal and expected when no variations have been exposed yet. The warning will clear once your experiment starts receiving traffic and data begins flowing.</p>



</p>



<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F731ozqk2bd4bwlki0qo2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F731ozqk2bd4bwlki0qo2.png" alt="Security Agent Experiment Configuration" width="800" height="429"></a><br>
</p>



<h3>
  
  
  <strong>Step 4: Configure Premium Model Experiment</strong>
</h3>

<p>
  "Click for details"
  <p>Navigate to <strong>AI Configs → support-agent</strong>. In the right navigation menu, click the plus (+) sign next to <strong>Experiments</strong> to create a new experiment</p>
<h4>
  
  
  <strong>Experiment Design</strong>
</h4>

<p><strong>Experiment type:</strong></p>

<ul>
<li>Keep <code>Feature change</code> selected (default)</li>
</ul>

<p><strong>Name:</strong> <code>Premium Model Value Analysis</code></p>
<h4>
  
  
  <strong>Hypothesis and Metrics</strong>
</h4>

<p><strong>Hypothesis:</strong> <code>Claude Opus 4 justifies premium cost with superior positive feedback rate</code></p>

<p><strong>Randomize by:</strong> <code>user</code></p>

<p><strong>Metrics:</strong> Click "Select metrics or metric groups" and add:</p>

<ol>
<li>
<code>Positive feedback rate</code> → Select first to set as <strong>Primary</strong>
</li>
<li><code>Negative feedback rate</code></li>
<li><code>p95_total_user_latency</code></li>
<li><code>average_total_user_tokens</code></li>
<li><code>ai_cost_per_request</code></li>
</ol>
<h4>
  
  
  <strong>Audience Targeting</strong>
</h4>

<p><strong>Flag or AI Config</strong></p>

<ul>
<li>Click the dropdown and select <strong>support-agent</strong>
</li>
</ul>

<p><strong>Targeting rule:</strong></p>

<ul>
<li>Click the dropdown and select <strong>Rule 4</strong>
</li>
<li>This will configure: <code>If Context</code> → <code>is in Segment</code> → <code>Other Paid</code>
</li>
</ul>
<h4>
  
  
  <strong>Audience Allocation</strong>
</h4>

<p><strong>Variations served outside of this experiment:</strong></p>

<ul>
<li><code>other-paid</code></li>
</ul>

<p><strong>Sample size:</strong> Set to <code>100%</code> of users in this experiment</p>

<p><strong>Variations split:</strong> Click "Edit" and configure:</p>

<p>Note: Before setting these percentages, scroll down to the <strong>Control</strong> field below and set <code>other-paid</code> as the control variation first, otherwise you won't be able to allocate 50% traffic to it.</p>

<ul>
<li>
<code>rag-search-enhanced</code>: <code>0%</code>
</li>
<li>
<code>eu-free</code>: <code>0%</code>
</li>
<li>
<code>eu-paid</code>: <code>0%</code>
</li>
<li>
<code>other-free</code>: <code>0%</code>
</li>
<li>
<code>other-paid</code>: <code>50%</code>
</li>
<li>
<code>international-standard</code>: <code>0%</code>
</li>
<li>
<code>claude-opus-treatment</code>: <code>50%</code>
</li>
</ul>

<p><strong>Control:</strong></p>

<ul>
<li><code>other-paid</code></li>
</ul>
<h4>
  
  
  <strong>Statistical Approach and Success Criteria</strong>
</h4>

<p><strong>Statistical approach:</strong> <code>Bayesian</code><br>
<strong>Threshold:</strong> <code>90%</code></p>

<p>Click <strong>"Save"</strong><br>
Click <strong>"Start experiment"</strong> to launch.</p>

<p><strong>Note</strong>: You may see a "Health warning" indicator after starting the experiment. This is normal and expected when no variations have been exposed yet. The warning will clear once your experiment starts receiving traffic and data begins flowing.</p>



</p>



<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc6nwj3pddvthcvu3ak3v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc6nwj3pddvthcvu3ak3v.png" alt="Premium Model Value Analysis Experiment Configuration" width="800" height="432"></a><br>
</p>



<h2>
  
  
  Understanding Your Experimental Design
</h2>

<p><strong>If Two Independent Experiments Running Concurrently:</strong></p>

<p>Since these are the <strong>same users</strong>, each user experiences:</p>

<ul>
<li>One security variation (<code>Basic Security</code> or <code>Strict Security</code>)</li>
<li>One model variation (<code>Claude Opus 4 Treatment</code> OR <code>Other Paid (GPT-4o)</code>)</li>
</ul>

<p>Random assignment ensures balance: ~50 users get each combination naturally.</p>

<h2>
  
  
  Generating Experiment Data
</h2>

<h3>
  
  
  <strong>Step 5: Run Traffic Generator</strong>
</h3>

<p>Start your backend and generate realistic experiment data. Choose between sequential or concurrent traffic generation:</p>

<h4>
  
  
  <strong>Concurrent Traffic Generator (Recommended for large datasets)</strong>
</h4>

<p>For faster experiment data generation with parallel requests:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start backend API</span>
uv run uvicorn api.main:app <span class="nt">--reload</span> <span class="nt">--port</span> 8000

<span class="c"># Generate experiment data with 10 concurrent requests (separate terminal)</span>
uv run python <span class="nt">-u</span> tools/concurrent_traffic_generator.py <span class="nt">--queries</span> 200 <span class="nt">--concurrency</span> 10
</code></pre>

</div>



<p><strong>Configuration</strong>:</p>

<ul>
<li>
<strong>200 queries</strong> by default (edit script to adjust)</li>
<li>
<strong>10 concurrent requests</strong> running in parallel</li>
<li>
<strong>2000-second timeout</strong> (33 minutes) per request to handle MCP tool rate limits</li>
</ul>

<p>Note: runtime depends largely on if you retain MCP tool enablement as these take much longer to complete.</p>

<p>
  "For smaller test runs or debugging"
  <h4>
  
  
  <strong>Sequential Traffic Generator (Simple, one-at-a-time)</strong>
</h4>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start backend API</span>
uv run uvicorn api.main:app <span class="nt">--reload</span> <span class="nt">--port</span> 8000

<span class="c"># Generate experiment data sequentially (separate terminal)</span>
uv run python tools/traffic_generator.py <span class="nt">--queries</span> 50 <span class="nt">--delay</span> 2
</code></pre>

</div>


<p><strong>What Happens During Simulation:</strong></p>

<ol>
<li><p><strong>Knowledge extraction</strong><br>
Claude analyzes your docs and identifies 20+ realistic topics</p></li>
<li><p><strong>Query generation</strong><br>
Each test randomly selects from these topics for diversity</p></li>
<li><p><strong>AI-powered evaluation</strong><br>
Claude judges responses as thumbs_up/thumbs_down/neutral</p></li>
<li><p><strong>Automatic tracking</strong><br>
All metrics flow to LaunchDarkly in real-time</p></li>
</ol>



</p>

<p><strong>Generation Output</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>📚 Analyzing knowledge base...
✅ Generated 23 topics

⚡ Sending 200 requests with 10 concurrent workers...

✅ [1/200] Success (23.4s) - other_paid: What is reinforcement learning?...
✅ [2/200] Success (45.2s) - other_paid: How does Q-learning work?...
⏱️  [15/200] Timeout (&gt;2000s) - other_paid: Complex research query...
                              ↑ This is normal - MCP rate limits
✅ [200/200] Success (387.1s) - other_paid: Explain temporal difference...

======================================================================
✅ COMPLETE
======================================================================
Total time: 45.3 minutes (2718s)
Successful: 195/200 (97.5%)
Failed: 5/200 (2.5%)
Average: 13.6s per query (with concurrency)
</code></pre>

</div>



<p><strong>Performance Notes</strong>:</p>

<ul>
<li>Most queries complete in 10-60 seconds</li>
<li>Queries using <code>semantic_scholar</code> MCP tool may take 5-20 minutes due to API rate limits</li>
<li>Concurrent execution handles slow requests gracefully by continuing with others</li>
<li>Failed/timeout requests (less than 5% typically) don't affect experiment validity</li>
</ul>

<p><strong>Monitor Results</strong>: Refresh your LaunchDarkly experiment "Results" tabs to see data flowing in. Cost metrics appear as custom events alongside feedback and token metrics.</p>

<h2>
  
  
  Interpreting Your Results (After Data Collection)
</h2>

<p>Once your experiments have collected data from ~100 users per variation, you'll see results in the LaunchDarkly UI. Here's how to interpret them:</p>

<h3>
  
  
  <strong>1. Security Agent Analysis: Does enhanced security improve safety without significantly impacting positive feedback rates?</strong>
</h3>

<p><strong>Reality Check</strong>: Not all metrics reach significance at the same rate. In our security experiment we ran over 2,000 more users than the model experiment, turning off the MCP tools and using <code>--pii-percentage 100</code> to maximize pii detection:</p>

<ul>
<li>
<strong>Latency</strong>: 87% confidence (nearly significant, clear 36% improvement)</li>
<li>
<strong>Cost</strong>: 21% confidence (high variance, needs 5-10x more data)</li>
<li>
<strong>Feedback</strong>: 58% confidence (sparse signal, needs 5-10x more data)</li>
</ul>

<p>This is normal! Low-variance metrics (latency, tokens) prove out quickly. High-variance metrics (cost, feedback) need massive samples. <strong>You may not be able to wait for every metric to hit 90%</strong>. Use strong signals on some metrics plus directional insights on others.</p>

<blockquote>
<h2>
  
  
  ✅ VERDICT: Deploy Strict Security: Enhanced Privacy is Worth the Modest Cost
</h2>

<p>The results tell a compelling story: <strong>Latency (p95)</strong> is approaching significance with <strong>87% confidence</strong> that Strict Security is faster, a win we didn't anticipate. <strong>Cost per request</strong> shows <strong>79% confidence</strong> that Basic Security costs less (or conversely, 21% confidence that Strict costs more), also approaching significance. Meanwhile, <strong>positive feedback rate</strong> remains inconclusive with only <strong>58% confidence</strong> that Strict Security performs better, indicating we need more data to draw conclusions about user satisfaction.</p>
</blockquote>

<p><strong>The Hidden Cost Paradox:</strong></p>

<p>Strict Security uses <strong>FREE Mistral</strong> for PII detection, yet <strong>increases total system cost by 14.6%</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Basic Security (Claude Haiku):
- Supervisor: gpt-4o-mini     ~\$0.0001
- Security:   claude-haiku    ~\$0.0003
- Support:    gpt-4o          ~\$0.0242
Total: \$0.0246

Strict Security (Mistral):
- Supervisor: gpt-4o-mini     ~\$0.0001
- Security:   mistral         \$0.0000  (FREE!)
- Support:    gpt-4o          ~\$0.0280  (+15.7%)
Total: \$0.0281 (+14.6%)
</code></pre>

</div>



<p><strong>Why does the support agent cost more?</strong> More aggressive PII redaction removes context, forcing the support agent to generate longer, more detailed responses to compensate for the missing information. This demonstrates why <strong>system-level experiments</strong> matter. Optimizing one agent can inadvertently increase costs downstream.</p>

<p><strong>Decision Logic:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>IF latency increase ≤ 3s
   AND cost increase ≤ 15% AND confidence ≥ 75%
   AND positive_feedback_rate stable or improving
   AND enhanced_privacy_protection = true
THEN deploy_strict_security()
ELSE need_more_data()
</code></pre>

</div>



<p><strong>Bottom line:</strong> Deploy Strict Security. We expected latency to stay within 3 seconds of baseline, but discovered a <strong>36% improvement</strong> instead (87% confidence). Mistral is significantly faster than Claude Haiku. Combined with enhanced privacy protection, this more than justifies the modest 14.5% cost increase (79% confidence). </p>

<p><strong>Read across:</strong> At scale, paying ~\$0.004 more per request for significantly better privacy compliance <em>and</em> faster responses is a clear win for EU users and privacy-conscious segments.</p>

<p><strong>The Data That Proves It:</strong><br>
<br></p>

<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6q2n6iehyrxj6eiyivdw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6q2n6iehyrxj6eiyivdw.png" alt="Security Level Experiment Results" width="800" height="569"></a><br>
</p>



<h3>
  
  
  <strong>2. Premium Model Value Analysis: Does Claude Opus 4 justify its premium cost with superior positive feedback rates?</strong>
</h3>

<blockquote>
<h2>
  
  
  🔴 VERDICT: Reject Claude Opus 4
</h2>

<p>The experiment delivered a decisive verdict: <strong>Positive feedback rate</strong> showed a significant failure with <strong>99.5% confidence</strong> that GPT-4o is superior. <strong>Cost per request</strong> is approaching significance with <strong>76% confidence</strong> that Claude Opus is <strong>33% more expensive</strong>, while <strong>latency (p95)</strong> reached significance with <strong>91% confidence</strong> that Claude Opus is <strong>81% slower</strong>. The <strong>cost-to-value ratio</strong> tells the whole story: <strong>-1.9x</strong>, meaning we're paying 33% more for 64% worse performance: a clear case of premium pricing without premium results.</p>
</blockquote>

<p><strong>Decision Logic:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>IF positive_feedback_rate increase ≥ 15%
   AND probability_to_beat for positive_feedback_rate ≥ 90%
   AND probability_to_beat for cost ≥ 90%
   AND cost-value ratio increase ≥ .25
THEN deploy_claude_opus_4()
ELSE keep_current_model()
</code></pre>

</div>



<p><strong>Bottom line:</strong> Premium price delivered worse results on every metric. Experiment was stopped when positive feedback rate reached significance.</p>

<p><strong>Read across:</strong> GPT-4o dominates on performance and speed and most likely also on cost.</p>

<p><strong>The Numbers Don't Lie:</strong></p>



<p><br>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjqy1vb29qzc7or51vs1c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjqy1vb29qzc7or51vs1c.png" alt="Premium Model Value Analysis Results" width="800" height="596"></a><br>
</p>



<h3>
  
  
  <strong>Key Insights from Real Experiment Data</strong>
</h3>

<ol>
<li><p>Low-variance metrics (latency, tokens) reach significance quickly (~1,000 samples). High-variance metrics (cost, feedback) may need 5,000-10,000+ samples. This isn't a flaw in your experiment but the reality of statistical power. Don't chase 90% confidence on every metric; focus on directional insights for high-variance metrics and statistical proof for low-variance ones.</p></li>
<li><p>Using a free Mistral model for security reduced that agent's cost to \$0, yet <strong>increased total system cost by 14.5%</strong> because downstream agents had to work harder with reduced context. However, the experiment also revealed an <strong>unexpected 36% latency improvement</strong>. Mistral is not just free but significantly faster. LaunchDarkly's user-level tracking captured both effects, enabling an informed decision: enhanced privacy + faster responses for ~\$0.004 more per request is a worthwhile tradeoff.</p></li>
<li><p>At 87% confidence for latency (vs 90% target), the 36% improvement is clear enough for decision-making. Perfect statistical significance is ideal, but 85-89% confidence combined with other positive signals (stable feedback, acceptable cost) can justify deployment when the effect size is large.</p></li>
</ol>

<h2>
  
  
  Experimental Limitations &amp; Mitigations
</h2>

<p><strong>Model-as-Judge Evaluation</strong></p>

<p>We use Claude to evaluate response quality rather than real users, which represents a limitation of this experimental setup. However, research shows that model-as-judge approaches correlate well with human preferences, as documented in <a href="https://arxiv.org/abs/2212.08073" rel="noopener noreferrer">Anthropic's Constitutional AI paper</a>.</p>

<p><strong>Independent Experiments</strong></p>

<p>While random assignment naturally balances security versions across model versions, preventing systematic bias, you cannot analyze interaction effects between security and model choices. If interaction effects are important to your use case, consider running a proper <a href="https://en.wikipedia.org/wiki/Factorial_experiment" rel="noopener noreferrer">factorial experiment design</a>.</p>

<p><strong>Statistical Confidence</strong><br>
LaunchDarkly uses <strong><a href="https://dev.to/home/experimentation/bayesian">Bayesian statistics</a></strong> to calculate confidence, where 90% confidence means there's a 90% probability the true effect is positive. This is NOT the same as p-value &lt; 0.10 from <a href="https://en.wikipedia.org/wiki/Frequentist_inference" rel="noopener noreferrer">frequentist tests</a>. We set the threshold at 90% (rather than 95%) to balance false positives versus false negatives, though for mission-critical features you should consider raising the confidence threshold to 95%.</p>

<h2>
  
  
  Common Mistakes You Just Avoided
</h2>

<p>❌ <strong>"Let's run the experiment for a week and see"</strong></p>

<p>✅ <strong>We defined success criteria upfront</strong> (≥15% improvement threshold)</p>

<p>❌ <strong>"We need 90% confidence on every metric to ship"</strong></p>

<p>✅ <strong>We used 87% confidence + directional signals</strong> (36% latency win was decision-worthy)</p>

<p>❌ <strong>"Let's run experiments until all metrics reach significance"</strong></p>

<p>✅ <strong>We understood variance</strong> (cost/feedback need 5-10x more data than latency)</p>

<p>❌ <strong>"Agent-level metrics show the full picture"</strong></p>

<p>✅ <strong>We tracked user-level workflows</strong> (revealed downstream cost increases)</p>

<h2>
  
  
  What You've Accomplished
</h2>

<p>You've built a <strong>data-driven optimization engine</strong> with statistical rigor through falsifiable hypotheses and clear success criteria. Your predefined success criteria ensure clear decisions and prevent post-hoc rationalization. Every feature investment now has quantified business impact for ROI justification, and you have a framework for continuous optimization through ongoing measurable experimentation.</p>

<h2>
  
  
  Troubleshooting
</h2>

<h3>
  
  
  <strong>Long Response Times (&gt;20 minutes)</strong>
</h3>

<p>If you see requests taking exceptionally long, the root cause is likely the <code>semantic_scholar</code> MCP tool hitting API rate limits, which causes 30-second retry delays. Queries using this tool may take 5-20 minutes to complete. The 2000-second timeout handles this gracefully, but if you need faster responses (60-120 seconds typical), consider removing <code>semantic_scholar</code> from tool configurations. You can verify this issue by checking logs for <code>HTTP/1.1 429</code> errors indicating rate limiting.</p>

<h3>
  
  
  <strong>Cost Metrics Not Appearing</strong>
</h3>

<p>If <code>ai_cost_per_request</code> events aren't showing in LaunchDarkly, first verify that <code>utils/cost_calculator.py</code> has pricing configured for your models. Cost is only tracked when requests complete successfully (not on timeout or error). The system flushes cost events to LaunchDarkly immediately after each request completion. To debug, look for <code>COST CALCULATED:</code> and <code>COST TRACKING (async):</code> messages in your API logs.</p>

<h2>
  
  
  Beyond This Tutorial: Advanced AI Experimentation Patterns
</h2>

<h3>
  
  
  <strong>Other AI experimentation types you can run in LaunchDarkly</strong>
</h3>

<p><em>Context from earlier:</em> you ran two experiments:</p>

<ul>
<li>
<strong>Security‑agent test</strong>: a <strong>bundle change</strong> (both prompt/instructions <strong>and</strong> model changed).</li>
<li>
<strong>Premium‑model test</strong>: a <strong>model‑only</strong> change.</li>
</ul>

<p>AI Configs come in two modes: <strong>prompt‑based</strong> (single‑step completions) and <strong>agent‑based</strong> (multi‑step workflows with tools). Below are additional patterns to explore.</p>




<h4>
  
  
  Experiments you can run <strong>entirely in AI Configs</strong> (no app redeploy)
</h4>

<ul>
<li><p><strong>Prompt &amp; template experiments (prompt‑based or agent instructions)</strong><br>
Duplicate a variation and iterate on system/assistant messages or agent instructions to measure adherence to schema, tone, or qualitative satisfaction. Use LaunchDarkly Experimentation to tie those variations to product metrics.</p></li>
<li><p><strong>Model‑parameter experiments</strong><br>
In a single model, vary parameters like <code>temperature</code> or <code>max_tokens</code>, and (optionally) add <strong>custom parameters</strong> you define (for example, an internal <code>max_tool_calls</code> or decoding setting) directly on the variation.</p></li>
<li><p><strong>Tool‑bundle experiments (agent mode or tool‑enabled completions)</strong><br>
Attach/detach reusable tools from the <strong>Tools Library</strong> to compare stacks (e.g., <code>search_v2</code>, a reranker, or MCP‑exposed research tools) across segments. Keep one variable at a time when possible</p></li>
<li><p><strong>Cost/latency trade‑offs</strong><br>
Compare "slim" vs "premium" stacks by segment. Track tokens, time‑to‑first‑token, duration, and satisfaction to decide where higher spend is warranted.</p></li>
</ul>

<blockquote>
<p><strong>Practical notes</strong></p>

<ul>
<li>Use <strong>Experimentation</strong> for behavior impact (clicks, task success); use the <strong>Monitoring</strong> tab for LLM‑level metrics (tokens, latency, errors, satisfaction).</li>
</ul>
<h2>
  
  
  - You <strong>can't</strong> run a guarded rollout and an experiment on the same flag at the same time. Pick one: guarded rollout for risk‑managed releases, experiment for causal measurement.
</h2>
</blockquote>

<h4>
  
  
  Patterns that <strong>usually need feature flags and/or custom instrumentation</strong>
</h4>

<ul>
<li><p><strong>Fine‑grained RAG tuning</strong><br>
k‑values, similarity thresholds, chunking, reranker swaps, and cache policy are typically coded inside your retrieval layer. Expose these as flags or AI Config custom parameters if you want to A/B them.</p></li>
<li><p><strong>Tool‑routing guardrails</strong><br>
Fallback flows (e.g., retry with a different tool/model on error), escalation rules, or heuristics need logic in your agent/orchestrator. Gate those behaviors behind flags and measure with custom metrics.</p></li>
<li><p><strong>Safety guardrail calibration</strong><br>
Moderation thresholds, red‑team prompts, and PII sensitivity levers belong in a dedicated safety service or the agent wrapper. Wire them to flags so you can raise/lower sensitivity by segment (e.g., enterprise vs free).</p></li>
<li><p><strong>Session budget enforcement</strong><br>
Monitoring will show token costs and usage, but enforcing per‑session or per‑org budgets (denylist, degrade model, or stop‑tooling) requires application logic. Wrap policies in flags before you experiment.</p></li>
</ul>




<h4>
  
  
  Targeting &amp; segmentation ideas (works across all the above)
</h4>

<ul>
<li>Route by <strong>plan/tier</strong>, <strong>geo</strong>, <strong>device</strong>, or <strong>org</strong> using AI Config targeting rules and percentage rollouts.</li>
<li>Keep variations narrow (one change per experiment) to avoid confounding; reserve "bundle" tests for tool‑stack bake‑offs.</li>
</ul>

<p><strong>Advanced Practices:</strong> Require statistical evidence before shipping configuration changes. Pair each variation with clear success metrics, then A/B test prompt or tool adjustments and use confidence intervals to confirm improvements. When you introduce the new code paths above, protect them behind feature flags so you can run sequential tests, <a href="https://dev.to/home/multi-armed-bandits">multi-armed bandits</a> for faster convergence, or change your <a href="https://dev.to/guides/experimentation/designing-experiments">experiment design</a> to understand how prompts, tools, and safety levers interact.</p>

<h2>
  
  
  From Chaos to Clarity
</h2>

<p>Across this three-part series, you've transformed from hardcoded AI configurations to a scientifically rigorous, data-driven optimization engine. <strong><a href="https://dev.to/tutorials/agents-langgraph">Part 1</a></strong> established your foundation with a dynamic multi-agent <a href="https://langchain-ai.github.io/langgraph/" rel="noopener noreferrer">LangGraph</a> system controlled by <a href="https://dev.to/guides/ai-configs">LaunchDarkly AI Configs</a>, eliminating the need for code deployments when adjusting AI behavior. <strong><a href="https://dev.to/tutorials/multi-agent-mcp-targeting">Part 2</a></strong> added sophisticated targeting with geographic privacy rules, user segmentation by plan tiers, and <a href="https://modelcontextprotocol.io/" rel="noopener noreferrer">MCP (Model Context Protocol)</a> tool integration for real academic research capabilities. This tutorial, <strong>Part 3</strong> completed your journey with statistical experimentation that proves ROI and guides optimization decisions with mathematical confidence rather than intuition.</p>

<p>You now possess a defensible AI system that adapts to changing requirements, scales across user segments, and continuously improves through measured experimentation. Your stakeholders receive concrete evidence for AI investments, your engineering team deploys features with statistical backing, and your users benefit from optimized experiences driven by real data rather than assumptions. The chaos of ad-hoc AI development has given way to clarity through systematic, scientific product development.</p>

<h2>
  
  
  Resources
</h2>

<h2>
  
  
  - <strong><a href="https://dev.to/home/experimentation">LaunchDarkly Experimentation Docs</a></strong> - Deep dive into statistical methods
</h2>

<p><strong>Remember:</strong> Every AI decision backed by data is a risk avoided and a lesson learned. Start small, measure everything, ship with confidence.</p>

