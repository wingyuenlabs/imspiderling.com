---
Title: Duplicati vs Borgmatic: Which Backup Tool?
Description: 
Author: selfhosting.sh
Date: 2026-02-26T22:11:16.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Quick Verdict
</h2>

<p>Borgmatic is the better backup tool for most self-hosters. Its deduplication is genuinely superior, it handles large datasets faster, and its YAML config files are version-controllable and reproducible. Duplicati wins if you need a web UI or native cloud storage support (S3, B2, Google Drive) without extra tooling. If you are comfortable with the command line, pick Borgmatic. If you want to click through a GUI and back up directly to cloud storage, pick Duplicati.</p>

<h2>
  
  
  Overview
</h2>

<p>Duplicati is a GUI-first backup tool written in C#/.NET. It runs a web interface on port 8200 where you configure backup jobs, select sources, choose destinations, and set schedules. It supports a wide range of cloud backends natively — Amazon S3, Backblaze B2, Google Drive, OneDrive, SFTP, and more. Encryption uses AES-256, and backups are incremental and block-based.</p>

<p>Borgmatic is a Python wrapper around BorgBackup. It replaces Borg's verbose CLI with a clean YAML configuration file. You define your source directories, repositories, retention policies, and hooks in a single <code>config.yaml</code>, then borgmatic handles the rest — creating archives, pruning old ones, and running consistency checks. Borg's deduplication engine is content-defined chunking, which means identical data across different files or backups is stored only once. Remote backups go over SSH to any server running Borg.</p>

<p>Both tools run well in Docker. Duplicati uses the LinuxServer.io image (<code>lscr.io/linuxserver/duplicati</code>), while Borgmatic uses the official image from the borgmatic-collective (<code>ghcr.io/borgmatic-collective/borgmatic</code>). Both can be scheduled — Duplicati through its web UI scheduler, Borgmatic through cron (typically configured inside the container).</p>

<h2>
  
  
  Feature Comparison
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Duplicati</th>
<th>Borgmatic</th>
</tr>
</thead>
<tbody>
<tr>
<td>Interface</td>
<td>Web UI (port 8200)</td>
<td>CLI + YAML config</td>
</tr>
<tr>
<td>Underlying engine</td>
<td>Custom C#/.NET</td>
<td>BorgBackup (Python/C)</td>
</tr>
<tr>
<td>Deduplication</td>
<td>Block-level incremental</td>
<td>Content-defined chunking (superior)</td>
</tr>
<tr>
<td>Encryption</td>
<td>AES-256</td>
<td>AES-256-CTR (via Borg)</td>
</tr>
<tr>
<td>Compression</td>
<td>Zip, 7z, no compression</td>
<td>LZ4, zstd, lzma, none</td>
</tr>
<tr>
<td>Cloud backends</td>
<td>S3, B2, Google Drive, OneDrive, SFTP, WebDAV, 20+</td>
<td>SSH/SFTP (native); cloud via rclone mount</td>
</tr>
<tr>
<td>Remote protocol</td>
<td>Varies by backend</td>
<td>SSH with Borg server</td>
</tr>
<tr>
<td>Scheduling</td>
<td>Built-in UI scheduler</td>
<td>Cron (inside or outside container)</td>
</tr>
<tr>
<td>Restore method</td>
<td>Web UI or CLI</td>
<td>CLI (<code>borgmatic extract</code>)</td>
</tr>
<tr>
<td>Configuration</td>
<td>Database + web UI settings</td>
<td>Single YAML file (version-controllable)</td>
</tr>
<tr>
<td>Docker image</td>
<td><code>lscr.io/linuxserver/duplicati</code></td>
<td><code>ghcr.io/borgmatic-collective/borgmatic</code></td>
</tr>
<tr>
<td>Cross-platform</td>
<td>Windows, macOS, Linux</td>
<td>Linux, macOS (Borg requirement)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Installation Complexity
</h2>

