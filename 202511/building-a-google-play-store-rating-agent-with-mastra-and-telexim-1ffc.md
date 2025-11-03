---
Title: Building a Google Play Store Rating Agent with Mastra and Telex.im
Description: 
Author: Wani
Date: 2025-11-03T21:36:57.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>As part of an assignment to integrate AI agents with Telex.im, an AI-augmented chat platform that also serves as a Slack alternative for education, communities, and bootcamps. I set out to build something simple yet useful: an agent that fetches Google Play Store app ratings on demand. The requirements were straightforward - create an agent that could look up data from an API. What started as a simple idea turned into a fascinating journey through the Mastra framework and the A2A protocol.</p>

<p>In this post, I'll walk you through the entire process - from initial setup to deployment and integration with Telex.im, including the challenges I faced and how I solved them.</p>

<p><a href="https://telex.im/ai-coworkers/playstore-rating-fetcher-agent-40fda3935c73" rel="noopener noreferrer">🔗 Try the live agent: PlayStore Rating Agent on Telex.im</a> - Ask it about any Android app and get instant ratings, install counts, reviews, and developer information from the Google Play Store.</p>

<p><strong>The Vision</strong></p>

<ul>
<li>Before diving into code, I needed to define what success looked like:</li>
<li>On-Demand Lookup: Users should be able to ask "What's the rating for Instagram?" and get instant, detailed results</li>
<li>Rich Information: Beyond just ratings, provide installs, reviews, developer info, and more</li>
<li>Seamless Integration: Work perfectly with Telex.im using the A2A protocol</li>
</ul>

<p><strong>Getting Started with Mastra</strong></p>

<p><strong>Initial Setup</strong></p>

<p>The Mastra CLI made getting started incredibly smooth. I ran:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm create mastra@latest -y
</code></pre>

</div>



<p>The wizard walked me through setup, asking for:</p>

<ul>
<li>Project name: playstore-rating-agent</li>
<li>Model provider: I chose Google Gemini 2.0 Flash (no credit card required!) and added my API key</li>
<li>Example setup: Started with the weather agent template</li>
</ul>

<p>Within minutes, I had a working project structure:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>src/
├── mastra/
│   ├── agents/
│   │   └── weather-agent.ts
│   ├── tools/
│   │   └── weather-tool.ts
│   ├── workflows/
│   │   └── weather-workflow.ts
│   ├── scorers/
│   │   └── weather-scorer.ts
│   └── index.ts
├── .env.example
├── package.json
└── tsconfig.json
</code></pre>

</div>



<p>What Worked: The CLI's project scaffolding gave me a perfect template to adapt. Having real, working examples made understanding the framework much easier.</p>

<p><strong>Mastra Features I Used</strong></p>

<p>Throughout this project, I leveraged several key Mastra features that made development smooth and efficient:</p>

<p><strong>1. Mastra Agents</strong></p>

<p>The core feature I used was Mastra's Agent system. Agents in Mastra are AI-powered entities that can understand user queries, make decisions, and execute tools. My playStoreAgent was configured with:</p>

<ul>
<li>Custom instructions defining its behavior and personality</li>
<li>Integration with Google Gemini 2.0 Flash model</li>
<li>Access to custom tools for fetching Play Store data</li>
<li>Built-in memory for maintaining conversation context</li>
</ul>

<p><strong>2. Mastra Tools</strong></p>

<p>I created a custom tool using Mastra's createTool function. Tools are the actions your agent can perform. Key features I used:</p>

<ul>
<li>Zod Schema Validation: Both input and output schemas with type safety</li>
<li>Async Execution: Native support for async operations like API calls</li>
<li>Error Handling: Built-in error propagation and handling</li>
<li>Type Inference: Automatic TypeScript types from Zod schemas</li>
</ul>

<p><strong>3. Mastra Memory</strong></p>

<p>The Memory feature allowed my agent to remember previous conversations using LibSQLStore:</p>

<ul>
<li>Persistent conversation history across sessions</li>
<li>Automatic context management</li>
<li>Easy retrieval of past interactions</li>
<li>Built-in storage adapters (I used LibSQLStore with SQLite)</li>
</ul>

<p><strong>4. Mastra Server &amp; API Routes</strong></p>

