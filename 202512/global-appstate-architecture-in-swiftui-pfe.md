---
Title: Global AppState Architecture in SwiftUI
Description: 
Author: Sebastien Lato
Date: 2025-12-17T21:24:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>As SwiftUI apps grow, one question always comes up:</p>

<blockquote>
<p>“Where does this state live?”</p>
</blockquote>

<ul>
<li>authentication state
</li>
<li>selected tab
</li>
<li>deep links
</li>
<li>onboarding flow
</li>
<li>global errors
</li>
<li>app lifecycle events
</li>
<li>feature coordination
</li>
</ul>

<p>If you scatter this state across views and ViewModels, you end up with:</p>

<ul>
<li>unpredictable navigation
</li>
<li>duplicated logic
</li>
<li>impossible-to-debug bugs
</li>
<li>state resetting at the wrong time
</li>
</ul>

<p>The solution is a <strong>Global AppState</strong>.</p>

<p>This post shows how to design a <strong>clean, scalable AppState architecture</strong> for SwiftUI — without turning your app into a giant god object.</p>




<h2>
  
  
  🧠 What Is AppState?
</h2>

<p><strong>AppState</strong> represents <strong>global, cross-feature state</strong> that:</p>

<ul>
<li>outlives individual screens
</li>
<li>coordinates multiple features
</li>
<li>reacts to lifecycle changes
</li>
<li>drives navigation
</li>
<li>survives view recreation
</li>
</ul>

<p>AppState is <strong>not</strong> for:</p>

<ul>
<li>local UI state
</li>
<li>per-screen form values
</li>
<li>temporary toggles
</li>
</ul>

<p>Think of it as the <em>brain of the app</em>, not the UI.</p>




<h2>
  
  
  🧱 1. What Belongs in AppState (and What Doesn’t)
</h2>

<h3>
  
  
  ✅ Good AppState candidates
</h3>

<ul>
<li>user session / auth status
</li>
<li>selected tab
</li>
<li>global navigation route
</li>
<li>deep link handling
</li>
<li>app phase (foreground/background)
</li>
<li>global loading / syncing flags
</li>
<li>global error presentation
</li>
</ul>

<h3>
  
  
  ❌ What should NOT be in AppState
</h3>

<ul>
<li>text field values
</li>
<li>scroll positions
</li>
<li>local animations
</li>
<li>feature-specific data lists
</li>
<li>temporary UI toggles
</li>
</ul>

<p>Rule of thumb:</p>

<blockquote>
<p>If multiple features care about it → AppState<br><br>
If only one screen cares → ViewModel</p>
</blockquote>




<h2>
  
  
  🧩 2. Defining a Clean AppState Model
</h2>

