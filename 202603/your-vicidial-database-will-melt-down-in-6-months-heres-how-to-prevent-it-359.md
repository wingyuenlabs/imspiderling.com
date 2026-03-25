---
Title: Your VICIdial Database Will Melt Down in 6 Months. Here's How to Prevent It.
Description: 
Author: Jason Shouldice
Date: 2026-03-25T22:01:12.000Z
Robots: noindex,nofollow
Template: index
---
<p>It starts the same way every time. Your VICIdial runs fine for six months. Then one Monday morning, agents can't log in. Real-time reports take 30 seconds to load. The hopper drains faster than it fills. CPU is pegged -- not by Asterisk, but by MySQL grinding through a query on a <code>vicidial_log</code> table with 47 million rows because nobody set up archival.</p>

<p>The database is VICIdial's single most critical component. Every agent action, call event, disposition, real-time report, hopper query, and dial-level calculation flows through one MySQL instance. In a VICIdial cluster, you can add telephony servers for call capacity and web servers for agent connections. You cannot cluster the database. When it slows down, everything slows down.</p>

<h2>
  
  
  Why VICIdial's Database Is Different
</h2>

<p>VICIdial's access patterns are unusual and demanding compared to most web applications:</p>

<p><strong>High-frequency writes.</strong> Multiple Perl daemons update agent states, call statuses, and dial metrics across dozens of tables every second. The <code>AST_update.pl</code> process alone performs hundreds of UPDATEs per second in a busy 50-agent campaign.</p>

<p><strong>Constant polling.</strong> Real-time reports don't use WebSockets or push notifications. They poll the database every second. Ten managers with reports open = 10 SELECTs per second against <code>vicidial_live_agents</code> and <code>vicidial_auto_calls</code>, on top of everything else.</p>

<p><strong>Mixed workload.</strong> Short, fast transactional queries (agent state updates) run simultaneously with long analytical queries (campaign reports spanning millions of records). Transactional queries need low latency. Analytical queries need throughput. MySQL serves both from the same instance.</p>

<p><strong>MyISAM table-level locking.</strong> VICIdial uses MyISAM exclusively. A single slow SELECT on <code>vicidial_log</code> blocks every write on that table until it completes. This is the fundamental scaling constraint.</p>

<h2>
  
  
  The Table That Eats Your Server
</h2>

<p><code>vicidial_manager</code> is the fastest-growing table in most deployments and the one most overlooked. It stores every AMI command VICIdial sends to Asterisk -- Originate, Hangup, Monitor. At 50 agents with a 5:1 dial ratio, that's 250+ Originate commands per cycle, plus hangups, plus monitors. This table hits 100 million rows in under a year if not archived.</p>

<p>Growth rates for a 50-agent outbound operation running 8 hours daily:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Table</th>
<th>Growth/Month</th>
<th>6-Month Size</th>
</tr>
</thead>
<tbody>
<tr>
<td>vicidial_log</td>
<td>~1M rows</td>
<td>6M rows, ~2 GB</td>
</tr>
<tr>
<td>vicidial_log_extended</td>
<td>~1M rows</td>
<td>6M rows, ~3 GB</td>
</tr>
<tr>
<td>call_log</td>
<td>~1M rows</td>
<td>6M rows, ~2 GB</td>
</tr>
<tr>
<td>vicidial_carrier_log</td>
<td>~2-3M rows</td>
<td>15M rows, ~4 GB</td>
</tr>
<tr>
<td>vicidial_manager</td>
<td>~5-10M rows</td>
<td>40M+ rows, ~8 GB</td>
</tr>
<tr>
<td>recording_log</td>
<td>~500K rows</td>
<td>3M rows, ~1 GB</td>
</tr>
<tr>
<td>server_performance</td>
<td>~15K/day/server</td>
<td>2.7M rows/server</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  MEMORY Tables: VICIdial's Real-Time Engine
</h2>

