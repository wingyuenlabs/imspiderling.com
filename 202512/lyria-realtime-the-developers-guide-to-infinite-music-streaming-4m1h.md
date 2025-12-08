---
Title: Lyria RealTime: The Developer’s Guide to Infinite Music Streaming
Description: 
Author: Guillaume Vernade
Date: 2025-12-08T21:31:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>You love generating static songs with classic text to music models? Prepare to conduct a never-ending symphony. Introducing <strong><a href="https://deepmind.google/models/lyria/lyria-realtime/" rel="noopener noreferrer">Lyria RealTime</a></strong>, Google DeepMind’s experimental model that doesn't just generate music—it <em>jams</em> with you like it did during the Toro y Moi IO pre-show:</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/thAhd82XnMc">
</iframe>
</p>

<p>While traditional music generation models work like a jukebox (input prompt -&gt; wait -&gt; get song), Lyria RealTime operates on the principle of <strong>"Music as a Verb."</strong> It creates a persistent, bidirectional streaming connection that produces a continuous 48kHz stereo stream. You can steer, warp, and morph the audio in the moment, making it the first generative model truly designed for interactive experiences.</p>

<p>And the best part? Right now the model is <strong>free to use</strong>!</p>

<p>Here's quick summary of what you'll learn about in this guide:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe91qwp3ixq4cqnpqmuvk.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe91qwp3ixq4cqnpqmuvk.jpeg" alt="Lyria RealTime infographics" width="800" height="436"></a></p>



<p>This guide will walk you through building with Lyria RealTime using the Gemini API.</p>

<p><strong>This guide will cover:</strong></p>

<ol>
<li> <strong>How Lyria RealTime Works (The "Goldfish Memory" Architecture)</strong>
</li>
<li> <strong>Project Setup</strong>
</li>
<li> <strong>Basic Streaming (The "Hello World" of Music)</strong>
</li>
<li> <strong>Steering the Stream (Weighted Prompts)</strong>
</li>
<li> <strong>Advanced Configuration (BPM, Density, &amp; Scale)</strong>
</li>
<li> <strong>Blueprints for the Future: Advanced Use Cases</strong>
</li>
<li> <strong>Prompting Strategies &amp; Best Practices</strong>
</li>
<li> <strong>Where to play with Lyria Real Time</strong>
</li>
</ol>

<p>Jump directly to the last section if you want to play directly with Lyria RealTime, for ex. as a <a href="https://aistudio.google.com/apps/bundled/promptdj-midi" rel="noopener noreferrer">DJ</a>, driving a <a href="https://aistudio.google.com/apps/bundled/spacedj" rel="noopener noreferrer">spaceship</a> or using your <a href="https://aistudio.google.com/apps/bundled/lyria_camera" rel="noopener noreferrer">camera</a>.</p>

<blockquote>
<p><strong>Note</strong>: for an interactive version of this post, checkout the <a href="https://colab.research.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_started_LyriaRealTime.ipynb" rel="noopener noreferrer">python cookbook</a>.</p>
</blockquote>


<h2>
  
  
  1) How Lyria RealTime Works
</h2>

<p>Lyria RealTime uses a low-latency WebSocket connection to maintain a live communication channel with the model. Unlike offline models that plan a whole song structure (Intro-Verse-Chorus), Lyria operates on a <strong>chunk-based autoregression</strong> system.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb2uub6nyc7bnp6e61saa.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb2uub6nyc7bnp6e61saa.gif" alt="How Lyria RealTime Works" width="720" height="405"></a></p>

<p>It generates audio in 2-second chunks, looking back for a few seconds of context to maintain the rhythmic "groove" while looking forward at your current controls to decide the style. This means the model doesn't "compose songs" in the traditional sense; it navigates musical states.</p>


<h2>
  
  
  2) Project Setup
</h2>

<p>To follow this guide, you will need:</p>

<ul>
<li>  An API key from Google AI Studio (it can be a free one).</li>
<li>  The Google Gen AI SDK.</li>
</ul>

