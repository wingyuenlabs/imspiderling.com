---
Title: Reusing CloudFront, ALB, and API Gateway in a Serverless Platform
Description: 
Author: Sebastian Mincewicz
Date: 2026-03-17T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Modern serverless platforms often need to support a mix of public-facing entry points - web applications and APIs - while keeping internal communication private and tightly controlled. Non-functional requirements around security, compliance, and operational isolation typically drive this.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxanvn5a04iewc7tzl6fz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxanvn5a04iewc7tzl6fz.png" alt="Post Banner" width="800" height="474"></a></p>

<p>At the same time, such platforms are increasingly built around multiple domain services, each representing a bounded context and owned by a small team. Independent deployments, limited blast radius, and clear ownership are usually explicit architectural goals.</p>

<p>These goals, however, often collide with another equally important requirement: speed. In fast-paced development environments, the ability to spin up feature environments on demand, test changes quickly, and tear everything down without friction is critical to maintaining delivery velocity.</p>

<p>This post explores how architectural choices at the edge - specifically around CloudFront, Application Load Balancers (ALB), and API Gateway - can either enable or severely constrain that speed. It looks at how careful reuse of these components can support isolated, per-service deployments and on-demand environments without driving up cost or operational complexity, and where the trade-offs start to appear.</p>

<h2>
  
  
  Architectural choices
</h2>

<h3>
  
  
  Frontend
</h3>

<p>In many serverless platforms, the frontend acts as the primary public entry point. A common pattern is to place CloudFront at the edge and route requests to different backend origins depending on their nature.</p>

<p>Static assets are typically served from S3, while dynamic requests are forwarded to compute workloads that render pages or handle frontend-driven API calls. These dynamic workloads may run as Lambda functions or containers, often inside a VPC to meet security and compliance requirements.</p>

<p>At this point, there is an architectural choice to make: whether dynamic frontend traffic should be handled by the <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-the-right-aws-service-for-your-microservice-endpoints/services-comparison.html" rel="noopener noreferrer">API Gateway or an Application Load Balancer (ALB)</a>.</p>

<p>From a non-functional requirements perspective, separating public access from private execution is key. While API Gateway is well-suited for exposing managed APIs, frontend traffic often benefits from being terminated at ALB, with requests forwarded to compute that remains fully private within a VPC. <strong>CloudFront supports private origins through VPC-origin integrations with internal ALB, whereas private API Gateways cannot be used directly as CloudFront origins</strong>. This makes ALB a more natural fit when frontend compute is intentionally kept off the public internet.</p>

<p>Placing <a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/application/lambda-functions.html" rel="noopener noreferrer">Lambda functions behind an ALB</a> and inside a VPC also addresses increasingly common security requirements in regulated environments. When compute is fronted by an internal ALB, CloudFront becomes the only public ingress path by design. The ALB is not internet-facing and cannot be accessed directly, which ensures that all external traffic is consistently terminated, inspected, and controlled at the edge.</p>

<p>This pattern significantly reduces the risk of accidental or intentional bypass of edge-level controls. Combined with VPC-based execution, it enables tighter governance of inbound and outbound traffic through WAF, security groups, routing, and egress filtering. Together, these measures provide stronger guarantees than relying solely on managed service boundaries or publicly reachable endpoints.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1pvx6i2nzflns66bs19w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1pvx6i2nzflns66bs19w.png" alt="CF-ALB-Frontend" width="800" height="551"></a></p>

<p>From an operational perspective, ALB offers lower latency for HTTP workloads, native support for both host-based and path-based routing, and seamless integration with both Lambda and container-based runtimes such as ECS. These characteristics make it particularly well-suited for frontend and backend-for-frontend (BFF) use cases, where routing flexibility, performance, and network controls often outweigh API-centric features such as request validation, usage plans, or request transformation.</p>

<p>This does not diminish the role of API Gateway in the overall architecture. API Gateway remains a strong choice for internal and external APIs, especially where API lifecycle management, authentication, throttling, and contract enforcement are primary concerns. Using ALB at the frontend edge simply allows each component to be applied where it fits best, rather than forcing a single service to satisfy fundamentally different requirements.</p>

<h3>
  
  
  Internal API
</h3>

<p>Internally, platforms often expose functionality through APIs consumed by other services or internal clients. A key architectural decision is whether to operate a single internal API Gateway or multiple gateways aligned with services, domains, or teams.</p>

<p>There is no universal answer. Smaller platforms or early-stage systems often benefit from a single internal API, as it simplifies discovery and reduces operational overhead. As platforms grow, multiple APIs aligned with domains or "two-pizza teams" can become more appropriate, particularly when ownership boundaries, release cadence, or non-functional requirements diverge.</p>

