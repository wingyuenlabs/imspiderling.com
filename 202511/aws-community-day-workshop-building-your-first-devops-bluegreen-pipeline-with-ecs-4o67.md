---
Title: AWS community day Workshop: Building Your First DevOps Blue/Green Pipeline with ECS
Description: 
Author: Jones Ndzenyuy
Date: 2025-11-21T21:49:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>In this workshop, I will guide you step by step how to build a Blue-Green deployment pipeline with AWS pipeline and deploys on AWS ECS with EC2. This deployment strategy helps teams to first test an app while routing traffic to a non-prod route, when confirmed that the app functions as expected, then the traffic is routed to prod. In case of any failure, a rollback is initiated for the previously working version hence avoiding downtime. </p>

<h2>
  
  
  Architecture
</h2>

<h3>
  
  
  App Flow
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F00ic54n0vvkm826uc5p2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F00ic54n0vvkm826uc5p2.png" alt="Application Archichitecture" width="800" height="626"></a></p>

<p>The Infrastructure consists of an Application Load Balancer which exposes the app on Port 80, routes traffic to the autoscaling group managed by ECS, a fleet of EC2 instances running in two AZs for High availability. Each of these instances have ECS agent installed and the necessary software for running our Docker containers. These interact with Aurora PostgreSQL database in RDS for data storage. Aurora PostgreSQL database is configured to replicate data in another availability zone for a resilient database configuration. When there is a spike in traffic, ECS will launch other containers in order to accommodate the load.</p>

<h3>
  
  
  Deployment Flow
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feye9iygsdput5ol4nz50.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feye9iygsdput5ol4nz50.png" alt=" " width="800" height="450"></a></p>

<p>Provided the developers do changes and want to update the deployed app,  our CI/CD pipeline is configured to automate the deployment. With code stored in GitLab, once there is a push, AWS CodePipeline is triggered via a connection app setup and authenticated to Gitlab, Codebuild builds, containerize and push the image to ECR, next Codedeploy deploys the new app, first will deploy it with port 8080 for tests, if the team is happy with their changes, they can approve for traffic to be switch to the newly deployed container and terminate the former. The former containers can be configured to still run for some time while the newly deployed is tested, if everything is fine, then the last version of the deployed app can be terminated.</p>

<h2>
  
  
  Build it step by step
</h2>

<p><code>**Note**: _If you face issues putting it up and working, go to the optional stage and deploy with Cloudformation_</code></p>

<p>Ensure you have a code editor like VScode, you have a Gitlab account, an AWS account will appropriate permissions(full access to RDS, ECS, ECR, IAM), configure awscli on you terminal with access keys  </p>

<h3>
  
  
  Clone the Code
</h3>

<p>On your local computer terminal(i use wsl with ubuntu os) run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://gitlab.com/ndzenyuy/tripmgmt.git
<span class="nb">cd </span>tripmgmt
<span class="nb">rm</span> <span class="nt">-rf</span> .git
git init
code <span class="nb">.</span>
</code></pre>

</div>



<p>It will clone the repository that has the application code to your local machine, enter the folder and remove the .git file so as to remove the url for the repository from whence the code was pulled, therefore making it possible to use the code and push to your own repository. </p>

<p>On the browser, go to GitLab and create an account if you don't have one, in it, create a repository named "tripmgmt", you can make the project public and don't create it with a READme.md file. Now copy the repository http url and go back to your terminal and run the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git remote add origin &lt;your repo url&gt;
git add <span class="nb">.</span>
git commit <span class="nt">-m</span> <span class="s2">"Initial commit"</span>
git push <span class="nt">--set-upstream</span> origin main
</code></pre>

</div>



<p>These will push the code to the newly created repository.</p>

<h2>
  
  
  Setup AWS Resources
</h2>

<h3>
  
  
  Create required AWS IAM Roles
</h3>

<ul>
<li>
<strong>Task Execution Role</strong>: Grants the Amazon ECS container and Fargate agents permission to make AWS API calls on your behalf.</li>
</ul>

<p>To create the <em>ecsTaskExecutionRole</em> IAM role  </p>

<ol>
<li>Open the IAM console </li>
<li>In the navigation pane, choose Roles, Create role.</li>
<li>In the Trusted entity type, select AWS Service</li>
<li>Under Use case, select Elastic Container Service and Elastic Container Service Task, then choose Next: Permissions.</li>
<li>In the Attach permissions policy section, search for <em>AmazonECSTaskExecutionRolePolicy</em>, select the policy, and then choose Next: Review.</li>
<li>For Role Name, type <em>ecsTaskExecutionRole</em> and choose Create role.</li>
<li>Please save value of the Role ARN in the open text file to use in the later section of the workshop.</li>
</ol>

