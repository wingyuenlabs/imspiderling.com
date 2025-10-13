---
Title: MLZC25-22. Regularización en Regresión Logística: Ajuste Fino del Parámetro C para Mejor Generalización
Description: 
Author: Jesus Oviedo Riquelme
Date: 2025-10-13T22:11:29.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>🎯 Objetivo del Post</strong>: Aprenderás qué es la regularización, por qué es crucial para evitar overfitting, cómo funciona el parámetro C en regresión logística, y cómo encontrar el valor óptimo que maximice el rendimiento en validación.</p>
</blockquote>

<h2>
  
  
  🎯 El Problema del Overfitting
</h2>

<p>Imagina que estás estudiando para un examen. Puedes:</p>

<p><strong>Opción A</strong>: Memorizar todas las respuestas del libro de práctica</p>

<ul>
<li>✅ Perfecto en práctica (100%)</li>
<li>❌ Malo en el examen real (60%)</li>
</ul>

<p><strong>Opción B</strong>: Entender los conceptos fundamentales</p>

<ul>
<li>✅ Bueno en práctica (85%)</li>
<li>✅ Bueno en el examen real (82%)</li>
</ul>

<p><strong>Opción A = Overfitting</strong> (memorización)<br>
<strong>Opción B = Generalización</strong> (comprensión)</p>
<h3>
  
  
  Overfitting en ML
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Modelo sobreajustado
</span><span class="n">Accuracy</span> <span class="n">Train</span><span class="p">:</span> <span class="mi">99</span><span class="o">%</span>  <span class="err">✅</span> <span class="err">¡</span><span class="n">Excelente</span><span class="err">!</span>
<span class="n">Accuracy</span> <span class="n">Val</span><span class="p">:</span>   <span class="mi">70</span><span class="o">%</span>  <span class="err">❌</span> <span class="err">¡</span><span class="n">Terrible</span><span class="err">!</span>
<span class="c1"># → El modelo memorizó el training set pero no generalizó
</span>
<span class="c1"># Modelo bien regularizado
</span><span class="n">Accuracy</span> <span class="n">Train</span><span class="p">:</span> <span class="mi">85</span><span class="o">%</span>  <span class="err">✓</span> <span class="n">Bueno</span>
<span class="n">Accuracy</span> <span class="n">Val</span><span class="p">:</span>   <span class="mi">84</span><span class="o">%</span>  <span class="err">✓</span> <span class="n">Bueno</span>
<span class="c1"># → El modelo aprendió patrones generales
</span></code></pre>

</div>

<h2>
  
  
  🔧 ¿Qué es la Regularización?
</h2>

<p><strong>Regularización</strong> es una técnica para <strong>penalizar la complejidad</strong> del modelo, forzándolo a ser más simple y generalizar mejor.</p>
<h3>
  
  
  Analogía: Simplificar para Entender Mejor
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Sin Regularización</th>
<th>Con Regularización</th>
</tr>
</thead>
<tbody>
<tr>
<td>"Si llueve Y es lunes Y es marzo Y el viento sopla del norte Y... → lleva paraguas"</td>
<td>"Si llueve → lleva paraguas"</td>
</tr>
<tr>
<td>Regla compleja, específica</td>
<td>Regla simple, general</td>
</tr>
<tr>
<td>Overfitting</td>
<td>Generalización</td>
</tr>
</tbody>
</table></div>
<h3>
  
  
  Tipos de Regularización
</h3>

<ol>
<li>
<p><strong>L1 (Lasso)</strong></p>

<ul>
<li>Penaliza la <strong>suma absoluta</strong> de coeficientes</li>
<li>Algunos coeficientes se vuelven <strong>exactamente cero</strong>
</li>
<li><strong>Selección automática de features</strong></li>
</ul>
</li>
<li>
<p><strong>L2 (Ridge)</strong> ← <strong>Usado en Logistic Regression por defecto</strong></p>

<ul>
<li>Penaliza la <strong>suma de cuadrados</strong> de coeficientes</li>
<li>Coeficientes se vuelven <strong>pequeños</strong> pero no cero</li>
<li><strong>Reduce magnitud de todos los coeficientes</strong></li>
</ul>
</li>
<li>
<p><strong>ElasticNet</strong></p>

<ul>
<li>Combina L1 y L2</li>
<li>Balance entre ambas técnicas</li>
</ul>
</li>
</ol>
<h2>
  
  
  📊 El Parámetro C en Logistic Regression
</h2>

<p>En scikit-learn, el parámetro <strong>C</strong> controla la <strong>fuerza de la regularización</strong>:</p>
<h3>
  
  
  Definición
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">C</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">/</span> <span class="n">λ</span>
</code></pre>

</div>


