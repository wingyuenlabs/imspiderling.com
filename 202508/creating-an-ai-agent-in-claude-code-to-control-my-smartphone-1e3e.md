---
Title: Creating an AI Agent in Claude Code to Control my Smartphone
Description: 
Author: Tiago Danin
Date: 2025-08-22T19:19:38.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Automating Android app documentation and testing with artificial intelligence</strong></p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/zp38frEGcPA">
</iframe>
</p>

<h2>
  
  
  The Initial Challenge
</h2>

<p>I was facing a specific challenge: I needed to automatically document various flows of an Android application. That's when I started exploring ways to accomplish this process, and one of the tools I decided to test was Claude Code.</p>

<p>Initially, Claude Code didn't adapt very well. However, as an Android developer, during a chat conversation, it was demonstrated how to use ADB commands to simulate user actions like clicks, typing, and other interactions, so that the AI could control my smartphone.</p>

<p>I thought about looking for an MCP (Model Context Protocol) that already implemented this functionality, but I couldn't find any suitable one, so I decided to build my own: <a href="https://github.com/TiagoDanin/Android-Debug-Bridge-MCP" rel="noopener noreferrer">Android-Debug-Bridge-MCP</a>.</p>

<h3>
  
  
  The Power of ADB UI Automator
</h3>

<p>One of the most valuable features of ADB is the UI Automator, which returns a detailed XML of the current interface. This makes the use of the <code>adb shell tap x y</code> command much more precise for clicking on specific screen elements, for example, since it shows exactly their positions and the LLM understands the XML format better than an image.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2kasp7z92qy43ktzibyr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2kasp7z92qy43ktzibyr.png" alt=" " width="800" height="384"></a></p>

<p>What I needed to organize was:</p>

<ul>
<li>What types of elements to detect in the interface</li>
<li>How to show the content and position of each element</li>
<li>How to allow Claude Code to execute the correct action at the exact position</li>
</ul>

<h3>
  
  
  Evolving the Agent
</h3>

<p>After integrating Claude Code with my MCP, I rewrote the agent and gradually improved it until I reached an efficient flow. My AI agent follows a simple but effective pattern:</p>

<ol>
<li>
<strong>Visualize</strong> the current screen content</li>
<li>
<strong>Execute</strong> a specific action</li>
<li>
<strong>Repeat</strong> the process</li>
<li>
<strong>Generate</strong> a complete test report</li>
</ol>

<h2>
  
  
  Real Example: Testing Stock Pulse App
</h2>

<p>I'll show how the agent automatically tested the process of adding a stock (NVDA) within an app I helped develop called StockPulse:</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/zp38frEGcPA">
</iframe>
</p>

<h3>
  
  
  Command Used
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@agent-app-tester Open br.com.idopterlabs.Tickerapp, add an Nvidia stock to the portfolio, click Save, and I expect the stock's current data to be displayed on the screen. 
</code></pre>

</div>



<p>The agent then executed the entire flow automatically. It created a test folder called <code>tickerapp_nvda_test</code>, opened the StockPulse app, and began its work. First, it captured an initial screenshot showing the empty portfolio screen. Then it navigated to add a new stock by tapping the "+" button and captured the stock selection screen.</p>

<p>Next, the agent selected NVIDIA (NVDA) from the list, verified the stock details were properly displayed, and generated a complete report with all screenshots and results documented.</p>

<h3>
  
  
  Final Result
</h3>

<p>The agent automatically generated:</p>

<ul>
<li>Organized screenshots</li>
<li>Complete markdown report</li>
<li>Documentation of each step</li>
<li>Success/failure status for each stage</li>
</ul>

<p>All of this in less than 2 minutes, without manual intervention!</p>

<h2>
  
  
  Use Cases and Possibilities
</h2>

<p>The combination of MCP with Claude Code opens up various possibilities:</p>

<p><strong>Test Automation</strong> becomes incredibly powerful with this approach. You can execute automated regression tests for critical flows without any manual intervention, validate interface elements and layouts automatically, and simulate realistic user interactions for usability testing.</p>

<p><strong>Intelligent Documentation</strong> is another game-changer. The system can automatically capture organized screenshots for app documentation, create detailed documentation of user journeys within applications, generate technical manuals with step-by-step guides and visual evidence, and even perform periodic monitoring to verify application states.</p>

<p>The possibilities are practically infinite - any need involving automated control of Android devices can benefit from this solution.</p>

<h2>
  
  
  Quick Setup
</h2>

<p>Getting started is straightforward. You'll need ADB installed and configured on your system, an Android device connected or emulator running, and Claude Code properly set up.</p>

<p>To install the MCP, simply run <code>npm install -g android-debug-bridge-mcp</code> in your terminal. Then add it to Claude Code with the command <code>claude mcp add --scope project android-debug-bridge-mcp -- npx android-debug-bridge-mcp</code>.</p>

<p>Next, configure a custom agent with specific prompts for your test cases. You can find my configuration at: <a href="https://github.com/TiagoDanin/Android-Debug-Bridge-MCP/issues/4" rel="noopener noreferrer">app-tester.md</a></p>

<p>Once everything is set up, you can start executing tests using simple and direct commands to begin automation.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Claude Code, when combined with the right tools and well-structured instructions, demonstrates impressive potential for automation. This experiment represents only the beginning of a broader exploration - there's plenty of room for future improvements.</p>

<p>It's important to recognize that for more complex scenarios, the expertise of a professional QA is still necessary. AI automation complements, but doesn't completely replace, specialized testing knowledge.</p>

<p>Despite the limitations, I confess it was extremely gratifying to develop this solution. It not only met my initial objective of documenting application flows but also opened doors to new automation possibilities that previously seemed impractical.</p>

<p>The project will continue evolving, and I hope to bring new features in the future.</p>

<blockquote>
<p>Thanks to my friend Iago Cavalcante for reviewing the translation and introducing me to Claude Code.</p>
</blockquote>

