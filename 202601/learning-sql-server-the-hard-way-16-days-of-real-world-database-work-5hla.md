---
Title: Learning SQL Server the Hard Way: 16 Days of Real-World Database Work
Description: 
Author: Luis Faria
Date: 2026-01-26T21:34:03.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>From "I've never used SQL Server" to "Here's my 1,000-line operational runbook": How I turned a job opportunity into a portfolio-building sprint.</strong></p>

<blockquote>
<p><em>Hard work is my preferred language and I try to speak it fluently.</em></p>
</blockquote>




<h2>
  
  
  🎯 The Opportunity
</h2>

<h3>
  
  
  When "I Don't Know SQL Server" Becomes a Challenge
</h3>

<p>A friend reached out with an intriguing proposition: <em>"Do you work with Microsoft SQL Server? We're desperate to fill a school data administrator role."</em></p>

<p>My honest answer? <strong>No—but I know databases.</strong></p>

<p>My background spans PostgreSQL, MySQL, MongoDB, and GraphQL. I've built ETL pipelines, optimized queries, designed schemas, and managed production data systems. The fundamentals are universal: normalization, indexing, backup strategies, referential integrity, stored procedures.</p>

<p>SQL Server syntax? Just a dialect I hadn't learned yet.</p>

<p>But here's the thing about job opportunities in unfamiliar territory: <strong>saying "I can learn it" isn't enough.</strong> Hiring managers hear that every day. What they want is proof.</p>

<h3>
  
  
  The Real Challenge
</h3>

<blockquote>
<p><strong>Can I go from zero SQL Server experience to interview-ready in two weeks, with portfolio-quality deliverables to prove it?</strong></p>
</blockquote>

<p>This wasn't just about learning T-SQL syntax. The role required:</p>

<ul>
<li>Managing school data systems (student records, attendance, class scheduling)</li>
<li>Running reports for leadership and teaching staff</li>
<li>Integrating data from legacy systems like SEQTA and Synergetic</li>
<li>Maintaining backup/recovery procedures</li>
<li>Documenting operations for non-technical staff</li>
<li>Operating responsibly with child-safety-sensitive data</li>
</ul>

<p><strong>The approach:</strong> Treat it like a master's assignment. I've spent months tackling academic projects with a disciplined workflow: <em>Receive Brief → Research → Design → Build → Document → Present → NEXT.</em> Why not leverage that momentum?</p>

<p>This is where strategic use of LLMs came into play. Instead of aimlessly "learning SQL Server," I needed a structured challenge that would simulate real-world job responsibilities.</p>

<h3>
  
  
  The Prompt That Launched the Project
</h3>

<p>The prompt:</p>

<blockquote>
<p><em>"I need to demonstrate enterprise-level SQL Server skills for a school data administrator role. Create a comprehensive 3-level assessment covering: (1) database fundamentals and backup/restore, (2) reporting and data integration, (3) operational documentation and training. Structure it like an internal deliverable with real-world scenarios matching school systems like SEQTA and Synergetic."</em></p>
</blockquote>




<h2>
  
  
  The Problem
</h2>

<p>Build a "School Data Platform" on SQL Server, documented like an internal deliverable. <em>Do the deed and show the proof</em>.</p>

<p>The structure emerged as a 3-level assessment simulation <code>(Level 1 → Level 2 → Level 3)</code>, matching exactly what the role calls for:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Assessment Level</th>
<th>Focus Area</th>
<th>Real-World Equivalent</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Level 1</strong></td>
<td>Database Fundamentals</td>
<td>"Won't break production"</td>
</tr>
<tr>
<td><strong>Level 2</strong></td>
<td>Data Integration &amp; Reporting</td>
<td>"Can generate reports and move data between systems"</td>
</tr>
<tr>
<td><strong>Level 3</strong></td>
<td>Production Operations</td>
<td>"Documents well, trains staff, operates safely"</td>
</tr>
</tbody>
</table></div>

<p>📦 <a href="https://github.com/lfariabr/stc-datalab" rel="noopener noreferrer">StC DataLab Repo</a></p>