<p>Several critical tables run in the MEMORY storage engine -- entirely in RAM, no disk I/O:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Table</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td>vicidial_live_agents</td>
<td>State of every logged-in agent</td>
</tr>
<tr>
<td>vicidial_live_inbound_agents</td>
<td>Inbound-eligible agents</td>
</tr>
<tr>
<td>vicidial_auto_calls</td>
<td>Every active auto-dial call</td>
</tr>
<tr>
<td>vicidial_hopper</td>
<td>Leads queued for dialing</td>
</tr>
<tr>
<td>vicidial_live_sip_channels</td>
<td>Active SIP channels</td>
</tr>
</tbody>
</table></div>

<p>MEMORY tables lose all data on MySQL restart. This is by design. Agents log back in, hopper refills automatically. Brief disruption during production hours if MySQL restarts unexpectedly.</p>

<p>The MEMORY table ceiling is <code>max_heap_table_size</code> in my.cnf. If a MEMORY table exceeds this, INSERTs fail -- agents can't log in or calls don't get placed. Set to 64 MB minimum. 128-256 MB for 500+ agents.</p>

<h2>
  
  
  my.cnf Tuning: What Actually Matters
</h2>

<h3>
  
  
  Universal Settings (Every Deployment)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[mysqld]</span>
<span class="c"># THE most important single setting
</span><span class="err">skip-name-resolve</span>

<span class="c"># VICIdial's connection-heavy architecture
</span><span class="py">max_connections</span> <span class="p">=</span> <span class="s">2000</span>

<span class="c"># Disable query cache -- more harm than good with VICIdial's write patterns
</span><span class="py">query_cache_size</span> <span class="p">=</span> <span class="s">0</span>
<span class="py">query_cache_type</span> <span class="p">=</span> <span class="s">0</span>

<span class="c"># Allow concurrent inserts during SELECT on MyISAM
</span><span class="py">concurrent_insert</span> <span class="p">=</span> <span class="s">2</span>

<span class="c"># VICIdial opens hundreds of tables
</span><span class="py">table_open_cache</span> <span class="p">=</span> <span class="s">4096</span>

<span class="c"># Temp table limits
</span><span class="py">tmp_table_size</span> <span class="p">=</span> <span class="s">64M</span>
<span class="py">max_heap_table_size</span> <span class="p">=</span> <span class="s">64M</span>

<span class="c"># Thread cache
</span><span class="py">thread_cache_size</span> <span class="p">=</span> <span class="s">128</span>

<span class="c"># Slow query logging -- enable always
</span><span class="py">slow_query_log</span> <span class="p">=</span> <span class="s">1</span>
<span class="py">slow_query_log_file</span> <span class="p">=</span> <span class="s">/var/log/mysql/slow-query.log</span>
<span class="py">long_query_time</span> <span class="p">=</span> <span class="s">2</span>
</code></pre>

</div>



<p><strong>The skip-name-resolve story:</strong> Without it, MySQL performs a reverse DNS lookup on every incoming connection. In a VICIdial cluster with telephony servers, web servers, and dozens of cron-spawned Perl scripts maintaining constant connections, those DNS lookups pile up. You get "Too many connections" errors even with max_connections set high enough -- because connections are stuck waiting for DNS. This single line has saved more VICIdial installations than any other my.cnf change.</p>

<p>When you enable skip-name-resolve, use IP addresses (not hostnames) in GRANT statements.</p>

<h3>
  
  
  Sizing by Deployment
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Agents</th>
<th>key_buffer_size</th>
<th>RAM Budget</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>50</td>
<td>512M</td>
<td>16 GB server, ~2 GB MySQL buffers</td>
<td>Leaves 14 GB for OS cache</td>
</tr>
<tr>
<td>100</td>
<td>1024M</td>
<td>32 GB server, ~4 GB MySQL buffers</td>
<td>Leaves 28 GB for OS cache</td>
</tr>
<tr>
<td>500</td>
<td>4096M</td>
<td>64 GB server, ~10 GB MySQL buffers</td>
<td>Leaves 54 GB for OS cache</td>
</tr>
</tbody>
</table></div>

