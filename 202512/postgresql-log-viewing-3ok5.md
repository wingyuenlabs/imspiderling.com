---
Title: PostgreSQL Log Viewing
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:45:55.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT 
op_date,
op,
op_data -&gt;&gt; 'worder_m_id',
op_data -&gt;&gt; 'item_attribute1_id',
op_data -&gt;&gt; 'qty_man'
FROM uyumlog.log
WHERE table_name = 'prdt_worder_m' and op_data -&gt;&gt; 'worder_m_id'   = '4149' 
ORDER BY op_date asc

</code></pre>

</div>



