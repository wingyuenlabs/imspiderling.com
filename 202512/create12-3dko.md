---
Title: create12
Description: 
Author: Query Filter
Date: 2025-12-03T21:48:32.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/* -------------------------------------------------------------
   ONE SINGLE EXECUTION – works in DBVisualizer / isql / any tool
   Shows rows from last 45 days for every table that has INTERNTIMESTAMP
   ------------------------------------------------------------- */

declare @sql nvarchar(8000)   -- use nvarchar if your ASE supports it (15.7+)
        -- or varchar(8000) on very old versions

select @sql = ''

/* Build the big UNION ALL query */
select @sql = @sql + 
       'select ''' + o.name + ''' as table_name, count(*) as rows_last_45_days from ' + o.name +
       ' where INTERNTIMESTAMP &gt;= dateadd(day, -45, getdate()) union all '
from sysobjects o
where o.type = 'U'
  and exists (select 1 from syscolumns c 
              where c.id = o.id and c.name = 'INTERNTIMESTAMP')
order by o.name

/* Safely remove the trailing ' union all ' – no negative length */
if len(@sql) &gt; 11
   select @sql = left(@sql, len(@sql) - 11)   -- removes last 11 chars
else
   select @sql = 'select ''No tables found'' as table_name, 0 as rows_last_45_days'

/* Execute it */
exec(@sql)
go
</code></pre>

</div>