<p>Donde λ (lambda) es el parámetro de regularización tradicional.</p>
<h3>
  
  
  Interpretación de C
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Valor de C</th>
<th>λ</th>
<th>Regularización</th>
<th>Efecto</th>
</tr>
</thead>
<tbody>
<tr>
<td>
<strong>C muy pequeño</strong> (0.01)</td>
<td>λ muy grande</td>
<td><strong>Fuerte</strong></td>
<td>Modelo muy simple, puede underfit</td>
</tr>
<tr>
<td>
<strong>C pequeño</strong> (0.1)</td>
<td>λ grande</td>
<td>Fuerte</td>
<td>Modelo simple</td>
</tr>
<tr>
<td>
<strong>C medio</strong> (1.0)</td>
<td>λ medio</td>
<td>Moderada</td>
<td>Balance</td>
</tr>
<tr>
<td>
<strong>C grande</strong> (10)</td>
<td>λ pequeño</td>
<td>Débil</td>
<td>Modelo complejo</td>
</tr>
<tr>
<td>
<strong>C muy grande</strong> (100)</td>
<td>λ muy pequeño</td>
<td><strong>Muy débil</strong></td>
<td>Modelo muy complejo, puede overfit</td>
</tr>
</tbody>
</table></div>
<h3>
  
  
  Visualización del Efecto de C
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Complejidad del Modelo
    │
    │                                    ╱ Overfitting
    │                              ╱
    │                        ╱
    │                  ╱        
    │            ╱              
    │      ╱                    ← Sweet spot
    │╱ Underfitting              
    └────────────────────────────────────────────
        0.01    0.1     1.0    10     100         C

        ←── Más regularización   Menos regularización ──→
</code></pre>

</div>

<h2>
  
  
  💻 Implementación: Encontrar el Mejor C
</h2>
<h3>
  
  
  Paso 1: Preparar los Datos
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pandas</span> <span class="k">as</span> <span class="n">pd</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">import</span> <span class="n">matplotlib.pyplot</span> <span class="k">as</span> <span class="n">plt</span>
<span class="kn">from</span> <span class="n">sklearn.model_selection</span> <span class="kn">import</span> <span class="n">train_test_split</span>
<span class="kn">from</span> <span class="n">sklearn.linear_model</span> <span class="kn">import</span> <span class="n">LogisticRegression</span>
<span class="kn">from</span> <span class="n">sklearn.feature_extraction</span> <span class="kn">import</span> <span class="n">DictVectorizer</span>
<span class="kn">from</span> <span class="n">sklearn.metrics</span> <span class="kn">import</span> <span class="n">accuracy_score</span>

<span class="c1"># Cargar y limpiar datos
</span><span class="n">url</span> <span class="o">=</span> <span class="sh">"</span><span class="s">https://raw.githubusercontent.com/alexeygrigorev/datasets/master/course_lead_scoring.csv</span><span class="sh">"</span>
<span class="n">df</span> <span class="o">=</span> <span class="n">pd</span><span class="p">.</span><span class="nf">read_csv</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

<span class="n">categorical_cols</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">select_dtypes</span><span class="p">(</span><span class="n">include</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">object</span><span class="sh">'</span><span class="p">]).</span><span class="n">columns</span><span class="p">.</span><span class="nf">tolist</span><span class="p">()</span>
<span class="n">numerical_cols</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">select_dtypes</span><span class="p">(</span><span class="n">include</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">int64</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">float64</span><span class="sh">'</span><span class="p">]).</span><span class="n">columns</span><span class="p">.</span><span class="nf">tolist</span><span class="p">()</span>

<span class="n">df_clean</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">copy</span><span class="p">()</span>
<span class="k">for</span> <span class="n">col</span> <span class="ow">in</span> <span class="n">categorical_cols</span><span class="p">:</span>
    <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">]</span> <span class="o">=</span> <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">].</span><span class="nf">fillna</span><span class="p">(</span><span class="sh">'</span><span class="s">NA</span><span class="sh">'</span><span class="p">)</span>
<span class="k">for</span> <span class="n">col</span> <span class="ow">in</span> <span class="n">numerical_cols</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">col</span> <span class="o">!=</span> <span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">:</span>
        <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">]</span> <span class="o">=</span> <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">].</span><span class="nf">fillna</span><span class="p">(</span><span class="mf">0.0</span><span class="p">)</span>

