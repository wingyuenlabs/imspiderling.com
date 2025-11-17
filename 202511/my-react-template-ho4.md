---
Title: My React template
Description: 
Author: Thanos Korakas
Date: 2025-11-17T22:07:22.000Z
Robots: noindex,nofollow
Template: index
---
<p>You can find the original post on my <a href="https://tkorakas.dev/blog/react-template/" rel="noopener noreferrer">blog</a></p>

<h2>
  
  
  The Problem
</h2>

<p>Lately I've found myself wanting to work on some side project ideas. Every time I had to implement the same stuff again and again: basic library configuration, routing, layout, forms, authentication. Most of the time I would start a project using Vite and copy-paste parts from other apps I'd worked on.</p>

<p>That alone could take a couple of hours or days without making any progress on my idea.</p>

<h2>
  
  
  My Tech Stack
</h2>

<p>The things I usually need are simple. I like working with Tanstack Query for handling HTTP requests and caching. I implement authentication using sessions where an HTTP-only cookie is set by the backend, and on the frontend I make a request against a <code>/me</code> or <code>/user</code> endpoint to check if the user has a session. I set up some interceptors to log the user out on 401 requests. Finally, I like to wire basic input fields so I can build forms without having to care about error handling or wiring the field every time.</p>

<p>Here is a more detailed list of the libraries that I use:</p>

<ul>
<li>React Router</li>
<li>Tanstack Query</li>
<li>React Hook Form</li>
<li>Zod</li>
<li>Ky (previously Axios)</li>
<li>Chakra UI</li>
</ul>

<h2>
  
  
  The Solution
</h2>

<p>It was time for my side project to be a React template to help me build my next ideas faster. I published the code on <a href="https://github.com/tkorakas/react-template" rel="noopener noreferrer">GitHub</a>.</p>

<h2>
  
  
  Project Architecture
</h2>

<p>In my project, I prefer separating files by logic/context rather than file type. So you won't find components and hooks folders in the project.</p>

<p>It follows a DDD-like approach where each feature goes into the features folder. Features are composed of a template file and a view model where I wire stuff like forms or HTTP requests.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>src/
├── features/
│   ├── login/
│   │   ├── index.tsx
│   │   └── use-handler.ts
│   └── register/
│       ├── index.tsx
│       └── use-hander.ts
├── data-access/
│   ├── api.ts
│   └── users.schema.ts
└── commong/
    └── auth/
</code></pre>

</div>



<p>I keep features agnostic of storage or external dependencies by putting them in the data-access folder. So data-access describes and validates API requests using Ky and Zod.</p>

<p>Then in the view model, I use Tanstack Query to call the API and get a response.</p>

<p>In the past, I used to create custom hooks inside data-access so I didn't expose Tanstack Query into my features. Although this helped a lot with testing and keeping features agnostic of implementation details, it required a lot of boilerplate and honestly, I depend a lot on Tanstack Query for many things, so I decided to simplify the setup.</p>

<h2>
  
  
  Mock Server
</h2>

<p>I also created a mock server to help me prototype fast without touching the backend yet. The mock server supports authentication using an HTTP-only cookie, MFA by printing an OTP to the console, and OAuth2 providers for login—currently it implements GitHub.</p>

<p>The mock server can read JSON files from the filesystem and use them as a database. It also supports all CRUD operations. That way I can build CRUD APIs fast to verify my ideas. Usually it helps me a lot to build a POC of the UI and improve it by using it, instead of spending more time designing and thinking about how it should work.</p>

<h2>
  
  
  Future Plans
</h2>

<p>I plan to continue working on the template by adding support for more OAuth providers like Google and enhancing security by implementing XSRF and other best practices.</p>

<p>Further improvements include:</p>

<ul>
<li>Support for HOTP/TOTP</li>
<li>More features regarding user management like profile management and password changes</li>
<li>Roles &amp; permissions, perhaps payments</li>
<li>Helpers for handling tables by integrating Tanstack Table and custom helpers all wired with Chakra components</li>
</ul>

<p>If you have any issues with the template, please create an <a href="https://github.com/tkorakas/react-template/issues" rel="noopener noreferrer">issue</a> or share any other ideas.</p>

<p>Thanks for reading.</p>

