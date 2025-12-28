---
Title: 68. Text Justification | LeetCode | Top Interview 150 | Coding Questions
Description: 
Author: Debesh P.
Date: 2025-12-28T21:53:49.000Z
Robots: noindex,nofollow
Template: index
---
<blockquote>
<h3>
  
  
  Problem Link
</h3>

<p><a href="https://leetcode.com/problems/text-justification/" rel="noopener noreferrer">https://leetcode.com/problems/text-justification/</a></p>
</blockquote>




<blockquote>
<h3>
  
  
  Detailed Step-by-Step Explanation
</h3>

<p><a href="https://leetcode.com/problems/text-justification/solutions/7446966/most-optimal-strategy-with-code-beats-10-zhfo" rel="noopener noreferrer">https://leetcode.com/problems/text-justification/solutions/7446966/most-optimal-strategy-with-code-beats-10-zhfo</a></p>
</blockquote>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzfvjerg81k3aaonqiia4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzfvjerg81k3aaonqiia4.png" alt="leetcode 68" width="800" height="450"></a></p>




<h3>
  
  
  Solution
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>class Solution {
    public List&lt;String&gt; fullJustify(String[] words, int maxWidth) {
        List&lt;String&gt; result = new ArrayList&lt;&gt;();
        int n = words.length;
        int index = 0;

        while (index &lt; n) {
            int totalChars = words[index].length();
            int last = index + 1;

            while (last &lt; n) {
                if (totalChars + 1 + words[last].length() &gt; maxWidth)
                    break;
                totalChars += 1 + words[last].length();
                last++;
            }

            StringBuilder sb = new StringBuilder();
            int diff = last - index - 1;

            if (last == n || diff == 0) {
                for (int i = index; i &lt; last; i++) {
                    sb.append(words[i]);
                    if (i &lt; last - 1) {
                        sb.append(" ");
                    }
                }
                int remainingSpaces = maxWidth - sb.length();
                appendSpaces(sb, remainingSpaces);
            } else {
                int spaces = (maxWidth - totalChars) / diff;
                int extraSpaces = (maxWidth - totalChars) % diff;

                for (int i = index; i &lt; last; i++) {
                    sb.append(words[i]);
                    if (i &lt; last - 1) {
                        int spacesToApply = spaces + (i - index &lt; extraSpaces ? 1 : 0);
                        appendSpaces(sb, spacesToApply + 1);
                    }
                }
            }

            result.add(sb.toString());
            index = last;
        }

        return result;
    }

    private void appendSpaces(StringBuilder sb, int count) {
        for (int i = 0; i &lt; count; i++) {
            sb.append(" ");
        }
    }
}
</code></pre>

</div>



