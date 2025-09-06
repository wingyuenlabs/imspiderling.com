---
Title: Creating and Configuring Virtual Networks in Azure: A Step-by-Step Guide
Description: 
Author: Oladosu Ibrahim
Date: 2025-09-06T21:17:16.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <strong>Introduction</strong>
</h2>

<p>Every cloud application begins with a secure and scalable network foundation. In Azure, <strong>Virtual Networks (VNets)</strong> provide the backbone for connecting applications, databases, and security appliances while maintaining private, isolated environments.</p>

<p>In this guide, we’ll walk through how to:</p>

<ul>
<li>Create hub-and-spoke virtual networks for a web app.</li>
<li>Add subnets for frontend, backend, and firewall workloads.</li>
<li>Configure secure VNet peering to enable private communication.</li>
</ul>

<p>By the end, you’ll know how to design a simple but production-ready network architecture in Azure.</p>

<h2>
  
  
  <strong>Skilling Objectives</strong>
</h2>

<p>You will learn how to:</p>

<ul>
<li>Deploy virtual networks and subnets in Azure.</li>
<li>Set up a <strong>hub-and-spoke network topology</strong>.</li>
<li>Configure <strong>VNet peering</strong> for secure inter-network communication.</li>
</ul>

<h2>
  
  
  <strong>Architecture Overview</strong>
</h2>

<p>The hub-and-spoke model you created is a common <strong>Azure networking pattern</strong> designed for scalability, security, and manageability.</p>

<ul>
<li>The <strong>hub-vnet</strong> acts as the central control point. It hosts shared services such as the Azure Firewall, monitoring tools, or VPN gateways. By centralizing security here, you avoid duplicating firewall or gateway resources in every network.</li>
<li>
<p>The <strong>app-vnet</strong> serves as the spoke network. It contains two isolated subnets:</p>

<ul>
<li>The <strong>frontend subnet</strong>, which hosts the web servers that users interact with.</li>
<li>The <strong>backend subnet</strong>, which hosts databases or internal services that should never be exposed directly to the internet.</li>
</ul>


</li>

<li><p><strong>VNet peering</strong> creates a secure, private connection between the hub and spoke, allowing traffic to flow as if both networks were part of a single VNet, while still keeping administrative boundaries.</p></li>

</ul>

<p>This architecture ensures that application traffic flows privately through Azure’s backbone without touching the public internet, while also keeping workloads segmented for better security and compliance.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fni29qxj7w5e3hr9wa9eu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fni29qxj7w5e3hr9wa9eu.png" alt="Image17" width="800" height="352"></a></p>

<h2>
  
  
  <strong>Step 1: Create the App Virtual Network &amp; Subnets</strong>
</h2>

<p>💡 <strong>Why start here?</strong><br>
The application network (spoke) hosts the workload — web servers in the frontend subnet and databases in the backend subnet. Segmenting resources into subnets improves <strong>security</strong> and <strong>traffic control</strong>.</p>

<ol>
<li>Sign in to the <a href="https://portal.azure.com" rel="noopener noreferrer">Azure Portal</a>.</li>
<li><p>Search for <strong>Virtual Networks</strong> and click <strong>+ Create</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbt7jka4g1f3tg1l9bhzn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbt7jka4g1f3tg1l9bhzn.png" alt="Image1" width="800" height="771"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyncr6otvv0rnxno51x0k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyncr6otvv0rnxno51x0k.png" alt="Image2" width="800" height="593"></a></p></li>
<li><p>Select the existing resource group or create a new resource group e.g <code>RG1</code>.</p></li>
<li><p>Enter the virtual network name <code>app-vnet</code> and choose <strong>East US</strong> as the region.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzumedysbm38lkb4hrj80.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzumedysbm38lkb4hrj80.png" alt="Image3" width="800" height="580"></a></p></li>
<li><p>Define the IP address space as <code>10.1.0.0/16</code>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnyl0xagb3t0lsf5xs412.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnyl0xagb3t0lsf5xs412.png" alt="Image4" width="800" height="580"></a></p></li>
<li><p>Under subnets, create one named <strong>frontend</strong> with address range <code>10.1.0.0/24</code>, and another named <strong>backend</strong> with address range <code>10.1.1.0/24</code>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1dhyhc8zqa3h95dxkquf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1dhyhc8zqa3h95dxkquf.png" alt="Image5" width="800" height="377"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1xxx9iadtpd3hpn47f2g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1xxx9iadtpd3hpn47f2g.png" alt="Image6" width="800" height="376"></a></p></li>
<li><p>Click <strong>Review + create</strong>, then <strong>Create</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffk2la7143q9r8gumbc7n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffk2la7143q9r8gumbc7n.png" alt="Image7" width="800" height="755"></a></p></li>
</ol>

