---
Title: Creare una PWA con Laravel e Bootstrap | Building a PWA with Laravel and Bootstrap
Description: 
Author: Roberto Celano
Date: 2025-10-18T22:07:26.000Z
Robots: noindex,nofollow
Template: index
---
<h3><center>Introduzione | Introduction</center></h3>  

<p><strong>Italiano:</strong> Questo articolo è disponibile sia in italiano che in inglese. Scrolla verso il basso per la versione in inglese.<br><br>
<strong>English:</strong> This article is available in both Italian and English. Scroll down for the English version. </p>


<h2>
  
  
  🇮🇹 Come ho trasformato un sito <strong>Laravel</strong> in una <strong>PWA</strong> moderna
</h2>

<p>Negli ultimi anni le <strong>Progressive Web App</strong> (<strong>PWA</strong>) sono diventate uno standard per offrire un’esperienza simile a quella delle app native, direttamente dal browser. Nel mio caso, volevo che il mio portfolio personale – sviluppato in <strong>Laravel</strong> + <strong>Bootstrap</strong> – potesse essere installato su dispositivi mobile e desktop, mantenendo la leggerezza di un sito classico.</p>
<h3>
  
  
  1. Creare il Manifest
</h3>

<p>Il primo passo è aggiungere il file <code>manifest.json</code> nella directory <code>public/</code>.<br>
Contiene le informazioni che definiscono il comportamento della tua app una volta installata.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Roberto Celano | Web Developer"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"short_name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RC Dev"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"start_url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"display"</span><span class="p">:</span><span class="w"> </span><span class="s2">"standalone"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"background_color"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#0d1117"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"theme_color"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#00bcd4"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"icons"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"src"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/assets/icons/icon-192.png"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"sizes"</span><span class="p">:</span><span class="w"> </span><span class="s2">"192x192"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"image/png"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"src"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/assets/icons/icon-512.png"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"sizes"</span><span class="p">:</span><span class="w"> </span><span class="s2">"512x512"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"image/png"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Ricorda di linkarlo nel tuo layout principale (<code>resources/views/layouts/app.blade.php</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"manifest"</span> <span class="na">href=</span><span class="s">"{{ asset('manifest.json') }}"</span><span class="nt">&gt;</span>
</code></pre>

</div>



<h3>
  
  
  2. Registrare il Service Worker
</h3>

<p>Nel file <code>public/service-worker.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nb">self</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">install</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Service Worker installato</span><span class="dl">"</span><span class="p">);</span>
<span class="p">});</span>

<span class="nb">self</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">fetch</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">event</span><span class="p">.</span><span class="nf">respondWith</span><span class="p">(</span><span class="nf">fetch</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">request</span><span class="p">));</span>
<span class="p">});</span>
</code></pre>

</div>



<p>E nella tua view Blade, aggiungi:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nt">&lt;script&gt;</span>
<span class="k">if </span><span class="p">(</span><span class="dl">"</span><span class="s2">serviceWorker</span><span class="dl">"</span> <span class="k">in</span> <span class="nb">navigator</span><span class="p">)</span> <span class="p">{</span>
  <span class="nb">navigator</span><span class="p">.</span><span class="nx">serviceWorker</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="dl">"</span><span class="s2">/service-worker.js</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Service Worker registrato</span><span class="dl">"</span><span class="p">))</span>
    <span class="p">.</span><span class="k">catch</span><span class="p">(</span><span class="nx">err</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Errore:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">err</span><span class="p">));</span>
<span class="p">}</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>



<h3>
  
  
  3. Aggiungere il prompt d’installazione
</h3>

<p>Con questo snippet puoi mostrare il messaggio “<strong>Installa App</strong>”:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">deferredPrompt</span><span class="p">;</span>
<span class="nb">window</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">beforeinstallprompt</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
  <span class="nx">deferredPrompt</span> <span class="o">=</span> <span class="nx">e</span><span class="p">;</span>
  <span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">installButton</span><span class="dl">"</span><span class="p">).</span><span class="nx">style</span><span class="p">.</span><span class="nx">display</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">block</span><span class="dl">"</span><span class="p">;</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Quando l’utente clicca il bottone:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">installButton</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">click</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">deferredPrompt</span><span class="p">.</span><span class="nf">prompt</span><span class="p">();</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">outcome</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">deferredPrompt</span><span class="p">.</span><span class="nx">userChoice</span><span class="p">;</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Scelta utente:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">outcome</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  4. Test con Lighthouse
</h3>

<p>Apri <strong>Chrome</strong> → <strong>DevTools</strong> → <strong>Lighthouse</strong> → <strong>Progressive Web App</strong>.<br>
Ti mostrerà se la tua <strong>PWA</strong> è installabile e conforme agli standard.</p>
<h3>
  
  
  5. Risultato finale
</h3>