<p>What matters more than the number of gateways, however, is recognising that <strong>API shape and deployment shape are not the same thing</strong>. A single logical API does not require a single infrastructure deployment unit, just as multiple services do not automatically justify multiple gateways. Treating these concerns separately allows teams to optimise for independent releases, blast-radius reduction, and clearer ownership, while keeping the external API surface coherent.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Febepn857ewyrgoupjiwv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Febepn857ewyrgoupjiwv.png" alt="SingleVsMultipleAPIGs" width="800" height="740"></a></p>

<p>This distinction becomes especially important in high-paced environments, where the ability to evolve services independently should not be constrained by how APIs are presented or grouped at the edge.</p>

<h3>
  
  
  Backend services and infrastructure-as-code
</h3>

<p>From an infrastructure-as-code (IaC) perspective, backend services are often deployed as independent units to support isolation, targeted releases, and reduced blast radius. This aligns naturally with domain-driven design, where each service represents a bounded context and can evolve independently.</p>

<p>When multiple backend services are exposed through <strong>a single API Gateway</strong>, this model becomes more nuanced. API Gateway configuration changes - such as routes, integrations, or method settings - are only made effective through an explicit <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-deploy-api.html" rel="noopener noreferrer">deployment step</a>. As a result, while service-specific configuration can be defined independently, the API Gateway itself still has a shared deployment lifecycle.</p>

<p>This does not make per-service backend stacks incompatible with a single API Gateway, but it does require additional structure. In practice, this means decomposing API Gateway configuration into distinct concerns: a base or core API configuration, per-service or per-domain route and integration definitions, and a dedicated deployment component responsible for applying changes to the gateway. That deployment component must be notified when any service updates its portion of the API configuration, without introducing tight coupling between services or requiring knowledge of how many services exist.</p>

<p>Without this explicit separation, a shared API Gateway can easily become a point of implicit coupling, where independent service deployments are forced to coordinate around a central API deployment. With the right decomposition and signalling in place, however, a single logical API can still support independent service lifecycles and parallel provisioning.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxxdvnz4dealdxtlsbqq2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxxdvnz4dealdxtlsbqq2.png" alt="TFsplit" width="800" height="916"></a></p>

<p>In contrast, using an <strong>API Gateway per backend service</strong> naturally scopes configuration and deployment to the service itself, avoiding the need for cross-service coordination at the gateway level. The trade-off is increased operational overhead and a more fragmented API surface. The choice between these approaches is therefore not about feasibility, but about how much complexity is absorbed by infrastructure design versus operational management, and how much flexibility is required as the platform continues to evolve.</p>

<h2>
  
  
  On-demand feature environments
</h2>

<h3>
  
  
  Why they matter
</h3>

<p>Fast feedback is essential. Feature environments allow teams to validate changes in realistic conditions, unblock reviews, and catch integration issues early. While tools like <a href="https://www.localstack.cloud/" rel="noopener noreferrer">LocalStack</a> can help with local development, they have limitations - especially for integration testing, CI pipelines, and workflows involving multiple services.</p>

<p>On-demand environments bridge that gap, but naïvely provisioning full stacks per feature quickly becomes expensive and slow, too slow.</p>

<h3>
  
  
  Reuse as an enabler
</h3>

<p>Not all infrastructure needs to be duplicated. Components like CloudFront, ALB, and API Gateway are surprisingly well-suited to reuse, provided routing is designed with that goal in mind.</p>

<p>This is not just a cost consideration. Creating or modifying edge components such as CloudFront distributions or load balancers can easily take ten minutes or more, which makes them a poor fit for fast, iterative workflows. When feature environments depend on provisioning or reconfiguring these components, feedback loops slow down dramatically.</p>

<p>The key, therefore, is deciding what varies per environment and what stays shared. By reusing long-lived edge infrastructure and shifting environment-specific concerns into routing, configuration, and backend resources, platforms can keep costs predictable and provisioning times low. The trade-off is that this requires upfront discipline in routing design, naming conventions, and configuration boundaries, but that investment pays off quickly as the environment count and delivery pace increase.</p>

<h3>
  
  
  Ephemeral on-demand environments
</h3>

<p>Ephemeral environments, the golden grail, aim for maximum speed. They rely on shared edge infrastructure and differentiate environments through routing.</p>

<p>This approach typically requires:</p>

<ul>
<li>Shared CloudFront configuration with a common domain name</li>
<li>Path-based routing at ALB</li>
<li>Frontend support for runtime <code>basePath</code> and <code>assetPrefix</code> changes</li>
<li>Multiple API Gateway stages mapped to environment identifiers</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9c9hypc1z8yyq7fh6k09.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9c9hypc1z8yyq7fh6k09.png" alt="EphmeralEnvs" width="800" height="290"></a></p>

