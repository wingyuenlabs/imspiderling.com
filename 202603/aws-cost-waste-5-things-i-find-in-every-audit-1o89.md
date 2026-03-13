---
Title: AWS Cost Waste: 5 Things I Find in Every Audit
Description: 
Author: Mariusz Gębala
Date: 2026-03-13T22:08:57.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>AWS cost waste</strong> is money spent on cloud resources that deliver zero value - orphaned volumes, logs stored forever, idle databases, and infrastructure nobody remembers deploying. In most accounts, it adds up to 27-35% of the total bill.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>#</th>
<th>Waste pattern</th>
<th>Typical annual cost</th>
<th>Fix effort</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Orphaned EBS volumes</td>
<td>$2,000+ per TB</td>
<td>1 Terraform line</td>
</tr>
<tr>
<td>2</td>
<td>CloudWatch logs without retention</td>
<td>15% of monthly bill</td>
<td>1 CLI command per log group</td>
</tr>
<tr>
<td>3</td>
<td>Unnecessary NAT Gateways</td>
<td>$1,166/year per 3-AZ setup</td>
<td>Conditional Terraform</td>
</tr>
<tr>
<td>4</td>
<td>gp2 volumes instead of gp3</td>
<td>20% of EBS spend</td>
<td>In-place migration, zero downtime</td>
</tr>
<tr>
<td>5</td>
<td>Over-provisioned RDS</td>
<td>$350+/month per idle instance</td>
<td>Environment-aware sizing</td>
</tr>
</tbody>
</table></div>

<p>According to a Flexera report, organizations waste 27% of their cloud spending. I have mixed feelings about this. In the audits I've conducted throughout my career, the result has more often been closer to 35%. Never mind the numbers. More important is the fact that almost no one notices wasted money until they actually check it.</p>

<p>Interestingly, these aren't some exotic edge cases. The same pattern usually repeats itself - five similar problems for every customer. In this article, I present a list of the most common cases.</p>

<h2>
  
  
  1. Orphaned EBS volumes
</h2>

<p>Did you have EC2 for testing? Great. Did you test everything you needed to? Even better. Did you shut down the instances? Well, you're clearly a professional who cares about costs. But wait... Did you really select "terminate EBS on shutdown"? Oh, no? And you've probably tested hundreds of instances over the last year? Let's do the math. Let's be optimistic, you had 50 of these instances. The cost is 0.08-0.10 USD per GB per month. Let's not bother with the math; I'll leave that to you.</p>

<p>One audit reported 2.4 TB of orphaned volumes (across three regions). $2.3k just went "into the cloud" and nobody actually noticed. But who's going to stop a rich man?</p>

<p><strong>Find them:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws ec2 describe-volumes <span class="se">\</span>
  <span class="nt">--filters</span> <span class="nv">Name</span><span class="o">=</span>status,Values<span class="o">=</span>available <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'Volumes[].{ID:VolumeId,Size:Size,Type:VolumeType,Created:CreateTime}'</span> <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<p>If that table has more than zero rows, you're paying for storage nobody uses.</p>

