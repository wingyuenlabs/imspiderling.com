---
Title: Four Disaster Recovery (DR) strategies in AWS explained
Description: 
Author: Wakeup Flower
Date: 2025-09-21T21:37:59.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  <strong>1. Multi-Site (Active-Active)</strong>
</h3>

<ul>
<li>
<strong>Description:</strong> Full production runs simultaneously in multiple regions.</li>
<li>
<p><strong>Real-Life Example:</strong></p>

<ul>
<li>
<strong>Global banking system</strong>: Customers in New York, London, and Tokyo need access to their accounts at <strong>all times</strong>.</li>
<li>If the New York data center fails, London or Tokyo handles transactions <strong>without downtime</strong>.</li>
</ul>


</li>

<li><p><strong>RTO/RPO:</strong> Near zero (instant failover)</p></li>

<li><p><strong>Cost:</strong> Very high (you pay for multiple full environments)</p></li>

<li><p><strong>Use Case:</strong> Mission-critical apps with <strong>zero tolerance for downtime</strong>, e.g., stock trading platforms, airline reservation systems.</p></li>

</ul>




<h3>
  
  
  <strong>2. Warm Standby</strong>
</h3>

<ul>
<li>
<strong>Description:</strong> A smaller-scale version of your environment is running in another region. It can scale up when needed.</li>
<li>
<p><strong>Real-Life Example:</strong></p>

<ul>
<li>
<strong>E-commerce website</strong>: Main site in <strong>US-East</strong>, warm standby in <strong>US-West</strong> with <strong>minimum servers and database replicas</strong>.</li>
<li>During a disaster in US-East, US-West scales up to handle full traffic.</li>
</ul>


</li>

<li><p><strong>RTO/RPO:</strong> Medium — typically minutes to a few hours</p></li>

<li><p><strong>Cost:</strong> Medium — you pay for standby resources, not full production</p></li>

<li><p><strong>Use Case:</strong> Apps that are <strong>critical but can tolerate brief downtime</strong>, e.g., online stores, internal enterprise applications.</p></li>

</ul>




<h3>
  
  
  <strong>3. Pilot Light</strong>
</h3>

<ul>
<li>
<strong>Description:</strong> Minimal critical resources are running; rest of the environment is off but can be launched on-demand.</li>
<li>
<p><strong>Real-Life Example:</strong></p>

<ul>
<li>
<strong>SaaS analytics platform</strong>: Only the <strong>database</strong> is running in a secondary region.</li>
<li>During a disaster, application servers, load balancers, and other services are launched quickly.</li>
</ul>


</li>

<li><p><strong>RTO/RPO:</strong> Medium-High — some time required to bring services online</p></li>

<li><p><strong>Cost:</strong> Low-Medium — only the critical part runs continuously</p></li>

<li><p><strong>Use Case:</strong> Apps where <strong>cost savings are important</strong> but <strong>faster recovery than backup/restore</strong> is needed, e.g., SaaS reporting tools, business intelligence dashboards.</p></li>

</ul>




<h3>
  
  
  <strong>4. Backup &amp; Restore</strong>
</h3>

<ul>
<li>
<strong>Description:</strong> Data is backed up; environment is built only when needed.</li>
<li>
<p><strong>Real-Life Example:</strong></p>

<ul>
<li>
<strong>Archival video content</strong>: Stored in S3 with snapshots.</li>
<li>If the primary site is lost, you <strong>restore the content to a new environment</strong>, which may take hours or days.</li>
</ul>


</li>

<li><p><strong>RTO/RPO:</strong> High — hours to days; data loss depends on backup frequency</p></li>

<li><p><strong>Cost:</strong> Low — you only pay for storage, not running instances</p></li>

<li><p><strong>Use Case:</strong> Non-critical workloads or infrequent access content, e.g., backups, dev/test environments, archival systems.</p></li>

</ul>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>DR Strategy</th>
<th>RTO</th>
<th>RPO</th>
<th>Cost</th>
<th>Complexity</th>
<th>Best For</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Multi-Site</strong></td>
<td>Very low</td>
<td>Near zero</td>
<td>High</td>
<td>High</td>
<td>Mission-critical apps, zero downtime</td>
</tr>
<tr>
<td><strong>Backup &amp; Restore</strong></td>
<td>High</td>
<td>Depends on backup</td>
<td>Low</td>
<td>Low</td>
<td>Non-critical workloads, archival data</td>
</tr>
<tr>
<td><strong>Warm Standby</strong></td>
<td>Medium</td>
<td>Low-Medium</td>
<td>Medium</td>
<td>Medium</td>
<td>Critical apps with moderate downtime tolerance</td>
</tr>
<tr>
<td><strong>Pilot Light</strong></td>
<td>Medium-High</td>
<td>Low-Medium</td>
<td>Low-Medium</td>
<td>Medium</td>
<td>Cost-conscious apps needing faster recovery than backup</td>
</tr>
</tbody>
</table></div>

