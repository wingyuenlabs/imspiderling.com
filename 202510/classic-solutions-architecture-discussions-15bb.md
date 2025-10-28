---
Title: Classic solutions architecture discussions
Description: 
Author: Aisalkyn Aidarova
Date: 2025-10-28T21:56:42.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  WhatIsTheTime.com — Architecture Journey (AWS)
</h1>

<h2>
  
  
  0) Baseline PoC
</h2>

<ul>
<li>
<strong>Architecture:</strong> 1× public EC2 (t2.micro) + <strong>Elastic IP</strong> (EIP).</li>
<li>
<strong>Pros:</strong> Fast, simple, stable IP.</li>
<li>
<strong>Cons:</strong> Single point of failure; vertical changes = <strong>downtime</strong>.</li>
</ul>

<h2>
  
  
  1) Vertical Scale
</h2>

<ul>
<li>
<strong>Action:</strong> Stop instance → change type (e.g., m5.large) → start.</li>
<li>
<strong>IP:</strong> Still stable (EIP).</li>
<li>
<strong>Trade-off:</strong> More capacity, but <strong>planned downtime</strong> and still 1 box.</li>
</ul>

<h2>
  
  
  2) Naïve Horizontal Scale (no LB)
</h2>

<ul>
<li>
<strong>Action:</strong> Add more EC2s, each with its own EIP; users connect by IP.</li>
<li>
<strong>Cons:</strong> Users must know multiple IPs; EIP <strong>account/region limits</strong> (~5); no health checks; hard to add/remove nodes.</li>
</ul>

<h2>
  
  
  3) Route 53 “A record to instances”
</h2>

<ul>
<li>
<strong>Action:</strong> A record (TTL=1h) → list of instance public IPs.</li>
<li>
<strong>Pros:</strong> No EIP management; DNS returns current IPs.</li>
<li>
<strong>Gotcha:</strong> <strong>TTL caching</strong>. If an instance is removed, clients may keep a stale IP for up to TTL → perceived downtime.</li>
</ul>

<h2>
  
  
  4) Proper Horizontal Scale with Load Balancer
</h2>

<ul>
<li>
<strong>Action:</strong> Put <strong>ALB/NLB</strong> public; EC2s become <strong>private</strong> behind it.</li>
<li>
<strong>DNS:</strong> Route 53 <strong>Alias</strong> record (A/AAAA Alias) → LB DNS name (LB IPs change!).</li>
<li>
<p><strong>Benefits:</strong></p>

<ul>
<li>
<strong>Health checks</strong>: Unhealthy targets don’t receive traffic.</li>
<li>
<strong>No client TTL pain</strong>: LB stays constant; you can add/remove targets anytime.</li>
<li>
<strong>Security</strong>: SG on EC2s allows <strong>only</strong> LB SG.</li>
</ul>


</li>

</ul>

<h2>
  
  
  5) Auto Scaling Group (ASG)
</h2>

<ul>
<li>
<strong>Action:</strong> ASG manages private EC2 fleet behind the LB.</li>
<li>
<strong>Scaling:</strong> Policies by CPU, RPS, ALB request count, or target tracking.</li>
<li>
<strong>Benefit:</strong> Right-sized capacity over the day; <strong>no manual node ops</strong>.</li>
</ul>

<h2>
  
  
  6) Multi-AZ High Availability
</h2>

<ul>
<li>
<strong>Action:</strong> Put ALB across <strong>≥2 AZs</strong>; ASG spans <strong>≥2 AZs</strong> (e.g., min 2, one per AZ).</li>
<li>
<strong>Outcome:</strong> Survives <strong>AZ failure</strong>; traffic shifts automatically.</li>
<li>
<strong>Tip:</strong> Ensure subnets in different AZs are <strong>public for ALB</strong>, <strong>private for EC2</strong> with NAT if instances need egress.</li>
</ul>

<h2>
  
  
  7) Cost Optimization
</h2>

<ul>
<li>
<strong>Steady floor:</strong> Reserve the <strong>ASG min capacity</strong> using <strong>Reserved Instances</strong> or <strong>Compute Savings Plans</strong>.</li>
<li>
<strong>Burst capacity:</strong> Use <strong>On-Demand</strong> or <strong>Spot</strong> via <strong>MixedInstancesPolicy</strong> (Spot for stateless, interrupt-tolerant).</li>
<li>
<strong>Right-size:</strong> Regularly re-evaluate instance families/sizes.</li>
</ul>




<h2>
  
  
  Security Group (SG) Pattern (minimal)
</h2>

<ul>
<li>
<strong>ALB SG:</strong> Inbound 80/443 from <code>0.0.0.0/0</code>; Outbound to EC2 SG on app port (e.g., 80).</li>
<li>
<strong>EC2 SG:</strong> Inbound <strong>only from ALB SG</strong> on app port; outbound as needed.</li>
<li>
<strong>No public IP</strong> on EC2 in private subnets.</li>
</ul>

<h2>
  
  
  Route 53 Records
</h2>

<ul>
<li>
<strong>Public site:</strong> <code>whatisthetime.com</code> → <strong>Alias A</strong> to ALB.</li>
<li>
<strong>API subdomain:</strong> <code>api.whatisthetime.com</code> → <strong>Alias A</strong> to same/different ALB if you split tiers.</li>
<li>
<strong>Don’t</strong> use plain A to LB IPs (they aren’t stable). <strong>Do</strong> use Alias.</li>
</ul>

<h2>
  
  
  Health Checks
</h2>

<ul>
<li>
<strong>ALB Target Group:</strong> HTTP 200 on <code>/health</code> (fast, lightweight).</li>
<li>
<strong>Tune:</strong> Healthy threshold (e.g., 2), interval (e.g., 10s), timeout (e.g., 5s). Faster detection → faster failout.</li>
</ul>

