---
Title: PECOS Data Extraction Pipeline - DevOps Documentation
Description: 
Author: Durrell  Gemuh
Date: 2026-03-13T21:55:03.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Overview
</h2>

<p>The PECOS Data Extraction Pipeline is an enterprise-grade ETL workflow that extracts, transforms, and loads healthcare provider data from CMS PECOS datasets. The pipeline processes four datasets (Clinicians, Practices, Canonical Providers, Detail Tables) in parallel using PySpark and AWS Glue, orchestrated by AWS Step Functions.</p>

<p><strong>Repository</strong>: <a href="https://github.com/durrello/PECOS-Data-Extraction-Pipeline" rel="noopener noreferrer">https://github.com/durrello/PECOS-Data-Extraction-Pipeline</a></p>

<h2>
  
  
  Architecture
</h2>

<h3>
  
  
  AWS Architecture (Primary)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>                    ┌─────────────────────────────────────┐
                    │         AWS Step Functions           │
                    │      State Machine Orchestrator      │
                    └───────────────┬─────────────────────┘
                                    │
                          [ValidateInput]  ← Pass State
                                    │
                    ┌───────────────▼─────────────────────┐
                    │         ExtractParallel              │ ← Parallel State
                    │  ┌──────┐ ┌──────┐ ┌──────┐ ┌────┐ │
                    │  │Clin. │ │Prac. │ │Canon.│ │Det.│ │ ← Task States
                    │  │ ETL  │ │ ETL  │ │ ETL  │ │ETL │ │
                    │  └──────┘ └──────┘ └──────┘ └────┘ │
                    │   ↺ 3 retries per branch            │
                    └───────────────┬─────────────────────┘
                                    │
                          [AllSucceeded?]  ← Choice State
                         /                         \
               [NotifySuccess]              [NotifyFailure]
                     │                              │
             [PipelineSucceed]            [PipelineFail]
              ← Succeed State              ← Fail State
</code></pre>

</div>



<h3>
  
  
  Alternative Architectures
</h3>

<h4>
  
  
  Local Python (Development)
</h4>

<ul>
<li>Uses ThreadPoolExecutor for parallel execution</li>
<li>Local file system for data storage</li>
<li>Console logging for notifications</li>
<li>No AWS dependencies required</li>
</ul>

<h4>
  
  
  Docker (EMR Simulation)
</h4>

<ul>
<li>Containerized EMR-like environment</li>
<li>Volume mounts for data persistence</li>
<li>Isolated Python 3.11 + PySpark 3.5.1 environment</li>
</ul>

<h2>
  
  
  AWS Services Used
</h2>

<h3>
  
  
  Core Services
</h3>

<ul>
<li>
<strong>AWS Step Functions</strong>: Orchestrates the ETL pipeline workflow</li>
<li>
<strong>AWS Glue</strong>: Serverless PySpark ETL jobs (4 parallel jobs)</li>
<li>
<strong>Amazon S3</strong>: Data lake for input, output, and logs</li>
<li>
<strong>AWS IAM</strong>: Fine-grained permissions for services</li>
<li>
<strong>Amazon SNS</strong>: Email notifications for pipeline status</li>
</ul>

<h3>
  
  
  Supporting Services
</h3>

<ul>
<li>
<strong>AWS CloudWatch</strong>: Logging and monitoring</li>
<li>
<strong>AWS CloudWatch Logs</strong>: Detailed execution logs</li>
<li>
<strong>AWS X-Ray</strong>: Distributed tracing (optional)</li>
</ul>

<h2>
  
  
  Prerequisites
</h2>

<h3>
  
  
  AWS Prerequisites
</h3>

<ul>
<li>AWS CLI v2 configured with admin permissions</li>
<li>S3 bucket for data storage (auto-created by bootstrap)</li>
<li>IAM permissions to create roles, policies, and services</li>
</ul>

<h3>
  
  
  Local Development Prerequisites
</h3>

<ul>
<li>Python 3.11 (required for PySpark compatibility)</li>
<li>Java 17+ (for Spark runtime)</li>
<li>Docker (optional, for containerized runs)</li>
</ul>

<h2>
  
  
  Deployment Options
</h2>

<h3>
  
  
  Option 1: AWS One-Command Bootstrap (Recommended)
</h3>

<p>The bootstrap script automates the entire AWS deployment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Basic deployment</span>
./bootstrap.sh

