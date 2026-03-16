---
Title: How we built a real-time pharmacogenomic agent with Gemini Live and C++23 at 40 nanoseconds
Description: 
Author: Riyane El Qoqui
Date: 2026-03-16T21:49:25.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This article was created for the Gemini Live Agent Challenge hackathon.</em></p>




<h2>
  
  
  The Problem
</h2>

<p>Every year, 2 million people are hospitalized from adverse drug reactions. Around 100,000 die. Not from their disease — from the treatment.</p>

<p>Most of these deaths are not accidents. They are written in the patient's genome, waiting to be read.</p>

<p>A patient who is a CYP2D6 Ultra-Rapid Metabolizer converts codeine into morphine at 4× the normal rate. Prescribe standard codeine — you've ordered a fatal overdose. The doctor didn't know. The pharmacist didn't know. Nobody checked the genome.</p>

<p>This is not a rare edge case. CYP2D6 Poor Metabolizers represent ~8% of the European population. HLA-B*5701 carriers — for whom abacavir causes fatal hypersensitivity — represent ~6% of HIV patients in Western cohorts. These variants are not exotic. They are common. They are documented. And they are routinely ignored at the point of prescription.</p>

<p>Pharmacogenomic tools exist. They live in separate systems, require manual lookup, and operate on a different timescale than a clinical consultation. By the time the alert surfaces, the prescription is already signed.</p>

<p>The problem is not data. It is latency.</p>




<h2>
  
  
  Architecture: Two Layers That Never Blur
</h2>

<p>PharmaShield is built on one strict rule: <strong>the LLM makes zero clinical decisions.</strong></p>

<p>This is not a stylistic choice. A language model that hallucinates a phenotype in a clinical context is not a UX problem — it is a liability. Clinical decisions must be deterministic, auditable, and reproducible. That is C++ territory.</p>

<p>The system splits into two layers with no overlap.</p>

<p><strong>Gemini 2.5 Flash Live — the intelligence layer.</strong><br>
Listens to the ambient audio stream of the consultation in real-time. Extracts entities: patient ID, drug name, current medications, age, renal function. Fires a tool call the moment both a patient identifier and a drug name are identified — without waiting for the doctor to finish speaking. It does not evaluate risk. It extracts and delegates.</p>

<p><strong>C++23 Sentinel Core — the deterministic layer.</strong><br>
Receives the tool call. Binary searches a memory-mapped file in ~40 nanoseconds. Runs a 5-step pharmacogenomic scoring pipeline — entirely hardcoded from CPIC clinical guidelines. Returns a structured <code>RiskResult</code>. No inference. No probability. No temperature.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>     Doctor speaks → Gemini Live (audio stream)
                      ↓
     Extracts: patient_id + drug + current_meds
                      ↓
     Tool call: check_pharmacogenomics()
                      ↓
     C++23 Sentinel Core — mmap + binary search — 40ns
                      ↓
     5-step risk pipeline (PGx + physiology + DDI)
                      ↓
     If critical → Gemini voice barge-in interrupts doctor
</code></pre>

</div>



<p>End-to-end latency from the doctor saying the drug name to the audio alert: ~500ms. The C++ engine contributes 40 nanoseconds. The bottleneck is Gemini's TTFT — the intelligence layer, not the safety layer. <strong>That asymmetry is intentional.</strong></p>




<h2>
  
  
  Why C++23 for Medicine
</h2>

<p>The question is not why C++23. The question is why anything else would be acceptable.</p>

<p>This is a safety-critical path. Every millisecond of latency between a prescription and a contraindication alert is a window where the doctor moves forward. We cannot afford allocations. We cannot afford virtual dispatch. We cannot afford a garbage collector deciding to run mid-consultation.</p>

<p>So we did what the hardware demands.</p>

<p><strong>Memory mapping over database.</strong></p>

<p><code>patients.bin</code> is a flat binary file. One record per patient, 16 bytes, <code>alignas(16)</code>. At startup, a single <code>mmap(PROT_READ, MAP_PRIVATE)</code> maps the entire dataset into the process address space. No I/O on the hot path. No query planner. No connection pool. The dataset is a typed C++ span:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="n">std</span><span class="o">::</span><span class="n">span</span><span class="o">&lt;</span><span class="k">const</span> <span class="n">PatientRisk</span><span class="o">&gt;</span> <span class="n">m_data</span> <span class="o">=</span> <span class="n">m_region</span><span class="p">.</span><span class="n">get_span</span><span class="p">();</span>
</code></pre>