<p>Use <code>@Observable</code> for modern SwiftUI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@Observable</span>
<span class="kd">class</span> <span class="kt">AppState</span> <span class="p">{</span>
    <span class="k">var</span> <span class="nv">session</span><span class="p">:</span> <span class="kt">UserSession</span><span class="p">?</span>
    <span class="k">var</span> <span class="nv">selectedTab</span><span class="p">:</span> <span class="kt">Tab</span> <span class="o">=</span> <span class="o">.</span><span class="n">home</span>
    <span class="k">var</span> <span class="nv">route</span><span class="p">:</span> <span class="kt">AppRoute</span><span class="p">?</span>
    <span class="k">var</span> <span class="nv">isSyncing</span> <span class="o">=</span> <span class="kc">false</span>
    <span class="k">var</span> <span class="nv">pendingDeepLink</span><span class="p">:</span> <span class="kt">DeepLink</span><span class="p">?</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This object:</p>

<ul>
<li>lives for the lifetime of the app</li>
<li>is injected once</li>
<li>drives high-level behavior</li>
</ul>




<h2>
  
  
  🌍 3. Injecting AppState at the App Root
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@main</span>
<span class="kd">struct</span> <span class="kt">MyApp</span><span class="p">:</span> <span class="kt">App</span> <span class="p">{</span>
    <span class="kd">@State</span> <span class="kd">private</span> <span class="k">var</span> <span class="nv">appState</span> <span class="o">=</span> <span class="kt">AppState</span><span class="p">()</span>

    <span class="k">var</span> <span class="nv">body</span><span class="p">:</span> <span class="kd">some</span> <span class="kt">Scene</span> <span class="p">{</span>
        <span class="kt">WindowGroup</span> <span class="p">{</span>
            <span class="kt">RootView</span><span class="p">()</span>
                <span class="o">.</span><span class="nf">environment</span><span class="p">(</span><span class="n">appState</span><span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now every view can access it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@Environment</span><span class="p">(</span><span class="kt">AppState</span><span class="o">.</span><span class="k">self</span><span class="p">)</span> <span class="k">var</span> <span class="nv">appState</span>
</code></pre>

</div>



<p>No singletons.<br>
No global variables.<br>
Fully testable.</p>


<h2>
  
  
  🧭 4. State-Driven Navigation
</h2>

<p>Navigation should respond to state, not side effects.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">enum</span> <span class="kt">AppRoute</span><span class="p">:</span> <span class="kt">Hashable</span> <span class="p">{</span>
    <span class="k">case</span> <span class="n">login</span>
    <span class="k">case</span> <span class="n">home</span>
    <span class="k">case</span> <span class="nf">profile</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="kt">String</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In your root view:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="o">.</span><span class="nf">navigationDestination</span><span class="p">(</span><span class="nv">for</span><span class="p">:</span> <span class="kt">AppRoute</span><span class="o">.</span><span class="k">self</span><span class="p">)</span> <span class="p">{</span> <span class="n">route</span> <span class="k">in</span>
    <span class="k">switch</span> <span class="n">route</span> <span class="p">{</span>
    <span class="k">case</span> <span class="o">.</span><span class="nv">login</span><span class="p">:</span>
        <span class="kt">LoginView</span><span class="p">()</span>
    <span class="k">case</span> <span class="o">.</span><span class="nv">home</span><span class="p">:</span>
        <span class="kt">HomeView</span><span class="p">()</span>
    <span class="k">case</span> <span class="o">.</span><span class="nf">profile</span><span class="p">(</span><span class="k">let</span> <span class="nv">id</span><span class="p">):</span>
        <span class="kt">ProfileView</span><span class="p">(</span><span class="nv">userID</span><span class="p">:</span> <span class="n">id</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Trigger navigation by updating state:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="n">appState</span><span class="o">.</span><span class="n">route</span> <span class="o">=</span> <span class="o">.</span><span class="nf">profile</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="n">user</span><span class="o">.</span><span class="n">id</span><span class="p">)</span>
</code></pre>

</div>



<p>Navigation becomes:</p>

<ul>
<li>predictable</li>
<li>testable</li>
<li>deep-link friendly</li>
</ul>




<h2>
  
  
  🔗 5. Deep Linking via AppState
</h2>

<p>Handle deep links centrally:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">func</span> <span class="nf">handle</span><span class="p">(</span><span class="n">_</span> <span class="nv">link</span><span class="p">:</span> <span class="kt">DeepLink</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">switch</span> <span class="n">link</span> <span class="p">{</span>
    <span class="k">case</span> <span class="o">.</span><span class="nf">profile</span><span class="p">(</span><span class="k">let</span> <span class="nv">id</span><span class="p">):</span>
        <span class="n">route</span> <span class="o">=</span> <span class="o">.</span><span class="nf">profile</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="n">id</span><span class="p">)</span>
    <span class="k">case</span> <span class="o">.</span><span class="nv">home</span><span class="p">:</span>
        <span class="n">selectedTab</span> <span class="o">=</span> <span class="o">.</span><span class="n">home</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Views do not parse URLs.<br>
Features do not know about URLs.</p>

<p>AppState translates external intent → internal state.</p>


<h2>
  
  
  🔄 6. App Lifecycle Integration
</h2>

<p>AppState is the perfect place to react to lifecycle changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@Environment</span><span class="p">(\</span><span class="o">.</span><span class="n">scenePhase</span><span class="p">)</span> <span class="k">var</span> <span class="nv">phase</span>

<span class="o">.</span><span class="nf">onChange</span><span class="p">(</span><span class="nv">of</span><span class="p">:</span> <span class="n">phase</span><span class="p">)</span> <span class="p">{</span> <span class="n">newPhase</span> <span class="k">in</span>
    <span class="k">switch</span> <span class="n">newPhase</span> <span class="p">{</span>
    <span class="k">case</span> <span class="o">.</span><span class="nv">background</span><span class="p">:</span>
        <span class="nf">saveState</span><span class="p">()</span>
    <span class="k">case</span> <span class="o">.</span><span class="nv">active</span><span class="p">:</span>
        <span class="nf">refreshIfNeeded</span><span class="p">()</span>
    <span class="k">default</span><span class="p">:</span>
        <span class="k">break</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This keeps lifecycle logic out of views and ViewModels.</p>




<h2>
  
  
  ⚠️ 7. Avoid the “God AppState” Anti-Pattern
</h2>

<p>Do not put everything in AppState.</p>

<p>Bad:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">class</span> <span class="kt">AppState</span> <span class="p">{</span>
    <span class="k">var</span> <span class="nv">feedPosts</span><span class="p">:</span> <span class="p">[</span><span class="kt">Post</span><span class="p">]</span>
    <span class="k">var</span> <span class="nv">profileViewModel</span><span class="p">:</span> <span class="kt">ProfileViewModel</span>
    <span class="k">var</span> <span class="nv">searchText</span><span class="p">:</span> <span class="kt">String</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Good:</p>

<ul>
<li>AppState coordinates</li>
<li>ViewModels own feature data</li>
<li>Services handle side effects</li>
</ul>

<p>If AppState grows too large:<br>
👉 split into sub-states.</p>


<h2>
  
  
  🧱 8. Composing AppState with Sub-States
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">@Observable</span>
<span class="kd">class</span> <span class="kt">AppState</span> <span class="p">{</span>
    <span class="k">var</span> <span class="nv">sessionState</span> <span class="o">=</span> <span class="kt">SessionState</span><span class="p">()</span>
    <span class="k">var</span> <span class="nv">navigationState</span> <span class="o">=</span> <span class="kt">NavigationState</span><span class="p">()</span>
    <span class="k">var</span> <span class="nv">syncState</span> <span class="o">=</span> <span class="kt">SyncState</span><span class="p">()</span>
<span class="p">}</span>
</code></pre>

</div>


<p>Each sub-state has:</p>

<ul>
<li>clear responsibility</li>
<li>limited surface area</li>
</ul>

<p>This scales beautifully in large apps.</p>


<h2>
  
  
  🧪 9. Testing AppState
</h2>

<p>Because AppState is just data:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">func</span> <span class="nf">test_navigation_changes_route</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">let</span> <span class="nv">appState</span> <span class="o">=</span> <span class="kt">AppState</span><span class="p">()</span>
    <span class="n">appState</span><span class="o">.</span><span class="n">route</span> <span class="o">=</span> <span class="o">.</span><span class="n">login</span>
    <span class="kt">XCTAssertEqual</span><span class="p">(</span><span class="n">appState</span><span class="o">.</span><span class="n">route</span><span class="p">,</span> <span class="o">.</span><span class="n">login</span><span class="p">)</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Test deep links:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight swift"><code><span class="kd">func</span> <span class="nf">test_profile_deep_link</span><span class="p">()</span> <span class="p">{</span>
    <span class="k">let</span> <span class="nv">appState</span> <span class="o">=</span> <span class="kt">AppState</span><span class="p">()</span>
    <span class="n">appState</span><span class="o">.</span><span class="nf">handle</span><span class="p">(</span><span class="o">.</span><span class="nf">profile</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="s">"123"</span><span class="p">))</span>
    <span class="kt">XCTAssertEqual</span><span class="p">(</span><span class="n">appState</span><span class="o">.</span><span class="n">route</span><span class="p">,</span> <span class="o">.</span><span class="nf">profile</span><span class="p">(</span><span class="nv">id</span><span class="p">:</span> <span class="s">"123"</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<p>No UI.<br>
No mocks.<br>
No hacks.</p>


<h2>
  
  
  🧠 10. Mental Model Cheat Sheet
</h2>

<ul>
<li>AppState = coordination</li>
<li>ViewModels = feature logic</li>
<li>Views = rendering + intent</li>
<li>Services = side effects</li>
</ul>

<p>State flows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight sql"><code><span class="k">User</span> <span class="n">Action</span>
   <span class="err">↓</span>
<span class="n">ViewModel</span>
   <span class="err">↓</span>
<span class="n">AppState</span> <span class="p">(</span><span class="n">if</span> <span class="k">global</span><span class="p">)</span>
   <span class="err">↓</span>
<span class="k">View</span> <span class="n">reacts</span>
</code></pre>

</div>






<h2>
  
  
  🚀 Final Thoughts
</h2>

<p>A well-designed Global AppState:</p>

<ul>
<li>simplifies navigation</li>
<li>stabilizes lifecycle behavior</li>
<li>makes deep links trivial</li>
<li>removes global hacks</li>
<li>improves testability</li>
<li>scales across teams</li>
</ul>

<p>It’s one of the most powerful architectural tools in SwiftUI — when used with restraint.</p>

