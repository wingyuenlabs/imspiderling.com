---
Title: Angular State Management with RxJS: Complete Guide | Observables, Subjects & BehaviorSubjects
Description: 
Author: Md. Maruf Rahman
Date: 2026-01-29T20:58:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>When I first started building Angular applications, I thought I needed Redux or NgRx for state management. Then I realized that Angular is built on <strong>RxJS</strong>, and I could manage state effectively using just Observables, Subjects, and BehaviorSubjects. This approach is lighter, simpler, and works perfectly for most applications.</p>

<p><strong>RxJS</strong> is Angular's foundation for reactive programming. It provides powerful tools for managing asynchronous data streams, which makes it perfect for state management. Using BehaviorSubject to store state, exposing Observables for components to subscribe to, and using RxJS operators for derived state, you can build robust state management without external libraries.</p>

<blockquote>
<p>📖 <strong>Want the complete guide with more examples and advanced patterns?</strong> Check out the <a href="https://marufrahman.live/blog/angular-state-management-rxjs" rel="noopener noreferrer">full article on my blog</a> for an in-depth tutorial with additional code examples, troubleshooting tips, and real-world use cases.</p>
</blockquote>

<h2>
  
  
  What is RxJS State Management?
</h2>

<p><strong>RxJS State Management</strong> provides:</p>

<ul>
<li>
<strong>Reactive state</strong> - State as Observable streams</li>
<li>
<strong>BehaviorSubject</strong> - Stores current state value</li>
<li>
<strong>Subjects</strong> - Event streams and notifications</li>
<li>
<strong>Operators</strong> - Transform and combine state</li>
<li>
<strong>No external libraries</strong> - Built into Angular</li>
<li>
<strong>Type-safe</strong> - Full TypeScript support</li>
<li>
<strong>Testable</strong> - Easy to test with RxJS testing utilities</li>
</ul>

<h2>
  
  
  BehaviorSubject for State
</h2>

<p>Create a state management service:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BehaviorSubject</span><span class="p">,</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">map</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">AppState</span> <span class="p">{</span>
  <span class="nl">user</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span>
  <span class="nl">businesses</span><span class="p">:</span> <span class="kr">any</span><span class="p">[];</span>
  <span class="nl">selectedBusiness</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span>
<span class="p">}</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">StateService</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">stateSubject</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">BehaviorSubject</span><span class="o">&lt;</span><span class="nx">AppState</span><span class="o">&gt;</span><span class="p">({</span>
    <span class="na">user</span><span class="p">:</span> <span class="kc">null</span><span class="p">,</span>
    <span class="na">businesses</span><span class="p">:</span> <span class="p">[],</span>
    <span class="na">selectedBusiness</span><span class="p">:</span> <span class="kc">null</span>
  <span class="p">});</span>

  <span class="k">public</span> <span class="nx">state$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nf">asObservable</span><span class="p">();</span>

  <span class="nf">getState</span><span class="p">():</span> <span class="nx">AppState</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nf">setState</span><span class="p">(</span><span class="nx">partialState</span><span class="p">:</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">AppState</span><span class="o">&gt;</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nf">next</span><span class="p">({</span>
      <span class="p">...</span><span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span>
      <span class="p">...</span><span class="nx">partialState</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Specific state selectors</span>
  <span class="nx">getUser</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">state</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">user</span><span class="p">)</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">state</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">businesses</span><span class="p">)</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nx">getSelectedBusiness</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">state</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">selectedBusiness</span><span class="p">)</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>Key Points:</strong></p>

<ul>
<li>
<code>BehaviorSubject</code> stores the current state value</li>
<li>Expose <code>Observable</code> for components to subscribe</li>
<li>Provide methods to update state immutably</li>
<li>Create selectors for specific state slices</li>
</ul>

<h2>
  
  
  Using State in Components
</h2>

