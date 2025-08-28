---
Title: Better Sidekiq Classes
Description: 
Author: Braden King
Date: 2025-08-28T22:12:04.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://github.com/sidekiq/sidekiq" rel="noopener noreferrer">Sidekiq</a> is pretty much the go-to solution for enqueuing jobs for background processing when working on a Ruby-based project. It's simple to implement, has a clear DSL, and is well-supported by common testing frameworks like RSpec. That being said, and I may be nitpicking, but the standard implementation examples do add a little extra overhead if you wish to be more flexible with your job business logic, and additional test setup is required.</p>

<h2>
  
  
  The "normal" way
</h2>

<p>Given we want a little background job that updates a user's name and any other business logic needed (to keep the example simple), we could create a standard Sidekiq job with that logic and call it from our code as such:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">UpdateUserJob</span>
  <span class="kp">include</span> <span class="no">Sidekiq</span><span class="o">::</span><span class="no">Job</span>

  <span class="k">def</span> <span class="nf">perform</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="nb">name</span><span class="p">)</span>
    <span class="n">user</span> <span class="o">=</span> <span class="no">User</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="n">user_id</span><span class="p">)</span>
    <span class="n">user</span><span class="p">.</span><span class="nf">update!</span><span class="p">(</span><span class="ss">name: </span><span class="nb">name</span><span class="p">)</span>
    <span class="c1"># .. additional business logic here</span>
  <span class="k">end</span>
<span class="k">end</span>

<span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">perform_async</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">)</span> <span class="c1"># perform the job as soon as possible</span>
<span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">perform_in</span><span class="p">(</span><span class="mi">5</span><span class="p">.</span><span class="nf">minutes</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">)</span> <span class="c1"># delay the processing by 5 minutes</span>
</code></pre>

</div>



<p>This is simple enough. But what if we want to test the logic being performed inside this job? Firstly, we'd need to update our <code>spec_helper</code> to let it know what to do with a job. If you're using RSpec, you might even need to add the <a href="https://github.com/philostler/rspec-sidekiq" rel="noopener noreferrer">rspec-sidekiq</a> gem for additional configuration and matcher options. A classic-looking spec for this job might look something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="n">it</span> <span class="s2">"updates the user"</span> <span class="k">do</span>
  <span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">perform_async</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">)</span>
  <span class="n">expect</span><span class="p">(</span><span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">jobs</span><span class="p">.</span><span class="nf">size</span><span class="p">).</span><span class="nf">to</span> <span class="n">eq</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
  <span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">drain</span>
  <span class="n">expect</span><span class="p">(</span><span class="no">User</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">name</span><span class="p">).</span><span class="nf">to</span> <span class="n">eq</span><span class="p">(</span><span class="s2">"bob"</span><span class="p">)</span>
<span class="k">end</span>
</code></pre>

</div>



<p>We need to <code>drain</code> the job before we can test the result of its work. Alternatively, we could wrap the test inside one of the Sidekiq testing helper methods:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="n">it</span> <span class="s2">"updates the user"</span> <span class="k">do</span>
  <span class="no">Sidekiq</span><span class="o">::</span><span class="no">Testing</span><span class="p">.</span><span class="nf">inline!</span> <span class="k">do</span>
    <span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">perform_async</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">)</span>
    <span class="n">expect</span><span class="p">(</span><span class="no">User</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">name</span><span class="p">).</span><span class="nf">to</span> <span class="n">eq</span><span class="p">(</span><span class="s2">"bob"</span><span class="p">)</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>This does work as expected however it does leave a little to be desired. We have to remember to do this whenever testing job logic (the number of times I've had to search through previous specs to remind myself of the syntax), it adds a few extra lines to the tests which are not necessary and there is a spec performance hit as well.</p>

<h2>
  
  
  The "use case" way
</h2>

<p>So, what options do we have? Well, the first option would be to extract the logic performed by the job into a class which we would then pass the job attributes to for processing. That way we can independently test the business logic without having to mess with background workers and the like. I have used various names for these classes, <strong>Use Case</strong>, <strong>Resource</strong>, or <strong>Library</strong> to name a few (use whatever best suits you and your conventions). Our new job would look something like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">UpdateUserJob</span>
  <span class="kp">include</span> <span class="no">Sidekiq</span><span class="o">::</span><span class="no">Job</span>

  <span class="k">def</span> <span class="nf">perform</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="nb">name</span><span class="p">)</span>
    <span class="no">UpdateUserUseCase</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="nb">name</span><span class="p">).</span><span class="nf">call</span>
  <span class="k">end</span>
