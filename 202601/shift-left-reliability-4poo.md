---
Title: Shift-Left Reliability
Description: 
Author: Rob Fox
Date: 2026-01-12T21:34:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>We've become exceptionally good at incident response. Modern teams restore service quickly, run thoughtful postmortems, and hold themselves accountable through corrective actions.</p>

<p>And yet…</p>

<p>A team ships a change that passes every test, gets all the required approvals, and still brings down checkout for 47 minutes. The postmortem conclusion? "We should have known our latency SLO was already at 94% before deploying."</p>

<p>Many postmortems point to the same root cause: changes we introduced ourselves. Not hardware failures. Not random outages. Just software behaving exactly as we told it to.</p>

<p>We continue to treat reliability as something to evaluate once those changes are already live. This isn't a failure of tooling or process. It's a question of <em>when</em> we decide whether a system is ready.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjk0ji4myj0ridnenw5kk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjk0ji4myj0ridnenw5kk.png" alt="Shift-Left Reliability" width="800" height="398"></a></p>

<h2>
  
  
  The paradox
</h2>

<p>We've invested heavily in observing and responding to failure - better alerting, faster incident response, thorough postmortems. Teams care deeply about reliability and spend significant time optimizing how they respond to incidents.</p>

<p>But when in a service's lifecycle are they supposed to <em>define</em> reliability? Where's the innovation that happens <em>before</em> deployment?</p>

<h2>
  
  
  Where reliability decisions actually happen today
</h2>

<p>I've seen multiple teams running identical technology stacks with completely different SLOs, metrics, and alerts. Nobody told them what to implement, what's best-practice or how to tune their alerts. They want to be good reliability citizens, but getting from the theory in the handbook to putting that theory into practice is not straightforward.</p>

<p>Services regularly move into production with SLOs being created months later - or never. Dashboards are missing, insufficient, or inconsistent. "Looks fine to me" during PR reviews. Tribal knowledge. Varying levels of understanding across teams.</p>

<p>Reliability is fundamentally bespoke and ungoverned. That's the core issue.</p>

<h2>
  
  
  The missing layer
</h2>

<p>GitHub gave us version control for code. Terraform gave us version control for infrastructure. Security has transformed with shift-left - finding flaws as code is written, not after deployment.</p>

<p>We're still missing version control for reliability.</p>

<p>We need a specification that defines requirements, validates them against reality, and generates the artifacts: dashboards, SLOs, alerts, escalation policies. If the specification is validated and the artifacts created, the same tool can check in real-time whether a service is in breach - and block high-risk deployments in CI/CD.</p>

<h2>
  
  
  What shift-left reliability actually means
</h2>

<p>Shift-left reliability doesn't mean more alerts and dashboards, more postmortems or more people in the room.<br>
It means:</p>

<ol>
<li>Spec - Define reliability requirements as code before production deployment</li>
<li>Validate - Test those requirements against reality</li>
<li>Enforce - Gate deployments through CI/CD</li>
</ol>

<p>Engineers don't write PromQL or Grafana JSON - they declare intent, <br>
and reliability becomes deterministic. Outcomes are predictable, <br>
consistent, transparent, and follow best practice.</p>
<h2>
  
  
  An executable reliability contract
</h2>

<p>Keep it simple. A team creates a <em>service.yaml</em> file with their reliability intent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>name: payment-api
tier: critical
type: api
team: payments
dependencies:
 - postgresql
 - redis
</code></pre>

</div>



<p>Here is a complete <em>service.yaml</em> <a href="https://rsionnach.github.io/nthlayer/getting-started/first-service/#complete-service-example" rel="noopener noreferrer">example</a>.</p>

<p>Tooling validates metrics, SLOs, and error budgets then generates these artifacts automatically. This is the approach I am exploring with an open-source project called NthLayer.</p>

<p>NthLayer runs in any CI/CD pipeline - GitHub Actions, ArgoCD, Jenkins, Tekton, GitLab CI. The goal isn't to be an inflexible blocker; it's visible risk and explicit decisions. Overrides are fine when they're intentional, logged, and owned.</p>

<p>When a deployment is attempted, the specification is evaluated against reality:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$ nthlayer check-deploy - service payment-api
 ERROR: Deployment blocked
 - availability SLO at 99.2% (target: 99.95%)
 - error budget exhausted: -47 minutes remaining
 - 3 P1 incidents in last 7 days

Exit code: 2 (BLOCKED)
</code></pre>

</div>



<h2>
  
  
  Why now?
</h2>

<p>SLOs have had 8+ years to mature and move from the Google SRE Handbook into mainstream practice. GitOps has normalized declarative configuration. Platform Engineering has matured as a discipline. The concepts are ready but the tooling has lagged behind.</p>

<p>This is a deliberate shift in approach. Reliability is no longer up for debate during incidents. Services have defined owners with deterministic standards. We can stop reinventing the reliability wheel every time a new service is onboarded. If requirements change, update the <em>service.yaml</em>, run NthLayer and every service benefits from adopting the new standard.</p>

<h2>
  
  
  What this does not replace
</h2>

<p>NthLayer doesn't replace service catalogs, developer portals, observability platforms, or incident management. It doesn't predict failures or eliminate human judgment. It's upstream of all these systems.</p>

<p>The goal: a reliability specification, automated deployment gates and to reduce cognitive load to implement best practices.</p>

<h2>
  
  
  Open questions
</h2>

<p>I don't have all the answers but two questions I keep returning to are:</p>

<p><strong>Contract Drift:</strong> What happens when the spec says 99.95% but reality has been 99.5% for months? Is the contract wrong, or is the service broken?<br>
<strong>Emergency Overrides:</strong> How should they work? Who approves? How do you prevent them from becoming the default?</p>
<h2>
  
  
  The timing problem
</h2>

<p>Where do reliability decisions actually happen in your organization? What would it look like to decide readiness before deployment? What reliability rules do you wish you could enforce automatically?</p>

<p>The timing problem isn't going away. The only question is whether you address it before deployment - or learn about it in the postmortem.</p>



<p>NthLayer is open source and looking for early adopters. If you're tired of reliability being an afterthought:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>pip install nthlayer
nthlayer init
nthlayer check-deploy --service your-service
</code></pre>

</div>



<p>→ <a href="https://github.com/rsionnach/nthlayer" rel="noopener noreferrer">github.com/rsionnach/nthlayer</a></p>

<p>Star the repo, open an issue, or tell me I'm wrong. I want to hear how reliability decisions happen in your organization.</p>




<p><em>Rob Fox is a Senior Site Reliability Engineer focused on platform and reliability tooling. He's exploring how reliability engineering can move earlier in the software delivery lifecycle. Find him on <a href="https://github.com/rsionnach/nthlayer" rel="noopener noreferrer">GitHub</a>.</em></p>

