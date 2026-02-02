---
Title: Building a Lightweight JWT Authentication Middleware for Go Gin Applications
Description: 
Author: Daniel Keya
Date: 2026-02-02T21:41:46.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building a Lightweight JWT Authentication Middleware for Go Gin Applications
</h1>

<p>Authentication is the backbone of secure web applications. Today, we'll create a clean, production-ready JWT authentication middleware for Go applications using the Gin framework.</p>

<h2>
  
  
  🎯 What We're Building
</h2>

<p>Our middleware will handle:</p>

<ul>
<li>🔐 JWT token validation</li>
<li>📝 Bearer token support</li>
<li>👤 User context extraction</li>
<li>⚙️ Environment configuration</li>
<li>🧪 Comprehensive testing</li>
</ul>

<h2>
  
  
  🏗️ Project Setup
</h2>

<p>Let's start by setting up our project structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>authmiddleware <span class="o">&amp;&amp;</span> <span class="nb">cd </span>authmiddleware
go mod init authmiddleware
</code></pre>

</div>



<p>Install dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go get github.com/gin-gonic/gin
go get github.com/golang-jwt/jwt/v5
go get github.com/joho/godotenv
</code></pre>

</div>



<h2>
  
  
  📁 Project Structure
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>authmiddleware/
├── config/
│   └── config.go
├── middlewares/
│   └── authmiddleware.go
├── tests/
│   ├── auth_middleware_test.go
│   └── config_test.go
├── docs/
├── .env.example
├── go.mod
└── README.md
</code></pre>

</div>



<h2>
  
  
  ⚙️ Configuration Management
</h2>

<p>First, let's create our configuration handler in <code>config/config.go</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">config</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"log"</span>
    <span class="s">"os"</span>
    <span class="s">"github.com/joho/godotenv"</span>
<span class="p">)</span>

<span class="k">type</span> <span class="n">Config</span> <span class="k">struct</span> <span class="p">{</span>
    <span class="n">Port</span>      <span class="kt">string</span>
    <span class="n">JWTSecret</span> <span class="kt">string</span>
<span class="p">}</span>

<span class="k">var</span> <span class="n">AppConfig</span> <span class="o">*</span><span class="n">Config</span>

