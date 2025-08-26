---
Title: Fundamentos modernos y primera interacción con IA. Parte 1
Description: 
Author: Antonio Manuel Pérez López
Date: 2025-08-26T21:28:01.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Más allá de la aplicación web tradicional
</h2>

<p>Como desarrollador con décadas de experiencia, he sido testigo y protagonista de la evolución de la web: desde los monolitos renderizados en servidor, pasando por la revolución de las SPAs (Single-Page Applications) con AJAX, hasta el auge de los frameworks de componentes como React. Cada etapa trajo consigo nuevos patrones de diseño, desafíos de rendimiento y abstracciones. Hoy, nos encontramos en el umbral de una transformación de similar magnitud: la era de las aplicaciones de IA nativas.</p>

<p>Este capítulo no pretende enseñarte a programar (seguramente no sería un buen ejemplo a seguir). Su objetivo es investigar un nuevo <em>modelo mental</em> arquitectónico. Las aplicaciones que integraremos con Inteligencia Artificial Generativa no son simplemente aplicaciones tradicionales con una API de LLM atornillada. Requieren un enfoque fundamentalmente diferente en cuanto a la renderización, el manejo del estado, la validación de datos y la comunicación entre cliente y servidor. Las herramientas que hemos elegido —Next.js con App Router, TypeScript en su máxima expresión y el Vercel AI SDK— no son meras preferencias, sino decisiones arquitectónicas deliberadas que responden a los desafíos únicos de este nuevo paradigma.</p>

<p>Abordaremos tres pilares fundamentales:</p>

<ol>
<li> <strong>La arquitectura de renderizado de Next.js:</strong> Cómo los React Server Components (RSC) nos permiten crear interfaces más rápidas y seguras, un requisito indispensable para las costosas y, a menudo, lentas inferencias de los modelos de IA.</li>
<li> <strong>TypeScript como contrato de sistema:</strong> Cómo usamos tipos avanzados y validación con Zod no solo para la seguridad, sino para definir un "contrato" estricto con una entidad no determinista como es un LLM.</li>
<li> <strong>Vercel AI SDK como capa de comunicación:</strong> Cómo una capa de abstracción bien diseñada nos permite mantener la agilidad y evitar el acoplamiento a un único proveedor de IA en un ecosistema que evoluciona a una velocidad vertiginosa.</li>
</ol>

<p>Prepárate para reevaluar algunos patrones establecidos y adoptar un nuevo conjunto de herramientas mentales. Empecemos.</p>




<h2>
  
  
  Arquitectura avanzada en Next.js: El paradigma de los React Server Components (RSC)
</h2>

<p>El App Router de Next.js no es una simple mejora sobre el Pages Router; es un cambio de paradigma completo que devuelve gran parte de la lógica de renderizado al servidor, donde siempre debió estar para ciertas tareas. Para nosotros, como arquitectos de aplicaciones de IA, esto tiene implicaciones profundas.</p>

<h3>
  
  
  El dúo dinámico: Server Components vs. Client Components
</h3>

<p>El modelo mental clave a interiorizar es que, por defecto, <strong>todo es un Server Component</strong>. Un RSC es un fragmento de UI que se ejecuta exclusivamente en el servidor, una única vez durante el ciclo de vida de la petición. Su resultado es una descripción de la UI (un formato intermedio similar a HTML) que se envía al cliente.</p>

<p><strong>Características clave de los Server Components (RSC):</strong></p>

<ul>
<li>  <strong>Javascript de cliente cero:</strong> No envían ni una sola línea de su propio código JavaScript al navegador. Esto resulta en un TTI (Time to Interactive) drásticamente reducido.</li>
<li>  <strong>Acceso directo al backend:</strong> Pueden acceder directamente a recursos del servidor (bases de datos, sistemas de archivos, APIs internas, variables de entorno secretas) sin necesidad de una capa de API intermedia. Esto es revolucionario para la IA, ya que nos permite obtener contexto o ejecutar lógica de backend directamente desde el componente que lo necesita.</li>
<li>  <strong>Renderizado asíncrono:</strong> Pueden ser funciones <code>async</code>. Esto permite realizar operaciones de datos (ej. <code>fetch</code>, consultas a BBDD) y esperar su resolución antes de completar el renderizado.</li>
</ul>

