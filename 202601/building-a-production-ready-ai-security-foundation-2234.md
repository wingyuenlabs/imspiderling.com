---
Title: Building a Production-Ready AI Security Foundation
Description: 
Author: Aron Eidelman
Date: 2026-01-23T21:30:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>Scaling Generative AI applications from proof-of-concept to production is often bottlenecked by security concerns, specifically sensitive data exposure and prompt injection.</p>

<p>Establishing a production-ready posture requires a <strong>defense-in-depth strategy</strong> across three layers:</p>

<ul>
<li>
<strong>Application Layer:</strong> Real-time threat detection and mitigation.</li>
<li>
<strong>Data Layer:</strong> Enforcing privacy controls and compliance.</li>
<li>
<strong>Infrastructure:</strong> Network segmentation and compute isolation.</li>
</ul>

<p>To implement these controls, this guide details three hands-on labs focused on securing these specific architectural planes.</p>

<h2>
  
  
  Protect the Application in Real-Time: Model Armor
</h2>

<p>The application layer, where users directly interact with your AI model, is the <strong>most exposed surface</strong> in a GenAI application. This surface is frequently targeted by attackers using prompts and responses to exploit vulnerabilities.</p>

<p>This lab focuses on securing the application and model layers by demonstrating how to deploy a comprehensive security service called <strong><a href="https://docs.cloud.google.com/security-command-center/docs/model-armor-overview" rel="noopener noreferrer">Model Armor</a></strong>. Model Armor acts as an intelligent firewall, analyzing prompts and responses in real-time to detect and block threats before they can cause harm.</p>

<p>In this lab, you learn to mitigate critical risks, including:</p>

<ul>
<li>
<strong>Prompt injection &amp; jailbreaking:</strong> Malicious users crafting prompts to bypass safety guardrails or extract confidential data. You will create a Model Armor security policy that automatically detects and blocks these attempts.</li>
<li>
<strong>Malicious URL detection:</strong> Blocking users who embed dangerous links in prompts, which could be part of an indirect injection.</li>
<li>
<strong>Sensitive data leakage:</strong> Preventing the model from inadvertently exposing Personally Identifiable Information (PII) in its responses.</li>
</ul>

<p><strong>The Key Components:</strong></p>

<p>You will create reusable templates that define what Model Armor should analyze, detect, and block. The <code>block-unsafe-prompts</code> template targets malicious inputs, while the <code>data-loss-prevention</code> template prevents sensitive data from being exposed in prompts or responses.</p>

<p>After completing this lab, you will have the blueprint to integrate Model Armor directly into your application’s backend API, ensuring that every request to your model first passes through this real-time threat detection layer.</p>


<div class="crayons-card c-embed">

  <br>
<strong>Go to the lab!</strong><br>
<strong>Lab:</strong> <a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/4-securing-ai-applications/securing-ai-applications#0" rel="noopener noreferrer">Securing AI Applications</a>

<p><strong>Objective:</strong> <em>Learn to use Model Armor to secure Generative AI applications against prompt injection and data leakage.</em></p>


</div>


<h2>
  
  
  Safeguard AI Data with Sensitive Data Protection
</h2>

<p>While the application layer needs real-time defense, the data used for training and testing AI models requires protection before it even enters the development environment. Raw customer data poses significant privacy challenges, and developers need high-quality data that is safe and compliant.</p>

<p>This lab guides you through building an <strong>automated data sanitization pipeline</strong> to protect sensitive information used in AI development. You will use <a href="https://docs.cloud.google.com/sensitive-data-protection/docs/sensitive-data-protection-overview" rel="noopener noreferrer">Google Cloud’s Sensitive Data Protection (SDP)</a> to inspect, classify, and de-identify Personally Identifiable Information (PII) across various data formats.</p>

<p><strong>The Key Components:</strong></p>

<ul>
<li>
<strong>Inspection Templates:</strong> You define an inspection template to look for specific sensitive information types, or <strong>infoTypes</strong>, that are relevant to your data and geography, such as credit card numbers or SSNs.</li>
<li>
<strong>De-identification Templates:</strong> You build separate de-identification templates for different data formats, giving you granular control:

<ul>
<li>
<strong>Unstructured Data:</strong> Replacing sensitive values in text files (like chat logs) with their <code>infoType</code> name to preserve context.</li>
<li>
<strong>Structured Data:</strong> Using record transformations like <strong>character masking</strong> on CSV files to preserve data utility for testing while still de-identifying sensitive fields.</li>
<li>
<strong>Image Data:</strong> Leveraging optical character recognition (OCR) to detect and redact sensitive text embedded within images.</li>
</ul>


</li>

<li>Automated Jobs: You configure a single job that <strong>automatically applies the correct redaction</strong> based on the file type it detects and inspects, automating the security workflow for data stored in Cloud Storage.</li>

