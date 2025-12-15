---
Title: Apache Dev List Digest: Iceberg, Polaris, Arrow & Parquet (Dec 9th - Dec15th, 2025)
Description: 
Author: Alex Merced
Date: 2025-12-15T21:47:40.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Get Data Lakehouse Books:</strong></p>

<ul>
<li><a href="https://drmevn.fyi/tableformatblog" rel="noopener noreferrer">Apache Iceberg: The Definitive Guide</a></li>
<li><a href="https://drmevn.fyi/tableformatblog-62P6t" rel="noopener noreferrer">Apache Polaris: The Defintive Guide</a></li>
<li><a href="https://www.manning.com/books/architecting-an-apache-iceberg-lakehouse" rel="noopener noreferrer">Architecting an Apache Iceberg Lakehouse</a></li>
<li><a href="https://www.puppygraph.com/ebooks/apache-iceberg-digest-vol-1" rel="noopener noreferrer">The Apache Iceberg Digest: Vol. 1</a></li>
</ul>

<p><strong>Lakehouse Community:</strong></p>

<ul>
<li><a href="https://www.datalakehousehub.com" rel="noopener noreferrer">Join the Data Lakehouse Community</a></li>
<li><a href="https://lakehouseblogs.com" rel="noopener noreferrer">Data Lakehouse Blog Roll</a></li>
<li><a href="https://osscommunity.com" rel="noopener noreferrer">OSS Community Listings</a></li>
<li><a href="https://developer.dremio.com" rel="noopener noreferrer">Dremio Lakehouse Developer Hub</a></li>
</ul>




<p>The week of December 9 to 15, 2025 saw brisk activity across the developer mailing lists of Apache Iceberg, Polaris, Arrow, and Parquet. This digest highlights notable release votes, design proposals, and community news from each project, keeping data engineers and decision makers up to speed with the latest developments in the open data lakehouse ecosystem:contentReference[oaicite:0]{index=0}.</p>

<h2>
  
  
  Apache Iceberg
</h2>

<h3>
  
  
  Patch Release 1.10.1 in Progress
</h3>

<p>Apache Iceberg is preparing a 1.10.1 patch release. The initial RC0 vote was canceled after licensing issues were identified in certain module jars:contentReference[oaicite:1]{index=1}. A new release candidate (RC1) was proposed on December 14 and is currently under vote as the official 1.10.1 release:contentReference[oaicite:2]{index=2}. This minor version aims to deliver critical bug fixes and stability improvements for the 1.10.x branch:contentReference[oaicite:3]{index=3}. </p>

<ul>
<li>
<strong>Impact:</strong> Once approved, the 1.10.1 update will provide production users with important fixes (e.g. dependency license corrections and runtime stability tweaks), ensuring a smoother experience on the latest Iceberg 1.x line:contentReference[oaicite:4]{index=4}:contentReference[oaicite:5]{index=5}.</li>
</ul>

<h3>
  
  
  December Board Report &amp; Community Updates
</h3>

<p>The Iceberg team shared a draft of its December board report on the dev mailing list:contentReference[oaicite:6]{index=6}. This quarterly report outlines recent progress (such as the pending 1.10.1 release and Format V4 design work) and celebrates community growth. Notably, it congratulates several new committers and PMC members who have been added to the project in the last quarter:contentReference[oaicite:7]{index=7} – a positive sign of an expanding contributor base. <em>(The report helps Apache leadership oversee Iceberg’s health and momentum.)</em> </p>

<h3>
  
  
  Iceberg Summit 2026 – Call for Papers
</h3>

<p>An <strong>[ANNOUNCE]</strong> went out inviting talk submissions for the inaugural <strong>Apache Iceberg Summit 2026</strong>:contentReference[oaicite:8]{index=8}. The call for papers (CFP) is now open (closing Jan 18, 2026) for speakers to propose sessions. This dedicated Iceberg conference highlights the project’s growing ecosystem and user community. </p>

<ul>
<li>
<strong>Why it matters:</strong> Having a standalone Iceberg Summit underlines the project’s maturation and broad interest. It offers a forum for sharing real-world use cases, integrations, and best practices – valuable for both engineers and decision-makers evaluating Iceberg.</li>
</ul>

<h3>
  
  
  Multi-Table Transactions Planning
</h3>

<p>Design discussions continue around supporting multi-statement transactions spanning multiple tables in Iceberg. Contributors are exploring an “Interactive Commit” API approach to coordinate atomic changes across tables (useful for complex ETL or schema evolution scenarios). A dedicated community sync call was scheduled for Dec 18 to refine the proposal and gather feedback on this multi-table transaction design:contentReference[oaicite:9]{index=9}.</p>

