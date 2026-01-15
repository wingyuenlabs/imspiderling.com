---
Title: Using Ruby Model Classes, Service Objects and Interactors
Description: 
Author: Zil Norvilis
Date: 2026-01-15T21:47:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Using a <strong>model class</strong> (even a simple PORO in <code>app/models</code>) for features has several advantages in Rails.<br>
This is why developers often prefer “a model class” over scattering logic in controllers, helpers, or initializers.</p>

<p>Below are the <strong>key advantages</strong>:</p>


<h2>
  
  
  1. <strong>Centralized, reusable logic</strong>
</h2>

<p>If a feature’s logic is placed in a model class (e.g., <code>FeatureFlag</code>, <code>BetaAccess</code>, <code>Onboarding</code>), you can reuse it from:</p>

<ul>
<li>controllers</li>
<li>views</li>
<li>background jobs</li>
<li>services</li>
<li>Pundit policies</li>
</ul>

<p>Instead of duplicating the logic in multiple places.</p>


<h2>
  
  
  2. <strong>Keeps controllers and views clean</strong>
</h2>

<p>Rails controllers and views should stay thin.<br>
Putting domain logic in a model keeps your design clean (Fat Model, Skinny Controller).</p>

<p>Example:<br>
Instead of:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">if</span> <span class="n">user</span><span class="p">.</span><span class="nf">admin?</span> <span class="o">&amp;&amp;</span> <span class="no">SomeConfig</span><span class="p">.</span><span class="nf">beta_enabled?</span>
</code></pre>

</div>



<p>You do:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">if</span> <span class="no">BetaAccess</span><span class="p">.</span><span class="nf">allowed_for?</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
</code></pre>

</div>






<h2>
  
  
  3. <strong>Better testability</strong>
</h2>

<p>Models are the easiest to test:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">RSpec</span><span class="p">.</span><span class="nf">describe</span> <span class="no">BetaAccess</span> <span class="k">do</span>
  <span class="n">describe</span> <span class="s2">".allowed_for?"</span> <span class="k">do</span>
    <span class="c1"># ...</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>No need to spin up controllers or simulate web requests.</p>




<h2>
  
  
  4. <strong>Encapsulation of rules</strong>
</h2>

<p>If your feature has logic that may grow, models keep it in one place.</p>

<p>Example: onboarding flow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">Onboarding</span>
  <span class="k">def</span> <span class="nf">completed?</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
    <span class="n">user</span><span class="p">.</span><span class="nf">profile_filled?</span> <span class="o">&amp;&amp;</span> <span class="n">user</span><span class="p">.</span><span class="nf">verified?</span> <span class="o">&amp;&amp;</span> <span class="n">user</span><span class="p">.</span><span class="nf">tutorial_done?</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>If later you add new onboarding rules — you update one class.</p>




<h2>
  
  
  5. <strong>Better naming + improved readability</strong>
</h2>

<p>A dedicated model communicates intent clearly:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">if</span> <span class="no">FeatureFlag</span><span class="p">.</span><span class="nf">enabled?</span><span class="p">(</span><span class="ss">:new_ui</span><span class="p">)</span>
</code></pre>

</div>



<p>is more readable than:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">if</span> <span class="no">Rails</span><span class="p">.</span><span class="nf">configuration</span><span class="p">.</span><span class="nf">x</span><span class="p">.</span><span class="nf">new_ui_enabled</span>
</code></pre>

</div>



<p>or random constants.</p>




<h2>
  
  
  6. <strong>Supports persistence easily later</strong>
</h2>

<p>You might start with:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">FeatureFlag</span>
  <span class="no">FLAGS</span> <span class="o">=</span> <span class="p">{</span> <span class="ss">new_ui: </span><span class="kp">false</span> <span class="p">}</span>
<span class="k">end</span>
</code></pre>

</div>



<p>Later decide to store flags in DB:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">FeatureFlag</span> <span class="o">&lt;</span> <span class="no">ApplicationRecord</span>
<span class="k">end</span>
</code></pre>

</div>



<p>Same interface, no major changes in the rest of the app.</p>




<h2>
  
  
  7. <strong>Integrates cleanly with Pundit / CanCan / services</strong>
</h2>

<p>A model class naturally fits into authorization and service layers.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">FeatureFlagPolicy</span> <span class="o">&lt;</span> <span class="no">ApplicationPolicy</span>
  <span class="k">def</span> <span class="nf">enable?</span>
    <span class="n">user</span><span class="p">.</span><span class="nf">admin?</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<h2>
  
  
  This wouldn’t be as clean if the logic lived in a helper or initializer.
