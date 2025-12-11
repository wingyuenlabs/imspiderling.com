---
Title: Untangling the Rails Monolith - quick look at the code
Description: 
Author: rbglod
Date: 2025-12-11T22:05:47.000Z
Robots: noindex,nofollow
Template: index
---
<p>Last time I wrote about the data separation at the database level. As we already know <strong>each component should work as an (almost) independent service</strong>. How can that be achevied at the codebase level? What to do when for example in Accounting we need address data for an Employee? How can we get the Organization subscription plan if we're in the Checkout component?</p>

<p>Each of those components should work as a separate app ideally. If we imagine them as a standalone apps, it might be easier to design future code and not couple services with each other.</p>

<p>As in the previous blog post, let's focus on some kind-of real life example.</p>

<h3>
  
  
  Marketplace
</h3>

<p>Let's say we have a simple app in which a User can register an account, select given subscription plan, put ads with their Goods and also buy Goods from other users. Depending on the subscription type, we offer them better shipping options, discounts, etc.</p>

<p>We can describe few components in this scenario.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>app/
|- components/
  |- users/
  |- marketplace/
  |- payments/
  |- subscriptions/
</code></pre>

</div>



<p>Each of the components has their own <code>controllers/</code>, <code>models/</code>, <code>services/</code>, etc. In more complex apps we might see <code>infrastructure/</code>, <code>domain/</code>, <code>repositories/</code> and others, but let's stick to simple example.</p>

<p>In ideal world, each of the component lives on it's own - Users know only about the <code>Users</code> model (and other related to Users), they check if User is logged in, update their passwords, account data, etc. </p>

<p>But at some point user hits a <code>Marketplace</code> and want to order something. In <code>Marketplace</code> component we want to know where the order should go, who is the recipient and what possible discounts we should offer.</p>

<p>Instead of doing something like this<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># app/components/marketplace/controllers/orders_controller.rb</span>

<span class="k">def</span> <span class="nf">create</span>
  <span class="vi">@order</span> <span class="o">=</span> <span class="no">Marketplace</span><span class="o">::</span><span class="no">Models</span><span class="o">::</span><span class="no">Order</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span>
    <span class="ss">user_id: </span><span class="n">current_user</span><span class="p">.</span><span class="nf">id</span><span class="p">,</span>
    <span class="ss">shipping_address: </span><span class="n">current_user</span><span class="p">.</span><span class="nf">shipping_address</span><span class="p">,</span>
    <span class="ss">offering: </span><span class="n">current_user</span><span class="p">.</span><span class="nf">subscription_plan</span><span class="p">,</span>
    <span class="ss">discounts: </span><span class="n">current_user</span><span class="p">.</span><span class="nf">discounts</span><span class="p">,</span>
    <span class="ss">items: </span><span class="n">order_params</span><span class="p">[</span><span class="ss">:items_ids</span><span class="p">]</span>
  <span class="p">)</span>
  <span class="o">...</span>
<span class="k">end</span>
</code></pre>

</div>



<p>We don't want to couple <code>Marketplace</code> with <code>Users</code>. We definitely shouldn't use <code>user</code> relations here, as they might change in future and this code would break.<br>
The default scope of <code>user.discounts</code> might change and it'd be hard to debug, why suddenly the behavior of our endpoint changed.</p>

<p>Instead we should use Facades or APIs or any other approach that will let us put the query to another component in one place and make sure we always get the same result.</p>

<p>What I mean through that is the <code>users/</code> domain could expose their public API for other domains to fetch users' data.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># app/components/users/public/api.rb</span>

