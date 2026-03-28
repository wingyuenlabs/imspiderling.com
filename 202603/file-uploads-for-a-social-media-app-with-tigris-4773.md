---
Title: File uploads for a social media app with Tigris
Description: 
Author: Twilight
Date: 2026-03-28T21:48:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>Social media apps live and die on media handling. Users post photos, upload videos, share documents. Your backend could route every file through your servers, but that burns bandwidth and adds latency. Tigris gives you a way to push files straight from the browser to object storage, with the data replicated across multiple regions worldwide.</p>

<p>Tigris is an S3-compatible object storage service. Your data lives in multiple regions at once. A user in Tokyo uploads a photo; a follower in Berlin loads it from a nearby cache. You do not pick a primary region. Tigris handles distribution.</p>

<p>This post walks through using the Tigris JavaScript SDK to handle file uploads for a social media app.</p>

<h2>
  
  
  Setup
</h2>

<p>Install the SDK:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> @tigrisdata/storage
</code></pre>

</div>



<p>Create a bucket at <a href="https://console.storage.dev" rel="noopener noreferrer">console.storage.dev</a> and grab an access key. Put the credentials in your environment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">TIGRIS_STORAGE_ACCESS_KEY_ID</span><span class="o">=</span>tid_your_access_key
<span class="nv">TIGRIS_STORAGE_SECRET_ACCESS_KEY</span><span class="o">=</span>tsec_your_secret_key
<span class="nv">TIGRIS_STORAGE_BUCKET</span><span class="o">=</span>social-media-uploads
</code></pre>

</div>



<h2>
  
  
  Server-side uploads
</h2>

<p>Some uploads should go through your server. Profile pictures, for instance, where you want to validate image dimensions before storing. The <code>put</code> function handles this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">put</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tigrisdata/storage</span><span class="dl">"</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">uploadAvatar</span><span class="p">(</span><span class="nx">userId</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">imageBuffer</span><span class="p">:</span> <span class="nx">Buffer</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">put</span><span class="p">(</span><span class="s2">`avatars/</span><span class="p">${</span><span class="nx">userId</span><span class="p">}</span><span class="s2">.jpg`</span><span class="p">,</span> <span class="nx">imageBuffer</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="dl">"</span><span class="s2">image/jpeg</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">access</span><span class="p">:</span> <span class="dl">"</span><span class="s2">public</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">allowOverwrite</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">?.</span><span class="nx">url</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The returned <code>url</code> points to the uploaded file. Because <code>access</code> is <code>"public"</code>, the URL works without authentication. Any user on the platform can load the avatar.</p>

<p>The SDK infers content type from the file extension when you omit <code>contentType</code>. Setting it explicitly is safer when the extension might not match the actual format.</p>

<h2>
  
  
  Large files with multipart upload
</h2>

