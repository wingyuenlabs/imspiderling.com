---
Title: # I Built a MindsEye x Google AI Stack in 6 Repos (Without Cloud Credits or API Budget… Yet)
Description: 
Author: PEACEBINFLOW
Date: 2025-11-13T21:47:22.000Z
Robots: noindex,nofollow
Template: index
---
<p>Intro</p>

<p>Over the last few days I’ve been quietly assembling something I’ve wanted for a long time:</p>

<p>A Google-native AI layer where prompts, runs, devlogs, and analytics all live in one shared brain — powered by my MindsEye framework — but implemented as small, focused GitHub repos.</p>

<p>I don’t have:</p>

<p>Google Cloud budget</p>

<p>Gemini API credits</p>

<p>Or a giant infra stack</p>

<p>So I did the next best thing:</p>

<p>Designed everything using Google Workspace surfaces (Sheets, Docs, Gmail, Forms, etc.)</p>

<p>Structured the code so it’s cloud-ready the moment I get funding/credits</p>

<p>Kept everything open in six connected repos.</p>

<p>This post is the overview: what each repo does, how they connect, and where MindsEye fits in the middle.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcg5wuqfdaghl6ic6c0t1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcg5wuqfdaghl6ic6c0t1.png" alt=" " width="800" height="1200"></a></p>

<p>The Six Repos (Quick Map)</p>

<p>Here are the six public repos that make up the system:</p>

<p>Workspace automation layer<br>
👉 <a href="https://github.com/PEACEBINFLOW/mindseye-workspace-automation/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-workspace-automation/tree/main</a></p>

<p>Ledger (Prompt Evolution Tree + runs)<br>
👉 <a href="https://github.com/PEACEBINFLOW/mindseye-google-ledger" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-ledger</a></p>

<p>Gemini orchestrator<br>
👉 <a href="https://github.com/PEACEBINFLOW/mindseye-gemini-orchestrator" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-gemini-orchestrator</a></p>

<p>Devlog generator (for Dev.to-style posts)<br>
👉 <a href="https://github.com/PEACEBINFLOW/mindseye-google-devlog/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-devlog/tree/main</a></p>

<p>Analytics layer (exports + dashboards)<br>
👉 <a href="https://github.com/PEACEBINFLOW/mindseye-google-analytics/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-analytics/tree/main</a></p>

<p>Workflow atlas (“portal maps”)<br>
👉 <a href="https://github.com/PEACEBINFLOW/mindseye-google-workflows/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-workflows/tree/main</a></p>

<p>Together, they turn Google Workspace into a kind of MindsEye-powered AI console:</p>

<p>Workspace events → logged as time-labeled nodes and runs</p>

<p>Nodes → executed via Gemini (when API access is available)</p>

<p>Runs → narrated as devlogs + aggregated as analytics</p>

<p>All flows → described as YAML workflows so nothing is “mystery glue”.</p>

<p>MindsEye’s Role in All This</p>

<p>MindsEye, in my head, is the cognitive layer:</p>

<p>It sees events (Gmail, Docs, Drive…)</p>

<p>It reasons over them (prompt evolution, runs, success rates…)</p>

<p>It remembers (ledger, devlogs, analytics history).</p>

<p>These repos are basically the Google-native skeleton for MindsEye:</p>

<p>Time-labeled prompts and experiments (Prompt Evolution Tree)</p>

<p>Cross-app workflows as “portals” between Google surfaces</p>

<p>A structure where Gemini/Google AI can later plug in without redesigning everything.</p>

<p>Right now it’s “offline-brain-mode” (no paid API calls), but all the pathways are there.</p>

<ol>
<li>mindseye-workspace-automation</li>
</ol>

<p>Google Workspace as the entry point</p>

<p>🔗 <a href="https://github.com/PEACEBINFLOW/mindseye-workspace-automation/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-workspace-automation/tree/main</a></p>

<p>This repo is where Google Workspace actually touches the system.</p>

<p>The idea:</p>

<p>Use Apps Script to listen for events in:</p>

<p>Gmail (labels, threads)</p>

<p>Google Docs (custom menu actions)</p>

