---
Title: AWS IAM Roles Anywhere Demo
Description: 
Author: John McCracken
Date: 2025-08-24T19:00:20.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>IAM user access keys are the normal way to access AWS from an external source. Generally this is the method to use for machine/code access to your AWS estate. </p>

<p>This is ok, but come with some issues:</p>

<ul>
<li>Long lived credentials expand the window of opportunity for an attacker to exploit the access key.</li>
<li>Rotating secrets can be hard to manage; synchronising credentials rotating with third parties can be complex.</li>
<li>It just doesn't scale, anything over a handful of access keys becomes hard to manage.</li>
</ul>

<p>The AWS preferred solution is to use <a href="https://aws.amazon.com/iam/roles-anywhere/" rel="noopener noreferrer">AWS IAM Roles Anywhere</a> for workloads outside of AWS.</p>

<ul>
<li>Short term rotating credentials</li>
<li>Uses industry standard X.509 certificates</li>
<li>Reduces management complexity</li>
</ul>

<p>Although a bit more complex to set up, AWS IAM Roles Anywhere is a much more elegant, scalable solution for workloads. It also ticks all current AWS security best practices.</p>

<h2>
  
  
  How does IAM Roles Anywhere work?
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp6m3rklocglev65fks26.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp6m3rklocglev65fks26.jpg" alt="IAM Roles Anywhere process flow" width="661" height="678"></a></p>

<ul>
<li>Workloads use <strong>SSL Certificates</strong> (<strong>end-entity certificate</strong>) which have been signed by a <strong>Certificate Authority</strong> (<strong>CA</strong>). The CA can either be external or provided by AWS.</li>
<li>Create a Trust Anchor between IAM Roles Anywhere and the CA</li>
<li>Create a Profile, which defines the IAM policy to apply after successful authentication</li>
</ul>

<p>Be aware that although IAM Roles Anywhere is free, if you use AWS Private CA there will be a cost, it’s around $400 per month! I believe you can cancel within the first 30 days free of charge.</p>

<p>In this demo, I’ll stick with a free external self generated one. Its a bit of a pain to generate, but hopefully this demo makes it as painless as possible.</p>

<h2>
  
  
  What this guide intends to do
</h2>

<ul>
<li>Generate an external CA</li>
<li>Generate an end-entity certificate which is signed off against the CA.</li>
<li>Setup a IAM Roles Anywhere Trust Anchor with the CA certificate</li>
<li>Setup a IAM Roles Anywhere Profile and link a IAM Policy</li>
<li>Setup a local machine to use <strong>aws_signing_helper</strong> in an AWS Profile.</li>
<li>Demonstrate it works with AWS CLI and Python/boto3.
##Setting up IAM Roles Anywhere</li>
</ul>

<p>I’ve tried to script as much as possible, that way you can get it running as quickly as possible and it still offers the full details for those who want to dig deeper under the hood.</p>

<p>I’ve created a repo for all this goodness: <a href="https://github.com/johnmccuk/aws-iam-roles-anywhere-public-demo" rel="noopener noreferrer">aws-iam-roles-anywhere-public-demo</a></p>

<p>This contains the following:</p>

<ul>
<li>Shell script to generate CA and end certificates</li>
<li>Terraform to deploy to AWS</li>
<li>Python example script to test access</li>
</ul>

<p>All instructions are for Linux/MacOS. If you're using Windows, you've got bigger problems than trying to follow this blog.</p>

<h3>
  
  
  Generate a Certificate Authority and a End Entity Certificate
</h3>

<p>On the CLI and clone the repo and go to the root directory. There is a bash script located here called <code>create-ca.sh</code> which can be used to automatically generate the required Certificates. </p>

<p>This uses a config file <code>root-ca/root-ca.conf</code>, you can use the default values or edit to suit your requirements.</p>

<p>Open the script <a href="https://github.com/johnmccuk/aws-iam-roles-anywhere-public-demo/blob/main/create-ca.sh" rel="noopener noreferrer">create-ca.sh</a> and have a look over what it's actually doing, the comments hopefully help clarify.</p>

<p>The certificates values are stored in <a href="https://github.com/johnmccuk/aws-iam-roles-anywhere-public-demo/blob/main/root-ca/root-ca.conf" rel="noopener noreferrer">root-ca.conf</a> and <a href="https://github.com/johnmccuk/aws-iam-roles-anywhere-public-demo/blob/main/root-ca/client-ca.conf" rel="noopener noreferrer">client-ca.conf</a>.</p>

