---
Title: 238. Product of Array Except Self | LeetCode | Top Interview 150 | Coding Questions
Description: 
Author: Debesh P.
Date: 2025-12-17T21:36:07.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<h3>
  
  
  Problem Link
</h3>

<p><a href="https://leetcode.com/problems/product-of-array-except-self/" rel="noopener noreferrer">https://leetcode.com/problems/product-of-array-except-self/</a></p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftkvpcrez97yutul6ur0c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftkvpcrez97yutul6ur0c.png" alt="leetcode 238" width="800" height="450"></a></p>




<h3>
  
  
  Solution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] left = new int[n];
        int[] right = new int[n];

        left[0] = 1;
        for(int i=1; i&lt;n; i++) {
            left[i] = left[i-1] * nums[i-1]; 
        }

        right[n-1] = 1;
        for(int i=n-2; i&gt;=0; i--) {
            right[i] = right[i+1] * nums[i+1];
        }

        int[] ans = new int[n];
        for(int i=0; i&lt;n; i++) {
            ans[i] = left[i] * right[i];
        }

        return ans;

    }
}
</code></pre>

</div>



