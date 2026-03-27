---
Title: How to Build a BIN Lookup Middleware in Node.js in Under an Hour
Description: 
Author: Sam
Date: 2026-03-27T21:37:26.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  TL;DR
</h2>

<ul>
<li>A BIN (Bank Identification Number) is the first 6 to 8 digits of any payment card and tells you the issuing bank, card type, country of origin, and whether the card is prepaid or credit.</li>
<li>You will build a reusable Express middleware function that intercepts incoming card data, calls a BIN lookup API, and attaches structured metadata to <code>req.binInfo</code>.</li>
<li>The final middleware lets you block prepaid cards, filter by country, flag high-risk issuers, and add card branding in one clean layer before your payment logic runs.</li>
<li>The API requires two credentials stored in <code>.env</code>: <code>BIN_API_KEY</code> and <code>X_USER_ID</code>. Both are sent as request headers on every lookup call.</li>
<li>BIN values sent to the API must be between 6 and 8 digits. Fewer than 6 cannot identify an issuer. More than 8 starts encoding the cardholder account number, which you must never send to a third-party service.</li>
<li>Stack: Node.js 18+, Express 4, Axios, dotenv.</li>
<li>Total build time: under 60 minutes if you follow every step.</li>
</ul>




<h2>
  
  
  Why BIN Lookup Middleware Is Worth Your Time
</h2>

<p>If you process payments, subscriptions, or any kind of card-based transaction, you already know that not all cards are created equal.</p>

<p>A prepaid card from a high-risk country behaves very differently from a corporate Visa issued by a US bank. Your fraud rules should know the difference before a charge attempt ever reaches Stripe, Braintree, or your processor of choice.</p>

<p>The cleanest place to handle that logic is middleware. You run the BIN check once, attach the result to the request object, and every subsequent handler in your chain can read from <code>req.binInfo</code> without making a second API call. Your fraud logic, logging layer, and analytics all pull from the same enriched source.</p>

<p>This is a pattern used in production payment platforms. It is not complicated once you see it laid out step by step.</p>

<p>Let us build it.</p>




<h2>
  
  
  Prerequisites
</h2>

<p>Before you write a single line of code, make sure your environment is ready.</p>

<p><strong>Node.js version:</strong> 18 or higher. Run <code>node -v</code> to check. Node 18 ships with the native <code>fetch</code> API, but we will use Axios in this tutorial because it gives you cleaner error handling and automatic JSON parsing.</p>

<p><strong>Package manager:</strong> npm or yarn. Either works.</p>

<p><strong>Packages you will install:</strong></p>

<ul>
<li>
<code>express</code> (version 4.x) for the HTTP server and routing</li>
<li>
<code>axios</code> for HTTP requests to the BIN API</li>
<li>
<code>dotenv</code> for managing your API credentials as environment variables</li>
</ul>

<p><strong>A BIN lookup API key and User ID:</strong> This tutorial uses <a href="https://www.binsearchlookup.com/" rel="noopener noreferrer">binsearchlookup.com</a> for the lookup calls. Sign up, grab your API key and your User ID from the dashboard, and keep both nearby. You will need them in Step 2.</p>

<p><strong>A tool for sending test requests:</strong> curl, Postman, or the VS Code REST Client extension all work.</p>




<h2>
  
  
  Step 1: Set Up a Minimal Express Server
</h2>

<p>Create a new project folder and initialize it.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">mkdir </span>bin-middleware-demo
<span class="nb">cd </span>bin-middleware-demo
npm init <span class="nt">-y</span>
npm <span class="nb">install </span>express axios dotenv
</code></pre>

</div>



<p>Create your entry file and the middleware directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">touch </span>index.js
<span class="nb">touch</span> .env
<span class="nb">mkdir </span>middleware
<span class="nb">touch </span>middleware/binLookup.js
</code></pre>

</div>



<p>Open <code>index.js</code> and write the skeleton server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// index.js</span>
<span class="c1">// Load environment variables from .env before anything else</span>
<span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">).</span><span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">express</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>

<span class="c1">// Parse incoming JSON bodies</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>

<span class="kd">const</span> <span class="nx">binLookupMiddleware</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./middleware/binLookup</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Apply the BIN middleware only to the validate-card route</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">"</span><span class="s2">/validate-card</span><span class="dl">"</span><span class="p">,</span> <span class="nx">binLookupMiddleware</span><span class="p">,</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="c1">// At this point req.binInfo is already populated by the middleware</span>
  <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
    <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">binInfo</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Server running on port </span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Run <code>node index.js</code> and confirm the server starts. You should see the log line in your terminal. The <code>/validate-card</code> endpoint will return an error right now because the middleware file is still empty, but the structure is correct. Move on.</p>




