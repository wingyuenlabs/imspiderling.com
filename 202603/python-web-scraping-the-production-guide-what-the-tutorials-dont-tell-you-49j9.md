---
Title: Python Web Scraping: The Production Guide (What the Tutorials Don't Tell You)
Description: 
Author: Otto Brennan
Date: 2026-03-05T21:46:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>Python web scraping has a reputation problem. Every tutorial shows you the 10-line BeautifulSoup example that works great... until you try it on a real site.</p>

<p>Then you hit:</p>

<ul>
<li>403 Forbidden</li>
<li>Empty responses (JavaScript-rendered content)</li>
<li>Rate limiting after 50 requests</li>
<li>CAPTCHAs</li>
<li>IP bans</li>
</ul>

<p>I've built scrapers professionally for years. Here's what actually works.</p>

<h2>
  
  
  The Stack
</h2>

<p>For most scraping projects you need exactly two things:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">pip</span> <span class="n">install</span> <span class="n">requests</span> <span class="n">beautifulsoup4</span> <span class="n">lxml</span> <span class="n">playwright</span>
<span class="n">playwright</span> <span class="n">install</span> <span class="n">chromium</span>
</code></pre>

</div>



<p><code>requests</code> + <code>beautifulsoup4</code> for static HTML. <code>playwright</code> for JavaScript-heavy sites. That's it.</p>

<h2>
  
  
  Part 1: The Right Way to Make Requests
</h2>

<p>Most beginners do this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>
<span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">'</span><span class="s">https://example.com/products</span><span class="sh">'</span><span class="p">)</span>
</code></pre>

</div>



<p>Real sites will block you within minutes. Here's what you actually need:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>
<span class="kn">import</span> <span class="n">time</span>
<span class="kn">import</span> <span class="n">random</span>

<span class="n">HEADERS</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">'</span><span class="s">User-Agent</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Accept</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">text/html,application/xhtml+xml,application/xhtml;q=0.9,image/avif,image/webp,*/*;q=0.8</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Accept-Language</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">en-US,en;q=0.9</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Accept-Encoding</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">gzip, deflate, br</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">DNT</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">1</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Connection</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">keep-alive</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Upgrade-Insecure-Requests</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">1</span><span class="sh">'</span><span class="p">,</span>
<span class="p">}</span>

<span class="k">def</span> <span class="nf">get_page</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">session</span><span class="p">,</span> <span class="n">retries</span><span class="o">=</span><span class="mi">3</span><span class="p">):</span>
    <span class="k">for</span> <span class="n">attempt</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">retries</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="c1"># Random delay — looks human, avoids rate limits
</span>            <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="mf">1.5</span><span class="p">,</span> <span class="mf">4.0</span><span class="p">))</span>
            <span class="n">response</span> <span class="o">=</span> <span class="n">session</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">headers</span><span class="o">=</span><span class="n">HEADERS</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">10</span><span class="p">)</span>
            <span class="n">response</span><span class="p">.</span><span class="nf">raise_for_status</span><span class="p">()</span>
            <span class="k">return</span> <span class="n">response</span>
        <span class="k">except</span> <span class="n">requests</span><span class="p">.</span><span class="n">exceptions</span><span class="p">.</span><span class="n">HTTPError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="k">if</span> <span class="n">e</span><span class="p">.</span><span class="n">response</span><span class="p">.</span><span class="n">status_code</span> <span class="o">==</span> <span class="mi">429</span><span class="p">:</span>
                <span class="c1"># Rate limited — back off exponentially
</span>                <span class="n">wait</span> <span class="o">=</span> <span class="p">(</span><span class="mi">2</span> <span class="o">**</span> <span class="n">attempt</span><span class="p">)</span> <span class="o">*</span> <span class="mi">10</span>
                <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Rate limited. Waiting </span><span class="si">{</span><span class="n">wait</span><span class="si">}</span><span class="s">s...</span><span class="sh">"</span><span class="p">)</span>
                <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">wait</span><span class="p">)</span>
            <span class="k">elif</span> <span class="n">e</span><span class="p">.</span><span class="n">response</span><span class="p">.</span><span class="n">status_code</span> <span class="ow">in</span> <span class="p">(</span><span class="mi">403</span><span class="p">,</span> <span class="mi">503</span><span class="p">):</span>
                <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Blocked on attempt </span><span class="si">{</span><span class="n">attempt</span> <span class="o">+</span> <span class="mi">1</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
                <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="mi">5</span><span class="p">,</span> <span class="mi">15</span><span class="p">))</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="k">raise</span>
    <span class="k">return</span> <span class="bp">None</span>

<span class="c1"># Use a Session to maintain cookies across requests
</span><span class="k">with</span> <span class="n">requests</span><span class="p">.</span><span class="nc">Session</span><span class="p">()</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
    <span class="n">page</span> <span class="o">=</span> <span class="nf">get_page</span><span class="p">(</span><span class="sh">'</span><span class="s">https://example.com/products</span><span class="sh">'</span><span class="p">,</span> <span class="n">session</span><span class="p">)</span>
</code></pre>

</div>



<p>Key points:</p>

<ol>
<li>
<strong>Use a real browser User-Agent</strong> — the default <code>python-requests/2.x.x</code> is an instant flag</li>
<li>
<strong>Random delays</strong> — uniform distribution looks more human than fixed delays</li>
<li>
<strong>Exponential backoff on 429s</strong> — respect rate limits or get IP-banned</li>
<li>
<strong>Sessions</strong> — cookies persist, which many sites require for navigation</li>
</ol>

<h2>
  
  
  Part 2: Parsing HTML with BeautifulSoup
</h2>

<p>Once you have the HTML, parsing is straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">bs4</span> <span class="kn">import</span> <span class="n">BeautifulSoup</span>

<span class="k">def</span> <span class="nf">parse_products</span><span class="p">(</span><span class="n">html</span><span class="p">):</span>
    <span class="n">soup</span> <span class="o">=</span> <span class="nc">BeautifulSoup</span><span class="p">(</span><span class="n">html</span><span class="p">,</span> <span class="sh">'</span><span class="s">lxml</span><span class="sh">'</span><span class="p">)</span>  <span class="c1"># lxml is faster than html.parser
</span>    <span class="n">products</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="c1"># Find all product cards
</span>    <span class="k">for</span> <span class="n">card</span> <span class="ow">in</span> <span class="n">soup</span><span class="p">.</span><span class="nf">select</span><span class="p">(</span><span class="sh">'</span><span class="s">.product-card</span><span class="sh">'</span><span class="p">):</span>
        <span class="n">product</span> <span class="o">=</span> <span class="p">{</span>
            <span class="sh">'</span><span class="s">name</span><span class="sh">'</span><span class="p">:</span> <span class="n">card</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">.product-title</span><span class="sh">'</span><span class="p">).</span><span class="nf">get_text</span><span class="p">(</span><span class="n">strip</span><span class="o">=</span><span class="bp">True</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">price</span><span class="sh">'</span><span class="p">:</span> <span class="n">card</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">.price</span><span class="sh">'</span><span class="p">).</span><span class="nf">get_text</span><span class="p">(</span><span class="n">strip</span><span class="o">=</span><span class="bp">True</span><span class="p">),</span>
            <span class="sh">'</span><span class="s">url</span><span class="sh">'</span><span class="p">:</span> <span class="n">card</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">a</span><span class="sh">'</span><span class="p">)[</span><span class="sh">'</span><span class="s">href</span><span class="sh">'</span><span class="p">],</span>
            <span class="c1"># Handle missing elements gracefully
</span>            <span class="sh">'</span><span class="s">rating</span><span class="sh">'</span><span class="p">:</span> <span class="n">card</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">.rating</span><span class="sh">'</span><span class="p">).</span><span class="nf">get_text</span><span class="p">(</span><span class="n">strip</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span> <span class="k">if</span> <span class="n">card</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">.rating</span><span class="sh">'</span><span class="p">)</span> <span class="k">else</span> <span class="bp">None</span><span class="p">,</span>
        <span class="p">}</span>
        <span class="n">products</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">product</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">products</span>
</code></pre>

</div>



<p><strong>Always use <code>.select_one()</code> with a fallback for optional fields.</strong> Sites change their HTML. Your scraper needs to handle missing elements without crashing.</p>

<h2>
  
  
  Part 3: Handling JavaScript-Rendered Sites
</h2>

<p>~40% of modern sites render their content with JavaScript. <code>requests</code> gets the empty skeleton. You need a real browser.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">playwright.sync_api</span> <span class="kn">import</span> <span class="n">sync_playwright</span>
<span class="kn">import</span> <span class="n">time</span>

<span class="k">def</span> <span class="nf">scrape_js_site</span><span class="p">(</span><span class="n">url</span><span class="p">):</span>
    <span class="k">with</span> <span class="nf">sync_playwright</span><span class="p">()</span> <span class="k">as</span> <span class="n">p</span><span class="p">:</span>
        <span class="n">browser</span> <span class="o">=</span> <span class="n">p</span><span class="p">.</span><span class="n">chromium</span><span class="p">.</span><span class="nf">launch</span><span class="p">(</span><span class="n">headless</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
        <span class="n">context</span> <span class="o">=</span> <span class="n">browser</span><span class="p">.</span><span class="nf">new_context</span><span class="p">(</span>
            <span class="c1"># Use a real viewport size
</span>            <span class="n">viewport</span><span class="o">=</span><span class="p">{</span><span class="sh">'</span><span class="s">width</span><span class="sh">'</span><span class="p">:</span> <span class="mi">1920</span><span class="p">,</span> <span class="sh">'</span><span class="s">height</span><span class="sh">'</span><span class="p">:</span> <span class="mi">1080</span><span class="p">},</span>
            <span class="c1"># Pass real browser headers
</span>            <span class="n">extra_http_headers</span><span class="o">=</span><span class="p">{</span>
                <span class="sh">'</span><span class="s">Accept-Language</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">en-US,en;q=0.9</span><span class="sh">'</span><span class="p">,</span>
            <span class="p">}</span>
        <span class="p">)</span>
        <span class="n">page</span> <span class="o">=</span> <span class="n">context</span><span class="p">.</span><span class="nf">new_page</span><span class="p">()</span>

        <span class="c1"># Navigate and wait for content to load
</span>        <span class="n">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="n">url</span><span class="p">)</span>
        <span class="n">page</span><span class="p">.</span><span class="nf">wait_for_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">.product-card</span><span class="sh">'</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">10000</span><span class="p">)</span>

        <span class="c1"># Optional: scroll to trigger lazy loading
</span>        <span class="n">page</span><span class="p">.</span><span class="nf">evaluate</span><span class="p">(</span><span class="sh">'</span><span class="s">window.scrollTo(0, document.body.scrollHeight)</span><span class="sh">'</span><span class="p">)</span>
        <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="mi">2</span><span class="p">)</span>

        <span class="c1"># Get the fully-rendered HTML
</span>        <span class="n">html</span> <span class="o">=</span> <span class="n">page</span><span class="p">.</span><span class="nf">content</span><span class="p">()</span>
        <span class="n">browser</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>

    <span class="k">return</span> <span class="n">html</span>
</code></pre>

</div>



<p>When to use Playwright vs requests:</p>

<ul>
<li>
<code>requests</code>: static HTML, APIs, anything that works in <code>curl</code>
</li>
<li>
<code>playwright</code>: React/Vue/Angular apps, infinite scroll, login flows, anything that needs JS</li>
</ul>

<h2>
  
  
  Part 4: Pagination
</h2>

<p>Most real scraping jobs involve multiple pages. Here's the pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">scrape_all_pages</span><span class="p">(</span><span class="n">base_url</span><span class="p">):</span>
    <span class="n">all_items</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="n">page_num</span> <span class="o">=</span> <span class="mi">1</span>

    <span class="k">with</span> <span class="n">requests</span><span class="p">.</span><span class="nc">Session</span><span class="p">()</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
        <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
            <span class="n">url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">base_url</span><span class="si">}</span><span class="s">?page=</span><span class="si">{</span><span class="n">page_num</span><span class="si">}</span><span class="sh">"</span>
            <span class="n">response</span> <span class="o">=</span> <span class="nf">get_page</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">session</span><span class="p">)</span>

            <span class="k">if</span> <span class="ow">not</span> <span class="n">response</span><span class="p">:</span>
                <span class="k">break</span>

            <span class="n">soup</span> <span class="o">=</span> <span class="nc">BeautifulSoup</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">text</span><span class="p">,</span> <span class="sh">'</span><span class="s">lxml</span><span class="sh">'</span><span class="p">)</span>
            <span class="n">items</span> <span class="o">=</span> <span class="nf">parse_products</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">text</span><span class="p">)</span>

            <span class="k">if</span> <span class="ow">not</span> <span class="n">items</span><span class="p">:</span>
                <span class="k">break</span>  <span class="c1"># No more results
</span>
            <span class="n">all_items</span><span class="p">.</span><span class="nf">extend</span><span class="p">(</span><span class="n">items</span><span class="p">)</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Page </span><span class="si">{</span><span class="n">page_num</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">items</span><span class="p">)</span><span class="si">}</span><span class="s"> items (total: </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">all_items</span><span class="p">)</span><span class="si">}</span><span class="s">)</span><span class="sh">"</span><span class="p">)</span>

            <span class="c1"># Check for next page link
</span>            <span class="n">next_btn</span> <span class="o">=</span> <span class="n">soup</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">a[rel=</span><span class="sh">"</span><span class="s">next</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
            <span class="k">if</span> <span class="ow">not</span> <span class="n">next_btn</span><span class="p">:</span>
                <span class="k">break</span>

            <span class="n">page_num</span> <span class="o">+=</span> <span class="mi">1</span>

    <span class="k">return</span> <span class="n">all_items</span>
</code></pre>

</div>



<h2>
  
  
  Part 5: Storing the Data
</h2>

<p>Don't just print results. Save them properly from the start:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">csv</span>
<span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">sqlite3</span>
<span class="kn">from</span> <span class="n">datetime</span> <span class="kn">import</span> <span class="n">datetime</span>

<span class="k">def</span> <span class="nf">save_to_csv</span><span class="p">(</span><span class="n">items</span><span class="p">,</span> <span class="n">filename</span><span class="p">):</span>
    <span class="k">if</span> <span class="ow">not</span> <span class="n">items</span><span class="p">:</span>
        <span class="k">return</span>
    <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">filename</span><span class="p">,</span> <span class="sh">'</span><span class="s">w</span><span class="sh">'</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">''</span><span class="p">,</span> <span class="n">encoding</span><span class="o">=</span><span class="sh">'</span><span class="s">utf-8</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">writer</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nc">DictWriter</span><span class="p">(</span><span class="n">f</span><span class="p">,</span> <span class="n">fieldnames</span><span class="o">=</span><span class="n">items</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nf">keys</span><span class="p">())</span>
        <span class="n">writer</span><span class="p">.</span><span class="nf">writeheader</span><span class="p">()</span>
        <span class="n">writer</span><span class="p">.</span><span class="nf">writerows</span><span class="p">(</span><span class="n">items</span><span class="p">)</span>
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Saved </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">items</span><span class="p">)</span><span class="si">}</span><span class="s"> items to </span><span class="si">{</span><span class="n">filename</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">save_to_sqlite</span><span class="p">(</span><span class="n">items</span><span class="p">,</span> <span class="n">db_path</span><span class="p">,</span> <span class="n">table_name</span><span class="p">):</span>
    <span class="n">conn</span> <span class="o">=</span> <span class="n">sqlite3</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">db_path</span><span class="p">)</span>
    <span class="n">cursor</span> <span class="o">=</span> <span class="n">conn</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>

    <span class="c1"># Create table from first item's keys
</span>    <span class="k">if</span> <span class="n">items</span><span class="p">:</span>
        <span class="n">columns</span> <span class="o">=</span> <span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="p">.</span><span class="nf">join</span><span class="p">([</span><span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">k</span><span class="si">}</span><span class="s"> TEXT</span><span class="sh">"</span> <span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="n">items</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nf">keys</span><span class="p">()])</span>
        <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">CREATE TABLE IF NOT EXISTS </span><span class="si">{</span><span class="n">table_name</span><span class="si">}</span><span class="s"> (</span><span class="si">{</span><span class="n">columns</span><span class="si">}</span><span class="s">, scraped_at TEXT)</span><span class="sh">"</span><span class="p">)</span>

        <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">items</span><span class="p">:</span>
            <span class="n">values</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">item</span><span class="p">.</span><span class="nf">values</span><span class="p">())</span> <span class="o">+</span> <span class="p">[</span><span class="n">datetime</span><span class="p">.</span><span class="nf">now</span><span class="p">().</span><span class="nf">isoformat</span><span class="p">()]</span>
            <span class="n">placeholders</span> <span class="o">=</span> <span class="sh">'</span><span class="s">, </span><span class="sh">'</span><span class="p">.</span><span class="nf">join</span><span class="p">([</span><span class="sh">'</span><span class="s">?</span><span class="sh">'</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="n">values</span><span class="p">])</span>
            <span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">INSERT INTO </span><span class="si">{</span><span class="n">table_name</span><span class="si">}</span><span class="s"> VALUES (</span><span class="si">{</span><span class="n">placeholders</span><span class="si">}</span><span class="s">)</span><span class="sh">"</span><span class="p">,</span> <span class="n">values</span><span class="p">)</span>

    <span class="n">conn</span><span class="p">.</span><span class="nf">commit</span><span class="p">()</span>
    <span class="n">conn</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
</code></pre>

</div>



<p>SQLite is underused for scraping. It's zero-setup, handles millions of rows, and makes deduplication easy.</p>

<h2>
  
  
  Common Mistakes
</h2>

<p><strong>Mistake 1: Scraping what you should be using an API for</strong></p>

<p>Check for a public API before scraping. <code>robots.txt</code> before scraping too — it tells you what the site allows.</p>

<p><strong>Mistake 2: Not handling network errors</strong></p>

<p>Networks are flaky. Always wrap requests in try/except and implement retry logic (like the <code>get_page()</code> function above).</p>

<p><strong>Mistake 3: Running requests in a tight loop</strong></p>

<p>No delay = IP ban within minutes. <code>time.sleep(random.uniform(1.5, 4.0))</code> between requests is the minimum.</p>

<p><strong>Mistake 4: Storing data as you go vs. collecting then storing</strong></p>

<p>If your scraper crashes on page 47, you lose everything. Store incrementally or checkpoint frequently.</p>

<h2>
  
  
  Putting It Together
</h2>

<p>Here's a complete, production-ready scraper for a static site:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>
<span class="kn">import</span> <span class="n">time</span>
<span class="kn">import</span> <span class="n">random</span>
<span class="kn">import</span> <span class="n">csv</span>
<span class="kn">from</span> <span class="n">bs4</span> <span class="kn">import</span> <span class="n">BeautifulSoup</span>

<span class="n">HEADERS</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">'</span><span class="s">User-Agent</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Accept</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">text/html,application/xhtml+xml,application/xhtml;q=0.9,*/*;q=0.8</span><span class="sh">'</span><span class="p">,</span>
    <span class="sh">'</span><span class="s">Accept-Language</span><span class="sh">'</span><span class="p">:</span> <span class="sh">'</span><span class="s">en-US,en;q=0.5</span><span class="sh">'</span><span class="p">,</span>
<span class="p">}</span>

