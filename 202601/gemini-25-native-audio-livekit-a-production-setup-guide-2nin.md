---
Title: Gemini 2.5 Native Audio + LiveKit: A Production Setup Guide
Description: 
Author: Behram
Date: 2026-01-16T21:55:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>10-minute tutorials make voice AI look simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">getToken</span><span class="p">(</span><span class="nx">roomName</span><span class="p">);</span>
<span class="k">await</span> <span class="nx">room</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="nx">url</span><span class="p">,</span> <span class="nx">token</span><span class="p">);</span>
<span class="c1">// Done! ✨</span>
</code></pre>

</div>



<p>Production reality is different:</p>

<ul>
<li>Who creates the room?</li>
<li>How does the agent identify users?</li>
<li>What happens when failures occur?</li>
<li>How do you prevent duplicate agents?</li>
</ul>

<p>After shipping <strong>1000+ voice AI sessions</strong> using <strong>LiveKit + Gemini Realtime</strong> on Next.js/Cloud Run, here's what tutorials skip.</p>

<p><strong>This article covers:</strong> JWT auth, auto-dispatch patterns, audio subscription timing, greeting guards, and production robustness.</p>

<p><strong>Prerequisites:</strong> Basic Next.js, LiveKit Cloud account, Firebase Auth configured.</p>




<h2>
  
  
  The Complete Flow (11 Steps)
</h2>

<p>Before diving into code, understand the full sequence:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────┐
│   Browser   │  1. User clicks "Start Interview"
└──────┬──────┘
       │ 2. POST /api/start-bot
       │    Authorization: Bearer &lt;firebase-token&gt;
       ▼
┌─────────────────┐
│  Next.js API    │  3. Verify token with Firebase
│ /api/start-bot  │  4. Create LiveKit room
└────────┬────────┘  5. Generate JWT + metadata
         │           6. Return token to browser
         ▼
┌───────────────────────┐
│   LiveKit Cloud       │  7. Browser connects
│ wss://your.livekit... │  8. Auto-dispatch agent
└───────────────────────┘
         │
         ▼
┌───────────────────────┐
│  Cloud Run Worker     │  9. entrypoint() runs
│  (Voice Agent)        │  10. Load user state from DB
└───────────────────────┘  11. Generate personalized greeting
</code></pre>

</div>



<p>This isn't just "connect to a room"—it's an <strong>orchestrated sequence</strong> where order matters.</p>

<p>Let's break down each part with the mistakes I made.</p>




<h2>
  
  
  Part 1: The JWT Token Factory (Backend API)
</h2>

<h3>
  
  
  ❌ Mistake #1: Exposing Credentials to Browser
</h3>

<p>I've seen this in production codebases:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// NEVER DO THIS</span>
<span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AccessToken</span><span class="p">(</span>
  <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NEXT_PUBLIC_LIVEKIT_API_KEY</span><span class="p">,</span>  <span class="c1">// ❌ EXPOSED IN BROWSER</span>
  <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NEXT_PUBLIC_LIVEKIT_SECRET</span><span class="p">,</span>   <span class="c1">// ❌ SECURITY DISASTER</span>
<span class="p">);</span>
</code></pre>

</div>



<p><strong>Why it's terrible:</strong></p>

<ul>
<li>Anyone can create tokens for ANY room</li>
<li>Attackers can impersonate users</li>
<li>Zero authentication</li>
</ul>

<h3>
  
  
  ✅ The Right Way: Server-Side Token Generation
</h3>

