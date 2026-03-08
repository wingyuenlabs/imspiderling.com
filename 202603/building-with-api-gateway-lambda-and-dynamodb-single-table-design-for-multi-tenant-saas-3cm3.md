---
Title: Building with API Gateway, Lambda and DynamoDB Single-Table Design for Multi-Tenant SaaS
Description: 
Author: Renaldi
Date: 2026-03-08T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>When I build multi-tenant SaaS on AWS, one of my favorite combinations is <strong>API Gateway + Lambda + DynamoDB</strong>. It gives me a clean serverless control plane, fast iteration, and a lot of flexibility in how I model data and enforce tenancy.</p>

<p>In this post, I will walk through an end-to-end implementation pattern for a <strong>multi-tenant SaaS API</strong> using:</p>

<ul>
<li>
<strong>Amazon API Gateway</strong> for API ingress</li>
<li>
<strong>AWS Lambda</strong> for business logic</li>
<li>
<strong>Amazon DynamoDB</strong> with a <strong>single-table design</strong>
</li>
<li>
<strong>Amazon Cognito + JWT</strong> for authentication</li>
<li>
<strong>Per-tenant throttling and quotas</strong> using API Gateway usage plans (with a Lambda authorizer pattern)</li>
</ul>

<p>I will focus on the architectural and data modeling decisions that matter in production:</p>

<ul>
<li>Tenant isolation patterns</li>
<li>Partition key design</li>
<li>Hot partition mitigation</li>
<li>GSIs for access patterns</li>
<li>Auth context propagation (Cognito/JWT)</li>
<li>Per-tenant throttling and quotas</li>
</ul>

<p>I will also include code and an implementation walkthrough so you can adapt the pattern to your own SaaS.</p>




<h2>
  
  
  Why this stack works so well for multi-tenant SaaS
</h2>

<p>I like this stack because it lets me combine <strong>application-level tenancy controls</strong> with <strong>infrastructure-level scalability</strong>:</p>

<ul>
<li>
<strong>API Gateway</strong> gives me request routing, auth integration, throttling, and observability.</li>
<li>
<strong>Lambda</strong> gives me a stateless execution layer where I can consistently apply tenant-aware logic.</li>
<li>
<strong>DynamoDB</strong> gives me low-latency reads/writes and a data model that can be shaped around access patterns, which is exactly what multi-tenant SaaS needs.</li>
</ul>

<p>The biggest challenge is not deploying these services. It is designing them so that one tenant cannot interfere with another tenant, and so that one noisy tenant does not dominate throughput.</p>

<p>That is where the design details matter.</p>




<h2>
  
  
  What I am building in this walkthrough
</h2>

<p>To make the examples concrete, I will use a simple <strong>B2B issue tracking SaaS</strong>:</p>

<ul>
<li>Each <strong>tenant</strong> is a customer organization</li>
<li>Each tenant has:

<ul>
<li>users</li>
<li>projects</li>
<li>tickets</li>
<li>comments</li>
</ul>


</li>

<li>Users authenticate with <strong>Cognito</strong>
</li>

<li>API calls are authorized via JWT and mapped to a tenant context</li>

<li>Data lives in one DynamoDB table (single-table design)</li>

</ul>

<p>Example endpoints:</p>

<ul>
<li><code>POST /projects/{projectId}/tickets</code></li>
<li><code>GET /projects/{projectId}/tickets?status=OPEN</code></li>
<li><code>GET /me/tickets</code></li>
<li><code>POST /tickets/{ticketId}/comments</code></li>
</ul>

<p>Important design principle:</p>

<blockquote>
<p>I do not trust the client to tell me the tenant ID in the path/body for authorization purposes. I derive tenancy from the <strong>validated JWT</strong>, then enforce it in my Lambda logic and DynamoDB key design.</p>
</blockquote>




<h2>
  
  
  Architecture Overview
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxpcmu4r9xgxldubuqxoo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxpcmu4r9xgxldubuqxoo.png" alt=" " width="800" height="391"></a></p>

<p>This architecture uses:</p>

<ol>
<li>
<strong>Cognito</strong> to issue JWTs</li>
<li>
<strong>API Gateway (REST API)</strong> as the front door</li>
<li>
<strong>Lambda Authorizer</strong> to validate JWT and inject normalized auth context</li>
<li>
<strong>Lambda handlers</strong> for business operations</li>
<li>
<strong>DynamoDB single table</strong> for all entities</li>
<li>
<strong>API Gateway usage plans</strong> for per-tenant throttling/quota (via authorizer metering key)</li>
</ol>

<p>I am intentionally using a <strong>Lambda authorizer</strong> (instead of only a native Cognito authorizer) because it lets me:</p>

<ul>
<li>normalize claims (<code>tenantId</code>, <code>roles</code>, <code>plan</code>)</li>
<li>enforce custom auth checks</li>
<li>return a <strong><code>usageIdentifierKey</code></strong> to support per-tenant usage plans and quotas in API Gateway REST API</li>
</ul>




<h2>
  
  
  End-to-end request flow (from login to DynamoDB write)
</h2>

<p>Let’s walk through a <code>POST /projects/{projectId}/tickets</code> call.</p>

<ol>
<li>
<p><strong>User signs in via Cognito</strong></p>

<ul>
<li>Cognito issues a JWT (ID token or access token, depending your claim strategy)</li>
<li>The token contains user identity and tenant-related claims (for example <code>tenant_id</code>, <code>plan</code>, <code>roles</code>)</li>
</ul>
</li>
<li>
<p><strong>Client calls API Gateway</strong></p>

<ul>
<li>Sends <code>Authorization: Bearer &lt;JWT&gt;</code>
</li>
</ul>
</li>
<li>
<p><strong>API Gateway invokes Lambda Authorizer</strong></p>

<ul>
<li>Validates the JWT signature and issuer</li>
<li>Extracts/normalizes claims</li>
<li>Returns:

<ul>
<li>IAM policy (<code>Allow</code>)</li>
<li>
<code>context</code> (tenant ID, user ID, roles, plan)</li>
<li>
<code>usageIdentifierKey</code> (tenant-specific metering key for usage plan enforcement)</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>API Gateway invokes Lambda handler</strong></p>

<ul>
<li>The handler reads the authorizer context from <code>event.requestContext.authorizer</code>
</li>
</ul>
</li>
<li>
<p><strong>Lambda writes to DynamoDB</strong></p>

<ul>
<li>Uses tenant-prefixed keys and tenant-aware access patterns</li>
<li>Applies conditional expressions as needed</li>
<li>Writes base item + GSI attributes</li>
</ul>
</li>
<li>
<p><strong>CloudWatch metrics/logs capture telemetry</strong></p>

<ul>
<li>API Gateway request count/throttles</li>
<li>Lambda duration/errors</li>
<li>DynamoDB consumed capacity/throttles</li>
</ul>
</li>
</ol>

<p>This pattern keeps tenancy and throttling context consistent across the request path.</p>




<h2>
  
  
  Tenant isolation patterns (and which one I am using)
</h2>

<p>Before writing a single key schema, I decide what kind of tenancy model I need.</p>

<h3>
  
  
  1) Silo model (strongest isolation)
</h3>

<p>Each tenant gets isolated infrastructure (separate table / account / stack).</p>

<p><strong>Pros</strong></p>

<ul>
<li>Strong isolation</li>
<li>Easier compliance stories for some customers</li>
<li>No noisy neighbor at data layer</li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li>Higher operational overhead</li>
<li>Harder fleet-wide schema changes</li>
<li>More expensive for small tenants</li>
</ul>

<h3>
  
  
  2) Pool model (shared infrastructure, tenant-aware app logic)
</h3>

<p>All tenants share the same infrastructure (same API, same Lambda, same table), but every request and every item is tenant-scoped.</p>

<p><strong>Pros</strong></p>

<ul>
<li>Cost-efficient</li>
<li>Operationally simpler for most SaaS products</li>
<li>Scales well if keys are designed correctly</li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li>You must be disciplined about isolation in code and data model</li>
<li>Hot partitions/noisy neighbors need mitigation</li>
</ul>

<h3>
  
  
  3) Bridge model (hybrid)
</h3>

