---
Title: Build your own tunnel in Rust: Expose local sites to the Web with blazing performance
Description: 
Author: Bohdan
Date: 2025-12-06T21:48:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Sidecar pattern is supported by Tauri to allow applications to leverage external binaries written in other languages (Go, Python, C++) or to utilize closed-source tools.</p>

<p>The <code>cloudflared</code> executable is included in the application's distribution resources. To support cross-platform deployment, multiple versions of the binary are bundled, and Tauri's resource manager resolves the appropriate binary at runtime based on the host architecture.</p>

<p>When the Spot Serve invokes <code>cloudflared tunnel --url http://localhost:8080</code>, the following sequence occurs:</p>

<ol>
<li><p><strong>Transport Negotiation</strong>: <code>cloudflared</code> initiates an outbound connection to Cloudflare's edge network (Argo). By default, it attempts to use the QUIC protocol (based on UDP) for its performance benefits (reduced latency, congestion control). If QUIC fails (e.g., due to strict UDP firewalls), it falls back to HTTP/2 over TCP.   </p></li>
<li><p><strong>Authentication</strong>: For "Quick Tunnels" (TryCloudflare), no pre-authentication is required. The edge network assigns a temporary, random DNS record under <code>trycloudflare.com</code>.</p></li>
<li><p><strong>Tunnel Establishment</strong>: A persistent session is created. Cloudflare's edge acts as the ingress point.</p></li>
<li><p><strong>Reverse Proxying</strong>:</p></li>
</ol>

<ul>
<li>
<strong>Ingress</strong>: A user makes an HTTPS request to <a href="https://random-name.trycloudflare.com" rel="noopener noreferrer">https://random-name.trycloudflare.com</a>.</li>
<li>
<strong>Routing</strong>: Cloudflare terminates the TLS at the edge, inspects the Host header, and routes the request into the established tunnel connection.</li>
<li>
<strong>Egress</strong>: <code>cloudflared</code> receives the request via the tunnel protocol (Capsule), reconstructs it, and forwards it to <code>localhost:8080</code>.</li>
<li>
<strong>Response</strong>: The local server's response follows the reverse path.</li>
</ul>

<p>This mechanism completely bypasses the need for inbound firewall rules, as the connection is strictly outbound.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4q1aiscpyxr3b0xqirpo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4q1aiscpyxr3b0xqirpo.png" alt="Sidecar tunnel" width="800" height="144"></a></p>

<h2>
  
  
  Code Walkthrough
</h2>

<p><strong>Defining the Sidecar Configuration</strong></p>

<p>In the <code>tauri.conf.json</code> file, the external binary is registered. This tells the bundler to include the binary in the final build artifact.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"tauri"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"bundle"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"externalBin"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"binaries/cloudflared"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="nl">"allowlist"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"shell"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"all"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
        </span><span class="nl">"execute"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
        </span><span class="nl">"sidecar"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This configuration necessitates that the binaries (e.g., <code>cloudflared-x86_64-unknown-linux-gnu</code>) exist in the <code>src-tauri/binaries</code> directory.</p>

<p>The core logic resides in a Rust command module. We must handle the asynchronous spawning of the process and the non-blocking reading of its output streams.</p>

<p>First, we define a structure to hold the active child process. Since Tauri commands are stateless functions, we use <code>tauri::State</code> to manage persistence across calls.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="k">use</span> <span class="nn">std</span><span class="p">::</span><span class="nn">sync</span><span class="p">::</span><span class="n">Mutex</span><span class="p">;</span>
<span class="k">use</span> <span class="nn">tauri</span><span class="p">::</span><span class="n">command</span><span class="p">;</span>
<span class="k">use</span> <span class="nn">tauri</span><span class="p">::</span><span class="nn">api</span><span class="p">::</span><span class="nn">process</span><span class="p">::{</span><span class="n">Command</span><span class="p">,</span> <span class="n">CommandEvent</span><span class="p">};</span>

<span class="k">pub</span> <span class="k">struct</span> <span class="n">TunnelState</span> <span class="p">{</span>
    <span class="c1">// Mutex to safely share the process handle across threads</span>
    <span class="k">pub</span> <span class="n">child_process</span><span class="p">:</span> <span class="n">Mutex</span><span class="o">&lt;</span><span class="nb">Option</span><span class="o">&lt;</span><span class="nn">tauri</span><span class="p">::</span><span class="nn">api</span><span class="p">::</span><span class="nn">process</span><span class="p">::</span><span class="n">CommandChild</span><span class="o">&gt;&gt;</span><span class="p">,</span>
<span class="p">}</span>