<p>Create a Next.js API route that verifies the user BEFORE creating tokens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// frontend/src/app/api/start-bot/route.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">NextRequest</span><span class="p">,</span> <span class="nx">NextResponse</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">next/server</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">RoomServiceClient</span><span class="p">,</span> <span class="nx">AccessToken</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">livekit-server-sdk</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">adminAuth</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@/lib/firebase-admin</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">POST</span><span class="p">(</span><span class="nx">req</span><span class="p">:</span> <span class="nx">NextRequest</span><span class="p">)</span> <span class="p">{</span>
  <span class="c1">// STEP 1: Verify Firebase token (The Gatekeeper)</span>
  <span class="kd">const</span> <span class="nx">authHeader</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">Authorization</span><span class="dl">'</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">authHeader</span><span class="p">?.</span><span class="nf">startsWith</span><span class="p">(</span><span class="dl">'</span><span class="s1">Bearer </span><span class="dl">'</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">NextResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Unauthorized</span><span class="dl">'</span> <span class="p">},</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">401</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">idToken</span> <span class="o">=</span> <span class="nx">authHeader</span><span class="p">.</span><span class="nf">split</span><span class="p">(</span><span class="dl">'</span><span class="s1">Bearer </span><span class="dl">'</span><span class="p">)[</span><span class="mi">1</span><span class="p">];</span>

  <span class="kd">let</span> <span class="nx">user_id</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="c1">// Server-side verification with Firebase Admin SDK</span>
    <span class="kd">const</span> <span class="nx">decodedToken</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">adminAuth</span><span class="p">.</span><span class="nf">verifyIdToken</span><span class="p">(</span><span class="nx">idToken</span><span class="p">);</span>
    <span class="nx">user_id</span> <span class="o">=</span> <span class="nx">decodedToken</span><span class="p">.</span><span class="nx">uid</span><span class="p">;</span>  <span class="c1">// ⭐ REAL verified user ID</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">authError</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">NextResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Invalid Token</span><span class="dl">'</span> <span class="p">},</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">401</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// STEP 2: LiveKit setup</span>
  <span class="kd">const</span> <span class="nx">livekitUrl</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LIVEKIT_URL</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">apiKey</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LIVEKIT_API_KEY</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">apiSecret</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">LIVEKIT_API_SECRET</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">roomService</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomServiceClient</span><span class="p">(</span><span class="nx">livekitUrl</span><span class="p">,</span> <span class="nx">apiKey</span><span class="p">,</span> <span class="nx">apiSecret</span><span class="p">);</span>

  <span class="c1">// STEP 3: Create room</span>
  <span class="kd">const</span> <span class="nx">room_name</span> <span class="o">=</span> <span class="s2">`room-</span><span class="p">${</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">().</span><span class="nf">toString</span><span class="p">(</span><span class="mi">36</span><span class="p">).</span><span class="nf">substring</span><span class="p">(</span><span class="mi">7</span><span class="p">)}</span><span class="s2">`</span><span class="p">;</span>

  <span class="k">await</span> <span class="nx">roomService</span><span class="p">.</span><span class="nf">createRoom</span><span class="p">({</span>
    <span class="na">name</span><span class="p">:</span> <span class="nx">room_name</span><span class="p">,</span>
    <span class="na">emptyTimeout</span><span class="p">:</span> <span class="mi">60</span><span class="p">,</span>      <span class="c1">// Auto-cleanup after 60s</span>
    <span class="na">maxParticipants</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span>    <span class="c1">// 1 user + 1 agent only</span>
  <span class="p">});</span>

  <span class="c1">// STEP 4: Generate token with verified identity</span>
  <span class="kd">const</span> <span class="nx">at</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AccessToken</span><span class="p">(</span><span class="nx">apiKey</span><span class="p">,</span> <span class="nx">apiSecret</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">identity</span><span class="p">:</span> <span class="nx">user_id</span><span class="p">,</span>  <span class="c1">// ⭐ Embedded in token</span>
  <span class="p">});</span>

  <span class="nx">at</span><span class="p">.</span><span class="nf">addGrant</span><span class="p">({</span>
    <span class="na">roomJoin</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">room</span><span class="p">:</span> <span class="nx">room_name</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">at</span><span class="p">.</span><span class="nf">toJwt</span><span class="p">();</span>

  <span class="k">return</span> <span class="nx">NextResponse</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
    <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="nx">token</span><span class="p">,</span>
    <span class="nx">room_name</span><span class="p">,</span>
    <span class="na">identity</span><span class="p">:</span> <span class="nx">user_id</span>
  <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Key Decisions:</strong></p>

<p>✅ <strong>Server generates tokens</strong> (credentials never touch browser)<br><br>
✅ <strong>Firebase verifies user</strong> BEFORE creating room<br><br>
✅ <strong>User ID embedded</strong> as <code>identity</code> in LiveKit token<br><br>
✅ <strong>Random room names</strong> (users can't guess existing rooms)</p>


<h2>
  
  
  Part 2: Explicit vs Auto-Dispatch
</h2>

<p>Now that you have a room and token, how does the agent join?</p>
<h3>
  
  
  Pattern A: Manual Dispatch (What I Used First)
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Create token</span>
<span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">at</span><span class="p">.</span><span class="nf">toJwt</span><span class="p">();</span>

<span class="c1">// Separately dispatch agent</span>
<span class="k">await</span> <span class="nx">roomService</span><span class="p">.</span><span class="nf">dispatchAgent</span><span class="p">({</span>
  <span class="na">agentName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">my-agent</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">room</span><span class="p">:</span> <span class="nx">room_name</span><span class="p">,</span>
<span class="p">});</span>

<span class="k">return</span> <span class="p">{</span> <span class="nx">token</span> <span class="p">};</span>
</code></pre>

</div>


<p><strong>Problems I Hit:</strong></p>

<ul>
<li>Requires 2 API calls to LiveKit</li>
<li>Race condition: User joins before agent</li>
<li>If dispatch fails, user sits in empty room</li>
</ul>
<h3>
  
  
  Pattern B: Auto-Dispatch via JWT ⭐ (Production Choice)
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">RoomConfiguration</span><span class="p">,</span> <span class="nx">RoomAgentDispatch</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@livekit/protocol</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Attach dispatch config to user's token</span>
<span class="nx">at</span><span class="p">.</span><span class="nx">roomConfig</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">RoomConfiguration</span><span class="p">({</span>
  <span class="na">agents</span><span class="p">:</span> <span class="p">[</span>
    <span class="k">new</span> <span class="nc">RoomAgentDispatch</span><span class="p">({</span>
      <span class="na">agentName</span><span class="p">:</span> <span class="dl">'</span><span class="s1">noah-voice-agent</span><span class="dl">'</span><span class="p">,</span>
    <span class="p">}),</span>
  <span class="p">],</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">at</span><span class="p">.</span><span class="nf">toJwt</span><span class="p">();</span>
</code></pre>

</div>


<p><strong>Why it's better:</strong></p>

<p>✅ <strong>Atomic</strong>: Agent dispatch happens when user joins<br><br>
✅ <strong>LiveKit handles retries</strong> (more reliable)<br><br>
✅ <strong>One API call</strong> instead of two<br><br>
✅ <strong>No race conditions</strong></p>

<p><strong>Real Impact:</strong> Reduced agent dispatch failures from ~5% to &lt;0.1%.</p>


<h2>
  
  
  Part 3: Agent Entry Point - The Critical Pattern
</h2>

<p>Your agent receives a job when the user joins. Here's where most tutorials fail you.</p>
<h3>
  
  
  ❌ Mistake #2: Waiting for Participant Before Starting Session
</h3>

<p>This looks logical but breaks in production:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">async</span> <span class="k">def</span> <span class="nf">entrypoint</span><span class="p">(</span><span class="n">ctx</span><span class="p">:</span> <span class="n">JobContext</span><span class="p">):</span>
    <span class="k">await</span> <span class="n">ctx</span><span class="p">.</span><span class="nf">connect</span><span class="p">()</span>

    <span class="c1"># Wait for user to appear
</span>    <span class="n">participant</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">wait_for_participant</span><span class="p">(</span><span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">)</span>
    <span class="n">user_id</span> <span class="o">=</span> <span class="n">participant</span><span class="p">.</span><span class="n">identity</span>

    <span class="c1"># Load their data from database
</span>    <span class="n">data</span> <span class="o">=</span> <span class="k">await</span> <span class="n">db</span><span class="p">.</span><span class="nf">load_user_state</span><span class="p">(</span><span class="n">user_id</span><span class="p">)</span>
    <span class="n">agent</span> <span class="o">=</span> <span class="nc">MyAgent</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span>

    <span class="c1"># Start session
</span>    <span class="n">session</span> <span class="o">=</span> <span class="nc">AgentSession</span><span class="p">(</span><span class="n">llm</span><span class="o">=</span><span class="p">...)</span>
    <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">start</span><span class="p">(</span><span class="n">room</span><span class="o">=</span><span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">,</span> <span class="n">agent</span><span class="o">=</span><span class="n">agent</span><span class="p">)</span>  <span class="c1"># ❌ TOO LATE!
</span></code></pre>

</div>



<p><strong>Symptoms:</strong></p>

<ul>
<li>Agent connects but can't hear user</li>
<li>Logs show: <code>subscribed=False</code>
</li>
<li>Works in local testing (lucky timing)</li>
<li>Fails in production randomly</li>
</ul>

<p><strong>Root Cause:</strong> Audio subscription happens <strong>inside</strong> <code>session.start()</code>. If you wait for participant identity first, you miss the subscription window.</p>

<h3>
  
  
  ✅ The Fix: Start Session First, Then Personalize
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">async</span> <span class="k">def</span> <span class="nf">entrypoint</span><span class="p">(</span><span class="n">ctx</span><span class="p">:</span> <span class="n">JobContext</span><span class="p">):</span>
    <span class="c1"># STEP 1: Create placeholder agent
</span>    <span class="n">placeholder_db</span> <span class="o">=</span> <span class="nc">DatabaseService</span><span class="p">(</span><span class="sh">"</span><span class="s">pending</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">agent</span> <span class="o">=</span> <span class="nc">InterviewAgent</span><span class="p">(</span><span class="sh">"</span><span class="s">pending</span><span class="sh">"</span><span class="p">,</span> <span class="n">placeholder_db</span><span class="p">,</span> <span class="bp">None</span><span class="p">)</span>

    <span class="c1"># STEP 2: Create session
</span>    <span class="n">session</span> <span class="o">=</span> <span class="nc">AgentSession</span><span class="p">(</span>
        <span class="n">llm</span><span class="o">=</span><span class="n">google</span><span class="p">.</span><span class="n">realtime</span><span class="p">.</span><span class="nc">RealtimeModel</span><span class="p">(</span>
            <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gemini-live-2.5-flash-native-audio</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">voice</span><span class="o">=</span><span class="sh">"</span><span class="s">Puck</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">instructions</span><span class="o">=</span><span class="n">agent</span><span class="p">.</span><span class="n">instructions</span><span class="p">,</span>
            <span class="n">vertexai</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
        <span class="p">),</span>
    <span class="p">)</span>

    <span class="c1"># STEP 3: Start session IMMEDIATELY (SDK subscribes to audio here)
</span>    <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">start</span><span class="p">(</span><span class="n">room</span><span class="o">=</span><span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">,</span> <span class="n">agent</span><span class="o">=</span><span class="n">agent</span><span class="p">)</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ Session started - audio pipeline active</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># STEP 4: NOW get participant (session already listening)
</span>    <span class="n">participant</span> <span class="o">=</span> <span class="bp">None</span>
    <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">30</span><span class="p">):</span>  <span class="c1"># 30s timeout
</span>        <span class="k">if</span> <span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="n">remote_participants</span><span class="p">:</span>
            <span class="n">participant</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="n">remote_participants</span><span class="p">.</span><span class="nf">values</span><span class="p">())[</span><span class="mi">0</span><span class="p">]</span>
            <span class="k">break</span>
        <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>

    <span class="k">if</span> <span class="ow">not</span> <span class="n">participant</span><span class="p">:</span>
        <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sh">"</span><span class="s">⚠️ No participant after 30s</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span>

    <span class="n">user_id</span> <span class="o">=</span> <span class="n">participant</span><span class="p">.</span><span class="n">identity</span>  <span class="c1"># ⭐ This is Firebase UID from token
