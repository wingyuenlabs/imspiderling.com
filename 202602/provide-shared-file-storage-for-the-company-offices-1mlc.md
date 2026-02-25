---
Title: Provide shared file storage for the company offices
Description: 
Author: Precious Uchechukwu Nwafor
Date: 2026-02-25T22:09:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>​In a world where teams are geographically dispersed but need to stay perfectly in sync, local hardware just doesn't cut it anymore. Welcome to your roadmap for building a rock-solid Azure Files infrastructure; designed for high-speed collaboration, protected by instant snapshots, and locked down behind a private virtual network.</p>

<p>Today, we will be working with a company that is geographically dispersed with offices in different locations. The offices need a way to share files and disseminate information. For example, the Finance department needs to confirm cost information for auditing and compliance. These files should be easy to access and load without delay. Some content should only be accessed from selected corporate virtual networks.</p>

<p>Let's get to it!</p>

<h2>
  
  
  PHASE 1: Create a <strong>storage account</strong> for the <strong>finance department’s shared files</strong>
</h2>

<p><code>Action plan</code></p>

<p><strong>1</strong> : In the  Azure <strong>portal</strong>, search for and select <strong>Storage accounts</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F49vjxp5cen1w83id9ns6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F49vjxp5cen1w83id9ns6.png" alt=" Searching for storage account" width="800" height="126"></a></p>

<p><strong>2</strong> : Select <strong>+ Create</strong>.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6hcqammoeu538jualfeu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6hcqammoeu538jualfeu.png" alt="Initializing creation of storage accout" width="800" height="192"></a></p>

<p><strong>3</strong> : For  <strong>Resource group</strong>, select Create  <strong>new **. Give your resource group a  **name</strong> and select  <strong>OK ** to  **save</strong> your  <strong>changes</strong>. Proceed to provide a  <strong>Storage</strong> account  <strong>name</strong>. Set the  <strong>Performance</strong> to  <strong>Premium</strong>.  Set the  <strong>Premium account type</strong> to  <strong>File shares</strong>.  Set the  <strong>Redundancy</strong> to  <strong>Zone-redundant storage</strong>. Select  <strong>Review + Create</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsvllu3zf0su6kjl7slel.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsvllu3zf0su6kjl7slel.png" alt="unique identifiers" width="800" height="534"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F78u4lmpb8rs21ta28svp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F78u4lmpb8rs21ta28svp.png" alt="Validation in progress" width="800" height="260"></a></p>

<p><strong>4</strong> : Select  <strong>Create</strong>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F51mpm1ziilq59poc9b4r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F51mpm1ziilq59poc9b4r.png" alt="Creating storage" width="800" height="942"></a></p>

<p><strong>5</strong> : Wait for the  <strong>resource to deploy</strong>. Select  <strong>Go to resource</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F644ub3t8hqvtrizbxnze.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F644ub3t8hqvtrizbxnze.png" alt="Initializing deployment" width="800" height="188"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp0j1q99grfhn7zrqozvx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp0j1q99grfhn7zrqozvx.png" alt="Deployment in progress" width="800" height="268"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffn0v4x40wvy8ass38drb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffn0v4x40wvy8ass38drb.png" alt="Go to resoursce" width="800" height="283"></a></p>

<h2>
  
  
  PHASE 2: Create and configure a file share with directory.
</h2>

<p><code>Action plan</code></p>

<p><strong>1</strong> : Create a file share for the corporate office. <strong>In</strong> the <strong>storage account</strong>, in the <strong>Data storage section</strong>, select the <strong>File shares blade</strong>.Select <strong>+ File share</strong> </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fioc55jxwx45t8h0bm44y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fioc55jxwx45t8h0bm44y.png" alt=" " width="800" height="442"></a></p>

<p><strong>2</strong> : Proceed to provide a <strong>Name</strong>. <strong>Review</strong> the other <strong>options</strong>, but <strong>take the defaults</strong>. Select <strong>Review + Create</strong>. Select <strong>Create</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fms5iq9e7do973n29p325.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fms5iq9e7do973n29p325.png" alt="Review + Create" width="800" height="557"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzxhipxkpli7msuhua7av.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzxhipxkpli7msuhua7av.png" alt="Create" width="800" height="919"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn0wq8svv5re7i7nmueod.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn0wq8svv5re7i7nmueod.png" alt=" " width="800" height="88"></a></p>

