---
Title: Building an Open Source, Agent Friendly Mobile Engagement Tool
Description: 
Author: Jace Yoo
Date: 2026-03-09T22:01:42.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the past few weeks I have been spending a lot of time talking with mobile app makers. Most of them are small teams, sometimes just one or two developers. Despite the team size, many of these apps have users from all over the world.</p>

<p>One thing became clear very quickly during those conversations.</p>

<p>Small teams usually do not have great tools for user engagement.</p>

<p>Large companies often rely on complex systems like Braze or similar platforms. Those tools can be powerful, but they are often too heavy for small teams. They require a lot of setup, dedicated dashboards, and a marketing workflow that many product focused teams simply do not have.</p>

<p>Because of this, many developers end up doing one of the following.</p>

<p>They send a few manual push notifications.<br>
They build a very simple system internally.<br>
Or they do nothing at all.</p>

<p>But at the same time, these teams are building amazing products. Many of them have strong communities and loyal users. It felt clear to us that small teams deserve better tools to help them improve activation and retention.</p>

<p>Our Background in Mobile Engagement</p>

<p>My team and I have been working in the mobile engagement space for years. We previously built systems that delivered billions of messages to mobile app users across many different apps.</p>

<p>Through that experience we learned a few important things.</p>

<p>First, engagement is rarely about sending more notifications. It is about sending the right message at the right moment.</p>

<p>Second, the most important engagement moments are usually tied directly to user behavior inside the product. For example:</p>

<ul>
<li>when a user completes a meaningful action</li>
<li>when a user stops using the app for a few days</li>
<li>when a user discovers a new feature</li>
<li>when a user reaches a milestone</li>
</ul>

<p>These moments already exist inside the product logic. They are part of the user journey.</p>

<p>That observation led us to an interesting question.</p>

<p>What If Engagement Logic Came From the App Itself?</p>

<p>Most engagement tools require you to design campaigns manually in a dashboard. You create segments, define triggers, and configure messages separately from the application code.</p>

<p>But the actual user journey already lives inside the codebase.</p>

<p>The flows, the important actions, the meaningful milestones. All of that information is already there.</p>

<p>So we started experimenting with a different idea.</p>

<p>What if a system could read the structure of an app and help generate engagement scenarios automatically?</p>

<p>This idea eventually became an open source project focused on mobile engagement automation.</p>

<p>Design Principles</p>

<p>When we started building this project, we wanted it to follow a few simple principles.</p>

<p>Open Source</p>

<p>Developers should be able to see how everything works. Engagement logic should not be a black box. Teams should be able to customize the system to fit their own products.</p>

<p>SDK Free</p>

<p>We wanted to avoid forcing developers to install heavy SDK dependencies just to try the system. Many teams prefer lightweight integrations that do not add complexity to their app.</p>

<p>Agent Friendly</p>

<p>The system is designed so AI agents can understand the structure of the project and help generate engagement scenarios.</p>

<p>Agents can analyze the codebase, identify important user flows, and suggest messaging moments.</p>

<p>Config Driven</p>

<p>Campaign logic can be modified through configuration rather than requiring a full app redeployment. This makes it easier to experiment with engagement strategies.</p>

<p>The Goal</p>

<p>The goal is simple.</p>

<p>Help mobile apps automatically discover meaningful moments in their user journey and turn those moments into engagement scenarios that improve activation and retention.</p>

<p>Instead of manually designing every campaign, developers can start with a system that understands the product flow and proposes engagement ideas.</p>

<p>Still Early, Still Learning</p>

<p>This project is still very early, and we are continuing to learn from the developer community.</p>

<p>If you are building a mobile app, I would love to hear how you currently think about user engagement.</p>

<p>How do you handle push notifications?<br>
Do you design campaigns intentionally, or do they evolve naturally as the product grows?</p>

<p>I am curious to hear what works well for you and what feels painful today.</p>

<p>Check out the repo for further details.<br>
</p>
<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://assets.dev.to/assets/github-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/openclix" rel="noopener noreferrer">
        openclix
      </a> / <a href="https://github.com/openclix/openclix" rel="noopener noreferrer">
        openclix
      </a>
    </h2>
    <h3>
      Open-source, local-first mobile app retention and engagement automation with agent-driven workflows.
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<p>
<a rel="noopener noreferrer" href="https://github.com/user-attachments/assets/90bd137c-d7d4-4806-befb-94b45e005718#gh-dark-mode-only"><img alt="Event Flyer - dark" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Fuser-attachments%2Fassets%2F90bd137c-d7d4-4806-befb-94b45e005718%23gh-dark-mode-only"></a>
<a rel="noopener noreferrer" href="https://github.com/user-attachments/assets/be5ab0e3-1d3d-4e17-b13a-67d2622f1a38#gh-light-mode-only"><img alt="Event Flyer - light" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Fuser-attachments%2Fassets%2Fbe5ab0e3-1d3d-4e17-b13a-67d2622f1a38%23gh-light-mode-only"></a>
</p>

<div><strong>OpenClix</strong></div>

<div>Open-source, local-first foundation for config-driven mobile engagement logic.</div>



<div>
<a href="https://openclix.ai" rel="nofollow noopener noreferrer">Website</a>
<span> · </span>
<a href="https://github.com/openclix/openclix/./docs" rel="noopener noreferrer">Docs</a>
<span> · </span>
<a href="https://github.com/openclix/openclix" rel="noopener noreferrer">GitHub</a>
</div>

<div class="markdown-heading">
<h2 class="heading-element">Introduction</h2>
</div>

<p>OpenClix helps teams run onboarding, habit, re-engagement, and feature-discovery messaging with local-first execution.</p>

<p>It is designed to be:</p>

<ul>
<li>Open source and auditable</li>
<li>Source-first (vendored client code, not runtime SDK lock-in)</li>
<li>Agent-friendly (explicit interfaces and clear edit points)</li>
</ul>

<div class="markdown-heading">
<h2 class="heading-element">Installation</h2>
</div>

<p>OpenClix is delivered as skills + source templates.</p>

<p></p>

<b>For Humans</b>

<div class="markdown-heading">
<h3 class="heading-element">Option A: Let an agent do it</h3>
</div>

<p>Paste this prompt into your coding agent:</p>

<div class="snippet-clipboard-content notranslate position-relative overflow-auto">
<pre class="notranslate"><code>Install OpenClix skills from https://github.com/openclix/openclix and integrate OpenClix into this project
Use openclix-init to detect platform, copy templates into the dedicated OpenClix namespace,
wire initialization/event/lifecycle touchpoints, and run build verification.
If this project already has local notifications outside OpenClix, detect whether they are migration-capable,
ask whether to migrate supported flows into OpenClix or keep them as-is, and default to keeping them unchanged.
Then use openclix-design-campaigns to create .openclix/campaigns/app-profile.json
and generate .openclix/campaigns/openclix-config.json.
Do not</code></pre>…</div>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/openclix/openclix" rel="noopener noreferrer">View on GitHub</a></div>
</div>



