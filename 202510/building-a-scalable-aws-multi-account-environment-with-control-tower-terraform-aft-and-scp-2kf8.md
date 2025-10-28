---
Title: Building a Scalable AWS Multi-Account Environment with Control Tower, Terraform AFT, and SCP Guardrails
Description: 
Author: Santanu Das
Date: 2025-10-28T22:07:15.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🔖 Overview
</h2>

<p>This document outlines the AWS account structure, governance, and control strategy used across our organization.<br>
It describes how AWS Control Tower, AFT, and SCPs interact to provide a compliant, secure, and automated multi-account landing zone — aligning with AWS Well-Architected and CIS Benchmarks.</p>
<h2>
  
  
  🌍 Organizational Structure
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>OU</th>
<th>Description</th>
<th>Primary Accounts</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Security OU</strong></td>
<td>Core compliance and security monitoring</td>
<td>
<code>Log Archive</code>, <code>Audit</code>
</td>
</tr>
<tr>
<td><strong>Internal OU</strong></td>
<td>Shared platform services and IAM root</td>
<td>
<code>Shared Services</code>, <code>IAM/Root</code>
</td>
</tr>
<tr>
<td><strong>NPR Networking OU</strong></td>
<td>Non-Production networking environment</td>
<td>
<code>Internal Comms</code>, <code>External Comms</code>
</td>
</tr>
<tr>
<td><strong>PRD Networking OU</strong></td>
<td>Production networking environment</td>
<td>
<code>Internal Comms</code>, <code>External Comms</code>
</td>
</tr>
<tr>
<td><strong>Deprecated OU</strong></td>
<td>Legacy accounts (no new workloads)</td>
<td>Various (read-only)</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  🧭 Regional Strategy
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Region</th>
<th>Rationale</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>IAM Identity Center (SSO)</strong></td>
<td><code>us-east-1</code></td>
<td>Global endpoint for AWS SSO and Organizations</td>
</tr>
<tr>
<td><strong>Control Tower Management Account</strong></td>
<td>
<code>us-east-1</code> backend</td>
<td>Required by AWS</td>
</tr>
<tr>
<td><strong>All Member Accounts</strong></td>
<td>
<code>eu-west-2</code> (London)</td>
<td>Primary data residency &amp; workload region</td>
</tr>
<tr>
<td><strong>Backup / DR</strong></td>
<td>
<code>eu-west-1</code> (Ireland)</td>
<td>Optional failover region</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  💠 Conceptual Flow Diagram
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>                 ┌──────────────────────────────────────────────┐
                 │               Management Account             │
                 │ <span class="o">(</span>AWS Organizations + Control Tower + AFT<span class="o">)</span>    │
                 └──────────────────────────────────────────────┘
                                   │
                 ┌──────────────────────────────────────────────┐
                 │ Control Tower Landing Zone                   │
                 │ • Security OU <span class="o">(</span>LogArchive + Audit<span class="o">)</span>           │
                 │ • Creates baseline guardrails <span class="o">(</span>AWS-managed<span class="o">)</span>  │
                 │ • Delegates to AFT <span class="k">for </span>account provisioning  │
                 └──────────────────────────────────────────────┘
                                   │ Management Acct <span class="o">(</span>us-east-1<span class="o">)</span>
              ┌────────────────────┴────────────────────┐
              │                                         │
    ┌──────────────────────────────┐          ┌──────────────────────────────┐
    │ Account Factory <span class="k">for </span>TF <span class="o">(</span>AFT<span class="o">)</span> │          │ AWS Organizations <span class="o">(</span>Org Root<span class="o">)</span> │
    │ • GitOps: account requests   │          │ • OU hierarchy <span class="o">(</span>SEC, INT,    │
    │ • Customizations pipelines   │          │   NPR, PRD, DEPRECATED<span class="o">)</span>      │
    │ • Baselines, tagging, roles  │          │ • SCPs attached per OU       │
    └──────────────────────────────┘          └──────────────────────────────┘
              │                                         │
              │                                         │
              │              ┌────────────────────────────────────────┐
              │              │ SCP layer <span class="o">(</span>Preventive Guardrails<span class="o">)</span>      │
              │              │ • Enforced at Org root / OU level      │
              │              │ • Deny/allow APIs before IAM evaluated │
              │              │ • Prevents config drift or unsafe ops  │
              │              └────────────────────────────────────────┘
              │
     ┌────────┴─────────┐
     │ Enrolled Account │
     │ <span class="o">(</span>e.g. Internal,  │
     │ NPR, PRD, etc.<span class="o">)</span>  │
     └──────────────────┘
