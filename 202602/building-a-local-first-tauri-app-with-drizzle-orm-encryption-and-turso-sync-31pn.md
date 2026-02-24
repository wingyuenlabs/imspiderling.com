---
Title: Building a Local-First Tauri App with Drizzle ORM, Encryption, and Turso Sync (tauri-plugin-libsql)
Description: 
Author: Huakun Shen
Date: 2026-02-24T21:45:58.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building a Local-First Tauri App with Drizzle ORM, Encryption, and Turso Sync
</h1>

<blockquote>
<p><a href="https://github.com/HuakunShen/tauri-plugin-libsql" rel="noopener noreferrer">https://github.com/HuakunShen/tauri-plugin-libsql</a></p>
</blockquote>

<p>I've been building desktop apps with <a href="https://tauri.app/" rel="noopener noreferrer">Tauri</a> for a while now, and one thing that consistently caused friction was the database layer. Tauri's official <code>@tauri-apps/plugin-sql</code> gets you SQLite, but it has no encryption support and no Drizzle integration that actually works inside a WebView. So I built my own plugin — <code>tauri-plugin-libsql</code> — and this post covers why I built it, the design decisions I made, and a few genuinely weird bugs I ran into along the way.</p>




<h2>
  
  
  The Problem with Databases in Tauri
</h2>

<p>Tauri apps run your UI in a WebView. That WebView is essentially a browser — it has no Node.js <code>fs</code> module, no native bindings, no ability to open SQLite files directly. All access to the filesystem has to go through Tauri's IPC layer: your TypeScript code calls <code>invoke()</code>, and a Rust command handler does the actual work.</p>

<p>The official plugin, <code>@tauri-apps/plugin-sql</code>, handles this. It's fine for basic use. But I kept running into three walls:</p>

<p><strong>1. No encryption.</strong> The plugin uses sqlx under the hood, and sqlx doesn't support SQLite encryption. If you want an encrypted database — say, to protect user data if the device is lost — you're on your own.</p>

<p><strong>2. Drizzle ORM's migrator doesn't work in a WebView.</strong> I love Drizzle. Type-safe queries, a clean migration workflow, great ergonomics. But Drizzle's built-in migrator calls <code>readMigrationFiles()</code>, which uses Node's <code>fs</code> module at runtime. That API doesn't exist in a WebView. You hit a wall immediately when you try to use <code>drizzle-kit migrate</code> in a Tauri app.</p>

<p><strong>3. No Turso support.</strong> I wanted to build local-first apps — data lives on disk, works offline, but syncs to a cloud database when connected. Turso's embedded replica mode does exactly this, but <code>@tauri-apps/plugin-sql</code> has no concept of it.</p>




<h2>
  
  
  Why libsql
</h2>

<p><a href="https://github.com/tursodatabase/libsql" rel="noopener noreferrer">libsql</a> is Turso's fork of SQLite. For the purposes of a Tauri plugin, it offers three things the standard sqlite3 crate doesn't:</p>

<ul>
<li>
<strong>Native encryption</strong> via the <code>encryption</code> feature flag — AES-256-CBC, no extra native libraries</li>
<li>
<strong>Embedded replica mode</strong> — local SQLite file that syncs bidirectionally with Turso cloud</li>
<li>
<strong>A clean async Rust API</strong> with a builder pattern that makes the three modes (local, replica, remote) easy to configure</li>
</ul>

<p>The Rust API looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="c1">// Local only</span>
<span class="k">let</span> <span class="n">db</span> <span class="o">=</span> <span class="nn">Builder</span><span class="p">::</span><span class="nf">new_local</span><span class="p">(</span><span class="s">"myapp.db"</span><span class="p">)</span><span class="nf">.build</span><span class="p">()</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>

<span class="c1">// Embedded replica (local file + Turso sync)</span>
<span class="k">let</span> <span class="n">db</span> <span class="o">=</span> <span class="nn">Builder</span><span class="p">::</span><span class="nf">new_remote_replica</span><span class="p">(</span><span class="s">"local.db"</span><span class="p">,</span> <span class="s">"libsql://mydb.turso.io"</span><span class="p">,</span> <span class="s">"token"</span><span class="p">)</span>
    <span class="nf">.build</span><span class="p">()</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>

