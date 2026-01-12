---
Title: Why Your Secret Sharing Tool Needs Post-Quantum Cryptography Today
Description: 
Author: Whaaat!
Date: 2026-01-12T21:24:02.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The "Harvest Now, Decrypt Later" Threat
</h2>

<p>Quantum computers capable of breaking RSA and ECC encryption don't exist yet. But here's the problem: adversaries are already collecting encrypted data today, planning to decrypt it once quantum computers arrive.</p>

<p>For sensitive data that needs to remain confidential for years, this is a real threat.</p>

<h2>
  
  
  What is Post-Quantum Cryptography?
</h2>

<p>Post-quantum cryptography (PQC) uses mathematical problems that are hard for both classical AND quantum computers to solve. In August 2024, NIST standardized three PQC algorithms:</p>

<ul>
<li>
<strong>ML-KEM (Kyber)</strong> - Key encapsulation</li>
<li>
<strong>ML-DSA (Dilithium)</strong> - Digital signatures</li>
<li>
<strong>SLH-DSA (SPHINCS+)</strong> - Hash-based signatures</li>
</ul>

<h2>
  
  
  Implementing PQC in a Web Application
</h2>

<p>I recently added PQC support to <a href="https://notrust.now" rel="noopener noreferrer">NoTrust.now</a>, a zero-knowledge secret sharing tool. Here's how:</p>

<h3>
  
  
  Key Exchange with ML-KEM-768
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Using crystals-kyber-js library</span>
<span class="k">import</span> <span class="p">{</span> <span class="nx">MlKem768</span> <span class="p">}</span> <span class="k">from</span> <span class="dl">'</span><span class="s1">crystals-kyber-js</span><span class="dl">'</span><span class="p">;</span>

<span class="c1">// Receiver generates keypair</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">publicKey</span><span class="p">,</span> <span class="nx">privateKey</span><span class="p">]</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">MlKem768</span><span class="p">.</span><span class="nf">generateKeyPair</span><span class="p">();</span>

<span class="c1">// Sender encapsulates a shared secret</span>
<span class="kd">const</span> <span class="p">[</span><span class="nx">ciphertext</span><span class="p">,</span> <span class="nx">sharedSecret</span><span class="p">]</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">MlKem768</span><span class="p">.</span><span class="nf">encapsulate</span><span class="p">(</span><span class="nx">publicKey</span><span class="p">);</span>

<span class="c1">// Receiver decapsulates to get the same shared secret</span>
<span class="kd">const</span> <span class="nx">decryptedSecret</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">MlKem768</span><span class="p">.</span><span class="nf">decapsulate</span><span class="p">(</span><span class="nx">ciphertext</span><span class="p">,</span> <span class="nx">privateKey</span><span class="p">);</span>
</code></pre>

</div>



<h3>
  
  
  Hybrid Approach
</h3>

<p>For defense in depth, combine PQC with classical crypto:</p>

<ol>
<li>Generate ephemeral X25519 keypair (classical)</li>
<li>Generate ephemeral ML-KEM-768 keypair (post-quantum)</li>
<li>Combine both shared secrets: <code>finalKey = HKDF(x25519Secret || kyberSecret)</code>
</li>
</ol>

<p>This ensures security even if one algorithm is broken.</p>

<h2>
  
  
  Try It Out
</h2>

<p>You can test PQC secret sharing at <a href="https://notrust.now/createpqc" rel="noopener noreferrer">NoTrust.now/createpqc</a>. The encryption happens entirely in your browser - zero-knowledge architecture means the server never sees your plaintext.</p>

<h2>
  
  
  Resources
</h2>

<ul>
<li><a href="https://www.nist.gov/pqcrypto" rel="noopener noreferrer">NIST PQC Standards</a></li>
<li><a href="https://github.com/aspect-security/crystals-kyber-js" rel="noopener noreferrer">crystals-kyber-js</a></li>
<li><a href="https://blog.cloudflare.com/post-quantum-cryptography-for-developers/" rel="noopener noreferrer">Post-Quantum Cryptography for Developers</a></li>
</ul>




<p>What do you think about PQC adoption? Too early or just in time? Let me know in the comments.</p>

