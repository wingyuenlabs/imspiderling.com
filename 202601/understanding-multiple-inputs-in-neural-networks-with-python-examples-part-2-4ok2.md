---
Title: Understanding Multiple Inputs in Neural Networks (With Python Examples) — Part 2
Description: 
Author: Rijul Rajesh
Date: 2026-01-21T21:06:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>In the <a href="https://dev.to/rijultp/understanding-multiple-inputs-in-neural-networks-with-python-examples-1ng9">previous article</a>, we plotted a surface using inputs related to <strong>Petal Width</strong> and <strong>Sepal Width</strong> to predict whether the species is <strong>Setosa</strong> or not.</p>

<p>Let’s continue with the remaining steps.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsjvpp7j7hyzafdzklfrp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsjvpp7j7hyzafdzklfrp.png" alt=" " width="453" height="443"></a></p>

<p>After plotting the bent surface, we need to multiply all the points by <strong>-0.1</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">import</span> <span class="n">matplotlib.pyplot</span> <span class="k">as</span> <span class="n">plt</span>

<span class="c1"># Create a grid of dots across the entire bottom plane
</span><span class="n">p_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>
<span class="n">s_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>
<span class="n">P_grid</span><span class="p">,</span> <span class="n">S_grid</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">meshgrid</span><span class="p">(</span><span class="n">p_dots</span><span class="p">,</span> <span class="n">s_dots</span><span class="p">)</span>

<span class="c1"># Calculate surface values
</span><span class="n">Y_grid_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">maximum</span><span class="p">(</span>
    <span class="mi">0</span><span class="p">,</span>
    <span class="p">(</span><span class="n">P_grid</span> <span class="o">*</span> <span class="o">-</span><span class="mf">2.5</span><span class="p">)</span> <span class="o">+</span> <span class="p">(</span><span class="n">S_grid</span> <span class="o">*</span> <span class="o">-</span><span class="mf">0.6</span><span class="p">)</span> <span class="o">+</span> <span class="mf">1.6</span>
<span class="p">)</span>

<span class="c1"># Scale the bent surface
</span><span class="n">Y_grid_dots</span> <span class="o">=</span> <span class="n">Y_grid_dots</span> <span class="o">*</span> <span class="o">-</span><span class="mf">0.1</span>

<span class="n">fig</span> <span class="o">=</span> <span class="n">plt</span><span class="p">.</span><span class="nf">figure</span><span class="p">()</span>
<span class="n">ax</span> <span class="o">=</span> <span class="n">fig</span><span class="p">.</span><span class="nf">add_subplot</span><span class="p">(</span><span class="mi">111</span><span class="p">,</span> <span class="n">projection</span><span class="o">=</span><span class="sh">'</span><span class="s">3d</span><span class="sh">'</span><span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">plot_surface</span><span class="p">(</span>
    <span class="n">P_grid</span><span class="p">,</span>
    <span class="n">S_grid</span><span class="p">,</span>
    <span class="n">Y_grid_dots</span><span class="p">,</span>
    <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span>
<span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">set_xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Petal Width</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Sepal Width</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_zlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Setosa Output</span><span class="sh">'</span><span class="p">)</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbpvs8rkfqztuir8tf7lk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbpvs8rkfqztuir8tf7lk.png" alt=" " width="469" height="455"></a></p>

<p>Now we do the exact same thing for the bottom node in the hidden layer.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faku9h5lwk1q5zn16awx8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faku9h5lwk1q5zn16awx8.png" alt=" " width="685" height="75"></a></p>

<p>We apply ReLU to this and then multiply the result by <strong>1.5</strong> to get the final orange bent surface.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">import</span> <span class="n">matplotlib.pyplot</span> <span class="k">as</span> <span class="n">plt</span>

<span class="c1"># Create a grid of dots across the entire bottom plane
</span><span class="n">p_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>
<span class="n">s_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>
<span class="n">P_grid</span><span class="p">,</span> <span class="n">S_grid</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">meshgrid</span><span class="p">(</span><span class="n">p_dots</span><span class="p">,</span> <span class="n">s_dots</span><span class="p">)</span>

<span class="c1"># Bottom hidden node calculation
</span><span class="n">Y_grid_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">maximum</span><span class="p">(</span>
    <span class="mi">0</span><span class="p">,</span>
    <span class="p">(</span><span class="n">P_grid</span> <span class="o">*</span> <span class="o">-</span><span class="mf">1.5</span><span class="p">)</span> <span class="o">+</span> <span class="p">(</span><span class="n">S_grid</span> <span class="o">*</span> <span class="mf">0.4</span><span class="p">)</span> <span class="o">+</span> <span class="mf">0.7</span>
<span class="p">)</span>

<span class="c1"># Apply output weight
</span><span class="n">Y_grid_dots</span> <span class="o">=</span> <span class="n">Y_grid_dots</span> <span class="o">*</span> <span class="mf">1.5</span>

