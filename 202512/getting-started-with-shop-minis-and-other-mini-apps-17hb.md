---
Title: Getting started with Shop Minis (and other mini apps!)
Description: 
Author: Gadget
Date: 2025-12-04T21:54:12.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Learn about the Shopify Shop Mini ecosystem and how to get started building Shop Minis with custom backends.
</h2>

<p><strong>TLDR:</strong> Shop Minis are custom, cross-merchant apps that enhance the buyer experience in Shopify’s Shop app. Gadget now has a template that handles authentication, session management, and data multitenancy for Shop Mini apps.</p>

<p>Shopify recently announced Shop Mini apps, providing developers a way to provide shoppers custom experiences in Shopify’s Shop mobile app. I’m going to talk a bit about what mini apps are, what is unique about Shop Mini apps (compared to “traditional” admin-embedded Shopify apps), and how you can start building and submitting your Shop Minis.</p>

<p>If you need a custom backend or database for your Shop Mini, check out the <a href="https://app.gadget.dev/auth/fork?domain=shop-mini-template.gadget.app&amp;_gl=1*z0127b*_gcl_au*MTQyNzc3MzA4Ny4xNzU4ODEzNDI4" rel="noopener noreferrer">Gadget template</a> and follow the README to start making authenticated requests from your mini to Gadget.</p>

<p>Prefer a video?</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/gAJzSVnuivo?start=2">
</iframe>
</p>

<h2>
  
  
  What are mini apps?
</h2>

<p>Mini apps aren’t unique to the Shop app. In fact, on November 13, Apple announced their own <a href="https://developer.apple.com/news/?id=xcz1s7cz" rel="noopener noreferrer">Mini Apps Partner Program</a>, accompanied by mini app billing APIs (which are notably absent for Shop Mini apps!).</p>

<p>Mini apps are small applications built with web technologies like HTML, JavaScript, and CSS, that run inside a host application rather than inside a web browser. A host app provides some combination of an outer shell, navigation, identity, APIs, and native UI elements, and the mini app delivers focused functionality inside that wrapper.</p>

<p>It’s like if you were eating a sandwich, and inside that sandwich was another, smaller sandwich. The outer sandwich provides structure and a foundation, something you can hold. The inner sandwich: a delightful twist of new flavors. There is probably a food truck in Portland that sells these already.</p>

<p>A better example: building games for the Discord mobile app:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fblfvdnignqqsk2bm4ib6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fblfvdnignqqsk2bm4ib6.png" alt="Mini apps can be built inside the Discord mobile app!" width="584" height="854"></a><br>
Mini apps can be built inside the Discord mobile app!</p>

<p>You can think of it like embedding a web-powered widget inside a native container. The host controls high-level concerns; the mini concentrates on a specific user interaction. With mini apps for mobile, you can bypass building, deploying, or managing a full mobile app and instead deliver reach through the existing host app’s user base.</p>

<p>The recently announced ChatGPT App SDK is another example of a mini app. The Apps SDK enables devs to build custom functionality inside ChatGPT (for the 800 million monthly users) and provides an API and design system you can use to extend ChatGPT’s base functionality.</p>

<h2>
  
  
  What are Shop Minis?
</h2>

<p>Shop Minis are just mini apps for Shopify’s Shop mobile app and its user base. (The Shop app is a central hub for all merchants selling on Shopify. Instead of a shopper having to find and visit individual shops, they can discover ALL Shopify products in a single place.)</p>

<p>Instead of building tools for merchants or storefronts like you would when building traditional embedded Shopify apps, a Shop Mini delivers enhancements to the buyer experience across any merchant using Shopify.</p>

<p>Use cases for Shop Minis include things like: custom style pickers, color or complement-color finders, enhanced product discovery and recommendations, or other buyer-facing UI features. In some ways, you can think of a Shop mini as analogous to a theme extension, but instead of modifying a storefront, you’re extending the Shop app itself for shoppers.</p>

<p>Shop Mini app examples:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhqkeq00zvigrr2yvpyxn.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhqkeq00zvigrr2yvpyxn.png" alt="Shop Mini app examples" width="584" height="854"></a></p>

<p>This is powerful because it gives you reach across all Shopify merchants, without requiring each merchant to install or configure the mini individually. Once your mini is approved, any buyer using Shop can use your mini to find product recommendations.</p>

<h2>
  
  
  How to build Shop Minis
</h2>

<p>At the moment, the Shop Mini functionality is part of a pilot program, not yet broadly available. However, Shopify has established a framework for early partners to develop, submit, and ship minis.</p>

<p>If you are already familiar with traditional web tools and React, building a functioning Mini won’t be difficult (once you download XCode and/or Android Studio.) Shopify provides a well documented API for fetching cross-merchant products and, for apps with custom backends, has provided an <a href="https://github.com/Shopify/shop-minis/blob/main/supabase/README.md" rel="noopener noreferrer">auth example using Supabase Edge Functions</a>.</p>

