---
Title: Easy Ledger CLI: a management tool for ledger-cli ledgers
Description: 
Author: Mikael
Date: 2025-12-13T21:39:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey all!</p>

<p>I recently started using <code>ledger-cli</code>, a Fat-free Accounting tool.<br>
In their own words:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>What it does offer is a double-entry accounting journal with all the flexibility and muscle of its modern day cousins, without any of the fat.
</code></pre>

</div>



<p>Check <a href="https://github.com/ledger/ledger?tab=readme-ov-file" rel="noopener noreferrer">their github</a> for more details. It's really a great concept and lovely to use for reporting and tracking your accounting.</p>

<p>The only caveat for me is the commands you type to append transactions to your ledger can get quite long and tedious. One of the main "issues" from the command line interface is that you have to write your ledger file path every time you enter a transaction (yes, the DB is a simple text file 😊). The author of ledger did add a ledger-mode allowing you to use shortcuts and syntax highlighting in order to ease the management of the ledger file, but I do not want to work from a file and would much rather add my transactions via cli in an easy, clean and straight forward way. I also do not want to know about my actual ledger file, why should I manage it myself?</p>

<p>In order to fix this, I wrote a very basic cli (for now) that allows you to:</p>

<ul>
<li>manage multiple ledgers (think one for your company, one for yourself)</li>
<li>seamlessly switch between those</li>
<li>add stocks and transactions in an interactive way</li>
</ul>

<p>In the near future I plan to support all supported transaction types and details as well as adding a few basic reports and maybe the ability to pre-configure your own ledger commands.</p>

<p>Give it a try, let me know what you think, contribute!<br>
<a href="https://www.npmjs.com/package/easy-ledger-cli" rel="noopener noreferrer">Link to the npm package</a></p>

