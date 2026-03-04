---
Title: DNS Lookups: A Developer's Debugging Guide
Description: 
Author: APIVerve
Date: 2026-03-04T21:33:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>DNS is one of the oldest and most fundamental internet technologies. Every domain name lookup, every website visit, every email delivery depends on DNS working correctly. When DNS has problems, things break in ways that aren't always obvious.</p>

<p>Understanding DNS lookups—what they check, what they reveal, and how to interpret results—makes debugging network and email issues dramatically faster. A simple DNS query can reveal problems that would otherwise take hours of investigation to uncover.</p>

<h2>
  
  
  How DNS Works
</h2>

<p>The Domain Name System translates human-readable domain names into IP addresses that computers use to communicate. When you type <code>example.com</code> into a browser, DNS resolves that name to an IP address like <code>93.184.216.34</code>.</p>

<p>This resolution happens through a hierarchy of servers:</p>

<p><strong>Your local DNS resolver</strong> - Usually provided by your ISP or configured manually (like Google's 8.8.8.8 or Cloudflare's 1.1.1.1). This resolver caches responses and queries other servers when needed.</p>

<p><strong>Root nameservers</strong> - The top of the DNS hierarchy. They know where to find information about top-level domains (.com, .org, .net, etc.).</p>

<p><strong>TLD nameservers</strong> - Servers responsible for specific top-level domains. The .com nameservers know which nameservers are authoritative for individual .com domains.</p>

<p><strong>Authoritative nameservers</strong> - The servers that actually hold the DNS records for a specific domain. They provide definitive answers about that domain's configuration.</p>

<p>When you look up a domain, your resolver queries through this hierarchy to find the authoritative answer, then caches the result according to TTL (Time To Live) values.</p>

<h2>
  
  
  Types of DNS Records
</h2>

<p>Different DNS record types serve different purposes. Understanding what each type does helps you know which records to check for different problems.</p>

<p><strong>A Records</strong> map domain names to IPv4 addresses. This is the most basic record type—"this domain points to this IP." Most web hosting relies on A records.</p>

<p><strong>AAAA Records</strong> map domain names to IPv6 addresses. The same concept as A records, but for the newer IP address format.</p>

<p><strong>CNAME Records</strong> create aliases. A CNAME says "this domain is actually that other domain." When you look up a CNAME, you follow the chain until you reach an A record. CDNs and load balancers commonly use CNAMEs.</p>

<p><strong>MX Records</strong> specify mail servers. When someone sends email to <a href="mailto:user@example.com">user@example.com</a>, MX records tell the sending server which mail servers handle email for example.com. MX records include priority values—lower numbers are tried first.</p>

<p><strong>TXT Records</strong> store arbitrary text. They're used for domain verification, SPF email authentication, DKIM signatures, and various other purposes where domains need to publish text data.</p>

<p><strong>NS Records</strong> identify authoritative nameservers. These records tell the DNS system which servers are authoritative for a domain.</p>

<p><strong>SOA Records</strong> contain administrative information about the DNS zone—serial numbers, refresh intervals, and responsible party contact information.</p>

<p>A DNS lookup API returns structured data that's easier to work with than parsing command-line output:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span>
  <span class="dl">'</span><span class="s1">https://api.apiverve.com/v1/dnslookup?domain=example.com</span><span class="dl">'</span><span class="p">,</span>
  <span class="p">{</span> <span class="na">headers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">YOUR_API_KEY</span><span class="dl">'</span> <span class="p">}</span> <span class="p">}</span>
<span class="p">);</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">response</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

<span class="c1">// Check if domain can receive email</span>
<span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">data</span><span class="p">.</span><span class="nx">records</span><span class="p">.</span><span class="nx">MX</span> <span class="o">||</span> <span class="nx">data</span><span class="p">.</span><span class="nx">records</span><span class="p">.</span><span class="nx">MX</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">No MX records - domain cannot receive email</span><span class="dl">'</span><span class="p">);</span>
<span class="p">}</span>

