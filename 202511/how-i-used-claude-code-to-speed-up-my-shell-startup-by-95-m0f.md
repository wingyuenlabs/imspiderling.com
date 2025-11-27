---
Title: How I Used Claude Code to Speed Up My Shell Startup by 95%
Description: 
Author: Nick Taylor
Date: 2025-11-27T21:22:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>My terminal was sluggish. Every time I opened a new tab, there was this annoying delay before I could start typing. I decided to dig into it, and with Claude Code's help, I went from a <strong>770ms startup time to just 40ms</strong>. That's a 19x improvement.</p>

<h2>
  
  
  The Problem
</h2>

<p>It wasn't that I accumulated too much tooling. Most things I have in my .zshrc, I needed, but each thing I tacked on added to my shell startup time. I honestly hadn't looked into this and just lived with it. Then, John Lindquist posted this the other day so I figured, let Claude Code speed it up for me.</p>

<p><iframe class="tweet-embed" id="tweet-1993037322984866168-791" src="https://platform.twitter.com/embed/Tweet.html?id=1993037322984866168">
</iframe>

  // Detect dark theme
  var iframe = document.getElementById('tweet-1993037322984866168-791');
  if (document.body.className.includes('dark-theme')) {
    iframe.src = "https://platform.twitter.com/embed/Tweet.html?id=1993037322984866168&amp;theme=dark"
  }



</p>

<h2>
  
  
  Measuring the Baseline
</h2>

<p>First thing I needed to know how bad it actually was:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">for </span>i <span class="k">in </span>1 2 3 4 5<span class="p">;</span> <span class="k">do</span> /usr/bin/time zsh <span class="nt">-i</span> <span class="nt">-c</span> <span class="nb">exit </span>2&gt;&amp;1<span class="p">;</span> <span class="k">done</span>
</code></pre>

</div>



<p>Yikes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>0.94 real
0.71 real
0.74 real
0.73 real
0.72 real
</code></pre>

</div>



<p><strong>Average: ~770ms per shell startup.</strong> Almost a full second just to open a terminal. Not good.</p>

<h2>
  
  
  What Was Slowing Things Down
</h2>

<p>Claude Code helped me identify the main culprits:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tool</th>
<th>Impact</th>
<th>Why It Sucks</th>
</tr>
</thead>
<tbody>
<tr>
<td>nvm</td>
<td>~300-500ms</td>
<td>Loads the entire Node.js environment every time</td>
</tr>
<tr>
<td>pyenv init</td>
<td>~100-200ms</td>
<td>Python version management initialization</td>
</tr>
<tr>
<td>security command</td>
<td>~50-100ms</td>
<td>Fetching API key from macOS Keychain</td>
</tr>
<tr>
<td>brew shellenv</td>
<td>~30-50ms</td>
<td>Runs a subshell to get Homebrew paths</td>
</tr>
<tr>
<td>gcloud completion</td>
<td>~20-30ms</td>
<td>Google Cloud completions</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  The Big Unlock: Lazy Loading
</h2>

<p>Most of the tools don't need to be loaded until I actually use them. So defer the expensive stuff until the first time I run a command.</p>

<h3>
  
  
  Self-Destructing Wrappers to the Rescue
</h3>

<p>This is the clever part. Thanks Claude Code. The wrapper functions remove themselves after first use:</p>

<ol>
<li>
<strong>First call</strong>: Wrapper runs, does the slow initialization, then deletes itself</li>
<li>
<strong>After that</strong>: Direct execution, zero overhead</li>
</ol>

<p>You only pay the cost once per session.</p>

<h3>
  
  
  nvm Lazy Loading
</h3>

