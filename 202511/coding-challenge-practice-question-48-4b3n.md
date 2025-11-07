---
Title: Coding Challenge Practice - Question 48
Description: 
Author: Bukunmi Odugbesan
Date: 2025-11-07T21:49:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to implement a string subtraction function with all non-negative integers in string.</p>

<p>The boilerplate code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function subtract(num1, num2) {
  // your code here
}
</code></pre>

</div>



<p>First, remove leading zeros from both input strings. If the string is empty, replace with zero.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code> num1 = num1.replace(/^0+/, '') || '0';
 num2 = num2.replace(/^0+/, '') || '0';
</code></pre>

</div>



<p>If num1 &lt; num2, the function returns 0 because a negative number can't be the result<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if (compare(num1, num2) &lt; 0) return "0";
</code></pre>

</div>



<p>Make num2 the same length as num1 by adding leading zeros to be able to subtract digit by digit from right to left<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>num2 = num2.padStart(num1.length, '0');
</code></pre>

</div>



<p>Subtract from right to left, keeping a borrow (0 or 1). The borrow is 0 when the value being subtracted from is greater than the value being subtracted, otherwise it is 1.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let res = '';
let borrow = 0;

 for (let i = num1.length - 1; i &gt;= 0; i--) {
    let digit1 = +num1[i];
    let digit2 = +num2[i] + borrow;

    if (digit1 &lt; digit2) {
      digit1 += 10;
      borrow = 1;
    } else {
      borrow = 0;
    }

    res = (digit1 - digit2) + res;
  }

  return res.replace(/^0+/, '') || '0';
</code></pre>

</div>



<p>digit1 is the value of num1[i], digit2 is the value of num2[i] + borrow. If digit1 &lt; digit2, add 10 to digit1 and set borrow to 1, otherwise set borrow to 0. Remove the leading zeros from the result.</p>

<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>function subtract(num1, num2) {
  // your code here
   num1 = num1.replace(/^0+/, '') || '0';
  num2 = num2.replace(/^0+/, '') || '0';

  if (compare(num1, num2) &lt; 0) return "0";

  let res = '';
  let borrow = 0;

  num2 = num2.padStart(num1.length, '0');

  for (let i = num1.length - 1; i &gt;= 0; i--) {
    let digit1 = +num1[i];
    let digit2 = +num2[i] + borrow;

    if (digit1 &lt; digit2) {
      digit1 += 10;
      borrow = 1;
    } else {
      borrow = 0;
    }

    res = (digit1 - digit2) + res;
  }

  return res.replace(/^0+/, '') || '0';
}

function compare(a, b) {
  if (a.length !== b.length) return a.length - b.length;
  return a.localeCompare(b);
}
</code></pre>

</div>



<p>That's all folks!</p>

