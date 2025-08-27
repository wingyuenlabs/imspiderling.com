---
Title: Docker Compose: Speed Up Your Workflow with Profiles, Extends and Depends_on
Description: 
Author: Altair Lage
Date: 2025-08-27T21:48:40.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Docker Compose: Speed Up Your Workflow with <em>Profiles</em>, <em>Extends</em> and <em>Depends_on</em>
</h1>

<p>If you’re reading this article, you’ve probably found yourself wrestling with a <code>docker-compose.yml</code>, right?<br>
Whether you’re a backend developer, DevOps, cloud engineer, or a Docker beginner, as a project grows it’s common for Docker Compose to start showing day‑to‑day challenges. Maybe you have services that are only needed occasionally (like debugging or testing tools), repeated configurations across different services, or you run into errors because one container started before another was ready. Fortunately, Docker Compose has advanced features to deal with these scenarios and make your development environment more organized and flexible. Three of the main features are <strong>profiles</strong>, <strong>extends</strong> and <strong>depends_on</strong>. That’s what we’ll talk about in this article.</p>

<p>In this article, we’ll go through each of these features with explanations and practical examples. You’ll learn to use <strong>profiles</strong> to enable/disable optional services as needed, <strong>extends</strong> to prevent redundancy and repetition (the DRY – <em>Don’t Repeat Yourself</em> – principle), and to manage container startup sequencing with <strong>depends_on</strong>. We’ll also share best practices and how these resources can simplify your team’s workflow.<br>
 <br>
 </p>
<h2>
  
  
  <em>Profiles</em>: Controlling Optional Services in Docker compose
</h2>

<p>In many projects, beyond the core services—like your web app and your database—there are auxiliary services used only in specific situations: for example, an integration test container, a monitoring tool, or a debugging utility.<br>
Including all these services in Docker compose and starting them every time can waste resources and time. That’s where <strong>profiles</strong> in Docker compose come in.</p>

<p>The <strong>profiles</strong> feature lets you <strong>activate services selectively</strong>. You can assign one or more profiles to certain services so they <strong>only start when the corresponding profile is enabled</strong>. Services without a profile defined <strong>always start</strong>, while those with a profile only come up if you explicitly request that profile.<br>
This is excellent for keeping both essential services and optional ones (debug, tests, etc.) in the same Docker compose file without having them running all the time.</p>

<p>Imagine you have a main application and a database that should always run in development, but you also have a container to run integration tests and another for a fake SMTP server (to capture emails in dev) that aren’t essential. We can mark the non‑essential services as optional:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">3.9"</span>
<span class="na">services</span><span class="pi">:</span>
  <span class="na">app</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">my-web-app:1.0</span>
    <span class="c1"># ... (web application configurations, ports, volumes, etc.) e.g.:</span>
    <span class="c1"># ports:</span>
    <span class="c1">#   - "8000:8000"</span>
    <span class="c1"># depends_on:</span>
    <span class="c1">#   - db</span>

  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">POSTGRES_DB=mydb</span>
      <span class="pi">-</span> <span class="s">POSTGRES_USER=user</span>
      <span class="pi">-</span> <span class="s">POSTGRES_PASSWORD=passwd</span>
    <span class="c1"># Database service always needed in development</span>

  <span class="na">tests_runner</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">my-tests:latest</span>
    <span class="na">profiles</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">test"</span><span class="pi">]</span>
    <span class="c1"># Non-essential service to run on-demand integration tests</span>

  <span class="na">mailhog</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">mailhog/mailhog</span>
    <span class="na">profiles</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">devtools"</span><span class="pi">]</span>
    <span class="c1"># Optional service to capture and view emails sent by the app in dev</span>
</code></pre>

</div>



<p>In the YAML above, <code>app</code> and <code>db</code> don’t have a profile, therefore they will always be started. The <code>tests_runner</code> is associated with the <code>"test"</code> profile, and <code>mailhog</code> with the <code>"devtools"</code> profile. That means that by default those two won’t be executed — unless you explicitly specify them.</p>

<p>In practice, you can control this via the Docker Compose CLI:</p>

<ul>
<li>Running <code>docker compose up</code> (with no profiles) would start only the default services, i.e., <code>app</code> and <code>db</code>.</li>
<li>Running <code>docker compose --profile test up</code> would start <code>app</code>, <code>db</code> <strong>and also</strong> the <code>tests_runner</code> service (because the <strong>test</strong> profile was enabled).</li>
<li>Running <code>docker compose --profile devtools up</code> would start <code>app</code>, <code>db</code> <strong>and</strong> <code>mailhog</code>. You can even enable <strong>multiple profiles at once</strong> (for example, <code>docker compose --profile test --profile devtools up</code> to include both) or use <code>docker compose --profile "*"</code> to bring up <strong>all</strong> services from all profiles.</li>
</ul>

