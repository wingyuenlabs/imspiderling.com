---
Title: Notifications in JavaScript: Introducing notify-zh 🔔
Description: 
Author: xavier gonzalez
Date: 2025-09-07T21:30:13.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey everyone!<br>
<a href="https://notify.piratacode.com/" rel="noopener noreferrer">https://notify.piratacode.com/</a></p>

<p>I want to share a new notification library I've created with you: notify-zh. I've always looked for a simple and lightweight way to display notifications in my JavaScript projects, and since I couldn't find anything that fit my needs, I decided to build it myself.</p>

<p>What does notify-zh do?<br>
notify-zh is a very easy-to-use and customizable library for showing simple notifications in your web applications. Its main goal is to be as minimalist and lightweight as possible without sacrificing important features like:</p>

<p><em><strong>Simple setup: You don't need much code to get started.</strong></em></p>

<blockquote>
<p>Multiple positions: Show notifications in different corners of the screen.</p>

<p>Customizable themes:<br>
 Choose from predefined themes or create your own.</p>

<p>Configurable duration: Control how long the notification remains visible.</p>

<p>Close action: Users can manually close the notification.</p>
</blockquote>

<p><strong>Why use notify-zh?</strong><br>
If you're looking for a quick and efficient solution to add notifications to your project without a heavy library, notify-zh is an excellent choice. It's perfect for:</p>

<p>Small and medium-sized projects.</p>

<p>Prototypes and MVPs (Minimum Viable Products).</p>

<p>Projects where performance is a priority.</p>

<p>How to get started?<br>
Installation is simple with npm:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Bash

npm install notify-zh
</code></pre>

</div>



<p>Then, you can import and use it in your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>JavaScript

import { showNotification } from 'notify-zh';

// Success notification
showNotification('Your action has been completed!', {
  type: 'success',
  position: 'top-right',
  duration: 3000
});

// Error notification
showNotification('An error occurred. Please try again.', {
  type: 'error',
  title: 'Error'
});
</code></pre>

</div>



<p>I'd love to hear your feedback!<br>
notify-zh is available on npm. I'd love for you to try it out in your projects and give me your feedback! You can find the full documentation on its npm page: <a href="https://www.npmjs.com/package/notify-zh" rel="noopener noreferrer">https://www.npmjs.com/package/notify-zh</a><br>
or <br>
<a href="https://notify.piratacode.com/" rel="noopener noreferrer">https://notify.piratacode.com/</a><br>
Thanks for reading, and I hope notify-zh is useful to you! 😊</p>

