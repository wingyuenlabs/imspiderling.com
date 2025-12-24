---
Title: A Step-by-Step Guide to AWS Elastic Beanstalk :Simplifying Application Deployment
Description: 
Author:  Ganiyat Olagoke Adebayo
Date: 2025-12-24T21:02:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>Deploying applications in the cloud can sometimes feel overwhelming, especially when you have to manage servers, scaling, monitoring, and security on your own. This is where <strong>AWS Elastic Beanstalk</strong> comes in.</p>

<h2>
  
  
  <strong>AWS Elastic Beanstalk</strong> is a service that makes it easy to deploy, manage, and scale applications on AWS without worrying about the underlying infrastructure. You simply upload your application, and Elastic Beanstalk automatically handles the setup and management for you.
</h2>

<p>Elastic Beanstalk supports popular platforms such as:</p>

<ul>
<li>Java</li>
<li>.NET</li>
<li>Node.js</li>
<li>Python</li>
<li>PHP</li>
<li>Ruby</li>
<li>Docker</li>
</ul>




<h3>
  
  
  How AWS Elastic Beanstalk Works
</h3>

<p>When you deploy an application using Elastic Beanstalk, AWS automatically provisions and manages the required resources, including:</p>

<ul>
<li>Amazon EC2 instances</li>
<li>Load balancers</li>
<li>Auto Scaling groups</li>
<li>Monitoring and health checks</li>
</ul>

<p>Although Elastic Beanstalk manages these resources for you, you still have access to them if you need more control.</p>




<h3>
  
  
  Benefits of AWS Elastic Beanstalk
</h3>

<p>Some key benefits include:</p>

<ul>
<li>
<strong>Easy deployment</strong> – Upload your application and let AWS handle the rest</li>
<li>
<strong>Automatic scaling</strong> – Your application scales up or down based on traffic</li>
<li>
<strong>Cost-effective</strong> – You only pay for the AWS resources used</li>
<li>
<strong>Less infrastructure management</strong> – Focus more on development, less on servers</li>
<li>
<strong>Flexible control</strong> – You can customize settings when needed</li>
</ul>




<h3>
  
  
  AWS Elastic Beanstalk vs Amazon EC2
</h3>

<p>A simple way to understand the difference:</p>

<ul>
<li><p><strong>Amazon EC2</strong>:<br>
You are responsible for managing servers, updates, scaling, and monitoring.</p></li>
<li><p><strong>AWS Elastic Beanstalk</strong>:<br>
AWS manages the infrastructure, while you focus on your application.</p></li>
</ul>

<p>Elastic Beanstalk actually runs on EC2, but it removes the complexity of managing everything manually.</p>




<h3>
  
  
  When Should You Use AWS Elastic Beanstalk?
</h3>

<p>AWS Elastic Beanstalk is a good choice if:</p>

<ul>
<li>You want a fast and easy way to deploy applications</li>
<li>You do not want to manage servers directly</li>
<li>You are new to AWS and want a simpler starting point</li>
<li>You need automatic scaling and monitoring</li>
</ul>




<h2>
  
  
  Here is a Step-by-Step Guide to Creating an AWS Elastic Beanstalk Application
</h2>

<h3>
  
  
  Step 1: Sign in to the AWS Management Console
</h3>

<p>Go to the AWS Management Console and sign in with your AWS account.</p>




<h3>
  
  
  Step 2: Navigate to Elastic Beanstalk
</h3>

<ul>
<li>In the AWS search bar, type <strong>Elastic Beanstalk</strong>
</li>
<li>Click <strong>Elastic Beanstalk</strong> from the services list</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flqxrle0n42h7r3en8vx6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flqxrle0n42h7r3en8vx6.png" alt=" " width="800" height="428"></a></p>




<h3>
  
  
  Step 3: Create a New Application
</h3>

<ul>
<li>Click <strong>Create application</strong>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F446wozr5465fu2i59bqq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F446wozr5465fu2i59bqq.png" alt=" " width="800" height="453"></a></p>

<ul>
<li>Enter an <strong>Application name</strong> (for example: <code>getting-started-app</code>)</li>
<li>(Optional) Add a description</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm7poobsz7b58wzhba1cp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm7poobsz7b58wzhba1cp.png" alt=" " width="800" height="446"></a></p>




