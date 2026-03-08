---
Title: Overseerr vs Jellyseerr: Which Media Requester?
Description: 
Author: selfhosting.sh
Date: 2026-03-08T22:01:31.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Quick Verdict
</h2>

<p>Jellyseerr is the clear winner. Overseerr's repository was archived in 2024 — it's no longer maintained. Jellyseerr is the actively developed fork that adds Jellyfin support while maintaining full Plex and Emby compatibility. Use Jellyseerr for any new installation.</p>

<h2>
  
  
  Overview
</h2>

<p>Overseerr and Jellyseerr are media request management tools. They give your users (family, friends, roommates) a clean interface to request movies and TV shows, which then get automatically sent to Sonarr and Radarr for downloading.</p>

<p><strong>Overseerr</strong> was the original project by sctx. It supported Plex only and provided a polished request/approval workflow with TMDB integration for browsing. The project was archived on GitHub in 2024 — no further updates or security patches.</p>

<p><strong>Jellyseerr</strong> is a fork of Overseerr by Fallenbagel. It adds Jellyfin and Emby support alongside the original Plex integration. Development is active with regular releases.</p>

<h2>
  
  
  Feature Comparison
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Overseerr</th>
<th>Jellyseerr</th>
</tr>
</thead>
<tbody>
<tr>
<td>Plex integration</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Jellyfin integration</td>
<td>No</td>
<td>Yes</td>
</tr>
<tr>
<td>Emby integration</td>
<td>No</td>
<td>Yes</td>
</tr>
<tr>
<td>Sonarr integration</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Radarr integration</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Request management</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Auto-approval rules</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>User quotas</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>TMDB browsing</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Issue reporting</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Notifications (Discord, Slack, email, etc.)</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Mobile-friendly UI</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<td>Multi-server support</td>
<td>Plex only</td>
<td>Plex, Jellyfin, Emby</td>
</tr>
<tr>
<td>Docker image</td>
<td><code>lscr.io/linuxserver/overseerr</code></td>
<td><code>fallenbagel/jellyseerr</code></td>
</tr>
<tr>
<td>Active development</td>
<td><strong>No — archived</strong></td>
<td>Yes</td>
</tr>
<tr>
<td>License</td>
<td>MIT</td>
<td>MIT</td>
</tr>
<tr>
<td>Last release</td>
<td>2024 (archived)</td>
<td>2026 (active)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Installation Complexity
</h2>

<p>Both are single-container Docker deployments with minimal configuration. The setup process is nearly identical:</p>

<ol>
<li>Deploy the container</li>
<li>Open the web UI</li>
<li>Connect your media server (Plex/Jellyfin/Emby)</li>
<li>Connect Sonarr and Radarr</li>
<li>Configure user access and request rules</li>
</ol>

<p>Jellyseerr has a few more options during setup (choosing between Plex, Jellyfin, or Emby) but the overall complexity is the same.</p>

<h2>
  
  
  Performance and Resource Usage
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Metric</th>
<th>Overseerr</th>
<th>Jellyseerr</th>
</tr>
</thead>
<tbody>
<tr>
<td>RAM (idle)</td>
<td>~150-200 MB</td>
<td>~150-200 MB</td>
</tr>
<tr>
<td>CPU</td>
<td>Low</td>
<td>Low</td>
</tr>
<tr>
<td>Disk</td>
<td>~200 MB</td>
<td>~200 MB</td>
</tr>
<tr>
<td>Database</td>
<td>SQLite (embedded)</td>
<td>SQLite (embedded)</td>
</tr>
<tr>
<td>Port</td>
<td>5055</td>
<td>5055</td>
</tr>
</tbody>
</table></div>

<p>Resource usage is nearly identical. Jellyseerr is based on Overseerr's codebase, so performance characteristics are the same.</p>

<h2>
  
  
  Community and Support
</h2>

<p><strong>Overseerr</strong> had an active community before archival. Documentation still exists but won't be updated. Bug reports and feature requests are closed.</p>

<p><strong>Jellyseerr</strong> has inherited much of Overseerr's community, especially Jellyfin users who were never served by Overseerr. Active GitHub issues, Discord community, and regular releases. The project has strong momentum.</p>

<h2>
  
  
  Use Cases
</h2>

<h3>
  
  
  Choose Jellyseerr If...
</h3>

<ul>
<li>You use Jellyfin or Emby as your media server</li>
<li>You're setting up a new media request system</li>
<li>You want ongoing security updates and bug fixes</li>
<li>You want active community support</li>
</ul>

<h3>
  
  
  Choose Overseerr If...
</h3>

<ul>
<li>You already have it running and it works — but plan to migrate eventually</li>
<li>There is no good reason to install Overseerr today</li>
</ul>

<h2>
  
  
  Migrating from Overseerr to Jellyseerr
</h2>

<p>Jellyseerr can import Overseerr's database:</p>

<ol>
<li>Stop Overseerr</li>
<li>Copy Overseerr's <code>db/</code> directory</li>
<li>Deploy Jellyseerr and point it to the copied database</li>
<li>Re-authenticate your media server connection</li>
<li>Verify existing requests and users carried over</li>
</ol>

<p>The migration preserves request history, user accounts, and configuration.</p>

<h2>
  
  
  Final Verdict
</h2>

<p><strong>Use Jellyseerr.</strong> Overseerr is archived — no security patches, no bug fixes, no new features. Jellyseerr is the maintained successor with broader media server support. Even if you only use Plex, Jellyseerr is the right choice because it's actively developed.</p>

<h2>
  
  
  FAQ
</h2>

<h3>
  
  
  Is Jellyseerr just Overseerr with Jellyfin support?
</h3>

<p>It started that way, but it has diverged. Jellyseerr includes bug fixes, Emby support, and features that were never added to Overseerr. It's the actively maintained version of the codebase.</p>

<h3>
  
  
  Can I use Jellyseerr with Plex only?
</h3>

<p>Yes. Jellyseerr supports Plex, Jellyfin, and Emby. You don't need Jellyfin to use it.</p>

<h3>
  
  
  Is Overseerr safe to keep running?
</h3>

<p>It works, but it won't receive security patches. For a tool that handles user authentication and connects to your media automation stack, running unmaintained software is a risk worth addressing.</p>

<h3>
  
  
  Does Jellyseerr work with Prowlarr?
</h3>

<p>Jellyseerr doesn't interact with Prowlarr directly. It sends requests to Sonarr/Radarr, which use Prowlarr for indexer searches. They're complementary, not competing tools.</p>

<h2>
  
  
  Related
</h2>

<ul>
<li><a href="https://dev.to/apps/overseerr/">How to Self-Host Overseerr with Docker</a></li>
<li><a href="https://dev.to/apps/jellyseerr/">How to Self-Host Jellyseerr with Docker</a></li>
<li><a href="https://dev.to/apps/sonarr/">How to Self-Host Sonarr</a></li>
<li><a href="https://dev.to/apps/radarr/">How to Self-Host Radarr</a></li>
<li><a href="https://dev.to/compare/prowlarr-vs-jackett/">Prowlarr vs Jackett</a></li>
<li><a href="https://dev.to/compare/sonarr-vs-radarr/">Sonarr vs Radarr</a></li>
<li><a href="https://dev.to/apps/jellyfin/">How to Self-Host Jellyfin</a></li>
<li><a href="https://dev.to/compare/jellyfin-vs-plex/">Jellyfin vs Plex</a></li>
<li><a href="https://dev.to/best/download-management/">Best Self-Hosted Download Management</a></li>
</ul>

