---
Title: CloudFront and AWS global accelerator
Description: 
Author: Aisalkyn Aidarova
Date: 2025-10-28T22:02:51.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  <strong>1. What CloudFront Is</strong>
</h3>

<ul>
<li>
<strong>Type:</strong> Content Delivery Network (CDN)</li>
<li>
<strong>Purpose:</strong> Improves <strong>read performance</strong> and reduces <strong>latency</strong> by <strong>caching website content</strong> at AWS <strong>edge locations</strong> around the world.</li>
<li>
<strong>Global Infrastructure:</strong> 200+ <strong>Points of Presence (PoPs)</strong> including <strong>Edge Locations</strong> and <strong>Regional Edge Caches</strong>.</li>
</ul>




<h3>
  
  
  <strong>2. How It Works</strong>
</h3>

<ol>
<li>
<strong>User requests</strong> content (e.g., image, video, or webpage).</li>
<li><strong>Edge location checks cache:</strong></li>
</ol>

<ul>
<li>If <strong>content is cached</strong>, it’s delivered immediately (low latency).</li>
<li>If <strong>not cached</strong>, CloudFront fetches it from the <strong>origin</strong> (S3, EC2, ALB, or custom HTTP server).

<ol>
<li>The <strong>response</strong> is then <strong>cached</strong> locally for future users.</li>
</ol>
</li>
</ul>




<h3>
  
  
  <strong>3. CloudFront Origins</strong>
</h3>

<ul>
<li>
<p><strong>Amazon S3</strong> → for static files and websites.</p>

<ul>
<li>Secured with <strong>Origin Access Control (OAC)</strong> (replaces the old OAI).</li>
</ul>


</li>

<li><p><strong>HTTP/HTTPS servers</strong> → for dynamic sites or APIs.</p></li>

<li><p><strong>Load Balancers / EC2 / ECS / EKS</strong> → for private VPC applications.</p></li>

<li><p><strong>Custom Origins</strong> → any HTTP-based service inside or outside AWS.</p></li>

</ul>




<h3>
  
  
  <strong>4. Security Features</strong>
</h3>

<ul>
<li>
<strong>OAC (Origin Access Control)</strong>: Restricts direct S3 bucket access.</li>
<li>
<strong>AWS Shield Standard</strong>: Built-in DDoS protection.</li>
<li>
<strong>AWS WAF (Web Application Firewall)</strong>: Filters malicious requests.</li>
<li>
<strong>HTTPS / TLS</strong>: End-to-end encryption from viewer → CloudFront → origin.</li>
</ul>




<h3>
  
  
  <strong>5. CloudFront vs S3 Cross-Region Replication</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>CloudFront (CDN)</th>
<th>S3 Cross-Region Replication</th>
</tr>
</thead>
<tbody>
<tr>
<td>Purpose</td>
<td>Cache static/dynamic content for global users</td>
<td>Replicate entire buckets for disaster recovery</td>
</tr>
<tr>
<td>Reach</td>
<td>216+ global edge locations</td>
<td>Only the regions you configure</td>
</tr>
<tr>
<td>Latency</td>
<td>Very low (served from nearest edge)</td>
<td>Depends on region proximity</td>
</tr>
<tr>
<td>Updates</td>
<td>Cached, refreshed periodically</td>
<td>Real-time replication</td>
</tr>
<tr>
<td>Best for</td>
<td><strong>Static assets, global apps, media</strong></td>
<td><strong>Backups, compliance, DR</strong></td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  <strong>6. Benefits</strong>
</h3>

<p>✅ Low latency<br>
✅ Global availability<br>
✅ Scalability<br>
✅ Security (DDoS, WAF)<br>
✅ Cost-effective caching (reduced origin load)</p>
<h2>
  
  
  <strong>1. Step-by-Step Recap</strong>
</h2>
<h3>
  
  
  <strong>Step 1 — Create an S3 Bucket</strong>
</h3>

<ul>
<li>
<strong>Bucket name:</strong> <code>demo-cloudfront-stephane-v4</code>
</li>
<li>Default settings (no public access).</li>
<li>
<p>Uploaded:</p>

<ul>
<li><code>index.html</code></li>
<li><code>beach.jpg</code></li>
<li><code>coffee.jpg</code></li>
</ul>
</li>
</ul>

