---
Title: Apex D. OpenClaw, Mejorar la Busqueda.
Description: 
Author: Victor Aguilar C.
Date: 2026-02-22T21:41:19.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Ampliar las capacidades de búsqueda de OpenClaw
</h1>

<p>Este apéndice cubre cómo ir más allá de Brave Search con proveedores de búsqueda alternativos: Exa.ai (custom skill) y Perplexity (opción built-in).</p>

<h2>
  
  
  Por qué Exa.ai no es un simple “swap”
</h2>

<p>OpenClaw trae web search built-in impulsado por Brave. Podrías preguntarte: “¿Puedo solo cambiar un setting para usar Exa.ai en su lugar?” La respuesta corta es no, pero por buenas razones.</p>

<p>La configuración de OpenClaw solo soporta dos search providers built-in:</p>

<ul>
<li>Brave Search (default)
</li>
<li>Perplexity</li>
</ul>

<p>Estas son integraciones “native”, o sea, OpenClaw sabe cómo hablar con ellas out of the box.</p>

<p><strong>Exa.ai es diferente</strong>. Es un neural/semantic search engine que requiere su propio formato de API, método de autenticación y manejo de respuestas. En vez de esperar a que OpenClaw agregue soporte nativo, podemos agregarlo nosotros usando una skill. Piensa en las skills como “plugins” o “extensions” para tu AI assistant. Una skill es un folder con instrucciones sobre cuándo usarla y scripts que hacen el trabajo real. Las skills te permiten extender las capacidades de OpenClaw sin modificar su core code.</p>

<h2>
  
  
  Brave vs. Exa: cuándo usar cada uno
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb6phrihttklkelepg0q9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb6phrihttklkelepg0q9.png" alt=" " width="800" height="98"></a></p>

<ul>
<li><p><strong>Usa Brave cuando:</strong> necesitas una respuesta rápida, verificar facts, o buscar websites específicos.</p></li>
<li><p><strong>Usa Exa cuando:</strong> estás investigando un tema, necesitas fuentes de alta calidad, o quieres resultados semánticamente relevantes (no solo keyword matches).</p></li>
</ul>




<h2>
  
  
  Configurar la skill de Exa.ai
</h2>