<h2>
  
  
  Step 2: Store Both Credentials Safely
</h2>

<p>The BIN lookup API requires two headers on every request: your API key and your User ID. Both live in your <code>.env</code> file so they never appear hardcoded in source code.</p>

<p>Open <code>.env</code> and add your credentials:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># .env

# Your BIN API key from binsearchlookup.com
BIN_API_KEY=bsl_7ffc440d90186fc5040d214bbed9e584686941e2e89d5168c137fe836e158a03

# Your User ID from your binsearchlookup.com account dashboard
X_USER_ID=e65dadd2-4797-4f77-9316-fe535f9b9268

PORT=3000
</code></pre>

</div>



<p>Never commit this file. Add it to <code>.gitignore</code> right now:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">echo</span> <span class="s2">".env"</span> <span class="o">&gt;&gt;</span> .gitignore
</code></pre>

</div>



<p>This one habit prevents credential leaks. Your credentials live in the environment, and your code reads them with <code>process.env.BIN_API_KEY</code> and <code>process.env.X_USER_ID</code>. No hardcoded strings, ever.</p>




<h2>
  
  
  Step 3: Extract the BIN From the Request
</h2>

<p>The BIN is the first 6 to 8 digits of a card number. In real integrations you often receive the full card number on a server-to-server call, or a tokenized version that still carries the BIN prefix.</p>

<p>For this tutorial the client sends a JSON body like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"cardNumber"</span><span class="p">:</span><span class="w"> </span><span class="s2">"5510291234567890"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Your middleware needs to strip everything except the first 8 digits, then validate that the result falls within the required 6-to-8 digit range. Fewer than 6 digits cannot identify a unique issuer. More than 8 digits starts encoding the individual account number, which is sensitive cardholder data you must never send to a third-party API.</p>

<p>Open <code>middleware/binLookup.js</code> and write the extraction logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// middleware/binLookup.js</span>
<span class="kd">const</span> <span class="nx">axios</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">axios</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// BIN must be between 6 and 8 digits -- no more, no less.</span>
<span class="c1">// Fewer than 6 cannot identify a unique issuer.</span>
<span class="c1">// More than 8 encodes the account number, which is sensitive cardholder data.</span>
<span class="kd">function</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">cardNumber</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">cardNumber</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

  <span class="c1">// Remove any spaces or dashes the client might have included</span>
  <span class="kd">const</span> <span class="nx">cleaned</span> <span class="o">=</span> <span class="nx">cardNumber</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\D</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>

  <span class="c1">// Take the first 8 digits at most</span>
  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nx">cleaned</span><span class="p">.</span><span class="nf">substring</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>

  <span class="c1">// Reject if we cannot produce at least 6 digits</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>

  <span class="k">return</span> <span class="nx">bin</span><span class="p">;</span>
<span class="p">}</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">binLookupMiddleware</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">cardNumber</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">);</span>

  <span class="c1">// Strict range check: the BIN must be 6, 7, or 8 digits</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">bin</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">8</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">BIN must be between 6 and 8 digits. Please provide a valid card number.</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Attach the raw BIN to req for reference downstream</span>
  <span class="nx">req</span><span class="p">.</span><span class="nx">bin</span> <span class="o">=</span> <span class="nx">bin</span><span class="p">;</span>

  <span class="c1">// The API call happens in the next step -- for now just continue</span>
  <span class="nf">next</span><span class="p">();</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Notice that the middleware calls <code>next()</code> at the end. That is the Express convention. When you are done with your work, you hand control to the next handler in the chain. If you need to stop the request early, like the 400 response above, you return the response directly without calling <code>next()</code>.</p>




<h2>
  
  
  Step 4: Call the BIN Lookup API
</h2>

<p>Now replace the placeholder <code>next()</code> with the actual API request. You will call the BIN lookup service, receive a structured response, and attach it to <code>req.binInfo</code>.</p>

<p>The endpoint and required headers look like this when called directly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-H</span> <span class="s2">"X-API-Key: bsl_7ffc440d90186fc5040d214bbed9e584686941e2e89d5168c137fe836e158a03"</span> <span class="se">\</span>
     <span class="nt">-H</span> <span class="s2">"X-User-ID: e65dadd2-4797-4f77-9316-fe535f9b9268"</span> <span class="se">\</span>
     <span class="s2">"https://api.binsearchlookup.com/lookup?bin=551029"</span>
