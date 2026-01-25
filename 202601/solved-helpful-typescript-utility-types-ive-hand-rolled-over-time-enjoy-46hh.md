---
Title: Solved: Helpful TypeScript Utility Types I’ve hand rolled over time, enjoy
Description: 
Author: Darian Vance
Date: 2026-01-25T21:32:59.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  🚀 Executive Summary
</h3>

<p><strong>TL;DR:</strong> The article addresses common TypeScript challenges such as inflexible built-in utilities, complex rule enforcement, and type-constant drift. It introduces custom utility types like <code>MakeOptional</code>, <code>RequireAtLeastOne</code>, and <code>ObjectValues</code> to provide granular control, enforce ‘at least one’ conditions, and automatically synchronize types with constants, leading to more robust and maintainable code.</p>

<h4>
  
  
  🎯 Key Takeaways
</h4>

<ul>
<li>
<code>MakeOptional</code> provides granular control by making only specified keys <code>K</code> of a type <code>T</code> optional, unlike <code>Partial</code> which affects all properties, by combining <code>Omit</code> and <code>Partial&gt;</code>.</li>
<li>
<code>RequireAtLeastOne</code> enforces that at least one property from a specific set of keys <code>K</code> must be present in an object <code>T</code>, solving complex “either/or” typing scenarios through a union of valid type combinations.</li>
<li>
<code>ObjectValues</code> automatically derives a union type from the values of an object type using <code>T[keyof T]</code>, ensuring types remain synchronized with runtime constants and preventing maintenance burdens, especially when the constant is defined <code>as const</code>.</li>
</ul>

<p>Discover how to enhance your TypeScript projects with powerful, custom utility types. This guide provides reusable solutions to common typing challenges, helping you write cleaner, safer, and more maintainable code by moving beyond built-in defaults.</p>

<h2>
  
  
  The Symptoms: Repetitive and Inflexible TypeScript Patterns
</h2>

<p>As applications grow in complexity, you may find yourself wrestling with TypeScript’s type system to accurately model your data structures and business logic. If you’ve encountered the following symptoms, it’s a sign that you could benefit from a custom utility type toolkit:</p>

<ul>
<li>You frequently create one-off types that are just slight variations of existing interfaces, leading to boilerplate and maintenance overhead.</li>
<li>Built-in utility types like <code>Partial&lt;T&gt;</code> or <code>Required&lt;T&gt;</code> feel too blunt, applying their logic to every property when you only need to modify a few.</li>
<li>You struggle to enforce complex validation rules at the type level, such as requiring “at least one of these specific fields” to be present.</li>
<li>Your types and runtime constants (like configuration objects) drift out of sync, forcing you to update them in two separate places and introducing the risk of errors.</li>
</ul>

<p>These challenges indicate a need for more precise and reusable type definitions. Let’s explore three hand-rolled utility types that solve these exact problems.</p>

<h2>
  
  
  Solution 1: Granular Control with <code>MakeOptional&lt;T, K&gt;</code>
</h2>

<h3>
  
  
  The Problem: <code>Partial&lt;T&gt;</code> is All or Nothing
</h3>

<p>TypeScript’s built-in <code>Partial&lt;T&gt;</code> is useful, but it makes every single property of a type optional. This is often not what we need. For instance, when updating a user profile, you might allow the <code>bio</code> and <code>profilePictureUrl</code> to be optional in the payload, but the user’s <code>id</code> is always required to identify them. Using <code>Partial&lt;User&gt;</code> would incorrectly make the <code>id</code> optional too.</p>

<h3>
  
  
  The Utility Type: <code>MakeOptional&lt;T, K&gt;</code>
</h3>

<p>This utility type allows you to pick specific keys from a type <code>T</code> and make only them optional, leaving the rest untouched. It provides the granularity that <code>Partial&lt;T&gt;</code> lacks.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>type MakeOptional&lt;T, K extends keyof T&gt; = Omit&lt;T, K&gt; &amp; Partial&lt;Pick&lt;T, K&gt;&gt;;
</code></pre>

</div>



<p>This works by creating an intersection of two types:</p>

<ol>
<li>
<code>Omit&lt;T, K&gt;</code>: Takes the original type <code>T</code> and removes the keys <code>K</code> that we want to make optional. The remaining properties are still required.</li>
<li>
<code>Partial&lt;Pick&lt;T, K&gt;&gt;</code>: First, it <code>Pick</code>s only the keys <code>K</code> from <code>T</code>, and then it wraps them in <code>Partial</code> to make them all optional.</li>
</ol>

<p>The <code>&amp;</code> operator combines them, resulting in a new type where the specified keys <code>K</code> are optional and all other keys from <code>T</code> remain as they were.</p>

<h3>
  
  
  Real-World Example: Updating a User Configuration
</h3>

