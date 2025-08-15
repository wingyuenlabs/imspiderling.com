---
Title: ওয়েব ফর্মের জাদু – HTML Input Types বাংলায়
Description: 
Author: Tamima Akther Lima
Date: 2025-08-15T21:00:15.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  ওয়েব ফর্মের জাদু – HTML Input Types বাংলায়
</h2>

<p><strong><em>Introduction:</em></strong><br>
ওয়েবসাইটের ফর্ম মানেই  ট্যাগের ব্যবহার। নাম, পাসওয়ার্ড, ইমেইল, তারিখ, এমনকি রঙ বাছাই করতেও  সাহায্য করে। HTML-এর বিভিন্ন input types জানলে ফর্ম হবে আরও ইন্টারেক্টিভ, ইউজার-ফ্রেন্ডলি, আর ডেটা ভ্যালিডেশন হবে সহজ।<br>
আজ আমরা একে একে সব ইনপুট টাইপ বাংলায় বুঝব, সাথে থাকবে কোড উদাহরণ।</p>

<p>HTML-এ  ট্যাগের অনেক রকম type আছে। কিছু জনপ্রিয় টাইপের নাম:</p>

<ul>
<li>text</li>
<li>password</li>
<li>email</li>
<li>number</li>
<li>date</li>
<li>time</li>
<li>datetime-local</li>
<li>month</li>
<li>week</li>
<li>color</li>
<li>file</li>
<li>checkbox</li>
<li>radio</li>
<li>range</li>
<li>search</li>
<li>tel</li>
<li>url</li>
<li>hidden</li>
<li>submit</li>
<li>reset</li>
<li>button</li>
</ul>

<p>1️⃣ text – সাধারণ লেখার বক্স</p>

<p>টেক্সট ইনপুট সবচেয়ে বেশি ব্যবহৃত হয়। এখানে ইউজার যেকোনো লেখা টাইপ করতে পারে।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="text" placeholder="Enter Your Name:"&gt;
</code></pre>

</div>



<p>💡 টিপস: placeholder দিলে ফিল্ড বুঝতে সুবিধা হয়।</p>

<p>2️⃣ password – গোপন লেখার বক্স</p>

<p>পাসওয়ার্ড টাইপ করলে টেক্সট দেখা যায় না, তারকা (*) বা বিন্দু আকারে দেখায়।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="password" placeholder="Enter password:"&gt;

</code></pre>

</div>



<p>3️⃣ email – ইমেইল চেকার</p>

<p>শুধু সঠিক ইমেইল ফরম্যাট হলে ফর্ম সাবমিট হবে।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="email" placeholder="example@email.com"&gt;

</code></pre>

</div>



<p>4️⃣ number – শুধু সংখ্যা</p>

<p>শুধুমাত্র সংখ্যা টাইপ করা যাবে, চাইলে min ও max দিয়ে সীমা ঠিক করা যায়।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="number" min="1" max="100"&gt;

</code></pre>

</div>



<p>5️⃣ date – ক্যালেন্ডার থেকে তারিখ</p>

<p>তারিখ সিলেক্ট করার জন্য একটি সুন্দর ক্যালেন্ডার ওপেন হবে।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="date"&gt;

</code></pre>

</div>



<p>6️⃣ checkbox – একাধিক অপশন সিলেক্ট</p>

<p>একাধিক উত্তর একসাথে বেছে নিতে চাইলে checkbox ব্যবহার করা হয়।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="checkbox" name="gender" value="male"&gt; Male
&lt;input type="checkbox" name="gender" value="female"&gt; Female

</code></pre>

</div>



<p>7️⃣ radio – একটাই বেছে নিন</p>

<p>একই গ্রুপের মধ্যে শুধু একটি অপশন সিলেক্ট করা যায়।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="radio" name="fruits" value="mango"&gt; Mango
&lt;input type="radio" name="fruits" value="apple"&gt; Apple

</code></pre>

</div>



<p>8️⃣ submit – ফর্ম সাবমিটের বাটন</p>

<p>ডেটা পাঠানোর জন্য ব্যবহৃত হয়।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="submit" value="submit"&gt;

</code></pre>

</div>



<p>9️⃣ file – ফাইল আপলোড</p>

<p>যেকোনো ফাইল সিলেক্ট করে আপলোড করা যায়।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;input type="file"&gt;

</code></pre>

</div>



<p>🔟 search – সার্চ ফিল্ড</p>

<p>এই টাইপ সাধারণ টেক্সট ইনপুট এর মত, কিন্তু ব্রাউজারে সার্চ ফাংশনের জন্য অপ্টিমাইজ করা থাকে।<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;label for="site-search"&gt;Search Website:&lt;/label&gt;
&lt;input type="search" id="site-search" name="site-search" placeholder="Search"&gt;

</code></pre>

</div>



<p>🎯 ডেমো ফর্ম – সব ইনপুট একসাথে:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;form&gt;
  &lt;label for="name"&gt;Name:&lt;/label&gt;
  &lt;input type="text" id="name" name="name" placeholder="Enter your name"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="password"&gt;Password:&lt;/label&gt;
  &lt;input type="password" id="password" name="password" placeholder="Enter your password"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="email"&gt;Email:&lt;/label&gt;
  &lt;input type="email" id="email" name="email" placeholder="Enter your email"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="age"&gt;Age:&lt;/label&gt;
  &lt;input type="number" id="age" name="age" min="1" max="100"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="dob"&gt;Date of Birth:&lt;/label&gt;
  &lt;input type="date" id="dob" name="dob"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="time"&gt;Preferred Time:&lt;/label&gt;
  &lt;input type="time" id="time" name="time"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="datetime"&gt;Appointment:&lt;/label&gt;
  &lt;input type="datetime-local" id="datetime" name="datetime"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="color"&gt;Favorite Color:&lt;/label&gt;
  &lt;input type="color" id="color" name="color"&gt;&lt;br&gt;&lt;br&gt;

  &lt;label for="file"&gt;Upload File:&lt;/label&gt;
  &lt;input type="file" id="file" name="file"&gt;&lt;br&gt;&lt;br&gt;

  &lt;input type="submit" value="Submit"&gt;
&lt;/form&gt;

</code></pre>

</div>



<p>HTML Input Types হল ওয়েব ফর্মের প্রাণ। প্রতিটি টাইপের নিজস্ব কাজ আছে, আর সঠিকভাবে ব্যবহার করলে ফর্ম হবে সুন্দর, সহজ, আর ইউজার-ফ্রেন্ডলি।</p>

<p>এখন আপনার পালা – আপনি কোন ইনপুট টাইপ সবচেয়ে বেশি ব্যবহার করেন? কমেন্টে জানান।</p>