<p>Drive (folder summaries)</p>

<p>Forms (new responses)</p>

<p>Normalize those events into a common shape:</p>

<p>surface (gmail/docs/drive/forms)</p>

<p>source_id, source_url</p>

<p>title, summary, event_type</p>

<p>Then send them through a “portal” to the ledger as either:</p>

<p>a new prompt node</p>

<p>or a new run attached to an existing node.</p>

<p>Think of this repo as:</p>

<p>“Whenever something interesting happens in Google Workspace, MindsEye finds out about it.”</p>

<ol>
<li>mindseye-google-ledger</li>
</ol>

<p>The Prompt Evolution Tree (PET) + runs</p>

<p>🔗 <a href="https://github.com/PEACEBINFLOW/mindseye-google-ledger" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-ledger</a></p>

<p>This is the core database, and it’s built on plain Google Sheets so it’s cheap and accessible.</p>

<p>It defines:</p>

<p>nodes sheet → every prompt / idea / variation</p>

<p>runs sheet → every execution of a node</p>

<p>Plus:</p>

<p>A Prompt Evolution Tree (PET) schema:</p>

<p>node_id, parent_node_id</p>

<p>prompt_type, status, tags</p>

<p>linked doc_url for long-form prompt docs</p>

<p>Apps Script that:</p>

<p>Handles Forms → ledger (new nodes via Google Forms)</p>

<p>Auto-creates Docs per node</p>

<p>Keeps everything indexed by node_id.</p>

<p>Goal:</p>

<p>Make prompt design &amp; experiments first-class, time-labeled data — not random notes in your head.</p>

<p>Everything else (orchestrator, devlog, analytics) treats this ledger as the single source of truth.</p>

<ol>
<li>mindseye-gemini-orchestrator</li>
</ol>

<p>When I do have Gemini access, this runs the show</p>

<p>🔗 <a href="https://github.com/PEACEBINFLOW/mindseye-gemini-orchestrator" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-gemini-orchestrator</a></p>

<p>This repo is the part that reads from the ledger, calls Google AI / Gemini, and logs back the result.</p>

<p>Right now, it’s wired as:</p>

<p>Node.js + TypeScript skeleton</p>

<p>Config for:</p>

<p>LEDGER_SHEET_ID, LEDGER_NODES_RANGE, LEDGER_RUNS_RANGE</p>

<p>GEMINI_MODEL_ID (e.g. gemini-1.5-pro)</p>

<p>Code divided into:</p>

<p>sheets_client.ts → read/write nodes/runs</p>

<p>gemini_client.ts → Google AI API interface (future)</p>

<p>runner.ts → orchestrate “run node N and log result”</p>

<p>index.ts / CLI → run single node or batches.</p>

<p>Because I don’t have paid Gemini access yet, this is:</p>

<p>Designed first, powered later — but the interface and data shape are ready.</p>

<p>The moment I can connect a service account + Gemini key, this becomes the execution engine.</p>

<ol>
<li>mindseye-google-devlog</li>
</ol>

<p>Dev.to-style logs straight from the ledger</p>

<p>🔗 <a href="https://github.com/PEACEBINFLOW/mindseye-google-devlog/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-devlog/tree/main</a></p>

<p>This repo’s entire job is to turn time windows of ledger activity into narrative devlogs.</p>

<p>It:</p>

<p>Reads nodes + runs from the ledger (via Sheets API)</p>

<p>Groups runs by node_id</p>

<p>Builds a DevlogData structure with:</p>

<p>run counts</p>

<p>contexts (gmail, docs, forms, etc.)</p>

<p>first/last run time</p>

<p>sample outputs/notes</p>

<p>Renders markdown using a Handlebars template ready for Dev.to.</p>

<p>Optionally, when Gemini access is available, it can:</p>

<p>Ask Gemini to write a summary intro for the week’s activity.</p>

<p>So instead of manually tracking everything, I can auto-generate posts like:</p>

<p>“Here’s what MindsEye experimented with this week across Gmail, Docs, and Devlogs.”</p>

<ol>
<li>mindseye-google-analytics</li>
</ol>

