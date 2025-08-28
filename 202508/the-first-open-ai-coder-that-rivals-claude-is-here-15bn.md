---
Title: The first open AI coder that rivals Claude is here
Description: 
Author: <devtips/>
Date: 2025-08-28T21:45:06.000Z
Robots: noindex,nofollow
Template: index
---
<p><span></span></p>
<h2 id="6281">Quen 3 Coder isn’t just another model it’s an agentic coding monster that might change the game for devs and AI labs alike.</h2>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2ALkVoiTYJwpfDQPWcCuUTEA.jpeg" width="800" height="800"><h2 id="88a9">Programmers, you might want to sit down for this</h2>
<p id="cd84">If you’ve been chilling comfortably under the warm blanket of Claude 4’s dominance in AI-assisted coding… wake up. A Chinese company just threw a firecracker under your office chair.</p>
<p id="c0dd">Alibaba has released <strong>Quen 3 Coder</strong>, and it’s not just another model. It’s an <strong>openweight</strong>, <strong>agentic</strong>, <strong>long-horizon</strong>, <strong>multi-trillion-token-trained monster</strong> that can literally run, test, and debug your code… from the command line.</p>
<p id="2249">Yep. CLI-native. Open access. Claude-level benchmarks.</p>
<p id="09be">And if that didn’t make you clench your keyboard a little tighter this thing was trained with reinforcement learning in <strong>20,000 parallel environments</strong>, has a <strong>256k–1M token context window</strong>, and shows <strong>benchmark scores near or above GPT-4.1 and Claude 4</strong>.</p>
<p id="8d3e">Oh and it’s <strong>open</strong>. 🤯</p>
<p id="7fb0">If you’re a dev, it’s hard not to both admire and fear it.</p>
<p id="c273">We’ll unpack what this new model actually is, how it works, what it means for the coding world, and whether it’s even worth running if you don’t have access to a GPU farm and an oil tycoon’s electricity budget.</p>
<h2 id="eb8a">Table of contents</h2>
<ol>

<li id="d571">Why devs should care</li>

<li id="d1f0">Meet Quen 3 Coder: China’s openweight coding beast</li>

<li id="734b">How it was trained: 20,000 coders in a digital bootcamp</li>

<li id="892f">The new Quen CLI: AI agent goes terminal mode</li>

<li id="0326">Context window madness: Hold your whole startup</li>

<li id="6c52">But can you run it locally? Lol no.</li>

<li id="6d12">So, can it dethrone Claude?</li>

<li id="f6d2">The plot twist: OpenAI vs. Google vs. China</li>

<li id="694d">Conclusion + Final thoughts</li>

</ol>
<span></span><span></span><span></span><h2 id="f783">1. Why devs should care</h2>
<p id="47ee">Let’s be real. A lot of AI announcements feel like hype cycles on autopilot.<br>Another model, another claim, another benchmark. Yawn.</p>
<p id="08be">But Quen 3 Coder? This one deserves your full attention.</p>
<p id="ba43">Because it’s not just about better autocomplete. It’s about <strong>AI agents actively writing, executing, and validating your code without you.</strong> And it’s open.</p>
<p id="c5df">Think about that for a second.</p>
<p id="55ec">This isn’t just GitHub Copilot giving you a gentle nudge on function names.<br>This is a model that can spin up in a terminal, test outputs, fix its own mistakes, and wrap up before your coffee cools.</p>
<p id="389c"><strong>So if you’re a:</strong></p>
<ul>

<li id="ef8e">

<strong>Solo dev:</strong> It’s like hiring a junior engineer who never asks for PTO.</li>

<li id="7aa2">

<strong>Startup:</strong> You might ship faster without hiring another full-stack.</li>

<li id="4e9c">

<strong>Enterprise team lead:</strong> You’re either integrating these tools or falling behind the ones who are.</li>

<li id="bd9f">

<strong>Student:</strong> You now have access to tooling better than what most FAANG interns had two years ago.</li>

</ul>
<p id="6f9d">And if you’re in the “I just write code for fun” camp? Even better.<br>Because now you can tinker with the same tech that billion-dollar labs are testing internally.</p>
<h3 id="7417">TLDR:</h3>
<ul>

