---
Title: I Hired AWS SAs for 5 Years. Now That I've Left, I Can Tell You the Truth
Description: 
Author: Supreet Sethi
Date: 2025-11-27T21:41:45.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>tl;dr</strong>: I spent five years interviewing SA candidates at AWS. Comprehensive AWS training doesn't correlate with hire decisions. Production talk and clear thinking do. Now that I'm external, I can say it plainly: most interview prep is wasted effort.</p>




<h2>
  
  
  The Uncomfortable Truth
</h2>

<p>I was a Principal SA at AWS for two years. I sat on hiring panels as a Manager after that for three years. I made hire/no-hire decisions. I watched hundreds of candidates prepare the wrong way and then blame themselves for not studying hard enough.</p>

<p>Now that I've left, I can say this plainly without corporate-speak: <strong>your comprehensive AWS study guide is doing more harm than good.</strong></p>

<p>We don't care if you memorized the Well-Architected Framework. We didn't care if you studied all 200 AWS services. We cared if you could think clearly about tradeoffs, defend a decision under pressure, and admit when you didn't know something.</p>

<p>Yet every study guide, bootcamp, and LinkedIn thread screams the same lie: <strong>you need comprehensive AWS training</strong>.</p>

<p>You don't. In fact, it often works against you.</p>

<h2>
  
  
  What Happens in the Room
</h2>

<p>Your interviewer—especially if they're a Principal or staff engineer—already knows you haven't deployed infrastructure across 47 different AWS services. They're not testing breadth. They're testing <em>depth of thinking</em> and <em>production judgment</em>.</p>

<p>Here's what I paid attention to when candidates walked in:</p>

<p><strong>1. Production context—where's the real decision?</strong><br>
"I've built with EC2 and RDS" is fine. But I listened for <em>why</em> you made choices. Instance families? Why? Scaling problems? Walk me through one. Connection pooling in production? Perfect—now we're talking about real engineering.</p>

<p>The candidates I voted to hire almost always had one or two services they knew cold because they shipped something real. The ones reading from study guides tried to sound like they knew everything, and it was immediately obvious.</p>

<p><strong>2. Can you think on your feet?</strong><br>
I watched how people handled "how would you architect X?" If they recited a framework answer, I'd already mentally moved on to the next candidate. If they asked me clarifying questions, identified genuine tradeoffs, and said "I'd lean toward Fargate here because—," I'd lean forward.</p>

<p>The best moment in an interview is when a candidate says "I haven't used that, but here's my hypothesis based on..." That's the real move. That's how actual architects work. They don't know everything. They think.</p>

<p><strong>3. Honest gaps are revealing</strong><br>
I built my hiring questions specifically to find the difference between "studied this last week" and "this is my blind spot." When someone said, "I'd need to research that, but here's my starting point," they immediately became more credible than someone bulldozing with buzzwords.</p>

<p>The worst interviews? Candidates stalling on basic questions while clearly trying to remember what slide 47 of their bootcamp said.</p>

<h2>
  
  
  The Training Industrial Complex (Or: Why Your $500 Course Hurt Your Chances)
</h2>

<p>Five years on the hiring side has shown me a clear pattern: candidates who take comprehensive AWS training courses often <em>perform worse</em> than those who don't.</p>

<p>It's counterintuitive, but the data's there:</p>

<ul>
<li>Candidate studies 200 services -&gt; interviews with surface-level knowledge of many, depth in none</li>
<li>Candidate ships one project with EC2/RDS -&gt; interviews with real architectural thinking</li>
<li>Interview panel immediately spots rehearsed answers and deprioritizes</li>
<li>Candidate blames themselves: "I should have studied the ECS section more"</li>
</ul>

<p>Wrong diagnosis. Your study guide didn't help you; it convinced you that breadth matters more than depth.</p>

<p>The cruel cycle:</p>

<ul>
<li>Candidate feels anxious &gt; buys $500 "Complete AWS for SAs" course</li>
<li>Course teaches 200 services at surface level</li>
<li>Candidate memorizes facts they'll forget in 2 weeks</li>
<li>Interview panel spots the rehearsed answers within 10 minutes</li>
<li>Candidate doesn't get the job</li>
<li><strong>They blame themselves for not studying harder</strong></li>
</ul>

<p>I've sat through hundreds of interviews. The training industry profits off this anxiety loop while genuinely hurting candidate performance.</p>

<h2>
  
  
  What You Actually Need
</h2>

<p><strong>For the phone screen:</strong></p>

<ul>
<li>Production experience with at least one primary service (compute, database, or whatever)</li>
<li>Ability to discuss a real project: what worked, what didn't, what you'd change</li>
<li>Willingness to think through new problems without pretending you have all the answers</li>
<li>AWS fundamentals (VPC, IAM, Security Groups, GW, basic networking)—these are legitimate</li>
</ul>

<p><strong>For the technical discussion:</strong></p>