<h2>
  
  
  ASG Essentials
</h2>

<ul>
<li>
<strong>Launch Template:</strong> AMI, instance profile (IAM role), SG, user data (app start).</li>
<li>
<strong>Subnets:</strong> Private subnets across multiple AZs.</li>
<li>
<strong>Desired/Min/Max:</strong> e.g., <code>2/2/6</code>.</li>
<li>
<strong>Scaling policy:</strong> Target tracking (e.g., 50% CPU) or ALB request count per target.</li>
</ul>

<h2>
  
  
  Common Gotchas (great interview talking points)
</h2>

<ul>
<li>
<strong>DNS TTL vs. instance churn:</strong> Use LB to avoid TTL staleness.</li>
<li>
<strong>EIP limits</strong> and operational pain at scale.</li>
<li>
<strong>Single-AZ ASG</strong> still fails if that AZ dies; <strong>Multi-AZ is mandatory</strong> for HA.</li>
<li>
<strong>Health check path</strong> wrong → all targets marked unhealthy.</li>
<li>
<strong>User data idempotence:</strong> Make bootstrapping safe on restarts.</li>
<li>
<strong>Statelessness:</strong> Required to freely scale/replace nodes (no local session state).</li>
</ul>

<h2>
  
  
  Quick Lab Checklist (hands-on)
</h2>

<ol>
<li>
<strong>VPC:</strong> 2 public subnets (ALB), 2 private subnets (EC2) across <strong>2 AZs</strong>; 1 NAT GW.</li>
<li>
<strong>IAM Role:</strong> EC2 instance profile (e.g., SSM, CloudWatch logs).</li>
<li>
<strong>ALB:</strong> Internet-facing; listener 80/443; target group HTTP:80.</li>
<li>
<strong>ASG:</strong> Launch template + MixedInstancesPolicy (OD+Spot optional), min=2, desired=2, max=6.</li>
<li>
<strong>Route 53:</strong> <strong>Alias</strong> <code>whatisthetime.com</code> → ALB.</li>
<li>
<strong>App:</strong> Simple HTTP server returning current time + <code>/health</code> 200.</li>
<li>
<strong>Test:</strong> Kill instances; watch ALB route around, ASG replace, zero user downtime.</li>
</ol>

<h2>
  
  
  One-liner Interview Summary
</h2>

<blockquote>
<p>Start with a single EIP-backed EC2 (downtime on resize), then move to Route 53 A records and discover TTL issues. Fix with an internet-facing ALB and <strong>Alias</strong> record, put EC2s private behind it, add <strong>ASG</strong> for elasticity, and span <strong>multiple AZs</strong> for HA. Lock in cost on the steady floor with <strong>RIs/Savings Plans</strong>, burst with <strong>On-Demand/Spot</strong>, and rely on <strong>health checks</strong> and <strong>SG-to-SG</strong> rules for resilience and security.</p>
</blockquote>

<h1>
  
  
  MyClothes.com — Stateful, Multi-Tier, Scalable Architecture
</h1>

<h2>
  
  
  1️⃣ Problem
</h2>

<ul>
<li>Users add clothes to a <strong>shopping cart</strong>.</li>
<li>Requests can hit <strong>different EC2 instances</strong> → cart lost.</li>
<li>Goal: <strong>Keep the web tier stateless</strong> so scaling remains easy.</li>
</ul>




<h2>
  
  
  2️⃣ Solutions to Handle State
</h2>

<h3>
  
  
  <strong>A. ELB Stickiness (Session Affinity)</strong>
</h3>

<ul>
<li>ELB keeps user → same backend EC2.</li>
<li>Works short-term; simple to enable.</li>
<li>❌ Loses session if instance terminates.</li>
<li>❌ Not horizontally resilient.</li>
</ul>




<h3>
  
  
  <strong>B. Store Cart in Client Cookies</strong>
</h3>

<ul>
<li>Cart data stored in <strong>browser cookies</strong>.</li>
<li>Every request sends full cart info → any EC2 can rebuild state.</li>
<li>✅ Web tier fully stateless.</li>
<li>⚠️ Size limit: 4 KB per cookie.</li>
<li>⚠️ Security: must validate data integrity.</li>
<li>⚠️ Performance: heavier HTTP payloads.</li>
</ul>




<h3>
  
  
  <strong>C. Server-Side Sessions via ElastiCache (Redis/Memcached)</strong>
</h3>

<ul>
<li>Client keeps only <strong>Session ID</strong> (small cookie).</li>
<li>EC2 instances store/retrieve cart data in <strong>ElastiCache</strong> using that session ID.</li>
<li>✅ Millisecond latency, centralized truth.</li>
<li>✅ Survives instance rotation.</li>
<li>✅ Easy horizontal scale.</li>
<li>⚠️ Cache invalidation required; session expiry policy needed.</li>
<li>🔁 <strong>Alternative:</strong> store sessions in <strong>DynamoDB</strong> (fully managed, persistent).</li>
</ul>




<h2>
  
  
  3️⃣ Persistent User Data Layer
</h2>

<ul>
<li>Long-term info (user profiles, addresses, orders) → <strong>RDS</strong>.</li>
<li>EC2 → RDS via secure SG reference.</li>
<li>✅ Structured relational storage.</li>
<li>✅ Multi-AZ standby (automatic failover).</li>
<li>✅ Read replicas (scale reads up to 15).</li>
</ul>




<h2>
  
  
  4️⃣ Read Scaling &amp; Caching Patterns
</h2>

<h3>
  
  
  <strong>RDS Read Replicas</strong>
</h3>

<ul>
<li>Reads go to replicas → lighten primary write load.</li>
<li>Async replication delay possible (~seconds).</li>
</ul>

<h3>
  
  
  <strong>ElastiCache Lazy Loading</strong>
