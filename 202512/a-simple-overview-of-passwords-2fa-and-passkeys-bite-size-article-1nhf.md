---
Title: A Simple Overview of Passwords, 2FA, and Passkeys (Bite-size Article)
Description: 
Author: koshirok096
Date: 2025-12-19T21:31:44.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Introduction
</h1>

<p>It may feel a little late to talk about this now, but—are you using Passkeys?</p>

<p>Over the history of the web and modern technology, several authentication methods have come and gone. If we focus only on the major ones, the mainstream progression has been Password → 2FA/MFA → Passkey. Today, Passkeys are considered the most secure option and are positioned as an official W3C standard—effectively the “new default.”<br><br>
That said, many services still rely on Password-only or 2FA/MFA logins, so we live in a world where multiple authentication methods coexist.</p>

<p>This is a short article, but I’d like to briefly整理 the evolution of these authentication methods.</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs8urojb8busq1fq83dn6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs8urojb8busq1fq83dn6.png" alt=" " width="800" height="450"></a></p>

<h1>
  
  
  What is a Password?
</h1>

<p>A Password is, needless to say, a “secret phrase” you memorize and type in yourself.<br><br>
In the early days of the web, many services relied solely on this simple authentication method. However, due to its security weaknesses, password-only authentication has gradually been phased out, especially among large platforms.</p>

<p><strong>Pros:</strong>  </p>

<ul>
<li>Simple and easy for anyone to use
</li>
</ul>

<p><strong>Cons:</strong>  </p>

<ul>
<li>Easily stolen and high-risk
</li>
<li>Often reused across multiple services, leading to greater exposure
</li>
<li>Data breaches on the server side are outside the user’s control and cannot be prevented by personal effort
</li>
</ul>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvosq3xmr0col4e035as.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxvosq3xmr0col4e035as.png" alt=" " width="800" height="450"></a></p>

<h1>
  
  
  What is 2FA (Two-Factor Authentication)?
</h1>

<p>2FA strengthens password-based logins by adding <strong>one more step</strong> for increased security.<br><br>
This method is widely used today, and many people are already familiar with it. SMS codes, authenticator apps, and email verification are typical examples. By combining them, the system requires:</p>

<p><strong>“You know the password” + “You can confirm the code on your device”</strong></p>

<p>This two-step process significantly raises the security level.</p>

<p><strong>Pros:</strong>  </p>

<ul>
<li>Even if your password leaks, attackers cannot easily break in
</li>
</ul>

<p><strong>Cons:</strong>  </p>

<ul>
<li>More steps than password-only login
</li>
<li>Not all 2FA methods are equal (e.g., SMS is relatively weak, TOTP apps are moderate, hardware keys like FIDO2 are strong)
</li>
<li>Codes can still be stolen through phishing attacks
</li>
</ul>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa1zq81s6jf9398q7fx5k.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa1zq81s6jf9398q7fx5k.png" alt=" " width="800" height="450"></a></p>

<h1>
  
  
  What is a Passkey?
</h1>

<p>Passkeys are promoted by major tech companies such as Apple, Google, and Microsoft, and are currently positioned as the W3C standard—the “successor” to passwords. This new authentication method has gained significant attention in recent years, especially because it <strong>eliminates the need for passwords altogether</strong>.</p>

<p>Your device stores a cryptographic key (a private key), and when logging in, the device automatically proves your identity using your PIN or biometric authentication (face or fingerprint).</p>

<p><strong>Pros:</strong>  </p>

<ul>
<li>No passwords → extremely low risk of theft or data leakage
</li>
<li>Fast and convenient, secured by device-level protection and biometrics
</li>
</ul>

<p><strong>Cons:</strong>  </p>

<ul>
<li>Not yet supported by all services; adoption is still ongoing
</li>
<li>If you lose your device, you cannot simply “remember” or recreate your key—backup and sync are essential
</li>
</ul>




<h1>
  
  
  A Simple Comparison of the Three Methods
</h1>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Category</th>
<th><strong>Password</strong></th>
<th><strong>2FA (MFA)</strong></th>
<th><strong>Passkey</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Core Mechanism</strong></td>
<td>Entering a memorized string</td>
<td>Password + additional verification code</td>
<td>Signing with a private key stored on device (no password)</td>
</tr>
<tr>
<td><strong>Who Holds the Key?</strong></td>
<td>User (memory)</td>
<td>User (memory + code delivery device)</td>
<td>Device (private key)</td>
</tr>
<tr>
<td><strong>Authentication Flow</strong></td>
<td>Send password → match check</td>
<td>Password → SMS/app code</td>
<td>Biometric or PIN unlocks private key for signing</td>
</tr>
<tr>
<td><strong>Security Level</strong></td>
<td>Low</td>
<td>Medium</td>
<td>High (phishing-resistant)</td>
</tr>
<tr>
<td><strong>Strengths</strong></td>
<td>Simple and widely supported</td>
<td>More secure than passwords</td>
<td>Fast, secure, and passwordless</td>
</tr>
<tr>
<td><strong>Weaknesses</strong></td>
<td>Easily leaked or reused</td>
<td>Extra steps; codes can be stolen</td>
<td>Requires backup if device is lost</td>
</tr>
<tr>
<td><strong>Main Risks</strong></td>
<td>Theft, reuse, data breaches</td>
<td>Phishing via code relay</td>
<td>Device theft + PIN exposure</td>
</tr>
<tr>
<td><strong>Availability</strong></td>
<td>Works everywhere</td>
<td>Supported by most major services</td>
<td>Expanding but not universal yet</td>
</tr>
</tbody>
</table></div>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk77coowmz8t80xpm5ymj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk77coowmz8t80xpm5ymj.png" alt=" " width="800" height="450"></a></p>

<h1>
  
  
  Conclusion
</h1>

<p>In this article, I simply整理ed my own understanding of authentication technologies, focusing only on the high-level concepts without diving too deeply into technical details. If any explanation feels incomplete, I appreciate your understanding.</p>

<p>When you look at all three methods side by side, Passkeys indeed offer extremely strong security—yet they are not perfect. Each authentication method from the past also had its own strengths and weaknesses. It’s interesting that even in a world with such advanced technology, we still haven’t developed a “flawless” authentication system.</p>

<p>Thank you for reading!</p>

