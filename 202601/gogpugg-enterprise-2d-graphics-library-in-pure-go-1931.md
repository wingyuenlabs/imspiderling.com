---
Title: gogpu/gg: Enterprise 2D Graphics Library in Pure Go
Description: 
Author: Andrey Kolkov
Date: 2026-01-22T22:05:00.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>Latest: v0.20.0</strong> — Enterprise-grade GPU backend, color emoji, anti-aliased rendering. Part of the <strong>280K+ LOC Pure Go</strong> graphics ecosystem.</p>

<p><strong>Previous:</strong> <a href="https://dev.to/kolkov/gpu-compute-shaders-in-pure-go-gogpugg-v0150-1cjk">GPU Compute Shaders in Pure Go (v0.15.0)</a></p>
</blockquote>

<h2>
  
  
  What is gogpu/gg?
</h2>

<p><strong>gg</strong> is a production-grade 2D graphics library for Go, designed to power:</p>

<ul>
<li>
<strong>IDEs</strong> — Syntax highlighting, code rendering, UI components</li>
<li>
<strong>Browsers</strong> — Canvas-like rendering, WebGPU acceleration</li>
<li>
<strong>Professional apps</strong> — Vector graphics, data visualization, games</li>
</ul>

<p><strong>Key differentiator:</strong> Pure Go. No CGO. No external dependencies. Just <code>go build</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go get github.com/gogpu/gg@v0.20.0
</code></pre>

</div>






<h2>
  
  
  Feature Overview
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Category</th>
<th>Capabilities</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Drawing</strong></td>
<td>Rectangles, circles, ellipses, arcs, lines, polygons, stars, bezier curves</td>
</tr>
<tr>
<td><strong>Paths</strong></td>
<td>MoveTo, LineTo, QuadraticTo, CubicTo, ArcTo, Close + fluent PathBuilder</td>
</tr>
<tr>
<td><strong>Text</strong></td>
<td>TrueType fonts, MSDF rendering, bidirectional text, color emoji</td>
</tr>
<tr>
<td><strong>Images</strong></td>
<td>PNG/JPEG I/O, 7 pixel formats, affine transforms, mipmaps</td>
</tr>
<tr>
<td><strong>Compositing</strong></td>
<td>29 blend modes (Porter-Duff, Advanced, HSL), layer isolation</td>
</tr>
<tr>
<td><strong>Rendering</strong></td>
<td>Software (CPU), GPU (Vulkan/Metal/DX12), hybrid auto-selection</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>SIMD optimization, parallel tile rendering, LRU caching</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Quick Start
</h2>

