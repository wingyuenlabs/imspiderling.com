---
Title: I built a WASM execution firewall for AI agents — here’s why
Description: 
Author: Xnfinite 
Date: 2026-01-10T21:17:28.000Z
Robots: noindex,nofollow
Template: index
---
<p>I’ve been spending a lot of time lately experimenting with autonomous AI agents doing real work—writing code, running tools, coordinating steps. The kind of thing that goes beyond chat and starts touching real systems.</p>

<p>One thing that keeps bothering me is what happens when those agents start generating or modifying code that runs on its own. Not theoretical. Not a hallucination. Just actual code—specifically WASM modules—that gets compiled and executed without human review.</p>

<p>That’s where the idea for Night Core came from.</p>

<p>⸻</p>

<p>What I’m building</p>

<p>Night Core is a console for controlling execution of WebAssembly modules, especially when the code comes from agents, remote systems, or any source that isn’t fully trusted.</p>

<p>Before anything runs, it applies a few basic rules:<br>
    • Signature verification (Ed25519)<br>
    • Optional human approval before execution<br>
    • Logging and audit trail<br>
    • Sandboxed execution using Wasmtime</p>

<p>The architecture separates a worker that enforces policy from a UI that helps with approvals and visibility. It’s built in Rust with a Tauri frontend and TypeScript for the UI.</p>

<p>The code’s open here:<br>
<a href="https://github.com/xnfinite/nightcoreapp%EF%BF%BC" rel="noopener noreferrer">https://github.com/xnfinite/nightcoreapp￼</a></p>

<p>⸻</p>

<p>Why this matters (to me)</p>

<p>Most of the agent discussion I see is about whether the output is correct. But I’m more interested in what happens when that output becomes an action—especially code execution.</p>

<p>Once something runs, you’re already in response mode. Logs, alerts, and sandboxes are helpful, but they’re all after the fact. That’s what pushed me to treat execution itself as the boundary.</p>

<p>⸻</p>

<p>The threat model is simple<br>
    • Agents can generate or modify WASM<br>
    • The code might be functional but not necessarily safe<br>
    • Sandboxing limits damage, but doesn’t eliminate risk<br>
    • Repetition without review increases that risk</p>

<p>⸻</p>

<p>Still early, but here’s what I’m wondering<br>
    • Should AI-generated code always require a human review before it runs?<br>
    • Where is that too slow to be useful?<br>
    • How do we reason about trust and intent in generated artifacts?</p>

<p>This isn’t finished. It’s a working sketch. But I’d like to hear how others are thinking about this, or if you’re seeing the same edge cases.</p>

<p>You can see the full thread and screenshots here: <a href="https://x.com/xnfinitecore/status/2010067157774991457?s=46" rel="noopener noreferrer">https://x.com/xnfinitecore/status/2010067157774991457?s=46</a> </p>

<p>GitHub: <a href="https://github.com/xnfinite/nightcoreapp%EF%BF%BC" rel="noopener noreferrer">https://github.com/xnfinite/nightcoreapp￼</a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftib86zyeek3okmcjg8ud.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftib86zyeek3okmcjg8ud.png" alt="nightcore console" width="800" height="429"></a><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpj1fo6qq2cekwb3h4ewc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpj1fo6qq2cekwb3h4ewc.png" alt="Night Core console showing a pending WASM execution request" width="800" height="430"></a></p>

