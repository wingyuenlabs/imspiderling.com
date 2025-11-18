---
Title: Creating Storage Account for Finance Department's shared file
Description: 
Author: Barisi Lenyie
Date: 2025-11-18T21:49:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>In every project, the first thing is to create a <strong>Resource Group</strong><br>
But in this article, I will start by creating a <strong>Storage Account</strong>, </p>

<p><strong>Create a Storage Account for any Department share file, but in this article, I will use the Finance Department</strong></p>

<p><strong>Step 1</strong><br>
In the search bar, type <strong>storage account</strong> and hit enter, and choose the storage account that is grayed out<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhjt14tzq6ccmlapcgll7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhjt14tzq6ccmlapcgll7.png" alt="storage accounts" width="800" height="354"></a></p>

<p><strong>Step 2</strong><br>
select <strong>+ Create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0xc5ackybpnyoazv9psd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0xc5ackybpnyoazv9psd.png" alt="+ create" width="800" height="354"></a></p>

<p><strong>Step 3</strong><br>
In the create environment, navigate to the <strong>resource group</strong> section and click <strong>create new</strong>, but if there is an existing resource group, click the drop-down arrow and select and click <strong>ok</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F28kqltyxdafftdrektvc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F28kqltyxdafftdrektvc.png" alt="Resource Group" width="800" height="354"></a></p>

<p><strong>Step 4</strong><br>
locate the <strong>storage account name</strong> section and type a name in the bar. Remember the naming convention that it must be <strong>globally unique</strong>, choose a <strong>Region</strong> as usual, and for <strong>Performance</strong>, choose <strong>Premium</strong>, for <strong>premium account type</strong> select <strong>file share</strong>, and for <strong>redundancy</strong>, select <strong>zone redundant storage</strong>, and <strong>review + create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1rnx0fxz9w5qi08olcd0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1rnx0fxz9w5qi08olcd0.png" alt="Storage account name, region, performance, premium account type, redundancy &amp; review + create" width="800" height="354"></a></p>

<p><strong>Step 5</strong><br>
click <strong>create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fajunim80esc114awjcrd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fajunim80esc114awjcrd.png" alt="create" width="800" height="354"></a></p>

<p><strong>Step 6</strong><br>
Click <strong>Go to Resource</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0inhh3x5wd0z9mmtfpn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0inhh3x5wd0z9mmtfpn.png" alt="go to resource" width="800" height="354"></a></p>

<p><strong>Steps on how to add a directory to the file share created for the Finance Department</strong></p>

<p><strong>Step 1</strong><br>
By the <strong>overview</strong> panel, navigate to <strong>Data Storage</strong> and click the drop-down arrow and select <strong>file share</strong>, and from the Tabs, select *<em>+file *</em><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fucmedjactvympchsdy1j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fucmedjactvympchsdy1j.png" alt="overview, data storage, file share &amp; + file" width="800" height="354"></a></p>

<p><strong>Step 2</strong><br>
Give a name to the file and click <strong>Review + create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fceniokxvxdnq7doolopr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fceniokxvxdnq7doolopr.png" alt="name, review + create" width="800" height="354"></a></p>

<p><strong>Step 3</strong><br>
Click <strong>create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqoitum56iee98147y8je.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqoitum56iee98147y8je.png" alt="create" width="800" height="354"></a></p>

<p><strong>Steps on how to Add a Directory to the File share for a particular department or segment for future testing and uploading of files</strong></p>

<p><strong>Step 1</strong><br>
Click <strong>+ Add Directory</strong>, and in the pop-up bar, enter a name (e.g., Finance) and click <strong>OK</strong>. <br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fet7rxwfrj2hpxwp7i5rm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fet7rxwfrj2hpxwp7i5rm.png" alt="+ Add Directory" width="800" height="354"></a></p>

<p><strong>Step 2</strong><br>
click <strong>Browse</strong> and select finance<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fybndiz07iwb0y7y0hdje.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fybndiz07iwb0y7y0hdje.png" alt="browse &amp; finance" width="800" height="354"></a></p>

<p><strong>Step 3</strong><br>
Click on <strong>Upload</strong> on the tab, click <strong>Browse for file</strong>, and click <strong>Upload</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk22evaysvrqh0nylyh6m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk22evaysvrqh0nylyh6m.png" alt="upload" width="800" height="354"></a></p>

<p><strong>Steps on how to configure and test a snapshot</strong></p>

