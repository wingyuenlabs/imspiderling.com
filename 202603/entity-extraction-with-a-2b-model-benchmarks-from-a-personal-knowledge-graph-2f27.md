---
Title: 3º. Entity extraction with a 2B model: benchmarks from a personal knowledge graph
Description: 
Author: Víctor
Date: 2026-03-14T21:48:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>When you're building a personal knowledge graph — the kind that automatically discovers that "Ana García" appears in your emails, your calendar, and tomorrow's meeting notes — you need entity extraction. The industry answer is to throw GPT-4 at it and move on. But when your system runs on a mini-PC in someone's living room, you need something that fits in 2GB of RAM.</p>

<p>We benchmarked <code>qwen3-vl:2b-instruct-q4_K_M</code> — a 2-billion parameter multimodal model, quantized to 4-bit — running locally through Ollama. The same model that describes our photos also extracts entities from text. One model, two jobs, less RAM.</p>

<p>Here's what we found.</p>

<h2>
  
  
  The setup
</h2>

<p>We built a benchmark suite with two tasks:</p>

<p><strong>Text extraction</strong> — 15 cases across notes, emails, and diary entries. Mix of Spanish and English. Each case has human-annotated ground truth: which persons, projects, locations, and topics should be extracted.</p>

<p><strong>Vision extraction</strong> — 10 photos ranging from restaurant dinners to construction sites to landscape shots. Each photo goes through two stages: the model describes the image, then a second pass extracts entities from that description.</p>

<p>The extraction prompt is deliberately simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Extract named entities from the following text. Return ONLY a JSON object:
- persons: array of person names mentioned
- projects: array of project/product names mentioned
- locations: array of place names mentioned
- topics: array of key topics/themes (max 3)

Rules:
- Only extract what is EXPLICITLY mentioned
- Do not invent or infer entities not present
- Normalize names (capitalize properly)
</code></pre>

</div>



<p>Matching uses embedding similarity (qwen3-embedding, 1024d) with a 0.75 threshold instead of exact string matching. "Parte Vieja" matches "Parte Vieja" obviously, but "edge caching" also matches "edge caching approach" because the embeddings are close enough.</p>

<h2>
  
  
  Text extraction: the numbers
</h2>

<p>Overall F1: <strong>0.645</strong>. Zero parse errors across all 15 cases — the model always returned valid JSON. Average latency: 2-4 seconds per case on CPU.</p>

<p>But the overall F1 hides a story. Let's break it down by entity type:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Entity type</th>
<th>Avg F1</th>
<th>What happened</th>
</tr>
</thead>
<tbody>
<tr>
<td>Persons</td>
<td>~0.87</td>
<td>Near-perfect. The model's strongest category by far</td>
</tr>
<tr>
<td>Locations</td>
<td>~0.72</td>
<td>Handles Spanish geography beautifully</td>
</tr>
<tr>
<td>Projects</td>
<td>~0.65</td>
<td>Good when names are explicit, invents sometimes</td>
</tr>
<tr>
<td>Topics</td>
<td>~0.30</td>
<td>Weakest — but also the most subjective category</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Persons: the killer feature
</h3>

<p>The model nails names. Full names, first names, Spanish names with accents — it gets them right:</p>

<ul>
<li>"Marta Ibáñez", "Javier Losada", "Rubén" — all extracted from a construction note. ✓</li>
<li>"Carmen Pueyo", "Víctor García", "Diego Martínez" — from an email thread. ✓</li>
<li>"Tom Preston-Werner" — from an English conference note. ✓</li>
<li>"José Miguel Aguirre" — from a text full of nicknames. ✓</li>
<li>"Roberto Casas", "Víctor", "Lucía", "Sandra" — four people from a sprint review email. All four. ✓</li>
</ul>

<p>Where it stumbles: a diary entry mentioning "Papá" and "Mamá" — the model extracted them as persons. Technically correct (they are persons), but the human ground truth didn't include them because they're not named individuals. This is a recurring pattern: <strong>the model extracts more than the human annotated</strong>, which hurts precision without being wrong.</p>

<p>The other pattern: the model extracted "Javier" as a separate person from "Javier Losada". Both in the same note. That's an entity resolution problem, not an extraction problem — and the knowledge graph handles it downstream with merge candidates.</p>

<h3>
  
  
  Locations: surprisingly good at Spanish geography
</h3>

