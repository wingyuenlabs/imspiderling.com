---
Title: Next.js 16 Type Safety: Async PageProps & Typed Routes
Description: 
Author: Bharath Kumar S
Date: 2025-12-22T20:57:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Next.js has quietly become one of the most type-safe full-stack frameworks out there — yet many teams still use only a fraction of what it offers.</p>

<p>In this post, I’ll walk through practical, production-ready type safety features in modern Next.js (App Router), including:</p>

<ul>
<li><strong>Next.js TypeScript plugin</strong></li>
<li><strong>Statically Typed Links</strong></li>
<li><strong>Route-aware type helpers</strong></li>
<li><strong>Auto complete environment variables</strong></li>
<li><strong>Typed <em>NextRequest</em> and <em>NextResponse</em></strong></li>
</ul>

<p>This is not a beginner guide — it focuses on features that actually prevent bugs in real-world applications.</p>



<h2>
  
  
  Next.js TypeScript plugin
</h2>

<p>Next.js ships with a custom TypeScript plugin that enhances type-checking beyond what plain <code>tsc</code> can do.</p>

<p>It understands Next.js concepts such as:</p>

<ul>
<li>File-based routing</li>
<li>Server vs Client Components</li>
<li>Metadata APIs</li>
<li>Layout and page conventions</li>
</ul>

<p>To ensure VS Code uses this plugin:</p>

<ol>
<li>Open the Command Palette <code>(Ctrl / ⌘ + Shift + P)</code>
</li>
<li>Search for TypeScript: Select TypeScript Version</li>
<li>Choose Use Workspace Version</li>
</ol>

<p>This allows VS Code to pick up Next.js–specific type rules automatically via <code>next-env.d.ts</code>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5fc1c14u54jzavpetofh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5fc1c14u54jzavpetofh.png" alt="Search in CMD Palette" width="800" height="214"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjh7h76op3jhhtvc9lio4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjh7h76op3jhhtvc9lio4.png" alt="Select TS Version" width="800" height="183"></a></p>

<p>Once enabled, you’ll get IntelliSense and in-context documentation for Next.js-specific APIs such as metadata and caching options. </p>

<p>Showing available options and in-context documentation<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpp4wf1gqckuzu1f9155q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpp4wf1gqckuzu1f9155q.png" alt="Intellisense" width="800" height="104"></a></p>

<p>Other useful checks provided by the plugin include:</p>

<ul>
<li>Ensuring the <code>"use client"</code> directive is used correctly</li>
<li>Preventing client-only hooks (like useState) from being used in Server Components</li>
<li>Catching invalid exports in <code>page.tsx</code>, <code>layout.tsx</code>, and <code>route.ts</code>
</li>
</ul>



<h2>
  
  
  Statically Typed Links
</h2>

<p>Next.js can generate statically typed routes to prevent typos and invalid navigation.</p>

<p>Enable typed routes in <code>next.config.ts</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typedRoutes: true,
}

export default nextConfig
</code></pre>

</div>



<p>Next.js will generate route definitions inside <code>.next/types</code>, which TypeScript uses to validate links and navigation.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fth0wy2v2who5v2bmjsnm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fth0wy2v2who5v2bmjsnm.png" alt="Route Types" width="800" height="98"></a></p>

<p>This ensures:</p>

<ul>
<li>Invalid routes are caught at compile time</li>
<li>Route refactors are safer</li>
<li>IntelliSense suggests only valid paths</li>
</ul>



<h2>
  
  
  Route-Aware Type Helpers
</h2>

<p>Next.js generates global route-aware helpers for App Router types. These are available without imports and are generated during <code>next dev</code>, <code>next build</code>, or via <code>next typegen</code>.</p>

<p>Available helpers include:</p>

<ul>
<li>PageProps</li>
<li>LayoutProps</li>
<li>RouteContext</li>
</ul>
<h3>
  
  
  Without PageProps
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export default function Details({
  params,
  searchParams,
}: {
  params: { slotId: string };
  searchParams: { name?: string };
}) {
  const { slotId } = params;
  const { name } = searchParams;

  return (
    &lt;div&gt;
      Slot: {slotId} &lt;br /&gt;
      Name: {name}
    &lt;/div&gt;
  );
}
</code></pre>