<ul>
<li>
<strong>ECS Container Instance Role</strong>: This IAM role is required for the EC2 launch type. The Amazon ECS container agent makes calls to the Amazon ECS API on your behalf. Container instances that run the agent require an IAM policy and role for the service to know that the agent belongs to you.</li>
</ul>

<p>To create the <strong>ecsInstanceRole</strong> IAM role</p>

<ol>
<li>In the navigation pane, choose Roles, Create role.</li>
<li>In the Trusted entity type, select AWS Service</li>
<li>Under Use case, select EC2, then choose Next: Permissions.</li>
<li>In the Attach permissions policy section, search for <em>AmazonEC2ContainerServiceforEC2Role</em>, select the policy, and then choose Next: Review.</li>
<li>For Role Name, type <em>ecsInstanceRole</em> and choose Create role.</li>
</ol>

<ul>
<li>
<strong>ECS CodeDeploy Role</strong>: Before you can use the CodeDeploy blue/green deployment type with Amazon ECS, the CodeDeploy service needs permissions to update your Amazon ECS service on your behalf. These permissions are provided by the CodeDeploy IAM role.</li>
</ul>

<p>To create the <em>ecsCodeDeployRole</em> IAM role  </p>

<ol>
<li>In the navigation pane, choose Roles, Create role.</li>
<li>In the Trusted entity type, select AWS Service</li>
<li>Under Use case, select CodeDeploy and CodeDeploy - ECS, then choose Next: Permissions.</li>
<li>In the Attach permissions policy section, <em>AWSCodeDeployRoleForECS</em> policy should come as selected, choose Next: Review.</li>
<li>For Role Name, type <em>ecsCodeDeployRole</em> and choose Create role</li>
</ol>

<h3>
  
  
  Create Container Repository
</h3>

<ol>
<li>Open the Amazon ECR console.</li>
<li>In the navigation pane, under Private Registry choose Repositories.</li>
<li>On the Repositories page, choose Create repository.</li>
<li>For Repository name, enter a name for your repository like <em>devops/tripmgmtdemo</em>
</li>
<li>For Tag immutability, keep it Mutable for this workshop. Repositories configured with immutable tags will prevent image tags from being overwritten. For more information, see Image Tag Mutability.</li>
<li>For Encryption settings, select AWS Key Management Service (KMS). Use an AWS managed key. To read more about encrypting at rest, see ECR Encryption at Rest.</li>
<li>For Scan on push, keep it Disabled for this workshop. Repositories configured to scan on push will start an image scan whenever an image is pushed, otherwise image scans need to be started manually. For more information, see Image Scanning.</li>
<li>Choose Create repository.</li>
<li>Please save value of the Repository URL in the open text file to use in the later section of the workshop.</li>
</ol>

<h3>
  
  
  Setup Aurora PostgreSQL RDS Database
</h3>

<p>In this section, we will setup Aurora PostgreSQL RDS with Multi-AZ configuration.</p>

<ol>
<li>Go to the AWS Management Console and open the Amazon RDS console.</li>
<li>In the navigation pane, choose Databases.</li>
<li>Choose Create database.</li>
<li>In Choose a database creation method, choose Standard Create.</li>
<li>In Engine options, choose Aurora (PostgreSQL Compatible).</li>
<li>In Templates, choose Dev/Test template. 

<ol>
<li>Clear Auto generate a password check box.</li>
<li>Enter Master password value as <em>SuperSecretPGSqlPwd##2006</em> and enter the same password in Confirm password.</li>
</ol>


</li>

<li>In the DB cluster identifier field, give Aurora DB cluster name, <em>tripmgmtdb-cluster</em>
</li>

<li>To enter your master password, do the following in Credential Settings</li>

<li>For Instance configuration, choose <em>db.t4g.medium</em> under <em>Burstable classes (includes t classes)</em>
</li>

<li>For Availability &amp; durability, choose Create an Aurora Replica/Reader node in a different AZ (recommended for scaled availability)</li>

<li>Keep defaults for <strong>Connectivity, Babelfish settings, Database authentication, Monitoring sections</strong>.</li>

<li>For Additional configuration, enter <em>tripmgmt</em> under Initial database name.</li>

<li>Choose Create database.</li>

