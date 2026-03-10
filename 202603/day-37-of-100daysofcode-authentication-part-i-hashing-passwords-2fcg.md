---
Title: Day 37 of #100DaysOfCode — Authentication Part I: Hashing Passwords
Description: 
Author: M Saad Ahmad
Date: 2026-03-10T22:02:52.000Z
Robots: noindex,nofollow
Template: index
---
<p>When building authentication systems, <strong>one of the biggest security mistakes that can be made is storing passwords in plain text</strong>.</p>

<p>If your database ever gets leaked, every user’s password becomes instantly visible.</p>

<p>That’s why modern applications <strong>never store real passwords</strong> — they store <strong>hashed passwords</strong>.</p>

<p>For Day 37, the goal was to understand:</p>

<ul>
<li>Why password hashing is important</li>
<li>What hashing actually means</li>
<li>What salting is</li>
<li>How to hash passwords using <strong>bcrypt</strong>
</li>
<li>How login verification works</li>
<li>How to implement this in a <strong>Node.js + MongoDB</strong> backend</li>
</ul>




<h2>
  
  
  Why Password Hashing Is Important
</h2>

<p>A <strong>bad implementation</strong> looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"email"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user@email.com"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"password"</span><span class="p">:</span><span class="w"> </span><span class="s2">"mypassword123"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>If the database is compromised, attackers can instantly see user passwords.</p>

<p>This becomes dangerous because:</p>

<ul>
<li>Many users reuse passwords</li>
<li>Attackers can access other accounts (email, banking, social media)</li>
</ul>




<h3>
  
  
  The Correct Way: Store Hashed Passwords
</h3>

<p>A <strong>secure implementation</strong> stores a hashed version of the password:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"email"</span><span class="p">:</span><span class="w"> </span><span class="s2">"user@email.com"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"password"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$2b$10$kjsdfhksjdfh..."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Now even if the database leaks, the attacker <strong>cannot directly read the password</strong>.</p>

<p>Instead of the real password, they only see a <strong>hash</strong>.</p>




<h2>
  
  
  What is Password Hashing?
</h2>

<p><strong>Hashing</strong> converts a password into a <strong>random-looking string</strong> using a mathematical algorithm.</p>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>password123
</code></pre>

</div>



<p>Becomes something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>$2b$10$Fv5Qn2H...
</code></pre>

</div>






<h3>
  
  
  Important Properties of Hashing
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Property</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td>Deterministic</td>
<td>The same input produces the same output (when using the same salt)</td>
</tr>
<tr>
<td>One-way</td>
<td>The hash cannot be reversed back into the original password</td>
</tr>
<tr>
<td>Fixed length</td>
<td>Hash outputs have a consistent structure</td>
</tr>
</tbody>
</table></div>

<p>⚠️ Important clarification:</p>

<p>Hashing is <strong>not encryption</strong>.</p>

<ul>
<li><strong>Encryption → reversible</strong></li>
<li><strong>Hashing → one-way</strong></li>
</ul>




<h2>
  
  
  🧂 What is Salting?
</h2>

<p>A <strong>salt</strong> is random data added to a password before hashing.</p>

<p>Example concept:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>password + randomSalt → hash
</code></pre>

</div>



<p>Why this matters:</p>

<p>Without salting, attackers can use <strong>rainbow tables</strong> (precomputed hash databases) to guess passwords quickly.</p>

<p>Salting ensures that:</p>

<ul>
<li>The <strong>same password produces different hashes</strong>
</li>
<li>Rainbow table attacks become ineffective</li>
</ul>

<p>Good news:</p>

<p>Libraries like <strong>bcrypt automatically generate and store the salt</strong>.</p>




<h2>
  
  
  Installing bcrypt (Node.js)
</h2>

<p>To hash passwords in a Node.js application, we commonly use <strong>bcrypt</strong>.</p>

<p>Install it using npm:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>npm <span class="nb">install </span>bcrypt
</code></pre>