<h3>
  
  
  Hello World
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="s">"github.com/gogpu/gg"</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">dc</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">NewContext</span><span class="p">(</span><span class="m">512</span><span class="p">,</span> <span class="m">512</span><span class="p">)</span>
    <span class="k">defer</span> <span class="n">dc</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

    <span class="c">// Clear background</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">ClearWithColor</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">White</span><span class="p">)</span>

    <span class="c">// Draw anti-aliased circle</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">SetHexColor</span><span class="p">(</span><span class="s">"#3498db"</span><span class="p">)</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">DrawCircle</span><span class="p">(</span><span class="m">256</span><span class="p">,</span> <span class="m">256</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>

    <span class="c">// Draw stroked rectangle</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">SetHexColor</span><span class="p">(</span><span class="s">"#e74c3c"</span><span class="p">)</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">SetLineWidth</span><span class="p">(</span><span class="m">3</span><span class="p">)</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">DrawRectangle</span><span class="p">(</span><span class="m">156</span><span class="p">,</span> <span class="m">156</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">200</span><span class="p">)</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">Stroke</span><span class="p">()</span>

    <span class="n">dc</span><span class="o">.</span><span class="n">SavePNG</span><span class="p">(</span><span class="s">"output.png"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Text Rendering
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"github.com/gogpu/gg"</span>
    <span class="s">"github.com/gogpu/gg/text"</span>
<span class="p">)</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">dc</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">NewContext</span><span class="p">(</span><span class="m">800</span><span class="p">,</span> <span class="m">200</span><span class="p">)</span>
    <span class="k">defer</span> <span class="n">dc</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

    <span class="n">dc</span><span class="o">.</span><span class="n">ClearWithColor</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">White</span><span class="p">)</span>

    <span class="c">// Load font</span>
    <span class="n">source</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewFontSourceFromFile</span><span class="p">(</span><span class="s">"Roboto-Regular.ttf"</span><span class="p">)</span>
    <span class="k">defer</span> <span class="n">source</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

    <span class="c">// Render text</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">SetFont</span><span class="p">(</span><span class="n">source</span><span class="o">.</span><span class="n">Face</span><span class="p">(</span><span class="m">48</span><span class="p">))</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">SetColor</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">Black</span><span class="p">)</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">DrawString</span><span class="p">(</span><span class="s">"Hello, GoGPU!"</span><span class="p">,</span> <span class="m">50</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>

    <span class="c">// Centered text</span>
    <span class="n">dc</span><span class="o">.</span><span class="n">DrawStringAnchored</span><span class="p">(</span><span class="s">"Centered Text"</span><span class="p">,</span> <span class="m">400</span><span class="p">,</span> <span class="m">150</span><span class="p">,</span> <span class="m">0.5</span><span class="p">,</span> <span class="m">0.5</span><span class="p">)</span>

    <span class="n">dc</span><span class="o">.</span><span class="n">SavePNG</span><span class="p">(</span><span class="s">"text.png"</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Gradients
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">dc</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">NewContext</span><span class="p">(</span><span class="m">400</span><span class="p">,</span> <span class="m">400</span><span class="p">)</span>
<span class="k">defer</span> <span class="n">dc</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

<span class="c">// Linear gradient</span>
<span class="n">grad</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">NewLinearGradientBrush</span><span class="p">(</span><span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">400</span><span class="p">,</span> <span class="m">400</span><span class="p">)</span>
<span class="n">grad</span><span class="o">.</span><span class="n">AddColorStop</span><span class="p">(</span><span class="m">0</span><span class="p">,</span> <span class="n">gg</span><span class="o">.</span><span class="n">Hex</span><span class="p">(</span><span class="s">"#ff6b6b"</span><span class="p">))</span>
<span class="n">grad</span><span class="o">.</span><span class="n">AddColorStop</span><span class="p">(</span><span class="m">0.5</span><span class="p">,</span> <span class="n">gg</span><span class="o">.</span><span class="n">Hex</span><span class="p">(</span><span class="s">"#4ecdc4"</span><span class="p">))</span>
<span class="n">grad</span><span class="o">.</span><span class="n">AddColorStop</span><span class="p">(</span><span class="m">1</span><span class="p">,</span> <span class="n">gg</span><span class="o">.</span><span class="n">Hex</span><span class="p">(</span><span class="s">"#45b7d1"</span><span class="p">))</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SetFillBrush</span><span class="p">(</span><span class="n">grad</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawRectangle</span><span class="p">(</span><span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">400</span><span class="p">,</span> <span class="m">400</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SavePNG</span><span class="p">(</span><span class="s">"gradient.png"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Transforms
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">dc</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">NewContext</span><span class="p">(</span><span class="m">400</span><span class="p">,</span> <span class="m">400</span><span class="p">)</span>
<span class="k">defer</span> <span class="n">dc</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>

<span class="n">dc</span><span class="o">.</span><span class="n">ClearWithColor</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">White</span><span class="p">)</span>

<span class="c">// Save state, transform, draw, restore</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Push</span><span class="p">()</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Translate</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="m">200</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Rotate</span><span class="p">(</span><span class="n">math</span><span class="o">.</span><span class="n">Pi</span> <span class="o">/</span> <span class="m">4</span><span class="p">)</span>  <span class="c">// 45 degrees</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Scale</span><span class="p">(</span><span class="m">1.5</span><span class="p">,</span> <span class="m">1.5</span><span class="p">)</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SetHexColor</span><span class="p">(</span><span class="s">"#9b59b6"</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawRectangle</span><span class="p">(</span><span class="o">-</span><span class="m">50</span><span class="p">,</span> <span class="o">-</span><span class="m">50</span><span class="p">,</span> <span class="m">100</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>

<span class="n">dc</span><span class="o">.</span><span class="n">Pop</span><span class="p">()</span>  <span class="c">// Restore original state</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SavePNG</span><span class="p">(</span><span class="s">"transform.png"</span><span class="p">)</span>
</code></pre>

</div>






<h2>
  
  
  Canvas API
</h2>

<p>gg provides a familiar Canvas-like API inspired by HTML5 Canvas and Cairo:</p>

<h3>
  
  
  Drawing Primitives
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Rectangles</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawRectangle</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">width</span><span class="p">,</span> <span class="n">height</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawRoundedRectangle</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">,</span> <span class="n">w</span><span class="p">,</span> <span class="n">h</span><span class="p">,</span> <span class="n">radius</span><span class="p">)</span>

<span class="c">// Circles and ellipses</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawCircle</span><span class="p">(</span><span class="n">cx</span><span class="p">,</span> <span class="n">cy</span><span class="p">,</span> <span class="n">radius</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawEllipse</span><span class="p">(</span><span class="n">cx</span><span class="p">,</span> <span class="n">cy</span><span class="p">,</span> <span class="n">rx</span><span class="p">,</span> <span class="n">ry</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawArc</span><span class="p">(</span><span class="n">cx</span><span class="p">,</span> <span class="n">cy</span><span class="p">,</span> <span class="n">r</span><span class="p">,</span> <span class="n">startAngle</span><span class="p">,</span> <span class="n">endAngle</span><span class="p">)</span>

<span class="c">// Lines</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawLine</span><span class="p">(</span><span class="n">x1</span><span class="p">,</span> <span class="n">y1</span><span class="p">,</span> <span class="n">x2</span><span class="p">,</span> <span class="n">y2</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">MoveTo</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">LineTo</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>

<span class="c">// Curves</span>
<span class="n">dc</span><span class="o">.</span><span class="n">QuadraticTo</span><span class="p">(</span><span class="n">cx</span><span class="p">,</span> <span class="n">cy</span><span class="p">,</span> <span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">CubicTo</span><span class="p">(</span><span class="n">cx1</span><span class="p">,</span> <span class="n">cy1</span><span class="p">,</span> <span class="n">cx2</span><span class="p">,</span> <span class="n">cy2</span><span class="p">,</span> <span class="n">x</span><span class="p">,</span> <span class="n">y</span><span class="p">)</span>

<span class="c">// Polygons</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawPolygon</span><span class="p">(</span><span class="n">points</span><span class="o">...</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawRegularPolygon</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">cx</span><span class="p">,</span> <span class="n">cy</span><span class="p">,</span> <span class="n">r</span><span class="p">,</span> <span class="n">rotation</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawStar</span><span class="p">(</span><span class="n">cx</span><span class="p">,</span> <span class="n">cy</span><span class="p">,</span> <span class="n">outerR</span><span class="p">,</span> <span class="n">innerR</span><span class="p">,</span> <span class="n">points</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Styles
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Colors</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetColor</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">Red</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetRGB</span><span class="p">(</span><span class="m">1.0</span><span class="p">,</span> <span class="m">0.5</span><span class="p">,</span> <span class="m">0.0</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetRGBA</span><span class="p">(</span><span class="m">1.0</span><span class="p">,</span> <span class="m">0.5</span><span class="p">,</span> <span class="m">0.0</span><span class="p">,</span> <span class="m">0.8</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetHexColor</span><span class="p">(</span><span class="s">"#3498db"</span><span class="p">)</span>

<span class="c">// Strokes</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetLineWidth</span><span class="p">(</span><span class="m">2.0</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetLineCap</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">LineCapRound</span><span class="p">)</span>    <span class="c">// Butt, Round, Square</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetLineJoin</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">LineJoinMiter</span><span class="p">)</span>  <span class="c">// Miter, Round, Bevel</span>
<span class="n">dc</span><span class="o">.</span><span class="n">SetDash</span><span class="p">(</span><span class="m">5</span><span class="p">,</span> <span class="m">3</span><span class="p">)</span>  <span class="c">// Pattern: 5px on, 3px off</span>

<span class="c">// Fill and stroke</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Stroke</span><span class="p">()</span>
<span class="n">dc</span><span class="o">.</span><span class="n">FillPreserve</span><span class="p">()</span>   <span class="c">// Fill but keep path</span>
<span class="n">dc</span><span class="o">.</span><span class="n">StrokePreserve</span><span class="p">()</span> <span class="c">// Stroke but keep path</span>
</code></pre>

</div>



<h3>
  
  
  Clipping
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Clip to circle</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawCircle</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Clip</span><span class="p">()</span>

<span class="c">// All subsequent drawing is clipped</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawImage</span><span class="p">(</span><span class="n">img</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">)</span>

<span class="n">dc</span><span class="o">.</span><span class="n">ResetClip</span><span class="p">()</span>
</code></pre>

</div>






<h2>
  
  
  Fluent PathBuilder
</h2>

<p>Build complex paths with method chaining for use with Scene Graph:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">import</span> <span class="p">(</span>
    <span class="s">"github.com/gogpu/gg"</span>
    <span class="s">"github.com/gogpu/gg/scene"</span>
<span class="p">)</span>

<span class="c">// Build path with fluent API</span>
<span class="n">path</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">BuildPath</span><span class="p">()</span><span class="o">.</span>
    <span class="n">MoveTo</span><span class="p">(</span><span class="m">100</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span><span class="o">.</span>
    <span class="n">LineTo</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span><span class="o">.</span>
    <span class="n">QuadTo</span><span class="p">(</span><span class="m">250</span><span class="p">,</span> <span class="m">150</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">200</span><span class="p">)</span><span class="o">.</span>
    <span class="n">CubicTo</span><span class="p">(</span><span class="m">150</span><span class="p">,</span> <span class="m">250</span><span class="p">,</span> <span class="m">100</span><span class="p">,</span> <span class="m">250</span><span class="p">,</span> <span class="m">50</span><span class="p">,</span> <span class="m">200</span><span class="p">)</span><span class="o">.</span>
    <span class="n">Close</span><span class="p">()</span><span class="o">.</span>
    <span class="n">Circle</span><span class="p">(</span><span class="m">300</span><span class="p">,</span> <span class="m">150</span><span class="p">,</span> <span class="m">50</span><span class="p">)</span><span class="o">.</span>
    <span class="n">Star</span><span class="p">(</span><span class="m">400</span><span class="p">,</span> <span class="m">150</span><span class="p">,</span> <span class="m">40</span><span class="p">,</span> <span class="m">20</span><span class="p">,</span> <span class="m">5</span><span class="p">)</span><span class="o">.</span>
    <span class="n">RoundRect</span><span class="p">(</span><span class="m">50</span><span class="p">,</span> <span class="m">250</span><span class="p">,</span> <span class="m">100</span><span class="p">,</span> <span class="m">80</span><span class="p">,</span> <span class="m">10</span><span class="p">)</span><span class="o">.</span>
    <span class="n">Build</span><span class="p">()</span>

<span class="c">// Use with scene graph</span>
<span class="n">s</span> <span class="o">:=</span> <span class="n">scene</span><span class="o">.</span><span class="n">NewScene</span><span class="p">()</span>
<span class="n">s</span><span class="o">.</span><span class="n">Fill</span><span class="p">(</span><span class="n">scene</span><span class="o">.</span><span class="n">FillNonZero</span><span class="p">,</span> <span class="n">scene</span><span class="o">.</span><span class="n">IdentityAffine</span><span class="p">(),</span>
    <span class="n">scene</span><span class="o">.</span><span class="n">SolidBrush</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">Hex</span><span class="p">(</span><span class="s">"#2ecc71"</span><span class="p">)),</span> <span class="n">scene</span><span class="o">.</span><span class="n">NewPathShape</span><span class="p">(</span><span class="n">path</span><span class="p">))</span>
</code></pre>

</div>



<p><strong>Available shape methods:</strong></p>

<ul>
<li>
<code>MoveTo</code>, <code>LineTo</code>, <code>QuadTo</code>, <code>CubicTo</code>, <code>Close</code>
</li>
<li>
<code>Circle</code>, <code>Ellipse</code>
</li>
<li>
<code>Rect</code>, <code>RoundRect</code>
</li>
<li>
<code>Polygon</code>, <code>Star</code>
</li>
</ul>




<h2>
  
  
  Text Rendering
</h2>

<h3>
  
  
  Font Loading
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// From file</span>
<span class="n">source</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewFontSourceFromFile</span><span class="p">(</span><span class="s">"font.ttf"</span><span class="p">)</span>

<span class="c">// From bytes</span>
<span class="n">source</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewFontSourceFromBytes</span><span class="p">(</span><span class="n">fontBytes</span><span class="p">)</span>

<span class="c">// Create face at specific size</span>
<span class="n">face</span> <span class="o">:=</span> <span class="n">source</span><span class="o">.</span><span class="n">Face</span><span class="p">(</span><span class="m">24</span><span class="p">)</span>  <span class="c">// 24pt</span>
<span class="k">defer</span> <span class="n">source</span><span class="o">.</span><span class="n">Close</span><span class="p">()</span>
</code></pre>

</div>



<h3>
  
  
  Multi-Face (Font Fallback)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Primary font + emoji fallback</span>
<span class="n">mainFont</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewFontSourceFromFile</span><span class="p">(</span><span class="s">"Roboto.ttf"</span><span class="p">)</span>
<span class="n">emojiFont</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewFontSourceFromFile</span><span class="p">(</span><span class="s">"NotoColorEmoji.ttf"</span><span class="p">)</span>

<span class="n">multiFace</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewMultiFace</span><span class="p">(</span>
    <span class="n">mainFont</span><span class="o">.</span><span class="n">Face</span><span class="p">(</span><span class="m">16</span><span class="p">),</span>
    <span class="n">text</span><span class="o">.</span><span class="n">NewFilteredFace</span><span class="p">(</span><span class="n">emojiFont</span><span class="o">.</span><span class="n">Face</span><span class="p">(</span><span class="m">16</span><span class="p">),</span> <span class="n">text</span><span class="o">.</span><span class="n">RangeEmoji</span><span class="p">),</span>
<span class="p">)</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SetFont</span><span class="p">(</span><span class="n">multiFace</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawString</span><span class="p">(</span><span class="s">"Hello World! 🎉🚀"</span><span class="p">,</span> <span class="m">50</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Text Layout with Wrapping
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">opts</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">LayoutOptions</span><span class="p">{</span>
    <span class="n">MaxWidth</span><span class="o">:</span>    <span class="m">400</span><span class="p">,</span>
    <span class="n">WrapMode</span><span class="o">:</span>    <span class="n">text</span><span class="o">.</span><span class="n">WrapWordChar</span><span class="p">,</span>  <span class="c">// Word-first, char fallback</span>
    <span class="n">Alignment</span><span class="o">:</span>   <span class="n">text</span><span class="o">.</span><span class="n">AlignCenter</span><span class="p">,</span>
    <span class="n">LineSpacing</span><span class="o">:</span> <span class="m">1.2</span><span class="p">,</span>
<span class="p">}</span>

<span class="n">layout</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">LayoutText</span><span class="p">(</span><span class="n">longText</span><span class="p">,</span> <span class="n">face</span><span class="p">,</span> <span class="m">16</span><span class="p">,</span> <span class="n">opts</span><span class="p">)</span>

<span class="c">// Render using glyph renderer</span>
<span class="n">renderer</span> <span class="o">:=</span> <span class="n">text</span><span class="o">.</span><span class="n">NewGlyphRenderer</span><span class="p">()</span>
<span class="n">outlines</span> <span class="o">:=</span> <span class="n">renderer</span><span class="o">.</span><span class="n">RenderLayout</span><span class="p">(</span><span class="n">layout</span><span class="p">)</span>

<span class="c">// Or access line metrics directly</span>
<span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">line</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">layout</span><span class="o">.</span><span class="n">Lines</span> <span class="p">{</span>
    <span class="c">// line.Y is the baseline Y position</span>
    <span class="c">// line.Width, line.Ascent, line.Descent available</span>
    <span class="c">// line.Glyphs contains positioned glyphs</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Color Emoji
</h3>

<p>gg supports both bitmap (CBDT/CBLC) and vector (COLR/CPAL) color emoji:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Extract bitmap emoji (Noto Color Emoji)</span>
<span class="n">extractor</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">emoji</span><span class="o">.</span><span class="n">NewCBDTExtractor</span><span class="p">(</span><span class="n">cbdtData</span><span class="p">,</span> <span class="n">cblcData</span><span class="p">)</span>
<span class="n">glyph</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">extractor</span><span class="o">.</span><span class="n">GetGlyph</span><span class="p">(</span><span class="n">glyphID</span><span class="p">,</span> <span class="n">ppem</span><span class="p">)</span>
<span class="n">img</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">png</span><span class="o">.</span><span class="n">Decode</span><span class="p">(</span><span class="n">bytes</span><span class="o">.</span><span class="n">NewReader</span><span class="p">(</span><span class="n">glyph</span><span class="o">.</span><span class="n">Data</span><span class="p">))</span>

<span class="c">// Parse vector emoji layers (Segoe UI Emoji)</span>
<span class="n">parser</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">emoji</span><span class="o">.</span><span class="n">NewCOLRParser</span><span class="p">(</span><span class="n">colrData</span><span class="p">,</span> <span class="n">cpalData</span><span class="p">)</span>
<span class="n">glyph</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">parser</span><span class="o">.</span><span class="n">GetGlyph</span><span class="p">(</span><span class="n">glyphID</span><span class="p">,</span> <span class="n">paletteIndex</span><span class="p">)</span>
<span class="k">for</span> <span class="n">_</span><span class="p">,</span> <span class="n">layer</span> <span class="o">:=</span> <span class="k">range</span> <span class="n">glyph</span><span class="o">.</span><span class="n">Layers</span> <span class="p">{</span>
    <span class="c">// Render each layer with layer.Color</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Layer Compositing
</h2>

<h3>
  
  
  29 Blend Modes
</h3>

<p><strong>Porter-Duff:</strong> Clear, Src, Dst, SrcOver, DstOver, SrcIn, DstIn, SrcOut, DstOut, SrcAtop, DstAtop, Xor</p>

<p><strong>Advanced:</strong> Multiply, Screen, Overlay, Darken, Lighten, ColorDodge, ColorBurn, HardLight, SoftLight, Difference, Exclusion</p>

<p><strong>HSL:</strong> Hue, Saturation, Color, Luminosity<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="n">dc</span><span class="o">.</span><span class="n">PushLayer</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">BlendMultiply</span><span class="p">,</span> <span class="m">0.8</span><span class="p">)</span>  <span class="c">// Blend mode + opacity</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SetHexColor</span><span class="p">(</span><span class="s">"#e74c3c"</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawCircle</span><span class="p">(</span><span class="m">150</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>

<span class="n">dc</span><span class="o">.</span><span class="n">SetHexColor</span><span class="p">(</span><span class="s">"#3498db"</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawCircle</span><span class="p">(</span><span class="m">250</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>

<span class="n">dc</span><span class="o">.</span><span class="n">PopLayer</span><span class="p">()</span>  <span class="c">// Composite layer</span>
</code></pre>

</div>



<h3>
  
  
  Alpha Masks
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Create mask from shape</span>
<span class="n">dc</span><span class="o">.</span><span class="n">DrawCircle</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">)</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>
<span class="n">mask</span> <span class="o">:=</span> <span class="n">dc</span><span class="o">.</span><span class="n">AsMask</span><span class="p">()</span>

<span class="c">// Apply mask to new context</span>
<span class="n">dc2</span> <span class="o">:=</span> <span class="n">gg</span><span class="o">.</span><span class="n">NewContext</span><span class="p">(</span><span class="m">400</span><span class="p">,</span> <span class="m">400</span><span class="p">)</span>
<span class="n">dc2</span><span class="o">.</span><span class="n">SetMask</span><span class="p">(</span><span class="n">mask</span><span class="p">)</span>
<span class="n">dc2</span><span class="o">.</span><span class="n">DrawImage</span><span class="p">(</span><span class="n">backgroundImage</span><span class="p">,</span> <span class="m">0</span><span class="p">,</span> <span class="m">0</span><span class="p">)</span>  <span class="c">// Only visible through mask</span>
</code></pre>

</div>






<h2>
  
  
  Scene Graph (Retained Mode)
</h2>

<p>For complex, frequently-updated scenes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">import</span> <span class="p">(</span>
    <span class="s">"github.com/gogpu/gg"</span>
    <span class="s">"github.com/gogpu/gg/scene"</span>
<span class="p">)</span>

<span class="n">s</span> <span class="o">:=</span> <span class="n">scene</span><span class="o">.</span><span class="n">NewScene</span><span class="p">()</span>

<span class="c">// Build scene graph with layer compositing</span>
<span class="n">s</span><span class="o">.</span><span class="n">PushLayer</span><span class="p">(</span><span class="n">scene</span><span class="o">.</span><span class="n">BlendNormal</span><span class="p">,</span> <span class="m">1.0</span><span class="p">,</span> <span class="no">nil</span><span class="p">)</span>  <span class="c">// blend, alpha, clip</span>

<span class="n">s</span><span class="o">.</span><span class="n">Fill</span><span class="p">(</span><span class="n">scene</span><span class="o">.</span><span class="n">FillNonZero</span><span class="p">,</span> <span class="n">scene</span><span class="o">.</span><span class="n">IdentityAffine</span><span class="p">(),</span>
    <span class="n">scene</span><span class="o">.</span><span class="n">SolidBrush</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">Red</span><span class="p">),</span> <span class="n">scene</span><span class="o">.</span><span class="n">NewCircleShape</span><span class="p">(</span><span class="m">150</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">))</span>

<span class="n">s</span><span class="o">.</span><span class="n">Fill</span><span class="p">(</span><span class="n">scene</span><span class="o">.</span><span class="n">FillNonZero</span><span class="p">,</span> <span class="n">scene</span><span class="o">.</span><span class="n">IdentityAffine</span><span class="p">(),</span>
    <span class="n">scene</span><span class="o">.</span><span class="n">SolidBrush</span><span class="p">(</span><span class="n">gg</span><span class="o">.</span><span class="n">Blue</span><span class="p">),</span> <span class="n">scene</span><span class="o">.</span><span class="n">NewCircleShape</span><span class="p">(</span><span class="m">250</span><span class="p">,</span> <span class="m">200</span><span class="p">,</span> <span class="m">100</span><span class="p">))</span>

<span class="n">s</span><span class="o">.</span><span class="n">PopLayer</span><span class="p">()</span>

<span class="c">// Render to pixmap</span>
<span class="n">renderer</span> <span class="o">:=</span> <span class="n">scene</span><span class="o">.</span><span class="n">NewRenderer</span><span class="p">(</span><span class="n">width</span><span class="p">,</span> <span class="n">height</span><span class="p">)</span>
<span class="n">renderer</span><span class="o">.</span><span class="n">Render</span><span class="p">(</span><span class="n">s</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Benefits:</strong></p>

<ul>
<li>GPU-optimized command batching</li>
<li>Efficient dirty region tracking</li>
<li>Parallel tile rendering</li>
</ul>




<h2>
  
  
  Backend Architecture
</h2>

<p>gg supports three rendering backends:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Backend</th>
<th>Package</th>
<th>Description</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Native</strong></td>
<td><code>backend/native/</code></td>
<td>Pure Go via gogpu/wgpu</td>
<td>Default, zero deps</td>
</tr>
<tr>
<td><strong>Rust</strong></td>
<td><code>backend/rust/</code></td>
<td>wgpu-native FFI</td>
<td>Max performance</td>
</tr>
<tr>
<td><strong>Software</strong></td>
<td><code>backend/software/</code></td>
<td>CPU rasterizer</td>
<td>Fallback</td>
</tr>
</tbody>
</table></div>



<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">import</span> <span class="s">"github.com/gogpu/gg/backend"</span>

<span class="c">// Auto-select best available</span>
<span class="n">b</span> <span class="o">:=</span> <span class="n">backend</span><span class="o">.</span><span class="n">Default</span><span class="p">()</span>

<span class="c">// Explicit selection</span>
<span class="n">b</span> <span class="o">:=</span> <span class="n">backend</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">backend</span><span class="o">.</span><span class="n">BackendNative</span><span class="p">)</span>
<span class="n">b</span> <span class="o">:=</span> <span class="n">backend</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">backend</span><span class="o">.</span><span class="n">BackendRust</span><span class="p">)</span>     <span class="c">// Requires -tags rust</span>
<span class="n">b</span> <span class="o">:=</span> <span class="n">backend</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">backend</span><span class="o">.</span><span class="n">BackendSoftware</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  GPU Backend Features (v0.20.0)
</h3>

<ul>
<li>
<strong>Command Encoder</strong> — State machine for GPU command recording</li>
<li>
<strong>Texture Management</strong> — Lazy views with <code>sync.Once</code>
</li>
<li>
<strong>Buffer Mapping</strong> — Async with device polling</li>
<li>
<strong>Pipeline Cache</strong> — FNV-1a descriptor hashing</li>
<li>
<strong>Compute Shaders</strong> — WGSL shaders for path rasterization</li>
</ul>




<h2>
  
  
  Performance
</h2>

<h3>
  
  
  Benchmarks
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Operation</th>
<th>Time</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>sRGB → Linear</td>
<td>0.16ns</td>
<td>260x faster than math.Pow</td>
</tr>
<tr>
<td>LayerCache.Get</td>
<td>90ns</td>
<td>Thread-safe LRU</td>
</tr>
<tr>
<td>DirtyRegion.Mark</td>
<td>10.9ns</td>
<td>Lock-free atomic</td>
</tr>
<tr>
<td>MSDF lookup</td>
<td>&lt;10ns</td>
<td>Zero-allocation</td>
</tr>
<tr>
<td>Path iteration</td>
<td>438ns</td>
<td>iter.Seq, 0 allocs</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Optimizations
</h3>

<ul>
<li>
<strong>SIMD</strong> — Vectorized color operations</li>
<li>
<strong>Parallel rendering</strong> — Tile-based multi-threaded rasterization</li>
<li>
<strong>LRU caching</strong> — Glyph cache, texture cache, layer cache</li>
<li>
<strong>GPU compute</strong> — Path flattening, tile binning, coverage calculation</li>
</ul>




<h2>
  
  
  Anti-Aliased Rendering
</h2>

<p>v0.19.0 introduced professional-grade anti-aliasing using the <strong>tiny-skia algorithm</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// AA is enabled by default</span>
<span class="n">dc</span><span class="o">.</span><span class="n">Fill</span><span class="p">()</span>

<span class="c">// Disable AA for performance</span>
<span class="n">dc</span><span class="o">.</span><span class="n">FillNoAA</span><span class="p">()</span>
</code></pre>

</div>



<p><strong>Implementation:</strong></p>

<ul>
<li>4x supersampling with coverage accumulation</li>
<li>
<code>AlphaRuns</code> — RLE-encoded sparse alpha buffer</li>
<li>SIMD batch blending for 16 pixels at a time</li>
<li>Same algorithm as Chrome, Android, Flutter</li>
</ul>




<h2>
  
  
  The GoGPU Ecosystem
</h2>

<p>gg is part of a complete Pure Go graphics stack:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Project</th>
<th>Version</th>
<th>LOC</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>gogpu/gg</strong></td>
<td>v0.20.0</td>
<td>~122K</td>
<td>2D graphics (this library)</td>
</tr>
<tr>
<td>gogpu/wgpu</td>
<td>v0.10.1</td>
<td>~95K</td>
<td>Pure Go WebGPU</td>
</tr>
<tr>
<td>gogpu/naga</td>
<td>v0.8.4</td>
<td>~33K</td>
<td>WGSL shader compiler</td>
</tr>
<tr>
<td>gogpu/gogpu</td>
<td>v0.11.1</td>
<td>~28K</td>
<td>Graphics framework</td>
</tr>
</tbody>
</table></div>

<p><strong>Total: ~280K lines of Pure Go.</strong></p>




<h2>
  
  
  Design Principles
</h2>

<ol>
<li>
<strong>Pure Go</strong> — No CGO, easy cross-compilation, single binary</li>
<li>
<strong>GPU-First</strong> — Designed for GPU acceleration from day one</li>
<li>
<strong>Production-Ready</strong> — Enterprise-grade error handling, logging</li>
<li>
<strong>API Stability</strong> — Semantic versioning, deprecation policy</li>
</ol>

<h3>
  
  
  Inspired By
</h3>

<ul>
<li>
<strong>vello</strong> (Rust) — GPU compute shaders, sparse strips</li>
<li>
<strong>tiny-skia</strong> (Rust) — Anti-aliasing, stroke expansion</li>
<li>
<strong>kurbo</strong> (Rust) — Path algorithms</li>
<li>
<strong>peniko</strong> (Rust) — Brush/paint system</li>
<li>
<strong>skrifa/swash</strong> (Rust) — Font parsing, color emoji</li>
</ul>




<h2>
  
  
  Installation
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go get github.com/gogpu/gg@v0.20.0
</code></pre>

</div>



<p><strong>Requirements:</strong> Go 1.25+</p>

<h3>
  
  
  Build Tags
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Default: Native + Software backends</span>
go build ./...

<span class="c"># With Rust backend (requires wgpu-native)</span>
go build <span class="nt">-tags</span> rust ./...

<span class="c"># Software only</span>
go build <span class="nt">-tags</span> nogpu ./...
</code></pre>

</div>






<h2>
  
  
  Examples
</h2>

<h3>
  
  
  Complete Examples
</h3>

<ul>
<li>
<a href="https://github.com/gogpu/gg/tree/main/examples/basic" rel="noopener noreferrer"><code>examples/basic/</code></a> — Basic shapes and colors</li>
<li>
<a href="https://github.com/gogpu/gg/tree/main/examples/text" rel="noopener noreferrer"><code>examples/text/</code></a> — Text rendering</li>
<li>
<a href="https://github.com/gogpu/gg/tree/main/examples/color_emoji" rel="noopener noreferrer"><code>examples/color_emoji/</code></a> — Color emoji extraction</li>
<li>
<a href="https://github.com/gogpu/gg/tree/main/examples/gpu" rel="noopener noreferrer"><code>examples/gpu/</code></a> — GPU backend usage</li>
<li>
<a href="https://github.com/gogpu/gg/tree/main/examples/scene" rel="noopener noreferrer"><code>examples/scene/</code></a> — Scene graph</li>
</ul>

<h3>
  
  
  Run Examples
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cd </span>examples/basic
go run main.go
</code></pre>

</div>






<h2>
  
  
  Documentation
</h2>

<ul>
<li>
<strong><a href="https://github.com/gogpu/gg" rel="noopener noreferrer">README</a></strong> — Quick start</li>
<li>
<strong><a href="https://github.com/gogpu/gg/blob/main/docs/ARCHITECTURE.md" rel="noopener noreferrer">ARCHITECTURE.md</a></strong> — System design</li>
<li>
<strong><a href="https://github.com/gogpu/gg/blob/main/ROADMAP.md" rel="noopener noreferrer">ROADMAP.md</a></strong> — Development milestones</li>
<li>
<strong><a href="https://pkg.go.dev/github.com/gogpu/gg" rel="noopener noreferrer">pkg.go.dev</a></strong> — API reference</li>
</ul>




<h2>
  
  
  Links
</h2>

<ul>
<li>
<strong>Repository:</strong> <a href="https://github.com/gogpu/gg" rel="noopener noreferrer">https://github.com/gogpu/gg</a>
</li>
<li>
<strong>Release v0.20.0:</strong> <a href="https://github.com/gogpu/gg/releases/tag/v0.20.0" rel="noopener noreferrer">https://github.com/gogpu/gg/releases/tag/v0.20.0</a>
</li>
<li>
<strong>GoGPU Organization:</strong> <a href="https://github.com/gogpu" rel="noopener noreferrer">https://github.com/gogpu</a>
</li>
<li>
<strong>Discussions:</strong> <a href="https://github.com/orgs/gogpu/discussions" rel="noopener noreferrer">https://github.com/orgs/gogpu/discussions</a>
</li>
</ul>




<p><strong>122K lines. Pure Go. Production-ready 2D graphics.</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go get github.com/gogpu/gg@v0.20.0
</code></pre>

</div>



<p>Star the repo if you find it useful!</p>




<p><em>Part of the GoGPU Journey series</em></p>

