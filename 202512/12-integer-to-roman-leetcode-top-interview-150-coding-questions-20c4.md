---
Title: 12. Integer to Roman | LeetCode | Top Interview 150 | Coding Questions
Description: 
Author: Debesh P.
Date: 2025-12-25T21:53:49.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<h3>
  
  
  Problem Link
</h3>

<p><a href="https://leetcode.com/problems/integer-to-roman/" rel="noopener noreferrer">https://leetcode.com/problems/integer-to-roman/</a></p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0dvhrr8defkjps7k2om3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0dvhrr8defkjps7k2om3.png" alt="leetcode 12" width="800" height="450"></a></p>




<h3>
  
  
  Solution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Solution {
    public String intToRoman(int num) {

        int[] values = {
            1000, 900, 500, 400,
            100, 90, 50, 40,
            10, 9, 5, 4,
            1
        };

        String[] symbols = {
            "M", "CM", "D", "CD",
            "C", "XC", "L", "XL",
            "X", "IX", "V", "IV",
            "I"
        };

        StringBuilder ans = new StringBuilder();

        for (int i = 0; i &lt; values.length; i++) {
            while (num &gt;= values[i]) {
                num -= values[i];
                ans.append(symbols[i]);
            }
        }

        return ans.toString();
    }
}
</code></pre>

</div>



