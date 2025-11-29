---
Title: From missed flight to mission: building a 24-hour reminder service on AWS
Description: 
Author: Elmaur Dehni Namata
Date: 2025-11-29T21:19:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>A few months ago, my mom missed her flight. She’d been swamped all week, errands, and work. She thought the flight was later, got to the airport, it was too late her flight had gone. When she told me, I felt that gut drop of <em>“we could’ve avoided this.”</em> </p>

<p>This wasn’t just about building a reminder service. It was about my journey into DevOps learning AWS resources one by one, connecting them, and seeing how cloud skills can solve everyday problems.</p>

<p>From a missed flight to a working solution, this project shows that even small ideas can become powerful learning experiences.<br>
That one moment turned into this project.</p>

<h2>
  
  
  🛠️ stack
</h2>

<ul>
<li>
<strong>Data:</strong> Appointments saved in DynamoDB (name, date, time).
</li>
<li>
<strong>Compute:</strong> AWS Lambda scans appointments and decides who needs reminders.
</li>
<li>
<strong>Notify:</strong> Amazon SNS sends email/SMS notifications. </li>
</ul>

<h2>
  
  
  Step-by-Step Walkthrough
</h2>

<h3>
  
  
  Create the DynamoDB Table
</h3>

<ul>
<li>
<strong>Table name:</strong> <code>Appointment</code> </li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftkutd7m850e4lsd0z8ag.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftkutd7m850e4lsd0z8ag.png" alt=" " width="800" height="505"></a></p>

<ul>
<li>
<strong>Primary key:</strong> Partition key = <code>Name</code> (String)
</li>
<li>
<strong>Attributes:</strong>

<ul>
<li>
<code>Name</code>: e.g., “Dentist Visit”
</li>
<li>
<code>Date</code>: ISO string, e.g., “2025-12-29” </li>
</ul>


</li>

</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fifnwwg6y1a1kc3l9mzq4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fifnwwg6y1a1kc3l9mzq4.png" alt=" " width="800" height="359"></a></p>

<h3>
  
  
  Create the SNS Topic and Subscribe
</h3>

<ul>
<li>
<strong>Topic name:</strong> <code>AppointmentReminders</code> (Standard) </li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F58ldbzzi4gl6y761hyp7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F58ldbzzi4gl6y761hyp7.png" alt=" " width="800" height="433"></a></p>

<ul>
<li>Copy the <strong>Topic ARN</strong> — you’ll need this for Lambda.
</li>
<li>Add a subscription:

<ul>
<li>Protocol: Email (or SMS)
</li>
<li>Endpoint: Your email address
</li>
</ul>


</li>

<li>Confirm the subscription via the AWS email link.
</li>

</ul>

<h2>
  
  
  Create the Lambda Function (Node.js)
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdueeim8gwpv8di52oiek.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdueeim8gwpv8di52oiek.png" alt=" " width="800" height="486"></a></p>

<ul>
<li>Runtime:** Node.js 18.x
</li>
<li>Handler:** <code>index.handler</code>
</li>
<li>File name:** `index.js</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0kpecryj3gayjqvksmus.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0kpecryj3gayjqvksmus.png" alt=" " width="800" height="420"></a></p>

<h2>
  
  
  Give Lambda the Right IAM Permissions
</h2>

<p>Attach a minimal inline policy to your Lambda’s execution role:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdalc45h1dbzpzb9f77f6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdalc45h1dbzpzb9f77f6.png" alt=" " width="773" height="536"></a></p>

<p>** choosing the specific action that the resources in DynamoDB will need.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyhwh7v186b9dndoyefh3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyhwh7v186b9dndoyefh3.png" alt=" " width="800" height="480"></a></p>

<p>** And for SNS</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F148fv4e4vtqqrl7vaptj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F148fv4e4vtqqrl7vaptj.png" alt=" " width="800" height="427"></a></p>

<p>** After that is done give a to the policy.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyhw4e91ekqlvoanrxcei.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyhw4e91ekqlvoanrxcei.png" alt=" " width="800" height="433"></a></p>

<p>** Review and click create.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcz8ubvjiuzpvu06dtp0i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcz8ubvjiuzpvu06dtp0i.png" alt=" " width="800" height="332"></a></p>

<h2>
  
  
  Add a test item in DynamoDB:
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzcz5u6s8byd2c8lol794.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzcz5u6s8byd2c8lol794.png" alt=" " width="800" height="359"></a></p>

<h2>
  
  
  Run Lambda → Test event
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F13v7k8clp98hyes1l4l7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F13v7k8clp98hyes1l4l7.png" alt=" " width="800" height="358"></a></p>

<h2>
  
  
  <strong>Result</strong> I tested with a dentist appointment and received an email reminder no more last-minute panics.   .
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0hjmr3n540fy5wufzz6h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0hjmr3n540fy5wufzz6h.png" alt=" " width="800" height="1780"></a></p>

<h2>
  
  
  🚀 what I learned
</h2>

<p>what this project solve Forgetting is easy; systems aren’t.<br>
24-hour timing gives you enough time to prepare.<br>
The missed flight hurt, but it sparked something useful: a small, dependable system that gives you a 24-hour heads up for what matters. You don’t need a huge platform just a table, a function, a topic, and a schedule.</p>

<p>** Cloud skills solving everyday problems.**<br>
This is my Devops/cloud engineering journey.</p>

