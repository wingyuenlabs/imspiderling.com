---
Title: I Built a Module System for a Language That Doesn't Have One
Description: 
Author: Claudia Nadalin
Date: 2025-12-27T21:26:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>You know what's funny about PineScript? It's 2025 and we're writing trading indicators like it's 1995. One file. Everything global. No modules. No imports. Just you, your code, and the slow descent into madness.</p>

<p>I'd been putting up with this for a while. I had a fairly complex indicator that used code I'd split across multiple TradingView libraries. Seemed like the right thing to do at the time. Separate concerns. Keep things clean. Very professional.</p>

<p>Then I needed to change one function.</p>

<h2>
  
  
  The Workflow From Hell
</h2>

<p>Here's what updating a library function looked like:</p>

<ol>
<li>Edit the code locally</li>
<li>Push to git</li>
<li>Copy-paste into TradingView's editor</li>
<li>Republish the library</li>
<li>Note the new version number</li>
<li>Open my indicator script</li>
<li>Update the import statement with the new version</li>
<li>Save that</li>
<li>Push that to git</li>
<li>Copy-paste into TradingView's editor</li>
<li>Save again</li>
</ol>

<p>And that's for ONE library. I had five.</p>

<p>You ever see that Seinfeld episode where George tries to do the opposite of every instinct he has? That's what this workflow felt like, except I wasn't getting a job with the Yankees at the end of it. I was just trying to fix a bug.</p>

<p>There had to be another way.</p>

<h2>
  
  
  Surely Someone's Solved This
</h2>

<p>I started researching. Googled things like "PineScript bundler", "PineScript multiple files", "PineScript module system".</p>

<p>I found:</p>

<ul>
<li>VS Code extensions for syntax highlighting (cool, but not what I need)</li>
<li>PyneCore/PyneSys, which transpile PineScript to Python for running backtests locally (interesting, but different problem)</li>
<li>A lot of forum posts from people with the same frustration and no solutions</li>
</ul>

<p>What I didn't find was a bundler. Nobody had built the thing I wanted.</p>

<p>But even before searching, I knew why this was hard. I knew what any solution would require. And it wasn't pretty.</p>

<h2>
  
  
  Why This Is Actually Hard
</h2>

<p>JavaScript has something called scope. When Webpack bundles your code, it can wrap each module in a function and they're isolated from each other:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">var</span> <span class="nx">__module_1__</span> <span class="o">=</span> <span class="p">(</span><span class="nf">function </span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">function</span> <span class="nf">double</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">x</span> <span class="o">*</span> <span class="mi">2</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="p">{</span> <span class="nx">double</span> <span class="p">};</span>
<span class="p">})();</span>

<span class="kd">var</span> <span class="nx">__module_2__</span> <span class="o">=</span> <span class="p">(</span><span class="nf">function </span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">function</span> <span class="nf">double</span><span class="p">(</span><span class="nx">x</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">x</span> <span class="o">+</span> <span class="mi">100</span><span class="p">;</span>
  <span class="p">}</span> <span class="c1">// No collision - different scope</span>
  <span class="k">return</span> <span class="p">{</span> <span class="nx">double</span> <span class="p">};</span>
<span class="p">})();</span>
</code></pre>

</div>



<p>Two functions named <code>double</code>, no problem. They live in different scopes. They can't see each other. Webpack leverages this to keep modules isolated.</p>

<p>PineScript doesn't have this.</p>

<p>There are no closures. No way to create isolated scopes. Everything is basically global. If you define <code>double</code> in one file and <code>double</code> in another file, then smash those files together, you've got a collision. One overwrites the other. Your code breaks.</p>

<p>So any PineScript bundler would need to rename things. Take the <code>double</code> function from <code>utils/math.pine</code> and rename it to something like <code>__utils_math__double</code>. Do the same for every export from every file. Update all the references. No collisions.</p>

<p>I knew this was the path before I wrote a single line of code. The question was: how do you actually rename things in code reliably?</p>

<h2>
  
  
  The Find/Replace Trap
</h2>

