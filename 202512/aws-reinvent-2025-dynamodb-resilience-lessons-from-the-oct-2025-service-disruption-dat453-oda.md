---
Title: AWS re:Invent 2025 - DynamoDB: Resilience & lessons from the Oct 2025 service disruption (DAT453)
Description: 
Author: Kazuya
Date: 2025-12-08T21:05:35.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 - DynamoDB: Resilience &amp; lessons from the Oct 2025 service disruption (DAT453)</strong></p>

<blockquote>
<p>In this video, Craig Howard from the DynamoDB team provides a detailed technical postmortem of the October 20th US-East-1 outage where DynamoDB returned zero IPs from DNS. He explains how DynamoDB's DNS management system works using Route 53, including a locking mechanism implemented via DNS text records and a tree-based plan deployment system. The outage was caused by a race condition between three enactors where unusual lock acquisition delays led to installing a stale plan that was immediately cleaned up, leaving DNS in an inconsistent state. Howard shares key lessons learned including using Amazon Q for log analysis and formal modeling validation, the importance of cellularization strategies, distinguishing between fast and slow update paths, leveraging races for resilience through request hedging, and thoughtfully evaluating dependencies. He emphasizes converting errors to frozen states and designing systems around things that won't change: security, durability, availability, and latency.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/YZUNNzLDWb8">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fomcm4fmejag512524jn9.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=20" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5vyju674t3exu8exrz3b.jpg" alt="Thumbnail 20" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=40" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs12ruigt1g8wbc16ouxx.jpg" alt="Thumbnail 40" width="800" height="450"></a></p>

<h3>
  
  
  DynamoDB's Unchanging Priorities and the October 20th US East 1 Outage
</h3>

<p>First comes security, then durability, then availability, and then latency. Like other AWS services, these are DynamoDB's priorities in this order. Years ago, Jeff Bezos was interviewed and he was  asked, "Hey, what's going to change in the next 10 years?" And his response was, "That's an interesting question, but the question that I don't get asked enough, and I should get asked all the time, is what's not going to change in 10 years?" He said, "Those are the things that we need to put a lot of our effort into,  because I can't imagine a future where a customer says, 'You know what, I love Amazon. I just wish items were delivered slower,' or 'I love Amazon, I just wish prices were higher.'" It's impossible. So this is where we spend our time, and these are the things for DynamoDB that are the things that won't change.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=60" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzxpsq8omzoh2j32chkrn.jpg" alt="Thumbnail 60" width="800" height="450"></a></p>

<p>This is where we spend our time. My name's Craig Howard. I've been at Amazon for 20 years. Sixteen of those years I've been in AWS, and the past six have been on DynamoDB, and I spend my time on these things, the things that won't change. These are things that are behind the scenes. I find them very fulfilling, but it's also kind of hard to explain to my family what I actually do every day.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=90" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1r1ybo9q04llif0289xp.jpg" alt="Thumbnail 90" width="800" height="450"></a></p>

<p>On October 20th in US East 1, we did not live up to our standards in availability.  Customers don't normally notice what I do. They don't see the fix that went in that prevented a future event. They don't see the behind-the-scenes scaling work that led us to onboard thousands of more customers. They don't notice until they really notice, and on October 20th they noticed. So I want to start with an apology. We apologize for the impact that we caused to you and to your customers.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3yxazo2a840jrvfiaesc.jpg" alt="Thumbnail 120" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F76zc35xmon9harwwcxka.jpg" alt="Thumbnail 150" width="800" height="450"></a></p>

<p>After every event like this, no matter what size,  we go through a formal process, a formal postmortem called a Correction of Error, or COE. And these are learning opportunities. We review these all across AWS so that we can learn from each other and we can all make our services and AWS as a whole better and better for you and your customers. And the goal is always to learn. So in this formal document, there's a bunch of sections outlined that everybody has to fill in, and my favorite is the lessons learned section. And so in this session, I'm going to share what we learned, and my goal is not just to make DynamoDB  better, my goal is not just to make AWS better. My goal is to help you make your systems better for your customers.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=160" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqx5cbwyz3e7plw1qvirr.jpg" alt="Thumbnail 160" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F727ituj10idttikdd9ro.jpg" alt="Thumbnail 170" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=180" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxelb1vdrcapc5pbj0hdg.jpg" alt="Thumbnail 180" width="800" height="450"></a></p>

<h3>
  
  
  How DynamoDB Uses DNS: Scale, Complexity, and the Impact of Returning Zero IPs
</h3>

<p>So, I work on DynamoDB.  The session is going to focus purely on DynamoDB's impact. I'm not going to talk about all the downstream impacts. We were the cause of the larger event, but I'm going to focus on  that. And what happened was during this event, we returned zero IPs from DNS, so customers could not connect to DynamoDB. So I'm going to go through  how DynamoDB uses DNS. I'm going to talk about how our DNS management system works. I'm going to talk about the specific race condition that caused the impact. I'll talk about how we mitigated the impact during the event and spend a bunch of time on lessons learned.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1trbnngc9m6q8hzvj1io.jpg" alt="Thumbnail 200" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=210" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkt02rbeqnrs18rmafvsg.jpg" alt="Thumbnail 210" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=230" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0fv6agzsp6wxjxkoz7ou.jpg" alt="Thumbnail 230" width="800" height="450"></a></p>

<p>So starting with how DynamoDB uses DNS.  Everything gets harder at scale. Every order of magnitude that your system goes through, you have to revisit some assumption that you've made in the past,  and DynamoDB has been through many orders of magnitude growth over its time. Likewise, our use of DNS has gotten more complex over time as we've had to rethink some assumptions. And so some examples of this might be, at some point your service grows and you go from having a single load balancer to multiple load balancers. Well, now you've got to figure out how do you get  the traffic from one load balancer to the other and how do you handle all of that.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=240" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wez4cps40akkvqt3o54.jpg" alt="Thumbnail 240" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsx49id5dng7xzstg5ded.jpg" alt="Thumbnail 260" width="800" height="450"></a></p>

<p>Another one is you get bigger, you may get to the point where it's not practical to have one instance  type across your entire fleet. And so you start mixing and matching, and now you've got more complexity to deal with, the capacity behind individual load balancers. So half a million requests per second is a pretty big number. You start to think about things a little bit differently. Well, for us,  we have hundreds of tables that are driving half a million requests per second or more, just concurrently all the time, every day. This is just normal for us. It's not just over 500,000 requests per second. Amazon stores on Prime Day, most recently, they did 151 million requests per second on their own. That's just one of our customers.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=280" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F12viv78kys5hy6zj87dq.jpg" alt="Thumbnail 280" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=290" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fytd7icqlmy7volmh4sq5.jpg" alt="Thumbnail 290" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fucjb96perpikotcpvspv.jpg" alt="Thumbnail 300" width="800" height="450"></a></p>

<p>So as I said, we've been through several orders of magnitude growth, and we're continuing to deal with this growth and rethink various aspects of our architecture. We have hundreds of load balancers per region.  We have hundreds of instances behind every load balancer. In order to be low cost, we mix and match instance types and all the complexity that comes with that.  Every load balancer has IPv4 and IPv6.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=310" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwc77dkrn1ie6jg0d8v0q.jpg" alt="Thumbnail 310" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=330" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F365ftlk4eacpd0q4jdqb.jpg" alt="Thumbnail 330" width="800" height="450"></a></p>