<li>For Databases, choose the name of the new Aurora DB cluster.
On the RDS console, the details for new DB cluster appear. The DB cluster and its DB instance have a status of creating until the DB cluster is ready to use. When the state changes to available for both, you can connect to the DB cluster. Depending on the DB instance class and the amount of storage, it can take up to 20 minutes before the new DB cluster is available.</li>

<li>On the Connectivity &amp; security tab, note the port and the endpoint of the writer DB instance. Please save value of the endpoint and port of the cluster in the open text file to use in the later section of the workshop. Endpoint URL should be of format tripmgmtdb-cluster.cluster-UNIQUEID.AWSREGION.rds.amazonaws.com</li>

<li>Click on Writer Node and on the Connectivity &amp; security tab, please save value of the Security Group Id under Security section (i.e. sg-xxxxxxxx) in the open text file to use in the later section of the workshop.</li>

</ol>

<ul>
<li><strong>Create Application Load Balancer and Target Groups</strong></li>
</ul>

<p>We will create an Amazon EC2 application load balancer. This will be the public endpoint to access Trip Management Monolith Application. The load balancer must use a VPC with two public subnets in different Availability Zones. <br>
First create Target groups for your load balancer</p>

<ol>
<li>Sign in to the AWS Management Console and open the Amazon EC2 console </li>
<li>In the navigation pane, choose Target groups and select Create target group.</li>
<li>Under Basic configuration, select IP addresses target type.</li>
<li>In Name, enter a target group name <em>alb-tg-tripmgmtdemo-1</em>
</li>
<li>In Protocol choose <em>HTTP</em>. In Port, enter <em>80</em>.</li>
<li>Keep rest of the settings as defaults.</li>
<li>Select Next: Register Targets.</li>
<li>Select Create target group.</li>
<li>Repeat above steps to create another target group for Protocol <em>HTTP</em> and Port <em>8080</em>, name it <em>alb-tg-tripmgmtdemo-2</em>
</li>
</ol>

<p><strong>To create an Amazon EC2 application load balancer</strong></p>

<ol>
<li>Sign in to the AWS Management Console and open the Amazon EC2 console </li>
<li>In the navigation pane, choose Load Balancers, choose Create Load Balancer.</li>
<li>Choose Application Load Balancer, and then choose Create.</li>
<li>In Name, enter the name of your load balancer <em>tripmgmtdemo-alb</em>
</li>
<li>In Scheme, choose internet-facing.</li>
<li>In IP address type, choose <em>ipv4</em>.</li>
<li>Under Network mapping, in VPC, choose the default VPC, and then choose any two availability zones under Mappings. Please save value of the Availability Zone / Subnet ids in the open text file to use in the later section of the workshop.</li>
<li>Under Security groups, choose Create new security group

<ol>
<li>Give it the name <em>ALBSecurityGroup</em> in the Security group name field.</li>
<li>Give it the description  in the Description field.</li>
<li>Add Inbound Rule, Allow 80 port (HTTP) inbound traffic from My IP</li>
<li>Add Inbound Rule, Allow 8080 port (CustomTCPPort) inbound traffic from My IP</li>
<li>Select Create security group</li>
<li>Please save value of the newly created Security Group Id (i.e. sg-xxxxxxxx..) in the open text file to use in the later section of the workshop.</li>
<li>Come back to Create Application Load Balancer tab, refresh Security groups and select newly created Security group, removing any existing pre-selected Security groups.</li>
</ol>
</li>
<li>Under Listeners and routing, configure two listener ports for your load balancer:

<ol>
<li>Under Protocol, choose HTTP, and Under Port, enter 80.</li>
<li>Under Default action, select alb-tg-tripmgmtdemo-1 Target Group.</li>
<li>Choose Add listener.</li>
<li>Under Protocol, choose HTTP, and Under Port, enter 8080.</li>
<li>Under Default action, select alb-tg-tripmgmtdemo-2 Target Group.</li>
</ol>
</li>
<li>Choose Create load balancer.</li>
<li>Go to Load Balancers and click on the newly created load balancer. From the Description tab, please save value of the DNS name of the Load Balancer in the open text file to use in the later section of the workshop.</li>
</ol>
<h2>
  
  
  Setup ECS Cluster
</h2>
<h3>
  
  
  Create ECS Cluster
</h3>

<p>Amazon ECS creates an Amazon EC2 Auto Scaling launch template and Auto Scaling group on your behalf as part of the AWS CloudFormation stack.</p>