<p>My first instinct was simple: just use find/replace. Find <code>double</code>, replace with <code>__utils_math__double</code>. Done.</p>

<p>Here's why that falls apart immediately. Imagine this code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>double(x) =&gt;
    x * 2

myLabel = "Call double() for twice the value"
doubleCheck = true
plot(double(close), title="double")
</code></pre>

</div>



<p>You want to rename the <code>double</code> function to <code>__utils_math__double</code>.</p>

<p>With find/replace, you'd get:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>__utils_math__double(x) =&gt;
    x * 2

myLabel = "Call __utils_math__double() for twice the value"  // Broken - changed string content
__utils_math__doubleCheck = true                              // Broken - renamed wrong variable
plot(__utils_math__double(close), title="__utils_math__double")   // Broken - changed string content
</code></pre>

</div>



<p>You've corrupted string literals and renamed a completely unrelated variable that happened to start with "double".</p>

<p>Find/replace sees code as a flat string of characters. It has no idea that <code>"double"</code> inside quotes is a string, not a function reference. It can't tell that <code>doubleCheck</code> is a different identifier. It just sees the letters d-o-u-b-l-e and does its thing.</p>

<p>I needed a way to understand the <em>structure</em> of the code. To know that this <code>double</code> is a function definition, that <code>double</code> is a function call, and that <code>"double"</code> is just text inside a string that should be left alone.</p>

<p>This is where I got stuck. Building something that truly understands PineScript's structure would take months. Parsing is hard. PineScript has weird indentation rules, line continuation, type inference, all sorts of quirks.</p>

<p>Then I found pynescript.</p>

<h2>
  
  
  The Missing Piece
</h2>

<p>pynescript is a Python library that parses PineScript into something called an Abstract Syntax Tree (AST), and can unparse it back into PineScript.</p>

<p>If "Abstract Syntax Tree" sounds intimidating, don't worry. It's actually a simple concept. An AST is basically the DOM, but for code instead of HTML.</p>

<p>When your browser gets HTML, it doesn't keep it as a string. It parses it into a tree structure where each node knows what it is. A <code>&lt;div&gt;</code>. A <code>&lt;p&gt;</code>. A text node. You can manipulate these nodes programmatically - add classes, change text, move things around - without doing string manipulation on raw HTML.</p>

<p>An AST does the same thing for code. When you parse this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>double(x) =&gt;
    x * 2

myLabel = "Call double()"
</code></pre>

</div>



<p>You get a tree like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Script
├── FunctionDefinition
│   ├── name: "double"           ← I'm a function definition
│   ├── parameters: ["x"]
│   └── body: BinaryOp(x * 2)
│
└── Assignment
    ├── target: "myLabel"
    └── value: StringLiteral     ← I'm a string, leave me alone
        └── "Call double()"
</code></pre>

</div>



<p>Now renaming becomes surgical. You say: "Find all nodes of type <code>FunctionDefinition</code> where name equals <code>double</code>. Rename those. Find all nodes of type <code>FunctionCall</code> where the function is <code>double</code>. Rename those too. Leave <code>StringLiteral</code> nodes alone."</p>

<p>The string content is untouched because it's a <code>StringLiteral</code> node, not a <code>FunctionCall</code> node. <code>doubleCheck</code> is untouched because it's a completely different identifier node. The structure tells you what's what.</p>

<p>pynescript had already done the hard work of parsing PineScript correctly. Months of work with ANTLR (a serious parser generator). I could just <code>pip install pynescript</code> and get access to that tree.</p>

<p>I had my strategy (rename with prefixes), and now I had my tool (AST manipulation via pynescript). Time to see if it actually worked.</p>

<h2>
  
  
  The Spike
</h2>

<p>Before committing to building a full tool, I wanted to prove the concept. Take two PineScript files, parse them, rename stuff in the AST, merge them, unparse, see if TradingView accepts the output.</p>

<p>I created two tiny files:</p>

<p><strong>math_utils.pine:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>//@version=5
// @export double

double(x) =&gt;
    x * 2
</code></pre>

</div>



<p><strong>main.pine:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>//@version=5
// @import { double } from "./math_utils.pine"

