---
Title: Setting Up Minecraft Server in AWS
Description: 
Author: Carmen
Date: 2025-08-26T22:06:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hosting your own Minecraft server is one of the best ways to enjoy the game with friends, explore custom worlds, or add plugins that make the experience unique. In this guide, we’ll walk through how to set up a fully functional Minecraft server on <strong>Amazon Web Services (AWS)</strong> using <strong>EC2</strong>. By the end, you’ll have a server that starts automatically, uses a static IP so players can always connect, and can be customized with plugins or new worlds, all while keeping costs under control.</p>

<h2>
  
  
  Step 1: Set up the Server (EC2 Instance)
</h2>

<ol>
<li>
<strong>Log in</strong> to <a href="https://aws.amazon.com/console/" rel="noopener noreferrer">AWS Management Console</a>.</li>
<li>
<p><strong>Choose a region</strong>: In the top-right corner, select the AWS region closest to your players.</p>

<blockquote>
<p><em>Example: If most players are on the US West Coast, select one of the us-west regions.</em></p>
</blockquote>
</li>
<li><p><strong>Open the EC2 service:</strong> Search for EC2 in the console search bar and open the dashboard.</p></li>
<li><p><strong>Launch a new instance:</strong> Click on Launch instance.</p></li>
<li>
<p><strong>Configure the instance:</strong></p>

<ol>
<li>Give the instance a descriptive name, such as “<em>Minecraft Server</em>”.</li>
<li>Choose <strong>Amazon Linux 2023 AMI.</strong>
</li>
<li>Set the architecture to <strong>64-bit (Arm).</strong>
</li>
</ol>


</li>

<li>

<p><strong>Select the instance type:</strong></p>

<ol>
<li>Start with <em>t4g.small</em>, which is enough for a basic Minecraft server (it’s fine for a few players, but laggy with mods or &gt;5 players. If you expect more, go to <em>t4g.medium</em> or larger).</li>
<li>If you need more performance later, you can <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html" rel="noopener noreferrer">resize/upgrade the instance</a>.</li>
<li>For compute-intensive workloads, consider using the <em>C family</em> (Compute Optimized) instances.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff6cx5nk89jsmv6dya2s8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff6cx5nk89jsmv6dya2s8.png" alt=" " width="800" height="450"></a></p>


</li>

<li>

<p><strong>Create a Key Pair:</strong></p>

<ol>
<li>Under <strong>Key pair name</strong>, click <strong>Create a new key pair</strong>.</li>
<li>Enter a name for the key (e.g.<em>, minecraft-key</em>).</li>
<li>Click <strong>Create a key pair.</strong>
</li>
<li>The private key file (<code>.pem</code>) will be automatically downloaded to your computer. Keep this file safe (you will need it later if you want to connect to the server).</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3j8ghdrm03li0642etf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3j8ghdrm03li0642etf.png" alt=" " width="800" height="450"></a></p>


</li>

<li>

<p><strong>Configure Network Settings:</strong></p>

<ol>
<li>Scroll down to the <strong>Network settings</strong> section and click <strong>Edit.</strong>
</li>
<li>Select the <strong>default VPC</strong>. Leave the subnet as <em>No preference.</em>
</li>
<li>Make sure <strong>Auto-assign public IP</strong> is <strong>enabled.</strong>
</li>
<li>Under <strong>Firewall (security groups):</strong>

<ol>
<li>Choose <strong>Create security group.</strong>
</li>
<li>Name it something like <em>Minecraft Server SG.</em>
</li>
<li>(Optional) Add a description such as: <em>Security group for Minecraft Server. Ports open for SSH (22) and Minecraft TCP (25565).</em>
</li>
</ol>


</li>

<li>

<p><strong>Add inbound rules:</strong></p>

<ol>
<li>
<p><strong>SSH access (EC2 Instance Connect):</strong></p>

