---
Title: Configuring AWS Named Profiles
Description: 
Author: Lucas de Camargo
Date: 2025-12-13T21:34:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Following up with our series of engineering an IaC Startup on AWS, there's this one addendum I must talk about.</p>

<p>When working with multiple AWS accounts you need a way to tell the AWS CLI and Terraform which account to use for each operation. This is where <strong>named profiles</strong> come in.</p>

<p>A named profile is a collection of credentials and configuration settings that represents a specific identity in a specific AWS account. Instead of constantly switching credentials or passing them as command-line arguments, you define profiles once in your local AWS configuration files and reference them by name.</p>

<p>For example, you might have:</p>

<ul>
<li>
<code>management-admin</code> - Administrator access to your management account</li>
<li>
<code>backend-dev-admin</code> - Administrator access to your development account</li>
<li>
<code>backend-prod-developer</code> - Limited developer access to production</li>
</ul>

<p>Named profiles are essential when working with Terraform and Terragrunt. Each stack may need to deploy resources to a different AWS account or assume different roles. By using named profiles in your Terraform provider configuration, you explicitly declare which identity should be used:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">provider</span> <span class="s2">"aws"</span> <span class="p">{</span>
  <span class="nx">region</span>  <span class="p">=</span> <span class="s2">"us-east-1"</span>
  <span class="nx">profile</span> <span class="p">=</span> <span class="s2">"backend-dev-admin"</span>
<span class="p">}</span>
</code></pre>

</div>



<p>When Terraform runs, it looks for this profile in your local AWS configuration files and authenticates accordingly.</p>

<p>AWS profiles are configured in two files in your home directory:</p>

<p><strong><code>~/.aws/credentials</code></strong> - Stores authentication credentials (access keys or SSO configuration)</p>

<p><strong><code>~/.aws/config</code></strong> - Stores regional and output preferences</p>

<p>There are two primary ways to authenticate a profile.</p>

<h2>
  
  
  Single Sign-On (SSO) - Recommended
</h2>

<p>This is the secure, modern approach that aligns with AWS best practices and the governance foundation we established. Instead of long-lived access keys, SSO provides temporary credentials that automatically expire:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="c"># ~/.aws/credentials
</span><span class="nn">[profile management-admin]</span>
<span class="py">sso_start_url</span> <span class="p">=</span> <span class="s">https://mycorp.awsapps.com/start</span>
<span class="py">sso_region</span> <span class="p">=</span> <span class="s">us-east-1</span>
<span class="py">sso_account_id</span> <span class="p">=</span> <span class="s">123456789012</span>
<span class="py">sso_role_name</span> <span class="p">=</span> <span class="s">AdministratorAccess</span>
</code></pre>

</div>



<p>It is recommended to explicitly define the configuration values for each profile:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="c"># ~/.aws/config
</span><span class="nn">[profile management-admin]</span>
<span class="py">region</span> <span class="p">=</span> <span class="s">us-east-1</span>
<span class="py">output</span> <span class="p">=</span> <span class="s">json</span>
</code></pre>

</div>



<p>To use an SSO profile, you need to login with the AWS CLI before running Terraform or any CLI command, which opens a browser for authentication and caches temporary credentials.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws sso login <span class="nt">--profile</span> management-admin

<span class="c"># Test it with</span>
aws sts get-caller-identity <span class="nt">--profile</span> management-admin
</code></pre>

</div>



<h2>
  
  
  Access Keys
</h2>

<p>Long-lived access keys can be stored directly in <code>~/.aws/credentials</code>, but <strong>this approach is discouraged by AWS security best practices because</strong>:</p>

<ul>
<li>Keys don't expire automatically</li>
<li>If accidentally committed to Git or exposed, they remain valid until manually rotated</li>
<li>They don't provide the same audit trail as SSO
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight ini"><code><span class="c"># ~/.aws/credentials (avoid this approach if possible)
</span><span class="nn">[management-admin]</span>
<span class="py">aws_access_key_id</span> <span class="p">=</span> <span class="s">AKIAIOSFODNN7EXAMPLE</span>
<span class="py">aws_secret_access_key</span> <span class="p">=</span> <span class="s">wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY</span>
</code></pre>