<p>We want to manage capacity together, but they end up being separate DNS trees because of how DNS works.  Just so we're all on the same page, when I say DNS, I'm talking about the DNS name that customers use to connect to DynamoDB. The first thing you do is your system does a resolution of this name or whatever other region that you're in. The DNS system maps that to an IP address, an IPv4 address, or an IPv6  address. If you don't get an answer from this, there's nothing you can do because the system's actually connected to the IP address, not the name.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=340" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw5pecz4mmdb61y9kub6z.jpg" alt="Thumbnail 340" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=350" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5k1ekfqcmnzwd9iop3e2.jpg" alt="Thumbnail 350" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkq2k1f960p8oqgauy4qd.jpg" alt="Thumbnail 370" width="800" height="450"></a></p>

<p>As you can see here, you can use dig, the  command line tool, and this is dig for an IPv6 address in US East One. And this is the answer that you get back, or the response you get back  from dig, and I'll highlight here that this is the particular IP address that you get. It can be a little hard to read, but this indicates that this was a successful request. There's no error here, and you got back one answer for one query. The query was, hey, what's the quad A record for dynamodb.us-east-1.api.aws? And the answer is the IP address that you got. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=380" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbg4sk5ar8gcc9u139dcj.jpg" alt="Thumbnail 380" width="800" height="450"></a></p>

<p>But right now, if you go and you ask our IPv4 only name for a quad A record, you're going to get back no answers, right? You'll see  zero answers for your one query. It's not an error, there just isn't a record. And this simulates exactly what happened during the event. If you asked for an A record from this domain, you would have gotten back the same response, no errors, no responses. If you asked for an IPv6, likewise, you would have gotten no errors, no response.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=410" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9m2gfgtj7x6wao3frh2k.jpg" alt="Thumbnail 410" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=450" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr283e76grgi9nsfka83b.jpg" alt="Thumbnail 450" width="800" height="450"></a></p>

<h3>
  
  
  DNS Architecture at Scale: Resolvers, Control Planes, and Managing Hundreds of Load Balancers
</h3>

<p>So DNS itself is a little more complex than you initially think. There's a lot more going on.  When you're querying DNS because you want to use a service, your instance then makes a request and it goes to the local resolver cache in the box, then goes out to the DNS resolver, the recursive resolver. If you're running an EC2, you're probably using the EC2 resolver. It knows how to forward multiple requests to walk the DNS hierarchy, and then it will eventually send the request on to the authoritative server, which is where the hosted zone is to get the actual answer that's there. Resolvers include a cache, so DNS is extremely high traffic.  It's a great place to put one.</p>

<p>But if you own a web property, you need to configure your DNS, and so there's another whole workflow for DNS. You've got an operator, or increasingly automation, and automation does the vast majority of this, talks to the control plane and says, hey, here's my load balancer, these are the IP addresses that are associated with it. And then the control plane will then asynchronously propagate those updates out to the authoritative servers so they can be serviced to customers.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=490" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbcjy6kgqjme0zhs2uj3.jpg" alt="Thumbnail 490" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=510" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F66mqp32k9a3natl4emjb.jpg" alt="Thumbnail 510" width="800" height="450"></a></p>

<p>We have multiple, as I mentioned, we have multiple load balancers. Different load balancers have different capacities. It's indicated by the weight. The weight is just  an arbitrary number, so when the authoritative server picks one, it picks one at random, but it's a weighted response. And so one thing that's common for us as we're growing, we add capacity. Maybe we add more instances, maybe we swap out to more powerful instance types.  And so we need to adjust the weight. So it's a very common operation. We do this all the time. It's all completely automated.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=520" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9dakxorgiyrcvyjtoo0a.jpg" alt="Thumbnail 520" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsec5em4vxzcg3w89xpuj.jpg" alt="Thumbnail 540" width="800" height="450"></a></p>

<p>Another thing DNS is responsible for is handling failure.  So DNS has health checks built in, so it's constantly doing all these checks for you. If something's completely unhealthy, it will not return the IP address associated with that load balancer or availability zone, whatever it is that you've set up your health checks for. That's great. Whenever you can use health checks, absolutely do that. That is completely the right way to go. </p>

<p>But a harder one you also have to deal with is partial failures. Maybe the system's failing in a way where you'd like to avoid sending traffic there, but it's not unhealthy enough that the health checks will fail. And so when you need to do this, you need to do some form of DNS update to administratively say, hey, don't send the traffic there, please stop returning the IP addresses for that availability zone, load balancer, or whatever it is.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=570" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbh4bnp5cb9tg713dofx2.jpg" alt="Thumbnail 570" width="800" height="450"></a></p>

<p>For DynamoDB, we're an extremely low latency service. Predictable low latency is one of the  things that we offer, that's core to what we do, and we keep raising that latency bar over time. And so one of our goals is we want, once a customer gets routed to one of the front ends of DynamoDB, a load balancer and the capacity behind it, we want to keep that as close as possible to where the customer is making the request from. We want to shrink that network distance because that's meaningful to us.</p>

<p>That can lead to hotspots as we end up with capacity and supply and demand mismatches, where the requests are coming from, how much capacity we have in a particular location, so we're going to manage that tension as well. DNS is complex, and as we scale,</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fixaj519712cdjn22ldq2.jpg" alt="Thumbnail 610" width="800" height="450"></a></p>

<p>the complexity keeps increasing. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=620" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0x64hio0gxdhkc165kyh.jpg" alt="Thumbnail 620" width="800" height="450"></a></p>

<h3>
  
  
  The DNS Management System: Planner, Enactor, and Route 53-Based Locking Protocol
</h3>

<p>I need to be clear, during this entire event, Route 53 behaved completely normally. There were no anomalies at all from Route 53. This was our misconfiguration of Route 53. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=630" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fko6cd9iysbveb02k1ymm.jpg" alt="Thumbnail 630" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=640" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq6jtmtb63n0g75jr4ibe.jpg" alt="Thumbnail 640" width="800" height="450"></a></p>

<p>Okay, so I'll walk you through how we've set up our DNS management system. We have two components: the planner  and the enactor. So the planner's job is to look at the state of the region. It might make some calls out to other services.  It might talk to NLB and say, hey, how are these load balancers configured? It might go look at some of the EC2 configuration of what instance types are where. It might go look at some metrics of where traffic's coming from. And the input to this is an operator taking action, like calling an API to say I want to add capacity or I want to change the high-level traffic. I want to send more traffic over here and less over there. And again, most of this these days is automation, but we present these higher-level APIs and then we produce a plan.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=680" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbubk9cvvhyhi65dwu7pl.jpg" alt="Thumbnail 680" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=700" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffv336khpfv3iis80yz9u.jpg" alt="Thumbnail 700" width="800" height="450"></a></p>

<p>So the plan is just a flat text file, JSON, and it describes the state of what we  want the entire region to look like. It's nice being just a flat file. We can store the history of this in S3, so we can go forwards and back to old plans if we need to. And then the enactor's job is to consume the latest plan  and actually talk to the Route 53 control API and configure DNS to look like this. So it's the thing that is talking to Route 53 and understands individual DNS records. Nothing before this understands that.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=750" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjyvo9n651jmde2j65jgh.jpg" alt="Thumbnail 750" width="800" height="450"></a></p>

<p>We designed the enactor so it will be able to function no matter what kind of large-scale event was going on or event in any particular region, right? Because we knew that we might need this system to go recover from a large event, and so it has to work in all conditions. This setup that I've described is completely identical in every region, no differences whatsoever. But we're going to zoom in on the enactor because this is the component that caused us the trouble. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=780" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn262efogayoh4cgizdgg.jpg" alt="Thumbnail 780" width="800" height="450"></a></p>

