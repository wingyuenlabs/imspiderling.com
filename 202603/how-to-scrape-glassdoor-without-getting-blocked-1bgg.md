---
Title: How to Scrape Glassdoor Without Getting Blocked
Description: 
Author: agenthustler
Date: 2026-03-26T22:09:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>Glassdoor is one of the most valuable sources for job market data, company reviews, and salary information. However, it's also one of the most challenging sites to scrape. Here's how to do it reliably.</p>

<h2>
  
  
  Why Glassdoor is Hard to Scrape
</h2>

<p>Glassdoor uses several anti-bot measures:</p>

<ul>
<li>Login walls for most content</li>
<li>Cloudflare protection</li>
<li>Dynamic JavaScript rendering</li>
<li>Aggressive rate limiting</li>
<li>CAPTCHA challenges</li>
</ul>

<h2>
  
  
  The Right Approach: Playwright + Stealth
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip <span class="nb">install </span>playwright
playwright <span class="nb">install </span>chromium
</code></pre>

</div>



<h3>
  
  
  Setting Up a Stealth Browser
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">playwright.sync_api</span> <span class="kn">import</span> <span class="n">sync_playwright</span>
<span class="kn">import</span> <span class="n">random</span><span class="p">,</span> <span class="n">time</span>

<span class="k">def</span> <span class="nf">create_stealth_browser</span><span class="p">():</span>
    <span class="n">pw</span> <span class="o">=</span> <span class="nf">sync_playwright</span><span class="p">().</span><span class="nf">start</span><span class="p">()</span>
    <span class="n">browser</span> <span class="o">=</span> <span class="n">pw</span><span class="p">.</span><span class="n">chromium</span><span class="p">.</span><span class="nf">launch</span><span class="p">(</span>
        <span class="n">headless</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span>
        <span class="n">args</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">--disable-blink-features=AutomationControlled</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">--no-sandbox</span><span class="sh">"</span><span class="p">]</span>
    <span class="p">)</span>
    <span class="n">context</span> <span class="o">=</span> <span class="n">browser</span><span class="p">.</span><span class="nf">new_context</span><span class="p">(</span>
        <span class="n">viewport</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">width</span><span class="sh">"</span><span class="p">:</span> <span class="mi">1920</span><span class="p">,</span> <span class="sh">"</span><span class="s">height</span><span class="sh">"</span><span class="p">:</span> <span class="mi">1080</span><span class="p">},</span>
        <span class="n">user_agent</span><span class="o">=</span><span class="sh">"</span><span class="s">Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) </span><span class="sh">"</span>
                   <span class="sh">"</span><span class="s">AppleWebKit/537.36 (KHTML, like Gecko) </span><span class="sh">"</span>
                   <span class="sh">"</span><span class="s">Chrome/120.0.0.0 Safari/537.36</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">locale</span><span class="o">=</span><span class="sh">"</span><span class="s">en-US</span><span class="sh">"</span>
    <span class="p">)</span>
    <span class="k">return</span> <span class="n">pw</span><span class="p">,</span> <span class="n">browser</span><span class="p">,</span> <span class="n">context</span>

<span class="k">def</span> <span class="nf">human_delay</span><span class="p">():</span>
    <span class="n">time</span><span class="p">.</span><span class="nf">sleep</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">uniform</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="mi">5</span><span class="p">))</span>
</code></pre>

</div>



