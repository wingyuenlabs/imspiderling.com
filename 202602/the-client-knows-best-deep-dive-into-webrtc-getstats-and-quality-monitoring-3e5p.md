---
Title: The Client Knows Best: Deep Dive into WebRTC getStats() and Quality Monitoring
Description: 
Author: Lalit Mishra
Date: 2026-02-14T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Opening Context – Why Server Logs Lie
</h2>

<p>In the distributed architecture of real-time communications, the server is often the last to know about a degradation in quality. You can scale your Selective Forwarding Units (SFUs) to handle ten thousand concurrent streams, ensure your signaling WebSocket servers have sub-millisecond latency, and provision massive bandwidth headroom, yet your users will still report frozen video and robotic audio.</p>

<p>Traditional backend observability relies on server-side logs: socket connection states, ICE (Interactive Connectivity Establishment) completion events, and SFU ingress/egress bitrates. These metrics are necessary but insufficient. They represent the infrastructure's view of the world, not the user's reality.</p>

<p>A "green" server dashboard frequently coexists with a "red" user experience. The server knows it forwarded a packet. It does not know if that packet arrived at the client, if it arrived in the correct order, or if the client’s decoder had the CPU cycles to render it. Packet loss, jitter buffer underruns, and decode failures are strictly client-side phenomena.</p>

<p>The only authoritative source of truth for media quality is the client itself. The WebRTC <code>RTCPeerConnection.getStats()</code> API provides a direct window into the media engine's internal state. It exposes the raw telemetry required to distinguish between a network failure, a device limitation, and a platform bug. To build a robust WebRTC platform, you must treat the client not just as a consumer of media, but as an active telemetry node in your observability mesh.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhpm0y41s6h506xcxd533.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhpm0y41s6h506xcxd533.png" alt="Meme Placeholder: Backend dashboard all green. User screaming “Video frozen.” Caption: “When your logs say OK but the client says otherwise.”" width="800" height="800"></a></p>

<h2>
  
  
  The getStats Standard – Understanding the W3C Report Model
</h2>

<p>The W3C WebRTC Statistics API is a standardized interface that returns a <code>RTCStatsReport</code>. Unlike earlier, browser-specific implementations (such as the legacy <code>getStats</code> in Chrome), the modern standard returns a map-like structure where every entry represents a specific component of the peer connection at a specific point in time.</p>

<p>Invoking <code>await peerConnection.getStats()</code> returns an iterable map of <code>RTCStats</code> dictionaries. Navigating this map requires an understanding of the relationship between the report types. The architecture is a graph, not a tree.</p>

<p>The critical report types for quality monitoring are:</p>

<ul>
<li>
<strong><code>inbound-rtp</code></strong>: Describes media received from a remote peer. This is where you find packet loss, bytes received, and decode metrics for incoming video/audio.</li>
<li>
<strong><code>outbound-rtp</code></strong>: Describes media sent by the local client. This tracks bytes sent and encoding CPU usage.</li>
<li>
<strong><code>remote-inbound-rtp</code></strong>: This is a mirror. It contains data sent back via RTCP Receiver Reports (RR), telling the sender how the receiver is perceiving the stream (primarily Round Trip Time and fraction lost).</li>
<li>
<strong><code>candidate-pair</code></strong>: Describes the active transport path (local IP to remote IP). This provides the most accurate RTT and current throughput capacity.</li>
<li>
<strong><code>transport</code></strong>: Aggregated transport metrics, including encryption (DTLS) state.</li>
</ul>

<p>Each report contains a <code>timestamp</code> (high-resolution DOMHighResTimeStamp), a <code>type</code>, and an <code>id</code>. The <code>id</code> is crucial for linking reports. For example, an <code>inbound-rtp</code> report will have a <code>transportId</code> field pointing to the associated <code>transport</code> report.</p>

