---
Title: Availability - Heartbeats (2)
Description: 
Author: Udayan Sawant
Date: 2025-11-15T21:23:40.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzmdo2rqviddg8qkj1hfd.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzmdo2rqviddg8qkj1hfd.jpg" alt="Heartbeats" width="800" height="450"></a></p>

<p>We introduced heartbeats as periodic "I'm alive" messages in distributed systems, unpacked how they support failure detection and cluster membership, and compared different heartbeat topologies: centralized monitors, peer-to-peer checks, and gossip-based designs. Recap<br>
We also talked about how intervals, timeouts, and simple failure detection logic turn into a real trade-off between fast detection and noisy false positives. With that mental model in place, we're ready to build a small system, examine its failure modes, and refine it toward something production-worthy.</p>



<p><strong>Heartbeats Are More Than "I'm Alive"</strong><br>
Once you have a periodic signal, you can sneak in extra metadata.<br>
Common piggybacked fields:</p>

<ul>
<li>Current load (CPU, memory, request rate)</li>
<li>Version or build hash (for safe rolling deployments)</li>
<li>Epoch/term info (for consensus / leader election)</li>
<li>Shard ownership or partition state</li>
</ul>

<p>Examples in real systems:</p>

<ul>
<li>Load balancers: health checks may include not just "HTTP 200" but also whether the instance is overloaded.</li>
<li>Kubernetes: readiness and liveness probes gate scheduling/traffic. The kubelet periodically reports node status to the control plane.</li>
<li>Consensus protocols: Raft leaders send periodic heartbeats (AppendEntries RPCs, even empty) to assert leadership and prevent elections.</li>
</ul>

<p>The heartbeat becomes a low-bandwidth control channel for the cluster.</p>



<p><strong>A Tiny Heartbeat System in Python (for Intuition)</strong></p>

<p>Let's sketch a simple heartbeat system in Python using asyncio.</p>

<p>Toy Model</p>

<ul>
<li>Each worker node: Keeps sending heartbeats to a central monitor over HTTP.</li>
<li>The monitor: Tracks last seen times; Marks nodes as "suspected dead" if they go silent.</li>
</ul>

<p>This is not production-ready, but it maps the theory to something concrete.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import time
from typing import Dict
from fastapi import FastAPI
from pydantic import BaseModel
import uvicorn

app = FastAPI()

HEARTBEAT_TIMEOUT = 5.0  # seconds
last_seen: Dict[str, float] = {}

class Heartbeat(BaseModel):
    node_id: str
    ts: float

@app.post("/heartbeat")
async def heartbeat(hb: Heartbeat):
    last_seen[hb.node_id] = hb.ts
    return {"status": "ok"}

@app.get("/status")
async def status():
    now = time.time()
    status = {}
    for node, ts in last_seen.items():
        delta = now - ts
        status[node] = {
            "last_seen": ts,
            "age": delta,
            "alive": delta &lt; HEARTBEAT_TIMEOUT,
        }
    return status

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
</code></pre>

</div>



<p>Worker (Node) Sending Heartbeats<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import asyncio
import time
import httpx

MONITOR_URL = "http://localhost:8000/heartbeat"
NODE_ID = "node-1"
INTERVAL = 1.0  # seconds

async def send_heartbeats():
    async with httpx.AsyncClient() as client:
        while True:
            payload = {"node_id": NODE_ID, "ts": time.time()}
            try:
                await client.post(MONITOR_URL, json=payload, timeout=1.0)
            except Exception as e:
                # In real systems, you'd log this and possibly backoff
                print(f"Failed to send heartbeat: {e}")
            await asyncio.sleep(INTERVAL)

if __name__ == "__main__":
    asyncio.run(send_heartbeats())
</code></pre>

</div>



<p>Run the monitor, start a couple of workers, and then kill one worker process. Within ~5 seconds, /status will show it as not alive.</p>

<p>You just implemented:</p>

<ul>
<li>A heartbeat sender</li>
<li>A central monitor</li>
<li>Timeouts and liveness calculation</li>
</ul>

<p>In real systems, this evolves into:</p>

<ul>
<li>Redundant monitors (no single point of failure)</li>
<li>Persistent state or shared stores (so status survives restarts)</li>
<li>Gossip instead of centralization</li>
<li>Smarter failure detectors</li>
</ul>

<p>But the mental model stays the same.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1yv2xjuqms34pdsbh7rt.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1yv2xjuqms34pdsbh7rt.PNG" alt="request/response view of the toy implementation" width="748" height="617"></a></p>




