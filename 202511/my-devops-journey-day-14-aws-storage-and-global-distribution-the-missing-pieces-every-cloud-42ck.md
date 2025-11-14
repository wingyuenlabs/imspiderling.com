---
Title: My DevOps Journey: Day 14 - AWS Storage and Global Distribution: The Missing Pieces Every Cloud Engineer Must Master
Description: 
Author: Sheersh Sinha
Date: 2025-11-14T21:42:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>In <a href="https://dev.to/sheershsinha/my-devops-journey-part-13-aws-s3-essentials-the-backbone-of-cloud-storage-10ln">Day 13</a>, I explored how Amazon S3 changed the way I think about storage - versioning, lifecycle policies, encryption, and automation taught me that <strong>data durability is the backbone of cloud systems</strong>.</p>

<p>But as my sandbox grew into a multi-instance setup, I faced a new reality:</p>

<blockquote>
<p>S3 alone cannot solve all storage needs.<br><br>
Some data must live with compute.<br><br>
Some must be shared across servers.<br><br>
Some must be queried.<br><br>
Some must be protected.<br><br>
And some must be delivered globally with low latency.</p>
</blockquote>

<p>This realization pushed me into a deeper AWS chapter: <strong>EBS, EFS, FSx, RDS, DynamoDB, Route 53, and CloudFront</strong>.</p>

<p>Each one filled a gap I didn't even know existed - until the day I faced a real problem. Let me take you through that journey.</p>




<h2>
  
  
  1. When My EC2 Lost Its Data - The EBS Lesson
</h2>

<p>One day, I rebooted my EC2 instance… and my application files were gone.</p>

<p>I had forgotten the most basic AWS truth:</p>

<blockquote>
<p><strong>"EC2 storage disappears unless you attach persistent volumes."</strong></p>
</blockquote>

<p>That's when I discovered <strong>EBS (Elastic Block Store)</strong> - block storage volumes that live independently of the EC2 instance.</p>

<h3>
  
  
  What I learned about EBS:
</h3>

<ul>
<li>Persistent even if the EC2 is stopped</li>
<li>Can be detached &amp; attached to any EC2</li>
<li>Great for OS boot volumes, DBs, or app data</li>
</ul>

<h3>
  
  
  Hands-on Commands:
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe3yw1dldkob8ckkjy4g8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe3yw1dldkob8ckkjy4g8.png" alt=" " width="800" height="211"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7b1l2r64byx3w2gimqds.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7b1l2r64byx3w2gimqds.png" alt=" " width="800" height="270"></a></p>

<p><strong>Key Lesson:</strong> Compute and storage must be decoupled for reliability.</p>




<h2>
  
  
  2. When Multiple EC2s Needed Shared Storage - Enter EFS
</h2>

<p>Soon, I deployed two EC2 instances behind a load balancer.</p>

<p>But then something broke:</p>

<ul>
<li>One instance had updated config files</li>
<li>The other still used old ones</li>
<li>My app behaved inconsistently</li>
</ul>

<p>That's when I needed shared file storage → <strong>Amazon EFS</strong></p>

<h3>
  
  
  Why EFS was a game changer:
</h3>

<ul>
<li>Multiple EC2s can read/write the same files</li>
<li>Perfect for web servers, content directories, shared uploads</li>
<li>Auto-scaling storage</li>
<li>Pay only for what you use</li>
</ul>

<h3>
  
  
  Mount EFS:
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpggam4o8kcu8eg43g5cp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpggam4o8kcu8eg43g5cp.png" alt=" " width="800" height="43"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgj9lw44etwxihp4i1nsz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgj9lw44etwxihp4i1nsz.png" alt=" " width="800" height="117"></a></p>

<p>EFS solved consistency issues instantly. No more "it works on server 1, but not server 2."</p>




<h2>
  
  
  3. Snapshots &amp; Disaster Recovery - EBS Snapshots
</h2>

<p>Later, I needed safe backups before making risky changes.</p>

<p>That's when I learned the magic of:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx1xwg9bu9c5w5li0fhrd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx1xwg9bu9c5w5li0fhrd.png" alt=" " width="593" height="531"></a></p>

<ul>
<li>Snapshots are incremental &amp; stored in S3 internally</li>
<li>If anything breaks: restore → create new volume → attach</li>
</ul>

<blockquote>
<p><strong>"Always protect your state before making changes."</strong></p>
</blockquote>




<h2>
  
  
  4. High-Performance File Systems - FSx
</h2>

<p>AWS FSx came next. At first, I ignored it - until I learned it powers:</p>

<ul>
<li>High-performance workloads</li>
<li>Machine learning pipelines</li>
<li>Windows application file shares</li>
<li>Data-intensive HPC workloads</li>
</ul>

<blockquote>
<p>Use EFS for Linux. Use FSx for anything heavy-duty.</p>
</blockquote>




<h2>
  
  
  5. When My App Needed a Database That Wouldn't Crash - RDS
</h2>

<p>I deployed MySQL on EC2 once. <strong>Bad idea.</strong></p>

<p>Maintenance, backups, failover… all on me.</p>

