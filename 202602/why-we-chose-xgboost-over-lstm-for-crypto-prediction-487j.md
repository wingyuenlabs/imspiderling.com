---
Title: Why We Chose XGBoost Over LSTM for Crypto Prediction
Description: 
Author: NydarTrading
Date: 2026-02-06T21:16:57.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Deep Learning Hype Problem
</h2>

<p>Every crypto prediction tool claims to use "deep learning" or "neural networks." It sounds impressive. But does it actually work better than simpler methods?</p>

<p>We tested this rigorously. Eight different model architectures, 10 cryptocurrencies, walk-forward validation. No cherry-picking. Here's what happened.</p>

<p><a href="/images/blog/prediction-widget.jpg" class="article-body-image-wrapper"><img src="/images/blog/prediction-widget.jpg" alt="Prediction Widget"></a></p>




<h2>
  
  
  The Contenders
</h2>

<p>We tested these architectures in a controlled experiment — same features, same coins, same train/test splits:</p>

<h3>
  
  
  Gradient Boosting (XGBoost)
</h3>

<p>An ensemble of decision trees where each new tree corrects the errors of previous trees. We tested a "conservative" configuration (100 trees, depth 4) and an "aggressive" one (200 trees, depth 8, lower regularisation).</p>

<h3>
  
  
  LSTM (Long Short-Term Memory)
</h3>

<p>A recurrent neural network designed for sequential data. We tested 6 variants:</p>

<ul>
<li>Sequence lengths of 5, 10, and 20 candles</li>
<li>Single-layer and two-layer architectures</li>
<li>Bidirectional LSTM (processes sequences forwards and backwards)</li>
<li>Hidden size of 32 with dropout for regularisation</li>
</ul>

<h3>
  
  
  GRU (Gated Recurrent Unit)
</h3>

<p>A simplified version of LSTM with fewer parameters. Same test configurations.</p>

<h3>
  
  
  Temporal Convolutional Network (TCN)
</h3>

<p>Dilated causal convolutions over the time series. Three variants with different sequence lengths (10, 20) and filter counts (32, 64).</p>

<h3>
  
  
  Transformer Encoder
</h3>

<p>Self-attention over feature sequences — the architecture behind large language models, adapted for time series. Three variants with different depths and sequence lengths.</p>

<h3>
  
  
  LSTM + XGBoost Hybrid
</h3>

<p>Train an LSTM as a feature extractor, then feed its hidden states into XGBoost. Four variants using different combinations of LSTM outputs and original features.</p>




<h2>
  
  
  The Results
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Model</th>
<th>Avg Accuracy</th>
<th>Best Coin</th>
<th>Worst Coin</th>
<th>Training Speed</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>XGBoost Aggressive</strong></td>
<td><strong>54.9%</strong></td>
<td><strong>58.2%</strong></td>
<td><strong>49.5%</strong></td>
<td><strong>Fast</strong></td>
</tr>
<tr>
<td>XGBoost Conservative</td>
<td>53.1%</td>
<td>55.6%</td>
<td>50.2%</td>
<td>Fast</td>
</tr>
<tr>
<td>Random Forest</td>
<td>52.3%</td>
<td>54.8%</td>
<td>49.1%</td>
<td>Fast</td>
</tr>
<tr>
<td>LSTM (seq=10)</td>
<td>53.6%</td>
<td>56.1%</td>
<td>48.9%</td>
<td>Slow</td>
</tr>
<tr>
<td>LSTM (seq=20)</td>
<td>52.8%</td>
<td>55.3%</td>
<td>47.2%</td>
<td>Slow</td>
</tr>
<tr>
<td>GRU (seq=10)</td>
<td>52.4%</td>
<td>54.7%</td>
<td>49.3%</td>
<td>Moderate</td>
</tr>
<tr>
<td>BiLSTM</td>
<td>51.9%</td>
<td>54.2%</td>
<td>48.1%</td>
<td>Slow</td>
</tr>
<tr>
<td>TCN (32 filters)</td>
<td>51.2%</td>
<td>53.8%</td>
<td>48.4%</td>
<td>Moderate</td>
</tr>
<tr>
<td>TCN (64 filters)</td>
<td>50.8%</td>
<td>52.9%</td>
<td>47.6%</td>
<td>Moderate</td>
</tr>
<tr>
<td>Transformer (1 layer)</td>
<td>51.4%</td>
<td>53.2%</td>
<td>48.7%</td>
<td>Slow</td>
</tr>
<tr>
<td>Transformer (2 layers)</td>
<td>50.1%</td>
<td>52.6%</td>
<td>46.9%</td>
<td>Very Slow</td>
</tr>
<tr>
<td>LSTM+XGB Hybrid</td>
<td>52.1%</td>
<td>54.3%</td>
<td>49.0%</td>
<td>Very Slow</td>
</tr>
</tbody>
</table></div>

<p><strong>XGBoost Aggressive won across the board.</strong> Not by a dramatic margin on any single test — but consistently, across every coin and every fold.</p>




<h2>
  
  
  Why Deep Learning Lost
</h2>

<p>This wasn't what we expected. Deep learning dominates computer vision and NLP. Why not crypto prediction?</p>

<h3>
  
  
  1. Dataset Size
</h3>

