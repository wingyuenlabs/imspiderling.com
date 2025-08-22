---
Title: SQL Joins Explained - INNER, LEFT, RIGHT, FULL
Description: 
Author: Roxana Maria Haidiner
Date: 2025-08-22T19:31:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>When working with databases, your data is often stored in more than one table.<br>
But how can you bring the data together?</p>

<p>This is what SQL JOINs are for.<br>
A JOIN helps you combine rows from two tables based on a related column, usually a column that appears in both.</p>
<h2>
  
  
  Types of SQL JOINs
</h2>

<p>Here are the most common types of JOINs and what they do:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>JOIN Type</th>
<th>What it returns</th>
</tr>
</thead>
<tbody>
<tr>
<td>INNER JOIN</td>
<td>Only rows that exist in both tables</td>
</tr>
<tr>
<td>LEFT JOIN</td>
<td>All rows from the left table and matching rows from the right table</td>
</tr>
<tr>
<td>RIGHT JOIN</td>
<td>All rows from the right table and matching rows from the left table</td>
</tr>
<tr>
<td>FULL JOIN</td>
<td>All rows from both tables</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  What Is a SQL JOIN?
</h2>

<p>Let’s say you have:</p>

<ul>
<li>One table with a list of <strong>employees</strong>
</li>
<li>Another table with a list of <strong>departments</strong>
</li>
</ul>

<p>Each employee works in a department. If we want to know <strong>who works where</strong>, we need to connect the tables using a JOIN.</p>

<p>The JOIN compares values in both tables and brings the matching rows together.</p>


<h2>
  
  
  Example Tables
</h2>
<h3>
  
  
  Table: Employees
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>employee_id</th>
<th>name</th>
<th>department_id</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Sarah James</td>
<td>10</td>
</tr>
<tr>
<td>2</td>
<td>Mark White</td>
<td>20</td>
</tr>
<tr>
<td>3</td>
<td>Olivia Reed</td>
<td>NULL</td>
</tr>
</tbody>
</table></div>
<h3>
  
  
  Table: Departments
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>department_id</th>
<th>department_name</th>
</tr>
</thead>
<tbody>
<tr>
<td>10</td>
<td>Engineering</td>
</tr>
<tr>
<td>20</td>
<td>Sales</td>
</tr>
<tr>
<td>30</td>
<td>HR</td>
</tr>
</tbody>
</table></div>


<h2>
  
  
  INNER JOIN
</h2>

<p>Returns only employees who are assigned to a department.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgod0npmk54twlr1znl6r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgod0npmk54twlr1znl6r.png" alt="INNER JOIN" width="200" height="200"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT name, department_name
FROM Employees
INNER JOIN Departments
ON Employees.department_id = Departments.department_id;
</code></pre>

</div>



<p>➝</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>name</th>
<th>department_name</th>
</tr>
</thead>
<tbody>
<tr>
<td>Sarah James</td>
<td>Engineering</td>
</tr>
<tr>
<td>Mark White</td>
<td>Sales</td>
</tr>
</tbody>
</table></div>

<p><em>Employees without a department, and departments with no employees, are not shown.</em></p>

<h2>
  
  
  LEFT JOIN
</h2>

<p>Returns all employees, even if they don’t belong to any department.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Figzlm2s34if92cl9bf3x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Figzlm2s34if92cl9bf3x.png" alt="LEFT JOIN" width="200" height="200"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT name, department_name
FROM Employees
LEFT JOIN Departments
ON Employees.department_id = Departments.department_id;
</code></pre>

</div>



<p>➝</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>name</th>
<th>department_name</th>
</tr>
</thead>
<tbody>
<tr>
<td>Sarah James</td>
<td>Engineering</td>
</tr>
<tr>
<td>Mark White</td>
<td>Sales</td>
</tr>
<tr>
<td>Olivia Reed</td>
<td>NULL</td>
</tr>
</tbody>
</table></div>

<p>You get <strong>all employees</strong>.<br>
If they have no department, the result will show <code>NULL</code>.</p>


<h2>
  
  
  RIGHT JOIN
</h2>

<p>Returns <strong>all departments</strong>, even if no employee is assigned to them.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbdfxz7yfr4cpr1kx5t4j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbdfxz7yfr4cpr1kx5t4j.png" alt="RIGHT JOIN" width="200" height="200"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT name, department_name
FROM Employees
RIGHT JOIN Departments
ON Employees.department_id = Departments.department_id;
</code></pre>

