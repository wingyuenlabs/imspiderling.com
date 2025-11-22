---
Title: How to connect the Next.js MCP server to VS Code Copilot Chat
Description: 
Author: Andrej Kirejeŭ
Date: 2025-11-22T21:35:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>The MCP server functionality is built in starting from Next.js 16 and doesn't require any additional configuration in the settings files. The path to the MCP server API is <code>/_next/mcp</code>.</p>

<p>Assuming your dev server is running on the port number 5002, do the following:</p>

<ol>
<li>In the <code>.vscode</code> folder in the root of your project, create a <code>mcp.json</code> file.</li>
<li>
<p>Put the following content inside it:<br>
</p>
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"servers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
  </span><span class="nl">"nextjs-mcp"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"http://localhost:5002/_next/mcp"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"http"</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">},</span><span class="w">
</span><span class="nl">"inputs"</span><span class="p">:</span><span class="w"> </span><span class="p">[]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</li>
<li><p>Save the file.</p></li>
<li><p>Run your project's dev server.</p></li>
<li><p>Open Copilot Chat.</p></li>
<li>
<p>Ask something specific to Next.js, for example:</p>

<blockquote>
<p>nextjs devtools, are there any build errors?</p>
</blockquote>
</li>
<li><p>If prompted, allow running the MCP server and executing actions. </p></li>
</ol>

<p>In our case, the answer was:</p>

<blockquote>
<p>No Next.js devtools build errors are currently reported. Let me know if you want me to keep monitoring or run a build/test explicitly.</p>
</blockquote>

<p>Now let's ask what the MCP server could do for us:</p>

<blockquote>
<p>nextjs devtools, tell me what you are able to do?</p>

<p>I’m connected to the Next.js devtools MCP server, so I can pull diagnostics like build/runtime errors, fetch the dev-server logs, list app/pages routes, and inspect server actions. If you need anything along those lines—checking status, seeing routes, digging into errors—just let me know which detail you want.</p>
</blockquote>

<p>Enjoy, and happy vibe coding!</p>