<p>Ora il mio sito <strong>robertocelano.dev</strong> può essere installato come app su smartphone, tablet e desktop.<br>
L’esperienza è fluida, il caricamento è rapido e l’interfaccia mantiene il design Bootstrap originale.</p>


<h2>
  
  
  🇬🇧 How I turned a <strong>Laravel</strong> site into a modern <strong>PWA</strong>
</h2>

<p>In recent years, <strong>Progressive Web Apps</strong> (<strong>PWA</strong>) have become the standard for bringing native-like experiences to the browser. In my case, I wanted my personal portfolio — built with Laravel + Bootstrap — to be installable on mobile and desktop, while keeping the simplicity of a classic website.</p>
<h3>
  
  
  1. Create the Manifest
</h3>

<p>Add a <code>manifest.json</code> file in your <code>public/ directory</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Roberto Celano | Web Developer"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"short_name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RC Dev"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"start_url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"display"</span><span class="p">:</span><span class="w"> </span><span class="s2">"standalone"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"background_color"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#0d1117"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"theme_color"</span><span class="p">:</span><span class="w"> </span><span class="s2">"#00bcd4"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"icons"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"src"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/assets/icons/icon-192.png"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"sizes"</span><span class="p">:</span><span class="w"> </span><span class="s2">"192x192"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"image/png"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"src"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/assets/icons/icon-512.png"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"sizes"</span><span class="p">:</span><span class="w"> </span><span class="s2">"512x512"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"image/png"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Then link it in your main Blade layout (<code>resources/views/layouts/app.blade.php</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"manifest"</span> <span class="na">href=</span><span class="s">"{{ asset('manifest.json') }}"</span><span class="nt">&gt;</span>
</code></pre>

</div>



<h3>
  
  
  2. Register the Service Worker
</h3>

<p>In <code>public/service-worker.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nb">self</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">install</span><span class="dl">"</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Service Worker installed</span><span class="dl">"</span><span class="p">);</span>
<span class="p">});</span>

<span class="nb">self</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">fetch</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">event</span><span class="p">.</span><span class="nf">respondWith</span><span class="p">(</span><span class="nf">fetch</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">request</span><span class="p">));</span>
<span class="p">});</span>
</code></pre>

</div>



<p>And in your Blade view:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="nt">&lt;script&gt;</span>
<span class="k">if </span><span class="p">(</span><span class="dl">"</span><span class="s2">serviceWorker</span><span class="dl">"</span> <span class="k">in</span> <span class="nb">navigator</span><span class="p">)</span> <span class="p">{</span>
  <span class="nb">navigator</span><span class="p">.</span><span class="nx">serviceWorker</span><span class="p">.</span><span class="nf">register</span><span class="p">(</span><span class="dl">"</span><span class="s2">/service-worker.js</span><span class="dl">"</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">then</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Service Worker registered</span><span class="dl">"</span><span class="p">))</span>
    <span class="p">.</span><span class="k">catch</span><span class="p">(</span><span class="nx">err</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Error:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">err</span><span class="p">));</span>
<span class="p">}</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>



<h3>
  
  
  3. Add the installation prompt
</h3>

<p>With this snippet you can show the “<strong>Install App</strong>” message:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">let</span> <span class="nx">deferredPrompt</span><span class="p">;</span>
<span class="nb">window</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">beforeinstallprompt</span><span class="dl">"</span><span class="p">,</span> <span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">e</span><span class="p">.</span><span class="nf">preventDefault</span><span class="p">();</span>
  <span class="nx">deferredPrompt</span> <span class="o">=</span> <span class="nx">e</span><span class="p">;</span>
  <span class="nb">document</span><span class="p">.</span><span class="nf">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">installButton</span><span class="dl">"</span><span class="p">).</span><span class="nx">style</span><span class="p">.</span><span class="nx">display</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">block</span><span class="dl">"</span><span class="p">;</span>
<span class="p">});</span>
</code></pre>

</div>



<p>When the user clicks the button:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">installButton</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">click</span><span class="dl">"</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">deferredPrompt</span><span class="p">.</span><span class="nf">prompt</span><span class="p">();</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">outcome</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">deferredPrompt</span><span class="p">.</span><span class="nx">userChoice</span><span class="p">;</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">User choice:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">outcome</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  4. Test with Lighthouse
</h3>

<p>Open <strong>Chrome</strong> → <strong>DevTools</strong> → <strong>Lighthouse</strong> → <strong>Progressive Web App</strong>.<br>
It will show whether your <strong>PWA</strong> meets installability standards.</p>

<h3>
  
  
  5. Final result
</h3>

<p>Now <strong>robertocelano.dev</strong> can be installed like an app on smartphones, tablets, and desktops — fast, responsive, and consistent with its Bootstrap design.</p>




<h2>
  
  
  <center>Traduzione</center>
</h2>

<blockquote>
<p>Questo articolo è stato tradotto con l'ausilio di strumenti di traduzione professionali.<br>
This article was translated with the help of professional translation tools.</p>
</blockquote>