<p>✅ At this point, you have a dedicated application network ready for workloads.</p>

<h2>
  
  
  <strong>Step 2: Create the Hub Virtual Network</strong>
</h2>

<p>💡 <strong>Why a hub?</strong><br>
The hub is the central network that connects to external services and enforces <strong>security policies</strong> like firewalls, monitoring, or VPNs.</p>

<ol>
<li><p>Go back to <strong>Virtual Networks</strong> and click <strong>+ Create</strong> again.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjm46pijrd3aw0faus6w8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjm46pijrd3aw0faus6w8.png" alt="Image8" width="800" height="593"></a></p></li>
<li><p>Use the same resource group <code>RG1</code>.</p></li>
<li><p>Name this network <code>hub-vnet</code> and choose <strong>East US</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbt7515pnl7axoxis3y1x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbt7515pnl7axoxis3y1x.png" alt="Image9" width="800" height="756"></a></p></li>
<li><p>Assign the address space <code>10.0.0.0/16</code>.</p></li>
<li><p>Edit the default subnet/Add a subnet named <strong>AzureFirewallSubnet</strong> with the range <code>10.0.0.0/26</code>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F622urvx3vrnxv2uny1a7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F622urvx3vrnxv2uny1a7.png" alt="Image10" width="800" height="376"></a></p></li>
<li><p>Select <strong>Review + create</strong>, then <strong>Create</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0w13h0serzeg6hhoyakv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0w13h0serzeg6hhoyakv.png" alt="Image11" width="800" height="877"></a></p></li>
</ol>

<p>✅ You now have a hub network prepared to secure and route traffic.</p>

<h2>
  
  
  <strong>Step 3: Configure VNet Peering</strong>
</h2>

<p>💡 <strong>Why VNet peering?</strong><br>
Peering allows virtual networks to <strong>communicate privately</strong> without using the public internet. This ensures secure traffic flow between your application and hub services.</p>

<ol>
<li><p>Open <strong>app-vnet</strong> from the list of virtual networks.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbz49nb4lnq71t0g92rbf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbz49nb4lnq71t0g92rbf.png" alt="Image12" width="800" height="519"></a></p></li>
<li><p>Under <strong>Settings</strong>, select <strong>Peerings</strong>, then click <strong>+ Add</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9edddj7varl5gk6uikn3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9edddj7varl5gk6uikn3.png" alt="Image13" width="800" height="739"></a></p></li>
<li><p>Enter the remote peering link name as <code>app-vnet-to-hub</code>.</p></li>
<li><p>Select the virtual network <code>hub-vnet</code> as the peer.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi2esgytpk9moz3k2alx2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi2esgytpk9moz3k2alx2.png" alt="Image14" width="800" height="606"></a></p></li>
<li><p>For the local peering link name, enter <code>hub-to-app-vnet</code>.</p></li>
<li><p>Leave other settings at their defaults and click <strong>Add</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzicu2wkyxlcizjh5w10x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzicu2wkyxlcizjh5w10x.png" alt="Image15" width="800" height="410"></a></p></li>
</ol>

<p>✅Once deployment completes, confirm that the peering status shows <strong>Connected</strong>.</p>

<h2>
  
  
  Step 4: Verify the Architecture
</h2>

<p>💡 <strong>Why verify?</strong><br>
Testing ensures that connectivity works before deploying critical workloads.</p>

<ul>
<li>Open both <code>app-vnet</code> and <code>hub-vnet</code> in the portal to confirm their subnets exist.</li>
<li>Check that the <strong>peering connection</strong> shows as connected.
<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fds2jp48z1g8z02q7psx5.png" alt="Image16" width="800" height="376">
</li>
</ul>

<h2>
  
  
  <strong>Conclusion</strong>
</h2>

<p>In this hands-on guide, you’ve built the foundation for a secure and scalable Azure network:</p>

<ul>
<li>An <strong>app virtual network</strong> with separate frontend and backend subnets.</li>
<li>A <strong>hub virtual network</strong> with a firewall subnet.</li>
<li>
<strong>VNet peering</strong> to enable secure, private communication between the two.</li>
</ul>

<p>This hub-and-spoke design is widely used in Azure networking to keep applications isolated yet connected to centralized services.</p>

