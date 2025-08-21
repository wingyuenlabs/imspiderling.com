---
Title: How to Build a Self-Correcting AI Agent for Product Search in E-Commerce
Description: 
Author: Chris Zhang
Date: 2025-08-21T20:26:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>Shopify just launched AI agents that let shoppers search, explore, and purchase using natural language.</p>

<p>If you’ve tried retrieval-augmented generation (RAG) pipelines for product search, you’ve probably hit the usual walls: vague results, brittle prompts, and silent failures when the data isn’t structured just right. When your catalog involves complex product descriptions, categorizations and multiple supporting documents, a basic retrieval or prompt-based approach just doesn’t cut it.</p>

<p>In the age of agentic commerce, how can we enable users to say things like “I have a small family of four. We live in Munich. What’s the best internet plan for us?” and have the system identify relevant products, draft an initial proposal, review and refine it based on available data, and engage in a meaningful conversation?</p>

<p>In this post, you’ll learn how to build a practical AI agent for searching product catalogs using <a href="https://upsidelab.io/tools/enthusiast" rel="noopener noreferrer">Enthusiast</a>, an AI toolkit designed for e-commerce and knowledge-intensive tasks. We will cover setting up the environment, customizing the agent, and quickly testing it on sample data.</p>

<p>But first, let’s look at how agentic workflows differ from traditional pipelines and why that matters.</p>

<h2>
  
  
  <strong>Non-Agentic Workflow vs. Agentic Workflow</strong>
</h2>

<p>In a traditional (non-agentic) workflow, product search is driven by fixed queries or rigid filter logic. It’s simple and fast, but struggles with nuanced language or evolving user intent. The system can’t adapt on the fly. It just follows predefined instructions.<br>
On the other hand, an agentic workflow introduces flexibility and adaptability. AI agents dynamically interpret user inputs, construct queries intelligently, and adjust their approach based on the context of the interaction and feedback received. This allows them to handle more complex, ambiguous requests while improving reliability and user experience.</p>
<h2>
  
  
  <strong>What Makes Up an AI Agent</strong>
</h2>

<p>To build an effective AI agent for product catalog search, the following components are essential:</p>

<ul>
<li>Input Handling: Accepts and interprets user requests.</li>
<li>Feedback Handling and Memory: Incorporates user and system feedback to improve future interactions and maintains memory of past interactions.</li>
<li>Tools: Interfaces with external tools or databases to execute tasks.</li>
<li>Reasoning: Analyzes input and feedback to make informed decisions.</li>
</ul>

<p>To build such an agent, we need an execution environment. Let’s explore how Enthusiast can serve as an effective option.</p>
<h2>
  
  
  <strong>Introducing Enthusiast</strong>
</h2>

<p>Most LangChain tutorials stop at toy examples or require heavy customization to support real-world workflows. Enthusiast changes that. It’s built from the ground up to support:</p>

<ul>
<li>Tool-based agents with LangChain and ReAct</li>
<li>SQL-backed querying with Django or external sources</li>
<li>Structured memory and retry logic out of the box</li>
<li>Open-source, customizable behavior</li>
<li>Self-hosting with cloud/local model support</li>
</ul>

<p>Whether you're debugging search in a product catalog or surfacing relevant documents across internal departments, Enthusiast gives you a working foundation in minutes with real production logic, not just playground demos.</p>
<h2>
  
  
  <strong>Build in action</strong>
</h2>

<p>Alright, now let’s bring that to life. We’ll walk through a real case: spinning up a local environment, loading data, and creating a self-correcting LangChain agent that actually understands and interacts with your product catalog.</p>

<p><strong>Setting Up the Development Environment</strong></p>

<p>To get started, you need to set up your development environment by cloning the Enthusiast starter repository and using its Docker configuration.</p>

<ol>
<li><p>Clone the repository:<br>
<code>git clone https://github.com/upsidelab/enthusiast-starter</code></p></li>
<li><p>Navigate into the repository directory:<br>
<code>cd enthusiast-starter</code></p></li>
<li><p>Copy default configuration file and add your own OpenAI API key:<br>
<code>cp config/env.sample config/env</code><br>
<code>echo OPENAI_API_KEY=xxxx &gt;&gt; config/env</code></p></li>
<li><p>Build and run the Docker containers:<br>
<code>docker compose up</code></p></li>
</ol>

<p>Once it’s running, open your browser and go to: <a href="http://localhost:10001" rel="noopener noreferrer">http://localhost:10001</a> Log in with the default credentials:****</p>

<ul>
<li>Email: <a href="mailto:admin@example.com">admin@example.com</a>
</li>
<li>Password: changeme</li>
</ul>

<p>You’ll be prompted to create your first dataset. Give it a name, for example, “My Dataset”. </p>