<ul>
<li>Mastra's server capabilities made integration straightforward:</li>
<li>Built-in OpenAPI documentation generation</li>
<li>Swagger UI for testing endpoints</li>
<li>Custom route registration with registerApiRoute</li>
<li>JSON-RPC 2.0 support for A2A protocol compliance</li>
</ul>

<p><strong>5. Mastra Logger</strong></p>

<ul>
<li>The PinoLogger integration provided excellent observability:</li>
<li>Structured logging with configurable levels</li>
<li>Request/response tracking</li>
<li>Error logging with stack traces</li>
<li>Performance monitoring</li>
</ul>

<p><strong>6. Mastra Bundler Configuration</strong></p>

<p>I used the bundler settings to handle external dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>bundler: {
  externals: ["google-play-scraper"],
}
</code></pre>

</div>



<p>This ensured the <code>google-play-scraper</code> package was properly handled during deployment.</p>

<p><strong>Building the Core Components</strong></p>

<p><strong>1. The Play Store Scraping Tool</strong></p>

<p>My first task was creating a tool that could actually fetch app data. I installed <code>google-play-scraper</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>pnpm add google-play-scraper
</code></pre>

</div>



<p>Then adapted the weather tool template to create <code>playstore-tool.ts</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export const playStoreRatingTool = createTool({
  id: 'get-playstore-rating',
  description: 'Get current ratings and information for an app from Google Play Store',
  inputSchema: z.object({
    appName: z.string().describe('Name of the app to search for'),
  }),
  outputSchema: z.object({
    appId: z.string(),
    title: z.string(),
    rating: z.number(),
    ratingsCount: z.number(),
    reviews: z.number(),
    installs: z.string(),
    price: z.union([z.string(), z.number()]),
    developer: z.string(),
    lastUpdated: z.string(),
    version: z.string(),
    url: z.string(),
  }),
  execute: async ({ context }) =&gt; {
    // Search and fetch app details
    const searchResults = await gplay.search({
      term: context.appName,
      num: 1,
    });

    const appDetails = await gplay.app({ 
      appId: searchResults[0].appId 
    });

    return {
      // ... formatted app data
    };
  },
});
</code></pre>

</div>



<p><strong>2. Creating the Agent</strong></p>

<p>Next, I created <code>playstore-agent.ts</code> by adapting the weather agent:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export const playStoreAgent = new Agent({
  name: 'PlayStore Rating Agent',
  instructions: `
    You are a helpful assistant that provides Google Play Store app ratings and information.

    Your primary function is to help users get app ratings and details from the Google Play Store.
    // ... detailed instructions
  `,
  model: 'google/gemini-2.0-flash',
  tools: { playStoreRatingTool },
  memory: new Memory({
    storage: new LibSQLStore({
      url: 'file:../mastra.db',
    }),
  }),
});
</code></pre>

</div>



<p>What Worked: Mastra's Memory system worked out of the box. The agent could remember previous conversations seamlessly.</p>

<p>What Didn't (The Big Issue): My agent was returning raw JSON to users! Instead of a nice formatted message, users saw:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{"type":"tool-result","runId":"d880b4a8-a8d5-49db-bee9-316e70f38e7f"...}
</code></pre>

</div>



<p>This was messy and unprofessional. I realized the agent instructions needed to be much more explicit about formatting.</p>

<p><strong>3. Fixing the Formatting Issue</strong></p>

<p>I updated the agent instructions with very specific formatting guidance:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>instructions: `
  // ... previous instructions

  - IMPORTANT: Format the response in a clean, readable way. 
    DO NOT include raw JSON or tool output.
  - Present the information like this:

    [App Name] has a rating of [X.X]/5.0 with [number] total ratings. 
    It has been installed [installs] times and was last updated on [date]. 
    The current version is [version] and it is offered for [price] by [developer].

  - Keep responses natural and conversational
  - Never show the raw tool results or JSON data to the user
`,
</code></pre>

</div>



<p>What Worked: Being explicit in the instructions solved the problem completely. The agent now formats responses beautifully:</p>

<blockquote>
<p>"WhatsApp Messenger has a rating of 4.39/5.0 with 212,153,070 total ratings. It has been installed 10,000,000,000+ times and was last updated on 2025-10-29. The current version varies and it is offered for free by WhatsApp LLC."</p>
</blockquote>

