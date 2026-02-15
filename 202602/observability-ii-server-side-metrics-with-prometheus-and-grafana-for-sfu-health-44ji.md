---
Title: Observability II – Server-Side Metrics with Prometheus and Grafana for SFU Health
Description: 
Author: Lalit Mishra
Date: 2026-02-15T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Opening Context – Why Client Metrics Are Not Enough
</h2>

<p>In the previous installment of this series, we established that the client is the ultimate arbiter of quality. Through <code>getStats()</code>, we can detect freezing video, robotic audio, and rising jitter. However, client-side telemetry suffers from a critical limitation: it describes symptoms, not causes. A sudden spike in packet loss across 500 users in a specific region might be a local ISP outage, or it might be a Selective Forwarding Unit (SFU) undergoing a retransmission storm due to CPU saturation. To the client, the experience is identical: degraded media. To the platform engineer, the remediation paths are diametrically opposed.</p>

<p>Client metrics are effectively lagging indicators of infrastructure health. By the time a client reports a freeze, the degradation has already occurred. To achieve five-nine reliability (99.999%), we require leading indicators—signals that predict failure before user perception is impacted.</p>

<p>Server-side observability provides this causal visibility. While the client tells you <em>that</em> a call failed, the server tells you <em>why</em>. Was the signaling node out of file descriptors? Did the SFU’s event loop stall processing DTLS handshakes? Did a NACK storm saturate the egress bandwidth? This article focuses on instrumenting the backend and media tier to expose these hidden architectural stresses, enabling a shift from reactive firefighting to proactive capacity management.</p>




<h2>
  
  
  Observability Pillars in Real-Time Systems
</h2>

<p>The standard observability triad—Metrics, Logs, and Traces—applies to WebRTC, but the implementation priority differs significantly from traditional REST APIs.</p>

<p><strong>Metrics</strong> are the lifeblood of real-time systems. They are aggregatable, cheap to store, and queryable over long time horizons. In a high-throughput SFU environment where thousands of packets are processed per second, metrics provide the only viable way to visualize aggregate health (e.g., "Total NACKs per second" or "Average ICE negotiation time"). This article focuses exclusively on metrics-based observability.</p>

<p><strong>Logs</strong> in WebRTC are prohibitively expensive for the media plane. Logging every dropped packet or retransmission request would saturate disk I/O faster than the network interface. Logs must be reserved for control-plane events: signaling errors, authentication failures, and ICE state transitions. They are reactive tools used for post-mortem analysis of specific session failures, not for real-time health monitoring.</p>

<p><strong>Tracing</strong>, while powerful in microservices, faces unique challenges in UDP-based media paths. RTP packets lack the convenient header space for trace IDs found in HTTP, and the overhead of sampling distributed traces across a mesh of media servers often outweighs the benefit.</p>

<p>Therefore, for the operational architect, the primary monitoring surface is a robust time-series metric pipeline. We will implement this using Prometheus for ingestion and storage, and Grafana for visualization and correlation.</p>




<h2>
  
  
  Instrumenting Python / Quart Backend with prometheus_client
</h2>

<p>The signaling plane is the orchestrator of WebRTC. If signaling latency increases, room joins become sluggish, and users abandon calls before media flows. We will instrument a Python Quart backend (an async super-set of Flask widely used in modern Python architectures) using the official <code>prometheus_client</code> library.</p>

<p>The goal is to expose a <code>/metrics</code> endpoint that a Prometheus scraper can poll. We need to track three distinct categories of signals:</p>

<ol>
<li>
<strong>Throughput:</strong> Active connections and message rates.</li>
<li>
<strong>Latency:</strong> Time taken to process joins, offers, and answers.</li>
<li>
<strong>Errors:</strong> Authentication failures and validation errors.</li>
</ol>

<p><strong>Production Instrumentation Example</strong></p>

