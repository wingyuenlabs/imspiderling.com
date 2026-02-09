---
Title: AI for PHP Developers. Practical Use of TransformersPHP
Description: 
Author: Samuel Akopyan
Date: 2026-02-09T21:22:28.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  AI in PHP: Not Theory, but a Real Starting Point
</h2>

<p>In my previous articles, I described at a fairly high level why AI seems to be everywhere, yet barely intersects with everyday PHP development. Not because PHP is "unsuitable", but because the conversation usually happens outside our tasks and our habitual way of thinking. And, of course, because there is very little material that explains AI specifically for PHP developers - their problems and their mindset.</p>

<p>After the publication, I was asked the same question several times, in different forms:</p>

<blockquote>
<p>Okay, suppose that's true. Where do you actually start?</p>
</blockquote>

<p>And this is probably the most interesting question I received. Below, I'll try to answer it.</p>




<h2>
  
  
  About the "Entry Point"
</h2>

<p>When you hear "using AI in a project", your mind likely jumps to a lot of unnecessary things at once: infrastructure, model training, experiments, separate services, new roles in the team, and so on.</p>

<p>But honestly, in most PHP projects we simply don't need all that (even though figuring it out is a fascinating task in itself).</p>

<p>In practice, we do not need to:</p>

<ul>
<li>train models from scratch</li>
<li>understand gradient descent</li>
<li>collect datasets, etc.</li>
</ul>

<p>It's important to understand that AI ≠ Data Science. My friends, in most PHP projects no one trains models. Please remember this! At least when we're talking about applied PHP projects that use ready-made models, not research or model training.</p>

<p>What we actually need is to take an existing tool and carefully integrate it into the current logic of the application - just like we do with any other library.</p>

<p>And this is where things get interesting.</p>

<h2>
  
  
  What Is TransformersPHP, and What Do You Do With It?
</h2>

<p>Everyone already knows about using LLMs via APIs. It's useful and convenient, but in that form AI remains something external - a service you send text to. What's inside is a black box.</p>

<p>At some point, I wanted to look at a more "down-to-earth" level: not text generation, but semantic representation: embeddings, classification, and search tasks. This is where most applied backend problems are actually solved - text search, comparison, automatic classification - not human–model dialogue.</p>

<p>And here it suddenly turned out that there are tools that let you do this directly in PHP, without a Python stack. One of them is <a href="https://transformers.codewithkyrian.com/" rel="noopener noreferrer">TransformersPHP</a>.</p>

<p>It's important to clarify right away: this is not an attempt to turn PHP into Python, nor a universal solution. It's a library for <a href="https://hazelcast.com/foundations/ai-machine-learning/machine-learning-inference/" rel="noopener noreferrer">inference</a> - using already trained models.</p>

<p>In my view, TransformersPHP is one of the most interesting and illustrative projects in the modern PHP ML ecosystem. Special thanks to its author, <a href="https://github.com/CodeWithKyrian" rel="noopener noreferrer">Kyrian Obikwelu</a>, for creating and maintaining it.</p>

<p>In short, it's a library that allows you to use transformer models (BERT, RoBERTa, DistilBERT, and others) directly from PHP - without Python and without external APIs. </p>

<p>Essentially, it's a PHP-oriented wrapper around <a href="https://huggingface.co/docs/transformers/en/index" rel="noopener noreferrer">Hugging Face Transformers</a> ideas, adapted for the PHP ecosystem and real-world use cases.</p>

<h2>
  
  
  The Key Feature: Local Inference
</h2>

<p>Models are loaded and executed inside the PHP application itself (via <a href="https://github.com/microsoft/onnxruntime" rel="noopener noreferrer">ONNX Runtime</a>). This opens up important architectural possibilities:</p>

<ul>
<li>no network calls to LLM APIs</li>
<li>full control over data (important for privacy-sensitive environments)</li>
<li>predictable and stable request latency</li>
<li>offline operation (after the first run and model download)</li>
</ul>

<p>TransformersPHP supports common NLP tasks such as embeddings, text classification, semantic similarity, and more.</p>

<h2>
  
  
  What I Like Most
</h2>

<p>The most valuable feeling is the absence of context switching.</p>

<p>I stay in PHP. I write the same code. I deploy the same way. I use the same architectural approaches. Nothing changes.</p>

<p>In this setup, the model is not some "magical object", but just another data source. Unusual, yes - but no more mysterious than any other.</p>

<p>And that completely changes your attitude toward the topic. Don't you agree?</p>

<h2>
  
  
  Running a Model in 10 Minutes
</h2>

<p>The first launch is critical.<br>
If it's complicated, that's usually where everything ends.</p>

<p>With TransformersPHP, the feeling is the opposite: it's more like working with a regular dependency.</p>

<p>The full installation guide is available in the <a href="https://transformers.codewithkyrian.com/getting-started#installation" rel="noopener noreferrer">documentation</a>.<br>
For simplicity, we'll run everything in Docker with all dependencies included.</p>

<p>Yes, the Dockerfile looks large - but it's one-time infrastructure. The actual launch and first demo really take just a few minutes.</p>
<h2>
  
  
  Requirements
</h2>

<ul>
<li>PHP 8.1 or higher</li>
<li>Composer</li>
<li>PHP FFI extension</li>
<li>JIT compilation (optional, for performance)</li>
<li>Increased memory limit (for heavier tasks like text generation)</li>
</ul>
<h2>
  
  
  Project Structure
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/demo/
  ├── app/
  │    ├── demo.php
  │    ├── semantic-search.php
  ├── docker/
  │    ├── Dockerfile
  ├── docker-composer.yaml
  ├── composer.json

</code></pre>

</div>

<h2>
  
  
  Installation (Docker)
</h2>

<p>File: docker-compose.yml<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">networks</span><span class="pi">:</span>
  <span class="na">ai-for-php-developers</span><span class="pi">:</span>
    <span class="na">driver</span><span class="pi">:</span> <span class="s">bridge</span>