<p>Lesson Learned: AI agents need very explicit instructions. Don't assume they'll figure out formatting on their own.</p>

<h3>
  
  
  The A2A Protocol Integration
</h3>

<p>This was the most crucial part - making the agent work with Telex.im, a Slack alternative for communities that also functions as an AI agent platform like Make.</p>

<p><strong>Understanding A2A</strong></p>

<p>The A2A (Agent-to-Agent) protocol is based on JSON-RPC 2.0. It standardizes how agents communicate, ensuring:</p>

<ul>
<li>Consistent message formats</li>
<li>Proper context management</li>
<li>Structured artifacts and results</li>
<li>Standard error handling</li>
<li>Creating the A2A Route Handler</li>
</ul>

<p>I created a2a-agent-route.ts to bridge Mastra with the A2A protocol, using Mastra's registerApiRoute feature:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export const a2aAgentRoute = registerApiRoute('/a2a/agent/:agentId', {
  method: 'POST',
  handler: async (c) =&gt; {
    const body = await c.req.json();
    const { jsonrpc, id: requestId, method, params } = body;

    // Validate JSON-RPC 2.0 format
    if (jsonrpc !== '2.0' || !requestId) {
      return c.json({
        jsonrpc: '2.0',
        id: requestId || null,
        error: {
          code: -32600,
          message: 'Invalid Request: jsonrpc must be "2.0" and id is required'
        }
      }, 400);
    }

    // Convert A2A messages to Mastra format
    const mastraMessages = messagesList.map((msg) =&gt; ({
      role: msg.role,
      content: msg.parts?.map((part) =&gt; {
        if (part.kind === 'text') return part.text;
        if (part.kind === 'data') return JSON.stringify(part.data);
        return '';
      }).join('\n') || ''
    }));

    // Execute agent and return A2A-compliant response
    const response = await agent.generate(mastraMessages);

    return c.json({
      jsonrpc: '2.0',
      id: requestId,
      result: {
        id: taskId || randomUUID(),
        contextId: contextId || randomUUID(),
        status: { state: 'completed', /* ... */ },
        artifacts: [/* ... */],
        history: [/* ... */],
        kind: 'task'
      }
    });
  }
});
</code></pre>

</div>



<p>What Worked: Following the JSON-RPC 2.0 spec strictly ensured compatibility. The validation caught malformed requests early.</p>

<p>What Didn't: Initially, I forgot to handle the artifacts and history arrays properly. These are crucial for Telex to display results correctly.</p>

<p><strong>Registering Everything with Mastra</strong></p>

<p>In index.ts, I wired everything together using Mastra's central configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import { Mastra } from '@mastra/core/mastra';
import { PinoLogger } from '@mastra/loggers';
import { LibSQLStore } from '@mastra/libsql';
import { playStoreAgent } from './agents/playstore-agent';
import { scheduledRatingCheckWorkflow } from './workflows/playstore-workflow';
import { playStoreScorer } from './scorers/playstore-scorer';
import { a2aAgentRoute } from './routes/a2a-agent-route';

export const mastra = new Mastra({
  agents: { playStoreAgent },
  workflows: { scheduledRatingCheckWorkflow },
  scorers: { playStoreScorer },
  storage: new LibSQLStore({ url: 'file:./mastra.db' }),
  logger: new PinoLogger({
    name: 'PlayStoreAgent',
    level: 'debug',
  }),
  observability: {
    default: { enabled: true },
  },
  server: {
    build: {
      openAPIDocs: true,
      swaggerUI: true,
    },
    apiRoutes: [a2aAgentRoute],
  },
  bundler: {
    externals: ["google-play-scraper"],
  },
});
</code></pre>

</div>



<p>What Worked: Mastra's centralized configuration made it easy to see the full picture and manage all components in one place.</p>

<p><strong>Deployment to Mastra Cloud</strong></p>

<p>Deployment was surprisingly straightforward:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Build and test locally first
mastra dev
</code></pre>

</div>