<ol>
<li>By default, SSH (port 22) is enabled, but you must restrict it to the correct IP range for EC2 Instance Connect in you AWS region.</li>
<li>Find the correct CIDR range in the <a href="https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html#aws-ip-download" rel="noopener noreferrer">AWS IP address ranges documentation</a>.</li>
<li>For example, in the <code>eu-west-3</code> (Paris) region, the IP CIDR range is: <code>35.180.112.80/29</code> .</li>
<li>
<p>In <strong>Inbound rules</strong>, edit the SSH rule and set <strong>Source Custom</strong>, then paste the correct IP range for your region.</p>

<blockquote>
<p>💡 <em>Tip: To find the IP CIDR, download the JSON file from the documentation, open it in your browser as raw data, and search (Control+F) for <code>EC2_INSTANCE_CONNECT</code>.</em></p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftrjhv0npsk3ht2pf4v26.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftrjhv0npsk3ht2pf4v26.png" alt=" " width="800" height="450"></a></p>
</li>
</ol>


</li>

<li>

<p><strong>Minecraft traffic:</strong> Add another inbound rule:</p>

<ol>
<li>
<strong>Type:</strong> <em>Custom TCP</em>.</li>
<li>
<strong>Port Range:</strong> <em>25565</em>.</li>
<li>
<strong>Source:</strong> <em>0.0.0.0/0</em> (to allow players to connect).</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faorch2b5h575f19zreq1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faorch2b5h575f19zreq1.png" alt=" " width="800" height="450"></a></p>


</li>

</ol>

</li>

</ol>

</li>

<li>

<p><strong>Add Storage:</strong></p>

<ol>
<li>By default your instance comes with an <strong>8 GB gp3 root volume</strong>, which is enough for a small server.</li>
<li>If you expect more data (e.g., larger worlds, mods, or backups), you may increase the size here.</li>
<li>AWS Free Tier includes up to <strong>30GB</strong> of Elastic Block Storage (EBS), so you can expand the volume without leaving the free tier (as long as you stay under the 30GB).</li>
</ol>


</li>

</ol>

<h2>
  
  
  Step 2: Set up Minecraft Automatically
</h2>

<ol>
<li>
<strong>Add a startup script:</strong> We’ll use a bash script so that the Minecraft server is automatically downloaded, configured and started every time the EC2 instance launches.</li>
<li>
<p><strong>Locate the User Data field:</strong></p>

<ol>
<li>Scroll down to the <strong>Advanced Details</strong> section.</li>
<li>Expand it, then scroll to the bottom to find the <strong>User data</strong> field.</li>
<li>
<p>Copy and paste the following script into that field:<br>
</p>
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>
<span class="nb">set</span> <span class="nt">-euo</span> pipefail

<span class="c"># ===== CONFIG =====</span>
<span class="c"># Replace with the latest server .jar URL from the official page</span>
<span class="nv">MINECRAFT_URL</span><span class="o">=</span><span class="s2">"https://piston-data.mojang.com/v1/objects/6bce4ef400e4efaa63a13d5e6f6b500be969ef81/server.jar"</span>
<span class="nv">MINECRAFT_DIR</span><span class="o">=</span><span class="s2">"/opt/minecraft/server"</span>
<span class="nv">MINECRAFT_USER</span><span class="o">=</span><span class="s2">"minecraft"</span>
<span class="nv">SERVICE_NAME</span><span class="o">=</span><span class="s2">"minecraft"</span>

<span class="c"># ===== PREREQS =====</span>
<span class="c"># Install Java (Amazon Corretto)</span>
<span class="nb">sudo </span>yum <span class="nb">install</span> <span class="nt">-y</span> java-21-amazon-corretto-headless

<span class="c"># Create a dedicated user and directories</span>
<span class="nb">id</span> <span class="nt">-u</span> <span class="s2">"</span><span class="nv">$MC_USER</span><span class="s2">"</span> <span class="o">&gt;</span>/dev/null 2&gt;&amp;1 <span class="o">||</span> useradd <span class="nt">--system</span> <span class="nt">--create-home</span> <span class="nt">--shell</span> /sbin/nologin <span class="s2">"</span><span class="nv">$MC_USER</span><span class="s2">"</span>
<span class="nb">mkdir</span> <span class="nt">-p</span> <span class="s2">"</span><span class="nv">$MC_DIR</span><span class="s2">"</span>
<span class="nb">cd</span> <span class="s2">"</span><span class="nv">$MC_DIR</span><span class="s2">"</span>