<p>Subscribe to state changes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span><span class="p">,</span> <span class="nx">OnInit</span><span class="p">,</span> <span class="nx">OnDestroy</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Subscription</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">StateService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./state.service</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessListComponent</span> <span class="k">implements</span> <span class="nx">OnInit</span><span class="p">,</span> <span class="nx">OnDestroy</span> <span class="p">{</span>
  <span class="nl">businesses</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>
  <span class="nl">selectedBusiness</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="kc">null</span><span class="p">;</span>
  <span class="k">private</span> <span class="nx">subscription</span><span class="p">:</span> <span class="nx">Subscription</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">stateService</span><span class="p">:</span> <span class="nx">StateService</span><span class="p">)</span> <span class="p">{}</span>

  <span class="nf">ngOnInit</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="c1">// Subscribe to businesses</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">subscription</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">().</span><span class="nf">subscribe</span><span class="p">(</span>
      <span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">businesses</span> <span class="o">=</span> <span class="nx">businesses</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="c1">// Subscribe to selected business</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">getSelectedBusiness</span><span class="nf">$</span><span class="p">().</span><span class="nf">subscribe</span><span class="p">(</span>
      <span class="nx">business</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">selectedBusiness</span> <span class="o">=</span> <span class="nx">business</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nf">ngOnDestroy</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">subscription</span><span class="p">)</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">subscription</span><span class="p">.</span><span class="nf">unsubscribe</span><span class="p">();</span>
    <span class="p">}</span>
  <span class="p">}</span>

  <span class="nf">selectBusiness</span><span class="p">(</span><span class="nx">business</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nf">setState</span><span class="p">({</span> <span class="na">selectedBusiness</span><span class="p">:</span> <span class="nx">business</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">loadBusinesses</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="c1">// Load businesses and update state</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">businessService</span><span class="p">.</span><span class="nf">getBusinesses</span><span class="p">().</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nf">setState</span><span class="p">({</span> <span class="nx">businesses</span> <span class="p">});</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Using Async Pipe (Recommended)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">StateService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./state.service</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessListComponent</span> <span class="p">{</span>
  <span class="nx">businesses$</span><span class="p">:</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">();</span>
  <span class="nx">selectedBusiness$</span><span class="p">:</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">getSelectedBusiness</span><span class="nf">$</span><span class="p">();</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">stateService</span><span class="p">:</span> <span class="nx">StateService</span><span class="p">)</span> <span class="p">{}</span>

  <span class="nf">selectBusiness</span><span class="p">(</span><span class="nx">business</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nf">setState</span><span class="p">({</span> <span class="na">selectedBusiness</span><span class="p">:</span> <span class="nx">business</span> <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="c">&lt;!-- Template with async pipe --&gt;</span>
<span class="nt">&lt;div</span> <span class="na">*ngIf=</span><span class="s">"businesses$ | async as businesses"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">*ngFor=</span><span class="s">"let business of businesses"</span><span class="nt">&gt;</span>
    {{ business.name }}
  <span class="nt">&lt;/div&gt;</span>
<span class="nt">&lt;/div&gt;</span>

<span class="nt">&lt;div</span> <span class="na">*ngIf=</span><span class="s">"selectedBusiness$ | async as selected"</span><span class="nt">&gt;</span>
  Selected: {{ selected.name }}
<span class="nt">&lt;/div&gt;</span>
</code></pre>

</div>



<p><strong>Benefits of Async Pipe:</strong></p>

<ul>
<li>Automatically subscribes and unsubscribes</li>
<li>Prevents memory leaks</li>
<li>Cleaner component code</li>
<li>No manual subscription management</li>
</ul>

<h2>
  
  
  RxJS Operators for State
</h2>

<p>Use operators for derived state:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">map</span><span class="p">,</span> <span class="nx">filter</span><span class="p">,</span> <span class="nx">distinctUntilChanged</span><span class="p">,</span> <span class="nx">shareReplay</span><span class="p">,</span> <span class="nx">debounceTime</span><span class="p">,</span> <span class="nx">switchMap</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">combineLatest</span><span class="p">,</span> <span class="k">of</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">StateService</span> <span class="p">{</span>
  <span class="c1">// ... previous code ...</span>

  <span class="c1">// Filtered businesses</span>
  <span class="nx">getActiveBusinesses</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">state</span> <span class="o">=&gt;</span> <span class="nx">state</span><span class="p">.</span><span class="nx">businesses</span><span class="p">),</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="nx">businesses</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">isActive</span><span class="p">)),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">(),</span>
      <span class="nf">shareReplay</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Combined state</span>
  <span class="nx">getBusinessWithUser</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nf">combineLatest</span><span class="p">([</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">getUser</span><span class="nf">$</span><span class="p">(),</span>
      <span class="k">this</span><span class="p">.</span><span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">()</span>
    <span class="p">]).</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(([</span><span class="nx">user</span><span class="p">,</span> <span class="nx">businesses</span><span class="p">])</span> <span class="o">=&gt;</span> <span class="p">({</span>
        <span class="nx">user</span><span class="p">,</span>
        <span class="na">userBusinesses</span><span class="p">:</span> <span class="nx">businesses</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">userId</span> <span class="o">===</span> <span class="nx">user</span><span class="p">.</span><span class="nx">id</span><span class="p">)</span>
      <span class="p">}))</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// Debounced search</span>
  <span class="nf">searchBusinesses</span><span class="p">(</span><span class="nx">term</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">of</span><span class="p">(</span><span class="nx">term</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">debounceTime</span><span class="p">(</span><span class="mi">300</span><span class="p">),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">(),</span>
      <span class="nf">switchMap</span><span class="p">(</span><span class="nx">term</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">businessService</span><span class="p">.</span><span class="nf">search</span><span class="p">(</span><span class="nx">term</span><span class="p">))</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="c1">// State with filtering</span>
  <span class="nx">getBusinessesByStatus</span><span class="nf">$</span><span class="p">(</span><span class="nx">status</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">().</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="nx">businesses</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">status</span> <span class="o">===</span> <span class="nx">status</span><span class="p">)),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">()</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Common RxJS Operators for State
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Operator</th>
<th>Use Case</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>map</code></td>
<td>Transform state</td>
<td><code>map(state =&gt; state.user)</code></td>
</tr>
<tr>
<td><code>filter</code></td>
<td>Filter state values</td>
<td><code>filter(user =&gt; user.isActive)</code></td>
</tr>
<tr>
<td><code>distinctUntilChanged</code></td>
<td>Skip duplicate values</td>
<td><code>distinctUntilChanged()</code></td>
</tr>
<tr>
<td><code>shareReplay</code></td>
<td>Cache and share</td>
<td><code>shareReplay(1)</code></td>
</tr>
<tr>
<td><code>debounceTime</code></td>
<td>Debounce updates</td>
<td><code>debounceTime(300)</code></td>
</tr>
<tr>
<td><code>switchMap</code></td>
<td>Switch to new stream</td>
<td><code>switchMap(id =&gt; this.getData(id))</code></td>
</tr>
<tr>
<td><code>combineLatest</code></td>
<td>Combine multiple streams</td>
<td><code>combineLatest([stream1$, stream2$])</code></td>
</tr>
<tr>
<td><code>mergeMap</code></td>
<td>Merge multiple streams</td>
<td><code>mergeMap(item =&gt; this.process(item))</code></td>
</tr>
<tr>
<td><code>takeUntil</code></td>
<td>Complete on signal</td>
<td><code>takeUntil(this.destroy$)</code></td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Subject for Events
</h2>

<p>Use Subject for event streams:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Subject</span><span class="p">,</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">NotificationService</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">notificationsSubject</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Subject</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span><span class="p">();</span>
  <span class="k">public</span> <span class="nx">notifications$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">notificationsSubject</span><span class="p">.</span><span class="nf">asObservable</span><span class="p">();</span>

  <span class="nf">showNotification</span><span class="p">(</span><span class="nx">message</span><span class="p">:</span> <span class="kr">string</span><span class="p">,</span> <span class="kd">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">success</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">info</span><span class="dl">'</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">notificationsSubject</span><span class="p">.</span><span class="nf">next</span><span class="p">({</span> 
      <span class="nx">message</span><span class="p">,</span> 
      <span class="kd">type</span><span class="p">,</span> 
      <span class="na">timestamp</span><span class="p">:</span> <span class="nb">Date</span><span class="p">.</span><span class="nf">now</span><span class="p">(),</span>
      <span class="na">id</span><span class="p">:</span> <span class="nb">Math</span><span class="p">.</span><span class="nf">random</span><span class="p">().</span><span class="nf">toString</span><span class="p">(</span><span class="mi">36</span><span class="p">)</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">clearNotifications</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">notificationsSubject</span><span class="p">.</span><span class="nf">next</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage in component</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">MyComponent</span> <span class="p">{</span>
  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">notificationService</span><span class="p">:</span> <span class="nx">NotificationService</span><span class="p">)</span> <span class="p">{}</span>

  <span class="nf">save</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="c1">// Save logic</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">businessService</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="nx">data</span><span class="p">).</span><span class="nf">subscribe</span><span class="p">({</span>
      <span class="na">next</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">notificationService</span><span class="p">.</span><span class="nf">showNotification</span><span class="p">(</span><span class="dl">'</span><span class="s1">Saved successfully</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">success</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">},</span>
      <span class="na">error</span><span class="p">:</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">notificationService</span><span class="p">.</span><span class="nf">showNotification</span><span class="p">(</span><span class="dl">'</span><span class="s1">Save failed</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">);</span>
      <span class="p">}</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Subject vs BehaviorSubject
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Subject</th>
<th>BehaviorSubject</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Initial Value</strong></td>
<td>No</td>
<td>Yes (required)</td>
</tr>
<tr>
<td><strong>Current Value</strong></td>
<td>No</td>
<td>Yes (stored)</td>
</tr>
<tr>
<td><strong>New Subscribers</strong></td>
<td>No immediate value</td>
<td>Gets current value immediately</td>
</tr>
<tr>
<td><strong>Use Case</strong></td>
<td>Events, notifications</td>
<td>State management</td>
</tr>
<tr>
<td><strong>Example</strong></td>
<td><code>new Subject()</code></td>
<td><code>new BehaviorSubject(initialValue)</code></td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Advanced State Patterns
</h2>

