---
Title: Calling a Public API Gateway Custom Domain from a Private VPC (Without Reconfiguring Your Clients)
Description: 
Author: Dan Guisinger
Date: 2026-01-28T20:43:54.000Z
Robots: noindex,nofollow
Template: index
---
<p><em><a href="https://medium.com/@dguisinger/calling-a-public-api-gateway-custom-domain-from-a-private-vpc-b395f861ffa3" rel="noopener noreferrer">Originally published on Medium</a></em></p>

<p>There’s a deceptively simple requirement that AWS API Gateway still doesn’t support cleanly:</p>

<blockquote>
<p>Use a public, regional API Gateway with a custom domain, and allow private VPC workloads to call that same domain without changing client configuration.</p>
</blockquote>

<p>If you’ve tried this, you’ve probably been sent in circles—by documentation, AI tools, and even AWS recommendations—toward solutions that either don’t deploy or don’t work.</p>

<p>This post documents what actually breaks, why most suggested approaches fail, and the pragmatic workaround that finally solved it.</p>




<h2>
  
  
  The setup
</h2>

<p>You have a public API Gateway endpoint exposed as:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://api.yourdomain.com
</code></pre>

</div>



<p>That domain is used everywhere:</p>

<ul>
<li>Browsers</li>
<li>External consumers</li>
<li>Shared SDKs</li>
</ul>

<p>You also have private workloads (Lambda, ECS, EC2) running in isolated subnets with no NAT gateway. Those workloads need to call the same hostname, not an execute-api URL, and not an environment-specific variant.</p>

<p>Changing the base URL internally would fracture client configuration and defeat the point of having a stable public domain.</p>




<h2>
  
  
  Why the obvious solutions fail
</h2>

<p>The first approach most people try is an interface VPC endpoint for execute-api, combined with a Route 53 private hosted zone that resolves api.yourdomain.com to the endpoint inside the VPC.</p>

<p>DNS-wise, this works. Traffic routes privately. The request never touches the internet.</p>

<p>But the TLS handshake fails.</p>

<p>At that point, nothing reaches API Gateway:</p>

<ul>
<li>No API is identified</li>
<li>No routing occurs</li>
<li>No authentication runs</li>
</ul>

<p>TLS negotiation fails before HTTP exists.</p>




<h2>
  
  
  “Just use private custom domains”
</h2>

<p>This recommendation comes up constantly—and it’s simply incorrect for this scenario.</p>

<p>API Gateway private custom domains only work with private API endpoints. Domain associations explicitly require the API itself to be private. You cannot attach a private custom domain to a public regional API.</p>

<p>CloudFormation rejects it.<br>
The console won’t allow it.<br>
There is no workaround.</p>

<p>This is a product limitation, not a configuration error.</p>


<h2>
  
  
  “Use the execute-api endpoint internally”
</h2>

<p>That works technically, but it forces:</p>

<ul>
<li>Different base URLs per environment</li>
<li>API IDs and stage names in clients</li>
<li>Special-case logic in shared SDKs</li>
</ul>

<p>For systems designed around a single public domain, this is architectural debt.</p>


<h2>
  
  
  The real problem
</h2>

<p>Once you strip away the noise, the issue is very small and very specific.</p>

<p>When you access API Gateway through a VPC endpoint, the endpoint presents a TLS certificate for:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>*.execute-api.{region}.amazonaws.com
</code></pre>

</div>



<p>Even if you connect using:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://api.yourdomain.com
</code></pre>

</div>



<p>The certificate chain is valid.<br>
The issuer is trusted.<br>
The only failure is <strong>hostname mismatch</strong>.</p>

<p>And because TLS fails, API Gateway never sees the request at all.</p>


<h2>
  
  
  The pragmatic workaround
</h2>

<p>If you accept that:</p>

<ul>
<li>You are connecting to an AWS-owned service</li>
<li>Over a VPC endpoint</li>
<li>Using AWS-issued certificates</li>
<li>With DNS fully under your control</li>
</ul>

<p>…then the fix is to <strong>explicitly accept execute-api certificates</strong> when the only TLS error is a name mismatch.</p>

<p>This preserves:</p>

<ul>
<li>One public domain</li>
<li>One client configuration</li>
<li>Private routing without NAT</li>
<li>Full encryption in transit</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwnli4un2ue6zb09bypb6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwnli4un2ue6zb09bypb6.png" alt=" " width="800" height="533"></a></p>


<h2>
  
  
  Applying this in .NET
</h2>
<h2>
  
  
  Configuring the HTTP client
</h2>

<p>This is the missing piece you called out earlier.<br>
The TLS behavior must be applied when registering the HttpClient.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">services</span><span class="p">.</span><span class="nf">AddHttpClient</span><span class="p">(</span><span class="s">"ApiClient"</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">ConfigureForVpcEndpoint</span><span class="p">();</span>
</code></pre>

</div>



<p>Any SDK or service using this client will now tolerate API Gateway’s certificate when accessed via a VPC endpoint.</p>