<p>Imagine you have a strict <code>AppConfig</code> interface. You want to create a function that allows overriding just the <code>theme</code> or <code>notifications</code> settings.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>interface AppConfig {
  readonly appId: string;
  readonly apiEndpoint: string;
  theme: 'dark' | 'light';
  notifications: {
    email: boolean;
    push: boolean;
  };
}

// Our custom utility type
type MakeOptional&lt;T, K extends keyof T&gt; = Omit&lt;T, K&gt; &amp; Partial&lt;Pick&lt;T, K&gt;&gt;;

// Define the type for our update payload
type ConfigUpdatePayload = MakeOptional&lt;AppConfig, 'theme' | 'notifications'&gt;;

/*
The resulting type is:
{
  readonly appId: string;
  readonly apiEndpoint: string;
  theme?: 'dark' | 'light';       // Now optional
  notifications?: {               // Now optional
    email: boolean;
    push: boolean;
  };
}
*/

function updateConfig(update: ConfigUpdatePayload) {
  // ... implementation
}

// VALID: We provide all required fields and one optional one.
updateConfig({
  appId: 'abc-123',
  apiEndpoint: 'https://api.example.com',
  theme: 'dark',
});

// INVALID: Missing a required field 'apiEndpoint'
// TypeScript Error: Property 'apiEndpoint' is missing in type...
updateConfig({
  appId: 'abc-123',
  notifications: { email: true, push: false },
});
</code></pre>

</div>



<h3>
  
  
  Comparison: <code>MakeOptional</code> vs. <code>Partial</code>
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th></th>
<th></th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Feature</strong></td>
<td><strong><code>Partial&lt;T&gt;</code></strong></td>
<td><strong><code>MakeOptional&lt;T, K&gt;</code></strong></td>
</tr>
<tr>
<td>Granularity</td>
<td>All-or-nothing. Affects all properties of <code>T</code>.</td>
<td>Selective. Affects only the specified keys <code>K</code>.</td>
</tr>
<tr>
<td>Common Use Case</td>
<td>Mocking objects for tests where most fields are irrelevant.</td>
<td>Typing API PATCH request payloads or update functions.</td>
</tr>
<tr>
<td>Flexibility</td>
<td>Low. Cannot preserve required status for any fields.</td>
<td>High. Can precisely model which fields can be omitted.</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Solution 2: Enforcing Complex Rules with <code>RequireAtLeastOne&lt;T, K&gt;</code>
</h2>

<h3>
  
  
  The Problem: Typing “Either/Or” Scenarios
</h3>

<p>Standard interfaces are great for defining shapes where all properties are independent. But what if your business logic requires that *at least one* property from a specific set must be provided? For example, a function to find a user might accept a <code>userId</code>, an <code>email</code>, or a <code>username</code>. It needs at least one, but a consumer could provide more. A standard interface can’t enforce this “at least one” rule.</p>

<h3>
  
  
  The Utility Type: <code>RequireAtLeastOne&lt;T, K&gt;</code>
</h3>

<p>This advanced utility type ensures that at least one of the specified keys <code>K</code> exists on the object.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>type RequireAtLeastOne&lt;T, Keys extends keyof T = keyof T&gt; =
    Pick&lt;T, Exclude&lt;keyof T, Keys&gt;&gt; 
    &amp; {
        [K in Keys]-?: Required&lt;Pick&lt;T, K&gt;&gt; &amp; Partial&lt;Pick&lt;T, Exclude&lt;Keys, K&gt;&gt;&gt;
    }[Keys];
</code></pre>

</div>



<p>This type is more complex, but its core logic is to create a union of all possible valid combinations. For each key in <code>Keys</code>, it creates a type where that specific key is required and the other keys in the set are optional. The union <code>[Keys]</code> at the end means the final type must match one of these valid combinations.</p>

<h3>
  
  
  Real-World Example: Flexible Search Parameters
</h3>

<p>Let’s model a search function that requires either a document <code>id</code> or a <code>slug</code> to locate a document.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>interface DocumentIdentifiers {
  id: string;
  slug: string;
  authorId: number; // This field is independent and always optional
}

// Our utility type
type RequireAtLeastOne&lt;T, Keys extends keyof T = keyof T&gt; =
    Pick&lt;T, Exclude&lt;keyof T, Keys&gt;&gt; 
    &amp; {
        [K in Keys]-?: Required&lt;Pick&lt;T, K&gt;&gt; &amp; Partial&lt;Pick&lt;T, Exclude&lt;Keys, K&gt;&gt;&gt;
    }[Keys];

// We need at least 'id' or 'slug'
type FindDocumentParams = RequireAtLeastOne&lt;Partial&lt;DocumentIdentifiers&gt;, 'id' | 'slug'&gt;;

