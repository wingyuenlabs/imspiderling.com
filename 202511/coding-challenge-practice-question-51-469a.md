---
Title: Coding Challenge Practice - Question 51
Description: 
Author: Bukunmi Odugbesan
Date: 2025-11-10T21:46:21.000Z
Robots: noindex,nofollow
Template: index
---
<p>Implement a function to reverse a linked list. The boilerplate code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const reverseLinkedList = (list) =&gt; {
    // your code
}
</code></pre>

</div>



<p>Set the new head to null, and list is the current node being reversed.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let prev = null;
let curr = list
</code></pre>

</div>



<p>For each node, save the next node temporarily, reverse the link and move <code>curr</code> and <code>prev</code> forward.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>while(curr) {
const next = curr.next;
curr.next = prev;
prev = curr;
curr = next;
</code></pre>

</div>



<p>When the loop ends, <code>prev</code> points to the new head of the reversed list. The final code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const reverseLinkedList = (list) =&gt; {
    // your code
    let prev = null;
    let curr = list;

    while(curr) {
        const next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next
    }
    return prev;
}
</code></pre>

</div>



<p>That's all folks!</p>

