---
Title: Memoo - Record once, run anywhere
Description: 
Author: Miguel
Date: 2026-03-16T21:47:21.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Disclaimer: This blog post was created for the purposes of entering the #GeminiLiveAgentChallenge hackathon.</strong></p>

<h2>
  
  
  Introduction
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw1rxlqspl62uv9qmxlux.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw1rxlqspl62uv9qmxlux.gif" alt=" " width="600" height="342"></a></p>

<p>In the world of business automation, repetitive browser workflows are everywhere—data entry, form submissions, report generation, and routine testing. But most automation tools require extensive coding or fragile CSS selectors that break when websites change.</p>

<p>Enter <strong>Memoo</strong>, a multimodal AI-powered UI Navigator that watches your screen, listens to voice context, and transforms one-time workflows into reusable, executable playbooks with step-by-step evidence.</p>

<p>Built with <strong>Google Gemini models</strong> and deployed entirely on <strong>Google Cloud</strong>, Memoo represents a new paradigm for browser automation—one that combines vision understanding, live voice interaction, and cloud-native infrastructure.</p>

<h2>
  
  
  The Problem: Why Traditional Automation Falls Short
</h2>

<p>Traditional browser automation tools like Selenium, Playwright, and Puppeteer are powerful but come with significant limitations:</p>

<ol>
<li>
<strong>Fragile Selectors</strong>: CSS selectors break when websites change</li>
<li>
<strong>No Context</strong>: Tools don't understand <em>why</em> you're clicking something</li>
<li>
<strong>Blind Execution</strong>: No awareness of page state or unexpected changes</li>
<li>
<strong>High Learning Curve</strong>: Requires programming knowledge</li>
<li>
<strong>Limited Reusability</strong>: Hardcoded values don't adapt to different data</li>
</ol>

<p>Memoo solves these problems by using <strong>Google Gemini's multimodal capabilities</strong> to see, understand, and interact with browsers the way a human would.</p>

<h2>
  
  
  Core Architecture
</h2>

<p>Memoo consists of five main surfaces:</p>

<ul>
<li>
<strong>Web Frontend</strong> (<code>apps/web</code>): Next.js App Router with the live capture interface</li>
<li>
<strong>API Backend</strong> (<code>apps/api</code>): FastAPI server with Gemini integrations</li>
<li>
<strong>Video Studio</strong> (<code>apps/video</code>): Remotion-based demo video generation</li>
<li>
<strong>Agent Service</strong> (<code>apps/agent</code>): Stagehand-compatible autonomous browser agent</li>
<li>
<strong>Sandbox</strong> (<code>apps/sandbox</code>): Visible Chromium sandbox for live playback</li>
</ul>

<p>The stack is deployed on <strong>Google Cloud</strong> with:</p>

<ul>
<li>Cloud Run (web + API)</li>
<li>Cloud SQL (PostgreSQL)</li>
<li>Cloud Storage (evidence assets)</li>
<li>Compute Engine (visible browser sandbox)</li>
<li>Secret Manager (credentials)</li>
<li>Artifact Registry (container images)</li>
<li>Cloud Build (CI/CD pipeline)</li>
</ul>

<h2>
  
  
  Google AI Integrations
</h2>

<h3>
  
  
  1. Gemini Vision: Real-Time Screen Understanding
</h3>

<p>The core of Memoo's <strong>Teach Mode</strong> is its ability to watch a user's screen and detect meaningful actions in real-time. This uses <strong>Gemini 2.0 Flash</strong>'s multimodal vision capabilities.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># apps/api/app/services/gemini_live.py
</span>
<span class="kn">from</span> <span class="n">google</span> <span class="kn">import</span> <span class="n">genai</span>
<span class="kn">from</span> <span class="n">google.genai</span> <span class="kn">import</span> <span class="n">types</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">analyse_frame</span><span class="p">(</span>
    <span class="n">image_b64</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
    <span class="n">previous_events</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">],</span>
    <span class="n">mime_type</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="sh">'</span><span class="s">image/jpeg</span><span class="sh">'</span><span class="p">,</span>
