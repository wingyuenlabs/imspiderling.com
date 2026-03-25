---
Title: How I Built an AI Compliance System for Charter Aviation Using RAG and Pinecone
Description: 
Author: Tilak Raj
Date: 2026-03-25T22:06:30.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." alt="Uploading image" width="800" height="400"></a>Building RAG for compliance-critical domains is not the same as building RAG for a<br>
general-purpose chatbot. When the outputs affect an operator's certification status, wrong<br>
answers have real consequences. This is the full architecture walkthrough of Navlyt — the<br>
compliance operating system I built for charter aviation operators.<br>
Navlyt tracks FAA, Transport Canada, and EASA regulatory requirements for small charter aviation<br>
operators. It answers compliance questions, monitors obligation status, and generates required<br>
documentation. The challenge: regulatory documents are dense, cross-referenced, and version-controlled<br>
in ways that break standard RAG approaches.<br>
This article covers the complete technical implementation — chunking strategy, retrieval architecture,<br>
answer generation with citations, and the accuracy validation approach I use for a compliance-critical<br>
domain.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnsrgqcqtf1bwknovhumj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnsrgqcqtf1bwknovhumj.png" alt=" " width="800" height="446"></a><br>
The Problem With Standard RAG for Regulatory Content<br>
Regulatory documents have properties that make standard paragraph-level chunking produce poor retrieval<br>
results:<br>
• Cross-references: A requirement in one section will reference definitions in another section. A chunk<br>
containing only the requirement — without the referenced definition — produces incomplete context for<br>
the retrieval system.<br>
• Applicability conditions: Whether a regulation applies to a specific operator depends on conditions<br>
that may be defined in a completely separate section of the document. Standard chunking separates<br>
the requirement from its applicability criteria.<br>
• Version control: Regulatory documents are amended over time. Different operators may be subject<br>
to different document versions depending on their certificate conditions. The retrieval system needs to<br>
be version-aware.<br>
• Term definitions: Regulatory language uses precisely defined terms. 'Air taxi' means something<br>
specific under Canadian Aviation Regulations that is different from the common use of the phrase. The<br>
system needs to retrieve term definitions when they appear in retrieved content.<br>
The Chunking Strategy<br>
After significant experimentation, I settled on a four-tier chunking approach:<br>
Tier 1: Section-level chunks for definitions and applicability<br>
Complete regulatory sections that define terms or establish applicability criteria are kept as single chunks.<br>
These sections are typically 200–800 tokens. Splitting them would disconnect the definition from its scope<br>
conditions.<br>
Tier 2: Paragraph-level chunks for specific requirements<br>
Individual regulatory requirements — the 'you shall' and 'the operator must' provisions — are chunked at<br>
paragraph level. Each chunk is enriched with metadata capturing the section number, regulation name,<br>
version, and the top-level applicability category.<br>
Tier 3: Manually created summary chunks for complex multi-section requirements<br>
Some compliance requirements span multiple sections that must be read together. For the 20 most<br>
frequently queried complex requirements in the Navlyt corpus, I created manual summary chunks that<br>
synthesize the relevant provisions into a single context unit. These are expensive to maintain but critical for<br>
accurate retrieval on the most important queries.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcooxwca9i3aeevcv1bt9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcooxwca9i3aeevcv1bt9.png" alt=" " width="800" height="446"></a><br>
Tier 4: Cross-reference resolution chunks<br>
For each chunk containing a cross-reference to another section, I create a composite chunk that includes<br>
the original content and the referenced content. This increases chunk count but eliminates the most<br>
common retrieval failure mode — retrieving a requirement without its referenced definition.<br>
The Pinecone Index Architecture<br>
I use a single Pinecone index with namespace separation by regulation type. The namespace strategy<br>
allows targeted retrieval when the query context makes the applicable regulation clear — an operator with a<br>
Transport Canada certificate asking about pilot currency requirements doesn't need to search FAR Part 135<br>
content.<br>
TYPESCRIPT<br>
// Pinecone namespace structure const NAMESPACES = { transport_canada: 'tc_cars', // Canadian<br>
Aviation Regulations faa_part_135: 'faa_135', // FAR Part 135 - Air Taxi faa_part_91: 'faa_91', //<br>
FAR Part 91 - General Operating easa_cs23: 'easa_cs23', // EASA CS-23 Aircraft operator_specific:<br>
'ops_spec', // Company-specific Operations Specs }; // Query with operator context to target relevant<br>
namespaces async function retrieveCompliance( query: string, operatorContext: OperatorContext ):<br>
Promise { const targetNamespaces = resolveApplicableNamespaces(operatorContext); const queryEmbedding<br>
= await embedQuery(query); const results = await Promise.all( targetNamespaces.map(ns =&gt;<br>
pinecone.index('navlyt-regulations').namespace(ns).query({ vector: queryEmbedding, topK: 5,<br>
includeMetadata: true, filter: { is_current: { '$eq': true }, applicability_categories: { '$in':<br>
operatorContext.certificateCategories } } }) ) ); return mergeAndRerankResults(results, query); }<br>
The Answer Generation Pipeline<br>
The generation step is where compliance RAG diverges most from standard RAG. Every generated answer<br>
must:<br>
• Cite the specific regulatory provision it is based on (section number, regulation name, version)<br>
• Explicitly state when the answer applies only to certain operator categories<br>
• Flag when the query touches an area of regulatory ambiguity or where the answer may depend on<br>
factors not in the provided context<br>
• Never speculate — if the retrieved context does not contain a clear answer, the system must say so<br>
rather than generate a plausible-sounding response<br>
TYPESCRIPT<br>
// System prompt for compliance answer generation const COMPLIANCE_SYSTEM_PROMPT = ` You are a<br>
regulatory compliance assistant for charter aviation operators. Your answers must be based ONLY on<br>
the regulatory content provided in the context. RULES: 1. Every factual claim must cite the specific<br>
section: [REG_NAME §SECTION] 2. If context does not clearly answer the question, say: 'The provided<br>
regulations do not clearly address this. Consult a Transport Canada approved AMO or aviation lawyer.'</p>

