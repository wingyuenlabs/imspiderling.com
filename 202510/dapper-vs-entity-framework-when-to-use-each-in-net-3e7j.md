---
Title: Dapper vs Entity Framework: When to Use Each in .NET
Description: 
Author: Alex Fernandes
Date: 2025-10-07T21:58:32.000Z
Robots: noindex,nofollow
Template: index
---
<p>Choosing between <strong>Dapper</strong> and <strong>Entity Framework</strong> is one of the most important architectural decisions when building data-driven applications in <strong>.NET</strong>.</p>

<p>Both are excellent ORM (Object-Relational Mapping) tools—but they serve different purposes depending on your project’s <strong>performance</strong>, <strong>complexity</strong>, and <strong>development speed</strong> requirements.</p>

<p>This complete guide explains when to use each, their pros and cons, and includes <strong>real CRUD examples</strong> in C# for both <strong>Dapper</strong> and <strong>Entity Framework</strong>.</p>

<h2>
  
  
  Dapper: Lightweight and High Performance
</h2>

<p><strong>Dapper</strong> is a <strong>micro ORM</strong> that offers direct SQL control with minimal overhead. It’s perfect when you want <strong>speed</strong>, <strong>simplicity</strong>, and <strong>full control</strong> over your database queries.</p>

<p>It’s ideal for:</p>

<ul>
<li>High-performance APIs and microservices.</li>
<li>Projects that rely heavily on <strong>custom SQL</strong>.</li>
<li>Rapid migrations from legacy SQL codebases.</li>
</ul>

<h3>
  
  
  ✅ Pros of Dapper
</h3>

<ul>
<li>
<strong>Simple and transparent:</strong> You always see the SQL being executed.</li>
<li>
<strong>Extremely fast:</strong> Often outperforms Entity Framework due to minimal abstraction.</li>
<li>
<strong>Lightweight:</strong> No complex setup or heavy dependencies.</li>
<li>
<strong>Perfect for SQL experts:</strong> Gives you maximum query flexibility.</li>
</ul>

<h3>
  
  
  ❌ Cons of Dapper
</h3>

<ul>
<li>
<strong>Manual query management:</strong> Large or complex SQL statements can become hard to maintain.</li>
<li>
<strong>Lacks advanced ORM features:</strong> No change tracking, migrations, or LINQ integration.</li>
</ul>

<h2>
  
  
  💻 Example: Dapper CRUD Repository in .NET
</h2>