<p>Por otro lado, cuando necesitamos interactividad (manejar eventos, estado, hooks de ciclo de vida), marcamos un componente con la directiva <code>"use client"</code>. Esto no convierte solo a ese componente en un Client Component, sino que establece una <strong>frontera</strong>: ese componente y todos los que importe (que no sean a su vez RSC) formarán parte del bundle de JavaScript que se envía al cliente para su hidratación.</p>

<h3>
  
  
  Patrones de composición para aplicaciones de IA
</h3>

<p>La maestría no reside en saber qué es un RSC o un Client Component, sino en saber cómo combinarlos. Para las aplicaciones de IA, estos patrones son cruciales:</p>

<ul>
<li>  <strong>Patrón de "Cáscara Estática, Contenido Dinámico" (Static Shell, Dynamic Content):</strong> La estructura principal de la aplicación (layout, navegación, cabeceras) se define como RSCs. Son estáticos, se cargan instantáneamente y no cambian. Dentro de esta cáscara, las áreas interactivas (como una ventana de chat o un formulario de prompt) se definen como Client Components. Esto nos da lo mejor de ambos mundos: una carga inicial ultrarrápida y una interactividad rica donde se necesita.</li>
<li>  <strong>Streaming de UI con Suspense:</strong> Las respuestas de los LLMs pueden tardar varios segundos en empezar a generarse (Time To First Token). Hacer que el usuario espere a que toda la página cargue es inaceptable. Los RSC se integran nativamente con <code>Suspense</code>. Podemos envolver un componente RSC que realiza una llamada a un LLM en un <code>Suspense</code> con un <code>fallback</code> (p. ej., un spinner). Next.js enviará inmediatamente el HTML de la página con el spinner, y cuando la llamada al LLM resuelva, streameará la UI resultante para reemplazar el spinner, sin necesidad de una recarga de página.</li>
<li>  <strong>Server Actions: Mutaciones sin API boilerplate:</strong> Son funciones asíncronas que se definen en el servidor (en un RSC o en un archivo con <code>"use server"</code>) pero que se pueden invocar directamente desde un Client Component. Simplifican radicalmente las mutaciones de datos. En nuestro contexto, una Server Action es la forma perfecta de manejar el envío de un prompt desde un formulario en un Client Component a nuestra lógica de backend que llamará a la IA, sin tener que escribir manualmente un <code>fetch</code> a un endpoint de API.</li>
</ul>

<p><strong>Diagrama arquitectónico conceptual:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwe4vipwnr25me86wtc28.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwe4vipwnr25me86wtc28.png" alt="Diagrama Arquitectónico Conceptual" width="728" height="478"></a></p>

<p>Dominar esta arquitectura es el primer paso para construir aplicaciones de IA que sean robustas y ofrezcan una experiencia de usuario excepcional.</p>




<h2>
  
  
  TypeScript para expertos: Blindando la interfaz con la IA
</h2>

<p>Si tienes experiencia en javascript seguro que has visto cómo los sistemas de tipos han pasado de ser una curiosidad académica a una herramienta indispensable en sistemas a gran escala. En el desarrollo de IA, TypeScript trasciende la mera seguridad de tipos para convertirse en una herramienta de <strong>diseño de contratos</strong>. Nuestra interacción con un LLM es, por naturaleza, una comunicación con un sistema no determinista. Nuestro trabajo como ingenieros es imponer determinismo y estructura sobre esa interacción.</p>

<h3>
  
  
  Inferencia y genéricos para abstracciones reutilizables
</h3>

