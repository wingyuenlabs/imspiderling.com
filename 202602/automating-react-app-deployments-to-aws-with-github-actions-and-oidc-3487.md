---
Title: Automating React App deployments to AWS with GitHub Actions and OIDC
Description: 
Author: Francisco Silvério
Date: 2026-02-01T21:26:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>After deploying my <a href="https://portfolio.franciscogsilverio.com/" rel="noopener noreferrer">personal website</a> to AWS, I realized the need to automate the deployment process to streamline the delivery of new features. For that, you can’t go wrong with a CI/CD pipeline powered by your Git provider — in this case, GitHub and GitHub Actions.</p>

<p>In this article, I’ll walk through the architecture and CI/CD pipeline I use to deploy a React application to S3 + CloudFront, authenticated via GitHub Actions OIDC (no long-lived AWS credentials).</p>

<p>This setup is suitable for real-world projects and follows AWS and GitHub best practices.</p>




<h2>
  
  
  Architecture Overview
</h2>

<p>Before diving into CI/CD, let’s clarify the infrastructure that was <strong>already in place</strong>.</p>




<h2>
  
  
  Initial AWS Setup (Pre-requisites)
</h2>

<p>The following resources were already created before the pipeline was written.</p>

<p><strong>1 - S3 bucket</strong></p>

<ul>
<li>Static website hosting enabled</li>
<li>Public access configured appropriately</li>
<li>Hosts the built React assets (index.html, /assets/*, etc.)</li>
</ul>

<p><strong>2 - CloudFront distribution</strong></p>

<ul>
<li>Origin pointing to the S3 website endpoint</li>
<li>Default root object: index.html</li>
<li>Caching enabled</li>
<li>Custom domain configured</li>
</ul>

<p><strong>3 - Name.com</strong></p>

<ul>
<li>Custom domain pointing to the CloudFront distribution</li>
</ul>

<p>Once this setup is complete, the website can already be accessed via the custom domain.<br>
The CI/CD pipeline’s job is to automate updates safely and consistently.</p>


<h2>
  
  
  AWS: Configuring OIDC Authentication
</h2>

<p><strong>1 - Create an OIDC Identity Provider</strong></p>

<p>In AWS IAM:</p>

<ul>
<li>Provider URL:
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://token.actions.githubusercontent.com
</code></pre>

</div>


<ul>
<li>Audience:
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sts.amazonaws.com
</code></pre>

</div>


<p>This allows AWS to trust GitHub as an identity provider.</p>



<p><strong>2 - Create an IAM Role for GitHub Action</strong></p>

<p>This role will be assumed by the GitHub Actions runner.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2012-10-17"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Statement"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Principal"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"Federated"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:iam::&lt;ACCOUNT_ID&gt;:oidc-provider/token.actions.githubusercontent.com"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"sts:AssumeRoleWithWebIdentity"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Condition"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"StringEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"token.actions.githubusercontent.com:aud"</span><span class="p">:</span><span class="w"> </span><span class="s2">"sts.amazonaws.com"</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"StringLike"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"token.actions.githubusercontent.com:sub"</span><span class="p">:</span><span class="w"> </span><span class="s2">"repo:&lt;ORG&gt;/&lt;REPO&gt;:*"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">

</span></code></pre>

</div>



<p><strong>3 - Attach Required IAM Permissions</strong></p>

<p>The role needs permissions to:</p>

<ul>
<li>Sync files to S3</li>
<li>Create CloudFront invalidations</li>
</ul>

<p>Example policy (simplified):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"2012-10-17"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"Statement"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"s3:PutObject"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"s3:DeleteObject"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"s3:ListBucket"</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"arn:aws:s3:::&lt;BUCKET_NAME&gt;"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"arn:aws:s3:::&lt;BUCKET_NAME&gt;/*"</span><span class="w">
      </span><span class="p">]</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Effect"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Allow"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"cloudfront:CreateInvalidation"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"*"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">

</span></code></pre>

</div>






<h2>
  
  
  GitHub Actions Pipeline Overview
</h2>

<p>The pipeline is split into three jobs:</p>

<ol>
<li>install – install dependencies</li>
<li>build – build the React app and upload artifacts</li>
<li>deploy – authenticate to AWS, deploy to S3, invalidate CloudFront</li>
</ol>

<p>This separation improves clarity, caching, and debuggability.</p>




<p><strong>Job 1: Installing Dependencies</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>install:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install dependencies
      uses: actions/setup-node@v4
      with:
        node-version: "22.x"
        cache: 'npm'
    - run: npm ci
</code></pre>

</div>



<p><strong>What this job does</strong></p>

<ol>
<li>Checks out the repository</li>
<li>Sets up Node.js 22</li>
<li>Uses npm ci for deterministic installs</li>
<li>Enables dependency caching for faster builds</li>
</ol>

<p>This job ensures the dependency tree is valid before moving forward.</p>

<p><strong>Job 2: Building the React Application</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>build:
  runs-on: ubuntu-latest
  needs: install
  steps:
    - uses: actions/checkout@v4
    - name: Use Node.js 22
      uses: actions/setup-node@v4
      with:
        node-version: "22.x"
        cache: 'npm'
    - run: npm ci
    - run: npm run build
    - name: Upload dist
      uses: actions/upload-artifact@v4
      with:
        name: dist
        path: ./dist
</code></pre>

</div>



<p><strong>What this job does</strong></p>

<ul>
<li>Rebuilds the application in a clean environment</li>
<li>Produces a static dist/ folder</li>
<li>Uploads the build output as a pipeline artifact</li>
</ul>

<p>Artifacts allow the deploy job to be fully decoupled from the build process.</p>

<p><strong>Job 3: Deploying to AWS (S3 + CloudFront)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: aws
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Download dist artifact
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ARN }}
          aws-region: ${{ vars.AWS_REGION }}
          role-session-name: ${{ vars.ROLE_SESSION_NAME}}

      - name: Deploy to S3
        run: |
          aws s3 sync dist s3://${{ vars.S3_BUCKET_NAME }} \
            --delete \
            --exact-timestamps

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"

</code></pre>

</div>



<p><strong>What this job does</strong></p>

<ul>
<li>Downloads the build artifact</li>
<li>Authenticates to AWS</li>
<li>Uploads the static files to the bucket</li>
<li>Invalidate CloudFront's cache</li>
</ul>

<p>Invalidating /* ensures CloudFront fetches the new version immediately.</p>




<h2>
  
  
  Final Result
</h2>

<p>After every push to main:</p>

<ul>
<li>The app is built</li>
<li>Assets are uploaded to S3</li>
<li>CloudFront cache is invalidated</li>
<li>The custom domain serves the latest version reliably</li>
</ul>

<p>All without storing a single AWS secret in GitHub.</p>




<h2>
  
  
  References
</h2>

<p><a href="https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect" rel="noopener noreferrer">- GitHub Actions OIDC</a></p>

<p><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html" rel="noopener noreferrer">- AWS IAM OIDC</a></p>

<p><a href="https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html" rel="noopener noreferrer">- CloudFront invalidations</a></p>

<p><a href="https://github.com/aws-actions/configure-aws-credentials" rel="noopener noreferrer">- aws-actions/configure-aws-credentials</a></p>

