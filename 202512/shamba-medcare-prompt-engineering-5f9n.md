---
Title: Shamba-MedCare Prompt Engineering
Description: 
Author: Nicanor Korir
Date: 2025-12-01T21:30:44.000Z
Robots: noindex,nofollow
Template: index
---
<p>Some background context:</p>

<blockquote>
<p>I am building a simple plant disease diagnosis solution using AI, inspired by my farming background and advancements in intelligent technological tools</p>
</blockquote>

<p>You can check out the <a href="http://shamba-medcare.vercel.app/" rel="noopener noreferrer">Shamba-MedCare App here</a>. Sorry for testing, you'll have to use your own api keys until the public launch is available. The keys are stored in  the browser's local storage, so they are private</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdn4gszzoyu0qn7jngevv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdn4gszzoyu0qn7jngevv.png" alt=" " width="800" height="1104"></a></p>

<blockquote>
<p>For context here, whenever you read LLM(Large Language Model), I mostly Claude. I like to use LLM since it's generic, and this solution can be fitted to any LLM.</p>
</blockquote>

<p>I played around with several prompts in order to nail the best results. This is how I transformed my prompt engineering journey with Shamba-MedCare:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0jao6jelvpcoku09ueqr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0jao6jelvpcoku09ueqr.png" alt=" " width="800" height="624"></a></p>

<p>My first prompt to LLM Vision was embarrassingly naive:</p>

<blockquote>
<p>"What disease does this plant have?"</p>
</blockquote>

<p>The response was a 2,000-word essay about plant pathology in general. Helpful for a textbook. Useless for a farmer with a dying tomato plant. Getting AI to return <strong>structured, actionable, budget-aware diagnoses</strong> took iteration. Here's what I learned.</p>

<h2>
  
  
  The Architecture
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4snwsw6mlb9e49xjw60g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4snwsw6mlb9e49xjw60g.png" alt=" " width="800" height="578"></a></p>

<p>Two prompts matter: the <strong>system prompt</strong> (who LLM(e.g. claude pretends to be) and the <strong>analysis prompt</strong> (what to do with this specific image).</p>

<h2>
  
  
  System Prompt: Creating "Shamba"
</h2>

<p>Prompts work better with a persona. I created <strong>Shamba persona</strong>, an agricultural pathologist who:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>You are Shamba, an expert agricultural pathologist. You analyze
plant images to identify diseases, pests, and nutrient deficiencies.

Your expertise includes:
- 50+ crop types worldwide
- Fungal, bacterial, viral, and physiological disorders
- Traditional and modern treatment methods
- Practical advice for resource-limited farmers

Guidelines:
1. Always include at least one FREE/traditional treatment
2. Describe WHERE symptoms appear (for visual mapping)
3. Be honest about uncertainty—use confidence scores
4. Recommend professional help for severe cases
</code></pre>

</div>



<p>The key line: <strong>"Always include at least one FREE/traditional treatment."</strong></p>

<p>Without that explicit instruction, the LLM defaulted to commercial products. Helpful for a suburban gardener. Useless for a farmer who can't afford a $15 fungicide.</p>

<h2>
  
  
  Failure #1: The JSON Nightmare
</h2>

<p>My first structured attempt asked LLM to return JSON, which it did pretty well. Wrapped in markdown code fences. With helpful commentary before and after.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Here's my analysis:

</code></pre>

</div>

<p><br>
json<br>
{ "disease": "Early Blight" }<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
This is a common fungal disease...
</code></pre>

</div>



<p>My parser choked. The fix was explicit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Return ONLY a valid JSON object. No markdown, no commentary,
no text before or after. Start with { and end with }
</code></pre>

</div>



<p>Still failed 10% of the time. So I added backend parsing that:</p>

<ol>
<li>Strips markdown fences if present</li>
<li>Extracts JSON from surrounding text</li>
<li>Validates against the expected schema</li>
</ol>

<h2>
  
  
  Failure #2: Location Descriptions
</h2>

<p>For the visual heatmap feature, I needed LLM to describe WHERE damage appeared. My prompt asked for "affected regions."</p>

<p>LLM returned: "The affected area is significant." This was not helpful, I needed the exact coordinates, and I tried out several solutions. This was close to perfect:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Describe affected regions with:
- Location(helpful for heatmaps): top-left, center, lower-right, edges, margins
- Coverage: percentage of area affected (e.g., "35%")
- Spread direction: "Moving from lower leaves upward."
</code></pre>

</div>



<p>Now LLM returns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"affected_regions"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"location"</span><span class="p">:</span><span class="w"> </span><span class="s2">"lower-left"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"severity"</span><span class="p">:</span><span class="w"> </span><span class="s2">"severe"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Dark brown lesions with concentric rings"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"coverage"</span><span class="p">:</span><span class="w"> </span><span class="mi">15</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"location"</span><span class="p">:</span><span class="w"> </span><span class="s2">"center"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"severity"</span><span class="p">:</span><span class="w"> </span><span class="s2">"moderate"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"coverage"</span><span class="p">:</span><span class="w"> </span><span class="mi">20</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>That's enough to generate a heatmap overlay for now</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foqc89g3pyhnmm7bs75cj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foqc89g3pyhnmm7bs75cj.png" alt=" " width="800" height="578"></a></p>

<h2>
  
  
  Failure #3: Treatment Cost Blindness
</h2>

<p>Early on, treatments came out randomly ordered. Sometimes the $50 systemic fungicide appeared first. Sometimes, the free wood ash remedy.</p>

<p>The problem: LLM has no inherent understanding of budget constraints. I had to structure it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Provide treatments in EXACTLY this order:
1. FREE TIER: Traditional/home remedies ($0)
2. LOW COST: Basic solutions ($1-5)
3. MEDIUM COST: Commercial organic ($5-20)
4. HIGH COST: Synthetic/professional ($20+)

Each tier must have at least one option if applicable.
</code></pre>

</div>



<p>The response schema enforced this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"treatments"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"method"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Wood ash paste"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"cost_tier"</span><span class="p">:</span><span class="w"> </span><span class="s2">"free"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"estimated_cost"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$0"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"ingredients"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"Wood ash"</span><span class="p">,</span><span class="w"> </span><span class="s2">"Water"</span><span class="p">],</span><span class="w">
      </span><span class="nl">"application"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Apply directly to affected areas"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"availability"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Common from cooking fires"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"method"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Neem oil spray"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"cost_tier"</span><span class="p">:</span><span class="w"> </span><span class="s2">"low"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"estimated_cost"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$1-3"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  The Plant Part-Specific Prompt Strategy