<p>No nos detendremos en la sintaxis básica. En cambio, nos centraremos en cómo los genéricos avanzados nos permiten construir abstracciones robustas. Por ejemplo, podemos crear una función de envoltura para las llamadas a la IA que maneje la validación, el logging y los errores de forma genérica, sin importar la forma (schema) de la respuesta esperada:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">async</span> <span class="kd">function</span> <span class="nf">safeAICall</span><span class="o">&lt;</span><span class="nx">T</span> <span class="kd">extends</span> <span class="nx">z</span><span class="p">.</span><span class="nx">ZodType</span><span class="o">&gt;</span><span class="p">(</span>
  <span class="nx">prompt</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="nx">schema</span><span class="p">:</span> <span class="nx">T</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nx">infer</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">}</span> <span class="o">|</span> <span class="p">{</span> <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="na">error</span><span class="p">:</span> <span class="kr">string</span> <span class="p">}</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="c1">// ... lógica de llamada a la IA ...</span>
  <span class="c1">// ... validación de la respuesta contra el schema ...</span>
  <span class="c1">// Devolvemos un tipo discriminado seguro</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Este patrón nos permite manejar diferentes "formas" de respuesta de la IA de una manera completamente type-safe.</p>

<h3>
  
  
  Zod: El contrato inmutable entre humano y máquina
</h3>

<p><code>zod</code> es la pieza central de nuestra estrategia de validación. Lo tratamos como la <strong>única fuente de verdad</strong> para la estructura de los datos que fluyen hacia y desde el LLM.</p>

<ul>
<li>  <strong>Definición del Schema:</strong> Un schema de Zod es más que una validación; es una declaración de intenciones. Define la estructura exacta que esperamos.</li>
<li>  <strong>Inferencia de tipos (<code>z.infer</code>):</strong> La magia de Zod reside en <code>z.infer&lt;typeof miSchema&gt;</code>. Esto nos permite derivar automáticamente un tipo de TypeScript estático a partir de un validador de runtime. Se acabaron las definiciones de tipos duplicadas y la desincronización entre la validación y el tipado estático.</li>
<li>  <strong>El Método <code>.describe()</code>:</strong> A menudo subestimado, el método <code>.describe()</code> es crucial en el desarrollo con IA. Cuando en capítulos posteriores pidamos al LLM que genere JSON estructurado, las descripciones que proporcionamos en el schema de Zod actúan como "hints" o instrucciones para el modelo, mejorando drásticamente la calidad y fiabilidad de su salida.</li>
</ul>

<p><strong>Flujo de datos validado:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsy2n23n000kuk8lgs0op.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsy2n23n000kuk8lgs0op.png" alt="Flujo de Datos Validado" width="800" height="48"></a></p>

<p>Este doble filtro, en la entrada y en la salida, es nuestra principal línea de defensa contra respuestas inesperadas o malformadas del LLM, garantizando que el resto de nuestra aplicación siempre opere sobre datos predecibles y seguros.</p>




<h2>
  
  
  Vercel AI SDK Core: La capa de comunicación unificada
</h2>

<p>El ecosistema de la IA generativa está en un estado de cambio constante. Los modelos se actualizan, nuevos proveedores emergen y los precios fluctúan. Acoplar nuestra lógica de negocio directamente a la API de un proveedor específico (sea OpenAI, Google, Anthropic, etc.) es una receta para la obsolescencia técnica y la rigidez.</p>

<p>El Vercel AI SDK nos proporciona una capa de abstracción esencial que nos aísla de esta complejidad.</p>

<h3>
  
  
  El principio "proveedor agnóstico"
</h3>

