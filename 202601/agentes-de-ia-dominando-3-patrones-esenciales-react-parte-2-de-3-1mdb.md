---
Title: Agentes de IA: Dominando 3 Patrones Esenciales (ReAct). Parte 2 de 3
Description: 
Author: Gabriel Melendez
Date: 2026-01-04T21:44:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>Artículo [<a href="https://dev.to/gabrielmrojas/agentes-de-ia-dominando-3-patrones-esenciales-tool-using-parte-1-de-3-244l">Parte 1</a>]</p>

<p>El código de estos patrones está disponible en Github. [<a href="https://github.com/gabrielmrojas/1-AI-Agent-Fundamentals-Patterns" rel="noopener noreferrer">Repo</a>]</p>

<h3>
  
  
  El Patrón ReAct (Reason + Act) – Cuando el Agente empieza a "hablar solo"
</h3>

<p>Si con el patrón "Tool Using" (Nivel 1) le dimos "manos" a la IA para que pudiera tocar el mundo exterior, con ReAct le estamos instalando un "cerebro funcional".</p>

<p>Hagamos memoria: en el Nivel 1 todo era muy lineal. Estímulo → Respuesta. Tú pedías un dato, el agente disparaba una herramienta y listo. Sin dudas ni planes. Pero seamos realistas, el mundo real es desordenado. ¿Qué pasa cuando la pregunta no se resuelve con un solo clic? ¿Qué pasa cuando hay que investigar, comparar datos y luego hacer un cálculo?</p>

<p>Ahí es donde el modelo lineal se estrella y entra en juego ReAct (propuesto por gente muy lista de Princeton y Google en 2022), que hoy en día es básicamente el estándar de la industria.</p>

<h3>
  
  
  ¿Qué es exactamente ReAct?
</h3>

<p>El nombre viene de Reasoning + Acting (Razonar + Actuar).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgic8kfua8ha4fg9wmkri.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgic8kfua8ha4fg9wmkri.jpg" alt="Image 1" width="800" height="436"></a></p>

<p>La idea rompedora aquí no es que use herramientas (eso ya lo teníamos), sino que le obligamos al LLM a tener un Monólogo Interno ("Thought Trace"). En lugar de lanzarse a responder a lo loco, el agente entra en un bucle cognitivo:</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Pensar → Actuar → Observar → Repetir
</code></pre>

</div>

<p>El agente literalmente "habla consigo mismo" en los logs. Planifica su siguiente paso basándose en lo que acaba de descubrir hace un segundo. Esto le permite corregir el rumbo sobre la marcha si las cosas no salen como esperaba.<br>
Destripando el Bucle: El caso del "Detective Histórico"</p>

<p>Vamos a ver qué pasa bajo el capó con el ejemplo que acabamos de montar. El usuario pregunta: "¿Cuál es la raíz cuadrada de la edad del presidente actual de Francia?"</p>

<p>Si le preguntas esto a un modelo "a pelo", alucinará o te dirá que no sabe la fecha de hoy. Pero fíjate cómo lo gestiona un agente ReAct paso a paso:</p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Iteración 1:

    Thought (Lo que piensa): "Vale, no tengo ni idea de la edad del presidente. De hecho, ni siquiera sé quién es el presidente ahora mismo. Primero busquemos eso".

    Action (Lo que hace): search_tool("Presidente de Francia actual")

    Observation (Lo que ve): "Emmanuel Macron."

Iteración 2:

    Thought: "Ok, ya tenemos el nombre: Macron. Ahora necesito su edad."

    Action: search_tool("Emmanuel Macron edad")

    Observation: "47 años."

Iteración 3:

    Thought: "Tengo el 47. El usuario quería la raíz cuadrada de esto."

    Action: calculator_tool("sqrt(47)")

    Observation: "6.8556..."

Respuesta Final: "La raíz cuadrada de la edad de Emmanuel Macron (47) es aproximadamente 6.85."
</code></pre>

</div>
<p>¿Ves la magia? El paso 3 era imposible sin haber descubierto el paso 2 primero. El agente ha ido "atando cabos".</p>
<h3>
  
  
  Ahora vamos al codigo
</h3>