</h3>

<ul>
<li>EC2 → ElastiCache → if miss → fetch from RDS → populate cache.</li>
<li>✅ Faster user experience.</li>
<li>✅ Less CPU on DB.</li>
<li>⚠️ Need cache invalidation policy (TTL, write-through).</li>
</ul>




<h2>
  
  
  5️⃣ Multi-AZ &amp; High Availability
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>HA Feature</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>Route 53</td>
<td>Globally redundant</td>
<td>Alias to ALB</td>
</tr>
<tr>
<td>ALB</td>
<td>Multi-AZ</td>
<td>Public subnets</td>
</tr>
<tr>
<td>ASG</td>
<td>Multi-AZ</td>
<td>Private subnets across 2–3 AZs</td>
</tr>
<tr>
<td>RDS</td>
<td>Multi-AZ standby + replicas</td>
<td>Auto failover</td>
</tr>
<tr>
<td>ElastiCache</td>
<td>Redis Multi-AZ replication groups</td>
<td>Optional cross-AZ failover</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  6️⃣ Security Groups (SG-to-SG Reference Model)
</h2>

<ul>
<li>
<strong>ALB SG:</strong> Inbound <code>80/443</code> from <code>0.0.0.0/0</code> → Outbound to EC2 SG.</li>
<li>
<strong>EC2 SG:</strong> Inbound only from ALB SG → Outbound to ElastiCache + RDS SGs.</li>
<li>
<strong>ElastiCache SG:</strong> Inbound only from EC2 SG.</li>
<li>
<strong>RDS SG:</strong> Inbound only from EC2 SG.</li>
<li>❌ No public access to EC2/RDS/ElastiCache.</li>
</ul>




<h2>
  
  
  7️⃣ Architecture Summary (3-Tier Pattern)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>          [ Users ]
             │
        [ Route 53 ]
             │
        [ ALB (Multi-AZ) ]
             │
   ┌───────────────────────┐
   │  Auto Scaling Group   │
   │  EC2 Web Tier (Stateless)
   │   ↳ Session in ElastiCache
   │   ↳ Persistent data in RDS
   └───────────────────────┘
        │            │
 [ElastiCache]    [RDS Multi-AZ + Read Replicas]
 (session/cache)   (user, orders)
</code></pre>

</div>






<h2>
  
  
  8️⃣ Key Takeaways for Solutions Architects
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concern</th>
<th>Solution</th>
<th>AWS Component</th>
</tr>
</thead>
<tbody>
<tr>
<td>Stateless web tier</td>
<td>Offload state to cache/db</td>
<td>ElastiCache / DynamoDB</td>
</tr>
<tr>
<td>Sticky user sessions</td>
<td>ELB stickiness (temporary)</td>
<td>ALB feature</td>
</tr>
<tr>
<td>Session persistence</td>
<td>Session ID + cache</td>
<td>Redis/Memcached</td>
</tr>
<tr>
<td>Persistent data</td>
<td>Central DB</td>
<td>RDS</td>
</tr>
<tr>
<td>Scale reads</td>
<td>Read replicas / cache</td>
<td>RDS / ElastiCache</td>
</tr>
<tr>
<td>Disaster recovery</td>
<td>Multi-AZ everywhere</td>
<td>ALB, ASG, RDS, Redis</td>
</tr>
<tr>
<td>Security</td>
<td>SG chaining</td>
<td>SG references</td>
</tr>
<tr>
<td>Performance</td>
<td>Cache reads, offload RDS</td>
<td>Lazy loading pattern</td>
</tr>
<tr>
<td>Cost</td>
<td>Right-size instances, use caching</td>
<td>RI + cache hit ratio</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  9️⃣ One-Sentence Summary (Interview-Ready)
</h2>

<blockquote>
<p>MyClothes.com is a <strong>3-tier, stateless-web, stateful-data architecture</strong> using <strong>ALB + ASG + Multi-AZ RDS + ElastiCache (Redis)</strong>.<br>
Stateless EC2s hold no session; session data lives in cache; persistent data lives in RDS; all layers are Multi-AZ and secured via SG-to-SG references.</p>
</blockquote>

<h1>
  
  
  MyWordPress.com — Scalable Stateful Web App on AWS
</h1>

<h2>
  
  
  1️⃣ Overview
</h2>

<p>WordPress = <strong>stateful</strong>, because it stores:</p>

<ul>
<li>
<strong>User content</strong> → database (MySQL/Aurora)</li>
<li>
<strong>Uploaded media</strong> → shared file system (images, videos)</li>
<li>
<strong>App files/config</strong> → web tier (PHP/Apache)</li>
</ul>

<p>Goal: make it <strong>scalable, multi-AZ, and highly available</strong>.</p>




<h2>
  
  
  2️⃣ Database Layer
</h2>

<h3>
  
  
  Option 1: <strong>Amazon RDS (MySQL)</strong>
</h3>

<ul>
<li>Multi-AZ (synchronous standby)</li>
<li>Optional <strong>Read Replicas</strong> for scaling reads</li>
<li>Managed backups, patching</li>
</ul>

<h3>
  
  
  Option 2: <strong>Amazon Aurora (MySQL-compatible)</strong>
</h3>

<ul>
<li>Fully managed, auto-healing, auto-scaling</li>
<li>Multi-AZ by design</li>
<li>Up to 15 read replicas</li>
<li>Optionally <strong>Global Database</strong> for worldwide reads</li>
</ul>

<p>✅ <strong>Use Aurora</strong> if you want fewer ops and global scalability.</p>




<h2>
  
  
  3️⃣ Storage Layer: EBS vs EFS
</h2>

<h3>
  
  
  <strong>EBS (Elastic Block Store)</strong>
</h3>