<p><strong>Import a Sample Product Dataset</strong><br>
Enthusiast comes with a sample set of products that can be useful if you want to get started quickly. In this case, we have a set of products that represent different phone and mobile plans - with varying internet speeds, data limits, landline access, cable TV options, and more. They make a great test case for experimenting with different approaches to agentic product recommendations. </p>

<p>Let’s import this into our dataset:</p>

<ol>
<li>Click on “Add Source” in the top-right corner of the screen.</li>
<li>From the dropdown, select “Product source”.</li>
<li>A popup will appear for configuring the source.</li>
<li>Select “Sample Product Source” from the list and click “Add”.</li>
<li>You should now see it listed under configured sources.</li>
<li>Repeat the same process for documents by selecting “Document source” from the dropdown.</li>
<li>This time, choose “Sample Document Source” as the type and add it as well.</li>
</ol>

<p>Enthusiast will automatically index the dataset so it’s searchable right away.</p>

<p>Once the data is loaded, you can go to the Products tab to verify that the sample data was successfully imported and indexed. This ensures that your dataset is ready for querying by the agent.</p>
<h2>
  
  
  <strong>Create a Custom Agent Structure</strong>
</h2>

<p>Now that your product catalog is loaded, it’s time to build an agent that can operate on it. Enthusiast supports extending and modifying agent behavior through the enthusiast_custom directory in the project.</p>

<ol>
<li><p>Inside the enthusiast-starter repository, locate the src/enthusiast_custom directory. This is the package that contains your custom agents and plugins. This code will be bundled by the Dockerfile and automatically installed into your Enthusiast instance.</p></li>
<li><p>Let’s also install a plugin that provides a reusable base implementation for a ReAct-style agent. Run the following command inside the src/ directory to add the plugin:<br>
<code>poetry add enthusiast-agent-re-act</code></p></li>
<li><p>Then, create a new directory inside enthusiast_custom, calling it for example product_search. Inside this directory, add an empty <strong>init</strong>.py file to make it a Python package. This is where you’ll define your agent’s implementation.</p></li>
<li><p>Add your new agent to the config/settings_override.py file so that Enthusiast can recognize it. Update the AVAILABLE_AGENTS dictionary to include your custom module:<br>
</p></li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">AVAILABLE_AGENTS</span> <span class="o">=</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">Product Search</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">enthusiast_custom.product_search</span><span class="sh">"</span>
<span class="p">}</span>
</code></pre>

</div>


<ol>
<li>You can now rebuild and restart your Docker Compose setup to apply these changes:
<code>docker compose up --build</code>
</li>
</ol>

<p>Once the application is restarted, you’ll see your new agent listed in the UI on the left. Time to give it some logic.</p>
<h2>
  
  
  <strong>Step 1 – Generate an SQL Query</strong>
</h2>

<p>We’ll start with a basic implementation that generates an SQL query and executes it on the product catalog indexed in Enthusiast. The agent will reason through user queries and interact with the catalog to retrieve relevant results.</p>

<p>To do this, we’ll use the enthusiast-agent-re-act plugin that we added earlier. It provides a BaseReActAgent class, which defines the core structure of a ReAct-style agent, including how it connects prompts, tools, memory, and output processing.</p>

<p>Here’s how we’ll structure the product_search agent module:<br>
<em>product_search/agent.py</em></p>

<p>Start by defining the agent class. In a basic scenario, no overrides are required - agent’s default implementation will respond to user’s queries by creating an agent executor configured with tools and memory, and will pass the user’s request there.<br>
Here’s what the simplest implementation looks like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">enthusiast_agent_re_act</span> <span class="kn">import</span> <span class="n">BaseReActAgent</span>

<span class="k">class</span> <span class="nc">ProductSearchAgent</span><span class="p">(</span><span class="n">BaseReActAgent</span><span class="p">):</span>
    <span class="k">pass</span>
</code></pre>

</div>



<p><code>product_search/product_search_tool.py</code></p>

<p>Next, implement a tool the agent can use to run SQL queries against your product catalog.</p>

<p>Let’s first declare the expected input schema using a Pydantic model. This schema will be provided to the agent together with the tool definition, to let the agent determine what’s needed to call this tool. Since we specify that the tool requires an SQL query, the agent will try to produce one based on everything it knows so far in order to invoke it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">pydantic</span> <span class="kn">import</span> <span class="n">BaseModel</span><span class="p">,</span> <span class="n">Field</span>

<span class="k">class</span> <span class="nc">ProductSearchToolInput</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">sql select query to execute on the catalog_product table. include both where and order clauses</span><span class="sh">"</span><span class="p">)</span>

</code></pre>

</div>