<span class="c"># ===== DOWNLOAD SERVER =====</span>
<span class="c"># Download the Minecraft server JAR from the URL</span>
<span class="c"># (Overwrites existing server.jar if present)</span>
wget <span class="nt">-O</span> server.jar <span class="s2">"</span><span class="nv">$MINECRAFT_URL</span><span class="s2">"</span>

<span class="c"># ===== FIRST BOOT TO GENERATE FILES =====</span>
<span class="c"># Run once to create eula.txt and default configs (ignore non-zero exit)</span>
java <span class="nt">-Xmx1024M</span> <span class="nt">-Xms1024M</span> <span class="nt">-jar</span> server.jar nogui <span class="o">||</span> <span class="nb">true</span>

<span class="c"># Accept EULA</span>
<span class="nb">sed</span> <span class="nt">-i</span> <span class="s1">'s/false/true/p'</span> eula.txt

<span class="c"># ===== CALCULATE HEAP SIZE FROM TOTAL RAM =====</span>
<span class="c"># Use ~50% of total memory, clamp to [1024, 16384] MB, round down to nearest 256 MB</span>
<span class="nv">TOTAL_MEM_MB</span><span class="o">=</span><span class="si">$(</span><span class="nb">awk</span> <span class="s1">'/MemTotal/ {printf "%.0f", $2/1024}'</span> /proc/meminfo<span class="si">)</span>
<span class="nv">XMX_MB</span><span class="o">=</span><span class="k">$((</span> TOTAL_MEM_MB <span class="o">/</span> <span class="m">2</span> <span class="k">))</span>
<span class="k">if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$XMX_MB</span><span class="s2">"</span> <span class="nt">-lt</span> 1024 <span class="o">]</span><span class="p">;</span> <span class="k">then </span><span class="nv">XMX_MB</span><span class="o">=</span>1024<span class="p">;</span> <span class="k">fi
if</span> <span class="o">[</span> <span class="s2">"</span><span class="nv">$XMX_MB</span><span class="s2">"</span> <span class="nt">-gt</span> 16384 <span class="o">]</span><span class="p">;</span> <span class="k">then </span><span class="nv">XMX_MB</span><span class="o">=</span>16384<span class="p">;</span> <span class="k">fi
</span><span class="nv">XMX_MB</span><span class="o">=</span><span class="k">$((</span> <span class="o">(</span>XMX_MB/256<span class="o">)*</span><span class="m">256</span> <span class="k">))</span>

<span class="c"># ===== START/STOP SCRIPTS =====</span>
<span class="c"># Create start script</span>
<span class="nb">cat</span> <span class="o">&gt;</span> start <span class="o">&lt;&lt;</span><span class="sh">'</span><span class="no">EOF</span><span class="sh">'
#!/bin/bash
set -euo pipefail

MC_DIR="/opt/minecraft/server"
cd "</span><span class="nv">$MC_DIR</span><span class="sh">"

# Recompute heap on each start (handles instance type changes)
TOTAL_MEM_MB=</span><span class="si">$(</span><span class="nb">awk</span> <span class="s1">'/MemTotal/ {printf "%.0f", $2/1024}'</span> /proc/meminfo<span class="si">)</span><span class="sh">
XMX_MB=</span><span class="k">$((</span> TOTAL_MEM_MB <span class="o">/</span> <span class="m">2</span> <span class="k">))</span><span class="sh">
if [ "</span><span class="nv">$XMX_MB</span><span class="sh">" -lt 1024 ]; then XMX_MB=1024; fi
if [ "</span><span class="nv">$XMX_MB</span><span class="sh">" -gt 16384 ]; then XMX_MB=16384; fi
XMX_MB=</span><span class="k">$((</span> <span class="o">(</span>XMX_MB/256<span class="o">)*</span><span class="m">256</span> <span class="k">))</span><span class="sh">

