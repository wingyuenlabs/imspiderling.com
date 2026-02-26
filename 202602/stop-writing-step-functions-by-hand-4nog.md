---
Title: Stop Writing Step Functions by Hand
Description: 
Author: Zed
Date: 2026-02-26T22:10:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you've worked with Step Functions at any real scale, you know the concept is great — orchestrate services, handle retries, build resilient workflows. The problem has always been the <em>authoring experience</em>.</p>

<p>You'd start with a CDK stack. Chain some constructs together with <code>.next().next().next()</code>. Scatter raw JSONPath strings everywhere. Hope that <code>$.orderResult.Payload.body.items[0].price</code> actually resolves to something when it hits production. When it doesn't? Enjoy your <code>States.Runtime</code> error in CloudWatch at 2am with zero useful context about which path expression is wrong.</p>

<p>AWS introduced JSONata last year to replace JSONPath, and it's a massive improvement. But even with JSONata, you're still writing ASL by hand — still managing <code>Arguments</code>, <code>Assign</code>, <code>Output</code> fields, still wiring <code>{% $states.input.whatever %}</code> expressions throughout a JSON document that grows to hundreds of lines for any non-trivial workflow.</p>

<p>I kept thinking: this is just a program. Control flow, data transformation, error handling. Why am I writing it in JSON?</p>

<p>So I built <a href="https://github.com/DevNamedZed/simplesteps" rel="noopener noreferrer">SimpleSteps</a> — a TypeScript-to-ASL compiler. You write workflows as typed async functions, and the compiler handles the rest.</p>

<h2>
  
  
  What This Looks Like in Practice
</h2>