<li id="07e0">Quen 3 Coder isn’t just about writing code it can run and test it too.</li>

<li id="f739">It’s open, which means you (yes, you) can use the same tools that top labs are.</li>

<li id="dae7">Devs who ignore this shift are gonna be the ones asking if their jobs are safe in 6 months.</li>

</ul>
<span></span><span></span><span></span><h2 id="c729">2. Meet Quen 3 Coder: China’s openweight coding beast</h2>
<p id="63d8">In the AI coding space, Claude 4 has been the gold standard. Fast, agentic, scary good at interpreting vague prompts and turning them into well-structured code.</p>
<p id="4180">Now, for the first time, it’s got real competition.<br> And it’s not coming from Silicon Valley it’s coming from Hangzhou.</p>
<p id="5ca5">Alibaba just released <strong>Quen 3 Coder</strong>, a brand new <strong>openweight coding model</strong> that isn’t just close to Claude… it’s <em>dangerously close.</em></p>
<p id="2445">Let’s break this down.</p>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AnpFG73Y5fTnSw90ZGBSckA.png" width="800" height="533"><h3 id="06d5">2.1. What’s an openweight model?</h3>
<p id="8181">Unlike closed APIs like ChatGPT or Claude, <strong>openweight models give you full access to the model’s weights</strong> the actual guts of the neural net. You can fine-tune it, deploy it, run it in your stack, modify it, whatever.</p>
<p id="69fc">But Quen isn’t just openweight it’s <strong>agentic</strong>, too.</p>
<h3 id="a4ee">2.2. Agentic… like, self-operating?</h3>
<p id="fc4a">Exactly.<br>Quen 3 Coder isn’t just generating code; it can <strong>run</strong>, <strong>execute</strong>, and <strong>test</strong> your code on the fly just like a real developer. It can reason over runtime output, correct itself, and even suggest next steps.</p>
<p id="b0ba">It’s built using a <strong>mixture-of-experts (MoE)</strong> architecture, which basically means the model is divided into specialized subnetworks “experts” that activate depending on the task. Think of it like summoning a specific class in an RPG: Tank for infra, Mage for JS, Bard for regex.</p>
<p id="0377">That MoE design makes it more <strong>efficient</strong> and <strong>scalable</strong>, even when performing Claude-level tasks.</p>
<h3 id="2cf3">2.3. And performance?</h3>
<p id="a092">Let’s just say it’s not messing around.</p>
<p id="ca99">According to benchmarks (more on those in a sec), <strong>Quen 3 Coder beats GPT-4.1</strong>, is well above the recent Chinese model <strong>Kimmy K2</strong>, and comes <strong>very close to Claude 4</strong> all while using a <strong>smaller model size.</strong></p>
<p id="73a1">Smaller means faster inference. Less GPU load.<br>And for cloud deployments, that means saved.</p>
<h3 id="6aec">TLDR:</h3>
<ul>

<li id="7d40">Quen 3 Coder is an openweight, MoE-style coding model from Alibaba.</li>

<li id="3cb0">It can generate, run, and fix code making it a legit agent, not just a code monkey.</li>

<li id="9010">Benchmarks show it competes with Claude 4 but uses fewer resources.</li>

