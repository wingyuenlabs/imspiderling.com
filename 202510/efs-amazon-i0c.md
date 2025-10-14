---
Title: EFS Amazon
Description: 
Author: Aisalkyn Aidarova
Date: 2025-10-14T21:53:28.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🌩️ What Is AWS Storage?
</h2>

<p>Think of <strong>AWS storage</strong> like <strong>online hard drives</strong> that you rent from Amazon’s data centers.<br>
Just like your computer has a <strong>C: drive</strong>, AWS gives you many different types of <strong>cloud drives</strong>, each made for a specific purpose.</p>

<p>AWS stores your data safely in big, secure buildings called <strong>data centers</strong>, all around the world.</p>


<h2>
  
  
  🧱 3 Main Types of AWS Storage
</h2>

<p>There are <strong>three main ways</strong> to store data in AWS — just like in real life, you use different places for different things.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Type</th>
<th>What It Feels Like</th>
<th>Example Use</th>
<th>AWS Service</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Object storage</strong></td>
<td>Like Google Drive or Dropbox</td>
<td>Store pictures, videos, backups</td>
<td><strong>S3 (Simple Storage Service)</strong></td>
</tr>
<tr>
<td><strong>Block storage</strong></td>
<td>Like your computer’s hard disk</td>
<td>Run operating systems or databases</td>
<td><strong>EBS (Elastic Block Store)</strong></td>
</tr>
<tr>
<td><strong>File storage</strong></td>
<td>Like a shared folder in an office</td>
<td>Multiple computers share the same files</td>
<td><strong>EFS (Elastic File System)</strong></td>
</tr>
</tbody>
</table></div>

<p>Let’s understand each one.</p>


<h2>
  
  
  ☁️ 1. Amazon S3 (Simple Storage Service)
</h2>
<h3>
  
  
  What it is:
</h3>

<p>Imagine a huge <strong>online folder</strong> where you can put any kind of file — photos, videos, text, etc.<br>
This folder is called a <strong>bucket</strong> in AWS.</p>

<p>You don’t need to manage a server; AWS does it for you.</p>
<h3>
  
  
  What you can do:
</h3>

<ul>
<li>Create a <strong>bucket</strong> (your folder)</li>
<li>Upload or download files</li>
<li>Make files <strong>public or private</strong>
</li>
<li>Set <strong>rules</strong> to automatically move old files to cheaper storage (like Glacier)</li>
<li>Store <strong>unlimited data</strong>
</li>
</ul>
<h3>
  
  
  Simple example:
</h3>

<p>You can upload a picture to S3, get a link, and share it with someone — just like sharing a file from Google Drive.</p>


<h2>
  
  
  💾 2. Amazon EBS (Elastic Block Store)
</h2>
<h3>
  
  
  What it is:
</h3>

<p>This is like the <strong>hard drive</strong> inside your computer.</p>

<p>When you create a <strong>virtual computer (EC2 instance)</strong> in AWS, it needs a disk to save data — that’s where EBS comes in.</p>

<p>Each EBS volume can be connected to <strong>one EC2 instance</strong> at a time.</p>
<h3>
  
  
  What you can do:
</h3>

<ul>
<li>Attach it to an EC2 instance (like plugging a USB drive)</li>
<li>Store your system files and databases</li>
<li>Take <strong>snapshots</strong> (backups) of your disk</li>
<li>Restore data if something goes wrong</li>
</ul>
<h3>
  
  
  Simple example:
</h3>

<p>If your EC2 instance is your computer, then EBS is your <strong>C: drive</strong>.</p>


<h2>
  
  
  📂 3. Amazon EFS (Elastic File System)
</h2>
<h3>
  
  
  What it is:
</h3>

<p>Think of this like a <strong>shared network folder</strong> that many computers can use at the same time.</p>

<p>EFS automatically grows and shrinks as you add or remove files.<br>
It’s used when multiple EC2 instances need to <strong>share the same data</strong>.</p>
<h3>
  
  
  What you can do:
</h3>

