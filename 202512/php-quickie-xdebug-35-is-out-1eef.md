---
Title: Php quickie: Xdebug 3.5 is out
Description: 
Author: david duymelinck
Date: 2025-12-08T21:07:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>I have been waiting for the new release, because it has PHP 8.5 support.</p>

<h2>
  
  
  Installation
</h2>

<p>I recommend to use <a href="https://github.com/php/pie" rel="noopener noreferrer">PIE</a>. The main reason is that you can use it on all the OS's. So not need to figure out how the extension needs to be installed on your OS. <br>
There is even a PIE command that checks the composer.json file for missing extensions. </p>

<p>The one gotcha I encountered was not finding <code>phpize</code>. For the people that are not familiar with the command, it is a part of the <code>php-dev</code> package, and it is build for a specific PHP version. The great thing about the package is that you can add the version, <code>php8.5-dev</code>, so that a <code>phpize</code> command is created with that version, <code>phpize8.5</code>. For PIE the name of the command doesn't matter.</p>

<p>The installation command is simply <code>pie install xdebug/xdebug</code>. Run <code>php --version</code> to verify xdebug is enabled. And optionally check the config file if you need to make changes.</p>

<h2>
  
  
  Usage
</h2>

<p>I'm still exploring the debug possibilities. But for the pipe operator it can <a href="https://bugs.xdebug.org/bug_view_page.php?bug_id=00002363" rel="noopener noreferrer">intercept the intermediate values</a>, that is cool.</p>

<p>Happy debugging!</p>

