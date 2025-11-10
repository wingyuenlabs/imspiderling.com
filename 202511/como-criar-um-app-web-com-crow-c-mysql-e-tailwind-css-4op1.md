---
Title: Como Criar um App Web com Crow C++, MySQL e Tailwind CSS
Description: 
Author: Marcos Oliveira
Date: 2025-11-10T21:36:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>Assista série/playlist que possui 9 vídeos que mostra passo a passo como desenvolver uma aplicação web para gerenciamento de tarefas com <a href="https://terminalroot.com.br/tags#crow" rel="noopener noreferrer">Crow C++</a>, <a href="https://terminalroot.com.br/tags#mysql" rel="noopener noreferrer">MySQL</a> e <a href="https://terminalroot.com.br/tags#css" rel="noopener noreferrer">Tailwind CSS</a>.</p>

<h2>
  
  
  <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">📹 Playlist</a>
</h2>

<h3>
  
  
  <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">▶︎•၊၊||၊|။||||။၊|။•</a>
</h3>

<blockquote>
<p>ToDo++(<code>crow-todo</code>)</p>
</blockquote>




<h2>
  
  
  💻 Passo a passo para rodar na sua máquina ↓
</h2>

<blockquote>
<p>Windows e Unix-Like(macOS,GNU/Linux,*BSD, Haiku,...)</p>
</blockquote>




<h2>
  
  
  🪟 Windows:
</h2>

<h3>
  
  
  ▶️ Instale:
</h3>

<ol>
<li><a href="https://terminalroot.com.br/2025/06/como-instalar-o-crow-cpp-no-windows.html" rel="noopener noreferrer">Como Instalar o Crow e C++ no Windows</a></li>
<li><a href="https://terminalroot.com.br/2023/05/como-instalar-o-apache-php-8-e-mysql-no-windows.html#mysql" rel="noopener noreferrer">Instale o MySQL no Window</a></li>
<li><a href="https://terminalroot.com.br/2023/09/como-conectar-ao-mysql-com-c-cpp-no-windows.html" rel="noopener noreferrer">Como Conectar ao MySQL com C/C++ no Windows</a></li>
</ol>

<h3>
  
  
  ©️ Clone o projeto:
</h3>

<blockquote>
<p>Ou faça o download direto e depois descompacte.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/terroo/crow-todo
<span class="nb">cd </span>crow
</code></pre>

</div>



<h3>
  
  
  🎲 Faça o dump da base de dados:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mysql <span class="nt">-u</span> root <span class="nt">-p</span> &lt; cpp.sql
</code></pre>

</div>



<h3>
  
  
  🗄️ Altere suas credenciais para o MySQL:
</h3>

<blockquote>
<p>Abra o arquivo: <code>todo.cpp</code> e mude as linhas <strong>16</strong>(<code>[YOUR_USER]</code>) e <strong>17</strong>(<code>[YOUR_PASS]</code>):<br>
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
<p>Assumindo que as bibliotecas <code>asio</code> e <code>crow</code> estão no caminho: <code>C:\Includes</code>, modifique o caminho se você não seguiu o tutorial de instalação citado acima.</p>
</blockquote>

<h3>
  
  
  🏃 Rode:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>app.exe
</code></pre>

</div>



<h3>
  
  
  🌎 Acesse pelo navegador:
</h3>

<p><a href="http://localhost:18081/" rel="noopener noreferrer">http://localhost:18081/</a></p>




<h2>
  
  
  🦬 Unix-Like:
</h2>

<blockquote>
<p>macOS, GNU/Linux, *BSD, Haiku,...</p>
</blockquote>

<h3>
  
  
  ▶️ Instale:
</h3>

<ol>
<li><a href="https://terminalroot.com.br/2025/08/como-rodar-o-crow-cpp-com-apache.html" rel="noopener noreferrer">Como Instalar o Crow e C++ no Apache</a></li>
<li><a href="https://terminalroot.com.br/2023/03/como-instalar-mariadb-mysql-no-ubuntu-2210.html" rel="noopener noreferrer">Instale o MySQL</a></li>
<li><a href="https://terminalroot.com.br/2023/10/como-conectar-mysql-com-c-cpp-no-ubuntu-e-qualquer-distro.html" rel="noopener noreferrer">Instale o Conector C/C++</a></li>
</ol>

<h3>
  
  
  ©️ Clone o projeto:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/terroo/crow-todo
<span class="nb">cd </span>crow
</code></pre>

</div>



<h3>
  
  
  🎲 Faça o dump da base de dados:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mysql <span class="nt">-u</span> root <span class="nt">-p</span> &lt; cpp.sql
</code></pre>

</div>



<h3>
  
  
  🗄️ Altere suas credenciais para o MySQL:
</h3>

<blockquote>
<p>Abra o arquivo: <code>todo.cpp</code> e mude as linhas <strong>16</strong>(<code>[YOUR_USER]</code>) e <strong>17</strong>(<code>[YOUR_PASS]</code>):<br>
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
<p>Com <a href="https://terminalroot.com.br/tags#cmake" rel="noopener noreferrer">CMake</a>.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cmake <span class="nt">-B</span> build <span class="nb">.</span>
cmake <span class="nt">--build</span> build
</code></pre>

</div>



<h3>
  
  
  🏃 Rode:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>./build/app-crow-todo
</code></pre>

</div>



<h3>
  
  
  🌎 Acesse pelo navegador:
</h3>

<p><a href="http://localhost:18081/" rel="noopener noreferrer">http://localhost:18081/</a></p>

<p>Dados para logar no aplicativo:</p>

<ul>
<li>Login: <code>marcos@cpp.io</code>
</li>
<li>Senha: <code>teste123@</code>
</li>
</ul>

<p>Se quiser criar suas credenciais para logar, insira pelo MySQL, exemplo:</p>

<blockquote>
<p><code>mysql -u root -p -D cpp # Depois informe sua senha</code><br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">INSERT</span> <span class="k">INTO</span> <span class="nv">`login`</span> <span class="k">VALUES</span> <span class="p">(</span><span class="k">NULL</span><span class="p">,</span><span class="s1">'SEU NOME'</span><span class="p">,</span><span class="s1">'SOBRENOME'</span><span class="p">,</span><span class="s1">'SEU_USUARIO@EMAIL.COM'</span><span class="p">,</span> <span class="n">SHA2</span><span class="p">(</span><span class="s1">'[SUA_SENHA]'</span><span class="p">,</span> <span class="mi">256</span><span class="p">),</span><span class="s1">'admin'</span><span class="p">);</span>
</code></pre>

</div>



<blockquote>
<p>Altere os dados em maiúsculos por seus correspondentes dados.</p>
</blockquote>




<h2>
  
  
  📹 <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">Série de vídeos</a> que mostram como <code>crow-todo</code>/ToDo++ foi feito:
</h2>

<h3>
  
  
  <a href="https://www.youtube.com/watch?v=8rUIkUj7fjk&amp;list=PLUJBQEDDLNcmn_qxFhZYa02Y_gHDBXfly&amp;index=24" rel="noopener noreferrer">▶︎•၊၊||၊|။||||။၊|။•</a>
</h3>