<p><strong>Why not allocate more to MySQL?</strong> MyISAM stores data in .MYD files read through the OS filesystem cache. <code>key_buffer_size</code> only caches index blocks (.MYI files). The real performance for data reads comes from the OS cache -- which uses whatever RAM MySQL isn't consuming. Rule of thumb: MySQL buffers should use no more than 40-50% of total RAM.</p>

<p><strong>Why set innodb_buffer_pool_size at all?</strong> VICIdial uses MyISAM, but MySQL's internal system tables use InnoDB. A small InnoDB buffer pool (256M-2G) prevents internal operations from thrashing disk.</p>

<h2>
  
  
  Slow Query Identification
</h2>

<p>Enable the slow query log always. Overhead is negligible compared to debugging time saved.</p>

<p>Install Percona Toolkit and analyze:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>yum <span class="nb">install </span>percona-toolkit
pt-query-digest /var/log/mysql/slow-query.log <span class="o">&gt;</span> /tmp/slow-query-report.txt
</code></pre>

</div>



<p>The usual suspects in VICIdial deployments:</p>

<ol>
<li>
<strong>Report queries on vicidial_log</strong> -- SELECTs spanning millions of rows, triggered by managers running "last 90 days" reports</li>
<li>
<strong>Hopper fills on vicidial_list</strong> -- Scanning for callable leads on a 10M+ row table with inadequate indexes</li>
<li>
<strong>Recording search on recording_log</strong> -- Millions of rows, date range searches</li>
<li>
<strong>vicidial_manager cleanup</strong> -- Periodic DELETEs on a 50M+ row table that lock it for seconds</li>
</ol>

<p>When MySQL is actively struggling, check what's running:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SHOW</span> <span class="k">FULL</span> <span class="n">PROCESSLIST</span><span class="p">;</span>
</code></pre>

</div>



<p>Look for "Locked" state (MyISAM contention), "Copying to tmp table" (complex queries hitting disk), "Sorting result" (ORDER BY without indexes), and multiple identical queries (report polling storms).</p>

<h2>
  
  
  Essential Missing Indexes
</h2>

<p>VICIdial's install scripts create most indexes, but these are commonly missing and make a measurable difference:</p>

<h3>
  
  
  vicidial_log
</h3>

<p>The default indexes cover lead_id and uniqueid, but report queries filter by call_date:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_log</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_call_date</span> <span class="p">(</span><span class="n">call_date</span><span class="p">);</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_log</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_campaign_call_date</span> <span class="p">(</span><span class="n">campaign_id</span><span class="p">,</span> <span class="n">call_date</span><span class="p">);</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_log</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_user_call_date</span> <span class="p">(</span><span class="k">user</span><span class="p">,</span> <span class="n">call_date</span><span class="p">);</span>
</code></pre>

</div>



<p>The composite (campaign_id, call_date) index makes campaign-specific reports dramatically faster -- the most common analytical query pattern.</p>

<h3>
  
  
  vicidial_list
</h3>

