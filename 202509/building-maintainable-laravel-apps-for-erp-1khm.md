---
Title: Building Maintainable Laravel Apps for ERP
Description: 
Author: Tahsin Abrar
Date: 2025-09-03T21:43:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>Keep your controllers thin. Put business rules in <strong>Services</strong> and database work in <strong>Repositories/Models</strong>. Always <strong>validate</strong> with <strong>Form Requests</strong>. Use <strong>transactions</strong> for multi-step writes. This setup is easy to test, easy to change, and ready for ERP scale.</p>




<h2>
  
  
  Why this matters
</h2>

<p>In ERP projects, one module touches another. If you mix database code, validation, and business rules inside controllers, changes become risky. A small change in one place can break many screens.</p>

<p>A clean structure helps:</p>

<ul>
<li>
<strong>Controller</strong>: handles HTTP only.</li>
<li>
<strong>Service</strong>: holds business rules.</li>
<li>
<strong>Repository</strong>: talks to the database (via Eloquent).</li>
<li>
<strong>Model</strong>: defines table mapping and relationships.</li>
<li>
<strong>Form Request</strong>: validates incoming data.</li>
</ul>




<h2>
  
  
  Recommended folder layout
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>app/
 ├─ Http/
 │   ├─ Controllers/
 │   │   └─ UserController.php
 │   └─ Requests/
 │       └─ StoreUserRequest.php
 ├─ Models/
 │   └─ User.php
 ├─ Repositories/
 │   └─ UserRepository.php
 └─ Services/
     └─ UserService.php
</code></pre>

</div>



<blockquote>
<p>You can follow the same pattern for every module: Employees, Vendors, Products, Orders, Invoices, etc.</p>
</blockquote>




<h2>
  
  
  Step-by-step: User module (CRUD)
</h2>

<h3>
  
  
  1) Migration (database as last line of defense)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// database/migrations/2025_09_03_000000_create_users_table.php</span>
<span class="kn">use</span> <span class="nc">Illuminate\Database\Migrations\Migration</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">Illuminate\Database\Schema\Blueprint</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">Illuminate\Support\Facades\Schema</span><span class="p">;</span>

<span class="k">return</span> <span class="k">new</span> <span class="kd">class</span> <span class="kd">extends</span> <span class="nc">Migration</span> <span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">up</span><span class="p">():</span> <span class="kt">void</span> <span class="p">{</span>
        <span class="nc">Schema</span><span class="o">::</span><span class="nf">create</span><span class="p">(</span><span class="s1">'users'</span><span class="p">,</span> <span class="k">function</span> <span class="p">(</span><span class="kt">Blueprint</span> <span class="nv">$table</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$table</span><span class="o">-&gt;</span><span class="nf">id</span><span class="p">();</span>
            <span class="nv">$table</span><span class="o">-&gt;</span><span class="nf">string</span><span class="p">(</span><span class="s1">'name'</span><span class="p">,</span> <span class="mi">255</span><span class="p">);</span>
            <span class="nv">$table</span><span class="o">-&gt;</span><span class="nf">string</span><span class="p">(</span><span class="s1">'email'</span><span class="p">)</span><span class="o">-&gt;</span><span class="nf">unique</span><span class="p">();</span>
            <span class="nv">$table</span><span class="o">-&gt;</span><span class="nf">string</span><span class="p">(</span><span class="s1">'password'</span><span class="p">);</span>
            <span class="nv">$table</span><span class="o">-&gt;</span><span class="nf">timestamps</span><span class="p">();</span>
            <span class="nv">$table</span><span class="o">-&gt;</span><span class="nf">softDeletes</span><span class="p">();</span>
        <span class="p">});</span>
    <span class="p">}</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">down</span><span class="p">():</span> <span class="kt">void</span> <span class="p">{</span>
        <span class="nc">Schema</span><span class="o">::</span><span class="nf">dropIfExists</span><span class="p">(</span><span class="s1">'users'</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p><strong>Why</strong>: Unique and not-null constraints protect your data even if a bug slips past validation.</p>




<h3>
  
  
  2) Model (simple and clear)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Models/User.php</span>