<p>We structure the instrumentation as a middleware layer to keep business logic clean. We define our metrics globally to ensure they persist across the application lifecycle.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">time</span>
<span class="kn">from</span> <span class="n">quart</span> <span class="kn">import</span> <span class="n">Quart</span><span class="p">,</span> <span class="n">request</span><span class="p">,</span> <span class="n">Response</span>
<span class="kn">from</span> <span class="n">prometheus_client</span> <span class="kn">import</span> <span class="n">Counter</span><span class="p">,</span> <span class="n">Gauge</span><span class="p">,</span> <span class="n">Histogram</span><span class="p">,</span> <span class="n">generate_latest</span><span class="p">,</span> <span class="n">CONTENT_TYPE_LATEST</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">Quart</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>

<span class="c1"># --- Metric Definitions ---
</span>
<span class="c1"># Gauges: For values that go up and down (State)
</span><span class="n">ACTIVE_WEBSOCKETS</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">signaling_websockets_active</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Number of currently active WebSocket signaling connections</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">[</span><span class="sh">'</span><span class="s">region</span><span class="sh">'</span><span class="p">]</span>
<span class="p">)</span>

<span class="n">ACTIVE_ROOMS</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">signaling_rooms_active</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Number of active rooms with at least one participant</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">[</span><span class="sh">'</span><span class="s">region</span><span class="sh">'</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># Counters: For cumulative events (Throughput/Errors)
</span><span class="n">SIGNALING_MESSAGES_TOTAL</span> <span class="o">=</span> <span class="nc">Counter</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">signaling_messages_total</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Total number of signaling messages processed</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">[</span><span class="sh">'</span><span class="s">msg_type</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">direction</span><span class="sh">'</span><span class="p">]</span> <span class="c1"># direction: inbound/outbound
</span><span class="p">)</span>

<span class="n">AUTH_FAILURES_TOTAL</span> <span class="o">=</span> <span class="nc">Counter</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">signaling_auth_failures_total</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Total number of authentication failures</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">[</span><span class="sh">'</span><span class="s">reason</span><span class="sh">'</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># Histograms: For distribution of duration (Latency)
# Buckets optimized for sub-second signaling operations
</span><span class="n">ICE_NEGOTIATION_DURATION</span> <span class="o">=</span> <span class="nc">Histogram</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">signaling_ice_negotiation_seconds</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Time taken from Offer to ICE Connected state</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">[</span><span class="sh">'</span><span class="s">status</span><span class="sh">'</span><span class="p">],</span>
    <span class="n">buckets</span><span class="o">=</span><span class="p">[</span><span class="mf">0.05</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">,</span> <span class="mf">0.25</span><span class="p">,</span> <span class="mf">0.5</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">,</span> <span class="mf">2.5</span><span class="p">,</span> <span class="mf">5.0</span><span class="p">,</span> <span class="mf">10.0</span><span class="p">]</span>
<span class="p">)</span>