<p><strong>Install the SDK:</strong><br>
<strong>Python</strong> (<code>3.12+</code> recommended):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install</span> <span class="s2">"google-genai&gt;=1.52.0"</span>
</code></pre>

</div>



<p><strong>JavaScript / TypeScript:</strong><br>
You'll need at least the 1.30 version of the <a href="https://googleapis.github.io/js-genai/" rel="noopener noreferrer">JS/TS SDK</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @google/genai
</code></pre>

</div>



<blockquote>
<p><strong>Note</strong>: The following examples use the Python SDK for demonstration. For JS/TS code sample, check the <a href="https://aistudio.google.com/apps/bundled/promptdj-midi?showAssistant=true&amp;showCode=true" rel="noopener noreferrer">AI studio Apps</a>.</p>
</blockquote>




<h2>
  
  
  3) Basic Streaming
</h2>

<p>To start a session, you connect to the model (<code>models/lyria-realtime-exp</code>), send an initial configuration, and start the stream. The interaction loop is asynchronous: you send commands, and the server continuously yields raw audio chunks.</p>

<p><em>[Note: Ensure you are using the <code>v1alpha</code> API version for experimental models like Lyria]</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">google</span> <span class="kn">import</span> <span class="n">genai</span>
<span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>

<span class="n">client</span> <span class="o">=</span> <span class="n">genai</span><span class="p">.</span><span class="nc">Client</span><span class="p">(</span><span class="n">http_options</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">api_version</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">v1alpha</span><span class="sh">'</span><span class="p">})</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="k">async</span> <span class="k">def</span> <span class="nf">receive_audio</span><span class="p">(</span><span class="n">session</span><span class="p">):</span>
        <span class="sh">"""</span><span class="s">Background task to process incoming audio chunks.</span><span class="sh">"""</span>
        <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
            <span class="k">async</span> <span class="k">for</span> <span class="n">message</span> <span class="ow">in</span> <span class="n">session</span><span class="p">.</span><span class="nf">receive</span><span class="p">():</span>
                <span class="k">if</span> <span class="n">message</span><span class="p">.</span><span class="n">server_content</span><span class="p">.</span><span class="n">audio_chunks</span><span class="p">:</span>
                    <span class="c1"># 'data' is raw 16-bit PCM audio at 48kHz
</span>                    <span class="n">audio_data</span> <span class="o">=</span> <span class="n">message</span><span class="p">.</span><span class="n">server_content</span><span class="p">.</span><span class="n">audio_chunks</span><span class="p">.</span><span class="n">data</span>
                    <span class="c1"># Add your audio playback logic here!
</span>            <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">10</span><span class="o">**-</span><span class="mi">12</span><span class="p">)</span>

    <span class="k">async</span> <span class="nf">with </span><span class="p">(</span>
        <span class="n">client</span><span class="p">.</span><span class="n">aio</span><span class="p">.</span><span class="n">live</span><span class="p">.</span><span class="n">music</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">model</span><span class="o">=</span><span class="sh">'</span><span class="s">models/lyria-realtime-exp</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">session</span><span class="p">,</span>
        <span class="n">asyncio</span><span class="p">.</span><span class="nc">TaskGroup</span><span class="p">()</span> <span class="k">as</span> <span class="n">tg</span><span class="p">,</span>
    <span class="p">):</span>
        <span class="c1"># 1. Start listening for audio
</span>        <span class="n">tg</span><span class="p">.</span><span class="nf">create_task</span><span class="p">(</span><span class="nf">receive_audio</span><span class="p">(</span><span class="n">session</span><span class="p">))</span>

        <span class="c1"># 2. Send initial musical concept
</span>        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">set_weighted_prompts</span><span class="p">(</span>
            <span class="n">prompts</span><span class="o">=</span><span class="p">[</span><span class="n">types</span><span class="p">.</span><span class="nc">WeightedPrompt</span><span class="p">(</span><span class="n">text</span><span class="o">=</span><span class="sh">'</span><span class="s">elevator music</span><span class="sh">'</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="mf">1.0</span><span class="p">)]</span>
        <span class="p">)</span>

        <span class="c1"># 3. Set the vibe (BPM, Temperature)
