---
Title: Generate design skill files for your agentic tools with TypeUI
Description: 
Author: Zoltán Szőgyényi
Date: 2026-03-07T21:33:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>We’ve all been there. You’re using Cursor, Claude Code, or Copilot, and you ask it to whip up a quick dashboard component. The logic is flawless, the state management is perfect, and then... you look at the UI.</p>

<p>Suddenly, your app has three new shades of blue that don't exist in your Tailwind config, the padding is completely arbitrary, and the AI has decided to invent its own design system on the fly.</p>

<p>AI models are absolute wizards at writing logic, but without strict constraints, they hallucinate design tokens like crazy.</p>

<p>That’s why I want to talk about typeui.sh — an awesome open-source CLI tool that acts as a dedicated design layer for your agentic tooling.</p>

<h2>
  
  
  What exactly is typeui.sh?
</h2>

<p>Think of <a href="https://typeui.sh/" rel="noopener noreferrer">typeui.sh</a> as a strict design manager for your AI assistants.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwidy7fr9jmuy5tl4oh3w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwidy7fr9jmuy5tl4oh3w.png" alt="screenshot of typeui.sh terminal" width="800" height="374"></a></p>

<p>It’s a CLI tool that generates and manages skill.md files right inside your project. These files act as the ultimate source of truth for your design system and style guide. </p>

<p>By feeding these highly curated instructions to your AI, you force the LLM to actually respect your design tokens, component structures, and styling rules.</p>

<p>No more fighting over CSS classes. You just get polished, production-ready UI components that actually match your brand.</p>

<h2>
  
  
  The 4 Commands You Need to Know
</h2>

<p>The CLI is super intuitive and stays out of your way. Here is the core workflow:</p>

<h2>
  
  
  1. generate
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx typeui.sh generate
</code></pre>

</div>



<p>This is where the magic starts. Instead of writing a massive, 500-line system prompt explaining what a "primary button" looks like in your app, this command scaffolds the provider skill files directly in your current project. Your AI assistant reads this file and instantly understands your design constraints.</p>

<h2>
  
  
  2. update
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx typeui.sh update
</code></pre>

</div>



<p>Design systems aren't static. When you tweak your spacing scales or add a new brand color, you just run update. It refreshes your existing provider skill files so your AI is always working with the latest guidelines, without nuking your custom configurations.</p>

<h2>
  
  
  3. list
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx typeui.sh list
</code></pre>

</div>



<p>Don't want to build a design system from scratch? I don't blame you. Running list lets you browse a registry of pre-built design system specifications. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fej5137u6d3eys51gg315.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fej5137u6d3eys51gg315.png" alt="design systems on typeui" width="800" height="599"></a></p>

<p>It’s perfect for hackathons or when you just want to bootstrap a new project with a gorgeous, proven design language.</p>

<h2>
  
  
  4. pull
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx typeui.sh pull [name]
</code></pre>

</div>



<p>Once you spot a design system you like in the registry, just pull it. For example, pulling a "modern" spec will download that specific markdown file directly into your workspace. It comes pre-loaded with comprehensive design tokens, instantly teaching your AI how to design within that specific aesthetic.</p>

<h2>
  
  
  Open source
</h2>

<p>The best part? The core CLI is <a href="https://github.com/bergside/typeui.sh" rel="noopener noreferrer">open-source under the MIT license</a>. You can use it for your local workflows entirely for free.</p>

<p>If you're a freelancer or an agency looking for access to our <a href="//typeui.sh/design-systems">curated premium design systems</a> and priority support, they also offer one-time payment licenses (which is a breath of fresh air in today's subscription-fatigued world).</p>

<h2>
  
  
  Wrapping up
</h2>

<p>As we rely more on AI to write our code, controlling the output is becoming just as important as the generation itself. If you're tired of playing CSS whack-a-mole with your LLM, give this a try.</p>

<p>Check out the docs and try it out here: <a href="https://typeui.sh/" rel="noopener noreferrer">typeui.sh</a></p>