<p>This tool receives an SQL string from the agent, executes it using Django’s ORM, serializes the resulting product objects, and returns a message with the result. The NAME and DESCRIPTION fields in the tool definition help the agent determine when this tool is relevant to the current task. </p>

<p>Here’s a basic version of the tool implementation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">enthusiast_common.tools</span> <span class="kn">import</span> <span class="n">BaseLLMTool</span>
<span class="kn">from</span> <span class="n">catalog.models</span> <span class="kn">import</span> <span class="n">Product</span>
<span class="kn">from</span> <span class="n">django.core</span> <span class="kn">import</span> <span class="n">serializers</span>

<span class="k">class</span> <span class="nc">ProductSearchTool</span><span class="p">(</span><span class="n">BaseLLMTool</span><span class="p">):</span>
    <span class="n">NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">products_search_tool</span><span class="sh">"</span>
    <span class="n">DESCRIPTION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">use it to search for products</span><span class="sh">"</span>
    <span class="n">ARGS_SCHEMA</span> <span class="o">=</span> <span class="n">ProductSearchToolInput</span>
    <span class="n">RETURN_DIRECT</span> <span class="o">=</span> <span class="bp">False</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="n">products</span> <span class="o">=</span> <span class="n">Product</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">raw</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="nf">_sanitize_sql</span><span class="p">(</span><span class="n">sql_query</span><span class="p">))</span>
        <span class="n">serialized</span> <span class="o">=</span> <span class="n">serializers</span><span class="p">.</span><span class="nf">serialize</span><span class="p">(</span><span class="sh">"</span><span class="s">json</span><span class="sh">"</span><span class="p">,</span> <span class="nf">list</span><span class="p">(</span><span class="n">products</span><span class="p">))</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Found the following products: </span><span class="si">{</span><span class="n">serialized</span><span class="si">}</span><span class="s">. Notify the user.</span><span class="sh">"</span>
    <span class="k">def</span> <span class="nf">_sanitize_sql</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
               <span class="c1"># This is where you should add some extra safety checks to the SQL query before you execute it on a live database
</span>        <span class="k">return</span> <span class="n">sql_query</span>
</code></pre>

</div>



<p><code>product_search/prompt.py</code></p>

<p>Then, create the system prompt that will guide how the agent reasons and interacts with tools. Add the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">PRODUCT_FINDER_AGENT_PROMPT</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
I want you to help finding products using the ReACT (Reasoning and Acting) approach.
Always verify your answer
Use a json blob to specify a tool by providing an action key (tool name) and an action_input key (tool input).

Valid </span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="s"> values: {tool_names}

Provide only ONE action per $JSON_BLOB, as shown:


{{
  </span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="s">: $TOOL_NAME,
  </span><span class="sh">"</span><span class="s">action_input</span><span class="sh">"</span><span class="s">: $INPUT
}}

For each step, follow the format:
User query: the user</span><span class="sh">'</span><span class="s">s question or request
Thought: what you should do next
Action: 
{{
  </span><span class="sh">"</span><span class="s">action</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">&lt;tool&gt;</span><span class="sh">"</span><span class="s">,
  </span><span class="sh">"</span><span class="s">action_input</span><span class="sh">"</span><span class="s">: {{</span><span class="sh">"</span><span class="s">&lt;tool_argument_name&gt;</span><span class="sh">"</span><span class="s">: </span><span class="sh">"</span><span class="s">&lt;tool_argument_value&gt;</span><span class="sh">"</span><span class="s">, ...}}
}}
Observation: the result returned by the tool
</span><span class="gp">...</span> <span class="p">(</span><span class="n">repeat</span> <span class="n">Thought</span><span class="o">/</span><span class="n">Action</span><span class="o">/</span><span class="n">Action</span> <span class="n">Input</span><span class="o">/</span><span class="n">Observation</span> <span class="k">as</span> <span class="n">needed</span><span class="p">)</span>
<span class="n">Thought</span><span class="p">:</span> <span class="n">I</span> <span class="n">now</span> <span class="n">have</span> <span class="n">the</span> <span class="n">necessary</span> <span class="n">information</span>
<span class="n">Final</span> <span class="n">Answer</span><span class="p">:</span> <span class="n">the</span> <span class="n">response</span> <span class="n">to</span> <span class="n">the</span> <span class="n">user</span>

<span class="s">Here are the tools you can use:
{tools}

Do not come up with any other types of JSON than specified above.
Your output to user should always begin with </span><span class="sh">'''</span><span class="s">Final Answer: &lt;output&gt;</span><span class="sh">'''</span><span class="s">