<p>"Valdespartera", "Villanueva de Gállego", "La Ternasca", "Parte Vieja", "Urgull", "Benasque", "Añisclo" — these aren't exactly world-famous cities. They're neighborhoods, hiking valleys, and small towns in Aragon and the Basque Country. The model got them all.</p>

<p>It also correctly classified "San Sebastián" as a location (not a person), "Ordesa" as a location (not a project), and "calle San Miguel" as a variant of "San Miguel." The embedding similarity matching helped here — "calle San Miguel" and "San Miguel" have near-perfect similarity.</p>

<p>One amusing misclassification: "eu-west-1" (an AWS region) was extracted as a location. I mean... it is a location. Just not the kind we meant.</p>

<h3>
  
  
  Projects: good when explicit, creative when not
</h3>

<p>When the text says "Micelclaw OS" or "MACP Protocol" or "OpenClaw Gateway", the model finds them with 100% accuracy. Named projects are easy.</p>

<p>The problem is when the model decides something is a project that isn't. "Pilotaje" (a construction technique) got classified as a project. "Txuletón" (a steak cut) became a project. "Barna" (slang for Barcelona) appeared as a project. The model is trying to be helpful — if it can't figure out which category something fits, it hedges by putting it in projects.</p>

<h3>
  
  
  Topics: where F1 lies
</h3>

<p>Topics scored ~0.30 F1. That sounds terrible. But look at what actually happened:</p>

<p>A diary entry about a trip to San Sebastián. Human ground truth: <code>["viaje", "desconexión"]</code> (trip, disconnecting). Model output: <code>["pintxos", "txuletón", "playa"]</code> (pintxos, steak, beach).</p>

<p>Both are correct summaries of the same diary entry. The human abstracted ("it was a trip about disconnecting"), the model got specific ("there were pintxos and beach"). The embedding similarity between "viaje" and "txuletón" is 0.67 — below the 0.75 threshold — so it counts as a miss.</p>

<p>This pattern repeats across almost every case. The human writes abstract topics; the model extracts concrete ones. For a knowledge graph, the model's approach is arguably better — "pintxos" is more searchable than "desconexión."</p>

<h3>
  
  
  Bilingual without trying
</h3>

<p>We mixed Spanish and English cases without telling the model which language to expect. It handled both without issues. "Tom Preston-Werner" from an English note and "José Miguel Aguirre" from a Spanish one were extracted with the same accuracy. The extraction prompt is in English; the input text is in whatever language the user writes in. The model doesn't care.</p>

<h3>
  
  
  The nickname challenge
</h3>

<p>The hardest test case was a Spanish note full of nicknames: "Pepe", "Tere", "Boli", "Txe", plus the full name "José Miguel Aguirre."</p>

<p>The model extracted "Pepe" and "José Miguel Aguirre" as separate persons — it didn't connect the nickname to the full name. It found "Tere" and "Txe" but missed "Boli." Three out of four nicknames is honestly better than expected for a 2B model.</p>

<p>Resolving "Pepe" = "José Miguel Aguirre" is entity resolution, not extraction. That's handled by the knowledge graph's merge candidate system — when two person nodes co-occur frequently, the system flags them for manual or automated merging.</p>

<h2>
  
  
  Vision extraction: description first, entities second
</h2>

<p>The photo pipeline works in two stages: the model describes the image, then the same extraction prompt runs on that description. This means the quality of entity extraction depends entirely on the quality of the description.</p>

<p>Overall vision F1: <strong>0.532</strong>. But the descriptions themselves are far better than the F1 suggests.</p>

<h3>
  
  
  The descriptions are impressive
</h3>

<p>A photo of an olive grove landscape:</p>

<blockquote>
<p><em>"This image captures a vast, sunlit landscape of rolling hills and valleys, likely in a rural or agricultural region. The scene is dominated by rows of olive trees planted in a dense, geometric pattern across the slopes."</em></p>
</blockquote>

<p>A construction site photo:</p>

<blockquote>
<p><em>"A red and silver laser level is set up on a tripod, indicating precise work is being done. The site is surrounded by dirt, sand, and a few trees."</em></p>
</blockquote>

<p>The model identified olive trees, a laser level on a tripod, and even recognized a 3D structural engineering model from a screenshot. For 2B parameters quantized to 4-bit, running on CPU, this is remarkable.</p>

<h3>
  
  
  Where vision extraction breaks down
</h3>

<p>The main issue: when photos contain people, the model says "four people sitting at a table" or "three people walking on a boardwalk." It counts them, describes what they're doing, but can't identify them. This is expected — face recognition requires a separate pipeline (we use InsightFace for that).</p>

