---
Title: Building Your First MCP Server in Python
Description: 
Author: OnlineProxy
Date: 2025-12-16T20:48:36.000Z
Robots: noindex,nofollow
Template: index
---
<p>We have reached a pivotal moment in the evolution of Large Language Models (LLMs). For a long time, we treated them as "brains in a jar"—brilliant reasoning engines disconnected from the world, capable only of generating text based on training data. The Model Context Protocol (MCP) changes this paradigm entirely. It provides a standardized interface for these models to perceive (Resources), act (Tools), and function within specific interaction patterns (Prompts).</p>

<p>If you are a senior engineer or a developer looking to move beyond basic chatbot wrappers, building an MCP server is the critical next step. It allows you to expose local files, databases, and executable logic to clients like Claude Desktop or IDEs like Cursor.</p>

<p>This guide explores the architecture of a full-featured MCP server using the Python SDK. We will prioritize the <code>FastMCP</code> framework for its developer ergonomics, enabling us to implement tools, resources, and prompts in a cohesive system.</p>

<h2>
  
  
  Why Construct a Server from Scratch?
</h2>

<p>Before writing a single line of code, we must ask: Is this necessary? If a server for your specific need (e.g., a PostgreSQL interface) already exists in the open-source ecosystem, the senior engineering decision is often to use the existing solution. Redundancy is technical debt.</p>

<p>However, custom business logic, specific local file manipulation, or unique workflow automations require bespoke servers. The Python SDK offers the most streamlined path for this, particularly for data-heavy operations. While TypeScript is a valid option, Python’s dominance in the AI space makes it the natural choice for integration.</p>

<p>We will build a "Multitool" server—a comprehensive implementation demonstrating the three core pillars of MCP:</p>

<ol>
<li>
<strong>Tools</strong>: Executable functions (e.g., a calculator).</li>
<li>
<strong>Resources</strong>: Read-only context (e.g., documentation files).</li>
<li>
<strong>Prompts</strong>: Templated interactions (e.g., meeting summaries).</li>
</ol>

<h2>
  
  
  The Trinity of Logic: Tools, Resources, and Prompts
</h2>

<p>To understand the architecture, we must visualize the user flow. An LLM client connects to your server. It queries the capabilities. Your server responds with a list of tools it can call, resources it can read, and prompt templates the user can trigger.</p>

<p><strong>1. The Execution Layer: Tools</strong><br>
Tools are the hands of the model. They allow the LLM to perform calculations, execute scripts, or fetch dynamic data.</p>

<p><strong>2. The Context Layer: Resources</strong><br>
Resources are the eyes of the model. They expose data—logs, code files, or database records—as direct context. Unlike tools, resources are generally read-only and passive.</p>

<p><strong>3. The Interaction Layer: Prompts</strong><br>
Prompts are predefined workflows. Instead of typing "You are an executive assistant, please summarize this..." every time, the server exposes a "Meeting Summary" template with dynamic arguments.</p>
<h2>
  
  
  Step-by-Step Implementation Guide
</h2>

<p>We will use <code>uv</code> for dependency management and <code>fastmcp</code> to scaffold the server. This modern Python stack avoids the bloat of traditional virtual environment management.</p>

<p>Phase 1: Environment and Initialization<br>
First, verify your Python version. We are targeting Python 3.12+.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>python --version
</code></pre>

</div>



<p>Next, install <code>uv</code>, a high-performance Python package installer and resolver.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Verify installation
uv --version
</code></pre>

</div>



<p>Initialize your project structure. This creates a clean workspace without the "dependency hell" often associated with Python projects.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>uv init .
uv venv
source .venv/bin/activate # On Windows: .venv\Scripts\activate
uv add mcp[cli]
</code></pre>

</div>



<p><strong>Phase 2: The Core Logic (<code>server.py</code>)</strong><br>
We create <code>server.py</code> as our entry point. We import FastMCP and instantiate our server.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from mcp.server.fastmcp import FastMCP
import math

# Initialize the server
mcp = FastMCP("Calculator Server")
</code></pre>

</div>



<h2>
  
  
  Implementing Mathematical Tools
</h2>

