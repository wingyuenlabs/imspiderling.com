---
Title: Building funsize-engineer
Description: 
Author: Jessica Rudd
Date: 2026-01-20T22:13:31.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>In the modern software landscape, the barrier between an idea and a deployable artifact is often the friction of boilerplate and configuration. This post documents the creation of funsize-engineer, a Python package designed to serve as my terminal-based “business card”. Inspired by David Neal’s npm calling card, I wanted to create a Python equivalent that could be easily distributed and run via PyPI.The goal was simple: create a distributable Python package that, when run, displays professional contact information and ASCII art in a visually appealing format. However, the method of creation was novel. Instead of writing every line of code manually, I utilized Google Antigravity, an agentic coding platform. This article demonstrates how the “Agent-First” development paradigm shifts the engineer’s role from typist to architect, focusing on task orchestration and verification rather than syntax.</p>

<h2>
  
  
  Antigravity Core Functionality
</h2>

<p>Antigravity operates on a fundamental shift in the developer-tool relationship. It is not merely an autocomplete engine; it is an autonomous agent capable of executing complex, multi-step tasks.</p>

<p>Two core components enable this workflow:</p>

<ol>
<li>
<strong>The Manager View</strong>: Unlike a standard chat interface, Antigravity maintains a structured “Manager View” of the project. It tracks high-level objectives, breaks them down into sub-tasks (e.g., “Scaffold Repository”, “Implement Core Logic”, “Configure CI/CD”), and maintains context across the entire development lifecycle. This allows the agent to “remember” architectural decisions made in step one while executing step ten.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyswp0tkjh7ch2pui87ef.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyswp0tkjh7ch2pui87ef.png" alt=" " width="800" height="477"></a></p>

<ol>
<li>
<strong>Artifact-Driven Workflow</strong>: The agent does not just stream code into a void. It generates Artifacts—structured documents like Implementation Plans, Task Lists, and Code Diffs. These artifacts serve as checkpoints. The agent proposes a plan, the engineer reviews and approves it, and only then does the agent execute. This “Human-in-the-Loop” model ensures that autonomy does not come at the cost of control. </li>
</ol>

<h2>
  
  
  The Package Creation Workflow
</h2>

<p>The development of funsize-engineer followed a structured lifecycle, orchestrated by the agent.</p>

<h3>
  
  
  Phase 1: Planning and Scaffolding
</h3>

<p>The project began with a high-level directive. The agent analyzed the requirements—a Python package, rich library for UI, and PyPI distribution—and generated an implementation plan. It then scaffolded the directory structure, creating pyproject.toml for modern standards-compliant packaging and setting up the initial git repository.</p>

<h3>
  
  
  Phase 2: Core Logic Development
</h3>

<p>With the foundation in place, the agent implemented the core functionality in funsize_engineer/card.py. It utilized the rich library to create a grid layout, integrating ASCII art loaded from an asset file and styling the text with specific color tokens (e.g., #8A2BE2 for branding).</p>

<h3>
  
  
  Phase 3: CI/CD and Versioning
</h3>

<p>The most complex phase involved automating the release process. The agent configured GitHub Actions workflows to handle dual publishing streams:</p>

<ul>
<li><p><strong>TestPyPI</strong>: For development snapshots from the develop branch.</p></li>
<li><p><strong>PyPI</strong>: For stable releases from the main branch.</p></li>
</ul>

<h2>
  
  
  Agentic Artifact Review: Validating the Workflow
</h2>

<p>A critical moment in the project was the configuration of the CI/CD pipeline. The agent proposed a GitHub Actions workflow to automate publishing.</p>

<p><strong>Initial Plan Artifact (Excerpt):</strong></p>

<ul>
<li>Task: Configure GitHub Actions for PyPI publishing.</li>
<li>Action: Create .github/workflows/publish.yml.</li>
<li>Developer Review: Required.</li>
</ul>

<p><strong>Code Diff (Excerpt):</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7wi28oo8aupaw61h16c0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7wi28oo8aupaw61h16c0.png" alt=" " width="" height=""></a></p>

<p>This artifact allowed the developer to verify that the workflow was correctly targeting both branches and, crucially, that fetch-depth: 0 was included to ensure setuptools-scm could correctly calculate versions from git tags.</p>

<h2>
  
  
  Troubleshooting and Iteration
</h2>

<p>No development process is without friction. We encountered two significant technical hurdles that required agentic problem-solving.</p>

<h3>
  
  
  1. The “Dev Version” Trap
</h3>

<p>During the setup of the automated versioning, we encountered an issue where releases from the main branch were being tagged with “dev” suffixes (e.g., 0.2.11.dev0) instead of clean release versions, despite having a correct git tag (e.g., v0.2.11).</p>

<p><strong>The Issue</strong>: setuptools-scm calculates versions based on the distance from the last tag. In our GitHub Actions workflow, we were creating a tag and then checking it out. However, setuptools-scm saw the commit after the tag (the one triggering the workflow) and assumed it was a “dirty” state, appending a .dev suffix. Additionally, the workflow wasn’t properly stripping the v prefix from tags before passing them to the build system.</p>

<p><strong>The Resolution</strong>: The agent diagnosed the issue and implemented a robust fix:</p>

<ol>
<li>
<strong>Explicit Versioning</strong>: Instead of relying on git state, we forced setuptools-scm to use the exact version we wanted by setting the SETUPTOOLS_SCM_PRETEND_VERSION environment variable.</li>
<li>
<strong>Prefix Handling</strong>: We added logic to strip the v prefix from git tags (e.g., v0.2.11 -&gt; 0.2.11) to ensure Python package compatibility.</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgtsmcrpkl4a101n1aste.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgtsmcrpkl4a101n1aste.png" alt=" " width="800" height="75"></a></p>

<h3>
  
  
  2. Rendering Diagrams on GitHub Pages
</h3>

<p>We wanted to include a Mermaid.js diagram to visualize the agentic workflow. However, GitHub Pages (via Jekyll) rendered the mermaid code blocks as raw text instead of diagrams.</p>

<p><strong>The Fix</strong>: The agent identified that the default markdown code block syntax ( `<code>mermaid ) wasn’t being processed by the Mermaid JavaScript library. We switched to using raw HTML &lt;pre class=”mermaid”&gt;</code> tags, which allowed the diagram to render perfectly without requiring complex Jekyll plugins.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpjaumi1mgkxc67ow6js3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpjaumi1mgkxc67ow6js3.png" alt=" " width="800" height="132"></a></p>

<h2>
  
  
  Final Code &amp; Output
</h2>

<p>The result is a lightweight, installable package. The core logic resides in funsize_engineer/card.py, which orchestrates the rich components.</p>

<p><strong>Final Example Code:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzpn91r5ci19avp0ba9nr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzpn91r5ci19avp0ba9nr.png" alt=" " width="800" height="432"></a></p>

<p><strong>Example Output:</strong></p>

<p>Running pipx run funsize-engineer now renders the following in the terminal:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz5574gazu8mnfbwg55bq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz5574gazu8mnfbwg55bq.png" alt=" " width="616" height="694"></a></p>

<h2>
  
  
  Conclusion
</h2>

<p>The creation of funsize-engineer highlights the power of the Agent-First methodology. By offloading the implementation details—from directory scaffolding to complex CI/CD scripting—to Antigravity, I was able to focus on the what and why of the product. The result is a professional-grade Python package built in a fraction of the time, with the agent acting not just as a coder, but as a partner in the engineering process.</p>