<h2>
  
  
  Execution Plan (Dec 2025 - Jan 2026)
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Date</th>
<th>Focus</th>
<th>Deliverables</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Dec 20-21</strong></td>
<td>Setup &amp; Schema</td>
<td>SQL Server Express + SSMS installation, DB creation, table structure</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Dec 22-23</strong></td>
<td>Backup &amp; Restore</td>
<td>Full backup/restore procedures, documentation with screenshots</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Dec 24-25</strong></td>
<td>Data Generation</td>
<td>Realistic seed data with edge cases</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Dec 26-27</strong></td>
<td>Reporting Views</td>
<td>Student profiles, class rolls, attendance summaries</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Dec 28-29</strong></td>
<td>Stored Procedures</td>
<td>Parameter-based queries, optimization</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Dec 30-31</strong></td>
<td>Import/Export</td>
<td>CSV handling, staging tables, data validation</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Jan 1-2</strong></td>
<td>Runbook &amp; Documentation</td>
<td>Operational procedures, troubleshooting guide</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Jan 3-4</strong></td>
<td>Demo Preparation</td>
<td>Presentation script, screenshots, talking points</td>
<td>✅</td>
</tr>
<tr>
<td><strong>Jan 5</strong></td>
<td>Final Review</td>
<td>Validate all components, practice demo</td>
<td>✅</td>
</tr>
</tbody>
</table></div>

<blockquote>
<p><a href="https://github.com/search?q=repo%3Alfariabr%2Fmasters-swe-ai++stc+OR+%28stc%29+OR+std+OR+%28std%29&amp;type=commits&amp;s=committer-date&amp;o=desc" rel="noopener noreferrer">Complete changelog with 30+ commits</a></p>
</blockquote>




<h2>
  
  
  🤖 The Solution
</h2>

<h3>
  
  
  Building an Enterprise Data Infrastructure
</h3>

<p>What started as "learn SQL Server syntax" evolved into a complete operational simulation. Here's what I built:</p>

<h3>
  
  
  System Architecture
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszt4gnqj361vgjgkydvl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszt4gnqj361vgjgkydvl.png" alt="Mermaid Diagram with Data Flow" width="800" height="199"></a></p>

<p><a href="https://github.com/lfariabr/masters-swe-ai/blob/master/2025-T2/T2-Extra/stc_datalab/screenshots/ArchitectureOverview.jpeg" rel="noopener noreferrer">See it in full size</a></p>

<h3>
  
  
  <strong>Database Architecture</strong>
</h3>

<p>The foundation is a normalized relational database representing a school's core operational data:</p>

<p><strong>Core Tables (6):</strong></p>

<ul>
<li>
<strong>Students</strong> (200 records) — Privacy-sensitive fields including medical info, emergency contacts, and boarding status</li>
<li>
<strong>Staff</strong> (20 records) — Role-based attributes (Teacher, Principal, ICT, Admin, Counselor)</li>
<li>
<strong>Subjects</strong> (12 records) — Curriculum structure covering Math, English, Science, Humanities, Arts, Technology</li>
<li>
<strong>Classes</strong> (30 records) — Teacher assignments, room scheduling, year level groupings</li>
<li>
<strong>Enrollments</strong> (500 records) — Student-class relationships with status tracking (Active, Withdrawn, Completed, Pending)</li>
<li>
<strong>Attendance</strong> (800 records) — Daily tracking with status codes (Present, Absent, Late, Excused) across 10 days</li>
</ul>

<p><strong>Design Principles:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Example: Students table with constraints</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">Students</span> <span class="p">(</span>
    <span class="n">student_id</span> <span class="nb">INT</span> <span class="k">IDENTITY</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span><span class="mi">1</span><span class="p">)</span> <span class="k">PRIMARY</span> <span class="k">KEY</span><span class="p">,</span>
    <span class="n">student_number</span> <span class="n">NVARCHAR</span><span class="p">(</span><span class="mi">20</span><span class="p">)</span> <span class="k">UNIQUE</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">first_name</span> <span class="n">NVARCHAR</span><span class="p">(</span><span class="mi">50</span><span class="p">)</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">medical_info</span> <span class="n">NVARCHAR</span><span class="p">(</span><span class="mi">500</span><span class="p">),</span> <span class="c1">-- Privacy sensitive</span>
    <span class="n">emergency_contact</span> <span class="n">NVARCHAR</span><span class="p">(</span><span class="mi">100</span><span class="p">),</span>
    <span class="n">enrollment_year</span> <span class="nb">INT</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="k">INDEX</span> <span class="n">idx_enrollment_year</span> <span class="p">(</span><span class="n">enrollment_year</span><span class="p">),</span>
    <span class="k">INDEX</span> <span class="n">idx_student_number</span> <span class="p">(</span><span class="n">student_number</span><span class="p">)</span>
