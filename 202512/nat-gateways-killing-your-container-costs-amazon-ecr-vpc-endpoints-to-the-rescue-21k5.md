---
Title: NAT Gateways Killing Your Container Costs? Amazon ECR VPC endpoints to the Rescue
Description: 
Author: Hardeep Singh Tiwana
Date: 2025-12-19T21:32:27.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Picture this</strong>. Your AWS bill hits, and there it is: <strong>$10K in NAT Gateway charges</strong> for 3 NAT GWs in <strong><code>us-east-1</code></strong>. You started to dig in, and see <strong>~$8K</strong> comes from <strong>NatGateway-Bytes</strong> (<strong>Data Processed</strong>) alone, assuming most of it tied to ECR image pulls. I've helped teams spot this exact issue using Cost Explorer and VPC Flow logs, watching container deployments quietly eat budgets. The solution? Amazon ECR VPC endpoints. They <strong>dropped NAT bills by &gt;75%</strong> in one setup I worked on. Let's walk through spotting it, the math, and the flow change.</p>

<p><strong>TL;DR:</strong> </p>

<blockquote>
<ul>
<li>ECR image pulls through NAT Gateways cost $0.045/GB.</li>
<li>VPC Interface Endpoints cost $0.01/GB (<strong>78% cheaper</strong>).</li>
<li>Real example: ~$8K/month → ~$2K/month = <strong>~$70K annual savings</strong>.</li>
</ul>
</blockquote>




<h2>
  
  
  💡 Key Takeaways
</h2>

<p><strong>The Problem:</strong> NAT Gateways charge $0.045/GB for data processing. For ECR-heavy workloads, this adds up fast, as our <strong>example case</strong> shows $8,010/month in data processing charges alone!</p>

<p><strong>The Solution:</strong> Deploy three VPC endpoints to <strong>route ECR traffic privately</strong>:</p>

<ol>
<li>
<strong>ECR API Interface Endpoint</strong> (<strong><code>com.amazonaws.&lt;region&gt;.ecr.api</code></strong>)

<ul>
<li>Handles authentication and image manifests</li>
<li>Cost: ~$22/month per AZ + minimal data charges</li>
<li>Required: Must deploy in each AZ for high availability</li>
</ul>
</li>
<li>
<strong>ECR Docker Interface Endpoint</strong> (<strong><code>com.amazonaws.&lt;region&gt;.ecr.dkr</code></strong>)

<ul>
<li>Handles Docker pull/push commands</li>
<li>Cost: ~$22/month per AZ + minimal data charges</li>
<li>Required: Must deploy in each AZ for high availability</li>
</ul>
</li>
<li>
<strong>S3 Gateway Endpoint</strong> (<strong><code>com.amazonaws.&lt;region&gt;.s3</code></strong>) <strong>⭐ THE MOST CRITICAL ONE</strong>

<ul>
<li>Handles actual image layer downloads (99%+ of your data!)</li>
<li>Cost: <strong>$0.00 (FREE!)</strong>
</li>
<li>
<strong>Required:</strong> Without this, your image layers still hit NAT Gateways</li>
</ul>
</li>
</ol>

<ul>
<li>
<strong>The Savings:</strong> For 178,000 GB/month of ECR traffic:

<ul>
<li>
<strong>Before:</strong> $8,108.55/month (NAT Gateways)</li>
<li>
<strong>After:</strong> $1,823.80/month (VPC Endpoints)</li>
<li>
<strong>Savings:</strong> $6,284.75/month <strong>(77.5%) = $75,417/year</strong>
</li>
</ul>


</li>

</ul>

<p><strong>Why This Works?:</strong> ECR stores Docker image layers in S3. The free S3 Gateway endpoint handles 95%+ of your data transfer, while the two paid Interface endpoints handle control plane operations. All three work together to eliminate NAT Gateway data processing charges.</p>

<p><strong>Implementation Time:</strong> ~30 minutes with Terraform, plus 48 hours to validate savings in Cost Explorer.</p>

<p><strong>Critical Success Factor:</strong> You MUST deploy all three endpoints. Deploying only the ECR endpoints without the S3 Gateway endpoint will save you almost nothing because the bulk of your data will still flow through NAT Gateways</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpcmr1j07m4pj0mlga1wy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpcmr1j07m4pj0mlga1wy.png" alt="Compare both models" width="800" height="340"></a></p>




<h2>
  
  
  Let's start with the Brutal Math: NAT vs. Endpoints Head-to-Head
</h2>

<p>Think standard 3-AZ VPC with private subnets and container workloads. NAT charges <strong><code>$0.045</code></strong> per hour per AZ plus <strong><code>$0.045</code></strong> per GB processed. Endpoints run <strong><code>$0.01</code></strong> per hour per ENI and <strong><code>$0.01</code></strong> per GB. Much better for high volume.</p>

<p><strong>Note:</strong> AWS requires 2 VPC interface endpoints per AZ for complete ECR private access: <strong><code>ecr.api</code></strong>, <strong><code>ecr.dkr</code></strong>, and <strong><code>s3</code></strong> (layers), making it 6 ENIs total in a 3-AZ setup. The S3 Gateway endpoint modifies route tables and creates no ENIs. If you like to read more on this, follow links at the end of this post.</p>

<ul>
<li>
<strong><code>ecr.api</code></strong> → Interface endpoint (ENI per AZ)</li>
<li>
<strong><code>ecr.dkr</code></strong> → Interface endpoint (ENI per AZ)</li>
<li>
<strong><code>s3</code></strong> → Gateway endpoint (NO ENIs, modifies route tables)</li>
</ul>

<h3>
  
  
  NAT Gateway vs VPC Endpoints Cost Comparison
