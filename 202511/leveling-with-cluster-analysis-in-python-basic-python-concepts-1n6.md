---
Title: Leveling with cluster analysis in Python: basic Python concepts
Description: 
Author: Hilton Fernandes
Date: 2025-11-06T21:36:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>This is the 2nd of a series of 5 little articles that intend to present a simple idea of time series, and their implementation in Python. The purpose here is to present both a time series problem, and how we can solve it in simple Python code.</p>

<p>Only very basic knowledge of Python and time series are needed as most concepts will be explained with care.</p>

<h2>
  
  
  Roadmap
</h2>

<p>The 1st article of the series, <a href="https://dev.to/hilton_fernandes_eaac26ab/leveling-with-cluster-analysis-in-python-400p">Leveling with cluster analysis in Python: general concepts</a> presented the basic concepts of this series. This one, the 2nd one, will present basic Python concepts and techniques to be used in the solution. The 3rd one will present a solution implemented in Python. The 4th article will add a sinusoidal decomposition of the data after the filtering of the solution. And the 5th and last one will use all the elements to address a real problem in cryptocurrencies.</p>

<h2>
  
  
  Some simple Python ideas
</h2>

<h3>
  
  
  Libraries, modules and submodules
</h3>

<p>Isaac Newton, that created a huge part of modern Physics and Mathematics once said the <a href="https://en.wikiquote.org/wiki/Isaac_Newton#Quotes" rel="noopener noreferrer">he could see further because because he standed in the shoulder of giants</a>.</p>

<p>This concept is behind most software codes: they do not create everything, they use a large part of what was already created, mainly in the form of <a href="https://en.wikipedia.org/wiki/Library_(computing)" rel="noopener noreferrer">sofware libraries</a>. Python is very good at this, and here is the part of the code used here that will use some libraries. In Python, a library is usually named a <em>module</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span> 

<span class="kn">import</span> <span class="n">matplotlib.pyplot</span> <span class="k">as</span> <span class="n">plt</span>
<span class="kn">from</span> <span class="n">sklearn.cluster</span> <span class="kn">import</span> <span class="n">KMeans</span>
</code></pre>

</div>



<p>The idea that the current code will bring information from another one is encompassed in the word <code>import</code> in the code above. Another important point is that programmers usually prefer to write less. So, in the 1st line, the library <code>numpy</code> is renamed as <code>np</code>. In another line, the module <code>matplotlib.pyplot</code> is renamed simply as <code>plt</code>.</p>

<p>And in the last line of the code, another shortening is presented: instead of renaming a code fragment in a shorter from, Python lets one pick only what a programmer needs. In this case, only <code>KMeans</code> will be picked from <code>sklearn.cluster</code>.</p>

<p>A less important point is that use of a dot lets selecting a part of a module; that is: a <em>submodule</em>. In the code shown, <code>matplotlib.pyplot</code> means the submodule <code>pyplot</code> of the module <code>matplotlib</code>. And, of course, <code>sklearn.cluster</code> is the submodule <code>cluster</code> of the module <code>sklearn</code>. Creating submodules increases the organization of larger modules, as this divides them in specialized parts.</p>

<h3>
  
  
  Pieces of information in scalar variables
</h3>

<p>The next part of the code deals with storing information that shall be processed. Here a list of <em>scalar variables</em>, or variables that are individual:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">coeff</span> <span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">0.25</span>

<span class="n">base</span> <span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">0.0</span>
<span class="n">ladder</span> <span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">1.0</span>
<span class="n">n_points</span> <span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">101</span>
<span class="n">n_half</span> <span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="n">n_points</span> <span class="o">//</span> <span class="mi">2</span>

<span class="n">delta</span> <span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="mf">1.0</span> <span class="o">/</span> <span class="n">n_points</span>
</code></pre>

</div>



<p>There are two types of information in this code fragment: <code>int</code>, that hold only integral values. In this case the count of points (<code>n_point</code>) and the half of that count, <code>n_half</code>.</p>

<p>The double bar in the line for <code>n_half</code> is used to make sure the division of <code>n_points</code> by <code>2</code> will generate a number of type <code>int</code>, and not a <code>float</code> number. That is: it will hold <code>50</code>, and not <code>50.5</code>, as the division of <code>101</code> by <code>2</code> would create.</p>

<p>The type <code>float</code> can be used to hold number with a decimal part. For instance, <code>coeff</code> will be used to hold a coefficient. In this case, <code>0.25</code>.  Since the problem here is to represent a discontuinuity, a set of values will be close to a <code>base</code> level, while other ones will be above it, in <code>ladder</code>.</p>

