---
Title: From OBO APIs to Agent Identities: Entra Conditional Access Still Works the Same
Description: 
Author: Anton Staykov
Date: 2026-03-16T22:09:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Six years ago I wrote <a href="https://github.com/Dayzure/AzureSQLDelegatedAuth" rel="noopener noreferrer">a small sample</a> to help me better understand how the On-Behalf-Of (OBO) flow actually works — a browser SPA calling a middle-tier Web API, which called Azure SQL <strong>on behalf of</strong> the signed-in user. Today it might be an assistive AI agent calling a MCP Server with the delegated constrains of the end-user. Same rules apply. This article explains why. </p>

<p>Three sentences that should survive beyond any particular technology:</p>

<blockquote>
<p>If your system acts on behalf of a user, delegation rules apply.<br>
If delegation rules apply, Conditional Access applies at token issuance for the downstream resource.<br>
APIs, agents, and MCP servers don't change that — they just change the shape of the middle tier.</p>
</blockquote>

<p>If an AI agent acts on behalf of a user, your existing Conditional Access policies that govern how users access corporate data <strong>already apply</strong> — automatically. You don't need to invent "agent-specific Conditional Access" for assistive agents. Assistive agents don't bypass Conditional Access. They inherit it.</p>

<p><strong>If you understand OBO and Conditional Access from classic Web APIs, you already understand how Entra Agent ID behaves for assistive AI agents.</strong> Delegation semantics didn't change. Conditional Access behavior didn't change. Only the shape of the middle tier changed.</p>

<h2>
  
  
  1. The Classic Delegated Access Pattern
</h2>

<p>The baseline architecture is the standard delegated access chain:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User
  ↓
SPA (interactive client)
  ↓  access token  [aud = Web API]
Web API (confidential client)
  ↓  OBO token     [aud = Resource]
Azure SQL  |  MCP Server
</code></pre>

</div>



<p>Two facts to hold firmly:</p>

<ul>
<li>The Web API <strong>never authenticates as itself</strong> to the downstream resource. It always acts <strong>on behalf of the signed-in user</strong>.</li>
<li>The downstream resource — Azure SQL, Microsoft Graph, or an MCP server — sees a delegated token. It evaluates the user's identity and the delegated scopes. Conditional Access policies have already been enforced by Entra during token issuance.</li>
</ul>

<h2>
  
  
  2. The Web API has a real identity — and delegation is explicitly granted
</h2>

<p>This is the detail most diagrams omit, and it matters in production.</p>

<p>The Web API is not just "code that runs somewhere". In Microsoft Entra it is a registered application with its own identity. That identity authenticates to the token endpoint as a <strong>confidential client</strong>, using a client secret or certificate.</p>

<p>In a real OBO implementation, the Web API does two things simultaneously:</p>

<ol>
<li>
<strong>Authenticates as itself</strong> — proving "I am the Web API" using its own credentials.</li>
<li>
<strong>Requests a delegated token</strong> for the downstream resource <em>on behalf of</em> the user, presenting the incoming user token as an assertion.</li>
</ol>

<p>Critically, the capability to do OBO is <strong>not implicit</strong>. The Web API's identity must be <strong>explicitly granted</strong> the delegated permissions (scopes) for the downstream resource. The tenant administrator grants this; the Web API cannot do it on its own authority.</p>

<blockquote>
<p><strong>Practical framing:</strong> the Web API authenticates as itself, but it can only mint delegated tokens because the tenant has explicitly authorized it to act on the user's behalf for the specific downstream resource.</p>
</blockquote>

<h2>
  
  
  3. OBO in 90 Seconds
</h2>

<p>The flow has three steps:</p>

<ol>
<li>The user signs in to the client and the client receives an access token whose audience is the Web API. The client uses that access token to call the Web API.</li>
<li>The Web API calls the token endpoint and exchanges that client token for a new one.</li>
<li>The resulting token is minted for the downstream resource and carries the user's identity from step 1 and delegated scopes from step 2.</li>
</ol>

<p>In protocol terms, the exchange looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight http"><code><span class="err">POST /&lt;tenant_id&gt;/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&amp;assertion=&lt;incoming_user_access_token&gt;
&amp;requested_token_use=on_behalf_of
&amp;scope=https://database.windows.net/.default
&amp;client_id=&lt;api_client_id&gt;
&amp;client_secret=&lt;api_client_secret&gt;
</span></code></pre>

</div>



<p>Note both elements in the same request: <strong>client authentication</strong> (<code>client_id</code> + <code>client_secret</code>) and the <strong>user assertion</strong> (<code>assertion</code>). Both are required.</p>

<blockquote>
<p><strong>The invariant:</strong> OBO is strictly delegated. It uses delegated scopes, not application roles. Application permissions are not involved.</p>
</blockquote>

