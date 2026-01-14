---
Title: Everything you want to know about Ethereum Stateless
Description: 
Author: Yelyzaveta Dymchenko
Date: 2026-01-14T20:57:25.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>The Ethereum State is becoming unmanageable.</strong></p>

<p>To verify blocks today, running a node requires a 2TB+ NVMe SSD and significant bandwidth. This hardware barrier forces centralization, pushing users to rely on trusted third parties like Infura or Alchemy instead of verifying the chain themselves.</p>

<p>The solution is <strong>Stateless Consensus</strong> - a fundamental shift in how nodes communicate. This article explores the mechanics of Verkle Trees, the engineering constraints of the network, and why solving the "Concurrency Trap" is critical for Ethereum's roadmap.</p>




<h2>
  
  
  The Constraints: Latency &amp; Bandwidth
</h2>

<p><strong>Why can't we just send the data needed to verify a block?</strong></p>

<p>To verify a block statelessly, a node needs specific account values (balances, nonces, code) along with a cryptographic proof that these values are correct. This package is called <strong>The Witness</strong>.</p>

<p>The problem is network propagation. Blocks must propagate across the global P2P network in seconds. If we used today's math, the Witness would be <strong>10-20 MB</strong> per block. At that size, propagation slows down, stale block rates increase, and only massive data centers can keep up.</p>

<p>To make statelessness viable, we need to shrink that proof from 20 MB to <strong>&lt;100 KB</strong>.</p>




<h2>
  
  
  The Architecture of Statelessness
</h2>

<p>Statelessness splits the network into two distinct classes of nodes:</p>

<ol>
<li> <strong>Proposers (Heavy):</strong> These nodes store the full 2TB state. They build the blocks and generate the Witness.</li>
<li> <strong>Validators (Light):</strong> These nodes store nothing. They receive the <code>(Block, Witness)</code> tuple and verify it mathematically.</li>
</ol>




<h2>
  
  
  The Bottleneck: Merkle Patricia Tries
</h2>

<p>Ethereum currently uses Merkle Patricia Tries (Radix-16). In this structure, every node has 16 children. To prove that one specific child value exists, you must provide the hashes of all <strong>15 siblings</strong> at every level of the tree's depth.</p>

<p><strong>The Math:</strong></p>

<blockquote>
<p>Size ≈ 15 × Depth × 32 bytes</p>
</blockquote>

<p>This scales linearly. As the state grows, the tree gets deeper, and the proof gets bigger. Currently, this results in roughly <strong>3-4 KB per account</strong>. In a busy block consuming 30M gas, the total witness data exceeds the bandwidth budget of the P2P network.</p>




<h2>
  
  
  The Fix: Verkle Trees &amp; Vector Commitments
</h2>

<p>To solve this, Ethereum is moving to <strong>Verkle Trees</strong>.</p>

<p>In a Verkle Tree, we make the structure much <strong>wider</strong>. Each node has <strong>256 children</strong> instead of 16. Normally, proving one child among 256 would require sending 255 sibling hashes—a massive amount of data.</p>

<h3>
  
  
  The Solution: Polynomials
</h3>

<p>Instead of hashing the siblings, we treat the 256 children as coefficients (v) of a polynomial P(x). It looks like this:</p>

<blockquote>
<p>P(x) = v₀ + v₁x + v₂x² + ... + v₂₅₅x²⁵⁵</p>
</blockquote>

<p>We then "commit" to this entire polynomial using <strong>KZG Commitments</strong>.</p>

<h3>
  
  
  The KZG Magic Formula 🔮
</h3>

<p>The commitment C is a single curve point, derived from a secret value s. It is calculated by multiplying the Generator point G by the polynomial's value at s:</p>

<blockquote>
<p>C = G • P(s)</p>
</blockquote>

<p>To prove a specific child value y at index z, the prover calculates a <strong>Quotient Polynomial</strong> Q(x):</p>

<blockquote>
<p>Q(x) = (P(x) - y) / (x - z)</p>
</blockquote>

<p><strong>Why this formula works:</strong><br>
The logic relies on polynomial divisibility. If the value at index <code>z</code> is truly <code>y</code>, then subtracting <code>y</code> from the polynomial makes it perfectly divisible by <code>(x - z)</code>. If the value were anything else, there would be a "remainder" left over, and <code>Q(x)</code> would not be a valid polynomial. By providing the commitment to <code>Q(x)</code>, the prover demonstrates that the division was clean and the data is correct.</p>

<p>The "Proof" is simply the commitment to this quotient polynomial <code>Q(x)</code>. This proof is a constant <strong>48 bytes</strong>, regardless of whether the tree is wide or narrow.</p>

