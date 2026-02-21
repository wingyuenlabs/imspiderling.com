---
Title: How to Deploy a Dynamic Web Application on AWS (Beginner-Friendly Guide)
Description: 
Author: Oluwatobiloba Oludare
Date: 2026-02-21T21:39:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>Deploying a dynamic web application on AWS can feel overwhelming at first especially with so many services involved. In this guide, I’ll walk through a simple, practical approach to deploying a dynamic web app using core AWS services.</p>

<p>This is perfect for beginners learning cloud computing and DevOps.</p>

<p><strong>What is a Dynamic Web Application?</strong></p>

<p>Unlike static websites, dynamic web applications:</p>

<ul>
<li>Process user input</li>
<li>Connect to databases</li>
<li>Generate content in real time</li>
</ul>

<p><strong>Examples include:</strong> Login systems, Dashboards, E-commerce sites</p>

<p>Web apps built with PHP</p>

<p>AWS Services We’ll Use:</p>

<p>To deploy a dynamic web app, we typically need:<br>
🔹 Compute: EC2 – Hosts the application server<br>
🔹 Database: RDS – Stores application data<br>
🔹 Networking &amp; Security : VPC – Network environment and Security Groups <br>
🔹 Storage: S3 <br>
🔹 Route 53<br>
🔹 Application Load Balancer<br>
🔹 Auto-scaling group</p>

<p>🪜 <strong>Step-by-Step Deployment</strong></p>

<p><strong>1. Select appropriate Region.</strong></p>

<p><strong>2. Create VPC and enable DNS hostname.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzfhwehsm7z7u16h1hmeq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzfhwehsm7z7u16h1hmeq.png" alt=" " width="800" height="162"></a></p>

<p><strong>3.  Create all these Security Groups:</strong></p>

<p>-For ec2 instance connect endpoint (EICE) - No inbound rules- use ssh and limit it to vpc CIDR</p>

<p>-For application load balancer (alb-Sg) - select vpc - allow http &amp; https for anywhere </p>

<p>-For Webserver security group (web-sg)- select vpc - allow http &amp; https, limit to alb-sg - allow ssh, limit to eice.</p>

<p>-Security group to database (db-sg) -- select vpc - Allow MySQL, limit it to webserver-sg - again allow MySQL, limit it dm-sg. </p>

<p>-Security group for data migration (dms) - select vpc- allow ssh, limit it to eice.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flr1rgwy9age67893ufow.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flr1rgwy9age67893ufow.png" alt=" " width="800" height="197"></a></p>

<p><strong>4. Create instance connect endpoint</strong> in private subnet - select ec2 instance connect endpoint - select vpc - Use eice-sg - select private subnet -private-app-az2 </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3276zhrxkala1shcej02.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3276zhrxkala1shcej02.png" alt=" " width="800" height="276"></a></p>

<p><strong>5. Create S3 bucket and upload the application code accordingly.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz07evzgla8ekaf0j1len.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz07evzgla8ekaf0j1len.png" alt=" " width="800" height="234"></a></p>

<p><strong>6. Create IAM Policy</strong></p>

<ul>
<li>Select S3 - grant S3: GetObject &amp; S3: ListBucket.  -

<ul>
<li>under Resources - you can limit it using the ARN (but for practice select all)
-- Click on add  more permission 
-- select Secret Manager --  grant permission - GetSecretValue &amp; DescribeSecret.</li>
</ul>


</li>

</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmadexfopukbqzrf8dayn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmadexfopukbqzrf8dayn.png" alt=" " width="800" height="285"></a></p>

<p><strong>7. Create Role</strong>- select Ec2 as use case - select the policy you just created -- name your role.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Forji6sqlaod0txpphgga.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Forji6sqlaod0txpphgga.png" alt=" " width="800" height="298"></a></p>

<p><strong>8. Create Subnet group</strong> -- select vpc - select your two AZ  -- select your two private Subnets.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fehchslvqmhsu3no7qw4p.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fehchslvqmhsu3no7qw4p.png" alt=" " width="800" height="145"></a></p>

<p><strong>9.  Create Database RDS</strong>-</p>

<ul>
<li>Select Standard create </li>
<li>Select MySQL</li>
<li>Select the latest mysql database engine version
</li>
<li>Select the free tier</li>
<li>Name your DB instance Identifier </li>
<li>Select managed in AWS Secrets Manager
</li>
<li>Select include previous generation classes </li>
<li>Select instance type -- Select your Subnet group
</li>
<li>Use db-sg as the security group
</li>
<li>Select your AZ </li>
<li>Under configuration give the initial database name.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F34md9klwxn71w3d2ivde.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F34md9klwxn71w3d2ivde.png" alt=" " width="800" height="346"></a></p>

