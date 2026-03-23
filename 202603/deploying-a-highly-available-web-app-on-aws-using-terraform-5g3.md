---
Title: Deploying a Highly Available Web App on AWS Using Terraform
Description: 
Author: Mary Mutua
Date: 2026-03-23T22:13:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>Today’s Terraform work took me from a single configurable EC2 web server to a clustered, load-balanced deployment on AWS.</p>

<p>The two big ideas I focused on were:</p>

<ul>
<li>using <strong>input variables</strong> to remove hardcoded values</li>
<li>moving from a <strong>single server</strong> setup to a <strong>highly available architecture</strong> using an <strong>Application Load Balancer (ALB)</strong> and an <strong>Auto Scaling Group (ASG)</strong>
</li>
</ul>

<h2>
  
  
  From Hardcoded to Configurable
</h2>

<p>A hardcoded Terraform setup works once, but it becomes difficult to reuse. If region, instance type, and port are written directly in <code>main.tf</code>, every change means editing the infrastructure code itself.</p>

<p>That’s where input variables help.</p>

<p>With Terraform variables, I could define settings like:</p>

<ul>
<li>AWS region</li>
<li>EC2 instance type</li>
<li>application port</li>
<li>environment name</li>
<li>server name</li>
</ul>

<p>and then reference them in Terraform with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">var</span><span class="err">.&lt;</span><span class="nx">name</span><span class="err">&gt;</span> 

</code></pre>

</div>



<p>For example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"server_port"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"The port the server will use for HTTP requests"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">8080</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"instance_type"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"EC2 instance type"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"t3.micro"</span>
<span class="p">}</span>

</code></pre>

</div>



<p>This made the infrastructure reusable without changing the main logic.</p>

<h2>
  
  
  Building the Configurable Web Server
</h2>

<p>For the first lab, I deployed a single EC2 web server using:</p>

<ul>
<li>an AWS provider configured with a variable-driven region</li>
<li>a security group that allowed traffic on the configured application port</li>
<li>an EC2 instance using user_data to install and run a simple web page</li>
</ul>

<p>I also used Terraform data sources to look up:</p>

<ul>
<li>the default VPC</li>
<li>default subnets</li>
<li>a current Ubuntu AMI</li>
</ul>

<p>The important thing here was that the infrastructure behavior came from variables instead of hardcoded values.</p>

<p>After <code>terraform apply</code>, I was able to open the EC2 public DNS in the browser and confirm the page was working.</p>

<h2>
  
  
  From Single Server to Clustered Architecture
</h2>

<p>The next step was to make the app more resilient.</p>

<p>A single EC2 instance is simple, but if that instance fails, the application goes down with it. To improve that, I built a clustered version using:</p>

<ul>
<li>Application Load Balancer (ALB)</li>
<li>Target Group</li>
<li>Listener</li>
<li>Launch Template</li>
<li>Auto Scaling Group (ASG)</li>
</ul>

<p>The request flow looked like this:</p>

<p><code>Browser -&gt; ALB -&gt; Target Group -&gt; EC2 instances in Auto Scaling Group<br>
</code><br>
In this design:</p>

<ul>
<li>the ALB receives traffic from users</li>
<li>the listener forwards traffic to the target group</li>
<li>the target group tracks healthy EC2 instances</li>
<li>the launch template defines how instances should be created</li>
<li>the ASG keeps the desired number of instances running</li>
</ul>

<p>I also used this Terraform data source to fetch availability zones dynamically:<br>
<code>data "aws_availability_zones" "all" {}<br>
</code><br>
That was important because the clustered setup needed to work across multiple subnets and availability zones.</p>

<h2>
  
  
  Why This Matters
</h2>

<p>The difference between the two deployments became much clearer as I worked through them.</p>

<h2>
  
  
  Configurable web server
</h2>

<ul>
<li>one EC2 instance</li>
<li>simpler to understand</li>
<li>direct browser-to-instance access</li>
<li>variables make the setup reusable</li>
</ul>

<h2>
  
  
  Clustered web server
</h2>

<ul>
<li>multiple EC2 instances</li>
<li>traffic goes through the ALB</li>
<li>instances are managed by the ASG</li>
<li>better availability and closer to real production architecture</li>
</ul>

<h2>
  
  
  Documentation Matters Too
</h2>

<p>Part of today’s work was also exploring the Terraform documentation for:</p>

<ul>
<li> aws_autoscaling_group</li>
<li> aws_lb</li>
<li> input variables</li>
</ul>

<p>This reminded me that learning Terraform is not just about writing code. It is also about understanding the official docs, knowing what resource arguments mean, and being able to connect Terraform blocks to real architecture decisions.</p>

<h2>
  
  
  Challenges I Ran Into
</h2>

<p>Today was not easy. I had to think through:</p>

<ul>
<li>the difference between <code>variable</code>, <code>resource</code>, and <code>data</code>
</li>
<li>how load balancing changes application access</li>
<li>AWS Free Tier considerations</li>
<li>how scaling architecture differs from a single-server deployment</li>
</ul>

<p>But seeing both deployments work in the browser made everything much more concrete.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>The biggest lesson for me today was this:</p>

<ul>
<li>input variables make Terraform flexible</li>
<li>
<strong>ALBs</strong> and <strong>ASGs</strong> make infrastructure more resilient</li>
</ul>

<p>By the end of the day, I had successfully deployed:</p>

<ul>
<li>a configurable single EC2 web server</li>
<li>a clustered web app behind an Application Load Balancer</li>
</ul>

<p>That was a big step in understanding how Terraform moves from simple infrastructure to something much closer to real-world cloud architecture.</p>

<h2>
  
  
  Follow My Journey
</h2>

<p>This is Day 4 of my 30-Day Terraform Challenge.</p>

<p>See you on Day 5 🚀</p>