<p>Once you're comfortable with the contents, run the script: </p>

<p><code>sh ./create-ca.sh</code></p>

<p>When prompted, select <strong>Y</strong> for any prompts.</p>

<p>You should now have the following created in the root-ca folder:</p>

<ul>
<li>certs folder containing a CA .pem file</li>
<li>db folder with CA version data</li>
<li>private folder with a private key and certificate signing request file</li>
<li>clients folder with end-entity certificate key/pem file</li>
</ul>

<p>This is all the files required to get started. Be aware this is all basic stuff and shouldn't be used in a production environment. </p>

<p><strong>Note</strong>: this script copies the following files from the ./clients/ folder into your ~/.ssh folder: </p>

<p><code>~/.ssh/iam-roles-anywhere-demo.key</code><br>
<code>~/.ssh/iam-roles-anywhere-demo.crt</code></p>

<p>Please delete these manually afterwards.</p>
<h3>
  
  
  Deploy the Infrastructure as Code
</h3>

<p>Feel free to deploy by click-ops, it's pretty self explanatory, but everything really should be IaC.</p>

<p>Make sure you have authenticated to the AWS account you wish to use, by pasting Access keys into the CLI or however you normally do it.</p>

<p>From the root directory, <code>cd terraform</code>then</p>

<p><code>terraform init</code><br>
<code>terraform apply</code></p>

<p>Plan output should create the following:</p>

<p><strong>aws_rolesanywhere_trust_anchor</strong><br>
Create a Trust Anchor, this connects IAM Roles Anywhere to the certificate.</p>

<p><strong>aws_iam_role</strong><br>
This is the role used by IAM Anywhere, it has a condition that the certificate Common Name (CN) must be ‘IAM Anywhere Demo’.</p>

<p><strong>aws_iam_role_policy_attachment</strong><br>
Attach the managed policy <strong>AmazonS3ReadOnlyAccess</strong> to the role</p>

<p><strong>aws_rolesanywhere_profile</strong><br>
Create an IAM Roles Anywhere Profile called ‘iam-roles-anywhere-demo’. Note that the profile also is linked to the managed policy <strong>AmazonS3ReadOnlyAccess</strong>. The profile can set the maximum permission boundary. </p>

<p>For example, if I had attached AmazonS3FullAccess to the role, the Profile permissions would restrict S3 permissions to read only.</p>

<p>Check over the plan and if all looks good to you, type <strong>yes</strong> to deploy.</p>

<p>If all goes well, the resources should be deployed to your specified AWS Account. </p>

<p>The output should contain a key <strong>credential_process</strong>, copy the value for this key and paste it somewhere safe for now. This outputs the aws command required later to authenticate. It should be something similar to this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>credential_process = "~/.aws/aws_signing_helper credential-process --certificate ~/.ssh/iam-roles-anywhere.crt --private-key ~/.ssh/iam-roles-anywhere.key --trust-anchor-arn arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:trust-anchor/880ee543-af45-48c7-b5cd-bf0b73e95059 --profile-arn arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:profile/048f1bf7-da4c-41cd-99ed-ee6b93021297 --role-arn arn:aws:iam::ACCOUNTID:role/iam-roles-anywhere-demo"
profile-arn = "arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:profile/048f1bf7-da4c-41cd-99ed-ee6b93021297"
role-arn = "arn:aws:iam::ACCOUNTID:role/iam-roles-anywhere-demo"
trust-anchor-arn = "arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:trust-anchor/880ee543-af45-48c7-b5cd-bf0b73e95059"
</code></pre>

</div>



<h2>
  
  
  Test AWS Access
</h2>

<p>If you want to view the resources in the AWS console, go to the IAM Dashboard, then Roles and then scroll to the bottom of the main window.</p>

<p>Firstly, AWS provide <a href="https://docs.aws.amazon.com/rolesanywhere/latest/userguide/credential-helper.html" rel="noopener noreferrer">aws_signing_helper</a> to simplify access, download and install, make sure its in your <code>PATH</code>.</p>

<p>You can use this file to retrieve short term credentials and then add them to environmental variables as you can with AWS IAM access keys, but I have found a better way is to update the <code>~/.aws/config</code> file:</p>