<ol>
<li>Open the Amazon ECS console.</li>
<li>In the navigation pane, choose <em>Clusters</em>.</li>
<li>On the Clusters page, choose <em>Create Cluster</em>.</li>
<li>Under Cluster configuration, for Cluster name, enter <em>ecs-cluster-tripmgmtdemo</em>
</li>
<li>Under Infrastructure, keep <em>AWS Fargate (serverless)</em> selected and also select <em>Amazon EC2 instances</em>. Next, configure the Auto Scaling group which acts as the capacity provider.</li>
</ol>

<p>Select Create new group, and then provide the following details about the group:</p>

<ul>
<li>For Operating system/Architecture, choose Amazon Linux 2.</li>
<li>For EC2 instance type, choose t3.large. Here, t3.large selected as required for this workshop to complete in given time.</li>
<li><p>For Capacity, enter 2 for the minimum number and 4 for the maximum number of instances to launch in the Auto Scaling group.</p></li>
<li><p>Under Network settings for Amazon EC2 instances, choose default VPC and same two subnets, which were selected at that time of creating Application Load Balancer in previous section.</p></li>
<li><p>Choose Create</p></li>
</ul>

<p><strong>Create Security Group for EC2 Container Instance</strong></p>

<ol>
<li>Open the Amazon EC2 console.</li>
<li>On the navigation pane, under Network &amp; Security, choose Security Groups</li>
<li>On the next page, choose Create security group.</li>
<li>For Security group name, enter ECS-ALB-SecurityGroup for the security group name and give meaningful description under Description.</li>
<li>Choose default VPC</li>
<li>Add Inbound Rule, Allow 80 port (HTTP) inbound traffic from Application Loadbalancer Security Group you have created in earlier section (i.e. sg-xxxxx).</li>
<li>Add Inbound Rule, Allow 8080 port (CustomTCPPort) inbound traffic from Application Loadbalancer Security Group you have created in earlier section (i.e. sg-xxxxx).</li>
<li>Select Create security group</li>
<li>Please save value of the newly created Security Group Id (i.e. sg-xxxxxxxx..) in the open text file to use in the later section of the workshop.</li>
</ol>
<h3>
  
  
  Create ECS Task
</h3>

<ol>
<li>
<p>First create Amazon CloudWatch Log Group, where taskdef is configured to send all logs.</p>

<ol>
<li>Go to CloudWatch Console </li>
<li>Select Log Groups,</li>
<li>Click Create log group button,</li>
<li>Log Group Name: enter <em>tripmgmt-demo-ecstask-loggrp</em> and Retention setting select 5 days</li>
<li>Choose Create</li>
</ol>
</li>
<li><p>In order to create an ECS Task, we will need to create a ECS Task Definition file. Go to you cli, make sure it is pointing to the folder <em>tripmgmt</em> and delete the file <em>taskdef.json</em> by running the command<br>
</p></li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>rm taskdef.json
</code></pre>

</div>


<ol>
<li>Copy below given environment variables in plain text-file and change respective environment variable values as per the workshop resources created by you in the previous sections. After modifying values of respective environment variables, execute these export commands on the commandline.
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">LOG_GROUP</span><span class="o">=</span>tripmgmt-demo-ecstask-loggrp
<span class="nb">export </span><span class="nv">DB_USERNAME</span><span class="o">=</span>postgres
<span class="nb">export </span><span class="nv">DB_PASSWORD</span><span class="o">=</span>SuperSecretPGSqlPwd##2006
<span class="nb">export </span><span class="nv">AWS_DEFAULT_REGION</span><span class="o">=&lt;&lt;</span><span class="no">AWS</span><span class="sh">-REGION&gt;&gt;

export TASK_EXECUTION_ROLE_ARN="arn:aws:iam::&lt;&lt;AccountID&gt;&gt;:role/ecsTaskExecutionRole"
export AURORA_PGSQL_RDS_URL="tripmgmtdb-cluster.cluster-&lt;&lt;UNIQUEID&gt;&gt;.&lt;&lt;AWS-REGION&gt;&gt;.rds.amazonaws.com"
export ECR_LATEST_IMAGE_URL="&lt;&lt;AccountID&gt;&gt;.dkr.ecr.&lt;&lt;AWS-REGION&gt;&gt;.amazonaws.com/devops/tripmgmtdemo:latest"
</span></code></pre>

</div>


