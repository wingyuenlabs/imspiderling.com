---
Title: New pricing model for CloudFront
Description: 
Author: Jens Båvenmark
Date: 2025-11-18T21:58:37.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwsvo65tkgsiayexx85ut.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwsvo65tkgsiayexx85ut.png" alt="Fixed-Price Plans for CloudFront" width="700" height="350"></a></p>

<p>AWS just released a new pricing model for CloudFront. Now you can pay a fixed price.</p>

<p>As most of you know, calculating the cost in AWS can be hard, especially on resources where traffic is the cost driver (like CloudFront). So this change will make it a lot easier.</p>

<p>So how will this work then?</p>

<p>There will be different plans with varying costs that include CloudFront and other services in the pricing. You will select the plan you want when deploying your CloudFront Distribution.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1osqkzd6kujdhw0qa3p5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1osqkzd6kujdhw0qa3p5.png" alt="Flat-Rate security and delivery plans" width="700" height="574"></a></p>

<p>For current distributions, you can “migrate” them to the plan of your choice by selecting to switch to a plan for your distribution in the console (right now, the console is the only place to set plans). If the current deployment configuration matches what the plan provides, you will be able to use that plan.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxz1a2strcttj2umpebxl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxz1a2strcttj2umpebxl.png" alt="Migrate to plan" width="700" height="190"></a></p>

<h2>
  
  
  Included Services
</h2>

<p>The different plans include these services, but not all plans include all services. Some services are for the higher plans (see the image below for details about each plan).</p>

<ul>
<li>CloudFront CDN</li>
<li>WAF and DDoS protection</li>
<li>DNS (Route 53)</li>
<li>TLS Certificate (ACM)</li>
<li>CloudWatch Log Ingestion (Storage costs for log still apply)</li>
<li>Serverless Edge Compute (CloudFront Functions, not Lambda@Edge)</li>
<li>Bot Management (Business plan and above)</li>
</ul>

<p>You will also get S3 credits that can be used across all your accounts S3 costs.</p>

<p>And data transfer to CloudFront is automatically waived.</p>

<h2>
  
  
  The Different Plans
</h2>

<p>There are four different plans, as well as the old pay-as-you-go pricing.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9o3jpd79434caouff52q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9o3jpd79434caouff52q.png" alt="The different plans" width="700" height="520"></a></p>

<p>As you see in this image, you can easily see what each plan includes. All higher plans include the services of the lower plans.</p>

<h3>
  
  
  Free Plan
</h3>

<p>This is, in my opinion, the best part of these pricing plans. The Free plan will make it possible for people learning AWS or running small workloads to use CloudFront without having to worry about getting a large bill at the end of the month.</p>

<p>And what you get in the free tier is not a bad setup for many small companies and PoC sites. The included WAF will make it easy to secure you distributions and you can add up to 5 different rules.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmaap60rgia7asb04lb2k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmaap60rgia7asb04lb2k.png" alt="The 5 free waf rules for free plan" width="700" height="484"></a></p>

<p>Additionally, 5 GB of S3 storage is available for free per CloudFront Distribution. But before you start planning on removing your S3 bill by deploying 100 Free distributions, you should know that every account will be limited to three Free Plan distributions.</p>

<h3>
  
  
  Pro Plan
</h3>

<p>For $15/month, this plan will likely be the most popular choice for companies with medium traffic, as it supports 10 million requests / 50 TB per month, and includes WAF protection for your WordPress, PHP, and SQL databases, as well as logging support.</p>

<h3>
  
  
  Business Plan
</h3>

<p>Jumping up in cost with $200/month, the Business Plan will not be for everyone, but will be a must for more heavily trafficked sites (125M requests / 50 TB per month). It also comes with Bot management, more advanced DDoS protection, and the first plan to support Private Origins in your VPC (a great security feature where you do not need to have your ALB in public subnets and have public IPs).</p>

<h3>
  
  
  Premium Plan
</h3>

<p>For your sites with extremely high traffic (500M requests / 50 TB per month) or where you require even greater security (like regex-based WAF filtering), the Premium Plan can be yours for only $1000/month.</p>

<p>With it, you will have features such as automatic origin failover, mTLS for end-users, and high-speed private origin routing.</p>

<h3>
  
  
  Pay-as-you-go Pricing
</h3>

<p>If you don't want to choose a plan, you can continue to use Pay-as-you-go pricing. Current distributions will continue on this “plan” if not “migrated” to one of the other plans.</p>

<p>So, when would you want to use this “plan”? If your distribution has a really heavy load so that the Premium Plan is not enough (over 500M requests or 50 TB per month), or when you need a special setting on the services that is not included in the other plans, like Lambda@Edge.</p>

<p>But if you don't need that, you should select a plan that matches your traffic.</p>

<h2>
  
  
  But what about when?
</h2>

<p>So changes are scary, and for many technicians, there will be many questions, especially when deciding on migrating current distributions to a plan. So I will answer the questions I can here.</p>

<h3>
  
  
  What happens if I exceed the usage allowance?
</h3>

<p>If your usage exceeds the limit of your plan, your distribution will continue to work and receive traffic just as before. But you may experience reduced performance. You will not be charged anything more than your set monthly cost.</p>

<h3>
  
  
  Can I change plans whenever?
</h3>

<p>You can deploy a new distribution or “migrate” an existing distribution and select a plan at any time, and the cost of that plan will be applied to the current billing cycle.</p>

<p>If you want to change plan (downgrade or upgrade), the change will take effect in the next billing cycle.</p>

<p>If you cancel a plan, it will take effect at the beginning of the next billing cycle, where the distribution will revert to pay-as-you-go pricing.</p>

<h3>
  
  
  Can I have as many plans as I want?
</h3>

<p>From the start, you will be able to have three free plans and 100 paid plans per account.</p>

<p>I believe the limit for paid plans can be increased after several customers have requested it, but for now, this is the limit.</p>

<h3>
  
  
  Can I monitor if I am exceeding my plan's limit?
</h3>

<p>Yes, there will be CloudWatch metrics for this, and AWS will also send you an email if you are approaching your plan's limit.</p>

<h3>
  
  
  Can I use this with Amplify?
</h3>

<p>No, not right now, Amplify has its own pricing model.</p>

<h3>
  
  
  Can I move a Plan to another distribution?
</h3>

<p>No. A plan is connected to a distribution and can not be moved to another distribution.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>This is a significant improvement from AWS, making CloudFront costs easier to predict.</p>

<p>Of course, there are some “features” I would have liked to have, like the possibility to manage this with IaC, a solution for handling Blue/Green deployments without having to have two different plans, and the possibility to switch to a pay-as-you-go model whenever you want from a set plan. But I believe these will be implemented when feedback comes back to AWS after the release, and they see what limitations the customers are facing.</p>

<p>All information in this blog post was the information available at the time of release on the 17th of November 2025. If something changes, I will try to update this blog post, but as always, check the <a href="https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html" rel="noopener noreferrer">AWS documentation</a> for the latest information.</p>