<h3>
  
  
  1. State with Actions
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BehaviorSubject</span><span class="p">,</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">map</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>

<span class="kr">interface</span> <span class="nx">State</span> <span class="p">{</span>
  <span class="nl">businesses</span><span class="p">:</span> <span class="kr">any</span><span class="p">[];</span>
  <span class="nl">loading</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">;</span>
  <span class="nl">error</span><span class="p">:</span> <span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="p">;</span>
<span class="p">}</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessStateService</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">stateSubject</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">BehaviorSubject</span><span class="o">&lt;</span><span class="nx">State</span><span class="o">&gt;</span><span class="p">({</span>
    <span class="na">businesses</span><span class="p">:</span> <span class="p">[],</span>
    <span class="na">loading</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
    <span class="na">error</span><span class="p">:</span> <span class="kc">null</span>
  <span class="p">});</span>

  <span class="k">public</span> <span class="nx">state$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nf">asObservable</span><span class="p">();</span>
  <span class="k">public</span> <span class="nx">businesses$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">map</span><span class="p">(</span><span class="nx">s</span> <span class="o">=&gt;</span> <span class="nx">s</span><span class="p">.</span><span class="nx">businesses</span><span class="p">));</span>
  <span class="k">public</span> <span class="nx">loading$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">map</span><span class="p">(</span><span class="nx">s</span> <span class="o">=&gt;</span> <span class="nx">s</span><span class="p">.</span><span class="nx">loading</span><span class="p">));</span>
  <span class="k">public</span> <span class="nx">error$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">map</span><span class="p">(</span><span class="nx">s</span> <span class="o">=&gt;</span> <span class="nx">s</span><span class="p">.</span><span class="nx">error</span><span class="p">));</span>

  <span class="c1">// Actions</span>
  <span class="nf">setLoading</span><span class="p">(</span><span class="nx">loading</span><span class="p">:</span> <span class="nx">boolean</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">updateState</span><span class="p">({</span> <span class="nx">loading</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">setBusinesses</span><span class="p">(</span><span class="nx">businesses</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">updateState</span><span class="p">({</span> <span class="nx">businesses</span><span class="p">,</span> <span class="na">loading</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span> <span class="na">error</span><span class="p">:</span> <span class="kc">null</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">setError</span><span class="p">(</span><span class="nx">error</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">updateState</span><span class="p">({</span> <span class="nx">error</span><span class="p">,</span> <span class="na">loading</span><span class="p">:</span> <span class="kc">false</span> <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">addBusiness</span><span class="p">(</span><span class="nx">business</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">current</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">updateState</span><span class="p">({</span>
      <span class="na">businesses</span><span class="p">:</span> <span class="p">[...</span><span class="nx">current</span><span class="p">.</span><span class="nx">businesses</span><span class="p">,</span> <span class="nx">business</span><span class="p">]</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">updateBusiness</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">number</span><span class="p">,</span> <span class="nx">updates</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">current</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">updateState</span><span class="p">({</span>
      <span class="na">businesses</span><span class="p">:</span> <span class="nx">current</span><span class="p">.</span><span class="nx">businesses</span><span class="p">.</span><span class="nf">map</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> 
        <span class="nx">b</span><span class="p">.</span><span class="nx">id</span> <span class="o">===</span> <span class="nx">id</span> <span class="p">?</span> <span class="p">{</span> <span class="p">...</span><span class="nx">b</span><span class="p">,</span> <span class="p">...</span><span class="nx">updates</span> <span class="p">}</span> <span class="p">:</span> <span class="nx">b</span>
      <span class="p">)</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">deleteBusiness</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">number</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">current</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">updateState</span><span class="p">({</span>
      <span class="na">businesses</span><span class="p">:</span> <span class="nx">current</span><span class="p">.</span><span class="nx">businesses</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">id</span> <span class="o">!==</span> <span class="nx">id</span><span class="p">)</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="k">private</span> <span class="nf">updateState</span><span class="p">(</span><span class="nx">partial</span><span class="p">:</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">State</span><span class="o">&gt;</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nf">next</span><span class="p">({</span>
      <span class="p">...</span><span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span>
      <span class="p">...</span><span class="nx">partial</span>
    <span class="p">});</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  2. State with Effects
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BehaviorSubject</span><span class="p">,</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">tap</span><span class="p">,</span> <span class="nx">catchError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BusinessService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./business.service</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessStateService</span> <span class="p">{</span>
  <span class="c1">// ... state setup ...</span>

  <span class="nf">loadBusinesses</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">setLoading</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businessService</span><span class="p">.</span><span class="nf">getBusinesses</span><span class="p">().</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">tap</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">setBusinesses</span><span class="p">(</span><span class="nx">businesses</span><span class="p">);</span>
      <span class="p">}),</span>
      <span class="nf">catchError</span><span class="p">(</span><span class="nx">error</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">setError</span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
        <span class="k">return</span> <span class="p">[];</span>
      <span class="p">})</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nf">saveBusiness</span><span class="p">(</span><span class="nx">business</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nf">setLoading</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businessService</span><span class="p">.</span><span class="nf">save</span><span class="p">(</span><span class="nx">business</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">tap</span><span class="p">(</span><span class="nx">saved</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">addBusiness</span><span class="p">(</span><span class="nx">saved</span><span class="p">);</span>
      <span class="p">}),</span>
      <span class="nf">catchError</span><span class="p">(</span><span class="nx">error</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nf">setError</span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
        <span class="k">throw</span> <span class="nx">error</span><span class="p">;</span>
      <span class="p">})</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  3. State with Selectors
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Injectable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">map</span><span class="p">,</span> <span class="nx">distinctUntilChanged</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessStateService</span> <span class="p">{</span>
  <span class="c1">// ... state setup ...</span>

  <span class="c1">// Selectors</span>
  <span class="nx">getBusinessById</span><span class="nf">$</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">number</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businesses$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="nx">businesses</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">id</span> <span class="o">===</span> <span class="nx">id</span><span class="p">)),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">()</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nx">getActiveBusinesses</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businesses$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="nx">businesses</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="nx">b</span> <span class="o">=&gt;</span> <span class="nx">b</span><span class="p">.</span><span class="nx">isActive</span><span class="p">)),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">()</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nx">getBusinessesCount</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">number</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businesses$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="nx">businesses</span><span class="p">.</span><span class="nx">length</span><span class="p">),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">()</span>
    <span class="p">);</span>
  <span class="p">}</span>

  <span class="nx">hasBusinesses</span><span class="nf">$</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="nx">boolean</span><span class="o">&gt;</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businesses$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
      <span class="nf">map</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="nx">businesses</span><span class="p">.</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">),</span>
      <span class="nf">distinctUntilChanged</span><span class="p">()</span>
    <span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Component Patterns
