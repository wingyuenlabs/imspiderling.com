---
Title: Configuring and Enabling fail2ban to mitigate Wordpress authentication theft with Debian 12
Description: 
Author: Thomas Blevins
Date: 2025-11-08T21:29:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>We've recently run into a problem with our Wordpress site occasionally getting bombarded with login requests. To safeguard against this, we have implemented fail2ban on our Linux Machine to rate limit these requests.</p>

<h1>
  
  
  Nginx Considerations
</h1>

<p>I am using fail2ban against Nginx access logs, and I've seen multiple times a recommendation to utilize Nginx's built-in rate-limiting <code>limit-req</code> (<a href="https://blog.nginx.org/blog/rate-limiting-nginx" rel="noopener noreferrer">Rate Limiting with Nginx</a>), and their <code>zone</code> idea seems to be what I'm doing when looking for specific requests.</p>

<p>I'm instead implementing fail2ban on its own, and just reading the access logs.</p>

<h1>
  
  
  What is fail2ban?
</h1>

<p>Fail2ban reactively scans <code>log files</code> for requests matching a <code>filter</code> (known as a fail) that (over a <code>findtime</code> duration) break the <code>maxretry</code> limit.  If this happens, it locks them in <code>jail</code> for a <code>bantime</code>, stopping further requests.</p>

<h1>
  
  
  Installing fail2ban
</h1>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt upgrade
<span class="nb">sudo </span>apt <span class="nb">install </span>fail2ban
</code></pre>

</div>



<p>Ensure the package is installed correctly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>fail2ban-client <span class="nt">--version</span>
</code></pre>

</div>



<h1>
  
  
  Configuring fail2ban
</h1>

<p>Copy the default files into files that you can safely customize, without risk of overwriting them with a package update.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cp</span> /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
<span class="nb">cp</span> /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
</code></pre>

</div>



<h1>
  
  
  Configuring the Filter
</h1>

<p><code>/etc/fail2ban/filter.d/wordpress-login.conf</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight conf"><code>[<span class="n">Definition</span>]

<span class="c"># Match any access log line that starts with an IP address and attempts to access the wp-login via POST
# If we do want to limit xml access
#           ^&lt;HOST&gt; .*"(POST) /xmlrpc\.php.*"
</span><span class="n">failregex</span> = ^&lt;<span class="n">HOST</span>&gt; .*<span class="s2">"(POST) /wp-login\.php.*"</span>
            ^&lt;<span class="n">HOST</span>&gt; .*<span class="s2">"(GET|POST) /wp-admin/.*"</span>

<span class="c"># We don't want to ban people on asset requests, just the authentication requests
</span><span class="n">ignoreregex</span> = ^&lt;<span class="n">HOST</span>&gt; .*<span class="s2">"(GET|POST) /wp-admin/(images|js|css)/.*"</span>
</code></pre>

</div>



<h1>
  
  
  Configuring the Jail
</h1>

<p><code>/etc/fail2ban/jail.local</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight conf"><code>[<span class="n">wordpress</span>-<span class="n">login</span>]
<span class="n">enabled</span> = <span class="n">true</span>
<span class="n">filter</span> = <span class="n">wordpress</span>-<span class="n">login</span>
<span class="n">port</span> = <span class="n">http</span>,<span class="n">https</span>
<span class="n">logpath</span> = /<span class="n">var</span>/<span class="n">log</span>/<span class="n">nginx</span>/*.<span class="n">access</span>.<span class="n">log</span>
<span class="n">action</span> = <span class="n">iptables</span>-<span class="n">multiport</span>[<span class="n">name</span>=<span class="n">http</span>, <span class="n">port</span>=<span class="s2">"http,https"</span>, <span class="n">protocol</span>=<span class="n">tcp</span>]
<span class="n">maxretry</span> = <span class="m">10</span>
<span class="n">findtime</span> = <span class="m">60</span>
<span class="n">bantime</span> = <span class="m">3600</span>
</code></pre>

</div>



<ul>
<li>The <code>logpath</code> is utilizing a wildcard to scan all site access logs.</li>
<li>The <code>action</code> is specifying that we will use the <code>iptables</code> of the firewall to restrict the requests.</li>
</ul>

<h1>
  
  
  Testing fail2ban
</h1>

<p>Before booting this up, you can test your individual log files (I believe wildcard is not supported) to see if your filter is reading the "failures" correctly.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>fail2ban-regex /var/log/nginx/my.access.log /etc/fail2ban/filter.d/wordpress-login.conf <span class="nt">--print-all-matched</span>
</code></pre>

</div>



<p>Though, keep in mind you'll have to have logs that will match this filter regex to see anything here.</p>

<h1>
  
  
  Starting and Stopping fail2ban
</h1>

<p>To Start:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>systemctl <span class="nb">enable </span>fail2ban
<span class="nb">sudo </span>systemctl start fail2ban
</code></pre>

</div>



<p>To Stop:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>systemctl disable fail2ban
<span class="nb">sudo </span>systemctl stop fail2ban
</code></pre>

</div>



<h1>
  
  
  Monitoring fail2ban jails
</h1>

<p>To see how a specific jail is faring:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>fail2ban-client status wordpress-login
</code></pre>

</div>



<p>and you'll see something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Status for the jail: wordpress-login
|- Filter
|  |- Currently failed: 156
|  |- Total failed:     101134
|  `- File list:        /var/log/nginx/my.access.log ...
`- Actions
   |- Currently banned: 0
   |- Total banned:     1145
   `- Banned IP list:   
</code></pre>

</div>



<h1>
  
  
  Extra
</h1>

<ul>
<li>Also check out the default <code>sshd</code> jail, all you have to do is enable it.</li>
</ul>

<h1>
  
  
  Resources
</h1>

<ul>
<li><a href="https://developers.cloudflare.com/waf/rate-limiting-rules/best-practices/" rel="noopener noreferrer">Cloudfare - Rate Limiting Best Practices</a></li>
<li><a href="https://bornoe.org/blog/2023/09/basic-fail2ban-commands/" rel="noopener noreferrer">Basic fail2ban Commands</a></li>
</ul>

