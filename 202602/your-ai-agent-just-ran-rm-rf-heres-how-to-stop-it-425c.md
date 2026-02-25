---
Title: Your AI Agent Just Ran rm -rf / — Here's How to Stop It
Description: 
Author: Etienne
Date: 2026-02-25T21:58:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>AI coding agents like Claude Code, Cursor, and Copilot can execute shell commands, edit files, and call APIs on your behalf. That's powerful — until the agent runs something dangerous.</p>

<p>I built <a href="https://github.com/AxonLabsDev/BodAIGuard" rel="noopener noreferrer">BodAIGuard</a> to solve this. It's a guardrail that sits between AI agents and your system, blocking dangerous actions before they execute.</p>

<h2>
  
  
  The Problem
</h2>

<p>AI agents can be tricked through <strong>prompt injection</strong>: malicious text hidden in tool results, emails, or web pages that hijacks the agent's behavior.</p>

<p>Without protection, the agent might actually execute dangerous commands.</p>

<h2>
  
  
  How BodAIGuard Works
</h2>

<p>BodAIGuard evaluates every action before it runs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>AI Agent → BodAIGuard → System
            ↑
    45 block rules
    45 confirm rules
    12 categories
    Prompt injection scanner
</code></pre>

</div>



<p><strong>5 enforcement modes:</strong></p>

<ul>
<li>
<strong>Claude Code hooks</strong> — intercepts tool calls via PreToolUse</li>
<li>
<strong>MCP Proxy</strong> — wraps any MCP server with security</li>
<li>
<strong>HTTP Proxy</strong> — sits between agent and API</li>
<li>
<strong>Shell Guard</strong> — bash integration via extdebug</li>
<li>
<strong>REST API</strong> — evaluate commands programmatically</li>
</ul>

<h2>
  
  
  Quick Start
</h2>

<p>Download the binary for your platform from <a href="https://github.com/AxonLabsDev/BodAIGuard/releases/latest" rel="noopener noreferrer">GitHub Releases</a>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Make it executable</span>
<span class="nb">chmod</span> +x bodaiguard-linux-x64
<span class="nb">sudo mv </span>bodaiguard-linux-x64 /usr/local/bin/bodaiguard

<span class="c"># Install Claude Code hooks</span>
bodaiguard <span class="nb">install </span>hooks

<span class="c"># Test it</span>
bodaiguard <span class="nb">test</span> <span class="s1">'rm -rf /'</span>
<span class="c"># → BLOCK: Filesystem root destruction</span>

bodaiguard <span class="nb">test</span> <span class="s1">'ls -la'</span>
<span class="c"># → ALLOW</span>
</code></pre>

</div>



<h2>
  
  
  MCP Proxy Mode (New in v0.8.0)
</h2>

<p>Wrap any MCP server with security:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"my-tool"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"bodaiguard"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"mcp-proxy"</span><span class="p">,</span><span class="w"> </span><span class="s2">"node"</span><span class="p">,</span><span class="w"> </span><span class="s2">"/path/to/mcp-server.js"</span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Every <code>tools/call</code> is evaluated against the rules. Dangerous calls get a JSON-RPC error instead of executing.</p>

<h2>
  
  
  Prompt Injection Detection
</h2>

<p>BodAIGuard scans content for injection attacks:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>bodaiguard scan <span class="s1">'Check encoded payloads'</span>
<span class="c"># → Detects base64-encoded attacks, hidden HTML, zero-width obfuscation</span>
</code></pre>

</div>



<p>Detection covers:</p>

<ul>
<li>Instruction overrides and role hijacking</li>
<li>ChatML delimiter injection</li>
<li>Base64-encoded attacks</li>
<li>Hidden HTML injection</li>
<li>Zero-width character obfuscation</li>
</ul>

<h2>
  
  
  Building Your Own Detector (Tutorial)
</h2>

<p>Want to understand the concepts? Here's a minimal 3-tier detector in Python:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">re</span><span class="p">,</span> <span class="n">base64</span><span class="p">,</span> <span class="n">unicodedata</span>

<span class="c1"># Tier 1: Regex patterns for known attack types
</span><span class="n">PATTERNS</span> <span class="o">=</span> <span class="p">[</span>
    <span class="p">(</span><span class="sa">r</span><span class="sh">'</span><span class="s">you\s+are\s+now\s+(DAN|evil|unrestricted)</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">role_hijack</span><span class="sh">'</span><span class="p">),</span>
    <span class="p">(</span><span class="sa">r</span><span class="sh">'</span><span class="s">&lt;\|im_start\|&gt;</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">chatml_delimiter</span><span class="sh">'</span><span class="p">),</span>
    <span class="p">(</span><span class="sa">r</span><span class="sh">'</span><span class="s">pretend\s+you\s+have\s+no\s+restrictions</span><span class="sh">'</span><span class="p">,</span> <span class="sh">'</span><span class="s">jailbreak</span><span class="sh">'</span><span class="p">),</span>
<span class="p">]</span>