ulimit -n 4096
exec java -Xmx</span><span class="k">${</span><span class="nv">XMX_MB</span><span class="k">}</span><span class="sh">M -Xms</span><span class="k">${</span><span class="nv">XMX_MB</span><span class="k">}</span><span class="sh">M -jar server.jar nogui
</span><span class="no">EOF
</span><span class="nb">chmod</span> +x start

<span class="c"># Create stop script</span>
<span class="nb">cat</span> <span class="o">&gt;</span> stop <span class="o">&lt;&lt;</span><span class="sh">'</span><span class="no">EOF</span><span class="sh">'
#!/bin/bash
# Graceful stop via systemd is preferred: systemctl stop minecraft
# Fallback: kill Java owned by the minecraft user running the server.jar
pkill -u minecraft -f 'java .*server</span><span class="se">\.</span><span class="sh">jar' || true
</span><span class="no">EOF
</span><span class="nb">chmod</span> +x stop

<span class="c"># Permissions</span>
<span class="nb">chown</span> <span class="nt">-R</span> <span class="s2">"</span><span class="nv">$MC_USER</span><span class="s2">"</span>:<span class="s2">"</span><span class="nv">$MC_USER</span><span class="s2">"</span> /opt/minecraft

<span class="c"># ===== SYSTEMD SERVICE =====</span>
<span class="c"># Create a systemd service to run Minecraft on startup</span>
<span class="nb">cat</span> <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">' &gt; /etc/systemd/system/minecraft.service
[Unit]
Description=Minecraft Server
Wants=network-online.target
After=network-online.target

[Service]
User=</span><span class="k">${</span><span class="nv">MC_USER</span><span class="k">}</span><span class="sh">
WorkingDirectory=</span><span class="k">${</span><span class="nv">MC_DIR</span><span class="k">}</span><span class="sh">
ExecStart=</span><span class="k">${</span><span class="nv">MC_DIR</span><span class="k">}</span><span class="sh">/start
ExecStop=</span><span class="k">${</span><span class="nv">MC_DIR</span><span class="k">}</span><span class="sh">/stop
Restart=on-failure
RestartSec=15
NoNewPrivileges=true
ProtectSystem=full
ProtectHome=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
</span><span class="no">EOF

</span><span class="c"># Enable and start the service</span>
systemctl daemon-reload
systemctl <span class="nb">enable</span> <span class="k">${</span><span class="nv">SERVICE_NAME</span><span class="k">}</span>.service
systemctl start <span class="k">${</span><span class="nv">SERVICE_NAME</span><span class="k">}</span>.service
</code></pre>


<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyr2mvll0g9opfvzd5k5n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyr2mvll0g9opfvzd5k5n.png" alt=" " width="800" height="450"></a></p>
</li>
</ol>


</li>

<li>

<p><strong>Get the latest Minecraft server link:</strong></p>

<ol>
<li>To ensure you’re running the newest version, go to the <a href="https://www.minecraft.net/en-us/download/server" rel="noopener noreferrer">official Minecraft Server download page</a>.</li>
<li>Find the link to “<em>Download minecraft_server.jar</em>”.</li>
<li>Right-click the <em>minecraft_server.jar</em> link and copy the URL.</li>
<li>
<p>Replace the value of <code>MINECRAFT_URL</code> in the script above with this new link.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6pr8c4vtsqbrfuz4o0n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6pr8c4vtsqbrfuz4o0n.png" alt=" " width="800" height="509"></a></p>
</li>
</ol>


</li>

<li>

<p><strong>Launch the instance:</strong></p>

<ol>
<li>Once you’ve added the script, click <strong>Launch Instance.</strong>
</li>
<li>
<p>The EC2 instance may take 5-10 minutes on first boot while it installs dependencies and sets up the server.</p>

<blockquote>
<p>🧪 <em>Test: After the instance finishes launching, copy its public IPv4 address from the EC2 console. Open Minecraft, go to Multiplayer, add a new server, and paste the IP address. Within a few minutes, you should be able to connect to your new Minecraft server.</em></p>
</blockquote>
</li>
</ol>


</li>

</ol>

<h2>
  
  
  Step 3: Set Up Static IP
</h2>