</ul>
<span></span><span></span><span></span><h2 id="fcd9">3. How it was trained: 20,000 coders in a digital bootcamp</h2>
<p id="45c8">Let’s talk about the training stack behind this beast because Quen 3 Coder wasn’t just trained; it was <em>hardened</em>.</p>
<p id="e011">This model saw more code than most devs will ever write, review, or rage-quit over in a lifetime. And it learned it all in parallel.</p>
<h3 id="6896">3.1. 7.5 trillion tokens, 70% code</h3>
<p id="2972">That’s not a typo.<br>Quen 3 Coder was trained on <strong>7.5 trillion tokens</strong>, with a whopping <strong>70% of the dataset being code</strong>. That includes code across languages, styles, architectures, and probably enough cursed legacy spaghetti to break your linter.</p>
<p id="65c1">To put that in perspective:</p>
<blockquote><p id="278d">A developer with 50 years of experience has maybe written or read 100 million tokens of code.<br> Quen has seen <strong>75,000x more</strong>.</p></blockquote>
<h3 id="8651">3.2. The model trained itself using another model</h3>
<p id="b93d">And here’s the meta part.</p>
<p id="b400">Instead of relying on noisy or low-quality data, the team used <strong>their previous model to filter and clean</strong> the new training set. Basically, they let the old AI help the new AI figure out what’s worth learning.</p>
<p id="7c8d">That’s AI bootstrapping AI.<br>Which is either brilliant or horrifying, depending on how much you trust recursive optimization.</p>
<h3 id="07ca">3.3. Reinforcement learning in 20,000 environments</h3>
<p id="046b">This isn’t just static data slurping.<br>Quen 3 Coder was trained using <strong>long-horizon reinforcement learning</strong> in <strong>20,000 parallel simulated coding environments.</strong></p>
<p id="cd7e">That means the model didn’t just learn from static examples it <strong>tried to solve real-world programming tasks</strong>, with real execution and feedback, across thousands of isolated environments.</p>
<p id="f55f">Imagine a virtual bootcamp with 20,000 devs solving the same issue at once except none of them need sleep, argue over tabs vs. spaces, or forget to commit.</p>
<p id="5d80">That’s what Alibaba built.</p>
<h3 id="5b8a">TLDR:</h3>
<ul>

<li id="6274">Quen 3 Coder was trained on 7.5T tokens, 70% of which was code.</li>

<li id="1b22">It used older models to clean training data (AI training AI).</li>

<li id="1ae4">It ran RLHF in 20k parallel coding environments, simulating real dev workflows.</li>

<li id="3fd1">This is less “pretrained model” and more “digital army of coders grinding Leetcode 24/7.”</li>

</ul>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2ADY1IvrRc59Nz95CnzoOxuA.png" width="800" height="1200"><span></span><span></span><span></span><h2 id="f5d5">4. The new Quen CLI: AI agent goes terminal mode</h2>
<p id="3fda">If you thought this was just another API endpoint or chat interface, think again.<br> Quen 3 Coder comes with a fully functional <strong>CLI tool</strong> and it’s not some afterthought wrapper.</p>
<p id="650a">It’s a fork of the recently open-sourced <strong>Gemini CLI</strong> (from Google), and it brings <strong>agentic coding right into your terminal.</strong></p>
<h3 id="1a69">4.1. Wait, what does it do?</h3>
<p id="0ccf">You give it a coding task from the CLI.<br> It runs it, tests it, checks output, and can even <strong>rewrite parts of your codebase</strong> as needed.</p>
<p id="05be">All from your shell.</p>
<p id="dd1b">It’s basically an AI dev that lives inside your terminal like a digital ghost coder.<br> It’ll write your Python script, run your test suite, tell you why your linter is mad, and then fix it before you have time to rage-Google the error.</p>
<h3 id="1aab">4.2. Why is that a big deal?</h3>
<p id="094b">Most current AI tools are sandboxed to the browser or IDE.<br> Even ones like Claude and GPT need plugin scaffolding or a bunch of janky copy-paste pipelines to get them working in a real dev flow.</p>
<p id="24f5">But this?<br> You fire up a terminal, run the <strong>Quen CLI</strong>, and it’s off to the races <strong>reading and writing your actual codebase</strong>, just like a teammate who lives in Bash.</p>
<h3 id="fa5d">4.3. The Gemini CLI DNA</h3>
<p id="aafc">Because it’s based on the <strong>Gemini CLI</strong>, you inherit some seriously solid tooling:</p>
<ul>

<li id="b684">Easy shell integration</li>

<li id="a4e1">Smart file-level operations</li>

<li id="ad11">Real command execution feedback</li>

<li id="155b">Multi-file workflows (no prompt-hacking to fit context)</li>

</ul>
<p id="0963">And Quen 3 Coder takes that up a notch by <strong>actually being able to reason over execution traces and suggest fixes.</strong> Which means fewer loops of trial and error, and more “it just works” moments.</p>
<h3 id="2c1a">TLDR</h3>
<ul>

<li id="538a">Quen 3 Coder comes with a real CLI, forked from Gemini CLI.</li>