<span class="c1">// Initial sync on connect</span>
<span class="n">db</span><span class="nf">.sync</span><span class="p">()</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>
</code></pre>

</div>






<h2>
  
  
  Making Drizzle Work in a WebView
</h2>

<p>Drizzle has a lesser-known driver called <a href="https://orm.drizzle.team/docs/get-started-sqlite#http-proxy" rel="noopener noreferrer">sqlite-proxy</a>. Instead of opening a database file directly, you give Drizzle a callback function. Drizzle generates SQL queries, calls your function with them, and you return the results. It was designed for HTTP-based SQLite proxies, but it works perfectly for Tauri's <code>invoke()</code> pattern.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">drizzle</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">drizzle-orm/sqlite-proxy</span><span class="dl">"</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">invoke</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tauri-apps/api/core</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">proxy</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">sql</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">params</span><span class="p">:</span> <span class="nx">unknown</span><span class="p">[],</span> <span class="nx">method</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">method</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">run</span><span class="dl">"</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">invoke</span><span class="p">(</span><span class="dl">"</span><span class="s2">plugin:libsql|execute</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">db</span><span class="p">:</span> <span class="nx">path</span><span class="p">,</span> <span class="na">query</span><span class="p">:</span> <span class="nx">sql</span><span class="p">,</span> <span class="na">values</span><span class="p">:</span> <span class="nx">params</span> <span class="p">});</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">rows</span><span class="p">:</span> <span class="p">[]</span> <span class="p">};</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">rows</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">invoke</span><span class="p">(</span><span class="dl">"</span><span class="s2">plugin:libsql|select</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span> <span class="na">db</span><span class="p">:</span> <span class="nx">path</span><span class="p">,</span> <span class="na">query</span><span class="p">:</span> <span class="nx">sql</span><span class="p">,</span> <span class="na">values</span><span class="p">:</span> <span class="nx">params</span> <span class="p">});</span>
  <span class="k">return</span> <span class="p">{</span> <span class="na">rows</span><span class="p">:</span> <span class="nx">rows</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nb">Object</span><span class="p">.</span><span class="nx">values</span><span class="p">)</span> <span class="p">};</span>
<span class="p">};</span>

<span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="nf">drizzle</span><span class="p">(</span><span class="nx">proxy</span><span class="p">,</span> <span class="p">{</span> <span class="nx">schema</span> <span class="p">});</span>
</code></pre>

</div>



<p>The plugin exports <code>createDrizzleProxy(path)</code> which wraps this pattern. You get full Drizzle — type-safe queries, joins, transactions, all of it — running against a local SQLite file via Rust, inside a Tauri WebView.</p>




<h2>
  
  
  Solving the Migration Problem
</h2>

<p>Once Drizzle works, you want migrations. The standard <code>drizzle-kit migrate</code> reads <code>.sql</code> files from disk at runtime. In a WebView: no filesystem, no <code>fs</code>, dead end.</p>

<p>The solution is Vite's <code>import.meta.glob</code>. Instead of reading files at runtime, Vite inlines the SQL file contents directly into the JavaScript bundle at build time:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">migrations</span> <span class="o">=</span> <span class="k">import</span><span class="p">.</span><span class="nx">meta</span><span class="p">.</span><span class="nx">glob</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">(</span><span class="dl">"</span><span class="s2">./drizzle/*.sql</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">eager</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="na">query</span><span class="p">:</span> <span class="dl">"</span><span class="s2">?raw</span><span class="dl">"</span><span class="p">,</span>      <span class="c1">// import as raw string</span>
  <span class="na">import</span><span class="p">:</span> <span class="dl">"</span><span class="s2">default</span><span class="dl">"</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<p>This gives you an object like <code>{ "./drizzle/0000_init.sql": "CREATE TABLE ...", ... }</code>. You pass it to <code>migrate()</code>, which tracks applied migrations in a <code>__drizzle_migrations</code> table and runs pending ones in order:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">await</span> <span class="nx">Database</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="dl">"</span><span class="s2">sqlite:myapp.db</span><span class="dl">"</span><span class="p">);</span>