function findDocument(params: FindDocumentParams) {
  // ... implementation
}

// VALID calls
findDocument({ id: 'doc-123' });
findDocument({ slug: 'my-first-post' });
findDocument({ id: 'doc-123', slug: 'my-first-post' });
findDocument({ id: 'doc-123', authorId: 42 }); // other optional fields are fine

// INVALID: Missing both 'id' and 'slug'
// TypeScript Error: Type '{ authorId: number; }' has no properties in common 
// with type 'Required&lt;Pick&lt;...&gt;&gt; &amp; ...'.
findDocument({ authorId: 42 });
</code></pre>

</div>



<h2>
  
  
  Solution 3: Deriving Types from Constants with <code>ObjectValues&lt;T&gt;</code>
</h2>

<h3>
  
  
  The Problem: Keeping Types and Constants in Sync
</h3>

<p>A common pattern is to define a set of string constants in an object for things like event names, status codes, or user roles. You then create a separate union type by hand to represent those values. This creates a maintenance burden: if you add a new role to the constant object, you must remember to update the union type as well. Forgetting to do so can lead to subtle bugs that TypeScript won’t catch.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// The error-prone manual approach
export const USER_ROLES = {
  ADMIN: 'admin',
  EDITOR: 'editor',
  VIEWER: 'viewer',
  // If we add a new role here...
};

// ...we have to remember to update the type here!
export type UserRole = 'admin' | 'editor' | 'viewer';
</code></pre>

</div>



<h3>
  
  
  The Utility Type: <code>ObjectValues&lt;T&gt;</code>
</h3>

<p>This simple but powerful utility creates a union type from the values of a given object type. It ensures your types are always derived directly from your constants, eliminating drift.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>type ObjectValues&lt;T&gt; = T[keyof T];
</code></pre>

</div>



<p>This works by using the indexed access type <code>T[keyof T]</code>. <code>keyof T</code> creates a union of all the keys of <code>T</code>. When used as an index, <code>T[…]</code> looks up the types of all those keys and combines them into a new union type.</p>

<h3>
  
  
  Real-World Example: Defining Application Statuses
</h3>

<p>Let’s define a single source of truth for deployment statuses and derive the type automatically.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Our single source of truth for statuses.
// 'as const' is crucial here! It tells TypeScript to infer the most specific
// type possible (e.g., 'PENDING' instead of just 'string').
export const DEPLOYMENT_STATUS = {
  PENDING: 'pending',
  IN_PROGRESS: 'in_progress',
  SUCCESS: 'success',
  FAILED: 'failed',
} as const;

// Our utility type
type ObjectValues&lt;T&gt; = T[keyof T];

// The derived type. It is now automatically synced with the constant.
// Result: type DeploymentStatus = "pending" | "in_progress" | "success" | "failed"
export type DeploymentStatus = ObjectValues&lt;typeof DEPLOYMENT_STATUS&gt;;

function setDeploymentStatus(id: string, status: DeploymentStatus) {
  console.log(`Setting deployment ${id} to status: ${status}`);
}

// VALID
setDeploymentStatus('deploy-abc', DEPLOYMENT_STATUS.IN_PROGRESS);

// INVALID
// TypeScript Error: Argument of type '"error"' is not assignable to 
// parameter of type 'DeploymentStatus'.
setDeploymentStatus('deploy-xyz', 'error');
</code></pre>

</div>



<h2>
  
  
  Conclusion: Building Your Own Type Toolkit
</h2>

<p>While TypeScript’s built-in utility types are a great starting point, the real power comes from composing them to solve the specific, recurring problems in your codebase. By creating your own small, focused utility types like <code>MakeOptional</code>, <code>RequireAtLeastOne</code>, and <code>ObjectValues</code>, you can:</p>

<ul>
<li>
<strong>Reduce Boilerplate:</strong> Stop writing one-off types and create reusable, descriptive solutions.</li>
<li>
<strong>Increase Type Safety:</strong> Model complex business rules directly in the type system, catching errors at compile time instead of runtime.</li>
<li>
<strong>Improve Maintainability:</strong> Create a single source of truth for your types and constants, making your code easier to refactor and understand.</li>
</ul>

<p>Start looking for repetitive type patterns in your projects today. Chances are, you can abstract them into a helpful utility type and begin building a robust type toolkit that will pay dividends across your entire application.</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" alt="Darian Vance" width="758" height="289"></a></p>

<p>👉 <a href="https://wp.me/pbK4oa-a2" rel="noopener noreferrer">Read the original article on TechResolve.blog</a></p>




<p>☕ <strong>Support my work</strong>  </p>

<p>If this article helped you, you can buy me a coffee:  </p>

<p>👉 <a href="https://buymeacoffee.com/darianvance" rel="noopener noreferrer">https://buymeacoffee.com/darianvance</a></p>