</h2>

<h2>
  
  
  Why use model classes for features?
</h2>

<p>Using a model (ActiveRecord or PORO) gives you:</p>

<ul>
<li>Clean architecture</li>
<li>Centralized logic</li>
<li>Reusability</li>
<li>Easier testing</li>
<li>Readable code</li>
<li>Extensibility if the feature grows</li>
<li>Smooth integration with Pundit and other layers</li>
</ul>




<h2>
  
  
  What about Service Object?
</h2>

<p><strong>You <em>can</em> use a service</strong> or other patterns.<br>
A model class is <em>not</em> the only valid approach.</p>

<p>The reason some teams choose a <em>model</em> instead of a <em>service</em> depends on what kind of logic they are modeling.</p>

<p>Here’s a clear breakdown.</p>


<h2>
  
  
  <strong>Why not a service? (When a service is the wrong fit)</strong>
</h2>
<h2>
  
  
  <strong>1. Services are for <em>actions</em>, not *state</strong>*
</h2>

<p>A service object usually represents something the system <em>does</em>:</p>

<ul>
<li><code>SendEmail.new.call</code></li>
<li><code>ImportCsv.new.call</code></li>
<li><code>CreateSubscription.new.call</code></li>
</ul>

<p>Services are “verb-like.”</p>

<p>Feature flags (or similar domain rules) are <em>not actions</em>.<br>
They’re <strong>state + rules about state</strong>.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">FeatureFlag</span><span class="p">.</span><span class="nf">enabled?</span><span class="p">(</span><span class="ss">:new_ui</span><span class="p">)</span>
</code></pre>

</div>



<p>A service would feel unnatural:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">FeatureFlagChecker</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="ss">:new_ui</span><span class="p">).</span><span class="nf">call</span>
</code></pre>

</div>



<p>You’re not <em>doing</em> anything; you're <em>querying domain state</em>.</p>




<h2>
  
  
  <strong>2. Services don’t naturally hold reusable domain logic</strong>
</h2>

<p>If you put your logic in a service, you often end up with this anti-pattern:</p>

<ul>
<li>One service to check feature flags</li>
<li>Another to read them</li>
<li>Another to list them</li>
<li>Another to validate something</li>
<li>Another for authorization</li>
</ul>

<p>The logic becomes scattered.</p>

<p>A model consolidates it.</p>




<h2>
  
  
  <strong>3. Services can’t be easily plugged into Rails tools</strong>
</h2>

<p>Rails gives you conveniences for models:</p>

<ul>
<li>Cleaner testing</li>
<li>Namespacing (<code>app/models</code>)</li>
<li>Integration with Pundit</li>
<li>Lazy loading / autoload</li>
<li>DB migrations if you later add persistence</li>
</ul>

<p>Services don’t get these benefits.</p>




<h2>
  
  
  <strong>Why a model is often the better fit</strong>
</h2>

<h2>
  
  
  <strong>1. It represents a *domain concept</strong>*
</h2>

<p>Feature flags, onboarding rules, eligibility rules, user states —<br>
these aren’t actions.<br>
These are <em>entities in your domain</em>.</p>

<p>Rails convention: entities → model classes.</p>

<p>Example domain models that aren’t ActiveRecord:</p>

<ul>
<li><code>Authenticator</code></li>
<li><code>Cart</code></li>
<li><code>Checkout</code></li>
<li><code>Onboarding</code></li>
<li><code>SubscriptionRules</code></li>
<li><code>FeatureFlag</code></li>
</ul>

<p>Rails devs often keep PORO domain models in <code>app/models</code>.</p>


<h2>
  
  
  <strong>2. Models provide a clear, consistent API</strong>
</h2>

<p>Models give you intuitive, domain-driven APIs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">FeatureFlag</span><span class="p">.</span><span class="nf">enabled?</span><span class="p">(</span><span class="ss">:new_ui</span><span class="p">)</span>
<span class="no">Eligibility</span><span class="p">.</span><span class="nf">for</span><span class="p">(</span><span class="n">user</span><span class="p">).</span><span class="nf">allowed?</span>
<span class="no">Onboarding</span><span class="p">.</span><span class="nf">completed?</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
<span class="no">PlanPrice</span><span class="p">.</span><span class="nf">for</span><span class="p">(</span><span class="ss">:premium</span><span class="p">)</span>
</code></pre>

</div>



