---
Title: Adding Bayesian Ensemble + Monte Carlo to an NPB Prediction App
Description: 
Author: YMori
Date: 2026-03-23T21:48:12.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>I've been running a personal NPB (Japanese pro baseball) prediction app:</p>

<ul>
<li>
<strong>Dashboard</strong>: <a href="https://npb-prediction.streamlit.app/" rel="noopener noreferrer">npb-prediction.streamlit.app</a>
</li>
<li>
<strong>GitHub</strong>: <a href="https://github.com/yasumorishima/npb-prediction" rel="noopener noreferrer">npb-prediction</a>
</li>
</ul>

<p>It used Marcel projections (3-year weighted average) and ML (XGBoost/LightGBM). Decent, but I wanted better accuracy. After adding Bayesian corrections, the predicted standings changed significantly.</p>

<h2>
  
  
  Terms
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Term</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Marcel</strong></td>
<td>Predict next year from weighted average of past 3 years</td>
</tr>
<tr>
<td><strong>Bayesian</strong></td>
<td>Combine prior knowledge with data. Gives uncertainty estimates</td>
</tr>
<tr>
<td><strong>CI</strong></td>
<td>Credible interval — range where the true value falls with 80%/95% probability</td>
</tr>
<tr>
<td><strong>OPS</strong></td>
<td>On-base + Slugging. Overall batting metric</td>
</tr>
<tr>
<td><strong>ERA</strong></td>
<td>Earned Run Average. Runs allowed per 9 innings</td>
</tr>
<tr>
<td><strong>MAE</strong></td>
<td>Mean Absolute Error. Average prediction miss. Lower = better</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Problems with the Previous Approach
</h2>

<h3>
  
  
  Problem 1: All Foreign Players Treated as "Average"
</h3>

<p>Marcel needs 3 years of NPB data. First-year foreign players have none, so all 24 of them were treated as league-average. Dalbec (Giants, .355 wOBA in MLB) and Hummel (BayStars, .240 wOBA) were calculated identically.</p>

<h3>
  
  
  Problem 2: Skill Metrics Ignored
</h3>

<p>Marcel averages past results directly. Two players with OPS .800 might have very different K% and BB% profiles, which affects how stable their performance will be next year.</p>

<h3>
  
  
  Problem 3: No Uncertainty
</h3>

<p>"Maki's OPS: .812" gives no sense of how much it might vary. The difference between .750-.870 and .790-.830 matters a lot for team projections.</p>

<h2>
  
  
  What Changed with Bayesian Integration
</h2>

<h3>
  
  
  Foreign Players: Average → Individual Predictions
</h3>

<p>Built a model to convert MLB/KBO stats to NPB projections. For example, a .350 wOBA MLB hitter maps to approximately <code>.350 × 1.235 = .432</code> NPB-equivalent wOBA.</p>

<p>All 24 players' names and prior-league stats were individually web-verified (guessing English names from katakana is surprisingly error-prone).</p>

<p><strong>Foreign hitter examples:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Player</th>
<th>Team</th>
<th>Prior wOBA</th>
<th>NPB Pred OPS</th>
<th>80% CI</th>
</tr>
</thead>
<tbody>
<tr>
<td>Sano</td>
<td>Dragons</td>
<td>.370</td>
<td>.760</td>
<td>.632–.889</td>
</tr>
<tr>
<td>Seymour</td>
<td>Buffaloes</td>
<td>.365</td>
<td>.735</td>
<td>.607–.863</td>
</tr>
<tr>
<td>Dalbec</td>
<td>Giants</td>
<td>.355</td>
<td>.725</td>
<td>.577–.884</td>
</tr>
<tr>
<td>Hummel</td>
<td>BayStars</td>
<td>.240</td>
<td>.694</td>
<td>.530–.849</td>
</tr>
</tbody>
</table></div>

<p><strong>Foreign pitcher examples:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Player</th>
<th>Team</th>
<th>Prior ERA</th>
<th>NPB Pred ERA</th>
<th>80% CI</th>
</tr>
</thead>
<tbody>
<tr>
<td>Quijada</td>
<td>Swallows</td>
<td>3.26</td>
<td>2.76</td>
<td>1.28–4.24</td>
</tr>
<tr>
<td>Hjelle</td>
<td>Buffaloes</td>
<td>3.90</td>
<td>3.34</td>
<td>1.05–5.59</td>
</tr>
<tr>
<td>Cox</td>
<td>BayStars</td>
<td>8.86</td>
<td>3.36</td>
<td>1.82–4.85</td>
</tr>
</tbody>
</table></div>