<p>This way, profiles let you keep a single Docker Compose file with everything your project might need, while <strong>turning services on or off as necessary</strong>. That keeps your dev environment lighter and faster day‑to‑day. For example, one team member can run <code>docker compose up</code> and focus on the essentials, while another, debugging a specific issue, can enable the <code>devtools</code> profile to see detailed logs or capture emails.<br>
 </p>
<h3>
  
  
  Important best practices:**
</h3>

<p><strong>Do not put your core containers into a profile.</strong> Leave them without a profile so they always start by default. Reserve profiles for optional components or scenario‑specific ones (like <em>test</em>, <em>debug</em>, <em>monitoring</em>, etc.).</p>

<p><strong>Name profiles clearly:</strong> Choose names that make the purpose obvious (e.g., <code>test</code>, <code>dev</code>, <code>debug</code>, <code>monitoring</code>, <code>ci</code>). Avoid generic terms so the whole team understands when to use them. Document in the project README which profiles exist and how to activate them.<br>
 <br>
 </p>
<h2>
  
  
  Reusing Configurations with <strong>extends</strong> and Eliminating Repetition
</h2>

<p>As your Docker Compose configuration grows, you may notice several services sharing common settings. For example, imagine we have two services inside a web application: a <strong>web</strong> service and a background <strong>worker</strong>, both using the <strong>same base image</strong>, mounting the <strong>same code volume</strong>, and needing the <strong>same environment variables</strong>, such as database connection strings and credentials. Repeating identical blocks of configuration for each service makes the Compose file long, repetitive, and harder to maintain: any change requires editing multiple places, going against the DRY (<em>Don’t Repeat Yourself</em>) principle.</p>

<p>To avoid duplication, Docker Compose lets you <strong>extend common configurations across services</strong>. The <code>extends</code> feature works like inheritance: you define a base service (which can even be in another file) with shared options, and then other services “inherit” from that base, overriding or adding parameters as needed. In YAML, you can also use anchors and aliases to achieve a similar effect in a simple way within the same file.</p>

<p>Example: we have two services, <strong>web</strong> and <strong>worker</strong>, that share most of the config. Let’s create a reusable base configuration and apply it to both:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># We define a YAML anchor with the common configurations</span>
<span class="na">x-base-service</span><span class="pi">:</span> <span class="nl">&amp;common_config</span>  
  <span class="na">image</span><span class="pi">:</span> <span class="s">my-web-app:1.0</span>
  <span class="na">volumes</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">.:/app</span>  <span class="c1"># mounts the current directory into the container (useful for development)</span>
  <span class="na">environment</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">DATABASE_URL=postgres://user:passwd@db:5432/mydb</span>
  <span class="na">restart</span><span class="pi">:</span> <span class="s2">"</span><span class="s">always"</span>
  <span class="c1"># ... (any other common option, e.g., network, logging configs, etc.)</span>

<span class="na">services</span><span class="pi">:</span>
  <span class="na">web</span><span class="pi">:</span>
    <span class="na">&lt;&lt;</span><span class="pi">:</span> <span class="nv">*common_config</span>      <span class="c1"># Imports all configs defined in common_config</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">npm start</span>     <span class="c1"># Specific command to run the web application</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">db</span>                 <span class="c1"># (example: web depends on db running)</span>

  <span class="na">worker</span><span class="pi">:</span>
    <span class="na">&lt;&lt;</span><span class="pi">:</span> <span class="nv">*common_config</span>      <span class="c1"># Reuses the same base configuration</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">npm run worker</span>  <span class="c1"># Specific command to run the background worker</span>

  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:15</span>
    <span class="c1"># ... (database configurations)</span>
</code></pre>

</div>



<p>In the snippet above, we use a special <code>x-base-service</code> key at the top of the YAML to define a block of common configurations identified by <code>&amp;common_config</code>. Then, in the <strong>web</strong> and <strong>worker</strong> services, we use <code>&lt;&lt;: *common_config</code> to <strong>merge</strong> those shared configurations into each service. That way, both get <code>image</code>, <code>volumes</code>, <code>environment</code>, and <code>restart</code> from the template, and we add only what changes in each one. In this case, the specific command, and in the web service, the database dependency.</p>

<p>If tomorrow you need to change an environment variable or a logging option for <em>all</em> services, just edit it once (in the anchor block) and <strong>both the web and worker containers will automatically receive the update</strong>. Much simpler than remembering to change it in two or three places.<br>
 </p>