<ul>
<li>Deep knowledge of services <em>you've used</em>
</li>
<li>Framework for thinking about tradeoffs (cost vs. complexity, availability vs. simplicity, etc.)</li>
<li>Ability to pivot and explore adjacent solutions</li>
<li>Comfortable saying "I'd need to research that, but here's my starting hypothesis"</li>
</ul>

<p>That's genuinely it.</p>

<h2>
  
  
  The Real Test: A Rugby Social Network Migration
</h2>

<p>Let me walk you through an actual scenario AWS uses in interviews—and what separates the good architects from the ones who fail despite being technically sound.</p>

<p><strong>The problem:</strong> A rugby-focused social network with 2M users, growing 5x in the next 12 months. Currently on-premises in Dallas. Budget: $25K/month (15K hosting, 10K CDN via Akamai). Users spread: US 30%, Europe 50%, Asia 20%. They need to move to AWS.</p>

<p><strong>Bad SA answer</strong> (technically competent, but fails):</p>

<ul>
<li>"We'd use EC2 for compute, RDS for the database, S3 for video storage, CloudFront for CDN to replace Akamai, ElastiCache for sessions, Route53 for DNS, Auto Scaling groups, VPC with public/private subnets across three AZs, IAM roles for service-to-service auth, CloudWatch for monitoring, potentially Lambda for event processing, maybe Kinesis for real-time notifications, SQS for async jobs..."</li>
<li>Goes deeper: task definitions, RTO/RPO, security groups, NACLs, database backup strategies</li>
<li>Panel internally: "They know AWS components. But I have no idea if they can actually make a decision."</li>
<li>
<strong>Rejected</strong> (despite being technically correct)</li>
</ul>

<p><strong>Good SA answer</strong> (the one I hire):</p>

<ul>
<li>"Three regions: us-east-1 for US traffic, eu-west-1 for Europe, ap-southeast-1 for Asia. Each region gets an application tier and a database. We'll start with EC2 for the app layer—I'd use t3.large with ASGs to handle the 5x growth. RDS Multi-AZ in each region for data durability. We're replacing Akamai with CloudFront, which saves 10K/month immediately and actually improves latency in Europe and Asia."</li>
<li>Pauses. Asks clarifying questions: "Are writes distributed or mostly from content creators? How real-time are notifications? Is the data read-heavy or write-heavy?"</li>
<li>Listens to answer, then adapts: "If it's 80/20 read/write, we could add ElastiCache in front of RDS to reduce database load during peak traffic. If the user base is mostly mobile, we might want to think about API Gateway + Lambda instead of EC2 to reduce operational overhead—but EC2 gives us more predictable scaling for a 5x ramp."</li>
<li>When asked about databases: "RDS is safe here because we're relational. DynamoDB would require a schema redesign we don't have time for during a 5x growth. We validate that choice once we see real query patterns."</li>
<li>
<strong>Hired</strong> (someone who thinks, not someone who lists)</li>
</ul>

<p>The difference? The good SA made <em>three architectural decisions</em> (regional distribution, EC2 over serverless, RDS over NoSQL) with <em>clear reasoning</em> based on <em>constraints</em>. The bad SA listed every service and hoped some combination was right.</p>

<p>Then I tweaked the problem: "What if we need to migrate 500TB of video content and we can't afford 6 months of dual-running on-prem + cloud?"</p>

<p><strong>Good SA pivots in real time:</strong></p>

<ul>
<li>"We'd use AWS DataSync to replicate data continuously for the first month while we're in parallel. Once the on-prem systems reach end of life, we flip the DNS. But more importantly, we don't need S3 for video storage if we're staying with Akamai for CDN—Akamai has edge storage. So we replicate video to Akamai's origin once, we store backups in S3, but the application doesn't serve from S3. That keeps our architecture simple and maintains the budget."</li>
<li>
<strong>Still hired</strong> (adapted correctly)</li>
</ul>

<p><strong>Bad SA in same scenario:</strong></p>