</code></pre>

</div>



<p>Two things to notice here. First, the BIN is passed as a query parameter (<code>?bin=</code>), not as a URL path segment. Second, both <code>X-API-Key</code> and <code>X-User-ID</code> are required headers. A request missing either one will be rejected by the API.</p>

<p>A successful response returns a JSON object similar to this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"bin"</span><span class="p">:</span><span class="w"> </span><span class="s2">"551029"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"brand"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MASTERCARD"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CREDIT"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"category"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CLASSIC"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"issuer"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Citibank"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"country"</span><span class="p">:</span><span class="w"> </span><span class="s2">"US"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"phone"</span><span class="p">:</span><span class="w"> </span><span class="s2">"+18006274276"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"website"</span><span class="p">:</span><span class="w"> </span><span class="s2">"www.citibank.com"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"country"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"United States"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"alpha2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"US"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"alpha3"</span><span class="p">:</span><span class="w"> </span><span class="s2">"USA"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"numeric"</span><span class="p">:</span><span class="w"> </span><span class="s2">"840"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"currency"</span><span class="p">:</span><span class="w"> </span><span class="s2">"USD"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"prepaid"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Now write the full middleware that makes this call. Replace the entire contents of <code>middleware/binLookup.js</code> with the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// middleware/binLookup.js -- full version with API call</span>
<span class="kd">const</span> <span class="nx">axios</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">axios</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// BIN must be exactly 6 to 8 digits.</span>
<span class="c1">// Anything shorter cannot identify an issuer.</span>
<span class="c1">// Anything longer encodes the account number -- never send that to a third party.</span>
<span class="kd">function</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">cardNumber</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">cardNumber</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">cleaned</span> <span class="o">=</span> <span class="nx">cardNumber</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\D</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nx">cleaned</span><span class="p">.</span><span class="nf">substring</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">return</span> <span class="nx">bin</span><span class="p">;</span>
<span class="p">}</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">binLookupMiddleware</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">cardNumber</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">bin</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">8</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">BIN must be between 6 and 8 digits. Please provide a valid card number.</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="nx">req</span><span class="p">.</span><span class="nx">bin</span> <span class="o">=</span> <span class="nx">bin</span><span class="p">;</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">axios</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="dl">"</span><span class="s2">https://api.binsearchlookup.com/lookup</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">params</span><span class="p">:</span> <span class="p">{</span>
          <span class="c1">// Pass the extracted BIN as a query parameter</span>
          <span class="c1">// The API accepts 6, 7, or 8 digits</span>
          <span class="na">bin</span><span class="p">:</span> <span class="nx">bin</span><span class="p">,</span>
        <span class="p">},</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="c1">// Both headers are required -- both are read from environment variables</span>
          <span class="dl">"</span><span class="s2">X-API-Key</span><span class="dl">"</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BIN_API_KEY</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">X-User-ID</span><span class="dl">"</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">X_USER_ID</span><span class="p">,</span>
        <span class="p">},</span>
        <span class="c1">// Fail fast: if the BIN API takes longer than 3 seconds, soft-fail</span>
        <span class="na">timeout</span><span class="p">:</span> <span class="mi">3000</span><span class="p">,</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="c1">// Attach the full enriched BIN data to the request object.</span>
    <span class="c1">// Every downstream handler can now read req.binInfo without a second API call.</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">;</span>

    <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Log the failure internally but never expose credentials or internals to the client</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">BIN lookup failed:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>

    <span class="c1">// Soft-fail: set binInfo to null and continue.</span>
    <span class="c1">// A BIN API outage should not take your entire checkout flow down.</span>
    <span class="c1">// Your route handler decides what to do when binInfo is null.</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>Your middleware now handles both credentials, enforces the BIN digit range, and attaches everything downstream handlers need.</p>




<h2>
  
  
  Step 5: Use the Metadata for Real Business Rules
</h2>

<p>The middleware has done its job. <code>req.binInfo</code> is populated. Every handler downstream can read it without making another API call.</p>

<p>Open <code>index.js</code> and replace the skeleton route handler with real policy logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// index.js -- full version with business rules</span>
<span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">).</span><span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">express</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>

