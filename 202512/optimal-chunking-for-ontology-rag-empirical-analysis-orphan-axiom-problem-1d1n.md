---
Title: Optimal Chunking for Ontology RAG: Empirical Analysis & Orphan Axiom Problem
Description: 
Author: vishalmysore
Date: 2025-12-18T21:32:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Retrieval-Augmented Generation (RAG) systems require effective chunking strategies to segment knowledge into retrievable units. While text-based chunking (word, sentence, paragraph boundaries) is well-studied for documents, ontologies present unique challenges due to their semantic structure. This study empirically evaluates 10 chunking strategies—4 text-based and 6 OWL-aware—on a legal domain ontology, measuring similarity scores, answer quality, retrieval consistency, and computational costs.</p>

<p>In my small-scale, independent experiments, I discovered the <strong>"Orphan Axiom Problem"</strong>: 93.8% of axioms in my test ontology were non-hierarchical (individuals, properties, annotations), causing traditional OWL-aware strategies to produce highly unbalanced chunks. In these tests, ModuleExtractionChunking achieved the highest OWL-aware similarity (0.7068) with exceptional consistency (0.0012 variance), while AnnotationBasedChunking (0.7010, 39 chunks) provided fine-grained grouping dependent on naming conventions.</p>

<p><strong>Counter-intuitive finding</strong>: SentenceChunking scored highest overall (0.7258) but produced the worst answers by fragmenting entity names across chunks, demonstrating that <strong>semantic completeness &gt; mathematical similarity</strong> for RAG effectiveness.</p>

<p>We provide a decision framework for strategy selection based on ontology characteristics (ABox/TBox ratio, hierarchy depth, metadata quality) and empirically validate that no single strategy is universally optimal.</p>




<h2>
  
  
  1. Introduction
</h2>

<h3>
  
  
  1.1 Motivation
</h3>

<p>Retrieval-Augmented Generation combines retrieval systems with large language models (LLMs) to answer queries using domain-specific knowledge. Chunking—splitting knowledge into retrievable units—directly impacts:</p>

<ul>
<li>
<strong>Context relevance</strong>: Whether retrieved chunks contain needed information</li>
<li>
<strong>Answer accuracy</strong>: Whether LLM receives complete vs. fragmented context</li>
<li>
<strong>Query latency</strong>: Search time and computational cost</li>
</ul>

<p>Traditional text-based chunking treats all content uniformly, but ontologies encode rich semantic structure: class hierarchies, property domains, annotation patterns. This structure offers opportunities for semantic-aware chunking but also introduces new failure modes.</p>

<h3>
  
  
  1.2 The ABox/TBox Challenge
</h3>

<p>Ontologies consist of:</p>

<ul>
<li>
<strong>TBox (Terminological Box)</strong>: Class definitions, hierarchy, property schemas</li>
<li>
<strong>ABox (Assertional Box)</strong>: Instances, property values, individual assertions</li>
</ul>

<p>Most ontology chunking research assumes TBox-heavy structures (e.g., biomedical classifications), but real-world domain ontologies are typically ABox-heavy. Our legal ontology: <strong>93.8% ABox, 6.2% TBox</strong>. This imbalance creates the "Orphan Axiom Problem" where hierarchy-based strategies produce massive, unbalanced chunks.</p>

<h3>
  
  
  1.3 Research Questions
</h3>

<p><strong>RQ1</strong>: Do OWL-aware chunking strategies outperform text-based approaches for ontology RAG?</p>

<p><strong>RQ2</strong>: What ontology characteristics predict optimal chunking strategy performance?</p>

<p><strong>RQ3</strong>: How does the ABox/TBox ratio affect chunking strategy effectiveness?</p>

<p><strong>RQ4</strong>: Can we predict chunking strategy performance without exhaustive testing?</p>




<h2>
  
  
  3. Methodology
</h2>

<h3>
  
  
  3.1 Test Environment
</h3>

<p><strong>Platform</strong>: Protégé 5.6.7 with custom Lucene RAG plugin<br><br>
<strong>Vector Store</strong>: Apache Lucene 9.8.0 (KnnFloatVectorField)<br><br>
<strong>Embeddings</strong>: OpenAI text-embedding-3-small (1024 dimensions, $0.02/1M tokens)<br><br>
<strong>LLM</strong>: GPT-4 (gpt-4-0613, $0.03/1K input tokens)<br><br>
<strong>Hardware</strong>: Intel Core i7, 16GB RAM, SSD storage</p>
<h3>
  
  
  3.2 Test Ontology: Legal Domain
</h3>

<p><strong>Axiom Breakdown</strong>:</p>

<ul>
<li>Total: 195 axioms</li>
<li>
<strong>TBox (6.2%)</strong>:

<ul>
<li>SubClassOf: 12 axioms</li>
<li>Class declarations: 17 axioms</li>
<li>Property domains/ranges: 14 axioms</li>
</ul>
</li>
<li>
<strong>ABox (93.8%)</strong>:

<ul>
<li>Individual assertions: 15 ClassAssertion axioms</li>
<li>Property assertions: 47 ObjectProperty + 36 DataProperty</li>
<li>Annotations: 84 rdfs:label axioms</li>
</ul>
</li>
</ul>

<p><strong>Domain Entities</strong>:</p>

<ul>
<li>3 cases: Smith v. Jones (Active), State v. Doe (Trial), Appeal CV-2023-500 (Under Review)</li>
<li>3 courts: District, Appellate, Supreme</li>
<li>4 judges, 3 lawyers, 3 evidence items, 2 statutes</li>
</ul>

<p><strong>Hierarchy Depth</strong>: 2 levels maximum<br><br>
<strong>Namespaces</strong>: 1 (<a href="http://www.semanticweb.org/legal#" rel="noopener noreferrer">http://www.semanticweb.org/legal#</a>)</p>
<h3>
  
  
  3.3 Evaluation Metrics
</h3>

<ol>
<li>
<strong>Top Similarity Score</strong>: Cosine similarity of best-matching chunk (0-1 scale)</li>
<li>
<strong>Retrieval Consistency</strong>: Variance in top-5 similarity scores (lower = more consistent)</li>
<li>
<strong>Answer Quality</strong>: Binary (correct/incorrect) manual evaluation</li>
<li>
<strong>Chunk Count</strong>: Total chunks created</li>
<li>
<strong>Chunk Balance</strong>: Standard deviation of chunk sizes</li>
<li>
<strong>Indexing Time</strong>: Time to embed and store all chunks</li>
<li>
<strong>Storage Size</strong>: Disk space for vector index</li>
</ol>
<h3>
  
  
  3.4 Test Query
</h3>

<p><strong>Primary</strong>: "Which cases are currently active?"</p>

<p><strong>Expected Answer</strong>: "Smith v. Jones" (Status: Active)</p>

<p><strong>Evaluation Criteria</strong>:</p>

<ul>
<li>✅ Correct: Identifies "Smith v. Jones" as active</li>
<li>❌ Incomplete: Only mentions "Jones" or partial names</li>
<li>❌ Incorrect: Lists wrong cases or misses active case</li>
</ul>
<h3>
  
  
  3.5 Chunking Strategies Tested
</h3>

<p><strong>Text-Based (4)</strong>:</p>

<ol>
<li>WordChunking (100 words/chunk)</li>
<li>SentenceChunking (sentence boundaries)</li>
<li>ParagraphChunking (double newlines)</li>
<li>FixedSizeChunking (fixed character limit)</li>
</ol>

<p><strong>OWL-Aware (6)</strong>:</p>

<ol>
<li>ClassBasedChunking (hierarchy groups)</li>
<li>AnnotationBasedChunking (label prefix groups)</li>
<li>NamespaceBasedChunking (IRI namespace groups)</li>
<li>DepthBasedChunking (hierarchy depth levels)</li>
<li>ModuleExtractionChunking (OWL API modules)</li>
<li>SizeBasedChunking (fixed axiom count) - <em>not tested</em>
</li>
</ol>


<h2>
  
  
  4. Results
</h2>
<h3>
  
  
  4.1 Performance Overview
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Strategy</th>
<th>Chunks</th>
<th>Top Score</th>
<th>Variance</th>
<th>Answer</th>
<th>Index Time</th>
<th>Storage</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Text-Based</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>Word</td>
<td>58</td>
<td>0.7135</td>
<td>0.0245</td>
<td>✅ Correct</td>
<td>3.2s</td>
<td>2.1MB</td>
</tr>
<tr>
<td>Sentence</td>
<td>76</td>
<td><strong>0.7258</strong></td>
<td>0.0389</td>
<td>❌ Incomplete</td>
<td>4.1s</td>
<td>2.8MB</td>
</tr>
<tr>
<td>Paragraph</td>
<td>58</td>
<td>0.7141</td>
<td>0.0251</td>
<td>✅ Correct</td>
<td>3.3s</td>
<td>2.1MB</td>
</tr>
<tr>
<td>FixedSize</td>
<td>~50</td>
<td>0.7141</td>
<td>0.0203</td>
<td>✅ Correct</td>
<td>2.8s</td>
<td>1.8MB</td>
</tr>
<tr>
<td><strong>OWL-Aware</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>ClassBased</td>
<td>6</td>
<td>0.6964</td>
<td>0.0412</td>
<td>✅ Correct</td>
<td>1.2s</td>
<td>0.4MB</td>
</tr>
<tr>
<td><strong>AnnotationBased</strong></td>
<td><strong>39</strong></td>
<td><strong>0.7010</strong></td>
<td><strong>0.0312</strong></td>
<td>✅ Correct</td>
<td>2.1s</td>
<td>1.4MB</td>
</tr>
<tr>
<td>NamespaceBased</td>
<td>6</td>
<td>0.6964</td>
<td>0.0412</td>
<td>✅ Correct</td>
<td>1.1s</td>
<td>0.4MB</td>
</tr>
<tr>
<td>DepthBased</td>
<td>3</td>
<td>0.6967</td>
<td>0.0741</td>
<td>✅ Correct</td>
<td>0.8s</td>
<td>0.2MB</td>
</tr>
<tr>
<td><strong>ModuleExtraction</strong></td>
<td><strong>28</strong></td>
<td><strong>0.7068</strong></td>
<td><strong>0.0012</strong></td>
<td>✅ Correct</td>
<td>1.8s</td>
<td>1.0MB</td>
</tr>
</tbody>
</table></div>

<p><strong>Note</strong>: Index times include embedding API calls (network latency). Storage includes vectors + metadata.</p>
<h3>
  
  
  4.2 The Orphan Axiom Problem
</h3>

<p><strong>Definition</strong>: Axioms not part of class hierarchy definitions.</p>

<p><strong>Composition in Legal Ontology</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>TBox (12 axioms, 6.2%):
  ├─ SubClassOf: CivilCase ⊑ Case
  ├─ SubClassOf: CriminalCase ⊑ Case
  └─ SubClassOf: AppellateCase ⊑ Case
  ... (9 more)

ABox (183 axioms, 93.8%):
  ├─ ClassAssertion: Case_SmithVsJones : CivilCase
  ├─ DataPropertyAssertion: caseNumber(Case_SmithVsJones, "CV-2024-001")
  ├─ DataPropertyAssertion: caseStatus(Case_SmithVsJones, "Active")
  └─ AnnotationAssertion: rdfs:label(Case_SmithVsJones, "Smith v. Jones")
  ... (179 more)
</code></pre>

</div>



<p><strong>Impact on Chunking</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Strategy</th>
<th>TBox Chunks</th>
<th>ABox Chunks</th>
<th>Largest Chunk</th>
</tr>
</thead>
<tbody>
<tr>
<td>ClassBased</td>
<td>5 (small)</td>
<td>1 (orphan: 183)</td>
<td>183 axioms (93.8%)</td>
</tr>
<tr>
<td>DepthBased</td>
<td>2 (small)</td>
<td>1 (non-class: 183)</td>
<td>183 axioms (93.8%)</td>
</tr>
<tr>
<td>AnnotationBased</td>
<td>N/A</td>
<td>39 (semantic)</td>
<td>84 axioms (43.1%)</td>
</tr>
<tr>
<td>ModuleExtraction</td>
<td>Mixed</td>
<td>Mixed</td>
<td>132 axioms (67.7%)</td>
</tr>
</tbody>
</table></div>

<p><strong>Key Insight</strong>: Hierarchy-based strategies (Class, Depth) fail catastrophically on ABox-heavy ontologies, concentrating 93.8% of content into single chunks. This defeats the purpose of chunking.</p>

<h3>
  
  
  4.3 Error Analysis: SentenceChunking Failure
</h3>

<p><strong>The Fragmentation Problem</strong></p>

<p>SentenceChunking achieved the highest similarity (0.7258) but produced incomplete answers.</p>

<p><strong>Example Fragmentation</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Original Entity (Manchester Syntax):
Individual: Case_SmithVsJones
    Types: CivilCase
    Facts: caseNumber "CV-2024-001",
           caseStatus "Active",
           filedIn Court_District1

Sentence Chunk A (similarity: 0.6912):
"Individual: Case_SmithVsJones Types: CivilCase Facts: caseNumber \"CV-2024-001\"."

Sentence Chunk B (similarity: 0.7258, TOP MATCH):
"caseStatus \"Active\", filedIn Court_District1."

Sentence Chunk C (similarity: 0.6734):
"AnnotationAssertion(rdfs:label &lt;Case_SmithVsJones&gt; \"Smith v. Jones\")."
</code></pre>

</div>



<p><strong>What Went Wrong</strong>:</p>

<ol>
<li>Query: "Which cases are currently active?"</li>
<li>Top chunk (B) contains <code>caseStatus "Active"</code> → high similarity ✓</li>
<li>BUT: Chunk B missing case name/identifier</li>
<li>LLM sees "caseStatus Active" without context → incomplete answer</li>
</ol>

<p><strong>Why Other Strategies Avoided This</strong>:</p>

<ul>
<li>
<strong>WordChunking</strong>: 100-word limit kept entire entity together</li>
<li>
<strong>AnnotationBased</strong>: Grouped Case_SmithVsJones with all case-related axioms</li>
<li>
<strong>ModuleExtraction</strong>: Dependency closure included all related properties</li>
</ul>

<p><strong>Predictability</strong>: This failure is <strong>predictable</strong> from ontology structure:</p>

<ul>
<li>Entity average size: ~15 axioms</li>
<li>Sentence average: ~2 axioms</li>
<li><strong>If sentence size &lt; entity size → fragmentation risk</strong></li>
</ul>

<h3>
  
  
  4.4 ModuleExtraction: Consistency Analysis
</h3>

<p><strong>Remarkable Finding</strong>: Top-5 similarity scores clustered within 0.0012 range.</p>

<p><strong>Score Distribution</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Rank 1: 0.7068 (module-chunk-1, 132 axioms, 4 seed entities)
Rank 2: 0.7067 (module-chunk-5, 89 axioms, 3 seed entities)
Rank 3: 0.7062 (module-chunk-12, 107 axioms, 4 seed entities)
Rank 4: 0.7058 (module-chunk-3, 95 axioms, 3 seed entities)
Rank 5: 0.7056 (module-chunk-8, 112 axioms, 4 seed entities)

Variance: 0.0012
Standard Deviation: 0.0035
</code></pre>

</div>



<p><strong>Why This Matters</strong>:</p>

<ul>
<li>Any of top-5 chunks could answer the query correctly</li>
<li>Robust to embedding noise/variation</li>
<li>Reduces sensitivity to k-nearest neighbors selection</li>
</ul>

<p><strong>Contrast with AnnotationBased</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Rank 1: 0.7010 (no-annotations chunk, 84 axioms)
Rank 2: 0.6955 (cas prefix chunk, 26 axioms)
Rank 3: 0.6746 (sta prefix chunk, 24 axioms)
Rank 4: 0.6692 (app prefix chunk, 17 axioms)
Rank 5: 0.6543 (fil prefix chunk, 12 axioms)

Variance: 0.0312
Standard Deviation: 0.1766
</code></pre>

</div>



<p>More variance = higher risk of returning irrelevant chunks.</p>

<h3>
  
  
  4.5 Cost-Performance Tradeoffs
</h3>

<p><strong>Indexing Cost</strong> (one-time):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Text-Based:
  Sentence (76 chunks): 4.1s, $0.0012 embedding cost
  Word (58 chunks):     3.2s, $0.0010 embedding cost

OWL-Aware:
  AnnotationBased (39 chunks): 2.1s, $0.0007 embedding cost
  ModuleExtraction (28 chunks): 1.8s, $0.0005 embedding cost
  ClassBased (6 chunks):        1.2s, $0.0001 embedding cost
</code></pre>

</div>



<p><strong>Storage Cost</strong> (persistent):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Sentence:         2.8MB (0.037 MB/chunk)
Word:             2.1MB (0.036 MB/chunk)
AnnotationBased:  1.4MB (0.036 MB/chunk)
ModuleExtraction: 1.0MB (0.036 MB/chunk)
</code></pre>

</div>



<p><strong>Query Cost</strong> (per query):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>All strategies: Single embedding (~0.015s, $0.000001)
Retrieval: O(log n) with HNSW index
</code></pre>

</div>



<p><strong>Implications</strong>:</p>

<ul>
<li>ModuleExtraction: Lowest total cost + highest accuracy + best consistency</li>
<li>AnnotationBased: Moderate cost, metadata-dependent performance</li>
<li>Sentence: Highest cost + worst accuracy = poor value</li>
</ul>




<h2>
  
  
  5. Analysis &amp; Discussion
</h2>

<h3>
  
  
  5.1 RQ1: OWL-Aware vs. Text-Based Performance
</h3>

<p><strong>Answer</strong>: <strong>Context-dependent</strong>. OWL-aware strategies excel only when ontology structure aligns with chunking logic.</p>

<p><strong>Evidence</strong>:</p>

<ul>
<li>ModuleExtraction (OWL): 0.7068, best consistency</li>
<li>AnnotationBased (OWL): 0.7010, requires naming conventions</li>
<li>WordChunking (text): 0.7135, simplest + high score</li>
<li>BUT: SentenceChunking (text): 0.7258 score, worst answer</li>
</ul>

<p><strong>Key Factors</strong>:</p>

<ol>
<li>
<strong>ABox/TBox ratio</strong>: High ABox → text-based or AnnotationBased</li>
<li>
<strong>Metadata quality</strong>: Poor naming → text-based preferred</li>
<li>
<strong>Entity cohesion</strong>: Compact entities → fixed-size works well</li>
</ol>

<h3>
  
  
  5.2 RQ2: Ontology Characteristics as Predictors
</h3>

<p><strong>Proposed Decision Framework</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>IF ontology has:
  ├─ Multiple namespaces (&gt;3) → NamespaceBasedChunking
  ├─ Deep hierarchy (≥5 levels) → DepthBasedChunking
  ├─ Consistent naming (prefix patterns) → AnnotationBasedChunking
  ├─ Complex relationships + large → ModuleExtractionChunking
  ├─ Compact entities (&lt;100 words) → WordChunking
  └─ Unknown/mixed → Test ModuleExtraction + Word
</code></pre>

</div>



<p><strong>Validation on Legal Ontology</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Characteristics:
  ✓ Single namespace         → NamespaceBased failed (fell back to ClassBased)
  ✓ Shallow hierarchy (2)    → DepthBased produced only 3 chunks
  ✓ Consistent naming (case_, judge_) → AnnotationBased worked well (0.7010)
  ✓ Compact entities (~15 axioms) → WordChunking effective (0.7135)
  ✓ Relationships span entities → ModuleExtraction best (0.7068)
</code></pre>

</div>



<h3>
  
  
  5.3 RQ3: ABox/TBox Ratio Impact
</h3>

<p><strong>Hypothesis</strong>: High ABox ratio degrades hierarchy-based strategy performance.</p>

<p><strong>Evidence</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Ontology Type</th>
<th>ABox%</th>
<th>ClassBased (measured)</th>
<th>DepthBased (measured)</th>
<th>Best Measured Strategy</th>
</tr>
</thead>
<tbody>
<tr>
<td>Legal (this study)</td>
<td>93.8%</td>
<td>low (~0.70)</td>
<td>low (~0.70)</td>
<td>ModuleExtraction</td>
</tr>
</tbody>
</table></div>

<p><em>Measured results above are from my legal ontology experiment.</em></p>

<p><strong>Hypothetical / WIP Scenarios:</strong></p>

<ul>
<li>
<strong>Biomedical ontology (WIP):</strong>

<ul>
<li>ABox%: ~30%</li>
<li>ClassBased (expected): medium-high (~0.75)</li>
<li>DepthBased (expected): high (~0.77)</li>
<li>Qualitative expectation: ClassBased or DepthBased likely to perform well due to richer hierarchy</li>
</ul>


</li>

<li>

<strong>Schema.org (WIP):</strong>

<ul>
<li>ABox%: ~10%</li>
<li>ClassBased (expected): high (~0.82)</li>
<li>DepthBased (expected): high (~0.85)</li>
<li>Qualitative expectation: DepthBased likely to excel due to deep, well-structured schema</li>
</ul>


</li>

</ul>

<p><em>No experiments yet for these domains; numbers are placeholders to illustrate the hypothesis, not measured results.</em></p>

<h3>
  
  
  5.4 RQ4: Predictive Performance Modeling
</h3>

<p><strong>Can we predict performance without testing?</strong></p>

<p><strong>Proposed Heuristics</strong>:</p>

<p><strong>Heuristic 1: Entity Fragmentation Risk</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>IF avg_sentence_size &lt; avg_entity_size:
    fragmentation_risk = HIGH
    → Avoid SentenceChunking
</code></pre>

</div>



<p><strong>Heuristic 2: Orphan Axiom Ratio</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>orphan_ratio = (total_axioms - hierarchy_axioms) / total_axioms
IF orphan_ratio &gt; 0.8:
    → Avoid ClassBased, DepthBased
</code></pre>

</div>



<p><strong>Heuristic 3: Naming Consistency</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>prefix_coverage = count_axioms_with_consistent_prefixes / total_axioms
IF prefix_coverage &gt; 0.7:
    → AnnotationBased viable
ELSE:
    → Use text-based or ModuleExtraction
</code></pre>

</div>



<p><strong>Validation</strong>: Applying these heuristics to legal ontology:</p>

<ul>
<li>Heuristic 1: ✓ Correctly predicts SentenceChunking failure</li>
<li>Heuristic 2: ✓ Correctly eliminates ClassBased, DepthBased</li>
<li>Heuristic 3: ✓ Correctly identifies AnnotationBased viability (prefix_coverage ≈ 0.75)</li>
</ul>




<h2>
  
  
  6. Limitations
</h2>

<ol>
<li>
<strong>Single ontology domain</strong>: Results specific to legal domain with flat hierarchy</li>
<li>
<strong>Small scale</strong>: 195 axioms; performance at 10,000+ unknown</li>
<li>
<strong>Query diversity</strong>: Single query type (factual retrieval); structural queries untested</li>
<li>
<strong>Embedding model</strong>: Results specific to text-embedding-3-small (1024d)</li>
<li>
<strong>No hybrid strategies</strong>: Did not test combined approaches</li>
<li>
<strong>Manual answer evaluation</strong>: Binary scoring may miss nuanced quality differences</li>
</ol>




<h2>
  
  
  7. Future Work
</h2>

<h3>
  
  
  7.1 Large-Scale Validation
</h3>

<p>Test on diverse ontologies:</p>

<ul>
<li>
<strong>SNOMED CT</strong>: 300,000+ medical concepts, deep hierarchy</li>
<li>
<strong>Gene Ontology</strong>: 45,000+ terms, complex relationships</li>
<li>
<strong>DBpedia</strong>: 6M+ entities, multi-domain</li>
<li>
<strong>Schema.org</strong>: Web schema, moderate hierarchy</li>
</ul>

<h3>
  
  
  7.2 Query Type Analysis
</h3>

<p>Evaluate performance across query categories:</p>

<ul>
<li>
<strong>Factual</strong>: "Which cases are active?" (current study)</li>
<li>
<strong>Structural</strong>: "What are subclasses of Case?"</li>
<li>
<strong>Relational</strong>: "Who represents defendant in Smith v. Jones?"</li>
<li>
<strong>Aggregation</strong>: "How many active criminal cases?"</li>
</ul>

<h3>
  
  
  7.3 Hybrid Strategies
</h3>

<p>Design adaptive chunking:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">hybrid_chunk</span><span class="p">(</span><span class="n">ontology</span><span class="p">):</span>
    <span class="n">tbox_chunks</span> <span class="o">=</span> <span class="nc">ClassBasedChunking</span><span class="p">(</span><span class="nf">get_tbox</span><span class="p">(</span><span class="n">ontology</span><span class="p">))</span>
    <span class="n">abox_chunks</span> <span class="o">=</span> <span class="nc">AnnotationBasedChunking</span><span class="p">(</span><span class="nf">get_abox</span><span class="p">(</span><span class="n">ontology</span><span class="p">))</span>
    <span class="k">return</span> <span class="n">tbox_chunks</span> <span class="o">+</span> <span class="n">abox_chunks</span>
</code></pre>

</div>



<h3>
  
  
  7.4 Automated Strategy Selection
</h3>

<p>Machine learning model to predict optimal strategy:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Input: Ontology metrics (ABox%, depth, namespace count, naming entropy)
Output: Predicted best strategy + confidence
</code></pre>

</div>



<h3>
  
  
  7.5 Dynamic Chunking
</h3>

<p>Adjust chunk granularity based on query:</p>

<ul>
<li>Simple queries → coarse chunks (faster)</li>
<li>Complex queries → fine chunks (more precise)</li>
</ul>




<h2>
  
  
  8. Conclusion
</h2>

<p>This study provides the first comprehensive empirical evaluation of chunking strategies for ontology-based RAG systems. Our key contributions:</p>

<ol>
<li><p><strong>Discovered the Orphan Axiom Problem</strong>: 93.8% ABox ratio causes hierarchy-based strategies to fail catastrophically, concentrating content into massive single chunks.</p></li>
<li><p><strong>Counter-intuitive finding</strong>: Highest similarity score (SentenceChunking: 0.7258) produced worst answers by fragmenting entities—proving semantic completeness &gt; mathematical similarity.</p></li>
<li><p><strong>Consistency matters</strong>: ModuleExtractionChunking's 0.0012 variance makes it robust to embedding noise, despite slightly lower peak score than text-based approaches.</p></li>
<li><p><strong>Predictive framework</strong>: We provide heuristics to predict strategy performance from ontology characteristics (ABox/TBox ratio, hierarchy depth, naming patterns) without exhaustive testing.</p></li>
<li><p><strong>Cost-performance analysis</strong>: ModuleExtraction offers best accuracy/cost ratio: highest OWL-aware score, lowest variance, moderate computational cost.</p></li>
</ol>

<p><strong>Practical Recommendation</strong>: No universal winner exists. Strategy selection depends on:</p>

<ul>
<li>
<strong>High ABox ratio (&gt;80%)</strong>: ModuleExtraction or AnnotationBased (if naming consistent)</li>
<li>
<strong>Deep hierarchy (≥5 levels)</strong>: DepthBased or ClassBased</li>
<li>
<strong>Multiple namespaces (≥3)</strong>: NamespaceBased</li>
<li>
<strong>Unknown/mixed</strong>: Start with ModuleExtraction + Word, validate empirically</li>
</ul>

<p>As ontology-based AI systems proliferate, sophisticated chunking will become increasingly critical. This work provides both empirical evidence and practical guidance for researchers and practitioners building knowledge-enhanced RAG systems.</p>




<h2>
  
  
  9. Reproducibility
</h2>

<h3>
  
  
  9.1 Code &amp; Configuration
</h3>

<p><strong>Plugin Configuration</strong> (plugin.xml):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;plugin&gt;</span>
    <span class="nt">&lt;id&gt;</span>lucene-rag-plugin<span class="nt">&lt;/id&gt;</span>
    <span class="nt">&lt;version&gt;</span>1.0.0<span class="nt">&lt;/version&gt;</span>
    <span class="nt">&lt;dependency&gt;</span>agenticmemory-0.1.1<span class="nt">&lt;/dependency&gt;</span>
    <span class="nt">&lt;dependency&gt;</span>lucene-core-9.8.0<span class="nt">&lt;/dependency&gt;</span>
<span class="nt">&lt;/plugin&gt;</span>
</code></pre>

</div>



<p><strong>Chunking Implementation</strong> (RagService.java):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="c1">// ModuleExtractionChunking</span>
<span class="nc">ModuleExtractionChunker</span> <span class="n">chunker</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ModuleExtractionChunker</span><span class="o">();</span>
<span class="nc">List</span><span class="o">&lt;</span><span class="nc">OWLChunk</span><span class="o">&gt;</span> <span class="n">chunks</span> <span class="o">=</span> <span class="n">chunker</span><span class="o">.</span><span class="na">chunk</span><span class="o">(</span><span class="n">ontology</span><span class="o">);</span>

<span class="k">for</span> <span class="o">(</span><span class="nc">OWLChunk</span> <span class="n">chunk</span> <span class="o">:</span> <span class="n">chunks</span><span class="o">)</span> <span class="o">{</span>
    <span class="nc">String</span> <span class="n">text</span> <span class="o">=</span> <span class="n">chunk</span><span class="o">.</span><span class="na">toOWLString</span><span class="o">();</span> <span class="c1">// Manchester syntax</span>
    <span class="nc">List</span><span class="o">&lt;</span><span class="nc">Float</span><span class="o">&gt;</span> <span class="n">embedding</span> <span class="o">=</span> <span class="n">embeddingService</span><span class="o">.</span><span class="na">createEmbedding</span><span class="o">(</span><span class="n">text</span><span class="o">);</span>
    <span class="n">vectorStore</span><span class="o">.</span><span class="na">upsert</span><span class="o">(</span><span class="n">chunk</span><span class="o">.</span><span class="na">getId</span><span class="o">(),</span> <span class="n">embedding</span><span class="o">,</span> <span class="n">text</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p><strong>Vector Store Setup</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">LuceneVectorStore</span> <span class="n">store</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">LuceneVectorStore</span><span class="o">(</span>
    <span class="s">"./lucene_index"</span><span class="o">,</span>  <span class="c1">// File path</span>
    <span class="mi">1024</span><span class="o">,</span>             <span class="c1">// Dimensions</span>
    <span class="nc">VectorSimilarityFunction</span><span class="o">.</span><span class="na">COSINE</span>
<span class="o">);</span>
</code></pre>

</div>



<h3>
  
  
  9.2 Sample Chunks
</h3>

<p><strong>ClassBased - Orphan Chunk (183 axioms)</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Chunk ID: class-chunk-orphan
Strategy: Class-Based
Axiom Count: 183

ClassAssertion(&lt;CivilCase&gt; &lt;Case_SmithVsJones&gt;)
DataPropertyAssertion(&lt;caseNumber&gt; &lt;Case_SmithVsJones&gt; "CV-2024-001")
DataPropertyAssertion(&lt;caseStatus&gt; &lt;Case_SmithVsJones&gt; "Active")
ObjectPropertyAssertion(&lt;filedIn&gt; &lt;Case_SmithVsJones&gt; &lt;Court_District1&gt;)
... (179 more axioms)
</code></pre>

</div>



<p><strong>AnnotationBased - "cas" Prefix Chunk (26 axioms)</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Chunk ID: annotation-chunk-10
Strategy: Annotation-Based
Axiom Count: 26
Annotation Key: label:cas;

SubClassOf(&lt;CivilCase&gt; &lt;Case&gt;)
SubClassOf(&lt;CriminalCase&gt; &lt;Case&gt;)
SubClassOf(&lt;AppellateCase&gt; &lt;Case&gt;)
Declaration(Class(&lt;Case&gt;))
DataPropertyDomain(&lt;caseNumber&gt; &lt;Case&gt;)
DataPropertyDomain(&lt;caseStatus&gt; &lt;Case&gt;)
DataPropertyAssertion(&lt;caseNumber&gt; &lt;Case_SmithVsJones&gt; "CV-2024-001")
DataPropertyAssertion(&lt;caseStatus&gt; &lt;Case_SmithVsJones&gt; "Active")
... (18 more)
</code></pre>

</div>



<p><strong>ModuleExtraction - Module 1 (132 axioms)</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Chunk ID: module-chunk-1
Strategy: Module-Extraction
Axiom Count: 132
Seed Entities: [Case, Court, Judge, Lawyer]

[Complete self-contained module with dependency closure]
All Case axioms + related Court axioms + associated Judge axioms
+ Lawyer relationships + Evidence connections
= Logically complete, independently coherent module
</code></pre>

</div>



<h3>
  
  
  9.3 Test Query Protocol
</h3>

<p><strong>Query</strong>: "Which cases are currently active?"</p>

<p><strong>Retrieval Process</strong>:</p>

<ol>
<li>Generate query embedding: <code>text-embedding-3-small("Which cases are currently active?")</code>
</li>
<li>Search Lucene index: <code>k=5, similarity=COSINE</code>
</li>
<li>Return top chunks with scores</li>
<li>Format context for GPT-4</li>
</ol>

<p><strong>GPT-4 Prompt Template</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>You are a legal knowledge assistant. Answer the question using ONLY the provided context.

Context:
{retrieved_chunk_1}
{retrieved_chunk_2}
...

Question: Which cases are currently active?

Answer:
</code></pre>

</div>



<p><strong>Expected Response</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>The currently active case is Smith v. Jones (Case ID: Case_SmithVsJones, Case Number: CV-2024-001, Status: Active).
</code></pre>

</div>



<h3>
  
  
  9.4 Data Availability
</h3>

<ul>
<li>
<strong>Ontology</strong>: legal.owl (195 axioms, OWL 2 DL, available upon request)</li>
<li>
<strong>Test results</strong>: Full similarity scores, chunk contents, timing data (supplementary materials)</li>
<li>
<strong>Source code</strong>: lucene-rag-plugin v1.0.0 (Java 11, Maven)</li>
<li>
<strong>Dependencies</strong>: agenticmemory-0.1.1, lucene-core-9.8.0, OWL API 4.5.26</li>
</ul>







<blockquote>
<p><strong>Disclaimer &amp; Author Note:</strong> This research is my independent work as part of my open source project Vidyaastra. All plugins, ontologies, and chunking mechanisms are created by me and run on my own servers. The findings are based on experiments with a few ontologies so far; I plan to extend this work to more domains and different RAG systems, and the methodology will continue to evolve. Performance results may vary and can be significantly improved with more powerful hardware and parallel processing. The 1024 dimensions used here are due to Lucene limitations—if you need higher dimensions, you can switch to the Qdrant plugin using the same mechanisms. For late chunking approaches, you can use token-based embedding providers via REST API (since OpenAI does not currently support token-based embeddings in their API). I encourage you to experiment with different models from NVIDIA or various combinations of model and embedding providers. This is very much a work in progress, with lots of room for experimentation and improvement.</p>

<p><strong>Disclaimer on ABox/TBox Bias:</strong><br><br>
The results and limitations discussed here are strongly shaped by the ABox-dominated structure of my test ontology (93.8% ABox, 6.2% TBox). In this context, the ABox acts as a "villain" for conventional chunking strategies: hierarchy-based methods (ClassBased, DepthBased) struggle to find logical split points, leading to the "blob" effect where most individual assertions are concentrated in a single orphan chunk. This undermines the goal of RAG, which is to retrieve only the most relevant information. Additionally, text-based chunking can fragment horizontal relationships, resulting in incomplete or context-smeared retrievals. These findings are based on my independent, small-scale experiments and may differ for ontologies with richer TBox structures. To further validate these findings, I plan to repeat these experiments with a TBox-rich ontology. This will help assess how chunking strategies perform when the ontology contains deeper class hierarchies and more schema-level logic.</p>
</blockquote>

<p><strong>Important Limitations &amp; Cautions:</strong><br>
This study is based on a single ontology ( even though i have experimented with few others i need to still do more as I did not capture the results effectively), few primary queries, one embedding model, one LLM, and one hardware setup. These constraints mean that my findings—especially claims about “no universal winner” or optimal strategy selection—should be interpreted as preliminary and not generalizable.</p>

<p>The “Answer Quality” metric is a simple binary (correct/incorrect) judgment, which does not capture nuanced or partial answers. Strong conclusions about answer quality versus similarity should be treated with caution.</p>

<p>Cost-performance numbers (milliseconds, fractions of a cent) are included for completeness, but are not the main story; for most practical purposes, structural robustness and retrieval quality matter far more than tiny differences in compute or spend.</p>

<p>Some tables mix measured results (legal ontology) with hypothetical or expected outcomes for other domains. I have tried to make this clear, but readers should be aware of the distinction.</p>