<p>👉 <strong>Purpose:</strong> Store static website files (HTML, images, CSS, JS, etc.) privately — not public.</p>


<h3>
  
  
  <strong>Step 2 — Test Access from S3</strong>
</h3>

<ul>
<li>
<strong>Object URL</strong> (unauthenticated): ❌ <em>AccessDenied</em> — because public access is blocked.</li>
<li>
<strong>Open via S3 console</strong>: ✅ Works via <strong>pre-signed URL</strong>, which is temporary and private.</li>
</ul>

<p>👉 <strong>Takeaway:</strong> Objects remain <strong>private</strong>, which is good practice for security.</p>


<h3>
  
  
  <strong>Step 3 — Create a CloudFront Distribution</strong>
</h3>

<ul>
<li>
<strong>Type:</strong> Single website or app.</li>
<li>
<strong>Origin type:</strong> Amazon S3.</li>
<li>
<strong>Origin:</strong> Selected the S3 bucket (<code>demo-cloudfront-stephane-v4</code>).</li>
<li>
<strong>Origin path:</strong> <em>(blank)</em> since files are in the root.</li>
<li>
<strong>Access:</strong> Enabled <strong>Private S3 Bucket Access</strong> → CloudFront creates <strong>Origin Access Control (OAC)</strong> automatically.</li>
<li>
<strong>Distribution domain:</strong> Uses default CloudFront domain (e.g., <code>d1abcde123.cloudfront.net</code>).</li>
</ul>

<p>👉 <strong>Purpose:</strong> Let CloudFront fetch files <em>privately</em> from S3 without exposing them publicly.</p>


<h3>
  
  
  <strong>Step 4 — CloudFront Configures Security</strong>
</h3>

<p>CloudFront automatically:</p>