<p><strong>3</strong> : We will add a directory to the file share for the finance department. Select your <strong>file share</strong> and select <strong>+Add directory</strong>.<br>
Name the new directory <strong>finance</strong>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjzqlvd44vozssl5goneg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjzqlvd44vozssl5goneg.png" alt="Directory name" width="800" height="169"></a></p>

<p><strong>4</strong> : Select <strong>Browse</strong> and then select the <strong>finance</strong> directory. Notice you can <strong>Add directory</strong> to further organize your file share. <strong>Upload</strong> a file of your choosing.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F32ui4y8ekg6acp9vr6ou.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F32ui4y8ekg6acp9vr6ou.png" alt="Browse" width="800" height="291"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqn26p7of6ijult4ja4gr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqn26p7of6ijult4ja4gr.png" alt="Upload to finance" width="800" height="206"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi540dn41ityhvn3h1e8u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi540dn41ityhvn3h1e8u.png" alt=" " width="800" height="194"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmloofeohfnzg87agjfvx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmloofeohfnzg87agjfvx.png" alt="Uploading file" width="800" height="218"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn9w2llj171wch4k4oxd8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn9w2llj171wch4k4oxd8.png" alt="Uploaded doc" width="800" height="267"></a></p>

<h2>
  
  
  PHASE 3: <strong>Configure</strong> and <strong>test snapshots</strong>. This is similar to blob storage, you need to protect against accidental deletion of files. We decide to use snapshots.
</h2>

<p><code>Action plan</code></p>

<p><strong>1</strong> : Select your <strong>file share</strong>. In the <strong>Operations section</strong>, select the <strong>Snapshots blade</strong>. Select <strong>+ Add snapshot</strong>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frvqrb7lidivntwplw4w6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frvqrb7lidivntwplw4w6.png" alt="Add snapshot" width="800" height="368"></a></p>

<p><strong>2</strong>: The <strong>comment</strong> is <strong>optional</strong>. Select <strong>OK</strong>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft7urtq3vqv16v6cabyad.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft7urtq3vqv16v6cabyad.png" alt="deploying the snapshot" width="800" height="364"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmyeccv6huptuc9sfzpyf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmyeccv6huptuc9sfzpyf.png" alt="Successfull" width="800" height="67"></a></p>

<p><strong>3</strong> : Select your <strong>snapshot</strong> and <strong>verify</strong> your <strong>file directory</strong> and <strong>uploaded file</strong> are <strong>included</strong>. In the <strong>Properties pane</strong> select <strong>Delete</strong>. Select <strong>Yes</strong> to confirm the <strong>deletion</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqci5sy8rdsnofv8rdjhi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqci5sy8rdsnofv8rdjhi.png" alt=" " width="800" height="230"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqpnjplo54ook52n3e5bg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqpnjplo54ook52n3e5bg.png" alt=" " width="800" height="301"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpvdw34gyve236mnp59ei.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpvdw34gyve236mnp59ei.png" alt=" " width="800" height="264"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffgbwyhwip4va45om8rup.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffgbwyhwip4va45om8rup.png" alt="locating fileshare" width="800" height="255"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbyvydanqh76s62epk80.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzbyvydanqh76s62epk80.png" alt="Deleting snapshot" width="800" height="245"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsm7gh2ftyqabrzyqtdbw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsm7gh2ftyqabrzyqtdbw.png" alt="Delete confirmation" width="800" height="175"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbd29h2gg4mc8vo136vqr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbd29h2gg4mc8vo136vqr.png" alt="Delete successful" width="800" height="142"></a></p>

<p><strong>4</strong> : Practice using snapshots to restore a file. Return to your <strong>file share</strong>. Browse to your <strong>file directory</strong>. Locate your <strong>uploaded file</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2wiclnh2f88c15lxzthi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2wiclnh2f88c15lxzthi.png" alt="Locating directory" width="800" height="306"></a></p>

<p><strong>5</strong> : Select the Snapshots <strong>blade</strong> and then select your <strong>snapshot</strong>. Navigate to the <strong>file</strong> you want to <strong>restore</strong>. Select the <strong>file</strong> and then select <strong>Restore</strong>. Provide a <strong>Restored</strong> file <strong>name</strong>. Verify your <strong>file directory</strong> has the <strong>restored file</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr54da89sd8rd0gqdv20i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr54da89sd8rd0gqdv20i.png" alt="locating directory" width="800" height="243"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcc6rbt9m3luj1wjjuqu4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcc6rbt9m3luj1wjjuqu4.png" alt="restoring deleted file" width="800" height="225"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhvjixd6q4ztdwnsu1ze9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhvjixd6q4ztdwnsu1ze9.png" alt="Selecting ok" width="800" height="398"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F33v6jumpuywz86d10sy0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F33v6jumpuywz86d10sy0.png" alt="Successfully restored" width="800" height="46"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2x8oiifwi4nu69f1exwy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2x8oiifwi4nu69f1exwy.png" alt="Verifying restored file" width="800" height="332"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj1hwndx9ikzehks9hyww.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj1hwndx9ikzehks9hyww.png" alt="Verified" width="800" height="192"></a></p>

