---
Title: How Excel is Used in Real-World Data Analysis
Description: 
Author: Brian Munene
Date: 2026-03-27T22:05:20.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>A practical walkthrough from beginner to building a dashboard</em></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdcivnxdeb9gqnqugpn3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdcivnxdeb9gqnqugpn3.png" alt="Final Dashboard" width="800" height="478"></a></p>

<p>I have always known Excel as a spreadsheet application that I have mostly used to complete class assignments and do simple calculations. I have recently learnt that Excel is more than that. Excel is not just a spreadsheet application. It is a powerful tool for cleaning data, analyzing it, and presenting insights in a way that supports decision-making.</p>

<p>As part of this learning journey, I worked on a real project: building a <strong>Jumia Product Performance Dashboard</strong>. Throughout this article, I will use examples from this project to explain how Excel is used in real-world data analysis, from loading raw data to building an interactive dashboard.</p>




<h2>
  
  
  Getting Started with Excel
</h2>

<p>The first step was installing and opening Excel on desktop. Once launched, the interface introduces several important components:</p>

<ul>
<li>
<strong>Workbook</strong>: The entire Excel file
</li>
<li>
<strong>Worksheet (Sheet)</strong>: Individual tabs within a workbook
</li>
<li>
<strong>Rows and Columns</strong>: Where data is stored
</li>
<li>
<strong>Cells</strong>: The intersection of rows and columns
</li>
<li>
<strong>Ribbon</strong>: Contains all Excel tools and features
</li>
<li>
<strong>Formula Bar</strong>: Used to enter and edit formulas
</li>
</ul>

<p>Understanding this layout makes it easier to navigate Excel and perform tasks efficiently.</p>




<h2>
  
  
  Loading Data into Excel
</h2>

<p>One of the first practical things I learned was how to load external data. Excel can open CSV files, text files, and data from various other sources. The most straightforward way to load a CSV is to just open it directly in Excel through File &gt; Open, or you can use the Data tab and click "From Text/CSV".</p>

<p>When importing, Excel gives you a preview and lets you set things like the delimiter (comma, tab, semicolon), the data types for each column, and the encoding.</p>

<p>For my project, I worked with a dataset of product listings from Jumia, one of the largest e-commerce platforms in Africa. The dataset had columns for product name, current price, old price, discount percentage, number of reviews, and customer ratings. Nothing too complicated on the surface, but as I quickly learned, raw data is almost never clean enough to analyze straight away.</p>

<h2>
  
  
  Data Cleaning: The Part Nobody Talks About Enough
</h2>

<p>This is where I spent most of my time, and I think that is true for most data work in general. The Jumia dataset had several issues that needed fixing before I could do anything meaningful with it.</p>

<p><strong>Handling missing values.</strong> Some products were missing reviews or ratings. In Excel, you can spot these by filtering columns or using functions like <code>COUNTBLANK()</code> to count empty cells. Depending on the situation, you might fill in missing values with an average, remove the rows entirely, or leave them and account for it in your analysis. The <strong>Reviews</strong> and <strong>Rating</strong> columns had missing values. Instead of leaving them blank, I used:</p>

<ul>
<li>
<strong>Find and Replace (Ctrl + H)</strong>
</li>
</ul>

<p>To replace:</p>

<ul>
<li>Blank reviews → <code>"No Reviews"</code>
</li>
<li>Blank ratings → <code>"No Rating"</code> </li>
</ul>

<p><strong>Removing duplicates.</strong> The dataset had some duplicate product entries. Excel has a built-in "Remove Duplicates" feature under the Data tab that makes this pretty painless. You select the columns you want to check for duplicates, click the button, and Excel tells you how many duplicates it found and removed. I went from having some repeated products down to 109 unique entries.</p>

<p><strong>Converting data types.</strong> This was a big one. The price columns came in with "KSh" (Kenyan Shillings) as text stuck to the numbers. You cannot do math on text, so I needed to strip out "KSh". The Find and Replace tool (Ctrl+H) was a lifesaver again. I searched for "KSh" and replaced it with nothing. Then I changed the data type to <code>Currency</code>.  </p>

<p><strong>Cleaning the discount column.</strong> The discount percentages also needed to be converted from text to numeric format. Again, Find and Replace came in handy, along with the <code>LEFT()</code> and <code>RIGHT()</code> functions for extracting parts of text strings.</p>

<p><strong>Fixing the rating column.</strong> Ratings were stored as text like "4.5 out of 5" instead of just the number 4.5. I used text functions to extract just the numeric portion.</p>

