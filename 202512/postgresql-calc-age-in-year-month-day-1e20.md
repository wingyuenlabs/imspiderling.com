---
Title: PostgreSQL | Calc Age in Year, Month, Day
Description: 
Author: Fazlı Akça
Date: 2025-12-07T21:33:18.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE OR REPLACE FUNCTION get_age_ymd(p_birthday date)
RETURNS text AS
$$
DECLARE
    v_years int;
    v_months int;
    v_days int;
BEGIN
    SELECT 
        EXTRACT(year FROM age(current_date, p_birthday))::int,
        EXTRACT(month FROM age(current_date, p_birthday))::int,
        EXTRACT(day   FROM age(current_date, p_birthday))::int
    INTO v_years, v_months, v_days;

    RETURN v_years || ' yıl ' || v_months || ' ay ' || v_days || ' gün';
END;
$$ LANGUAGE plpgsql IMMUTABLE;
</code></pre>

</div>



<blockquote>
<p>Örnek Kullanım<br>
</p>
</blockquote>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT
REG.REGISTER_CODE    AS "Sicil No",
REG.REGISTER_NAME    AS "Adı",
REG.REGISTER_SURNAME AS "Soyadı",
REG.BIRTHDAY         AS "Doğum Tarihi",
GET_AGE_YMD(REG.birthday::date) AS "Yaş"

FROM HRMD_REGISTER REG

</code></pre>

</div>



