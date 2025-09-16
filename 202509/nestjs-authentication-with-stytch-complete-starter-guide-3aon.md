---
Title: NestJS Authentication with Stytch: Complete Starter Guide
Description: 
Author: Michał Miler
Date: 2025-09-16T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Implementing authentication in <strong>NestJS</strong> can be challenging, especially when ensuring security and scalability. Instead of building authentication from scratch, you can integrate <a href="https://stytch.com/" rel="noopener noreferrer"><strong>Stytch</strong></a>, a modern authentication platform, to handle secure user management with ease. In this guide, we’ll walk through the <a href="https://github.com/u11d-com/stytch-nestjs-starter" rel="noopener noreferrer"><code>stytch-nestjs-starter</code></a> repository - a production-ready example that shows how to integrate <strong>Stytch authentication with NestJS</strong> using smart caching and session management.</p>

<h2>
  
  
  <strong>Why Choose Stytch Over Building Your Own Auth?</strong>
</h2>

<p>Building authentication from scratch means dealing with:</p>

<ul>
<li><strong>Password hashing and salt management</strong></li>
<li><strong>Session security and token management</strong></li>
<li><strong>Account verification and password reset flows</strong></li>
<li><strong>Rate limiting and brute force protection</strong></li>
<li><strong>Compliance with security standards</strong></li>
<li><strong>Regular security updates and patches</strong></li>
</ul>

<p>Stytch eliminates these concerns by providing:</p>

<ul>
<li>✅ <strong>Battle-tested security</strong> with enterprise-grade protection</li>
<li>✅ <strong>Comprehensive authentication methods</strong> (email/password, magic links, social login, MFA)</li>
<li>✅ <strong>Global infrastructure</strong> with 99.99% uptime SLA</li>
<li>✅ <strong>Developer-friendly APIs</strong> with excellent documentation</li>
<li>✅ <strong>Compliance certifications</strong> (SOC 2, ISO 27001, GDPR)</li>
</ul>

<h2>
  
  
  <strong>Understanding the Integration Approaches</strong>
</h2>

<h3>
  
  
  <strong>Frontend or Backend Integration</strong>
</h3>

<p>Stytch offers two primary integration patterns, each suited for different application architectures:</p>

<p><strong>Frontend Integration</strong>: Ideal for client-side applications where authentication logic runs in the browser. This approach uses Stytch's JavaScript SDK to handle authentication flows directly on the frontend, with the backend serving as a resource server that validates session tokens.</p>

<p><strong>Backend Integration</strong>: Perfect for server-side rendered applications or when you need centralized authentication control. This is the approach demonstrated in our starter, where the NestJS backend handles all authentication logic using Stytch's server-side APIs, providing better security and session management control.</p>

<p>Our starter implements the backend integration pattern, giving you:</p>

<ul>
<li>
<strong>Enhanced Security</strong>: Sensitive operations happen server-side</li>
<li>
<strong>Centralized Session Management</strong>: All authentication logic in one place</li>
<li>
<strong>Better Performance</strong>: Server-side caching reduces API calls</li>
<li>
<strong>Simplified Frontend</strong>: Frontend only needs to handle session tokens</li>
</ul>

<p>Learn more about integration patterns: <a href="https://stytch.com/docs/getting-started/stytch-architecture" rel="noopener noreferrer">Stytch Architecture Guide</a></p>

<h3>
  
  
  <strong>Consumer vs B2B</strong>
</h3>

<p>Stytch supports both Consumer (B2C) and Business (B2B) authentication models:</p>

<p><strong>Consumer Authentication (B2C)</strong>: Designed for applications serving individual users. Features include:</p>

<ul>
<li>Individual user accounts with email/password authentication</li>
<li>Social login providers (Google, Apple, Facebook, etc.)</li>
<li>Magic links and SMS-based authentication</li>
<li>Self-service password reset and account management</li>
</ul>

<p><strong>Business Authentication (B2B)</strong>: Built for applications serving organizations and teams. Includes:</p>

<ul>
<li>Organization-based user management</li>
<li>Single Sign-On (SSO) with SAML and OIDC</li>
<li>Just-in-Time (JIT) provisioning</li>
<li>Advanced admin controls and member management</li>
<li>Multi-tenant architecture support</li>
</ul>