<p>Videos are the bulk of social media storage. A 500 MB video clip does not fit in a single request. Pass <code>multipart: true</code> and the SDK splits the file into chunks and uploads them in parallel.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">put</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tigrisdata/storage</span><span class="dl">"</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">uploadVideo</span><span class="p">(</span><span class="nx">file</span><span class="p">:</span> <span class="nx">ReadableStream</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">put</span><span class="p">(</span><span class="dl">"</span><span class="s2">videos/new-post.mp4</span><span class="dl">"</span><span class="p">,</span> <span class="nx">file</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">multipart</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="dl">"</span><span class="s2">video/mp4</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">access</span><span class="p">:</span> <span class="dl">"</span><span class="s2">private</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">onUploadProgress</span><span class="p">:</span> <span class="p">({</span> <span class="nx">loaded</span><span class="p">,</span> <span class="nx">total</span><span class="p">,</span> <span class="nx">percentage</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">percentage</span><span class="p">}</span><span class="s2">% uploaded`</span><span class="p">);</span>
    <span class="p">},</span>
  <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">return</span> <span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>onUploadProgress</code> callback fires as chunks complete. Wire it to a progress bar in the UI and users can see how far along the upload is.</p>

<p>Private files need a signed URL for access. Generate one with <code>getPresignedUrl</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">getPresignedUrl</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tigrisdata/storage</span><span class="dl">"</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">url</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">getPresignedUrl</span><span class="p">(</span><span class="dl">"</span><span class="s2">videos/new-post.mp4</span><span class="dl">"</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">operation</span><span class="p">:</span> <span class="dl">"</span><span class="s2">get</span><span class="dl">"</span><span class="p">,</span>
  <span class="na">expiresIn</span><span class="p">:</span> <span class="mi">3600</span><span class="p">,</span>
<span class="p">});</span>
</code></pre>

</div>



<p>That URL works for one hour. After that, generate a new one.</p>

<h2>
  
  
  Client-side uploads
</h2>

<p>Routing files through your server wastes bandwidth. The Tigris SDK ships a browser-compatible module that uploads files directly from the client, skipping your server entirely.</p>

<p>Your server generates a presigned URL, the browser uploads to Tigris using that URL, and your server never touches the file bytes. The <code>@tigrisdata/storage/client</code> package wraps this into a single <code>upload</code> function.</p>

<p>First, add an API route that the client module calls to initiate the upload:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// pages/api/upload.ts (Next.js example)</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">getPresignedUrl</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tigrisdata/storage</span><span class="dl">"</span><span class="p">;</span>

<span class="k">export</span> <span class="k">default</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">handler</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">url</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">getPresignedUrl</span><span class="p">(</span><span class="s2">`uploads/</span><span class="p">${</span><span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">filename</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">operation</span><span class="p">:</span> <span class="dl">"</span><span class="s2">put</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">contentType</span><span class="p">,</span>
    <span class="na">expiresIn</span><span class="p">:</span> <span class="mi">3600</span><span class="p">,</span>
  <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">url</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">500</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">url</span><span class="p">.</span><span class="nx">error</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">200</span><span class="p">).</span><span class="nf">json</span><span class="p">(</span><span class="nx">url</span><span class="p">.</span><span class="nx">data</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Then in the browser:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;input</span> <span class="na">type=</span><span class="s">"file"</span> <span class="na">onchange=</span><span class="s">"handleUpload(event)"</span> <span class="nt">/&gt;</span>

<span class="nt">&lt;script </span><span class="na">type=</span><span class="s">"module"</span><span class="nt">&gt;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">upload</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tigrisdata/storage/client</span><span class="dl">"</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">handleUpload</span><span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">file</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nx">files</span><span class="p">[</span><span class="mi">0</span><span class="p">];</span>

  <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">upload</span><span class="p">(</span><span class="s2">`posts/</span><span class="p">${</span><span class="nx">file</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="nx">file</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">url</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/upload</span><span class="dl">"</span><span class="p">,</span>
    <span class="na">multipart</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">contentType</span><span class="p">:</span> <span class="nx">file</span><span class="p">.</span><span class="nx">type</span><span class="p">,</span>
    <span class="na">onUploadProgress</span><span class="p">:</span> <span class="p">({</span> <span class="nx">percentage</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nf">updateProgressBar</span><span class="p">(</span><span class="nx">percentage</span><span class="p">);</span>
    <span class="p">},</span>
  <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">);</span>
    <span class="k">return</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Uploaded:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">.</span><span class="nx">url</span><span class="p">);</span>
<span class="p">}</span>
<span class="nt">&lt;/script&gt;</span>
</code></pre>

</div>



<p>The <code>url</code> option tells the client module where your server endpoint is. The client calls that endpoint to get a presigned URL, then uploads the file bytes straight to Tigris. Your server CPU and bandwidth stay free.</p>

<h2>
  
  
  Listing and deleting files
</h2>

<p>A user deletes a post. You need to remove the associated media. The SDK provides <code>list</code> and <code>remove</code> for this.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">list</span><span class="p">,</span> <span class="nx">remove</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">@tigrisdata/storage</span><span class="dl">"</span><span class="p">;</span>

<span class="k">async</span> <span class="kd">function</span> <span class="nf">deletePostMedia</span><span class="p">(</span><span class="nx">postId</span><span class="p">:</span> <span class="kr">string</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">files</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">list</span><span class="p">({</span> <span class="na">prefix</span><span class="p">:</span> <span class="s2">`posts/</span><span class="p">${</span><span class="nx">postId</span><span class="p">}</span><span class="s2">/`</span> <span class="p">});</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">files</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">throw</span> <span class="nx">files</span><span class="p">.</span><span class="nx">error</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="k">for </span><span class="p">(</span><span class="kd">const</span> <span class="nx">item</span> <span class="k">of</span> <span class="nx">files</span><span class="p">.</span><span class="nx">data</span><span class="p">?.</span><span class="nx">items</span> <span class="o">??</span> <span class="p">[])</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nf">remove</span><span class="p">(</span><span class="nx">item</span><span class="p">.</span><span class="nx">key</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><code>list</code> with a <code>prefix</code> filter returns only objects under that path. Paginate with <code>paginationToken</code> when a prefix contains more than 100 objects.</p>

<h2>
  
  
  Error handling
</h2>

<p>Every SDK function returns <code>{ data, error }</code>. Check <code>error</code> before using <code>data</code>. This pattern works well in both server and client code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">put</span><span class="p">(</span><span class="dl">"</span><span class="s2">photo.jpg</span><span class="dl">"</span><span class="p">,</span> <span class="nx">buffer</span><span class="p">);</span>

<span class="k">if </span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">Upload failed:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">result</span><span class="p">.</span><span class="nx">error</span><span class="p">);</span>
  <span class="k">return</span><span class="p">;</span>
<span class="p">}</span>

<span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="dl">"</span><span class="s2">Stored at:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">result</span><span class="p">.</span><span class="nx">data</span><span class="p">?.</span><span class="nx">url</span><span class="p">);</span>
</code></pre>

</div>



<p>No exceptions to catch. No guessing about what went wrong. The error object contains enough information to surface a message to the user or log for debugging.</p>

<h2>
  
  
  Why Tigris
</h2>

<p>Most object storage providers charge for egress. Social media apps serve the same files over and over; followers reload images, videos get replayed. Egress fees add up fast. Tigris charges zero for egress.</p>

<p>The global distribution is the other reason. You deploy one bucket. Tigris replicates the data across regions. Users in different continents load files from nearby infrastructure without any extra configuration from you.</p>

<p>The SDK is simpler than configuring the AWS S3 SDK. Three environment variables and you are done. No endpoint URLs to manage, no region parameters to pass on every call.</p>

<p>The Tigris JavaScript SDK handles the common file operations a social media app needs: upload, download, list, delete, and presigned URLs. Client-side uploads keep your server costs down. Global distribution keeps load times low for users everywhere.</p>

