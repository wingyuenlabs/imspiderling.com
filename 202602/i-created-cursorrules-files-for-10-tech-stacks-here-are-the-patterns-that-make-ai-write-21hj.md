---
Title: I Created .cursorrules Files for 10 Tech Stacks. Here Are the Patterns That Make AI Write Production Code.
Description: 
Author: Matthew Hou
Date: 2026-02-19T22:12:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>After using Cursor daily for 6 months, I realized the difference between "AI-generated demo code" and "AI-generated production code" comes down to one file: <code>.cursorrules</code>.</p>

<p>This is the configuration file that tells Cursor HOW to code in your project. Without it, Cursor defaults to generic patterns. With a good one, it codes like a senior developer who's read your entire codebase.</p>

<p>Here's what I learned creating rules files for 10 different tech stacks.</p>

<h2>
  
  
  What .cursorrules Actually Does
</h2>

<p>When you put a <code>.cursorrules</code> file in your project root, Cursor reads it as system-level context for every interaction. It's like giving the AI a coding style guide, architecture overview, and list of conventions — all in one file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Project Root/
├── .cursorrules     ← Cursor reads this automatically
├── src/
├── tests/
└── package.json
</code></pre>

</div>



<h2>
  
  
  The 5 Sections Every .cursorrules File Needs
</h2>

<h3>
  
  
  1. Identity
</h3>

<p>Tell the AI what it is — an expert in YOUR stack.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>You are an expert in TypeScript, Next.js 14 App Router, 
Supabase, and Tailwind CSS. You write production-ready 
code that handles errors, edge cases, and concurrent access.
</code></pre>

</div>



<h3>
  
  
  2. Code Style
</h3>

<p>The conventions your team follows.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>## Code Style
- Use functional components with TypeScript interfaces
- Prefer named exports over default exports  
- Use early returns for guard clauses
- const arrow functions for components
</code></pre>

</div>



<h3>
  
  
  3. Architecture
</h3>

<p>How your project is structured.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>## Architecture
- App Router: use Server Components by default, 
  'use client' only when needed
- Data fetching in Server Components with async/await
- Server Actions for mutations with 'use server'
- Zod for all input validation
</code></pre>

</div>



<h3>
  
  
  4. Anti-Patterns
</h3>

<p>What the AI should NEVER do. This is the most important section.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>## Never
- Never use 'any' type
- Never catch errors silently (always log + handle)
- Never use useEffect for data fetching
- Never put secrets in client components
- Never use inline styles (use Tailwind)
</code></pre>

</div>



<h3>
  
  
  5. Patterns
</h3>

<p>Show specific patterns by example.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>## Error Handling Pattern
Every async function should follow:
try {
  // happy path
} catch (err) {
  if (err instanceof SpecificError) {
    // handle specifically
  }
  logger.error({ err, context }, 'Description');
  throw err; // or return error response
}
</code></pre>

</div>



<h2>
  
  
  Real-World Example: Next.js + Supabase
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># .cursorrules

You are an expert in TypeScript, Next.js 14 (App Router), 
Supabase, Tailwind CSS, and Shadcn UI.

## Code Style
- TypeScript strict mode, no `any`
- Functional components with `const Component = () =&gt; {}`
- Named exports only
- Destructure props in function parameters
- Use early returns for guard clauses

## Next.js Conventions
- Server Components by default
- 'use client' only for: event handlers, hooks, browser APIs
- Use `generateMetadata` for SEO
- loading.tsx for suspense, error.tsx for error boundaries
- Server Actions for mutations with Zod validation

## Supabase Patterns
- Server-side: use `createServerClient()` from `@supabase/ssr`
- Client-side: use `createBrowserClient()` 
- Always handle Supabase errors: `const { data, error } = await...`
- Use Row Level Security — never trust the client
- Prefer `.single()` when expecting one row

## Styling
- Tailwind only — no CSS modules, no inline styles
- Shadcn components for UI elements
- Mobile-first: sm:, md:, lg:
- Use `cn()` utility for conditional classes

## Never
- Never use `useEffect` for data fetching
- Never use `any` or type assertions (`as`)
- Never put Supabase service key in client code
- Never skip error handling on Supabase queries
- Never use default exports
</code></pre>

</div>



<h2>
  
  
  The Difference in Practice
</h2>

<p><strong>Without .cursorrules:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Cursor generates:</span>
<span class="k">export</span> <span class="k">default</span> <span class="kd">function</span> <span class="nf">UserProfile</span><span class="p">({</span> <span class="nx">userId</span> <span class="p">})</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">[</span><span class="nx">user</span><span class="p">,</span> <span class="nx">setUser</span><span class="p">]</span> <span class="o">=</span> <span class="nf">useState</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>

  <span class="nf">useEffect</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">fetch</span><span class="p">(</span><span class="s2">`/api/users/</span><span class="p">${</span><span class="nx">userId</span><span class="p">}</span><span class="s2">`</span><span class="p">)</span>
      <span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">r</span> <span class="o">=&gt;</span> <span class="nx">r</span><span class="p">.</span><span class="nf">json</span><span class="p">())</span>
      <span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="nx">setUser</span><span class="p">);</span>
  <span class="p">},</span> <span class="p">[</span><span class="nx">userId</span><span class="p">]);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">user</span><span class="p">)</span> <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="nx">Loading</span><span class="p">...</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span>  <span class="k">return</span> <span class="o">&lt;</span><span class="nx">div</span> <span class="nx">style</span><span class="o">=</span><span class="p">{{</span><span class="na">padding</span><span class="p">:</span> <span class="mi">20</span><span class="p">}}</span><span class="o">&gt;</span><span class="p">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/div&gt;</span><span class="err">;
