---
Title: Irish-Name-Repo 2 - picoCTF '19 (web)
Description: 
Author: David Disu
Date: 2025-09-07T21:57:46.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk01z47emxfryc7h64j22.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk01z47emxfryc7h64j22.png" alt="Challenge description" width="800" height="467"></a></p>

<p>Coming from this challenge's prequel <a href="https://dev.to/davidonlinearchive/irish-name-repo-1-picoctf-19-web-16ka">Irish-Name-Repo 1 - picoCTF '19</a>, I was hellbent thinking I had to encode the password parameter. I tried several SQL injection variations, including:<br>
<code>' oR 1=1 --</code> - case manipulation<br>
<code>%27%20%20%6f%72%20%31%3d%31%20%2d%2d</code>- URL encoding<br>
<code>'/**/ or /**/ 1=1 /**/ --</code> - Comment obfuscation <br>
<code>00%' or 1=1 --</code> - null hex encoding<br>
...and other combinations but to no avail. Once I shifted my attention, the solution became straightforward.</p>

<p><strong>STEPS TO SOLUTION</strong><br>
use <code>admin'--</code> in the username parameter. </p>

<p><strong>Breakdown:</strong></p>

<ul>
<li>
<code>admin</code> - value for username query. </li>
<li>
<code>'</code> - closes the input string.</li>
<li>
<code>--</code> - comments out the remaining query.</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fksui9e78y3476b6cf0lb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fksui9e78y3476b6cf0lb.png" alt="Challenge solved" width="800" height="249"></a></p>

<p><strong>FLAG:</strong> <em>picoCTF{m0R3_SQL_plz_fa983901}</em></p>

<p><strong>PWNSOME REFERENCES</strong><br>
<a href="https://portswigger.net/support/sql-injection-bypassing-common-filters" rel="noopener noreferrer">https://portswigger.net/support/sql-injection-bypassing-common-filters</a><br>
<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection" rel="noopener noreferrer">https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection</a></p>