<p><em>Note: Internal API design is irrelevant here</em></p>

<p>When these conditions are met, spinning up a new environment becomes little more than adding routing rules and deploying service-specific resources. Cleanup is trivial, and costs remain largely constant regardless of the number of environments. Neither CloudFront nor the ALB needs to be aware of how many environments exist or may exist in the future.</p>

<h3>
  
  
  Semi-static on-demand environments
</h3>

<p>Some frontends (e.g., Next.js) require environment-specific configuration at build time and cannot easily adapt to runtime path changes. In these cases, a semi-static approach is often necessary.</p>

<p>Here, each environment may have:</p>

<ul>
<li>Shared CloudFront configuration with alternate domain names and static Route 53 records</li>
<li>Shared ALB with host-based routing</li>
<li>Any API Gateway configuration implementation</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgt35u9wu8jd44aaboi12.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgt35u9wu8jd44aaboi12.png" alt="SemiStaticEnvs" width="800" height="280"></a></p>

<p><em>Note: Internal API design is irrelevant here</em></p>

<p>In contrast to the ephemeral environments option, this model requires CloudFront and Route 53 to be prepared upfront. Each environment must be explicitly represented through distribution configuration and DNS records, making environments longer-lived and less dynamic by design.</p>

<h3>
  
  
  Practical limits and quotas
</h3>

<p>In practice, hard service quotas are rarely the first blocker. More often, provisioning time and operational friction become limiting factors as environments scale.</p>

<p>That said, quotas do influence how far reuse patterns can go and whether a limit can be raised or must be designed around matters. API Gateway REST APIs support a limited number of stages per API (10 by default, soft limit), which constrains stage-based environment reuse. Private API Gateway custom domain names are also subject to soft limits. On the load-balancing side, an ALB supports up to 100 listener rules (soft limit) and 100 target groups (hard limit), the latter requiring an architectural change.</p>

<p>For example, reusing a single ALB across ten environments can consume listener rules perhaps not so quickly, while a single API Gateway with one stage per environment approaches its stage limit fast. These constraints don't prevent reuse, but they highlight the need to understand early which limits can be adjusted and which require rethinking the design.</p>

<h2>
  
  
  Conclusion
</h2>

<p>The patterns described in this post are not a single fixed architecture, but a <strong>set of reuse-oriented edge and routing strategies</strong> for serverless platforms built on CloudFront, ALB, and API Gateway. Their value lies in being applied selectively.</p>

<p>For <strong>lower environments</strong> - feature, preview, and integration stages - reusing edge components and differentiating environments through routing enables fast provisioning, easy teardown, and predictable costs. These environments benefit most from shared CloudFront distributions, shared ALBs, and carefully structured API Gateway reuse, where speed and feedback outweigh the need for strict isolation.</p>

<p><strong>Upper environments are different</strong>. Staging, pre-production, and production typically require:</p>

<ul>
<li>Standalone, fully isolated infrastructure</li>
<li>Right-sized capacity and scaling characteristics</li>
<li>Stable DNS and edge configuration</li>
<li>The ability to run meaningful load, stress, and performance tests against an environment that truly behaves like production</li>
</ul>

<p>In those environments, reuse at the edge is usually inappropriate. Dedicated CloudFront distributions, ALBs, and API Gateways are not an optimisation failure - they are a requirement.</p>

<p>Supporting both approaches within the same platform does introduce additional complexity, particularly in infrastructure-as-code. Designing IaC that remains DRY while allowing some environments to share edge infrastructure and others to be fully isolated requires clear layering, strong conventions, and explicit ownership boundaries. This is more demanding than a uniform setup, but it is entirely achievable with deliberate design.</p>

<p>These patterns optimise for:</p>

<ul>
<li>Fast feedback and experimentation where environment churn is high</li>
<li>Independent service evolution with controlled blast radius</li>
<li>Predictable cost and provisioning time at scale</li>
<li>The ability to evolve toward stricter isolation when needed</li>
</ul>

<p>They are not optimised for:</p>

<ul>
<li>Maximum isolation across all environments</li>
<li>Treating edge infrastructure as immutable per environment</li>
<li>Eliminating all shared components</li>
</ul>

<p>Being explicit about where reuse applies - and where it must not - allows the platform to move quickly without compromising correctness. The goal is not to reuse everything, but to reuse the <em>right</em> things, in the <em>right</em> places, for the <em>right</em> reasons.</p>