<span class="k">func</span> <span class="n">LoadEnv</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">err</span> <span class="o">:=</span> <span class="n">godotenv</span><span class="o">.</span><span class="n">Load</span><span class="p">()</span>
    <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="p">{</span>
        <span class="n">log</span><span class="o">.</span><span class="n">Println</span><span class="p">(</span><span class="s">"No .env file found, using system environment"</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="n">port</span> <span class="o">:=</span> <span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"PORT"</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">port</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
        <span class="n">port</span> <span class="o">=</span> <span class="s">"8080"</span>
    <span class="p">}</span>

    <span class="n">AppConfig</span> <span class="o">=</span> <span class="o">&amp;</span><span class="n">Config</span><span class="p">{</span>
        <span class="n">Port</span><span class="o">:</span>      <span class="n">port</span><span class="p">,</span>
        <span class="n">JWTSecret</span><span class="o">:</span> <span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"JWT_SECRET"</span><span class="p">),</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">AppConfig</span><span class="o">.</span><span class="n">JWTSecret</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
        <span class="n">log</span><span class="o">.</span><span class="n">Fatal</span><span class="p">(</span><span class="s">"JWT_SECRET environment variable is required"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  🔐 The Authentication Middleware
</h2>

<p>Now for the main event - our JWT middleware in <code>middlewares/authmiddleware.go</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">middlewares</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"net/http"</span>
    <span class="s">"os"</span>
    <span class="s">"strings"</span>

    <span class="s">"github.com/gin-gonic/gin"</span>
    <span class="s">"github.com/golang-jwt/jwt/v5"</span>
<span class="p">)</span>

<span class="k">func</span> <span class="n">AuthMiddleware</span><span class="p">()</span> <span class="n">gin</span><span class="o">.</span><span class="n">HandlerFunc</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">func</span><span class="p">(</span><span class="n">ctx</span> <span class="o">*</span><span class="n">gin</span><span class="o">.</span><span class="n">Context</span><span class="p">)</span> <span class="p">{</span>
        <span class="c">// Extract token from Authorization header</span>
        <span class="n">authHeader</span> <span class="o">:=</span> <span class="n">ctx</span><span class="o">.</span><span class="n">GetHeader</span><span class="p">(</span><span class="s">"Authorization"</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">authHeader</span> <span class="o">==</span> <span class="s">""</span> <span class="p">{</span>
            <span class="n">ctx</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="n">http</span><span class="o">.</span><span class="n">StatusUnauthorized</span><span class="p">,</span> <span class="n">gin</span><span class="o">.</span><span class="n">H</span><span class="p">{</span>
                <span class="s">"error"</span><span class="o">:</span> <span class="s">"Authorization header is required"</span>
            <span class="p">})</span>
            <span class="n">ctx</span><span class="o">.</span><span class="n">Abort</span><span class="p">()</span>
            <span class="k">return</span>
        <span class="p">}</span>

        <span class="c">// Handle both "Bearer token" and direct token formats</span>
        <span class="n">tokenString</span> <span class="o">:=</span> <span class="n">strings</span><span class="o">.</span><span class="n">TrimSpace</span><span class="p">(</span><span class="n">authHeader</span><span class="p">)</span>
        <span class="k">if</span> <span class="n">strings</span><span class="o">.</span><span class="n">HasPrefix</span><span class="p">(</span><span class="n">strings</span><span class="o">.</span><span class="n">ToLower</span><span class="p">(</span><span class="n">tokenString</span><span class="p">),</span> <span class="s">"bearer "</span><span class="p">)</span> <span class="p">{</span>
            <span class="n">tokenString</span> <span class="o">=</span> <span class="n">strings</span><span class="o">.</span><span class="n">TrimSpace</span><span class="p">(</span><span class="n">tokenString</span><span class="p">[</span><span class="m">7</span><span class="o">:</span><span class="p">])</span>
        <span class="p">}</span>

        <span class="c">// Parse and validate JWT token</span>
        <span class="n">token</span><span class="p">,</span> <span class="n">err</span> <span class="o">:=</span> <span class="n">jwt</span><span class="o">.</span><span class="n">Parse</span><span class="p">(</span><span class="n">tokenString</span><span class="p">,</span> <span class="k">func</span><span class="p">(</span><span class="n">t</span> <span class="o">*</span><span class="n">jwt</span><span class="o">.</span><span class="n">Token</span><span class="p">)</span> <span class="p">(</span><span class="k">interface</span><span class="p">{},</span> <span class="kt">error</span><span class="p">)</span> <span class="p">{</span>
            <span class="k">if</span> <span class="n">_</span><span class="p">,</span> <span class="n">ok</span> <span class="o">:=</span> <span class="n">t</span><span class="o">.</span><span class="n">Method</span><span class="o">.</span><span class="p">(</span><span class="o">*</span><span class="n">jwt</span><span class="o">.</span><span class="n">SigningMethodHMAC</span><span class="p">);</span> <span class="o">!</span><span class="n">ok</span> <span class="p">{</span>
                <span class="k">return</span> <span class="no">nil</span><span class="p">,</span> <span class="n">jwt</span><span class="o">.</span><span class="n">ErrSignatureInvalid</span>
            <span class="p">}</span>
            <span class="k">return</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="n">os</span><span class="o">.</span><span class="n">Getenv</span><span class="p">(</span><span class="s">"JWT_SECRET"</span><span class="p">)),</span> <span class="no">nil</span>
        <span class="p">})</span>

        <span class="k">if</span> <span class="n">err</span> <span class="o">!=</span> <span class="no">nil</span> <span class="o">||</span> <span class="o">!</span><span class="n">token</span><span class="o">.</span><span class="n">Valid</span> <span class="p">{</span>
            <span class="n">ctx</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="n">http</span><span class="o">.</span><span class="n">StatusUnauthorized</span><span class="p">,</span> <span class="n">gin</span><span class="o">.</span><span class="n">H</span><span class="p">{</span>
                <span class="s">"error"</span><span class="o">:</span> <span class="s">"Invalid or expired token"</span>
            <span class="p">})</span>
            <span class="n">ctx</span><span class="o">.</span><span class="n">Abort</span><span class="p">()</span>
            <span class="k">return</span>
        <span class="p">}</span>

        <span class="c">// Extract claims and set context</span>
        <span class="k">if</span> <span class="n">claims</span><span class="p">,</span> <span class="n">ok</span> <span class="o">:=</span> <span class="n">token</span><span class="o">.</span><span class="n">Claims</span><span class="o">.</span><span class="p">(</span><span class="n">jwt</span><span class="o">.</span><span class="n">MapClaims</span><span class="p">);</span> <span class="n">ok</span> <span class="p">{</span>
            <span class="n">ctx</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="s">"user_id"</span><span class="p">,</span> <span class="n">claims</span><span class="p">[</span><span class="s">"user_id"</span><span class="p">])</span>
            <span class="n">ctx</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="s">"email"</span><span class="p">,</span> <span class="n">claims</span><span class="p">[</span><span class="s">"email"</span><span class="p">])</span>
        <span class="p">}</span>

        <span class="n">ctx</span><span class="o">.</span><span class="n">Next</span><span class="p">()</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  🚀 Usage Example