</span>    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">✅ User identity: </span><span class="si">{</span><span class="n">user_id</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># STEP 5: Hydrate agent with real data
</span>    <span class="n">agent</span><span class="p">.</span><span class="n">user_id</span> <span class="o">=</span> <span class="n">user_id</span>
    <span class="n">agent</span><span class="p">.</span><span class="n">db_service</span> <span class="o">=</span> <span class="nc">DatabaseService</span><span class="p">(</span><span class="n">user_id</span><span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">initial_data</span> <span class="o">=</span> <span class="k">await</span> <span class="n">agent</span><span class="p">.</span><span class="n">db_service</span><span class="p">.</span><span class="nf">get_candidate_data</span><span class="p">()</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">❌ DB failed: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">initial_data</span> <span class="o">=</span> <span class="p">{}</span>  <span class="c1"># Fallback to fresh session
</span>
    <span class="n">agent</span><span class="p">.</span><span class="n">initial_data</span> <span class="o">=</span> <span class="n">initial_data</span>

    <span class="c1"># STEP 6: Determine phase (new vs resume)
</span>    <span class="n">agent</span><span class="p">.</span><span class="n">current_phase</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">_determine_initial_phase</span><span class="p">(</span><span class="n">initial_data</span><span class="p">)</span>

    <span class="c1"># STEP 7: Generate personalized greeting
</span>    <span class="n">greeting</span> <span class="o">=</span> <span class="n">agent</span><span class="p">.</span><span class="nf">get_greeting_instruction</span><span class="p">()</span>
    <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">generate_reply</span><span class="p">(</span><span class="n">instructions</span><span class="o">=</span><span class="n">greeting</span><span class="p">)</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ Greeting triggered</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>The Pattern:</strong> Start → Listen → Identify → Personalize → Greet</p>

<p><strong>Impact:</strong> 100% audio subscription success rate.</p>




<h2>
  
  
  Part 4: The Greeting Guard Pattern
</h2>

<p>Even with audio working, there's another trap.</p>

<h3>
  
  
  ❌ Mistake #3: Tools Fire During Greeting
</h3>

<p><strong>What happens:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Agent: "Hi, I'm Noah, your AI career c—"
User: "Hello!" (eager)
Gemini: *calls process_response() mid-greeting*
Database: *saves garbage data*
Agent: *confused about conversation state*
</code></pre>

</div>



<h3>
  
  
  ✅ Solution: Greeting Guard Flag
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">InterviewAgent</span><span class="p">(</span><span class="n">Agent</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">):</span>
        <span class="nf">super</span><span class="p">().</span><span class="nf">__init__</span><span class="p">(</span><span class="n">instructions</span><span class="o">=</span><span class="n">SYSTEM_PROMPT</span><span class="p">)</span>
        <span class="n">self</span><span class="p">.</span><span class="n">greeting_complete</span> <span class="o">=</span> <span class="bp">False</span>  <span class="c1"># ⭐ Start locked
