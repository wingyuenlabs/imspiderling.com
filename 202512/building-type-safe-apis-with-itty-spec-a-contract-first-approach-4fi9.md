---
Title: Building Type-Safe APIs with itty-spec: A Contract-First Approach
Description: 
Author: Robert Pitt
Date: 2025-12-14T21:43:46.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Building Type-Safe APIs with itty-spec: A Contract-First Approach
</h2>

<p><strong><a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer">itty-spec</a></strong> is a powerful library that brings type-safe, contract-first API development to <a href="https://github.com/kwhitley/itty-router" rel="noopener noreferrer">itty-router</a>. By defining your API contracts using standard schema libraries, you get automatic validation, full TypeScript type inference, and seamless OpenAPI documentation generation—all while maintaining compatibility with edge computing environments.</p>

<h3>
  
  
  The Problem: Building APIs Without Contracts
</h3>

<p>Traditional API development often involves:</p>

<ul>
<li>
<strong>Manual validation</strong> scattered across route handlers</li>
<li>
<strong>Type definitions</strong> that drift from actual runtime behavior</li>
<li>
<strong>Inconsistent error handling</strong> for invalid requests</li>
<li>
<strong>Outdated documentation</strong> that requires manual maintenance</li>
<li>
<strong>No runtime guarantee</strong> that handlers match their documented contracts</li>
</ul>

<p>These issues lead to bugs, security vulnerabilities, and poor developer experience. <a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> solves these problems by making your schema definitions the single source of truth for routes, validation, types, and documentation.</p>

<h3>
  
  
  Core Philosophy: Contract-First Development
</h3>

<p><a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> follows a contract-first approach where you define your API's structure and validation rules upfront using standard schema libraries. The contract becomes the foundation that drives:</p>

<ol>
<li>
<strong>Route registration</strong> - Automatically creates routes from contract definitions</li>
<li>
<strong>Request validation</strong> - Validates all incoming data against schemas</li>
<li>
<strong>Type inference</strong> - Provides full TypeScript types for handlers</li>
<li>
<strong>Response validation</strong> - Ensures handlers return valid responses</li>
<li>
<strong>Documentation generation</strong> - Automatically creates OpenAPI specifications</li>
</ol>

<h3>
  
  
  Key Features and Capabilities
</h3>

<h4>
  
  
  1. Type-Safe Handlers
</h4>

<p>One of the most powerful features of <a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> is how it provides fully typed request objects to your handlers. When you define a contract, TypeScript automatically infers the types for:</p>

<ul>
<li>
<strong>Path parameters</strong> - Extracted from route patterns like <code>/users/:id</code>
</li>
<li>
<strong>Query parameters</strong> - Typed and validated query strings</li>
<li>
<strong>Request headers</strong> - Validated header objects</li>
<li>
<strong>Request body</strong> - Typed request payloads
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createContract</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">itty-spec</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zod</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">contract</span> <span class="o">=</span> <span class="nf">createContract</span><span class="p">({</span>
  <span class="na">getUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users/:id</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">query</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
      <span class="na">include</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">enum</span><span class="p">([</span><span class="dl">'</span><span class="s1">posts</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">comments</span><span class="dl">'</span><span class="p">]).</span><span class="nf">optional</span><span class="p">(),</span>
    <span class="p">}),</span>
    <span class="na">headers</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
      <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span>
    <span class="p">}),</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">200</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span> <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="c1">// In your handler, everything is fully typed!</span>
