---
Title: Scaling API Access with Azure API Management: From Manual to Self-Service
Description: 
Author: Anoush
Date: 2026-01-04T22:03:40.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Introduction
</h2>

<p>Working on a healthcare application using .NET and Azure. The backend is .NET Core with SQL Server and Azure Functions on Azure Cloud. This application is a B2B product, where we need to expose our APIs to external clients and users who integrate with our services.</p>

<p>We came out first with a traditional approach where each client would get custom API endpoints and we generate a GUID id (API key) for each client/user. We store the ids in our database and based on API key we authorize and authenticate the clients. But as we scaled up hundreds/thousands of facilities and users, this became unmanageable. Also, it is not secure to have the main API endpoint accessible to public. We needed a better way.</p>

<p>This article describes how I architected our API access to be scalable, self-service and more secure. Basically, going from a manually configured API setup to an automated multi-tenant API gateway that serves thousands of users with zero manual configuration.</p>

<h2>
  
  
  The Problem
</h2>

<p>Our application needed to serve hundreds or thousands of external users across multiple facilities. Each facility needed access to our APIs to exchange data. But here's the challenge: each client has access to different APIs and needed different parameters to push and pull their data.</p>

<p>For example:</p>

<ul>
<li>Facility A needs facilityId=5001&amp;locationId=1001</li>
<li>Facility B needs facilityId=5002&amp;locationId=1002</li>
<li>Facility C needs facilityId=5003&amp;locationId=1003$specificParams=Ture</li>
</ul>

<p>The traditional approach would be to create separate API id called API Key in the database and endpoints would access their data based on their API Key. Or worse, require each facility to pass their own parameters and trust they're sending the right ones. Neither scales. The first creates endpoint sprawl. The second creates security issues.</p>

<p>We needed our users to sign up, set up their settings (set their facility, location and other parameters in UI) and choose their API endpoint that could serve everyone securely, with each user automatically getting their correct parameters without manual configuration.</p>

<h2>
  
  
  Architecture and Setup
</h2>

<p>We built our solution using three Azure services: Azure API Management (APIM), Azure Functions, and Azure Table Storage.<br>
APIM acts as our API gateway. It's the single entry point for all external users. Functions handle our business logic and subscription management. I used APIM to set up API policies. Table Storage holds each subscription's custom parameters.</p>

<p>Here's how I set it up. In APIM, I created one API endpoint. This single endpoint serves all facilities. When a user calls this endpoint with their subscription key, APIM does all the heavy lifting automatically through a policy.</p>

<p>I have a separate function that is fully dedicated to managing APIM subscriptions. I created two main APIs: one to manage subscription creation (POST /api/subscription/create) and one to retrieve subscription parameters (GET /api/subscription/{id}). The first one is called when users sign up. The second one is called by APIM's policy to get that user's parameters.</p>

<p>In Table Storage, we store a simple table called ApiSubscriptions with three columns: SubscriptionId (the primary key), SubscriptionKey, and Parameters. When a user signs up, I store their parameters here as a query string like facilityId=5001&amp;locationId=1001.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwzy28dhtegn8g56wxy0i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwzy28dhtegn8g56wxy0i.png" alt=" " width="800" height="533"></a></p>
<h2>
  
  
  The Flow
</h2>

<p>When a user calls our API, here's what happens automatically:<br>
The facility sends a GET request to <a href="https://api.example.com/data/ids" rel="noopener noreferrer">https://api.example.com/data/ids</a> with their subscription key in the header: Ocp-Apim-Subscription-Key: abc123...<br>
APIM receives the request and validates the subscription key. If it's invalid, the request is rejected immediately.</p>

<p>Once validated, APIM executes our policy. The policy does three things in sequence:</p>

<p><strong>First</strong>, it calls Azure AD to get an OAuth token. This token is needed to call our backend Function App securely. APIM makes a POST request to <a href="https://login.microsoftonline.com/%7Btenant%7D/oauth2/v2.0/token" rel="noopener noreferrer">https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token</a> with client credentials. Azure AD returns a bearer token.</p>

<p><strong>Second</strong>, APIM calls our Function App to get this subscription's parameters. It makes a GET request to /api/subscription/{subscriptionId} with the bearer token in the Authorization header. The Function App looks up the subscription in Table Storage and returns the parameters: facilityId=5001&amp;locationId=1001.</p>

<p><strong>Third</strong>, APIM appends these parameters to the main backend API call. It rewrites the URL from /data/ids to /data/ids?code={functionCode}&amp;facilityId=5001&amp;locationId=1001 and forwards the request to the Function App with the OAuth token.</p>

<p>The Function App processes the request with parameters and returns the facility's requested data. APIM forwards this response back to the user.<br>
All of this happens in milliseconds. The facility doesn't know any of this is happening. They just send their subscription key (in the header of the call) and get their data back.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fji7omdsp2gv13dpd8nqb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fji7omdsp2gv13dpd8nqb.png" alt="Azure architecture diagram showing APIM policy flow with 3 steps: OAuth token retrieval, subscription parameter lookup, and backend API call with authentication" width="800" height="533"></a></p>

<p><em>The diagram shows the complete flow from API Client through APIM to backend services with OAuth authentication and dynamic parameter injection.</em></p>

<p><strong>The APIM Policy</strong><br>
The entire flow is controlled by an APIM policy. This is an XML-based configuration that runs for every API request. Here's what our policy looks like, broken down into the three main steps:<br>
<strong>Step 1: Get OAuth Token</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;send-request mode="new" response-variable-name="tokenResponse" timeout="10"&gt;
    &lt;set-url&gt;https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token&lt;/set-url&gt;
    &lt;set-method&gt;POST&lt;/set-method&gt;
    &lt;set-header name="Content-Type"&gt;
        &lt;value&gt;application/x-www-form-urlencoded&lt;/value&gt;
    &lt;/set-header&gt;
    &lt;set-body&gt;grant_type=client_credentials&amp;client_id={id}&amp;client_secret={secret}&amp;scope={id}/.default&lt;/set-body&gt;