<span class="kd">const</span> <span class="nx">binLookupMiddleware</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./middleware/binLookup</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Define which issuer countries your business accepts</span>
<span class="kd">const</span> <span class="nx">ALLOWED_COUNTRIES</span> <span class="o">=</span> <span class="p">[</span><span class="dl">"</span><span class="s2">US</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CA</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">GB</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">AU</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">DE</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">FR</span><span class="dl">"</span><span class="p">];</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">"</span><span class="s2">/validate-card</span><span class="dl">"</span><span class="p">,</span> <span class="nx">binLookupMiddleware</span><span class="p">,</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span><span class="p">;</span>

  <span class="c1">// If the BIN lookup failed (API outage, unknown BIN), enforce your fallback policy.</span>
  <span class="c1">// This example blocks the transaction. You could also allow it with a lower trust score.</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">binInfo</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">422</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Unable to verify card origin. Please try again.</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Rule 1: Block prepaid cards.</span>
  <span class="c1">// Prepaid cards are high-risk for chargebacks and subscription fraud.</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">binInfo</span><span class="p">.</span><span class="nx">prepaid</span> <span class="o">===</span> <span class="kc">true</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">403</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Prepaid cards are not accepted for this service.</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="dl">"</span><span class="s2">PREPAID_CARD_BLOCKED</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Rule 2: Block cards issued outside your allowed countries.</span>
  <span class="c1">// binInfo.country.alpha2 is the ISO 3166-1 two-letter country code.</span>
  <span class="kd">const</span> <span class="nx">issuerCountry</span> <span class="o">=</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">country</span><span class="p">?.</span><span class="nx">alpha2</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">issuerCountry</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="nx">ALLOWED_COUNTRIES</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">issuerCountry</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">403</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Cards from this region are not supported.</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="dl">"</span><span class="s2">COUNTRY_NOT_ALLOWED</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">country</span><span class="p">:</span> <span class="nx">issuerCountry</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// All checks passed. Return enriched card metadata to the caller.</span>
  <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
    <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">bin</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">bin</span><span class="p">,</span>
    <span class="na">cardBrand</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">brand</span><span class="p">,</span>
    <span class="na">cardType</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">type</span><span class="p">,</span>
    <span class="na">issuer</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">issuer</span><span class="p">?.</span><span class="nx">name</span><span class="p">,</span>
    <span class="na">country</span><span class="p">:</span> <span class="nx">issuerCountry</span><span class="p">,</span>
    <span class="na">prepaid</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">prepaid</span><span class="p">,</span>
    <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Card validated successfully.</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Server running on port </span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Each rule is a single conditional block. You can add, remove, or reorder rules without touching the middleware at all. The middleware fetches and attaches. The route handler enforces policy. Two different responsibilities, two different places.</p>




<h2>
  
  
  Step 6: Test Every Scenario With Curl
</h2>

<p>Start your server:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>node index.js
</code></pre>

</div>



<p><strong>Test 1: A standard US credit card</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/validate-card <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"cardNumber": "5510291234567890"}'</span>
</code></pre>

</div>



<p>Expected: <code>200 OK</code> with card brand, type, issuer name, and country.</p>

<p><strong>Test 2: A prepaid card</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/validate-card <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"cardNumber": "4023601234567890"}'</span>
</code></pre>

</div>



<p>Expected: <code>403 Forbidden</code> with <code>PREPAID_CARD_BLOCKED</code> code.</p>

<p><strong>Test 3: Missing card number</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/validate-card <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{}'</span>
</code></pre>

</div>



<p>Expected: <code>400 Bad Request</code> with a BIN digit range error.</p>

<p><strong>Test 4: Card number too short to produce a valid BIN</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>curl <span class="nt">-X</span> POST http://localhost:3000/validate-card <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Content-Type: application/json"</span> <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"cardNumber": "411"}'</span>
</code></pre>

</div>



<p>Expected: <code>400 Bad Request</code>. The extracted BIN is only 3 digits, which is below the required minimum of 6.</p>

<p><strong>Test 5: Card issued in a blocked country</strong></p>

<p>Use a BIN known to belong to an issuer outside your <code>ALLOWED_COUNTRIES</code> list.<br>
Expected: <code>403 Forbidden</code> with <code>COUNTRY_NOT_ALLOWED</code> code and the country alpha2 value in the response body.</p>

<blockquote>
<p><strong>How your middleware maps to a raw API call:</strong> If you were hitting the BIN lookup API directly, the request would look exactly like this:</p>


<pre class="highlight shell"><code>curl <span class="nt">-H</span> <span class="s2">"X-API-Key: bsl_7ffc440d90186fc5040d214bbed9e584686941e2e89d5168c137fe836e158a03"</span> <span class="se">\</span>
     <span class="nt">-H</span> <span class="s2">"X-User-ID: e65dadd2-4797-4f77-9316-fe535f9b9268"</span> <span class="se">\</span>
     <span class="s2">"https://api.binsearchlookup.com/lookup?bin=551029"</span>
