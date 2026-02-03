---
Title: Bridging Worlds: Building a SIP-to-WebRTC Gateway with Python and Drachtio
Description: 
Author: Lalit Mishra
Date: 2026-02-03T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Billion-Dollar Bridge
</h2>

<p>In the rush toward AI agents and browser-based communication, it is easy to forget where the money is. It isn't just in peer-to-peer video; it's in the <strong>Public Switched Telephone Network (PSTN)</strong>. Every major cloud contact center, conferencing platform, and voice AI startup eventually faces the same requirement: "We need to let users dial in from a phone," or "Our AI agent needs to call a customer's mobile."</p>

<p>This requires bridging <strong>SIP (Session Initiation Protocol)</strong>—the 90s-era standard that powers the telecom world—with <strong>WebRTC</strong>, the modern browser standard.</p>

<p>While they share common ancestors (SDP, RTP), they are practically alien to each other. SIP is text-based, transactional, and runs over UDP/TCP. WebRTC is event-based, encrypted (DTLS-SRTP), and runs over ICE/WebSockets. Building a gateway to bridge them is one of the hardest infrastructure challenges in real-time engineering.</p>

<p>For more detailed explanation, do check my YouTube Channel: <a href="https://www.youtube.com/@lalit_096/videos" rel="noopener noreferrer">The Lalit Official</a></p>

<p>A meme for making your mood light and enhance your humour.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl396dkslkvb9fh50hziw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl396dkslkvb9fh50hziw.png" alt="A split-panel meme" width="800" height="800"></a></p>

<h2>
  
  
  The Protocol Gap: Why Direct Interop Fails
</h2>

<p>To the uninitiated, SIP and WebRTC look similar. Both use the <strong>Session Description Protocol (SDP)</strong> to negotiate media. However, the transport layer tells a different story.</p>

<ol>
<li>
<strong>Signaling Mismatch:</strong> SIP is a transactional protocol. A <code>INVITE</code> request expects a <code>100 Trying</code>, <code>180 Ringing</code>, and <code>200 OK</code> response. It handles retransmissions, hop-by-hop routing, and <code>Via</code> header manipulation. WebRTC signaling is undefined by the standard—it’s just a JSON blob sent over a WebSocket.</li>
<li>
<strong>Security Mandates:</strong> SIP trunks (especially from legacy carriers) often send plain text SIP and unencrypted RTP audio (G.711). WebRTC <em>mandates</em> encryption. It requires DTLS for key exchange and SRTP for media. A browser will simply reject a plain RTP stream.</li>
<li>
<strong>NAT Traversal:</strong> SIP assumes relatively static IPs or simple NATs. WebRTC assumes hostile network environments, requiring ICE (Interactive Connectivity Establishment) and STUN/TURN servers to punch holes in firewalls.</li>
</ol>

<p>If you try to terminate a SIP trunk directly in a Python script using a raw socket, you will spend months reinventing the wheel of transaction state machines and header parsing. You need a dedicated SIP stack.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmlelva9yj534zfpq5b1t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmlelva9yj534zfpq5b1t.png" alt="Left side: " width="800" height="446"></a></p>

<h2>
  
  
  The Middleware Solution: Drachtio &amp; The Sidecar Pattern
</h2>

<p>In the Node.js world, <strong>Drachtio</strong> has emerged as a powerhouse SIP middleware. Built by Dave Horton, it consists of a C++ core (for high-performance message parsing) and a Node.js signaling resource framework (<code>drachtio-srf</code>).</p>

<p>For a Python shop, using a Node.js tool might seem counter-intuitive. However, the Python ecosystem lacks a SIP stack with the maturity of Drachtio or the raw power of Kamailio. The <strong>Sidecar Pattern</strong> offers the best of both worlds:</p>

<ul>
<li>
<strong>Drachtio (Node.js)</strong> acts as the <strong>SIP Edge</strong>. It handles the low-level "noise" of SIP: parsing headers, managing transaction timers, and handling keep-alives.</li>
<li>
<strong>Flask/Quart (Python)</strong> acts as the <strong>Brain</strong>. It handles the business logic: "Is this user active?", "Which AI agent should handle this call?", "Record this call?".</li>
</ul>

