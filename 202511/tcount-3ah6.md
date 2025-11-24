---
Title: tcount
Description: 
Author: Query Filter
Date: 2025-11-24T22:08:44.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT table_name,
       TO_NUMBER(extractvalue(xmltype(
           dbms_xmlgen.getxml(
               'SELECT COUNT(*) c FROM ' || table_name)),
           '//C')) AS row_count
FROM user_tables
ORDER BY row_count DESC;

</code></pre>

</div>