<span class="c"># With custom environment and email</span>
./bootstrap.sh production your-email@example.com
</code></pre>

</div>



<p><strong>What the bootstrap script does:</strong></p>

<ol>
<li>Creates IAM roles with minimal required permissions</li>
<li>Verifies/creates S3 bucket</li>
<li>Uploads code, config, and data to S3</li>
<li>Creates/updates 4 AWS Glue ETL jobs</li>
<li>Sets up SNS topic and email notifications</li>
<li>Deploys Step Functions state machine</li>
<li>Starts initial pipeline execution</li>
</ol>

<p><strong>Output:</strong></p>

<ul>
<li>Execution ARN for monitoring</li>
<li>Console URLs for Step Functions and Glue</li>
<li>SNS topic ARN for notifications</li>
</ul>

<h3>
  
  
  Option 2: Manual AWS Deployment
</h3>

<p>For custom deployments or CI/CD integration:</p>

<h4>
  
  
  1. IAM Roles Setup
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Step Functions execution role</span>
aws iam create-role <span class="nt">--role-name</span> PECOSStepFunctionsRole <span class="se">\</span>
  <span class="nt">--assume-role-policy-document</span> <span class="s1">'{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "states.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'</span>

<span class="c"># Glue execution role</span>
aws iam create-role <span class="nt">--role-name</span> PECOSGlueRole <span class="se">\</span>
  <span class="nt">--assume-role-policy-document</span> <span class="s1">'{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "glue.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'</span>
</code></pre>

</div>



<h4>
  
  
  2. S3 Bucket Setup
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nv">BUCKET</span><span class="o">=</span><span class="s2">"durrell-backend-bucket-terraform"</span>
aws s3 mb s3://<span class="nv">$BUCKET</span>
</code></pre>

</div>



<h4>
  
  
  3. Upload Artifacts
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Upload Spark jobs</span>
aws s3 <span class="nb">sync </span>spark_jobs/ s3://<span class="nv">$BUCKET</span>/spark_jobs/ <span class="se">\</span>
  <span class="nt">--exclude</span> <span class="s2">"*.pyc"</span> <span class="nt">--exclude</span> <span class="s2">"__pycache__/*"</span>

<span class="c"># Upload config</span>
aws s3 <span class="nb">cp </span>config/pipeline_config.yaml s3://<span class="nv">$BUCKET</span>/config/

<span class="c"># Upload input data</span>
aws s3 <span class="nb">sync </span>data/input/ s3://<span class="nv">$BUCKET</span>/data/input/
</code></pre>

</div>



<h4>
  
  
  4. Create Glue Jobs
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Example for Clinicians ETL job</span>
aws glue create-job <span class="se">\</span>
  <span class="nt">--name</span> <span class="s2">"PECOS-Clinicians-ETL"</span> <span class="se">\</span>
  <span class="nt">--role</span> <span class="s2">"arn:aws:iam::ACCOUNT_ID:role/PECOSGlueRole"</span> <span class="se">\</span>
  <span class="nt">--command</span> <span class="s2">"{
    </span><span class="se">\"</span><span class="s2">Name</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">glueetl</span><span class="se">\"</span><span class="s2">,
    </span><span class="se">\"</span><span class="s2">ScriptLocation</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">s3://</span><span class="nv">$BUCKET</span><span class="s2">/spark_jobs/glue_clinicians.py</span><span class="se">\"</span><span class="s2">,
    </span><span class="se">\"</span><span class="s2">PythonVersion</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">3</span><span class="se">\"</span><span class="s2">
  }"</span> <span class="se">\</span>
  <span class="nt">--default-arguments</span> <span class="s2">"{
    </span><span class="se">\"</span><span class="s2">--config</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">s3://</span><span class="nv">$BUCKET</span><span class="s2">/config/pipeline_config.yaml</span><span class="se">\"</span><span class="s2">,
    </span><span class="se">\"</span><span class="s2">--additional-python-modules</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">pyyaml</span><span class="se">\"</span><span class="s2">,
    </span><span class="se">\"</span><span class="s2">--enable-continuous-cloudwatch-log</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">true</span><span class="se">\"</span><span class="s2">,
    </span><span class="se">\"</span><span class="s2">--enable-metrics</span><span class="se">\"</span><span class="s2">: </span><span class="se">\"</span><span class="s2">true</span><span class="se">\"</span><span class="s2">
  }"</span> <span class="se">\</span>
  <span class="nt">--glue-version</span> <span class="s2">"4.0"</span> <span class="se">\</span>
  <span class="nt">--number-of-workers</span> 2 <span class="se">\</span>
  <span class="nt">--worker-type</span> <span class="s2">"G.1X"</span>