<p><strong>Converting negative reviews.</strong> Some review counts came through as negative numbers, which does not make sense for a count of reviews. The <code>ABS()</code> function quickly converted those to positive values. Find and Replace can also be used to achieve the same.</p>

<p><strong>The TRIM function</strong> was another one I used a lot. <code>TRIM()</code> removes extra spaces from text, which is one of those invisible problems that can cause formulas to break or lookups to fail. If a product name has a trailing space, Excel treats it as a different value than the same name without the space, which can mess up your analysis without you even realizing it.</p>

<h2>
  
  
  Building New Columns: Data Enrichment
</h2>

<p>Once the data was clean, the next step was to create new columns that would make the analysis richer.</p>

<p><strong>Discount Amount</strong> was straightforward. It is just the Old Price minus the Current Price, giving you the actual shilling value of the discount rather than just the percentage.</p>

<p><strong>Rating Category</strong> used an <code>IF</code> formula to classify each product:</p>

<ul>
<li>"Poor" for ratings below 3</li>
<li>"Average" for ratings between 3 and 4.4</li>
<li>"Excellent" for ratings of 4.5 and above</li>
</ul>

<p>In Excel, that looks something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>=IF(H2&lt;3,"Poor",IF(H2&lt;4.5,"Average","Excellent"))
</code></pre>

</div>



<p><strong>Discount Category</strong> followed the same logic:</p>

<ul>
<li>"Low Discount" for discounts below 20%</li>
<li>"Medium Discount" for discounts between 20% and 40%</li>
<li>"High Discount" for discounts above 40%</li>
</ul>

<p>These new categories turned raw numbers into meaningful groupings that I could later use for pivot tables and dashboard filters.</p>

<h2>
  
  
  Understanding Data Types (A Simple but Powerful Tip)
</h2>

<p>One simple but very useful trick I learned is how to quickly identify data types in Excel:</p>

<ul>
<li>
<strong>Numbers are aligned to the right</strong>
</li>
<li>
<strong>Text is aligned to the left</strong>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F86hu33rnj6bhrskt42ji.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F86hu33rnj6bhrskt42ji.png" alt="Notice how text data is left aligned while numbers are right aligned within columns" width="800" height="156"></a></p>

<p>This helps you immediately spot issues. For example, if a column that should contain numbers is left-aligned, it means Excel is treating it as text, which can break calculations.</p>




<h2>
  
  
  Useful Functions I Picked Up Along the Way
</h2>

<p>Beyond the cleaning functions, here are some of the Excel functions that came up repeatedly during this project:</p>

<p><code>AVERAGE()</code> calculates the mean of a range. I used this to find that the average product rating was about 3.9 out of 5, the average discount was 36%, and the average current price was around KSh 1,210.</p>

<p><code>COUNT()</code> and <code>COUNTA()</code> count the number of cells with data. <code>COUNT()</code> works for numbers only, while <code>COUNTA()</code> counts any non-empty cell.</p>

<p><code>MAX()</code> and <code>MIN()</code> find the highest and lowest values. The most expensive product was a 32-piece Portable Cordless Drill Set at KSh 3,750, while the cheapest was a 3-piece Knitting Crochet Needle Set at just KSh 38.</p>

<p><code>SUM()</code> adds up values in a range. The total number of reviews across all 109 products was 723.</p>

<p><code>COUNTIF()</code> and <code>SUMIF()</code> are conditional versions that only count or sum values that meet a specific condition. These were useful for breaking down totals by category.</p>

<p><code>VLOOKUP()</code> searches for a value in one column and returns a corresponding value from another column. While I did not use it heavily in this particular project, it is one of those functions that comes up constantly in real-world Excel work.</p>

<h2>
  
  
  Pivot Tables: Where Excel Gets Really Powerful
</h2>

<p>If there is one feature that changed how I think about Excel, it is pivot tables. A pivot table lets you summarize, reorganize, and analyze large datasets without writing complex formulas. You drag and drop fields into rows, columns, values, and filters, and Excel does the aggregation for you.</p>

<p>For example, I created pivot tables that showed:</p>

<ul>
<li>The count of products in each discount category (59 in High Discount, 32 in Medium, 18 in Low)</li>
<li>The count of products in each rating category (23 Excellent, 22 Average, 12 Poor)</li>
<li>The average rating and review count broken down by discount level</li>
<li>The top 10 products by discount percentage and by number of reviews</li>
</ul>