</div>



<p>➝</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>name</th>
<th>department_name</th>
</tr>
</thead>
<tbody>
<tr>
<td>Sarah James</td>
<td>Engineering</td>
</tr>
<tr>
<td>Mark White</td>
<td>Sales</td>
</tr>
<tr>
<td>NULL</td>
<td>HR</td>
</tr>
</tbody>
</table></div>

<p>You get <strong>all departments</strong>.<br>
If no one works there, the result shows <code>NULL</code> for the name.</p>


<h2>
  
  
  FULL JOIN
</h2>

<p>Returns <strong>all employees</strong> and <strong>all departments</strong>.<br>
If they match, they are shown together.<br>
If not, one side will be <code>NULL</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgui4dnu9xw2gs8peqs6o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgui4dnu9xw2gs8peqs6o.png" alt="FULL JOIN" width="200" height="200"></a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT name, department_name
FROM Employees
FULL JOIN Departments
ON Employees.department_id = Departments.department_id;
</code></pre>

</div>



<p>➝</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>name</th>
<th>department_name</th>
</tr>
</thead>
<tbody>
<tr>
<td>Sarah James</td>
<td>Engineering</td>
</tr>
<tr>
<td>Mark White</td>
<td>Sales</td>
</tr>
<tr>
<td>Olivia Reed</td>
<td>NULL</td>
</tr>
<tr>
<td>NULL</td>
<td>HR</td>
</tr>
</tbody>
</table></div>

<p>This gives you the <strong>full picture</strong>.</p>




<h2>
  
  
  Summary
</h2>

<ul>
<li>
<strong>Use INNER JOIN</strong> when you want only matching data</li>
<li>
<strong>Use LEFT JOIN</strong> when you want all rows from the first table</li>
<li>
<strong>Use RIGHT JOIN</strong> when you want all rows from the second table</li>
<li>
<strong>Use FULL JOIN</strong> when you want everything, matched or not</li>
</ul>




<h2>
  
  
  How to Do This Visually in DbSchema
</h2>

<p>You can use <a href="https://dbschema.com/" rel="noopener noreferrer">DbSchema</a> to build JOINs without writing any SQL.</p>

<ol>
<li><p>Drag the two tables into the canvas</p></li>
<li><p>Connect them by dragging from one column to the other, to create a <a href="https://dbschema.com/blog/tutorials/what-is-a-foreign-key/" rel="noopener noreferrer">foreign key</a></p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvjvi7aisecuknwxwm7ua.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvjvi7aisecuknwxwm7ua.png" alt="Foreign Key in DbSchema" width="627" height="236"></a></p>

<ol>
<li>Open the <a href="https://dbschema.com/documentation/query-builder.html" rel="noopener noreferrer">Query Builder</a> and cascade related tables</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flxnbuzdum6q1r33f6a55.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flxnbuzdum6q1r33f6a55.png" alt="Query Builder in DbSchema" width="507" height="202"></a></p>

<ol>
<li><p>Right-click the line to select the <strong>JOIN type</strong></p></li>
<li><p>Choose the columns you want to display</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhkri05pzowdpcm6zf2v6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhkri05pzowdpcm6zf2v6.png" alt="Query Builder in DbSchema" width="569" height="229"></a></p>

<ol>
<li>Run the query and see the results</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvt5ov03ctyss978d464f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvt5ov03ctyss978d464f.png" alt="Query Builder in DbSchema" width="703" height="369"></a></p>

<p>This is a great way to <strong>learn how JOINs work</strong>, because <a href="https://dbschema.com/download.html" rel="noopener noreferrer">DbSchema</a> auto-generates SQL code for you!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd22yjuvqycap94bz0op3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd22yjuvqycap94bz0op3.png" alt="Query Builder in DbSchema" width="587" height="176"></a></p>

<blockquote>
<p><strong>Note:</strong> The JOIN types shown in the menu depend on the database you are connected to.<br>
Some databases (like SQLite) do not support <code>RIGHT JOIN</code> or <code>FULL JOIN</code>.<br>
In these cases, DbSchema will only show the join types supported by your database.</p>
</blockquote>

<h2>
  
  
  Learn More
</h2>

<p>If you're learning SQL and want more examples, visit our full SQL Tutorial here:<br>
[<a href="https://dbschema.com/blog/tutorials/" rel="noopener noreferrer">https://dbschema.com/blog/tutorials/</a>]</p>