<p>So the problem that we have, though, is we need the enactor to be healthy in order to make forward progress. It has to get a new plan and make the requests, and it's got to run on an instance somewhere at some point. And so we have to deal with the case where, well, what happens if that enactor is in the availability zone that's failed or whatever? We need something, and so of course, obvious things, you don't run one of them, you run three of them. And so we run three of them across three different availability zones, again, same in every region. And they're all doing the same work. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpxp0715t0onb5x9s0gs0.jpg" alt="Thumbnail 790" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=800" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8uhsyn4u8lnntlqwmped.jpg" alt="Thumbnail 800" width="800" height="450"></a></p>

<p>They're consuming the same plan, they're producing the same records, so it's effectively idempotent what they're doing. Now we don't have risk of a single box failing for whatever reason,  so we know we can always make forward progress. But because you've got three entities all trying to do the same work,  you want to coordinate them and serialize the work in some fashion.</p>

<p>It's hard to reason about a bunch of changes that are happening at once. It's totally safe for our protocol. It's okay for us to have two enactors do the same thing at the same time. All the updates are idempotent. It's an eventually consistent system, which we find works extremely well at scale. We had another session where we talked a lot about embracing eventual consistency at the highest scales. We want these systems to be as resilient as possible, so we don't want them talking to anything else. We don't want them talking to each other because there's just more places for it to fail, but it's just hard to reason about.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=850" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjtr3h0wntli1u8mxrxhf.jpg" alt="Thumbnail 850" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=860" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc4b2totl8otwtkuiieyd.jpg" alt="Thumbnail 860" width="800" height="450"></a></p>

<p>And so we looked at it and we said, hey, can we serialize this by adding a lock, some form of concurrency control, just so it's easier for us to reason about  because simple is more resilient. And it turns out this entire pattern is used in several places in AWS for different services. And what everybody else who does this does is they  use DynamoDB to create a lock. But being DynamoDB, we can't use DynamoDB to recover from DynamoDB if DynamoDB is down. So we said, okay, we can't do that.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=890" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftd6a9nxhcpo7ifwsihgg.jpg" alt="Thumbnail 890" width="800" height="450"></a></p>

<p>We didn't want to add any other dependencies again because we have to recover in the worst outages. And so we said, right now, all we depend on is the plan and Route 53. Can we use Route 53 to create a lock? Well, you can create a text record,  and the existence of that text record indicates who the holder of the lock is. Right, so the value of the text record is the hostname of the lock holder, and then you put some epoch seconds for how long this lock is alive for.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=920" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F74qjusqjk66x56tr19x1.jpg" alt="Thumbnail 920" width="800" height="450"></a></p>

<p>And you just check the presence of this, but not through the data plane because that's eventually consistent and you're not going to do the right thing. But if you go through the control API, you will transactionally know if that's there or not and if you hold the lock. But that means you have to have this protocol whenever you're doing an  update. And so in the control plane for Route 53, it's called in the control plane for Route 53, they have a change record sets batch, and by definition, that batch is transactional. Everything in that batch will either be completely committed or nothing will.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp0p3ohuoefanvdd70xfs.jpg" alt="Thumbnail 960" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=980" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs02kaivhhke5uimfqsuu.jpg" alt="Thumbnail 980" width="800" height="450"></a></p>

<p>And so you do this strange thing at the start of every update, and you delete the lock that you know is there. Because this delete will only succeed if the lock still looks like the way you thought it did. Right, so it looks like it's my lock because the host names still match. I know when it expires, this is my lock, cool, I can proceed. This fails, the whole thing aborts, nothing changes.  And then you recreate the exact same record, probably with a new timestamp to indicate that you're making forward progress. And that's the locking protocol. You just put this preamble at the beginning of every batch, and then you add whatever actual other updates you want to do.  You're updating your DNS to do whatever it is that else you're doing.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1000" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi4fi79w2ek0diap2smdz.jpg" alt="Thumbnail 1000" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1020" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F19sj18macoj1fwvcsmrm.jpg" alt="Thumbnail 1020" width="800" height="450"></a></p>

<h3>
  
  
  Building DNS Trees with UUIDs: Managing Multiple Endpoints and Lock Contention
</h3>

<p>It turns out this works fantastic. This is not what, this is not the problem that we had. This pattern is great, and it allows you to implement this locking and serialization when the only thing you can depend on is the Route 53 control API. It's very cool. But  whenever you have a system with locking, you have to deal with lock contention. Right? One enactor is going to get the lock. What do you do with the other guys? So for us, we don't really care about the throughput of the system. We have multiple enactors purely to make sure that we just continue to make forward progress.  And so the standard thing works great for us here, you just retry and back off. Right? We don't care that these guys aren't making forward progress because we just need one of them to do it because they're all trying to do the same work.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1040" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fawrkiiz0c3zedwta5i7s.jpg" alt="Thumbnail 1040" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F45ysfwgoshm641l3cod4.jpg" alt="Thumbnail 1060" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1070" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp3h7gtvwe5cv62g9af2k.jpg" alt="Thumbnail 1070" width="800" height="450"></a></p>

<p>And so this is what we do. This is what we've done for years. We  use Route 53 for locking to make sure only one enactor is actually making forward progress. If the others do, it's no big deal. Our protocol works fine. It's essentially consistent. Be careful, this is going to be some blatant foreshadowing, this is a consensus algorithm. So what we're actually doing, what the enactor is actually  doing when it's doing this is we're building up this tree structure. Right, so we have an individual load balancer, has an IP address, has a weight associated with it. We have a whole bunch of them.  Like I said, we've got hundreds of these. I can fit two on the slides, so that's what I've got.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1080" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fohtzk5sbdmcy9iwb9ace.jpg" alt="Thumbnail 1080" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1100" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyfha1mgelt38neav9iez.jpg" alt="Thumbnail 1100" width="800" height="450"></a></p>

<p>But above that we have,  you know, a weighted round robin record, which is the plan for this planned version number. And so when the customer resolves this record in the authoritative server, it will return the IP address of one of its child records and it picks that randomly based on the weights. But this is not a stable name. We need at the top the stable name that the customer's actually going to use. So we put an alias record that points to the current  active plan, right? And so again, when the authoritative server resolves this record, it just follows the alias just like you would a pointer or a symlink or whatever to the root of the next tree.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpwktzwhrz8dn3y7irvh2.jpg" alt="Thumbnail 1120" width="800" height="450"></a></p>

<p>And when you want to make a change, you just go build a whole new tree. Right, so in this case, you know, the automation or an  operator said, hey, we added some capacity behind load balancer one. And so you'll see that the weight's increased, we built this whole new tree, and this pattern's great. Because it allows you a chance to go test this whole new tree structure. No customer's going to hit it yet because the top-level alias isn't pointing at that tree. When you're happy with it, all you do is update the alias, the root alias, and now the customers are able to hit this. And one even nicer property is if you don't like this change, rollback is just one alias flip away.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1160" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuo9ruge73gyfuerdpj93.jpg" alt="Thumbnail 1160" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1180" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffbxdgb0qail1xenr9crl.jpg" alt="Thumbnail 1180" width="800" height="450"></a></p>