</span>        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">set_music_generation_config</span><span class="p">(</span>
            <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">LiveMusicGenerationConfig</span><span class="p">(</span><span class="n">bpm</span><span class="o">=</span><span class="mi">90</span><span class="p">,</span> <span class="n">temperature</span><span class="o">=</span><span class="mf">1.0</span><span class="p">)</span>
        <span class="p">)</span>

        <span class="c1"># 4. Drop the beat
</span>        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">play</span><span class="p">()</span>

        <span class="c1"># Keep the session alive
</span>        <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">30</span><span class="p">)</span> 

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">asyncio</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="nf">main</span><span class="p">())</span>
</code></pre>

</div>



<p>Congratulations, you've got some elevator music!</p>

<p>Not impressed? That's just the beginning, dear padawan, now comes the cool part.</p>




<h2>
  
  
  4) Steering the Stream (Weighted Prompts)
</h2>

<p>This is where the magic happens. Unlike static generation, you can send new <code>WeightedPrompt</code> messages <em>while the music is playing</em> to smoothly transition the genre, instruments, or mood.</p>

<p>The <code>weight</code> parameter is your fader. A weight of <code>1.0</code> is standard, but you can use multiple prompts to blend influences.</p>

<p><strong>Example: Morphing from Piano to Live Performance</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>

<span class="c1"># Send this while the loop is running to shift the style
</span><span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">set_weighted_prompts</span><span class="p">(</span>
    <span class="n">prompts</span><span class="o">=</span><span class="p">[</span>
        <span class="c1"># Keep the piano strong
</span>        <span class="p">{</span><span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Piano</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">weight</span><span class="sh">"</span><span class="p">:</span> <span class="mf">2.0</span><span class="p">},</span>
        <span class="c1"># Add a subtle meditative layer
</span>        <span class="n">types</span><span class="p">.</span><span class="nc">WeightedPrompt</span><span class="p">(</span><span class="n">text</span><span class="o">=</span><span class="sh">"</span><span class="s">Meditation</span><span class="sh">"</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="mf">0.5</span><span class="p">),</span>
        <span class="c1"># Push the 'Live' feeling
</span>        <span class="n">types</span><span class="p">.</span><span class="nc">WeightedPrompt</span><span class="p">(</span><span class="n">text</span><span class="o">=</span><span class="sh">"</span><span class="s">Live Performance</span><span class="sh">"</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="mf">1.0</span><span class="p">),</span>
    <span class="p">]</span>
<span class="p">)</span>
</code></pre>

</div>



<blockquote>
<p><strong>Note:</strong> As the model generates chunks after chunks, the changes can take a few seconds (usually around 2s) to be reflected in the music.</p>
</blockquote>

<h4>
  
  
  Pro Tip: Cross-fading
</h4>

<p>Drastic prompt changes can be abrupt. For professional results, implement client-side cross-fading by sending intermediate weight values rapidly (e.g., every 500ms) to "morph" the music smoothly.</p>

