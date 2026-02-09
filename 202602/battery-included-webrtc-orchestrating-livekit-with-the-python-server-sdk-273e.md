---
Title: Battery-Included WebRTC: Orchestrating LiveKit with the Python Server SDK
Description: 
Author: Lalit Mishra
Date: 2026-02-09T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Evolution from "Plumbing" to "Platform"
</h2>

<p>For the better part of a decade, building scalable real-time video applications meant becoming a plumber. You didn't just build an app; you managed <code>janus-gateway</code> config files, tuned <code>mediasoup</code> workers, wrestled with <code>coturn</code> for NAT traversal, and wrote custom C++ wrappers to handle recording. You were effectively building a telecom carrier from scratch just to add video chat to a website.</p>

<p><strong>LiveKit</strong> represents the maturation of this stack. It is an opinionated, "batteries-included" WebRTC infrastructure that abstracts the low-level media transport (SFU) while exposing rigorous control via SDKs.</p>

<p>For the Python backend architect, LiveKit fundamentally shifts the responsibility model. You no longer manage media <em>packets</em>; you manage media <em>sessions</em>. Your Python backend becomes the <strong>Orchestrator</strong>, using the <code>livekit-api</code> server SDK to provision rooms, mint security tokens, and trigger cloud recordings, all while communicating with the LiveKit server via high-performance <strong>Twirp</strong> (a structured RPC framework based on Protobuf).</p>

<h2>
  
  
  The LiveKit Ecosystem: Architecture of Abstraction
</h2>

<p>To orchestrate LiveKit, you must understand its topology. It is not a monolithic black box but a distributed system of services.</p>

<ol>
<li>
<strong>LiveKit Server (Go):</strong> The SFU (Selective Forwarding Unit). It handles the heavy lifting: receiving RTP packets, performing bandwidth estimation, and forwarding streams to subscribers.</li>
<li>
<strong>Client SDKs:</strong> Libraries running on the user's device (React, Swift, Kotlin, Unity). They handle device capture and the WebRTC handshake.</li>
<li>
<strong>Server SDKs (Python/Go/Node):</strong> This is your domain. The Server SDK does <em>not</em> handle media (usually). It handles <strong>Signaling and Control</strong>. It talks to the LiveKit Server to say, "Create a room," "Mute this user," or "Start recording."</li>
</ol>

<p>This separation is critical. Your Python Flask/FastAPI application is the <strong>Control Plane</strong>. The LiveKit Server is the <strong>Data Plane</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3hs67nbww7ojxiaebdsb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3hs67nbww7ojxiaebdsb.png" alt="a system architecture diagram. Visual: Central node " width="800" height="446"></a></p>

<h2>
  
  
  The Orchestrator's Tool: <code>livekit-api</code>
</h2>

<p>In Python, the interaction happens primarily through the <code>livekit-api</code> package. Unlike the <code>livekit</code> package (which is used for building Real-time Agents that send/receive audio), <code>livekit-api</code> is purely for HTTP/RPC management.</p>

<h3>
  
  
  1. The Gatekeeper: JWT Authentication
</h3>

<p>LiveKit delegates authentication entirely to your backend. The LiveKit server has no user database. Instead, it relies on <strong>JSON Web Tokens (JWT)</strong> signed with an API Key and Secret that you share between your Python backend and the LiveKit server.</p>

<p>When a user wants to join a room, they request a token from your API. Your Python code defines exactly what that user can do via <strong>Video Grants</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">os</span>
<span class="kn">from</span> <span class="n">livekit</span> <span class="kn">import</span> <span class="n">api</span>

<span class="c1"># Ensure LIVEKIT_API_KEY and LIVEKIT_API_SECRET are in env
</span><span class="k">def</span> <span class="nf">create_participant_token</span><span class="p">(</span><span class="n">room_name</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">participant_identity</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">is_admin</span><span class="p">:</span> <span class="nb">bool</span> <span class="o">=</span> <span class="bp">False</span><span class="p">):</span>
    <span class="n">grant</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">VideoGrants</span><span class="p">(</span>
        <span class="n">room_join</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
        <span class="n">room</span><span class="o">=</span><span class="n">room_name</span><span class="p">,</span>
        <span class="n">can_publish</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
        <span class="n">can_subscribe</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
        <span class="c1"># Administrative powers
