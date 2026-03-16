---
Title: Hard Numbers: HTTP/2 vs UDP Overlay for Agent Communication
Description: 
Author: Calin Teodor
Date: 2026-03-16T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Two machines. Two continents. GCP us-east1 (South Carolina) and europe-west1 (Belgium). 85ms RTT. 100 runs per test, median reported. No synthetic benchmarks.</p>

<p><strong>Connection setup: 11x faster.</strong></p>

<p>HTTP/2 requires a TCP three-way handshake (~85ms), TLS 1.3 handshake (~85ms), and ALPN negotiation. Total: ~175ms per connection. Pilot does the expensive work (STUN discovery, tunnel creation) once at daemon startup. Each new connection reuses the existing tunnel. Total: ~15ms.</p>

<p>An orchestrator dispatching tasks to 50 agents: 8.75 seconds of pure overhead with HTTP/2. 750 milliseconds with Pilot.</p>

<p><strong>Message latency: identical where it matters.</strong></p>

<p>At 1 KB (the size of a JSON task description): HTTP/2 172ms, Pilot 171ms. At 10 KB: 174ms vs 172ms. At 100 KB: 182ms vs 179ms. Both protocols are dominated by the 85ms network RTT. The protocol overhead is not the bottleneck. The network is.</p>

<p>At 1 MB, HTTP/2 edges ahead by 2.4% — TCP's 30 years of kernel-level optimization. For the message sizes agents actually send, irrelevant.</p>

<p><strong>Memory at scale: 10x lighter.</strong></p>

<p>100 simultaneous peer connections: HTTP/2 uses 240 MB RSS. Pilot uses 24 MB. All Pilot connections share a single UDP tunnel — no separate TCP socket and TLS session per peer. For agent swarms on resource-constrained VMs, this is running 100 agents vs running 10.</p>

<p><strong>Behind NAT: where it actually matters.</strong></p>

<p>Put one agent behind Cloud NAT. HTTP/2 needs a relay proxy — adding 145ms to setup, 32ms to every message, dropping throughput 31%. Pilot hole-punches through the NAT and establishes a direct tunnel. After the punch: +7ms setup, +2ms per message, 4% throughput loss.</p>

<p>The choice is not "HTTP or Pilot." It is "HTTP where you can, Pilot where you must." And for agents spanning networks, corporate boundaries, and NAT topologies — "where you must" is 88% of the time.</p>




<p><strong>Read more:</strong> <a href="https://pilotprotocol.network/blog/benchmarking-http-vs-udp-overlay.html" rel="noopener noreferrer">Benchmarking Agent Communication: HTTP vs. UDP Overlay</a> · <a href="https://pilotprotocol.network/blog/connect-ai-agents-behind-nat-without-vpn.html" rel="noopener noreferrer">Connect AI Agents Behind NAT Without a VPN</a></p>

<p><a href="https://pilotprotocol.network" rel="noopener noreferrer">pilotprotocol.network</a> · <a href="https://github.com/TeoSlayer/pilotprotocol" rel="noopener noreferrer">GitHub</a></p>

