---
Title: How to Calculate a Dynamic Truncated Mean in Power BI Using DAX
Description: 
Author: Luca Liu
Date: 2026-01-06T21:57:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Why You Need a Truncated Mean
</h2>

<p>In data analysis, the standard AVERAGE function is a workhorse, but it has a significant weakness: it is highly susceptible to distortion from outliers. A single extreme value, whether high or low, can skew the entire result, misrepresenting the data's true central tendency.</p>

<p>This is where the truncated mean becomes essential. It provides a more robust measure of average by excluding a specified percentage of the smallest and largest values from the calculation.</p>

<p>While modern Power BI models have a built-in TRIMMEAN function, this function is often unavailable when using a Live Connection to an older Analysis Services (SSAS) model. This article provides a robust, manual DAX pattern that replicates this functionality and remains fully dynamic, responding to all slicers and filters in your report.</p>

<h2>
  
  
  The DAX Solution for a Dynamic Truncated Mean
</h2>

<p>This measure calculates a 20% truncated mean by removing the bottom 10% and top 10% of values before averaging the remaining 80%.</p>

<p>You can paste this code directly into the "New Measure" formula bar.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Trimmed Mean (20%) = 
VAR TargetTable = 'FactTable'
VAR TargetColumn = 'FactTable'[MeasureColumn]
VAR LowerPercentile = 0.10 // Defines the bottom 10% to trim
VAR UpperPercentile = 0.90 // Defines the top 10% to trim (1.0 - 0.10)

// 1. Find the value at the 10th percentile
VAR MinThreshold =
    PERCENTILEX.INC(
        FILTER( 
            TargetTable, 
            NOT( ISBLANK( TargetColumn ) ) 
        ),
        TargetColumn,
        LowerPercentile
    )

// 2. Find the value at the 90th percentile
VAR MaxThreshold =
    PERCENTILEX.INC(
        FILTER( 
            TargetTable, 
            NOT( ISBLANK( TargetColumn ) ) 
        ),
        TargetColumn,
        UpperPercentile
    )

// 3. Calculate the average, including only values between the thresholds
RETURN
CALCULATE(
    AVERAGEX(
        FILTER(
            TargetTable,
            TargetColumn &gt;= MinThreshold &amp;&amp;
            TargetColumn &lt;= MaxThreshold
        ),
        TargetColumn
    )
)
</code></pre>

</div>

<h2>
  
  
  Deconstructing the DAX Logic
</h2>

<p>This formula works in three distinct steps, all of which execute within the current filter context (e.g., whatever slicers the user has selected).</p>

<ol>
<li>Define Key Variables</li>
<li>
<code>TargetTable</code> &amp; <code>TargetColumn</code>: We assign the table and column names to variables for clean, reusable code. You must change 'FactTable'[MeasureColumn] to match your data model.</li>
<li>
<code>LowerPercentile</code> / <code>UpperPercentile</code>: We define the boundaries. 0.10 and 0.90 mean we are trimming the bottom 10% and top 10%. To trim 5% from each end (a 10% total trim), you would use 0.05 and 0.95.</li>
</ol>
<h3>
  
  
  2. Find the Percentile Thresholds
</h3>

<ul>
<li>
<code>MinThreshold</code> &amp; <code>MaxThreshold</code>: These variables store the actual values that correspond to our percentile boundaries.</li>
<li>
<code>PERCENTILEX.INC</code>: We use this "iterator" function because it allows us to first FILTER the table.</li>
<li>`FILTER(..., NOT(ISBLANK(...))): This is a crucial step. We calculate the percentiles only for rows where our target column is not blank. This prevents BLANK() values from skewing the percentile calculation.</li>
<li>The result is that <code>MinThreshold</code> holds the value of the 10th percentile (e.g., 4.5) and <code>MaxThreshold</code> holds the value of the 90th percentile (e.g., 88.2) for the currently visible data.</li>
</ul>
<h3>
  
  
  3. Calculate the Final Average
</h3>

<ul>
<li>
<code>RETURN CALCULATE(...)</code>: The CALCULATE function is the key to making the measure dynamic. It ensures the entire calculation respects the filters applied by any slicers or visuals in the report.</li>
<li><p><code>AVERAGEX(FILTER(...))</code>: The core of the calculation. We use AVERAGEX to iterate over a table.</p></li>
<li>
<p><code>FILTER(...)</code>: We filter our TargetTable a final time. This filter is the "trim." It keeps only the rows where the value in TargetColumn is:</p>

<ul>
<li>
<strong>Greater than or equal to</strong> our MinThreshold</li>
<li><strong>AND</strong></li>
<li>
<strong>Less than or equal to</strong> our MaxThreshold</li>
</ul>
</li>
<li><p><code>AVERAGEX(..., TargetColumn)</code>: <code>AVERAGEX</code> then calculates the simple average of <code>TargetColumn</code> for only the rows that passed the filter.</p></li>
</ul>
<h2>
  
  
  Conclusion
</h2>

<p>By implementing this DAX pattern, you create a robust, dynamic, and outlier-resistant KPI. This measure provides a more accurate picture of your data's central tendency and will correctly re-calculate on the fly as users interact with your Power BI report.</p>


<h2>
  
  
  Explore more
</h2>


<div class="ltag__user ltag__user__id__1230121">
    <a href="/luca1iu" class="ltag__user__link profile-image-link">
      <div class="ltag__user__pic">
        <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F1230121%2F2521cc84-ad7d-458c-99e5-b4d82f625a88.jpg" alt="luca1iu image">
      </div>
    </a>
  <div class="ltag__user__content">
    <h2>
<a class="ltag__user__link" href="/luca1iu">Luca Liu</a>Follow
</h2>
    <div class="ltag__user__summary">
      <a class="ltag__user__link" href="/luca1iu">Hello there! 👋 I'm Luca, a Business Intelligence Developer with passion for all things data. Proficient in Python, SQL, Power BI, Tableau, SAP Business Objects.</a>
    </div>
  </div>
</div>



<p>Thank you for taking the time to explore data-related insights with me. I appreciate your engagement.</p>

<p><a href="https://www.linkedin.com/in/lucaliu-data" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">🚀 Connect with me on LinkedIn</a>
</p>

<p><a href="https://twitter.com/Luca_DataTeam" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">🎃 Connect with me on X</a>
</p>