<p>Now, Migrating data into RDS database:</p>

<p><strong>10.  Create Ec2 instance</strong> -- select without key pair -- select private subnet private app az2 -- select dms-sg  - attach your role under instance profile. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1zsjcfwu00e55h88i9ph.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1zsjcfwu00e55h88i9ph.png" alt=" " width="800" height="464"></a></p>

<ol>
<li>Connect to the ec2 instance using the management console and use the migration script accordingly. Check out for the scripts to migrate data to RDS database here:  <a href="https://github.com/Oluwatobiloba-Oludare/Dynamic-Web-Application-on-AWS" rel="noopener noreferrer">https://github.com/Oluwatobiloba-Oludare/Dynamic-Web-Application-on-AWS</a>)</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl2x5mebtbji0sl1j5v87.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl2x5mebtbji0sl1j5v87.png" alt=" " width="800" height="337"></a></p>

<ol>
<li> After successfully migrating the data, you can terminate the Ec2 instance. </li>
</ol>

<p><strong>13.  Create another ec2 instance in private subnet for webserver</strong> -- select without a keypair -- use web-sg  -- attach the S3 role under instance profile. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvgerwuns3fsam9s7zy7k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvgerwuns3fsam9s7zy7k.png" alt=" " width="800" height="439"></a></p>

<p><strong>14.  Use the Deployment Script accordingly.</strong> (Check out the script in this github:  <a href="https://github.com/Oluwatobiloba-Oludare/Dynamic-Web-Application-on-AWS" rel="noopener noreferrer">https://github.com/Oluwatobiloba-Oludare/Dynamic-Web-Application-on-AWS</a> )</p>

<p><strong>15.  Create Target group</strong> -- select instance - select VPC -- Under Advance Health check, enter 200,301,302 -- select the instance and include and create.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyns80n24uavn95fyyjs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyns80n24uavn95fyyjs.png" alt=" " width="800" height="260"></a></p>

<p><strong>16.  Create Application Load balancer</strong></p>

<ul>
<li>select vpc - select the two AZ in your VPC -- select public subnet for the two AZ -- Use ALB-sg -- Under Listener, select  Redirect to URL and select full URL -</li>
<li>Add another listener -- select https -- select target group -- under default SSL, select your certificate.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhpo65yclvb2kbq3nn3m8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhpo65yclvb2kbq3nn3m8.png" alt=" " width="800" height="387"></a></p>

<p><strong>17.  Create a Record in our Domain name-</strong>   use www -- Alias -- select region -- select Application and classic load balancer </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fajvbotrq8dw2umhjfjvz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fajvbotrq8dw2umhjfjvz.png" alt=" " width="800" height="353"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk3hvry2amsypce2lamjm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk3hvry2amsypce2lamjm.png" alt=" " width="800" height="362"></a></p>

<p>Now, use your domain name to check your website on the internet. </p>

<p><strong>Congratulations!</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1s1coxntxgx9on1z8bz5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1s1coxntxgx9on1z8bz5.png" alt=" " width="800" height="418"></a></p>

<p><strong>Now, let's be prepare for sudden traffic by setting up our auto-scaling group.</strong> </p>

<p><strong>18. Create AMI</strong></p>

<ul>
<li>Go to the ec2 instance running and select actions and select image and templates and select create image 
-- select Tag image and snapshots together 
-- under Key, type Name and input your AMI name.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2v7l2hdxsyyr1jpwa99j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2v7l2hdxsyyr1jpwa99j.png" alt=" " width="800" height="248"></a></p>

<p><strong>19. Create Launch template</strong><br>
-- select Auto scaling guidelines <br>
-- select My AMIs and select owned by me <br>
-- select t2 micro -- select web sg.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nkcxrz9b9ybmra59ldq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nkcxrz9b9ybmra59ldq.png" alt=" " width="800" height="433"></a></p>

<p>You can terminate the ec2 instance. </p>

<p><strong>20. Create Auto scaling group</strong></p>

<ul>
<li>Select vpc - select the two private app </li>
<li>Select attach to an existing load balancer </li>
<li>Select target group </li>
<li>Select the two Health checks </li>
<li>Select the number of scaling you want </li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmprjhshp3i97cpfychd4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmprjhshp3i97cpfychd4.png" alt=" " width="800" height="244"></a></p>

<p>✍️ <strong>Final Thoughts</strong></p>

<p>Learning AWS becomes easier when you build real projects. Deploying a dynamic web application is one of the best beginner exercises to understand how cloud services work together.</p>

<p>If you are learning AWS, I highly recommend trying this hands-on project.</p>

<p>Follow along for more practical AWS, Docker, and cloud learning posts.</p>

