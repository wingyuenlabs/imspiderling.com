---
Title: Training GitHub Repository Embeddings using Stars
Description: 
Author: Dmitry Nikitko
Date: 2026-01-02T21:55:07.000Z
Robots: noindex,nofollow
Template: index
---
<ul>
<li>
<a href="https://puzer.github.io/github_recommender/" rel="noopener noreferrer">Demo. 100% Client-side. WASM for KNN</a>

<ul>
<li>Generates a skill radar based on your GitHub Stars</li>
<li>Сompares your profile with famous developers</li>
<li>Recommend repos you might like</li>
</ul>


</li>

<li><a href="https://github.com/Puzer/github-repo-embeddings" rel="noopener noreferrer">Sources + Dataset</a></li>

</ul>

<h1>
  
  
  TL;DR
</h1>

<ul>
<li>
<strong>The Idea:</strong> People use GitHub Stars as bookmarks. This is an excellent signal for understanding which repositories are semantically similar.</li>
<li>
<strong>The Data:</strong> Processed ~1TB of raw data from GitHub Archive (BigQuery) to build an interest matrix of 4 million developers.</li>
<li>
<strong>The ML:</strong> Trained embeddings for 300k+ repositories using Metric Learning (EmbeddingBag + MultiSimilarityLoss).</li>
<li>
<strong>The Frontend:</strong> Built a client-only demo that runs vector search (KNN) directly in the browser via WASM, with no backend involved.</li>
<li>
<strong>The Result:</strong> The system finds non-obvious library alternatives and allows for semantic comparison of developer profiles.</li>
</ul>




<h1>
  
  
  Personal Motivation
</h1>

<p>Finishing ideas is usually harder than it looks. It's easy to build a prototype, but the real struggle begins afterwards: polishing the rough edges, writing the text, and setting up the demo. This project is my attempt to go the full distance and tie up one of these "loose ends."</p>

<p>I also started thinking about the nature of our GitHub Stars. We are used to treating them simply as bookmarks "for later." But in reality, they are a valuable digital asset. They are a snapshot of our professional interests and skills. I wondered: can we put this passive asset to work? Can we use the accumulated knowledge of millions of developers to build a repository recommendation system and allow people to compare their technical interests?</p>




<h1>
  
  
  The Concept
</h1>

<h2>
  
  
  Cluster Hypothesis
</h2>

<p>The people reading this article are much more similar to you in interests than a randomly selected person from the street. In our universe, similar things often appear in similar contexts.</p>

<p>We intuitively sense these "hidden" preferences. If you see a new colleague typing in Vim and successfully exiting it without help, you've probably already built a mental vector of their interests: you can likely discuss patching KDE2 under FreeBSD with them, but asking for advice on an RGB gaming mouse might be a miss.</p>

<h2>
  
  
  Repo Representation
</h2>

<p>Let's apply this to repositories. We want to obtain a space where semantically similar repos are located close to each other.</p>

<p>To simplify, imagine a 2D space where the position depends on two main characteristics:</p>

<ul>
<li>
<strong>Axis X:</strong> Data (Preparation &amp; Analysis) vs Models (Training &amp; Inference).</li>
<li>
<strong>Axis Y:</strong> Local / Single-node vs Big Data / Cluster.</li>
</ul>

<p>
  <a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2FPuzer%2Fgithub-repo-embeddings%2Fraw%2Fmain%2Fimages%2Fimg_1.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2FPuzer%2Fgithub-repo-embeddings%2Fraw%2Fmain%2Fimages%2Fimg_1.jpg" alt="semantic quadrants" width="800" height="339"></a>
</p>

<p>In reality, the neural network learns these axes (features) itself. They aren't always interpretable by humans, but the mathematical essence remains: similar repos are pulled together into clusters based on some learned features.</p>

<p>With these vectors, we can:</p>

<ul>
<li>  Search for similar repositories using <strong>Cosine Similarity</strong> (measuring the angle between vectors).</li>
<li>  Obtain a user interest vector by simply averaging the vectors of their <strong>starred</strong> repositories.</li>
<li>  Compare user profiles with each other.</li>
</ul>




<h1>
  
  
  Signal Source
