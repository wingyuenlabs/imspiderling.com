---
Title: Create a minimal site with Elucid8
Description: 
Author: Richard Hainsworth
Date: 2025-09-02T21:59:43.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Elucid8 system can be used to create websites based on RakuDoc. The article is to show how to create a minimal website.</p>

<p>Elucid8 ("elucidate") is still being developed, and more information can be found in the <a href="https://github.com/elucid8-org" rel="noopener noreferrer">Github elucid8-org repositories</a>.</p>

<p>The following need to be present:</p>

<ol>
<li>A recent version of Rakudo v.2025.01 or later.</li>
<li>Dart sass - <a href="https://sass-lang.com/dart-sass/" rel="noopener noreferrer">see Sass website</a> for installation instructions.</li>
<li>Elucid8::Build - <code>zef install Elucid8::Build</code>
</li>
<li>Elucid8::Run-locally - <code>zef install Elucid8::Run-locally</code>
</li>
</ol>

<p>Assuming that the <code>zef</code> installs <code>bin/</code> files to a location in the PATH, then the utilities in the next section should run without problem.</p>

<h2>
  
  
  Setting up minimal site
</h2>

<p>In an empty directory (to be concrete, lets call it <code>webdir</code>), which will be the root for the website build, run the following</p>

<ol>
<li><code>eluci8-setup</code></li>
<li><code>gather-sources</code></li>
<li><code>elucid8-build</code></li>
<li><code>run-locally</code></li>
</ol>

<p>Then point a browser at <code>localhost:5000</code></p>

<p>That is all for the minimum site.</p>

<p>Now change the file <code>site-source/en/index.rakudoc</code>.</p>

<p>Then run <code>elucid8-build; run-locally</code> again to see the changes.</p>

<h3>
  
  
  Explanation of steps
</h3>

<h4>
  
  
  Step 1 Minimal files
</h4>

<p>Here <em>elucid8-setup</em> copies resources in the <code>Elucid8::Build</code> distribution to create the minimal configuration entries in <code>webdir</code>,<br>
a sample text and some minimal plugins.</p>

<p>The directory structure under <code>webdir</code> will be something like<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>- config/
  - 01-base.raku
  - 02-plugins.raku
  - 03-plugin-options.raku
  - 04-repos
- misc/
- site-sources/
  - en/
    - index.rakudoc
    - examples.rakudoc

</code></pre>

</div>



<blockquote>
<p>Since Elucid8 is being built from the bottom up to be multi-lingual, it is intended that all of these directories can be named in a local variation. The file <code>config/01-base.raku</code> contains the tokens that are used within <em>Elucid8::Build</em>.</p>
</blockquote>

<h4>
  
  
  Step 2. Gathering sources
</h4>

<p>It is intended that each set of language sources will be in a different repo. <code>gather-sources</code> looks at <code>config/04-repos</code> for files and repo information, clones repositories, runs <code>git blame</code> against them and stores a file description in <code>misc/</code>.</p>

<p>After this step, <code>website</code> will contain the file <code>misc/repo-info.rakuon</code>.</p>

<p>For the minimal website, the RakuDoc v2 specification is pulled from the Raku repository. This document also shows many of the features of RakuDoc.</p>

<p>The <code>config/04-repos</code> shows an example of how to map documents from the repository to subdirectories within <code>publication</code>.</p>

<h4>
  
  
  Step 3 Build
</h4>

<p>At this step, the build process starts. A processor engine is created that uses the plugins described in <code>config/02-plugins</code>. These are run with options described in <code>config/03-plugin-options</code>.</p>

<p>After this step, <code>misc/</code> also contains <code>ui-dictionary.rakuon</code>, which will have the English version of the UI tokens. When this dictionary is appropriately edited, other languages will be available.</p>

<blockquote>
<p>Elucid8 makes a distinction between the language of the UI and the language of the contents.</p>
</blockquote>

<p>A new directory <code>publication/</code> has now appeared, and this will contain the HTML version of the index file. </p>

<h3>
  
  
  Step 4
</h3>

<p>At this step, a Cro app is run that takes the HTML in <code>publication</code> and serves the files to <code>localhost:5000</code>.</p>

<h2>
  
  
  Customisation
</h2>

<p>There are many ways to customise the site:</p>

<ul>
<li>make sure to change <code>config/03-plugin-options</code> and the <code>root-domain</code> field of SiteMap, so that the SEO map points to your website.</li>
<li>adding RakuDoc sources, remember to add links in index.rakudoc</li>
<li>adding plugins to create new RakuDoc blocks</li>
<li>use the ListFiles block to automatically include in <code>index.rakudoc</code> all the other RakuDoc sources in your <code>local-sources/</code> directory.</li>
</ul>

<p>Examples of what can be done can be seen in <a href="https://github.com/elucid8-org/sandpit" rel="noopener noreferrer">Sandpit repo, for Raku documentation</a></p>