<span class="k">impl</span> <span class="nb">Default</span> <span class="k">for</span> <span class="n">TunnelState</span> <span class="p">{</span>
    <span class="k">fn</span> <span class="nf">default</span><span class="p">()</span> <span class="k">-&gt;</span> <span class="k">Self</span> <span class="p">{</span>
        <span class="k">Self</span> <span class="p">{</span>
            <span class="n">child_process</span><span class="p">:</span> <span class="nn">Mutex</span><span class="p">::</span><span class="nf">new</span><span class="p">(</span><span class="nb">None</span><span class="p">),</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Spawning the Tunnel</strong></p>

<p>The <code>start_tunnel</code> function orchestrates the sidecar. Note the use of <code>Command::new_sidecar</code> which handles the path resolution automatically.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="nd">#[command]</span>
<span class="k">pub</span> <span class="k">fn</span> <span class="nf">start_tunnel</span><span class="p">(</span>
    <span class="n">app_handle</span><span class="p">:</span> <span class="nn">tauri</span><span class="p">::</span><span class="n">AppHandle</span><span class="p">,</span>
    <span class="n">port</span><span class="p">:</span> <span class="nb">u16</span><span class="p">,</span>
    <span class="n">state</span><span class="p">:</span> <span class="nn">tauri</span><span class="p">::</span><span class="n">State</span><span class="o">&lt;</span><span class="n">TunnelState</span><span class="o">&gt;</span><span class="p">,</span>
<span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="p">(),</span> <span class="nb">String</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">state_guard</span> <span class="o">=</span> <span class="n">state</span><span class="py">.child_process</span><span class="nf">.lock</span><span class="p">()</span><span class="nf">.map_err</span><span class="p">(|</span><span class="n">_</span><span class="p">|</span> <span class="s">"Failed to lock state"</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>

    <span class="k">if</span> <span class="n">state_guard</span><span class="nf">.is_some</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">return</span> <span class="nf">Err</span><span class="p">(</span><span class="s">"Tunnel is already running"</span><span class="nf">.into</span><span class="p">());</span>
    <span class="p">}</span>

    <span class="c1">// Configure the sidecar command: cloudflared tunnel --url http://localhost:PORT</span>
    <span class="k">let</span> <span class="p">(</span><span class="k">mut</span> <span class="n">rx</span><span class="p">,</span> <span class="n">child</span><span class="p">)</span> <span class="o">=</span> <span class="nn">Command</span><span class="p">::</span><span class="nf">new_sidecar</span><span class="p">(</span><span class="s">"cloudflared"</span><span class="p">)</span>
       <span class="nf">.map_err</span><span class="p">(|</span><span class="n">e</span><span class="p">|</span> <span class="nd">format!</span><span class="p">(</span><span class="s">"Failed to create sidecar command: {}"</span><span class="p">,</span> <span class="n">e</span><span class="p">))</span><span class="o">?</span>
       <span class="nf">.args</span><span class="p">(</span><span class="o">&amp;</span><span class="p">[</span><span class="s">"tunnel"</span><span class="p">,</span> <span class="s">"--url"</span><span class="p">,</span> <span class="o">&amp;</span><span class="nd">format!</span><span class="p">(</span><span class="s">"http://localhost:{}"</span><span class="p">,</span> <span class="n">port</span><span class="p">)])</span>
       <span class="nf">.spawn</span><span class="p">()</span>
       <span class="nf">.map_err</span><span class="p">(|</span><span class="n">e</span><span class="p">|</span> <span class="nd">format!</span><span class="p">(</span><span class="s">"Failed to spawn sidecar: {}"</span><span class="p">,</span> <span class="n">e</span><span class="p">))</span><span class="o">?</span><span class="p">;</span>

    <span class="c1">// Store the child handle for later termination</span>
    <span class="o">*</span><span class="n">state_guard</span> <span class="o">=</span> <span class="nf">Some</span><span class="p">(</span><span class="n">child</span><span class="p">);</span>

    <span class="c1">// Spawn an async task to listen to stdout/stderr events</span>
    <span class="nn">tauri</span><span class="p">::</span><span class="nn">async_runtime</span><span class="p">::</span><span class="nf">spawn</span><span class="p">(</span><span class="k">async</span> <span class="k">move</span> <span class="p">{</span>
        <span class="c1">// Regex to find the TryCloudflare URL</span>
        <span class="c1">// Pattern matches: https://[random-subdomain].trycloudflare.com</span>
        <span class="k">let</span> <span class="n">url_regex</span> <span class="o">=</span> <span class="nn">regex</span><span class="p">::</span><span class="nn">Regex</span><span class="p">::</span><span class="nf">new</span><span class="p">(</span><span class="s">r"https://[a-zA-Z0-9-]+\.trycloudflare\.com"</span><span class="p">)</span><span class="nf">.unwrap</span><span class="p">();</span>

        <span class="k">while</span> <span class="k">let</span> <span class="nf">Some</span><span class="p">(</span><span class="n">event</span><span class="p">)</span> <span class="o">=</span> <span class="n">rx</span><span class="nf">.recv</span><span class="p">()</span><span class="k">.await</span> <span class="p">{</span>
            <span class="k">match</span> <span class="n">event</span> <span class="p">{</span>
                <span class="nn">CommandEvent</span><span class="p">::</span><span class="nf">Stderr</span><span class="p">(</span><span class="n">line</span><span class="p">)</span> <span class="p">|</span> <span class="nn">CommandEvent</span><span class="p">::</span><span class="nf">Stdout</span><span class="p">(</span><span class="n">line</span><span class="p">)</span> <span class="k">=&gt;</span> <span class="p">{</span>
                    <span class="c1">// Log the raw output for debugging</span>
                    <span class="nd">println!</span><span class="p">(</span><span class="s">"[cloudflared]: {}"</span><span class="p">,</span> <span class="n">line</span><span class="p">);</span>

                    <span class="c1">// Check for the URL</span>
                    <span class="k">if</span> <span class="k">let</span> <span class="nf">Some</span><span class="p">(</span><span class="n">mat</span><span class="p">)</span> <span class="o">=</span> <span class="n">url_regex</span><span class="nf">.find</span><span class="p">(</span><span class="o">&amp;</span><span class="n">line</span><span class="p">)</span> <span class="p">{</span>
                        <span class="k">let</span> <span class="n">public_url</span> <span class="o">=</span> <span class="n">mat</span><span class="nf">.as_str</span><span class="p">()</span><span class="nf">.to_string</span><span class="p">();</span>
                        <span class="nd">println!</span><span class="p">(</span><span class="s">"Tunnel Established: {}"</span><span class="p">,</span> <span class="n">public_url</span><span class="p">);</span>

                        <span class="c1">// Emit the URL back to the frontend</span>
                        <span class="n">app_handle</span><span class="nf">.emit_all</span><span class="p">(</span><span class="s">"tunnel-url"</span><span class="p">,</span> <span class="n">public_url</span><span class="p">)</span><span class="nf">.unwrap</span><span class="p">();</span>
                    <span class="p">}</span>
                <span class="p">}</span>
                <span class="nn">CommandEvent</span><span class="p">::</span><span class="nf">Terminated</span><span class="p">(</span><span class="n">payload</span><span class="p">)</span> <span class="k">=&gt;</span> <span class="p">{</span>
                    <span class="nd">println!</span><span class="p">(</span><span class="s">"Tunnel terminated: {:?}"</span><span class="p">,</span> <span class="n">payload</span><span class="p">);</span>
                    <span class="k">break</span><span class="p">;</span>
                <span class="p">}</span>
                <span class="n">_</span> <span class="k">=&gt;</span> <span class="p">{}</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">});</span>

    <span class="nf">Ok</span><span class="p">(())</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Stopping the tunnel is as critical as starting it. If the user closes the app, the <code>cloudflared</code> process must die.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight rust"><code><span class="nd">#[command]</span>
<span class="k">pub</span> <span class="k">fn</span> <span class="nf">stop_tunnel</span><span class="p">(</span><span class="n">state</span><span class="p">:</span> <span class="nn">tauri</span><span class="p">::</span><span class="n">State</span><span class="o">&lt;</span><span class="n">TunnelState</span><span class="o">&gt;</span><span class="p">)</span> <span class="k">-&gt;</span> <span class="nb">Result</span><span class="o">&lt;</span><span class="p">(),</span> <span class="nb">String</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">let</span> <span class="k">mut</span> <span class="n">state_guard</span> <span class="o">=</span> <span class="n">state</span><span class="py">.child_process</span><span class="nf">.lock</span><span class="p">()</span><span class="nf">.map_err</span><span class="p">(|</span><span class="n">_</span><span class="p">|</span> <span class="s">"Failed to lock state"</span><span class="p">)</span><span class="o">?</span><span class="p">;</span>

    <span class="k">if</span> <span class="k">let</span> <span class="nf">Some</span><span class="p">(</span><span class="n">child</span><span class="p">)</span> <span class="o">=</span> <span class="n">state_guard</span><span class="nf">.take</span><span class="p">()</span> <span class="p">{</span>
        <span class="n">child</span><span class="nf">.kill</span><span class="p">()</span><span class="nf">.map_err</span><span class="p">(|</span><span class="n">e</span><span class="p">|</span> <span class="nd">format!</span><span class="p">(</span><span class="s">"Failed to kill process: {}"</span><span class="p">,</span> <span class="n">e</span><span class="p">))</span><span class="o">?</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="nf">Ok</span><span class="p">(())</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Crucially, in a production app, this logic should also be hooked into the application's global <code>WindowEvent::Destroyed</code> or exit handler to prevent zombie processes.</p>




<p>Repository: <a href="">https://github.com/explicit-logic/spot-serve-gui</a></p>

