---
Title: Performance Tuning: Linux Kernel Optimizations for 10k+ Connections
Description: 
Author: Lalit Mishra
Date: 2025-12-20T21:30:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  1. Introduction
</h2>

<p>In high-concurrency real-time architectures, the performance bottleneck inevitably shifts from the application layer to the operating system. A well-optimized Flask-SocketIO application running on Gevent or Eventlet is theoretically capable of handling tens of thousands of concurrent connections. However, in a default Linux environment, such an application will likely crash or stop accepting connections long before CPU or memory resources are saturated.</p>

<p>This plateau occurs because the Linux kernel, out of the box, is tuned for general-purpose computing, not for acting as a massive termination point for persistent TCP connections. For a WebSocket server, where connections are long-lived and stateful, resource exhaustion manifests in the form of file descriptor limits, ephemeral port starvation, and TCP stack congestion. This article outlines the specific kernel-level tuning required to scale Flask-SocketIO beyond the 10,000-connection barrier.</p>

<h2>
  
  
  2. The 1024 Limit: Understanding File Descriptors
</h2>

<p>In Unix-like operating systems, "everything is a file." This includes TCP sockets. When a client connects to your server, the kernel allocates a file descriptor (FD) to represent that socket.</p>

<p>By default, most Linux distributions enforce a strict limit of 1024 open file descriptors per process. This is a legacy constraint. For a WebSocket server, this means that after roughly 1,000 concurrent users (plus a few descriptors for log files and shared libraries), the application will crash or throw <code>OSError: [Errno 24] Too many open files</code>.</p>

<h3>
  
  
  Soft Limits vs. Hard Limits
</h3>

<p>The kernel distinguishes between the "soft limit" (user-configurable ceiling) and the "hard limit" (absolute ceiling set by root).</p>

<p>To support 10k+ connections, you must increase these limits both at the system level and the process level (systemd/Supervisor).</p>

<p><strong>Verification:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">$ </span><span class="nb">ulimit</span> <span class="nt">-n</span>
1024

</code></pre>

</div>



<p><strong>Remediation:</strong><br>
For a production system, this should be raised significantly, often to 65535 or higher.</p>

<ol>
<li>
<strong>System-wide (<code>/etc/security/limits.conf</code>):</strong>
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="k">*</span> soft nofile 65535
<span class="k">*</span> hard nofile 65535

</code></pre>

</div>


<ol>
<li>
<strong>Systemd Service (<code>/etc/systemd/system/app.service</code>):</strong>
Systemd ignores user limits. You must explicitly define them in the unit file:
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight ini"><code>
<span class="py">LimitNOFILE</span><span class="p">=</span><span class="s">65535</span>

</code></pre>

</div>


<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw6fae0sw2eeimoln0ovh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw6fae0sw2eeimoln0ovh.png" alt="File Descriptor Bottleneck" width="800" height="446"></a></p>
<h2>
  
  
  3. Ephemeral Ports: The Redis Connection Bottleneck
</h2>

<p>While file descriptors limit <em>incoming</em> connections, ephemeral ports limit <em>outgoing</em> connections. This distinction is critical for Flask-SocketIO architectures that rely on a message broker like Redis.</p>

<p>When your Flask application connects to Redis (or Nginx connects to your upstream Flask Gunicorn workers), it opens a TCP socket. This outgoing connection requires a local IP address and a local port. The kernel assigns this local port from a predefined range known as the "ephemeral port range."</p>
<h3>
  
  
  The Exhaustion Scenario
</h3>

<p>By default, this range is often narrow (e.g., 32768–60999). This provides only ~28,000 ports. In high-throughput scenarios where the Flask app is furiously publishing to Redis or Nginx is proxying massive traffic, the server can run out of available local ports.</p>

<p><strong>Symptoms:</strong></p>

<ul>
<li>
<code>EADDRNOTAVAIL</code> (Cannot assign requested address) errors in logs.</li>
<li>Sudden inability of the Flask app to talk to Redis, despite Redis being healthy.</li>
<li>Nginx returning 502 Bad Gateway because it cannot open a socket to the upstream.</li>
</ul>

<p><strong>Tuning:</strong><br>
Widen the range to the maximum allowable by the TCP spec (1024–65535).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check current range</span>
sysctl net.ipv4.ip_local_port_range

<span class="c"># Expand range (add to /etc/sysctl.conf)</span>
sysctl <span class="nt">-w</span> net.ipv4.ip_local_port_range<span class="o">=</span><span class="s2">"1024 65535"</span>

</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq9hqpyr3d1g8greay9pi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq9hqpyr3d1g8greay9pi.png" alt="Ephemeral Port Exhaustion" width="800" height="446"></a></p>

<h2>
  
  
  4. TIME_WAIT Reuse: Optimizing TCP Stack Recycling
</h2>

<p>The most misunderstood aspect of TCP scaling is the <code>TIME_WAIT</code> state. When a TCP connection is closed, the side that initiated the close enters <code>TIME_WAIT</code> for a duration of <code>2 * MSL</code> (Maximum Segment Lifetime), typically 60 seconds. This ensures that any delayed packets related to that connection are handled correctly and not attributed to a new connection on the same port.</p>

<p>In a high-churn environment (e.g., clients constantly refreshing pages or reconnecting), the server can accumulate tens of thousands of sockets in <code>TIME_WAIT</code>. These sockets consume system resources and, more critically, lock up the ephemeral port 4-tuple, preventing new outgoing connections.</p>

<h3>
  
  
  The <code>tcp_tw_recycle</code> Hazard
</h3>

<p>Historically, guides recommended enabling <code>net.ipv4.tcp_tw_recycle</code>. <strong>Do not do this.</strong> It was removed in Linux kernel 4.12 because it breaks connections for users behind NAT (Network Address Translation) by aggressively dropping packets with out-of-order timestamps.</p>

