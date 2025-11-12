---
Title: full text
Description: 
Author: Query Filter
Date: 2025-11-12T21:17:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>CREATE OR REPLACE PROCEDURE get_blob_full_text(<br>
    p_key1 IN VARCHAR2,<br>
    p_key2 IN VARCHAR2 DEFAULT NULL<br>
) IS<br>
    l_full_text CLOB := EMPTY_CLOB();<br>
    l_chunk_content VARCHAR2(32767);<br>
BEGIN<br>
    -- Enable large buffer for DBMS_OUTPUT<br>
    DBMS_OUTPUT.ENABLE(1000000);</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>FOR rec IN (
    SELECT 
        blob_to_text_range(t.VERSIONS, c.start_byte, c.end_byte) as chunk_content
    FROM CacheIOIStates t
    CROSS JOIN (
        SELECT 
            level as chunk_num,
            (level * 4000 - 3999) as start_byte,
            (level * 4000) as end_byte
        FROM dual 
        CONNECT BY level &lt;= CEIL(
            (SELECT blob_length(VERSIONS) FROM CacheIOIStates WHERE IOIID = p_key1) / 4000
        )
    ) c
    WHERE t.IOIID = p_key1
    ORDER BY c.chunk_num
) LOOP
    IF rec.chunk_content IS NOT NULL THEN
        DBMS_OUTPUT.PUT_LINE(rec.chunk_content);
    END IF;
END LOOP;
</code></pre>

</div>

<p>EXCEPTION<br>
    WHEN OTHERS THEN<br>
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);<br>
END get_blob_full_text;<br>
/</p>

