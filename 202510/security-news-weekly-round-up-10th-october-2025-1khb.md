---
Title: Security news weekly round-up - 10th October 2025
Description: 
Author: Habdul Hazeez
Date: 2025-10-10T22:08:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>Do not get bored when I tell you that you are about to read another security review that's mostly about malware and vulnerabilities. Don't get frustrated because these threats are real and you and I need to be informed about them at all times.</p>

<p>With that out of the way, the vulnerabilities affect cloud and network systems. Meanwhile, the malware that we'll discuss can affect both mobile and desktop devices. In the mix, we have something related to cryptography and an attack against Google's Gemini.</p>

<p>That's it for the introduction. Let's begin.</p>




<h2>
  
  
  <a href="https://thehackernews.com/2025/10/detour-dog-caught-running-dns-powered.html" rel="noopener noreferrer">Detour Dog Caught Running DNS-Powered Malware Factory for Strela Stealer</a>
</h2>

<p>It's one thing to run a campaign without getting caught. However, it all changes when security researchers put a name to your activities because now they know: someone is behind these related activities. That's what happened in this situation.</p>

<p>The first question that you'll ask is: What is Strela Stealer? <a href="https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/a-deep-dive-into-strela-stealer-and-how-it-targets-european-countries/" rel="noopener noreferrer">Trustwave defines it as follows</a>:</p>

<blockquote>
<p>"The Strela Stealer (rus. Cтрела, lit. 'Arrow') is an infostealer that exfiltrates email log-in credentials and has been in the wild since late 2022. Strela Stealer is a precisely focused malware, targeting two email clients — Mozilla Thunderbird and Microsoft Outlook — on systems located in chosen European countries."</p>
</blockquote>

<p>Detour Dog comes into the picture because they are distributing it by compromising WordPress websites and embedding malicious JavaScript that uses "DNS TXT records as a communication channel for a traffic distribution system."</p>

<h2>
  
  
  <a href="https://www.bleepingcomputer.com/news/security/signal-adds-new-cryptographic-defense-against-quantum-attacks/" rel="noopener noreferrer">Signal adds new cryptographic defense against quantum attacks</a>
</h2>

<p>I will file this approach from Signal under <em><a href="https://pentera.io/glossary/proactive-security-cybersecurity-strategy/" rel="noopener noreferrer">proactive security</a></em> measures. Meaning, they now know the threat can materialize in the future, and they are deploying defenses to counter it when they do. Now, what are quantum attacks?</p>

<p><a href="https://identitymanagementinstitute.org/cybersecurity-quantum-attack/" rel="noopener noreferrer">Identity management institute defines quantum attack as</a> "the risk that quantum computers may be used to attack traditional cryptographic schemes." Signal uses classical public-key cryptography (e.g. elliptic-curve key exchange) in its encryption protocol, which is precisely the kind of scheme threatened by quantum attacks. However, Signal is also deploying hybrid post-quantum mechanisms (<a href="https://signal.org/docs/specifications/pqxdh/" rel="noopener noreferrer">PQXDH</a>) to mitigate that risk.</p>

<h2>
  
  
  <a href="https://www.welivesecurity.com/en/malware/threats-lurking-pdf-files/" rel="noopener noreferrer">Beware of threats lurking in booby-trapped PDF files</a>
</h2>

<p>What are the threats? You might ask. As detailed in the article, they include embedded scripts, hidden or malicious links, and exploitation of vulnerabilities in PDF readers.</p>

<p>The following is advice and the key takeaway from the article:</p>

<blockquote>
<p>The use of PDFs for malicious ends is also a reminder that security threats typically don’t arrive wrapped in suspicious-looking files. The tried-and-true rule applies here, too: Treat every unexpected link and attachment with caution and rely on trusted tools to protect your data and devices.</p>
</blockquote>

<h2>
  
  
  <a href="https://www.securityweek.com/critical-vulnerability-puts-60000-redis-servers-at-risk-of-exploitation/" rel="noopener noreferrer">Critical Vulnerability Puts 60,000 Redis Servers at Risk of Exploitation</a>
</h2>