</div>



<p>150,000 patients. ~1.8 MB. Fits entirely in L3 cache. Once warm, lookups never touch DRAM.</p>

<p><strong>Binary search over index.</strong></p>

<p>The file is sorted by <code>subject_id</code> at build time — enforced by <code>builder.py</code>, asserted at startup. The lookup is a single call:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">auto</span> <span class="n">range</span> <span class="o">=</span> <span class="n">std</span><span class="o">::</span><span class="n">ranges</span><span class="o">::</span><span class="n">equal_range</span><span class="p">(</span><span class="n">m_data</span><span class="p">,</span> <span class="n">patient_id</span><span class="p">,</span> <span class="p">{},</span> <span class="o">&amp;</span><span class="n">PatientRisk</span><span class="o">::</span><span class="n">subject_id</span><span class="p">);</span>
</code></pre>

</div>



<p>O(log N). ~40 nanoseconds with warm cache. No hash table. No tree traversal. No pointer chasing. The CPU's prefetcher handles the rest.</p>

<p><strong><code>constexpr</code> tables over runtime logic.</strong></p>

<p>Every clinical rule — CYP phenotype matrices, HLA flags, drug interaction pairs, inhibitor strengths — lives in <code>constexpr</code> arrays evaluated at compile time. The phenoconversion transition matrices are branchless lookups:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">constexpr</span> <span class="n">PhenoType</span> <span class="n">DOWNGRADE_MATRIX</span><span class="p">[</span><span class="mi">5</span><span class="p">][</span><span class="mi">3</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="p">{</span><span class="n">PhenoType</span><span class="o">::</span><span class="n">UNKNOWN</span><span class="p">,</span> <span class="n">PhenoType</span><span class="o">::</span><span class="n">UNKNOWN</span><span class="p">,</span> <span class="n">PhenoType</span><span class="o">::</span><span class="n">UNKNOWN</span><span class="p">},</span>
    <span class="p">{</span><span class="n">PhenoType</span><span class="o">::</span><span class="n">PM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">PM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">PM</span><span class="p">},</span>
    <span class="p">{</span><span class="n">PhenoType</span><span class="o">::</span><span class="n">IM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">PM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">PM</span><span class="p">},</span>
    <span class="p">{</span><span class="n">PhenoType</span><span class="o">::</span><span class="n">EM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">IM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">PM</span><span class="p">},</span>
    <span class="p">{</span><span class="n">PhenoType</span><span class="o">::</span><span class="n">UM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">EM</span><span class="p">,</span>      <span class="n">PhenoType</span><span class="o">::</span><span class="n">IM</span><span class="p">},</span>
<span class="p">};</span>
</code></pre>

</div>



<p>No branches. No heap. The scoring pipeline runs entirely in registers and L1 cache.</p>

<p><strong>Hash collision detection at compile time.</strong></p>

