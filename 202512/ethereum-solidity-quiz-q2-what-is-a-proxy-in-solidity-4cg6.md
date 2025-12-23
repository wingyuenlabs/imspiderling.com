---
Title: Ethereum-Solidity Quiz Q2: What is a proxy in Solidity?
Description: 
Author: MihaiHng
Date: 2025-12-23T21:57:48.000Z
Robots: noindex,nofollow
Template: index
---
<p>A proxy in Solidity is a design pattern used to enable contract upgradability. This is important because smart contract code is immutable once deployed on the blockchain.</p>

<p>A proxy is a smart contract that stores state variables while delegating all its logic to one or several implementation contracts.</p>

<p>When the proxy receives a call, it forwards it to the logic contract using a low-level <strong>delegatecall</strong>, which is crucial, because it executes the code from the implementation contract but within the context (storage, msg.sender, msg.value, etc.) of the proxy contract itself.</p>

<p>The basic upgradeable arhitecture:</p>

<ol>
<li>Proxy contract — Stores the state and delegates calls to the implementation</li>
<li>Implementation contract — Contains the actual logic</li>
<li>Admin — A separate address(or Multisig even better) that can upgrade the implementation contract address stored in the proxy</li>
</ol>

<p>Common proxy patterns:</p>

<ol>
<li>UUPS (Universal Upgradeable Proxy Standard) — The upgrade logic is in the implementation contract</li>
<li>Transparent Proxy — Uses a more complex pattern to avoid function selector conflicts</li>
<li>Beacon Proxy — Multiple proxies can point to a single beacon contract that holds the implementation address</li>
</ol>

