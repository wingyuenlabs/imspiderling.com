---
Title: [AWS] DevTools Evangelism: CodeDeploy Edition
Description: 
Author: Nao San
Date: 2025-11-30T21:28:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>This article is a machine translation of the contents of the following URL, which I wrote in Japanese:</p>

<p><a href="https://qiita.com/Nana_777/items/c7ebc842c4557f8d811d" rel="noopener noreferrer">https://qiita.com/Nana_777/items/c7ebc842c4557f8d811d</a></p>

<h2>
  
  
  Introduction
</h2>

<p>This is the third post in the Japan AWS Top Engineers Advent Calendar 2025.</p>

<p>In the previous post, we discussed AWS CodeCommit, which manages system assets. This time, we'll discuss CodeDeploy, which is used to deploy source code and system configurations.</p>

<p>↓ Click here for the Japan AWS Top Engineers Advent Calendar 2025</p>

<p><a href="https://qiita.com/advent-calendar/2025/aws-top-engineers" rel="noopener noreferrer">https://qiita.com/advent-calendar/2025/aws-top-engineers</a></p>

<p>↓ Previous AWS CodeCommit post</p>

<p><a href="https://dev.to/aws-builders/aws-devtools-evangelism-codecommit-edition-43e">https://dev.to/aws-builders/aws-devtools-evangelism-codecommit-edition-43e</a></p>

<h2>
  
  
  What is AWS CodeDeploy?
</h2>

<p>CodeDeploy is a service that automates deployments for AWS services such as Amazon EC2 and Lambda.<br>
AWS CodeDeploy enables safer deployments, making it easier to implement deployment strategies such as canary and blue-green deployments, which involve gradual deployments.</p>
<h2>
  
  
  Deployment Strategy Examples
</h2>
<h3>
  
  
  Blue-Green Deployment
</h3>

<p>The existing version and the new version are run simultaneously.<br>
100% of traffic is sent to the new version while its operation is tested. If there are no problems with the new version, 100% of traffic is set to flow to the new version.<br>
The new version's operation is monitored for a set period of time, and if there are any problems, traffic is returned to the existing version. If there are no problems, the existing version is discontinued.<br>
While this allows for quick rollback, if there is a problem with the new version, it will temporarily affect 100% of traffic.</p>
<h3>
  
  
  Canary Deployment
</h3>

<p>This type of deployment is similar to blue-green deployment, but instead of switching over 100% of traffic, traffic is gradually shifted at a fixed rate, such as 10%.<br>
This method limits the scope of impact if there is a problem with the new version, but it takes a relatively long time for the deployment to complete (when 100% of traffic has been shifted to the new version).<br>
Example: If you shift 10% of traffic per minute, it will take a total of 10 minutes.</p>

<p>↓I asked AI to draw a diagram like this, and it came up with this image.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvmwqx85ziuc7enonod11.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvmwqx85ziuc7enonod11.png" alt="image.png" width="800" height="436"></a></p>
<h3>
  
  
  Linear Deployment
</h3>

<p>This is also similar to blue-green deployment, but it initially shifts a small portion of traffic, say 10%, to the new version. If no issues are found within a certain period of time, the remaining 90% is shifted to the new version.</p>

<p>This method limits the impact of any issues with the new version and allows for faster deployment completion than linear deployment. However, because only a small portion of traffic is initially tested, there is a risk of missing issues due to unusual traffic patterns with a low probability of occurrence.</p>
<h2>
  
  
  AWS CodeDeploy Components
</h2>
<h3>
  
  
  Applications
</h3>
<h4>
  
  
  Configuration Example
</h4>

<p>In the AWS CodeDeploy console, select "Applications" from the left menu.<br>
Select Create Application to display the application creation screen.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnrm5b8sa9c8vczpzayqz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnrm5b8sa9c8vczpzayqz.png" alt="image.png" width="709" height="211"></a><br>
When creating an application, you set the application name and computing platform.<br>
You can choose the application platform from "EC2/On-Premises," "AWS Lambda," or "Amazon ECS."<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv3qqoi2ikth7emi3zq1b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv3qqoi2ikth7emi3zq1b.png" alt="image.png" width="733" height="481"></a></p>
<h3>
  
  
  Deployment Group