<ul>
<li>
<code>LOG_GROUP</code> : Verify name as you have created in Step No 1 in this section, only change if you have used different log group name.</li>
<li>
<code>DB_USERNAME</code>: Database username set while creating Aurora PostgreSQL DB Cluster.</li>
<li>
<code>DB_PASSWORD</code>: Database password set while creating Aurora PostgreSQL DB Cluster.</li>
<li>
<code>TASK_EXECUTION_ROLE_ARN</code>: Task Execution Role created in Create IAM Roles section.</li>
<li>
<code>AURORA_PGSQL_RDS_URL</code>: Aurora PostgreSQL RDS Cluster URL as created in Create Aurora PostgreSQL DB section.</li>
<li> <code>ECR_LATEST_IMAGE_URL</code>: Elastic Container Registry Image URL from Create Container Repository section.</li>
<li>Create the task definition from the command line by running
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>envsubst &lt; "taskdef_src.json" &gt; "taskdef.json"
aws ecs register-task-definition --cli-input-json file://taskdef.json
</code></pre>

</div>


<ol>
<li><p>Please save the value of the TaskDefinition ARN from output in the open text file to use in the later section of the workshop. It is at the very top of the output and should look something like arn:aws:ecs:&lt;&gt;:&lt;&gt;:task-definition/task-tripmgmt:1.</p></li>
<li><p>Commit taskdef.json in repository.<br>
</p></li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git add taskdef.json
git commit -m "Updated Taskdef"
</code></pre>

</div>


<ol>
<li>You can also verify the Task Definition in the Amazon ECS console , left navigation bar under Amazon ECS, Task Definitions.</li>
</ol>

<p><strong>Prepare AppSpec File</strong><br>
AWS CodeDeploy requires AppSpec YAML-formatted file to manage deployment. In the project folder on the terminal, run the code below and edit the value of the taskdefinition arn to reflect what you had in step 4 above(similar to: arn:aws:ecs:&lt;&gt;:&lt;&gt;:task-definition/task-tripmgmt:1.)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>nano appspec.yaml
</code></pre>

</div>



<ol>
<li>Commit appspec.yaml in repository
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git add appspec.yaml
git commit -m "appspec file with Taskdef url"
git push
</code></pre>

</div>



<h2>
  
  
  Optional: Deploy the infrastructure with Cloudformation (Recommended)
</h2>

<ol>
<li>Login to the console and search for cloudformation</li>
<li>Create a new stack</li>
<li>Prerequisite - Prepare template: Choose existing stack</li>
<li>Specify template: Upload a template file</li>
<li>Upload a template file: Choose file -&gt; search on your computer, in the tripmgmt/cloudformation-infra folder/subfolder and choose tripmgmt.yml</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F44cy9ddsubuf4rjqcaw4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F44cy9ddsubuf4rjqcaw4.png" alt=" " width="800" height="421"></a></p>

<p>On specify Stack details<br>
<code><br>
Stack name: tripmgmt<br>
KeyPairName: &lt;select a keypair in you account or create on if absent&gt;<br>
PublicSubnet1Id: &lt;select a public subnet&gt;<br>
PublicSubnet2Id: &lt;select a public subnet&gt;<br>
VpcId: &lt;select the default vpc id&gt;<br>
click next<br>
</code><br>
Scroll down and acknowledge, </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw0xw2ij4jauqfeynhky4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw0xw2ij4jauqfeynhky4.png" alt=" " width="800" height="425"></a></p>

<p>and click next scroll and click submit</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyufpcenfheyzswjc2lrx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyufpcenfheyzswjc2lrx.png" alt=" " width="800" height="432"></a></p>

<p>This will create the infrastructure needed to run our Pipeline.</p>

<h2>
  
  
  DevOps Pipeline with Blue/Green Deployment
</h2>

<p>The Engineering team commit changes to code base and then push those changes to a git repository. It triggers AWS CodePipeline action. AWS CodeBuild will compile source code and build &amp; store container image in an Elastic Container Registry. AWS CodeDeploy will initiate Blue/Green deployment of ECS Task(s) in an ECS Cluster through ECS Service.</p>

<h3>
  
  
  Setup CodeBuild with GitLab Integration
</h3>

<p>To setup AWS CodeBuild, first we need to create/modify buildspec.yml file to describe build steps. After that, we need to configure the CodeBuild Project linking with the GitLab Repository.<br>
On the cli, run the code below and edit the values of the following parameters to suite those you have:</p>

<ul>
<li><code>AWS_ACCOUNT_ID</code></li>
<li><code>AWS_DEFAULT_REGION</code></li>
<li>
<code>REPOSITORY_URI</code>
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>nano buildspec.yml
</code></pre>

