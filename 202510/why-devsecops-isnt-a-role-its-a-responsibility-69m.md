---
Title: Why DevSecOps Isn't a Role. It's a Responsibility
Description: 
Author: Anderson Leite
Date: 2025-10-29T21:36:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Remember when (at least part of) the industry realised "DevOps isn't a job title"? We're making the same mistake again, now with DevSecOps. Companies are hiring "DevSecOps Engineers," thinking they've solved security. </p>

<p>They've just renamed the problem.</p>

<h2>
  
  
  The Déjà Vu Moment
</h2>

<p>Some years ago, organisations started to rush into hire "DevOps Engineers" believing they could buy their way into better collaboration and faster delivery. Instead, they got siloed teams with new titles, operating much like the old separation between development and operations.</p>

<p>The 2016 <em>State of DevOps Report</em> by Puppet (<a href="https://dora.dev/research/2016/2016-state-of-devops-report.pdf" rel="noopener noreferrer">published by DORA.dev now</a>) found that high-performing organisations treated DevOps as a cultural practice, not a job description. Teams that successfully adopted DevOps had shared ownership across roles. The same principle applies to security.</p>

<h2>
  
  
  What DevSecOps Really Means
</h2>

<p>DevSecOps isn't about adding another specialist to your team. It's embedding security thinking throughout your software lifecycle:</p>

<ul>
<li>
<strong>Planning</strong>: Threat modelling during design reviews</li>
<li>
<strong>Development</strong>: Secure coding practices and peer review</li>
<li>
<strong>Build</strong>: Automated security scanning in CI/CD</li>
<li>
<strong>Deploy</strong>: Infrastructure-as-code validation and policy enforcement</li>
<li>
<strong>Operate</strong>: Runtime security monitoring and incident response</li>
</ul>

<p>When security becomes everyone's concern, it stops being an afterthought.</p>

<h2>
  
  
  The Trap of Security Silos
</h2>

<p>Here's what happens when you hire a "DevSecOps Engineer":</p>

<ol>
<li>The team feels relieved "security is handled now"</li>
<li>Developers stop thinking about security in daily work</li>
<li>The DevSecOps person becomes a bottleneck</li>
<li>Security issues pile up in a backlog</li>
<li>When breaches happen, there's finger-pointing</li>
</ol>

<p>This creates false ownership. Security becomes "someone else's problem" — just with a fancier title. One person can't review every line of code, attend every architecture discussion, or monitor every deployment. They become a gatekeeper rather than an enabler.</p>

<p>According to the <a href="https://dora.dev/research/2023/dora-report/2023-dora-accelerate-state-of-devops-report.pdf" rel="noopener noreferrer">2023 State of DevOps Report</a> by DORA (DevOps Research and Assessment), organisations with distributed security practices had 2.6x faster incident response times compared to those with centralised security teams.</p>

<h2>
  
  
  Shared Responsibility in Practice
</h2>

<p>What does shared security responsibility actually look like?</p>

<h3>
  
  
  CI/CD Pipeline Security
</h3>

<ul>
<li>
<strong>Developers</strong> write secure code and run pre-commit hooks</li>
<li>
<strong>Platform teams</strong> configure and maintain security scanning tools</li>
<li>
<strong>Security specialists</strong> set policies and review high-risk changes</li>
<li>
<strong>Everyone</strong> responds to vulnerability alerts in their domain</li>
</ul>

<h3>
  
  
  Infrastructure-as-Code Reviews
</h3>

<ul>
<li>
<strong>Infrastructure engineers</strong> implement secure-by-default configurations</li>
<li>
<strong>Developers</strong> review IaC changes that affect their applications</li>
<li>
<strong>Security teams</strong> provide baseline policies and compliance checks</li>
<li>
<strong>Managers</strong> ensure time is allocated for security work</li>
</ul>

<h3>
  
  
  Threat Modelling
</h3>

<ul>
<li>
<strong>Product managers</strong> identify high-value assets and attack surfaces</li>
<li>
<strong>Architects</strong> design systems with security constraints in mind</li>
<li>
<strong>Engineers</strong> implement controls and mitigations</li>
<li>
<strong>Security experts</strong> facilitate sessions and provide guidance</li>
</ul>

<p>Everyone contributes based on their expertise and context.</p>

<h2>
  
  
  Building a Security-First Culture
</h2>

<p>Cultural change doesn't happen by mandate. It requires:</p>

<p><strong>Psychological Safety</strong>: Teams need to report vulnerabilities without blame. Google's <a href="https://rework.withgoogle.com/intl/en/guides/understanding-team-effectiveness" rel="noopener noreferrer">Project Aristotle research</a> identified psychological safety as the foundation of high-performing teams. The same applies to security: a blameless postmortem culture focused on systems, not individuals, encourages transparency.</p>

