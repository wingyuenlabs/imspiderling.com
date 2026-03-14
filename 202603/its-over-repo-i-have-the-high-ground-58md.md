---
Title: It's over, Repo. I have the high ground!
Description: 
Author: Anju Karanji
Date: 2026-03-14T21:41:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Your boyfriend has messed up. Royally. He knows it. You know it. The neighbor knows it. Everyone and their mother knows it!<br>
The words form in your head -<br>
<em>I. Told. You. So.</em><br>
And then, this other voice, "Don't be a dick, Anju" seemingly comes out of nowhere!<br>
And then, dang! You swallow the sentence.<br>
Congratulations. You’re basically Mother Teresa.<br>
Just... hotter.</p>

<p>Next scenario: The man cuts you off in traffic. Your hand is already on the horn.<br>
A very specific two-word insult involving his mother… is loading. <br>
And again: Don't be a dick, Anju.<br>
Hand moves away, curse dies in your throat. You’re Buddha baby! You are practically levitating.</p>

<p>Or when you’ve crafted the perfect insult.<br>
You know the kind: <em>Elegant</em>. Perfect. A gift from the universe. Frame-worthy! <br>
And right before you say it… you get a vision of future-you; rocking back and forth at 2:30 a.m., ruminating, feeling guilty. Forever. Until eternity.<br>
The insult is swallowed. The masterpiece lost to history 😭  <br>
Did I mention it already? Yes🙂‍↕️ - I am Jesus. (The Vatican has been notified.)  </p>

<p>It whispers… go on the date with that awkward, geeky classmate. He might be the next Jeff Bezos. Think of the future, Anju. <br>
My God, you short-sighted woman!</p>

<p>That voice? That sanctimonious, buzzkill voice? <br>
That's the linter for ya!</p>

<p>There are many flavors of linters.<br>
Golint is your strict asian mum. Checkstyle is your scary PE teacher from 3rd grade.  But today we’ll talking about Obi-Wan of the bunch - ESLint. Strict older brother energy. Caring but also kind and responsible.</p>

<p>ESLint runs as its own process. Loosely coupled from your IDE, like Obi-Wan living close to Luke but not exactly in the same part of the desert. </p>

<p>With every debounced keystroke, ESLint quietly parses your JavaScript into an AST. Basically, it takes your itty-bitty const x = 1 and turns it into this… A whole crime scene. Over one variable. Iconic! 😏<br>
</p>

<p> </p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="p">{</span>
  <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Program</span><span class="dl">"</span><span class="p">,</span>
  <span class="dl">"</span><span class="s2">body</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span>
      <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">VariableDeclaration</span><span class="dl">"</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">kind</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">const</span><span class="dl">"</span><span class="p">,</span>
      <span class="dl">"</span><span class="s2">declarations</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">VariableDeclarator</span><span class="dl">"</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Identifier</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">x</span><span class="dl">"</span> <span class="p">},</span>
          <span class="dl">"</span><span class="s2">init</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Literal</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">value</span><span class="dl">"</span><span class="p">:</span> <span class="mi">1</span> <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">}</span>
  <span class="p">]</span>
<span class="p">}</span><span class="err">  </span>
</code></pre>

</div>



<p>It walks that entire tree, checks every node for rule violations, and fires off context.report(); Which is just ESLint politely trying to say:</p>

<p>"Hey… maybe don’t be a dick."</p>

<p>ESLint does this with hundreds of rules. <br>
Today, we are not incarnating ESLint. Obi-Wan has seen wars. We are building the bite-sized baby Yoda. Adorable. But the Force? Very much present.</p>

<p>Our sabers: catch every rogue console.log left behind in production code.  </p>

<p>Here's the thing about our baby linter - he’s not very smart.  <br>
Basically: see the frog → eat the frog.</p>

<p>It visits every single node in the AST. The moment it spots a CallExpression that looks like console.log, it calls report().<br>
</p>

<p> </p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">walk</span><span class="p">(</span><span class="nx">node</span><span class="p">,</span> <span class="nx">visitor</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">node</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">node</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">object</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">visitor</span><span class="p">[</span><span class="nx">node</span><span class="p">.</span><span class="nx">type</span><span class="p">])</span> <span class="p">{</span>
    <span class="nx">visitor</span><span class="p">[</span><span class="nx">node</span><span class="p">.</span><span class="nx">type</span><span class="p">](</span><span class="nx">node</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">key</span> <span class="k">of</span> <span class="nb">Object</span><span class="p">.</span><span class="nf">keys</span><span class="p">(</span><span class="nx">node</span><span class="p">))</span> <span class="p">{</span>
    <span class="nf">walk</span><span class="p">(</span><span class="nx">node</span><span class="p">[</span><span class="nx">key</span><span class="p">],</span> <span class="nx">visitor</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nf">walk</span><span class="p">(</span><span class="nx">ast</span><span class="p">,</span> <span class="p">{</span>
  <span class="nc">CallExpression</span><span class="p">(</span><span class="nx">node</span><span class="p">)</span> <span class="p">{</span>
    <span class="nf">noConsoleLog</span><span class="p">(</span><span class="nx">node</span><span class="p">,</span> <span class="nx">report</span><span class="p">);</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Then we wrap it up as a CLI tool using <code>npm link</code>, point it at any repo, and let it loose. </p>

<p>That’s it! <a href="https://github.com/sagi0312/my-linter" rel="noopener noreferrer">Repo Link</a> <br>
You have your very own bite-sized conscience - ready to torture your repo!  </p>

<p>In a nutshell:</p>

<p>ESLint = Obi-Wan.<br>
Your repo = Anakin.<br>
Moral of the story = Obi-Wan(ESlint) stops Anakin from turning into Darth Vader!<br>
Battle cry = It's over, Anakin! I have the high ground!</p>

<p>I rest my saber!</p>

