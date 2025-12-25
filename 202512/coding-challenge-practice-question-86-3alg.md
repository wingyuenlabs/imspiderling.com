---
Title: Coding Challenge Practice - Question 86
Description: 
Author: Bukunmi Odugbesan
Date: 2025-12-25T21:41:24.000Z
Robots: noindex,nofollow
Template: index
---
<p>The task is to implement a Queue by using Stack.</p>

<p>The boilerplate code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Queue {
  enqueue(element) { 
    // add new element to the rare
  }
  peek() { 
    // get the head element
  }
  size() { 
    // return count of element
  }
  dequeue() {
    // remove the head element
  }
}

</code></pre>

</div>



<p>A Queue follows the First In, First Out rule.</p>

<p>Create an array that stores all the queue elements, and a pointer that points to the front of the queue.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>this.items = [];
this.head = 0;
</code></pre>

</div>



<p>To add an element to the end of the queue<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>enqueue(element) {
  this.items.push(element);
}
</code></pre>

</div>



<p>To get the front element<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>peek() {
  return this.size() === 0 ? undefined : this.items[this.head];
}
</code></pre>

</div>



<p>To know how many valid elements are in the queue<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>size() {
  return this.items.length - this.head;
}
</code></pre>

</div>



<p>To remove the front element<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>dequeue() {
  if (this.size() === 0) return undefined;

  const value = this.items[this.head];
  this.head++;

  return value;
}
</code></pre>

</div>



<p>Dequeue stores the front value, moves the pointer forward and returns the removed value. If the queue is empty, undefined is returned.</p>

<p>The final code<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Queue {
  constructor() {
    this.items = [];
    this.head = 0;
  }
  enqueue(element) { 
    // add new element to the rare
    this.items.push(element);
  }
  peek() { 
    // get the head element
    return this.size() === 0 ? undefined : this.items[this.head];
  }
  size() { 
    // return count of element
    return this.items.length - this.head;
  }
  dequeue() {
    // remove the head element
    if(this.size() === 0) return undefined;

    const value = this.items[this.head];
    this.head++

    if(this.head &gt;  50 &amp;&amp; this.head * 2 &gt;= this.items.length) {
      this.items = this.items.slice(this.head);
      this.head = 0;
    }
    return value;
  }
}

</code></pre>

</div>



<p>That's all folks!</p>

