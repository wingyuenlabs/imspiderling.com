---
Title: Building Synapse: A Full-Featured Social Media App with React Native & Supabase (Launching Soon!)
Description: 
Author: Alex
Date: 2025-12-23T21:41:53.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  What is Synapse?
</h2>

<p>Synapse is a social media platform designed for meaningful connections. Think X/Threads, but with support for rich content blocks, mathematical equations, audio posts, files, and a privacy-first approach and much more.</p>

<p><strong>Key Features:</strong></p>

<ul>
<li>Multi-block posts (text, images, videos, audio, files, live photos)</li>
<li>Markdown with syntax highlighting and MathJax support</li>
<li>Feed recommendations</li>
<li>Real-time direct messaging</li>
<li>Full-text search</li>
</ul>

<h2>
  
  
  Why We Built This
</h2>

<p>The idea for Synapse came from a frustration that many researchers and developers share: <strong>existing social platforms aren't built for technical discourse.</strong></p>

<h3>
  
  
  The Problems We Kept Running Into
</h3>

<p><strong>1. Fragmented Discussions</strong></p>

<p>Try following a machine learning research discussion on X (Twitter). A single paper breakdown gets split into 15+ tweets in a thread. Equations are screenshots. Code is images. Context gets lost in quote tweets and replies. It's chaos.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1e5mtbunf6w1l316kkbj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1e5mtbunf6w1l316kkbj.png" alt="Three apps are needed to read the whole content" width="800" height="605"></a></p>

<p><strong>2. No Built-in Support for Rich Content</strong></p>

<p>Want to share a PDF of your paper? Upload an audio explanation of your code? Attach a dataset file? You can't. Current platforms treat everything as text + images/video, forcing researchers and developers to link out to external services for anything beyond basic media.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F61uru26rpkjr04r6lark.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F61uru26rpkjr04r6lark.png" alt="The way research is shared on X" width="800" height="379"></a></p>

<p><strong>3. Discovery is Broken</strong></p>

<p>Finding the right people to follow in your niche is surprisingly hard. Algorithms optimize for engagement, not relevance. There's no good way to discover researchers in your specific subfield or developers working on similar problems.</p>

<h3>
  
  
  The Vision
</h3>

<p>We wanted a platform where you could write a single, cohesive post that includes:</p>

<ul>
<li>Your explanation in markdown with proper formatting</li>
<li>Mathematical equations rendered beautifully (LaTeX/MathJax)</li>
<li>Code snippets with syntax highlighting</li>
<li>The actual PDF or dataset file</li>
<li>An audio walkthrough for those who prefer listening</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnjmb03h2qi2cp3ma4c1q.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnjmb03h2qi2cp3ma4c1q.png" alt="A post on Synapse - all in one" width="800" height="450"></a></p>

<p>All in one place. No threads. No external links. No screenshots of equations.</p>

<h2>
  
  
  Who Is This For?
</h2>

<p>We're building Synapse primarily for two communities:</p>

<h3>
  
  
  Researchers &amp; Academics
</h3>

<p>Scientists who want to share their work, discuss papers, and connect with peers in their field. Whether you're in ML, physics, biology, or any technical discipline, you deserve a platform that speaks your language - literally, with native LaTeX support.</p>

<h3>
  
  
  Developers &amp; Technical Creators
</h3>

<p>Programmers who want to share ideas, tutorials, and projects in a beautiful, readable format. Write technical content with proper code highlighting, attach relevant files, and build an audience that cares about substance over virality.</p>

<p>We believe the best ideas deserve better than a character limit and a thread of fragmented thoughts.</p>

<h2>
  
  
  The Tech Stack
</h2>

<h3>
  
  
  Frontend
</h3>

<ul>
<li>
<strong>React Native 0.81</strong> with the new architecture enabled</li>
<li>
<strong>Expo SDK 54</strong> for rapid development and OTA updates</li>
<li>
<strong>React 19</strong> with the latest concurrent features</li>
<li>
<strong>TypeScript</strong> in strict mode (no <code>any</code> allowed!)</li>
</ul>

<h3>
  
  
  State Management
</h3>

<p>We went with a hybrid approach:</p>

