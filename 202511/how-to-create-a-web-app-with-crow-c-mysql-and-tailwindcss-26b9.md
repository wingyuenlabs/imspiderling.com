---
Title: How to Create a Web App with Crow C++, MySQL, and TailwindCSS
Description: 
Author: Marcos Oliveira
Date: 2025-11-10T21:51:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>Watch the series/playlist containing 9 videos that show step-by-step how to develop a web application for task management with <a href="https://terminalroot.com/tags#crow" rel="noopener noreferrer">Crow C++</a>, <a href="https://terminalroot.com/tags#mysql" rel="noopener noreferrer">MySQL</a>, and <a href="https://terminalroot.com/tags#css" rel="noopener noreferrer">Tailwind CSS</a>.</p>

<h2>
  
  
  <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">📹 Playlist</a>
</h2>

<h3>
  
  
  <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">▶︎•၊၊||၊|။||||။၊|။•</a>
</h3>

<blockquote>
<p>The videos are in Portuguese; use YouTube's automatic translation (Audio Track) to find your language.</p>
</blockquote>




<h2>
  
  
  💻 Step-by-step to run on your machine ↓
</h2>

<blockquote>
<p>Windows and Unix-Like (macOS, GNU/Linux, *BSD, Haiku,...)</p>
</blockquote>




<h2>
  
  
  🪟 Windows:
</h2>

<h3>
  
  
  ▶️ Install:
</h3>

<ol>
<li><a href="https://terminalroot.com/how-to-install-crow-cpp-on-windows/" rel="noopener noreferrer">How to Install Crow and C++ on Windows</a></li>
<li><a href="https://terminalroot.com.br/2023/05/como-instalar-o-apache-php-8-e-mysql-no-windows.html#mysql" rel="noopener noreferrer">Install MySQL on Windows</a></li>
<li><a href="https://terminalroot.com.br/2023/09/como-conectar-ao-mysql-com-c-cpp-no-windows.html" rel="noopener noreferrer">How to Connect to MySQL with C/C++ on Windows</a></li>
</ol>

<h3>
  
  
  ©️ Clone the project:
</h3>

<blockquote>
<p>Or download it directly and then unzip it.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/terroo/crow-todo
<span class="nb">cd </span>crow
</code></pre>

</div>



<h3>
  
  
  🎲 Dump the database:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mysql <span class="nt">-u</span> root <span class="nt">-p</span> &lt; cpp.sql
</code></pre>

</div>



<h3>
  
  
  🗄️ Change your MySQL credentials:
</h3>

<blockquote>
<p>Open the file: <code>todo.cpp</code> and change lines <strong>16</strong> (<code>[YOUR_USER]</code>) and <strong>17</strong> (<code>[YOUR_PASS]</code>):<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ...</span>
  <span class="n">connect</span> <span class="o">=</span> <span class="n">mysql_real_connect</span><span class="p">(</span>
    <span class="n">connect</span><span class="p">,</span>
    <span class="s">"localhost"</span><span class="p">,</span>
    <span class="s">"[YOUR_USER]"</span><span class="p">,</span>
    <span class="s">"[YOUR_PASS]"</span><span class="p">,</span>
    <span class="s">"cpp"</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="mi">0</span>
  <span class="p">);</span>
<span class="c1">// ...</span>
</code></pre>

</div>



<h3>
  
  
  🏗️ Compile:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>g++ main.cpp <span class="nt">-I</span><span class="s2">"C:/Includes/asio/include"</span> <span class="nt">-I</span><span class="s2">"C:/Includes/crow/include"</span> <span class="nt">-lws2_32</span> <span class="nt">-lmswsock</span> <span class="nt">-o</span> app.exe
</code></pre>

</div>



<blockquote>
<p>Assuming the <code>asio</code> and <code>crow</code> libraries are in the path: <code>C:\Includes</code>, modify the path if you did not follow the installation tutorial mentioned above.</p>
</blockquote>

<h3>
  
  
  🏃 Run:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>app.exe
</code></pre>

</div>



<h3>
  
  
  🌎 Access via browser:
</h3>

<p><a href="http://localhost:18081/" rel="noopener noreferrer">http://localhost:18081/</a></p>