<ul>
<li>Create one EFS</li>
<li>Mount it (connect) to many EC2 instances</li>
<li>Store and share files in real time</li>
<li>Data is stored in <strong>multiple availability zones</strong>, so it’s safe</li>
</ul>
<h3>
  
  
  Simple example:
</h3>

<p>In an office, if 5 people open and edit the same document stored in a shared folder — that’s how EFS works.</p>


<h2>
  
  
  🧊 4. Amazon Glacier (Archival Storage)
</h2>
<h3>
  
  
  What it is:
</h3>

<p>Glacier is used to <strong>store old data you don’t use often</strong>, but want to keep safe — like backups or old records.</p>

<p>It’s <strong>very cheap</strong>, but if you want to get your files back, it takes a few hours.</p>
<h3>
  
  
  Simple example:
</h3>

<p>Think of Glacier like putting old photo albums in your attic — safe, cheap, but not quickly accessible.</p>


<h2>
  
  
  🗺️ 5. Where Is Your Data Stored?
</h2>

<p>AWS divides the world into <strong>regions</strong> (like cities) and <strong>availability zones (AZs)</strong> (like buildings).<br>
When you create storage, you pick a <strong>region</strong>, for example:</p>

<ul>
<li>
<code>us-east-1</code> → Virginia, USA</li>
<li>
<code>us-west-2</code> → Oregon, USA</li>
</ul>

<p>Your data is saved <strong>in multiple buildings (AZs)</strong> so that if one fails, your data is still safe.</p>


<h2>
  
  
  💰 6. How You Pay for AWS Storage
</h2>

<p>You pay only for what you use:</p>

<ul>
<li>
<strong>S3</strong>: by amount of data + number of requests</li>
<li>
<strong>EBS</strong>: by size of disk (GB per month)</li>
<li>
<strong>EFS</strong>: by storage amount and usage time</li>
<li>
<strong>Glacier</strong>: by how much you store + how often you retrieve</li>
</ul>


<h2>
  
  
  🧠 7. What You Need to Know First (Base Knowledge)
</h2>

<p>Before learning AWS storage, you should understand:</p>

<ol>
<li>What is a <strong>server</strong> (EC2)</li>
<li>What is a <strong>region and availability zone</strong>
</li>
<li>What is a <strong>VPC</strong> (virtual network in AWS)</li>
<li>How to <strong>log in to AWS Console</strong>
</li>
<li>How to use the <strong>AWS Management Console</strong> (the web interface)</li>
</ol>

<p>Once you know these, AWS storage will make sense.</p>


<h2>
  
  
  🪜 8. Easiest Learning Path
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Step</th>
<th>What to Learn</th>
<th>What to Do</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Learn S3</td>
<td>Create a bucket, upload files, make one public</td>
</tr>
<tr>
<td>2</td>
<td>Learn EBS</td>
<td>Create EC2 → add an extra EBS volume → see how it stores files</td>
</tr>
<tr>
<td>3</td>
<td>Learn EFS</td>
<td>Create file system → mount on 2 EC2s → see both share files</td>
</tr>
<tr>
<td>4</td>
<td>Learn Glacier</td>
<td>Move S3 files to Glacier for cheap long-term storage</td>
</tr>
</tbody>
</table></div>
<h3>
  
  
  <strong>What is NFS (Network File System)?</strong>
</h3>

<p><strong>NFS</strong> stands for <strong>Network File System</strong>.<br>
It’s a way for <strong>computers to share files over a network</strong> — like how you share a folder on Google Drive, but for computers on the same network.</p>


<h3>
  
  
  <strong>Example:</strong>
</h3>

<p>Imagine you have:</p>

<ul>
<li>One <strong>main computer (server)</strong> that stores files.</li>
<li>Several <strong>other computers (clients)</strong> that want to use those files.</li>
</ul>

<p>With <strong>NFS</strong>, the clients can “mount” (connect to) the shared folder from the server and use it <strong>as if it were on their own computer</strong> — open, edit, save, etc.</p>


<h3>
  
  
  <strong>Why NFS is useful</strong>
</h3>

<ul>
<li>Everyone sees the <strong>same files</strong> in real time.</li>
<li>Great for <strong>teams or systems</strong> that need to share data (like multiple EC2 servers in AWS).</li>
<li>Saves space — no need to copy the same file to every machine.</li>
</ul>


