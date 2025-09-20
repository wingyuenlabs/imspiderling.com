---
Title: What is Active Directory (AD) domain for FSx authentication
Description: 
Author: Wakeup Flower
Date: 2025-09-20T21:52:02.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  <strong>Amazon FSx for Windows File Server</strong>
</h3>

<p>It’s a fully managed <strong>Windows-native file system</strong> on AWS.</p>

<ul>
<li>It supports <strong>SMB protocol</strong> (the same used by Windows file shares).</li>
<li>It integrates with <strong>Active Directory (AD)</strong> for <strong>user authentication and access control</strong>.</li>
</ul>




<h3>
  
  
  <strong>“Set the Active Directory domain for authentication” — what it means</strong>
</h3>

<p>When you create an FSx for Windows File Server, you must tell AWS how it should handle <strong>user authentication</strong> and <strong>permissions</strong>.</p>

<p>FSx doesn’t manage users by itself — instead, it <strong>joins an Active Directory domain</strong>.<br>
That way, your existing users and groups (from AD) can:</p>

<ul>
<li>Authenticate (log in) to the file share.</li>
<li>Have permissions (read/write/deny) applied using standard <strong>NTFS and SMB ACLs</strong>.</li>
</ul>




<h3>
  
  
  ✅ Two main options:
</h3>

<ol>
<li><strong>AWS Managed Microsoft AD</strong></li>
</ol>

<ul>
<li>You let AWS manage an AD domain.</li>
<li>FSx joins this domain.</li>
<li>Users in this AD can access the file system.</li>
</ul>

<ol>
<li><strong>Self-Managed AD (on-premises or in EC2)</strong></li>
</ol>

<ul>
<li>If you already have an AD (on-prem or in the cloud), FSx can be joined to that domain using <strong>AWS Directory Service AD Connector</strong>.</li>
<li>This allows your existing corporate AD users to authenticate.</li>
</ul>




<h3>
  
  
  🔎 In practice:
</h3>

<p>When setting up FSx, you’ll be asked for AD details such as:</p>

<ul>
<li>Domain name (e.g., <code>corp.example.com</code>)</li>
<li>DNS IPs of your domain controllers</li>
<li>A service account (with permissions to join machines to the domain)</li>
</ul>

<p>After FSx joins the domain:</p>

<ul>
<li>Users connect like they would to a normal Windows file share (<code>\\fsxshare\folder</code>).</li>
<li>Authentication and access are handled by AD.</li>
</ul>




<p>👉 So the phrase means:<br>
When you <strong>create an FSx for Windows file system</strong>, you must tell AWS <strong>which Active Directory domain</strong> it should use for <strong>user login and access control</strong>.</p>

<p>Here’s a clear <strong>comparison table</strong> for <strong>Amazon FSx for Windows File Server vs Amazon EFS</strong> in the context of your SharePoint scenario:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature / Requirement</th>
<th>Amazon FSx for Windows File Server</th>
<th>Amazon EFS (Elastic File System)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Supported Protocol</strong></td>
<td>SMB (Windows file sharing)</td>
<td>NFS (Linux/Unix file sharing)</td>
</tr>
<tr>
<td><strong>Windows AD Integration</strong></td>
<td>✅ Fully integrates with Active Directory</td>
<td>❌ No native AD integration</td>
</tr>
<tr>
<td><strong>File Semantics</strong></td>
<td>✅ Windows-native (NTFS, ACLs, locks)</td>
<td>❌ POSIX permissions only</td>
</tr>
<tr>
<td><strong>High Availability</strong></td>
<td>✅ Multi-AZ option available</td>
<td>✅ Multi-AZ option available</td>
</tr>
<tr>
<td><strong>Typical Workload</strong></td>
<td>Windows applications, SharePoint, SQL Server</td>
<td>Linux applications, web servers, containers</td>
</tr>
<tr>
<td><strong>Access from Windows Clients</strong></td>
<td>✅ Seamless SMB access</td>
<td>❌ Limited (requires NFS client for Windows)</td>
</tr>
<tr>
<td><strong>Use Case Fit for SharePoint</strong></td>
<td>✅ Ideal</td>
<td>❌ Not suitable</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  🔑 Takeaways:
</h3>

<ul>
<li>FSx for Windows File Server is <strong>Windows-native</strong> and supports <strong>AD authentication</strong>, making it the correct choice for SharePoint.</li>
<li>EFS is <strong>Linux/NFS-only</strong> and <strong>cannot natively handle Windows ACLs or AD users</strong>, so it <strong>doesn’t meet the requirements</strong>.</li>
</ul>

