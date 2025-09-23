---
Title: Performance Tuning for Nginx: 7 Tips to Slash TTFB and Boost Speed
Description: 
Author: Ramer Labs
Date: 2025-09-23T22:00:24.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Time To First Byte (TTFB) is the single most visible metric for a site’s perceived speed. Even if your HTML, CSS, and JavaScript are minified, a sluggish Nginx front‑end can add hundreds of milliseconds before the browser sees any data. In this tutorial we’ll walk through seven concrete steps you can apply on a typical Ubuntu 22.04 server to bring TTFB down to single‑digit milliseconds while keeping the configuration readable and maintainable.</p>




<h2>
  
  
  1. Install and Enable Brotli Compression
</h2>

<p>Brotli often outperforms gzip for text‑based assets (HTML, CSS, JS) while consuming comparable CPU. Ubuntu’s official repositories ship the <code>ngx_brotli</code> module as a dynamic loadable module.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install prerequisites</span>
<span class="nb">sudo </span>apt-get update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt-get <span class="nb">install</span> <span class="nt">-y</span> build-essential libpcre3 libpcre3-dev zlib1g-dev libssl-dev git

<span class="c"># Clone the module source</span>
git clone https://github.com/google/ngx_brotli.git
<span class="nb">cd </span>ngx_brotli <span class="o">&amp;&amp;</span> git submodule update <span class="nt">--init</span>

<span class="c"># Build Nginx with the module (replace &lt;nginx-version&gt; with your installed version)</span>
<span class="nb">cd</span> /usr/src/nginx-
./configure <span class="nt">--add-dynamic-module</span><span class="o">=</span>../ngx_brotli
make modules
<span class="nb">sudo cp </span>objs/ngx_http_brotli_filter_module.so /etc/nginx/modules/
<span class="nb">sudo cp </span>objs/ngx_http_brotli_static_module.so /etc/nginx/modules/

<span class="c"># Enable the module in nginx.conf</span>
<span class="nb">sudo </span>bash <span class="nt">-c</span> <span class="s1">'cat &lt;&lt;EOF &gt;&gt; /etc/nginx/nginx.conf
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;
EOF'</span>
</code></pre>

</div>



<p>Add a simple Brotli block to your server configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">server</span> <span class="p">{</span>
    <span class="kn">listen</span> <span class="mi">443</span> <span class="s">ssl</span> <span class="s">http2</span><span class="p">;</span>
    <span class="c1"># ... SSL settings ...</span>

    <span class="kn">brotli</span> <span class="no">on</span><span class="p">;</span>
    <span class="kn">brotli_comp_level</span> <span class="mi">5</span><span class="p">;</span>
    <span class="kn">brotli_types</span> <span class="nc">text/plain</span> <span class="nc">text/css</span> <span class="nc">application/javascript</span> <span class="nc">application/json</span> <span class="nc">image/svg</span><span class="s">+xml</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>brotli_comp_level</code> of 5 gives a good CPU‑to‑compression ratio. Test with <code>curl -H "Accept-Encoding: br" -I https://example.com</code> and verify the <code>content-encoding: br</code> header.</p>




<h2>
  
  
  2. Fine‑Tune Gzip as a Fallback
</h2>

<p>Not all browsers support Brotli, so keep gzip as a safety net. The default Nginx gzip settings are conservative.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">gzip</span> <span class="no">on</span><span class="p">;</span>
<span class="k">gzip_vary</span> <span class="no">on</span><span class="p">;</span>
<span class="k">gzip_proxied</span> <span class="s">any</span><span class="p">;</span>
<span class="k">gzip_comp_level</span> <span class="mi">4</span><span class="p">;</span>
<span class="k">gzip_min_length</span> <span class="mi">256</span><span class="p">;</span>
<span class="k">gzip_types</span> 
    <span class="nc">application/json</span> 
    <span class="nc">application/javascript</span> 
    <span class="nc">text/css</span> 
    <span class="nc">text/plain</span> 
    <span class="nc">text/xml</span> 
    <span class="nc">image/svg</span><span class="s">+xml</span><span class="p">;</span>
</code></pre>

</div>



<p>Lower <code>gzip_comp_level</code> to 4 to reduce CPU spikes while still achieving ~70% size reduction for most assets.</p>




<h2>
  
  
  3. Optimize SSL/TLS Handshake
</h2>

<p>A heavy TLS handshake can dominate TTFB, especially on first connections. Use modern ciphers and enable session tickets.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">ssl_protocols</span> <span class="s">TLSv1.3</span> <span class="s">TLSv1.2</span><span class="p">;</span>
<span class="k">ssl_prefer_server_ciphers</span> <span class="no">on</span><span class="p">;</span>
<span class="k">ssl_ciphers</span> 
    <span class="s">"TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256"</span><span class="p">;</span>