<h3>
  
  
  <strong>Simple analogy</strong>
</h3>

<p>Think of <strong>NFS</strong> like a <strong>shared drive in an office</strong>:</p>

<ul>
<li>The <strong>server</strong> is the file cabinet.</li>
<li>The <strong>clients</strong> are the employees’ computers.</li>
<li>Everyone can open and work on the same files stored in that cabinet.</li>
</ul>
<h3>
  
  
  <strong>1. What EFS Is</strong>
</h3>

<ul>
<li>
<strong>EFS (Elastic File System)</strong> is a <strong>managed Network File System (NFS)</strong>.</li>
<li>It allows <strong>multiple EC2 instances</strong> (even across multiple Availability Zones) to <strong>share the same data</strong>.</li>
<li>It behaves like a <strong>shared folder</strong> in the cloud, accessible from multiple Linux servers.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdrp5suo3cig6fns14br1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdrp5suo3cig6fns14br1.png" alt=" " width="800" height="533"></a></p>


<h3>
  
  
  <strong>2. Key Features</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Fully managed</strong></td>
<td>AWS takes care of setup, scaling, and maintenance.</td>
</tr>
<tr>
<td><strong>Scalable</strong></td>
<td>Automatically grows and shrinks as you add/remove files.</td>
</tr>
<tr>
<td><strong>Highly available</strong></td>
<td>Spans multiple AZs for fault tolerance.</td>
</tr>
<tr>
<td><strong>Expensive</strong></td>
<td>Costs roughly <strong>3× more than GP2 EBS</strong>, but you pay <strong>only for what you use</strong>.</td>
</tr>
<tr>
<td><strong>Supports Linux only</strong></td>
<td>Uses the <strong>NFSv4.1 protocol</strong> (not supported on Windows).</td>
</tr>
</tbody>
</table></div>


<h3>
  
  
  <strong>3. Performance and Throughput Modes</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Mode</th>
<th>When to Use</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>General Purpose</strong></td>
<td>Low-latency workloads like web apps, CMS, or home directories</td>
<td>Default</td>
</tr>
<tr>
<td><strong>Max I/O</strong></td>
<td>High-latency but high-throughput workloads like big data or media processing</td>
<td>For parallel operations</td>
</tr>
</tbody>
</table></div>

<p><strong>Throughput Options:</strong></p>

<ul>
<li>
<strong>Bursting</strong> – Throughput increases with storage size.</li>
<li>
<strong>Provisioned</strong> – Manually set throughput independent of storage.</li>
<li>
<strong>Elastic</strong> – Automatically scales based on workload (best for unpredictable loads).</li>
</ul>


<h3>
  
  
  <strong>4. Storage Classes and Lifecycle Management</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Storage Tier</th>
<th>Description</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>EFS Standard</strong></td>
<td>For frequently accessed files</td>
<td>Default</td>
</tr>
<tr>
<td><strong>EFS Infrequent Access (IA)</strong></td>
<td>Lower cost for rarely accessed files</td>
<td>Backup or less-used data</td>
</tr>
<tr>
<td><strong>EFS Archive</strong></td>
<td>Very low cost for long-term cold storage</td>
<td>Archival data</td>
</tr>
</tbody>
</table></div>

<p><strong>Lifecycle Policies</strong> automatically move files between tiers:</p>

<ul>
<li>Example: Move from <strong>Standard → IA</strong> after 60 days of no access.</li>
</ul>


<h3>
  
  
  <strong>5. Availability and Durability</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Option</th>
<th>Zones</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>EFS Regional (Standard)</strong></td>
<td>Multi-AZ</td>
<td>Production systems needing high durability</td>
</tr>
<tr>
<td><strong>EFS One Zone</strong></td>
<td>Single AZ</td>
<td>Development or cost-sensitive workloads</td>
</tr>
</tbody>
</table></div>


<h3>
  
  
  <strong>6. Common Use Cases</strong>
</h3>