<h4>
  
  
  The native <strong>extends</strong> directive
</h4>

<p>Compose also supports the native <code>extends</code> directive. With it, you can, for example, have a <code>common-services.yml</code> file that defines a base service, and then, in your <code>docker-compose.yml</code>, make another service extend that external definition. The effect is the same: reusing configurations. This even lets you override some values as needed.</p>

<p>Example:</p>

<p>Imagine that the <code>common-services.yml</code> in our example has the following configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">webapp</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span> <span class="s">.</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8000:8000"</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">/data"</span>
</code></pre>

</div>



<p>You could then extend these configurations in your <code>docker-compose.yml</code> using the <code>extends</code> directive in the desired services:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">web</span><span class="pi">:</span>
    <span class="na">build</span><span class="pi">:</span> <span class="s">alpine</span>
    <span class="na">command</span><span class="pi">:</span> <span class="s">echo</span>
    <span class="na">extends</span><span class="pi">:</span>
      <span class="na">file</span><span class="pi">:</span> <span class="s">common-services.yml</span>
      <span class="na">service</span><span class="pi">:</span> <span class="s">webapp</span>
  <span class="na">webapp</span><span class="pi">:</span>
    <span class="na">extends</span><span class="pi">:</span>
      <span class="na">file</span><span class="pi">:</span> <span class="s">common-services.yml</span>
      <span class="na">service</span><span class="pi">:</span> <span class="s">webapp</span>
</code></pre>

</div>



<p>You’ll get exactly the same result as if you had written a <code>docker-compose.yaml</code> with the same configuration values for build, ports and volumes directly—i.e., “hardcoded.”</p>

<p>Use whichever approach you prefer: YAML anchors are great within a single file, while <code>extends</code> shines when splitting Compose files across multiple files.</p>

<p><strong>Workflow benefits:</strong> by eliminating duplication, you <strong>reduce errors and keep the Compose configuration cleaner</strong>. In development teams, this means everyone shares consistent configurations. If multiple microservices use the same image or variables, you ensure they’re all using exactly the same values. Plus, the file becomes smaller and easier to understand—new developers can quickly identify what’s common to all services and what’s specific to each.<br>
 <br>
 </p>
<h2>
  
  
  Orchestrating Startup Order with <strong>depends_on</strong>
</h2>

<p>Another classic challenge in multi‑container environments is <strong>ensuring that certain services only start when others are already ready</strong>. Imagine your web API depends on a database. If the API container comes up <em>before</em> the database is up, the app will likely fail to connect and its startup may be compromised. These situations are frustrating, but Docker Compose helps with the <code>depends_on</code> parameter.</p>

<p><code>depends_on</code> lets you declare <strong>explicit dependencies between services</strong>, making Compose start containers in the correct order. In the short syntax, you simply list the names of the services another depends on, and Compose will ensure it starts the “dependency” containers first and stops them last. For example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">api</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">my-api:latest</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">db</span>
      <span class="pi">-</span> <span class="s">redis</span>
  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">mysql:8.0</span>
  <span class="na">redis</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">redis:7-alpine</span>
</code></pre>

</div>



<p>In the example above, when you run <code>docker compose up</code>, Compose will start <code>db</code> and <code>redis</code> first, and only then start the <code>api</code> container. Likewise, when stopping containers, it will stop <code>api</code> before bringing down <code>db</code> and <code>redis</code>. This helps avoid many issues related to startup and shutdown sequencing.</p>

<p><strong>But what if we want to make sure the database is **</strong>actually**** ready for connections before starting the API?** The good news is that recent versions of Compose support the long syntax of <code>depends_on</code>, which has conditions. We can specify a condition like <code>service_healthy</code> to indicate that the depended‑on service only counts as “ready” when its <em>healthcheck</em> is OK. Let’s improve the example by adding a <code>healthcheck</code> to the database:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">api</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">my-api:latest</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="na">db</span><span class="pi">:</span>
        <span class="na">condition</span><span class="pi">:</span> <span class="s">service_healthy</span>   <span class="c1"># wait for db healthcheck to pass</span>
      <span class="na">redis</span><span class="pi">:</span>
        <span class="na">condition</span><span class="pi">:</span> <span class="s">service_started</span>   <span class="c1"># wait for redis to start (container started)</span>
    <span class="c1"># ... (rest of API config, ports, etc.)</span>

  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">mysql:8.0</span>
    <span class="na">healthcheck</span><span class="pi">:</span>
      <span class="na">test</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">CMD"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">mysqladmin"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">ping"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">-h"</span><span class="pi">,</span> <span class="s2">"</span><span class="s">localhost"</span><span class="pi">]</span>  <span class="c1"># checks if MySQL responds</span>
      <span class="na">interval</span><span class="pi">:</span> <span class="s">5s</span>
      <span class="na">timeout</span><span class="pi">:</span> <span class="s">5s</span>
      <span class="na">retries</span><span class="pi">:</span> <span class="m">5</span>

  <span class="na">redis</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">redis:7-alpine</span>
    <span class="c1"># (we could include a healthcheck here as well, if we want to monitor readiness)</span>