<span class="n">REQUEST_PROCESSING_TIME</span> <span class="o">=</span> <span class="nc">Histogram</span><span class="p">(</span>
    <span class="sh">'</span><span class="s">http_request_duration_seconds</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">HTTP request latency</span><span class="sh">'</span><span class="p">,</span>
    <span class="p">[</span><span class="sh">'</span><span class="s">method</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">endpoint</span><span class="sh">'</span><span class="p">],</span>
    <span class="n">buckets</span><span class="o">=</span><span class="p">[</span><span class="mf">0.01</span><span class="p">,</span> <span class="mf">0.05</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">,</span> <span class="mf">0.5</span><span class="p">,</span> <span class="mf">1.0</span><span class="p">]</span>
<span class="p">)</span>

<span class="c1"># --- Middleware &amp; Endpoints ---
</span>
<span class="nd">@app.before_request</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">start_timer</span><span class="p">():</span>
    <span class="n">request</span><span class="p">.</span><span class="n">start_time</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span>

<span class="nd">@app.after_request</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">record_metrics</span><span class="p">(</span><span class="n">response</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">request</span><span class="p">.</span><span class="n">endpoint</span> <span class="o">==</span> <span class="sh">'</span><span class="s">metrics</span><span class="sh">'</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">response</span>

    <span class="n">latency</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span> <span class="o">-</span> <span class="n">request</span><span class="p">.</span><span class="n">start_time</span>
    <span class="n">REQUEST_PROCESSING_TIME</span><span class="p">.</span><span class="nf">labels</span><span class="p">(</span>
        <span class="n">method</span><span class="o">=</span><span class="n">request</span><span class="p">.</span><span class="n">method</span><span class="p">,</span> 
        <span class="n">endpoint</span><span class="o">=</span><span class="n">request</span><span class="p">.</span><span class="n">endpoint</span>
    <span class="p">).</span><span class="nf">observe</span><span class="p">(</span><span class="n">latency</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">response</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">'</span><span class="s">/metrics</span><span class="sh">'</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">metrics</span><span class="p">():</span>
    <span class="c1"># Expose the standard Prometheus metrics endpoint
</span>    <span class="k">return</span> <span class="nc">Response</span><span class="p">(</span><span class="nf">generate_latest</span><span class="p">(),</span> <span class="n">mimetype</span><span class="o">=</span><span class="n">CONTENT_TYPE_LATEST</span><span class="p">)</span>

<span class="c1"># --- WebSocket Handler Example ---
</span>
<span class="nd">@app.websocket</span><span class="p">(</span><span class="sh">'</span><span class="s">/ws/signaling</span><span class="sh">'</span><span class="p">)</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">signaling_socket</span><span class="p">():</span>
    <span class="n">ACTIVE_WEBSOCKETS</span><span class="p">.</span><span class="nf">labels</span><span class="p">(</span><span class="n">region</span><span class="o">=</span><span class="sh">'</span><span class="s">us-east-1</span><span class="sh">'</span><span class="p">).</span><span class="nf">inc</span><span class="p">()</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
            <span class="n">data</span> <span class="o">=</span> <span class="k">await</span> <span class="n">websocket</span><span class="p">.</span><span class="nf">receive</span><span class="p">()</span>
            <span class="n">msg_type</span> <span class="o">=</span> <span class="nf">parse_message_type</span><span class="p">(</span><span class="n">data</span><span class="p">)</span> <span class="c1"># Hypothetical helper
</span>
            <span class="n">SIGNALING_MESSAGES_TOTAL</span><span class="p">.</span><span class="nf">labels</span><span class="p">(</span>
                <span class="n">msg_type</span><span class="o">=</span><span class="n">msg_type</span><span class="p">,</span> 
                <span class="n">direction</span><span class="o">=</span><span class="sh">'</span><span class="s">inbound</span><span class="sh">'</span>
            <span class="p">).</span><span class="nf">inc</span><span class="p">()</span>

            <span class="c1"># Business logic processing...
</span>            <span class="k">await</span> <span class="nf">process_signal</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>

    <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
        <span class="c1"># Log error
</span>        <span class="k">pass</span>
    <span class="k">finally</span><span class="p">:</span>
        <span class="n">ACTIVE_WEBSOCKETS</span><span class="p">.</span><span class="nf">labels</span><span class="p">(</span><span class="n">region</span><span class="o">=</span><span class="sh">'</span><span class="s">us-east-1</span><span class="sh">'</span><span class="p">).</span><span class="nf">dec</span><span class="p">()</span>

<span class="c1"># --- Simulation of ICE tracking ---
</span>
<span class="k">async</span> <span class="k">def</span> <span class="nf">handle_ice_completion</span><span class="p">(</span><span class="n">session_id</span><span class="p">,</span> <span class="n">start_time</span><span class="p">):</span>
    <span class="n">duration</span> <span class="o">=</span> <span class="n">time</span><span class="p">.</span><span class="nf">time</span><span class="p">()</span> <span class="o">-</span> <span class="n">start_time</span>
    <span class="n">ICE_NEGOTIATION_DURATION</span><span class="p">.</span><span class="nf">labels</span><span class="p">(</span><span class="n">status</span><span class="o">=</span><span class="sh">'</span><span class="s">success</span><span class="sh">'</span><span class="p">).</span><span class="nf">observe</span><span class="p">(</span><span class="n">duration</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">app</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="mi">5000</span><span class="p">)</span>

</code></pre>

</div>



<p><strong>Key Architectural Considerations:</strong></p>

<ul>
<li>
<strong>Label Cardinality:</strong> Notice we use <code>region</code> and <code>msg_type</code> as labels. <em>Never</em> use <code>user_id</code> or <code>room_id</code> as a label. High cardinality labels explode the time-series database (TSDB) index, causing Prometheus to consume excessive memory and crash.</li>
<li>
<strong>Histogram Buckets:</strong> The default Prometheus buckets are optimized for general web traffic (up to 10s). For signaling, we care about the 50ms to 500ms range. We explicitly redefine buckets to capture the granularity of "fast" vs "slow" negotiations.</li>
</ul>




<h2>
  
  
  Monitoring SFU Health – Janus and Mediasoup
</h2>

<p>The signaling server tells us about user intent; the SFU (Selective Forwarding Unit) tells us about media reality. Whether you use Janus, Mediasoup, or Jitsi, the SFU is a black box that consumes CPU to route UDP packets.</p>

<p>Most SFUs do not expose a <code>/metrics</code> endpoint natively in Prometheus format. They usually offer an Admin API (Janus) or an Observer/Stats API (Mediasoup) that returns JSON. We must implement the "Exporter Pattern"—a lightweight sidecar service that polls the SFU and translates JSON stats into Prometheus metrics.</p>

<p><strong>The "Leading Indicators" of SFU Failure</strong></p>

<p>Before an SFU crashes or drops packets, specific metrics spike:</p>

<ol>
<li>
<strong>NACK Count (Negative Acknowledgement):</strong> A receiver requests a packet retransmission. A spike here indicates network congestion.</li>
<li>
<strong>PLI/FIR Count (Picture Loss Indication):</strong> A receiver requests a full keyframe. This is expensive. If this spikes, the encoder is working overtime, driving up CPU.</li>
<li>
<strong>Event Loop Lag:</strong> If the SFU is single-threaded (like generic Janus plugins) or uses a worker-per-core model (Mediasoup), measuring the delay in the event loop reveals CPU saturation before top/htop does.</li>
</ol>

<p><strong>Python Exporter for Janus (Conceptual Implementation)</strong></p>

<p>This script runs alongside the Janus instance, polling its Admin API every 5 seconds.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">time</span>
<span class="kn">import</span> <span class="n">requests</span>
<span class="kn">from</span> <span class="n">prometheus_client</span> <span class="kn">import</span> <span class="n">start_http_server</span><span class="p">,</span> <span class="n">Gauge</span>

<span class="c1"># Define SFU Metrics
</span><span class="n">SFU_BITRATE_IN</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span><span class="sh">'</span><span class="s">sfu_bitrate_in_bits</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Ingress bitrate</span><span class="sh">'</span><span class="p">)</span>
<span class="n">SFU_BITRATE_OUT</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span><span class="sh">'</span><span class="s">sfu_bitrate_out_bits</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Egress bitrate</span><span class="sh">'</span><span class="p">)</span>
<span class="n">SFU_NACKS_TOTAL</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span><span class="sh">'</span><span class="s">sfu_nacks_total</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Total NACKs received</span><span class="sh">'</span><span class="p">)</span>
<span class="n">SFU_CPU_USAGE</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span><span class="sh">'</span><span class="s">sfu_cpu_usage_percent</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">SFU Process CPU Usage</span><span class="sh">'</span><span class="p">)</span>
<span class="n">SFU_HANDLES</span> <span class="o">=</span> <span class="nc">Gauge</span><span class="p">(</span><span class="sh">'</span><span class="s">sfu_active_handles</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">Number of active WebRTC handles</span><span class="sh">'</span><span class="p">)</span>

