---
Title: Getting Started with AWS Config: Monitor, Detect, and Remediate Non-Compliant Resources
Description: 
Author: harsh patel
Date: 2025-10-23T21:48:32.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Getting Started with AWS Config
</h1>

<p>AWS Config is a powerful management service that continuously monitors and records the configuration of your AWS resources. It helps ensure compliance by evaluating these resources and their relationships within a specific AWS Region. Whenever a resource deviates from the defined configuration rules, AWS Config flags it as non-compliant, allowing you to identify and remediate issues proactively.</p>

<p>In this Cloud Lab, you’ll learn how to use AWS Config to enforce compliance on EC2 security groups. You’ll begin by creating an IAM role that grants AWS Config the necessary permissions to perform its operations. Next, you’ll configure AWS Config to monitor security groups in the us-east-1 (N. Virginia) Region. After that, you’ll define and apply Config Rules to evaluate compliance across your resources.</p>

<p>Once AWS Config is set up, you’ll intentionally create non-compliant resources to observe how it detects and reports them. You’ll then configure remediation actions that automatically bring these resources back into compliance.</p>

<p>By the end of this lab, you’ll be able to confidently use AWS Config to:</p>

<ul>
<li>Monitor AWS resources in real time
</li>
<li>Detect configuration drift or non-compliance
</li>
<li>Automatically enforce your organization’s security and governance policies</li>
</ul>




<h2>
  
  
  Architecture diagram
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbbyv1dxz8ro00uumqhu4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbbyv1dxz8ro00uumqhu4.png" alt=" " width="800" height="478"></a></p>

<h2>
  
  
  1. Create an IAM Role
</h2>

<p>An IAM role is an AWS identity with a set of defined permissions. Unlike a user, a role can be assumed by trusted entities, including AWS services, to gain temporary access for specific actions.</p>

<p>Every role consists of:</p>

<ul>
<li>A trust policy — defines who can assume the role.</li>
<li>A permissions policy — defines what those entities can do.</li>
</ul>

<p>In this step, you’ll create an IAM role for AWS Config, granting it permissions to monitor and manage AWS resources.</p>




<h3>
  
  
  Steps to Create the IAM Role
</h3>

<ol>
<li><p><strong>Open the IAM Console</strong><br><br>
Search for “IAM” in the AWS Console and select it to open the dashboard.</p></li>
<li><p><strong>Create a New Role</strong><br><br>
Go to Roles → Create role.</p></li>
<li><p><strong>Select the Trusted Entity</strong><br><br>
Under Use case, search and select Config.<br><br>
Click Next.</p></li>
<li><p><strong>Attach Permissions</strong><br><br>
The policy AWSConfigServiceRolePolicy will be automatically selected.<br><br>
Click Next.</p></li>
<li><p><strong>Review and Create</strong><br><br>
The default name AWSServiceRoleForConfig will appear automatically.<br><br>
Click Create role.</p></li>
</ol>

<blockquote>
<p>⚠️ Note: If the error “A role with this name already exists” appears, use the existing service role instead of creating a new one.</p>
</blockquote>




<h2>
  
  
  2. Start the Configuration Recorder
</h2>

<p>The Configuration Recorder is the core of AWS Config. It continuously tracks configuration changes for selected AWS resources.<br><br>
In this step, you’ll enable it to monitor EC2 security groups within your Region.</p>




<h3>
  
  
  Steps to Start the Configuration Recorder
</h3>

<ol>
<li><p><strong>Open AWS Config</strong><br><br>
Search for “Config” and select the service.<br><br>
You’ll land on the AWS Config Home page.</p></li>
<li><p><strong>Set Up AWS Config</strong><br><br>
Click <em>Set up AWS Config</em> in the sidebar.</p></li>
<li>
<p><strong>Select Specific Resources</strong><br><br>
Under Recording method, choose <em>Specific resource types</em>.<br><br>
Add:   </p>

<ul>
<li>AWS EC2 SecurityGroup → Frequency: Continuous</li>
</ul>
</li>
<li><p><strong>Assign the IAM Role</strong><br><br>
Under Data governance, choose <em>Select a role from your account</em> and pick the previously created role.</p></li>
<li><p><strong>Configure Delivery Method</strong><br><br>
Under Delivery method, select <em>Create a bucket</em>.<br><br>
This S3 bucket will store all configuration logs.<br><br>
Click <em>Next</em>.</p></li>
<li><p><strong>Skip Rule Selection</strong><br><br>
On the Rules page, click <em>Next</em> without selecting any rules (we’ll add them later).</p></li>
<li><p><strong>Review and Confirm</strong><br><br>
Review your setup and click <em>Confirm</em>.<br><br>
AWS Config will now record configuration changes for all security groups in your region.</p></li>
</ol>

<blockquote>
<p>💡 If Config Is Already Enabled:<br><br>
Go to Settings → Edit, select Specific resource types, and configure only AWS EC2 SecurityGroup for Continuous recording.</p>
</blockquote>




<h2>
  
  
  3. Add AWS Config Rules
</h2>

<p>AWS Config Rules allow AWS Config to evaluate resources against compliance standards. AWS provides managed rules, and you can also create custom rules.</p>

<p>In this lab, you’ll define managed rules to enforce your organization’s policy:</p>

<ul>
<li>
<strong>restricted-ssh</strong> → Ensures no public SSH access (port 22)
</li>
</ul>

<h3>
  
  
  Steps to Add AWS Config Rules
</h3>

<h4>
  
  
  Add the “restricted-ssh” Rule
</h4>