<ul>
<li>
<strong>React Query 5</strong> for server state and caching</li>
<li>
<strong>Zustand</strong> for client-side stores</li>
<li>
<strong>Context API</strong> for global app state (Auth, Audio, Analytics)</li>
</ul>

<h3>
  
  
  Backend
</h3>

<ul>
<li>
<strong>Supabase</strong> as our Backend-as-a-Service

<ul>
<li>PostgreSQL for the database</li>
<li>Supabase Auth with email OTP</li>
<li>Supabase Storage (S3-compatible) for media</li>
<li>Realtime subscriptions for live updates</li>
<li>Edge Functions (Deno) for serverless compute</li>
</ul>


</li>

</ul>

<h2>
  
  
  Technical Deep Dives
</h2>

<h3>
  
  
  1. The Content Block System
</h3>

<p>Instead of storing posts as flat text, we designed a flexible block-based architecture:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">type</span> <span class="nx">ContentBlock</span> <span class="o">=</span> <span class="nx">TextBlock</span> <span class="o">|</span> <span class="nx">MediaBlock</span> <span class="o">|</span> <span class="nx">AudioBlock</span> <span class="o">|</span> <span class="nx">FileBlock</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">MediaBlock</span> <span class="p">{</span>
  <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">media</span><span class="dl">'</span><span class="p">;</span>
  <span class="nl">items</span><span class="p">:</span> <span class="nx">MediaItem</span><span class="p">[];</span>
<span class="p">}</span>

