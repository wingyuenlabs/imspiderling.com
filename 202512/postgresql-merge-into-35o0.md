---
Title: PostgreSQL Merge Into
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:44:32.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>UPDATE PSMT_INVOICE_M

SET    SHIPPING_COUNTRY_ID = SRC.COUNTRY_ID, 
       SHIPPING_CITY_ID    = SRC.CITY_ID,
       SHIPPING_TOWN_ID    = SRC.TOWN_ID 

FROM   ( SELECT 
         PM.INVOICE_M_ID,
         FE.COUNTRY_ID,
         FE.CITY_ID,
         FE.TOWN_ID
         FROM PSMT_INVOICE_M PM  
         JOIN FIND_ENTITY    FE ON PM.ENTITY_ID = FE.ENTITY_ID 

        ) SRC    

WHERE  PSMT_INVOICE_M.INVOICE_M_ID = SRC.INVOICE_M_ID

</code></pre>

</div>