<p><strong>Duplicati</strong> is simpler to get running. Pull the LinuxServer.io image, map your volumes, and open the web UI. Everything is configured through the browser — source folders, destination, schedule, encryption passphrase, retention. No config files to write. A complete Docker Compose setup:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">duplicati</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">lscr.io/linuxserver/duplicati:2.0.9.3</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">duplicati</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">PUID=1000</span>
      <span class="pi">-</span> <span class="s">PGID=1000</span>
      <span class="pi">-</span> <span class="s">TZ=America/New_York</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">duplicati-config:/config</span>
      <span class="pi">-</span> <span class="s">/path/to/backups:/backups</span>
      <span class="pi">-</span> <span class="s">/path/to/source:/source:ro</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8200:8200"</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">duplicati-config</span><span class="pi">:</span>
</code></pre>

</div>



<p>Start it, open <code>http://your-server:8200</code>, and click through the wizard to create your first backup job.</p>

<p><strong>Borgmatic</strong> requires writing a YAML configuration file before anything runs. You need to define repositories, source directories, retention policy, and optionally encryption and hooks. The Docker setup mounts this config file and your SSH keys (for remote repos) into the container:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">services</span><span class="pi">:</span>
  <span class="na">borgmatic</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">ghcr.io/borgmatic-collective/borgmatic:1.9.5</span>
    <span class="na">container_name</span><span class="pi">:</span> <span class="s">borgmatic</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">TZ=America/New_York</span>
      <span class="pi">-</span> <span class="s">BORG_PASSPHRASE=your-strong-passphrase-here</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">./borgmatic/config.yaml:/etc/borgmatic/config.yaml:ro</span>
      <span class="pi">-</span> <span class="s">./borgmatic/ssh:/root/.ssh:ro</span>
      <span class="pi">-</span> <span class="s">borg-cache:/root/.cache/borg</span>
      <span class="pi">-</span> <span class="s">/path/to/source:/source:ro</span>
      <span class="pi">-</span> <span class="s">/path/to/local-repo:/repository</span>
    <span class="na">restart</span><span class="pi">:</span> <span class="s">unless-stopped</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">borg-cache</span><span class="pi">:</span>
</code></pre>

</div>



<p>You also need to initialize the Borg repository before the first backup (<code>borgmatic init --encryption repokey-blake2</code>), and set up cron inside the container or use the image's built-in cron support. More steps, but the result is a fully declarative, reproducible setup.</p>

<p><strong>Winner: Duplicati</strong> for ease of setup. If you have never configured backups before, Duplicati's web UI walks you through every step. Borgmatic assumes you know what a Borg repository is and how SSH keys work.</p>

<h2>
  
  
  Performance and Resource Usage
</h2>

<p>This is where Borgmatic pulls ahead decisively.</p>

<p><strong>Deduplication efficiency.</strong> Borg uses content-defined chunking (CDC), which means it splits files into variable-size chunks based on content boundaries. When a file changes, only the modified chunks are stored. Duplicati uses fixed-block incremental backups, which is less efficient — small insertions at the beginning of a file can cause most blocks to shift, resulting in more data stored. For large, frequently-changing datasets (databases, mail stores, VM images), Borg's deduplication can use 30-50% less storage than Duplicati over time.</p>

<p><strong>Backup speed.</strong> Borgmatic/Borg is faster on subsequent backups because its deduplication index makes it efficient at identifying unchanged data. Duplicati can be slower on large datasets because its block-based comparison and database operations add overhead. Users with multi-terabyte backup sets consistently report Borgmatic completing in a fraction of the time Duplicati takes.</p>

<p><strong>Resource usage.</strong> Duplicati's web UI and C#/.NET runtime consume more RAM at idle (typically 200-400 MB). Borgmatic is lighter — Borg itself uses modest memory during backups (100-300 MB depending on dataset size) and nothing between runs since it is cron-triggered, not a persistent daemon.</p>

<p><strong>Restore speed.</strong> Borgmatic restores are fast because Borg stores deduplicated chunks that can be reassembled quickly. Duplicati restores from cloud backends can be slow due to the number of small encrypted files it creates and the reassembly process.</p>

<p><strong>Winner: Borgmatic.</strong> Superior deduplication, faster backups on large datasets, lower resource usage. Duplicati is adequate for small backup sets (under 500 GB) but struggles at scale.</p>

<h2>
  
  
  Community and Support
</h2>

<p><strong>Duplicati</strong> has a large user base, particularly among Windows and cross-platform users. The project has been in development since 2008, though the 2.x version has been in beta for years (a common criticism). The forum is active, and the GUI makes it accessible to a broader audience. GitHub activity has slowed in recent years, with long gaps between releases.</p>