</h2>

<h3>
  
  
  Using takeUntil Pattern
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span><span class="p">,</span> <span class="nx">OnInit</span><span class="p">,</span> <span class="nx">OnDestroy</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Subject</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">takeUntil</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">StateService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./state.service</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessComponent</span> <span class="k">implements</span> <span class="nx">OnInit</span><span class="p">,</span> <span class="nx">OnDestroy</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">destroy$</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Subject</span><span class="o">&lt;</span><span class="k">void</span><span class="o">&gt;</span><span class="p">();</span>
  <span class="nl">businesses</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[];</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">stateService</span><span class="p">:</span> <span class="nx">StateService</span><span class="p">)</span> <span class="p">{}</span>

  <span class="nf">ngOnInit</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">()</span>
      <span class="p">.</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">takeUntil</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">destroy$</span><span class="p">))</span>
      <span class="p">.</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">businesses</span> <span class="o">=</span> <span class="nx">businesses</span><span class="p">;</span>
      <span class="p">});</span>
  <span class="p">}</span>

  <span class="nf">ngOnDestroy</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">destroy$</span><span class="p">.</span><span class="nf">next</span><span class="p">();</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">destroy$</span><span class="p">.</span><span class="nf">complete</span><span class="p">();</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Using Async Pipe (Best Practice)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Observable</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">StateService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./state.service</span><span class="dl">'</span><span class="p">;</span>

