---
Title: Manual version bumps using semantic release with Azure DevOps
Description: 
Author: Daniel Marques
Date: 2025-12-28T21:43:54.000Z
Robots: noindex,nofollow
Template: index
---
<p>The semantic-release package (<a href="https://github.com/semantic-release/semantic-release" rel="noopener noreferrer">https://github.com/semantic-release/semantic-release</a>) automates the version management and release process of a project. It determines the next version number based on the commit messages that adhere to the Conventional Commits specification, generates release notes, and publishes the release automatically.</p>

<p>However, some developers prefer more control over when to increment major and minor versions. Companies like JetBrains use the year as a major version and an auto-incrementing integer as a minor version as illustrated in the image below.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb466nrztrxka14etreoc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb466nrztrxka14etreoc.png" alt="JetBrain versioning" width="800" height="498"></a></p>

<p>Since semantic-release offers various built-in tools, such as a release notes generator, it is interesting to keep using it and just change the bumping logic. That is what I will show in this post.</p>

<h1>
  
  
  Azure DevOps pipeline for Manual bumping
</h1>

<p>My approach involves creating an Azure DevOps pipeline that runs the semantic-release with the following steps:</p>

<ol>
<li>The pipeline prompts the user to specify the bump type (major, minor, or patch) and the desired version number.</li>
<li>Once the user confirms the choice, if the user opted for a major or minor version, the pipeline transitions to an approval state.</li>
<li>Upon approval, the pipeline increments the version according to the chosen bump type.</li>
<li>The pipeline verifies that the bump aligns with the desired version number.</li>
</ol>

<p>The code snipet below shows the Azure Devops pipeline YAML with steps above. The parameters is used to request user bump type (default bump type set to patch) and wished version. Its values is set as environment variables that are used at semantic release configuration.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">parameters</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">bumpType</span>
    <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
    <span class="na">default</span><span class="pi">:</span> <span class="s2">"</span><span class="s">patch"</span>
    <span class="na">values</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">major</span>
      <span class="pi">-</span> <span class="s">minor</span>
      <span class="pi">-</span> <span class="s">patch</span>
  <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">bumpNumber</span>
    <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
    <span class="na">default</span><span class="pi">:</span> <span class="s2">"</span><span class="s">0"</span>

<span class="na">pool</span><span class="pi">:</span>
  <span class="na">vmImage</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>

<span class="na">jobs</span><span class="pi">:</span>
<span class="pi">-</span> <span class="na">job</span><span class="pi">:</span> <span class="s">approval</span>
  <span class="na">pool</span><span class="pi">:</span> <span class="s">server</span>
  <span class="na">steps</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">ManualValidation@1</span>
      <span class="s">...</span>
      <span class="na">condition</span><span class="pi">:</span> <span class="s">ne('${{ parameters.bumpType }}', 'patch')</span>
<span class="pi">-</span> <span class="na">job</span><span class="pi">:</span> <span class="s">create_tag</span>
  <span class="na">dependsOn</span><span class="pi">:</span> <span class="s">approval</span>
  <span class="na">steps</span><span class="pi">:</span>

   <span class="s">...</span>

    <span class="s">- script</span><span class="err">:</span> <span class="pi">|</span>
        <span class="s">npx semantic-release</span>
      <span class="na">displayName</span><span class="pi">:</span> <span class="s">Run semantic-release setting ${{ parameters.bumpType }} version to ${{ parameters.bumpNumber }}</span>
      <span class="na">env</span><span class="pi">:</span>
        <span class="na">SEMANTIC_RELEASE_BUMP_TYPE</span><span class="pi">:</span> <span class="s">${{ parameters.bumpType }}</span>
        <span class="na">SEMANTIC_RELEASE_BUMP_NUMBER</span><span class="pi">:</span> <span class="s">${{ parameters.bumpNumber }</span>

</code></pre>

</div>



<p>That’s it! With this setup, you can manually bump the versions of your project without having to worry about commit messages.</p>

<p>The following code snipet, show the semantic release configuration plugin configuration at <strong>release.config.cjs</strong>. For the plugin <strong>@semantic-release/commit-analyzer</strong> (which bumps version according to commit messages) is set to always increase the version according to bump type options choosen by the user.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// file release.config.cjs</span>
<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">{</span>
  <span class="p">...</span>
  <span class="na">plugins</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">...</span>
    <span class="p">[</span><span class="dl">'</span><span class="s1">@semantic-release/commit-analyzer</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">releaseRules</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span> <span class="na">release</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SEMANTIC_RELEASE_BUMP_TYPE</span> <span class="p">}</span>
    <span class="p">]</span> <span class="p">}],</span>
    <span class="dl">'</span><span class="s1">./verify-release.js</span><span class="dl">'</span>
  <span class="p">]</span>
