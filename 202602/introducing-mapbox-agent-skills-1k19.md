---
Title: Introducing Mapbox Agent Skills
Description: 
Author: Matthew Podwysocki
Date: 2026-02-11T22:09:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>I'm excited to share something we've been working on that makes building with Mapbox a lot easier, especially if you're using AI coding assistants like Claude Code, Cursor, or GitHub Copilot.</p>

<p>We've built a collection of Agent Skills that teach AI assistants how to build better Mapbox applications. Think of it as giving your AI assistant a crash course in Mapbox development best practices.</p>

<h2>
  
  
  TL;DR
</h2>

<p>Agent Skills teach AI assistants Mapbox development best practices. We've built 15 skills and counting covering web/iOS/Android integration, performance optimization, search implementation, geospatial operations, map design, migrations, security, and common patterns. Install them and your AI will know things like "cluster 1000+ markers" and "use Search Box API for autocomplete."</p>

<p><strong>Install all skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx add-skill mapbox/mapbox-agent-skills
</code></pre>

</div>



<p><strong>Install specific skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx add-skill mapbox/mapbox-agent-skills <span class="nt">--skill</span> mapbox-web-performance-patterns
</code></pre>

</div>



<p><strong>List available skills:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx add-skill mapbox/mapbox-agent-skills <span class="nt">--list</span>
</code></pre>

</div>



<h2>
  
  
  What Are Skills?
</h2>

<p>If you've been using AI coding assistants, you've probably noticed they're pretty good at writing code, but sometimes they miss the nuances. They might suggest patterns that work but aren't optimal, or miss important details about how things should be done.</p>

<p><a href="https://agentskills.io/home" rel="noopener noreferrer">Agent Skills</a> are a way to fix this. They're folders with instructions and guidance that AI assistants can load and use when helping you build. Unlike tools (which let AI do things) or prompts (which give specific instructions), skills provide domain expertise. The know-how.</p>

<p>When an AI assistant has access to Mapbox skills, it knows things like "if you're adding 1000+ markers, you should use clustering" or "use the Search Box API for autocomplete, not the Geocoding API." It's the difference between an AI that knows how to code and one that knows how to code Mapbox apps specifically.</p>

<h2>
  
  
  See It In Action
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmszl57sdlm1lpru87kcr.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmszl57sdlm1lpru87kcr.gif" alt="Store Locator App" width="760" height="405"></a></p>

<p><em>This production-ready store locator was built in minutes using Claude Code with Mapbox Agent Skills. The AI assistant handled marker patterns, distance calculations, search functionality, and responsive design automatically—following Mapbox best practices from the start.</em></p>

<h2>
  
  
  Why Skills Are Important
</h2>

<p>Here's the thing: Mapbox has a lot of surface area. We have SDKs for <a href="https://docs.mapbox.com/mapbox-gl-js/guides/" rel="noopener noreferrer">web</a>, <a href="https://docs.mapbox.com/ios/maps/guides/" rel="noopener noreferrer">iOS</a>, and <a href="https://docs.mapbox.com/android/maps/guides/" rel="noopener noreferrer">Android</a>. We have multiple search APIs with different use cases. We have performance patterns that matter at scale. We have migration paths from other platforms.</p>

<p>When you're building with an AI assistant, you want it to know all this stuff. You don't want to spend time correcting it when it suggests using 1000 individual markers instead of clustering. You don't want it to miss that you should debounce search requests. You want it to just know.</p>

<p>That's what these skills do. They encode the knowledge we've built up from working with thousands of developers building Mapbox applications. The patterns that work. The mistakes to avoid. The right tool for the job.</p>

<h2>
  
  
  What's Inside the Box
</h2>

<p>We've organized the skills into a few main areas. Here's what's available:</p>

<h3>
  
  
  Platform Integration
</h3>

<p><strong>Web, iOS, and Android patterns</strong> - These skills cover the fundamentals of integrating Mapbox into your app, whether you're building for the web with React, Vue, Svelte, or Angular, or going native with Swift/SwiftUI on iOS or Kotlin/Jetpack Compose on Android. They know about framework-specific patterns, lifecycle management, and platform best practices.</p>

