---
Title: Coding Challenge Practice - Question 44
Description: 
Author: Bukunmi Odugbesan
Date: 2025-11-03T21:46:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to implement binary search given a sorted array of ascending numbers, which might have duplicates, to return the element right after the last appearance of a target number.</p>

<p>The boilerplate code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function elementAfter(arr, target){
  // your code here
}
</code></pre>

</div>



<p>The left is the first index of the array, the right is the last index of the array.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let left = 0;
let right = arr.length - 1;
let lastIndex = -1

</code></pre>

</div>



<p>Find the middle of the array<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let mid = Math.floor((left + right) / 2)
</code></pre>

</div>



<p>If the target is found <code>arr[mid] === target</code>, the value is stored in result, and the search is continued to the right to see if there's a later duplicate.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if(arr[mid] === target) {
lastIndex = mid;
left= mid + 1
}
</code></pre>

</div>



<p>If the middle value is smaller than the target, the search is continued to the right<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>else if (arr[mid] &lt; target) {
left = mid + 1
}
</code></pre>

</div>



<p>If it is larger, the search is continued to the left.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>else {
right = mid - 1
}
</code></pre>

</div>



<p>If not found, return undefined<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if (lastIndex === target) return undefined;
</code></pre>

</div>



<p>If the target is the last element, then there's nothing after it<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if(lastIndex === arr.length - 1) return undefined
</code></pre>

</div>



<p>The result returned is the element just after the last element<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>return arr[lastIndex + 1]
</code></pre>

</div>



<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function elementAfter(arr, target){
  // your code here
  let left = 0;
  let right = arr.length - 1;
  let lastIndex = -1;

  while(left &lt;= right) {
    let mid = Math.floor((left + right) / 2);

    if(arr[mid] === target) {
      lastIndex = mid;
      left = mid + 1;
    } else if(arr[mid] &lt; target) {
      left = mid + 1
    } else {
      right = mid - 1;
    }
  }
  if(lastIndex === -1) return undefined;
  if(lastIndex === 0) return undefined;

  return arr[lastIndex + 1]
}

</code></pre>

</div>



<p>That's all folks!</p>

