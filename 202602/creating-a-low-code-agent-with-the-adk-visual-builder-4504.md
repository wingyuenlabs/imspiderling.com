---
Title: Creating a Low-code Agent with the ADK Visual Builder
Description: 
Author: xbill
Date: 2026-02-21T21:43:35.000Z
Robots: noindex,nofollow
Template: index
---
<p>Leveraging the Google Agent Development Kit (ADK) and the underlying Gemini LLM to build low code in the Python programming language deployed securely to Google Cloud Run.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F722gp42o7m9epkc213p4.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F722gp42o7m9epkc213p4.jpeg" width="800" height="436"></a></p>

<h4>
  
  
  Python Version Management
</h4>

<p>One of the downsides of the wide deployment of Python has been managing the language versions across platforms and maintaining a supported version.</p>

<p>The <strong>pyenv</strong> tool enables deploying consistent versions of Python:</p>

<p><a href="https://github.com/pyenv/pyenv" rel="noopener noreferrer">GitHub - pyenv/pyenv: Simple Python version management</a></p>

<p>As of writing — the mainstream python version is 3.13. To validate your current Python:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~$ python --version
Python 3.13.12

xbill@penguin:~$ pyenv version
3.13.12 (set by /home/xbill/.pyenv/version)
</code></pre>

</div>



<h4>
  
  
  Gemini CLI
</h4>

<p>If not pre-installed you can download the Gemini CLI to interact with the source files and provide real-time assistance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm install -g @google/gemini-cli
</code></pre>

</div>



<h4>
  
  
  Testing the Gemini CLI Environment
</h4>

<p>Once you have all the tools and the correct Node.js version in place- you can test the startup of Gemini CLI. You will need to authenticate with a Key or your Google Account:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>gemini
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjf9fz2qgv4bicut1fo3m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjf9fz2qgv4bicut1fo3m.png" width="800" height="450"></a></p>

<h4>
  
  
  Node Version Management
</h4>

<p>Gemini CLI needs a consistent, up to date version of Node. The <strong>nvm</strong> command can be used to get a standard Node environment:</p>

<p><a href="https://github.com/nvm-sh/nvm" rel="noopener noreferrer">GitHub - nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions</a></p>

<h4>
  
  
  Agent Development Kit
</h4>

<p>The <a href="https://www.google.com/search?q=Google+Agent+Development+Kit&amp;rlz=1CAIWTJ_enUS1114&amp;oq=what+is+the+adk+google&amp;gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIICAEQABgWGB4yCAgCEAAYFhgeMggIAxAAGBYYHjIICAQQABgWGB4yCAgFEAAYFhgeMggIBhAAGBYYHjIKCAcQABgKGBYYHjINCAgQABiGAxiABBiKBTIKCAkQABiABBiiBNIBCDMxODlqMGo3qAIAsAIA&amp;sourceid=chrome&amp;ie=UTF-8&amp;mstk=AUtExfB5Oo7ZHHcDEHu7aqZiPBA2l1c-QGh5dB7xkkDPIiYcn8O1Imt2IHNR7bzA6JnyDCSDCUGpGWTeBW14namlN_QqzJLLI5-px1BE9jfSxwli6njPDPERjm5pRqNP3uC6HhUKiRcTJ1T8x5LHQrCkVxylw7QWg0N8B4dQDIcWpnVX9Gc&amp;csui=3&amp;ved=2ahUKEwjYu-G8p-uSAxXrv4kEHUbpLo0QgK4QegQIARAB" rel="noopener noreferrer">Google Agent Development Kit</a> (ADK) is an open-source, Python-based framework designed to streamline the creation, deployment, and orchestration of sophisticated, multi-agent AI systems. It treats agent development like software engineering, offering modularity, state management, and built-in tools (like Google Search) to build autonomous agents.</p>

<p>The ADK can be installed from here:</p>

<p><a href="https://google.github.io/adk-docs/" rel="noopener noreferrer">Agent Development Kit (ADK)</a></p>

<h4>
  
  
  Where do I start?
</h4>

<p>The strategy for starting low code agent development is a incremental step by step approach.</p>

<p>The agents in the demo are based on the original code lab:</p>

<p><a href="https://codelabs.developers.google.com/codelabs/create-low-code-agent-with-ADK-visual-builder#0" rel="noopener noreferrer">Create and deploy low code ADK (Agent Deployment Kit) agents using ADK Visual Builder | Google Codelabs</a></p>

<p>First, the basic development environment is setup with the required system variables, and a working Gemini CLI configuration.</p>

<p>Then, a minimal ADK Agent is built with the visual builder. Next— the entire solution is deployed to Google Cloud Run.</p>

<h4>
  
  
  Setup the Basic Environment
</h4>

<p>At this point you should have a working Python environment and a working Gemini CLI installation. The next step is to clone the GitHub samples repository with support scripts:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>cd ~
git clone https://github.com/xbill9/adkui
</code></pre>

