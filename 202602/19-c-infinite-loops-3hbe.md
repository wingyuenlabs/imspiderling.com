---
Title: 20. C# (Infinite Loops)
Description: 
Author: Sabin Sim
Date: 2026-02-17T21:36:05.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  0. The Real Goal of This Lesson
</h2>

<blockquote>
<p>A <code>while</code> loop repeats based on a condition,<br>
but what actually stops the loop is <strong>state change</strong>.</p>
</blockquote>

<p>The tools that create state change are:</p>

<ul>
<li><code>+=</code></li>
<li><code>-=</code></li>
<li><code>*=</code></li>
<li><code>++</code></li>
<li><code>--</code></li>
</ul>

<p>Without them, loops do not evolve.</p>

<p>They freeze.</p>

<p>And frozen state inside a loop is dangerous.</p>




<h2>
  
  
  1. Start With a Basic Review
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">int</span> <span class="n">number</span> <span class="p">=</span> <span class="m">0</span><span class="p">;</span>

<span class="k">while</span> <span class="p">(</span><span class="n">number</span> <span class="p">&lt;</span> <span class="m">5</span><span class="p">)</span>
<span class="p">{</span>
    <span class="n">number</span> <span class="p">=</span> <span class="n">number</span> <span class="p">+</span> <span class="m">1</span><span class="p">;</span>
    <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">number</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This prints:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1
2
3
4
5
</code></pre>

</div>



<p>Why?</p>

<p>Because the state (<code>number</code>) changes every iteration.</p>




<h2>
  
  
  2. The <code>+=</code> Operator (Compound Assignment)
</h2>

<p>This line:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">number</span> <span class="p">=</span> <span class="n">number</span> <span class="p">+</span> <span class="m">1</span><span class="p">;</span>
</code></pre>

</div>



<p>Can be simplified to:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">number</span> <span class="p">+=</span> <span class="m">1</span><span class="p">;</span>
</code></pre>

</div>



<p>It means:</p>

<blockquote>
<p>Take the current value of <code>number</code>,<br>
add 1,<br>
and store the result back into <code>number</code>.</p>
</blockquote>

<p>No logic change.</p>

<p>Only structural simplification.</p>




<h2>
  
  
  3. Runnable Example — Increasing by 2
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">Program</span>
<span class="p">{</span>
    <span class="k">static</span> <span class="k">void</span> <span class="nf">Main</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">int</span> <span class="n">number</span> <span class="p">=</span> <span class="m">0</span><span class="p">;</span>

        <span class="k">while</span> <span class="p">(</span><span class="n">number</span> <span class="p">&lt;</span> <span class="m">10</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">number</span> <span class="p">+=</span> <span class="m">2</span><span class="p">;</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">number</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="s">"Finished."</span><span class="p">);</span>
        <span class="n">Console</span><span class="p">.</span><span class="nf">ReadKey</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>2
4
6
8
10
</code></pre>

</div>



<p>The loop progresses because the state moves forward.</p>




<h2>
  
  
  4. <code>+=</code> Also Works With Strings
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">Program</span>
<span class="p">{</span>
    <span class="k">static</span> <span class="k">void</span> <span class="nf">Main</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">string</span> <span class="n">text</span> <span class="p">=</span> <span class="s">"ab"</span><span class="p">;</span>
        <span class="n">text</span> <span class="p">+=</span> <span class="s">"cd"</span><span class="p">;</span>

        <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">text</span><span class="p">);</span> <span class="c1">// abcd</span>
        <span class="n">Console</span><span class="p">.</span><span class="nf">ReadKey</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Strings can accumulate values.</p>

<p>This is still state change.</p>

<p>Different type.</p>

<p>Same principle.</p>




<h2>
  
  
  5. Dangerous Code — Infinite Loop
</h2>

<p>Now the critical part.</p>

<h3>
  
  
  Incorrect Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System</span><span class="p">;</span>

<span class="k">class</span> <span class="nc">Program</span>
<span class="p">{</span>
    <span class="k">static</span> <span class="k">void</span> <span class="nf">Main</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="kt">int</span> <span class="n">number</span> <span class="p">=</span> <span class="m">0</span><span class="p">;</span>

        <span class="k">while</span> <span class="p">(</span><span class="n">number</span> <span class="p">&lt;</span> <span class="m">10</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">number</span> <span class="p">*=</span> <span class="m">2</span><span class="p">;</span>
            <span class="n">Console</span><span class="p">.</span><span class="nf">WriteLine</span><span class="p">(</span><span class="n">number</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>What happens?</p>

<p>The console keeps printing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>0
0
0
0
...
</code></pre>

</div>



<p>The program never stops.</p>




<h2>
  
  
  6. Why Did This Happen?
</h2>

<p>Initial state:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>number = 0
</code></pre>

</div>



<p>Inside the loop:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>number *= 2;
</code></pre>

</div>



<p>0 × 2 = 0</p>

<p>The state does not change.</p>

<p>Condition:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>0 &lt; 10 → always true
</code></pre>

</div>



<p>The loop never reaches a false condition.</p>

<p>This is an infinite loop.</p>




<h2>
  
  
  7. Definition of an Infinite Loop
</h2>

<blockquote>
<p>A loop whose condition never becomes false.</p>
</blockquote>

<p>The usual causes:</p>

<ul>
<li>No state change</li>
<li>Incorrect state change</li>
<li>Condition that can never fail</li>
</ul>

<p>Infinite loops are logic failures.</p>

<p>Not syntax failures.</p>




<h2>
  
  
  8. Fixing the Problem
</h2>

<p>Change the initial value:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">int</span> <span class="n">number</span> <span class="p">=</span> <span class="m">1</span><span class="p">;</span>
</code></pre>

</div>



<p>Now the flow becomes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1 → 2 → 4 → 8 → 16
</code></pre>

</div>



<p>When <code>number</code> becomes 16:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>16 &lt; 10 → false
</code></pre>

</div>



<p>The loop exits correctly.</p>

<p>Termination restored.</p>




<h2>
  
  
  9. The <code>++</code> Operator (Increment Operator)
</h2>

<p>This:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">number</span> <span class="p">+=</span> <span class="m">1</span><span class="p">;</span>
</code></pre>

</div>



<p>Can be shortened to:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">number</span><span class="p">++;</span>
</code></pre>

</div>



<p>It means:</p>

<blockquote>
<p>Increase <code>number</code> by 1.</p>
</blockquote>

<p>Same state change.</p>

<p>Cleaner syntax.</p>




<h2>
  
  
  10. Pre-Increment vs Post-Increment (Simplified for Now)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">number</span><span class="p">++;</span>
<span class="p">++</span><span class="n">number</span><span class="p">;</span>
</code></pre>

</div>



<p>Inside a loop body:</p>

<p>There is no visible difference.</p>

<p>The difference only matters when used inside expressions.</p>

<p>At this stage, treat them as equivalent for loop progression.</p>




<h2>
  
  
  11. The Structural Model of a Loop
</h2>

<p>Every loop requires three components:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td>Condition</td>
<td>Decides whether to continue</td>
</tr>
<tr>
<td>State variable</td>
<td>Value used inside condition</td>
</tr>
<tr>
<td>State change</td>
<td>Mechanism that eventually makes the condition false</td>
</tr>
</tbody>
</table></div>

<p>If one is missing:</p>

<ul>
<li>Infinite loop</li>
<li>Immediate termination</li>
<li>Logical inconsistency</li>
</ul>

<p>Loops are not magic.</p>

<p>They are structured repetition systems.</p>




<h2>
  
  
  12. A Critical Habit
</h2>

<p>Before writing any loop, ask:</p>

<blockquote>
<p>“When does this loop stop?”</p>
</blockquote>

<p>If you cannot answer that clearly,</p>

<p>Do not write the loop yet.</p>




<h2>
  
  
  13. Practice Exercises
</h2>

<p>Implement the following using <code>while</code>:</p>

<ol>
<li>Print numbers from 1 to 100</li>
<li>Increase by 2 each time</li>
<li>Print only multiples of 5</li>
<li>Count down from 100 to 0</li>
</ol>

<p>Each one forces you to think about:</p>

<ul>
<li>Condition</li>
<li>State</li>
<li>State change</li>
</ul>




<h2>
  
  
  Final Summary
</h2>

<ul>
<li>
<code>+=</code> accumulates</li>
<li>
<code>++</code> increments by one</li>
<li>Loops depend on state change</li>
<li>Infinite loops are state change failures</li>
</ul>