<ul>
<li>
<strong>Use Case:</strong> If implemented, this feature would enable atomic commits across multiple Iceberg tables (for example, updating a fact table and its derived aggregate table together). This is important for data workflows that require all-or-nothing updates across datasets to maintain consistency.</li>
</ul>

<h2>
  
  
  Apache Polaris
</h2>

<h3>
  
  
  1.3.0-incubating Released
</h3>

<p>Apache Polaris (incubating) has officially shipped version <strong>1.3.0-incubating</strong> after passing the community vote on its release candidate. This release includes several enhancements:</p>

<ul>
<li>
<strong>Generic Table GA:</strong> The “generic table” feature is now production-ready, enabling Polaris to catalog external table formats (e.g. Apache Hudi, Delta Lake) in a stable way:contentReference[oaicite:10]{index=10}.
</li>
<li>
<strong>Cloud Integration Improvements:</strong> Strengthened integration tests and fixes for cloud environments (improving reliability on platforms like AWS/Azure):contentReference[oaicite:11]{index=11}.
</li>
<li>
<strong>Bug Fixes:</strong> Numerous bug fixes and reliability enhancements to core services.
</li>
</ul>

<p><em>(As an incubating project, Polaris required IPMC approval for the release, which has been secured. This milestone brings Polaris closer to a mature state on its path to Apache graduation.)</em></p>

<h3>
  
  
  New Backend: Google Spanner Support Proposed
</h3>

<p>A discussion was started to add <strong>Google Cloud Spanner</strong> as a backend option for Polaris’s catalog/storage layer:contentReference[oaicite:12]{index=12}. In the proposal, a contributor outlined plans to integrate Spanner, Google’s globally distributed SQL database, as a supported metadata store or transaction backend for Polaris. </p>

<ul>
<li>
<strong>Benefit:</strong> Spanner integration would broaden Polaris’s multi-cloud story beyond Hadoop/Hive metastores, making it easier to deploy Polaris in GCP environments. This reflects the community’s goal to support a wide range of data stores (SQL and NoSQL) under the Polaris catalog abstraction.</li>
</ul>

<h3>
  
  
  Incubator Report &amp; Community Growth
</h3>

<p>The Polaris podling prepared its December <strong>Incubator report</strong>, highlighting recent progress and community momentum:contentReference[oaicite:13]{index=13}. The report notes the successful 1.3.0 release and ongoing efforts like improved release automation. It also helps mentors track Polaris’s readiness for graduation. In community news, <strong>Yong Zheng</strong> was welcomed as a new member of the Polaris PPMC (Podling Project Management Committee):contentReference[oaicite:14]{index=14}. This addition to the project’s leadership indicates growing contributor engagement and trust. </p>

<ul>
<li>
<strong>Community note:</strong> Regular community sync meetings continued (one was held on Dec 11), and a virtual development sprint was scheduled for Dec 16 to focus on onboarding, docs, and open issues. These activities show an active, organized community coalescing around Polaris’s roadmap.</li>
</ul>

<h2>
  
  
  Apache Arrow
</h2>

<h3>
  
  
  Arrow Go 18.5.0 Released
</h3>

<p>The Arrow Go subproject released version <strong>18.5.0</strong>, a minor update comprising 38 commits from 17 contributors . The vote passed cleanly, and the release was officially announced on Dec 12.</p>

<ul>
<li>
<strong>Why it matters:</strong> The Go implementation is widely used in lightweight analytics and ETL pipelines. These regular updates help keep Arrow’s Go support on par with its C++, Rust, and Java siblings, ensuring multi-language consistency.</li>
</ul>

<p><a href="https://lists.apache.org/thread/pfvnrr6sdqphvd1mt77v6rg5x2hl39qc" rel="noopener noreferrer">Release vote thread</a></p>

<h3>
  
  
  Arrow Java 20.0.0 – Delayed for IP Clearance
</h3>

<p>Apache Arrow Java’s proposed 20.0.0 release was temporarily paused due to an IP clearance requirement involving a third-party contribution . While the community supported the release in principle, Apache policy mandates formal review before including new dependencies.</p>

<ul>
<li>
<strong>Takeaway:</strong> This demonstrates the ASF’s commitment to license transparency and compliance. Once clearance is approved, Arrow Java 20.0.0 is expected to proceed, offering new features and performance optimizations for Java-based analytics engines.</li>
</ul>

<h3>
  
  
  New TimestampWithOffset Type Approved
</h3>

