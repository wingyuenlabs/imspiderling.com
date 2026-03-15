---
Title: Publishing OAS-Based API Doc on GitHub Pages
Description: 
Author: Kengo Hakomori
Date: 2026-03-15T22:00:22.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Environment
</h2>

<ul>
<li>
<p>Mac</p>

<ul>
<li>CPU: Apple M4</li>
<li>OS: Sequoia 15.1.1</li>
</ul>


</li>

<li>

<p><a href="https://www.zsh.org" rel="noopener noreferrer">Zsh</a></p>

<ul>
<li>Version: 5.9</li>
</ul>


</li>

<li>

<p>cURL</p>

<ul>
<li>Version: <a href="https://github.com/curl/curl/tree/curl-8_11_1" rel="noopener noreferrer">8.11.1</a>
</li>
</ul>


</li>

<li>

<p>Docker Desktop</p>

<ul>
<li>
<a href="https://docs.docker.com/desktop/release-notes" rel="noopener noreferrer">Version</a>: 4.37.2</li>
</ul>


</li>

<li>

<p><a href="https://docs.github.com/ja/rest" rel="noopener noreferrer">GitHub API</a></p>

<ul>
<li>
<a href="https://docs.github.com/en/rest/about-the-rest-api/api-versions" rel="noopener noreferrer">Version</a>: 2022-11-28</li>
</ul>


</li>

</ul>

<h2>
  
  
  Dir Structure
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.
├── docker
│   ├── .env
│   └── docker-compose.yml
├── index.html
├── nginx.conf
└── openapi
    └── openapi.yml
</code></pre>

</div>



<h3>
  
  
  openapi/openapi.yml
</h3>

<p>* Just a sample.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">openapi</span><span class="pi">:</span> <span class="s">3.0.3</span>
<span class="na">info</span><span class="pi">:</span>
  <span class="na">title</span><span class="pi">:</span> <span class="s">Sample API</span>
  <span class="na">version</span><span class="pi">:</span> <span class="s">1.0.0</span>
<span class="na">paths</span><span class="pi">:</span>
  <span class="na">/hello</span><span class="pi">:</span>
    <span class="na">get</span><span class="pi">:</span>
      <span class="na">summary</span><span class="pi">:</span> <span class="s">Hello World Request</span>
      <span class="na">responses</span><span class="pi">:</span>
        <span class="s2">"</span><span class="s">200"</span><span class="err">:</span>
          <span class="na">description</span><span class="pi">:</span> <span class="s">OK</span>
          <span class="na">content</span><span class="pi">:</span>
            <span class="na">text/plain</span><span class="pi">:</span>
              <span class="na">schema</span><span class="pi">:</span>
                <span class="na">type</span><span class="pi">:</span> <span class="s">string</span>
                <span class="na">example</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Hello,</span><span class="nv"> </span><span class="s">World!"</span>
</code></pre>

</div>



<h3>
  
  
  index.html
</h3>

<h4>
  
  
  Swagger UI
</h4>

<p>Most of the content is the same as what is written <a href="https://github.com/swagger-api/swagger-ui/blob/v5.18.2/docs/usage/installation.md" rel="noopener noreferrer">here.</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"description"</span> <span class="na">content=</span><span class="s">"SwaggerUI"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;title&gt;</span>SwaggerUI<span class="nt">&lt;/title&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"https://unpkg.com/swagger-ui-dist@5.11.0/swagger-ui.css"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">id=</span><span class="s">"swagger-ui"</span><span class="nt">&gt;&lt;/div&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://unpkg.com/swagger-ui-dist@5.11.0/swagger-ui-bundle.js"</span> <span class="na">crossorigin</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="nt">&lt;script&gt;</span>
      <span class="nb">window</span><span class="p">.</span><span class="nx">onload</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="nb">window</span><span class="p">.</span><span class="nx">ui</span> <span class="o">=</span> <span class="nc">SwaggerUIBundle</span><span class="p">({</span>
          <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./openapi/openapi.yml</span><span class="dl">'</span><span class="p">,</span>
          <span class="na">dom_id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">#swagger-ui</span><span class="dl">'</span>
        <span class="p">});</span>
      <span class="p">};</span>
    <span class="nt">&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<ul>
