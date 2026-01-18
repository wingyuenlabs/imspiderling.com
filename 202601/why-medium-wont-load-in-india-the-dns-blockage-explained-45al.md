---
Title: Why Medium Won't Load in India: The DNS Blockage Explained
Description: 
Author: Akhilesh
Date: 2026-01-18T21:05:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>A Deep Dive into Content Blocking, DNS Manipulation, and How to Access Medium</p>

<p>If you're in India and trying to access Medium.com only to be greeted with connection timeouts, DNS errors, or blank pages, you're not alone. Thousands of Indian users face this frustrating issue daily. But what's actually happening behind the scenes? Let's break down the technical and regulatory maze that's keeping you from your favorite articles.</p>

<p>The Problem: Medium is Blocked (Sort of)<br>
Medium isn't experiencing a technical outage in India. The platform works perfectly fine globally. The issue is selective DNS blocking implemented by Indian Internet Service Providers (ISPs) under orders from the Department of Telecommunications (DoT) and the Ministry of Electronics and Information Technology (MeitY).<br>
What You'll Experience:</p>

<ul>
<li>DNS Resolution Failures: "This site can't be reached" errors</li>
<li>Timeout Errors: Pages that load indefinitely</li>
<li>Intermittent Access: Works sometimes, doesn't work other times</li>
<li>ISP-Dependent Issues: Works on some networks but not others</li>
</ul>

<p>Understanding DNS Blocking: How It Works<br>
To understand why Medium won't load, we need to understand how the internet actually works.<br>
Normal DNS Flow (How websites should load):</p>

<ol>
<li>You type medium.com in your browser</li>
<li>Your device asks your ISP's DNS server: "What's the IP address for medium.com?"</li>
<li>DNS server responds: "It's 162.159.152.4"</li>
<li>Your browser connects to that IP address</li>
<li>Medium loads successfully</li>
</ol>

<p>What's Happening in India (DNS Manipulation):</p>

<ol>
<li>You type medium.com in your browser</li>
<li>Your device asks your ISP's DNS server for the IP address</li>
<li>ISP's DNS server either:</li>
</ol>

<ul>
<li>Returns a fake/incorrect IP address</li>
<li>Doesn't respond at all</li>
<li>Returns an error</li>
</ul>

<ol>
<li>Your browser can't connect to Medium</li>
<li>Result: "This site can't be reached" or timeout errors</li>
</ol>

<p>Why Is This Happening?<br>
The blocking stems from India's Information Technology Act, 2000, specifically Section 69A, which allows the government to issue blocking orders to ISPs.<br>
Common Reasons for Blocking:</p>

<ol>
<li>Specific Content Removal Requests</li>
</ol>

<ul>
<li>Government agencies request removal of specific articles</li>
<li>Instead of removing content, Medium may refuse or not respond quickly</li>
<li>Result: Entire domain gets blocked</li>
</ul>

<ol>
<li>Compliance Issues</li>
</ol>

<ul>
<li>Platforms must comply with IT Rules 2021</li>
<li>Requirements include appointing local compliance officers</li>
<li>Non-compliance can trigger blocking orders</li>
</ul>

<ol>
<li>Collateral Damage</li>
</ol>

<ul>
<li>Sometimes blocks target specific subdomains or articles</li>
<li>ISPs implement broad blocks affecting the entire domain</li>
<li>Medium becomes inaccessible even though only specific content was problematic</li>
</ul>

<ol>
<li>Inconsistent Implementation</li>
</ol>

<ul>
<li>Different ISPs implement blocks differently</li>
<li>Some use DNS blocking, others use IP blocking</li>
<li>Creates patchy, unreliable access</li>
</ul>

<p>The Technical Methods of Blocking<br>
Indian ISPs use several techniques to restrict access:</p>

<ol>
<li>DNS Poisoning/Spoofing
The ISP's DNS server intentionally returns wrong information for blocked domains.</li>
</ol>

<p>`&gt; User Request: What's medium.com's IP?</p>

<blockquote>
<p>ISP DNS Response: 0.0.0.0 (invalid) or no response<br>
Result: Site won't load`</p>
</blockquote>