</code></pre>


<p>Your middleware handles both headers automatically from <code>.env</code>. Your route handlers and your frontend never touch credentials directly. That is the entire point.</p>
</blockquote>

<p>Run all five tests before moving on. If any response is unexpected, re-read the middleware and double-check your <code>.env</code> file.</p>


<h2>
  
  
  Step 7: Add a Simple In-Memory Cache (Optional but Recommended)
</h2>

<p>You do not need to call the BIN API on every single request. A BIN is permanently tied to an issuer. It does not change. If the same BIN appears twice in a session or across requests within any window, serve it from cache.</p>

<p>Here is a lightweight in-memory cache using a plain JavaScript <code>Map</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// middleware/binLookup.js -- final version with caching</span>
<span class="kd">const</span> <span class="nx">axios</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">axios</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Simple in-memory cache: BIN string -&gt; API response data.</span>
<span class="c1">// In production, replace this with Redis and set a TTL of 24 hours.</span>
<span class="kd">const</span> <span class="nx">binCache</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Map</span><span class="p">();</span>

<span class="kd">function</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">cardNumber</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">cardNumber</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">cleaned</span> <span class="o">=</span> <span class="nx">cardNumber</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\D</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nx">cleaned</span><span class="p">.</span><span class="nf">substring</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">return</span> <span class="nx">bin</span><span class="p">;</span>
<span class="p">}</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">binLookupMiddleware</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">cardNumber</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">bin</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">8</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">BIN must be between 6 and 8 digits. Please provide a valid card number.</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="nx">req</span><span class="p">.</span><span class="nx">bin</span> <span class="o">=</span> <span class="nx">bin</span><span class="p">;</span>

  <span class="c1">// Serve from cache if available -- avoids a network call entirely</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">binCache</span><span class="p">.</span><span class="nf">has</span><span class="p">(</span><span class="nx">bin</span><span class="p">))</span> <span class="p">{</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">binCache</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">bin</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span> <span class="c1">// Useful for metrics and logging</span>
    <span class="k">return</span> <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">axios</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="dl">"</span><span class="s2">https://api.binsearchlookup.com/lookup</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">params</span><span class="p">:</span> <span class="p">{</span> <span class="na">bin</span><span class="p">:</span> <span class="nx">bin</span> <span class="p">},</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="dl">"</span><span class="s2">X-API-Key</span><span class="dl">"</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BIN_API_KEY</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">X-User-ID</span><span class="dl">"</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">X_USER_ID</span><span class="p">,</span>
        <span class="p">},</span>
        <span class="na">timeout</span><span class="p">:</span> <span class="mi">3000</span><span class="p">,</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="c1">// Store result in cache before attaching to req</span>
    <span class="nx">binCache</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">bin</span><span class="p">,</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">);</span>

    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">;</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
    <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">BIN lookup failed:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
    <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<p>For a production system, replace the <code>Map</code> with a Redis client and set a TTL of 24 hours. BIN data is stable, and Redis handles memory management, persistence, and expiry automatically.</p>




<h2>
  
  
  Step 8: Structure Your Project for the Long Term
</h2>

<p>Here is the final folder structure you should have:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>bin-middleware-demo/
  middleware/
    binLookup.js       # Fetches BIN data and attaches it to req.binInfo
  index.js             # Express server, route definitions, and business rules
  .env                 # API credentials and config (never commit this)
  .gitignore
  package.json
</code></pre>

</div>



<p>As your project grows, split the business rules into a separate <code>validators/cardPolicy.js</code> file and import it into the route handler. The middleware stays thin and focused on fetching. The policy layer stays focused on rules. Clean separation means easier unit testing and easier onboarding for every engineer who joins the project after you.</p>




<h2>
  
  
  Full Code Reference
</h2>

<p>Here is every file in its final state so you can copy, paste, and run without scrolling back through the steps.</p>

<h3>
  
  
  <code>.env</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>BIN_API_KEY=bsl_7ffc440d90186fc5040d214bbed9e584686941e2e89d5168c137fe836e158a03
X_USER_ID=e65dadd2-4797-4f77-9316-fe535f9b9268
PORT=3000
</code></pre>

</div>



<h3>
  
  
  <code>middleware/binLookup.js</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">axios</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">axios</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// In-memory cache to avoid redundant API calls for the same BIN.</span>
