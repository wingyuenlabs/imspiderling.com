---
Title: Dummy Guide to Schemas and Data Modelling In Power BI
Description: 
Author: Owen Avedi 
Date: 2026-02-01T21:17:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>Here is a super simple explanation of schemas and data modelling in Power BI. Think of Power BI like building with interlocking blocks.</p>

<h2>
  
  
  What is Data Modelling?
</h2>

<p>Data Modelling is deciding which block is going to interlock with another, and how they should interlock so that you can build something useful.<br>
You take all your messy tables, organize them, and connect them properly so that Power BI can, in turn, answer questions like:</p>

<ul>
<li><p>"How much did each client spend last year?"</p></li>
<li><p>"Which products sell best in a particular country?"</p></li>
</ul>

<h2>
  
  
  What is a Schema?
</h2>

<p>First off, we need to know what a schema consists of. It has the <strong>fact table</strong>, which is the biggest table with lots of rows: it can contain numbers you want to add up (measures) or IDs linking to other tables.</p>

<p>It also consists of <strong>Dimension tables</strong>, which are smaller tables with unique rows that also contain descriptive attributes you use to filter.</p>

<p>The schema is the final blueprint that shows:</p>

<ul>
<li><p>The tables you have</p></li>
<li><p>The columns inside each table</p></li>
<li><p>How the tables are connected to each other</p></li>
</ul>

<p>There are 3 very common schemas in PowerBI:</p>

<h2>
  
  
  1. Star Schema
</h2>

<p>This is the easiest for beginners and highly recommended in Power BI. It looks like one big central table with many small tables around it.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffxekxdeb5ve4eknjmk5h.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffxekxdeb5ve4eknjmk5h.gif" alt=" " width="439" height="354"></a></p>

<p>The fact table has the numbers you want to add up: Amount, Quantity e.t.c.<br>
All the other tables are called dimension tables; they connect to the fact table usually with one number column, e.g., Customer ID, Product ID, or Date.</p>

<h2>
  
  
  2. Snowflake Schema
</h2>

<p>It looks like star schema, but some dimensions are split further. It still has the fact table, but the dimension tables are split into extra levels.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbr6ozvfsw99f1yxkkbo1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbr6ozvfsw99f1yxkkbo1.png" alt=" " width="800" height="483"></a></p>

<p>It's best for very large data bases where storage matters a lot. Only use when you really need it, as it's harder and slightly slower than star schema.</p>

<p>Something to know,</p>

<h2>
  
  
  Relationships - How tables talk to each other
</h2>

<p>In Power BI, it is a line connecting two tables so filters flow between them.<br>
Relationships almost always in good models → <strong>one-to-many</strong> because it offers fast calculations, correct filtering, and no wrong totals, there's also:</p>

<ul>
<li><p><strong>One Side</strong>- Has unique values, for example, customer ID in customer table, each ID appears only once.</p></li>
<li><p><strong>Many Side</strong>- Has repeating values, many rows can have the same values, for example, customer ID in sales table, same customer buys many times.</p></li>
</ul>

<p>Overall, stick with star schema, it's what most good Power BI models use.</p>