<p><strong>Tooling That Enables</strong>: Security tools should help developers ship faster, not slow them down. Fast feedback loops: Catching issues in the IDE or at commit time for example helps to reduce friction. The shift-left movement, documented in the <a href="https://www.bsimm.com/" rel="noopener noreferrer">Building Security In Maturity Model (BSIMM)</a>, shows that earlier detection dramatically reduces remediation costs.</p>

<p><strong>Incentives That Align</strong>: If teams are measured only on feature velocity, security loses. Include security metrics in team goals: vulnerability remediation time, security test coverage, or compliance adherence.</p>

<p><strong>Knowledge Sharing</strong>: Security champions programs, internal training sessions, or capture-the-flag exercises democratize security knowledge. The <a href="https://owasp.org/www-project-security-champions-guidebook/" rel="noopener noreferrer">OWASP Security Champions Guide</a> provides a framework for building these programs.</p>

<p><strong>Investment in Training</strong>: Allocate budget for security training across all roles, not just security staff.</p>

<h2>
  
  
  Measuring DevSecOps Maturity
</h2>

<p>How do you know your approach is working?</p>

<p><strong>Process Metrics</strong>:</p>

<ul>
<li>Time from vulnerability discovery to remediation</li>
<li>Percentage of security issues caught before production</li>
<li>Number of security-related incidents</li>
<li>Compliance audit findings trends</li>
</ul>

<p><strong>Cultural Metrics</strong>:</p>

<ul>
<li>Cross-functional participation in security reviews</li>
<li>Security improvements suggested by non-security staff</li>
<li>Team confidence scores around security practices</li>
</ul>

<p><strong>Operational Metrics</strong>:</p>

<ul>
<li>Mean time to detect (MTTD) security issues</li>
<li>Mean time to respond (MTTR) to incidents</li>
<li>Percentage of automated vs. manual security checks</li>
</ul>

<p>Avoid vanity metrics like "number of DevSecOps engineers hired" or "security tools purchased." These measure inputs, not outcomes, outcome-based metrics drive real improvement.</p>

<h2>
  
  
  Making It Real: A Roadmap
</h2>

<p>Starting this journey? Here's a pragmatic approach:</p>

<p><strong>Month 1-2: Assess and Educate</strong></p>

<ul>
<li>Survey current security practices</li>
<li>Identify gaps and quick wins</li>
<li>Run awareness sessions across teams (maybe even start with this one!)</li>
</ul>

<p><strong>Month 3-4: Enable and Automate</strong></p>

<ul>
<li>Implement security scanning in CI/CD</li>
<li>Establish baseline policies for IaC and containers</li>
<li>Create shared documentation and runbooks</li>
</ul>

<p><strong>Month 5-6: Embed and Iterate</strong></p>

<ul>
<li>Integrate security into sprint planning</li>
<li>Start regular cross-functional security reviews</li>
<li>Gather feedback and adjust processes</li>
</ul>

<p><strong>Ongoing: Measure and Improve</strong></p>

<ul>
<li>Review metrics quarterly</li>
<li>Celebrate security wins publicly</li>
<li>Evolve practices as threats change</li>
</ul>

<h2>
  
  
  The Role of Security Specialists
</h2>

<p>This doesn't mean security expertise doesn't matter. Specialised security professionals are crucial — but their role shifts from gatekeepers to enablers:</p>

<ul>
<li>Build platforms and tools that make secure practices easy</li>
<li>Provide expertise for complex threat modelling</li>
<li>Stay ahead of emerging threats and vulnerabilities</li>
<li>Coach teams and build security capabilities</li>
<li>Handle security incidents and forensics</li>
</ul>

<p>They're force multipliers, not single points of failure.</p>




<p><strong>Security isn't a department: It's a habit.</strong> DevSecOps succeeds when everyone owns it.</p>

<p>Stop looking for the perfect DevSecOps hire. Start building a culture where security is everyone's responsibility, supported by the right expertise, tools, and incentives. That's when security truly shifts left — not just in your pipeline, but in your thinking.</p>

<p>The question isn't "Who should we hire to do DevSecOps?" It's "How do we make our entire team security-conscious?" Answer that, and you'll build something far more resilient than any single role could provide.</p>




<p><em>Further Reading:</em></p>

<ul>
<li><a href="https://csrc.nist.gov/projects/devsecops/resources" rel="noopener noreferrer">NIST NCCoE Software Supply Chain and DevOps Security Practices. </a></li>
<li>
<a href="https://itrevolution.com/articles/the-three-ways-principles-underpinning-devops/" rel="noopener noreferrer">The Three Ways: The Principles Underpinning DevOps</a> by Gene Kim</li>
<li><a href="https://ruggedsoftware.org/" rel="noopener noreferrer">Rugged DevOps Manifesto</a></li>
</ul>