<span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">Send a screenshot frame to Gemini Vision and return detected events.</span><span class="sh">"""</span>

    <span class="n">client</span> <span class="o">=</span> <span class="n">genai</span><span class="p">.</span><span class="nc">Client</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="n">google_api_key</span><span class="p">)</span>
    <span class="n">image_bytes</span> <span class="o">=</span> <span class="n">base64</span><span class="p">.</span><span class="nf">b64decode</span><span class="p">(</span><span class="n">image_b64</span><span class="p">)</span>

    <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">client</span><span class="p">.</span><span class="n">aio</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
        <span class="n">model</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="n">gemini_model</span><span class="p">,</span>
        <span class="n">contents</span><span class="o">=</span><span class="p">[</span>
            <span class="n">types</span><span class="p">.</span><span class="nc">Content</span><span class="p">(</span>
                <span class="n">parts</span><span class="o">=</span><span class="p">[</span>
                    <span class="n">types</span><span class="p">.</span><span class="n">Part</span><span class="p">.</span><span class="nf">from_text</span><span class="p">(</span><span class="n">prompt</span><span class="p">),</span>
                    <span class="n">types</span><span class="p">.</span><span class="n">Part</span><span class="p">.</span><span class="nf">from_bytes</span><span class="p">(</span><span class="n">data</span><span class="o">=</span><span class="n">image_bytes</span><span class="p">,</span> <span class="n">mime_type</span><span class="o">=</span><span class="n">mime_type</span><span class="p">),</span>
                <span class="p">]</span>
            <span class="p">)</span>
        <span class="p">],</span>
        <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
            <span class="n">response_mime_type</span><span class="o">=</span><span class="sh">'</span><span class="s">application/json</span><span class="sh">'</span><span class="p">,</span>
        <span class="p">),</span>
    <span class="p">)</span>
</code></pre>

</div>



<p>The vision prompt asks Gemini to:</p>

<ul>
<li>Detect meaningful browser interactions (navigations, clicks, inputs, submissions)</li>
<li>Identify CSS selectors or human-readable element descriptions</li>
<li>Capture observed text from the screen</li>
<li>Provide confidence scores for each detection</li>
<li>Include visible evidence cues</li>
</ul>

<p><strong>Key Features:</strong></p>

<ul>
<li>
<strong>Grounded Detection</strong>: Only reports actions supported by visible evidence</li>
<li>
<strong>Context Awareness</strong>: Maintains history of previously detected actions to avoid duplicates</li>
<li>
<strong>Selector Intelligence</strong>: Prefers descriptive selectors ("Login button") over fragile CSS</li>
<li>
<strong>Confidence Scoring</strong>: Returns confidence values to filter low-quality detections</li>
</ul>

<h3>
  
  
  2. Gemini Live: Voice Navigation Assistant
</h3>

<p>Memoo's <strong>Navigator Live</strong> feature uses the <strong>Gemini Live API</strong> for real-time voice interaction during workflow recording. Implemented using the official <code>@google/genai</code> SDK:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// apps/web/src/hooks/use-gemini-live.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">GoogleGenAI</span><span class="p">,</span> <span class="nx">Modality</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@google/genai</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">ai</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">GoogleGenAI</span><span class="p">({</span> <span class="nx">apiKey</span> <span class="p">});</span>

