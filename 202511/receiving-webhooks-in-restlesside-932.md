---
Title: Receiving Webhooks in RestlessIDE
Description: 
Author: Mike
Date: 2025-11-12T21:15:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>So I was speaking with the Backend Standup guys the other day (as one does) and one of them explained a problem they were having. They were attempting to do a  Stripe integration but were running into problems with the webhook portion.</p>

<p>For those who need a reminder, a webhook is basically a callback from an online service into your application. For example, with Stripe (a payment processing service) you can set up subscriptions with your customers, where they sign up and pay you a certain amount each month. To handle this, Stripe generates an invoice automatically, and then charges them for it, again automatically. In order to give your application some visibility into this process, you can configure Stripe to hit a specified endpoint on your application with payment updates. This also applies to things like their card being declined or expired, or them canceling the subscription through Stripe directly. All very helpful things to be notified about.</p>

<p>Anyway, my friend was having an issue because he was running his dev server on his local machine, which doesn’t have an external hostname, and thus didn’t have a URL he could tell Stripe to hit. What to do?</p>

<p>If he were using RestlessIDE (the future <a href="https://restlesside.com" rel="noopener noreferrer">best web-based development environment</a>!) this would be a non-issue. Here are the steps to set it up:</p>

<h3>
  
  
  1. Create your workspace with either Port 3000 or Port 3030 set to HTTP/HTTPS.
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdp0lybp7bqpjd6kixit7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdp0lybp7bqpjd6kixit7.png" width="800" height="239"></a></p>

<p>It’s probably also a good idea to use SSL. Setting this up will tell your container to open up the port for external access over HTTP, which will be needed for the webhook.</p>

<h3>
  
  
  2. Set up your application to listen on the port you chose.
</h3>

<p>The details will vary a bit depending on the specifics of your programming language/environment. In NodeJS, we typically just add a <code>PORT=3000</code> to our <code>.env</code> and the corresponding code in the application to look there. If you’re using PHP, RestlessIDE automatically configures your web server to listen on Port 3000, so you should be all set.</p>

<h3>
  
  
  3. Configure the service you’re working with to hit your RestlessIDE application.
</h3>

<p>Your Workspace will be working on a particular host name, in the format:</p>

<p><code>https://&lt;workspace&gt;-&lt;username&gt;.hosts.restlesside.com</code></p>

<p>This is where you edit your code. To access your running application (once you start it, that is!) you can just go to that same URL with Port 3000 like this:</p>

<p><code>https://&lt;workspace&gt;-&lt;username&gt;.hosts.restlesside.com:3000</code></p>

<p>You’ll need to set up your application to receive the webhook, which often involves a handshake step as you’ll need to verify that incoming request is indeed coming from the service you’re expecting. You’ll need to create an endpoint for this endpoint, let’s say <code>/webhooks/receive</code>. Once it’s all configured you’ll need to register this webhook somewhere in service’s UI. Just enter:</p>

<p><code>https://&lt;workspace&gt;-&lt;username&gt;.hosts.restlesside.com:3000/webhooks/receive</code></p>

<p>in the appropriate spot and you should be good to go.</p>

<p>See, things get a lot easier if you’re coding the web <em>from the web</em>. If you’re not convinced, here are <a href="https://restlesside.com/#reasons" rel="noopener noreferrer">10 more reasons why coding in a browser is the best</a>.</p>

<p>Til next time.</p>

<p>(Photo by <a href="https://unsplash.com/@steve_j?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText" rel="noopener noreferrer">Steve Johnson</a> on <a href="https://unsplash.com/photos/gray-scale-photo-of-door-lever-6sB8gMRlEAU?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText" rel="noopener noreferrer">Unsplash)</a></p>

