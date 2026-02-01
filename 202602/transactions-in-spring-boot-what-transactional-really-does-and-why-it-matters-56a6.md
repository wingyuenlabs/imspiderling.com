---
Title: Transactions in Spring Boot: What `@Transactional` Really Does (and Why It Matters)
Description: 
Author: Gianfranco Coppola
Date: 2026-02-01T21:47:54.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  1. Introduction: Why Transactions Matter More Than You Think
</h2>

<p>If you’ve been working with Spring Boot for a while, chances are you’ve already used <code>@Transactional</code>.<br><br>
Maybe you added it because “that’s what everyone does”, or because a tutorial told you so.</p>

<p>And most of the time… things <em>seem</em> to work.</p>

<p>Until one day:</p>

<ul>
<li>a payment is charged but the order is not created</li>
<li>a user is created, but their profile is missing</li>
<li>a retry suddenly creates duplicate data</li>
<li>or worse: production data ends up in an inconsistent state</li>
</ul>

<p>That’s usually the moment when you realize that <strong>transactions are not just a database feature — they are a core business concept</strong>.</p>

<p>In backend development, especially in stateful systems, <strong>partial success is often worse than total failure</strong>.<br><br>
A transaction is what allows you to say:</p>

<blockquote>
<p>“Either everything succeeds, or nothing does.”</p>
</blockquote>

<p>Spring Boot makes transactions <em>easy to use</em>, but also <em>easy to misunderstand</em>.<br><br>
And misunderstanding them can lead to subtle bugs that are extremely hard to debug.</p>

<p>In this article, we’ll start from the basics:</p>

<ul>
<li>what a transaction really is</li>
<li>why it is fundamental in backend systems</li>
<li>and how this concept maps to Spring Boot and <code>@Transactional</code>
</li>
</ul>

<p>Before touching any annotation, we need to get the mental model right.</p>


<h2>
  
  
  2. What Is a Transaction? (The Basics, Done Right)
</h2>

<p>At its core, a <strong>transaction</strong> is a logical unit of work.</p>

<p>It groups multiple operations into a <strong>single, indivisible action</strong>.</p>

<p>Think about a very common backend use case:</p>

<ul>
<li>create an order</li>
<li>decrease product stock</li>
<li>save a payment record</li>
</ul>

<p>If one of these steps fails, the system should not be left in a half-completed state.</p>

<p>Without transactions, your system might end up like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>✔ Order created
✔ Payment saved
✘ Stock update failed
</code></pre>

</div>



<p>That’s not a technical issue.<br><br>
That’s a <strong>business problem</strong>.</p>
<h3>
  
  
  The ACID Properties
</h3>

<p>Transactions are usually described using the <strong>ACID</strong> acronym. You’ve probably heard of it, but let’s translate it into practical backend terms.</p>
<h4>
  
  
  Atomicity
</h4>

<p>“All or nothing.”</p>

<p>Either all operations inside the transaction succeed, or all of them are rolled back.<br><br>
No partial updates, no “we’ll fix it later”.</p>
<h4>
  
  
  Consistency
</h4>

<p>The system moves from one valid state to another valid state.</p>

<p>Constraints, invariants, and business rules must always hold — <em>before and after</em> the transaction.</p>
<h4>
  
  
  Isolation
</h4>

<p>Concurrent transactions should not step on each other’s toes.</p>

<p>What one transaction sees (or doesn’t see) from another transaction is controlled and predictable.</p>

<p>This is where things start getting tricky — and interesting.</p>
<h4>
  
  
  Durability
</h4>

<p>Once a transaction is committed, it stays committed.</p>

<p>Even if the application crashes right after, the data is there.</p>


<h3>
  
  
  Transactions Are Not Just About Databases
</h3>

<p>This is an important point that often gets overlooked.</p>

<p>Transactions:</p>

<ul>
<li>are <strong>implemented by the database</strong>
</li>
<li>but <strong>defined by your business logic</strong>
</li>
</ul>

<p>The database doesn’t know what an “order” or a “payment” is.<br><br>
It only knows rows, tables, and constraints.</p>

<p>It’s the backend application that decides:</p>

<ul>
<li>what belongs together</li>
<li>where the transactional boundary starts</li>
<li>and where it ends</li>
</ul>

<p>This is exactly why frameworks like Spring exist:<br><br>
to help you <strong>define transactional boundaries in your application code</strong>, not in SQL scripts.</p>


<h3>
  
  
  Why Transactions Are Fundamental in Backend Systems
</h3>

<p>Modern backend systems are:</p>

<ul>
<li>concurrent</li>
<li>stateful</li>
<li>failure-prone by nature

<ul>
<li>Networks fail.</li>
<li>Databases timeout.</li>
<li>External services go down.</li>
</ul>
</li>
</ul>

<p>Transactions are your <strong>last line of defense</strong> against data corruption.</p>

<p>They don’t make failures disappear — but they make failures <em>safe</em>.</p>

<p>And that’s the key idea we’ll carry into Spring Boot and <code>@Transactional</code>.</p>


<h2>
  
  
  3. Transactions in Spring Boot: The Big Picture
</h2>

<p>Before diving deeper into <code>@Transactional</code>, it’s important to understand <strong>how Spring manages transactions at a high level</strong>.<br><br>
Not the full internal implementation — just enough to avoid the most common (and painful) mistakes.</p>

