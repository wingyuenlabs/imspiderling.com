---
Title: Building Production-Ready Multi-Tenant SaaS in Rust with Actix-web
Description: 
Author: chinedu
Date: 2026-03-22T21:54:58.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>How I built tenant isolation for SmartFarmAI a poultry farm management platform serving farms across Nigeria and Tanzania.</strong></p>

<p>I've been building <a href="https://smartfarmai.ng/" rel="noopener noreferrer">SmartFarmAI</a>, an AI-powered poultry farm management platform, and one of the hardest architectural decisions I had to make early on was: <strong>how do I safely isolate data between farms?</strong></p>

<p>When a farmer in Lagos logs in and checks their egg production numbers, they should never under any circumstance see data from a 60,000-bird enterprise operation in Tanzania. One bug, one missed <code>WHERE</code> clause, and you're leaking customer data. In agriculture, that's not just a privacy issue it's a business-ending trust violation.</p>

<p>This article walks through exactly how I solved this using <strong>Rust, Actix-web, and PostgreSQL Row-Level Security (RLS)</strong> the patterns, the gotchas I hit in production, and the code that keeps it all safe.</p>

<p>If you're building any kind of multi-tenant SaaS in Rust, this should save you weeks of trial and error.</p>

<p><strong><em>The pattern in a nutshell:</em></strong> <em>Actix middleware extracts the tenant from each request, starts a DB transaction, calls</em> <code>SET LOCAL app.current_tenant_id</code><em>, and lets Postgres RLS policies automatically scope every query no manual</em> <code>WHERE org_id = $1</code> <em>needed.</em></p>

<p><em>This article assumes you're comfortable with Rust basics, Actix-web routing, and have at least heard of PostgreSQL Row-Level Security. If you've never touched RLS before, you'll still be able to follow along I introduce the concepts as we go.</em></p>

<h2>
  
  
  Multi-Tenancy: Pick Your Strategy
</h2>

<p>Before writing any code, you have to decide how to isolate tenant data. There are three common approaches:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fntn4y49ai8uja906d49b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fntn4y49ai8uja906d49b.png" alt="Database-per-Tenant: Architecture diagram illustrating the database-per-tenant multi-tenancy pattern and its operational challenges. A central PostgreSQL cluster connects to many separate databases, DB Tenant 1 through DB Tenant 6 and beyond to 100+, each represented as individual database icons spreading across the top of the diagram. Red arrows on the left depict a management and migration nightmare, showing how update scripts must be rolled out individually to every tenant database with clock icons indicating time delays. Green arrows on the right show the benefit of maximum isolation, with each tenant receiving completely separate data access. The bottom notes that this approach requires Docker/K8s at scale, complex Prometheus/Grafana monitoring, and coordinated Redis migration tooling for rollouts." width="800" height="446"></a></p>

<p><strong>Database-per-tenant</strong>: Every customer gets their own PostgreSQL database. Maximum isolation, but a nightmare to manage at scale. Migrations become a coordinated rollout across hundreds of databases.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1h3t16i6iu6m6cqy0ldq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1h3t16i6iu6m6cqy0ldq.png" alt="Schema-per-Tenant: Architecture diagram showing the schema-per-tenant multi-tenancy pattern as a middle ground between shared database and database-per-tenant approaches. A single PostgreSQL instance contains multiple colored schema segments. Schema Tenant 1, Schema Tenant 2, through Schema 100+ represented as pie-slice sections within one database. Red arrows on the left illustrate schema migration pain at scale, as each schema requires individual migration rollout. The top right shows a tangled knot of lines representing connection pooling complexity as the number of schemas grows. Green arrows on the right show isolated data access per tenant. The bottom notes Docker/K8s scaling is better than database-per-tenant, but still requires complex monitoring and coordinated rollout tooling." width="800" height="446"></a></p>

<p><strong>Schema-per-tenant</strong>: One database, but each customer gets their own schema. Better than separate databases, but schema migrations are still painful and connection pooling gets complicated fast.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F10ay6dd3ahv11n2dmxud.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F10ay6dd3ahv11n2dmxud.png" alt="Shared Database with RLS: " width="800" height="446"></a></p>

<p><strong>Shared database with Row-Level Security</strong>: One database, one schema, one set of tables but PostgreSQL itself enforces which rows each tenant can see. This is what I chose for SmartFarmAI.</p>

<p>Why RLS? Because it pushes security enforcement down to the database layer. My application code doesn't need to remember to add <code>WHERE org_id = $1</code> to every query. PostgreSQL does it automatically. If I forget a filter in my Rust code? Doesn't matter the database won't return rows that don't belong to the current tenant.</p>

<p>For an early-stage SaaS serving poultry farmers, this gives me the isolation guarantees I need without the operational overhead of managing hundreds of databases.</p>

<h2>
  
  
  The Data Model: org_id Everywhere
</h2>

<p>SmartFarmAI's domain hierarchy looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Organization (tenant)
  └── Farm(s)
       └── Batch(es) / Pen House(s)
            └── Egg Records, Mortality Logs, Vaccination Schedules...
</code></pre>

</div>



<p>The key decision: <strong>every table carries</strong> <code>org_id</code>, even child tables. You might think "batches already belong to a farm, and farms belong to an org why duplicate <code>org_id</code> on the batch table?" Two reasons:</p>

<ol>
<li><p><strong>RLS policies need a direct column reference.</strong> PostgreSQL can't follow foreign key chains in a policy expression. The policy needs to see <code>org_id</code> on the row it's evaluating.</p></li>
<li><p><strong>Composite foreign keys enforce correctness.</strong> A batch doesn't just reference a <code>farm_id</code> it references <code>(org_id, farm_id)</code>. This makes it structurally impossible to accidentally associate a batch with a farm from a different organization.</p></li>
</ol>

