---
Title: Build Your First ADK Agent Workforce
Description: 
Author: Mollie Pettit
Date: 2025-12-12T21:01:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>The world of <a href="https://cloud.google.com/use-cases/generative-ai?e=48754805&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Generative AI</a> is evolving rapidly, and <a href="https://cloud.google.com/discover/what-are-ai-agents?e=48754805&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">AI Agents</a> are at the forefront of this change. An AI agent is a software system designed to act on your behalf. They show reasoning, planning, and memory and have a level of autonomy to make decisions, learn, and adapt.</p>

<p>At its core, an AI agent uses a <a href="https://cloud.google.com/ai/llms?e=48754805&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">large language model (LLM)</a>, like <a href="https://ai.google.dev/gemini-api/docs/models?utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Gemini</a> to understand and reason. This allows it to process information from various sources, create a plan, and execute a series of tasks to reach a predefined objective. This is the key difference between a simple prompt-and-response and an agent: the ability to act on a multi-step plan.</p>

<p>The great news is that you can now easily build your own AI agents, even without deep expertise, thanks to <a href="https://google.github.io/adk-docs/" rel="noopener noreferrer">Agent Development Kit (ADK)</a>. ADK is an open-source <a href="https://google.github.io/adk-docs/get-started/python/" rel="noopener noreferrer">Python</a> and <a href="https://google.github.io/adk-docs/get-started/java/" rel="noopener noreferrer">Java</a> framework by Google designed to simplify agent creation.</p>

<p>To guide you, this post introduces three hands-on labs that cover the core patterns of agent development:</p>

<ol>
<li>Building your first autonomous agent</li>
<li>Empowering that agent with tools to interact with external services</li>
<li>Orchestrate a multi-agent system where specialized agents collaborate</li>
</ol>

<h2>
  
  
  Build your first agent
</h2>

<p><a href="https://codelabs.developers.google.com/devsite/codelabs/build-agents-with-adk-foundation?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">This lab</a> introduces the foundational principles of <a href="https://github.com/google/adk-python" rel="noopener noreferrer">ADK</a> by guiding you through the construction of a <strong>personal assistant agent</strong>.</p>

<p>You will write the code for the agent itself and will interact directly with the agent's core reasoning engine, powered by <a href="https://ai.google.dev/gemini-api/docs/models?utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Gemini</a>, to see how it responds to a simple request. This lab is focused on building the fundamental scaffolding of every agent you'll create.</p>


<div class="crayons-card c-embed">

  <br>
<strong>Go to the lab!</strong><br>
<strong>Lab:</strong> <a href="https://codelabs.developers.google.com/devsite/codelabs/build-agents-with-adk-foundation?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Building AI Agents with ADK:The Foundation</a>

<p><strong>Objective:</strong> <em>Write the essential code to define and run a basic agent, learning the core structure of ADK.</em><br>

</p>
</div>


<h2>
  
  
  Empower your agent with tools
</h2>

<p>An agent without custom tools can only rely on its built-in knowledge. To make it more powerful for your specific use-case, you can give it access to specialized tools. In this lab, you will learn three different ways to add tools:</p>

<ul>
<li>
<strong>Build a Custom Tool</strong>: Write a currency exchange tool from scratch.</li>
<li>
<strong>Integrate a Built-in Tool</strong>: Add ADK's pre-built <a href="https://google.github.io/adk-docs/tools/built-in-tools/#google-search" rel="noopener noreferrer">Google Search tool</a>.</li>
<li>
<strong>Leverage a Third-Party Tool</strong>: Import and use a <a href="https://python.langchain.com/docs/integrations/tools/wikipedia/" rel="noopener noreferrer">Wikipedia tool</a> from the LangChain library.</li>
</ul>


<div class="crayons-card c-embed">

  <br>
<strong>Go to the lab!</strong><br>
<strong>Lab:</strong> <a href="https://codelabs.developers.google.com/devsite/codelabs/build-agents-with-adk-empowering-with-tools?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Empower ADK Agents with Tools</a>

<p><strong>Objective:</strong> <em>Learn how to make your agent truly useful by giving it tools to interact with external applications and services.</em><br>

</p>
</div>


<h2>
  
  
  Build a Team of Specialized Agents
</h2>

<p>When a task is too complex for a single agent, you can build out a multi-agent team. This lab goes deep into the power of <a href="https://cloud.google.com/discover/what-is-a-multi-agent-system?e=48754805&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">multi-agent systems</a> by having you build a "movie pitch development team" that can research, write, and analyze a film concept.</p>

<p>You will learn how to use <a href="https://google.github.io/adk-docs/agents/workflow-agents/" rel="noopener noreferrer">ADK's Workflow Agents</a> to control the flow of work automatically, without needing user input at every step. You'll also learn how to use the session state to pass information between the agents.</p>


<div class="crayons-card c-embed">

  <br>
<strong>Go to the lab!</strong><br>
<strong>Lab:</strong> <a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/3-developing-agents/build-a-multi-agent-system-with-adk?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Build Multi-Agent Systems with ADK</a>

<p><strong>Objective:</strong> <em>Orchestrate a complex, automated workflow with a team of specialized agents that work in sequence, in loops, and in parallel.</em><br>

</p>
</div>


<h2>
  
  
  Summary: Build Your First AI Teammate Today
</h2>

<p>Ready to build your first <a href="https://cloud.google.com/discover/what-are-ai-agents?e=48754805&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">AI agents</a>? Dive into the codelabs from this post:</p>

<ul>
<li><a href="https://codelabs.developers.google.com/devsite/codelabs/build-agents-with-adk-foundation?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Building AI Agents with ADK: The Foundation</a></li>
<li><a href="https://codelabs.developers.google.com/devsite/codelabs/build-agents-with-adk-empowering-with-tools?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Empower ADK Agents with Tools</a></li>
<li><a href="https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/3-developing-agents/build-a-multi-agent-system-with-adk?hl=en#0&amp;utm_campaign=CDR_0x6e136736_default_b456239055&amp;utm_medium=external&amp;utm_source=blog" rel="noopener noreferrer">Build Multi-Agent Systems with ADK</a></li>
</ul>

<p>Share your progress and connect with others on the journey using the hashtag <strong>#ProductionReadyAI</strong>. Happy learning! </p>

<p>These labs are part of the <strong>Developing Agents</strong> module in our official <a href="https://cloud.google.com/blog/topics/developers-practitioners/production-ready-ai-with-google-cloud-learning-path" rel="noopener noreferrer">Production-Ready AI with Google Cloud</a> program. Explore the full curriculum for more content that will help you bridge the gap from a promising prototype to a production-grade AI application.</p>

