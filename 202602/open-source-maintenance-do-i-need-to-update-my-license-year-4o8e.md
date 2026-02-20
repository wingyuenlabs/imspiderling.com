---
Title: Open Source Maintenance: Do I Need to Update My License Year?
Description: 
Author: Romain Lespinasse
Date: 2026-02-20T22:05:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve been maintaining a project since 2019 and have reached a milestone like <strong>v5.x</strong>, you might wonder if that "2019" in your <code>LICENSE</code> file makes your project look abandoned. </p>

<p>Here is a quick guide on how to handle license dates effectively without overthinking it.</p>

<h2>
  
  
  The "First Publication" Rule
</h2>

<p>In an MIT or Apache license, the year (e.g., <code>Copyright (c) 2019</code>) marks the <strong>year of first publication</strong>. <br>
It isn't an expiration date. However, as your project evolves through major versions, your code changes significantly.</p>
<h3>
  
  
  Using the Copyright Range
</h3>

<p>Instead of choosing between the start year and the current year, the professional standard is to use a <strong>range</strong>.</p>

<ul>
<li>
<strong>Bad:</strong> <code>Copyright (c) 2019</code> (Looks abandoned)</li>
<li>
<strong>Better:</strong> <code>Copyright (c) 2026</code> (Wipes out the history of the original work)</li>
<li>
<strong>Best:</strong> <code>Copyright (c) 2019-2026</code> (Protects the original v1.0 and the current v5.x)</li>
</ul>
<h2>
  
  
  When to Update (The SemVer Strategy)
</h2>

<p>If you follow <strong>Semantic Versioning</strong>, you don't need to touch the license for every patch. </p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Version Type</th>
<th>Update License?</th>
<th>Reason</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Major (v5.0.0)</strong></td>
<td><strong>Highly Recommended</strong></td>
<td>Significant refactors or breaking changes are new "works."</td>
</tr>
<tr>
<td><strong>Minor (v5.1.0)</strong></td>
<td><strong>Recommended</strong></td>
<td>New features should be covered under the latest year.</td>
</tr>
<tr>
<td><strong>Patch (v5.1.1)</strong></td>
<td><strong>Optional</strong></td>
<td>Bug fixes rarely require a copyright update.</td>
</tr>
</tbody>
</table></div>
<h2>
  
  
  Automation: The "Set and Forget" Method
</h2>

<p>If you find manual updates tedious, you can use a GitHub Action to check your license at the start of every year.</p>

<p><a href="https://github.com/FantasticFiasco/action-update-license-year" rel="noopener noreferrer"><code>FantasticFiasco/action-update-license-year</code></a> GitHub Action will handle it for you.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Update copyright year(s) in license file</span>

<span class="na">on</span><span class="pi">:</span>
    <span class="na">schedule</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0</span><span class="nv"> </span><span class="s">3</span><span class="nv"> </span><span class="s">1</span><span class="nv"> </span><span class="s">1</span><span class="nv"> </span><span class="s">*'</span> <span class="c1"># 03:00 AM on January 1</span>

<span class="na">jobs</span><span class="pi">:</span>
    <span class="na">update-license-year</span><span class="pi">:</span>
        <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
        <span class="na">steps</span><span class="pi">:</span>
            <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v6</span>
              <span class="na">with</span><span class="pi">:</span>
                  <span class="na">fetch-depth</span><span class="pi">:</span> <span class="m">0</span>
            <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">FantasticFiasco/action-update-license-year@v3</span>
              <span class="na">with</span><span class="pi">:</span>
                  <span class="na">token</span><span class="pi">:</span> <span class="s">${{ secrets.GITHUB_TOKEN }}</span>
</code></pre>

</div>



<p>This action will detect your <code>2019</code> start date and automatically transform it into <code>2019-2026</code>, keeping your project looking active and professional.</p>

<h2>
  
  
  Summary
</h2>

<p>For a project spanning several years, simply update your license header to:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Copyright (c) 2019-2026 [Your Name]
</code></pre>

</div>



<p>This tells the world your project is established, battle-tested since 2019, and actively maintained in 2026.</p>

<blockquote>
<p>💡 Note<br>
This post was originally published on <a href="https://www.romainlespinasse.dev/posts/maintaining-license-years/" rel="noopener noreferrer">my personal blog</a>.</p>
</blockquote>

