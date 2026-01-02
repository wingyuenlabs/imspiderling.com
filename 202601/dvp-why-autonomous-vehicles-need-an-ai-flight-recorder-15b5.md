---
Title: DVP: Why Autonomous Vehicles Need an AI Flight Recorder
Description: 
Author: VeritasChain Standards Organization (VSO)
Date: 2026-01-02T21:52:55.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>When a plane crashes, we have the black box. When an autonomous vehicle makes a fatal decision, what do we have?</strong></p>

<ul>
<li>
<strong>DVP - Driving Vehicle Protocol Specification</strong>: <a href="https://veritaschain.org/vap/dvp" rel="noopener noreferrer">DVP - Driving Vehicle Protocol</a>
</li>
</ul>

<p>The answer, in most cases, is <em>proprietary data locked in a vendor's silo</em> — if it exists at all.</p>

<p>As we deploy AI systems that make life-or-death decisions on public roads, the gap between aviation's century of accountability standards and the automotive industry's "trust us" approach becomes increasingly untenable.</p>

<p>This article introduces the <strong>Driving Vehicle Protocol (DVP)</strong> — an open standard for creating tamper-evident audit trails of AI decision-making in autonomous vehicles. Think of it as bringing the discipline of aviation's flight recorders to the age of self-driving cars.</p>




<h2>
  
  
  The Problem: AI Decisions Without Receipts
</h2>

<h3>
  
  
  Current State of AV Data Logging
</h3>

<p>Today's autonomous vehicles generate terabytes of data per hour. LiDAR point clouds. Camera feeds. Radar returns. But here's the critical gap:</p>

<blockquote>
<p><strong>There's no standardized, tamper-evident record of what the AI <em>decided</em> and <em>why</em>.</strong></p>
</blockquote>

<p>Each manufacturer implements their own logging format (if any). When incidents occur, investigations depend on:</p>

<ul>
<li>Vendor cooperation</li>
<li>Proprietary data formats</li>
<li>Trust that logs haven't been modified</li>
</ul>

<p>This is fundamentally different from aviation, where flight data recorders follow strict international standards (ED-112A, TSO-C124).</p>

<h3>
  
  
  Why This Matters in 2025
</h3>

<p><strong>Regulatory pressure is mounting:</strong></p>

<ul>
<li>
<strong>UNECE WP.29</strong> is actively developing UN GTR (Global Technical Regulations) for Automated Driving Systems, with work continuing through 2026</li>
<li>
<strong>EU AI Act</strong> classifies autonomous vehicles as "high-risk AI systems" under Annex III, requiring comprehensive logging (Article 12)</li>
<li>
<strong>ISO 21448 (SOTIF)</strong> establishes safety frameworks but lacks standardized audit trail specifications</li>
</ul>

<p><strong>The liability landscape is shifting:</strong></p>