<p>Some tenants are pooled, some are siloed (for enterprise/compliance tiers).</p>

<p><strong>Pros</strong></p>

<ul>
<li>Flexible business model</li>
<li>Lets you upsell isolation</li>
</ul>

<p><strong>Cons</strong></p>

<ul>
<li>More deployment and routing complexity</li>
</ul>

<h3>
  
  
  What I am using here
</h3>

<p>For this post, I am using the <strong>pool model</strong> with strong tenant-aware design:</p>

<ul>
<li>Tenant ID is always derived from auth context</li>
<li>DynamoDB keys are tenant-prefixed</li>
<li>Access patterns are tenant-scoped</li>
<li>API Gateway usage plans enforce per-tenant limits</li>
<li>Lambda code refuses cross-tenant access even if a user manipulates IDs</li>
</ul>

<p>This is a very practical pattern for SaaS teams that want speed and scale without overbuilding day one.</p>




<h2>
  
  
  Single-table design mindset for multi-tenant SaaS
</h2>

<p>A lot of DynamoDB pain comes from designing tables like relational schemas.</p>

<p>I do the opposite: I start with <strong>access patterns</strong>, then design keys around them.</p>

<h3>
  
  
  Access patterns I need
</h3>

<p>For this SaaS, I care about these reads/writes:</p>

<ol>
<li>Create a ticket in a project</li>
<li>List tickets by project (optionally filtered by status)</li>
<li>List tickets assigned to the current user</li>
<li>Get a ticket by ID</li>
<li>Add/list comments for a ticket</li>
<li>List projects for a tenant</li>
<li>Look up user membership / role in a tenant</li>
</ol>

<p>These access patterns will drive:</p>

<ul>
<li>primary key design (<code>PK</code>, <code>SK</code>)</li>
<li>GSI design</li>
<li>where I shard to avoid hot partitions</li>
</ul>




<h2>
  
  
  Partition key design (tenant-aware and access-pattern-first)
</h2>

<p>This is the part that usually makes or breaks multi-tenant DynamoDB.</p>

<h3>
  
  
  A common mistake
</h3>

<p>A very common first attempt is:</p>

<ul>
<li><code>PK = TENANT#&lt;tenantId&gt;</code></li>
<li><code>SK = &lt;everything else&gt;</code></li>
</ul>

<p>That can work for small tenants, but it becomes risky when a tenant is large or very active because <strong>all writes for that tenant</strong> concentrate on a small number of partitions.</p>

<h3>
  
  
  My design approach
</h3>

<p>I still make keys tenant-aware, but I <strong>distribute writes across tenant sub-collections</strong>.</p>

<p>I use patterns like:</p>

<ul>
<li>
<code>PK = TENANT#&lt;tenantId&gt;</code> for tenant metadata and low-volume tenant-level items</li>
<li>
<code>PK = TENANT#&lt;tenantId&gt;#PROJECT#&lt;projectId&gt;</code> for project and ticket collections</li>
<li>
<code>PK = TENANT#&lt;tenantId&gt;#TICKET#&lt;ticketId&gt;</code> for ticket root + comments (if comment-heavy)</li>
<li>Sharded GSIs (or write-sharded partitions) for high-volume list patterns</li>
</ul>

<p>This gives me tenant isolation without forcing every high-volume write into one tenant partition.</p>




<h2>
  
  
  Example item model (single table)
</h2>

<p>I usually keep a few shared attributes on every item:</p>

<ul>
<li>
<code>PK</code>, <code>SK</code>
</li>
<li><code>entityType</code></li>
<li><code>tenantId</code></li>
<li>timestamps (<code>createdAt</code>, <code>updatedAt</code>)</li>
<li>optional GSI attributes (<code>GSI1PK</code>, <code>GSI1SK</code>, <code>GSI2PK</code>, <code>GSI2SK</code>)</li>
</ul>

<h3>
  
  
  Example items
</h3>

<h4>
  
  
  Tenant metadata
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"PK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT#t_123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"SK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"META"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"entityType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"tenantId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"t_123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Acme Corp"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"plan"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pro"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"createdAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T00:00:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  Project
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"PK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT#t_123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"SK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PROJECT#p_001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"entityType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PROJECT"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"tenantId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"t_123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"projectId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"p_001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Platform"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"createdAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T00:00:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  Ticket (project-scoped partition)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"PK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT#t_123#PROJECT#p_001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"SK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TICKET#tk_9001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"entityType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TICKET"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"tenantId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"t_123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"projectId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"p_001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"ticketId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tk_9001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"API returns 500 on retry"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"OPEN"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"priority"</span><span class="p">:</span><span class="w"> </span><span class="s2">"HIGH"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"assigneeUserId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"u_77"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"createdByUserId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"u_12"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"createdAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T01:00:00Z"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"updatedAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T01:00:00Z"</span><span class="p">,</span><span class="w">

  </span><span class="nl">"GSI1PK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT#t_123#PROJECT#p_001#STATUS#OPEN"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"GSI1SK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"UPDATED#2026-02-25T01:00:00Z#TICKET#tk_9001"</span><span class="p">,</span><span class="w">

  </span><span class="nl">"GSI2PK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT#t_123#ASSIGNEE#u_77"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"GSI2SK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"UPDATED#2026-02-25T01:00:00Z#TICKET#tk_9001"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h4>
  
  
  Comment (ticket-scoped partition)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"PK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"TENANT#t_123#TICKET#tk_9001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"SK"</span><span class="p">:</span><span class="w"> </span><span class="s2">"COMMENT#2026-02-25T01:05:00Z#c_001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"entityType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"COMMENT"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"tenantId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"t_123"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"ticketId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tk_9001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"commentId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"c_001"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"authorUserId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"u_12"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"body"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Investigating now"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"createdAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T01:05:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Why this layout works
</h3>

<ul>
<li>Tenant is embedded in every key path</li>
<li>Tickets are grouped by project for efficient project views</li>
<li>Comments are grouped by ticket for efficient comment threads</li>
<li>User-assignee lookup is handled by GSI</li>
<li>Status-based listing is handled by GSI</li>
</ul>

<p>This is a good default shape for many SaaS workloads.</p>




<h2>
  
  
  GSIs for access patterns (and how I decide them)
</h2>

<p>I try to make every GSI exist for a <strong>specific query</strong>, not “just in case”.</p>

<h3>
  
  
  GSI1: List tickets by project + status
</h3>

<p><strong>Use case:</strong> <code>GET /projects/{projectId}/tickets?status=OPEN</code></p>

<ul>
<li><code>GSI1PK = TENANT#&lt;tenantId&gt;#PROJECT#&lt;projectId&gt;#STATUS#&lt;status&gt;</code></li>
<li><code>GSI1SK = UPDATED#&lt;timestamp&gt;#TICKET#&lt;ticketId&gt;</code></li>
</ul>

<p>This lets me list tickets by status and sort by most recent updates.</p>

<h3>
  
  
  GSI2: List tickets assigned to a user
</h3>

<p><strong>Use case:</strong> <code>GET /me/tickets</code></p>

<ul>
<li><code>GSI2PK = TENANT#&lt;tenantId&gt;#ASSIGNEE#&lt;userId&gt;</code></li>
<li><code>GSI2SK = UPDATED#&lt;timestamp&gt;#TICKET#&lt;ticketId&gt;</code></li>
</ul>

<p>This gives me a tenant-safe “my work” view.</p>

<h3>
  
  
  Optional GSI3: Lookup by public ticket reference (if needed)
</h3>

<p>If you expose a user-friendly ticket reference like <code>ACME-194</code>, I may add a sparse GSI:</p>

<ul>
<li><code>GSI3PK = TENANT#&lt;tenantId&gt;#REF#ACME-194</code></li>
<li><code>GSI3SK = TICKET#&lt;ticketId&gt;</code></li>
</ul>

<p>I do not add this unless I truly need it.</p>




<h2>
  
  
  Hot partition mitigation (the part many examples skip)
</h2>

<p>A single-table design can be “correct” and still fail under traffic if hot partitions are ignored.</p>

<p>Here are the patterns I use.</p>

<h3>
  
  
  1) Avoid concentrating all writes into one tenant root partition
</h3>