<p>Usaremos Agno, para crear nuestro ejemplo del patron ReAct, implementando Pensar → Actuar → Observar → Repetir<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">os</span>
<span class="kn">import</span> <span class="n">sys</span>
<span class="kn">import</span> <span class="n">logging</span>
<span class="kn">import</span> <span class="n">traceback</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">List</span><span class="p">,</span> <span class="n">Optional</span>
<span class="kn">from</span> <span class="n">dotenv</span> <span class="kn">import</span> <span class="n">load_dotenv</span><span class="p">,</span> <span class="n">find_dotenv</span>
<span class="kn">from</span> <span class="n">agno.agent</span> <span class="kn">import</span> <span class="n">Agent</span>
<span class="kn">from</span> <span class="n">agno.models.openai</span> <span class="kn">import</span> <span class="n">OpenAIChat</span>
<span class="kn">from</span> <span class="n">agno.tools.tavily</span> <span class="kn">import</span> <span class="n">TavilyTools</span>

<span class="c1"># 1. Global Logging and Error Handling Configuration
</span><span class="n">LOG_DIR</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">dirname</span><span class="p">(</span><span class="n">__file__</span><span class="p">),</span> <span class="sh">"</span><span class="s">log</span><span class="sh">"</span><span class="p">)</span>
<span class="n">LOG_FILE</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">LOG_DIR</span><span class="p">,</span> <span class="sh">"</span><span class="s">logs.txt</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="ow">not</span> <span class="n">os</span><span class="p">.</span><span class="n">path</span><span class="p">.</span><span class="nf">exists</span><span class="p">(</span><span class="n">LOG_DIR</span><span class="p">):</span>
    <span class="n">os</span><span class="p">.</span><span class="nf">makedirs</span><span class="p">(</span><span class="n">LOG_DIR</span><span class="p">)</span>

