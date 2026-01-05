---
Title: MAWA - El lenguaje simple en sintaxis como Python de bajo nivel Parte 2, Probando mi lenguaje.
Description: 
Author: Samuel Leonardo
Date: 2026-01-05T21:17:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>Bueno por si no era suficiente el BOOM del Imp en MAWA (para los que no sepan de que estoy hablando vayan a este POST <a href="https://dev.to/samuel_leonardo_37aff38b4/mawa-un-lenguaje-tan-simple-como-python-pero-tan-potente-como-assembler-el-asm-moderno-pero-1ng5">https://dev.to/samuel_leonardo_37aff38b4/mawa-un-lenguaje-tan-simple-como-python-pero-tan-potente-como-assembler-el-asm-moderno-pero-1ng5</a> y se contextualizarán del comando Imp) con la sintaxis Imp ('A') o Imp ("Hola"), etc... reduciendo mucho código, pues nunca es suficiente, ya que les traigo ACTUALIZACIONES con nueva información sobre mi lenguaje, Hola soy Samuel Leonardo Páez Garzón tengo 13 años, y este post te va a encantar en especial si te gusta el bajo nivel en la programación, o si eres OSDev, Bueno ahora si las actualizaciones.</p>

<p>Número 1:<br>
Mejora Abismal del Imp</p>

<p>El comando Imp ya permite imprimir cadenas largas, variables y demás cosas en el lenguaje, observen:</p>

<p>Comando:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>String (STR) = "Developer
Imp ("Hola, ", STR.0x0A, " como has estado", 0x04)
</code></pre>

</div>



<p>Salida:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fao80gni94oqg5mp2fuhl.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fao80gni94oqg5mp2fuhl.PNG" alt=" " width="722" height="456"></a></p>

<p>Puedes creer que este código está hecho en 106 Bytes, asombroso MAWA tiene un sistema de funciones que le permite no reescribir rutinas para imprimir o imprimir caracter por caracter sino llamar funciones incrustadas por el compilador.</p>

<p>La estructura de MAWA es bastante chevre y larga para hoy el lenguaje está en un estimado de por ahí casi 7000 líneas y aproximadamente en todos sus modos unos 350 comandos en el bajo nivel tiene aprox. unos 25 o 30 comandos ya que si MAWA tiene compatibilidad con las arquitecturas, esto que significa para su comprensión:</p>

<p>Significa que MAWA puede usarse en Real Mode (16 BITS), Protected Mode (32 BITS) y Long Mode (64 BITS), y muy pronto tendrá un sistema para pasar a 32 BITS o 64 BITS con un solo comando, Wow, No les parece asombroso, he hecho más comandos pero por privacidad, NO los expongo por mi privacidad pero pronto expondré otros de los 25 comandos que ya hay.</p>

<p>Si quieren ver la estructura MAWA es así:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ARQ 16 ; Aquí declaras en que tecnología usarás MAWA
DIR 0x7C00 ; Aquí declaras donde cargas tu código para que MAWA cargue las direcciones de variables o otras cosas en esa dirección

String (Nombre) = "Samuel Leonardo Páez Garzón" ; Aquí creo una variable
Imp ("Hola ", Nombre, 0x04) ; Aquí imprimo la cadena "Hola" y la variable
InfiniteLoop() ; Esto hace un bucle infinito para no seguir avanzando en NASM sería jmp $

#Add (ALL/MWBN) ; Esto incrusta las funciones de MAWA en el archivo si está línea no está el código no te funcionara

Instruction(Rellenar-510) ; Esto rellena a 510 Bytes el archivo necesario si es un BootLoader
Insert Bytes16(0xAA55) ; Esto inserta la firma de arranque 55 AA también este comando se vale por Insert WORD(0xAA55)
</code></pre>

</div>



<p>Si quieren ver el canal oficial de MAWA metansé a este video, les pido que lo vean y lo compartan, aunque no da detalles sobre sintaxis, si te contextualiza de como es el lenguaje aquí en la comunidad DEV es donde muestro eso cuando el canal sea famoso ahí SI mostrare videos con la sintaxis de MAWA:</p>

<p><a href="https://www.youtube.com/watch?v=EnuehLUTByU" rel="noopener noreferrer">https://www.youtube.com/watch?v=EnuehLUTByU</a></p>

<p>Si les gusto MAWA compartan y esperen más novedades PRONTO.</p>

<p>Samuel Leonardo Páez Garzón - 5 de Enero del 2026<br>
Todos los derechos de autor reservados</p>

