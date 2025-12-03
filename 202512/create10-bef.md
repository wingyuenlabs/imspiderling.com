---
Title: create10
Description: 
Author: Query Filter
Date: 2025-12-03T21:34:30.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT 
    utc.table_name,
    xt.rows_last_45_days
FROM (
    SELECT table_name
    FROM   user_tab_columns
    WHERE  UPPER(column_name) = 'INTERNTIMESTAMP'
      AND  data_type LIKE 'TIMESTAMP%'
) utc
CROSS JOIN XMLTABLE(
    '/ROWSET/ROW/C/text()'
    PASSING DBMS_XMLGEN.GETXMLTYPE(
        'SELECT COUNT(*) c FROM "' || utc.table_name || 
        '" WHERE INTERNTIMESTAMP &gt;= SYSDATE - 45'
    )
    COLUMNS rows_last_45_days NUMBER PATH '.'
) xt
ORDER BY xt.rows_last_45_days DESC NULLS LAST, utc.table_name;
</code></pre>

</div>



