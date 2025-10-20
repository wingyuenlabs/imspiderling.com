---
Title: My DevOps Journey: Part 10 - Deep Dive into AWS EC2 (Elastic Compute Cloud)
Description: 
Author: Sheersh Sinha
Date: 2025-10-20T21:48:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>After understanding how cloud computing works in my previous post, I was eager to move from concepts to creation. I wanted to actually build something in the cloud - and that's when I discovered <strong>Amazon EC2 (Elastic Compute Cloud)</strong> - the beating heart of AWS infrastructure.</p>

<p>This was the first time I felt like I wasn't just using the cloud… I was running it.</p>

<h2>
  
  
  What is EC2?
</h2>

<p><strong>EC2</strong> is Amazon's virtual server - it's like renting a computer in the cloud that you can access anytime, configure however you want, and scale instantly.</p>

<p>Think of EC2 as your personal Linux or Windows machine - but hosted globally and available on demand. Each instance (server) you launch can host an application, a script, a database, or even an entire architecture.</p>

<p>When I launched my first EC2 instance, it wasn't just a VM - it was a sandbox for experimentation.</p>

<h2>
  
  
  Understanding AMI - The Blueprint of Your Cloud Machines
</h2>

<p>Every EC2 instance starts from an <strong>AMI (Amazon Machine Image)</strong>.</p>

<p>An AMI is like a template that defines:</p>

<ul>
<li>The operating system (Ubuntu, Amazon Linux, Windows)</li>
<li>Pre-installed packages</li>
<li>System configuration</li>
</ul>

<p>When I was testing my Log Analyzer project, I picked an <strong>Ubuntu 22.04 AMI</strong>, installed git, cron, gzip, and cloned my repository. I later created my own custom AMI - so I could launch pre-configured instances instantly, saving time in future experiments.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekqa1tkni61iinbp6zhc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekqa1tkni61iinbp6zhc.png" alt=" " width="800" height="133"></a></p>

<p><strong>Key Lesson:</strong> Building custom AMIs means infrastructure can be versioned just like code.</p>

<h2>
  
  
  Instance Metadata and User Data - Automating Configuration
</h2>

<p>One of the most powerful features of EC2 is <strong>instance metadata</strong> and <strong>user data</strong>. They allow your instance to know about itself and even auto-configure during boot.</p>

<h3>
  
  
  Instance Metadata
</h3>

<p>This is the instance's "self-awareness." It stores dynamic information like:</p>

<ul>
<li>Instance ID</li>
<li>Public/Private IP</li>
<li>Security group</li>
<li>Region</li>
</ul>

<p>Command to view metadata (from inside EC2):</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7akgu2vd670lw69ivcm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7akgu2vd670lw69ivcm.png" alt=" " width="692" height="406"></a></p>

<h3>
  
  
  User Data
</h3>

<p>This is a script that runs automatically when your EC2 instance starts for the first time. I used it to automate dependency installation and log setup:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">sudo </span>apt update <span class="nt">-y</span>
<span class="nb">sudo </span>apt <span class="nb">install</span> <span class="nt">-y</span> git cron <span class="nb">gzip
</span>git clone https://github.com/sheersh123/bash-log-analyzer.git
<span class="nb">cd </span>bash-log-analyzer
<span class="nb">chmod</span> +x log_analyzer.sh
<span class="o">(</span>crontab <span class="nt">-l</span> 2&gt;/dev/null<span class="p">;</span> <span class="nb">echo</span> <span class="s2">"0 0 * * * /home/ubuntu/bash-log-analyzer/log_analyzer.sh /var/log/syslog"</span><span class="o">)</span> | crontab -
</code></pre>

</div>



<p>When I launched this instance - it auto-setup my environment. No manual SSH, no copy-paste - everything was ready within minutes.</p>

<p><strong>Key Lesson:</strong> Automation doesn't start with tools like Ansible - it starts with User Data scripts.</p>

<h2>
  
  
  EC2 Instance Types and Pricing Models - The Cost Optimization Game
</h2>

<p>AWS gives you flexibility not just in size, but in how you pay.</p>

<h3>
  
  
  Instance Types
</h3>

<p>Each type is designed for a specific workload:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Family</th>
<th>Example</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td>General Purpose</td>
<td>t3.micro</td>
<td>Lightweight apps, testing</td>
</tr>
<tr>
<td>Compute Optimized</td>
<td>c5.large</td>
<td>High CPU workloads</td>
</tr>
<tr>
<td>Memory Optimized</td>
<td>r5.large</td>
<td>Databases, analytics</td>
</tr>
<tr>
<td>Storage Optimized</td>
<td>i3.large</td>
<td>High disk I/O operations</td>
</tr>
<tr>
<td>GPU Instances</td>
<td>p3.2xlarge</td>
<td>ML, deep learning</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Pricing Models
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Model</th>
<th>Description</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>On-Demand</strong></td>
<td>Pay hourly - no commitment</td>
<td>Testing, short workloads</td>
</tr>
<tr>
<td><strong>Reserved</strong></td>
<td>Commit for 1-3 years, lower cost</td>
<td>Long-term stable apps</td>
</tr>
<tr>
<td><strong>Spot</strong></td>
<td>Use spare capacity at huge discounts</td>
<td>Flexible, interruptible tasks</td>
</tr>
<tr>
<td><strong>Savings Plan</strong></td>
<td>Flexible compute commitment</td>
<td>Mixed workloads</td>
</tr>
</tbody>
</table></div>

<p>When I started, I used <strong>t2.micro (Free Tier)</strong> to experiment. It was enough for scripts, GitHub syncs, and learning automation.</p>

