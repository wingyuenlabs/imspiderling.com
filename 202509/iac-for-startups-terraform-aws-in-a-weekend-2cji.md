---
Title: IaC for Startups: Terraform + AWS in a Weekend
Description: 
Author: Manuel B
Date: 2025-09-09T21:28:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>Infrastructure as Code (IaC) isn't just for enterprise teams. As a startup, you can build production-ready AWS infrastructure in a weekend using Terraform's reusable modules. This pragmatic approach helps you scale fast while maintaining reliability and cost control.</p>

<h2>
  
  
  Table of Contents
</h2>

<ul>
<li>Why Startups Need IaC From Day One</li>
<li>Weekend Roadmap</li>
<li>Day 1: Building the Foundation</li>
<li>Day 2: Application Infrastructure</li>
<li>CloudWatch Monitoring and Alerts</li>
<li>Deployment Commands</li>
<li>Cost Optimization Tips</li>
<li>Security Best Practices</li>
<li>Production Considerations</li>
<li>Conclusion</li>
</ul>

<h2>
  
  
  Why Startups Need IaC From Day One
</h2>

<p>Many startups delay infrastructure automation, thinking it's premature optimization. This is a costly mistake. IaC provides:</p>

<ul>
<li>
<strong>Reproducible environments</strong> - Dev, staging, and prod are identical</li>
<li>
<strong>Version control</strong> - Infrastructure changes are tracked and reviewable
</li>
<li>
<strong>Cost optimization</strong> - Resources are defined explicitly, preventing drift</li>
<li>
<strong>Team scaling</strong> - New developers can spin up environments instantly</li>
<li>
<strong>Disaster recovery</strong> - Rebuild your entire stack with one command</li>
</ul>

<h2>
  
  
  Weekend Roadmap
</h2>

<h3>
  
  
  Day 1: Foundation &amp; VPC
</h3>

<ul>
<li>Set up Terraform workspace</li>
<li>Build networking foundation</li>
<li>Configure security groups and NACLs</li>
</ul>

<h3>
  
  
  Day 2: Application Infrastructure
</h3>

<ul>
<li>Deploy ECS Fargate cluster</li>
<li>Set up RDS database</li>
<li>Configure monitoring and alerts</li>
</ul>

<h2>
  
  
  Day 1: Building the Foundation
</h2>

<h3>
  
  
  Project Structure
</h3>

<p>Start with a clean, modular structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>terraform/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── ecs/
│   └── rds/
├── shared/
│   └── variables.tf
└── README.md
</code></pre>

</div>



<h3>
  
  
  VPC Module (<code>modules/vpc/main.tf</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"environment"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Environment name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"vpc_cidr"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"CIDR block for VPC"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"10.0.0.0/16"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"availability_zones"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Availability zones"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="p">[</span><span class="s2">"us-east-1a"</span><span class="p">,</span> <span class="s2">"us-east-1b"</span><span class="p">]</span>
<span class="p">}</span>

<span class="c1"># VPC</span>
<span class="nx">resource</span> <span class="s2">"aws_vpc"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">cidr_block</span>           <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">vpc_cidr</span>
  <span class="nx">enable_dns_hostnames</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="nx">enable_dns_support</span>   <span class="p">=</span> <span class="kc">true</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-vpc"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Internet Gateway</span>
<span class="nx">resource</span> <span class="s2">"aws_internet_gateway"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span> <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-igw"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Public Subnets</span>
<span class="nx">resource</span> <span class="s2">"aws_subnet"</span> <span class="s2">"public"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">availability_zones</span><span class="p">)</span>

  <span class="nx">vpc_id</span>                  <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">cidr_block</span>              <span class="p">=</span> <span class="nx">cidrsubnet</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">vpc_cidr</span><span class="p">,</span> <span class="mi">8</span><span class="p">,</span> <span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">)</span>
  <span class="nx">availability_zone</span>       <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">availability_zones</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">]</span>
  <span class="nx">map_public_ip_on_launch</span> <span class="p">=</span> <span class="kc">true</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-public-${count.index + 1}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
    <span class="nx">Type</span>        <span class="p">=</span> <span class="s2">"Public"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Private Subnets</span>
<span class="nx">resource</span> <span class="s2">"aws_subnet"</span> <span class="s2">"private"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">availability_zones</span><span class="p">)</span>

  <span class="nx">vpc_id</span>            <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">cidr_block</span>        <span class="p">=</span> <span class="nx">cidrsubnet</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">vpc_cidr</span><span class="p">,</span> <span class="mi">8</span><span class="p">,</span> <span class="nx">count</span><span class="p">.</span><span class="nx">index</span> <span class="err">+</span> <span class="mi">100</span><span class="p">)</span>
  <span class="nx">availability_zone</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">availability_zones</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">]</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-private-${count.index + 1}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
    <span class="nx">Type</span>        <span class="p">=</span> <span class="s2">"Private"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># NAT Gateways</span>