<span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  <strong>Operational Features</strong>
</h3>

<p><strong>1. Reporting Views (4 core views)</strong></p>

<ul>
<li>
<code>vw_StudentProfile</code> — Complete student records with emergency contacts</li>
<li>
<code>vw_ClassRoll</code> — Daily attendance with class lists and teacher assignments</li>
<li>
<code>vw_AttendanceDaily</code> — Roll call summaries with absence follow-up contacts</li>
<li>
<code>vw_EnrollmentSummary</code> — Class capacity planning with utilization metrics</li>
</ul>

<p><strong>2. Stored Procedures (4 parameterized)</strong></p>

<ul>
<li>
<code>sp_GetStudentProfile</code> — Multi-result set with profile + enrollments + attendance</li>
<li>
<code>sp_EnrollmentSummaryByYear</code> — Year-level filtering with capacity indicators</li>
<li>
<code>sp_AttendanceByDate</code> — Date range queries for specific time periods</li>
<li>
<code>sp_GetTableDataExport</code> — Generic data export for Power BI integration</li>
</ul>

<p><strong>3. Data Integration Pipeline</strong></p>

<ul>
<li>CSV import staging tables with validation rules</li>
<li>Referential integrity checks before production load</li>
<li>Error logging and rollback procedures</li>
<li>Export functionality for SEQTA/Power BI sync</li>
</ul>

<p><strong>4. Backup &amp; Recovery</strong></p>

<ul>
<li>Full backup T-SQL scripts (SQL Server Express compatible)</li>
<li>Differential backup procedures</li>
<li>Three-stage restore validation (verify → test → production)</li>
<li>RPO: 1 hour | RTO: 30 minutes</li>
</ul>

<h3>
  
  
  <strong>How It Works in Practice</strong>
</h3>

<p><strong>Scenario 1: Morning Roll Call</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Teacher logs in at 8:45 AM, needs today's class roll</span>
<span class="k">EXEC</span> <span class="n">sp_AttendanceByDate</span> 
    <span class="o">@</span><span class="n">StartDate</span> <span class="o">=</span> <span class="s1">'2025-01-22'</span><span class="p">,</span> 
    <span class="o">@</span><span class="n">EndDate</span> <span class="o">=</span> <span class="s1">'2025-01-22'</span><span class="p">;</span>
<span class="c1">-- Returns: Student list with attendance status, emergency contacts for absences</span>
</code></pre>

</div>



<p><strong>Scenario 2: Semester Planning</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Leadership needs Year 7 enrollment metrics for 2026 planning</span>
<span class="k">EXEC</span> <span class="n">sp_EnrollmentSummaryByYear</span> <span class="o">@</span><span class="n">EnrollmentYear</span> <span class="o">=</span> <span class="mi">2026</span><span class="p">;</span>
<span class="c1">-- Returns: Class utilization, capacity warnings, subject distribution</span>
</code></pre>

</div>



<p><strong>Scenario 3: System Integration</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- SEQTA export runs daily at 6 AM, imports new attendance data</span>
<span class="c1">-- 1. Load CSV into staging table</span>
<span class="c1">-- 2. Validate referential integrity (all student_ids exist)</span>
<span class="c1">-- 3. Merge into production Attendance table</span>
<span class="c1">-- 4. Log success/failures for monitoring</span>
</code></pre>

</div>



<h3>
  
  
  <strong>Data Quality by Design</strong>
</h3>

<p>Intentional edge cases throughout the seed data:</p>

