---
Title: AWS Cloud Praticttioner #02
Description: 
Author: FeliDrummond
Date: 2026-03-19T22:07:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hello guys, continuing our AWS Cloud Practitioner series, today we are going to study a specific case and explain how we can solve it using AWS resources.</p>

<p>According to the following case:<br>
A software development company needs to notify the engineering team whenever a new bug is reported in their bug tracking system. Some team members need to be notified immediately, while others can process the bug reports later.</p>

<p>How can this be solved in AWS Cloud?</p>

<p>Simple: you can use Amazon SNS. With Amazon SNS, it is possible to subscribe different endpoints to SNS topics and ensure that the right people or services receive information in real time.</p>

<p>What is an example of a service that can receive information from an SNS topic?</p>

<p>One example is Amazon SQS queues. SQS is a message queue service that allows systems to communicate in a decoupled way by storing messages in a queue until they are processed.</p>

<p>SQS queues can be configured as standard queues or First-In-First-Out (FIFO) queues. For example, you can create:</p>

<p>Queue 1 (SQS) → sends email<br>
Queue 2 (SQS) → saves data to a database<br>
Queue 3 (SQS) → sends data to analytics</p>

<p>And then subscribe each of these three queues to an SNS topic called "New user registered".</p>

<p>Every time a message is published to this topic, all three subscribed queues will receive the information and execute their specific tasks at the same time, in a decoupled way. If one queue fails to process the message, it will not affect the others.</p>

<p>This way, actions like sending emails, saving data to a database, and sending analytics events are triggered automatically at the right time.</p>

<p>Thank you everyone for more one day with me!</p>

