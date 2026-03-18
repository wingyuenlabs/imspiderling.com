---
Title: The Founding Engineer’s Checklist: Every Essential System for a Battle-Tested Launch
Description: 
Author: Francisco Luna
Date: 2026-03-18T21:55:41.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've developed products for startups. Most of them have failed and not because of the code. The backend was functional, queries were fast and the infrastructure could handle actual users, even if the traffic was small.</p>

<p>Why did them fail? Lack of PMF, scope creep and tedious operation management that made the 24/7 IT support guy for months.  </p>

<p>The real world of startup software development is not as easy as deploying your app and asking users to come; you also need to be able to add new features and see how users interact with your app. </p>

<p>I had to learn these things with downtime and scope creep, but they will ensure you have a battle tested launch.</p>

<p><strong>Note: This blog goes beyond rate limiting and using environment variables, often times, real world services are not mentioned in coding bootcamps or university.</strong></p>




<h2>
  
  
  🔴 Critical - Infrastructure and Deployment
</h2>

<ol>
<li><p>Provisioning: Ensure you have your infrastructure documented, whether in <a href="https://developer.hashicorp.com/terraform" rel="noopener noreferrer">Terraform</a>, <a href="https://www.pulumi.com/" rel="noopener noreferrer">Pulumi</a> or simple documentation if your product doesn't have too many moving parts yet. Your infrastructure and setup needs to be replicable.</p></li>
<li><p>CI/CD Pipelines: Make sure you have deployment scripts or actions to deploy your system and application. Always double check the secrets, permissions and environments available in your setup. </p></li>
<li><p>Secrets Management: Annotate all your application secret templates in .env.example files, secure them in your cloud provider and make sure to rotate the credentials accordingly. </p></li>
<li><p>Testing: For God's sake, don't skip testing even for MVP. Prioritize testing inputs, happy paths and use cases. I recommend using Playwright to test the entire application workflow and integration tests to test your use cases.</p></li>
</ol>

<h2>
  
  
  🔴 Critical - Data Layer Reliability
</h2>

<ol>
<li><p>Database Migrations: My least favorite one. Make sure you have a clear way to do this. You can use your ORM built in migration tools, custom scripts, CI/CD pipelines or even manual deployments. </p></li>
<li><p>Backups &amp; Recovery: Create backups of your database at least daily if you're using a managed database. Back up your server in case you're using a VPS.</p></li>
<li><p>Caching Strategy: Make sure your Redis setup and materialized views work as expcected and have clear policies.</p></li>
</ol>

<p><strong>Remember this small checklist before running schema migrations:</strong></p>

<ul>
<li>A new field in a table should never be <code>NOT NULL</code>. It should always have a default value or be nullable.</li>
<li>Don't take normalization too seriously. Sometimes it's better to use JSONB over complicating a new product with new tables.</li>
<li>Your new schema should be compatible with previous features. Whatever trade-offs you made, will have consequences later. </li>
<li>Don't trust your ORM too much. Always double-check the SQL code yourself.</li>
<li>Apply the principle of least privilege to everything. This means if your user needs to read the database, give them read-only access.</li>
</ul>

<p><em>(I'm also building a tool to make this process less painful in PostgreSQL! - It'll be revealed soon.)</em></p>

<h2>
  
  
  🔴 Critical - Identity and Security
</h2>

<ol>
<li><p>JWT tokens: It's up to you if you use JWT with a long TTL or refresh tokens. What matters is that you save the JWT secret in an environment variable and you have a disaster recovery strategy if it gets exposed.</p></li>
<li><p>Auth: Don't reinvent the wheel. Keep it simple and don't implement more services than needed whether you use custom auth or a managed service. If you use a manage service, keep the data in your own database.</p></li>
<li><p>RBAC &amp; ABAC: Have your roles and policies documented. Create custom policies and checks to prevent repeating if statemements across your app.</p></li>
<li><p>SSL &amp; Proper Configuration: Make sure your domains and infrastructure run over HTTPS, you have your robots.txt files, metadata, icons, SEO, app redirections and external services. </p></li>
<li><p>Transactional Messaging: Set up a service such as <a href="https://resend.com/" rel="noopener noreferrer">Resend</a> or <a href="https://aws.amazon.com/ses/" rel="noopener noreferrer">AWS SES</a> and configure the SPF and DKIM records in your DNS. Use proper email domains and avoid spamming your users. To create templates you can use <a href="https://mjml.io/" rel="noopener noreferrer">MJML</a>, as it allows you to create professional and responsive emails.</p></li>
</ol>

<p><strong>Tips:</strong></p>