<h2>
  
  
  🦬 Unix-Like:
</h2>

<blockquote>
<p>macOS, GNU/Linux, *BSD, Haiku,...</p>
</blockquote>

<h3>
  
  
  ▶️ Install:
</h3>

<ol>
<li><a href="https://terminalroot.com/how-to-run-crow-cpp-with-apache/" rel="noopener noreferrer">How to Install Crow and C++ on Apache</a></li>
<li><a href="https://terminalroot.com.br/2023/03/como-instalar-mariadb-mysql-no-ubuntu-2210.html" rel="noopener noreferrer">Install MySQL</a></li>
<li><a href="https://terminalroot.com.br/2023/10/como-conectar-mysql-com-c-cpp-no-ubuntu-e-qualquer-distro.html" rel="noopener noreferrer">Install the C/C++ Connector</a></li>
</ol>

<h3>
  
  
  ©️ Clone the project:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/terroo/crow-todo
<span class="nb">cd </span>crow
</code></pre>

</div>



<h3>
  
  
  🎲 Dump the database:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mysql <span class="nt">-u</span> root <span class="nt">-p</span> &lt; cpp.sql
</code></pre>

</div>



<h3>
  
  
  🗄️ Change your MySQL credentials:
</h3>

<blockquote>
<p>Open the file: <code>todo.cpp</code> and change lines <strong>16</strong> (<code>[YOUR_USER]</code>) and <strong>17</strong> (<code>[YOUR_PASS]</code>):<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// ...</span>
  <span class="n">connect</span> <span class="o">=</span> <span class="n">mysql_real_connect</span><span class="p">(</span>
    <span class="n">connect</span><span class="p">,</span>
    <span class="s">"localhost"</span><span class="p">,</span>
    <span class="s">"[YOUR_USER]"</span><span class="p">,</span>
    <span class="s">"[YOUR_PASS]"</span><span class="p">,</span>
    <span class="s">"cpp"</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="mi">0</span>
  <span class="p">);</span>
<span class="c1">// ...</span>
</code></pre>

</div>



<h3>
  
  
  🏗️ Compile:
</h3>

<blockquote>
<p>With <a href="https://terminalroot.com.br/tags#cmake" rel="noopener noreferrer">CMake</a>.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cmake <span class="nt">-B</span> build <span class="nb">.</span>
cmake <span class="nt">--build</span> build
</code></pre>

</div>



<h3>
  
  
  🏃 Run:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./build/app-crow-todo
</code></pre>

</div>



<h3>
  
  
  🌎 Access via browser:
</h3>

<p><a href="http://localhost:18081/" rel="noopener noreferrer">http://localhost:18081/</a></p>

<p>Credentials to log into the application:</p>

<ul>
<li>Login: <code>marcos@cpp.io</code>
</li>
<li>Password: <code>teste123@</code>
</li>
</ul>

<p>If you want to create your own credentials to log in, insert them via MySQL, for example:</p>

<blockquote>
<p><code>mysql -u root -p -D cpp # Then enter your password</code><br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="nv">`login`</span> <span class="k">VALUES</span> <span class="p">(</span><span class="k">NULL</span><span class="p">,</span><span class="s1">'YOUR NAME'</span><span class="p">,</span><span class="s1">'LAST NAME'</span><span class="p">,</span><span class="s1">'YOUR_USER@EMAIL.COM'</span><span class="p">,</span> <span class="n">SHA2</span><span class="p">(</span><span class="s1">'[YOUR_PASSWORD]'</span><span class="p">,</span> <span class="mi">256</span><span class="p">),</span><span class="s1">'admin'</span><span class="p">);</span>
</code></pre>

</div>



<blockquote>
<p>Replace the uppercase data with your corresponding details.</p>
</blockquote>




<h2>
  
  
  📹 <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">Video Series</a> showing how <code>crow-todo</code>/ToDo++ was made:
</h2>

<h3>
  
  
  <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">▶︎•၊၊||၊|။||||။၊|။•</a>
</h3>

<blockquote>
<p>The videos are in Portuguese; use YouTube's automatic translation (Audio Track) to find your language.</p>
</blockquote>