<li id="c4e3">It can run, test, and modify code directly from the terminal.</li>

<li id="7b2d">Forget chat-based tools this one speaks Bash.</li>

</ul>
<span></span><span></span><span></span><h2 id="20e0">5. Context window madness: Hold your whole startup</h2>
<p id="90b8">If you’ve ever cursed a model for “forgetting” what you told it two prompts ago, this part’s for you.</p>
<p id="0477">Quen 3 Coder doesn’t just have memory.<br>It has <strong>ridiculous</strong> memory.</p>
<p id="7e21">Its context window starts at <strong>256,000 tokens</strong> and can stretch up to <strong>1 million tokens</strong>.</p>
<p id="efaa">That’s not just long. That’s <em>“hold your entire backend repo and CI/CD pipeline while debugging”</em> long.</p>
<h3 id="2cd2">5.1. For context: what is context?</h3>
<p id="3132">A model’s “context window” is how much it can hold in memory during a single interaction your prompt, the code, supporting files, comments, logs, and output.</p>
<p id="dbb8">Old-school models were limited to 4k tokens. GPT-4 hovers at 128k. Claude 2 hit 200k.<br>Quen 3 Coder just dunked on all of them.</p>
<h3 id="9ddd">5.2. What can you actually fit in 1 million tokens?</h3>
<p id="580b"><strong>Rough estimate:</strong></p>
<ul>

<li id="ca7b">The full source code of a startup’s backend</li>

<li id="6139">Frontend JS framework, build files, config</li>

<li id="ac36">All your API documentation</li>

<li id="3acb">Internal tools and scripts</li>

<li id="5980">Every TODO and FIXME you’ve ignored since 2022</li>

</ul>
<p id="c2bc">And then it still has room to read your log files and reason over a recent stacktrace.</p>
<p id="d528">This kind of capacity is game-changing for <strong>agentic workflows</strong>, where the model needs to reason across multiple files, layers, and feedback loops.</p>
<h3 id="95f1">5.3. But what’s the point if others do it too?</h3>
<p id="12b5">Claude 4 also supports long contexts but Quen is open, leaner, and potentially faster to deploy in custom setups.</p>
<p id="3a81">Want to fine-tune it on your own codebase, documents, and logs?<br> You actually <em>can.</em></p>
<h3 id="9a44">TLDR</h3>
<ul>

<li id="9a5d">Quen 3 Coder supports 256k–1M token context windows.</li>

<li id="8005">That’s enough for your entire codebase and all its tech debt.</li>

<li id="888c">This makes it ideal for multi-file debugging, agentic tasks, and in-depth reasoning over large projects.</li>

</ul>
<span></span><span></span><span></span><h2 id="67e2">6. But can you run it locally? Lol no.</h2>
<p id="6c3d">We know what you’re thinking:</p>
<blockquote><p id="a2d8">“It’s openweight! Time to download it and run it on my laptop!”</p></blockquote>
<p id="4e36">Nope. Stop. Delete that thought.<br> Unless your laptop is secretly a GPU cluster parked in a datacenter with its own substation, <strong>you’re not running Quen 3 Coder locally</strong>.</p>
<h3 id="7aba">6.1. Why not?</h3>
<p id="1521">Because this thing is <strong>massive</strong>.</p>
<p id="b23a">The flagship version of Quen 3 Coder clocks in at <strong>480 billion parameters</strong>.<br> That’s beyond chonky. That’s “rent-a-rack-of-H100s” big.</p>
<p id="a592">If you actually wanted to spin it up on-prem, you’d need:</p>
<ul>

<li id="232c">Tens of thousands of dollars in high-end GPUs</li>

<li id="4e8c">Specialized infrastructure to load and shard the model</li>

<li id="81a2">A cooling system that doesn’t melt your ceiling</li>

<li id="34ef">And enough electricity to power a small village</li>

</ul>
<h3 id="4272">6.2. So how do you use it?</h3>
<p id="7b56">Like most serious openweight models: <strong>via API.</strong></p>
<p id="61d3">You get access through a cloud provider (Alibaba Cloud or a third-party hosting Quen), and run your interactions through:</p>
<ul>

<li id="fda7">REST API or WebSocket</li>