<p><strong>Prerequisites:</strong> una instalación funcional de OpenClaw, una Exa.ai API key (consigue una en <a href="https://exa.ai" rel="noopener noreferrer">https://exa.ai</a>), y comodidad básica con la command line.</p>

<h3>
  
  
  Step 1: Crear el folder de la skill
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir</span> <span class="nt">-p</span> ~/.openclaw/workspace/skills/exa-search/scripts
</code></pre>

</div>



<h3>
  
  
  Step 2: Crear la definición de la skill
</h3>

<p>Crea un archivo llamado <code>SKILL.md</code> dentro del folder de la skill:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nano ~/.openclaw/workspace/skills/exa-search/SKILL.md
</code></pre>

</div>



<p>Pega el siguiente contenido:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="nn">---</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">exa-search</span>
<span class="na">description</span><span class="pi">:</span> <span class="s">Neural/semantic web search using Exa.ai API. Use for research queries requiring high-quality, semantically relevant results. Better than keyword search for conceptual queries, finding similar content, or research-heavy tasks. Requires EXA_API_KEY environment variable.</span>
<span class="nn">---</span>

<span class="c1"># Exa Search</span>

<span class="s">Neural search via Exa.ai. Returns semantically relevant results, not just keyword matches.</span>

<span class="c1">## When to Use</span>

<span class="pi">-</span> <span class="s">Research queries needing deep relevance (not just keywords)</span>
<span class="pi">-</span> <span class="s">Finding articles/papers on specific concepts</span>
<span class="pi">-</span> <span class="s">Semantic similarity searches</span>
<span class="pi">-</span> <span class="s">When Brave search results aren't cutting it</span>

<span class="c1">## Usage</span>

<span class="s">python3 scripts/exa_search.py "your search query"</span>
<span class="s">python3 scripts/exa_search.py "your query" --num-results 10 --summary</span>

<span class="c1">## Options</span>

<span class="pi">|</span> <span class="err">Flag</span> <span class="err">|</span> <span class="err">Description</span> <span class="err">|</span> <span class="err">Default</span> <span class="err">|</span>
<span class="err">|</span><span class="s">------|-------------|---------|</span>
<span class="err">|</span><span class="s"> --num-results N | Number of results | 5 |</span>
<span class="err">|</span><span class="s"> --text | Include page text snippets | off |</span>
<span class="err">|</span><span class="s"> --summary | Include AI summaries | off |</span>
<span class="err">|</span><span class="s"> --type TYPE | Search type: auto, neural, keyword | auto |</span>
</code></pre>

</div>



<p>Guarda y cierra el archivo (en nano: Ctrl+X, luego Y, luego Enter).</p>

<h3>
  
  
  Step 3: Crear el script de búsqueda
</h3>

<p>Este es el script en Python que llama al API de Exa:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nano ~/.openclaw/workspace/skills/exa-search/scripts/exa_search.py
</code></pre>

</div>



<p>Pega este código:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1">#!/usr/bin/env python3
</span><span class="sh">"""</span><span class="s">Exa.ai neural search CLI.</span><span class="sh">"""</span>

<span class="kn">import</span> <span class="n">argparse</span>
<span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">os</span>
<span class="kn">import</span> <span class="n">urllib.request</span>
<span class="kn">import</span> <span class="n">urllib.error</span>
<span class="kn">from</span> <span class="n">pathlib</span> <span class="kn">import</span> <span class="n">Path</span>

<span class="n">API_URL</span> <span class="o">=</span> <span class="sh">"</span><span class="s">https://api.exa.ai/search</span><span class="sh">"</span>

<span class="c1"># Load .env file if present
</span><span class="k">def</span> <span class="nf">load_env</span><span class="p">():</span>
    <span class="n">env_path</span> <span class="o">=</span> <span class="nc">Path</span><span class="p">(</span><span class="n">__file__</span><span class="p">).</span><span class="n">parent</span><span class="p">.</span><span class="n">parent</span> <span class="o">/</span> <span class="sh">"</span><span class="s">.env</span><span class="sh">"</span>
    <span class="k">if</span> <span class="n">env_path</span><span class="p">.</span><span class="nf">exists</span><span class="p">():</span>
        <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">env_path</span><span class="p">.</span><span class="nf">read_text</span><span class="p">().</span><span class="nf">strip</span><span class="p">().</span><span class="nf">split</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">):</span>
            <span class="k">if</span> <span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">line</span> <span class="ow">and</span> <span class="ow">not</span> <span class="n">line</span><span class="p">.</span><span class="nf">startswith</span><span class="p">(</span><span class="sh">"</span><span class="s">#</span><span class="sh">"</span><span class="p">):</span>
                <span class="n">key</span><span class="p">,</span> <span class="n">val</span> <span class="o">=</span> <span class="n">line</span><span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
                <span class="k">if</span> <span class="n">key</span><span class="p">.</span><span class="nf">strip</span><span class="p">()</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">:</span>
                    <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">[</span><span class="n">key</span><span class="p">.</span><span class="nf">strip</span><span class="p">()]</span> <span class="o">=</span> <span class="n">val</span><span class="p">.</span><span class="nf">strip</span><span class="p">()</span>

<span class="nf">load_env</span><span class="p">()</span>