<p>Traversing this structure requires iterating over the report values and filtering by type. The sheer volume of data returned—often dozens of reports per second—means that raw ingestion is prohibitively expensive. We must identify and extract only the signals that matter.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8nsvvt1n74yw57yzikr1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8nsvvt1n74yw57yzikr1.png" alt="Diagram showing RTCPeerConnection at center, branching to inbound-rtp, outbound-rtp, candidate-pair, transport reports, with timestamps and metric labels." width="800" height="446"></a></p>

<h2>
  
  
  Identifying Critical Metrics
</h2>

<p>Not all stats are created equal. While the spec defines hundreds of metrics, only a specific subset correlates directly with perceived user quality (QoE).</p>

<h3>
  
  
  Round Trip Time (RTT)
</h3>

<p>RTT is the primary indicator of network latency. High RTT degrades interactivity and forces the congestion control algorithm to be more conservative.</p>

<ul>
<li>
<strong>Source:</strong> <code>candidate-pair.currentRoundTripTime</code>. This is the most immediate measure of the network path.</li>
<li>
<strong>Secondary Source:</strong> <code>remote-inbound-rtp.roundTripTime</code>. This is calculated from RTCP reports and is useful for understanding the "view from the other side."</li>
</ul>

<h3>
  
  
  Jitter
</h3>

<p>Jitter measures the variance in packet arrival time. The WebRTC jitter buffer must delay playback to smooth this variance. If jitter exceeds the buffer size, packets are discarded, resulting in robotic audio or frozen video.</p>

<ul>
<li>
<strong>Source:</strong> <code>inbound-rtp.jitter</code>.</li>
</ul>

<h3>
  
  
  Packets Lost
</h3>

<p>Packet loss is the enemy of smooth media. WebRTC can tolerate minor loss via NACKs (Negative Acknowledgments) and FEC (Forward Error Correction), but sustained loss &gt; 5% usually results in visible artifacts.</p>

<ul>
<li>
<strong>Source:</strong> <code>inbound-rtp.packetsLost</code>. Note that this is a <em>cumulative</em> counter. You must calculate the delta between samples to determine the current loss rate.</li>
</ul>

<h3>
  
  
  Frames Decoded and Dropped
</h3>

<p>This is the "smoking gun" for video freezes.</p>

<ul>
<li>
<strong><code>framesDecoded</code></strong>: A cumulative count of frames successfully passed through the decoder.</li>
<li>
<strong><code>framesDropped</code></strong>: Frames received but discarded before decoding (usually due to CPU saturation or buffer underruns).</li>
</ul>