<p>Instead of this slow startup code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Before: runs every shell startup (~400ms)</span>
<span class="o">[</span> <span class="nt">-s</span> <span class="s2">"/opt/homebrew/opt/nvm/nvm.sh"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="se">\.</span> <span class="s2">"/opt/homebrew/opt/nvm/nvm.sh"</span>
nvm use default <span class="nt">--silent</span>
</code></pre>

</div>



<p>now, I use wrapper functions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># After: only runs when you actually need node/npm/npx</span>
nvm<span class="o">()</span> <span class="o">{</span>
  <span class="nb">unset</span> <span class="nt">-f</span> nvm node npm npx
  <span class="o">[</span> <span class="nt">-s</span> <span class="s2">"/opt/homebrew/opt/nvm/nvm.sh"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="se">\.</span> <span class="s2">"/opt/homebrew/opt/nvm/nvm.sh"</span>
  <span class="o">[</span> <span class="nt">-s</span> <span class="s2">"/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="se">\.</span> <span class="s2">"/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"</span>
  nvm <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span>
<span class="o">}</span>
node<span class="o">()</span> <span class="o">{</span> nvm use default <span class="nt">--silent</span><span class="p">;</span> unfunction node npm npx<span class="p">;</span> node <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
npm<span class="o">()</span> <span class="o">{</span> nvm use default <span class="nt">--silent</span><span class="p">;</span> unfunction node npm npx<span class="p">;</span> npm <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
npx<span class="o">()</span> <span class="o">{</span> nvm use default <span class="nt">--silent</span><span class="p">;</span> unfunction node npm npx<span class="p">;</span> npx <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
</code></pre>

</div>



<p>The <code>unset -f</code> and <code>unfunction</code> commands remove the wrapper after first use. After that, it's like the tools were loaded normally.</p>

<h3>
  
  
  pyenv Gets the Same Treatment
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Before</span>
<span class="nb">eval</span> <span class="s2">"</span><span class="si">$(</span>pyenv init -<span class="si">)</span><span class="s2">"</span>

<span class="c"># After</span>
pyenv<span class="o">()</span> <span class="o">{</span>
  <span class="nb">unset</span> <span class="nt">-f</span> pyenv
  <span class="nb">eval</span> <span class="s2">"</span><span class="si">$(</span><span class="nb">command </span>pyenv init -<span class="si">)</span><span class="s2">"</span>
  pyenv <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span>
<span class="o">}</span>
</code></pre>

</div>



<p>First <code>pyenv</code> call takes ~150ms to initialize, then it's direct execution forever.</p>

<h3>
  
  
  Google Cloud SDK
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gcloud<span class="o">()</span> <span class="o">{</span>
  <span class="nb">unset</span> <span class="nt">-f</span> gcloud gsutil bq
  <span class="o">[</span> <span class="nt">-f</span> <span class="s1">'/Users/yolo/.local/google-cloud-sdk/path.zsh.inc'</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
    <span class="nb">.</span> <span class="s1">'/Users/yolo/.local/google-cloud-sdk/path.zsh.inc'</span>
  <span class="o">[</span> <span class="nt">-f</span> <span class="s1">'/Users/yolo/.local/google-cloud-sdk/completion.zsh.inc'</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
    <span class="nb">.</span> <span class="s1">'/Users/yolo/.local/google-cloud-sdk/completion.zsh.inc'</span>
  gcloud <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span>
<span class="o">}</span>
gsutil<span class="o">()</span> <span class="o">{</span> gcloud<span class="p">;</span> gsutil <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
bq<span class="o">()</span> <span class="o">{</span> gcloud<span class="p">;</span> bq <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span><span class="p">;</span> <span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  Homebrew Caching
</h3>

<p>Homebrew's environment doesn't change often, so just cache it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Before: subshell every time</span>
<span class="nb">eval</span> <span class="s2">"</span><span class="si">$(</span>/opt/homebrew/bin/brew shellenv<span class="si">)</span><span class="s2">"</span>

<span class="c"># After: cache to file, only regenerate if brew changes</span>
<span class="k">if</span> <span class="o">[[</span> <span class="o">!</span> <span class="nt">-f</span> ~/.zsh_brew_cache <span class="o">||</span> ~/.zsh_brew_cache <span class="nt">-ot</span> /opt/homebrew/bin/brew <span class="o">]]</span><span class="p">;</span> <span class="k">then</span>
  /opt/homebrew/bin/brew shellenv <span class="o">&gt;</span> ~/.zsh_brew_cache
<span class="k">fi
</span><span class="nb">source</span> ~/.zsh_brew_cache
</code></pre>

</div>



<h3>
  
  
  API Key Laziness
</h3>

<p>I was hitting the macOS Keychain on every shell startup:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Before: slow keychain lookup every time</span>
<span class="nb">export </span><span class="nv">OPENAI_API_KEY</span><span class="o">=</span><span class="si">$(</span>security find-generic-password <span class="nt">-a</span> <span class="nv">$USER</span> <span class="nt">-s</span> openai_api_key <span class="nt">-w</span><span class="si">)</span>
</code></pre>

</div>



<p>Since I only need this for npm stuff, I moved it into the npm wrapper:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm<span class="o">()</span> <span class="o">{</span>
  nvm use default <span class="nt">--silent</span>
  unfunction node npm npx
  <span class="o">[</span> <span class="nt">-z</span> <span class="s2">"</span><span class="nv">$OPENAI_API_KEY</span><span class="s2">"</span> <span class="o">]</span> <span class="o">&amp;&amp;</span> <span class="se">\</span>
    <span class="nb">export </span><span class="nv">OPENAI_API_KEY</span><span class="o">=</span><span class="si">$(</span>security find-generic-password <span class="nt">-a</span> <span class="nv">$USER</span> <span class="nt">-s</span> openai_api_key <span class="nt">-w</span><span class="si">)</span>
  npm <span class="s2">"</span><span class="nv">$@</span><span class="s2">"</span>
<span class="o">}</span>
</code></pre>

</div>



<p>More on the <a href="https://one-tip-a-week.beehiiv.com/p/one-tip-a-week-securely-load-secrets-from-your-keychain" rel="noopener noreferrer">macOS Keychain tip in my newsletter</a>.</p>

<h2>
  
  
  Other Quick Wins
</h2>

<p>I also cleaned up some basic stuff:</p>

<ul>
<li>Combined 7 different PATH modifications into one line</li>
<li>Removed duplicate <code>GPG_TTY</code> exports</li>
<li>Fixed ordering so <code>STARSHIP_CONFIG</code> gets set before <code>starship init</code>
</li>
</ul>

<h2>
  
  
  The Results
</h2>

<p>After all the changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">for </span>i <span class="k">in </span>1 2 3 4 5<span class="p">;</span> <span class="k">do</span> /usr/bin/time zsh <span class="nt">-i</span> <span class="nt">-c</span> <span class="nb">exit </span>2&gt;&amp;1<span class="p">;</span> <span class="k">done</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>0.06 real
0.04 real
0.04 real
0.03 real
0.04 real
</code></pre>

</div>



<p><strong>Average: ~40ms</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Before</th>
<th>After</th>
<th>Improvement</th>
</tr>
</thead>
<tbody>
<tr>
<td>770ms</td>
<td>40ms</td>
<td><strong>95% faster</strong></td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  The "Trade-off", Not Really
</h2>

<p>Yeah, there's a one-time cost when you first use each tool:</p>

<ul>
<li>First <code>node</code>/<code>npm</code>/<code>npx</code>: +400ms</li>
<li>First <code>pyenv</code>: +150ms
</li>
<li>First <code>gcloud</code>: +50ms</li>
</ul>

<p>But it's once per terminal session and honestly barely noticeable compared to what the commands actually do.</p>

<h2>
  
  
  Try It
</h2>

<p>If your shell is slow, first measure the total startup time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">time </span>zsh <span class="nt">-i</span> <span class="nt">-c</span> <span class="nb">exit</span>
</code></pre>

</div>



<p>If it's over 200ms, you've got room to improve. To see exactly what's slow, profile your .zshrc or whatever shell you're using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Add to top of .zshrc</span>
zmodload zsh/zprof

<span class="c"># Add to bottom</span>
zprof
</code></pre>

</div>



<p>This breaks down which specific commands are eating up your startup time.</p>

<h2>
  
  
  Wrapping Up
</h2>

<p>Dev tooling adds up fast and it's easy to not notice the death by a thousand paper cuts. This lazy loading pattern fixes it without any real downsides.</p>

<p>Big ups to John L. and Claude Code for helping me figure out the bottlenecks and solutions.</p>

<p>Note: The performance impact numbers and comparison table were generated with Claude Code's help. Mileage may vary depending on your specific setup. 😅</p>

<p>If you want to stay in touch, all my socials are on <a href="https://nickyt.online" rel="noopener noreferrer">nickyt.online</a>.</p>

<p>Until the next one!</p>

<p>Photo by <a href="https://unsplash.com/@marcsm?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText" rel="noopener noreferrer">Marc Sendra Martorell</a> on <a href="https://unsplash.com/photos/time-lapsed-of-street-lights--Vqn2WrfxTQ?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText" rel="noopener noreferrer">Unsplash</a></p>