<p>This is a common pattern. If you implement a database storage engine, you probably go build trees like this.  You start at the leaf, you work your way up, you update the root. This is standard stuff. But it's really cool that this actually works in the Route 53 control plane. We can build our DNS around this. One little aside, you know, we read a lot of computer science textbooks, and this was a great decision from a computer science perspective, but we made every  record a UUID. Because it was just machines, they don't care. Let me tell you, when you're debugging this with your UUIDs, which we had to do on October 20th, this was difficult. And this slowed us down, and this is a lesson, right? Like this was technically the right decision. But we didn't consider the API of the operators.</p>

<p>And late at night when we're all tired. So when you're building your systems, when you're worried about resiliency, you must think about what a human's going to see when they're trying to debug the system live, when your pager's going off. So I've already shown you, we have two different domain names. We started off with the IPv4 name. This is the name that we've used since the beginning. Again, this pattern is exactly identical across every region.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1240" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F801uffx8smxbz0dzohxe.jpg" alt="Thumbnail 1240" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1250" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqaxqcm60tx8fj28as6to.jpg" alt="Thumbnail 1250" width="800" height="450"></a></p>

<p>We now support dual stacks, so IPv4 and IPv6. There's more. We have FIPS endpoints. If you need FIPS, you're aware of FIPS. It's a government crypto standard.  But we also have cells endpoints, and I'll talk more about cells later. But the point here is we have the system is managing many different DNS endpoints in their own hosted zones.  And our locking protocol is the existence of a text record in a zone. And so what we actually have is a whole bunch of different locks, one for every endpoint.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1280" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqok52kxdbiaxtpia3j67.jpg" alt="Thumbnail 1280" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1290" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0rps9fi5d02cu1xn9v3.jpg" alt="Thumbnail 1290" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Furvuq6waqwnnehy6m1v7.jpg" alt="Thumbnail 1300" width="800" height="450"></a></p>

<h3>
  
  
  The Race Condition: Stale Plan Installation and Record Cleanup Collision
</h3>

<p>There are ways to stretch the locks across endpoints, and they get complicated, so you can totally do that. We chose not to do that because we knew we had an eventually consistent system and it's already complex and we didn't want to add another level of complexity because simple is generally the most resilient. All right, so the race.  This bug, this bug was a race.  You won't have this exact bug  in your system, you won't have this exact race. But you will have races and you will have bugs.</p>

<p>I've read a lot of correction of error documents over my time at Amazon, and the worst ones are, we had a bug, action item, fix the bug, action item, add a test for the bug. I don't learn much from that because the bugs are unique. What's helpful, and where I learned a lot from, is like, hey, here's this new class of thing that I wasn't thinking about. Here's how to structure your system, so even if there is a bug here, it won't cause problems. Here's some debugging techniques, you know, here's some operational techniques.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1350" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffg9jxked3z6qieyc7nb5.jpg" alt="Thumbnail 1350" width="800" height="450"></a></p>

<p>And so you're all curious about the very specifics of this bug, but please don't just spend your time thinking about that. The important takeaway for this is all the rest of it, all around it. How can you apply this to make your systems more resilient for you and your customers? Okay. So fundamentally, we had a race between  installing a new plan and cleaning up old records.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1360" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo75nyufo9pstiqniezl5.jpg" alt="Thumbnail 1360" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4snzhppi6q0y5on2w8zh.jpg" alt="Thumbnail 1370" width="800" height="450"></a></p>

<p>So plan install. We hit a condition that led to us installing  a stale plan, a much more stale plan than we'd considered. So I'm already having to simplify because slides are in two dimensions and I only have an hour to go through all  this. But again, there are multiple endpoints. So recall that this is happening across multiple endpoints and we're looping over all of them as well.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhsf7hbz0f742ex4rseaf.jpg" alt="Thumbnail 1390" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1400" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F024n0wezdvkkcxxo3eaz.jpg" alt="Thumbnail 1400" width="800" height="450"></a></p>

<p>We've got three enactors, right? Numbers are all made up for the illustration here, but they're all on plan 110, right? Two of them go to acquire the lock. One succeeds. One backs off and retries.  The first one does his work. More plans are generated, more plans are generated, and more plans are generated, and eventually the backoff's finished. This whole pattern repeats.  We try and get the lock. But the same guy backs off.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1410" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm3rywhmd426hkoekel30.jpg" alt="Thumbnail 1410" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1420" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcp5axs9c6mz1n41agdnb.jpg" alt="Thumbnail 1420" width="800" height="450"></a></p>

<p>When you have lock contention, what you'd expect to  see is the lock contention being evenly distributed across all the enactors. What we saw this time was one enactor failing to acquire the lock multiple times in a row.  And in a public RCA we said unusual delays, and in talking to customers after the fact, we realized that that was way too ambiguous of language. What we mean is this. It was very unusual for one enactor to fail to get the lock multiple times in a row, and then the backoff protocol caused it to get behind.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1440" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1rzd34plxucemdjpm0c5.jpg" alt="Thumbnail 1440" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1450" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F075qrhqv7c3vvmix0vb0.jpg" alt="Thumbnail 1450" width="800" height="450"></a></p>

<p>Okay. And so this pattern repeated,  and again more plans are generated, everybody's moving forward, making forward progress. So far, no problem, right? We have this idempotent, eventually consistent system.  But now this guy gets the lock, which is inevitable. And he's now about to install a much older plan than we'd ever considered. We're going back and then installing a new one, that's fine, we thought about that. This is much older than we ever imagined.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1470" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpt44pji9lawvxv0kcz9r.jpg" alt="Thumbnail 1470" width="800" height="450"></a></p>

<p>So that's how we got the stale plan, right? That's what triggered the latent race condition.  So record cleaning, right, we're building whole new trees, which means every update we're just building the entire structure of DynamoDB over and over again. We're generating a lot of records. Well, we don't want to run out of records. Route 53 can handle a lot of records, but eventually at some point we were worried about running out of records.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1490" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftjg38s4lqd6430n8elc7.jpg" alt="Thumbnail 1490" width="800" height="450"></a></p>

<p>So we have to clean up old trees,  right? And so we have the new plan that's been installed, and we keep a whole bunch so we can update the root alias to point to an old tree, no big deal, like we want this there. And what we do is we look at the stuff that's really old and we say, hey, let's get rid of that. That's fundamentally what the record cleaner is doing,</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1510" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu76b7c3taahubdv81ku4.jpg" alt="Thumbnail 1510" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1520" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2a6gvtpywj1faup765rn.jpg" alt="Thumbnail 1520" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fec803misijdhmmgl9i02.jpg" alt="Thumbnail 1540" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1550" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiqhfbn0grkjqb2toefgz.jpg" alt="Thumbnail 1550" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4ms3migim3ykm0484gjl.jpg" alt="Thumbnail 1560" width="800" height="450"></a></p>

<p>deleting the really old stuff, leaving some  relatively recent stuff, and we've got the active one that's taking traffic right now. And so the race is we have that unusual  delay, the multiple backoffs and retries. We had a stale plan that was able to become active, a much more stale plan than we're used to being able to get in. And the record cleaner's going to run it, so we got our three enactors, and they're all trying to do different things. One of them says, hey, I just finished putting 145 in, it's time for me to  go clean up stuff that is sufficiently old. The next guy says, hey, 146 is here, time for me to install that one. And that third enactor,  who experienced the unusual delay, is still working on the old plan. And so there's a sequence of steps here where we can install that,  the old plan, and then immediately run the cleanup.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1590" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2bbghhacoybrw2q3q9lu.jpg" alt="Thumbnail 1590" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbize75gjkia9508ltrs.jpg" alt="Thumbnail 1610" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1620" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdeb12hrjmczctpvu2lm3.jpg" alt="Thumbnail 1620" width="800" height="450"></a></p>

