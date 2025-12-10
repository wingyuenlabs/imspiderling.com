---
Title: Apigee Handbook on GCP: Quick Start Guide for starting
Description: 
Author: Bryan Granado
Date: 2025-12-10T21:27:18.000Z
Robots: noindex,nofollow
Template: index
---
<p>Has your DevOps or Platform Engineering team just handed you access to an Apigee organization in Google Cloud (GCP) and said, “You’re good to deploy your APIs”? If you opened the console and wondered where to start, this post is for you.</p>

<p>We’re not going to talk about VPC peering or instance provisioning (DevOps already handled that and it costs money). We’ll focus on what matters to you as a Developer: how to take that infrastructure “shell” and turn it into a secure, optimized, and functional gateway for your microservices.</p>

<h2>
  
  
  1. High-level overview: where you stand
</h2>

<p>Apigee is an API Management platform. Think of it as an “Intelligent Receptionist” standing in front of your servers (the backend).</p>

<p>When DevOps configures the environment, they usually provide you with a default domain. Typically the invocation URL looks like:</p>

<p><code>https://{org-name}-{environment}.apigee.net</code></p>

<p>Or, if they set up a custom domain:<br>
<code>https://api.yourcompany.com</code></p>

<p>Your job is not to configure that domain; your job is to define what happens after the request reaches that URL.</p>


<h2>
  
  
  2. What is an API Proxy?
</h2>

<p>The fundamental building block in Apigee is the <strong>API Proxy</strong>.</p>

<p>Think of it as a façade. The client (a mobile app, web app) never speaks directly to your backend (Cloud Run, GKE, VM). It speaks with the Proxy.</p>

<p>A Proxy lets you:</p>

<ol>
<li>
<strong>Decouple:</strong> Change your backend implementation (Java → Go, VM → Serverless) while keeping the public Proxy URL stable.</li>
<li>
<strong>Secure:</strong> Validate tokens (OAuth, API Keys) before traffic reaches your backend.</li>
<li>
<strong>Transform:</strong> Accept XML and return JSON without touching backend code.</li>
</ol>


<h2>
  
  
  3. Anatomy of a Proxy: Endpoint vs Target
</h2>

<p>This is where many get confused. A Proxy has two worlds:</p>
<h3>
  
  
  A. Proxy Endpoint (the face to the client)
</h3>

<p>This defines how the outside world consumes your API. Here you define:</p>

<ul>
<li>The <strong>Base Path</strong> (e.g., <code>/v1/users</code>).</li>
<li>Security rules (who can enter).</li>
<li>What happens <em>before</em> the request goes to the backend.</li>
</ul>
<h3>
  
  
  B. Target Endpoint (the connection to the backend)
</h3>

<p>This defines where Apigee fetches the data. Here you define:</p>

<ul>
<li>The URL of your backend service.</li>
<li>SSL/TLS settings.</li>
<li>What happens <em>when the backend responds</em>.</li>
</ul>

<blockquote>
<p>Didactic note: The flow is:<br>
Client -&gt; <strong>Proxy Endpoint</strong> -&gt; (logic) -&gt; <strong>Target Endpoint</strong> -&gt; Backend</p>
</blockquote>


<h2>
  
  
  4. Creating your first Proxy: Default vs Optimized
</h2>
<h3>
  
  
  The “Default” way (Console UI)
</h3>

<p>For learning, the GCP Console is great.</p>

<ol>
<li>Go to <strong>Apigee &gt; Proxies &gt; Create New</strong>.</li>
<li>Select <strong>Reverse Proxy</strong>.</li>
<li>Fill in:

<ul>
<li>
<strong>Name:</strong> <code>user-service-v1</code>
</li>
<li>
<strong>Base Path:</strong> <code>/v1/users</code>
</li>
<li>
<strong>Target (Backend):</strong> <code>https://my-backend-on-cloudrun.app</code>
</li>
</ul>
</li>
<li>Deploy.
This generates a basic XML. It works, but for production you want more control.</li>
</ol>
<h3>
  
  
  The “Optimized” way (Development best practices)
</h3>

<p>A better professional approach is to avoid editing XML directly in the web console (it’s error-prone and hard to version). Instead, develop locally (VS Code + Apigee extension) and upload the bundle.<br>
Because this post is for beginners, here’s a link that dives deeper: </p>

<p><a href="https://cloud.google.com/apigee/docs/api-platform/reference/api-proxy-configuration-reference?hl=es-419" rel="noopener noreferrer">docs</a></p>


<h2>
  
  
  5. Environment variables: TargetServers and KVMs
</h2>

<p>This is where you collaborate with DevOps. Environments cost money; you don’t want a separate Proxy per environment.</p>

