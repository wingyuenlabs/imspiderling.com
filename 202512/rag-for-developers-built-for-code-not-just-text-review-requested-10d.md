---
Title: RAG for Developers — Built for Code, Not Just Text (Review Requested)
Description: 
Author: Demon Slayer
Date: 2025-12-13T21:42:27.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  <strong><em>Code-Aware RAG Tool  — Looking for Developer Feedback</em></strong>
</h2>

<p>We’ve been building a code-first RAG tool that actually understands how codebases work, not just how text looks in embeddings. The goal is simple: when you ask a question, you get the right functions, related calls, and supporting code, not random nearby snippets.</p>

<p>What’s Inside</p>

<p>AST-based code chunking with Tree-sitter (Python, JavaScript, TypeScript)<br>
Functions, classes, imports, calls, docstrings — all extracted explicitly</p>

<p>A clean async ingestion pipeline with strict tool → agent → storage boundaries</p>

<p>Semantic vector search as the starting point, not the end</p>

<p>In-memory dependency graph expansion:</p>

<p>Built lazily from chunk metadata</p>

<p>No persistence, no globals, no backend shortcuts</p>

<p>Stable qualified IDs (file::entity)</p>

<p>Context expansion via BFS over calls and imports to pull in code that’s actually connected</p>

<p>Backend-agnostic vector store layer, so storage can change without rewriting logic</p>

<p>Why We Think This Is Useful</p>

<p>You get related code paths, not just similar text</p>

<p>Context stays small, relevant, and debuggable</p>

<p>The architecture avoids hidden state and scaling surprises</p>

<p>What We’d Love Feedback On</p>

<p>If you’ve worked with large repos or built RAG systems before, we’d really value your thoughts on:</p>

<p>The “graph as derived state” design</p>

<p>Chunk metadata choices (calls, imports, QIDs)</p>

<p>Retrieval + expansion flow</p>

<p>Any edge cases you think would show up in real production codebases</p>

<p>Even quick reactions or gut checks are welcome.</p>