<p><strong>Pitfalls: Where Heartbeats Get You in Trouble</strong><br>
Heartbeats are simple; failure detection is not.</p>

<p><strong>1. Network Partitions vs Crashes</strong><br>
If a node stops sending heartbeats, did it:</p>

<ul>
<li>Crash?</li>
<li>Lose network connectivity in one direction?</li>
<li>Hit a local resource issue (GC freeze, kernel stall)?</li>
<li>Suffer a partial partition where only some peers can see it?</li>
</ul>

<p>From the cluster's point of view, all of these look similar: no heartbeat.<br>
This is why systems often distinguish:</p>

<ul>
<li>suspected vs definitely dead</li>
<li>transient vs permanent failure</li>
</ul>

<p>And why many protocols allow nodes to rejoin after being declared dead, usually with higher "generation" or epoch numbers.</p>

<p><strong>2. False Positives (Flapping)</strong><br>
If your timeout is too aggressive, you end up in the nightmare scenario:</p>

<ul>
<li>Node is alive but slow.</li>
<li>You mark it dead.</li>
<li>Failover kicks in.</li>
<li>The node comes back.</li>
<li>Now you have duplicate leaders or conflicting state.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu3pq53cb66j7al79u37q.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu3pq53cb66j7al79u37q.PNG" alt="false positive due to pause" width="506" height="626"></a></p>

<p>To avoid this, production systems often:</p>

<ul>
<li>Require multiple missed heartbeats before declaring failure.</li>
<li>Use suspicion levels rather than booleans.</li>
<li>Back off decisions if there's a known network issue.</li>
</ul>

<ol>
<li>Scalability and Overhead
In very large clusters, heartbeats aren't free:</li>
</ol>

<ul>
<li>A fully connected graph (everyone heartbeating everyone) is O(N²).</li>
<li>Even centralized monitoring can become a bottleneck in big deployments.</li>
</ul>

<p>Mitigations:</p>

<ul>
<li>Gossip / partial views instead of full meshes.</li>
<li>Hierarchical monitors (local agents report to regional controllers).</li>
<li>Adaptive intervals (idle components heartbeat less often).</li>
</ul>




<p><strong>Heartbeats in Systems You Already Know</strong><br>
This isn't an academic pattern - you've already met it in many places:</p>

<ul>
<li>Kubernetes: Nodes and pods are constantly being probed; readiness/liveness checks and node status reporting are heartbeat flavored under the hood.</li>
<li>Distributed Databases (Cassandra, etcd, ZooKeeper): Use heartbeats for membership, leader election, and ensuring quorum health. Cassandra combines gossip + φ-accrual detectors to avoid premature death certificates. </li>
<li>Service Meshes / API Gateways: Sidecars and control planes trade health info to know where to route traffic.</li>
<li>Load Balancers &amp; Health Checks: From AWS ALB to Nginx, health checks (active or passive) are heartbeat cousins: same idea, different framing.</li>
</ul>




<p><strong>Design Checklist for Heartbeats (In the Real World)</strong><br>
When you add heartbeats to a system, ask yourself:</p>

<ul>
<li>Who monitors what? Central node? Peer-to-peer? Gossip?</li>
<li>What's the interval and timeout? How fast do you need detection vs how noisy is the environment?</li>
<li>What exactly happens on failure? Do you remove from load balancer, trigger leader election, alert humans?</li>
<li>How do nodes rejoin? Can a previously-dead node come back safely (with a new epoch/generation)?</li>
<li>What's the scale? 10 nodes, 1000 nodes, or 100k IoT devices with flaky connections?</li>
<li>Do you piggyback metadata? Version, load, shard info, etc.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkbgc5vucjo8hvm3rwrlp.PNG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkbgc5vucjo8hvm3rwrlp.PNG" alt="Design Checklist for Heartbeats (In the Real World)" width="205" height="656"></a></p>

<p>If you can answer these, your heartbeat design is already ahead of many real production setups.</p>




<p><strong>TL;DR</strong><br>
Heartbeats are the kind of thing you rarely brag about in postmortems or blog posts - until they break.<br>
They're just small, repetitive, almost boring messages. But they give distributed systems something like a nervous system: a way to sense which parts are alive, which are failing, and when to adapt.<br>
Design them carelessly, and you get false alarms, flapping nodes, and mysterious outages. Design them thoughtfully, and your system can lose machines, racks, and zones while users barely notice.<br>
In a distributed world, silence is ambiguity. Heartbeats turn that silence into information.</p>