<p>A well-configured Target Endpoint should NOT hardcode a backend URL like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;HTTPTargetConnection&gt;</span>
    <span class="nt">&lt;URL&gt;</span>https://my-backend-dev.com<span class="nt">&lt;/URL&gt;</span>
<span class="nt">&lt;/HTTPTargetConnection&gt;</span>
</code></pre>

</div>






<p>What is a TargetServer?<br>
A TargetServer is a named variable at environment level.</p>

<p>In DEV, the TargetServer named <code>my-backend-server</code> points to <code>dev.backend.com</code> (Cloud Run, VM, etc).</p>

<p>Your task: Ask DevOps to create the TargetServer for each environment (or create it yourself if you have permissions). Then reference the TargetServer by name in your XML.</p>


<h2>
  
  
  6. Flows and Match Rules: the Proxy brain
</h2>

<p>A Proxy isn't an empty tube; it makes decisions. That’s implemented using Flows.</p>

<p>The execution flow inside an endpoint (Proxy or Target) follows this order:</p>

<p><strong>PreFlow:</strong> Always runs. Put global logic here (e.g., traffic protection, API Key validation).</p>

<p><strong>Conditional Flows:</strong> Run only if a condition (Match) applies.</p>

<p><strong>PostFlow:</strong> Always runs at the end (e.g., transaction logging).</p>

<p>Configuring Conditional Flows with Matches</p>

<p>Suppose your API exposes:</p>

<p><strong>GET /users (List)</strong></p>

<p><strong>POST /users (Create)</strong></p>

<p>You might not want the same rules for both (POST may require stricter validation). Use <code>&lt;Condition&gt;</code>.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;Flow</span> <span class="na">name=</span><span class="s">"/CreateUser post"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;Description&gt;</span>Logic for creating users<span class="nt">&lt;/Description&gt;</span>
  <span class="nt">&lt;Condition&gt;</span>(proxy.pathsuffix MatchesPath "/users") and (request.verb = "POST")<span class="nt">&lt;/Condition&gt;</span>

  <span class="nt">&lt;Request&gt;</span>
    <span class="nt">&lt;Step&gt;</span>
      <span class="nt">&lt;Name&gt;</span>Validate-JSON-Input<span class="nt">&lt;/Name&gt;</span>
    <span class="nt">&lt;/Step&gt;</span>
  <span class="nt">&lt;/Request&gt;</span>
<span class="nt">&lt;/Flow&gt;</span>
</code></pre>

</div>



<p>A more complete example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;ProxyEndpoint</span> <span class="na">name=</span><span class="s">"default"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;PreFlow</span> <span class="na">name=</span><span class="s">"PreFlow"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;Request&gt;</span>
      <span class="nt">&lt;Step&gt;&lt;Name&gt;</span>Verify-API-Key<span class="nt">&lt;/Name&gt;&lt;/Step&gt;</span>
    <span class="nt">&lt;/Request&gt;</span>
  <span class="nt">&lt;/PreFlow&gt;</span>

  <span class="nt">&lt;Flow</span> <span class="na">name=</span><span class="s">"CreateResource"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;Condition&gt;</span>(request.verb = "POST")<span class="nt">&lt;/Condition&gt;</span>
    <span class="nt">&lt;Request&gt;</span>
      <span class="nt">&lt;Step&gt;&lt;Name&gt;</span>Quota-Check<span class="nt">&lt;/Name&gt;&lt;/Step&gt;</span>
    <span class="nt">&lt;/Request&gt;</span>
  <span class="nt">&lt;/Flow&gt;</span>

  <span class="nt">&lt;PostFlow</span> <span class="na">name=</span><span class="s">"PostFlow"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;Response&gt;</span>
      <span class="nt">&lt;Step&gt;&lt;Name&gt;</span>Remove-Internal-Headers<span class="nt">&lt;/Name&gt;&lt;/Step&gt;</span>
    <span class="nt">&lt;/Response&gt;</span>
  <span class="nt">&lt;/PostFlow&gt;</span>
<span class="nt">&lt;/ProxyEndpoint&gt;</span>
</code></pre>

</div>






<h2>
  
  
  Context and flow variables
</h2>

<p>Apigee is “context aware”: everything passing through the proxy generates variables you can read in conditions:</p>

<ul>
<li>
<code>request.header.Content-Type</code> — to know the incoming payload type.</li>
<li>
<code>response.status.code</code> — to detect backend failures.</li>
<li>
<code>developer.app.name</code> — to identify the calling app.</li>
</ul>