<ul>
<li>
<strong>NULL values</strong> — Missing phone numbers (9%), NULL emergency contacts</li>
<li>
<strong>Casing inconsistencies</strong> — Lowercase first names, uppercase emails, trailing spaces</li>
<li>
<strong>International scenarios</strong> — Singapore/Jakarta addresses for boarding students</li>
<li>
<strong>Duplicate data</strong> — Shared email addresses to test deduplication logic</li>
<li>
<strong>Invalid formats</strong> — Phone numbers marked as '???', incomplete grades ('INC')</li>
</ul>

<p>This messy data simulates real school system exports (SEQTA, Synergetic) where cleaning and validation are critical.</p>




<h2>
  
  
  Tech Stack
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Technology</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Database</strong></td>
<td>SQL Server 2022 Express</td>
<td>On-premise simulation (macOS via Docker)</td>
</tr>
<tr>
<td><strong>Management</strong></td>
<td>SSMS + sqlcmd CLI</td>
<td>GUI and scripted operations</td>
</tr>
<tr>
<td><strong>Data Generation</strong></td>
<td>T-SQL CTEs + temp tables</td>
<td>Deterministic seed data with edge cases</td>
</tr>
<tr>
<td><strong>Backup</strong></td>
<td>Native SQL Server backups</td>
<td>Full/differential with RPO/RTO targets</td>
</tr>
<tr>
<td><strong>Integration</strong></td>
<td>CSV imports via BULK INSERT</td>
<td>Simulates SEQTA/Synergetic exports</td>
</tr>
<tr>
<td><strong>Documentation</strong></td>
<td>Markdown + Mermaid</td>
<td>Runbooks, training guides, flowcharts</td>
</tr>
</tbody>
</table></div>

<p><strong>Project Structure:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>stc_datalab/
├── sql/
│   ├── 00_create_db.sql          # Initial database creation
│   ├── 01_schema.sql             # Tables, constraints, indexes
│   ├── 02_seed_data.sql          # 1,500+ records with edge cases
│   ├── 03_views.sql              # 4 reporting views
│   ├── 04_stored_procedures.sql  # 4 parameterized SPs
│   ├── 05_import_export.sql      # CSV integration logic
│   └── 07_backup_restore.sql     # Backup/recovery procedures
├── data/
│   ├── students_import.csv       # Sample import data
│   ├── classes_import.csv
│   └── enrollments_import.csv
├── docs/
│   ├── Assessment1/              # Level 1: Setup &amp; basics
│   ├── Assessment2/              # Level 2: Integration
│   └── Assessment3/              # Level 3: Operations
│       ├── 06_runbook.md         # 1,000+ line operational guide
│       ├── 07_demo_script.md     # Interview presentation
│       └── 08_staff_training_guide.md
└── screenshots/                  # 15+ annotated screenshots
</code></pre>

</div>






<h2>
  
  
  The Impact: Confidence Through Deliverables
</h2>

<p>This wasn't just practice—it was portfolio-building with interview-ready artifacts:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Technical documentation</strong></td>
<td>3,000+ lines across 15 files</td>
</tr>
<tr>
<td><strong>SQL scripts</strong></td>
<td>10 files, 800+ lines of T-SQL</td>
</tr>
<tr>
<td><strong>Seed data generated</strong></td>
<td>1,562 records across 6 tables</td>
</tr>
<tr>
<td><strong>Views &amp; procedures</strong></td>
<td>8 reusable database objects</td>
</tr>
<tr>
<td><strong>Operational runbook</strong></td>
<td>1,000+ lines with flowcharts</td>
</tr>
<tr>
<td><strong>Training materials</strong></td>
<td>Non-technical staff guide</td>
</tr>
<tr>
<td><strong>Time investment</strong></td>
<td>16 days, committed execution</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  <strong>What This Proves</strong>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Skill Category</th>
<th>Evidence</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Database fundamentals</strong></td>
<td>Schema design with normalization, constraints, indexes</td>
</tr>
<tr>
<td><strong>T-SQL proficiency</strong></td>
<td>CTEs, window functions, stored procedures, error handling</td>
</tr>
<tr>
<td><strong>Data integration</strong></td>
<td>CSV imports with staging, validation, rollback procedures</td>
</tr>
<tr>
<td><strong>Backup/recovery</strong></td>
<td>Full/differential backups, 3-stage restore validation</td>
</tr>
<tr>
<td><strong>Documentation</strong></td>
<td>Runbooks, training guides, troubleshooting flowcharts</td>
</tr>
<tr>
<td><strong>Production mindset</strong></td>
<td>Security (least privilege), audit logging, change management</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  <strong>Interview Readiness</strong>
</h3>

