---
Title: Headless Raspberry Pi Homelab – Part 2: Network Setup & Device Verification w/ Xfinity & Netgear
Description: 
Author: astromodem
Date: 2025-12-28T21:51:11.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Project Overview
</h2>

<p>This section documents the process of integrating a headless Raspberry Pi into a home network using an Xfinity router and a Netgear switch. The objective is to verify physical connectivity, switch-level visibility, DHCP assignment, and overall network reachability in preparation for remote access and database deployment. Each step reinforces foundational networking concepts aligned with the OSI Model, specifically Layers 1 (Physical), 2 (Data Link), and 3 (Network).</p>

<ul>
<li>Layer 1 (Physical) - Cabling and power</li>
<li>Layer 2 (Data Link) – Switch configuration, switch port status, and MAC address visibility</li>
<li>Layer 3 (Network) - IP Addressing and ICMP reachability</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwjuhfnprvhlmhlvczogd.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwjuhfnprvhlmhlvczogd.jpeg" alt="Hardware used to configure the Raspberry Pi" width="800" height="548"></a></p>

<h3>
  
  
  Hardware Used
</h3>

<ul>
<li><p><a href="https://www.amazon.com/dp/B07TYK4RL8?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1&amp;th=1" rel="noopener noreferrer">Raspberry Pi 4 Kit</a></p></li>
<li><p><a href="https://www.amazon.com/dp/B07PJ7XZ7X?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1&amp;th=1" rel="noopener noreferrer">Netgear 5-Port Smart Managed Switch</a></p></li>
</ul>

<h3>
  
  
  Step 1: Connect an Ethernet cable to the router.
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftb8vv4zygm7t2ywwqur0.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftb8vv4zygm7t2ywwqur0.jpeg" alt="Ethernet cable connected to router" width="768" height="1024"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg64f69wq5rhnxfjltevc.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg64f69wq5rhnxfjltevc.jpeg" alt="Layout of the Ethernet cable to the router" width="768" height="1024"></a></p>

<h3>
  
  
  Step 2: Set up the Netgear switch
</h3>

<ol>
<li>Connect the switch to its power supply</li>
<li>Connect the Ethernet cable from the router to one of the ports on the switch</li>
<li>Verify that the switch is powered on and the port that the Ethernet cable is connected to is illuminated to confirm its <strong>up</strong> status.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5t987jrb72c0v2nmo4ah.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5t987jrb72c0v2nmo4ah.jpeg" alt="Netgear switch connected to the router via ethernet" width="800" height="1231"></a></p>

<h3>
  
  
  Step 3: Verify that the switch is recognized on the network (Xfinity Network)
</h3>

<ol>
<li><p>Open the Xfinity Admin Tool through your browser and enter your credentials to log in<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb0fw2460lbs7cir3wwgo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb0fw2460lbs7cir3wwgo.png" alt="Xfinity Admin Tool home page" width="800" height="500"></a></p></li>
<li><p>Once logged in, select the <strong>Connected Devices</strong> located on the left-hand side of the page</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flm1f4ls0fl3shm2ik99f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flm1f4ls0fl3shm2ik99f.png" alt="Xfinity menu options" width="462" height="1512"></a></p>

<ol>
<li>The router identified the switch and automatically assigned an IP Address through DHCP (<strong>Dynamic Host Configuration Protocol</strong>) and identified the MAC address through ARP (<strong>Address Resolution Protocol</strong>). In Step 2, the switch was connected to the router via Ethernet, and that connection status was established and verified.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F08bu1493qkrgx3al9di7.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F08bu1493qkrgx3al9di7.jpeg" alt="Netgear switch identified on the network" width="800" height="165"></a></p>

<h3>
  
  
  Step 4: Install the Netgear Discovery Tool
</h3>

<p>Download and install the Netgear Discovery Tool for your operating system</p>

<p><a href="https://www.netgear.com/support/product/netgear-discovery-tool" rel="noopener noreferrer">Official Netgear Discovery Tool</a></p>

