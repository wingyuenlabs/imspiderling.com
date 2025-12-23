---
Title: Day 24.Setting Up an Application Load Balancer for an EC2 Instance
Description: 
Author: Thu Kha Kyawe
Date: 2025-12-23T21:56:02.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  Lab Information
</h3>

<p>The Nautilus DevOps team is currently working on setting up a simple application on the AWS cloud. They aim to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.</p>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Set up an Application Load Balancer named devops-alb.<br>
Create a target group named devops-tg.<br>
Create a security group named devops-sg to open port 80 for the public.<br>
Attach this security group to the ALB.<br>
The ALB should route traffic on port 80 to port 80 of the devops-ec2 instance.<br>
Make appropriate changes in the default security group attached to the EC2 instance if necessary.<br>
</code></pre>

</div>
<h3>
<br>
  <br>
  <br>
  Lab Solutions<br>
</h3>

<p>🧭 Step-by-Step Solution<br>
🔹 STEP 1: Create Security Group for ALB (devops-sg)</p>

<p>Go to EC2 → Security Groups</p>

<p>Click Create security group</p>

<p>Set:</p>

<p>Security group name: devops-sg</p>

<p>Description: ALB security group</p>

<p>VPC: Default VPC</p>

<p>Inbound rules → Add rule</p>

<p>Type: HTTP</p>

<p>Port: 80</p>

<p>Source: 0.0.0.0/0</p>

<p>Leave Outbound rules as default</p>

<p>Click Create security group</p>

<p>✅ This allows public internet traffic to the ALB.</p>

<p>🔹 STEP 2: Create Target Group (devops-tg)</p>

<p>Go to EC2 → Target Groups</p>

<p>Click Create target group</p>

<p>Configure:</p>

<p>Target type: Instances</p>

<p>Target group name: devops-tg</p>

<p>Protocol: HTTP</p>

<p>Port: 80</p>

<p>VPC: Default VPC</p>

<p>Health checks</p>

<p>Protocol: HTTP</p>

<p>Path: / (default, works with Nginx)</p>

<p>Click Next</p>

<p>Select devops-ec2</p>

<p>Port: 80</p>

<p>Click Include as pending</p>

<p>Click Create target group</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj6irzehho0kn06sdu29n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj6irzehho0kn06sdu29n.png" alt=" " width="800" height="348"></a></p>

<p>🔹 STEP 3: Create Application Load Balancer (devops-alb)</p>

<p>Go to EC2 → Load Balancers</p>

<p>Click Create load balancer</p>

<p>Choose Application Load Balancer</p>

<p>Basic configuration</p>

<p>Name: devops-alb</p>

<p>Scheme: Internet-facing</p>

<p>IP address type: IPv4</p>

<p>Network mapping</p>

<p>VPC: Default VPC</p>

<p>Availability Zones: Select at least 2 subnets</p>

<p>Security groups</p>

<p>Remove default SG</p>

<p>Select devops-sg</p>

<p>Listeners &amp; routing</p>

<p>Listener: HTTP : 80</p>

<p>Forward to: devops-tg</p>

<p>Click Create load balancer</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd7vk4u548zzbshuwmrt8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd7vk4u548zzbshuwmrt8.png" alt=" " width="800" height="349"></a></p>

<p>🔹 STEP 4: Update EC2 Security Group (VERY IMPORTANT)</p>

<p>Your EC2 instance must allow traffic from the ALB, otherwise it will fail health checks.</p>

<p>Go to EC2 → Instances</p>

<p>Select devops-ec2</p>

<p>Click its Security Group</p>

<p>Edit inbound rules</p>

<p>Add rule:</p>

<p>Type: HTTP</p>

<p>Port: 80</p>

<p>Source: Security group</p>

<p>Select: devops-sg</p>

<p>❌ Do NOT leave it open to 0.0.0.0/0 unless the lab allows it<br>
✅ Best practice is ALB → EC2 only</p>

<p>Save rules.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsshbu6d2vpz21kbf2j55.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsshbu6d2vpz21kbf2j55.png" alt=" " width="800" height="349"></a></p>

<p>🔹 STEP 5: Verify Everything Works<br>
1️⃣ Check Target Group Health</p>

<p>EC2 → Target Groups → devops-tg</p>

<p>Targets should show Healthy</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9cuavtgcaixujws8yyra.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9cuavtgcaixujws8yyra.png" alt=" " width="800" height="349"></a></p>

<p>2️⃣ Access the ALB</p>

<p>EC2 → Load Balancers → devops-alb</p>

<p>Copy DNS name, e.g.:</p>

<p>devops-alb-30750798.us-east-1.elb.amazonaws.com</p>

<p>Open in browser:</p>

<p>http://</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fan0soxpz1honnoi5so1w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fan0soxpz1honnoi5so1w.png" alt=" " width="800" height="216"></a></p>

<p>✅ You should see the Nginx welcome page</p>




<h5>
  
  
  <strong>Resources &amp; Next Steps</strong>
</h5>

<h5>
  
  
  📦 Full Code Repository: <a href="https://github.com/thukhakyawe/100-Days-Of-Cloud-AWS-KodeKloud-Challenges-Solutions" rel="noopener noreferrer">KodeKloud Learning Labs</a>
</h5>

<h5>
  
  
  📖 More Deep Dives: <a href="https://thukhakyawe.hashnode.dev/" rel="noopener noreferrer">Whispering Cloud Insights</a> - Read other technical articles
</h5>

<h5>
  
  
  💬 Join Discussion: <a href="https://dev.to/thukhakyawe_cloud">DEV Community</a> - Share your thoughts and questions
</h5>

<h5>
  
  
  💼 Let's Connect: <a href="https://www.linkedin.com/in/thukhakyawe/" rel="noopener noreferrer">LinkedIn</a> - I'd love to connect with you
</h5>




<h5>
  
  
  <strong>Credits</strong>
</h5>

<h5>
  
  
  • All labs are from: <a href="https://kodekloud.com/" rel="noopener noreferrer">KodeKloud</a>
</h5>

<h5>
  
  
  • I sincerely appreciate your provision of these valuable resources.
</h5>




