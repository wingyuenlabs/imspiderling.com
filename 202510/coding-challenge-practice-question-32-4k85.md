---
Title: Coding Challenge Practice - Question 32
Description: 
Author: Bukunmi Odugbesan
Date: 2025-10-21T21:26:14.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to implement a Binary Search to find the index of a given array of unique and ascending integers.</p>

<p>The boilerplate code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function binarySearch(arr, target){
  // your code here
}
</code></pre>

</div>



<p>A binary search algorithm is used to find the position of a target value within a sorted array.</p>

<p>To find the index, start by initializing the boundaries<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let left = 0
let right = arr.length - 1
</code></pre>

</div>



<p>Left is 0, because arrays start from index 0. Right is the length of the array minus 1, because the index of the last item will be one short of the array's length.</p>

<p>While there's a valid range from left to right, compute the middle index and compare it to the target value<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>while (left &lt;= right) {
const mid = ((left + right) / 2);
const midVal = arr[mid]

if (midVal === target) {
      return mid; // found it
    } else if (midVal &lt; target) {
      left = mid + 1; // search right half
    } else {
      right = mid - 1; // search left half
    }
}
</code></pre>

</div>



<p>If the middle value is less than the target, ignore the left half of the array and search the right. If it is greater than the target, ignore the right side and search the left. </p>

<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function binarySearch(arr, target){
  // your code here
  let left = 0;
  let right = arr.length - 1;

  while(left &lt;= right) {
    const mid = Math.floor((left + right) / 2);
    const midVal = arr[mid];

    if(midVal === target) {
      return mid
    } else if(midVal &lt; target) {
      left = mid + 1
    } else {
      right = mid - 1
    }
  }
  return -1;
}
</code></pre>

</div>



<p>That's all folks!</p>