<span class="na">services</span><span class="pi">:</span>
  <span class="na">app</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span>
      <span class="na">context</span><span class="pi">:</span> <span class="s">.</span>
      <span class="na">dockerfile</span><span class="pi">:</span> <span class="s">docker/Dockerfile</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">.:/var/www</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8088:8088"</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">php -S 0.0.0.0:8088 -t app</span>
    <span class="na">networks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">ai-for-php-developers</span>
</code></pre>

</div>



<p>File: docker/Dockerfile<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FROM php:8.2-fpm

# ------------------------------
# Install system dependencies
# ------------------------------
RUN apt-get update &amp;&amp; apt-get install -y \
    libzip-dev \
    zip \
    unzip \
    git \
    libxml2-dev \
    libcurl4-openssl-dev \
    libpng-dev \
    libonig-dev \
    &amp;&amp; rm -rf /var/lib/apt/lists/*

# ------------------------------
# Install PHP extensions
# ------------------------------
RUN docker-php-ext-install zip pdo_mysql bcmath xml mbstring curl gd pcntl

# ------------------------------
# Enable FFI
# ------------------------------
RUN apt-get update &amp;&amp; apt-get install -y \
    libffi-dev \
    pkg-config \
    &amp;&amp; rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install ffi
RUN echo "ffi.enable=1" &gt; /usr/local/etc/php/conf.d/ffi.ini

# ------------------------------
# Install ONNX Runtime
# ------------------------------
ENV ONNXRUNTIME_VERSION=1.17.1

RUN curl -L https://github.com/microsoft/onnxruntime/releases/download/v${ONNXRUNTIME_VERSION}/onnxruntime-linux-x64-${ONNXRUNTIME\_VERSION}.tgz \
    | tar -xz \
    &amp;&amp; cp onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}/lib/libonnxruntime.so* /usr/lib/ \
    &amp;&amp; ldconfig \
    &amp;&amp; rm -rf onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}

# ------------------------------
# Install Composer
# ------------------------------
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

# Copy existing application directory contents
COPY . /var/www

# Configure PHP
RUN echo "memory_limit = 512M" &gt;&gt; /usr/local/etc/php/conf.d/docker-php-ram-limit.ini
RUN echo "max_execution_time = 300" &gt;&gt; /usr/local/etc/php/conf.d/docker-php-max-execution-time.ini

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
</code></pre>

</div>



<p>File: composer.json<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"project"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"minimum-stability"</span><span class="p">:</span><span class="w"> </span><span class="s2">"stable"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"prefer-stable"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"require"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"codewithkyrian/transformers"</span><span class="p">:</span><span class="w"> </span><span class="s2">"~0.6.2"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"config"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"allow-plugins"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"codewithkyrian/platform-package-installer"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Run the Environment
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker compose build <span class="nt">--pull</span>
docker compose up <span class="nt">-d</span>
docker compose <span class="nb">exec </span>app /bin/bash <span class="nt">-c</span> <span class="s2">"composer install"</span>
</code></pre>

</div>



<p>The idea is simple: the example should run locally without manual environment setup.</p>

<h2>
  
  
  Basic Demo Example
</h2>

<p>Let's start with the simplest task: sentiment analysis.<br>
If everything above worked correctly, you can run the example below (note that the first run may take a few seconds):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker compose <span class="nb">exec </span>app php app/demo.php
</code></pre>

</div>



<p>File: app/demo.php<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">require_once</span> <span class="k">__DIR__</span> <span class="mf">.</span> <span class="s1">'/../vendor/autoload.php'</span><span class="p">;</span>

<span class="kn">use</span> <span class="k">function</span> <span class="n">Codewithkyrian\Transformers\Pipelines\pipeline</span><span class="p">;</span>

<span class="c1">// Dedicate a sentiment analysis pipeline</span>
<span class="nv">$classifier</span> <span class="o">=</span> <span class="nf">pipeline</span><span class="p">(</span><span class="s1">'sentiment-analysis'</span><span class="p">);</span>

<span class="nv">$out</span> <span class="o">=</span> <span class="nv">$classifier</span><span class="p">([</span><span class="s1">'I love transformers!'</span><span class="p">]);</span>
<span class="nb">print_r</span><span class="p">(</span><span class="nv">$out</span><span class="p">);</span>

<span class="nv">$out</span> <span class="o">=</span> <span class="nv">$classifier</span><span class="p">([</span><span class="s1">'I hate transformers!'</span><span class="p">]);</span>
<span class="nb">print_r</span><span class="p">(</span><span class="nv">$out</span><span class="p">);</span>
<span class="nc">The</span> <span class="n">result</span> <span class="n">is</span> <span class="n">exactly</span> <span class="n">what</span> <span class="n">you</span><span class="err">'</span><span class="n">d</span> <span class="n">expect</span><span class="mf">.</span>
<span class="nc">I</span> <span class="n">love</span> <span class="n">transformers</span><span class="o">!</span>
<span class="k">Array</span> <span class="p">(</span> 
  <span class="p">[</span><span class="n">label</span><span class="p">]</span> <span class="o">=&gt;</span> <span class="no">POSITIVE</span> 
  <span class="p">[</span><span class="n">score</span><span class="p">]</span> <span class="o">=&gt;</span> <span class="mf">0.99978870153427</span> 
<span class="p">)</span>

<span class="nc">I</span> <span class="n">hate</span> <span class="n">transformers</span><span class="o">!</span>
<span class="k">Array</span> <span class="p">(</span> 
  <span class="p">[</span><span class="n">label</span><span class="p">]</span> <span class="o">=&gt;</span> <span class="no">NEGATIVE</span> 
  <span class="p">[</span><span class="n">score</span><span class="p">]</span> <span class="o">=&gt;</span> <span class="mf">0.99863630533218</span> 
<span class="p">)</span>
</code></pre>

</div>



<p>The purpose of this example is purely psychological: to remove the barrier.<br>
A model in PHP is just another object you can work with.</p>
<h2>
  
  
  Architectural Role of TransformersPHP
</h2>

<p>This library does not compete with large LLM services like GPT or Claude. It covers a different, very important layer:</p>

<ul>
<li>fast embeddings</li>
<li>local classification</li>
<li>semantic search</li>
<li>lightweight NLP without external dependencies</li>
</ul>

<p>In combination with PHP, this makes perfect sense. PHP remains the core business-logic layer, while transformers become an embedded tool rather than a remote service.</p>

<p>TransformersPHP is a great example of how modern ML is gradually becoming less "foreign" to PHP and more native to its ecosystem - through careful engineering bridges like ONNX.</p>
<h2>
  
  
  Real Case: Semantic Search Over Events
</h2>

<p>Tutorial examples are nice, but they get boring quickly. Let's look at a task that actually occurs in backend systems.</p>

<p>Run the example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>docker compose exec app php app/semantic-search.php
</code></pre>

</div>



<h2>
  
  
  Scenario
</h2>

<p>We have events with short textual descriptions. A user searches for:</p>

<ul>
<li>"sanctions against IT companies"</li>
<li>"space race among countries in the region"</li>
</ul>

<p>These exact phrases may not appear in the data at all. Traditional keyword search starts to struggle: synonyms, morphology, different languages, hacks on top of hacks.</p>

<h2>
  
  
  A Thought Experiment
</h2>

<p>Suppose we have an event feed like this:</p>

<ul>
<li>new restrictions introduced against technology corporations</li>
<li>regional countries increase investments in satellite programs</li>
<li>escalation of political conflict in several provinces</li>
</ul>

<p>Now the user searches: "space race among countries in the region".</p>

<p>None of those words must appear literally. And that's normal - people don't think the way systems store text.</p>

<h2>
  
  
  Solution Idea
</h2>

<p>Instead of guessing words, we search by meaning - in an engineering sense:</p>

<ul>
<li>event description → vector</li>
<li>user query → vector</li>
<li>find nearest vectors</li>
</ul>

<p>Embeddings become a universal index of meaning.</p>

<h2>
  
  
  Steps
</h2>

<ul>
<li>Take events {id, title, description}</li>
<li>Compute embeddings only for description</li>
<li>Embed the user query</li>
<li>Find nearest vectors</li>
<li>Sort and return results</li>
</ul>

<p>No model training. No complex infrastructure.<br>
Event embeddings are computed once and stored anywhere.<br>
Queries are embedded at search time.</p>

<p>And… voilà.<br>
No model training.<br>
No magic.<br>
Just a different way to represent text.</p>
<h2>
  
  
  Operation Logic
</h2>

<p>We'll place the operation logic in a separate class, SemanticEventSearch. This class doesn't claim to be the best code on the planet, and is written for demonstration purposes only - so we'll omit any comments on its quality.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">final</span> <span class="kd">class</span> <span class="nc">SemanticEventSearch</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="kt">string</span> <span class="nv">$model</span> <span class="o">=</span> <span class="s1">'Xenova/paraphrase-multilingual-MiniLM-L12-v2'</span><span class="p">;</span>
    <span class="k">private</span> <span class="kt">string</span> <span class="nv">$cachePath</span><span class="p">;</span>
    <span class="k">private</span> <span class="kt">string</span> <span class="nv">$defaultQuery</span> <span class="o">=</span> <span class="s1">'санкции против IT-компаний'</span><span class="p">;</span>
    <span class="k">private</span> <span class="kt">int</span> <span class="nv">$topN</span><span class="p">;</span>

    <span class="k">private</span> <span class="kt">?string</span> <span class="nv">$query</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="cd">/** @var list&lt;array{id:int,title:string,description:string}&gt; */</span>
    <span class="k">private</span> <span class="kt">array</span> <span class="nv">$events</span><span class="p">;</span>

    <span class="cd">/** @var array&lt;int, list&lt;float|int&gt;&gt; */</span>
    <span class="k">private</span> <span class="kt">array</span> <span class="nv">$eventEmbeddingsById</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="k">private</span> <span class="nv">$embedder</span><span class="p">;</span>

    <span class="cd">/**
     * Create a new semantic search instance.
     *
     * @param int $topN Number of results to return.
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">(</span><span class="kt">int</span> <span class="nv">$topN</span> <span class="o">=</span> <span class="mi">3</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">cachePath</span> <span class="o">=</span> <span class="k">__DIR__</span> <span class="mf">.</span> <span class="s1">'/../embeddings.events.json'</span><span class="p">;</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">events</span> <span class="o">=</span> <span class="p">[];</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">embedder</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">topN</span> <span class="o">=</span> <span class="nv">$topN</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Inject events that will be indexed/searched.
     *
     * @param list&lt;array{id:int,title:string,description:string}&gt; $events
     * @return $this
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">setEvents</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$events</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">events</span> <span class="o">=</span> <span class="nv">$events</span><span class="p">;</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span> <span class="o">=</span> <span class="p">[];</span>
        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Set the embeddings model identifier.
     *
     * Switching model invalidates in-memory embeddings.
     *
     * @param string $model
     * @return $this
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">setModel</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$model</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">model</span> <span class="o">=</span> <span class="nv">$model</span><span class="p">;</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">embedder</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span> <span class="o">=</span> <span class="p">[];</span>
        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Set the query to be searched.
     *
     * @param string $query
     * @return $this
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">setQuery</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$query</span><span class="p">):</span> <span class="kt">self</span>
    <span class="p">{</span>
        <span class="nv">$q</span> <span class="o">=</span> <span class="nb">trim</span><span class="p">(</span><span class="nv">$query</span><span class="p">);</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">query</span> <span class="o">=</span> <span class="nv">$q</span> <span class="o">===</span> <span class="s1">''</span> <span class="o">?</span> <span class="kc">null</span> <span class="o">:</span> <span class="nv">$q</span><span class="p">;</span>
        <span class="k">return</span> <span class="nv">$this</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Run the end-to-end semantic search pipeline (cache -&gt; embed query -&gt; score -&gt; top-N).
     *
     * @return array{query:string,results:list&lt;array{score:float,event:array{id:int,title:string,description:string}}&gt;}
     * @throws RuntimeException If events are not set or embeddings output is unexpected.
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">run</span><span class="p">():</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="nb">count</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">events</span><span class="p">)</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nc">RuntimeException</span><span class="p">(</span><span class="s1">'Events list is empty. Call setEvents() before run().'</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="k">if</span> <span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">embedder</span> <span class="o">===</span> <span class="kc">null</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">embedder</span> <span class="o">=</span> <span class="nf">pipeline</span><span class="p">(</span><span class="s1">'embeddings'</span><span class="p">,</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">model</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">loadEmbeddingsFromCacheIfCompatible</span><span class="p">();</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">ensureAllEventEmbeddings</span><span class="p">();</span>

        <span class="nv">$query</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">query</span> <span class="o">??</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">defaultQuery</span><span class="p">;</span>
        <span class="nv">$queryVec</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">embedText</span><span class="p">(</span><span class="nv">$query</span><span class="p">);</span>

        <span class="nv">$results</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">search</span><span class="p">(</span><span class="nv">$queryVec</span><span class="p">);</span>
        <span class="k">return</span> <span class="p">[</span>
            <span class="s1">'query'</span> <span class="o">=&gt;</span> <span class="nv">$query</span><span class="p">,</span>
            <span class="s1">'results'</span> <span class="o">=&gt;</span> <span class="nv">$results</span><span class="p">,</span>
        <span class="p">];</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Compute an embedding vector for a single text.
     *
     * @param string $text
     * @return list&lt;float|int&gt;
     * @throws RuntimeException
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">embedText</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$text</span><span class="p">):</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="nv">$emb</span> <span class="o">=</span> <span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">embedder</span><span class="p">)(</span><span class="nv">$text</span><span class="p">,</span> <span class="n">normalize</span><span class="o">:</span> <span class="kc">true</span><span class="p">,</span> <span class="n">pooling</span><span class="o">:</span> <span class="s1">'mean'</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="nb">is_array</span><span class="p">(</span><span class="nv">$emb</span><span class="p">)</span> <span class="o">||</span> <span class="o">!</span><span class="k">isset</span><span class="p">(</span><span class="nv">$emb</span><span class="p">[</span><span class="mi">0</span><span class="p">])</span> <span class="o">||</span> <span class="o">!</span><span class="nb">is_array</span><span class="p">(</span><span class="nv">$emb</span><span class="p">[</span><span class="mi">0</span><span class="p">]))</span> <span class="p">{</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nc">RuntimeException</span><span class="p">(</span><span class="s1">'Unexpected embeddings output format'</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="nv">$emb</span><span class="p">[</span><span class="mi">0</span><span class="p">];</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Cosine similarity between two vectors.
     *
     * @param list&lt;float|int&gt; $a
     * @param list&lt;float|int&gt; $b
     * @return float
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">cosineSimilarity</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$a</span><span class="p">,</span> <span class="kt">array</span> <span class="nv">$b</span><span class="p">):</span> <span class="kt">float</span>
    <span class="p">{</span>
        <span class="nv">$n</span> <span class="o">=</span> <span class="nb">min</span><span class="p">(</span><span class="nb">count</span><span class="p">(</span><span class="nv">$a</span><span class="p">),</span> <span class="nb">count</span><span class="p">(</span><span class="nv">$b</span><span class="p">));</span>

        <span class="nv">$dot</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>
        <span class="nv">$normA</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>
        <span class="nv">$normB</span> <span class="o">=</span> <span class="mf">0.0</span><span class="p">;</span>

        <span class="k">for</span> <span class="p">(</span><span class="nv">$i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nv">$i</span> <span class="o">&lt;</span> <span class="nv">$n</span><span class="p">;</span> <span class="nv">$i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$x</span> <span class="o">=</span> <span class="p">(</span><span class="n">float</span><span class="p">)</span> <span class="nv">$a</span><span class="p">[</span><span class="nv">$i</span><span class="p">];</span>
            <span class="nv">$y</span> <span class="o">=</span> <span class="p">(</span><span class="n">float</span><span class="p">)</span> <span class="nv">$b</span><span class="p">[</span><span class="nv">$i</span><span class="p">];</span>

            <span class="nv">$dot</span> <span class="o">+=</span> <span class="nv">$x</span> <span class="o">*</span> <span class="nv">$y</span><span class="p">;</span>
            <span class="nv">$normA</span> <span class="o">+=</span> <span class="nv">$x</span> <span class="o">*</span> <span class="nv">$x</span><span class="p">;</span>
            <span class="nv">$normB</span> <span class="o">+=</span> <span class="nv">$y</span> <span class="o">*</span> <span class="nv">$y</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">if</span> <span class="p">(</span><span class="nv">$normA</span> <span class="o">&lt;=</span> <span class="mf">0.0</span> <span class="o">||</span> <span class="nv">$normB</span> <span class="o">&lt;=</span> <span class="mf">0.0</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span> <span class="mf">0.0</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="nv">$dot</span> <span class="o">/</span> <span class="p">(</span><span class="nb">sqrt</span><span class="p">(</span><span class="nv">$normA</span><span class="p">)</span> <span class="o">*</span> <span class="nb">sqrt</span><span class="p">(</span><span class="nv">$normB</span><span class="p">));</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Load a JSON file and decode to array.
     *
     * @param string $path
     * @return array|null
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">loadJsonFile</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$path</span><span class="p">):</span> <span class="kt">?array</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="nb">is_file</span><span class="p">(</span><span class="nv">$path</span><span class="p">))</span> <span class="p">{</span>
            <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="nv">$raw</span> <span class="o">=</span> <span class="nb">file_get_contents</span><span class="p">(</span><span class="nv">$path</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="nv">$raw</span> <span class="o">===</span> <span class="kc">false</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="nv">$data</span> <span class="o">=</span> <span class="nb">json_decode</span><span class="p">(</span><span class="nv">$raw</span><span class="p">,</span> <span class="kc">true</span><span class="p">);</span>
        <span class="k">return</span> <span class="nb">is_array</span><span class="p">(</span><span class="nv">$data</span><span class="p">)</span> <span class="o">?</span> <span class="nv">$data</span> <span class="o">:</span> <span class="kc">null</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Encode and save data to JSON file.
     *
     * @param string $path
     * @param array $data
     * @throws RuntimeException
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">saveJsonFile</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$path</span><span class="p">,</span> <span class="kt">array</span> <span class="nv">$data</span><span class="p">):</span> <span class="kt">void</span>
    <span class="p">{</span>
        <span class="nv">$json</span> <span class="o">=</span> <span class="nb">json_encode</span><span class="p">(</span><span class="nv">$data</span><span class="p">,</span> <span class="no">JSON_UNESCAPED_UNICODE</span> <span class="o">|</span> <span class="no">JSON_PRETTY_PRINT</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="nv">$json</span> <span class="o">===</span> <span class="kc">false</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nc">RuntimeException</span><span class="p">(</span><span class="s1">'Failed to encode JSON'</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="nv">$ok</span> <span class="o">=</span> <span class="nb">file_put_contents</span><span class="p">(</span><span class="nv">$path</span><span class="p">,</span> <span class="nv">$json</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="nv">$ok</span> <span class="o">===</span> <span class="kc">false</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nc">RuntimeException</span><span class="p">(</span><span class="s1">'Failed to write cache file: '</span> <span class="mf">.</span> <span class="nv">$path</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Load cached event embeddings only if they were produced by the current model.
     *
     * @return void
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">loadEmbeddingsFromCacheIfCompatible</span><span class="p">():</span> <span class="kt">void</span>
    <span class="p">{</span>
        <span class="nv">$cached</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">loadJsonFile</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">cachePath</span><span class="p">);</span>

        <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="nb">is_array</span><span class="p">(</span><span class="nv">$cached</span><span class="p">)</span> <span class="o">||</span> <span class="o">!</span><span class="k">isset</span><span class="p">(</span><span class="nv">$cached</span><span class="p">[</span><span class="s1">'model'</span><span class="p">],</span> <span class="nv">$cached</span><span class="p">[</span><span class="s1">'events'</span><span class="p">])</span> <span class="o">||</span> <span class="o">!</span><span class="nb">is_array</span><span class="p">(</span><span class="nv">$cached</span><span class="p">[</span><span class="s1">'events'</span><span class="p">]))</span> <span class="p">{</span>
            <span class="k">return</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">if</span> <span class="p">(</span><span class="nv">$cached</span><span class="p">[</span><span class="s1">'model'</span><span class="p">]</span> <span class="o">!==</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">model</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$cached</span><span class="p">[</span><span class="s1">'events'</span><span class="p">]</span> <span class="k">as</span> <span class="nv">$row</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">if</span> <span class="p">(</span><span class="k">isset</span><span class="p">(</span><span class="nv">$row</span><span class="p">[</span><span class="s1">'id'</span><span class="p">],</span> <span class="nv">$row</span><span class="p">[</span><span class="s1">'embedding'</span><span class="p">])</span> <span class="o">&amp;&amp;</span> <span class="nb">is_array</span><span class="p">(</span><span class="nv">$row</span><span class="p">[</span><span class="s1">'embedding'</span><span class="p">]))</span> <span class="p">{</span>
                <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span><span class="p">[(</span><span class="n">int</span><span class="p">)</span> <span class="nv">$row</span><span class="p">[</span><span class="s1">'id'</span><span class="p">]]</span> <span class="o">=</span> <span class="nv">$row</span><span class="p">[</span><span class="s1">'embedding'</span><span class="p">];</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Ensure embeddings exist for all events and persist them to cache.
     *
     * @return void
     * @throws RuntimeException
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">ensureAllEventEmbeddings</span><span class="p">():</span> <span class="kt">void</span>
    <span class="p">{</span>
        <span class="nv">$missing</span> <span class="o">=</span> <span class="p">[];</span>
        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">events</span> <span class="k">as</span> <span class="nv">$event</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$id</span> <span class="o">=</span> <span class="p">(</span><span class="n">int</span><span class="p">)</span> <span class="nv">$event</span><span class="p">[</span><span class="s1">'id'</span><span class="p">];</span>
            <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="k">isset</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span><span class="p">[</span><span class="nv">$id</span><span class="p">]))</span> <span class="p">{</span>
                <span class="nv">$missing</span><span class="p">[]</span> <span class="o">=</span> <span class="nv">$event</span><span class="p">;</span>
            <span class="p">}</span>
        <span class="p">}</span>

        <span class="k">if</span> <span class="p">(</span><span class="nb">count</span><span class="p">(</span><span class="nv">$missing</span><span class="p">)</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$missing</span> <span class="k">as</span> <span class="nv">$event</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$id</span> <span class="o">=</span> <span class="p">(</span><span class="n">int</span><span class="p">)</span> <span class="nv">$event</span><span class="p">[</span><span class="s1">'id'</span><span class="p">];</span>
            <span class="nv">$text</span> <span class="o">=</span> <span class="p">(</span><span class="n">string</span><span class="p">)</span> <span class="nv">$event</span><span class="p">[</span><span class="s1">'description'</span><span class="p">];</span>

            <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span><span class="p">[</span><span class="nv">$id</span><span class="p">]</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">embedText</span><span class="p">(</span><span class="nv">$text</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="nv">$toCache</span> <span class="o">=</span> <span class="p">[</span>
            <span class="s1">'model'</span> <span class="o">=&gt;</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">model</span><span class="p">,</span>
            <span class="s1">'events'</span> <span class="o">=&gt;</span> <span class="nb">array_values</span><span class="p">(</span><span class="nb">array_map</span><span class="p">(</span>
                <span class="k">fn</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$event</span><span class="p">):</span> <span class="kt">array</span> <span class="o">=&gt;</span> <span class="p">[</span>
                    <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="n">int</span><span class="p">)</span> <span class="nv">$event</span><span class="p">[</span><span class="s1">'id'</span><span class="p">],</span>
                    <span class="s1">'embedding'</span> <span class="o">=&gt;</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span><span class="p">[(</span><span class="n">int</span><span class="p">)</span> <span class="nv">$event</span><span class="p">[</span><span class="s1">'id'</span><span class="p">]],</span>
                <span class="p">],</span>
                <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">events</span>
            <span class="p">)),</span>
        <span class="p">];</span>

        <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">saveJsonFile</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">cachePath</span><span class="p">,</span> <span class="nv">$toCache</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Score all events against the query embedding and return the top-N results.
     *
     * @param list&lt;float|int&gt; $queryVec
     * @return list&lt;array{score:float,event:array{id:int,title:string,description:string}}&gt;
     */</span>
    <span class="k">private</span> <span class="k">function</span> <span class="n">search</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$queryVec</span><span class="p">):</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="nv">$scored</span> <span class="o">=</span> <span class="p">[];</span>

        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">events</span> <span class="k">as</span> <span class="nv">$event</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$id</span> <span class="o">=</span> <span class="p">(</span><span class="n">int</span><span class="p">)</span> <span class="nv">$event</span><span class="p">[</span><span class="s1">'id'</span><span class="p">];</span>
            <span class="nv">$score</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">cosineSimilarity</span><span class="p">(</span><span class="nv">$queryVec</span><span class="p">,</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">eventEmbeddingsById</span><span class="p">[</span><span class="nv">$id</span><span class="p">]);</span>

            <span class="nv">$scored</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[</span>
                <span class="s1">'score'</span> <span class="o">=&gt;</span> <span class="nv">$score</span><span class="p">,</span>
                <span class="s1">'event'</span> <span class="o">=&gt;</span> <span class="nv">$event</span><span class="p">,</span>
            <span class="p">];</span>
        <span class="p">}</span>

        <span class="nb">usort</span><span class="p">(</span><span class="nv">$scored</span><span class="p">,</span> <span class="k">static</span> <span class="k">fn</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$a</span><span class="p">,</span> <span class="kt">array</span> <span class="nv">$b</span><span class="p">):</span> <span class="kt">int</span> <span class="o">=&gt;</span> <span class="nv">$b</span><span class="p">[</span><span class="s1">'score'</span><span class="p">]</span> <span class="o">&lt;=&gt;</span> <span class="nv">$a</span><span class="p">[</span><span class="s1">'score'</span><span class="p">]);</span>

        <span class="k">return</span> <span class="nb">array_slice</span><span class="p">(</span><span class="nv">$scored</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">topN</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="cd">/**
     * Render results as plain text.
     *
     * @param string $query
     * @param list&lt;array{score:float,event:array{id:int,title:string,description:string}}&gt; $results
     * @return void
     */</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">render</span><span class="p">(</span><span class="kt">string</span> <span class="nv">$query</span><span class="p">,</span> <span class="kt">array</span> <span class="nv">$results</span><span class="p">):</span> <span class="kt">void</span>
    <span class="p">{</span>
        <span class="k">echo</span> <span class="s2">"Query: </span><span class="si">{</span><span class="nv">$query</span><span class="si">}</span><span class="se">\n\n</span><span class="s2">"</span><span class="p">;</span>
        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$results</span> <span class="k">as</span> <span class="nv">$row</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$event</span> <span class="o">=</span> <span class="nv">$row</span><span class="p">[</span><span class="s1">'event'</span><span class="p">];</span>
            <span class="nv">$score</span> <span class="o">=</span> <span class="p">(</span><span class="n">float</span><span class="p">)</span> <span class="nv">$row</span><span class="p">[</span><span class="s1">'score'</span><span class="p">];</span>

            <span class="k">echo</span> <span class="s2">"["</span> <span class="mf">.</span> <span class="nb">number_format</span><span class="p">(</span><span class="nv">$score</span><span class="p">,</span> <span class="mi">4</span><span class="p">)</span> <span class="mf">.</span> <span class="s2">"] #</span><span class="si">{</span><span class="nv">$event</span><span class="p">[</span><span class="s1">'id'</span><span class="p">]</span><span class="si">}</span><span class="s2"> </span><span class="si">{</span><span class="nv">$event</span><span class="p">[</span><span class="s1">'title'</span><span class="p">]</span><span class="si">}</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span>
            <span class="k">echo</span> <span class="s2">"  </span><span class="si">{</span><span class="nv">$event</span><span class="p">[</span><span class="s1">'description'</span><span class="p">]</span><span class="si">}</span><span class="se">\n\n</span><span class="s2">"</span><span class="p">;</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Data Preparing
</h2>

<p>Let's assume this is our data, collected from various sources. For simplicity, let's place it in an array.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nv">$events</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">1</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Restrictions Against Technology Corporations'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'New economic measures have been introduced against major technology companies.'</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">2</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Development of Space Programs'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'Several countries in the region have increased funding for national satellite projects.'</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">3</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Escalation of Political Conflict'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'An escalation of politically motivated conflict has occurred in several provinces.'</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">4</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Restrictions Against the IT Sector'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'The government announced new restrictions for companies operating in the information technology sector.'</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">5</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Rising Inflation and Key Interest Rate Review'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'The central bank raised the key interest rate amid accelerating inflation and rising prices of imported goods.'</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">6</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Launch of a Small Business Support Program'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'Authorities announced preferential loans and tax incentives for small and medium-sized businesses in the regions.'</span><span class="p">,</span>
    <span class="p">],</span>
    <span class="p">[</span>
        <span class="s1">'id'</span> <span class="o">=&gt;</span> <span class="mi">8</span><span class="p">,</span>
        <span class="s1">'title'</span> <span class="o">=&gt;</span> <span class="s1">'Data Breach in Online Retail'</span><span class="p">,</span>
        <span class="s1">'description'</span> <span class="o">=&gt;</span> <span class="s1">'An online retailer is investigating a leak of customers'</span> <span class="n">personal</span> <span class="n">data</span> <span class="n">following</span> <span class="n">the</span> <span class="n">compromise</span> <span class="n">of</span> <span class="n">employee</span> <span class="n">accounts</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 9,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Medical</span> <span class="nc">Breakthrough</span><span class="o">:</span> <span class="k">New</span> <span class="nc">Diagnostic</span> <span class="nc">Method</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">Researchers</span> <span class="n">presented</span> <span class="n">a</span> <span class="n">method</span> <span class="k">for</span> <span class="n">early</span> <span class="n">disease</span> <span class="n">diagnosis</span> <span class="n">using</span> <span class="n">biomarkers</span><span class="p">,</span> <span class="n">significantly</span> <span class="n">reducing</span> <span class="n">analysis</span> <span class="n">time</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 10,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Seasonal</span> <span class="nc">Increase</span> <span class="n">in</span> <span class="nc">Illness</span> <span class="nc">Rates</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">Several</span> <span class="n">cities</span> <span class="n">have</span> <span class="n">reported</span> <span class="n">a</span> <span class="n">rise</span> <span class="n">in</span> <span class="n">respiratory</span> <span class="n">infections</span><span class="p">,</span> <span class="n">with</span> <span class="n">clinics</span> <span class="n">increasing</span> <span class="n">patient</span> <span class="n">intake</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 12,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Drought</span> <span class="k">and</span> <span class="nc">Risks</span> <span class="n">to</span> <span class="nc">Agriculture</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">Due</span> <span class="n">to</span> <span class="n">prolonged</span> <span class="n">drought</span><span class="p">,</span> <span class="n">farmers</span> <span class="n">predict</span> <span class="n">lower</span> <span class="n">crop</span> <span class="n">yields</span><span class="p">,</span> <span class="k">and</span> <span class="n">support</span> <span class="n">measures</span> <span class="k">for</span> <span class="n">agriculture</span> <span class="n">are</span> <span class="n">being</span> <span class="n">discussed</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 13,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="k">Final</span> <span class="n">of</span> <span class="n">a</span> <span class="nc">Major</span> <span class="nc">Sports</span> <span class="nc">Tournament</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">In</span> <span class="n">the</span> <span class="n">decisive</span> <span class="k">match</span> <span class="n">of</span> <span class="n">the</span> <span class="n">season</span><span class="p">,</span> <span class="n">the</span> <span class="n">team</span> <span class="n">secured</span> <span class="n">victory</span> <span class="n">in</span> <span class="n">extra</span> <span class="n">time</span><span class="p">,</span> <span class="n">setting</span> <span class="n">a</span> <span class="k">new</span> <span class="nc">attendance</span> <span class="n">record</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 14,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Player</span> <span class="nc">Transfer</span> <span class="k">and</span> <span class="nc">Squad</span> <span class="nc">Reinforcement</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">The</span> <span class="n">club</span> <span class="n">signed</span> <span class="n">a</span> <span class="k">new</span> <span class="nc">striker</span><span class="p">,</span> <span class="n">aiming</span> <span class="n">to</span> <span class="n">strengthen</span> <span class="n">its</span> <span class="n">attacking</span> <span class="n">line</span> <span class="n">ahead</span> <span class="n">of</span> <span class="n">a</span> <span class="n">series</span> <span class="n">of</span> <span class="n">derby</span> <span class="n">matches</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 15,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="k">New</span> <span class="nc">Rules</span> <span class="k">for</span> <span class="nc">Marketplaces</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">The</span> <span class="n">regulator</span> <span class="n">proposed</span> <span class="n">requirements</span> <span class="k">for</span> <span class="n">product</span> <span class="n">labeling</span> <span class="k">and</span> <span class="n">commission</span> <span class="n">transparency</span> <span class="n">on</span> <span class="n">online</span> <span class="n">trading</span> <span class="n">platforms</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 17,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Disruptions</span> <span class="n">in</span> <span class="nc">Semiconductor</span> <span class="nc">Supply</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">Electronics</span> <span class="n">manufacturers</span> <span class="n">warned</span> <span class="n">of</span> <span class="n">chip</span> <span class="n">supply</span> <span class="n">delays</span> <span class="n">due</span> <span class="n">to</span> <span class="n">export</span> <span class="n">restrictions</span> <span class="k">and</span> <span class="n">factory</span> <span class="n">overloads</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 18,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Opening</span> <span class="n">of</span> <span class="n">a</span> <span class="nc">Contemporary</span> <span class="nc">Art</span> <span class="nc">Festival</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">A</span> <span class="n">contemporary</span> <span class="n">art</span> <span class="n">festival</span> <span class="n">has</span> <span class="n">opened</span> <span class="n">in</span> <span class="n">the</span> <span class="n">capital</span><span class="p">,</span> <span class="n">featuring</span> <span class="n">exhibitions</span><span class="p">,</span> <span class="n">performances</span><span class="p">,</span> <span class="k">and</span> <span class="n">artist</span> <span class="n">lectures</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 19,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Major</span> <span class="nc">Deal</span> <span class="n">in</span> <span class="n">the</span> <span class="nc">Real</span> <span class="nc">Estate</span> <span class="nc">Market</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">An</span> <span class="n">investment</span> <span class="n">fund</span> <span class="n">acquired</span> <span class="n">a</span> <span class="n">portfolio</span> <span class="n">of</span> <span class="n">commercial</span> <span class="n">real</span> <span class="n">estate</span><span class="p">,</span> <span class="n">planning</span> <span class="n">renovation</span> <span class="k">and</span> <span class="n">improved</span> <span class="n">energy</span> <span class="n">efficiency</span><span class="mf">.</span><span class="s1">',
    ],
    [
        '</span><span class="n">id</span><span class="s1">' =&gt; 20,
        '</span><span class="n">title</span><span class="s1">' =&gt; '</span><span class="nc">Ocean</span> <span class="nc">Research</span> <span class="k">and</span> <span class="k">New</span> <span class="nc">Findings</span><span class="s1">',
        '</span><span class="n">description</span><span class="s1">' =&gt; '</span><span class="nc">A</span> <span class="n">scientific</span> <span class="n">expedition</span> <span class="n">collected</span> <span class="n">data</span> <span class="n">on</span> <span class="n">ocean</span> <span class="n">currents</span> <span class="k">and</span> <span class="n">water</span> <span class="n">temperature</span><span class="p">,</span> <span class="n">refining</span> <span class="n">climate</span> <span class="n">change</span> <span class="n">forecasts</span><span class="mf">.</span><span class="err">'</span><span class="p">,</span>
    <span class="p">],</span>
<span class="p">];</span>
</code></pre>

