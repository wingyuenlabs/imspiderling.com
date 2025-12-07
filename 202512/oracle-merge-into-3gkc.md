---
Title: Oracle Merge Into
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:43:51.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>MERGE INTO FINT_CAD_D TRG
USING
      (
         SELECT 
         CD.ROWID AS RID,
         CM.CREDIT_ACC_ID
         FROM        FINT_CAD_D CD  
         JOIN  FINT_CAD_M CM ON CD.CAD_M_ID = CM.CAD_M_ID  
         WHERE CM.CAD_M_ID = 4918

       ) SRC  ON (TRG.ROWID = SRC.RID)

WHEN MATCHED THEN 

UPDATE SET TRG.CREDIT_ACC_ID = SRC.CREDIT_ACC_ID

</code></pre>

</div>