</span><span class="p">}</span>
</code></pre>

</div>



<p>Problems: default export, <code>any</code> types, useEffect for data fetching, no error handling, inline styles, client-side when it should be server.</p>

<p><strong>With .cursorrules:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Cursor generates:</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">createServerClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@/lib/supabase/server</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">notFound</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">next/navigation</span><span class="dl">'</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">UserProfileProps</span> <span class="p">{</span>
  <span class="nl">userId</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>

<span class="k">export</span> <span class="kd">const</span> <span class="nx">UserProfile</span> <span class="o">=</span> <span class="k">async </span><span class="p">({</span> <span class="nx">userId</span> <span class="p">}:</span> <span class="nx">UserProfileProps</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">supabase</span> <span class="o">=</span> <span class="nf">createServerClient</span><span class="p">();</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">user</span><span class="p">,</span> <span class="nx">error</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">supabase</span>
    <span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">users</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">select</span><span class="p">(</span><span class="dl">'</span><span class="s1">id, name, email, avatar_url</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">eq</span><span class="p">(</span><span class="dl">'</span><span class="s1">id</span><span class="dl">'</span><span class="p">,</span> <span class="nx">userId</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">single</span><span class="p">();</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">error</span> <span class="o">||</span> <span class="o">!</span><span class="nx">user</span><span class="p">)</span> <span class="k">return</span> <span class="nf">notFound</span><span class="p">();</span>

  <span class="k">return </span><span class="p">(</span>
    <span class="o">&lt;</span><span class="nx">div</span> <span class="nx">className</span><span class="o">=</span><span class="dl">"</span><span class="s2">p-5</span><span class="dl">"</span><span class="o">&gt;</span>
      <span class="o">&lt;</span><span class="nx">h1</span> <span class="nx">className</span><span class="o">=</span><span class="dl">"</span><span class="s2">text-xl font-bold</span><span class="dl">"</span><span class="o">&gt;</span><span class="p">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">name</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/h1</span><span class="err">&gt;
</span>      <span class="o">&lt;</span><span class="nx">p</span> <span class="nx">className</span><span class="o">=</span><span class="dl">"</span><span class="s2">text-muted-foreground</span><span class="dl">"</span><span class="o">&gt;</span><span class="p">{</span><span class="nx">user</span><span class="p">.</span><span class="nx">email</span><span class="p">}</span><span class="o">&lt;</span><span class="sr">/p</span><span class="err">&gt;
</span>    <span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;
</span>  <span class="p">);</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Server Component, named export, typed props, Supabase with error handling, Tailwind classes. Production-ready on the first generation.</p>

<h2>
  
  
  What I Learned From 10 Stacks
</h2>

<h3>
  
  
  Pattern 1: Anti-patterns matter more than patterns
</h3>

<p>The "Never" section is 3x more impactful than the "Do" section. AI models are trained on millions of code examples — many of which contain bad practices. Telling the AI what NOT to do steers it away from the most common garbage.</p>

<h3>
  
  
  Pattern 2: Be specific about your version
</h3>

<p>"Use Next.js" generates Next.js 12 patterns. "Use Next.js 14 App Router with Server Components" generates modern code. AI defaults to the most common patterns, which are often outdated.</p>

<h3>
  
  
  Pattern 3: Include your error handling convention
</h3>

<p>Every team handles errors differently. Specify yours explicitly, or you'll get a mix of try/catch, error returns, and silent failures.</p>

<h3>
  
  
  Pattern 4: Reference your existing code
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>## Reference Files
See src/lib/supabase/server.ts for the server client pattern.
See src/components/ui/ for Shadcn component usage.
</code></pre>

</div>



<p>This tells Cursor to look at your actual code for context, not just follow generic patterns.</p>

<h3>
  
  
  Pattern 5: Update regularly
</h3>

<p>Your .cursorrules should evolve with your project. Added a new convention? Update the file. Found a pattern Cursor keeps getting wrong? Add it to the "Never" section.</p>

<h2>
  
  
  Getting Started
</h2>

<ol>
<li>Create <code>.cursorrules</code> in your project root</li>
<li>Start with the 5 sections above</li>
<li>After your first Cursor session, look at what it got wrong</li>
<li>Add those mistakes to the "Never" section</li>
<li>Repeat. Your rules file gets better every week.</li>
</ol>




<p>💡 <em>I created production-ready .cursorrules files for 10 tech stacks (Next.js, FastAPI, TypeScript/Node, Rust, Go, Flutter, Vue/Nuxt, Swift, Django, React Native). Grab them all in my <a href="https://updatewave.gumroad.com" rel="noopener noreferrer">AI Coding Prompt Templates Pack</a> on Gumroad — $9, includes 50 prompts and workflow templates too.</em></p>

<p><em>What's in your .cursorrules? I'd love to see what conventions other teams enforce. Drop yours in the comments.</em></p>