<p>Below is a simple <strong>DapperUserRepository</strong> using <strong>SQLite</strong>.<br>
It demonstrates how to implement <code>Add</code>, <code>Get</code>, <code>Update</code>, and <code>Delete</code> methods using SQL and Dapper’s lightweight API.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System.Data</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Threading.Tasks</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Dapper</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Microsoft.Data.Sqlite</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">DapperUserRepository</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="kt">string</span> <span class="n">_connectionString</span> <span class="p">=</span> <span class="s">"Data Source=app.db"</span><span class="p">;</span>

    <span class="k">private</span> <span class="n">IDbConnection</span> <span class="nf">CreateConnection</span><span class="p">()</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nf">SqliteConnection</span><span class="p">(</span><span class="n">_connectionString</span><span class="p">);</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">AddUserAsync</span><span class="p">(</span><span class="n">User</span> <span class="n">user</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">const</span> <span class="kt">string</span> <span class="n">sql</span> <span class="p">=</span> <span class="s">"INSERT INTO Users (Name, Email) VALUES (@Name, @Email)"</span><span class="p">;</span>
        <span class="k">using</span> <span class="nn">var</span> <span class="n">connection</span> <span class="p">=</span> <span class="nf">CreateConnection</span><span class="p">();</span>
        <span class="k">await</span> <span class="n">connection</span><span class="p">.</span><span class="nf">ExecuteAsync</span><span class="p">(</span><span class="n">sql</span><span class="p">,</span> <span class="n">user</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">User</span><span class="p">?&gt;</span> <span class="nf">GetUserByIdAsync</span><span class="p">(</span><span class="kt">int</span> <span class="n">id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">const</span> <span class="kt">string</span> <span class="n">sql</span> <span class="p">=</span> <span class="s">"SELECT * FROM Users WHERE Id = @Id"</span><span class="p">;</span>
        <span class="k">using</span> <span class="nn">var</span> <span class="n">connection</span> <span class="p">=</span> <span class="nf">CreateConnection</span><span class="p">();</span>
        <span class="k">return</span> <span class="k">await</span> <span class="n">connection</span><span class="p">.</span><span class="n">QuerySingleOrDefaultAsync</span><span class="p">&lt;</span><span class="n">User</span><span class="p">&gt;(</span><span class="n">sql</span><span class="p">,</span> <span class="k">new</span> <span class="p">{</span> <span class="n">Id</span> <span class="p">=</span> <span class="n">id</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">UpdateUserAsync</span><span class="p">(</span><span class="n">User</span> <span class="n">user</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">const</span> <span class="kt">string</span> <span class="n">sql</span> <span class="p">=</span> <span class="s">"UPDATE Users SET Name = @Name, Email = @Email WHERE Id = @Id"</span><span class="p">;</span>
        <span class="k">using</span> <span class="nn">var</span> <span class="n">connection</span> <span class="p">=</span> <span class="nf">CreateConnection</span><span class="p">();</span>
        <span class="k">await</span> <span class="n">connection</span><span class="p">.</span><span class="nf">ExecuteAsync</span><span class="p">(</span><span class="n">sql</span><span class="p">,</span> <span class="n">user</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">DeleteUserAsync</span><span class="p">(</span><span class="kt">int</span> <span class="n">id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">const</span> <span class="kt">string</span> <span class="n">sql</span> <span class="p">=</span> <span class="s">"DELETE FROM Users WHERE Id = @Id"</span><span class="p">;</span>
        <span class="k">using</span> <span class="nn">var</span> <span class="n">connection</span> <span class="p">=</span> <span class="nf">CreateConnection</span><span class="p">();</span>
        <span class="k">await</span> <span class="n">connection</span><span class="p">.</span><span class="nf">ExecuteAsync</span><span class="p">(</span><span class="n">sql</span><span class="p">,</span> <span class="k">new</span> <span class="p">{</span> <span class="n">Id</span> <span class="p">=</span> <span class="n">id</span> <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">IEnumerable</span><span class="p">&lt;</span><span class="n">User</span><span class="p">&gt;&gt;</span> <span class="nf">GetAllUsersAsync</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">const</span> <span class="kt">string</span> <span class="n">sql</span> <span class="p">=</span> <span class="s">"SELECT * FROM Users"</span><span class="p">;</span>
        <span class="k">using</span> <span class="nn">var</span> <span class="n">connection</span> <span class="p">=</span> <span class="nf">CreateConnection</span><span class="p">();</span>
        <span class="k">return</span> <span class="k">await</span> <span class="n">connection</span><span class="p">.</span><span class="n">QueryAsync</span><span class="p">&lt;</span><span class="n">User</span><span class="p">&gt;(</span><span class="n">sql</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This approach provides <strong>full SQL visibility</strong>, making Dapper an excellent choice for performance-critical services.</p>

<h2>
  
  
  Entity Framework: Productive and Full-Featured ORM
</h2>

<p><strong>Entity Framework (EF)</strong> is a <strong>complete ORM</strong> that abstracts database operations, allowing you to interact with data using <strong>C# objects and LINQ</strong>. It simplifies development, increases readability, and reduces repetitive boilerplate code.</p>

<p>It’s perfect for:</p>

<ul>
<li>Enterprise applications with complex data models.</li>
<li>Teams that value <strong>maintainability and readability</strong>.</li>
<li>Projects requiring features like <strong>migrations</strong>, <strong>tracking</strong>, or <strong>relationships</strong>.</li>
</ul>

<h3>
  
  
  ✅ Pros of Entity Framework
</h3>

<ul>
<li>
<strong>Abstracts SQL:</strong> Lets you focus on domain models, not queries.</li>
<li>
<strong>Boosts productivity:</strong> Simplifies CRUD and relationships.</li>
<li>
<strong>Readable and maintainable:</strong> Improves code and schema clarity.</li>
<li>
<strong>Integrated tooling:</strong> Built-in migrations, seeding, and tracking.</li>
</ul>

<h3>
  
  
  ❌ Cons of Entity Framework
</h3>

<ul>
<li>
<strong>Slower in some cases:</strong> Slight performance trade-off compared to Dapper.</li>
<li>
<strong>Less SQL control:</strong> Automatically generates queries that may not always be optimal.</li>
<li>
<strong>Heavier abstraction:</strong> Adds complexity in debugging and fine-tuning performance.</li>
</ul>

<h2>
  
  
  💻 Example: Entity Framework CRUD Repository in .NET
</h2>

<p>Here’s a <strong>EfUserRepository</strong> class using <strong>Entity Framework Core</strong>.<br>
It performs the same CRUD operations but with less manual SQL handling, relying on EF’s abstraction and LINQ.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">using</span> <span class="nn">System.Threading.Tasks</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">System.Linq</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">EfUserRepository</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="n">AppDbContext</span> <span class="n">_context</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">EfUserRepository</span><span class="p">(</span><span class="n">AppDbContext</span> <span class="n">context</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_context</span> <span class="p">=</span> <span class="n">context</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">AddUserAsync</span><span class="p">(</span><span class="n">User</span> <span class="n">user</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_context</span><span class="p">.</span><span class="n">Users</span><span class="p">.</span><span class="nf">Add</span><span class="p">(</span><span class="n">user</span><span class="p">);</span>
        <span class="k">await</span> <span class="n">_context</span><span class="p">.</span><span class="nf">SaveChangesAsync</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span><span class="p">&lt;</span><span class="n">User</span><span class="p">?&gt;</span> <span class="nf">GetUserByIdAsync</span><span class="p">(</span><span class="kt">int</span> <span class="n">id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="k">await</span> <span class="n">_context</span><span class="p">.</span><span class="n">Users</span><span class="p">.</span><span class="nf">FindAsync</span><span class="p">(</span><span class="n">id</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">UpdateUserAsync</span><span class="p">(</span><span class="n">User</span> <span class="n">user</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">_context</span><span class="p">.</span><span class="n">Users</span><span class="p">.</span><span class="nf">Update</span><span class="p">(</span><span class="n">user</span><span class="p">);</span>
        <span class="k">await</span> <span class="n">_context</span><span class="p">.</span><span class="nf">SaveChangesAsync</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">DeleteUserAsync</span><span class="p">(</span><span class="kt">int</span> <span class="n">id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">user</span> <span class="p">=</span> <span class="k">await</span> <span class="n">_context</span><span class="p">.</span><span class="n">Users</span><span class="p">.</span><span class="nf">FindAsync</span><span class="p">(</span><span class="n">id</span><span class="p">);</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">user</span> <span class="k">is</span> <span class="k">not</span> <span class="k">null</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">_context</span><span class="p">.</span><span class="n">Users</span><span class="p">.</span><span class="nf">Remove</span><span class="p">(</span><span class="n">user</span><span class="p">);</span>
            <span class="k">await</span> <span class="n">_context</span><span class="p">.</span><span class="nf">SaveChangesAsync</span><span class="p">();</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="n">IQueryable</span><span class="p">&lt;</span><span class="n">User</span><span class="p">&gt;</span> <span class="nf">GetAllUsers</span><span class="p">()</span> <span class="p">=&gt;</span> <span class="n">_context</span><span class="p">.</span><span class="n">Users</span><span class="p">.</span><span class="nf">AsQueryable</span><span class="p">();</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This example shows how Entity Framework reduces manual SQL handling, improving <strong>code readability</strong> and <strong>developer productivity</strong>.</p>

<h2>
  
  
  Dapper vs Entity Framework: Quick Comparison
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Dapper</th>
<th>Entity Framework</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Type</strong></td>
<td>Micro ORM</td>
<td>Full ORM</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Faster</td>
<td>Slightly slower</td>
</tr>
<tr>
<td><strong>Control over SQL</strong></td>
<td>Full</td>
<td>Abstracted</td>
</tr>
<tr>
<td><strong>Ease of Use</strong></td>
<td>Simple</td>
<td>Very high-level</td>
</tr>
<tr>
<td><strong>Features</strong></td>
<td>Basic</td>
<td>Advanced (migrations, LINQ, tracking)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Final Considerations
</h2>

<p>When performance and SQL control are your top priorities, <strong>Dapper</strong> is unbeatable. When productivity, maintainability, and abstraction are more important, <strong>Entity Framework</strong> is the right choice.</p>

<p>Before deciding, it’s wise to build a <strong>Proof of Concept (POC)</strong> for your specific scenario to measure performance and development effort.<br>
And remember — <strong>you can use both</strong>: many developers combine <strong>Dapper</strong> for performance-critical queries and <strong>Entity Framework</strong> for general data management.</p>