<ul>
<li>Attached to <strong>one EC2 instance in one AZ</strong>
</li>
<li>Good for single-instance setup
❌ Not shared between instances
❌ Data not accessible across AZs</li>
</ul>

<h3>
  
  
  <strong>EFS (Elastic File System)</strong>
</h3>

<ul>
<li>
<strong>NFS (Network File System)</strong> managed by AWS</li>
<li>Mountable from multiple EC2s across AZs</li>
<li>Each AZ gets its own <strong>ENI (Elastic Network Interface)</strong> for EFS access</li>
<li>Files instantly available to all web servers</li>
</ul>

<p>✅ Best choice for WordPress <strong>uploads directory</strong> (<code>/wp-content/uploads</code>)</p>




<h2>
  
  
  4️⃣ Web Tier Architecture
</h2>

<ul>
<li>
<strong>ALB (Application Load Balancer)</strong> → public entry point</li>
<li>
<strong>ASG (Auto Scaling Group)</strong> → EC2 (Apache/PHP/WordPress)</li>
<li>EC2s in <strong>private subnets</strong> across ≥2 AZs</li>
<li>EC2 mounts <strong>EFS</strong> for shared media</li>
<li>EC2 connects to <strong>Aurora</strong> for posts, users, metadata</li>
<li>EC2 bootstraps via <strong>User Data</strong> script (install + config WordPress)</li>
</ul>




<h2>
  
  
  5️⃣ DNS &amp; Routing
</h2>

<ul>
<li>
<strong>Route 53 Alias</strong> → ALB DNS name</li>
<li>Users hit <code>mywordpress.com</code>
</li>
<li>ALB → target group (EC2 instances)</li>
<li>Sticky sessions <strong>not required</strong> (WordPress sessions handled via DB/cache)</li>
</ul>




<h2>
  
  
  6️⃣ High Availability &amp; Scalability
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>HA Feature</th>
<th>Scaling</th>
</tr>
</thead>
<tbody>
<tr>
<td>ALB</td>
<td>Multi-AZ</td>
<td>Automatically handles targets</td>
</tr>
<tr>
<td>ASG</td>
<td>Multi-AZ</td>
<td>Scale in/out based on CPU or request count</td>
</tr>
<tr>
<td>Aurora</td>
<td>Multi-AZ + Replicas</td>
<td>Scale reads; failover for writes</td>
</tr>
<tr>
<td>EFS</td>
<td>Multi-AZ mount targets</td>
<td>Scales automatically</td>
</tr>
<tr>
<td>Route 53</td>
<td>Multi-region DNS failover (optional)</td>
<td>Global endpoint</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  7️⃣ Security Group Design
</h2>

<ul>
<li>
<strong>ALB SG</strong>: allow inbound 80/443 from <code>0.0.0.0/0</code>; outbound → EC2 SG</li>
<li>
<strong>EC2 SG</strong>: inbound from ALB SG on port 80; outbound → EFS SG (2049), Aurora SG (3306)</li>
<li>
<strong>EFS SG</strong>: inbound from EC2 SG on 2049</li>
<li>
<strong>Aurora SG</strong>: inbound from EC2 SG on 3306</li>
</ul>




<h2>
  
  
  8️⃣ Backup &amp; Disaster Recovery
</h2>

<ul>
<li>
<strong>Aurora Backups &amp; Snapshots</strong> → daily automated + manual</li>
<li>
<strong>EFS Lifecycle Policy</strong> → move old files to EFS-IA (cheaper)</li>
<li>
<p><strong>Cross-Region Replication</strong></p>

<ul>
<li>Aurora Global DB → multi-region reads/failover</li>
<li>EFS Replication → to secondary region (optional)</li>
</ul>


</li>

</ul>




<h2>
  
  
  9️⃣ Cost Optimization
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Optimization</th>
</tr>
</thead>
<tbody>
<tr>
<td>Aurora</td>
<td>Use Serverless v2 or Reserved Instances</td>
</tr>
<tr>
<td>EFS</td>
<td>Enable Lifecycle Policy (IA tier)</td>
</tr>
<tr>
<td>EC2</td>
<td>Use Auto Scaling + Reserved/Spot Mix</td>
</tr>
<tr>
<td>ALB</td>
<td>Shared ALB for multiple apps if possible</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  🔟 Architecture Diagram (conceptual)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>             [ Users ]
                │
         [ Route 53 ]
                │
          [ ALB (Multi-AZ) ]
                │
     ┌─────────────────────────┐
     │ Auto Scaling Group (EC2)│
     │ WordPress + Apache/PHP  │
     │ Mounts shared /wp-content/uploads
     └─────────────────────────┘
          │              │
     [ Amazon EFS ]   [ Aurora MySQL ]
     Shared Media     Multi-AZ + Read Replicas
</code></pre>

</div>






<h2>
  
  
  ✅ Key Takeaways for Solution Architects
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>AWS Service</th>
<th>Why</th>
</tr>
</thead>
<tbody>
<tr>
<td>Shared file storage</td>
<td><strong>EFS</strong></td>
<td>Needed for uploads consistency</td>
</tr>
<tr>
<td>Database HA</td>
<td><strong>Aurora (MySQL)</strong></td>
<td>Simplifies scaling and failover</td>
</tr>
<tr>
<td>Load balancing</td>
<td><strong>ALB</strong></td>
<td>Multi-AZ entry, HTTP/HTTPS</td>
</tr>
<tr>
<td>Compute elasticity</td>
<td><strong>ASG (EC2)</strong></td>
<td>Scale based on demand</td>
</tr>
<tr>
<td>DNS &amp; routing</td>
<td><strong>Route 53 Alias</strong></td>
<td>Stable domain endpoint</td>
</tr>
<tr>
<td>Resilience</td>
<td>Multi-AZ + backups</td>
<td>Survives single-AZ failure</td>
</tr>
<tr>
<td>Cost control</td>
<td>Lifecycle &amp; reserved capacity</td>
<td>Efficient scaling</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  🧠 One-liner (Interview-Ready)
</h3>