<ul>
<li>If you're using images or fonts in your transactional emails, make sure they belong in your domain (e.g., images.mydomain.com/logo).</li>
<li>Keep your emails as simple as possible and make sure your links work as expected. </li>
</ul>

<h2>
  
  
  🟡 Important - Observability
</h2>

<ol>
<li><p>Frontend Analytics: Don't let your marketing team go blind. You can use <a href="https://umami.is/" rel="noopener noreferrer">Umami</a> for this, but other tools may work as well. </p></li>
<li><p>Frontend error handling: You need to see if your application is not working well for other users. I recommend using <a href="https://sentry.io/" rel="noopener noreferrer">Sentry</a> to monitor your frontend errors.</p></li>
<li><p>Logging: You need to use structured logging in services such as CloudWatch or BetterStack. Even a well structured Pino in a Docker container is valid if you're in the MVP stage. One important note, never display passwords or credentials in your loggers. Stick to non-sensitive fields like the user ID.</p></li>
<li><p>Health Checks and Alerts: Keep it simple for MVP. Use a service like <a href="https://betterstack.com/" rel="noopener noreferrer">BetterStack</a> or your cloud provider to ping your <code>/health</code> endpoint every 2 minutes or thereabouts. This way you'll receive an email when one of your services is down. </p></li>
<li><p>Server Monitoring and Observability: This step is not often needed if your system is small. You can implement <a href="https://prometheus.io/" rel="noopener noreferrer">Prometheus</a> and <a href="https://grafana.com/" rel="noopener noreferrer">Grafana</a> to monitor your infrastructure state (RAM usage, clusters metrics), although most cloud services have their built in monitors and metrics. </p></li>
</ol>

<p><strong>Tips:</strong></p>

<ul>
<li>Make sure to ping the database and other critical services such as Redis when calling your <code>/health</code> endpoint. </li>
<li>Prometheus and Grafana are powerful but high-maintenance. Stick to built-in cloud metrics for your MVP to keep your focus on the product, not the dashboard.</li>
</ul>

<h2>
  
  
  🟢 Good to Have - Operations
</h2>

<ol>
<li><p>Internal modules: Make the admin modules and initial app configuration seamless to use. Remember marketing, product and your internal team also need to use the application. Give the other departments what they need so you're not the IT support guy all day :)</p></li>
<li><p>Setup should be easy: You should not be using manual SQL and scripts every single time you deploy the system. Make the first app setup to initialize the platform intuitive, even if it's for yourself only. Don't develop hate towards your own system.</p></li>
<li><p>Architecture and scope matters: I once had to build a product solo with no guidance. I built poor architecture and I took every single request from the client. And of course, it collapsed under its own weight... </p></li>
</ol>

<p><strong>Tips:</strong></p>

<ul>
<li>Learn software architecture (how to create code that can be tested and maintained in the future)</li>
<li>Internal tooling is as important as customer facing features in most serious apps. Yes, your CEO and the marketing team are also your clients.</li>
<li>Get the user count and basic metrics for the team please. This will save you hours of IT support.</li>
<li>Learn about negotiation (how to set boundaries, build a MVP with proper scope and educate non-technical people about product development).</li>
</ul>

<h2>
  
  
  🟢 Good to have - Documentation
</h2>

<ol>
<li><p>OpenAPI documentation: <a href="https://swagger.io/specification/" rel="noopener noreferrer">OpenAPI 3.0</a> (OAS 3.0) is a specification you can use to document your API. You don't need it, but it makes collaboration and scalability easier. If you're building a serious product that will be maintained over time, consider adding it.</p></li>
<li><p>Onboarding docs: Document the current state of the system and what needs to be done, feature flags, how to run things locally, important trade-offs made, the architecture and tech stack of the system and alongside the most critical technical debt to pay.</p></li>
<li><p>Scalability plans: They don't need to be AWS architecture diagrams yet; a Notion page is enough. But you need to identify the current bottlenecks in your system and have a plan to scale.</p></li>
</ol>

<p><strong>Tips:</strong></p>

<ul>
<li>Make sure your API has consistent response structures, examples and clear body data.</li>
<li>Even if you have plans to work solo, the documentation is useful for your clients and your future self.</li>
<li>Most bottlenecks to scale will show up in the server and application layer. For this, you'll need to use load balancers, consider serverless architecture or use scalability groups.
</li>
<li>Node.js is good, but sometimes Golang can be better for specific services. Understand your technologies well.</li>
</ul>




<p>You can learn more about my work on my website, where I help companies build their MVPs and Backend Applications: <a href="https://www.itsfranciscoluna.com/" rel="noopener noreferrer">https://www.itsfranciscoluna.com/</a></p>

<p>Thanks for reading! Leave a comment if you want to share other tips you know for a successful MVP launch :)</p>

