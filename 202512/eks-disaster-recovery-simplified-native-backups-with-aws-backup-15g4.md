---
Title: EKS Disaster Recovery, Simplified: Native Backups with AWS Backup
Description: 
Author: Shittu Sulaimon (Barry)
Date: 2025-12-21T21:53:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>For years, platform engineers have shared the same quiet nightmare: backing up EKS at scale. As clusters grow and teams stay lean, disaster recovery stops being optional and becomes mandatory.</p>

<p>As clusters grow and teams stay lean, disaster recovery stops being a “nice to have” and turns into a hard requirement. And until recently, meeting that requirement usually meant either of these things: Velero pain, custom scripts, manually managed S3 buckets, constant anxiety about whether your PVs actually match your cluster state.</p>

<p>It worked but it was fragile, time consuming, and easy to get wrong.</p>

<p><strong>The turning point: November 10, 2025</strong></p>

<p>Last month, AWS finally closed a long standing gap by introducing native Amazon EKS support in AWS Backup. And this isn’t a minor feature drop, it’s a real shift from DIY backup engineering to managed reliability.</p>

<p>Here’s why this matters.</p>

<p>This isn't just a small update; it's a shift from "manual assembly" to "managed simplicity." Here is why this native integration is a game changer compared to the old way of doing things:</p>

<p><strong>1. Composite Recovery Points (the missing piece):</strong> Before now, EKS backups were fragmented:</p>

<ul>
<li>Cluster configs in one place</li>
<li>EBS snapshots somewhere else</li>
<li>Hope holding everything together</li>
</ul>

<p>Now: AWS Backup captures cluster state + persistent storage (EBS, EFS, S3) as a single, consistent recovery point.</p>

<p>No more guessing if your data and manifests are in sync.</p>

<p><strong>2. One pane of glass (finally):</strong> If you already use AWS Backup for EC2, RDS, or DynamoDB, this will feel familiar.</p>

<ul>
<li>EKS backups now live in the same service, with the same workflows, policies, and visibility.</li>
<li>No extra controllers.</li>
<li>No per-cluster Velero babysitting.</li>
</ul>

<p><strong>3. Policy-driven, not script-driven:</strong> Instead of CronJobs living inside your clusters, you define Backup Plans:</p>

<p><em>“Back up every 6 hours. Retain for 30 days.”</em></p>

<p>AWS handles scheduling, encryption, immutability, and lifecycle management automatically.</p>

<p>This is what “set and forget” is supposed to look like.</p>

<p><strong>4. Restores without the stress:</strong> Restores no longer feel like a gamble.You can:</p>

<ul>
<li>Restore an entire cluster</li>
<li>Recover a single namespace</li>
<li>Roll back individual persistent volumes</li>
<li>Even restore into a brand-new EKS cluster provisioned as part of the process</li>
</ul>

<p>That’s real operational confidence.</p>

<p>Why this matters right now</p>

<p><strong>This isn’t just about accidental deletions. Native EKS backup gives you a safety net for:</strong></p>

<ul>
<li>Cluster upgrades (1.30 → 1.31 without fear)</li>
<li>AMI rollouts that don’t go as planned</li>
<li>Security patches you have to apply</li>
<li>Kubernetes API changes that can break workloads</li>
</ul>

<p>If you’re running production EKS, this is one of those features that quietly changes how you sleep at night.</p>

<p>AWS didn’t just add a backup option; they removed a whole category of operational stress.</p>

<p><strong>Practical Guide: Enabling Native EKS Backups with AWS Backup</strong><br>
You already have an existing cluster in AWS EKS, to take backup of your cluster,</p>

<p>Navigate to your AWS Backup resource, then go to settings, and then select configure resource</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faosiltx8cgl0ouy583yh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faosiltx8cgl0ouy583yh.png" alt=" " width="800" height="341"></a></p>

<p>After enabling the protected resources and have included AWS EKS, then we can proceed to protected resources on the console, click on <strong>Create On-Demand Backup</strong> </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F26wf1c72u2qdl94yswzj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F26wf1c72u2qdl94yswzj.png" alt=" " width="800" height="303"></a></p>

<p>I have created a custom role <strong>EKS-BACKUP-ROLE-EXAMPLE</strong> that is used for backup, where i have attached some important policies: </p>

