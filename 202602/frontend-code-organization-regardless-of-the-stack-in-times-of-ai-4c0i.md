---
Title: Frontend Code organization, regardless of the stack in times of AI 🤖
Description: 
Author: Angela Pat.
Date: 2026-02-03T20:28:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Let's talk about Frontend architecture. We all know that Frontend has become complex with the days and why is it so? <br>
Front-end is where all user requirements, user enhancements, use of the Business domain, handling of errors, optimization, and so on. Our frontend architecture is responsible for the cohesion of the new and old codebase.</p>
<h3>
  
  
  In the time of AI agents 🤖
</h3>

<p>Even now, with the rise of AI agent coding tools, <strong>clean and well organized code remains critical</strong>. <br>
Without clear instructions, AI can easily overcomplicate solutions. AI agents understand domain logic through code structure. The clearer and more consistent patterns you have, the less context the Agent holds, because the agent will follow a predictable organization.</p>

<p>For that reason, my rule of thumb is simple: <strong>Follow a modular approach</strong> to code organization</p>
<h3>
  
  
  What is a modular approach?
</h3>

<p>A modular approach is the practice of breaking down a solution or user requirements into smaller units, where each unit has a single responsibility.</p>

<p>Following this approach helps maintain a solution over the long term and makes it easier to add new features, whether they are simple or complex.</p>
<h3>
  
  
  With so many stacks out there, why not use the basic organization offered by a library or framework?
</h3>

<p>Of course you can! A framework or library often provides a playground or a basic project structure to get you started. However, as tasks grow bigger and features begin to interact with one another, it becomes our responsibility to break down big user requirements or concepts into self-contained units.</p>
<h3>
  
  
  A use case:
</h3>

<p>Let's say your Admin panel website, needs a new feature an <strong>Employees overview</strong> page with the following use cases:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Page: Employees Overview***
 - Filter (search, role, age range)
 - List employees
 - Open modal for Editing a new employee 
 - Open a modal for adding a new employee
</code></pre>

</div>



<p>Now, we can design the architecture by breaking the solution into smaller building blocks.</p>

<p><strong>For example:</strong> </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyks2ewu29hs89u5rry5m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyks2ewu29hs89u5rry5m.png" alt="Rough idea structured in blocks" width="800" height="1264"></a></p>

<p>With our rough idea in place, we can start organizing our code using a modular approach. We take a big feature and split it into smaller and independent blocks called modules.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>project 
└─── common/            
│       └─── components/
│               └──── Modal
│       └─── hooks/ 
│               └─── useUserLanguage.ts  
│       └─── models/ 
│               └─── user.ts 
└─── core/
│       └─── authentication/
│       └─── i18n/
│       └─── translations/
│       └─── router/
│       └─── api/
│       └─── store/
└───modules/
│   └───employees-overview/
|       │─── EmployeesOverview.tsx (Main component)
│       └─── components/
│             └────── employee-filter/
│             └────── employee-list/
│             └────── employee-add/
│             └────── employee-edit/
│                        └────── hooks/
│                                 └───── useFetchUser.ts
│                                 └───── useUpdateUser.ts
│                        └────── components/
│                        └────── helpers/ 
│                        └────── models/  
│                        └────── EmployeeEdit.tsx
└───lib/
│   └───utils

</code></pre>

</div>



<p>Let me explain you shortly about some of the important folders and what functionality they cover</p>

<ul>
<li>
<strong>common</strong> <em>Also called <strong>shared</strong></em>: Contains code that can be reused across modules (features). You can organize them by <strong>components</strong>, <strong>hooks</strong>, <strong>helpers</strong>, etc.

<ul>
<li>
<strong>helpers</strong> <em>Also called <strong>functions</strong></em>: Pure functions that performance specific tasks.</li>
<li>
<strong>models</strong>: Represent your entities. It's recommended that your entities match the backend for consistency and a single source of truth, with slight differences when needed, for example, dates (the backend uses UTC, while the frontend uses a Date object).</li>
</ul>


</li>

<li>

<strong>modules</strong>: Feature-based code aligned with your business goals.</li>

<li><p><strong>core</strong> (optional): Contains the foundational code of your app. This folder is optional because you can either keep these files in a dedicated core folder, making it easy to find anything your app relies on, or place them directly in your main structure.</p></li>

<li><p><strong>lib</strong> <em>Also called <strong>utility</strong></em> functions: Generic technical functions that are <strong>not related to domain or business logic</strong> and can be used across your code.</p></li>

</ul>

<h2>
  
  
  Conclusion 💞
</h2>

<p>Start thinking in terms of features, responsibilities, and blocks and let your team and AI agents maintain the code more easily 💪</p>