</div>



<p>After editing, run <code>ctrl + x</code>, when prompted to save changes press <code>Y</code> then press enter to confirm.</p>

<p><strong>Setup CodeBuild Project</strong></p>

<ol>
<li>Open the CodeBuild console.</li>
<li>On the Build projects page, choose Create build project.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5wt4ghqwv0xh1k7c5278.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5wt4ghqwv0xh1k7c5278.png" alt=" " width="800" height="155"></a></p>

<ol>
<li>In Project configuration: Enter a name for this build project <em>tripmgmt-demo-build</em>. Build project names must be unique across each AWS account. You can also include an optional description of the build project to help other users understand what this project is used for.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpef83dgmvnqug8fygt7r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpef83dgmvnqug8fygt7r.png" alt=" " width="800" height="497"></a></p>

<ol>
<li>If required, Select Build badge to make your project's build status visible and embeddable.</li>
<li>In Source: For Source provider, choose GitLab.</li>
<li>You will then get an error stating that you have not connected to GitLab.</li>
<li>Press the Manage account credentials link.</li>
<li>Select GitLab as your credential type.</li>
<li>In Connection, press create a new GitLab connection.</li>
<li>Give the connection the name <em>tripmgmt-connection</em>.</li>
<li>Press the Connect to GitLab button and authenticate.</li>
<li>Authorize AWS Connector for GitLab.</li>
<li>After being redirected back to the AWS Console, press Connect.</li>
<li>You should now be able to save and add your new GitLab connection.</li>
<li>From Repository, choose the repository you have created for this project demo.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4w9xw1nplc5h9e3gr4on.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4w9xw1nplc5h9e3gr4on.png" alt=" " width="800" height="575"></a></p>

<ol>
<li>For Source version, type main.</li>
<li>In Environment: choose Managed Image with Operating system as Ubuntu,</li>
<li>For Runtime(s) choose Standard and Image choose the one with Standard:5, having Image Version as Always use the latest image for this runtime version</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgzrxwe5ae6mro3d4eno6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgzrxwe5ae6mro3d4eno6.png" alt=" " width="800" height="659"></a></p>

<ol>
<li>For Service role choose New service role and give it a meaningful name <em>codebuild-tripmgmt-demo-build-service-role</em>
</li>
<li>Open Aditional configuration in the same section. Check mark Privileged action.</li>
<li>For Buildspec, select that you want to Use a buildspec file, add the buildspec.yml file location and name - <em>buildspec.yml</em>
</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpsfdx20tchso6gy3eq6c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpsfdx20tchso6gy3eq6c.png" alt=" " width="800" height="476"></a></p>

<ol>
<li>For Logs, check mark CloudWatch logs and give Group name as <em>codebuild-logs</em>, Stream name as <em>tripmgmtdemo-build</em>.</li>
<li>Click Create build project.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6ncspkv38basuoblg06.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6ncspkv38basuoblg06.png" alt=" " width="800" height="521"></a></p>

<ol>
<li>
<p>Now edit newly created service role <em>codebuild-tripmgmt-demo-build-service-role</em> to allow accessing ECR repository through <em>AmazonEC2ContainerRegistryPowerUser</em> managed policy.</p>

<ol>
<li>Open codebuild-tripmgmt-demo-build-service-role role from IAM console .</li>
<li>Choose Attach policies.</li>
<li>To narrow the available policies to attach, for Filter, type AmazonEC2ContainerRegistryPowerUser</li>
<li>Check the box to the left of the AWS managed policy and choose Attach policy and Update.</li>
</ol>


</li>

<li><p>Select the build project and Start build to test GitLab and CodeBuild integration.</p></li>

<li><p>Post successful build, you can verify new docker image in Amazon ECR console.</p></li>

</ol>

<h3>
  
  
  Deploy on Amazon ECS with EC2
</h3>

<p>As per the architectural diagram, Engineering team commit changes to code base and then push those changes to a Git repository. It triggers AWS CodePipeline action, AWS CodeBuild will compile source code and build &amp; store container image in an Elastic Container Registry. AWS CodeDeploy will initiate Blue/Green deployment of ECS Task(s) in an ECS Cluster through ECS Service. Upon successful deployment of ECS Task(s), users can access Web Portal through DNS URL pointing to ALB endpoint. ALB endpoint serves client requests from ECS Cluster after automatically adjusting capacity to maintain steady state.</p>

<p><strong>Create CodeDeploy Application</strong></p>

<p>To create a CodeDeploy application</p>