<p>Services… not so much:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">FeatureFlagService</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="ss">:new_ui</span><span class="p">).</span><span class="nf">enabled?</span>
<span class="no">EligibilityService</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="n">user</span><span class="p">).</span><span class="nf">allowed?</span>
<span class="no">OnboardingService</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="n">user</span><span class="p">).</span><span class="nf">completed?</span>
</code></pre>

</div>



<p>This is more verbose, less “domain-sounding.”</p>




<h2>
  
  
  <strong>3. Models grow better over time</strong>
</h2>

<p>Feature flags start simple but later evolve:</p>

<ul>
<li>per-user access</li>
<li>groups / roles</li>
<li>rollout percentages</li>
<li>time-based enabling</li>
<li>database storage</li>
<li>admin UI</li>
<li>audit logs</li>
</ul>

<p>Models evolve cleanly.</p>

<p>Services become spaghetti when they accumulate state + logic.</p>




<h2>
  
  
  <strong>So when would a service be better?</strong>
</h2>

<p>A service is appropriate if:</p>

<ul>
<li>The feature is about <strong>executing a task</strong>
</li>
<li>Something that has a start and end</li>
<li>It produces a result or side effect</li>
</ul>

<p>Examples:</p>

<ul>
<li><code>GenerateReport</code></li>
<li><code>ChargeCreditCard</code></li>
<li><code>SendWelcomeEmail</code></li>
</ul>

<p>Not good for:</p>

<ul>
<li>feature flags</li>
<li>eligibility checks</li>
<li>rules</li>
<li>state machines</li>
<li>business constraints</li>
</ul>

<p>Those map better to models.</p>




<h2>
  
  
  <strong>What is an Interactor?</strong>
</h2>

<p>In Rails (especially with gems like <a href="https://github.com/collectiveidea/interactor" rel="noopener noreferrer">ActiveInteractor</a> or <a href="https://github.com/collectiveidea/interactor" rel="noopener noreferrer">Interactor</a>), an <strong>Interactor</strong> is a pattern for encapsulating <strong>a single unit of business logic</strong> — usually a <strong>transactional action</strong> that may:</p>

<ul>
<li>Take input</li>
<li>Perform a multi-step operation</li>
<li>Return a result (success/failure)</li>
<li>Handle errors cleanly</li>
</ul>

<p>Think of Interactors as <strong>“coordinators of actions”</strong>, often orchestrating multiple models and services.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">CreateOrder</span>
  <span class="kp">include</span> <span class="no">Interactor</span>

  <span class="k">def</span> <span class="nf">call</span>
    <span class="n">order</span> <span class="o">=</span> <span class="no">Order</span><span class="p">.</span><span class="nf">create!</span><span class="p">(</span><span class="n">context</span><span class="p">.</span><span class="nf">params</span><span class="p">)</span>
    <span class="no">PaymentProcessor</span><span class="p">.</span><span class="nf">charge</span><span class="p">(</span><span class="n">order</span><span class="p">)</span>
    <span class="no">NotificationMailer</span><span class="p">.</span><span class="nf">order_created</span><span class="p">(</span><span class="n">order</span><span class="p">).</span><span class="nf">deliver_later</span>

    <span class="n">context</span><span class="p">.</span><span class="nf">order</span> <span class="o">=</span> <span class="n">order</span>
  <span class="k">rescue</span> <span class="no">StandardError</span> <span class="o">=&gt;</span> <span class="n">e</span>
    <span class="n">context</span><span class="p">.</span><span class="nf">fail!</span><span class="p">(</span><span class="ss">error: </span><span class="n">e</span><span class="p">.</span><span class="nf">message</span><span class="p">)</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>






<h2>
  
  
  <strong>Models vs Services vs Interactors</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Concept</th>
<th>Responsibility</th>
<th>Examples</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Model</strong></td>
<td>Represents domain state &amp; rules; encapsulates attributes &amp; behavior</td>
<td>
<code>FeatureFlag</code>, <code>User</code>, <code>Subscription</code>
</td>
</tr>
<tr>
<td><strong>Service</strong></td>
<td>Performs a discrete action; can use multiple models</td>
<td>
<code>PaymentProcessor</code>, <code>EmailSender</code>
</td>
</tr>
<tr>
<td><strong>Interactor</strong></td>
<td>Orchestrates <strong>a workflow or transaction</strong> using models &amp; services; handles success/failure</td>
<td>
<code>CreateOrder</code>, <code>SendWeeklyReport</code>, <code>EnrollUserInCourse</code>
</td>
</tr>
</tbody>
</table></div>

