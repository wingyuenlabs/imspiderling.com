---
Title: Angular Testing with Jasmine and Karma: Complete Guide | Unit Testing, Component Testing & E2E
Description: 
Author: Md. Maruf Rahman
Date: 2026-01-29T21:01:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>I'll be honest—when I first started with Angular, I skipped testing. I thought it was too time-consuming, and my code worked fine without it. Then I had to refactor a large feature, and I broke three things I didn't even know existed. That's when I learned the value of good tests. They give you confidence to refactor, catch bugs before they reach production, and serve as documentation for how your code should work.</p>

<p><strong>Angular</strong> provides excellent testing support out of the box with <strong>Jasmine</strong> (the testing framework) and <strong>Karma</strong> (the test runner). <strong>TestBed</strong> makes it easy to configure testing modules, mock dependencies, and test components in isolation. The testing utilities are powerful enough to test complex scenarios while being simple enough to write tests quickly.</p>

<blockquote>
<p>📖 <strong>Want the complete guide with more examples and advanced patterns?</strong> Check out the <a href="https://marufrahman.live/blog/angular-testing-jasmine-karma" rel="noopener noreferrer">full article on my blog</a> for an in-depth tutorial with additional code examples, troubleshooting tips, and real-world use cases.</p>
</blockquote>

<h2>
  
  
  What is Angular Testing?
</h2>

<p><strong>Angular Testing</strong> provides:</p>

<ul>
<li>
<strong>Jasmine</strong> - Behavior-driven testing framework</li>
<li>
<strong>Karma</strong> - Test runner for running tests in browsers</li>
<li>
<strong>TestBed</strong> - Testing utilities for configuring test modules</li>
<li>
<strong>Component Testing</strong> - Test components in isolation</li>
<li>
<strong>Service Testing</strong> - Test business logic and HTTP calls</li>
<li>
<strong>Pipe &amp; Directive Testing</strong> - Test custom transformations</li>
<li>
<strong>Async Testing</strong> - Test asynchronous operations</li>
<li>
<strong>Mocking</strong> - Mock dependencies with spies</li>
</ul>

<h2>
  
  
  Component Testing
</h2>

