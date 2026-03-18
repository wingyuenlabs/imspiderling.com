---
Title: AI Coding Tip 011 - Initialize Agents.md
Description: 
Author: Maxi Contieri
Date: 2026-03-18T22:12:31.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Summarize your architecture rules and give your AI a persistent memory</em></p>

<blockquote>
<p>TL;DR: When you use the /init command, you create a context file that saves you from repeating instructions in every new session.</p>
</blockquote>

<h1>
  
  
  Common Mistake ❌
</h1>

<p>You waste time copying and pasting the same project rules into every new chat. </p>

<p>You forgot to tell the AI about your specific test commands. </p>

<p>When you start a fresh session, the AI loses all previous context about your coding style or architecture.</p>

<p>Create your own <em>system prompts</em>, persist them and keep an eye on them regularly.</p>

<h1>
  
  
  Problems Addressed 😔
</h1>

<ul>
<li>Repetitive and manual meta-prompting</li>
<li>Inconsistent <a href="https://dev.to/mcsee/code-smell-48-code-without-standards-2f5m">code styles</a> across sessions</li>
<li>High token usage from redundant instructions</li>
<li>AI hallucinations about build or test commands</li>
<li>Onboarding new projects</li>
</ul>

<h1>
  
  
  How to Do It 🛠️
</h1>

<ol>
<li>Open your terminal in your project root directory.</li>
<li>Type <a href="https://opencode.ai/docs/rules/" rel="noopener noreferrer"><code>/init</code></a> if you use Claude Code or OpenCode.</li>
<li>Review the generated <code>CLAUDE.md</code> or <code>AGENTS.md</code> file.</li>
<li>Add specific "landmines" or things the AI can't see in the code.</li>
<li>Save the file, <a href="https://dev.to/mcsee/ai-coding-tip-006-review-every-line-before-commit-bmm">review the commit diffs</a>, and start your next coding task.</li>
<li>You can do it again when you have new rules.</li>
</ol>

<h1>
  
  
  Benefits 🎯
</h1>

<p>When you initialize (/Init) your project, the AI reads all the repository immediately. </p>

<p>You get consistent results without extra effort. </p>

<p>You save tokens because you don't have to describe your stack every time.</p>

<h1>
  
  
  Context 🧠
</h1>

<p>AI assistants start every session with a blank slate. </p>

<p>They don't remember what you told them yesterday. </p>

<p>Tools like Claude Code look for a specific file to understand your project. </p>

<p>When you provide this saved file, you close the gap between separate AI sessions.</p>

<h2>
  
  
  Prompt Reference 📝
</h2>

<p><strong>Bad Prompt:</strong></p>



<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>Remember to use 2 spaces for indentation 
and always run 'npm run test:unit' 
before you finish the task.
</code></pre>

</div>


<p><strong>Good Prompt:</strong></p>




<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>git clone https://github.com/torvalds/linux.git

/Init
</code></pre>

</div>

<h1>
  
  
  Considerations ⚠️
</h1>

<p>Keep the generated file short. </p>

<p>When you add too much text, the AI might get confused. </p>

<p>Don't include information that the AI can already find in your <code>package.json</code> or <code>README.md</code>.</p>
<h1>
  
  
  Type 📝
</h1>

<p>[X] Semi-Automatic</p>
<h1>
  
  
  Limitations ⚠️
</h1>

<p>You can only use this tip with AI tools that support persistent context files. </p>

<p>If you change your build tools, you can update the Markdown file manually, run /init again, or do both.</p>

<p>In Windsurf, there is no literal /init terminal command that automatically generates a configuration file, as Claude Code or OpenCode does.</p>

<p>Windsurf uses a more agentic approach where the AI (Cascade) either discovers your rules automatically or creates them when you ask.</p>

<blockquote>
<p>Cascade, initialize a .windsurfrules file for this project. Include my tech stack, coding standards, and build commands.</p>
</blockquote>
<h1>
  
  
  Tags 🏷️
