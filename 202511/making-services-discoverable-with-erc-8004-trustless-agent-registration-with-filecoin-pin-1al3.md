---
Title: Making Services Discoverable with ERC-8004: Trustless Agent Registration with Filecoin Pin
Description: 
Author: Matt Hamilton
Date: 2025-11-04T21:51:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>Making Services Discoverable with ERC-8004: Trustless Agent Registration with Filecoin Pin</p>

<p>I've been thinking a lot about agent metadata lately. Not the sexy stuff, not the AI models or the clever algorithms. The boring bit. Where do you put the JSON file that describes what an agent does?</p>

<p>It's one of those problems that seems trivial until you actually try to solve it properly. You can't store it on-chain (too expensive). You can't use regular IPFS pinning (it might disappear). You can't use AWS (defeats the whole point of decentralisation). So what do you do?</p>

<p>This week, I finally got a chance to properly explore this by registering GitHub's MCP server as an <a href="https://eips.ethereum.org/EIPS/eip-8004" rel="noopener noreferrer">ERC-8004</a> agent with <a href="https://docs.filecoin.io/builder-cookbook/filecoin-pin" rel="noopener noreferrer">Filecoin Pin</a>. Here's what I learnt.</p>

<h2>
  
  
  The Problem
</h2>

<p>The issue is simple enough to explain: when you register an agent, you need to point to some metadata, a JSON file describing what it does, how to connect to it, what it's capable of. The <code>tokenURI</code> in ERC-8004 parlance.</p>

<p>You can't store this on-chain. A typical agent card is 1-2KB of JSON. On Ethereum mainnet, that's prohibitively expensive. Even on L2s, you're looking at a chunk of change for every agent registration.</p>

<p>So you store it off-chain and reference it. Which raises the obvious question: where?</p>

<p>Generic IPFS pinning services? They can drop your content. No guarantees. Centralised storage like AWS? Single point of failure, ongoing costs, and frankly defeats the point. Store everything on-chain anyway? Not practical.</p>

<p>I needed something that would actually work long-term.</p>

<h2>
  
  
  Filecoin Pin
</h2>

<p>I'd been aware of Filecoin Pin for a while but hadn't actually tried it. The pitch is compelling: you get cryptographic proof that your data is being stored. Daily PDP (Proof of Data Possession) checks. Decentralised across Filecoin's storage network. IPFS compatible so it works with existing tooling.</p>

<p>The key bit is that last part about proof. With generic IPFS pinning, you're trusting that someone, somewhere, is keeping your content around. With Filecoin Pin, you can actually verify it. Daily proofs that storage providers are holding your data.</p>

<p>That matters for agent metadata. If you're building something meant to last years, you need more than hope.</p>

<h2>
  
  
  What I Actually Built
</h2>

<p>For this experiment, I took GitHub's existing MCP server and registered it as an ERC-8004 agent. Not building something new, just making something that already exists discoverable through the registry.</p>

<p>GitHub already runs a public MCP server at <code>https://api.githubcopilot.com/mcp/</code>. It provides repository management, issue tracking, PR tools. The usual GitHub stuff exposed through the Model Context Protocol. It's real, it's running, and anyone can use it.</p>

<p>What it didn't have was an ERC-8004 identity. No way for other agents to discover it. No on-chain registration. No verifiable metadata storage.</p>

<p>So I created an agent card (a JSON file describing the service), stored it on Filecoin Pin, and registered it on the Identity Registry on Base Sepolia. Now it has a verifiable, discoverable identity.</p>

<h3>
  
  
  The Agent Card
</h3>

