---
Title: Building EigenAI: Teaching Math Foundations of AI Through Interactive Code
Description: 
Author: Luis Faria
Date: 2025-12-06T21:29:32.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>From determinants to hill climbing algorithms—how I turned academic math into an interactive learning platform.</strong></p>

<blockquote>
<p><em>"Whether it's concrete or code, structure is everything."</em></p>
</blockquote>




<h2>
  
  
  🎓 The Challenge: Making Math "Click"
</h2>

<p>As a self-taught software engineer transitioning from 10+ years in project management, I enrolled in <strong>MFA501 – Mathematical Foundations of Artificial Intelligence</strong> at Torrens University Australia under <a href="https://au.linkedin.com/in/james-v-70183b28" rel="noopener noreferrer">Dr. James Vakilian</a>. The subject covered everything from linear algebra to optimization algorithms—the mathematical backbone of modern AI applications in:</p>

<ul>
<li>
<strong>Machine Learning</strong> (model training, optimization)</li>
<li>
<strong>Natural Language Processing</strong> (text embeddings, transformations)</li>
<li>
<strong>Computer Vision</strong> (image processing, feature extraction)</li>
<li>
<strong>Speech Recognition</strong> (signal processing, pattern matching)</li>
</ul>

<p>But here's the problem: <strong>abstract math doesn't stick unless you build something with it.</strong></p>

<p>So instead of just solving problems on paper, I built <strong><a href="https://eigen-ai.streamlit.app/" rel="noopener noreferrer">EigenAI</a></strong> — an interactive Streamlit app that teaches mathematical concepts through live computation, step-by-step explanations, and real-time visualizations.</p>

<blockquote>
<p><strong>Can we make eigenvalues, gradients, and hill climbing algorithms as intuitive as playing with Legos?</strong></p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnzx8b9m0691khi325dlo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnzx8b9m0691khi325dlo.png" alt="Lego Wallpaper" width="800" height="533"></a></p>

<p>That question drove the entire project.</p>




<h2>
  
  
  🤖 What Is EigenAI?
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkit7ysi5jp0fjo7uy72g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkit7ysi5jp0fjo7uy72g.png" alt="EigenAI taking a coffee getting ready to teach" width="800" height="573"></a></p>

<p><strong>EigenAI</strong> (playing on "eigenvalues" and "AI foundations") is a web-based educational platform that implements core mathematical concepts from AI foundations. It's structured around <strong>four assessments</strong> that progressively build complexity, with the app implementing the three case study assessments (2A, 2B, 3):</p>

<h3>
  
  
  <strong>The 12-Week Journey</strong>
</h3>

<p>The subject covered 12 progressive modules:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Week</th>
<th>Topic</th>
<th>Overview</th>
</tr>
</thead>
<tbody>
<tr>
<td>Weeks 1-5</td>
<td>Linear Algebra Foundations</td>
<td>Sets, vectors, matrices, transformations, eigenvalues</td>
</tr>
<tr>
<td>Weeks 6-9</td>
<td>Calculus &amp; Optimization</td>
<td>Derivatives, integrals, hill climbing, simulated annealing, genetic algorithms</td>
</tr>
<tr>
<td>Weeks 10-12</td>
<td>Probability, Statistics &amp; Logic</td>
<td>Foundations for AI reasoning and decision-making</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><strong><em>Note: Module 6 taught by <a href="https://www.niushashafiabady.com/" rel="noopener noreferrer">Dr. Niusha Shafiabady</a></em></strong></p>
</blockquote>




<h3>
  
  
  <strong>Assessment 1: Linear Algebra Fundamentals</strong> <em>(Online Quiz)</em>
</h3>

<ul>
<li>✅ Matrix operations (addition, multiplication, transpose)
</li>
<li>✅ Vector operations (magnitude, unit vectors, dot product, cross product)
</li>
<li>✅ Systems of equations (elimination, Gaussian elimination)
</li>
<li>✅ Linear transformations (stretching, reflection, projection)
</li>
</ul>

<p><strong>The Challenge:</strong> 60-minute timed quiz covering Modules 1-2 foundational concepts—no coding, pure mathematical understanding.</p>

