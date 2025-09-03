---
Title: A Guide to Database Normalization & Denormalization (With Visual Examples and Practical Use Cases)
Description: 
Author: Amos Augo
Date: 2025-09-03T21:17:31.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzuaua8s8qne3tcsr3dl7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzuaua8s8qne3tcsr3dl7.png" alt=" " width="800" height="533"></a></p>

<h2>
  
  
  <strong>Table of Contents</strong>
</h2>

<ol>
<li>
<strong>Introduction to Normalization</strong>
</li>
<li>
<strong>First Normal Form (1NF)</strong>
</li>
<li>
<strong>Second Normal Form (2NF)</strong>
</li>
<li>
<strong>Third Normal Form (3NF)</strong>
</li>
<li>
<strong>Boyce-Codd Normal Form (BCNF)</strong>
</li>
<li>
<strong>Fourth Normal Form (4NF)</strong>
</li>
<li>
<strong>Fifth Normal Form (5NF)</strong>
</li>
<li>
<strong>Denormalization: When and Why to Use It</strong>
</li>
<li>
<strong>Summary &amp; Best Practices</strong>
</li>
</ol>




<h2>
  
  
  <strong>Introduction to Normalization</strong>
</h2>

<p>Normalization is the process of organizing data to minimize redundancy and improve integrity. It involves splitting tables and defining relationships.  </p>

<p><strong>Key Goals:</strong>  </p>

<ul>
<li>Eliminate duplicate data.
</li>
<li>Ensure data dependencies make sense.
</li>
<li>Optimize storage and maintainability.
</li>
</ul>

<h3>
  
  
  <strong>Levels of Normalization:</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1NF → 2NF → 3NF → BCNF → 4NF → 5NF  
</code></pre>

</div>






<h3>
  
  
  <strong>1. First Normal Form (1NF)</strong>
</h3>

<p>Every table column must contain atomic (single) values with no nested lists, arrays, or repeating groups. This ensures strong data independence and lays the groundwork for higher normal forms.</p>

<p><strong>Rules:</strong>  </p>

<ul>
<li>All columns contain atomic (indivisible) values.
</li>
<li>No repeating groups.</li>
</ul>

<h4>
  
  
  <strong>Example: Before 1NF</strong>
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>OrderID</strong></th>
<th><strong>Products</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>101</td>
<td>Laptop, Mouse, Keyboard</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  <strong>After 1NF</strong>
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>OrderID</strong></th>
<th><strong>Product</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>101</td>
<td>Laptop</td>
</tr>
<tr>
<td>101</td>
<td>Mouse</td>
</tr>
<tr>
<td>101</td>
<td>Keyboard</td>
</tr>
</tbody>
</table></div>




<h3>
  
  
  <strong>2. Second Normal Form (2NF)</strong>
</h3>

<p>A table is in 2NF if it is already in 1NF and all non-key attributes fully depend on the entire primary key, not just part of it. In this way, partial dependencies are prevented and redundancy reduced.</p>

<p><strong>Rules:</strong>  </p>

<ul>
<li>Must be in 1NF.
</li>
<li>No partial dependencies (all non-key columns depend on the full primary key).</li>
</ul>

<h4>
  
  
  <strong>Example: Before 2NF</strong>
</h4>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>OrderID (PK)</strong></th>
<th><strong>ProductID (PK)</strong></th>
<th><strong>ProductName</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>101</td>
<td>P1</td>
<td>Laptop</td>
</tr>
</tbody>
</table></div>

<h4>
  
  
  <strong>After 2NF</strong>
</h4>

<p><strong>Orders Table:</strong><br><br>
| <strong>OrderID (PK)</strong> |   </p>

<p><strong>Products Table:</strong><br><br>
| <strong>ProductID (PK)</strong> | <strong>ProductName</strong> |  </p>

<p><strong>OrderDetails Table:</strong><br><br>
| <strong>OrderID (PK, FK)</strong> | <strong>ProductID (PK, FK)</strong> |  </p>




<h3>
  
  
  <strong>3. Third Normal Form (3NF)</strong>
</h3>

<p>A relation is in 3NF if it is in 2NF and no non-prime attribute depends transitively on a candidate key. Essentially, each non-key attribute must directly depend on the key, the whole key, and nothing but the key.</p>

<p>This eliminates transitive dependencies and further enforces data integrity.</p>

<p><strong>Rules:</strong>  </p>

<ul>
<li>Must be in 2NF.
</li>
<li>No transitive dependencies (non-key columns depend only on the primary key).</li>
</ul>

<h4>
  
  
  <strong>Example: Before 3NF</strong>
</h4>

<p>| <strong>StudentID</strong> | <strong>Department</strong> | <strong>DepartmentHead</strong> |  </p>

<h4>
  
  
  <strong>After 3NF</strong>
</h4>

<p><strong>Students Table:</strong><br><br>
| <strong>StudentID</strong> | <strong>Department</strong> |  </p>

<p><strong>Departments Table:</strong><br><br>
| <strong>Department</strong> | <strong>DepartmentHead</strong> |  </p>




<h3>
  
  
  <strong>3b. Boyce-Codd Normal Form (BCNF)</strong>
</h3>

<p>This is a refinement of 3NF where every determinant must be a candidate key. Used to resolve corner cases where 3NF still has undesirable dependencies.</p>