<span class="n">logging</span><span class="p">.</span><span class="nf">basicConfig</span><span class="p">(</span>
    <span class="n">level</span><span class="o">=</span><span class="n">logging</span><span class="p">.</span><span class="n">INFO</span><span class="p">,</span>
    <span class="nb">format</span><span class="o">=</span><span class="sh">"</span><span class="s">%(asctime)s [%(levelname)s] %(message)s</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">handlers</span><span class="o">=</span><span class="p">[</span>
        <span class="n">logging</span><span class="p">.</span><span class="nc">FileHandler</span><span class="p">(</span><span class="n">LOG_FILE</span><span class="p">,</span> <span class="n">encoding</span><span class="o">=</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">logging</span><span class="p">.</span><span class="nc">StreamHandler</span><span class="p">(</span><span class="n">sys</span><span class="p">.</span><span class="n">stdout</span><span class="p">)</span>
    <span class="p">]</span>
<span class="p">)</span>

<span class="n">logger</span> <span class="o">=</span> <span class="n">logging</span><span class="p">.</span><span class="nf">getLogger</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">global_exception_handler</span><span class="p">(</span><span class="n">exctype</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tb</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Captures unhandled exceptions and records them in the log.</span><span class="sh">"""</span>
    <span class="n">error_msg</span> <span class="o">=</span> <span class="sh">""</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">traceback</span><span class="p">.</span><span class="nf">format_exception</span><span class="p">(</span><span class="n">exctype</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tb</span><span class="p">))</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Unhandled exception:</span><span class="se">\n</span><span class="si">{</span><span class="n">error_msg</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">sys</span><span class="p">.</span><span class="nf">__excepthook__</span><span class="p">(</span><span class="n">exctype</span><span class="p">,</span> <span class="n">value</span><span class="p">,</span> <span class="n">tb</span><span class="p">)</span>

<span class="n">sys</span><span class="p">.</span><span class="n">excepthook</span> <span class="o">=</span> <span class="n">global_exception_handler</span>

<span class="c1"># 2. Environment Variables Loading
</span><span class="n">env_path</span> <span class="o">=</span> <span class="nf">find_dotenv</span><span class="p">()</span>
<span class="k">if</span> <span class="n">env_path</span><span class="p">:</span>
    <span class="nf">load_dotenv</span><span class="p">(</span><span class="n">env_path</span><span class="p">)</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">.env file loaded from: </span><span class="si">{</span><span class="n">env_path</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="k">else</span><span class="p">:</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">warning</span><span class="p">(</span><span class="sh">"</span><span class="s">.env file not found</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># 3. Tool Definitions
</span><span class="k">def</span> <span class="nf">calculate</span><span class="p">(</span><span class="n">expression</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">
    Solves a simple mathematical expression (addition, subtraction, multiplication, division).
    Useful for calculating year or date differences.

    Args:
        expression (str): The mathematical expression to evaluate (e.g., </span><span class="sh">"</span><span class="s">2024 - 1789</span><span class="sh">"</span><span class="s">).
    </span><span class="sh">"""</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="c1"># Allow only safe characters for basic eval
</span>        <span class="n">allowed_chars</span> <span class="o">=</span> <span class="sh">"</span><span class="s">0123456789+-*/(). </span><span class="sh">"</span>
        <span class="k">if</span> <span class="nf">all</span><span class="p">(</span><span class="n">c</span> <span class="ow">in</span> <span class="n">allowed_chars</span> <span class="k">for</span> <span class="n">c</span> <span class="ow">in</span> <span class="n">expression</span><span class="p">):</span>
            <span class="n">result</span> <span class="o">=</span> <span class="nf">eval</span><span class="p">(</span><span class="n">expression</span><span class="p">)</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Result: </span><span class="si">{</span><span class="n">result</span><span class="si">}</span><span class="sh">"</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">Error: Disallowed characters in expression.</span><span class="sh">"</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Error while calculating: </span><span class="si">{</span><span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span>

<span class="c1"># 4. Agno Agent Configuration (ReAct Pattern)
</span><span class="n">model_id</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">BASE_MODEL</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">gpt-4o</span><span class="sh">"</span><span class="p">)</span>

<span class="n">agent</span> <span class="o">=</span> <span class="nc">Agent</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="nc">OpenAIChat</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="n">model_id</span><span class="p">),</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[</span><span class="nc">TavilyTools</span><span class="p">(),</span> <span class="n">calculate</span><span class="p">],</span>
    <span class="n">instructions</span><span class="o">=</span><span class="p">[</span>
        <span class="sh">"</span><span class="s">You are a researcher using the ReAct (Reason + Act) method.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">1. Think step-by-step about what information you need to answer the user</span><span class="sh">'</span><span class="s">s question.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">2. Use the search tool (Tavily) to find specific dates, facts, or data.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">3. Use the calculator (</span><span class="sh">'</span><span class="s">calculate</span><span class="sh">'</span><span class="s">) for any mathematical operation or time calculation.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">4. Do not guess historical information. If you don</span><span class="sh">'</span><span class="s">t have a piece of data, look it up.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">5. Show your reasoning clearly: </span><span class="sh">'</span><span class="s">Thought:</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="s">Action:</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="s">Observation:</span><span class="sh">'</span><span class="s">.</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">6. Continue investigating until you have a complete and verified answer.</span><span class="sh">"</span>
    <span class="p">],</span>
<span class="p">)</span>

<span class="c1"># 5. User Interface
</span><span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">Starting Historical Detective Agent (ReAct)...</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">--- Historical Detective - ReAct Pattern ---</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">Type </span><span class="sh">'</span><span class="s">exit</span><span class="sh">'</span><span class="s"> to quit.</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">user_input</span> <span class="o">=</span> <span class="nf">input</span><span class="p">(</span><span class="sh">"</span><span class="s">Researcher, what is your question?: </span><span class="sh">"</span><span class="p">)</span>

            <span class="k">if</span> <span class="n">user_input</span><span class="p">.</span><span class="nf">lower</span><span class="p">()</span> <span class="o">==</span> <span class="sh">"</span><span class="s">exit</span><span class="sh">"</span><span class="p">:</span>
                <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">The user has ended the session.</span><span class="sh">"</span><span class="p">)</span>
                <span class="k">break</span>

            <span class="k">if</span> <span class="ow">not</span> <span class="n">user_input</span><span class="p">.</span><span class="nf">strip</span><span class="p">():</span>
                <span class="k">continue</span>

            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">User query: </span><span class="si">{</span><span class="n">user_input</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">Investigating...</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>
            <span class="n">agent</span><span class="p">.</span><span class="nf">print_response</span><span class="p">(</span><span class="n">user_input</span><span class="p">,</span> <span class="n">stream</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">show_tool_calls</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">except</span> <span class="nb">KeyboardInterrupt</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">Keyboard interrupt detected.</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">break</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Error in main loop: </span><span class="si">{</span><span class="nf">str</span><span class="p">(</span><span class="n">e</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">An error occurred: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>

</code></pre>

</div>



<p>¿Dónde está el ReAct en el código?</p>

<ul>
<li><p>PENSAR (Think): Se define en las instructions del agente.<br>
Código: instructions=["You are a researcher using the ReAct method...", "Think step-by-step...", "Show your reasoning..."]. <br>
Esto obliga al LLM a generar un "Thought Trace" (monólogo interno) antes de hacer nada.</p></li>
<li><p>ACTUAR (Act): El agente tiene capacidades externas definidas en tools.<br>
Código: tools=[TavilyTools(), calculate].<br>
Si necesita datos, usa Tavily; si necesita matemáticas, usa calculate.</p></li>
<li><p>OBSERVAR (Observe): El framework (Agno) ejecuta la herramienta y le devuelve el resultado al agente.<br>
Código: Ocurre automáticamente dentro de la clase Agent. El agente "lee" el retorno de calculate (ej: "Result: 6.85").</p></li>
<li><p>REPETIR (Repeat): El bucle continúa hasta que el agente tiene suficiente información.<br>
Código: agent.print_response(..., show_tool_calls=True)<br>
El parámetro show_tool_calls=True es vital: te permite ver en la consola cómo el agente "dispara" las acciones en tiempo real.</p></li>
</ul>

<h3>
  
  
  ¿Por qué nos gusta tanto este patrón? (Pros)
</h3>

<p>Resuelve problemas de "varios saltos" (Multi-hop): Es la capacidad de responder preguntas donde A te lleva a B, y B te lleva a C.</p>

<p>Capacidad de "Auto-curación" (Self-Healing): Esto es vital. Imagina que busca "Edad de Macron" y Google falla. Un script normal colapsaría. Un agente ReAct piensa: "Vaya, la búsqueda falló. Bueno, voy a buscar su fecha de nacimiento y calculo la edad yo mismo".</p>

<p>Adiós a la Caja Negra: Como desarrollador, puedes leer los Thoughts. Sabes exactamente por qué el agente decidió usar la calculadora y no otra cosa.</p>

<p>Menos mentiras (Alucinaciones): Al obligarle a basar cada paso en una observación real ("Observation"), es mucho más difícil que se invente datos.</p>

<h3>
  
  
  No todo es color de rosa (Contras)
</h3>

<p>Es lento (Latencia): ReAct es secuencial. Pensar 3 veces significa llamar al LLM 3 veces + ejecutar herramientas. Prepárate para esperas de 10 a 30 segundos.</p>

<p>La factura de la API (Coste): Ese "monólogo interno" consume tokens como si no hubiera un mañana. Tu historial de contexto se llena rapidísimo.</p>

<p>El riesgo de la obsesión: A veces entran en bucle. "Busco X → No está → Pienso que debo buscar X otra vez → Busco X...".</p>

<p>Prompting delicado: Necesitas un Prompt de Sistema muy bien ajustado para que el modelo sepa cuándo dejar de pensar y darte la respuesta.</p>

<h3>
  
  
  Consejos de trinchera (Ingeniería)
</h3>

<p>Si vas a poner esto en producción (usando Agno, LangChain o lo que sea), grábate esto a fuego:</p>

<p>Ponle un freno (Max Iterations): Configura siempre un límite de iteraciones (ej. 10 pasos). Si no ha resuelto el problema en 10 pasos, no lo va a resolver en 100 y solo te está quemando dinero.</p>

<p>Enséñale a callar (Stop Sequences): Técnicamente, el LLM debe parar de escribir justo después de lanzar la Action. Si no lo cortas ahí, empezará a inventarse la Observation él mismo (alucinar el resultado de la herramienta). Los frameworks modernos suelen manejar esto, pero vigílalo.</p>

<p>Limpia la basura (Gestión de Contexto): En chats largos, borra las trazas de pensamiento antiguas y quédate solo con la respuesta final. Si no, el agente se quedará sin memoria "RAM" (ventana de contexto) enseguida.</p>

<h3>
  
  
  ¿Dónde se usa esto en la vida real?
</h3>

<p>Agentes de Programación (tipo Devin o Copilot): El flujo es: "Escribo código → Ejecuto → Veo el error → Pienso cómo arreglarlo → Reescribo".</p>

<p>Soporte Técnico Nivel 2: "Leo el ticket → Miro el estado del servidor → Miro los logs del usuario → Cruzo datos → Respondo".</p>

<p>Analistas Financieros: "Busco precio actual → Busco noticias de última hora → Comparo con el histórico → Genero recomendación".</p>

<p>Happy Coding! 🤖</p>

