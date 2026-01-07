---
Title: Craft and Control Resource Pools on MSSQL
Description: 
Author: Amar Abaz
Date: 2026-01-07T21:52:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>Time to take control of your server's resources..<br>
On your MS SQL instance not all workloads should be treated equally.<br>
One heavy query executed from SQL Server Management Studio (SSMS) can easily consume CPU and memory, impacting production workloads.</p>

<p>Resource Governor is an underused but <strong>powerful feature</strong> in SQL Server.<br>
With a simple function and resource pool, you can dramatically improve stability and predictability in multi-user environments.</p>

<p>In this post, I’ll show how to use SQL Server Resource Governor to,</p>

<ul>
<li>Detect connections coming from SSMS,</li>
<li>Assign them to a limited workload group,</li>
<li>Restrict their CPU and memory usage.</li>
</ul>

<p><strong>What is Resource Governor ?</strong><br>
Resource Governor is a SQL Server feature that allows you to control how much CPU and memory different workloads can consume.<br>
This approach is especially useful for production safety, or DBA-controlled access.</p>

<p>After this setup any query executed from Microsoft SQL Server Management Studio will be assigned to a limited resource pool.<br>
All other connections remain in the default pool.</p>

<p><em>This also means you can customize resource pools to suit your specific needs by modifying the function below.</em></p>

<p>You can control CPU, RAM resources for:</p>

<ul>
<li>Specific SQL users,</li>
<li>Reporting or BI tools,</li>
<li>ETL jobs,</li>
<li>Ad-hoc users,</li>
<li>Third-party applications,</li>
<li>Nightly or background workloads.
All you need to do is modify the function and route those sessions into the appropriate workload group.</li>
</ul>

<p>Let's begin with example on limiting CPU &amp; RAM for sessions comming from SSMS Studio,<br>
<strong>1. Enable Governor.</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ALTER RESOURCE GOVERNOR RECONFIGURE;  
GO
</code></pre>

</div>



<p><strong>2. Create your function.</strong><br>
This will decides which workload group a session should use.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE FUNCTION [dbo].[ResourceGroup_Users]
RETURNS SYSNAME
WITH SCHEMABINDING
AS
BEGIN
DECLARE @WorkloadGroup SYSNAME;
DECLARE @ProgramName SYSNAME;

SET @ProgramName = CONVERT(SYSNAME, PROGRAM_NAME());
IF @ProgramName like 'Microsoft SQL Server Management Studio%'
BEGIN
    SET @WorkloadGroup = 'LimitedResourceGroup';
END
ELSE
BEGIN
    SET @WorkloadGroup = 'default';
END

RETURN @WorkloadGroup;

END
GO
</code></pre>

</div>



<p>Also, if you want to restrict specific SQL users, you can easily modify the this function by adding a additional condition like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>IF SUSER_NAME() = 'testuser' THEN ...
</code></pre>

</div>



<p>This allows you to route specific users to a resource pool with customized limits.</p>

<p><strong>3. Create a Resource Pool.</strong><br>
This ensures SSMS queries can never consume more than 30% CPU, 30% memory.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE RESOURCE POOL [LimitedResourcePool]
WITH
(min_cpu_percent=0,
max_cpu_percent=30,
min_memory_percent=0,
max_memory_percent=30
)
</code></pre>

</div>



<p><strong>4. Create a Workload Group.</strong><br>
Now we link a workload group to the limited resource pool.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE WORKLOAD GROUP [LimitedResourceGroup]
USING [LimitedResourcePool];
GO
</code></pre>

</div>



<p><strong>5. Enable the Classifier Function.</strong><br>
Finally, tell Resource Governor to use our function and reconfigure it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ALTER RESOURCE GOVERNOR
WITH (CLASSIFIER_FUNCTION=[dbo].[ResourceGroup_Users]);
GO
ALTER RESOURCE GOVERNOR RECONFIGURE
GO
</code></pre>

</div>



<p>Keep in mind that,</p>

<ul>
<li>These changes apply only to new sessions, not to existing ones. You should restart the SQL service or terminate sessions for changes to affect.</li>
<li>Resource Governor is available in SQL Server Standard Edition starting with SQL Server 2025. For all previous versions, it is an Enterprise-only feature. </li>
</ul>

<p>To monitor which sessions are currently using your resource pool, you can run the following query. This will help you ensure that the sessions are properly classified and using the intended resources.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT
s.session_id,
s.login_name,
rg.name AS resource_pool_name
FROM
sys.dm_exec_sessions AS s
JOIN
sys.dm_resource_governor_workload_groups AS wg
ON s.group_id = wg.group_id
JOIN
sys.dm_resource_governor_resource_pools AS rg
ON wg.pool_id = rg.pool_id
WHERE
s.is_user_process = 1;
</code></pre>

</div>



<p>Now go ahead and put those resource pools to work.<br>
I hope this helps you keep your SQL Server running smoothly! <br>
Don’t miss my other posts for more tips and best practices.</p>