<h3>
  
  
  Step 5: Start Searching
</h3>

<h4>
  
  
  Homepage
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8e65pvn8uyc92y8x84dh.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8e65pvn8uyc92y8x84dh.jpeg" alt="Netgear Discovery Tool homepage" width="800" height="500"></a></p>

<h4>
  
  
  Searching process
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7vjfubwrpiezbzmwzgbm.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7vjfubwrpiezbzmwzgbm.jpeg" alt="Netgear Discovery Tool searching process" width="800" height="500"></a></p>

<h4>
  
  
  Switch discovered
</h4>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp661t5v9uvkedtxyu08q.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp661t5v9uvkedtxyu08q.jpeg" alt="Netgear Discovery Tool" width="800" height="495"></a></p>

<h3>
  
  
  Step 6: Log in to the Netgear Admin page
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flm8zfxz7d889i5ulzfqo.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flm8zfxz7d889i5ulzfqo.jpeg" alt="Netgear Admin Page login" width="800" height="471"></a></p>

<h3>
  
  
  Step 7: Verifying switch port status
</h3>

<p>Once logged in, select <strong>Port Status</strong> located on the left</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fufmjx7te5stwkz29m6p4.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fufmjx7te5stwkz29m6p4.jpeg" alt="Netgear Port Status" width="800" height="1294"></a></p>

<p>The Ethernet cable connected from the router to the switch port in step 2 has been identified by the Netgear Admin page, verifying that the port status is up.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2n7dznuzpiog5kr0xnnt.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2n7dznuzpiog5kr0xnnt.jpeg" alt="Netgear port status" width="800" height="119"></a></p>

<h3>
  
  
  Step 8: Integrate the Raspberry Pi into the Switch
</h3>

<ol>
<li>Connect the power source to the Raspberry Pi</li>
<li>Connect an Ethernet cable to one of the ports located on the switch</li>
<li>Connect the Ethernet cable from the switch to the port of the Raspberry Pi</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F81gags5lr51084bbho1u.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F81gags5lr51084bbho1u.jpeg" alt="Raspberry Pi integration" width="726" height="1080"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft8j961l0fns0x04gv418.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft8j961l0fns0x04gv418.jpeg" alt="Raspberry Pi integration" width="768" height="1024"></a></p>

<h3>
  
  
  Step 9: Verify that the Raspberry Pi is recognized on the network (Xfinity Network)
</h3>

<ul>
<li>Repeat Step 3 and verify that the Raspberry Pi's hostname is populated in the <strong>Connected Devices</strong>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwjvxe2vrp23575b4r25a.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwjvxe2vrp23575b4r25a.jpeg" alt="Raspberry Pi hostname" width="800" height="99"></a></p>

<h3>
  
  
  Step 10: Verify that the Raspberry Pi is recognized on the switch (Netgear)
</h3>

<ul>
<li>Repeat Step 7 and verify that the Ethernet cable is connected between the Raspberry Pi and the switch, and the connected port is showing an <strong>up</strong> status</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftqmmggrdhegvy062vwo3.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftqmmggrdhegvy062vwo3.jpeg" width="800" height="211"></a></p>

<h3>
  
  
  Step 11: Ping the Raspberry Pi
</h3>

<p>The ping command verifies that the Raspberry Pi is reachable within the local area network, and the Pi replies with ICMP (Internet Control Message Protocol) echo reply packets.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftwdzxxry9j8bt0ww5e48.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftwdzxxry9j8bt0ww5e48.jpeg" alt="Performing the ping command on the Raspberry Pi" width="800" height="453"></a></p>

<p>This completes the physical connectivity and network visibility of the Raspberry Pi and the switch to the local area network.</p>

<h3>
  
  
  Next Steps (Part 3)
</h3>

<ul>
<li>Demonstrate how to SSH into the Raspberry Pi's Linux operating system</li>
<li>Update and upgrade the Linux packages</li>
<li>Perform the installation and provisioning of MySQL</li>
</ul>

