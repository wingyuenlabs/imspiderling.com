---
Title: Automating Tests with Playwright and PageObject: A Practical Approach
Description: 
Author: Rodrigo Cabral
Date: 2025-08-14T21:10:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Embark on a transformative journey into the realm of test automation with our latest article: "Automating Tests with Playwright and PageObject: A Practical Approach." Discover the synergy between Playwright and PageObject, unraveling a powerhouse combination that not only streamlines operational processes but also empowers the creation of resilient and easily maintainable tests.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdjtiaolzxrw1tq4nqu6g.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdjtiaolzxrw1tq4nqu6g.gif" alt="intro" width="500" height="292"></a></p>




<h2>
  
  
  Introduction to Playwright and PageObject
</h2>

<p>Automating web tests can be a challenging task, especially when dealing with complex and ever-evolving applications. That's where "pageobject" comes into play an approach aimed at simplifying the structuring and organization of tests. By adopting "pageobject," we can create a clear and reusable framework where each web page and its elements are represented as objects, making test writing and maintenance a breeze.<br>
In this article, we will delve into how "pageobject," combined with the powerful Playwright library, can revolutionize the way we automate tests, making them more practical and efficient. Get ready to explore an approach that can accelerate your workflow and deliver consistent results in your web test automations.<br>
I'd like to remind you that this article is a continuation of <a href="https://dev.to/rodrigoobc/getting-started-with-playwright-introduction-to-web-testing-automation-1fdh">Getting Started with Playwright — Introduction to Web Testing Automation</a>, and it would be beneficial to have a basic understanding of Playwright for a more in-depth grasp of what will be explained here. Therefore, I recommend reading the first article.</p>
<h3>
  
  
  PageObject?
</h3>

<p>The concept of "PageObject" is an approach aimed at organizing and structuring automated tests in a more intuitive and reusable manner. Essentially, each web page is mapped as an object, containing the elements and actions that can be performed on that page. This mapping allows tests to interact with page elements in a simple and consistent way, making test writing and maintenance easier.</p>

<p>In practical terms, imagine you're automating a login process on a website. With "PageObject," instead of repeating code for locating and interacting with login elements in each test, you can create a specific object for the login page. This object will have methods to fill in the username field, the password field, and click the login button. This way, in every test involving login, you simply invoke these methods from the page object, making the code cleaner, more readable, and easy to maintain. If there's any change to the login page, you only need to update the page object, not all tests using that page. This enhances the efficiency and scalability of your automated tests.</p>


<h2>
  
  
  Structuring the Project in the "PageObject"
</h2>

<p>Alright, let's kick off, setting up the "PageObject" structure.</p>

<p>The most common idea is to split this into two folders: one for Page files (stuff related to the page itself) and another for tests. This keeps things tidy, separating the code for interacting with the app from the actual tests. So, let's get our hands dirty and build this structure!</p>

<p>So, we'll have the following folder architecture:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa4o6fwm2ynrmknl4g9h8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa4o6fwm2ynrmknl4g9h8.png" alt="Simple Structure of PageObject" width="350" height="286"></a></p>

<ul>
<li>
<strong>pages</strong>: Folder, where you'll find files with classes that map the actions of each "Page" and its "Objects."</li>
<li>
<strong>tests</strong>: Folder, where you'll find the "specs" files for our project.</li>
<li>
<strong>utils</strong>: Folder, where you'll find functions and code snippets that we can reuse in various scenarios.</li>
</ul>

<p>Now let's get practical:</p>

