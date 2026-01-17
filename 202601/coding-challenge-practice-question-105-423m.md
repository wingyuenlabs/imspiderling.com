---
Title: Coding Challenge Practice - Question 105
Description: 
Author: Bukunmi Odugbesan
Date: 2026-01-17T21:41:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to create a function that removes duplicates from an array.</p>

<p>The boilerplate code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function deduplicate(arr) {
  // your code here
}
</code></pre>

</div>



<p>The objective includes removing the duplicates without creating a new array. Track items that have been seen as the array is being sorted<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const seen = new Set();
</code></pre>

</div>



<p>Using the two pointers approach, <code>readIndex</code> will read through the array while <code>writeIndex</code> writes unique elements.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let writeIndex = 0;
</code></pre>

</div>



<p>When an unseen element is encountered, it is added to the Set and copied to the <code>writeIndex</code> position.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>for (let readIndex = 0; readIndex &lt; arr.length; readIndex++) {
    const item = arr[readIndex];

    if (!seen.has(item)) {
      // Mark as seen
      seen.add(item);

      if (writeIndex !== readIndex) {
        arr[writeIndex] = item;
      }

      writeIndex++;
    }
  }
</code></pre>

</div>



<p>After processing, truncate the array to the new length.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>arr.length = writeIndex;
</code></pre>

</div>



<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function deduplicate(arr) {
  // your code here
  const seen = new Set();
  let writeIndex = 0;

  for(let readIndex = 0; readIndex &lt; arr.length; readIndex++) {
    const item = arr[readIndex];

    if(!seen.has(item)) {
      seen.add(item);

      if(writeIndex !== readIndex) {
        arr[writeIndex] = item;
      }
      writeIndex++;
    }
  }
  arr.length = writeIndex;

  return arr;
}
</code></pre>

</div>



<p>That's all folks!</p>

