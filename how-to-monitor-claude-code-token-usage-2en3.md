---
Title: How to monitor Claude code token usage?
Description: CCUsage


If you're leveraging the Claude API for your coding projects, keeping track of your token consumption is crucial to managing costs and staying within limits. 
Well, there's a handy tool desi...
Author: c3phalik
Date: 2025-08-12T05:56:58.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhk6lq7fvt4i0f44gyndf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhk6lq7fvt4i0f44gyndf.png" alt="CC Usage example screenshot" width="800" height="290"></a></p>

<h2>
  
  
  Monitor Your Claude API Token Usage with <em><strong>CCUsage</strong></em>
</h2>

<p>If you're leveraging the Claude API for your coding projects, keeping track of your token consumption is crucial to managing costs and staying within limits. </p>

<p>Well, there's a handy tool designed to help you monitor your usage effectively: CCUsage. This tool provides a straightforward way to check your Claude API token usage and associated costs.</p>

<h3>
  
  
  Getting Started with CCUsage
</h3>

<p>To use CCUsage, you’ll need to have Node.js installed, as the tool runs via npx. Here are the key commands to get started:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx ccusage@latest
</code></pre>

</div>



<p>Check Token Usage by Time Period<br>
Run the following command to view your token usage for a specific timeframe:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx ccusage@latest opts [daily | weekly | monthly]
</code></pre>

</div>



<p>This command allows you to analyze your usage daily, weekly, or monthly, helping you understand your consumption patterns and costs.</p>

<p>Live Monitoring with Automatic Token Limit Detection<br>
For real-time insights, CCUsage offers a live monitoring feature that automatically detects your token limits based on your usage history:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npx ccusage@latest blocks --live
</code></pre>

</div>



<p>If you're using Claude code with the API (I don't know why would anyone do that), this tool will help you a great deal.</p>

<p>And if you use CC via the monthly plans, it's really cool to check how much you have saved using the subscription.</p>