<p>When a Level 3+ ADS is engaged, the manufacturer — not the driver — bears responsibility. Proving what the system did (or didn't do) becomes legally critical.</p>




<h2>
  
  
  Enter DVP: The Driving Vehicle Protocol
</h2>

<p>DVP is an <strong>industry application profile</strong> within the broader <strong>VAP (Verifiable AI Provenance) Framework</strong> — the same framework that produced VCP for algorithmic trading.</p>

<h3>
  
  
  Core Design Philosophy
</h3>

<p>The fundamental insight: <strong>You don't need to standardize everything — just the audit interface.</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌─────────────────────────────────────────┐
│  Proprietary (Competition Zone)          │
│  ─────────────────────────────────────  │
│  • Sensor fusion algorithms              │
│  • Neural network architectures          │
│  • Path planning implementations         │
│  • Internal data formats                 │
└─────────────────────────────────────────┘
                    ↓ 
            Minimal Audit Interface
                    ↓
┌─────────────────────────────────────────┐
│  DVP Common Header (Audit Zone)          │
│  ─────────────────────────────────────  │
│  ① timestamp     - When                 │
│  ② event_type    - What category        │
│  ③ action        - What happened        │
│  ④ model_id      - Which AI model       │
│  ⑤ prev_hash     - Cryptographic chain  │
└─────────────────────────────────────────┘
</code></pre>

</div>



<p>This separation is critical. It means:</p>

<ul>
<li>No exposure of competitive IP</li>
<li>Minimal integration overhead</li>
<li>Universal auditability</li>
</ul>




<h2>
  
  
  Technical Architecture
</h2>

<h3>
  
  
  Event Types Registry
</h3>

<p>DVP defines a standardized taxonomy for autonomous driving events:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">enum</span> <span class="kn">import</span> <span class="n">Enum</span>

<span class="k">class</span> <span class="nc">DVPEventType</span><span class="p">(</span><span class="n">Enum</span><span class="p">):</span>
    <span class="c1"># Perception Events
</span>    <span class="n">PERCEPTION_OBJECT_DETECTED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PRC_OBJ</span><span class="sh">"</span>
    <span class="n">PERCEPTION_LANE_DETECTED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PRC_LAN</span><span class="sh">"</span>
    <span class="n">PERCEPTION_SIGN_DETECTED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PRC_SGN</span><span class="sh">"</span>
    <span class="n">PERCEPTION_CONFIDENCE_LOW</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PRC_LOW</span><span class="sh">"</span>

    <span class="c1"># Planning Events
</span>    <span class="n">PLANNING_PATH_GENERATED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PLN_PTH</span><span class="sh">"</span>
    <span class="n">PLANNING_MANEUVER_DECIDED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PLN_MNV</span><span class="sh">"</span>
    <span class="n">PLANNING_ROUTE_CHANGED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">PLN_RTE</span><span class="sh">"</span>

    <span class="c1"># Control Commands
</span>    <span class="n">CONTROL_ACCELERATION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">CTL_ACC</span><span class="sh">"</span>
    <span class="n">CONTROL_BRAKE</span> <span class="o">=</span> <span class="sh">"</span><span class="s">CTL_BRK</span><span class="sh">"</span>
    <span class="n">CONTROL_STEERING</span> <span class="o">=</span> <span class="sh">"</span><span class="s">CTL_STR</span><span class="sh">"</span>
    <span class="n">CONTROL_EMERGENCY</span> <span class="o">=</span> <span class="sh">"</span><span class="s">CTL_EMG</span><span class="sh">"</span>

    <span class="c1"># System Events
</span>    <span class="n">SYSTEM_MODE_CHANGE</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SYS_MOD</span><span class="sh">"</span>
    <span class="n">SYSTEM_TAKEOVER_REQUEST</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SYS_TOR</span><span class="sh">"</span>
    <span class="n">SYSTEM_FALLBACK_INITIATED</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SYS_FLB</span><span class="sh">"</span>
    <span class="n">SYSTEM_ODD_BOUNDARY</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SYS_ODD</span><span class="sh">"</span>  <span class="c1"># Operational Design Domain
</span></code></pre>

</div>



<h3>
  
  
  The Hash Chain: Tamper-Evidence
</h3>

<p>Like its sibling protocol VCP (for finance), DVP uses hash chains to ensure tamper-evidence:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">hashlib</span>
<span class="kn">import</span> <span class="n">json</span>
<span class="kn">from</span> <span class="n">nacl.signing</span> <span class="kn">import</span> <span class="n">SigningKey</span>
<span class="kn">from</span> <span class="n">datetime</span> <span class="kn">import</span> <span class="n">datetime</span><span class="p">,</span> <span class="n">timezone</span>
<span class="kn">import</span> <span class="n">uuid</span>

<span class="k">class</span> <span class="nc">DVPEventLogger</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">
    DVP-compliant event logger for autonomous driving systems.
    Maintains cryptographic hash chain for tamper-evidence.
    </span><span class="sh">"""</span>

    <span class="n">GENESIS_HASH</span> <span class="o">=</span> <span class="sh">"</span><span class="s">0</span><span class="sh">"</span> <span class="o">*</span> <span class="mi">64</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">signing_key</span><span class="p">:</span> <span class="n">SigningKey</span><span class="p">,</span> <span class="n">vehicle_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">model_id</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">signing_key</span> <span class="o">=</span> <span class="n">signing_key</span>
        <span class="n">self</span><span class="p">.</span><span class="n">vehicle_id</span> <span class="o">=</span> <span class="n">vehicle_id</span>
        <span class="n">self</span><span class="p">.</span><span class="n">model_id</span> <span class="o">=</span> <span class="n">model_id</span>
        <span class="n">self</span><span class="p">.</span><span class="n">prev_hash</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">GENESIS_HASH</span>

    <span class="k">def</span> <span class="nf">_canonical_json</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">obj</span><span class="p">:</span> <span class="nb">dict</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">bytes</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">RFC 8785 compliant JSON canonicalization</span><span class="sh">"""</span>
        <span class="k">return</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span>
            <span class="n">obj</span><span class="p">,</span>
            <span class="n">sort_keys</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
            <span class="n">separators</span><span class="o">=</span><span class="p">(</span><span class="sh">'</span><span class="s">,</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">:</span><span class="sh">'</span><span class="p">),</span>
            <span class="n">ensure_ascii</span><span class="o">=</span><span class="bp">False</span>
        <span class="p">).</span><span class="nf">encode</span><span class="p">(</span><span class="sh">'</span><span class="s">utf-8</span><span class="sh">'</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">log_event</span><span class="p">(</span>
        <span class="n">self</span><span class="p">,</span>
        <span class="n">event_type</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
        <span class="n">action</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span>
        <span class="n">sensor_inputs</span><span class="p">:</span> <span class="nb">dict</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
        <span class="n">decision_factors</span><span class="p">:</span> <span class="nb">dict</span> <span class="o">=</span> <span class="bp">None</span><span class="p">,</span>
        <span class="n">confidence</span><span class="p">:</span> <span class="nb">float</span> <span class="o">=</span> <span class="bp">None</span>
    <span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
        <span class="sh">"""</span><span class="s">
        Create a DVP-compliant event record with hash chain linkage.

        Returns:
            Complete DVP event record with cryptographic signature
        </span><span class="sh">"""</span>
        <span class="c1"># UUIDv7 ensures temporal ordering
</span>        <span class="n">event_id</span> <span class="o">=</span> <span class="nf">str</span><span class="p">(</span><span class="n">uuid</span><span class="p">.</span><span class="nf">uuid7</span><span class="p">())</span>
        <span class="n">timestamp</span> <span class="o">=</span> <span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">(</span><span class="n">timezone</span><span class="p">.</span><span class="n">utc</span><span class="p">).</span><span class="nf">isoformat</span><span class="p">()</span>

        <span class="c1"># Construct header (minimal required fields)
</span>        <span class="n">header</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">EventID</span><span class="sh">"</span><span class="p">:</span> <span class="n">event_id</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">Timestamp</span><span class="sh">"</span><span class="p">:</span> <span class="n">timestamp</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">VehicleID</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">vehicle_id</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">EventType</span><span class="sh">"</span><span class="p">:</span> <span class="n">event_type</span>
        <span class="p">}</span>

        <span class="c1"># Construct payload (domain-specific content)
