---
Title: When Disk Space is Not Enough: A Lesson in Challenging MySQL Database Cleansing
Description: 
Author: Taufiq Abdullah
Date: 2026-02-25T21:31:11.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flbkbm1kg9ko6zlcviw85.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flbkbm1kg9ko6zlcviw85.webp" alt=" " width="640" height="640"></a></p>

<p>Database maintenance is rarely a walk in the park, especially when dealing with high-volume transactional data. Recently, We was tasked with cleansing a MySQL database that had ballooned to <strong>220GB</strong>. The mission was simple: remove data from 2018 to 2023 across four tables to reclaim space and improve performance.</p>

<p>However, as any Dev knows, "simple" tasks often hide complex traps. Here is how we navigated storage limits and technical hurdles to get the job done.</p>




<h2>
  
  
  The Challenge: The 200GB "Elephant" in the Room
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqe0h2c5yz7u1fn4zqthx.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqe0h2c5yz7u1fn4zqthx.jpg" alt=" " width="800" height="538"></a></p>

<p>Out of the 220GB total size, one single table accounted for <strong>200GB</strong>. This table was heavy with transactional records and—the real culprit—<strong>BLOB attachments</strong>. Our goal was to keep only the data from 2024 onwards.</p>

<h3>
  
  
  Phase 1: The "Clone and Swap" Strategy (The Ideal Plan)
</h3>

<p>Our first approach was a classic "blue-green" table migration:</p>

<ol>
<li>
<strong>Clone:</strong> Create a clone of the existing tables with a new name.</li>
<li>
<strong>Filter:</strong> Insert only the 2024–present data into the new tables.</li>
<li>
<strong>Swap:</strong> Rename the existing tables and swap them with the new ones.</li>
<li>
<strong>Cleanup:</strong> Drop the old tables once verified.</li>
</ol>

<ul>
<li>
<strong>The Pros:</strong> It’s safe. If something goes wrong, the original data remains untouched as a "checkpoint."</li>
<li>
<strong>The Result in Dev:</strong> Flawless. Despite the time required for backup and restore, it worked exactly as expected.</li>
</ul>

<h3>
  
  
  The Production Reality Check
</h3>

<p>When we moved to production, we hit a brick wall. The production server had a <strong>500GB total capacity</strong>, but only <strong>100GB of free space</strong> remained. Trying to clone a 200GB table into a 100GB hole is mathematically impossible. We needed a Plan B.</p>




<h2>
  
  
  Phase 2: The "Chunked Delete" Strategy (The Safe Bet?)
</h2>

<p>We pivoted to a more granular approach: deleting data directly from the existing tables. To prevent long-running queries or locking the database, we followed these steps:</p>

<ul>
<li>Deleted data in year-by-year ranges (2018, 2019, etc.).</li>
<li>Used <strong>Bash scripts</strong> to automate chunked deletions with <code>LIMIT</code> clauses to keep the system responsive and avoid "hanging" the DB.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7m13ik6cfq20j3hn5h3.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7m13ik6cfq20j3hn5h3.jpeg" alt=" " width="792" height="966"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F62w0l4et4kzsd63hv1fi.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F62w0l4et4kzsd63hv1fi.jpeg" alt=" " width="792" height="234"></a></p>

<h3>
  
  
  The Trap: The <code>OPTIMIZE TABLE</code> Paradox
</h3>

<p>The deletion worked, but MySQL doesn't automatically shrink the file size on disk after a <code>DELETE</code> (due to the "High Water Mark" and data fragmentation). To actually reclaim the 100GB+ of space, we needed to run <code>OPTIMIZE TABLE</code>.</p>

<blockquote>
<p><strong>The Critical Oversight:</strong> <code>OPTIMIZE TABLE</code> works by creating a temporary copy of the table. Even though we had deleted half the data, the operation still required enough free space to rewrite the entire table. Once again, our 100GB of free space was the bottleneck. We were stuck.</p>
</blockquote>




<h2>
  
  
  Phase 3: The "Off-Site Surgery" (The Final Solution)
</h2>

<p>Desperate times call for creative architecture. We decided to perform the "heavy lifting" on a separate environment.</p>

<ol>
<li> <strong>Export:</strong> We dumped the massive 200GB table and moved it to a secondary server with enough storage.</li>
<li> <strong>Cleanse &amp; Shrink:</strong> On the secondary server, we performed the deletions and ran <code>OPTIMIZE TABLE</code>.</li>
<li> <strong>The Result:</strong> The optimized table dropped significantly below 100GB.</li>
<li> <strong>Import:</strong> We transferred the now-compacted table back to the production server under a new name.</li>
<li> <strong>The Final Swap:</strong> Since the new table was now small enough to fit in the 100GB free space, we successfully swapped it with the original and dropped the old 200GB giant.</li>
</ol>




<h2>
  
  
  Key Takeaways for us
</h2>

<ul>
<li>
<strong>Free Space is Your Best Friend:</strong> Always check your <code>df -h</code> before planning a migration. Most MySQL maintenance operations (like <code>OPTIMIZE</code> or <code>ALTER</code>) require at least <strong>2x the size</strong> of your largest table in free space.</li>
<li>
<strong>Optimize with Caution:</strong> Remember that <code>OPTIMIZE TABLE</code> is not an "in-place" fix; it is a rebuild.</li>
<li>
<strong>Think Outside the Box:</strong> If your local disk is full, use a secondary staging server to process the data before bringing it back home.</li>
</ul>

<p>Database cleansing is as much about managing infrastructure as it is about writing SQL. Have you ever faced a storage "deadlock" like this? I’d love to hear how you handled it in the comments!</p>