<ol>
<li><p>IP Address Blocking<br>
Even if you know Medium's IP address, the ISP blocks traffic to it.<br>
<code>Your Request → ISP Router → BLOCKED → Medium's Server<br>
Result: Connection timeout</code></p></li>
<li><p>Deep Packet Inspection (DPI)<br>
ISPs analyze your traffic and block anything going to/from blocked domains.<br>
<code>ISP detects: "This traffic is going to medium.com"<br>
Action: Drop the packets<br>
Result: Connection fails</code></p></li>
<li><p>SNI-based Blocking<br>
During HTTPS connection, your browser reveals which site you're accessing (Server Name Indication). ISPs can block based on this.<br>
Why DNS Issues Specifically?<br>
DNS blocking is the most common method in India because:<br>
✓ Easy to implement - ISPs just modify their DNS servers<br>
✓ Cost-effective - No expensive hardware needed<br>
✓ Plausible deniability - Can be blamed on "technical issues"<br>
✓ Selective - Can block specific domains without affecting others<br>
✗ Easily bypassed - Users can switch DNS servers<br>
How to Access Medium from India<br>
Here are practical solutions, ranked by ease of implementation:<br>
Solution 1: Change Your DNS Server (Easiest)<br>
Instead of using your ISP's DNS, use public DNS servers:<br>
Google DNS:</p></li>
</ol>

<p>Primary: 8.8.8.8<br>
Secondary: 8.8.4.4</p>

<p>Cloudflare DNS:</p>

<p>Primary: 1.1.1.1<br>
Secondary: 1.0.0.1</p>

<p>How to Change DNS (Windows):</p>

<ol>
<li>Open Settings → Network &amp; Internet</li>
<li>Click on your connection → Properties</li>
<li>Scroll to DNS server assignment → Edit</li>
<li>Select Manual, toggle IPv4 ON</li>
<li>Enter preferred DNS: 1.1.1.1</li>
<li>Enter alternate DNS: 1.0.0.1</li>
<li>Save</li>
</ol>

<p>How to Change DNS (Android):</p>

<ol>
<li>Settings → Wi-Fi</li>
<li>Long-press your network → Modify</li>
<li>Advanced options → IP settings → Static</li>
<li>Enter DNS 1: 1.1.1.1</li>
<li>Enter DNS 2: 1.0.0.1</li>
<li>Save</li>
</ol>

<p>How to Change DNS (iOS):</p>

<ol>
<li>Settings → Wi-Fi</li>
<li>Tap (i) next to your network</li>
<li>Configure DNS → Manual</li>
<li>Remove existing DNS, add 1.1.1.1 and 1.0.0.1</li>
<li>Save</li>
</ol>

<p>Solution 2: Use VPN Services<br>
VPNs encrypt all your traffic and route it through servers outside India.<br>
Recommended VPNs:</p>

<ul>
<li>ProtonVPN (free tier available)</li>
<li>Windscribe (free 10GB/month)</li>
<li>Mullvad</li>
<li>NordVPN</li>
<li>ExpressVPN</li>
</ul>

<p>Why VPNs Work:</p>

<ul>
<li>Your ISP can't see which websites you're accessing</li>
<li>All traffic appears to go to the VPN server</li>
<li>VPN server accesses Medium on your behalf</li>
</ul>

<p>Solution 3: Use Tor Browser<br>
Tor routes your traffic through multiple encrypted nodes, making it extremely difficult to block.<br>
Pros:</p>

<ul>
<li>Free and open-source</li>
<li>Strong privacy protection</li>
<li>Bypasses most blocking</li>
</ul>

<p>Cons:</p>

<ul>
<li>Slower browsing speeds</li>
<li>Some sites block Tor exit nodes</li>
</ul>

<p>Solution 4: Browser Extensions<br>
Extensions like HTTPS Everywhere and Decentraleyes can sometimes help, though they're less effective against DNS blocking.</p>

<p>Solution 5: Use Alternative Frontends<br>
Some services create mirrors or alternative interfaces:</p>

<ul>
<li>Scribe.rip (Medium alternative frontend)</li>
<li>Archive.is (saves Medium articles)</li>
</ul>

<p>The Broader Implications<br>
This isn't just about accessing Medium. It represents a concerning trend:</p>