<p>The most reliable way to test an MCP server is with deterministic logic. We will implement a calculator. Note that for the LLM to use these tools effectively, <strong>type hints and docstrings are mandatory</strong>. The detailed description in the docstring is what the model uses to decide when to call the tool.</p>

<p>We define basic arithmetic and slightly more complex operations. For example, a square root function must handle edge cases, such as negative numbers, by returning descriptive strings rather than raising unhandled exceptions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@mcp.tool()
def add(a: int, b: int) -&gt; int:
    """Add two numbers together."""
    return a + b

@mcp.tool()
def multiply(a: int, b: int) -&gt; int:
    """Multiply two numbers together."""
    return a * b

@mcp.tool()
def sqrt(x: float) -&gt; str:
    """Calculate the square root of a number."""
    if x &lt; 0:
        return "Cannot calculate square root of a negative number"
    return str(math.sqrt(x))
</code></pre>

</div>



<p>Mathematically, we are enabling the LLM to interpret natural language requests like "What is the square root of 144?", while our code handles the computation.</p>

<p><strong>Phase 3: Integrating Resources</strong><br>
Resources allow us to attach static content. For this implementation, let’s assume we have a Markdown file on our desktop containing technical documentation (e.g., a TypeScript SDK guide) that we want the LLM to reference.</p>

<p>We use the <code>@mcp.resource</code> decorator. The critical component here is the URI schema (<code>resource://...</code>).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@mcp.resource("resource://docs/typescript-sdk")
def get_typescript_sdk_docs() -&gt; str:
    """Reads the TypeScript SDK documentation."""
    # Implementation reading local .md file
    path = "./docs/typescript_sdk.md" 
    try:
        with open(path, "r") as f:
            return f.read()
    except FileNotFoundError:
        return "Documentation file not found."
</code></pre>

</div>



<p>This acts as a retrieval pipe. When the user asks a question about the SDK, the model can pull this resource into its context window automatically.</p>

<p><strong>Phase 4: Constructing Dynamic Prompts</strong><br>
Prompts are perhaps the most underutilized aspect of MCP. They act as "saved searches" or "macros" for the LLM.</p>

<p>We will create a <strong>Meeting Summary</strong> prompt. This requires dynamic arguments: <code>date</code>, <code>title</code>, and <code>transcript</code>. Instead of raw text manipulation, we define the prompt to accept these variables and inject them into a structured narrative.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@mcp.prompt()
def meeting_summary(date: str, title: str, transcript: str) -&gt; str:
    """Generates a structured meeting summary."""
    return f"""
    You are an executive assistant. Analyze the following meeting.

    Data: {date}
    Title: {title}

    Transcript:
    {transcript}

    Provide a comprehensive analysis including participants and key decisions.
    """
</code></pre>

</div>



<p>In the UI (like Claude Desktop), this renders as a form. The user enters the variables, and the server constructs the final prompt sent to the model.</p>

<h2>
  
  
  Debugging: The MCP Inspector
</h2>

<p>Developing strictly via "trial and error" inside a chat interface is inefficient. You need to restart the host application repeatedly to pick up code changes.</p>

<p>The <strong>MCP Inspector</strong> is a browser-based debugging suite. It allows you to simulate a client connection, list capabilities, and invoke tools manually.</p>

<p>To launch the inspector against your local code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>uv run mcp-dev server.py
# Or directly naming the inspector if installed via npm/npx:
npx @modelcontextprotocol/inspector uv run server.py
</code></pre>

</div>



<p>The inspector runs a proxy server on a specific port (e.g., <code>localhost:3000</code>). It provides specific tabs for:</p>

<ol>
<li>
<strong>Tools</strong>: You can input <code>a=10, b=2</code>and execute add.</li>
<li>
<strong>Resources</strong>: View the loaded text content of your resources.</li>
<li>
<strong>Prompts</strong>: Fill in the template fields and view the generated text payload.</li>
</ol>

<p><strong>Crucial Insight</strong>: If you utilize the Inspector via the provided URL (e.g., <code>localhost:5173</code>), ensure you check the terminal for the session token/URL generated. Browsers enforce security on local connections; opening the link directly from the terminal with the token included helps bypass authorization hurdles.</p>

<h2>
  
  
  Configuration: Connecting to Claude Desktop
</h2>

<p>Once the server passes the Inspector's tests, we integrate it into a production-style environment like Claude Desktop. This requires a configuration file located at:</p>

<ul>
<li>Windows: <code>%APPDATA%\Claude\claude_desktop_config.json</code>
</li>
<li>MacOS: <code>~/Library/Application Support/Claude/claude_desktop_config.json</code>
The configuration uses the <code>stdio</code> transport layer by default. This connects the standard input/output of the Python script to the host application.
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "mcpServers": {
    "calculator-server": {
      "command": "uv",
      "args": [
        "run",
        "server.py"
      ],
      "env": {
        "PYTHONUTF8": "1"
      }
    }
  }
}
</code></pre>