</span>        <span class="n">payload</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">Action</span><span class="sh">"</span><span class="p">:</span> <span class="n">action</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">ModelID</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">model_id</span>
        <span class="p">}</span>

        <span class="k">if</span> <span class="n">sensor_inputs</span><span class="p">:</span>
            <span class="n">payload</span><span class="p">[</span><span class="sh">"</span><span class="s">SensorInputs</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">sensor_inputs</span>
        <span class="k">if</span> <span class="n">decision_factors</span><span class="p">:</span>
            <span class="n">payload</span><span class="p">[</span><span class="sh">"</span><span class="s">DecisionFactors</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">decision_factors</span>
        <span class="k">if</span> <span class="n">confidence</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
            <span class="n">payload</span><span class="p">[</span><span class="sh">"</span><span class="s">Confidence</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">confidence</span>

        <span class="c1"># Calculate event hash (includes previous hash for chain)
</span>        <span class="n">hash_input</span> <span class="o">=</span> <span class="p">(</span>
            <span class="n">self</span><span class="p">.</span><span class="nf">_canonical_json</span><span class="p">(</span><span class="n">header</span><span class="p">)</span> <span class="o">+</span>
            <span class="n">self</span><span class="p">.</span><span class="nf">_canonical_json</span><span class="p">(</span><span class="n">payload</span><span class="p">)</span> <span class="o">+</span>
            <span class="n">self</span><span class="p">.</span><span class="n">prev_hash</span><span class="p">.</span><span class="nf">encode</span><span class="p">()</span>
        <span class="p">)</span>
        <span class="n">event_hash</span> <span class="o">=</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">hash_input</span><span class="p">).</span><span class="nf">hexdigest</span><span class="p">()</span>

        <span class="c1"># Create security envelope
</span>        <span class="n">security</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">Version</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">1.0</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">PrevHash</span><span class="sh">"</span><span class="p">:</span> <span class="n">self</span><span class="p">.</span><span class="n">prev_hash</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">EventHash</span><span class="sh">"</span><span class="p">:</span> <span class="n">event_hash</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">HashAlgo</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">SHA256</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">SignAlgo</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">ED25519</span><span class="sh">"</span>
        <span class="p">}</span>

        <span class="c1"># Sign the event hash
</span>        <span class="n">signature</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">signing_key</span><span class="p">.</span><span class="nf">sign</span><span class="p">(</span><span class="n">event_hash</span><span class="p">.</span><span class="nf">encode</span><span class="p">())</span>
        <span class="n">security</span><span class="p">[</span><span class="sh">"</span><span class="s">Signature</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">signature</span><span class="p">.</span><span class="n">signature</span><span class="p">.</span><span class="nf">hex</span><span class="p">()</span>

        <span class="c1"># Update chain state
</span>        <span class="n">self</span><span class="p">.</span><span class="n">prev_hash</span> <span class="o">=</span> <span class="n">event_hash</span>

        <span class="k">return</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">Header</span><span class="sh">"</span><span class="p">:</span> <span class="n">header</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">Payload</span><span class="sh">"</span><span class="p">:</span> <span class="n">payload</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">Security</span><span class="sh">"</span><span class="p">:</span> <span class="n">security</span>
        <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Real-World Usage Example
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">nacl.signing</span> <span class="kn">import</span> <span class="n">SigningKey</span>

<span class="c1"># Initialize logger for vehicle
</span><span class="n">signing_key</span> <span class="o">=</span> <span class="n">SigningKey</span><span class="p">.</span><span class="nf">generate</span><span class="p">()</span>
<span class="n">logger</span> <span class="o">=</span> <span class="nc">DVPEventLogger</span><span class="p">(</span>
    <span class="n">signing_key</span><span class="o">=</span><span class="n">signing_key</span><span class="p">,</span>
    <span class="n">vehicle_id</span><span class="o">=</span><span class="sh">"</span><span class="s">VEH-2025-DEMO-001</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">model_id</span><span class="o">=</span><span class="sh">"</span><span class="s">perception_v3.2.1</span><span class="sh">"</span>
<span class="p">)</span>

<span class="c1"># Log perception event: pedestrian detected
</span><span class="n">event1</span> <span class="o">=</span> <span class="n">logger</span><span class="p">.</span><span class="nf">log_event</span><span class="p">(</span>
    <span class="n">event_type</span><span class="o">=</span><span class="sh">"</span><span class="s">PRC_OBJ</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">PEDESTRIAN_DETECTED</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">sensor_inputs</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">lidar_confidence</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.94</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">camera_confidence</span><span class="sh">"</span><span class="p">:</span> <span class="mf">0.91</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">radar_detected</span><span class="sh">"</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">distance_m</span><span class="sh">"</span><span class="p">:</span> <span class="mf">32.5</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">velocity_ms</span><span class="sh">"</span><span class="p">:</span> <span class="o">-</span><span class="mf">1.2</span>  <span class="c1"># Approaching
</span>    <span class="p">},</span>
    <span class="n">decision_factors</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">fusion_method</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">late_fusion_v2</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">classification</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pedestrian_adult</span><span class="sh">"</span>
    <span class="p">},</span>
    <span class="n">confidence</span><span class="o">=</span><span class="mf">0.93</span>
<span class="p">)</span>

<span class="c1"># Log planning decision: emergency brake initiated
</span><span class="n">event2</span> <span class="o">=</span> <span class="n">logger</span><span class="p">.</span><span class="nf">log_event</span><span class="p">(</span>
    <span class="n">event_type</span><span class="o">=</span><span class="sh">"</span><span class="s">CTL_EMG</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">EMERGENCY_BRAKE_INITIATED</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">sensor_inputs</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">ttc_seconds</span><span class="sh">"</span><span class="p">:</span> <span class="mf">2.1</span><span class="p">,</span>  <span class="c1"># Time to collision
</span>        <span class="sh">"</span><span class="s">current_speed_ms</span><span class="sh">"</span><span class="p">:</span> <span class="mf">15.6</span>
    <span class="p">},</span>
    <span class="n">decision_factors</span><span class="o">=</span><span class="p">{</span>
        <span class="sh">"</span><span class="s">trigger</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">pedestrian_in_path</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">brake_force_percent</span><span class="sh">"</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">predicted_stop_distance_m</span><span class="sh">"</span><span class="p">:</span> <span class="mf">18.2</span>
    <span class="p">},</span>
    <span class="n">confidence</span><span class="o">=</span><span class="mf">0.99</span>
<span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">event2</span><span class="p">,</span> <span class="n">indent</span><span class="o">=</span><span class="mi">2</span><span class="p">))</span>
</code></pre>

</div>



<p><strong>Output:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Header"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"EventID"</span><span class="p">:</span><span class="w"> </span><span class="s2">"019416a8-7c3f-7000-8000-000000000001"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Timestamp"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2025-01-03T12:34:56.789012+00:00"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"VehicleID"</span><span class="p">:</span><span class="w"> </span><span class="s2">"VEH-2025-DEMO-001"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"EventType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CTL_EMG"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"Payload"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"EMERGENCY_BRAKE_INITIATED"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"ModelID"</span><span class="p">:</span><span class="w"> </span><span class="s2">"perception_v3.2.1"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"SensorInputs"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"ttc_seconds"</span><span class="p">:</span><span class="w"> </span><span class="mf">2.1</span><span class="p">,</span><span class="w">
      </span><span class="nl">"current_speed_ms"</span><span class="p">:</span><span class="w"> </span><span class="mf">15.6</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"DecisionFactors"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"trigger"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pedestrian_in_path"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"brake_force_percent"</span><span class="p">:</span><span class="w"> </span><span class="mi">100</span><span class="p">,</span><span class="w">
      </span><span class="nl">"predicted_stop_distance_m"</span><span class="p">:</span><span class="w"> </span><span class="mf">18.2</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"Confidence"</span><span class="p">:</span><span class="w"> </span><span class="mf">0.99</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"Security"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"PrevHash"</span><span class="p">:</span><span class="w"> </span><span class="s2">"a3f2...previous event hash..."</span><span class="p">,</span><span class="w">
    </span><span class="nl">"EventHash"</span><span class="p">:</span><span class="w"> </span><span class="s2">"7b4c...current event hash..."</span><span class="p">,</span><span class="w">
    </span><span class="nl">"HashAlgo"</span><span class="p">:</span><span class="w"> </span><span class="s2">"SHA256"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"SignAlgo"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ED25519"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Signature"</span><span class="p">:</span><span class="w"> </span><span class="s2">"9d3e...64 byte signature..."</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h2>
  
  
  Merkle Tree Anchoring for External Verification
</h2>

<h3>
  
  
  The Problem with Continuous Logging
</h3>

<p>An AV generates thousands of events per minute. Anchoring every event to a blockchain would be:</p>

<ul>
<li>Prohibitively expensive</li>
<li>Bandwidth-intensive</li>
<li>Unnecessary</li>
</ul>

<h3>
  
  
  The Solution: Batch Anchoring
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[During Operation - Local Processing]

Event₁ → Event₂ → Event₃ → ... → Eventₙ
   ↓        ↓        ↓              ↓
[Hash Chain continuously validated locally]
   ↓        ↓        ↓              ↓
        Accumulated in local storage

[Periodic Anchoring - e.g., every 10 minutes or trip end]

         ┌─────────┐
         │  Root   │ ← Only this 32-byte hash
         └────┬────┘    gets anchored externally
        ┌─────┴─────┐
      ┌─┴─┐       ┌─┴─┐
      │H₁₂│       │H₃₄│
      └─┬─┘       └─┬─┘
     ┌──┴──┐    ┌──┴──┐
    H(E₁) H(E₂) H(E₃) H(E₄)
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">hashlib</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">List</span>

<span class="k">def</span> <span class="nf">build_merkle_tree</span><span class="p">(</span><span class="n">event_hashes</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="nb">str</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="sh">"""</span><span class="s">
    Build RFC 6962-compliant Merkle tree.

    Args:
        event_hashes: List of event hash strings

    Returns:
        Merkle root hash
    </span><span class="sh">"""</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">event_hashes</span><span class="p">:</span>
        <span class="k">raise</span> <span class="nc">ValueError</span><span class="p">(</span><span class="sh">"</span><span class="s">Cannot build tree from empty list</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Convert to leaf nodes (0x00 prefix per RFC 6962)
</span>    <span class="n">leaves</span> <span class="o">=</span> <span class="p">[</span>
        <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="sa">b</span><span class="sh">'</span><span class="se">\x00</span><span class="sh">'</span> <span class="o">+</span> <span class="n">h</span><span class="p">.</span><span class="nf">encode</span><span class="p">()).</span><span class="nf">digest</span><span class="p">()</span>
        <span class="k">for</span> <span class="n">h</span> <span class="ow">in</span> <span class="n">event_hashes</span>
    <span class="p">]</span>

    <span class="c1"># Pad to power of 2 if necessary
</span>    <span class="k">while</span> <span class="nf">len</span><span class="p">(</span><span class="n">leaves</span><span class="p">)</span> <span class="o">&amp;</span> <span class="p">(</span><span class="nf">len</span><span class="p">(</span><span class="n">leaves</span><span class="p">)</span> <span class="o">-</span> <span class="mi">1</span><span class="p">):</span>
        <span class="n">leaves</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">leaves</span><span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">])</span>

    <span class="c1"># Build tree bottom-up
