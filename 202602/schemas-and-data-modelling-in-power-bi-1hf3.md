---
Title: Schemas and data modelling in Power BI
Description: 
Author: Byrone_Code
Date: 2026-02-07T21:45:39.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  INTRODUCTION
</h2>

<p>Data is now crucial in every industry, and its role is especially <br>
important in the context of finance-related processes. In addition to collecting information from various sources, an equally important issue is its effective analysis and visualisation. Special software, such as Microsoft Power BI, is used for this purpose.</p>

<p>One crucial aspect of using Power BI effectively is understanding the different types of schemas used to structure data. In this article, we will explore the various schemas in Power BI, their characteristics, and when to use each one to maximize the efficiency of your data models.</p>

<h2>
  
  
  What is a Schema in Power BI?
</h2>

<p>A <strong>data schema</strong> is a structure that defines the relationships of data in a database or other data storage system.<br>
Schemas define how data is connected and related within the model, influencing the efficiency and performance of data queries and reports. Understanding schemas helps in designing best data models that support comprehensive analysis.</p>

<h1>
  
  
  Types of Schemas in Power BI
</h1>

<p><strong>1. STAR SCHEMA</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8exop1i4ewva9rmnwhjf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8exop1i4ewva9rmnwhjf.png" alt=" " width="800" height="534"></a></p>

<p><strong>Definition:</strong> The star schema is a simple and commonly used schema in data warehousing. It consists of a central fact table surrounded by dimension tables, forming a star-like pattern.</p>

<p><strong>Structure</strong> The central fact table contains quantitative data (e.g., sales), while the dimension tables hold descriptive attributes related to the facts (e.g. Employee, Date, Territory).</p>

<p><strong>Use Cases</strong> Star schemas are ideal for straightforward reporting and querying. They are efficient for read-heavy operations, making them suitable for dashboards and summary reports.</p>

<p><strong>2. SNOWFLAKE SCHEMA</strong><br>
<strong>Definition:</strong> The snowflake schema is a normalized version of the star schema. In this design, dimension tables are further divided into related tables, resulting in a more complex structure.</p>

<p><strong>Structure:</strong> The normalization process eliminates redundancy by splitting dimension tables into multiple related tables. This results in a web-like structure, resembling a snowflake.</p>

<p><strong>Use Cases:</strong> Snowflake schemas are used in scenarios requiring detailed data models and efficient storage. They are beneficial when dealing with large datasets where data redundancy needs to be minimized.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq0w8f7bkjmhwli2iqc8t.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq0w8f7bkjmhwli2iqc8t.png" alt=" " width="800" height="559"></a></p>

<p><strong>3. GALAXIES SCHEMA(OR FACT CONSTELLATION SCHEMA)</strong></p>

<p><strong>Definition:</strong> The galaxies schema, also known as the fact constellation schema, involves multiple fact tables that share dimension tables, creating a complex, interconnected data model.</p>

<p><strong>Structure</strong> This schema consists of multiple fact tables linked to shared dimension tables, enabling the analysis of different business processes within a single model.</p>

<p><strong>Use Cases:</strong>Galaxies schemas are suitable for large-scale enterprise environments where multiple related business processes need to be analyzed. They support complex queries and detailed reporting across various domains.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv5cnydbrqzphphamltlx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv5cnydbrqzphphamltlx.png" alt=" " width="800" height="400"></a></p>

<h2>
  
  
  How Schemas Impact PowerBI
</h2>

<p><strong>Impact on performance</strong><br>
<em><strong>Faster Queries (Star Schema)</strong></em>: A star schema with a central fact table and direct, denormalized dimensions minimizes the number of joins the engine must process.<br>
<em><strong>Slower Queries (Snowflake Schema)</strong></em>: Normalizing dimensions into multiple related tables requires more joins, which can slow down report responsiveness, especially with large datasets.<br>
<em><strong>Efficient Compression</strong></em>: The VertiPaq engine thrives on star schemas, creating smaller in-memory models that improve visual and calculation speed. </p>

<p><strong>Impact on DAX and Usability</strong><br>
<em><strong>Simpler DAX (Star Schema)</strong></em>: A clean star schema reduces the need for complex, nested DAX calculations because relationships are direct and easy to follow.<br>
<em><strong>Filter Propagation:</strong></em> In a star schema, filters move directly from dimension tables to the fact table, ensuring consistent and predictable results.<br>
<strong>Reduced Complexity:</strong> A star schema is easier for users to understand and navigate, making self-service analytics more intuitive. </p>

<p><strong>Impact on Data Integrity and Storage</strong><br>
<em><strong>Data Accuracy (Star Schema)</strong></em>: While denormalized (some redundancy), star schemas are generally better at preventing ambiguous, bidirectional, or many-to-many relationships, thus reducing the risk of double-counting.<br>
<em><strong>Storage Efficiency (Snowflake Schema)</strong></em>: Snowflake schemas reduce redundancy, which can save space. However, this is rarely necessary in modern Power BI environments where speed is prioritized over storage costs.</p>

<h2>
  
  
  WHAT IS DATA MODELLING?
</h2>

<p>Data modelling is the process of creating a visual representation of how data is arranged and related in a database or system.</p>

<p>A data modeller develops a detailed plan for how data will be stored and arranged in a database, much like an architect does to construct a building like creating the blueprint of a building.</p>

<p><strong>Why is Data Modelling important?</strong><br>
Data modelling is important because it turns raw data into a clear, reliable structure that supports accurate analysis, efficient systems, and good decision-making.</p>

<p><em><strong>1. Clarity &amp; Structure</strong></em><br>
Data modelling defines what data exists, how it’s related, and what it means. Without it, data is just a messy pile of tables and columns.</p>

<p><em><strong>2. Better Decision-Making</strong></em><br>
A good model ensures data is consistent, accurate, and complete, so reports and dashboards actually reflect reality—not misleading numbers.</p>

<p><strong><em>3. Performance &amp; Efficiency</em></strong><br>
Proper models (e.g. star/snowflake schemas) make queries faster and systems more scalable—critical in analytics, BI, and big data environments.<br>
<strong><em>Data Integrity &amp; Quality</em></strong><br>
Constraints, relationships, and rules in a model prevent duplication, inconsistency, and errors (garbage in = garbage out).</p>

<p><strong><em>Easier Maintenance &amp; Scalability</em></strong><br>
When business needs change, a well-designed model can be extended without breaking everything else.</p>

<p><em><strong>Common Language Between Teams</strong></em><br>
Data models act as a bridge between business users, analysts, and engineers, reducing misunderstandings.</p>

<p><strong><em>Foundation for Analytics &amp; AI</em></strong><br>
Machine learning, reporting, forecasting—none of these work well without a solid underlying data model.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Understanding different schemas in Power BI is crucial for designing efficient data models. Each schema has unique advantages: the star schema is ideal for straightforward reporting and querying, offering simplicity and ease of use; the snowflake schema provides detailed, normalized structures, reducing redundancy and optimizing storage; and the galaxies schema supports complex, large-scale data models with multiple fact tables sharing dimension tables. Choosing the right schema improves query performance, data storage efficiency, and data refresh operations. By mastering these schemas, you can create robust and scalable data models, enabling your organization to make data-driven decisions effectively.</p>

