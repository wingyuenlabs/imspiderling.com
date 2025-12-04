---
Title: Swift Testing #1: Migración de una Suite de pruebas de XCTest a Swift Testing
Description: 
Author: David Goyes
Date: 2025-12-04T21:54:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Para crear una "suite" de pruebas de XCTest se usa un código parecido al que muestro a continuación:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">import</span> <span class="kt">XCTest</span>
<span class="kd">@testable</span> <span class="kd">import</span> <span class="kt">ProductionProject</span>
<span class="c1">//</span>
<span class="kd">class</span> <span class="kt">SomeXCTest_deprecated</span><span class="p">:</span> <span class="kt">XCTestCase</span> <span class="p">{</span>
    <span class="kd">func</span> <span class="nf">testX</span><span class="p">()</span> <span class="k">throws</span> <span class="p">{</span>
        <span class="c1">// GIVEN </span>
        <span class="c1">// ...</span>

        <span class="c1">// WHEN </span>
        <span class="c1">// ...</span>

        <span class="c1">// THEN ...</span>
        <span class="kt">XCTAssert</span><span class="p">(</span><span class="kt">A</span> <span class="o">==</span> <span class="kt">B</span><span class="p">,</span> <span class="s">"Expected </span><span class="se">\(</span><span class="kt">A</span><span class="se">)</span><span class="s"> but received </span><span class="se">\(</span><span class="kt">B</span><span class="se">)</span><span class="s">"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Si quisiera hacer la conversión a Swift Testing, lo convertiría así, donde puedo destacar el uso de <code>struct</code> en lugar de <code>class</code>, y el hecho de que se usa la etiqueta <code>@Test</code> en lugar del prefijo <code>test</code>. Por otro lado, notar el uso del macro <code>#expect</code> en lugar de <code>XCTAssert</code>. <code>#expect</code> evalúa una condición booleana que se muestra en el reporte de pruebas y que no interrumpe la ejecución del código en caso de que no se cumpla.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">import</span> <span class="kt">Testing</span>
<span class="kd">@testable</span> <span class="kd">import</span> <span class="kt">ProductionProject</span>
<span class="c1">// </span>
<span class="kd">struct</span> <span class="kt">SomeTest</span> <span class="p">{</span>
    <span class="kd">@Test</span>
    <span class="kd">func</span> <span class="nf">x</span><span class="p">()</span> <span class="k">throws</span> <span class="p">{</span>
        <span class="c1">// GIVEN </span>
        <span class="c1">// ...</span>
        <span class="c1">// WHEN </span>
        <span class="c1">// ...</span>
        <span class="c1">// THEN ...</span>
        <span class="cp">#expect(A == B, "Expected \(A) but received \(B)")</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Swift Testing solo está disponible a partir de Xcode 16. Supuéstamente requiere Swift 6 (según la <a href="https://developer.apple.com/documentation/testing/" rel="noopener noreferrer">documentación</a>), sin embargo, creé un proyecto con Swift 5 e igual puedo usar Swift Testing.</p>

<h2>
  
  
  Nombramiento de una prueba
</h2>

<p>En Swift Testing, el nombre de la prueba puede venir dado por el nombre del método, o por una descripción dentro de la etiqueta <code>@Test</code>. También se puede nombrar el método con "backticks" en lugar de usar la descripción dentro de la etiqueta - Hay un error de compilación se se usan las dos técnicas simultáneamente para la misma prueba. Esto representa un salto significativo a nivel de legibilidad del reporte de pruebas, puesto que ahora se pueden usar espacios para separar palabras.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">import</span> <span class="kt">Testing</span>
<span class="kd">@testable</span> <span class="kd">import</span> <span class="kt">ProductionProject</span>
<span class="c1">// </span>
<span class="kd">struct</span> <span class="kt">SomeTest</span> <span class="p">{</span>
    <span class="kd">@Test</span><span class="p">(</span><span class="s">"GIVEN some conditions, WHEN executing something, THEN it should return X"</span><span class="p">)</span>
    <span class="kd">func</span> <span class="nf">x</span><span class="p">()</span> <span class="k">throws</span> <span class="p">{</span>
        <span class="c1">// ...</span>
    <span class="p">}</span>
<span class="c1">//</span>
    <span class="kd">func</span> <span class="err">`</span><span class="kt">GIVEN</span> <span class="kd">some</span> <span class="n">conditions</span><span class="p">,</span> <span class="kt">WHEN</span> <span class="n">executing</span> <span class="n">something</span><span class="p">,</span> <span class="kt">THEN</span> <span class="n">it</span> <span class="n">should</span><span class="o">...</span><span class="err">`</span><span class="p">()</span> <span class="k">throws</span> <span class="p">{</span>
        <span class="c1">// ...</span>
    <span class="p">}</span>
<span class="c1">//</span>
    <span class="kd">@Test</span>
    <span class="kd">func</span> <span class="kt">GIVENsomeconditionsWHENexecutingsomethingTHENitShouldReturnX</span><span class="p">()</span> <span class="k">throws</span> <span class="p">{</span>
        <span class="c1">// ...</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Algunas características adicionales:</p>

<ul>
<li>Se puede tener XCTest y Swift Testing de forma simultánea en el mismo archivo y proyecto.</li>
<li>No es necesario migrar todas las pruebas de XCTest a Swift Testing.</li>
<li>El framework prioriza "value-types" así que, si no se requiere el <code>tearDown</code> (que se implementa con <code>deinit</code> de una clase), hay que procurar usar <code>struct</code>.</li>
</ul>

<p>En el video de Swift and Tips (citado en las referencias) se menciona que hay que importar el paquete (SPM) de swift-testing, aunque por experimentación puedo asegurar que Swift Testing ya viene integrado en Xcode.</p>

<h2>
  
  
  Removiendo dependencia manual de Swift Testing vieja
</h2>

<p>El proyecto del video de Swift and Tips estaba incluyendo un paquete de Swift para Swift Testing (<a href="https://github.com/swiftlang/swift-testing/releases" rel="noopener noreferrer">https://github.com/swiftlang/swift-testing/releases</a>) cuya versión era 0.10.<br>
Con esta versión traté de crear una prueba que fallara (<code>#expect(1 == 2)</code>), pero aunque en la consola aparecía fallida, en el reporte se mostraba como si pasara.<br>
Por esta razón, eliminé la dependencia de SPM para Swift Testing, para trabajar directamente con la versión incluida en Xcode 26.<br>
Para ello, fui a <code>OnlineStoreMV &gt; Project &gt; Package Dependency</code> y eliminé manualmente la dependencia.<br>
Luego, al tratar de compilar las pruebas obtuve el siguiente error, cuando se importaba la biblioteca <code>Testing</code>:</p>

<blockquote>
<p>"unable to find module dependency _TestingInternals"<br>
Para solucionarlo, limpié el proyecto (<code>Cmd+Shift+K</code>) y volví a compilarlo.</p>
</blockquote>
<h2>
  
  
  Interoperabilidad entre XCTAssert y <a class="mentioned-user" href="https://dev.to/test">@test</a>
</h2>

<p>Para que <code>XCTAssert</code>funcione, requiere vivir en una clase de tipo XCTestCase para que sea ejecutada por un <a href="https://developer.apple.com/documentation/xctest/xctestrun" rel="noopener noreferrer"><code>XCTestRun</code></a> en un <a href="https://developer.apple.com/documentation/xctest/xctestsuite" rel="noopener noreferrer"><code>XCTestSuite</code></a>. Por esta razón, si trato de usar un XCTAssert dentro de un método marcado con <code>@Test</code> de Swift Testing, no va a funcionar.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">import</span> <span class="kt">XCTest</span>
<span class="kd">import</span> <span class="kt">Testing</span>
<span class="kd">struct</span> <span class="kt">MyTests</span> <span class="p">{</span>
  <span class="kd">@Test</span>
  <span class="kd">func</span> <span class="nf">something</span><span class="p">()</span> <span class="p">{</span>
    <span class="kt">XCTAssert</span><span class="p">(</span><span class="mi">2</span> <span class="o">==</span> <span class="mi">3</span><span class="p">)</span> <span class="c1">// ¡No funciona! La prueba pasa.</span>
    <span class="cp">#expect(2 == 3) // Aquí la prueba no pasa, que es lo correcto.</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>No obstante, si una prueba marcada con <code>@Test</code> no tiene ningún <code>#expect</code> sino solo <code>XCTAssert</code>, igual es reconocida por el runner de pruebas de Swift Testing.</p>

<h2>
  
  
  Referencias bibliográficas
</h2>

<ul>
<li>Video " Mastering Swift Testing: Installation &amp; Refactoring from XCTest to <a class="mentioned-user" href="https://dev.to/test">@test</a> Macro", <a href="https://www.youtube.com/watch?v=zXjM1cFUwW4" rel="noopener noreferrer">aquí</a>.</li>
<li>Lista de reproducción "Swift Testing" (Swift and Tips), <a href="https://www.youtube.com/playlist?list=PLHWvYoDHvsOV67md_mU5nMN_HDZK7rEKn" rel="noopener noreferrer">aquí</a>.</li>
<li>Documentación sobre Swift Testing, <a href="https://developer.apple.com/documentation/testing" rel="noopener noreferrer">aquí</a>.</li>
<li>Meet Swift Testing, WWDC 2024, <a href="https://developer.apple.com/videos/play/wwdc2024/10179/" rel="noopener noreferrer">aquí</a>.</li>
<li>Go further with Swift Testing, WWDC 2024, <a href="https://developer.apple.com/videos/play/wwdc2024/10195/" rel="noopener noreferrer">aquí</a>.</li>
<li>Repositorio de github de Swift Testing, <a href="https://github.com/swiftlang/swift-testing" rel="noopener noreferrer">aquí</a>.</li>
<li>Artículo, "Is MVVM Necessary for Developing Apps with SwiftUI?", <a href="https://swiftandtips.com/is-mvvm-necessary-for-developing-apps-with-swiftui" rel="noopener noreferrer">aquí</a>.</li>
<li>Proyecto "OnlineStore made with SwiftUI (Vanilla) and Observation", <a href="https://github.com/pitt500/OnlineStoreMV/tree/swift-testing" rel="noopener noreferrer">aquí</a>.</li>
</ul>

