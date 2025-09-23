---
Title: 🔍 Applying Flawfinder: A Lightweight SAST Tool to Secure C/C++ Codebases
Description: 
Author: JEFFERSON ROSAS CHAMBILLA
Date: 2025-09-23T21:59:34.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Introduction: Why SAST for C/C++?</strong></p>

<p>Static Application Security Testing (SAST) is a foundational practice in modern secure software development. Unlike dynamic or runtime analysis, SAST examines source code without executing it, identifying vulnerabilities early in the development lifecycle when they are cheapest and easiest to fix.</p>

<p>While enterprise-grade tools often dominate the conversation, lightweight, open-source alternatives like <strong>Flawfinder</strong> offer tremendous value. This is especially true for teams working with C/C++, embedded systems, or legacy codebases where simplicity, speed, and zero cost are critical.</p>

<p>In this article, we'll apply Flawfinder to a sample C++ program, interpret its findings, and discuss how to integrate it into a development workflow.</p>

<p><strong>Why Flawfinder?</strong></p>

<p>Flawfinder is a command-line static analysis tool designed specifically for C and C++. Created by David A. Wheeler, it scans source code for calls to functions known to be risky (like <code>strcpy</code>, <code>gets</code>, <code>sprintf</code>) which commonly lead to:</p>

<ul>
<li>  <strong>Buffer overflows (CWE-120)</strong>
</li>
<li>  <strong>Format string vulnerabilities (CWE-134)</strong>
</li>
<li>  <strong>Command injection (CWE-78)</strong>
</li>
</ul>

<p>Instead of performing complex abstract syntax tree (AST) analysis, Flawfinder uses simple but effective pattern-matching against a curated database of dangerous function calls. This makes it:</p>

<p>✅ <strong>Blazingly fast</strong> - Scans thousands of lines of code in seconds.<br>
✅ <strong>Lightweight</strong> - No compilation or complex setup required.<br>
✅ <strong>Free and Open-Source</strong> - No licensing fees.<br>
✅ <strong>Beginner-friendly</strong> - Easy to install and interpret results.</p>

<p>And as requested, it's <strong>not</strong> SonarQube, Snyk, Semgrep, or Veracode.</p>

<p><strong>Hands-On: Scanning a C++ Project</strong></p>

<p><strong>Step 1: Installation</strong></p>

<p>Installing Flawfinder is straightforward. It's available via package managers for most Linux distributions and macOS.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># On Ubuntu/Debian</span>
<span class="nb">sudo </span>apt-get <span class="nb">install </span>flawfinder

<span class="c"># On macOS using Homebrew</span>
brew <span class="nb">install </span>flawfinder

<span class="c"># Using pip (Python Package Manager)</span>
pip <span class="nb">install </span>flawfinder
</code></pre>

</div>



<p>Verify the installation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>flawfinder <span class="nt">--version</span>
</code></pre>

</div>



<p><strong>Step 2: Create a Sample C++ File to Scan</strong></p>

<p>Let's create a simple C++ file (<code>vulnerable_example.cpp</code>) that contains some common security pitfalls. This will give Flawfinder something interesting to report.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// vulnerable_example.cpp</span>
<span class="cp">#include</span> <span class="cpf">&lt;iostream&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;cstring&gt;</span><span class="cp">
#include</span> <span class="cpf">&lt;cstdio&gt;</span><span class="cp">
</span>
<span class="kt">int</span> <span class="nf">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="kt">char</span> <span class="n">src</span><span class="p">[</span><span class="mi">10</span><span class="p">]</span> <span class="o">=</span> <span class="s">"Hello"</span><span class="p">;</span>
    <span class="kt">char</span> <span class="n">dest</span><span class="p">[</span><span class="mi">5</span><span class="p">];</span> <span class="c1">// Oops! Too small.</span>

    <span class="c1">// 1. Risky function: strcpy (potential buffer overflow)</span>
    <span class="n">strcpy</span><span class="p">(</span><span class="n">dest</span><span class="p">,</span> <span class="n">src</span><span class="p">);</span>

    <span class="c1">// 2. Risky function: sprintf (potential buffer overflow)</span>
    <span class="kt">char</span> <span class="n">buffer</span><span class="p">[</span><span class="mi">10</span><span class="p">];</span>
    <span class="n">sprintf</span><span class="p">(</span><span class="n">buffer</span><span class="p">,</span> <span class="s">"The number is %d"</span><span class="p">,</span> <span class="mi">42</span><span class="p">);</span>

    <span class="c1">// 3. Risky function: gets (extremely dangerous)</span>
    <span class="c1">// char input[50];</span>
    <span class="c1">// gets(input); // Uncommenting this would be a major flaw!</span>

    <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Dest: "</span> <span class="o">&lt;&lt;</span> <span class="n">dest</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>
    <span class="n">std</span><span class="o">::</span><span class="n">cout</span> <span class="o">&lt;&lt;</span> <span class="s">"Buffer: "</span> <span class="o">&lt;&lt;</span> <span class="n">buffer</span> <span class="o">&lt;&lt;</span> <span class="n">std</span><span class="o">::</span><span class="n">endl</span><span class="p">;</span>

    <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Step 3: Run the Scan</strong></p>

