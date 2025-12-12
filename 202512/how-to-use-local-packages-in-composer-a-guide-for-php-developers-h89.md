---
Title: How to use local packages in Composer: a guide for PHP developers
Description: 
Author: Roberto B.
Date: 2025-12-12T21:07:26.000Z
Robots: noindex,nofollow
Template: index
---
<p>When working on a PHP project, it’s common to rely on external libraries published on Packagist. Composer makes installing and managing these dependencies effortless. But what if you need to work with a dependency locally, outside Packagist?</p>

<p>For example, maybe you’re building a project that depends on a library, and you want to test changes in that library immediately, without having to publish a new version or push to GitHub every time. Composer’s <code>repositories</code> option lets you do exactly that: override a dependency with a local directory on your machine.</p>

<p>In this article, we’ll walk through a practical scenario and explain why this workflow is useful, how to set it up, and which situations it solves.</p>

<h2>
  
  
  The Scenario
</h2>

<p>You’re developing a PHP project that depends on:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"require"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"storyblok/php-management-api-client"</span><span class="p">:</span><span class="w"> </span><span class="s2">"@dev"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"vlucas/phpdotenv"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^5.6"</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"repositories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"path"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"../php-management-api-client"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Here, the project requires the package <code>storyblok/php-management-api-client</code>. Normally, Composer would download it from Packagist. But using the <code>repositories</code> section with <code>"type": "path"</code> tells Composer:</p>

<blockquote>
<p>Instead of downloading this package from Packagist, use the copy located in <code>../php-management-api-client</code>.</p>
</blockquote>

<p>This means you can clone the library next to your project, modify it, and immediately see those changes reflected when you run your project.</p>

<h2>
  
  
  Why use a local package?
</h2>

<p>Here are some <em>very common</em> and <em>practical</em> scenarios where this workflow shines:</p>

<h3>
  
  
  1. Developing a feature or fix across two repositories
</h3>

<p>You’re building a new feature in your main project, but it requires changes in a dependency. Instead of waiting to merge and publish a new version of the dependency, you want to work on both side-by-side.<br>
With a local path repository, changes to the library update instantly.</p>
<h3>
  
  
  2. Debugging library behavior
</h3>

<p>Sometimes a dependency doesn’t behave as expected. You may need to add logging, inspect the internal state, or step through the library code to identify the issue. Working from a local copy makes deep debugging far easier.</p>
<h3>
  
  
  3. Contributing to Open Source packages
</h3>

<p>If you're preparing a PR for an Open Source library, you want to test your changes in a real project before submitting. Using a local path repository avoids temporary commits or using forks just for testing code.</p>
<h3>
  
  
  4. Working offline or in restricted environments
</h3>

<p>This sounds more like a workaround, but some enterprise environments block access to GitHub or Packagist. A local path package gives you full offline access during development.</p>
<h3>
  
  
  5. Handling unreleased versions or experimental APIs
</h3>

<p>If you're experimenting with an internal API or testing breaking changes, you may not want to publish anything yet. A local repository provides a secure workspace.</p>
<h3>
  
  
  6. Rapid iteration without version-bumping
</h3>

<p>When you’re iterating quickly, continuously bumping semantic versions or updating branches can be a hassle. A local path repository lets you skip version management until you're ready to publish.</p>
<h2>
  
  
  How to use a local package with Composer
</h2>

<p>Here’s what you need:</p>
<h3>
  
  
  1. Clone the dependency locally
</h3>

<p>Place it somewhere near your project, for example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>~/Projects/my-project  
~/Projects/php-management-api-client  &lt;-- cloned library
</code></pre>

</div>



<blockquote>
<p>Note that the cloned library folder name doesn’t have to match the package name as long as the path matches the URL definition (see next point).</p>
</blockquote>

<h3>
  
  
  2. Update <code>composer.json</code>
</h3>

<p>Add a path repository pointing to the folder:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="nl">"repositories"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
  </span><span class="p">{</span><span class="w">
    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"path"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"url"</span><span class="p">:</span><span class="w"> </span><span class="s2">"../php-management-api-client"</span><span class="w">
    </span><span class="nl">"options"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"symlink"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">]</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  3. Require the package at a development version
</h3>

<p>When working with a local package, Composer needs to know that it should use a <strong>development</strong> version rather than a stable release from Packagist. You can specify this in different ways, but, in my opinion, the most reliable option is to require a specific dev branch, usually <code>dev-main</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>"require": {
  "storyblok/php-management-api-client": "dev-main"
}
</code></pre>

</div>



<h4>
  
  
  Understanding the options
</h4>

<p>Composer supports several types of version constraints. Here are the ones most relevant when using a local path repository:</p>

<ul>
<li>
<code>dev-main</code> : uses the development version from the <code>main</code> branch.
This is the most predictable and recommended option for local development if you are using the <code>main</code> branch for the library.</li>
<li>
<code>@dev</code> : allows Composer to install <strong>any</strong> development version (for example: <code>dev-main</code>, <code>dev-master</code>, or other dev branches).
This is more flexible than using <code>dev-main</code>, but less explicit.</li>
<li>
<code>"*"</code>: accepts <strong>any version</strong> (stable or dev).
I used it in the past, but I would not recommend it, as Composer may choose an unexpected version.</li>
</ul>

<p>For clarity and consistency, especially when working with a cloned library, using <code>dev-main</code> ensures Composer consistently links to the exact development branch (<code>main</code> in this case) you're working on.</p>

<h3>
  
  
  4. Install or update
</h3>

<p>Run:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>composer update storyblok/php-management-api-client
</code></pre>

</div>



<h2>
  
  
  Best practices
</h2>

<ul>
<li>Keep the local clone clean; avoid committing temporary debug code.</li>
<li>Switch back to the Packagist version after finishing development.</li>
<li>Avoid committing the local path configuration to the repository unless intentional.</li>
</ul>

