---
Title: The Simplest Way to Make Just Interactive: Use --choose
Description: 
Author: Romain Lespinasse
Date: 2026-02-20T22:02:50.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The "Native" Way
</h2>

<p>While you <em>can</em> manually pipe <code>just --list</code> into <code>fzf</code>, <code>just</code> has a built-in feature that handles this natively.<br>
If you have <code>fzf</code> installed, you don't need a custom recipe at all.</p>
<h3>
  
  
  The One-Liner
</h3>

<p>Simply run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>just <span class="nt">--choose</span>
</code></pre>

</div>



<p>This command automatically parses your recipes, opens <code>fzf</code>, and executes your selection once you hit Enter.</p>

<h2>
  
  
  Level Up with Shell Aliases
</h2>

<p>If you find yourself running specific types of tasks frequently, you can combine <code>just --choose</code> with a shell alias.<br>
This is perfect for filtering down to specific "namespaces" if you use a prefix naming convention (e.g., <code>docker-build</code>, <code>docker-up</code>).</p>

<p>Add this to your <code>.zshrc</code> or <code>.bashrc</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># General interactive just</span>
<span class="nb">alias </span><span class="nv">j</span><span class="o">=</span><span class="s1">'just --choose'</span>

<span class="c"># Interactive just filtered for Docker tasks only</span>
<span class="nb">alias </span><span class="nv">jd</span><span class="o">=</span><span class="s1">'just --choose --chooser "fzf --query=docker"'</span>
</code></pre>

</div>



<h2>
  
  
  Customizing the UI
</h2>

<p>You can customize the look and feel of the chooser globally by setting the <code>JUST_CHOOSER</code> environment variable.<br>
This allows you to add colors or change the layout without touching your <code>justfile</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">JUST_CHOOSER</span><span class="o">=</span><span class="s1">'fzf --header "⚡ Select Task" --height 40% --layout reverse --border'</span>
</code></pre>

</div>



<h2>
  
  
  Why this is better:
</h2>

<ol>
<li>
<strong>Zero Boilerplate:</strong> You don't need to pollute your <code>justfile</code> with a "menu" recipe.</li>
<li>
<strong>Clean Output:</strong> It respects docstrings and ignores internal recipes (those starting with an underscore).</li>
<li>
<strong>Speed:</strong> It’s a single binary call rather than a chain of piped commands.</li>
</ol>

<h2>
  
  
  The "One-Touch" Justfile
</h2>

<p>If you want the interactive menu to appear by default when you just type <code>just</code> in your terminal, add this to the top of your file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Run the interactive chooser by default
default:
    @just --choose
</code></pre>

</div>



<h2>
  
  
  Conclusion
</h2>

<p>Ultimately, using <code>just --choose</code> transforms your justfile from a static list of commands into a dynamic, user-friendly CLI.</p>

<p>By moving away from manual piping and embracing built-in flags and environment variables, you reduce maintenance overhead while significantly improving your daily terminal workflow.</p>

<p>Whether you’re automating a complex Docker environment or just looking for a faster way to trigger local builds, making your tools interactive by default ensures that your productivity stays as high as your command-line efficiency.</p>

<blockquote>
<p>💡 Note<br>
This post was originally published on <a href="https://www.romainlespinasse.dev/posts/interactive-just-recipes/" rel="noopener noreferrer">my personal blog</a>.</p>
</blockquote>

