---
Title: 14. Longest Common Prefix | LeetCode | Top Interview 150 | Coding Questions
Description: 
Author: Debesh P.
Date: 2025-12-26T21:23:45.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<h3>
  
  
  Problem Link
</h3>

<p><a href="https://leetcode.com/problems/longest-common-prefix/" rel="noopener noreferrer">https://leetcode.com/problems/longest-common-prefix/</a></p>
</blockquote>




<blockquote>
<h3>
  
  
  Detailed Step-by-Step Explanation
</h3>

<p><a href="https://leetcode.com/problems/longest-common-prefix/solutions/7441779/most-optimal-solution-beats-100-best-app-jxyq" rel="noopener noreferrer">https://leetcode.com/problems/longest-common-prefix/solutions/7441779/most-optimal-solution-beats-100-best-app-jxyq</a></p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp76wfwhexntzfhb646j4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp76wfwhexntzfhb646j4.png" alt="leetcode 14" width="800" height="450"></a></p>




<h3>
  
  
  Solution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Solution {
    public String longestCommonPrefix(String[] strs) {

        int n = strs.length;

        if (strs == null || n == 0) {
            return "";
        }

        String prefix = strs[0];

        for (int i = 1; i &lt; n; i++) {
            while (strs[i].indexOf(prefix) != 0) {
                prefix = prefix.substring(0, prefix.length() - 1);
                if (prefix.isEmpty()) {
                    return "";
                }
            }
        }

        return prefix;
    }
}
</code></pre>

</div>