<span class="c1">// Get the IP address the domain points to</span>
<span class="kd">const</span> <span class="nx">ipAddress</span> <span class="o">=</span> <span class="nx">data</span><span class="p">.</span><span class="nx">records</span><span class="p">.</span><span class="nx">A</span><span class="p">?.[</span><span class="mi">0</span><span class="p">];</span>
</code></pre>

</div>



<p>This structured response makes it easy to check specific record types and integrate DNS verification into automated workflows.</p>

<h2>
  
  
  Debugging Connection Problems
</h2>

<p>When an application can't connect to a service, DNS is a common culprit. Connection errors that say "host not found" or "DNS resolution failed" point directly to DNS. But DNS problems can also cause timeouts and connection refused errors when the domain resolves to the wrong IP.</p>

<p><strong>Check if the domain resolves at all.</strong> A domain that doesn't resolve in DNS simply doesn't exist from the internet's perspective. This could mean the domain registration expired, DNS isn't configured, or there's a problem with the authoritative nameservers.</p>

<p><strong>Check if it resolves to the expected IP.</strong> A domain might resolve successfully but to an old IP address. After infrastructure changes, DNS might still point to decommissioned servers. Comparing the resolved IP to expected values catches this quickly.</p>

<p><strong>Check if nameservers are responding.</strong> If a domain's authoritative nameservers are down, DNS resolution fails. The NS records tell you which nameservers should be responding.</p>

<p><strong>Check multiple DNS resolvers.</strong> Different resolvers may have different cached data. If your local resolver has stale cache while others work fine, you've identified the problem.</p>

<h2>
  
  
  DNS Propagation
</h2>

<p>DNS changes don't take effect instantly. When you update DNS records, those changes need to propagate through the caching hierarchy. This propagation can take minutes to hours, sometimes up to 48 hours for full global propagation.</p>

<p><strong>TTL determines cache duration.</strong> Each DNS record has a Time To Live value that tells resolvers how long to cache the result. A TTL of 3600 seconds means resolvers can cache the record for one hour before re-querying.</p>

<p><strong>Different resolvers update at different times.</strong> A resolver that queried your domain 5 minutes before you made a change will serve the old record for almost the full TTL. A resolver that queries after your change gets the new record immediately.</p>

<p><strong>This causes "works for some users" problems.</strong> During propagation, some users see old records, others see new records. Traffic splits between old and new destinations until propagation completes.</p>

<p>DNS propagation checkers query multiple resolvers around the world and show whether they're returning consistent results. During propagation, you'll see mixed results. Once propagation completes, all resolvers return the same data.</p>

<h2>
  
  
  Email Delivery Issues
</h2>

<p>Email delivery depends heavily on DNS. When email isn't arriving, DNS is often involved.</p>

<p><strong>MX records must exist and be correct.</strong> A domain without MX records can't receive email. MX records must point to mail servers that actually accept mail for that domain.</p>

<p><strong>MX records have priorities.</strong> Multiple MX records with different priority values provide failover. The lowest priority number is tried first. If that server is unavailable, senders try higher-numbered priorities.</p>

<p><strong>Mail servers must be reachable.</strong> MX records pointing to non-existent or unreachable servers cause email delivery failures. Checking whether MX servers respond on port 25 confirms they're actually operating.</p>

<p><strong>SPF, DKIM, and DMARC use DNS.</strong> Email authentication mechanisms publish their configuration in TXT records. Misconfigured authentication records cause email to be rejected or marked as spam.</p>

<p>When debugging email issues, checking MX records reveals whether email configuration exists. Checking TXT records shows whether email authentication is properly configured.</p>

<h2>
  
  
  TTL Strategy
</h2>

<p>TTL values balance freshness against efficiency. Lower TTLs mean faster propagation but more DNS queries. Higher TTLs reduce query load but make changes slow to take effect.</p>