<ol>
<li>State applicability conditions explicitly: 'This requirement applies to [operator category]
operating [aircraft type].' 4. NEVER speculate or infer requirements not stated in the provided
context. 5. Flag ambiguities: If two provisions appear to conflict, state this explicitly. 6. Always
end answers that involve certification with: 'Verify current requirements with Transport Canada
before taking action.' <code>; async function generateComplianceAnswer( query: string, retrievedChunks:
RegulatoryChunk[], operatorContext: OperatorContext ): Promise { const context =
formatChunksForContext(retrievedChunks); const response = await anthropic.messages.create({ model:
'claude-sonnet-4-20250514', max_tokens: 1000, system: COMPLIANCE_SYSTEM_PROMPT, messages: [{ role:
'user', content: </code> Operator context: ${JSON.stringify(operatorContext)} Regulatory context:
${context} Question: ${query} ` }] }); return parseComplianceResponse(response, retrievedChunks); }
Accuracy Validation — How I Measure If the System Is Actually Right
Standard RAG accuracy metrics (faithfulness, answer relevance) are insufficient for compliance-critical
applications. I built a validation framework specifically for regulatory accuracy:
Human expert validation set
I worked with a Transport Canada-licensed aviation consultant to build a validation set of 200 compliance
questions with verified correct answers. Every significant system change is evaluated against this set.
Current accuracy: 94.2% on the validation set.
Confidence scoring
Every answer includes a confidence score based on: the similarity scores of the retrieved chunks, whether
the top retrieved chunk directly addresses the query vs. only tangentially relates, and whether the query
touches areas flagged as high-ambiguity in the regulatory corpus.
Flagging for human review
Queries with confidence below 0.75, queries that trigger the 'regulations do not clearly address this'
response, and queries about recently amended sections automatically flag for human expert review. These
are presented to the operator with a clear disclaimer.
TYPESCRIPT
interface ComplianceAnswer { answer: string; citations: RegulatoryCitation[]; confidence: number; //
0-1 requiresHumanReview: boolean; applicabilityNote?: string; ambiguityWarning?: string;
lastRegUpdateCheck: string; // ISO date of most recent regulation version } // Citation structure
preserves full traceability interface RegulatoryCitation { regulation: string; // 'CARs' | 'FAR Part
135' | 'EASA CS-23' section: string; // '703.88(b)(2)' version: string; // 'CARs_2024_Q3' chunkId:
string; // Links back to source chunk relevanceScore: number; // Pinecone similarity score }
Key Lessons for Building Compliance RAG
n Lesson 1: Domain expert involvement is not optional
I could not have built the validation set, identified the critical cross-reference pairs, or calibrated the confidence
thresholds without working alongside a qualified aviation compliance expert. Budget for this. It is not a nice-to-have.
n Lesson 2: Chunk quality matters more than retrieval algorithm quality
I experimented with different embedding models and reranking approaches. The biggest accuracy improvements
came from improving chunk quality — specifically the cross-reference resolution chunks and the manual summary
chunks — not from changing the retrieval algorithm.
n Lesson 3: 'I don't know' is the right answer sometimes
The temptation in any RAG system is to always generate a plausible-sounding answer. In a compliance domain, a
wrong confident answer is worse than an honest 'the regulations don't clearly address this.' Build strong
non-answer logic.
n Lesson 4: Regulation changes require a maintenance process
Regulatory corpora are not static. Transport Canada amends the CARs regularly. Building Navlyt without a
regulatory update monitoring and corpus refresh process would mean the system gradually becomes wrong. This
maintenance process needs to be built before launch, not after.
Accuracy 94.2% On 200-question human-validated compliance test set
Avg latency 1.8s End-to-end from query to cited answer (p50)
Human review
flag rate 6.3% Queries automatically flagged for expert review due to low
confidence or ambiguity
n Navlyt is live at navlyt.com — full compliance OS for charter aviation operators.
n More on my AI architecture decisions: tilakraj.info/blog
About the Author
Tilak Raj is the CEO &amp; Founder of Brainfy AI, building vertical AI SaaS products across agriculture, insurance, aviation
compliance, real estate, and more from Edmonton, Alberta, Canada. He has shipped 8 live AI products and writes about AI
engineering, vertical SaaS architecture, and building from Canada. Follow on Dev.to: dev.to/tilakraj | Website: tilakraj.info</li>
</ol>

