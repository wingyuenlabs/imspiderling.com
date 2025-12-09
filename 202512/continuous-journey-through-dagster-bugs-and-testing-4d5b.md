---
Title: Continuous Journey through Dagster - bugs and testing
Description: 
Author: Steven Hur
Date: 2025-12-09T21:25:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>Lately, I've been diving deep into open-source contributions for <code>Dagster</code>. I think I am getting bit more comfortable with their codebases which hastened my working process(placebo?). Today, I want to share the issues I've tackled recently and talk about a significant roadblock I'm currently facing.</p>

<p><strong>My Recent Contributions</strong><br>
I focused on fixing several bugs and improving stability across different parts of the <code>Dagster</code>. Here is a breakdown of the issues I worked on:</p>

<ol>
<li>Fixing ECS Pipes Client Execution</li>
</ol>

<p>The Issue: Users were encountering an <code>IndexError</code> when launching tasks using the <code>PipesECSClient</code>. This caused pipelines to crash unexpectedly in ECS environments.</p>

<p>The Fix: I added proper exception handling and bounds checking to ensure the client launches tasks smoothly without crashing on index errors.</p>

<p><a href="https://github.com/dagster-io/dagster/issues/32936" rel="noopener noreferrer">Issue #32936</a></p>

<ol>
<li>Resolving Asset Specs Mapping Dependencies</li>
</ol>

<p>The Issue: There was a logic error in <code>AssetsDefinition.map_asset_specs</code> that caused failures when attempting to add dependencies while input definitions were already set.</p>

<p>The Fix: I adjusted the core logic to correctly handle the mapping of asset specs even when inputs are pre-configured.</p>

<p><a href="https://github.com/dagster-io/dagster/issues/32913" rel="noopener noreferrer">Issue #32913</a></p>

<ol>
<li>[WIP]Correcting Asset Sensor Event Processing</li>
</ol>

<p>The Issue: The <code>asset_sensor</code> had a critical bug where it would only process the last materialization event if multiple partitions materialized simultaneously. This issue stems from a <code>race condition</code>, making it notoriously difficult to reproduce and debug in a local environment.</p>

<p>The Fix: Still working in progress but initially, I modified the sensor logic to ensure every single materialization event is captured and processed, regardless of concurrency. Precise approach with careful testing is required for further progress.</p>

<p><a href="https://github.com/dagster-io/dagster/issues/32853" rel="noopener noreferrer">Issue #32853</a></p>

<ol>
<li>[WIP]Implementing Merge Support for Polars &amp; Delta Lake</li>
</ol>

<p>The Use Case: Currently, the <code>dagster-deltalake</code> I/O manager allows writing data, but it lacks out-of-the-box support for the merge operation when using Polars.</p>

<p>The Implementation: I am working on updating the <code>dagster_deltalake/handler.py</code> to support merge mode. The logic involves checking if the write mode is set to merge. If so, instead of calling the standard <code>write_deltalake()</code> function, it creates a <code>DeltaTable</code> object and executes the merge operation. </p>

<p><a href="https://github.com/dagster-io/dagster/issues/32644?reload=1" rel="noopener noreferrer">Issue #32644</a></p>

<p><strong>The CI</strong><br>
While fixing the code was satisfying, getting the Pull Requests (PRs) merged has been a different story. I am currently stuck in a loop regarding <code>CI</code> tests.</p>

<p>The Situation:</p>

<ol>
<li>I run the unit tests locally on my environment, and everything passes perfectly.</li>
<li>I push the code to GitHub, and the <code>CI</code> pipeline fails.</li>
<li>Because of this, I can't get a proper code review from the maintainers.</li>
</ol>

<p>It is frustrating because I cannot reproduce the errors locally. It could be an environment configuration mismatch, a <code>linting</code> rule that strictly applies in <code>CI</code>, or a hidden dependency issue.</p>

<p><strong>Next Steps</strong><br>
I plan to reach out to the <code>Dagster</code> team and the community for guidance. I need to understand how their <code>CI</code> environment differs from a standard local setup so I can replicate the failure and fix it. Sometimes, reading thousands of lines of codes and fixing errors is easier than testing.</p>