<span class="k">def</span> <span class="nf">polite_get</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="n">url</span><span class="p">,</span> <span class="n">min_delay</span><span class="o">=</span><span class="mf">1.5</span><span class="p">,</span> <span class="n">max_delay</span><span class="o">=</span><span class="mf">4.0</span><span class="p">):</span>
    <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="n">min_delay</span><span class="p">,</span> <span class="n">max_delay</span><span class="p">))</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">r</span> <span class="o">=</span> <span class="n">session</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">headers</span><span class="o">=</span><span class="n">HEADERS</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">15</span><span class="p">)</span>
        <span class="n">r</span><span class="p">.</span><span class="nf">raise_for_status</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">r</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Error fetching </span><span class="si">{</span><span class="n">url</span><span class="si">}</span><span class="s">: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="bp">None</span>

<span class="k">def</span> <span class="nf">scrape_site</span><span class="p">(</span><span class="n">start_url</span><span class="p">,</span> <span class="n">output_file</span><span class="o">=</span><span class="sh">'</span><span class="s">results.csv</span><span class="sh">'</span><span class="p">):</span>
    <span class="n">results</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">with</span> <span class="n">requests</span><span class="p">.</span><span class="nc">Session</span><span class="p">()</span> <span class="k">as</span> <span class="n">session</span><span class="p">:</span>
        <span class="n">page</span> <span class="o">=</span> <span class="mi">1</span>
        <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
            <span class="n">url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="si">{</span><span class="n">start_url</span><span class="si">}</span><span class="s">?page=</span><span class="si">{</span><span class="n">page</span><span class="si">}</span><span class="sh">"</span>
            <span class="n">response</span> <span class="o">=</span> <span class="nf">polite_get</span><span class="p">(</span><span class="n">session</span><span class="p">,</span> <span class="n">url</span><span class="p">)</span>
            <span class="k">if</span> <span class="ow">not</span> <span class="n">response</span><span class="p">:</span>
                <span class="k">break</span>

            <span class="n">soup</span> <span class="o">=</span> <span class="nc">BeautifulSoup</span><span class="p">(</span><span class="n">response</span><span class="p">.</span><span class="n">text</span><span class="p">,</span> <span class="sh">'</span><span class="s">lxml</span><span class="sh">'</span><span class="p">)</span>

            <span class="c1"># Parse your target elements
