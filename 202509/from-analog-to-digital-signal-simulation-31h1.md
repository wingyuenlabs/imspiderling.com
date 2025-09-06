---
Title: From Analog to Digital: Signal Simulation
Description: 
Author: Md Sajib Pramanic
Date: 2025-09-06T22:08:35.000Z
Robots: noindex,nofollow
Template: index
---
<p>Have you ever wondered how your music, voice calls, or even video streams travel from the physical world into the digital devices we use every day? The answer lies in <strong>signal processing</strong> the art of converting an analog signal into a digital one.</p>

<p>In this post, I’ll walk you through a simple MATLAB simulation where I generate an analog sine wave, sample it, quantize it, and finally encode it into binary form. By the end, you’ll see how an analog signal transforms step by step into a digital stream.</p>




<h2>
  
  
  1. Generating an Analog Signal
</h2>

<p>Let’s start with a simple sine wave of 100 Hz. Since MATLAB doesn’t truly handle “continuous” signals, I approximate it with a very fine time step:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight matlab"><code><span class="n">t</span> <span class="o">=</span> <span class="mi">0</span><span class="p">:</span><span class="mf">0.0001</span><span class="p">:</span><span class="mf">0.01</span><span class="p">;</span>       <span class="c1">% very fine step (continuous-like time)</span>
<span class="n">f</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>                 <span class="c1">% frequency = 100 Hz</span>
<span class="n">x_analog</span> <span class="o">=</span> <span class="nb">sin</span><span class="p">(</span><span class="mi">2</span><span class="o">*</span><span class="nb">pi</span><span class="o">*</span><span class="n">f</span><span class="o">*</span><span class="n">t</span><span class="p">);</span>

<span class="nb">plot</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="n">x_analog</span><span class="p">,</span> <span class="s1">'LineWidth'</span><span class="p">,</span> <span class="mf">1.5</span><span class="p">);</span>
<span class="nb">title</span><span class="p">(</span><span class="s1">'Analog Signal (Sine Wave)'</span><span class="p">);</span>
</code></pre>

</div>



<p>This gives me the smooth curve — my <strong>reference signal</strong>.</p>




<h2>
  
  
  2. Sampling
</h2>

<p>According to the <strong>Nyquist theorem</strong>, I need to sample at least twice the frequency of the signal (≥ 200 Hz).</p>

<p>I tested three cases:</p>

<ul>
<li>
<strong>150 Hz (below Nyquist)</strong> → aliasing and distortion</li>
<li>
<strong>200 Hz (at Nyquist)</strong> → minimal capture, jagged</li>
<li>
<strong>500 Hz (above Nyquist)</strong> → clean sampling, closer to analog
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight matlab"><code><span class="n">Fs</span> <span class="o">=</span> <span class="mi">500</span><span class="p">;</span>   <span class="c1">% try 150, 200, and 500</span>
<span class="n">Ts</span> <span class="o">=</span> <span class="mi">1</span><span class="p">/</span><span class="n">Fs</span><span class="p">;</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">0</span><span class="p">:</span><span class="n">Ts</span><span class="p">:</span><span class="mf">0.01</span><span class="p">;</span>
<span class="n">x_sampled</span> <span class="o">=</span> <span class="nb">sin</span><span class="p">(</span><span class="mi">2</span><span class="o">*</span><span class="nb">pi</span><span class="o">*</span><span class="n">f</span><span class="o">*</span><span class="n">n</span><span class="p">);</span>

<span class="nb">stem</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">x_sampled</span><span class="p">,</span> <span class="s1">'filled'</span><span class="p">);</span>
<span class="nb">title</span><span class="p">([</span><span class="s1">'Sampled Signal at Fs = '</span> <span class="nb">num2str</span><span class="p">(</span><span class="n">Fs</span><span class="p">)</span> <span class="s1">' Hz'</span><span class="p">]);</span>
</code></pre>

</div>






<h2>
  
  
  3. Quantization
</h2>

<p>Next, I map the sampled values to discrete amplitude levels. Using more bits increases the number of levels:</p>

<ul>
<li>
<strong>8 bits</strong> → 256 levels</li>
<li>
<strong>16 bits</strong> → 65,536 levels</li>
<li>
<strong>64 bits</strong> → practically continuous
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight matlab"><code><span class="n">bits</span> <span class="o">=</span> <span class="mi">8</span><span class="p">;</span>   <span class="c1">% try 8, 16, and 64</span>
<span class="n">levels</span> <span class="o">=</span> <span class="mi">2</span><span class="o">^</span><span class="n">bits</span><span class="p">;</span>
<span class="n">x_min</span> <span class="o">=</span> <span class="nb">min</span><span class="p">(</span><span class="n">x_sampled</span><span class="p">);</span>
<span class="n">x_max</span> <span class="o">=</span> <span class="nb">max</span><span class="p">(</span><span class="n">x_sampled</span><span class="p">);</span>
<span class="n">q_step</span> <span class="o">=</span> <span class="p">(</span><span class="n">x_max</span> <span class="o">-</span> <span class="n">x_min</span><span class="p">)/</span><span class="n">levels</span><span class="p">;</span>

<span class="n">x_index</span> <span class="o">=</span> <span class="nb">round</span><span class="p">((</span><span class="n">x_sampled</span> <span class="o">-</span> <span class="n">x_min</span><span class="p">)/</span><span class="n">q_step</span><span class="p">);</span>
<span class="n">x_quantized</span> <span class="o">=</span> <span class="n">x_index</span><span class="o">*</span><span class="n">q_step</span> <span class="o">+</span> <span class="n">x_min</span><span class="p">;</span>

<span class="nb">stem</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">x_quantized</span><span class="p">,</span> <span class="s1">'filled'</span><span class="p">);</span>
<span class="nb">title</span><span class="p">([</span><span class="s1">'Quantized Signal ('</span> <span class="nb">num2str</span><span class="p">(</span><span class="n">bits</span><span class="p">)</span> <span class="s1">'-bit)'</span><span class="p">]);</span>
</code></pre>

</div>



<p>With fewer bits, you can clearly see stair-step distortions. As I increase the bits, the quantized signal becomes almost identical to the sampled one.</p>




<h2>
  
  
  4. Encoding
</h2>

<p>Finally, each quantized value is represented as a binary word. For example, with 8 bits, each sample becomes an 8-bit binary code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight matlab"><code><span class="n">binary_codes</span> <span class="o">=</span> <span class="nb">dec2bin</span><span class="p">(</span><span class="n">x_index</span><span class="p">,</span> <span class="n">bits</span><span class="p">);</span>
<span class="nb">disp</span><span class="p">(</span><span class="n">binary_codes</span><span class="p">(</span><span class="mi">1</span><span class="p">:</span><span class="mi">10</span><span class="p">,:));</span>   <span class="c1">% first 10 codes</span>
</code></pre>

</div>



<p>When concatenated, they form a <strong>digital bitstream</strong> — the format that computers and communication systems love.</p>




<h2>
  
  
  Points
</h2>

<ul>
<li>If you sample <strong>below Nyquist</strong>, you risk losing information (aliasing).</li>
<li>Increasing sampling rate improves signal quality but costs more data.</li>
<li>More quantization bits reduce distortion, but increase storage and bandwidth.</li>
<li>At the end, your “smooth” analog wave becomes a long sequence of <strong>1s and 0s</strong>.</li>
</ul>




<p>You can try experimenting with different frequencies and bit depths in your own code. It’s the best way to see how analog signals become digital and to appreciate the balance between accuracy and efficiency in real-world systems.</p>