</h3>
<h4>
  
  
  Configuration Example
</h4>

<p>After creating an application, create a deployment group to associate with it.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbaqccxym8nuk39rjp9e4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbaqccxym8nuk39rjp9e4.png" alt="image.png" width="704" height="619"></a><br>
When creating a deployment group, you specify the deployment group name, service role, and deployment settings.<br>
For the service role, select a previously created role, but select a role that has permissions to operate the service to be deployed.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj37rnq6f0nncbkbzwlbq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj37rnq6f0nncbkbzwlbq.png" alt="image.png" width="737" height="596"></a><br>
For the deployment settings, you can choose the pre-defined "All At Once" setting, or choose linear or canary deployment, which are performed at a preset percentage.<br>
*In this example, we chose linear deployment, which shifts 10 percent of traffic to the new version every minute.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fon27x4vxwt0erx69hlqf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fon27x4vxwt0erx69hlqf.png" alt="image.png" width="740" height="198"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv7398o8g1wu28lb4w3ui.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv7398o8g1wu28lb4w3ui.png" alt="image.png" width="726" height="325"></a><br>
There are no appropriate options for the percentage of options. If you don't have a detailed deployment method, you can create a deployment configuration to set detailed deployment methods (how often and at what rate deployment should proceed).<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnir2uhdzqpnjtq4c4t30.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnir2uhdzqpnjtq4c4t30.png" alt="image.png" width="559" height="585"></a><br>
Detailed settings are optional, but they allow you to configure alarm settings and whether or not to enable rollback.<br>
You can set this if you want to rollback if an updated Lambda function causes an error during deployment.<br>
For example, you can set a CloudWatch alarm to fire when a Lambda function error occurs, and then set that alarm as an alarm on this deployment group screen.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxn9d5kplp2j6b7s17cm5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxn9d5kplp2j6b7s17cm5.png" alt="image.png" width="745" height="550"></a></p>
<h2>
  
  
  Deployment
</h2>

<p>Deployment can be performed using the AWS console, but in practice, a deployment strategy is often implemented using IaC code.<br>
This section explains deployment settings in the AWS console and in IaC code.</p>
<h3>
  
  
  Deployment in the AWS Console
</h3>
<h4>
  
  
  Creating a Lambda Function to Deploy
</h4>

<p>Create a Lambda function to deploy and an alias for the new version.<br>
The Lambda function created as the existing version simply returns the message "Hello from Lambda!".<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0f7kzycuos87m50o0rzh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0f7kzycuos87m50o0rzh.png" alt="image.png" width="428" height="188"></a></p>
<h4>
  
  
  Creating a Lambda Function Version
</h4>

<p>Create a version of the current Lambda function.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgwmmszyilz6mn42ir6vf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgwmmszyilz6mn42ir6vf.png" alt="image.png" width="800" height="260"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7nyhoogag5dbk7dopi8b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7nyhoogag5dbk7dopi8b.png" alt="image.png" width="751" height="246"></a></p>

<p>*The version number is "3" because I tested it a little in advance.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fskn8id1kkn95232cysvp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fskn8id1kkn95232cysvp.png" alt="image.png" width="800" height="195"></a></p>
<h4>
  
  
  Create an Alias
</h4>

<p>Create an alias to control traffic for each version.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0qeu4ljp9dif6us3c3o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0qeu4ljp9dif6us3c3o.png" alt="image.png" width="800" height="369"></a></p>
<h4>
  
  
  Updating the Lambda Function
</h4>

<p>After updating the code, create a new version of this function.<br>
The code sends the message "Hello from Lambda V2!"<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm3al0in2yhacnifyu1yn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm3al0in2yhacnifyu1yn.png" alt="image.png" width="422" height="207"></a></p>
<h4>
  
  
  Creating a New Version
</h4>

<p>I created a new version (the version is 5 because I tested it outside of validation).<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp2av6uzo5z2tqrtgkc84.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp2av6uzo5z2tqrtgkc84.png" alt="image.png" width="800" height="173"></a></p>
<h4>
  
  
  Deploying with CodeDeploy