<p>And so what happens is we have this, right? We have the root that's looking at what is currently the newest, the highest plan version. We can flip it to the old plan. Again, this is an eventually consistent system, so we thought about this case, but then it immediately gets cleaned up because it is just so old, and we're left like this. And this wouldn't have been  that big. It would have been a very large event, but we would have gotten over it very quickly because the enactor keeps running. It's constantly looking at new plans and installing new stuff, right? But we hit some inconsistent state, and this is some more wording we use in the public RCA that in retrospect was probably a little too ambiguous.  We thought about this, and we thought about, hey, we've got to be able to roll back, and so we created this other record you see at the bottom of the screen there, the rollback record. Because we wanted an operator to be able to go in with an update to whatever was  most recently active, right?</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1660" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0b0aci8fapdu1r3ujvsj.jpg" alt="Thumbnail 1660" width="800" height="450"></a></p>

<p>So for every time we update the route record, we also update the rollback record to point to whatever was most recently updated. We do that as part of the transaction, so it has to happen. Either both these things have to happen or neither of them can happen. But when that plan's missing, our code couldn't handle the fact that we've tried to point the rollback to nothing, because what the current one is nothing. Which meant that the enactor couldn't make any forward progress, and this is why we had to get operators involved. So this is the inconsistent state that we refer to in the public RCA. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1670" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgt64yteh5nch3d8iz6l8.jpg" alt="Thumbnail 1670" width="800" height="450"></a></p>

<h3>
  
  
  Mitigation Timeline: From Impact at 6:48 UTC to Global Fix by October 28th
</h3>

<p>So how we mitigated this. The impact began at 6:48. That's when we installed, or we deleted, the active record.  By 7:38, we'd have understood—because all these times are all UTC—I understood what the root cause was within seconds or minutes of the event happening and deleting the records. The team responsible, just one subteam in DynamoDB, the team responsible, got hundreds of alarms firing. Pagers going off like crazy, and that was just the one team. This happened all across AWS. So some alarms were fine-grained, some alarms were coarse-grained. And if you go to these events over the years, a very common action item is, oh, go add another alarm. We had so many alarms that we had one alarm that pointed us directly at root cause, but it got lost in the noise.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1740" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv6xu6spppkudxq783hbv.jpg" alt="Thumbnail 1740" width="800" height="450"></a></p>

<p>And so as you're going through some of your events and your gut reaction is to go add another alarm, think about in the largest events, how do you deal with the deluge of tickets and alerts and alarms. It took us 15 minutes to sort through all of this to find the actual root cause. Again, we thought about this, we'd kind of practiced the scenarios where  we can't make forward progress, but all of our tooling assumed that we'd be able to use the enactor, and the enactor was the component that broke in this case. And so all our tools, a lot of our tools didn't work. We still would have been able to fix the impact without it, just we were able to speed things up by focusing on fixing some internal tooling first.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1770" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frm3ktv9fa9379z1qeyu1.jpg" alt="Thumbnail 1770" width="800" height="450"></a></p>

<p>And 9:25 UTC is when we got the DNS healthy again by getting a healthy tree installed.  We got the records in the authoritative DNS at 9:25. But again, remember there's a cache in the resolver, and then in DNS negative caching works differently than positive caching. So the presence of, the absence of a record tends to be cached longer. There's good reasons why you do that. But so individual customers would have started to observe DynamoDB's DNS again between 9:25 and 9:40 UTC.</p>

<p>So another one for you to think about is, you know, caches help you with a lot of your resilience stories, but in some cases they can prolong outages. And so think about that. Caching can be great for masking transient issues and improving performance in many scenarios, but think about whether, when you're down, this caching layer is helping you or hurting you. That consideration might make you make a different choice.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1820" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnq0sh5lza3soyobe74pt.jpg" alt="Thumbnail 1820" width="800" height="450"></a></p>

<p>At 12:38 UTC, we completed disabling DNS management, the automated system across all the regions.  At this point, we didn't know exactly what the race was. We knew there was a race condition, and we knew it could have hit any region at any time, so we wanted to make sure it was disabled everywhere else. We wanted to contain the impact to the one region where we unfortunately lost this race.</p>

<p>This was another nice thing that we had in that we knew we could pause this. We knew we could turn this system off and freeze the world in place. Again, think through your system and where you can freeze your world. What degradations are you okay with? We couldn't necessarily add capacity because all the automation was disabled, but that's fine. We don't require adding capacity emergently.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1870" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbvnt5iuxk206k7phua2v.jpg" alt="Thumbnail 1870" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1890" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd9snkn6yqmp12qerzhoa.jpg" alt="Thumbnail 1890" width="800" height="450"></a></p>

<p>We zoom back out and start looking at the day level here now. The event happened on October 20th.  By October 22nd, we had the race fixed and validated, and I'll talk more about the validation. By October 24th, we deployed the fix to the first region, and by October 28th, we had it across all of our regions worldwide.  Our automation was back on, and we were behaving normally, operating totally normally again.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1900" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F66n4hzm2jpwpsedo3jnc.jpg" alt="Thumbnail 1900" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1910" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyi0pta38sgm2iqfole4s.jpg" alt="Thumbnail 1910" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1950" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftr4om8cmi1906hb6v4tv.jpg" alt="Thumbnail 1950" width="800" height="450"></a></p>

<h3>
  
  
  Lessons Learned: Analysis with Amazon Q, Formal Modeling with TLA+, and Validating Production Code
</h3>

<p>Alright, so lessons.  I've got a bunch of them. We learned a lot from this event. These are the highlights that I want to share.  Analyzing these events can take time. There's the analysis in the heat of the moment so you can triage and make customers happy again, get the system back up. That's one type of analysis you have to do and need to do very quickly. There's another type of analysis where you look at, after the fact, what happened and how to fix the system to make it more resilient. They're similar but different. Speeding up both of them is important, but the mindset, of course, when you think of this is like, it's 3 in the morning local time,  pages are going off, you've got hundreds of pages, and you know there's impact right now.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fivluy39kxgrnj359pieq.jpg" alt="Thumbnail 1960" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=1970" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4xd66y6fj7anf93m8gc1.jpg" alt="Thumbnail 1970" width="800" height="450"></a></p>

<p>You log into the system and you're like, there's a bug in this log file somewhere.  This is like an eye test. You know it's there, you've got to find it. You pull out all your log analysis tools.  You use grep. Finding this is hard. These logs are megabytes or larger. CloudWatch log analysis is really helpful for this, for a whole class of errors. You're just looking for when the first error happened, what's the exception message, and you start thinking through how that works. That's fine, but it's still a needle in a haystack.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2000" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiio6fauxjml6z5zbokg4.jpg" alt="Thumbnail 2000" width="800" height="450"></a></p>

<p>For some of the more complex events where it's not just finding the exception and processing that, where it's about timing and you're looking at a sequence of events, well now you've got to level up.  You need a more powerful tool. The other thing is that over the years, people have been operating or working in the system and they've added log lines to help them with this, that, or something else, and so the logs themselves aren't necessarily a nice audit trail that you would want to debug this particular issue. Again, think through your log files because they are the API that your operators use to figure out what's going on, so give them some love too.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2050" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fopa40ey63x72m6ab7ykw.jpg" alt="Thumbnail 2050" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9qkwrte2rjly1k0xhz5h.jpg" alt="Thumbnail 2060" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2070" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdhuk7ekejs3jrcg7wchp.jpg" alt="Thumbnail 2070" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2080" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F62tk5medvi42n0981quo.jpg" alt="Thumbnail 2080" width="800" height="450"></a></p>