</code></pre>

</div>



<p>Now, Compose will <strong>wait for the MySQL healthcheck to succeed</strong> — in this case, the <code>mysqladmin ping</code> command indicating the server is responding—before starting the <code>api</code> service.<br>
For Redis, we use <code>service_started</code> simply to ensure the container has started. There’s no wait for “Healthy”, because we might not have defined a <code>healthcheck</code> for it. <br>
This setup reflects a common scenario: wait for the database to become available and also ensure the Redis cache has started before the application comes online. In practice, <code>api</code> will only begin initializing when Compose verifies that <code>db</code> is <strong>healthy</strong> and <code>redis</code> is <strong>already running</strong>, avoiding immediate “could not connect to database” failures.<br>
 </p>

<h4>
  
  
  Important tips:
</h4>

<p>To use <code>condition: service_healthy</code>, make sure to define an adequate <code>healthcheck</code> in the dependency service. Otherwise Compose has no way to know the health state and will treat the container as ready as soon as the process starts.<br>
In the example, we used a native MySQL command (<code>mysqladmin ping</code>) to check availability. For PostgreSQL, for example, there’s <code>pg_isready</code>.<br>
Also keep in mind that although Compose waits for a successful healthcheck, <strong>it’s recommended that your application also implement connection retry logic</strong>. That gives you extra robustness in case of unexpected conditions—for example, a slight delay even after the healthcheck or a brief connection drop. In short, <code>depends_on</code> handles the initial orchestration (and already helps a lot), but good resilience practices in the application are always welcome.</p>

<p><strong>In a team development context</strong>, using <code>depends_on</code>, especially with healthchecks, standardizes how everyone brings the environment up. New developers don’t need to “guess” the order to start each service manually or run wait scripts. A simple <code>docker compose up</code> already does everything in the right sequence. This greatly reduces the “works on my machine” class of errors caused by startup race conditions, making everyone’s workflow more reliable.<br>
 <br>
 </p>

<h2>
  
  
  How These Features Help Teams
</h2>

<ul>
<li><p><strong>Profiles = Flexibility for different scenarios:</strong><br><br>
They let each team member run only the set of services needed for their task. This speeds up the development cycle. For example, running integration tests without bringing up the entire stack, or running debugging tools only when you’re going to use them. Teams can define standard profiles (like <code>dev</code>, <code>test</code>, <code>debug</code>) and thus avoid the need for multiple Compose files for environment variations. The result is a customizable yet centralized environment in a single file.<br>
 </p></li>
<li><p><strong>Extends = Consistent configuration and less repeated code:</strong><br><br>
By reusing configurations with <code>extends</code> or anchors, you ensure all related services share identical parameters where it makes sense: same base image, same credentials, same logging policies, etc. This <strong>avoids discrepancies</strong> that could cause “it works here but not there.” Also, reducing repetition makes editing the Compose faster. A change to a common port or variable reflects everywhere, saving time and avoiding oversights. In teams, this consistency means fewer configuration bugs and a smoother onboarding for anyone reading/editing Compose for the first time.<br>
 </p></li>
<li><p><strong>Depends_on = Ordered startup and fewer failures:</strong><br><br>
With well‑defined dependencies—and using healthcheck conditions when applicable—the process of starting the environment becomes <strong>much more reliable</strong>. Developers don’t need to take manual steps to ensure “the database is up” before running the application, for example. In CI pipelines and local demos, everything comes up in the right order automatically. This reduces time lost with trivial troubleshooting like “oh, never mind, you just had to start service X first...” and keeps the focus on the application logic.</p></li>
</ul>

<p>In short, these advanced Docker Compose features act as small <em>productivity multipliers</em> for teams: less time spent tweaking configuration and more time on the application itself.<br>
 <br>
 </p>

<h2>
  
  
  Final tip
</h2>

<ul>
<li>
<strong>Keep your Docker Compose up to date:</strong> Make sure you’re using a recent version to access new features, bug fixes and, most importantly, to maintain compatibility with the Docker engine and avoid runtime issues!</li>
</ul>

