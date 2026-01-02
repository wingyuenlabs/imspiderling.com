---
Title: Versioning Redis Cache Keys to Prevent Stale Data During Spring Boot Deployments
Description: 
Author: İbrahim Gündüz
Date: 2026-01-02T21:56:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>Applications often generate content that needs to be consumed later at runtime. This content may be stored in the file system, written to a database table, shared through a message broker, or placed in a shared cache. Since software systems evolve over time, the consumers of previously generated content are expected to handle these changes. However, this is not always possible.</p>

<p>Let's take a real world example:</p>

<h2>
  
  
  Example: Structure Changes On Cached Objects
</h2>

<p>The class shown below retrieves data for a specific user from the users table, constructs a UserProfile object, and caches the result in Redis.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">UserProfileService</span> <span class="o">{</span>
    <span class="kd">private</span> <span class="kd">final</span> <span class="nc">UserRepository</span> <span class="n">userRepository</span><span class="o">;</span>

    <span class="kd">public</span> <span class="nf">UserProfileService</span><span class="o">(</span><span class="nc">UserRepository</span> <span class="n">userRepository</span><span class="o">)</span> <span class="o">{</span>
        <span class="k">this</span><span class="o">.</span><span class="na">userRepository</span> <span class="o">=</span> <span class="n">userRepository</span><span class="o">;</span>
    <span class="o">}</span>

    <span class="nd">@Cacheable</span><span class="o">(</span><span class="s">"userProfile"</span><span class="o">)</span>
    <span class="kd">public</span> <span class="nc">UserProfile</span> <span class="nf">getProfile</span><span class="o">(</span><span class="nc">Long</span> <span class="n">userId</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">User</span> <span class="n">user</span> <span class="o">=</span> <span class="n">userRepository</span><span class="o">.</span><span class="na">findById</span><span class="o">(</span><span class="n">userId</span><span class="o">).</span><span class="na">orElseThrow</span><span class="o">();</span>
        <span class="k">return</span> <span class="k">new</span> <span class="nf">UserProfile</span><span class="o">(</span>
                <span class="n">user</span><span class="o">.</span><span class="na">getId</span><span class="o">(),</span>
                <span class="n">user</span><span class="o">.</span><span class="na">getFirstName</span><span class="o">(),</span>
                <span class="n">user</span><span class="o">.</span><span class="na">getLastName</span><span class="o">()</span>
        <span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>At a later point, we decided to extend the UserProfile object by adding an email field and populating it from the database.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight diff"><code>         return new UserProfile(
                 user.getId(),
                 user.getFirstName(),
<span class="gd">-                user.getLastName()
</span><span class="gi">+                user.getLastName(),
+                user.getEmail()
</span>         );
     }
 }
</code></pre>

</div>



<p>Because Spring had already populated the cache with instances of the previous object structure, subsequent calls to the getProfile() method would fail due to a deserialization error when reading the outdated cached values.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>org.springframework.data.redis.serializer.SerializationException: Cannot serialize
</code></pre>

</div>



<p>Or something similar depending on your serializer configuration.</p>

<h2>
  
  
  How To Solve
</h2>

<p>Although both the problem and its solution are conceptually simple, the impact can be severe in production environments. Such errors may cause extended downtime, failed deployments, or forced rollbacks — especially when cache entries have long TTL values.</p>

<p>To enable safe deployments and rollbacks, cache entries must be versioned. A common and effective approach is to include a version identifier—optionally combined with the application name when using a shared cache—in the cache key. This ensures that newer application versions do not attempt to read incompatible cached data created by older versions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Configuration</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">CacheConfiguration</span> <span class="o">{</span>
    <span class="nd">@Bean</span>
    <span class="kd">public</span> <span class="nc">RedisCacheConfiguration</span> <span class="nf">redisCacheConfiguration</span><span class="o">(</span><span class="nc">ObjectProvider</span><span class="o">&lt;</span><span class="nc">BuildProperties</span><span class="o">&gt;</span> <span class="n">buildPropertiesProvider</span><span class="o">)</span> <span class="o">{</span>
        <span class="nc">BuildProperties</span> <span class="n">buildProperties</span> <span class="o">=</span> <span class="n">buildPropertiesProvider</span><span class="o">.</span><span class="na">getIfAvailable</span><span class="o">();</span>

        <span class="nc">String</span> <span class="n">name</span> <span class="o">=</span> <span class="n">buildProperties</span> <span class="o">!=</span> <span class="kc">null</span> <span class="o">?</span> <span class="n">buildProperties</span><span class="o">.</span><span class="na">getName</span><span class="o">()</span> <span class="o">:</span> <span class="s">"application"</span><span class="o">;</span>
        <span class="nc">String</span> <span class="n">version</span> <span class="o">=</span> <span class="n">buildProperties</span> <span class="o">!=</span> <span class="kc">null</span> <span class="o">?</span> <span class="n">buildProperties</span><span class="o">.</span><span class="na">getVersion</span><span class="o">()</span> <span class="o">:</span> <span class="s">"dev"</span><span class="o">;</span>

        <span class="k">return</span> <span class="nc">RedisCacheConfiguration</span><span class="o">.</span><span class="na">defaultCacheConfig</span><span class="o">()</span>
                <span class="o">.</span><span class="na">prefixCacheNameWith</span><span class="o">(</span><span class="nc">String</span><span class="o">.</span><span class="na">format</span><span class="o">(</span><span class="s">"%s:%s:"</span><span class="o">,</span> <span class="n">name</span><span class="o">,</span> <span class="n">version</span><span class="o">));</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>The cache key can be constructed using build information from Maven (via Maven resource filtering or the spring-boot-maven-plugin with the build-info goal), a Git commit hash (for example, using the git-commit-id-plugin), or any other metadata injected by the CI/CD pipeline.</p>

<p>When Redis is configured with a global key prefix as shown above, cached entries are stored with versioned keys, for example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>127.0.0.1:6379&gt; keys *
1) "payment-gateway:1.1.0:userProfile::1"
2) "payment-gateway:1.1.1:userProfile::1"
</code></pre>

</div>



<p>Because each application instance accesses only cache keys matching its own version, deployments and rollbacks can be performed safely without encountering deserialization errors caused by incompatible cached data.</p>

<p>--</p>

<p>Hope you find it helpful. As always, you can find code example in the following repository.</p>

<ul>
<li><a href="https://github.com/ibrahimgunduz34/blog-code-examples/tree/master/cache-versioning" rel="noopener noreferrer">Cache Key Versioning Code Example</a></li>
</ul>

<p>Thanks for reading!</p>

<h2>
  
  
  Credits
</h2>

<ul>
<li><a href="https://www.baeldung.com/spring-boot-build-properties" rel="noopener noreferrer">Add Build Properties to a Spring Boot Application</a></li>
<li><a href="https://www.baeldung.com/spring-git-information" rel="noopener noreferrer">Injecting Git Information Into Spring</a></li>
<li><a href="https://docs.spring.io/spring-data/redis/reference/redis/redis-cache.html" rel="noopener noreferrer">Spring Data Redis Configuration</a></li>
</ul>

