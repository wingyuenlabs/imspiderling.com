---
Title: Safer Pushes, SIEM Webhooks, and Verifiable Releases
Description: 
Author: Joe Rucci
Date: 2026-03-04T21:45:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>Recent Ghostable updates make secret operations safer by default, improve resilience during outages, and standardize release integrity evidence.</p>

<p>We shipped a set of product updates focused on one practical outcome: <strong>make secret operations safer by default, easier to monitor, and easier to verify in real team workflows</strong>.</p>

<p>These updates span CLI, Desktop, and Dashboard behavior, and reduce day-to-day risk without changing Ghostable's <a href="https://ghostable.dev/blog/ghostable-v2-zero-knowledge-security" rel="noopener noreferrer">zero-knowledge security model</a>.</p>

<h2>
  
  
  Latest versions
</h2>

<ul>
<li>Ghostable CLI: <strong>v2.5.2</strong> (<a href="https://www.npmjs.com/package/@ghostable/cli" rel="noopener noreferrer">NPM</a>)</li>
<li>Ghostable Desktop: <strong>v1.1.3</strong> (<a href="https://fls-9fe4cb31-f981-461b-b08f-6d8b2fed8bbf.laravel.cloud/downloads/desktop/stable/Ghostable-1.1.3.dmg" rel="noopener noreferrer">Download macOS client</a>)</li>
</ul>

<h2>
  
  
  Push and sync safety is stricter by default
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>❌ Strict conflict mode blocked this push due to version drift.
  - LOCAL_ACCESS_SECRET: <span class="nb">local </span>v19 vs server v21
  - LOCAL_ACCESS_SECRET_02: <span class="nb">local </span>v1 vs server v2
Run <span class="sb">`</span>ghostable <span class="nb">env </span>state refresh<span class="sb">`</span> to update versions, or use <span class="nt">--force-overwrite</span><span class="nb">.</span>
</code></pre>

</div>



<p>Conflict handling on push and sync flows now defaults to strict behavior, which reduces accidental overwrites when multiple team members or systems are changing environment state in parallel.</p>

<p>The CLI now supports explicit conflict controls with <code>--conflict-mode warn|strict</code> and <code>--force-overwrite</code> when an intentional overwrite is actually what you want. Local version baselines are also maintained more consistently through environment state refresh and automatic refresh on pull.</p>

<p>Conflict responses are now deterministic: HTTP <code>409</code> with a structured <code>conflicts[]</code> payload. That gives both human users and automation a reliable contract for handling stale-state writes.</p>

<h2>
  
  
  Audit webhooks are now observable operations
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxjqpnedzdvjzq6duumu3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxjqpnedzdvjzq6duumu3.png" alt="webhook audit user interface" width="800" height="553"></a></p>

<p>Organizations can now configure audit webhooks with signed delivery. Delivery behavior includes retries and dead-letter handling, and we added metrics plus health visibility for outcomes so teams can distinguish transient delivery issues from persistent integration failures.</p>

<p>This closes a common monitoring gap: audit events are only useful if security teams can trust they are actually arriving and can quickly detect when they are not. The implementation details and templates live in the <a href="https://docs.ghostable.dev/v2/digging-deeper/siem-audit-webhook-templates" rel="noopener noreferrer">SIEM and audit webhook docs</a>.</p>

<h2>
  
  
  Deploy fallback is controlled, encrypted, and fail-closed
</h2>

<p>The CLI now supports encrypted deploy bundle caching with controlled stale fallback via <code>--allow-stale-cache</code>, bounded by TTL and integrity checks. This improves deployment resilience during temporary service disruptions without introducing silent security regressions.</p>

<p>Invalid, tampered, or expired cache artifacts fail closed, and fallback does not bypass authentication failures. In other words, this improves continuity under outages, not trust assumptions. See the deployment behavior details in <a href="https://docs.ghostable.dev/v2/digging-deeper/deployments" rel="noopener noreferrer">Deployments</a>.</p>

<h2>
  
  
  Desktop conflict handling is more explicit
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6jxoj1q69ihl65b3bhqj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6jxoj1q69ihl65b3bhqj.png" alt="screenshot from Ghostable desktop client alerting user to server-side changes detected" width="716" height="550"></a></p>

<p>Desktop now decodes structured conflict payloads and gives direct guidance in stale-version cases: reload for a fresh baseline or intentionally overwrite when appropriate. The goal is clarity under pressure, especially when multiple actors are touching the same environment.</p>

<h2>
  
  
  Release integrity evidence is now standardized
</h2>

<p>Release integrity assets are now produced and published as part of the release process: checksums, SBOMs, and provenance where the platform supports it. Verification guidance and security controls documentation were also updated.</p>

<p>For private repositories, workflows were adjusted to handle attestation-plan limitations while still publishing practical integrity evidence teams can validate. If your organization validates build origin and artifact lineage, this aligns with the direction of frameworks like <a href="https://slsa.dev/spec/v1.0/provenance" rel="noopener noreferrer">SLSA provenance</a> and gives you concrete artifacts to check.</p>

<h2>
  
  
  What this changes for teams
</h2>

<ul>
<li>Fewer accidental secret overwrites during concurrent changes</li>
<li>Clearer stale-state prompts and conflict resolution flows</li>
<li>Better deploy reliability during temporary outages, without weakening controls</li>
<li>More transparent audit trails and delivery visibility for security teams</li>
<li>Stronger trust signals for teams that verify what they install</li>
</ul>

<h2>
  
  
  What this does not change
</h2>

<p>These updates do not relax authentication boundaries, do not allow Ghostable to read secret values, and do not turn fallback behavior into an override of security controls. They are operational hardening updates to default behavior and visibility, not a shift in security model.</p>

<h2>
  
  
  Upgrade and verify
</h2>

<p>If you want to adopt these changes now, review <a href="https://docs.ghostable.dev/v2/digging-deeper/supply-chain-verification" rel="noopener noreferrer">supply-chain verification</a>, and share the updated controls view with security stakeholders via the <a href="https://docs.ghostable.dev/v2/digging-deeper/security-controls-matrix" rel="noopener noreferrer">security controls matrix</a>. You can also share Ghostable's trust overview from <a href="https://ghostable.dev/trust" rel="noopener noreferrer">ghostable.dev/trust</a>.</p>

<h3>
  
  
  Desktop
</h3>

<p>Open Ghostable, then go to <code>Ghostable -&gt; Check for Updates</code> and choose <strong>Install and Relaunch</strong>.</p>

<h3>
  
  
  CLI
</h3>

<p>Update the CLI globally with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> @ghostable/cli@latest
</code></pre>

</div>