<span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessComponent</span> <span class="p">{</span>
  <span class="nx">businesses$</span><span class="p">:</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">getBusinesses</span><span class="nf">$</span><span class="p">();</span>
  <span class="nx">loading$</span><span class="p">:</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="nx">boolean</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">loading$</span><span class="p">;</span>
  <span class="nx">error$</span><span class="p">:</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">string</span> <span class="o">|</span> <span class="kc">null</span><span class="o">&gt;</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nx">error$</span><span class="p">;</span>

  <span class="nf">constructor</span><span class="p">(</span><span class="k">private</span> <span class="nx">stateService</span><span class="p">:</span> <span class="nx">StateService</span><span class="p">)</span> <span class="p">{}</span>

  <span class="nf">ngOnInit</span><span class="p">():</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nf">loadBusinesses</span><span class="p">().</span><span class="nf">subscribe</span><span class="p">();</span>
  <span class="p">}</span>

  <span class="nf">selectBusiness</span><span class="p">(</span><span class="nx">business</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateService</span><span class="p">.</span><span class="nf">setSelectedBusiness</span><span class="p">(</span><span class="nx">business</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Best Practices
</h2>

<ol>
<li>
<strong>Use BehaviorSubject for state</strong> - Stores current value</li>
<li>
<strong>Use Subject for events</strong> - Event streams and notifications</li>
<li>
<strong>Always unsubscribe</strong> - Or use async pipe</li>
<li>
<strong>Use async pipe in templates</strong> - Automatic subscription management</li>
<li>
<strong>Use distinctUntilChanged</strong> - Prevent unnecessary updates</li>
<li>
<strong>Use shareReplay</strong> - Cache expensive computations</li>
<li>
<strong>Create specific selectors</strong> - For derived state</li>
<li>
<strong>Keep state services focused</strong> - Single responsibility</li>
<li>
<strong>Use combineLatest</strong> - For combining multiple streams</li>
<li>
<strong>Document state structure</strong> - And update patterns</li>
<li>
<strong>Use TypeScript interfaces</strong> - For type safety</li>
<li>
<strong>Immutable updates</strong> - Always create new state objects</li>
<li>
<strong>Handle errors</strong> - In state services</li>
<li>
<strong>Use operators wisely</strong> - Don't over-complicate</li>
</ol>

<h2>
  
  
  Common Patterns
</h2>

<h3>
  
  
  State Service Structure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="p">@</span><span class="nd">Injectable</span><span class="p">({</span>
  <span class="na">providedIn</span><span class="p">:</span> <span class="dl">'</span><span class="s1">root</span><span class="dl">'</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">FeatureStateService</span> <span class="p">{</span>
  <span class="c1">// Private BehaviorSubject</span>
  <span class="k">private</span> <span class="nx">stateSubject</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">BehaviorSubject</span><span class="o">&lt;</span><span class="nx">State</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">initialState</span><span class="p">);</span>

  <span class="c1">// Public Observable</span>
  <span class="k">public</span> <span class="nx">state$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nf">asObservable</span><span class="p">();</span>

  <span class="c1">// Specific selectors</span>
  <span class="k">public</span> <span class="nx">feature$</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">state$</span><span class="p">.</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">map</span><span class="p">(</span><span class="nx">s</span> <span class="o">=&gt;</span> <span class="nx">s</span><span class="p">.</span><span class="nx">feature</span><span class="p">));</span>

  <span class="c1">// Get current state</span>
  <span class="nf">getState</span><span class="p">():</span> <span class="nx">State</span> <span class="p">{</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="c1">// Update state</span>
  <span class="nf">setState</span><span class="p">(</span><span class="nx">partial</span><span class="p">:</span> <span class="nb">Partial</span><span class="o">&lt;</span><span class="nx">State</span><span class="o">&gt;</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nf">next</span><span class="p">({</span>
      <span class="p">...</span><span class="k">this</span><span class="p">.</span><span class="nx">stateSubject</span><span class="p">.</span><span class="nx">value</span><span class="p">,</span>
      <span class="p">...</span><span class="nx">partial</span>
    <span class="p">});</span>
  <span class="p">}</span>

  <span class="c1">// Actions</span>
  <span class="nf">loadData</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span> <span class="p">}</span>
  <span class="nf">saveData</span><span class="p">(</span><span class="nx">data</span><span class="p">:</span> <span class="kr">any</span><span class="p">):</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="o">&gt;</span> <span class="p">{</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Error Handling Pattern
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">loadBusinesses</span><span class="p">():</span> <span class="nx">Observable</span><span class="o">&lt;</span><span class="kr">any</span><span class="p">[]</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">this</span><span class="p">.</span><span class="nf">setLoading</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
  <span class="k">this</span><span class="p">.</span><span class="nf">setError</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>

  <span class="k">return</span> <span class="k">this</span><span class="p">.</span><span class="nx">businessService</span><span class="p">.</span><span class="nf">getBusinesses</span><span class="p">().</span><span class="nf">pipe</span><span class="p">(</span>
    <span class="nf">tap</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">setBusinesses</span><span class="p">(</span><span class="nx">businesses</span><span class="p">);</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">setLoading</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="p">}),</span>
    <span class="nf">catchError</span><span class="p">(</span><span class="nx">error</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">setError</span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">);</span>
      <span class="k">this</span><span class="p">.</span><span class="nf">setLoading</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
      <span class="k">return</span> <span class="k">of</span><span class="p">([]);</span>
    <span class="p">})</span>
  <span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  When to Use RxJS vs NgRx