<p>Here's an order validation workflow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">machine</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SimpleStepsStateMachine</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">'</span><span class="s1">OrderWorkflow</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">workflow</span><span class="p">:</span> <span class="nx">Steps</span><span class="p">.</span><span class="nf">createFunction</span><span class="p">(</span>
    <span class="k">async </span><span class="p">(</span><span class="na">context</span><span class="p">:</span> <span class="nx">SimpleStepContext</span><span class="p">,</span> <span class="na">input</span><span class="p">:</span> <span class="p">{</span> <span class="na">orderId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span> <span class="nl">customerId</span><span class="p">:</span> <span class="kr">string</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">order</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">validateOrder</span><span class="p">.</span><span class="nf">call</span><span class="p">({</span> <span class="na">orderId</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">orderId</span> <span class="p">});</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">order</span><span class="p">.</span><span class="nx">valid</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Invalid order</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">}</span>

      <span class="k">await</span> <span class="nx">orders</span><span class="p">.</span><span class="nf">putItem</span><span class="p">({</span>
        <span class="na">Item</span><span class="p">:</span> <span class="p">{</span>
          <span class="na">id</span><span class="p">:</span> <span class="p">{</span> <span class="na">S</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">orderId</span> <span class="p">},</span>
          <span class="na">customerId</span><span class="p">:</span> <span class="p">{</span> <span class="na">S</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">customerId</span> <span class="p">},</span>
          <span class="na">total</span><span class="p">:</span> <span class="p">{</span> <span class="na">N</span><span class="p">:</span> <span class="nc">String</span><span class="p">(</span><span class="nx">order</span><span class="p">.</span><span class="nx">total</span><span class="p">)</span> <span class="p">},</span>
          <span class="na">status</span><span class="p">:</span> <span class="p">{</span> <span class="na">S</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CONFIRMED</span><span class="dl">'</span> <span class="p">},</span>
        <span class="p">},</span>
      <span class="p">});</span>

      <span class="k">return</span> <span class="p">{</span> <span class="na">orderId</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">orderId</span><span class="p">,</span> <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">CONFIRMED</span><span class="dl">'</span> <span class="p">};</span>
    <span class="p">},</span>
  <span class="p">),</span>
<span class="p">});</span>
</code></pre>

</div>



<p>That compiles to a full JSONata-mode ASL state machine. The compiler infers all the <code>Arguments</code>, <code>Assign</code>, and <code>Output</code> fields from how you use variables. You never write a <code>{% %}</code> expression. You never think about <code>$states.input</code> vs <code>$states.result</code>. You just write TypeScript.</p>

<p>For comparison, here's the kind of ASL that gets generated from a simple Lambda call:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"QueryLanguage"</span><span class="p">:</span><span class="w"> </span><span class="s2">"JSONata"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"StartAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Invoke_helloFn"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"States"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Invoke_helloFn"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:lambda:us-east-1:123456789:function:Hello"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Arguments"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"{% $states.input.name %}"</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="nl">"Assign"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"result"</span><span class="p">:</span><span class="w"> </span><span class="s2">"{% $states.result %}"</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Return_Result"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"Return_Result"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Pass"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Output"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"greeting"</span><span class="p">:</span><span class="w"> </span><span class="s2">"{% $result.greeting %}"</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="nl">"End"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>You could write that by hand. Or you could write this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">helloFn</span><span class="p">.</span><span class="nf">call</span><span class="p">({</span> <span class="na">name</span><span class="p">:</span> <span class="nx">input</span><span class="p">.</span><span class="nx">name</span> <span class="p">});</span>
<span class="k">return</span> <span class="p">{</span> <span class="na">greeting</span><span class="p">:</span> <span class="nx">result</span><span class="p">.</span><span class="nx">greeting</span> <span class="p">};</span>
</code></pre>

</div>



<p>Multiply that difference across a real workflow with branching, loops, parallel execution, error handling, and retries — and the gap between "readable" and "not" gets enormous.</p>

<h2>
  
  
  Why This Matters
</h2>

<p><strong>Type safety that actually catches things.</strong> The <code>validateOrder</code> binding is typed as <code>Lambda&lt;{ orderId: string }, { valid: boolean; total: number }&gt;</code>. Pass it the wrong shape and TypeScript tells you at compile time. In raw ASL, you find out when your workflow fails in production because you fat-fingered a JSONata variable name inside a <code>{% %}</code> block.</p>

<p><strong>Standard JS methods compile to JSONata.</strong> This is where it gets fun. The compiler maps 64+ JavaScript methods directly to JSONata built-ins. <code>Math.round()</code>, <code>str.toUpperCase()</code>, <code>arr.filter()</code>, <code>arr.map()</code> — they all compile to their JSONata equivalents. You can even write lambda expressions in <code>.map()</code>, <code>.filter()</code>, and <code>.reduce()</code> and they'll auto-compile to JSONata higher-order functions. You're writing TypeScript that <em>looks and feels</em> like TypeScript, but it becomes a state machine.</p>

<p><strong>It's actually testable.</strong> <code>@simplesteps/local</code> ships a local execution engine that runs your compiled state machines with mocked services. No Docker, no AWS credentials, no deploying anything. You write Jest tests like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">compile</span><span class="p">,</span> <span class="nx">AslSerializer</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@simplesteps/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">LocalRunner</span><span class="p">,</span> <span class="nx">StateMachineError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@simplesteps/local</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Compile your workflow to ASL</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nf">compile</span><span class="p">({</span> <span class="na">tsconfigPath</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./tsconfig.workflows.json</span><span class="dl">'</span> <span class="p">});</span>
<span class="kd">const</span> <span class="nx">asl</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">AslSerializer</span><span class="p">.</span><span class="nf">serialize</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">stateMachines</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nx">definition</span><span class="p">));</span>

<span class="c1">// Mock services by ARN — just functions that return data</span>
<span class="kd">const</span> <span class="nx">services</span> <span class="o">=</span> <span class="p">{</span>
  <span class="dl">'</span><span class="s1">arn:aws:lambda:...ValidateOrder</span><span class="dl">'</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">valid</span><span class="p">:</span> <span class="kc">true</span> <span class="p">}),</span>
  <span class="dl">'</span><span class="s1">arn:aws:lambda:...ChargePayment</span><span class="dl">'</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">({</span> <span class="na">chargeId</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ch_123</span><span class="dl">'</span> <span class="p">}),</span>
<span class="p">};</span>

<span class="c1">// Execute locally and assert on the output</span>
<span class="kd">const</span> <span class="nx">runner</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LocalRunner</span><span class="p">(</span><span class="nx">asl</span><span class="p">,</span> <span class="p">{</span> <span class="nx">services</span> <span class="p">});</span>
<span class="kd">const</span> <span class="nx">output</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">runner</span><span class="p">.</span><span class="nf">execute</span><span class="p">({</span> <span class="na">orderId</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ORD-1</span><span class="dl">'</span><span class="p">,</span> <span class="na">total</span><span class="p">:</span> <span class="mf">99.99</span> <span class="p">});</span>
<span class="nf">expect</span><span class="p">(</span><span class="nx">output</span><span class="p">.</span><span class="nx">status</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">completed</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p>You can test every branch by swapping mocks (return <code>{ valid: false }</code> and verify the rejection path runs), test error handling by throwing <code>StateMachineError</code> from a mock, and inspect the full execution trace with <code>executeWithTrace()</code> to verify state ordering, step counts, and that specific services were or weren't called. Compare that to your current Step Functions testing strategy, which is probably "deploy and pray."</p>

<p><strong>It's readable.</strong> Hand a new team member a 20-line async function versus a 150-line ASL document full of JSONata expressions and <code>Assign</code> blocks. I know which one I'd rather onboard someone with.</p>

<h2>
  
  
  The Compiler Goes Deep
</h2>

<p>This isn't a thin wrapper that handles happy-path <code>await = Task</code> and punts on everything else. The full TypeScript control flow maps to ASL:</p>

<p><code>if/else</code> and <code>switch/case</code> become Choice states. <code>while</code> and <code>do...while</code> become loops. <code>for...of</code> compiles to Map states for parallel iteration. <code>Promise.all()</code> becomes a Parallel state. <code>try/catch</code> generates Catch rules. <code>throw</code> becomes a Fail state. Retry policies are just an options argument on <code>.call()</code>.</p>

<p>Under the hood, the compiler does whole-program data flow analysis with constant propagation, cross-file import resolution with cycle detection, and pure function inlining. When something goes wrong, you get real diagnostics — 50+ error codes with root-cause attribution and poisoned-value chain tracking. Not a generic "invalid state machine definition" from CloudFormation twenty minutes into a deploy.</p>

<h2>
  
  
  64 Typed AWS Service Bindings
</h2>

<p>16 services have optimized direct integrations — Lambda, DynamoDB, SQS, SNS, EventBridge, S3, Secrets Manager, SSM, ECS, Bedrock, Glue, CodeBuild, Athena, Batch, StepFunction, and HttpEndpoint. On top of that, there are 48 SDK-generated bindings with full type signatures, plus <code>Steps.awsSdk()</code> as an escape hatch for anything else. All of them are typed. All of them participate in data flow analysis.</p>

<h2>
  
  
  CDK Integration
</h2>

<p>You don't have to abandon your existing stack. <code>SimpleStepsStateMachine</code> is a CDK construct that drops right in:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">SimpleStepsStateMachine</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@simplesteps/cdk</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">machine</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SimpleStepsStateMachine</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">'</span><span class="s1">MyWorkflow</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">workflow</span><span class="p">:</span> <span class="nx">Steps</span><span class="p">.</span><span class="nf">createFunction</span><span class="p">(</span><span class="k">async </span><span class="p">(</span><span class="nx">context</span><span class="p">,</span> <span class="nx">input</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="c1">// your workflow</span>
  <span class="p">}),</span>
<span class="p">});</span>

