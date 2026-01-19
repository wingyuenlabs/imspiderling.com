---
Title: DNS Explained in very simple words(For Everyone)
Description: 
Author: Debashis Das
Date: 2026-01-19T21:24:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you heard about DNS or Domain Name System and want to know the basic overview of how exactly DNS works, then dont worry, i will explain it to you in a very simple term, not with handy word.</p>

<p>So Now Let's start with the simple question.</p>

<h3>
  
  
  How does your browser know where a website lives?
</h3>

<p>When you type:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
👉 google.com
</code></pre>

</div>



<p>Your browser is actually confused 🤔🙄😵<br>
Because computers <strong>do not understand names</strong> like "google", "facebook", "instagram" etc.</p>

<p>Computers only understnad <strong>numbers</strong>.</p>

<p>So someone has to tell the browser:</p>

<blockquote>
<p>"Hey, google.com live at this number"</p>
</blockquote>

<p>That "someone" is <strong>DNS</strong>.</p>


<h3>
  
  
  What DNS Really Is (No Tech Meaning):
</h3>

<p>DNS is like the phonebook of the internet.</p>

<p>In your phone:</p>

<ul>
<li>You save names like (Mom, Dad, Brother, Friends, Ofiice)</li>
<li>But calling happens using a phone number</li>
</ul>

<p>On the internet:</p>

<ul>
<li>You type a website name</li>
<li>DNS finds the number</li>
<li>Browser goes there for lookup</li>
</ul>

<p>Without DNS, the internet would look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
Type 142.250.182.14 to open 'Google'
</code></pre>

</div>



<p>And nobody wants that to remember the whole number.<br>
What if we have some tool that remember those large number for us,it's kind of a middleware.And that's the reason why DNS plays a important role for us to find website actual location.</p>


<h3>
  
  
  Why DNS Record Exist
</h3>

<p>DNS record are just small pieces of information that answer different questions.</p>

<p>Like:</p>

<ul>
<li>Where is this Websites?</li>
<li>Who is in charge of this domain?</li>
<li>Where should email go?</li>
<li>Is this domain trusted?</li>
</ul>

<p>Instead of putting everything in one place, DNS uses <strong>different record</strong>, each with one clear job. </p>


<h3>
  
  
  NS Record-Who is Responsible for This Website?
</h3>

<p>Imagine an apartment building 🏢<br>
You don't ask any random people about a flat-you go to the main office.</p>
<h4>
  
  
  NS record is that main office.
</h4>

<p>It tells the internet:</p>

<blockquote>
<p>"Ask this company for all information about this domain."</p>
</blockquote>

<p>So NS record answers:</p>

<ul>
<li>Who manages this website?</li>
<li>Who has the authority?</li>
</ul>

<p>Without NS:</p>

<ul>
<li>Internet would not know where to ask questions and to whome.</li>
</ul>


<h3>
  
  
  A Record- Where Is the Website?
</h3>

<p>This is the most important record.<br>
A Record tells the exact address of the website.<br>
Think of it like:</p>

<ul>
<li>House name -&gt; House number</li>
</ul>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
example.com -&gt; 93.128.332.34
</code></pre>

</div>



<p>Your browser:</p>

<ul>
<li>Find this number</li>
<li>Goes direct to the server with the help of that number</li>
<li>And finally load the website for us in the browser</li>
</ul>

<p>Simple and direct.</p>




<h3>
  
  
  AAAA Record - Same Job, Newer System
</h3>

<p>AAAA record does the <strong>same thing as A record</strong>.<br>
The only differece:</p>

<ul>
<li>A record -&gt; Old type address</li>
<li>AAAA record -&gt; new type address</li>
</ul>

<p>You don't need to worry much about it.<br>
Just remember:</p>

<blockquote>
<p>A and AAAA both tell where the website lives.</p>
</blockquote>


<h3>
  
  
  CNAME Record - Another Name for the Same Place
</h3>

<p>Something a place has two names.</p>

<p>Example:</p>

<ul>
<li>"Main street shop"</li>
<li>"MS Shop"</li>
</ul>

<p>Both go to the CNAME does.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
www.example.com -&gt; example.com
</code></pre>

</div>



<p>This means:</p>

<ul>
<li>
<code>www.example.com</code> is just another name</li>
<li>Real website is <code>example.com</code> </li>
</ul>

<p>Easy difference to remember:</p>

<ul>
<li>A Record -&gt; name to number <code>[google.com -&gt; 123.652.88.90]</code>
</li>
<li>CNAME -&gt; name to name <code>[it's like giving someone to a nickname]</code>
</li>
</ul>




<h3>
  
  
  MX Record - How Emails Know Where to Go
</h3>

<p>Now let's talk about email📧 <br>
When someone sends:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>hello.example.com
</code></pre>

</div>



<p>The internet asks:</p>

<blockquote>
<p>"Where should i deliver this email?"</p>
</blockquote>

<p>The answer comes from the MX record.The MX is like a post office address.<br>
It btells:</p>

<ul>
<li>Which mail server recieves emails</li>
<li>For that domain</li>
</ul>

<p>No MX record = No email delivery.</p>




<h3>
  
  
  TXT Record - Extra Notes and Proof
</h3>

<p>TXT records are like <strong>stricky notes</strong> attached to a domain.<br>
They arev used to:</p>

<ul>
<li>Prove ownership </li>
<li>Keeps email safe</li>
<li>Verify services</li>
</ul>

<p>Example:</p>

<ul>
<li>Google asks: "Is this your domain?"</li>
<li>You add a TXT record</li>
<li>Google checks it</li>
<li>Done ✅</li>
</ul>

<h3>
  
  
  TXT records don't affect websites directly -- they help in the background.
</h3>

<p>How Everything Works Together (One simple Story)<br>
Let's say someone opens your website.</p>

<p><strong>Step-by-step:</strong></p>

<ol>
<li><p>Browser looks for NS<br>
   -&gt; Who controls this domain?</p></li>
<li><p>DNS server gives A / AAAA <br>
   -&gt; Here is the website address</p></li>
<li><p>If <code>www</code> is used, CNAME helps<br>
   -&gt; Redirects to main name</p></li>
<li><p>If someone sends email, MX helps<br>
   -&gt; delivers email properly</p></li>
<li><p>TXT keeps things verified and safe</p></li>
</ol>

<p>All Records work together like a team.</p>




<h3>
  
  
  One Complete DNS Setup (Easy View)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>NS    -&gt; Who manage the domain
A     -&gt; WEbsite address
AAAA  -&gt; Newer website address
CNAME -&gt; Another name
MX    -&gt; Email delivery
TXT   -&gt; Proof and Security
</code></pre>

</div>






<h3>
  
  
  Final Words (Very important)
</h3>

<p>DNS sounds scary only because of it's name.</p>

<p>But in reality, its just:</p>

<blockquote>
<p>A system that helps names find the right place.</p>
</blockquote>

<p>If you understand <code>Phonebook</code>, <code>House address</code> and <code>Post office</code>, then you already understand DNS.</p>