<h3>
  
  
  Scraping Job Listings
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">scrape_glassdoor_jobs</span><span class="p">(</span><span class="n">query</span><span class="p">,</span> <span class="n">location</span><span class="o">=</span><span class="sh">"</span><span class="s">United States</span><span class="sh">"</span><span class="p">,</span> <span class="n">max_pages</span><span class="o">=</span><span class="mi">3</span><span class="p">):</span>
    <span class="n">pw</span><span class="p">,</span> <span class="n">browser</span><span class="p">,</span> <span class="n">context</span> <span class="o">=</span> <span class="nf">create_stealth_browser</span><span class="p">()</span>
    <span class="n">page</span> <span class="o">=</span> <span class="n">context</span><span class="p">.</span><span class="nf">new_page</span><span class="p">()</span>
    <span class="n">jobs</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">search_url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">https://www.glassdoor.com/Job/jobs.htm?sc.keyword=</span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="s">&amp;locT=N&amp;locId=1</span><span class="sh">"</span>
        <span class="n">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="n">search_url</span><span class="p">,</span> <span class="n">wait_until</span><span class="o">=</span><span class="sh">"</span><span class="s">networkidle</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">human_delay</span><span class="p">()</span>

        <span class="k">for</span> <span class="n">page_num</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="n">max_pages</span><span class="p">):</span>
            <span class="n">page</span><span class="p">.</span><span class="nf">wait_for_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">jobListing</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">,</span> <span class="n">timeout</span><span class="o">=</span><span class="mi">10000</span><span class="p">)</span>
            <span class="n">listings</span> <span class="o">=</span> <span class="n">page</span><span class="p">.</span><span class="nf">query_selector_all</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">jobListing</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>

            <span class="k">for</span> <span class="n">listing</span> <span class="ow">in</span> <span class="n">listings</span><span class="p">:</span>
                <span class="n">title_el</span> <span class="o">=</span> <span class="n">listing</span><span class="p">.</span><span class="nf">query_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">job-title</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
                <span class="n">company_el</span> <span class="o">=</span> <span class="n">listing</span><span class="p">.</span><span class="nf">query_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">emp-name</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
                <span class="n">location_el</span> <span class="o">=</span> <span class="n">listing</span><span class="p">.</span><span class="nf">query_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">emp-location</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>

                <span class="n">jobs</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                    <span class="sh">"</span><span class="s">title</span><span class="sh">"</span><span class="p">:</span> <span class="n">title_el</span><span class="p">.</span><span class="nf">inner_text</span><span class="p">()</span> <span class="k">if</span> <span class="n">title_el</span> <span class="k">else</span> <span class="sh">""</span><span class="p">,</span>
                    <span class="sh">"</span><span class="s">company</span><span class="sh">"</span><span class="p">:</span> <span class="n">company_el</span><span class="p">.</span><span class="nf">inner_text</span><span class="p">()</span> <span class="k">if</span> <span class="n">company_el</span> <span class="k">else</span> <span class="sh">""</span><span class="p">,</span>
                    <span class="sh">"</span><span class="s">location</span><span class="sh">"</span><span class="p">:</span> <span class="n">location_el</span><span class="p">.</span><span class="nf">inner_text</span><span class="p">()</span> <span class="k">if</span> <span class="n">location_el</span> <span class="k">else</span> <span class="sh">""</span>
                <span class="p">})</span>

            <span class="n">next_btn</span> <span class="o">=</span> <span class="n">page</span><span class="p">.</span><span class="nf">query_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">pagination-next</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
            <span class="k">if</span> <span class="n">next_btn</span><span class="p">:</span>
                <span class="n">next_btn</span><span class="p">.</span><span class="nf">click</span><span class="p">()</span>
                <span class="nf">human_delay</span><span class="p">()</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="k">break</span>
    <span class="k">finally</span><span class="p">:</span>
        <span class="n">browser</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
        <span class="n">pw</span><span class="p">.</span><span class="nf">stop</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">jobs</span>
</code></pre>

</div>



