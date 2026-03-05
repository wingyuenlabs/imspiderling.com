---
Title: Introducing Hive Index: A real-time custom json operations explorer for Hive
Description: 
Author: Serpent7776
Date: 2026-03-05T21:35:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>I'd like to share <strong>Hive Index</strong>, a web UI for exploring Hive blockchain activity with clear, live stats, built in Rust 🦀.</p>

<p>It runs as a single binary, continuously pulling Hive blocks and updating its internal database with metadata about the blocks and operations.</p>

<p>Currently, it aggregates <code>custom_json</code> operations and shows ranked stats for them.</p>

<p>If I read the data correctly, the top 10 is dominated by Splinterlands, but there's also <code>follow</code> operation at #7.</p>

<p>The data was synced starting at block 7,000,000 (it doesn't seem there was anything interesting before that). The current database weight is about 9GB.</p>

<p>If you're interested in trying it out, head to <a href="https://index.hive-adventures.com/" rel="noopener noreferrer">https://index.hive-adventures.com/</a></p>

<p>Let me know if it's useful to you and if you have any questions or suggestions!</p>

