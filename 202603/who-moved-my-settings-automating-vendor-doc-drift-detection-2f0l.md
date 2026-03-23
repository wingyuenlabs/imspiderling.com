---
Title: Who Moved My Settings? Automating Vendor Doc Drift Detection
Description: 
Author: Rick Cogley
Date: 2026-03-23T21:58:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>Spencer Johnson's <em>Who Moved My Cheese?</em> is about adapting when things change around you. In cloud security compliance, the cheese moves constantly. Microsoft renames a menu. Google moves a toggle to a different admin panel. Your carefully written implementation guide now points to a screen that no longer exists. Your client follows the old steps and hits a dead end.</p>

<p>We built an automated pipeline that crawls vendor documentation weekly, compares it against our internal implementation guides, and flags meaningful drift: renamed UI paths, deprecated features, relocated settings. It runs on Cloudflare Workers with a Durable Workflow, uses Claude for semantic analysis, and costs about 50 cents a week at scale. Here's how.</p>

<h2>
  
  
  The Problem: Documentation Drift
</h2>

<p>We maintain implementation guides for security controls, such as "Enable MFA for Admin Accounts in Microsoft Entra" or "Configure Conditional Access Policies." Each guide has specific, click-by-click instructions referencing particular admin portal locations, feature names, and configuration options.</p>

<p>Cloud vendors update their platforms constantly. Microsoft alone pushes changes to Entra ID (formerly Azure AD) multiple times a month. When they rename a feature or reorganize an admin panel, our guides become incorrect. We call this <strong>documentation drift</strong>.</p>

<p>Before building this system, drift detection was entirely manual. A consultant would notice a discrepancy during a client engagement, report it, and someone would update the guide. The feedback loop was slow and reactive.</p>

<h2>
  
  
  The Architecture: A Three-Gate Pipeline
</h2>

<p>We wanted something cheap to run, resistant to false alarms, and automated. The solution is a three-gate pipeline where each gate filters out unnecessary work before reaching the next, more expensive step.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmermaid.ink%2Fimg%2FZmxvd2NoYXJ0IFRECiAgICBBW0Nyb24gVHJpZ2dlciBNb25kYXkgNiBBTSBKU1RdIC0tPiBCW0xvYWQgQWN0aXZlIERvYyBTb3VyY2VzXQogICAgQiAtLT4gQ3tHYXRlIDEgUlNTIENoZWNrfQogICAgQyAtLT58Tm8gY2hhbmdlc3wgU2tpcDFbU2tpcCAtIFNhdmUgY29zdF0KICAgIEMgLS0-fFBvc3NpYmxlIGNoYW5nZXN8IERbQ3Jhd2wgVmVuZG9yIERvY3NdCiAgICBEIC0tPiBFe0dhdGUgMiBDb250ZW50IEhhc2h9CiAgICBFIC0tPnxIYXNoIG1hdGNoZXN8IFNraXAyW1NraXAgLSBObyByZWFsIGNoYW5nZV0KICAgIEUgLS0-fEhhc2ggZGlmZmVyc3wgRntHYXRlIDMgQUkgQW5hbHlzaXN9CiAgICBGIC0tPnxObyBtZWFuaW5nZnVsIGRyaWZ0fCBHW01hcmsgQ3VycmVudF0KICAgIEYgLS0-fERyaWZ0IGRldGVjdGVkfCBIW0ZsYWcgZm9yIFJldmlld10%3D%3Ftheme%3Ddefault" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmermaid.ink%2Fimg%2FZmxvd2NoYXJ0IFRECiAgICBBW0Nyb24gVHJpZ2dlciBNb25kYXkgNiBBTSBKU1RdIC0tPiBCW0xvYWQgQWN0aXZlIERvYyBTb3VyY2VzXQogICAgQiAtLT4gQ3tHYXRlIDEgUlNTIENoZWNrfQogICAgQyAtLT58Tm8gY2hhbmdlc3wgU2tpcDFbU2tpcCAtIFNhdmUgY29zdF0KICAgIEMgLS0-fFBvc3NpYmxlIGNoYW5nZXN8IERbQ3Jhd2wgVmVuZG9yIERvY3NdCiAgICBEIC0tPiBFe0dhdGUgMiBDb250ZW50IEhhc2h9CiAgICBFIC0tPnxIYXNoIG1hdGNoZXN8IFNraXAyW1NraXAgLSBObyByZWFsIGNoYW5nZV0KICAgIEUgLS0-fEhhc2ggZGlmZmVyc3wgRntHYXRlIDMgQUkgQW5hbHlzaXN9CiAgICBGIC0tPnxObyBtZWFuaW5nZnVsIGRyaWZ0fCBHW01hcmsgQ3VycmVudF0KICAgIEYgLS0-fERyaWZ0IGRldGVjdGVkfCBIW0ZsYWcgZm9yIFJldmlld10%3D%3Ftheme%3Ddefault" alt="Three-Gate Pipeline" width="656" height="1205"></a></p>