</h2>

<h3>
  
  
  Use RxJS State Management When:
</h3>

<ul>
<li>✅ Medium-sized applications</li>
<li>✅ Simple to moderate state complexity</li>
<li>✅ No need for time-travel debugging</li>
<li>✅ Team familiar with RxJS</li>
<li>✅ Want to avoid external dependencies</li>
</ul>

<h3>
  
  
  Consider NgRx When:
</h3>

<ul>
<li>✅ Large enterprise applications</li>
<li>✅ Complex state with many interactions</li>
<li>✅ Need time-travel debugging</li>
<li>✅ Multiple teams working on same codebase</li>
<li>✅ Need strict state management patterns</li>
</ul>

<h2>
  
  
  Resources and Further Reading
</h2>

<ul>
<li>
<strong><a href="https://marufrahman.live/blog/angular-state-management-rxjs" rel="noopener noreferrer">📚 Full Angular State Management Guide</a></strong> - Complete tutorial with advanced examples, troubleshooting, and best practices</li>
<li>
<strong><a href="https://marufrahman.live/blog/angular-services-dependency-injection" rel="noopener noreferrer">Angular Services Guide</a></strong> - Services with RxJS</li>
<li>
<strong><a href="https://marufrahman.live/blog/angular-component-communication" rel="noopener noreferrer">Angular Component Communication</a></strong> - State sharing between components</li>
<li>
<strong><a href="https://marufrahman.live/blog/angular-http-client-interceptors" rel="noopener noreferrer">Angular HTTP Client Guide</a></strong> - HTTP with RxJS</li>
<li>
<a href="https://rxjs.dev/" rel="noopener noreferrer">RxJS Documentation</a> - Official RxJS docs</li>
<li>
<a href="https://angular.io/guide/observables" rel="noopener noreferrer">Angular Observables Guide</a> - Official Angular observables guide</li>
<li>
<a href="https://rxjs.dev/guide/operators" rel="noopener noreferrer">RxJS Operators Guide</a> - Complete operators reference</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p><strong>RxJS</strong> provides powerful tools for state management in Angular. With BehaviorSubject, Subjects, and operators, you can build scalable state management solutions without external libraries. This approach works well for medium to large Angular applications.</p>