<p>Because with transactions in Spring, <strong>the “how” matters almost as much as the “what”</strong>.</p>


<h3>
  
  
  Declarative vs Programmatic Transactions
</h3>

<p>Spring supports two ways of managing transactions:</p>
<h4>
  
  
  1. Programmatic transactions
</h4>

<p>You explicitly start, commit, and rollback a transaction in code.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">TransactionStatus</span> <span class="n">status</span> <span class="o">=</span> <span class="n">transactionManager</span><span class="o">.</span><span class="na">getTransaction</span><span class="o">(</span><span class="n">definition</span><span class="o">);</span>
<span class="k">try</span> <span class="o">{</span>
    <span class="c1">// business logic</span>
    <span class="n">transactionManager</span><span class="o">.</span><span class="na">commit</span><span class="o">(</span><span class="n">status</span><span class="o">);</span>
<span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">Exception</span> <span class="n">ex</span><span class="o">)</span> <span class="o">{</span>
    <span class="n">transactionManager</span><span class="o">.</span><span class="na">rollback</span><span class="o">(</span><span class="n">status</span><span class="o">);</span>
    <span class="k">throw</span> <span class="n">ex</span><span class="o">;</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This works, but:</p>

<ul>
<li>it’s verbose</li>
<li>it mixes infrastructure concerns with business logic</li>
<li>it doesn’t scale well in complex services</li>
</ul>

<p>You <em>can</em> use it, but in most Spring Boot applications, you shouldn’t.</p>

<h4>
  
  
  2. Declarative transactions (the Spring way)
</h4>

<p>This is where <code>@Transactional</code> comes in.</p>

<p>You declare <strong>what should be transactional</strong>, not <em>how</em> to manage the transaction.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="c1">// business logic</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Spring takes care of:</p>

<ul>
<li>opening the transaction</li>
<li>committing it if everything goes well</li>
<li>rolling it back if something goes wrong</li>
</ul>

<p>This separation of concerns is one of the reasons why Spring-based backends are so readable and maintainable.</p>




<h3>
  
  
  The Role of <code>PlatformTransactionManager</code>
</h3>

<p>At runtime, Spring delegates all transaction operations to a <strong><code>PlatformTransactionManager</code></strong>.</p>

<p>Think of it as an abstraction layer between:</p>

<ul>
<li>your application</li>
<li>and the actual transaction implementation</li>
</ul>

<p>Depending on what you use, Spring will plug in a different implementation:</p>

<ul>
<li>
<code>DataSourceTransactionManager</code> → JDBC</li>
<li>
<code>JpaTransactionManager</code> → JPA / Hibernate</li>
<li>
<code>ReactiveTransactionManager</code> → reactive stacks</li>
</ul>

<p>This abstraction is what allows you to write <strong>framework-agnostic transactional code</strong>, while still being tightly integrated with your persistence technology.</p>

<p>You almost never interact with it directly — but it’s always there.</p>




<h2>
  
  
  4. What Actually Happens When You Use <code>@Transactional</code>
</h2>

<p>At first glance, <code>@Transactional</code> looks deceptively simple.</p>

<p>You put it on a method, and magically:</p>

<ul>
<li>a transaction starts</li>
<li>your logic runs</li>
<li>everything is committed or rolled back</li>
</ul>

<p>And in <em>happy-path demos</em>, that’s exactly what happens.</p>

<p>In real-world applications, however, <strong>where and how you use <code>@Transactional</code> makes a huge difference</strong>.</p>

<p>Let’s break it down.</p>




<h3>
  
  
  The Simplest (and Most Common) Use Case
</h3>

<p>The most basic usage looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">paymentRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">payment</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>If an exception is thrown during the method execution:</p>

<ul>
<li>Spring marks the transaction for rollback</li>
<li>all database changes are reverted</li>
</ul>

<p>If the method completes successfully:</p>

<ul>
<li>the transaction is committed</li>
</ul>

<p>So far, so good.</p>

<p>But this simplicity hides a lot of assumptions.</p>

<p>One of the biggest misconceptions is thinking of <code>@Transactional</code> as something that <em>adds behavior</em>.<br>
It doesn’t. It defines a <strong>transactional boundary</strong>:</p>

<p>In other words, when you annotate a method with <code>@Transactional</code>, <strong>Spring does NOT modify your method</strong>.</p>

<p>Instead, Spring:</p>

<ol>
<li> creates a <strong>proxy</strong> around your bean</li>
<li> intercepts calls to transactional methods</li>
<li> starts a transaction <em>before</em> the method execution</li>
<li> commits or rolls back <em>after</em> the method returns or throws an exception</li>
</ol>

<p>This is done using <strong>Spring AOP (Aspect-Oriented Programming)</strong>.</p>

<p>In practice, the flow looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Client → Spring Proxy → Transaction Interceptor
        → Your Method → Transaction Commit / Rollback
</code></pre>

</div>



<p>Why is this important?</p>

<p>Because <strong>only method calls that go through the proxy are transactional</strong>.</p>

<p>This single sentence explains:</p>

<ul>
<li>why self-invocation doesn’t work</li>
<li>why <code>@Transactional</code> on <code>private</code> methods is ignored</li>
<li>why calling a transactional method from the same class can silently break everything</li>
</ul>

<p>We’ll get back to this later in the pitfalls section, but keep this mental model in mind.</p>




<h3>
  
  
  Where <code>@Transactional</code> Should Live (and Where It Shouldn’t)
</h3>

<p>A common question is: <em>where do I put <code>@Transactional</code>?</em></p>

<p>The short answer:</p>

<blockquote>
<p><strong>On service-layer methods that define a business operation.</strong></p>
</blockquote>

<p>Typically:</p>

<ul>
<li>❌ Controllers → no business logic, no transactions</li>
<li>⚠️ Repositories → usually too low-level</li>
<li>✅ Services → perfect place for transactional boundaries</li>
</ul>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderService</span> <span class="o">{</span>

    <span class="nd">@Transactional</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">(</span><span class="nc">CreateOrderCommand</span> <span class="n">command</span><span class="o">)</span> <span class="o">{</span>
        <span class="c1">// validate input</span>
        <span class="c1">// persist order</span>
        <span class="c1">// update stock</span>
        <span class="c1">// trigger side effects</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This makes your transactional boundary:</p>

<ul>
<li>explicit</li>
<li>easy to reason about</li>
<li>aligned with business use cases</li>
</ul>




<h3>
  
  
  Class-Level vs Method-Level <code>@Transactional</code>
</h3>

<p>Spring allows you to annotate:</p>

<ul>
<li>a single method</li>
<li>or the entire class
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderService</span> <span class="o">{</span>
    <span class="o">...</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This means:</p>

<ul>
<li>
<strong>every public method</strong> is transactional by default</li>
<li>unless overridden at method level</li>
</ul>

<p>This can be useful, but it’s also dangerous if overused.</p>

<p>From experience:</p>

<ul>
<li>class-level works well for <em>simple</em> services</li>
<li>method-level is safer for complex ones</li>
</ul>

<p>Explicit is better than implicit — especially with transactions.</p>




<p>At this point, we have a solid foundation:</p>

<ul>
<li>what a transaction is</li>
<li>why it matters</li>
<li>how Spring manages it under the hood</li>
</ul>




<h2>
  
  
  5. Understanding <code>@Transactional</code> Attributes (The Real Ones)
</h2>

<p><code>@Transactional</code> is not just an on/off switch.</p>

<p>Behind that single annotation there are <strong>rules that control how transactions behave</strong>, especially when:</p>

<ul>
<li>multiple methods interact</li>
<li>exceptions are thrown</li>
<li>concurrent operations happen</li>
</ul>

<p>Most bugs related to transactions come from <strong>default assumptions</strong> that turn out to be wrong.</p>

<p>Let’s go through the attributes that actually matter in real projects.</p>




<h3>
  
  
  5.1 Propagation: How Transactions Interact with Each Other
</h3>

<p>Propagation defines <strong>what happens when a transactional method is called from another transactional method</strong>.</p>

<p>This is by far the most important attribute.</p>

<h4>
  
  
  <code>REQUIRED</code> (Default)
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">propagation</span> <span class="o">=</span> <span class="nc">Propagation</span><span class="o">.</span><span class="na">REQUIRED</span><span class="o">)</span>
</code></pre>

</div>



<p>Meaning:</p>

<ul>
<li>join the existing transaction if there is one</li>
<li>otherwise, create a new transaction</li>
</ul>

<p>This is what you want <strong>most of the time</strong>.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderService</span><span class="o">.</span><span class="na">saveOrder</span><span class="o">();</span>
    <span class="n">paymentService</span><span class="o">.</span><span class="na">charge</span><span class="o">();</span>
<span class="o">}</span>
</code></pre>

</div>



<p>If <code>charge()</code> fails:</p>

<ul>
<li>the entire transaction rolls back</li>
<li><strong>nothing is persisted</strong></li>
</ul>

<p>This is usually correct and desirable.</p>




<h4>
  
  
  <code>REQUIRES_NEW</code>
</h4>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">propagation</span> <span class="o">=</span> <span class="nc">Propagation</span><span class="o">.</span><span class="na">REQUIRES_NEW</span><span class="o">)</span>
</code></pre>

</div>



<p>Meaning:</p>

<ul>
<li>suspend the existing transaction</li>
<li>start a completely new one</li>
</ul>

<p>Classic use case:</p>

<ul>
<li>audit logs</li>
<li>technical events</li>
<li>actions that must persist <strong>even if the main transaction fails</strong>
</li>
</ul>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">auditService</span><span class="o">.</span><span class="na">logOrderAttempt</span><span class="o">(</span><span class="n">order</span><span class="o">);</span> <span class="c1">// REQUIRES_NEW</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nf">RuntimeException</span><span class="o">(</span><span class="s">"Payment failed"</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Result:</p>

<ul>
<li>order → rolled back</li>
<li>audit log → committed</li>
</ul>

<p>This is powerful — and dangerous if misused.</p>




<h4>
  
  
  Other Propagation Types (Quick but Honest)
</h4>

<ul>
<li>
<code>SUPPORTS</code> → join if exists, otherwise run non-transactionally</li>
<li>
<code>MANDATORY</code> → fail if no transaction exists</li>
<li>
<code>NOT_SUPPORTED</code> → suspend any existing transaction</li>
<li>
<code>NEVER</code> → fail if a transaction exists</li>
<li>
<code>NESTED</code> → savepoints (DB-dependent)</li>
</ul>

<p>👉 In most applications:</p>

<ul>
<li>
<code>REQUIRED</code> and <code>REQUIRES_NEW</code> cover 95% of use cases</li>
<li>the others are niche and should be used intentionally</li>
</ul>




<h3>
  
  
  5.2 Isolation: How Much You See of Other Transactions
</h3>

<p>Isolation defines <strong>how concurrent transactions affect each other</strong>.</p>

<p>Default:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">isolation</span> <span class="o">=</span> <span class="nc">Isolation</span><span class="o">.</span><span class="na">DEFAULT</span><span class="o">)</span>
</code></pre>

</div>



<p>This delegates to the database default (often <code>READ_COMMITTED</code>).</p>

<h4>
  
  
  The Practical Levels
</h4>

<ul>
<li>
<strong>READ_COMMITTED</strong>
You only see committed data.
Good balance between consistency and performance.</li>
<li>
<strong>REPEATABLE_READ</strong>
Data read once won’t change during the transaction.
Prevents non-repeatable reads.</li>
<li>
<strong>SERIALIZABLE</strong>
Full isolation. Transactions behave as if executed sequentially.
Very safe. Very expensive.</li>
</ul>

<p>Higher isolation = fewer anomalies = lower throughput.</p>

<p>👉 Rule of thumb:</p>

<ul>
<li>trust your DB defaults</li>
<li>increase isolation only when you have a <em>proven</em> concurrency problem</li>
</ul>




<h3>
  
  
  5.3 Rollback Rules: The Most Common Source of Bugs
</h3>

<p>This is where many Spring developers get burned.</p>

<p>By default:</p>

<blockquote>
<p><strong>Spring rolls back only on unchecked exceptions (<code>RuntimeException</code>) and <code>Error</code>.</strong></p>
</blockquote>

<p>That means this will <strong>NOT rollback</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="kd">throws</span> <span class="nc">Exception</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nf">Exception</span><span class="o">(</span><span class="s">"Checked exception"</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>From a business perspective, this operation <strong>clearly failed</strong>.<br><br>
From Spring’s perspective, however, this is a <em>checked exception</em> — and the transaction is committed.</p>

<p>No rollback. No warning. Just inconsistent data.<br>
Yes, really.</p>
<h4>
  
  
  Explicit Rollback Rules
</h4>

<p>You can override this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">rollbackFor</span> <span class="o">=</span> <span class="nc">Exception</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
</code></pre>

</div>



<p>Or the opposite:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">noRollbackFor</span> <span class="o">=</span> <span class="nc">BusinessException</span><span class="o">.</span><span class="na">class</span><span class="o">)</span>
</code></pre>

</div>



<p>This is essential when:</p>

<ul>
<li>using checked exceptions</li>
<li>modeling business failures explicitly</li>
</ul>




<h4>
  
  
  A Better Approach: Business Exceptions as Runtime Exceptions
</h4>

<p>In most real-world Spring Boot applications, business failures <strong>should invalidate the transaction</strong>.</p>

<p>A clean and effective way to model this is by using <strong>custom unchecked exceptions</strong>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="kd">public</span> <span class="kd">class</span> <span class="nc">PaymentFailedException</span> <span class="kd">extends</span> <span class="nc">RuntimeException</span> <span class="o">{</span>

<span class="o">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nf">PaymentFailedException</span><span class="o">();</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This approach has several advantages:</p>

<ul>
<li>rollback happens automatically</li>
<li>transactional behavior is explicit</li>
<li>no need for extra configuration</li>
<li>business intent is clear</li>
</ul>

<p>If the operation fails, the transaction fails. No ambiguity.</p>

<p>👉 <strong>Always be explicit</strong> if you rely on checked exceptions.</p>




<h3>
  
  
  5.4 <code>readOnly</code>: Small Flag, Big Impact
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">readOnly</span> <span class="o">=</span> <span class="kc">true</span><span class="o">)</span>
</code></pre>

</div>



<p>This:</p>

<ul>
<li>hints the persistence provider</li>
<li>may optimize flushing and dirty checking</li>
<li>documents intent clearly</li>
</ul>

<p>Perfect for:</p>

<ul>
<li>query-only service methods</li>
<li>read-heavy paths</li>
</ul>

<p>Not a silver bullet — but a good habit.</p>




<h3>
  
  
  5.5 <code>timeout</code>: A Safety Net
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span><span class="o">(</span><span class="n">timeout</span> <span class="o">=</span> <span class="mi">5</span><span class="o">)</span>
</code></pre>

</div>



<p>If the transaction runs longer than 5 seconds:</p>

<ul>
<li>it’s rolled back</li>
</ul>

<p>Useful for:</p>

<ul>
<li>protecting DB resources</li>
<li>preventing stuck transactions</li>
</ul>

<p>Especially relevant under load.</p>




<h3>
  
  
  A Hard-Earned Lesson
</h3>

<p>Most transactional bugs are not caused by:</p>

<ul>
<li>wrong SQL</li>
<li>broken databases</li>
</ul>

<p>They come from:</p>

<ul>
<li>wrong assumptions about propagation</li>
<li>unexpected rollback behavior</li>
<li>hidden transactional boundaries</li>
</ul>

<p>Understanding these attributes turns <code>@Transactional</code> from a “magic annotation” into a <strong>precise tool</strong>.</p>




<h2>
  
  
  6. Common Transactional Pitfalls in Spring Boot
</h2>

<p>At this point, we understand how transactions <em>should</em> work.</p>

<p>Unfortunately, many transactional bugs don’t come from a lack of knowledge —<br><br>
they come from <strong>small details that are easy to miss and hard to debug</strong>.</p>

<p>Let’s go through the most common pitfalls you’ll encounter in real Spring Boot applications.</p>


<h3>
  
  
  6.1 Self-Invocation: The Silent Transaction Killer
</h3>

<p>This is probably <strong>the most famous Spring transactional pitfall</strong>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderService</span> <span class="o">{</span>

    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
        <span class="n">saveOrder</span><span class="o">();</span> <span class="c1">// ❌ no transaction</span>
    <span class="o">}</span>

    <span class="nd">@Transactional</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">saveOrder</span><span class="o">()</span> <span class="o">{</span>
        <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>At first glance, this looks fine.</p>

<p>It’s not.</p>

<h4>
  
  
  What’s the problem?
</h4>

<p>The call to <code>saveOrder()</code> happens <strong>inside the same class</strong>.<br><br>
It never goes through the Spring proxy.</p>

<p>Result:</p>

<ul>
<li>
<code>@Transactional</code> is completely ignored</li>
<li>no transaction is started</li>
<li>no error, no warning</li>
</ul>

<p>This is one of the reasons transactional bugs feel “random”.</p>
<h4>
  
  
  How to fix it
</h4>

<ul>
<li>move the transactional method to another bean</li>
<li>or make sure it’s called from outside the class
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Service</span>
<span class="kd">public</span> <span class="kd">class</span> <span class="nc">OrderPersistenceService</span> <span class="o">{</span>

    <span class="nd">@Transactional</span>
    <span class="kd">public</span> <span class="kt">void</span> <span class="nf">saveOrder</span><span class="o">(</span><span class="nc">Order</span> <span class="n">order</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<h3>
  
  
  6.2 <code>@Transactional</code> on <code>private</code> (or non-public) Methods
</h3>

<p>Another classic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">private</span> <span class="kt">void</span> <span class="nf">saveOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="o">...</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This will <strong>never work</strong>.</p>

<p>Spring proxies intercept <strong>public method calls only</strong> (by default).<br><br>
Private, protected, or package-private methods are ignored.</p>

<p>Again:</p>

<ul>
<li>no exception</li>
<li>no warning</li>
<li>just no transaction</li>
</ul>

<p>👉 <strong>Transactional methods must be public. Always.</strong></p>


<h3>
  
  
  6.3 Catching Exceptions and Accidentally Preventing Rollback
</h3>

<p>This one is subtle — and extremely common.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="k">try</span> <span class="o">{</span>
        <span class="n">paymentService</span><span class="o">.</span><span class="na">charge</span><span class="o">();</span>
    <span class="o">}</span> <span class="k">catch</span> <span class="o">(</span><span class="nc">PaymentFailedException</span> <span class="n">ex</span><span class="o">)</span> <span class="o">{</span>
        <span class="n">log</span><span class="o">.</span><span class="na">error</span><span class="o">(</span><span class="s">"Payment failed"</span><span class="o">,</span> <span class="n">ex</span><span class="o">);</span>
    <span class="o">}</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Looks harmless, right?</p>

<p>But now:</p>

<ul>
<li>the exception is swallowed</li>
<li>the method completes normally</li>
<li>the transaction is <strong>committed</strong>
</li>
</ul>

<p>Spring rolls back <strong>only if the exception escapes the transactional boundary</strong>.</p>

<h4>
  
  
  Correct approaches
</h4>

<p>Either:</p>

<ul>
<li>rethrow the exception</li>
<li>or mark the transaction for rollback manually
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nc">TransactionAspectSupport</span><span class="o">.</span><span class="na">currentTransactionStatus</span><span class="o">().</span><span class="na">setRollbackOnly</span><span class="o">();</span>
</code></pre>

</div>



<p>But in most cases, <strong>rethowing is the cleanest solution</strong>.</p>




<h3>
  
  
  6.4 Mixing Transactional and Non-Transactional Logic
</h3>

<p>Sometimes transactional methods grow too much:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">emailService</span><span class="o">.</span><span class="na">sendConfirmationEmail</span><span class="o">();</span> <span class="c1">// ❌</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Problems:</p>

<ul>
<li>emails are slow</li>
<li>emails can fail</li>
<li>emails should not be part of a DB transaction</li>
</ul>

<p>If the email fails:</p>

<ul>
<li>do you really want to rollback the order?</li>
</ul>

<p>Probably not.</p>

<h4>
  
  
  Better approach
</h4>

<ul>
<li>keep transactions short</li>
<li>isolate side effects</li>
<li>use events or async processing</li>
</ul>

<p>Transactional boundaries should protect <strong>data consistency</strong>, not external systems.</p>




<h3>
  
  
  6.5 Unexpected Rollbacks (<code>UnexpectedRollbackException</code>)
</h3>

<p>Sooner or later, you’ll see this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>UnexpectedRollbackException: Transaction silently rolled back
</code></pre>

</div>



<p>This usually happens when:</p>

<ul>
<li>an inner transactional method marks the transaction as rollback-only</li>
<li>but the outer method tries to commit</li>
</ul>

<p>Common causes:</p>

<ul>
<li>caught exceptions inside nested transactional calls</li>
<li>mixed propagation settings</li>
<li>overuse of <code>REQUIRES_NEW</code>
</li>
</ul>

<p>When you see this exception:</p>

<ul>
<li>don’t look at the commit</li>
<li>look at <strong>what marked the transaction as rollback-only earlier</strong>
</li>
</ul>




<h3>
  
  
  6.6 Long-Running Transactions
</h3>

<p>Technically correct. Practically dangerous.</p>

<p>Long transactions:</p>

<ul>
<li>lock rows for too long</li>
<li>reduce throughput</li>
<li>increase deadlock probability</li>
</ul>

<p>Common causes:</p>

<ul>
<li>doing I/O inside transactions</li>
<li>calling remote services</li>
<li>waiting for user input (yes, it happens…)</li>
</ul>

<p>Rule:</p>

<blockquote>
<p><strong>Transactions should be as short as possible, but as long as necessary.</strong></p>
</blockquote>




<h3>
  
  
  A Pattern You’ll Start to Recognize
</h3>

<p>Most transactional problems share a common theme:</p>

<ul>
<li>the code <em>looks</em> correct</li>
<li>the behavior is implicit</li>
<li>Spring does exactly what you told it to do — not what you meant</li>
</ul>

<p>That’s why:</p>

<ul>
<li>understanding proxies</li>
<li>defining clear boundaries</li>
<li>modeling failures explicitly</li>
</ul>

<p>…is more important than memorizing annotations.</p>




<h2>
  
  
  7. <code>@Transactional</code> and <code>@Async</code>: A Dangerous Combination
</h2>

<p>At some point, almost every Spring Boot developer tries to combine:</p>

<ul>
<li>
<code>@Transactional</code> → consistency</li>
<li>
<code>@Async</code> → performance</li>
</ul>

<p>On paper, it sounds like a great idea.</p>

<p>In practice, it’s one of the <strong>most misunderstood and dangerous combinations</strong> in Spring.</p>

<p>Let’s clear things up.</p>




<h3>
  
  
  The Core Problem: Different Threads, Different Transactions
</h3>

<p><code>@Transactional</code> is <strong>thread-bound</strong>.</p>

<p>A transaction:</p>

<ul>
<li>is associated with the current thread</li>
<li>lives and dies inside that thread</li>
</ul>

<p><code>@Async</code>, on the other hand:</p>

<ul>
<li>executes the method in a <strong>different thread</strong>
</li>
<li>outside the original call stack</li>
</ul>

<p>So this code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">asyncService</span><span class="o">.</span><span class="na">sendConfirmationEmail</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Async</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">sendConfirmationEmail</span><span class="o">(</span><span class="nc">Order</span> <span class="n">order</span><span class="o">)</span> <span class="o">{</span>
    <span class="c1">// ...</span>
<span class="o">}</span>
</code></pre>

</div>



<p>does <strong>not</strong> mean:</p>

<blockquote>
<p>“send the email in the same transaction, but asynchronously”</p>
</blockquote>

<p>It means:</p>

<blockquote>
<p>“start a completely separate execution, with no transaction at all (unless explicitly defined)”</p>
</blockquote>




<h3>
  
  
  The Most Common Wrong Assumption
</h3>

<p>Many developers assume:</p>

<blockquote>
<p>“If the async method is called from a transactional one, it participates in the same transaction.”</p>
</blockquote>

<p>It doesn’t.</p>

<p>Ever.</p>

<p>Different thread = different transactional context.</p>




<h3>
  
  
  What Happens in Practice
</h3>

<p>Let’s look at a slightly more subtle example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">asyncService</span><span class="o">.</span><span class="na">notifyWarehouse</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="k">throw</span> <span class="k">new</span> <span class="nf">RuntimeException</span><span class="o">(</span><span class="s">"Payment failed"</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Possible outcome:</p>

<ul>
<li>transaction rolls back</li>
<li>order is NOT persisted</li>
<li>async method <strong>still runs</strong>
</li>
<li>warehouse is notified about an order that doesn’t exist</li>
</ul>

<p>This is how distributed inconsistencies are born.</p>




<h3>
  
  
  Making <code>@Async</code> Transactional (Yes, But…)
</h3>

<p>You <em>can</em> put <code>@Transactional</code> on an async method:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Async</span>
<span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">notifyWarehouse</span><span class="o">(</span><span class="nc">Order</span> <span class="n">order</span><span class="o">)</span> <span class="o">{</span>
    <span class="o">...</span>
<span class="o">}</span>
</code></pre>

</div>



<p>This creates:</p>

<ul>
<li>a <strong>new transaction</strong>
</li>
<li>completely independent from the original one</li>
</ul>

<p>This might be fine — or disastrous — depending on intent.</p>

<p>Again: <strong>there is no shared transaction</strong>.</p>




<h3>
  
  
  Better Patterns Than <code>@Transactional + @Async</code>
</h3>

<h4>
  
  
  1. Transactional Events
</h4>

<p>Spring provides a much safer mechanism:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@Transactional</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">placeOrder</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">orderRepository</span><span class="o">.</span><span class="na">save</span><span class="o">(</span><span class="n">order</span><span class="o">);</span>
    <span class="n">applicationEventPublisher</span><span class="o">.</span><span class="na">publishEvent</span><span class="o">(</span><span class="k">new</span> <span class="nc">OrderPlacedEvent</span><span class="o">(</span><span class="n">order</span><span class="o">));</span>
<span class="o">}</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@TransactionalEventListener</span><span class="o">(</span><span class="n">phase</span> <span class="o">=</span> <span class="nc">TransactionPhase</span><span class="o">.</span><span class="na">AFTER_COMMIT</span><span class="o">)</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">onOrderPlaced</span><span class="o">(</span><span class="nc">OrderPlacedEvent</span> <span class="n">event</span><span class="o">)</span> <span class="o">{</span>
    <span class="n">sendEmail</span><span class="o">(</span><span class="n">event</span><span class="o">);</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Now:</p>

<ul>
<li>the event is processed <strong>only if the transaction commits</strong>
</li>
<li>no ghost side effects</li>
<li>no inconsistent state</li>
</ul>

<p>This pattern is gold.</p>




<h4>
  
  
  2. Messaging / Event-Driven Architecture
</h4>

<p>For more complex systems:</p>

<ul>
<li>Kafka</li>
<li>RabbitMQ</li>
<li>cloud queues</li>
</ul>

<p>Persist state first, then publish events.</p>

<p>Transactions protect <strong>your database</strong>, not the world.</p>




<h3>
  
  
  A Simple Rule That Saves a Lot of Pain
</h3>

<blockquote>
<p><strong>Never assume an async operation is part of your transaction.<br><br>
It never is.</strong></p>
</blockquote>

<p>If consistency matters:</p>

<ul>
<li>finish the transaction</li>
<li>then trigger async behavior</li>
</ul>

<p>Always in that order.</p>




<h3>
  
  
  Final Thought on <code>@Async</code>
</h3>

<p><code>@Async</code> is not dangerous by itself.</p>

<p>What’s dangerous is:</p>

<ul>
<li>mixing it with transactions</li>
<li>without understanding thread boundaries</li>
</ul>

<p>Once you internalize this model, the behavior becomes predictable — and safe.</p>




<h2>
  
  
  8. Transaction Logging and Debugging
</h2>

<p>One of the most frustrating things about transactional bugs is that <strong>everything looks fine until it’s not</strong>.</p>

<p>No errors.<br><br>
No stack traces.<br><br>
Just data in the wrong state.</p>

<p>When that happens, logging is often the <strong>only way to understand what Spring is actually doing</strong>.</p>

<p>Let’s see how to make transactions visible.</p>


<h3>
  
  
  Why Transactional Bugs Are Hard to Debug
</h3>

<p>Transactional behavior is:</p>

<ul>
<li>implicit</li>
<li>proxy-based</li>
<li>spread across multiple layers</li>
</ul>

<p>So when a transaction:</p>

<ul>
<li>starts</li>
<li>commits</li>
<li>rolls back</li>
<li>or is marked as rollback-only</li>
</ul>

<p>…it usually happens <strong>outside your business code</strong>.</p>

<p>Without proper logs, you’re debugging blind.</p>


<h3>
  
  
  Enabling Transaction Logs in Spring Boot
</h3>

<p>Spring exposes very useful logs — you just need to turn them on.</p>

<p>In <code>application.yml</code> (or <code>application.properties</code>):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">logging</span><span class="pi">:</span>
  <span class="na">level</span><span class="pi">:</span>
    <span class="na">org.springframework.transaction</span><span class="pi">:</span> <span class="s">DEBUG</span>
</code></pre>

</div>



<p>This alone already shows:</p>

<ul>
<li>when a transaction is created</li>
<li>when it’s committed</li>
<li>when it’s rolled back</li>
</ul>




<h3>
  
  
  Logging Transaction Boundaries
</h3>

<p>With transaction logging enabled, you’ll start seeing logs like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Creating new transaction with name [OrderService.placeOrder]
Participating in existing transaction
Committing JDBC transaction
Rolling back JDBC transaction
</code></pre>

</div>



<p>This tells you:</p>

<ul>
<li>
<strong>where</strong> the transaction starts</li>
<li>
<strong>which method</strong> owns it</li>
<li><strong>how nested calls behave</strong></li>
</ul>

<p>When debugging propagation issues, this is invaluable.</p>




<h3>
  
  
  Hibernate / JPA Logs: Seeing What Actually Hits the DB
</h3>

<p>Transactions are about <em>when</em> changes are flushed.</p>

<p>To see <em>what</em> is executed, enable SQL logs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">logging</span><span class="pi">:</span>
  <span class="na">level</span><span class="pi">:</span>
    <span class="na">org.hibernate.SQL</span><span class="pi">:</span> <span class="s">DEBUG</span>
</code></pre>

</div>



<p>Optionally, parameter binding:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">logging</span><span class="pi">:</span>
  <span class="na">level</span><span class="pi">:</span>
    <span class="na">org.hibernate.type.descriptor.sql</span><span class="pi">:</span> <span class="s">TRACE</span>
</code></pre>

</div>



<p>Now you can correlate:</p>

<ul>
<li>transaction boundaries</li>
<li>SQL statements</li>
<li>commit / rollback events</li>
</ul>

<p>This is often where inconsistencies finally make sense.</p>




<h3>
  
  
  Debugging Rollbacks That “Come from Nowhere”
</h3>

<p>If you’ve ever seen this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>UnexpectedRollbackException: Transaction silently rolled back
</code></pre>

</div>



<p>it means:</p>

<ul>
<li>the transaction was marked as rollback-only earlier</li>
<li>but the outer layer tried to commit it anyway</li>
</ul>

<p>To debug this:</p>

<ol>
<li> enable transaction logs</li>
<li> look for a <code>setRollbackOnly</code> event</li>
<li> check for swallowed exceptions</li>
<li> inspect nested transactional methods</li>
</ol>

<p>The rollback never comes from nowhere — it’s just hidden.</p>




<h3>
  
  
  Business Logging vs Transaction Logging
</h3>

<p>A common mistake is relying only on business logs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"Order placed successfully"</span><span class="o">);</span>
</code></pre>

</div>



<p>This log may appear:</p>

<ul>
<li>
<strong>before</strong> the transaction commits</li>
<li>even if the transaction later rolls back</li>
</ul>

<p>If you need certainty, log:</p>

<ul>
<li>after commit</li>
<li>or via transactional events
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight java"><code><span class="nd">@TransactionalEventListener</span><span class="o">(</span><span class="n">phase</span> <span class="o">=</span> <span class="nc">TransactionPhase</span><span class="o">.</span><span class="na">AFTER_COMMIT</span><span class="o">)</span>
<span class="kd">public</span> <span class="kt">void</span> <span class="nf">onOrderCommitted</span><span class="o">(</span><span class="nc">OrderPlacedEvent</span> <span class="n">event</span><span class="o">)</span> <span class="o">{</span>
    <span class="n">log</span><span class="o">.</span><span class="na">info</span><span class="o">(</span><span class="s">"Order committed: {}"</span><span class="o">,</span> <span class="n">event</span><span class="o">.</span><span class="na">getOrderId</span><span class="o">());</span>
<span class="o">}</span>
</code></pre>

</div>



<p>Now your logs reflect <strong>reality</strong>, not intent.</p>




<h3>
  
  
  A Debugging Workflow That Actually Works
</h3>

<p>When dealing with transactional issues:</p>

<ol>
<li> Enable Spring transaction logs</li>
<li> Enable SQL logs</li>
<li> Identify transaction boundaries</li>
<li> Track exception flow</li>
<li> Verify commit / rollback timing</li>
</ol>

<p>This approach turns “random behavior” into <strong>deterministic behavior</strong>.</p>




<h3>
  
  
  Final Takeaway
</h3>

<p>Transactions don’t fail silently.</p>

<p>They fail <em>quietly</em>.</p>

<p>Logging is what gives them a voice.</p>

<p>Once you get used to reading transaction logs, you’ll start spotting problems <strong>before</strong> they reach production.</p>




<h2>
  
  
  9. Conclusion &amp; Takeaways
</h2>

<p>Transactions are one of the most powerful tools in Spring Boot — but they are also one of the most misunderstood.</p>

<p>Here’s what you should remember:</p>

<ol>
<li> <strong>Transactions are about business consistency</strong>, not just database operations.
Define your transactional boundaries around <em>business operations</em>, not technical details.</li>
<li> <strong><code>@Transactional</code> is declarative, but precise.</strong>
Understand propagation, isolation, rollback rules, and method visibility.</li>
<li>
<p><strong>Exceptions drive rollbacks.</strong></p>

<ul>
<li>Unchecked exceptions → rollback by default</li>
<li>Checked exceptions → explicit rollback required
Model your business exceptions carefully.</li>
</ul>
</li>
<li>
<p><strong>Avoid common pitfalls:</strong></p>

<ul>
<li>Self-invocation</li>
<li>Private methods</li>
<li>Catching exceptions and swallowing them</li>
<li>Long-running transactions</li>
<li>Mixing async and transactional logic without awareness</li>
</ul>
</li>
<li><p><strong>Logging is your friend.</strong><br><br>
Enable transaction and SQL logging to debug propagation, rollback, and commit behavior. Use <code>@TransactionalEventListener</code> for post-commit business logging.</p></li>
<li><p><strong>Async and transactions are tricky.</strong><br><br>
Transactions are thread-bound. Async methods run in a different thread and have a separate transactional context. Prefer events or queues for safe decoupling.</p></li>
</ol>




<h3>
  
  
  Final Thought
</h3>

<p>Spring Boot gives you <strong>powerful tools</strong>, but with great power comes great responsibility.<br><br>
Transactions can protect your data, but only if you understand how they work — not just how they look.</p>




<p>💬 <strong>I’d love to hear from you:</strong></p>

<ul>
<li>What are the trickiest transactional issues you’ve faced?</li>
<li>Do you have any favorite patterns for async + transactional operations?</li>
<li>Any hidden pitfalls you’ve learned the hard way?</li>
</ul>

<p>Share your experiences in the comments — let’s learn from each other.</p>