<ul>
<li>Creates an <strong>OAC</strong> (Origin Access Control).</li>
<li>Adds a new <strong>bucket policy</strong> to S3:
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="w">  </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Principal"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Service"</span><span class="p">:</span><span class="w"> </span><span class="s2">"cloudfront.amazonaws.com"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"s3:GetObject"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:s3:::demo-cloudfront-stephane-v4/*"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"StringEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"AWS:SourceArn"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span></code></pre>

</div>


<p>👉 <strong>Meaning:</strong> Only <strong>this specific CloudFront distribution</strong> can read objects from the S3 bucket.</p>


<h3>
  
  
  <strong>Step 5 — Wait for Deployment</strong>
</h3>

<ul>
<li>Status: <em>Deploying → Deployed</em> (takes 5–10 minutes typically).</li>
<li>Once deployed, use the <strong>CloudFront domain name</strong>, e.g.:
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  https://d1abcde123.cloudfront.net/index.html
  https://d1abcde123.cloudfront.net/coffee.jpg
</code></pre>

</div>


<p>👉 <strong>Result:</strong> Files load instantly via the <strong>nearest edge location</strong>.</p>


<h2>
  
  
  <strong>2. What’s Happening Behind the Scenes</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Action</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>1. User requests <code>coffee.jpg</code>
</td>
<td>Request goes to the nearest <strong>edge location</strong>
</td>
</tr>
<tr>
<td>2. CloudFront checks cache</td>
<td>If not cached, it fetches from the <strong>S3 origin</strong> securely</td>
</tr>
<tr>
<td>3. Object cached</td>
<td>Cached for faster delivery next time</td>
</tr>
<tr>
<td>4. Second user (same region)</td>
<td>Served instantly from the <strong>edge cache</strong>, not S3</td>
</tr>
<tr>
<td>5. Private access maintained</td>
<td>S3 bucket is never exposed publicly</td>
</tr>
</tbody>
</table></div>


<h2>
  
  
  <strong>3. Key Benefits of This Setup</strong>
</h2>

<p>✅ <strong>Security:</strong> S3 stays private (no public ACLs).<br>
✅ <strong>Speed:</strong> Edge caching reduces latency worldwide.<br>
✅ <strong>Cost optimization:</strong> Reduced S3 GET requests.<br>
✅ <strong>Scalability:</strong> Automatic high availability and DDoS protection.<br>
✅ <strong>Flexibility:</strong> Can later attach WAF, custom domain, SSL certificate (ACM).</p>


<h2>
  
  
  <strong>4. Exam &amp; Real-World Tips</strong>
</h2>

<ul>
<li>
<strong>OAC (Origin Access Control)</strong> is the <strong>modern</strong> approach.
Don’t use OAI (Origin Access Identity) — it’s legacy.</li>
<li>
<strong>Viewer Protocol Policy:</strong> Always set to <strong>Redirect HTTP → HTTPS</strong> for production.</li>
<li>
<strong>Cache Behavior:</strong> Default TTL, Max TTL, and Min TTL affect cost and performance.</li>
<li>
<strong>Custom Domain:</strong> Use Route 53 + ACM certificate if you want your own domain.</li>
<li>
<strong>Invalidations:</strong> Used to remove outdated cache objects (<code>/*</code> to clear all).</li>
</ul>
<h2>
  
  
  <strong>1. CloudFront + VPC Origin (Modern &amp; Recommended Method)</strong>
</h2>
<h3>
  
  
  <strong>Purpose</strong>
</h3>

<p>Connect CloudFront <strong>securely</strong> to <strong>private applications</strong> hosted inside a <strong>VPC</strong>, such as:</p>

<ul>
<li><strong>Application Load Balancer (ALB)</strong></li>
<li><strong>Network Load Balancer (NLB)</strong></li>
<li><strong>EC2 Instances in private subnets</strong></li>
</ul>

<p>All traffic between CloudFront and your backend stays <strong>within AWS’s private network</strong> — never exposed to the public Internet.</p>


<h3>
  
  
  <strong>Architecture Overview</strong>
</h3>

<p><strong>User → CloudFront (Edge) → VPC Origin → Private ALB/EC2/NLB</strong></p>

<ul>
<li>
<strong>Users</strong> connect to the CloudFront distribution (global edge network).</li>
<li>
<strong>CloudFront</strong> uses a <strong>VPC Origin</strong> to reach into your <strong>private VPC</strong>.</li>
<li>
<strong>Private ALB/EC2/NLB</strong> processes requests internally.</li>
<li>
<strong>No public IPs</strong> are required on backend resources.</li>
</ul>


<h3>
  
  
  <strong>How to Configure</strong>
</h3>

<ol>
<li>
<strong>Create your application</strong> in a private subnet (ALB/EC2/NLB).</li>
</ol>

<ul>
<li>
<p>Security groups allow CloudFront to access the target via the VPC origin endpoint.</p>

<ol>
<li><strong>Go to CloudFront → Create Distribution</strong></li>
</ol>
</li>
<li><p><strong>Origin Type:</strong> VPC Origin</p></li>
<li><p><strong>Select VPC:</strong> choose your VPC</p></li>
<li>
<p><strong>Origin Endpoint:</strong> your ALB or instance endpoint inside that VPC</p>

<ol>
<li><strong>Set Access Control:</strong></li>
</ol>
</li>
<li><p>No public access to ALB or instance.</p></li>
<li>
<p>CloudFront handles all traffic via internal networking.</p>

<ol>
<li>
<strong>Deploy</strong> the distribution.
CloudFront automatically creates and manages a <strong>private link</strong> to your VPC.</li>
</ol>
</li>
</ul>


<h3>
  
  
  <strong>Advantages</strong>
</h3>

<p>✅ <strong>Private communication</strong> — no need for public subnets or internet-facing endpoints.<br>
✅ <strong>Stronger security</strong> — no IP whitelisting or manual security-group edits.<br>
✅ <strong>Simpler maintenance</strong> — CloudFront manages connectivity.<br>
✅ <strong>Lower latency</strong> — stays on AWS’s internal backbone network.<br>
✅ <strong>Compliance-friendly</strong> — avoids public exposure entirely.</p>


<h2>
  
  
  <strong>2. Legacy Method (Pre-VPC Origin)</strong>
</h2>

<p>Before VPC origins, you had to:</p>

<ul>
<li>Make your <strong>ALB or EC2 instance public</strong>.</li>
<li>Obtain the <strong>list of all CloudFront edge IP ranges</strong> (from AWS JSON file).</li>
<li>Add them manually to your <strong>security groups</strong> to restrict access.</li>
</ul>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Example security group rule (old way)</span>
Type: HTTP
Source: &lt;CloudFront Edge IP Range&gt;
</code></pre>

</div>



<h3>
  
  
  <strong>Drawbacks</strong>
</h3>

<p>❌ Tedious and error-prone (IP ranges change over time).<br>
❌ Requires <strong>public exposure</strong> of ALB or EC2.<br>
❌ Security risk if someone modifies security groups.<br>
❌ More maintenance and less automation.</p>




<h2>
  
  
  <strong>3. Summary Table</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th><strong>VPC Origin (New)</strong></th>
<th><strong>Public Origin (Old)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Exposure</td>
<td>Private (no Internet)</td>
<td>Public</td>
</tr>
<tr>
<td>Setup</td>
<td>Automatic via CloudFront</td>
<td>Manual IP whitelist</td>
</tr>
<tr>
<td>Security</td>
<td>AWS internal network</td>
<td>Internet-facing</td>
</tr>
<tr>
<td>Maintenance</td>
<td>Minimal</td>
<td>High</td>
</tr>
<tr>
<td>Best for</td>
<td>Private ALBs, EC2s, APIs</td>
<td>Legacy or hybrid setups</td>
</tr>
</tbody>
</table></div>




<p>✅ <strong>Exam Tip:</strong><br>
If you see a question saying <em>“Deliver content from private subnets without making ALB public”</em>,<br>
→ <strong>Answer:</strong> Use <strong>CloudFront with a VPC Origin</strong>.</p>

<h3>
  
  
  <strong>CloudFront Geo Restriction (Geo Blocking)</strong>
</h3>

<p><strong>Definition:</strong><br>
Geo Restriction in CloudFront allows you to <strong>control access</strong> to your content <strong>based on the viewer’s country</strong>.</p>




<h3>
  
  
  <strong>How It Works</strong>
</h3>

<ul>
<li>CloudFront determines the user’s <strong>country</strong> using a <strong>third-party Geo-IP database</strong> (based on the requester’s IP address).</li>
<li>
<p>You can then either:</p>

<ul>
<li>✅ <strong>Allowlist</strong> specific countries → only these can access content.</li>
<li>🚫 <strong>Blocklist</strong> specific countries → all others can access.</li>
</ul>


</li>

</ul>




<h3>
  
  
  <strong>Use Cases</strong>
</h3>

<ul>
<li>
<strong>Copyright and licensing compliance</strong> (e.g., streaming restrictions by region)</li>
<li>
<strong>Regulatory or export control</strong> requirements</li>
<li>
<strong>Security</strong> — blocking known high-risk regions</li>
</ul>




<h3>
  
  
  <strong>How to Enable (Console Path)</strong>
</h3>

<ol>
<li>Open your <strong>CloudFront Distribution</strong>.</li>
<li>Go to <strong>Security → Geographic Restrictions</strong>.</li>
<li>Click <strong>Edit</strong>.</li>
<li>Choose:</li>
</ol>

<ul>
<li>
<strong>Allowlist</strong> → specify countries allowed.</li>
<li>
<strong>Blocklist</strong> → specify countries denied.

<ol>
<li><strong>Save changes.</strong></li>
</ol>
</li>
</ul>




<h3>
  
  
  <strong>Example</strong>
</h3>

<p>If you choose an <strong>allowlist</strong> for <strong>India</strong> and <strong>United States</strong>,<br>
only viewers from those countries can access your distribution; all others receive an HTTP <code>403 Forbidden</code>.</p>




<p>✅ <strong>Key Exam Tip:</strong><br>
Geo Restriction applies <strong>at the CloudFront level</strong>, not at the origin (like S3). It’s enforced <strong>before</strong> requests reach your backend.</p>

<h3>
  
  
  <strong>CloudFront Pricing &amp; Price Classes</strong>
</h3>

<p><strong>Concept:</strong><br>
CloudFront delivers content via <strong>edge locations</strong> all over the world.<br>
Because bandwidth costs differ by region, <strong>data transfer pricing</strong> also varies.</p>




<h3>
  
  
  <strong>1. Data Transfer Pricing</strong>
</h3>

<ul>
<li>Each <strong>region or continent</strong> has its own <strong>per-GB cost</strong>.</li>
<li>
<p>Example:</p>

<ul>
<li>🇺🇸 <strong>US / Canada / Mexico</strong> → ~$0.085 / GB (first 10 TB)</li>
<li>🇮🇳 <strong>India</strong> → ~$0.17 / GB (about twice the price)</li>
</ul>


</li>

<li><p>The <strong>more data</strong> you transfer, the <strong>lower</strong> the per-GB cost (volume discount).</p></li>

</ul>




<h3>
  
  
  <strong>2. Price Classes</strong>
</h3>

<p>To control costs, you can <strong>limit which edge locations</strong> CloudFront uses:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Price Class</strong></th>
<th><strong>Edge Coverage</strong></th>
<th><strong>Cost</strong></th>
<th><strong>Best For</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>All</strong></td>
<td>All global edge locations</td>
<td>💲💲💲 (highest)</td>
<td>Best performance, global users</td>
</tr>
<tr>
<td><strong>200</strong></td>
<td>Most regions (excludes most expensive)</td>
<td>💲💲</td>
<td>Balanced cost vs reach</td>
</tr>
<tr>
<td><strong>100</strong></td>
<td>Only North America &amp; Europe</td>
<td>💲 (lowest)</td>
<td>Apps targeting US/EU audiences</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  <strong>3. Visualization</strong>
</h3>

<p>🌍 <strong>Price Class 100:</strong> North America + Europe<br>
🌍 <strong>Price Class 200:</strong> Adds Asia, South America, Middle East, Africa<br>
🌍 <strong>Price Class All:</strong> Every AWS edge location worldwide</p>




<h3>
  
  
  <strong>4. Exam Tip</strong>
</h3>

<p>If a question mentions:</p>

<ul>
<li>
<em>“Reduce CloudFront cost while maintaining good global coverage”</em> → <strong>Price Class 200</strong>
</li>
<li>
<em>“App only for US + EU users”</em> → <strong>Price Class 100</strong>
</li>
<li>
<em>“Worldwide users need best performance”</em> → <strong>Price Class All</strong>
</li>
</ul>

<h3>
  
  
  <strong>CloudFront Cache Invalidations</strong>
</h3>

<p><strong>Purpose:</strong><br>
When you update content in your <strong>origin</strong> (like S3 or EC2),<br>
CloudFront <strong>edge locations</strong> still serve the <em>old cached version</em> until the <strong>TTL expires</strong>.<br>
If you want users to see the <strong>new content immediately</strong>,<br>
you perform a <strong>cache invalidation</strong>.</p>




<h3>
  
  
  <strong>How It Works</strong>
</h3>

<ol>
<li>Each edge location stores cached copies of objects (e.g., <code>index.html</code>, <code>images/*</code>).</li>
<li>When you update your origin (e.g., upload a new HTML or image file), the cached objects remain unchanged until TTL ends.</li>
<li>To refresh them instantly, you issue a <strong>CloudFront invalidation</strong> request:</li>
</ol>

<ul>
<li>For one file: <code>/index.html</code>
</li>
<li>For multiple: <code>/images/*</code>
</li>
<li>For all: <code>/*</code>
</li>
</ul>

<p>CloudFront then removes those objects from <strong>every edge cache</strong>, forcing the next request to fetch the latest version from the origin.</p>




<h3>
  
  
  <strong>Example</strong>
</h3>

<ul>
<li>
<p>You update:</p>

<ul>
<li><code>index.html</code></li>
<li>Several files under <code>/images/</code>
</li>
</ul>


</li>

<li><p>You create an invalidation for:<br><br>
</p></li>

</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  /index.html
  /images/*
</code></pre>

</div>



<ul>
<li>Result:
Edge caches delete the old versions.
Next viewer request = fresh data from origin (S3 or ALB).</li>
</ul>




<h3>
  
  
  <strong>Cost Note</strong>
</h3>

<ul>
<li>
<strong>First 1,000 invalidation paths/month</strong> → Free.</li>
<li>After that → small charge per path.</li>
</ul>




<h3>
  
  
  <strong>Exam Tip</strong>
</h3>

<p>If a question says:</p>

<blockquote>
<p>“Users still see outdated content after updates to the S3 bucket…”</p>
</blockquote>

<p>✅ <strong>Answer:</strong> Perform a <strong>CloudFront invalidation</strong> (don’t make objects public or change TTL).</p>

<h3>
  
  
  <strong>AWS Global Accelerator — Overview</strong>
</h3>

<p><strong>Definition:</strong><br>
AWS <strong>Global Accelerator (GA)</strong> is a <strong>networking service</strong> that improves the <strong>availability and performance</strong> of your global applications by routing user traffic through the <strong>AWS global network</strong> instead of the public internet.</p>




<h3>
  
  
  <strong>1. The Problem It Solves</strong>
</h3>

<ul>
<li>
<p>Without GA, global users access your app over the <strong>public internet</strong>, causing:</p>

<ul>
<li>High latency (many router hops)</li>
<li>Unstable connections</li>
<li>Slower failover between regions</li>
</ul>


</li>

<li><p>Goal → Enter AWS’s private backbone network <strong>as early as possible</strong>.</p></li>

</ul>




<h3>
  
  
  <strong>2. How It Works</strong>
</h3>

<ul>
<li>
<strong>Uses Anycast IPs</strong> (two static, global IPs).</li>
<li>User requests go to the <strong>nearest AWS edge location</strong>.</li>
<li>
<p>From there, traffic travels through <strong>AWS’s internal network</strong> to your backend:</p>

<ul>
<li>
<strong>ALB</strong>, <strong>NLB</strong>, or <strong>EC2 instance</strong> (public or private).</li>
</ul>


</li>

<li><p><strong>Health checks</strong> monitor endpoints and <strong>automatically fail over</strong> within 1 minute if one region becomes unhealthy.</p></li>

</ul>




<h3>
  
  
  <strong>3. Key Benefits</strong>
</h3>

<p>✅ <strong>Static Anycast IPs</strong> (no DNS caching issues)<br>
✅ <strong>Low latency</strong> and <strong>consistent performance</strong><br>
✅ <strong>Automatic regional failover</strong><br>
✅ <strong>Works with TCP &amp; UDP</strong> (HTTP and non-HTTP apps)<br>
✅ <strong>DDoS protection</strong> via <strong>AWS Shield</strong><br>
✅ <strong>Simplified whitelisting</strong> — only two global IPs</p>




<h3>
  
  
  <strong>4. Supported Backends (Endpoints)</strong>
</h3>

<ul>
<li><strong>Elastic IPs</strong></li>
<li><strong>EC2 instances</strong></li>
<li>
<strong>ALB / NLB</strong> (public or private)</li>
</ul>




<h3>
  
  
  <strong>5. CloudFront vs. Global Accelerator</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th><strong>CloudFront</strong></th>
<th><strong>Global Accelerator</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Primary use</strong></td>
<td>CDN (cache content)</td>
<td>Global traffic routing</td>
</tr>
<tr>
<td><strong>Protocol</strong></td>
<td>HTTP / HTTPS</td>
<td>Any TCP or UDP</td>
</tr>
<tr>
<td><strong>Content</strong></td>
<td>Static &amp; dynamic (cached)</td>
<td>Dynamic only (no caching)</td>
</tr>
<tr>
<td><strong>Performance boost</strong></td>
<td>From <strong>edge cache</strong>
</td>
<td>From <strong>AWS global network routing</strong>
</td>
</tr>
<tr>
<td><strong>IP type</strong></td>
<td>DNS-based</td>
<td>2 static Anycast IPs</td>
</tr>
<tr>
<td><strong>Best for</strong></td>
<td>Websites, media, APIs</td>
<td>Gaming, IoT, real-time apps, multi-region HA</td>
</tr>
<tr>
<td><strong>Failover</strong></td>
<td>via DNS TTL</td>
<td>via health checks (&lt;1 min)</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  <strong>6. Exam Tip</strong>
</h3>

<p>If you see:</p>

<blockquote>
<p>“Need global static IPs and lowest-latency routing for TCP/UDP apps”<br>
✅ <strong>Answer:</strong> AWS Global Accelerator</p>
</blockquote>

<p>If you see:</p>

<blockquote>
<p>“Need to deliver cached static and dynamic web content”<br>
✅ <strong>Answer:</strong> Amazon CloudFront</p>
</blockquote>

<h2>
  
  
  <strong>AWS Global Accelerator – Hands-On Summary</strong>
</h2>

<h3>
  
  
  <strong>1. What You Built</strong>
</h3>

<ul>
<li>
<p>Two <strong>EC2 instances</strong> running simple web servers:</p>

<ul>
<li>One in <strong>us-east-1 (Virginia)</strong>
</li>
<li>One in <strong>ap-south-1 (Mumbai)</strong>
</li>
</ul>


</li>

<li><p>Each returned a region-specific message (e.g., <em>Hello from US East 1</em>).</p></li>

<li><p>Then, a <strong>Global Accelerator</strong> was created to route users to the closest healthy endpoint.</p></li>

</ul>




<h3>
  
  
  <strong>2. Key Configuration Steps</strong>
</h3>

<h4>
  
  
  <strong>(a) Create EC2 Instances</strong>
</h4>

<ul>
<li>
<strong>AMI:</strong> Amazon Linux 2, <code>t2.micro</code>
</li>
<li>
<strong>User Data:</strong> simple HTTP server printing region name</li>
<li>
<strong>Security Group:</strong> allow <strong>HTTP (port 80)</strong> from anywhere</li>
<li>
<strong>No key pair</strong> needed (no SSH access required).</li>
</ul>

<h4>
  
  
  <strong>(b) Create the Global Accelerator</strong>
</h4>

<ul>
<li>
<p><strong>Listener:</strong></p>

<ul>
<li>Port 80 / TCP</li>
<li>Optional client affinity = <em>none</em> or <em>source IP</em>
</li>
</ul>


</li>

<li>

<p><strong>Endpoint Groups:</strong></p>

<ul>
<li>Region 1: us-east-1</li>
<li>Region 2: ap-south-1</li>
<li>
<strong>Traffic dial = 100%</strong> each (equal weight)</li>
<li>
<strong>Health checks:</strong> HTTP → “/” every 10 s, threshold = 3</li>
</ul>


</li>

<li><p><strong>Endpoints:</strong> add each EC2 instance.</p></li>

</ul>

<h4>
  
  
  <strong>(c) Test</strong>
</h4>

<ul>
<li>
<p>Accelerator provides:</p>

<ul>
<li><strong>2 static Anycast IPs</strong></li>
<li>
<strong>1 DNS name</strong> (global)</li>
</ul>


</li>

<li><p>From Europe → routed to <strong>us-east-1</strong> (nearest healthy region).</p></li>

<li><p>From Indonesia (via VPN) → routed to <strong>ap-south-1</strong>.</p></li>

</ul>

<h4>
  
  
  <strong>(d) Health Check + Failover Demo</strong>
</h4>

<ul>
<li>Removed HTTP rule from Mumbai instance’s security group → health check = ❌ unhealthy.</li>
<li>Within ≈ 1 minute, Global Accelerator rerouted all traffic to <strong>us-east-1</strong> automatically.</li>
</ul>




<h3>
  
  
  <strong>3. Cleanup</strong>
</h3>

<ul>
<li>Terminate both EC2 instances.</li>
<li>
<strong>Disable</strong> and then <strong>delete</strong> the Accelerator.</li>
<li>Verify that billing stops.</li>
</ul>




<h3>
  
  
  <strong>4. Pricing Overview</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Cost Component</strong></th>
<th><strong>Rate (approx.)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Accelerator hourly charge</td>
<td>$0.025 per hour</td>
</tr>
<tr>
<td>Data transfer (edge → origin)</td>
<td>$0.01 – $0.08 per GB depending on region</td>
</tr>
</tbody>
</table></div>

<p><em>(Service is billed while enabled, so always delete after labs.)</em></p>




<h3>
  
  
  <strong>5. Key Takeaways</strong>
</h3>

<p>✅ <strong>Anycast IPs:</strong> two fixed global addresses.<br>
✅ <strong>Health checks + automatic failover (&lt; 1 min).</strong><br>
✅ <strong>Traffic routing via AWS backbone</strong>, not the public Internet.<br>
✅ <strong>Supports TCP &amp; UDP</strong>, public or private ALB/NLB/EC2.<br>
✅ <strong>Expensive but enterprise-grade performance.</strong></p>




<p><strong>Exam tip:</strong></p>

<blockquote>
<p><em>“Global users need lowest-latency access and instant regional failover using static IPs.”</em><br>
→ <strong>Answer:</strong> AWS Global Accelerator.</p>
</blockquote>