<h3>
  
  
  Gate 1: RSS Check (Free, Milliseconds)
</h3>

<p>Microsoft Learn exposes an RSS endpoint that reports page modification dates. Before crawling anything, we check whether the vendor has even published changes since our last run. If the RSS timestamps haven't moved, we skip the entire source. This gate eliminates most checks in a typical week.</p>

<h3>
  
  
  Gate 2: Content Hash (Cheap, Seconds)
</h3>

<p>When RSS indicates something might have changed, we crawl the vendor page and compute a SHA-256 hash of the markdown content. If the hash matches what we stored last time, the page content hasn't actually changed. Maybe the RSS date shifted for editorial reasons, or the page got a cosmetic update. Skip.</p>

<h3>
  
  
  Gate 3: AI Drift Analysis (API Cost, Seconds)
</h3>

<p>Only when the content has genuinely changed do we send it to Claude for semantic comparison against our implementation guide. The AI isn't doing a text diff. It detects specific categories of drift:</p>

<ul>
<li>
<strong>UI path changes</strong>: "Settings &gt; Security" became "Protection &gt; Authentication"</li>
<li>
<strong>Feature deprecation</strong>: a feature we reference no longer exists</li>
<li>
<strong>Step reordering</strong>: the vendor changed the sequence of configuration steps</li>
<li>
<strong>New prerequisites</strong>: a step now requires something it didn't before</li>
<li>
<strong>Setting relocation</strong>: a toggle moved from one admin panel to another</li>
</ul>

<p>It deliberately ignores minor wording changes, formatting differences, and additive features that don't affect existing instructions.</p>

<h2>
  
  
  Why a Companion Worker?
</h2>

<p>Our main application runs on SvelteKit with Cloudflare's adapter, which doesn't support Workflow classes or <code>scheduled</code> handlers. So the crawler runs as a separate Worker that shares the same D1 database. Two Workers, one database, clean separation of concerns.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmermaid.ink%2Fimg%2FZmxvd2NoYXJ0IExSCiAgICBzdWJncmFwaCBNYWluW1B1bHNlIFN2ZWx0ZUtpdF0KICAgICAgICBkaXJlY3Rpb24gVEIKICAgICAgICBBUEkxWy9hcGkvZG9jLWZyZXNobmVzcy9zdGF0dXNdCiAgICAgICAgQVBJMlsvYXBpL2RvYy1mcmVzaG5lc3MvcmV2aWV3XQogICAgZW5kCiAgICBzdWJncmFwaCBDb21wYW5pb25bRG9jIEZyZXNobmVzcyBXb3JrZXJdCiAgICAgICAgZGlyZWN0aW9uIFRCCiAgICAgICAgQ3JvbltDcm9uIFRyaWdnZXJdIC0tPiBXRltEdXJhYmxlIFdvcmtmbG93XQogICAgICAgIE1hbnVhbFtQT1NUIC9ydW5dIC0tPiBXRgogICAgZW5kCiAgICBEQlsoRDEgQ2VudHJhbCldCiAgICBNYWluIDwtLT4gREIKICAgIENvbXBhbmlvbiA8LS0-IERC%3Ftheme%3Ddefault" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmermaid.ink%2Fimg%2FZmxvd2NoYXJ0IExSCiAgICBzdWJncmFwaCBNYWluW1B1bHNlIFN2ZWx0ZUtpdF0KICAgICAgICBkaXJlY3Rpb24gVEIKICAgICAgICBBUEkxWy9hcGkvZG9jLWZyZXNobmVzcy9zdGF0dXNdCiAgICAgICAgQVBJMlsvYXBpL2RvYy1mcmVzaG5lc3MvcmV2aWV3XQogICAgZW5kCiAgICBzdWJncmFwaCBDb21wYW5pb25bRG9jIEZyZXNobmVzcyBXb3JrZXJdCiAgICAgICAgZGlyZWN0aW9uIFRCCiAgICAgICAgQ3JvbltDcm9uIFRyaWdnZXJdIC0tPiBXRltEdXJhYmxlIFdvcmtmbG93XQogICAgICAgIE1hbnVhbFtQT1NUIC9ydW5dIC0tPiBXRgogICAgZW5kCiAgICBEQlsoRDEgQ2VudHJhbCldCiAgICBNYWluIDwtLT4gREIKICAgIENvbXBhbmlvbiA8LS0-IERC%3Ftheme%3Ddefault" alt="Companion Worker Architecture" width="800" height="400"></a></p>