<li>
<p>Updated <strong>url</strong> to the relative path of the OAS file.<br>
</p>
<pre class="highlight html"><code><span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;script&gt;</span>
      <span class="nb">window</span><span class="p">.</span><span class="nx">onload</span> <span class="o">=</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="nb">window</span><span class="p">.</span><span class="nx">ui</span> <span class="o">=</span> <span class="nc">SwaggerUIBundle</span><span class="p">({</span>
          <span class="c1">// before</span>
          <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://petstore3.swagger.io/api/v3/openapi.json</span><span class="dl">'</span><span class="p">,</span>
          <span class="c1">// after</span>
          <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./openapi/openapi.yml</span><span class="dl">'</span><span class="p">,</span>
        <span class="p">});</span>
      <span class="p">};</span>
    <span class="nt">&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
<li>
<p><a href="https://registry.npmjs.org/swagger-ui-dist" rel="noopener noreferrer">Using <code>latest</code> for version specification is also acceptable.</a><br>
</p>
<pre class="highlight html"><code><span class="nt">&lt;html</span> <span class="na">lang=</span><span class="s">"en"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="c">&lt;!-- before --&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"https://unpkg.com/swagger-ui-dist@5.11.0/swagger-ui.css"</span> <span class="nt">/&gt;</span>
    <span class="c">&lt;!-- after --&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"stylesheet"</span> <span class="na">href=</span><span class="s">"https://unpkg.com/swagger-ui-dist@latest/swagger-ui.css"</span> <span class="nt">/&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="c">&lt;!-- before --&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://unpkg.com/swagger-ui-dist@5.11.0/swagger-ui-bundle.js"</span> <span class="na">crossorigin</span><span class="nt">&gt;&lt;/script&gt;</span>
    <span class="c">&lt;!-- after --&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://unpkg.com/swagger-ui-dist@latest/swagger-ui-bundle.js"</span> <span class="na">crossorigin</span><span class="nt">&gt;&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
</ul>

<h4>
  
  
  ReDoc
</h4>