<p>Drug names are hashed with FNV-1a at compile time. Collisions are caught by a <code>consteval</code> function — the binary will not compile if two drugs hash to the same value. The safety check costs zero nanoseconds at runtime.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">consteval</span> <span class="kt">bool</span> <span class="nf">check_drug_hash_collisions</span><span class="p">()</span> <span class="p">{</span> <span class="p">...</span> <span class="p">}</span>
<span class="k">static_assert</span><span class="p">(</span><span class="n">check_drug_hash_collisions</span><span class="p">(),</span> <span class="s">"FATAL: Hash collision in DRUG_RULES!"</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>Software prefetching on keystrokes.</strong></p>

<p>As the doctor types a patient ID, every keystroke fires a prefetch. The engine estimates the ID range from the partial prefix and issues <code>__builtin_prefetch</code> on the corresponding cache lines. By the time Gemini fires the tool call, the patient's record is already in L1.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">for</span> <span class="p">(</span><span class="kt">ptrdiff_t</span> <span class="n">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="n">prefetch_count</span><span class="p">;</span> <span class="n">i</span> <span class="o">+=</span> <span class="mi">4</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">__builtin_prefetch</span><span class="p">(</span><span class="k">reinterpret_cast</span><span class="o">&lt;</span><span class="k">const</span> <span class="kt">char</span><span class="o">*&gt;</span><span class="p">(</span><span class="o">&amp;</span><span class="n">it</span><span class="p">[</span><span class="n">i</span><span class="p">]),</span> <span class="mi">0</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The result: the 40ns figure is not a best-case benchmark. It is the steady-state cost of a warm-cache lookup on a properly structured dataset. The CPU does not execute algorithms. It executes memory access patterns. We designed for the machine.</p>




<h2>
  
  
  Phenoconversion: The Risk Nobody Models in Real-Time
</h2>

<p>Pharmacogenomics has a known blind spot. Most systems stop at the genome.</p>

<p>The genome tells you what enzymes a patient <em>can</em> produce. It does not tell you what enzymes they are <em>currently</em> producing. These are not the same thing.</p>

<p>A patient who is genetically CYP2D6 Normal Metabolizer — EM, textbook — has normal codeine metabolism on paper. Prescribe codeine. Safe, correct, defensible.</p>

<p>Except they are currently taking paroxetine.</p>

<p>Paroxetine is a strong CYP2D6 inhibitor. It does not care what the genome says. It physically blocks the enzyme. The patient's <em>functional</em> phenotype is now PM — Poor Metabolizer. Codeine accumulates. The active morphine metabolite builds. The patient stops breathing.</p>

<p>The genome said EM. The clinical reality was PM. No static PGx lookup catches this. You need to know what the patient is <em>currently taking</em> — and you need to model the interaction at runtime.</p>

<p>This is phenoconversion. And it is the primary clinical differentiator of PharmaShield.</p>

<p><strong>The mechanism.</strong></p>

<p>Every Gemini tool call accumulates the patient's current medication list across the consultation. The C++ engine receives these as a <code>std::vector&lt;std::string&gt;</code> in <code>ClinicalContext</code>. Each drug name is hashed with FNV-1a and matched against two compile-time tables: inhibitors and inducers, classified by CYP and strength.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight cpp"><code><span class="k">struct</span> <span class="nc">PhenoModifier</span> <span class="p">{</span>
    <span class="kt">uint32_t</span> <span class="n">drug_hash</span><span class="p">;</span>
    <span class="n">CypOffset</span> <span class="n">cyp_index</span><span class="p">;</span>
    <span class="kt">uint8_t</span> <span class="n">strength</span><span class="p">;</span>  <span class="c1">// 0=weak 1=moderate 2=strong</span>
<span class="p">};</span>

<span class="k">constexpr</span> <span class="n">PhenoModifier</span> <span class="n">INHIBITORS</span><span class="p">[]</span> <span class="o">=</span> <span class="p">{</span>
    <span class="p">{</span><span class="n">fnv1a</span><span class="p">(</span><span class="s">"paroxetine"</span><span class="p">),</span>     <span class="n">CypOffset</span><span class="o">::</span><span class="n">CYP2D6</span><span class="p">,</span> <span class="mi">2</span><span class="p">},</span>  <span class="c1">// strong</span>
    <span class="p">{</span><span class="n">fnv1a</span><span class="p">(</span><span class="s">"fluoxetine"</span><span class="p">),</span>     <span class="n">CypOffset</span><span class="o">::</span><span class="n">CYP2D6</span><span class="p">,</span> <span class="mi">2</span><span class="p">},</span>  <span class="c1">// strong</span>
    <span class="p">{</span><span class="n">fnv1a</span><span class="p">(</span><span class="s">"bupropion"</span><span class="p">),</span>      <span class="n">CypOffset</span><span class="o">::</span><span class="n">CYP2D6</span><span class="p">,</span> <span class="mi">2</span><span class="p">},</span>  <span class="c1">// strong</span>
    <span class="p">{</span><span class="n">fnv1a</span><span class="p">(</span><span class="s">"clarithromycin"</span><span class="p">),</span> <span class="n">CypOffset</span><span class="o">::</span><span class="n">CYP3A5</span><span class="p">,</span> <span class="mi">2</span><span class="p">},</span>  <span class="c1">// strong</span>
    <span class="p">{</span><span class="n">fnv1a</span><span class="p">(</span><span class="s">"amiodarone"</span><span class="p">),</span>     <span class="n">CypOffset</span><span class="o">::</span><span class="n">CYP2C9</span><span class="p">,</span> <span class="mi">2</span><span class="p">},</span>  <span class="c1">// strong</span>
    <span class="c1">// ...</span>
<span class="p">};</span>
</code></pre>