</span>        <span class="n">room_admin</span><span class="o">=</span><span class="n">is_admin</span><span class="p">,</span>  
        <span class="n">room_record</span><span class="o">=</span><span class="n">is_admin</span>
    <span class="p">)</span>

    <span class="n">token</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">AccessToken</span><span class="p">()</span> \
       <span class="p">.</span><span class="nf">with_identity</span><span class="p">(</span><span class="n">participant_identity</span><span class="p">)</span> \
       <span class="p">.</span><span class="nf">with_name</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">User </span><span class="si">{</span><span class="n">participant_identity</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span> \
       <span class="p">.</span><span class="nf">with_grants</span><span class="p">(</span><span class="n">grant</span><span class="p">)</span> \
       <span class="p">.</span><span class="nf">with_ttl</span><span class="p">(</span><span class="mi">60</span> <span class="o">*</span> <span class="mi">60</span><span class="p">)</span> <span class="c1"># 1 hour expiration
</span>
    <span class="k">return</span> <span class="n">token</span><span class="p">.</span><span class="nf">to_jwt</span><span class="p">()</span>

</code></pre>

</div>



<p><strong>Architectural Note:</strong> Never generate tokens on the client. Always generate them server-side. This allows you to revoke access, enforce bans, or dynamically assign permissions (e.g., a "stage hand" user who can mute others but not publish video) based on your application's business logic.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8hx6zd5fzylyosi6eavy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8hx6zd5fzylyosi6eavy.png" alt="a security flow diagram. Visual: Step 1: Client sends " width="800" height="446"></a></p>

<h3>
  
  
  2. Room Lifecycle Management
</h3>

<p>While rooms can be created automatically when a user joins, production systems often require <strong>Explicit Room Provisioning</strong>. You might want to create a room 5 minutes before a meeting starts, set specific timeouts, or limit the max participants.</p>

<p>Using the <code>LiveKitAPI</code> client, this becomes a strictly typed <code>async</code> operation.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">asyncio</span>
<span class="kn">from</span> <span class="n">livekit</span> <span class="kn">import</span> <span class="n">api</span>

<span class="k">async</span> <span class="k">def</span> <span class="nf">provision_meeting_room</span><span class="p">(</span><span class="n">meeting_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="c1"># Initialize the API client
</span>    <span class="n">lkapi</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">LiveKitAPI</span><span class="p">(</span>
        <span class="n">url</span><span class="o">=</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">LIVEKIT_URL</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">api_key</span><span class="o">=</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">LIVEKIT_API_KEY</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">api_secret</span><span class="o">=</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">LIVEKIT_API_SECRET</span><span class="sh">"</span><span class="p">),</span>
    <span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="c1"># Create a room with strict settings
</span>        <span class="n">room_info</span> <span class="o">=</span> <span class="k">await</span> <span class="n">lkapi</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="nf">create_room</span><span class="p">(</span>
            <span class="n">api</span><span class="p">.</span><span class="nc">CreateRoomRequest</span><span class="p">(</span>
                <span class="n">name</span><span class="o">=</span><span class="n">meeting_id</span><span class="p">,</span>
                <span class="n">empty_timeout</span><span class="o">=</span><span class="mi">300</span><span class="p">,</span> <span class="c1"># Close after 5 mins if empty
</span>                <span class="n">max_participants</span><span class="o">=</span><span class="mi">50</span><span class="p">,</span>
                <span class="n">metadata</span><span class="o">=</span><span class="sh">'</span><span class="s">{</span><span class="sh">"</span><span class="s">type</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">webinar</span><span class="sh">"</span><span class="s">, </span><span class="sh">"</span><span class="s">host_id</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">user_123</span><span class="sh">"</span><span class="s">}</span><span class="sh">'</span>
            <span class="p">)</span>
        <span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Room </span><span class="sh">'</span><span class="si">{</span><span class="n">room_info</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="sh">'</span><span class="s"> created with SID: </span><span class="si">{</span><span class="n">room_info</span><span class="p">.</span><span class="n">sid</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">room_info</span>
    <span class="k">finally</span><span class="p">:</span>
        <span class="k">await</span> <span class="n">lkapi</span><span class="p">.</span><span class="nf">aclose</span><span class="p">()</span>

</code></pre>

</div>



<p>This API also allows you to <strong>moderate</strong> active rooms. You can programmatically mute a disruptive user, update their permissions (e.g., promoting an attendee to a speaker), or remove them entirely.</p>

<h3>
  
  
  3. Webhooks: The Feedback Loop
</h3>

<p>Orchestration is bidirectional. Your Python backend tells LiveKit what to do, but LiveKit must also tell your backend what happened. Did the recording finish? Did the room close? Did a user disconnect unexpectedly?</p>

<p>LiveKit pushes these events via <strong>Webhooks</strong>. Your backend must verify the cryptographic signature of these webhooks to ensure they are legitimate.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">flask</span> <span class="kn">import</span> <span class="n">Flask</span><span class="p">,</span> <span class="n">request</span>
<span class="kn">from</span> <span class="n">livekit</span> <span class="kn">import</span> <span class="n">api</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">Flask</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>
<span class="n">receiver</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">WebhookReceiver</span><span class="p">(</span><span class="n">api</span><span class="p">.</span><span class="nc">TokenVerifier</span><span class="p">())</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">'</span><span class="s">/livekit/webhook</span><span class="sh">'</span><span class="p">,</span> <span class="n">methods</span><span class="o">=</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">handle_webhook</span><span class="p">():</span>
    <span class="n">auth_header</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">headers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">Authorization</span><span class="sh">'</span><span class="p">)</span>
    <span class="n">body</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">data</span><span class="p">.</span><span class="nf">decode</span><span class="p">(</span><span class="sh">'</span><span class="s">utf-8</span><span class="sh">'</span><span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">event</span> <span class="o">=</span> <span class="n">receiver</span><span class="p">.</span><span class="nf">receive</span><span class="p">(</span><span class="n">body</span><span class="p">,</span> <span class="n">auth_header</span><span class="p">)</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">return</span> <span class="sh">"</span><span class="s">Invalid signature</span><span class="sh">"</span><span class="p">,</span> <span class="mi">401</span>

    <span class="k">if</span> <span class="n">event</span><span class="p">.</span><span class="n">event</span> <span class="o">==</span> <span class="sh">"</span><span class="s">room_finished</span><span class="sh">"</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Room </span><span class="si">{</span><span class="n">event</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="n">name</span><span class="si">}</span><span class="s"> ended. Duration: </span><span class="si">{</span><span class="n">event</span><span class="p">.</span><span class="n">room</span><span class="p">.</span><span class="n">duration</span><span class="si">}</span><span class="s">s</span><span class="sh">"</span><span class="p">)</span>
        <span class="c1"># Trigger billing logic or cleanup
</span>
    <span class="k">elif</span> <span class="n">event</span><span class="p">.</span><span class="n">event</span> <span class="o">==</span> <span class="sh">"</span><span class="s">participant_joined</span><span class="sh">"</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">User </span><span class="si">{</span><span class="n">event</span><span class="p">.</span><span class="n">participant</span><span class="p">.</span><span class="n">identity</span><span class="si">}</span><span class="s"> joined.</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">return</span> <span class="sh">"</span><span class="s">ok</span><span class="sh">"</span>

</code></pre>

</div>



<h2>
  
  
  The "Magic" of Abstraction: Simulcast &amp; Dynacast
</h2>

<p>In raw WebRTC (e.g., using <code>mediasoup</code>), enabling <strong>Simulcast</strong> (sending multiple qualities of the same video) requires complex client-side configuration and server-side handling. You have to manually negotiate spatial layers.</p>

<p>LiveKit abstracts this entirely.</p>

<ol>
<li>
<strong>Simulcast:</strong> The Client SDK automatically publishes 3 layers (low, medium, high) if the bandwidth permits.</li>
<li>
<strong>Dynacast:</strong> The LiveKit Server monitors what every subscriber is <em>actually</em> watching. If User A minimizes User B's video to a 100x100 thumbnail, the server <em>automatically</em> switches User A to the low-quality stream of User B. If User A maximizes the video, the server switches them to high-quality.</li>
</ol>

<p>This optimization saves massive amounts of bandwidth and CPU on the client side, and as a Python architect, you get it for free. You don't write code for it; it is an infrastructure guarantee.</p>

<h2>
  
  
  Egress: The Server-Side Recording Pipeline
</h2>

<p>Recording is the hardest problem in WebRTC. You cannot just "save" packets because they are encrypted (SRTP) and arrive out of order with variable bitrates. Traditionally, you had to spin up a headless Chrome instance with Selenium, join the call, and screen record it. It was brittle and resource-heavy.</p>

<p>LiveKit provides <strong>Egress</strong> as a first-class service. It runs its own worker pool (often using GStreamer/Chrome under the hood) but exposes a clean API to your Python backend.</p>

<p>You can trigger a <strong>Composite Recording</strong> (mixing audio/video into a standard layout) or a <strong>Track Recording</strong> (saving raw ISO feeds) with one call.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2bwsqsywvvzy8tyf9zue.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2bwsqsywvvzy8tyf9zue.png" alt="a diagram showing the Egress flow. Visual: " start="" width="800" height="446"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">async</span> <span class="k">def</span> <span class="nf">start_recording</span><span class="p">(</span><span class="n">room_name</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="n">lkapi</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">LiveKitAPI</span><span class="p">(...)</span>

    <span class="c1"># Configure output to S3
</span>    <span class="n">s3_output</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">EncodedFileOutput</span><span class="p">(</span>
        <span class="n">filepath</span><span class="o">=</span><span class="sa">f</span><span class="sh">"</span><span class="s">recordings/</span><span class="si">{</span><span class="n">room_name</span><span class="si">}</span><span class="s">/</span><span class="si">{</span><span class="sh">'</span><span class="si">{</span><span class="n">time</span><span class="si">}</span><span class="sh">'</span><span class="si">}</span><span class="s">.mp4</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">s3</span><span class="o">=</span><span class="n">api</span><span class="p">.</span><span class="nc">S3Upload</span><span class="p">(</span>
            <span class="n">access_key</span><span class="o">=</span><span class="sh">"</span><span class="s">...</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">secret</span><span class="o">=</span><span class="sh">"</span><span class="s">...</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">bucket</span><span class="o">=</span><span class="sh">"</span><span class="s">my-bucket</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">region</span><span class="o">=</span><span class="sh">"</span><span class="s">us-east-1</span><span class="sh">"</span>
        <span class="p">)</span>
    <span class="p">)</span>

    <span class="n">request</span> <span class="o">=</span> <span class="n">api</span><span class="p">.</span><span class="nc">RoomCompositeEgressRequest</span><span class="p">(</span>
        <span class="n">room_name</span><span class="o">=</span><span class="n">room_name</span><span class="p">,</span>
        <span class="n">layout</span><span class="o">=</span><span class="sh">"</span><span class="s">grid</span><span class="sh">"</span><span class="p">,</span> <span class="c1"># or 'speaker-dark', 'single-speaker'
</span>        <span class="nb">file</span><span class="o">=</span><span class="n">s3_output</span><span class="p">,</span>
        <span class="c1"># Encode options (H.264 High Profile)
</span>        <span class="n">preset</span><span class="o">=</span><span class="n">api</span><span class="p">.</span><span class="n">EncodingOptionsPreset</span><span class="p">.</span><span class="n">H264_1080P_30</span>
    <span class="p">)</span>

    <span class="n">info</span> <span class="o">=</span> <span class="k">await</span> <span class="n">lkapi</span><span class="p">.</span><span class="n">egress</span><span class="p">.</span><span class="nf">start_room_composite_egress</span><span class="p">(</span><span class="n">request</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Recording started. Egress ID: </span><span class="si">{</span><span class="n">info</span><span class="p">.</span><span class="n">egress_id</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

</code></pre>

</div>



<p>This single function call replaces weeks of engineering work required to build a custom recording pipeline using FFmpeg or GStreamer directly.</p>

<h2>
  
  
  Conclusion: Velocity vs. Control
</h2>

<p>LiveKit does not prevent you from dropping down to the metal if you need to; you can still write raw Go services that interface with the SFU. However, for 95% of use cases—telehealth, virtual classrooms, live events—the abstraction level provided by the Python SDK is the "sweet spot."</p>

<p>By treating WebRTC as a managed service rather than a protocol to be implemented, you shift your engineering effort from <strong>infrastructure maintenance</strong> (keeping the SFU alive, handling reconnects) to <strong>application features</strong> (moderation tools, AI integration, recording workflows). In the modern real-time economy, that velocity is your competitive advantage.</p>

