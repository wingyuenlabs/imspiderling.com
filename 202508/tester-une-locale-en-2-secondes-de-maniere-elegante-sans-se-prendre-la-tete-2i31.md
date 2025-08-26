---
Title: 🧪 Tester une locale en 2 secondes de manière élégante (sans se prendre la tête)
Description: 
Author: OCL
Date: 2025-08-26T21:43:52.000Z
Robots: noindex,nofollow
Template: index
---
<p>Cet article explique comment valider une locale (comme fr-FR ou en-US) en 2 secondes pour éviter les erreurs d'affichage ou les crashes, en utilisant Intl.DateTimeFormat qui lève une erreur si la locale est invalide.</p>




<p>Tu veux tester une locale parce que t’as peur que ton utilisateur ait déclaré <code>mi-MI</code> pour du <strong>Minion</strong> ou <code>tlh-QAAK</code> pour du <strong>Klingon</strong> ?</p>




<h2>
  
  
  ⚠️ Le scénario catastrophe
</h2>

<p><strong>Imagine</strong> : ton utilisateur sélectionne une <strong>locale inexistante</strong> (<code>xyz-XYZ</code>), et soudain :</p>

<ul>
<li>Ton frontend affiche des <em>undefined</em> partout.</li>
<li>Ton backend crash avec un <code>RangeError: invalid language tag</code>.
<strong>Avec ce snippet, tu évites ça en 2 lignes.</strong>
</li>
</ul>




<h2>
  
  
  🇫🇷 Une locale, c’est quoi ?
</h2>

<p>Les codes comme <code>fr-FR</code> ou <code>en-US</code> suivent la norme <strong>IETF BCP 47</strong>. Ils se composent de :</p>

<ul>
<li>
<strong>2 lettres pour la langue</strong> (<code>fr</code>, <code>en</code>… selon <strong>ISO 639-1</strong>).</li>
<li>
<strong>2 lettres pour le pays</strong> (<code>FR</code>, <code>US</code>… selon <strong>ISO 3166-1 alpha-2</strong>).</li>
</ul>




<h2>
  
  
  😱 Le problème : 45 000 combinaisons possibles*
</h2>

<ul>
<li>
<strong>180 langues</strong> × <strong>250 pays</strong> = <strong>45 000 locales théoriques</strong>.
&gt; <em>"45 000 ?! 🥵 ¿La cucaracha?"</em>
</li>
</ul>

<p><em>*Heureusement, 99% sont inutiles</em></p>




<h2>
  
  
  ✅ La solution : <code>Zod</code> + <code>Intl</code> (ou <code>Intl</code> seul)
</h2>

<h3>
  
  
  Avec <code>Zod</code> (pour les fans de validation) :
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">zod</span><span class="dl">"</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">localeSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">refine</span><span class="p">(</span>
  <span class="p">(</span><span class="nx">locale</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">try</span> <span class="p">{</span>
      <span class="k">new</span> <span class="nx">Intl</span><span class="p">.</span><span class="nc">DateTimeFormat</span><span class="p">(</span><span class="nx">locale</span><span class="p">);</span>
      <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
    <span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
      <span class="k">return</span> <span class="kc">false</span><span class="p">;</span>
    <span class="p">}</span>
  <span class="p">},</span>
  <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Locale invalide. Essayez 'fr-FR' ou 'en-US'.</span><span class="dl">"</span> <span class="p">}</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Sans <code>Zod</code> (version minimaliste) :
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">isValidLocale</span><span class="p">(</span><span class="nx">locale</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">boolean</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="k">new</span> <span class="nx">Intl</span><span class="p">.</span><span class="nc">DateTimeFormat</span><span class="p">(</span><span class="nx">locale</span><span class="p">);</span>
    <span class="k">return</span> <span class="kc">true</span><span class="p">;</span>
  <span class="p">}</span> <span class="k">catch</span> <span class="p">{</span>
    <span class="k">return</span> <span class="kc">false</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  🎯 Résultat :
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">testLocale</span><span class="p">(</span><span class="dl">"</span><span class="s2">fr-FR</span><span class="dl">"</span><span class="p">);</span> <span class="c1">// ✅ Valide</span>
<span class="nf">testLocale</span><span class="p">(</span><span class="dl">"</span><span class="s2">tlh-QAAK</span><span class="dl">"</span><span class="p">);</span> <span class="c1">// ❌ Invalide (désolé, fans de Star Trek)</span>
</code></pre>

</div>






<h2>
  
  
  🔧 Bonus : Quelles locales sont supportées ?
</h2>

<p>Pour vérifier <strong>ce que ton environnement supporte</strong> (navigateur/Node.js) :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="nx">Intl</span><span class="p">.</span><span class="nx">DateTimeFormat</span><span class="p">.</span><span class="nf">supportedLocalesOf</span><span class="p">([</span><span class="dl">"</span><span class="s2">fr-FR</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">tlh-QAAK</span><span class="dl">"</span><span class="p">]));</span>
<span class="c1">// → ["fr-FR"] (le klingon n’est pas encore standardisé)</span>
</code></pre>

</div>



<p><strong>Pourquoi ça marche ?</strong><br>
<code>Intl.DateTimeFormat</code> lève une erreur si la locale est invalide. On exploite ce mécanisme pour valider en 1 ligne.</p>




<h2>
  
  
  👍 Pour en savoir plus :
</h2>

<ul>
<li><a href="https://tools.ietf.org/html/bcp47" rel="noopener noreferrer">IETF BCP 47</a></li>
<li><a href="https://en.wikipedia.org/wiki/ISO_639-1" rel="noopener noreferrer">ISO 639-1</a></li>
<li><a href="https://en.wikipedia.org/wiki/ISO_3166-1" rel="noopener noreferrer">ISO 3166-1</a></li>
</ul>




<p><strong>Pense à valider tes locales et sur ce, Poulétikamassala !</strong> 👋</p>

<p>Cet article a été initialement publié le 20/08/205 sur mon <a href="https://oclcode.hashnode.dev/tester-une-locale" rel="noopener noreferrer">blog hashnode.dev</a>.</p>

