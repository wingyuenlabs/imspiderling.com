---
Title: From Prompt to Practical: Evolving HandyFEM’s User Flow with Claude.ai + Mermaid.live
Description: 
Author: Constanza Diaz
Date: 2025-10-14T21:43:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’m building <strong>HandyFEM</strong>, a directory + marketplace connecting women professionals in construction and repairs with clients. In my <a href="https://dev.to/constanza_diaz_dev/building-a-full-stack-web-app-from-scratch-first-steps-gi8">previous post</a>, I sketched an initial user flow using Figma and ChatGPT. This time I pushed further: I briefed <strong>Claude</strong> with the full app context and iterated toward a <strong>color-coded, developer-ready UI/navigation diagram</strong> using <strong>Mermaid</strong>.</p>

<p>This post documents what I tried, what failed, and what finally worked—so you can reproduce the setup without losing hours to tooling dead ends.</p>




<h2>
  
  
  Goal
</h2>

<p>Create a <strong>single source of truth</strong> for HandyFEM’s navigation and user journeys—clear enough for stakeholders, and concrete enough for developers to implement screens, forms, and decisions.</p>




<h2>
  
  
  Attempt 1: RapidChart (didn’t pan out)
</h2>

<p>I first tried to auto-generate the diagram via <strong>RapidChart</strong> prompts. Multiple attempts—including smaller subsets—didn’t render a chart at all. Interesting idea, but in my case it wasn’t reliable enough to ship.</p>

<p><em>Tooling takeaway:</em> if your diagram generator is brittle, switch to a text-based standard you can version-control.</p>




<h2>
  
  
  Pivot: Mermaid for reproducible, versioned diagrams
</h2>

<p>Claude suggested alternatives, and I decided to give <strong>Mermaid</strong> a try. It immediately produced a <strong>decent first draft</strong>—already more complete than my earlier diagram, even if it lacked some visual cues.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foou5l60tdd23dltt69k1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foou5l60tdd23dltt69k1.png" alt="First attempt of Mermaid code made by Claude.ai" width="800" height="393"></a></p>

<p>To enrich it, I <strong>pasted my previous flow</strong> and asked Claude to <strong>integrate both</strong>. That gave me structure + detail.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsy9p8v1zmnp2tb2i9v52.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsy9p8v1zmnp2tb2i9v52.png" alt="Asking Claude to integrate both charts" width="729" height="740"></a></p>




<h2>
  
  
  The final diagram (color-coded for quick scanning)
</h2>

<p><strong>Legend:</strong></p>

<ul>
<li>
<strong>Yellow</strong> = Buttons.
</li>
<li>
<strong>Light blue</strong> = Forms.
</li>
<li>
<strong>Teal</strong> = Screens.
</li>
<li>
<strong>Purple</strong> = Decision points.
</li>
<li>
<strong>Blue</strong> = Professional UI.
</li>
<li>
<strong>Green</strong> = Client UI.
</li>
<li>
<strong>Red</strong> = Admin UI.
</li>
<li>
<strong>Gray</strong> = System processes.
</li>
</ul>

<p>This mapping keeps discussions concrete. Designers and developers can point to the exact element in the flow and agree on behavior.</p>

<blockquote>
<p>Note: I iterated a few times in Claude to complete the <strong>admin</strong> path.</p>
</blockquote>

<p>The image below is a small screenshot just for reference.</p>

<p><strong>UI/Navigation flow and User journey flow:</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fep0us9lzjb1xoao6nqi2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fep0us9lzjb1xoao6nqi2.png" alt="Final UI/Navigation flow diagram (unreadable)" width="664" height="920"></a></p>

<p><a href="https://www.mermaidchart.com/app/projects/75e90edc-ad33-442c-a2c1-e97de4c73342/diagrams/66e54f91-963a-4145-b23b-2f6d9ab2f876/share/invite/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkb2N1bWVudElEIjoiNjZlNTRmOTEtOTYzYS00MTQ1LWIyM2ItMmY2ZDlhYjJmODc2IiwiYWNjZXNzIjoiQ29tbWVudCIsImlhdCI6MTc2MDQ3NTI5OH0.Unm61oiCBHYJt1VUdePIxYAaHUlOri0mOFTKa-utKSM" rel="noopener noreferrer">Full size:</a></p>




