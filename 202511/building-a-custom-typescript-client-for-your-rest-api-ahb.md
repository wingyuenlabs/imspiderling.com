---
Title: Building a Custom TypeScript Client for Your REST API
Description: 
Author: LiavZi
Date: 2025-11-17T21:58:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>The examples in this post are available in a demo repository here: <a href="https://github.com/liavzi/custom-open-api-ts-client" rel="noopener noreferrer">https://github.com/liavzi/custom-open-api-ts-client</a>.</p>

<h2>
  
  
  The Motivation
</h2>

<p>In one of the projects I'm working on, we use a simple API service to communicate with the server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">class</span> <span class="nc">ApiService</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">baseUrl</span> <span class="o">=</span> <span class="dl">''</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
  <span class="p">}</span>

  <span class="nf">get</span><span class="p">(</span><span class="nx">endpoint</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="p">...</span>
  <span class="p">}</span>

  <span class="nf">post</span><span class="p">(</span><span class="nx">endpoint</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">body</span><span class="p">:</span> <span class="kr">any</span><span class="p">:</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="p">...</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// when I need to use it</span>
<span class="nx">apiService</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">"</span><span class="s2">iHateToCopyThisEveryTime</span><span class="dl">"</span><span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">response</span><span class="p">:</span> <span class="nx">AnotherTypeINeedToManuallyCreateEveryTime</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{});</span>
</code></pre>

</div>



<p>The first problem is that I always have to manually pass the endpoint URL. This usually means copy pasting it from the backend, which is repetitive and easy to mess up.</p>

<p>The second problem is even worse: whenever I need to GET or POST json data, I also need to manually create a matching TypeScript interface that represents the server’s request or response model. This is tedious and error prone. If someone changes a property name or adds a new field on the server and forgets to update the client, things break silently.</p>

<p>So the goal became clear: <strong>automatically generate a TypeScript client</strong>. Whenever an API endpoint is added or changed on the server, the client should get a matching, fully typed function, automatically. No more copy-pasting URLs, no more mismatched interfaces, and no more guessing. Just calling strongly typed functions that feel like any other regular TypeScript function.</p>

<h2>
  
  
  Before Diving In
</h2>

<ul>
<li>I’m using ASP.NET Core on the backend and Angular on the frontend, but the general idea works with almost any tech stack.</li>
<li>Here, I’m using the <a href="https://heyapi.dev/" rel="noopener noreferrer">Hey API</a> package to generate TypeScript models. This package can also generate full Angular, React, and other clients. However, I ran into a challenge: I wanted to integrate it into an existing project that already had a service handling API calls. Merging the two wasn’t straightforward, so I decided to use Hey API solely for model generation.</li>
</ul>

<h2>
  
  
  The Plan
</h2>

<ul>
<li>
Generate an OpenAPI
specification automatically on every server build.</li>
<li>
Generate the request and response models from that spec using Hey API.</li>
<li>
Generate matching client API services,one for each controller on the server, so the frontend always has up-to-date, fully typed functions.</li>
</ul>

<h2>
  
  
  Generate an OpenAPI specification automatically on every server build <a></a>
</h2>

<p>ASP.NET Core supports OpenAPI out of the box, so generating the specification is pretty straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="err">﻿</span><span class="k">using</span> <span class="nn">Backend.OpenApi</span><span class="p">;</span>

<span class="kt">var</span> <span class="n">builder</span> <span class="p">=</span> <span class="n">WebApplication</span><span class="p">.</span><span class="nf">CreateBuilder</span><span class="p">(</span><span class="n">args</span><span class="p">);</span>

<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="nf">AddControllers</span><span class="p">();</span>
<span class="c1">// Learn more about configuring OpenAPI at https://aka.ms/aspnet/openapi</span>
<span class="n">builder</span><span class="p">.</span><span class="n">Services</span><span class="p">.</span><span class="nf">AddOpenApi</span><span class="p">(</span><span class="s">"internal-api"</span><span class="p">,</span> <span class="n">options</span> <span class="p">=&gt;</span>
<span class="p">{</span>
    <span class="n">options</span><span class="p">.</span><span class="nf">AddOperationTransformer</span><span class="p">(</span><span class="k">new</span> <span class="nf">AddMethodNameOperationTransformer</span><span class="p">());</span>
<span class="p">});</span>

