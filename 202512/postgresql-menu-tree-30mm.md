---
Title: PostgreSQL Menu Tree
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:37:29.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>with recursive menu_tree (mnu_id, mnu_name, prt_id, mnu_level) as 

   ( 
         select 
         mnu_id, 
         substring(mnu_name,0,60) as mnu_name, 
         prt_id, 
         1 as mnu_level
         from appd_menu 
         where mnu_id = 1 

         union all 

         select 
         m.mnu_id, 
         t.mnu_name||'--&gt;'||m.mnu_name, 
         m.prt_id, 
         t.mnu_level + 1 
         from appd_menu m, menu_tree t 
         where t.mnu_id = m.prt_id 

     ) 

select 

mnu_name, 
split_part(mnu_name,'--&gt;', 1) node1,
split_part(mnu_name,'--&gt;', 2) node2,
split_part(mnu_name,'--&gt;', 3) node3,
split_part(mnu_name,'--&gt;', 4) node4,
split_part(mnu_name,'--&gt;', 5) node5,
split_part(mnu_name,'--&gt;', 6) node6,
split_part(mnu_name,'--&gt;', 7) node7,
split_part(mnu_name,'--&gt;', 8) node8,
mnu_level 

from menu_tree order by mnu_name

</code></pre>

</div>



