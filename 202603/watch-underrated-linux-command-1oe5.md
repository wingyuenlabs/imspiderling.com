---
Title: watch — underrated Linux command
Description: 
Author: Michael Saparov
Date: 2026-03-06T21:39:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>Many Linux users write small scripts to monitor system state — CPU usage, disk space, open connections, and more.<br>
But there is a much simpler tool built into most Linux systems: watch.<br>
It allows you to run any command repeatedly and see the output update in real time.</p>
<h2>
  
  
  Basic usage
</h2>

<p>The syntax is simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>watch command
</code></pre>

</div>



<p>By default, the command runs every 2 seconds.</p>

<h2>
  
  
  Example: monitoring the mail queue
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>watch mailq
</code></pre>

</div>



<p>This refreshes the list of outgoing messages every two seconds so you can see the mail queue shrink or grow in real time.</p>

<h2>
  
  
  Example: watching network connections
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>watch ss -tuln
</code></pre>

</div>



<p>This is useful when debugging network services or checking if a port becomes active.</p>

<h2>
  
  
  Example: monitoring processes
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>watch "ps aux | grep nginx"
</code></pre>

</div>



<p>This lets you observe whether a process starts, stops, or changes.</p>

<h2>
  
  
  Changing the interval
</h2>

<p>You can change the refresh interval with <code>-n</code>.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>watch -n 1 mailq
</code></pre>

</div>



<p>This runs the command every 1 second.</p>

<h2>
  
  
  Highlighting differences
</h2>

<p>One of the most useful options is <code>-d</code>, which highlights changes between updates.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>watch -d netstat -tuln
</code></pre>

</div>



<p>This makes it much easier to see what changed since the previous refresh.</p>

<h2>
  
  
  Conclusion
</h2>

<p>watch is a small but powerful tool.<br>
It can turn almost any command into a simple real-time monitoring dashboard without writing scripts.<br>
Sometimes the best tools in Linux are also the simplest.</p>