</h1>

<p>To get quality vectors, I used a hybrid approach.</p>

<h3>
  
  
  1. Text (README.md) - For Initialization
</h3>

<p>Many repositories have a README. This is a great source for a "cold start". I used the <a href="https://github.com/QwenLM/Qwen3-Embedding/blob/44548aa5f0a0aed1c76d64e19afe47727a325b8f/examples/qwen3_embedding_transformers.py#L54" rel="noopener noreferrer"><strong>Qwen3-Embedding-0.6B</strong></a> model, which supports <strong>MRL (Matryoshka Representation Learning)</strong>, keeping only the first <strong>128D</strong> (the most important components). I used these vectors as the <em>initial weight initialization</em> for the trainable model.<br><br>
<em>Note:</em> This step adds about 10% to the final quality. To avoid overcomplicating the code with extra dependencies, I skipped this step in the public repo—the model learns perfectly fine from scratch (random init), just a bit slower.</p>

<h3>
  
  
  2. Stars Matrix - For Main Training
</h3>

<p>Text is good, but it doesn't show how tools are used <em>together</em>. This is where collaborative filtering comes into play.</p>

<blockquote>
<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>User</th>
<th>Starred repositories</th>
</tr>
</thead>
<tbody>
<tr>
<td>A</td>
<td>Pandas, Dask, SK-Learn, Numpy</td>
</tr>
<tr>
<td>B</td>
<td>Vue, React, TypeScript, Vite</td>
</tr>
</tbody>
</table></div>
</blockquote>

<p>There are many approaches to training on such data: graph algorithms (LightGCN) or matrix factorization. I chose <strong>Metric Learning</strong> because it requires fewer GPU resources (1xGPU with ~8GB) and offers flexibility in managing the vector space.</p>




<h1>
  
  
  Data Preparation
</h1>

<p>Data was sourced from the public <a href="https://www.gharchive.org/#bigquery" rel="noopener noreferrer">GitHub Archive</a> dataset in BigQuery.<br>
I needed two queries:</p>

<ol>
<li> <strong>Stars (WatchEvent):</strong> Collect users with 10 to 800 stars (filtering out bots and inactive users). Preserving the order of stars.</li>
<li> <strong>Meta (PushEvent):</strong> Collect repository names, commit dates, and descriptions.</li>
</ol>

<p>In total, the queries processed about 1 TB of data and almost fit within the BigQuery Free Tier. The output was a Parquet file with ~4 million users and ~2.5 million unique repositories.</p>




<h1>
  
  
  Training Vectors
</h1>

<h2>
  
  
  Model Choice
</h2>

<p>I wanted to make the solution as lightweight as possible for the browser, so I immediately ruled out Transformers.<br>
My model is a classic <code>torch.nn.EmbeddingBag</code>. Essentially, it's just a large lookup table <code>repo_id -&gt; vector[128]</code> that can efficiently aggregate (average) vectors.</p>

<h2>
  
  
  Sampling and Loss Function
</h2>

<p>How do we explain to the neural network that Pandas and Numpy are "close"?</p>

<p>I split each user's list of stars into two random, non-overlapping "buckets" and used <code>EmbeddingBag</code> to calculate aggregated embedding of each bucket.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>User</th>
<th>Bucket</th>
<th>Repos in bucket</th>
<th>Mean(Embeddings in Bucket)</th>
</tr>
</thead>
<tbody>
<tr>
<td>A</td>
<td>A1</td>
<td>Numpy, Dask, SciPy</td>
<td>[0.2, -1.1, 0.9, ...]</td>
</tr>
<tr>
<td>A</td>
<td>A2</td>
<td>Pandas, SK-Learn</td>
<td>[0.1, -1.3, 0.6, ...]</td>
</tr>
<tr>
<td>B</td>
<td>B1</td>
<td>Vue, Vite</td>
<td>[-0.4, 0.6, 0.2, ...]</td>
</tr>
<tr>
<td>B</td>
<td>B2</td>
<td>React, TypeScript</td>
<td>[-0.3, 0.7, 0.1, ...]</td>
</tr>
</tbody>
</table></div>

