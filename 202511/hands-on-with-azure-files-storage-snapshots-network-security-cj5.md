---
Title: Hands-On with Azure Files: Storage, Snapshots & Network Security
Description: 
Author: Samuel Ojo
Date: 2025-11-11T21:58:26.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Introduction:</strong><br>
In this lab, I set up a secure Azure Files environment tailored for a finance department. The goal was to create a premium storage account, configure file shares and directories, enable snapshots for recovery, and restrict access using virtual networks. This walkthrough is ideal for anyone looking to build enterprise-grade file storage with layered security and recovery options.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxo9mteslmuxmer6qd5kp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxo9mteslmuxmer6qd5kp.png" alt="Architectural Diagram" width="800" height="656"></a></p>

<p><strong>Skilling tasks</strong></p>

<ul>
<li>Create a storage account specifically for file shares.</li>
<li>Configure a file share and directory.</li>
<li>Configure snapshots and practice restoring files.</li>
<li>Restrict access to a specific virtual network and subnet.</li>
</ul>

<p><strong>Create and configure a storage account for Azure Files.</strong></p>

<p><strong>1. Create a storage account for the finance department’s shared files.</strong> </p>

<ul>
<li>In the portal, search for and select <code>Storage accounts</code>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe49ue179fir8zwh1kopy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe49ue179fir8zwh1kopy.png" alt="Search Storage accounts" width="800" height="449"></a></p>

<ul>
<li>Select + <strong>Create</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgej3xdq8tvchonzbg7cj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgej3xdq8tvchonzbg7cj.png" alt=" " width="800" height="449"></a></p>

<ul>
<li>For <strong>Resource group</strong> select <strong>Create new</strong>. Give your resource group a <strong>name</strong> and select <strong>OK</strong> to save your changes.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5el4bnm5lvrx52o0eugx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5el4bnm5lvrx52o0eugx.png" alt="New Resource group" width="800" height="448"></a></p>

<ul>
<li><p>Provide a <strong>Storage account name</strong>. Ensure the name meets the naming requirements.</p></li>
<li><p>Set the <strong>Performance</strong> to <strong>Premium</strong>.</p></li>
<li><p>Set the <strong>Premium account type</strong> to <strong>File shares</strong>.</p></li>
<li><p>Set the <strong>Redundancy</strong> to <strong>Zone-redundant storage</strong>.</p></li>
<li><p>Select <strong>Review</strong> and then <strong>Create</strong> the storage account.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwuojkg2gvyrpkftgpepb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwuojkg2gvyrpkftgpepb.png" alt="Review + Create" width="800" height="449"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnmgdr6rgfyuf99lc99ce.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnmgdr6rgfyuf99lc99ce.png" alt="Create" width="800" height="449"></a></p>

<ul>
<li>Wait for the resource to deploy.</li>
<li>Select <strong>Go to resource</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsb7tpbbygoqfxgoji4sl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsb7tpbbygoqfxgoji4sl.png" alt="Go to resource" width="800" height="449"></a></p>

<p><strong>Create and configure a file share with directory.</strong></p>

<ol>
<li>Create a file share for the corporate office.</li>
</ol>

<ul>
<li>In the storage account, in the <strong>Data storage</strong> section, select the <strong>File shares</strong> blade.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fty4vpihki2u6nxl6m725.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fty4vpihki2u6nxl6m725.png" alt="File shares" width="800" height="448"></a></p>

<ul>
<li>Select <strong>+ File share</strong> and provide a <strong>Name</strong>.</li>
<li>Review the other options, but take the defaults.</li>
<li>Select <strong>Create</strong>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flgtin05deklxir1kz250.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flgtin05deklxir1kz250.png" alt="Create File shares" width="800" height="451"></a></p>

<p><strong>2. Add a directory to the file share for the finance department. For future testing, upload a file.</strong></p>

<ul>
<li>Select your file share and select <strong>+ Add directory</strong>.</li>
<li>Name the new directory <code>finance</code>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq93gmsbt1ggeh3ol2buu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq93gmsbt1ggeh3ol2buu.png" alt="finance directory" width="800" height="450"></a></p>

<ul>
<li>Select <strong>Browse</strong> and then select the <strong>finance</strong> directory.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wfihakbfxml0ecx9iht.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wfihakbfxml0ecx9iht.png" alt="Browse finance directory" width="800" height="450"></a></p>