</div>



<p><strong>Important:</strong> The <code>default</code> profile (with no <code>profile</code> prefix) is the profile used when no specific profile is specified. For safety, leave it unconfigured.</p>

<h2>
  
  
  Testing Profiles
</h2>

<p>Once you've configured your profiles, it's essential to verify they're working correctly before attempting to deploy infrastructure.</p>

<p>For SSO profiles, start by logging in:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Login to the SSO profile</span>
aws sso login <span class="nt">--profile</span> management-admin
</code></pre>

</div>



<p>This opens your browser for authentication. After successfully logging in, the credentials are cached locally (typically for 8-12 hours, depending on your SSO configuration).</p>

<p>For access key profiles, the test is simpler since no login is required.</p>

<p>Test your profiles by retrieving your caller identity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>aws sts get-caller-identity <span class="nt">--profile</span> management-admin
</code></pre>

</div>



<p>You should see output like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
    </span><span class="nl">"UserId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AROA...:user@example.com"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Account"</span><span class="p">:</span><span class="w"> </span><span class="s2">"123456789012"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Arn"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:sts::123456789012:assumed-role/AdministratorAccess/user@example.com"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>You can also list resources to verify permissions:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># List S3 buckets</span>
aws s3 <span class="nb">ls</span> <span class="nt">--profile</span> management-admin

<span class="c"># List EC2 instances in a specific region</span>
aws ec2 describe-instances <span class="nt">--region</span> us-east-1 <span class="nt">--profile</span> management-admin
</code></pre>

</div>



<p>If you see "UnauthorizedOperation" or "AccessDenied" errors, check that your IAM user or role has the necessary permissions attached.</p>

<h2>
  
  
  Deploying Governance before SSO
</h2>

<p>Here's the chicken and the egg! The problem with the first and recommended approach is that we need to first enable SSO in AWS and define an account for each user with the right accesss permissions. Without SSO, we're left with no choice rather using a temporary access key to work with IaC on AWS.</p>

<p>If creating an admin user in IAM with no restrictions feels too much and not a good governance approach for you, I have written a set of granular permissions that are needed for deploying our organization resources until we configure SSO.</p>

<p>Check out the permissions in my <a href="https://github.com/lucasdecamargo/terragrunt-live-example/tree/main/policies" rel="noopener noreferrer">GitHub repository</a> for the Terragrunt Live Environment example.</p>

<ul>
<li>In your AWS Console, go to IAM -&gt; Policies, and create a new policy for each of the JSON documents I have under the <code>policies</code> folder in my repository.</li>
<li>Next, go again to IAM -&gt; Users, create a new <code>terragrunt-root</code> user, and attach these policies to it.</li>
<li>Generate a new pair o acess keys, and add it your AWS credentials file, as we discussed, with some name of <code>acme-root</code>, for example.</li>
</ul>

<h2>
  
  
  What's Next?
</h2>

<p>Now that we've configured AWS named profiles for secure authentication across multiple accounts, we have everything we need to actually deploy our governance infrastructure.</p>

<p>Follow me to get updates about my series <em>IaC Startup on AWS</em>. I'm writing new posts about deploying governance infrastructure with Terragrunt and Terraform.</p>

<p>Also, make sure to check my <a href="https://github.com/lucasdecamargo" rel="noopener noreferrer">GitHub profile</a> for new projects and updates.</p>

<p>Buy me a coffee if you like this post! :)</p>

<p><a href="https://buymeacoffee.com/lucasdecamargo" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fcdn.buymeacoffee.com%2Fbuttons%2Fdefault-orange.png" alt="Buy Me A Coffee" width="434" height="100"></a></p>