<p><strong>The Freeze Detector:</strong><br>
The most reliable way to detect a video freeze is to monitor the delta of <code>framesDecoded</code> over time.<br>
If <code>bytesReceived</code> is increasing (network is flowing) but <code>framesDecoded</code> delta is zero, the user is looking at a frozen frame. This specific signature distinguishes a network cutoff (where bytes stop) from a decoder failure or keyframe starvation (where bytes flow but video doesn't update).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb7jux8f667vtdi9mv8iq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb7jux8f667vtdi9mv8iq.png" alt="Time-series graph showing framesDecoded plateauing while connectionState remains connected." width="800" height="446"></a></p>
<h2>
  
  
  Client-Side Normalization Architecture
</h2>

<p>Sending the full <code>getStats()</code> dump to your backend every second is an anti-pattern. It wastes client bandwidth, CPU, and storage. The client must act as an edge processor, normalizing and aggregating data before transmission.</p>

<p><strong>The Strategy:</strong></p>

<ol>
<li>
<strong>Poll Interval:</strong> Poll <code>getStats()</code> every 1 to 5 seconds.</li>
<li>
<strong>Delta Computation:</strong> Store the previous report. Calculate <code>current_value - previous_value</code> for cumulative counters like <code>bytesReceived</code>, <code>packetsLost</code>, and <code>framesDecoded</code>.</li>
<li>
<strong>Normalization:</strong> Convert raw bytes to bits-per-second (bps). Convert cumulative loss to "loss per interval."</li>
<li>
<strong>Batching:</strong> Accumulate 5-10 samples before sending to the server to reduce HTTP/WebSocket overhead.</li>
</ol>
<h3>
  
  
  Production-Grade JavaScript Implementation
</h3>

<p>The following class implements a robust telemetry agent. It handles the difference between standard and legacy stats and computes the critical deltas required for freeze detection.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">class</span> <span class="nc">WebRTCTelemetryAgent</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">peerConnection</span><span class="p">,</span> <span class="nx">ingestionUrl</span><span class="p">,</span> <span class="nx">pollingIntervalMs</span> <span class="o">=</span> <span class="mi">2000</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">pc</span> <span class="o">=</span> <span class="nx">peerConnection</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">ingestionUrl</span> <span class="o">=</span> <span class="nx">ingestionUrl</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">intervalMs</span> <span class="o">=</span> <span class="nx">pollingIntervalMs</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">previousStats</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Map</span><span class="p">();</span> <span class="c1">// Store previous reports by ID</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">timer</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">sessionId</span> <span class="o">=</span> <span class="nx">crypto</span><span class="p">.</span><span class="nf">randomUUID</span><span class="p">();</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">buffer</span> <span class="o">=</span> <span class="p">[];</span>
  <span class="p">}</span>

  <span class="nf">start</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">timer</span> <span class="o">=</span> <span class="nf">setInterval</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nf">collectStats</span><span class="p">(),</span> <span class="k">this</span><span class="p">.</span><span class="nx">intervalMs</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="nf">stop</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">timer</span><span class="p">)</span> <span class="nf">clearInterval</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">timer</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">flush</span><span class="p">();</span> <span class="c1">// Send remaining data</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">collectStats</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">pc</span><span class="p">.</span><span class="nx">connectionState</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">closed</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">stop</span><span class="p">();</span>
      <span class="k">return</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">try</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">report</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">pc</span><span class="p">.</span><span class="nf">getStats</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">metrics</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">parseReport</span><span class="p">(</span><span class="nx">report</span><span class="p">);</span>

      <span class="k">if </span><span class="p">(</span><span class="nx">metrics</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">buffer</span><span class="p">.</span><span class="nf">push</span><span class="p">(</span><span class="nx">metrics</span><span class="p">);</span>
        <span class="c1">// Batch size of 5 samples creates a ~10s send window</span>
        <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">buffer</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;=</span> <span class="mi">5</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">this</span><span class="p">.</span><span class="nf">flush</span><span class="p">();</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">e</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Telemetry error:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">e</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="nf">parseReport</span><span class="p">(</span><span class="nx">report</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="nx">selectedCandidatePair</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">inboundVideo</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="kd">let</span> <span class="nx">inboundAudio</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>

    <span class="c1">// 1. Identify active candidate pair and inbound streams</span>
    <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">stats</span> <span class="k">of</span> <span class="nx">report</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">stats</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">candidate-pair</span><span class="dl">'</span> <span class="o">&amp;&amp;</span> <span class="nx">stats</span><span class="p">.</span><span class="nx">state</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">succeeded</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">selectedCandidatePair</span> <span class="o">=</span> <span class="nx">stats</span><span class="p">;</span>
      <span class="p">}</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">stats</span><span class="p">.</span><span class="nx">type</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">inbound-rtp</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">stats</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">video</span><span class="dl">'</span><span class="p">)</span> <span class="nx">inboundVideo</span> <span class="o">=</span> <span class="nx">stats</span><span class="p">;</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">stats</span><span class="p">.</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">audio</span><span class="dl">'</span><span class="p">)</span> <span class="nx">inboundAudio</span> <span class="o">=</span> <span class="nx">stats</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">selectedCandidatePair</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">timestamp</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">();</span>

    <span class="c1">// 2. Compute Deltas for Video</span>
    <span class="kd">const</span> <span class="nx">videoMetrics</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">computeStreamDeltas</span><span class="p">(</span><span class="nx">inboundVideo</span><span class="p">,</span> <span class="dl">'</span><span class="s1">video</span><span class="dl">'</span><span class="p">);</span>

    <span class="k">return</span> <span class="p">{</span>
      <span class="na">sessionId</span><span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">sessionId</span><span class="p">,</span>
      <span class="na">timestamp</span><span class="p">:</span> <span class="nx">timestamp</span><span class="p">,</span>
      <span class="na">rtt</span><span class="p">:</span> <span class="nx">selectedCandidatePair</span><span class="p">.</span><span class="nx">currentRoundTripTime</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span> <span class="c1">// ms</span>
      <span class="na">availableOutgoingBitrate</span><span class="p">:</span> <span class="nx">selectedCandidatePair</span><span class="p">.</span><span class="nx">availableOutgoingBitrate</span><span class="p">,</span>
      <span class="na">video</span><span class="p">:</span> <span class="nx">videoMetrics</span><span class="p">,</span>
      <span class="c1">// Audio logic omitted for brevity</span>
    <span class="p">};</span>
  <span class="p">}</span>

  <span class="nf">computeStreamDeltas</span><span class="p">(</span><span class="nx">currentStats</span><span class="p">,</span> <span class="nx">kind</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">currentStats</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">prev</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">previousStats</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">currentStats</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">previousStats</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">currentStats</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="nx">currentStats</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">prev</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span> <span class="c1">// First sample, cannot compute delta</span>

    <span class="kd">const</span> <span class="nx">timeDelta</span> <span class="o">=</span> <span class="p">(</span><span class="nx">currentStats</span><span class="p">.</span><span class="nx">timestamp</span> <span class="o">-</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">timestamp</span><span class="p">)</span> <span class="o">/</span> <span class="mi">1000</span><span class="p">;</span> <span class="c1">// seconds</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">timeDelta</span> <span class="o">&lt;=</span> <span class="mi">0</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

    <span class="c1">// 3. Key Calculations</span>
    <span class="kd">const</span> <span class="nx">packetsLostDelta</span> <span class="o">=</span> <span class="nx">currentStats</span><span class="p">.</span><span class="nx">packetsLost</span> <span class="o">-</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">packetsLost</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">packetsReceivedDelta</span> <span class="o">=</span> <span class="nx">currentStats</span><span class="p">.</span><span class="nx">packetsReceived</span> <span class="o">-</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">packetsReceived</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">totalPackets</span> <span class="o">=</span> <span class="nx">packetsLostDelta</span> <span class="o">+</span> <span class="nx">packetsReceivedDelta</span><span class="p">;</span>

    <span class="c1">// Packet Loss Fraction (0.0 to 1.0)</span>
    <span class="kd">const</span> <span class="nx">lossFraction</span> <span class="o">=</span> <span class="nx">totalPackets</span> <span class="o">&gt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="nx">packetsLostDelta</span> <span class="o">/</span> <span class="nx">totalPackets</span> <span class="p">:</span> <span class="mi">0</span><span class="p">;</span>

    <span class="c1">// Bitrate Calculation</span>
    <span class="kd">const</span> <span class="nx">bytesDelta</span> <span class="o">=</span> <span class="nx">currentStats</span><span class="p">.</span><span class="nx">bytesReceived</span> <span class="o">-</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">bytesReceived</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">bitrate</span> <span class="o">=</span> <span class="p">(</span><span class="nx">bytesDelta</span> <span class="o">*</span> <span class="mi">8</span><span class="p">)</span> <span class="o">/</span> <span class="nx">timeDelta</span><span class="p">;</span> <span class="c1">// bits per second</span>

    <span class="c1">// Freeze Detection Logic</span>
    <span class="kd">const</span> <span class="nx">framesDecodedDelta</span> <span class="o">=</span> <span class="nx">currentStats</span><span class="p">.</span><span class="nx">framesDecoded</span> <span class="o">-</span> <span class="nx">prev</span><span class="p">.</span><span class="nx">framesDecoded</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">isFrozen</span> <span class="o">=</span> <span class="p">(</span><span class="nx">kind</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">video</span><span class="dl">'</span> <span class="o">&amp;&amp;</span> <span class="nx">bitrate</span> <span class="o">&gt;</span> <span class="mi">10000</span> <span class="o">&amp;&amp;</span> <span class="nx">framesDecodedDelta</span> <span class="o">===</span> <span class="mi">0</span><span class="p">);</span>

    <span class="k">return</span> <span class="p">{</span>
      <span class="na">bitrate</span><span class="p">:</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="nx">bitrate</span><span class="p">),</span>
      <span class="na">packetLoss</span><span class="p">:</span> <span class="nf">parseFloat</span><span class="p">(</span><span class="nx">lossFraction</span><span class="p">.</span><span class="nf">toFixed</span><span class="p">(</span><span class="mi">4</span><span class="p">)),</span>
      <span class="na">jitter</span><span class="p">:</span> <span class="nx">currentStats</span><span class="p">.</span><span class="nx">jitter</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span> <span class="c1">// ms</span>
      <span class="na">framesDecodedPerSecond</span><span class="p">:</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">round</span><span class="p">(</span><span class="nx">framesDecodedDelta</span> <span class="o">/</span> <span class="nx">timeDelta</span><span class="p">),</span>
      <span class="na">isFrozen</span><span class="p">:</span> <span class="nx">isFrozen</span>
    <span class="p">};</span>
  <span class="p">}</span>

  <span class="k">async</span> <span class="nf">flush</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">buffer</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">payload</span> <span class="o">=</span> <span class="p">[...</span><span class="k">this</span><span class="p">.</span><span class="nx">buffer</span><span class="p">];</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">buffer</span> <span class="o">=</span> <span class="p">[];</span>

    <span class="c1">// Use beacon or fetch keepalive to ensure delivery on page unload</span>
    <span class="nf">fetch</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ingestionUrl</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span> <span class="p">},</span>
      <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="na">metrics</span><span class="p">:</span> <span class="nx">payload</span> <span class="p">}),</span>
      <span class="na">keepalive</span><span class="p">:</span> <span class="kc">true</span>
    <span class="p">}).</span><span class="k">catch</span><span class="p">(</span><span class="nx">e</span> <span class="o">=&gt;</span> <span class="nx">console</span><span class="p">.</span><span class="nf">warn</span><span class="p">(</span><span class="dl">"</span><span class="s2">Failed to send telemetry</span><span class="dl">"</span><span class="p">,</span> <span class="nx">e</span><span class="p">));</span>
  <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>This code does the heavy lifting: it ignores static counters and provides the backend with rate-based metrics (<code>framesDecodedPerSecond</code>, <code>bitrate</code>) and a boolean <code>isFrozen</code> flag.</p>