<p><strong>Why It Matters:</strong> These fundamentals are the building blocks for understanding how data flows through neural networks and ML algorithms.</p>

<blockquote>
<p><em>Note: Assessment 1 was a quiz-only assessment. The EigenAI app implements the three case study assessments (2A, 2B, 3) that required coding solutions.</em></p>
</blockquote>




<h3>
  
  
  <strong>Assessment 2A: Determinants &amp; Eigenvalues</strong> <em>(Case Study)</em>
</h3>

<ul>
<li>✅ Recursive determinant calculation for n×n matrices
</li>
<li>✅ Eigenvalue and eigenvector computation (2×2 matrices)
</li>
<li>✅ Step-by-step mathematical notation using SymPy
</li>
<li>✅ Input validation and error handling
</li>
</ul>

<p><strong>The Challenge:</strong> Implement cofactor expansion from scratch—no NumPy allowed for core logic, only pure Python.</p>

<p><strong>Why It Matters:</strong> Eigenvalues and eigenvectors are the foundation of:</p>

<ul>
<li>
<strong>PCA (Principal Component Analysis)</strong> — dimensionality reduction for large datasets</li>
<li>
<strong>Eigenfaces</strong> — facial recognition algorithms</li>
<li>
<strong>Feature compression</strong> — reducing computational cost in ML models</li>
</ul>

<p>Understanding determinants reveals why singular matrices break these algorithms.</p>




<h3>
  
  
  <strong>Assessment 2B: Calculus &amp; Neural Networks</strong> <em>(Case Study)</em>
</h3>