<span class="kt">var</span> <span class="n">app</span> <span class="p">=</span> <span class="n">builder</span><span class="p">.</span><span class="nf">Build</span><span class="p">();</span>

<span class="k">if</span> <span class="p">(</span><span class="n">app</span><span class="p">.</span><span class="n">Environment</span><span class="p">.</span><span class="nf">IsDevelopment</span><span class="p">())</span>
<span class="p">{</span>
    <span class="n">app</span><span class="p">.</span><span class="nf">MapOpenApi</span><span class="p">();</span>
<span class="p">}</span>

<span class="n">app</span><span class="p">.</span><span class="nf">UseAuthorization</span><span class="p">();</span>

<span class="n">app</span><span class="p">.</span><span class="nf">MapControllers</span><span class="p">();</span>

<span class="n">app</span><span class="p">.</span><span class="nf">Run</span><span class="p">();</span>

</code></pre>

</div>



<p>Pay special attention to the following line, which adds a custom operation transformer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="n">options</span><span class="p">.</span><span class="nf">AddOperationTransformer</span><span class="p">(</span><span class="k">new</span> <span class="nf">AddMethodNameOperationTransformer</span><span class="p">());</span>
</code></pre>

</div>



<p>and the code of the transformer:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="err">﻿</span><span class="k">using</span> <span class="nn">Microsoft.AspNetCore.Mvc.Controllers</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Microsoft.AspNetCore.OpenApi</span><span class="p">;</span>
<span class="k">using</span> <span class="nn">Microsoft.OpenApi</span><span class="p">;</span>

<span class="k">namespace</span> <span class="nn">Backend.OpenApi</span><span class="p">;</span>

