---
Title: Explain the difference between subquery,CTE and stored procedures
Description: 
Author: Roy Shavi Kinyua
Date: 2025-11-06T21:50:06.000Z
Robots: noindex,nofollow
Template: index
---
<ol>
<li>Stored Procedure</li>
</ol>

<p>A stored procedure is a precompiled collection of SQL statements (and sometimes control logic) stored in the database. It’s like a function that can be called whenever needed. It can accept parameters such as input and output, and is also often used for complex logic, such as loops.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wyd9942rpxzrwhosz5k.JPG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wyd9942rpxzrwhosz5k.JPG" alt=" " width="644" height="238"></a></p>

<ol>
<li>CTE (Common Table Expression)
A CTE is a temporary named result set you can use within a single query. Think of it as a short-term, readable alias for a subquery. It exists only during query execution. CTE's make complex queries easier to read and maintain.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkg77hkg9jdbc751f22i3.JPG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkg77hkg9jdbc751f22i3.JPG" alt=" " width="640" height="167"></a></p>

<ol>
<li>Subquery
A subquery is a query nested inside another query; in other words, it is like a query within a query. It can appear within the SELECT, FROM, or WHERE clause, and it executes each time the main query is run.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjimps1x95onye5z63dfo.JPG" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjimps1x95onye5z63dfo.JPG" alt=" " width="368" height="172"></a></p>