<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">createRouter</span><span class="p">({</span>
  <span class="nx">contract</span><span class="p">,</span>
  <span class="na">handlers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">getUser</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// request.params.id is typed as string</span>
      <span class="c1">// request.query.include is typed as 'posts' | 'comments' | undefined</span>
      <span class="c1">// request.validatedHeaders['x-api-key'] is typed as string</span>

      <span class="kd">const</span> <span class="nx">userId</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">params</span><span class="p">.</span><span class="nx">id</span><span class="p">;</span> <span class="c1">// TypeScript knows this is a string</span>
      <span class="kd">const</span> <span class="nx">include</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">query</span><span class="p">.</span><span class="nx">include</span><span class="p">;</span> <span class="c1">// Type-safe enum access</span>

      <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">userId</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">John</span><span class="dl">'</span> <span class="p">});</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p>The type system ensures you can't accidentally access properties that don't exist and provides autocomplete for all available fields.</p>

<h4>
  
  
  2. Validation-Driven Contracts
</h4>

<p><a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> uses a middleware pipeline that automatically validates all incoming requests before they reach your handlers. The validation happens in a specific order:</p>

<ol>
<li>
<strong>Path parameters</strong> - Extracted from the URL and validated against optional <code>pathParams</code> schema</li>
<li>
<strong>Query parameters</strong> - Parsed from the query string and validated</li>
<li>
<strong>Headers</strong> - Normalized and validated</li>
<li>
<strong>Request body</strong> - Parsed from JSON and validated (for POST/PUT/PATCH requests)</li>
</ol>

<p>If validation fails at any step, the request is rejected with a 400 status code and detailed error information, without ever reaching your handler code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">contract</span> <span class="o">=</span> <span class="nf">createContract</span><span class="p">({</span>
  <span class="na">createUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">request</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">max</span><span class="p">(</span><span class="mi">100</span><span class="p">),</span>
      <span class="na">email</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">email</span><span class="p">(),</span>
      <span class="na">age</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">().</span><span class="nf">int</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">18</span><span class="p">).</span><span class="nf">max</span><span class="p">(</span><span class="mi">120</span><span class="p">),</span>
    <span class="p">}),</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">200</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span> <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
      <span class="mi">400</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span> <span class="na">details</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">any</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="c1">// If the request body doesn't match the schema, validation fails automatically</span>
<span class="c1">// Your handler only receives validated, typed data</span>
</code></pre>

</div>



<p>This means your handlers can focus on business logic, knowing that all input data already conforms to your contract's requirements.</p>

<h4>
  
  
  3. Standard Schema Spec Support
</h4>

<p><a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> leverages the <a href="https://github.com/standard-schema/spec" rel="noopener noreferrer">Standard Schema V1 specification</a> to provide vendor-agnostic schema support. This means you can use any schema library that implements the Standard Schema spec, including:</p>

<ul>
<li>
<strong>Zod</strong> (v4) - Currently fully supported with OpenAPI generation</li>
<li>
<strong>Valibot</strong> - Compatible with validation (OpenAPI support planned)</li>
<li>
<strong>ArkType</strong> - Compatible with validation (OpenAPI support planned)</li>
<li>Any other library implementing Standard Schema V1</li>
</ul>

<p>The Standard Schema spec provides a unified interface (<code>StandardSchemaV1</code>) that includes:</p>

<ul>
<li>
<code>~standard.vendor</code> - Identifies the schema library</li>
<li>
<code>~standard.validate()</code> - Standardized validation method</li>
<li>Type inference capabilities through TypeScript</li>
</ul>

<p>This vendor-agnostic approach means you're not locked into a specific schema library and can migrate between them without changing your contract definitions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Works with Zod</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zod</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">zodSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">});</span>

<span class="c1">// Works with Valibot (once supported)</span>
<span class="k">import</span> <span class="o">*</span> <span class="k">as</span> <span class="nx">v</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">valibot</span><span class="dl">'</span><span class="p">;</span>
<span class="kd">const</span> <span class="nx">valibotSchema</span> <span class="o">=</span> <span class="nx">v</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">name</span><span class="p">:</span> <span class="nx">v</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">});</span>

