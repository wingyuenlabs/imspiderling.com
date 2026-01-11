---
Title: API First in Practice: How We Made Frontend Types Predictable and Stable
Description: 
Author: Dmitrii Verbetchii
Date: 2026-01-11T21:09:09.000Z
Robots: noindex,nofollow
Template: index
---
<p>In many teams, frontend and backend evolve in parallel — and very often, they drift apart.<br>
I’ve seen this happen multiple times: API changes, frontend assumptions stay the same, and bugs appear only after the release.</p>

<p>In our company, we decided to treat this problem as a process issue, not just a tooling one. The solution we landed on was an API First approach, where the frontend relies strictly on the API contract defined by the backend — nothing more, nothing less.</p>

<p>This article is about how we implemented this approach in practice and what it changed for our frontend development.</p>
<h2>
  
  
  The Problem: Frontend–Backend Desynchronization
</h2>

<p>Before adopting API First, our workflow looked familiar:</p>

<ul>
<li>backend changes an endpoint</li>
<li>frontend still uses old assumptions</li>
<li>manual TypeScript types become outdated</li>
<li>bugs appear late — sometimes only in production</li>
</ul>

<p>Even with good communication and documentation, the reality was simple:<br>
<strong>the frontend always found out about breaking changes too late.</strong></p>

<p>TypeScript helps, but only if the types are accurate. And manually maintaining those types doesn’t scale.</p>
<h2>
  
  
  Our Solution: API First as a Contract, Not Documentation
</h2>

<p>In our team, API First means one simple rule:</p>

<blockquote>
<p>The frontend uses only the TypeScript types and interfaces generated from the backend API.</p>
</blockquote>

<p>The OpenAPI (Swagger) schema is not <em>just documentation</em>.<br>
It is the <strong>single source of truth.</strong></p>

<p>If something is not described in the API contract, the frontend doesn’t assume it exists.</p>
<h2>
  
  
  How It Works in Practice (CRUD User Example)
</h2>

<p>Let’s take a simple CRUD example: <strong>users</strong>.</p>

<p>Before the backend implementation is even finished, the backend team provides us with an <strong>OpenAPI schema in YAML format</strong> describing the future API.</p>

<p>Here is a simplified but real example of such a schema:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>openapi: 3.0.3
info:
  title: User Service API
  description: API for managing users
  version: "1.0"

paths:
  /users:
    get:
      tags:
        - users
      summary: Get list of users
      operationId: listUsers
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
        - name: status
          in: query
          schema:
            type: string
            enum: [active, inactive]
      responses:
        "200":
          description: List of users
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UserListResponse"

    post:
      tags:
        - users
      summary: Create new user
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateUserRequest"
      responses:
        "201":
          description: User created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"

  /users/{id}:
    get:
      tags:
        - users
      summary: Get user by ID
      operationId: getUser
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        "200":
          description: User data
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
        email:
          type: string
          format: email
        status:
          type: string
          enum: [active, inactive]

    CreateUserRequest:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
        email:
          type: string
          format: email

    UserListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: "#/components/schemas/User"
        total:
          type: integer
</code></pre>

</div>



<p>This YAML is enough for the frontend to start working — even if the backend implementation is still in progress.</p>

<h2>
  
  
  Generating TypeScript Types and API Requests
</h2>

<p>On the frontend, we use <strong>openapi-generator</strong> to generate TypeScript code from this schema.<br>
The generator gives us:</p>

<ul>
<li>TypeScript types and interfaces</li>
<li>API request functions</li>
</ul>

<p>We don’t write axios calls manually anymore.<br>
We don’t guess request shapes.<br>
We use only what was generated.</p>

<p>For example:</p>

<ul>
<li>
<code>getUser</code> clearly returns a User</li>
<li>
<code>createUser</code> expects <code>CreateUserRequest</code>
This makes the frontend code extremely predictable.</li>
</ul>
<h2>
  
  
  Generated Types as the Foundation of Frontend Code
</h2>

<p>Once the types are generated, they are used everywhere:</p>

<ul>
<li>API layer</li>
<li>React components</li>
<li>Forms</li>
<li>Validation</li>
<li>UI state</li>
</ul>

<p>A very practical example is <strong>enums</strong>.<br>
From the OpenAPI schema, we get a generated enum like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>export const UserStatusEnum = {
  Active: 'active',
  Inactive: 'inactive',
} as const;

export type UserStatusEnum =
  typeof UserStatusEnum[keyof typeof UserStatusEnum];
</code></pre>

</div>



<p>Now the frontend knows that <code>User.status</code> can only be <code>"active"</code> or <code>"inactive"</code>.</p>

<p>We use this enum to:</p>

<ul>
<li>build dropdown options</li>
<li>create filters</li>
<li>drive conditional UI logic</li>
</ul>

<p>No hardcoded strings.<br>
No duplicated constants.<br>
No silent mismatches.</p>

<h2>
  
  
  CI as an Early Warning System
</h2>

<p>This approach really shines once CI is involved.</p>

<p>Our process looks like this:</p>

<ul>
<li>frontend regularly fetches the latest Swagger schema</li>
<li>TypeScript types are regenerated</li>
<li>CI runs <code>tsc</code> and frontend unit tests</li>
</ul>

<p>If the backend changes something unexpectedly:</p>

<ul>
<li>generated types change</li>
<li>TypeScript compilation fails</li>
<li>unit tests fail</li>
</ul>

<p>The important part is <strong>when</strong> this happens.</p>

<blockquote>
<p>Not after the release.<br>
Not in production.<br>
But during CI.</p>
</blockquote>

<p>This means backend and frontend developers are aware of the problem immediately.<br>
Frontend tests become an additional safety net for backend changes.</p>

<h2>
  
  
  What We Gained from This Approach
</h2>

<p>After adopting API First, we noticed clear improvements:</p>

<ul>
<li>predictable and stable frontend types</li>
<li>faster development</li>
<li>fewer questions between teams</li>
<li>earlier detection of breaking changes</li>
<li>much higher confidence when refactoring</li>
</ul>

<p>The frontend stopped <em>guessing</em> and started <strong>trusting the contract</strong>.</p>

<h2>
  
  
  How to Introduce This in Another Team
</h2>

<p>If you want to try this approach, my advice would be:</p>

<ol>
<li>Start with one service</li>
<li>Generate types first, requests later if needed</li>
<li>Enforce usage of generated types</li>
<li>Add type checks to CI</li>
<li>Treat OpenAPI as a contract, not documentation</li>
</ol>

<p>This is less about tools and more about discipline.</p>

<h2>
  
  
  Final Thoughts
</h2>

<p>API First is not about Swagger files or generators.<br>
It’s about predictability.</p>

<p>For a TypeScript frontend, having a reliable contract changes how you think about data.<br>
When the API is the source of truth, the frontend becomes simpler, safer, and more confident.</p>

