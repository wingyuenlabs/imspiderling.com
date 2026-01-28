---
Title: Solved: Tracking API Rate Limits: Middleware Solution for Express.js/Python
Description: 
Author: Darian Vance
Date: 2026-01-28T20:59:53.000Z
Robots: noindex,nofollow
Template: index
---
<h3>
  
  
  🚀 Executive Summary
</h3>

<p><strong>TL;DR:</strong> Uncontrolled third-party API usage often results in <code>429 Too Many Requests</code> errors and service disruptions. This article presents a proactive, centralized rate limit tracking solution using Redis, implemented as middleware in Express.js and a decorator in Python FastAPI, to monitor API usage and gracefully pause before hitting limits.</p>

<h4>
  
  
  🎯 Key Takeaways
</h4>

<ul>
<li>Proactive API rate limit tracking using Redis prevents <code>429</code> errors by checking available quota before making external calls, rather than reacting to failures.</li>
<li>The solution leverages Redis as a high-performance, centralized store for shared rate limit status, crucial for maintaining consistency across distributed application instances.</li>
<li>Rate limit updates are dynamically managed by extracting <code>x-ratelimit-remaining</code> and <code>x-ratelimit-reset</code> headers from external API responses, setting the remaining count in Redis with an appropriate Time To Live (TTL).</li>
</ul>

<h1>
  
  
  Tracking API Rate Limits: Middleware Solution for Express.js/Python
</h1>

<h2>
  
  
  Introduction
</h2>

<p>In the world of interconnected services, APIs are the glue that holds our applications together. However, every seasoned engineer has faced the dreaded <code>429 Too Many Requests</code> error. Relying on third-party APIs without respecting their rate limits is a recipe for disaster, leading to service disruptions, cascading failures, and even temporary account suspension.</p>

<p>The common approach is reactive: you make a call, get a <code>429</code> error, and then back off. This is inefficient and makes your application brittle. A far more robust solution is to be proactive. Instead of waiting to be told you’ve hit a limit, what if your application could keep track of its own usage and gracefully pause before ever receiving an error?</p>

<p>In this tutorial, we will build exactly that: a proactive, centralized rate limit tracking solution using Redis. We’ll implement this pattern as a middleware in an Express.js application and as a decorator in a Python FastAPI application. This approach is perfect for distributed systems where multiple instances of your service share the same API quota.</p>

<h2>
  
  
  Prerequisites
</h2>

<p>Before we begin, ensure you have the following tools and knowledge:</p>

<ul>
<li>Node.js (v16+) and npm or yarn installed.</li>
<li>Python (v3.8+) and pip installed.</li>
<li>Docker and Docker Compose for running a local Redis instance.</li>
<li>A basic understanding of REST APIs and HTTP headers.</li>
<li>Familiarity with either Express.js or a Python web framework like FastAPI or Flask.</li>
<li>Access to a command-line terminal.</li>
</ul>

<h2>
  
  
  Step-by-Step Guide
</h2>

<h3>
  
  
  Step 1: Setting Up the Shared State with Redis
</h3>

<p>Our tracking mechanism needs a central, fast, and reliable place to store the current rate limit status. Redis is the perfect tool for this job due to its high-performance, in-memory nature and atomic operations. We’ll use Docker Compose to spin up a Redis container quickly.</p>

<p>Create a file named <code>docker-compose.yml</code> in your project directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># docker-compose.yml
version: '3.8'
services:
  redis:
    image: "redis:alpine"
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  redis_data:
</code></pre>

</div>



<p>This configuration defines a service named <code>redis</code> that uses the official lightweight Alpine image. It maps the container’s port 6379 to your local machine’s port 6379 and creates a Docker volume to persist data.</p>

<p>Now, start the container in detached mode from your terminal:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Bash Command
docker-compose up -d
</code></pre>

</div>



<p>You can verify that Redis is running by connecting to it with the Redis CLI (if installed locally) or through another Docker command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Bash Command
docker exec -it [CONTAINER_ID] redis-cli ping
# Expected output: PONG
</code></pre>

</div>



<p>With our shared state manager running, we’re ready to build the application logic.</p>

<h3>
  
  
  Step 2: Implementing the Rate Limit Tracker in Express.js
</h3>

<p>For our Node.js example, we’ll create an Express middleware. This middleware will intercept requests that need to call an external API. It will first check Redis for the available quota before proceeding. After the external call, it will read the rate limit headers from the response and update Redis accordingly.</p>

<p>First, initialize a Node.js project and install the necessary dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Bash Commands
npm init -y
npm install express redis axios
</code></pre>

</div>