<p><strong>Key difference:</strong></p>

<ul>
<li>Services = do <strong>one thing</strong>
</li>
<li>Interactors = <strong>orchestrate multiple things</strong> as a single business operation</li>
</ul>




<h2>
  
  
  <strong>How Interactors Fit Between Models and Services</strong>
</h2>

<ol>
<li>
<strong>Models</strong> → hold state and domain logic
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code>   <span class="no">FeatureFlag</span><span class="p">.</span><span class="nf">enabled?</span><span class="p">(</span><span class="ss">:new_ui</span><span class="p">)</span>
</code></pre>

</div>



<ol>
<li>
<strong>Services</strong> → perform actions related to one model or domain
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code>   <span class="no">PaymentProcessor</span><span class="p">.</span><span class="nf">charge</span><span class="p">(</span><span class="n">order</span><span class="p">)</span>
</code></pre>

</div>



<ol>
<li>
<strong>Interactors</strong> → coordinate multiple models and services into a single, transactional workflow
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code>   <span class="no">CreateOrder</span><span class="p">.</span><span class="nf">call</span><span class="p">(</span><span class="ss">params: </span><span class="n">order_params</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Analogy:</strong></p>

<ul>
<li>Model = <strong>the Lego bricks</strong>
</li>
<li>Service = <strong>a single Lego creation</strong> (one feature like a door or wheel)</li>
<li>Interactor = <strong>the full Lego set</strong> (puts multiple creations together into a working system)</li>
</ul>




<h2>
  
  
  <strong>When to use Interactors vs Service vs Model</strong>
</h2>

<ul>
<li>
<strong>Use a Model</strong>: for state, rules, calculations, or queries</li>
<li>
<strong>Use a Service</strong>: for single actions that act on models</li>
<li>
<strong>Use an Interactor</strong>: for multi-step workflows that may fail and need clean orchestration</li>
</ul>

<p>Example workflow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="c1"># Model</span>
<span class="k">class</span> <span class="nc">User</span><span class="p">;</span> <span class="k">end</span>
<span class="k">class</span> <span class="nc">FeatureFlag</span><span class="p">;</span> <span class="k">end</span>

<span class="c1"># Service</span>
<span class="k">class</span> <span class="nc">WelcomeEmailSender</span><span class="p">;</span> <span class="k">end</span>

<span class="c1"># Interactor</span>
<span class="k">class</span> <span class="nc">OnboardNewUser</span>
  <span class="kp">include</span> <span class="no">Interactor</span>

  <span class="k">def</span> <span class="nf">call</span>
    <span class="n">user</span> <span class="o">=</span> <span class="no">User</span><span class="p">.</span><span class="nf">create!</span><span class="p">(</span><span class="n">context</span><span class="p">.</span><span class="nf">params</span><span class="p">)</span>
    <span class="no">WelcomeEmailSender</span><span class="p">.</span><span class="nf">send</span><span class="p">(</span><span class="n">user</span><span class="p">)</span>
    <span class="n">context</span><span class="p">.</span><span class="nf">success_message</span> <span class="o">=</span> <span class="s2">"Welcome </span><span class="si">#{</span><span class="n">user</span><span class="p">.</span><span class="nf">name</span><span class="si">}</span><span class="s2">!"</span>
  <span class="k">rescue</span> <span class="o">=&gt;</span> <span class="n">e</span>
    <span class="n">context</span><span class="p">.</span><span class="nf">fail!</span><span class="p">(</span><span class="ss">error: </span><span class="n">e</span><span class="p">.</span><span class="nf">message</span><span class="p">)</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>






<h2>
  
  
  <strong>Summary</strong>
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Pattern</th>
<th>Best for</th>
<th>Bad for</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Model (PORO or ActiveRecord)</strong></td>
<td>Domain concepts, rules, states</td>
<td>One-time actions</td>
</tr>
<tr>
<td><strong>Service</strong></td>
<td>Executable actions (“do X”)</td>
<td>Representing domain objects</td>
</tr>
<tr>
<td><strong>Initializer / config</strong></td>
<td>Static rules</td>
<td>Rules that may grow or need dependencies</td>
</tr>
<tr>
<td><strong>Interactor</strong></td>
<td>Orchestrating multi-step workflows / transactions</td>
<td>Single-purpose state or simple rules</td>
</tr>
</tbody>
</table></div>

<p>Originally posted at <a href="https://devblog.launchzilla.net/Model-Classes-and-Service-Objects/" rel="noopener noreferrer">DevBlog</a>.</p>