</code></pre>

</div>

<h2>
  
  
  🧩 Control Tower &amp; AFT Integration
</h2>
<h3>
  
  
  Landing Zone
</h3>

<ul>
<li>Deployed via AWS Control Tower in the management account.</li>
<li>Establishes:

<ul>
<li>AWS Organizations</li>
<li>Log Archive and Audit accounts</li>
<li>Baseline guardrails (AWS-managed)</li>
</ul>
</li>
</ul>
<h3>
  
  
  Account Factory for Terraform (AFT)
</h3>

<ul>
<li>Provides GitOps-based account lifecycle management.</li>
<li>Each account is provisioned using Terraform pipelines that:

<ul>
<li>Enroll the account in Control Tower</li>
<li>Apply OU-specific baselines (e.g., Config, logging)</li>
<li>Tag accounts automatically</li>
</ul>
</li>
<li>AFT runs in its own AFT Management Account.</li>
</ul>
<h3>
  
  
  Account Hierarchy Diagram
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>Management Account
 ├── AWS Control Tower <span class="o">(</span>Landing Zone<span class="o">)</span>
 ├── AFT Pipelines <span class="o">(</span>Account Factory <span class="k">for </span>Terraform<span class="o">)</span>
 └── AWS Organizations <span class="o">(</span>Root OU<span class="o">)</span>
      ├── Security OU
      ├── Internal OU
      ├── NPR Networking OU
      ├── PRD Networking OU
      └── Deprecated OU
</code></pre>

</div>

<h2>
  
  
  🧰 Networking &amp; Shared Services Strategy
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Service</th>
<th>Owning Account</th>
<th>Scope</th>
<th>Sharing Mechanism</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Transit Gateway (TGW)</strong></td>
<td>Internal Comms (per env)</td>
<td>Environment-specific</td>
<td>AWS RAM (within the env)</td>
</tr>
<tr>
<td><strong>Internet Gateway (IGW)</strong></td>
<td>External Comms (per env)</td>
<td>Environment-specific</td>
<td>Single point of entry - not shared</td>
</tr>
<tr>
<td><strong>NAT Gateway (NGW)</strong></td>
<td>Shared Services</td>
<td>Cross-environment</td>
<td>Single point of exit - not shared</td>
</tr>
<tr>
<td><strong>Network Firewall (NFW)</strong></td>
<td>Shared Services</td>
<td>Cross-environment</td>
<td>AWS RAM + TGW routing</td>
</tr>
<tr>
<td><strong>VPC Endpoint Services</strong></td>
<td>Shared Services</td>
<td>Org-wide</td>
<td>Route53 Reslover</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  🔐 Governance &amp; Security Controls
</h2>

<p>Governance is enforced using <strong>three layers</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Type</th>
<th>Enforcement Mechanism</th>
</tr>
</thead>
<tbody>
<tr>
<td>Preventive</td>
<td><strong>Service Control Policies (SCPs)</strong></td>
<td>AWS Organizations</td>
</tr>
<tr>
<td>Detective</td>
<td><strong>Config / Security Hub / GuardDuty</strong></td>
<td>Audit account</td>
</tr>
<tr>
<td>Proactive</td>
<td><strong>CloudFormation Hooks / AFT Customizations</strong></td>
<td>Terraform baselines</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  🚫 Core SCP Pack (Preventive Guardrails)
</h2>
<h3>
  
  
  1️⃣ Deny Root User Access
</h3>