<span class="kn">namespace</span> <span class="nn">App\Models</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">Illuminate\Database\Eloquent\Model</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">Illuminate\Database\Eloquent\SoftDeletes</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">User</span> <span class="kd">extends</span> <span class="nc">Model</span>
<span class="p">{</span>
    <span class="kn">use</span> <span class="nc">SoftDeletes</span><span class="p">;</span>

    <span class="k">protected</span> <span class="nv">$fillable</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'name'</span><span class="p">,</span> <span class="s1">'email'</span><span class="p">,</span> <span class="s1">'password'</span><span class="p">];</span>
    <span class="k">protected</span> <span class="nv">$hidden</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'password'</span><span class="p">];</span>
    <span class="k">protected</span> <span class="nv">$casts</span> <span class="o">=</span> <span class="p">[</span>
        <span class="s1">'email_verified_at'</span> <span class="o">=&gt;</span> <span class="s1">'datetime'</span><span class="p">,</span>
    <span class="p">];</span>
<span class="p">}</span>
</code></pre>

</div>






<h3>
  
  
  3) Repository (all DB calls live here)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Repositories/UserRepository.php</span>
<span class="kn">namespace</span> <span class="nn">App\Repositories</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">App\Models\User</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">UserRepository</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">all</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="nc">User</span><span class="o">::</span><span class="nf">orderByDesc</span><span class="p">(</span><span class="s1">'id'</span><span class="p">)</span><span class="o">-&gt;</span><span class="nf">paginate</span><span class="p">(</span><span class="mi">20</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">find</span><span class="p">(</span><span class="kt">int</span> <span class="nv">$id</span><span class="p">):</span> <span class="kt">User</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="nc">User</span><span class="o">::</span><span class="nf">findOrFail</span><span class="p">(</span><span class="nv">$id</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">create</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$data</span><span class="p">):</span> <span class="kt">User</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="nc">User</span><span class="o">::</span><span class="nf">create</span><span class="p">(</span><span class="nv">$data</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">update</span><span class="p">(</span><span class="kt">User</span> <span class="nv">$user</span><span class="p">,</span> <span class="kt">array</span> <span class="nv">$data</span><span class="p">):</span> <span class="kt">User</span>
    <span class="p">{</span>
        <span class="nv">$user</span><span class="o">-&gt;</span><span class="nf">update</span><span class="p">(</span><span class="nv">$data</span><span class="p">);</span>
        <span class="k">return</span> <span class="nv">$user</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">delete</span><span class="p">(</span><span class="kt">User</span> <span class="nv">$user</span><span class="p">):</span> <span class="kt">bool</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="p">(</span><span class="n">bool</span><span class="p">)</span> <span class="nv">$user</span><span class="o">-&gt;</span><span class="nb">delete</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h3>
  
  
  4) Service (business rules + transactions)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Services/UserService.php</span>
<span class="kn">namespace</span> <span class="nn">App\Services</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">App\Repositories\UserRepository</span><span class="p">;</span>
<span class="kn">use</span> <span class="no">Illuminate\Support\Facades\DB</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">Illuminate\Support\Facades\Hash</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">UserService</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">(</span><span class="k">private</span> <span class="kt">UserRepository</span> <span class="nv">$users</span><span class="p">)</span> <span class="p">{}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">list</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">users</span><span class="o">-&gt;</span><span class="nf">all</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">create</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$data</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="c1">// business rules go here</span>
        <span class="nv">$data</span><span class="p">[</span><span class="s1">'password'</span><span class="p">]</span> <span class="o">=</span> <span class="nc">Hash</span><span class="o">::</span><span class="nf">make</span><span class="p">(</span><span class="nv">$data</span><span class="p">[</span><span class="s1">'password'</span><span class="p">]);</span>

        <span class="k">return</span> <span class="no">DB</span><span class="o">::</span><span class="nf">transaction</span><span class="p">(</span><span class="k">function</span> <span class="p">()</span> <span class="k">use</span> <span class="p">(</span><span class="nv">$data</span><span class="p">)</span> <span class="p">{</span>
            <span class="c1">// If you need to write to multiple tables, do it here.</span>
            <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">users</span><span class="o">-&gt;</span><span class="nf">create</span><span class="p">(</span><span class="nv">$data</span><span class="p">);</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">update</span><span class="p">(</span><span class="kt">int</span> <span class="nv">$id</span><span class="p">,</span> <span class="kt">array</span> <span class="nv">$data</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$user</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">users</span><span class="o">-&gt;</span><span class="nf">find</span><span class="p">(</span><span class="nv">$id</span><span class="p">);</span>

        <span class="k">if</span> <span class="p">(</span><span class="o">!</span><span class="k">empty</span><span class="p">(</span><span class="nv">$data</span><span class="p">[</span><span class="s1">'password'</span><span class="p">]))</span> <span class="p">{</span>
            <span class="nv">$data</span><span class="p">[</span><span class="s1">'password'</span><span class="p">]</span> <span class="o">=</span> <span class="nc">Hash</span><span class="o">::</span><span class="nf">make</span><span class="p">(</span><span class="nv">$data</span><span class="p">[</span><span class="s1">'password'</span><span class="p">]);</span>
        <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
            <span class="k">unset</span><span class="p">(</span><span class="nv">$data</span><span class="p">[</span><span class="s1">'password'</span><span class="p">]);</span>
        <span class="p">}</span>

        <span class="k">return</span> <span class="no">DB</span><span class="o">::</span><span class="nf">transaction</span><span class="p">(</span><span class="k">function</span> <span class="p">()</span> <span class="k">use</span> <span class="p">(</span><span class="nv">$user</span><span class="p">,</span> <span class="nv">$data</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">users</span><span class="o">-&gt;</span><span class="nf">update</span><span class="p">(</span><span class="nv">$user</span><span class="p">,</span> <span class="nv">$data</span><span class="p">);</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">delete</span><span class="p">(</span><span class="kt">int</span> <span class="nv">$id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$user</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">users</span><span class="o">-&gt;</span><span class="nf">find</span><span class="p">(</span><span class="nv">$id</span><span class="p">);</span>
        <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">users</span><span class="o">-&gt;</span><span class="nb">delete</span><span class="p">(</span><span class="nv">$user</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<blockquote>
<p>Use <code>DB::transaction()</code> whenever you do two or more writes that must succeed or fail together (classic ERP need).</p>
</blockquote>




<h3>
  
  
  5) Form Request (industry standard validation)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Http/Requests/StoreUserRequest.php</span>
<span class="kn">namespace</span> <span class="nn">App\Http\Requests</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">Illuminate\Foundation\Http\FormRequest</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">StoreUserRequest</span> <span class="kd">extends</span> <span class="nc">FormRequest</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">authorize</span><span class="p">():</span> <span class="kt">bool</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="kc">true</span><span class="p">;</span> <span class="c1">// add role/permission checks if needed</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">rules</span><span class="p">():</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="p">[</span>
            <span class="s1">'name'</span>     <span class="o">=&gt;</span> <span class="s1">'required|string|max:255'</span><span class="p">,</span>
            <span class="s1">'email'</span>    <span class="o">=&gt;</span> <span class="s1">'required|email|unique:users,email'</span><span class="p">,</span>
            <span class="s1">'password'</span> <span class="o">=&gt;</span> <span class="s1">'required|min:8'</span><span class="p">,</span>
        <span class="p">];</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Create an update request too (email unique except current user, password optional).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Http/Requests/UpdateUserRequest.php</span>
<span class="kn">namespace</span> <span class="nn">App\Http\Requests</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">Illuminate\Foundation\Http\FormRequest</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">Illuminate\Validation\Rule</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">UpdateUserRequest</span> <span class="kd">extends</span> <span class="nc">FormRequest</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">authorize</span><span class="p">():</span> <span class="kt">bool</span> <span class="p">{</span> <span class="k">return</span> <span class="kc">true</span><span class="p">;</span> <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">rules</span><span class="p">():</span> <span class="kt">array</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="p">[</span>
            <span class="s1">'name'</span>  <span class="o">=&gt;</span> <span class="s1">'sometimes|required|string|max:255'</span><span class="p">,</span>
            <span class="s1">'email'</span> <span class="o">=&gt;</span> <span class="p">[</span>
                <span class="s1">'sometimes'</span><span class="p">,</span><span class="s1">'required'</span><span class="p">,</span><span class="s1">'email'</span><span class="p">,</span>
                <span class="nc">Rule</span><span class="o">::</span><span class="nf">unique</span><span class="p">(</span><span class="s1">'users'</span><span class="p">,</span> <span class="s1">'email'</span><span class="p">)</span><span class="o">-&gt;</span><span class="nf">ignore</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">route</span><span class="p">(</span><span class="s1">'id'</span><span class="p">)),</span>
            <span class="p">],</span>
            <span class="s1">'password'</span> <span class="o">=&gt;</span> <span class="s1">'nullable|min:8'</span><span class="p">,</span>
        <span class="p">];</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h3>
  
  
  6) Controller (thin and readable)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Http/Controllers/UserController.php</span>
<span class="kn">namespace</span> <span class="nn">App\Http\Controllers</span><span class="p">;</span>

<span class="kn">use</span> <span class="nc">App\Services\UserService</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">App\Http\Requests\StoreUserRequest</span><span class="p">;</span>
<span class="kn">use</span> <span class="nc">App\Http\Requests\UpdateUserRequest</span><span class="p">;</span>

<span class="kd">class</span> <span class="nc">UserController</span> <span class="kd">extends</span> <span class="nc">Controller</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">function</span> <span class="n">__construct</span><span class="p">(</span><span class="k">private</span> <span class="kt">UserService</span> <span class="nv">$service</span><span class="p">)</span> <span class="p">{}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">index</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">return</span> <span class="nf">response</span><span class="p">()</span><span class="o">-&gt;</span><span class="nf">json</span><span class="p">(</span><span class="nv">$this</span><span class="o">-&gt;</span><span class="n">service</span><span class="o">-&gt;</span><span class="k">list</span><span class="p">());</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">store</span><span class="p">(</span><span class="kt">StoreUserRequest</span> <span class="nv">$request</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$user</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">service</span><span class="o">-&gt;</span><span class="nf">create</span><span class="p">(</span><span class="nv">$request</span><span class="o">-&gt;</span><span class="nf">validated</span><span class="p">());</span>
        <span class="k">return</span> <span class="nf">response</span><span class="p">()</span><span class="o">-&gt;</span><span class="nf">json</span><span class="p">(</span><span class="nv">$user</span><span class="p">,</span> <span class="mi">201</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">update</span><span class="p">(</span><span class="kt">UpdateUserRequest</span> <span class="nv">$request</span><span class="p">,</span> <span class="kt">int</span> <span class="nv">$id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$user</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">service</span><span class="o">-&gt;</span><span class="nf">update</span><span class="p">(</span><span class="nv">$id</span><span class="p">,</span> <span class="nv">$request</span><span class="o">-&gt;</span><span class="nf">validated</span><span class="p">());</span>
        <span class="k">return</span> <span class="nf">response</span><span class="p">()</span><span class="o">-&gt;</span><span class="nf">json</span><span class="p">(</span><span class="nv">$user</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">function</span> <span class="n">destroy</span><span class="p">(</span><span class="kt">int</span> <span class="nv">$id</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">service</span><span class="o">-&gt;</span><span class="nb">delete</span><span class="p">(</span><span class="nv">$id</span><span class="p">);</span>
        <span class="k">return</span> <span class="nf">response</span><span class="p">()</span><span class="o">-&gt;</span><span class="nf">json</span><span class="p">(</span><span class="kc">null</span><span class="p">,</span> <span class="mi">204</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>






<h3>
  
  
  7) Routes
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// routes/api.php</span>
<span class="kn">use</span> <span class="nc">App\Http\Controllers\UserController</span><span class="p">;</span>

<span class="nc">Route</span><span class="o">::</span><span class="nf">get</span><span class="p">(</span><span class="s1">'/users'</span><span class="p">,</span> <span class="p">[</span><span class="nc">UserController</span><span class="o">::</span><span class="n">class</span><span class="p">,</span> <span class="s1">'index'</span><span class="p">]);</span>
<span class="nc">Route</span><span class="o">::</span><span class="nf">post</span><span class="p">(</span><span class="s1">'/users'</span><span class="p">,</span> <span class="p">[</span><span class="nc">UserController</span><span class="o">::</span><span class="n">class</span><span class="p">,</span> <span class="s1">'store'</span><span class="p">]);</span>
<span class="nc">Route</span><span class="o">::</span><span class="nf">put</span><span class="p">(</span><span class="s1">'/users/{id}'</span><span class="p">,</span> <span class="p">[</span><span class="nc">UserController</span><span class="o">::</span><span class="n">class</span><span class="p">,</span> <span class="s1">'update'</span><span class="p">]);</span>
<span class="nc">Route</span><span class="o">::</span><span class="nb">delete</span><span class="p">(</span><span class="s1">'/users/{id}'</span><span class="p">,</span> <span class="p">[</span><span class="nc">UserController</span><span class="o">::</span><span class="n">class</span><span class="p">,</span> <span class="s1">'destroy'</span><span class="p">]);</span>
</code></pre>

</div>






<h2>
  
  
  Validation vs Database constraints (short note)
</h2>

<ul>
<li>
<strong>Validate first</strong> (Form Request). This gives nice error messages (HTTP 422) and blocks bad data early.</li>
<li>
<strong>Keep constraints</strong> (unique, not null, foreign keys). These guard your database if something slips through.</li>
</ul>

<p>Both layers together give you strong data quality.</p>




<h2>
  
  
  Patterns that scale well in ERP
</h2>

<ul>
<li><p><strong>DTOs or Arrays?</strong><br>
Arrays are fine for simple cases. DTOs (data objects) help when payloads grow big.</p></li>
<li><p><strong>Events &amp; Jobs</strong><br>
Send emails, logs, or sync tasks as <strong>queued jobs</strong> after commits.<br>
Example: <code>UserCreated</code> event → listener dispatches <code>SendWelcomeEmail</code>.</p></li>
<li><p><strong>API Resources</strong><br>
Use Laravel <code>JsonResource</code> to format consistent API responses.</p></li>
<li><p><strong>Soft Deletes</strong><br>
Enable for important tables. ERP users often “restore” records.</p></li>
<li><p><strong>Global Rules</strong><br>
Put cross-cutting rules (e.g., multi-tenant scopes, company_id filters) in global scopes or services.</p></li>
</ul>




<h2>
  
  
  A quick ERP example: create an Order with items (transaction)
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="c1">// app/Services/OrderService.php (sketch)</span>
<span class="k">public</span> <span class="k">function</span> <span class="n">createOrder</span><span class="p">(</span><span class="kt">array</span> <span class="nv">$data</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">return</span> <span class="no">DB</span><span class="o">::</span><span class="nf">transaction</span><span class="p">(</span><span class="k">function</span> <span class="p">()</span> <span class="k">use</span> <span class="p">(</span><span class="nv">$data</span><span class="p">)</span> <span class="p">{</span>
        <span class="nv">$order</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">orders</span><span class="o">-&gt;</span><span class="nf">create</span><span class="p">([</span>
            <span class="s1">'customer_id'</span> <span class="o">=&gt;</span> <span class="nv">$data</span><span class="p">[</span><span class="s1">'customer_id'</span><span class="p">],</span>
            <span class="s1">'total'</span>       <span class="o">=&gt;</span> <span class="mi">0</span><span class="p">,</span>
        <span class="p">]);</span>

        <span class="nv">$total</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>
        <span class="k">foreach</span> <span class="p">(</span><span class="nv">$data</span><span class="p">[</span><span class="s1">'items'</span><span class="p">]</span> <span class="k">as</span> <span class="nv">$item</span><span class="p">)</span> <span class="p">{</span>
            <span class="nv">$line</span> <span class="o">=</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">orderLines</span><span class="o">-&gt;</span><span class="nf">create</span><span class="p">([</span>
                <span class="s1">'order_id'</span>  <span class="o">=&gt;</span> <span class="nv">$order</span><span class="o">-&gt;</span><span class="n">id</span><span class="p">,</span>
                <span class="s1">'product_id'</span><span class="o">=&gt;</span> <span class="nv">$item</span><span class="p">[</span><span class="s1">'product_id'</span><span class="p">],</span>
                <span class="s1">'qty'</span>       <span class="o">=&gt;</span> <span class="nv">$item</span><span class="p">[</span><span class="s1">'qty'</span><span class="p">],</span>
                <span class="s1">'price'</span>     <span class="o">=&gt;</span> <span class="nv">$item</span><span class="p">[</span><span class="s1">'price'</span><span class="p">],</span>
                <span class="s1">'subtotal'</span>  <span class="o">=&gt;</span> <span class="nv">$item</span><span class="p">[</span><span class="s1">'qty'</span><span class="p">]</span> <span class="o">*</span> <span class="nv">$item</span><span class="p">[</span><span class="s1">'price'</span><span class="p">],</span>
            <span class="p">]);</span>
            <span class="nv">$total</span> <span class="o">+=</span> <span class="nv">$line</span><span class="o">-&gt;</span><span class="n">subtotal</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="nv">$this</span><span class="o">-&gt;</span><span class="n">orders</span><span class="o">-&gt;</span><span class="nf">update</span><span class="p">(</span><span class="nv">$order</span><span class="p">,</span> <span class="p">[</span><span class="s1">'total'</span> <span class="o">=&gt;</span> <span class="nv">$total</span><span class="p">]);</span>

        <span class="c1">// fire events if needed</span>
        <span class="k">return</span> <span class="nv">$order</span><span class="o">-&gt;</span><span class="nf">fresh</span><span class="p">(</span><span class="s1">'lines'</span><span class="p">);</span>
    <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p>If any insert fails, the whole order is rolled back. This is key for ERP integrity.</p>




<h2>
  
  
  Testing (fast confidence)
</h2>

<ul>
<li>
<strong>Unit test services</strong> with repository fakes or an in-memory database.</li>
<li>
<strong>Feature test controllers</strong> to check validation and responses.</li>
</ul>

<p>Example (very short):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="k">public</span> <span class="k">function</span> <span class="n">test_it_creates_user_with_valid_payload</span><span class="p">()</span>
<span class="p">{</span>
    <span class="nv">$payload</span> <span class="o">=</span> <span class="p">[</span>
        <span class="s1">'name'</span> <span class="o">=&gt;</span> <span class="s1">'Alice'</span><span class="p">,</span>
        <span class="s1">'email'</span> <span class="o">=&gt;</span> <span class="s1">'alice@example.com'</span><span class="p">,</span>
        <span class="s1">'password'</span> <span class="o">=&gt;</span> <span class="s1">'secret1234'</span><span class="p">,</span>
    <span class="p">];</span>

    <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">postJson</span><span class="p">(</span><span class="s1">'/api/users'</span><span class="p">,</span> <span class="nv">$payload</span><span class="p">)</span>
        <span class="o">-&gt;</span><span class="nf">assertCreated</span><span class="p">()</span>
        <span class="o">-&gt;</span><span class="nf">assertJsonPath</span><span class="p">(</span><span class="s1">'email'</span><span class="p">,</span> <span class="s1">'alice@example.com'</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>






<h2>
  
  
  Common mistakes to avoid
</h2>

<ul>
<li>Putting DB queries directly in controllers.</li>
<li>Skipping Form Requests and relying only on try/catch.</li>
<li>Mixing HTTP concerns with business logic.</li>
<li>Not using transactions for multi-table writes.</li>
<li>No pagination on list endpoints.</li>
<li>No unique/foreign key constraints in migrations.</li>
</ul>

