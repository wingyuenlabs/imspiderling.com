---
Title: Formal Semantics as the Missing Control Layer for AI-Assisted Software Engineering
Description: 
Author: Sven A. Schäfer
Date: 2026-01-06T22:05:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>Recent research from 2025 increasingly points in a common direction:<br>
AI alone is not the paradigm shift in software engineering — formal semantics is.</p>

<p>Across requirements engineering, formal methods, program verification, and AI-native tooling, a shared pattern emerges. Large language models can significantly assist software construction only when meaning is made explicit, structured, and formally constrained. Without such constraints, AI-driven development struggles with reproducibility, explainability, and long-term maintainability.</p>

<p>This article synthesizes recent peer-reviewed papers, empirical studies, and formal-methods research to show how explicit semantics, formal specifications, and deterministic acceptance mechanisms together form the foundation for a more reliable, scalable form of AI-assisted software engineering. While no single work proposes a complete end-to-end paradigm, the individual building blocks are now clearly visible.</p>

<blockquote>
<p>Requirements Are Not Documentation — They Are Control Structures</p>
</blockquote>

<p>A central shift in the 2025 literature is the reframing of requirements. Rather than being treated as descriptive artifacts, requirements are increasingly understood as structuring and controlling inputs for AI systems.</p>

<p>Ferrari and Spoletini’s two-way roadmap on formal requirements engineering and large language models positions formal requirements as a means to improve controllability, explainability, and verifiability when using LLMs in software development. Their work argues that formalized requirements provide a structural interface between human intent and machine generation.</p>

<p>This position is empirically reinforced by Caporusso and Perdue’s ISCAP 2025 study. They demonstrate that a requirements-first workflow, in which structured specifications are generated before invoking LLMs for code, leads to measurably higher quality outputs compared to direct prompt-to-code approaches.</p>

<p>Complementing this, Lu et al.’s multi-agent vision for requirements development and formalization proposes a system where requirements are actively refined and formalized before downstream code generation. While primarily conceptual, the work underscores that “requirements → formalization” is being treated as a first-class quality lever.</p>

<blockquote>
<p>The emerging pattern is not AI versus formal methods, but AI embedded within formal control.</p>
</blockquote>

<p>Another consistent theme in 2025 research is a clear division of responsibility between AI systems and formal mechanisms.</p>

<p>AI excels at proposing candidate artifacts — contracts, postconditions, annotations, or documentation. However, the decision about correctness, validity, or acceptance must remain deterministic and formal.</p>

<p>Teuber and Beckert illustrate this separation in their work on LLM-supported Java verification. In their setup, LLMs generate candidate JML specifications, while deductive verification acts as an acceptance gate within the evaluated scope.</p>

<p>Related work by Zhang, Wang, and Zhai shows that even relatively small models can infer meaningful postconditions. While this research focuses on feasibility rather than full formal verification, it supports the broader claim that LLMs can produce formal specification elements suitable for downstream checking.</p>

<p>Recio Abad and colleagues extend this idea by using formal JML specifications as context to improve LLM-generated Java documentation. Their results show that formal inputs discipline AI outputs, even when the generated artifacts themselves are not formal proofs.</p>

<p>Across these works, a consistent structure appears:<br>
AI proposes — formal mechanisms decide.</p>

<blockquote>
<p>Semantic consistency is a system-level property, maintained through explicit semantic models and deterministic validation mechanisms.</p>
</blockquote>

<p>Beyond individual artifacts, several 2025 contributions address semantic consistency at the system level.</p>

<p>In model-based systems engineering, Cibrián et al. demonstrate that semantic constraints can be explicitly encoded at the metamodel level and automatically validated across complex SysML v2 model landscapes.</p>

<p>Parallel insights emerge from the AI-native compiler discourse. Cogo, Oliva, and Hassan identify reproducibility and interoperability as core unresolved challenges when translating high-level intent into executable systems.</p>

<p>A complementary perspective comes from outside software engineering: the 2025 Think7 policy work on Rules as Code. In governance contexts, intent and regulation are increasingly transformed into machine-executable representations to improve transparency, traceability, and consistency.</p>

<blockquote>
<p>Purely intent-driven approaches tend to encounter structural limits once systems grow in complexity.</p>
</blockquote>

<p>The contrast becomes especially clear when examining purely conversational programming paradigms.<br>
Become a member</p>

<p>Bamil’s work on vibe coding explicitly acknowledges challenges related to reproducibility, maintainability, and explainability when code is produced primarily through conversational interaction.</p>

<p>Empirical work by Sarkar and Drosos further shows that developers using conversational programming tools must rely heavily on iterative inspection and manual verification to build trust. Similarly, Sapkota et al. distinguish vibe coding from more agentic approaches and emphasize that lack of formal acceptance criteria remains a core limitation.</p>

