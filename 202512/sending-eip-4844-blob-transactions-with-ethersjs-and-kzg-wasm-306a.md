---
Title: Sending EIP-4844 Blob Transactions with ethers.js and kzg-wasm
Description: 
Author: Kurt
Date: 2025-12-22T21:19:43.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkb1umvy0hci4jl35xdh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkb1umvy0hci4jl35xdh.png" alt="BlobBytes" width="800" height="374"></a></p>

<p>EIP-4844 introduces <strong>blob transactions (Type 3)</strong> as part of Proto-Danksharding. Sending blob transactions in practice comes with constraints around tooling, RPC support, and transaction preparation.</p>

<p>This article summarizes hands-on experimentation with <strong>ethers.js v6</strong> and <strong>kzg-wasm</strong>, focusing on what works, what does not, and how to get a minimal setup running.</p>




<p>If you just want to create a blob without diving into code, <strong><a href="https://blobsender.xyz" rel="noopener noreferrer">https://blobsender.xyz</a></strong> lets anyone write a message and submit it as a blob on Ethereum.</p>




<h3>
  
  
  Current limitations
</h3>

<ul>
<li><p><strong>Browser wallets and connectors do not support blob transactions yet.</strong><br>
They must be sent from a backend or script controlling a private key directly.</p></li>
<li><p><strong>RPC support is inconsistent.</strong><br>
Even on Sepolia, many public RPC endpoints reject blob transactions or do not support EIP-4844.</p></li>
</ul>




<h3>
  
  
  RPC support on Sepolia
</h3>

<p>RPC support was one of the main friction points. About <strong>20 Sepolia RPC providers</strong> were tested. Many rejected blob transactions or lacked EIP-4844 support.</p>

<p>The following public RPC endpoints worked reliably:</p>

<p><a href="https://sepolia.infura.io/v3/YOUR_API_KEY" rel="noopener noreferrer">https://sepolia.infura.io/v3/YOUR_API_KEY</a><br>
<a href="https://ethereum-sepolia-rpc.publicnode.com" rel="noopener noreferrer">https://ethereum-sepolia-rpc.publicnode.com</a><br>
<a href="https://eth-sepolia.api.onfinality.io/public" rel="noopener noreferrer">https://eth-sepolia.api.onfinality.io/public</a><br>
<a href="https://0xrpc.io/sep" rel="noopener noreferrer">https://0xrpc.io/sep</a><br>
<a href="https://sepolia.drpc.org" rel="noopener noreferrer">https://sepolia.drpc.org</a></p>

<p>This list is included in the repository to save others testing time.</p>




<h3>
  
  
  Repository and code
</h3>

<p>All working examples live here:</p>

<p><a href="https://github.com/0xKurt/eip-4844-ethers-examples" rel="noopener noreferrer">https://github.com/0xKurt/eip-4844-ethers-examples</a></p>

<p>The repository contains <strong>two scripts</strong>:</p>

<ul>
<li>
<strong>send-blob.ts</strong> – a simple blob transaction</li>
<li>
<strong>send-blob-contract.ts</strong> – a blob transaction combined with a contract call</li>
</ul>

<p>Both use <strong>ethers.js v6</strong> and <strong>kzg-wasm</strong>, and send <strong>Type 3 transactions</strong>.</p>

<p>At a high level, sending a blob transaction requires preparing a fixed-size blob, generating a <strong>KZG commitment</strong> and <strong>proof</strong>, and sending a transaction with <code>type: 3</code>.</p>




<h3>
  
  
  Blobscan
</h3>

<p><strong>Blobscan</strong> allows inspection of:</p>

<ul>
<li>blob versioned hashes</li>
<li>blob gas prices</li>
<li>other blob-related metadata</li>
</ul>

<p>The scripts print direct links to Blobscan using the blob versioned hash returned by ethers, making verification and inspection easy.</p>




<h3>
  
  
  Full walkthrough
</h3>

<p>This post is a condensed summary. A complete step-by-step walkthrough of setup, code, and transaction flow is available in the original Medium article:</p>

<p><a href="https://medium.com/@Kurt0x/sending-eip-4844-blob-transactions-using-ethers-js-and-kzg-wasm-d84224be6b81" rel="noopener noreferrer">https://medium.com/@Kurt0x/sending-eip-4844-blob-transactions-using-ethers-js-and-kzg-wasm-d84224be6b81</a></p>