</span>
    <span class="nd">@function_tool</span><span class="p">()</span>
    <span class="k">async</span> <span class="k">def</span> <span class="nf">process_user_response</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">data</span><span class="p">:</span> <span class="nb">dict</span><span class="p">):</span>
        <span class="c1"># GUARD: Block tool execution during greeting
</span>        <span class="k">if</span> <span class="ow">not</span> <span class="n">self</span><span class="p">.</span><span class="n">greeting_complete</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">SYSTEM: Wait for greeting to complete. Do not process yet.</span><span class="sh">"</span>

        <span class="c1"># Normal logic continues here
</span>        <span class="k">await</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>
        <span class="bp">...</span>

<span class="c1"># Unlock tools after greeting completes
</span><span class="nd">@session.on</span><span class="p">(</span><span class="sh">"</span><span class="s">agent_state_changed</span><span class="sh">"</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">on_state_changed</span><span class="p">(</span><span class="n">event</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">event</span><span class="p">.</span><span class="n">old_state</span> <span class="o">==</span> <span class="sh">"</span><span class="s">speaking</span><span class="sh">"</span> <span class="ow">and</span> <span class="n">event</span><span class="p">.</span><span class="n">new_state</span> <span class="o">==</span> <span class="sh">"</span><span class="s">listening</span><span class="sh">"</span><span class="p">:</span>
        <span class="k">if</span> <span class="ow">not</span> <span class="n">agent</span><span class="p">.</span><span class="n">greeting_complete</span><span class="p">:</span>
            <span class="n">agent</span><span class="p">.</span><span class="n">greeting_complete</span> <span class="o">=</span> <span class="bp">True</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ Greeting done, tools unlocked</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Why it works:</strong></p>

<ul>
<li>Agent speaks full greeting uninterrupted</li>
<li>Tools unlock only after state transition: <code>speaking</code> → <code>listening</code>
</li>
<li>No premature database writes</li>
</ul>

<p><strong>Real Data:</strong> Eliminated 100% of corrupted session starts.</p>




<h2>
  
  
  Part 5: Robustness Patterns
</h2>

<p>Production code needs fallbacks. Here are patterns from actual failures.</p>

<h3>
  
  
  Pattern 1: Timeout with Fallback
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Don't wait forever for participant
</span><span class="n">participant</span> <span class="o">=</span> <span class="bp">None</span>
<span class="k">for</span> <span class="n">attempt</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">30</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="n">remote_participants</span><span class="p">:</span>
        <span class="n">participant</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">ctx</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="n">remote_participants</span><span class="p">.</span><span class="nf">values</span><span class="p">())[</span><span class="mi">0</span><span class="p">]</span>
        <span class="k">break</span>
    <span class="k">await</span> <span class="n">asyncio</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>

<span class="k">if</span> <span class="ow">not</span> <span class="n">participant</span><span class="p">:</span>
    <span class="c1"># FALLBACK: Log and gracefully exit
</span>    <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sh">"</span><span class="s">⚠️ No participant after 30s</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">generate_reply</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">I can</span><span class="sh">'</span><span class="s">t hear anyone. Please refresh and try again.</span><span class="sh">"</span>
    <span class="p">)</span>
    <span class="k">return</span>  <span class="c1"># Exit cleanly
