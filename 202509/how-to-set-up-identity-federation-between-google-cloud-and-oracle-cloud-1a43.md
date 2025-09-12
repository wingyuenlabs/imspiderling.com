---
Title: How to set up Identity Federation between Google Cloud and Oracle Cloud
Description: 
Author: Faris Durrani
Date: 2025-09-12T21:38:51.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>How to set up Identity SAML Federation between Google Cloud Platform (GCP) and Oracle Cloud Infrastructure (OCI)</em></p>

<p>Setting up Identity Federation will allow users to log into OCI using their GCP IAM organization credentials, rather than logging in using a new username-password in OCI. This can improve credential management and security by having a central place to store all user logins.</p>

<p>We will replicate the steps in <a href="https://docs.oracle.com/en-us/iaas/Content/database-at-gcp/oagcp-task-6-federation.htm" rel="noopener noreferrer">Oracle Docs: Task 6: Set Up Identity Federation (Optional)</a> with some UI updates. This is an optional succession to the <a href="https://dev.to/farisdurrani/how-to-create-an-oracle-autonomous-databasegoogle-cloud-380o">How to create an Oracle Autonomous Database@Google Cloud</a> article we wrote.</p>

<h2>
  
  
  Prerequisites
</h2>

<ol>
<li>Have an OCI account</li>
<li>Have a GCP account and an associating GCP Workspace IAM Admin console at <a href="https://admin.google.com" rel="noopener noreferrer">https://admin.google.com</a>. Note that this requires you having and associating a private  DNS domain (e.g. example.com) to the Google Cloud Admin console</li>
</ol>

<h2>
  
  
  1. Create groups in GCP Admin Console
</h2>

<p>While not strictly necessary, this helps to confirm the JIT (Just-in-time) provisioning works as intended. You can create any group but we recommend at least one group name already present in OCI and GCP ("odbg-db-family-administrators") and one other only present in GCP ("example-group415") for POC purposes.</p>

<p>We create a number of groups in the GCP Admin console: <a href="https://admin.google.com/u/1/ac/groups" rel="noopener noreferrer">https://admin.google.com/u/1/ac/groups</a>. If you came from the Oracle Database@Google Cloud article, we created the groups for Autonomous Database access based on <a href="https://docs.oracle.com/en-us/iaas/Content/database-at-gcp/oagcp-task-5-rbac.htm" rel="noopener noreferrer">Oracle Docs: Task 5: Set Up Role Based Access Control</a>:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F34rdzmk31skq3uy1rkci.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F34rdzmk31skq3uy1rkci.png" alt="GCP groups" width="800" height="507"></a></p>

<h2>
  
  
  2. Create a new custom SAML app in GCP
</h2>

<p>Go to <a href="https://admin.google.com/u/1/ac/apps/unified" rel="noopener noreferrer">https://admin.google.com/u/1/ac/apps/unified</a> and click on <strong>Add app</strong> &gt; <strong>Add custom SAML app</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fte5fsky9lcgef411ngv9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fte5fsky9lcgef411ngv9.png" alt="Add custom SAML app" width="800" height="510"></a></p>

<p>We added the following app details:</p>

<ul>
<li>
<strong>App name:</strong> OracleCloudFederation</li>
<li>
<strong>Description:</strong> Configures identity federation between Google Cloud and Oracle Cloud for Oracle Database@Google Cloud use.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6c2z8ej8d60hvx6jakea.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6c2z8ej8d60hvx6jakea.png" alt="App name" width="800" height="454"></a></p>

<p>Click CONTINUE.</p>

<p>In the next page, <strong>Download Metadata</strong>. Leave this page open for now.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbkgvn6qzox4di99otnnz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbkgvn6qzox4di99otnnz.png" alt="Download metadata" width="800" height="451"></a></p>

<h2>
  
  
  3. Add SAML IdP in OCI
</h2>

<p>Log into your OCI account. Go to <strong>☰ Menu</strong> &gt; <strong>Identity &amp; Security</strong> &gt; <strong>Domains</strong> &gt; <strong>Default</strong> (or another domain) &gt; <strong>Federation</strong> &gt; <strong>Actions</strong> &gt; <strong>Add SAML IdP</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foqonbg4khykar2t4wf8g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foqonbg4khykar2t4wf8g.png" alt="Add SAML IdP" width="800" height="481"></a></p>

<p>We give the SAML identity provider a random name:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fix1nr5jobsho5iq50jin.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fix1nr5jobsho5iq50jin.png" alt="SAML name" width="800" height="629"></a></p>

<p>Import the downloaded IdP metadata from GCP:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdq6knljjfb8lbbgmyhd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdq6knljjfb8lbbgmyhd.png" alt="Import IdP metadata" width="800" height="723"></a></p>

<p>Leave this page open. </p>

<h2>
  
  
  4. Add SAML IdP in GCP
