---
Title: Where cloud security policies break down
Description: 
Author: NickTheSecurityDude
Date: 2026-01-15T22:02:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>A great policy is worthless if it's not being followed.  When I take a look at our detection platform, I see that we have a number of resources that violate the policy we've carefully constructed.  Trying to remediate these is like playing whack-a-mole, just when I think I've got them all, new ones pop-up.</p>

<p>Take S3 for example, whether these are old resources or new ones I<br>
routinely find one out of compliance, for example with the public access block off.  Now, even with that off the bucket is not public, but it's one step or one mistake away from being available to anyone.  The public access block was a guardrail added by AWS 6 years ago to prevent one of the most common vulnerabilities at the time, public S3 buckets.  Back then it seemed like every month or so a major company was having an S3 bucket leaked.</p>

<p>So how can you prevent things like this?  One way is using AWS Config, however this is  difficult to setup auto-remediation for and often request a custom Lambda function which you need to then maintain.  This is where Preventive Security Posture Management (PSPM) comes in.</p>

<p>A PSPM focuses on enforcing policy continuously and automatically, not just detecting violations after the fact. Instead of alerting you that something drifted from policy requiring manual cleanup, a PSPM prevents or immediately corrects the drift as it happens.</p>

<p>Now wait a minute, if I already have a CNAPP, why do I need a PSPM?  A CNAPP provides broad visibility across cloud, workload, and application risk, including runtime and vulnerability context. But a PSPM like Turbot complements a CNAPP by ensuring cloud policies are always enforced preventing misconfigurations from occurring and persisting in your environment.</p>

<p>Without automated policy enforcement, you rely on people and processes, violations will always happen and detection and manual cleanup can take weeks or months.  In order to be secure, automatic policy enforcement needs to be in place.  Turbot makes this possible without having to write lots of custom code.</p>

<p>Automate prevention to reduce alert fatigue, eliminate manual cleanup, and close security gaps as soon as they appear. Learn more about Turbot here: <a href="https://fandf.co/4ps42Qm" rel="noopener noreferrer">https://fandf.co/4ps42Qm</a></p>

<p>Special thanks to Turbot for sponsoring this post!</p>