<span class="k">end</span>

<span class="k">class</span> <span class="nc">UpdateUserUseCase</span>
  <span class="k">def</span> <span class="nf">initialize</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="nb">name</span><span class="p">)</span>
    <span class="vi">@user_id</span> <span class="o">=</span> <span class="n">user_id</span>
    <span class="vi">@name</span> <span class="o">=</span> <span class="nb">name</span>
  <span class="k">end</span>

  <span class="k">def</span> <span class="nf">call</span>
    <span class="n">user</span> <span class="o">=</span> <span class="no">User</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="vi">@user_id</span><span class="p">)</span>
    <span class="n">user</span><span class="p">.</span><span class="nf">update!</span><span class="p">(</span><span class="ss">name: </span><span class="vi">@name</span><span class="p">)</span>
    <span class="c1"># .. additional business logic here</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>Now, when it comes to testing, we could just forgo testing the actual job class as it does nothing except pass on some arguments to a plain old Ruby object. We can just test the class directly. Sidekiq is also already extensively tested so there is little point in testing whether the jobs on the queue have increased or not unless you are expecting new jobs to be created as a knock-on effect of your business logic.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="n">it</span> <span class="s2">"updates the user"</span> <span class="k">do</span>
  <span class="no">UpdateUserUseCase</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">).</span><span class="nf">call</span>
  <span class="n">expect</span><span class="p">(</span><span class="no">User</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="mi">1</span><span class="p">).</span><span class="nf">name</span><span class="p">).</span><span class="nf">to</span> <span class="n">eq</span><span class="p">(</span><span class="s2">"bob"</span><span class="p">)</span>
<span class="k">end</span>
</code></pre>

</div>



<p>The added benefit to doing it this way is that you have a plain old Ruby object that you can call independently and inline in your code elsewhere should the need arise. For instance, you may want to defer the processing to the background when handling a mass import of data, however, when being performed as a single HTTP request to your API, you could wish to do it inline.</p>

<p>The downside to this is that we now, however, have had to create an additional class to provide this flexibility. That means, if we wish to keep this convention going, for each <code>Job</code> class, there would be a corresponding <code>UseCase</code> class. What if we could take this idea one step further? What if we could turn our Job class into a plain old Ruby object right from the start? Let's rewrite our <code>UpdateUserJob</code> class.</p>

<h2>
  
  
  The "runner" way (with POROs)
</h2>