<p>The agent card is a JSON file that describes the MCP server's capabilities and how to connect to it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://eips.ethereum.org/EIPS/eip-8004#registration-v1"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"GitHub Integration Agent"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AI agent providing GitHub repository, issue, and pull request management capabilities..."</span><span class="p">,</span><span class="w">
  </span><span class="nl">"endpoints"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MCP"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"endpoint"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://api.githubcopilot.com/mcp/"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"version"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.0.0"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"capabilities"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"tools"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"repository_management"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Browse code, search files, analyze commits..."</span><span class="w">
          </span><span class="p">},</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"issue_management"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Create, update, search, and manage GitHub issues..."</span><span class="w">
          </span><span class="p">},</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"pull_request_management"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Review PRs, manage approvals, merge conflicts..."</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"supportedTrust"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"reputation"</span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This agent card gets uploaded to Filecoin Pin, which returns a CID (Content Identifier). That CID then gets registered on-chain as <code>ipfs://&lt;CID&gt;/github-agent-card.json</code>, effectively creating a verifiable, discoverable identity for GitHub's existing MCP server.</p>

<h2>
  
  
  The Complete Workflow
</h2>

<p>Here's how the pieces fit together:</p>

<h3>
  
  
  1. Upload to Filecoin Pin
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>filecoin-pin add <span class="nt">--auto-fund</span> github-agent-card.json
</code></pre>

</div>



<p>This uploads your agent card and returns:</p>

<ul>
<li>
<strong>Root CID</strong> - The IPFS identifier</li>
<li>
<strong>Dataset ID</strong> - For checking PDP proof status</li>
<li>
<strong>Storage deal confirmation</strong> - Proof it's being stored</li>
</ul>

<p>The <code>--auto-fund</code> flag ensures your storage provider wallet has sufficient USDFC (Filecoin stablecoin) to pay for storage.</p>

<h3>
  
  
  2. Register on Base Sepolia
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>cast send 0x7177a6867296406881E20d6647232314736Dd09A <span class="se">\</span>
  <span class="s2">"register(string)"</span> <span class="se">\</span>
  <span class="s2">"ipfs://&lt;CID&gt;/github-agent-card.json"</span> <span class="se">\</span>
  <span class="nt">--rpc-url</span> https://sepolia.base.org <span class="se">\</span>
  <span class="nt">--private-key</span> <span class="nv">$PRIVATE_KEY</span>
</code></pre>

</div>



<p>This mints an ERC-721 NFT representing your agent on the ERC-8004 Identity Registry.</p>

<p><strong>Why Base Sepolia?</strong></p>