<p>We train embeddings so that two conditions are met <em>simultaneously</em>:</p>

<ol>
<li> Buckets from the same user are as close as possible (A1 &lt;-pull-&gt; A2).</li>
<li> Buckets from different users are as far apart as possible (B1 &lt;-push-&gt; A2).</li>
</ol>

<p>Obviously, the model cannot perfectly satisfy both conditions simultaneously for all users. However, by balancing these two forces, gradient descent attempts to adjust the repository vectors to minimize the overall error.</p>

<p>For the loss function, <a href="https://kevinmusgrave.github.io/pytorch-metric-learning/losses/#multisimilarityloss" rel="noopener noreferrer"><strong>MultiSimilarityLoss</strong></a> from the <code>pytorch-metric-learning</code> library performed the best.</p>

<blockquote>
<p>Note: Here, we should pay tribute to <a href="https://github.com/facebookresearch/StarSpace" rel="noopener noreferrer">StarSpace</a>, which pioneered this concept 8 years ago.</p>
</blockquote>

<h2>
  
  
  Advanced Methods vs. Simplicity
</h2>

<p>It was logical to assume that the <em>sequence</em> of stars (the order in which you star repos) holds a strong signal, so I tried approaches à la Word2Vec (sliding window).</p>

<p>Surprisingly, <strong>the simplest random split worked best</strong>. Perhaps the timing data is too noisy, or I simply failed to extract value from it.</p>