<p>If all writes use <code>PK = TENANT#&lt;tenantId&gt;</code>, a large tenant can get hot fast.</p>

<p><strong>Mitigation:</strong> partition by natural sub-aggregates (project, ticket thread, etc.)</p>

<h3>
  
  
  2) Use write sharding for high-volume list/index patterns
</h3>

<p>Suppose one tenant has a huge number of ticket updates, and your status GSI gets hot.</p>

<p>I can shard the GSI partition key:</p>

<ul>
<li><code>GSI1PK = TENANT#t_123#PROJECT#p_001#STATUS#OPEN#SHARD#03</code></li>
</ul>

<p>Then query multiple shards and merge results in Lambda.</p>

<p>How do I pick the shard?</p>

<ul>
<li>Hash <code>ticketId</code> or <code>(ticketId + status)</code> into <code>N</code> buckets</li>
<li>Start small (for example 4 or 8 shards)</li>
<li>Increase if traffic grows</li>
</ul>

<p>Example helper:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">shardFor</span><span class="p">(</span><span class="nx">value</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">shardCount</span> <span class="o">=</span> <span class="mi">8</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="nx">hash</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">value</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">hash</span> <span class="o">=</span> <span class="p">(</span><span class="nx">hash</span> <span class="o">*</span> <span class="mi">31</span> <span class="o">+</span> <span class="nx">value</span><span class="p">.</span><span class="nf">charCodeAt</span><span class="p">(</span><span class="nx">i</span><span class="p">))</span> <span class="o">&gt;&gt;&gt;</span> <span class="mi">0</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">shard</span> <span class="o">=</span> <span class="nx">hash</span> <span class="o">%</span> <span class="nx">shardCount</span><span class="p">;</span>
  <span class="k">return</span> <span class="nx">shard</span><span class="p">.</span><span class="nf">toString</span><span class="p">().</span><span class="nf">padStart</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="dl">"</span><span class="s2">0</span><span class="dl">"</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  3) Separate write-heavy entities from read-heavy aggregations
</h3>

<p>If comments are very high volume, I often avoid mixing them in the same partition as ticket metadata.</p>

<p>That is why in the example above I used:</p>

<ul>
<li>ticket record under <code>TENANT#...#PROJECT#...</code>
</li>
<li>comments under <code>TENANT#...#TICKET#...</code>
</li>
</ul>

<h3>
  
  
  4) Use on-demand capacity early, then tune if needed
</h3>

<p>For newer SaaS products, I often start with <strong>on-demand</strong> capacity because it reduces operational tuning.</p>

<p>As traffic patterns stabilize, I decide whether provisioned + autoscaling is worth it.</p>

<h3>
  
  
  5) Watch metrics by access pattern, not just table-wide
</h3>

<p>I monitor:</p>

<ul>
<li>Throttled requests</li>
<li>Consumed read/write capacity</li>
<li>Latency by endpoint</li>
<li>GSI-specific pressure</li>
</ul>

<p>If one endpoint is slow, I inspect whether the underlying key pattern is too concentrated.</p>




<h2>
  
  
  Auth context propagation (Cognito/JWT -&gt; API Gateway -&gt; Lambda)
</h2>

<p>This is the glue that keeps tenant isolation consistent.</p>

<h3>
  
  
  What I want from auth context
</h3>

<p>By the time my business Lambda runs, I want a normalized auth context like this:</p>

<ul>
<li><code>tenantId</code></li>
<li><code>userId</code></li>
<li><code>roles</code></li>
<li><code>plan</code></li>
<li>(optional) <code>scopes</code>, <code>email</code>, <code>orgId</code>, <code>isSupportUser</code>
</li>
</ul>

<p>I do not want each Lambda handler parsing arbitrary JWT claims differently.</p>

<h3>
  
  
  Cognito claim strategy
</h3>

<p>You have a few options:</p>

<ul>
<li>Use Cognito <strong>ID token</strong> claims directly (simpler if tenant custom attributes are on the user profile)</li>
<li>Use <strong>Pre Token Generation</strong> trigger to inject normalized tenant claims into the token you use for APIs</li>
<li>Use groups + custom claims for role/plan mapping</li>
</ul>

<p>For this post, I will assume the token contains:</p>

<ul>
<li><code>sub</code></li>
<li>
<code>custom:tenant_id</code> (or <code>tenant_id</code>)</li>
<li>
<code>custom:plan</code> (or <code>plan</code>)</li>
<li><code>cognito:groups</code></li>
</ul>




<h2>
  
  
  Lambda Authorizer example (TypeScript)
</h2>

<p>This authorizer validates the Cognito JWT, extracts tenant context, and returns:</p>

<ul>
<li>an Allow policy</li>
<li>authorizer context</li>
<li>
<code>usageIdentifierKey</code> (for per-tenant API Gateway usage plan metering)</li>
</ul>

<blockquote>
<p>Note: In API Gateway REST APIs, <code>usageIdentifierKey</code> should be the <strong>API key value</strong> you want metered. In production, I usually map <code>tenantId -&gt; meteringKey</code> and return the metering key, not the raw tenant ID.<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// authorizer.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span>
  <span class="nx">APIGatewayTokenAuthorizerEvent</span><span class="p">,</span>
  <span class="nx">APIGatewayAuthorizerResult</span><span class="p">,</span>
<span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-lambda</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">CognitoJwtVerifier</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-jwt-verify</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">userPoolId</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">USER_POOL_ID</span><span class="o">!</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">clientId</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">APP_CLIENT_ID</span><span class="o">!</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">verifier</span> <span class="o">=</span> <span class="nx">CognitoJwtVerifier</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
  <span class="nx">userPoolId</span><span class="p">,</span>
  <span class="na">tokenUse</span><span class="p">:</span> <span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">,</span> <span class="c1">// or "access" if your claims are injected there</span>
  <span class="nx">clientId</span><span class="p">,</span>
<span class="p">});</span>

<span class="kd">function</span> <span class="nf">buildPolicy</span><span class="p">(</span><span class="nx">principalId</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">effect</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Allow</span><span class="dl">"</span> <span class="o">|</span> <span class="dl">"</span><span class="s2">Deny</span><span class="dl">"</span><span class="p">,</span> <span class="nx">resource</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="nx">principalId</span><span class="p">,</span>
    <span class="na">policyDocument</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">Version</span><span class="p">:</span> <span class="dl">"</span><span class="s2">2012-10-17</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">Statement</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">Action</span><span class="p">:</span> <span class="dl">"</span><span class="s2">execute-api:Invoke</span><span class="dl">"</span><span class="p">,</span>
          <span class="na">Effect</span><span class="p">:</span> <span class="nx">effect</span><span class="p">,</span>
          <span class="na">Resource</span><span class="p">:</span> <span class="nx">resource</span><span class="p">,</span>
        <span class="p">},</span>
      <span class="p">],</span>
    <span class="p">},</span>
  <span class="p">};</span>
<span class="p">}</span>

