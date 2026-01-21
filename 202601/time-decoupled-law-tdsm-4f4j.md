---
Title: Time-Decoupled Law (TDSM)
Description: 
Author: Abstract36
Date: 2026-01-21T20:59:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>Abstract<br>
Blockchain privacy protocols have traditionally focused on concealing participant identities and transaction values, while treating temporal information as benign metadata. However, recent advances in on-chain analytics demonstrate that transaction timing constitutes a powerful side channel, enabling correlation and deanonymization even in systems employing strong cryptographic primitives such as zero-knowledge proofs.<br>
In this paper, we introduce Time-Decoupled Law (TDSM), a protocol-level primitive that treats time as an independent object of enforcement rather than a public parameter derived from block metadata. TDSM enforces non-manipulable execution delays without relying on block.timestamp, trusted oracles, or administrative control, thereby eliminating a fundamental class of timing-based correlation attacks.<br>
We formalize the concept of temporal unlinkability, define the adversarial model under which timing inference is feasible, and demonstrate how existing privacy systems fail to address this dimension. By decoupling execution eligibility from observable blockchain time, TDSM enables privacy-preserving applications in which neither identity nor timing information can be reliably correlated.</p>

<ol>
<li>Introduction
1.1 Motivation
Privacy in blockchain systems has largely been framed as the problem of hiding who participates in a transaction and what values are transferred. This framing has led to the development of mixers, confidential transactions, and zero-knowledge proofs that effectively obfuscate identity-level linkages.
However, public blockchains are inherently transparent with respect to time. Even when sender identities and transferred values are hidden, the temporal relationship between protocol actions—such as deposits and withdrawals—remains visible to all observers.
Empirical evidence from chain analytics shows that this temporal metadata can be exploited to correlate otherwise unlinkable events. In practice, timing-based inference often dominates cryptographic protections, particularly when anonymity sets are small or user behavior is predictable.
This observation exposes a fundamental gap in existing threat models: identity privacy does not imply temporal privacy.</li>
</ol>

<p>1.2 Limitations of Existing Approaches<br>
Most blockchain protocols implement time-dependent logic using publicly observable signals such as block timestamps or block height. These mechanisms exhibit several inherent weaknesses:<br>
    • unlock conditions are deterministic and globally observable;<br>
    • timing behavior can be statistically modeled;<br>
    • delay mechanisms are often voluntary rather than enforced;<br>
    • validators may influence timestamps within protocol bounds.<br>
Privacy protocols that employ zero-knowledge proofs successfully hide transaction participants but typically allow users to choose when to execute a withdrawal. This user-controlled timing introduces a side channel that adversaries can exploit to reduce anonymity sets and perform probabilistic deanonymization.<br>
These limitations are architectural rather than incidental.</p>

<p>1.3 Time as a First-Class Security Primitive<br>
This work argues that time must be treated as a first-class security primitive, subject to explicit protocol enforcement. A privacy-preserving system should ensure not only that actions are unlinkable by identity, but also that they are unlinkable by temporal correlation.<br>
To address this gap, we propose Time-Decoupled Law (TDSM): a protocol abstraction in which execution eligibility is governed by an internal, immutable time mechanism rather than by publicly observable block metadata or external oracles.<br>
In TDSM, time functions as an enforceable law rather than as informational metadata.</p>

<p>1.4 Contributions<br>
This paper makes the following contributions:<br>
    1. Identification of timing correlation as a fundamental privacy attack vector.<br>
    2. Introduction of Time-Decoupled Law (TDSM) as a new protocol primitive.<br>
    3. Formalization of temporal unlinkability.<br>
    4. Threat-model comparison between existing privacy systems and TDSM.<br>
    5. Demonstration of practical applications enabled by enforced temporal privacy.</p>

<p>1.5 Organization<br>
    • Section 2: Threat Model<br>
    • Section 3: Related Work<br>
    • Section 4: Time-Decoupled Law Model<br>
    • Section 5: Security Analysis<br>
    • Section 6: Applications<br>
    • Section 7: Conclusion and Future Work</p>