<h3>
  
  
  The HTTP client handler
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">public</span> <span class="k">static</span> <span class="n">IHttpClientBuilder</span> <span class="nf">ConfigureForVpcEndpoint</span><span class="p">(</span>
    <span class="k">this</span> <span class="n">IHttpClientBuilder</span> <span class="n">builder</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">return</span> <span class="n">builder</span><span class="p">.</span><span class="nf">ConfigurePrimaryHttpMessageHandler</span><span class="p">(()</span> <span class="p">=&gt;</span>
        <span class="k">new</span> <span class="n">SocketsHttpHandler</span>
        <span class="p">{</span>
            <span class="n">SslOptions</span> <span class="p">=</span> <span class="k">new</span> <span class="n">SslClientAuthenticationOptions</span>
            <span class="p">{</span>
                <span class="n">RemoteCertificateValidationCallback</span> <span class="p">=</span>
                    <span class="n">ValidateApiGatewayCertificate</span>
            <span class="p">}</span>
        <span class="p">});</span>
<span class="p">}</span><span class="err">```</span>
<span class="p">{%</span> <span class="n">endraw</span> <span class="p">%}</span>


<span class="err">##</span> <span class="n">Certificate</span> <span class="n">validation</span>
<span class="p">{%</span> <span class="n">raw</span> <span class="p">%}</span>


<span class="err">```</span><span class="n">csharp</span>
<span class="k">private</span> <span class="k">static</span> <span class="kt">bool</span> <span class="nf">ValidateApiGatewayCertificate</span><span class="p">(</span>
    <span class="kt">object</span> <span class="n">sender</span><span class="p">,</span>
    <span class="n">X509Certificate</span><span class="p">?</span> <span class="n">certificate</span><span class="p">,</span>
    <span class="n">X509Chain</span><span class="p">?</span> <span class="n">chain</span><span class="p">,</span>
    <span class="n">SslPolicyErrors</span> <span class="n">sslPolicyErrors</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">if</span> <span class="p">(</span><span class="n">sslPolicyErrors</span> <span class="p">==</span> <span class="n">SslPolicyErrors</span><span class="p">.</span><span class="n">None</span><span class="p">)</span>
        <span class="k">return</span> <span class="k">true</span><span class="p">;</span>

    <span class="k">if</span> <span class="p">(</span><span class="n">sslPolicyErrors</span> <span class="p">==</span> <span class="n">SslPolicyErrors</span><span class="p">.</span><span class="n">RemoteCertificateNameMismatch</span> <span class="p">&amp;&amp;</span>
        <span class="n">certificate</span> <span class="p">!=</span> <span class="k">null</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="kt">var</span> <span class="n">subject</span> <span class="p">=</span> <span class="n">certificate</span><span class="p">.</span><span class="n">Subject</span><span class="p">;</span>

        <span class="k">if</span> <span class="p">(</span><span class="n">subject</span><span class="p">.</span><span class="nf">Contains</span><span class="p">(</span><span class="s">"execute-api"</span><span class="p">,</span> <span class="n">StringComparison</span><span class="p">.</span><span class="n">OrdinalIgnoreCase</span><span class="p">)</span> <span class="p">&amp;&amp;</span>
            <span class="n">subject</span><span class="p">.</span><span class="nf">Contains</span><span class="p">(</span><span class="s">"amazonaws.com"</span><span class="p">,</span> <span class="n">StringComparison</span><span class="p">.</span><span class="n">OrdinalIgnoreCase</span><span class="p">))</span>
        <span class="p">{</span>
            <span class="k">return</span> <span class="k">true</span><span class="p">;</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="k">false</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This does not disable TLS validation. It narrowly allows a known, trusted AWS certificate that fails hostname validation only because API Gateway cannot present your custom-domain certificate over a VPC endpoint.</p>

<h2>
  
  
  Why this isn’t as scary as it looks
</h2>

<p>This is safe because <em>it’s constrained</em>:</p>

<ul>
<li>The trust chain is still validated</li>
<li>Only AWS API Gateway certificates are accepted</li>
<li>DNS is private and controlled</li>
</ul>

<p>Traffic never leaves AWS’s network</p>

<p>This is not a blanket “ignore SSL errors” hack—it’s compensating for a missing feature.</p>

<h2>
  
  
  The real takeaway
</h2>

<p>There is currently <strong>no supported way</strong> to combine:</p>

<ul>
<li>A <strong>public regional API Gateway</strong>
</li>
<li>A <strong>custom domain</strong>
</li>
<li><strong>Private VPC access</strong></li>
<li>A <strong>single, stable hostname</strong>
</li>
<li>Correct TLS without client changes</li>
</ul>

<p>Until AWS fills that gap, this workaround is the cleanest solution I’ve found—and the only one that doesn’t require re-architecting clients around internal-only URLs.</p>

<p>Sometimes the hardest problems aren’t misconfigurations.<br>
They’re the seams where cloud abstractions stop.</p>




<p><strong><em>About the author</em></strong>: <a href="https://danguisinger.com" rel="noopener noreferrer">Dan Guisinger</a> is a consultant specializing in system and security architecture on AWS.</p>