<p>You can use the above to build your work locally. If all is fine, you should see this on your terminal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>◐ Preparing development environment...
✓ Initial bundle complete
◇ Starting Mastra dev server...
WARN [2025-11-01 12:47:40.680 +0100] (PlayStoreAgent): Mastra telemetry is deprecated and will be removed on the Nov 4th release. Instead use AI Tracing. More info can be found here: https://github.com/mastra-ai/mastra/issues/8577 and here: https://mastra.ai/en/docs/observability/ai-tracing/overview
mastra-cloud-ai-tracing-exporter disabled: MASTRA_CLOUD_ACCESS_TOKEN environment variable not set. 🚀 Sign up for Mastra Cloud at https://cloud.mastra.ai to see your AI traces online and obtain your access token.

 mastra  0.17.7 ready in 25760 ms

│ Playground: http://localhost:4111/
│ API:        http://localhost:4111/api
</code></pre>

</div>



<p>To deploy and get your URL:</p>

<ul>
<li>Simply push your code to GitHub</li>
<li>Go to Mastra Cloud</li>
<li>Import your GitHub repo and start your deployment</li>
</ul>

<p>If all is successful, you should get your domain URL:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://playstore-rating-fetcher-agent.mastra.cloud/a2a/agent/playStoreAgent
</code></pre>

</div>



<p>What Worked: The deployment process was seamless. Mastra Cloud handles infrastructure, scaling, and monitoring automatically - no complex configuration needed.</p>

<p>What Didn't: Initially confused about whether I needed to set up infrastructure. Turns out Mastra Cloud handles everything. Also tried deploying using <code>mastra deploy</code> command, but it didn't work. The GitHub integration approach was more reliable.</p>

<p><strong>Testing the Integration</strong></p>

<p>Before integrating with Telex, I tested the endpoint directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>curl -X POST https://playstore-rating-fetcher-agent.mastra.cloud/a2a/agent/playStoreAgent \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "test-001",
    "method": "message/send",
    "params": {
      "message": {
        "kind": "message",
        "role": "user",
        "parts": [{"kind": "text", "text": "What is the rating for Instagram?"}],
        "messageId": "msg-001",
        "taskId": "task-001"
      },
      "configuration": {"blocking": true}
    }
  }'
</code></pre>

</div>



<p>What Worked: Getting a proper JSON response confirmed the A2A implementation was correct.</p>

<p><strong>Integrating with Telex.im</strong></p>

<p>The final step was creating the Telex workflow configuration. Following the weather agent example, I created:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "active": false,
  "category": "utilities",
  "description": "A workflow that gives Google Play Store app ratings",
  "id": "pR8sT0r3aG3nT1dX",
  "long_description": "\n      You are a helpful assistant...",
  "name": "playstore_rating_agent",
  "nodes": [
    {
      "id": "playstore_agent",
      "name": "playstore agent",
      "parameters": {},
      "position": [816, -112],
      "type": "a2a/mastra-a2a-node",
      "typeVersion": 1,
      "url": "https://playstore-rating-fetcher-agent.mastra.cloud/a2a/agent/playStoreAgent"
    }
  ],
  "pinData": {},
  "settings": {
    "executionOrder": "v1"
  },
  "short_description": "Get app ratings from Google Play Store"
}
</code></pre>

</div>



<p>What Worked: Once I matched the exact format of the weather agent example (including lowercase naming, position arrays, etc.), it integrated perfectly with Telex.im.</p>

<p>Try it yourself: Check out the live PlayStore Rating Agent on <a href="https://telex.im" rel="noopener noreferrer">Telex.im</a> - just ask it about any Android app like "What's the rating for Spotify?" and it will instantly fetch real-time data from the Google Play Store including ratings, review counts, install numbers, and developer information.</p>

<p><strong>Key Challenges and Solutions</strong></p>

<p><strong>Challenge 1: Raw JSON in Responses</strong></p>

<p>Problem: Agent was showing tool output directly to users<br>
Solution: Updated agent instructions to explicitly format responses conversationally</p>

<p><strong>Challenge 2: Telex Workflow Format</strong></p>

<p>Problem: My initial workflow JSON didn't match Telex's expected format<br>
Solution: Followed the weather agent example exactly - lowercase names, specific position format, no extra fields</p>

<p><strong>Challenge 3: A2A Protocol Compliance</strong></p>

<p>Problem: Understanding the artifacts and the history structure<br>
Solution: Studied the blog post example carefully and implemented the exact same structure</p>

<p><strong>Results and Performance</strong></p>

