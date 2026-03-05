---
Title: How I Built 3 Chainlink CRE Workflows in One Week — FBA Oracle, Privacy Payroll & DeFi Intelligence at 3 AM in Secunderabad
Description: 
Author: Venkateshwar Rao Nagala
Date: 2026-03-05T21:29:47.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  How I Built 3 Chainlink CRE Workflows in One Week — FBA Oracle, Privacy Payroll &amp; DeFi Intelligence at 3 AM in Secunderabad — From a 2021 Podcast to a 3 AM Deployment
</h1>

<p><strong>Author:</strong> Venkateshwar Rao Nagala<br>
<strong>GitHub:</strong> <a href="https://github.com/tenalirama2025-creator/chainlink-prediction-settler" rel="noopener noreferrer">https://github.com/tenalirama2025-creator/chainlink-prediction-settler</a><br>
<strong>Competition:</strong> Chainlink Convergence Hackathon 2026</p>


<h2>
  
  
  Demo Video
</h2>

<p>▶️ <strong>Watch the full pipeline in action:</strong><br>
[INSERT DEMO VIDEO LINK HERE]</p>


<h2>
  
  
  My Story
</h2>

<p>I worked on blockchain projects earlier using JavaScript, RUST and Python for settlement of forex transactions across banks. The blockchain forex settlement enabled banks to settle forex transactions within minutes down from weeks. This blockchain experience helped me in my decision to be part of this Convergence Hackathon 2026.</p>

<p>Last month I won the Velocity Award at the Solo.io MCP &amp; AI Agents Hackathon<br>
for building a COBOL→Rust modernization pipeline with zero-trust AgentGateway<br>
security. This month I applied that same velocity to the Chainlink Convergence<br>
Hackathon.</p>

<p>My interest in Chainlink did not start in February 2026 when I received the<br>
hackathon email. It started on May 1, 2021 — I have the MIT Open email to<br>
prove it — when I watched Lex Fridman's podcast with Sergey Nazarov on Smart<br>
Contracts and Chainlink. That podcast planted a seed. I have followed Chainlink<br>
Foundation activities ever since — Smart Conferences, Bootcamps, developer<br>
updates. Five years later, at 3 AM in Secunderabad, I was watching Chainlink<br>
CRE workflows simulate green on my terminal.</p>

<p>On February 28, 2026 — just one week before the March 8 deadline — I received<br>
the Chainlink Convergence hackathon email. I was still finishing the Solo.io<br>
submission. Another round of late nights was about to begin.</p>

<p>I have 30 years of software experience — from writing Fortran for gearbox<br>
manufacturing simulations in my final year of Mechanical Engineering, to 10<br>
years of IBM Assembler programming for insurance companies in the USA<br>
(Homecomm/Lifecomm), to Core Banking reconciliation at RBC, to Big Data<br>
Analytics at AIG, to Assembler and COBOL at Thrivent. I gathered all of that<br>
experience and decided to compete.</p>


<h2>
  
  
  The Problem That Started It All
</h2>

<p>Prediction markets need trustworthy, automated settlement. But a single AI<br>
oracle can hallucinate. A single LLM can be wrong with high confidence.<br>
A single data source can be compromised.</p>

<p>I had studied Byzantine fault tolerance throughout my career in distributed<br>
systems. I knew about Stellar's Federated Byzantine Agreement protocol. The<br>
question was: could FBA be applied to LLM oracles?</p>

<p>The answer became this project.</p>


<h2>
  
  
  The "Aha Moment"
</h2>

<p>Initially I decided to go with claude-opus-4-6 LLM for prediction, but <br>
decided to add another LLM that is GPT-4o. If anyone of the LLM <br>
hallucinates then one of them will give the correct prediction, <br>
but the problem was how to decide which one is correct and other <br>
is not correct. The solution is Byzantine algorithm.</p>

<p>I knew that consensus on blockchain networks is achieved through Byzantine<br>
algorithms. I had three project ideas initially — a prediction market settler,<br>
a privacy-preserving payroll workflow, and a DeFi market intelligence tool.<br>
After seeing the scope, I decided to combine all three as individual MCPs in<br>
a single unified project.</p>

<p>The "aha moment" came when I saw the FBA consensus result appear — Claude<br>
Opus saying NO at 99% confidence, GPT-4o saying NO at 95% confidence, the<br>
FBA engine confirming quorum intersection — and knowing that result was about<br>
to be written to a smart contract on the blockchain via MetaMask. That moment<br>
made every late night worth it.</p>