</div>



<p><strong>Developer Note</strong>: Ensure the pathing is absolute or that the cwd (current working directory) is handled correctly within your script if you are not pointing explicitly to the file path.</p>

<h2>
  
  
  Transport Layers: Stdio vs. SSE vs. Streamable HTTP
</h2>

<p>The protocol supports different transport mechanisms.</p>

<ol>
<li>
<strong>Stdio (Standard Input/Output)</strong>: The default for local processes. It is fast, relies on process piping, and requires no networking ports. This is ideal for local desktop integration.</li>
<li>
<strong>SSE (Server-Sent Events)</strong>: Previously a standalone transport, it is now often wrapped within streamable HTTP. It allows for unidirectional updates from server to client.</li>
<li>
<strong>Streamable HTTP</strong>: The modern standard for remote MCP servers. It uses POST requests for client-to-server messages and SSE for server-to-client messages.</li>
</ol>

<p>If you wish to expose your server over a network (e.g., controlling a remote virtual machine), you must switch from <code>stdio</code> to http.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># In server.py
if __name__ == "__main__":
    # Allows switching based on deployment needs
    mcp.run(transport="stdio") 
    # Or for network exposure:
    # mcp.run(transport="sse")
</code></pre>

</div>



<p>When debugging <code>http</code>/<code>sse</code> via the Inspector, the connection URL changes. You must reference the specific endpoint (often ending in <code>/mcp</code> or <code>/sse</code>), and you generally need to handle CORS and authentication if moving beyond <code>localhost</code>.</p>

<h2>
  
  
  The "Vibe Coding" Workflow
</h2>

<p>A recurring theme in modern development is LLM-assisted coding—or "vibe coding." When building this MCP server, we do not write every boilerplate line manually. Instead, we act as architects.</p>

<p>We feed the context documentation (the MCP SDK <code>README.md</code> or <code>llms.txt</code>) into an IDE like Cursor. We then prompt the model:</p>

<blockquote>
<p>"I want to create an MCP server with a calculator tool. Use the provided docs. Structure it with FastMCP."</p>
</blockquote>

<p>The LLM generates the scaffold. We then iterate:</p>

<blockquote>
<p>"Add a resource that reads a markdown file from the desktop."<br>
"Add a prompt template for meeting summaries."</p>
</blockquote>

<p>However, LLMs make mistakes. For instance, they might hallucinate methods like <code>mcp.list_prompts()</code> which don't exist in the high-level <code>FastMCP</code> abstraction, or confuse <code>mcp.tool</code> with <code>mcp.resource</code>.</p>

<p><strong>The Senior Developer's Role</strong>: Your job shifts from syntax generation to verification. You must verify that the decorators match the intent and that the logic inside the functions handles exceptions (like dividing by zero). You use the Inspector to validate the LLM's output. If the implementation fails, you revert to a clean state (make backups of your <code>server.py</code>!) and refine the prompt.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>We have built a server that calculates, reads documentation, and structures textual analysis. We have moved from a static chat interface to a dynamic, integrated system.</p>

<p>The power of MCP lies in its extensibility. Today, it is a calculator; tomorrow, it is a server that queries your internal SQL database, fetches live stock data via HTTP APIs, or manages your Docker containers.</p>

<p>The ecosystem is shifting. We are no longer just prompting models; we are architecting the environments in which they operate. By mastering the Python SDK, creating robust tools, and debugging effectively with the Inspector, you position yourself at the forefront of this agentic shift.</p>

<p><strong>Go build something that does work, not just says work.</strong></p>

