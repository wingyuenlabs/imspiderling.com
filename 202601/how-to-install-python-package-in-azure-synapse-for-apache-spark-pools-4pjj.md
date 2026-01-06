---
Title: How to Install Python Package in Azure Synapse for Apache Spark pools
Description: 
Author: Luca Liu
Date: 2026-01-06T21:58:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Efficiently Installing Python Packages in Azure Synapse Analytics
</h2>

<p>When working in Azure Synapse notebooks, you can use the %pip command (e.g., %pip install pandas) in a code cell to install packages. However, this method is temporary. The package is only installed for the current notebook session and must be re-installed every time the session starts.</p>

<p>This repetition can lead to significant delays in notebook execution and is inefficient for frequently run jobs.</p>

<p>A more permanent and efficient solution is to install packages directly onto the Apache Spark pool. This approach ensures the libraries are pre-installed and automatically available in every session attached to that pool.</p>

<h2>
  
  
  How to Install Packages at the Spark Pool Level
</h2>

<p>This method involves uploading a requirements.txt file that specifies the packages and versions you need.</p>

<ol>
<li>Go to your Azure Synapse workspace in the Azure portal.</li>
<li>Navigate to the "Manage" section on the left-hand side.</li>
<li>Select "Apache Spark pools" under the "Analytics pools" section.</li>
<li>Choose the Spark pool where you want to install the package.</li>
<li>move your mouth to the three dots on the right side of the Spark pool and click on "Packages".</li>
<li>upload <code>requirements.txt</code> file which contains the list of packages you want to install. </li>
<li>Click Apply to save the changes.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Futjmsqs39tv57h4az884.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Futjmsqs39tv57h4az884.png" width="800" height="498"></a></p>

<p>The Spark pool will update and automatically install the specified packages. This may take a few minutes. Once complete, all notebooks attached to this pool will have access to these libraries by default.</p>

<h2>
  
  
  How to generate <code>requirements.txt</code> file
</h2>

<p>The requirements.txt file is a simple text file that lists the packages to be installed. You can easily generate this file from your local Python environment.</p>

<p>Open your terminal or command prompt and run the following command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>pip freeze <span class="o">&gt;</span> requirements.txt
</code></pre>

</div>


<p>This command captures all packages and their exact versions from your current environment and saves them into a file named requirements.txt. Uploading this file ensures that the exact same package versions are installed in your Synapse environment, providing consistency and preventing dependency conflicts.</p>


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