</code></pre>

</div>



<h4>
  
  
  5. SNS Notifications Setup
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Create topic</span>
<span class="nv">TOPIC_ARN</span><span class="o">=</span><span class="si">$(</span>aws sns create-topic <span class="nt">--name</span> pecos-pipeline-notifications <span class="nt">--query</span> TopicArn <span class="nt">--output</span> text<span class="si">)</span>

<span class="c"># Subscribe email</span>
aws sns subscribe <span class="nt">--topic-arn</span> <span class="nv">$TOPIC_ARN</span> <span class="nt">--protocol</span> email <span class="nt">--notification-endpoint</span> your-email@example.com
</code></pre>

</div>



<h4>
  
  
  6. Deploy State Machine
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Substitute variables in ASL</span>
<span class="nb">sed</span> <span class="nt">-e</span> <span class="s2">"s/</span><span class="se">\$</span><span class="s2">{REGION}/us-east-1/g"</span> <span class="se">\</span>
    <span class="nt">-e</span> <span class="s2">"s/</span><span class="se">\$</span><span class="s2">{ACCOUNT_ID}/YOUR_ACCOUNT_ID/g"</span> <span class="se">\</span>
    <span class="nt">-e</span> <span class="s2">"s/</span><span class="se">\$</span><span class="s2">{BUCKET}/</span><span class="nv">$BUCKET</span><span class="s2">/g"</span> <span class="se">\</span>
    state_machine/pecos_state_machine.asl.json <span class="o">&gt;</span> temp_asl.json

<span class="c"># Create state machine</span>
aws stepfunctions create-state-machine <span class="se">\</span>
  <span class="nt">--name</span> <span class="s2">"PECOS-Extraction-Pipeline"</span> <span class="se">\</span>
  <span class="nt">--definition</span> file://temp_asl.json <span class="se">\</span>
  <span class="nt">--role-arn</span> <span class="s2">"arn:aws:iam::ACCOUNT_ID:role/PECOSStepFunctionsRole"</span>
</code></pre>

</div>



<h3>
  
  
  Option 3: Local Python Development
</h3>

<p>For development and testing without AWS:</p>

<h4>
  
  
  Environment Setup
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Install Python 3.11</span>
brew <span class="nb">install </span>python@3.11  <span class="c"># macOS</span>
python3.11 <span class="nt">-m</span> venv .venv
<span class="nb">source</span> .venv/bin/activate
pip <span class="nb">install</span> <span class="nt">-r</span> requirements.txt
</code></pre>

</div>



<h4>
  
  
  Run Pipeline Locally
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Full pipeline</span>
python state_machine/pipeline_orchestrator.py <span class="se">\</span>
  <span class="nt">--input</span> state_machine/sample_input.json <span class="se">\</span>
  <span class="nt">--config</span> config/pipeline_config.yaml

<span class="c"># Individual ETL jobs</span>
python spark_jobs/etl_clinicians.py <span class="nt">--execution-id</span> DEV-001
</code></pre>

</div>



<h3>
  
  
  Option 4: Docker Development
</h3>

<p>EMR-like containerized environment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Build and run</span>
docker build <span class="nt">-f</span> docker/Dockerfile <span class="nt">-t</span> pecos-pipeline <span class="nb">.</span>
docker-compose up

<span class="c"># Or run specific command</span>
docker run <span class="nt">--rm</span> <span class="nt">-v</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/data:/app/data pecos-pipeline <span class="se">\</span>
  python state_machine/pipeline_orchestrator.py
</code></pre>

</div>



<h2>
  
  
  Configuration Management
</h2>

<h3>
  
  
  Pipeline Configuration (pipeline_config.yaml)
</h3>

<p>The configuration is environment-aware and supports both local and AWS deployments:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">pipeline</span><span class="pi">:</span>
  <span class="na">name</span><span class="pi">:</span> <span class="s2">"</span><span class="s">PECOS-Extraction-Pipeline"</span>
  <span class="na">environment</span><span class="pi">:</span> <span class="s2">"</span><span class="s">local"</span>  <span class="c1"># local | dev | staging | prod</span>
  <span class="na">execution_id_prefix</span><span class="pi">:</span> <span class="s2">"</span><span class="s">PECOS"</span>