</div>



<h2>
  
  
  Run the example
</h2>

<p>It's simple here: just run our code and wait for the results.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">require_once</span> <span class="k">__DIR__</span> <span class="mf">.</span> <span class="s1">'/../vendor/autoload.php'</span><span class="p">;</span>

<span class="kn">use</span> <span class="k">function</span> <span class="n">Codewithkyrian\Transformers\Pipelines\pipeline</span><span class="p">;</span>

<span class="k">final</span> <span class="kd">class</span> <span class="nc">SemanticEventSearch</span> <span class="p">{</span><span class="mf">...</span><span class="p">}</span>

<span class="nv">$events</span> <span class="o">=</span> <span class="p">[</span><span class="mf">...</span><span class="p">];</span>

<span class="nv">$query</span> <span class="o">=</span> <span class="s1">'sanctions against IT companies'</span><span class="p">;</span>

<span class="nv">$search</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">SemanticEventSearch</span><span class="p">(</span><span class="n">topN</span><span class="o">:</span> <span class="mi">3</span><span class="p">);</span>
<span class="nv">$search</span><span class="o">-&gt;</span><span class="nf">setModel</span><span class="p">(</span><span class="s1">'Xenova/paraphrase-multilingual-MiniLM-L12-v2'</span><span class="p">);</span>
<span class="nv">$search</span><span class="o">-&gt;</span><span class="nf">setEvents</span><span class="p">(</span><span class="nv">$events</span><span class="p">);</span>
<span class="nv">$search</span><span class="o">-&gt;</span><span class="nf">setQuery</span><span class="p">(</span><span class="nv">$query</span><span class="p">);</span>
<span class="nv">$out</span> <span class="o">=</span> <span class="nv">$search</span><span class="o">-&gt;</span><span class="nf">run</span><span class="p">();</span>
<span class="nv">$search</span><span class="o">-&gt;</span><span class="nf">render</span><span class="p">(</span><span class="nv">$out</span><span class="p">[</span><span class="s1">'query'</span><span class="p">],</span> <span class="nv">$out</span><span class="p">[</span><span class="s1">'results'</span><span class="p">]);</span>
</code></pre>

