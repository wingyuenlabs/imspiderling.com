---
Title: How I Built GM-Genie: A Cinematic AI Game Master with Gemini Live API
Description: 
Author: Vasilis Stefanopoulos
Date: 2026-03-16T21:47:01.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This project was created for the Gemini Live Agent Challenge hackathon</em></p>




<h2>
  
  
  The Problem: Text-Only RPGs Break Immersion
</h2>

<p>Every tabletop RPG player knows the feeling — your game master describes a breathtaking ruin half-swallowed by ash dunes, but all you see is... a chat window. Text generates the world in your mind, but there's a ceiling on how cinematic that experience can get.</p>

<p>What if your AI game master could <em>show</em> you the world as it narrated, play a sound effect when the door creaks open, shift the ambient drone when combat starts — all without breaking the flow of conversation?</p>

<p>That's what I built with <strong>GM-Genie</strong>: a voice-first RPG narrator that weaves AI-generated scene art, dynamic sound effects, and real-time audio narration into a single seamless experience. You talk to your GM, your GM talks back — and the world materializes around you.</p>




<h2>
  
  
  Architecture: From Multi-Agent to Zero-Tool
</h2>

<p>Here's the honest story: the architecture you see in the final version is not the one I originally designed. I planned a multi-agent pipeline with function calling for everything — dice, inventory, scene generation, SFX. That version had a 70% connection crash rate in voice mode.</p>

<p>The current architecture threw nearly all of that away.</p>

<h3>
  
  
  What's Running Now
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Voice Mode (PRIMARY):
  gemini-2.5-flash-native-audio-latest  ←  WebSocket bidi audio
  ├── ZERO tools in voice pipeline
  ├── DicePool injects pre-rolled results into system prompt
  ├── Server transcribes GM speech → SceneDetector analyzes it
  └── SceneDetector triggers: scene image / SFX / ambient (server-side)

Text Mode (FALLBACK):
  gemini-2.5-flash + Google ADK
  └── Tools: roll_dice, manage_inventory, check_stats, generate_scene, trigger_sfx
</code></pre>

</div>



<p>The voice session is a pure audio conversation. No function calling, no tool dispatch, no round-trips to decide what to do. The model speaks, the server listens to the transcript, and intelligence fires server-side.</p>

<h3>
  
  
  Backend
</h3>

<p>FastAPI with two endpoints:</p>

<ul>
<li>
<code>POST /api/chat</code> — SSE streaming, text mode</li>
<li>
<code>WS /api/live</code> — WebSocket, bidirectional audio, voice mode</li>
</ul>

<h3>
  
  
  Frontend
</h3>

<p>React 19 + Vite + Tailwind CSS v4. WorldSelect → Lobby → Game. Audio runs through Web Audio API with separate AudioWorklet processors for capture (16kHz) and playback (24kHz).</p>




<h2>
  
  
  The Interesting Parts
</h2>

<h3>
  
  
  1. Zero-Tool Voice Architecture
</h3>

<p>The native-audio Gemini model (<code>gemini-2.5-flash-native-audio-latest</code>) crashed approximately 70% of the time when function calling was active. The WebSocket would close mid-session with codes 1000, 1008, or 1011 — silently, without a clear error.</p>

<p>The fix: remove every tool from the voice pipeline. No function calling at all.</p>

<p>But then how does anything happen? The GM still needs dice results. Scenes still need to generate. Sound still needs to fire.</p>

