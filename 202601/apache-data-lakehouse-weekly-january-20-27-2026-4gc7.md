---
Title: Apache Data Lakehouse Weekly: January 20–27, 2026
Description: 
Author: Alex Merced
Date: 2026-01-27T22:00:25.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Get Data Lakehouse Books:</strong></p>

<ul>
<li><a href="https://drmevn.fyi/tableformatblog" rel="noopener noreferrer">Apache Iceberg: The Definitive Guide</a></li>
<li><a href="https://drmevn.fyi/tableformatblog-62P6t" rel="noopener noreferrer">Apache Polaris: The Defintive Guide</a></li>
<li><a href="https://hubs.la/Q03GfY4f0" rel="noopener noreferrer">Architecting an Apache Iceberg Lakehouse</a></li>
<li><a href="https://www.puppygraph.com/ebooks/apache-iceberg-digest-vol-1" rel="noopener noreferrer">The Apache Iceberg Digest: Vol. 1</a></li>
</ul>

<p><strong>Lakehouse Community:</strong></p>

<ul>
<li><a href="https://www.datalakehousehub.com" rel="noopener noreferrer">Join the Data Lakehouse Community</a></li>
<li><a href="https://lakehouseblogs.com" rel="noopener noreferrer">Data Lakehouse Blog Roll</a></li>
<li><a href="https://osscommunity.com" rel="noopener noreferrer">OSS Community Listings</a></li>
<li><a href="https://developer.dremio.com" rel="noopener noreferrer">Dremio Lakehouse Developer Hub</a></li>
</ul>




<p>The fourth week of January 2026 brings significant technical proposals and continued community momentum across the Apache lakehouse ecosystem. This week featured major architectural discussions around column-level operations in Iceberg, continued graduation progress for Polaris, and the recent releases of Arrow 23.0.0 and Parquet 1.17.0 beginning to see broader adoption.</p>

<h2>
  
  
  Apache Iceberg
</h2>

<p><strong>Efficient Column Updates Proposal Ignites Community Discussion:</strong> Anurag Mantripragada introduced a comprehensive proposal for enabling efficient column-level updates in Iceberg, addressing critical pain points for AI/ML workloads with wide tables containing thousands of columns. The proposal explores both table-format enhancements and Parquet file-format changes to reduce write amplification when adding or updating feature columns, embeddings, or incremental computations. Anton Okolnychyi provided early feedback praising both approaches while noting complexity concerns around keeping row groups aligned in the Iceberg-native approach. Péter Váry and Gábor Szűcs revealed they've been working on similar functionality and submitted an Iceberg Summit talk on the topic, expressing interest in collaboration. The community enthusiasm suggests this could become a major V4 feature. <a href="http://www.mail-archive.com/dev@iceberg.apache.org/msg12454.html" rel="noopener noreferrer">Discussion thread</a> | <a href="https://github.com/apache/iceberg/issues/15146" rel="noopener noreferrer">GitHub Issue #15146</a></p>

<p><strong>Iceberg-Spark Community Sync Successfully Launched:</strong> Following the proposal earlier in January, the first dedicated Iceberg-Spark Community Sync took place on January 20th. The monthly forum provides focused attention to Spark integration topics including sort order reporting, Spark 4.1 support, and Datafusion-Comet integration—discussions that had grown too numerous for the main community sync. This specialized forum reflects the depth of Spark-specific development while keeping the main community sync focused on broader cross-engine discussions.</p>

<p><strong>Atlanta Meetup Continues Grassroots Community Building:</strong> The Apache Iceberg Meetup ATL held its first event of 2026 on January 21st, maintaining the grassroots community engagement momentum established throughout 2025. Organizers are developing a structured CFP process to encourage diverse presenters and topics throughout the year, demonstrating how local meetups complement the global summit and online syncs in building a vibrant Iceberg community.</p>

<p><strong>Summit CFP Selection Underway:</strong> With the January 18th submission deadline now passed, the selection committee led by Russell Spitzer has begun reviewing proposals for the April 8-9 Iceberg Summit in San Francisco. The community is anticipating strong participation following the over 200 submissions received for the previous summit.</p>

<h2>
  
  
  Apache Polaris
</h2>