<p>Our starter is configured for <strong>Consumer Authentication</strong>, making it ideal for SaaS applications, marketplaces, and consumer-facing platforms. However, the architecture can be easily adapted for B2B use cases by switching to Stytch's B2B APIs.</p>

<p>Learn more about the differences: <a href="https://stytch.com/docs/getting-started/b2b-vs-consumer-auth" rel="noopener noreferrer">B2B vs Consumer Auth</a></p>

<h2>
  
  
  <strong>Architecture Deep Dive</strong>
</h2>

<h3>
  
  
  <strong>System Architecture Overview</strong>
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F633t3qlqxegwijwkxe7e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F633t3qlqxegwijwkxe7e.png" alt="System Architecture Overview" width="743" height="339"></a></p>

<h3>
  
  
  <strong>The Complete Authentication Flow</strong>
</h3>

<p>Let's visualize how authentication works in our starter:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feufyusi8bn7o0wxu4n70.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feufyusi8bn7o0wxu4n70.png" alt="The Complete Authentication Flow" width="724" height="552"></a></p>

<h2>
  
  
  <strong>Exploring the Starter Repository</strong>
</h2>

<h3>
  
  
  <strong>Key Features Implemented</strong>
</h3>

<h3>
  
  
  <strong>1. Smart Session Caching</strong>
</h3>

<p>One of the most critical optimizations in the starter is the Redis-based session caching:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="c1">// Storing sessions in Redis with TTL</span>
<span class="k">private</span> <span class="k">async</span> <span class="nf">storeSession</span><span class="p">(</span>
  <span class="nx">sessionToken</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span>
  <span class="p">{</span> <span class="nx">session_id</span><span class="p">,</span> <span class="na">user_id</span><span class="p">:</span> <span class="nx">stytchUserId</span><span class="p">,</span> <span class="nx">expires_at</span> <span class="p">}:</span> <span class="nx">Session</span><span class="p">,</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="nx">SessionTokenModel</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">userService</span><span class="p">.</span><span class="nf">fetchByStytchId</span><span class="p">(</span><span class="nx">stytchUserId</span><span class="p">);</span>

  <span class="kd">const</span> <span class="na">value</span><span class="p">:</span> <span class="nx">CachedSession</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">userId</span><span class="p">:</span> <span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">,</span>
    <span class="nx">stytchUserId</span><span class="p">,</span>
  <span class="p">};</span>

  <span class="kd">const</span> <span class="nx">ttl</span> <span class="o">=</span> <span class="nx">expires_at</span>
    <span class="p">?</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="nx">expires_at</span><span class="p">).</span><span class="nf">getTime</span><span class="p">()</span> <span class="o">-</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">getTime</span><span class="p">()</span>
    <span class="p">:</span> <span class="k">this</span><span class="p">.</span><span class="nx">stytchService</span><span class="p">.</span><span class="nx">sessionDurationMinutes</span> <span class="o">*</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">hashedSessionToken</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">hashSessionToken</span><span class="p">(</span><span class="nx">sessionToken</span><span class="p">);</span>
  <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">cacheManager</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">hashedSessionToken</span><span class="p">,</span> <span class="nx">value</span><span class="p">,</span> <span class="nx">ttl</span><span class="p">);</span>

  <span class="k">return</span> <span class="p">{</span> <span class="nx">sessionToken</span> <span class="p">};</span>
<span class="p">}</span>

</code></pre>

</div>



<p>This approach means:</p>

<ul>
<li>
<strong>No Stytch API calls</strong> on every request</li>
<li><strong>Sub-millisecond session verification</strong></li>
<li>
<strong>Automatic session expiration</strong> matching Stytch TTL</li>
<li>
<strong>Reduced API costs</strong> and improved performance</li>
</ul>

<h3>
  
  
  <strong>2. Intelligent Session Refresh</strong>
</h3>