<ul>
<li>GitHub stars: ~11,000+</li>
<li>Forum: Active community forum</li>
<li>Docker pulls: High (via LinuxServer.io)</li>
<li>Release cadence: Irregular; 2.x has been "beta" for a long time</li>
<li>Documentation: Decent, mostly community-contributed</li>
</ul>

<p><strong>Borgmatic</strong> benefits from BorgBackup's strong reputation in the Linux sysadmin community. Borg itself is mature, well-audited, and actively maintained. Borgmatic adds a user-friendly configuration layer on top. The borgmatic documentation is excellent — clear, comprehensive, and well-organized.</p>

<ul>
<li>GitHub stars: ~7,000+ (borgmatic) + ~11,000+ (BorgBackup)</li>
<li>Community: Strong in Linux/sysadmin circles, Borg mailing list</li>
<li>Docker pulls: Growing (official borgmatic-collective image)</li>
<li>Release cadence: Regular updates for both borgmatic and Borg</li>
<li>Documentation: Excellent (borgmatic docs are a model of clarity)</li>
</ul>

<p><strong>Winner: Borgmatic.</strong> More actively maintained, better documentation, and backed by the rock-solid BorgBackup project. Duplicati's perpetual beta status is a concern for a tool you trust with your data.</p>

<h2>
  
  
  Use Cases
</h2>

<h3>
  
  
  Choose Duplicati If...
</h3>

<ul>
<li>You want a web UI for configuring and monitoring backups</li>
<li>You need to back up directly to cloud storage (S3, B2, Google Drive, OneDrive) without additional tools</li>
<li>You are on Windows or need cross-platform support</li>
<li>You are new to self-hosted backups and want a guided setup experience</li>
<li>Your backup set is under 500 GB and you value convenience over performance</li>
<li>You want to restore individual files through a browser interface</li>
</ul>

<h3>
  
  
  Choose Borgmatic If...
</h3>

<ul>
<li>You are comfortable with the command line and YAML configuration</li>
<li>You have large datasets (1 TB+) where deduplication efficiency matters</li>
<li>You want version-controllable, reproducible backup configurations</li>
<li>You back up to a remote server over SSH (a common self-hosting pattern)</li>
<li>You want the best deduplication and compression available</li>
<li>You value stability and active maintenance over GUI convenience</li>
<li>You run Linux servers and want a tool built for that environment</li>
<li>You want to integrate backups with other automation (hooks for pre/post backup scripts)</li>
</ul>

<h2>
  
  
  Final Verdict
</h2>

<p><strong>Borgmatic is the better choice for self-hosters.</strong> The typical self-hosting setup is a Linux server backing up to another Linux server (or a local repository), and that is exactly what Borgmatic is built for. Borg's deduplication is best-in-class, the YAML config is clean and auditable, and the project is actively maintained by people who care about data integrity.</p>

<p>Duplicati is not a bad tool — its cloud backend support is genuinely useful if you need to push backups to Google Drive or Backblaze B2 without setting up rclone. And the web UI removes friction for users who do not want to write config files. But the perpetual beta status, slower performance on large datasets, and higher resource usage make it the weaker choice for serious self-hosted infrastructure.</p>

<p>If you are managing a homelab with Docker containers and want reliable, efficient, automated backups — set up Borgmatic, write a <code>config.yaml</code>, and forget about it. Your data will be safer for it.</p>

<h2>
  
  
  Related
</h2>

<ul>
<li><a href="https://dev.to/apps/duplicati/">How to Self-Host Duplicati</a></li>
<li><a href="https://dev.to/apps/borgmatic/">How to Self-Host Borgmatic</a></li>
<li><a href="https://dev.to/apps/restic/">How to Self-Host Restic</a></li>
<li><a href="https://dev.to/compare/restic-vs-borgbackup/">Restic vs BorgBackup</a></li>
<li><a href="https://dev.to/foundations/backup-3-2-1-rule/">Backup Strategy: The 3-2-1 Rule</a></li>
<li><a href="https://dev.to/best/backup/">Best Self-Hosted Backup Solutions</a></li>
<li><a href="https://dev.to/foundations/docker-compose-basics/">Docker Compose Basics</a></li>
</ul>