</div>



<p>Then import it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">bcrypt</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">bcrypt</span><span class="dl">"</span><span class="p">);</span>
</code></pre>

</div>






<h3>
  
  
  Hashing Passwords During User Registration
</h3>

<p>When a user registers, we <strong>hash the password before storing it in the database</strong>.</p>

<p>Example inside a controller:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">bcrypt</span> <span class="o">=</span> <span class="nf">require</span><span class="p">(</span><span class="dl">"</span><span class="s2">bcrypt</span><span class="dl">"</span><span class="p">);</span>

<span class="kd">const</span> <span class="nx">hashedPassword</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">bcrypt</span><span class="p">.</span><span class="nf">hash</span><span class="p">(</span><span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span> <span class="mi">10</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  What does <code>10</code> mean?
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>10 = salt rounds
</code></pre>

</div>



<p>Salt rounds determine <strong>how many times the hashing algorithm runs</strong>.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Rounds</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td>Lower</td>
<td>Faster but less secure</td>
</tr>
<tr>
<td>Higher</td>
<td>More secure but slower</td>
</tr>
</tbody>
</table></div>

<p>Most applications use <strong>10–12 rounds</strong>.</p>




<h2>
  
  
  Saving the User in the Database
</h2>

<p>Example with MongoDB + Mongoose:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">user</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">User</span><span class="p">.</span><span class="nf">create</span><span class="p">({</span>
  <span class="na">email</span><span class="p">:</span> <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">email</span><span class="p">,</span>
  <span class="na">password</span><span class="p">:</span> <span class="nx">hashedPassword</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Now the database stores <strong>only the hashed password</strong>.</p>




<h3>
  
  
  Comparing Passwords During Login
</h3>

<p>When a user logs in:</p>

<ol>
<li>The user enters a password</li>
<li>The server compares it with the stored hash</li>
</ol>

<p>Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">isMatch</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">bcrypt</span><span class="p">.</span><span class="nf">compare</span><span class="p">(</span>
  <span class="nx">req</span><span class="p">.</span><span class="nx">body</span><span class="p">.</span><span class="nx">password</span><span class="p">,</span>
  <span class="nx">user</span><span class="p">.</span><span class="nx">password</span>
<span class="p">);</span>
</code></pre>

</div>



<p>If the result is:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>true
</code></pre>

</div>



<p>The password is correct, and login can proceed.</p>




<h3>
  
  
  Authentication Flow (High-Level)
</h3>

<p>Here is the simplified authentication flow:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>User registers
      ↓
Password gets hashed
      ↓
Stored in the database
      ↓
User logs in
      ↓
Entered password compared with stored hash
      ↓
If match → authentication successful
</code></pre>

</div>






<h2>
  
  
  Example Mongoose User Schema
</h2>

<p>Your <strong>User model</strong> might look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">userSchema</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">mongoose</span><span class="p">.</span><span class="nc">Schema</span><span class="p">({</span>
  <span class="na">email</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span>
    <span class="na">required</span><span class="p">:</span> <span class="kc">true</span>
  <span class="p">},</span>
  <span class="na">password</span><span class="p">:</span> <span class="p">{</span>
    <span class="na">type</span><span class="p">:</span> <span class="nb">String</span><span class="p">,</span>
    <span class="na">required</span><span class="p">:</span> <span class="kc">true</span>
  <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>Important rule:</p>

<p><strong>Passwords must always be stored in hashed form.</strong></p>

<p>Never store plain-text passwords in a production application.</p>




<h2>
  
  
  Key Takeaways
</h2>

<ul>
<li>Never store passwords as plain text</li>
<li>Always hash passwords before saving them</li>
<li>
<strong>bcrypt</strong> is one of the most widely used password hashing libraries</li>
<li>Salting protects against rainbow table attacks</li>
<li>Use <code>bcrypt.compare()</code> to verify login passwords</li>
</ul>

<p>Thank you for reading. Feel free to share your thoughts!</p>