<p>Open <code>~/.aws/config</code> and create a new entry profile:</p>

<p><code>[profile iam-roles-anywhere-demo]</code></p>

<p>Now directly below this, paste the Terraform output mentioned previously, so it should end up something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[profile iam-roles-anywhere-demo]
credential_process = aws_signing_helper credential-process --certificate ~/.ssh/iam-roles-anywhere.crt --private-key ~/.ssh/iam-roles-anywhere.key --trust-anchor-arn arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:trust-anchor/880ee543-af45-48c7-b5cd-bf0b73e95059 --profile-arn arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:profile/048f1bf7-da4c-41cd-99ed-ee6b93021297 --role-arn arn:aws:iam::ACCOUNTID:role/iam-roles-anywhere-demo
profile-arn = "arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:profile/048f1bf7-da4c-41cd-99ed-ee6b93021297"
role-arn = "arn:aws:iam::ACCOUNTID:role/iam-roles-anywhere-demo"
trust-anchor-arn = "arn:aws:rolesanywhere:eu-west-1:ACCOUNTID:trust-anchor/880ee543-af45-48c7-b5cd-bf0b73e95059”
</code></pre>

</div>



<p>Now save and exit file</p>

<p><strong>Note</strong>: You might need to replace <code>/.ssh</code> with the full path, in my case this was <code>/Users/john.mccracken/.ssh</code></p>

<h3>
  
  
  Testing with AWS CLI
</h3>

<p><code>aws sts get-caller-identity --profile iam-roles-anywhere-demo</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
    "UserId": "XXXX",
    "Account": "1234567890",
    "Arn": "arn:aws:sts::1234567890:assumed-role/iam-roles-anywhere-demo/00bc75fc5a6709cca9402a061453a2f3a9"
}
</code></pre>

</div>



<p>Then try</p>

<p><code>aws s3 ls --profile iam-roles-anywhere-demo</code></p>

<h3>
  
  
  Testing programmatically with Python/boto3
</h3>

<p>In the python-iam-anywhere-test folder, there is a test.py file which will list buckets from the Account, there is a poetry.lock file with the required dependencies (boto3).</p>

<p>From the root directory, cd <code>python-iam-anywhere-test</code> then</p>

<p><code>poetry install</code><br>
<code>poetry run python test.py<br>
</code></p>

<p>This should list all S3 buckets.</p>

<h2>
  
  
  Conclusion
</h2>

<p>IAM Roles Anywhere is a great elegant solution for external access to AWS resources. Setting up your own CA is a pain, but it's once done, the integration with AWS is pretty seamless and straightforward. </p>

<p>aws_signing_helper makes life really simple, if you want an idea of how complex the process is under the hood, check out the <a href="https://github.com/aws/rolesanywhere-credential-helper" rel="noopener noreferrer">repo</a>.</p>

<p>Now I have IAM Roles Anywhere setup, I intend to use it going forward rather than access keys.</p>

<p>Any feedback, please get in touch. I'd love to hear from you.</p>

<h2>
  
  
  Additional links
</h2>

<p>Always standing on the shoulder of giants:</p>

<ul>
<li><p><a href="https://medium.com/cyberark-engineering/calling-aws-services-from-your-on-premises-servers-using-iam-roles-anywhere-3e335ed648be" rel="noopener noreferrer">Create a CA for use with AWS IAM Roles Anywhere</a></p></li>
<li><p><a href="https://aws.amazon.com/blogs/security/iam-roles-anywhere-with-an-external-certificate-authority/" rel="noopener noreferrer">IAM Roles Anywhere with an external certificate authority</a></p></li>
<li><p><a href="https://aws.amazon.com/blogs/security/extend-aws-iam-roles-to-workloads-outside-of-aws-with-iam-roles-anywhere/" rel="noopener noreferrer">Extend AWS IAM roles to workloads outside of AWS with IAM Roles Anywhere</a></p></li>
<li><p><a href="https://docs.aws.amazon.com/rolesanywhere/latest/userguide/credential-helper.html#credential-helper-credential-process" rel="noopener noreferrer">Get temporary security credentials from IAM Roles Anywhere</a></p></li>
<li><p><a href="https://github.com/ivanr/bulletproof-tls/tree/master/private-ca" rel="noopener noreferrer">Examples derived from Ivans Ristic's Github</a></p></li>
</ul>

