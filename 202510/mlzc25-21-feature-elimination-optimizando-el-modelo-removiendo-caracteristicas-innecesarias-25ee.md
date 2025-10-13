---
Title: MLZC25-21. Feature Elimination: Optimizando el Modelo Removiendo Características Innecesarias
Description: 
Author: Jesus Oviedo Riquelme
Date: 2025-10-13T22:10:44.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<p><strong>🎯 Objetivo del Post</strong>: Aprenderás la técnica de eliminación de características (feature elimination) para identificar variables que puedes remover sin afectar (o incluso mejorando) el rendimiento del modelo, creando modelos más simples y eficientes.</p>
</blockquote>

<h2>
  
  
  🤔 ¿Por Qué Eliminar Features?
</h2>

<p>Tener <strong>más features no siempre es mejor</strong>. Principio de parsimonia (Navaja de Ockham):</p>

<blockquote>
<p>"Entre dos modelos con rendimiento similar, el más simple es preferible"</p>
</blockquote>

<h3>
  
  
  Problemas de Tener Muchas Features
</h3>

<p>❌ <strong>Overfitting</strong>: El modelo aprende ruido en lugar de patrones reales<br>
❌ <strong>Tiempo de entrenamiento</strong>: Más lento con más features<br>
❌ <strong>Complejidad</strong>: Más difícil de entender e interpretar<br>
❌ <strong>Costo computacional</strong>: Más memoria y recursos<br>
❌ <strong>Maldición de la dimensionalidad</strong>: Con muchas dimensiones, los datos se vuelven "esparcidos"</p>
<h3>
  
  
  Ventajas de un Modelo Simple
</h3>

<p>✅ <strong>Generaliza mejor</strong>: Menos propenso a overfitting<br>
✅ <strong>Más rápido</strong>: Entrena y predice más rápido<br>
✅ <strong>Más interpretable</strong>: Fácil de explicar<br>
✅ <strong>Menos recursos</strong>: Menor costo computacional<br>
✅ <strong>Más robusto</strong>: Menos sensible a ruido</p>
<h2>
  
  
  🔍 ¿Qué es Feature Elimination?
</h2>

<p><strong>Feature Elimination</strong> es el proceso sistemático de:</p>

<ol>
<li>Entrenar un modelo con todas las features</li>
<li>Remover una feature a la vez</li>
<li>Evaluar el impacto en el rendimiento</li>
<li>Identificar features que aportan poco o nada</li>
</ol>
<h3>
  
  
  Metodología
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Accuracy Baseline (todas las features) = 0.85

Feature 1 removida → Accuracy = 0.84  (↓0.01)
Feature 2 removida → Accuracy = 0.85  (↔ 0.00)  ← Candidata a eliminar
Feature 3 removida → Accuracy = 0.82  (↓0.03)
Feature 4 removida → Accuracy = 0.85  (↔ 0.00)  ← Candidata a eliminar
...
</code></pre>

</div>

<h2>
  
  
  💻 Implementación Paso a Paso
</h2>
<h3>
  
  
  Paso 1: Entrenar Modelo Baseline
</h3>

<p>Primero, necesitamos nuestro modelo de referencia con todas las features:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">pandas</span> <span class="k">as</span> <span class="n">pd</span>
<span class="kn">import</span> <span class="n">numpy</span> <span class="k">as</span> <span class="n">np</span>
<span class="kn">from</span> <span class="n">sklearn.model_selection</span> <span class="kn">import</span> <span class="n">train_test_split</span>
<span class="kn">from</span> <span class="n">sklearn.linear_model</span> <span class="kn">import</span> <span class="n">LogisticRegression</span>
<span class="kn">from</span> <span class="n">sklearn.feature_extraction</span> <span class="kn">import</span> <span class="n">DictVectorizer</span>
<span class="kn">from</span> <span class="n">sklearn.metrics</span> <span class="kn">import</span> <span class="n">accuracy_score</span>

<span class="c1"># Cargar y preparar datos (código del post anterior)
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

<span class="n">X_train_df</span> <span class="o">=</span> <span class="n">df_train_full</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="n">X_val_df</span> <span class="o">=</span> <span class="n">df_val</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="c1"># One-hot encoding
</span><span class="n">dv</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">X_train</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
<span class="n">X_val</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

<span class="c1"># Entrenar modelo baseline
</span><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">MODELO BASELINE (CON TODAS LAS FEATURES)</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">60</span><span class="p">)</span>