<h3>
  
  
  Extracting Salary Data
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">scrape_salaries</span><span class="p">(</span><span class="n">company_slug</span><span class="p">):</span>
    <span class="n">pw</span><span class="p">,</span> <span class="n">browser</span><span class="p">,</span> <span class="n">context</span> <span class="o">=</span> <span class="nf">create_stealth_browser</span><span class="p">()</span>
    <span class="n">page</span> <span class="o">=</span> <span class="n">context</span><span class="p">.</span><span class="nf">new_page</span><span class="p">()</span>
    <span class="n">salaries</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="n">url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">https://www.glassdoor.com/Salary/</span><span class="si">{</span><span class="n">company_slug</span><span class="si">}</span><span class="s">-Salaries.htm</span><span class="sh">"</span>
        <span class="n">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="n">url</span><span class="p">,</span> <span class="n">wait_until</span><span class="o">=</span><span class="sh">"</span><span class="s">networkidle</span><span class="sh">"</span><span class="p">)</span>
        <span class="nf">human_delay</span><span class="p">()</span>

        <span class="n">rows</span> <span class="o">=</span> <span class="n">page</span><span class="p">.</span><span class="nf">query_selector_all</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">salaries-list-item</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
        <span class="k">for</span> <span class="n">row</span> <span class="ow">in</span> <span class="n">rows</span><span class="p">:</span>
            <span class="n">title</span> <span class="o">=</span> <span class="n">row</span><span class="p">.</span><span class="nf">query_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">salary-title</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
            <span class="n">pay</span> <span class="o">=</span> <span class="n">row</span><span class="p">.</span><span class="nf">query_selector</span><span class="p">(</span><span class="sh">'</span><span class="s">[data-test=</span><span class="sh">"</span><span class="s">salary-amount</span><span class="sh">"</span><span class="s">]</span><span class="sh">'</span><span class="p">)</span>
            <span class="k">if</span> <span class="n">title</span> <span class="ow">and</span> <span class="n">pay</span><span class="p">:</span>
                <span class="n">salaries</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                    <span class="sh">"</span><span class="s">job_title</span><span class="sh">"</span><span class="p">:</span> <span class="n">title</span><span class="p">.</span><span class="nf">inner_text</span><span class="p">(),</span>
                    <span class="sh">"</span><span class="s">salary</span><span class="sh">"</span><span class="p">:</span> <span class="n">pay</span><span class="p">.</span><span class="nf">inner_text</span><span class="p">()</span>
                <span class="p">})</span>
    <span class="k">finally</span><span class="p">:</span>
        <span class="n">browser</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
        <span class="n">pw</span><span class="p">.</span><span class="nf">stop</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">salaries</span>
</code></pre>

</div>



<h2>
  
  
  Using Proxy Rotation
</h2>

<p>Glassdoor is aggressive about blocking IPs. Using a proxy service is essential. <a href="https://www.scraperapi.com?fp_ref=the52" rel="noopener noreferrer">ScraperAPI</a> handles IP rotation and JavaScript rendering:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">requests</span>

<span class="k">def</span> <span class="nf">scrape_via_proxy</span><span class="p">(</span><span class="n">url</span><span class="p">):</span>
    <span class="n">params</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">api_key</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">YOUR_KEY</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">url</span><span class="sh">"</span><span class="p">:</span> <span class="n">url</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">render</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">true</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">country_code</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">us</span><span class="sh">"</span>
    <span class="p">}</span>
    <span class="n">response</span> <span class="o">=</span> <span class="n">requests</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">http://api.scraperapi.com</span><span class="sh">"</span><span class="p">,</span> <span class="n">params</span><span class="o">=</span><span class="n">params</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">text</span>
</code></pre>

</div>



<p>For residential proxy rotation, <a href="https://thordata.com/?via=the-data" rel="noopener noreferrer">ThorData</a> provides IPs that look like real users, which is critical for sites with strong anti-bot measures.</p>

<h2>
  
  
  Best Practices
</h2>

<ol>
<li>
<strong>Rate limit aggressively</strong> — 1 request every 3-5 seconds minimum</li>
<li>
<strong>Rotate user agents</strong> — maintain a pool of 20+ realistic user agent strings</li>
<li>
<strong>Use sessions wisely</strong> — don't create a new session for every request</li>
<li>
<strong>Handle CAPTCHAs gracefully</strong> — back off when you encounter them</li>
<li>
<strong>Cache results</strong> — don't re-scrape data you already have</li>
</ol>

<h2>
  
  
  Monitoring Your Scrapers
</h2>

<p>Track your scraper's performance with <a href="https://scrapeops.io/?fpr=the-data28" rel="noopener noreferrer">ScrapeOps</a>. Monitor success rates, response times, and detect when Glassdoor changes its anti-bot measures.</p>

<h2>
  
  
  Legal Considerations
</h2>

<p>Always check Glassdoor's Terms of Service before scraping. Use the data for personal research and analysis. Don't republish scraped content or use it for competitive intelligence without proper legal review.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Scraping Glassdoor requires patience and the right tools. Combine browser automation with proxy rotation, add human-like delays, and always respect the site's resources. The salary and review data is incredibly valuable for job market research when collected responsibly.</p>