<ol>
<li>Erosion of Internet Freedom
India has seen increasing internet restrictions, from social media bans to website blocking.</li>
<li>Lack of Transparency
Blocking orders are often not publicly disclosed. Users don't know why sites are blocked or for how long.</li>
<li>Collateral Damage
Broad domain blocking affects innocent content alongside problematic content.</li>
<li>Technical Literacy Gap
Most users don't understand DNS or how to bypass blocks, creating information inequality.</li>
<li>Impact on Content Creators
Indian writers on Medium lose access to their own audience and earnings.
What Needs to Change?
For the Government:</li>
</ol>

<ul>
<li>Transparency: Publish blocking orders with justifications</li>
<li>Proportionality: Block specific content, not entire platforms</li>
<li>Due Process: Allow platforms to appeal blocking decisions</li>
<li>Compliance Support: Help platforms understand and meet Indian requirements</li>
</ul>

<p>For Platforms:</p>

<ul>
<li>Local Representation: Appoint compliance officers as required</li>
<li>Responsive Action: Address takedown requests promptly</li>
<li>User Communication: Inform users about regional restrictions</li>
</ul>

<p>For ISPs:</p>

<ul>
<li>Precise Implementation: Block only what's ordered, not entire domains</li>
<li>User Notification: Inform users when sites are blocked and why</li>
</ul>

<p>For Users:</p>

<ul>
<li>Digital Literacy: Learn about DNS, VPNs, and internet infrastructure</li>
<li>Advocacy: Support organizations fighting for internet freedom</li>
<li>Technical Solutions: Use the workarounds outlined above</li>
</ul>

<p>The Legal Framework<br>
Understanding the law helps contextualize the issue:<br>
Information Technology Act, 2000 - Section 69A:<br>
Gives the government power to block public access to information "in the interest of sovereignty and integrity of India, defence of India, security of the State, friendly relations with foreign States or public order."<br>
IT Rules, 2021:<br>
Requires intermediaries to:</p>

<ul>
<li>Appoint grievance officers</li>
<li>Remove content within 36 hours of government order</li>
<li>Implement proactive content moderation</li>
<li>Preserve user data</li>
</ul>

<p>Lack of Oversight:</p>

<ul>
<li>No independent review board</li>
<li>Orders can be indefinite</li>
<li>Limited appeal mechanisms</li>
<li>Minimal public accountability</li>
</ul>

<p>Real-World Impact<br>
Let's talk numbers and stories:<br>
<em>Statistics:</em></p>

<ul>
<li>India ranks among top countries for internet shutdowns globally</li>
<li>Hundreds of websites face blocking orders annually</li>
<li>Millions of users affected by DNS manipulation</li>
<li>Growing trend: 2023 saw 50+ major website blocks</li>
</ul>

<p>User Stories:<br>
"I'm a freelance writer. Medium is where I publish and earn. When it got blocked, I lost access to my dashboard, couldn't publish new articles, and missed out on earnings. I had to learn about VPNs just to access my own content." - Priya, Content Writer<br>
"As a developer, I rely on dev.to and Medium for technical tutorials. DNS blocking forced me to waste time setting up VPNs instead of learning. It's frustrating." - Rahul, Software Engineer<br>
Technical Deep Dive: DNS in Detail<br>
For the technically curious, let's explore DNS more deeply:<br>
DNS Query Flow:</p>

<blockquote>
<ol>
<li>Browser Cache Check
↓ (if not found)</li>
<li>Operating System Cache
↓ (if not found)</li>
<li>Router Cache
↓ (if not found)</li>
<li>ISP's DNS Resolver
↓ (if not found)</li>
<li>Root Name Servers
↓</li>
<li>TLD Name Servers (.com)
↓</li>
<li>Authoritative Name Server (medium.com)
↓</li>
<li>IP Address Returned
Where Blocking Happens:
Step 4 - ISP's DNS Resolver
The ISP intercepts at this point and either:</li>
</ol>
</blockquote>

<ul>
<li>Returns false information</li>
<li>Drops the request</li>
<li>Redirects to a block page</li>
</ul>