<p>The two services communicate via high-speed <strong>HTTP Webhooks</strong> or a shared <strong>Redis</strong> bus. Drachtio receives the <code>INVITE</code>, pauses processing, asks Python what to do, and then executes the signaling instruction.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Face8tbwcw7d2mgql3hre.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Face8tbwcw7d2mgql3hre.png" alt="From left to right: " width="800" height="446"></a></p>

<h3>
  
  
  Comparison: Drachtio vs. Kamailio (KEMI)
</h3>

<p>The alternative to Drachtio is <strong>Kamailio</strong>, the legendary open-source SIP server. With the <strong>KEMI (Kamailio Embedded Interface)</strong> framework, you can write SIP routing logic directly in Python scripts embedded within Kamailio.</p>

<ul>
<li>
<strong>Kamailio + KEMI:</strong> Extreme performance (tens of thousands of calls per second). However, it has a brutal learning curve. You must understand SIP routing blocks, memory management, and C-like configuration syntax. Debugging embedded Python crashes can be difficult.</li>
<li>
<strong>Drachtio:</strong> High performance (thousands of calls per second). Extremely developer-friendly API. Decouples logic from the SIP engine.</li>
</ul>

<p>For most modern WebRTC gateways (Cloud PBX, AI Voice Agents), Drachtio provides a faster time-to-market with sufficient scale. Kamailio is reserved for massive carrier-grade switching.</p>

<h2>
  
  
  Architecture: The Inbound Call Flow
</h2>

<p>Let's trace a call from a regular phone number to a browser-based agent.</p>

<h3>
  
  
  1. The SIP INVITE
</h3>

<p>A call arrives at your infrastructure via a SIP Trunk. Drachtio listens on port 5060 (UDP/TCP).<br>
<code>INVITE sip:+15550199@sip.myapp.com SIP/2.0</code></p>
<h3>
  
  
  2. The Python Authorization
</h3>

<p>Drachtio parses the <code>INVITE</code>. Instead of routing it immediately, it fires a webhook (or Redis message) to your Flask application:<br>
<code>POST /webhook/voice/incoming</code><br>
Payload: <code>{ "caller": "+1234567890", "callee": "+15550199", "call_id": "..." }</code></p>

<p>Your Python app checks the database. "Is +15550199 assigned to an active Agent?" It finds that Agent ID <code>agent-42</code> is online.</p>
<h3>
  
  
  3. The Bridging Instruction
</h3>

<p>Python responds to Drachtio: "Bridge this call to the WebRTC session for <code>agent-42</code>."</p>
<h3>
  
  
  4. Media Negotiation (The Critical Step)
</h3>

<p>This is where the magic happens. The SIP trunk offers <strong>G.711 (PCMU)</strong> audio over <strong>RTP</strong>. The browser requires <strong>Opus</strong> audio over <strong>SRTP</strong>. You cannot just connect the sockets.</p>

<p>Drachtio commands <strong>RTPEngine</strong> (a kernel-space media proxy) to allocate endpoints.</p>

<ul>
<li>
<strong>Side A (SIP):</strong> IP: 1.2.3.4, Codec: PCMU, Proto: RTP/AVP</li>
<li>
<strong>Side B (WebRTC):</strong> IP: 5.6.7.8, Codec: Opus, Proto: UDP/TLS/RTP/SAVPF (DTLS-SRTP)</li>
</ul>

<p>RTPEngine acts as the translator, transcoding audio and terminating encryption in real-time.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgx49am5laqilnwdkt11h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgx49am5laqilnwdkt11h.png" alt="A server labeled RTPEngine" width="800" height="446"></a></p>
<h2>
  
  
  Python Integration: The Orchestrator
</h2>

