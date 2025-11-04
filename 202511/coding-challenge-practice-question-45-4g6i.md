---
Title: Coding Challenge Practice - Question 45
Description: 
Author: Bukunmi Odugbesan
Date: 2025-11-04T21:28:57.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to implement myNew function to return an object just like the new constructor.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const myNew = (constructor, ...args) =&gt; {
  // your code here
}
</code></pre>

</div>



<p>How does the new operator work? It creates a new empty object, sets the prototype of that object to the constructor's prototype, calls the constructor function, and returns the object the constructor returns or the newly created one. </p>

<p>If the constructor is not a function, throw an error<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if (typeof constructor !== 'function') {
    throw new TypeError('myNew: first argument must be a constructor function');
  }
</code></pre>

</div>



<p>Create the new object to be returned if the constructor does not return one<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const obj = Object.create(constructor.prototype);
</code></pre>

</div>



<p>Call the constructor with "this" set to the new object<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const result = constructor.apply(obj, args);
</code></pre>

</div>



<p>If the constructor returns an object (or function), return it. Otherwise, return the new function<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>return (result !== null &amp;&amp; (typeof result === 'object' || typeof result === 'function')) 
    ? result 
    : obj;
</code></pre>

</div>



<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const myNew = (constructor, ...args) =&gt; {
  // your code here
  if(typeof constructor !== 'function') {
    throw new TypeError('myNew: first argumant must be a function')
  }
  const obj = Object.create(constructor.prototype);

  const result = constructor.apply(obj, args)

  return (result !== null &amp;&amp; (typeof result === 'object' || typeof result === 'function'))
  ? result : obj;
}
</code></pre>

</div>



<p>That's all folks!</p>