<ul>
<li>Notice you can <strong>Add directory</strong> to further organize your file share.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feyqddj5b4boou7qrhcat.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feyqddj5b4boou7qrhcat.png" alt="Add directory" width="800" height="450"></a></p>

<ul>
<li>
<strong>Upload</strong> a file of your choosing.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpd85t9lb95oay4mxlne7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpd85t9lb95oay4mxlne7.png" alt="Upload File" width="800" height="450"></a></p>

<p><strong>Configure and test snapshots.</strong></p>

<p><strong>1. Similar to blob storage, you need to protect against accidental deletion of files. You decide to use snapshots.</strong> </p>

<ul>
<li>Select your file share.</li>
<li>In the <strong>Operations</strong> section, select the <strong>Snapshots</strong> blade.</li>
<li>Select <strong>+ Add snapshot</strong>. The comment is optional. Select <strong>OK</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fieqlyvkq9kaligi11ku7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fieqlyvkq9kaligi11ku7.png" alt="Add snapshot" width="800" height="449"></a></p>

<ul>
<li>Select your snapshot and verify your file directory and uploaded file are included.</li>
</ul>

<ol>
<li>Practice using snapshots to restore a file.</li>
</ol>

<ul>
<li>Return to your <strong>file share</strong>.</li>
<li>
<strong>Browse</strong> to your file directory.</li>
<li>Locate your uploaded file and in the <strong>Properties</strong> pane select <strong>Delete</strong>. Select <strong>Yes</strong> to confirm the deletion.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fayfda2obr4xzh4ypznl0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fayfda2obr4xzh4ypznl0.png" alt="delete file" width="800" height="450"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fugs28rwt7iljq6ijr3it.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fugs28rwt7iljq6ijr3it.png" alt="Confirm deleted file" width="800" height="448"></a></p>

<ul>
<li>Select the <strong>Snapshots</strong> blade and then select your snapshot.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fltc9mu7pv8349fl4o70g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fltc9mu7pv8349fl4o70g.png" alt="Select Snapshots" width="800" height="449"></a></p>

<ul>
<li>Navigate to the file you want to restore,</li>
<li>Select the file and the select <strong>Restore</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdxzo5cuz0jksszf6yobx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdxzo5cuz0jksszf6yobx.png" alt="Restore" width="800" height="450"></a></p>

<ul>
<li>Provide a <strong>Restored file name</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsf16c7wxxe9jw4z5zouk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsf16c7wxxe9jw4z5zouk.png" alt="Restored file" width="800" height="451"></a></p>

<ul>
<li>Verify your file directory has the restored file.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7asvu8robt1giae6bibr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7asvu8robt1giae6bibr.png" alt="Verified Restored file" width="800" height="449"></a></p>

<p><strong>Configure restricting storage access to selected virtual networks.</strong></p>

<ol>
<li>This tasks in this section require a virtual network with subnet. In a production environment these resources would already be created.</li>
</ol>

<p><strong>Search for and select Virtual networks.</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnhwyristvck6yir78fnx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnhwyristvck6yir78fnx.png" alt="Search Virtual networks" width="800" height="451"></a></p>

<ul>
<li>Select <strong>Create</strong>. Select your resource group. and give the virtual network a <strong>name</strong>.</li>
<li>Take the defaults for other parameters, select <strong>Review + create</strong>, and then <strong>Create</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ucfu4zjkfc9fhofwjje.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ucfu4zjkfc9fhofwjje.png" alt="Create Virtual networks" width="800" height="448"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmxc2nv6ookm342bwxzli.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmxc2nv6ookm342bwxzli.png" alt="Review + create" width="800" height="449"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmbosb3rmjfur11aoq72x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmbosb3rmjfur11aoq72x.png" alt="create" width="800" height="449"></a></p>

<ul>
<li>Wait for the resource to deploy.</li>
<li>Select <strong>Go to resource</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs0znetvg4x2oo9e68srv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs0znetvg4x2oo9e68srv.png" alt="Go to resource" width="800" height="451"></a></p>

<ul>
<li>In the <strong>Settings</strong> section, select the <strong>Subnets</strong> blade.</li>
<li>Select the <strong>default</strong> subnet.</li>
<li>In the <strong>Service endpoints</strong> section choose <strong>Microsoft.Storage</strong> in the <strong>Services</strong> drop-down.</li>
<li>Do not make any other changes.</li>
<li>Be sure to <strong>Save</strong> your changes.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh0x2hhcci4g9cwpdffan.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh0x2hhcci4g9cwpdffan.png" alt="Subnet" width="800" height="450"></a></p>

