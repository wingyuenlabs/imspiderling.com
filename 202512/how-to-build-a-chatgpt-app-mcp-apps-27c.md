---
Title: How to build a ChatGPT App (MCP Apps)
Description: 
Author: Abe Wheeler
Date: 2025-12-05T21:59:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>There isn’t much content out there to help developers build their embedded ChatGPT Apps, so I figured I’d do a quick consolidation &amp; write-up. As far as I’ve seen, this is the extent of the official tooling:</p>

<ul>
<li>
<a href="https://developers.openai.com/apps-sdk" rel="noopener noreferrer">OpenAI Apps SDK Documentation</a> (launched 10/6) - overall documentation.</li>
<li>
<a href="https://github.com/openai/openai-apps-sdk-examples" rel="noopener noreferrer">openai/openai-apps-sdk-examples</a> (launched 10/6) - 7 example apps.</li>
<li>
<a href="https://github.com/openai/apps-sdk-ui" rel="noopener noreferrer">openai/apps-sdk-ui</a> (launched 11/21) - low-level React component library consistent with the design requirements.</li>
<li>
<a href="https://github.com/modelcontextprotocol/ext-apps" rel="noopener noreferrer">MCP App Extension protocol</a> (launched 11/21) - generic protocol of the ChatGPT App runtime/interface.</li>
</ul>

<p>I won’t rehash the documentation basics. Instead, I’ll review my biggest takeaways after building a few apps.</p>

<p><strong>Lesson 1: Embrace MCP</strong></p>

<p>The ChatGPT App documentation makes Apps sound like they use MCP, but they’re not MCP themselves. That’s not quite right. Think of these apps as a GUI feature of MCP, and architect your apps entirely according to MCP concepts. Every UI/page is just a <a href="https://modelcontextprotocol.io/specification/2025-11-25/server/resources" rel="noopener noreferrer"><strong>Resource</strong></a> and every API is just a <a href="https://modelcontextprotocol.io/specification/2025-11-25/server/tools" rel="noopener noreferrer"><strong>Tool</strong></a>. Get comfortable with those abstractions. An <strong>App</strong> has one or more <strong>Resources</strong>, a <strong>Resource</strong> has one or more <strong>Tools</strong>.</p>

<p>My original toy apps didn’t properly adhere to those boundaries, and I found the abstractions I naturally built broke down when they came in contact with production ChatGPT. It’s a bit easier to recognize the core abstraction now that MCP started adding these interfaces to the protocol, but it’s only been a week and a half since they started, and the interfaces are still highly unstable.</p>

<p><strong>Lesson 2: Invalidate all the caches</strong></p>

<p>When deploying your App to ChatGPT, it can be difficult to tell if your Resource changes have been picked up. To make sure you’re always interacting with the latest version, you have to update the Resource URI on your MCP server <strong>AND</strong> “Refresh” your App from the ChatGPT Connector modal on every single change. I set up my project to append a base-32 timestamp to Resource URIs on every build so they always cache-bust on the ChatGPT side, but I still always have to refresh the connection on every UI change.</p>

<p><strong>Lesson 3: But Wait! There’s More!</strong></p>

<p>The official OpenAI documentation lists only about 2/3 of the actual runtime API. I’m not God or sama, so I can’t say that these undocumented fields are here to stay, but you can build more functionality than currently explained. Here’s the complete global runtime list that I just queried from my app running in ChatGPT:</p>

<ol>
<li>callCompletion: <em>(...i)=&gt; {…}</em>
</li>
<li>callTool: <em>(...i)=&gt; {…}</em>
</li>
<li>displayMode: "inline"</li>
<li>downloadFile: <em>(...i)=&gt; {…}</em>
</li>
<li>locale: "en-US"</li>
<li>maxHeight: undefined</li>
<li>notifyEscapeKey: <em>(...i)=&gt; {…}</em>
</li>
<li>notifyIntrinsicHeight: <em>(...i)=&gt; {…}</em>
</li>
<li>notifyNavigation: <em>(...i)=&gt; {…}</em>
</li>
<li>notifySecurityPolicyViolation: <em>(...i)=&gt; {…}</em>
</li>
<li>openExternal: <em>(...i)=&gt; {…}</em>
</li>
<li>openPromptInput: <em>(...i)=&gt; {…}</em>
</li>
<li>requestCheckout: <em>(...i)=&gt; {…}</em>
</li>
<li>requestClose: <em>(...i)=&gt; {…}</em>
</li>
<li>requestDisplayMode: <em>(...i)=&gt; {…}</em>
</li>
<li>requestLinkToConnector: <em>(...i)=&gt; {…}</em>
</li>
<li>requestModal: <em>(...i)=&gt; {…}</em>
</li>
<li>safeArea: {insets: {…}}</li>
<li>sendFollowUpMessage: <em>(...i)=&gt; {…}</em>
</li>
<li>sendInstrument: <em>(...i)=&gt; {…}</em>
</li>
<li>setWidgetState: <em>u=&gt; {…}</em>
</li>
<li>streamCompletion: <em>(...l)=&gt; {…}</em>
</li>
<li>subjectId: "v1/…"</li>
<li>theme: "dark"</li>
<li>toolInput: {}</li>
<li>toolOutput: {text: 'Rendered Show a simple counter tool!'}</li>
<li>toolResponseMetadata: null</li>
<li>updateWidgetState: <em>(...i)=&gt; {…}</em>
</li>
<li>uploadFile: <em>(...i)=&gt; {…}</em>
</li>
<li>userAgent: {device: {…}, capabilities: {…}}</li>
<li>view: {params: null, mode: 'inline'}</li>
<li>widget: {state: {…}, props: {…}, setState: <em>ƒ</em>}</li>
<li>widgetState: {count: 0}</li>
</ol>

<p>Be careful with the <a href="https://github.com/openai/openai-apps-sdk-examples" rel="noopener noreferrer">example apps</a>. They don’t respect all of these platform globals, documented or not. They also still don’t use the <a href="https://github.com/openai/apps-sdk-ui" rel="noopener noreferrer">apps-sdk-ui</a> React component library (as of this writing), so they’re already pretty outdated.</p>

<p>Hope that was helpful! If you’re interested in playing around with ChatGPT Apps, I built an open-source quickstart &amp; local ChatGPT simulator that I’ve found really helpful for visualizing the runtime &amp; iterating quickly. I hosted it here if you want to play around with it!</p>

<p><a href="https://sunpeak.ai/#simulator" rel="noopener noreferrer">https://sunpeak.ai/#simulator</a></p>

<p>Would really <a href="https://github.com/Sunpeak-AI/sunpeak" rel="noopener noreferrer">appreciate a star</a> if you can spare one!</p>