<p><strong>Generic Table Feature Approaching Stable Status:</strong> Following the 1.3.0-incubating release earlier in January, the Polaris community is preparing to graduate the "Generic Table" capability from beta to stable status. This feature enables Polaris to catalog external table formats like Apache Hudi and Delta Lake, significantly expanding Polaris's value proposition as a multi-format catalog beyond Iceberg-only support.</p>

<p><strong>Cloud Integration Testing Expansion Progresses:</strong> With AWS credits now available to the project, contributors are actively expanding integration testing against real cloud infrastructure. The focus includes IAM AssumeRole flows and credential vending scenarios that are difficult to simulate locally, improving production-readiness validation for enterprise deployments on AWS, Azure, and Google Cloud platforms.</p>

<p><strong>Graduation Momentum Continues:</strong> Regular community syncs and development sprints maintained their cadence through the third week of January, with the expanding Podling Project Management Committee (PPMC) reflecting healthy governance maturation. Multiple contributors continue stepping into leadership roles as the project advances toward full Apache graduation.</p>

<h2>
  
  
  Apache Arrow
</h2>

<p><strong>23.0.0 Release Adoption Beginning:</strong> Following the January 18th release of Apache Arrow 23.0.0, the community is beginning to see adoption of the major release covering over three months of development with 417 commits from 71 distinct contributors. The release includes improvements across C++, Python, Java, R, and Go implementations, maintaining Arrow's role as the universal columnar interchange format.</p>

<p><strong>Multi-Language Consistency Maintained:</strong> The 23.0.0 release emphasizes keeping Arrow's implementations consistent across languages, ensuring seamless data interchange whether teams use PyArrow, Arrow Java, Arrow C++, or Arrow R. This consistency remains critical for the lakehouse ecosystem where different query engines and tools need to exchange data efficiently without copying or re-encoding.</p>

<p><strong>Leadership Stability Reinforced:</strong> With Antoine Pitrou, one of Arrow's co-creators, formally serving as PMC Chair, the project enters 2026 with stable leadership providing consistent technical vision. This governance continuity supports Arrow's expansion of language bindings and compute capabilities while maintaining format stability.</p>

<h2>
  
  
  Apache Parquet
</h2>

<p><strong>1.17.0 Release Sees Initial Adoption:</strong> Following the official January 13th release of Apache Parquet 1.17.0, early adopters are beginning the transition to the modernized version that sets Java 11 as the new minimum runtime requirement. This represents a significant modernization milestone as the community drops Java 8 support, aligning Parquet with contemporary Java library standards.</p>

<p><strong>Java 11 Migration Impact:</strong> The move to Java 11 minimum requirement signals that users still on legacy Java versions should plan upgrades. This modernization parallels discussions in the Iceberg community around moving to Java 17, reflecting a broader trend toward contemporary Java platforms across the lakehouse stack and enabling the use of modern language features in future development.</p>

<p><strong>V3 Format Groundwork in Early Stages:</strong> While no concrete proposals have emerged, informal discussions continue about what a Parquet V3 format might include. Topics under consideration include improved metadata layouts, enhanced bloom filter indexing, and cloud-native encoding optimizations—all aimed at reducing scan times without breaking backward compatibility.</p>

<h2>
  
  
  Cross-Project Themes
</h2>

<p>The lakehouse ecosystem demonstrates strong coordination as 2026's development cycle accelerates. The synchronized Java modernization across Iceberg and Parquet—with Parquet completing its Java 11 transition and Iceberg discussions pointing toward Java 17—enables cleaner dependency management and access to modern language features. Community infrastructure investments span from Iceberg's Summit and specialized syncs to Polaris's expanded testing infrastructure, translating technical discussions into practical implementation guidance. The focus on column-level operations in Iceberg, combined with Arrow's multi-language consistency and Parquet's format evolution discussions, positions the stack for increasingly sophisticated AI/ML workloads requiring efficient feature engineering and embedding management at scale.</p>

<h2>
  
  
  Looking Ahead
</h2>

<p>The coming weeks will see the Iceberg selection committee announce accepted Summit talks, continued refinement of the efficient column updates proposal, and broader adoption of the Arrow 23.0.0 and Parquet 1.17.0 releases. Polaris's Generic Table feature stabilization and expanded integration testing will advance the project's graduation timeline. As the lakehouse ecosystem matures, the community's emphasis on production-grade reliability, efficient wide-table operations, and coordinated modernization ensures these open standards remain the foundation for next-generation analytics platforms.</p>