<span class="n">JANUS_ADMIN_URL</span> <span class="o">=</span> <span class="sh">"</span><span class="s">http://localhost:7088/admin</span><span class="sh">"</span>
<span class="n">ADMIN_SECRET</span> <span class="o">=</span> <span class="sh">"</span><span class="s">janusoverlord</span><span class="sh">"</span>

<span class="k">def</span> <span class="nf">fetch_janus_stats</span><span class="p">():</span>
    <span class="n">payload</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">janus</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">get_status</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">transaction</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">monitor_req</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">admin_secret</span><span class="sh">"</span><span class="p">:</span> <span class="n">ADMIN_SECRET</span>
    <span class="p">}</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="n">JANUS_ADMIN_URL</span><span class="p">,</span> <span class="n">json</span><span class="o">=</span><span class="n">payload</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span>
        <span class="n">data</span> <span class="o">=</span> <span class="n">response</span><span class="p">.</span><span class="nf">json</span><span class="p">()</span>

        <span class="k">if</span> <span class="sh">"</span><span class="s">data</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">data</span><span class="p">:</span>
            <span class="c1"># Janus exposes active handles (roughly equivalent to peer connections)
</span>            <span class="n">SFU_HANDLES</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="n">data</span><span class="p">[</span><span class="sh">"</span><span class="s">data</span><span class="sh">"</span><span class="p">].</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">sessions</span><span class="sh">"</span><span class="p">,</span> <span class="mi">0</span><span class="p">))</span>

            <span class="c1"># Note: Detailed stream stats often require iterating over sessions
