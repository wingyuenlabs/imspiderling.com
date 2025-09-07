---
Title: Day 44: Relational Database Service in AWS
Description: 
Author: Udoh Deborah
Date: 2025-09-07T20:31:31.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What is AWS Relational Database Service (RDS)?
</h2>

<p>Amazon RDS is a managed database service by AWS that makes it easy to set up, operate, and scale relational databases in the cloud.</p>

<p>A relational database organizes data into tables (rows and columns) with relationships between them, and it uses SQL (Structured Query Language) for querying and data manipulation.</p>

<p>Instead of managing servers, patching, backups, and scaling manually, RDS handles these heavy-lifting tasks for you.</p>

<h2>
  
  
  Why use RDS?
</h2>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1.  Fully managed – AWS automates provisioning, patching, backups, and monitoring.
2.  Scalability – Easily scale compute and storage vertically or horizontally (read replicas).
3.  High availability – Multi-AZ deployments provide automatic failover for production systems.
4.  Security – Supports VPC isolation, IAM integration, encryption at rest (KMS), and encryption in transit (SSL/TLS).
5.  Cost-effective – Pay only for what you use; supports free tier for testing.
</code></pre>

</div>

<p>Supported Database Engines in RDS</p>

<p>AWS RDS supports multiple relational database engines:<br>
    • Amazon Aurora (MySQL &amp; PostgreSQL-compatible, AWS-optimized, high performance)<br>
    • MySQL<br>
    • PostgreSQL<br>
    • MariaDB<br>
    • Oracle (license-included or BYOL)<br>
    • Microsoft SQL Server</p>

<p>Aurora is AWS’s own DB engine — up to 5x faster than MySQL and 3x faster than PostgreSQL.</p>

<p>Core Components of RDS<br>
    1.  DB Instance → The actual database environment (compute + storage).<br>
    2.  Endpoint &amp; Port → The hostname &amp; port to connect your applications.<br>
    3.  Security Groups → Control inbound/outbound access to the DB.<br>
    4.  Parameter Groups → Configurations for the DB engine.<br>
    5.  Option Groups → Add-ons like Oracle TDE, SQL Server features, etc.<br>
    6.  Subnet Group → Defines which subnets RDS can launch into.</p>

<p>Key Features<br>
    1.  Multi-AZ Deployment → Ensures high availability by maintaining a standby copy.<br>
    2.  Read Replicas → Scale read-heavy workloads by creating copies for read-only queries.<br>
    3.  Automated Backups → Point-in-time recovery, retention up to 35 days.<br>
    4.  Monitoring → CloudWatch metrics, Performance Insights, Enhanced Monitoring.<br>
    5.  Encryption → At-rest (via AWS KMS) and in-transit (SSL).<br>
    6.  IAM DB Authentication → Connect without passwords, using IAM tokens.</p>

<p>How RDS Works in Practice</p>

<p>Imagine you’re building an e-commerce application:<br>
    • Your app runs on EC2 instances or ECS/EKS containers.<br>
    • Instead of hosting MySQL on the EC2 VM (where you manage everything), you create an RDS MySQL instance.<br>
    • The app connects via RDS endpoint:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>mysql -h mydb.rds.amazonaws.com -P 3306 -u admin -p


</code></pre>

</div>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>• AWS automatically takes care of:
• OS/database patching
• Disk scaling
• Automatic failover (if Multi-AZ)
• Backups for disaster recovery
</code></pre>

</div>

<p>You just focus on schema design and, queries. </p>

<p>When to Use RDS</p>

<ul>
<li>For traditional relational database workloads (finance, e-commerce, ERP, CRM, etc.)</li>
<li>When you need high availability and automatic scaling.</li>
<li>If you want to reduce DBA overhead (patching, backups, monitoring).</li>
<li>For applications that use SQL-based relational data models.</li>
</ul>

<p>Step-by-step: Create a Free-Tier MySQL RDS and connect from EC2</p>

<p>Plan at a glance<br>
    1.  Create a MySQL RDS (free tier) — note endpoint &amp; port.<br>
    2.  Create an EC2 instance (same VPC/subnet as RDS or with network path).<br>
    3.  Create an IAM role with RDS access (use for enhanced security if you use IAM DB auth; otherwise needed for instance profile tasks).<br>
    4.  Attach the IAM role to EC2 instance.<br>
    5.  Configure Security Groups so EC2 can reach RDS on port 3306.<br>
    6.  Install mysql client on EC2 and connect to the RDS endpoint using DB credentials.<br>
    7.  Take screenshot of successful connection (the mysql&gt; prompt or a simple query result).</p>