<span class="nx">myLambda</span><span class="p">.</span><span class="nf">grantInvoke</span><span class="p">(</span><span class="nx">machine</span><span class="p">);</span>
<span class="nx">myTable</span><span class="p">.</span><span class="nf">grantWriteData</span><span class="p">(</span><span class="nx">machine</span><span class="p">);</span>
</code></pre>

</div>



<p>CDK tokens (<code>Ref</code>, <code>Fn::GetAtt</code>) propagate through cleanly — the compiler understands CloudFormation intrinsics and handles substitution at synth time. You can also define workflows in separate files using <code>sourceFile</code> + <code>bindings</code> if you prefer to keep your workflow logic out of your stack definitions.</p>

<p>There's also a CLI if you just want raw ASL output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx simplesteps compile workflow.ts <span class="nt">-o</span> output/
<span class="c"># JSONata is the default; use --query-language jsonpath if you need it</span>
npx simplesteps compile workflow.ts <span class="nt">-o</span> output/ <span class="nt">--query-language</span> jsonpath
</code></pre>

</div>



<h2>
  
  
  Try It
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @simplesteps/core @simplesteps/cdk @simplesteps/local
</code></pre>

</div>



<p>There's a <a href="https://devnamedzed.github.io/simplesteps/" rel="noopener noreferrer">playground</a> where you can paste TypeScript and see compiled ASL in real time, a <a href="https://github.com/DevNamedZed/simplesteps/tree/main/examples/starters/testing" rel="noopener noreferrer">testing starter</a> with a full Jest setup you can clone and run immediately, plus <a href="https://github.com/DevNamedZed/simplesteps/tree/main/examples/showcase" rel="noopener noreferrer">29 showcase examples</a> covering every language feature.</p>

<p>If you've ever stared at a 200-line ASL document wondering which <code>{% $states.result %}</code> is the one that's wrong, or you've lost an afternoon debugging a CDK <code>.next()</code> chain, give it a shot. I'd love to hear what you think.</p>

<p><a href="https://github.com/DevNamedZed/simplesteps" rel="noopener noreferrer">GitHub</a> · <a href="https://devnamedzed.github.io/simplesteps/" rel="noopener noreferrer">Playground</a> · <a href="https://github.com/DevNamedZed/simplesteps/tree/main/docs" rel="noopener noreferrer">Docs</a></p>