</span></code></pre>

</div>



<h3>
  
  
  Pattern 2: Database Connection Fallback
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">try</span><span class="p">:</span>
    <span class="n">initial_data</span> <span class="o">=</span> <span class="k">await</span> <span class="n">db</span><span class="p">.</span><span class="nf">get_candidate_data</span><span class="p">()</span>
<span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
    <span class="n">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">❌ DB connection failed: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="c1"># FALLBACK: Start fresh session instead of crashing
</span>    <span class="n">initial_data</span> <span class="o">=</span> <span class="p">{}</span>
    <span class="k">await</span> <span class="n">session</span><span class="p">.</span><span class="nf">generate_reply</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">Hi! Let</span><span class="sh">'</span><span class="s">s start fresh today. Tell me about your background.</span><span class="sh">"</span>
    <span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Pattern 3: Duplicate Agent Prevention
</h3>

<p><strong>Problem:</strong> Health check fails → Cloud Run keeps retrying → Multiple agents speak simultaneously</p>

<p><strong>Solution:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Health check server MUST bind to 0.0.0.0
</span><span class="k">def</span> <span class="nf">start_health_server</span><span class="p">():</span>
    <span class="n">port</span> <span class="o">=</span> <span class="nf">int</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">'</span><span class="s">PORT</span><span class="sh">'</span><span class="p">,</span> <span class="mi">8080</span><span class="p">))</span>
    <span class="n">httpd</span> <span class="o">=</span> <span class="nc">HTTPServer</span><span class="p">(</span>
        <span class="p">(</span><span class="sh">'</span><span class="s">0.0.0.0</span><span class="sh">'</span><span class="p">,</span> <span class="n">port</span><span class="p">),</span>  <span class="c1"># ⭐ NOT 'localhost'
</span>        <span class="n">HealthCheckHandler</span>
    <span class="p">)</span>
    <span class="n">httpd</span><span class="p">.</span><span class="nf">serve_forever</span><span class="p">()</span>

