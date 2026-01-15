---
Title: Understand AWS IAM Identifiers
Description: 
Author: Mayur Bhatti
Date: 2026-01-15T21:18:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>When working with AWS Security, one thing that often confuses beginners is IAM Identifiers.</p>

<p>You may have seen terms like <strong>ARN</strong>, <strong>UserID</strong>, <strong>RoleID</strong>, and <strong>FriendlyName</strong>.</p>

<p>Wonder why AWS need so many identifiers for the same thing?</p>

<p>Here we will break it down clearly so you can understand what each identifier is, why it exists and when you will use it.</p>




<h2>
  
  
  What are AWS IAM Identifiers?
</h2>

<p>In AWS Identity and Access Management (IAM), every identity and resource must be uniquely identifiable, so AWS achieves this using different types of identifiers, each designed for a specific purpose:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F46zzqbz7zxzkgrnxohmt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F46zzqbz7zxzkgrnxohmt.png" alt=" " width="752" height="356"></a></p>

<p>In simple terms, consider:</p>

<ul>
<li>Friendly Name = Display Name</li>
<li>ARN = Full Address</li>
<li>Unique ID = Government-issued ID number</li>
</ul>




<p>Let's break each in detail:</p>

<h2>
  
  
  1. Friendly Name 
</h2>

<p>Friendly names are the names that we assign to IAM resources, such as</p>

<ul>
<li>IAM User</li>
<li>IAM Role</li>
<li>IAM Group</li>
<li>IAM Policy</li>
</ul>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>IAM user = “mayur”
IAM role = "ec2-s3-readonly-role”
</code></pre>

</div>



<p><strong>Why Friendly name exists:</strong></p>

<ul>
<li>Easy for humans to read and remember</li>
<li>Used in the AWS Console</li>
<li>Used in CLI commands and scripts</li>
</ul>

<p><strong>Important </strong></p>

<ul>
<li>Must be unique within the same account</li>
</ul>

<blockquote>
<p>Friendly names are not globally unique, so two AWS accounts can have the same friendly name.</p>
</blockquote>

<h2>
  
  
  2. ARN
</h2>

<p>An ARN is similar to a fully qualified domain name, a globally unique identifier.</p>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>arn:aws:iam::123456789012:user/mayur
</code></pre>

</div>



<p><strong>ARN structure:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>arn:partition:service:region:account-id:resource
</code></pre>

</div>



<p> <br>
<strong>Why ARNs matter:</strong></p>

<ul>
<li>Used in IAM policies</li>
<li>Used by AWS services internally</li>
<li>Required for cross-account access</li>
</ul>

<blockquote>
<p>AWS trust ARNs, not the friendly names.</p>
</blockquote>
<h2>
  
  
  3. Unique ID
</h2>

<p>Every IAM resource also gets a unique ID assigned by AWS.</p>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AIDAJQABLZS4A3QDU576Q
</code></pre>

</div>



<p><strong>Why AWS uses this:</strong></p>

<ul>
<li>Friendly names can change</li>
<li>ARNs can change if the path changes</li>
<li>Unique IDs ensure consistency </li>
</ul>

<blockquote>
<p>You will not use these directly, but AWS relies on them internally, as they cannot be changed or reused even if resources are deleted.</p>
</blockquote>

<h2>
  
  
  4. Paths
</h2>

<p>Paths allow you to logically group IAM resources</p>

<p><strong>Example:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/dev/admins/dev-admin
</code></pre>

</div>



<p><strong>Paths:</strong></p>

<ul>
<li>don’t affect permission</li>
<li>Help with Organisation</li>
<li>Are included in the ARN</li>
</ul>

<p><strong>When paths are useful:</strong></p>

<ul>
<li>Large Enterprises</li>
<li>Multiple teams</li>
<li>Environment separation</li>
</ul>




<h2>
  
  
  How these Identifiers work together:
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnqf73reopx0nlhbg5sxv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnqf73reopx0nlhbg5sxv.png" alt=" " width="782" height="428"></a></p>

<p>Each identifier serves a different audience:</p>

<ul>
<li>Humans —&gt; Friendly Names</li>
<li>Policies &amp; Services —&gt; ARNs</li>
<li>AWS Internal systems —&gt; Unique IDs</li>
</ul>




<h2>
  
  
  Common Mistakes to Avoid:
</h2>

<ul>
<li>Assuming friendly names are globally unique</li>
<li>Renaming IAM resources without checking the impact on other resources</li>
<li>Confusing role ARN with Instance profile ARN</li>
<li>Using wildcards carelessly in ARN policies</li>
</ul>

<p><strong>Once you understand why each identifier exists, IAM becomes easier and safer to manage.</strong></p>