</h2>

<p>Click on <strong>Export SAML metadata</strong> to export the OCI SAML metadata details.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh6ibq8bxf4zno1nin6ec.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh6ibq8bxf4zno1nin6ec.png" alt="OCI SAML metadata" width="800" height="671"></a></p>

<p>Switch view from <strong>Metadata file</strong> to <strong>Manual export</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flehy5girtp2pi3r0z8u1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flehy5girtp2pi3r0z8u1.png" alt="Manual export" width="800" height="405"></a></p>

<p>Go back to the GCP Admin page. Click CONTINUE.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4y6y1md14ni45ye84xep.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4y6y1md14ni45ye84xep.png" alt="Continue GCP Admin" width="800" height="471"></a></p>

<p>Copy paste the OCI values into GCP like so:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>GCP</th>
<th>OCI</th>
</tr>
</thead>
<tbody>
<tr>
<td>ACS URL</td>
<td>Assertion consumer service URL</td>
</tr>
<tr>
<td>Entity ID</td>
<td>Provider ID</td>
</tr>
</tbody>
</table></div>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fan2q1p9r1yvztz2i0p7l.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fan2q1p9r1yvztz2i0p7l.png" alt="SAML Metadata direction" width="800" height="450"></a></p>

<p>Leave the <strong>Name ID</strong> details as their default.</p>

<p>Click CONTINUE.</p>

<h2>
  
  
  4. Add attribute mapping in GCP
</h2>

<p>In the next page, add the following attribute </p>

<ul>
<li>First name → FirstName</li>
<li>Last name → LastName</li>
<li>Primary email → PrimaryEmail</li>
</ul>

<p>Under the <strong>Group membership</strong> section, add all the groups of the users you want to be sent to OCI. Enter the <strong>App attribute</strong> <code>MemberOf</code>. Click FINISH.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fydwejhhz0vjpral7a2qf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fydwejhhz0vjpral7a2qf.png" alt="Attribute mapping" width="800" height="595"></a></p>

<h2>
  
  
  5. Turn on User access in GCP
</h2>

<p>In the next page showing the SAML app details, click on <strong>User access</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz4ih33675183a6xss5hk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz4ih33675183a6xss5hk.png" alt="Click User access" width="800" height="316"></a></p>

<p>Switch to <strong>ON for everyone</strong>. Click SAVE.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvgj6g5ess6daonn7a6px.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvgj6g5ess6daonn7a6px.png" alt="Switch on" width="800" height="305"></a></p>

<h2>
  
  
  6. Finish SAML app creation in OCI
</h2>

<p>Go back to the OCI page. We finished importing GCP's IdP metadata file. Click <strong>Next</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvonw7hdbgilcehyhr1vn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvonw7hdbgilcehyhr1vn.png" alt="Next" width="800" height="411"></a></p>

<p>Change <strong>Requested Name ID format</strong> to <strong>Email address</strong>. Click <strong>Next</strong> and <strong>Create IdP</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpdo424gvkvty2g61817g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpdo424gvkvty2g61817g.png" alt="Email ID format" width="800" height="459"></a></p>

<h2>
  
  
  7. Activate and add to IdP policy
</h2>

<p>Now we want to activate our newly-created IdP app and add it as an option to the login policy. Click the app.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgqge1028vbeeufn7zg8h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgqge1028vbeeufn7zg8h.png" alt="Click app" width="800" height="439"></a></p>

<p>Click on the <strong>Activate IdP</strong> button (under Actions or in the banner).</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcia8x7dsfj4bit1tn40s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcia8x7dsfj4bit1tn40s.png" alt="Activate IdP" width="800" height="358"></a></p>

<p>Next, we need to add the IdP app to the IdP policy. Go back to the <strong>Federation</strong> page and click on the <strong>Default Identity Provider Policy</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2mbrx6nao8r7dzcabz56.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2mbrx6nao8r7dzcabz56.png" alt="Click Default Identity Provider Policy" width="800" height="817"></a></p>

<p>Under the <strong>Identity provider rules</strong>, edit the first IdP rule.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk6n24b3ecq8j51gf1l76.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk6n24b3ecq8j51gf1l76.png" alt="Edit rule" width="800" height="348"></a></p>

<p>Add the newly-created IdP app to the list of <strong>Assign identity providers</strong>. Click <strong>Save changes</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F53fvv10og59qkv2b9901.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F53fvv10og59qkv2b9901.png" alt="Add app" width="800" height="462"></a></p>

<p>Now you can login but without any groups.</p>

<h2>
  
  
  8. Set up JIT
</h2>

<p>To enable the user's groups in GCP to appear in OCI and sync the user's group membership, we need to enable Just-in-time (JIT) provisioning so the group membership info can be shared from GCP to OCI.</p>