<p>In our case, after the fact, we're looking at these megabytes of logs across three different systems trying to line up the timing of individual things, and it was really tricky.  In the post-event analysis, we actually turned to Amazon Q here, and it really helped us find these patterns that we were looking for.  One of the things we could say is, here are the three log files, the code's over here.  It was able to help us in some cases because we wanted to know the sequence of events that led to the impact that we observed. It did that, and it was actually really helpful because  it's like, here you go, I noticed this anomaly. That's really cool, that's really super powerful.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2100" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh9vrclavasz78hfinapo.jpg" alt="Thumbnail 2100" width="800" height="450"></a></p>

<p>Because at 6:49 and 33 seconds, it saw the first failure. Which means one of us is wrong, because I just said this failure started at 6:48. What Amazon Q found for us was not the cause of the event,  it found an effect. Judgment is still required here.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2130" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmt577yeigbhwdtumz50r.jpg" alt="Thumbnail 2130" width="800" height="450"></a></p>

<p>What Amazon Q did was find the proximate cause, and it really helped speed us up by narrowing it down to the specific sequence we had to go find and helping us build up that timeline. By producing that more detailed timeline around this area, we were able to go in as operators and figure out exactly what happened. </p>

<p>The lesson here is that analysis can take a fair bit of time, especially when you're tired. These events always happen when you're tired, right? Amazon Q might not be the smartest operator that you ever work with, but it doesn't get tired. It doesn't visually scan over two lines that look almost identical but aren't in megabytes of logs. It's a fantastic tool and really focuses your effort and helps you zero in on things very quickly.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffggmnv600vjkoubsde09.jpg" alt="Thumbnail 2170" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2210" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flaammf8brzh8mdaxlabn.jpg" alt="Thumbnail 2210" width="800" height="450"></a></p>

<p>Testing is another area that  can be difficult for this type of bug. This timing issue is such that you might run a unit test over this area a million times and never see the sequence of events. All the unit tests passed, or maybe you did see a failure, but you're human and you hit retry in your CI/CD pipeline and it went away, so you thought it was a transient failure or a timeout somewhere. Did the test highlight something real that could be a big problem, or was it a product of the test environment? Unit testing or traditional automated testing is exceptionally difficult in this type of scenario. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2250" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9n29e2uoj7esykctv1ys.jpg" alt="Thumbnail 2250" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa6nw2cm5f8x6jgeuvovq.jpg" alt="Thumbnail 2260" width="800" height="450"></a></p>

<p>One thing that we've had tremendous success with at AWS in general and DynamoDB in particular is formal modeling. You build a mathematical model of all the states that your system can be in. You come up with a bunch of invariants, then mechanistically the system will go through and evaluate all the possible state transitions and highlight when you violate an invariant. But critically, we didn't have a formal model for this system, although we have lots of formal models for other things in DynamoDB. And we wanted one really quick. Formal models are difficult to construct, and  Amazon Q helped speed us up here. This is another one of the things that we did after the fact because we needed to validate any fix that we did. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2280" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4o9mhdppxj3rqimbd6y6.jpg" alt="Thumbnail 2280" width="800" height="450"></a></p>

<p>We started with a description of the algorithm that we had, looking at old design documents and comments in the code and things like that. We use TLA+, there are lots of options, it's just the one that we chose. We were able to iterate with Amazon Q and have it go back and forth.  It helped us come up with a bunch of invariants in this formal model. One of the cool things for me as somebody who's been working on the system for a while is it validated the original design. The original design was actually correct. Over time, the system had diverged from the design in little details in the implementation. Even if we'd had a formal model at the beginning, it wouldn't necessarily have helped us in this case.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2320" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn65h1bwp8x49mcnfek4h.jpg" alt="Thumbnail 2320" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2330" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frd0q5zww92miv9qt40jn.jpg" alt="Thumbnail 2330" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2350" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft9c62q8yu237mahu65o4.jpg" alt="Thumbnail 2350" width="800" height="450"></a></p>

<p>So far, this is a cool academic exercise, whatever. But the next step was super cool, because we said, hey, we got this model over here. You can see the source code that was running in production on that  day. Where did we screw up? And Amazon Q helped us find some inconsistencies. To me, this one's game changing.  Customers don't care about your formal models, they don't care how cool your algorithm is. They care about the system that's running in production today that they're interacting with. That's the thing you need to validate, and anything you can do to connect those dots is hugely valuable. </p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbj3t4a0g6k9jb5hawmyl.jpg" alt="Thumbnail 2390" width="800" height="450"></a></p>

<p>But again, you still need judgment. Some of the inconsistencies that it found were simplifications we put in the model. In the model we said, hey, we got this fixed queue of plans to work through because that's just how it works in the model, in TLA+. In reality, we have these infinitely generated, dynamically generated plans. But it did show us differences that were critical, and that led to this latent race condition. We were then able to make fixes in the code base  and rerun this analysis with Amazon Q. We got a nice pat on the back. Hey, this did fix it. So this is how we gained confidence that we could go enable the system again in every region, and we wouldn't hit this particular bug again.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2410" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdwgsfad2j4zbfuo5lerr.jpg" alt="Thumbnail 2410" width="800" height="450"></a></p>

<p>Formal modeling absolutely has a place in your  toolkit. Anything timing related, those consensus algorithms in disguise. Yes, I wish we'd thought about doing a formal modeling system way back, but we certainly have now. Previously it was really hard to know if that formal model actually matched what you had running in production, and so you can help validate that because that's what matters to your customers.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2440" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flsdbp2iwfku8ybg85nqi.jpg" alt="Thumbnail 2440" width="800" height="450"></a></p>

<h3>
  
  
  Reducing Regional Blast Radius: Cellularization Strategies and DNS Isolation Trade-offs
</h3>

<p>The other thing about all this is that DNS is inherently regional. The way we've got everything set up as a regional service, DNS is how we get there because it's looking at how do I route traffic across the region. Where is capacity in this region? And that led to a single update causing regional impact. So one of the things that we really want to do is look at how do we take this regional setup and make it so that any individual operation cannot have regional blast radius.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2480" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fixmo0lzpwxfvbch2sdjd.jpg" alt="Thumbnail 2480" width="800" height="450"></a></p>

<p>On the day of the event, DynamoDB did have some cellularization. We've been cellularizing for years. We're constantly working on leveling up, and we've recently been working on cellularizing up towards the front end of DynamoDB. We're continuing to push those cell boundaries all the way up. The closer you get to the customer, the more choices there are to make. There's many different ways you can implement cells and many different reasons why you might want to implement cells.</p>

<p>For folks that aren't aware, cells is you're taking a large unit, a large system, and you're saying, hey, instead of having one implementation of that, one big DynamoDB for the entire region, what we want is multiple DynamoDBs, right? And so they become isolated. Each unit of that we call a cell. Like I said, we've been working on cellularization for years in DynamoDB, and various components were cellularized on this day, but unfortunately not DNS management.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2550" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fb136hhfkuduakb7ep6.jpg" alt="Thumbnail 2550" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2580" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgtjjy4gird2onwdiya26.jpg" alt="Thumbnail 2580" width="800" height="450"></a></p>