</div>



<h2>
  
  
  Result
</h2>

<p>The output matches by meaning, not words. The first result is the most suitable for the population with our request.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Query: sanctions against IT companies
[0.4288] #4 Restrictions against the IT sector
  The government announced new restrictions for companies operating in the IT industry.
[0.3356] #15 New rules for marketplaces
  The regulator proposed requirements for product labeling and commission transparency.
[0.2598] #8 Data breach in online retail
  An online store investigates a customer data leak after account compromise.
</code></pre>

</div>



<h2>
  
  
  Production Use Cases
</h2>

<p>This already looks like a product approach, not an experiment.<br>
And it has useful properties:</p>

<ul>
<li>weak dependence on exact wording</li>
<li>works well with short texts</li>
<li>easily combined with classic filters (date, region, type)</li>
</ul>

<p>This is not "AI for the sake of AI", but practical, explainable, debuggable logic.</p>

<h2>
  
  
  Limitations and Caveats
</h2>

<p>This is not a silver bullet.</p>

<p>Models are large. Performance matters. Some problems are still better solved with plain SQL.</p>

<p>But that's a normal engineering discussion - about trade-offs, not magic or black boxes.</p>




<h2>
  
  
  Where to Go Next
</h2>

<p>For me, TransformersPHP is a strong example that AI can be used directly in PHP projects without changing the stack or introducing Python.</p>

<p>In my open and free book "AI for PHP Developers", I explore similar cases: when it makes sense, how to choose models, and how not to turn a project into a collection of experimental features.</p>

<p>All examples can be downloaded and run using a ready-made Docker environment from GitHub repository: <br>
<a href="https://github.com/apphp/ai-for-php-developers-examples" rel="noopener noreferrer">https://github.com/apphp/ai-for-php-developers-examples</a></p>

<p>You can also run all examples from the book directly: <br>
<a href="https://aiwithphp.org/books/ai-for-php-developers/examples/set-lang/en" rel="noopener noreferrer">https://aiwithphp.org/books/ai-for-php-developers/examples</a></p>

<p>If the topic resonates - I'd be glad to discuss and get feedback.<br>
Especially interesting to hear what tasks you already solve (or want to solve) with AI in PHP projects.</p>