<p>Then I discovered <strong>RDS</strong> - AWS-managed SQL databases.</p>

<h3>
  
  
  RDS gives you:
</h3>

<ul>
<li>Automated backups</li>
<li>Multi-AZ replication</li>
<li>Auto patching</li>
<li>Read replicas</li>
<li>Monitoring</li>
<li>Snapshots</li>
<li>Encryption</li>
</ul>

<h3>
  
  
  Launching RDS:
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft6a9ylv8slbgwtsef1rt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft6a9ylv8slbgwtsef1rt.png" alt=" " width="690" height="621"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx2hfcd93fuiytl13uz4v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx2hfcd93fuiytl13uz4v.png" alt=" " width="487" height="370"></a></p>

<p>Suddenly, my backend was stable, consistent, and safe.</p>




<h2>
  
  
  6. When Performance Outgrew SQL - DynamoDB Saved Me
</h2>

<p>A scenario happened:</p>

<p>My app needed:</p>

<ul>
<li>Millisecond reads</li>
<li>Millions of requests</li>
<li>No downtime</li>
<li>No maintenance</li>
</ul>

<p>Enter <strong>DynamoDB</strong> - serverless NoSQL at massive scale.</p>

<h3>
  
  
  DynamoDB strengths:
</h3>

<ul>
<li>Auto-scaling read/write</li>
<li>Global tables</li>
<li>TTL for auto-expiry</li>
<li>On-demand pricing</li>
<li>Zero maintenance</li>
</ul>

<p>DynamoDB made my backend <strong>"infinitely scalable"</strong> without touching servers.</p>




<h2>
  
  
  7. When My App Needed a Domain - Route 53
</h2>

<p>Next problem: I needed a custom domain for my app.</p>

<p><strong>Route 53</strong> gave me:</p>

<ul>
<li>DNS records</li>
<li>Traffic routing</li>
<li>Health checks</li>
<li>Failover routing</li>
<li>Latency-based routing</li>
</ul>

<p>Now my app wasn't just functional - it had an online identity.</p>




<h2>
  
  
  8. When I Needed Global Speed - CloudFront Saved My Bandwidth
</h2>

<p>Last issue: My images, CSS files, and videos loaded slowly across regions.</p>

<p><strong>CloudFront</strong> fixed everything.</p>

<h3>
  
  
  CloudFront advantages:
</h3>

<ul>
<li>Distributes content globally</li>
<li>Reduces latency</li>
<li>Caches static files</li>
<li>Secures content with HTTPS</li>
<li>Integrates with S3, EC2, and Load Balancers</li>
<li>DDoS protection via AWS Shield</li>
</ul>

<p>Now my system felt "global." Users anywhere experienced fast performance.</p>




<h2>
  
  
  Final Architecture (Day 14 Vision)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Layer</strong></th>
<th><strong>Service</strong></th>
<th><strong>Purpose</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>CDN</td>
<td>CloudFront</td>
<td>Global CDN Delivery</td>
</tr>
<tr>
<td>DNS/Domain</td>
<td>Route 53</td>
<td>Domain &amp; Routing</td>
</tr>
<tr>
<td>Load Balancer</td>
<td>ELB</td>
<td>Distributes Traffic</td>
</tr>
<tr>
<td>Web Servers</td>
<td>EC2 (EFS)</td>
<td>Compute/Shared Filesystem</td>
</tr>
<tr>
<td>Database</td>
<td>RDS / DynamoDB</td>
<td>Managed DB / NoSQL at Scale</td>
</tr>
<tr>
<td>Storage</td>
<td>S3, EBS, FSx/EFS</td>
<td>Object/Block/File Storage</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  What I Learned (Deep DevOps Mindset)
</h2>

<ul>
<li>
<strong>Don't trust EC2 storage</strong> - use EBS</li>
<li>
<strong>Don't duplicate files across instances</strong> - use EFS</li>
<li>
<strong>Don't run SQL manually</strong> - use RDS</li>
<li>
<strong>Don't force SQL for all workloads</strong> - use DynamoDB</li>
<li>
<strong>Don't expose raw S3</strong> - use CloudFront</li>
<li>
<strong>Don't buy domains elsewhere</strong> - Route 53 is built for AWS</li>
<li>
<strong>Don't skip backups</strong> - use Snapshots</li>
</ul>

<blockquote>
<p>This blog wasn't just content - it was an evolution in the way I think about designing cloud systems.</p>
</blockquote>

<p><strong>"DevOps isn't about tools - it's about building systems that don't break when life gets busy."</strong></p>




<h2>
  
  
  What's Next (Day 15 - Serverless + Observability)
</h2>

<p><strong>Coming up next:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Theme</strong></th>
<th><strong>Topics</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Serverless</td>
<td>AWS Lambda, Triggers, Events, Destinations</td>
</tr>
<tr>
<td>Monitoring</td>
<td>CloudWatch Metrics &amp; Logs, Dashboards, Alarms</td>
</tr>
<tr>
<td>Alerts</td>
<td>Amazon SNS, SQS message queues</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p>This is where automation becomes truly event-driven and intelligent.</p>
</blockquote>