<p>A sample Shop Mini app - tutorial coming soon:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fshaxxktepxhuiorf80la.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fshaxxktepxhuiorf80la.png" alt="A sample Shop Mini app - tutorial coming soon!" width="800" height="503"></a></p>

<p>Here is what you need to know if you decide to build a Shop Mini app:</p>

<ul>
<li>Minis are built in React using a project generated by a command-line interface, via a <code>@shopify/shop-minis</code> CLI.</li>
<li>Mobile-first: Even though you’re building in React, you need to build and test your mini using Android Studio or XCode. If you try to build on a web browser you will only ever get mock data. <em>This is a trap!</em> Just use the mobile simulators from the start!</li>
<li>Styling &amp; UI ecosystem: The mini runs with a provided component library + styling support (Tailwind CSS, a set of SDK components, icons from Lucide), to ensure a consistent look and feel inside the Shop app. Using emojis instead of Lucide icons? <em>Instant rejection.</em>
</li>
<li>Limited dependencies: Shopify enforces strict dependency constraints. Using an npm package not on the list?  <em>Instant rejection.</em>
</li>
<li>Data fetching &amp; hooks: Shopify provides hooks to fetch relevant data like user info, product data, and other context. These hooks are the intended way to access Shopify data, especially merchant and product data. Reading product data from a custom endpoint? <em>Instant rejection.</em>
</li>
<li>Backend + auth architecture:

<ul>
<li>You can build a custom backend to support your mini, which requires you to set up JWT authentication on your server.</li>
<li>Before signing a JWT, you need to call the Mini Admin API with the provided Mini API key to verify that the auth request is legitimate.</li>
<li>Session and user management, and data multi-tenancy must be handled manually.</li>
<li>The mini must request the openid scope and trusted domains need to be registered in <code>manifest.json</code>.</li>
</ul>


</li>

</ul>

<p>The auth flow for custom Shop Mini backends:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F07aqqr44oqhwsk1fo3hi.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F07aqqr44oqhwsk1fo3hi.png" alt="The auth flow for custom Shop Mini backends" width="800" height="887"></a></p>

<ul>
<li>Analytics &amp; metrics: The mini framework includes analytics support so you can monitor how often your mini is used and measure conversions, such as how often buyers use your mini to go from browsing to purchase. This helps you understand impact and performance.</li>
</ul>

<p>The <a href="https://shopify.dev/docs/api/shop-minis" rel="noopener noreferrer">Shop Mini docs</a> are great and will help you get started. There are <a href="https://shopify.dev/docs/api/shop-minis/guidelines" rel="noopener noreferrer">strict guidelines</a> that you need to follow when building Shop Minis, including a hefty list of prohibited practices. Familiarize yourself with these items before you start building!</p>

<h2>
  
  
  Publishing your Shop Mini
</h2>

<p>The Shop Mini CLI has a <code>submit</code> command you can use to send your mini to Shopify for review. </p>

<p>As part of the submission, you must supply your source code and dependencies, which will be reviewed by the Shopify team. You also need to record a video demonstrating the full end-to-end flow: what happens from the buyer opening the Shop app, to launching/using the mini, to the end result (and benefit!) for the buyer.</p>

<p>I expect these reviews to be incredibly thorough. Mini apps reflect on Shopify and their Shop app, and poor-quality minis will discourage buyers from using Shop in the first place. The review will check for compliance: security, performance, bundle size (Shopify is targeting &lt;3 seconds for load and &lt;5MB bundle size), correct behavior, and user experience, and the team will be making sure you don’t violate any of the Shop Mini guidelines.</p>

<p>There’s a financial incentive for early builders: Shopify is offering a bounty of $5,000–$10,000 if the mini is submitted, approved, and launched by December 20. And if your mini makes use of AI, there is also reimbursement available for AI usage up to $500 per partner per month.</p>

<p>Because the Shop Mini ecosystem is still relatively new, documentation and tooling are evolving. Treat this as an opportunity to influence the ecosystem. If you build now, you’ll learn early and be able to provide feedback to Shopify that shapes how Shop Minis work going forward.</p>

<h2>
  
  
  Summary
</h2>

<p>Shop Minis are a new mechanism for embedding buyer-facing functionality inside the Shop mobile app. Built in React and bound by strict constraints (styling, dependencies, performance, security), they allow developers to ship cross-merchant experiences at scale. Devs get access to the massive user base already using the Shop app. And with a custom backend and database, you have flexibility to deliver custom features, data, and behaviors to these users.</p>

<p>If you want to build features that serve buyers directly without asking merchants to install or configure anything, minis are worth exploring.</p>

<p>Start building Shop Minis today, with the <a href="https://app.gadget.dev/auth/fork?domain=shop-mini-template.gadget.app&amp;_gl=1*fosmn8*_gcl_au*MTQyNzc3MzA4Ny4xNzU4ODEzNDI4" rel="noopener noreferrer">Gadget template</a>. Setup instructions are in the README.</p>