<p>So when you are doing this and you are thinking through cells in your systems, you have this fundamental problem of I have my customers that need to get to their resources, and their resources will be in a cell of some sort, right? So how do we know that customer A has to go to cell 1 and customer B has to go to cell 2? The standard approach, and this is a great approach, by default you should probably take this approach,  right? You have a cell router. You have a fleet that is sitting there and all the requests go to the cell router, and its only job is to say that customer A is in cell 1 and customer B is in cell 2.</p>

<p>The system is extremely simple. Simple is the most resilient. You don't have to change the system very often at all. This leads to a lot of benefits having this system server side, right? Customers don't have to know what's going on. You used to point your DNS at your overall system, now you point at the cell router and suddenly everything is cellularized. That's great. No cooperation with customers. That also means you can update cell assignments if you need to, which is a real challenge, and not having to have the customer cooperate with that. That's great, right?</p>

<p>There's some downsides to this approach. The cell router is still regional. If you have a problem in that system, all the cells are down. The DNS for the cell router is still regional, right? And you need to update that as you add capacity to your cell router, right? But again, I think the trade-offs are probably worth it for almost everybody. For various reasons, and it has to do with DynamoDB scale and latency and all that, we took a different path.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2670" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffaypr7igfpdlce3tt290.jpg" alt="Thumbnail 2670" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2690" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft8l7spq0eftoxt7kkenu.jpg" alt="Thumbnail 2690" width="800" height="450"></a></p>

<p>This is an alternative. It's equally valid. I think each one of you needs to choose for your systems and why you're doing cells, which one's going to make a difference for you. The path that we're taking  is every cell is getting its own independent DNS name, top level, and critically also its own DNS management system. This has always been the plan, right? Then what we have to do is we have to get the customers there. And so what we're doing is we're changing how DNS works. And every customer is going to have their own DNS  name. And we'll have a list, you know, CNAMEs so that when customer A resolves their own DNS endpoint, they'll get routed to cell 1 and customer B will get routed to cell 2.</p>

<p>So the only shared fate in this world is that list of cell names, and it's just a list of cell names. There's a lot less shared fate here. It's not why we took this path, but it is one of the benefits of this path. It's more complex though, because I just said customers have to go update the DNS that they're using, and for a big established service like DynamoDB, it's really hard. Which reminds me, please go update your SDK.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2740" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flr1h4a40jb7mm9ivw4z8.jpg" alt="Thumbnail 2740" width="800" height="450"></a></p>

<p>So you may have noticed this depending on which credential provider you used, but about 12 to 18 months ago,  we started having customers send requests to account ID.ddb.region.api.aws. And that right now is a CNAME to the legacy name, but this is why we're doing it, to enable this. Just in general, keeping your SDK up to date is best practice. We're constantly learning new things about timeout values and connection pooling algorithms and all that stuff. And so being on the latest is just generally good practice, but in particular, we need folks to update their SDK so that we can complete our cellularization journey. So please, please, please, please go update your SDK.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fztj2gfvh00lcmw0nt8hp.jpg" alt="Thumbnail 2790" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2810" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7u4bt5incy3m891oho0.jpg" alt="Thumbnail 2810" width="800" height="450"></a></p>

<p>So cells are a great way to reduce regional impact,  but think through why you're doing cells. What are the most important reasons for you? Because there's so many ripple decisions, and it impacts a lot of the architecture throughout your system. The cell routing problem is one of them. But what else can we isolate?  Cells is one form of isolation. But as I said, DynamoDB management in a regional service is inherently regional. We're managing regional capacity. We're sharing capacity across endpoints like IPv4 and IPv6. How do we deal with this?</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2830" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fut612t14ih8zmpejjpah.jpg" alt="Thumbnail 2830" width="800" height="450"></a></p>

<p>And so this is more an area of exploration for us. We don't have the answer here. When we first designed all this, we said, hey, we could do zonal DNS,  which wouldn't be exposed to customers as like, hey, here's AZ1's DNS and here's AZ2's DNS, but as a component in that larger tree structure. We decided we didn't want to do that because it makes the capacity management problem a lot harder. We want to think about the capacity in the larger sense, because that's the easiest thing to do. But maybe that's okay. Maybe we should be trading off some capacity planning accuracy for more isolation.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2880" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7cjb7js5c416qtvyzw1t.jpg" alt="Thumbnail 2880" width="800" height="450"></a></p>

<p>Maybe we should be looking at the various load balancers and just sharding them across DNS names. Maybe we should be having multiple plans enacted, active at different points at different weights. We're going to be constantly exploring this, but there's this fundamental trade-off here with the isolation that you get.  You're trading some precision in global knowledge or regional knowledge or whatever boundaries you're looking at to operate in smaller units. And that moves complexity. And for all of this stuff, a more complex system tends to fail more often, and so you need to weigh that because maybe that simplicity is a win for you. Maybe those benefits aren't worth it.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1iir52l2ximsw296188n.jpg" alt="Thumbnail 2930" width="800" height="450"></a></p>

<h3>
  
  
  Temporal Isolation and Request Hedging: Fast Lanes, Slow Lanes, and Racing for Resiliency
</h3>

<p>But also, you've got to look at there's a large number of small outages versus a small number of large outages, and how does complexity impact which category your system falls into and why that is.  Another dimension of isolation is time. What's going to happen quickly? What can happen slowly? And how would your system actually handle the difference between these two types of things? Systems tend to evolve around the most strict requirements, which in our case is, hey, how fast can you go update DNS.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=2960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fakksz8v00nfrbozqww7z.jpg" alt="Thumbnail 2960" width="800" height="450"></a></p>

<p>But if you think about capacity adds for DynamoDB,  we don't need to add capacity quickly. We're an extremely large service. We forecast out in weeks, months in advance. We get the capacity in, we get it up and running well before we need it. Our system's designed so that we don't have to do any of these things in response to an event because during an event is when those types of things are least likely to work. And so we plan ahead and try and statically analyze all this.</p>

<p>But we looked at it, and we found that most of the new plans that we were generating were because of a capacity add or a shifting request traffic or shifting bulk traffic around in response to things. And those things, turns out, they don't have to happen quickly.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3020" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wl703h1y4joqmzvgr9i.jpg" alt="Thumbnail 3020" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3030" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7cdcjvzt08m2ge7xdgf5.jpg" alt="Thumbnail 3030" width="800" height="450"></a></p>

<p>If we hadn't generated as many plans, we never would have exposed the underlying latent race condition. But you contrast that with one of the  things we do, which is handle partial failures, where we decide we have to get traffic out of an Availability Zone and we have to do it quickly. We need that update to go through the system  quickly. That's really important to what we're doing. We actually go so far as to send messages to all of the servers in the Availability Zone we're removing and say, go send a connection close on your next HTTP response and then terminate TCP. Force the client to go re-resolve and go to a different Availability Zone.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffgadi4o9zwy8447itgnk.jpg" alt="Thumbnail 3060" width="800" height="450"></a></p>

<p>So we want that to go quickly, and one of the things that we can do is say, we've got the fast lane and the slow lane.  All the everyday stuff goes through the slow lane. It's okay if these changes take hours. We can do them super carefully and all this coordinated rollout. And we've got the fast lane for the emergencies, where it's got to happen now. And then the automation and the operator would choose, do you want to go in the emergency lane or the everyday lane? But the challenge then is emergencies don't happen every day. If you're not exercising the emergency path all the time, how do you know it's going to work when you need it? And so there's benefit to folding these two together and having one system. We know DNS updates are going to work when we need them because we're just doing them all the time.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3110" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fibnay75jb96qk5g3czn8.jpg" alt="Thumbnail 3110" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foqgvpiyksz5k3jdcgzh4.jpg" alt="Thumbnail 3120" width="800" height="450"></a></p>