<span class="c1">// Replace with Redis in production and set a 24-hour TTL.</span>
<span class="kd">const</span> <span class="nx">binCache</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">Map</span><span class="p">();</span>

<span class="c1">// BIN must be between 6 and 8 digits.</span>
<span class="c1">// 6 digits is the minimum to identify a unique issuer.</span>
<span class="c1">// 8 digits is the maximum before you start encoding the account number.</span>
<span class="kd">function</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">)</span> <span class="p">{</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">cardNumber</span> <span class="o">||</span> <span class="k">typeof</span> <span class="nx">cardNumber</span> <span class="o">!==</span> <span class="dl">"</span><span class="s2">string</span><span class="dl">"</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="kd">const</span> <span class="nx">cleaned</span> <span class="o">=</span> <span class="nx">cardNumber</span><span class="p">.</span><span class="nf">replace</span><span class="p">(</span><span class="sr">/</span><span class="se">\D</span><span class="sr">/g</span><span class="p">,</span> <span class="dl">""</span><span class="p">);</span>
  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nx">cleaned</span><span class="p">.</span><span class="nf">substring</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span><span class="p">)</span> <span class="k">return</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">return</span> <span class="nx">bin</span><span class="p">;</span>
<span class="p">}</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="k">async</span> <span class="kd">function</span> <span class="nf">binLookupMiddleware</span><span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">,</span> <span class="nx">next</span><span class="p">)</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="p">{</span> <span class="nx">cardNumber</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">;</span>

  <span class="kd">const</span> <span class="nx">bin</span> <span class="o">=</span> <span class="nf">extractBin</span><span class="p">(</span><span class="nx">cardNumber</span><span class="p">);</span>

  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">bin</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&lt;</span> <span class="mi">6</span> <span class="o">||</span> <span class="nx">bin</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">8</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">400</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">BIN must be between 6 and 8 digits. Please provide a valid card number.</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Attach the raw BIN for downstream reference</span>
  <span class="nx">req</span><span class="p">.</span><span class="nx">bin</span> <span class="o">=</span> <span class="nx">bin</span><span class="p">;</span>

  <span class="c1">// Return cached result if available</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">binCache</span><span class="p">.</span><span class="nf">has</span><span class="p">(</span><span class="nx">bin</span><span class="p">))</span> <span class="p">{</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">binCache</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">bin</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
    <span class="k">return</span> <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="k">try</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">response</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">axios</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span>
      <span class="dl">"</span><span class="s2">https://api.binsearchlookup.com/lookup</span><span class="dl">"</span><span class="p">,</span>
      <span class="p">{</span>
        <span class="na">params</span><span class="p">:</span> <span class="p">{</span> <span class="na">bin</span><span class="p">:</span> <span class="nx">bin</span> <span class="p">},</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
          <span class="c1">// X-API-Key and X-User-ID are both required by the API</span>
          <span class="dl">"</span><span class="s2">X-API-Key</span><span class="dl">"</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">BIN_API_KEY</span><span class="p">,</span>
          <span class="dl">"</span><span class="s2">X-User-ID</span><span class="dl">"</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">X_USER_ID</span><span class="p">,</span>
        <span class="p">},</span>
        <span class="na">timeout</span><span class="p">:</span> <span class="mi">3000</span><span class="p">,</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="nx">binCache</span><span class="p">.</span><span class="nf">set</span><span class="p">(</span><span class="nx">bin</span><span class="p">,</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">response</span><span class="p">.</span><span class="nx">data</span><span class="p">;</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
    <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span> <span class="k">catch </span><span class="p">(</span><span class="nx">error</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// Soft-fail: log internally, do not expose API details to the client</span>
    <span class="nx">console</span><span class="p">.</span><span class="nf">error</span><span class="p">(</span><span class="dl">"</span><span class="s2">BIN lookup failed:</span><span class="dl">"</span><span class="p">,</span> <span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
    <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
    <span class="nf">next</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">};</span>
</code></pre>

</div>



<h3>
  
  
  <code>index.js</code>
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">dotenv</span><span class="dl">"</span><span class="p">).</span><span class="nf">config</span><span class="p">();</span>

<span class="kd">const</span> <span class="nx">express</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">express</span><span class="dl">"</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">app</span> <span class="o">=</span> <span class="nf">express</span><span class="p">();</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">use</span><span class="p">(</span><span class="nx">express</span><span class="p">.</span><span class="nf">json</span><span class="p">());</span>

<span class="kd">const</span> <span class="nx">binLookupMiddleware</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">./middleware/binLookup</span><span class="dl">"</span><span class="p">);</span>

<span class="c1">// Cards issued outside this list will be blocked</span>
<span class="kd">const</span> <span class="nx">ALLOWED_COUNTRIES</span> <span class="o">=</span> <span class="p">[</span><span class="dl">"</span><span class="s2">US</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">CA</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">GB</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">AU</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">DE</span><span class="dl">"</span><span class="p">,</span> <span class="dl">"</span><span class="s2">FR</span><span class="dl">"</span><span class="p">];</span>

<span class="nx">app</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="dl">"</span><span class="s2">/validate-card</span><span class="dl">"</span><span class="p">,</span> <span class="nx">binLookupMiddleware</span><span class="p">,</span> <span class="p">(</span><span class="nx">req</span><span class="p">,</span> <span class="nx">res</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">binInfo</span> <span class="o">=</span> <span class="nx">req</span><span class="p">.</span><span class="nx">binInfo</span><span class="p">;</span>

  <span class="c1">// BIN lookup failed (API outage or unknown BIN) -- block and ask to retry</span>
  <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">binInfo</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">422</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Unable to verify card origin. Please try again.</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Rule 1: Block prepaid cards</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">binInfo</span><span class="p">.</span><span class="nx">prepaid</span> <span class="o">===</span> <span class="kc">true</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">403</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Prepaid cards are not accepted for this service.</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="dl">"</span><span class="s2">PREPAID_CARD_BLOCKED</span><span class="dl">"</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Rule 2: Block cards from unsupported issuer countries</span>
  <span class="kd">const</span> <span class="nx">issuerCountry</span> <span class="o">=</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">country</span><span class="p">?.</span><span class="nx">alpha2</span><span class="p">;</span>
  <span class="k">if </span><span class="p">(</span><span class="nx">issuerCountry</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="nx">ALLOWED_COUNTRIES</span><span class="p">.</span><span class="nf">includes</span><span class="p">(</span><span class="nx">issuerCountry</span><span class="p">))</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">status</span><span class="p">(</span><span class="mi">403</span><span class="p">).</span><span class="nf">json</span><span class="p">({</span>
      <span class="na">success</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">error</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Cards from this region are not supported.</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">code</span><span class="p">:</span> <span class="dl">"</span><span class="s2">COUNTRY_NOT_ALLOWED</span><span class="dl">"</span><span class="p">,</span>
      <span class="na">country</span><span class="p">:</span> <span class="nx">issuerCountry</span><span class="p">,</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// All checks passed</span>
  <span class="k">return</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">({</span>
    <span class="na">success</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">bin</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">bin</span><span class="p">,</span>
    <span class="na">cardBrand</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">brand</span><span class="p">,</span>
    <span class="na">cardType</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">type</span><span class="p">,</span>
    <span class="na">issuer</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">issuer</span><span class="p">?.</span><span class="nx">name</span><span class="p">,</span>
    <span class="na">country</span><span class="p">:</span> <span class="nx">issuerCountry</span><span class="p">,</span>
    <span class="na">prepaid</span><span class="p">:</span> <span class="nx">binInfo</span><span class="p">.</span><span class="nx">prepaid</span><span class="p">,</span>
    <span class="na">cacheHit</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">binCacheHit</span><span class="p">,</span>
    <span class="na">message</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Card validated successfully.</span><span class="dl">"</span><span class="p">,</span>
  <span class="p">});</span>