<p>The hopper fill query scans for callable leads. Default indexes are insufficient for large lists:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_list</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_list_status</span> <span class="p">(</span><span class="n">list_id</span><span class="p">,</span> <span class="n">status</span><span class="p">);</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_list</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_status_modify</span> <span class="p">(</span><span class="n">status</span><span class="p">,</span> <span class="n">modify_date</span><span class="p">);</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_list</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_phone_number</span> <span class="p">(</span><span class="n">phone_number</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  recording_log
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">recording_log</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_lead_date</span> <span class="p">(</span><span class="n">lead_id</span><span class="p">,</span> <span class="n">start_time</span><span class="p">);</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">recording_log</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_start_time</span> <span class="p">(</span><span class="n">start_time</span><span class="p">);</span>
</code></pre>

</div>



<p>Before adding: check with <code>SHOW INDEX FROM tablename</code> to verify the index doesn't already exist. Adding an index to a 10M+ row MyISAM table locks it for the duration. Schedule during off-hours. Periodically run <code>OPTIMIZE TABLE</code> on heavily-written tables to rebuild indexes and reclaim space from deleted rows -- but expect 5-30 minutes of table lock on large tables.</p>

<h2>
  
  
  Table Archival: The Maintenance That Saves Everything
</h2>

<p>If your deployment has run 90+ days without archival, your database carries dead weight that slows every query, every lock, every OPTIMIZE operation.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Table</th>
<th>Archive After</th>
<th>Reason</th>
</tr>
</thead>
<tbody>
<tr>
<td>vicidial_manager</td>
<td>7 days</td>
<td>Fastest growing, almost never queried historically</td>
</tr>
<tr>
<td>vicidial_carrier_log</td>
<td>30 days</td>
<td>High volume, low long-term value</td>
</tr>
<tr>
<td>call_log</td>
<td>60 days</td>
<td>Raw CDR, rarely accessed after initial reporting</td>
</tr>
<tr>
<td>vicidial_log</td>
<td>90 days</td>
<td>Reports rarely go back further</td>
</tr>
<tr>
<td>vicidial_agent_log</td>
<td>90 days</td>
<td>Agent session data</td>
</tr>
<tr>
<td>recording_log</td>
<td>180 days</td>
<td>Keep metadata longer for compliance</td>
</tr>
</tbody>
</table></div>

<p>VICIdial includes <code>ADMIN_archive_log_tables.pl</code>. Set up as a daily cron:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>0 1 <span class="k">*</span> <span class="k">*</span> <span class="k">*</span> /usr/share/astguiclient/ADMIN_archive_log_tables.pl <span class="nt">--daily</span> 2&gt;&amp;1 <span class="o">&gt;&gt;</span> /var/log/astguiclient/archive.log
</code></pre>

</div>



<p>When deleting from large MyISAM tables, always use LIMIT to avoid locking the table for minutes. A single DELETE of 50 million rows locks the table for the duration. Break it into chunks of 100,000-500,000.</p>

<h3>
  
  
  Creating Archive Tables
</h3>

<p>Before archiving, verify <code>_archive</code> tables exist. If not:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SHOW</span> <span class="n">TABLES</span> <span class="k">LIKE</span> <span class="s1">'vicidial_log_archive'</span><span class="p">;</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">vicidial_log_archive</span> <span class="k">LIKE</span> <span class="n">vicidial_log</span><span class="p">;</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">vicidial_log_archive</span> <span class="k">ADD</span> <span class="k">INDEX</span> <span class="n">idx_call_date</span> <span class="p">(</span><span class="n">call_date</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  The Nuclear Option
</h3>

<p>If tables have grown to hundreds of millions of rows and archival takes too long:</p>

<ol>
<li>Rename the bloated table</li>
<li>Create a fresh table with the same structure</li>
<li>Copy recent data (last 90 days) into the fresh table</li>
<li>Verify data integrity</li>
<li>Drop or rename the old table to archive</li>
</ol>

<p>This is faster than DELETE because it builds sequentially rather than performing random deletes. Requires enough disk for both tables temporarily, and VICIdial must be stopped during the swap.</p>

<h2>
  
  
  MEMORY Table Conversion for High-Scale
</h2>

<p>Above 150-200 agents, converting high-traffic tables from MyISAM to MEMORY is one of the most impactful single optimizations.</p>

<p><code>vicidial_live_agents</code> tracks every agent's real-time state and gets read by every report, every dialer process, every AGI script. On MyISAM, table-level locks create contention: a report SELECT blocks the dialer's UPDATE marking an agent available, which blocks an AGI routing a call. MEMORY eliminates disk I/O and drops lock duration from milliseconds to microseconds.</p>

<p>Convert: vicidial_live_agents, vicidial_live_inbound_agents, vicidial_auto_calls, vicidial_hopper, vicidial_live_sip_channels. Never convert vicidial_log or vicidial_list (too large, persistent data).</p>

<p>MEMORY requires fixed-width CHAR fields (no VARCHAR). Recent VICIdial SVN revisions (3800+) include built-in MEMORY table support with automatic conversion on startup. Check your version before doing manual conversions.</p>

<h2>
  
  
  The TIMESTAMP Migration
</h2>

<p>Recent VICIdial SVN updates convert DATETIME columns to TIMESTAMP across 40+ tables. TIMESTAMP uses 4 bytes vs DATETIME's 8 bytes -- half the storage, faster comparisons. But the ALTER TABLE on a 50M-row MyISAM table locks it for 30+ minutes. Plan this during a maintenance window with a full backup first.</p>

<h2>
  
  
  Corruption Recovery
</h2>

<p>MyISAM tables corrupt during power failures, unexpected shutdowns, and full disks. Symptoms: "Table marked as crashed" errors, unexpected query results, MySQL refusing to start.</p>

<p>First check: <code>CHECK TABLE tablename</code>. If issues found: <code>REPAIR TABLE tablename</code>. For severe cases, use myisamchk directly on the .MYI and .MYD files with MySQL stopped.</p>

<h2>
  
  
  Replication for Reporting
</h2>

<p>The single most effective way to eliminate report queries from impacting production is MySQL master-slave replication. Run reports against the replica. Production call operations hit the master only.</p>

<p>Configure binary logging on the master:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="nn">[mysqld]</span>
<span class="py">log-bin</span> <span class="p">=</span> <span class="s">mysql-bin</span>
<span class="py">binlog_format</span> <span class="p">=</span> <span class="s">MIXED</span>
<span class="py">expire_logs_days</span> <span class="p">=</span> <span class="s">7</span>
<span class="py">sync_binlog</span> <span class="p">=</span> <span class="s">0    # Async for performance</span>
</code></pre>

</div>



<p>The replica receives all writes via the binary log and stays in sync. Report queries -- the big SELECTs that scan millions of rows -- hit the replica instead of locking tables on the master. For 100+ agent deployments, this is often the single biggest performance improvement after basic my.cnf tuning.</p>

<h2>
  
  
  Per-Thread Buffer Warning
</h2>

<p>Settings like <code>sort_buffer_size</code>, <code>read_buffer_size</code>, and <code>join_buffer_size</code> are allocated per-thread when needed. With <code>max_connections = 6000</code>, the theoretical maximum for sort buffers alone: 6000 x 16 MB = 96 GB. MySQL allocates on-demand, not all at once, but if you see unexpected memory pressure, per-thread buffers are the first place to look.</p>

<h2>
  
  
  The PROCESSLIST Emergency Toolkit
</h2>

<p>When things are actively on fire:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">information_schema</span><span class="p">.</span><span class="n">PROCESSLIST</span>
<span class="k">WHERE</span> <span class="n">COMMAND</span> <span class="o">!=</span> <span class="s1">'Sleep'</span> <span class="k">AND</span> <span class="nb">TIME</span> <span class="o">&gt;</span> <span class="mi">2</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="nb">TIME</span> <span class="k">DESC</span><span class="p">;</span>
</code></pre>

</div>



<p>What the states mean:</p>

<ul>
<li>
<strong>Locked</strong> -- MyISAM table-level lock contention. Another query is holding the table lock.</li>
<li>
<strong>Copying to tmp table</strong> -- Complex query creating a temp table on disk. Usually means a missing index or a report query scanning too many rows.</li>
<li>
<strong>Sorting result</strong> -- ORDER BY on a large result set without an appropriate index.</li>
<li>
<strong>Multiple identical queries</strong> -- Real-time report polling storm. Ten managers, one query each per second. Reduce the refresh interval or set up a caching layer.</li>
</ul>

<p>Kill a runaway query if needed: <code>KILL process_id;</code> -- but understand that killing a MyISAM write can leave the table in a state requiring REPAIR TABLE.</p>

<h2>
  
  
  MyISAM vs InnoDB: The VICIdial Position
</h2>

<p>You might wonder why VICIdial uses MyISAM when InnoDB has row-level locking that would solve the contention problem. Matt Florell has been explicit: "We do not recommend using InnoDB under any circumstances." The codebase is designed and tested against MyISAM behavior. The Perl daemons, keepalive processes, and hopper logic all assume MyISAM's specific behaviors around table-level locking, concurrent inserts, and fulltext indexing.</p>

<p>Converting VICIdial tables to InnoDB would introduce subtle bugs that only manifest under production load -- exactly the worst time to discover them. Stick with MyISAM. Optimize it aggressively. Use MEMORY tables for the hot real-time tables. Set up replication for reporting. And archive religiously.</p>

<h2>
  
  
  Connection Management
</h2>

<p>VICIdial's architecture is connection-heavy. Perl daemons, web servers, cron scripts, agent sessions -- they all maintain database connections. Without proper configuration, you'll see "Too many connections" errors during peak hours.</p>

<p>Set <code>max_connections</code> based on your deployment size. 2000 for 50 agents, 3000 for 100 agents, 6000 for 500 agents. These seem high, but VICIdial's keepalive and update processes create far more connections than the agent count suggests.</p>

<p><code>wait_timeout</code> and <code>interactive_timeout</code> control how long idle connections persist. Default MySQL values (28800 seconds / 8 hours) are too high for VICIdial. Set both to 600 seconds (10 minutes). This cleans up stale connections from crashed Perl processes that didn't disconnect properly.</p>

<p><code>thread_cache_size</code> avoids the overhead of creating new threads for frequent connections. Set to 128 for small deployments, 256-512 for large ones. MySQL reuses cached threads instead of spawning new ones, which matters when connection turnover is high.</p>

<h2>
  
  
  Disk I/O: The Hidden Bottleneck
</h2>

<p>On busy VICIdial deployments, disk I/O often matters more than CPU. MyISAM writes data synchronously to .MYD files. Recording 200 concurrent calls writes ~200 MB/minute. When the database and recordings compete for the same disk, everything slows.</p>

<p>Solution: separate disks or partitions. Database files on SSD (preferably NVMe). Recordings on a separate volume. If you only have one disk, at minimum put the MySQL data directory and the recording spool on separate partitions so I/O operations don't interleave.</p>

<p>Monitor with <code>iostat -x 5</code> during peak hours. If <code>%util</code> is above 80% on your database disk, you need either faster storage or to split the workload.</p>

<h2>
  
  
  Maintenance Schedule
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Task</th>
<th>Frequency</th>
<th>Impact</th>
</tr>
</thead>
<tbody>
<tr>
<td>Archival (ADMIN_archive_log_tables.pl)</td>
<td>Daily, 1 AM</td>
<td>Low -- runs in background</td>
</tr>
<tr>
<td>vicidial_manager cleanup</td>
<td>Daily</td>
<td>Medium -- LIMIT deletes to avoid lock</td>
</tr>
<tr>
<td>OPTIMIZE TABLE on vicidial_log</td>
<td>Monthly, maintenance window</td>
<td>High -- locks table 5-30 min</td>
</tr>
<tr>
<td>OPTIMIZE TABLE on vicidial_list</td>
<td>Monthly, maintenance window</td>
<td>High -- locks table</td>
</tr>
<tr>
<td>pt-query-digest analysis</td>
<td>Weekly</td>
<td>None -- reads log only</td>
</tr>
<tr>
<td>Index review</td>
<td>Quarterly</td>
<td>Varies</td>
</tr>
<tr>
<td>Full database backup</td>
<td>Daily</td>
<td>Low if using mysqldump during quiet hours</td>
</tr>
</tbody>
</table></div>

<p>The database is the one part of VICIdial you can't horizontally scale. Treat it accordingly. <a href="https://vicistack.com/proof/" rel="noopener noreferrer">ViciStack</a> ships deployments with MySQL pre-tuned, MEMORY tables optimized, and archival jobs running from day one.</p>

<p>Originally published at <a href="https://vicistack.com/blog/vicidial-mysql-optimization/" rel="noopener noreferrer">https://vicistack.com/blog/vicidial-mysql-optimization/</a></p>