<span class="c1">// Replace with a real lookup (DynamoDB / Secrets Manager / config service)</span>
<span class="k">async</span> <span class="kd">function</span> <span class="nf">lookupMeteringKey</span><span class="p">(</span><span class="nx">tenantId</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="c1">// Example only. In production, return the API Gateway API key *value* assigned to that tenant.</span>
  <span class="k">return</span> <span class="s2">`meter-</span><span class="p">${</span><span class="nx">tenantId</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span>
  <span class="nx">event</span><span class="p">:</span> <span class="nx">APIGatewayTokenAuthorizerEvent</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">APIGatewayAuthorizerResult</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">raw</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">authorizationToken</span> <span class="o">||</span> <span class="dl">""</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">token</span> <span class="o">=</span> <span class="nx">raw</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/^Bearer</span><span class="se">\s</span><span class="sr">+/i</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">claims</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">verifier</span><span class="p">.</span><span class="nf">verify</span><span class="p">(</span><span class="nx">token</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">tenantId</span> <span class="o">=</span>
      <span class="p">(</span><span class="nx">claims</span><span class="p">[</span><span class="dl">"</span><span class="s2">custom:tenant_id</span><span class="dl">"</span><span class="p">]</span> <span class="k">as</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">)</span> <span class="o">??</span>
      <span class="p">(</span><span class="nx">claims</span><span class="p">[</span><span class="dl">"</span><span class="s2">tenant_id</span><span class="dl">"</span><span class="p">]</span> <span class="k">as</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">tenantId</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Missing tenant claim</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="kd">const</span> <span class="nx">userId</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">claims</span><span class="p">.</span><span class="nx">sub</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">plan</span> <span class="o">=</span>
      <span class="nc">String</span><span class="p">(</span>
        <span class="p">(</span><span class="nx">claims</span><span class="p">[</span><span class="dl">"</span><span class="s2">custom:plan</span><span class="dl">"</span><span class="p">]</span> <span class="k">as</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">)</span> <span class="o">??</span>
        <span class="p">(</span><span class="nx">claims</span><span class="p">[</span><span class="dl">"</span><span class="s2">plan</span><span class="dl">"</span><span class="p">]</span> <span class="k">as</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">)</span> <span class="o">??</span>
        <span class="dl">"</span><span class="s2">free</span><span class="dl">"</span>
      <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">groups</span> <span class="o">=</span> <span class="p">(</span><span class="nx">claims</span><span class="p">[</span><span class="dl">"</span><span class="s2">cognito:groups</span><span class="dl">"</span><span class="p">]</span> <span class="k">as</span> <span class="kr">string</span><span class="p">[]</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">)</span> <span class="o">??</span> <span class="p">[];</span>
    <span class="kd">const</span> <span class="nx">meteringKey</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">lookupMeteringKey</span><span class="p">(</span><span class="nx">tenantId</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">policy</span> <span class="o">=</span> <span class="nf">buildPolicy</span><span class="p">(</span><span class="nx">userId</span><span class="p">,</span> <span class="dl">"</span><span class="s2">Allow</span><span class="dl">"</span><span class="p">,</span> <span class="nx">event</span><span class="p">.</span><span class="nx">methodArn</span><span class="p">);</span>

    <span class="k">return</span> <span class="p">{</span>
      <span class="p">...</span><span class="nx">policy</span><span class="p">,</span>
      <span class="na">context</span><span class="p">:</span> <span class="p">{</span>
        <span class="nx">tenantId</span><span class="p">,</span>
        <span class="nx">userId</span><span class="p">,</span>
        <span class="nx">plan</span><span class="p">,</span>
        <span class="na">rolesJson</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">groups</span><span class="p">),</span>
      <span class="p">},</span>
      <span class="na">usageIdentifierKey</span><span class="p">:</span> <span class="nx">meteringKey</span><span class="p">,</span>
    <span class="p">};</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// API Gateway treats "Unauthorized" specially</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Unauthorized</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Why I normalize here instead of in every Lambda
</h3>

<p>This avoids duplicated auth parsing and reduces inconsistent authorization logic across handlers.</p>

<p>It also gives me one place to:</p>

<ul>
<li>map plans</li>
<li>map roles</li>
<li>support internal support-user impersonation rules (if needed)</li>
<li>connect metering/throttling to tenancy</li>
</ul>




<h2>
  
  
  Business Lambda: reading auth context and enforcing tenancy
</h2>

<p>Now let’s create a ticket.</p>

<p>This handler:</p>

<ul>
<li>reads tenant context from the authorizer</li>
<li>ignores any tenant ID in the request body</li>
<li>writes a ticket item to DynamoDB using a tenant-prefixed key</li>
<li>populates GSIs for list access patterns
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// create-ticket.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">APIGatewayProxyEvent</span><span class="p">,</span> <span class="nx">APIGatewayProxyResult</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-lambda</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">DynamoDBClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/client-dynamodb</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">DynamoDBDocumentClient</span><span class="p">,</span> <span class="nx">PutCommand</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/lib-dynamodb</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">randomUUID</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">crypto</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">ddb</span> <span class="o">=</span> <span class="nx">DynamoDBDocumentClient</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="k">new</span> <span class="nc">DynamoDBClient</span><span class="p">({}));</span>
<span class="kd">const</span> <span class="nx">TABLE_NAME</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TABLE_NAME</span><span class="o">!</span><span class="p">;</span>

<span class="kd">type</span> <span class="nx">AuthorizerCtx</span> <span class="o">=</span> <span class="p">{</span>
  <span class="na">tenantId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">userId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">plan</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">rolesJson</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">};</span>

<span class="kd">function</span> <span class="nf">response</span><span class="p">(</span><span class="nx">statusCode</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">body</span><span class="p">:</span> <span class="nx">unknown</span><span class="p">):</span> <span class="nx">APIGatewayProxyResult</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="nx">statusCode</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span> <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">body</span><span class="p">),</span>
  <span class="p">};</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">getAuthContext</span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">APIGatewayProxyEvent</span><span class="p">):</span> <span class="nx">AuthorizerCtx</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">auth</span> <span class="o">=</span> <span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">requestContext</span><span class="p">.</span><span class="nx">authorizer</span> <span class="o">??</span> <span class="p">{})</span> <span class="k">as</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">unknown</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">tenantId</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">auth</span><span class="p">.</span><span class="nx">tenantId</span> <span class="o">??</span> <span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">userId</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">auth</span><span class="p">.</span><span class="nx">userId</span> <span class="o">??</span> <span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">plan</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">auth</span><span class="p">.</span><span class="nx">plan</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">free</span><span class="dl">"</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">rolesJson</span> <span class="o">=</span> <span class="nx">auth</span><span class="p">.</span><span class="nx">rolesJson</span> <span class="p">?</span> <span class="nc">String</span><span class="p">(</span><span class="nx">auth</span><span class="p">.</span><span class="nx">rolesJson</span><span class="p">)</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">[]</span><span class="dl">"</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">tenantId</span> <span class="o">||</span> <span class="o">!</span><span class="nx">userId</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Missing auth context</span><span class="dl">"</span><span class="p">);</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">tenantId</span><span class="p">,</span> <span class="nx">userId</span><span class="p">,</span> <span class="nx">plan</span><span class="p">,</span> <span class="nx">rolesJson</span> <span class="p">};</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">shardFor</span><span class="p">(</span><span class="nx">value</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">shardCount</span> <span class="o">=</span> <span class="mi">8</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="nx">hash</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
  <span class="k">for </span><span class="p">(</span><span class="kd">let</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o">&lt;</span> <span class="nx">value</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">hash</span> <span class="o">=</span> <span class="p">(</span><span class="nx">hash</span> <span class="o">*</span> <span class="mi">31</span> <span class="o">+</span> <span class="nx">value</span><span class="p">.</span><span class="nf">charCodeAt</span><span class="p">(</span><span class="nx">i</span><span class="p">))</span> <span class="o">&gt;&gt;&gt;</span> <span class="mi">0</span><span class="p">;</span>
  <span class="p">}</span>
  <span class="k">return</span> <span class="nc">String</span><span class="p">(</span><span class="nx">hash</span> <span class="o">%</span> <span class="nx">shardCount</span><span class="p">).</span><span class="nf">padStart</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="dl">"</span><span class="s2">0</span><span class="dl">"</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">APIGatewayProxyEvent</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">APIGatewayProxyResult</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="nx">tenantId</span><span class="p">,</span> <span class="nx">userId</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">getAuthContext</span><span class="p">(</span><span class="nx">event</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">projectId</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">pathParameters</span><span class="p">?.</span><span class="nx">projectId</span><span class="p">;</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">projectId</span><span class="p">)</span> <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">400</span><span class="p">,</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">projectId is required</span><span class="dl">"</span> <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">body</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">body</span> <span class="p">?</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">body</span><span class="p">)</span> <span class="p">:</span> <span class="p">{};</span>
    <span class="kd">const</span> <span class="nx">title</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">body</span><span class="p">.</span><span class="nx">title</span> <span class="o">??</span> <span class="dl">""</span><span class="p">).</span><span class="nf">trim</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">assigneeUserId</span> <span class="o">=</span> <span class="nx">body</span><span class="p">.</span><span class="nx">assigneeUserId</span> <span class="p">?</span> <span class="nc">String</span><span class="p">(</span><span class="nx">body</span><span class="p">.</span><span class="nx">assigneeUserId</span><span class="p">)</span> <span class="p">:</span> <span class="kc">undefined</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">priority</span> <span class="o">=</span> <span class="nx">body</span><span class="p">.</span><span class="nx">priority</span> <span class="p">?</span> <span class="nc">String</span><span class="p">(</span><span class="nx">body</span><span class="p">.</span><span class="nx">priority</span><span class="p">)</span> <span class="p">:</span> <span class="dl">"</span><span class="s2">MEDIUM</span><span class="dl">"</span><span class="p">;</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">title</span><span class="p">)</span> <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">400</span><span class="p">,</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">title is required</span><span class="dl">"</span> <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">ticketId</span> <span class="o">=</span> <span class="s2">`tk_</span><span class="p">${</span><span class="nf">randomUUID</span><span class="p">()}</span><span class="s2">`</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">now</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">();</span>
    <span class="kd">const</span> <span class="nx">status</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">OPEN</span><span class="dl">"</span><span class="p">;</span>

    <span class="c1">// Optional GSI sharding for high-volume status lists</span>
    <span class="kd">const</span> <span class="nx">statusShard</span> <span class="o">=</span> <span class="nf">shardFor</span><span class="p">(</span><span class="nx">ticketId</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">item</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">PK</span><span class="p">:</span> <span class="s2">`TENANT#</span><span class="p">${</span><span class="nx">tenantId</span><span class="p">}</span><span class="s2">#PROJECT#</span><span class="p">${</span><span class="nx">projectId</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="na">SK</span><span class="p">:</span> <span class="s2">`TICKET#</span><span class="p">${</span><span class="nx">ticketId</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="na">entityType</span><span class="p">:</span> <span class="dl">"</span><span class="s2">TICKET</span><span class="dl">"</span><span class="p">,</span>

      <span class="nx">tenantId</span><span class="p">,</span>
      <span class="nx">projectId</span><span class="p">,</span>
      <span class="nx">ticketId</span><span class="p">,</span>
      <span class="nx">title</span><span class="p">,</span>
      <span class="nx">status</span><span class="p">,</span>
      <span class="nx">priority</span><span class="p">,</span>
      <span class="nx">assigneeUserId</span><span class="p">,</span>
      <span class="na">createdByUserId</span><span class="p">:</span> <span class="nx">userId</span><span class="p">,</span>
      <span class="na">createdAt</span><span class="p">:</span> <span class="nx">now</span><span class="p">,</span>
      <span class="na">updatedAt</span><span class="p">:</span> <span class="nx">now</span><span class="p">,</span>

      <span class="c1">// List tickets by project + status</span>
      <span class="na">GSI1PK</span><span class="p">:</span> <span class="s2">`TENANT#</span><span class="p">${</span><span class="nx">tenantId</span><span class="p">}</span><span class="s2">#PROJECT#</span><span class="p">${</span><span class="nx">projectId</span><span class="p">}</span><span class="s2">#STATUS#</span><span class="p">${</span><span class="nx">status</span><span class="p">}</span><span class="s2">#SHARD#</span><span class="p">${</span><span class="nx">statusShard</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="na">GSI1SK</span><span class="p">:</span> <span class="s2">`UPDATED#</span><span class="p">${</span><span class="nx">now</span><span class="p">}</span><span class="s2">#TICKET#</span><span class="p">${</span><span class="nx">ticketId</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>

      <span class="c1">// List tickets by assignee</span>
      <span class="p">...(</span><span class="nx">assigneeUserId</span> <span class="o">&amp;&amp;</span> <span class="p">{</span>
        <span class="na">GSI2PK</span><span class="p">:</span> <span class="s2">`TENANT#</span><span class="p">${</span><span class="nx">tenantId</span><span class="p">}</span><span class="s2">#ASSIGNEE#</span><span class="p">${</span><span class="nx">assigneeUserId</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
        <span class="na">GSI2SK</span><span class="p">:</span> <span class="s2">`UPDATED#</span><span class="p">${</span><span class="nx">now</span><span class="p">}</span><span class="s2">#TICKET#</span><span class="p">${</span><span class="nx">ticketId</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="p">}),</span>
    <span class="p">};</span>

    <span class="k">await</span> <span class="nx">ddb</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span>
      <span class="k">new</span> <span class="nc">PutCommand</span><span class="p">({</span>
        <span class="na">TableName</span><span class="p">:</span> <span class="nx">TABLE_NAME</span><span class="p">,</span>
        <span class="na">Item</span><span class="p">:</span> <span class="nx">item</span><span class="p">,</span>
        <span class="na">ConditionExpression</span><span class="p">:</span> <span class="dl">"</span><span class="s2">attribute_not_exists(PK) AND attribute_not_exists(SK)</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">})</span>
    <span class="p">);</span>

    <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">201</span><span class="p">,</span> <span class="p">{</span>
      <span class="nx">ticketId</span><span class="p">,</span>
      <span class="nx">status</span><span class="p">,</span>
      <span class="na">createdAt</span><span class="p">:</span> <span class="nx">now</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">create-ticket failed</span><span class="dl">"</span><span class="p">,</span> <span class="nx">err</span><span class="p">);</span>
    <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">500</span><span class="p">,</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Internal server error</span><span class="dl">"</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Isolation note