</span>            <span class="c1"># In production, you might query specific loop stats or use the event handler
</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Error scraping Janus: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">main</span><span class="p">():</span>
    <span class="c1"># Start Prometheus exporter server on port 8000
</span>    <span class="nf">start_http_server</span><span class="p">(</span><span class="mi">8000</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">SFU Exporter running on :8000</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="nf">fetch_janus_stats</span><span class="p">()</span>
        <span class="c1"># In a real exporter, we would also read /proc or use psutil 
</span>        <span class="c1"># to get the specific CPU usage of the SFU process.
</span>        <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">5</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="nf">main</span><span class="p">()</span>

</code></pre>

</div>



<p>For <strong>Mediasoup</strong>, the approach is slightly different. Since Mediasoup runs as a Node.js library or Rust worker, you typically hook into the <code>worker.on('newrouter')</code> and <code>transport.on('trace')</code> events to push metrics to a central aggregator rather than polling.</p>




<h2>
  
  
  Prometheus Architecture and Scrape Strategy
</h2>

<p>Prometheus uses a <strong>pull model</strong>. It wakes up at a configured interval, reaches out to your targets (Signaling API, SFU Exporter), and retrieves the current state of all metrics.</p>

<h3>
  
  
  Scrape Configuration (<code>prometheus.yml</code>)
</h3>

<p>The configuration defines <em>who</em> to scrape and <em>how often</em>. For WebRTC, a 15-second scrape interval is standard. 1 minute is too slow (media degrades in seconds); 1 second is too heavy for the TSDB.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">global</span><span class="pi">:</span>
  <span class="na">scrape_interval</span><span class="pi">:</span> <span class="s">15s</span>
  <span class="na">evaluation_interval</span><span class="pi">:</span> <span class="s">15s</span>

<span class="na">scrape_configs</span><span class="pi">:</span>
  <span class="c1"># 1. Signaling Tier</span>
  <span class="pi">-</span> <span class="na">job_name</span><span class="pi">:</span> <span class="s1">'</span><span class="s">signaling_backend'</span>
    <span class="na">static_configs</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">targets</span><span class="pi">:</span> <span class="pi">[</span><span class="s1">'</span><span class="s">10.0.1.5:5000'</span><span class="pi">,</span> <span class="s1">'</span><span class="s">10.0.1.6:5000'</span><span class="pi">]</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="na">service</span><span class="pi">:</span> <span class="s1">'</span><span class="s">signaling-quart'</span>
      <span class="na">env</span><span class="pi">:</span> <span class="s1">'</span><span class="s">production'</span>

  <span class="c1"># 2. Media Tier (SFU Exporters)</span>
  <span class="pi">-</span> <span class="na">job_name</span><span class="pi">:</span> <span class="s1">'</span><span class="s">sfu_nodes'</span>
    <span class="na">static_configs</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">targets</span><span class="pi">:</span> <span class="pi">[</span><span class="s1">'</span><span class="s">10.0.2.10:8000'</span><span class="pi">,</span> <span class="s1">'</span><span class="s">10.0.2.11:8000'</span><span class="pi">]</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="na">service</span><span class="pi">:</span> <span class="s1">'</span><span class="s">janus-gateway'</span>
      <span class="na">region</span><span class="pi">:</span> <span class="s1">'</span><span class="s">us-east-1'</span>

  <span class="c1"># 3. Infrastructure (Node Exporter)</span>
  <span class="pi">-</span> <span class="na">job_name</span><span class="pi">:</span> <span class="s1">'</span><span class="s">node_exporter'</span>
    <span class="na">static_configs</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">targets</span><span class="pi">:</span> <span class="pi">[</span><span class="s1">'</span><span class="s">10.0.2.10:9100'</span><span class="pi">,</span> <span class="s1">'</span><span class="s">10.0.2.11:9100'</span><span class="pi">]</span>

</code></pre>

</div>



<p><strong>Federation Strategy:</strong><br>
In a global WebRTC deployment, you cannot have a single Prometheus instance scraping servers in Tokyo, Frankfurt, and Virginia. The latency is too high. The architecture requires a <strong>Prometheus instance in each region</strong> scraping local targets. A central "Federated Prometheus" or a solution like Thanos/Cortex then aggregates specific high-level metrics (e.g., "Total Users Global") from the regional instances for a single-pane-of-glass view.</p>


<h2>
  
  
  Grafana Dashboards – Correlating Infrastructure and Media Health
</h2>

<p>Grafana is where metrics become insights. The power of Grafana lies in correlation—overlaying distinct metrics to find causality.</p>
<h3>
  
  
  Dashboard Design Philosophy
</h3>

<p>A WebRTC Health Dashboard should not just list stats. It should visually group related failure domains.</p>

<p><strong>Panel 1: The "Is it Broken?" Graph (The Symptom)</strong></p>

<ul>
<li>
<strong>Metric:</strong> <code>rate(signaling_auth_failures_total[1m])</code> vs <code>active_rooms</code>.</li>
<li>
<strong>Insight:</strong> If failures spike while rooms drop, the platform is down.</li>
</ul>

<p><strong>Panel 2: The "Why is it Broken?" Correlation (The Cause)</strong></p>

<ul>
<li>
<strong>Left Y-Axis:</strong> CPU Usage (%)</li>
<li>
<strong>Right Y-Axis:</strong> Bitrate Throughput (Mbps)</li>
<li>Let’s assume CPU spikes to 90% but bitrate stays flat. This indicates the SFU is processing logic (perhaps DTLS handshakes or infinite loops) rather than forwarding packets.</li>
<li>If Bitrate spikes and CPU spikes, it is a capacity issue (Autoscaling needed).</li>
</ul>

<p><strong>Panel 3: The "Network vs. Server" Check</strong></p>

<ul>
<li>
<strong>Metric:</strong> <code>rate(sfu_nacks_total[1m])</code>
</li>
<li>A NACK storm is a classic "death spiral." As the server struggles to retransmit packets, it consumes more CPU and bandwidth, causing more packets to drop, triggering more NACKs. Visualizing NACK rate alongside CPU usage confirms this diagnosis.</li>
</ul>
<h4>
  
  
  Essential PromQL Queries
</h4>

<p><strong>1. 95th Percentile Signaling Latency</strong><br>
Calculating the "long tail" of user wait times.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>histogram_quantile(0.95, sum(rate(signaling_ice_negotiation_seconds_bucket[5m])) by (le))

</code></pre>

</div>



<p><strong>2. SFU Saturation Index</strong><br>
Detecting when specific SFU cores are overloaded (crucial for single-threaded SFUs).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)