<ul>
<li>
<strong>AWSBackupServiceRolePolicyForBackup policy</strong> </li>
<li>
<strong>AWSBackupServiceRolePolicyForRestores policy</strong> </li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkf3idq0m51msvo1idt3m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkf3idq0m51msvo1idt3m.png" alt=" " width="800" height="662"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuoi7qnhe9ukg5incp9tg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuoi7qnhe9ukg5incp9tg.png" alt=" " width="800" height="637"></a></p>

<p>Now we can see that the backup is in progress</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb88uptsjk8298b39fsc9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb88uptsjk8298b39fsc9.png" alt=" " width="800" height="209"></a></p>

<p>You can verify your cluster backup is complete using the BAckup page or from the EKS page</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzoqo050ouy1gedryn6tu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzoqo050ouy1gedryn6tu.png" alt=" " width="800" height="129"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkkmc6v25h0hmc0s4txt5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkkmc6v25h0hmc0s4txt5.png" alt=" " width="800" height="331"></a></p>

<p>That is how to back up; now moving to the <strong>Restore process</strong><br>
Go to the <strong>backup resource</strong> in AWS then navigate to <strong>protected resources</strong>, click on the <strong>Resource ID</strong> of the cluster you wish to back up, choose the composite recovery point and click on the restore button</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdauixjqk92f146ntwjzf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdauixjqk92f146ntwjzf.png" alt=" " width="800" height="360"></a></p>

<p>During the restore workflow, AWS Backup presents several recovery options. You can choose the restore scope, either recovering the entire EKS cluster or targeting a specific namespace. You also select a restore destination, which can be the original source cluster, an existing EKS cluster, or a brand-new cluster created as part of the restore process.</p>

<p>In this walkthrough, I chose to restore into a newly created EKS cluster to demonstrate the full capabilities of native EKS recovery and validate how well the feature supports clean, isolated disaster recovery scenarios.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fguer8ohxzfccwft09200.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fguer8ohxzfccwft09200.png" alt=" " width="800" height="397"></a></p>

<p>In the next step, you’re asked to select the storage resources to include in the restore. In this case, no persistent storage was attached to my workloads, but AWS Backup also supports restoring data backed by Amazon EBS, EFS, and S3, ensuring that application data and cluster state remain consistent.</p>

<p>Once the restore is initiated, AWS Backup begins recreating the cluster based on the selected recovery options, provisioning a new EKS environment as specified. This doesn’t replace disciplined GitOps workflows or careful upgrade strategies, but it finally gives platform teams a native, reliable safety net. For production EKS environments, that’s a meaningful shift in operational confidence.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp5f2v9ad64fcfq426wjk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp5f2v9ad64fcfq426wjk.png" alt=" " width="800" height="176"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3sgwfxmyy72zkhprlvk6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3sgwfxmyy72zkhprlvk6.png" alt=" " width="800" height="467"></a></p>

<p><strong>Final Thoughts</strong></p>

<p>Native Amazon EKS support in AWS Backup removes much of the complexity that platform teams have traditionally had to manage themselves. It delivers consistent, policy-driven backups and predictable restores without introducing additional controllers or operational overhead.</p>

<p>That said, there are still important considerations:</p>

<ul>
<li><p>Not all Kubernetes resources are restored exactly as-is, particularly external integrations and cloud-managed services</p></li>
<li><p>Restore time depends heavily on persistent volume size and data footprint</p></li>
<li><p>Standard AWS Backup costs apply for snapshots, storage, and retention</p></li>
<li><p>This does not replace GitOps—it complements it by providing a last-known-good runtime recovery option</p></li>
</ul>

<p>For teams running production EKS on AWS, this feature doesn’t eliminate the need for good engineering practices, but it significantly reduces the risk and stress associated with cluster-level failures. That alone makes it a meaningful addition to the EKS operational toolkit.</p>

<h3>
  
  
  References
</h3>

<ul>
<li><p><a href="https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html" rel="noopener noreferrer">AWS Backup Documentation</a>  </p></li>
<li><p><a href="https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html" rel="noopener noreferrer">Amazon EKS Documentation</a>  </p></li>
<li><p><a href="https://docs.aws.amazon.com/eks/latest/userguide/integration-backup.html" rel="noopener noreferrer">AWS Blog: Native EKS Backup Integration</a>  </p></li>
<li><p><a href="https://aws.amazon.com/blogs/aws/secure-eks-clusters-with-the-new-support-for-amazon-eks-in-aws-backup/" rel="noopener noreferrer">AWS Blog: Secure EKS Clusters with AWS Backup</a>  </p></li>
</ul>