<p>Now, create a file named <code>rateLimitTracker.js</code> for our middleware logic:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// rateLimitTracker.js
const { createClient } = require('redis');
const axios = require('axios');

const redisClient = createClient();
redisClient.on('error', (err) =&gt; console.log('Redis Client Error', err));

// A unique key for tracking a specific API endpoint.
const GITHUB_API_LIMIT_KEY = 'rate-limit:github-api';

const rateLimitTracker = async (req, res, next) =&gt; {
  if (!redisClient.isOpen) {
    await redisClient.connect();
  }

  try {
    const remaining = await redisClient.get(GITHUB_API_LIMIT_KEY);

    // If the key exists and the count is 0, we've hit the limit.
    if (remaining !== null &amp;&amp; Number(remaining) &lt;= 0) {
      return res.status(429).json({ 
        message: 'GitHub API rate limit exceeded. Please try again later.' 
      });
    }

    // Attach a function to the response object to update the limit after the request.
    res.updateRateLimit = async (apiResponse) =&gt; {
      const newRemaining = apiResponse.headers['x-ratelimit-remaining'];
      const resetTimestamp = apiResponse.headers['x-ratelimit-reset'];

      if (newRemaining !== undefined &amp;&amp; resetTimestamp !== undefined) {
        const ttl = Number(resetTimestamp) - Math.floor(Date.now() / 1000);
        // Set the new value with an expiry time (Time To Live).
        await redisClient.set(GITHUB_API_LIMIT_KEY, newRemaining, { EX: ttl &gt; 0 ? ttl : 1 });
        console.log(`Updated Redis: Remaining calls = ${newRemaining}, Resets in ${ttl}s`);
      }
    };

    next();
  } catch (error) {
    console.error('Error in rate limit middleware:', error);
    // Fail open: if Redis fails, let the request proceed.
    next();
  }
};

module.exports = rateLimitTracker;
</code></pre>

</div>



<p>The key logic here is the <code>res.updateRateLimit</code> function. We attach it to the response object so our main route handler can call it <em>after</em> the external API call succeeds, passing in the response from that call.</p>

<p>Next, let’s integrate this into an Express server. Create a file named <code>server.js</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// server.js
const express = require('express');
const axios = require('axios');
const rateLimitTracker = require('./rateLimitTracker');

const app = express();
const PORT = 3000;

// Apply the middleware to the route that calls the external API.
app.get('/github/user/:username', rateLimitTracker, async (req, res) =&gt; {
  try {
    const { username } = req.params;
    const githubResponse = await axios.get(`https://api.github.com/users/${username}`);

    // Call the function attached by our middleware to update the limit.
    if (res.updateRateLimit) {
      res.updateRateLimit(githubResponse);
    }

    res.json(githubResponse.data);
  } catch (error) {
    // Also update the limit on failure, as a failed request might still count against the quota.
    if (error.response &amp;&amp; res.updateRateLimit) {
      res.updateRateLimit(error.response);
    }
    const status = error.response ? error.response.status : 500;
    const message = error.response ? error.response.data : 'Internal Server Error';
    res.status(status).json({ message });
  }
});

app.listen(PORT, () =&gt; {
  console.log(`Express server running on http://localhost:${PORT}`);
});
</code></pre>

</div>



<p>When you call <code>/github/user/someuser</code>, our middleware checks Redis. If the quota is available, it proceeds. The route handler calls the GitHub API and then uses <code>res.updateRateLimit</code> to save the latest rate limit information back to Redis.</p>

<h3>
  
  
  Step 3: Implementing the Rate Limit Tracker in Python (FastAPI)
</h3>

<p>The principle in Python is identical. We’ll use a dependency injection approach with FastAPI, which provides a clean and reusable way to encapsulate our logic.</p>

<p>First, set up a virtual environment and install the dependencies:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Bash Commands
python3 -m venv venv
source venv/bin/activate
pip install fastapi "uvicorn[standard]" redis httpx
</code></pre>

</div>



<p>Now, create a file named <code>main.py</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># main.py
import redis.asyncio as redis
import httpx
import time
from fastapi import FastAPI, Depends, HTTPException, Response

# --- Configuration &amp; Clients ---
app = FastAPI()
redis_client = redis.from_url("redis://localhost:6379", decode_responses=True)
GITHUB_API_LIMIT_KEY = "rate-limit:github-api"