<p>The session refresh interceptor automatically extends sessions before they expire:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">private</span> <span class="k">async</span> <span class="nf">checkAndRefreshSession</span><span class="p">(</span>
  <span class="nx">request</span><span class="p">:</span> <span class="nx">RequestWithUser</span><span class="p">,</span>
  <span class="nx">response</span><span class="p">:</span> <span class="nx">Response</span><span class="p">,</span>
<span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">sessionToken</span> <span class="o">=</span> <span class="nx">request</span><span class="p">.</span><span class="nx">headers</span><span class="p">[</span><span class="dl">'</span><span class="s1">authorization</span><span class="dl">'</span><span class="p">]?.</span><span class="nf">split</span><span class="p">(</span><span class="dl">'</span><span class="s1"> </span><span class="dl">'</span><span class="p">)[</span><span class="mi">1</span><span class="p">];</span>

  <span class="kd">const</span> <span class="nx">hashedSessionToken</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">authService</span><span class="p">.</span><span class="nf">hashSessionToken</span><span class="p">(</span><span class="nx">sessionToken</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">sessionTtl</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">cacheManager</span><span class="p">.</span><span class="nf">ttl</span><span class="p">(</span><span class="nx">hashedSessionToken</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">expirationTime</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">(</span><span class="nx">sessionTtl</span><span class="p">);</span>

  <span class="kd">const</span> <span class="nx">shouldRefresh</span> <span class="o">=</span>
    <span class="nx">expirationTime</span><span class="p">.</span><span class="nf">getTime</span><span class="p">()</span> <span class="o">-</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">getTime</span><span class="p">()</span> <span class="o">&lt;=</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">sessionRefreshThresholdMinutes</span> <span class="o">*</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">;</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">shouldRefresh</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">refreshResult</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nx">authService</span><span class="p">.</span><span class="nf">refreshSession</span><span class="p">({</span>
      <span class="nx">sessionToken</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="nx">response</span><span class="p">.</span><span class="nf">setHeader</span><span class="p">(</span><span class="dl">'</span><span class="s1">X-New-Session-Token</span><span class="dl">'</span><span class="p">,</span> <span class="nx">refreshResult</span><span class="p">.</span><span class="nx">sessionToken</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p>This ensures users never experience unexpected logouts while maintaining security.</p>

<h3>
  
  
  <strong>3. Elegant Session Access with CurrentSession Decorator</strong>
</h3>

<p>The <code>CurrentSession</code> decorator provides a clean, type-safe way to access authenticated user data in your controllers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="p">@</span><span class="nd">Controller</span><span class="p">(</span><span class="dl">'</span><span class="s1">resources</span><span class="dl">'</span><span class="p">)</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">ResourceController</span> <span class="p">{</span>
  <span class="p">@</span><span class="nd">Get</span><span class="p">()</span>
  <span class="nf">findAll</span><span class="p">(@</span><span class="nd">CurrentSession</span><span class="p">()</span> <span class="nx">session</span><span class="p">:</span> <span class="nx">CachedSession</span><span class="p">):</span> <span class="nx">ResourceModel</span><span class="p">[]</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">logger</span><span class="p">.</span><span class="nf">debug</span><span class="p">(</span><span class="s2">`User </span><span class="p">${</span><span class="nx">session</span><span class="p">.</span><span class="nx">userId</span><span class="p">}</span><span class="s2"> requested all resources`</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">resourceService</span><span class="p">.</span><span class="nf">findAll</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>

</code></pre>

</div>



<p><strong>How it works:</strong></p>

<ol>
<li>
<strong>AuthGuard Integration</strong>: The guard validates the session token and attaches user data to the request</li>
<li>
<strong>Type Safety</strong>: Returns strongly-typed <code>CachedSession</code> object with <code>userId</code> and <code>stytchUserId</code>
</li>
<li>
<strong>Automatic Extraction</strong>: Seamlessly extracts session data without manual request parsing</li>
<li>
<strong>Null Safety</strong>: Returns <code>undefined</code> for unauthenticated requests, handled gracefully by the guard
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight tsx"><code><span class="k">export</span> <span class="kd">const</span> <span class="nx">CurrentSession</span> <span class="o">=</span> <span class="nf">createParamDecorator</span><span class="p">(</span>
  <span class="p">(</span><span class="nx">data</span><span class="p">:</span> <span class="nx">unknown</span><span class="p">,</span> <span class="nx">ctx</span><span class="p">:</span> <span class="nx">ExecutionContext</span><span class="p">):</span> <span class="nx">CachedSession</span> <span class="o">|</span> <span class="kc">undefined</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">request</span> <span class="o">=</span> <span class="nx">ctx</span><span class="p">.</span><span class="nf">switchToHttp</span><span class="p">().</span><span class="nx">getRequest</span><span class="o">&lt;</span><span class="nx">RequestWithUser</span><span class="o">&gt;</span><span class="p">();</span>
    <span class="k">return</span> <span class="nx">request</span><span class="p">.</span><span class="nx">user</span><span class="p">;</span>
  <span class="p">},</span>
<span class="p">);</span>

</code></pre>

</div>



<p>This pattern eliminates boilerplate code and provides a consistent way to access user context across your application.</p>

<h3>
  
  
  <strong>4. Flexible User Creation Methods</strong>
</h3>

<p>The starter supports two user creation approaches, though in most cases you'll only need one method depending on your application type.</p>

<p><strong>Method A: Self Sign-up (Public Registration)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/auth/sign-up <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "email": "user@example.com",
    "password": "JHFKhpe_t1VfBULE_IsMhWC5WN4yx0ke",
    "firstName": "John",
    "lastName": "Doe"
  }'</span>