</h3>

<p>Configuration: 3 AZs with 3 NAT Gateways vs 3 VPC Endpoints</p>

<h4>
  
  
  VPC Endpoint Configuration:
</h4>

<ul>
<li>com.amazonaws..ecr.api (Interface) - $0.01/hour per AZ + $0.01/GB</li>
<li>com.amazonaws..ecr.dkr (Interface) - $0.01/hour per AZ + $0.01/GB</li>
<li>com.amazonaws..s3 (Gateway) - FREE (no hourly or data charges)</li>
</ul>

<h4>
  
  
  NAT Gateway Configuration:
</h4>

<ul>
<li>3 NAT Gateways (one per AZ) - $0.045/hour each + $0.045/GB</li>
</ul>

<p>Here's the model, scaled to $8K spend as data baseline (730 hours a month, 9 endpoints: 3 per AZ for ECR API, Docker, and S3):</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Data Volume (GB/mo)</th>
<th>NAT Cost ($)</th>
<th>VPC Endpoint Cost ($)</th>
<th>Monthly Savings ($)</th>
<th>Savings %</th>
</tr>
</thead>
<tbody>
<tr>
<td>100</td>
<td>103.05</td>
<td>44.80</td>
<td>58.25</td>
<td>56.5%</td>
</tr>
<tr>
<td>500</td>
<td>121.05</td>
<td>48.80</td>
<td>72.25</td>
<td>59.7%</td>
</tr>
<tr>
<td>1,000</td>
<td>143.55</td>
<td>53.80</td>
<td>89.75</td>
<td>62.5%</td>
</tr>
<tr>
<td>5,000</td>
<td>323.55</td>
<td>93.80</td>
<td>229.75</td>
<td>71.0%</td>
</tr>
<tr>
<td>10,000</td>
<td>548.55</td>
<td>143.80</td>
<td>404.75</td>
<td>73.8%</td>
</tr>
<tr>
<td>50,000</td>
<td>2,348.55</td>
<td>543.80</td>
<td>1,804.75</td>
<td>76.8%</td>
</tr>
<tr>
<td>100,000</td>
<td>4,598.55</td>
<td>1,043.80</td>
<td>3,554.75</td>
<td>77.3%</td>
</tr>
<tr>
<td><strong>178,000</strong></td>
<td><strong>8,108.55</strong></td>
<td><strong>1,823.80</strong></td>
<td><strong>6,284.75</strong></td>
<td><strong>77.5%</strong></td>
</tr>
</tbody>
</table></div>

<p>Total NAT spend declines like a falling rock, at production scale, you will see ROI in days.</p>




<h2>
  
  
  Example use case with assumptions
</h2>

<p>Assume we have 3 NAT Gateways in <strong><code>us-east-1</code></strong> processing 178,000 GB of ECR traffic monthly.</p>

<p>Cost Breakdown for Total Monthly Cost: $8,108.55</p>

<ol>
<li>
<p>NAT Gateway Hourly Charges: $98.55 </p>

<ul>
<li>$0.045 per hour × 3 NAT Gateways × 730 hours/month</li>
<li>This covers the provisioning cost for maintaining 3 NAT Gateways (one per AZ)</li>
</ul>
</li>
<li>
<p>Data Processing Charges: $8,010.00</p>

<ul>
<li>$0.045 per GB × 178,000 GB</li>
<li>This is the charge for processing all data flowing through the NAT Gateways</li>
</ul>
</li>
</ol>

<ul>
<li>
<p>Per NAT Gateway:</p>

<ul>
<li>Hourly cost: $32.85/month per gateway</li>
<li>Data processing (if evenly distributed): $2,670.00/month per gateway</li>
</ul>

<p><strong>Important Note:</strong> The data processing charge of $8,010 represents the vast majority (98.8%) of our assumed total NAT Gateway costs. Since we're processing ECR (Elastic Container Registry) traffic within the same region, we won't incur additional data transfer charges for the traffic itself, but the NAT Gateway data processing fee still applies.</p>


</li>

</ul>

<h2>
  
  
  Prerequisites:
</h2>

<ul>
<li>Private subnets with NAT Gateway access</li>
<li>ECR repositories in the same region</li>
<li>Security groups allowing HTTPS (443) from workloads</li>
</ul>




<h2>
  
  
  Hunt Down Those Hidden ECR Pull Fees
</h2>

<p>Start in AWS Cost Explorer. In Group by, select Dimension <strong>Usage Type</strong>, Filter to <strong>Service:</strong> <code>EC2 - Other</code> and <strong>Usage type group:</strong> for <strong><code>EC2: NAT Gateway - Data Processed</code></strong> and <strong><code>EC2: NAT Gateway - Running Hours</code></strong>. You'll see <strong><code>NatGateway-Bytes</code></strong> racking up that e.g. $8K at $0.045 per GB, plus <strong><code>NatGateway-Hours</code></strong> for the $0.045 hourly per AZ hit.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feoob6pqjjgvqj854f053.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feoob6pqjjgvqj854f053.png" alt="Cost Explorer Filters" width="628" height="1388"></a></p>

<p><strong>For proof</strong>, enable <strong>VPC Flow Logs</strong> on your subnets. Filter for port <strong><code>443</code></strong> traffic to <strong><code>ecr.api</code></strong> or <strong><code>ecr.dkr</code></strong> domains (Specifically, Look for destination port 443 traffic to IP addresses in the ECR service IP ranges, available via AWS IP ranges JSON). </p>

<p>Do you see private subnet bytes flooding NAT ENIs? That's the problem. Every pull sends a small request out via NAT, fetches metadata, then hauls gigabytes back, doubling up on processing fees. (If it is an Inter-AZ hop, it add $0.01 per GB more. Caught this pattern adding ~$3000 a month extra in a recent cluster review.)</p>