<blockquote>
<p>MyWordPress.com uses <strong>ALB + Auto Scaling EC2s + EFS + Aurora MySQL (Multi-AZ)</strong> to achieve a <strong>highly available, scalable WordPress deployment</strong>, where EFS centralizes uploaded media and Aurora stores content and users.</p>
</blockquote>

<h1>
  
  
  MyWordPress.com — Scalable WordPress Architecture on AWS
</h1>

<h2>
  
  
  1️⃣ Objective
</h2>

<p>Build a <strong>fully scalable, highly available WordPress</strong> site where:</p>

<ul>
<li>Multiple EC2 instances serve the same site content.</li>
<li>Uploaded media (images, videos) are <strong>accessible from all instances</strong>.</li>
<li>Blog data (posts, users, settings) are <strong>stored in a managed database</strong>.</li>
<li>The solution is <strong>fault-tolerant and globally scalable</strong>.</li>
</ul>




<h2>
  
  
  2️⃣ Core Challenges
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Problem</th>
<th>Why it Matters</th>
</tr>
</thead>
<tbody>
<tr>
<td>Uploads stored locally (EBS)</td>
<td>Works only with one instance; fails in Multi-AZ or Auto Scaling setups.</td>
</tr>
<tr>
<td>Database consistency</td>
<td>Must be highly available and durable for posts, users, settings.</td>
</tr>
<tr>
<td>Global scalability</td>
<td>Users worldwide need fast reads and reliable writes.</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  3️⃣ Database Layer
</h2>

<h3>
  
  
  ✅ <strong>Aurora MySQL</strong>
</h3>

<ul>
<li>
<strong>Drop-in replacement</strong> for MySQL RDS.</li>
<li>
<strong>Multi-AZ</strong> by default (one primary, multiple read replicas).</li>
<li>Up to <strong>15 read replicas</strong> (faster read scaling).</li>
<li>
<strong>Global Database</strong> option for cross-region replication.</li>
<li>
<strong>Automatic failover</strong> and backups → less admin work.</li>
</ul>

<p>💡 <strong>Trade-off:</strong> Aurora costs more than RDS, but offers better performance and scalability.</p>




<h2>
  
  
  4️⃣ Storage Layer: EBS vs. EFS
</h2>

<h3>
  
  
  ❌ <strong>EBS (Elastic Block Store)</strong>
</h3>

<ul>
<li>Block storage <strong>attached to a single EC2 instance</strong> in one AZ.</li>
<li>Great for single-server setups.</li>
<li>❌ Not shareable across instances or AZs.</li>
<li>❌ Causes missing uploads when traffic is load balanced.</li>
</ul>

<h3>
  
  
  ✅ <strong>EFS (Elastic File System)</strong>
</h3>

<ul>
<li>
<strong>Managed NFS (Network File System)</strong> accessible from multiple EC2s across AZs.</li>
<li>Automatically grows/shrinks as files are added or removed.</li>
<li>
<strong>Multi-AZ architecture</strong> using <strong>ENIs</strong> (Elastic Network Interfaces) in each AZ.</li>
<li>Perfect for <code>/wp-content/uploads</code> directory in WordPress.</li>
</ul>

<p>💡 <strong>Trade-off:</strong> EFS is <strong>more expensive</strong> than EBS, but essential for <strong>multi-instance architectures</strong>.</p>




<h2>
  
  
  5️⃣ Web Tier (Application Layer)
</h2>

<ul>
<li>
<strong>Auto Scaling Group (ASG)</strong> spans <strong>multiple AZs</strong>.</li>
<li>
<strong>EC2 instances</strong>: install WordPress + Apache/PHP.</li>
<li>Mount <strong>EFS</strong> at <code>/var/www/html/wp-content/uploads</code>.</li>
<li>Connect to <strong>Aurora MySQL</strong> for data.</li>
<li>User Data bootstrap script installs WordPress automatically.</li>
<li>ALB (Application Load Balancer) distributes traffic evenly.</li>
</ul>




<h2>
  
  
  6️⃣ High-Level Architecture
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>         [ Users ]
            │
      [ Route 53 DNS ]
            │
     [ ALB (Multi-AZ) ]
            │
 ┌─────────────────────────┐
 │ Auto Scaling Group (EC2)│
 │  WordPress + Apache/PHP │
 │  Mounted EFS Volume     │
 └─────────────────────────┘
     │               │
 [Amazon EFS]     [Aurora MySQL]
 Shared Uploads   Multi-AZ DB
</code></pre>

</div>






<h2>
  
  
  7️⃣ High Availability (HA)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>HA Strategy</th>
</tr>
</thead>
<tbody>
<tr>
<td>ALB</td>
<td>Multi-AZ</td>
</tr>
<tr>
<td>ASG</td>
<td>Multi-AZ EC2s</td>
</tr>
<tr>
<td>Aurora</td>
<td>Multi-AZ + Read Replicas</td>
</tr>
<tr>
<td>EFS</td>
<td>Multi-AZ mount targets</td>
</tr>
<tr>
<td>Route 53</td>
<td>Global DNS + health checks</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  8️⃣ Security Groups
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Inbound</th>
<th>Outbound</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>ALB</strong></td>
<td>
<code>0.0.0.0/0</code> on 80/443</td>
<td>To EC2 SG</td>
</tr>
<tr>
<td><strong>EC2</strong></td>
<td>From ALB SG</td>
<td>To Aurora SG (3306), EFS SG (2049)</td>
</tr>
<tr>
<td><strong>Aurora</strong></td>
<td>From EC2 SG (3306)</td>
<td>-</td>
</tr>
<tr>
<td><strong>EFS</strong></td>
<td>From EC2 SG (2049)</td>
<td>-</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  9️⃣ Optional Enhancements
</h2>