</div>


<p>This approach is explicit and works well for smaller components.</p>



<h3>
  
  
  With PageProps
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export default async function Details(props: PageProps&lt;"/details/[slotId]"&gt;) {
  const { slotId } = await props.params;
  const { name } = await props.searchParams;

  return (
    &lt;div&gt;
      Slot: {slotId} &lt;br /&gt;
      Name: {name}
    &lt;/div&gt;
  );
}
</code></pre>

</div>


<p>This version is more concise and automatically stays in sync with the route definition.</p>



<h3>
  
  
  Without LayoutProps
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import type { ReactNode } from "react";

export default function RootLayout({
  children,
}: {
  children: ReactNode;
}) {
  return (
    &lt;html lang="en"&gt;
      &lt;body&gt;{children}&lt;/body&gt;
    &lt;/html&gt;
  );
}
</code></pre>

</div>




<h3>
  
  
  With LayoutProps
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export default function RootLayout(props: LayoutProps&lt;"/"&gt;) {
  return (
    &lt;html lang="en"&gt;
      &lt;body&gt;
        {props.children}
      &lt;/body&gt;
    &lt;/html&gt;
  );
}
</code></pre>

</div>




<h3>
  
  
  Without RouteContext
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export async function GET(
  _request: NextRequest,
  { params }: { params: Promise&lt;{ id: string }&gt; }
) {
  const { id } = await params;
</code></pre>

</div>




<h3>
  
  
  With RouteContext
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export async function GET(
  _request: NextRequest,
  ctx: RouteContext&lt;"/api/[id]"&gt; // you will get intellisense
) {
  const { id } = await ctx.params;
</code></pre>

</div>


<p>Using RouteContext gives you full IntelliSense for route params and avoids manual typing.</p>



<h2>
  
  
  Auto complete environment variables
</h2>

<p>Enable it in next.config.ts<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>![next env config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9ozpitstpxw6dxf5g7o7.png)
</code></pre>

</div>



<p>Now you will get intellisense support </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyx0wnyfww7mp0t4nrbcc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyx0wnyfww7mp0t4nrbcc.png" alt="Typed Env" width="788" height="188"></a></p>



<h2>
  
  
  Typed NextRequest and NextResponse
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import { NextRequest, NextResponse } from "next/server";

export async function GET(
  _request: NextRequest
) {

  return NextResponse.json(
    { msg: 'Hello!!! })
}
</code></pre>

</div>



<p>NextRequest extends the Web Request API with:</p>

<ul>
<li>Typed cookies</li>
<li>Parsed URLs via nextUrl</li>
<li>Middleware-friendly helpers</li>
</ul>

<p>NextResponse extends the Web Response API with:</p>

<ul>
<li>json()</li>
<li>redirect()</li>
<li>Cookie helpers</li>
<li>Edge-friendly APIs</li>
</ul>



<h2>
  
  
  Final Thoughts
</h2>

<p>Next.js type safety isn’t about writing more types —<br>
it’s about letting the framework encode architectural rules directly into your editor.</p>

<p>When used well, TypeScript becomes a guardrail, not a burden,<br>
and IntelliSense becomes an extension of your system design.</p>




<p>Reference - <a href="https://nextjs.org/docs/app/api-reference/config/typescript" rel="noopener noreferrer">Next.js - TypeScript</a></p>



<h2>
  
  
  Connect with Me
</h2>

<p>If you enjoyed this post or want to follow my work:</p>

<ul>
<li>
<strong>LinkedIn:</strong> <a href="https://www.linkedin.com/in/bharath-kumar-s-a4339aa7/" rel="noopener noreferrer">Bharath Kumar S</a>
</li>
<li>
<strong>GitHub:</strong> <a href="https://github.com/Bharath-Kumar-S" rel="noopener noreferrer">Bharath Kumar</a>
</li>
</ul>

<p>Feel free to <strong>connect, star my projects, or reach out</strong> — I love discussing Next.js, TypeScript, and modern frontend best practices!</p>