<span class="c1"># División de datos
</span><span class="n">df_train_full</span><span class="p">,</span> <span class="n">df_temp</span> <span class="o">=</span> <span class="nf">train_test_split</span><span class="p">(</span><span class="n">df_clean</span><span class="p">,</span> <span class="n">test_size</span><span class="o">=</span><span class="mf">0.4</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
<span class="n">df_val</span><span class="p">,</span> <span class="n">df_test</span> <span class="o">=</span> <span class="nf">train_test_split</span><span class="p">(</span><span class="n">df_temp</span><span class="p">,</span> <span class="n">test_size</span><span class="o">=</span><span class="mf">0.5</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>

<span class="n">y_train</span> <span class="o">=</span> <span class="n">df_train_full</span><span class="p">[</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">].</span><span class="n">values</span>
<span class="n">y_val</span> <span class="o">=</span> <span class="n">df_val</span><span class="p">[</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">].</span><span class="n">values</span>
<span class="n">y_test</span> <span class="o">=</span> <span class="n">df_test</span><span class="p">[</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">].</span><span class="n">values</span>

<span class="n">X_train_df</span> <span class="o">=</span> <span class="n">df_train_full</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="n">X_val_df</span> <span class="o">=</span> <span class="n">df_val</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="n">X_test_df</span> <span class="o">=</span> <span class="n">df_test</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="c1"># One-hot encoding
</span><span class="n">dv</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">X_train</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
<span class="n">X_val</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
<span class="n">X_test</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_test_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ Datos preparados</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Train: </span><span class="si">{</span><span class="n">X_train</span><span class="p">.</span><span class="n">shape</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Val:   </span><span class="si">{</span><span class="n">X_val</span><span class="p">.</span><span class="n">shape</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Test:  </span><span class="si">{</span><span class="n">X_test</span><span class="p">.</span><span class="n">shape</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>

<h3>
  
  
  Paso 2: Probar Diferentes Valores de C
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">QUESTION 6: BÚSQUEDA DEL MEJOR PARÁMETRO C</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="c1"># Valores de C a probar (según el homework)
</span><span class="n">C_values</span> <span class="o">=</span> <span class="p">[</span><span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">100</span><span class="p">]</span>

<span class="c1"># Diccionario para guardar resultados
</span><span class="n">results</span> <span class="o">=</span> <span class="p">{}</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="si">{</span><span class="sh">'</span><span class="s">C</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">8</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Acc Train</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Acc Val</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Acc Val (3 dec)</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">18</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Nota</span><span class="sh">'</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="k">for</span> <span class="n">C</span> <span class="ow">in</span> <span class="n">C_values</span><span class="p">:</span>
    <span class="c1"># Entrenar modelo con C específico
</span>    <span class="n">model</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span>
        <span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> 
        <span class="n">C</span><span class="o">=</span><span class="n">C</span><span class="p">,</span> 
        <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> 
        <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span>
    <span class="p">)</span>
    <span class="n">model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

    <span class="c1"># Evaluar en train y validation
</span>    <span class="n">y_pred_train</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_train</span><span class="p">)</span>
    <span class="n">y_pred_val</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">)</span>

    <span class="n">accuracy_train</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_train</span><span class="p">,</span> <span class="n">y_pred_train</span><span class="p">)</span>
    <span class="n">accuracy_val</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_val</span><span class="p">)</span>

    <span class="c1"># Redondear a 3 decimales (según homework)
</span>    <span class="n">accuracy_val_rounded</span> <span class="o">=</span> <span class="nf">round</span><span class="p">(</span><span class="n">accuracy_val</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span>

    <span class="c1"># Guardar resultado
</span>    <span class="n">results</span><span class="p">[</span><span class="n">C</span><span class="p">]</span> <span class="o">=</span> <span class="n">accuracy_val_rounded</span>

    <span class="c1"># Determinar nota sobre overfitting
</span>    <span class="n">diff</span> <span class="o">=</span> <span class="n">accuracy_train</span> <span class="o">-</span> <span class="n">accuracy_val</span>
    <span class="k">if</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.02</span><span class="p">:</span>
        <span class="n">nota</span> <span class="o">=</span> <span class="sh">"</span><span class="s">✓ Bien balanceado</span><span class="sh">"</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.05</span><span class="p">:</span>
        <span class="n">nota</span> <span class="o">=</span> <span class="sh">"</span><span class="s">~ Leve overfit</span><span class="sh">"</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">nota</span> <span class="o">=</span> <span class="sh">"</span><span class="s">⚠ Overfit</span><span class="sh">"</span>

    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">C</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">8.2</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">accuracy_train</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">12.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">accuracy_val</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">12.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
          <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">accuracy_val_rounded</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">18.3</span><span class="n">f</span><span class="si">}</span><span class="s">  </span><span class="si">{</span><span class="n">nota</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>
</code></pre>

</div>


<p><strong>Salida esperada</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>QUESTION 6: BÚSQUEDA DEL MEJOR PARÁMETRO C
======================================================================

       C    Acc Train       Acc Val  Acc Val (3 dec)  Nota
----------------------------------------------------------------------
    0.01     0.847765     0.845890               0.846  ✓ Bien balanceado
    0.10     0.849260     0.849315               0.849  ✓ Bien balanceado
    1.00     0.849315     0.849315               0.849  ✓ Bien balanceado
   10.00     0.849315     0.849315               0.849  ✓ Bien balanceado
  100.00     0.849315     0.849315               0.849  ✓ Bien balanceado
----------------------------------------------------------------------
</code></pre>

</div>



<h3>
  
  
  Paso 3: Identificar el Mejor C
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">ANÁLISIS DE RESULTADOS</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="c1"># Encontrar el máximo accuracy
</span><span class="n">max_accuracy</span> <span class="o">=</span> <span class="nf">max</span><span class="p">(</span><span class="n">results</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span>

<span class="c1"># Encontrar todos los C con máximo accuracy
</span><span class="n">best_Cs</span> <span class="o">=</span> <span class="p">[</span><span class="n">c</span> <span class="k">for</span> <span class="n">c</span><span class="p">,</span> <span class="n">acc</span> <span class="ow">in</span> <span class="n">results</span><span class="p">.</span><span class="nf">items</span><span class="p">()</span> <span class="k">if</span> <span class="n">acc</span> <span class="o">==</span> <span class="n">max_accuracy</span><span class="p">]</span>

<span class="c1"># Si hay empate, seleccionar el menor C (más regularización)
</span><span class="n">best_C</span> <span class="o">=</span> <span class="nf">min</span><span class="p">(</span><span class="n">best_Cs</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Resultados ordenados por accuracy:</span><span class="sh">"</span><span class="p">)</span>
<span class="k">for</span> <span class="n">c</span><span class="p">,</span> <span class="n">acc</span> <span class="ow">in</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">results</span><span class="p">.</span><span class="nf">items</span><span class="p">(),</span> <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="p">(</span><span class="o">-</span><span class="n">x</span><span class="p">[</span><span class="mi">1</span><span class="p">],</span> <span class="n">x</span><span class="p">[</span><span class="mi">0</span><span class="p">])):</span>
    <span class="n">marker</span> <span class="o">=</span> <span class="sh">"</span><span class="s"> ← MEJOR</span><span class="sh">"</span> <span class="k">if</span> <span class="n">c</span> <span class="o">==</span> <span class="n">best_C</span> <span class="k">else</span> <span class="sh">""</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  C = </span><span class="si">{</span><span class="n">c</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">6.2</span><span class="n">f</span><span class="si">}</span><span class="s">  →  Accuracy = </span><span class="si">{</span><span class="n">acc</span><span class="si">:</span><span class="p">.</span><span class="mi">3</span><span class="n">f</span><span class="si">}{</span><span class="n">marker</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">🎯 Mejor C: </span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Accuracy: </span><span class="si">{</span><span class="n">max_accuracy</span><span class="si">:</span><span class="p">.</span><span class="mi">3</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">best_Cs</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">💡 Nota: Empate entre C = </span><span class="si">{</span><span class="n">best_Cs</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Seleccionamos el menor C (</span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="s">) → Más regularización</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Razón: Modelo más simple con el mismo rendimiento</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">✅ RESPUESTA QUESTION 6: </span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Paso 4: Visualizar el Efecto de C
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Preparar datos para visualización
</span><span class="n">Cs_list</span> <span class="o">=</span> <span class="nf">sorted</span><span class="p">(</span><span class="n">results</span><span class="p">.</span><span class="nf">keys</span><span class="p">())</span>
<span class="n">accuracies</span> <span class="o">=</span> <span class="p">[</span><span class="n">results</span><span class="p">[</span><span class="n">c</span><span class="p">]</span> <span class="k">for</span> <span class="n">c</span> <span class="ow">in</span> <span class="n">Cs_list</span><span class="p">]</span>

<span class="c1"># Crear gráfico
</span><span class="n">fig</span><span class="p">,</span> <span class="p">(</span><span class="n">ax1</span><span class="p">,</span> <span class="n">ax2</span><span class="p">)</span> <span class="o">=</span> <span class="n">plt</span><span class="p">.</span><span class="nf">subplots</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">15</span><span class="p">,</span> <span class="mi">5</span><span class="p">))</span>

<span class="c1"># Gráfico 1: Accuracy vs C (escala logarítmica)
</span><span class="n">ax1</span><span class="p">.</span><span class="nf">plot</span><span class="p">(</span><span class="n">Cs_list</span><span class="p">,</span> <span class="n">accuracies</span><span class="p">,</span> <span class="sh">'</span><span class="s">o-</span><span class="sh">'</span><span class="p">,</span> <span class="n">linewidth</span><span class="o">=</span><span class="mi">2</span><span class="p">,</span> <span class="n">markersize</span><span class="o">=</span><span class="mi">8</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="sh">'</span><span class="s">steelblue</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">axhline</span><span class="p">(</span><span class="n">y</span><span class="o">=</span><span class="n">max_accuracy</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="sh">'</span><span class="s">red</span><span class="sh">'</span><span class="p">,</span> <span class="n">linestyle</span><span class="o">=</span><span class="sh">'</span><span class="s">--</span><span class="sh">'</span><span class="p">,</span> 
            <span class="n">label</span><span class="o">=</span><span class="sa">f</span><span class="sh">'</span><span class="s">Máximo: </span><span class="si">{</span><span class="n">max_accuracy</span><span class="si">:</span><span class="p">.</span><span class="mi">3</span><span class="n">f</span><span class="si">}</span><span class="sh">'</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">set_xscale</span><span class="p">(</span><span class="sh">'</span><span class="s">log</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">set_xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">C (escala logarítmica)</span><span class="sh">'</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">set_ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Accuracy en Validación</span><span class="sh">'</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">set_title</span><span class="p">(</span><span class="sh">'</span><span class="s">Accuracy vs Parámetro C</span><span class="sh">'</span><span class="p">,</span> <span class="n">fontsize</span><span class="o">=</span><span class="mi">14</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">grid</span><span class="p">(</span><span class="n">alpha</span><span class="o">=</span><span class="mf">0.3</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">legend</span><span class="p">()</span>

<span class="c1"># Marcar el mejor C
</span><span class="n">best_idx</span> <span class="o">=</span> <span class="n">Cs_list</span><span class="p">.</span><span class="nf">index</span><span class="p">(</span><span class="n">best_C</span><span class="p">)</span>
<span class="n">ax1</span><span class="p">.</span><span class="nf">plot</span><span class="p">(</span><span class="n">best_C</span><span class="p">,</span> <span class="n">accuracies</span><span class="p">[</span><span class="n">best_idx</span><span class="p">],</span> <span class="sh">'</span><span class="s">r*</span><span class="sh">'</span><span class="p">,</span> <span class="n">markersize</span><span class="o">=</span><span class="mi">20</span><span class="p">,</span> 
         <span class="n">label</span><span class="o">=</span><span class="sa">f</span><span class="sh">'</span><span class="s">Mejor C = </span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="sh">'</span><span class="p">)</span>

<span class="c1"># Gráfico 2: Diferencia respecto al mejor
</span><span class="n">differences</span> <span class="o">=</span> <span class="p">[</span><span class="n">acc</span> <span class="o">-</span> <span class="n">max_accuracy</span> <span class="k">for</span> <span class="n">acc</span> <span class="ow">in</span> <span class="n">accuracies</span><span class="p">]</span>
<span class="n">colors</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">green</span><span class="sh">'</span> <span class="k">if</span> <span class="n">d</span> <span class="o">==</span> <span class="mi">0</span> <span class="k">else</span> <span class="sh">'</span><span class="s">orange</span><span class="sh">'</span> <span class="k">if</span> <span class="n">d</span> <span class="o">&gt;</span> <span class="o">-</span><span class="mf">0.01</span> <span class="k">else</span> <span class="sh">'</span><span class="s">red</span><span class="sh">'</span> 
          <span class="k">for</span> <span class="n">d</span> <span class="ow">in</span> <span class="n">differences</span><span class="p">]</span>

<span class="n">ax2</span><span class="p">.</span><span class="nf">bar</span><span class="p">(</span><span class="nf">range</span><span class="p">(</span><span class="nf">len</span><span class="p">(</span><span class="n">Cs_list</span><span class="p">)),</span> <span class="n">differences</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="n">colors</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span><span class="p">)</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">set_xticks</span><span class="p">(</span><span class="nf">range</span><span class="p">(</span><span class="nf">len</span><span class="p">(</span><span class="n">Cs_list</span><span class="p">)))</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">set_xticklabels</span><span class="p">([</span><span class="sa">f</span><span class="sh">'</span><span class="si">{</span><span class="n">c</span><span class="si">}</span><span class="sh">'</span> <span class="k">for</span> <span class="n">c</span> <span class="ow">in</span> <span class="n">Cs_list</span><span class="p">])</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">set_xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">C</span><span class="sh">'</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">set_ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Diferencia vs Máximo</span><span class="sh">'</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">set_title</span><span class="p">(</span><span class="sh">'</span><span class="s">Pérdida de Accuracy vs Mejor Modelo</span><span class="sh">'</span><span class="p">,</span> 
              <span class="n">fontsize</span><span class="o">=</span><span class="mi">14</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">axhline</span><span class="p">(</span><span class="n">y</span><span class="o">=</span><span class="mi">0</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="sh">'</span><span class="s">black</span><span class="sh">'</span><span class="p">,</span> <span class="n">linestyle</span><span class="o">=</span><span class="sh">'</span><span class="s">-</span><span class="sh">'</span><span class="p">,</span> <span class="n">linewidth</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">ax2</span><span class="p">.</span><span class="nf">grid</span><span class="p">(</span><span class="n">axis</span><span class="o">=</span><span class="sh">'</span><span class="s">y</span><span class="sh">'</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.3</span><span class="p">)</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">tight_layout</span><span class="p">()</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<h2>
  
  
  📊 Análisis Detallado del Efecto de C
</h2>

<h3>
  
  
  Comparar Train vs Validation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">COMPARACIÓN TRAIN vs VALIDATION POR C</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">80</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="si">{</span><span class="sh">'</span><span class="s">C</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">8</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Train Acc</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Val Acc</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Diferencia</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">12</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Interpretación</span><span class="sh">'</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">80</span><span class="p">)</span>

<span class="k">for</span> <span class="n">C</span> <span class="ow">in</span> <span class="n">C_values</span><span class="p">:</span>
    <span class="n">model</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="n">C</span><span class="p">,</span> 
                               <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
    <span class="n">model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

    <span class="n">acc_train</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_train</span><span class="p">,</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_train</span><span class="p">))</span>
    <span class="n">acc_val</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">))</span>
    <span class="n">diff</span> <span class="o">=</span> <span class="n">acc_train</span> <span class="o">-</span> <span class="n">acc_val</span>

    <span class="k">if</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.01</span><span class="p">:</span>
        <span class="n">interp</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Excelente generalización</span><span class="sh">"</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.03</span><span class="p">:</span>
        <span class="n">interp</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Buena generalización</span><span class="sh">"</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.05</span><span class="p">:</span>
        <span class="n">interp</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Leve overfitting</span><span class="sh">"</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">interp</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Overfitting significativo</span><span class="sh">"</span>

    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">C</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">8.2</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">acc_train</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">12.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">acc_val</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">12.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
          <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">diff</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">12.6</span><span class="n">f</span><span class="si">}</span><span class="s">  </span><span class="si">{</span><span class="n">interp</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Analizar Coeficientes
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">EFECTO DE C EN LOS COEFICIENTES</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="c1"># Entrenar modelos con C extremos
</span><span class="n">model_low_C</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="mf">0.01</span><span class="p">,</span> 
                                  <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
<span class="n">model_high_C</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="mi">100</span><span class="p">,</span> 
                                   <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>

<span class="n">model_low_C</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
<span class="n">model_high_C</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

<span class="c1"># Comparar magnitud de coeficientes
</span><span class="n">coef_low</span> <span class="o">=</span> <span class="n">model_low_C</span><span class="p">.</span><span class="n">coef_</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>
<span class="n">coef_high</span> <span class="o">=</span> <span class="n">model_high_C</span><span class="p">.</span><span class="n">coef_</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Estadísticas de coeficientes:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="si">{</span><span class="sh">'</span><span class="s">Métrica</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">C = 0.01</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">15</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">C = 100</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">15</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">60</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="sh">'</span><span class="s">Media absoluta</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">np</span><span class="p">.</span><span class="nf">mean</span><span class="p">(</span><span class="n">np</span><span class="p">.</span><span class="nf">abs</span><span class="p">(</span><span class="n">coef_low</span><span class="p">))</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">np</span><span class="p">.</span><span class="nf">mean</span><span class="p">(</span><span class="n">np</span><span class="p">.</span><span class="nf">abs</span><span class="p">(</span><span class="n">coef_high</span><span class="p">))</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="sh">'</span><span class="s">Máximo absoluto</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">np</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="n">np</span><span class="p">.</span><span class="nf">abs</span><span class="p">(</span><span class="n">coef_low</span><span class="p">))</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">np</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="n">np</span><span class="p">.</span><span class="nf">abs</span><span class="p">(</span><span class="n">coef_high</span><span class="p">))</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="sh">'</span><span class="s">Std de coeficientes</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">np</span><span class="p">.</span><span class="nf">std</span><span class="p">(</span><span class="n">coef_low</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">np</span><span class="p">.</span><span class="nf">std</span><span class="p">(</span><span class="n">coef_high</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">💡 Observación:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   C pequeño (0.01) → Coeficientes más pequeños (más regularización)</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   C grande (100) → Coeficientes más grandes (menos regularización)</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  🎯 Evaluación Final en Test Set
</h2>

<p>Una vez encontrado el mejor C, evaluamos en el conjunto de test:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">EVALUACIÓN FINAL EN TEST SET</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="c1"># Entrenar modelo final con el mejor C
</span><span class="n">final_model</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span>
    <span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> 
    <span class="n">C</span><span class="o">=</span><span class="n">best_C</span><span class="p">,</span> 
    <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> 
    <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span>
<span class="p">)</span>
<span class="n">final_model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

<span class="c1"># Evaluar en todos los conjuntos
</span><span class="n">y_pred_train_final</span> <span class="o">=</span> <span class="n">final_model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_train</span><span class="p">)</span>
<span class="n">y_pred_val_final</span> <span class="o">=</span> <span class="n">final_model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">)</span>
<span class="n">y_pred_test_final</span> <span class="o">=</span> <span class="n">final_model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_test</span><span class="p">)</span>

<span class="n">acc_train_final</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_train</span><span class="p">,</span> <span class="n">y_pred_train_final</span><span class="p">)</span>
<span class="n">acc_val_final</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_val_final</span><span class="p">)</span>
<span class="n">acc_test_final</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_test</span><span class="p">,</span> <span class="n">y_pred_test_final</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Modelo final (C = </span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="s">):</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Accuracy Train: </span><span class="si">{</span><span class="n">acc_train_final</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="s"> (</span><span class="si">{</span><span class="n">acc_train_final</span><span class="o">*</span><span class="mi">100</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s">%)</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Accuracy Val:   </span><span class="si">{</span><span class="n">acc_val_final</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="s"> (</span><span class="si">{</span><span class="n">acc_val_final</span><span class="o">*</span><span class="mi">100</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s">%)</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Accuracy Test:  </span><span class="si">{</span><span class="n">acc_test_final</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="s"> (</span><span class="si">{</span><span class="n">acc_test_final</span><span class="o">*</span><span class="mi">100</span><span class="si">:</span><span class="p">.</span><span class="mi">2</span><span class="n">f</span><span class="si">}</span><span class="s">%)</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Verificar consistencia
</span><span class="k">if</span> <span class="nf">abs</span><span class="p">(</span><span class="n">acc_val_final</span> <span class="o">-</span> <span class="n">acc_test_final</span><span class="p">)</span> <span class="o">&lt;</span> <span class="mf">0.02</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">✅ Excelente: Val y Test son muy similares</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   El modelo generaliza bien a datos nuevos</span><span class="sh">"</span><span class="p">)</span>
<span class="k">elif</span> <span class="nf">abs</span><span class="p">(</span><span class="n">acc_val_final</span> <span class="o">-</span> <span class="n">acc_test_final</span><span class="p">)</span> <span class="o">&lt;</span> <span class="mf">0.05</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">✓ Bueno: Val y Test son razonablemente similares</span><span class="sh">"</span><span class="p">)</span>
<span class="k">else</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">⚠️ Advertencia: Val y Test difieren significativamente</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Puede haber sobreajuste en validation</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  💡 Mejores Prácticas en Regularización
</h2>

<h3>
  
  
  ✅ DO (Hacer)
</h3>

<ol>
<li>
<strong>Probar múltiples valores de C</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="n">C_values</span> <span class="o">=</span> <span class="p">[</span><span class="mf">0.001</span><span class="p">,</span> <span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">100</span><span class="p">,</span> <span class="mi">1000</span><span class="p">]</span>
</code></pre>

</div>



<ol>
<li>
<strong>Usar validación cruzada para mayor robustez</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="kn">from</span> <span class="n">sklearn.model_selection</span> <span class="kn">import</span> <span class="n">GridSearchCV</span>
   <span class="n">param_grid</span> <span class="o">=</span> <span class="p">{</span><span class="sh">'</span><span class="s">C</span><span class="sh">'</span><span class="p">:</span> <span class="p">[</span><span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">100</span><span class="p">]}</span>
   <span class="n">grid_search</span> <span class="o">=</span> <span class="nc">GridSearchCV</span><span class="p">(</span><span class="nc">LogisticRegression</span><span class="p">(),</span> <span class="n">param_grid</span><span class="p">,</span> <span class="n">cv</span><span class="o">=</span><span class="mi">5</span><span class="p">)</span>
</code></pre>

</div>



<ol>
<li>
<strong>Evaluar múltiples métricas</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># No solo accuracy, también precision, recall, F1
</span></code></pre>

</div>



<ol>
<li>
<strong>Verificar en test set al final</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># Usar test solo una vez, al final
</span></code></pre>

</div>



<h3>
  
  
  ❌ DON'T (No Hacer)
</h3>

<ol>
<li>
<strong>No usar validation set</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ❌ MAL: Optimizar en train
</span>   <span class="c1"># ✅ BIEN: Optimizar en validation
</span></code></pre>

</div>



<ol>
<li>
<strong>Optimizar en test set</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ❌ MAL: Elegir C basándose en test
</span>   <span class="c1"># Esto introduce data leakage
</span></code></pre>

</div>



<ol>
<li>
<strong>Ignorar la diferencia train-val</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># Si train &gt;&gt; val → overfitting
</span>   <span class="c1"># Necesitas más regularización (C menor)
</span></code></pre>

</div>



<ol>
<li>
<strong>No considerar el contexto</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># Balance entre interpretabilidad y rendimiento
</span></code></pre>

</div>



<h2>
  
  
  📝 Código Completo para Referencia
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pandas</span> <span class="k">as</span> <span class="n">pd</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">from</span> <span class="n">sklearn.model_selection</span> <span class="kn">import</span> <span class="n">train_test_split</span>
<span class="kn">from</span> <span class="n">sklearn.linear_model</span> <span class="kn">import</span> <span class="n">LogisticRegression</span>
<span class="kn">from</span> <span class="n">sklearn.feature_extraction</span> <span class="kn">import</span> <span class="n">DictVectorizer</span>
<span class="kn">from</span> <span class="n">sklearn.metrics</span> <span class="kn">import</span> <span class="n">accuracy_score</span>

<span class="c1"># 1. Preparar datos
</span><span class="n">url</span> <span class="o">=</span> <span class="sh">"</span><span class="s">https://raw.githubusercontent.com/alexeygrigorev/datasets/master/course_lead_scoring.csv</span><span class="sh">"</span>
<span class="n">df</span> <span class="o">=</span> <span class="n">pd</span><span class="p">.</span><span class="nf">read_csv</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>

<span class="n">categorical_cols</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">select_dtypes</span><span class="p">(</span><span class="n">include</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">object</span><span class="sh">'</span><span class="p">]).</span><span class="n">columns</span><span class="p">.</span><span class="nf">tolist</span><span class="p">()</span>
<span class="n">numerical_cols</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">select_dtypes</span><span class="p">(</span><span class="n">include</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">int64</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">float64</span><span class="sh">'</span><span class="p">]).</span><span class="n">columns</span><span class="p">.</span><span class="nf">tolist</span><span class="p">()</span>

<span class="n">df_clean</span> <span class="o">=</span> <span class="n">df</span><span class="p">.</span><span class="nf">copy</span><span class="p">()</span>
<span class="k">for</span> <span class="n">col</span> <span class="ow">in</span> <span class="n">categorical_cols</span><span class="p">:</span>
    <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">]</span> <span class="o">=</span> <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">].</span><span class="nf">fillna</span><span class="p">(</span><span class="sh">'</span><span class="s">NA</span><span class="sh">'</span><span class="p">)</span>
<span class="k">for</span> <span class="n">col</span> <span class="ow">in</span> <span class="n">numerical_cols</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">col</span> <span class="o">!=</span> <span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">:</span>
        <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">]</span> <span class="o">=</span> <span class="n">df_clean</span><span class="p">[</span><span class="n">col</span><span class="p">].</span><span class="nf">fillna</span><span class="p">(</span><span class="mf">0.0</span><span class="p">)</span>

<span class="c1"># 2. División
</span><span class="n">df_train_full</span><span class="p">,</span> <span class="n">df_temp</span> <span class="o">=</span> <span class="nf">train_test_split</span><span class="p">(</span><span class="n">df_clean</span><span class="p">,</span> <span class="n">test_size</span><span class="o">=</span><span class="mf">0.4</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
<span class="n">df_val</span><span class="p">,</span> <span class="n">df_test</span> <span class="o">=</span> <span class="nf">train_test_split</span><span class="p">(</span><span class="n">df_temp</span><span class="p">,</span> <span class="n">test_size</span><span class="o">=</span><span class="mf">0.5</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>

<span class="n">y_train</span> <span class="o">=</span> <span class="n">df_train_full</span><span class="p">[</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">].</span><span class="n">values</span>
<span class="n">y_val</span> <span class="o">=</span> <span class="n">df_val</span><span class="p">[</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">].</span><span class="n">values</span>

<span class="n">X_train_df</span> <span class="o">=</span> <span class="n">df_train_full</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">X_val_df</span> <span class="o">=</span> <span class="n">df_val</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>

<span class="c1"># 3. One-hot encoding
</span><span class="n">dv</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">X_train</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
<span class="n">X_val</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

<span class="c1"># 4. Probar diferentes C
</span><span class="n">C_values</span> <span class="o">=</span> <span class="p">[</span><span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">,</span> <span class="mi">100</span><span class="p">]</span>
<span class="n">results</span> <span class="o">=</span> <span class="p">{}</span>

<span class="k">for</span> <span class="n">C</span> <span class="ow">in</span> <span class="n">C_values</span><span class="p">:</span>
    <span class="n">model</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="n">C</span><span class="p">,</span> <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
    <span class="n">model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

    <span class="n">y_pred_val</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">)</span>
    <span class="n">accuracy_val</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_val</span><span class="p">)</span>
    <span class="n">accuracy_val_rounded</span> <span class="o">=</span> <span class="nf">round</span><span class="p">(</span><span class="n">accuracy_val</span><span class="p">,</span> <span class="mi">3</span><span class="p">)</span>

    <span class="n">results</span><span class="p">[</span><span class="n">C</span><span class="p">]</span> <span class="o">=</span> <span class="n">accuracy_val_rounded</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">C=</span><span class="si">{</span><span class="n">C</span><span class="si">}</span><span class="s">: accuracy = </span><span class="si">{</span><span class="n">accuracy_val_rounded</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># 5. Encontrar mejor C
</span><span class="n">max_accuracy</span> <span class="o">=</span> <span class="nf">max</span><span class="p">(</span><span class="n">results</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span>
<span class="n">best_C</span> <span class="o">=</span> <span class="nf">min</span><span class="p">([</span><span class="n">c</span> <span class="k">for</span> <span class="n">c</span><span class="p">,</span> <span class="n">acc</span> <span class="ow">in</span> <span class="n">results</span><span class="p">.</span><span class="nf">items</span><span class="p">()</span> <span class="k">if</span> <span class="n">acc</span> <span class="o">==</span> <span class="n">max_accuracy</span><span class="p">])</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Mejor C: </span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="s"> (accuracy: </span><span class="si">{</span><span class="n">max_accuracy</span><span class="si">:</span><span class="p">.</span><span class="mi">3</span><span class="n">f</span><span class="si">}</span><span class="s">)</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  🎯 Resumen del Homework Completo
</h2>

<p>Ahora tenemos todas las respuestas:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span> <span class="o">+</span> <span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">RESUMEN COMPLETO - HOMEWORK 3: CLASIFICACIÓN</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Question 1 - Moda de </span><span class="sh">'</span><span class="s">industry</span><span class="sh">'</span><span class="s">:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Respuesta: retail</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Question 2 - Mayor correlación:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Respuesta: annual_income y interaction_count</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Question 3 - Mayor Mutual Information Score:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Respuesta: employment_status</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Question 4 - Accuracy de Regresión Logística:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Respuesta: 0.85</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Question 5 - Feature menos útil:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Respuesta: employment_status</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Question 6 - Mejor valor de C:</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Respuesta: </span><span class="si">{</span><span class="n">best_C</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="sh">"</span> <span class="o">+</span> <span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ HOMEWORK COMPLETADO</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  🎓 Conclusión
</h2>

<p>En este post final aprendimos sobre <strong>regularización</strong>:</p>

<ol>
<li>✅ <strong>Qué es la regularización</strong> y por qué previene overfitting</li>
<li>✅ <strong>Cómo funciona el parámetro C</strong> en regresión logística</li>
<li>✅ <strong>Cómo encontrar el mejor C</strong> mediante búsqueda sistemática</li>
<li>✅ <strong>Cómo evaluar</strong> el modelo final en test set</li>
</ol>

<p><strong>Puntos clave</strong>:</p>

<ul>
<li>🎯 C controla la fuerza de regularización (C pequeño = más regularización)</li>
<li>📊 Siempre optimizar en validation, no en test</li>
<li>🔍 Si hay empate, elegir el C más pequeño (modelo más simple)</li>
<li>⚖️ Balance entre train y validation accuracy es crucial</li>
</ul>

<h2>
  
  
  🚀 Siguientes Pasos
</h2>

<p>¡Felicitaciones! Has completado el Homework 3 de Clasificación. Ahora puedes:</p>

<ol>
<li>✅ <strong>Experimentar</strong> con otros datasets de clasificación</li>
<li>✅ <strong>Probar</strong> otros algoritmos (Random Forest, Gradient Boosting)</li>
<li>✅ <strong>Explorar</strong> técnicas avanzadas (SMOTE para desbalance, ensemble methods)</li>
<li>✅ <strong>Aplicar</strong> lo aprendido a problemas reales de tu trabajo</li>
</ol>




<p><em>¿Qué valor de C encontraste óptimo? ¿Notaste diferencias significativas entre valores? ¡Comparte tus resultados!</em></p>