</h3>

<p>The handler never accepts <code>tenantId</code> from the request body for authorization purposes.</p>

<p>Even if a client sends a different tenant ID, it does not matter because the handler uses <code>tenantId</code> from the validated authorizer context.</p>




<h2>
  
  
  Querying with GSIs (list tickets by project + status)
</h2>

<p>Now let’s implement <code>GET /projects/{projectId}/tickets?status=OPEN</code>.</p>

<p>Because I sharded <code>GSI1PK</code>, I may need to query multiple shards and merge the results.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// list-project-tickets.ts</span>
<span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">APIGatewayProxyEvent</span><span class="p">,</span> <span class="nx">APIGatewayProxyResult</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-lambda</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">DynamoDBClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/client-dynamodb</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">DynamoDBDocumentClient</span><span class="p">,</span> <span class="nx">QueryCommand</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@aws-sdk/lib-dynamodb</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">ddb</span> <span class="o">=</span> <span class="nx">DynamoDBDocumentClient</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="k">new</span> <span class="nc">DynamoDBClient</span><span class="p">({}));</span>
<span class="kd">const</span> <span class="nx">TABLE_NAME</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">TABLE_NAME</span><span class="o">!</span><span class="p">;</span>

<span class="kd">function</span> <span class="nf">response</span><span class="p">(</span><span class="nx">statusCode</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">body</span><span class="p">:</span> <span class="nx">unknown</span><span class="p">):</span> <span class="nx">APIGatewayProxyResult</span> <span class="p">{</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="nx">statusCode</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">"</span><span class="s2">Content-Type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span> <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">(</span><span class="nx">body</span><span class="p">),</span>
  <span class="p">};</span>
<span class="p">}</span>

