---
Title: From Seconds to Milliseconds: Fixing Python Cold Starts with SnapStart
Description: 
Author: Anne
Date: 2025-10-27T21:49:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve built a Python Lambda that uses Pydantic heavily (or other heavy model initialization, schema loading, or framework bootstrapping), you’ve probably hit this painful boundary: cold starts are dominated by your app’s initialization time, not AWS’s runtime overhead. Even if AWS could spin up pods in less than 100 ms, your own code might spend 800 ms or more importing modules, constructing many Pydantic models, loading schemas, doing dependency injection, etc. That kind of latency becomes a dealbreaker for synchronous, latency-sensitive APIs.</p>

<p>With the availability of Lambda SnapStart for Python (runtime 3.12+), that barrier is now much lower: you can snapshot the fully-initialized environment and restore from it. In other words: the heavy startup cost of initializing your Pydantic models happens once during deployment, rather than every time the function runs.</p>

<h2>
  
  
  What is Lambda SnapStart (for Python) and how it changes cold starts
</h2>

<p>Before SnapStart, Lambda cold starts involved:</p>

<ol>
<li>Downloading your code package</li>
<li>Starting the language runtime (Python interpreter)</li>
<li>Running all initialization code (imports, global-level constructs, class definitions, module-level state, constructor logic, etc)</li>
<li>Finally handling the incoming event</li>
</ol>

<p>If your startup logic is heavy (for example, constructing &gt;100 Pydantic model classes, loading JSON or YAML schemas, instantiating global validators or caches, etc.), that initialization phase can be your bottleneck.</p>

<p>SnapStart changes that dramatically (for supported runtimes). The flow is:</p>

<ol>
<li>You enable SnapStart on a published version of your Lambda (or via alias to a version).</li>
<li>When that version is published, Lambda “warms up” the function once, runs all initialization code, sets up your global state exactly as your code would do for a first invocation.</li>
<li>Lambda then takes a memory + disk snapshot (a Firecracker micro-VM snapshot) of that fully initialized state, encrypts and persists it in a cache.</li>
<li>Later, when a new environment is needed, (a cold) Lambda restores from that snapshot, essentially skipping the heavy init and resuming from the “post-init” state.</li>
<li>Your handler logic runs as-is.</li>
</ol>

<p>In effect, a lot of the “cold start” cost is eliminated. AWS claims that you can get “sub-second” startup times even for heavy apps.</p>

<h2>
  
  
  Snapshot safety, uniqueness, and runtime hooks
</h2>

<p>Because SnapStart literally reuses memory state, if your initialization code produces unique artifacts (for example, a new random UUID on startup, or a unique timestamp, or establishes network sockets), those may incorrectly get reused across invocations if not handled carefully.</p>

<p>To mitigate this, AWS provides runtime hooks for Python:</p>

<ul>
<li>
<code>@register_before_snapshot</code>: functions that run right before the snapshot is taken. Use this hook to “undo” or clean up any ephemeral or unique initialization state (e.g. close DB connections, clear caches, reset random seeds). </li>
<li>
<code>@register_after_restore</code>: functions that run immediately upon restore from snapshot, before handling the first event. Use this to reinstantiate anything you can’t reuse (e.g. re-open connections, reinitialize entropy, refresh tokens). </li>
</ul>

<p>These hooks help ensure your function remains correct even when operating from a snapshot.</p>

<h2>
  
  
  Using SnapStart in CDK for Python Lambdas
</h2>

<p>Let’s get into how to configure SnapStart when deploying via AWS CDK (in Python). The high-level steps are:</p>

<ol>
<li>Use a Lambda construct (e.g. aws_lambda.Function or the PythonFunction from the aws_lambda_python_alpha module).</li>
<li>Ensure your runtime is at least Python 3.12 (SnapStart is supported on Python 3.12+).</li>
<li>Enable the SnapStart property.</li>
<li>Add a version alias so that SnapStart can be applied to the published version.</li>
</ol>

<p>Here is a sample skeleton (CDK in Python):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">aws_cdk</span> <span class="kn">import</span> <span class="p">(</span>
  <span class="n">Stack</span><span class="p">,</span>
  <span class="n">Duration</span><span class="p">,</span>
  <span class="n">aws_lambda</span> <span class="k">as</span> <span class="n">_lambda</span><span class="p">,</span>
  <span class="n">aws_lambda_python_alpha</span> <span class="k">as</span> <span class="n">lambda_python</span><span class="p">,</span>
  <span class="n">aws_iam</span> <span class="k">as</span> <span class="n">iam</span><span class="p">,</span>