<span class="k">ssl_session_cache</span> <span class="s">shared:SSL:10m</span><span class="p">;</span>
<span class="k">ssl_session_timeout</span> <span class="s">1d</span><span class="p">;</span>
<span class="k">ssl_session_tickets</span> <span class="no">on</span><span class="p">;</span>
</code></pre>

</div>



<p>TLS 1.3 eliminates the RSA key exchange round‑trip, shaving ~30 ms off the handshake on most clients.</p>




<h2>
  
  
  4. Leverage Cache‑Control Headers
</h2>

<p>Tell browsers and CDNs what to cache. Over‑caching can cause stale content; under‑caching forces unnecessary re‑validation.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">location</span> <span class="p">~</span><span class="sr">*</span> <span class="err">\</span><span class="s">.(css|js|svg|png|jpg|jpeg|gif|webp)</span>$ <span class="p">{</span>
    <span class="kn">expires</span> <span class="s">30d</span><span class="p">;</span>
    <span class="kn">add_header</span> <span class="s">Cache-Control</span> <span class="s">"public,</span> <span class="s">immutable"</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">location</span> <span class="n">/</span> <span class="p">{</span>
    <span class="kn">try_files</span> <span class="nv">$uri</span> <span class="nv">$uri</span><span class="n">/</span> <span class="n">/index.html</span><span class="p">;</span>
    <span class="kn">expires</span> <span class="s">-1</span><span class="p">;</span>
    <span class="kn">add_header</span> <span class="s">Cache-Control</span> <span class="s">"no-store,</span> <span class="s">no-cache,</span> <span class="s">must-revalidate,</span> <span class="s">proxy-revalidate"</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Static assets get a long <code>max‑age</code> while dynamic pages stay fresh.</p>




<h2>
  
  
  5. Disable Unused Modules
</h2>

<p>Every compiled‑in module adds overhead to the request processing pipeline. On a lean LEMP stack you can safely drop <code>mail</code>, <code>stream</code>, and <code>geoip</code> modules.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># List currently loaded modules</span>
nginx <span class="nt">-V</span> 2&gt;&amp;1 | <span class="nb">grep</span> <span class="nt">--</span> <span class="s1">'--add-dynamic-module'</span>

<span class="c"># Re‑compile Nginx without the unwanted ones (example)</span>
./configure <span class="se">\</span>
    <span class="nt">--without-http_mail_module</span> <span class="se">\</span>
    <span class="nt">--without-http_stream_module</span> <span class="se">\</span>
    <span class="nt">--without-http_geoip_module</span>
make <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>make <span class="nb">install</span>
</code></pre>

</div>



<p>A slimmer binary reduces memory footprint and speeds up worker startup.</p>




<h2>
  
  
  6. Tune Worker Processes and Connections
</h2>

<p>Match Nginx workers to the number of CPU cores and set an appropriate <code>worker_connections</code> limit.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">worker_processes</span> <span class="s">auto</span><span class="p">;</span>    <span class="c1"># Uses all available cores</span>
<span class="k">worker_rlimit_nofile</span> <span class="mi">65535</span><span class="p">;</span>

<span class="k">events</span> <span class="p">{</span>
    <span class="kn">worker_connections</span> <span class="mi">8192</span><span class="p">;</span>   <span class="c1"># 8k concurrent connections per worker</span>
    <span class="kn">multi_accept</span> <span class="no">on</span><span class="p">;</span>
    <span class="kn">use</span> <span class="s">epoll</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>auto</code> directive ensures Nginx scales automatically when you add more cores in a cloud VM.</p>




<h2>
  
  
  7. Continuous Monitoring and Load Testing
</h2>

<p>After each change, benchmark with <code>wrk</code> or <code>hey</code> and monitor real‑world latency with Grafana + Prometheus.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Simple load test (10 seconds, 200 connections, 2 threads)</span>
wrk <span class="nt">-t2</span> <span class="nt">-c200</span> <span class="nt">-d10s</span> https://example.com/
</code></pre>

</div>



<p>Collect the <code>http_request_duration_seconds</code> metric from the Nginx exporter and set alerts for any 95th‑percentile spike above 100 ms.</p>




<h2>
  
  
  Conclusion
</h2>

<p>By installing Brotli, fine‑tuning gzip, tightening TLS, using precise cache headers, stripping unused modules, aligning workers with hardware, and keeping an eye on metrics, you can reliably bring TTFB into the low‑digit range. The steps above are incremental—apply them one at a time, verify with a tool like <code>webpagetest.org</code>, and roll back if you see regressions.</p>

<p>If you’re looking for a reliable partner to audit your Nginx setup or need help with a migration to a more scalable architecture, consider checking out <a href="https://lacidaweb.com" rel="noopener noreferrer">https://lacidaweb.com</a> for practical guidance and managed services.</p>

