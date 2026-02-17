---
Title: Surprise: You Can "Intercept" the C# lock Statement
Description: 
Author: Dmitry Dorogoy
Date: 2026-02-17T22:09:04.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  You can “hijack” <code>lock</code> in C#. Here is how it works, and why you should not do it.
</h1>

<p>Many developers think <code>lock</code> is some kind of runtime “magic”. It is not. In most cases it is plain syntax sugar: the compiler rewrites it into calls to <code>System.Threading.Monitor</code>.</p>

<p>That rewrite has an ugly edge case. If your project defines its own <code>System.Threading.Monitor</code>, the compiler can bind to <em>your</em> type instead of the BCL one. In other words, you can change what <code>lock</code> means.</p>

<p>This is a party trick. Also a foot-gun. Treat it as a cautionary tale, not a technique.</p>

<blockquote>
<p>Note: starting with <strong>.NET 9</strong> and <strong>C# 13</strong>, <code>lock</code> has a special fast path when the expression is a <code>System.Threading.Lock</code>. In that case, it compiles to <code>using (x.EnterScope()) { ... }</code> instead of <code>Monitor.Enter/Exit</code>.<br><br>
The hijack shown below applies to the classic <code>lock (object)</code> path.</p>
</blockquote>




<h2>
  
  
  What the compiler generates for <code>lock</code>
</h2>