<span class="p">)</span>
<span class="kn">from</span> <span class="n">constructs</span> <span class="kn">import</span> <span class="n">Construct</span>

<span class="k">class</span> <span class="nc">MyLambdaStack</span><span class="p">(</span><span class="n">Stack</span><span class="p">):</span>
  <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">scope</span><span class="p">:</span> <span class="n">Construct</span><span class="p">,</span> <span class="nb">id</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
    <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">scope</span><span class="p">,</span> <span class="nb">id</span><span class="p">,</span> <span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>

    <span class="c1"># 1. Define the Lambda
</span>    <span class="n">my_fn</span> <span class="o">=</span> <span class="n">lambda_python</span><span class="p">.</span><span class="nc">PythonFunction</span><span class="p">(</span>
      <span class="n">self</span><span class="p">,</span> <span class="sh">"</span><span class="s">MyPydanticFn</span><span class="sh">"</span><span class="p">,</span>
      <span class="n">runtime</span><span class="o">=</span><span class="n">_lambda</span><span class="p">.</span><span class="n">Runtime</span><span class="p">.</span><span class="n">PYTHON_3_12</span><span class="p">,</span> <span class="c1"># 2. Correct runtime
</span>      <span class="n">handler</span><span class="o">=</span><span class="sh">"</span><span class="s">app.handler</span><span class="sh">"</span><span class="p">,</span>
      <span class="n">entry</span><span class="o">=</span><span class="sh">"</span><span class="s">path/to/your/code</span><span class="sh">"</span><span class="p">,</span>
      <span class="n">memory_size</span><span class="o">=</span><span class="mi">1024</span><span class="p">,</span>
      <span class="n">timeout</span><span class="o">=</span><span class="n">Duration</span><span class="p">.</span><span class="nf">seconds</span><span class="p">(</span><span class="mi">30</span><span class="p">),</span>
      <span class="n">snap_start</span><span class="p">:</span> <span class="k">lambda</span><span class="p">.</span><span class="n">SnapStartConf</span><span class="p">.</span><span class="n">ON_PUBLISHED_VERSIONS</span> <span class="c1"># 3. Enable SnapStart
</span>    <span class="p">)</span>

    <span class="c1"># 4. Create a version
</span>    <span class="n">version</span> <span class="o">=</span> <span class="n">my_fn</span><span class="p">.</span><span class="n">current_version</span>  <span class="c1"># forces a Version artifact
</span></code></pre>

</div>



<p><em>One thing to notice in that pattern:<br>
We rely on <code>my_fn.current_version</code> to force CDK to emit a AWS::Lambda::Version resource (without that, there’s no version to attach SnapStart to).</em></p>

<h2>
  
  
  Things to consider
</h2>

<ol>
<li><p>The pre-snapshot initialization phase has a time limit (the INIT time limit of 10 seconds) as usual. Your before-snapshot hooks count toward that.</p></li>
<li><p>After snapshot restore, the @after_restore hook must finish within 10 seconds as well, else it might throw a SnapStartTimeoutException.</p></li>
<li><p>Whenever your code (or dependencies) change in a way that should invalidate the snapshot, you must force a new version so a new snapshot is captured. If CDK's version hashing doesn't catch something, you can call version.invalidate_version_based_on(...) with a changing token (e.g. content hash of certain files) to ensure version recreation.</p></li>
<li><p>At the time of writing, SnapStart is not supported with ephemeral storage &gt; 512 MB, EFS, provisioned concurrency, or containers.</p></li>
</ol>

<h2>
  
  
  Conclusion
</h2>

<p>In summary, AWS Lambda SnapStart for Python removes one of the biggest hurdles to running complex, model-heavy applications in a serverless environment. By snapshotting your function’s fully initialized state at deployment time, it eliminates the repeated cost of initializing frameworks like Pydantic on every cold start. With a small configuration change in CDK and a few runtime hooks to manage transient state, you can achieve consistently fast startup times and make Lambda a practical, scalable choice even for Python applications that used to be too slow to launch.</p>

