---
Title: Rust Ownership Explained: Why Some Values Move and Others Copy
Description: 
Author: itzlg | cyborg
Date: 2025-09-01T21:17:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>When we assign a value to a new variable in Rust, the value is usually moved. But here’s the catch ,this depends on the type of the value.</p>

<p>Rust plays a little trick when you assign values: some get copied, others get moved, depending on their type.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9kzjavxdw4i0ob1aui85.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9kzjavxdw4i0ob1aui85.jpg" alt="rust funny image" width="600" height="600"></a></p>

<h2>
  
  
  Primitive Types (Copy)
</h2>

<p>For primitive types like integers, booleans, and floats, the value is not moved. Instead, it is copied.</p>

<p>Why?</p>

<ul>
<li>They implement the Copy trait.</li>
<li>They are stored on the stack.</li>
<li>They have a fixed size known at compile time.</li>
<li>Copying them is extremely cheap and efficient.</li>
</ul>

<p>👉 Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let x = 5;
let y = x; // y gets a copy, x is still valid
println!("x = {}, y = {}", x, y);

</code></pre>

</div>



<p>Both x and y remain valid because i32 or u32 implements Copy.</p>

<h2>
  
  
  Complex Types (Move)
</h2>

<p>For types like String, Vec, and most custom structs, Rust does not copy the value by default, it moves it.</p>

<p>Why?</p>

<ul>
<li>They are stored in the heap.</li>
<li>Their size is not fixed at compile time.</li>
<li>Copying them would require duplicating potentially large amounts of data, which is expensive.</li>
</ul>

<p>👉 Example:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let s1 = String::from("hello");
let s2 = s1; // ownership moved
// println!("{}", s1); // ❌ Error: s1 is no longer valid

</code></pre>

</div>



<h2>
  
  
  How to Avoid Moving
</h2>

<p>Sometimes, you don’t want to lose access to the original variable . Rust gives you two options:</p>

<h3>
  
  
  Clone it:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);

</code></pre>

</div>



<p>This creates a deep copy of the value (including nested values if present) and allocates new memory in the heap which is safe but potentially expensive<br>
<strong>Note</strong>: (If it’s not a String, e.g., a struct, you can still call .clone() as long as the type implements the Clone trait.)</p>

<h3>
  
  
  Borrow (use references):
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>let s1 = String::from("hello");
let s2 = &amp;s1; // borrow instead of move
println!("s1 = {}, s2 = {}", s1, s2);

</code></pre>

</div>



<h2>
  
  
  Copy vs Clone
</h2>

<h3>
  
  
  Copy:
</h3>

<ul>
<li>Shallow copy</li>
<li>For small, fixed-size types stored on the stack</li>
<li>Very cheap</li>
</ul>

<h3>
  
  
  Clone:
</h3>

<ul>
<li>Deep copy</li>
<li>For complex types stored on the heap</li>
<li>Allocates new memory, can be expensive</li>
</ul>

<p>Rust’s ownership model might feel strict at first, but once you understand the difference between Move, Copy, and Clone, things start to click. The rules are there to help you write safe and efficient code without worrying about memory bugs.</p>

<p>If you found this helpful, please leave a comment or reaction 👏 it really motivates me to share more Rust content.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F77fxsdw29ltctrmk47ht.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F77fxsdw29ltctrmk47ht.gif" alt="leave comment" width="480" height="270"></a></p>

<p>You can also connect with me on X (Twitter): @<a href="https://x.com/MeLovegupta" rel="noopener noreferrer">melovegupta</a> 🚀</p>