<p>The companion Worker uses Cloudflare Workflows for durability. Each doc source gets its own named steps. If a crawl job fails partway through, the workflow resumes from the last completed step rather than starting over. Durable sleep steps let the Worker wait for asynchronous crawl jobs without burning CPU time.</p>

<h2>
  
  
  Dealing with a Flaky API
</h2>

<p>The Cloudflare Browser Rendering <code>/crawl</code> endpoint is asynchronous and occasionally unreliable. Jobs sometimes error out immediately for no apparent reason. We handle this with:</p>

<ul>
<li>
<strong>3 crawl attempts</strong> with 20-second backoff between retries</li>
<li>
<strong>15-second stagger</strong> between sources to avoid rate limiting</li>
<li>
<strong>Polling with durable sleep</strong>: the Worker sleeps (zero CPU cost) between poll checks</li>
</ul>

<p>This brought our success rate from roughly 50% to about 90%. The remaining failures are typically transient and resolve on the next weekly run.</p>

<h2>
  
  
  What Drift Detection Looks Like
</h2>

<p>When the system detects drift, it stores a bilingual summary (English and Japanese) with structured change details. A staff member reviews the flag and either updates the implementation guide or dismisses it as a false positive.</p>

<p>Real drift we've caught so far:</p>

<ul>
<li>Microsoft deprecated per-user MFA in favor of Conditional Access policies</li>
<li>UI paths in the Entra admin center were reorganized</li>
<li>A grace period for legacy authentication blocking was removed</li>
</ul>

<p>Each of these would have eventually caused confusion during a client engagement. Instead, we caught them within a week of the vendor change.</p>

<h2>
  
  
  Cost
</h2>

<p>For our current 10 doc sources (5 M365 controls × 2 languages), the weekly cost rounds to zero. At scale (say 120 sources covering Microsoft 365, Google Workspace, and Cloudflare) we estimate about $0.50 per week. Gate 1 and Gate 2 eliminate most of the expensive AI calls.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Component</th>
<th>Weekly Cost</th>
</tr>
</thead>
<tbody>
<tr>
<td>RSS checks (Gate 1)</td>
<td>Free</td>
</tr>
<tr>
<td>Browser Rendering crawls</td>
<td>~$0.30</td>
</tr>
<tr>
<td>Claude AI analysis (Gate 3)</td>
<td>~$0.15</td>
</tr>
<tr>
<td>Worker compute + D1 storage</td>
<td>Free tier</td>
</tr>
<tr>
<td><strong>Total (~120 sources)</strong></td>
<td><strong>~$0.50</strong></td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  What's Next
</h2>

<p>The current system covers Microsoft 365 identity controls. The pipeline is vendor-agnostic. Adding a new doc source is just a database row with a URL and crawl configuration. Next steps:</p>

<ul>
<li>
<strong>Expand coverage</strong> to remaining M365 controls plus Google Workspace and Cloudflare</li>
<li>
<strong>Build the review UI</strong> so staff can view diffs, approve updates, and track guide versions</li>
<li>
<strong>Vendor-specific adapters</strong> for Gate 1 (Google and Cloudflare have different changelog patterns than Microsoft's RSS)</li>
<li>
<strong>Automatic guide updates</strong> where the AI suggests specific text changes, not just flags</li>
</ul>

<h2>
  
  
  Key Takeaways
</h2>

<p>Vendor documentation drift is a real operational risk for any team maintaining compliance guides. Catching it doesn't require a complex system. A pipeline that filters aggressively at each stage keeps both costs and false alarms low. Cloudflare's Worker platform, with Workflows for durability and Browser Rendering for crawling, turned out to be a natural fit for this kind of periodic, asynchronous inspection work.</p>

<p>The code runs once a week, costs pocket change, and has already caught real drift that would have burned consultant time. That's a good trade.</p>




<p><em>Originally published at <a href="https://cogley.jp/articles/automating-vendor-doc-drift-detection" rel="noopener noreferrer">cogley.jp</a></em></p>

<p><em>Rick Cogley is CEO of <a href="https://esolia.co.jp/en/" rel="noopener noreferrer">eSolia Inc.</a>, providing bilingual IT outsourcing and infrastructure services in Tokyo, Japan.</em></p>

