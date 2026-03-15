---
Title: I Built a Voice-First AI Photo & Document Editor with the Gemini Live API— Here's How
Description: 
Author: Bizzi Cole87
Date: 2026-03-15T22:04:07.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This article was created for the purposes of entering the Gemini Live Agent Challenge hackathon. #GeminiLiveAgentChallenge</em></p>




<p>There's a version of photo editing where you don't touch a single slider. You click the part of the image you want to change, say what you want out loud, and watch it happen. That's Say Edit.</p>

<p>Over the past few weeks I built Say Edit — a voice-first AI workspace that lets you edit images and navigate documents entirely by speaking. It's powered by the <strong>Gemini Live API</strong>, <strong>Gemini image generation</strong>, and deployed on <strong>Google Cloud Run</strong>. This article is the behind-the-scenes of how I built it, what broke, and what surprised me.</p>




<h2>
  
  
  The Core Idea
</h2>

<p>Most AI tools make you type. You open a chat window, describe what you want, wait for a response, copy it somewhere, repeat. I wanted to eliminate every one of those steps for two use cases I found genuinely painful:</p>

<ol>
<li>
<strong>Editing a photo</strong> — you know exactly what you want to change, but you have to hunt through menus, masks, and sliders to get there.</li>
<li>
<strong>Reading a dense document</strong> — you have a question, but finding the exact passage means scrolling through 80 pages.</li>
</ol>

<p>The answer to both is the same: a persistent voice session that listens continuously, understands your intent, and acts immediately.</p>




<h2>
  
  
  The Voice Loop — Gemini Live API
</h2>

<p>The foundation of Say Edit is a bi-directional WebSocket with the <strong>Gemini Live API</strong> (<code>gemini-2.5-flash-native-audio-preview-12-2025</code>). This is not a push-to-talk button. The model listens the entire time you're in the workspace.</p>

<p>Here's what the audio pipeline looks like on the browser side:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">source</span> <span class="o">=</span> <span class="nx">audioCtx</span><span class="p">.</span><span class="nf">createMediaStreamSource</span><span class="p">(</span><span class="nx">stream</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">processor</span> <span class="o">=</span> <span class="nx">audioCtx</span><span class="p">.</span><span class="nf">createScriptProcessor</span><span class="p">(</span><span class="mi">4096</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
<span class="nx">source</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">processor</span><span class="p">);</span>
<span class="nx">processor</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">audioCtx</span><span class="p">.</span><span class="nx">destination</span><span class="p">);</span>

<span class="nx">processor</span><span class="p">.</span><span class="nx">onaudioprocess</span> <span class="o">=</span> <span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">float32</span> <span class="o">=</span> <span class="nx">e</span><span class="p">.</span><span class="nx">inputBuffer</span><span class="p">.</span><span class="nf">getChannelData</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
  <span class="c1">// Float32 → Int16 → base64</span>
  <span class="kd">const</span> <span class="nx">int16</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Int16Array</span><span class="p">(</span><span class="nx">float32</span><span class="p">.</span><span class="nx">length</span><span class="p">);</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">float32</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">int16</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="o">-</span><span class="mi">32768</span><span class="p">,</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(</span><span class="mi">32767</span><span class="p">,</span> <span class="nx">float32</span><span class="p">[</span><span class="nx">i</span><span class="p">]</span> <span class="o">*</span> <span class="mi">32768</span><span class="p">));</span>
  <span class="p">}</span>
  <span class="nx">session</span><span class="p">.</span><span class="nf">sendRealtimeInput</span><span class="p">({</span>
    <span class="na">media</span><span class="p">:</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nf">btoa</span><span class="p">(</span><span class="nb">String</span><span class="p">.</span><span class="nf">fromCharCode</span><span class="p">(...</span><span class="k">new</span> <span class="nc">Uint8Array</span><span class="p">(</span><span class="nx">int16</span><span class="p">.</span><span class="nx">buffer</span><span class="p">))),</span> <span class="na">mimeType</span><span class="p">:</span> <span class="dl">'</span><span class="s1">audio/pcm;rate=16000</span><span class="dl">'</span> <span class="p">}</span>
  <span class="p">});</span>
<span class="p">};</span>
</code></pre>

</div>



<p>16kHz PCM, streamed continuously. The model responds with audio at 24kHz, which I queue through <code>AudioBufferSourceNode</code> for gapless playback.</p>