<p><strong>2. The storage account should only be accessed from the virtual network you just created.</strong></p>

<ul>
<li>Return to your <strong>files storage account</strong>.</li>
<li>In the <strong>Security + networking</strong> section, select the <strong>Networking</strong> blade.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faksbixedkbw5on2wd8eg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faksbixedkbw5on2wd8eg.png" alt="files storage account" width="800" height="451"></a></p>

<ul>
<li>Change the <strong>Public network access</strong> to <strong>Enabled from selected virtual networks and IP addresses</strong>.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgeqcqktzgroavsknb2n9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgeqcqktzgroavsknb2n9.png" alt="Public network access" width="800" height="450"></a></p>

<ul>
<li>In the <strong>Virtual networks</strong> section, select <strong>Add existing virtual network</strong>.</li>
<li>Select your virtual network and subnet, select <strong>Add</strong>.</li>
<li>Be sure to <strong>Save</strong> your changes.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdf8lkpkke0o5hvp1cqqy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdf8lkpkke0o5hvp1cqqy.png" alt="Virtual networks" width="800" height="451"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsp977meq2it76ggmqqdp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsp977meq2it76ggmqqdp.png" alt="Save Virtual networks" width="800" height="450"></a></p>

<ul>
<li>Select the <strong>Storage browser</strong> and navigate to your file share.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwjiywzs57q0k07o7nf8y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwjiywzs57q0k07o7nf8y.png" alt="Storage browser" width="800" height="450"></a></p>

<ul>
<li>Verify the message not authorized to perform this operation. You are not connecting from the virtual network.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpixt5o6ekku6yl4hfkio.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpixt5o6ekku6yl4hfkio.png" alt=" " width="800" height="452"></a></p>

<p> 🧠 <strong>Key Terms Explained for Beginners</strong><br>
If you're new to Azure, here are some important terms used in this lab and what they mean:</p>

<p><strong>Azure Portal:</strong> The web-based dashboard where you manage all your Azure services. Think of it as your cloud control center.</p>

<p><strong>Resource Group:</strong> A container that holds related Azure resources like storage accounts, virtual networks, and more. It helps you organize and manage them together.</p>

<p><strong>Storage Account:</strong> A secure space in Azure where you store data—files, blobs, queues, and tables. It’s the foundation for using Azure Files.</p>

<p><strong>Azure Files:</strong> A cloud-based file sharing service that works like a traditional file server. You can access it using standard file protocols.</p>

<p><strong>File Share:</strong> A folder-like structure inside Azure Files where you store and organize files. You can create directories within it.</p>

<p><strong>Directory:</strong> A subfolder within a file share. In this lab, we created one called finance to organize departmental files.</p>

<p><strong>Snapshot:</strong> A read-only backup of your file share at a specific point in time. It’s useful for restoring deleted or changed files.</p>

<p><strong>Restore:</strong> The process of bringing back a deleted or previous version of a file using a snapshot.</p>

<p><strong>Virtual Network (VNet):</strong> A private network in Azure that lets your resources communicate securely. It’s like your own cloud-based LAN.</p>

<p><strong>Subnet:</strong> A smaller segment within a virtual network that helps organize and isolate resources.</p>

<p><strong>Service Endpoint:</strong> A way to securely connect your virtual network to Azure services like Storage without going over the public internet.</p>

<p><strong>Zone-Redundant Storage (ZRS):</strong> A storage option that keeps your data safe by replicating it across multiple zones in a region.</p>

<p><strong>Premium Performance Tier:</strong> A high-speed storage option optimized for low-latency and high-throughput workloads.</p>

<p><strong>Public Network Access:</strong> A setting that controls whether your storage account can be accessed from the internet or only from specific networks.</p>

<p><strong>✅ Conclusion</strong><br>
This lab covered the full lifecycle of setting up Azure Files for secure departmental use. From premium storage configuration to snapshots and network restrictions, each step reinforced best practices for enterprise-grade file sharing. These skills are directly applicable to production environments where data protection and access control are critical.</p>

<p><strong><em>Thanks for reading — see you in the next one</em></strong></p>