<h3>
  
  
  The Correct Fix: <code>tcp_tw_reuse</code>
</h3>

<p>Instead, use <code>net.ipv4.tcp_tw_reuse</code>. This allows the kernel to reclaim a <code>TIME_WAIT</code> socket for a <em>new outgoing connection</em> if the new connection's timestamp is strictly greater than the last packet seen on the old connection. This is safe for most internal infrastructure (like Flask connecting to Redis).</p>

<p><strong>Configuration (<code>/etc/sysctl.conf</code>):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Allow reuse of sockets in TIME_WAIT state for new connections</span>
net.ipv4.tcp_tw_reuse <span class="o">=</span> 1

</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqd8qhkhgkuqon9oe31cm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqd8qhkhgkuqon9oe31cm.png" alt="TCP Connection Lifecycle" width="800" height="446"></a></p>

<h2>
  
  
  5. Load Testing: Simulating Reality with Artillery
</h2>

<p>Standard HTTP benchmarking tools like <code>ab</code> (Apache Bench) are useless for WebSockets. They measure request throughput (requests/sec), whereas the primary metric for WebSockets is <strong>concurrency</strong> (simultaneous open connections) and <strong>message latency</strong>.</p>

<p>To verify your kernel tuning, you must simulate sustained connection loads. <strong>Artillery</strong> and <strong>Locust</strong> are the industry standards here.</p>

<h3>
  
  
  Testing Strategy
</h3>

<ol>
<li>
<strong>Ramp Up:</strong> Don't connect 10k users instantly; this triggers "thundering herd" protection or syn-flood defenses. Ramp up over minutes.</li>
<li>
<strong>Sustain:</strong> Hold the connections open.</li>
<li>
<strong>Broadcast:</strong> While connections are held, trigger a broadcast event to measure the latency of the Redis backplane and the Nginx proxy buffering.</li>
</ol>

<p>If your test fails at exactly 1024 users, you missed a file descriptor limit. If it fails around 28,000 users, you likely hit the ephemeral port limit.</p>

<h2>
  
  
  6. Monitoring: Key Metrics in Grafana
</h2>

<p>Observability is the only way to confirm that kernel tuning is effective. When running high-concurrency workloads, you must monitor specific OS-level metrics.</p>

<h3>
  
  
  1. File Descriptors (Open Files)
</h3>

<p>Track <code>process_open_fds</code> for the Gunicorn/uWSGI process. If this line flattens at a specific number (e.g., 1024 or 4096) while CPU is low, you have hit a hard limit.</p>

<h3>
  
  
  2. TCP States
</h3>

<p>Monitor the count of sockets in each state, specifically:</p>

<ul>
<li>
<code>ESTABLISHED</code>: Should match your active user count.</li>
<li>
<code>TIME_WAIT</code>: If this spikes to 30k+, you have a churn problem or need <code>tcp_tw_reuse</code>.</li>
<li>
<code>Allocated Sockets</code> (<code>sockstat</code>): The total memory consumed by networking buffers.</li>
</ul>

<h3>
  
  
  3. Conntrack Table
</h3>

<p>If you are using <code>iptables</code> or Docker, the <code>nf_conntrack</code> table limits how many connections the firewall tracks. Use <code>dmesg</code> to check for "nf_conntrack: table full, dropping packet".</p>

<ul>
<li>
<strong>Tune:</strong> <code>sysctl -w net.netfilter.nf_conntrack_max=131072</code>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6cck81fz00y8wdgeqy9n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6cck81fz00y8wdgeqy9n.png" alt="Grafana High-Concurrency Dashboard" width="800" height="446"></a></p>

<h2>
  
  
  7. Trade-offs and Production Considerations
</h2>

<p>Aggressive kernel tuning is not without risk. It removes safety rails designed to protect the system from resource exhaustion.</p>

<ol>
<li>
<strong>Security:</strong> Expanding the ephemeral port range makes port scanning slightly easier, though this is negligible inside a private VPC.</li>
<li>
<strong>Stability:</strong> Setting file descriptor limits too high (e.g., millions) can allow a memory leak in the application to crash the entire server rather than just the process.</li>
<li>
<strong>Connection Tracking:</strong> Increasing <code>nf_conntrack_max</code> consumes kernel memory (RAM). Ensure your server has sufficient RAM to store the state of 100k+ tracked connections.</li>
</ol>

<p><strong>Golden Rule:</strong> Never apply sysctl settings blindly. Apply them via configuration management (Ansible/Terraform), document <em>why</em> they are needed, and validate with load testing.</p>

<h2>
  
  
  8. Conclusion
</h2>

<p>Scaling Flask-SocketIO to 10,000+ connections is an exercise in systems engineering as much as software development. The default Linux configuration is conservative, designed for general desktop or low-traffic server usage. By systematically addressing the file descriptor limits (<code>ulimit</code>), expanding the ephemeral port range (<code>ip_local_port_range</code>), and optimizing TCP reuse (<code>tcp_tw_reuse</code>), you unlock the operating system's ability to support high-concurrency workloads.</p>

<p><strong>Production Readiness Checklist:</strong></p>

<ul>
<li>[ ] <code>ulimit -n</code> &gt; 65535 for the Gunicorn process.</li>
<li>[ ] <code>net.ipv4.ip_local_port_range</code> set to <code>1024 65535</code>.</li>
<li>[ ] <code>net.ipv4.tcp_tw_reuse</code> enabled (1).</li>
<li>[ ] <code>net.ipv4.tcp_tw_recycle</code> disabled (0) or absent.</li>
<li>[ ] <code>nf_conntrack_max</code> increased if using stateful firewalls.</li>
</ul>