<span class="n">model_baseline</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span>
    <span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> 
    <span class="n">C</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span> 
    <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> 
    <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span>
<span class="p">)</span>
<span class="n">model_baseline</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

<span class="c1"># Evaluar
</span><span class="n">y_pred_baseline</span> <span class="o">=</span> <span class="n">model_baseline</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">)</span>
<span class="n">accuracy_baseline</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_baseline</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Accuracy baseline: </span><span class="si">{</span><span class="n">accuracy_baseline</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Número de features: </span><span class="si">{</span><span class="n">X_train</span><span class="p">.</span><span class="n">shape</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">✅ Modelo baseline entrenado</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Paso 2: Feature Elimination - Prueba Individual
</h3>

<p>Ahora probamos remover cada feature y ver cómo afecta al accuracy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">FEATURE ELIMINATION - ANÁLISIS INDIVIDUAL</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="c1"># Features a probar (según el homework)
</span><span class="n">features_to_test</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">industry</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">employment_status</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">lead_score</span><span class="sh">'</span><span class="p">]</span>

<span class="c1"># Diccionario para guardar resultados
</span><span class="n">feature_importance</span> <span class="o">=</span> <span class="p">{}</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="si">{</span><span class="sh">'</span><span class="s">Feature</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Acc sin feature</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">15</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Diferencia</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">15</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Impacto</span><span class="sh">'</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="k">for</span> <span class="n">feature</span> <span class="ow">in</span> <span class="n">features_to_test</span><span class="p">:</span>
    <span class="c1"># Crear dataframes SIN la feature
</span>    <span class="n">X_train_no_feat</span> <span class="o">=</span> <span class="n">X_train_df</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">ignore</span><span class="sh">'</span><span class="p">)</span>
    <span class="n">X_val_no_feat</span> <span class="o">=</span> <span class="n">X_val_df</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">ignore</span><span class="sh">'</span><span class="p">)</span>

    <span class="c1"># Verificar si la feature existe
</span>    <span class="k">if</span> <span class="n">feature</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">X_train_df</span><span class="p">.</span><span class="n">columns</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">⚠️ Feature </span><span class="sh">'</span><span class="si">{</span><span class="n">feature</span><span class="si">}</span><span class="sh">'</span><span class="s"> no encontrada</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">continue</span>

    <span class="c1"># One-hot encoding sin la feature
</span>    <span class="n">dv_no_feat</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
    <span class="n">X_train_transformed</span> <span class="o">=</span> <span class="n">dv_no_feat</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_no_feat</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
    <span class="n">X_val_transformed</span> <span class="o">=</span> <span class="n">dv_no_feat</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_no_feat</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

    <span class="c1"># Entrenar modelo sin la feature
</span>    <span class="n">model_no_feat</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span>
        <span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> 
        <span class="n">C</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span> 
        <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> 
        <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span>
    <span class="p">)</span>
    <span class="n">model_no_feat</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train_transformed</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

    <span class="c1"># Evaluar
</span>    <span class="n">y_pred_no_feat</span> <span class="o">=</span> <span class="n">model_no_feat</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val_transformed</span><span class="p">)</span>
    <span class="n">accuracy_no_feat</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_no_feat</span><span class="p">)</span>

    <span class="c1"># Calcular diferencia
</span>    <span class="n">diff</span> <span class="o">=</span> <span class="n">accuracy_baseline</span> <span class="o">-</span> <span class="n">accuracy_no_feat</span>
    <span class="n">feature_importance</span><span class="p">[</span><span class="n">feature</span><span class="p">]</span> <span class="o">=</span> <span class="n">diff</span>

    <span class="c1"># Determinar impacto