<p>Go back to the <strong>Federation</strong> page and click on the IdP app.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb51ewlf69e7c74tfatvc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb51ewlf69e7c74tfatvc.png" alt="Click app" width="800" height="512"></a></p>

<p>Click <strong>Actions</strong> &gt; <strong>Configure JIT</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2f4deep3qftrr1s4r9tg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2f4deep3qftrr1s4r9tg.png" alt="Configure JIT" width="800" height="361"></a></p>

<p>Enable these settings:</p>

<ul>
<li>Enable Just-In-Time (JIT) provisioning</li>
<li>Create a new identity domain user</li>
<li>Update the existing identity domain user</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff4ysqg57kg9ka5r7z8cm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff4ysqg57kg9ka5r7z8cm.png" alt="Enable settings" width="800" height="521"></a></p>

<p>Add these <strong>Map user attributes info</strong>:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>IdP user attribute type</th>
<th>IdP user attribute name</th>
<th>Maps to</th>
<th>Identity domain user attributes</th>
</tr>
</thead>
<tbody>
<tr>
<td>NameID</td>
<td>NameID value</td>
<td>→</td>
<td>User Name</td>
</tr>
<tr>
<td>Attribute</td>
<td>LastName</td>
<td>→</td>
<td>Last name</td>
</tr>
<tr>
<td>Attribute</td>
<td>PrimaryEmail</td>
<td>→</td>
<td>Primary Work Email</td>
</tr>
<tr>
<td>Attribute</td>
<td>FirstName</td>
<td>→</td>
<td>First name</td>
</tr>
</tbody>
</table></div>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3mbp2jx0q3r9ukecy8vq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3mbp2jx0q3r9ukecy8vq.png" alt="Map user attributes info" width="800" height="409"></a></p>

<p>Now, we assign group mappings. Enter the following values:</p>

<ul>
<li>
<strong>Group membership attribute name:</strong> <code>MemberOf</code>
</li>
<li>
<strong>Assign implicit group membership:</strong> Select this</li>
<li>
<strong>When assigning group membership...:</strong> Merge with existing group memberships</li>
<li>
<strong>When a group is not found...:</strong> Ignore the missing group</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxh3we99bwa3jbi49b929.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxh3we99bwa3jbi49b929.png" alt="Group mappings" width="800" height="575"></a></p>

<p>Click <strong>Update</strong>.</p>

<h2>
  
  
  9. Verify Federated login
</h2>

<p>We are done! Now, we need to test the OCI login using GCP credentials.</p>

<p>Log out of OCI as necessary. Log in to your OCI account at <a href="https://cloud.oracle.com" rel="noopener noreferrer">https://cloud.oracle.com</a>. You should see your GCP login option here.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fga3wfm19i099oh0qh1dk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fga3wfm19i099oh0qh1dk.png" alt="GCP Login option" width="800" height="906"></a></p>

<p>Log in using that button, which should lead you to your GCP login, and your successful OCI login.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwanwmj3hb3vw7m1hj1z1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwanwmj3hb3vw7m1hj1z1.png" alt="OCI login" width="800" height="765"></a></p>

<p><em>ℹ Troubleshoot: if the login is unsuccessful, check back the OCI JIT settings are correct.</em></p>

<p>We'll log out and log back in as the OCI admin so we can see all the groups and users added.</p>

<p>Here, we can see my GCP user added to the OCI list of users.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6xzv61am2s51r041ke45.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6xzv61am2s51r041ke45.png" alt="New user" width="800" height="677"></a></p>

<p>And I can see that new user is a member of the "odbg-db-family-administrators" group, which that user is in GCP. Note that while the user is also in the GCP group "example-group415", this group did not get transferred into OCI because that group was never created in OCI in the first place. The JIT only matches groups present in both OCI and GCP.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2ru4hi9tj22y82irugba.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2ru4hi9tj22y82irugba.png" alt="Groups" width="800" height="923"></a></p>

<p>You now have successfully created an identity federation from GCP to OCI.</p>

<h2>
  
  
  References
</h2>

<ol>
<li><a href="https://docs.oracle.com/en-us/iaas/Content/database-at-gcp/oagcp-task-6-federation.htm" rel="noopener noreferrer">Oracle Docs: Task 6: Set Up Identity Federation (Optional)</a></li>
</ol>

<h2>
  
  
  Safe harbor statement
</h2>

<p><em>The information provided on this channel/article/story is solely intended for informational purposes and cannot be used as a part of any contractual agreement. The content does not guarantee the delivery of any material, code, or functionality, and should not be the sole basis for making purchasing decisions. The postings on this site are my own and do not necessarily reflect the views or work of Oracle or Mythics, LLC.</em></p>

<p><em>This work is licensed under a <a href="http://creativecommons.org/licenses/by/4.0" rel="noopener noreferrer">Creative Commons Attribution 4.0 International License</a>.</em></p>

