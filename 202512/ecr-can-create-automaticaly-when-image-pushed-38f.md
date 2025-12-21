---
Title: ECR can create automaticaly when image pushed
Description: 
Author: Soushi Hiruta
Date: 2025-12-21T21:51:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>ECR can create automaticaly when image pushed</p>

<h2>
  
  
  Update
</h2>

<p><a href="https://aws.amazon.com/about-aws/whats-new/2025/12/amazon-ecr-creating-repositories-on-push/" rel="noopener noreferrer">https://aws.amazon.com/about-aws/whats-new/2025/12/amazon-ecr-creating-repositories-on-push/</a></p>

<h2>
  
  
  Repository Create Template
</h2>

<p>Frist need to create Repository Create template.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuziqnnbb7n45pr4oqp5q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuziqnnbb7n45pr4oqp5q.png" alt=" " width="800" height="165"></a></p>

<p>Specific Prefix ECR Automaticaly create or You can create any repository.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8f30hpreog1uwckzxrml.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8f30hpreog1uwckzxrml.png" alt=" " width="800" height="258"></a></p>

<p>Whether to overwrite image tags or not can be set at the template creation stage.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F319nhn1kuprh9164741g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F319nhn1kuprh9164741g.png" alt=" " width="800" height="154"></a></p>

<h2>
  
  
  Try
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>docker buildx build -t test-auto-create --platform=linux/amd64 .

aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin xxxxxxxxxxxx.dkr.ecr.us-west-2.amazonaws.com

docker tag 3413614e55cd xxxxxxxxxxxx.dkr.ecr.us-west-2.amazonaws.com/test-auto-create:latest

docker push xxxxxxxxxxxx.dkr.ecr.us-west-2.amazonaws.com/test-auto-create:latest
</code></pre>

</div>



<p>I verified this by running the <code>aws ecr describe-repositories</code> command before and after. You should be able to see that the ECR is created.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>3a4,17
&gt;             "repositoryArn": "arn:aws:ecr:us-west-2:xxxxxxxxxxxx:repository/test-auto-create",
&gt;             "registryId": "xxxxxxxxxxxx",
&gt;             "repositoryName": "test-auto-create",
&gt;             "repositoryUri": "xxxxxxxxxxxx.dkr.ecr.us-west-2.amazonaws.com/test-auto-create",
&gt;             "createdAt": "2025-12-21T12:39:57.362000+09:00",
&gt;             "imageTagMutability": "IMMUTABLE",
&gt;             "imageScanningConfiguration": {
&gt;                 "scanOnPush": false
&gt;             },
&gt;             "encryptionConfiguration": {
&gt;                 "encryptionType": "AES256"
&gt;             }
&gt;         },
&gt;         {
</code></pre>

</div>



<h2>
  
  
  Summary
</h2>

<p>ECR is also automatically created in the pull-through cache, which should make it easier to create pull cache repositories from ECR public or Docker Hub.</p>