<h2>
  
  
  What I Built
</h2>

<p>A complete <strong>AI-powered prediction market settlement system</strong> with:</p>

<ul>
<li><p><strong>ai_prediction_mcp</strong> — FBA consensus oracle (Claude Opus + GPT-4o) via<br>
Chainlink CRE. Both LLMs independently evaluate a prediction statement.<br>
FBA computes quorum intersection. On agreement: auto-settle on-chain.<br>
On disagreement: autonomous settlement BLOCKED — human review triggered.</p></li>
<li><p><strong>payroll_mcp</strong> — Privacy-preserving batch payroll settlement. Recipient<br>
addresses and salary amounts never appear on-chain. Only a keccak256 batch<br>
hash is recorded. CRE Confidential Compute keeps all sensitive data private.</p></li>
<li><p><strong>hyperliquid_mcp</strong> — Hyperliquid DEX market intelligence. Queries 229<br>
perpetual markets via CRE Confidential HTTP. Computes BTC bid-ask spread<br>
in basis points. API credentials managed at DON level — never exposed.</p></li>
<li><p><strong>Rust CLI</strong> — FBA consensus engine (TypeScript port of Stellar's SCP),<br>
fetch-fed, simulate, and status commands.</p></li>
<li><p><strong>PredictionMarket.sol</strong> — Solidity smart contract deployed and verified<br>
on Ethereum Sepolia. Verified on Sourcify, Blockscout, and Routescan.</p></li>
</ul>

<p><strong>Live FBA Result:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Statement:        "Did the Fed cut rates at March 2025 FOMC?"
Claude Opus:      NO  |  99% confidence
GPT-4o:           NO  |  95% confidence
FBA Outcome:      NO  |  97% confidence
Quorum:           ✅ INTERSECTED
Safety:           ✅ CLEARED
Action:           SETTLE NO — Fed held rates confirmed
</code></pre>

</div>






<h2>
  
  
  Architecture
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────────────────────────────┐
│                    CHAINLINK CRE LAYER                          │
│                                                                 │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │ payroll_mcp  │  │ hyperliquid_mcp  │  │ ai_prediction    │  │
│  │              │  │                  │  │ _mcp             │  │
│  │ Privacy-     │  │ DEX Market       │  │ FBA Oracle       │  │
│  │ preserving   │  │ Intelligence     │  │ (Claude+GPT-4o)  │  │
│  │ batch settle │  │ BTC/ETH spreads  │  │                  │  │
│  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘  │
└─────────┼───────────────────┼──────────────────────┼────────────┘
          │                   │                      │
          ▼                   ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RUST CLI (settler)                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              FBA CONSENSUS ENGINE                       │   │
│  │   Claude Opus ──┐                                       │   │
│  │                 ├──► Quorum Intersection ──► SETTLE     │   │
│  │   GPT-4o    ────┘    (Stellar Protocol)    or REVIEW    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────┐
│              ETHEREUM SEPOLIA (Smart Contract)                  │
│  PredictionMarket.sol                                           │
│  0xEA856dF995C58DEc18221C907DC221c4487Ae499                    │
│  Question: "Did the Fed cut rates at March 2025 FOMC?"         │
└─────────────────────────────────────────────────────────────────┘
</code></pre>

</div>






<h2>
  
  
  The Three Thrillers
</h2>

<h3>
  
  
  Thriller 1: The MetaMask Maze 🦊
</h3>

<p>When I wanted to install MetaMask, I searched for it online. The search showed<br>
a page full of websites. I clicked one that instructed me to download a .msi<br>
file for Windows. The downloaded files ended up in my project folder — around<br>
10,000 files in two separate folders. This filled my C drive completely.</p>

<p>I only discovered this during the GitHub upload — the files showed up in git<br>
and I immediately deleted them from VSCode and GitHub.</p>

<p>The MetaMask website itself was a maze for a first-time visitor. They clearly<br>
display browser extension options — Chrome, Edge, Firefox — but when I clicked<br>
the Chrome extension, the GET button was greyed out with a message:<br>
"You can't add or remove items when in Incognito or Guest mode."</p>

<p>I was in Incognito mode on Edge. I switched to normal mode. Same message.<br>
I eventually found the Chrome Web Store option — but that led to another<br>
problem.</p>

<p>I created Account 1 and added a wallet. Then imported Account 1, creating an<br>
"Imported Account 1." I got 0.05 Sepolia ETH from the Google faucet and added<br>
it to Account 1.</p>

<p>When I tried to Deploy &amp; Run my Solidity contract in Remix, MetaMask picked<br>
the Imported Account 1 — which had no funds. I was confused. I had just added<br>
tokens!</p>

<p>I investigated. I went back to the MetaMask Chrome extension and discovered<br>
that Account 1 and Imported Account 1 had DIFFERENT wallet addresses. MetaMask<br>
prioritises the Imported Account during Deploy &amp; Run.</p>

<p>I deleted the Imported Account 1. Recompiled. Deployed. This time MetaMask<br>
used Account 1 with the funds — and the contract was deployed successfully.</p>


<h3>
  
  
  Thriller 2: The Wrong Contract Address 📋
</h3>

<p>After deployment I committed the contract address to GitHub — but I captured<br>
the wrong URL. The address I saved was for the default Remix Storage contract,<br>
not my PredictionMarket.sol.</p>

<p>The same wrong address was deployed to CRE by Thomas of Chainlink team. Two days<br>
before submission I discovered the mistake.</p>

<p>I corrected the address everywhere — .env, config files, README. Then I<br>
informed Thomas from the Chainlink team and requested redeployment with the<br>
correct address. Waiting for him to complete the redeployment — with the<br>
submission deadline two days away — was nothing short of watching a thriller<br>
movie.</p>


<h3>
  
  
  Thriller 3: The 3 AM CRE Debugging Session 🌙
</h3>

<p>The night of March 3-4, 2026. It was past midnight in Secunderabad. I was<br>
trying to get all three CRE workflows simulating.</p>

<p>The errors came one after another:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✗ ConfigPath must be a valid existing file: ./config.staging.json
✗ environment variable PAYROLL_API_KEY_VALUE not found
✗ 'btoa' is not defined
✗ Expected identifier but found ","  (double comma syntax error)
✗ Buffer.from base64 encoding required
</code></pre>

</div>



<p>Each error revealed the next. Missing config file. Wrong secrets format.<br>
Wrong base64 encoding function. Double comma from a PowerShell replace command.</p>

<p>At 1:42 AM the terminal showed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>╭──────────────────────────────────────────────────────╮
│ Simulation complete! Ready to deploy your workflow?  │
╰──────────────────────────────────────────────────────╯
</code></pre>

</div>



<p>All three workflows. Green. That moment made every late night worth it.</p>




<h2>
  
  
  CRE Workflow Deep Dive
</h2>

<h3>
  
  
  ai_prediction_mcp — FBA Consensus Oracle
</h3>

<p>The key innovation: secrets fetched at DON level, passed into node mode as<br>
arguments. API keys for Claude and OpenAI never leave Chainlink's confidential<br>
compute environment.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Fetch secrets at DON level — same pattern as payroll_mcp</span>
<span class="kd">const</span> <span class="nx">claudeSecret</span> <span class="o">=</span> <span class="nx">runtime</span><span class="p">.</span><span class="nf">getSecret</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">config</span><span class="p">.</span><span class="nx">claudeApiKeySecretName</span> <span class="p">}).</span><span class="nf">result</span><span class="p">();</span>
<span class="kd">const</span> <span class="nx">openaiSecret</span> <span class="o">=</span> <span class="nx">runtime</span><span class="p">.</span><span class="nf">getSecret</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">config</span><span class="p">.</span><span class="nx">openaiApiKeySecretName</span> <span class="p">}).</span><span class="nf">result</span><span class="p">();</span>

<span class="c1">// Pass both keys into runInNodeMode</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">runtime</span>
  <span class="p">.</span><span class="nf">runInNodeMode</span><span class="p">(</span>
    <span class="nx">runFbaConsensus</span><span class="p">,</span>
    <span class="nx">consensusIdenticalAggregation</span><span class="o">&lt;</span><span class="nx">SettlementResult</span><span class="o">&gt;</span><span class="p">()</span>
  <span class="p">)(</span><span class="nx">claudeApiKey</span><span class="p">,</span> <span class="nx">openaiApiKey</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">result</span><span class="p">();</span>
</code></pre>

</div>



<h3>
  
  
  payroll_mcp — Privacy-Preserving Settlement
</h3>

<p>Only the batch hash is recorded on-chain. Amounts and recipient addresses<br>
stay inside CRE confidential compute:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// PRIVATE — never written on-chain</span>
<span class="kd">type</span> <span class="nx">PayrollEntry</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">recipient</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">amount</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">memo</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">};</span>

<span class="c1">// Only this is recorded on-chain</span>
<span class="k">return</span> <span class="p">{</span>
  <span class="na">batchHash</span><span class="p">:</span> <span class="nf">computeBatchHash</span><span class="p">(</span><span class="nx">batch</span><span class="p">),</span>
  <span class="na">privacy</span><span class="p">:</span> <span class="dl">"</span><span class="s2">recipient_addresses_and_amounts_not_recorded_onchain</span><span class="dl">"</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  hyperliquid_mcp — DeFi Market Intelligence
</h3>

<p>CRE Confidential HTTP — API credentials managed at DON level:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">body</span><span class="p">:</span> <span class="nx">Buffer</span><span class="p">.</span><span class="k">from</span><span class="p">(</span>
  <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">type</span><span class="p">:</span> <span class="dl">"</span><span class="s2">meta</span><span class="dl">"</span> <span class="p">})</span>
<span class="p">).</span><span class="nf">toString</span><span class="p">(</span><span class="dl">"</span><span class="s2">base64</span><span class="dl">"</span><span class="p">),</span>  <span class="c1">// CRE requires base64 encoded body</span>
</code></pre>

</div>






<h2>
  
  
  Key Technical Lessons
</h2>

<p><strong>On CRE secrets:</strong><br>
Use <code>secretsNames</code> format in secrets.yaml — not a <code>secrets</code> list. Each<br>
workflow needs its own local <code>./secrets.yaml</code> — sharing <code>../secrets.yaml</code><br>
causes cross-workflow secret conflicts.</p>

<p><strong>On CRE HTTP:</strong><br>
The CRE HTTPClient requires request body as base64. Use<br>
<code>Buffer.from(JSON.stringify(body)).toString("base64")</code> — not btoa() which<br>
is not available in the CRE runtime.</p>

<p><strong>On MetaMask:</strong><br>
Always install as a Chrome extension from the Chrome Web Store. Never download<br>
desktop packages. Delete any Imported Accounts before deploying — they take<br>
priority over regular accounts during Remix deployment.</p>

<p><strong>On contract verification:</strong><br>
Always click the Contract tab on Blockscout after deployment and verify the<br>
contract name matches your Solidity file — not the default Remix Storage<br>
contract.</p>

<p><strong>On FBA for LLMs:</strong><br>
Byzantine fault tolerance is not just for distributed databases. Applied to<br>
LLM oracles, FBA provides quorum intersection guarantees — a hallucinating<br>
model cannot unilaterally settle a market.</p>




<h2>
  
  
  Chainlink CRE Dashboard
</h2>

<p>[INSERT THOMAS'S DASHBOARD SCREENSHOT HERE]</p>

<p>All three workflows active on Chainlink's DON — confirmed by the Chainlink<br>
team. Running every 10 minutes via CronCapability.</p>




<h2>
  
  
  Quick Start
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/tenalirama2025-creator/chainlink-prediction-settler
<span class="nb">cd </span>chainlink-prediction-settler

<span class="c"># Build Rust CLI</span>
cargo build

<span class="c"># Simulate FBA consensus (free — no API keys needed)</span>
./target/debug/settler fba <span class="se">\</span>
  <span class="nt">--statement</span> <span class="s2">"The Fed maintained rates at 4.25-4.50%"</span> 

<span class="c"># Run CRE workflow simulations</span>
<span class="nb">cd </span>workflow/payroll_mcp
<span class="nv">$env</span>:PAYROLL_API_KEY_VALUE <span class="o">=</span> <span class="s2">"dev-api-key"</span>
cre workflow simulate <span class="nb">.</span> <span class="nt">--target</span> staging-settings

<span class="nb">cd</span> ../hyperliquid_mcp
cre workflow simulate <span class="nb">.</span> <span class="nt">--target</span> staging-settings

<span class="nb">cd</span> ../ai_prediction_mcp
<span class="nv">$env</span>:CLAUDE_API_KEY_VALUE<span class="o">=</span><span class="s2">"dev-api-key"</span>
<span class="nv">$env</span>:OPENAI_API_KEY_VALUE<span class="o">=</span><span class="s2">"dev-api-key"</span>
cre workflow simulate <span class="nb">.</span> <span class="nt">--target</span> staging-settings
</code></pre>

</div>






<h2>
  
  
  Prize Tracks
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Track</th>
<th>Amount</th>
<th>How This Project Qualifies</th>
</tr>
</thead>
<tbody>
<tr>
<td>Prediction Markets</td>
<td>$16,000</td>
<td>FBA consensus settles real on-chain predictions</td>
</tr>
<tr>
<td>Privacy</td>
<td>$16,000</td>
<td>payroll_mcp — batch hash only, amounts never on-chain</td>
</tr>
<tr>
<td>CRE &amp; AI</td>
<td>$17,000</td>
<td>3 production CRE workflows running on DON</td>
</tr>
<tr>
<td>Risk &amp; Compliance</td>
<td>$16,000</td>
<td>FBA blocks unsafe settlements — human review triggered</td>
</tr>
<tr>
<td>DeFi</td>
<td>$16,000</td>
<td>hyperliquid_mcp — DEX intelligence via confidential HTTP</td>
</tr>
</tbody>
</table></div>

<p><strong>Total potential: $81,000</strong></p>




<h2>
  
  
  My Message to You
</h2>

<p>Please take the risk if you enjoy the thrills that accompany such projects<br>
with tight deadlines.</p>

<p>Two days before submission you realize the contract URL is wrong. The same<br>
mistake was hosted on CRE by Thomas of Chainlink team. Requesting him to redeploy<br>
after correcting the mistake, and waiting for him to complete the redeployment,<br>
is nothing short of watching a thriller movie.</p>

<p>I received the hackathon email one week before the deadline. I had a Solo.io<br>
submission still in progress. I had never built a CRE workflow before.</p>

<p>I entered anyway.</p>

<p>The MIT Open email from May 1, 2021 — the Lex Fridman podcast with Sergey<br>
Nazarov that started this journey — sits in my inbox. Five years from that<br>
email to this submission.</p>

<p>Take the risk. The thrill is worth it.</p>




<h2>
  
  
  About Me
</h2>

<p><strong>Venkateshwar Rao Nagala</strong></p>

<ul>
<li>GATE 1994 AIR 444 — Top 0.4% of India's engineering graduates</li>
<li>Fortran — Final year Mechanical Engineering project:
Computer Assisted Manufacturing of Gear Boxes</li>
<li>1994-2004 — IBM Assembler expert, insurance companies USA
(Homecomm/Lifecomm)</li>
<li>RBC — Senior Project Manager, Core Banking (RDDA/BDDA)</li>
<li>AIG — Manager Big Data Analytics</li>
<li>Thrivent — Consultant, Assembler 80% / COBOL 20%</li>
<li>Solo.io 2026 — Velocity Award 🏆
"Also read my Solo.io Velocity Award blog: 
<a href="https://dev.to/venkateshwar_raonagala_4/how-i-added-zero-trust-guardrails-to-4-mcp-servers-using-agentgateway-and-modernized-legacy-cobol-1fl8">https://dev.to/venkateshwar_raonagala_4/how-i-added-zero-trust-guardrails-to-4-mcp-servers-using-agentgateway-and-modernized-legacy-cobol-1fl8</a>
</li>
<li>Chainlink follower since May 1, 2021 (MIT Open — Sergey Nazarov podcast)</li>
</ul>

<p>📧 <a href="mailto:tenalirama2019@gmail.com">tenalirama2019@gmail.com</a><br>
📱 +91-9701908080<br>
🐙 <a href="https://github.com/tenalirama2025-creator" rel="noopener noreferrer">https://github.com/tenalirama2025-creator</a></p>




<p><em>Built with Rust. Orchestrated with Chainlink CRE. Settled on Ethereum Sepolia.</em></p>

<p><em>From Assembler to blockchain — one late night at a time.</em> 🚀</p>

<p><strong>GitHub:</strong> <a href="https://github.com/tenalirama2025-creator/chainlink-prediction-settler" rel="noopener noreferrer">https://github.com/tenalirama2025-creator/chainlink-prediction-settler</a><br>
<strong>Contract:</strong> 0xEA856dF995C58DEc18221C907DC221c4487Ae499 (Sepolia)<br>
<strong>Demo:</strong> [INSERT VIDEO LINK]</p>

