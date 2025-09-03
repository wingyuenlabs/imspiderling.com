---
Title: Code Wars - 6kyu 'Split Strings' solution in Python
Description: 
Author: claudiogiubrone
Date: 2025-09-03T21:26:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hello wonderful people,</p>

<p>As a beginner programmer, I've started putting into practice what I'm learning by solving problems on platforms like Codewars.</p>

<p>To help myself while studying, I thought of documenting my mental process toward the solutions. Later, I realized that sharing my experiences and the approaches I take to these problems could be useful for others on their own learning journey.</p>

<p>This guide provides a detailed breakdown of the thought process and implementation for solving the Code Wars kata "<a href="https://www.codewars.com/kata/515de9ae9dcfc28eb6000001/python" rel="noopener noreferrer">Split Strings"</a> (level 6kyu) coding challenge.</p>

<p>It is designed to help new programmers understand how to approach and solve a common problem, focusing on logical steps and robust code.</p>




<h3>
  
  
  <strong>The Problem</strong>
</h3>

<p>The task is to write a function that takes a string and splits it into a list of two-character pairs. If the string contains an odd number of characters, the final pair must be completed by adding an underscore (<code>_</code>).</p>

<ul>
<li>
<strong>Example 1</strong>: The string <code>'abc'</code> should be transformed into <code>['ab', 'c_']</code>.</li>
<li>
<strong>Example 2</strong>: The string <code>'abcdef'</code> should become <code>['ab', 'cd', 'ef']</code>.</li>
</ul>




<h3>
  
  
  <strong>Step 1: Handling the Edge Case (Odd-Length Strings)</strong>
</h3>

<p>The most important requirement is handling strings with an odd number of characters. A simple check at the beginning of the function can solve this. The modulo operator (<code>%</code>) is perfect for this, as it returns the remainder of a division. If the length of the string divided by 2 has a remainder of 1 (<code>len(s) % 2 != 0</code>), the string's length is odd. In this case, an underscore should be appended to the string. This ensures that the main logic of the function can always work with an even-length string.</p>

<h3>
  
  
  <strong>Step 2: Iterating and Slicing the String</strong>
</h3>

<p>The core of the solution requires processing the string in chunks of two characters. A <code>for</code> loop is ideal for this. The <code>range()</code> function can be used with a third argument, <code>step</code>, to control the loop's increment. By setting the step to <code>2</code>, the loop's index <code>i</code> will jump from 0 to 2 to 4 and so on, allowing us to capture two characters at a time.</p>

<p>Within the loop, <strong>string slicing</strong> <code>s[i:i+2]</code> is used. This operation returns a substring starting from index <code>i</code> up to (but not including) index <code>i+2</code>, effectively grabbing a two-character pair.</p>

<h3>
  
  
  <strong>Step 3: Storing the Results</strong>
</h3>

<p>As each pair is generated, it needs to be stored in a collection. A <strong>list</strong> is the perfect data structure for this task. The process is straightforward:</p>

<ol>
<li> Initialize an empty list before the loop.</li>
<li> Use the <code>.append()</code> method to add each two-character slice to the list during each iteration of the loop.</li>
<li> After the loop completes, the function should return the final list.</li>
</ol>




<h3>
  
  
  <strong>The Final Code</strong>
</h3>

<p>Combining all the steps above results in a concise and robust function. The code is designed to be self-contained and ready for use in a test environment, where the platform handles providing the input to the function directly.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">solution</span><span class="p">(</span><span class="n">s</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    Splits a string into a list of character pairs.
    If the string has an odd number of characters, the last pair is padded with an underscore.
    </span><span class="sh">"""</span>
    <span class="n">ls</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="c1"># Step 1: Handle odd-length strings by appending an underscore.
</span>    <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">s</span><span class="p">)</span> <span class="o">%</span> <span class="mi">2</span> <span class="o">!=</span> <span class="mi">0</span><span class="p">:</span>
        <span class="n">s</span> <span class="o">+=</span> <span class="sh">'</span><span class="s">_</span><span class="sh">'</span>

    <span class="c1"># Step 2: Loop through the string in steps of 2.
</span>    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nf">len</span><span class="p">(</span><span class="n">s</span><span class="p">),</span> <span class="mi">2</span><span class="p">):</span>
        <span class="c1"># Step 3: Slice and append each pair to the list.
</span>        <span class="n">ls</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">s</span><span class="p">[</span><span class="n">i</span><span class="p">:</span><span class="n">i</span><span class="o">+</span><span class="mi">2</span><span class="p">])</span>

    <span class="k">return</span> <span class="n">ls</span>
</code></pre>

</div>






<h3>
  
  
  <strong>Conclusion</strong>
</h3>

<p>I have also create a repo on github. <br>
You can see if it can help you at this <a href="https://github.com/claudiogiubrone/code_wars_solutions_python.git" rel="noopener noreferrer">link</a>.</p>