</span>            <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">soup</span><span class="p">.</span><span class="nf">select</span><span class="p">(</span><span class="sh">'</span><span class="s">.item</span><span class="sh">'</span><span class="p">):</span>
                <span class="n">results</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                    <span class="sh">'</span><span class="s">title</span><span class="sh">'</span><span class="p">:</span> <span class="n">item</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">h2</span><span class="sh">'</span><span class="p">).</span><span class="nf">get_text</span><span class="p">(</span><span class="n">strip</span><span class="o">=</span><span class="bp">True</span><span class="p">),</span>
                    <span class="sh">'</span><span class="s">link</span><span class="sh">'</span><span class="p">:</span> <span class="n">item</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">a</span><span class="sh">'</span><span class="p">)[</span><span class="sh">'</span><span class="s">href</span><span class="sh">'</span><span class="p">],</span>
                    <span class="sh">'</span><span class="s">description</span><span class="sh">'</span><span class="p">:</span> <span class="n">item</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">p</span><span class="sh">'</span><span class="p">).</span><span class="nf">get_text</span><span class="p">(</span><span class="n">strip</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span> <span class="k">if</span> <span class="n">item</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">p</span><span class="sh">'</span><span class="p">)</span> <span class="k">else</span> <span class="sh">''</span><span class="p">,</span>
                <span class="p">})</span>

            <span class="c1"># Check for next page