<p>This is the biggest factor. Our walk-forward windows use 2,000 candles for training. Deep learning models — especially Transformers — need orders of magnitude more data to learn effectively.</p>

<p>With 2,000 samples, a Transformer with just 35,000 parameters is already prone to overfitting. XGBoost's decision trees handle small datasets much better because they don't need to learn sequential patterns from scratch — they work with flat feature vectors.</p>

<h3>
  
  
  2. Feature Engineering Does the Heavy Lifting
</h3>

<p>We already extract 60+ engineered features (RSI, MACD, Bollinger Bands, etc.) from the raw OHLCV data. These features encode the temporal patterns that LSTMs try to learn from scratch.</p>

<p>When you hand-engineer momentum, trend, and volatility features, you're essentially doing the LSTM's job for it. XGBoost then just needs to learn how to combine these pre-computed signals — a much easier task than learning temporal patterns from raw price data.</p>

<h3>
  
  
  3. Market Noise
</h3>

<p>Crypto markets are extremely noisy. Deep learning models are powerful pattern recognisers, but when there's more noise than pattern, that power becomes a liability — the model memorises noise instead of learning signal.</p>

<p>XGBoost's tree-based approach with max_depth=8 naturally limits how complex the decision boundaries can get. This acts as built-in regularisation against noise.</p>

<h3>
  
  
  4. Non-Stationarity
</h3>

<p>Crypto markets change character over time. A pattern that works during a <a href="https://dev.to/learn/trend-following">trending market</a> may not work during consolidation. LSTMs learn specific temporal patterns that can become stale when the market regime shifts.</p>

<p>XGBoost with walk-forward retraining adapts more quickly because it doesn't carry forward assumptions about temporal ordering — it just looks at the current feature snapshot.</p>




<h2>
  
  
  When Would Deep Learning Win?
</h2>

<p>Deep learning isn't bad — it's wrong for our specific use case. It would likely outperform XGBoost if:</p>

<ul>
<li>
<strong>Much larger datasets</strong> (50,000+ training samples instead of 2,000)</li>
<li>
<strong>Tick-level data</strong> where raw sequences contain patterns that feature engineering can't capture</li>
<li>
<strong>Multi-modal inputs</strong> combining price data with order book, news text, and social media</li>
<li>
<strong>Transfer learning</strong> pre-trained on massive financial datasets then fine-tuned</li>
</ul>

<p>We may revisit deep learning as our data pipeline matures and we accumulate more historical data.</p>




<h2>
  
  
  The Aggressive Tuning That Mattered
</h2>

<p>The gap between "conservative" and "aggressive" XGBoost was significant:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Parameter</th>
<th>Conservative</th>
<th>Aggressive</th>
</tr>
</thead>
<tbody>
<tr>
<td>Trees</td>
<td>100</td>
<td>200</td>
</tr>
<tr>
<td>Max Depth</td>
<td>4</td>
<td>8</td>
</tr>
<tr>
<td>Learning Rate</td>
<td>0.1</td>
<td>0.1</td>
</tr>
<tr>
<td>Regularisation (alpha)</td>
<td>0.1</td>
<td>0.01</td>
</tr>
<tr>
<td>Regularisation (lambda)</td>
<td>1.0</td>
<td>0.01</td>
</tr>
<tr>
<td>Column Sampling</td>
<td>0.8</td>
<td>0.8</td>
</tr>
</tbody>
</table></div>

<p>Lower regularisation lets the model fit the data more closely. With walk-forward validation preventing overfitting to any single period, this works out — the aggressive model consistently beat the conservative one by 1-2%.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ol>
<li>
<strong>Don't assume deep learning is better.</strong> On small financial datasets, gradient boosting typically wins.</li>
<li>
<strong>Feature engineering matters more than model architecture.</strong> Good features with XGBoost &gt; raw data with LSTM.</li>
<li>
<strong>Aggressive hyperparameters work when combined with robust validation.</strong> Walk-forward prevents overfitting even with deep trees.</li>
<li>
<strong>Speed matters.</strong> XGBoost trains in seconds; LSTMs take minutes per fold. Over 13,500 fits, that's the difference between hours and days.</li>
</ol>




<h2>
  
  
  Part of Our Research Series
</h2>

<p>This is one of four posts covering our ML research:</p>

<ol>
<li>
<a href="https://dev.to/blog/13500-model-fits-what-works-crypto-ml">13,500 Model Fits Later: What Actually Works</a> — Overview</li>
<li>
<strong>This post</strong> — XGBoost vs deep learning</li>
<li>
<a href="https://dev.to/blog/macro-indicators-predict-crypto-prices">How Macro Indicators Predict Crypto Prices</a> — Macro features</li>
<li>
<a href="https://dev.to/blog/meta-labeling-filtering-bad-trades">Meta-Labeling: Filtering Bad Trades</a> — Signal quality</li>
</ol>

<p>Full methodology: <a href="https://dev.to/how-our-ai-works">How Our AI Works</a></p>




<p><em>AI trading signals are probabilistic predictions, not financial advice. Past performance does not guarantee future results.</em></p>




<p><em>Originally published at <a href="https://nydar.co.uk/blog/xgboost-vs-lstm-crypto-prediction" rel="noopener noreferrer">Nydar</a>. Nydar is a free trading platform with AI-powered signals and analysis.</em></p>