<span class="nx">resource</span> <span class="s2">"aws_eip"</span> <span class="s2">"nat"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">)</span>

  <span class="nx">domain</span> <span class="p">=</span> <span class="s2">"vpc"</span>
  <span class="nx">depends_on</span> <span class="p">=</span> <span class="p">[</span><span class="nx">aws_internet_gateway</span><span class="p">.</span><span class="nx">main</span><span class="p">]</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-nat-eip-${count.index + 1}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_nat_gateway"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">)</span>

  <span class="nx">allocation_id</span> <span class="p">=</span> <span class="nx">aws_eip</span><span class="p">.</span><span class="nx">nat</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
  <span class="nx">subnet_id</span>     <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-nat-${count.index + 1}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>

  <span class="nx">depends_on</span> <span class="p">=</span> <span class="p">[</span><span class="nx">aws_internet_gateway</span><span class="p">.</span><span class="nx">main</span><span class="p">]</span>
<span class="p">}</span>

<span class="c1"># Route Tables</span>
<span class="nx">resource</span> <span class="s2">"aws_route_table"</span> <span class="s2">"public"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span> <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">route</span> <span class="p">{</span>
    <span class="nx">cidr_block</span> <span class="p">=</span> <span class="s2">"0.0.0.0/0"</span>
    <span class="nx">gateway_id</span> <span class="p">=</span> <span class="nx">aws_internet_gateway</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-public-rt"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_route_table"</span> <span class="s2">"private"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">aws_nat_gateway</span><span class="p">.</span><span class="nx">main</span><span class="p">)</span>

  <span class="nx">vpc_id</span> <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">route</span> <span class="p">{</span>
    <span class="nx">cidr_block</span>     <span class="p">=</span> <span class="s2">"0.0.0.0/0"</span>
    <span class="nx">nat_gateway_id</span> <span class="p">=</span> <span class="nx">aws_nat_gateway</span><span class="p">.</span><span class="nx">main</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-private-rt-${count.index + 1}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Route Table Associations</span>
<span class="nx">resource</span> <span class="s2">"aws_route_table_association"</span> <span class="s2">"public"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">)</span>

  <span class="nx">subnet_id</span>      <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
  <span class="nx">route_table_id</span> <span class="p">=</span> <span class="nx">aws_route_table</span><span class="p">.</span><span class="nx">public</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_route_table_association"</span> <span class="s2">"private"</span> <span class="p">{</span>
  <span class="nx">count</span> <span class="p">=</span> <span class="nx">length</span><span class="p">(</span><span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">private</span><span class="p">)</span>

  <span class="nx">subnet_id</span>      <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">private</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
  <span class="nx">route_table_id</span> <span class="p">=</span> <span class="nx">aws_route_table</span><span class="p">.</span><span class="nx">private</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
<span class="p">}</span>