<p>The part that changed how I think about this kind of app: <strong>interruption</strong>. When you speak while the model is responding, the Live API sends <code>serverContent.interrupted</code>. I clear the audio queue instantly and the model re-listens. No waiting. That single behavior makes the whole thing feel like talking to a person rather than waiting for a request.</p>




<h2>
  
  
  Image Editing — Hotspot + Voice + Gemini Image Generation
</h2>

<p>The image editor works in three steps:</p>

<ol>
<li>
<strong>Click</strong> anywhere on the image. The frontend records the pixel coordinates <code>(x, y)</code> as a "hotspot" and renders a crosshair.</li>
<li>
<strong>Speak</strong> your edit. The Gemini Live session calls the <code>edit_image_region</code> tool with your spoken instruction and the hotspot coordinates.</li>
<li>
<strong>Watch</strong> the edit apply. The current image is sent as base64 to <code>gemini-2.0-flash-exp-image-generation</code> alongside the prompt and coordinates. The result replaces the current frame in the history stack.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Tool definition registered with the Live session</span>
<span class="p">{</span>
  <span class="nl">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">edit_image_region</span><span class="dl">'</span><span class="p">,</span>
  <span class="nx">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Edit a specific region of the current image.</span><span class="dl">'</span><span class="p">,</span>
  <span class="nx">parameters</span><span class="p">:</span> <span class="p">{</span>
    <span class="nl">type</span><span class="p">:</span> <span class="nx">Type</span><span class="p">.</span><span class="nx">OBJECT</span><span class="p">,</span>
    <span class="nx">properties</span><span class="p">:</span> <span class="p">{</span>
      <span class="nl">edit_prompt</span><span class="p">:</span> <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="nx">Type</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="nx">hotspot_x</span><span class="p">:</span> <span class="p">{</span> <span class="nl">type</span><span class="p">:</span> <span class="nx">Type</span><span class="p">.</span><span class="nx">NUMBER</span> <span class="p">},</span>
      <span class="nx">hotspot_y</span><span class="p">:</span> <span class="p">{</span> <span class="nl">type</span><span class="p">:</span> <span class="nx">Type</span><span class="p">.</span><span class="nx">NUMBER</span> <span class="p">},</span>
    <span class="p">},</span>
    <span class="nx">required</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">edit_prompt</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">hotspot_x</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">hotspot_y</span><span class="dl">'</span><span class="p">],</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The edit call to Gemini image generation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">ai</span><span class="p">.</span><span class="nx">models</span><span class="p">.</span><span class="nf">generateContent</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gemini-3.1-flash-image-preview</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">contents</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">inlineData</span><span class="p">:</span> <span class="p">{</span> <span class="na">mimeType</span><span class="p">:</span> <span class="nx">file</span><span class="p">.</span><span class="kd">type</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="nx">base64</span> <span class="p">}</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">text</span><span class="p">:</span> <span class="s2">`Edit: "</span><span class="p">${</span><span class="nx">editPrompt</span><span class="p">}</span><span class="s2">". Focus around pixel (x: </span><span class="p">${</span><span class="nx">x</span><span class="p">}</span><span class="s2">, y: </span><span class="p">${</span><span class="nx">y</span><span class="p">}</span><span class="s2">). Return ONLY the edited image.`</span> <span class="p">}</span>
  <span class="p">],</span>
  <span class="na">config</span><span class="p">:</span> <span class="p">{</span> <span class="na">responseModalities</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">IMAGE</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Every edit is non-destructive — pushed onto a history stack with full undo/redo and a hold-to-compare button that fades back to the original.</p>

<h3>
  
  
  The Stale Closure Problem
</h3>

<p>This one burned me for a while. The Live API's <code>onmessage</code> callback is registered once at session start. Any React state it closes over is immediately stale — meaning <code>history[historyIndex]</code> would always point to the original image, no matter how many edits had been applied.</p>

<p>The fix was to maintain refs that mirror the state and always read from those inside the callback:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">historyRef</span> <span class="o">=</span> <span class="nx">useRef</span><span class="o">&lt;</span><span class="nx">File</span><span class="p">[]</span><span class="o">&gt;</span><span class="p">([</span><span class="nx">initialFile</span><span class="p">]);</span>
<span class="kd">const</span> <span class="nx">historyIndexRef</span> <span class="o">=</span> <span class="nx">useRef</span><span class="o">&lt;</span><span class="kr">number</span><span class="o">&gt;</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
<span class="nx">historyRef</span><span class="p">.</span><span class="nx">current</span> <span class="o">=</span> <span class="nx">history</span><span class="p">;</span>       <span class="c1">// kept in sync on every render</span>
<span class="nx">historyIndexRef</span><span class="p">.</span><span class="nx">current</span> <span class="o">=</span> <span class="nx">historyIndex</span><span class="p">;</span>

<span class="c1">// Inside the async tool handler — always gets the live value</span>
<span class="kd">const</span> <span class="nx">imageFile</span> <span class="o">=</span> <span class="nx">historyRef</span><span class="p">.</span><span class="nx">current</span><span class="p">[</span><span class="nx">historyIndexRef</span><span class="p">.</span><span class="nx">current</span><span class="p">];</span>
</code></pre>

</div>



<p>This is now a fixture of how I build anything on top of the Live API.</p>




<h2>
  
  
  Document Navigation — Spatial Search + Live Highlights
</h2>

<p>The document workspace is a different beast. When you ask a question, the AI shouldn't just answer — it should show you exactly where in the document the answer lives.</p>

<p>The pipeline:</p>

<p><strong>1. Ingestion (NestJS backend on Google Cloud Run)</strong></p>

<p>When you upload a PDF, the backend runs it through <code>pdfjs-dist</code>, extracts text items with their transform matrices, groups words into lines by Y-coordinate proximity, and merges lines into sentence-level chunks. Every chunk stores a tight bounding box <code>[x, y, width, height]</code> alongside the text.</p>

<p>One gotcha: PDF coordinates use bottom-left origin, but the React PDF viewer uses top-left. Every Y coordinate gets flipped during ingestion:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nx">y</span> <span class="o">=</span> <span class="nx">pageHeight</span> <span class="o">-</span> <span class="nx">transform</span><span class="p">[</span><span class="mi">5</span><span class="p">]</span> <span class="o">-</span> <span class="p">(</span><span class="nx">height</span> <span class="o">||</span> <span class="mi">10</span><span class="p">)</span>
</code></pre>

</div>



<p>Each chunk is then embedded with <code>gemini-embedding-001</code> and stored in a <strong>Supabase pgvector</strong> index with an HNSW index for fast cosine similarity search.</p>

<p><strong>2. Search during the Live session</strong></p>

<p>When you ask a question, the Live session calls <code>search_document</code>. The frontend hits the backend's <code>/query</code> endpoint, which embeds your query and runs a cosine similarity search against the document's chunks. The top results come back with page numbers and bounding boxes.</p>

<p><strong>3. Spatial highlight</strong></p>

<p>The model then calls <code>focus_document_section</code> with the page and an array of bounding box coordinates. The frontend jumps the PDF viewer to the right page and renders yellow highlight overlays at the exact pixel positions of the relevant sentences:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Convert stored PDF-point bboxes to percentage overlays</span>
<span class="kd">const</span> <span class="nx">highlight</span> <span class="o">=</span> <span class="p">{</span>
  <span class="nx">pageIndex</span><span class="p">,</span>
  <span class="na">left</span><span class="p">:</span>   <span class="p">(</span><span class="nx">bx</span> <span class="o">/</span> <span class="nx">pageWidthPx</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">,</span>
  <span class="na">top</span><span class="p">:</span>    <span class="p">(</span><span class="nx">by</span> <span class="o">/</span> <span class="nx">pageHeightPx</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">,</span>
  <span class="na">width</span><span class="p">:</span>  <span class="p">(</span><span class="nx">bw</span> <span class="o">/</span> <span class="nx">pageWidthPx</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">,</span>
  <span class="na">height</span><span class="p">:</span> <span class="p">(</span><span class="nx">bh</span> <span class="o">/</span> <span class="nx">pageHeightPx</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="p">,</span>
<span class="p">};</span>
</code></pre>

</div>



<p>You hear the answer and see it highlighted on the page at the same moment.</p>




<h2>
  
  
  The Compose Studio
</h2>

<p>Beyond single-image editing, Say Edit has a Compose Studio that merges two photos by voice:</p>

<blockquote>
<p><em>"Dress the person from A in the outfit from B."</em><br>
<em>"Put the product on the background from B."</em></p>
</blockquote>

<p>Both images are encoded as base64 and sent to Gemini image generation together:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">ai</span><span class="p">.</span><span class="nx">models</span><span class="p">.</span><span class="nf">generateContent</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gemini-3.1-flash-image-preview</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">contents</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">inlineData</span><span class="p">:</span> <span class="p">{</span> <span class="na">mimeType</span><span class="p">:</span> <span class="nx">imgA</span><span class="p">.</span><span class="kd">type</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="nx">base64A</span> <span class="p">}</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">inlineData</span><span class="p">:</span> <span class="p">{</span> <span class="na">mimeType</span><span class="p">:</span> <span class="nx">imgB</span><span class="p">.</span><span class="kd">type</span><span class="p">,</span> <span class="na">data</span><span class="p">:</span> <span class="nx">base64B</span> <span class="p">}</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">text</span><span class="p">:</span> <span class="s2">`Composition request: "</span><span class="p">${</span><span class="nx">prompt</span><span class="p">}</span><span class="s2">". Return ONLY the composed image.`</span> <span class="p">}</span>
  <span class="p">],</span>
  <span class="na">config</span><span class="p">:</span> <span class="p">{</span> <span class="na">responseModalities</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">IMAGE</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">TEXT</span><span class="dl">'</span><span class="p">]</span> <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>The result lands in the same history stack as regular edits — so you can compose two images together and then keep refining the result by voice.</p>




<h2>
  
  
  Infrastructure — Google Cloud Run
</h2>

<p>Both the frontend (React + Vite) and the backend (NestJS) are containerized and deployed to <strong>Google Cloud Run</strong>. The Dockerfile for the frontend bakes the Vite environment variables at build time using <code>ARG</code> injection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">ARG</span><span class="s"> VITE_GEMINI_API_KEY</span>
<span class="k">ARG</span><span class="s"> VITE_BACKEND_URL</span>
<span class="k">RUN </span><span class="nb">echo</span> <span class="s2">"VITE_GEMINI_API_KEY=</span><span class="nv">$VITE_GEMINI_API_KEY</span><span class="s2">"</span> <span class="o">&gt;</span> .env
<span class="k">RUN </span><span class="nb">echo</span> <span class="s2">"VITE_BACKEND_URL=</span><span class="nv">$VITE_BACKEND_URL</span><span class="s2">"</span> <span class="o">&gt;&gt;</span> .env
<span class="k">RUN </span>npx vite build
</code></pre>

</div>



<p>Deploy with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gcloud run deploy say-edit <span class="se">\</span>
  <span class="nt">--source</span> <span class="nb">.</span> <span class="se">\</span>
  <span class="nt">--region</span> us-central1 <span class="se">\</span>
  <span class="nt">--allow-unauthenticated</span> <span class="se">\</span>
  <span class="nt">--set-build-env-vars</span> <span class="s2">"VITE_GEMINI_API_KEY=...,VITE_BACKEND_URL=..."</span>
</code></pre>

</div>






<h2>
  
  
  What I Learned
</h2>

<p><strong>The Live API's interruption model is the whole game.</strong> Every other voice interface I've used makes you wait. The ability to interrupt mid-sentence — backed by a proper server-side signal rather than a client-side hack — is what makes Say Edit feel like a real conversation.</p>

<p><strong>Tool design is UX design.</strong> The <code>get_current_hotspot</code> tool exists purely so users can say "edit this" without re-stating coordinates. Getting the tool schema right — what the model calls, when, and with what arguments — determines the quality of the interaction more than any UI element.</p>

<p><strong>Refs are mandatory in Live API callbacks.</strong> Any async callback registered at session start captures stale React state. The ref-mirror pattern is non-negotiable.</p>

<p><strong>PDF extraction is harder than it looks.</strong> <code>pdfjs-dist</code> gives you transform matrices, not sentences. The grouping and chunking pipeline is the most underestimated part of the whole system.</p>




<h2>
  
  
  Try It
</h2>

<p><strong>Live demo:</strong> <a href="https://glyph-client-229364276486.us-central1.run.app/" rel="noopener noreferrer">https://glyph-client-229364276486.us-central1.run.app/</a><br>
<strong>Client repo:</strong> <a href="https://github.com/greatsage-raphael/say_edit" rel="noopener noreferrer">https://github.com/greatsage-raphael/say_edit</a><br>
<strong>Server repo:</strong> <a href="https://github.com/greatsage-raphael/say_edit_server" rel="noopener noreferrer">https://github.com/greatsage-raphael/say_edit_server</a></p>

<p>Built with the Gemini Live API, NaNo Banana, Google Cloud Run, and Supabase.</p>

<p><em>This article was created for the purposes of entering the Gemini Live Agent Challenge hackathon. #GeminiLiveAgentChallenge</em></p>