<p>Players with poor prior-league stats get pulled toward league average (Bayesian regression effect), but with wider CIs = lower confidence.</p>

<h3>
  
  
  Japanese Players: K%/BB%/BABIP Corrections
</h3>

<p>Three models combined into a final prediction:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Model</th>
<th>Weight</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>Marcel</td>
<td>35%</td>
<td>Strong baseline, especially for pitcher ERA</td>
</tr>
<tr>
<td>Bayesian correction</td>
<td>40%</td>
<td>K%/BB%/BABIP/age adjustment on top of Marcel</td>
</tr>
<tr>
<td>ML</td>
<td>25%</td>
<td>XGBoost/LightGBM</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Did Accuracy Improve?
</h2>

<p>8-year backtest (2018–2025, predict each year and compare to actual):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Marcel MAE</th>
<th>Bayesian MAE</th>
<th>Improvement prob.</th>
</tr>
</thead>
<tbody>
<tr>
<td>Hitter wOBA</td>
<td>0.05023</td>
<td><strong>0.04980</strong></td>
<td>97.1%</td>
</tr>
<tr>
<td>Pitcher ERA</td>
<td>1.23008</td>
<td><strong>1.22241</strong></td>
<td>97.1%</td>
</tr>
</tbody>
</table></div>

<p>Small improvement, but consistent — <strong>97% probability of beating Marcel across 8 years</strong>.</p>

<h3>
  
  
  Historical Marcel Accuracy for Context
</h3>

<p><strong>Overall (8 years × 12 teams = 96 team-years):</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Wins MAE</td>
<td><strong>6.4 wins</strong></td>
</tr>
<tr>
<td>Avg rank error</td>
<td>1.42 positions</td>
</tr>
<tr>
<td>Exact rank rate</td>
<td>18%</td>
</tr>
<tr>
<td>Within 1 rank</td>
<td>65%</td>
</tr>
</tbody>
</table></div>

<p><strong>Recent examples of Marcel misses:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Year</th>
<th>Team</th>
<th>Actual</th>
<th>Predicted</th>
<th>Miss</th>
</tr>
</thead>
<tbody>
<tr>
<td>2025</td>
<td>Swallows (CL)</td>
<td>57W (6th)</td>
<td>72W (4th)</td>
<td>+15</td>
</tr>
<tr>
<td>2024</td>
<td>SoftBank (PL)</td>
<td>91W (1st)</td>
<td>75W (2nd)</td>
<td>-16</td>
</tr>
<tr>
<td>2024</td>
<td>Buffaloes (PL)</td>
<td>63W (5th)</td>
<td>78W (1st)</td>
<td>+15</td>
</tr>
</tbody>
</table></div>

<p><strong>Patterns:</strong></p>

<ul>
<li>Overestimates bottom teams, underestimates top teams (regression to mean)</li>
<li>Can't predict collapses (2024 Buffaloes: defending champions → 5th place)</li>
<li>Foreign player impact not captured when all treated as average</li>
</ul>

<h2>
  
  
  How Did the 2026 Standings Change?
</h2>

<h3>
  
  
  Central League — Tigers Runaway Disappears, 4-Team Deadlock
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Team</th>
<th>Marcel</th>
<th>Bayesian</th>
<th>Diff</th>
<th>P(Pennant)</th>
</tr>
</thead>
<tbody>
<tr>
<td>Tigers</td>
<td><strong>80W (1st)</strong></td>
<td>72W (1st)</td>
<td><strong>-9</strong></td>
<td>26.0%</td>
</tr>
<tr>
<td>Giants</td>
<td>71W (3rd)</td>
<td>71W (2nd)</td>
<td>+0</td>
<td>20.2%</td>
</tr>
<tr>
<td>Dragons</td>
<td>69W (5th)</td>
<td>71W (3rd)</td>
<td>+2</td>
<td>21.2%</td>
</tr>
<tr>
<td>BayStars</td>
<td>71W (2nd)</td>
<td>71W (4th)</td>
<td>-1</td>
<td>20.2%</td>
</tr>
<tr>
<td>Carp</td>
<td>70W (4th)</td>
<td>69W (5th)</td>
<td>-1</td>
<td>12.3%</td>
</tr>
<tr>
<td>Swallows</td>
<td>64W (6th)</td>
<td>61W (6th)</td>
<td>-3</td>
<td>0.1%</td>
</tr>
</tbody>
</table></div>

