---
Title: Oracle Menu Tree
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:38:35.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>select sys_connect_by_path (substr(mnu_name,0,50) ,'--&gt;')  as path, level
from appd_menu
start with mnu_id = 1
connect by prior mnu_id = prt_id
order by path

</code></pre>

</div>