<p>Instead of saying <em>"I can learn SQL Server"</em>, I can now walk into an interview and say:</p>

<blockquote>
<p><em>"I built a production-grade school data platform with 6 normalized tables, 8 reporting objects, comprehensive backup procedures, and operational documentation. Here's the GitHub repo, here's the demo script, and here are the 15 annotated screenshots. Let me show you the runbook."</em></p>
</blockquote>




<h2>
  
  
  Future Roadmap: From Simulation to Production
</h2>

<p>While this project is interview-focused, the architecture supports real-world expansion:</p>

<h3>
  
  
  <strong>1. Power BI Dashboard Integration</strong>
</h3>

<ul>
<li>Connect reporting views to interactive dashboards</li>
<li>Real-time attendance monitoring with alerting</li>
<li>Enrollment trend analysis across years</li>
<li>Teacher workload visualization</li>
</ul>

<h3>
  
  
  <strong>2. Automated SEQTA Sync</strong>
</h3>

<ul>
<li>Scheduled SSIS packages for nightly imports</li>
<li>Incremental updates with change data capture</li>
<li>Email notifications on import failures</li>
<li>Data quality scorecards</li>
</ul>

<h3>
  
  
  <strong>3. Advanced Security &amp; Compliance</strong>
</h3>

<ul>
<li>Row-level security based on staff roles</li>
<li>Transparent data encryption for medical_info</li>
<li>Audit tables with temporal queries</li>
<li>GDPR-compliant data retention policies</li>
</ul>

<h3>
  
  
  <strong>4. Performance Optimization</strong>
</h3>

<ul>
<li>Columnstore indexes for historical reporting</li>
<li>Query Store analysis for slow queries</li>
<li>Database partitioning by enrollment_year</li>
<li>Read replicas for Power BI loads</li>
</ul>

<h3>
  
  
  <strong>5. Cloud Migration Path</strong>
</h3>

<ul>
<li>Azure SQL Database deployment</li>
<li>Geo-replication for disaster recovery</li>
<li>Azure Data Factory for ETL orchestration</li>
<li>Integration with Microsoft 365 (SharePoint, Teams)</li>
</ul>




<h2>
  
  
  Key Takeaways
</h2>

<p>This project reinforced several engineering principles:</p>

<ol>
<li>
<strong>Build to prove, not just to practice</strong> — Every decision was portfolio-oriented</li>
<li>
<strong>Documentation = Deliverable</strong> — The runbook is as important as the code</li>
<li>
<strong>Simulate real constraints</strong> — SQL Server Express limits forced production-ready design</li>
<li>
<strong>Edge cases reveal skill</strong> — Intentional data quality issues prove validation competency</li>
<li>
<strong>Timeline discipline</strong> — 16-day execution plan kept momentum and accountability</li>
</ol>




<h2>
  
  
  Try It Yourself
</h2>

<p>The complete project is open source and ready to deploy:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Resource</th>
<th>Link</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>GitHub Repo</strong></td>
<td><a href="https://github.com/lfariabr/stc-datalab" rel="noopener noreferrer">github.com/lfariabr/stc-datalab</a></td>
</tr>
<tr>
<td><strong>Setup Guide</strong></td>
<td><a href="https://github.com/lfariabr/stc-datalab/tree/main/docs/Assessment1" rel="noopener noreferrer">Assessment 1 Documentation</a></td>
</tr>
<tr>
<td><strong>Operational Runbook</strong></td>
<td><a href="https://github.com/lfariabr/stc-datalab/blob/main/docs/Assessment3/06_runbook.md" rel="noopener noreferrer">06_runbook.md</a></td>
</tr>
<tr>
<td><strong>Demo Script</strong></td>
<td><a href="https://github.com/lfariabr/stc-datalab/blob/main/docs/Assessment3/07_demo_script.md" rel="noopener noreferrer">07_demo_script.md</a></td>
</tr>
</tbody>
</table></div>