<p>The final agent works beautifully on Telex.im (a Slack alternative for bootcamps and communities with AI agent capabilities):</p>

<p>✅ Fast: Responses in 1-2 seconds<br>
✅ Accurate: Pulls real-time data from Play Store<br>
✅ User-Friendly: Clean, formatted responses<br>
✅ Reliable: Proper error handling for edge cases<br>
✅ Integrated: Works seamlessly with Telex.im</p>

<p><strong>Lessons Learned</strong></p>

<p><strong>Start with Examples:</strong> The weather agent template was invaluable. Don't reinvent the wheel.</p>

<p><strong>Be Explicit with AI:</strong> Agent instructions need to be very detailed. The formatting issue taught me this.</p>

<p><strong>Follow Standards Strictly:</strong> The A2A protocol has specific requirements. Don't deviate.</p>

<p><strong>Test Incrementally:</strong> Testing each component (tool → agent → workflow → A2A) separately made debugging much easier.</p>

<p><strong>Error Handling is Critical:</strong> Always assume external APIs can fail. Handle errors gracefully.</p>

<p><strong>Documentation Matters:</strong> Reading the Mastra docs and the integration blog post saved hours of trial and error. It would have saved me more if I had started with it first.</p>

<p><strong>Leverage Mastra's Features:</strong> Using built-in features like Memory, Scorers, and Workflows saved me from building infrastructure from scratch.</p>

<p><strong>Future Enhancements</strong></p>

<p>Some ideas for v2:</p>

<p>Trend Analysis: Track rating changes over time and alert on significant drops</p>

<p>Competitor Comparison: Compare multiple similar apps side-by-side</p>

<p>iOS Support: Add App Store integration using a similar pattern</p>

<p>Review Sentiment: Analyze user reviews for sentiment trends</p>

<p>Automated Reports: Generate weekly summaries for monitored apps</p>

<p><strong>Conclusion</strong></p>

<p>Building this PlayStore Rating Agent was an excellent learning experience. The Mastra framework made it surprisingly easy to create a production-ready AI agent with features like custom tools, persistent memory, workflow orchestration, and scoring systems. The A2A protocol ensured seamless integration with Telex.im, an AI agent platform like Zapier that doubles as a Slack alternative for education and communities.</p>

<p><strong>The key takeaways:</strong></p>

<ul>
<li>Mastra's feature-rich framework (Agents, Tools, Memory, Workflows, Scorers) accelerates development</li>
<li>The A2A protocol provides real standardization for agent communication</li>
<li>Starting with good examples accelerates development significantly</li>
<li>Proper error handling and clear agent instructions are non-negotiable</li>
<li>Mastra Cloud's deployment process eliminates infrastructure headaches</li>
</ul>

<p>If you're building AI agents that need to integrate with platforms like Telex.im, I highly recommend the Mastra + A2A approach. It saves you from reinventing the wheel while maintaining flexibility for your specific use case.</p>

<p>Want to try it? <a href="https://telex.im/ai-coworkers/playstore-rating-fetcher-agent-40fda3935c73" rel="noopener noreferrer">Check out the PlayStore Rating Agent on Telex.im</a> and ask it about any Android app!</p>

<p><strong>Resources</strong></p>

<p><strong>Mastra Documentation:</strong> <a href="https://docs.mastra.ai" rel="noopener noreferrer">https://docs.mastra.ai</a></p>

<p><strong>Telex.im:</strong> <a href="https://telex.im" rel="noopener noreferrer">https://telex.im</a> - An AI agent platform like Slack alternative for communities, education, and bootcamps</p>

<p><strong>A2A Protocol Blog:</strong> <a href="https://fynix.dev/blog/telex-x-mastra" rel="noopener noreferrer">https://fynix.dev/blog/telex-x-mastra</a></p>

<p><strong>Source Code:</strong> github.com/toluwanithepm/playstore-rating-fetcher-agent</p>

<p><strong>Live Agent:</strong> <a href="https://telex.im/ai-coworkers/playstore-rating-fetcher-agent-40fda3935c73" rel="noopener noreferrer">PlayStore Rating Agent on Telex.im</a></p>

<p>Have you built any AI agents with Mastra? What challenges did you face? I'd love to hear about your experience in the comments!</p>

