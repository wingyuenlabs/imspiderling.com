---
Title: JWT Explained for Beginners — With Simple Math Analogies
Description: 
Author: Zeba
Date: 2026-03-01T21:33:47.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>When a user logs in, the server needs to remember who they are on every request. One way is sessions, but that means storing data on the server. JWT solves this differently — all the user info like user ID, roles, and expiry is packed into the token itself, so the server just verifies the signature on each request without hitting the database every time.</em></p>

<p>A JWT, or JSON Web Token, is made of three parts joined by dots: Header, Payload, and Signature.<br>
<code>header.payload.signature</code></p>

<ol>
<li>The Header tells us which algorithm was used to sign the token — like HS256 or RS256 — along with the token type.</li>
<li>
<p>The Payload holds the actual user data, called "claims" — things like the user's ID, their role, when the token was issued (iat), and when it expires (exp). Common claims are: </p>

<p><code>sub → Subject (user ID)<br>
 iss → Issuer<br>
 aud → Audience<br>
 exp → Expiration time<br>
 iat → Issued at</code></p>
</li>
<li><p>The Signature is what keeps the token trustworthy.</p></li>
</ol>

<p>To create the Signature, the server takes the Base64-encoded Header and Payload, joins them, and runs them through a hashing function along with a secret key. Think of it like: <br>
<code>Header + Payload = message<br>
message + key = signature.</code><br>
This signature is then attached to the token.</p>

<p><strong>When the user sends the token back,</strong> the server needs to verify it. There are two ways depending on the algorithm. <br>
<strong><u>1: With HS256 (symmetric)</u></strong> :  During request validation, the server takes the message from the token, uses the same secret to generate a new signature, and compares it with the signature inside the token.<br>
Let's understand with an analogy:<br>
Assume:<br>
<code>secret = 3  &amp;  Signing rule is :  multiplication by secret <br>
signature = message × secret</code></p>

<p><strong>During token generation</strong></p>

<p>Assume </p>

<p><code>message = 10</code></p>

<p><code>signature =  10 * 3 = 30<br>
So token ---&gt;  10.30</code></p>

<p><strong>During Token Validation (Coming User Request (with token))</strong></p>

<p>Assume coming <code>token ---&gt;  10.30</code></p>

<p>Application will recalculate signature using the token's message:</p>

<p><code>signaure(Coming User Request) = 10 * 3  = 30</code></p>

<p>Now compare this signature with the signature attched in the token</p>

<p><code>signature(Request User's 30) == signature calculated (30)<br>
30 == 30 --&gt; True <br>
✔ Token is valid.</code></p>

<p><strong>Now, assume, attacker modified the message(Token)</strong></p>

<p><code>Assume coming token ---&gt; 18.30</code></p>

<p>Application will recalculate signature using the token's message:</p>

<p><code>signaure(Request) = 18 * 3  = 36</code><br>
But the signature is 30, so </p>

<p><code>signature(User's 36)  != signature (30)  <br>
❌ Token is Invalid.  <br>
</code></p>



<p><em>But what if we don't want to share the same secret key everywhere? That's where RS256 comes in.</em></p>

<p><strong><u> 2: With RS256 (asymmetric):</u></strong>  The server(Application) signs using a private key, but verification is done using a public key. This is useful in microservices where multiple services need to verify tokens without ever seeing the private key.</p>

<p>There are two keys:</p>

<p><code>Private key → used to SIGN<br>
Public key → used to VERIFY</code></p>

<p>Only the Identity Provider has the private key. Your API only has the public key.</p>

<p><strong>🔐 Signing Process (Token Generation)</strong></p>

<p>Assume a very simple fake math model:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Private key operation = multiply by 3
Public key operation = divide by 3
</code></pre>

</div>



<p><strong>(Note: This is NOT real RSA — just to understand concept.)</strong></p>

<p>Assume the message generated is 10<br>
<code>message = 10</code></p>

<p><strong>Now Sign Using Private Key</strong><br>
<code>signature = message × 3<br>
signature = 10 × 3 = 30</code></p>

<p>So full token conceptually: <code>10.30</code></p>

<p><strong>Verification Process:</strong><br>
<code>Requested token: 10.30<br>
message = 10 &amp; signature = 30</code></p>

<p><code>public key = divide by 3</code></p>

<p>Now it verifies:<br>
<code>expected_message = signature ÷ 3  = 30 ÷ 3 = 10<br>
</code>Now compare:<br>
<code>expected_message == original message?<br>
10 == 10 → TRUE</code><br>
✔ Token is valid.</p>

<p><u><strong>Let's see if the message (token) is modified"</strong><br>
</u><br>
<strong>Case 1: Attacker modified the message</strong> <br>
<code>Token Received: 20.30, but original token was 10.30<br>
so recieved message = 20 &amp; signature = 30<br>
</code><br>
<strong>Verification:</strong><br>
<code>expected_message = signature ÷ 3 =&gt; 30 ÷ 3 = 10</code><br>
Compare:<br>
<code>10 == 20 → FALSE<br>
❌ Token is Invalid.</code></p>

<p><strong>Case 2: Attacker/User Used the wrong Public Key</strong></p>

<p>Suppose used public key = divide by 5 (original was divided by 3)<br>
Requested token: 10.30</p>

<p>Then:<br>
<code>expected_message = 30 ÷ 5 = 6</code></p>

<p>Compare:<br>
<code>6 == 10 → FALSE<br>
❌ Invalid</code></p>

<p><em>This is why even if someone steals your token and changes the user ID to get admin access, it won't work. The signature will fail verification.</em></p>

<p>One important thing to remember. The payload is only Base64 encoded, not encrypted. Anyone can decode and read it. So never store sensitive data like passwords inside a JWT. The signature only proves the token wasn't tampered with; it does not hide the data.</p>

