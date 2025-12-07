---
Title: Dynamic List Data Authority
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:41:56.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>DO $$

DECLARE

result CONSTANT refcursor:= 'result';

BEGIN 

PERFORM RPA_HRMD_REGISTER (@UsrId@::INTEGER);  
PERFORM RPA_HRMD_EMPLOYEE (@UsrId@::INTEGER);

OPEN result FOR 

SELECT

PYR.PAYROLL_ID, 
EMP.EMPLOYEE_ID, 
REG.REGISTER_ID,
REG.REGISTER_CODE    AS "SİCİL NO", 
REG.REGISTER_NAME||' '||REG.REGISTER_SURNAME AS "ADI SOYADI",
REG.CITIZENSHIP_NO   AS "TC KİMLİK NO",
PYR.AMT_NET          AS "NET ÖDENEN"

FROM       HRMT_PAYROLL     PYR   
INNER JOIN RP_HRMD_EMPLOYEE EMP ON PYR.EMPLOYEE_ID = EMP.EMPLOYEE_ID  
INNER JOIN RP_HRMD_REGISTER REG ON REG.REGISTER_ID = EMP.REGISTER_ID  

WHERE TO_CHAR(PYR.PAYROLL_YEAR)  = TO_CHAR('@Year@')  
AND   TO_CHAR(PYR.PAYROLL_MONTH) = TO_CHAR('@Month@')  

ORDER BY REG.REGISTER_CODE;

END
$$;

FETCH ALL FROM result;

</code></pre>

</div>