</code></pre>

</div>



<p><strong>3. Bitrate per User (Quality Proxy)</strong><br>
If the total bitrate is stable but user count doubles, the average quality per user has halved.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sum(rate(sfu_bitrate_out_bits[1m])) / sum(sfu_active_handles)

</code></pre>

</div>






<h2>
  
  
  Alerting Strategy – From Reactive to Proactive
</h2>

<p>Alerts should wake you up for imminent failures, not minor fluctuations. We use <strong>Alertmanager</strong> to define rules based on the PromQL queries above.</p>

<h3>
  
  
  Example Alert Rules (<code>alerts.yml</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">groups</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">sfu_health</span>
  <span class="na">rules</span><span class="pi">:</span>
  <span class="c1"># Rule 1: CPU Saturation Warning</span>
  <span class="c1"># Trigger if CPU &gt; 80% for more than 2 minutes.</span>
  <span class="pi">-</span> <span class="na">alert</span><span class="pi">:</span> <span class="s">SFUHighCPU</span>
    <span class="na">expr</span><span class="pi">:</span> <span class="s">100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) &gt; </span><span class="m">80</span>
    <span class="na">for</span><span class="pi">:</span> <span class="s">2m</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="na">severity</span><span class="pi">:</span> <span class="s">warning</span>
    <span class="na">annotations</span><span class="pi">:</span>
      <span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">SFU</span><span class="nv"> </span><span class="s">instance</span><span class="nv"> </span><span class="s">{{</span><span class="nv"> </span><span class="s">$labels.instance</span><span class="nv"> </span><span class="s">}}</span><span class="nv"> </span><span class="s">is</span><span class="nv"> </span><span class="s">under</span><span class="nv"> </span><span class="s">high</span><span class="nv"> </span><span class="s">load"</span>

  <span class="c1"># Rule 2: NACK Storm Detection (Leading Indicator)</span>
  <span class="c1"># Trigger if NACKs increase by 500% compared to 5 minutes ago.</span>
  <span class="pi">-</span> <span class="na">alert</span><span class="pi">:</span> <span class="s">SFUNackStorm</span>
    <span class="na">expr</span><span class="pi">:</span> <span class="s">rate(sfu_nacks_total[1m]) &gt; 5 * (rate(sfu_nacks_total[5m] offset 10m))</span>
    <span class="na">for</span><span class="pi">:</span> <span class="s">1m</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="na">severity</span><span class="pi">:</span> <span class="s">critical</span>
    <span class="na">annotations</span><span class="pi">:</span>
      <span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">NACK</span><span class="nv"> </span><span class="s">storm</span><span class="nv"> </span><span class="s">detected</span><span class="nv"> </span><span class="s">on</span><span class="nv"> </span><span class="s">{{</span><span class="nv"> </span><span class="s">$labels.instance</span><span class="nv"> </span><span class="s">}}.</span><span class="nv"> </span><span class="s">Network</span><span class="nv"> </span><span class="s">congestion</span><span class="nv"> </span><span class="s">imminent."</span>

  <span class="c1"># Rule 3: Zero Bitrate (Lagging Indicator)</span>
  <span class="c1"># Trigger if users are present but no bits are flowing.</span>
  <span class="pi">-</span> <span class="na">alert</span><span class="pi">:</span> <span class="s">SFUSilentFailure</span>
    <span class="na">expr</span><span class="pi">:</span> <span class="s">sfu_active_handles &gt; 0 and rate(sfu_bitrate_out_bits[1m]) == </span><span class="m">0</span>
    <span class="na">for</span><span class="pi">:</span> <span class="s">1m</span>
    <span class="na">labels</span><span class="pi">:</span>
      <span class="na">severity</span><span class="pi">:</span> <span class="s">page</span>
    <span class="na">annotations</span><span class="pi">:</span>
      <span class="na">summary</span><span class="pi">:</span> <span class="s2">"</span><span class="s">SFU</span><span class="nv"> </span><span class="s">{{</span><span class="nv"> </span><span class="s">$labels.instance</span><span class="nv"> </span><span class="s">}}</span><span class="nv"> </span><span class="s">has</span><span class="nv"> </span><span class="s">users</span><span class="nv"> </span><span class="s">but</span><span class="nv"> </span><span class="s">zero</span><span class="nv"> </span><span class="s">throughput."</span>