<p>Taken together, the current body of work supports several carefully bounded conclusions:</p>

<ul>
<li>Explicit, formal semantics is a central enabler for scalable and reliable AI-assisted software engineering.</li>
<li>LLMs are well-suited to propose formal artifacts, with formal validation providing the final grounding.</li>
<li>Deterministic acceptance mechanisms are technically feasible and already demonstrated in specific domains.</li>
<li>Semantic consistency can be enforced at the system level, provided that meaning is explicitly modeled.</li>
<li>No existing work covers the full paradigm end-to-end, but the necessary formal components are clearly identifiable.</li>
</ul>

<p>The emerging picture from 2025 research is not one of automation replacing discipline, but of discipline enabling automation.</p>

<p>AI can accelerate software construction — but only when embedded in a framework of explicit semantics, formal specifications, and deterministic acceptance. This perspective aligns with applied efforts such as <a href="https://secos.rocks" rel="noopener noreferrer">Secos Rocks</a>, which investigates semantics-first and contract-driven approaches to AI-supported software construction.</p>

<p>The future of AI-assisted software engineering is therefore not prompt-driven improvisation, but semantics-centered construction.</p>

<h2>
  
  
  References
</h2>

<p>Ferrari, A.; Spoletini, P. (2025).<br>
Formal requirements engineering and large language models: A two-way roadmap.<br>
<a href="https://www.sciencedirect.com/science/article/pii/S0950584925000369" rel="noopener noreferrer">Information and Software Technology Volume 181, May 2025, 107697.</a></p>

<p>Caporusso, N.; Perdue, J. (2025).<br>
Preserving Requirements-Driven Development in the Age of AI: An Empirical Evaluation of the Impact of Software Specifications in LLM-Assisted Coding.<br>
<a href="https://iscap.us/proceedings/2025/pdf/6416.pdf" rel="noopener noreferrer">Proceedings of ISCAP 2025.</a></p>

<p>Lu, Y.; et al. (2025).<br>
Requirements Development and Formalization for Reliable Code Generation: A Multi-Agent Vision.<br>
<a href="https://arxiv.org/abs/2508.18675" rel="noopener noreferrer">arXiv:2508.18675.</a></p>

<p>Teuber, S.; Beckert, B. (2025).<br>
Next Steps in LLM-Supported Java Verification.<br>
<a href="https://arxiv.org/abs/2502.01573" rel="noopener noreferrer">arXiv:2502.01573.</a></p>

<p>Zhang, G.; Wang, Z.; Zhai, J. (2025).<br>
Breaking the Myth: Can Small Models Infer Postconditions Too?<br>
<a href="https://arxiv.org/abs/2507.10182" rel="noopener noreferrer">arXiv:2507.10182.</a></p>

<p>Recio Abad, J.; et al. (2025).<br>
Formal Methods Meets Readability: Auto-Documenting JML Java Code.<br>
<a href="https://arxiv.org/abs/2506.09230" rel="noopener noreferrer">arXiv:2506.09230.</a></p>

<p>Cibrián Sánchez, E.; et al. (2025).<br>
Ensuring Semantic Consistency in SysML v2 Models Through Metamodel-Driven Validation.<br>
<a href="https://ieeexplore.ieee.org/document/11077130" rel="noopener noreferrer">IEEE Access 2025.3587786.</a></p>

<p>Cogo, F. R.; Oliva, G. A.; Hassan, A. E. (2025).<br>
Compiler.next: A Search-Based Compiler to Power the AI-Native Future of Software Engineering.<br>
<a href="https://arxiv.org/abs/2510.24799" rel="noopener noreferrer">arXiv:2510.24799.</a></p>

<p>Rapson, J.; et al. (2025).<br>
Rules as Code for a More Transparent and Efficient Global Economy.<br>
<a href="https://www.think7.org/publications/rules-as-code-for-a-more-transparent-and-efficient-global-economy/" rel="noopener noreferrer">Think7 (G7) Policy Brief.</a></p>

<p>Bamil, V. (2025).<br>
Vibe Coding: Toward an AI-Native Paradigm for Semantic and Intent-Driven Programming.<br>
<a href="https://arxiv.org/abs/2510.17842" rel="noopener noreferrer">arXiv:2510.17842.</a></p>

<p>Sarkar, A.; Drosos, I. (2025).<br>
Vibe Coding: Programming Through Conversation with Artificial Intelligence.<br>
<a href="https://arxiv.org/abs/2506.23253" rel="noopener noreferrer">arXiv:2506.23253.</a></p>

<p>Sapkota, S.; et al. (2025).<br>
Vibe Coding vs. Agentic Coding: Fundamentals and Practical Implications of Agentic AI.<br>
<a href="https://arxiv.org/abs/2505.19443" rel="noopener noreferrer">arXiv:2505.19443.</a></p>