<span class="k">def</span> <span class="nf">search</span><span class="p">(</span><span class="n">query</span><span class="p">,</span> <span class="n">num_results</span><span class="o">=</span><span class="mi">5</span><span class="p">,</span> <span class="n">include_text</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span> <span class="n">include_summary</span><span class="o">=</span><span class="bp">False</span><span class="p">,</span> <span class="n">search_type</span><span class="o">=</span><span class="sh">"</span><span class="s">auto</span><span class="sh">"</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Execute Exa search.</span><span class="sh">"""</span>
    <span class="n">api_key</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">EXA_API_KEY</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">api_key</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">EXA_API_KEY not set. Add it to .env file or environment.</span><span class="sh">"</span><span class="p">}</span>

    <span class="n">payload</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">query</span><span class="sh">"</span><span class="p">:</span> <span class="n">query</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">numResults</span><span class="sh">"</span><span class="p">:</span> <span class="n">num_results</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="p">:</span> <span class="n">search_type</span><span class="p">,</span>
    <span class="p">}</span>

    <span class="n">contents</span> <span class="o">=</span> <span class="p">{}</span>
    <span class="k">if</span> <span class="n">include_text</span><span class="p">:</span>
        <span class="n">contents</span><span class="p">[</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="bp">True</span>
    <span class="k">if</span> <span class="n">include_summary</span><span class="p">:</span>
        <span class="n">contents</span><span class="p">[</span><span class="sh">"</span><span class="s">summary</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="bp">True</span>
    <span class="k">if</span> <span class="n">contents</span><span class="p">:</span>
        <span class="n">payload</span><span class="p">[</span><span class="sh">"</span><span class="s">contents</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">contents</span>

    <span class="n">headers</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">x-api-key</span><span class="sh">"</span><span class="p">:</span> <span class="n">api_key</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">Content-Type</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">application/json</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">User-Agent</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">OpenClaw/1.0</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">Accept</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">application/json</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">}</span>

    <span class="n">req</span> <span class="o">=</span> <span class="n">urllib</span><span class="p">.</span><span class="n">request</span><span class="p">.</span><span class="nc">Request</span><span class="p">(</span>
        <span class="n">API_URL</span><span class="p">,</span>
        <span class="n">data</span><span class="o">=</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">payload</span><span class="p">).</span><span class="nf">encode</span><span class="p">(</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">headers</span><span class="o">=</span><span class="n">headers</span><span class="p">,</span>
        <span class="n">method</span><span class="o">=</span><span class="sh">"</span><span class="s">POST</span><span class="sh">"</span>
    <span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="k">with</span> <span class="n">urllib</span><span class="p">.</span><span class="n">request</span><span class="p">.</span><span class="nf">urlopen</span><span class="p">(</span><span class="n">req</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">30</span><span class="p">)</span> <span class="k">as</span> <span class="n">resp</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">resp</span><span class="p">.</span><span class="nf">read</span><span class="p">().</span><span class="nf">decode</span><span class="p">(</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">))</span>
    <span class="k">except</span> <span class="n">urllib</span><span class="p">.</span><span class="n">error</span><span class="p">.</span><span class="n">HTTPError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">HTTP </span><span class="si">{</span><span class="n">e</span><span class="p">.</span><span class="n">code</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="n">e</span><span class="p">.</span><span class="nf">read</span><span class="p">().</span><span class="nf">decode</span><span class="p">(</span><span class="sh">'</span><span class="s">utf-8</span><span class="sh">'</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">replace</span><span class="sh">'</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>
    <span class="k">except</span> <span class="n">urllib</span><span class="p">.</span><span class="n">error</span><span class="p">.</span><span class="n">URLError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Request failed: </span><span class="si">{</span><span class="n">e</span><span class="p">.</span><span class="n">reason</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>


<span class="k">def</span> <span class="nf">format_results</span><span class="p">(</span><span class="n">data</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Format results for display.</span><span class="sh">"""</span>
    <span class="k">if</span> <span class="sh">"</span><span class="s">error</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">data</span><span class="p">:</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Error: </span><span class="si">{</span><span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">error</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span>

    <span class="n">results</span> <span class="o">=</span> <span class="n">data</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">results</span><span class="sh">"</span><span class="p">,</span> <span class="p">[])</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">results</span><span class="p">:</span>
        <span class="k">return</span> <span class="sh">"</span><span class="s">No results found.</span><span class="sh">"</span>

    <span class="n">lines</span> <span class="o">=</span> <span class="p">[</span><span class="sa">f</span><span class="sh">"</span><span class="s">Found </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">results</span><span class="p">)</span><span class="si">}</span><span class="s"> results:</span><span class="se">\n</span><span class="sh">"</span><span class="p">]</span>

    <span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="n">r</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="n">results</span><span class="p">,</span> <span class="mi">1</span><span class="p">):</span>
        <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">## </span><span class="si">{</span><span class="n">i</span><span class="si">}</span><span class="s">. </span><span class="si">{</span><span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Untitled</span><span class="sh">'</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">URL: </span><span class="si">{</span><span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">url</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">N/A</span><span class="sh">'</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">if</span> <span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">publishedDate</span><span class="sh">"</span><span class="p">):</span>
            <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Published: </span><span class="si">{</span><span class="n">r</span><span class="p">[</span><span class="sh">'</span><span class="s">publishedDate</span><span class="sh">'</span><span class="p">][</span><span class="si">:</span><span class="mi">10</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">author</span><span class="sh">"</span><span class="p">):</span>
            <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Author: </span><span class="si">{</span><span class="n">r</span><span class="p">[</span><span class="sh">'</span><span class="s">author</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">summary</span><span class="sh">"</span><span class="p">):</span>
            <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Summary: </span><span class="si">{</span><span class="n">r</span><span class="p">[</span><span class="sh">'</span><span class="s">summary</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">):</span>
            <span class="n">text</span> <span class="o">=</span> <span class="n">r</span><span class="p">[</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">][:</span><span class="mi">500</span><span class="p">]</span> <span class="o">+</span> <span class="sh">"</span><span class="s">...</span><span class="sh">"</span> <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">))</span> <span class="o">&gt;</span> <span class="mi">500</span> <span class="k">else</span> <span class="n">r</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>
            <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Text: </span><span class="si">{</span><span class="n">text</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sh">""</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">data</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">costDollars</span><span class="sh">"</span><span class="p">):</span>
        <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Cost: $</span><span class="si">{</span><span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">costDollars</span><span class="sh">'</span><span class="p">].</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">total</span><span class="sh">'</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span><span class="si">:</span><span class="p">.</span><span class="mi">4</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">return</span> <span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">lines</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">parser</span> <span class="o">=</span> <span class="n">argparse</span><span class="p">.</span><span class="nc">ArgumentParser</span><span class="p">(</span><span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">Exa.ai neural search</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">parser</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="sh">"</span><span class="s">query</span><span class="sh">"</span><span class="p">,</span> <span class="n">help</span><span class="o">=</span><span class="sh">"</span><span class="s">Search query</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">parser</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="sh">"</span><span class="s">--num-results</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">-n</span><span class="sh">"</span><span class="p">,</span> <span class="nb">type</span><span class="o">=</span><span class="nb">int</span><span class="p">,</span> <span class="n">default</span><span class="o">=</span><span class="mi">5</span><span class="p">)</span>
    <span class="n">parser</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="sh">"</span><span class="s">--text</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">-t</span><span class="sh">"</span><span class="p">,</span> <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">store_true</span><span class="sh">"</span><span class="p">,</span> <span class="n">help</span><span class="o">=</span><span class="sh">"</span><span class="s">Include text snippets</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">parser</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="sh">"</span><span class="s">--summary</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">-s</span><span class="sh">"</span><span class="p">,</span> <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">store_true</span><span class="sh">"</span><span class="p">,</span> <span class="n">help</span><span class="o">=</span><span class="sh">"</span><span class="s">Include AI summaries</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">parser</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="sh">"</span><span class="s">--type</span><span class="sh">"</span><span class="p">,</span> <span class="n">choices</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">auto</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">neural</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">keyword</span><span class="sh">"</span><span class="p">],</span> <span class="n">default</span><span class="o">=</span><span class="sh">"</span><span class="s">auto</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">parser</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="sh">"</span><span class="s">--json</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">-j</span><span class="sh">"</span><span class="p">,</span> <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">store_true</span><span class="sh">"</span><span class="p">,</span> <span class="n">help</span><span class="o">=</span><span class="sh">"</span><span class="s">Output raw JSON</span><span class="sh">"</span><span class="p">)</span>

    <span class="n">args</span> <span class="o">=</span> <span class="n">parser</span><span class="p">.</span><span class="nf">parse_args</span><span class="p">()</span>

    <span class="n">result</span> <span class="o">=</span> <span class="nf">search</span><span class="p">(</span>
        <span class="n">query</span><span class="o">=</span><span class="n">args</span><span class="p">.</span><span class="n">query</span><span class="p">,</span>
        <span class="n">num_results</span><span class="o">=</span><span class="n">args</span><span class="p">.</span><span class="n">num_results</span><span class="p">,</span>
        <span class="n">include_text</span><span class="o">=</span><span class="n">args</span><span class="p">.</span><span class="n">text</span><span class="p">,</span>
        <span class="n">include_summary</span><span class="o">=</span><span class="n">args</span><span class="p">.</span><span class="n">summary</span><span class="p">,</span>
        <span class="n">search_type</span><span class="o">=</span><span class="n">args</span><span class="p">.</span><span class="nb">type</span><span class="p">,</span>
    <span class="p">)</span>

    <span class="nf">print</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">result</span><span class="p">,</span> <span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span> <span class="k">if</span> <span class="n">args</span><span class="p">.</span><span class="n">json</span> <span class="k">else</span> <span class="nf">format_results</span><span class="p">(</span><span class="n">result</span><span class="p">))</span>


<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>
</code></pre>

</div>



<p>Guarda y cierra el archivo.</p>

<h3>
  
  
  Step 4: Hacer el script ejecutable
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">chmod</span> +x ~/.openclaw/workspace/skills/exa-search/scripts/exa_search.py
</code></pre>

</div>



<h3>
  
  
  Step 5: Agregar tu API key
</h3>

<p>Crea un archivo <code>.env</code> para guardar tu API key de forma segura:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nano ~/.openclaw/workspace/skills/exa-search/.env
</code></pre>

</div>



<p>Agrega tu key (reemplaza por tu key real):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">EXA_API_KEY</span><span class="o">=</span>your-api-key-here
</code></pre>

</div>



<p>Guarda y cierra.</p>

<blockquote>
<p><strong>Security Note</strong>: El archivo .env mantiene tu API key fuera del script. Nunca hagas commit de API keys en version control ni las compartas públicamente.</p>
</blockquote>

<h3>
  
  
  Step 6: Probar la skill
</h3>

<p>Corre una búsqueda de prueba:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd ~/.openclaw/workspace/skills/exa-search
python3 scripts/exa_search.py "artificial intelligence in healthcare"
</code></pre>

</div>



<p>Deberías ver resultados formateados. Prueba con summaries:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python3 scripts/exa_search.py <span class="s2">"artificial intelligence in healthcare"</span> <span class="nt">--summary</span> <span class="nt">--num-results</span> 3
</code></pre>

</div>



<h2>
  
  
  Cómo usa tu IA la skill de Exa
</h2>

<p>Una vez que la skill está configurada, tu OpenClaw assistant va a:</p>

<ol>
<li>
<strong>Detectar automáticamente</strong> cuándo usarla basándose en la descripción de la skill</li>
<li>
<strong>Elegir Exa sobre Brave</strong> para consultas research-heavy o conceptuales</li>
<li>
<strong>Hacer fall back a Brave para lookups</strong> rápidos donde la velocidad importa</li>
</ol>

<p>También puedes pedir explícitamente: “use Exa search for…” si quieres asegurarte de que use el neural search.</p>

<h2>
  
  
  Exa Skill File Summary
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9lp842b1i2f6tfie9va4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9lp842b1i2f6tfie9va4.png" alt=" " width="800" height="215"></a></p>




<h3>
  
  
  Exa Cost Considerations
</h3>

<p>Exa.ai cobra por búsqueda:</p>

<ul>
<li>Basic search: ~$0.005 por query</li>
<li>Con summaries: ~$0.01 por query</li>
<li>Deep search: ~$0.015 por query</li>
</ul>

<p>Para la mayoría de usuarios que hacen unas pocas research queries al día, esto cuesta centavos. Un uso intenso podría estar en $1–5/mes. Compáralo con el valor: una búsqueda de alta calidad en Exa puede reemplazar 10+ búsquedas en Brave cuando estás tratando de entender un tema complejo.</p>




<h2>
  
  
  Cambiar a Perplexity (alternativa built-in)
</h2>

<p>A diferencia de Exa, Perplexity es un provider built-in en OpenClaw. No requiere skill: solo un cambio de config.</p>

<p>Perplexity es un AI-powered search engine que devuelve respuestas sintetizadas (no solo links), incluye citations a fuentes, y maneja follow-up questions de forma conversacional. Piensa en esto como “ChatGPT que busca en la web”, no como un search engine tradicional.</p>

<ul>
<li><p><strong>Elige Perplexity cuando</strong>: quieres una respuesta directa con fuentes, no una lista de links para leer.</p></li>
<li><p><strong>Elige Exa cuando</strong>: necesitas source documents de alta calidad para research, no summaries pre-digeridos.</p></li>
</ul>

<h2>
  
  
  Step 1: Conseguir un API key
</h2>

<p>Ve a <a href="https://www.perplexity.ai/settings/api" rel="noopener noreferrer">https://www.perplexity.ai/settings/api</a>, crea una cuenta o sign in, y genera un API key (empieza con pplx-).</p>

<h2>
  
  
  Step 2: Actualizar OpenClaw config
</h2>

<p>Abre tu archivo de configuración de OpenClaw:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>nano ~/.openclaw/openclaw.json
</code></pre>

</div>



<p>Encuentra la sección tools.web.search y actualízala:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="nl">"tools"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="nl">"web"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"search"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"enabled"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
      </span><span class="nl">"provider"</span><span class="p">:</span><span class="w"> </span><span class="s2">"perplexity"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"perplexity"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"apiKey"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pplx-your-api-key-here"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"model"</span><span class="p">:</span><span class="w"> </span><span class="s2">"sonar-pro"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Guarda el archivo.</p>

<blockquote>
<p><strong>Important</strong>: Debes especificar "model": "sonar-pro" cuando uses un Perplexity API key directo (pplx-). El model name default de OpenClaw (perplexity/sonar-pro) está formateado para OpenRouter, no para el Perplexity API directo. Sin este fix, obtendrás un error 400 “Invalid model”.</p>
</blockquote>

<h2>
  
  
  Step 3: Reiniciar OpenClaw
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>openclaw gateway restart
</code></pre>

</div>



<p>Listo. Tu assistant ahora usará Perplexity para web searches en lugar de Brave.</p>

<p>Volver a Brave: para revertir, cambia el provider de vuelta a <strong>"brave"</strong> en la misma sección y actualiza el API key al de Brave Search.</p>




<p>Usar múltiples search providers</p>

<p>OpenClaw solo** soporta un built-in search provider** a la vez. Sin embargo, puedes combinar un built-in provider con la Exa skill:</p>

<ol>
<li>Usar <strong>Perplexity</strong> como built-in provider (para respuestas rápidas AI-summarized)</li>
<li>Agregar la <strong>Exa skill</strong> (para deep research cuando haga falta)</li>
</ol>

<p>Esto te da lo mejor de ambos mundos sin estar cambiando configs todo el tiempo. Tu agent usará Perplexity para consultas generales y automáticamente recurrirá a Exa cuando necesite resultados más profundos y semánticamente relevantes.</p>




<h2>
  
  
  Exa y Perplexity Troubleshooting
</h2>

<ul>
<li><p><strong><em>“EXA_API_KEY not set.”</em></strong> Asegúrate de que tu <code>.env</code> esté en <code>~/.openclaw/workspace/skills/exa-search/.env</code>, que tenga tu key como <code>EXA_API_KEY=your-key-here</code>, y que no haya espacios extra alrededor del <code>=.</code></p></li>
<li><p><strong><em>“HTTP 403” o “error code: 1010” desde Exa</em></strong>. Es un error de protección de Cloudflare. El script incluye headers para evitarlo, pero si aún aparece, espera unos segundos e intenta de nuevo, y revisa que tu API key sea válida en <a href="https://exa.ai" rel="noopener noreferrer">https://exa.ai</a>.</p></li>
<li><p>No devuelve resultados Exa. Prueba una consulta más amplia, revisa que tu cuenta Exa.ai tenga créditos, o usa <code>--type</code> neural para forzar neural search mode.</p></li>
<li><p><strong><em>Perplexity “Invalid model” error (400)</em></strong>. Si ves algo como Invalid model 'perplexity/sonar-pro'... significa que OpenClaw está usando el formato de OpenRouter, pero tu pplx- API key va al Perplexity API directo. Arréglalo seteando el model explícitamente a <code>"sonar-pro"</code> (no "perplexity/sonar-pro"). Ver los pasos de setup de Perplexity arriba.</p></li>
</ul>

