---
Title: Sentinel CLI: A Self-Healing AWS Monitoring Agent Powered by GitHub Copilot
Description: 
Author: Mahesh
Date: 2026-02-08T20:53:50.000Z
Robots: noindex,nofollow
Template: index
---
<p>What I Built<br>
I built Sentinel CLI, a Python-based intelligent agent designed to bridge the gap between cloud infrastructure monitoring and automated remediation. It uses boto3 to scan the us-east-1 region for stopped EC2 instances and, rather than just alerting, it utilizes the GitHub Copilot CLI to reason and suggest the exact CLI commands needed to restore the service.</p>

<p>Demo<br>
GitHub Repository: <a href="https://github.com/mpawar006/sentinel-cli" rel="noopener noreferrer">https://github.com/mpawar006/sentinel-cli</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8jcit5puib4mnnxl4hmo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8jcit5puib4mnnxl4hmo.png" alt=" " width="800" height="448"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxq1l8ebjl0iqelpr1j8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsxq1l8ebjl0iqelpr1j8.png" alt=" " width="800" height="449"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsrynn6entpsbtbhs2bkc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsrynn6entpsbtbhs2bkc.png" alt=" " width="800" height="423"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvlf2zq4o7gowdb1c8wwm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvlf2zq4o7gowdb1c8wwm.png" alt=" " width="800" height="449"></a></p>

<p>My Experience with GitHub Copilot CLI<br>
Building this project on my ThinkPad T490 highlighted the power of "Agentic DevOps."</p>

<ul>
<li><p>Intelligent Reasoning: I integrated my Python script directly with the Copilot CLI using subprocess. This allowed the script to pass context to the AI and receive sophisticated, production-ready AWS commands in return.</p></li>
<li><p>Problem Solving: During development, Copilot helped me resolve Windows-specific UTF-8 encoding issues so that my terminal alerts (🚨) and formatting looked professional and clear.</p></li>
<li><p>Human-in-the-Loop: The CLI's interactive nature allowed me to build a secure approval flow, ensuring that an architect always verifies a "healing" command before execution.</p></li>
</ul>

