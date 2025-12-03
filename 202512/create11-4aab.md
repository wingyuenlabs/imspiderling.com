---
Title: create11
Description: 
Author: Query Filter
Date: 2025-12-03T21:44:42.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>declare @sql varchar(8000)
select @sql = ''
select @sql = @sql + 'select "'+name+'" , count(*) from '+name+
              ' where INTERNTIMESTAMP &gt;= dateadd(dd,-45,getdate()) union all '
from sysobjects where type='U' 
  and exists (select 1 from syscolumns c where c.id=object_id(name) and c.name='INTERNTIMESTAMP')
order by name

-- remove the trailing ' union all '
select @sql = substring(@sql,1,len(@sql)-11)  
exec(@sql)
</code></pre>

</div>