<span class="c1"># S3 paths (AWS) vs local paths</span>
<span class="na">paths</span><span class="pi">:</span>
  <span class="na">base_input_dir</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://bucket/data/input"</span>   <span class="c1"># AWS</span>
  <span class="na">base_output_dir</span><span class="pi">:</span> <span class="s2">"</span><span class="s">s3://bucket/data/output"</span> <span class="c1"># AWS</span>

<span class="na">local_paths</span><span class="pi">:</span>
  <span class="na">base_input_dir</span><span class="pi">:</span> <span class="s2">"</span><span class="s">data/input"</span>    <span class="c1"># Local</span>
  <span class="na">base_output_dir</span><span class="pi">:</span> <span class="s2">"</span><span class="s">data/output"</span>  <span class="c1"># Local</span>

<span class="c1"># Dataset-specific configs</span>
<span class="na">datasets</span><span class="pi">:</span>
  <span class="na">clinicians</span><span class="pi">:</span>
    <span class="na">input_file</span><span class="pi">:</span> <span class="s2">"</span><span class="s">clinicians.csv"</span>
    <span class="na">partition_by</span><span class="pi">:</span> <span class="pi">[</span><span class="s2">"</span><span class="s">state"</span><span class="pi">]</span>
    <span class="na">primary_key</span><span class="pi">:</span> <span class="s2">"</span><span class="s">npi"</span>
    <span class="na">required_columns</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">...</span><span class="pi">]</span>
</code></pre>

</div>



<h3>
  
  
  Environment Variables
</h3>

<p>Override configuration at runtime:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">export </span><span class="nv">PECOS_BASE_INPUT</span><span class="o">=</span>data/input
<span class="nb">export </span><span class="nv">PECOS_BASE_OUTPUT</span><span class="o">=</span>data/output
<span class="nb">export </span><span class="nv">PECOS_LOG_DIR</span><span class="o">=</span>logs
</code></pre>

</div>



<h2>
  
  
  Monitoring and Observability
</h2>

<h3>
  
  
  AWS Monitoring
</h3>

<h4>
  
  
  Step Functions Console
</h4>

<ul>
<li>Real-time execution visualization</li>
<li>State transition timing</li>
<li>Error details and stack traces</li>
<li>Execution history and re-runs</li>
</ul>

<h4>
  
  
  Glue Console
</h4>

<ul>
<li>Job run status and duration</li>
<li>Resource utilization (DPUs)</li>
<li>Error logs and troubleshooting</li>
</ul>

<h4>
  
  
  CloudWatch Logs
</h4>

<ul>
<li>Structured logs from all components</li>
<li>Log groups:

<ul>
<li><code>/aws/states/PECOS-Extraction-Pipeline</code></li>
<li><code>/aws-glue/jobs/logs-v2/</code></li>
</ul>


</li>

</ul>

<h4>
  
  
  CloudWatch Metrics
</h4>

<ul>
<li>Step Functions: Execution metrics</li>
<li>Glue: Job performance metrics</li>
<li>S3: Storage and access metrics</li>
</ul>

<h3>
  
  
  Local Monitoring
</h3>

<h4>
  
  
  Console Logging
</h4>

<ul>
<li>Structured JSON logs with timestamps</li>
<li>Execution summaries in <code>logs/{execution_id}_summary.json</code>
</li>
<li>Dataset-specific logs in <code>logs/{execution_id}_{dataset}.log</code>
</li>
</ul>

<h4>
  
  
  File System Monitoring
</h4>

<ul>
<li>Output Parquet files in output</li>
<li>Partitioned by state/status</li>
<li>Snappy-compressed for efficiency</li>
</ul>

<h2>
  
  
  Pipeline Execution
</h2>

<h3>
  
  
  Starting Executions
</h3>

<h4>
  
  
  AWS Step Functions
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Prepare input</span>
<span class="nv">EXECUTION_INPUT</span><span class="o">=</span><span class="s1">'{
  "trigger_source": "manual",
  "run_date": "'</span><span class="si">$(</span><span class="nb">date</span> +%Y-%m-%d<span class="si">)</span><span class="s1">'",
  "environment": "production",
  "config_s3_uri": "s3://bucket/config/pipeline_config.yaml",
  "input_s3_prefix": "s3://bucket/data/input/",
  "output_s3_prefix": "s3://bucket/data/output/"
}'</span>