<ol>
<li>Open the CodeDeploy console  and choose Create application.</li>
<li>In Application name, enter the name you want to use <em>Deploy-Tripmgmt-Demo-App</em>
</li>
<li>In Compute platform, choose Amazon ECS.</li>
<li>Choose Create application.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx06my0prcus5l8makaj2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx06my0prcus5l8makaj2.png" alt=" " width="800" height="546"></a></p>

<p><em><strong>To create a CodeDeploy deployment group</strong></em></p>

<ol>
<li>On your application page's Deployment groups tab, choose Create deployment group.</li>
<li>In Deployment group name, enter a name that describes the deployment group <em>deploygrp-tripmgmt-demo</em>
</li>
<li>In Service role, choose a service role ecsCodeDeployRole, which was created in earlier section Create required AWS IAM Roles and grants CodeDeploy access to Amazon ECS.</li>
<li>In Environment configuration, choose your Amazon ECS cluster name and service name.</li>
<li>From Load balancers, choose the name of the load balancer that serves traffic to your Amazon ECS service.</li>
<li>From Production listener port, choose the port and protocol for the listener that serves production traffic to your Amazon ECS service. (Production listener port : 80 and Test listener port : 8080)</li>
<li>From Target group 1 name and Target group 2 name, choose the target groups used to route traffic during your deployment. Make sure that these are the target groups you created for your load balancer.</li>
<li>
<p>Under Deployment settings,</p>

<ul>
<li>Choose Specify when to reroute traffic and choose 0 days, 0 hours, and 5 minutes. This will reroute the traffic to successfully deployed updated tasks after 5 Minutes. This is the time required by you to verify the successfully deployed updated tasks and decide whether to rollback or continue reroute the traffic.</li>
<li>Under Deployment configuration, choose CodeDeployDefault:ECSAllAtOnce. It will configure CodeDeploy to shift all traffic to the updated Amazon ECS container at once.</li>
<li>Under Original revision termination, and choose 0 days, 0 hours, and 0 minutes. It will configure CodeDeploy to terminate the original tasks immediately after rerouting the traffic. You can choose to keep the original tasks for a desired duration as required. After termination of original tasks, you cannot rollback manually or automatically.</li>
</ul>
</li>
<li><p>Choose Create deployment group.</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdxyzd94oyhcx0s9lsb7j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdxyzd94oyhcx0s9lsb7j.png" alt=" " width="800" height="327"></a></p>

<h3>
  
  
  Create CodePipeline
</h3>

<p>In this section, we will create a DevOps pipeline with the following actions:</p>

<ul>
<li>A source stage with a GitLab action</li>
<li>A build stage with a CodeBuild action</li>
<li>A deployment stage with an Amazon ECS deploy action with Blue/Green Deployment.</li>
</ul>

<ol>
<li>Sign in to the AWS Management Console and open the CodePipeline console </li>
<li>On the Welcome page, Getting started page, or the Pipelines page, choose Create pipeline.</li>
<li>In Step 1: Choose creation settings, pick Build custom pipeline</li>
<li>In Step 2: Choose pipeline settings, in Pipeline name, enter <em>codepipeline-tripmgmt-demo</em>.</li>
<li>In Service role, Choose New service role to allow CodePipeline to create a new service role in IAM. Enter name as codepipeline-tripmgmt-demo-service-role</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7r3s5wyfkpmdk1ol25ck.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7r3s5wyfkpmdk1ol25ck.png" alt=" " width="800" height="605"></a></p>

<ol>
<li>In Artifact store, Choose Default location to use the default artifact store, such as the Amazon S3 artifact bucket designated as the default, for your pipeline in the region you have selected for your pipeline.</li>
<li>Choose Next.</li>
<li>In Step 2: Add source stage, in Source provider, choose GitLab. In Connection select the connection you created earlier. Following this, select the repository and default branch you chose earlier. The rest can be kept at default.</li>
<li>Choose Next.</li>
<li>In Step 3: Add build stage, in Build provider, choose Other build providers and then AWS CodeBuild. In Project name choose the name of the build project. The rest can be kept default.</li>
<li>Choose Next.</li>
<li>Choose Skip test stage at the bottom of the page.</li>
<li>In Step 4: Add deploy stage, in Deploy provider choose Amazon ECS (Blue/Green)</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fww37yzij4yncd7buo43u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fww37yzij4yncd7buo43u.png" alt=" " width="800" height="720"></a></p>