</span>    <span class="k">if</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="n">impact</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Mejora ↑</span><span class="sh">"</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.01</span><span class="p">:</span>
        <span class="n">impact</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Mínimo →</span><span class="sh">"</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.03</span><span class="p">:</span>
        <span class="n">impact</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Bajo ↓</span><span class="sh">"</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.05</span><span class="p">:</span>
        <span class="n">impact</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Moderado ↓↓</span><span class="sh">"</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">impact</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Alto ↓↓↓</span><span class="sh">"</span>

    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">feature</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">accuracy_no_feat</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">diff</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">15.6</span><span class="n">f</span><span class="si">}</span><span class="s">  </span><span class="si">{</span><span class="n">impact</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Salida esperada</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FEATURE ELIMINATION - ANÁLISIS INDIVIDUAL
======================================================================

Feature                  Acc sin feature      Diferencia  Impacto
----------------------------------------------------------------------
industry                        0.849315        0.000000  Mínimo →
employment_status               0.852740       -0.003425  Mejora ↑
lead_score                      0.835616        0.013699  Bajo ↓
----------------------------------------------------------------------
</code></pre>

</div>



<h3>
  
  
  Paso 3: Identificar Feature Menos Útil (Question 5)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">QUESTION 5: FEATURE CON MENOR IMPACTO</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="c1"># Encontrar la feature con MENOR diferencia (puede ser negativa)
</span><span class="n">min_diff_feature</span> <span class="o">=</span> <span class="nf">min</span><span class="p">(</span><span class="n">feature_importance</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="n">feature_importance</span><span class="p">.</span><span class="n">get</span><span class="p">)</span>
<span class="n">min_diff</span> <span class="o">=</span> <span class="n">feature_importance</span><span class="p">[</span><span class="n">min_diff_feature</span><span class="p">]</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Ranking por impacto al remover:</span><span class="sh">"</span><span class="p">)</span>
<span class="k">for</span> <span class="n">idx</span><span class="p">,</span> <span class="p">(</span><span class="n">feat</span><span class="p">,</span> <span class="n">diff</span><span class="p">)</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="nf">sorted</span><span class="p">(</span><span class="n">feature_importance</span><span class="p">.</span><span class="nf">items</span><span class="p">(),</span> 
                                          <span class="n">key</span><span class="o">=</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="p">[</span><span class="mi">1</span><span class="p">]),</span> <span class="mi">1</span><span class="p">):</span>
    <span class="n">marker</span> <span class="o">=</span> <span class="sh">"</span><span class="s"> ← MENOR IMPACTO</span><span class="sh">"</span> <span class="k">if</span> <span class="n">feat</span> <span class="o">==</span> <span class="n">min_diff_feature</span> <span class="k">else</span> <span class="sh">""</span>
    <span class="n">direction</span> <span class="o">=</span> <span class="sh">"</span><span class="s">↑ Mejora</span><span class="sh">"</span> <span class="k">if</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="k">else</span> <span class="sh">"</span><span class="s">↓ Empeora</span><span class="sh">"</span> <span class="k">if</span> <span class="n">diff</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="k">else</span> <span class="sh">"</span><span class="s">→ Sin cambio</span><span class="sh">"</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">idx</span><span class="si">}</span><span class="s">. </span><span class="si">{</span><span class="n">feat</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">25</span><span class="si">}</span><span class="s"> : </span><span class="si">{</span><span class="n">diff</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.6</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">direction</span><span class="si">}{</span><span class="n">marker</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">🎯 Feature con MENOR diferencia (más prescindible):</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Feature: </span><span class="sh">'</span><span class="si">{</span><span class="n">min_diff_feature</span><span class="si">}</span><span class="sh">'"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   Diferencia: </span><span class="si">{</span><span class="n">min_diff</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">min_diff</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   ✅ Removerla MEJORA el accuracy en </span><span class="si">{</span><span class="nf">abs</span><span class="p">(</span><span class="n">min_diff</span><span class="p">)</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="k">elif</span> <span class="n">min_diff</span> <span class="o">&lt;</span> <span class="mf">0.01</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   ✅ Removerla tiene impacto MÍNIMO</span><span class="sh">"</span><span class="p">)</span>
<span class="k">else</span><span class="p">:</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">   ⚠️ Removerla empeora el accuracy en </span><span class="si">{</span><span class="n">min_diff</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">✅ RESPUESTA QUESTION 5: </span><span class="si">{</span><span class="n">min_diff_feature</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Interpretación de los Resultados
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">INTERPRETACIÓN DETALLADA</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="k">for</span> <span class="n">feature</span><span class="p">,</span> <span class="n">diff</span> <span class="ow">in</span> <span class="n">feature_importance</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="si">{</span><span class="n">feature</span><span class="si">}</span><span class="s">:</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Diferencia: </span><span class="si">{</span><span class="n">diff</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  ✅ Remover esta feature MEJORA el modelo</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Razón: Probablemente añadía ruido o correlación espuria</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.01</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  ✅ Esta feature es PRESCINDIBLE</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Razón: No aporta información significativa</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">elif</span> <span class="n">diff</span> <span class="o">&lt;</span> <span class="mf">0.03</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  ⚠️ Esta feature aporta información LIMITADA</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Razón: Tiene algo de valor pero no crítica</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  ❌ Esta feature es IMPORTANTE</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">  Razón: Removerla afecta significativamente el rendimiento</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  📊 Análisis Más Profundo
</h2>

<h3>
  
  
  Comparar Múltiples Métricas
</h3>

<p>No solo accuracy, también podemos ver otras métricas:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">sklearn.metrics</span> <span class="kn">import</span> <span class="n">precision_score</span><span class="p">,</span> <span class="n">recall_score</span><span class="p">,</span> <span class="n">f1_score</span>

<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">ANÁLISIS MULTI-MÉTRICA</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">80</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="si">{</span><span class="sh">'</span><span class="s">Feature</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">20</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Accuracy</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">10</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Precision</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">10</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">Recall</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">10</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="sh">'</span><span class="s">F1-Score</span><span class="sh">'</span><span class="si">:</span><span class="o">&gt;</span><span class="mi">10</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">-</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">80</span><span class="p">)</span>

<span class="c1"># Baseline
</span><span class="n">y_pred_baseline</span> <span class="o">=</span> <span class="n">model_baseline</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="sh">'</span><span class="s">Baseline (todas)</span><span class="sh">'</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">20</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_baseline</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">precision_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_baseline</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">recall_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_baseline</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
      <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">f1_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_baseline</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Sin cada feature
</span><span class="k">for</span> <span class="n">feature</span> <span class="ow">in</span> <span class="n">features_to_test</span><span class="p">:</span>
    <span class="n">X_train_no_feat</span> <span class="o">=</span> <span class="n">X_train_df</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">ignore</span><span class="sh">'</span><span class="p">)</span>
    <span class="n">X_val_no_feat</span> <span class="o">=</span> <span class="n">X_val_df</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">ignore</span><span class="sh">'</span><span class="p">)</span>

    <span class="n">dv_temp</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
    <span class="n">X_train_temp</span> <span class="o">=</span> <span class="n">dv_temp</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_no_feat</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
    <span class="n">X_val_temp</span> <span class="o">=</span> <span class="n">dv_temp</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_no_feat</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

    <span class="n">model_temp</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span> 
                                     <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
    <span class="n">model_temp</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train_temp</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
    <span class="n">y_pred_temp</span> <span class="o">=</span> <span class="n">model_temp</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val_temp</span><span class="p">)</span>

    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="sh">'</span><span class="s">Sin </span><span class="sh">'</span> <span class="o">+</span> <span class="n">feature</span><span class="si">:</span><span class="o">&lt;</span><span class="mi">20</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
          <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_temp</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
          <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">precision_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_temp</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
          <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">recall_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_temp</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="s"> </span><span class="sh">"</span>
          <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="nf">f1_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_temp</span><span class="p">)</span><span class="si">:</span><span class="o">&gt;</span><span class="mf">10.4</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Visualizar el Impacto
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">matplotlib.pyplot</span> <span class="k">as</span> <span class="n">plt</span>

<span class="c1"># Preparar datos para visualización
</span><span class="n">features</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">feature_importance</span><span class="p">.</span><span class="nf">keys</span><span class="p">())</span>
<span class="n">differences</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">feature_importance</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span>

<span class="c1"># Crear gráfico
</span><span class="n">plt</span><span class="p">.</span><span class="nf">figure</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span> <span class="mi">6</span><span class="p">))</span>