<span class="c1"># Security Group for ALB</span>
<span class="nx">resource</span> <span class="s2">"aws_security_group"</span> <span class="s2">"alb"</span> <span class="p">{</span>
  <span class="nx">name_prefix</span> <span class="p">=</span> <span class="s2">"${var.environment}-alb-"</span>
  <span class="nx">vpc_id</span>      <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">description</span> <span class="p">=</span> <span class="s2">"HTTP"</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">80</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">80</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
  <span class="p">}</span>

  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">description</span> <span class="p">=</span> <span class="s2">"HTTPS"</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">443</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">443</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
  <span class="p">}</span>

  <span class="nx">egress</span> <span class="p">{</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">0</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">0</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"-1"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-alb-sg"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>

  <span class="nx">lifecycle</span> <span class="p">{</span>
    <span class="nx">create_before_destroy</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Security Group for ECS</span>
<span class="nx">resource</span> <span class="s2">"aws_security_group"</span> <span class="s2">"ecs"</span> <span class="p">{</span>
  <span class="nx">name_prefix</span> <span class="p">=</span> <span class="s2">"${var.environment}-ecs-"</span>
  <span class="nx">vpc_id</span>      <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>

  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">description</span>     <span class="p">=</span> <span class="s2">"HTTP from ALB"</span>
    <span class="nx">from_port</span>       <span class="p">=</span> <span class="mi">80</span>
    <span class="nx">to_port</span>         <span class="p">=</span> <span class="mi">80</span>
    <span class="nx">protocol</span>        <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">security_groups</span> <span class="p">=</span> <span class="p">[</span><span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">alb</span><span class="p">.</span><span class="nx">id</span><span class="p">]</span>
  <span class="p">}</span>

  <span class="nx">egress</span> <span class="p">{</span>
    <span class="nx">from_port</span>   <span class="p">=</span> <span class="mi">0</span>
    <span class="nx">to_port</span>     <span class="p">=</span> <span class="mi">0</span>
    <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"-1"</span>
    <span class="nx">cidr_blocks</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"0.0.0.0/0"</span><span class="p">]</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-ecs-sg"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>

  <span class="nx">lifecycle</span> <span class="p">{</span>
    <span class="nx">create_before_destroy</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Outputs</span>
<span class="nx">output</span> <span class="s2">"vpc_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ID of the VPC"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"public_subnet_ids"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"IDs of the public subnets"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">[*].</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"private_subnet_ids"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"IDs of the private subnets"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">private</span><span class="p">[*].</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"alb_security_group_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ID of the ALB security group"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">alb</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"ecs_security_group_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ID of the ECS security group"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">ecs</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Development Environment (<code>environments/dev/main.tf</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">required_version</span> <span class="p">=</span> <span class="s2">"&gt;= 1.0"</span>
  <span class="nx">required_providers</span> <span class="p">{</span>
    <span class="nx">aws</span> <span class="p">=</span> <span class="p">{</span>
      <span class="nx">source</span>  <span class="p">=</span> <span class="s2">"hashicorp/aws"</span>
      <span class="nx">version</span> <span class="p">=</span> <span class="s2">"~&gt; 5.0"</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">provider</span> <span class="s2">"aws"</span> <span class="p">{</span>
  <span class="nx">region</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">aws_region</span>
<span class="p">}</span>

<span class="c1"># Variables</span>
<span class="nx">variable</span> <span class="s2">"aws_region"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"AWS region"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"us-east-1"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"environment"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Environment name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"dev"</span>
<span class="p">}</span>

<span class="c1"># VPC Module</span>
<span class="nx">module</span> <span class="s2">"vpc"</span> <span class="p">{</span>
  <span class="nx">source</span> <span class="p">=</span> <span class="s2">"../../modules/vpc"</span>

  <span class="nx">environment</span>        <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="nx">vpc_cidr</span>          <span class="p">=</span> <span class="s2">"10.0.0.0/16"</span>
  <span class="nx">availability_zones</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"us-east-1a"</span><span class="p">,</span> <span class="s2">"us-east-1b"</span><span class="p">]</span>
<span class="p">}</span>

<span class="c1"># Outputs</span>
<span class="nx">output</span> <span class="s2">"vpc_id"</span> <span class="p">{</span>
  <span class="nx">value</span> <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">vpc_id</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Day 2: Application Infrastructure
</h2>

<h3>
  
  
  ECS Fargate Module (<code>modules/ecs/main.tf</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"environment"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Environment name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"vpc_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"VPC ID"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"private_subnet_ids"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Private subnet IDs"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"public_subnet_ids"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Public subnet IDs"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"ecs_security_group_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ECS security group ID"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"alb_security_group_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ALB security group ID"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"app_name"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Application name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"webapp"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"app_port"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Application port"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">3000</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"desired_count"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Desired number of tasks"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">2</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"cpu"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"CPU units"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">256</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"memory"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Memory in MB"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">512</span>
<span class="p">}</span>

<span class="c1"># ECS Cluster</span>
<span class="nx">resource</span> <span class="s2">"aws_ecs_cluster"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">name</span> <span class="p">=</span> <span class="s2">"${var.environment}-cluster"</span>

  <span class="nx">setting</span> <span class="p">{</span>
    <span class="nx">name</span>  <span class="p">=</span> <span class="s2">"containerInsights"</span>
    <span class="nx">value</span> <span class="p">=</span> <span class="s2">"enabled"</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-cluster"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># ECS Task Definition</span>
<span class="nx">resource</span> <span class="s2">"aws_ecs_task_definition"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">family</span>                   <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}"</span>
  <span class="nx">execution_role_arn</span>       <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">ecs_task_execution_role</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">task_role_arn</span>           <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">ecs_task_role</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">network_mode</span>            <span class="p">=</span> <span class="s2">"awsvpc"</span>
  <span class="nx">requires_compatibilities</span> <span class="p">=</span> <span class="p">[</span><span class="s2">"FARGATE"</span><span class="p">]</span>
  <span class="nx">cpu</span>                     <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">cpu</span>
  <span class="nx">memory</span>                  <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">memory</span>

  <span class="nx">container_definitions</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">([</span>
    <span class="p">{</span>
      <span class="nx">name</span>  <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">app_name</span>
      <span class="nx">image</span> <span class="p">=</span> <span class="s2">"nginx:latest"</span>  <span class="c1"># Replace with your app image</span>

      <span class="nx">portMappings</span> <span class="p">=</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="nx">containerPort</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">app_port</span>
          <span class="nx">protocol</span>      <span class="p">=</span> <span class="s2">"tcp"</span>
        <span class="p">}</span>
      <span class="p">]</span>

      <span class="nx">logConfiguration</span> <span class="p">=</span> <span class="p">{</span>
        <span class="nx">logDriver</span> <span class="p">=</span> <span class="s2">"awslogs"</span>
        <span class="nx">options</span> <span class="p">=</span> <span class="p">{</span>
          <span class="nx">awslogs-group</span>         <span class="p">=</span> <span class="nx">aws_cloudwatch_log_group</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">name</span>
          <span class="nx">awslogs-region</span>        <span class="p">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">aws_region</span><span class="p">.</span><span class="nx">current</span><span class="p">.</span><span class="nx">name</span>
          <span class="nx">awslogs-stream-prefix</span> <span class="p">=</span> <span class="s2">"ecs"</span>
        <span class="p">}</span>
      <span class="p">}</span>

      <span class="nx">environment</span> <span class="p">=</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="nx">name</span>  <span class="p">=</span> <span class="s2">"ENVIRONMENT"</span>
          <span class="nx">value</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">}</span>
  <span class="p">])</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Application Load Balancer</span>
<span class="nx">resource</span> <span class="s2">"aws_lb"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">name</span>               <span class="p">=</span> <span class="s2">"${var.environment}-alb"</span>
  <span class="nx">internal</span>           <span class="p">=</span> <span class="kc">false</span>
  <span class="nx">load_balancer_type</span> <span class="p">=</span> <span class="s2">"application"</span>
  <span class="nx">security_groups</span>    <span class="p">=</span> <span class="p">[</span><span class="nx">var</span><span class="p">.</span><span class="nx">alb_security_group_id</span><span class="p">]</span>
  <span class="nx">subnets</span>           <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">public_subnet_ids</span>

  <span class="nx">enable_deletion_protection</span> <span class="p">=</span> <span class="kc">false</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-alb"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_lb_target_group"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}-tg"</span>
  <span class="nx">port</span>        <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">app_port</span>
  <span class="nx">protocol</span>    <span class="p">=</span> <span class="s2">"HTTP"</span>
  <span class="nx">vpc_id</span>      <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">vpc_id</span>
  <span class="nx">target_type</span> <span class="p">=</span> <span class="s2">"ip"</span>

  <span class="nx">health_check</span> <span class="p">{</span>
    <span class="nx">enabled</span>             <span class="p">=</span> <span class="kc">true</span>
    <span class="nx">healthy_threshold</span>   <span class="p">=</span> <span class="s2">"3"</span>
    <span class="nx">interval</span>            <span class="p">=</span> <span class="s2">"30"</span>
    <span class="nx">matcher</span>             <span class="p">=</span> <span class="s2">"200"</span>
    <span class="nx">path</span>                <span class="p">=</span> <span class="s2">"/"</span>
    <span class="nx">port</span>                <span class="p">=</span> <span class="s2">"traffic-port"</span>
    <span class="nx">protocol</span>            <span class="p">=</span> <span class="s2">"HTTP"</span>
    <span class="nx">timeout</span>             <span class="p">=</span> <span class="s2">"5"</span>
    <span class="nx">unhealthy_threshold</span> <span class="p">=</span> <span class="s2">"2"</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}-tg"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_lb_listener"</span> <span class="s2">"front_end"</span> <span class="p">{</span>
  <span class="nx">load_balancer_arn</span> <span class="p">=</span> <span class="nx">aws_lb</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">port</span>              <span class="p">=</span> <span class="s2">"80"</span>
  <span class="nx">protocol</span>          <span class="p">=</span> <span class="s2">"HTTP"</span>

  <span class="nx">default_action</span> <span class="p">{</span>
    <span class="nx">type</span>             <span class="p">=</span> <span class="s2">"forward"</span>
    <span class="nx">target_group_arn</span> <span class="p">=</span> <span class="nx">aws_lb_target_group</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">arn</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># ECS Service</span>
<span class="nx">resource</span> <span class="s2">"aws_ecs_service"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">name</span>            <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}"</span>
  <span class="nx">cluster</span>         <span class="p">=</span> <span class="nx">aws_ecs_cluster</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">task_definition</span> <span class="p">=</span> <span class="nx">aws_ecs_task_definition</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">desired_count</span>   <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">desired_count</span>
  <span class="nx">launch_type</span>     <span class="p">=</span> <span class="s2">"FARGATE"</span>

  <span class="nx">network_configuration</span> <span class="p">{</span>
    <span class="nx">security_groups</span>  <span class="p">=</span> <span class="p">[</span><span class="nx">var</span><span class="p">.</span><span class="nx">ecs_security_group_id</span><span class="p">]</span>
    <span class="nx">subnets</span>         <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">private_subnet_ids</span>
    <span class="nx">assign_public_ip</span> <span class="p">=</span> <span class="kc">false</span>
  <span class="p">}</span>

  <span class="nx">load_balancer</span> <span class="p">{</span>
    <span class="nx">target_group_arn</span> <span class="p">=</span> <span class="nx">aws_lb_target_group</span><span class="p">.</span><span class="nx">app</span><span class="p">.</span><span class="nx">arn</span>
    <span class="nx">container_name</span>   <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">app_name</span>
    <span class="nx">container_port</span>   <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">app_port</span>
  <span class="p">}</span>

  <span class="nx">depends_on</span> <span class="p">=</span> <span class="p">[</span><span class="nx">aws_lb_listener</span><span class="p">.</span><span class="nx">front_end</span><span class="p">]</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># CloudWatch Log Group</span>
<span class="nx">resource</span> <span class="s2">"aws_cloudwatch_log_group"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">name</span>              <span class="p">=</span> <span class="s2">"/ecs/${var.environment}/${var.app_name}"</span>
  <span class="nx">retention_in_days</span> <span class="p">=</span> <span class="mi">30</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-${var.app_name}-logs"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># IAM Roles</span>
<span class="nx">resource</span> <span class="s2">"aws_iam_role"</span> <span class="s2">"ecs_task_execution_role"</span> <span class="p">{</span>
  <span class="nx">name</span> <span class="p">=</span> <span class="s2">"${var.environment}-ecsTaskExecutionRole"</span>

  <span class="nx">assume_role_policy</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span>
    <span class="nx">Version</span> <span class="p">=</span> <span class="s2">"2012-10-17"</span>
    <span class="nx">Statement</span> <span class="p">=</span> <span class="p">[</span>
      <span class="p">{</span>
        <span class="nx">Action</span> <span class="p">=</span> <span class="s2">"sts:AssumeRole"</span>
        <span class="nx">Effect</span> <span class="p">=</span> <span class="s2">"Allow"</span>
        <span class="nx">Principal</span> <span class="p">=</span> <span class="p">{</span>
          <span class="nx">Service</span> <span class="p">=</span> <span class="s2">"ecs-tasks.amazonaws.com"</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">]</span>
  <span class="p">})</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-ecsTaskExecutionRole"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_iam_role_policy_attachment"</span> <span class="s2">"ecs_task_execution_role"</span> <span class="p">{</span>
  <span class="nx">role</span>       <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">ecs_task_execution_role</span><span class="p">.</span><span class="nx">name</span>
  <span class="nx">policy_arn</span> <span class="p">=</span> <span class="s2">"arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_iam_role"</span> <span class="s2">"ecs_task_role"</span> <span class="p">{</span>
  <span class="nx">name</span> <span class="p">=</span> <span class="s2">"${var.environment}-ecsTaskRole"</span>

  <span class="nx">assume_role_policy</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span>
    <span class="nx">Version</span> <span class="p">=</span> <span class="s2">"2012-10-17"</span>
    <span class="nx">Statement</span> <span class="p">=</span> <span class="p">[</span>
      <span class="p">{</span>
        <span class="nx">Action</span> <span class="p">=</span> <span class="s2">"sts:AssumeRole"</span>
        <span class="nx">Effect</span> <span class="p">=</span> <span class="s2">"Allow"</span>
        <span class="nx">Principal</span> <span class="p">=</span> <span class="p">{</span>
          <span class="nx">Service</span> <span class="p">=</span> <span class="s2">"ecs-tasks.amazonaws.com"</span>
        <span class="p">}</span>
      <span class="p">}</span>
    <span class="p">]</span>
  <span class="p">})</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-ecsTaskRole"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Data Sources</span>
<span class="nx">data</span> <span class="s2">"aws_region"</span> <span class="s2">"current"</span> <span class="p">{}</span>

<span class="c1"># Outputs</span>
<span class="nx">output</span> <span class="s2">"cluster_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ECS cluster ID"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_ecs_cluster</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"alb_dns_name"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ALB DNS name"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_lb</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">dns_name</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"alb_zone_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"ALB zone ID"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_lb</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">zone_id</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  RDS Module (<code>modules/rds/main.tf</code>)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"environment"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Environment name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"vpc_id"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"VPC ID"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"private_subnet_ids"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Private subnet IDs"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"allowed_security_groups"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Security groups allowed to access RDS"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">list</span><span class="p">(</span><span class="nx">string</span><span class="p">)</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="p">[]</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"db_name"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Database name"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"appdb"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"db_username"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Database username"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"dbadmin"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"db_password"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Database password"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">sensitive</span>   <span class="p">=</span> <span class="kc">true</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"instance_class"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"RDS instance class"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="s2">"db.t3.micro"</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"allocated_storage"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Allocated storage in GB"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">20</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"backup_retention_period"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Backup retention period in days"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">number</span>
  <span class="nx">default</span>     <span class="p">=</span> <span class="mi">7</span>
<span class="p">}</span>

<span class="c1"># Security Group for RDS</span>
<span class="nx">resource</span> <span class="s2">"aws_security_group"</span> <span class="s2">"rds"</span> <span class="p">{</span>
  <span class="nx">name_prefix</span> <span class="p">=</span> <span class="s2">"${var.environment}-rds-"</span>
  <span class="nx">vpc_id</span>      <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">vpc_id</span>

  <span class="nx">ingress</span> <span class="p">{</span>
    <span class="nx">description</span>     <span class="p">=</span> <span class="s2">"MySQL/Aurora"</span>
    <span class="nx">from_port</span>       <span class="p">=</span> <span class="mi">3306</span>
    <span class="nx">to_port</span>         <span class="p">=</span> <span class="mi">3306</span>
    <span class="nx">protocol</span>        <span class="p">=</span> <span class="s2">"tcp"</span>
    <span class="nx">security_groups</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">allowed_security_groups</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-rds-sg"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>

  <span class="nx">lifecycle</span> <span class="p">{</span>
    <span class="nx">create_before_destroy</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># DB Subnet Group</span>
<span class="nx">resource</span> <span class="s2">"aws_db_subnet_group"</span> <span class="s2">"default"</span> <span class="p">{</span>
  <span class="nx">name</span>       <span class="p">=</span> <span class="s2">"${var.environment}-db-subnet-group"</span>
  <span class="nx">subnet_ids</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">private_subnet_ids</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-db-subnet-group"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># RDS Instance</span>
<span class="nx">resource</span> <span class="s2">"aws_db_instance"</span> <span class="s2">"default"</span> <span class="p">{</span>
  <span class="nx">identifier</span>     <span class="p">=</span> <span class="s2">"${var.environment}-database"</span>
  <span class="nx">engine</span>         <span class="p">=</span> <span class="s2">"mysql"</span>
  <span class="nx">engine_version</span> <span class="p">=</span> <span class="s2">"8.0"</span>
  <span class="nx">instance_class</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">instance_class</span>

  <span class="nx">allocated_storage</span>     <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">allocated_storage</span>
  <span class="nx">max_allocated_storage</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">allocated_storage</span> <span class="p">*</span> <span class="mi">2</span>

  <span class="nx">db_name</span>  <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">db_name</span>
  <span class="nx">username</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">db_username</span>
  <span class="nx">password</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">db_password</span>

  <span class="nx">vpc_security_group_ids</span> <span class="p">=</span> <span class="p">[</span><span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">rds</span><span class="p">.</span><span class="nx">id</span><span class="p">]</span>
  <span class="nx">db_subnet_group_name</span>   <span class="p">=</span> <span class="nx">aws_db_subnet_group</span><span class="p">.</span><span class="nx">default</span><span class="p">.</span><span class="nx">name</span>

  <span class="nx">backup_retention_period</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">backup_retention_period</span>
  <span class="nx">backup_window</span>          <span class="p">=</span> <span class="s2">"03:00-04:00"</span>
  <span class="nx">maintenance_window</span>     <span class="p">=</span> <span class="s2">"sun:04:00-sun:05:00"</span>

  <span class="nx">skip_final_snapshot</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="nx">deletion_protection</span> <span class="p">=</span> <span class="kc">false</span>

  <span class="nx">performance_insights_enabled</span> <span class="p">=</span> <span class="kc">false</span>
  <span class="nx">monitoring_interval</span>         <span class="p">=</span> <span class="mi">0</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-database"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Outputs</span>
<span class="nx">output</span> <span class="s2">"rds_hostname"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"RDS instance hostname"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_db_instance</span><span class="p">.</span><span class="nx">default</span><span class="p">.</span><span class="nx">address</span>
  <span class="nx">sensitive</span>   <span class="p">=</span> <span class="kc">true</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"rds_port"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"RDS instance port"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_db_instance</span><span class="p">.</span><span class="nx">default</span><span class="p">.</span><span class="nx">port</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"rds_username"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"RDS instance root username"</span>
  <span class="nx">value</span>       <span class="p">=</span> <span class="nx">aws_db_instance</span><span class="p">.</span><span class="nx">default</span><span class="p">.</span><span class="nx">username</span>
  <span class="nx">sensitive</span>   <span class="p">=</span> <span class="kc">true</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Complete Environment Configuration
</h3>

<p>Update <code>environments/dev/main.tf</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Add to existing dev environment</span>
<span class="nx">module</span> <span class="s2">"ecs"</span> <span class="p">{</span>
  <span class="nx">source</span> <span class="p">=</span> <span class="s2">"../../modules/ecs"</span>

  <span class="nx">environment</span>             <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="nx">vpc_id</span>                 <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">vpc_id</span>
  <span class="nx">private_subnet_ids</span>     <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">private_subnet_ids</span>
  <span class="nx">public_subnet_ids</span>      <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">public_subnet_ids</span>
  <span class="nx">ecs_security_group_id</span>  <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">ecs_security_group_id</span>
  <span class="nx">alb_security_group_id</span>  <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">alb_security_group_id</span>

  <span class="nx">app_name</span>      <span class="p">=</span> <span class="s2">"myapp"</span>
  <span class="nx">desired_count</span> <span class="p">=</span> <span class="mi">1</span>  <span class="c1"># Lower for dev</span>
  <span class="nx">cpu</span>          <span class="p">=</span> <span class="mi">256</span>
  <span class="nx">memory</span>       <span class="p">=</span> <span class="mi">512</span>
<span class="p">}</span>

<span class="nx">module</span> <span class="s2">"rds"</span> <span class="p">{</span>
  <span class="nx">source</span> <span class="p">=</span> <span class="s2">"../../modules/rds"</span>

  <span class="nx">environment</span>               <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="nx">vpc_id</span>                   <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">vpc_id</span>
  <span class="nx">private_subnet_ids</span>       <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">private_subnet_ids</span>
  <span class="nx">allowed_security_groups</span>  <span class="p">=</span> <span class="p">[</span><span class="nx">module</span><span class="p">.</span><span class="nx">vpc</span><span class="p">.</span><span class="nx">ecs_security_group_id</span><span class="p">]</span>

  <span class="nx">db_password</span>             <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">db_password</span>
  <span class="nx">instance_class</span>          <span class="p">=</span> <span class="s2">"db.t3.micro"</span>
  <span class="nx">backup_retention_period</span> <span class="p">=</span> <span class="mi">1</span>  <span class="c1"># Minimal for dev</span>
<span class="p">}</span>

<span class="nx">variable</span> <span class="s2">"db_password"</span> <span class="p">{</span>
  <span class="nx">description</span> <span class="p">=</span> <span class="s2">"Database password"</span>
  <span class="nx">type</span>        <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">sensitive</span>   <span class="p">=</span> <span class="kc">true</span>
<span class="p">}</span>

<span class="nx">output</span> <span class="s2">"alb_dns_name"</span> <span class="p">{</span>
  <span class="nx">value</span> <span class="p">=</span> <span class="nx">module</span><span class="p">.</span><span class="nx">ecs</span><span class="p">.</span><span class="nx">alb_dns_name</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  CloudWatch Monitoring and Alerts
</h2>

<p>Add monitoring to your modules:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># CloudWatch Alarms for ECS</span>
<span class="nx">resource</span> <span class="s2">"aws_cloudwatch_metric_alarm"</span> <span class="s2">"high_cpu"</span> <span class="p">{</span>
  <span class="nx">alarm_name</span>          <span class="p">=</span> <span class="s2">"${var.environment}-high-cpu"</span>
  <span class="nx">comparison_operator</span> <span class="p">=</span> <span class="s2">"GreaterThanThreshold"</span>
  <span class="nx">evaluation_periods</span>  <span class="p">=</span> <span class="s2">"2"</span>
  <span class="nx">metric_name</span>         <span class="p">=</span> <span class="s2">"CPUUtilization"</span>
  <span class="nx">namespace</span>           <span class="p">=</span> <span class="s2">"AWS/ECS"</span>
  <span class="nx">period</span>              <span class="p">=</span> <span class="s2">"120"</span>
  <span class="nx">statistic</span>           <span class="p">=</span> <span class="s2">"Average"</span>
  <span class="nx">threshold</span>           <span class="p">=</span> <span class="s2">"80"</span>
  <span class="nx">alarm_description</span>   <span class="p">=</span> <span class="s2">"This metric monitors ecs cpu utilization"</span>

  <span class="nx">dimensions</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">ServiceName</span> <span class="p">=</span> <span class="nx">aws_ecs_service</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">name</span>
    <span class="nx">ClusterName</span> <span class="p">=</span> <span class="nx">aws_ecs_cluster</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">name</span>
  <span class="p">}</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-high-cpu-alarm"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1"># SNS Topic for Alerts</span>
<span class="nx">resource</span> <span class="s2">"aws_sns_topic"</span> <span class="s2">"alerts"</span> <span class="p">{</span>
  <span class="nx">name</span> <span class="p">=</span> <span class="s2">"${var.environment}-alerts"</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span>        <span class="p">=</span> <span class="s2">"${var.environment}-alerts"</span>
    <span class="nx">Environment</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Deployment Commands
</h2>

<p>Deploy your infrastructure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Development</span>
<span class="nb">cd </span>environments/dev
terraform init
terraform plan <span class="nt">-var</span><span class="o">=</span><span class="s2">"db_password=your-secure-password"</span>
terraform apply <span class="nt">-var</span><span class="o">=</span><span class="s2">"db_password=your-secure-password"</span>

<span class="c"># Production (copy dev to prod with appropriate sizing)</span>
<span class="nb">cd</span> ../prod
terraform init
terraform plan <span class="nt">-var</span><span class="o">=</span><span class="s2">"db_password=your-secure-password"</span>
terraform apply <span class="nt">-var</span><span class="o">=</span><span class="s2">"db_password=your-secure-password"</span>
</code></pre>

</div>



<h2>
  
  
  Cost Optimization Tips
</h2>

<h3>
  
  
  Right-Sizing Resources
</h3>

<ul>
<li>
<strong>Dev</strong>: t3.micro instances, minimal RDS</li>
<li>
<strong>Prod</strong>: Start small and scale based on metrics</li>
<li>Use Fargate Spot for non-critical workloads</li>
</ul>

<h3>
  
  
  Resource Scheduling
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Auto-scaling for ECS</span>
<span class="nx">resource</span> <span class="s2">"aws_appautoscaling_target"</span> <span class="s2">"ecs_target"</span> <span class="p">{</span>
  <span class="nx">max_capacity</span>       <span class="p">=</span> <span class="mi">10</span>
  <span class="nx">min_capacity</span>       <span class="p">=</span> <span class="mi">2</span>
  <span class="nx">resource_id</span>        <span class="p">=</span> <span class="s2">"service/${aws_ecs_cluster.main.name}/${aws_ecs_service.main.name}"</span>
  <span class="nx">scalable_dimension</span> <span class="p">=</span> <span class="s2">"ecs:service:DesiredCount"</span>
  <span class="nx">service_namespace</span>  <span class="p">=</span> <span class="s2">"ecs"</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_appautoscaling_policy"</span> <span class="s2">"scale_up"</span> <span class="p">{</span>
  <span class="nx">name</span>               <span class="p">=</span> <span class="s2">"${var.environment}-scale-up"</span>
  <span class="nx">policy_type</span>        <span class="p">=</span> <span class="s2">"TargetTrackingScaling"</span>
  <span class="nx">resource_id</span>        <span class="p">=</span> <span class="nx">aws_appautoscaling_target</span><span class="p">.</span><span class="nx">ecs_target</span><span class="p">.</span><span class="nx">resource_id</span>
  <span class="nx">scalable_dimension</span> <span class="p">=</span> <span class="nx">aws_appautoscaling_target</span><span class="p">.</span><span class="nx">ecs_target</span><span class="p">.</span><span class="nx">scalable_dimension</span>
  <span class="nx">service_namespace</span>  <span class="p">=</span> <span class="nx">aws_appautoscaling_target</span><span class="p">.</span><span class="nx">ecs_target</span><span class="p">.</span><span class="nx">service_namespace</span>

  <span class="nx">target_tracking_scaling_policy_configuration</span> <span class="p">{</span>
    <span class="nx">predefined_metric_specification</span> <span class="p">{</span>
      <span class="nx">predefined_metric_type</span> <span class="p">=</span> <span class="s2">"ECSServiceAverageCPUUtilization"</span>
    <span class="p">}</span>
    <span class="nx">target_value</span> <span class="p">=</span> <span class="mf">70.0</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Security Best Practices
</h2>

<h3>
  
  
  Secrets Management
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Use AWS Secrets Manager for sensitive data</span>
<span class="nx">resource</span> <span class="s2">"aws_secretsmanager_secret"</span> <span class="s2">"db_password"</span> <span class="p">{</span>
  <span class="nx">name</span> <span class="p">=</span> <span class="s2">"${var.environment}/database/password"</span>
<span class="p">}</span>

<span class="nx">resource</span> <span class="s2">"aws_secretsmanager_secret_version"</span> <span class="s2">"db_password"</span> <span class="p">{</span>
  <span class="nx">secret_id</span>     <span class="p">=</span> <span class="nx">aws_secretsmanager_secret</span><span class="p">.</span><span class="nx">db_password</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">secret_string</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">db_password</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Network Security
</h3>

<ul>
<li>All databases in private subnets</li>
<li>Security groups with minimal access</li>
<li>VPC Flow Logs for network monitoring</li>
<li>WAF for public-facing applications</li>
</ul>

<h2>
  
  
  Production Considerations
</h2>

<h3>
  
  
  State Management
</h3>

<p>Use remote state with S3 and DynamoDB locking:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">terraform</span> <span class="p">{</span>
  <span class="nx">backend</span> <span class="s2">"s3"</span> <span class="p">{</span>
    <span class="nx">bucket</span>         <span class="p">=</span> <span class="s2">"your-terraform-state-bucket"</span>
    <span class="nx">key</span>            <span class="p">=</span> <span class="s2">"environments/prod/terraform.tfstate"</span>
    <span class="nx">region</span>         <span class="p">=</span> <span class="s2">"us-east-1"</span>
    <span class="nx">encrypt</span>        <span class="p">=</span> <span class="kc">true</span>
    <span class="nx">dynamodb_table</span> <span class="p">=</span> <span class="s2">"terraform-state-lock"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Multi-Environment Strategy
</h3>

<ul>
<li>Separate AWS accounts for prod</li>
<li>Environment-specific variable files</li>
<li>Automated testing of Terraform changes</li>
<li>GitOps workflow with pull request reviews</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>You now have production-ready AWS infrastructure that scales with your startup growth. The modular Terraform approach provides enterprise-grade reliability while remaining startup-friendly in complexity and cost.</p>

<p>This infrastructure foundation delivers:</p>

<ul>
<li>
<strong>High availability</strong> across multiple AZs with automatic failover</li>
<li>
<strong>Scalable containerized applications</strong> using ECS Fargate</li>
<li>
<strong>Managed database services</strong> with automated backups and maintenance</li>
<li>
<strong>Comprehensive monitoring</strong> with CloudWatch metrics and alerts</li>
<li>
<strong>Cost optimization</strong> through right-sizing and intelligent auto-scaling</li>
</ul>

<p>The weekend time investment in infrastructure automation creates a solid foundation that supports rapid scaling as your startup grows, while maintaining the operational reliability your customers expect.</p>

<p><strong>Need help building production-ready AWS infrastructure for your startup?</strong> I specialize in Terraform consulting and can set up scalable, cost-optimized cloud infrastructure that grows with your business. <a href="https://manuelbauka.dev" rel="noopener noreferrer">Check out my DevOps services and portfolio</a> or <a href="https://manuelbauka.dev/#contact" rel="noopener noreferrer">contact me directly</a> to discuss your infrastructure needs.</p>




<p><em>This is part 2 of my "DevOps for Startups" series. Part 1 covered automated React deployment pipelines with GitHub Actions and AWS</em></p>