<ul>
<li>
<strong>CloudFront + S3 Offloading:</strong>
Store and serve images from S3, cache via CloudFront CDN for global reach.</li>
<li>
<strong>Aurora Global Database:</strong>
Multi-region read replicas for global scaling.</li>
<li>
<strong>EFS Lifecycle Policy:</strong>
Move old uploads to Infrequent Access (EFS-IA) to reduce costs.</li>
<li>
<strong>AWS Backup:</strong>
Manage snapshots for Aurora and EFS centrally.</li>
</ul>




<h2>
  
  
  🔟 Cost Optimization Tips
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Optimization</th>
</tr>
</thead>
<tbody>
<tr>
<td>EC2</td>
<td>Use ASG + Spot + Reserved mix</td>
</tr>
<tr>
<td>Aurora</td>
<td>Serverless v2 or Savings Plans</td>
</tr>
<tr>
<td>EFS</td>
<td>Lifecycle to IA + burst throughput</td>
</tr>
<tr>
<td>ALB</td>
<td>Shared across multiple apps if possible</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  ✅ Key Takeaways
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>Service</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td>Shared file storage</td>
<td><strong>EFS</strong></td>
<td>Keeps uploads consistent across instances</td>
</tr>
<tr>
<td>Managed DB</td>
<td><strong>Aurora (MySQL)</strong></td>
<td>Reliable, Multi-AZ backend</td>
</tr>
<tr>
<td>Load balancing</td>
<td><strong>ALB</strong></td>
<td>Public access + even traffic</td>
</tr>
<tr>
<td>Auto scaling</td>
<td><strong>ASG (EC2)</strong></td>
<td>Elastic web tier</td>
</tr>
<tr>
<td>Multi-AZ fault tolerance</td>
<td><strong>Aurora + EFS + ALB</strong></td>
<td>Survive AZ outage</td>
</tr>
<tr>
<td>Global scalability</td>
<td><strong>Aurora Global DB / CloudFront</strong></td>
<td>Low latency reads</td>
</tr>
<tr>
<td>Cost control</td>
<td><strong>Lifecycle policies + mixed instance types</strong></td>
<td>Optimized operations</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  🧠 Interview Summary (One Sentence)
</h3>

<blockquote>
<p>MyWordPress.com uses <strong>ALB + Auto Scaling EC2s + shared EFS + Aurora MySQL (Multi-AZ)</strong> to host a <strong>highly available, scalable WordPress site</strong>, ensuring all instances share media via EFS and data via Aurora.</p>
</blockquote>

<h1>
  
  
  Quick Deployment &amp; Fast Application Instantiation in AWS
</h1>

<h2>
  
  
  1️⃣ Problem
</h2>

<p>When launching a full environment (EC2, RDS, EBS, etc.), setup can take time:</p>

<ul>
<li>Installing apps and dependencies</li>
<li>Loading data or schemas</li>
<li>Configuring settings (URLs, credentials)</li>
<li>Bootstrapping environments</li>
</ul>

<p>We want <strong>faster deployments</strong> — especially for Auto Scaling, disaster recovery, or blue/green environments.</p>




<h2>
  
  
  2️⃣ EC2 Startup Acceleration Methods
</h2>

<h3>
  
  
  <strong>A. Golden AMI</strong>
</h3>

<ul>
<li>
<p>Prebuild and bake an <strong>Amazon Machine Image</strong> (AMI) with:</p>

<ul>
<li>OS updates</li>
<li>Application code</li>
<li>Libraries/dependencies</li>
<li>Config templates</li>
</ul>


</li>

<li><p>Launch new EC2 instances directly from this <strong>Golden AMI</strong>.</p></li>

</ul>

<p>✅ <strong>Benefits</strong></p>

<ul>
<li>Instant launch (everything preinstalled)</li>
<li>Consistent configuration</li>
<li>Reduces provisioning time dramatically</li>
</ul>

<p>💡 <strong>Use Case:</strong> Auto Scaling Groups launching web servers from a pre-baked WordPress or Nginx AMI.</p>




<h3>
  
  
  <strong>B. User Data (Bootstrapping)</strong>
</h3>

<ul>
<li>
<strong>User Data</strong> scripts run automatically at instance startup.</li>
<li>
<p>Used for <strong>dynamic setup</strong> such as:</p>

<ul>
<li>Fetching credentials or endpoints</li>
<li>Configuring app environment variables</li>
<li>Registering with load balancers</li>
<li>Starting services</li>
</ul>


</li>

</ul>

<p>✅ <strong>Best Practice</strong></p>

<ul>
<li>Use <strong>User Data for dynamic config only</strong>, not heavy software installs.</li>
<li>Combine with <strong>Golden AMI</strong> for speed and flexibility.</li>
</ul>

<p>💡 <strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="c"># EC2 user data</span>
<span class="nv">DB_ENDPOINT</span><span class="o">=</span><span class="si">$(</span>aws ssm get-parameter <span class="nt">--name</span> <span class="s2">"/myapp/db_endpoint"</span> <span class="nt">--query</span> <span class="s2">"Parameter.Value"</span> <span class="nt">--output</span> text<span class="si">)</span>
<span class="nb">sed</span> <span class="nt">-i</span> <span class="s2">"s/DB_HOST_PLACEHOLDER/</span><span class="nv">$DB_ENDPOINT</span><span class="s2">/"</span> /var/www/html/config.php
systemctl restart httpd
</code></pre>

</div>






<h3>
  
  
  <strong>C. Hybrid Approach (Golden AMI + User Data)</strong>
</h3>