# --- Rate Limiter Logic as a Dependency ---
async def github_rate_limiter(response: Response):
    """
    A FastAPI dependency that checks and updates API rate limits.
    """
    remaining_str = await redis_client.get(GITHUB_API_LIMIT_KEY)

    if remaining_str is not None and int(remaining_str) &lt;= 0:
        raise HTTPException(
            status_code=429,
            detail="GitHub API rate limit exceeded. Please try again later."
        )

    # Yield control to the route handler. Code after this runs after the response is sent.
    yield

    # This part runs after the request has been processed. We use the response object
    # that FastAPI gives us access to.
    api_headers = response.context.get("api_headers") if hasattr(response, "context") else None
    if api_headers:
        new_remaining = api_headers.get("x-ratelimit-remaining")
        reset_timestamp = api_headers.get("x-ratelimit-reset")

        if new_remaining is not None and reset_timestamp is not None:
            ttl = int(reset_timestamp) - int(time.time())
            await redis_client.set(
                GITHUB_API_LIMIT_KEY, 
                new_remaining, 
                ex=ttl if ttl &gt; 0 else 1
            )
            print(f"Updated Redis: Remaining calls = {new_remaining}, Resets in {ttl}s")

# --- API Route ---
@app.get("/github/user/{username}", dependencies=[Depends(github_rate_limiter)])
async def get_github_user(username: str, response: Response):
    async with httpx.AsyncClient() as client:
        try:
            github_res = await client.get(f"https://api.github.com/users/{username}")
            github_res.raise_for_status() # Raise an exception for 4xx/5xx responses

            # Store headers in the response context to be read by the dependency later.
            response.context = {"api_headers": github_res.headers}
            return github_res.json()

        except httpx.HTTPStatusError as e:
            # Also update the rate limit on error.
            response.context = {"api_headers": e.response.headers}
            raise HTTPException(
                status_code=e.response.status_code, 
                detail=e.response.json()
            )
</code></pre>

</div>



<p>In this FastAPI example, <code>github_rate_limiter</code> is a “dependency” function. FastAPI executes the code before the <code>yield</code> statement before running the route handler. After the route handler completes, the code after <code>yield</code> is executed. This is a powerful pattern for setup and teardown logic. We pass headers from the route handler back to the dependency using a shared <code>response.context</code> object.</p>

<p>Run the application with Uvicorn:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Bash Command
uvicorn main:app --reload
</code></pre>

</div>



<h2>
  
  
  Common Pitfalls
</h2>

<h3>
  
  
  1. Race Conditions in a Distributed Environment
</h3>

<p>Imagine two application instances running in parallel. Both read the same value from Redis (e.g., “1 remaining call”). Both proceed to make an API call, using up two requests when they thought only one was available. While our header-based approach mitigates this by always trusting the server’s response, simpler “decrement-on-call” logic is vulnerable.</p>

<p><strong>Solution:</strong> For simple decrementing counters, use Redis’s atomic operations like <code>DECR</code> or <code>DECRBY</code>. These commands guarantee that the decrement operation is performed as a single, uninterruptible step, preventing race conditions. For our header-based solution, the main risk is clock skew. Always add a small buffer (e.g., 5-10 seconds) to the TTL you calculate from the reset timestamp to account for network latency and minor clock differences.</p>

<h3>
  
  
  2. Forgetting to Handle the “Limit Reached” Case Gracefully
</h3>

<p>Our code correctly returns a <code>429</code> error when the limit is reached, but what should the client do next? Simply failing the user’s request might not be the best user experience.</p>

<p><strong>Solution:</strong> Implement a more sophisticated strategy. Instead of immediately failing, you could push the request into a queue (like RabbitMQ or a Redis list) with a delayed execution time. A separate worker process can then retry these jobs after the rate limit has reset. This turns a hard failure into a temporary delay, creating a much more resilient system.</p>

<h2>
  
  
  Conclusion
</h2>

<p>By proactively tracking API rate limits, you transform your application from a reactive victim of API constraints to a resilient and well-behaved citizen of the microservices ecosystem. We’ve demonstrated how to build a robust, centralized tracking system using Redis for both Express.js and Python FastAPI applications. This pattern not only prevents <code>429</code> errors but also improves reliability and provides a foundation for more advanced features like request queuing and dynamic backoff strategies.</p>

<p>This foundational concept can be extended to track multiple API endpoints, handle different rate limit policies, and integrate into your observability stack for better monitoring. Start implementing this in your services today to build more stable and predictable applications.</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwjgilechjb8hqoqlrg1.png" alt="Darian Vance" width="758" height="289"></a></p>

<p>👉 <a href="https://wp.me/pbK4oa-bp" rel="noopener noreferrer">Read the original article on TechResolve.blog</a></p>




<p>☕ <strong>Support my work</strong>  </p>

<p>If this article helped you, you can buy me a coffee:  </p>

<p>👉 <a href="https://buymeacoffee.com/darianvance" rel="noopener noreferrer">https://buymeacoffee.com/darianvance</a></p>

