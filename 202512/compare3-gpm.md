---
Title: compare3
Description: 
Author: Query Filter
Date: 2025-12-02T21:41:53.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT 'ONLY_IN_US_1' AS location, t1.table_name
FROM dba_tables t1
LEFT JOIN dba_tables t2
       ON t1.table_name = t2.table_name
      AND t2.owner = 'GLOBAL_COMET_US_2'
WHERE t1.owner = 'GLOBAL_COMET_US_1'
  AND t2.table_name IS NULL

UNION ALL

SELECT 'ONLY_IN_US_2' AS location, t2.table_name
FROM dba_tables t2
LEFT JOIN dba_tables t1
       ON t2.table_name = t1.table_name
      AND t1.owner = 'GLOBAL_COMET_US_1'
WHERE t2.owner = 'GLOBAL_COMET_US_2'
  AND t1.table_name IS NULL

ORDER BY location, table_name;

</code></pre>

</div>