<p>La belleza del AI SDK radica en su simplicidad. Cambiar de un modelo de Google a uno de Anthropic (suponiendo capacidades similares) es, idealmente, un cambio de una sola línea:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// De:</span>
<span class="kd">const</span> <span class="nx">model</span> <span class="o">=</span> <span class="nf">google</span><span class="p">(</span><span class="dl">'</span><span class="s1">gemini-2.5-pro</span><span class="dl">'</span><span class="p">);</span>
<span class="c1">// A:</span>
<span class="kd">const</span> <span class="nx">model</span> <span class="o">=</span> <span class="nf">anthropic</span><span class="p">(</span><span class="dl">'</span><span class="s1">claude-3-opus-20240229</span><span class="dl">'</span><span class="p">);</span>
</code></pre>

</div>



<p>Esta abstracción nos permite experimentar, comparar y pivotar entre proveedores con un coste de ingeniería mínimo, una capacidad estratégica en este campo.</p>

<h3>
  
  
  Anatomía de <code>generateText</code>
</h3>

<p><code>generateText</code> es nuestra herramienta de trabajo fundamental para interacciones simples y no conversacionales. Su firma encapsula los elementos esenciales de una llamada a un LLM:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">text</span><span class="p">,</span> <span class="nx">usage</span><span class="p">,</span> <span class="nx">finishReason</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">generateText</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="nf">google</span><span class="p">(</span><span class="dl">'</span><span class="s1">gemini-2.5-pro</span><span class="dl">'</span><span class="p">),</span>
  <span class="na">system</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Eres un asistente experto en literatura del Siglo de Oro.</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">prompt</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Analiza la métrica del primer soneto de Garcilaso.</span><span class="dl">'</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<ul>
<li>  <strong><code>model</code>:</strong> La instancia del modelo que hemos elegido, abstraída por el SDK.</li>
<li>  <strong><code>system</code>:</strong> El "system prompt" es nuestra oportunidad para dar al modelo una personalidad, un contexto y unas reglas de comportamiento generales que aplicarán a toda la interacción. Es una de las herramientas más potentes del prompt engineering.</li>
<li>  <strong><code>prompt</code>:</strong> La instrucción específica para esta llamada.</li>
</ul>

<p>El objeto devuelto nos da no solo el <code>text</code> generado, sino también metadatos valiosos como <code>usage</code> (el recuento de tokens, crucial para el control de costes) y <code>finishReason</code> (por qué el modelo dejó de generar texto).</p>

<h3>
  
  
  El poder del streaming con <code>streamText</code>
</h3>

<p>Para cualquier aplicación interactiva, el streaming no es una opción, es una necesidad. <code>streamText</code> es la contraparte asíncrona de <code>generateText</code>. En lugar de esperar a que la respuesta completa esté disponible, nos devuelve un <code>AsyncIterable</code> que podemos consumir a medida que los "tokens" llegan desde el servidor.</p>

<p>Esto se integra perfectamente con la arquitectura de RSC y Suspense que discutimos. En el backend, un Route Handler puede usar <code>streamText</code> y devolver el stream directamente. En el frontend, el hook <code>useChat</code> (que veremos en el próximo capítulo) consumirá este stream para renderizar la respuesta de forma progresiva, mejorando drásticamente la experiencia de usuario percibida.</p>

<h2>
  
  
  Conclusiones de la parte teórica
</h2>

<p>Hemos sentado las bases. No hemos escrito una aplicación de IA completa todavía, pero hemos hecho algo más importante: hemos establecido una arquitectura robusta y un modelo mental preparado para los desafíos que nos esperan.</p>

<p>Hemos entendido que la combinación de <strong>renderizado en servidor (RSC)</strong> para la estructura, <strong>validación estricta de contratos (TypeScript/Zod)</strong> para los datos y una <strong>capa de comunicación agnóstica (Vercel AI SDK)</strong> para la lógica de IA, nos proporciona la flexibilidad, seguridad y rendimiento necesarios para construir aplicaciones de IA de nivel profesional.</p>

<p>Con estos cimientos en su sitio, estamos listos para construir nuestra primera aplicación interactiva. En los próximos capítulos, tomaremos estos conceptos y los aplicaremos para crear un chatbot en tiempo real, manejando el streaming de principio a fin.</p>