<ul>
<li>WordPress shared uploads directory</li>
<li>CMS and content repositories</li>
<li>Web servers sharing same static assets</li>
<li>Data science and analytics workloads</li>
<li>Shared configuration files for microservices</li>
</ul>


<h3>
  
  
  <strong>7. Exam Tips</strong>
</h3>

<ul>
<li>EFS is <strong>NFS-based</strong> → cannot be accessed with SMB or CIFS (Windows).</li>
<li>
<strong>Cannot SSH</strong> into EFS — it’s a managed file system, not a server.</li>
<li>
<strong>Mount targets</strong> must be allowed in <strong>Security Groups</strong> (NFS port <strong>2049</strong>).</li>
<li>
<strong>Encryption</strong> at rest uses <strong>AWS KMS</strong>.</li>
<li>
<strong>Automatically scales</strong> in size and throughput.</li>
</ul>

<p>\</p>
<h2>
  
  
  <strong>1. Create the EFS File System</strong>
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp4g18z2q9njynttm9scm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp4g18z2q9njynttm9scm.png" alt=" " width="800" height="533"></a></p>
<h3>
  
  
  <strong>Basic Setup</strong>
</h3>

<ul>
<li>Go to <strong>Amazon EFS → Create file system</strong>
</li>
<li>Choose <strong>Customize</strong> (instead of “Quick Create”) to see all options.</li>
<li>Leave the <strong>name</strong> blank or optional.</li>
<li>Choose the <strong>default VPC</strong>.</li>
</ul>


<h2>
  
  
  <strong>2. File System Type</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Option</th>
<th>Description</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Regional (recommended)</strong></td>
<td>Spans multiple AZs for <strong>high availability and durability</strong>
</td>
<td><strong>Production</strong></td>
</tr>
<tr>
<td><strong>One Zone</strong></td>
<td>Stored in a single AZ, cheaper, less durable</td>
<td><strong>Development/Test</strong></td>
</tr>
</tbody>
</table></div>


<h2>
  
  
  <strong>3. Backup and Lifecycle</strong>
</h2>

<ul>
<li>
<strong>Automatic backups:</strong> Keep <strong>enabled</strong> (recommended).</li>
<li>
<p><strong>Lifecycle management:</strong> Move data automatically between tiers:</p>

<ul>
<li>After <strong>30 days</strong> → move to <strong>Infrequent Access (IA)</strong>
</li>
<li>After <strong>90 days</strong> → move to <strong>Archive</strong>
</li>
<li>When accessed again → move back to <strong>Standard</strong>
</li>
</ul>
</li>
</ul>

<p>This helps <strong>reduce costs</strong> for rarely accessed data.</p>


<h2>
  
  
  <strong>4. Encryption and Performance Settings</strong>
</h2>

<ul>
<li>
<strong>Encryption:</strong> Enable (default, uses KMS).</li>
<li><strong>Throughput modes:</strong></li>
</ul>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Mode</th>
<th>Description</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Bursting</strong></td>
<td>Throughput scales with size.</td>
<td>Default mode.</td>
</tr>
<tr>
<td><strong>Elastic</strong></td>
<td>Automatically adjusts throughput up/down.</td>
<td>Best for <strong>unpredictable workloads</strong>.</td>
</tr>
<tr>
<td><strong>Provisioned</strong></td>
<td>Manually set throughput (e.g. 100 MB/s).</td>
<td>For <strong>consistent, predictable</strong> workloads.</td>
</tr>
</tbody>
</table></div>

<p><strong>Recommended:</strong> Elastic throughput + General Purpose performance.</p>


<h2>
  
  
  <strong>5. Network Access</strong>
</h2>

<ul>
<li>Choose <strong>default VPC</strong>.</li>
<li>Automatically creates <strong>mount targets</strong> in 3 AZs (for Regional).</li>
<li>Create a <strong>security group</strong> (e.g. <code>sg-efs-demo</code>) with <strong>no inbound rules</strong> yet.</li>
</ul>

<p>Then assign that group to the EFS.</p>

<p><strong>Port:</strong> NFS (2049)</p>


<h2>
  
  
  <strong>6. Launch EC2 Instances</strong>
</h2>