<p>Testing DNS Blocking:<br>
Windows Command Prompt:<br>
<code>nslookup medium.com 8.8.8.8<br>
nslookup medium.com (your ISP's DNS)</code></p>

<p>Compare the results. If ISP's DNS fails but Google's succeeds, you've confirmed DNS blocking.<br>
Linux/Mac Terminal:<br>
<code>dig medium.com @8.8.8.8<br>
dig medium.com</code><br>
Encrypted DNS Solutions:<br>
DNS over HTTPS (DoH):<br>
Encrypts DNS queries, preventing ISP snooping.<br>
Enable in browsers:</p>

<ul>
<li>Firefox: Settings → Network Settings → Enable DNS over HTTPS</li>
<li>Chrome: Settings → Privacy and security → Security → Use secure DNS</li>
</ul>

<p>DNS over TLS (DoT):<br>
Similar to DoH but uses TLS protocol.<br>
Benefit: ISPs can't see which domains you're querying.<br>
Alternative Platforms<br>
While fighting for Medium access, consider these alternatives:<br>
For Writing:</p>

<ul>
<li>Dev.to - Developer-focused, similar community features</li>
<li>Hashnode - Tech blogging with custom domains</li>
<li>Substack - Newsletter-focused, monetization built-in</li>
<li>Ghost - Self-hosted, full control</li>
</ul>

<p>For Reading:</p>

<ul>
<li>Pocket - Save articles for offline reading</li>
<li>Feedly - RSS reader for following multiple sources</li>
<li>Flipboard - Curated content aggregator</li>
</ul>

<p>The Future: What's Coming?<br>
Trends to Watch:</p>

<ol>
<li>Increased Encryption:
More websites using HTTPS, encrypted DNS, making blocking harder.</li>
<li>Decentralized Platforms:
Blockchain-based content platforms resistant to censorship.</li>
<li>Legal Challenges:
Courts examining the constitutionality of blocking orders.</li>
<li>International Pressure:
Global organizations advocating for internet freedom in India.</li>
<li>Technical Arms Race:
As blocking methods evolve, so do bypass techniques.</li>
</ol>

<p><strong>Taking Action</strong><br>
<em>What You Can Do:</em></p>

<ol>
<li>Educate Yourself:
Understand how the internet works, your rights, and available tools.</li>
<li>Use Technical Solutions:
Implement DNS changes, VPNs, or Tor as needed.</li>
<li>Support Advocacy Groups:
Organizations like Internet Freedom Foundation, SFLC.in fight for digital rights.</li>
<li>Document Issues:
Report blocking to platforms like AccessNow, keep records.</li>
<li>Spread Awareness:
Share information about blocking and solutions with others.</li>
<li>Legal Action:
File RTI requests, approach courts if your rights are violated.</li>
</ol>

<p><em>Conclusion</em><br>
Medium's inaccessibility in India isn't a bug—it's a feature of the current internet governance system. DNS blocking is a deliberate choice made by authorities, implemented by ISPs, and suffered by millions of users.<br>
But knowledge is power. Understanding why Medium won't load is the first step. Learning how DNS works, what blocking methods are used, and how to bypass them empowers you to reclaim your internet freedom.<br>
The solutions exist. Whether it's switching to public DNS servers (easiest), using a VPN (most comprehensive), or exploring Tor (most private), you have options.<br>
More importantly, this issue highlights the urgent need for transparent, proportionate, and accountable internet governance in India. The internet should be open, accessible, and free—not fragmented by arbitrary blocking.<br>
Remember: Accessing information is your right. Don't let DNS manipulation take that away.</p>

<p>Quick Reference Card<br>
Problem: Medium won't load in India<br>
Cause: DNS blocking by ISPs under government orders<br>
Fastest Solution: Change DNS to 1.1.1.1 and 1.0.0.1<br>
Most Secure Solution: Use a reputable VPN<br>
Alternative: Access via Scribe.rip or archive sites<br>
Long-term: Advocate for internet freedom and transparency</p>

<p>Resources</p>

<p>Internet Freedom Foundation: <a href="https://internetfreedom.in" rel="noopener noreferrer">https://internetfreedom.in</a><br>
Cloudflare DNS Setup: <a href="https://1.1.1.1/dns/" rel="noopener noreferrer">https://1.1.1.1/dns/</a><br>
ProtonVPN (Free): <a href="https://protonvpn.com" rel="noopener noreferrer">https://protonvpn.com</a><br>
Tor Browser: <a href="https://www.torproject.org" rel="noopener noreferrer">https://www.torproject.org</a><br>
DNS Leak Test: <a href="https://dnsleaktest.com" rel="noopener noreferrer">https://dnsleaktest.com</a></p>