<p>1) Create the RDS MySQL instance (console)<br>
    • AWS Console → RDS → Databases → Create database.<br>
    • Select Standard Create → Engine: MySQL.<br>
    • Edition: choose a free-tier compatible engine and version (e.g., MySQL 8.x) and db.t2.micro / db.t3.micro (Free Tier eligible).<br>
    • Templates: Free tier.<br>
    • DB instance identifier, username, and password — save these credentials.<br>
    • Networking:<br>
    • Select the VPC where your EC2 will be (or create matching one).<br>
    • Set Public accessibility = No (recommended). If you need direct public access for testing, set Yes — but prefer private + EC2 in same VPC.<br>
    • Choose a subnet group and availability zone (defaults fine).<br>
    • Connectivity &amp; security:<br>
    • Choose or create a Security Group that allows inbound MySQL (port 3306) from your EC2 security group (best) or your IP (for quick test).<br>
    • Create the DB. Wait until status becomes Available.<br>
    • Copy the Endpoint and Port (you’ll need these).</p>

<p>2) Create an EC2 instance (console)<br>
    • EC2 → Instances → Launch instance.<br>
    • AMI: Amazon Linux 2 or Ubuntu LTS (examples below include both).<br>
    • Instance type: t2.micro (free tier).<br>
    • Configure Network: pick the same VPC and a subnet that can route to the RDS subnet.<br>
    • IAM role: you can attach the IAM role here (create role first if required).<br>
    • Security group: allow SSH (port 22) FROM your IP and allow outbound to RDS (outbound default is usually all traffic).<br>
    • Launch and SSH into the instance.</p>

<p>3) Create IAM role with RDS access (console)<br>
    • IAM → Roles → Create role → AWS service → EC2 → Next.<br>
    • Attach policies:<br>
    • For general RDS management or IAM DB Auth: attach AmazonRDSFullAccess (only for learning) OR more restricted policies (prefer).<br>
    • If you plan to use IAM DB Authentication for MySQL, attach AmazonRDSFullAccess or ensure rds-db:connect permissions are present for the DB resource.<br>
    • Name the role (e.g., EC2-RDS-Access-Role) and create.<br>
    • Attach to EC2 instance: in EC2 console → Select instance → Actions → Security → Modify IAM role → attach role.</p>

<p>Note: If you do not use IAM DB auth, you still can attach role for other automation tasks; DB connection will rely on DB username/password.</p>

<p>4) Security groups (important)<br>
    • RDS security group inbound rule: allow MySQL/Aurora (TCP 3306) from the EC2 instance security group (use SG ID as source). This is more secure than permitting IP addresses.<br>
    • EC2 security group: Allow SSH from your IP (TCP 22). Outbound: allow destination port 3306 (default outbound allow all works).</p>

<p>5) Install mysql client on EC2 &amp; connect (commands)</p>

<p>Amazon Linux 2<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Update
sudo yum update -y

# Install MariaDB client (works as mysql client)
sudo yum install -y mariadb

# Or if you prefer the mysql client binary for MySQL:
# sudo yum install -y mysql

</code></pre>

</div>



<p>Ubuntu<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo apt update &amp;&amp; sudo apt upgrade -y
sudo apt install -y mysql-client
</code></pre>

</div>



<p>Connect to RDS (Remember to replace placeholders)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Example:
mysql -h your-rds-endpoint.rds.amazonaws.com -P 3306 -u your_db_user -p

# Then enter the DB password when prompted.

</code></pre>

</div>



<p>Once connected you should see the mysql&gt; prompt. Run a simple query:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SHOW DATABASES;
SELECT NOW();
</code></pre>

</div>



<p>6) If you want to use IAM DB Authentication (optional)<br>
    • Enable IAM DB auth when creating RDS (or modify parameter group).<br>
    • Create role with rds-db:connect and proper trust relationship for EC2.<br>
    • Use AWS CLI on EC2 to get an authentication token:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Example: generate token (install aws cli v2 and configure credentials on EC2)
aws rds generate-db-auth-token --hostname your-rds-endpoint.rds.amazonaws.com --port 3306 --region us-east-1 --username your_db_user
# Use the resulting token as the password with mysql client and --enable-cleartext-plugin if needed:
mysql --host=your-rds-endpoint.rds.amazonaws.com --port=3306 --user=your_db_user --enable-cleartext-plugin --password=TOKEN

</code></pre>

</div>



<p>(Using IAM DB auth has extra setup steps — ask if you want a full walkthrough.)</p>

<p>7) Troubleshooting checklist<br>
    • RDS status = Available.<br>
    • Use the correct endpoint and port.<br>
    • DB username &amp; password are correct (copy/paste carefully).<br>
    • EC2 and RDS are in same VPC or connected via routing (peered VPC requires routing &amp; SG updates).<br>
    • Security groups: RDS SG inbound permits EC2 SG on port 3306.<br>
    • If RDS is not public, EC2 must be in same VPC or have VPN/direct connect/bastion.<br>
    • If using public access, ensure RDS public endpoint and SG allow your IP (less secure).</p>

<p>8) Capturing the small-win screenshot<br>
    • SSH to EC2, run the mysql command and capture the terminal showing:<br>
    • mysql&gt; prompt OR<br>
    • Output of SHOW DATABASES; or SELECT NOW();<br>
    • Save screenshot(s) and post as proof. That’s your small win </p>

