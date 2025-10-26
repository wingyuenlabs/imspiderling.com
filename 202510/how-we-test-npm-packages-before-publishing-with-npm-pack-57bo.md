---
Title: How we test NPM packages before publishing with npm pack
Description: 
Author: Yoriiis
Date: 2025-10-26T21:54:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>When working on multiple interdependent NPM packages across projects, testing a new version before publishing is essential.</p>

<p>At Prisma Media, we maintain several packages on our private NPM registry, used directly on our websites. While each package comes with a standalone demo, we regularly need to validate development versions in real environments.</p>




<h2>
  
  
  Understanding <code>npm pack</code>
</h2>

<p>Running <code>npm pack</code> builds your package locally and generates a <code>.tgz</code> archive identical to what would be published with <code>npm publish</code>. The archive is created in the current directory and includes only the files defined in the <code>files</code> field of your <code>package.json</code> or filtered by <code>.npmignore</code>.</p>

<p>It's also a convenient way to verify what your package will actually ship before publishing. Inspecting the generated archive helps ensure that only the expected files are included.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdxpxrafmq6y8v9bu786.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxdxpxrafmq6y8v9bu786.png" alt=" raw `npm-pack` endraw  terminal output" width="800" height="732"></a></p>




<h2>
  
  
  Testing options
</h2>

<p>Developers have a few options to test packages locally:</p>

<ul>
<li>
<strong>Local linking</strong>: <code>npm link</code> works if both the package and consuming project are cloned locally</li>
<li>
<strong>Preview or shared environments</strong>: <code>npm link</code> fails here. Other options include:

<ul>
<li>Publishing a tagged version (like <code>beta</code>), it requires deployment to the registry</li>
<li>Using <code>npm pack</code>, it generates the <code>.tgz</code> archive that would be published, without touching the registry</li>
</ul>


</li>

</ul>

<p>We chose <code>npm pack</code> for its simplicity and safety. It allows developers and product teams to test new versions in local or preview environments, without affecting production workflows.</p>




<h2>
  
  
  Implementation
</h2>

<p>Our package pipelines follow a standard sequence: <code>install</code>, <code>build</code>, <code>test</code>, <code>pack</code> and finally <code>deploy</code>.</p>

<p>For details on our reusable GitLab CI template you can read</p>


<div class="ltag__link">
  <a href="/yoriiis" class="ltag__link__link">
    <div class="ltag__link__pic">
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F360690%2Ff2a06ade-26ab-4c55-9a57-67512b2c7275.jpeg" alt="yoriiis">
    </div>
  </a>
  <a href="https://dev.to/yoriiis/scaling-gitlab-cicd-with-reusable-templates-5ho0" class="ltag__link__link">
    <div class="ltag__link__content">
      <h2>Scaling GitLab CI/CD with reusable templates</h2>
      <h3>Yoriiis ・ Oct 15</h3>
      <div class="ltag__link__taglist">
        <span class="ltag__link__tag">#devops</span>
        <span class="ltag__link__tag">#gitlab</span>
        <span class="ltag__link__tag">#ci</span>
        <span class="ltag__link__tag">#automation</span>
      </div>
    </div>
  </a>
</div>


<p>The <code>npm pack</code> job is <strong>manual</strong>. When a developer needs a dev version:</p>

<ol>
<li>Navigate to the latest pipeline of the merge request</li>
<li>Trigger the <code>npm-pack</code> job</li>
<li>The job runs and uploads the TGZ as an artifact</li>
<li>Install and test locally:</li>
</ol>

<ul>
<li>Download and extract the artifact</li>
<li>Install the TGZ in your project:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   npm <span class="nb">install</span> &lt;path_to_tgz_file&gt;
</code></pre>

</div>



<ul>
<li>This replaces the version from the registry with the local TGZ version. Example diff:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code>   - "player": "1.0.0"
   + "player": "file:player-1.1.0.tgz"
</code></pre>

</div>



<ul>
<li>Commit the TGZ file to your branch <strong>if you plan to deploy a preview</strong>
</li>
<li>Once committed, the CI will install the TGZ</li>
</ul>

<h2>
  
  
  GitLab CI template
</h2>

<p>Here is a simplified <code>.gitlab-ci.yml</code> template for the <code>npm-pack</code> job:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">npm-pack</span><span class="pi">:</span>
  <span class="na">variables</span><span class="pi">:</span>
    <span class="na">PACKAGES_DIR</span><span class="pi">:</span> <span class="s1">'</span><span class="s">npm-pack-packages'</span>
  <span class="na">image</span><span class="pi">:</span> <span class="s">node:22-alpine</span>
  <span class="na">script</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">mkdir $PACKAGES_DIR</span>
    <span class="pi">-</span> <span class="s">PACKAGE_TGZ=$(npm pack)</span>
    <span class="pi">-</span> <span class="s">mv "$PACKAGE_TGZ" $PACKAGES_DIR</span>
  <span class="na">artifacts</span><span class="pi">:</span>
    <span class="na">paths</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">$PACKAGES_DIR</span>
  <span class="na">when</span><span class="pi">:</span> <span class="s">manual</span>
</code></pre>

</div>



<p>This job generates the TGZ locally and stores it as an artifact. The file can then be installed for local or preview testing.</p>

<p>For the GitHub Actions equivalent</p>

<p><a href="https://github.com/yoriiis/dev.to/tree/main/projects/npm-pack" class="ltag_cta ltag_cta--branded">Discover npm-pack workflow examples on GitHub</a>
</p>

<h2>
  
  
  Advantages
</h2>

<ul>
<li>Developers can test packages locally <strong>without publishing</strong>
</li>
<li>The TGZ can be versioned in the dev branch, allowing CI to install it on preview environments</li>
<li>Product teams can validate features before merging</li>
</ul>

<blockquote>
<p>⚠️ Remember: <strong>TGZ files are for development and preview only</strong>, never for production.</p>
</blockquote>




<h2>
  
  
  Key points
</h2>

<ul>
<li>
<code>npm pack</code> replaces <code>npm link</code> for shared or CI environments</li>
<li>When switching back to a registry version, always run <code>npm uninstall &lt;package&gt;</code> before reinstalling</li>
<li>Version TGZ files only for temporary testing; never use them in production</li>
<li>CI pipelines can safely produce TGZ artifacts for local or preview testing</li>
<li>Manual triggering keeps pipelines efficient</li>
</ul>




<h2>
  
  
  Conclusion
</h2>

<p>Using <code>npm pack</code> to test packages locally or in preview environments simplifies cross-project development. It sits between local linking and full registry publishing, giving developers and product teams a faster way to validate changes.</p>

<p>By integrating this workflow into CI, we can safely validate packages, maintain developer productivity and ensure quality before official publication.</p>

<p>The complete example is available on GitHub for you to try out! 🧑‍💻</p>

<p><a href="https://github.com/yoriiis/dev.to/tree/main/projects/npm-pack" class="ltag_cta ltag_cta--branded">GitHub npm-pack on GitHub</a>
</p>




<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://docs.npmjs.com/cli/v11/commands/npm-pack" rel="noopener noreferrer"><code>npm pack</code> documentation</a></li>
<li><a href="https://docs.npmjs.com/cli/v11/commands/npm-link" rel="noopener noreferrer"><code>npm link</code> documentation</a></li>
<li><a href="https://docs.npmjs.com/cli/v11/commands/npm-publish" rel="noopener noreferrer"><code>npm publish</code> documentation</a></li>
</ul>