Begin!
Chat history: {chat_history}
User query: {input}
{agent_scratchpad}</span><span class="sh">"""</span>
</code></pre>

</div>



<p><code>product_search/config.py</code></p>

<p>Finally, wire everything together in the config file. This tells Enthusiast which components make up your agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">get_config</span><span class="p">(</span><span class="n">conversation_id</span><span class="p">:</span> <span class="nb">int</span><span class="p">,</span> <span class="n">streaming</span><span class="p">:</span> <span class="nb">bool</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">AgentConfigWithDefaults</span><span class="p">:</span>
    <span class="k">return</span> <span class="nc">AgentConfigWithDefaults</span><span class="p">(</span>
        <span class="n">conversation_id</span><span class="o">=</span><span class="n">conversation_id</span><span class="p">,</span>
        <span class="n">agent_class</span><span class="o">=</span><span class="n">ProductSearchAgent</span><span class="p">,</span>
        <span class="n">llm_tools</span><span class="o">=</span><span class="p">[</span>
            <span class="nc">LLMToolConfig</span><span class="p">(</span>
                <span class="n">tool_class</span><span class="o">=</span><span class="n">AgentProductSearchTool</span><span class="p">,</span>
            <span class="p">),</span>
        <span class="p">],</span>
        <span class="n">prompt_template</span><span class="o">=</span><span class="n">ChatPromptTemplate</span><span class="p">.</span><span class="nf">from_messages</span><span class="p">(</span>
            <span class="p">[</span>
                <span class="p">(</span>
                    <span class="sh">"</span><span class="s">system</span><span class="sh">"</span><span class="p">,</span>
                    <span class="n">PRODUCT_FINDER_AGENT_PROMPT</span><span class="p">,</span>
                <span class="p">),</span>
            <span class="p">]</span>
        <span class="p">),</span>
        <span class="n">llm</span><span class="o">=</span><span class="nc">LLMConfig</span><span class="p">(</span>
            <span class="n">callbacks</span><span class="o">=</span><span class="p">[</span><span class="nc">ReactAgentWebsocketCallbackHandler</span><span class="p">(</span><span class="n">conversation_id</span><span class="p">)],</span>
            <span class="n">streaming</span><span class="o">=</span><span class="n">streaming</span><span class="p">,</span>
        <span class="p">),</span>
        <span class="n">agent_callback_handler</span><span class="o">=</span><span class="nc">AgentCallbackHandlerConfig</span><span class="p">(</span>
            <span class="n">handler_class</span><span class="o">=</span><span class="n">AgentActionWebsocketCallbackHandler</span><span class="p">,</span> <span class="n">args</span><span class="o">=</span><span class="p">{</span><span class="sh">"</span><span class="s">conversation_id</span><span class="sh">"</span><span class="p">:</span> <span class="n">conversation_id</span><span class="p">}</span>
        <span class="p">),</span>
    <span class="p">)</span>
</code></pre>

</div>



<p>Once these components are in place and the Docker container is rebuilt, try executing a sample query:<br>
What’s the best plan for a small family?<br>
The agent will reason about the input, construct an SQL query, and invoke the search tool, likely failing due to invalid schema or search criteria. Let’s see what we can do with that.</p>
<h2>
  
  
  <strong>Step 2 – Let the Agent Handle Its Own Errors</strong>
</h2>

<p>In the initial version, if the SQL query generated by the agent was incorrect, the tool would simply fail without giving the agent any indication of what went wrong. We can improve this by modifying the tool to catch SQL errors and return the error message as part of the response.</p>

<p>This way, the agent can treat the error as feedback and make another attempt, refining the query on its own.</p>

<p>To do this, update the run method in ProductSearchTool as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">django.core</span> <span class="kn">import</span> <span class="n">serializers</span>
<span class="kn">import</span> <span class="n">logging</span>

<span class="n">logger</span> <span class="o">=</span> <span class="n">logging</span><span class="p">.</span><span class="nf">getLogger</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">ProductSearchTool</span><span class="p">(</span><span class="n">BaseLLMTool</span><span class="p">):</span>
    <span class="n">NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">products_search_tool</span><span class="sh">"</span>
    <span class="n">DESCRIPTION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">use it to search for products</span><span class="sh">"</span>
    <span class="n">ARGS_SCHEMA</span> <span class="o">=</span> <span class="n">ProductSearchToolInput</span>
    <span class="n">RETURN_DIRECT</span> <span class="o">=</span> <span class="bp">False</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">products</span> <span class="o">=</span> <span class="n">Product</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">raw</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="nf">_sanitize_sql</span><span class="p">(</span><span class="n">sql_query</span><span class="p">))</span>
            <span class="n">serialized</span> <span class="o">=</span> <span class="n">serializers</span><span class="p">.</span><span class="nf">serialize</span><span class="p">(</span><span class="sh">"</span><span class="s">json</span><span class="sh">"</span><span class="p">,</span> <span class="nf">list</span><span class="p">(</span><span class="n">products</span><span class="p">))</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Found the following products: </span><span class="si">{</span><span class="n">serialized</span><span class="si">}</span><span class="s">. Notify the user.</span><span class="sh">"</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="n">e</span><span class="p">)</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">The query you</span><span class="sh">'</span><span class="s">ve generated is incorrect. Error </span><span class="si">{</span><span class="nf">type</span><span class="p">(</span><span class="n">e</span><span class="p">).</span><span class="n">__name__</span><span class="si">}</span><span class="s"> - </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="s">. Please fix it and try again. Make sure it</span><span class="sh">'</span><span class="s">s a valid SQL.</span><span class="sh">"</span>
</code></pre>

</div>



<p>With this change, when the SQL query fails, the agent gets the error message and can use it to revise its approach. Since the agent maintains memory of previous steps, it can iterate on its output to try and produce a valid query.</p>

<p>Try running the same query again:<br>
What’s the best plan for a small family?</p>

<p>If the first attempt fails, the agent will receive the error, analyze it, and try to generate a better query.</p>
<h2>
  
  
  <strong>Step 3 – Help the Agent Understand the Data</strong>
</h2>

<p>Letting the agent correct its own mistakes is helpful, but trial and error can be inefficient. Instead of waiting for the agent to fail and recover, we can give it a clearer understanding of the data structure up front.</p>

<p>One simple way to do this is by including a few sample rows from the product catalog directly in the prompt. This helps the agent understand both the schema and the shape of the data, which improves its chances of generating valid queries from the start.</p>

<p>To add this context, let’s override the get_answer method in your agent like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ProductSearchAgent</span><span class="p">(</span><span class="n">BaseReActAgent</span><span class="p">):</span>
    <span class="k">def</span> <span class="nf">get_answer</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">input_text</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
        <span class="n">sample_products</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">_injector</span><span class="p">.</span><span class="n">product_retriever</span><span class="p">.</span><span class="nf">_get_sample_products_json</span><span class="p">()</span>
        <span class="n">agent_executor</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="nf">_build_agent_executor</span><span class="p">()</span>
        <span class="n">agent_output</span> <span class="o">=</span> <span class="n">agent_executor</span><span class="p">.</span><span class="nf">invoke</span><span class="p">(</span>
            <span class="p">{</span><span class="sh">"</span><span class="s">input</span><span class="sh">"</span><span class="p">:</span> <span class="n">input_text</span><span class="p">,</span> <span class="sh">"</span><span class="s">sample_products</span><span class="sh">"</span><span class="p">:</span> <span class="n">sample_products</span><span class="p">},</span>
            <span class="n">config</span><span class="o">=</span><span class="n">self</span><span class="p">.</span><span class="nf">_build_invoke_config</span><span class="p">()</span>
        <span class="p">)</span>
        <span class="k">return</span> <span class="n">agent_output</span><span class="p">[</span><span class="sh">"</span><span class="s">output</span><span class="sh">"</span><span class="p">]</span>

</code></pre>

</div>



<p>This method will use functionality provided by the base class to build a LangChain-based agent executor, pass the input to it, and return the response to the user. One important change here is that besides user’s input (passed as input_text ), it will also pull a few sample products from the database and will inject them into the agent’s system prompt as sample_products.</p>

<p>In your prompt template (prompt.py), add this placeholder at the end:<br>
Here are some sample products in the database: {sample_products}</p>

<p>This additional context will be included with every call to the agent. It initializes the agent with a basic understanding of the structure and shape of the data, which makes it easier for the agent to construct accurate queries from the start.<br>
Let’s give it a try.</p>

<p>You should notice that the agent now constructs queries that better match how the data is shaped. For example, it may use the category column to search for plans labeled as “Home,” or rely on the properties column to filter for plans with specific internet speeds.</p>
<h2>
  
  
  <strong>Step 4 – Retry When No Results Are Found</strong>
</h2>

<p>Even if the agent is capable of generating valid SQL queries and has seen sample data, there’s still a chance it will produce a query that technically runs but returns no results.</p>

<p>In the current implementation, when that happens, the tool simply returns an empty list, and the agent assumes there are no relevant options. But in reality, the issue may be with how the agent built the query, not with a lack of products.</p>

<p>To address this, we can update the tool to return a clear message when no products are found—encouraging the agent to try a different approach. Here’s how the updated run method might look:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">ProductSearchTool</span><span class="p">(</span><span class="n">BaseLLMTool</span><span class="p">):</span>
    <span class="n">NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">products_search_tool</span><span class="sh">"</span>
    <span class="n">DESCRIPTION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">use it to search for products</span><span class="sh">"</span>
    <span class="n">ARGS_SCHEMA</span> <span class="o">=</span> <span class="n">ProductSearchToolInput</span>
    <span class="n">RETURN_DIRECT</span> <span class="o">=</span> <span class="bp">False</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">products</span> <span class="o">=</span> <span class="n">Product</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">raw</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="nf">_sanitize_sql</span><span class="p">(</span><span class="n">sql_query</span><span class="p">))</span>
            <span class="n">products</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">products</span><span class="p">)</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="n">e</span><span class="p">)</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">The query you</span><span class="sh">'</span><span class="s">ve generated is incorrect. Error </span><span class="si">{</span><span class="nf">type</span><span class="p">(</span><span class="n">e</span><span class="p">).</span><span class="n">__name__</span><span class="si">}</span><span class="s"> - </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="s">. Please fix it and try again. Make sure it</span><span class="sh">'</span><span class="s">s a valid SQL.</span><span class="sh">"</span>

        <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">products</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
            <span class="n">serialized</span> <span class="o">=</span> <span class="n">serializers</span><span class="p">.</span><span class="nf">serialize</span><span class="p">(</span><span class="sh">"</span><span class="s">json</span><span class="sh">"</span><span class="p">,</span> <span class="n">products</span><span class="p">)</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Found the following products: </span><span class="si">{</span><span class="n">serialized</span><span class="si">}</span><span class="s">. Notify the user.</span><span class="sh">"</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">No products found using this search criteria. Try using a different query and try again.</span><span class="sh">"</span>
</code></pre>

</div>



<p>With this change, the agent receives explicit feedback when a query returns no matches. It can then choose to revise the query and try again with broader or alternative criteria.</p>

<p>This gives the agent an opportunity to step back and reconsider its assumptions, leading to better resilience and more accurate results when dealing with uncertain or ambiguous user requests.</p>

<h2>
  
  
  <strong>Step 5 – Respect the Expected Number of Results</strong>
</h2>

<p>In some cases, a user might indicate how many products they want to see—perhaps just one recommendation or the top three matches. Right now, the agent doesn’t take that into account. It may return a long list of results, even if the user only wanted a few.</p>

<p>We can improve this by passing the expected number of results as part of the tool input. The tool will then check whether the number of matches exceeds this limit. If it does, it will prompt the agent to follow up and narrow the criteria.</p>

<p>First, update the input schema to include this new parameter:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">AgentProductSearchToolInput</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">sql select query to execute on the catalog_product table. include both where and order clauses</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">result_number</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="n">description</span><span class="o">=</span><span class="sh">"</span><span class="s">the number of results that the user wants to get</span><span class="sh">"</span><span class="p">)</span>
<span class="n">Then</span><span class="p">,</span> <span class="n">update</span> <span class="n">the</span> <span class="n">run</span> <span class="n">method</span> <span class="ow">in</span> <span class="n">the</span> <span class="n">tool</span> <span class="n">to</span> <span class="n">handle</span> <span class="n">this</span> <span class="nb">input</span><span class="p">:</span>
<span class="k">class</span> <span class="nc">ProductSearchTool</span><span class="p">(</span><span class="n">BaseLLMTool</span><span class="p">):</span>
    <span class="n">NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">products_search_tool</span><span class="sh">"</span>
    <span class="n">DESCRIPTION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">use it to search for products</span><span class="sh">"</span>
    <span class="n">ARGS_SCHEMA</span> <span class="o">=</span> <span class="n">AgentProductSearchToolInput</span>
    <span class="n">RETURN_DIRECT</span> <span class="o">=</span> <span class="bp">False</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">sql_query</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">result_number</span><span class="p">:</span> <span class="nb">int</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">products</span> <span class="o">=</span> <span class="n">Product</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">raw</span><span class="p">(</span><span class="n">self</span><span class="p">.</span><span class="nf">_sanitize_sql</span><span class="p">(</span><span class="n">sql_query</span><span class="p">))</span>
            <span class="n">products</span> <span class="o">=</span> <span class="nf">list</span><span class="p">(</span><span class="n">products</span><span class="p">)</span>
        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">logger</span><span class="p">.</span><span class="nf">info</span><span class="p">(</span><span class="n">e</span><span class="p">)</span>
            <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">The query you</span><span class="sh">'</span><span class="s">ve generated is incorrect. Error </span><span class="si">{</span><span class="nf">type</span><span class="p">(</span><span class="n">e</span><span class="p">).</span><span class="n">__name__</span><span class="si">}</span><span class="s"> - </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="s">. Please fix it and try again. Make sure it</span><span class="sh">'</span><span class="s">s a valid SQL.</span><span class="sh">"</span>

        <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">products</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">:</span>
            <span class="n">serialized_products</span> <span class="o">=</span> <span class="n">serializers</span><span class="p">.</span><span class="nf">serialize</span><span class="p">(</span><span class="sh">"</span><span class="s">json</span><span class="sh">"</span><span class="p">,</span> <span class="n">products</span><span class="p">)</span>
            <span class="k">if</span> <span class="nf">len</span><span class="p">(</span><span class="n">products</span><span class="p">)</span> <span class="o">&gt;</span> <span class="n">result_number</span><span class="p">:</span>
                <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Found the following products: </span><span class="si">{</span><span class="n">serialized_products</span><span class="si">}</span><span class="s">. However, the user would like to get at most </span><span class="si">{</span><span class="n">result_number</span><span class="si">}</span><span class="s">. Ask a follow up question that will make it possible to limit the amount of results further. Don</span><span class="sh">'</span><span class="s">t return these products to the user yet.</span><span class="sh">"</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Found the following products: </span><span class="si">{</span><span class="n">serialized_products</span><span class="si">}</span><span class="s">. Notify the user.</span><span class="sh">"</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">No products found using this search criteria. Try using a different query and try again.</span><span class="sh">"</span>
</code></pre>

</div>



<p>This addition helps turn the agent into a more effective product search assistant. Instead of assuming that the initial results are appropriate, the agent now reflects on the quantity of data returned, checks it against user expectations, and adjusts accordingly. This creates a more collaborative flow where the agent and user refine the query together to land on a relevant result.</p>

<h2>
  
  
  <strong>Step 6 – Enable the Agent to Finalize a Purchase</strong>
</h2>

<p>Once the user finds a plan that matches their needs, the next logical step is to help them act on it. Right now, our agent can recommend products but doesn’t support any kind of checkout process.</p>

<p>To make this possible, we’ll give the agent the ability to generate a contract URL the user can follow to finalize their purchase. This effectively allows the agent to transition from discovery to action.</p>

<p>Start by creating a new tool, PurchaseTool, which accepts a plan_sku and returns a contract finalization link:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">enthusiast_common.tools</span> <span class="kn">import</span> <span class="n">BaseLLMTool</span>
<span class="kn">from</span> <span class="n">pydantic</span> <span class="kn">import</span> <span class="n">BaseModel</span><span class="p">,</span> <span class="n">Field</span>

<span class="k">class</span> <span class="nc">PurchaseToolInput</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">plan_sku</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="sh">"</span><span class="s">SKU of the plan that the customer wants to purchase</span><span class="sh">"</span><span class="p">)</span>

<span class="k">class</span> <span class="nc">PurchaseTool</span><span class="p">(</span><span class="n">BaseLLMTool</span><span class="p">):</span>
    <span class="n">NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">purchase_tool</span><span class="sh">"</span>
    <span class="n">DESCRIPTION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">use when the user wants to sign a contract for a plan</span><span class="sh">"</span>
    <span class="n">ARGS_SCHEMA</span> <span class="o">=</span> <span class="n">PurchaseToolInput</span>
    <span class="n">RETURN_DIRECT</span> <span class="o">=</span> <span class="bp">False</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">plan_sku</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">https://fiberup.upsidelab.net/finalize_contract?</span><span class="si">{</span><span class="n">plan_sku</span><span class="si">}</span><span class="sh">"</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Show the following url to the user, and ask them to use it to finalize their signature </span><span class="si">{</span><span class="n">url</span><span class="si">}</span><span class="sh">"</span>
<span class="n">Then</span><span class="p">,</span> <span class="n">register</span> <span class="n">this</span> <span class="n">tool</span> <span class="ow">in</span> <span class="n">your</span> <span class="n">agent</span><span class="err">’</span><span class="n">s</span> <span class="n">config</span><span class="p">:</span>
<span class="kn">from</span> <span class="n">.purchase_tool</span> <span class="kn">import</span> <span class="n">PurchaseTool</span>
<span class="bp">...</span>
<span class="n">llm_tools</span><span class="o">=</span><span class="p">[</span>
    <span class="nc">LLMToolConfig</span><span class="p">(</span><span class="n">tool_class</span><span class="o">=</span><span class="n">AgentProductSearchTool</span><span class="p">),</span>
    <span class="nc">LLMToolConfig</span><span class="p">(</span><span class="n">tool_class</span><span class="o">=</span><span class="n">PurchaseTool</span><span class="p">),</span>
<span class="p">],</span>
</code></pre>

</div>



<p>Lastly, modify the search tool’s return message slightly to encourage the agent to propose a contract. The agent will likely figure it out even without this hint, but there’s no harm in pushing it more explicitly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Found the following products: </span><span class="si">{</span><span class="n">serialized_products</span><span class="si">}</span><span class="s">. Notify the user and propose signing a contract for this plan.</span><span class="sh">"</span>
</code></pre>

</div>



<p>With this addition, your agent becomes a guided assistant that helps the user discover a suitable plan and smoothly transition into completing the purchase.</p>

<h2>
  
  
  <strong>Step 7 – Ask for Additional Customer Details</strong>
</h2>

<p>Before the agent pushes the user to sign a contract, it can also ensure that it collects any additional information needed to complete the process—such as the customer’s name and location.<br>
To support this, update the PurchaseToolInput schema with two new fields:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">PurchaseToolInput</span><span class="p">(</span><span class="n">BaseModel</span><span class="p">):</span>
    <span class="n">plan_sku</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="sh">"</span><span class="s">SKU of the plan that the customer wants to purchase</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">customer_name</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="sh">"</span><span class="s">Name of the customer</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">zipcode</span><span class="p">:</span> <span class="nb">str</span> <span class="o">=</span> <span class="nc">Field</span><span class="p">(</span><span class="sh">"</span><span class="s">Zipcode of the customer</span><span class="sh">"</span><span class="p">)</span>
<span class="n">Then</span> <span class="n">modify</span> <span class="n">the</span> <span class="n">run</span> <span class="n">method</span> <span class="n">to</span> <span class="n">include</span> <span class="n">this</span> <span class="n">information</span> <span class="ow">in</span> <span class="n">the</span> <span class="n">contract</span> <span class="n">URL</span><span class="p">:</span>
<span class="kn">import</span> <span class="n">urllib.parse</span>

<span class="k">class</span> <span class="nc">PurchaseTool</span><span class="p">(</span><span class="n">BaseLLMTool</span><span class="p">):</span>
    <span class="n">NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">purchase_tool</span><span class="sh">"</span>
    <span class="n">DESCRIPTION</span> <span class="o">=</span> <span class="sh">"</span><span class="s">use when the user wants to sign a contract for a plan</span><span class="sh">"</span>
    <span class="n">ARGS_SCHEMA</span> <span class="o">=</span> <span class="n">PurchaseToolInput</span>
    <span class="n">RETURN_DIRECT</span> <span class="o">=</span> <span class="bp">False</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">plan_sku</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">customer_name</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">zipcode</span><span class="p">:</span> <span class="nb">str</span><span class="p">):</span>
        <span class="n">url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">https://fiberup.upsidelab.net/finalize_contract?</span><span class="si">{</span><span class="n">plan_sku</span><span class="si">}</span><span class="s">&amp;name=</span><span class="si">{</span><span class="n">urllib</span><span class="p">.</span><span class="n">parse</span><span class="p">.</span><span class="nf">quote_plus</span><span class="p">(</span><span class="n">customer_name</span><span class="p">)</span><span class="si">}</span><span class="s">&amp;zipcode=</span><span class="si">{</span><span class="n">zipcode</span><span class="si">}</span><span class="sh">"</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Show the following url to the user, and ask them to use it to finalize their signature </span><span class="si">{</span><span class="n">url</span><span class="si">}</span><span class="sh">"</span>
</code></pre>

</div>



<p>Thanks to the structured schema and tool description, the agent will know that it must collect these inputs from the user before invoking the tool. If the information isn’t provided initially, the agent can follow up with questions like:</p>

<p>Could you tell me your name and zip code so I can finalize the contract?</p>

<p>This closes the loop and ensures that the agent not only helps discover the right plan but can also guide the user through to a complete and personalized purchase process.</p>

<h2>
  
  
  <strong>Summary</strong>
</h2>

<p>In this walkthrough, we explored how to build a practical AI agent for product catalog search using Enthusiast. Starting from a basic ReAct-style agent capable of generating SQL queries, we incrementally introduced more sophisticated behaviors:</p>

<ul>
<li>Error recovery through exception feedback</li>
<li>Schema-aware reasoning via sample data</li>
<li>Retry logic when no results are found</li>
<li>Adapting results to match user expectations</li>
<li>Finalizing user purchases with structured follow-up</li>
<li>Collecting required customer details before contract generation</li>
</ul>

<p>Each step was designed to bring the agent closer to an experience that feels like a helpful, iterative assistant.</p>

<p>To try this yourself, check out the Enthusiast starter repository: <a href="https://github.com/upsidelab/enthusiast" rel="noopener noreferrer">https://github.com/upsidelab/enthusiast</a></p>

<p>For full documentation and examples, visit: <a href="https://upsidelab.io/tools/enthusiast" rel="noopener noreferrer">https://github.com/upsidelab/enthusiast</a></p>

