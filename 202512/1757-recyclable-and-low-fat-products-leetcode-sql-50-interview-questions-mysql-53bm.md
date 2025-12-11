---
Title: 1757. Recyclable and Low Fat Products | LeetCode | SQL 50 | Interview Questions | MySQL
Description: 
Author: Debesh P.
Date: 2025-12-11T21:24:21.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<h3>
  
  
  Problem Link
</h3>

<p><a href="https://leetcode.com/problems/recyclable-and-low-fat-products/" rel="noopener noreferrer">https://leetcode.com/problems/recyclable-and-low-fat-products/</a></p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5odjv6r9x3vr3n4srjpn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5odjv6r9x3vr3n4srjpn.png" alt="leetcode 1757" width="800" height="425"></a></p>




<h3>
  
  
  Solution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Write your MySQL query statement below
select
    product_id
    from Products
where
    low_fats='Y'
    and
    recyclable='Y';
</code></pre>

</div>