<p>The problem for the benchmark is that "four people" gets extracted as a person entity, which counts as a false positive against a ground truth that says "no specific persons." This systematically tanks the persons F1 for vision.</p>

<h3>
  
  
  The ground truth problem
</h3>

<p>Here's what we learned about benchmarking entity extraction: <strong>the human is the bottleneck, not the model.</strong></p>

<p>For photo 7 (construction site), the human annotated objects as: <code>["tripod", "briefcase", "net", "brick", "concrete"]</code>. The model found: <code>["red brick wall", "large concrete block", "red and silver laser level", "tripod", "dirt", "sand", "trees", "house", "sunlight", "clear sky"]</code>.</p>

<p>The model extracted 10 objects where the human listed 5. The model's list is more complete and more accurate — "red and silver laser level" is a better description than what the human wrote. But the F1 score penalizes the model for being thorough, because every "extra" extraction hurts precision.</p>

<p>This is a fundamental issue with evaluating extraction against human annotations. The human annotates what they think is important. The model extracts what is present. For a knowledge graph that needs to be comprehensive, the model's approach is correct — you want to capture everything and let the search ranking decide what's relevant.</p>

<h2>
  
  
  Latency: the real constraint
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Task</th>
<th>Average</th>
<th>Range</th>
</tr>
</thead>
<tbody>
<tr>
<td>Text extraction</td>
<td>~3s</td>
<td>1.5–4.2s</td>
</tr>
<tr>
<td>Vision description</td>
<td>~2.4s</td>
<td>1.4–6.2s</td>
</tr>
<tr>
<td>Vision extraction</td>
<td>~1.8s</td>
<td>1.1–4.4s</td>
</tr>
</tbody>
</table></div>

<p>All on CPU, all sequential through a single Ollama instance with a priority semaphore. These numbers are for the async pipeline — the user never waits for them. A note gets created in ~50ms; the entity extraction happens 2-4 seconds later in the background.</p>

<p>The first request after a cold start took 66 seconds (model loading into RAM). After that, Ollama keeps the model loaded and subsequent requests are fast. This is why we keep a single model in memory — loading and unloading models per task would destroy latency.</p>

<h2>
  
  
  What we'd change
</h2>

<p><strong>Lower the similarity threshold for topics.</strong> The 0.75 threshold is too strict for abstract concepts. "Viaje" and "pintxos" are obviously related in context, but their embeddings are only 0.67 similar. For persons and locations, 0.75 is fine. For topics, 0.60 might be more appropriate.</p>

<p><strong>Post-process the "N people" pattern.</strong> When the vision model says "four women" or "three people," the extraction prompt shouldn't classify that as a person entity. A simple regex filter on the extraction output would fix the most common false positive.</p>

<p><strong>Embrace the verbosity.</strong> The model extracts more than a human would annotate. Instead of fighting this, design the knowledge graph to handle it — use confidence scores and the heat system to surface what matters and let the rest decay naturally.</p>

<h2>
  
  
  The takeaway
</h2>

<p>A 2B parameter model, quantized to 4-bit, running on CPU:</p>

<ul>
<li>
<strong>Persons:</strong> F1 0.87 — production-ready for a personal knowledge graph</li>
<li>
<strong>Locations:</strong> F1 0.72 — solid, handles non-English geography</li>
<li>
<strong>Projects:</strong> F1 0.65 — good enough with downstream deduplication</li>
<li>
<strong>Topics:</strong> F1 0.30 — misleading number, the model is actually more thorough than the human</li>
<li>
<strong>Parse reliability:</strong> 0 errors in 25 cases — always returns valid JSON</li>
<li>
<strong>Latency:</strong> 2-4 seconds async, invisible to the user</li>
</ul>

<p>Is it as good as GPT-4? No. Is it good enough to build a personal knowledge graph that automatically discovers connections between your notes, emails, and calendar? Yes. And it runs on your hardware, processes your data locally, and costs zero per extraction.</p>

<p>For a personal system processing maybe 50-100 new records per day, a 2B model with 3-second extraction time and ~0.87 F1 on the entities that matter most (people and places) is more than enough. The knowledge graph doesn't need to be perfect — it needs to be useful. And "Ana García appears in 3 emails and tomorrow's meeting" is useful even if the system also extracted "txuletón" as a project.</p>




<p><em>Next up: how we built a knowledge graph using just PostgreSQL — no Neo4j, no Apache AGE, just recursive CTEs and an entity_links table.</em></p>