<p><strong>Quick Start (Docker):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Clone the repo</span>
git clone https://github.com/lfariabr/stc-datalab.git
<span class="nb">cd </span>stc-datalab

<span class="c"># 2. Start SQL Server Express</span>
docker run <span class="nt">-e</span> <span class="s2">"ACCEPT_EULA=Y"</span> <span class="se">\</span>
  <span class="nt">-e</span> <span class="s2">"MSSQL_SA_PASSWORD=StC_SchoolLab2025!"</span> <span class="se">\</span>
  <span class="nt">-e</span> <span class="s2">"MSSQL_PID=Express"</span> <span class="se">\</span>
  <span class="nt">-p</span> 1433:1433 <span class="nt">--name</span> sqlserver <span class="se">\</span>
  <span class="nt">-d</span> mcr.microsoft.com/mssql/server:2022-latest

<span class="c"># 3. Create database and schema</span>
sqlcmd <span class="nt">-S</span> localhost <span class="nt">-U</span> sa <span class="nt">-P</span> <span class="s1">'StC_SchoolLab2025!'</span> <span class="nt">-C</span> <span class="nt">-i</span> sql/00_create_db.sql
sqlcmd <span class="nt">-S</span> localhost <span class="nt">-U</span> sa <span class="nt">-P</span> <span class="s1">'StC_SchoolLab2025!'</span> <span class="nt">-C</span> <span class="nt">-i</span> sql/01_schema.sql

<span class="c"># 4. Seed demo data</span>
sqlcmd <span class="nt">-S</span> localhost <span class="nt">-U</span> sa <span class="nt">-P</span> <span class="s1">'StC_SchoolLab2025!'</span> <span class="nt">-C</span> <span class="nt">-i</span> sql/02_seed_data.sql

<span class="c"># 5. Test reporting</span>
sqlcmd <span class="nt">-S</span> localhost <span class="nt">-U</span> sa <span class="nt">-P</span> <span class="s1">'StC_SchoolLab2025!'</span> <span class="nt">-C</span> <span class="nt">-Q</span> <span class="se">\</span>
  <span class="s2">"USE StC_SchoolLab; EXEC sp_AttendanceByDate @StartDate='2025-01-22', @EndDate='2025-01-22';"</span>
</code></pre>

</div>






<h2>
  
  
  Let's Connect!
</h2>

<p>This project exemplifies my approach to technical challenges: structured execution, production-quality deliverables, and comprehensive documentation. If you're:</p>

<ul>
<li>Building enterprise data systems</li>
<li>Working with SQL Server in education/non-profit sectors</li>
<li>Interested in data engineering best practices</li>
<li>Hiring for database administration roles</li>
</ul>

<p>I'd love to connect:</p>

<ul>
<li>
<strong>LinkedIn:</strong> <a href="https://www.linkedin.com/in/lfariabr/" rel="noopener noreferrer">linkedin.com/in/lfariabr</a>
</li>
<li>
<strong>GitHub:</strong> <a href="https://github.com/lfariabr" rel="noopener noreferrer">github.com/lfariabr</a>
</li>
<li>
<strong>Portfolio:</strong> <a href="https://luisfaria.dev" rel="noopener noreferrer">luisfaria.dev</a>
</li>
</ul>




<p><strong>Tech Stack Summary:</strong></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Current Implementation</th>
<th>Production Extensions</th>
</tr>
</thead>
<tbody>
<tr>
<td>SQL Server 2022 Express, SSMS, T-SQL, Docker, Markdown</td>
<td>Azure SQL Database, SSIS, Power BI, Columnstore Indexes, TDE, Azure Data Factory</td>
</tr>
</tbody>
</table></div>




<p><em>Built with 🎓 and database discipline by <a href="https://luisfaria.dev" rel="noopener noreferrer">Luis Faria</a></em></p>

<blockquote>
<p><em>Hard work is my preferred language and I try to speak it fluently.</em></p>
</blockquote>