<ul>
<li>✅ Numerical integration (Trapezoid, Simpson's Rule, Adaptive Simpson)
</li>
<li>✅ RRBF (Recurrent Radial Basis Function) gradient computation
</li>
<li>✅ Manual backpropagation without TensorFlow/PyTorch
</li>
<li>✅ Comparative analysis of integration methods with error bounds
</li>
</ul>

<p><strong>The Challenge:</strong> Compute gradients by hand for a recurrent network—feel the chain rule in your bones.</p>

<p><strong>Why It Matters:</strong> Before using <code>model.fit()</code>, you should understand what <code>.backward()</code> actually does.</p>




<h3>
  
  
  <strong>Assessment 3: AI Optimization Algorithms</strong> <em>(Case Study)</em>
</h3>

<ul>
<li>✅ Hill Climbing algorithm for binary image reconstruction
</li>
<li>✅ Stochastic sampling variant (speed vs. accuracy trade-off)
</li>
<li>✅ Pattern complexity selector (simple vs. complex cost landscapes)
</li>
<li>✅ Real-time cost progression visualization
</li>
</ul>

<p><strong>The Challenge:</strong> Reconstruct a 10×10 binary image from random noise using only local search—no global optimization, no backtracking.</p>

<p><strong>Why It Matters:</strong> Hill climbing is the foundation of gradient descent, simulated annealing, and evolutionary algorithms. If you understand local optima here, you understand why neural networks get stuck.</p>

<blockquote>
<p><strong>💡 Key Insight from Module 6 (<a href="https://www.niushashafiabady.com/" rel="noopener noreferrer">Dr. Niusha Shafiabady</a>):</strong></p>

<p>Hill climbing can get stuck in local optima with no guarantee of finding the global optimum. The cure?</p>

<ul>
<li>
<strong>Random restarts</strong> (try multiple starting points)</li>
<li>
<strong>Random mutations</strong> (introduce noise)</li>
<li>
<strong>Probabilistic acceptance</strong> (simulated annealing)</li>
</ul>

<p>This limitation explains why modern AI uses ensemble methods and stochastic optimization.</p>
</blockquote>




<h2>
  
  
  🗓️ Project Timeline &amp; Results
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Month</th>
<th>Assessment</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>October 2025</td>
<td>Linear Algebra Quiz</td>
<td><strong>72.5% (C)</strong></td>
</tr>
<tr>
<td>October 2025</td>
<td>Determinants &amp; Eigenvalues</td>
<td><strong>82% (D)</strong></td>
</tr>
<tr>
<td>November 2025</td>
<td>Integrals &amp; RRBF</td>
<td><strong>84% (D)</strong></td>
</tr>
<tr>
<td>December 2025</td>
<td>Hill Climbing</td>
<td>Awaiting results</td>
</tr>
</tbody>
</table></div>

<p><strong>Total Duration:</strong> 12 weeks of intensive mathematical foundations for AI</p>




<h2>
  
  
  🏗️ Technical Architecture
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Technology</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Frontend</strong></td>
<td>Streamlit</td>
<td>Interactive UI with zero JavaScript</td>
</tr>
<tr>
<td><strong>Core Logic</strong></td>
<td>Pure Python 3.10+</td>
<td>Type-hinted, no NumPy in algorithms</td>
</tr>
<tr>
<td><strong>Math Rendering</strong></td>
<td>SymPy + matplotlib</td>
<td>LaTeX-quality equations</td>
</tr>
<tr>
<td><strong>Deployment</strong></td>
<td>Streamlit Cloud</td>
<td>One-click deploy from GitHub</td>
</tr>
<tr>
<td><strong>Version Control</strong></td>
<td>Git + GitHub</td>
<td>Full project history since commit 1</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  <strong>Why Pure Python for Core Logic?</strong>
</h3>

<p>The assessment required implementing algorithms <strong>without numerical libraries</strong> to demonstrate understanding of the underlying math. This constraint forced me to:</p>

<ul>
<li>Write cofactor expansion from scratch (not just <code>np.linalg.det()</code>)</li>
<li>Implement Simpson's Rule manually (not just <code>scipy.integrate.quad()</code>)</li>
<li>Build hill climbing with custom neighbor generation (not just <code>scipy.optimize.minimize()</code>)</li>
</ul>

<p><strong>Result:</strong> Deep understanding of how these algorithms actually work under the hood.</p>




<h2>
  
  
  🗝️ Key Features &amp; Lessons Learned
</h2>

<h3>
  
  
  <strong>1. Modular Architecture That Scales</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>eigenai/
├── app.py                    # Main Streamlit entry point
├── views/                    # UI components (one per assessment)
│   ├── set1Problem1.py      # Determinants UI
│   ├── set1Problem2.py      # Eigenvalues UI
│   ├── set2Problem1.py      # Integration UI
│   ├── set2Problem2.py      # RRBF UI
│   └── set3Problem1.py      # Hill Climbing UI
└── resolvers/                # Pure Python algorithms
    ├── determinant.py
    ├── eigen_solver.py
    ├── integrals.py
    ├── rrbf.py
    ├── hill_climber.py
    └── constructor.py
</code></pre>

</div>



<p><strong>Lesson Learned:</strong> Separating algorithm logic from UI made testing 10x easier. When debugging the cost function, the UI stayed unchanged. When improving visualizations, the core math stayed untouched.</p>

<p><strong>Iterative Development:</strong> EigenAI evolved through 23+ versions:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version</th>
<th>Milestone</th>
</tr>
</thead>
<tbody>
<tr>
<td>v0.0.1</td>
<td>Streamlit setup, assets, pages</td>
</tr>
<tr>
<td>v0.1.0</td>
<td>✅ Assessment 2A submission</td>
</tr>
<tr>
<td>v0.1.8</td>
<td>Added Hill Climbing Binary Image Reconstruction</td>
</tr>
<tr>
<td>v0.2.0</td>
<td>✅ Assessment 2B submission (Integration + RRBF)</td>
</tr>
<tr>
<td>v0.2.4</td>
<td>Added stochastic sampling to Hill Climber</td>
</tr>
<tr>
<td>v0.2.6</td>
<td>Added complex pattern selector</td>
</tr>
<tr>
<td>v0.3.0</td>
<td>✅ Assessment 3 submission (Hill Climbing Algorithm)</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p>Each assessment pushed the app forward—turning coursework into production-ready features. Detailed <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/projects/eigenai/docs/changelog.md" rel="noopener noreferrer"><code>CHANGELOG.md</code></a></p>
</blockquote>




<h3>
  
  
  <strong>2. Hill Climbing: When "Good Enough" Is Good Enough</strong>
</h3>

<p>The most fascinating part was implementing <strong>Hill Climbing</strong> for image reconstruction:</p>

<p><strong>The Problem:</strong></p>

<ul>
<li>Start with a random 10×10 binary image (noise)</li>
<li>Target: A circle pattern (100 pixels to match)</li>
<li>Cost function: Hamming distance (count mismatched pixels)</li>
<li>Neighborhood: Flip one pixel at a time (100 neighbors per state)</li>
</ul>

<p><strong>The Algorithm:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">while</span> <span class="n">cost</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="ow">and</span> <span class="n">iterations</span> <span class="o">&lt;</span> <span class="n">max_iterations</span><span class="p">:</span>
    <span class="n">neighbors</span> <span class="o">=</span> <span class="nf">generate_all_100_neighbors</span><span class="p">(</span><span class="n">current_state</span><span class="p">)</span>
    <span class="n">best_neighbor</span> <span class="o">=</span> <span class="nf">min</span><span class="p">(</span><span class="n">neighbors</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="n">cost_function</span><span class="p">)</span>

    <span class="k">if</span> <span class="nf">cost</span><span class="p">(</span><span class="n">best_neighbor</span><span class="p">)</span> <span class="o">&gt;=</span> <span class="nf">cost</span><span class="p">(</span><span class="n">current_state</span><span class="p">):</span>
        <span class="k">break</span>  <span class="c1"># Stuck at local optimum
</span>
    <span class="n">current_state</span> <span class="o">=</span> <span class="n">best_neighbor</span>
    <span class="n">iterations</span> <span class="o">+=</span> <span class="mi">1</span>
</code></pre>

</div>



<p><strong>Results:</strong></p>

<ul>
<li>Simple pattern (circle): <strong>100% success rate</strong>, avg 147 iterations</li>
<li>Complex pattern (checkerboard): <strong>85% success rate</strong>, gets stuck in local optima</li>
<li>Stochastic sampling (50 neighbors): <strong>95% success</strong>, 2x faster</li>
</ul>

<p><strong>The Insight:</strong> Hill climbing works beautifully on smooth cost landscapes but fails on complex ones.</p>

<p><strong>This limitation explains why modern AI uses:</strong></p>

<ul>
<li>
<strong>Simulated annealing</strong> — allows temporary cost increases (probabilistic acceptance)</li>
<li>
<strong>Genetic algorithms</strong> — explores multiple paths simultaneously (population-based)</li>
<li>
<strong>Gradient descent with momentum</strong> — escapes shallow local minima (velocity-based)</li>
</ul>




<h3>
  
  
  <strong>3. Stochastic Sampling: The Speed vs. Accuracy Trade-Off</strong>
</h3>

<p>One enhancement I added beyond requirements was <strong>stochastic hill climbing</strong>:</p>

<p>Instead of evaluating all 100 neighbors, randomly sample 50.</p>

<p><strong>Trade-offs:</strong></p>

<ul>
<li>⚡ <strong>Speed:</strong> 2x faster per iteration</li>
<li>⚠️ <strong>Accuracy:</strong> May miss optimal move 5% of the time</li>
<li>📊 <strong>Final cost:</strong> Avg 0.5 pixels worse than full evaluation</li>
</ul>

<p><strong>Real-world application:</strong> When you have 10,000 neighbors (e.g., 100×100 image), evaluating all is impractical. Stochastic sampling becomes mandatory.</p>




<h2>
  
  
  KPIs
</h2>

<p>For the hill climbing implementation, I tracked:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Simple Pattern</th>
<th>Complex Pattern</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Initial Cost</strong></td>
<td>~50 mismatched pixels</td>
<td>~50 mismatched pixels</td>
</tr>
<tr>
<td><strong>Final Cost</strong></td>
<td>0 (perfect)</td>
<td>0-8 (may get stuck)</td>
</tr>
<tr>
<td><strong>Iterations</strong></td>
<td>~147</td>
<td>~500 (hits plateau limit)</td>
</tr>
<tr>
<td><strong>Time</strong></td>
<td>&lt;0.03s</td>
<td>&lt;0.2s</td>
</tr>
<tr>
<td><strong>Neighbors Evaluated</strong></td>
<td>~14,700</td>
<td>~50,000</td>
</tr>
<tr>
<td><strong>Success Rate</strong></td>
<td>100%</td>
<td>85%</td>
</tr>
</tbody>
</table></div>

<p><strong>Key Takeaway:</strong> Problem structure matters more than algorithm sophistication. A simple greedy search beats complex methods on convex problems.</p>




<h2>
  
  
  💥 Insights
</h2>

<p>This project transformed my understanding of AI math:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Before</th>
<th>After</th>
</tr>
</thead>
<tbody>
<tr>
<td>"Eigenvalues are λ where det(A - λI) = 0" (memorized formula)</td>
<td>Built cofactor expansion recursively, <strong>saw</strong> how determinants break down</td>
</tr>
<tr>
<td>"Gradient descent minimizes loss" (vague intuition)</td>
<td>Computed RRBF gradients by hand, <strong>felt</strong> the chain rule propagate</td>
</tr>
<tr>
<td>"Hill climbing gets stuck in local optima" (heard in lectures)</td>
<td>Watched hill climbing fail on checkerboards, <strong>understood</strong> why cost landscape matters</td>
</tr>
</tbody>
</table></div>

<p>This transformation from abstract concepts to concrete understanding has fundamentally changed how I approach AI problems: I now see the math not as a collection of formulas, but as a toolkit of interconnected ideas that I can manipulate and reason about directly.</p>

<p>The hands-on experience has given me a deep, intuitive grasp of the mathematical foundations that underpin modern AI, enabling me to approach complex problems with both confidence and clarity, and to think about optimization and machine learning as <strong>algorithms to apply</strong> and <strong>mathematical principles</strong> that I can understand and leverage in practice.</p>




<h2>
  
  
  ❓ What's Next for EigenAI?
</h2>

<p><strong>Module 6 introduced three optimization paradigms:</strong></p>

<ul>
<li>✅ <strong>Hill Climbing</strong> (implemented in Assessment 3)</li>
<li>🕐 <strong>Simulated Annealing</strong> (probabilistic escape from local optima)</li>
<li>🕐 <strong>Genetic Algorithms</strong> (population-based evolutionary search)</li>
</ul>

<p><strong>Upcoming v0.4.X+ features:</strong></p>

<p><strong>Enhanced Optimization Suite:</strong></p>

<ul>
<li>Simulated Annealing comparison (temperature schedules, acceptance probability)</li>
<li>Genetic Algorithm variant (crossover, mutation, selection operators)</li>
<li>A* Search for pathfinding (admissible heuristics)</li>
<li>Q-Learning demo (reinforcement learning basics)</li>
</ul>

<p><strong>Platform Enhancements:</strong></p>

<ul>
<li>
<strong>Authentication</strong> — user login and progress tracking</li>
<li>
<strong>LLM Integration</strong> — GPT-4 powered step-by-step explanations with rate limiting</li>
<li>
<strong>Custom Agent Framework</strong> — Built from the ground-up using knowledge graphs and reasoning for problem-solving</li>
<li>
<strong>Supabase BaaS</strong> — cloud storage for user data and solutions</li>
<li>
<strong>Backend Framework</strong> — FastAPI or Flask for RESTful API</li>
<li>
<strong>Weekly Digest</strong> — agentic integration for learning analytics</li>
<li>
<strong>Test Coverage</strong> — comprehensive unit testing with pytest</li>
<li>
<strong>Security Enhancements</strong> — input sanitization, HTTPS enforcement</li>
</ul>




<h2>
  
  
  Try It Out
</h2>

<p>If you want to explore EigenAI:</p>

<ul>
<li>
<strong>🌍 Live Demo:</strong> <a href="https://eigen-ai.streamlit.app/" rel="noopener noreferrer">eigen-ai.streamlit.app</a>
</li>
<li>📋 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set1Problem1/MFA501_Assessment2_Set1Problem1_report_Faria_Luis.pdf" rel="noopener noreferrer">Assessment 2A, S1P1, Determinants, Reflective Report</a>
</li>
<li>📹 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set1Problem1/MFA501_Assessment2_Set1Problem1_video_Faria_Luis.mp4" rel="noopener noreferrer">Assessment 2A, S1P1, Determinants, Video Demo</a>
</li>
<li>📋 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set1Problem2/MFA501_Assessment2_Set1Problem2_report_Faria_Luis.pdf" rel="noopener noreferrer">Assessment 2A, S1P2, Eigenvalues, Reflective Report</a>
</li>
<li>📹 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set1Problem2/MFA501_Assessment2_Set1Problem2_video_Faria_Luis.mp4" rel="noopener noreferrer">Assessment 2A, S1P2, Eigenvalues, Video Demo</a>
</li>
<li>📋 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set2Problem1/MFA501_Assessment2B_Set1_report_Faria_Luis.pdf" rel="noopener noreferrer">Assessment 2B, S2P1, Integrals, Reflective Report</a>
</li>
<li>📹 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set2Problem1/MFA501_Assessment2B_Set1_demo_Faria_Luis.mp4" rel="noopener noreferrer">Assessment 2B, S2P1, Integrals, Video Demo</a>
</li>
<li>📋 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set2Problem2/MFA501_Assessment2B_Set2_report_Faria_Luis.pdf" rel="noopener noreferrer">Assessment 2B, S2P2, RRBF, Reflective Report</a>
</li>
<li>📹 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment2/Set2Problem2/MFA501_Assessment2B_Set2_demo_Faria_Luis.mp4" rel="noopener noreferrer">Assessment 2B, S2P2, RRBF, Video Demo</a>
</li>
<li>📋 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment3/Set3Problem1/MFA501_Assessment3_report_Faria_Luis.pdf" rel="noopener noreferrer">Assessment 3, Hill Climbing, Reflective Report</a>
</li>
<li>📹 <a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-MFA/assignments/Assessment3/Set3Problem1/MFA501_Assessment3_demo_Faria_Luis.mp4" rel="noopener noreferrer">Assessment 3, Hill Climbing, Video Demo</a>
</li>
<li>🤖 <a href="https://github.com/lfariabr/masters-swe-ai/tree/master/2025-T2/T2-MFA/projects/eigenai" rel="noopener noreferrer">EigenAi Source Code</a>
</li>
</ul>




<h2>
  
  
  Let's Connect!
</h2>

<p>Building EigenAI has been the perfect bridge between mathematical theory and practical software engineering. If you're:</p>

<ul>
<li>Learning AI/ML foundations</li>
<li>Building educational tools</li>
<li>Passionate about making math accessible</li>
<li>Interested in optimization algorithms</li>
</ul>

<p>I’d love to connect:</p>

<ul>
<li>
<strong>LinkedIn:</strong> <a href="https://www.linkedin.com/in/lfariabr/" rel="noopener noreferrer">linkedin.com/in/lfariabr</a>
</li>
<li>
<strong>GitHub:</strong> <a href="https://github.com/lfariabr" rel="noopener noreferrer">github.com/lfariabr</a>
</li>
<li>
<strong>Portfolio:</strong> <a href="https://luisfaria.dev" rel="noopener noreferrer">luisfaria.dev</a>
</li>
</ul>




<h2>
  
  
  References &amp; Further Reading
</h2>

<p><strong>Academic Sources:</strong></p>

<ul>
<li>Strang, G. (2016). <em>Introduction to linear algebra</em> (5th ed.). Wellesley-Cambridge Press.</li>
<li>Goodfellow, I., Bengio, Y., &amp; Courville, A. (2016). <em>Deep learning</em>. MIT Press.</li>
<li>Nocedal, J., &amp; Wright, S. (2006). <em>Numerical optimization</em>. Springer.</li>
</ul>

<p><strong>Project Tech:</strong></p>

<ul>
<li><a href="https://docs.streamlit.io/" rel="noopener noreferrer">Streamlit Documentation</a></li>
<li><a href="https://docs.sympy.org/" rel="noopener noreferrer">SymPy Symbolic Math</a></li>
<li><a href="https://peps.python.org/pep-0484/" rel="noopener noreferrer">Python Type Hints (PEP 484)</a></li>
</ul>




<p><strong>Tags:</strong> #machinelearning #python #streamlit #ai #mathematics #optimization #hillclimbing #education</p>




<p><em>Built with ☕ and calculus by <a href="https://luisfaria.dev" rel="noopener noreferrer">Luis Faria</a><br><br>
Student @ Torrens University Australia | MFA501 | Dec 2025</em></p>