<p><strong>Key Takeaways:</strong></p>

<ul>
<li>
<strong>BehaviorSubject</strong> - Stores current state value (use for state)</li>
<li>
<strong>Subject</strong> - Event streams (use for events/notifications)</li>
<li>
<strong>Observables</strong> - Expose state to components</li>
<li>
<strong>RxJS Operators</strong> - Transform and combine state</li>
<li>
<strong>Async Pipe</strong> - Automatic subscription management</li>
<li>
<strong>Selectors</strong> - Derived state from base state</li>
<li>
<strong>Immutable Updates</strong> - Always create new state objects</li>
<li>
<strong>Best Practices</strong> - Unsubscribe, use operators wisely, keep services focused</li>
</ul>

<p>Whether you're building a simple dashboard or a complex enterprise application, RxJS state management provides the foundation you need. It handles all the reactive state logic while keeping your code clean and maintainable.</p>




<p><strong>What's your experience with RxJS state management in Angular? Share your tips and tricks in the comments below!</strong> 🚀</p>




<blockquote>
<p>💡 <strong>Looking for more details?</strong> This is a condensed version of my comprehensive guide. Read the <a href="https://marufrahman.live/blog/angular-state-management-rxjs" rel="noopener noreferrer">full article on my blog</a> for additional examples, advanced patterns, troubleshooting tips, and more in-depth explanations.</p>
</blockquote>

<p><em>If you found this guide helpful, consider checking out my other articles on <a href="https://marufrahman.live/blog" rel="noopener noreferrer">Angular development</a> and <a href="https://marufrahman.live" rel="noopener noreferrer">frontend development best practices</a>.</em></p>