<p><strong>Rules:</strong>  </p>

<ul>
<li>Must be in 3NF.
</li>
<li>Every determinant must be a superkey.</li>
</ul>

<h4>
  
  
  <strong>Example: Before BCNF</strong>
</h4>

<p>| <strong>StudentID</strong> | <strong>Course</strong> | <strong>Professor</strong> |  </p>

<h4>
  
  
  <strong>After BCNF</strong>
</h4>

<p><strong>StudentCourses Table:</strong><br><br>
| <strong>StudentID</strong> | <strong>Course</strong> |  </p>

<p><strong>ProfessorCourses Table:</strong><br><br>
| <strong>Professor</strong> | <strong>Course</strong> |  </p>




<h3>
  
  
  <strong>4. Fourth Normal Form (4NF)</strong>
</h3>

<p>A table is in 4NF if it is already in Boyce–Codd Normal Form (BCNF) and no non-trivial multivalued dependencies exist besides those originating from a superkey.</p>

<p>Having a table in the fourth normal form ensures that multiple independent relationships don’t cause data duplication across rows. </p>

<p><strong>Rules:</strong>  </p>

<ul>
<li>Must be in BCNF.
</li>
<li>No multi-valued dependencies.
</li>
</ul>

<h4>
  
  
  <strong>Example: Before 4NF</strong>
</h4>

<p>| <strong>EmployeeID</strong> | <strong>Skill</strong> | <strong>Language</strong> |  </p>

<h4>
  
  
  <strong>After 4NF</strong>
</h4>

<p><strong>EmployeeSkills Table:</strong><br><br>
| <strong>EmployeeID</strong> | <strong>Skill</strong> |  </p>

<p><strong>EmployeeLanguages Table:</strong><br><br>
| <strong>EmployeeID</strong> | <strong>Language</strong> |  </p>




<h3>
  
  
  <strong>5. Fifth Normal Form (5NF)</strong>
</h3>

<p>Also called Project–Join Normal Form, 5NF ensures every join dependency in the table is a consequence of candidate keys.</p>

<p>This form addresses complex join constraints and ensures data is irreducible and free of redundancy due to joint relationships.</p>

<p><strong>Rules:</strong>  </p>

<ul>
<li>Must be in 4NF.
</li>
<li>No join dependencies. </li>
</ul>

<h4>
  
  
  <strong>Example: Before 5NF</strong>
</h4>

<p>| <strong>Supplier</strong> | <strong>Part</strong> | <strong>Project</strong> |  </p>

<h4>
  
  
  <strong>After 5NF</strong>
</h4>

<p><strong>SupplierParts Table:</strong><br><br>
| <strong>Supplier</strong> | <strong>Part</strong> |  </p>

<p><strong>SupplierProjects Table:</strong><br><br>
| <strong>Supplier</strong> | <strong>Project</strong> |  </p>

<p><strong>PartProjects Table:</strong><br><br>
| <strong>Part</strong> | <strong>Project</strong> |  </p>




<h2>
  
  
  <strong>Denormalization: When and Why to Use It</strong>
</h2>

<p>Denormalization refers to intentionally adding redundancy to improve read performance.  </p>

<h3>
  
  
  <strong>When to Use:</strong>
</h3>

<ul>
<li>Read-heavy workloads (e.g., analytics).
</li>
<li>Reducing complex joins.
</li>
<li>Real-time applications.
</li>
</ul>

<h4>
  
  
  <strong>Example 1: E-Commerce Order History</strong>
</h4>

<p><strong>Denormalized Table:</strong><br><br>
| <strong>OrderID</strong> | <strong>CustomerName</strong> | <strong>ProductName</strong> | <strong>TotalPrice</strong> |  </p>

<h4>
  
  
  <strong>Example 2: Social Media Like Count</strong>
</h4>

<p><strong>Denormalized Column:</strong><br><br>
<code>Posts (PostID, Content, LikeCount)</code>  </p>




<h2>
  
  
  <strong>Summary &amp; Best Practices</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th><strong>Normal Form</strong></th>
<th><strong>Purpose</strong></th>
<th><strong>Denormalization Use Case</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>1NF</td>
<td>Atomic values</td>
<td>Rarely needed</td>
</tr>
<tr>
<td>2NF</td>
<td>Eliminate partial dependencies</td>
<td>Reporting systems</td>
</tr>
<tr>
<td>3NF</td>
<td>Remove transitive dependencies</td>
<td>Data warehouses</td>
</tr>
<tr>
<td>BCNF</td>
<td>Superkey dependencies</td>
<td>High-traffic web apps</td>
</tr>
<tr>
<td>4NF/5NF</td>
<td>Handle multi-valued/join dependencies</td>
<td>Complex enterprise systems</td>
</tr>
</tbody>
</table></div>

<p><strong>Recommendation:</strong>  </p>

<ul>
<li>
<strong>Normalize first</strong> for integrity.
</li>
<li>
<strong>Denormalize selectively</strong> for performance.
</li>
</ul>




<h3>
  
  
  <strong>Visual Workflow</strong>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Raw Data → 1NF → 2NF → 3NF → BCNF → 4NF → 5NF  
          ↓  
Denormalize (for reads)  
</code></pre>

</div>