&lt;/send-request&gt;
</code></pre>

</div>



<p>This calls Azure AD and gets back an access token. We store it in a variable called tokenResponse.</p>

<p><strong>Step 2: Get Subscription Parameters</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;send-request mode="new" response-variable-name="subscriptionResponse" timeout="10"&gt;
    &lt;set-url&gt;@($"https://{function-app}/api/subscription/{context.Subscription.Id}?code={code}")&lt;/set-url&gt;
    &lt;set-method&gt;GET&lt;/set-method&gt;
    &lt;set-header name="Authorization"&gt;
        &lt;value&gt;@("Bearer " + accessToken)&lt;/value&gt;
    &lt;/set-header&gt;
&lt;/send-request&gt;
</code></pre>

</div>



<p>This calls our Function App to get the parameters. Notice we're using context.Subscription.Id which is the unique subscription identifier in APIM. We send the OAuth token in the Authorization header.</p>

<p><strong>Step 3: Call Backend with Parameters</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;set-header name="Authorization"&gt;
    &lt;value&gt;@("Bearer " + accessToken)&lt;/value&gt;
&lt;/set-header&gt;

&lt;rewrite-uri template="@{
    string parameters = context.Variables["parameters"];
    return $"/api/data?code={code}&amp;{parameters}";
}" /&gt;
</code></pre>

</div>



<p>This appends the parameters to the backend call and sets the OAuth token in the Authorization header. The backend API receives the request with both authentication and the facility's specific parameters.</p>

<p>That's the entire policy. Three steps that run automatically for every request.</p>

<p>I also have other types of APIM Policies. For example, one that I use often is the rate-limit policy. That protects our APIs when there are a lot of calls during a short amount of time. This built-in policy will give "Too Many Requests" (HTTP 429) when that occurs.</p>

<h2>
  
  
  Subscription Management
</h2>

<p>We built a Function App to handle subscription lifecycle. It has two main APIs:</p>

<p>Create Subscription (POST /api/subscription/create)</p>

<p>This function does four things:</p>

<ol>
<li>Creates the user in APIM (if they don't exist)</li>
<li>Creates the subscription in APIM for the selected product</li>
<li>Stores the subscription parameters in Table Storage</li>
<li>Returns the subscription key and ID to the user</li>
</ol>

<p>Here's the data flow: user submits their email, name, product selection, and facility selection through our signup portal. The Function App creates an APIM user using their email as the identifier. Then it creates a subscription for that user to our API product. Then it stores the subscription ID and parameters (facilityId=X&amp;locationId=Y) in Table Storage. Finally, it returns the subscription key to the user.</p>

<p>*<em>Get Subscription *</em>(GET /api/subscription/{id})</p>

<p>This function is called by the APIM policy. It takes a subscription ID, looks it up in Table Storage, and returns the parameters. Simple lookup operation.</p>

<p>Both functions use OAuth authentication. Only APIM can call them because APIM uses Managed Identity to get a token from Azure AD.</p>

<p>One important configuration: the Function App's Managed Identity must authenticate to the correct Azure AD tenant. I explicitly set the tenant ID in the DefaultAzureCredential to ensure it gets tokens from the right Azure AD instance. Without this, authentication fails with a tenant mismatch error.</p>

<h2>
  
  
  Developer Portal
</h2>

<p>I built two signup portals: one integrated into the APIM Developer Portal and one standalone static website hosted on Azure Storage.</p>

<p>In the APIM Developer Portal, I created a custom page called "Create Subscription". Users can access this page, fill out a form with their email, name, product selection, facility selection, other settings and click "Create Subscription". The form calls our Function App's create endpoint, and the user immediately gets their subscription key displayed on screen.</p>

<p>The standalone portal is a simple HTML page with JavaScript. It does the same thing but can be shared with external users who don't need to log into the APIM portal. We use this for quick onboarding by email invitations.</p>

<p>Both portals load the facility list dynamically from another Function App endpoint (GET /api/facilities) which queries our database for active facilities. It also lets users choose their APIM Product or API Group.<br>
For the portals to call the Function App from the browser, I configured CORS in the APIM policy. I set allowed-origins to wildcard (*) for the POC. For production, this should be locked down to specific portal URLs only.</p>

<p>When the user submits the form, JavaScript calls POST /api/subscription/create with the payload example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "email": "user@example.com",
  "name": "John Doe",
  "product": "example-apis",
  "facilityId": "5001",
  "locationId": "1001"
}
</code></pre>

</div>



<p>The Function App does all the work and returns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{
  "subscriptionKey": "abc123...",
  "subscriptionId": "user-example-com-xyz789"
}
</code></pre>

</div>



<p>The portal displays this to the user with instructions on how to use it.</p>

<h2>
  
  
  Conclusion
</h2>

<p>This architecture solved our multi-tenant API access problem, without admin involvement and separates authentication and authorization using APIM. Each user gets their correct parameters automatically without manual configuration.</p>

<p>The key components:</p>

<ul>
<li>APIM handles routing and policy execution</li>
<li>Azure Functions manage subscriptions and business logic</li>
<li>Table Storage holds subscription parameters</li>
<li>OAuth secures all communications</li>
<li>APIM Policy ties it all together with three simple steps</li>
</ul>

<p>If you're building B2B APIs that need to serve multiple tenants with different parameters, this pattern works. It's scalable, secure, and self-service.</p>

<p>The full code and APIM policy are available in production. The system runs itself.</p>