<p><strong>Step 1</strong><br>
In the file share environment, navigate to the <strong>Operations</strong> section and click the drop-down, and select <strong>snapshot</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F912jc5exx8w8jpv3x6mz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F912jc5exx8w8jpv3x6mz.png" alt="operations,snapshot," width="800" height="354"></a></p>

<p><strong>Steps on how to restore files with a snapshot</strong></p>

<p><strong>Step 1</strong><br>
From the <strong>overview</strong> pane, navigate to <strong>Data Storage</strong> and from the drop-down, select <strong>file share</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgw1en7wma3snlye0af0u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgw1en7wma3snlye0af0u.png" alt="overview and data storage" width="800" height="354"></a></p>

<p><strong>Step 2</strong><br>
Select <strong>Browse</strong> and locate the uploaded file, and click the three dots at the extreme and select <strong>delete</strong> from the options provided<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkrio6p71wi56h2rag4bh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkrio6p71wi56h2rag4bh.png" alt="browse" width="800" height="354"></a></p>

<p><strong>Step 3</strong><br>
Click <strong>yes</strong> to confirm delete<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fondbdajgzrvvd0x83ga9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fondbdajgzrvvd0x83ga9.png" alt="selection of image" width="800" height="420"></a></p>

<p><strong>Step 4</strong><br>
Navigate to <strong>operations</strong> and from the drop-down arrow, select <strong>snapshot</strong>, and double-click on the uploaded file<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc3uzgoov8uhezw0saqrh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc3uzgoov8uhezw0saqrh.png" alt="operations" width="800" height="354"></a></p>

<p><strong>Step 5</strong><br>
On the uploaded image bar, click the three dots at the extreme and select <strong>restore</strong> from the options provided<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frfcpiu54jpmkpwnsc1b4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frfcpiu54jpmkpwnsc1b4.png" alt="uploaded image" width="800" height="354"></a></p>

<p><strong>Step 6</strong><br>
Give it a restore name and click <strong>ok</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa3iisikv17boqicxwcik.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa3iisikv17boqicxwcik.png" alt="Image" width="800" height="354"></a></p>

<p><strong>Step 7</strong><br>
Go back to <strong>browse</strong> for confirmation of the restore file, and also notice that the new restore file name will be the same as the one in the above step<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frroiawc7b0l684lk7a74.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frroiawc7b0l684lk7a74.png" alt="browse" width="800" height="354"></a></p>

<p><strong>Steps on how to configure restricting storage access to the selected virtual network</strong></p>

<p><strong>Step 1</strong><br>
In the search bar, type <strong>virtual networks</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F388tz0duh1ehdgsg4oaa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F388tz0duh1ehdgsg4oaa.png" alt="virtual networks" width="800" height="354"></a></p>

<p><strong>Step 2</strong><br>
select <strong>+create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxojky0019y037pkyll86.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxojky0019y037pkyll86.png" alt="+create" width="800" height="354"></a></p>

<p><strong>Step 3</strong><br>
Give the Virtual Network name in the bar provided, and click <strong>review  + create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpu4a7a8f5r40ed81dpx0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpu4a7a8f5r40ed81dpx0.png" alt="virtual network and review + create" width="800" height="354"></a></p>

<p><strong>Step 4</strong><br>
click <strong>create</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg2f8yaa2sk2iv57noskt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg2f8yaa2sk2iv57noskt.png" alt="create" width="800" height="354"></a></p>

<p><strong>Step 5</strong><br>
Click <strong>Go to Resource</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu2usz3amgsouojyjaol1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu2usz3amgsouojyjaol1.png" alt="go to resource" width="800" height="354"></a></p>

<p><strong>Step 6</strong><br>
In the <strong>overview</strong> panel, navigate to <strong>settings</strong> and from the drop-down options, select <strong>subnet</strong> and the default <br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqmhzhmljvzdu4lpfd7o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqmhzhmljvzdu4lpfd7o.png" alt="overview, settings, and subnet" width="800" height="354"></a></p>

<p><strong>Step 7</strong><br>
Navigate to the <strong>service endpoints</strong>, under <strong>service</strong>, click the drop-down, and select <strong>Microsoft Storage</strong><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpkblg1ynpi2o1f1k3qx5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpkblg1ynpi2o1f1k3qx5.png" alt="service endpoints" width="800" height="354"></a></p>

<p>I hope this article was helpful.</p>