<span class="k">await</span> <span class="nf">migrate</span><span class="p">(</span><span class="dl">"</span><span class="s2">sqlite:myapp.db</span><span class="dl">"</span><span class="p">,</span> <span class="nx">migrations</span><span class="p">);</span>

<span class="c1">// Now safe to use Drizzle</span>
<span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="nf">drizzle</span><span class="p">(</span><span class="nf">createDrizzleProxy</span><span class="p">(</span><span class="dl">"</span><span class="s2">sqlite:myapp.db</span><span class="dl">"</span><span class="p">),</span> <span class="p">{</span> <span class="nx">schema</span> <span class="p">});</span>
</code></pre>

</div>



<p>The order matters. Querying before <code>migrate()</code> runs gives you "no such table" errors.</p>




<h2>
  
  
  Encryption
</h2>

<p>The plugin supports two modes:</p>

<p><strong>Plugin-level encryption</strong> — configured once in Rust, applies to all databases. The key never touches JavaScript:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">let</span> <span class="n">config</span> <span class="o">=</span> <span class="nn">tauri_plugin_libsql</span><span class="p">::</span><span class="n">Config</span> <span class="p">{</span>
    <span class="n">base_path</span><span class="p">:</span> <span class="nf">Some</span><span class="p">(</span><span class="n">cwd</span><span class="p">),</span>
    <span class="n">encryption</span><span class="p">:</span> <span class="nf">Some</span><span class="p">(</span><span class="n">EncryptionConfig</span> <span class="p">{</span>
        <span class="n">cipher</span><span class="p">:</span> <span class="nn">Cipher</span><span class="p">::</span><span class="n">Aes256Cbc</span><span class="p">,</span>
        <span class="n">key</span><span class="p">:</span> <span class="nn">std</span><span class="p">::</span><span class="nn">env</span><span class="p">::</span><span class="nf">var</span><span class="p">(</span><span class="s">"DB_KEY"</span><span class="p">)</span>
            <span class="nf">.map</span><span class="p">(|</span><span class="n">k</span><span class="p">|</span> <span class="n">k</span><span class="nf">.into_bytes</span><span class="p">())</span>
            <span class="nf">.unwrap_or_default</span><span class="p">(),</span>
    <span class="p">}),</span>
<span class="p">};</span>

<span class="nn">tauri</span><span class="p">::</span><span class="nn">Builder</span><span class="p">::</span><span class="nf">default</span><span class="p">()</span>
    <span class="nf">.plugin</span><span class="p">(</span><span class="nn">tauri_plugin_libsql</span><span class="p">::</span><span class="nf">init_with_config</span><span class="p">(</span><span class="n">config</span><span class="p">))</span>
</code></pre>

</div>