<span class="c1">// Both can be used in contracts</span>
<span class="kd">const</span> <span class="nx">contract</span> <span class="o">=</span> <span class="nf">createContract</span><span class="p">({</span>
  <span class="na">endpoint</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/test</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">request</span><span class="p">:</span> <span class="nx">zodSchema</span><span class="p">,</span> <span class="c1">// or valibotSchema</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span> <span class="mi">200</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">zodSchema</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  4. Automated Documentation Generation
</h3>

<p>One of the most powerful features of <a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> is its ability to automatically generate OpenAPI 3.1 specifications from your contracts. This eliminates the need to manually maintain API documentation—your schema definitions become the documentation.</p>

<p>The OpenAPI generator:</p>

<ul>
<li>
<strong>Extracts schemas</strong> from all contract operations</li>
<li>
<strong>Converts path formats</strong> from <code>:param</code> to <code>{param}</code> (OpenAPI standard)</li>
<li>
<strong>Maps response schemas</strong> to OpenAPI response objects</li>
<li>
<strong>Includes headers, query params, and request bodies</strong> in the specification</li>
<li>
<strong>Deduplicates schemas</strong> using a registry system to avoid duplication
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createOpenApiSpecification</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">itty-spec/openapi</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">contract</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./contract</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">openApiSpec</span> <span class="o">=</span> <span class="nf">createOpenApiSpecification</span><span class="p">(</span><span class="nx">contract</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">title</span><span class="p">:</span> <span class="dl">'</span><span class="s1">User Management API</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">version</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1.0.0</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">A comprehensive API for managing users</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">servers</span><span class="p">:</span> <span class="p">[</span>
    <span class="p">{</span> <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://api.example.com</span><span class="dl">'</span><span class="p">,</span> <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Production</span><span class="dl">'</span> <span class="p">},</span>
    <span class="p">{</span> <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://staging-api.example.com</span><span class="dl">'</span><span class="p">,</span> <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Staging</span><span class="dl">'</span> <span class="p">},</span>
  <span class="p">],</span>
<span class="p">});</span>

<span class="c1">// The spec can be used for:</span>
<span class="c1">// - Serving interactive documentation (Swagger UI, Stoplight Elements)</span>
<span class="c1">// - Generating client SDKs (openapi-generator, swagger-codegen)</span>
<span class="c1">// - API testing tools (Postman, Insomnia)</span>
<span class="c1">// - CI/CD validation</span>
</code></pre>

</div>



<p>The generated OpenAPI spec is fully compliant with OpenAPI 3.1 and can be used with any tool that supports the specification.</p>

<h3>
  
  
  5. Type-Safe Response Helpers
</h3>

<p><a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> provides typed response helper methods on the request object that ensure your responses match your contract definitions:</p>

<ul>
<li>
<code>request.json(body, status?, headers?)</code> - Creates JSON responses with type validation</li>
<li>
<code>request.html(html, status?, headers?)</code> - Creates HTML responses</li>
<li>
<code>request.error(status, body, headers?)</code> - Creates error responses</li>
<li>
<code>request.noContent(status)</code> - Creates 204 No Content responses</li>
</ul>

<p>These helpers use TypeScript's type system to ensure:</p>

<ul>
<li>Only valid status codes from your contract can be used</li>
<li>Response bodies match the schema for the given status code</li>
<li>Headers match the optional header schema for the response
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="nx">contract</span> <span class="o">=</span> <span class="nf">createContract</span><span class="p">({</span>
  <span class="na">getUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users/:id</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">200</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span> <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
      <span class="mi">404</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">createRouter</span><span class="p">({</span>
  <span class="nx">contract</span><span class="p">,</span>
  <span class="na">handlers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">getUser</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">findUser</span><span class="p">(</span><span class="nx">request</span><span class="p">.</span><span class="nx">params</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">user</span><span class="p">)</span> <span class="p">{</span>
        <span class="c1">// TypeScript ensures this matches the 404 response schema</span>
        <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="mi">404</span><span class="p">,</span> <span class="p">{</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">User not found</span><span class="dl">'</span> <span class="p">});</span>
      <span class="p">}</span>

      <span class="c1">// TypeScript ensures this matches the 200 response schema</span>
      <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span> <span class="na">id</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">name</span> <span class="p">});</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>
</code></pre>

</div>



<p>If you try to return an invalid status code or a body that doesn't match the schema, TypeScript will catch the error at compile time.</p>

<h2>
  
  
  Real-World Example
</h2>

<p>Let's look at a complete example that demonstrates all these features working together:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createContract</span><span class="p">,</span> <span class="nx">createRouter</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">itty-spec</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createOpenApiSpecification</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">itty-spec/openapi</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zod</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Define schemas</span>
<span class="kd">const</span> <span class="nx">UserSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
  <span class="na">id</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">uuid</span><span class="p">(),</span>
  <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">),</span>
  <span class="na">email</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">email</span><span class="p">(),</span>
  <span class="na">createdAt</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">datetime</span><span class="p">(),</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">CreateUserSchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
  <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">),</span>
  <span class="na">email</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">email</span><span class="p">(),</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">ListUsersQuerySchema</span> <span class="o">=</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
  <span class="na">page</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">().</span><span class="nf">int</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="k">default</span><span class="p">(</span><span class="mi">1</span><span class="p">),</span>
  <span class="na">limit</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">().</span><span class="nf">int</span><span class="p">().</span><span class="nf">min</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">max</span><span class="p">(</span><span class="mi">100</span><span class="p">).</span><span class="k">default</span><span class="p">(</span><span class="mi">20</span><span class="p">),</span>
  <span class="na">search</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">optional</span><span class="p">(),</span>
<span class="p">});</span>

<span class="c1">// Define the contract</span>
<span class="kd">const</span> <span class="nx">contract</span> <span class="o">=</span> <span class="nf">createContract</span><span class="p">({</span>
  <span class="na">listUsers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">query</span><span class="p">:</span> <span class="nx">ListUsersQuerySchema</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
      <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span>
    <span class="p">}),</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">200</span><span class="p">:</span> <span class="p">{</span>
        <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
          <span class="na">users</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">array</span><span class="p">(</span><span class="nx">UserSchema</span><span class="p">),</span>
          <span class="na">total</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
          <span class="na">page</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
          <span class="na">limit</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">number</span><span class="p">(),</span>
        <span class="p">}),</span>
      <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
  <span class="na">createUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">request</span><span class="p">:</span> <span class="nx">CreateUserSchema</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
      <span class="dl">'</span><span class="s1">content-type</span><span class="dl">'</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">literal</span><span class="p">(</span><span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">),</span>
    <span class="p">}),</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">201</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">UserSchema</span> <span class="p">},</span>
      <span class="mi">400</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">(),</span> <span class="na">details</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">any</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
  <span class="na">getUser</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/users/:id</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">pathParams</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span>
      <span class="na">id</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="nf">uuid</span><span class="p">(),</span>
    <span class="p">}),</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">200</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">UserSchema</span> <span class="p">},</span>
      <span class="mi">404</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">error</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="c1">// Create the router with handlers</span>