<span class="p">};</span>
</code></pre>

</div>



<p>The <strong>verify-release.js</strong> plugin verifies if the new version is incremented as expected. This ensures that if the pipeline is executed with the same input for the second time, it will fail because the bump will go to an undesired value (taking the JetBrains example, setting the major version to the next year). You can see the verify-release.js code in the next snippet.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// file verify-release.js</span>
<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">verifyRelease</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">pluginConfig</span><span class="p">,</span> <span class="nx">context</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="kd">const</span> <span class="p">{</span> <span class="nx">lastRelease</span> <span class="o">=</span> <span class="p">{},</span> <span class="nx">nextRelease</span> <span class="o">=</span> <span class="p">{},</span> <span class="nx">logger</span> <span class="o">=</span> <span class="nx">console</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">context</span><span class="p">;</span>
        <span class="kd">const</span> <span class="nx">bumpType</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SEMANTIC_RELEASE_BUMP_TYPE</span><span class="p">;</span>
        <span class="kd">const</span> <span class="nx">bumpNumber</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">SEMANTIC_RELEASE_BUMP_NUMBER</span><span class="p">;</span>
        <span class="nx">logger</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Verifying expected release.</span><span class="dl">'</span><span class="p">);</span>
        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">bumpType</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">logger</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">SEMANTIC_RELEASE_BUMP_TYPE not set — skipping version verification.</span><span class="dl">'</span><span class="p">);</span>
          <span class="k">return</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">bumpType</span> <span class="o">==</span> <span class="dl">'</span><span class="s1">patch</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">logger</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">'</span><span class="s1">Bump type set to patch. Nothing to verify.</span><span class="dl">'</span><span class="p">);</span>
          <span class="k">return</span><span class="p">;</span>
        <span class="p">}</span>
        <span class="kd">const</span> <span class="nx">actual</span> <span class="o">=</span> <span class="nx">nextRelease</span> <span class="o">&amp;&amp;</span> <span class="nx">nextRelease</span><span class="p">.</span><span class="nx">version</span><span class="p">;</span>
        <span class="kd">const</span> <span class="nx">match</span> <span class="o">=</span> <span class="nx">actual</span><span class="p">.</span><span class="nf">match</span><span class="p">(</span><span class="sr">/^</span><span class="se">(\d</span><span class="sr">+</span><span class="se">)\.(\d</span><span class="sr">+</span><span class="se">)\.\d</span><span class="sr">+$/</span><span class="p">);</span>

        <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">match</span><span class="p">)</span> <span class="p">{</span>
          <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Invalid tag format: </span><span class="p">${</span><span class="nx">lastRelease</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="kd">const</span> <span class="nx">actualMajor</span> <span class="o">=</span> <span class="nc">Number</span><span class="p">(</span><span class="nx">match</span><span class="p">[</span><span class="mi">1</span><span class="p">]);</span>
        <span class="kd">const</span> <span class="nx">actualMinor</span> <span class="o">=</span> <span class="nc">Number</span><span class="p">(</span><span class="nx">match</span><span class="p">[</span><span class="mi">2</span><span class="p">]);</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">bumpType</span> <span class="o">==</span> <span class="dl">'</span><span class="s1">major</span><span class="dl">'</span> <span class="o">&amp;&amp;</span> <span class="nx">actualMajor</span> <span class="o">!=</span> <span class="nx">bumpNumber</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="s2">`Major version mismatch: expected </span><span class="p">${</span><span class="nx">bumpNumber</span><span class="p">}</span><span class="s2"> but will publish </span><span class="p">${</span><span class="nx">actualMajor</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
          <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Version verification failed: expected major version </span><span class="p">${</span><span class="nx">bumpNumber</span><span class="p">}</span><span class="s2">, got </span><span class="p">${</span><span class="nx">actualMajor</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
        <span class="p">}</span>
        <span class="k">if </span><span class="p">(</span><span class="nx">bumpType</span> <span class="o">==</span> <span class="dl">'</span><span class="s1">minor</span><span class="dl">'</span> <span class="o">&amp;&amp;</span> <span class="nx">actualMinor</span> <span class="o">!=</span> <span class="nx">bumpNumber</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">logger</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="s2">`Minor version mismatch: expected </span><span class="p">${</span><span class="nx">bumpNumber</span><span class="p">}</span><span class="s2"> but will publish </span><span class="p">${</span><span class="nx">actualMinor</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
          <span class="k">throw</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="s2">`Version verification failed: expected minor version </span><span class="p">${</span><span class="nx">bumpNumber</span><span class="p">}</span><span class="s2">, got </span><span class="p">${</span><span class="nx">actualMinor</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
        <span class="p">}</span>
        <span class="nx">logger</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Version verification passed: </span><span class="p">${</span><span class="nx">actual</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
      <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The picture below shows one example of pipeline execution that bumped major version.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2grjz8ru5nv2v1awde0h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2grjz8ru5nv2v1awde0h.png" alt=" " width="800" height="347"></a></p>

<h1>
  
  
  Building new version
</h1>

<p>Once the release is ready, you can begin building your application. For this example, I’ve created a simple hello world CLI in Go.</p>

<p>The pipeline’s steps are as follows:</p>

<ol>
<li>Checkout the code with tags</li>
<li>Gets a tag-based description of the current commit. If the commit lacks a tag, it creates a descriptive version based on the latest tag.</li>
<li>Sets Azure DevOps build number to the description from previous step</li>
<li>Generates cli executable and publishes it as build artifact.</li>
</ol>

<p>Here’s a code snippet that shows the above pipeline:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code>
<span class="na">variables</span><span class="pi">:</span>
  <span class="na">appName</span><span class="pi">:</span> <span class="s">hello-world</span>
  <span class="na">buildDir</span><span class="pi">:</span> <span class="s">build</span>

<span class="na">steps</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">checkout</span><span class="pi">:</span> <span class="s">self</span>
    <span class="na">fetchTags</span><span class="pi">:</span> <span class="kc">true</span>
    <span class="na">fetchDepth</span><span class="pi">:</span> <span class="m">0</span>

  <span class="pi">-</span> <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
      <span class="s">export VERSION=$(git describe --tags)</span>
      <span class="s">echo "##vso[build.updatebuildnumber]${VERSION}"</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Set</span><span class="nv"> </span><span class="s">build</span><span class="nv"> </span><span class="s">number"</span>

  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">GoTool@0</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">1.25"</span>

  <span class="pi">-</span> <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
      <span class="s">mkdir -p $(buildDir)</span>
      <span class="s">go build -o $(buildDir)/$(appName) ./cmd</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Build</span><span class="nv"> </span><span class="s">Go</span><span class="nv"> </span><span class="s">binary"</span>

  <span class="pi">-</span> <span class="na">script</span><span class="pi">:</span> <span class="pi">|</span>
      <span class="s">cd $(buildDir)</span>
      <span class="s">zip $(appName)-$(Build.BuildNumber).zip $(appName)</span>
    <span class="na">displayName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Create</span><span class="nv"> </span><span class="s">ZIP</span><span class="nv"> </span><span class="s">with</span><span class="nv"> </span><span class="s">version"</span>

  <span class="pi">-</span> <span class="na">task</span><span class="pi">:</span> <span class="s">PublishBuildArtifacts@1</span>
    <span class="na">inputs</span><span class="pi">:</span>
      <span class="na">PathtoPublish</span><span class="pi">:</span> <span class="s2">"</span><span class="s">$(buildDir)"</span>
      <span class="na">ArtifactName</span><span class="pi">:</span> <span class="s2">"</span><span class="s">release"</span>
      <span class="na">publishLocation</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Container"</span>

</code></pre>

</div>



<p>The following image illustrates one successful build.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2zq4gljkfeyhfauoc1o6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2zq4gljkfeyhfauoc1o6.png" alt=" " width="800" height="600"></a></p>

<p>The code used in this post is available in the github repository <a href="https://github.com/dmo2000/semantic-release-manual" rel="noopener noreferrer">https://github.com/dmo2000/semantic-release-manual</a></p>

