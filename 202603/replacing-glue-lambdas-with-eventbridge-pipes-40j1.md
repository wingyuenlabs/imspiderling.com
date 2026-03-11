---
Title: Replacing Glue Lambdas with EventBridge Pipes
Description: 
Author: Benjamin Ajewole
Date: 2026-03-11T21:41:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Serverless architectures on AWS often rely on Lambda for more than business logic. In many systems, Lambda functions are used as intermediaries between event sources and downstream services, handling simple filtering, transformation, or event forwarding. Although this approach works, it can introduce unnecessary compute, more infrastructure to manage, and added complexity. EventBridge Pipes offers a more streamlined alternative by connecting sources and targets directly, reducing the need for intermediary Lambda functions.</p>

<h2>
  
  
  The Traditional Pattern
</h2>

<p>A common approach starts with a source such as DynamoDB Streams capturing changes made to a table, after which a Lambda function consumes the stream record and forwards it to another AWS service such as SQS, EventBridge, or Step Functions. This design is flexible and familiar, but it often places Lambda in the middle of the flow even when the function is doing little more than forwarding records after light filtering or reshaping. In these cases, Lambda is being used more as infrastructure plumbing than as a true execution layer for application logic.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fupwfwmgvfneq6489tb1v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fupwfwmgvfneq6489tb1v.png" alt=" " width="800" height="153"></a><br>
<em>In the traditional design, a new order written to DynamoDB triggers a Lambda function through DynamoDB Streams. Lambda then filters, transforms, and forwards the event to SQS or Step Functions.</em></p>

<h2>
  
  
  How EventBridge Pipes Simplifies the Architecture
</h2>

<p>EventBridge Pipes provides a more direct way to move events from a source to a target without placing a Lambda function in the middle. Instead of writing and maintaining code whose only job is to receive records from DynamoDB Streams and forward them elsewhere, you can connect the stream directly to supported targets such as EventBridge, SQS, or Step Functions. This moves filtering, transformation, and routing from custom code into a managed AWS integration, reducing the number of moving parts and making the event flow easier to understand.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdu6ypk9pjpx8iqaot7n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdu6ypk9pjpx8iqaot7n.png" alt=" " width="800" height="129"></a><br>
<em>In the EventBridge Pipes design, a new order written to DynamoDB flows from DynamoDB Streams into a pipe, which filters, transforms, and routes the event directly to SQS or Step Functions.</em></p>

<h2>
  
  
  Advantages of Using EventBridge Pipes
</h2>

<ul>
<li>
<strong>Lower operational overhead and potential latency reduction</strong> With fewer Lambda functions in the middle, there are fewer deployments, permissions, logs, and runtime concerns to manage. In some cases, this can also improve end-to-end performance by removing an extra compute hop and avoiding Lambda cold starts.</li>
<li>
<strong>Reduces intermediary Lambda functions</strong>
 EventBridge Pipes can replace Lambda functions that exist only to move events from a source to a target.</li>
<li>
<strong>Built-in filtering and transformation</strong>
 Pipes can filter relevant events and reshape payloads before they reach the target, reducing the need for custom code.</li>
<li>
<strong>Cleaner event flow</strong>
 By creating a more direct connection between source and target, EventBridge Pipes makes the architecture easier to understand, maintain, and evolve.</li>
</ul>

<h2>
  
  
  When to Use EventBridge Pipes Instead of Lambda
</h2>

<p>Use EventBridge Pipes instead of Lambda when your main goal is to move events from a supported source to a supported target with minimal custom processing. It is a strong fit when a Lambda function would otherwise exist only to forward, lightly filter, or reshape events, because Pipes already supports point-to-point routing, filtering, input transformation, and optional enrichment.</p>

<h2>
  
  
  When Lambda Is Still the Better Choice
</h2>

<p>Lambda is still the better choice when the event requires complex business logic, external API calls, custom validation, enrichment logic that cannot be expressed cleanly through Pipes, or application-specific decision making. While EventBridge Pipes supports enrichment with services such as Lambda, API destinations, API Gateway, and Step Functions Express workflows, Pipes itself is best suited to managed integration rather than general-purpose compute.</p>

<h2>
  
  
  Conclusion
</h2>

<p>EventBridge Pipes is most valuable when the problem is integration, not computation. If a Lambda function exists only to consume an event, discard some records, reshape others, and forward the rest, that function is often a candidate for replacement. By moving filtering, optional enrichment, and transformation into a managed source-to-target service, EventBridge Pipes can reduce infrastructure overhead and make event-driven architectures easier to reason about. Lambda still has a clear place when real application logic is required, but for simple routing flows, Pipes is often the cleaner design choice.</p>