</code></pre>

</div>



<p><strong>Method B: Admin Invitation (Dashboard Apps)</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Using the create-user script</span>
yarn create-user admin@company.com

<span class="c"># Or via API with master key</span>
curl <span class="nt">-X</span> POST http://localhost:3000/auth/invite <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"X-Api-Key: your-master-key"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "email": "newuser@company.com",
    "firstName": "Jane",
    "lastName": "Smith"
  }'</span>

</code></pre>

</div>



<p>The invitation method sends a magic link email, allowing users to set their password securely.</p>

<p>After executing the <code>create-user</code> script or calling the endpoint manually, you'll receive an email message with a magic link that should redirect you to your frontend application to complete the password setup. However, if your frontend is not built yet, you can use server endpoint directly to reset your password until the frontend is ready:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/auth/password <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{
    "sessionToken": "&lt;token_from_magin_link&gt;",
    "password": "JHFKhpe_t1VfBULE_IsMhWC5WN4yx0ke"
  }'</span>
</code></pre>

</div>



<p>Learn more about password reset flows: <a href="https://stytch.com/docs/api/password-email-reset" rel="noopener noreferrer">Stytch Password Reset API</a></p>

<h2>
  
  
  <strong>Getting Started with the Starter</strong>
</h2>

<h3>
  
  
  <strong>1. Quick Setup</strong>
</h3>

<p>Clone and configure the repository:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git clone https://github.com/u11d-com/stytch-nestjs-starter.git
<span class="nb">cd </span>stytch-nestjs-starter
yarn <span class="nb">install
cp</span> .env.example .env
</code></pre>

</div>



<h3>
  
  
  <strong>2. Stytch Configuration</strong>
</h3>

<ol>
<li>Sign up at <a href="https://stytch.com/dashboard" rel="noopener noreferrer">Stytch Dashboard</a>
</li>
<li>Create a new project</li>
<li>Copy your Project ID and Secret to <code>.env</code>:
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>STYTCH_PROJECT_ID=project-test-xxx
STYTCH_SECRET=secret-test-xxx
STYTCH_SESSION_DURATION_MINUTES=60
STYTCH_SESSION_REFRESH_THRESHOLD_MINUTES=30
</code></pre>

</div>



<h3>
  
  
  <strong>3. Infrastructure Setup</strong>
</h3>

<p>Start the required services:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Start PostgreSQL and Redis</span>
docker compose up postgres redis <span class="nt">-d</span>

<span class="c"># Run database migrations</span>
yarn migration:run

<span class="c"># Start the development server</span>
yarn start:dev
</code></pre>

</div>



<h3>
  
  
  <strong>4. Testing Authentication</strong>
</h3>