<p>Here we can see the first "Page" created; it will be responsible for all the actions that the Amazon home page can perform.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="p">{</span> <span class="nx">expect</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">);</span>

<span class="kd">class</span> <span class="nc">AmazonHomePage</span> <span class="p">{</span>
    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span> <span class="o">=</span> <span class="p">{</span>
            <span class="dl">"</span><span class="s2">url</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">https://www.amazon.com.br/</span><span class="dl">"</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">SearchField</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="twotabsearchtextbox"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">SearchButton</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="nav-search-submit-button"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">ProductTarget</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Batman - A Corte das Corujas</span><span class="dl">"</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">titlePage</span><span class="dl">"</span><span class="p">:</span> <span class="dl">"</span><span class="s2">Amazon.com.br | Tudo pra você, de A a Z.</span><span class="dl">"</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">boxResults</span><span class="dl">"</span><span class="p">:</span><span class="s2">`[data-component-type="s-search-result"]`</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">TitleResult</span><span class="dl">"</span><span class="p">:</span><span class="dl">'</span><span class="s1">[class="a-size-base-plus a-color-base a-text-normal"]</span><span class="dl">'</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">navegatTo</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">goto</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">url</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">getSearchField</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">SearchField</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">SearchField</span><span class="p">)</span>
        <span class="k">await</span> <span class="nx">SearchField</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">SearchField</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">getSearchButton</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">SearchButton</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">SearchButton</span><span class="p">)</span>
        <span class="k">await</span> <span class="nx">SearchButton</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">SearchButton</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">clickSearchButton</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">SearchButton</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getSearchButton</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">SearchButton</span><span class="p">.</span><span class="nf">click</span><span class="p">()</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">searchProduct</span><span class="p">(</span><span class="nx">productName</span><span class="p">)</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">SearchField</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getSearchField</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">SearchField</span><span class="p">.</span><span class="nf">click</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">SearchField</span><span class="p">.</span><span class="nf">fill</span><span class="p">(</span><span class="nx">productName</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">getBoxsResult</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">boxResult</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">boxResults</span><span class="p">).</span><span class="nf">first</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">boxResult</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">boxResult</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">validateHomePage</span><span class="p">(){</span>
        <span class="kd">const</span> <span class="nx">currentUrl</span> <span class="o">=</span> <span class="nx">page</span><span class="p">.</span><span class="nf">url</span><span class="p">();</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">currentUrl</span><span class="p">).</span><span class="nf">toBe</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://www.amazon.com.br/</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">validateSearchProduct</span><span class="p">(</span><span class="nx">productName</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">searchProduct</span><span class="p">(</span><span class="nx">productName</span><span class="p">)</span>
        <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">clickSearchButton</span><span class="p">()</span>
        <span class="kd">let</span> <span class="nx">boxResult</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getBoxsResult</span><span class="p">()</span>

        <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">boxResult</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">TitleResult</span><span class="p">)).</span><span class="nf">toContainText</span><span class="p">(</span><span class="nx">productName</span><span class="p">)</span>

    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">clickBoxResultAfterSearch</span><span class="p">(</span><span class="nx">productName</span><span class="p">){</span>
        <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">validateSearchProduct</span><span class="p">(</span><span class="nx">productName</span><span class="p">)</span>
        <span class="kd">let</span> <span class="nx">boxResult</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getBoxsResult</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">boxResult</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="dl">'</span><span class="s1">[href*="/Batman-Corte-Corujas-Christa-Faust/"]</span><span class="dl">'</span><span class="p">).</span><span class="nf">first</span><span class="p">().</span><span class="nf">click</span><span class="p">()</span>

    <span class="p">}</span>
<span class="p">}</span>

<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">{</span> <span class="na">AmazonHomePage</span><span class="p">:</span> <span class="nx">AmazonHomePage</span> <span class="p">}</span>


</code></pre>

</div>



<p>We can observe in the code that the Amazon home page has been transformed into a class, while its actions have become functions of that class, and its elements have also become objects. This is the necessary abstraction to create a "Page Object."</p>

<p>We can notice that some functions are generic, making them reusable for other tests. We also see that we'll use the same search function for any desired product without additional code. Even for a test covering a different scenario, these functions can be leveraged.</p>

<p>This level of reusability wouldn't have been possible without using the "Page Object," as we can see in my previous article.</p>

<p><a href="https://dev.to/rodrigoobc/getting-started-with-playwright-introduction-to-web-testing-automation-1fdh">Getting Started with Playwright — Introduction to Web Testing Automation</a></p>

<p>Now, we will create the "page" for the product:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">expect</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">);</span>