<ul>
<li>Base AMI already contains app code/dependencies.</li>
<li>User Data customizes runtime configuration per environment (dev, stage, prod).</li>
</ul>

<p>✅ <strong>Used by Elastic Beanstalk</strong></p>

<ul>
<li>EB internally uses a <strong>prebaked AMI</strong> + environment-level user data.</li>
</ul>




<h2>
  
  
  3️⃣ RDS: Faster Database Provisioning
</h2>

<h3>
  
  
  <strong>Use Snapshots</strong>
</h3>

<ul>
<li>
<p>Instead of creating a new empty DB and re-running migrations or inserts:</p>

<ul>
<li><strong>Restore from an RDS snapshot</strong></li>
<li>Schema and data appear instantly.</li>
</ul>


</li>

</ul>

<p>✅ <strong>Benefits</strong></p>

<ul>
<li>Fast recovery &amp; cloning</li>
<li>Ideal for DR or staging environments</li>
</ul>

<p>💡 You can even share RDS snapshots across accounts.</p>




<h2>
  
  
  4️⃣ EBS: Faster Volume Initialization
</h2>

<h3>
  
  
  <strong>Use EBS Snapshots</strong>
</h3>

<ul>
<li>Create new EBS volumes from a <strong>snapshot</strong> rather than formatting blank storage.</li>
</ul>

<p>✅ <strong>Benefits</strong></p>

<ul>
<li>Volume already formatted and preloaded with data.</li>
<li>Ideal for restoring logs, config files, or cached data quickly.</li>
</ul>

<p>💡 <strong>Tip:</strong> EBS lazy-loads snapshot data; use <code>fio</code> to pre-warm for max performance.</p>




<h2>
  
  
  5️⃣ Summary Table
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Method</th>
<th>Purpose</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>EC2</td>
<td><strong>Golden AMI</strong></td>
<td>Prebake software for fast launch</td>
<td>Nginx, PHP, WordPress baked in</td>
</tr>
<tr>
<td>EC2</td>
<td><strong>User Data</strong></td>
<td>Dynamic config at startup</td>
<td>Fetch DB endpoint, update config</td>
</tr>
<tr>
<td>EC2</td>
<td><strong>Hybrid</strong></td>
<td>Mix of both</td>
<td>Elastic Beanstalk-style setup</td>
</tr>
<tr>
<td>RDS</td>
<td><strong>Restore Snapshot</strong></td>
<td>Instant schema &amp; data recovery</td>
<td>Restore production backup</td>
</tr>
<tr>
<td>EBS</td>
<td><strong>Snapshot Restore</strong></td>
<td>Preformatted, preloaded volume</td>
<td>Mount and run instantly</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  6️⃣ Exam/Interview Key Takeaways
</h2>

<ul>
<li>
<strong>Golden AMI:</strong> Fastest EC2 provisioning → identical servers at scale.</li>
<li>
<strong>User Data:</strong> Lightweight config → runtime flexibility.</li>
<li>
<strong>Snapshots:</strong> Quick restore → minimal downtime for DBs and disks.</li>
<li>
<strong>Elastic Beanstalk:</strong> Real-world example of <strong>AMI + User Data hybrid</strong>.</li>
<li>
<strong>Best Practice:</strong> Keep repeatable installs in AMIs, dynamic info in boot scripts.</li>
</ul>




<h3>
  
  
  🧠 One-Liner Summary
</h3>

<blockquote>
<p>To launch environments faster, use <strong>Golden AMIs</strong> for preinstalled apps, <strong>User Data</strong> for dynamic config, and restore <strong>RDS/EBS from snapshots</strong> instead of rebuilding from scratch.</p>
</blockquote>

<h1>
  
  
  Elastic Beanstalk (EB) — Simplifying Application Deployment
</h1>

<h2>
  
  
  1️⃣ The Challenge
</h2>

<p>Every web app we built so far included:</p>

<ul>
<li><strong>ALB (Load Balancer)</strong></li>
<li>
<strong>Auto Scaling Group (ASG)</strong> with EC2s across AZs</li>
<li>
<strong>RDS (database)</strong> and sometimes <strong>ElastiCache (cache)</strong>
</li>
</ul>

<p>👉 Building this <strong>manually every time</strong> = complex and slow.<br>
👉 Developers want to <strong>focus on code</strong>, not infrastructure.</p>


<h2>
  
  
  2️⃣ What Is Elastic Beanstalk?
</h2>

<p><strong>Elastic Beanstalk (EB)</strong> is a <strong>developer-centric PaaS</strong> that:</p>

<ul>
<li>Automatically provisions the underlying AWS resources (EC2, ASG, ELB, RDS, etc.)</li>
<li>Deploys and manages your <strong>application code</strong>
</li>
<li>Handles <strong>capacity, scaling, monitoring, and health checks</strong>
</li>
<li>Lets you still retain <strong>full access to the underlying resources</strong>
</li>
</ul>

<p>✅ You pay only for <strong>the resources</strong> (EC2, RDS, etc.),<br>
not for Beanstalk itself — <strong>Beanstalk is free.</strong></p>


<h2>
  
  
  3️⃣ Key Components
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Application</strong></td>
<td>Logical container for your code, configurations, and environments</td>
</tr>
<tr>
<td><strong>Application Version</strong></td>
<td>A specific, uploaded version of your code (e.g., <code>v1</code>, <code>v2</code>)</td>
</tr>
<tr>
<td><strong>Environment</strong></td>
<td>Running version of your app + its resources (ASG, ELB, EC2, RDS)</td>
</tr>
<tr>
<td><strong>Configuration Template</strong></td>
<td>Blueprint defining environment type, instance size, scaling, etc.</td>
</tr>
</tbody>
</table></div>

<p>You can have multiple environments under one app, e.g.:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>MyApp
 ├── Dev Environment
 ├── Test Environment
 └── Prod Environment