<p><strong>Key Lesson:</strong> Cloud computing rewards those who understand efficiency - not just scalability.</p>

<h2>
  
  
  AWS CLI - Controlling AWS from the Command Line
</h2>

<p>The <strong>AWS CLI</strong> became my favorite DevOps weapon. Instead of clicking through the console, I started managing everything through the terminal.</p>

<h3>
  
  
  Installing AWS CLI
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt <span class="nb">install </span>awscli <span class="nt">-y</span>
aws configure
</code></pre>

</div>



<p>You'll be prompted for:</p>

<ul>
<li>AWS Access Key</li>
<li>Secret Key</li>
<li>Default region</li>
<li>Output format (json, text, or table)</li>
</ul>

<h3>
  
  
  Common AWS CLI Commands
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Command</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>aws ec2 describe-instances</code></td>
<td>List all EC2 instances</td>
</tr>
<tr>
<td><code>aws s3 ls</code></td>
<td>List S3 buckets</td>
</tr>
<tr>
<td><code>aws ec2 stop-instances --instance-ids &lt;id&gt;</code></td>
<td>Stop a specific instance</td>
</tr>
<tr>
<td><code>aws s3 cp ./reports s3://my-devops-logs/</code></td>
<td>Upload files to S3</td>
</tr>
<tr>
<td><code>aws iam list-users</code></td>
<td>View IAM users</td>
</tr>
</tbody>
</table></div>

<p><strong>Key Lesson:</strong> The CLI is where DevOps engineers truly control the cloud - it's scriptable, repeatable, and fast.</p>

<h2>
  
  
  My AWS Task - Launching My First Windows EC2 Instance
</h2>

<p>After experimenting with Linux instances for scripting and automation, I wanted to test how DevOps workflows translate into Windows environments.</p>

<p>As part of my AWS Task, I decided to:</p>

<ul>
<li>Create a Windows VM on AWS EC2</li>
<li>Connect via <strong>RDP (Remote Desktop Protocol)</strong>
</li>
<li>Open CMD inside the instance</li>
<li>Verify system details such as hostname, architecture, and OS build</li>
</ul>

<p>This task helped me understand how cross-platform management works in the cloud.</p>

<p><strong>Tech Stack Used:</strong></p>

<ul>
<li>AWS EC2 (Windows)</li>
<li>RDP (Client pre-installed in Windows)</li>
</ul>

<p><strong>Commands Executed in CMD:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiot6q4z34sxqbejlftkp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiot6q4z34sxqbejlftkp.png" alt=" " width="800" height="449"></a></p>

<p>Once connected, I could view the instance's IP, processor, and memory details - confirming that my Windows EC2 instance was successfully deployed and live.</p>

<p><strong>Key Lesson:</strong> Managing Windows VMs on AWS gives a new perspective - it's not just about Linux automation; DevOps engineers often maintain hybrid environments where both OS types coexist.</p>

<p>This small exercise boosted my confidence in handling multi-OS infrastructure - a key skill when working in enterprise-scale DevOps setups.</p>

<h2>
  
  
  My Turning Point - The "Vanishing Instance" Moment
</h2>

<p>During one of my test runs, I accidentally terminated an EC2 instance without creating an AMI backup. All my logs and configurations vanished.</p>

<p>It was frustrating - but it also taught me one of the most powerful lessons in cloud computing:</p>

<blockquote>
<p>"In the cloud, if you didn't back it up, it never existed."</p>
</blockquote>

<p>Since then, I've built the habit of creating snapshots and AMIs before every experiment.</p>

<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>
<strong>EC2</strong> is the core of AWS computing - your virtual data center</li>
<li>
<strong>AMIs</strong> are blueprints for consistent deployments</li>
<li>
<strong>Metadata &amp; User Data</strong> enable automation from the first boot</li>
<li>Understanding pricing saves money and mistakes</li>
<li>The <strong>AWS CLI</strong> turns DevOps engineers into automation pros</li>
<li>
<strong>Hybrid environments</strong> (Linux + Windows) reflect real-world DevOps challenges</li>
</ul>

<h2>
  
  
  What's Next - Load Balancing &amp; Auto Scaling in AWS
</h2>

<p>Now that I've learned to launch and automate EC2 instances, the next step is understanding how to distribute traffic and maintain high availability.</p>

<p>In my next post, I'll explore how AWS helps scale applications seamlessly through <strong>Load Balancers</strong> and <strong>Auto Scaling Groups</strong>.</p>

<p>Here's what's coming next:</p>

<ul>
<li>
<strong>Load Balancer</strong> - The foundation of traffic management in AWS</li>
<li>
<strong>Application Load Balancer (ALB)</strong> - Handling HTTP/HTTPS traffic intelligently</li>
<li>
<strong>Network Load Balancer (NLB)</strong> - High-performance traffic routing at Layer 4</li>
<li>
<strong>Launch Templates</strong> - Predefined instance configurations for auto-scaling</li>
<li>
<strong>Types of Load Balancers</strong> - Understanding Classic vs Application vs Network</li>
<li>
<strong>Target Groups and Listeners</strong> - The logic behind routing and instance health checks</li>
<li>
<strong>Auto Scaling Group (ASG)</strong> - Automatically adjusting instance count based on demand</li>
</ul>




<p><strong>"Scaling isn't about adding servers - it's about maintaining stability while the world grows around your system."</strong></p>

<p>Stay tuned - the next post will be all about keeping your cloud architecture resilient, dynamic, and efficient.</p>

