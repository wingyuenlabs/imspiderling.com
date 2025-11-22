---
Title: 🚀 Integrating API Gateway with Private ALB: The New, Simpler, and More Scalable Way
Description: 
Author: Afu Tse (Chainiz)
Date: 2025-11-22T21:46:52.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  🧩 The Problem
</h2>

<p>When you work with microservices in AWS (especially in ECS, EKS, or internal applications inside a VPC), sooner or later you need to expose a REST endpoint through <strong>Amazon API Gateway</strong>, but without making your backend public.</p>

<p>For many years, the only “official” way to integrate <strong>API Gateway (REST)</strong> with a private <strong>Application Load Balancer (ALB)</strong> was by placing a <strong>Network Load Balancer (NLB)</strong> in the middle.</p>

<p>This created three common issues in real-world projects:</p>

<ol>
<li>More infrastructure than necessary (ALB + NLB just to create a bridge).</li>
<li>Higher latency because traffic needed to make an extra hop.</li>
<li>More cost and operational overhead: two load balancers to monitor, scale, and secure.</li>
</ol>

<p>For students or small teams, this architecture was confusing and far from intuitive:</p>

<blockquote>
<p>“Why do I need an NLB if my backend is already behind an ALB?”</p>
</blockquote>

<p>And yes… we were all asking ourselves the same thing.</p>




<h2>
  
  
  🔧 The Old Solution:
</h2>

<p>Until recently, the flow looked like this:</p>

<p><strong><em>API Gateway → VPC Link → NLB → ALB (Privado)</em></strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi834308o0ve6mlxzhrwz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi834308o0ve6mlxzhrwz.png" alt="old-solution" width="600" height="138"></a></p>

<p>The NLB acted as a “bridge” because API Gateway could only connect to an NLB using <strong>VPC Link</strong>. ALB wasn’t supported directly.</p>

<p>This worked, but:</p>

<ul>
<li>It was more complex to explain in classes or onboarding sessions.</li>
<li>Costs were higher (NLB hourly charges + NLCUs).</li>
<li>It introduced extra points of failure.</li>
<li>It didn’t feel natural for modern ALB-based architectures.</li>
</ul>




<h2>
  
  
  ⭐ The New Solution:
</h2>

<p>AWS finally heard our prayers 🙏.</p>

<p>Now API Gateway (REST) supports direct private integration with ALB using <strong>VPC Link v2</strong>.<br>
The new flow looks like this:</p>

<p><strong><em>API Gateway → VPC Link v2 → ALB (Privado)</em></strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8mz7qu1gfmxtvo8ocwbe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8mz7qu1gfmxtvo8ocwbe.png" alt="new-solution" width="599" height="110"></a></p>

<p>In summary:</p>

<ul>
<li>No NLB.</li>
<li>No unnecessary bridge.</li>
<li>Lower cost.</li>
<li>Lower latency.</li>
<li>Easier to teach and understand.</li>
</ul>

<p>This allows you to naturally expose your internal microservices behind a private ALB without adding any extra resources.</p>




<h2>
  
  
  ⚔️ Comparison: Before vs Now
</h2>

<div class="table-wrapper-paragraph"><table>
  <thead>
    <tr>
      <th>Aspect</th>
      <th>Old (NLB Required)</th>
      <th>New (Direct to ALB)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Infrastructure</td>
      <td>More complex (extra NLB)</td>
      <td>Much simpler</td>
    </tr>
    <tr>
      <td>Cost</td>
      <td>Hourly NLB + extra NLCUs</td>
      <td>Only ALB + API Gateway</td>
    </tr>
    <tr>
      <td>Latency</td>
      <td>Higher (extra hop through NLB)</td>
      <td>Lower</td>
    </tr>
    <tr>
      <td>Maintenance</td>
      <td>Two load balancers</td>
      <td>One load balancer</td>
    </tr>
    <tr>
      <td>Security</td>
      <td>Good, but more SG rules</td>
      <td>Equally secure, fewer failure points</td>
    </tr>
    <tr>
      <td>Clarity</td>
      <td>Hard to explain</td>
      <td>Much more intuitive</td>
    </tr>
    <tr>
      <td>Scalability</td>
      <td>Depends on NLB</td>
      <td>Highly scalable VPC Link v2</td>
    </tr>
    <tr>
      <td>Flexibility</td>
      <td>Limited to NLB</td>
      <td>Supports multiple ALBs/NLBs</td>
    </tr>
  </tbody>
</table></div>




<h2>
  
  
  🎓 Why is this change important?
</h2>

<p>If you’re learning cloud architecture — especially microservices — this change is a huge benefit:</p>

<ul>
<li>The architecture is easier to understand.</li>
<li>You can focus on real concepts, not historical workarounds.</li>
<li>Lower cost for student projects or test environments.</li>
<li>The traffic path becomes cleaner and more direct.</li>
<li>You can use ALB (which is more flexible) without depending on an NLB.</li>
</ul>

<p>It also unlocks modern patterns:</p>

<ul>
<li>Expose EKS/ECS microservices via API Gateway without making them public.</li>
<li>Build more secure internal corporate APIs.</li>
<li>Use a single VPC Link v2 to route to multiple ALBs.</li>
</ul>




<h2>
  
  
  🎯 Conclusions
</h2>

<p>AWS simplified a pattern that had been unnecessarily complex for years. The direct integration between API Gateway and private ALB:</p>

<ul>
<li>Lowers cost</li>
<li>Reduces complexity</li>
<li>Reduces latency</li>
<li>Improves architectural clarity</li>
<li>Is perfect for students, small teams, and enterprises</li>
</ul>

<p>If you were building internal APIs and previously needed an NLB just to bridge API Gateway to ALB… you can forget about that now.</p>

<p>The architecture is cleaner, more modern, and aligned with real cloud-native best practices.</p>




<h2>
  
  
  📚 References
</h2>

<p>Here are the official sources and recommended materials for deeper study:</p>

<ul>
<li>AWS Official Blog:
Build scalable REST APIs using Amazon API Gateway private integration with Application Load Balancer
<a href="https://aws.amazon.com/blogs/compute/build-scalable-rest-apis-using-amazon-api-gateway-private-integration-with-application-load-balancer/" rel="noopener noreferrer">https://aws.amazon.com/blogs/compute/build-scalable-rest-apis-using-amazon-api-gateway-private-integration-with-application-load-balancer/</a>
</li>
<li>API Gateway – VPC Links Documentation:
<a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-vpc-links.html" rel="noopener noreferrer">https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-vpc-links.html</a>
</li>
<li>Application Load Balancer Documentation:
<a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html" rel="noopener noreferrer">https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html</a>
</li>
<li>API Gateway REST APIs Documentation:
<a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-rest-api.html" rel="noopener noreferrer">https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-rest-api.html</a>
</li>
<li>AWS Well-Architected Best Practices:
<a href="https://aws.amazon.com/architecture/well-architected/" rel="noopener noreferrer">https://aws.amazon.com/architecture/well-architected/</a>
</li>
</ul>




<h2>
  
  
  🌐 Languages
</h2>

<p><a href="https://dev.to/chainiz/integrando-api-gateway-con-alb-privado-la-nueva-forma-mas-simple-y-escalable-370k">Spanish</a></p>