indicator("Test", overlay=true)

result = double(close)
plot(result)
</code></pre>

</div>



<p>Then wrote some Python to:</p>

<ol>
<li>Parse both files with pynescript</li>
<li>Find the function definition for <code>double</code> in the AST</li>
<li>Rename it to <code>__math_utils__double</code>
</li>
<li>Find references to <code>double</code> in main.pine's AST</li>
<li>Update them to <code>__math_utils__double</code>
</li>
<li>Merge and unparse</li>
</ol>

<p>I ran it. Out came:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>//@version=5
indicator("Test", overlay=true)

__math_utils__double(x) =&gt;
    x * 2

result = __math_utils__double(close)
plot(result)
</code></pre>

</div>



<p>I pasted it into TradingView.</p>

<p>It worked.</p>

<p>I sat there for a second, genuinely surprised. Not because it was magic, but because I'd expected to hit some weird edge case or parser limitation that would kill the whole idea. But no. It just... worked.</p>

<h2>
  
  
  Building the Real Thing
</h2>

<p>With the spike working, I built out the full tool:</p>

<ul>
<li>A CLI (<code>pinecone build</code>, <code>pinecone build --watch</code>)</li>
<li>Config file support</li>
<li>Proper dependency graph construction</li>
<li>Topological sorting (so dependencies come before the code that uses them)</li>
<li>Error messages that point to your original files, not the bundled output</li>
<li>A <code>--copy</code> flag that puts the output straight into your clipboard</li>
</ul>

<p>The module syntax uses comments so that unbundled code doesn't break if you accidentally paste it into TradingView:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// @import { customRsi } from "./indicators/rsi.pine"
// @export myFunction
</code></pre>

</div>



<p>TradingView's parser just sees these as comments and ignores them. PineCone sees them as directives.</p>

<p>The prefix strategy uses the file path. <code>src/utils/math.pine</code> becomes <code>__utils_math__</code>. Every exported function and variable from that file gets that prefix. Every reference to those exports gets updated.</p>

<p>It's not elegant. Your bundled output has ugly names like <code>__indicators_rsi__customRsi</code>. But it works. TradingView accepts it. There are no collisions. And you never have to look at the bundled output anyway - it's just an intermediate artifact, like compiled code.</p>

<h2>
  
  
  What Did I Actually Build?
</h2>

<p>I've been calling it "Webpack for PineScript" but that's not quite right. Webpack does a lot of stuff: code splitting, lazy loading, tree shaking, hot module replacement.</p>

<p>PineCone does one thing: it lets you write PineScript across multiple files with a module system, and compiles it down to a single file TradingView can understand.</p>

<p>It's a module system and bundler for a language that has neither.</p>

<p>PineCone doesn't add features to PineScript. It doesn't make your indicators faster. It doesn't connect to your broker. It just lets you organize your code like a normal human being in 2025.</p>

<p>And honestly? That's enough. That's the thing I needed.</p>

<h2>
  
  
  The Takeaway
</h2>

<p>If you're working with a language that's missing something fundamental, you might not be stuck. Look for parsers. Look for AST tools. If someone's already done the work of understanding the language's structure, you can build on top of that.</p>

<p>pynescript didn't solve my problem directly. It's a parser, not a bundler. But it solved the hard part - correctly parsing PineScript - and let me focus on the part I actually cared about: the module system.</p>

<p>Anyway, Pinecone is open source. If you're a PineScript developer who's ever felt the pain of managing multiple libraries, give it a try. And if you've got ideas for making it better, I'm all ears.</p>

<p>Serenity now.</p>




<p><em>Pinecone is available <a href="https://claudianadalin.github.io/pinecone" rel="noopener noreferrer">here</a>. Built with <a href="https://pypi.org/project/pynescript/" rel="noopener noreferrer">pynescript</a>, which you should also check out if you're doing anything programmatic with PineScript.</em></p>




<p>Claudia is a Frontend Developer. You can find more of her work <a href="https://www.claudianadalin.com/" rel="noopener noreferrer">here</a>.</p>

