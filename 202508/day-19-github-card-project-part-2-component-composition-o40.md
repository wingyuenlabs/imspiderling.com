---
Title: Day 19 - Github Card project Part 2 - Component Composition
Description: 
Author: Connie Leung
Date: 2025-08-30T21:07:15.000Z
Robots: noindex,nofollow
Template: index
---
<p>Day 19 - Github Card project Part 2 - Component Composition</p>

<h2>
  
  
  Table of Contents
</h2>

<ul>
<li>Create a GithubProfileCard component</li>
<li>Create a GithubProfileList component</li>
<li>Github Repositories</li>
<li>Resources</li>
</ul>

<p>The demo displays Github users in a card layout. There is a <code>GithubProfileList</code> that iterates a list of user names and displays each user in a <code>GithubProfileCard</code>.</p>

<h2>
  
  
  Create a GithubProfileCard component
</h2>

<h3>
  
  
  Vue 3 application
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// components/GithubProfileCard.vue</span>

<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">setup</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">useGithubProfile</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@/composables/useGithubProfile.ts</span><span class="dl">'</span>

<span class="kd">type</span> <span class="nx">Prop</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">username</span><span class="p">:</span> <span class="kr">string</span>
<span class="p">}</span>

<span class="kd">const</span> <span class="p">{</span> <span class="nx">username</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">defineProps</span><span class="o">&lt;</span><span class="nx">Prop</span><span class="o">&gt;</span><span class="p">()</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>



<p>The <code>GithubProfileCard</code> expects a <code>username</code> prop that can assign to the <code>username</code> ref of the <code>useGithubProfile</code> composable. The project is written in TypeScript, so <code>defineProps</code> macro can accept a prop type.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code>
<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">setup</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
<span class="kd">const</span> <span class="p">{</span> <span class="na">username</span><span class="p">:</span> <span class="nx">name</span><span class="p">,</span> <span class="nx">profile</span><span class="p">,</span> <span class="nx">error</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">useGithubProfile</span><span class="p">()</span>

<span class="nx">name</span><span class="p">.</span><span class="nx">value</span> <span class="o">=</span> <span class="nx">username</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>



<p>Destructure <code>username</code>, <code>profile</code>, and <code>error</code> refs from the <code>useGithubProfile</code> composable. <code>username</code> is aliased to <code>name</code> to not conflicted with the <code>username</code> prop.</p>

<p>When the <code>name</code> ref is updated, the composable retrieves the Github profile and the value is available in the the <code>profile</code> ref.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;template&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">v-if=</span><span class="s">"profile"</span><span class="nt">&gt;</span>
        <span class="nt">&lt;p&gt;</span>Username: {{ profile.login }}<span class="nt">&lt;/p&gt;</span>
        <span class="nt">&lt;p&gt;</span>Name: {{ profile.name }}<span class="nt">&lt;/p&gt;</span>
        <span class="nt">&lt;p&gt;</span>Bio: {{ profile.bio || 'N/A' }}<span class="nt">&lt;/p&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
    <span class="nt">&lt;div</span> <span class="na">v-else-if=</span><span class="s">"error"</span><span class="nt">&gt;</span>
        Error: {{ error }}
    <span class="nt">&lt;/div&gt;</span>
<span class="nt">&lt;/template&gt;</span>
</code></pre>

</div>



<p>The <code>v-if</code> directive checks the value of the <code>profile</code> ref. The template displays the login, name and bio when the profile is defined. The <code>v-else-if</code> directive examines the value of <code>error</code> and displays the error message When itis not blank.</p>

<h3>
  
  
  SvelteKit application
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// github-profile.type.ts</span>

<span class="k">export</span> <span class="kd">type</span> <span class="nx">GithubProfileItem</span> <span class="o">=</span> <span class="p">{</span> 
    <span class="na">key</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span> 
    <span class="nl">profile</span><span class="p">?:</span> <span class="nx">GithubProfile</span><span class="p">;</span> 
    <span class="nl">error</span><span class="p">?:</span> <span class="kr">string</span> 
<span class="p">}</span>
</code></pre>

</div>



<p>The  <code>GithubProfileItem</code> type consists of <code>profile</code> or <code>error</code> property.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// github-profile-card.svelte</span>

<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
    <span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">GithubProfileItem</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./github-profile-item.type</span><span class="dl">'</span><span class="p">;</span>

    <span class="kd">type</span> <span class="nx">Props</span> <span class="o">=</span> <span class="p">{</span>
        <span class="na">profile</span><span class="p">:</span> <span class="nx">GithubProfileItem</span>
    <span class="p">};</span>

    <span class="kd">const</span> <span class="p">{</span> <span class="na">profile</span><span class="p">:</span> <span class="nx">result</span> <span class="p">}:</span> <span class="nx">Props</span> <span class="o">=</span> <span class="nf">$props</span><span class="p">();</span>
    <span class="kd">const</span> <span class="p">{</span> <span class="nx">profile</span><span class="p">,</span> <span class="nx">error</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">result</span><span class="p">;</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>



<p>In this component, <code>$props()</code> is casted to the <code>Props</code> type. The <code>prop</code> is destructured to <code>profile</code> and <code>error</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code>{#if profile}
    <span class="nt">&lt;div&gt;</span>
        <span class="nt">&lt;p&gt;</span>Username: {profile.login}<span class="nt">&lt;/p&gt;</span>
        <span class="nt">&lt;p&gt;</span>Name: {profile.name}<span class="nt">&lt;/p&gt;</span>
        <span class="nt">&lt;p&gt;</span>Bio: {profile.bio || 'N/A'}<span class="nt">&lt;/p&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
{:else if error}
    <span class="nt">&lt;div&gt;</span>
        <span class="nt">&lt;p&gt;</span>Error: {error}<span class="nt">&lt;/p&gt;</span>
    <span class="nt">&lt;/div&gt;</span>
{/if}
</code></pre>

</div>



<p>In the template, the if-else-if control-flow syntax examines both <code>profile</code> and <code>error</code>. The <code>if</code> branch displays the Github profile when it is defined. The <code>else-if</code> branch displays the error message when it is not blank.</p>

<h3>
  
  
  Angular 20 application
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// github-profile-card.component.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">httpResource</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/common/http</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">ChangeDetectionStrategy</span><span class="p">,</span> <span class="nx">Component</span><span class="p">,</span> <span class="nx">computed</span><span class="p">,</span> <span class="nx">input</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">GithubProfile</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">../types/github-profile.type</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Component</span><span class="p">({</span>
    <span class="na">selector</span><span class="p">:</span> <span class="dl">'</span><span class="s1">app-github-profile-card</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">templateUrl</span><span class="p">:</span> <span class="dl">'</span><span class="s1">./github-profile-card.component.html</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">changeDetection</span><span class="p">:</span> <span class="nx">ChangeDetectionStrategy</span><span class="p">.</span><span class="nx">OnPush</span><span class="p">,</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">GithubProfileCardComponent</span> <span class="p">{</span>
    <span class="nx">username</span> <span class="o">=</span> <span class="nx">input</span><span class="p">.</span><span class="nx">required</span><span class="o">&lt;</span><span class="kr">string</span><span class="o">&gt;</span><span class="p">();</span>

    <span class="nx">profileResource</span> <span class="o">=</span> <span class="nx">httpResource</span><span class="o">&lt;</span><span class="nx">GithubProfile</span><span class="o">&gt;</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nf">username</span><span class="p">()</span> <span class="p">?</span> <span class="p">{</span> 
        <span class="na">url</span><span class="p">:</span> <span class="s2">`https://api.github.com/users/</span><span class="p">${</span><span class="k">this</span><span class="p">.</span><span class="nf">username</span><span class="p">()}</span><span class="s2">`</span><span class="p">,</span>
        <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">GET</span><span class="dl">'</span><span class="p">,</span>
        <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
            <span class="na">Authorization</span><span class="p">:</span> <span class="s2">`Bearer </span><span class="p">${</span><span class="nx">GITHUB_TOKEN</span><span class="p">}</span><span class="s2">`</span>
        <span class="p">}</span>
    <span class="p">}:</span> <span class="kc">undefined</span><span class="p">,</span> <span class="p">{</span>
        <span class="na">equal</span><span class="p">:</span> <span class="p">(</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span><span class="p">?.</span><span class="nx">login</span> <span class="o">===</span> <span class="nx">b</span><span class="p">?.</span><span class="nx">login</span><span class="p">,</span>
    <span class="p">});</span>

    <span class="nx">profile</span> <span class="o">=</span> <span class="nf">computed</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">profileResource</span><span class="p">.</span><span class="nf">hasValue</span><span class="p">()</span> <span class="p">?</span> <span class="k">this</span><span class="p">.</span><span class="nx">profileResource</span><span class="p">.</span><span class="nf">value</span><span class="p">()</span> <span class="p">:</span> <span class="kc">undefined</span><span class="p">);</span>

    <span class="nx">error</span> <span class="o">=</span> <span class="nf">computed</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">profileResource</span><span class="p">.</span><span class="nf">error</span><span class="p">()?.</span><span class="nx">message</span> <span class="o">||</span> <span class="dl">''</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<p>The component has a required signal input, <code>username</code>.</p>

<p>When the <code>username</code> input changes, the <code>profileResource</code> reactively creates a <code>HttpResourceRequest</code><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w"> 
    </span><span class="err">url:</span><span class="w"> </span><span class="err">`https://api.github.com/users/$</span><span class="p">{</span><span class="err">this.username()</span><span class="p">}</span><span class="err">`</span><span class="p">,</span><span class="w">
    </span><span class="err">method:</span><span class="w"> </span><span class="err">'GET'</span><span class="p">,</span><span class="w">
    </span><span class="err">headers:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="err">Authorization:</span><span class="w"> </span><span class="err">`Bearer</span><span class="w"> </span><span class="err">$</span><span class="p">{</span><span class="err">GITHUB_TOKEN</span><span class="p">}</span><span class="err">`</span><span class="w">
    </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The request has the Github URL, HTTP Method and an HTTP headers.</p>

<p>When the <code>profileResource</code> has a new value, the <code>profile</code> and <code>error</code> computed signals derive the profile and the error message respectively.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// github-profile-card.component.html

@let status = profileResource.status();
@if (status === 'loading') {
    &lt;p&gt;Loading profile...&lt;/p&gt;
} @else if (status === 'error') {
    &lt;p&gt;Error loading profile: {{ error() }}&lt;/p&gt;
} @else {
    @if (profile(); as profile) {
        &lt;div&gt;
            &lt;p&gt;Username: {{ profile.login }}&lt;/p&gt;
            &lt;p&gt;Name: {{ profile.name }}&lt;/p&gt;
            &lt;p&gt;Bio: {{ profile.bio || 'N/A' }}&lt;/p&gt;
        &lt;/div&gt;
    }
}
</code></pre>

</div>



<p>The HTML template conditionally displays <code>Loading profile...</code> when the status is loading. When the status is <code>error</code>, the template displays the error message. Finally, the else branch displays the profile's login, name and bio.</p>

<h2>
  
  
  Create a GithubProfileList component
</h2>

<p>Next, we create a <code>GithubProfileList</code> component that passes  a username to <code>GithubProfileCard</code> to render. The <code>GithubProfileList</code> is reusable because it also accepts any usernames list.  </p>

<h3>
  
  
  Vue 3 application
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// GithubProfileList.vue</span>

<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">setup</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
    <span class="k">import</span> <span class="nx">GithubProfileCard</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./GithubProfileCard.vue</span><span class="dl">'</span>

    <span class="kd">const</span> <span class="p">{</span> <span class="nx">usernames</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">defineProps</span><span class="o">&lt;</span><span class="p">{</span> <span class="na">usernames</span><span class="p">:</span> <span class="kr">string</span><span class="p">[]</span> <span class="p">}</span><span class="o">&gt;</span><span class="p">()</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>



<p>Destructure <code>defineProps</code> to extract the <code>usernames</code> list.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;template&gt;</span>
  <span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"header"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h1&gt;</span>Github Profile List (Vue 3 Ver.)<span class="nt">&lt;/h1&gt;</span>
  <span class="nt">&lt;/div&gt;</span>
  <span class="nt">&lt;GithubProfileCard</span> <span class="na">v-for=</span><span class="s">"username in usernames"</span> <span class="na">:key=</span><span class="s">"username"</span> <span class="na">:username=</span><span class="s">"username"</span> <span class="nt">/&gt;</span>
<span class="nt">&lt;/template&gt;</span>
</code></pre>

</div>



<p>The <code>v-for</code> directive iterates the <code>username</code> prop and binds <code>username</code> to both <code>key</code> and <code>username</code> attributes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// App.vue</span>

<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">setup</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
<span class="k">import</span> <span class="nx">GithubProfileList</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./components/GithubProfileList.vue</span><span class="dl">'</span>

<span class="kd">const</span> <span class="nx">usernames</span> <span class="o">=</span> <span class="p">[</span><span class="dl">'</span><span class="s1">johnsoncodehk</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">antfu</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">railsstudent</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">danielkellyio</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">hootlex</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">MooseSaeed</span><span class="dl">'</span><span class="p">]</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span>
<span class="o">&lt;</span><span class="nx">template</span><span class="o">&gt;</span>
  <span class="o">&lt;</span><span class="nx">GithubProfileList</span> <span class="p">:</span><span class="nx">usernames</span><span class="o">=</span><span class="dl">"</span><span class="s2">usernames</span><span class="dl">"</span> <span class="o">/&gt;</span>
<span class="o">&lt;</span><span class="sr">/template</span><span class="err">&gt;
</span></code></pre>

</div>



<p>The <code>App</code> component passes the <code>usernames</code> list to the <code>GithubProfileList</code>.</p>

<h3>
  
  
  SvelteKit application
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// github-profile-list.svelte</span>

<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span> 
    <span class="k">import</span> <span class="nx">GithubProfileCard</span> <span class="k">from</span> <span class="dl">"</span><span class="s2">./github-profile-card.svelte</span><span class="dl">"</span><span class="p">;</span>
    <span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">GithubProfileItem</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./github-profile-item.type</span><span class="dl">'</span><span class="p">;</span>

    <span class="kd">type</span> <span class="nx">Props</span> <span class="o">=</span> <span class="p">{</span>
        <span class="na">profiles</span><span class="p">:</span> <span class="nx">GithubProfileItem</span><span class="p">[]</span>
    <span class="p">};</span>

    <span class="kd">const</span> <span class="p">{</span> <span class="nx">profiles</span> <span class="p">}:</span> <span class="nx">Props</span> <span class="o">=</span> <span class="nf">$props</span><span class="p">();</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>



<p>The <code>GithubProfileList</code> destructures props from <code>$props()</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight html"><code><span class="nt">&lt;div</span> <span class="na">class=</span><span class="s">"header"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;h1&gt;</span>Github Profile List (Svelte ver.)<span class="nt">&lt;/h1&gt;</span>
<span class="nt">&lt;/div&gt;</span>
{#each profiles as profile (profile.key)}
    <span class="nt">&lt;GithubProfileCard</span> <span class="err">{</span><span class="na">profile</span><span class="err">}</span> <span class="nt">/&gt;</span>
{/each}
</code></pre>

</div>



<p>In the template, <code>#each</code> iterates profiles and bind the <code>profile</code> to the props of <code>GuthubProfileCard</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// +page.svelte</span>

<span class="o">&lt;</span><span class="nx">script</span> <span class="nx">lang</span><span class="o">=</span><span class="dl">"</span><span class="s2">ts</span><span class="dl">"</span><span class="o">&gt;</span>
    <span class="k">import</span> <span class="kd">type</span> <span class="p">{</span> <span class="nx">PageProps</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./$types</span><span class="dl">'</span><span class="p">;</span>
    <span class="k">import</span> <span class="nx">GithubProfileList</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">$lib/github-profile-list.svelte</span><span class="dl">'</span><span class="p">;</span>

    <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">results</span> <span class="p">}:</span> <span class="nx">PageProps</span> <span class="o">=</span> <span class="nf">$props</span><span class="p">();</span>

    <span class="kd">const</span> <span class="p">{</span> <span class="na">data</span><span class="p">:</span> <span class="nx">profiles</span> <span class="p">}</span> <span class="o">=</span> <span class="nx">results</span><span class="p">;</span>
<span class="o">&lt;</span><span class="sr">/script</span><span class="err">&gt;
</span></code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight html"><code>{#if !profiles || profiles.length === 0}
    <span class="nt">&lt;p&gt;</span>No profiles found.<span class="nt">&lt;/p&gt;</span>
{:else}
    <span class="nt">&lt;GithubProfileList</span> <span class="err">{</span><span class="na">profiles</span><span class="err">}</span> <span class="nt">/&gt;</span>
{/if}
</code></pre>

</div>



<p>The loader function loads the <code>profiles</code> and it is passed to the <code>GithubProfileList</code> as props.  </p>

<h3>
  
  
  Angular 20 application
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="k">import</span> <span class="p">{</span> <span class="nx">ChangeDetectionStrategy</span><span class="p">,</span> <span class="nx">Component</span><span class="p">,</span> <span class="nx">input</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">GithubProfileCardComponent</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./github-profile-card.coponent</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Component</span><span class="p">({</span>
    <span class="na">selector</span><span class="p">:</span> <span class="dl">'</span><span class="s1">app-github-profile-list</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">GithubProfileCardComponent</span><span class="p">],</span>
    <span class="na">template</span><span class="p">:</span> <span class="s2">`
        &lt;div class="header"&gt;
            &lt;h1&gt;Github Profile List (Angular Ver.)&lt;/h1&gt;
        &lt;/div&gt;
        @for (username of usernames(); track username) {
            &lt;app-github-profile-card [username]="username"/&gt;
        }
    `</span><span class="p">,</span>
    <span class="na">changeDetection</span><span class="p">:</span> <span class="nx">ChangeDetectionStrategy</span><span class="p">.</span><span class="nx">OnPush</span><span class="p">,</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">GithubProfileListComponent</span> <span class="p">{</span>
    <span class="nx">usernames</span> <span class="o">=</span> <span class="nx">input</span><span class="p">.</span><span class="nx">required</span><span class="o">&lt;</span><span class="kr">string</span><span class="p">[]</span><span class="o">&gt;</span><span class="p">();</span>   
<span class="p">}</span>
</code></pre>

</div>



<p>The component has a required signal input, <code>usernames</code>.</p>

<p><code>@for</code> iterates the <code>usernames</code> input and passes the <code>username</code> to the <code>username</code> input of the <code>GithubProfileCardComponent</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight typescript"><code><span class="c1">// app.component.ts</span>

<span class="k">import</span> <span class="p">{</span> <span class="nx">ChangeDetectionStrategy</span><span class="p">,</span> <span class="nx">Component</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">@angular/core</span><span class="dl">'</span><span class="p">;</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">GithubProfileListComponent</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">./github/components/github-profile-list.component</span><span class="dl">'</span><span class="p">;</span>

<span class="p">@</span><span class="nd">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="dl">'</span><span class="s1">app-root</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">imports</span><span class="p">:</span> <span class="p">[</span><span class="nx">GithubProfileListComponent</span><span class="p">],</span>
  <span class="na">template</span><span class="p">:</span> <span class="dl">'</span><span class="s1">&lt;app-github-profile-list [usernames]="usernames" /&gt;</span><span class="dl">'</span><span class="p">,</span>
  <span class="na">changeDetection</span><span class="p">:</span> <span class="nx">ChangeDetectionStrategy</span><span class="p">.</span><span class="nx">OnPush</span><span class="p">,</span>
<span class="p">})</span>
<span class="k">export</span> <span class="kd">class</span> <span class="nc">AppComponent</span> <span class="p">{</span>
  <span class="k">readonly</span> <span class="nx">usernames</span> <span class="o">=</span> <span class="p">[</span><span class="dl">'</span><span class="s1">johnsoncodehk</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">antfu</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">railsstudent</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">danielkellyio</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">hootlex</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">MooseSaeed</span><span class="dl">'</span><span class="p">];</span>   
<span class="p">}</span>
</code></pre>

</div>



<p>The <code>AppComponent</code> defines a readonly <code>usernames</code> list and it is passed to the <code>GithubProfileListComponent</code>. </p>

<p>We have successfully created <code>GithubProfileList</code> and <code>GithubProfileCard</code> components and passed the names from the parent to the child. The child component receives inputs input and displays them in the template. </p>

<h2>
  
  
  Github Repositories
</h2>

<ul>
<li>Vue 3: <a href="https://github.com/railsstudent/vue-github-profile" rel="noopener noreferrer">https://github.com/railsstudent/vue-github-profile</a>
</li>
<li>Svelte 5: <a href="https://github.com/railsstudent/svelte-github-profile" rel="noopener noreferrer">https://github.com/railsstudent/svelte-github-profile</a>
</li>
<li>Angular 20: <a href="https://github.com/railsstudent/angular-github-profile" rel="noopener noreferrer">https://github.com/railsstudent/angular-github-profile</a>
</li>
</ul>

<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://angular.dev/tools/cli/build-system-migration#build-time-value-replacement-define" rel="noopener noreferrer">Angular Build Time Variables</a></li>
</ul>