<p>This is not an obvious choice here. You need to think through these as well. So think through those trade-offs. Is there a benefit to separating them?  Or is there a benefit to making sure that everything works all the time because you're exercising it constantly? I put this slide up earlier about the enactor and the planner.  I still love this pattern of having the plan in between. You get the planner, which is doing a bunch of things, talking to a bunch of services that are control plane services. They're not as reliable because they don't need to be. And so if there are errors in that system, or we're talking to its dependencies, or whatever calculations it's doing, if there are errors there, they don't become errors that we propagate to DNS because the worst that happens is we just don't generate a new plan. If there is no new plan, the enactor can't pick up a new plan and can't go mess up DNS.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fumdnm19wrgfxll3veu10.jpg" alt="Thumbnail 3170" width="800" height="450"></a></p>

<p>And so having this type of system that you can freeze in place opens up so many opportunities.  So think in your systems, what failures do you have that you can convert from an error to just, oh no, the system's going to stay in place for a little while. How long can you live in that mode? How do you exercise that? This is a particularly nice pattern when you're propagating configuration from a control plane to a data plane. Being able to freeze like this is just super important.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fusj2bl0j2hlw8k7uacpk.jpg" alt="Thumbnail 3200" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3220" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm8sb1bu68fg6gkvi8qoj.jpg" alt="Thumbnail 3220" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3250" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4vnixxcaf7yacoq0alrt.jpg" alt="Thumbnail 3250" width="800" height="450"></a></p>

<p>This one was interesting to me because our bug was a race,  and the race caused a tremendous impact. But not all races are bad. Races can actually help with your resiliency. So don't go and try and eliminate every race from your system. One example that we talk about an awful lot is request hedging.  It's not a term we invented. It's the standard term. But at a high level, DynamoDB is really simple. A customer sends a request. It goes to one of the request routers whose job is to do authentication and then do a lookup to figure out for your particular piece of data that you're interested in which storage node is that on. And we've got tens of thousands, hundreds of thousands of these. We've got a ton of these. And so the request router has to do a lookup in metadata to know for that piece of  data where is it, which storage node is it on.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F17c2pa6z9h18z2sck2xy.jpg" alt="Thumbnail 3260" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh8s6i3dpjuhj9a9wa8r4.jpg" alt="Thumbnail 3300" width="800" height="450"></a></p>

<p>And so what we actually do is we send two requests to  metadata all the time, and we race them. We take the first response we get back, and we ignore the second one. This has a couple of benefits. It smooths out the tail latency. If there's a hiccup on one of those boxes for whatever reason, customers don't notice because the other box will win the race. And so I just get to discard that slow response. If one of them happens to fail, I don't have to wait to kick off a retry. You can think of this as an optimistic or pessimistic retry depending on if you're a glass half full or half empty kind of person. But  this pattern works great. It does double the load on the system.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3320" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqsrhhlgdja58vyx1h4f.jpg" alt="Thumbnail 3320" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3350" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4b1syxwxru0d3czgsim5.jpg" alt="Thumbnail 3350" width="800" height="450"></a></p>

<p>And for us with metadata, it's multiple orders of magnitude smaller than our other fleets, and so this is not a material cost for us, but it is a cost. But we've been doing this for years and we're super happy with it, to the point where for some customers, we recommend you do this  in your calls to DynamoDB. If you are very sensitive to tail latency, very sensitive to those small errors that you get, send two requests on two different connections and race them. Yes, it does cost more, you're doing, the system is doing a lot more work, but for some businesses, and we have customers that do this today, this is totally worth it. In another session we went into this a little more detail. You can go look for that if you want a little more detail about request hedging. Fantastic pattern. Races make DynamoDB resilient. Pretty cool.  Think about where they can help you as well.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3360" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsa66uobjepbjyum1hpg4.jpg" alt="Thumbnail 3360" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvqskybwwwehpadmag8c.jpg" alt="Thumbnail 3370" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqstil3j8qbjwtfncf66s.jpg" alt="Thumbnail 3390" width="800" height="450"></a></p>

<h3>
  
  
  Dependencies and Final Reflections: There's No Compression Algorithm for Experience
</h3>

<p>The last one that I wanted to talk about was dependencies, right?  I think there's two types of dependencies. You have a strong dependency where that system must be up in order for you to do anything. You know, for us, if we don't have DNS functional,  DynamoDB, there's nothing we can do. If we don't have functional load balancers, there's nothing we can do. And it's okay for the system to be down in that case, because the alternative is extremely complex, and so we have this gut reaction. So we had a problem with our DNS configuration, naturally we started talking about, hey, do we need DNS?  Right? Like, there's a bunch of other ways you could get traffic into DynamoDB. We could go build smart clients and all that stuff. No. Whenever we go build, we will have to go build health checks and capacity management, all the stuff we built anyways. And the Route 53 team thinks about this stuff all day every day. They live and breathe this stuff, right? We would be worse off by trying to do this rather than working with the Route 53 team.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3440" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F84ksyyb9fwwr9o0dbdgg.jpg" alt="Thumbnail 3440" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3460" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnu05aoiuaco1vvdi5tp2.jpg" alt="Thumbnail 3460" width="800" height="450"></a></p>

<p>So we're not going to follow the kind of that knee jerk gut reaction and replace DNS. We're going to keep going down this path. We're just going to make the systems more resilient. On the flip side, there's a bunch of other dependencies where there's just no way your system can go down if that dependency is down.  You know, you can get metrics on your DynamoDB tables through CloudWatch. If CloudWatch is impaired, there's no way we should ever fail a request in DynamoDB. We're just not going to do it. And so we think about these systems very differently, and we architect around these different types of failures.  So examine your dependencies, think about how you use them. Don't go knee jerk react to certain in response to, just think about how will your system be benefited by these dependencies and yes, how can you make them more resilient.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3480" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7djsd7x1x17gfrrydjo.jpg" alt="Thumbnail 3480" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3490" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr67kgs2upck5wfp7u5lb.jpg" alt="Thumbnail 3490" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3520" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffrnuzeeuo6fytagm3ugq.jpg" alt="Thumbnail 3520" width="800" height="450"></a></p>

<p>So we've learned a lot from this event. Hopefully you've learned ways to improve your systems for your customers. We talked about analyzing  these events, how to speed that up, how important that is, talked about testing, timing related issues, talked about cells for isolation, other forms of isolation. Talked about timing factors and when you want to separate out a fast update versus a slow update and when you want to collapse those two to make so you're always exercising the emergency systems. Talked about freezing the system in place and converting errors to a frozen state. We talked about how races can actually help you with resiliency. We talked about dependencies.  We're always working on leveling these things up, we'll never stop. These are the things that won't change for us.</p>

<p><a href="https://www.youtube.com/watch?v=YZUNNzLDWb8&amp;t=3530" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8jpokzqc202bouj2rue7.jpg" alt="Thumbnail 3530" width="800" height="450"></a></p>

<p>We apologize again for any impact we caused to you and your customers.  And I wanted to leave you with an Andy Jassy quote. There's no compression algorithm for experience. This is painful to go through, but we think we're significantly better off for having gone through it. We hope your systems are also better off for having gone through it as well. So thank you very much for your time.</p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