<ul>
<li>
<p>Create <strong>two EC2 instances</strong> (Instance A &amp; B):</p>

<ul>
<li>Use <strong>Amazon Linux 2</strong>
</li>
<li>Instance type: <strong>t2.micro</strong>
</li>
<li>Disable key pair (use <strong>EC2 Instance Connect</strong>)</li>
<li>Place Instance A in <strong>AZ-a</strong>, Instance B in <strong>AZ-b</strong>
</li>
</ul>
</li>
</ul>


<h2>
  
  
  <strong>7. Attach EFS to EC2</strong>
</h2>

<p>Now, directly in the <strong>EC2 launch wizard</strong>:</p>

<ul>
<li>Scroll to <strong>File Systems</strong>
</li>
<li>Add your <strong>EFS file system</strong>
</li>
<li>Mount point: <code>/mnt/efs/fs1</code>
</li>
<li>
<p>The console <strong>automatically creates</strong> and configures:</p>

<ul>
<li>Mount scripts</li>
<li>Correct <strong>security groups</strong> (<code>efs-sg-1</code>, <code>efs-sg-2</code>)</li>
<li>NFS rules (port 2049)</li>
</ul>
</li>
</ul>

<p>No manual user-data script needed anymore.</p>


<h2>
  
  
  <strong>8. Test EFS Sharing Between Instances</strong>
</h2>
<h3>
  
  
  <strong>On Instance A</strong>
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>su
<span class="nb">echo</span> <span class="s2">"hello world"</span> <span class="o">&gt;</span> /mnt/efs/fs1/hello.txt
<span class="nb">cat</span> /mnt/efs/fs1/hello.txt
</code></pre>

</div>


<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>hello world
</code></pre>

</div>



<h3>
  
  
  <strong>On Instance B</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">ls</span> /mnt/efs/fs1/
<span class="nb">cat</span> /mnt/efs/fs1/hello.txt
</code></pre>

</div>



<p>Output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>hello world
</code></pre>

</div>



<p>✅ Both instances see the <strong>same file</strong> — this confirms EFS is a <strong>shared NFS storage</strong>.</p>




<h2>
  
  
  <strong>9. Cleanup</strong>
</h2>

<ul>
<li>Terminate EC2 instances.</li>
<li>Delete EFS file system.</li>
<li>Delete extra auto-created security groups (e.g., <code>efs-sg-1</code>, <code>efs-sg-2</code>).</li>
</ul>




<h2>
  
  
  <strong>Key Takeaways for Students</strong>
</h2>

<ul>
<li>EFS = <strong>shared, managed NFS storage</strong> for multiple EC2s.</li>
<li>Use <strong>Elastic throughput + Regional type</strong> in production.</li>
<li>Accessible via <strong>port 2049 (NFS)</strong>.</li>
<li>
<strong>Pay per GB used</strong> — no need to pre-provision capacity.</li>
<li>
<strong>Mount once → shared across all instances</strong>.</li>
</ul>

<h2>
  
  
  <strong>1. Amazon EBS (Elastic Block Store)</strong>
</h2>

<h3>
  
  
  <strong>What it is:</strong>
</h3>

<ul>
<li>A <strong>block-level storage</strong> device that attaches to a <strong>single EC2 instance</strong>.</li>
<li>Behaves like a <strong>hard drive</strong> for your EC2 instance.</li>
</ul>

<h3>
  
  
  <strong>Key Points:</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Attachment</strong></td>
<td>One instance at a time (except io1/io2 Multi-Attach)</td>
</tr>
<tr>
<td><strong>AZ Scope</strong></td>
<td>Tied to one <strong>Availability Zone</strong>
</td>
</tr>
<tr>
<td><strong>Migration</strong></td>
<td>Use <strong>snapshots</strong> to move data between AZs</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Depends on volume type (gp2/gp3/io1/io2)</td>
</tr>
<tr>
<td><strong>Backup Impact</strong></td>
<td>Backups use I/O → may affect performance</td>
</tr>
<tr>
<td><strong>Cost</strong></td>
<td>Pay for <strong>provisioned capacity</strong> (size + IOPS)</td>
</tr>
<tr>
<td><strong>OS Support</strong></td>
<td>Works with <strong>Linux and Windows</strong>
</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  <strong>When to use:</strong>
</h3>

