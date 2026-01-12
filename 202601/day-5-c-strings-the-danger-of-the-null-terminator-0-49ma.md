---
Title: Day 5: C Strings: The Danger of the Null Terminator (\0)
Description: 
Author: Ujjawal Chaudhary
Date: 2026-01-12T21:18:02.000Z
Robots: noindex,nofollow
Template: index
---
<p>In Python or Java, strings are objects. You don't worry about how they end.</p>

<p>In C, a string is just a lie. It's actually just a character array.</p>

<p>So how does the computer know where the string stops? It looks for a secret hidden character called the <strong>Null Terminator</strong> (<code>\0</code>).</p>

<ul>
<li>If you forget space for this <code>\0</code>, functions like <code>printf</code> won't stop reading.</li>
<li>They will keep printing memory (garbage data, or even sensitive data) until they accidentally hit a 0 somewhere in your RAM.</li>
</ul>

<p>This is the root cause of the famous <strong>Buffer Overflow</strong> vulnerability.</p>

<h3>
  
  
  The Code
</h3>

<p>Here is the code demonstrating "Safe" vs "Unsafe" strings.</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>

// Day 5: The most dangerous character in C

#include &lt;stdio.h&gt;

int main() {
    // 1. Safe String (Compiler adds \0 automatically)
    // Size is 6 bytes ('H', 'e', 'l', 'l', 'o', '\0')
    char safe[] = "Hello"; 

    // 2. Unsafe String (Manually built, NO null terminator)
    // Size is 2 bytes. 
    char unsafe[2] = {'H', 'i'}; 

    printf("Safe String: %s\n", safe);

    // DANGER: printf keeps reading memory until it finds a 0.
    // It will print "Hi" followed by random garbage from your RAM!
    printf("Unsafe String: %s\n", unsafe);

    return 0;
}

📂 View the source code on GitHub: https://github.com/Ujjawal0711/30-Days
</code></pre>

</div>