<p>And finally, <code>delta</code> holds the step that will be used as a clock tick in our time series.</p>

<h3>
  
  
  A data generator
</h3>

<p>A time series usually is a series of data collected along the time. For instance, mean wage in a certain year. But to avoid the need of getting real data, this code will generate its own data. By means of a <a href="https://en.wikipedia.org/wiki/Pseudorandom_number_generator" rel="noopener noreferrer">pseudorandom number generator</a>, aka PRNG. In a few words, a PRNG is a mathematical algorithm that can generate a sequence of numbers without any pattern; that is: they look random. It's called <em>pseudorandom</em> because if one such algorithm is fed with a constant <em>seed</em> it will always generate the same sequence of numbers.</p>

<p>In this case it is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">rng</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="n">random</span><span class="p">.</span><span class="nf">default_rng</span><span class="p">(</span><span class="mi">42</span><span class="p">)</span>
</code></pre>

</div>



<p>In this code, <code>rng</code> is the name of the fabric of numbers, and it's created by calling the <em>function</em> (or piece of code) <code>defaut_rng</code> with the parameter <code>42</code>.  This function is in the submodule <code>np.random</code>.</p>

<h3>
  
  
  Pieces of information in arrays
</h3>

<p>Since a time series contain several data, a <em>scalar variable</em> can't be used to contain it. The module <code>numpy</code> has the resource of <em>arrays</em> or <code>ndarrays</code>. In an array, the elements are identified by an <em>index</em>, that is analogous to the apartment number in a building.</p>

<p>The following code fragment creates the arrays needed to this problem:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">x</span> <span class="p">:</span> <span class="n">np</span><span class="p">.</span><span class="n">ndarray</span> <span class="o">=</span> <span class="n">np</span><span class="p">.</span><span class="nf">linspace</span><span class="p">(</span><span class="mf">0.0</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">,</span> <span class="n">n_points</span><span class="p">)</span>
<span class="n">y</span> <span class="p">:</span> <span class="n">np</span><span class="p">.</span><span class="n">ndarray</span> <span class="o">=</span> <span class="n">rng</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="o">-</span><span class="mf">0.5</span><span class="p">,</span> <span class="mf">0.5</span><span class="p">,</span> <span class="n">size</span><span class="o">=</span><span class="n">n_points</span><span class="p">)</span>

<span class="n">y</span><span class="p">[</span><span class="n">n_half</span> <span class="p">:</span> <span class="p">]</span> <span class="o">+=</span> <span class="n">ladder</span>
</code></pre>

</div>



<p>In the 1st line, the array <code>x</code> receives <code>n_points</code> (aka <code>101</code>) numbers from <code>0.0</code> to <code>1.0</code>, subdivided in increments of <code>0.01</code>; that is: <code>0.0</code>, <code>0.01</code>, <code>0.02</code>, ..., <code>0.99</code>, <code>1.0</code></p>

<p>In the 2nd line, the array <code>y</code> receives <code>n_points</code> numbers chosen <em>pseudorandomly</em> between <code>-05</code> and <code>0.5</code>.</p>

<p>In the third line, the 2nd half of <code>y</code> receives an increment of <code>ladder</code>. That creates  the discontinuity to be solved in the remaining of the texts.</p>

<h2>
  
  
  Plotting the data
</h2>

<p>Please consider the following code fragment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">fig</span><span class="p">,</span> <span class="n">ax</span> <span class="o">=</span> <span class="n">plt</span><span class="p">.</span><span class="nf">subplots</span><span class="p">()</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">plot</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>

<span class="n">ax</span><span class="p">.</span><span class="nf">set_xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">x</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax</span><span class="p">.</span><span class="nf">set_ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">y</span><span class="sh">'</span><span class="p">)</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">grid</span><span class="p">(</span><span class="n">visible</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<p>The 1st line is a generic one, that can be used to create <em>plots</em> -- aka charts -- much more complicated than the ones used here. For instance, several plots in the same image.</p>

<p>The 2nd line plots the elements of two arrays, taking care of connecting by lines each one of their values.</p>

<p>The 3rd and 4th lines of the code label the <code>x</code> and <code>y</code> axis.</p>

<p>The 5th line of the code create a grid to ease the visualization of data. And the 6th and final line causes the chart to be shown in the screen.</p>

<p>The final result is like this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa1hv5vkfi32omlyezfty.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa1hv5vkfi32omlyezfty.jpeg" alt="A time series with a discontinuity" width="640" height="480"></a></p>

