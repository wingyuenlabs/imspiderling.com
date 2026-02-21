---
Title: Your AI Agent is Coding Against Fiction — How I Fixed Doc Drift with a Pre-Commit Hook
Description: 
Author: Russell Moss
Date: 2026-02-21T21:35:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you're vibe coding with Claude Code, Cursor, or Copilot, you've probably experienced this:</p>

<p>Start a project. Write a nice ARCHITECTURE.md.<br>
Feed it to your AI agent as context.<br>
Ship 20 commits.<br>
Realize half the API routes in your docs don't exist anymore.</p>

<p>Your agent is now making decisions based on documentation that's completely wrong. I call this doc drift, and it's the silent killer of AI-assisted development.<br>
The Problem<br>
AI coding agents are only as good as the context you give them. When your .cursorrules or CLAUDE.md reference an architecture doc that hasn't been updated in two weeks, every suggestion your agent makes is built on a foundation of lies.<br>
Manual doc maintenance doesn't scale — especially for solo devs who are moving fast.<br>
What I Built<br>
agent-guard is a zero-dependency CLI that creates a self-healing documentation layer for your project. It works in four layers:<br>
Standing Instructions — Writes rules into your AI agent config (.cursorrules, CLAUDE.md, etc.) telling the agent to update docs alongside code changes.<br>
Generated Inventories — Deterministic scripts extract truth directly from your source code. API routes, Prisma models, env vars — all auto-generated into markdown files that can't lie.<br>
Pre-commit Hook — Catches drift before it reaches your repo. If Claude Code is installed, it auto-updates your narrative docs. If not, it prints a ready-made prompt you can paste into your editor.<br>
CI/CD Audits — GitHub Actions that run on every push and PR, plus weekly scheduled health checks.<br>
Setup<br>
bashnpm install --save-dev <a class="mentioned-user" href="https://dev.to/mossrussell">@mossrussell</a>/agent-guard<br>
npx agent-guard init<br>
npx agent-guard sync<br>
The init wizard auto-detects your framework and sets everything up. The hook never blocks commits — it always exits cleanly.<br>
What It Looks Like in Practice<br>
When you commit with Claude Code available:<br>
✓ Doc-relevant changes detected — docs also updated. Nice!<br>
When Claude Code isn't available:<br>
⚠️  Documentation may need updating</p>

<p>Changed:<br>
  📡 API Routes (1 file):<br>
     - src/app/api/users/route.ts</p>

<p>┌─────────────────────────────────────┐<br>
│ Claude Code Prompt (copy-paste):     │<br>
└─────────────────────────────────────┘<br>
Why Zero Dependencies Matters<br>
For solo devs doing vibe coding, every dependency is a liability. agent-guard uses only Node.js built-ins. No runtime overhead, no supply chain risk, no version conflicts.<br>
Try It</p>

<p>npm: <a href="https://www.npmjs.com/package/@mossrussell/agent-guard" rel="noopener noreferrer">https://www.npmjs.com/package/@mossrussell/agent-guard</a><br>
GitHub: <a href="https://github.com/russellmoss/agent-guard" rel="noopener noreferrer">https://github.com/russellmoss/agent-guard</a></p>

<p>I'd love feedback — especially from other solo devs fighting doc drift. What's your current approach?</p>

