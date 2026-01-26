---
Title: Finally, a Modern CMS for Developers: Introducing NextBlock (Open Source)
Description: 
Author: NextBlock
Date: 2026-01-26T20:51:07.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  How we are breaking the cycle of bloated legacy platforms and complex headless configs with Next.js 16 and Supabase.
</h2>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/WUNLtZWZGL8">
</iframe>
</p>

<p>Every web developer knows the pain of choosing a Content Management System (CMS). It often feels like you are stuck between a rock and a hard place.</p>

<p>On one side, you have the “old guard” (like WordPress). It’s familiar, sure. But it is often bloated, slow, and — thanks to a reliance on endless plugins — a potential security minefield. On the other side, you have modern headless systems. They offer raw speed, but they can be a headache to set up, a nightmare for content editors to use, and often bury you in configuration files.</p>

<h4>
  
  
  We decided to break that cycle.
</h4>

<p>Enter NextBlock, an open-source, developer-first CMS built to be the refreshing alternative we’ve been waiting for. In this article, I’m going to break down exactly what we’ve built so far, our tech stack, and our vision for a sustainable open-source ecosystem.</p>

<h3>
  
  
  The Tech Stack
</h3>

<p>At its core, NextBlock is built on a modern foundation designed for the future of the web:</p>

<p>• Frontend: Next.js 16 for blazing-fast performance and Server Components.</p>

<p>• Backend: Supabase for handling data, auth, and storage.</p>

<p>• Styling: Tailwind CSS for a lean, utility-first design system.</p>

<p>But the tech is just the vehicle. The mission of NextBlock boils down to three core pillars: Speed, Experience, and Extensibility.</p>

<p>Pillar 1: Built for Speed</p>

<p>When we say “built for speed,” we aren’t talking about vague marketing promises. We are talking about concrete engineering wins that are baked into the core platform.</p>

<ol>
<li><p>Edge Caching &amp; ISR: We’ve set up caching at the edge, utilizing Next.js Incremental Static Regeneration (ISR). This ensures your site feels instantaneous regardless of where your users are located.</p></li>
<li><p>Critical Code Loading: We are obsessive about shipping only the code that is absolutely necessary. This includes using Tailwind CSS to purge unused styles, making the final production build incredibly lean.</p></li>
<li><p>Next-Gen Image Optimization: Images are usually the #1 culprit for slow websites. NextBlock solves this by baking in the Next.js  component, enabling modern formats like AVIF (which are significantly smaller than JPEGs), and using blur-up placeholders so pages feel snappy even while assets load.</p></li>
</ol>

<h3>
  
  
  Pillar 2: A Superior Experience (For Everyone)
</h3>

<p>Speed matters, but what is the point of a fast site if it is a pain to manage? We focused heavily on the experience for both the Content Creator and the Developer.</p>

<p>For the Content Creator: We wanted an editing experience that feels modern — not like a form from 2005.</p>

<p>• Notion-Style Editor: We upgraded the block editor (powered by Tiptap) to be intuitive and drag-and-drop friendly.</p>

<p>• Organized Media: No more chaotic file dumps. The media library supports folders and tagging.</p>

<p>• Revision History: We added one-click revision history, a game-changer for content teams who need safety when editing live pages.</p>

<p>For the Developer: We moved the project structure into an Nx Monorepo. This makes it significantly easier to manage dependencies and scale the codebase as you add new features. We also built a CLI tool (create-nextblock) that spins up a new project in seconds, and a documented Block SDK that allows you to build custom components—unlocking the real power of the platform.</p>

<h3>
  
  
  What’s Next: Phase 3 and the “Open Core” Model
</h3>

<p>We have built a fast, user-friendly foundation. Now, we are moving into Phase 3, where the vision expands from “just a tool” to a full ecosystem.</p>

<p>The focus right now is the development of a Premium E-commerce Module. This will be a full-blown commerce solution built to plug directly into NextBlock.</p>

<p>“Wait, premium? I thought this was Open Source?” This is a critical part of our vision for sustainability. We are adopting an Open Core model:</p>

<ol>
<li><p>The Core CMS: Always free, always open-source (AGPL license). This is the foundation.</p></li>
<li><p>Premium Modules: High-value, complex features (like E-commerce or Enterprise SSO) will be offered as paid add-ons.</p></li>
<li><p>The Marketplace: This fuels a future community marketplace where developers can share and sell their own blocks and themes.</p></li>
</ol>

<h2>
  
  
  Join the Ecosystem
</h2>

<p>As we say on our site: “More than a CMS, an ecosystem”. The goal isn’t just to ship code; it’s to build a platform where we can create amazing web experiences together.</p>

<p>NextBlock is being built in the open. If the idea of a faster, better, developer-first CMS sounds exciting to you, we want you involved.</p>

<p>• Star the Repo: [<a href="https://github.com/nextblock-cms/nextblock" rel="noopener noreferrer">https://github.com/nextblock-cms/nextblock</a>]</p>

<p>• Join the Community: [<a href="https://github.com/nextblock-cms/nextblock/discussions" rel="noopener noreferrer">https://github.com/nextblock-cms/nextblock/discussions</a>]</p>

<p>The question isn’t just what’s next for the web — it’s whether you are ready to help build it.</p>

