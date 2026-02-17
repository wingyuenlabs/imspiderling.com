---
Title: Coding Challenge Practice - Question 120
Description: 
Author: Bukunmi Odugbesan
Date: 2026-02-17T21:35:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to write a function to remove duplicate characters in a string<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function smallestUniqueSubstr(str) {
  // your code here
}

</code></pre>

</div>



<p>Count the remaining occurrences of each character. This stores how many times each character appears<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const remaining = {};
for(const ch of str) {
remaining[ch] = (remaining[ch] || 0) + 1;
}
</code></pre>

</div>



<p>Create data structures to build the final result and ensure uniqueness<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const stack = []
const inStack = new Set()
</code></pre>

</div>



<p>Iterate through the string. Decrease the count as the character is being processed<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>for(const ch of str) {
remaining[ch]--
</code></pre>

</div>



<p>Skip the character if it has already been added to the result<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if(inStack.has(ch)) return;
</code></pre>

</div>



<p>Remove characters from the stack if the stack is not empty, the current character is smaller top character, and the top character is greater than zero<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>while(
  stack.length &gt; 0 &amp;&amp;
  stack[stack.length - 1] &gt; ch &amp;&amp;
  remaining[stack[stack.length -1]] &gt; 0
) {
  inStack.delete(stack.pop());
}
</code></pre>

</div>



<p>Add the current character to the result if all conditions are not met<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>stack.push(ch);
inStack.add(ch);
</code></pre>

</div>



<p>Return the final result<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>return stack.join('')
</code></pre>

</div>



<p>The final solution<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function smallestUniqueSubstr(str) {
  // your code here
  const remaining = {};
  for(const ch of str) {
    remaining[ch] = (remaining[ch] || 0) + 1
  }

  const inStack = new Set();
  const stack = [];

  for(const ch of str) {
    remaining[ch]--;

    if(inStack.has(ch)) continue;

    while(stack.length &gt; 0 &amp;&amp; stack[stack.length - 1] &gt; ch &amp;&amp; 
    remaining[stack[stack.length -1]] &gt; 0) {
      inStack.delete(stack.pop());
    }
    stack.push(ch);
    inStack.add(ch);
  }
  return stack.join('')
}
</code></pre>

</div>



<p>That's all folks!</p>