</code></pre>

</div>






<h2>
  
  
  4️⃣ Beanstalk Tiers
</h2>

<h3>
  
  
  <strong>A. Web Server Environment Tier</strong>
</h3>

<ul>
<li>Standard web apps accessed via HTTP/HTTPS.</li>
<li>
<strong>Architecture:</strong>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  Users → ALB → ASG (EC2 Web Servers)
</code></pre>

</div>



<ul>
<li>EC2s run your app (PHP, Node.js, Java, etc.).</li>
<li>Scales based on load metrics (CPU, latency).</li>
</ul>

<h3>
  
  
  <strong>B. Worker Environment Tier</strong>
</h3>

<ul>
<li>Background or asynchronous jobs.</li>
<li>Messages are sent to <strong>SQS queue</strong>, consumed by worker EC2s.</li>
<li>Scales based on <strong>SQS queue length</strong>.</li>
<li>Ideal for batch processing, email sending, video conversion, etc.</li>
</ul>

<p>💡 You can <strong>connect</strong> both tiers:<br>
Web app pushes messages → Worker environment processes them.</p>




<h2>
  
  
  5️⃣ Deployment Options
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Mode</th>
<th>Description</th>
<th>Best For</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Single Instance</strong></td>
<td>One EC2 instance with optional EIP &amp; RDS</td>
<td>Dev/Test</td>
</tr>
<tr>
<td><strong>High Availability</strong></td>
<td>Multi-AZ ASG + ALB + optional RDS (Multi-AZ)</td>
<td>Production</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  6️⃣ Supported Platforms
</h2>

<p>Elastic Beanstalk supports major programming stacks:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Language / Platform</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td>Go</td>
<td>Web backend</td>
</tr>
<tr>
<td>Java SE / Tomcat</td>
<td>Spring Boot apps</td>
</tr>
<tr>
<td>.NET Core / .NET on Windows</td>
<td>Enterprise apps</td>
</tr>
<tr>
<td>Node.js</td>
<td>REST APIs</td>
</tr>
<tr>
<td>PHP</td>
<td>WordPress</td>
</tr>
<tr>
<td>Python</td>
<td>Flask, Django</td>
</tr>
<tr>
<td>Ruby</td>
<td>Rails</td>
</tr>
<tr>
<td>Docker (Single or Multi-Container)</td>
<td>Any containerized app</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  7️⃣ Deployment Lifecycle
</h2>

<ol>
<li><strong>Create Application</strong></li>
<li>
<strong>Upload Version</strong> (ZIP or WAR or Docker image)</li>
<li>
<strong>Launch Environment</strong> (web or worker tier)</li>
<li><strong>Manage Lifecycle</strong></li>
</ol>

<ul>
<li>Scale up/down</li>
<li>Monitor health</li>
<li>Update configurations

<ol>
<li>
<strong>Update Version</strong> (upload new code → deploy)</li>
</ol>
</li>
</ul>




<h2>
  
  
  8️⃣ Scaling &amp; Health
</h2>

<ul>
<li>
<strong>Auto Scaling</strong>: Adjusts EC2 capacity automatically.</li>
<li>
<strong>Elastic Load Balancer</strong>: Distributes traffic evenly.</li>
<li>
<strong>Health Monitoring</strong>: Beanstalk dashboard + CloudWatch integration.</li>
<li>
<strong>Rolling / Blue-Green Deployments</strong> supported for zero downtime.</li>
</ul>




<h2>
  
  
  9️⃣ Pricing Model
</h2>

<ul>
<li>
<strong>Elastic Beanstalk service itself</strong> → $0</li>
<li>
<p>You pay for:</p>

<ul>
<li>EC2 instances</li>
<li>Load balancer</li>
<li>RDS</li>
<li>EBS volumes</li>
<li>Data transfer</li>
</ul>


</li>

</ul>




<h2>
  
  
  🔟 Architecture Summary
</h2>

<h3>
  
  
  Web Tier (High Availability)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>           [ Users ]
              │
          [ ALB ]
              │
   ┌──────────────────────┐
   │  ASG (EC2 Instances) │
   │  WordPress / AppCode │
   └──────────────────────┘
              │
         [ RDS Database ]
</code></pre>

</div>



<h3>
  
  
  Worker Tier (Optional)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[ Web Tier ] → [ SQS Queue ] → [ Worker ASG (EC2s) ]
</code></pre>

</div>






<h2>
  
  
  ✅ Key Takeaways
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Elastic Beanstalk</strong></td>
<td>Simplifies app deployment (PaaS)</td>
</tr>
<tr>
<td><strong>Web vs Worker Tier</strong></td>
<td>Web = front-end, Worker = background jobs</td>
</tr>
<tr>
<td><strong>Hybrid Deployment</strong></td>
<td>Web pushes messages to Worker via SQS</td>
</tr>
<tr>
<td><strong>Scaling</strong></td>
<td>Auto Scaling + Load Balancer</td>
</tr>
<tr>
<td><strong>Deployment Choices</strong></td>
<td>Single instance (dev) or Multi-AZ HA (prod)</td>
</tr>
<tr>
<td><strong>Languages Supported</strong></td>
<td>Go, Java, .NET, Node.js, PHP, Python, Ruby, Docker</td>
</tr>
<tr>
<td><strong>Cost</strong></td>
<td>Only underlying resources are billed</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  🧠 One-Liner (Interview Summary)
</h3>

<blockquote>
<p>Elastic Beanstalk is AWS’s PaaS that lets developers deploy code fast without managing infrastructure — it automatically provisions <strong>ALB, ASG, EC2, RDS</strong>, supports <strong>multi-language apps</strong>, and offers <strong>web + worker tiers</strong> for scalable architectures.</p>
</blockquote>