<span class="c1"># Don't override Cloud Run's PORT variable
</span><span class="nf">load_dotenv</span><span class="p">(</span><span class="n">override</span><span class="o">=</span><span class="bp">False</span><span class="p">)</span>  <span class="c1"># ⭐ Critical
</span></code></pre>

</div>



<p><strong>Why:</strong> Cloud Run expects health checks on <code>0.0.0.0</code>. If you bind to <code>localhost</code>, it fails and creates zombie agents.</p>




<h2>
  
  
  Part 6: Testing the Full Flow
</h2>

<h3>
  
  
  Local Development Setup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Terminal 1: Cloud SQL Proxy (if using Cloud SQL)</span>
./cloud-sql-proxy PROJECT:REGION:INSTANCE <span class="nt">--port</span> 5432

<span class="c"># Terminal 2: Voice Agent</span>
<span class="nb">cd </span>voice-agent
<span class="nb">source</span> .venv/bin/activate
python src/main.py dev

<span class="c"># Terminal 3: Frontend</span>
<span class="nb">cd </span>frontend
npm run dev
</code></pre>

</div>



<h3>
  
  
  Critical Test Cases
</h3>

<ol>
<li>
<strong>Happy Path</strong>: User joins → Agent greets → Conversation flows</li>
<li>
<strong>User Refresh</strong>: User closes tab mid-call → Reconnects → Session resumes</li>
<li>
<strong>Database Timeout</strong>: DB slow → Agent uses fallback greeting</li>
<li>
<strong>No Participant</strong>: Room created but user never joins → Agent exits gracefully</li>
<li>
<strong>Network Drop</strong>: User loses connection → Reconnects → Conversation continues</li>
</ol>




