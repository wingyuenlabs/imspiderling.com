---
Title: AWS App Mesh Deprecated: Migration Guide Before September 2026 Shutdown
Description: 
Author: Kseniya Seliverstava
Date: 2026-02-15T21:25:26.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>AWS App Mesh End of Life: September 30, 2026.</strong>[¹] New customers locked out since September 24, 2024. Existing customers have 8 months to migrate. Only critical security updates until shutdown.</p>

<h2>
  
  
  AWS Official Replacements for App Mesh
</h2>

<ol>
<li>
<strong>ECS Service Connect</strong> (for ECS) - AWS-managed mesh without sidecars</li>
<li>
<strong>VPC Lattice</strong> (for EKS) - Application networking across VPCs and accounts</li>
<li>
<strong>Direct ALB routing</strong> - Simplest option for basic use cases</li>
</ol>

<h2>
  
  
  Real-World Migration Case Study
</h2>

<p><strong>Migrated one production service: App Mesh → Direct ALB</strong></p>

<p><strong>Timeline: ~6 hours total</strong></p>

<ul>
<li>Active migration: 2 hours</li>
<li>Troubleshooting: 4 hours</li>
</ul>

<p><strong>Breakdown:</strong></p>

<ul>
<li>Planning &amp; setup: 30 min</li>
<li>Task definition changes: 15 min</li>
<li>Service deployment: 20 min</li>
<li>Security group fixes: 1 hour</li>
<li>Health check troubleshooting: 2 hours</li>
<li>ALB configuration: 15 min</li>
<li>Testing: 1 hour</li>
</ul>

<p><strong>Results:</strong> Zero downtime. Full App Mesh removal. Instant rollback capability maintained.</p>

<p><strong>Key insight:</strong> First service takes 4-6 hours (learning curve). Subsequent services: 1-2 hours each.</p>

<h2>
  
  
  Critical Pre-Migration Requirements
</h2>

<h3>
  
  
  Rollback Strategy is Non-Negotiable
</h3>

<ul>
<li>Ensure all changes are reversible</li>
<li>Don't remove App Mesh infrastructure until new setup is validated in production</li>
<li>Avoid destructive configuration changes</li>
<li>Maintain instant revert capability</li>
</ul>

<h3>
  
  
  Phased Migration Approach
</h3>

<ol>
<li>Deploy new configuration alongside existing App Mesh</li>
<li>Test thoroughly without affecting production traffic</li>
<li>Gradually shift traffic to new infrastructure</li>
<li>Validate stability and performance</li>
<li>Remove App Mesh components only after proven success</li>
</ol>

<h2>
  
  
  Common Migration Challenges
</h2>

<p><strong>Security Groups:</strong> Update rules for direct service-to-service communication (no more Envoy proxies)</p>

<p><strong>Health Checks:</strong> Reconfigure ALB health checks for direct container access</p>

<p><strong>Service Discovery:</strong> Implement AWS Cloud Map or ALB target groups</p>

<h2>
  
  
  Migration Resources
</h2>

<ul>
<li>
<a href="https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html" rel="noopener noreferrer">AWS App Mesh End of Life</a>[¹]</li>
<li><a href="https://aws.amazon.com/blogs/containers/migrating-from-aws-app-mesh-to-amazon-ecs-service-connect/" rel="noopener noreferrer">Migration to ECS Service Connect</a></li>
<li><a href="https://docs.aws.amazon.com/vpc-lattice/" rel="noopener noreferrer">VPC Lattice Documentation</a></li>
</ul>




<p><strong>Action Required:</strong> 8 months until shutdown. Start migration planning now.</p>