<p>Most of the content is the same as what is written <a href="https://redocly.com/docs/redoc/deployment/html" rel="noopener noreferrer">here.</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="cp">&lt;!DOCTYPE html&gt;</span>
<span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;head&gt;</span>
    <span class="nt">&lt;title&gt;</span>Redoc<span class="nt">&lt;/title&gt;</span>
    <span class="c">&lt;!-- needed for adaptive design --&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">charset=</span><span class="s">"utf-8"</span><span class="nt">/&gt;</span>
    <span class="nt">&lt;meta</span> <span class="na">name=</span><span class="s">"viewport"</span> <span class="na">content=</span><span class="s">"width=device-width, initial-scale=1"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;link</span> <span class="na">href=</span><span class="s">"https://fonts.googleapis.com/css?family=Montserrat:300,400,700|Roboto:300,400,700"</span> <span class="na">rel=</span><span class="s">"stylesheet"</span><span class="nt">&gt;</span>

    <span class="c">&lt;!--
    Redoc doesn't change outer page styles
    --&gt;</span>
    <span class="nt">&lt;style&gt;</span>
      <span class="nt">body</span> <span class="p">{</span>
        <span class="nl">margin</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
        <span class="nl">padding</span><span class="p">:</span> <span class="m">0</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="nt">&lt;/style&gt;</span>
  <span class="nt">&lt;/head&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="nt">&lt;redoc</span> <span class="na">spec-url=</span><span class="s">'./openapi/openapi.yml'</span><span class="nt">&gt;&lt;/redoc&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://cdn.redoc.ly/redoc/latest/bundles/redoc.standalone.js"</span><span class="nt">&gt;</span> <span class="nt">&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</div>



<ul>
<li>
<p>Updated <strong>spec-url</strong> to the relative path of the OAS file.<br>
</p>
<pre class="highlight html"><code><span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="c">&lt;!-- before --&gt;</span>
    <span class="nt">&lt;redoc</span> <span class="na">spec-url=</span><span class="s">'http://petstore.swagger.io/v2/swagger.json'</span><span class="nt">&gt;&lt;/redoc&gt;</span>
    <span class="c">&lt;!-- after --&gt;</span>
    <span class="nt">&lt;redoc</span> <span class="na">spec-url=</span><span class="s">'./openapi/openapi.yml'</span><span class="nt">&gt;&lt;/redoc&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
<li>
<p><a href="https://github.com/Redocly/redoc/blob/v2.2.0/README.md#releases" rel="noopener noreferrer">Specifying a specific version is also acceptable.</a><br>
The list of acceptable versions should match what is written <a href="https://registry.npmjs.org/redoc" rel="noopener noreferrer">here.</a><br>
</p>
<pre class="highlight html"><code><span class="nt">&lt;html&gt;</span>
  <span class="nt">&lt;body&gt;</span>
    <span class="c">&lt;!-- before --&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://cdn.redoc.ly/redoc/latest/bundles/redoc.standalone.js"</span><span class="nt">&gt;</span> <span class="nt">&lt;/script&gt;</span>
    <span class="c">&lt;!-- after --&gt;</span>
    <span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"https://cdn.redoc.ly/redoc/v2.0.0/bundles/redoc.standalone.js"</span><span class="nt">&gt;</span> <span class="nt">&lt;/script&gt;</span>
  <span class="nt">&lt;/body&gt;</span>
<span class="nt">&lt;/html&gt;</span>
</code></pre>

</li>
</ul>

<h3>
  
  
  nginx.conf
</h3>

<p>This file is used to serve a preview of the OAS file via Nginx.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight nginx"><code><span class="k">server</span> <span class="p">{</span>
    <span class="kn">listen</span> <span class="mi">80</span><span class="p">;</span>

    <span class="kn">location</span> <span class="n">/</span> <span class="p">{</span>
        <span class="kn">root</span> <span class="n">/usr/share/nginx/html</span><span class="p">;</span>
        <span class="kn">index</span> <span class="s">index.html</span><span class="p">;</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<ul>
<li>
<a href="https://nginx.org/en/docs/beginners_guide.html" rel="noopener noreferrer">The default port used by Nginx is <strong>80</strong></a>, which is the standard port for HTTP.

<ul>
<li>We will continue to use this port even after overriding the config file.</li>
</ul>


</li>

</ul>

<h3>
  
  
  docker
</h3>

<p>The Nginx server runs inside a Docker container.</p>

<h4>
  
  
  ./.env
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>HOST_PORT={host-port}
</code></pre>

</div>



<h4>
  
  
  ./docker-compose.yml
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">nginx</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">nginx:1.28.2</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">${HOST_PORT}:80"</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="c1"># Serve the preview page.</span>
      <span class="pi">-</span> <span class="s">../index.html:/usr/share/nginx/html/index.html:ro</span>
      <span class="c1"># Serve the OAS file.</span>
      <span class="pi">-</span> <span class="s">../openapi:/usr/share/nginx/html/openapi:ro</span>
      <span class="c1"># Override the default Nginx configuration.</span>
      <span class="pi">-</span> <span class="s">../nginx.conf:/etc/nginx/conf.d/default.conf:ro</span>
</code></pre>

</div>



<ul>
<li><p>As <a href="https://docs.docker.com/reference/compose-file/services/#image" rel="noopener noreferrer">image</a>, we specify <a href="https://hub.docker.com/_/nginx" rel="noopener noreferrer">the one provided by the official.</a></p></li>
<li><p><a href="https://docs.docker.com/engine/storage/volumes/#use-a-read-only-volume" rel="noopener noreferrer"><code>ro</code> stands for <strong>read-only</strong>.</a></p></li>
</ul>

<h2>
  
  
  Previewing the OAS file
</h2>

<ol>
<li><p>Open the terminal.</p></li>
<li><p>Move the current-dir to the location where <code>docker-compose.yml</code> is located.</p></li>
<li>
<p><a href="https://docs.docker.com/reference/cli/docker/compose/up/" rel="noopener noreferrer">Create and start the container.</a><br>
</p>
<pre class="highlight shell"><code>docker compose up <span class="nt">-d</span>
</code></pre>


<p>The <code>-d</code> option runs the container in <strong>detached mode</strong>, allowing the terminal to remain available.</p>
</li>
<li><p>Open the preview page: <code>http://localhost:{host-port}</code></p></li>
<li><p>Check until satisfied.</p></li>
<li><p>Close the preview page.</p></li>
<li>
<p><a href="https://docs.docker.com/reference/cli/docker/compose/down/" rel="noopener noreferrer">Stop and remove the container.</a><br>
</p>
<pre class="highlight shell"><code>docker compose down
</code></pre>

</li>
</ol>

<h2>
  
  
  Preparation for publication
</h2>

<p>In order to send some GitHub API requests from a terminal.</p>

<h3>
  
  
  Personal Access Token
</h3>

<p><a href="https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic" rel="noopener noreferrer">Create a Personal Access Token (Classic).</a></p>

<p><code>repo</code> scope is always required.</p>

<h3>
  
  
  Declaring some variables
</h3>

<ul>
<li><p><code>&lt;PERSONAL_ACCESS_TOKEN&gt;</code><br>
The personal access token we created.</p></li>
<li><p><code>&lt;OWNER&gt;</code><br>
The account owner of the repository.</p></li>
<li><p><code>&lt;REPO&gt;</code><br>
The name of the repository.</p></li>
<li><p><code>&lt;BRANCH&gt;</code><br>
The branch name used to publish.<br>
</p></li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">PERSONAL_ACCESS_TOKEN</span><span class="o">=</span><span class="s2">"&lt;PERSONAL_ACCESS_TOKEN&gt;"</span>
<span class="nv">OWNER</span><span class="o">=</span><span class="s2">"&lt;OWNER&gt;"</span>
<span class="nv">REPO</span><span class="o">=</span><span class="s2">"&lt;REPO&gt;"</span>
<span class="nv">BRANCH</span><span class="o">=</span><span class="s2">"&lt;BRANCH&gt;"</span>
</code></pre>

</div>



<h2>
  
  
  Publishing
</h2>

<h3>
  
  
  Changing repository from private to public
</h3>

<p>In a free plan account, we cannot publish GitHub Pages from a private repository. Therefore, in this case, <a href="https://docs.github.com/en/rest/repos/repos?apiVersion=2022-11-28#update-a-repository" rel="noopener noreferrer">we need to update the repository from private to public.</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="nt">-X</span> PATCH <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$PERSONAL_ACCESS_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-GitHub-Api-Version: 2022-11-28"</span> <span class="se">\</span>
  <span class="s2">"https://api.github.com/repos/</span><span class="nv">$OWNER</span><span class="s2">/</span><span class="nv">$REPO</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "private": false
  }'</span>
</code></pre>

</div>



<p>or<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="nt">-X</span> PATCH <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$PERSONAL_ACCESS_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-GitHub-Api-Version: 2022-11-28"</span> <span class="se">\</span>
  <span class="s2">"https://api.github.com/repos/</span><span class="nv">$OWNER</span><span class="s2">/</span><span class="nv">$REPO</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "visibility": "public"
  }'</span>