<p>Your API will be available at <code>http://localhost:3000</code>.</p>

<p>Create your first admin user (execute script and set password using magic link sent to email address):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>yarn create-user admin@yourcompany.com
</code></pre>

</div>



<p>Then test the authentication flow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Login</span>
curl <span class="nt">-X</span> POST http://localhost:3000/auth/login <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"email": "admin@yourcompany.com", "password": "YourPassword123!"}'</span>

<span class="c"># Access protected resource</span>
curl <span class="nt">-X</span> GET http://localhost:3000/resources <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer YOUR_SESSION_TOKEN"</span>

</code></pre>

</div>



<h2>
  
  
  <strong>Production Considerations</strong>
</h2>

<h3>
  
  
  <strong>Security Best Practices</strong>
</h3>

<ol>
<li>
<strong>Environment Variables</strong>: Never commit real credentials to version control</li>
<li>
<strong>Master Key</strong>: Generate a cryptographically secure master key</li>
<li>
<strong>CORS Configuration</strong>: Restrict origins to your frontend domains</li>
<li>
<strong>HTTPS</strong>: Always use HTTPS in production</li>
<li>
<strong>Rate Limiting</strong>: Implement rate limiting for auth endpoints</li>
<li>
<strong>Session Duration Strategy</strong>: Balance longer sessions for user experience with shorter sessions for stronger security</li>
</ol>

<h3>
  
  
  <strong>Performance and Scalability</strong>
</h3>

<ol>
<li>
<strong>Redis Configuration</strong>: Use Redis Cluster for high availability and scalability</li>
<li>
<strong>Connection Pooling</strong>: Optimize database connection pools for stable performance under load</li>
<li>
<strong>Extended Caching</strong>: Introduce additional caching layers (e.g., user profile caching) to reduce database load</li>
</ol>

<h2>
  
  
  <strong>Extending the Starter</strong>
</h2>

<p>The starter provides a solid foundation that can be extended with additional features:</p>

<ul>
<li>
<strong>Role-Based Access Control (RBAC)</strong>: Add user roles and permissions</li>
<li>
<strong>Multi-Factor Authentication (MFA)</strong>: Implement TOTP, SMS, or other MFA methods</li>
<li>
<strong>Social Authentication</strong>: Google, GitHub, and other OAuth providers</li>
</ul>

<p>We can prepare detailed implementations for these extensions if the community is interested. Let us know which features would be most valuable for your projects!</p>

<h2>
  
  
  <strong>Conclusion</strong>
</h2>

<p>Building secure, scalable authentication doesn't have to be complex. The <code>stytch-nestjs-starter</code> provides a production-ready foundation that combines Stytch's security expertise with NestJS's architectural excellence.</p>

<p>By leveraging this starter, you get:</p>

<p>✅ <strong>Enterprise-grade security</strong> without the complexity</p>

<p>✅ <strong>Optimized performance</strong> with intelligent caching</p>

<p>✅ <strong>Developer productivity</strong> with comprehensive tooling</p>

<p>✅ <strong>Production readiness</strong> with Docker and monitoring</p>

<p>✅ <strong>Extensibility</strong> for your specific requirements</p>

<p>Whether you're building a SaaS application, internal dashboard, or customer-facing platform, this starter eliminates weeks of authentication development while providing a secure, maintainable foundation for your project.</p>

<p>Ready to get started? Clone the repository and have secure authentication running in minutes.</p>

<h2>
  
  
  <strong>Resources</strong>
</h2>

<ul>
<li>📖 <strong>Repository</strong>: <a href="https://github.com/u11d-com/stytch-nestjs-starter" rel="noopener noreferrer">stytch-nestjs-starter</a>
</li>
<li>🔐 <strong>Stytch Documentation</strong>: <a href="https://stytch.com/docs" rel="noopener noreferrer">stytch.com/docs</a>
</li>
<li>🏗️ <strong>NestJS Documentation</strong>: <a href="https://docs.nestjs.com/" rel="noopener noreferrer">docs.nestjs.com</a>
</li>
</ul>




<p><em>Have questions or suggestions? We'd love to hear from you! Open an issue or start a discussion on GitHub.</em></p>