<ol>
<li>From the AWS Config dashboard, select Rules → Add rule.</li>
<li>Choose <em>Add AWS managed rule</em>.</li>
<li>Search for and select <strong>restricted-ssh</strong>.</li>
<li>Click <em>Next</em>, leave defaults, and click <em>Next</em> again.</li>
<li>Review and click <em>Save</em>.</li>
</ol>

<p>This rule marks any EC2 security group that allows public SSH access as non-compliant.</p>

<p>AWS Config will now begin evaluating resources against these rules.</p>




<h2>
  
  
  4. Provision EC2 Resources
</h2>

<p>Now, you’ll create EC2 resources that AWS Config can evaluate.<br><br>
We’ll intentionally make one of them non-compliant to observe AWS Config’s detection capabilities.</p>

<h3>
  
  
  Create a Security Group
</h3>

<ol>
<li>Open the EC2 Dashboard → Search “EC2” in the console.</li>
<li>Go to <em>Network &amp; Security → Security Groups</em>.</li>
<li>Click <em>Create security group</em>.</li>
<li>Set:

<ul>
<li>Name: MySecurityGroup
</li>
<li>Description: Security group allowing SSH access</li>
</ul>
</li>
<li>Under <em>Inbound rules</em>, click <em>Add rule</em>:

<ul>
<li>Type: SSH
</li>
<li>Source: Anywhere-IPv4 (0.0.0.0/0)</li>
</ul>
</li>
<li>Click <em>Create security group.</em>
</li>
</ol>

<p>This configuration allows public SSH access — intentionally non-compliant per your rule.</p>




<h3>
  
  
  Create an EC2 Instance
</h3>

<ol>
<li>Go to <em>Instances → Launch instances.</em>
</li>
<li>Set the following:

<ul>
<li>Name: client
</li>
<li>AMI: Amazon Linux 2023 AMI
</li>
<li>Instance type: t2.micro
</li>
<li>Key pair: Proceed without key pair
</li>
<li>Network settings: Select existing security group → MySecurityGroup
</li>
<li>Storage: Default gp3</li>
</ul>
</li>
<li>Click <em>Launch instance.</em>
</li>
<li>Wait for the instance status to show <em>Running.</em>
</li>
</ol>

<p>AWS Config will now start evaluating these resources.</p>




<h2>
  
  
  5. Check the Resources’ Compliance
</h2>

<p>Now let’s verify if your security group comply with the defined rules.</p>

<h3>
  
  
  Steps to Verify Compliance
</h3>

<ol>
<li>Open AWS Config → Rules.</li>
<li>Under restricted-ssh, look for 1 Noncompliant resource(s).
If none appear, wait about a minute and refresh.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0ueicid61040xe0lnvna.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0ueicid61040xe0lnvna.png" alt=" " width="800" height="505"></a></p>

<ol>
<li>Click the restricted-ssh rule → view Resources in scope.</li>
<li>Click the Resource ID to confirm it’s your MySecurityGroup.</li>
<li>To view its details, click <em>View Configuration Item (JSON)</em>. You’ll find:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>json
"ipPermissions": [
    {
        "fromPort": 22,
        "ipProtocol": "tcp",
        "toPort": 22,
        "ipv4Ranges": [
            {
                "cidrIp": "0.0.0.0/0"
            }
        ]
    }
]
</code></pre>

</div>



<p>This shows that port 22 (SSH) is open to the public (0.0.0.0/0) — making the security group non-compliant.</p>




<h2>
  
  
  6. Add and Use a Remediation Action
</h2>

<p>Once AWS Config identifies non-compliance, you can fix it using remediation actions.<br><br>
AWS Config integrates with AWS Systems Manager Automations to perform predefined remediation workflows.</p>

<p>Here, you’ll create and execute a remediation action to automatically remove public SSH access from your EC2 security group.</p>




<h3>
  
  
  Add the Remediation Action
</h3>

<ol>
<li>Open AWS Config → Rules → restricted-ssh.</li>
<li>Click <em>Actions → Manage remediation</em>.</li>
<li>Under <em>Select remediation method</em>, choose <em>Manual remediation</em>.</li>
<li>In <em>Remediation action details</em>, select <strong>AWS-DisablePublicAccessForSecurityGroup</strong>.</li>
<li>For <em>Resource ID parameter</em>, choose <em>GroupId</em>.</li>
<li>In <em>Parameters</em>, set:

<ul>
<li>IpAddressToBlock: 0.0.0.0/0</li>
</ul>
</li>
<li>Click <em>Save changes.</em>
</li>
</ol>




<h3>
  
  
  Use the Remediation Action
</h3>

<ol>
<li>Open the restricted-ssh rule again.</li>
<li>In <em>Resources in scope</em>, select the non-compliant security group.</li>
<li>Click <em>Remediate.</em>
</li>
<li>Once the process completes, you’ll see “Action executed successfully.”</li>
<li>Refresh the page — the resource should disappear from the non-compliant list.</li>
<li>Finally, verify from the EC2 Dashboard → Security Groups that the inbound SSH rule allowing public access has been removed.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwyl39lznaliqro2movui.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwyl39lznaliqro2movui.png" alt=" " width="800" height="403"></a></p>

<p>You’ve successfully remediated your non-compliant resource.</p>




<h2>
  
  
  Conclusion
</h2>

<p>In this Cloud Lab, you learned how to:</p>

<ul>
<li>Configure AWS Config to monitor and record resource changes
</li>
<li>Define compliance rules for security groups
</li>
<li>Detect and review non-compliant resources
</li>
<li>Apply remediation actions to automatically fix violations
</li>
</ul>

<p>With AWS Config, you can continuously track, audit, and enforce configuration compliance — ensuring your AWS environment remains secure and aligned with organizational policies.</p>