</span>    <span class="k">while</span> <span class="nf">len</span><span class="p">(</span><span class="n">leaves</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">1</span><span class="p">:</span>
        <span class="n">next_level</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nf">len</span><span class="p">(</span><span class="n">leaves</span><span class="p">),</span> <span class="mi">2</span><span class="p">):</span>
            <span class="c1"># Internal nodes: 0x01 prefix per RFC 6962
</span>            <span class="n">combined</span> <span class="o">=</span> <span class="sa">b</span><span class="sh">'</span><span class="se">\x01</span><span class="sh">'</span> <span class="o">+</span> <span class="n">leaves</span><span class="p">[</span><span class="n">i</span><span class="p">]</span> <span class="o">+</span> <span class="n">leaves</span><span class="p">[</span><span class="n">i</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>
            <span class="n">next_level</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">combined</span><span class="p">).</span><span class="nf">digest</span><span class="p">())</span>
        <span class="n">leaves</span> <span class="o">=</span> <span class="n">next_level</span>

    <span class="k">return</span> <span class="n">leaves</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nf">hex</span><span class="p">()</span>


<span class="c1"># Example: Anchor a batch of events
</span><span class="n">batch_hashes</span> <span class="o">=</span> <span class="p">[</span><span class="n">event1</span><span class="p">[</span><span class="sh">"</span><span class="s">Security</span><span class="sh">"</span><span class="p">][</span><span class="sh">"</span><span class="s">EventHash</span><span class="sh">"</span><span class="p">],</span> 
                <span class="n">event2</span><span class="p">[</span><span class="sh">"</span><span class="s">Security</span><span class="sh">"</span><span class="p">][</span><span class="sh">"</span><span class="s">EventHash</span><span class="sh">"</span><span class="p">]]</span>
<span class="n">merkle_root</span> <span class="o">=</span> <span class="nf">build_merkle_tree</span><span class="p">(</span><span class="n">batch_hashes</span><span class="p">)</span>

<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Merkle Root: </span><span class="si">{</span><span class="n">merkle_root</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
<span class="c1"># This 32-byte hash can be anchored to blockchain/TSA
</span></code></pre>

</div>






<h2>
  
  
  Regulatory Alignment
</h2>

<h3>
  
  
  EU AI Act Compliance (Article 12)
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Requirement</th>
<th>DVP Implementation</th>
</tr>
</thead>
<tbody>
<tr>
<td>Automatic logging</td>
<td>Hash chain captures all decisions</td>
</tr>
<tr>
<td>Event timestamping</td>
<td>UUIDv7 + ISO 8601 timestamps</td>
</tr>
<tr>
<td>Operation duration</td>
<td>SYSTEM_MODE_CHANGE events</td>
</tr>
<tr>
<td>Input data logging</td>
<td>SensorInputs field</td>
</tr>
<tr>
<td>Reference database</td>
<td>ModelID + version tracking</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  UNECE WP.29 Alignment
</h3>

<p>DVP aligns with the emerging UN GTR on ADS by providing:</p>

<ul>
<li>
<strong>Scenario reconstruction</strong>: Complete causal chain from perception to action</li>
<li>
<strong>Failure mode documentation</strong>: SYSTEM_FALLBACK events with reasons</li>
<li>
<strong>Human oversight evidence</strong>: TAKEOVER_REQUEST and response timing</li>
<li>
<strong>ODD boundary compliance</strong>: Operational Design Domain events</li>
</ul>

<h3>
  
  
  ISO 21448 (SOTIF) Support
</h3>

<p>DVP enables SOTIF compliance by logging:</p>

<ul>
<li>Perception confidence levels</li>
<li>Sensor disagreement events</li>
<li>Edge case encounters</li>
<li>System uncertainty acknowledgment</li>
</ul>




<h2>
  
  
  Why This Matters for Developers
</h2>

<h3>
  
  
  If You're Building ADAS/ADS Software
</h3>

<p>DVP provides a <strong>vendor-neutral audit layer</strong> that:</p>

<ol>
<li>
<strong>Protects IP</strong>: Your algorithms remain proprietary</li>
<li>
<strong>Enables compliance</strong>: Ready for EU AI Act enforcement</li>
<li>
<strong>Supports forensics</strong>: Incident reconstruction becomes possible</li>
<li>
<strong>Future-proofs</strong>: As regulations tighten, you're already compliant</li>
</ol>

<h3>
  
  
  Integration is Minimal
</h3>

<p>DVP is designed as a <strong>sidecar</strong> — it doesn't require rewriting your perception stack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Minimal integration pattern
</span><span class="k">class</span> <span class="nc">YourExistingPerceptionSystem</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">dvp_logger</span><span class="p">:</span> <span class="n">DVPEventLogger</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">dvp</span> <span class="o">=</span> <span class="n">dvp_logger</span>

    <span class="k">def</span> <span class="nf">process_frame</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">sensor_data</span><span class="p">):</span>
        <span class="c1"># Your existing logic unchanged
</span>        <span class="n">result</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_run_inference</span><span class="p">(</span><span class="n">sensor_data</span><span class="p">)</span>

        <span class="c1"># Add DVP logging (one line)
</span>        <span class="n">self</span><span class="p">.</span><span class="n">dvp</span><span class="p">.</span><span class="nf">log_event</span><span class="p">(</span><span class="sh">"</span><span class="s">PRC_OBJ</span><span class="sh">"</span><span class="p">,</span> <span class="n">result</span><span class="p">.</span><span class="n">action</span><span class="p">,</span> 
                          <span class="n">sensor_inputs</span><span class="o">=</span><span class="n">result</span><span class="p">.</span><span class="n">inputs</span><span class="p">,</span>
                          <span class="n">confidence</span><span class="o">=</span><span class="n">result</span><span class="p">.</span><span class="n">confidence</span><span class="p">)</span>

        <span class="k">return</span> <span class="n">result</span>
</code></pre>

</div>






<h2>
  
  
  The Bigger Picture: VAP Framework
</h2>

<p>DVP is part of the <strong>Verifiable AI Provenance (VAP) Framework</strong> — a family of domain-specific protocols sharing common cryptographic foundations:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Profile</th>
<th>Domain</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>VCP</strong></td>
<td>Algorithmic Trading</td>
<td>v1.0 Released</td>
</tr>
<tr>
<td><strong>DVP</strong></td>
<td>Autonomous Driving</td>
<td>Specification Phase</td>
</tr>
<tr>
<td><strong>MAP</strong></td>
<td>Medical AI</td>
<td>Planned</td>
</tr>
<tr>
<td><strong>AAP</strong></td>
<td>Aviation AI</td>
<td>Planned</td>
</tr>
<tr>
<td><strong>EIP</strong></td>
<td>Energy Infrastructure</td>
<td>Planned</td>
</tr>
</tbody>
</table></div>

<p>The shared foundation means:</p>

<ul>
<li>Cross-domain expertise transfer</li>
<li>Unified tooling ecosystem</li>
<li>Regulatory familiarity</li>
</ul>




<h2>
  
  
  Getting Started
</h2>

<h3>
  
  
  Resources
</h3>

<ul>
<li>
<strong>VAP Framework Specification</strong>: <a href="https://github.com/veritaschain" rel="noopener noreferrer">github.com/veritaschain</a>
</li>
<li>
<strong>Technical Contact</strong>: <a href="mailto:technical@veritaschain.org">technical@veritaschain.org</a>
</li>
<li>
<strong>IETF Draft (VCP)</strong>: <a href="https://datatracker.ietf.org/doc/draft-kamimura-scitt-vcp/" rel="noopener noreferrer">datatracker.ietf.org/doc/draft-kamimura-scitt-vcp/</a>
</li>
</ul>

<h3>
  
  
  Contributing
</h3>

<p>DVP is developed by the <strong>VeritasChain Standards Organization (VSO)</strong> as an open standard. We welcome:</p>

<ul>
<li>Automotive OEM feedback</li>
<li>Tier-1 supplier input</li>
<li>Regulatory perspective</li>
<li>Academic collaboration</li>
</ul>

<p>Contact: <a href="mailto:standards@veritaschain.org">standards@veritaschain.org</a></p>




<h2>
  
  
  Conclusion
</h2>

<p>The autonomous vehicle industry is at an inflection point. The same forces that drove aviation to standardize flight data recorders — accidents, liability, regulation — are now converging on AV.</p>

<p>The question isn't <em>whether</em> standardized AI audit trails will become mandatory. It's <em>who will define them</em>.</p>

<p>DVP offers an open, technically sound, vendor-neutral answer. The code is simple. The cryptography is proven. The regulatory alignment is clear.</p>

<p><strong>The only question is: Will the industry adopt transparency voluntarily, or wait until the first major incident forces it?</strong></p>




<p><em>This article introduces concepts from the Verifiable AI Provenance (VAP) Framework developed by the VeritasChain Standards Organization. VCP (the finance profile) has been submitted to IETF SCITT working group. DVP specifications are in active development.</em></p>




<p><strong>About the Author</strong>: This article was written as part of the VeritasChain Standards Organization's developer outreach program. For more information, visit <a href="https://veritaschain.org" rel="noopener noreferrer">veritaschain.org</a>.</p>




<p><strong>Tags</strong>: #autonomousdriving #selfdrivingcars #ai #machinelearning #cryptography #opensource #adas #unece #euaiact #audit #compliance #embedded #automotive</p>