</code></pre>

</div>



<h3>
  
  
  <a href="https://docs.github.com/en/rest/pages/pages?apiVersion=2022-11-28#create-a-github-pages-site" rel="noopener noreferrer">Activating GitHub Pages</a>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="nt">-X</span> POST <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$PERSONAL_ACCESS_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-GitHub-Api-Version: 2022-11-28"</span> <span class="se">\</span>
  <span class="s2">"https://api.github.com/repos/</span><span class="nv">$OWNER</span><span class="s2">/</span><span class="nv">$REPO</span><span class="s2">/pages"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s2">"{
       </span><span class="se">\"</span><span class="s2">build_type</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">legacy</span><span class="se">\"</span><span class="s2">,
       </span><span class="se">\"</span><span class="s2">source</span><span class="se">\"</span><span class="s2">: {
         </span><span class="se">\"</span><span class="s2">branch</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="nv">$BRANCH</span><span class="se">\"</span><span class="s2">,
         </span><span class="se">\"</span><span class="s2">path</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">/</span><span class="se">\"</span><span class="s2">
       }
     }"</span>
</code></pre>

</div>



<ul>
<li>
<p>After GitHub Pages is enabled, the site based on <code>index.html</code> will be built and deployed. We can check the progress of the build by sending <a href="https://docs.github.com/en/rest/pages/pages?apiVersion=2022-11-28#get-latest-pages-build" rel="noopener noreferrer">the following request.</a><br>
</p>
<pre class="highlight shell"><code>curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="nt">-X</span> GET <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$PERSONAL_ACCESS_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-GitHub-Api-Version: 2022-11-28"</span> <span class="se">\</span>
  <span class="s2">"https://api.github.com/repos/</span><span class="nv">$OWNER</span><span class="s2">/</span><span class="nv">$REPO</span><span class="s2">/pages/builds/latest"</span>