<span class="c"># Start execution</span>
<span class="nv">EXECUTION_ARN</span><span class="o">=</span><span class="si">$(</span>aws stepfunctions start-execution <span class="se">\</span>
  <span class="nt">--state-machine-arn</span> <span class="nv">$STATE_MACHINE_ARN</span> <span class="se">\</span>
  <span class="nt">--input</span> <span class="s2">"</span><span class="nv">$EXECUTION_INPUT</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">--query</span> executionArn <span class="nt">--output</span> text<span class="si">)</span>
</code></pre>

</div>



<h4>
  
  
  Local Execution
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>python state_machine/pipeline_orchestrator.py <span class="se">\</span>
  <span class="nt">--input</span> state_machine/sample_input.json <span class="se">\</span>
  <span class="nt">--config</span> config/pipeline_config.yaml
</code></pre>

</div>



<h3>
  
  
  Execution States
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>State</th>
<th>Type</th>
<th>Description</th>
<th>Retry Behavior</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ValidateInput</code></td>
<td>Pass</td>
<td>Enrich input with metadata</td>
<td>N/A</td>
</tr>
<tr>
<td><code>ExtractParallel</code></td>
<td>Parallel</td>
<td>Run 4 ETL jobs concurrently</td>
<td>N/A</td>
</tr>
<tr>
<td><code>ExtractCliniciansTask</code></td>
<td>Task</td>
<td>Clinicians ETL</td>
<td>3 retries, 2x backoff</td>
</tr>
<tr>
<td><code>ExtractPracticesTask</code></td>
<td>Task</td>
<td>Practices ETL</td>
<td>3 retries, 2x backoff</td>
</tr>
<tr>
<td><code>ExtractCanonicalProvidersTask</code></td>
<td>Task</td>
<td>Canonical Providers ETL</td>
<td>3 retries, 2x backoff</td>
</tr>
<tr>
<td><code>ExtractDetailTablesTask</code></td>
<td>Task</td>
<td>Detail Tables ETL</td>
<td>3 retries, 2x backoff</td>
</tr>
<tr>
<td><code>AllSucceeded</code></td>
<td>Choice</td>
<td>Route based on results</td>
<td>N/A</td>
</tr>
<tr>
<td><code>NotifySuccess</code></td>
<td>Task</td>
<td>Send success notification</td>
<td>N/A</td>
</tr>
<tr>
<td><code>NotifyFailure</code></td>
<td>Task</td>
<td>Send failure notification</td>
<td>N/A</td>
</tr>
<tr>
<td><code>PipelineSucceed</code></td>
<td>Succeed</td>
<td>Terminal success</td>
<td>N/A</td>
</tr>
<tr>
<td><code>PipelineFail</code></td>
<td>Fail</td>
<td>Terminal failure</td>
<td>N/A</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Data Processing
</h2>

<h3>
  
  
  ETL Transformations
</h3>

<h4>
  
  
  Clinicians Dataset
</h4>

<ul>
<li>Filter active providers (<code>status = A</code>)</li>
<li>Zero-pad NPI to 10 digits</li>
<li>Derive <code>credential_group</code> (Physician/Mid-Level/Nursing/Other)</li>
<li>Compute <code>years_enrolled</code> from enrollment date</li>
<li>Data quality flag: <code>dq_has_specialty</code>
</li>
<li>Deduplicate on <code>(npi, state)</code>
</li>
</ul>

<h4>
  
  
  Practices Dataset
</h4>

<ul>
<li>Filter active practices</li>
<li>Derive <code>practice_size_category</code> (Solo/Small/Medium/Large/Enterprise)</li>
<li>Map state → US Census region</li>
<li>Flag multi-specialty groups</li>
<li>Deduplicate on <code>practice_id</code>
</li>
</ul>

<h4>
  
  
  Canonical Providers Dataset
</h4>

<ul>
<li>Build golden record using window functions</li>
<li>Compute <code>participation_score</code> (0-3: Medicare + Medicaid + Telehealth)</li>
<li>Assign <code>participation_tier</code> (Full/Partial/Limited/None)</li>
<li>Build <code>display_name</code> for individuals and organizations</li>
</ul>

<h4>
  
  
  Detail Tables Dataset
</h4>