<p><strong>Long TTLs (hours to days)</strong> work well for stable records that rarely change. Your main website IP that hasn't changed in years doesn't need a 5-minute TTL.</p>

<p><strong>Short TTLs (minutes)</strong> make sense when you're planning changes or need quick failover. Before a migration, reducing TTL ensures the old value ages out of caches quickly.</p>

<p><strong>Very short TTLs</strong> are sometimes used for dynamic DNS or rapid failover systems. TTLs under a minute are possible but increase DNS query volume significantly.</p>

<p>A common mistake is making changes with long TTLs still in effect. If your TTL is 24 hours, propagation takes up to 24 hours. Lower the TTL in advance, wait for the old TTL to expire, then make changes.</p>

<h2>
  
  
  Multiple A Records and Load Balancing
</h2>

<p>DNS can return multiple A records for a single domain. Clients typically pick one, often in round-robin fashion. This provides basic load balancing and redundancy.</p>

<p><strong>All returned IPs should be healthy.</strong> If DNS returns three IPs and one is unhealthy, roughly one-third of clients connect to the unhealthy server. This causes intermittent failures that can be difficult to diagnose.</p>

<p><strong>DNS doesn't check health.</strong> DNS servers don't verify that the IPs they return are actually working. They just return configured records. Sophisticated setups use health-checking DNS services that automatically remove unhealthy IPs.</p>

<p>When debugging intermittent connectivity issues, checking whether the domain returns multiple IPs and whether all are healthy reveals this class of problem quickly.</p>

<h2>
  
  
  CNAME Chains
</h2>

<p>CNAME records create aliases that point to other domains. Sometimes these chain—domain A is a CNAME to domain B, which is a CNAME to domain C, which finally has an A record.</p>

<p><strong>Long CNAME chains add latency.</strong> Each step requires additional resolution. Most DNS clients limit chain length to prevent infinite loops.</p>

<p><strong>CNAME chains complicate troubleshooting.</strong> When something goes wrong, you need to trace the entire chain to find the problem. The domain you're looking up might work fine, but something it points to might not.</p>

<p><strong>SSL certificates must cover the original domain.</strong> If you access <code>api.example.com</code> and it's a CNAME to <code>cdn.provider.net</code>, the SSL certificate must include <code>api.example.com</code>, not just the CNAME target.</p>

<p>Tracing CNAME chains reveals the actual endpoints your domain points to and helps identify where in the chain problems occur.</p>

<h2>
  
  
  DNS as a Monitoring Target
</h2>

<p>Given how critical DNS is, monitoring it makes sense:</p>

<p><strong>Availability monitoring</strong> - Alert when authoritative nameservers stop responding.</p>

<p><strong>Record change monitoring</strong> - Alert when DNS records change unexpectedly. This can indicate configuration errors, expired domains, or malicious changes.</p>

<p><strong>Propagation monitoring</strong> - After intentional changes, confirm propagation completes as expected.</p>

<p><strong>Response time monitoring</strong> - DNS should respond within milliseconds. Slow DNS resolution affects all dependent services.</p>

<p>DNS monitoring catches infrastructure problems that would otherwise manifest as mysterious application failures.</p>

<h2>
  
  
  Common DNS Problems
</h2>

<p><strong>Domain expired</strong> - Domain registrations must be renewed. An expired domain may stop resolving entirely or be taken over by someone else.</p>

<p><strong>Nameserver misconfiguration</strong> - The domain's nameservers must actually be authoritative for it. Misconfigured nameservers cause resolution failures.</p>

<p><strong>Missing records</strong> - A domain might exist but lack specific records. Missing MX records prevent email; missing A records prevent web access.</p>

<p><strong>Wrong records</strong> - Records pointing to old IPs, decommissioned servers, or incorrect values cause traffic to go to wrong destinations.</p>

<p><strong>TTL misconfiguration</strong> - Extremely long TTLs make recovery from problems slow. Extremely short TTLs increase load and can cause issues with rate-limited DNS services.</p>