</h2>

<p>Different plant parts reveal different problems, and I needed the right prompt to get the right problem with the best remedies. My prompt adapts:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fys5hvnecnd073rv1j4t4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fys5hvnecnd073rv1j4t4.png" alt=" " width="800" height="578"></a></p>

<p>For leaves:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Examine: color patterns, spot shapes, curling, holes, coating
Common issues: fungal spots, viral mosaic, nutrient chlorosis, pest damage
</code></pre>

</div>



<p>For roots:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Examine: color (white=healthy, brown/black=rot), texture, galls, structure
Common issues: root rot, nematode damage, waterlogging
</code></pre>

</div>



<p>This focus improves accuracy dramatically. Asking LLM to look for "anything wrong" produces vague results. Asking it to specifically check for concentric ring patterns in leaf spots? Now we're diagnosing Early Blight.</p>

<h2>
  
  
  The Final Prompt Structure
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[SYSTEM PROMPT]
You are Shamba, an agricultural pathologist...

[ANALYSIS PROMPT]
Analyze this {plant_part} image from a {crop_type} plant.

User's context: {additional_context}

Provide:
1. Image validation (correct plant part? good quality?)
2. Health score (0-100)
3. Disease identification with confidence (0.0-1.0)
4. Affected region locations for visual mapping
5. Treatments by cost tier (FREE mandatory)
6. Prevention tips

Return as JSON following this schema:
{response_schema}
</code></pre>

</div>



<h2>
  
  
  What I'd Do Differently
</h2>

<p><strong>Start with the output format first.</strong> I designed prompts around what I wanted LLM to do. I should have designed around what the farmer needed to see.</p>

<p>The heatmap feature was an afterthought. If I'd planned for it from day one, the location description format would have been baked in, not retrofitted. This is actually a useful feature for farmers if you can imagine an affected plant, there are heatmaps on the heavily affected areas</p>

<p><strong>Test with bad photos early.</strong> My development photos were well-lit, centered, single-issue plants. Real farmer photos are blurry, shadowy, and show three problems at once. The robustness I needed only emerged after testing with garbage inputs.</p>




<p><a href="https://github.com/nicanor/shamba-dawa" rel="noopener noreferrer">Source code on GitHub</a></p>

