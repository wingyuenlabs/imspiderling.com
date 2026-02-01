---
Title: #Schemas and Data Modelling in Power BI
Description: 
Author: Victor Karanja
Date: 2026-02-01T22:01:17.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Schemas and Data Modelling in Power BI
</h2>

<p>Data modelling is one of the most important steps in Power BI. A well-designed data model improves performance, accuracy, and ease to analyse, while a poor model can lead to slow reports and incorrect insights.</p>

<p>This article explains schemas and data modelling concepts in Power BI.</p>




<h2>
  
  
  What Is Data Modelling?
</h2>

<p>Data Modelling involves organizing data sources into a structured model. In Power BI, this means organizing tables and defining relationships between them to support accurate reporting and efficient calculations.</p>




<h2>
  
  
  What Is a Schema?
</h2>

<p>A schema is the structure and organization of data in a table. It's the logical arrangement of tables used in reports on Power BI.</p>

<p>In Power BI, schemas help determine:</p>

<ul>
<li>How data is connected</li>
<li>How filters flow between tables</li>
<li>How efficiently queries are executed. </li>
</ul>

<p>****Types of schemas in Power BI.</p>

<p>-Galaxy schema<br>
-Snowflakes schema<br>
-star schema</p>

<h2>
  
  
  Snowflakes schema
</h2>

<p>Here dimensions are split into sub dimensions, that can also be split into further smaller tables. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzf3acrco6ua0tqlb8j33.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzf3acrco6ua0tqlb8j33.jpg" alt="_snowflake diagram_" width="800" height="533"></a></p>

<h2>
  
  
  <em>Star schema</em>
</h2>

<p>It's the mostly schema used in Excel, where it has multiple dimensions and one fact table.<br>
its advantages are; </p>

<ul>
<li>Faster query performance</li>
<li>Easier to write DAX formulas</li>
<li>Better filter
-Easy to understand.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9hcmxfxrx8y4hbcc9bll.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9hcmxfxrx8y4hbcc9bll.png" alt="_star schema image_" width="800" height="533"></a></p>

<h2>
  
  
  Fact Tables
</h2>

<p>A fact table stores quantitative data (measures) that can be analysed.</p>

<h3>
  
  
  Characteristics of Fact Tables:
</h3>

<ul>
<li>Contain numerical values</li>
<li>Have many rows</li>
<li>Reference dimension tables using keys</li>
</ul>

<h3>
  
  
  Examples:
</h3>

<ul>
<li>Costomer Id</li>
<li>Name</li>
<li>Age</li>
<li>Region</li>
</ul>




<h2>
  
  
  Dimension Tables
</h2>

<p>A dimension table contains descriptive information that provides context to facts.</p>

<h3>
  
  
  Characteristics of Dimension Tables:
</h3>

<ul>
<li>Contain text or categorical data</li>
<li>Have fewer rows than fact tables</li>
<li>Used for filtering and grouping data</li>
</ul>

<h3>
  
  
  Examples:
</h3>

<ul>
<li>Date</li>
<li>Product</li>
<li>Customer ID</li>
<li>Location</li>
</ul>




<h2>
  
  
  What Is a Relationship in Power BI?
</h2>

<p>A relationship connects a column in one table to a column in another table, usually through a key.</p>

<p>Example:</p>

<ul>
<li>Sales[ProductID] → Product[ProductID]</li>
</ul>

<p>This allows Power BI to understand how records relate across tables.</p>




<h2>
  
  
  Relationship Types in Power BI
</h2>

<h3>
  
  
  One-to-Many
</h3>

<p>This is the most common and recommended relationship type.</p>

<ul>
<li>One record in a dimension table</li>
<li>Many matching records in a fact table</li>
</ul>

<p>Example:</p>

<ul>
<li>One product → many sales records</li>
</ul>

<p>Used in star schemas.</p>




<h3>
  
  
  Many-to-Many (<em>:</em>)
</h3>

<p>Occurs when both tables contain duplicate values.</p>

<ul>
<li>Can cause ambiguous results</li>
<li>Should be avoided when possible</li>
</ul>

<p>Use only when necessary and with caution.</p>




<h3>
  
  
  One-to-One (1:1)
</h3>

<ul>
<li>Each value appears once in both tables</li>
<li>Rarely used in analytical models</li>
</ul>




<h2>
  
  
  Relationship Direction
</h2>

<h3>
  
  
  Single Direction (Recommended)
</h3>

<ul>
<li>Filters flow from dimension tables to fact tables</li>
<li>Predictable and efficient behavior</li>
</ul>

<p>Best practice for star schemas.</p>




<h3>
  
  
  Both Direction (Bi-Directional)
</h3>

<ul>
<li>Filters flow both ways</li>
<li>Can cause confusion and performance issues</li>
</ul>

<p>Use only when absolutely necessary.</p>




<h2>
  
  
  Relationships in Star Schema
</h2>

<p>In a star schema:</p>

<ul>
<li>All dimension tables connect directly to the fact table</li>
<li>Relationships are one-to-many</li>
<li>Filter direction is single</li>
</ul>

<p>This results in:</p>

<ul>
<li>Faster performance</li>
<li>Simpler DAX formulas</li>
<li>Accurate filtering</li>
</ul>




<h2>
  
  
  Relationships in Snowflake Schema
</h2>

<p>In a snowflake schema:</p>

<ul>
<li>Dimension tables connect to other dimension tables</li>
<li>More relationships are required</li>
<li>More joins occur during queries</li>
</ul>

<p>This can:</p>

<ul>
<li>Reduce performance in Power BI</li>
<li>Complicate filter behavior</li>
<li>Make DAX harder to write and maintain</li>
</ul>




<h2>
  
  
  Why Relationships Matter in Power BI
</h2>

<p>Correct relationships:</p>

<ul>
<li>Ensure accurate calculations</li>
<li>Control how filters behave</li>
<li>Improve report performance</li>
<li>Prevent incorrect totals</li>
</ul>

<p>Poor relationships can lead to:</p>

<ul>
<li>Incorrect results</li>
<li>Slow visuals</li>
<li>Broken slicers and filters</li>
</ul>




<h2>
  
  
  Best Practices for Power BI Relationships
</h2>

<ul>
<li>Use star schema whenever possible</li>
<li>Keep relationships one-to-many</li>
<li>Use single-direction filtering</li>
</ul>

<h2>
  
  
  - Avoid many-to-many relationships
</h2>

<h2>
  
  
  <strong>Conclusion</strong>
</h2>

<p>Good data modelling is the foundation of effective Power BI reporting. By using well-structured schemas such as the star and snowflake schema and defining correct relationships, you can improve performance, ensure accurate calculations, and create reports that are easy to understand and maintain. Investing time in proper modelling leads to faster insights and more reliable decision-making.</p>