</code></pre>

</div>



<p><strong>Noise Reduction:</strong><br>
Use the <code>for</code> clause liberally. Real-time networks are jittery. A 10-second spike in NACKs is common; a 1-minute sustained spike is an incident. The <code>offset</code> modifier in Rule 2 is a powerful technique for detecting anomalies relative to a baseline rather than setting arbitrary static thresholds.</p>




<h2>
  
  
  Real Incident Walkthrough
</h2>

<p><strong>Scenario:</strong> It is 2:00 AM. Alerts fire for <code>SFUHighCPU</code> on <code>sfu-node-04</code>.</p>

<p><strong>Step 1: Triage (Grafana)</strong><br>
The on-call engineer opens the Grafana dashboard. They see <code>sfu-node-04</code> CPU at 95%. Other nodes are at 20%. This immediately rules out a global code bug or a region-wide outage. It is a "hotspot" issue.</p>

<p><strong>Step 2: Correlation</strong><br>
Looking at the "Active Rooms" panel, <code>sfu-node-04</code> is hosting a "mega-room" with 500 participants, while other nodes have 50 rooms with 2 participants each. The load balancer failed to distribute based on <em>participant</em> count and only distributed based on <em>room</em> count.</p>

<p><strong>Step 3: The NACK Spiral</strong><br>
The "NACK Rate" graph for <code>sfu-node-04</code> is vertical. The CPU is maxed out trying to retransmit packets for 500 users. Packet loss is increasing, causing more NACKs.</p>

