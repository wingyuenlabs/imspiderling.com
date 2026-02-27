---
Title: JustHTML.dev — AI-Assisted, Vanilla-First Web Development
Description: 
Author: Brad Simon
Date: 2026-02-27T21:25:20.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/weekend-2026-02-28">DEV Weekend Challenge: Community</a></em></p>

<h2>
  
  
  The Community
</h2>

<p>This project is for developers building small-to-medium web apps who feel forced into React/Vite stacks by default — even when their needs are simple.</p>

<p>A lot of us experimented with tools like Lovable and Bolt.new. They’re powerful. They scaffold React/Vite/Vue apps instantly.</p>

<p>But once AI entered the workflow — Claude, OpenAI, Copilot, Gemini — something changed.</p>

<p>If AI can generate clean HTML, CSS, and JavaScript directly… why are we defaulting to heavy frameworks for everything?</p>

<p>This is for the builders who are thinking:</p>

<p>“Is the complexity actually required — or just assumed?”</p>

<h2>
  
  
  What I Built
</h2>

<p>I built JustHTML.dev — a curated directory of websites built with vanilla HTML, CSS, and JavaScript.</p>

<p>No front-end frameworks.<br>
No client-side runtime layer.<br>
No dependency tree explosion.</p>

<p>It’s not anti-framework. It’s just intentional.</p>

<p>The site:</p>

<p>Automatically analyzes and scores submissions</p>

<p>Displays a public “Last Verified” status</p>

<p>Ranks sites based on implementation quality</p>

<p>Focuses on clarity and simplicity</p>

<p>It’s free. No ads. Just a place to inspire, teach, and showcase vanilla-first builds.</p>

<p>The goal is to prove that with AI assistance, vanilla is not a step backward. In many cases, it’s faster, cleaner, and easier to maintain.</p>
<h2>
  
  
  Demo
</h2>

<p>[<a href="https://justhtml.dev" rel="noopener noreferrer">https://justhtml.dev</a>]</p>
<h2>
  
  
  Code
</h2>

<p>The project is built with a custom C# back-end that renders HTML using simple templating and string replacement.</p>

<p>It intentionally avoids front-end frameworks and Node-based build systems. The final output delivered to the browser is clean, server-rendered HTML with minimal JavaScript.</p>

<p>Main HTML Layout Template:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;html lang="en"&gt;
&lt;head&gt;
    &lt;meta charset="UTF-8"&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
    {{ head }}
    &lt;link rel="stylesheet" href="/css/style.css"&gt;
    {{ page.css }}
&lt;/head&gt;
&lt;body&gt;
    {{&gt; header }}
    &lt;main&gt;
        {{ body }}
    &lt;/main&gt;
    {{&gt; footer }}
    &lt;script src="/js/main.js"&gt;&lt;/script&gt;
    {{ page.js }}
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

</div>



<p>Storing Metadata for the Index.html:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>---
title: "justhtml.dev — You don't need npm for that"
description: "A community for developers who build real things with Vanilla HTML, CSS, and JavaScript. No frontend frameworks. No npm. Just the web platform."
keywords: vanilla html, no npm, no react, web development, html css js
og_title: "justhtml.dev — You don't need npm for that"
og_description: "A community for developers who build real things with Vanilla HTML, CSS, and JavaScript."
og_image: https://justhtml.dev/img/og/og-home.jpg
layout: default
robots: index,follow
---

</code></pre>

</div>



<p>Automatically build sitemap.xml:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>    private async Task WriteSitemapXmlAsync()
    {
        var sb = new StringBuilder();
        sb.AppendLine("""&lt;?xml version="1.0" encoding="UTF-8"?&gt;""");
        sb.AppendLine("""&lt;urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"&gt;""");
        sb.AppendLine();

        // Static content pages (scanned from /content/*.html)
        foreach (var entry in GetStaticPageEntries())
        {
            AppendUrl(sb, $"{_baseUrl}{entry.url}", entry.lastmod, entry.changefreq, entry.priority);
        }

        // Dynamic showcase entries (listed only)
        var showcases = await db.Showcases
            .Where(s =&gt; !s.IsDeleted &amp;&amp; !s.IsHidden &amp;&amp; s.Status == SiteStatus.Listed)
            .Select(s =&gt; new { s.Slug, s.SubmittedAt, s.MetadataFreshedAt })
            .ToListAsync();

        foreach (var s in showcases)
        {
            var lastmod = (s.MetadataFreshedAt.HasValue &amp;&amp; s.MetadataFreshedAt.Value &gt; s.SubmittedAt)
                ? s.MetadataFreshedAt.Value.ToString("yyyy-MM-dd")
                : s.SubmittedAt.ToString("yyyy-MM-dd");

            AppendUrl(sb, $"{_baseUrl}/showcase/{s.Slug}.html", lastmod, "weekly", "0.6");
        }

        sb.AppendLine("&lt;/urlset&gt;");

        var path = Path.Combine(_webRootPath, "sitemap.xml");
        await File.WriteAllTextAsync(path, sb.ToString(), Encoding.UTF8);
        logger.LogInformation("sitemap.xml written ({Count} showcase entries)", showcases.Count);
    }
</code></pre>

</div>



<h2>
  
  
  How I Built It
</h2>

<p>Backend: .NET (minimal server setup)</p>

<p>Rendering: Lightweight HTML templating via string replacement</p>

<p>Frontend: Vanilla HTML, CSS, JavaScript</p>

<p>Deployment: Docker container</p>

<p>Verification system: Automated scanning and scoring logic</p>

<p>The architecture is intentionally simple:</p>

<p>Server builds HTML → Browser renders it → Minimal JS enhances it.</p>

<p>No hydration. No runtime framework. No massive dependency installs.</p>

<p>AI was heavily involved in development — assisting with structure, logic refinement, and iteration — but the architectural decisions were deliberate.</p>

<p>JustHTML.dev is an experiment in architectural restraint.<br>
AI reduces the cost of writing code. It doesn’t justify adding layers.</p>

<p>It was about building the simplest stack that gets the job done.</p>

<p>Coder B</p>