<p><strong>Prevent with Terraform:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aws_instance"</span> <span class="s2">"app"</span> <span class="p">{</span>
  <span class="nx">ami</span>           <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">ami_id</span>
  <span class="nx">instance_type</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">instance_type</span>

  <span class="nx">root_block_device</span> <span class="p">{</span>
    <span class="nx">volume_type</span>           <span class="p">=</span> <span class="s2">"gp3"</span>
    <span class="nx">delete_on_termination</span> <span class="p">=</span> <span class="kc">true</span>  <span class="c1"># This is the line that matters</span>
    <span class="nx">encrypted</span>             <span class="p">=</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>One line in your module. That's it. If your Terraform modules don't set this, every terminated instance leaves behind a volume that nobody will ever clean up.</p>

<h2>
  
  
  2. CloudWatch logs that never expire
</h2>

<p>We like having application logs, don't we? Let's log everything: Lambda, all ECS tasks, every API Gateway - EVERYTHING! Retention? And what if, in 15 years, someone asks why that ECS task crashed? Don't set it.</p>

<p>Logs are supposedly just text data. And it's hard to disagree, they are. It's worse when we log absolutely everything to CloudWatch. Although, no, that's not bad. What's bad is when we don't set any retention for those logs. Honestly, do you often find yourself reading logs older than a few days? Okay, that could still happen. But logs from a month ago? Probably once every 5 years would be useful, but even without that, you can survive. But even if you don't review them, remember - you have to pay for all those logs. It seems like peanuts, because it's only $0.03/GB. But they add up faster than you think. I've seen situations where CloudWatch was 15% of the monthly bill.</p>

<p>The conclusion is simple: if you let AWS automatically create log groups (which, contrary to appearances, is the default behavior), retention is infinite. Are you using Terraform? Then use the retention policy and you won't have to worry about unusually high bills.</p>

<p><strong>Find log groups with no retention:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws logs describe-log-groups <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'logGroups[?!retentionInDays].{Name:logGroupName,StoredBytes:storedBytes}'</span> <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<p><strong>Fix immediately:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Set 30-day retention on a specific log group</span>
aws logs put-retention-policy <span class="se">\</span>
  <span class="nt">--log-group-name</span> <span class="s2">"/aws/lambda/my-function"</span> <span class="se">\</span>
  <span class="nt">--retention-in-days</span> 30
</code></pre>

</div>



<p><strong>Prevent with Terraform:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Create the log group BEFORE the Lambda, so you control retention</span>
<span class="nx">resource</span> <span class="s2">"aws_cloudwatch_log_group"</span> <span class="s2">"lambda"</span> <span class="p">{</span>
  <span class="nx">name</span>              <span class="p">=</span> <span class="s2">"/aws/lambda/${var.function_name}"</span>
  <span class="nx">retention_in_days</span> <span class="p">=</span> <span class="mi">30</span>  <span class="c1"># ALWAYS set this</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  3. NAT Gateways nobody needs
</h2>

<p>Oh, I love this topic. You probably already know that overlay routing helps reduce the already high costs of implementing VM-Series. Just creating a NAT Gateway costs ~33 USD, and not even a single bit has passed through it. And imagine that you have to adhere to HA, meaning you install one NAT Gateway in each AZ, and you have three of them. It costs 100 USD just to install a NAT Gateway. Not to mention that you'll pay 0.045 USD per GB.</p>

<p>You know the problem? Most non-production environments seriously don't need three NAT Gateways. In fact, sometimes they don't need one at all.</p>

<p><strong>Check utilization:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check bytes processed by each NAT Gateway over the last 7 days</span>
aws cloudwatch get-metric-statistics <span class="se">\</span>
  <span class="nt">--namespace</span> AWS/NATGateway <span class="se">\</span>
  <span class="nt">--metric-name</span> BytesOutToDestination <span class="se">\</span>
  <span class="nt">--dimensions</span> <span class="nv">Name</span><span class="o">=</span>NatGatewayId,Value<span class="o">=</span>nat-0123456789abcdef0 <span class="se">\</span>
  <span class="nt">--start-time</span> <span class="si">$(</span><span class="nb">date</span> <span class="nt">-u</span> <span class="nt">-d</span> <span class="s1">'7 days ago'</span> +%Y-%m-%dT%H:%M:%S<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--end-time</span> <span class="si">$(</span><span class="nb">date</span> <span class="nt">-u</span> +%Y-%m-%dT%H:%M:%S<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--period</span> 604800 <span class="se">\</span>
  <span class="nt">--statistics</span> Sum
</code></pre>

</div>



<p><strong>Prevent with Terraform:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"environment"</span> <span class="p">{</span>
  <span class="nx">type</span> <span class="p">=</span> <span class="nx">string</span>
<span class="p">}</span>

<span class="c1"># 1 NAT Gateway in dev/staging, N in production</span>
<span class="nx">resource</span> <span class="s2">"aws_nat_gateway"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">count</span>         <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span> <span class="p">==</span> <span class="s2">"prod"</span> <span class="err">?</span> <span class="nx">length</span><span class="p">(</span><span class="nx">var</span><span class="p">.</span><span class="nx">azs</span><span class="p">)</span> <span class="err">:</span> <span class="mi">1</span>
  <span class="nx">allocation_id</span> <span class="p">=</span> <span class="nx">aws_eip</span><span class="p">.</span><span class="nx">nat</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
  <span class="nx">subnet_id</span>     <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">public</span><span class="p">[</span><span class="nx">count</span><span class="p">.</span><span class="nx">index</span><span class="p">].</span><span class="nx">id</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It's also worth checking whether your private subnets are using the internet at all. Maybe some of them only communicate with other AWS services? Endpoints are a much cheaper solution than NAT Gateways.</p>

<h2>
  
  
  4. gp2 volumes that should be gp3
</h2>

<p>This topic is also interesting. Basically, there's almost nothing you need to do here, and I see it practically everywhere.</p>

<p>Except I can guess where that comes from. It's common wisdom that newer something (in this case, a higher version is associated with something newer) means more expensive. So, someone who doesn't use AWS every day starts up EC2 and sees the choice between gp2 and gp3 EBS. They think, "I'll go with the older, cheaper one." Mmm... good luck! gp3 is about 20% cheaper than gp2, has 3,000 IOPS and 125 MB/s base throughput. Despite this, according to <a href="https://www.datadoghq.com/state-of-cloud-costs/" rel="noopener noreferrer">Datadog's State of Cloud Costs</a> report, gp2 accounts for 58% of EBS spending.</p>

<p>Generally, there's no scenario where gp2 is better - gp3 simply costs less and performs better. That's all.</p>

<p><strong>Find all gp2 volumes:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws ec2 describe-volumes <span class="se">\</span>
  <span class="nt">--filters</span> <span class="nv">Name</span><span class="o">=</span>volume-type,Values<span class="o">=</span>gp2 <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'Volumes[].{ID:VolumeId,Size:Size,State:State,Instance:Attachments[0].InstanceId}'</span> <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<p><strong>Migrate (no downtime):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws ec2 modify-volume <span class="nt">--volume-id</span> vol-0123456789abcdef0 <span class="nt">--volume-type</span> gp3
</code></pre>

</div>



<p>That's it. No shutdowns, no snapshots, no maintenance window. The migration occurs in the background while the volume remains connected and operational.</p>

<p><strong>Prevent with Terraform:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">variable</span> <span class="s2">"volume_type"</span> <span class="p">{</span>
  <span class="nx">type</span>    <span class="p">=</span> <span class="nx">string</span>
  <span class="nx">default</span> <span class="p">=</span> <span class="s2">"gp3"</span>

  <span class="nx">validation</span> <span class="p">{</span>
    <span class="nx">condition</span>     <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">volume_type</span> <span class="err">!</span><span class="p">=</span> <span class="s2">"gp2"</span>
    <span class="nx">error_message</span> <span class="p">=</span> <span class="s2">"Use gp3 instead of gp2. It's 20% cheaper with better baseline performance."</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>A validation block in the EC2 module rejects gp2 at plan time. This prevents anyone from accidentally deploying a costly option.</p>

<h2>
  
  
  5. Over-provisioned RDS instances
</h2>

<p>Time for dessert. Oh, how many companies are losing real money here. And let me give you an example. We have something to launch in production in eight months, so now let's use exactly the same parameters in the development environment that we'll use in production. So let's take a look at a <code>db.r6g.xlarge</code> instance. Cost? Let's say an average of $350. Needed for development? Yes, the same as a bicycle for a fish.</p>

<p>But this is still a rare case. In production, I've seen more than once someone set up RDS where the average CPU utilization is 5-8%. The last time such a move was in 2008, when the global crisis hit everyone.</p>

<p><strong>Check CPU utilization over the last 14 days:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws cloudwatch get-metric-statistics <span class="se">\</span>
  <span class="nt">--namespace</span> AWS/RDS <span class="se">\</span>
  <span class="nt">--metric-name</span> CPUUtilization <span class="se">\</span>
  <span class="nt">--dimensions</span> <span class="nv">Name</span><span class="o">=</span>DBInstanceIdentifier,Value<span class="o">=</span>my-database <span class="se">\</span>
  <span class="nt">--start-time</span> <span class="si">$(</span><span class="nb">date</span> <span class="nt">-u</span> <span class="nt">-d</span> <span class="s1">'14 days ago'</span> +%Y-%m-%dT%H:%M:%S<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--end-time</span> <span class="si">$(</span><span class="nb">date</span> <span class="nt">-u</span> +%Y-%m-%dT%H:%M:%S<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--period</span> 86400 <span class="se">\</span>
  <span class="nt">--statistics</span> Average <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<p><strong>Check for zero-connection databases:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws cloudwatch get-metric-statistics <span class="se">\</span>
  <span class="nt">--namespace</span> AWS/RDS <span class="se">\</span>
  <span class="nt">--metric-name</span> DatabaseConnections <span class="se">\</span>
  <span class="nt">--dimensions</span> <span class="nv">Name</span><span class="o">=</span>DBInstanceIdentifier,Value<span class="o">=</span>my-database <span class="se">\</span>
  <span class="nt">--start-time</span> <span class="si">$(</span><span class="nb">date</span> <span class="nt">-u</span> <span class="nt">-d</span> <span class="s1">'7 days ago'</span> +%Y-%m-%dT%H:%M:%S<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--end-time</span> <span class="si">$(</span><span class="nb">date</span> <span class="nt">-u</span> +%Y-%m-%dT%H:%M:%S<span class="si">)</span> <span class="se">\</span>
  <span class="nt">--period</span> 3600 <span class="se">\</span>
  <span class="nt">--statistics</span> Maximum <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<p><strong>Prevent with Terraform:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aws_db_instance"</span> <span class="s2">"main"</span> <span class="p">{</span>
  <span class="nx">instance_class</span>    <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span> <span class="p">==</span> <span class="s2">"prod"</span> <span class="err">?</span> <span class="s2">"db.r6g.large"</span> <span class="err">:</span> <span class="s2">"db.t4g.micro"</span>
  <span class="nx">multi_az</span>          <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span> <span class="p">==</span> <span class="s2">"prod"</span>
  <span class="nx">allocated_storage</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">environment</span> <span class="p">==</span> <span class="s2">"prod"</span> <span class="err">?</span> <span class="mi">100</span> <span class="err">:</span> <span class="mi">20</span>
  <span class="nx">storage_type</span>      <span class="p">=</span> <span class="s2">"gp3"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Environment-aware sizing. Dev gets the minimum, production gets what it needs. No more copying production configs into staging and forgetting about it.</p>

<h2>
  
  
  The pattern behind all five
</h2>

<p>You've probably noticed a key problem? Most of these topics don't apply to startups or small businesses that watch every cent twice. They apply to large companies. You know what's worse? That these large companies often look for savings on staffing in difficult times, not even on the things I mentioned. Nobody seems to pay attention to that. You know why? Because the staff has shrunk...</p>

<p>And it's not like I see this everywhere. Usually, the teams I work with are really well-equipped with AWS. It's just that there's a real shortage of resources to devote to cost optimization in the cloud.</p>

<h2>
  
  
  What to do about it
</h2>

<p>Simply take these ready-made commands and run them on your environment. It'll take you maybe 10 minutes, and you might save someone or yourself a full-time job.</p>

<p>If you want to go deeper into the topic - identify over-allocated computing resources, audit data transfer patterns, check liability coverage - that's a longer conversation. But start with these five. They can be checked for free, and most can be fixed for free.</p>

<p>I built <a href="https://github.com/gebalamariusz/cloud-audit" rel="noopener noreferrer">cloud-audit</a> to automate the security side of these checks - it runs 30+ checks in ~12 seconds. For cost specifically, the five CLI commands above are your starting point.</p>




<p><em>Originally published at <a href="https://haitmg.pl/blog/aws-cost-waste-audit-findings/" rel="noopener noreferrer">haitmg.pl</a></em></p>