<p>Here's what the schema looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Organizations (tenants)</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">organizations</span> <span class="p">(</span>
    <span class="n">id</span> <span class="n">UUID</span> <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="k">DEFAULT</span> <span class="n">gen_random_uuid</span><span class="p">(),</span>
    <span class="n">name</span> <span class="nb">TEXT</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">created_at</span> <span class="n">TIMESTAMPTZ</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">()</span>
<span class="p">);</span>

<span class="c1">-- Farms belong to an organization</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">farms</span> <span class="p">(</span>
    <span class="n">id</span> <span class="n">UUID</span> <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="k">DEFAULT</span> <span class="n">gen_random_uuid</span><span class="p">(),</span>
    <span class="n">org_id</span> <span class="n">UUID</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">REFERENCES</span> <span class="n">organizations</span><span class="p">(</span><span class="n">id</span><span class="p">),</span>
    <span class="n">name</span> <span class="nb">TEXT</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="k">location</span> <span class="nb">TEXT</span><span class="p">,</span>
    <span class="n">created_at</span> <span class="n">TIMESTAMPTZ</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">(),</span>

    <span class="c1">-- Composite unique for child FK references</span>
    <span class="k">UNIQUE</span> <span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">id</span><span class="p">)</span>
<span class="p">);</span>

<span class="c1">-- Batches belong to a farm, scoped by org</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">batches</span> <span class="p">(</span>
    <span class="n">id</span> <span class="n">UUID</span> <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="k">DEFAULT</span> <span class="n">gen_random_uuid</span><span class="p">(),</span>
    <span class="n">org_id</span> <span class="n">UUID</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">farm_id</span> <span class="n">UUID</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">batch_name</span> <span class="nb">TEXT</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">bird_count</span> <span class="nb">INTEGER</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="mi">0</span><span class="p">,</span>
    <span class="n">created_at</span> <span class="n">TIMESTAMPTZ</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">(),</span>

    <span class="c1">-- Composite FK: can't assign a batch to a farm in a different org</span>
    <span class="k">CONSTRAINT</span> <span class="n">batch_farm_fk</span>
        <span class="k">FOREIGN</span> <span class="k">KEY</span> <span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">farm_id</span><span class="p">)</span>
        <span class="k">REFERENCES</span> <span class="n">farms</span> <span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">id</span><span class="p">)</span>
        <span class="k">ON</span> <span class="k">DELETE</span> <span class="k">CASCADE</span><span class="p">,</span>

    <span class="k">UNIQUE</span> <span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">farm_id</span><span class="p">,</span> <span class="n">id</span><span class="p">)</span>
<span class="p">);</span>

<span class="c1">-- Egg production records</span>
<span class="k">CREATE</span> <span class="k">TABLE</span> <span class="n">egg_production_logs</span> <span class="p">(</span>
    <span class="n">id</span> <span class="n">UUID</span> <span class="k">PRIMARY</span> <span class="k">KEY</span> <span class="k">DEFAULT</span> <span class="n">gen_random_uuid</span><span class="p">(),</span>
    <span class="n">org_id</span> <span class="n">UUID</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">farm_id</span> <span class="n">UUID</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">batch_id</span> <span class="n">UUID</span> <span class="k">NOT</span> <span class="k">NULL</span><span class="p">,</span>
    <span class="n">record_date</span> <span class="nb">DATE</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="k">CURRENT_DATE</span><span class="p">,</span>
    <span class="n">eggs_collected</span> <span class="nb">INTEGER</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">CHECK</span> <span class="p">(</span><span class="n">eggs_collected</span> <span class="o">&gt;=</span> <span class="mi">0</span><span class="p">),</span>
    <span class="n">notes</span> <span class="nb">TEXT</span><span class="p">,</span>
    <span class="n">created_at</span> <span class="n">TIMESTAMPTZ</span> <span class="k">NOT</span> <span class="k">NULL</span> <span class="k">DEFAULT</span> <span class="n">NOW</span><span class="p">(),</span>

    <span class="k">CONSTRAINT</span> <span class="n">egg_log_batch_fk</span>
        <span class="k">FOREIGN</span> <span class="k">KEY</span> <span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">farm_id</span><span class="p">,</span> <span class="n">batch_id</span><span class="p">)</span>
        <span class="k">REFERENCES</span> <span class="n">batches</span> <span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">farm_id</span><span class="p">,</span> <span class="n">id</span><span class="p">)</span>
        <span class="k">ON</span> <span class="k">DELETE</span> <span class="k">CASCADE</span><span class="p">,</span>

    <span class="c1">-- One record per batch per day</span>
    <span class="k">CONSTRAINT</span> <span class="n">unique_egg_record</span> <span class="k">UNIQUE</span> <span class="p">(</span><span class="n">batch_id</span><span class="p">,</span> <span class="n">record_date</span><span class="p">)</span>
<span class="p">);</span>
</code></pre>

</div>



<p>Notice the pattern: every child table has a composite foreign key that threads <code>org_id</code> all the way down. This is intentional. It's one of those things that feels redundant until it saves you from a data leak.</p>

<h2>
  
  
  Enabling Row-Level Security
</h2>