<span class="p">});</span>

<span class="kd">const</span> <span class="nx">PORT</span> <span class="o">=</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">PORT</span> <span class="o">||</span> <span class="mi">3000</span><span class="p">;</span>
<span class="nx">app</span><span class="p">.</span><span class="nf">listen</span><span class="p">(</span><span class="nx">PORT</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nx">console</span><span class="p">.</span><span class="nf">log</span><span class="p">(</span><span class="s2">`Server running on port </span><span class="p">${</span><span class="nx">PORT</span><span class="p">}</span><span class="s2">`</span><span class="p">);</span>
<span class="p">});</span>
</code></pre>

</div>






<h2>
  
  
  Summary
</h2>

<p>You built a production-quality BIN lookup middleware in eight steps. Here is what you now have:</p>

<ul>
<li>A reusable <code>binLookupMiddleware</code> function that extracts the BIN, enforces the strict 6-to-8 digit rule, calls the lookup API with both required credentials, and attaches structured data to <code>req.binInfo</code>.</li>
<li>Both <code>BIN_API_KEY</code> and <code>X_USER_ID</code> stored safely in <code>.env</code> and passed as <code>X-API-Key</code> and <code>X-User-ID</code> headers on every API request.</li>
<li>A <code>/validate-card</code> endpoint that enforces prepaid card and country-of-origin rules using that metadata.</li>
<li>Graceful soft-fail handling so an API outage does not break your checkout flow.</li>
<li>An in-memory cache to eliminate redundant network calls for repeated BINs.</li>
<li>Clean separation between the data-fetching middleware and the business rule layer in the route handler.</li>
</ul>