<span class="kr">interface</span> <span class="nx">MediaItem</span> <span class="p">{</span>
  <span class="nl">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">image</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">video</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">live_photo</span><span class="dl">'</span><span class="p">;</span>
  <span class="nl">storagePath</span><span class="p">:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="nl">width</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">height</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="nl">thumbnailUrl</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
  <span class="c1">// Live photos have separate video/photo URLs</span>
  <span class="nl">videoStoragePath</span><span class="p">?:</span> <span class="kr">string</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Why this matters:</strong></p>

<ul>
<li>Posts can contain mixed content types</li>
<li>Each block handles its own rendering logic</li>
<li>Easy to add new block types in the future</li>
<li>Type-safe with runtime type guards</li>
</ul>

<h3>
  
  
  2. Optimistic Messaging
</h3>

<p>Nobody wants to wait for a server response to see their message. We implemented optimistic updates with local IDs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Generate local ID for immediate display</span>
<span class="kd">const</span> <span class="nx">localId</span> <span class="o">=</span> <span class="s2">`local_</span><span class="p">${</span><span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">()}</span><span class="s2">_</span><span class="p">${</span><span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">().</span><span class="nf">toString</span><span class="p">(</span><span class="mi">36</span><span class="p">)}</span><span class="s2">`</span><span class="p">;</span>

<span class="c1">// Show message immediately with "pending" status</span>
<span class="nf">addMessageToCache</span><span class="p">({</span>
  <span class="na">id</span><span class="p">:</span> <span class="nx">localId</span><span class="p">,</span>
  <span class="nx">content</span><span class="p">,</span>
  <span class="na">status</span><span class="p">:</span> <span class="dl">'</span><span class="s1">pending</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">created_at</span><span class="p">:</span> <span class="k">new</span> <span class="nc">Date</span><span class="p">().</span><span class="nf">toISOString</span><span class="p">()</span>
<span class="p">});</span>

<span class="c1">// Server confirms and provides real ID</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">supabase</span>
  <span class="p">.</span><span class="k">from</span><span class="p">(</span><span class="dl">'</span><span class="s1">messages</span><span class="dl">'</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">insert</span><span class="p">({</span> <span class="nx">content</span><span class="p">,</span> <span class="nx">conversation_id</span> <span class="p">})</span>
  <span class="p">.</span><span class="nf">select</span><span class="p">()</span>
  <span class="p">.</span><span class="nf">single</span><span class="p">();</span>

<span class="c1">// Reconcile local and server states</span>
<span class="nf">replaceLocalMessage</span><span class="p">(</span><span class="nx">localId</span><span class="p">,</span> <span class="nx">data</span><span class="p">);</span>
</code></pre>

</div>



<p>The tricky part? Handling edge cases:</p>

<ul>
<li>What if the server response arrives before the optimistic update?</li>
<li>What if the user sends multiple messages quickly?</li>
<li>How do we handle failures gracefully?</li>
</ul>

<p>We built an <code>OptimisticMessageManager</code> that handles all these scenarios with proper reconciliation logic.</p>

<h3>
  
  
  3. Signed URLs with Smart Caching
</h3>

<p>Media files in Supabase Storage require signed URLs for private access. The challenge: these URLs expire!</p>

<p>Our solution:</p>

<ul>
<li>Cache signed URLs with TTL tracking</li>
<li>Refresh URLs 5 minutes before expiration</li>
<li>Batch URL requests to reduce API calls</li>
<li>Propagate <code>postId</code> context for cache invalidation
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">class</span> <span class="nc">SignedUrlService</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">cache</span> <span class="o">=</span> <span class="k">new</span> <span class="nb">Map</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">,</span> <span class="nx">CachedUrl</span><span class="o">&gt;</span><span class="p">();</span>

  <span class="k">async</span> <span class="nf">getSignedUrl</span><span class="p">(</span><span class="nx">storagePath</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="nx">postId</span><span class="p">?:</span> <span class="kr">string</span><span class="p">):</span> <span class="nb">Promise</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">cacheKey</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nf">buildCacheKey</span><span class="p">(</span><span class="nx">storagePath</span><span class="p">,</span> <span class="nx">postId</span><span class="p">);</span>
    <span class="kd">const</span> <span class="nx">cached</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">cache</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nx">cacheKey</span><span class="p">);</span>

    <span class="k">if </span><span class="p">(</span><span class="nx">cached</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="k">this</span><span class="p">.</span><span class="nf">isExpiringSoon</span><span class="p">(</span><span class="nx">cached</span><span class="p">))</span> <span class="p">{</span>
      <span class="k">return</span> <span class="nx">cached</span><span class="p">.</span><span class="nx">url</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nf">fetchAndCache</span><span class="p">(</span><span class="nx">storagePath</span><span class="p">,</span> <span class="nx">postId</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  4. Feed Recommendation Engine
</h3>

<p>We built a server-side recommendation RPC that scores posts using multiple factors:</p>

<ul>
<li>Recency</li>
<li>Engagement metrics</li>
<li>User's social graph</li>
<li>Content type preferences
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// Edge Function: feed-recommendation</span>
<span class="kd">const</span> <span class="nx">candidates</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetchCandidates</span><span class="p">(</span><span class="nx">userId</span><span class="p">,</span> <span class="nx">excludeIds</span><span class="p">);</span>
<span class="kd">const</span> <span class="nx">scored</span> <span class="o">=</span> <span class="nx">candidates</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">post</span> <span class="o">=&gt;</span> <span class="p">({</span>
  <span class="p">...</span><span class="nx">post</span><span class="p">,</span>
  <span class="na">score</span><span class="p">:</span> <span class="nf">calculateRelevanceScore</span><span class="p">(</span><span class="nx">post</span><span class="p">,</span> <span class="nx">userPreferences</span><span class="p">)</span>
<span class="p">}));</span>

<span class="k">return</span> <span class="nx">scored</span>
  <span class="p">.</span><span class="nf">sort</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">score</span> <span class="o">-</span> <span class="nx">a</span><span class="p">.</span><span class="nx">score</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">slice</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="nx">limit</span><span class="p">);</span>
</code></pre>

</div>



<p>Users can toggle between chronological and recommended feeds.</p>

<h3>
  
  
  5. Audio Recording with Gesture Detection
</h3>

<p>We built custom <code>HoldToRecordButton</code> and <code>HoldToRecordOverlay</code> components:</p>

<ul>
<li>Long press starts recording</li>
<li>Drag to cancel zone aborts</li>
<li>Visual waveform feedback</li>
<li>Haptic feedback on state changes</li>
</ul>

<p>This required deep integration with React Native Gesture Handler and Expo Audio.</p>

<h2>
  
  
  Lessons Learned
</h2>

<h3>
  
  
  1. Start with TypeScript Strict Mode