<p><strong>Delegation problems</strong> - The parent zone (like .com) must properly delegate to your nameservers. Delegation issues cause resolution failures even when your nameservers are correctly configured.</p>

<h2>
  
  
  DNS and Security
</h2>

<p>DNS plays a role in several security considerations:</p>

<p><strong>Domain hijacking</strong> - If attackers gain control of DNS, they can redirect traffic to malicious servers. Monitoring for unexpected record changes helps detect this.</p>

<p><strong>DNS poisoning</strong> - Attacks that insert false data into DNS caches. DNSSEC helps prevent this by cryptographically signing DNS records.</p>

<p><strong>Domain takeover</strong> - When a CNAME points to a service (like a cloud bucket) that no longer exists, attackers can sometimes claim that service and receive traffic intended for your domain.</p>

<p><strong>Email spoofing</strong> - Without proper SPF, DKIM, and DMARC records, attackers can send email appearing to come from your domain.</p>

<p>DNS security starts with visibility—knowing what your records are and being alerted when they change unexpectedly.</p>

<h2>
  
  
  Practical DNS Debugging
</h2>

<p>When troubleshooting, a systematic approach helps:</p>

<p><strong>Start with basic resolution.</strong> Does the domain resolve at all? What does it resolve to?</p>

<p><strong>Check record types relevant to the problem.</strong> Web issues need A/AAAA records. Email issues need MX records. Specific features might need TXT records.</p>

<p><strong>Compare against expectations.</strong> Is the resolved IP what you expect? Are MX records pointing to your mail servers?</p>

<p><strong>Check propagation if changes were made recently.</strong> Are all resolvers returning the same data, or is propagation still in progress?</p>

<p><strong>Check the entire chain.</strong> If CNAMEs are involved, trace to the final destination.</p>

<p><strong>Check nameserver health.</strong> Are the authoritative nameservers responding properly?</p>

<p>This systematic approach finds most DNS problems quickly. The specific query that reveals the issue depends on the symptoms, but following this progression covers the common cases.</p>

<h2>
  
  
  The Debugging Checklist
</h2>

<p>For quick reference when troubleshooting:</p>

<ol>
<li>Does the domain resolve? (Check A/AAAA records)</li>
<li>Does it resolve to the expected IP?</li>
<li>Are nameservers responding? (Check NS records)</li>
<li>Is DNS consistent globally? (Check propagation)</li>
<li>What's the TTL? (How stale might caches be?)</li>
<li>For email: Do MX records exist and point to working servers?</li>
<li>For CNAMEs: What does the chain look like?</li>
</ol>

<p>Most debugging workflows skip DNS entirely, checking application code, server configuration, and network routes while ignoring the name resolution that happens before any of that matters.</p>

<p>Adding DNS to your debugging checklist catches a category of problems that would otherwise require extensive investigation to identify.</p>




<p>Perform DNS lookups programmatically with the <a href="https://apiverve.com/marketplace/dnslookup?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=dns-lookups-the-debugging-tool-you-forgot-about" rel="noopener noreferrer">DNS Lookup API</a>. Check global propagation with the <a href="https://apiverve.com/marketplace/dnspropagationchecker?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=dns-lookups-the-debugging-tool-you-forgot-about" rel="noopener noreferrer">DNS Propagation Checker API</a>. Verify mail server configuration with the <a href="https://apiverve.com/marketplace/mxlookup?utm_source=devto&amp;utm_medium=crosspost&amp;utm_campaign=dns-lookups-the-debugging-tool-you-forgot-about" rel="noopener noreferrer">MX Lookup API</a>. Debug DNS issues faster.</p>




<p><em>Originally published at <a href="https://blog.apiverve.com/post/dns-lookups-the-debugging-tool-you-forgot-about" rel="noopener noreferrer">APIVerve Blog</a></em></p>