<ul>
<li>Panics slightly. Falls back to memorized migration strategies.</li>
<li>Starts listing: "We could use AWS DataSync, AWS Direct Connect, AWS Snowball..."</li>
<li>Doesn't actually <em>choose</em> or <em>reason about cost and timeline</em>.</li>
<li>
<strong>Still rejected</strong> (couldn't think under pressure)</li>
</ul>

<h2>
  
  
  Why "Describe Every Component" Actually Kills Your Interview
</h2>

<p>When you try to faithfully describe every possible AWS component in your architecture, you're signaling something deadly to the panel: <strong>you're not making decisions. You're reciting a design.</strong></p>

<p>Here's what happens in the room:</p>

<p><strong>The candidate who maps components:</strong></p>

<ul>
<li>Lists RDS, EC2, S3, CloudFront, ElastiCache, Route53, IAM, VPC, subnets, security groups, NACLs, KMS, CloudWatch, CloudTrail...</li>
<li>For each one, provides correct technical details</li>
<li>Panel thinks: "They know AWS. But I don't know if they'd actually make this decision if costs were cut in half. Do they know why each component is there, or are they just pattern-matching to 'enterprise architecture template #42'?"</li>
</ul>

<p><strong>The candidate who makes three decisions with reasoning:</strong></p>

<ul>
<li>"Multi-region for latency (us-east, eu-west, ap-southeast). EC2+RDS+CloudFront at each region. No serverless yet—5x growth means we need predictable scaling."</li>
<li>For each decision, brief reasoning: cost, operational burden, time-to-migrate</li>
<li>When challenged ("What about DynamoDB?"), they know <em>why</em> they chose RDS instead</li>
<li>Panel thinks: "If requirements change, this person will adapt the design, not rebuild from a different AWS architecture template."</li>
</ul>

<p>The second person will design better systems under real pressure because they're reasoning about tradeoffs, not memorizing patterns.</p>

<p><strong>The interview is testing your ability to:</strong></p>

<ol>
<li>Make decisions under constraint (time, budget, team skills)</li>
<li>Explain decisions in 30 seconds (broad strokes)</li>
<li>Change your mind when new information arrives (during the tweaks)</li>
</ol>

<p>It's <em>not</em> testing whether you know all 200+ AWS services exist.</p>

<p>The cloud industry's obsession with breadth-first interview preparation isn't about better hiring. It's about justifying expensive training platforms while maintaining the illusion that AWS is harder than it actually is.</p>

<p><strong>AWS is not hard. Thinking clearly about tradeoffs is hard.</strong> And no course teaches that. Only production teaches that.</p>

<p>Now that I'm building hiring processes at a new company, I'm explicitly deprioritizing "studied the most material" in favor of "shipped the most thoughtful decisions." We ask fewer "how many EC2 instance types exist?" questions and more "walk me through a bad architectural decision you made and how you'd change it." That's actually predictive. That's the kind of candidate who will design better systems under real pressure.</p>

<h2>
  
  
  How to Actually Prepare (The Right Way)
</h2>

<p>Stop thinking in components. Start thinking in decisions.</p>

<p><strong>Pick a real migration or scaling problem.</strong> Use the rugby case as a template:</p>

<ul>
<li>2M users growing 5x in 12 months</li>
<li>Geographic distribution (not just US)</li>
<li>Budget constraint ($25K/month)</li>
<li>Existing infrastructure you're replacing</li>
</ul>

<p><strong>For your practice problem, write down:</strong></p>

<ol>
<li>
<strong>Three architectural decisions.</strong> Not "use EC2, RDS, and CloudFront." Write: "Multi-region distribution because 50% of traffic is Europe and latency matters. EC2 instead of Fargate because we need cost predictability during 5x growth and ops team is Linux-comfortable. RDS instead of DynamoDB because schema is relational and migration timeline is tight."</li>
<li>
<strong>Why each decision, in one sentence.</strong> If you need three sentences to justify it, you don't understand it well enough.</li>
<li>
<strong>What you'd change if...</strong> Cost cut in half? Timeline compressed to 6 weeks? User base only in US? Changing any constraint should trigger a design change. If your design doesn't flex, you don't own it.</li>
</ol>

<p><strong>In the interview, lead with the broad strokes:</strong></p>

<ul>
<li>"I'd go multi-region, EC2+RDS in each region, CloudFront for CDN, and here's why..."</li>
<li>Don't list components. List decisions.</li>
<li>When they ask about a component you didn't mention ("What about caching?"), say: "Good point—I'd add ElastiCache if query patterns show we need it, but we validate that in week two post-migration. Right now, I'm optimizing for certainty over premature optimization."</li>
</ul>

<p><strong>Be ready for the tweaks:</strong></p>

<ul>
<li>"What if migration takes 2 weeks, not 2 months?" → "We'd use DataSync and parallel run, but that changes our budget. What's the hard constraint?"</li>
<li>"What if the team is Node.js only, not infrastructure-familiar?" → "Serverless becomes more attractive. Lambda+RDS+API Gateway probably safer than managing ASGs."</li>
<li>"What if they won't pay for multi-region?" → "Single region initially, but we architect for multi-region addition—CloudFront origin failover, RDS read replicas we can promote. Route 53 health checks point to healthy region."</li>
</ul>

<p>This is how real architects work. You start with broad strokes, you defend those strokes, and you pivot when constraints change.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>If you're preparing for an AWS SA role and thinking "I'm not ready because I haven't studied enough," stop. You're probably overthinking this.</p>

<p>You're ready if you can talk about real production problems and think through new ones. That's the job.</p>

<p>Stop cramming. Start shipping. The interview will feel different when you have stories instead of slides.</p>




<p><strong>Curious if your production experience translates?</strong> Talk about a real architectural decision you made, no matter how small. That's your interview prep.</p>