<span class="n">fig</span> <span class="o">=</span> <span class="n">plt</span><span class="p">.</span><span class="nf">figure</span><span class="p">()</span>
<span class="n">ax</span> <span class="o">=</span> <span class="n">fig</span><span class="p">.</span><span class="nf">add_subplot</span><span class="p">(</span><span class="mi">111</span><span class="p">,</span> <span class="n">projection</span><span class="o">=</span><span class="sh">'</span><span class="s">3d</span><span class="sh">'</span><span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">plot_surface</span><span class="p">(</span>
    <span class="n">P_grid</span><span class="p">,</span>
    <span class="n">S_grid</span><span class="p">,</span>
    <span class="n">Y_grid_dots</span><span class="p">,</span>
    <span class="n">color</span><span class="o">=</span><span class="sh">'</span><span class="s">orange</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span>
<span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">set_xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Petal Width</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Sepal Width</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_zlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Setosa Output</span><span class="sh">'</span><span class="p">)</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffz1mu6yyrxnr308xxchd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffz1mu6yyrxnr308xxchd.png" alt=" " width="447" height="439"></a></p>

<p>Now we add the <strong>y-axis coordinates</strong> of the blue bent surface to the <strong>y-axis coordinates</strong> of the orange bent surface.</p>

<p>We do this for every single point, and we end up with this green surface.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">import</span> <span class="n">matplotlib.pyplot</span> <span class="k">as</span> <span class="n">plt</span>

<span class="c1"># Create a grid of dots across the entire bottom plane
</span><span class="n">p_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>
<span class="n">s_dots</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>
<span class="n">P_grid</span><span class="p">,</span> <span class="n">S_grid</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">meshgrid</span><span class="p">(</span><span class="n">p_dots</span><span class="p">,</span> <span class="n">s_dots</span><span class="p">)</span>

<span class="c1"># ----- Blue surface (top hidden node) -----
</span><span class="n">Y_blue</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">maximum</span><span class="p">(</span>
    <span class="mi">0</span><span class="p">,</span>
    <span class="p">(</span><span class="n">P_grid</span> <span class="o">*</span> <span class="o">-</span><span class="mf">2.5</span><span class="p">)</span> <span class="o">+</span> <span class="p">(</span><span class="n">S_grid</span> <span class="o">*</span> <span class="o">-</span><span class="mf">0.6</span><span class="p">)</span> <span class="o">+</span> <span class="mf">1.6</span>
<span class="p">)</span>
<span class="n">Y_blue</span> <span class="o">=</span> <span class="n">Y_blue</span> <span class="o">*</span> <span class="o">-</span><span class="mf">0.1</span>

<span class="c1"># ----- Orange surface (bottom hidden node) -----
</span><span class="n">Y_orange</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">maximum</span><span class="p">(</span>
    <span class="mi">0</span><span class="p">,</span>
    <span class="p">(</span><span class="n">P_grid</span> <span class="o">*</span> <span class="o">-</span><span class="mf">1.5</span><span class="p">)</span> <span class="o">+</span> <span class="p">(</span><span class="n">S_grid</span> <span class="o">*</span> <span class="mf">0.4</span><span class="p">)</span> <span class="o">+</span> <span class="mf">0.7</span>
<span class="p">)</span>
<span class="n">Y_orange</span> <span class="o">=</span> <span class="n">Y_orange</span> <span class="o">*</span> <span class="mf">1.5</span>

<span class="c1"># ----- Final output surface (sum) -----
</span><span class="n">Y_final</span> <span class="o">=</span> <span class="n">Y_blue</span> <span class="o">+</span> <span class="n">Y_orange</span>

<span class="n">fig</span> <span class="o">=</span> <span class="n">plt</span><span class="p">.</span><span class="nf">figure</span><span class="p">()</span>
<span class="n">ax</span> <span class="o">=</span> <span class="n">fig</span><span class="p">.</span><span class="nf">add_subplot</span><span class="p">(</span><span class="mi">111</span><span class="p">,</span> <span class="n">projection</span><span class="o">=</span><span class="sh">'</span><span class="s">3d</span><span class="sh">'</span><span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">plot_surface</span><span class="p">(</span>
    <span class="n">P_grid</span><span class="p">,</span>
    <span class="n">S_grid</span><span class="p">,</span>
    <span class="n">Y_final</span><span class="p">,</span>
    <span class="n">color</span><span class="o">=</span><span class="sh">'</span><span class="s">green</span><span class="sh">'</span><span class="p">,</span>
    <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span>
<span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">set_xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Petal Width</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Sepal Width</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_zlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Setosa Output</span><span class="sh">'</span><span class="p">)</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhwvk9bmxdbmvu2jnoanr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhwvk9bmxdbmvu2jnoanr.png" alt=" " width="454" height="441"></a></p>

<p>At the end, we need to add the bias, which is <strong>0</strong>.<br>
This does not change the green surface, so this is the final output for <strong>Setosa</strong>.</p>

<p>From this, we can understand that the value for <strong>Setosa</strong> is highest when the <strong>petal width</strong> is close to <strong>0</strong>.</p>

<p>Similarly, the value for <strong>Setosa</strong> is lowest when the petal width is close to <strong>1</strong>.</p>

<p>In the next article, we will explore how to use this neural network to predict <strong>Setosa</strong>, <strong>Versicolor</strong>, and <strong>Virginica</strong>.</p>

<p>You can try the examples out in <a href="https://colab.research.google.com/drive/1M8Zw5XnqvM5aHVrHYoLJEAwgmT_PbR7v?usp=sharing" rel="noopener noreferrer">the Colab notebook</a>.</p>

<p><strong>Looking for an easier way to install tools, libraries, or entire repositories?</strong><br>
Try <strong>Installerpedia</strong>: a <strong>community-driven, structured installation platform</strong> that lets you install almost anything with <strong>minimal hassle</strong> and <strong>clear, reliable guidance</strong>.</p>

<p>Just run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ipm <span class="nb">install </span>repo-name
</code></pre>

</div>



<p>… and you’re done! 🚀</p>

<p><a href="https://hexmos.com/freedevtools/installerpedia" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm2s3mzj8pfcq94a1y4at.png" alt="Installerpedia Screenshot" width="800" height="336"></a></p>

<p>🔗 <a href="https://hexmos.com/freedevtools/installerpedia/" rel="noopener noreferrer"><strong>Explore Installerpedia here</strong></a></p>

