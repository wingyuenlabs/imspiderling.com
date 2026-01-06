---
Title: MAWA - El lenguaje simple en sintaxis como Python de bajo nivel. Parte 3, Condicionales.
Description: 
Author: Samuel Leonardo
Date: 2026-01-06T22:03:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>Bueno señoras y señores vuelvo con novedades de MAWA que ya sucedieron pero dije que iba a dar más detalles así que aquí volví de nuevo después de un día.</p>

<p>Para los que no esten contextualizados acerca de MAWA o no conozcan algunos de sus comandos pueden ver la serie de este mismo POST para ver las partes anteriores.</p>

<p>Las Condicionales:<br>
Bueno para que se hagan una idea así se denomina un registro en MAWA:<br>
En este caso estamos en Real Mode (16 BITS)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Register (AL) = 255 o 0xFF
Register (BX) = 65535 o 0xFFFF
Register (ECX) = 4,294,967,295 o 0xFFFFFFFF
</code></pre>

</div>



<p>He introducido hace un poco de tiempo ya en el bajo nivel las condicionales nuevas evitando el estorboso aunque no feo de igual manera proceso de comparar valores en Assembler con la funcion <strong>cmp</strong>, ya que en ASM así le dices al CPU que te compare 2 valores:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>; Con el registro AL por ejemplo
cmp al, 5
je seCumplio
jne noSeCumplio
</code></pre>

</div>



<p>No es difícil PERO si es algo confuso para algunos ya que no se denomina, como en C++ con:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="c1">// Suponiendo que AL es un registro aunque aca sería una variable</span>
<span class="k">if</span> <span class="p">(</span><span class="n">al</span> <span class="o">==</span> <span class="mi">5</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Ejecucion del If</span>
<span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
    <span class="c1">// Ejecucion del else</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Aquí directamente se pone lo que se ejecuta dentro de los corchetes mientras que ASM no lo hace así sino hace calls a funciones pero MAWA ha quitado esa limitación inventada por mí mostrandonos esto:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Register (AL) = 5
Register (BL) = AL

Si: (AL == BL)
If:
    Imp ("Se cumplió")
Else:
    Imp ("No se cumplió")
End If
</code></pre>

</div>



<p>Llegan las condicionales permitiendonos comparar valores como registros, variables en memoria, o direcciones explicitas e incluso valores normales en un instante.</p>

<p>Permitiendonos usar operadores lógicos como AND o OR y también varias condiciones como el Else If para variar.</p>

<p>Si quieren ver el canal oficial de MAWA metansé a este video, les pido que lo vean y lo compartan, aunque no da detalles sobre sintaxis, si te contextualiza de como es el lenguaje aquí en la comunidad DEV es donde muestro eso cuando el canal sea famoso ahí SI mostrare videos con la sintaxis de MAWA:</p>

<p><a href="https://www.youtube.com/watch?v=EnuehLUTByU" rel="noopener noreferrer">https://www.youtube.com/watch?v=EnuehLUTByU</a></p>

<p>Si les gusto compartan y no se pierdan la parte 4</p>

<p>También estoy creando un Sistema Operativo pronto hablare bastante de él aunque en mi cuenta ya hay un post sobre él.</p>

<p>Samuel Leonardo Páez Garzón<br>
6 de Enero de 2025 - Todos los derechos de autor reservados</p>