</div>



<p>Then run <strong>init.sh</strong> from the cloned directory.</p>

<p>The script will attempt to determine your shell environment and set the correct variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>source init.sh
pip install -r requirements.txt
</code></pre>

</div>



<p>If your session times out or you need to re-authenticate- you can run the <strong>set_env.sh</strong> script to reset your environment variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>source set_env.sh
</code></pre>

</div>



<p>Variables like PROJECT_ID need to be setup for use in the various build scripts- so the set_env script can be used to reset the environment if you time-out.</p>

<h4>
  
  
  Running the Agent Code
</h4>

<p>First- start the ADK interface:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/adkui$ adk web
2026-02-21 14:31:53,810 - INFO - service_factory.py:266 - Using in-memory memory service
2026-02-21 14:31:53,811 - INFO - local_storage.py:84 - Using per-agent session storage rooted at /home/xbill/adkui
2026-02-21 14:31:53,811 - INFO - local_storage.py:110 - Using file artifact service at /home/xbill/adkui/.adk/artifacts
INFO: Started server process [25329]
INFO: Waiting for application startup.

+-----------------------------------------------------------------------------+
| ADK Web Server started |
| |
| For local testing, access at http://127.0.0.1:8000. |
+-----------------------------------------------------------------------------+

INFO: Application startup complete.
INFO: Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
</code></pre>

</div>



<h4>
  
  
  Running the Tool Locally
</h4>

<p>Connect to the local web interface:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffcvq2za6xu6sreoaqo1u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffcvq2za6xu6sreoaqo1u.png" width="800" height="450"></a></p>

<p>Then drill into the first Agent — Agent1:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fef4bjzh067fa0mt8ka6y.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fef4bjzh067fa0mt8ka6y.png" width="800" height="450"></a></p>

<p>This Agent uses the google_search tool along with the current context:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxb10ej3evhp6u3maor1j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxb10ej3evhp6u3maor1j.png" width="800" height="450"></a></p>

<h4>
  
  
  Deploying to Cloud Run
</h4>

<p>After the HTTP version of the Agent has been tested locally — it can be deployed remotely to Google Cloud Run.</p>

<p>First- switch to the directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>xbill@penguin:~/adkui$ python3 deploycloudrun.py 
🔍 Checking if service account adkvisualbuilder@comglitn.iam.gserviceaccount.com exists...
✅ Service account adkvisualbuilder already exists.
🔐 Assigning IAM roles to adkvisualbuilder@comglitn.iam.gserviceaccount.com...
</code></pre>

</div>



<p>When the build is complete- an endpoint will be returned. The service endpoint in this example is :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://agent1service-1056842563084.us-central1.run.app
</code></pre>

</div>



<p>The actual endpoint will vary based on your project settings.</p>

<h4>
  
  
  Review Service in Cloud Run
</h4>

<p>Navigate to the Google Cloud console and search for Cloud Run -</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frr8y43nja7dm32qnk87m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frr8y43nja7dm32qnk87m.png" width="800" height="450"></a></p>

<h4>
  
  
  Validate Cloud Run Deployment
</h4>

<p>Once you have the Endpoint — you can attempt a connection- navigate to in your browser:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffkyw6iomz6yjlu8gwb0f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffkyw6iomz6yjlu8gwb0f.png" width="800" height="450"></a></p>

<h4>
  
  
  Code Review with Gemini CLI
</h4>

<p>Gemini CLI was used to review the Agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✦ The Agent1 directory contains a minimal starter configuration for a Google ADK agent. Here is the code review for
  Agent1/root_agent.yaml:

  Agent1/root_agent.yaml

   1 name: Agent1
   2 model: gemini-2.5-flash
   3 agent_class: LlmAgent
   4 instruction: You are the root agent that coordinates other agents.
   5 sub_agents: []
   6 tools:
   7 - name: google_search

  Observations:

   1. Model Selection: It uses gemini-2.5-flash. Ensure your environment and API keys support this specific model
      version, as it is a cutting-edge/experimental release.
   2. Instruction Clarity: The instruction is extremely brief: "You are the root agent that coordinates other
      agents." However, sub_agents is currently an empty list [].
       * Recommendation: If this agent is intended to be a coordinator, you should define the sub_agents it will
         manage. If it's a standalone agent, update the instruction to reflect its specific task (e.g., "You are a
         research assistant...").
   3. Tooling: It only has access to google_search.
       * Recommendation: Since it's a "root agent," it might benefit from tools that allow it to interact with the
         filesystem or other services if it's meant to orchestrate the pipeline described in GEMINI.md.
</code></pre>

</div>



<h4>
  
  
  Summary
</h4>

<p>The Agent Development Kit was used to visually define a basic agent and added the Google Search Tool. This Agent was tested locally, and then deployed to Google Cloud Run. The Agent was code reviewed and validated with Gemini CLI.</p>

