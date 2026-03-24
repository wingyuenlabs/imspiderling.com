---
Title: How to Build an AI Employee Using MCP and Claude
Description: 
Author: 0n
Date: 2026-03-24T21:38:30.000Z
Robots: noindex,nofollow
Template: index
---
<p>What if your AI assistant could actually DO things — not just talk about them?</p>

<p>With MCP (Model Context Protocol) and 0nMCP, you can build an AI employee that:</p>

<ul>
<li>Responds to customer messages automatically</li>
<li>Scores leads and updates your CRM</li>
<li>Creates invoices and sends them</li>
<li>Posts to social media on schedule</li>
<li>Generates reports from your data</li>
</ul>

<p>Here's how.</p>

<h2>
  
  
  The Architecture
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>You (or a trigger) → Claude/AI → 0nMCP → APIs → Results
</code></pre>

</div>



<ol>
<li>A trigger fires (new message, schedule, webhook)</li>
<li>Claude receives the context</li>
<li>0nMCP executes the required API calls</li>
<li>Results flow back</li>
</ol>

<h2>
  
  
  Example 1: Auto-Responder
</h2>

<p>When a customer sends a message to your CRM:</p>

<ol>
<li>Webhook fires → hits your endpoint</li>
<li>Claude reads the message + contact history</li>
<li>Claude generates a personalized response</li>
<li>0nMCP sends the response via CRM API</li>
</ol>

<p>All automatic. Response time: under 10 seconds.</p>

<h2>
  
  
  Example 2: Lead Scorer
</h2>

<p>When a new contact is created:</p>

<ol>
<li>CRM webhook fires</li>
<li>Claude analyzes: email domain, title, company, source</li>
<li>Assigns a score 0-100</li>
<li>0nMCP updates the contact with the score and tags</li>
</ol>

<p>High-score leads get tagged "hot" and assigned to sales immediately.</p>

<h2>
  
  
  Example 3: Content Creator
</h2>

<p>On a daily schedule:</p>

<ol>
<li>Cron triggers the workflow</li>
<li>Claude generates a LinkedIn post based on your voice profile</li>
<li>0nMCP posts it via the social media API</li>
<li>Done — fresh content every day without lifting a finger</li>
</ol>

<h2>
  
  
  Setting It Up
</h2>

<h3>
  
  
  Step 1: Install 0nMCP
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install</span> <span class="nt">-g</span> 0nmcp
</code></pre>

</div>



<h3>
  
  
  Step 2: Connect Your Services
</h3>

<p>Add credentials for the services your AI employee needs:</p>

<ul>
<li>CRM for contacts and conversations</li>
<li>Stripe for invoicing</li>
<li>Slack for notifications</li>
<li>SendGrid for emails</li>
</ul>

<h3>
  
  
  Step 3: Create a Workflow
</h3>

<p>Save this as <code>auto-responder.0n</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"$0n"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"workflow"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Auto Responder"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"trigger"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"webhook"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"config"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"path"</span><span class="p">:</span><span class="w"> </span><span class="s2">"/hooks/auto-respond"</span><span class="w"> </span><span class="p">}</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"steps"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"step_001"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"service"</span><span class="p">:</span><span class="w"> </span><span class="s2">"anthropic"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"chat_completion"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"params"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"model"</span><span class="p">:</span><span class="w"> </span><span class="s2">"claude-sonnet-4-20250514"</span><span class="p">,</span><span class="w"> </span><span class="nl">"messages"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user"</span><span class="p">,</span><span class="w"> </span><span class="nl">"content"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Generate a reply to: {{inputs.message}}"</span><span class="p">}]</span><span class="w"> </span><span class="p">},</span><span class="w">
      </span><span class="nl">"output"</span><span class="p">:</span><span class="w"> </span><span class="s2">"reply"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"step_002"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"service"</span><span class="p">:</span><span class="w"> </span><span class="s2">"crm"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"action"</span><span class="p">:</span><span class="w"> </span><span class="s2">"send_message"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"params"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w"> </span><span class="nl">"contactId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"{{inputs.contactId}}"</span><span class="p">,</span><span class="w"> </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Email"</span><span class="p">,</span><span class="w"> </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="s2">"{{step_001.output.response}}"</span><span class="w"> </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Step 4: Run It
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>0nmcp run auto-responder.0n
</code></pre>

</div>



<p>Your AI employee is now live.</p>

<h2>
  
  
  The Cost
</h2>

<p>0nMCP is free and open source. The only cost is the AI API calls (Claude/OpenAI) — typically $0.01-0.05 per execution.</p>

<p>For a business running 100 automated actions per day, that's about $3/month.</p>

<h2>
  
  
  Get Started
</h2>

<ul>
<li>GitHub: <a href="https://github.com/0nork/0nMCP" rel="noopener noreferrer">github.com/0nork/0nMCP</a>
</li>
<li>npm: <a href="https://www.npmjs.com/package/0nmcp" rel="noopener noreferrer">npmjs.com/package/0nmcp</a>
</li>
<li>Website: <a href="https://0nmcp.com" rel="noopener noreferrer">0nmcp.com</a>
</li>
</ul>

<p><code>npm install -g 0nmcp</code> — your first AI employee starts today.</p>