<span class="kd">function</span> <span class="nf">getTenantId</span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">APIGatewayProxyEvent</span><span class="p">):</span> <span class="kr">string</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">auth</span> <span class="o">=</span> <span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">requestContext</span><span class="p">.</span><span class="nx">authorizer</span> <span class="o">??</span> <span class="p">{})</span> <span class="k">as</span> <span class="nb">Record</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">unknown</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">tenantId</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">auth</span><span class="p">.</span><span class="nx">tenantId</span> <span class="o">??</span> <span class="dl">""</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">tenantId</span><span class="p">)</span> <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Missing tenantId</span><span class="dl">"</span><span class="p">);</span>
  <span class="k">return</span> <span class="nx">tenantId</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">handler</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">event</span><span class="p">:</span> <span class="nx">APIGatewayProxyEvent</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">APIGatewayProxyResult</span><span class="o">&gt;</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">tenantId</span> <span class="o">=</span> <span class="nf">getTenantId</span><span class="p">(</span><span class="nx">event</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">projectId</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">pathParameters</span><span class="p">?.</span><span class="nx">projectId</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">status</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">event</span><span class="p">.</span><span class="nx">queryStringParameters</span><span class="p">?.</span><span class="nx">status</span> <span class="o">??</span> <span class="dl">"</span><span class="s2">OPEN</span><span class="dl">"</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">projectId</span><span class="p">)</span> <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">400</span><span class="p">,</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">projectId is required</span><span class="dl">"</span> <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">shardCount</span> <span class="o">=</span> <span class="mi">8</span><span class="p">;</span>
    <span class="kd">const</span> <span class="nx">queries</span> <span class="o">=</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">({</span> <span class="na">length</span><span class="p">:</span> <span class="nx">shardCount</span> <span class="p">},</span> <span class="p">(</span><span class="nx">_</span><span class="p">,</span> <span class="nx">n</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">shard</span> <span class="o">=</span> <span class="nc">String</span><span class="p">(</span><span class="nx">n</span><span class="p">).</span><span class="nf">padStart</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="dl">"</span><span class="s2">0</span><span class="dl">"</span><span class="p">);</span>
      <span class="k">return</span> <span class="nx">ddb</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span>
        <span class="k">new</span> <span class="nc">QueryCommand</span><span class="p">({</span>
          <span class="na">TableName</span><span class="p">:</span> <span class="nx">TABLE_NAME</span><span class="p">,</span>
          <span class="na">IndexName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI1</span><span class="dl">"</span><span class="p">,</span>
          <span class="na">KeyConditionExpression</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI1PK = :pk</span><span class="dl">"</span><span class="p">,</span>
          <span class="na">ExpressionAttributeValues</span><span class="p">:</span> <span class="p">{</span>
            <span class="dl">"</span><span class="s2">:pk</span><span class="dl">"</span><span class="p">:</span> <span class="s2">`TENANT#</span><span class="p">${</span><span class="nx">tenantId</span><span class="p">}</span><span class="s2">#PROJECT#</span><span class="p">${</span><span class="nx">projectId</span><span class="p">}</span><span class="s2">#STATUS#</span><span class="p">${</span><span class="nx">status</span><span class="p">}</span><span class="s2">#SHARD#</span><span class="p">${</span><span class="nx">shard</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
          <span class="p">},</span>
          <span class="na">Limit</span><span class="p">:</span> <span class="mi">25</span><span class="p">,</span> <span class="c1">// tune based on your UX pagination model</span>
          <span class="na">ScanIndexForward</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="c1">// newest first</span>
        <span class="p">})</span>
      <span class="p">);</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">results</span> <span class="o">=</span> <span class="k">await</span> <span class="nb">Promise</span><span class="p">.</span><span class="nf">all</span><span class="p">(</span><span class="nx">queries</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">items</span> <span class="o">=</span> <span class="nx">results</span><span class="p">.</span><span class="nf">flatMap</span><span class="p">((</span><span class="nx">r</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nx">Items</span> <span class="o">??</span> <span class="p">[]);</span>

    <span class="c1">// Merge and sort in-memory because we queried multiple shards</span>
    <span class="nx">items</span><span class="p">.</span><span class="nf">sort</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nc">String</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">updatedAt</span><span class="p">).</span><span class="nf">localeCompare</span><span class="p">(</span><span class="nc">String</span><span class="p">(</span><span class="nx">a</span><span class="p">.</span><span class="nx">updatedAt</span><span class="p">)));</span>

    <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">200</span><span class="p">,</span> <span class="p">{</span>
      <span class="nx">items</span><span class="p">,</span>
      <span class="na">count</span><span class="p">:</span> <span class="nx">items</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">err</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">list-project-tickets failed</span><span class="dl">"</span><span class="p">,</span> <span class="nx">err</span><span class="p">);</span>
    <span class="k">return</span> <span class="nf">response</span><span class="p">(</span><span class="mi">500</span><span class="p">,</span> <span class="p">{</span> <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Internal server error</span><span class="dl">"</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Tradeoff discussion
</h3>

<p>Sharding improves write scalability, but read logic is more complex because I need fan-out queries.</p>

<p>That is a normal DynamoDB tradeoff. I only add this complexity for access patterns that actually need it.</p>




<h2>
  
  
  Per-tenant throttling and quotas (API Gateway + authorizer metering)
</h2>

<p>This is one of the most useful SaaS controls and one of the least discussed implementation details.</p>

<h3>
  
  
  What I want
</h3>

<p>I want to enforce limits like:</p>

<ul>
<li>Free tier tenant: lower RPS and daily quota</li>
<li>Pro tier tenant: higher RPS and quota</li>
<li>Enterprise tenant: custom limits</li>
</ul>

<h3>
  
  
  Why API Gateway usage plans help
</h3>

<p>API Gateway REST APIs support:</p>

<ul>
<li>throttling</li>
<li>quotas</li>
<li>API keys</li>
<li>usage plans</li>
</ul>

<p>With a Lambda authorizer, I can return a <strong><code>usageIdentifierKey</code></strong> so API Gateway meters/throttles by a tenant-specific key, even when the client authenticates with JWT.</p>

<h3>
  
  
  Important implementation note
</h3>

<p>For REST APIs, set <code>apiKeySourceType</code> to <code>AUTHORIZER</code> so the usage key comes from the authorizer rather than an <code>x-api-key</code> header.</p>

<h3>
  
  
  CDK sketch (TypeScript)
</h3>

<p>Below is a simplified CDK sketch to show the pattern. You will need to adapt it to your stack structure.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">cdk</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Construct</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">constructs</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">apigw</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-apigateway</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">lambda</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-lambda</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">dynamodb</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">aws-cdk-lib/aws-dynamodb</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">MultiTenantApiStack</span> <span class="kd">extends</span> <span class="nc">cdk</span><span class="p">.</span><span class="nx">Stack</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="nx">scope</span><span class="p">:</span> <span class="nx">Construct</span><span class="p">,</span> <span class="nx">id</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">props</span><span class="p">?:</span> <span class="nx">cdk</span><span class="p">.</span><span class="nx">StackProps</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">super</span><span class="p">(</span><span class="nx">scope</span><span class="p">,</span> <span class="nx">id</span><span class="p">,</span> <span class="nx">props</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">table</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nc">Table</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">AppTable</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">partitionKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">PK</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">sortKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">SK</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">billingMode</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">BillingMode</span><span class="p">.</span><span class="nx">PAY_PER_REQUEST</span><span class="p">,</span>
      <span class="na">pointInTimeRecovery</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="na">removalPolicy</span><span class="p">:</span> <span class="nx">cdk</span><span class="p">.</span><span class="nx">RemovalPolicy</span><span class="p">.</span><span class="nx">RETAIN</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="nx">table</span><span class="p">.</span><span class="nf">addGlobalSecondaryIndex</span><span class="p">({</span>
      <span class="na">indexName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI1</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">partitionKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI1PK</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">sortKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI1SK</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">projectionType</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">ProjectionType</span><span class="p">.</span><span class="nx">ALL</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="nx">table</span><span class="p">.</span><span class="nf">addGlobalSecondaryIndex</span><span class="p">({</span>
      <span class="na">indexName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI2</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">partitionKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI2PK</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">sortKey</span><span class="p">:</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GSI2SK</span><span class="dl">"</span><span class="p">,</span> <span class="na">type</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">AttributeType</span><span class="p">.</span><span class="nx">STRING</span> <span class="p">},</span>
      <span class="na">projectionType</span><span class="p">:</span> <span class="nx">dynamodb</span><span class="p">.</span><span class="nx">ProjectionType</span><span class="p">.</span><span class="nx">ALL</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">authorizerFn</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">lambda</span><span class="p">.</span><span class="nc">Function</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">AuthorizerFn</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">runtime</span><span class="p">:</span> <span class="nx">lambda</span><span class="p">.</span><span class="nx">Runtime</span><span class="p">.</span><span class="nx">NODEJS_20_X</span><span class="p">,</span>
      <span class="na">handler</span><span class="p">:</span> <span class="dl">"</span><span class="s2">authorizer.handler</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="nx">lambda</span><span class="p">.</span><span class="nx">Code</span><span class="p">.</span><span class="nf">fromAsset</span><span class="p">(</span><span class="dl">"</span><span class="s2">dist/authorizer</span><span class="dl">"</span><span class="p">),</span>
      <span class="na">environment</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">USER_POOL_ID</span><span class="p">:</span> <span class="dl">"</span><span class="s2">your-user-pool-id</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">APP_CLIENT_ID</span><span class="p">:</span> <span class="dl">"</span><span class="s2">your-app-client-id</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">createTicketFn</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">lambda</span><span class="p">.</span><span class="nc">Function</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CreateTicketFn</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">runtime</span><span class="p">:</span> <span class="nx">lambda</span><span class="p">.</span><span class="nx">Runtime</span><span class="p">.</span><span class="nx">NODEJS_20_X</span><span class="p">,</span>
      <span class="na">handler</span><span class="p">:</span> <span class="dl">"</span><span class="s2">create-ticket.handler</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="nx">lambda</span><span class="p">.</span><span class="nx">Code</span><span class="p">.</span><span class="nf">fromAsset</span><span class="p">(</span><span class="dl">"</span><span class="s2">dist/create-ticket</span><span class="dl">"</span><span class="p">),</span>
      <span class="na">environment</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">TABLE_NAME</span><span class="p">:</span> <span class="nx">table</span><span class="p">.</span><span class="nx">tableName</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>

    <span class="nx">table</span><span class="p">.</span><span class="nf">grantReadWriteData</span><span class="p">(</span><span class="nx">createTicketFn</span><span class="p">);</span>

    <span class="kd">const</span> <span class="nx">api</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">apigw</span><span class="p">.</span><span class="nc">RestApi</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">SaaSApi</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">restApiName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">multi-tenant-saas-api</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">apiKeySourceType</span><span class="p">:</span> <span class="nx">apigw</span><span class="p">.</span><span class="nx">ApiKeySourceType</span><span class="p">.</span><span class="nx">AUTHORIZER</span><span class="p">,</span> <span class="c1">// key point for per-tenant metering</span>
      <span class="na">deployOptions</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">stageName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">prod</span><span class="dl">"</span><span class="p">,</span>
        <span class="na">metricsEnabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
        <span class="na">loggingLevel</span><span class="p">:</span> <span class="nx">apigw</span><span class="p">.</span><span class="nx">MethodLoggingLevel</span><span class="p">.</span><span class="nx">INFO</span><span class="p">,</span>
        <span class="na">dataTraceEnabled</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
        <span class="na">throttlingBurstLimit</span><span class="p">:</span> <span class="mi">2000</span><span class="p">,</span>
        <span class="na">throttlingRateLimit</span><span class="p">:</span> <span class="mi">1000</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">tokenAuthorizer</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">apigw</span><span class="p">.</span><span class="nc">TokenAuthorizer</span><span class="p">(</span><span class="k">this</span><span class="p">,</span> <span class="dl">"</span><span class="s2">JwtAuthorizer</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">handler</span><span class="p">:</span> <span class="nx">authorizerFn</span><span class="p">,</span>
      <span class="na">resultsCacheTtl</span><span class="p">:</span> <span class="nx">cdk</span><span class="p">.</span><span class="nx">Duration</span><span class="p">.</span><span class="nf">minutes</span><span class="p">(</span><span class="mi">5</span><span class="p">),</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">projects</span> <span class="o">=</span> <span class="nx">api</span><span class="p">.</span><span class="nx">root</span><span class="p">.</span><span class="nf">addResource</span><span class="p">(</span><span class="dl">"</span><span class="s2">projects</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">projectId</span> <span class="o">=</span> <span class="nx">projects</span><span class="p">.</span><span class="nf">addResource</span><span class="p">(</span><span class="dl">"</span><span class="s2">{projectId}</span><span class="dl">"</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">tickets</span> <span class="o">=</span> <span class="nx">projectId</span><span class="p">.</span><span class="nf">addResource</span><span class="p">(</span><span class="dl">"</span><span class="s2">tickets</span><span class="dl">"</span><span class="p">);</span>

    <span class="nx">tickets</span><span class="p">.</span><span class="nf">addMethod</span><span class="p">(</span>
      <span class="dl">"</span><span class="s2">POST</span><span class="dl">"</span><span class="p">,</span>
      <span class="k">new</span> <span class="nx">apigw</span><span class="p">.</span><span class="nc">LambdaIntegration</span><span class="p">(</span><span class="nx">createTicketFn</span><span class="p">),</span>
      <span class="p">{</span>
        <span class="na">authorizer</span><span class="p">:</span> <span class="nx">tokenAuthorizer</span><span class="p">,</span>
        <span class="na">authorizationType</span><span class="p">:</span> <span class="nx">apigw</span><span class="p">.</span><span class="nx">AuthorizationType</span><span class="p">.</span><span class="nx">CUSTOM</span><span class="p">,</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="c1">// Example usage plan (create multiple plans for free/pro/enterprise)</span>
    <span class="kd">const</span> <span class="nx">freePlan</span> <span class="o">=</span> <span class="nx">api</span><span class="p">.</span><span class="nf">addUsagePlan</span><span class="p">(</span><span class="dl">"</span><span class="s2">FreePlan</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">"</span><span class="s2">free-tier-plan</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">throttle</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">rateLimit</span><span class="p">:</span> <span class="mi">50</span><span class="p">,</span>
        <span class="na">burstLimit</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span>
      <span class="p">},</span>
      <span class="na">quota</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">limit</span><span class="p">:</span> <span class="mi">100000</span><span class="p">,</span>
        <span class="na">period</span><span class="p">:</span> <span class="nx">apigw</span><span class="p">.</span><span class="nx">Period</span><span class="p">.</span><span class="nx">DAY</span><span class="p">,</span>
      <span class="p">},</span>
    <span class="p">});</span>

    <span class="nx">freePlan</span><span class="p">.</span><span class="nf">addApiStage</span><span class="p">({</span>
      <span class="nx">api</span><span class="p">,</span>
      <span class="na">stage</span><span class="p">:</span> <span class="nx">api</span><span class="p">.</span><span class="nx">deploymentStage</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="c1">// API keys that represent tenant metering identities.</span>
    <span class="c1">// In production, create one per tenant and associate with the correct plan.</span>
    <span class="kd">const</span> <span class="nx">tenantApiKey</span> <span class="o">=</span> <span class="nx">api</span><span class="p">.</span><span class="nf">addApiKey</span><span class="p">(</span><span class="dl">"</span><span class="s2">TenantApiKey_t_123</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
      <span class="na">apiKeyName</span><span class="p">:</span> <span class="dl">"</span><span class="s2">tenant-t_123-metering-key</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">enabled</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
      <span class="c1">// value can be auto-generated or explicitly set</span>
    <span class="p">});</span>

    <span class="nx">freePlan</span><span class="p">.</span><span class="nf">addApiKey</span><span class="p">(</span><span class="nx">tenantApiKey</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  How I usually operationalize this
</h3>

<ul>
<li>Create usage plans by tier (<code>free</code>, <code>pro</code>, <code>enterprise</code>)</li>
<li>Create an API key per tenant (metering identity)</li>
<li>Store the key value (or a mapping) securely</li>
<li>Have the authorizer return the correct <code>usageIdentifierKey</code> for the tenant</li>
</ul>

<p>This gives me a clean SaaS throttle/quota model without making clients manage API keys directly.</p>




<h2>
  
  
  Authorization checks beyond JWT validation (critical for tenant safety)
</h2>

<p>JWT validation is not enough. I also enforce authorization in Lambda based on tenant and role.</p>

<p>Examples:</p>

<ul>
<li>User can create tickets only in projects belonging to their tenant</li>
<li>User can comment only on tickets in their tenant</li>
<li>Support/admin actions require specific roles/scopes</li>
<li>Cross-tenant resource IDs are rejected even if guessed</li>
</ul>

<h3>
  
  
  Example guard pattern
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">function</span> <span class="nf">requireRole</span><span class="p">(</span><span class="nx">rolesJson</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">undefined</span><span class="p">,</span> <span class="nx">allowed</span><span class="p">:</span> <span class="kr">string</span><span class="p">[])</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">roles</span> <span class="o">=</span> <span class="nx">rolesJson</span> <span class="p">?</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="nx">rolesJson</span><span class="p">)</span> <span class="k">as</span> <span class="kr">string</span><span class="p">[]</span> <span class="p">:</span> <span class="p">[];</span>
  <span class="kd">const</span> <span class="nx">ok</span> <span class="o">=</span> <span class="nx">roles</span><span class="p">.</span><span class="nf">some</span><span class="p">((</span><span class="nx">r</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">allowed</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">r</span><span class="p">));</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">ok</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">err</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Forbidden</span><span class="dl">"</span><span class="p">);</span>
    <span class="p">(</span><span class="nx">err</span> <span class="k">as</span> <span class="kr">any</span><span class="p">).</span><span class="nx">statusCode</span> <span class="o">=</span> <span class="mi">403</span><span class="p">;</span>
    <span class="k">throw</span> <span class="nx">err</span><span class="p">;</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>I typically build a small internal library for:</p>

<ul>
<li>auth context parsing</li>
<li>role checks</li>
<li>tenant-scoped key helpers</li>
<li>common error responses</li>
</ul>

<p>That reduces drift across Lambda functions.</p>




<h2>
  
  
  End-to-end implementation walkthrough
</h2>

<p>Here is a practical sequence I would follow to build this from scratch.</p>

<h3>
  
  
  Step 1: Define your access patterns first
</h3>

<p>Before touching CloudFormation/CDK, write down:</p>

<ul>
<li>reads</li>
<li>writes</li>
<li>sorting/filtering requirements</li>
<li>expected high-volume paths</li>
</ul>

<p>This will drive your <code>PK/SK</code> and GSI decisions.</p>

<h3>
  
  
  Step 2: Set up Cognito and define tenant claims
</h3>

<p>Create a user pool and app client. Decide how tenant context appears in JWTs.</p>

<p>I recommend normalizing these claims:</p>

<ul>
<li><code>tenant_id</code></li>
<li><code>plan</code></li>
<li>
<code>roles</code> or <code>groups</code>
</li>
</ul>

<p>If your current user profile attributes are inconsistent, fix that first. It will save a lot of pain later.</p>

<h3>
  
  
  Step 3: Build the Lambda authorizer
</h3>

<p>Implement JWT validation and claim normalization.</p>

<p>Outputs should include:</p>

<ul>
<li><code>tenantId</code></li>
<li><code>userId</code></li>
<li><code>rolesJson</code></li>
<li><code>plan</code></li>
<li>
<code>usageIdentifierKey</code> (tenant metering key)</li>
</ul>

<h3>
  
  
  Step 4: Create DynamoDB single table + GSIs
</h3>

<p>Start with only the GSIs you need for your first release.</p>

<p>A good initial setup:</p>

<ul>
<li>base table (<code>PK</code>, <code>SK</code>)</li>
<li>
<code>GSI1</code> for project+status ticket listing</li>
<li>
<code>GSI2</code> for assignee ticket listing</li>
</ul>

<h3>
  
  
  Step 5: Implement tenant-safe handlers
</h3>

<p>In every handler:</p>

<ul>
<li>parse auth context from API Gateway authorizer context</li>
<li>derive keys from auth context</li>
<li>never trust client-supplied tenant IDs for authorization</li>
<li>use conditional expressions when appropriate</li>
</ul>

<h3>
  
  
  Step 6: Add per-tenant usage plans
</h3>

<p>Create:</p>

<ul>
<li>usage plans by tier</li>
<li>API keys per tenant</li>
<li>tenant -&gt; metering key mapping</li>
</ul>

<p>Then return the metering key in the Lambda authorizer.</p>

<h3>
  
  
  Step 7: Test with real tokens and tenant boundary tests
</h3>

<p>I always run tests for:</p>

<ul>
<li>valid tenant access</li>
<li>invalid token</li>
<li>wrong tenant resource ID</li>
<li>quota exceeded / throttle behavior</li>
<li>large-tenant list/query traffic</li>
</ul>




<h2>
  
  
  Example request/response flow (local testing mindset)
</h2>

<h3>
  
  
  Create ticket request
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST <span class="se">\</span>
  <span class="s2">"https://{apiId}.execute-api.{region}.amazonaws.com/prod/projects/p_001/tickets"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer &lt;JWT&gt;"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "title": "API returns 500 on retry",
    "priority": "HIGH",
    "assigneeUserId": "u_77"
  }'</span>
</code></pre>

</div>



<h3>
  
  
  Expected response
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"ticketId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"tk_6b4d8e3e-...."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"status"</span><span class="p">:</span><span class="w"> </span><span class="s2">"OPEN"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"createdAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2026-02-25T01:00:00Z"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  List tickets by status
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="se">\</span>
  <span class="s2">"https://{apiId}.execute-api.{region}.amazonaws.com/prod/projects/p_001/tickets?status=OPEN"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer &lt;JWT&gt;"</span>
</code></pre>

</div>






<h2>
  
  
  Common mistakes I see in this architecture
</h2>

<h3>
  
  
  1) Trusting <code>tenantId</code> from the client
</h3>

<p>This is the fastest way to create cross-tenant data leaks.</p>

<p><strong>Fix:</strong> derive tenant context from validated JWT and enforce server-side.</p>

<h3>
  
  
  2) Designing the table like a relational schema
</h3>

<p>DynamoDB single-table design is access-pattern driven, not entity-table driven.</p>

<p><strong>Fix:</strong> start with queries and write paths first.</p>

<h3>
  
  
  3) Overusing one tenant root partition key
</h3>

<p>This can create hot partitions for large tenants.</p>

<p><strong>Fix:</strong> use tenant-aware sub-collections and shard high-volume patterns.</p>

<h3>
  
  
  4) Adding too many GSIs early
</h3>

<p>Every GSI adds write amplification and operational cost.</p>

<p><strong>Fix:</strong> add GSIs only for clear access patterns.</p>

<h3>
  
  
  5) Treating JWT validation as full authorization
</h3>

<p>A valid token does not mean a valid action.</p>

<p><strong>Fix:</strong> layer role checks and tenant-resource checks in Lambda.</p>

<h3>
  
  
  6) Ignoring quota/throttling strategy until later
</h3>

<p>By the time you need it, retrofitting it can be messy.</p>

<p><strong>Fix:</strong> establish a per-tenant metering pattern early (usage plans or app-level rate limiting).</p>




<h2>
  
  
  When I would extend this design
</h2>

<p>As the SaaS grows, I often add:</p>

<ul>
<li>
<strong>DynamoDB Streams</strong> for async workflows (notifications, audit, search indexing)</li>
<li>
<strong>SQS/EventBridge</strong> for background processing</li>
<li>
<strong>Fine-grained authorization model</strong> (RBAC/ABAC) beyond simple groups</li>
<li><strong>Tenant-level feature flags</strong></li>
<li>
<strong>Audit log table</strong> (append-only) for compliance</li>
<li>
<strong>Support-user safe impersonation</strong> with strict audit trails</li>
<li>
<strong>Silo/bridge routing</strong> for enterprise tenants needing dedicated infrastructure</li>
</ul>

<p>The nice part is that the core pattern (API Gateway + Lambda + DynamoDB + normalized auth context) still holds.</p>




<h2>
  
  
  Final thoughts
</h2>

<p>This stack is powerful because it lets me solve <strong>architecture and data modeling together</strong>.</p>

<p>The API layer, auth model, and DynamoDB key design are not separate decisions in a multi-tenant SaaS. They are one system:</p>

<ul>
<li>Auth defines the tenant context</li>
<li>The API propagates and enforces it</li>
<li>The data model encodes it</li>
<li>Throttling/quotas protect the platform from noisy neighbors</li>
</ul>

<p>If you get those four pieces aligned early, you can move fast without constantly revisiting foundational decisions.</p>

<p>If I were building a new SaaS MVP today on AWS, this would still be one of my first choices.</p>




<h2>
  
  
  References
</h2>

<ul>
<li>Amazon API Gateway REST API documentation</li>
<li>AWS Lambda developer guide</li>
<li>Amazon DynamoDB developer guide</li>
<li>DynamoDB single-table design patterns (AWS documentation and re:Invent talks)</li>
<li>Amazon Cognito user pools and JWT token documentation</li>
<li>API Gateway usage plans and API keys documentation</li>
<li>
<code>aws-jwt-verify</code> library documentation</li>
<li>DynamoDB best practices for partition key design and adaptive capacity</li>
<li>AWS Well-Architected Framework (Serverless Lens)</li>
</ul>

