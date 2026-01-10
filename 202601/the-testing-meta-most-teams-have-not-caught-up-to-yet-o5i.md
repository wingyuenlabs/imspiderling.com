---
Title: The testing meta most teams have not caught up to yet
Description: 
Author: Murat K Ozcan
Date: 2026-01-10T21:22:19.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  A QA operating system for AI testing
</h2>

<p>This post is for anyone shipping Playwright tests and watching AI output rot in review. I built this to reduce review bandwidth and stop AI from guessing - it is a QA operating system built from real test architecture, not prompt luck.</p>

<p>Quick links:</p>

<ul>
<li>
<a href="https://github.com/seontechnologies/playwright-utils" rel="noopener noreferrer">@seontechnologies/playwright-Utils</a>: Playwright DX and reusable utilities</li>
<li>
<a href="https://github.com/bmad-code-org/BMAD-METHOD" rel="noopener noreferrer">BMAD Method</a>: agentic development framework</li>
<li>
<a href="https://github.com/bmad-code-org/BMAD-METHOD/blob/c18904d6740529036135b773103a365e7e53b5c7/docs/explanation/features/tea-overview.md" rel="noopener noreferrer">TEA overview</a>: BMAD Test Architect workflows</li>
</ul>

<p>TL;DR: TEA (Test Architect) encodes test strategy and release gates, Playwright-Utils fixes Playwright DX, and Playwright MCPs give AI live verification. Together, they turn AI testing from promptware into repeatable engineering by standardizing patterns and verifying against the real system.</p>

<p>If your AI tests look productive but rot in review, you do not have a test suite, you have a slop factory.<br>
By slop I mean redundant coverage, wrong assertions, nondeterministic flows, and unreviewable diffs.</p>

<p>I stopped trusting AI test generation the day it could not explain my own quality standards, so I embedded them.</p>

<h2>
  
  
  Fixing DX first: Playwright-Utils
</h2>

<p>The first thing I addressed was Playwright DX. Playwright-Utils bridges the gap between Cypress-like ergonomics and Playwright's power with a functional core and fixture shell design. Many capabilities work as a pure function or as a fixture, for UI as well as API/backend flows. It standardizes the primitives teams drift on: auth, API, retries, logging, files, etc.</p>

<h2>
  
  
  The brain: TEA (Test Architect agent from BMAD)
</h2>

<p>BMAD TEA (Test Architect) is a quality operating model as an agent: risk-based test design, fixture architecture, CI/CD gates, and release readiness packaged as workflows. I'm a certified Siemens test architect. TEA is how I made my quality standards executable - workflows instead of tribal memory.</p>

<p>TEA spans eight workflows: <code>*framework</code>, <code>*ci</code>, <code>*test-design</code>, <code>*atdd</code> (acceptance test driven design), <code>*automate</code>, <code>*test-review</code>, <code>*trace</code>, and <code>*nfr-assess</code> - everything from strategy to gates.</p>

<h2>
  
  
  The hands and eyes: Playwright MCPs
</h2>

<p>Playwright MCPs arrived and turned diagnosis, execution, and fixing into a live loop. MCPs close the loop: the agent can run the flow, confirm the DOM against the accessibility tree / the network responses against the API spec, and use Playwright's Trace Viewer/UI mode as the verification canvas. For backend behavior, API flows are validated via <code>api-request</code> plus <code>recurse</code>, with an optional UI mode. Later, Playwright shipped its agent prompt packs, and I folded that guidance into TEA as an optional accelerator.</p>

<h2>
  
  
  How it works in practice
</h2>

<ol>
<li>TEA produces the test strategy and gate criteria for the feature.</li>
<li>TEA generates tests using Playwright-Utils patterns (auth, retries, API, logging, etc.).</li>
<li>MCPs verify UI flows / API behavior against the live system.</li>
<li>Failures feed back into TEA to repair, using the same standards.</li>
<li>Output: PR-ready tests plus traceability and gate artifacts (risk table, requirements-to-tests traceability matrix, release gate checklist).</li>
</ol>

<p>Before example: 20 AI tests -&gt; 12 flaky, 5 redundant, 3 wrong assertions<br>
After: risk plan + gates -&gt; P0/P1 only -&gt; selectors/API calls verified -&gt; behavior validated -&gt; fewer tests, higher signal</p>

<h2>
  
  
  Why this stack matters
</h2>

<p>Playwright is intentionally unopinionated, which means no two projects look the same and drift happens fast, even inside the same org. It optimizes for stability and speed over standardized DX, so teams reinvent the same primitives. Every team reinvents auth, API calls, retries, and logging. Then the AI arrives at a mess, and a mess produces slop.</p>

<p>Meanwhile, prompt-driven testing paths like <code>cy.prompt</code> lean into nondeterminism, which is the exact opposite of what testing exists to protect. The fix is not a better prompt; it is a better system: TEA to encode the standards, Playwright-Utils to enforce the patterns, and MCPs to verify UI flows against the live system, with <code>api-request</code> plus <code>recurse</code> covering backend behavior.</p>

<h2>
  
  
  What you get out of the box
</h2>

<p>Playwright-Utils provides nine utilities. Six are backend or frontend agnostic: <code>api-request</code> (with schema validation), <code>recurse</code>, <code>log</code>, <code>file-utils</code>, <code>burn-in</code>, and <code>auth-session</code>. Three are UI-focused:  <code>network-error-monitor</code>, <code>network-recorder</code>, and <code>intercept-network-call</code>.</p>

<p>TEA uses those patterns by default when you enable the integration in the BMAD installer, so the AI does not have to relearn the same documentation every time. That is context engineering in practice.</p>

<h2>
  
  
  Call to action
</h2>

<p>This stack is already open source and being used in production at <a href="https://seon.io/" rel="noopener noreferrer">SEON technologies</a>. If you want to level up your craft, use it. If you want to push it further, contribute. If you want to build with people who care about quality at this level, reach out.</p>