<p>For the classic <code>lock (obj) { ... }</code> case, the C# compiler generates code equivalent to this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">object</span> <span class="n">_lockObj</span> <span class="p">=</span> <span class="n">obj</span><span class="p">;</span>
<span class="kt">bool</span> <span class="n">_lockWasTaken</span> <span class="p">=</span> <span class="k">false</span><span class="p">;</span>
<span class="k">try</span>
<span class="p">{</span>
    <span class="n">System</span><span class="p">.</span><span class="n">Threading</span><span class="p">.</span><span class="n">Monitor</span><span class="p">.</span><span class="nf">Enter</span><span class="p">(</span><span class="n">_lockObj</span><span class="p">,</span> <span class="k">ref</span> <span class="n">_lockWasTaken</span><span class="p">);</span>
    <span class="c1">// Your code...</span>
<span class="p">}</span>
<span class="k">finally</span>
<span class="p">{</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">_lockWasTaken</span><span class="p">)</span> <span class="n">System</span><span class="p">.</span><span class="n">Threading</span><span class="p">.</span><span class="n">Monitor</span><span class="p">.</span><span class="nf">Exit</span><span class="p">(</span><span class="n">_lockObj</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>So <code>lock</code> is not “special” at runtime. It is a compiler pattern that expands into known calls.</p>




<h2>
  
  
  The hijack trick
</h2>

<p>If you define a type with the same fully-qualified name as the BCL type, you can make the compiler call your methods.</p>

<p>Minimal example:</p>

<ul>
<li>LINQPad snippet: <a href="https://share.linqpad.net/dmgs488o.linq" rel="noopener noreferrer">https://share.linqpad.net/dmgs488o.linq</a>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">Program</span>
<span class="p">{</span>
    <span class="k">static</span> <span class="k">readonly</span> <span class="kt">object</span> <span class="n">_sync</span> <span class="p">=</span> <span class="k">new</span><span class="p">();</span>

    <span class="k">static</span> <span class="k">void</span> <span class="nf">Main</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="c1">// Looks like a normal lock, but it is hijacked.</span>
        <span class="k">lock</span> <span class="p">(</span><span class="n">_sync</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"Working inside the lock..."</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// We "fake" the system namespace and class.</span>
<span class="k">namespace</span> <span class="nn">System.Threading</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">Monitor</span>
    <span class="p">{</span>
        <span class="k">public</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">Enter</span><span class="p">(</span><span class="kt">object</span> <span class="n">obj</span><span class="p">,</span> <span class="k">ref</span> <span class="kt">bool</span> <span class="n">lockTaken</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"Hijacked: Enter() was called"</span><span class="p">);</span>
            <span class="n">lockTaken</span> <span class="p">=</span> <span class="k">true</span><span class="p">;</span> <span class="c1">// Important: otherwise Exit() won't be called.</span>
        <span class="p">}</span>

        <span class="k">public</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">Exit</span><span class="p">(</span><span class="kt">object</span> <span class="n">obj</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"Hijacked: Exit() was called"</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Run it and you will see the <code>Hijacked:</code> messages. That is your proof that the compiler bound <code>lock</code> to <strong>your</strong> <code>System.Threading.Monitor</code>.</p>

<h3>
  
  
  Why does this work?
</h3>

<p>Because name resolution happens at compile time.</p>

<p>Your compilation contains a <code>System.Threading.Monitor</code> type, and there is also one in referenced assemblies. The compiler sees both and chooses one. If it picks yours, the rewrite still happens, but it targets your methods.</p>

<p>This is exactly what the compiler warning is trying to tell you.</p>




<h2>
  
  
  Do not ignore the warning
</h2>

<p>This code should produce <strong>CS0436</strong>. In this scenario it is not “noise”. It is the whole point.</p>

<p>CS0436 means:</p>

<ul>
<li>there is a conflict between a type in your compilation and an imported type</li>
<li>the compiler will use the type from your compilation</li>
</ul>

<p>If your custom <code>Enter</code> method does not actually lock, then multiple threads can run inside the supposed critical section at the same time. That breaks invariants and causes the worst kind of bugs: rare races that are hard to reproduce and disappear under the debugger.</p>

<p>It can also happen accidentally:</p>

<ul>
<li>a helper class is named <code>Monitor</code> and ends up in the wrong namespace</li>
<li>a dependency ships a conflicting type</li>
<li>warnings are suppressed globally and CS0436 is missed</li>
</ul>

<p>If you ever see CS0436 around <code>System.Threading.Monitor</code>, stop and investigate.</p>




<h2>
  
  
  A safer goal: measure lock contention
</h2>

<p>If your goal is observability, not sabotage, there is a better approach: measure contention without touching <code>lock</code> at all.</p>

<p>.NET exposes runtime events for monitor contention:</p>

<ul>
<li>
<code>ContentionStart_V2</code> has event id <strong>81</strong>
</li>
<li>
<code>ContentionStop_V1</code> has event id <strong>91</strong>
</li>
<li>
<code>ContentionStop_V1</code> includes <code>DurationNs</code>
</li>
<li>the keyword is <code>ContentionKeyword</code> (<strong>0x4000</strong>)</li>
</ul>

<p>You can listen to these events via <code>EventListener</code>.</p>

<ul>
<li>LINQPad snippet: <a href="https://share.linqpad.net/6iebniko.linq" rel="noopener noreferrer">https://share.linqpad.net/6iebniko.linq</a>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Diagnostics.Tracing</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Threading</span><span class="p">;</span>

<span class="k">sealed</span> <span class="k">class</span> <span class="nc">LockMonitor</span> <span class="p">:</span> <span class="n">EventListener</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="kt">long</span> <span class="n">TotalWaitTimeNs</span><span class="p">;</span>
    <span class="kt">int</span> <span class="n">_durationIndex</span> <span class="p">=</span> <span class="p">-</span><span class="m">1</span><span class="p">;</span>

    <span class="k">protected</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">OnEventSourceCreated</span><span class="p">(</span><span class="n">EventSource</span> <span class="n">source</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">source</span><span class="p">.</span><span class="n">Name</span> <span class="p">==</span> <span class="s">"Microsoft-Windows-DotNETRuntime"</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="nf">EnableEvents</span><span class="p">(</span><span class="n">source</span><span class="p">,</span> <span class="n">EventLevel</span><span class="p">.</span><span class="n">Informational</span><span class="p">,</span> <span class="p">(</span><span class="n">EventKeywords</span><span class="p">)</span><span class="m">0x4000</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">protected</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">OnEventWritten</span><span class="p">(</span><span class="n">EventWrittenEventArgs</span> <span class="n">eventData</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="c1">// ContentionStop_V1</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">eventData</span><span class="p">.</span><span class="n">EventId</span> <span class="p">!=</span> <span class="m">91</span> <span class="p">||</span> <span class="n">eventData</span><span class="p">.</span><span class="n">PayloadNames</span> <span class="k">is</span> <span class="k">null</span><span class="p">)</span>
            <span class="k">return</span><span class="p">;</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">_durationIndex</span> <span class="p">&lt;</span> <span class="m">0</span><span class="p">)</span>
            <span class="n">_durationIndex</span> <span class="p">=</span> <span class="n">eventData</span><span class="p">.</span><span class="n">PayloadNames</span><span class="p">.</span><span class="nf">IndexOf</span><span class="p">(</span><span class="s">"DurationNs"</span><span class="p">);</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">_durationIndex</span> <span class="p">&lt;</span> <span class="m">0</span><span class="p">)</span>
            <span class="k">return</span><span class="p">;</span>

        <span class="c1">// DurationNs is documented as a Double. Convert and keep nanoseconds as a long for easy accumulation.</span>
        <span class="kt">var</span> <span class="n">durationNs</span> <span class="p">=</span> <span class="p">(</span><span class="kt">long</span><span class="p">)</span><span class="n">Convert</span><span class="p">.</span><span class="nf">ToDouble</span><span class="p">(</span><span class="n">eventData</span><span class="p">.</span><span class="n">Payload</span><span class="p">![</span><span class="n">_durationIndex</span><span class="p">]!);</span>
        <span class="n">Interlocked</span><span class="p">.</span><span class="nf">Add</span><span class="p">(</span><span class="k">ref</span> <span class="n">TotalWaitTimeNs</span><span class="p">,</span> <span class="n">durationNs</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  What you get with this approach
</h3>

<ul>
<li>
<code>lock</code> remains a real lock.</li>
<li>You collect how much time threads spend waiting.</li>
<li>You can compare “good” and “bad” code paths and see contention clearly.</li>
</ul>

<p>If you need a deeper report (per lock object, stack traces, timelines), use tools like <code>dotnet-trace</code>, PerfView, or ETW/EventPipe-based profilers. The small <code>EventListener</code> above is a good minimal demo.</p>




<h2>
  
  
  Takeaway
</h2>

<ul>
<li>
<code>lock</code> is compiler sugar that usually expands into <code>System.Threading.Monitor.Enter/Exit</code>.</li>
<li>A type name collision can change what the compiler emits. CS0436 warns you that you are changing meaning.</li>
<li>Hijacking <code>lock</code> is a neat demo, but a terrible idea in real code.</li>
<li>For diagnostics, prefer contention events and proper tooling instead of tricks.</li>
</ul>