<ol>
<li>In AWS CodeDeploy application name, choose CodeDeploy Application name. In AWS CodeDeploy deployment group, choose CodeDeploy Application's Deployment Group.</li>
<li>In Amazon ECS task definition, choose BuildArtifact and enter taskdef.json.</li>
<li>In AWS CodeDeploy AppSpec file, choose BuildArtifact and enter appspec.yaml.</li>
<li>Choose Next.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nqwnam0m5jr7uc2y8lm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nqwnam0m5jr7uc2y8lm.png" alt=" " width="800" height="671"></a></p>

<ol>
<li>
<p>Review the information, and then choose Create pipeline. Immediately, stop the Pipeline Execution with Stop and Abandon pop-up action.<br>
Modify Deploy Stage</p>

<ol>
<li>Open (or remain in) the CodePipeline console  &amp; select the Pipeline name, for example codepipeline-tripmgmt-demo.</li>
<li>Click Edit, and scroll down at bottom and "Edit Stage" , click on Edit icon of Deploy Action Group.</li>
<li>In Input artifacts, choose SourceArtifact and remove BuildArtifact</li>
<li>Modify Amazon ECS task definition, choose SourceArtifact and keep taskdef.json.</li>
<li>Modify AWS CodeDeploy AppSpec file, choose SourceArtifact and keep appspec.yaml.</li>
<li>Choose Done</li>
<li>Scroll up and press Save.</li>
</ol>


</li>

</ol>

<h2>
  
  
  DevOps Pipeline in Action
</h2>

<ol>
<li>Go to Trip Management Monolith Application code base, preferrably home page and make a visible modification which you can identify after releasing a change. Look at the below screenshot, having (Tripment - New Change) at the end at line#7. File path: src/main/webapp/app/home/home.component.html</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkr3s8t7ftqtpclah41zd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkr3s8t7ftqtpclah41zd.png" alt=" " width="800" height="422"></a></p>

<ol>
<li>Commit applied changes and push changes to the GitLab repository.
This will trigger the pipeline:
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0klfduiogqrjokrbncou.png" alt=" " width="800" height="220">
</li>
</ol>

<p>On the browser where you pasted the DNS of the load balancer, check the app that is deployed, it should still be the one without the changes</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff3hpy5zqdag3a81w18iv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff3hpy5zqdag3a81w18iv.png" alt=" " width="800" height="217"></a></p>

<p>After the deployment is complete, before traffic is rerouted, check the browser with the 8080 port, you should see the web page with the changes:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkg6tz86vnmb1qdh9he4j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkg6tz86vnmb1qdh9he4j.png" alt=" " width="800" height="372"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3mjyheevihss93hn257y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3mjyheevihss93hn257y.png" alt=" " width="800" height="401"></a></p>

<p>Code deploy still deploying the new container on ECS, traffic is still flowing to the initial deployment. Once the deployment stage is complete, it will forward test traffic to the same alb link but on a different port:8080</p>

<p>The pipeline will wait 5 minutes for the newly deployed app to be tested(as configured in code pipeline). Once the time elapses, the pipeline will automatically route traffic</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqvszw9klkvdkb5egno88.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqvszw9klkvdkb5egno88.png" alt=" " width="800" height="137"></a></p>

<p>After tests, the engineer can now reroute traffic by clicking on reroute traffic, after they are sure their changes are as expected: </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzlu8j2p59xf0q60i4f25.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzlu8j2p59xf0q60i4f25.png" alt=" " width="800" height="336"></a></p>

<p>The traffic will instantly be rerouted to the new tasks, and will keep the former tasks should incase there is need for rollback after live traffic is routed to it, now both links for test and prod will display the same app, while waiting for a termination of the former: <br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F084nnzbksj3ic5lnty84.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F084nnzbksj3ic5lnty84.png" alt=" " width="800" height="427"></a></p>

<p>Traffic activity: <br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fby3elazw7qvptl39scuc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fby3elazw7qvptl39scuc.png" alt=" " width="800" height="330"></a></p>

<p>During the deployment phase, out of expected 2 task, ECS runs 4 tasks, this will ensure that the newly deployed app is stable before the termination of these can be authorized. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6398isnqwjwv606t4i23.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6398isnqwjwv606t4i23.png" alt=" " width="800" height="319"></a></p>

<h2>
  
  
  Clean-up
</h2>

<p>On the console, go to Cloudformation stacks and delete tripmgmt stack, or if you created the resources on the console, then delete them in the order they were created. Go to Developer tools and delete the pipeline, in codebuild, delete the build job.</p>