</span>            <span class="k">if</span> <span class="ow">not</span> <span class="n">soup</span><span class="p">.</span><span class="nf">select_one</span><span class="p">(</span><span class="sh">'</span><span class="s">.pagination .next</span><span class="sh">'</span><span class="p">):</span>
                <span class="k">break</span>
            <span class="n">page</span> <span class="o">+=</span> <span class="mi">1</span>
            <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Scraped page </span><span class="si">{</span><span class="n">page</span> <span class="o">-</span> <span class="mi">1</span><span class="si">}</span><span class="s">, </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">results</span><span class="p">)</span><span class="si">}</span><span class="s"> total items</span><span class="sh">"</span><span class="p">)</span>

    <span class="c1"># Save results
</span>    <span class="k">if</span> <span class="n">results</span><span class="p">:</span>
        <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">output_file</span><span class="p">,</span> <span class="sh">'</span><span class="s">w</span><span class="sh">'</span><span class="p">,</span> <span class="n">newline</span><span class="o">=</span><span class="sh">''</span><span class="p">,</span> <span class="n">encoding</span><span class="o">=</span><span class="sh">'</span><span class="s">utf-8</span><span class="sh">'</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
            <span class="n">writer</span> <span class="o">=</span> <span class="n">csv</span><span class="p">.</span><span class="nc">DictWriter</span><span class="p">(</span><span class="n">f</span><span class="p">,</span> <span class="n">fieldnames</span><span class="o">=</span><span class="n">results</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="nf">keys</span><span class="p">())</span>
            <span class="n">writer</span><span class="p">.</span><span class="nf">writeheader</span><span class="p">()</span>
            <span class="n">writer</span><span class="p">.</span><span class="nf">writerows</span><span class="p">(</span><span class="n">results</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Done. </span><span class="si">{</span><span class="nf">len</span><span class="p">(</span><span class="n">results</span><span class="p">)</span><span class="si">}</span><span class="s"> items saved to </span><span class="si">{</span><span class="n">output_file</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">'</span><span class="s">__main__</span><span class="sh">'</span><span class="p">:</span>
    <span class="nf">scrape_site</span><span class="p">(</span><span class="sh">'</span><span class="s">https://example.com/products</span><span class="sh">'</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  What's Next
</h2>

<p>This guide covers the fundamentals. Real-world projects also need:</p>

<ul>
<li>
<strong>Proxy rotation</strong> when one IP isn't enough</li>
<li>
<strong>CAPTCHA handling</strong> (2captcha, anti-captcha APIs)</li>
<li>
<strong>Distributed scraping</strong> across multiple machines</li>
<li>
<strong>Incremental runs</strong> that only fetch new data</li>
<li>
<strong>Monitoring</strong> so you know when the site changed its structure</li>
</ul>

<p>If you're building production scrapers and want a head start, I packaged up the scripts I reuse across projects into a <a href="https://ottobrennan.gumroad.com/l/kqfsv" rel="noopener noreferrer">Web Scraping Starter Kit</a> — proxy rotation, Playwright integration, unified storage, and anti-detection headers included.</p>

<p>Happy scraping. Be polite to the servers.</p>