<p>I also tried Hard Negative Miners, some other losses e.g. NTXentLoss (uses 4x more memory than MS, CrossBatchMemory didn't help) - but nothing managed to beat the original baseline.</p>

<p>Sometimes in ML, Occam's razor wins. And sometimes you realize that either the razor is dull, or you are.</p>




<h1>
  
  
  Quality Evaluation
</h1>

<p>We have vectors, but are they any good?<br><br>
One could use synthetic data from an LLM, but I found a more elegant Ground Truth—<strong>Awesome Lists</strong>. There are thousands of repositories on GitHub like "Awesome Python" or "Awesome React." These are human-curated clusters of similar libraries.<br>
I downloaded the READMEs of these lists, found collocations (which repos appear together), applied heuristic weighting, and used the NDCG metric to evaluate ranking. This allowed me to fairly compare different loss functions, hyperparameters, and sampling methods.</p>




<h1>
  
  
  Frontend: Showcase &amp; AI-Assisted Dev
</h1>

<p>Despite my 10 years of experience in Data Science, I am not an expert in frontend development. So the challenge was: build complex client-side logic without a backend, while not being a JS developer.</p>

<p>The entire frontend and "glue" code was written with the help of an <strong>AI Coding Agent</strong>.</p>

<h2>
  
  
  Architecture
</h2>

<ol>
<li> <strong>Data:</strong> The client downloads compressed embeddings (FP16, ~80 MB) and metadata, caching them in IndexedDB.</li>
<li> <strong>Search (WASM):</strong> Uses the core of the <a href="https://github.com/unum-cloud/USearch" rel="noopener noreferrer">USearch</a> library, compiled to WebAssembly.</li>
</ol>

<h2>
  
  
  Low-level Magic
</h2>

<p>Initially, I wanted to use a pre-calculated HNSW index, but it consumed more memory than raw embeddings.<br>
So I asked the agent to try implementing Exact Search (still using WASM).<br>
The agent found the low-level <code>_usearch_exact_search</code> methods and generated a worker (<code>coreWorker.js</code>) that manually manages memory, allocates buffers via <code>_malloc</code>, and juggles pointers.<br>
Browsers still don't handle native FP16 well, so the agent also had to write an on-the-fly FP16 -&gt; FP32 converter for reading vectors. To me, this looks like magic, but it works fast even on 300k vectors, without HNSW indexes.</p>

<h2>
  
  
  User Profile &amp; Skill Radar
</h2>

<p>Any demo becomes more engaging if it reveals something personal about the user. That’s why I added profile analytics.</p>

<h3>
  
  
  User Embedding
</h3>

<p>The math here is simple but effective:</p>

<ol>
<li> The client queries the GitHub API to fetch your starred repositories.</li>
<li> We take the embeddings of these repositories and average them.</li>
<li> The resulting <strong>mean vector</strong> is a digital fingerprint of your interests. Since it resides in the same metric space as the repositories, we can search for libraries that are "nearest" to you.</li>
</ol>

<h3>
  
  
  Skill Radar (Interpreting the Vector)
</h3>

<p>Staring at 128 raw numbers is boring. I wanted to visualize skills RPG-style.<br>
To create the radar axes (e.g., "GenAI", "Web3", "System Programming"), I used an LLM trick:</p>

<ol>
<li> I asked an LLM to generate lists of 20 "reference" repositories for 10 different categories.</li>
<li> I trained simple <strong>Logistic Regressions (Linear Probes)</strong> to distinguish the vectors of these categories.</li>
<li> Now, the browser simply passes the user vector through these models to generate probability scores for the chart.</li>
<li>It works since repos and user embeddings lives in the same space.</li>
</ol>

<h3>
  
  
  Serverless Sharing
</h3>

<p>To add a social element, I pre-calculated vectors for famous developers.<br>
The comparison uses Cosine Similarity, but with a twist. People struggle with abstract metrics (what does "0.6 similarity" actually mean?).<br>
So, I applied a <strong>Quantile Transformation</strong>: instead of raw cosine scores, we display percentiles. A "95% Match" means you are more similar to this person than 95% of random pairs.</p>

<p>To let you compare yourself with other people, I implemented <strong>Serverless Sharing</strong>: the user vector is compressed, Base64-encoded, and embedded directly into the URL fragment identifier (hash). No database, no backend—just pure client-side math.</p>




<h1>
  
  
  Results: Expectations vs. Reality
</h1>

<p>Besides metrics, I verified the model with the "eyeball test."</p>

<p><strong>What didn't work:</strong><br>
I hoped for beautiful vector arithmetic, like in NLP (<code>King - Man + Woman = Queen</code>).<br>
Hypothesis: <code>Pandas - Python + TypeScript = Danfo.js</code>.<br>
Reality: This didn't work. The repository vector space turned out to be more complex, and simple linear operations don't yield such interpretable results.<br>
Also, I hoped for a distinct cluster structure of embeddings, but unfortunately, it doesn't look very pronounced visually.</p>

<p><strong>What worked:</strong><br>
The main goal was achieved—the search finds alternatives I didn't know about, but which are semantically relevant.<br>
Unlike LLMs, which often have a bias towards the most popular solutions, this approach, based on the <em>behavior</em> of IT professionals, digs up:</p>

<ol>
<li> <strong>Niche Tools:</strong> Libraries used by pros but rarely written about in blogs.</li>
<li> <strong>Fresh Solutions:</strong> Repositories that gained popularity recently and share a similar "starring pattern."</li>
<li> <strong>Local-first:</strong> Everything runs locally on client devices.</li>
</ol>

<h1>
  
  
  Future Vision
</h1>

<p>The current demo is showing what is possible without a backend. But you can think about other use-cases:</p>

<ol>
<li><p><strong>Semantic Text Search:</strong><br>
Theoretically, one could take a text encoder and train a projection layer into the repository embedding space. This would allow searching for tools or people by abstract description.</p></li>
<li>
<p><strong>GitHub Tinder (Networking):</strong><br>
With user vectors, we can match people.</p>

<ul>
<li>Looking for a mentor or co-founder? The algorithm finds a person with a complementary stack.</li>
<li>Looking for a contributor? You can find developers who actively "star" similar projects but haven't seen yours yet.</li>
<li>HR Tech use-cases for position/candidate matching</li>
</ul>
</li>
<li><p><strong>Trend Analytics:</strong><br>
By adding a time dimension, we can visualize the evolution of technology. We could spot the birth of a new trend before it hits Hacker News. Though, this might be a chicken and egg problem :)</p></li>
</ol>

<h1>
  
  
  Final Words
</h1>

<p>After 3+ years of working with LLMs, I wanted to take a step back, "shake off the rust" and revisit classical approaches. I also wanted to show that "good old" embeddings can still be useful in a world that seems to revolve entirely around GenAI.</p>