<p>The pattern you learned here applies beyond BIN lookups. Any time you need to enrich an incoming request with external data before your main handler runs, middleware is the right tool. IP geolocation, device fingerprinting, user reputation scoring: all of them follow exactly this shape.</p>




<h2>
  
  
  FAQ
</h2>

<p><strong>What is a BIN number and why does it matter for payments?</strong></p>

<p>BIN stands for Bank Identification Number. It is the first 6 to 8 digits of a payment card number and it identifies the issuing bank, the card network (Visa, Mastercard, Amex), the card type (credit, debit, prepaid), and the country where the card was issued. Payment platforms use BIN data to run fraud checks, apply geographic restrictions, and display the correct card brand logo at checkout.</p>

<p><strong>Why does the BIN value sent to the API have to be between 6 and 8 digits?</strong></p>

<p>The BIN lookup database is indexed on 6-digit and 8-digit prefixes. Fewer than 6 digits does not uniquely identify an issuer. More than 8 digits begins encoding the individual account number, which is sensitive cardholder data that you must never send to any third-party service. The <code>extractBin</code> function enforces this range before the network call is ever made.</p>

<p><strong>What are X-API-Key and X-User-ID and why are both required?</strong></p>

<p><code>X-API-Key</code> authenticates your application to the BIN lookup API. <code>X-User-ID</code> identifies the specific account making the request, which allows the API provider to apply per-account rate limits, usage tracking, and billing. Both are assigned when you create an account at binsearchlookup.com. Storing them in <code>.env</code> and reading them with <code>process.env</code> keeps them out of your source code and out of version control.</p>

<p><strong>Is it safe to log the full card number on the server?</strong></p>

<p>No. You should never log a full card number. Extract only the BIN immediately and discard or tokenize the rest. The BIN alone is not sensitive because it does not identify the cardholder or authorize a transaction. Log the BIN, not the card number.</p>

<p><strong>What happens if the BIN API is down?</strong></p>

<p>The middleware uses a soft-fail pattern. If the API call throws an error or times out, <code>req.binInfo</code> is set to <code>null</code> and <code>next()</code> is called. Your route handler checks for <code>null</code> and decides the fallback policy. The example in this tutorial returns a 422 and asks the user to try again. You could also allow the transaction through with reduced confidence scoring. The decision belongs in the route handler, not in the middleware.</p>

<p><strong>Can I use this with Stripe or another payment processor?</strong></p>

<p>Yes. This middleware runs before you ever call Stripe. You enrich the request object first, apply your rules, and only then initiate the Stripe payment intent if everything passes. The BIN check adds a validation layer that sits entirely on your own server.</p>

<p><strong>Should I use 6-digit or 8-digit BINs?</strong></p>

<p>Use 8-digit BINs when the card number is long enough to produce them. The payments industry has been migrating toward 8-digit BINs since 2022, and modern BIN lookup APIs return more accurate results with 8 digits. The <code>extractBin</code> function in this tutorial uses <code>substring(0, 8)</code> to grab as many digits as possible up to the 8-digit maximum, then validates that at least 6 were available.</p>

<p><strong>How do I extend the middleware to handle debit cards differently?</strong></p>

<p>Read <code>req.binInfo.type</code> in your route handler. It will be <code>"CREDIT"</code>, <code>"DEBIT"</code>, or <code>"PREPAID"</code>. Write conditional logic for each type without touching the middleware at all. That is the benefit of keeping data fetching and policy enforcement in separate layers.</p>

<p><strong>What is the right timeout for a BIN lookup API call?</strong></p>

<p>A 2 to 3 second timeout is reasonable. BIN lookups are fast because the data is static and well-indexed. If your upstream API consistently takes longer than 1 second, check whether you are hitting a rate limit or whether a geographically closer endpoint is available.</p>




<p><em>If this tutorial helped you, share what rules you are enforcing in your own BIN middleware in the comments. The more specific your use case, the more others in the community can learn from it.</em></p>