<h3>
  
  
  Performance
</h3>

<p><strong>Web performance patterns</strong> - This one's huge. It covers everything from initialization strategies to handling thousands of markers, optimizing data loading, and managing memory. If you're building something that needs to perform at scale, this skill will save you a lot of debugging time.</p>

<h3>
  
  
  Search
</h3>

<p><strong>Search integration and patterns</strong> - We have two skills here. One walks through the complete workflow of adding search to your app (asking the right discovery questions, picking the right product, implementing it correctly). The other helps you choose between our different search tools and configure them properly. Between these two, your AI assistant will know exactly how to implement search for your use case.</p>

<h3>
  
  
  Geospatial Operations
</h3>

<p><strong>Geometric calculations and routing</strong> - This skill helps decide when to use offline geometric calculations (like measuring straight-line distance) versus when to use our routing APIs (like getting actual drive times). It's the "as the crow flies vs as the crow drives" decision framework, plus a ton of practical examples.</p>

<h3>
  
  
  Design and Cartography
</h3>

<p><strong>Map design, styles, and quality</strong> - Three skills covering cartographic principles (color theory, visual hierarchy, typography), common style patterns and layer configurations, and style validation and optimization. If you're doing custom map design, these will help you make maps that look professional and work well.</p>

<h3>
  
  
  Migration
</h3>

<p><strong>Google Maps and MapLibre migration</strong> - If you're moving from Google Maps Platform or MapLibre, these skills provide comprehensive migration guides with API equivalents and pattern translations. They know the gotchas and the breaking changes.</p>

<h3>
  
  
  Security
</h3>

<p><strong>Token security</strong> - Best practices for handling access tokens securely across all platforms. This is one of those things that's easy to get wrong, so having your AI assistant know the right patterns from the start is valuable.</p>

<h3>
  
  
  Common Patterns
</h3>

<p><strong>Store locators</strong> - A focused skill on building location finders and store locators, with patterns for markers, filtering, distance calculations, and all the UX details that make these features work well.</p>

<h2>
  
  
  Getting Started
</h2>

<p>Installing skills is straightforward. If you're using Claude Code, Cursor, or another AI assistant that supports Agent Skills, you can add them all at once:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx add-skill mapbox/mapbox-agent-skills
</code></pre>

</div>



<p>Or install specific skills if you only need certain ones:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx add-skill mapbox/mapbox-agent-skills <span class="nt">--skill</span> mapbox-web-performance-patterns
</code></pre>

</div>



<p>You can list what's available:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npx add-skill mapbox/mapbox-agent-skills <span class="nt">--list</span>
</code></pre>

</div>



<p>Once installed, your AI assistant will automatically use these skills when you're working on Mapbox projects. You don't need to do anything special. Just start building and you'll notice better suggestions, fewer mistakes, and more production-ready code.</p>

<p><strong>Pro tip:</strong> These skills work great with our Mapbox MCP servers. We have the <a href="https://github.com/mapbox/mcp-server" rel="noopener noreferrer">Mapbox MCP Server</a> for core Mapbox operations and the <a href="https://github.com/mapbox/mcp-devkit-server" rel="noopener noreferrer">Mapbox MCP DevKit Server</a> for development workflows. Skills provide the expertise (how to design good maps, which patterns to use), while MCP tools provide the actions (create styles, generate tokens, preview maps). Together they make a pretty powerful development workflow.</p>

<h2>
  
  
  Build Stuff and Let Us Know
</h2>

<p>We built these skills based on what we've learned from years of working with developers building Mapbox applications. But they'll get better as more people use them and give us feedback.</p>

<p>If you build something cool with these skills, or if you find gaps where your AI assistant could use more guidance, let us know. Open an issue on <a href="https://github.com/mapbox/mapbox-agent-skills" rel="noopener noreferrer">GitHub</a> or contribute a new skill if you've discovered patterns we should capture.</p>

<p>We're excited to see what you build.</p>

