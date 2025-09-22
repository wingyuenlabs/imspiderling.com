---
Title: JUMIA PRODUCT PERFORMANCE DASHBOARD ANALYSIS.
Description: 
Author: Faybeth Robina
Date: 2025-09-22T21:11:41.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Introduction</strong><br>
In our country Kenya Jumia is one of the largest online marketplace, offering a wide range of products like electronics, groceries, home appliance, home decor, beauty products and fashion products. It is part of the larger Jumia group, a pan African e-commerce company. Jumia Kenya was launched in May 2013 and overtime it has grown rapidly in terms of products, subscribers and vendors. Analyzing  the dataset that contains 115 products, one can understand how discount, customers reviews and ratings influence the online shopping trends.</p>

<p><strong>Dataset Overview</strong><br>
The raw dataset contained missing values, text based prices and inconsistent rating format. One has to follow the following steps to ensure the analysis is accurate :<br>
Figure below, shows the raw data</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv8tvx2cqy3xhhxbqcftg.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv8tvx2cqy3xhhxbqcftg.png" alt="Raw dataset" width="800" height="145"></a></p>

<ul>
<li><p>Replaced blanks in the review and ratings columns with 0 for completeness.</p></li>
<li><p>Converted the prices columns to numeric format</p></li>
<li><p>Discounts converted from string to numerical values.</p></li>
<li><p>Ratings extracted from text format to numeric format (e.g. 4.5 out of 5 to 4.5)</p></li>
<li><p>Reviews standardized to numeric values.</p></li>
<li><p>Removed duplicates.</p></li>
</ul>

<p><strong>Data Enrichment</strong></p>

<ul>
<li>Added new columns to categorize products based on their ratings("Poor" for ratings below 3, "Average" for 3-4, and "Excellent" for 4.5 and above).</li>
<li>Created a new column to categorized products based on discount percentage ("Low Discount" for &lt;20%, "Medium Discount" for 20-40%, "High Discount" for &gt;40%).</li>
<li>Calculated the absolute discount amount for each product (Old Price - Current Price).
Figure below shows the enriched data</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3siolgqjpf6d2a9vwm0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb3siolgqjpf6d2a9vwm0.png" alt="cleaned and enriched dataset" width="800" height="149"></a><br>
It is important to clean your data because it produces a structured dataset ready for analysis.</p>

<p><strong>Key Insights</strong><br>
The overall Key Performance Indicators(KPIs) are as follows;</p>

<ul>
<li>Average price: Ksh 1,181</li>
<li>Average discount: approximately 36.8%</li>
<li>Highest discount: 64%</li>
<li>Lowest discount: 1%</li>
<li>Total reviews analyzed: 723</li>
<li>Average product rating: approximately 2</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fszl8m4r9djd4u5g8u8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fszl8m4r9djd4u5g8u8.png" alt="KPIs" width="730" height="152"></a><br>
The figure below shows the Jumia performance dashboard.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8u0q49lksgi6c56l2d3o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8u0q49lksgi6c56l2d3o.png" alt="Jumia Dashboard" width="800" height="450"></a></p>

<p><strong>Discount Analysis</strong><br>
some products with high discounts have very low reviews counts, this shows that price cuts alone don't guarantee engagement<br>
The data shows that medium discount that is between 20% and 40% tend to attract more reviews than very high discounts.<br>
Discounts appear effective when paired with higher ratings.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyc2xsxgz4lksme95iyp1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyc2xsxgz4lksme95iyp1.png" alt="discount analysis vs reviews correlation" width="800" height="457"></a><br>
<strong>Ratings and Reviews Analysis</strong><br>
Products with higher ratings tend to have more reviews tend to have more reviews while poorly rated products have few to no reviews which indicates low engagement.<br>
<strong>Products</strong><br>
The bottom 5 lowest rated products are home decors, bags and  they have high discounts and poor ratings. This might imply that the product quality has issues rather than pricing.<br>
The figure below shows the top 5 low rated products.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5t4m5pr7k54sf9f1pdel.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5t4m5pr7k54sf9f1pdel.png" alt="bottom 5 low rated products" width="800" height="123"></a><br>
The top 5 highest rated products have excellent ratings and medium to high discount rating this shows that the higher the ratings and reviews the higher the sale of the product.</p>

<p><strong>Conclusion</strong><br>
In conclusion this report analysis shows us that discounting alone is not driving sustainable sales growth or customers loyalty. The analysis shows that while Jumia is offering a significant discounts the overall customer experience remains poor, the ratings and reviews are poor. Jumia needs to take different actions to ensure that there business is growing. Jumia should incentivize reviews by offering voucher rewards, loyalty points, or discount coupons for verified feedback.<br>
Jumia must shift from price-driven competition to a value-driven strategy. By elevating product quality, spotlighting high-rated categories, optimizing discount ranges, and building trust through reviews, Jumia can boost conversions, reduce returns, and grow long-term customer loyalty.</p>

