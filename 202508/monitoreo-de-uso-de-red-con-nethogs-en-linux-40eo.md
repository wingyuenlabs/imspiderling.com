---
Title: Monitoreo de Uso de Red con Nethogs en Linux
Description: 
Author: Kemer Ramoz
Date: 2025-08-14T21:39:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>En el entorno de Linux, es esencial contar con herramientas que permitan visualizar el consumo de ancho de banda por cada proceso o aplicación. <strong><a href="https://linuxmind.dev/2025/05/22/ver-uso-de-red-con-nethogs-2/" rel="noopener noreferrer">Nethogs</a></strong> es una de esas herramientas que facilita esta tarea de manera efectiva. A continuación, exploraremos qué es Nethogs, cómo instalarlo y cómo utilizarlo para monitorear el tráfico de red.</p>

<h2>
  
  
  ¿Qué es Nethogs?
</h2>

<p>Nethogs es una utilidad de consola que agrupa el uso de red por procesos en lugar de por conexiones o protocolos. Esto permite identificar rápidamente la fuente del tráfico, ya sea un servicio en segundo plano, una descarga activa o una conexión VPN, lo que resulta extremadamente útil para administradores de sistemas y usuarios avanzados.</p>

<h2>
  
  
  Instalación de Nethogs
</h2>

<p>La instalación de Nethogs varía según la distribución de Linux que utilices. Aquí te mostramos cómo hacerlo en algunas de las más populares:</p>

<ul>
<li>
<strong>Debian/Ubuntu</strong>:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  <span class="nb">sudo </span>apt-get update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt-get <span class="nb">install </span>nethogs
</code></pre>

</div>



<ul>
<li>
<strong>CentOS/RHEL</strong>:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  <span class="nb">sudo </span>yum <span class="nb">install </span>epel-release <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>yum <span class="nb">install </span>nethogs
</code></pre>

</div>



<ul>
<li>
<strong>Fedora</strong>:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  <span class="nb">sudo </span>dnf <span class="nb">install </span>nethogs
</code></pre>

</div>



<ul>
<li>
<strong>Arch Linux</strong>:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>  <span class="nb">sudo </span>pacman <span class="nt">-S</span> nethogs
</code></pre>

</div>



<h2>
  
  
  Uso Básico
</h2>

<p>Una vez instalado, puedes iniciar Nethogs en la interfaz de red que desees (por ejemplo, <code>eth0</code>) con el siguiente comando:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nethogs eth0
</code></pre>

</div>



<p>Esto desplegará un panel que lista los procesos que envían o reciben datos, mostrando información como el nombre del proceso, su PID, y las tasas de envío y recepción en kB/s.</p>

<h2>
  
  
  Opciones Útiles
</h2>

<p>Nethogs ofrece varias opciones que pueden ser útiles para personalizar su funcionamiento:</p>

<ul>
<li>
<code>-d &lt;segundos&gt;</code>: Establece el intervalo de refresco en segundos (por defecto, 1 segundo).</li>
<li>
<code>-t</code>: Activa el modo texto sin cuadros de dibujo.</li>
<li>
<code>-v</code>: Modo verboso que muestra estadísticas adicionales.</li>
<li>
<code>-a</code>: Muestra acumulados desde el arranque.</li>
</ul>

<h3>
  
  
  Ejemplos Prácticos
</h3>

<p>Puedes realizar un análisis cada 2 segundos en modo texto con el siguiente comando:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nethogs <span class="nt">-d</span> 2 <span class="nt">-t</span> eth0
</code></pre>

</div>



<p>Para monitorear múltiples interfaces al mismo tiempo, utiliza:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nethogs eth0 wlan0
</code></pre>

</div>



<h2>
  
  
  Integración con VPN
</h2>

<p>Nethogs es compatible con el uso de túneles VPN. Para analizar el tráfico de una interfaz virtual como <code>tun0</code> o <code>wg0</code>, simplemente ejecuta:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nethogs tun0
</code></pre>

</div>



<p>Esto te permitirá diferenciar el consumo de tráfico real de la conexión segura frente al tráfico local.</p>

<h2>
  
  
  Solución de Problemas Comunes
</h2>

<ul>
<li>
<strong>Permisos insuficientes</strong>: Recuerda que Nethogs requiere privilegios de root o sudo para funcionar correctamente.</li>
<li>
<strong>Interfaz no encontrada</strong>: Verifica el nombre correcto de la interfaz con el comando <code>ip link show</code>.</li>
<li>
<strong>Tráfico cifrado no aparece separado</strong>: Esto puede suceder si la VPN asigna puertos dinámicos. En este caso, analiza el total de <code>tun0</code> o <code>wg0</code>.</li>
</ul>

<h2>
  
  
  Conclusión
</h2>

<p>Nethogs es una herramienta ligera, intuitiva y muy eficaz para identificar el uso de red por proceso en sistemas Linux. Su fácil instalación, modo interactivo y compatibilidad con entornos VPN la convierten en un recurso indispensable para cualquier administrador o usuario avanzado que desee optimizar el análisis de tráfico de red. ¡Agrega Nethogs a tu caja de herramientas y mejora tu gestión del ancho de banda!</p>