<span class="k">public</span> <span class="k">class</span> <span class="nc">AddMethodNameOperationTransformer</span> <span class="p">:</span> <span class="n">IOpenApiOperationTransformer</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="k">async</span> <span class="n">Task</span> <span class="nf">TransformAsync</span><span class="p">(</span><span class="n">OpenApiOperation</span> <span class="n">operation</span><span class="p">,</span> <span class="n">OpenApiOperationTransformerContext</span> <span class="n">context</span><span class="p">,</span>
        <span class="n">CancellationToken</span> <span class="n">cancellationToken</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">context</span><span class="p">.</span><span class="n">Description</span><span class="p">.</span><span class="n">ActionDescriptor</span> <span class="k">is</span> <span class="k">not</span> <span class="n">ControllerActionDescriptor</span> <span class="n">controllerActionDescriptor</span><span class="p">)</span>
            <span class="k">return</span><span class="p">;</span>
        <span class="n">operation</span><span class="p">.</span><span class="nf">AddExtension</span><span class="p">(</span><span class="s">"x-method-name"</span><span class="p">,</span> <span class="k">new</span> <span class="nf">JsonNodeExtension</span><span class="p">(</span><span class="n">controllerActionDescriptor</span><span class="p">.</span><span class="n">ActionName</span><span class="p">));</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Thanks to the transformer, the generated OpenAPI spec will now include the actual C# method name as part of the operation metadata. This becomes extremely helpful when generating the TypeScript client, because we can produce clean, predictable function names instead of trying to infer them from routes.<br>
For example, consider an API endpoint that returns a list of books:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="p">[</span><span class="nx">HttpGet</span><span class="p">]</span>
<span class="k">public</span> <span class="nx">IEnumerable</span><span class="o">&lt;</span><span class="nx">Book</span><span class="o">&gt;</span> <span class="nc">GetAllBooks</span><span class="p">()</span>
<span class="p">{</span>
    <span class="k">return</span> <span class="nx">Books</span><span class="p">;</span>
<span class="p">}</span>

</code></pre>

</div>



<p>It appears in the generated JSON like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="dl">"</span><span class="s2">/api/Books</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
      <span class="dl">"</span><span class="s2">get</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
        <span class="dl">"</span><span class="s2">tags</span><span class="dl">"</span><span class="p">:</span> <span class="p">[</span>
          <span class="dl">"</span><span class="s2">Books</span><span class="dl">"</span>
        <span class="p">],</span>
        <span class="dl">"</span><span class="s2">responses</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
          <span class="dl">"</span><span class="s2">200</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
            <span class="dl">"</span><span class="s2">description</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">OK</span><span class="dl">"</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">content</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
              <span class="dl">"</span><span class="s2">text/plain</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="dl">"</span><span class="s2">schema</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                  <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">array</span><span class="dl">"</span><span class="p">,</span>
                  <span class="dl">"</span><span class="s2">items</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                    <span class="dl">"</span><span class="s2">$ref</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">#/components/schemas/Book</span><span class="dl">"</span>
                  <span class="p">}</span>
                <span class="p">}</span>
              <span class="p">},</span>
              <span class="dl">"</span><span class="s2">application/json</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="dl">"</span><span class="s2">schema</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                  <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">array</span><span class="dl">"</span><span class="p">,</span>
                  <span class="dl">"</span><span class="s2">items</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                    <span class="dl">"</span><span class="s2">$ref</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">#/components/schemas/Book</span><span class="dl">"</span>
                  <span class="p">}</span>
                <span class="p">}</span>
              <span class="p">},</span>
              <span class="dl">"</span><span class="s2">text/json</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                <span class="dl">"</span><span class="s2">schema</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                  <span class="dl">"</span><span class="s2">type</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">array</span><span class="dl">"</span><span class="p">,</span>
                  <span class="dl">"</span><span class="s2">items</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
                    <span class="dl">"</span><span class="s2">$ref</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">#/components/schemas/Book</span><span class="dl">"</span>
                  <span class="p">}</span>
                <span class="p">}</span>
              <span class="p">}</span>
            <span class="p">}</span>
          <span class="p">}</span>
        <span class="p">},</span>
        <span class="dl">"</span><span class="s2">x-method-name</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">GetAllBooks</span><span class="dl">"</span>
      <span class="p">},</span>

</code></pre>

</div>



<p>Now we want to generate the OpenAPI JSON on every build. <br>
ASP.NET Core makes this straightforward. We can simply follow the official instructions from <a href="https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0&amp;tabs=visual-studio%2Cvisual-studio-code#generate-openapi-documents-at-build-time" rel="noopener noreferrer">Microsoft’s documentation</a>.<br>
After that, we add the following lines to the .csproj file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight xml"><code>    <span class="nt">&lt;PropertyGroup&gt;</span>
        <span class="nt">&lt;OpenApiDocumentsDirectory&gt;</span>$(ProjectDir)../../Frontend/src/app/contracts<span class="nt">&lt;/OpenApiDocumentsDirectory&gt;</span>
    <span class="nt">&lt;/PropertyGroup&gt;</span>

    <span class="nt">&lt;Target</span> <span class="na">Name=</span><span class="s">"CreateTypescriptClient"</span> <span class="na">AfterTargets=</span><span class="s">"Build"</span> <span class="na">Condition=</span><span class="s">"'$(Configuration)' == 'Debug'"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;Exec</span> <span class="na">Command=</span><span class="s">"npm run generate-contracts"</span> <span class="na">WorkingDirectory=</span><span class="s">"$(ProjectDir)../../Frontend"</span> <span class="nt">/&gt;</span>
    <span class="nt">&lt;/Target&gt;</span>
</code></pre>

</div>



<p>This configuration instructs the build process to output the generated OpenAPI JSON into the client’s contracts directory.<br>
Then, whenever the project is built in debug mode, it automatically runs the generate-contracts npm script, ensuring your TypeScript client stays in sync with the API definitions.</p>
<h2>
  
  
  Client Side Model Generation<a></a>