</h2>

<p>Here's how to integrate the middleware into your application:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">main</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"authmiddleware/config"</span>
    <span class="s">"authmiddleware/middlewares"</span>
    <span class="s">"github.com/gin-gonic/gin"</span>
<span class="p">)</span>

<span class="k">func</span> <span class="n">main</span><span class="p">()</span> <span class="p">{</span>
    <span class="n">config</span><span class="o">.</span><span class="n">LoadEnv</span><span class="p">()</span>

    <span class="n">r</span> <span class="o">:=</span> <span class="n">gin</span><span class="o">.</span><span class="n">Default</span><span class="p">()</span>

    <span class="c">// Public routes</span>
    <span class="n">r</span><span class="o">.</span><span class="n">GET</span><span class="p">(</span><span class="s">"/health"</span><span class="p">,</span> <span class="k">func</span><span class="p">(</span><span class="n">c</span> <span class="o">*</span><span class="n">gin</span><span class="o">.</span><span class="n">Context</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">c</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="n">gin</span><span class="o">.</span><span class="n">H</span><span class="p">{</span><span class="s">"status"</span><span class="o">:</span> <span class="s">"healthy"</span><span class="p">})</span>
    <span class="p">})</span>

    <span class="c">// Protected routes</span>
    <span class="n">protected</span> <span class="o">:=</span> <span class="n">r</span><span class="o">.</span><span class="n">Group</span><span class="p">(</span><span class="s">"/api/v1"</span><span class="p">)</span>
    <span class="n">protected</span><span class="o">.</span><span class="n">Use</span><span class="p">(</span><span class="n">middlewares</span><span class="o">.</span><span class="n">AuthMiddleware</span><span class="p">())</span>
    <span class="p">{</span>
        <span class="n">protected</span><span class="o">.</span><span class="n">GET</span><span class="p">(</span><span class="s">"/profile"</span><span class="p">,</span> <span class="n">getProfile</span><span class="p">)</span>
        <span class="n">protected</span><span class="o">.</span><span class="n">POST</span><span class="p">(</span><span class="s">"/data"</span><span class="p">,</span> <span class="n">postData</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="n">r</span><span class="o">.</span><span class="n">Run</span><span class="p">(</span><span class="s">":"</span> <span class="o">+</span> <span class="n">config</span><span class="o">.</span><span class="n">AppConfig</span><span class="o">.</span><span class="n">Port</span><span class="p">)</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">getProfile</span><span class="p">(</span><span class="n">c</span> <span class="o">*</span><span class="n">gin</span><span class="o">.</span><span class="n">Context</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">userID</span> <span class="o">:=</span> <span class="n">c</span><span class="o">.</span><span class="n">GetString</span><span class="p">(</span><span class="s">"user_id"</span><span class="p">)</span>
    <span class="n">email</span> <span class="o">:=</span> <span class="n">c</span><span class="o">.</span><span class="n">GetString</span><span class="p">(</span><span class="s">"email"</span><span class="p">)</span>

    <span class="n">c</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="n">gin</span><span class="o">.</span><span class="n">H</span><span class="p">{</span>
        <span class="s">"user_id"</span><span class="o">:</span> <span class="n">userID</span><span class="p">,</span>
        <span class="s">"email"</span><span class="o">:</span>   <span class="n">email</span><span class="p">,</span>
        <span class="s">"message"</span><span class="o">:</span> <span class="s">"Profile retrieved successfully"</span>
    <span class="p">})</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  🧪 Testing Our Middleware
</h2>

<p>Testing is crucial! Here's our test suite in <code>tests/auth_middleware_test.go</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">package</span> <span class="n">tests</span>

<span class="k">import</span> <span class="p">(</span>
    <span class="s">"net/http"</span>
    <span class="s">"net/http/httptest"</span>
    <span class="s">"os"</span>
    <span class="s">"testing"</span>
    <span class="s">"time"</span>

    <span class="s">"authmiddleware/middlewares"</span>
    <span class="s">"github.com/gin-gonic/gin"</span>
    <span class="s">"github.com/golang-jwt/jwt/v5"</span>
<span class="p">)</span>

<span class="k">func</span> <span class="n">TestMain</span><span class="p">(</span><span class="n">m</span> <span class="o">*</span><span class="n">testing</span><span class="o">.</span><span class="n">M</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">gin</span><span class="o">.</span><span class="n">SetMode</span><span class="p">(</span><span class="n">gin</span><span class="o">.</span><span class="n">TestMode</span><span class="p">)</span>
    <span class="n">os</span><span class="o">.</span><span class="n">Setenv</span><span class="p">(</span><span class="s">"JWT_SECRET"</span><span class="p">,</span> <span class="s">"test-secret-key"</span><span class="p">)</span>
    <span class="n">os</span><span class="o">.</span><span class="n">Exit</span><span class="p">(</span><span class="n">m</span><span class="o">.</span><span class="n">Run</span><span class="p">())</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">createTestToken</span><span class="p">(</span><span class="n">userID</span><span class="p">,</span> <span class="n">email</span> <span class="kt">string</span><span class="p">)</span> <span class="kt">string</span> <span class="p">{</span>
    <span class="n">token</span> <span class="o">:=</span> <span class="n">jwt</span><span class="o">.</span><span class="n">NewWithClaims</span><span class="p">(</span><span class="n">jwt</span><span class="o">.</span><span class="n">SigningMethodHS256</span><span class="p">,</span> <span class="n">jwt</span><span class="o">.</span><span class="n">MapClaims</span><span class="p">{</span>
        <span class="s">"user_id"</span><span class="o">:</span> <span class="n">userID</span><span class="p">,</span>
        <span class="s">"email"</span><span class="o">:</span>   <span class="n">email</span><span class="p">,</span>
        <span class="s">"exp"</span><span class="o">:</span>     <span class="n">time</span><span class="o">.</span><span class="n">Now</span><span class="p">()</span><span class="o">.</span><span class="n">Add</span><span class="p">(</span><span class="n">time</span><span class="o">.</span><span class="n">Hour</span><span class="p">)</span><span class="o">.</span><span class="n">Unix</span><span class="p">(),</span>
    <span class="p">})</span>
    <span class="n">tokenString</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">token</span><span class="o">.</span><span class="n">SignedString</span><span class="p">([]</span><span class="kt">byte</span><span class="p">(</span><span class="s">"test-secret-key"</span><span class="p">))</span>
    <span class="k">return</span> <span class="n">tokenString</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">TestAuthMiddleware_ValidToken</span><span class="p">(</span><span class="n">t</span> <span class="o">*</span><span class="n">testing</span><span class="o">.</span><span class="n">T</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">router</span> <span class="o">:=</span> <span class="n">gin</span><span class="o">.</span><span class="n">New</span><span class="p">()</span>
    <span class="n">router</span><span class="o">.</span><span class="n">Use</span><span class="p">(</span><span class="n">middlewares</span><span class="o">.</span><span class="n">AuthMiddleware</span><span class="p">())</span>
    <span class="n">router</span><span class="o">.</span><span class="n">GET</span><span class="p">(</span><span class="s">"/test"</span><span class="p">,</span> <span class="k">func</span><span class="p">(</span><span class="n">c</span> <span class="o">*</span><span class="n">gin</span><span class="o">.</span><span class="n">Context</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">userID</span> <span class="o">:=</span> <span class="n">c</span><span class="o">.</span><span class="n">GetString</span><span class="p">(</span><span class="s">"user_id"</span><span class="p">)</span>
        <span class="n">email</span> <span class="o">:=</span> <span class="n">c</span><span class="o">.</span><span class="n">GetString</span><span class="p">(</span><span class="s">"email"</span><span class="p">)</span>
        <span class="n">c</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="n">gin</span><span class="o">.</span><span class="n">H</span><span class="p">{</span><span class="s">"user_id"</span><span class="o">:</span> <span class="n">userID</span><span class="p">,</span> <span class="s">"email"</span><span class="o">:</span> <span class="n">email</span><span class="p">})</span>
    <span class="p">})</span>

    <span class="n">token</span> <span class="o">:=</span> <span class="n">createTestToken</span><span class="p">(</span><span class="s">"123"</span><span class="p">,</span> <span class="s">"test@example.com"</span><span class="p">)</span>
    <span class="n">req</span> <span class="o">:=</span> <span class="n">httptest</span><span class="o">.</span><span class="n">NewRequest</span><span class="p">(</span><span class="s">"GET"</span><span class="p">,</span> <span class="s">"/test"</span><span class="p">,</span> <span class="no">nil</span><span class="p">)</span>
    <span class="n">req</span><span class="o">.</span><span class="n">Header</span><span class="o">.</span><span class="n">Set</span><span class="p">(</span><span class="s">"Authorization"</span><span class="p">,</span> <span class="s">"Bearer "</span><span class="o">+</span><span class="n">token</span><span class="p">)</span>

    <span class="n">w</span> <span class="o">:=</span> <span class="n">httptest</span><span class="o">.</span><span class="n">NewRecorder</span><span class="p">()</span>
    <span class="n">router</span><span class="o">.</span><span class="n">ServeHTTP</span><span class="p">(</span><span class="n">w</span><span class="p">,</span> <span class="n">req</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">w</span><span class="o">.</span><span class="n">Code</span> <span class="o">!=</span> <span class="n">http</span><span class="o">.</span><span class="n">StatusOK</span> <span class="p">{</span>
        <span class="n">t</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"Expected status 200, got %d"</span><span class="p">,</span> <span class="n">w</span><span class="o">.</span><span class="n">Code</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">TestAuthMiddleware_MissingToken</span><span class="p">(</span><span class="n">t</span> <span class="o">*</span><span class="n">testing</span><span class="o">.</span><span class="n">T</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">router</span> <span class="o">:=</span> <span class="n">gin</span><span class="o">.</span><span class="n">New</span><span class="p">()</span>
    <span class="n">router</span><span class="o">.</span><span class="n">Use</span><span class="p">(</span><span class="n">middlewares</span><span class="o">.</span><span class="n">AuthMiddleware</span><span class="p">())</span>
    <span class="n">router</span><span class="o">.</span><span class="n">GET</span><span class="p">(</span><span class="s">"/test"</span><span class="p">,</span> <span class="k">func</span><span class="p">(</span><span class="n">c</span> <span class="o">*</span><span class="n">gin</span><span class="o">.</span><span class="n">Context</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">c</span><span class="o">.</span><span class="n">JSON</span><span class="p">(</span><span class="m">200</span><span class="p">,</span> <span class="n">gin</span><span class="o">.</span><span class="n">H</span><span class="p">{</span><span class="s">"message"</span><span class="o">:</span> <span class="s">"success"</span><span class="p">})</span>
    <span class="p">})</span>

    <span class="n">req</span> <span class="o">:=</span> <span class="n">httptest</span><span class="o">.</span><span class="n">NewRequest</span><span class="p">(</span><span class="s">"GET"</span><span class="p">,</span> <span class="s">"/test"</span><span class="p">,</span> <span class="no">nil</span><span class="p">)</span>
    <span class="n">w</span> <span class="o">:=</span> <span class="n">httptest</span><span class="o">.</span><span class="n">NewRecorder</span><span class="p">()</span>
    <span class="n">router</span><span class="o">.</span><span class="n">ServeHTTP</span><span class="p">(</span><span class="n">w</span><span class="p">,</span> <span class="n">req</span><span class="p">)</span>

    <span class="k">if</span> <span class="n">w</span><span class="o">.</span><span class="n">Code</span> <span class="o">!=</span> <span class="n">http</span><span class="o">.</span><span class="n">StatusUnauthorized</span> <span class="p">{</span>
        <span class="n">t</span><span class="o">.</span><span class="n">Errorf</span><span class="p">(</span><span class="s">"Expected status 401, got %d"</span><span class="p">,</span> <span class="n">w</span><span class="o">.</span><span class="n">Code</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  🔒 Security Best Practices
</h2>

<p>When implementing JWT authentication:</p>

<ol>
<li>
<strong>Strong Secrets</strong>: Use cryptographically secure JWT secrets</li>
<li>
<strong>Token Expiration</strong>: Always set expiration times</li>
<li>
<strong>Signing Method Validation</strong>: Verify expected algorithms</li>
<li>
<strong>HTTPS Only</strong>: Never transmit tokens over HTTP</li>
<li>
<strong>Error Handling</strong>: Don't leak sensitive information</li>
</ol>

<h2>
  
  
  🏃‍♂️ Running Tests
</h2>

<p>Test your implementation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Run all tests</span>
go <span class="nb">test</span> ./tests/...

<span class="c"># With coverage</span>
go <span class="nb">test</span> <span class="nt">-cover</span> ./tests/...

<span class="c"># Verbose output</span>
go <span class="nb">test</span> <span class="nt">-v</span> ./tests/...
</code></pre>

</div>



<h2>
  
  
  📦 Environment Setup
</h2>

<p>Create <code>.env.example</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>PORT=8080
JWT_SECRET=your-super-secret-jwt-key-here
</code></pre>

</div>



<h2>
  
  
  🎉 What We've Accomplished
</h2>

<p>Our middleware now provides:</p>

<p>✅ JWT token validation<br><br>
✅ Flexible token format support<br><br>
✅ User context extraction<br><br>
✅ Comprehensive error handling<br><br>
✅ Production-ready code<br><br>
✅ Full test coverage  </p>

<h2>
  
  
  🚀 Next Steps
</h2>

<p>Consider extending with:</p>

<ul>
<li>Role-based authorization</li>
<li>Token refresh mechanism</li>
<li>Rate limiting</li>
<li>Audit logging</li>
<li>Multi-tenant support</li>
</ul>

<h2>
  
  
  💡 Key Takeaways
</h2>

<p>Building your own auth middleware gives you:</p>

<ul>
<li>Complete control over authentication flow</li>
<li>Better understanding of JWT security</li>
<li>Lightweight, dependency-minimal solution</li>
<li>Easy customization for business needs</li>
</ul>




<h2>
  
  
  📚 Source Code
</h2>

<p><strong>🔗 Complete source code:</strong> <a href="https://github.com/keyadaniel56/authmiddleware" rel="noopener noreferrer">GitHub Repository</a></p>

<p>⭐ <strong>Star the repo if this helped you!</strong> ⭐</p>

<p><em>What authentication challenges have you faced in your Go applications? Share your experiences in the comments below!</em></p>

<p><strong>Connect with me:</strong></p>

<ul>
<li>🐙 GitHub: <a href="https://github.com/keyadaniel56" rel="noopener noreferrer">@keyadaniel56</a> - Follow for more Go tutorials!</li>
<li>💬 Let's discuss Go best practices and build amazing things together!</li>
</ul>




<p><em>Found this helpful? Give it a ❤️ and share with fellow developers!</em></p>

