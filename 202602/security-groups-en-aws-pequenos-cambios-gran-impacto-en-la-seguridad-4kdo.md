---
Title: Security Groups en AWS: pequeños cambios, gran impacto en la seguridad
Description: 
Author: Ender Salas
Date: 2026-02-02T21:56:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>🧠 ¿Qué es un Security Group?</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fshtfhtyfazj2hg90brri.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fshtfhtyfazj2hg90brri.png" alt=" " width="800" height="533"></a></p>

<p>Un Security Group es un <strong>firewall virtual</strong> que controla qué tráfico puede entrar y salir de tus recursos en AWS (EC2, Load Balancer, RDS, etc.).</p>

<p>Funciona a nivel de <strong>instancia o servicio</strong>, no de red completa, y es el primer nivel real de defensa en la mayoría de las arquitecturas AWS.</p>

<p>📌 Son stateful:<br>
Si permites la entrada, la respuesta está permitida automáticamente.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjtbrxnnsrkfqbl0jsp5j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjtbrxnnsrkfqbl0jsp5j.png" alt=" " width="512" height="261"></a></p>

<p>🛠️ ¿Cómo se usa?<br>
Los Security Groups se basan en reglas explícitas:</p>

<ul>
<li>Puerto</li>
<li>Protocolo</li>
<li>Origen / destino</li>
</ul>

<p>Ejemplo simple:<br>
Permitir tráfico HTTP/HTTPS solo desde un Application Load Balancer</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fso1m4vre317d095a08cn.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fso1m4vre317d095a08cn.jpg" alt=" " width="800" height="376"></a></p>

<p>Bloquear cualquier otro acceso directo a la instancia</p>

<p>No se “niega” tráfico explícitamente:<br>
👉_ todo lo que no está permitido, queda bloqueado._</p>

<p><strong>✅ Buenas prácticas clave</strong></p>

<p>✔️ Principio de mínimo privilegio<br>
Abrir solo los puertos estrictamente necesarios.<br>
✔️ Referenciar Security Groups, no IPs<br>
Ejemplo: permitir tráfico desde el SG del Load Balancer al SG de la aplicación.<br>
✔️ Nunca usar 0.0.0.0/0 por comodidad<br>
Menos aún en puertos sensibles (22, 3389, bases de datos).<br>
✔️ Un rol, un Security Group<br>
Evita SG “comodín” usados por todo.<br>
✔️ Documentar las reglas<br>
Un SG sin contexto es deuda técnica.</p>

<p>🧩 Ejemplo práctico (arquitectura común):</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faikfm5lnqhlo8mfycbm0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faikfm5lnqhlo8mfycbm0.png" alt=" " width="800" height="457"></a></p>

<p><strong>ALB (público)</strong><br>
Permite 80/443 desde internet</p>

<p><strong>EC2 App .NET (privada)</strong><br>
Permite 80/443 solo desde el SG del ALB</p>

<p>Resultado:<br>
La aplicación <strong>no es accesible desde internet</strong><br>
Todo el tráfico pasa por un punto controlado</p>

<p>🚨 ¿Por qué es tan importante configurarlos bien?</p>

<p>Porque:</p>

<ul>
<li>Un Security Group mal configurado expone tu aplicación</li>
<li>Muchos incidentes de seguridad comienzan con un 0.0.0.0/0</li>
<li>Son fáciles de crear… y fáciles de romper</li>
</ul>

<p><strong>💡 En AWS, muchas veces la seguridad no falla por falta de servicios, sino por malas reglas.</strong></p>

<p>🏁 <strong>Cierre</strong><br>
Antes de pensar en WAF, Zero Trust o arquitecturas complejas,<br>
aprende a usar bien los Security Groups.</p>

<p>En AWS, un buen Security Group puede salvarte de un muy mal día.</p>