<p><strong>Dice</strong>: Pre-roll a full session's worth of results before the session starts and inject them into the system prompt. The model reads them and narrates accordingly.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">DicePool</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Pre-rolled dice injected into system prompt — no tool calls needed.</span><span class="sh">"""</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">seed</span><span class="p">:</span> <span class="nb">int</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span><span class="p">):</span>
        <span class="n">rng</span> <span class="o">=</span> <span class="n">random</span><span class="p">.</span><span class="nc">Random</span><span class="p">(</span><span class="n">seed</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">pool</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">d4</span><span class="sh">"</span><span class="p">:</span>  <span class="p">[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">4</span><span class="p">)</span>  <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">30</span><span class="p">)],</span>
            <span class="sh">"</span><span class="s">d6</span><span class="sh">"</span><span class="p">:</span>  <span class="p">[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">6</span><span class="p">)</span>  <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">40</span><span class="p">)],</span>
            <span class="sh">"</span><span class="s">d8</span><span class="sh">"</span><span class="p">:</span>  <span class="p">[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">8</span><span class="p">)</span>  <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">30</span><span class="p">)],</span>
            <span class="sh">"</span><span class="s">d10</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">10</span><span class="p">)</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">30</span><span class="p">)],</span>
            <span class="sh">"</span><span class="s">d12</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">12</span><span class="p">)</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">20</span><span class="p">)],</span>
            <span class="sh">"</span><span class="s">d20</span><span class="sh">"</span><span class="p">:</span> <span class="p">[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">20</span><span class="p">)</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">40</span><span class="p">)],</span>
            <span class="sh">"</span><span class="s">d100</span><span class="sh">"</span><span class="p">:[</span><span class="n">rng</span><span class="p">.</span><span class="nf">randint</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">100</span><span class="p">)</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">10</span><span class="p">)],</span>
        <span class="p">}</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_idx</span><span class="p">:</span> <span class="nb">dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">int</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">k</span><span class="p">:</span> <span class="mi">0</span> <span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">pool</span><span class="p">}</span>

    <span class="k">def</span> <span class="nf">draw</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">dice_type</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">int</span><span class="p">:</span>
        <span class="n">pool</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">pool</span><span class="p">[</span><span class="n">dice_type</span><span class="p">]</span>
        <span class="n">idx</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">_idx</span><span class="p">[</span><span class="n">dice_type</span><span class="p">]</span> <span class="o">%</span> <span class="nf">len</span><span class="p">(</span><span class="n">pool</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">_idx</span><span class="p">[</span><span class="n">dice_type</span><span class="p">]</span> <span class="o">+=</span> <span class="mi">1</span>
        <span class="k">return</span> <span class="n">pool</span><span class="p">[</span><span class="n">idx</span><span class="p">]</span>

    <span class="k">def</span> <span class="nf">prompt_block</span><span class="p">(</span><span class="n">self</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">Returns the pre-rolled pool formatted for system prompt injection.</span><span class="sh">"""</span>
        <span class="n">lines</span> <span class="o">=</span> <span class="p">[</span><span class="sh">"</span><span class="s">[PRE-ROLLED DICE POOL — use in order, top to bottom]</span><span class="sh">"</span><span class="p">]</span>
        <span class="k">for</span> <span class="n">k</span><span class="p">,</span> <span class="n">vals</span> <span class="ow">in</span> <span class="n">self</span><span class="p">.</span><span class="n">pool</span><span class="p">.</span><span class="nf">items</span><span class="p">():</span>
            <span class="n">lines</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">k</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="nf">str</span><span class="p">(</span><span class="n">v</span><span class="p">)</span> <span class="k">for</span> <span class="n">v</span> <span class="ow">in</span> <span class="n">vals</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">lines</span><span class="p">)</span>
</code></pre>

</div>



<p>The system prompt tells the GM to consume results in order from top to bottom. It's deterministic, fast, and requires zero API calls during the session.</p>

<p><strong>Scenes and sound</strong>: handled by <code>SceneDetector</code>.</p>

<h3>
  
  
  2. SceneDetector: Server-Side Intelligence
</h3>

<p>The <code>SceneDetector</code> monitors every GM transcript line and fires the right media event based on what the GM just said — no tool calls, no model decision-making.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Primary scene trigger: GM says "You see..." or "Before you stands..."
</span><span class="n">_VISUAL_CUE_PATTERNS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\byou see\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\bbefore (?:you|your eyes)\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\byou notice\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\btowering (?:above|over|before)\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\bsprawl(?:s|ing)? (?:before|below|around)\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\bloom(?:s|ing)? (?:over|above|ahead|before)\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># ... 10 more patterns
</span><span class="p">]</span>

<span class="c1"># Secondary: location transitions
</span><span class="n">_LOCATION_PATTERNS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\byou (?:enter|arrive|step into|walk into)\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="sa">r</span><span class="sh">"</span><span class="s">\bwelcome to\b</span><span class="sh">"</span><span class="p">,</span>
    <span class="c1"># ...
</span><span class="p">]</span>
</code></pre>