<span class="kd">class</span> <span class="nc">ProductPage</span><span class="p">{</span>
    <span class="nf">constructor</span><span class="p">(){</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span> <span class="o">=</span> <span class="p">{</span>
            <span class="dl">"</span><span class="s2">ProductTitle</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="productTitle"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">ProductDesc</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="bookDescription_feature_div"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">ProductValue</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="price"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">addCartButton</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="addToCart_feature_div"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">BuyNow</span><span class="dl">"</span><span class="p">:</span><span class="dl">'</span><span class="s1">[id="buyNow"]</span><span class="dl">'</span><span class="p">,</span>

        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">getTitleProduct</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">productName</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTitle</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">productName</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">productName</span>
    <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getDescProduct</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">ProductDesc</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductDesc</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">ProductDesc</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">ProductDesc</span>

    <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getValueProduct</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">ProductValue</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductValue</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">ProductValue</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">ProductValue</span>

    <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getAddCart</span><span class="p">(){</span>

        <span class="kd">let</span> <span class="nx">addCartButton</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">addCartButton</span><span class="p">).</span><span class="nf">first</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">addCartButton</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">addCartButton</span>

    <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getBuyNow</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">BuyNow</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">BuyNow</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">BuyNow</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">BuyNow</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">clickAddCart</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">addCart</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getAddCart</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">addCart</span><span class="p">.</span><span class="nf">click</span><span class="p">()</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">validateProductPage</span><span class="p">(</span><span class="nx">ProducName</span><span class="p">,</span><span class="nx">ProductDesc</span><span class="p">){</span>
        <span class="kd">let</span> <span class="nx">elementName</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getTitleProduct</span><span class="p">()</span>
        <span class="kd">let</span> <span class="nx">elementDesc</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getDescProduct</span><span class="p">()</span>

        <span class="kd">let</span> <span class="nx">addCartButton</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getAddCart</span><span class="p">()</span>
        <span class="kd">let</span> <span class="nx">BuyNow</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getBuyNow</span><span class="p">()</span>

        <span class="nf">expect</span><span class="p">(</span><span class="nx">elementName</span><span class="p">).</span><span class="nf">toHaveText</span><span class="p">(</span><span class="nx">ProducName</span><span class="p">)</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">elementDesc</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="nx">ProductDesc</span><span class="p">)</span>

        <span class="nf">expect</span><span class="p">(</span><span class="nx">addCartButton</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="dl">"</span><span class="s2">Adicionar ao carrinho</span><span class="dl">"</span><span class="p">)</span>
        <span class="nf">expect</span><span class="p">(</span><span class="nx">BuyNow</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="dl">"</span><span class="s2"> Comprar agora </span><span class="dl">"</span><span class="p">)</span>
    <span class="p">}</span>
<span class="p">}</span>



<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">{</span> <span class="na">ProductPage</span><span class="p">:</span> <span class="nx">ProductPage</span> <span class="p">}</span>

</code></pre>

</div>



<p>And "page" for the cart<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">expect</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">);</span>


<span class="kd">class</span> <span class="nc">CartPage</span> <span class="p">{</span>
    <span class="nf">constructor</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span> <span class="o">=</span> <span class="p">{</span>
            <span class="dl">"</span><span class="s2">CarTitle</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="NATC_SMART_WAGON_CONF_MSG_SUCCESS"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">closeBuy</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[id="sc-buy-box-ptc-button"] [class="sc-with-multicart"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">goToCart</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[data-csa-c-content-id="sw-gtc_CONTENT"]</span><span class="dl">'</span><span class="p">,</span>
            <span class="dl">"</span><span class="s2">ProductTitle</span><span class="dl">"</span><span class="p">:</span> <span class="dl">'</span><span class="s1">[data-a-max-rows="2"] [class="a-truncate-full a-offscreen"]</span><span class="dl">'</span><span class="p">,</span>

        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">getcartTitle</span><span class="p">()</span> <span class="p">{</span>
        <span class="kd">let</span> <span class="nx">CarTitle</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">CarTitle</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">CarTitle</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">CarTitle</span>
     <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getcloseBuy</span><span class="p">()</span> <span class="p">{</span> 
        <span class="kd">let</span> <span class="nx">closeBuy</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">closeBuy</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">closeBuy</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">closeBuy</span>

    <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getgoToCart</span><span class="p">()</span> <span class="p">{</span> 

        <span class="kd">let</span> <span class="nx">goToCart</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">goToCart</span><span class="p">)</span> 
        <span class="k">await</span> <span class="nx">goToCart</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">goToCart</span>
    <span class="p">}</span>
    <span class="k">async</span> <span class="nf">getProductTitle</span><span class="p">()</span> <span class="p">{</span> 

        <span class="kd">let</span> <span class="nx">ProductTitle</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">page</span><span class="p">.</span><span class="nf">locator</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTitle</span><span class="p">).</span><span class="nf">first</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">ProductTitle</span><span class="p">.</span><span class="nf">waitFor</span><span class="p">(</span><span class="dl">'</span><span class="s1">visible</span><span class="dl">'</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">ProductTitle</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">clickGotoCart</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">goToCart</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getgoToCart</span><span class="p">()</span>
        <span class="k">await</span> <span class="nx">goToCart</span><span class="p">.</span><span class="nf">click</span><span class="p">()</span>
    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">validateFirstCart</span><span class="p">(){</span>
        <span class="kd">let</span> <span class="nx">cartTitle</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getcartTitle</span><span class="p">()</span>
        <span class="kd">let</span> <span class="nx">closeBuy</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getcloseBuy</span><span class="p">()</span>
        <span class="kd">let</span> <span class="nx">goToCart</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getgoToCart</span><span class="p">()</span>

        <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">cartTitle</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="dl">"</span><span class="s2">Adicionado ao carrinho</span><span class="dl">"</span><span class="p">)</span>
        <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">closeBuy</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="dl">"</span><span class="s2">Finalizar carrinho da Amazon</span><span class="dl">"</span><span class="p">)</span>
        <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">goToCart</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="dl">"</span><span class="s2">Ir para o carrinho</span><span class="dl">"</span><span class="p">)</span>

    <span class="p">}</span>

    <span class="k">async</span> <span class="nf">validateSecondCart</span><span class="p">(</span><span class="nx">ProducName</span><span class="p">){</span>
        <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">clickGotoCart</span><span class="p">()</span>
        <span class="kd">let</span> <span class="nx">producTitle</span> <span class="o">=</span> <span class="k">await</span> <span class="k">this</span><span class="p">.</span><span class="nf">getProductTitle</span><span class="p">()</span>

        <span class="k">await</span> <span class="nf">expect</span><span class="p">(</span><span class="nx">producTitle</span><span class="p">).</span><span class="nf">toContainText</span><span class="p">(</span><span class="nx">ProducName</span><span class="p">)</span>
    <span class="p">}</span>


<span class="p">}</span>



<span class="nx">module</span><span class="p">.</span><span class="nx">exports</span> <span class="o">=</span> <span class="p">{</span> <span class="na">CartPage</span><span class="p">:</span> <span class="nx">CartPage</span> <span class="p">}</span>

</code></pre>

</div>



<p>Now, let's see how code reuse has benefited us. We can observe that each "Page" has its specific actions, allowing us to reuse them in all tests we perform from now on.</p>

<p>The part of accessing the website, performing the search, and validating the search is handled in the home page class. On the other hand, functions to validate the product and add it to the cart are executed in the product page class. Finally, the cart validation functions are concentrated on this last page.</p>

<p>Now, let's check how the test file turned out:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code>
<span class="c1">// @ts-check</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">test</span><span class="p">,</span> <span class="nx">expect</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">'</span><span class="s1">@playwright/test</span><span class="dl">'</span><span class="p">);</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">AmazonHomePage</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../pages/homePageAmazon</span><span class="dl">"</span><span class="p">)</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">ProductPage</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../pages/ProductPageAmazon</span><span class="dl">"</span><span class="p">)</span>
<span class="kd">const</span> <span class="p">{</span> <span class="nx">CartPage</span> <span class="p">}</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">../pages/CartPageAmazon</span><span class="dl">"</span><span class="p">)</span>

<span class="kd">const</span> <span class="nx">HomePage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">AmazonHomePage</span><span class="p">()</span>
<span class="kd">const</span> <span class="nx">productPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">ProductPage</span><span class="p">()</span>
<span class="kd">const</span> <span class="nx">cartPage</span> <span class="o">=</span> <span class="k">new</span> <span class="nc">CartPage</span><span class="p">()</span>

<span class="nf">test</span><span class="p">(</span><span class="s2">`Successfully Search for </span><span class="p">${</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">}</span><span class="s2"> on Amazon`</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nb">global</span><span class="p">.</span><span class="nx">page</span> <span class="o">=</span> <span class="nx">page</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">Navigate to the main screen of amazon.com</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">navegatTo</span><span class="p">();</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">Main screen of Amazon is displayed</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">validateHomePage</span><span class="p">()</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Search for the product  </span><span class="p">${</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">validateSearchProduct</span><span class="p">(</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">)</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Validate the found product`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">validateSearchProduct</span><span class="p">(</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">)</span>
  <span class="p">})</span>

<span class="p">});</span>

<span class="nf">test</span><span class="p">(</span><span class="dl">'</span><span class="s1">Successfully Add Product to Cart</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">({</span> <span class="nx">page</span> <span class="p">})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="nb">global</span><span class="p">.</span><span class="nx">page</span> <span class="o">=</span> <span class="nx">page</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">Navigate to the main screen of amazon.com</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">navegatTo</span><span class="p">();</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="dl">'</span><span class="s1">Main screen of Amazon is displayed</span><span class="dl">'</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">validateHomePage</span><span class="p">()</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Access the  </span><span class="p">${</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">}</span><span class="s2">  product page.`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">HomePage</span><span class="p">.</span><span class="nf">clickBoxResultAfterSearch</span><span class="p">(</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">)</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Validate the product page.`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">descProduct</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">Uma sociedade secreta, traiçoeira e liderada por famílias ricas e influentes assombra os cidadãos de Gotham City e coloca em risco a segurança da cidade e do próprio Batman. A Corte das Corujas, há muito escondida sob a sombras do submundo, vai mostrar suas garras novamente, e desta vez, da maneira mais impiedosa possível. Um romance original de Greg Cox, autor best-seller de diversas novelizações de filmes, entre as quais Batman – O Cavaleiro das Trevas Ressurge.</span><span class="dl">"</span> 
    <span class="k">await</span> <span class="nx">productPage</span><span class="p">.</span><span class="nf">validateProductPage</span><span class="p">(</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">,</span><span class="nx">descProduct</span> <span class="p">)</span>
    <span class="k">await</span> <span class="nx">productPage</span><span class="p">.</span><span class="nf">clickAddCart</span><span class="p">()</span>
  <span class="p">})</span>

  <span class="k">await</span> <span class="nx">test</span><span class="p">.</span><span class="nf">step</span><span class="p">(</span><span class="s2">`Validate the product in the cart.`</span><span class="p">,</span> <span class="k">async </span><span class="p">()</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="k">await</span> <span class="nx">cartPage</span><span class="p">.</span><span class="nf">validateFirstCart</span><span class="p">()</span>
    <span class="k">await</span> <span class="nx">cartPage</span><span class="p">.</span><span class="nf">validateSecondCart</span><span class="p">(</span><span class="nx">HomePage</span><span class="p">.</span><span class="nx">ObjectsPage</span><span class="p">.</span><span class="nx">ProductTarget</span><span class="p">)</span>
  <span class="p">})</span>

<span class="p">});</span>

</code></pre>

</div>



<p>The crucial aspect for using PageObject with Playwright is to define the Playwright “Page” class as a global variable, thereby enabling its use throughout your code.</p>

<p>We can observe in the test file that we have reused functions from the pages in the tests. Both test cases use various functions, leading to a reduction in the amount of code lines, improving future maintenance and reducing potential errors arising from redundancy.</p>




<h2>
  
  
  Conclusion
</h2>

<p>We conclude our exploration of the integration of PageObject with Playwright, a partnership that has proven to be crucial in the field of test automation. Throughout this article, various ways have been seen in which this combination not only optimizes operational processes but also strengthens the development of robust and easily maintainable tests. May this knowledge contribute to continuous improvements in test quality and simplify the complexity of software development.</p>




<h2>
  
  
  Sources and Useful Links
</h2>

<p>(Fast and reliable end-to-end testing for modern web apps | Playwright)<a href="//Download%20|%20Node.js%20(nodejs.org)">https://playwright.dev/</a>[<a href="https://nodejs.org/en/download" rel="noopener noreferrer">https://nodejs.org/en/download</a>]<br>
LinkedIn: (Rodrigo Cabral | LinkedIn)[<a href="https://www.linkedin.com/in/rodrigo-cabral-0280b3121/" rel="noopener noreferrer">https://www.linkedin.com/in/rodrigo-cabral-0280b3121/</a>]<br>
GitHub: ( RodrigoOBC· GitHub)[<a href="https://github.com/RodrigoOBC/" rel="noopener noreferrer">https://github.com/RodrigoOBC/</a>]</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4o0ctbvvnczattwqdg2.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4o0ctbvvnczattwqdg2.gif" alt="conclusion" width="600" height="249"></a></p>