<li id="76f8">The <strong>Quen CLI</strong> we mentioned earlier</li>

<li id="6932">Custom endpoints via LangChain or OpenDevin-style agents</li>

</ul>
<p id="c116">That way, you still get full agentic power without frying your dev laptop’s CPU fan.</p>
<h3 id="649f">TLDR</h3>
<ul>

<li id="ac9e">Quen 3 Coder is huge (480B parameters) and realistically can’t be run locally.</li>

<li id="af97">You’ll need an API key from a hosting provider to use it.</li>

<li id="7dcc">Openweight ≠ “easy to deploy at home.”</li>

<li id="4b08">Unless your “home” is an ex-NVIDIA warehouse.</li>

</ul>
<span>Press enter or click to view image in full size</span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AW1Gfsugnzfzi6FPylvda0g.png" width="800" height="800"><h3 id="1733">6.3. So, can it dethrone Claude?</h3>
<p id="3a9c">Okay, so Quen 3 Coder is:</p>
<ul>

<li id="8100">Open</li>

<li id="7909">Massive</li>

<li id="13e6">Trained on insane scale</li>

<li id="eaa1">Benchmarking just behind Claude 4</li>

<li id="c03c">Has a 1M token context</li>

<li id="71dc">Comes with a powerful CLI …and still can’t knock Claude off the throne. Yet.</li>

</ul>
<h3 id="85c4">6.4. Why?</h3>
<p id="cdeb">Because Claude didn’t win by just being smart it won by being <strong>consistent</strong>, <strong>well-integrated</strong>, and <strong>insanely good at reasoning</strong> across natural language and code.</p>
<p id="815e">Quen 3 Coder is <strong>close</strong>, but “close” still isn’t “better.”<br> And in the dev world, “better” often means “more predictable,” not just “more powerful.”</p>
<p id="1606">That said, Quen <em>is</em>:</p>
<ul>

<li id="2d16">

<strong>Open</strong>, which Claude isn’t</li>

<li id="fe59">

<strong>Agentic</strong>, with deeper CLI access than Claude’s sandboxed setup</li>

<li id="420b">

<strong>Cheaper to run at scale</strong> (in theory) due to its MoE architecture</li>

</ul>
<p id="678f">But dethroning Claude will take more than matching it.<br> It’ll take <strong>leapfrogging</strong> it and that hasn’t quite happened yet.</p>
<p id="342b">In the meantime, Claude still sits at the top.<br> But this is the closest anyone’s gotten especially with open weights.</p>
<p id="cac7">And honestly? That alone is <strong>massive</strong>.</p>
<span></span><span></span><span></span><h2 id="83b3">7. The plot twist: OpenAI vs. Google vs. China</h2>
<p id="f9a7">So while devs are busy comparing token limits and CLI flags, the <strong>real story</strong> might be happening behind the scenes in the PR war and race for dominance between OpenAI, Google, and now… Alibaba.</p>
<p id="750a">Let’s break it down.</p>
<h3 id="6863">7.1. OpenAI’s delayed open model</h3>
<p id="4054">OpenAI had a big open-source release planned rumored to be their own openweight Claude-killer.</p>
<p id="1a9c">But it got… delayed.<br> Why? Industry whispers suggest it’s because <strong>models like Quen would instantly overshadow it.</strong></p>
<p id="8ce8">That’s right.<br> The once-uncontested leader might be stalling because <strong>China just outpaced them in open agentic coding tech.</strong></p>
<p id="0791">And no one wants to get wrecked in their own marketing cycle..</p>
<h3 id="9a6b">7.2 Meanwhile, Google’s busy flexing math skills</h3>
<p id="1c3c">You’d think this would be OpenAI’s moment to strike back, but Google pulled a move of its own.</p>
<p id="cfa2">Both OpenAI and Google recently <strong>achieved gold medal–level performance in the International Mathematical Olympiad</strong> using their AI models.</p>
<p id="f120">But OpenAI tried to steal the showby announcing their win <em>before</em> the closing ceremonies.</p>
<p id="6f03">It backfired.</p>
<p id="0846">It came off as desperate. Instead of making OpenAI look strong, it made Google look classy and made people wonder what OpenAI was scrambling to hide.</p>
<h3 id="1d94">7.3. And then there’s Alibaba, quietly shipping</h3>
<p id="0f00">While the American labs fight over press cycles, <strong>Alibaba just shipped a model that works.</strong></p>
<p id="5567">Open. Agentic. With benchmarks.</p>
<p id="6feb">No hype reel, no tweet threads, no “co-founder just left to start an AI rival” headlines.</p>
<p id="17dc">Just code.</p>
<h3 id="d95d">TLDR</h3>
<ul>

