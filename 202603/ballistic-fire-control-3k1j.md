---
Title: Building a Ballistic Fire Control Simulator with Python, C# and Redis
Description: 
Author: Patryk
Date: 2026-03-23T21:44:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>description: How I built a real-time ballistic computer with Euler simulation, Coriolis effect, blast zones and a satellite map interface using microservices architecture.<br>
tags: python, csharp, redis, opensource</p>


<h1>
  
  
  Building a Ballistic Fire Control Simulator — BALISTIC V5
</h1>

<p>A few weeks ago I started wondering: <em>how hard would it be to simulate real external ballistics?</em> Not just a simple parabola — but actual physics with air resistance, wind, air density and even the Coriolis effect.</p>

<p>The result is <strong>BALISTIC V5</strong> — a full microservices ballistic fire control simulator.</p>

<p>👉 <strong>GitHub:</strong> <a href="https://github.com/InsaneInfinity/Balistic" rel="noopener noreferrer">https://github.com/InsaneInfinity/Balistic</a></p>


<h2>
  
  
  🏗️ Architecture
</h2>

<p>The system is split into two independent services communicating via <strong>Redis Streams</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────┐     Redis Stream      ┌──────────────────────┐
│   Python / Flask    │ ──────────────────►   │    C# Processor      │
│                     │   ballistics:stream   │                      │
│  - Leaflet Map      │                       │  - Euler Simulation  │
│  - Weather API      │ ◄──────────────────   │  - Coriolis Effect   │
│  - Results Panel    │   ballistics:result   │  - Blast Zones       │
│  - PDF Export       │                       │                      │
└─────────────────────┘                       └──────────────────────┘
</code></pre>

</div>



<p>Why Redis Streams instead of a simple key-value? Because every shot is an independent entry — no data gets overwritten during sequential multi-target fire.</p>




<h2>
  
  
  🧮 Physics Model
</h2>

<h3>
  
  
  Trajectory Simulation
</h3>

<p>For <strong>kinetic energy rounds</strong> (APFSDS, HEAT) I use an <strong>Euler method</strong> simulation with <code>dt=0.01s</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="kt">double</span> <span class="n">v</span> <span class="p">=</span> <span class="n">Math</span><span class="p">.</span><span class="nf">Sqrt</span><span class="p">(</span><span class="n">vx</span><span class="p">*</span><span class="n">vx</span> <span class="p">+</span> <span class="n">vy</span><span class="p">*</span><span class="n">vy</span><span class="p">);</span>
<span class="kt">double</span> <span class="n">drag</span> <span class="p">=</span> <span class="m">0.5</span> <span class="p">*</span> <span class="n">cd</span> <span class="p">*</span> <span class="n">rho</span> <span class="p">*</span> <span class="n">area</span> <span class="p">*</span> <span class="n">v</span> <span class="p">*</span> <span class="n">v</span><span class="p">;</span>
<span class="n">vx</span> <span class="p">+=</span> <span class="p">-(</span><span class="n">drag</span><span class="p">/</span><span class="n">mass</span><span class="p">)*(</span><span class="n">vx</span><span class="p">/</span><span class="n">v</span><span class="p">)*</span><span class="n">dt</span><span class="p">;</span>
<span class="n">vy</span> <span class="p">+=</span> <span class="p">(-</span><span class="m">9.81</span> <span class="p">-</span> <span class="p">(</span><span class="n">drag</span><span class="p">/</span><span class="n">mass</span><span class="p">)*(</span><span class="n">vy</span><span class="p">/</span><span class="n">v</span><span class="p">))*</span><span class="n">dt</span><span class="p">;</span>
</code></pre>

</div>



<p>For <strong>artillery rounds</strong> (HE, EXCALIBUR) I use an empirical angle formula based on the dist/max_range ratio — artillery always prefers a high-arc trajectory (45°–65°).</p>

<h3>
  
  
  Coriolis Effect
</h3>

<p>This was the most interesting part. The real deflection formula:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>d_cor = Ω · sin(φ) · v_avg · t² / 2
</code></pre>

</div>



<p>Where:</p>

<ul>
<li>
<code>Ω = 7.2921×10⁻⁵ rad/s</code> — Earth's angular velocity</li>
<li>
<code>φ</code> — shooter's latitude (dynamic, updates with every position change)</li>
<li>
<code>v_avg = dist / tof</code> — average horizontal velocity during flight</li>
</ul>

<p>Results for M107 HE at ~5km range, 54°N latitude: <strong>~3m deflection to the right</strong> — matches NATO FM 6-40 tables.</p>

<h3>
  
  
  Air Density
</h3>

<p>Calculated live from OpenWeatherMap data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">dens</span> <span class="o">=</span> <span class="p">(</span><span class="n">pressure</span> <span class="o">*</span> <span class="mi">100</span><span class="p">)</span> <span class="o">/</span> <span class="p">(</span><span class="mf">287.058</span> <span class="o">*</span> <span class="p">(</span><span class="n">temp</span> <span class="o">+</span> <span class="mf">273.15</span><span class="p">))</span>
</code></pre>