<span class="n">colors</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">green</span><span class="sh">'</span> <span class="k">if</span> <span class="n">d</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="k">else</span> <span class="sh">'</span><span class="s">red</span><span class="sh">'</span> <span class="k">if</span> <span class="n">d</span> <span class="o">&gt;</span> <span class="mf">0.01</span> <span class="k">else</span> <span class="sh">'</span><span class="s">orange</span><span class="sh">'</span> 
          <span class="k">for</span> <span class="n">d</span> <span class="ow">in</span> <span class="n">differences</span><span class="p">]</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">barh</span><span class="p">(</span><span class="n">features</span><span class="p">,</span> <span class="n">differences</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="n">colors</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.7</span><span class="p">)</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">xlabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Diferencia en Accuracy (Baseline - Sin Feature)</span><span class="sh">'</span><span class="p">,</span> 
           <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">ylabel</span><span class="p">(</span><span class="sh">'</span><span class="s">Feature</span><span class="sh">'</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">title</span><span class="p">(</span><span class="sh">'</span><span class="s">Impacto de Remover Cada Feature</span><span class="sh">'</span><span class="p">,</span> 
          <span class="n">fontsize</span><span class="o">=</span><span class="mi">14</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">axvline</span><span class="p">(</span><span class="n">x</span><span class="o">=</span><span class="mi">0</span><span class="p">,</span> <span class="n">color</span><span class="o">=</span><span class="sh">'</span><span class="s">black</span><span class="sh">'</span><span class="p">,</span> <span class="n">linestyle</span><span class="o">=</span><span class="sh">'</span><span class="s">--</span><span class="sh">'</span><span class="p">,</span> <span class="n">linewidth</span><span class="o">=</span><span class="mi">1</span><span class="p">)</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">grid</span><span class="p">(</span><span class="n">axis</span><span class="o">=</span><span class="sh">'</span><span class="s">x</span><span class="sh">'</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.3</span><span class="p">)</span>

<span class="c1"># Añadir anotaciones
</span><span class="k">for</span> <span class="n">i</span><span class="p">,</span> <span class="p">(</span><span class="n">feat</span><span class="p">,</span> <span class="n">diff</span><span class="p">)</span> <span class="ow">in</span> <span class="nf">enumerate</span><span class="p">(</span><span class="nf">zip</span><span class="p">(</span><span class="n">features</span><span class="p">,</span> <span class="n">differences</span><span class="p">)):</span>
    <span class="n">x_pos</span> <span class="o">=</span> <span class="n">diff</span> <span class="o">+</span> <span class="mf">0.001</span> <span class="k">if</span> <span class="n">diff</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="k">else</span> <span class="n">diff</span> <span class="o">-</span> <span class="mf">0.001</span>
    <span class="n">ha</span> <span class="o">=</span> <span class="sh">'</span><span class="s">left</span><span class="sh">'</span> <span class="k">if</span> <span class="n">diff</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="k">else</span> <span class="sh">'</span><span class="s">right</span><span class="sh">'</span>
    <span class="n">plt</span><span class="p">.</span><span class="nf">text</span><span class="p">(</span><span class="n">x_pos</span><span class="p">,</span> <span class="n">i</span><span class="p">,</span> <span class="sa">f</span><span class="sh">'</span><span class="si">{</span><span class="n">diff</span><span class="si">:</span><span class="p">.</span><span class="mi">4</span><span class="n">f</span><span class="si">}</span><span class="sh">'</span><span class="p">,</span> <span class="n">va</span><span class="o">=</span><span class="sh">'</span><span class="s">center</span><span class="sh">'</span><span class="p">,</span> <span class="n">ha</span><span class="o">=</span><span class="n">ha</span><span class="p">,</span> <span class="n">fontweight</span><span class="o">=</span><span class="sh">'</span><span class="s">bold</span><span class="sh">'</span><span class="p">)</span>

<span class="n">plt</span><span class="p">.</span><span class="nf">tight_layout</span><span class="p">()</span>
<span class="n">plt</span><span class="p">.</span><span class="nf">show</span><span class="p">()</span>
</code></pre>

</div>



<h2>
  
  
  🔄 Feature Elimination Recursivo
</h2>

<p>Para un análisis más completo, podríamos eliminar features de forma recursiva:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="se">\n</span><span class="s">FEATURE ELIMINATION RECURSIVO (RFE)</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">70</span><span class="p">)</span>

<span class="kn">from</span> <span class="n">sklearn.feature_selection</span> <span class="kn">import</span> <span class="n">RFE</span>

<span class="c1"># Usar RFE de sklearn
</span><span class="n">model_rfe</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span> 
                                <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>

<span class="c1"># Seleccionar top N features
</span><span class="n">n_features_to_select</span> <span class="o">=</span> <span class="mi">30</span>  <span class="c1"># Por ejemplo
</span>
<span class="n">rfe</span> <span class="o">=</span> <span class="nc">RFE</span><span class="p">(</span><span class="n">estimator</span><span class="o">=</span><span class="n">model_rfe</span><span class="p">,</span> <span class="n">n_features_to_select</span><span class="o">=</span><span class="n">n_features_to_select</span><span class="p">)</span>
<span class="n">rfe</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>

<span class="c1"># Evaluar
</span><span class="n">y_pred_rfe</span> <span class="o">=</span> <span class="n">rfe</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">)</span>
<span class="n">accuracy_rfe</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">y_pred_rfe</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Accuracy con RFE (</span><span class="si">{</span><span class="n">n_features_to_select</span><span class="si">}</span><span class="s"> features): </span><span class="si">{</span><span class="n">accuracy_rfe</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Accuracy baseline (</span><span class="si">{</span><span class="n">X_train</span><span class="p">.</span><span class="n">shape</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span><span class="si">}</span><span class="s"> features): </span><span class="si">{</span><span class="n">accuracy_baseline</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Diferencia: </span><span class="si">{</span><span class="n">accuracy_rfe</span> <span class="o">-</span> <span class="n">accuracy_baseline</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># Ver qué features fueron seleccionadas
</span><span class="n">feature_names</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">get_feature_names_out</span><span class="p">()</span>
<span class="n">selected_features</span> <span class="o">=</span> <span class="p">[</span><span class="n">feat</span> <span class="k">for</span> <span class="n">feat</span><span class="p">,</span> <span class="n">selected</span> <span class="ow">in</span> <span class="nf">zip</span><span class="p">(</span><span class="n">feature_names</span><span class="p">,</span> <span class="n">rfe</span><span class="p">.</span><span class="n">support_</span><span class="p">)</span> 
                    <span class="k">if</span> <span class="n">selected</span><span class="p">]</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Features seleccionadas: </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">selected_features</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  💡 Mejores Prácticas en Feature Elimination
</h2>

<h3>
  
  
  ✅ DO (Hacer)
</h3>

<ol>
<li>
<strong>Usar conjunto de validación separado</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ✅ BIEN: Evaluar en validation
</span>   <span class="n">accuracy_val</span> <span class="o">=</span> <span class="n">model</span><span class="p">.</span><span class="nf">score</span><span class="p">(</span><span class="n">X_val</span><span class="p">,</span> <span class="n">y_val</span><span class="p">)</span>
</code></pre>

</div>



<ol>
<li>
<strong>Considerar múltiples métricas</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># No solo accuracy, también precision, recall, F1
</span></code></pre>

</div>



<ol>
<li>
<strong>Probar eliminación combinada</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># Remover 2-3 features menos importantes juntas
</span></code></pre>

</div>



<ol>
<li>
<strong>Validar en conjunto de test</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># Después de seleccionar features, validar en test
</span></code></pre>

</div>



<h3>
  
  
  ❌ DON'T (No Hacer)
</h3>

<ol>
<li>
<strong>Eliminar sin analizar</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ❌ MAL: Eliminar aleatoriamente
</span></code></pre>

</div>



<ol>
<li>
<strong>Usar solo train para evaluar</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ❌ MAL: accuracy_train (puede ser engañoso)
</span></code></pre>

</div>



<ol>
<li>
<strong>Eliminar demasiadas features</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ❌ Perder información valiosa
</span></code></pre>

</div>



<ol>
<li>
<strong>Ignorar el contexto del negocio</strong>
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>   <span class="c1"># ❌ Algunas features son importantes por razones de negocio
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

<span class="n">X_train_df</span> <span class="o">=</span> <span class="n">df_train_full</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
<span class="n">X_val_df</span> <span class="o">=</span> <span class="n">df_val</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="sh">'</span><span class="s">converted</span><span class="sh">'</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">).</span><span class="nf">reset_index</span><span class="p">(</span><span class="n">drop</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="c1"># 3. Modelo baseline
</span><span class="n">dv</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
<span class="n">X_train</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
<span class="n">X_val</span> <span class="o">=</span> <span class="n">dv</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_df</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

<span class="n">model</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span> <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
<span class="n">model</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
<span class="n">accuracy_baseline</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">model</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val</span><span class="p">))</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Baseline accuracy: </span><span class="si">{</span><span class="n">accuracy_baseline</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># 4. Feature elimination
</span><span class="n">features_to_test</span> <span class="o">=</span> <span class="p">[</span><span class="sh">'</span><span class="s">industry</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">employment_status</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">lead_score</span><span class="sh">'</span><span class="p">]</span>
<span class="n">feature_importance</span> <span class="o">=</span> <span class="p">{}</span>

<span class="k">for</span> <span class="n">feature</span> <span class="ow">in</span> <span class="n">features_to_test</span><span class="p">:</span>
    <span class="n">X_train_no_feat</span> <span class="o">=</span> <span class="n">X_train_df</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">ignore</span><span class="sh">'</span><span class="p">)</span>
    <span class="n">X_val_no_feat</span> <span class="o">=</span> <span class="n">X_val_df</span><span class="p">.</span><span class="nf">drop</span><span class="p">(</span><span class="n">feature</span><span class="p">,</span> <span class="n">axis</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">errors</span><span class="o">=</span><span class="sh">'</span><span class="s">ignore</span><span class="sh">'</span><span class="p">)</span>

    <span class="n">dv_no_feat</span> <span class="o">=</span> <span class="nc">DictVectorizer</span><span class="p">(</span><span class="n">sparse</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>
    <span class="n">X_train_temp</span> <span class="o">=</span> <span class="n">dv_no_feat</span><span class="p">.</span><span class="nf">fit_transform</span><span class="p">(</span><span class="n">X_train_no_feat</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>
    <span class="n">X_val_temp</span> <span class="o">=</span> <span class="n">dv_no_feat</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="n">X_val_no_feat</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">(</span><span class="n">orient</span><span class="o">=</span><span class="sh">'</span><span class="s">records</span><span class="sh">'</span><span class="p">))</span>

    <span class="n">model_temp</span> <span class="o">=</span> <span class="nc">LogisticRegression</span><span class="p">(</span><span class="n">solver</span><span class="o">=</span><span class="sh">'</span><span class="s">liblinear</span><span class="sh">'</span><span class="p">,</span> <span class="n">C</span><span class="o">=</span><span class="mf">1.0</span><span class="p">,</span> 
                                     <span class="n">max_iter</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">42</span><span class="p">)</span>
    <span class="n">model_temp</span><span class="p">.</span><span class="nf">fit</span><span class="p">(</span><span class="n">X_train_temp</span><span class="p">,</span> <span class="n">y_train</span><span class="p">)</span>
    <span class="n">accuracy_temp</span> <span class="o">=</span> <span class="nf">accuracy_score</span><span class="p">(</span><span class="n">y_val</span><span class="p">,</span> <span class="n">model_temp</span><span class="p">.</span><span class="nf">predict</span><span class="p">(</span><span class="n">X_val_temp</span><span class="p">))</span>

    <span class="n">diff</span> <span class="o">=</span> <span class="n">accuracy_baseline</span> <span class="o">-</span> <span class="n">accuracy_temp</span>
    <span class="n">feature_importance</span><span class="p">[</span><span class="n">feature</span><span class="p">]</span> <span class="o">=</span> <span class="n">diff</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">feature</span><span class="si">}</span><span class="s">: diff = </span><span class="si">{</span><span class="n">diff</span><span class="si">:</span><span class="p">.</span><span class="mi">6</span><span class="n">f</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># 5. Identificar menos útil
</span><span class="n">min_feat</span> <span class="o">=</span> <span class="nf">min</span><span class="p">(</span><span class="n">feature_importance</span><span class="p">,</span> <span class="n">key</span><span class="o">=</span><span class="n">feature_importance</span><span class="p">.</span><span class="n">get</span><span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="se">\n</span><span class="s">Menos útil: </span><span class="si">{</span><span class="n">min_feat</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  🎯 Conclusión
</h2>

<p>El <strong>Feature Elimination</strong> nos permite:</p>

<ol>
<li>✅ <strong>Simplificar modelos</strong> sin perder rendimiento</li>
<li>✅ <strong>Identificar features prescindibles</strong>
</li>
<li>✅ <strong>Mejorar interpretabilidad</strong>
</li>
<li>✅ <strong>Reducir complejidad computacional</strong>
</li>
</ol>

<p><strong>Puntos clave</strong>:</p>

<ul>
<li>🎯 No todas las features son igualmente importantes</li>
<li>📊 Remover features puede incluso <strong>mejorar</strong> el modelo</li>
<li>🔍 Siempre evaluar en conjunto de validación</li>
<li>⚖️ Balancear simplicidad vs. rendimiento</li>
</ul>

<p>En el próximo y último post (MLZC25-22), exploraremos la <strong>regularización</strong> ajustando el parámetro C para encontrar el modelo óptimo que generalice mejor.</p>




<p><em>¿Qué features encontraste como menos útiles? ¿Intentaste remover múltiples features a la vez?</em></p>

