---
Title: Workload Discovery & free alternatives
Description: 
Author: Wakeup Flower
Date: 2025-10-07T21:36:03.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Workload Discovery on AWS</strong> is <strong>not free</strong> to use. It incurs costs based on the AWS resources it provisions during deployment.</p>

<h3>
  
  
  💰 Estimated Costs
</h3>

<p>For a <strong>single-instance deployment</strong> in the <strong>US East (N. Virginia)</strong> region, the estimated monthly cost is approximately <strong>$425.19 USD</strong>. This includes services such as Amazon Neptune, Amazon OpenSearch Service, AWS AppSync, Amazon VPC (NAT Gateway), AWS Config, and Amazon ECS (AWS Fargate Task). These services are billed on an hourly or per-resource basis, depending on the specific service ([AWS Documentation][1]).</p>

<p>If you opt for a <strong>multi-instance deployment</strong> to enhance availability, the estimated monthly cost increases to around <strong>$699.67 USD</strong></p>




<h2>
  
  
  <strong>1. What Workload Discovery is</strong>
</h2>

<p>Workload Discovery on AWS is <strong>not just a simple query tool</strong> — it’s a full <strong>architecture and dependency mapping solution</strong>.<br>
Its purpose is to automatically:</p>

<ul>
<li>Discover all workloads across <strong>accounts and regions</strong>.</li>
<li>Map <strong>relationships</strong> between resources.</li>
<li>Build <strong>architecture diagrams</strong> showing dependencies.</li>
</ul>

<p>That’s far more than a basic inventory. It’s a <strong>data processing and visualization platform</strong>.</p>




<h2>
  
  
  <strong>2. Why it needs extra AWS resources</strong>
</h2>

<p>To do this, Workload Discovery <strong>needs infrastructure to store, process, and display data</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Amazon Neptune</strong></td>
<td>Stores the discovered resources and their relationships as a graph database.</td>
</tr>
<tr>
<td><strong>Amazon OpenSearch Service</strong></td>
<td>Indexes and enables search of resource data, and provides visual exploration tools.</td>
</tr>
<tr>
<td><strong>AWS AppSync</strong></td>
<td>Acts as the API layer for querying workload data.</td>
</tr>
<tr>
<td><strong>Amazon ECS with AWS Fargate</strong></td>
<td>Runs the application that coordinates discovery and visualization.</td>
</tr>
<tr>
<td><strong>AWS Config</strong></td>
<td>Continuously collects inventory data about resources.</td>
</tr>
<tr>
<td><strong>VPC, networking components</strong></td>
<td>Secure connectivity for all these services.</td>
</tr>
</tbody>
</table></div>

<p>These are <strong>all AWS services that must be deployed</strong> in your account for Workload Discovery to work.</p>

<p>So it’s not simply “reading what exists” — it’s <strong>collecting, storing, and processing</strong> data across accounts and regions. That requires <strong>actual AWS resources</strong> running continuously.</p>

<p>💡 <strong>Analogy</strong>:<br>
Think of Workload Discovery like Google Maps for your AWS infrastructure. But Google Maps doesn’t just “look” at your roads — it builds a <strong>map database, visualization tools, and APIs</strong> to present the information. Those underlying systems cost money, and AWS must run similar infrastructure for Workload Discovery.</p>

<h2>
  
  
  <strong>1. AWS Resource Groups + Tag Editor (Free)</strong>
</h2>

<ul>
<li>
<strong>What it does:</strong> Lets you view resources grouped by tags, regions, or resource type.</li>
<li>
<p><strong>How to use:</strong></p>

<ul>
<li>Go to the <strong>AWS Management Console → Resource Groups</strong>.</li>
<li>Use <strong>Tag Editor</strong> to search across regions and accounts (requires IAM permissions).</li>
</ul>


</li>

<li><p><strong>Pros:</strong> Free, built into AWS, works across multiple resource types.</p></li>

<li><p><strong>Cons:</strong> Doesn’t automatically draw architecture diagrams or relationships.</p></li>

</ul>




<h2>
  
  
  <strong>2. AWS Config (Some features are free)</strong>
</h2>

<ul>
<li>
<strong>What it does:</strong> Tracks configuration changes and relationships between AWS resources.</li>
<li>
<strong>How to use:</strong> Enable AWS Config in your account(s), optionally aggregate data across accounts/regions.</li>
<li>
<strong>Pros:</strong> Can build a relationship graph for supported resources.</li>
<li>
<strong>Cons:</strong> AWS Config costs can add up for many resources. Relationship mapping is limited compared to Workload Discovery.</li>
<li>
<strong>Cost:</strong> AWS Config has a free tier but charges apply for configuration items and rules.</li>
</ul>




<h2>
  
  
  <strong>3. AWS Perspective (Free solution to deploy)</strong>
</h2>

<ul>
<li>
<strong>What it does:</strong> AWS Perspective is a free AWS Solution that <strong>visualizes your AWS cloud architecture</strong>.</li>
<li>
<p><strong>How to use:</strong></p>

<ul>
<li>Deploy AWS Perspective (CloudFormation template).</li>
<li>Connect your AWS accounts.</li>
<li>Automatically visualize resources and relationships.</li>
</ul>


</li>

<li><p><strong>Pros:</strong> Free to use (just pay for underlying AWS resources like storage). Creates architecture diagrams.</p></li>

<li><p><strong>Cons:</strong> Requires deployment and some setup. Doesn’t have the same depth as Workload Discovery for workload mapping.</p></li>

<li><p><strong>Cost:</strong> Mostly the cost of S3 storage, CloudWatch logs, etc., which is far lower than Workload Discovery.</p></li>

</ul>




<h2>
  
  
  <strong>4. Manual Tools</strong>
</h2>

<ul>
<li>
<p><strong>CloudMapper</strong> (open-source tool from Duo Labs)</p>

<ul>
<li>Can map AWS accounts and produce diagrams.</li>
<li>Requires access keys for your accounts.</li>
<li>Works across regions/accounts.</li>
<li>
<strong>Free</strong>, but needs manual setup.</li>
</ul>


</li>

<li>

<p><strong>Terraform / CloudFormation + Visualization tools</strong></p>

<ul>
<li>If your workloads are deployed via infrastructure-as-code, you can use tools like <code>Graphviz</code> or <code>cdktf</code> to generate diagrams.</li>
</ul>


</li>

</ul>




<h3>
  
  
  🔍 Recommendation
</h3>

<p>If your goal is <strong>a free or low-cost architecture view</strong> across accounts, I’d recommend:</p>

<ul>
<li>
<strong>First:</strong> AWS Resource Groups + Tag Editor to inventory resources.</li>
<li>
<strong>Then:</strong> AWS Perspective for a visual map of resources and relationships.
It’s <strong>free to deploy</strong> and far cheaper than Workload Discovery.</li>
</ul>