<p>Every time an EC2 instance turns off, its public IP address will change. Having to search for your new IP every time you turn your server on and off is inefficient. We will associate an Elastic IP with our EC2 instance to solve this problem. Elastic IP addresses are dedicated IP addresses that do not change and can be attached to different resources.</p>

<ol>
<li>
<p><strong>Open Elastic IPs:</strong></p>

<ol>
<li>In the EC2 dashboard, expand the left-hand menu.</li>
<li>
<p>Under <strong>Network &amp; Security</strong>, click <strong>Elastic IPs.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdik69lar1odiw72upwn8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdik69lar1odiw72upwn8.png" alt=" " width="800" height="450"></a></p>
</li>
</ol>


</li>

<li>

<p><strong>Allocate an Elastic IP:</strong></p>

<ol>
<li>Click <strong>Allocate Elastic IP address</strong> (top right).</li>
<li>Choose <strong>Public IPv4</strong> from Amazon’s pool of IPv4 addresses.</li>
<li>
<p>Click <strong>Allocate.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffrjdkxszgsrbr4uapauf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffrjdkxszgsrbr4uapauf.png" alt=" " width="800" height="450"></a></p>
</li>
</ol>


</li>

<li>

<p><strong>Name the IP:</strong> To keep things organized, rename the Elastic IP to something descriptive, like <em>Minecraft Server IP</em>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqrxv6cqy6ni2wphmmvp0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqrxv6cqy6ni2wphmmvp0.png" alt=" " width="800" height="450"></a></p>


</li>

<li>

<p><strong>Associate the Elastic IP with the instance:</strong></p>

<ol>
<li>Select the new Elastic IP.</li>
<li>
<p>Click <strong>Actions</strong> → <strong>Associate Elastic IP Address.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9fl3ryq16x2htih5fq6k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9fl3ryq16x2htih5fq6k.png" alt=" " width="800" height="450"></a></p>
</li>
<li><p>For <strong>Resource type</strong>, choose <strong>Instance.</strong></p></li>
<li><p>From the drop-down list, select the Minecraft EC2 instance we created before.</p></li>
<li><p>Click <strong>Associate.</strong></p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp3ylkno1rf7toipg3far.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp3ylkno1rf7toipg3far.png" alt=" " width="800" height="450"></a></p>


</li>

<li>

<p><strong>Verify the Elastic IP:</strong></p>

<ol>
<li>Go back to the Minecraft EC2 instance details.</li>
<li>Under <strong>Public IPv4 address</strong>, we should now see the Elastic IP.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqt0fdcdhvjx7s603pi3t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqt0fdcdhvjx7s603pi3t.png" alt=" " width="800" height="450"></a></p>


</li>

<li>

<p><strong>Connect to the server in Minecraft:</strong></p>

<ol>
<li>Open Minecraft and go to the <strong>Multiplayer screen.</strong>
</li>
<li>Click <strong>Add Server</strong> (or use <strong>Direct Connect).</strong>
</li>
<li>In the <strong>Server Address</strong> field, enter the Elastic IP.</li>
<li>The server should now be accessible with the static IP every time you restart the instance.</li>
</ol>


</li>

</ol>

<blockquote>
<p>⚠️ <em>Warning: Elastic IPs are free **as long as they are attached to a running instance</em><em>. If you allocate one but don’t associate it with anything, AWS may charge you a small hourly fee.</em></p>
</blockquote>

<h2>
  
  
  <strong>Extra: Customize the Server (Plugins, World Files, etc.)</strong>
</h2>

<p>Once the server is up and running, you may want to install plugins, change the world, or modify configuration files. To do that, you’ll need to connect directly to your EC2 instance.</p>

<ol>
<li>
<p><strong>Connect to the instance:</strong></p>