</div>



<p>The transition is branchless. Inhibition strength maps directly into a downgrade matrix:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Genetic phenotype:    CYP2D6 EM   ← from patients.bin
Current meds:         [paroxetine] ← from ClinicalContext
                           ↓
INHIBITORS match:     strength = 2 (strong)
                           ↓
              DOWNGRADE_MATRIX[EM][2] = PM
                           ↓
Functional phenotype: CYP2D6 PM   ← this is what the scorer uses
</code></pre>

</div>



<p>Strong inhibitor — EM becomes PM, IM becomes PM. Moderate inhibitor — EM becomes IM. Two moderate inhibitors stack to strong. The matrix handles every case in a single array lookup. No branches. No conditionals.</p>

<p><strong>The warning surface.</strong></p>

<p>When phenoconversion fires, the <code>RiskResult</code> carries a <code>PHENOCONV:Y</code> flag in the structured header. The frontend renders a dedicated badge. Gemini's vocal alert explicitly names the mechanism:</p>

<blockquote>
<p><em>"Warning Doctor — patient 99999004 is genetically a normal metabolizer, but paroxetine is converting them to a functional poor metabolizer. Codeine will accumulate. Consider a non-opioid alternative."</em></p>
</blockquote>

<p>The doctor hears not just a contraindication — they hear the mechanism. That is the difference between an alert that gets dismissed and one that changes the prescription.</p>

<p><strong>Why nobody does this in real-time.</strong></p>

<p>Static PGx tools operate on the genome alone. EHR-integrated alerting systems check drug-drug interactions separately, in a different module, on a different trigger. Nobody combines genomic phenotype, phenoconversion via current medications, physiological context, and drug-drug interactions into a single scoring pipeline that runs in under 500 microseconds.</p>

<p>We do. In 40 nanoseconds of C++ and a Gemini barge-in.</p>




<h2>
  
  
  Results
</h2>

<p><strong>Latency.</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Operation</th>
<th>Latency</th>
</tr>
</thead>
<tbody>
<tr>
<td>Binary search (warm cache)</td>
<td>~40 ns</td>
</tr>
<tr>
<td>Phenoconversion + scoring</td>
<td>~200 ns</td>
</tr>
<tr>
<td>Full tool call → RiskResult</td>
<td>&lt; 500 µs</td>
</tr>
<tr>
<td>Doctor says drug → audio alert</td>
<td>~500 ms</td>
</tr>
</tbody>
</table></div>

<p>The 500ms end-to-end is Gemini's TTFT. The C++ engine is not the bottleneck and was never going to be. We over-engineered the deterministic layer on purpose — because in a safety-critical path, the safety layer should never be the constraint.</p>

<p><strong>Dataset.</strong></p>

<p><code>patients.bin</code> is generated from published allele frequency distributions — gnomAD v4 for EUR, AFR, EAS, AMR populations — using Hardy-Weinberg simulation. No real patient data. No PII. Fully reproducible with a fixed seed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python tools/synthetic_data_generator.py <span class="nt">--patients</span> 10000 <span class="nt">--seed</span> 42 <span class="nt">--out</span> patients.bin
</code></pre>

</div>



<p>10,000 patients. 1.8 MB. Realistic phenotype distributions across 9 CYP enzymes, HLA alleles, G6PD, SLCO1B1, VKORC1, OPRM1.</p>