<p>Prevents use of root credentials in any account.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"DenyRootUser"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Deny"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"StringLike"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"aws:PrincipalArn"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:iam::*:root"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  2️⃣ Restrict Regions (EU + us-east-1)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"DenyOutsideApprovedRegionsExceptIdentity"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Deny"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"NotAction"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"iam:*"</span><span class="p">,</span><span class="s2">"organizations:*"</span><span class="p">,</span><span class="s2">"route53:*"</span><span class="p">,</span><span class="s2">"sso:*"</span><span class="p">,</span><span class="s2">"support:*"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"StringNotEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"aws:RequestedRegion"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"eu-west-1"</span><span class="p">,</span><span class="w"> </span><span class="s2">"eu-west-2"</span><span class="p">,</span><span class="w"> </span><span class="s2">"us-east-1"</span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  3️⃣ Deny Unapproved Network Creation
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"DenyUnapprovedNetworking"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Deny"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"ec2:CreateInternetGateway"</span><span class="p">,</span><span class="s2">"ec2:AttachInternetGateway"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"ec2:CreateNatGateway"</span><span class="p">,</span><span class="s2">"ec2:CreateVpcPeeringConnection"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"ec2:CreateTransitGateway"</span><span class="p">,</span><span class="s2">"ec2:DeleteTransitGateway"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  4️⃣  Restrict Service Creation
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Resource</th>
<th>Allowed Account(s)</th>
<th>SCP Condition</th>
</tr>
</thead>
<tbody>
<tr>
<td>TGW (Transit Gateway)</td>
<td>Internal Comms (PRD/NPR)</td>
<td>
<code>aws:PrincipalAccount</code> = INC IDs</td>
</tr>
<tr>
<td>IGW (Internet Gateway)</td>
<td>External Comms (PRD/NPR)</td>
<td>
<code>aws:PrincipalAccount</code> = EXC IDs</td>
</tr>
<tr>
<td>NFW (Network Firewall)</td>
<td>Shared Services</td>
<td>
<code>aws:PrincipalAccount</code> = SSV ID</td>
</tr>
<tr>
<td>NGW (NAT Gateway)</td>
<td>Shared Services</td>
<td>
<code>aws:PrincipalAccount</code> = SSV ID</td>
</tr>
<tr>
<td>VPC Endpoint Services</td>
<td>Shared Services</td>
<td>
<code>aws:PrincipalAccount</code> = SSV ID</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  5️⃣ Tag Enforcement
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RequireStandardTags"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Deny"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"ec2:RunInstances"</span><span class="p">,</span><span class="s2">"rds:CreateDBInstance"</span><span class="p">,</span><span class="s2">"s3:CreateBucket"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Null"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"aws:RequestTag/Environment"</span><span class="p">:</span><span class="w"> </span><span class="s2">"true"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"aws:RequestTag/Owner"</span><span class="p">:</span><span class="w"> </span><span class="s2">"true"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  6️⃣  Deny Org Tampering
</h3>

<p>Protects core logging and control-plane resources<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"DenyOrgTampering"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Deny"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"organizations:LeaveOrganization"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"cloudtrail:StopLogging"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"config:StopConfigurationRecorder"</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  7️⃣  Deprecated OU Policy
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Sid"</span><span class="p">:</span><span class="w"> </span><span class="s2">"DenyNewCreates"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Deny"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"*"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"StringEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"aws:ResourceTag/Environment"</span><span class="p">:</span><span class="w"> </span><span class="s2">"deprecated"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  🧠 Control Layers Explained
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Description</th>
<th>Managed By</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Control Tower</strong></td>
<td>Creates baseline governance (OU, guardrails, log archive, audit)</td>
<td>AWS</td>
</tr>
<tr>
<td><strong>AFT</strong></td>
<td>Automates account provisioning, tagging, baseline controls</td>
<td>Platform team</td>
</tr>
<tr>
<td><strong>SCPs</strong></td>
<td>Prevent actions that violate org standards (region, network, security)</td>
<td>Org root</td>
</tr>
<tr>
<td><strong>Delegated Security Accounts</strong></td>
<td>Detect &amp; monitor compliance (Config, GuardDuty)</td>
<td>Security team</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  🪜 Deployment Workflow
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Step</th>
<th>Action</th>
<th>Tool</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Enable Control Tower (Landing Zone)</td>
<td>Console / CLI</td>
</tr>
<tr>
<td>2</td>
<td>Bootstrap AFT management</td>
<td>Terraform</td>
</tr>
<tr>
<td>3</td>
<td>Create OUs + SCPs</td>
<td>Terraform</td>
</tr>
<tr>
<td>4</td>
<td>Submit account requests via AFT</td>
<td>GitOps</td>
</tr>
<tr>
<td>5</td>
<td>Apply OU-specific baselines</td>
<td>Terraform</td>
</tr>
<tr>
<td>6</td>
<td>Validate security controls</td>
<td>AWS Config / Security Hub</td>
</tr>
<tr>
<td>7</td>
<td>Continuous compliance</td>
<td>Detective + Proactive guardrails</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  🧩 Future Enhancements
</h2>

<ul>
<li>Add Policy Staging OU for testing new SCPs.</li>
<li>Integrate Proactive Controls (CloudFormation hooks).</li>
<li>Automate SCP compliance drift detection using AWS Config custom rules.</li>
<li>Add organizational backup plans (AWS Backup delegated admin).</li>
</ul>