<ul>
<li>Parse and classify enrollment events</li>
<li>Compute <code>enrollment_duration_days</code>
</li>
<li>Decode <code>reason_code</code> to human-readable descriptions</li>
<li>Classify <code>termination_type</code>
</li>
<li>Compute window-based <code>group_reassignment_rate</code>
</li>
<li>Flag <code>is_recent_enrollment</code> (&lt; 180 days) and <code>is_stale_record</code> (&gt; 5 years)</li>
</ul>

<h3>
  
  
  Output Format
</h3>

<p>All datasets output <strong>Snappy-compressed Parquet</strong> with metadata columns:</p>

<ul>
<li>
<code>_pipeline_execution_id</code>: Unique run identifier</li>
<li>
<code>_pipeline_ingested_at</code>: UTC ingestion timestamp</li>
<li>
<code>_pipeline_source_file</code>: Source file path</li>
</ul>

<p>Partitioning strategy:</p>

<ul>
<li>Clinicians/Practices/Canonical: <code>state=XX/</code>
</li>
<li>Detail Tables: <code>state=XX/status=Y/</code>
</li>
</ul>

<h2>
  
  
  Notifications
</h2>

<h3>
  
  
  AWS SNS Notifications
</h3>

<ul>
<li>Email notifications for success/failure</li>
<li>Structured JSON payload with execution details</li>
<li>Configurable recipients</li>
</ul>

<h3>
  
  
  Local Notifications
</h3>

<ul>
<li>Console output with execution summary</li>
<li>JSON-formatted logs</li>
<li>File-based execution summaries</li>
</ul>

<h2>
  
  
  Security
</h2>

<h3>
  
  
  IAM Permissions
</h3>

<h4>
  
  
  Principle of Least Privilege
</h4>

<ul>
<li>Step Functions role: Only Glue start/run, SNS publish, CloudWatch logs</li>
<li>Glue role: Scoped S3 access (only pipeline bucket), Glue service role</li>
</ul>

<h4>
  
  
  S3 Bucket Policies
</h4>

<ul>
<li>Separate permissions for input, output, and logs</li>
<li>No wildcard permissions</li>
<li>Encryption at rest (SSE-S3)</li>
</ul>

<h3>
  
  
  Network Security
</h3>

<ul>
<li>All services communicate within AWS network</li>
<li>No public endpoints exposed</li>
<li>VPC deployment possible for enhanced security</li>
</ul>

<h2>
  
  
  Cost Optimization
</h2>

<h3>
  
  
  AWS Glue
</h3>

<ul>
<li>G.1X worker type (cost-effective for CPU-bound workloads)</li>
<li>2 workers per job (parallel processing within job)</li>
<li>Auto-scaling disabled (predictable costs)</li>
<li>Timeout: 60 minutes</li>
</ul>

<h3>
  
  
  Step Functions
</h3>

<ul>
<li>Standard workflow (cost-effective for ETL orchestration)</li>
<li>No Express workflows needed</li>
</ul>

<h3>
  
  
  S3 Storage
</h3>

<ul>
<li>Intelligent tiering for output data</li>
<li>Lifecycle policies for logs</li>
<li>Snappy compression reduces storage costs</li>
</ul>

<h2>
  
  
  Troubleshooting
</h2>

<h3>
  
  
  Common Issues
</h3>

<h4>
  
  
  SNS Email Not Received
</h4>

<ul>
<li>Confirm subscription in email</li>
<li>Check SNS topic permissions</li>
<li>Verify email address format</li>
</ul>

<h4>
  
  
  Glue Job Failures
</h4>

<ul>
<li>Check CloudWatch logs: <code>/aws-glue/jobs/logs-v2/</code>
</li>
<li>Verify S3 permissions</li>
<li>Check PySpark version compatibility</li>
</ul>

<h4>
  
  
  Step Functions Timeouts
</h4>

<ul>
<li>Increase Glue job timeout</li>
<li>Optimize Spark configurations</li>
<li>Check for data skew</li>
</ul>

<h4>
  
  
  Local Development Issues
</h4>

<ul>
<li>Ensure Python 3.11 (PySpark requirement)</li>
<li>Check Java version (17+)</li>
<li>Verify virtual environment activation</li>
</ul>

<h3>
  
  
  Debugging Commands
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check execution status</span>
aws stepfunctions describe-execution <span class="nt">--execution-arn</span> <span class="nv">$EXECUTION_ARN</span>