<p>Navigate to the directory containing your source file and run Flawfinder. You can point it to a single file or an entire directory.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Scan a single file</span>
flawfinder vulnerable_example.cpp

<span class="c"># Scan the current directory and all subdirectories</span>
flawfinder ./
</code></pre>

</div>



<p><strong>Step 4: Interpreting the Results</strong></p>

<p>Flawfinder's output is color-coded and ranked by risk level (0-5, where 5 is the most severe). Here's what you might see for our example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Flawfinder version 2.0.19, (C) 2001-2019 David A. Wheeler.
Number of rules (primarily dangerous function names) in C/C++ ruleset: 220
Examining vulnerable_example.cpp

vulnerable_example.cpp:10: [4] (buffer) strcpy:
  Does not check for buffer overflows when copying to destination (CWE-120).
  Consider using strncpy or snprintf.

vulnerable_example.cpp:14: [4] (buffer) sprintf:
  Does not check for buffer overflows when writing to destination (CWE-120).
  Use snprintf or vsnprintf instead.

vulnerable_example.cpp:18: [5] (buffer) gets:
  This function is extremely dangerous because it may overflow the
  calling buffer. It is obsoleted by ISO/IEC 9899:1999 (C99) and
  should never be used. Use fgets() instead.

FINAL RESULTS:

Hits = 3
Lines analyzed = 20 in 0.1 seconds (200 lines/second)
3 Physical Source Lines of Code (SLOC) = 20
Hits@level = [0]   0 [1]   0 [2]   0 [3]   0 [4]   2 [5]   1
Hits@level+ = [0+]   3 [1+]   3 [2+]   3 [3+]   3 [4+]   3 [5+]   1
</code></pre>

</div>



<p><strong>Key takeaways from the report:</strong></p>

<ul>
<li>  <strong>Location:</strong> Each finding shows the file and line number.</li>
<li>  <strong>Risk Level:</strong> <code>[4]</code> and <code>[5]</code> indicate high-severity issues.</li>
<li>  <strong>Description:</strong> A brief explanation of the risk and the associated CWE.</li>
<li>  <strong>Recommendation:</strong> Suggests safer alternative functions.</li>
</ul>

<p><strong>Integrating Flawfinder into Your Workflow</strong></p>

<p>To move beyond one-off scans, you can integrate Flawfinder directly into your development process.</p>

<p><strong>1. Make it Part of Your CI/CD Pipeline (e.g., GitHub Actions)</strong></p>

<p>You can create a simple GitHub Action to run Flawfinder on every push or pull request. Create a file in your repo at <code>.github/workflows/flawfinder.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">SAST with Flawfinder</span>
<span class="na">on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">push</span><span class="pi">,</span> <span class="nv">pull_request</span><span class="pi">]</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">sast</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout code</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Install Flawfinder</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">sudo apt-get install -y flawfinder</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Run Flawfinder</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">flawfinder --sarflib ./</span>
</code></pre>

</div>



<p>This will output the results in SARIF format, which GitHub can natively display in the "Security" tab.</p>

<p><strong>2. Customizing Scans with Command-Line Options</strong></p>

<p>Flawfinder offers several options to tailor the output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Only show high-risk issues (level 4 and 5)</span>
flawfinder <span class="nt">--minlevel</span> 4 ./

<span class="c"># Generate a HTML report</span>
flawfinder <span class="nt">--html</span> <span class="nt">--output</span> results.html ./

<span class="c"># Suppress specific known issues (e.g., a false positive on line 10)</span>
flawfinder <span class="nt">--falsepositive</span> vulnerable_example.cpp:10
</code></pre>

</div>



<p><strong>Limitations and Considerations</strong></p>

<p>While Flawfinder is excellent for catching known dangerous patterns, it's important to understand its limitations:</p>

<ul>
<li>  <strong>False Positives:</strong> Pattern-matching can flag safe usage of functions</li>
<li>  <strong>Limited Scope:</strong> Only finds issues related to its built-in database</li>
<li>  <strong>No Data Flow Analysis:</strong> Cannot track variables across complex function calls</li>
</ul>

<p>For mission-critical projects, consider using Flawfinder alongside more advanced tools like Clang Static Analyzer or commercial solutions.</p>

<p><strong>Conclusion: Flawfinder's Place in Your Toolbox</strong></p>

<p>Flawfinder is not a silver bullet. Its pattern-matching approach can produce false positives and it won't catch complex logical flaws. However, as a fast, free, and focused tool, it is incredibly effective at what it does: <strong>catching obvious yet dangerous function calls in C/C++ code.</strong></p>

<p>It serves as an excellent first line of defense, especially for developers new to secure coding practices. By integrating it early and often, you can prevent well-known vulnerability classes from ever reaching production.</p>