<p><strong>Tigers dropped from 80W to 72W (-9).</strong> Skill corrections pulled them down. Giants stayed at 71W even after losing Okamoto to MLB. <strong>Four teams within 1 win of each other</strong> — Tigers 26%, Dragons 21%, Giants 20%, BayStars 20%. Swallows at 61W (78% chance of last place) after Murakami's MLB departure.</p>

<h3>
  
  
  Pacific League — Lions Surge
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Team</th>
<th>Marcel</th>
<th>Bayesian</th>
<th>Diff</th>
<th>P(Pennant)</th>
</tr>
</thead>
<tbody>
<tr>
<td>Hawks</td>
<td>80W (1st)</td>
<td>81W (1st)</td>
<td>+1</td>
<td>47.9%</td>
</tr>
<tr>
<td>Fighters</td>
<td>77W (2nd)</td>
<td>79W (2nd)</td>
<td>+2</td>
<td>27.2%</td>
</tr>
<tr>
<td>Buffaloes</td>
<td>74W (3rd)</td>
<td>78W (3rd)</td>
<td>+4</td>
<td>17.6%</td>
</tr>
<tr>
<td>Lions</td>
<td>69W (4th)</td>
<td>75W (4th)</td>
<td><strong>+6</strong></td>
<td>7.1%</td>
</tr>
<tr>
<td>Eagles</td>
<td>66W (5th)</td>
<td>67W (5th)</td>
<td>+1</td>
<td>0.1%</td>
</tr>
<tr>
<td>Marines</td>
<td>67W (6th)</td>
<td>65W (6th)</td>
<td>-2</td>
<td>0.1%</td>
</tr>
</tbody>
</table></div>

<p><strong>Lions +6 wins</strong> — foreign player projections offsetting Imai's MLB departure.</p>

<h2>
  
  
  Summary
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Problem</th>
<th>Before</th>
<th>After</th>
</tr>
</thead>
<tbody>
<tr>
<td>Foreign players</td>
<td>All league-average</td>
<td>24 individual projections from prior-league stats</td>
</tr>
<tr>
<td>Skill metrics</td>
<td>Not used</td>
<td>K%/BB%/BABIP corrections on Marcel</td>
</tr>
<tr>
<td>Uncertainty</td>
<td>None (point estimates)</td>
<td>80%/95% credible intervals on every prediction</td>
</tr>
<tr>
<td>Team standings</td>
<td>Single number</td>
<td>10,000 Monte Carlo sims with pennant probabilities</td>
</tr>
<tr>
<td>Accuracy</td>
<td>Marcel MAE 0.050</td>
<td>
<strong>0.0498</strong> (97% probability of improvement)</td>
</tr>
</tbody>
</table></div>

<p>The accuracy gain is modest, but "foreign players are no longer invisible," "MLB departures are reflected," and "every prediction comes with uncertainty" meaningfully changed the standings picture. The CL went from "Tigers runaway" to a four-team deadlock.</p>

<h3>
  
  
  Caveat: Data Limitations
</h3>

<p>During this work, I discovered that <strong>players who moved to MLB (Murakami, Okamoto)</strong> were still included in the team simulation — the roster filter only existed in the Streamlit display layer, not in the CSV generation pipeline. Fixed and regenerated, but <strong>there may be other oversights I haven't caught.</strong></p>

<p>This is a personal project without professional-grade QA. The data is best treated as automated model output, not authoritative predictions.</p>

<ul>
<li>
<strong>Dashboard</strong>: <a href="https://npb-prediction.streamlit.app/" rel="noopener noreferrer">npb-prediction.streamlit.app</a>
</li>
<li>
<strong>GitHub</strong>: <a href="https://github.com/yasumorishima/npb-prediction" rel="noopener noreferrer">github.com/yasumorishima/npb-prediction</a>
</li>
</ul>

<h2>
  
  
  Data Sources
</h2>

<ul>
<li>
<a href="https://baseball-data.com" rel="noopener noreferrer">Baseball Data Freak</a> — NPB player stats</li>
<li>
<a href="https://npb.jp" rel="noopener noreferrer">NPB Official</a> — Official records</li>
</ul>

