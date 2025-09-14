---
Title: Securing Azure Workloads with Azure Firewall: A Step-by-Step Implementation Guide
Description: 
Author: Isaiah Izibili
Date: 2025-09-14T22:07:59.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>As cloud applications scale, so do the threats they face. From unauthorized access to data exfiltration, the need for centralized, intelligent network security becomes non-negotiable. Enter Azure Firewall—a robust, cloud-native solution that offers deep packet inspection, application-level filtering, and threat intelligence.</p>

<p>In this guide, we walk through the deployment and configuration of Azure Firewall to protect an application virtual network (app-vnet). Each step is explained with its technical purpose and strategic value, so you can build not just a secure network—but a resilient one.</p>

<h2>
  
  
  Scenario Overview
</h2>

<p>Your organization is preparing for increased application usage and continuous integration via Azure DevOps. To meet these demands securely, you’ve identified the following requirements:</p>

<ul>
<li>Deploy Azure Firewall in app-vnet for centralized security.</li>
<li>Create a Firewall Policy to manage access rules.</li>
<li>Add an Application Rule to allow access to Azure DevOps.</li>
<li>Add a Network Rule to enable DNS resolution.</li>
</ul>

<h2>
  
  
  Step 1: Create Azure Firewall subnet in our existing virtual network
</h2>

<ol>
<li>In the search box at the top of the portal, enter Virtual networks. Select Virtual networks in the search results.</li>
<li>Select app-vnet.</li>
<li>Select Subnets.</li>
<li>Select + Subnet.</li>
<li>Enter the following information and select Save.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>**Property                      Value**
Name                                AzureFirewallSubnet
Address range                       10.1.63.0/26
</code></pre>

</div>



<p>Note: Leave all other settings as default.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdy7c7umu6bx70nd7g9t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdy7c7umu6bx70nd7g9t.png" alt="Azurefirewall" width="800" height="366"></a></p>

<h2>
  
  
  Step 2: Create an Azure Firewall
</h2>

<ol>
<li>In the search box at the top of the portal, enter Firewall. Select Firewall in the search results.</li>
<li>Select + Create.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyeltq0jxur4ls1tou4d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyeltq0jxur4ls1tou4d.png" alt="firecreate" width="800" height="282"></a><br>
<em>3. Create a firewall by using the values in the following table. For any property that is not specified, use the default value.</em></p>

<blockquote>
<p>Note: Azure Firewall can take a few minutes to deploy.<br>
</p>


</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>**Property              Value**
Resource group              RG1
Name                        app-vnet-firewall
Firewall SKU                Standard
Firewall management     Use a Firewall Policy to manage this firewall
Firewall policy             select Add new
Policy name             fw-policy
Region                      West US 3
Policy Tier             Standard
Choose a virtual network     Use existing
Virtual network              app-vnet (RG1)
Public IP address        Add new: fwpip
Enable Firewall Management NIC  uncheck the box
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw4x0xhd67xq4dyetmqgn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw4x0xhd67xq4dyetmqgn.png" alt="policy 1" width="800" height="365"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyec1obcp10ge9eqs6k6c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyec1obcp10ge9eqs6k6c.png" alt="firewall2" width="800" height="365"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2arzmxwoj6jrzuhdu0i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2arzmxwoj6jrzuhdu0i.png" alt="fwpol" width="800" height="367"></a></p>

<p><em>4. Select Review + create and then select Create.</em></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbi03nbmtn20dgptevha.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbi03nbmtn20dgptevha.png" alt="Created" width="800" height="362"></a></p>

<p>Step 3: Configure the Firewall Policy</p>

<ol>
<li>In the portal, search for and select Firewall Policies.</li>
<li>Select <strong>fw-policy</strong>.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frqtiv4s8z89i7a9khido.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frqtiv4s8z89i7a9khido.png" alt="firewall polcie" width="800" height="382"></a></p>

<p><strong>Step 3a: Add an Application Rule</strong></p>

<ol>
<li>In the Settings blade, select Application rules and then Add a rule collection.</li>
<li>Configure the application rule collection and then select Add.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Property                   Value
Name                            app-vnet-fw-rule-collection
Rule collection type             Application
Priority                     200
Rule collection action           Allow
Rule collection group            DefaultApplicationRuleCollectionGroup
Name                             AllowAzurePipelines
Source type                  IP address
Source                           10.1.0.0/23
Protocol                     https
Destination type             FQDN
Destination                  dev.azure.com, azure.microsoft.com
</code></pre>

</div>



<blockquote>
<p>Note: The AllowAzurePipelines rule allows the web application to access Azure Pipelines. The rule allows the web application to access the Azure DevOps service and the Azure website.</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe5fmgyoizj0rtzpn1h5d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe5fmgyoizj0rtzpn1h5d.png" alt="applicationrule" width="800" height="368"></a></p>

<p>Why It Matters:<br>
<em>This rule allows your application servers to securely connect to Azure DevOps for continuous updates. Using FQDN filtering, Azure Firewall can inspect outbound traffic and allow only specific domains—critical for reducing exposure to malicious sites.</em></p>

<p><strong>Step 3b: Add a Network Rule</strong></p>

<ol>
<li>In the Settings blade, select Network rules and then Add a network collection.</li>
<li>Configure the network rule and then select Add.
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
Property                Value
Name                        app-vnet-fw-nrc-dns
Rule collection type        Network
Priority                200
Rule collection action      Allow
Rule collection group       DefaultNetworkRuleCollectionGroup
Rule                        AllowDns
Source                      10.1.0.0/23
Protocol                    UDP
Destination ports           53
Destination addresses       1.1.1.1, 1.0.0.1
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frzvt4ss9bnsbgjtosl84.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frzvt4ss9bnsbgjtosl84.png" alt="collectionrule" width="800" height="367"></a></p>

<blockquote>
<p>DNS is the backbone of internet communication. Without it, your servers can’t resolve domain names. This rule ensures that your workloads can reach Cloudflare’s DNS servers securely, enabling name resolution for outbound traffic.</p>
</blockquote>

<h2>
  
  
  Step 4: Verify Deployment Status
</h2>

<ol>
<li>In the portal search for and select Firewall.</li>
<li>View the app-vnet-firewall and ensure the Provisioning state is Succeeded. This may take a few minutes.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdzkd1jdpnuklzo6ml11h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdzkd1jdpnuklzo6ml11h.png" alt="provisioningstate" width="800" height="367"></a></p>

<ol>
<li>In the portal serach for and select Firewall policies.</li>
<li>View the fw-policy and ensure the Provisioning state is Succeeded. This may take a few minutes.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw59vlvi4onm4r3omyt6r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw59vlvi4onm4r3omyt6r.png" alt="wallpolicies" width="800" height="368"></a></p>

<blockquote>
<p>Successful provisioning confirms that your firewall is active and ready to enforce rules. This step ensures that your security posture is operational before workloads begin communicating.</p>
</blockquote>

<p>Congratulations on completing the exercise. Here are the main takeaways:</p>

<p><strong>Key takeaways</strong></p>

<ul>
<li>Azure Firewall is a cloud-based security service that protects your Azure virtual network resources from incoming and outgoing threats.</li>
<li>An Azure firewall policy is a resource that contains one or more collections of NAT, network, and application rules.</li>
<li>Network rules allow or deny traffic based on IP addresses, ports, and protocols.</li>
<li>Application rules allow or deny traffic based on fully qualified domain names (FQDNs), URLs, and HTTP/HTTPS protocols.</li>
</ul>

