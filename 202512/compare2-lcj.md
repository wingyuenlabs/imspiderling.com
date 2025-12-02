---
Title: compare2
Description: 
Author: Query Filter
Date: 2025-12-02T21:39:04.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT
    CASE
        WHEN t1.table_name IS NOT NULL AND t2.table_name IS NULL
            THEN 'ONLY_IN_US_1'
        WHEN t2.table_name IS NOT NULL AND t1.table_name IS NULL
            THEN 'ONLY_IN_US_2'
    END AS location,
    COALESCE(t1.table_name, t2.table_name) AS table_name
FROM dba_tables t1
FULL OUTER JOIN dba_tables t2
    ON t1.table_name = t2.table_name
   AND t1.owner = 'GLOBAL_COMET_US_1'
   AND t2.owner = 'GLOBAL_COMET_US_2'
WHERE
    (t1.owner = 'GLOBAL_COMET_US_1' OR t1.owner IS NULL)
AND (t2.owner = 'GLOBAL_COMET_US_2' OR t2.owner IS NULL)
AND (
        (t1.table_name IS NOT NULL AND t2.table_name IS NULL) OR
        (t2.table_name IS NOT NULL AND t1.table_name IS NULL)
    )
ORDER BY location, table_name;

</code></pre>

</div>