<span class="k">class</span> <span class="o">&lt;&lt;</span> <span class="nb">self</span>
  <span class="no">UserResponse</span> <span class="o">=</span> <span class="no">Data</span><span class="p">.</span><span class="nf">define</span><span class="p">(</span><span class="ss">:shipping_address</span><span class="p">,</span> <span class="ss">:subscription_plan</span><span class="p">,</span> <span class="ss">:discounts</span><span class="p">)</span>

  <span class="k">def</span> <span class="nf">fetch_user_data</span><span class="p">(</span><span class="n">user_id</span><span class="p">:)</span>
    <span class="n">user</span> <span class="o">=</span> <span class="no">Users</span><span class="o">::</span><span class="no">Models</span><span class="o">::</span><span class="no">User</span><span class="p">.</span><span class="nf">find_by</span><span class="p">(</span><span class="n">user_id</span><span class="p">:)</span>
    <span class="no">UserResponse</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span>
      <span class="ss">shipping_address: </span><span class="n">user</span><span class="p">.</span><span class="nf">shipping_address</span><span class="p">,</span>
      <span class="ss">subscription_plan: </span><span class="n">user</span><span class="p">.</span><span class="nf">subscription_plan</span><span class="p">,</span>
      <span class="ss">discounts: </span><span class="n">user</span><span class="p">.</span><span class="nf">discounts</span><span class="p">.</span><span class="nf">active</span><span class="p">.</span><span class="nf">sum</span><span class="p">(</span><span class="ss">:discount_amount</span><span class="p">)</span>
    <span class="p">)</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># app/components/marketplace/controllers/orders_controller.rb</span>

<span class="k">def</span> <span class="nf">create</span>
  <span class="n">user_data</span> <span class="o">=</span> <span class="o">::</span><span class="no">Users</span><span class="o">::</span><span class="no">Public</span><span class="o">::</span><span class="no">Api</span><span class="p">.</span><span class="nf">fetch_user_data</span><span class="p">(</span><span class="ss">user_id: </span><span class="n">current_user</span><span class="p">.</span><span class="nf">id</span><span class="p">)</span>
  <span class="n">order</span> <span class="o">=</span> <span class="no">Marketplace</span><span class="o">::</span><span class="no">Models</span><span class="o">::</span><span class="no">Order</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span>
    <span class="ss">user_id: </span><span class="n">current_user</span><span class="p">.</span><span class="nf">id</span><span class="p">,</span>
    <span class="ss">shipping_address: </span><span class="n">user_data</span><span class="p">.</span><span class="nf">shipping_address</span><span class="p">,</span>
    <span class="ss">offering: </span><span class="n">user_data</span><span class="p">.</span><span class="nf">subscription_plan</span><span class="p">,</span>
    <span class="ss">discounts: </span><span class="n">user_data</span><span class="p">.</span><span class="nf">discounts</span><span class="p">,</span>
    <span class="ss">items: </span><span class="n">order_params</span><span class="p">[</span><span class="ss">:items_ids</span><span class="p">]</span>
  <span class="p">)</span>
<span class="k">end</span>
</code></pre>

</div>



<p>What this changes is that this API is the only source of truth. If every component uses <code>Users::Public::Api</code> to fetch User data, we don't need to worry about checking all references to <code>users.shipping_address</code> when changing something around users' address. The only thing we need to ensure is working as expected is the API we wrote - if it's still returning same values as before our refactor of <code>shipping_address</code>, then we're golden.</p>

<p>All <code>user.shipping_address</code> occurrences should happen within <code>users/</code> component, all other components should access it through <code>Users::Public::Api</code>.</p>

<p>The <code>users/</code> component is the owner of the code, and they know that they can't change it, break it, or do whatever that affects other domains that consume that data.</p>

<p>If they want to implement a change in a way how discounts are returned, then a new version of API might be implemented while the old one remains the same. Or, if everyone  (all components that use that data) agree and synchronize, then the change is possible in existing API.</p>

<h3>
  
  
  Why is it that important
</h3>

<p>This prevents from calling directly ActiveRecord models from different domains. Direct calls create strong coupling when it's not needed. We just need data from three columns, we don't need whole <code>User</code> record. What if those columns got migrated to different model? We'll try to read <code>user.shipping_address</code>, but it's no longer there and we need to debug. If we're using given component's API, if they migrate data to another table, they also make sure that the API returns the same values but from another data source.</p>

<p>For us (consumers) it doesn't matter from where the data comes from. We just want the address and the <code>users/</code> should give it to us if we're calling their API. If we call their models directly, we need to know where exactly the data is - and that's not needed if we're in <code>marketplace/</code>.</p>