<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">createRouter</span><span class="p">({</span>
  <span class="nx">contract</span><span class="p">,</span>
  <span class="na">handlers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">listUsers</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// All types are inferred from the contract</span>
      <span class="kd">const</span> <span class="p">{</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">limit</span><span class="p">,</span> <span class="nx">search</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">query</span><span class="p">;</span>
      <span class="kd">const</span> <span class="nx">apiKey</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">validatedHeaders</span><span class="p">[</span><span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">];</span>

      <span class="c1">// Business logic here</span>
      <span class="kd">const</span> <span class="nx">users</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">findUsers</span><span class="p">({</span> <span class="nx">page</span><span class="p">,</span> <span class="nx">limit</span><span class="p">,</span> <span class="nx">search</span><span class="p">,</span> <span class="nx">apiKey</span> <span class="p">});</span>

      <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
        <span class="na">users</span><span class="p">:</span> <span class="nx">users</span><span class="p">.</span><span class="nx">items</span><span class="p">,</span>
        <span class="na">total</span><span class="p">:</span> <span class="nx">users</span><span class="p">.</span><span class="nx">total</span><span class="p">,</span>
        <span class="nx">page</span><span class="p">,</span>
        <span class="nx">limit</span><span class="p">,</span>
      <span class="p">});</span>
    <span class="p">},</span>

    <span class="na">createUser</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// request.validatedBody is fully typed as CreateUserSchema</span>
      <span class="kd">const</span> <span class="p">{</span> <span class="nx">name</span><span class="p">,</span> <span class="nx">email</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">validatedBody</span><span class="p">;</span>

      <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">createUser</span><span class="p">({</span> <span class="nx">name</span><span class="p">,</span> <span class="nx">email</span> <span class="p">});</span>

      <span class="c1">// TypeScript ensures the response matches UserSchema</span>
      <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
        <span class="na">id</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
        <span class="na">name</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
        <span class="na">email</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span>
        <span class="na">createdAt</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">createdAt</span><span class="p">.</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="p">},</span> <span class="mi">201</span><span class="p">);</span>
    <span class="p">},</span>

    <span class="na">getUser</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="c1">// request.params.id is typed as string (from uuid schema)</span>
      <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">findUserById</span><span class="p">(</span><span class="nx">request</span><span class="p">.</span><span class="nx">params</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>

      <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">user</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="mi">404</span><span class="p">,</span> <span class="p">{</span> <span class="na">error</span><span class="p">:</span> <span class="dl">'</span><span class="s1">User not found</span><span class="dl">'</span> <span class="p">});</span>
      <span class="p">}</span>

      <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
        <span class="na">id</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
        <span class="na">name</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">,</span>
        <span class="na">email</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span>
        <span class="na">createdAt</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">createdAt</span><span class="p">.</span><span class="nf">toISOString</span><span class="p">(),</span>
      <span class="p">});</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="c1">// Generate OpenAPI documentation</span>