<h2>
  
  
  Minimal, reproducible snippet (Mermaid)
</h2>

<p>Here’s a simplified fragment that captures the pattern I used—screens, forms, decisions, and role-scoped branches. You can drop this into any Mermaid-enabled markdown and expand it for your app.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>flowchart TD
  %% Legend-inspired classes
  classDef screen fill:#0fb,stroke:#066;       %% Teal
  classDef form fill:#b3e5fc,stroke:#0288d1;   %% Light blue
  classDef decision fill:#b39ddb,stroke:#512da8; %% Purple
  classDef button fill:#ffeb3b,stroke:#f57f17; %% Yellow
  classDef pro fill:#90caf9,stroke:#1565c0;    %% Blue (Professional UI)
  classDef cli fill:#a5d6a7,stroke:#2e7d32;    %% Green (Client UI)
  classDef admin fill:#ef9a9a,stroke:#c62828;  %% Red (Admin UI)
  classDef system fill:#bdbdbd,stroke:#616161; %% Gray (System)

  %% Entry
  A[Landing Screen]:::screen --&gt; B{Choose role}:::decision

  %% Client path
  B --&gt;|Client| C[Client Dashboard]:::screen
  C --&gt; D[Search Professionals Form]:::form
  D --&gt; E[Results Screen]:::screen
  E --&gt; F[Request Quote Button]:::button
  F --&gt; G[Request Form]:::form
  G --&gt; H[System: Notify matched pros]:::system

  %% Professional path
  B --&gt;|Professional| P[Pro Dashboard]:::screen
  P --&gt; Q[Complete Profile Form]:::form
  Q --&gt; R[Verify Identity Button]:::button
  R --&gt; S[System: KYC check]:::system
  S --&gt; T{Approved?}:::decision
  T --&gt;|Yes| U[Receive job requests]:::screen
  T --&gt;|No| V[Manual review]:::admin

  %% Admin path
  B --&gt;|Admin| A1[Admin Panel]:::admin
  A1 --&gt; A2[Moderate Listings]:::admin
  A1 --&gt; A3[Resolve Disputes]:::admin
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ty3xym6gtfyopxhyiwu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ty3xym6gtfyopxhyiwu.png" alt="simplified chart" width="800" height="698"></a></p>

<p><strong>Why this helps:</strong>  </p>

<ul>
<li>
<strong>Text-as-diagram</strong> means you can version the flow in Git, review it in pull requests, and co-edit with your team.
</li>
<li>
<strong>Color classes</strong> encode semantics you can reuse across large diagrams.
</li>
<li>
<strong>Branching by role</strong> makes gaps obvious and audit-able.</li>
</ul>




<h2>
  
  
  Practical notes
</h2>

<ul>
<li>
<strong>Prompt shape matters.</strong> My best results came from supplying Claude with the <strong>app glossary, roles, and UI primitives</strong> (screen, form, decision, button, system).
</li>
<li>
<strong>Iterate role by role.</strong> I filled Client and Professional paths first, then Admin.
</li>
<li>
<strong>Keep it modular.</strong> Break huge flows into <strong>linked sub-diagrams</strong> per feature if your tool or host chokes on size.
</li>
<li>
<strong>Treat it like code.</strong> Store the <code>.md</code> or <code>.mmd</code> in your repo and make changes via PRs.</li>
</ul>




<h2>
  
  
  What I’ll do next
</h2>

<ul>
<li>Split the mega-diagram into feature-level Mermaid files.
</li>
<li>Connect each node to <strong>issue links</strong> or <strong>Figma frames</strong> for implementation hand-off.
</li>
<li>Add <strong>acceptance criteria</strong> to decisions and forms to reduce ambiguity during dev.</li>
</ul>




<p>If you’re mapping a complex app, pairing a capable LLM with Mermaid gives you speed, structure, and a durable artifact your team can actually build from.</p>