<ol>
<li>Go to the EC2 dashboard and select the Minecraft server instance.</li>
<li>
<p>Click <strong>Connect</strong> at the top.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fklcd8ayttgwqey9yc6y5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fklcd8ayttgwqey9yc6y5.png" alt=" " width="800" height="450"></a></p>
</li>
<li><p>In the connection screen, make sure the user is set to <code>ec2-user</code>, then click <strong>Connect</strong> again.</p></li>
<li>
<p>If the connection fails, try rebooting the instance and waiting a few minutes, or review the <a href="https://repost.aws/knowledge-center/ec2-instance-connect-troubleshooting" rel="noopener noreferrer">common troubleshooting tips.</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjgahzgkkpz1o949g7l1t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjgahzgkkpz1o949g7l1t.png" alt=" " width="800" height="450"></a></p>
</li>
</ol>


</li>

<li>

<p><strong>Navigate to the server directory:</strong></p>

<ol>
<li>
<p>After logging in, you’ll be in a terminal. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvq9gg4o81jrjkzakk3tx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvq9gg4o81jrjkzakk3tx.png" alt=" " width="800" height="450"></a></p>
</li>
<li><p>Run: <code>cd /opt/minecraft/server/</code>. This directory contains all the Minecraft server files. Remember this location for future edits.</p></li>
<li><p>To check the server files, use the following command: <code>ls</code> . You should see files like <em>world/</em>, <em>logs/</em>, <em>server.jar</em>, as well as the <em>start</em> and <em>stop scripts.</em></p></li>
</ol>


</li>

<li>

<p><strong>Stop the server before making changes:</strong></p>

<ol>
<li>The server console runs in the background, so files can’t be edited while it’s active.</li>
<li>To stop the server, run: <code>sudo systemctl stop minecraft</code> .</li>
<li>Once the service is stopped, you can upload a new world, add plugins or edit configuration files.</li>
</ol>


</li>

<li>

<p><strong>Restart the server:</strong></p>

<ol>
<li>After making changes, restart the instance or run: <code>sudo systemctl start minecraft</code> .</li>
</ol>


</li>

</ol>

<blockquote>
<p>💡 Useful commands:</p>


<pre class="highlight shell"><code><span class="c"># Start the server</span>
<span class="nb">sudo </span>systemctl start minecraft

<span class="c"># Stop the server</span>
<span class="nb">sudo </span>systemctl stop minecraft

<span class="c"># Restart the server</span>
<span class="nb">sudo </span>systemctl restart minecraft

<span class="c"># Check if the server is running</span>
<span class="nb">sudo </span>systemctl status minecraft
</code></pre>

</blockquote>

<h2>
  
  
  Clean Up (Avoid Extra Charges)
</h2>

<p>When you’re done testing or hosting your server, it’s important to shut down resources to prevent ongoing AWS charges.</p>

<ol>
<li>
<strong>Terminate the EC2 instance:</strong>

<ol>
<li>In the EC2 dashboard, select your Minecraft instance.</li>
<li>Click <strong>Instance state → Terminate instance.</strong>
</li>
<li>Confirm the termination.</li>
</ol>


</li>

<li>

<strong>Release the Elastic IP:</strong>

<ol>
<li>In the EC2 dashboard, open the sidebar and go to <strong>Elastic IPs.</strong>
</li>
<li>Select the Elastic IP you associated with your instance.</li>
<li>Click <strong>Actions → Release Elastic IP addresses.</strong>
</li>
</ol>


</li>

</ol>

<p>Once done, you will no longer incur charges from this setup.</p>




<h2>
  
  
  Resources &amp; Useful Links
</h2>

<ul>
<li><a href="https://medium.com/@pzawadz/how-to-set-up-minecraft-server-java-edition-for-free-for-12-months-on-aws-38239753cca1" rel="noopener noreferrer">https://medium.com/@pzawadz/how-to-set-up-minecraft-server-java-edition-for-free-for-12-months-on-aws-38239753cca1</a></li>
<li><a href="https://aws.amazon.com/blogs/gametech/setting-up-a-minecraft-java-server-on-amazon-ec2/" rel="noopener noreferrer">https://aws.amazon.com/blogs/gametech/setting-up-a-minecraft-java-server-on-amazon-ec2/</a></li>
<li><a href="https://aws.amazon.com/free/" rel="noopener noreferrer">https://aws.amazon.com/free/</a></li>
<li><a href="https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server" rel="noopener noreferrer">https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server</a></li>
</ul>