<span class="kd">const</span> <span class="nx">openApiSpec</span> <span class="o">=</span> <span class="nf">createOpenApiSpecification</span><span class="p">(</span><span class="nx">contract</span><span class="p">,</span> <span class="p">{</span>
  <span class="na">title</span><span class="p">:</span> <span class="dl">'</span><span class="s1">User Management API</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">version</span><span class="p">:</span> <span class="dl">'</span><span class="s1">1.0.0</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">description</span><span class="p">:</span> <span class="dl">'</span><span class="s1">API for managing users</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">servers</span><span class="p">:</span> <span class="p">[{</span> <span class="na">url</span><span class="p">:</span> <span class="dl">'</span><span class="s1">https://api.example.com</span><span class="dl">'</span> <span class="p">}],</span>
<span class="p">});</span>

<span class="c1">// Export for use in Cloudflare Workers, Node.js, Bun, etc.</span>
<span class="k">export</span> <span class="k">default</span> <span class="p">{</span>
  <span class="na">fetch</span><span class="p">:</span> <span class="nx">router</span><span class="p">.</span><span class="nx">fetch</span><span class="p">,</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Architecture: How It Works
</h3>

<p>Under the hood, <a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> uses a sophisticated middleware system that:</p>

<ol>
<li>
<strong>Registers routes</strong> - Automatically creates routes from contract definitions</li>
<li>
<strong>Validates requests</strong> - Middleware pipeline validates path params, query, headers, and body</li>
<li>
<strong>Attaches types</strong> - Extends request objects with typed properties</li>
<li>
<strong>Provides helpers</strong> - Adds typed response helper methods</li>
<li>
<strong>Formats responses</strong> - Converts contract response objects to HTTP responses</li>
<li>
<strong>Handles errors</strong> - Catches validation errors and formats them appropriately</li>
</ol>

<p>The middleware execution order ensures that validation happens before your handlers run, and type information flows through the entire request lifecycle.</p>

<h3>
  
  
  Benefits Summary
</h3>

<p>Using <a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> provides several key benefits:</p>

<ol>
<li>
<strong>Type Safety</strong> - Catch errors at compile time, not runtime</li>
<li>
<strong>Reduced Boilerplate</strong> - Define routes, validation, and types in one place</li>
<li>
<strong>Automatic Validation</strong> - No need to manually validate requests</li>
<li>
<strong>Up-to-Date Documentation</strong> - OpenAPI specs generated from your contracts</li>
<li>
<strong>Vendor Flexibility</strong> - Use any Standard Schema V1 compatible library</li>
<li>
<strong>Edge Compatible</strong> - Works everywhere itty-router works (Cloudflare Workers, Node.js, Bun, etc.)</li>
<li>
<strong>Better DX</strong> - Full autocomplete and type checking in your IDE</li>
</ol>

<h3>
  
  
  Getting Started
</h3>

<p>To get started with <code>itty-spec</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>itty-spec@latest zod
<span class="c"># or</span>
pnpm add itty-spec@latest zod
</code></pre>

</div>



<p>Then define your contract and create your router:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">createContract</span><span class="p">,</span> <span class="nx">createRouter</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">itty-spec</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">z</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">zod</span><span class="dl">'</span><span class="p">;</span>

<span class="kd">const</span> <span class="nx">contract</span> <span class="o">=</span> <span class="nf">createContract</span><span class="p">({</span>
  <span class="na">hello</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">path</span><span class="p">:</span> <span class="dl">'</span><span class="s1">/hello</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">query</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">name</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">().</span><span class="k">default</span><span class="p">(</span><span class="dl">'</span><span class="s1">World</span><span class="dl">'</span><span class="p">)</span> <span class="p">}),</span>
    <span class="na">responses</span><span class="p">:</span> <span class="p">{</span>
      <span class="mi">200</span><span class="p">:</span> <span class="p">{</span> <span class="na">body</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">object</span><span class="p">({</span> <span class="na">message</span><span class="p">:</span> <span class="nx">z</span><span class="p">.</span><span class="nf">string</span><span class="p">()</span> <span class="p">})</span> <span class="p">},</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">router</span> <span class="o">=</span> <span class="nf">createRouter</span><span class="p">({</span>
  <span class="nx">contract</span><span class="p">,</span>
  <span class="na">handlers</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">hello</span><span class="p">:</span> <span class="k">async </span><span class="p">(</span><span class="nx">request</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
        <span class="na">message</span><span class="p">:</span> <span class="s2">`Hello, </span><span class="p">${</span><span class="nx">request</span><span class="p">.</span><span class="nx">query</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="s2">!`</span><span class="p">,</span>
      <span class="p">});</span>
    <span class="p">},</span>
  <span class="p">},</span>
<span class="p">});</span>

<span class="k">export</span> <span class="k">default</span> <span class="p">{</span> <span class="na">fetch</span><span class="p">:</span> <span class="nx">router</span><span class="p">.</span><span class="nx">fetch</span> <span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  Conclusion
</h3>

<p><a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer"><code>itty-spec</code></a> brings contract-first development to the itty-router ecosystem, providing a powerful yet simple way to build type-safe APIs. By leveraging standard schemas and automatic validation, it eliminates many common sources of bugs while improving developer experience and maintaining up-to-date documentation.</p>

<p>Whether you're building APIs for Cloudflare Workers, Node.js, or Bun, <code>itty-spec</code> gives you the tools you need to build robust, well-documented, and type-safe APIs with minimal boilerplate.</p>

<p>For more information, visit the <a href="https://github.com/robertpitt/itty-spec" rel="noopener noreferrer">GitHub repository</a>.</p>