<li id="b5fe">OpenAI delayed its open model possibly because Quen would’ve outshined it.</li>

<li id="f3c0">Google and OpenAI both hit gold-level math Olympiad AI scores but PR battles got weird.</li>

<li id="c1d7">Meanwhile, Alibaba just dropped Quen 3 Coder with almost no fanfare and real results.</li>

<li id="0158">This is no longer just about tools it’s a global AI chess match.</li>

</ul>
<span></span><span></span><span></span><h2 id="095c">Conclusion:</h2>
<p id="e089">So what do we make of all this?</p>
<p id="9aa3">Quen 3 Coder isn’t just a cool new model. It’s a <strong>clear signal</strong> that the age of open, agentic AI coders has fully arrived and not everyone leading the charge is based in San Francisco.</p>
<p id="8c80">It doesn’t dethrone Claude 4 just yet… but it doesn’t have to.<br> Because the <strong>bigger shift</strong> is that now, <em>anyone</em> with an API key and a bit of CLI-fu can harness tools that were previously only accessible inside billion-dollar AI labs.</p>
<p id="813e">That’s huge.</p>
<p id="ebb2">It also marks a shift from passive coding tools (like autocomplete) to <strong>active, reasoning agents</strong> that participate in your workflow.<br> And if you’re not experimenting with these tools yet, you’re already behind.</p>
<h3 id="4992">So what should devs actually do right now?</h3>
<ol>

<li id="24fb">

<strong>Try the model via API</strong> Look out for official endpoints on Alibaba Cloud or other providers hosting Quen 3 Coder.</li>

<li id="b751">

<strong>Install the Quen CLI</strong> Test small workflows, like bug detection or test generation, right from your terminal.</li>

<li id="4b50">

<strong>Compare outputs</strong> Benchmark Quen against Claude, GPT-4, and Code Llama on your own repo. You’ll learn fast where it shines (and where it still fumbles).</li>

<li id="e93b">

<strong>Stay updated</strong> The pace of agentic AI evolution is accelerating. Don’t wait for “perfect” start with what’s usable now.</li>

</ol>
<h3 id="eb88">TLDR</h3>
<ul>

<li id="5878">Quen 3 Coder is the most serious open rival to Claude 4 yet.</li>

<li id="f54e">Its performance, agentic design, and open access signal a major shift</li>

<li id="7450">It may not be the final boss but it’s no sidekick either.</li>

<li id="14ba">The future of dev work? Assisted, tested, and maybe even written by your shell-based AI teammate.</li>

</ul>
<h3 id="5435"><strong>Helpful resources</strong></h3>
<ul>

<li id="52cb"><a href="https://coderabbit.ai" rel="noopener ugc nofollow noreferrer"><strong>CodeRabbit (AI-powered code reviews)</strong></a></li>

<li id="c75a"><a href="https://www.cursor.so" rel="noopener ugc nofollow noreferrer"><strong>Cursor (AI-native VS Code fork)</strong></a></li>

<li id="2271"><a href="https://github.com/google/gemini-cli" rel="noopener ugc nofollow noreferrer"><strong>Gemini CLI (base for Quen CLI)</strong></a></li>

<li id="7bdf"><a href="https://github.com/microsoft/DeepSpeed" rel="noopener ugc nofollow noreferrer"><strong>Microsoft DeepSpeed (for open model training)</strong></a></li>

<li id="4b5f"><a href="https://huggingface.co/spaces/HuggingFaceH4/open-llm-leaderboard" rel="noopener ugc nofollow noreferrer"><strong>Hugging Face Open Models leaderboard</strong></a></li>

</ul>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AE822yMyX86mqo4UvRys4wQ.png" width="800" height="272">

