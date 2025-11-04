---
Title: Run LangChain Locally in 15 Minutes (Without a Single API Key)
Description: 
Author: Fabio Hiroki
Date: 2025-11-04T21:55:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>You don’t need an API key.</p>

<p>That’s the secret everyone seems to forget.</p>

<p>I wanted to dive into LangChain, the powerful framework for building with Large Language Models. But every guide, even the official docs, felt like a trap. They were outdated, incomplete, or pushed me toward a paid Anthropic key.</p>

<p>I wrestled with Python version errors. I hunted for missing dependencies. It was a frustrating maze designed to make me quit.</p>

<p>So I carved my own path.</p>

<p>This is the guide I wish I had. It’s your shortcut to running a powerful LLM on your own machine, completely for free, in the next 15 minutes.</p>

<p>P.S.: If you want to skip the article, you can check the code directly on <a href="https://github.com/fabiothiroki/langchain-quickstart" rel="noopener noreferrer">Github</a>.</p>

<h2>
  
  
  1. First, understand why this is a game-changer.
</h2>

<p>Running an LLM locally is about one thing: speed.</p>

<p>Not processing speed, but the speed of learning.</p>

<p>When you're just trying to get a feel for a new framework, the last thing you want is a roadblock. You don't need to sign up for an API, enter a credit card, or wait for an approval email just to run a "hello, world" prompt.</p>

<p>This local-first approach removes all that friction.</p>

<p>And as a bonus, it comes with three other superpowers:</p>

<ul>
<li>Zero Cost: It’s completely free, no matter how many prompts you run.</li>
<li>Total Privacy: Your data never leaves your machine.</li>
<li>Full Control: You choose the model, you own the process.</li>
</ul>

<p>This isn't just about saving money. It's about creating the fastest path to actually building and learning.</p>

<h2>
  
  
  2. Next, get your environment right (this is critical).
</h2>

<h3>
  
  
  Set Up Your Python Version.
</h3>

<p>This is where 99% of tutorials fail. They give vague instructions, but the AI world moves so fast that packages break between minor versions. We will avoid that entirely.</p>

<p>This tutorial was built and verified using Python 3.13.0. Sticking to this exact version removes any guesswork. For managing Python versions like a pro, I recommend <a href="https://github.com/pyenv/pyenv" rel="noopener noreferrer"><code>pyenv</code></a>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pyenv <span class="nb">install </span>3.13.0
pyenv <span class="nb">local </span>3.13.0
</code></pre>

</div>



<h3>
  
  
  Install Ollama.
</h3>

<p>Ollama is the magic that runs the LLM on your machine. Go to <a href="https://ollama.com/download/" rel="noopener noreferrer">https://ollama.com/download/</a> and download it for your system. Install it like any other application. It will run quietly in the background, ready when you need it.</p>

<h2>
  
  
  3. Now, start Ollama and download your AI model.
</h2>

<p>With Ollama installed, you need to make sure its server is running before you can download or use a model.</p>

<p>Open a new, separate terminal window and run this command. Leave this terminal running in the background.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ollama serve
</code></pre>

</div>



<p><em>Note: If you get an error that the server is already running, that's fine—it means the app started automatically.</em></p>

<p>Now, in your original terminal, pull the llama3.2 model.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ollama pull llama3.2
</code></pre>

</div>



<p>This might take a few minutes. You only have to pull once.</p>

<h2>
  
  
  4. Then, install the exact package versions.
</h2>

<p>To guarantee this works, we will install the exact library versions used in this tutorial.</p>

<p>First, create a project folder and set up a virtual environment (a critical best practice):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python <span class="nt">-m</span> venv venv
<span class="nb">source </span>venv/bin/activate
</code></pre>

</div>



<p>Now, install the pinned dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span><span class="nv">langchain</span><span class="o">==</span>1.0.3 langchain-ollama<span class="o">==</span>1.0.0
</code></pre>

</div>



<p>This ensures you have the precise code that we know works. No surprises, no cryptic error messages.</p>

<h2>
  
  
  5. Finally, write less than 20 lines of Python code.
</h2>

<p>You have the environment, the model, and the libraries. All that's left is to connect them.</p>

<p>Create a new Python file named main.py and paste in the following code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">langchain.agents</span> <span class="kn">import</span> <span class="n">create_agent</span>
<span class="kn">from</span> <span class="n">langchain_ollama</span> <span class="kn">import</span> <span class="n">ChatOllama</span>

<span class="k">def</span> <span class="nf">get_weather</span><span class="p">(</span><span class="n">city</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Get weather for a given city.</span><span class="sh">"""</span>
    <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Return the weather in </span><span class="si">{</span><span class="n">city</span><span class="si">}</span><span class="s"> in a joke.</span><span class="sh">"</span>

<span class="n">agent</span> <span class="o">=</span> <span class="nf">create_agent</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="nc">ChatOllama</span><span class="p">(</span><span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">llama3.2</span><span class="sh">"</span><span class="p">),</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[</span><span class="n">get_weather</span><span class="p">],</span>
    <span class="n">system_prompt</span><span class="o">=</span><span class="sh">"</span><span class="s">You are a helpful assistant</span><span class="sh">"</span><span class="p">,</span>
<span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="c1"># Run the agent
</span>    <span class="n">result</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">invoke</span><span class="p">(</span>
        <span class="p">{</span><span class="sh">"</span><span class="s">messages</span><span class="sh">"</span><span class="p">:</span> <span class="p">[{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">what is the weather in sf</span><span class="sh">"</span><span class="p">}]}</span>
    <span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="n">result</span><span class="p">[</span><span class="sh">"</span><span class="s">messages</span><span class="sh">"</span><span class="p">][</span><span class="o">-</span><span class="mi">1</span><span class="p">].</span><span class="n">content</span><span class="p">)</span>
</code></pre>

</div>



<p>The code creates the model, defines a simple instruction, chains them together, and asks a question. That’s the core of LangChain in action.</p>

<p>In your terminal, run the script:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python main.py
</code></pre>

</div>



<p>You should see something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Fog-get about it, it's always raining in San Francisco!
</code></pre>

</div>



<p>That's it.</p>

<p>You just ran a state-of-the-art LLM on your local machine using LangChain. You've bypassed the confusing docs, avoided all API keys, and built something that is 100% free and private.</p>

<p>Now, what will you build next?</p>

