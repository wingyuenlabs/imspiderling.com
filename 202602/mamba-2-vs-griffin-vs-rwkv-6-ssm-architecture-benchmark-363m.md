---
Title: Mamba-2 vs Griffin vs RWKV-6: SSM Architecture Benchmark
Description: 
Author: TildAlice
Date: 2026-02-14T21:34:19.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Linear-Time Transformer Replacement Everyone's Building
</h2>

<p>The quadratic complexity of attention — $O(n^2)$ for sequence length $n$ — stopped being theoretical the moment context windows hit 128k tokens. State Space Models (SSMs) promise $O(n)$ complexity without sacrificing quality, but three architectures dominate 2026: Mamba-2, Griffin, and RWKV-6.</p>

<p>I benchmarked all three on the same 1.3B parameter budget. The results challenged what I thought I knew about attention alternatives.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Ftildalice.io%2Fwp-content%2Fuploads%2F2026%2F02%2Fstock-mamba-2-griffin-rwkv-6-ssm-benchmark-1.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Ftildalice.io%2Fwp-content%2Fuploads%2F2026%2F02%2Fstock-mamba-2-griffin-rwkv-6-ssm-benchmark-1.jpg" alt="Close-up of a Seagate FireCuda SSD on a white background with three yellow rubber ducks." width="800" height="599"></a></p>
Photo by <a href="https://www.pexels.com/@zeleboba" rel="nofollow noopener noreferrer">Andrey Matveev</a> on <a href="https://www.pexels.com" rel="nofollow noopener noreferrer">Pexels</a>



<h2>
  
  
  What Makes SSMs Different From Transformers
</h2>

<p>Transformers compute attention scores between every token pair. For a 10k token sequence, that's 100M comparisons. SSMs instead maintain a fixed-size hidden state that gets updated sequentially:</p>

<p>$$h_t = \bar{A}h_{t-1} + \bar{B}x_t$$<br>
$$y_t = Ch_t$$</p>

<p>The matrices $\bar{A}, \bar{B}, C$ are learned, but crucially: $h_t$ doesn't grow with sequence length. You process 10 tokens or 100k tokens with the same memory footprint.</p>




<p><em>Continue reading the full article on <a href="https://tildalice.io/mamba-2-griffin-rwkv-6-ssm-benchmark/" rel="noopener noreferrer">TildAlice</a></em></p>