<p><strong>Demo scenarios — all guaranteed to trigger.</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Patient</th>
<th>Variant</th>
<th>Drug</th>
<th>Alert</th>
</tr>
</thead>
<tbody>
<tr>
<td>99999001</td>
<td>CYP2D6 UM</td>
<td>codeine</td>
<td>Fatal morphine accumulation</td>
</tr>
<tr>
<td>99999002</td>
<td>CYP2C19 PM</td>
<td>clopidogrel</td>
<td>Stent thrombosis</td>
</tr>
<tr>
<td>99999003</td>
<td>CYP2C9 PM + VKORC1</td>
<td>warfarin + amiodarone</td>
<td>Triple hit hemorrhage</td>
</tr>
<tr>
<td>99999004</td>
<td>CYP2D6 EM + paroxetine</td>
<td>codeine</td>
<td><strong>Phenoconversion</strong></td>
</tr>
<tr>
<td>99999005</td>
<td>HLA-B*5701</td>
<td>abacavir</td>
<td>Absolute contraindication</td>
</tr>
<tr>
<td>99999006</td>
<td>TPMT PM</td>
<td>azathioprine</td>
<td>Myelosuppression</td>
</tr>
<tr>
<td>99999007</td>
<td>UGT1A1*28</td>
<td>irinotecan</td>
<td>Severe neutropenia</td>
</tr>
<tr>
<td>99999008</td>
<td>Severe CKD</td>
<td>metformin</td>
<td>Lactic acidosis</td>
</tr>
</tbody>
</table></div>

<p>Patient 99999004 is the one worth watching. Genetically normal. Functionally lethal. Static PGx tools miss it entirely.</p>

<p><strong>What the scoring pipeline covers.</strong></p>

<p>Five steps, executed in sequence, every lookup:</p>

<ol>
<li>
<strong>HLA / G6PD</strong> — absolute contraindications, exits immediately on match</li>
<li>
<strong>Phenoconversion</strong> — genetic → functional phenotype override</li>
<li>
<strong>PGx base score</strong> — CPIC-hardcoded rules per drug × phenotype</li>
<li>
<strong>Physiological deltas</strong> — renal severity, hepatic impairment, age &gt;65 (Beers 2023), pregnancy, QTc &gt;450ms</li>
<li>
<strong>Drug-drug interactions</strong> — 50 critical pairs from CPIC/FDA/CredibleMeds, bidirectional</li>
</ol>

<p>Score is capped at 10. Every point is traceable to a specific guideline reference. No black box.</p>

<p><strong>What it is not.</strong></p>

<p>PharmaShield is a research prototype and a hackathon submission. It is not a certified medical device. It must not be used for real clinical decisions without formal validation and regulatory clearance. The drug interaction database covers ~50 critical pairs — a comprehensive formulary would require Micromedex or Lexicomp. The synthetic dataset covers the major pharmacogenes; some edge cases remain on the production roadmap.</p>

<p>We know exactly where the gaps are. That is what production roadmaps are for.</p>




<h2>
  
  
  Built Together
</h2>

<p>PharmaShield is a joint project. I built the C++23 engine and the backend. <strong>Lilia Ouadah</strong> — PhD candidate in computational genomics at the University Medical Center Groningen (UMCG) — brought the bioinformatics: the star allele calling pipeline, the CPIC guideline implementations, the phenoconversion logic, the allele frequency tables, and all the clinical verification work that makes the science defensible. We built the frontend together.</p>

<p>The project does not exist without her.</p>




<h2>
  
  
  Stack
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Technology</th>
</tr>
</thead>
<tbody>
<tr>
<td>AI model</td>
<td>Gemini 2.5 Flash Live</td>
</tr>
<tr>
<td>AI SDK</td>
<td>Google GenAI Python SDK</td>
</tr>
<tr>
<td>Backend</td>
<td>FastAPI + uvloop + WebSockets</td>
</tr>
<tr>
<td>PGx engine</td>
<td>C++23, pybind11, <code>mmap</code>, <code>std::ranges::equal_range</code>
</td>
</tr>
<tr>
<td>Cloud compute</td>
<td>Google Cloud Run (gen2)</td>
</tr>
<tr>
<td>Data storage</td>
<td>Google Cloud Storage</td>
</tr>
<tr>
<td>Frontend</td>
<td>Vanilla HTML5/JS, Web Audio API</td>
</tr>
</tbody>
</table></div>




<p><strong>GitHub:</strong> <a href="https://github.com/riyaneel/pharmashield" rel="noopener noreferrer">PharmaShield</a></p>

<p><em>#GeminiLiveAgentChallenge</em></p>