<p>The vulnerability is CVE-2025-49844, or RediShell, <a href="https://www.wiz.io/blog/wiz-research-redis-rce-cve-2025-49844" rel="noopener noreferrer">according to Wiz</a>. At the time of writing, there are two good news about the vulnerability. First, some versions of Redis are available with patches for the vulnerability and second, there is no evidence of exploitation in the wild.</p>

<p>Nonetheless, why should you care, 60k exposed servers are a lot. Also, if attackers can exploit the vulnerability, it can lead to <a href="https://www.cloudflare.com/learning/security/what-is-remote-code-execution/" rel="noopener noreferrer">code execution</a>.</p>

<p>From the article:</p>

<blockquote>
<p>RediShell (CVE-2025-49844) represents a critical security vulnerability that affects all Redis versions due to its root cause in the underlying Lua interpreter. With hundreds of thousands of exposed instances worldwide, this vulnerability poses a significant threat to organizations across all industries.</p>
</blockquote>

<h2>
  
  
  <a href="https://www.bleepingcomputer.com/news/security/google-wont-fix-new-ascii-smuggling-attack-in-gemini/" rel="noopener noreferrer">Google won’t fix new ASCII smuggling attack in Gemini</a>
</h2>

<p><a href="https://www.firetail.ai/blog/ghosts-in-the-machine-ascii-smuggling-across-various-llms" rel="noopener noreferrer">According to FireTail</a>, an ASCII smuggling attack "is a technique rooted in the abuse of the Unicode standard, specifically utilizing invisible control characters to embed hidden instructions within a seemingly benign string of text." At the time of writing, this attack can work against Google's Gemini and it looks like Google is not going to fix it, yet.</p>

<p>From the article:</p>

<blockquote>
<p>The researcher reported the findings to Google on September 18 but the tech giant dismissed the issue as not being a security bug and may only be exploited in the context of social engineering attacks. Even so, Markopoulos* showed that the attack can trick Gemini into supplying false information to users.</p>
</blockquote>

<p><small>*Markopoulos is the name of the security researcher behind this research.</small></p>

<h2>
  
  
  <a href="https://www.securityweek.com/the-y2k38-bug-is-a-vulnerability-not-just-a-date-problem-researchers-warn/" rel="noopener noreferrer">The Y2K38 Bug Is a Vulnerability, Not Just a Date Problem, Researchers</a>
</h2>

<p>The Y2K38 bug can cause computers to malfunction on January 19, 2038. I know, that's like 13 years into the future since we are in 2025. But, there is a problem that makes it relevant today just as it can be relevant in 13 years; attackers can trigger it today.</p>

<p>While reading the article (and you should read it!), note the following because it is scary when you think of this bug:</p>

<blockquote>
<p>The researcher has confirmed the impact of Y2K38 on cars, routers, printers, smart TVs, alarms and other physical security systems, smartwatches, and ebook readers. He believes highly critical assets such as nuclear submarines, satellites, telecoms systems, power plants, water facilities, missile systems, planes, and trains could be impacted as well.</p>
</blockquote>

<h2>
  
  
  <a href="https://thehackernews.com/2025/10/new-clayrat-spyware-targets-android.html" rel="noopener noreferrer">New ClayRat Spyware Targets Android Users via Fake WhatsApp and TikTok Apps</a>
</h2>

<p>It's a sneaky campaign, I would say. You can get infected under the guise that you are downloading and installing a popular application like WhatsApp ot TikTok. Also, the ClayRat spyware can self-propagate by sending malicious links to every contact in your phone book.</p>

<p>Furthermore, here is why it's even scarier when you are infected:</p>

<blockquote>
<p>Once active, the spyware can exfiltrate SMS messages, call logs, notifications, and device information; taking photos with the front camera; and even send SMS messages or place calls directly from the victim's device.</p>
</blockquote>

<h2>
  
  
  <strong>Credits</strong>
</h2>

<p>Cover photo by <a href="https://unsplash.com/@hudsoncrafted" rel="noopener noreferrer">Debby Hudson on Unsplash</a>.</p>




<p>That's it for this week, and I'll see you next time.</p>