</h2>

<p>Next, we’ll implement the generate-contracts script using the <a href="https://heyapi.dev/" rel="noopener noreferrer">Hey API</a> package.<br>
After installing HeyAPI, add the following scripts to your package.json:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="dl">"</span><span class="s2">scripts</span><span class="dl">"</span><span class="p">:</span> <span class="p">{</span>
    <span class="dl">"</span><span class="s2">ng</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">ng</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">start</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">ng serve</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">...</span>
    <span class="dl">"</span><span class="s2">generate-contracts</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">openapi-ts &amp;&amp; npm run generate-api-services</span><span class="dl">"</span><span class="p">,</span>
    <span class="dl">"</span><span class="s2">generate-api-services</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">node src/app/contracts/createApiServices.mjs</span><span class="dl">"</span>
  <span class="p">},</span>
</code></pre>

</div>



<p>The generate-contracts will now create types directly from the OpenAPI JSON.<br>
Hey API is highly customizable, look at the official documantaion to to see how to configure it using the <a href="https://github.com/liavzi/custom-open-api-ts-client/blob/main/Frontend/openapi-ts.config.ts" rel="noopener noreferrer">openapi-ts.config</a> file. </p>
<h2>
  
  
  Generate the Client's API Services<a></a>
</h2>

<p>Now, here’s the really interesting part. The script <a href="https://github.com/liavzi/custom-open-api-ts-client/blob/main/Frontend/src/app/contracts/createApiServices.mjs" rel="noopener noreferrer">createApiServices.mjs</a><br>
iterates over the OpenAPI JSON and generates custom API services using <a href="https://ts-morph.com/" rel="noopener noreferrer">ts-morph</a>.</p>

<p>Take some time to explore the file and its comments to see exactly how it works under the hood.</p>

<p>For example, check out the BooksApiService, which is one of the services generated by this script:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">ApiService</span><span class="p">,</span> <span class="nx">RequestParam</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../../../api-service</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Book</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@contracts</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span> <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span> <span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">BooksApiService</span> <span class="p">{</span>
    <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="k">readonly</span> <span class="nx">apiService</span><span class="p">:</span> <span class="nx">ApiService</span><span class="p">)</span> <span class="p">{</span>
    <span class="p">}</span>

    <span class="nf">getAllBooks</span><span class="p">(</span><span class="nx">apiServiceRequestParams</span><span class="p">?:</span> <span class="nx">RequestParam</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="nx">Book</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">apiService</span><span class="p">.</span><span class="nf">handleInternalApiCall</span><span class="p">({</span>
            <span class="na">url</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/Books</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">pathParams</span><span class="p">:</span> <span class="p">{},</span>
            <span class="na">queryParams</span><span class="p">:</span> <span class="p">{},</span>
            <span class="na">httpVerb</span><span class="p">:</span> <span class="dl">"</span><span class="s2">get</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">requestBody</span><span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span>
            <span class="nx">apiServiceRequestParams</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="nf">addBook</span><span class="p">(</span><span class="nx">requestBody</span><span class="p">:</span> <span class="nx">Book</span><span class="p">,</span> <span class="nx">apiServiceRequestParams</span><span class="p">?:</span> <span class="nx">RequestParam</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="nx">Book</span><span class="o">&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">apiService</span><span class="p">.</span><span class="nf">handleInternalApiCall</span><span class="p">({</span>
            <span class="na">url</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/Books</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">pathParams</span><span class="p">:</span> <span class="p">{},</span>
            <span class="na">queryParams</span><span class="p">:</span> <span class="p">{},</span>
            <span class="na">httpVerb</span><span class="p">:</span> <span class="dl">"</span><span class="s2">post</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">requestBody</span><span class="p">:</span> <span class="nx">requestBody</span><span class="p">,</span>
            <span class="nx">apiServiceRequestParams</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="nf">getBookByTitle</span><span class="p">(</span><span class="nx">title</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">apiServiceRequestParams</span><span class="p">?:</span> <span class="nx">RequestParam</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kc">undefined</span><span class="o">&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">apiService</span><span class="p">.</span><span class="nf">handleInternalApiCall</span><span class="p">({</span>
            <span class="na">url</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/Books/title/{title}</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">pathParams</span><span class="p">:</span> <span class="p">{</span><span class="nx">title</span><span class="p">},</span>
            <span class="na">queryParams</span><span class="p">:</span> <span class="p">{},</span>
            <span class="na">httpVerb</span><span class="p">:</span> <span class="dl">"</span><span class="s2">get</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">requestBody</span><span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span>
            <span class="nx">apiServiceRequestParams</span>
        <span class="p">});</span>
    <span class="p">}</span>

    <span class="nf">getBooksByAuthor</span><span class="p">(</span><span class="nx">author</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">apiServiceRequestParams</span><span class="p">?:</span> <span class="nx">RequestParam</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="nx">Book</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
        <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">apiService</span><span class="p">.</span><span class="nf">handleInternalApiCall</span><span class="p">({</span>
            <span class="na">url</span><span class="p">:</span> <span class="dl">"</span><span class="s2">/api/Books/author/{author}</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">pathParams</span><span class="p">:</span> <span class="p">{</span><span class="nx">author</span><span class="p">},</span>
            <span class="na">queryParams</span><span class="p">:</span> <span class="p">{},</span>
            <span class="na">httpVerb</span><span class="p">:</span> <span class="dl">"</span><span class="s2">get</span><span class="dl">"</span><span class="p">,</span>
            <span class="na">requestBody</span><span class="p">:</span> <span class="kc">undefined</span><span class="p">,</span>
            <span class="nx">apiServiceRequestParams</span>
        <span class="p">});</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>As you can see, it aligns perfectly with the server-side controller:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight csharp"><code><span class="k">namespace</span> <span class="nn">Backend.Controllers</span>