<span class="c"># View Glue job logs</span>
aws logs <span class="nb">tail</span> /aws-glue/jobs/logs-v2/ <span class="nt">--follow</span>

<span class="c"># List output files</span>
aws s3 <span class="nb">ls </span>s3://bucket/data/output/ <span class="nt">--recursive</span>

<span class="c"># Check S3 permissions</span>
aws s3api get-bucket-policy <span class="nt">--bucket</span> bucket
</code></pre>

</div>



<h2>
  
  
  Cleanup
</h2>

<h3>
  
  
  AWS Resource Cleanup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Automated teardown</span>
./teardown.sh
</code></pre>

</div>



<p><strong>Deletes:</strong></p>

<ul>
<li>Step Functions state machine</li>
<li>4 AWS Glue ETL jobs</li>
<li>SNS topic and subscriptions</li>
<li>S3 bucket and all contents</li>
<li>IAM roles and policies</li>
</ul>

<h3>
  
  
  Local Cleanup
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Remove outputs and logs</span>
<span class="nb">rm</span> <span class="nt">-rf</span> data/output/<span class="k">*</span> logs/<span class="k">*</span>

<span class="c"># Remove virtual environment</span>
<span class="nb">rm</span> <span class="nt">-rf</span> .venv
</code></pre>

</div>



<h2>
  
  
  Performance Tuning
</h2>

<h3>
  
  
  Spark Configurations
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">spark</span><span class="pi">:</span>
  <span class="na">configs</span><span class="pi">:</span>
    <span class="na">spark.sql.shuffle.partitions</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
    <span class="na">spark.default.parallelism</span><span class="pi">:</span> <span class="s2">"</span><span class="s">4"</span>
    <span class="na">spark.sql.adaptive.enabled</span><span class="pi">:</span> <span class="s2">"</span><span class="s">true"</span>
    <span class="na">spark.sql.parquet.compression.codec</span><span class="pi">:</span> <span class="s2">"</span><span class="s">snappy"</span>
</code></pre>

</div>



<h3>
  
  
  Glue Job Sizing
</h3>

<ul>
<li>G.1X workers: 1 DPU, 16GB RAM, 4 vCPU</li>
<li>Scale workers based on data volume</li>
<li>Monitor DPU utilization in Glue console</li>
</ul>

<h3>
  
  
  Parallel Processing
</h3>

<ul>
<li>4 ETL jobs run concurrently</li>
<li>No inter-job dependencies</li>
<li>Optimal for I/O bound workloads</li>
</ul>

<h2>
  
  
  Backup and Recovery
</h2>

<h3>
  
  
  Data Backup
</h3>

<ul>
<li>S3 versioning enabled on bucket</li>
<li>Cross-region replication for critical data</li>
<li>Regular snapshots of configuration</li>
</ul>

<h3>
  
  
  Pipeline Recovery
</h3>

<ul>
<li>Step Functions supports execution re-runs</li>
<li>Idempotent ETL jobs (overwrite mode)</li>
<li>Partial failure handling (continue with successful jobs)</li>
</ul>

<h2>
  
  
  Compliance and Governance
</h2>

<h3>
  
  
  Data Governance
</h3>

<ul>
<li>Structured logging with execution IDs</li>
<li>Data lineage tracking</li>
<li>Audit trails in CloudWatch</li>
</ul>

<h3>
  
  
  Healthcare Compliance
</h3>

<ul>
<li>PHI data handling considerations</li>
<li>Encryption at rest and in transit</li>
<li>Access logging and monitoring</li>
</ul>

<h2>
  
  
  Future Enhancements
</h2>

<h3>
  
  
  Potential Improvements
</h3>

<ul>
<li>Event-driven triggers (S3 event notifications)</li>
<li>Data quality frameworks (Great Expectations)</li>
<li>Advanced monitoring (custom CloudWatch dashboards)</li>
<li>Multi-region deployment</li>
<li>Blue/green deployment strategy</li>
</ul>

<h3>
  
  
  Scaling Considerations
</h3>

<ul>
<li>EMR migration for large datasets</li>
<li>Kubernetes deployment option</li>
<li>Serverless architecture evolution</li>
</ul>

<p>This documentation provides comprehensive DevOps guidance for deploying and managing the PECOS Data Extraction Pipeline across AWS, local, and containerized environments. The pipeline demonstrates enterprise-grade ETL patterns with robust error handling, monitoring, and cost optimization.</p>

