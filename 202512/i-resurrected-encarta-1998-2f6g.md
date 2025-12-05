---
Title: I Resurrected Encarta 1998
Description: 
Author: omu inetimi
Date: 2025-12-05T21:48:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>I built a resurrection of Microsoft Encarta—the 1998 CD-ROM encyclopedia—for #Kiroween that finally delivers on what we all wished it could do: infinite articles on any topic, generated on-demand. The twist: it's powered by GPT-4o-mini, but forced to write in authentic 1990s academic encyclopedia tone. Neutral language, no contractions, and absolutely no post-1998 references.</p>

<p>Huge props to Kiro for helping me ship this in a weekend with its spec-driven development.</p>

<p>The History<br>
Encarta launched in 1993 and became the definitive multimedia encyclopedia of the CD-ROM era. That distinctive Windows 95 interface—beveled buttons, blue title bars, serif fonts on cream backgrounds—represented the cutting edge of educational technology. But it had a fundamental limitation: static content. The 1998 edition contained roughly 42,000 articles, and if your topic wasn't included, you were out of luck.</p>

<p>The combinatorial explosion of human knowledge made comprehensive coverage impossible. By 2009, Microsoft discontinued Encarta entirely, unable to compete with Wikipedia's crowdsourced model.</p>

<p>Nearly 30 years later, LLMs finally make the original vision possible—an encyclopedia that can generate authoritative articles on anything.</p>

<p>The Setup<br>
I wrote four spec directories before touching code: ui-layout/ (pixel-perfect Windows 95 requirements), article-generator/ (LLM content structure and tone), enhancement-layer/ (2025 augmentation design), and kiroween-mode/ (VHS horror effects). Each spec had three files: requirements.md with GIVEN/WHEN/THEN acceptance criteria, design.md with architecture and correctness properties, and tasks.md with implementation checklist.</p>

<p>The design docs defined formal correctness properties that became implementation constraints. Properties like "window MUST be exactly 800x600 pixels" and "all raised elements use white/dark border pattern" ensured the Windows 95 aesthetic stayed authentic.</p>

<p>Two steering documents enforced tone across all generations:</p>

<p>encyclopedia-tone.md: "1998 mode uses neutral, academic language without contractions or post-1998 references"<br>
ui-consistency.md: Exact color codes (#C0C0C0, #000080), font stacks (MS Sans Serif → Tahoma → Arial), and icon rules (emoji only, no modern vectors)<br>
What Changed<br>
Normally with AI coding assistants, you're constantly re-explaining context. With specs and steering, you explain once. When I asked to "fix the sidebar categories," Kiro already knew the Windows 95 color palette, the bevel border patterns, and that responsive CSS units weren't allowed. The changes stayed within constraints automatically.</p>

<p>The most impressive generation was the complete Kiroween mode. I described wanting "VHS glitch effects with haunted content" and got back:</p>

<p>CSS @keyframes for glitch animations with transform and text-shadow manipulation<br>
Scanline overlay using repeating-linear-gradient at 2-4px intervals<br>
Static noise using SVG feTurbulence filter at 2% opacity<br>
Content generation with [REDACTED] sections, haunted footnotes referencing "3:33 AM modifications by [USER UNKNOWN]," and eerie cross-references to "The Vanishing" and "[ENTRY NOT FOUND]"<br>
Dark purple/green palette (#1A1A2E, #00FF88, #FF0066) replacing the Windows 95 silver<br>
Four agent hooks automated validation on every save:</p>

<p>ui-consistency.json scans CSS for forbidden modern properties<br>
tonal-validation.json reviews LLM prompts for era-appropriate language<br>
cross-reference-check.json verifies all "See also" links generate valid articles<br>
retro-commits.json generates CD-ROM era commit messages ("Updated Knowledge Module: Volcanology 3.2")<br>
Three Modes<br>
Classic 1998 Mode: Neutral academic tone, 3-5 paragraphs, ASCII art diagrams, cross-references—exactly like the original Encarta</p>

<p>Enhanced Mode: Retro article plus a "2025 Expansion" panel with timelines, modern examples, and analogies bridging 1998 to present</p>

<p>Kiroween Spooky Mode: The "Corrupted Encyclopedia" speaks from a glitching dimension, with VHS flicker effects, [REDACTED] sections, haunted footnotes, and references to researchers who disappeared under mysterious circumstances</p>

<p>Takehome<br>
Spec-driven development isn't just documentation—it's persistent context that shapes every interaction. The correctness properties in my design docs meant I could iterate quickly on features while Kiro ensured the Windows 95 aesthetic stayed pixel-perfect. Steering docs eliminated repetitive prompting—I never had to say "use #C0C0C0 for backgrounds" twice.</p>

<p>The combination of spec-driven development for architecture and vibe coding for rapid iteration worked perfectly. Specs for structure, vibes for speed.</p>

<p>Agent hooks acted as automated code reviewers, catching drift from the retro aesthetic before it could accumulate. MCP configuration enabled future features like search indexing and topic graph visualization without leaving the Kiro workflow.</p>

<p>Check out the project: github.com/mizzleinetimi/encart-ai</p>