<p><strong>Step 4: Resolution</strong><br>
Observability identified the root cause (uneven distribution) in minutes. The engineer manually drains the node or triggers a script to migrate the room (if the architecture supports cascading SFUs).</p>

<p><strong>Step 5: Post-Mortem</strong><br>
The fix is not just restarting the server. It is updating the load balancer logic to use the custom metric <code>sfu_active_handles</code> exposed by our exporter, rather than just round-robin. Observability closed the feedback loop.</p>




<h2>
  
  
  Architectural Reference Design
</h2>

<p>To build a production-grade WebRTC platform, your observability stack must be as robust as your media stack.</p>

<p>The reference architecture flows as follows:</p>

<ol>
<li>
<strong>Metric Sources:</strong>

<ul>
<li>
<strong>Quart Backend:</strong> Exposes <code>/metrics</code> via <code>prometheus_client</code>.</li>
<li>
<strong>SFU Nodes:</strong> Sidecar Exporters translate internal JSON stats to Prometheus format.</li>
<li>
<strong>Infrastructure:</strong> Standard <code>node_exporter</code> runs on every VM to capture raw CPU, Memory, and Disk I/O.</li>
</ul>
</li>
<li>
<strong>Ingestion:</strong>

<ul>
<li>
<strong>Regional Prometheus:</strong> Scrapes local targets every 15s.</li>
</ul>
</li>
<li>
<strong>Storage &amp; Aggregation:</strong>

<ul>
<li>
<strong>Thanos / Cortex:</strong> Aggregates data from Regional Prometheus instances for long-term storage and global querying.</li>
</ul>
</li>
<li>
<strong>Visualization:</strong>

<ul>
<li>
<strong>Grafana:</strong> Connects to Thanos/Prometheus. Uses templated dashboards to switch between "Global View" and "Per-Node View."</li>
</ul>
</li>
<li>
<strong>Alerting:</strong>

<ul>
<li>
<strong>Alertmanager:</strong> Deduplicates alerts (don't send 50 emails for 50 failing nodes; send one "Cluster Critical" email) and routes them to PagerDuty or Slack.</li>
</ul>
</li>
</ol>

<p>This architecture ensures that when a user complains about video quality, you are not guessing. You are navigating a data-rich map of your system’s internal physics. In WebRTC, you cannot fix what you cannot measure. Without server-side metrics, you are flying blind; with them, you are navigating with instruments.</p>




<h2>
  
  
  Conclusion:
</h2>

<p>Server-side observability transforms WebRTC operations from reactive troubleshooting to predictive engineering. By instrumenting signaling layers, exporting SFU health, and correlating infrastructure metrics in Prometheus and Grafana, you expose leading indicators—CPU saturation, NACK storms, negotiation latency—that surface failure before users feel it. Reliability at five-nines is not luck; it is disciplined measurement, intelligent alerting, and architectural feedback loops. When metrics guide scaling, balancing, and capacity planning, incidents become data points instead of disasters. Build dashboards that explain causality, alerts that respect context, and systems that speak before they break.</p>

<p>For more in-depth system design insights, explore <a href="https://www.youtube.com/@lalit_096/videos" rel="noopener noreferrer">The Lalit Official</a> on YouTube.</p>