<span class="p">{</span>
    <span class="k">using</span> <span class="nn">Microsoft.AspNetCore.Mvc</span><span class="p">;</span>
    <span class="k">using</span> <span class="nn">System.Collections.Generic</span><span class="p">;</span>

    <span class="p">[</span><span class="n">ApiController</span><span class="p">]</span>
    <span class="p">[</span><span class="nf">Route</span><span class="p">(</span><span class="s">"api/[controller]"</span><span class="p">)]</span>
    <span class="k">public</span> <span class="k">class</span> <span class="nc">BooksController</span> <span class="p">:</span> <span class="n">ControllerBase</span>
    <span class="p">{</span>
        <span class="k">private</span> <span class="k">static</span> <span class="k">readonly</span> <span class="n">List</span><span class="p">&lt;</span><span class="n">Book</span><span class="p">&gt;</span> <span class="n">Books</span> <span class="p">=</span> <span class="k">new</span><span class="p">()</span>
        <span class="p">{</span>
            <span class="k">new</span> <span class="n">Book</span> <span class="p">{</span> <span class="n">Title</span> <span class="p">=</span> <span class="s">"The Hobbit"</span><span class="p">,</span> <span class="n">Author</span> <span class="p">=</span> <span class="s">"J.R.R. Tolkien"</span> <span class="p">},</span>
            <span class="k">new</span> <span class="n">Book</span> <span class="p">{</span> <span class="n">Title</span> <span class="p">=</span> <span class="s">"1984"</span><span class="p">,</span> <span class="n">Author</span> <span class="p">=</span> <span class="s">"George Orwell"</span> <span class="p">},</span>
            <span class="k">new</span> <span class="n">Book</span> <span class="p">{</span> <span class="n">Title</span> <span class="p">=</span> <span class="s">"To Kill a Mockingbird"</span><span class="p">,</span> <span class="n">Author</span> <span class="p">=</span> <span class="s">"Harper Lee"</span> <span class="p">}</span>
        <span class="p">};</span>

        <span class="p">[</span><span class="n">HttpGet</span><span class="p">]</span>
        <span class="k">public</span> <span class="n">IEnumerable</span><span class="p">&lt;</span><span class="n">Book</span><span class="p">&gt;</span> <span class="nf">GetAllBooks</span><span class="p">()</span>
        <span class="p">{</span>
            <span class="k">return</span> <span class="n">Books</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="p">[</span><span class="n">HttpPost</span><span class="p">]</span>
        <span class="k">public</span> <span class="n">Book</span> <span class="nf">AddBook</span><span class="p">([</span><span class="n">FromBody</span><span class="p">]</span> <span class="n">Book</span> <span class="n">book</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">Books</span><span class="p">.</span><span class="nf">Add</span><span class="p">(</span><span class="n">book</span><span class="p">);</span>
            <span class="k">return</span> <span class="n">book</span><span class="p">;</span>
        <span class="p">}</span>

        <span class="p">[</span><span class="nf">HttpGet</span><span class="p">(</span><span class="s">"title/{title}"</span><span class="p">)]</span>
        <span class="k">public</span> <span class="n">Book</span><span class="p">?</span> <span class="nf">GetBookByTitle</span><span class="p">(</span><span class="kt">string</span> <span class="n">title</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">return</span> <span class="n">Books</span><span class="p">.</span><span class="nf">FirstOrDefault</span><span class="p">(</span><span class="n">b</span> <span class="p">=&gt;</span> <span class="n">b</span><span class="p">.</span><span class="n">Title</span><span class="p">.</span><span class="nf">Equals</span><span class="p">(</span><span class="n">title</span><span class="p">,</span> <span class="n">StringComparison</span><span class="p">.</span><span class="n">OrdinalIgnoreCase</span><span class="p">));</span>
        <span class="p">}</span>

        <span class="p">[</span><span class="nf">HttpGet</span><span class="p">(</span><span class="s">"author/{author}"</span><span class="p">)]</span>
        <span class="k">public</span> <span class="n">IEnumerable</span><span class="p">&lt;</span><span class="n">Book</span><span class="p">&gt;</span> <span class="nf">GetBooksByAuthor</span><span class="p">(</span><span class="kt">string</span> <span class="n">author</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">return</span> <span class="n">Books</span><span class="p">.</span><span class="nf">Where</span><span class="p">(</span><span class="n">b</span> <span class="p">=&gt;</span> <span class="n">b</span><span class="p">.</span><span class="n">Author</span><span class="p">.</span><span class="nf">Equals</span><span class="p">(</span><span class="n">author</span><span class="p">,</span> <span class="n">StringComparison</span><span class="p">.</span><span class="n">OrdinalIgnoreCase</span><span class="p">));</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">class</span> <span class="nc">Book</span>
    <span class="p">{</span>
        <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">Title</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
        <span class="k">public</span> <span class="n">required</span> <span class="kt">string</span> <span class="n">Author</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>The amazing part is that this approach frees us from remembering URLs or building query strings - we can just call regular functions. Check out<br>
<a href="https://github.com/liavzi/custom-open-api-ts-client/blob/main/Frontend/src/app/books/books.component.ts" rel="noopener noreferrer">books.component.ts</a> to see how simple it is to use.</p>

<p>Some might argue that this couples the client to the server’s structure.<br>
But in practice, for <strong>internal</strong> APIs, I don’t really care about URLs and query strings - they’re just implementation details. All I want is a straightforward way to call my server!</p>

<p>The final piece is implementing the <a href="https://github.com/liavzi/custom-open-api-ts-client/blob/main/Frontend/src/app/api-service.ts" rel="noopener noreferrer">ApiService</a>.<br>
The <code>handleInternalApiCall(args: InternalApiCallArgs)</code> method receives all the information needed to call the server, making integration with your existing project pretty straightforward. You can see my intentionally simplified implementation in the link above.</p>

<p>That’s it! With this setup, generating TypeScript clients from your OpenAPI spec is no longer a chore. You don’t have to worry about URLs, query strings, or boilerplate code - just call the API like a normal function.<br>
Give it a try in your own projects and see how much smoother your development can become.</p>