<ul>
<li>Databases</li>
<li>OS boot volumes</li>
<li>Applications that need <strong>low latency block storage</strong>
</li>
</ul>




<h2>
  
  
  <strong>2. Amazon EFS (Elastic File System)</strong>
</h2>

<h3>
  
  
  <strong>What it is:</strong>
</h3>

<ul>
<li>A <strong>network file system (NFS)</strong> that multiple EC2 instances can share at the same time.</li>
</ul>

<h3>
  
  
  <strong>Key Points:</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Attachment</strong></td>
<td>Multiple instances <strong>across AZs</strong>
</td>
</tr>
<tr>
<td><strong>AZ Scope</strong></td>
<td>
<strong>Regional</strong> (spans multiple AZs)</td>
</tr>
<tr>
<td><strong>Protocol</strong></td>
<td>Uses <strong>NFS</strong> (Linux only)</td>
</tr>
<tr>
<td><strong>Scaling</strong></td>
<td>Automatically scales with file size</td>
</tr>
<tr>
<td><strong>Cost</strong></td>
<td>Pay per <strong>GB used</strong>, 3× more expensive than EBS</td>
</tr>
<tr>
<td><strong>Storage Tiers</strong></td>
<td>Standard, Infrequent Access (IA), Archive</td>
</tr>
<tr>
<td><strong>Encryption</strong></td>
<td>Supported with KMS</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Choose <strong>Elastic</strong>, <strong>Bursting</strong>, or <strong>Provisioned</strong> modes</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  <strong>When to use:</strong>
</h3>

<ul>
<li>Shared content for web servers (e.g., WordPress uploads)</li>
<li>CMS or data sharing between EC2 instances</li>
<li>Applications needing <strong>shared access + scalability</strong>
</li>
</ul>




<h2>
  
  
  <strong>3. Instance Store</strong>
</h2>

<h3>
  
  
  <strong>What it is:</strong>
</h3>

<ul>
<li>
<strong>Physical storage</strong> attached directly to the EC2 host.</li>
<li>
<strong>Temporary</strong> — data is lost when instance stops or terminates.</li>
</ul>

<h3>
  
  
  <strong>Key Points:</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Speed</strong></td>
<td>Very fast (physically attached)</td>
</tr>
<tr>
<td><strong>Durability</strong></td>
<td>Data lost if instance stops or fails</td>
</tr>
<tr>
<td><strong>Cost</strong></td>
<td>Included in EC2 price</td>
</tr>
<tr>
<td><strong>Use Case</strong></td>
<td>Temporary data (e.g., caching, buffer files)</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  <strong>When to use:</strong>
</h3>

<ul>
<li>High-speed temporary storage</li>
<li>Caches or scratch data</li>
<li>Non-persistent workloads</li>
</ul>




<h2>
  
  
  <strong>4. Quick Comparison Table</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th><strong>EBS</strong></th>
<th><strong>EFS</strong></th>
<th><strong>Instance Store</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Type</td>
<td>Block storage</td>
<td>Network file system</td>
<td>Local storage</td>
</tr>
<tr>
<td>Shared Access</td>
<td>❌ Single EC2</td>
<td>✅ Multiple EC2</td>
<td>❌ No</td>
</tr>
<tr>
<td>AZ Scope</td>
<td>One AZ</td>
<td>Multi-AZ</td>
<td>One AZ</td>
</tr>
<tr>
<td>Persistence</td>
<td>✅ Persistent</td>
<td>✅ Persistent</td>
<td>❌ Temporary</td>
</tr>
<tr>
<td>Backup</td>
<td>Snapshots</td>
<td>Lifecycle tiers</td>
<td>None</td>
</tr>
<tr>
<td>OS Support</td>
<td>Linux &amp; Windows</td>
<td>Linux only</td>
<td>Linux &amp; Windows</td>
</tr>
<tr>
<td>Cost</td>
<td>Moderate</td>
<td>Higher</td>
<td>Free (included)</td>
</tr>
</tbody>
</table></div>