<h3>
  
  
  The Impact: Bandwidth Saved
</h3>

<ul>
<li>
<strong>Merkle Proof:</strong> ~3,000 bytes per account (Linear complexity).</li>
<li>
<strong>Verkle Proof:</strong> ~150 bytes per account (Constant complexity).
This reduces the stateless block witness from <strong>10 MB</strong> down to <strong>200 KB</strong>.</li>
</ul>




<h2>
  
  
  The Engineering Challenges
</h2>

<h3>
  
  
  1. The Trade-off: Trusted Setup
</h3>

<p>KZG Commitments require a "Structured Reference String" (SRS)—a set of parameters generated using a secret value $s$.</p>

<ul>
<li>
<strong>The Risk:</strong> If an attacker knows <code>s</code>, they can <strong>fake</strong> proofs <code>P'(s) = P(s)</code> and trick the network into accepting bad data.</li>
<li>
<strong>The Defense:</strong> The <strong>"Powers of Tau" Ceremony</strong>. Over 140,000 participants contributed entropy to generate these parameters. As long as <strong>one</strong> single person in that chain deleted their "toxic waste" (secret randomness), the system is mathematically secure.</li>
</ul>

<h3>
  
  
  2. The Concurrency Trap (Race Condition)
</h3>

<p>Who should generate these proofs? There are two approaches:</p>

<ol>
<li> <strong>Strong Statelessness:</strong> Users generate proofs for their own transactions.</li>
<li> <strong>Weak Statelessness:</strong> Block Builders generate proofs for everyone.</li>
</ol>

<p>Ethereum has chosen <strong>Weak Statelessness</strong> because of a fundamental distributed systems problem: <strong>The Race Condition.</strong></p>

<p>If users had to make these proofs themselves, their transactions would often fail when the network is busy. Therefore, the burden is placed on the <strong>Block Builders</strong> (Proposers), who have the server capacity to regenerate proofs instantly.</p>




<h2>
  
  
  Future Proofing: State Expiry
</h2>

<p>Another piece of the puzzle is <strong>State Expiry</strong> (EIP-7736). Statelessness solves the <em>verification</em> problem, but the underlying database (stored by Proposers) still grows endlessly.</p>

<ul>
<li>
<strong>Mechanism:</strong> Epoch-based tree rotation.</li>
<li>
<strong>Logic:</strong> If a "leaf" (account) is not accessed for ~1 year, it is dropped from the active tree.</li>
<li>
<strong>Restoration:</strong> To use that account again, the user must provide a proof from the "Archive" to reactivate it.</li>
</ul>




<h2>
  
  
  Conclusion: The "Verge" Vision
</h2>

<p>All of this math serves one singular vision in Vitalik’s roadmap, known as <strong>"The Verge."</strong> The goal is to make the chain so lightweight that verifying Ethereum no longer requires trusting centralized providers like Alchemy or Infura.</p>

<p><strong>Today:</strong> We trust.<br>
<strong>Tomorrow:</strong> We verify.</p>




<h2>
  
  
  Summary &amp; Specs
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Feature</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Tech</strong></td>
<td>KZG Polynomials compress 256 siblings into 48 bytes.</td>
</tr>
<tr>
<td><strong>Network</strong></td>
<td>Reduces bandwidth overhead by ~99%.</td>
</tr>
<tr>
<td><strong>UX</strong></td>
<td>"Weak" statelessness prevents transaction failures.</td>
</tr>
<tr>
<td><strong>Target</strong></td>
<td>The "Hegota" Upgrade (H2 2026).</td>
</tr>
</tbody>
</table></div>

<p><strong>Related EIPs:</strong></p>

<ul>
<li><a href="https://eips.ethereum.org/EIPS/eip-6800" rel="noopener noreferrer">#EIP6800 (Verkle Trees)</a></li>
<li><a href="https://eips.ethereum.org/EIPS/eip-4762" rel="noopener noreferrer">#EIP4762 (Gas Costs)</a></li>
</ul>

<h3>
  
  
  Top Resources
</h3>

<ol>
<li> <strong>Vitalik Buterin:</strong> "Verkle Trees"

<ul>
<li><a href="https://vitalik.eth.limo/general/2021/06/18/verkle.html" rel="noopener noreferrer">Link</a></li>
</ul>
</li>
<li> <strong>Dankrad Feist:</strong> "Verkle Trie for Eth1 State"

<ul>
<li><a href="https://dankradfeist.de/ethereum/2021/06/18/verkle-trie-for-eth1.html" rel="noopener noreferrer">Link</a></li>
</ul>
</li>
</ol>