<ol>
<li><p>Threat Model<br>
We consider a global passive adversary with the following capabilities:<br>
• full access to on-chain data;<br>
• ability to observe all transactions, events, and state transitions;<br>
• ability to perform statistical, graph-based, and Bayesian inference;<br>
• no ability to break cryptographic primitives.<br>
The adversary’s objective is to link protocol actions (e.g., deposits and withdrawals) belonging to the same user using timing information.<br>
We explicitly exclude:<br>
• cryptographic breaks;<br>
• censorship or transaction suppression;<br>
• compromised client devices.</p></li>
<li><p>Related Work<br>
3.1 Timestamp-Based Time Locks<br>
Traditional smart contracts rely on block.timestamp or block height for time-based logic. These mechanisms are deterministic, public, and vulnerable to manipulation within protocol bounds, making them unsuitable for privacy-sensitive use cases.</p></li>
</ol>

<p>3.2 Governance Timelocks<br>
DAO timelocks introduce mandatory delays between proposal approval and execution. While effective for governance safety, they expose public execution windows and often include administrative override mechanisms, offering no temporal privacy guarantees.</p>

<p>3.3 Privacy Mixers and zk-Based Protocols<br>
Zero-knowledge mixers decouple sender and receiver identities but leave withdrawal timing under user control. As a result, they remain vulnerable to timing correlation attacks, particularly under low-volume conditions.</p>

<p>3.4 Relayer-Based Systems<br>
Relayers improve sender privacy and UX but do not conceal execution time. In some cases, relayers introduce additional correlation surfaces.</p>

<p>3.5 Oracle-Based Delays<br>
Off-chain time verification introduces trust assumptions, censorship risks, and oracle manipulation vectors, violating the principles of permissionless execution.</p>

<ol>
<li>Time-Decoupled Law Model
4.1 Design Principles
TDSM is defined by the following properties:
• Timestamp independence
• Protocol-enforced delay
• Non-public execution eligibility
• No administrative override
• Deterministic state transitions</li>
</ol>

<p>4.2 Core Abstraction<br>
TDSM maintains an internal state machine that advances through discrete time states (“ticks”). Execution of protected actions is permitted only after successful verification of the required tick state.<br>
Crucially, the mapping between ticks and wall-clock time is not publicly observable.</p>

<p>4.3 Enforced Delay<br>
Unlike voluntary waiting, enforced delay ensures that:<br>
    • early execution is impossible;<br>
    • execution cannot be accelerated;<br>
    • the unlock condition cannot be inferred from on-chain metadata.</p>

<ol>
<li>Security Analysis
5.1 Timing Correlation Attack
In systems without enforced temporal privacy, an adversary can correlate deposit and withdrawal events by analyzing time gaps. This attack does not require breaking cryptography.</li>
</ol>

<p>5.2 Temporal Unlinkability<br>
We define temporal unlinkability as the inability of an adversary to link protocol actions using temporal information with probability significantly greater than random guessing.<br>
TDSM achieves temporal unlinkability by removing observable timing thresholds.</p>

<ol>
<li><p>Applications<br>
TDSM enables or strengthens the following classes of systems:<br>
• privacy mixers and vaults;<br>
• trustless escrow and settlement;<br>
• DAO governance with objective cooldown laws;<br>
• vesting and token unlock mechanisms;<br>
• anti-MEV execution buffers;<br>
• cross-chain security delays.</p></li>
<li><p>Conclusion and Future Work<br>
7.1 Conclusion<br>
This work demonstrates that temporal information constitutes a critical and under-addressed side channel in blockchain privacy systems. Existing protocols, even when employing advanced cryptographic techniques, implicitly assume that timing metadata is harmless.<br>
Time-Decoupled Law (TDSM) challenges this assumption by introducing a protocol primitive in which time is treated as an enforceable law rather than a public parameter. By eliminating deterministic and observable execution thresholds, TDSM provides temporal unlinkability, extending privacy guarantees beyond identity and value concealment.</p></li>
</ol>

<p>7.2 Future Work<br>
Future research directions include:<br>
    • formal proofs of temporal unlinkability;<br>
    • integration with zero-knowledge circuits;<br>
    • stochastic and adaptive time models;<br>
    • cross-chain and rollup applications;<br>
    • tooling for verification and benchmarking.</p>

<p>Closing Remark<br>
By elevating time to a first-class security primitive, Time-Decoupled Law opens a new design space for privacy-preserving decentralized systems, where when an action occurs is no longer exploitable metadata, but a property governed by immutable protocol law.</p>

<p>Repository<br>
The reference implementation of Time-Decoupled Law (TDSM) is publicly available and open source:<br>
GitHub:<br>
<a href="https://github.com/Abstract36/Time_Decoupled_Law" rel="noopener noreferrer">https://github.com/Abstract36/Time_Decoupled_Law</a></p>

