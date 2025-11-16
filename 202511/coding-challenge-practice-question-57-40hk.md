---
Title: Coding Challenge Practice - Question 57
Description: 
Author: Bukunmi Odugbesan
Date: 2025-11-16T21:38:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to implement Object.create()</p>

<p>The boilerplate code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function myObjectCreate(proto) {
  // your code here
}
</code></pre>

</div>



<p>If the prototype is null or not an object, throw an error<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if(proto === null || typeof !== "object") {
 throw new TypeError("Prototype must be an object or null")
}
</code></pre>

</div>



<p>An empty constructor function is declared, because every object in Javascript is created using a constructor.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function F() {}
</code></pre>

</div>



<p>The prototype of the empty constructor is set to the one provided<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>F.prototype = proto
</code></pre>

</div>



<p>Then, a new instance is returned<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>return new F()
</code></pre>

</div>



<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function myObjectCreate(proto) {
  // your code here
  if(proto === null || typeof proto !== "object") {
    throw new TypeError("Prototype must be object or null")
  }

  function F() {}
  F.prototype = proto;
  return new F;
}
</code></pre>

</div>



<p>That's all folks!</p>