<h2>
  
  
  4. Conditional Access: who is targeted vs. who "feels" it
</h2>

<p>This is where real misunderstanding happens — and the confusion compounds when agents enter the picture.</p>

<p><strong>Rule 1 — CA is assigned to the user and targets the resource.</strong><br>
A Conditional Access policy that requires MFA to access Azure SQL (or an MCP Server) is scoped to the user and the downstream resource. Evaluation happens when Microsoft Entra issues the delegated token for that resource — at the moment the OBO token is minted. Not when the user signs into the SPA. Not when the middle tier starts up.</p>

<p><strong>Rule 2 — The middle tier is not a CA subject.</strong><br>
Even though the Web API has its own registered identity, it is not the subject of a CA policy intended to govern how users access corporate data. The CA policy constrains the <strong>user's</strong> access, not the middle-tier service identity. This remains true regardless of what the middle tier is: Web API, assistive AI Agent, MCP Server, or anything else.</p>

<p><strong>Rule 3 — The middle tier still experiences CA outcomes.</strong><br>
When the Web API calls the token endpoint to acquire an OBO token, CA enforcement happens there. If CA requires step-up (like MFA), the token endpoint returns <code>interaction_required</code> with a <strong>claims challenge</strong>. The middle tier cannot satisfy this itself. It must surface this challenge back to the interactive client (the SPA in our case). The user performs the MFA, the client obtains a new token, and the middle tier retries the OBO exchange.</p>

<blockquote>
<p><strong>The one-liner:</strong> the middle tier is a CA messenger, not a CA subject.</p>
</blockquote>

<h2>
  
  
  5. Replacing the Web API with an (assistive) AI Agent (with an Agent Identity)
</h2>

<p>The transition to Entra Agent ID is explicit and mechanical:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Role</th>
<th>Classic World</th>
<th>Agentic World</th>
</tr>
</thead>
<tbody>
<tr>
<td>Middle Actor</td>
<td>Web API</td>
<td>AI Agent</td>
</tr>
<tr>
<td>Identity Type</td>
<td>app reg.</td>
<td>Agent Identity</td>
</tr>
<tr>
<td>Access Mode</td>
<td>Delegated (OBO)</td>
<td>Delegated (OBO)</td>
</tr>
<tr>
<td>CA Evaluation</td>
<td>On the resource</td>
<td>On the resource</td>
</tr>
</tbody>
</table></div>

<p>Nothing in the right column is new behavior. When using Entra Agent ID in delegated mode — the appropriate mode for an assistive agent acting on behalf of a signed-in user — three requirements apply:</p>

<ol>
<li>The Agent Identity must be <strong>granted the delegated permissions</strong> needed for the downstream resource.</li>
<li>The Agent Identity acquires a <strong>delegated token</strong> representing the user, carrying their identity and scopes.</li>
<li>The Agent Identity is <strong>constrained by Conditional Access</strong> policies that govern the user's access to that resource.</li>
</ol>

<blockquote>
<p><strong>The core continuity:</strong> you replaced the middle-tier actor, not the trust boundary.</p>
</blockquote>

<h2>
  
  
  6. MCP Server as the Downstream Resource
</h2>

<p>MCP servers can feel "agent-native," which leads engineers to assume the security model has changed around them. It hasn't.</p>

<p>An MCP Server is an <strong>OAuth-protected resource</strong> — a server that accepts access tokens. </p>

<p><strong>Do not pass tokens through.</strong> Equally valid for the AI agent acting on-behalf-of the user, but also for an MCP Server requesting authorizations to further resources on-behalf-of the user. </p>

<p>Beyond that, whether the downstream resource is Azure SQL, Microsoft Graph, or anything else, the CA and delegation rules are identical. From Microsoft Entra's perspective it is an OAuth-protected resource. Tokens are issued for it, delegated permissions are scoped to it, and Conditional Access policies target it.</p>

<h2>
  
  
  7. What to Actually Configure
</h2>

<ol>
<li>
<strong>Model your chain.</strong> Identify the downstream resource (REST API or MCP server) and the middle <em>actor</em> (Web API or agent identity in delegated mode). </li>
<li>
<strong>Make delegation explicit.</strong> Confirm the middle actor is granted the <em>delegated</em> permissions required for the downstream resource.</li>
<li>
<strong>Treat CA challenges as a product requirement.</strong> Expect <code>interaction_required</code> responses and claims challenges during downstream token acquisition. In your agent's backend, implement a handler for <code>MsalUiRequiredException</code> (for example). Design the user interaction path to satisfy step-up — do not swallow these errors.</li>
<li>
<strong>Never pass tokens through.</strong> Validate the incoming token for the agent audience, then acquire a fresh token for any downstream resource.</li>
</ol>