<h3>
  
  
  Step 4: Choose Your Platform
</h3>

<ul>
<li>
<p>Under <strong>Platform</strong>, select the programming language your app uses<br>
Examples:</p>

<ul>
<li>Node.js</li>
<li>Python</li>
<li>PHP</li>
<li>Java</li>
<li>.NET</li>
</ul>


</li>

<li><p>Choose the <strong>platform version</strong> (you can leave the default)</p></li>

</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcctcqz70du4e0kcsiv7i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcctcqz70du4e0kcsiv7i.png" alt=" " width="800" height="453"></a></p>




<h3>
  
  
  Step 5: Upload Your Application Code
</h3>

<ul>
<li>
<p>Under <strong>Application code</strong>, choose:</p>

<ul>
<li>
<strong>Sample application</strong> (recommended for first-time users),</li>
</ul>


</li>

</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzet02yz8iig87r9wsii2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzet02yz8iig87r9wsii2.png" alt=" " width="800" height="453"></a></p>

<p>or</p>

<ul>
<li>
<strong>Upload your code</strong> (if you already have an app)

<ul>
<li>If uploading your code, select your file (ZIP format)</li>
</ul>


</li>

</ul>




<h3>
  
  
  Step 6: Configure Application Settings (Service Role, Instance Profile, and Key Pair)
</h3>

<p>Before creating the environment, you need to configure security and access settings so Elastic Beanstalk can manage AWS resources and you can securely access your EC2 instances.</p>




<h4>
  
  
  Step 6.1: Configure the Elastic Beanstalk Service Role
</h4>

<p>The service role allows Elastic Beanstalk to interact with other AWS services on your behalf.</p>

<p><strong>To configure service access</strong><br>
Next, you need two roles. A service role allows Elastic Beanstalk to monitor your EC2 instances and upgrade your environment’s platform. An EC2 instance profile role permits tasks such as writing logs and interacting with other services.</p>

<ul>
<li>
<strong>To create or select the Service role</strong>
If you have previously created a Service role and would like to choose an existing one, select the value from the Service role drop-down and skip the remainder of these steps to create a Service role.</li>
</ul>

<p>If you don't see any values listed for the Service role, or you'd like to create a new one, continue with the next steps.</p>

<p><strong>For Service role, choose Create role.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqey0tygxdg04f19tk5c0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqey0tygxdg04f19tk5c0.png" alt=" " width="800" height="421"></a></p>

<p><strong>For Trusted entity type, choose AWS service.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fesk2vqlvk2y4sd2mwbk2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fesk2vqlvk2y4sd2mwbk2.png" alt=" " width="800" height="428"></a></p>

<p><strong>For Use case, choose Elastic Beanstalk – Environment.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc1hlcsdf5ht0jxc9y42f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc1hlcsdf5ht0jxc9y42f.png" alt=" " width="800" height="386"></a></p>

<p><strong>Choose Next.</strong></p>

<p><strong>Verify that Permissions policies include the following, then choose Next:</strong></p>

<p>AWSElasticBeanstalkEnhancedHealth</p>

<p>AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fya40g1n95e02uli49fxy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fya40g1n95e02uli49fxy.png" alt=" " width="800" height="289"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwscgomi3qxqxq6vf7mw3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwscgomi3qxqxq6vf7mw3.png" alt=" " width="800" height="310"></a></p>

<p><strong>Choose Create role.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl4zvjkqfvcrkgs97hbwa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl4zvjkqfvcrkgs97hbwa.png" alt=" " width="800" height="323"></a></p>

<p>Return to the Configure service access tab, refresh the list, then select the newly created service role.</p>

<p><strong>Step 6.2 To create or select an EC2 instance profile</strong><br>
The EC2 instance profile allows EC2 instances in your environment to access AWS services securely.<br>
If you have previously created an EC2 instance profile and would like to choose an existing one, select the value from the EC2 instance profile drop-down and skip the remainder of these steps to create an EC2 instance profile.</p>

<p>If you don't see any values listed for EC2 instance profile, or you'd like to create a new one, continue with the next steps.</p>