</div>



<p>When a pattern fires, the server calls <code>gemini-3-pro-image-preview</code> to generate a scene image and sends it to the frontend via the WebSocket. SFX and ambient changes work the same way — keyword hits on the transcript trigger a Freesound API search, result gets cached to disk, URL goes to the frontend.</p>

<p>The GM's prompt is written so it reliably uses the trigger phrases. The phrases aren't random — they're the natural language a good narrator uses when describing what a player sees. The model's instincts and the detector's patterns are aligned by design.</p>

<h3>
  
  
  3. Image Consistency Across Scenes
</h3>

<p>One early problem: scene images looked completely different from each other. The player's character would appear to change ethnicity, age, and clothing between images because each generation started from scratch.</p>

<p>The fix: capture a visual description of the player's character the first time the player speaks, and inject it into every scene prompt.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># In event_pipeline.py — first player transcript triggers character capture
</span><span class="k">if</span> <span class="ow">not</span> <span class="n">session_state</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">character_visual</span><span class="sh">"</span><span class="p">)</span> <span class="ow">and</span> <span class="n">is_player_turn</span><span class="p">:</span>
    <span class="n">character_visual</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">extract_character_visual</span><span class="p">(</span><span class="n">transcript_text</span><span class="p">)</span>
    <span class="n">session_state</span><span class="p">[</span><span class="sh">"</span><span class="s">character_visual</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">character_visual</span>

<span class="c1"># Every scene prompt includes:
</span><span class="sa">f</span><span class="sh">"</span><span class="s">CHARACTER APPEARANCE (keep consistent): </span><span class="si">{</span><span class="n">session_state</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">character_visual</span><span class="sh">'</span><span class="p">,</span> <span class="sh">''</span><span class="p">)</span><span class="si">}</span><span class="sh">"</span>
</code></pre>

</div>



<p>The visual description is extracted once and reused. Scenes now show the same character across every image in the session.</p>

<h3>
  
  
  4. Story Loom System
</h3>

<p>Each session needs a story that feels driven and purposeful, not procedurally generic. The Story Loom generates a campaign arc before the session begins using per-world d12 tables.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Formula: [ACTOR] wants to [ACTION] [SUBJECT] to [INTENT], but [DEVELOPMENT]
# Example output for ember_waste:
# "A rogue Iron Tide harpooneer who deserted after a failed harvest
#  is tracking a wounded juvenile Ash Strider
#  to harvest enough raw ichor to buy passage off the Char,
#  but the beast is already claimed by the Red Teeth — and they know she's here."
</span>
<span class="k">def</span> <span class="nf">roll_campaign_arc</span><span class="p">(</span><span class="n">world_key</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">CampaignArc</span><span class="p">:</span>
    <span class="n">tables</span> <span class="o">=</span> <span class="n">_LOOM_TABLES</span><span class="p">[</span><span class="n">world_key</span><span class="p">]</span>
    <span class="k">return</span> <span class="nc">CampaignArc</span><span class="p">(</span>
        <span class="n">actor</span><span class="o">=</span><span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">tables</span><span class="p">[</span><span class="sh">"</span><span class="s">actor</span><span class="sh">"</span><span class="p">]),</span>
        <span class="n">action</span><span class="o">=</span><span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">tables</span><span class="p">[</span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">]),</span>
        <span class="n">subject</span><span class="o">=</span><span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">tables</span><span class="p">[</span><span class="sh">"</span><span class="s">subject</span><span class="sh">"</span><span class="p">]),</span>
        <span class="n">intent</span><span class="o">=</span><span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">tables</span><span class="p">[</span><span class="sh">"</span><span class="s">intent</span><span class="sh">"</span><span class="p">]),</span>
        <span class="n">development</span><span class="o">=</span><span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">tables</span><span class="p">[</span><span class="sh">"</span><span class="s">development</span><span class="sh">"</span><span class="p">]),</span>
        <span class="n">phase</span><span class="o">=</span><span class="sh">"</span><span class="s">discovery</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">)</span>
</code></pre>

</div>



<p>The arc unfolds through four phases: discovery → escalation → climax → resolution. Each session advances the phase and generates a beat — a shorter encounter seed tuned to the current arc phase — so sessions feel like chapters of a larger story.</p>

