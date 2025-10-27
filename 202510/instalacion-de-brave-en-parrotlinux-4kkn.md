---
Title: Instalación de Brave en Parrot/Kali/Linux
Description: 
Author: mister_keto
Date: 2025-10-27T21:54:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>Es relativamente sencillo instalar Brave en Linux siguiendo estos pasos todo ello en la consola del SO. </p>

<ol>
<li>Actualizamos nuestro sistema operativo.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo apt update &amp;&amp; sudo apt upgrade -y
</code></pre>

</div>



<ol>
<li>Instalamos los paquetes necesarios para que funcione.</li>
</ol>

<p>Brave requiere curl, apt-transport-https y gnupg para manejar los repositorios:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo apt install apt-transport-https curl gnupg -y
</code></pre>

</div>



<ol>
<li>Agrega la clave GPG oficial de Brave</li>
</ol>

<p>Una clave GPG (del inglés GNU Privacy Guard) es una herramienta criptográfica que se utiliza para firmar, cifrar y verificar información o comunicaciones digitales, garantizando la autenticidad, integridad y confidencialidad de los datos.<br>
Sirve para proteger información mediante criptografía de clave pública y privada.</p>

<p>¿Cómo funciona una clave GPG?</p>

<p>Una clave GPG está compuesta por dos partes:</p>

<p><em>Clave privada → solo la posee el dueño.</em></p>

<ul>
<li>Sirve para firmar o descifrar mensajes.</li>
<li>Debe mantenerse en secreto.</li>
</ul>

<p><em>Clave pública → puede compartirse libremente.</em></p>

<ul>
<li>Sirve para que otros verifiquen tus firmas o te envíen mensajes cifrados.</li>
</ul>

<p>añadimos la clave GPG<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
</code></pre>

</div>



<ol>
<li>Añadimos el repositorio de Brave
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg] https://brave-browser-apt-release.s3.brave.com/ stable main" | sudo tee /etc/apt/sources.list.d/brave-browser-release.list
</code></pre>

</div>



<ol>
<li>Instalamos Brave
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo apt update
sudo apt install brave-browser -y
</code></pre>

</div>



<ol>
<li>Ejecutamos Brave de forma gráfica
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>brave-browser
</code></pre>

</div>