<p><strong>Choose Create role.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fltsqkk3m78xyzw0sx0d4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fltsqkk3m78xyzw0sx0d4.png" alt=" " width="800" height="383"></a></p>

<p><strong>For Trusted entity type, choose AWS service.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fymv7sl2csaotaebrsmph.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fymv7sl2csaotaebrsmph.png" alt=" " width="800" height="331"></a></p>

<p><strong>For Use case, choose Elastic Beanstalk – Compute.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj97xl10i3q02msyizyfa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj97xl10i3q02msyizyfa.png" alt=" " width="800" height="315"></a></p>

<p><strong>Choose Next.</strong></p>

<p><strong>Verify that Permissions policies include the following, then choose Next:</strong></p>

<p>AWSElasticBeanstalkWebTier</p>

<p>AWSElasticBeanstalkWorkerTier</p>

<p>AWSElasticBeanstalkMulticontainerDocker</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiwd33i32ntdlpf9wqn8c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiwd33i32ntdlpf9wqn8c.png" alt=" " width="800" height="311"></a></p>

<p><strong>Choose Create role.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw6ze97hcpxvbzjx5xpdn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw6ze97hcpxvbzjx5xpdn.png" alt=" " width="800" height="314"></a></p>

<p>Return to the Configure service access tab, refresh the list, then select the newly created EC2 instance profile.</p>

<p>To finish configuring and creating your application<br>
<strong>Step 6.3: Select a Key Pair</strong></p>

<p>A key pair allows you to securely connect to the EC2 instances created by Elastic Beanstalk.</p>

<p><strong>Search for key pair</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffdgvzqunv8uqr0bef3ot.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffdgvzqunv8uqr0bef3ot.png" alt=" " width="800" height="430"></a></p>

<p><strong>Under EC2 key pair: Select an existing key pair, or Click Create a new key pair</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxpennl411fiynmi21p3u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxpennl411fiynmi21p3u.png" alt=" " width="800" height="319"></a><br>
<strong>Enter a key pair name</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1qj65ynh2awmd8o8mnsv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1qj65ynh2awmd8o8mnsv.png" alt=" " width="800" height="315"></a><br>
Download and store the key file securely</p>

<p><strong>Without a key pair, you will not be able to connect to the EC2 instance using SSH.</strong></p>

<p>Optional steps: networking, databases, scaling parameters, advanced configuration for updates, monitoring, and logging.</p>

<p>On the Review page, which shows a summary of your choices, <strong>choose Submit.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwnqmuxfecu26icf4ujcq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwnqmuxfecu26icf4ujcq.png" alt=" " width="800" height="317"></a></p>




<h4>
  
  
  Step 6.4: Review Other Settings (Optional)
</h4>

<p>You can keep the default settings for now or adjust them if needed:</p>

<ul>
<li>Instance type (EC2 size)</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy0qeccj9m383gb7887mq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy0qeccj9m383gb7887mq.png" alt=" " width="800" height="313"></a></p>

<ul>
<li>Auto scaling options</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftk8r5thplmcdhv0eytlj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftk8r5thplmcdhv0eytlj.png" alt=" " width="800" height="420"></a></p>

<ul>
<li>Environment variables</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwqb16h5nye0tb524mxcq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwqb16h5nye0tb524mxcq.png" alt=" " width="800" height="387"></a></p>

<ul>
<li>Security groups</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fglcqomy1611243zow3cs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fglcqomy1611243zow3cs.png" alt=" " width="800" height="387"></a></p>

<p>Beginners can safely proceed with the default configuration.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi0058baunkyr3luijmo7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi0058baunkyr3luijmo7.png" alt=" " width="800" height="317"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmzkgtjgtjagjp7b2h5zb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmzkgtjgtjagjp7b2h5zb.png" alt=" " width="800" height="398"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedfu8suij4rcunwe41rk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedfu8suij4rcunwe41rk.png" alt=" " width="800" height="311"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc0pdjtncv5cfo4cnayqg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc0pdjtncv5cfo4cnayqg.png" alt=" " width="800" height="326"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7atn111bnl3q1nm8wbwo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7atn111bnl3q1nm8wbwo.png" alt=" " width="800" height="317"></a></p>




<h3>
  
  
  Step 7: Create the Environment