<h3>
  
  
  5. Dynamic Sound from Freesound API
</h3>

<p>Instead of bundling a static sound library, GM-Genie searches Freesound in real time based on what the SceneDetector infers from the transcript.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">async</span> <span class="k">def</span> <span class="nf">fetch_sfx</span><span class="p">(</span><span class="n">description</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">cache_dir</span><span class="p">:</span> <span class="n">Path</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Path</span> <span class="o">|</span> <span class="bp">None</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Search Freesound, download best match, cache to disk.</span><span class="sh">"""</span>
    <span class="n">cache_key</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">"</span><span class="s">[^\w]</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">_</span><span class="sh">"</span><span class="p">,</span> <span class="n">description</span><span class="p">.</span><span class="nf">lower</span><span class="p">())[:</span><span class="mi">40</span><span class="p">]</span>
    <span class="n">cached</span> <span class="o">=</span> <span class="n">cache_dir</span> <span class="o">/</span> <span class="sa">f</span><span class="sh">"</span><span class="s">sfx_</span><span class="si">{</span><span class="n">cache_key</span><span class="si">}</span><span class="s">.mp3</span><span class="sh">"</span>
    <span class="k">if</span> <span class="n">cached</span><span class="p">.</span><span class="nf">exists</span><span class="p">():</span>
        <span class="k">return</span> <span class="n">cached</span>
    <span class="c1"># Hit Freesound API, pick highest-rated result, download
</span>    <span class="bp">...</span>
    <span class="k">return</span> <span class="n">cached</span>
</code></pre>

</div>



<p>First request hits the API. Every subsequent session reuses the cached file. The SFX library builds itself as the game is played.</p>

<h3>
  
  
  6. Audio Pipeline: Continuous Capture, No Noise Gate
</h3>

<p>The AudioWorklet capture processor sends raw 16kHz PCM to the server in 84-byte chunks (2.6ms each). Early versions had a client-side noise gate to reduce API load. That was a mistake.</p>

<p>Gemini Live API has its own voice activity detection. When I added a noise gate on the client, it sent fragmented audio bursts — short bursts of sound separated by hard silences. The API's VAD couldn't recognize these as continuous speech. Player transcripts dropped to zero. The model stopped responding to the player entirely.</p>

<p>Removing the noise gate fixed it immediately.</p>

<p>The server-side fix was batching. 84-byte chunks are too granular for the API, so the server buffers them to ~3200 bytes (~100ms) before forwarding to the <code>LiveRequestQueue</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># server.py — mic batching
</span><span class="n">MIC_BATCH_BYTES</span> <span class="o">=</span> <span class="mi">3200</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">_mic_sender</span><span class="p">(</span><span class="n">live_queue</span><span class="p">,</span> <span class="n">mic_buffer</span><span class="p">):</span>
    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">chunk</span> <span class="o">=</span> <span class="k">await</span> <span class="n">mic_buffer</span><span class="p">.</span><span class="nf">get</span><span class="p">()</span>
        <span class="n">batch</span> <span class="o">=</span> <span class="n">chunk</span>
        <span class="k">while</span> <span class="nf">len</span><span class="p">(</span><span class="n">batch</span><span class="p">)</span> <span class="o">&lt;</span> <span class="n">MIC_BATCH_BYTES</span><span class="p">:</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="n">batch</span> <span class="o">+=</span> <span class="n">mic_buffer</span><span class="p">.</span><span class="nf">get_nowait</span><span class="p">()</span>
            <span class="k">except</span> <span class="n">asyncio</span><span class="p">.</span><span class="n">QueueEmpty</span><span class="p">:</span>
                <span class="k">break</span>
        <span class="n">live_queue</span><span class="p">.</span><span class="nf">send_realtime</span><span class="p">(</span>
            <span class="n">types</span><span class="p">.</span><span class="nc">Blob</span><span class="p">(</span><span class="n">data</span><span class="o">=</span><span class="n">batch</span><span class="p">,</span> <span class="n">mime_type</span><span class="o">=</span><span class="sh">"</span><span class="s">audio/pcm;rate=16000</span><span class="sh">"</span><span class="p">)</span>
        <span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  7. Transcript Deduplication
</h3>

<p>Gemini Live API sends transcripts in two stages: partial transcripts as the model speaks, then a finished transcript containing the complete text for that turn. Early versions concatenated everything — partials plus the finished event — so every player utterance appeared twice in the conversation log.</p>

<p>The fix: display partials to the frontend for real-time feedback, but use only the finished transcript for the conversation log. When the finished event arrives, discard whatever partial buffer was accumulating.</p>

<h3>
  
  
  8. Session Timing and Graceful Endings
</h3>

<p>Voice sessions run on a timer. The ending needs to feel narrative, not abrupt.</p>

<p>The implementation: at 30 seconds remaining, inject a warning into the GM's context. At 5 seconds remaining, inject a hard "STOP NOW" directive. After the timer hits zero, drain audio for 1.5 seconds to let the final sentence finish playing, then close the connection.</p>

<p>The timer pauses during reconnection and force-expires on <code>session_ended</code> events so the display stays accurate regardless of network interruptions.</p>




<h2>
  
  
  The Seven Worlds
</h2>

<p>Players choose from seven hand-crafted settings, each with distinct themes, lore factions, and story tables:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>World</th>
<th>Theme</th>
</tr>
</thead>
<tbody>
<tr>
<td>The Char (ember_waste)</td>
<td>Post-apocalyptic desert survival — Ash Striders, ichor economy, brutal factions</td>
</tr>
<tr>
<td>Neon Ghosts (chrome_cantrips)</td>
<td>Cyberpunk noir with an arcane undercurrent</td>
</tr>
<tr>
<td>The Sundered Skies (leviathan_divide)</td>
<td>Sky-city fantasy — civilization built on the backs of flying leviathans</td>
</tr>
<tr>
<td>The Drowning Sea (abyssal_wake)</td>
<td>Pirate horror with deep-sea dread</td>
</tr>
<tr>
<td>The Verdant Maw (iron_vein_wilds)</td>
<td>Nature-spirit fantasy — the wild is alive and it has opinions</td>
</tr>
<tr>
<td>The Crimson Siege (blood_hegemony)</td>
<td>Gothic vampire warfare</td>
</tr>
<tr>
<td>The Starbound Frontier (neon_frontier)</td>
<td>Space western with spirit bonds</td>
</tr>
</tbody>
</table></div>

<p>Plus a custom text input if none of these fit.</p>




<h2>
  
  
  The DM Framework in the Prompt
</h2>

<p>Architecture gets the model talking. The prompt makes it worth listening to. The system prompt bakes in several techniques drawn from live-play actual play:</p>

<p><strong>E.A.S.E.</strong> (Environment → Atmosphere → Senses → Events): Every scene description hits all four layers in sequence. You don't just hear what the place looks like — you feel the temperature, smell the air, and something happens.</p>

<p><strong>Rule of Three</strong>: Every location has exactly three interactable things. Never fewer (the player needs options), never more (too much to track). End on the most dramatically interesting one.</p>

<p><strong>Cold Open / Sensory Hook</strong>: Sessions start in the middle of something. No "You are an adventurer who has arrived in a town." The first line drops you into a moment.</p>

<p><strong>NPC One Distinct Feature + Want/Fear</strong>: Every named NPC gets one physical or behavioral detail that makes them memorable, plus a clear want and a clear fear. The GM always knows what an NPC will do because it knows why they do anything.</p>

<p><strong>Proactive World</strong>: NPCs don't wait for the player to interact. The world has momentum. Things happen whether the player acts or not.</p>




<h2>
  
  
  Challenges
</h2>

<h3>
  
  
  1. Tool Calls Crashed the Native-Audio Model
</h3>

<p>Function calling in <code>gemini-2.5-flash-native-audio-latest</code> caused WebSocket disconnects ~70% of the time. The errors were 1000/1008/1011 — generic enough that it took a while to correlate them with tool invocation.</p>

<p>Solution: remove all tools from the voice pipeline. Zero function calling in voice mode. Pre-roll dice, move media triggering server-side, handle state transitions in the server without asking the model.</p>

<h3>
  
  
  2. Noise Gate Broke Speech Detection
</h3>

<p>Client-side noise gating sent fragmented audio bursts. Gemini's VAD saw them as noise, not speech. Player turn transcripts flatlined.</p>

<p>Solution: continuous audio stream, no gate. The API handles silence detection.</p>

<h3>
  
  
  3. 84-Byte AudioWorklet Chunks
</h3>

<p>AudioWorklet sends 2.6ms chunks. The Live API needs sustained audio to work correctly.</p>

<p>Solution: buffer in server to ~3200 bytes (100ms) before forwarding to <code>LiveRequestQueue</code>.</p>

<h3>
  
  
  4. Transcript Duplication
</h3>

<p>Gemini sends partials + a final "finished" event containing the full text. Concatenating both produced doubled transcripts.</p>

<p>Solution: partials go to the frontend for real-time display; finished event goes to the conversation log; partial buffer discarded on finish.</p>

<h3>
  
  
  5. Filler Sounds Burned Quota
</h3>

<p>Early versions played filler audio ("Hmm...", "Let me think...") via separate TTS connections. Each one consumed Live API RPM quota and occasionally triggered its own reconnection.</p>

<p>Solution: inject fillers directly into the live session's <code>LiveRequestQueue</code>. No separate connection, no quota hit, zero latency overhead.</p>

<h3>
  
  
  6. Image Generation Quota Limits
</h3>

<p><code>gemini-2.5-flash-image</code> has a tight daily quota on free tier (~50 requests/day). Aggressive scene generation during testing exhausted it within hours.</p>

<p>Solution: switch to <code>gemini-3-pro-image-preview</code>, implement cooldowns (10s for visual cues, 15s for location triggers), and cache everything to disk.</p>




<h2>
  
  
  Tech Stack
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Technology</th>
</tr>
</thead>
<tbody>
<tr>
<td>Voice Engine</td>
<td>gemini-2.5-flash-native-audio-latest (WebSocket bidi)</td>
</tr>
<tr>
<td>Scene Generation</td>
<td>gemini-3-pro-image-preview</td>
</tr>
<tr>
<td>Text Fallback</td>
<td>gemini-2.5-flash + Google ADK</td>
</tr>
<tr>
<td>Sound Effects</td>
<td>Freesound API (dynamic search + disk cache)</td>
</tr>
<tr>
<td>Frontend</td>
<td>React 19 + Vite + Tailwind CSS v4</td>
</tr>
<tr>
<td>Backend</td>
<td>FastAPI + Uvicorn on Google Cloud Run</td>
</tr>
<tr>
<td>Audio Processing</td>
<td>Web Audio API + AudioWorklet</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  What I Learned
</h2>

<p><strong>1. Zero-tool architectures beat reliable-tool architectures for real-time audio.</strong><br>
When you're in a bidi audio stream, every tool call is a potential connection interruption. Moving intelligence to the server — where it's synchronous, controllable, and doesn't touch the model connection — is almost always the right call.</p>

<p><strong>2. Server-side intelligence scales better than client-side complexity.</strong><br>
The SceneDetector is 300 lines of Python with regex patterns and cooldown timers. It's simple to debug, easy to extend, and never hallucinates. Compare that to asking the model to decide when to show an image.</p>

<p><strong>3. The GM's personality IS the product.</strong><br>
Users don't notice the architecture. They notice whether the GM makes them feel something. Prompt engineering — the E.A.S.E. structure, the cold opens, the proactive NPCs — mattered more than any infrastructure decision.</p>

<p><strong>4. Pre-rolling dice beats function calling for latency.</strong><br>
Injecting a pool of results into the system prompt means zero round-trips during the session. The model reads the pool, narrates the results, and moves on. Latency drops to zero. The numbers are just as random.</p>

<p><strong>5. Continuous audio streams beat noise-gated streams for speech detection.</strong><br>
This one cost me two days. Trust the API's VAD.</p>




<h2>
  
  
  What's Next
</h2>

<ul>
<li>Multi-session campaigns with persistent world state between sessions</li>
<li>NPC voice portraits — each named NPC gets a distinct voice</li>
<li>Community-created world templates</li>
<li>Mobile-optimized UI for actual tabletop use</li>
</ul>




<p><em>Built with Google ADK, Gemini 2.5 Flash Native Audio, and way too much coffee.</em></p>

<p><em>#GeminiLiveAgentChallenge</em></p>