<ul>
<li>Low gas costs (it's an L2)</li>
<li>Official ERC-8004 reference implementation is deployed there</li>
<li>Easy to get testnet ETH from faucets</li>
</ul>

<h3>
  
  
  3. Verify PDP Proofs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>filecoin-pin data-set &lt;dataset-id&gt;
</code></pre>

</div>



<p>This shows you the proof status:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Dataset ID: 933
Root CID: bafybeihhal5hlbylkibniig6j72wdrm7lr4nf6z47natleh2jkyosrg7di
Storage Provider: f01234
Status: Active
Last PDP Proof: 2025-01-15 14:32:10 UTC
Next Proof: 2025-01-16 14:32:10 UTC
</code></pre>

</div>



<p><strong>Daily proofs</strong> mean you can always verify your agent metadata is still being stored. This is the cryptographic guarantee that sets Filecoin Pin apart.</p>

<h3>
  
  
  4. Agent Discovery
</h3>

<p>Now any application can discover and use your agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Query the registry</span>
cast call 0x7177a6867296406881E20d6647232314736Dd09A <span class="se">\</span>
  <span class="s2">"tokenURI(uint256)"</span> 55 <span class="se">\</span>
  <span class="nt">--rpc-url</span> https://sepolia.base.org

<span class="c"># Returns: ipfs://bafybeihhal5hlbylkibniig6j72wdrm7lr4nf6z47natleh2jkyosrg7di/github-agent-card.json</span>

<span class="c"># Fetch the agent card</span>
curl <span class="nt">-s</span> <span class="s2">"https://ipfs.io/ipfs/bafybeihhal5hlbylkibniig6j72wdrm7lr4nf6z47natleh2jkyosrg7di/github-agent-card.json"</span> | jq <span class="nb">.</span>
</code></pre>

</div>



<p>The agent card tells them how to connect to the GitHub MCP server and what capabilities are available.</p>

<h2>
  
  
  Why This Matters for Builders
</h2>

<p>This is still very much a work in progress. Everything here is running on testnets, and we're actively figuring out what the production infrastructure should look like. But having said that, the pattern is already useful for making any existing service discoverable as an ERC-8004 agent with verifiable, persistent metadata.</p>

<h3>
  
  
  For Agent Builders
</h3>

<p>You get:</p>

<ul>
<li>
<strong>Peace of mind</strong> - Your agent metadata won't disappear</li>
<li>
<strong>Verifiability</strong> - Anyone can check that storage is active</li>
<li>
<strong>Decentralisation</strong> - No single point of failure</li>
<li>
<strong>Standards compliance</strong> - Works with ERC-8004 ecosystem</li>
</ul>

<h3>
  
  
  For Agent Users
</h3>

<p>They get:</p>

<ul>
<li>
<strong>Trust</strong> - Can verify agents are legitimate and persistent</li>
<li>
<strong>Discovery</strong> - Find agents via on-chain registry</li>
<li>
<strong>Composability</strong> - Combine multiple agents together</li>
<li>
<strong>Transparency</strong> - See full agent capabilities before using</li>
</ul>

<h3>
  
  
  For the Ecosystem
</h3>

<p>We get:</p>

<ul>
<li>
<strong>Infrastructure</strong> for the emerging agent economy</li>
<li>
<strong>Interoperability</strong> via ERC-8004 standard</li>
<li>
<strong>Long-term viability</strong> with provable storage</li>
<li>
<strong>Foundation</strong> for reputation and validation systems</li>
</ul>

<p>Whilst these lists look a bit like marketing bullet points, each of these actually matters. The infrastructure piece is what I find most interesting, because we're finally building the plumbing that makes agent composition practical.</p>

<p>We'd love to have more developers try this out and give us feedback on what works and what doesn't.</p>

<h2>
  
  
  Try It Yourself
</h2>

<p>So if you want to give this a go, I've put together a complete tutorial that walks you through every step:</p>

<p><strong><a href="//./TUTORIAL.md">Register an ERC-8004 Agent with Filecoin Pin Storage</a></strong></p>

<p>The tutorial includes:</p>

<ul>
<li>Complete prerequisites and token setup</li>
<li>Step-by-step commands with expected outputs</li>
<li>Screenshot indicators showing what you should see</li>
<li>Troubleshooting for common issues</li>
<li>Example agent card you can customise</li>
</ul>

<p>Everything you need is in the tutorial. It's designed to take you from zero to a registered agent in about 30-45 minutes. Having said that, if it's your first time working with Filecoin or Base Sepolia, you might want to give yourself a bit more time to get familiar with the faucets and tooling.</p>

<h2>
  
  
  What's Next?
</h2>

<p>Well, this is just the beginning. The ERC-8004 standard includes three registries:</p>

<ol>
<li>
<strong>Identity Registry</strong> (what I demonstrated here) - Register and discover agents</li>
<li>
<strong>Reputation Registry</strong> (coming) - Build trust through verified actions</li>
<li>
<strong>Validation Registry</strong> (coming) - Third-party verification of agent behaviour</li>
</ol>

<p>Combining these with provable storage creates the foundation for a real agent economy. I think the reputation piece is going to be particularly interesting, because that's where agents can start to build track records that others can actually verify.</p>

<h3>
  
  
  Ideas to Explore
</h3>

<p>Some things I'm thinking about:</p>

<ul>
<li>
<strong>Multi-agent systems</strong> - Agents that discover and compose with other agents</li>
<li>
<strong>Reputation building</strong> - Agents that accumulate verified track records</li>
<li>
<strong>Validator networks</strong> - Decentralised verification of agent behaviour</li>
<li>
<strong>Agent marketplaces</strong> - Discover and use agents based on capabilities and reputation</li>
<li>
<strong>Cross-chain agents</strong> - Agents that operate across multiple networks</li>
</ul>

<p>There's probably a lot more that I haven't thought of yet. If any of this sounds like something you're working on, I'd love to hear about it.</p>

<h2>
  
  
  The Bigger Picture
</h2>

<p>I think we're at an inflection point. AI agents are becoming capable enough to act autonomously, but they need decentralised infrastructure to be truly trustless.</p>

<p>By solving the storage problem with Filecoin Pin and the identity problem with ERC-8004, we're enabling a new category of applications:</p>

<ul>
<li>
<strong>Autonomous trading agents</strong> with verifiable track records</li>
<li>
<strong>Code review agents</strong> that build reputation over time</li>
<li>
<strong>Data analysis agents</strong> that can be audited and verified</li>
<li>
<strong>Coordination agents</strong> that compose multiple specialised agents</li>
<li>
<strong>Personal assistants</strong> that users actually own and control</li>
</ul>

<p>The key insight? <strong>Agents aren't just smart contracts. They're long-lived entities that need persistent, verifiable infrastructure.</strong> It has been a challenge working out what that infrastructure looks like, but I think the combination of ERC-8004 and Filecoin Pin gets us pretty close.</p>

<h2>
  
  
  Getting Involved
</h2>

<p>The ERC-8004 ecosystem is just getting started, and we need builders to shape it.</p>

<p><strong>If you want to try this out:</strong></p>

<ol>
<li>Follow the <a href="//./TUTORIAL.md">tutorial</a> to register your first agent</li>
<li>Join the <a href="https://github.com/ethereum/EIPs/issues/8004" rel="noopener noreferrer">ERC-8004 discussions</a>
</li>
<li>Check out the <a href="https://github.com/ChaosChain/trustless-agents-erc-ri" rel="noopener noreferrer">reference implementation</a>
</li>
<li>Explore <a href="https://docs.filecoin.io/builder-cookbook/filecoin-pin" rel="noopener noreferrer">Filecoin Pin documentation</a>
</li>
</ol>

<p><strong>Some things you could build:</strong></p>

<ul>
<li>Register your existing services as ERC-8004 agents</li>
<li>Build agent discovery tools</li>
<li>Develop reputation systems</li>
<li>Create validator networks</li>
</ul>

<p>The infrastructure is ready. The standard is here. So if any of this sounds like it would be useful for what you're building, then get in touch and let's chat!</p>

<h2>
  
  
  Resources
</h2>

<ul>
<li>
<strong>Tutorial</strong>: <a href="https://docs.filecoin.io/builder-cookbook/filecoin-pin/" rel="noopener noreferrer">Register an ERC-8004 Agent with Filecoin Pin</a>
</li>
<li>
<strong>ERC-8004 Specification</strong>: <a href="https://eips.ethereum.org/EIPS/eip-8004" rel="noopener noreferrer">https://eips.ethereum.org/EIPS/eip-8004</a>
</li>
<li>
<strong>Filecoin Pin CLI</strong>: <a href="https://docs.filecoin.io/builder-cookbook/filecoin-pin/filecoin-pin-cli" rel="noopener noreferrer">https://docs.filecoin.io/builder-cookbook/filecoin-pin/filecoin-pin-cli</a>
</li>
<li>
<strong>Base Sepolia Faucet</strong>: <a href="https://www.alchemy.com/faucets/base-sepolia" rel="noopener noreferrer">https://www.alchemy.com/faucets/base-sepolia</a>
</li>
<li>
<strong>Example Agent</strong>: <a href="https://sepolia.basescan.org/token/0x7177a6867296406881E20d6647232314736Dd09A?a=55" rel="noopener noreferrer">View on Basescan</a>
</li>
</ul>