<p>While Drachtio manages the SIP state machine, your Python code manages the <em>application state</em>. Here is a conceptual example of how a Flask route orchestrates this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nd">@app.route</span><span class="p">(</span><span class="sh">'</span><span class="s">/hooks/sip-invite</span><span class="sh">'</span><span class="p">,</span> <span class="n">methods</span><span class="o">=</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">handle_sip_invite</span><span class="p">():</span>
    <span class="n">data</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">json</span>
    <span class="n">from_number</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">sip</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">from</span><span class="sh">'</span><span class="p">]</span>
    <span class="n">to_number</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">sip</span><span class="sh">'</span><span class="p">][</span><span class="sh">'</span><span class="s">to</span><span class="sh">'</span><span class="p">]</span>

    <span class="c1"># 1. Lookup the WebRTC user
</span>    <span class="n">agent</span> <span class="o">=</span> <span class="n">user_repo</span><span class="p">.</span><span class="nf">find_agent_by_did</span><span class="p">(</span><span class="n">to_number</span><span class="p">)</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">agent</span> <span class="ow">or</span> <span class="ow">not</span> <span class="n">agent</span><span class="p">.</span><span class="n">is_online</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">jsonify</span><span class="p">({</span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">reject</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="mi">480</span><span class="p">,</span> <span class="sh">"</span><span class="s">reason</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Temporarily Unavailable</span><span class="sh">"</span><span class="p">})</span>

    <span class="c1"># 2. Get Media Parameters (SDP) from RTPEngine
</span>    <span class="c1"># (In a real app, Drachtio handles the RTPEngine interaction, 
</span>    <span class="c1"># but Python might dictate the codec policy)
</span>
    <span class="c1"># 3. Notify the Browser via WebSocket
</span>    <span class="c1"># We send the incoming call event to the frontend React app
</span>    <span class="n">socket_manager</span><span class="p">.</span><span class="nf">emit</span><span class="p">(</span><span class="n">agent</span><span class="p">.</span><span class="nb">id</span><span class="p">,</span> <span class="sh">'</span><span class="s">incoming_call</span><span class="sh">'</span><span class="p">,</span> <span class="p">{</span>
        <span class="sh">'</span><span class="s">caller</span><span class="sh">'</span><span class="p">:</span> <span class="n">from_number</span><span class="p">,</span>
        <span class="sh">'</span><span class="s">sdp</span><span class="sh">'</span><span class="p">:</span> <span class="n">data</span><span class="p">[</span><span class="sh">'</span><span class="s">sdp</span><span class="sh">'</span><span class="p">]</span> <span class="c1"># The transcodable SDP from RTPEngine
</span>    <span class="p">})</span>

    <span class="k">return</span> <span class="nf">jsonify</span><span class="p">({</span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">ringing</span><span class="sh">"</span><span class="p">})</span> 

</code></pre>

</div>



<p>Notice that Python never touches a SIP header or a raw UDP packet. It deals with high-level concepts: Users, Status, and Signaling instructions.</p>

<h2>
  
  
  Handling Media: Why RTPEngine is Mandatory
</h2>

<p>You cannot build a production SIP-to-WebRTC gateway without a specialized media proxy. <strong>RTPEngine</strong> is the industry standard for this because it operates in <strong>kernel space</strong> for packet forwarding, minimizing latency and jitter.</p>

<p>Its responsibilities in this architecture are:</p>

<ol>
<li>
<strong>ICE Termination:</strong> It acts as a "Lite" ICE server, allowing the browser to connect to it even behind NAT.</li>
<li>
<strong>DTLS Handshake:</strong> It performs the cryptographic handshake with the browser to establish the SRTP keys.</li>
<li>
<strong>Transcoding:</strong> It converts the 8kHz G.711 stream from the PSTN into the 48kHz Opus stream for the browser (and vice versa).</li>
<li>
<strong>RTCP Feedback:</strong> It generates the necessary WebRTC keep-alives that browsers expect, which dumb SIP trunks do not provide.</li>
</ol>

<h2>
  
  
  Conclusion: Complexity Encapsulated
</h2>

<p>Building a SIP gateway used to require deep C++ knowledge and months of debugging race conditions. By leveraging the <strong>Sidecar Pattern</strong> with Drachtio and Python, you encapsulate that complexity. Drachtio handles the rigid, archaic rules of SIP. RTPEngine handles the heavy lifting of media encryption and transcoding. And your Python backend? It stays clean, modern, and focused on what matters: the experience of the user on the other end of the line.</p>