</h3>

<p>We enabled strict mode from day one. It caught countless bugs during development that would have been runtime errors otherwise. The upfront investment pays dividends.</p>

<h3>
  
  
  2. Server-Side Logic &gt; Client-Side Complexity
</h3>

<p>Instead of managing complex state on the client (like cascade deletes when blocking a user), we use database triggers:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">CREATE</span> <span class="k">TRIGGER</span> <span class="n">on_user_blocked</span>
<span class="k">AFTER</span> <span class="k">INSERT</span> <span class="k">ON</span> <span class="n">blocked_users</span>
<span class="k">FOR</span> <span class="k">EACH</span> <span class="k">ROW</span>
<span class="k">EXECUTE</span> <span class="k">FUNCTION</span> <span class="n">cleanup_follows_on_block</span><span class="p">();</span>
</code></pre>

</div>



<p>This ensures data consistency even if the client crashes mid-operation.</p>

<h3>
  
  
  3. Design for Offline First
</h3>

<p>We use MMKV (a fast key-value store) for local caching with TTLs:</p>

<ul>
<li>Profiles: 5 days</li>
<li>Feed data: 2 days</li>
<li>Messages: Synced with pending states</li>
</ul>

<p>The app feels snappy because most data is served from cache.</p>

<h3>
  
  
  4. Don't Underestimate Media Handling
</h3>

<p>Image/video upload seems simple until you consider:</p>

<ul>
<li>Compression for faster uploads</li>
<li>Thumbnail generation</li>
<li>Progress tracking</li>
<li>Upload cancellation</li>
<li>Quota enforcement</li>
<li>Signed URL management</li>
</ul>

<p>Our <code>postService.ts</code> is 56KB for a reason.</p>

<h3>
  
  
  5. React Query is a Game Changer
</h3>

<p>Before React Query, we had manual cache invalidation everywhere. Now:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">data</span><span class="p">,</span> <span class="nx">isLoading</span><span class="p">,</span> <span class="nx">refetch</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useQuery</span><span class="p">({</span>
  <span class="na">queryKey</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">posts</span><span class="dl">'</span><span class="p">,</span> <span class="nx">userId</span><span class="p">],</span>
  <span class="na">queryFn</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">fetchUserPosts</span><span class="p">(</span><span class="nx">userId</span><span class="p">),</span>
  <span class="na">staleTime</span><span class="p">:</span> <span class="mi">5</span> <span class="o">*</span> <span class="mi">60</span> <span class="o">*</span> <span class="mi">1000</span><span class="p">,</span>
<span class="p">});</span>

<span class="c1">// Mutations automatically invalidate related queries</span>
<span class="nf">useMutation</span><span class="p">({</span>
  <span class="na">mutationFn</span><span class="p">:</span> <span class="nx">createPost</span><span class="p">,</span>
  <span class="na">onSuccess</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">queryClient</span><span class="p">.</span><span class="nf">invalidateQueries</span><span class="p">({</span> <span class="na">queryKey</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">posts</span><span class="dl">'</span><span class="p">]</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  What's Next?
</h2>

<p>We're polishing the final details before App Store submission:</p>

<ul>
<li>Performance profiling on older devices</li>
<li>Accessibility audit</li>
<li>Final round of beta testing</li>
<li>App Store assets and screenshots</li>
</ul>

<h2>
  
  
  Want to Follow Along?
</h2>

<p>We're building in public! Follow our journey:</p>

<ul>
<li>X: <a href="https://x.com/alexrosrgz" rel="noopener noreferrer">https://x.com/alexrosrgz</a>
</li>
<li>LinkedIn: <a href="https://www.linkedin.com/in/alexrosrgz/" rel="noopener noreferrer">https://www.linkedin.com/in/alexrosrgz/</a>
</li>
<li>Landing page and waitlist: <a href="https://synapsehub.social" rel="noopener noreferrer">synapsehub.social</a>
</li>
</ul>

<p>If you're building something similar or have questions about our approach, drop a comment below. Happy to share more details about any aspect of the stack!</p>

<p><em>What technical challenges have you faced building mobile apps? Let's discuss in the comments!</em></p>