<span class="k">def</span> <span class="nf">detect</span><span class="p">(</span><span class="n">content</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
    <span class="c1"># Normalize unicode
</span>    <span class="n">text</span> <span class="o">=</span> <span class="n">unicodedata</span><span class="p">.</span><span class="nf">normalize</span><span class="p">(</span><span class="sh">'</span><span class="s">NFKC</span><span class="sh">'</span><span class="p">,</span> <span class="n">content</span><span class="p">)</span>
    <span class="n">text</span> <span class="o">=</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">'</span><span class="s">[\u200b-\u200d\u2060\ufeff]</span><span class="sh">'</span><span class="p">,</span> <span class="sh">''</span><span class="p">,</span> <span class="n">text</span><span class="p">)</span>

    <span class="c1"># Tier 1: Fast regex
</span>    <span class="k">for</span> <span class="n">pattern</span><span class="p">,</span> <span class="n">category</span> <span class="ow">in</span> <span class="n">PATTERNS</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">re</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="n">pattern</span><span class="p">,</span> <span class="n">text</span><span class="p">,</span> <span class="n">re</span><span class="p">.</span><span class="n">IGNORECASE</span><span class="p">):</span>
            <span class="k">return</span> <span class="p">{</span><span class="sh">'</span><span class="s">blocked</span><span class="sh">'</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span> <span class="sh">'</span><span class="s">reason</span><span class="sh">'</span><span class="p">:</span> <span class="n">category</span><span class="p">}</span>

    <span class="c1"># Tier 2: Base64 decode + re-scan
</span>    <span class="k">for</span> <span class="n">match</span> <span class="ow">in</span> <span class="n">re</span><span class="p">.</span><span class="nf">findall</span><span class="p">(</span><span class="sa">r</span><span class="sh">'</span><span class="s">[A-Za-z0-9+/]{24,}={0,2}</span><span class="sh">'</span><span class="p">,</span> <span class="n">text</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">decoded</span> <span class="o">=</span> <span class="n">base64</span><span class="p">.</span><span class="nf">b64decode</span><span class="p">(</span><span class="n">match</span><span class="p">,</span> <span class="n">validate</span><span class="o">=</span><span class="bp">True</span><span class="p">).</span><span class="nf">decode</span><span class="p">()</span>
            <span class="k">for</span> <span class="n">pattern</span><span class="p">,</span> <span class="n">category</span> <span class="ow">in</span> <span class="n">PATTERNS</span><span class="p">:</span>
                <span class="k">if</span> <span class="n">re</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="n">pattern</span><span class="p">,</span> <span class="n">decoded</span><span class="p">,</span> <span class="n">re</span><span class="p">.</span><span class="n">IGNORECASE</span><span class="p">):</span>
                    <span class="k">return</span> <span class="p">{</span><span class="sh">'</span><span class="s">blocked</span><span class="sh">'</span><span class="p">:</span> <span class="bp">True</span><span class="p">,</span> <span class="sh">'</span><span class="s">reason</span><span class="sh">'</span><span class="p">:</span> <span class="sa">f</span><span class="sh">'</span><span class="s">encoded_</span><span class="si">{</span><span class="n">category</span><span class="si">}</span><span class="sh">'</span><span class="p">}</span>
        <span class="k">except</span> <span class="nb">Exception</span><span class="p">:</span>
            <span class="k">pass</span>

    <span class="k">return</span> <span class="p">{</span><span class="sh">'</span><span class="s">blocked</span><span class="sh">'</span><span class="p">:</span> <span class="bp">False</span><span class="p">}</span>
</code></pre>

</div>



<p>This catches the basics. BodAIGuard goes much further with DOM parsing, Unicode lookalike detection, and YAML-driven rules.</p>

<h2>
  
  
  YAML-Driven Rules
</h2>

<p>All rules live in <code>default.yaml</code> — no code changes needed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">actions</span><span class="pi">:</span>
  <span class="na">destructive</span><span class="pi">:</span>
    <span class="na">block</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">pattern</span><span class="pi">:</span> <span class="s1">'</span><span class="s">rm\\s+(-[a-zA-Z]*)?\\s*/'</span>
        <span class="na">reason</span><span class="pi">:</span> <span class="s">Filesystem root destruction</span>
      <span class="pi">-</span> <span class="na">pattern</span><span class="pi">:</span> <span class="s1">'</span><span class="s">mkfs\\.'</span>
        <span class="na">reason</span><span class="pi">:</span> <span class="s">Filesystem format</span>
    <span class="na">confirm</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">pattern</span><span class="pi">:</span> <span class="s1">'</span><span class="s">rm\\s+-r'</span>
        <span class="na">reason</span><span class="pi">:</span> <span class="s">Recursive delete</span>

<span class="na">paths</span><span class="pi">:</span>
  <span class="na">block</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">~/.ssh/**</span>
    <span class="pi">-</span> <span class="s">/etc/shadow</span>
  <span class="na">readonly</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s">/etc/passwd</span>
</code></pre>

</div>



<h2>
  
  
  Links
</h2>

<ul>
<li>
<strong>GitHub</strong>: <a href="https://github.com/AxonLabsDev/BodAIGuard" rel="noopener noreferrer">AxonLabsDev/BodAIGuard</a>
</li>
<li>
<strong>Download</strong>: <a href="https://github.com/AxonLabsDev/BodAIGuard/releases/latest" rel="noopener noreferrer">Latest Release</a>
</li>
<li>
<strong>License</strong>: ALD Proprietary (free to use, source available on request)</li>
</ul>




<p><em>Built by a self-taught system orchestrator who got tired of AI agents doing dangerous things on his server.</em></p>