<span class="kd">const</span> <span class="nx">session</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">ai</span><span class="p">.</span><span class="nx">live</span><span class="p">.</span><span class="nf">connect</span><span class="p">({</span>
  <span class="na">model</span><span class="p">:</span> <span class="dl">'</span><span class="s1">gemini-2.0-flash-exp</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">config</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">responseModalities</span><span class="p">:</span> <span class="p">[</span><span class="nx">Modality</span><span class="p">.</span><span class="nx">AUDIO</span><span class="p">],</span>
    <span class="na">inputAudioTranscription</span><span class="p">:</span> <span class="p">{},</span>
    <span class="na">systemInstruction</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">parts</span><span class="p">:</span> <span class="p">[{</span>
        <span class="na">text</span><span class="p">:</span> <span class="s2">`You are Memoo Navigator, a calm workflow recording co-pilot.
        When you receive "[STEP DETECTED] &lt;description&gt;", decide whether
        to ask a short clarifying question or stay silent if the step is obvious.`</span>
      <span class="p">}]</span>
    <span class="p">},</span>
    <span class="na">speechConfig</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">voiceConfig</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">prebuiltVoiceConfig</span><span class="p">:</span> <span class="p">{</span> <span class="na">voiceName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Puck</span><span class="dl">'</span> <span class="p">},</span>
      <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
  <span class="na">callbacks</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">onopen</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">[GeminiLive] SDK Connected</span><span class="dl">'</span><span class="p">),</span>
    <span class="na">onmessage</span><span class="p">:</span> <span class="p">(</span><span class="nx">msg</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nf">handleGeminiResponse</span><span class="p">(</span><span class="nx">msg</span><span class="p">),</span>
    <span class="na">onclose</span><span class="p">:</span> <span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">[GeminiLive] Session closed</span><span class="dl">'</span><span class="p">),</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>How it Works:</strong></p>

<ol>
<li>
<strong>Bidirectional Audio</strong>: Streams user microphone input at 16kHz PCM</li>
<li>
<strong>Real-Time Transcription</strong>: Captures user speech as text for the playbook</li>
<li>
<strong>Clarification Questions</strong>: Gemini asks intelligent questions when steps are ambiguous</li>
<li>
<strong>Voice Responses</strong>: Uses the "Puck" voice for calm, operational responses</li>
<li>
<strong>Context Integration</strong>: Receives real-time updates when vision detects new steps</li>
</ol>

<p>The implementation handles:</p>

<ul>
<li>PCM audio conversion (Float32 to Int16 base64)</li>
<li>Transcript merging for streaming text</li>
<li>Audio playback queue management</li>
<li>Graceful session cleanup</li>
</ul>

<h3>
  
  
  3. Gemini Compile: From Raw Events to Semantic Playbooks
</h3>

<p>After capturing a workflow, raw events need to be transformed into structured, reusable playbook steps. <strong>Gemini Compile</strong> handles this intelligent transformation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># apps/api/app/services/gemini_compile.py
</span>
<span class="n">COMPILE_PROMPT</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">You are an expert workflow analyst. Given the following raw browser
interaction events recorded during a user session, produce a structured list of semantic
playbook steps.

For each step, output:
- title: A clear, human-readable description
- step_type: One of navigate, click, input, submit, verify, wait, action
- target_url: The URL involved, if applicable
- selector: CSS selector or element identifier
- variables: Dict of variable_name -&gt; template_string for parameterized values
- guardrails: Dict with </span><span class="sh">"</span><span class="s">verify</span><span class="sh">"</span><span class="s"> key describing expected state after this step

Important rules:
1. Detect variables automatically — anything that looks like personal data should become {{variable}}
2. Merge consecutive related events into single logical steps
3. Add verification guardrails after form submissions and page navigations
4. Use voice_note and gemini_clarification entries as context for step titles and guardrails
</span><span class="sh">"""</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">compile_events</span><span class="p">(</span><span class="n">raw_events</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">]:</span>
    <span class="sh">"""</span><span class="s">Call Gemini to compile raw capture events into semantic playbook steps.</span><span class="sh">"""</span>

    <span class="n">client</span> <span class="o">=</span> <span class="n">genai</span><span class="p">.</span><span class="nc">Client</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="n">google_api_key</span><span class="p">)</span>
    <span class="n">events_json</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">raw_events</span><span class="p">,</span> <span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span>
    <span class="n">prompt</span> <span class="o">=</span> <span class="n">COMPILE_PROMPT</span><span class="p">.</span><span class="nf">format</span><span class="p">(</span><span class="n">events_json</span><span class="o">=</span><span class="n">events_json</span><span class="p">)</span>

    <span class="n">response</span> <span class="o">=</span> <span class="k">await</span> <span class="n">client</span><span class="p">.</span><span class="n">aio</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>
        <span class="n">model</span><span class="o">=</span><span class="n">settings</span><span class="p">.</span><span class="n">gemini_model</span><span class="p">,</span>
        <span class="n">contents</span><span class="o">=</span><span class="n">prompt</span><span class="p">,</span>
        <span class="n">config</span><span class="o">=</span><span class="n">types</span><span class="p">.</span><span class="nc">GenerateContentConfig</span><span class="p">(</span>
            <span class="n">response_mime_type</span><span class="o">=</span><span class="sh">'</span><span class="s">application/json</span><span class="sh">'</span><span class="p">,</span>
        <span class="p">),</span>
    <span class="p">)</span>

    <span class="n">steps</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">text</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">normalize_steps</span><span class="p">(</span><span class="n">steps</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Intelligent Features:</strong></p>

<ul>
<li>
<strong>Variable Detection</strong>: Automatically identifies values that should be parameterized (names, emails, IDs)</li>
<li>
<strong>Step Merging</strong>: Combines related actions into logical business steps</li>
<li>
<strong>Guardrail Generation</strong>: Adds verification steps after critical actions</li>
<li>
<strong>Voice Context Integration</strong>: Uses spoken context to clarify ambiguous actions</li>
<li>
<strong>Business-Level Abstraction</strong>: Transforms low-level events into meaningful workflow steps</li>
</ul>

<h3>
  
  
  4. Execution Engine with Fallback to AI Agents
</h3>

<p>For playbook execution, Memoo uses Playwright for deterministic actions but falls back to an autonomous agent (Stagehand) when selectors fail:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># apps/api/app/services/playwright_executor.py
</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">execute_playbook_steps</span><span class="p">(</span>
    <span class="n">steps</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">],</span>
    <span class="n">row_data</span><span class="p">:</span> <span class="nb">dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span>
    <span class="o">*</span><span class="p">,</span>
    <span class="n">headless</span><span class="p">:</span> <span class="nb">bool</span> <span class="o">=</span> <span class="bp">True</span><span class="p">,</span>
    <span class="n">screenshot</span><span class="p">:</span> <span class="nb">bool</span> <span class="o">=</span> <span class="bp">True</span><span class="p">,</span>
    <span class="n">step_callback</span><span class="p">:</span> <span class="n">Any</span> <span class="o">|</span> <span class="bp">None</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
<span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">]:</span>
    <span class="sh">"""</span><span class="s">Execute a list of playbook steps via Playwright and return per-step results.</span><span class="sh">"""</span>

    <span class="k">async</span> <span class="k">with</span> <span class="nf">async_playwright</span><span class="p">()</span> <span class="k">as</span> <span class="n">pw</span><span class="p">:</span>
        <span class="n">browser_server</span> <span class="o">=</span> <span class="k">await</span> <span class="n">pw</span><span class="p">.</span><span class="n">chromium</span><span class="p">.</span><span class="nf">launch_server</span><span class="p">(</span>
            <span class="n">headless</span><span class="o">=</span><span class="n">headless</span><span class="p">,</span>
            <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">'</span><span class="s">--remote-debugging-port=0</span><span class="sh">'</span><span class="p">]</span>
        <span class="p">)</span>

        <span class="k">for</span> <span class="n">step</span> <span class="ow">in</span> <span class="n">steps</span><span class="p">:</span>
            <span class="k">try</span><span class="p">:</span>
                <span class="c1"># Try deterministic Playwright action
</span>                <span class="k">if</span> <span class="n">step_type</span> <span class="o">==</span> <span class="sh">'</span><span class="s">navigate</span><span class="sh">'</span><span class="p">:</span>
                    <span class="k">await</span> <span class="n">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="n">target_url</span><span class="p">)</span>
                <span class="k">elif</span> <span class="n">step_type</span> <span class="o">==</span> <span class="sh">'</span><span class="s">click</span><span class="sh">'</span><span class="p">:</span>
                    <span class="k">await</span> <span class="n">page</span><span class="p">.</span><span class="nf">click</span><span class="p">(</span><span class="n">selector</span><span class="p">)</span>
                <span class="c1"># ... other step types
</span>
            <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">action_error</span><span class="p">:</span>
                <span class="c1"># Fall back to autonomous Stagehand agent
</span>                <span class="n">task</span> <span class="o">=</span> <span class="nf">_build_autonomous_task</span><span class="p">(</span>
                    <span class="n">title</span><span class="o">=</span><span class="n">title</span><span class="p">,</span>
                    <span class="n">expected</span><span class="o">=</span><span class="n">expected</span><span class="p">,</span>
                    <span class="n">target_url</span><span class="o">=</span><span class="n">target_url</span><span class="p">,</span>
                    <span class="n">selector</span><span class="o">=</span><span class="n">selector</span><span class="p">,</span>
                    <span class="n">resolved_values</span><span class="o">=</span><span class="n">resolved_values</span><span class="p">,</span>
                <span class="p">)</span>
                <span class="n">agent_result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">_run_stagehand_step</span><span class="p">(</span>
                    <span class="n">settings</span><span class="p">,</span>
                    <span class="n">ws_endpoint</span><span class="o">=</span><span class="n">ws_endpoint</span><span class="p">,</span>
                    <span class="n">task</span><span class="o">=</span><span class="n">task</span><span class="p">,</span>
                <span class="p">)</span>
</code></pre>

</div>



<p>This hybrid approach ensures reliability:</p>

<ul>
<li>
<strong>Deterministic Path</strong>: Fast, exact execution for known selectors</li>
<li>
<strong>Autonomous Fallback</strong>: AI-powered execution when selectors fail</li>
<li>
<strong>Evidence Capture</strong>: Screenshots after every step for verification</li>
<li>
<strong>Real-Time Updates</strong>: Callbacks notify the UI of progress</li>
</ul>

<h2>
  
  
  Google Cloud Infrastructure
</h2>

<h3>
  
  
  Deployment Architecture
</h3>

<p>Memoo is entirely cloud-native on Google Cloud:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># infra/terraform/gcp/cloud_run.tf</span>

<span class="c1"># API Service with Cloud SQL connection</span>
<span class="nx">resource</span> <span class="s2">"google_cloud_run_v2_service"</span> <span class="s2">"api"</span> <span class="p">{</span>
  <span class="nx">name</span>     <span class="p">=</span> <span class="s2">"memoo-api"</span>
  <span class="nx">location</span> <span class="p">=</span> <span class="s2">"europe-west1"</span>

  <span class="nx">template</span> <span class="p">{</span>
    <span class="nx">service_account</span> <span class="p">=</span> <span class="nx">google_service_account</span><span class="p">.</span><span class="nx">api</span><span class="p">.</span><span class="nx">email</span>

    <span class="nx">vpc_access</span> <span class="p">{</span>
      <span class="nx">connector</span> <span class="p">=</span> <span class="nx">google_vpc_access_connector</span><span class="p">.</span><span class="nx">serverless</span><span class="p">.</span><span class="nx">id</span>
      <span class="nx">egress</span>    <span class="p">=</span> <span class="s2">"PRIVATE_RANGES_ONLY"</span>
    <span class="p">}</span>

    <span class="nx">volumes</span> <span class="p">{</span>
      <span class="nx">name</span> <span class="p">=</span> <span class="s2">"cloudsql"</span>
      <span class="nx">cloud_sql_instance</span> <span class="p">{</span>
        <span class="nx">instances</span> <span class="p">=</span> <span class="p">[</span><span class="nx">google_sql_database_instance</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">connection_name</span><span class="p">]</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="nx">containers</span> <span class="p">{</span>
      <span class="nx">env</span> <span class="p">{</span>
        <span class="nx">name</span> <span class="p">=</span> <span class="s2">"GOOGLE_API_KEY"</span>
        <span class="nx">value_source</span> <span class="p">{</span>
          <span class="nx">secret_key_ref</span> <span class="p">{</span>
            <span class="nx">secret</span>  <span class="p">=</span> <span class="nx">google_secret_manager_secret</span><span class="p">.</span><span class="nx">google_api_key</span><span class="p">.</span><span class="nx">secret_id</span>
            <span class="nx">version</span> <span class="p">=</span> <span class="s2">"latest"</span>
          <span class="p">}</span>
        <span class="p">}</span>
      <span class="p">}</span>

      <span class="nx">env</span> <span class="p">{</span>
        <span class="nx">name</span>  <span class="p">=</span> <span class="s2">"GEMINI_MODEL"</span>
        <span class="nx">value</span> <span class="p">=</span> <span class="s2">"gemini-2.0-flash-exp"</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Web Frontend</span>
<span class="nx">resource</span> <span class="s2">"google_cloud_run_v2_service"</span> <span class="s2">"web"</span> <span class="p">{</span>
  <span class="nx">name</span>     <span class="p">=</span> <span class="s2">"memoo-web"</span>
  <span class="nx">location</span> <span class="p">=</span> <span class="s2">"europe-west1"</span>

  <span class="nx">template</span> <span class="p">{</span>
    <span class="nx">containers</span> <span class="p">{</span>
      <span class="nx">env</span> <span class="p">{</span>
        <span class="nx">name</span>  <span class="p">=</span> <span class="s2">"NEXT_PUBLIC_GEMINI_LIVE_MODEL"</span>
        <span class="nx">value</span> <span class="p">=</span> <span class="s2">"gemini-2.0-flash-live-exp"</span>
      <span class="p">}</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Infrastructure Components:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Service</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Cloud Run</strong></td>
<td>Serverless containers for web and API (auto-scales to zero)</td>
</tr>
<tr>
<td><strong>Cloud SQL</strong></td>
<td>Managed PostgreSQL database for playbooks and runs</td>
</tr>
<tr>
<td><strong>Cloud Storage</strong></td>
<td>Evidence screenshots and artifacts</td>
</tr>
<tr>
<td><strong>Compute Engine</strong></td>
<td>Visible Chromium sandbox with noVNC for live playback</td>
</tr>
<tr>
<td><strong>Secret Manager</strong></td>
<td>Secure storage for API keys and credentials</td>
</tr>
<tr>
<td><strong>Artifact Registry</strong></td>
<td>Container image storage</td>
</tr>
<tr>
<td><strong>Cloud Build</strong></td>
<td>CI/CD pipeline for automated deployments</td>
</tr>
<tr>
<td><strong>VPC Connector</strong></td>
<td>Private connectivity between Cloud Run and Cloud SQL</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Deployment Scripts
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># scripts/gcp/deploy.sh</span>

<span class="nb">export </span><span class="nv">PROJECT_ID</span><span class="o">=</span><span class="s2">"your-project"</span>
<span class="nb">export </span><span class="nv">REGION</span><span class="o">=</span><span class="s2">"europe-west1"</span>

<span class="c"># Bootstrap Terraform state</span>
./scripts/gcp/bootstrap_tf_state.sh

<span class="c"># Deploy infrastructure</span>
<span class="nb">cd </span>infra/terraform/gcp
terraform init
terraform apply

<span class="c"># Build and push images</span>
gcloud builds submit <span class="nt">--tag</span> gcr.io/<span class="nv">$PROJECT_ID</span>/memoo-api
gcloud builds submit <span class="nt">--tag</span> gcr.io/<span class="nv">$PROJECT_ID</span>/memoo-web
</code></pre>

</div>



<h2>
  
  
  Key Features
</h2>

<h3>
  
  
  1. Teach Mode: Watch and Learn
</h3>

<ol>
<li>User shares screen via browser Screen Capture API</li>
<li>Frontend captures frames every 2-3 seconds</li>
<li>Backend sends frames to Gemini Vision</li>
<li>Gemini detects actions with grounding metadata</li>
<li>Navigator Live asks clarifying questions via voice</li>
<li>User speaks additional context</li>
<li>All events compiled into structured playbook</li>
</ol>

<h3>
  
  
  2. Run Mode: Execute with Evidence
</h3>

<ol>
<li>Playbook selected for execution</li>
<li>Row data provides variable values</li>
<li>Playwright executes deterministic steps</li>
<li>Stagehand agent handles ambiguous cases</li>
<li>Screenshot captured after each step</li>
<li>Evidence stored in Cloud Storage</li>
<li>Real-time progress updates via WebSocket</li>
</ol>

<h3>
  
  
  3. Visual Sandbox: Live Playback
</h3>

<p>For transparent execution, Memoo provides a visible Chromium sandbox:</p>

<ul>
<li>Deployed on Compute Engine with noVNC</li>
<li>CDP (Chrome DevTools Protocol) for programmatic control</li>
<li>Users can watch their playbooks execute in real-time</li>
<li>Perfect for debugging and demonstration</li>
</ul>

<h2>
  
  
  Technical Challenges &amp; Solutions
</h2>

<h3>
  
  
  Challenge 1: Real-Time Audio Streaming
</h3>

<p><strong>Problem:</strong> Gemini Live requires PCM audio at specific sample rates, but browser AudioContext uses Float32.</p>

<p><strong>Solution:</strong> Implemented custom PCM conversion:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">floatTo16BitB64</span><span class="p">(</span><span class="nx">samples</span><span class="p">:</span> <span class="nb">Float32Array</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">buf</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ArrayBuffer</span><span class="p">(</span><span class="nx">samples</span><span class="p">.</span><span class="nx">length</span> <span class="o">*</span> <span class="mi">2</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">view</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">DataView</span><span class="p">(</span><span class="nx">buf</span><span class="p">);</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">samples</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">s</span> <span class="o">=</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">max</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">,</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="nx">samples</span><span class="p">[</span><span class="nx">i</span><span class="p">]));</span>
    <span class="nx">view</span><span class="p">.</span><span class="nf">setInt16</span><span class="p">(</span><span class="nx">i</span> <span class="o">*</span> <span class="mi">2</span><span class="p">,</span> <span class="nx">s</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="nx">s</span> <span class="o">*</span> <span class="mh">0x8000</span> <span class="p">:</span> <span class="nx">s</span> <span class="o">*</span> <span class="mh">0x7fff</span><span class="p">,</span> <span class="kc">true</span><span class="p">);</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="nf">btoa</span><span class="p">(</span><span class="nx">bin</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 2: Transcript Merging
</h3>

<p><strong>Problem:</strong> Streaming transcription sends partial updates, full phrases, or deltas inconsistently.</p>

<p><strong>Solution:</strong> Implemented intelligent merge logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">mergeTranscriptProgress</span><span class="p">(</span><span class="nx">current</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">chunk</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">chunk</span> <span class="o">===</span> <span class="nx">current</span><span class="p">)</span> <span class="k">return</span> <span class="nx">current</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">chunk</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="nx">current</span><span class="p">))</span> <span class="k">return</span> <span class="nx">chunk</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">current</span><span class="p">.</span><span class="nf">startsWith</span><span class="p">(</span><span class="nx">chunk</span><span class="p">))</span> <span class="k">return</span> <span class="nx">current</span><span class="p">;</span>
  <span class="k">return</span> <span class="nf">appendTranscriptText</span><span class="p">(</span><span class="nx">current</span><span class="p">,</span> <span class="nx">chunk</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Challenge 3: Variable Detection
</h3>

<p><strong>Problem:</strong> How to distinguish between constants (e.g., "United States") and variables (e.g., "John Doe")?</p>

<p><strong>Solution:</strong> Gemini analyzes patterns and context:</p>

<ul>
<li>Personal names → <code>{{first_name}}</code>, <code>{{last_name}}</code>
</li>
<li>Email addresses → <code>{{email}}</code>
</li>
<li>Phone numbers → <code>{{phone}}</code>
</li>
<li>IDs and codes → <code>{{employee_id}}</code>
</li>
<li>Common options remain constant</li>
</ul>

<h3>
  
  
  Challenge 4: Selector Fragility
</h3>

<p><strong>Problem:</strong> CSS selectors break when websites change.</p>

<p><strong>Solution:</strong> Hybrid approach:</p>

<ol>
<li>Prefer descriptive selectors over raw CSS</li>
<li>Store human-readable descriptions as backup</li>
<li>Fall back to autonomous agent when selectors fail</li>
<li>Use Gemini's vision understanding to find elements</li>
</ol>

<h2>
  
  
  Performance &amp; Optimization
</h2>

<h3>
  
  
  Response Times
</h3>

<ul>
<li>
<strong>Vision Analysis</strong>: ~500-800ms per frame</li>
<li>
<strong>Voice Latency</strong>: ~300-500ms end-to-end</li>
<li>
<strong>Compile Time</strong>: ~2-3 seconds for 20-step workflow</li>
<li>
<strong>Step Execution</strong>: ~200-500ms per step (deterministic)</li>
</ul>

<h3>
  
  
  Cost Optimization
</h3>

<ul>
<li>
<strong>Cloud Run</strong>: Auto-scales to zero when idle</li>
<li>
<strong>Gemini Flash</strong>: Optimized for speed and cost</li>
<li>
<strong>Image Compression</strong>: JPEG at 80% quality before sending to API</li>
<li>
<strong>Frame Rate</strong>: Adaptive (2-3 seconds) based on activity</li>
<li>
<strong>Caching</strong>: Compiled playbooks stored in Cloud SQL</li>
</ul>

<h2>
  
  
  Future Enhancements
</h2>

<ol>
<li>
<strong>Team Workflows</strong>: Multi-user collaboration on playbooks</li>
<li>
<strong>Vault Integration</strong>: Secure credential storage</li>
<li>
<strong>Scheduled Runs</strong>: Cron-based execution</li>
<li>
<strong>Webhook Triggers</strong>: Event-based automation</li>
<li>
<strong>Advanced Retry</strong>: Smart error recovery</li>
<li>
<strong>Analytics</strong>: Execution insights and optimization</li>
</ol>

<h2>
  
  
  Conclusion
</h2>

<p>Memoo demonstrates the power of combining <strong>Google's multimodal AI</strong> with <strong>cloud-native infrastructure</strong> to solve real-world automation problems. By leveraging:</p>

<ul>
<li>
<strong>Gemini Vision</strong> for screen understanding</li>
<li>
<strong>Gemini Live</strong> for natural voice interaction</li>
<li>
<strong>Gemini Intelligence</strong> for semantic compilation</li>
<li>
<strong>Google Cloud</strong> for scalable, reliable deployment</li>
</ul>

<p>We've created a tool that makes browser automation accessible, reliable, and transparent.</p>

<p>The future of automation isn't code—it's conversation. And with Gemini and Google Cloud, that future is here.</p>




<p><strong>Built for the #GeminiLiveAgentChallenge</strong></p>

<p><em>This blog post was created for the purposes of entering the Gemini Live Agent Challenge hackathon. The Memoo project demonstrates innovative use of Google Gemini models (Gemini 2.0 Flash, Gemini Live API) and Google Cloud services to create a multimodal AI-powered browser navigator.</em></p>

<p><strong>Tech Stack:</strong></p>

<ul>
<li>Frontend: Next.js 15, React 19, Tailwind CSS</li>
<li>Backend: FastAPI, Python 3.12</li>
<li>AI: Google Gemini 2.0 Flash, Gemini Live API</li>
<li>Infrastructure: Google Cloud Run, Cloud SQL, Cloud Storage, Compute Engine</li>
<li>Automation: Playwright, Stagehand Agent</li>
<li>Video: Remotion for demo generation</li>
</ul>

<p><strong>Repository:</strong> <a href="https://github.com/migarci2/memoo" rel="noopener noreferrer">github.com/migarci2/memoo</a></p>