</h3>

<ul>
<li>Click <strong>Create application</strong>
</li>
<li>
<p>AWS will automatically set up the required resources, including:</p>

<ul>
<li>EC2 instances</li>
<li>Load balancer</li>
<li>Auto Scaling group</li>
<li>Monitoring and health checks</li>
</ul>


</li>

</ul>

<p><strong>This process may take a few minutes. Once complete, your application will be live.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7etmovmwhocv374cw2i0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7etmovmwhocv374cw2i0.png" alt=" " width="800" height="455"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3mdkunbiv542xu0t74xt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3mdkunbiv542xu0t74xt.png" alt=" " width="800" height="403"></a></p>

<p><strong>And your Beanstalk is LIVE</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fclvneayfk4o1nfyj0ocm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fclvneayfk4o1nfyj0ocm.png" alt=" " width="800" height="374"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flhmufxho74s9uy0kslgf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flhmufxho74s9uy0kslgf.png" alt=" " width="800" height="392"></a></p>

<p><strong>To access the EC2 created</strong><br>
On the AWS Console, search EC2</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F88g5qt03w6mhj87dx0jw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F88g5qt03w6mhj87dx0jw.png" alt=" " width="800" height="397"></a></p>

<p>** Check the dashboard to confirm the instance status is <strong>running</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa1j2vt425suxl29wbh3j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa1j2vt425suxl29wbh3j.png" alt=" " width="800" height="386"></a></p>

<p><strong>Click on the App created and then connect</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekv4wczx8anrf9u120zs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fekv4wczx8anrf9u120zs.png" alt=" " width="800" height="389"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frje7xnfhmwrzt00h0ar0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frje7xnfhmwrzt00h0ar0.png" alt=" " width="800" height="299"></a></p>

<p><strong>And here is the EC2 instance that was created  while setting up the Elastic Beanstalk</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcyo10pwzw0ehnaljuz9y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcyo10pwzw0ehnaljuz9y.png" alt=" " width="800" height="413"></a></p>

<p><strong>S3 Bucket created</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1y7zzg1clawcgocj7coo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1y7zzg1clawcgocj7coo.png" alt=" " width="800" height="419"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi7ahyi8frzgeejxnm3df.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi7ahyi8frzgeejxnm3df.png" alt=" " width="800" height="395"></a></p>

<p><strong>Auto Scaling group created</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7f489v4ft9uav1qmh797.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7f489v4ft9uav1qmh797.png" alt=" " width="800" height="432"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd3sqsk702ixngggw63yd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd3sqsk702ixngggw63yd.png" alt=" " width="800" height="437"></a></p>




<h3>
  
  
  Step 8: Terminate the Application
</h3>

<p>** To stop billing: <strong>Terminate the environment</strong> when you’re done<br>
To delete the application and all associated resources<br>
Delete all application versions.</p>

<p>Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.</p>

<p>In the navigation pane, choose Applications, and then choose getting-started-app.</p>

<p>In the navigation pane, find your application's name and choose Application versions.</p>

<p>On the Application versions page, select all application versions that you want to delete.</p>

<p>Choose Actions, and then choose Delete.</p>

<p>Turn on Delete versions from Amazon S3.</p>

<p>Choose Delete, and then choose Done.</p>

<p>Terminate the environment.</p>

<p>In the navigation pane, choose getting-started-app, and then choose GettingStartedApp-env in the environment list.</p>

<p>Choose Actions, and then choose Terminate Environment.</p>

<p>Confirm that you want to terminate GettingStartedApp-env by typing the environment name, and then choose Terminate.</p>

<p>Delete the getting-started-app application.</p>

<p>In the navigation pane, choose the getting-started-app.</p>

<p>Choose Actions, and then choose Delete application.</p>

<p>Confirm that you want to delete getting-started-app by typing the application name, and then choose Delete.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpz0ue09muuce3uocn6x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpz0ue09muuce3uocn6x.png" alt=" " width="800" height="417"></a></p>




<h3>
  
  
  Conclusion
</h3>

<p>AWS Elastic Beanstalk simplifies application deployment by handling infrastructure management for you behind the scenes. It is a great option for developers who want the power of AWS without the complexity of managing servers manually. </p>