<p>The Arrow format spec was officially extended to include a new <strong>TimestampWithOffset</strong> type . This change provides a standardized way to represent timestamps with time zone offsets — eliminating ambiguity when sharing or serializing time-based data.</p>

<ul>
<li>
<strong>Impact:</strong> This addition enhances time zone correctness across engines like PyArrow, Arrow Java, and DuckDB.</li>
</ul>

<h3>
  
  
  128-bit Timestamps Discussion (Still Open)
</h3>

<p>A separate discussion explored introducing <strong>128-bit timestamps</strong> (at picosecond precision) for scientific and high-frequency use cases . While the community remains divided on its necessity and performance tradeoffs, the topic reflects Arrow’s readiness to push format boundaries for niche workloads.</p>

<h3>
  
  
  Arrow Rust – Optimization Work
</h3>

<p>Apache Arrow Rust v57.1.0 was released earlier in the month, and contributors began discussing <strong>performance improvements to bitwise operations</strong>. These micro-optimizations are particularly relevant to projects like DataFusion that rely on Arrow Rust for vectorized computation.</p>

<ul>
<li>
<strong>Takeaway:</strong> Arrow Rust continues to grow as a backend engine choice, and contributors are proactively refining its compute kernels.</li>
</ul>

<h3>
  
  
  Governance – New PMC Chair
</h3>

<p>Antoine Pitrou, co-creator of Apache Arrow, was formally appointed <strong>PMC Chair</strong> of the project . This leadership transition reflects stability in the project’s governance and continued vision from its founding contributors.</p>

<h2>
  
  
  Apache Parquet
</h2>

<h3>
  
  
  String Column Optimization – FSST Proposal
</h3>

<p>The community continued design work on enhancing string and byte array encoding using <strong>FSST (Finite State Symbol Table)</strong> compression . Contributors discussed how to bundle or link to FSST as a CMake dependency.</p>

<ul>
<li>
<strong>Why it matters:</strong> Sharing a single compressed dictionary across multiple column pages can reduce size and improve scan speed for string-heavy data. These changes may appear in future versions of Parquet MR or Arrow-native readers.</li>
</ul>

<h3>
  
  
  ☕ Java 1.17.0 Prep – Goodbye Java 8
</h3>

<p>Preparations began for the next <strong>Parquet Java 1.17.0</strong> release. The community agreed to drop support for Java 8 and set <strong>Java 11</strong> as the new minimum runtime . This aligns Parquet with broader trends in open source Java libraries, allowing use of modern language features and better dependency hygiene.</p>

<ul>
<li>
<strong>Impact:</strong> Parquet users running older JVMs will need to upgrade; the trade-off is improved stability and modern build tooling.</li>
</ul>

<h3>
  
  
  Deprecating <code>file_path</code> Metadata
</h3>

<p>An agreement was reached to deprecate the <code>file_path</code> field in Parquet column chunk metadata . This legacy field is no longer needed in modern workflows and often ignored by consumers.</p>

<ul>
<li>
<strong>Takeaway:</strong> Cleaning up metadata reduces confusion and improves format clarity for tool authors and data engineers working with the spec.</li>
</ul>

<h3>
  
  
  Looking Ahead – Parquet Format v3?
</h3>

<p>Though unofficial, developers hinted at beginning scoping discussions for <strong>Parquet format v3</strong> after wrapping up the last format v2 features (like page checksums and bloom filters) .</p>

<ul>
<li>
<strong>Why it matters:</strong> A v3 format could bring larger architectural updates such as column type improvements, cloud-native encodings, or better streaming support. No concrete proposal has landed, but signs point to 2026 as the year v3 takes shape.</li>
</ul>

<h2>
  
  
  Wrap-Up
</h2>

<p>Across the board, the Apache lakehouse ecosystem showed strong end-of-year momentum:</p>

<ul>
<li>
<strong>Iceberg</strong> is stabilizing its 1.10.x branch while exploring advanced features like multi-table transactions and community expansion.</li>
<li>
<strong>Polaris</strong> delivered a solid new release, expanded its backend ambitions (Spanner), and continues maturing toward complete Apache graduation.</li>
<li>
<strong>Arrow</strong> pushed significant format enhancements (e.g. timezone support), released Go updates, and is planning future language-specific improvements.</li>
<li>
<strong>Parquet</strong> is finalizing v2-era cleanup while laying early groundwork for a v3 evolution.</li>
</ul>

<p>As these projects close out 2025, they continue to evolve in capability, governance, and adoption — ensuring the open data lakehouse remains a fast-moving, production-ready architecture for modern analytics.</p>