<h2>
  
  
  PHASE 4: Configure <strong>restricting storage access</strong> to <strong>selected virtual networks</strong>. This task in this section require a <strong>virtual network</strong> with <strong>subnet</strong>.
</h2>

<p><code>Action plan</code></p>

<p><strong>1</strong> : <strong>Search for</strong> and <strong>select Virtual networks</strong>. Select <strong>Create</strong>. Select your <strong>resource group</strong> and give the <strong>virtual network</strong> a <strong>name</strong>. Take the defaults for other parameters, select <strong>Review + create</strong>, and then <strong>Create</strong>. Wait for the <strong>resource</strong> to <strong>deploy</strong>.<br>
Select <strong>Go to resource</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv0ivcvenfol2mg5t40vt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv0ivcvenfol2mg5t40vt.png" alt="Search for virtual network" width="800" height="175"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwa04s6gwlql8x5j3rsgc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwa04s6gwlql8x5j3rsgc.png" alt="Select +create" width="800" height="172"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxhpr4dbjyzi8vmfrk2hy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxhpr4dbjyzi8vmfrk2hy.png" alt="Virtual network name" width="800" height="612"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsangfkjre088dsvipgu1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsangfkjre088dsvipgu1.png" alt="create and review" width="800" height="1140"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F413t3jx8k2jqe4wdx5wg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F413t3jx8k2jqe4wdx5wg.png" alt="validating" width="800" height="83"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fepd65amojrer38cbsar4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fepd65amojrer38cbsar4.png" alt="Go to resource" width="800" height="294"></a></p>

<p><strong>2</strong> : The <strong>storage account</strong> should only be <strong>accessed</strong> from the <strong>virtual network</strong> we just created. <strong>Return</strong> to your <strong>files storage account</strong>. In the <strong>Security + networking</strong> section, select the <strong>Networking blade</strong>. <strong>Change</strong> the <strong>Public network access</strong> to <strong>Enabled</strong> from <strong>selected virtual networks</strong> and <strong>IP addresses</strong>. In the <strong>Virtual networks section</strong>, select <strong>Add existing virtual network</strong>. Select your <strong>virtual network and subnet</strong>, select <strong>Add</strong>. Be sure to <strong>Save</strong> your <strong>changes</strong>. Select the <strong>Storage browser</strong> and navigate to your <strong>file share</strong>. <strong>Verify</strong> the message <strong>not authorized</strong> to <strong>perform</strong> this <strong>operation</strong>. You are <strong>not</strong> <strong>connecting</strong> from the <strong>virtual network</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwn5mfcc530vy6l9rrmt2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwn5mfcc530vy6l9rrmt2.png" alt="Network + security" width="800" height="541"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F59gm8ir1dlyepzfdzf62.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F59gm8ir1dlyepzfdzf62.png" alt="Adding virtual network" width="800" height="450"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw3m5txp41545fey9kf3m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw3m5txp41545fey9kf3m.png" alt="Deploying private storage endpoints" width="800" height="348"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7smiumd9h3rb5t0gfumv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7smiumd9h3rb5t0gfumv.png" alt="validating private storage endpoints" width="800" height="250"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5c4c9mip48dmsu491nm3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5c4c9mip48dmsu491nm3.png" alt="Error message validated" width="800" height="297"></a></p>

<p><strong>Conclusion</strong> : ​By completing this <strong>tutorial</strong>, we have successfully established a high-performance, enterprise-grade storage environment specifically engineered for secure corporate data sharing. We began by deploying a premium, zone-redundant storage account and organizing it with dedicated file shares and directories to ensure the finance department's auditing data remains easily accessible yet highly structured. To safeguard against data loss, we implemented a snapshot strategy and verified that files can be restored instantly, providing a vital safety net against accidental deletions. <strong>Finally</strong>, we hardened the entire infrastructure by restricting access exclusively to our authorized virtual network, ensuring the company's information is protected from unauthorized external connections.</p>

<p><strong>MISSION ACCOMPLISHED</strong>!</p>

