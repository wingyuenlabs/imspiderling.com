---
Title: Automating Threat Intel: How I Built a Fast, Containerised IP Triage Tool
Description: 
Author: shyn
Date: 2026-01-17T22:01:47.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🛡️ The Mission: Fighting "Analyst Fatigue"
</h2>

<p>As an aspiring Security Engineer, I learned quickly that triage is where most time is lost. When a firewall flags 50 suspicious connections, checking them one-by-one in a browser is slow and prone to error.</p>

<p>I built Sentinel-IP to solve this. It’s a Python tool that takes a list of IPs and instantly enriches them with threat intelligence, turning a 30-minute manual task into a 30-second automated one.</p>

<h2>
  
  
  🛠️ The Tech Stack
</h2>

<ul>
<li>    Python: For the automation logic and API handling.</li>
<li>    Docker: To ensure the tool works on any machine (Mac, Windows, or Linux) without setup headaches.</li>
<li>    AbuseIPDB API: For crowdsourced reports on brute-force and spam activity.</li>
<li>    AlienVault OTX API: For "Pulse" data—identifying if an IP is linked to known malware campaigns.</li>
</ul>

<h2>
  
  
  💡 Why I Pivoted from VirusTotal
</h2>

<p>Originally, I planned to include VirusTotal. However, their free tier allows only 4 requests per minute. For 50 IPs, the tool would have taken nearly 15 minutes to run!</p>

<p>By switching to AlienVault OTX, I removed the bottleneck. OTX allows for much higher request volumes, enabling the tool to scan dozens of IPs in seconds. This pivot taught me a vital lesson in Security Engineering: The best data is useless if it arrives too late to stop the attack.</p>

<h2>
  
  
  💻 How It Works (The Code)
</h2>

<p>The tool uses a simple <code>ips.txt</code> file as input. It queries the APIs and generates a clean results.csv for the analyst to review.</p>

<p>The tool uses a simple ips.txt file as input. It queries the APIs and generates a clean results.csv for the analyst to review.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
# snippet of the core logic
for ip in tqdm(ips, desc="Analyzing"):
    abuse_score = check_abuse_ip(ip) # Returns % confidence
    otx_pulses = check_alienvault(ip) # Returns count of threat pulses

    results.append({
        'IP': ip,
        'Abuse_Score%': abuse_score,
        'OTX_Pulses': otx_pulses
    })
</code></pre>

</div>



<h2>
  
  
  🔍 Real-World Use Cases
</h2>

<ol>
<li>The Firewall Log "Dump"</li>
</ol>

<p>Scenario: A company firewall blocks hundreds of failed SSH attempts. Application: Copy the IPs from the logs into Sentinel-IP. Impact: You can instantly filter for IPs with a 100% Abuse Score. Instead of investigating every block, you focus your energy on the verified botnets.</p>

<ol>
<li>Phishing Header Analysis</li>
</ol>

<p>Scenario: A suspicious email is reported. You find a "Source IP" in the email header. Application: Run that IP through the tool. Impact: If AlienVault OTX shows 5 Pulses related to "Credential Harvesting," you have immediate proof that the email is malicious and can purge it from the network.</p>

<h2>
  
  
  📈 Learning Outcomes
</h2>

<p>Building this project wasn't just about code; it was about understanding the SOC ecosystem:</p>

<ul>
<li> API Resilience: Handling 404 Not Found errors (which often mean an IP is "Clean") versus 401 Unauthorized errors.</li>
<li>    Containerization: Using Docker volumes to allow a container to write a CSV file directly to my Mac's desktop.</li>
<li>    Data Correlation: Understanding that an IP with a high Abuse Score and multiple OTX Pulses is a "Critical" threat that requires immediate blocking.</li>
</ul>




<h2>
  
  
  🔗 Check out the project on GitHub
</h2>

<p>If you found this tool helpful, feel free to check out the full source code and contribute to the project over on GitHub:</p>

<p>👉 <strong><a href="https://github.com/shynsec/Sentinel-IP" rel="noopener noreferrer">Sentinel-IP Repository</a></strong></p>

<p>Don't forget to ⭐️ the repo if you like what you see!</p>