<h2>
  
  
  Lessons Learned: The DO/DON'T Checklist
</h2>

<h3>
  
  
  DO ✅
</h3>

<ul>
<li>Generate tokens <strong>server-side</strong> (never expose credentials)</li>
<li>Embed user identity in JWT token</li>
<li>Use <strong>auto-dispatch</strong> via <code>roomConfig</code> (more reliable than manual)</li>
<li>Start session <strong>FIRST</strong>, get identity <strong>AFTER</strong>
</li>
<li>
<strong>Guard tools</strong> during greeting with state flag</li>
<li>Add <strong>timeouts and fallbacks</strong> everywhere</li>
<li>Bind health server to <code>0.0.0.0</code> for Cloud Run</li>
<li>Test with <strong>database failures</strong> and <strong>network drops</strong>
</li>
</ul>

<h3>
  
  
  DON'T ❌
</h3>

<ul>
<li>Put LiveKit secrets in <code>NEXT_PUBLIC_*</code> variables</li>
<li>Wait for participant before starting session</li>
<li>Allow tools to execute during greeting</li>
<li>Assume database is always available</li>
<li>Skip health check server (Cloud Run requires it)</li>
<li>Override Cloud Run's <code>$PORT</code> environment variable</li>
<li>Deploy without testing the full flow locally first</li>
</ul>




<h2>
  
  
  From Demo to Production
</h2>

<p>The gap between LiveKit tutorials and production isn't just code—it's <strong>robustness thinking</strong>.</p>

<ul>
<li>
<strong>Tutorials assume happy paths</strong> (user joins, everything works)</li>
<li>
<strong>Production has 10 failure modes</strong> per integration point</li>
</ul>

<p>After 3 weeks of debugging these issues in production, I learned:</p>

<ol>
<li>
<strong>Audio subscription is timing-sensitive</strong> (start session first)</li>
<li>
<strong>Auto-dispatch beats manual dispatch</strong> (atomic operations win)</li>
<li>
<strong>State guards prevent race conditions</strong> (greeting flag pattern)</li>
<li>
<strong>Fallbacks save user experience</strong> (DB down? Start fresh)</li>
<li>
<strong>Health checks matter on Cloud Run</strong> (bind to <code>0.0.0.0</code>)</li>
</ol>




<h2>
  
  
  What's Next?
</h2>

<p>In the next article, I'll cover:</p>

<ul>
<li>Tool calling patterns with complex state machines</li>
<li>Database-backed session resume</li>
<li>The 80% CPU problem (and how I fixed it)</li>
</ul>

<p><strong>Building production voice AI?</strong></p>

<ul>
<li>Check my <a href="https://github.com/Behruamm/gemini-livekit-demo" rel="noopener noreferrer">GitHub</a> for code examples</li>
<li>👉 Subscribe to my <a href="https://behruamm.substack.com/" rel="noopener noreferrer">Substack</a> for more real-world patterns</li>
</ul>

<p><strong>What's your biggest voice AI integration pain point? Comment below.</strong> 👇</p>