<h2>
  
  
  Using VPC Flow Logs to Track and Validate ECR Traffic Costs
</h2>

<p>Before deploying VPC endpoints, you need proof that ECR is actually consuming your NAT Gateway bandwidth. After deployment, you need validation that traffic shifted correctly. VPC Flow Logs provide both.</p>

<h3>
  
  
  Step 1: Enable VPC Flow Logs
</h3>

<p>Enable Flow Logs on your private subnets where container workloads run:</p>

<p><strong>Via AWS CLI:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws ec2 create-flow-logs <span class="se">\</span>
  <span class="nt">--resource-type</span> Subnet <span class="se">\</span>
  <span class="nt">--resource-ids</span> subnet-xxxxx subnet-yyyyy subnet-zzzzz <span class="se">\</span>
  <span class="nt">--traffic-type</span> ALL <span class="se">\</span>
  <span class="nt">--log-destination-type</span> cloud-watch-logs <span class="se">\</span>
  <span class="nt">--log-group-name</span> /aws/vpc/flowlogs <span class="se">\</span>
  <span class="nt">--deliver-logs-permission-arn</span> arn:aws:iam::ACCOUNT_ID:role/flowlogsRole
</code></pre>

</div>



<p><strong>Via Terraform:</strong> : <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/flow_log" rel="noopener noreferrer">Follow link to see the module on terraform website</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight terraform"><code><span class="k">resource</span> <span class="s2">"aws_flow_log"</span> <span class="s2">"private_subnets"</span> <span class="p">{</span>
  <span class="nx">iam_role_arn</span>    <span class="p">=</span> <span class="nx">aws_iam_role</span><span class="p">.</span><span class="nx">flow_logs</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">log_destination</span> <span class="p">=</span> <span class="nx">aws_cloudwatch_log_group</span><span class="p">.</span><span class="nx">flow_logs</span><span class="p">.</span><span class="nx">arn</span>
  <span class="nx">traffic_type</span>    <span class="p">=</span> <span class="s2">"ALL"</span>
  <span class="nx">vpc_id</span>          <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Step 2: Identify Top HTTPS Destinations
</h3>

<p>Run this CloudWatch Logs Insights query to find your highest-volume HTTPS destinations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>fields @timestamp, srcAddr, dstAddr, dstPort, bytes, action
| filter dstPort = 443
| filter interfaceId like /eni-/
| stats sum(bytes) as totalBytes by dstAddr
| sort totalBytes desc
| limit 50
</code></pre>

</div>



<p>This shows which destinations consume the most bandwidth on port 443. The top destinations are likely S3 IPs (for ECR image layers).</p>

<h3>
  
  
  Step 3: Identify S3 and ECR Service IP Ranges
</h3>

<p>VPC Flow Logs show IP addresses, not domain names. Download AWS's IP ranges to identify both S3 and ECR traffic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Download AWS IP ranges</span>
curl <span class="nt">-o</span> ip-ranges.json https://ip-ranges.amazonaws.com/ip-ranges.json

<span class="c"># Inspect services for your region</span>
jq <span class="nt">-r</span> <span class="s1">'.prefixes[] | select(.region=="us-east-1") | .service'</span> ip-ranges.json | <span class="nb">sort</span> <span class="nt">-u</span>
</code></pre>

</div>



<p>Once you know the correct service values, narrow it down, since ECR doesn't have a designated value, we use AMAZON:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Once you know the correct service values, narrow it down, for example:</span>
jq <span class="nt">-r</span> <span class="s1">'.prefixes[] | select(.service=="AMAZON" or .service=="S3" and .region=="us-east-1") | .ip_prefix'</span> ip-ranges.json
</code></pre>

</div>



<p><strong>Example IP ranges for <code>us-east-1</code></strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>44.223.121.0/24
44.223.122.0/24
98.80.195.0/25
98.80.238.0/23
3.5.0.0/19
1.178.4.0/24
</code></pre>

</div>



<p><strong>You will see &gt;95% of traffic for S3:</strong> </p>

<ul>
<li>S3 (where ECR stores image layers - 95%+ of your traffic)</li>
<li>ECR (API and Docker registry - &lt;5% of your traffic)
<strong>Why This Matters:</strong> Your 178,000 GB/month is primarily S3 traffic (image layer downloads), not ECR API calls. You must track S3 IPs to see the real cost impact!</li>
</ul>

<p><em>(Always check the current AWS IP ranges JSON for your specific region)</em></p>

<h3>
  
  
  Step 4: Calculate NAT Gateway ECR+S3 Traffic
</h3>

<p>Filter Flow Logs for traffic to BOTH S3 and ECR IPs through NAT Gateway ENIs:</p>

<p><strong>NOTE:</strong> </p>

<ol>
<li>Do NOT copy paste as it is, update <code>filter dstAddr like</code> line to match the range from previus command output.</li>
<li>Replace <code>/^3\.5\./ or dstAddr like /^52\.94\./ or dstAddr like /^3\.5\./</code> with real IPs you want to look for
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>fields @timestamp, srcAddr, dstAddr, dstPort, bytes, interfaceId
| filter dstPort = 443
| filter interfaceId like /eni-/ and action = "ACCEPT"
| filter dstAddr like /^3\.5\./ or dstAddr like /^52\.94\./ or dstAddr like /^3\.5\./
| stats sum(bytes) as totalBytes by interfaceId, dstAddr
| sort totalBytes desc
</code></pre>

</div>



<p><strong>Identify NAT Gateway ENIs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws ec2 describe-nat-gateways <span class="nt">--region</span> us-east-1 <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'NatGateways[].{NatGatewayId:NatGatewayId, NetworkInterfaceIds:NatGatewayAddresses[].NetworkInterfaceId}'</span> <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<p>Cross-reference the ENI IDs from your query results with NAT Gateway ENIs.<br>
<strong>💡 Pro Tip:</strong> The top destination IPs by bytes will be S3 ranges, not ECR ranges. This confirms that S3 Gateway endpoint is critical for cost savings!</p>
<h3>
  
  
  Step 5: Calculate Monthly Cost Impact
</h3>

<p>From your Flow Logs query results:</p>

<ol>
<li>
<strong>Sum total bytes</strong> through NAT Gateway ENIs to S3 + ECR IPs</li>
<li>
<strong>Convert to GB:</strong> totalBytes / 1,000,000,000 (AWS uses decimal GB)</li>
<li>
<strong>Calculate cost:</strong> GB × $0.045</li>
</ol>

<p><strong>Cost Calculation Example:</strong></p>

<ul>
<li>
<strong>Flow Logs show:</strong> 191,102,976,000 bytes to S3/ECR</li>
<li>
<strong>Convert:</strong> 191,102,976,000 / 1,000,000,000 = 191.10 GB</li>
<li>For 178,000 GB/month: 178,000 × $0.045 = <strong>$8,010/month</strong>
</li>
</ul>

<p><strong>Traffic Breakdown (typical):</strong></p>

<ul>
<li>S3 image layers: ~177,850 GB (99.91%)</li>
<li>ECR API calls: ~50 GB (0.03%)</li>
<li>ECR Docker registry: ~100 GB (0.06%)</li>
</ul>
<h3>
  
  
  Step 6: Validate After VPC Endpoint Deployment
</h3>

<p>After deploying VPC endpoints, confirm traffic shifted to private IPs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>fields @timestamp, srcAddr, dstAddr, dstPort, bytes, interfaceId
| filter dstPort = 443
| filter dstAddr like /^10\./
| filter interfaceId like /eni-/
| stats sum(bytes) as totalBytes by interfaceId
| sort totalBytes desc
</code></pre>

</div>



<p>What you should see:</p>

<ul>
<li>✓ Traffic now goes to private 10.x.x.x IPs (VPC endpoint ENIs)</li>
<li>✓ NAT Gateway ENIs show minimal S3/ECR traffic</li>
<li>✓ Total bytes shifted from NAT to VPC endpoints</li>
</ul>




<h3>
  
  
  ❌ But this validation method has problems ❌
</h3>

<p>⚠️ The above given filter only filters for RFC 1918 private IPs (10.0.0.0/8), but VPC endpoints use different address ranges:</p>

<p>Gateway Endpoints (S3, DynamoDB)</p>

<ul>
<li>Use prefix list routes (<code>pl-xxx</code>), not destination IPs in flow logs</li>
<li>dstAddr shows the actual S3 service IP (public range like <code>52.x.x.x</code>), not private</li>
<li>Flow log records bypass the interfaceId filter entirely because they hit the prefix list route directly</li>
</ul>

<p>Interface Endpoints (<code>ECR.api</code>, <code>ECR.dkr</code>, etc.)</p>

<ul>
<li>Use <strong>PrivateLink IPs</strong> in the VPC CIDR (e.g., <code>10.0.x.x</code> if your VPC is <code>10.0.0.0/16</code>)</li>
<li>dstAddr shows the endpoint ENI IP (private), but only if your VPC CIDR starts with 10.</li>
</ul>

<h4>
  
  
  So what would correct validation queries look like?
</h4>

<p><strong>1. Interface Endpoints (ECR, etc.) - Check PrivateLink traffic</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">fields</span> <span class="o">@</span><span class="nb">timestamp</span><span class="p">,</span> <span class="n">srcAddr</span><span class="p">,</span> <span class="n">dstAddr</span><span class="p">,</span> <span class="n">dstPort</span><span class="p">,</span> <span class="n">bytes</span><span class="p">,</span> <span class="n">interfaceId</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">dstPort</span> <span class="o">=</span> <span class="mi">443</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">dstAddr</span> <span class="k">like</span> <span class="o">/^</span><span class="mi">10</span><span class="err">\</span><span class="p">.</span><span class="o">/</span>  <span class="o">#</span> <span class="n">Your</span> <span class="n">VPC</span> <span class="n">CIDR</span> <span class="k">range</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">interfaceId</span> <span class="k">like</span> <span class="o">/</span><span class="n">eni</span><span class="o">-/</span>
<span class="o">|</span> <span class="n">stats</span> <span class="k">sum</span><span class="p">(</span><span class="n">bytes</span><span class="p">)</span> <span class="k">as</span> <span class="n">totalBytes</span> <span class="k">by</span> <span class="n">dstAddr</span><span class="p">,</span> <span class="n">interfaceId</span>
<span class="o">|</span> <span class="n">sort</span> <span class="n">totalBytes</span> <span class="k">desc</span>
</code></pre>

</div>



<p>⚠️ Only works if your VPC CIDR is <code>10.x.x.x</code>. <strong>Replace with your actual CIDR (e.g., 172.16. or 192.168.)</strong>.</p>

<p><strong>2. Gateway Endpoints (S3) - Check prefix list bypass</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">fields</span> <span class="o">@</span><span class="nb">timestamp</span><span class="p">,</span> <span class="n">srcAddr</span><span class="p">,</span> <span class="n">dstAddr</span><span class="p">,</span> <span class="n">dstPort</span><span class="p">,</span> <span class="n">bytes</span><span class="p">,</span> <span class="n">interfaceId</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">dstPort</span> <span class="o">=</span> <span class="mi">443</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">s3BucketName</span> <span class="o">!=</span> <span class="nv">""</span> <span class="k">or</span> <span class="n">dstAddr</span> <span class="k">like</span> <span class="o">/</span><span class="n">s3</span><span class="err">\</span><span class="p">.</span><span class="o">/</span>  <span class="o">#</span> <span class="n">S3</span> <span class="n">traffic</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">interfaceId</span> <span class="k">like</span> <span class="o">/</span><span class="n">nat</span><span class="o">-/</span> <span class="o">==</span> <span class="k">false</span>  <span class="o">#</span> <span class="k">Not</span> <span class="n">NAT</span> <span class="n">ENIs</span>
<span class="o">|</span> <span class="n">stats</span> <span class="k">sum</span><span class="p">(</span><span class="n">bytes</span><span class="p">)</span> <span class="k">as</span> <span class="n">totalBytes</span> <span class="k">by</span> <span class="n">dstAddr</span>
<span class="o">|</span> <span class="n">sort</span> <span class="n">totalBytes</span> <span class="k">desc</span>
</code></pre>

</div>



<p><strong>3. 🎯 NAT Gateway traffic drop (The real validation)🎯</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="n">fields</span> <span class="o">@</span><span class="nb">timestamp</span><span class="p">,</span> <span class="n">srcAddr</span><span class="p">,</span> <span class="n">dstAddr</span><span class="p">,</span> <span class="n">dstPort</span><span class="p">,</span> <span class="n">bytes</span><span class="p">,</span> <span class="n">interfaceId</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">dstPort</span> <span class="o">=</span> <span class="mi">443</span>
<span class="o">|</span> <span class="n">filter</span> <span class="n">interfaceId</span> <span class="k">like</span> <span class="o">/</span><span class="n">nat</span><span class="o">-/</span>
<span class="o">|</span> <span class="n">stats</span> <span class="k">sum</span><span class="p">(</span><span class="n">bytes</span><span class="p">)</span> <span class="k">as</span> <span class="n">totalBytes</span> <span class="k">by</span> <span class="n">interfaceId</span>
<span class="o">|</span> <span class="n">sort</span> <span class="n">totalBytes</span> <span class="k">desc</span>
</code></pre>

</div>



<p><strong>Before endpoints:</strong> High bytes on NAT ENIs<br>
<strong>After endpoints:</strong> Bytes drop significantly on those same ENIs.</p>
<h4>
  
  
  🎯 What success looks like 🎯
</h4>

<p><strong>BEFORE endpoints:</strong></p>

<ul>
<li>NAT ENI: 150 GB to s3.us-east-1.amazonaws.com</li>
<li>NAT ENI: 25 GB to 123456789012.dkr.ecr.us-east-1.amazonaws.com</li>
</ul>

<p><strong>AFTER endpoints:</strong></p>

<ul>
<li>NAT ENI: 5 GB (mostly external APIs)</li>
<li>Interface ENI: 25 GB to 10.0.2.100 (ECR.dkr endpoint)</li>
<li>S3 traffic: Prefix list route (no NAT ENI)</li>
</ul>

<p>Key metric: <strong>NAT ENI bytes drop</strong>. That's your validation.</p>

<p>The <code>/^10\./</code> filter only catches interface endpoints and only if your VPC uses that range. Use the NAT traffic reduction query instead.</p>

<p><strong>Validate endpoint ENI IDs:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># ECR API endpoint ENIs</span>
aws ec2 describe-vpc-endpoints <span class="nt">--region</span> us-east-1 <span class="se">\</span>
  <span class="nt">--filters</span> <span class="s2">"Name=service-name,Values=com.amazonaws.us-east-1.ecr.api"</span> <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'VpcEndpoints[*].NetworkInterfaceIds'</span> <span class="se">\</span>
  <span class="nt">--output</span> table

<span class="c"># ECR Docker endpoint ENIs</span>
aws ec2 describe-vpc-endpoints <span class="nt">--region</span> us-east-1 <span class="se">\</span>
  <span class="nt">--filters</span> <span class="s2">"Name=service-name,Values=com.amazonaws.us-east-1.ecr.dkr"</span> <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'VpcEndpoints[*].NetworkInterfaceIds'</span> <span class="se">\</span>
  <span class="nt">--output</span> table

<span class="c"># S3 Gateway endpoint (no ENIs - modifies route tables)</span>
aws ec2 describe-vpc-endpoints <span class="nt">--region</span> us-east-1 <span class="se">\</span>
  <span class="nt">--filters</span> <span class="s2">"Name=service-name,Values=com.amazonaws.us-east-1.s3"</span> <span class="se">\</span>
  <span class="nt">--query</span> <span class="s1">'VpcEndpoints[*].[VpcEndpointId,VpcEndpointType,RouteTableIds]'</span> <span class="se">\</span>
  <span class="nt">--output</span> table
</code></pre>

</div>



<h3>
  
  
  Step 7: Correlate with Cost Explorer
</h3>

<p>Confirm the cost impact in AWS Cost Explorer:</p>

<ol>
<li>
<strong>Navigate to:</strong> Cost Explorer → Cost &amp; Usage Reports</li>
<li>
<strong>Group by:</strong> Usage Type</li>
<li>
<strong>Filter Service:</strong> EC2 - Other</li>
<li>
<strong>Look for:</strong>

<ul>
<li>
<strong>NatGateway-Bytes</strong> (should drop ~75%)</li>
<li>
<strong>VpcEndpoint-Bytes</strong> (should increase proportionally)</li>
<li>
<strong>Time range:</strong> Compare 2 weeks before vs 2 weeks after deployment</li>
</ul>
</li>
</ol>

<p><strong>Expected results:</strong></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>- NAT Gateway data processing: $8,010 → ~$2,000 (75% reduction)
- VPC Endpoint data processing: $0 → ~$1,780
- Net savings: ~$6,285/month
</code></pre>

</div>
<h4>
  
  
  Understanding the Three-Endpoint Architecture
</h4>

<p><strong>Why you need all three endpoints:</strong></p>

<ol>
<li>
<strong>ECR API Interface Endpoint</strong> (<code>com.amazonaws.us-east-1.ecr.api</code>)

<ul>
<li>Handles authentication, authorization, image manifests</li>
<li>Low data volume (~50 GB/month)</li>
<li>Cost: $21.90/month (3 AZs × 730 hrs × $0.01) + ~$0.50 data</li>
</ul>
</li>
<li>
<strong>ECR Docker Interface Endpoint</strong> (<code>com.amazonaws.us-east-1.ecr.dkr</code>)

<ul>
<li>Handles Docker pull/push commands, layer discovery</li>
<li>Low data volume (~100 GB/month)</li>
<li>
<strong>Cost:</strong> $21.90/month (3 AZs × 730 hrs × $0.01) + ~$1.00 data</li>
</ul>
</li>
<li>
<strong>S3 Gateway Endpoint</strong> (<code>com.amazonaws.us-east-1.s3</code>) ← <strong>THE CRITICAL ONE</strong>

<ul>
<li>Handles actual image layer downloads (99%+ of your data!)</li>
<li>High data volume (~177,850 GB/month)</li>
<li>
<strong>Cost: $0.00 (FREE!)</strong> ← This is where your savings come from!
<strong>Without the S3 Gateway endpoint</strong>, your image layer downloads would still hit NAT Gateways even with ECR endpoints deployed!</li>
</ul>
</li>
</ol>

<p><strong>Pro Tips for Flow Logs Analysis</strong></p>

<ul>
<li>
<strong>✓ Track S3 IPs, not just ECR IPs</strong> - S3 is where 95%+ of ECR data flows </li>
<li>
<strong>✓ Enable Flow Logs on private subnets only</strong> - Reduces log volume and costs </li>
<li>
<strong>✓ Use CloudWatch Logs Insights</strong> - Best for ad-hoc queries and quick analysis </li>
<li>
<strong>✓ Consider Amazon Athena</strong> - Better for large-scale historical analysis </li>
<li>
<strong>✓ Set up CloudWatch alarms</strong> - Alert on unexpected NAT traffic spikes </li>
<li>
<strong>✓ Tag your resources</strong> - Makes NAT Gateways and VPC endpoints easier to identify </li>
<li>
<strong>✓ Factor in Flow Logs cost</strong> - Approximately $0.50/GB ingested to CloudWatch </li>
<li>
<strong>✓ Aggregate by 5-minute intervals</strong> - Reduces log volume without losing insights </li>
<li>
<strong>✓ Monitor for 2-4 weeks</strong> - Ensures you capture full deployment cycles and traffic patterns</li>
</ul>


<h2>
  
  
  Before and After: Understanding The Traffic Flow
</h2>

<ul>
<li>
<strong>Before:</strong> ECS Tasks → NAT Gateway → Internet → ECR/S3 (expensive)</li>
<li>
<strong>After:</strong> ECS Tasks → VPC Endpoints → AWS Private Network → ECR/S3 (optimized)</li>
</ul>
<h3>
  
  
  Before endpoints
</h3>

<ul>
<li>A pod in a private subnet hits NAT Gateway for every ECR pull</li>
<li>Request goes outbound to the internet, ECR API replies inbound through NAT processing, then Docker layers stream back with massive GBs.</li>
<li>Flow Logs show megabytes to NAT ENIs. Cost Explorer's <code>NatGateway-Bytes</code> balloons to $8K.</li>
</ul>
<h3>
  
  
  After, deploy
</h3>

<ul>
<li>
<strong><code>com.amazonaws.&lt;region&gt;.ecr.api</code></strong> and <strong><code>.ecr.dkr</code></strong> endpoints in each private subnet per AZ, turn on private DNS. </li>
<li>Pod traffic goes straight to the endpoint ENI via PrivateLink, no NAT or internet gateway. </li>
<li>AWS backbone handles the rest, ECR layers flow free within the region.</li>
<li>Flow Logs shift: zero NAT to ECR domains, all bytes on private 10.x endpoint IPs. </li>
<li>In Cost Explorer, NAT usage drop like a falling rock. </li>
<li>Look for usage types containing <strong><code>VpcEndpoint-Hours</code></strong> and <strong><code>VpcEndpoint-Bytes</code></strong> under the <strong>VPC</strong> service to confirm it is starting to show costs with much smaller amounts as compared to what NAT was showing.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5ngxpvlu0bvo7haec33r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5ngxpvlu0bvo7haec33r.png" alt="VPC Endpoint Costs" width="652" height="1382"></a></p>

<p>Rolled this out on a Kubernetes fleet processing 178,000 GB/mo ECR traffic. NAT crashed from $10K ($8K data processed) to $2K for services that still need it. Endpoints totaled $1.8k. Filter <strong>Data Transfer</strong> + <strong>EC2</strong> in Cost Explorer you will see <strong>EC2: NAT Gateway - Data Processed</strong> costs drop sharply, while <strong>VpcEndpoint-Hours</strong> + <strong>VpcEndpoint-Bytes</strong> take over at $0.01/GB.</p>
<h2>
  
  
  Cost After VPC Interface Endpoints: $$1,823.80/month
</h2>
<h3>
  
  
  New Cost Breakdown:
</h3>
<h4>
  
  
  NAT Gateway Costs:
</h4>

<ul>
<li>Hourly charges: $98.55 (gateways remain for other traffic)</li>
<li>Data processing: $0.00 (ECR traffic now bypasses NAT entirely)
#### VPC Interface Endpoint Costs:</li>
<li>Hourly charges: $43.80 (2 endpoints × 3 AZs × 730 hours × $0.01/hour)</li>
<li>Data processing: $1,780.00 (178,000 GB × $0.01/GB)
## The Impact:
💰 Monthly Savings: $6,284.75/month (77.5%) 💰 Annual Savings: $75,417.00/year</li>
</ul>
<h3>
  
  
  What You Need to Deploy:
</h3>
<h4>
  
  
  Required Interface Endpoints (per AZ):
</h4>

<ul>
<li>✅ com.amazonaws.us-east-1.ecr.api - For ECR API calls</li>
<li>✅ com.amazonaws.us-east-1.ecr.dkr - For Docker registry operations
#### Required Gateway Endpoint (VPC-wide - For ECR image layer storage - FREE):</li>
<li>✅ com.amazonaws.us-east-1.s3 - Deploy once per VPC (not per AZ)</li>
</ul>
<h4>
  
  
  A quick and dirty example Terraform code"
</h4>


<div class="highlight js-code-highlight">
<pre class="highlight terraform"><code><span class="k">resource</span> <span class="s2">"aws_vpc_endpoint"</span> <span class="s2">"s3"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span>            <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">service_name</span>      <span class="p">=</span> <span class="s2">"com.amazonaws.</span><span class="k">${</span><span class="kd">var</span><span class="p">.</span><span class="nx">aws_region</span><span class="k">}</span><span class="s2">.s3"</span>
  <span class="nx">vpc_endpoint_type</span> <span class="p">=</span> <span class="s2">"Gateway"</span>
  <span class="nx">route_table_ids</span>   <span class="p">=</span> <span class="nx">aws_route_table</span><span class="p">.</span><span class="nx">private</span><span class="p">[*].</span><span class="nx">id</span>
  <span class="nx">policy</span>            <span class="p">=</span> <span class="k">data</span><span class="p">.</span><span class="nx">aws_iam_policy_document</span><span class="p">.</span><span class="nx">s3_ecr_access</span><span class="p">.</span><span class="nx">json</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"s3-gateway"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="k">resource</span> <span class="s2">"aws_vpc_endpoint"</span> <span class="s2">"ecr-dkr-endpoint"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span>              <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">service_name</span>        <span class="p">=</span> <span class="s2">"com.amazonaws.</span><span class="k">${</span><span class="kd">var</span><span class="p">.</span><span class="nx">aws_region</span><span class="k">}</span><span class="s2">.ecr.dkr"</span>
  <span class="nx">vpc_endpoint_type</span>   <span class="p">=</span> <span class="s2">"Interface"</span>
  <span class="nx">private_dns_enabled</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="nx">security_group_ids</span>  <span class="p">=</span> <span class="p">[</span><span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">ecs_task</span><span class="p">.</span><span class="nx">id</span><span class="p">]</span>
  <span class="nx">subnet_ids</span>          <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">private</span><span class="p">[*].</span><span class="nx">id</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"ecr-dkr"</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="k">resource</span> <span class="s2">"aws_vpc_endpoint"</span> <span class="s2">"ecr-api-endpoint"</span> <span class="p">{</span>
  <span class="nx">vpc_id</span>              <span class="p">=</span> <span class="nx">aws_vpc</span><span class="p">.</span><span class="nx">main</span><span class="p">.</span><span class="nx">id</span>
  <span class="nx">service_name</span>        <span class="p">=</span> <span class="s2">"com.amazonaws.</span><span class="k">${</span><span class="kd">var</span><span class="p">.</span><span class="nx">aws_region</span><span class="k">}</span><span class="s2">.ecr.api"</span>
  <span class="nx">vpc_endpoint_type</span>   <span class="p">=</span> <span class="s2">"Interface"</span>
  <span class="nx">private_dns_enabled</span> <span class="p">=</span> <span class="kc">true</span>
  <span class="nx">security_group_ids</span>  <span class="p">=</span> <span class="p">[</span><span class="nx">aws_security_group</span><span class="p">.</span><span class="nx">ecs_task</span><span class="p">.</span><span class="nx">id</span><span class="p">]</span>
  <span class="nx">subnet_ids</span>          <span class="p">=</span> <span class="nx">aws_subnet</span><span class="p">.</span><span class="nx">private</span><span class="p">[*].</span><span class="nx">id</span>

  <span class="nx">tags</span> <span class="p">=</span> <span class="p">{</span>
    <span class="nx">Name</span> <span class="p">=</span> <span class="s2">"ecr-api"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Validation:
</h2>

<ul>
<li>Validate with: <code>nslookup ecr.api.us-east-1.amazonaws.com</code>
</li>
<li>Should resolve to private <code>10.x.x.x</code> addresses, not public IPs.</li>
</ul>

<h2>
  
  
  💡 Pro Tip: The S3 Gateway endpoint is critical but FREE.
</h2>

<ul>
<li>Add a free <strong>S3 Gateway endpoint</strong> for ECR layer storage access. While ECR endpoints handle API calls, image layers are stored in S3. The Gateway endpoint ensures this traffic also bypasses NAT at zero cost, so don't skip it. ECR stores image layers in S3, and without this endpoint, your layer downloads will still hit NAT Gateways!</li>
</ul>

<h2>
  
  
  Why Does This Work So Well?
</h2>

<p>The key is <strong>data processing rate difference</strong>:</p>

<ul>
<li>NAT Gateway: $0.045/GB</li>
<li>VPC Endpoint: $0.01/GB (78% cheaper per GB)</li>
</ul>

<p>Plus, VPC endpoints provide:</p>

<ul>
<li>Better security - Traffic never leaves AWS network</li>
<li>Lower latency - Direct path to ECR</li>
<li>Higher reliability - No internet gateway dependency</li>
<li>Simplified architecture - Private subnets can pull images directly</li>
</ul>

<h3>
  
  
  Another Implementation detail to keep in mind:
</h3>

<p>Your NAT Gateways stay in place for other internet-bound traffic (software updates, external APIs, etc.), but all ECR image pulls route through the VPC endpoints instead. This is a configuration change, not a replacement and you get the best of both worlds.</p>

<h2>
  
  
  Troubleshooting:
</h2>

<ul>
<li>DNS not resolving privately? Enable "Private DNS" on endpoints ✅</li>
<li>Still seeing NAT charges? Check security group rules allow 443 inbound ✅</li>
<li>Pulls timing out? Verify subnet route tables don't force internet gateway ✅</li>
<li>Endpoint not appearing in Cost Explorer? Wait 24-48 hours for billing data to populate; check under Service: "VPC" ✅</li>
<li>Validate endpoint status: <code>aws ec2 describe-vpc-endpoints --filters "Name=service-name,Values=com.amazonaws.us-east-1.ecr.api"</code> ✅</li>
</ul>

<h2>
  
  
  Troubleshooting Flow Logs Analysis
</h2>

<p><strong>Issue: Can't find NAT Gateway ENIs in Flow Logs</strong></p>

<ul>
<li>✅ Verify Flow Logs are enabled on the correct subnets</li>
<li>✅ Check that traffic-type is set to ALL (not just <code>ACCEPT</code> or <code>REJECT</code>)</li>
<li>✅ Wait 10-15 minutes after enabling for data to populate</li>
</ul>

<p><strong>Issue: S3/ECR IP ranges don't match traffic</strong></p>

<ul>
<li>✅ AWS IP ranges change periodically - always download the latest JSON</li>
<li>✅ Some regions have additional IP ranges not in the main prefixes</li>
<li>✅ Check for both IPv4 and IPv6 ranges if your VPC supports dual-stack</li>
<li>✅ Remember: Most traffic will be to S3 IPs, not ECR IPs!</li>
</ul>

<p><strong>Issue: Traffic still shows NAT Gateway after endpoint deployment</strong></p>

<ul>
<li>✅ Verify private_dns_enabled = true on Interface endpoints</li>
<li>✅ Check security groups allow port 443 from workload subnets</li>
<li>✅ Confirm route tables don't have explicit routes forcing internet gateway</li>
<li>✅ Verify S3 Gateway endpoint is associated with correct route tables</li>
<li>✅ Test DNS resolution: nslookup <code>ecr.api.us-east-1.amazonaws.com</code> should return <code>10.x.x.x</code>
</li>
<li>✅ Test S3 access: nslookup <code>s3.us-east-1.amazonaws.com</code> should resolve (Gateway endpoints don't change DNS)</li>
</ul>

<p><strong>Issue: Cost Explorer doesn't match Flow Logs calculations</strong></p>

<ul>
<li>✅ Flow Logs show raw bytes; Cost Explorer uses decimal GB (1 GB = 1,000,000,000 bytes)</li>
<li>✅ Cost Explorer has 24-48 hour delay for billing data</li>
<li>✅ Ensure you're comparing the same time periods</li>
<li>✅ Check for data transfer charges vs data processing charges</li>
<li>✅ Remember: S3 Gateway endpoint traffic is FREE, so you won't see it in VPC endpoint costs</li>
</ul>

<p><strong>Issue: Only seeing small data volumes to ECR IPs</strong></p>

<ul>
<li>✅ This is NORMAL! ECR API/Docker traffic is &lt;5% of total</li>
<li>✅ The bulk of your data goes to S3 IPs (image layers)</li>
<li>✅ If you're only filtering for ECR IPs, you're missing 95%+ of the traffic</li>
<li>✅ Update your query to include S3 IP ranges</li>
</ul>

<h2>
  
  
  Reality Check
</h2>

<p>This assumes full traffic shift (realistic for ECR-only optimization). Background NAT persists for other internet traffic. Monitor your Cost Explorer's NAT Gateway data processing charges weekly for the first month. You should see a 75%+ drop if ECR is your primary NAT consumer. If not, investigate other high-volume services using VPC Flow Logs.</p>

<h2>
  
  
  Next Steps
</h2>

<ol>
<li>Run Cost Explorer analysis (5 min)</li>
<li>Deploy endpoints in non-prod (30 min)</li>
<li>Validate with test pulls (10 min)</li>
<li>Monitor for 48 hours</li>
<li>Roll to production during maintenance window</li>
<li>Track Cost Explorer for 2 weeks to confirm savings</li>
</ol>

<p>Ready to fix it? Create the endpoints in console or Terraform, tag them like Name:ecr-api for tracking, test docker pull once private DNS propagates. Budget relief comes fast. Seen this work for you? Share in the comments.</p>

<blockquote>
<p>References:</p>

<ul>
<li><a href="https://aws.amazon.com/blogs/compute/setting-up-aws-privatelink-for-amazon-ecs-and-amazon-ecr/" rel="noopener noreferrer">Setting up AWS PrivateLink for Amazon ECS, and Amazon ECR</a></li>
<li><a href="https://aws.amazon.com/blogs/containers/using-vpc-endpoint-policies-to-control-amazon-ecr-access/" rel="noopener noreferrer">Using VPC endpoint policies to control Amazon ECR access</a></li>
</ul>
</blockquote>