<p><strong>Example: The "Morph" Function</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">cross_fade</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="n">old_prompt</span><span class="p">,</span> <span class="n">new_prompt</span><span class="p">,</span> <span class="n">duration</span><span class="o">=</span><span class="mf">2.0</span><span class="p">,</span> <span class="n">steps</span><span class="o">=</span><span class="mi">10</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">Smoothly morphs from one musical idea to another.</span><span class="sh">"""</span>
    <span class="n">step_time</span> <span class="o">=</span> <span class="n">duration</span> <span class="o">/</span> <span class="n">steps</span>

    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">steps</span> <span class="o">+</span> <span class="mi">1</span><span class="p">):</span>
        <span class="c1"># Calculate the blend ratio (alpha goes from 0.0 to 1.0)
</span>        <span class="n">alpha</span> <span class="o">=</span> <span class="n">i</span> <span class="o">/</span> <span class="n">steps</span>

        <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">set_weighted_prompts</span><span class="p">(</span>
            <span class="n">prompts</span><span class="o">=</span><span class="p">[</span>
                <span class="c1"># Fade out the old
</span>                <span class="n">types</span><span class="p">.</span><span class="nc">WeightedPrompt</span><span class="p">(</span><span class="n">text</span><span class="o">=</span><span class="n">old_prompt</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="mf">1.0</span> <span class="o">-</span> <span class="n">alpha</span><span class="p">),</span>
                <span class="c1"># Fade in the new
</span>                <span class="n">types</span><span class="p">.</span><span class="nc">WeightedPrompt</span><span class="p">(</span><span class="n">text</span><span class="o">=</span><span class="n">new_prompt</span><span class="p">,</span> <span class="n">weight</span><span class="o">=</span><span class="n">alpha</span><span class="p">),</span>
            <span class="p">]</span>
        <span class="p">)</span>
        <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">step_time</span><span class="p">)</span>

<span class="c1"># Usage in your main loop:
# Morph from 'Ambient' to 'Techno' over 5 seconds
</span><span class="k">await</span> <span class="nf">cross_fade</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="sh">"</span><span class="s">Ambient Drone</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">Hard Techno</span><span class="sh">"</span><span class="p">,</span> <span class="n">duration</span><span class="o">=</span><span class="mf">5.0</span><span class="p">)</span>
</code></pre>

</div>



<p>Note that this code sample assumes all your prompts have a weight of 1 which might not be the case. </p>




<h2>
  
  
  5) Advanced Configuration (The Knobs)
</h2>

<p>Lyria RealTime exposes parametric controls that change the structure of the music. If you aren't a musician, think of these controls as the physics of the audio world:</p>

<ul>
<li>
<strong>Density</strong> (0.0 - 1.0): Think of this as "Busyness."

<ul>
<li>
<em>Low</em> (0.1): A lonely drummer playing once every few seconds. Sparse.</li>
<li>
<em>High</em> (0.9): A chaotic orchestra where everyone plays at once. Intense.</li>
</ul>


</li>

<li>

<strong>Brightness</strong> (0.0 - 1.0): Think of this as "Muffled vs. Crisp."

<ul>
<li>
<em>Low</em> (0.1): Listening to music from outside a club, through a wall. Dark and bass-heavy.</li>
<li>
<em>High</em> (0.9): Listening through high-end headphones. Sharp, clear, and treble-heavy.</li>
</ul>


</li>

<li>

<strong>BPM</strong> (60 - 200): The heartbeat of the track (Beats Per Minute).</li>

<li>

<strong>Scale</strong>: The "Mood." It forces the music into a specific set of notes (Key/Mode).</li>

</ul>

<blockquote>
<p><strong>Important</strong>: While density and brightness can be changed smoothly on the fly, changing the BPM or Scale is a fundamental structural shift. You must call <code>reset_context()</code> for these changes to take effect. This will clear the model's "short-term memory," causing a hard cut in the audio.</p>
</blockquote>

<p><strong>Example: The "Hard Drop"</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Changing structural parameters requires a context reset
</span><span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">set_music_generation_config</span><span class="p">(</span>
    <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">LiveMusicGenerationConfig</span><span class="p">(</span>
        <span class="n">bpm</span><span class="o">=</span><span class="mi">140</span><span class="p">,</span> 
        <span class="n">scale</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="n">Scale</span><span class="p">.</span><span class="n">C_MAJOR_A_MINOR</span><span class="p">,</span> <span class="c1"># Force happy/neutral mood
</span>    <span class="p">)</span>
<span class="p">)</span>

<span class="c1"># This command is mandatory for BPM/Scale changes to apply!
</span><span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">reset_context</span><span class="p">()</span>
</code></pre>

</div>






<h2>
  
  
  6) Blueprints for the Future: Advanced Use Cases
</h2>

<p>We’ve covered basic streaming, but Lyria’s parametric controls allow for applications that connect the physical world to the audio stream. Here are four ideas to get you started.</p>

<h3>
  
  
  Use Case A: The "Biometric Beat" (Fitness &amp; Health)
</h3>

<p>Most fitness apps use static playlists that rarely match your actual pace. Because Lyria allows for real-time <code>bpm</code> and <code>density</code> control, you can build a music engine that is biologically coupled to the user.</p>

<ul>
<li>  <strong>Heart Rate Monitor (HRM) -&gt; BPM:</strong> Map the user's heart rate directly to the track's tempo.</li>
<li>  <strong>Accelerometer -&gt; Density:</strong> If the user is sprinting (high variance in movement), increase <code>density</code> to <code>1.0</code> to add percussion and complexity. If they stop to rest, drop <code>density</code> to <code>0.2</code> for an ambient breakdown.</li>
</ul>

<h3>
  
  
  Use Case B: The "Democratic DJ" (Social Streaming)
</h3>

<p>Since <code>WeightedPrompts</code> accept float values, you can build a collaborative radio station for Twitch streams or Discord bots where the audience votes on the genre. Instead of a winner-take-all system, Lyria can blend the votes.</p>

<ul>
<li>  <strong>Input:</strong> 100 users vote. 60 vote "<em>Cyberpunk</em>", 30 vote "<em>Jazz</em>", 10 vote "<em>Reggae</em>".</li>
<li>  <strong>Normalization:</strong> Convert votes to weights (0.6, 0.3, 0.1).</li>
<li>  <strong>Result:</strong> The model generates a dominant Cyberpunk track with clear Jazz harmonies and a subtle Reggae backbeat and changes it overtime according to the votes.</li>
</ul>

<h3>
  
  
  Use Case C: "Focus Flow" (Productivity)
</h3>

<p>Deep work requires different audio textures than brainstorming. You can map Lyria's <code>brightness</code> and <code>guidance</code> parameters to a Pomodoro timer to guide the user's cognitive state.</p>

<ul>
<li>  <strong>Deep Work Phase:</strong> Low <code>brightness</code> (darker, warmer sounds), Low <code>density</code> (minimal distractions), High <code>guidance</code> (repetitive, predictable).</li>
<li>  <strong>Break Phase:</strong> High <code>brightness</code> (energetic, crisp), High <code>density</code>, Low <code>guidance</code> (creative, surprising).</li>
</ul>

<h3>
  
  
  Use Case D: "Realtime Game music" (Gaming)
</h3>

<p>Coming from the gaming industry I could not avoid thinking of a gaming idea for Lyria Real Time. You could have Lyria create the music of the game in real time based on:</p>

<ul>
<li>  <strong>The game's own style:</strong> a bunch of prompts that defines the game and the overall ambiance,</li>
<li>  <strong>The environment:</strong> different prompts depending on whether you're in a busy city, in a forest or sailing the Greek seas,</li>
<li>  <strong>The player's action:</strong> are they fighting, then add the "epic" prompt, investigating instead, change it for the "mysterious" one,</li>
<li>  <strong>The players' current condition:</strong> You could change the BPM and the weight of a "danger" prompt depending on the player's health bar. The lower it is, the more stressful the music would be.</li>
</ul>




<h2>
  
  
  7) Prompting Strategies &amp; Best Practices
</h2>

<p><strong>The Prompt Formula:</strong><br>
Through testing, a reliable formula has emerged: <strong>[Genre Anchor] + [Instrumentation] + [Atmosphere]</strong>...</p>

<ul>
<li>  <strong>Instruments:</strong> <em>303 Acid Bass, Buchla Synths, Hang Drum, TR-909 Drum Machine</em>...</li>
<li>  <strong>Genres:</strong> <em>Acid Jazz, Bengal Baul, Glitch Hop, Shoegaze, Vaporwave</em>...</li>
<li>  <strong>Moods:</strong> <em>Crunchy Distortion, Ethereal Ambience, Ominous Drone, Swirling Phasers</em>...</li>
</ul>

<p><strong>Developer Best Practices:</strong></p>

<ul>
<li>  <strong>Buffer Your Audio:</strong> Because this is real-time streaming over the network, implement client-side audio buffering (2-3 chunks) to handle network jitter and ensure smooth playback.</li>
<li>  <strong>The "Settling" Period:</strong> When you start a stream or reset context, the model needs about 5-10 seconds to "settle" into a stable groove.</li>
<li>  <strong>Safety Filters:</strong> The model checks prompts against safety filters. Avoid asking for specific copyrighted artists ("Style of Taylor Swift"); instead, deconstruct their sound into descriptors ("Pop, female vocals, acoustic guitar").</li>
<li>  <strong>Instrumental Only:</strong> The model is only instrumental. While you can set <code>music_generation_mode</code> to <code>VOCALIZATION</code>, it produces vocal-like textures (oohs/aahs), not coherent lyrics.</li>
<li>  <strong>Session duration limit:</strong> The session are currently limited to 10mn, but you can just restart a new one afterwards.</li>
</ul>

<p>More details and prompt ideas in Lyria RealTime's <a href="https://ai.google.dev/gemini-api/docs/music-generation" rel="noopener noreferrer">documentation</a>.</p>


<h2>
  
  
  8. Ready to Jam? Choose your preferred way to play with Lyria RealTime
</h2>

<p>One of the easiest places to try is AI Studio, where a couple of cool apps are available for you to play with, and to vibe-customize to your needs:</p>

<ul>
<li>  <strong><a href="https://aistudio.google.com/apps/bundled/promptdj" rel="noopener noreferrer">Prompt DJ</a></strong>, <strong><a href="https://aistudio.google.com/apps/bundled/promptdj-midi" rel="noopener noreferrer">MIDI DJ</a></strong> and <strong><a href="https://labs.google/fx/tools/music-fx-dj" rel="noopener noreferrer">MusicFX</a></strong> (US only) let you add and mix multiple prompts in real time:</li>
</ul>

<p></p>

<ul>
<li>  <strong><a href="https://aistudio.google.com/apps/bundled/spacedj" rel="noopener noreferrer">Space DJ</a></strong> lets you navigate the universe of music genders with a spacecraft! I personally love navigating around the <em>italo-disco</em> and <em>euro-france</em> planets.</li>
</ul>

<p></p>

<ul>
<li>   <strong><a href="https://aistudio.google.com/apps/bundled/lyria_camera" rel="noopener noreferrer">Lyria Camera</a></strong> creates music in real time based on what it sees. I'd love to have that connected to my dashcam!</li>
</ul>

<p></p>

<ul>
<li><p>The <strong><a href="https://magenta.withgoogle.com/demos" rel="noopener noreferrer">Magenta website</a></strong> also features a lot of cool demos. It's also a great place to get more details on Deepmind's music generation models.</p></li>
<li><p>Finally, <strong>check the <a href="https://github.com/Giom-V/magic-mirror" rel="noopener noreferrer">magical mirror</a> demo</strong> I made that uses Lyria to create background music according to what it tells (Gemini generates the prompts on the fly):</p></li>
</ul>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/Z7FUh8B4im4">
</iframe>
</p>

<p>And now the floor is yours, what will you create using Lyria RealTime?</p>

<h4>
  
  
  Resources:
</h4>

<ul>
<li><a href="https://ai.google.dev/gemini-api/docs/music-generation" rel="noopener noreferrer">Documentation</a></li>
<li>Magenta <a href="http://magenta.withgoogle.com/" rel="noopener noreferrer">website</a> and <a href="https://magenta.withgoogle.com/blog" rel="noopener noreferrer">blog</a> for the latest news on the music generation models.</li>
<li>AI Studio <a href="https://aistudio.google.com/apps?source=showcase&amp;showcaseTag=gen-media" rel="noopener noreferrer">gen-media apps</a>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvk7aa727myswxdfemkja.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvk7aa727myswxdfemkja.png" alt="Congratulation" width="800" height="441"></a></p>