<p>To create a pivot table, you go to <code>Insert &gt; PivotTable</code>, select your data range, and choose where you want the table placed. Then you drag fields from the field list into the areas you need. It is surprisingly intuitive once you try it a couple of times.</p>

<p>The pivot tables served as the data source for most of the charts on my dashboard, which is exactly how professionals use them. Instead of pointing a chart directly at raw data, you point it at a pivot table that already has the summary you need. This keeps things clean and makes updates easier.</p>

<h2>
  
  
  Charts and Visualizations
</h2>

<p>Excel offers a solid range of chart types, and choosing the right one matters for communicating your findings clearly.</p>

<p><strong>Bar and column charts</strong> work well for comparing values across categories. I used a bar chart to show the top 10 products by discount percentage, which made it easy to see that the 6-in-1 Bottle Can Opener and Creative Owl Shape items had the highest discounts at over 60%.</p>

<p><strong>Horizontal bar charts</strong> are great when product names are long (which they often are in e-commerce data). My "Top 10 Products by Number of Reviews" chart used this format so you could actually read the product names without them getting cut off. The 120W Cordless Vacuum Cleaner led the pack with 69 reviews. I used the same chart to plot of ratings versus number of reviews which showed that the highest-reviewed products (177 reviews) had a 4.6 rating, which is above average but not the highest. Meanwhile, products with a perfect 5.0 rating only had 12 reviews. So higher ratings do not automatically mean more reviews, and that is a useful insight to consider for product strategy.</p>

<p><strong>Pie and donut charts</strong> show proportions. I used pie charts to show product distribution across discount categories and rating categories. For example, 54% of products fell into the High Discount bucket, while only 17% were in Low Discount.</p>

<p><strong>Line charts</strong> are useful for showing trends. I used one to plot discount percentage against number of reviews, which revealed an interesting pattern: the relationship is not linear. The product with the most reviews (118) had a 49% discount, but there were also products with much lower discounts that still had strong engagement.</p>

<h2>
  
  
  Building the Dashboard
</h2>

<p>All of this work came together in the dashboard. A good dashboard gives someone a quick, high-level view of the data and lets them drill down when they want more detail.</p>

<p>My Jumia Product Performance Dashboard has four KPI cards across the top:</p>

<ul>
<li>Total Number of Products: 109</li>
<li>Average Rating: 3.9</li>
<li>Average Discount Percentage: 36%</li>
<li>Total Number of Reviews: 723</li>
</ul>

<p>Below that, there are six charts covering the different angles of analysis, from top products by discount and reviews, to distribution charts showing how products spread across categories, to relationship charts exploring whether discounts and ratings actually drive customer engagement.</p>

<p>I also added slicers for Rating Category and Discount Category. Slicers are visual filters that you can click to instantly filter the data across multiple charts. So if someone wants to see only the "Excellent" rated products, they click that slicer button and every chart connected to the underlying pivot tables updates. It makes the dashboard interactive without any coding.</p>

<h2>
  
  
  Key Insights from the Analysis
</h2>

<p>Working through this dataset taught me that numbers rarely tell a simple story. Here are a few things that stood out:</p>

<p>The average discount is 36%, but more than half of all products (54%) have discounts above 40%. That suggests heavy discounting is the norm on the platform.</p>

<p>Higher discounts do not automatically lead to more reviews. The product with the highest number of reviews had a 49% discount, which is above average, but there were plenty of products with steep discounts that had very few reviews. Other factors like product type, visibility, and quality probably matter more.</p>

<p>Similarly, higher-rated products tend to get more reviews, but it is not a guarantee. Products with the most reviews (177) had a solid 4.6 rating, while products with a perfect 5.0 score only had 12 reviews. </p>

<p>Products rated "Poor" (below 3) made up only about 11% of the catalog, which suggests either a baseline of quality or that poorly rated products get removed from the platform over time.</p>

<h2>
  
  
  Why Excel Still Matters
</h2>

<p>After a week of working with Excel intensively, I can see why it remains a n important tool for analysts across industries. It is accessible. You do not need to set up a development environment or learn a programming language to start analyzing data. The visual, grid-based interface makes it easy to spot patterns and anomalies just by scrolling through your data.</p>

<p>It is also surprisingly powerful. Between formulas, pivot tables, conditional formatting, and the charting tools, you can go from raw data to a polished dashboard without ever leaving the application. For small to medium datasets, this is often all you need.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fddujiw14z2voeaiwjupu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fddujiw14z2voeaiwjupu.png" alt="Final Dashboard" width="800" height="478"></a></p>

