---
Title: compare5
Description: 
Author: Query Filter
Date: 2025-12-02T21:49:23.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>-- Declare variables for schema UIDs (run once)
DECLARE @uid1 int, @uid2 int
SELECT @uid1 = uid FROM sysusers WHERE name = 'GLOBAL_COMET_US_1'
SELECT @uid2 = uid FROM sysusers WHERE name = 'GLOBAL_COMET_US_2'

-- Query: Tables only in US_1
SELECT 'ONLY_IN_US_1' AS location, t1.name AS table_name
FROM   sysobjects t1
LEFT JOIN sysobjects t2 
       ON t1.name = t2.name
      AND t2.uid = @uid2
      AND t2.type = 'U'
WHERE  t1.uid = @uid1
  AND  t1.type = 'U'
  AND  t2.id IS NULL

UNION ALL

-- Query: Tables only in US_2
SELECT 'ONLY_IN_US_2' AS location, t2.name AS table_name
FROM   sysobjects t2
LEFT JOIN sysobjects t1 
       ON t2.name = t1.name
      AND t1.uid = @uid1
      AND t1.type = 'U'
WHERE  t2.uid = @uid2
  AND  t2.type = 'U'
  AND  t1.id IS NULL

ORDER BY 1, 2;
</code></pre>

</div>



