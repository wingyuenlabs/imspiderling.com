---
Title: Use LLM for EDA licenses analysis
Description: 
Author: Misha Zaslavskiy
Date: 2025-08-28T21:52:04.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Why?
</h2>

<p>If your company has an EDA license monitoring system, you have a lot of useful data available for analysis. And if you bought EDA tools but don't actively monitor license usage, you may want to check <a href="//dev.to/misha_zaslavskiy/licenses-monitoring-dashboard-installation-32kf">my earlier post</a>.</p>

<p>Now what do we do with this "big-data"? There most likely will be time gaps caused by infrastructure failures/maintenance, as well as evolution of suppliers and list of licenses over time, and this complicates manual analysis.</p>

<p>With the advancement of AI/ML tools, you could now connect your Prometheus database with AI assist tool via what's called MCP: Model Context Protocol. This allows LLM to query your database, automatically load relevant and up-to-date information in the context window, and as a result LLM will be able to accurately answer questions about usage, trends or whatever else you are interested in.</p>

<h2>
  
  
  How
</h2>

<p>Here is an example how to connect Prometheus DB to <a href="https://cursor.com/en" rel="noopener noreferrer">Cursor AI code editor</a>.</p>

<p>Go to <code>Cursor Settings</code> -&gt; <code>Tools &amp; Integration</code> -&gt; <code>New MCP Server</code> and add this code to the <code>.cursor/mcp.json</code> file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"prometheus"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"docker"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
        </span><span class="s2">"run"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"--network"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"cad"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"--name"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"prometheus-mcp-server"</span><span class="p">,</span><span class="w">  
        </span><span class="s2">"-p"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"8000:8000"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"-i"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"--rm"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"-e"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"PROMETHEUS_URL"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"ghcr.io/pab1it0/prometheus-mcp-server:latest"</span><span class="w">
      </span><span class="p">],</span><span class="w">
      </span><span class="nl">"env"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"PROMETHEUS_URL"</span><span class="p">:</span><span class="w"> </span><span class="s2">"http://prometheus:9090"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Note that this example uses Docker deployment of MCP server. You don't need to run this container manually, Cursor will do it automatically. For that option you'll obviously need access to Docker, and Prometheus DB should run on the same <code>cad</code> docker network. If your setup is different, you'll need to modify this JSON file accordingly, see the source reference below.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fylsta1c6vialtr6t2mg7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fylsta1c6vialtr6t2mg7.png" alt=" " width="800" height="500"></a></p>

<p>If your MCP tool show green status and tools that are available, you are good to go. Otherwise check the Docker container status and logs. </p>

<p>Here is the Cursor chat prompt example:</p>

<blockquote>
<p>query available licenses through MCP</p>
</blockquote>

<p>And the result looks quite useful:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0sbu8q92mayw7cledg94.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0sbu8q92mayw7cledg94.png" alt=" " width="800" height="1093"></a></p>

<h2>
  
  
  Sources
</h2>

<p><a href="https://playbooks.com/mcp/pab1it0-prometheus" rel="noopener noreferrer">https://playbooks.com/mcp/pab1it0-prometheus</a></p>