<p>This helps you implement defensive logic. If the backend returns 500, you don’t want to expose the backend stack trace; you can return a suitable response instead.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="nt">&lt;Condition&gt;</span>
  (response.status.code = 500)
<span class="nt">&lt;/Condition&gt;</span>
</code></pre>

</div>






<h2>
  
  
  Policies
</h2>

<p>Policies are configurations that control parts of the flow. Apigee provides many built-in policies, and you configure them via XML and attach them to steps in flows.</p>

<h1>
  
  
  Common policies:
</h1>

<ul>
<li>Verify API Key:
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="cp">&lt;?xml version="1.0" encoding="UTF-8" standalone="yes"?&gt;</span>
<span class="nt">&lt;VerifyAPIKey</span> <span class="na">async=</span><span class="s">"false"</span> <span class="na">continueOnError=</span><span class="s">"false"</span> <span class="na">enabled=</span><span class="s">"true"</span> <span class="na">name=</span><span class="s">"Verify-API-Key-Policy"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;APIKey</span> <span class="na">ref=</span><span class="s">"request.header.x-api-key"</span><span class="nt">/&gt;</span>
<span class="nt">&lt;/VerifyAPIKey&gt;</span>
</code></pre>

</div>



<ul>
<li>OAuthV2 (Verify Access Token):
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="cp">&lt;?xml version="1.0" encoding="UTF-8" standalone="yes"?&gt;</span>
<span class="nt">&lt;OAuthV2</span> <span class="na">async=</span><span class="s">"false"</span> <span class="na">continueOnError=</span><span class="s">"false"</span> <span class="na">enabled=</span><span class="s">"true"</span> <span class="na">name=</span><span class="s">"Verify-OAuth-Token-Policy"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;Operation&gt;</span>VerifyAccessToken<span class="nt">&lt;/Operation&gt;</span>
  <span class="nt">&lt;AccessToken</span> <span class="na">prefix=</span><span class="s">"Bearer"</span><span class="nt">&gt;</span>request.header.Authorization<span class="nt">&lt;/AccessToken&gt;</span>
<span class="nt">&lt;/OAuthV2&gt;</span>
</code></pre>

</div>



<ul>
<li>Access Control (IP-based filtering):
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code><span class="cp">&lt;?xml version="1.0" encoding="UTF-8" standalone="yes"?&gt;</span>
<span class="nt">&lt;AccessControl</span> <span class="na">async=</span><span class="s">"false"</span> <span class="na">continueOnError=</span><span class="s">"false"</span> <span class="na">enabled=</span><span class="s">"true"</span> <span class="na">name=</span><span class="s">"Access-Control-Policy"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;IPRules</span> <span class="na">noRuleMatchAction=</span><span class="s">"DENY"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;Rule</span> <span class="na">action=</span><span class="s">"DENY"</span> <span class="na">name=</span><span class="s">"Block-Specific-IP"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;SourceAddress</span> <span class="na">mask=</span><span class="s">"32"</span><span class="nt">&gt;</span>192.168.1.100<span class="nt">&lt;/SourceAddress&gt;</span>
    <span class="nt">&lt;/Rule&gt;</span>

    <span class="nt">&lt;Rule</span> <span class="na">action=</span><span class="s">"ALLOW"</span> <span class="na">name=</span><span class="s">"Allow-Internal-Network"</span><span class="nt">&gt;</span>
      <span class="nt">&lt;SourceAddress</span> <span class="na">mask=</span><span class="s">"24"</span><span class="nt">&gt;</span>10.0.0.0<span class="nt">&lt;/SourceAddress&gt;</span>
    <span class="nt">&lt;/Rule&gt;</span>
  <span class="nt">&lt;/IPRules&gt;</span>

  <span class="nt">&lt;IPAddresses</span> <span class="na">ref=</span><span class="s">"client.ip"</span><span class="nt">/&gt;</span>
<span class="nt">&lt;/AccessControl&gt;</span>
</code></pre>

</div>






<p>if you need more help with this, here found:</p>

<ul>
<li>Verify API Key:<a href="https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/verify-api-key-policy?hl=es-419" rel="noopener noreferrer">verify-key</a>
</li>
<li>OAuthV2: <a href="https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/oauthv2-policy?hl=es-419" rel="noopener noreferrer">authV2</a>
</li>
<li>Access Control: <a href="https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/access-control-policy?hl=es-419" rel="noopener noreferrer">access-control</a>
</li>
</ul>




<p>Great — if you got this far you now have a rapid-start checklist and the basic concepts to manage APIs in your Apigee environment.</p>

<p>I hope this helps.</p>

<p>I’m on Threads, LinkedIn and Instagram as brngranado. See you there for more content.</p>

