---
Title: Predicting Fuel Efficiency with Tree-Based Models: A Hands-On Machine Learning Walkthrough
Description: 
Author: Kenechukwu Anoliefo
Date: 2025-11-09T21:45:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Understanding how vehicle characteristics affect fuel efficiency is a classic regression problem — and an excellent way to explore tree-based models like <strong>Decision Trees</strong>, <strong>Random Forests</strong>, and <strong>XGBoost</strong>. In this project, I analyzed a dataset of cars and built models to predict <strong>fuel efficiency (MPG)</strong> with different configurations.</p>




<h3>
  
  
  🧩 Step 1 — Data Preparation
</h3>

<p>The dataset contained various vehicle features, including:</p>

<ul>
<li><code>vehicle_weight</code></li>
<li><code>engine_displacement</code></li>
<li><code>horsepower</code></li>
<li><code>acceleration</code></li>
<li><code>model_year</code></li>
<li><code>origin</code></li>
<li><code>fuel_type</code></li>
</ul>

<p>To ensure data consistency, all <strong>missing values were filled with zeros</strong>.<br>
Then I performed a <strong>train/validation/test split (60%/20%/20%)</strong>, using a <code>random_state=1</code> for reproducibility.</p>

<p>Next, I used <strong>DictVectorizer(sparse=True)</strong> to convert categorical and numerical features into a format suitable for scikit-learn models.</p>


<h3>
  
  
  🌳 Step 2 — Decision Tree Regressor
</h3>

<p>I began with a <strong>Decision Tree Regressor</strong> with <code>max_depth=1</code>.<br>
This simple tree helps visualize which feature the model uses first to split the data — effectively revealing the most influential variable in predicting MPG.</p>

<p><strong>Result:</strong><br>
The feature used for splitting was <strong><code>model_year</code></strong>, showing that newer vehicles tend to have different fuel efficiencies compared to older models.</p>


<h3>
  
  
  🌲 Step 3 — Random Forest Model
</h3>

<p>Next, I trained a <strong>Random Forest Regressor</strong> with the parameters:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">n_estimators</span><span class="o">=</span><span class="mi">10</span>  
<span class="n">random_state</span><span class="o">=</span><span class="mi">1</span>  
<span class="n">n_jobs</span><span class="o">=-</span><span class="mi">1</span>
</code></pre>

</div>



<p>Random forests aggregate multiple decision trees to reduce overfitting and improve accuracy.</p>

<p><strong>Validation RMSE:</strong> ≈ <strong>4.5</strong></p>

<p>This confirmed the model could capture relationships between engine specs and fuel efficiency quite effectively.</p>




<h3>
  
  
  ⚙️ Step 4 — Tuning n_estimators
</h3>

<p>To see how the number of trees affects performance, I trained models with <strong>n_estimators ranging from 10 to 200</strong> (step = 10).<br>
After monitoring RMSE, I observed the improvement plateaued after around <strong>80 estimators</strong>, indicating that adding more trees didn’t significantly enhance accuracy.</p>


<h3>
  
  
  🌾 Step 5 — Tuning max_depth
</h3>

<p>I then compared four values of <code>max_depth</code> — [10, 15, 20, 25] — each with increasing <code>n_estimators</code> from 10 to 200.<br>
The best mean RMSE occurred at <strong>max_depth = 20</strong>, which struck the right balance between bias and variance.</p>


<h3>
  
  
  🔍 Step 6 — Feature Importance
</h3>

<p>Random Forests provide an excellent built-in mechanism for feature importance.<br>
Training the model with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">n_estimators</span><span class="o">=</span><span class="mi">10</span><span class="p">,</span> <span class="n">max_depth</span><span class="o">=</span><span class="mi">20</span><span class="p">,</span> <span class="n">random_state</span><span class="o">=</span><span class="mi">1</span>
</code></pre>

</div>



<p>I found the most influential feature for predicting fuel efficiency to be <strong><code>engine_displacement</code></strong>, followed by <code>vehicle_weight</code> and <code>horsepower</code>.</p>

<p>This aligns well with domain knowledge — larger engines and heavier vehicles typically consume more fuel.</p>




<h3>
  
  
  ⚡ Step 7 — XGBoost Experiments
</h3>

<p>Finally, I trained an <strong>XGBoost regressor</strong>, tuning the <code>eta</code> (learning rate) parameter between <strong>0.3</strong> and <strong>0.1</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">xgb_params</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">'</span><span class="s">eta</span><span class="sh">'</span><span class="p">:</span> <span class="p">[</span><span class="mf">0.3</span> <span class="ow">or</span> <span class="mf">0.1</span><span class="p">],</span>
    <span class="sh">'</span><span class="s">max_depth</span><span class="sh">'</span><span class="p">:</span> <span class="mi">6</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">objective</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">reg:squarederror</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">nthread</span><span class="sh">'</span><span class="p">:</span> <span class="mi">8</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">seed</span><span class="sh">'</span><span class="p">:</span> <span class="mi">1</span>
<span class="p">}</span>
</code></pre>

</div>



<p>After 100 training rounds, the model with <strong>eta = 0.1</strong> delivered slightly better RMSE on the validation set — confirming that a smaller learning rate can yield smoother, more generalized models.</p>




<h3>
  
  
  🎯 Key Takeaways
</h3>

<ul>
<li>
<strong><code>model_year</code></strong> strongly influences fuel efficiency in modern cars.</li>
<li>
<strong>Random Forests</strong> with <code>n_estimators ≈ 80</code> and <code>max_depth=20</code> gave the most balanced performance.</li>
<li>
<strong>Engine displacement</strong> emerged as the most important predictor of MPG.</li>
<li>
<strong>XGBoost with a lower learning rate (eta=0.1)</strong> achieved the best validation score.</li>
</ul>




<h3>
  
  
  💡 Final Thoughts
</h3>

<p>This project demonstrates how <strong>iterative experimentation</strong> with tree-based models reveals both predictive strength and interpretability.<br>
From simple decision trees to tuned XGBoost models, each step provided insight into how vehicle characteristics drive fuel efficiency — and how model parameters affect performance.</p>

<p>If you’re learning machine learning, projects like this are perfect for mastering <strong>feature engineering, evaluation metrics, and model tuning.</strong></p>