<p><strong>Per-database encryption</strong> — key passed from the frontend:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Database</span><span class="p">.</span><span class="nf">load</span><span class="p">({</span>
  <span class="na">path</span><span class="p">:</span> <span class="dl">"</span><span class="s2">sqlite:secrets.db</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">encryption</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">cipher</span><span class="p">:</span> <span class="dl">"</span><span class="s2">aes256cbc</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">key</span><span class="p">:</span> <span class="nb">Array</span><span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="nx">myUint8Key32Bytes</span><span class="p">),</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Plugin-level is the better choice for most apps — it keeps the key in Rust where JavaScript can't inspect it.</p>




<h2>
  
  
  Turso Embedded Replica
</h2>

<p>This is where things get interesting. Turso's embedded replica mode maintains a local SQLite file that stays in sync with a Turso cloud database. Reads come from the local file (fast, offline-capable). Writes go to the remote. You pull the latest changes manually with <code>sync()</code>.</p>

<p>For a Tauri app, this is the ideal architecture. Your app works offline by default. When the user is connected, you sync on launch and on demand.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">db</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">Database</span><span class="p">.</span><span class="nf">load</span><span class="p">({</span>
  <span class="na">path</span><span class="p">:</span> <span class="dl">"</span><span class="s2">sqlite:local.db</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">syncUrl</span><span class="p">:</span> <span class="dl">"</span><span class="s2">libsql://mydb-org.turso.io</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">authToken</span><span class="p">:</span> <span class="k">import</span><span class="p">.</span><span class="nx">meta</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">VITE_TURSO_AUTH_TOKEN</span><span class="p">,</span>
<span class="p">});</span>

<span class="c1">// migrations run against the local replica</span>
<span class="k">await</span> <span class="nf">migrate</span><span class="p">(</span><span class="nx">db</span><span class="p">.</span><span class="nx">path</span><span class="p">,</span> <span class="nx">migrations</span><span class="p">);</span>

<span class="c1">// pull latest remote changes</span>
<span class="k">await</span> <span class="nx">db</span><span class="p">.</span><span class="nf">sync</span><span class="p">();</span>
</code></pre>

</div>



<p><code>Database.load()</code> does an initial sync automatically on connect. Subsequent <code>db.sync()</code> calls pull incremental changes.</p>




<h2>
  
  
  The Weird Bugs
</h2>

<p>Building this plugin involved a few bugs that were interesting enough to be worth writing up.</p>

<h3>
  
  
  Bug 1: The IPC That Never Responded
</h3>

<p>After adding Turso support, I noticed that connecting with a badly-formatted URL caused the app to hang indefinitely on the loading spinner. The Rust terminal showed a panic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>thread 'tokio-runtime-worker' panicked at libsql-0.9.29/src/database/builder.rs:409:66:
called `Result::unwrap()` on an `Err` value: http::Error(InvalidUri(InvalidFormat))
</code></pre>

</div>



<p>libsql's builder calls <code>unwrap()</code> internally when parsing the sync URL. This isn't a bug in the user's code — it's libsql panicking on an <code>Err</code> it should have returned. In Tauri, a panic inside an async command handler causes the IPC response to never be sent. The JavaScript <code>await</code> hangs forever. There's no timeout, no rejection — just silence.</p>

<p>The fix is <code>catch_unwind</code> from the <code>futures</code> crate:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">futures</span><span class="p">::</span><span class="n">FutureExt</span><span class="p">;</span>
<span class="k">use</span> <span class="nn">std</span><span class="p">::</span><span class="nn">panic</span><span class="p">::</span><span class="n">AssertUnwindSafe</span><span class="p">;</span>

<span class="k">let</span> <span class="n">db</span> <span class="o">=</span> <span class="nf">AssertUnwindSafe</span><span class="p">(</span><span class="k">async</span> <span class="k">move</span> <span class="p">{</span>
    <span class="c1">// ... libsql builder calls ...</span>
<span class="p">})</span>
<span class="nf">.catch_unwind</span><span class="p">()</span>
<span class="k">.await</span>
<span class="nf">.map_err</span><span class="p">(|</span><span class="n">_</span><span class="p">|</span> <span class="nn">Error</span><span class="p">::</span><span class="nf">InvalidDbUrl</span><span class="p">(</span>
    <span class="s">"libsql panicked — check your URL format"</span><span class="nf">.into</span><span class="p">()</span>
<span class="p">))</span><span class="o">??</span><span class="p">;</span>
</code></pre>

</div>



<p>The double <code>??</code> unwraps the panic result first (converted to <code>Error</code>), then the inner <code>Result&lt;Database, Error&gt;</code>. Now a bad URL gives a real error message instead of hanging.</p>

<h3>
  
  
  Bug 2: <code>execute_batch</code> Silently Failing with Embedded Replicas
</h3>

<p>The migration runner originally used <code>execute_batch()</code> from libsql to run multiple SQL statements atomically. This worked fine for local databases. With an embedded replica, migrations appeared to succeed — no errors — but the tables didn't actually get created. The Drizzle migrations table would record the migration as applied, then the next query would fail with "no such table".</p>

<p>After a lot of debugging, it turned out that <code>execute_batch()</code> doesn't correctly route writes through the embedded replica's write path in libsql 0.9.x. Individual <code>execute()</code> calls inside an explicit <code>BEGIN</code>/<code>COMMIT</code> do work:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">pub</span> <span class="k">async</span> <span class="k">fn</span> <span class="nf">batch</span><span class="p">(</span><span class="o">&amp;</span><span class="k">self</span><span class="p">,</span> <span class="n">queries</span><span class="p">:</span> <span class="nb">Vec</span><span class="o">&lt;</span><span class="nb">String</span><span class="o">&gt;</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="p">(),</span> <span class="n">Error</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">self</span><span class="py">.conn</span><span class="nf">.execute</span><span class="p">(</span><span class="s">"BEGIN"</span><span class="p">,</span> <span class="nn">Params</span><span class="p">::</span><span class="nb">None</span><span class="p">)</span><span class="k">.await</span><span class="o">?</span><span class="p">;</span>
    <span class="k">for</span> <span class="n">query</span> <span class="k">in</span> <span class="o">&amp;</span><span class="n">queries</span> <span class="p">{</span>
        <span class="k">if</span> <span class="k">let</span> <span class="nf">Err</span><span class="p">(</span><span class="n">e</span><span class="p">)</span> <span class="o">=</span> <span class="k">self</span><span class="py">.conn</span><span class="nf">.execute</span><span class="p">(</span><span class="n">query</span><span class="nf">.as_str</span><span class="p">(),</span> <span class="nn">Params</span><span class="p">::</span><span class="nb">None</span><span class="p">)</span><span class="k">.await</span> <span class="p">{</span>
            <span class="k">let</span> <span class="n">_</span> <span class="o">=</span> <span class="k">self</span><span class="py">.conn</span><span class="nf">.execute</span><span class="p">(</span><span class="s">"ROLLBACK"</span><span class="p">,</span> <span class="nn">Params</span><span class="p">::</span><span class="nb">None</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>
            <span class="k">return</span> <span class="nf">Err</span><span class="p">(</span><span class="nn">Error</span><span class="p">::</span><span class="nf">Libsql</span><span class="p">(</span><span class="n">e</span><span class="p">));</span>
        <span class="p">}</span>
    <span class="p">}</span>
    <span class="k">if</span> <span class="k">let</span> <span class="nf">Err</span><span class="p">(</span><span class="n">e</span><span class="p">)</span> <span class="o">=</span> <span class="k">self</span><span class="py">.conn</span><span class="nf">.execute</span><span class="p">(</span><span class="s">"COMMIT"</span><span class="p">,</span> <span class="nn">Params</span><span class="p">::</span><span class="nb">None</span><span class="p">)</span><span class="k">.await</span> <span class="p">{</span>
        <span class="k">let</span> <span class="n">_</span> <span class="o">=</span> <span class="k">self</span><span class="py">.conn</span><span class="nf">.execute</span><span class="p">(</span><span class="s">"ROLLBACK"</span><span class="p">,</span> <span class="nn">Params</span><span class="p">::</span><span class="nb">None</span><span class="p">)</span><span class="k">.await</span><span class="p">;</span>
        <span class="k">return</span> <span class="nf">Err</span><span class="p">(</span><span class="nn">Error</span><span class="p">::</span><span class="nf">Libsql</span><span class="p">(</span><span class="n">e</span><span class="p">));</span>
    <span class="p">}</span>
    <span class="nf">Ok</span><span class="p">(())</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Tedious, but it works reliably.</p>

<h3>
  
  
  Bug 3: The <code>$state</code> That Wasn't
</h3>

<p>This one was a Svelte 5 mistake in the demo app. I had <code>let dbInstance: Database | null = null</code> — a plain mutable variable, not a <code>$state</code>. The code that checked <code>{#if dbInstance}</code> in the template to show the "Reset DB" button was therefore non-reactive. Setting <code>dbInstance</code> in <code>onMount</code> had no effect on the template.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight svelte"><code><span class="c">&lt;!-- Before: dbInstance not reactive, button never shows --&gt;</span>
let dbInstance: Database | null = null;

<span class="c">&lt;!-- After: reactive, button appears when DB is loaded --&gt;</span>
let dbInstance = $state<span class="nt">&lt;Database</span> <span class="err">|</span> <span class="na">null</span><span class="nt">&gt;</span>(null);
</code></pre>

</div>



<p>Svelte 5 doesn't warn about this in all cases — if the variable is set in <code>onMount</code> and never re-assigned elsewhere, you might not get a warning. The template just silently doesn't update.</p>

<h3>
  
  
  Bug 4: Error State That Ate the Form
</h3>

<p>The CRUD error handling set <code>error = "Failed to add: ..."</code> on failures. The template used <code>{:else if error}</code> to show an error alert — which meant any failed add/toggle/delete replaced the entire todo form with the alert. The user had to refresh to get the form back.</p>

<p>The fix was to stop mixing fatal errors (load failure — should block the UI) with transient errors (write failure — should be a notification). Toast notifications handle the transient case; <code>error</code> state is now only set on fatal load failures.</p>




<h2>
  
  
  The Demo App
</h2>

<p>The plugin ships with a two-panel demo that makes the local vs. remote difference tangible:</p>

<ul>
<li>
<strong>Left panel</strong>: Local SQLite. Writes are instant.</li>
<li>
<strong>Right panel</strong>: Turso embedded replica. Enter your <code>syncUrl</code> and auth token, click Connect. Writes go to Turso — you can watch the latency.</li>
</ul>

<p>Both panels share the same <code>TodoList.svelte</code> component. The difference is just which props are passed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight svelte"><code><span class="c">&lt;!-- Local: instant writes --&gt;</span>
<span class="nt">&lt;TodoList</span> <span class="na">dbFile=</span><span class="s">"todos.db"</span> <span class="na">position=</span><span class="s">"bottom-left"</span> <span class="nt">/&gt;</span>

<span class="c">&lt;!-- Turso: network-latency writes --&gt;</span>
<span class="nt">&lt;TodoList</span>
  <span class="na">dbFile=</span><span class="s">"todos-turso.db"</span>
  <span class="na">syncUrl=</span><span class="si">{</span><span class="nx">appliedSyncUrl</span><span class="si">}</span>
  <span class="na">authToken=</span><span class="si">{</span><span class="nx">appliedAuthToken</span><span class="si">}</span>
  <span class="na">position=</span><span class="s">"bottom-right"</span>
  <span class="na">onDisconnect=</span><span class="si">{</span><span class="nx">handleDisconnect</span><span class="si">}</span>
<span class="nt">/&gt;</span>
</code></pre>

</div>



<p>The <code>position</code> prop routes sonner toasts to different corners so you can see which panel's events are which.</p>




<h2>
  
  
  Installation
</h2>

<p>If you want to try it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># src-tauri/Cargo.toml</span>
<span class="nn">[dependencies]</span>
<span class="py">tauri-plugin-libsql</span> <span class="p">=</span> <span class="s">"0.1.0"</span>

<span class="c"># For Turso sync:</span>
<span class="py">tauri-plugin-libsql</span> <span class="o">=</span> <span class="p">{</span> <span class="py">version</span> <span class="p">=</span> <span class="s">"0.1.0"</span><span class="p">,</span> <span class="py">features</span> <span class="p">=</span> <span class="p">[</span><span class="s">"replication"</span><span class="p">]</span> <span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>tauri-plugin-libsql-api
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="c1">// src-tauri/src/lib.rs</span>
<span class="nn">tauri</span><span class="p">::</span><span class="nn">Builder</span><span class="p">::</span><span class="nf">default</span><span class="p">()</span>
    <span class="nf">.plugin</span><span class="p">(</span><span class="nn">tauri_plugin_libsql</span><span class="p">::</span><span class="nf">init</span><span class="p">())</span>
    <span class="nf">.run</span><span class="p">(</span><span class="nn">tauri</span><span class="p">::</span><span class="nd">generate_context!</span><span class="p">())</span>
    <span class="nf">.expect</span><span class="p">(</span><span class="s">"error while running tauri application"</span><span class="p">);</span>
</code></pre>

</div>



<p>The source is on GitHub. Contributions welcome, especially testing on Windows and Linux.</p>




<h2>
  
  
  What's Next
</h2>

<p>A few things I'd like to add:</p>

<ul>
<li>
<strong>Auto-sync on network reconnect</strong> — Tauri has network status events; <code>db.sync()</code> could fire automatically</li>
<li>
<strong>Conflict visibility</strong> — embedded replica is last-write-wins with no conflict UI</li>
<li>
<strong>iOS/Android testing</strong> — the mobile stubs exist but haven't been verified</li>
</ul>

<p>If you're building a Tauri app and have hit the same walls with the official SQL plugin, give it a try.</p>