</code></pre>

</li>
<li><p>If GitHub Pages is already enabled, the 409 conflict error will be returned.</p></li>
<li>
<p>Since the <code>BRANCH</code> variable is used to specify the branch, the single quotes in the request body must be changed to double quotes. Assuming <code>BRANCH="main"</code>, the differences are as follows:<br>
</p>
<pre class="highlight shell"><code><span class="nv">BRANCH</span><span class="o">=</span><span class="s2">"main"</span>

curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="c"># soft</span>
  <span class="nt">-d</span> <span class="s2">"{
       </span><span class="se">\"</span><span class="s2">build_type</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">legacy</span><span class="se">\"</span><span class="s2">,
       </span><span class="se">\"</span><span class="s2">source</span><span class="se">\"</span><span class="s2">: {
         </span><span class="se">\"</span><span class="s2">branch</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="nv">$BRANCH</span><span class="se">\"</span><span class="s2">,
         </span><span class="se">\"</span><span class="s2">path</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">/</span><span class="se">\"</span><span class="s2">
       }
     }"</span>
  <span class="c"># hard</span>
  <span class="nt">-d</span> <span class="s1">'{
    "build_type": "legacy",
    "source": {
      "branch": "main",
      "path": "/"
    }
  }'</span>
</code></pre>

</li>
<li><p>The GitHub Pages URL is included in the <code>html_url</code> property of the response.</p></li>
</ul>

<h3>
  
  
  <a href="https://docs.github.com/en/rest/pages/pages?apiVersion=2022-11-28#get-a-github-pages-site" rel="noopener noreferrer">Retrieving GitHub Pages URL</a>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$PERSONAL_ACCESS_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-GitHub-Api-Version: 2022-11-28"</span> <span class="se">\</span>
  <span class="s2">"https://api.github.com/repos/</span><span class="nv">$OWNER</span><span class="s2">/</span><span class="nv">$REPO</span><span class="s2">/pages"</span>
</code></pre>

</div>



<ul>
<li>The URL is included in the <code>html_url</code> property of the response.</li>
</ul>

<h2>
  
  
  <a href="https://docs.github.com/en/rest/pages/pages?apiVersion=2022-11-28#delete-a-github-pages-site" rel="noopener noreferrer">Unpublishing</a>
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-L</span> <span class="se">\</span>
  <span class="nt">-X</span> DELETE <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$PERSONAL_ACCESS_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-GitHub-Api-Version: 2022-11-28"</span> <span class="se">\</span>
  <span class="s2">"https://api.github.com/repos/</span><span class="nv">$OWNER</span><span class="s2">/</span><span class="nv">$REPO</span><span class="s2">/pages"</span>
</code></pre>

</div>



<h2>
  
  
  Republishing
</h2>

<p>We just need to send <a href="https://docs.github.com/en/rest/pages/pages?apiVersion=2022-11-28#create-a-github-pages-site" rel="noopener noreferrer">the same request as when activating the site.</a></p>

<h2>
  
  
  Sample Repositories
</h2>

<ul>
<li>
<a href="https://github.com/kengo-hakomori/publishing-oas-based-api-doc-on-github-pages-swagger-ui" rel="noopener noreferrer">Swagger UI</a>

<ul>
<li><a href="https://kengo-hakomori.github.io/publishing-oas-based-api-doc-on-github-pages-swagger-ui/" rel="noopener noreferrer">GitHub Pages</a></li>
</ul>


</li>

<li>

<a href="https://github.com/kengo-hakomori/publishing-oas-based-api-doc-on-github-pages-redoc" rel="noopener noreferrer">ReDoc</a>

<ul>
<li><a href="https://kengo-hakomori.github.io/publishing-oas-based-api-doc-on-github-pages-redoc/" rel="noopener noreferrer">GitHub Pages</a></li>
</ul>


</li>

</ul>

<h2>
  
  
  Remark
</h2>

<p>This publishing procedure does not depend on the number of OAS files. i.e. even if a OAS file is split into multiple files, we can follow this procedure to publish.</p>