</ul>

<p>In a production environment, you would use these templates to create a fully automated, hands-off detection and de-identification process, often by setting up a <strong>job trigger</strong> whenever new raw customer data is uploaded. For sensitive data unique to your business, you can define <a href="https://cloud.google.com/sensitive-data-protection/docs/creating-custom-infotypes" rel="noopener noreferrer">custom infoTypes</a> within Sensitive Data Protection.</p>


<div class="crayons-card c-embed">

  <br>
<strong>Go to lab!</strong><br>
<strong>Lab:</strong> <a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/4-securing-ai-applications/securing-data-used-for-ai-applications#0" rel="noopener noreferrer">Securing Data Used for AI Applications</a>

<p><strong>Objective:</strong> Build an automated pipeline to inspect, classify, and de-identify PII for use in AI development using Sensitive Data Protection.<br>

</p>
</div>


<h2>
  
  
  Harden the AI Infrastructure Foundation
</h2>

<p>The final layer of defense is the underlying infrastructure that hosts your development, training, and deployment processes. A production-ready AI environment must be isolated, hardened, and protected from system tampering, privilege escalation, and accidental data exposure.</p>

<p>This lab focuses on mitigating common infrastructure threats by creating a multi-layered, secure foundation.</p>

<p><strong>The Key Components:</strong></p>

<ul>
<li>
<strong>Secure Network Foundation:</strong> You provision a secure <strong>Virtual Private Cloud (VPC)</strong> and subnet, configured with <strong>Private Google Access</strong> to ensure that compute resources can reach Google APIs over a private network, avoiding the public internet. You also deploy a <strong>Cloud NAT gateway</strong> to allow private instances to initiate controlled outbound connections without having a public IP.</li>
<li>
<strong>Hardened Compute:</strong> You deploy a secure <strong>Vertex AI Workbench instance</strong> inside your private VPC, which serves as your isolated development environment. You enforce the <strong>principle of least privilege</strong> by creating and assigning a dedicated service account with only the necessary roles. The instance itself is hardened by disabling root access and enabling security features like <strong>Secure Boot</strong>.</li>
<li>
<p><strong>Secure Storage:</strong> You create a fortified <strong>Cloud Storage bucket</strong> for your datasets, models, and artifacts. You apply strong configurations, including:</p>

<ul>
<li>
<strong>Enforce public access prevention</strong> to override any misconfigured IAM settings.</li>
<li>
<strong>Uniform bucket-level access</strong> for simpler, more predictable control.</li>
<li>
<strong>Object versioning</strong> and <strong>soft delete</strong> for recovery from accidental or malicious overwrites or deletions.</li>
<li>
<strong>Data access logs</strong> to provide a comprehensive and immutable audit trail.</li>
</ul>


</li>

</ul>

<p>For maximum security, this entire environment can be wrapped in a <a href="https://cloud.google.com/vpc-service-controls/docs/overview" rel="noopener noreferrer">VPC Service Controls</a> perimeter, which prevents data exfiltration by ensuring services can only be accessed by authorized resources within your private network perimeter.</p>


<div class="crayons-card c-embed">

  <br>
<strong>Go to the lab!</strong><br>
<strong>Lab:</strong> <a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/4-securing-ai-applications/securing-infrastructure-for-ai-applications#0" rel="noopener noreferrer">Securing Infrastructure for AI Applications</a>

<p><strong>Objective:</strong> Secure an AI development environment by implementing network isolation, hardened compute instances, and protected storage.<br>

</p>
</div>


<h2>
  
  
  Build Your Production-Ready AI Security Today
</h2>

<p><strong>Ready to move your AI project from prototype to a secure, production-grade application?</strong> Dive into the codelabs now to begin your journey across the application, data, and infrastructure layers:</p>

<ul>
<li><a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/4-securing-ai-applications/securing-ai-applications#0" rel="noopener noreferrer">Securing AI Applications
</a></li>
<li><a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/4-securing-ai-applications/securing-data-used-for-ai-applications#0" rel="noopener noreferrer">Securing Data Used for AI Applications</a></li>
<li><a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/4-securing-ai-applications/securing-infrastructure-for-ai-applications#0" rel="noopener noreferrer">Securing Infrastructure for AI Applications</a></li>
</ul>

<p>These labs are part of the <strong>Securing AI Applications</strong> module in our official <strong><a href="https://cloud.google.com/blog/topics/developers-practitioners/production-ready-ai-with-google-cloud-learning-path?e=48754805" rel="noopener noreferrer">Production-Ready AI with Google Cloud</a></strong> program. Explore the full curriculum for more content that will help you bridge the gap from a promising prototype to a production-grade AI application.</p>

<p>Share your progress and connect with others on the journey using the hashtag <strong>#ProductionReadyAI</strong>. Happy learning!</p>