<h2>
  
  
  Backend Ingestion and Processing (Python)
</h2>

<p>The backend must ingest these time-series bursts efficiently. A high-performance Python framework like FastAPI is ideal here. We define a strict schema using Pydantic to validate the incoming telemetry.</p>

<p>The ingestion layer has two responsibilities:</p>

<ol>
<li>
<strong>Persist raw data</strong> to the time-series database (TSDB).</li>
<li>
<strong>Evaluate real-time alerts</strong> (e.g., if freeze count &gt; X).
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">fastapi</span> <span class="kn">import</span> <span class="n">FastAPI</span><span class="p">,</span> <span class="n">BackgroundTasks</span><span class="p">,</span> <span class="n">HTTPException</span>
<span class="kn">from</span> <span class="n">pydantic</span> <span class="kn">import</span> <span class="n">BaseModel</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">List</span><span class="p">,</span> <span class="n">Optional</span>
<span class="kn">import</span> <span class="n">time</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">FastAPI</span><span class="p">()</span>

<span class="c1"># --- Schema Definition ---
</span>
<span class="k">class</span> <span class="nc">VideoMetrics</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">bitrate</span><span class="p">:</span> <span class="nb">int</span>
    <span class="n">packetLoss</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">jitter</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">framesDecodedPerSecond</span><span class="p">:</span> <span class="nb">int</span>
    <span class="n">isFrozen</span><span class="p">:</span> <span class="nb">bool</span>

