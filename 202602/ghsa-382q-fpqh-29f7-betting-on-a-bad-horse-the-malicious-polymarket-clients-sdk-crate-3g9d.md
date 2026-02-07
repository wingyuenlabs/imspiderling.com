---
Title: GHSA-382Q-FPQH-29F7: Betting on a Bad Horse: The Malicious `polymarket-clients-sdk` Crate
Description: 
Author: CVE Reports
Date: 2026-02-07T21:10:05.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Betting on a Bad Horse: The Malicious <code>polymarket-clients-sdk</code> Crate
</h1>

<blockquote>
<p><strong>Vulnerability ID:</strong> GHSA-382Q-FPQH-29F7<br>
<strong>CVSS Score:</strong> 10.0<br>
<strong>Published:</strong> 2026-02-06</p>
</blockquote>

<p>A classic supply chain attack targeting the Rust ecosystem. The package <code>polymarket-clients-sdk</code> appeared on crates.io, masquerading as an official SDK for the popular Polymarket prediction platform. Instead of helper functions for betting, it delivered a payload capable of exfiltrating credentials and compromising developer environments via malicious build scripts. This is a text-book example of Brandjacking combined with the inherent risks of arbitrary code execution during package installation.</p>

<h2>
  
  
  TL;DR
</h2>

<p>A malicious Rust crate impersonating the Polymarket SDK was discovered on crates.io. It executes arbitrary code immediately upon compilation, likely stealing environment variables and private keys. If you installed this, consider your machine compromised and your secrets stolen.</p>




<h3>
  
  
  ⚠️ Exploit Status: ACTIVE
</h3>

<h2>
  
  
  Technical Details
</h2>

<ul>
<li>
<strong>Attack Type</strong>: Supply Chain / Malicious Package</li>
<li>
<strong>CWE ID</strong>: CWE-506 (Embedded Malicious Code)</li>
<li>
<strong>Platform</strong>: Rust / crates.io</li>
<li>
<strong>Attack Vector</strong>: Network (masquerading as legitimate software)</li>
<li>
<strong>Mechanism</strong>: build.rs arbitrary code execution</li>
<li>
<strong>Privileges</strong>: User Level (inherits developer permissions)</li>
</ul>

<h2>
  
  
  Affected Systems
</h2>

<ul>
<li>Rust Development Environments</li>
<li>CI/CD Pipelines building Rust projects</li>
<li>Systems with crates.io access</li>
<li>
<strong>polymarket-clients-sdk</strong>: * (Fixed in: <code>N/A (Removed)</code>)</li>
</ul>

<h2>
  
  
  Mitigation Strategies
</h2>

<ul>
<li>Vendor verification using cargo-vet</li>
<li>Sandboxed build environments (Docker/Firejail)</li>
<li>Dependency pinning via Cargo.lock</li>
<li>Network restrictions during build time</li>
</ul>

<p><strong>Remediation Steps:</strong></p>

<ol>
<li>Identify projects containing 'polymarket-clients-sdk' in Cargo.toml</li>
<li>Remove the dependency immediately</li>
<li>Delete the 'target' directory and Cargo.lock</li>
<li>Assume full system compromise: Rotate all secrets, keys, and environment variables exposed to the machine</li>
<li>Rebuild the host system or container from a clean state</li>
</ol>

<h2>
  
  
  References
</h2>

<ul>
<li><a href="https://github.com/advisories/GHSA-382Q-FPQH-29F7" rel="noopener noreferrer">GHSA-382Q-FPQH-29F7 Advisory</a></li>
<li><a href="https://doc.rust-lang.org/cargo/reference/build-scripts.html" rel="noopener noreferrer">Rust Documentation: Build Scripts</a></li>
</ul>




<p><em><a href="https://cvereports.com/reports/GHSA-382Q-FPQH-29F7" rel="noopener noreferrer">Read the full report for GHSA-382Q-FPQH-29F7 on our website</a> for more details including interactive diagrams and full exploit analysis.</em></p>