</h1>

<ul>
<li>Standards</li>
</ul>
<h1>
  
  
  Level 🔋
</h1>

<p>[X] Beginner</p>
<h1>
  
  
  Related Tips 🔗
</h1>


<div class="ltag__link">
  <a href="/mcsee" class="ltag__link__link">
    <div class="ltag__link__pic">
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F366059%2F44d4a869-bb26-4b8e-aa73-6e596b4b4b8a.jpg" alt="mcsee">
    </div>
  </a>
  <a href="https://dev.to/mcsee/ai-coding-tip-006-review-every-line-before-commit-bmm" class="ltag__link__link">
    <div class="ltag__link__content">
      <h2>AI Coding Tip 006 - Review Every Line Before Commit</h2>
      <h3>Maxi Contieri ・ Feb 13</h3>
      <div class="ltag__link__taglist">
        <span class="ltag__link__tag">#ai</span>
        <span class="ltag__link__tag">#programming</span>
        <span class="ltag__link__tag">#development</span>
        <span class="ltag__link__tag">#softwaredevelopment</span>
      </div>
    </div>
  </a>
</div>



<ul>
<li><p>Use <em>.cursorrules</em> for local preferences</p></li>
<li><p>Create modular <em>AGENTS.md</em> files for monorepos</p></li>
<li><p>Keep your <em>README.md</em> updated for better AI indexing</p></li>
</ul>

<h1>
  
  
  Conclusion 🏁
</h1>

<p>When you use <code>/init</code>, you turn your AI from a temporary contributor into a permanent team member.</p>

<h1>
  
  
  More Information ℹ️
</h1>

<p><a href="https://opencode.ai/docs/rules/" rel="noopener noreferrer">OpenCode /Init</a></p>

<p><a href="https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview%23claudemd-guide" rel="noopener noreferrer">Claude Code: CLAUDE.md</a></p>

<p><a href="https://docs.cursor.com/context/rules-for-ai" rel="noopener noreferrer">.cursorrules in Cursor</a></p>

<p><a href="https://docs.windsurf.com/windsurf/cascade/memories" rel="noopener noreferrer">Windsurf rules</a></p>

<h1>
  
  
  Also Known As 🎭
</h1>

<ul>
<li><p>Project-AI-Onboarding</p></li>
<li><p>Context-Bootstrapping</p></li>
</ul>

<h1>
  
  
  Tools 🧰
</h1>

<ul>
<li><p>Claude Code</p></li>
<li><p>OpenCode</p></li>
<li><p>Cursor (via .cursorrules)</p></li>
</ul>

<h1>
  
  
  Disclaimer 📢
</h1>

<p>The views expressed here are my own.</p>

<p>I am a human who writes as best as possible for other humans. </p>

<p>I use AI proofreading tools to improve some texts.</p>

<p>I welcome constructive criticism and dialogue.</p>

<p>I shape these insights through 30 years in the software industry, 25 years of teaching, and writing over 500 articles and a book.</p>




<p>This article is part of the <em>AI Coding Tip</em> series.</p>


<div class="ltag__link">
  <a href="/mcsee" class="ltag__link__link">
    <div class="ltag__link__pic">
      <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F366059%2F44d4a869-bb26-4b8e-aa73-6e596b4b4b8a.jpg" alt="mcsee">
    </div>
  </a>
  <a href="https://dev.to/mcsee/ai-coding-tips-118n" class="ltag__link__link">
    <div class="ltag__link__content">
      <h2>AI Coding Tips</h2>
      <h3>Maxi Contieri ・ Jan 21</h3>
      <div class="ltag__link__taglist">
        <span class="ltag__link__tag">#webdev</span>
        <span class="ltag__link__tag">#ai</span>
        <span class="ltag__link__tag">#programming</span>
        <span class="ltag__link__tag">#beginners</span>
      </div>
    </div>
  </a>
</div>