<p>Metrics, charts, and dashboards over PET + runs</p>

<p>🔗 <a href="https://github.com/PEACEBINFLOW/mindseye-google-analytics/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-analytics/tree/main</a></p>

<p>This repo turns the ledger into numbers and graphs.</p>

<p>It includes:</p>

<p>Sample exports:</p>

<p>exports/nodes_sample_export.csv</p>

<p>exports/runs_sample_export.csv</p>

<p>Python scripts:</p>

<p>compute_stats.py →</p>

<p>total_nodes, total_runs, avg_runs_per_node</p>

<p>success rate per prompt_type</p>

<p>top models by score</p>

<p>surface usage (run_context)</p>

<p>generate_charts.py →</p>

<p>runs per day</p>

<p>runs by prompt_type</p>

<p>runs by run_context</p>

<p>Dashboard docs:</p>

<p>How to wire it all into Looker Studio using Sheets as the source</p>

<p>KPI definitions (kpi_definitions.md).</p>

<p>This is the “how is the system evolving?” lens:<br>
are prompts getting better, which surfaces are busy, which models behave best, etc.</p>

<ol>
<li>mindseye-google-workflows</li>
</ol>

<p>The atlas: “portal maps” for the whole system</p>

<p>🔗 <a href="https://github.com/PEACEBINFLOW/mindseye-google-workflows/tree/main" rel="noopener noreferrer">https://github.com/PEACEBINFLOW/mindseye-google-workflows/tree/main</a></p>

<p>This repo doesn’t run code; it defines how everything connects.</p>

<p>It includes:</p>

<p>workflows/*.yaml files like:</p>

<p>00_overview.yaml → high-level system map</p>

<p>workspace_event_to_ledger.yaml → how Gmail/Docs/Drive/Forms become nodes/runs</p>

<p>ledger_to_gemini.yaml → node → Gemini → run</p>

<p>devlog_generation.yaml → time window → markdown devlog</p>

<p>analytics_refresh.yaml → nightly exports → stats + charts</p>

<p>portal_routes.yaml:</p>

<p>Canonical “portals” like:</p>

<p>workspace_event_to_ledger</p>

<p>ledger_to_gemini</p>

<p>ledger_to_devlog</p>

<p>ledger_to_analytics</p>

<p>Mapping files:</p>

<p>mappings/repos.yaml → logical component → GitHub repo</p>

<p>mappings/google_apps.yaml → logical role → Google app</p>

<p>Python helpers:</p>

<p>validate_workflows.py → check workflow YAMLs are valid + portals consistent</p>

<p>visualize_workflows.py → generate Mermaid diagrams from workflows.</p>

<p>Basically:</p>

<p>This repo is the atlas. The others are the continents.</p>

<p>Why Google’s Ecosystem? (And What’s Missing)</p>

<p>I intentionally built this on Google Workspace + Google AI because:</p>

<p>Sheets, Docs, Gmail, Forms, Drive, Calendar → already where teams live</p>

<p>Devs can easily extend with Apps Script, Node.js, Zapier, etc.</p>

<p>Looker Studio + Sheets → low-cost dashboards</p>

<p>Gemini → natural fit for prompt/run experimentation once I have access</p>

<p>What I don’t currently have:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhahk4sqs8w2b01y6aw1n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhahk4sqs8w2b01y6aw1n.png" alt=" " width="800" height="1200"></a></p>

<p>✅ Cloud infra money (Google Cloud project funded, App Engine/Cloud Run hosting, etc.)</p>

<p>✅ Paid Gemini API access / higher-tier free credits</p>

<p>So right now, the stack runs mostly as:</p>

<p>GitHub repos + local scripts</p>

<p>Google Workspace + Apps Script</p>

<p>A clear blueprint for when funding/API keys show up.</p>

<p>But the important part is done:</p>

<p>The data model exists</p>

<p>The workflows are described</p>

<p>The repos are live and open.<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl2yb38x5taw8mzc7u3x8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl2yb38x5taw8mzc7u3x8.png" alt=" " width="800" height="1200"></a></p>