<p>Test Angular components with TestBed:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">ComponentFixture</span><span class="p">,</span> <span class="nx">TestBed</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BusinessListComponent</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./business-list.component</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BusinessService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">src/services/business.service</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="k">of</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">ReactiveFormsModule</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/forms</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HttpClientTestingModule</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http/testing</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">BusinessListComponent</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">component</span><span class="p">:</span> <span class="nx">BusinessListComponent</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">fixture</span><span class="p">:</span> <span class="nx">ComponentFixture</span><span class="o">&lt;</span><span class="nx">BusinessListComponent</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">businessService</span><span class="p">:</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">BusinessService</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">spy</span> <span class="o">=</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nf">createSpyObj</span><span class="p">(</span><span class="dl">'</span><span class="s1">BusinessService</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span><span class="dl">'</span><span class="s1">GetBusinesses</span><span class="dl">'</span><span class="p">]);</span>

    <span class="k">await</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">declarations</span><span class="p">:</span> <span class="p">[</span><span class="nx">BusinessListComponent</span><span class="p">],</span>
      <span class="na">providers</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span> <span class="na">provide</span><span class="p">:</span> <span class="nx">BusinessService</span><span class="p">,</span> <span class="na">useValue</span><span class="p">:</span> <span class="nx">spy</span> <span class="p">}</span>
      <span class="p">],</span>
      <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">ReactiveFormsModule</span><span class="p">,</span> <span class="nx">HttpClientTestingModule</span><span class="p">]</span>
    <span class="p">}).</span><span class="nf">compileComponents</span><span class="p">();</span>

    <span class="nx">fixture</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">createComponent</span><span class="p">(</span><span class="nx">BusinessListComponent</span><span class="p">);</span>
    <span class="nx">component</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">componentInstance</span><span class="p">;</span>
    <span class="nx">businessService</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">BusinessService</span><span class="p">)</span> <span class="k">as</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">BusinessService</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should create</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">component</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should load businesses on init</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">mockBusinesses</span> <span class="o">=</span> <span class="p">[{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Business 1</span><span class="dl">'</span> <span class="p">}];</span>
    <span class="nx">businessService</span><span class="p">.</span><span class="nx">GetBusinesses</span><span class="p">.</span><span class="nx">and</span><span class="p">.</span><span class="nf">returnValue</span><span class="p">(</span><span class="k">of</span><span class="p">({</span> <span class="na">data</span><span class="p">:</span> <span class="nx">mockBusinesses</span> <span class="p">}));</span>

    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">businessService</span><span class="p">.</span><span class="nx">GetBusinesses</span><span class="p">).</span><span class="nf">toHaveBeenCalled</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">component</span><span class="p">.</span><span class="nx">businesses</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">(</span><span class="nx">mockBusinesses</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should display businesses in template</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">component</span><span class="p">.</span><span class="nx">businesses</span> <span class="o">=</span> <span class="p">[{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Business 1</span><span class="dl">'</span> <span class="p">}];</span>
    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">compiled</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">nativeElement</span><span class="p">;</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">compiled</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">.business-name</span><span class="dl">'</span><span class="p">).</span><span class="nx">textContent</span><span class="p">).</span><span class="nf">toContain</span><span class="p">(</span><span class="dl">'</span><span class="s1">Business 1</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<p><strong>Key Points:</strong></p>

<ul>
<li>Use <code>TestBed.configureTestingModule()</code> to configure test module</li>
<li>Mock dependencies with <code>jasmine.createSpyObj()</code>
</li>
<li>Use <code>fixture.detectChanges()</code> to trigger change detection</li>
<li>Query DOM with <code>fixture.nativeElement</code> or <code>fixture.debugElement</code>
</li>
</ul>

<h3>
  
  
  Testing Component Inputs and Outputs
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">BusinessCardComponent</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">component</span><span class="p">:</span> <span class="nx">BusinessCardComponent</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">fixture</span><span class="p">:</span> <span class="nx">ComponentFixture</span><span class="o">&lt;</span><span class="nx">BusinessCardComponent</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">declarations</span><span class="p">:</span> <span class="p">[</span><span class="nx">BusinessCardComponent</span><span class="p">]</span>
    <span class="p">}).</span><span class="nf">compileComponents</span><span class="p">();</span>

    <span class="nx">fixture</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">createComponent</span><span class="p">(</span><span class="nx">BusinessCardComponent</span><span class="p">);</span>
    <span class="nx">component</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">componentInstance</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should display business name</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">component</span><span class="p">.</span><span class="nx">business</span> <span class="o">=</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test Business</span><span class="dl">'</span> <span class="p">};</span>
    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">compiled</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">nativeElement</span><span class="p">;</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">compiled</span><span class="p">.</span><span class="nx">textContent</span><span class="p">).</span><span class="nf">toContain</span><span class="p">(</span><span class="dl">'</span><span class="s1">Test Business</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should emit event on click</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">spyOn</span><span class="p">(</span><span class="nx">component</span><span class="p">.</span><span class="nx">businessSelected</span><span class="p">,</span> <span class="dl">'</span><span class="s1">emit</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">component</span><span class="p">.</span><span class="nx">business</span> <span class="o">=</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test Business</span><span class="dl">'</span> <span class="p">};</span>
    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">button</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">nativeElement</span><span class="p">.</span><span class="nf">querySelector</span><span class="p">(</span><span class="dl">'</span><span class="s1">button</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">button</span><span class="p">.</span><span class="nf">click</span><span class="p">();</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">component</span><span class="p">.</span><span class="nx">businessSelected</span><span class="p">.</span><span class="nx">emit</span><span class="p">).</span><span class="nf">toHaveBeenCalledWith</span><span class="p">(</span><span class="nx">component</span><span class="p">.</span><span class="nx">business</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Testing Component Forms
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">BusinessFormComponent</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">component</span><span class="p">:</span> <span class="nx">BusinessFormComponent</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">fixture</span><span class="p">:</span> <span class="nx">ComponentFixture</span><span class="o">&lt;</span><span class="nx">BusinessFormComponent</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(</span><span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">declarations</span><span class="p">:</span> <span class="p">[</span><span class="nx">BusinessFormComponent</span><span class="p">],</span>
      <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">ReactiveFormsModule</span><span class="p">]</span>
    <span class="p">}).</span><span class="nf">compileComponents</span><span class="p">();</span>

    <span class="nx">fixture</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">createComponent</span><span class="p">(</span><span class="nx">BusinessFormComponent</span><span class="p">);</span>
    <span class="nx">component</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">componentInstance</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should validate required fields</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">form</span> <span class="o">=</span> <span class="nx">component</span><span class="p">.</span><span class="nx">businessForm</span><span class="p">;</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">form</span><span class="p">.</span><span class="nx">valid</span><span class="p">).</span><span class="nf">toBeFalsy</span><span class="p">();</span>

    <span class="nx">form</span><span class="p">.</span><span class="nf">patchValue</span><span class="p">({</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test Business</span><span class="dl">'</span> <span class="p">});</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">form</span><span class="p">.</span><span class="nx">valid</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should submit form with valid data</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">spyOn</span><span class="p">(</span><span class="nx">component</span><span class="p">.</span><span class="nx">formSubmit</span><span class="p">,</span> <span class="dl">'</span><span class="s1">emit</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">component</span><span class="p">.</span><span class="nx">businessForm</span><span class="p">.</span><span class="nf">patchValue</span><span class="p">({</span>
      <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test Business</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">email</span><span class="p">:</span> <span class="dl">'</span><span class="s1">test@example.com</span><span class="dl">'</span>
    <span class="p">});</span>

    <span class="nx">component</span><span class="p">.</span><span class="nf">onSubmit</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">component</span><span class="p">.</span><span class="nx">formSubmit</span><span class="p">.</span><span class="nx">emit</span><span class="p">).</span><span class="nf">toHaveBeenCalled</span><span class="p">();</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Service Testing
</h2>

<p>Test Angular services:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TestBed</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HttpClientTestingModule</span><span class="p">,</span> <span class="nx">HttpTestingController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">BusinessService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./business.service</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">environment</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">src/environments/environment</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">BusinessService</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">service</span><span class="p">:</span> <span class="nx">BusinessService</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">httpMock</span><span class="p">:</span> <span class="nx">HttpTestingController</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">HttpClientTestingModule</span><span class="p">],</span>
      <span class="na">providers</span><span class="p">:</span> <span class="p">[</span><span class="nx">BusinessService</span><span class="p">]</span>
    <span class="p">});</span>
    <span class="nx">service</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">BusinessService</span><span class="p">);</span>
    <span class="nx">httpMock</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">HttpTestingController</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">afterEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">httpMock</span><span class="p">.</span><span class="nf">verify</span><span class="p">();</span> <span class="c1">// Verify no outstanding requests</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should fetch businesses</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">mockBusinesses</span> <span class="o">=</span> <span class="p">[{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Business 1</span><span class="dl">'</span> <span class="p">}];</span>

    <span class="nx">service</span><span class="p">.</span><span class="nc">GetBusinesses</span><span class="p">({}).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">businesses</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">businesses</span><span class="p">.</span><span class="nx">data</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">(</span><span class="nx">mockBusinesses</span><span class="p">);</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">req</span> <span class="o">=</span> <span class="nx">httpMock</span><span class="p">.</span><span class="nf">expectOne</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">environment</span><span class="p">.</span><span class="nx">ApiUrl</span><span class="p">}</span><span class="s2">business/get`</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nx">method</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nf">flush</span><span class="p">({</span> <span class="na">data</span><span class="p">:</span> <span class="nx">mockBusinesses</span> <span class="p">});</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should handle errors</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">service</span><span class="p">.</span><span class="nc">GetBusinesses</span><span class="p">({}).</span><span class="nf">subscribe</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">fail</span><span class="p">(</span><span class="dl">'</span><span class="s1">should have failed</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">error</span> <span class="o">=&gt;</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">status</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">500</span><span class="p">)</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">req</span> <span class="o">=</span> <span class="nx">httpMock</span><span class="p">.</span><span class="nf">expectOne</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">environment</span><span class="p">.</span><span class="nx">ApiUrl</span><span class="p">}</span><span class="s2">business/get`</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nf">flush</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span> <span class="na">statusText</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Server Error</span><span class="dl">'</span> <span class="p">});</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should create business</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">newBusiness</span> <span class="o">=</span> <span class="p">{</span> <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">New Business</span><span class="dl">'</span> <span class="p">};</span>
    <span class="kd">const</span> <span class="nx">createdBusiness</span> <span class="o">=</span> <span class="p">{</span> <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span> <span class="p">...</span><span class="nx">newBusiness</span> <span class="p">};</span>

    <span class="nx">service</span><span class="p">.</span><span class="nc">SaveBusiness</span><span class="p">(</span><span class="nx">newBusiness</span><span class="p">,</span> <span class="mi">0</span><span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">business</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">business</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">(</span><span class="nx">createdBusiness</span><span class="p">);</span>
    <span class="p">});</span>

    <span class="kd">const</span> <span class="nx">req</span> <span class="o">=</span> <span class="nx">httpMock</span><span class="p">.</span><span class="nf">expectOne</span><span class="p">(</span><span class="s2">`</span><span class="p">${</span><span class="nx">environment</span><span class="p">.</span><span class="nx">ApiUrl</span><span class="p">}</span><span class="s2">business/0/details`</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nx">method</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nx">body</span><span class="p">).</span><span class="nf">toEqual</span><span class="p">(</span><span class="nx">newBusiness</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nf">flush</span><span class="p">(</span><span class="nx">createdBusiness</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Testing Services with Dependencies
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">UserService</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">service</span><span class="p">:</span> <span class="nx">UserService</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">httpMock</span><span class="p">:</span> <span class="nx">HttpTestingController</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">loggerService</span><span class="p">:</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">LoggerService</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">loggerSpy</span> <span class="o">=</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nf">createSpyObj</span><span class="p">(</span><span class="dl">'</span><span class="s1">LoggerService</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span><span class="dl">'</span><span class="s1">log</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">error</span><span class="dl">'</span><span class="p">]);</span>

    <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">HttpClientTestingModule</span><span class="p">],</span>
      <span class="na">providers</span><span class="p">:</span> <span class="p">[</span>
        <span class="nx">UserService</span><span class="p">,</span>
        <span class="p">{</span> <span class="na">provide</span><span class="p">:</span> <span class="nx">LoggerService</span><span class="p">,</span> <span class="na">useValue</span><span class="p">:</span> <span class="nx">loggerSpy</span> <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">});</span>

    <span class="nx">service</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">UserService</span><span class="p">);</span>
    <span class="nx">httpMock</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">HttpTestingController</span><span class="p">);</span>
    <span class="nx">loggerService</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">LoggerService</span><span class="p">)</span> <span class="k">as</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">LoggerService</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should log errors</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">service</span><span class="p">.</span><span class="nc">GetUser</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">subscribe</span><span class="p">(</span>
      <span class="p">()</span> <span class="o">=&gt;</span> <span class="nf">fail</span><span class="p">(</span><span class="dl">'</span><span class="s1">should have failed</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">error</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">loggerService</span><span class="p">.</span><span class="nx">error</span><span class="p">).</span><span class="nf">toHaveBeenCalled</span><span class="p">();</span>
      <span class="p">}</span>
    <span class="p">);</span>

    <span class="kd">const</span> <span class="nx">req</span> <span class="o">=</span> <span class="nx">httpMock</span><span class="p">.</span><span class="nf">expectOne</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/users/1</span><span class="dl">'</span><span class="p">);</span>
    <span class="nx">req</span><span class="p">.</span><span class="nf">flush</span><span class="p">(</span><span class="dl">'</span><span class="s1">Error</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span> <span class="na">status</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span> <span class="na">statusText</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Server Error</span><span class="dl">'</span> <span class="p">});</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Pipe Testing
</h2>

<p>Test custom pipes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TruncatePipe</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./truncate.pipe</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">TruncatePipe</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">pipe</span><span class="p">:</span> <span class="nx">TruncatePipe</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">pipe</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">TruncatePipe</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should create</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">pipe</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should truncate long text</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">pipe</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="dl">'</span><span class="s1">This is a very long text</span><span class="dl">'</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">This is a ...</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should return original text if shorter than limit</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">pipe</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="dl">'</span><span class="s1">Short</span><span class="dl">'</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Short</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should handle null values</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">pipe</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="kc">null</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">''</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Testing Async Pipes
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">CurrencyPipe</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">TestBed</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">CurrencyPipe</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">pipe</span><span class="p">:</span> <span class="nx">CurrencyPipe</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">providers</span><span class="p">:</span> <span class="p">[</span><span class="nx">CurrencyPipe</span><span class="p">]</span>
    <span class="p">});</span>
    <span class="nx">pipe</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">CurrencyPipe</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should format currency</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="nx">pipe</span><span class="p">.</span><span class="nf">transform</span><span class="p">(</span><span class="mi">1000</span><span class="p">,</span> <span class="dl">'</span><span class="s1">USD</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toContain</span><span class="p">(</span><span class="dl">'</span><span class="s1">1,000</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Directive Testing
</h2>

<p>Test custom directives:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">Component</span><span class="p">,</span> <span class="nx">DebugElement</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">ComponentFixture</span><span class="p">,</span> <span class="nx">TestBed</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">By</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/platform-browser</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HighlightDirective</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./highlight.directive</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Component</span><span class="p">({</span>
  <span class="na">template</span><span class="p">:</span> <span class="dl">'</span><span class="s1">&lt;p appHighlight="yellow"&gt;Test&lt;/p&gt;</span><span class="dl">'</span>
<span class="p">})</span>
<span class="kd">class</span> <span class="nc">TestComponent</span> <span class="p">{}</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">HighlightDirective</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">fixture</span><span class="p">:</span> <span class="nx">ComponentFixture</span><span class="o">&lt;</span><span class="nx">TestComponent</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">element</span><span class="p">:</span> <span class="nx">DebugElement</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">declarations</span><span class="p">:</span> <span class="p">[</span><span class="nx">TestComponent</span><span class="p">,</span> <span class="nx">HighlightDirective</span><span class="p">]</span>
    <span class="p">});</span>
    <span class="nx">fixture</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">createComponent</span><span class="p">(</span><span class="nx">TestComponent</span><span class="p">);</span>
    <span class="nx">element</span> <span class="o">=</span> <span class="nx">fixture</span><span class="p">.</span><span class="nx">debugElement</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="nx">By</span><span class="p">.</span><span class="nf">directive</span><span class="p">(</span><span class="nx">HighlightDirective</span><span class="p">));</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should create directive</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">element</span><span class="p">).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should apply highlight on mouseenter</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">element</span><span class="p">.</span><span class="nf">triggerEventHandler</span><span class="p">(</span><span class="dl">'</span><span class="s1">mouseenter</span><span class="dl">'</span><span class="p">,</span> <span class="kc">null</span><span class="p">);</span>
    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">element</span><span class="p">.</span><span class="nx">nativeElement</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">backgroundColor</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">yellow</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should remove highlight on mouseleave</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">element</span><span class="p">.</span><span class="nf">triggerEventHandler</span><span class="p">(</span><span class="dl">'</span><span class="s1">mouseenter</span><span class="dl">'</span><span class="p">,</span> <span class="kc">null</span><span class="p">);</span>
    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>

    <span class="nx">element</span><span class="p">.</span><span class="nf">triggerEventHandler</span><span class="p">(</span><span class="dl">'</span><span class="s1">mouseleave</span><span class="dl">'</span><span class="p">,</span> <span class="kc">null</span><span class="p">);</span>
    <span class="nx">fixture</span><span class="p">.</span><span class="nf">detectChanges</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">element</span><span class="p">.</span><span class="nx">nativeElement</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">backgroundColor</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">''</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Async Testing
</h2>

<p>Test asynchronous operations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">fakeAsync</span><span class="p">,</span> <span class="nx">tick</span><span class="p">,</span> <span class="nx">flush</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="k">of</span><span class="p">,</span> <span class="nx">delay</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Async Operations</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should handle async operations with fakeAsync</span><span class="dl">'</span><span class="p">,</span> <span class="nf">fakeAsync</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="nx">value</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>

    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">value</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
    <span class="p">},</span> <span class="mi">1000</span><span class="p">);</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">value</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="nf">tick</span><span class="p">(</span><span class="mi">1000</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">value</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
  <span class="p">}));</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should handle multiple timers</span><span class="dl">'</span><span class="p">,</span> <span class="nf">fakeAsync</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="nx">value</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span>

    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">value</span><span class="o">++</span><span class="p">,</span> <span class="mi">100</span><span class="p">);</span>
    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">value</span><span class="o">++</span><span class="p">,</span> <span class="mi">200</span><span class="p">);</span>
    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="nx">value</span><span class="o">++</span><span class="p">,</span> <span class="mi">300</span><span class="p">);</span>

    <span class="nf">expect</span><span class="p">(</span><span class="nx">value</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">0</span><span class="p">);</span>
    <span class="nf">tick</span><span class="p">(</span><span class="mi">300</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">value</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="mi">3</span><span class="p">);</span>
  <span class="p">}));</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should flush all timers</span><span class="dl">'</span><span class="p">,</span> <span class="nf">fakeAsync</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="nx">value</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>

    <span class="nf">setTimeout</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">value</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
    <span class="p">},</span> <span class="mi">1000</span><span class="p">);</span>

    <span class="nf">flush</span><span class="p">();</span> <span class="c1">// Flush all pending timers</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">value</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
  <span class="p">}));</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should test observables</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">done</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">service</span><span class="p">.</span><span class="nf">getData</span><span class="p">().</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">data</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nf">expect</span><span class="p">(</span><span class="nx">data</span><span class="p">).</span><span class="nf">toBeDefined</span><span class="p">();</span>
      <span class="nf">done</span><span class="p">();</span>
    <span class="p">});</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should test observables with fakeAsync</span><span class="dl">'</span><span class="p">,</span> <span class="nf">fakeAsync</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="na">result</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span>

    <span class="nx">service</span><span class="p">.</span><span class="nf">getData</span><span class="p">().</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">data</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">result</span> <span class="o">=</span> <span class="nx">data</span><span class="p">;</span>
    <span class="p">});</span>

    <span class="nf">tick</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toBeDefined</span><span class="p">();</span>
  <span class="p">}));</span>
<span class="p">});</span>
</code></pre>

</div>



<h3>
  
  
  Testing RxJS Operators
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="k">of</span><span class="p">,</span> <span class="nx">throwError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">delay</span><span class="p">,</span> <span class="nx">catchError</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">rxjs/operators</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">RxJS Testing</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should test delayed observables</span><span class="dl">'</span><span class="p">,</span> <span class="nf">fakeAsync</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">let</span> <span class="na">result</span><span class="p">:</span> <span class="kr">any</span><span class="p">;</span>

    <span class="k">of</span><span class="p">(</span><span class="dl">'</span><span class="s1">data</span><span class="dl">'</span><span class="p">).</span><span class="nf">pipe</span><span class="p">(</span><span class="nf">delay</span><span class="p">(</span><span class="mi">1000</span><span class="p">)).</span><span class="nf">subscribe</span><span class="p">(</span><span class="nx">data</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="nx">result</span> <span class="o">=</span> <span class="nx">data</span><span class="p">;</span>
    <span class="p">});</span>

    <span class="nf">tick</span><span class="p">(</span><span class="mi">1000</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">result</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">data</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">}));</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should test error handling</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">done</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nf">throwError</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">new</span> <span class="nc">Error</span><span class="p">(</span><span class="dl">'</span><span class="s1">Test error</span><span class="dl">'</span><span class="p">))</span>
      <span class="p">.</span><span class="nf">pipe</span><span class="p">(</span>
        <span class="nf">catchError</span><span class="p">(</span><span class="nx">error</span> <span class="o">=&gt;</span> <span class="p">{</span>
          <span class="nf">expect</span><span class="p">(</span><span class="nx">error</span><span class="p">.</span><span class="nx">message</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Test error</span><span class="dl">'</span><span class="p">);</span>
          <span class="nf">done</span><span class="p">();</span>
          <span class="k">return</span> <span class="k">of</span><span class="p">(</span><span class="kc">null</span><span class="p">);</span>
        <span class="p">})</span>
      <span class="p">)</span>
      <span class="p">.</span><span class="nf">subscribe</span><span class="p">();</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Testing Guards
</h2>

<p>Test route guards:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TestBed</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">Router</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/router</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">AuthGuard</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./auth.guard</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">AuthService</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./auth.service</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">AuthGuard</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">guard</span><span class="p">:</span> <span class="nx">AuthGuard</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">authService</span><span class="p">:</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">AuthService</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">router</span><span class="p">:</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">Router</span><span class="o">&gt;</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">authSpy</span> <span class="o">=</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nf">createSpyObj</span><span class="p">(</span><span class="dl">'</span><span class="s1">AuthService</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span><span class="dl">'</span><span class="s1">isAuthenticated</span><span class="dl">'</span><span class="p">]);</span>
    <span class="kd">const</span> <span class="nx">routerSpy</span> <span class="o">=</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nf">createSpyObj</span><span class="p">(</span><span class="dl">'</span><span class="s1">Router</span><span class="dl">'</span><span class="p">,</span> <span class="p">[</span><span class="dl">'</span><span class="s1">navigate</span><span class="dl">'</span><span class="p">]);</span>

    <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">providers</span><span class="p">:</span> <span class="p">[</span>
        <span class="nx">AuthGuard</span><span class="p">,</span>
        <span class="p">{</span> <span class="na">provide</span><span class="p">:</span> <span class="nx">AuthService</span><span class="p">,</span> <span class="na">useValue</span><span class="p">:</span> <span class="nx">authSpy</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">provide</span><span class="p">:</span> <span class="nx">Router</span><span class="p">,</span> <span class="na">useValue</span><span class="p">:</span> <span class="nx">routerSpy</span> <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">});</span>

    <span class="nx">guard</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">AuthGuard</span><span class="p">);</span>
    <span class="nx">authService</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">AuthService</span><span class="p">)</span> <span class="k">as</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">AuthService</span><span class="o">&gt;</span><span class="p">;</span>
    <span class="nx">router</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">Router</span><span class="p">)</span> <span class="k">as</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">Router</span><span class="o">&gt;</span><span class="p">;</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should allow access when authenticated</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">authService</span><span class="p">.</span><span class="nx">isAuthenticated</span><span class="p">.</span><span class="nx">and</span><span class="p">.</span><span class="nf">returnValue</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">guard</span><span class="p">.</span><span class="nf">canActivate</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">true</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should redirect when not authenticated</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">authService</span><span class="p">.</span><span class="nx">isAuthenticated</span><span class="p">.</span><span class="nx">and</span><span class="p">.</span><span class="nf">returnValue</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">guard</span><span class="p">.</span><span class="nf">canActivate</span><span class="p">()).</span><span class="nf">toBe</span><span class="p">(</span><span class="kc">false</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">router</span><span class="p">.</span><span class="nx">navigate</span><span class="p">).</span><span class="nf">toHaveBeenCalledWith</span><span class="p">([</span><span class="dl">'</span><span class="s1">/login</span><span class="dl">'</span><span class="p">]);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Testing Interceptors
</h2>

<p>Test HTTP interceptors:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">TestBed</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HttpClientTestingModule</span><span class="p">,</span> <span class="nx">HttpTestingController</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http/testing</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">HTTP_INTERCEPTORS</span><span class="p">,</span> <span class="nx">HttpClient</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">AuthInterceptor</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./auth.interceptor</span><span class="dl">'</span><span class="p">;</span>

<span class="nf">describe</span><span class="p">(</span><span class="dl">'</span><span class="s1">AuthInterceptor</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">let</span> <span class="na">http</span><span class="p">:</span> <span class="nx">HttpClient</span><span class="p">;</span>
  <span class="kd">let</span> <span class="na">httpMock</span><span class="p">:</span> <span class="nx">HttpTestingController</span><span class="p">;</span>

  <span class="nf">beforeEach</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
      <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">HttpClientTestingModule</span><span class="p">],</span>
      <span class="na">providers</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
          <span class="na">provide</span><span class="p">:</span> <span class="nx">HTTP_INTERCEPTORS</span><span class="p">,</span>
          <span class="na">useClass</span><span class="p">:</span> <span class="nx">AuthInterceptor</span><span class="p">,</span>
          <span class="na">multi</span><span class="p">:</span> <span class="kc">true</span>
        <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">});</span>

    <span class="nx">http</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">HttpClient</span><span class="p">);</span>
    <span class="nx">httpMock</span> <span class="o">=</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">inject</span><span class="p">(</span><span class="nx">HttpTestingController</span><span class="p">);</span>
  <span class="p">});</span>

  <span class="nf">it</span><span class="p">(</span><span class="dl">'</span><span class="s1">should add authorization header</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="nx">http</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/data</span><span class="dl">'</span><span class="p">).</span><span class="nf">subscribe</span><span class="p">();</span>

    <span class="kd">const</span> <span class="nx">req</span> <span class="o">=</span> <span class="nx">httpMock</span><span class="p">.</span><span class="nf">expectOne</span><span class="p">(</span><span class="dl">'</span><span class="s1">/api/data</span><span class="dl">'</span><span class="p">);</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nf">has</span><span class="p">(</span><span class="dl">'</span><span class="s1">Authorization</span><span class="dl">'</span><span class="p">)).</span><span class="nf">toBeTruthy</span><span class="p">();</span>
    <span class="nf">expect</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">request</span><span class="p">.</span><span class="nx">headers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="dl">'</span><span class="s1">Authorization</span><span class="dl">'</span><span class="p">)).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">Bearer token</span><span class="dl">'</span><span class="p">);</span>
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>

</div>



<h2>
  
  
  Testing Best Practices
</h2>

<ol>
<li><strong>Write tests for each component, service, pipe, and directive</strong></li>
<li>
<strong>Use TestBed</strong> - For component and service testing</li>
<li>
<strong>Mock dependencies</strong> - With Jasmine spies</li>
<li>
<strong>Use HttpClientTestingModule</strong> - For HTTP testing</li>
<li><strong>Test both success and error scenarios</strong></li>
<li>
<strong>Use fakeAsync and tick</strong> - For async operations</li>
<li>
<strong>Query DOM elements</strong> - With debugElement</li>
<li><strong>Test component inputs, outputs, and events</strong></li>
<li>
<strong>Maintain high test coverage</strong> - Aim for 80%+</li>
<li><strong>Keep tests isolated and independent</strong></li>
<li>
<strong>Use descriptive test names</strong> - Clear what is being tested</li>
<li>
<strong>Clean up after each test</strong> - Use afterEach for cleanup</li>
<li>
<strong>Use beforeEach for setup</strong> - Common test configuration</li>
<li>
<strong>Test user interactions</strong> - Click, input, form submissions</li>
<li>
<strong>Test edge cases</strong> - Null values, empty arrays, errors</li>
</ol>

<h2>
  
  
  Common Testing Patterns
</h2>

<h3>
  
  
  Test Setup Helper
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">function</span> <span class="nf">createComponentFixture</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span>
  <span class="nx">component</span><span class="p">:</span> <span class="nx">Type</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="nx">imports</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[],</span>
  <span class="nx">providers</span><span class="p">:</span> <span class="kr">any</span><span class="p">[]</span> <span class="o">=</span> <span class="p">[]</span>
<span class="p">):</span> <span class="nx">ComponentFixture</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="nx">TestBed</span><span class="p">.</span><span class="nf">configureTestingModule</span><span class="p">({</span>
    <span class="na">declarations</span><span class="p">:</span> <span class="p">[</span><span class="nx">component</span><span class="p">],</span>
    <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">HttpClientTestingModule</span><span class="p">,</span> <span class="p">...</span><span class="nx">imports</span><span class="p">],</span>
    <span class="nx">providers</span>
  <span class="p">}).</span><span class="nf">compileComponents</span><span class="p">();</span>

  <span class="k">return</span> <span class="nx">TestBed</span><span class="p">.</span><span class="nf">createComponent</span><span class="p">(</span><span class="nx">component</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Mock Factory
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">function</span> <span class="nf">createMockService</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">methods</span><span class="p">:</span> <span class="kr">string</span><span class="p">[]):</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">SpyObj</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span> <span class="p">{</span>
  <span class="k">return</span> <span class="nx">jasmine</span><span class="p">.</span><span class="nx">createSpyObj</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">methods</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Test Data Builders
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">export</span> <span class="kd">class</span> <span class="nc">BusinessBuilder</span> <span class="p">{</span>
  <span class="k">private</span> <span class="nx">business</span><span class="p">:</span> <span class="kr">any</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">id</span><span class="p">:</span> <span class="mi">1</span><span class="p">,</span>
    <span class="na">name</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Test Business</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">isActive</span><span class="p">:</span> <span class="kc">true</span>
  <span class="p">};</span>

  <span class="nf">withId</span><span class="p">(</span><span class="nx">id</span><span class="p">:</span> <span class="kr">number</span><span class="p">):</span> <span class="nx">BusinessBuilder</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">business</span><span class="p">.</span><span class="nx">id</span> <span class="o">=</span> <span class="nx">id</span><span class="p">;</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nf">withName</span><span class="p">(</span><span class="nx">name</span><span class="p">:</span> <span class="kr">string</span><span class="p">):</span> <span class="nx">BusinessBuilder</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">business</span><span class="p">.</span><span class="nx">name</span> <span class="o">=</span> <span class="nx">name</span><span class="p">;</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nf">inactive</span><span class="p">():</span> <span class="nx">BusinessBuilder</span> <span class="p">{</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">business</span><span class="p">.</span><span class="nx">isActive</span> <span class="o">=</span> <span class="kc">false</span><span class="p">;</span>
    <span class="k">return</span> <span class="k">this</span><span class="p">;</span>
  <span class="p">}</span>

  <span class="nf">build</span><span class="p">():</span> <span class="kr">any</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="p">...</span><span class="k">this</span><span class="p">.</span><span class="nx">business</span> <span class="p">};</span>
  <span class="p">}</span>
<span class="p">}</span>

<span class="c1">// Usage</span>
<span class="kd">const</span> <span class="nx">business</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">BusinessBuilder</span><span class="p">()</span>
  <span class="p">.</span><span class="nf">withId</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">withName</span><span class="p">(</span><span class="dl">'</span><span class="s1">Test</span><span class="dl">'</span><span class="p">)</span>
  <span class="p">.</span><span class="nf">build</span><span class="p">();</span>
</code></pre>

</div>



<h2>
  
  
  Karma Configuration
</h2>

<p>Configure Karma for your tests:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// karma.conf.js</span>
<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">config</span><span class="p">)</span> <span class="p">{</span>
  <span class="nx">config</span><span class="p">.</span><span class="nf">set</span><span class="p">({</span>
    <span class="na">basePath</span><span class="p">:</span> <span class="dl">''</span><span class="p">,</span>
    <span class="na">frameworks</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">jasmine</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">@angular-devkit/build-angular</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">plugins</span><span class="p">:</span> <span class="p">[</span>
      <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">karma-jasmine</span><span class="dl">'</span><span class="p">),</span>
      <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">karma-chrome-launcher</span><span class="dl">'</span><span class="p">),</span>
      <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">karma-jasmine-html-reporter</span><span class="dl">'</span><span class="p">),</span>
      <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">karma-coverage</span><span class="dl">'</span><span class="p">),</span>
      <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">@angular-devkit/build-angular/plugins/karma</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">],</span>
    <span class="na">client</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">clearContext</span><span class="p">:</span> <span class="kc">false</span>
    <span class="p">},</span>
    <span class="na">coverageReporter</span><span class="p">:</span> <span class="p">{</span>
      <span class="na">dir</span><span class="p">:</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">path</span><span class="dl">'</span><span class="p">).</span><span class="nf">join</span><span class="p">(</span><span class="nx">__dirname</span><span class="p">,</span> <span class="dl">'</span><span class="s1">./coverage</span><span class="dl">'</span><span class="p">),</span>
      <span class="na">subdir</span><span class="p">:</span> <span class="dl">'</span><span class="s1">.</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">reporters</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">html</span><span class="dl">'</span> <span class="p">},</span>
        <span class="p">{</span> <span class="na">type</span><span class="p">:</span> <span class="dl">'</span><span class="s1">text-summary</span><span class="dl">'</span> <span class="p">}</span>
      <span class="p">]</span>
    <span class="p">},</span>
    <span class="na">reporters</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">progress</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">kjhtml</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">port</span><span class="p">:</span> <span class="mi">9876</span><span class="p">,</span>
    <span class="na">colors</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">logLevel</span><span class="p">:</span> <span class="nx">config</span><span class="p">.</span><span class="nx">LOG_INFO</span><span class="p">,</span>
    <span class="na">autoWatch</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span>
    <span class="na">browsers</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">Chrome</span><span class="dl">'</span><span class="p">],</span>
    <span class="na">singleRun</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
    <span class="na">restartOnFileChange</span><span class="p">:</span> <span class="kc">true</span>
  <span class="p">});</span>
<span class="p">};</span>
</code></pre>

</div>



<h2>
  
  
  Resources and Further Reading
</h2>

<ul>
<li>
<strong><a href="https://marufrahman.live/blog/angular-testing-jasmine-karma" rel="noopener noreferrer">📚 Full Angular Testing Guide</a></strong> - Complete tutorial with advanced examples, troubleshooting, and best practices</li>
<li>
<strong><a href="https://marufrahman.live/blog/angular-services-dependency-injection" rel="noopener noreferrer">Angular Services Guide</a></strong> - Testing services with dependency injection</li>
<li>
<strong><a href="https://marufrahman.live/blog/angular-reactive-forms-complete-guide" rel="noopener noreferrer">Angular Reactive Forms Guide</a></strong> - Testing forms and validators</li>
<li>
<strong><a href="https://marufrahman.live/blog/angular-http-client-interceptors" rel="noopener noreferrer">Angular HTTP Client Guide</a></strong> - Testing HTTP calls and interceptors</li>
<li>
<a href="https://angular.io/guide/testing" rel="noopener noreferrer">Angular Testing Documentation</a> - Official Angular testing guide</li>
<li>
<a href="https://jasmine.github.io/" rel="noopener noreferrer">Jasmine Documentation</a> - Jasmine testing framework</li>
<li>
<a href="https://karma-runner.github.io/" rel="noopener noreferrer">Karma Documentation</a> - Karma test runner</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p><strong>Testing</strong> is crucial for building reliable Angular applications. With <strong>Jasmine</strong> and <strong>Karma</strong>, you can write comprehensive tests for all parts of your application. Following these patterns ensures code quality and enables confident development.</p>

<p><strong>Key Takeaways:</strong></p>

<ul>
<li>
<strong>Jasmine</strong> - Behavior-driven testing framework</li>
<li>
<strong>Karma</strong> - Test runner for browser-based testing</li>
<li>
<strong>TestBed</strong> - Configure test modules and create components</li>
<li>
<strong>Component Testing</strong> - Test components in isolation</li>
<li>
<strong>Service Testing</strong> - Test business logic and HTTP calls</li>
<li>
<strong>Pipe &amp; Directive Testing</strong> - Test custom transformations</li>
<li>
<strong>Async Testing</strong> - Use fakeAsync and tick for async operations</li>
<li>
<strong>Mocking</strong> - Use Jasmine spies for dependencies</li>
<li>
<strong>Best Practices</strong> - High coverage, isolated tests, descriptive names</li>
</ul>

<p>Whether you're building a simple dashboard or a complex enterprise application, comprehensive testing provides the confidence you need to refactor and deploy with peace of mind.</p>




<p><strong>What's your experience with Angular Testing? Share your tips and tricks in the comments below!</strong> 🚀</p>




<blockquote>
<p>💡 <strong>Looking for more details?</strong> This is a condensed version of my comprehensive guide. Read the <a href="https://marufrahman.live/blog/angular-testing-jasmine-karma" rel="noopener noreferrer">full article on my blog</a> for additional examples, advanced patterns, troubleshooting tips, and more in-depth explanations.</p>
</blockquote>

<p><em>If you found this guide helpful, consider checking out my other articles on <a href="https://marufrahman.live/blog" rel="noopener noreferrer">Angular development</a> and <a href="https://marufrahman.live" rel="noopener noreferrer">frontend development best practices</a>.</em></p>