</h4>

<p>From the details screen of a deployment group you've already created, click "Create Deployment."<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdu9bo835o60z1lpbo2n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdu9bo835o60z1lpbo2n.png" alt="image.png" width="696" height="392"></a></p>

<p>:::note warn<br>
Deployment Group Role<br>
Since this deployment involves Lambda, we'll add a policy for Lambda operations to the deployment group.<br>
↓Add a policy for Lambda operations to this role.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcfcp02dii04sysl8l30i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcfcp02dii04sysl8l30i.png" alt="image.png" width="656" height="436"></a><br>
:::</p>

<p>In the deployment settings, specify the Lambda function name and alias, as well as the current and new versions of the Lambda function to which traffic will be migrated.<br>
In this example, we will gradually migrate traffic when upgrading the Lambda function from version 3 to version 5.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3yglr4vmobml6366h556.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3yglr4vmobml6366h556.png" alt="image.png" width="713" height="723"></a></p>

<p>↓This configuration<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>version: 0.0
Resources:
- myLambdaFunction:
Type: AWS::Lambda::Function
Properties:
Name: "qiita-naolambda-2025"
Alias: "qiitaLambdaDeployTest"
CurrentVersion: "3"
TargetVersion: "5"
</code></pre>

</div>



<p>:::note warn<br>
Deployment timing<br>
When you create a deployment from the AWS console, the deployment begins immediately.<br>
:::</p>

<p>You can monitor the deployment progress on the AWS CodeDeploy screen. In this example, we configured the deployment group to distribute traffic at 10% per minute, so you can see that it's progressing according to that setting.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd6e8r5bsbhgb1s5qzu5n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd6e8r5bsbhgb1s5qzu5n.png" alt="image.png" width="658" height="439"></a></p>

<p>You can also check the distribution of traffic to each version on the AWS Lambda alias screen.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdk00283v7seg27xxagn9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdk00283v7seg27xxagn9.png" alt="image.png" width="800" height="213"></a></p>

<p>When the deployment completes, all traffic will be assigned to the new version, and the deployment will be complete.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsmkv6cw2wquwohtmlpoi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsmkv6cw2wquwohtmlpoi.png" alt="image.png" width="800" height="207"></a></p>

<h3>
  
  
  Deploying with IaC Code
</h3>

<p>Deployment is possible using CodeDeploy from the AWS console, but you can also write deployment settings in IaC code.<br>
In actual system operations, configuration management and CI/CD are often used in IaC code to automate the reflection of system configuration and the execution of deployment strategies.<br>
In my previous article, I also described implementing a deployment strategy using CDK and CodeDeploy, so please take a look.<br>
*If I have time, I will add more information to this article.</p>

<p>↓ Previous deployment strategy articles</p>

<p><a href="https://qiita.com/Nana_777/items/4627fc27713217ca93db" rel="noopener noreferrer">https://qiita.com/Nana_777/items/4627fc27713217ca93db</a></p>

<h2>
  
  
  Finally
</h2>

<p>AWS CodeDeploy facilitates gradual deployment and quick, automated rollback in the event of issues.<br>
While the appropriate deployment strategy will vary depending on your project, we encourage you to learn how to use AWS CodeDeploy to adapt to any deployment strategy.</p>

<h2>
  
  
  Reference
</h2>

<p>↓ Official AWS CodeDeploy documentation</p>

<p><a href="https://docs.aws.amazon.com/ja_jp/codedeploy/latest/userguide/welcome.html" rel="noopener noreferrer">https://docs.aws.amazon.com/ja_jp/codedeploy/latest/userguide/welcome.html</a></p>

<p>↓ AWS CodeDeploy BlackBelt documentation (Japanese)</p>

<p><a href="https://pages.awscloud.com/rs/112-TZM-766/images/20210126_BlackBelt_CodeDeploy.pdf" rel="noopener noreferrer">https://pages.awscloud.com/rs/112-TZM-766/images/20210126_BlackBelt_CodeDeploy.pdf</a></p>