</div>






<h2>
  
  
  💥 Blast Zones
</h2>

<p>Based on NATO / FM 6-40 publicly available data:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Round</th>
<th>Total destruction</th>
<th>Heavy damage</th>
<th>Light damage</th>
<th>Hazard zone</th>
</tr>
</thead>
<tbody>
<tr>
<td>M107 HE / EXCALIBUR</td>
<td>30 m</td>
<td>100 m</td>
<td>300 m</td>
<td><strong>800 m</strong></td>
</tr>
<tr>
<td>HE STD (RAK 120mm)</td>
<td>8 m</td>
<td>30 m</td>
<td>80 m</td>
<td>200 m</td>
</tr>
<tr>
<td>APFSDS</td>
<td>kinetic only</td>
<td>—</td>
<td>—</td>
<td>—</td>
</tr>
<tr>
<td>HEAT</td>
<td>—</td>
<td>—</td>
<td>—</td>
<td>5 m</td>
</tr>
</tbody>
</table></div>

<p>These render as 4 concentric circles on the satellite map with tooltips.</p>




<h2>
  
  
  🔫 Supported Systems
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>System</th>
<th>Round</th>
<th>v₀</th>
<th>Trajectory</th>
</tr>
</thead>
<tbody>
<tr>
<td>AHS KRAB (155mm)</td>
<td>M107 HE</td>
<td>560 m/s</td>
<td>Artillery arc</td>
</tr>
<tr>
<td>AHS KRAB (155mm)</td>
<td>EXCALIBUR</td>
<td>560 m/s</td>
<td>Artillery arc</td>
</tr>
<tr>
<td>M120 RAK (120mm)</td>
<td>HE STD</td>
<td>280 m/s</td>
<td>Artillery arc</td>
</tr>
<tr>
<td>M120 RAK (120mm)</td>
<td>SMOKE</td>
<td>250 m/s</td>
<td>Artillery arc</td>
</tr>
<tr>
<td>LEOPARD 2 (120mm)</td>
<td>APFSDS</td>
<td>1650 m/s</td>
<td>Flat</td>
</tr>
<tr>
<td>LEOPARD 2 (120mm)</td>
<td>HEAT</td>
<td>1140 m/s</td>
<td>Flat</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  ✨ Features
</h2>

<ul>
<li>🗺️ Satellite map with shooter/target positioning</li>
<li>🎯 Multi-target selection — mark multiple targets, fire sequentially</li>
<li>📊 Live results panel — azimuth, elevation angle, TOF, wind drift + Coriolis</li>
<li>💥 4-zone blast radius visualization</li>
<li>📄 PDF session report export</li>
<li>🔄 Dynamic system/ammo switching without restart</li>
<li>🔐 Session token authorization on all fire endpoints</li>
</ul>




<h2>
  
  
  🚀 Key Lessons Learned
</h2>

<p><strong>1. Euler simulation has limits</strong><br>
For high-velocity artillery (v0=827 m/s — my initial wrong value!), the vacuum range formula gives unrealistic results. Real M107 HE has v0=560 m/s, not 827 m/s.</p>

<p><strong>2. Bisection method for elevation angle</strong><br>
Finding the correct elevation angle for a given range required a bisection search. The tricky part: artillery prefers the <em>high-arc</em> solution (&gt;45°), not the flat one.</p>

<p><strong>3. Coriolis is smaller than you think</strong><br>
At 5km range with 20s flight time at 54°N: only ~3m deflection. Significant for precision rounds like EXCALIBUR (CEP 10m), negligible for M107 HE (CEP 150m).</p>

<p><strong>4. Redis Streams &gt; Redis String for queues</strong><br>
Using <code>XADD</code>/<code>XREAD</code> with consumer groups means no data loss even if the C# processor is temporarily down — messages wait in the stream.</p>




<h2>
  
  
  ⚠️ Disclaimer
</h2>

<blockquote>
<p>Educational and simulation purposes only. Real fire control systems use 6-DOF models, balloon sonde meteorological data, barrel wear corrections and dozens of other factors.</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1ymazy7mc8b6dpho9k8y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1ymazy7mc8b6dpho9k8y.png" alt=" " width="800" height="293"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feso5v7psjt3e5c799s60.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feso5v7psjt3e5c799s60.png" alt=" " width="800" height="387"></a></p>

<h2>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9uz0lyrwd81j6j0gs3uk.png" alt=" " width="800" height="387">
</h2>

<p><strong>Full source:</strong> <a href="https://github.com/InsaneInfinity/Balistic" rel="noopener noreferrer">https://github.com/InsaneInfinity/Balistic</a></p>

<p>Would love to hear your thoughts — especially on the physics model! 🎯</p>