<p>Let's pretend you had never heard of Sidekiq or background queues, and you want to write some code to perform a specific task. For the sake of convention, I have ended the following class name with "Job" to match previous examples however by syntax definition, it is no longer what we would call a classic "job". It is now just a Plain Old Ruby Object (PORO; identical to our previous <code>UpdateUserUseCase</code>).<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">class</span> <span class="nc">UpdateUserJob</span>
  <span class="k">def</span> <span class="nf">initialize</span><span class="p">(</span><span class="n">user_id</span><span class="p">,</span> <span class="nb">name</span><span class="p">)</span>
    <span class="vi">@user_id</span> <span class="o">=</span> <span class="n">user_id</span>
    <span class="vi">@name</span> <span class="o">=</span> <span class="nb">name</span>
  <span class="k">end</span>

  <span class="k">def</span> <span class="nf">call</span>
    <span class="n">user</span> <span class="o">=</span> <span class="no">User</span><span class="p">.</span><span class="nf">find</span><span class="p">(</span><span class="vi">@user_id</span><span class="p">)</span>
    <span class="n">user</span><span class="p">.</span><span class="nf">update!</span><span class="p">(</span><span class="ss">name: </span><span class="vi">@name</span><span class="p">)</span>
    <span class="c1"># .. additional business logic here</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>With that in place, we need to set up some sort of mechanism to have this code be run by Sidekiq in our background queue. We can introduce a simple class to wrap the sidekiq calls, allowing us to inject our custom job class with its arguments. To not conflict with the Sidekiq namespace, I have created the <code>Runners</code> module which will run this code for me <em>(side note: no matter what project I've worked on in the past, I always seem to use these "runner" style classes for various things)</em>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">module</span> <span class="nn">Runners</span>
  <span class="k">class</span> <span class="nc">Sidekiq</span>
    <span class="kp">include</span> <span class="o">::</span><span class="no">Sidekiq</span><span class="o">::</span><span class="no">Job</span>

    <span class="n">sidekiq_options</span><span class="p">(</span><span class="ss">retry: </span><span class="mi">3</span><span class="p">,</span> <span class="ss">backtrace: </span><span class="kp">true</span><span class="p">,</span> <span class="ss">queue: </span><span class="s2">"default"</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">perform</span><span class="p">(</span><span class="n">job_class_name</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
      <span class="no">Kernel</span><span class="p">.</span><span class="nf">const_get</span><span class="p">(</span><span class="n">job_class_name</span><span class="p">).</span><span class="nf">new</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">).</span><span class="nf">call</span>
    <span class="k">end</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">job_class</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">perform_async</span><span class="p">(</span><span class="n">job_class</span><span class="p">.</span><span class="nf">to_s</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
    <span class="k">end</span>

    <span class="k">def</span> <span class="nf">run_in</span><span class="p">(</span><span class="n">delay_in_seconds</span><span class="p">,</span> <span class="n">job_class</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">perform_in</span><span class="p">(</span><span class="n">delay_in_seconds</span><span class="p">,</span> <span class="n">job_class</span><span class="p">.</span><span class="nf">to_s</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
    <span class="k">end</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>And with that, we have everything we need to be able to throw plain old Ruby classes into the background for processing. If we wish to run our business logic inline, we simply run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">UpdateUserJob</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">).</span><span class="nf">call</span>
</code></pre>

</div>



<p>but if we want to delegate the processing to the background queue, we can send it off immediately or delay it<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="no">Runners</span><span class="o">::</span><span class="no">Sidekiq</span><span class="p">.</span><span class="nf">new</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="no">UpdateUserJob</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">)</span> <span class="c1"># perform the job as soon as possible</span>
<span class="no">Runners</span><span class="o">::</span><span class="no">Sidekiq</span><span class="p">.</span><span class="nf">new</span><span class="p">.</span><span class="nf">run_in</span><span class="p">(</span><span class="mi">5</span><span class="p">.</span><span class="nf">minutes</span><span class="p">,</span> <span class="no">UpdateUserJob</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="s2">"bob"</span><span class="p">)</span> <span class="c1"># delay the processing by 5 minutes</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0qg0e63pe5tvzedbljf3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0qg0e63pe5tvzedbljf3.png" alt="Sidekiq Web View" width="800" height="174"></a></p>

<h2>
  
  
  Bells &amp; Whistles
</h2>

<p>Now that we have our lovely Sidekiq runner class, we can embellish it with a couple final bells and whistles. We might want to add / override the sidekiq_options on demand instead of being stuck with the previous hard-coded options. Perhaps we prefer using keyword arguments in our method definitions for the added clarity and ability to insert the arguments in any order. And maybe, we would like the job names in the Sidekiq UI to be a little more descriptive as to the actual job they are running so that we don't have to go digging through the arguments list to find specific jobs.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight ruby"><code><span class="k">module</span> <span class="nn">Runners</span>
  <span class="k">class</span> <span class="nc">Sidekiq</span>
    <span class="kp">include</span> <span class="o">::</span><span class="no">Sidekiq</span><span class="o">::</span><span class="no">Job</span>

    <span class="no">DEFAULT_OPTIONS</span> <span class="o">=</span> <span class="p">{</span> <span class="ss">retry: </span><span class="mi">3</span><span class="p">,</span> <span class="ss">backtrace: </span><span class="kp">true</span><span class="p">,</span> <span class="ss">queue: </span><span class="s2">"default"</span> <span class="p">}</span>

    <span class="k">def</span> <span class="nf">perform</span><span class="p">(</span><span class="n">job_class_name</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
      <span class="k">if</span> <span class="n">args</span> <span class="k">in</span> <span class="p">[</span><span class="no">Hash</span><span class="p">]</span>
        <span class="c1"># This handles the case of keyword arguments</span>
        <span class="n">job_class_name</span><span class="p">.</span><span class="nf">constantize</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="o">**</span><span class="n">args</span><span class="p">.</span><span class="nf">first</span><span class="p">.</span><span class="nf">symbolize_keys!</span><span class="p">).</span><span class="nf">call</span>
      <span class="k">else</span>
        <span class="c1"># This handles the case of basic arguments</span>
        <span class="n">job_class_name</span><span class="p">.</span><span class="nf">constantize</span><span class="p">.</span><span class="nf">new</span><span class="p">(</span><span class="o">*</span><span class="n">args</span><span class="p">).</span><span class="nf">call</span>
      <span class="k">end</span>
    <span class="k">end</span>

    <span class="k">def</span> <span class="nf">initialize</span><span class="p">(</span><span class="n">options</span> <span class="o">=</span> <span class="p">{})</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">sidekiq_options</span><span class="p">(</span><span class="no">DEFAULT_OPTIONS</span><span class="p">.</span><span class="nf">merge</span><span class="p">(</span><span class="n">options</span><span class="p">))</span>
    <span class="k">end</span>

    <span class="k">def</span> <span class="nf">run</span><span class="p">(</span><span class="n">job_class</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
      <span class="n">update_display_class</span><span class="p">(</span><span class="n">job_class</span><span class="p">)</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">perform_async</span><span class="p">(</span><span class="n">job_class</span><span class="p">.</span><span class="nf">to_s</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
    <span class="k">end</span>

    <span class="k">def</span> <span class="nf">run_in</span><span class="p">(</span><span class="n">delay_in_seconds</span><span class="p">,</span> <span class="n">job_class</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
      <span class="n">update_display_class</span><span class="p">(</span><span class="n">job_class</span><span class="p">)</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">perform_in</span><span class="p">(</span><span class="n">delay_in_seconds</span><span class="p">,</span> <span class="n">job_class</span><span class="p">.</span><span class="nf">to_s</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">)</span>
    <span class="k">end</span>

    <span class="kp">private</span>

    <span class="k">def</span> <span class="nf">update_display_class</span><span class="p">(</span><span class="n">job_class</span><span class="p">)</span>
      <span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">sidekiq_options</span><span class="p">.</span><span class="nf">merge!</span><span class="p">(</span><span class="s2">"display_class"</span> <span class="o">=&gt;</span> <span class="s2">"</span><span class="si">#{</span><span class="n">job_class</span><span class="si">}</span><span class="s2"> [</span><span class="si">#{</span><span class="nb">self</span><span class="p">.</span><span class="nf">class</span><span class="p">.</span><span class="nf">name</span><span class="si">}</span><span class="s2">]"</span><span class="p">)</span>
    <span class="k">end</span>
  <span class="k">end</span>
<span class="k">end</span>
</code></pre>

</div>



<p>In essence, you can now define your background job classes as basic Ruby objects, execute them inline, or transfer them to Sidekiq for background processing. The main requirement is maintaining a consistent DSL with your job classes, such as implementing a mandatory <code>call</code> (or equivalent: <code>perform</code> or <code>execute</code>) method. (Kudos to my friend <a href="https://github.com/krzyczak" rel="noopener noreferrer">Kris</a> for the original idea.)</p>