<span class="k">class</span> <span class="nc">TelemetryPoint</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">sessionId</span><span class="p">:</span> <span class="nb">str</span>
    <span class="n">timestamp</span><span class="p">:</span> <span class="nb">str</span>
    <span class="n">rtt</span><span class="p">:</span> <span class="nb">float</span>
    <span class="n">availableOutgoingBitrate</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="nb">float</span><span class="p">]</span> <span class="o">=</span> <span class="bp">None</span>
    <span class="n">video</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="n">VideoMetrics</span><span class="p">]</span> <span class="o">=</span> <span class="bp">None</span>

<span class="k">class</span> <span class="nc">TelemetryBatch</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">metrics</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="n">TelemetryPoint</span><span class="p">]</span>

<span class="c1"># --- Processing Logic ---
</span>
<span class="c1"># In-memory alert buffer (Replace with Redis in production)
</span><span class="n">alert_buffer</span> <span class="o">=</span> <span class="p">{}</span> 

<span class="k">def</span> <span class="nf">process_metrics_batch</span><span class="p">(</span><span class="n">batch</span><span class="p">:</span> <span class="n">TelemetryBatch</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    1. Write to Time-Series DB (e.g., InfluxDB, TimescaleDB)
    2. Check for freeze patterns
    </span><span class="sh">"""</span>
    <span class="k">for</span> <span class="n">point</span> <span class="ow">in</span> <span class="n">batch</span><span class="p">.</span><span class="n">metrics</span><span class="p">:</span>
        <span class="c1"># Mock DB Insertion
</span>        <span class="c1"># db.write_point("telemetry", tags={"session": point.sessionId}, fields=point.dict())
</span>
        <span class="c1"># Freeze Detection Logic
</span>        <span class="k">if</span> <span class="n">point</span><span class="p">.</span><span class="n">video</span> <span class="ow">and</span> <span class="n">point</span><span class="p">.</span><span class="n">video</span><span class="p">.</span><span class="n">isFrozen</span><span class="p">:</span>
            <span class="nf">check_alert_threshold</span><span class="p">(</span><span class="n">point</span><span class="p">.</span><span class="n">sessionId</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">check_alert_threshold</span><span class="p">(</span><span class="n">session_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    Simple hysteresis: Trigger alert if 3 consecutive freeze events occur.
    </span><span class="sh">"""</span>
    <span class="n">current_time</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span>
    <span class="n">state</span> <span class="o">=</span> <span class="n">alert_buffer</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">session_id</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="sh">"</span><span class="s">last_seen</span><span class="sh">"</span><span class="p">:</span> <span class="mi">0</span><span class="p">})</span>

    <span class="c1"># Reset if alerts are too far apart (e.g., &gt; 10 seconds)
</span>    <span class="k">if</span> <span class="n">current_time</span> <span class="o">-</span> <span class="n">state</span><span class="p">[</span><span class="sh">"</span><span class="s">last_seen</span><span class="sh">"</span><span class="p">]</span> <span class="o">&gt;</span> <span class="mi">10</span><span class="p">:</span>
        <span class="n">state</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span>

    <span class="n">state</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">+=</span> <span class="mi">1</span>
    <span class="n">state</span><span class="p">[</span><span class="sh">"</span><span class="s">last_seen</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">current_time</span>
    <span class="n">alert_buffer</span><span class="p">[</span><span class="n">session_id</span><span class="p">]</span> <span class="o">=</span> <span class="n">state</span>

    <span class="k">if</span> <span class="n">state</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">&gt;=</span> <span class="mi">3</span><span class="p">:</span>
        <span class="nf">trigger_pagerduty</span><span class="p">(</span><span class="n">session_id</span><span class="p">)</span>
        <span class="n">state</span><span class="p">[</span><span class="sh">"</span><span class="s">count</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span> <span class="c1"># Reset after alert
</span>
<span class="k">def</span> <span class="nf">trigger_pagerduty</span><span class="p">(</span><span class="n">session_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">[ALERT] CRITICAL: Sustained video freeze detected for session </span><span class="si">{</span><span class="n">session_id</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="c1"># --- API Endpoint ---
</span>
<span class="nd">@app.post</span><span class="p">(</span><span class="sh">"</span><span class="s">/v1/telemetry/ingest</span><span class="sh">"</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">ingest_telemetry</span><span class="p">(</span><span class="n">batch</span><span class="p">:</span> <span class="n">TelemetryBatch</span><span class="p">,</span> <span class="n">background_tasks</span><span class="p">:</span> <span class="n">BackgroundTasks</span><span class="p">):</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">batch</span><span class="p">.</span><span class="n">metrics</span><span class="p">:</span>
        <span class="k">raise</span> <span class="nc">HTTPException</span><span class="p">(</span><span class="n">status_code</span><span class="o">=</span><span class="mi">400</span><span class="p">,</span> <span class="n">detail</span><span class="o">=</span><span class="sh">"</span><span class="s">Empty batch</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Offload processing to background task to keep API response &lt; 20ms
</span>    <span class="n">background_tasks</span><span class="p">.</span><span class="nf">add_task</span><span class="p">(</span><span class="n">process_metrics_batch</span><span class="p">,</span> <span class="n">batch</span><span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">accepted</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">processed</span><span class="sh">"</span><span class="p">:</span> <span class="nf">len</span><span class="p">(</span><span class="n">batch</span><span class="p">.</span><span class="n">metrics</span><span class="p">)}</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="kn">import</span> <span class="n">uvicorn</span>
    <span class="n">uvicorn</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="n">app</span><span class="p">,</span> <span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">0.0.0.0</span><span class="sh">"</span><span class="p">,</span> <span class="n">port</span><span class="o">=</span><span class="mi">8000</span><span class="p">)</span>

</code></pre>

</div>



<p>This Python implementation leverages FastAPI's <code>BackgroundTasks</code> to ensure the ingestion endpoint returns immediately. We do not want telemetry upload to block the client's network thread, even if that thread is theoretically non-blocking in JS. The processing logic separates persistence from alerting.</p>

<h2>
  
  
  Storage Strategy
</h2>

<p>Relational databases are poor candidates for high-frequency telemetry. The write amplification of indexing millions of rows per hour will cripple a standard PostgreSQL instance.</p>

<p><strong>Recommended Storage Engines:</strong></p>

<ol>
<li>
<strong>TimescaleDB (PostgreSQL Extension):</strong> Excellent if you want SQL capability. It partitions tables by time (hypertables) to keep indices small and writes fast.</li>
<li>
<strong>InfluxDB:</strong> Purpose-built for high-write-volume observability.</li>
<li>
<strong>ClickHouse:</strong> If you are operating at massive scale (millions of concurrent sessions), the columnar storage of ClickHouse offers superior compression and query speed for analytics.</li>
</ol>

<p><strong>Schema Design:</strong><br>
The schema should avoid deep nesting. Flatten the critical metrics for faster aggregation.</p>

<ul>
<li>
<strong>Tags (Indexed):</strong> <code>session_id</code>, <code>room_id</code>, <code>user_id</code>, <code>platform</code> (e.g., "chrome_110"), <code>region</code>.</li>
<li>
<strong>Fields (Metrics):</strong> <code>rtt</code>, <code>jitter</code>, <code>packet_loss_pct</code>, <code>bitrate_bps</code>, <code>frames_decoded_delta</code>, <code>freeze_flag</code> (boolean/int).</li>
</ul>

<h2>
  
  
  Real-World Debugging Scenario
</h2>

<p>Consider a typical production incident. A VIP client reports: "The video froze for 20 seconds during the board meeting."</p>

<p>Without client stats, you check the server logs. The SFU logs show the WebSocket remained open (<code>connectionState: connected</code>). The bandwidth estimation logs show a drop, but not a disconnect. You are blind.</p>

<p>With the telemetry system built above, you query the specific <code>session_id</code> in your TSDB.</p>

<p><strong>The Forensic Analysis:</strong></p>

<ol>
<li>
<strong>Check Signaling:</strong> You confirm <code>connectionState</code> remained "connected." The user did not disconnect.</li>
<li>
<strong>Check Bitrate:</strong> You see <code>bitrate_bps</code> drop from 2Mbps to 50kbps at T+10:00.</li>
<li>
<strong>Check Packet Loss:</strong> Simulatneously, <code>packet_loss_pct</code> spikes to 15%.</li>
<li>
<strong>Check Freeze Flag:</strong> The <code>frames_decoded_delta</code> drops to 0 for exactly 18 seconds, triggering the <code>freeze_flag</code>.</li>
<li>
<strong>Check RTT:</strong> Crucially, you see <code>rtt</code> spike from 50ms to 800ms <em>before</em> the freeze.</li>
</ol>

<p><strong>Conclusion:</strong> The rising RTT and Packet Loss prove this was <strong>network congestion</strong> on the user's last mile (likely Wi-Fi contention), not a backend failure. The SFU reacted correctly by lowering bitrate (congestion control), but the loss was too high to sustain video. You can now confidently explain to the client that the issue lies in their local network environment, backed by hard data.</p>

<h2>
  
  
  Architectural Pattern: The Side-Channel Approach
</h2>

<p>Do not piggyback telemetry on your primary signaling WebSocket. Signaling is critical infrastructure; if your telemetry payload floods the WebSocket queue, you risk blocking "Offer/Answer" negotiation or "Candidate" exchange, causing the very failures you are trying to measure.</p>

<p><strong>The Side-Channel Pattern:</strong></p>

<ol>
<li>
<strong>Signaling Channel (WebSocket):</strong> Reserved strictly for SDP, ICE candidates, and roster updates. High priority.</li>
<li>
<strong>Telemetry Channel (HTTP/POST):</strong> A separate path for stats. Use <code>navigator.sendBeacon</code> or a low-priority <code>fetch</code> loop. If telemetry packets fail, it is acceptable; if signaling packets fail, the call drops.</li>
<li>
<strong>Aggregation Tier:</strong> A dedicated microservice (like the Python example above) that validates and queues data.</li>
<li>
<strong>Storage Tier:</strong> The Time-Series Database.</li>
<li>
<strong>Alert Engine:</strong> A worker that polls the TSDB or subscribes to the Aggregation Tier to trigger PagerDuty/Slack alerts based on freeze thresholds.</li>
</ol>

<h2>
  
  
  Conclusion
</h2>

<p>Observability in WebRTC is not about collecting logs; it is about reconstructing the user's reality. The <code>getStats</code> API is the only mechanism that bridges the gap between network physics and perceived quality. By implementing client-side delta computation, efficient ingestion pipelines, and intelligent freeze detection, engineering teams transform vague user complaints into actionable infrastructure insights. In the world of real-time video, if you aren't measuring the client, you aren't measuring anything.</p>

