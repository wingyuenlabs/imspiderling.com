---
Title: 151. Reverse Words in a String | LeetCode | Top Interview 150 | Coding Questions
Description: 
Author: Debesh P.
Date: 2025-12-26T22:03:15.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<h3>
  
  
  Problem Link
</h3>

<p><a href="https://leetcode.com/problems/reverse-words-in-a-string/" rel="noopener noreferrer">https://leetcode.com/problems/reverse-words-in-a-string/</a></p>
</blockquote>




<blockquote>
<h3>
  
  
  Detailed Step-by-Step Explanation
</h3>

<p><a href="https://leetcode.com/problems/reverse-words-in-a-string/solutions/7441810/most-optimal-solution-best-for-interview-aub2" rel="noopener noreferrer">https://leetcode.com/problems/reverse-words-in-a-string/solutions/7441810/most-optimal-solution-best-for-interview-aub2</a></p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwsgjhqib3v38cpmeqoam.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwsgjhqib3v38cpmeqoam.png" alt="leetcode 151" width="800" height="450"></a></p>




<h3>
  
  
  Solution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Solution {
    public String reverseWords(String s) {

        String[] words = s.trim().split("\\s+");
        StringBuilder result = new StringBuilder();

        for (int i = words.length - 1; i &gt;= 0; i--) {
            result.append(words[i]);
            if (i != 0) result.append(" ");
        }

        return result.toString();
    }
}
</code></pre>

</div>