<p>With the schema in place, enabling RLS is straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="c1">-- Enable RLS on all tenant-scoped tables</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">farms</span> <span class="n">ENABLE</span> <span class="k">ROW</span> <span class="k">LEVEL</span> <span class="k">SECURITY</span><span class="p">;</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">batches</span> <span class="n">ENABLE</span> <span class="k">ROW</span> <span class="k">LEVEL</span> <span class="k">SECURITY</span><span class="p">;</span>
<span class="k">ALTER</span> <span class="k">TABLE</span> <span class="n">egg_production_logs</span> <span class="n">ENABLE</span> <span class="k">ROW</span> <span class="k">LEVEL</span> <span class="k">SECURITY</span><span class="p">;</span>

<span class="c1">-- Create isolation policies</span>
<span class="k">CREATE</span> <span class="n">POLICY</span> <span class="n">tenant_isolation</span> <span class="k">ON</span> <span class="n">farms</span>
    <span class="k">USING</span> <span class="p">(</span><span class="n">org_id</span> <span class="o">=</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">)::</span><span class="n">uuid</span><span class="p">);</span>

<span class="k">CREATE</span> <span class="n">POLICY</span> <span class="n">tenant_isolation</span> <span class="k">ON</span> <span class="n">batches</span>
    <span class="k">USING</span> <span class="p">(</span><span class="n">org_id</span> <span class="o">=</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">)::</span><span class="n">uuid</span><span class="p">);</span>

<span class="k">CREATE</span> <span class="n">POLICY</span> <span class="n">tenant_isolation</span> <span class="k">ON</span> <span class="n">egg_production_logs</span>
    <span class="k">USING</span> <span class="p">(</span><span class="n">org_id</span> <span class="o">=</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">)::</span><span class="n">uuid</span><span class="p">);</span>

<span class="c1">-- WITH CHECK ensures inserts/updates also respect the tenant boundary</span>
<span class="k">CREATE</span> <span class="n">POLICY</span> <span class="n">tenant_write_isolation</span> <span class="k">ON</span> <span class="n">farms</span>
    <span class="k">FOR</span> <span class="k">INSERT</span>
    <span class="k">WITH</span> <span class="k">CHECK</span> <span class="p">(</span><span class="n">org_id</span> <span class="o">=</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">)::</span><span class="n">uuid</span><span class="p">);</span>

<span class="k">CREATE</span> <span class="n">POLICY</span> <span class="n">tenant_write_isolation</span> <span class="k">ON</span> <span class="n">batches</span>
    <span class="k">FOR</span> <span class="k">INSERT</span>
    <span class="k">WITH</span> <span class="k">CHECK</span> <span class="p">(</span><span class="n">org_id</span> <span class="o">=</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">)::</span><span class="n">uuid</span><span class="p">);</span>

<span class="k">CREATE</span> <span class="n">POLICY</span> <span class="n">tenant_write_isolation</span> <span class="k">ON</span> <span class="n">egg_production_logs</span>
    <span class="k">FOR</span> <span class="k">INSERT</span>
    <span class="k">WITH</span> <span class="k">CHECK</span> <span class="p">(</span><span class="n">org_id</span> <span class="o">=</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">)::</span><span class="n">uuid</span><span class="p">);</span>
</code></pre>

</div>



<p>The magic here is <code>current_setting('app.current_tenant_id')</code>. This is a PostgreSQL session variable not a table, not a function, just a key-value pair that lives for the duration of a transaction or session. Before any query runs, we set this variable to the current user's organization ID. PostgreSQL's RLS engine then uses it to filter every <code>SELECT</code>, <code>INSERT</code>, <code>UPDATE</code>, and <code>DELETE</code>.</p>

<p><strong>Important</strong>: RLS policies are enforced for regular users but <strong>not for the table owner or superusers</strong>. Make sure your application connects as a non-superuser role.</p>

<h2>
  
  
  The Middleware: Setting Tenant Context in Actix-web
</h2>

<p>This is where Rust and Actix-web come in. Every authenticated request needs to:</p>

<ol>
<li><p>Extract the tenant ID (from JWT claims)</p></li>
<li><p>Set the PostgreSQL session variable before any query runs</p></li>
<li><p>Ensure the variable is cleared when the request ends</p></li>
</ol>

<p>Here's how I built it:</p>

<h3>
  
  
  The TenantId Extractor
</h3>

<p>First, a newtype for tenant IDs. This prevents accidentally passing a user ID where an org ID is expected Rust's type system catches it at compile time.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">uuid</span><span class="p">::</span><span class="n">Uuid</span><span class="p">;</span>

<span class="nd">#[derive(Debug,</span> <span class="nd">Clone,</span> <span class="nd">Copy,</span> <span class="nd">PartialEq,</span> <span class="nd">Eq,</span> <span class="nd">Hash)]</span>
<span class="k">pub</span> <span class="k">struct</span> <span class="nf">TenantId</span><span class="p">(</span><span class="n">Uuid</span><span class="p">);</span>

<span class="k">impl</span> <span class="n">TenantId</span> <span class="p">{</span>
    <span class="k">pub</span> <span class="k">fn</span> <span class="nf">new</span><span class="p">(</span><span class="n">id</span><span class="p">:</span> <span class="n">Uuid</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="k">Self</span> <span class="p">{</span>
        <span class="k">Self</span><span class="p">(</span><span class="n">id</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="k">pub</span> <span class="k">fn</span> <span class="nf">as_uuid</span><span class="p">(</span><span class="o">&amp;</span><span class="k">self</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="o">&amp;</span><span class="n">Uuid</span> <span class="p">{</span>
        <span class="o">&amp;</span><span class="k">self</span><span class="na">.0</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">impl</span> <span class="nn">std</span><span class="p">::</span><span class="nn">fmt</span><span class="p">::</span><span class="n">Display</span> <span class="k">for</span> <span class="n">TenantId</span> <span class="p">{</span>
    <span class="k">fn</span> <span class="nf">fmt</span><span class="p">(</span><span class="o">&amp;</span><span class="k">self</span><span class="p">,</span> <span class="n">f</span><span class="p">:</span> <span class="o">&amp;</span><span class="k">mut</span> <span class="nn">std</span><span class="p">::</span><span class="nn">fmt</span><span class="p">::</span><span class="n">Formatter</span><span class="o">&lt;</span><span class="nv">'_</span><span class="o">&gt;</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="nn">std</span><span class="p">::</span><span class="nn">fmt</span><span class="p">::</span><span class="nb">Result</span> <span class="p">{</span>
        <span class="nd">write!</span><span class="p">(</span><span class="n">f</span><span class="p">,</span> <span class="s">"{}"</span><span class="p">,</span> <span class="k">self</span><span class="na">.0</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Extracting Tenant from JWT
</h3>

<p>In Actix-web, you can implement <code>FromRequest</code> to automatically extract the tenant ID from the JWT on every request:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">actix_web</span><span class="p">::{</span><span class="n">FromRequest</span><span class="p">,</span> <span class="n">HttpRequest</span><span class="p">,</span> <span class="nn">dev</span><span class="p">::</span><span class="n">Payload</span><span class="p">,</span> <span class="n">Error</span><span class="p">,</span> <span class="n">HttpMessage</span><span class="p">};</span>
<span class="k">use</span> <span class="nn">std</span><span class="p">::</span><span class="nn">future</span><span class="p">::{</span><span class="n">Ready</span><span class="p">,</span> <span class="n">ready</span><span class="p">};</span>

<span class="k">impl</span> <span class="n">FromRequest</span> <span class="k">for</span> <span class="n">TenantId</span> <span class="p">{</span>
    <span class="k">type</span> <span class="n">Error</span> <span class="o">=</span> <span class="n">Error</span><span class="p">;</span>
    <span class="k">type</span> <span class="n">Future</span> <span class="o">=</span> <span class="n">Ready</span><span class="o">&lt;</span><span class="nb">Result</span><span class="o">&lt;</span><span class="k">Self</span><span class="p">,</span> <span class="k">Self</span><span class="p">::</span><span class="n">Error</span><span class="o">&gt;&gt;</span><span class="p">;</span>

    <span class="k">fn</span> <span class="nf">from_request</span><span class="p">(</span><span class="n">req</span><span class="p">:</span> <span class="o">&amp;</span><span class="n">HttpRequest</span><span class="p">,</span> <span class="n">_payload</span><span class="p">:</span> <span class="o">&amp;</span><span class="k">mut</span> <span class="n">Payload</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="k">Self</span><span class="p">::</span><span class="n">Future</span> <span class="p">{</span>
        <span class="c1">// The auth middleware has already validated the JWT</span>
        <span class="c1">// and inserted claims into request extensions</span>
        <span class="k">match</span> <span class="n">req</span><span class="nf">.extensions</span><span class="p">()</span><span class="py">.get</span><span class="p">::</span><span class="o">&lt;</span><span class="n">AuthClaims</span><span class="o">&gt;</span><span class="p">()</span> <span class="p">{</span>
            <span class="nf">Some</span><span class="p">(</span><span class="n">claims</span><span class="p">)</span> <span class="k">=&gt;</span> <span class="nf">ready</span><span class="p">(</span><span class="nf">Ok</span><span class="p">(</span><span class="nn">TenantId</span><span class="p">::</span><span class="nf">new</span><span class="p">(</span><span class="n">claims</span><span class="py">.org_id</span><span class="p">))),</span>
            <span class="nb">None</span> <span class="k">=&gt;</span> <span class="nf">ready</span><span class="p">(</span><span class="nf">Err</span><span class="p">(</span><span class="nn">actix_web</span><span class="p">::</span><span class="nn">error</span><span class="p">::</span><span class="nf">ErrorUnauthorized</span><span class="p">(</span>
                <span class="s">"Missing tenant context"</span>
            <span class="p">))),</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  The RLS Middleware
</h3>

<p>Now the critical piece the middleware that sets <code>app.current_tenant_id</code> before your handler runs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">actix_web</span><span class="p">::{</span><span class="nn">dev</span><span class="p">::</span><span class="n">ServiceRequest</span><span class="p">,</span> <span class="n">Error</span><span class="p">,</span> <span class="n">HttpMessage</span><span class="p">};</span>
<span class="k">use</span> <span class="nn">sqlx</span><span class="p">::</span><span class="n">PgPool</span><span class="p">;</span>

<span class="k">pub</span> <span class="k">async</span> <span class="k">fn</span> <span class="nf">set_rls_context</span><span class="p">(</span>
    <span class="n">pool</span><span class="p">:</span> <span class="o">&amp;</span><span class="n">PgPool</span><span class="p">,</span>
    <span class="n">org_id</span><span class="p">:</span> <span class="o">&amp;</span><span class="n">TenantId</span><span class="p">,</span>
<span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="nn">sqlx</span><span class="p">::</span><span class="n">Transaction</span><span class="o">&lt;</span><span class="nv">'_</span><span class="p">,</span> <span class="nn">sqlx</span><span class="p">::</span><span class="n">Postgres</span><span class="o">&gt;</span><span class="p">,</span> <span class="n">Error</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">tx</span> <span class="o">=</span> <span class="n">pool</span><span class="nf">.begin</span><span class="p">()</span><span class="k">.await</span><span class="nf">.map_err</span><span class="p">(|</span><span class="n">e</span><span class="p">|</span> <span class="p">{</span>
        <span class="nn">actix_web</span><span class="p">::</span><span class="nn">error</span><span class="p">::</span><span class="nf">ErrorInternalServerError</span><span class="p">(</span><span class="nd">format!</span><span class="p">(</span><span class="s">"DB error: {}"</span><span class="p">,</span> <span class="n">e</span><span class="p">))</span>
    <span class="p">})</span><span class="o">?</span><span class="p">;</span>

    <span class="c1">// SET LOCAL scopes the variable to this transaction only</span>
    <span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, true)"</span><span class="p">)</span>
        <span class="nf">.bind</span><span class="p">(</span><span class="n">org_id</span><span class="nf">.to_string</span><span class="p">())</span>
        <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.map_err</span><span class="p">(|</span><span class="n">e</span><span class="p">|</span> <span class="p">{</span>
            <span class="nn">actix_web</span><span class="p">::</span><span class="nn">error</span><span class="p">::</span><span class="nf">ErrorInternalServerError</span><span class="p">(</span><span class="nd">format!</span><span class="p">(</span><span class="s">"RLS context error: {}"</span><span class="p">,</span> <span class="n">e</span><span class="p">))</span>
        <span class="p">})</span><span class="o">?</span><span class="p">;</span>

    <span class="nf">Ok</span><span class="p">(</span><span class="n">tx</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The critical detail here is the third argument to <code>set_config</code>: <code>true</code> <strong>means transaction-local</strong>. When the transaction commits or rolls back, the variable is automatically cleared. This is essential for connection pool safety more on that below.</p>

<h3>
  
  
  Using It in Handlers
</h3>

<p>With the extractor and the RLS context function, your route handlers look clean:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">actix_web</span><span class="p">::{</span><span class="n">web</span><span class="p">,</span> <span class="n">HttpResponse</span><span class="p">,</span> <span class="nb">Result</span><span class="p">};</span>
<span class="k">use</span> <span class="nn">sqlx</span><span class="p">::</span><span class="n">PgPool</span><span class="p">;</span>

<span class="k">pub</span> <span class="k">async</span> <span class="k">fn</span> <span class="nf">get_farms</span><span class="p">(</span>
    <span class="n">pool</span><span class="p">:</span> <span class="nn">web</span><span class="p">::</span><span class="n">Data</span><span class="o">&lt;</span><span class="n">PgPool</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="n">tenant</span><span class="p">:</span> <span class="n">TenantId</span><span class="p">,</span>
<span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="n">HttpResponse</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">tx</span> <span class="o">=</span> <span class="nf">set_rls_context</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">tenant</span><span class="p">)</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>

    <span class="c1">// No WHERE clause needed — RLS handles filtering</span>
    <span class="k">let</span> <span class="n">farms</span> <span class="o">=</span> <span class="nn">sqlx</span><span class="p">::</span><span class="nd">query_as!</span><span class="p">(</span>
        <span class="n">Farm</span><span class="p">,</span>
        <span class="s">"SELECT id, org_id, name, location, created_at FROM farms"</span>
    <span class="p">)</span>
    <span class="nf">.fetch_all</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
    <span class="k">.await</span>
    <span class="nf">.map_err</span><span class="p">(</span><span class="nn">actix_web</span><span class="p">::</span><span class="nn">error</span><span class="p">::</span><span class="n">ErrorInternalServerError</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>

    <span class="n">tx</span><span class="nf">.commit</span><span class="p">()</span><span class="k">.await</span>
        <span class="nf">.map_err</span><span class="p">(</span><span class="nn">actix_web</span><span class="p">::</span><span class="nn">error</span><span class="p">::</span><span class="n">ErrorInternalServerError</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>

    <span class="nf">Ok</span><span class="p">(</span><span class="nn">HttpResponse</span><span class="p">::</span><span class="nf">Ok</span><span class="p">()</span><span class="nf">.json</span><span class="p">(</span><span class="n">farms</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Notice: <strong>no</strong> <code>WHERE org_id = $1</code>. The query selects all farms, but RLS ensures the farmer in Lagos only sees their own farms. The 60,000-bird operation in Tanzania is invisible to them at the database level.</p>

<h2>
  
  
  The Gotcha That Almost Burned Me: Connection Pools and Session Variables
</h2>

<p>Here's the mistake that will bite you if you're not careful.</p>

<p>PostgreSQL session variables persist for the lifetime of a connection. When you use a connection pool (like sqlx's <code>PgPool</code>), connections are reused across requests. If you set <code>app.current_tenant_id</code> with session scope (<code>false</code> as the third argument to <code>set_config</code>), here's what happens:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Request from Org-A arrives
2. Connection #5 is checked out from the pool
3. SET app.current_tenant_id = 'org-a-uuid'  (session-scoped)
4. Query runs, returns Org-A's data ✅
5. Connection #5 is returned to the pool

6. Request from Org-B arrives
7. Connection #5 is checked out again
8. But app.current_tenant_id is STILL 'org-a-uuid' !!
9. Query runs... returns Org-A's data to Org-B's user 🚨
</code></pre>

</div>



<p><strong>This is a data leak.</strong> And it's subtle it depends on which connection the pool hands out, so it won't happen consistently in development. It'll show up in production under load.</p>

<p>The fix: <strong>always use transaction-local scope.</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="c1">// ✅ SAFE: Transaction-local — cleared automatically when tx ends</span>
<span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, true)"</span><span class="p">)</span>
    <span class="nf">.bind</span><span class="p">(</span><span class="n">org_id</span><span class="nf">.to_string</span><span class="p">())</span>
    <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
    <span class="k">.await</span><span class="o">?</span><span class="p">;</span>

<span class="c1">// ❌ DANGEROUS: Session-local — persists after connection returns to pool</span>
<span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, false)"</span><span class="p">)</span>
    <span class="nf">.bind</span><span class="p">(</span><span class="n">org_id</span><span class="nf">.to_string</span><span class="p">())</span>
    <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">)</span>
    <span class="k">.await</span><span class="o">?</span><span class="p">;</span>
</code></pre>

</div>



<p>By wrapping every tenant-scoped operation in a transaction and using <code>set_config(..., true)</code>, the variable is automatically cleaned up when the transaction ends. No stale tenant context, no data leaks.</p>

<h2>
  
  
  When NOT to Use RLS: Background Workers and the Outbox Pattern
</h2>

<p>SmartFarmAI uses an outbox pattern for event-driven processing. When a farmer logs egg production, the API handler inserts the record and publishes a domain event to an <code>outbox_events</code> table all in the same transaction. A background worker then picks up events and processes them (updating batch totals, triggering AI predictions, etc.).</p>

<p>Here's the thing: <strong>the outbox table should NOT have RLS enabled.</strong></p>

<p>Why? The background worker needs to poll for events across all organizations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">outbox_events</span>
<span class="k">WHERE</span> <span class="n">processed_at</span> <span class="k">IS</span> <span class="k">NULL</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">created_at</span> <span class="k">ASC</span>
<span class="k">LIMIT</span> <span class="mi">10</span>
<span class="k">FOR</span> <span class="k">UPDATE</span> <span class="n">SKIP</span> <span class="n">LOCKED</span><span class="p">;</span>
</code></pre>

</div>



<p>If RLS were enabled on this table, the worker would need a tenant context to read events but it doesn't belong to any tenant. It's infrastructure.</p>

<p>My rule of thumb:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Table Type</th>
<th>RLS?</th>
<th>Why</th>
</tr>
</thead>
<tbody>
<tr>
<td>Business data (farms, batches, eggs)</td>
<td>Yes</td>
<td>Tenant-scoped, must be isolated</td>
</tr>
<tr>
<td>Work queues (outbox_events, email_jobs)</td>
<td>No</td>
<td>Infrastructure, cross-tenant by design</td>
</tr>
<tr>
<td>Audit logs</td>
<td>Depends</td>
<td>RLS if tenant-facing, no RLS if admin-only</td>
</tr>
</tbody>
</table></div>

<p>But here's the nuance: when the worker processes an event and needs to update business data, it <strong>does</strong> need to set the RLS context for that specific event's organization:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">pub</span> <span class="k">async</span> <span class="k">fn</span> <span class="nf">process_event</span><span class="p">(</span>
    <span class="o">&amp;</span><span class="k">self</span><span class="p">,</span>
    <span class="n">pool</span><span class="p">:</span> <span class="o">&amp;</span><span class="n">PgPool</span><span class="p">,</span>
    <span class="n">event</span><span class="p">:</span> <span class="o">&amp;</span><span class="n">OutboxEvent</span><span class="p">,</span>
<span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="p">(),</span> <span class="nb">Box</span><span class="o">&lt;</span><span class="k">dyn</span> <span class="nn">std</span><span class="p">::</span><span class="nn">error</span><span class="p">::</span><span class="n">Error</span><span class="o">&gt;&gt;</span> <span class="p">{</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">tx</span> <span class="o">=</span> <span class="n">pool</span><span class="nf">.begin</span><span class="p">()</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>

    <span class="c1">// Set RLS context for this event's organization</span>
    <span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, true)"</span><span class="p">)</span>
        <span class="nf">.bind</span><span class="p">(</span><span class="n">event</span><span class="py">.org_id</span><span class="nf">.to_string</span><span class="p">())</span>
        <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span><span class="o">?</span><span class="p">;</span>

    <span class="c1">// Now we can safely update tenant-scoped tables</span>
    <span class="k">match</span> <span class="n">event</span><span class="py">.topic</span><span class="nf">.as_str</span><span class="p">()</span> <span class="p">{</span>
        <span class="s">"EggProductionRecorded"</span> <span class="k">=&gt;</span> <span class="p">{</span>
            <span class="k">self</span><span class="nf">.update_batch_totals</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="n">tx</span><span class="p">,</span> <span class="n">event</span><span class="p">)</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="s">"MortalityRecorded"</span> <span class="k">=&gt;</span> <span class="p">{</span>
            <span class="k">self</span><span class="nf">.update_mortality_stats</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="n">tx</span><span class="p">,</span> <span class="n">event</span><span class="p">)</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="n">_</span> <span class="k">=&gt;</span> <span class="p">{}</span>
    <span class="p">}</span>

    <span class="c1">// Mark event as processed</span>
    <span class="nn">sqlx</span><span class="p">::</span><span class="nd">query!</span><span class="p">(</span>
        <span class="s">"UPDATE outbox_events SET processed_at = NOW() WHERE id = $1"</span><span class="p">,</span>
        <span class="n">event</span><span class="py">.id</span>
    <span class="p">)</span>
    <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
    <span class="k">.await</span><span class="o">?</span><span class="p">;</span>

    <span class="n">tx</span><span class="nf">.commit</span><span class="p">()</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>
    <span class="nf">Ok</span><span class="p">(())</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This is the pattern that took me the longest to get right. The worker reads from an unprotected queue, then enters a tenant context per-event to do the actual work. Each event is processed in its own transaction with its own RLS scope.</p>

<p>I actually hit a bug in production where my outbox table initially had RLS enabled. The worker was fetching zero events even though the table had hundreds queued up. No errors just silent filtering. It took me an embarrassing amount of debugging before I realised the worker had no tenant context set, so RLS was filtering out everything.</p>

<h2>
  
  
  Testing Multi-Tenant Isolation
</h2>

<p>Testing RLS requires more thought than testing regular business logic. You need to verify that tenants genuinely can't see each other's data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="nd">#[tokio::test]</span>
<span class="k">async</span> <span class="k">fn</span> <span class="nf">test_tenant_isolation</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">let</span> <span class="n">pool</span> <span class="o">=</span> <span class="nf">setup_test_db</span><span class="p">()</span><span class="k">.await</span><span class="p">;</span>

    <span class="c1">// Create two organizations</span>
    <span class="k">let</span> <span class="n">org_a</span> <span class="o">=</span> <span class="nf">create_test_org</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="s">"Farm Corp A"</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>
    <span class="k">let</span> <span class="n">org_b</span> <span class="o">=</span> <span class="nf">create_test_org</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="s">"Farm Corp B"</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>

    <span class="c1">// Create farms under each org (using superuser/direct insert)</span>
    <span class="nf">create_test_farm</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">org_a</span><span class="p">,</span> <span class="s">"Org A Farm"</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>
    <span class="nf">create_test_farm</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">org_b</span><span class="p">,</span> <span class="s">"Org B Farm"</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>

    <span class="c1">// Query as Org A — should only see their farm</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">tx</span> <span class="o">=</span> <span class="n">pool</span><span class="nf">.begin</span><span class="p">()</span><span class="k">.await</span><span class="nf">.unwrap</span><span class="p">();</span>
    <span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, true)"</span><span class="p">)</span>
        <span class="nf">.bind</span><span class="p">(</span><span class="n">org_a</span><span class="nf">.to_string</span><span class="p">())</span>
        <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.unwrap</span><span class="p">();</span>

    <span class="k">let</span> <span class="n">farms</span><span class="p">:</span> <span class="nb">Vec</span><span class="o">&lt;</span><span class="n">Farm</span><span class="o">&gt;</span> <span class="o">=</span> <span class="nn">sqlx</span><span class="p">::</span><span class="nd">query_as!</span><span class="p">(</span><span class="n">Farm</span><span class="p">,</span> <span class="s">"SELECT * FROM farms"</span><span class="p">)</span>
        <span class="nf">.fetch_all</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.unwrap</span><span class="p">();</span>

    <span class="nd">assert_eq!</span><span class="p">(</span><span class="n">farms</span><span class="nf">.len</span><span class="p">(),</span> <span class="mi">1</span><span class="p">);</span>
    <span class="nd">assert_eq!</span><span class="p">(</span><span class="n">farms</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="py">.name</span><span class="p">,</span> <span class="s">"Org A Farm"</span><span class="p">);</span>

    <span class="n">tx</span><span class="nf">.commit</span><span class="p">()</span><span class="k">.await</span><span class="nf">.unwrap</span><span class="p">();</span>

    <span class="c1">// Query as Org B — should only see their farm</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">tx</span> <span class="o">=</span> <span class="n">pool</span><span class="nf">.begin</span><span class="p">()</span><span class="k">.await</span><span class="nf">.unwrap</span><span class="p">();</span>
    <span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, true)"</span><span class="p">)</span>
        <span class="nf">.bind</span><span class="p">(</span><span class="n">org_b</span><span class="nf">.to_string</span><span class="p">())</span>
        <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.unwrap</span><span class="p">();</span>

    <span class="k">let</span> <span class="n">farms</span><span class="p">:</span> <span class="nb">Vec</span><span class="o">&lt;</span><span class="n">Farm</span><span class="o">&gt;</span> <span class="o">=</span> <span class="nn">sqlx</span><span class="p">::</span><span class="nd">query_as!</span><span class="p">(</span><span class="n">Farm</span><span class="p">,</span> <span class="s">"SELECT * FROM farms"</span><span class="p">)</span>
        <span class="nf">.fetch_all</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.unwrap</span><span class="p">();</span>

    <span class="nd">assert_eq!</span><span class="p">(</span><span class="n">farms</span><span class="nf">.len</span><span class="p">(),</span> <span class="mi">1</span><span class="p">);</span>
    <span class="nd">assert_eq!</span><span class="p">(</span><span class="n">farms</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="py">.name</span><span class="p">,</span> <span class="s">"Org B Farm"</span><span class="p">);</span>

    <span class="n">tx</span><span class="nf">.commit</span><span class="p">()</span><span class="k">.await</span><span class="nf">.unwrap</span><span class="p">();</span>
<span class="p">}</span>

<span class="nd">#[tokio::test]</span>
<span class="k">async</span> <span class="k">fn</span> <span class="nf">test_cross_tenant_insert_blocked</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">let</span> <span class="n">pool</span> <span class="o">=</span> <span class="nf">setup_test_db</span><span class="p">()</span><span class="k">.await</span><span class="p">;</span>

    <span class="k">let</span> <span class="n">org_a</span> <span class="o">=</span> <span class="nf">create_test_org</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="s">"Org A"</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>
    <span class="k">let</span> <span class="n">org_b</span> <span class="o">=</span> <span class="nf">create_test_org</span><span class="p">(</span><span class="o">&amp;</span><span class="n">pool</span><span class="p">,</span> <span class="s">"Org B"</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>

    <span class="c1">// Set context as Org A, but try to insert with Org B's ID</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">tx</span> <span class="o">=</span> <span class="n">pool</span><span class="nf">.begin</span><span class="p">()</span><span class="k">.await</span><span class="nf">.unwrap</span><span class="p">();</span>
    <span class="nn">sqlx</span><span class="p">::</span><span class="nf">query</span><span class="p">(</span><span class="s">"SELECT set_config('app.current_tenant_id', $1, true)"</span><span class="p">)</span>
        <span class="nf">.bind</span><span class="p">(</span><span class="n">org_a</span><span class="nf">.to_string</span><span class="p">())</span>
        <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
        <span class="k">.await</span>
        <span class="nf">.unwrap</span><span class="p">();</span>

    <span class="c1">// This should be rejected by the WITH CHECK policy</span>
    <span class="k">let</span> <span class="n">result</span> <span class="o">=</span> <span class="nn">sqlx</span><span class="p">::</span><span class="nd">query!</span><span class="p">(</span>
        <span class="s">"INSERT INTO farms (org_id, name) VALUES ($1, $2)"</span><span class="p">,</span>
        <span class="n">org_b</span><span class="nf">.as_uuid</span><span class="p">(),</span>  <span class="c1">// Wrong org!</span>
        <span class="s">"Sneaky Farm"</span>
    <span class="p">)</span>
    <span class="nf">.execute</span><span class="p">(</span><span class="o">&amp;</span><span class="k">mut</span> <span class="o">*</span><span class="n">tx</span><span class="p">)</span>
    <span class="k">.await</span><span class="p">;</span>

    <span class="nd">assert!</span><span class="p">(</span><span class="n">result</span><span class="nf">.is_err</span><span class="p">());</span> <span class="c1">// RLS blocks it</span>
<span class="p">}</span>
</code></pre>

</div>



<p>These tests are non-negotiable. Run them in CI. They're the safety net that catches regressions before your farmers do.</p>

<h2>
  
  
  Production Lessons from SmartFarmAI
</h2>

<p>After running this architecture in production with farms across Nigeria and Tanzania, here's what I've learned:</p>

<p><strong>1. RLS has near-zero performance overhead.</strong> PostgreSQL evaluates the policy as part of query planning. For simple equality checks like <code>org_id = current_setting(...)::uuid</code>, the optimiser handles it efficiently. I haven't needed to do anything special for performance.</p>

<p><strong>2. Always index</strong> <code>org_id</code><strong>.</strong> Even though RLS handles filtering, the database still needs to efficiently find matching rows. Composite indexes on <code>(org_id, ...)</code> are essential:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_farms_org</span> <span class="k">ON</span> <span class="n">farms</span><span class="p">(</span><span class="n">org_id</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_batches_org_farm</span> <span class="k">ON</span> <span class="n">batches</span><span class="p">(</span><span class="n">org_id</span><span class="p">,</span> <span class="n">farm_id</span><span class="p">);</span>
<span class="k">CREATE</span> <span class="k">INDEX</span> <span class="n">idx_egg_logs_batch_date</span> <span class="k">ON</span> <span class="n">egg_production_logs</span><span class="p">(</span><span class="n">batch_id</span><span class="p">,</span> <span class="n">record_date</span> <span class="k">DESC</span><span class="p">);</span>
</code></pre>

</div>



<p><strong>3. Migrations need care.</strong> When you run migrations, you're typically using a superuser or the table owner which bypasses RLS. This is fine and expected, but be aware of it. Don't write migration scripts that assume RLS is filtering data.</p>

<p><strong>4. Debug with</strong> <code>current_setting()</code><strong>.</strong> When something seems off, check what the database thinks the current tenant is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">SELECT</span> <span class="n">current_setting</span><span class="p">(</span><span class="s1">'app.current_tenant_id'</span><span class="p">,</span> <span class="k">true</span><span class="p">);</span>
<span class="c1">-- Returns NULL if not set, the tenant UUID if set</span>
</code></pre>

</div>



<p>The second argument (<code>true</code>) tells PostgreSQL to return <code>NULL</code> instead of throwing an error if the variable isn't set. Extremely useful for debugging.</p>

<p><strong>5. The outbox worker issue is real.</strong> As I mentioned earlier if you enable RLS on infrastructure tables, your background workers will silently return zero results. No errors, no warnings. Set up monitoring for your outbox queue depth so you catch this fast.</p>

<h2>
  
  
  Wrapping Up
</h2>

<p>Multi-tenant data isolation isn't the kind of thing you want to get "mostly right." It needs to be airtight. PostgreSQL's Row-Level Security, combined with Rust's type system and Actix-web's middleware patterns, gives you defense in depth:</p>

<ul>
<li><p><strong>Database layer</strong>: RLS policies enforce isolation regardless of application bugs.</p></li>
<li><p><strong>Application layer</strong>: Transaction-scoped <code>SET LOCAL</code> prevents connection pool contamination.</p></li>
</ul>

<p>SmartFarmAI currently manages farms ranging from 550 birds to 60,000 birds. Every single one of them shares the same database, the same tables, the same application code. And none of them can see each other's data.</p>

<p>If you're building multi-tenant SaaS in Rust, start with RLS. Push security as far down the stack as it will go. And let Rust's type system catch everything else.</p>




<p><em>I'm Chinedu Okere I write about building production systems in Rust. If you're working on multi-tenant architecture or SaaS in Rust, I'd love to hear about your approach. Find me on</em> <a href="https://x.com/chinedu_10" rel="noopener noreferrer"><em>Twitter/X</em></a> <em>or check out</em> <a href="https://smartfarmai.ng/" rel="noopener noreferrer"><em>SmartFarmAI</em></a><em>.</em></p>

