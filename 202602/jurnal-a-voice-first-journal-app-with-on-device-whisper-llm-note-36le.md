---
Title: Jurnal: A Voice-First Journal App with On-Device Whisper + LLM Note
Description: 
Author: pizidavi
Date: 2026-02-15T21:18:42.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/github-2026-01-21">GitHub Copilot CLI Challenge</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>Jurnal is a simple mobile journaling app that makes it easy to capture "what happened today" using your voice. Instead of starting from a blank page, you record a short voice note, and the app turns it into a clean, structured journal entry you can skim later.</p>

<blockquote>
<p><code>Jurnal</code> is a <em>Romanian</em> term that translates into English as <em>diary</em> or <em>journal</em>.</p>
</blockquote>

<p>The core flow is:</p>

<ul>
<li>Record audio in-app</li>
<li>Transcribe locally on-device using a Whisper model </li>
<li>Send the transcription to a larger LLM to turn it into a real note, with sections like summary/highlights/intentions in Mardown</li>
</ul>

<p>What I like about this approach is the combination of privacy-first transcription with the "writing assistant" part happening after: I still speak naturally, and the app handles organizing the content into something readable.</p>

<p>Teck stack:</p>

<ul>
<li>React Native with Expo</li>
<li>
<code>react-native-executorch</code> to handle local transcription model</li>
<li>
<code>@tanstack/ai</code> to provide online LLM utility</li>
<li>TailwindCSS by Uniwind for styling</li>
</ul>

<p>Future improvements I'm excited about:</p>

<ol>
<li>Custom templates for different styles</li>
<li>Choosing custom models for transcription and enrichment</li>
<li>Manual editing of the final note in-app</li>
<li>Smarter search with local <strong>RAG</strong>
</li>
<li>A background queue for long-running processing</li>
</ol>

<h2>
  
  
  Demo
</h2>

<p>🔗 Repo: <a href="https://github.com/pizidavi/jurnal-app" rel="noopener noreferrer">https://github.com/pizidavi/jurnal-app</a></p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Home</th>
<th>Note</th>
<th>Recording</th>
<th>Settings</th>
</tr>
</thead>
<tbody>
<tr>
<td><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcz8ebp3czyt3zzx445jp.jpg" alt="Home screen" width="591" height="1280"></td>
<td><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmxfkui33xnbu950cm9xz.jpg" alt="Recording new note" width="591" height="1280"></td>
<td><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fey8t58jk4uyx6vvpu9t5.jpg" alt="Note screen" width="591" height="1280"></td>
<td><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftzwu5vzs7sykxbqhcrvh.jpg" alt="Setting screen" width="591" height="1280"></td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  My Experience with GitHub Copilot CLI
</h2>

<p>Copilot CLI was most helpful when I treated planning as a first-class step, not a formality. The biggest productivity boost came from writing a tight plan, implementing in small slices, and then letting the AI validate (and correct) itself.</p>

<p>To support that workflow, I made the project intentionally strict: ESLint rules are restrictive, TypeScript is in strict mode with other custom strict types, and I pushed the assistant to run <code>lint</code> and <code>typecheck</code> command before considering anything as done. That created fast feedback loops and prevents subtle regressions from piling up.</p>

<p>I also built a custom <strong>Orchestrator</strong> agent that follows a <code>plan → implement → review</code> cycle and repeats until the plan is completed. It was really fun to see the agents correct themselves when they missed a step or introduced a bug.</p>

<p>I'm extremely positive about the future of Copilot CLI: the UI is good and the interaction model fits real development.<br>
One thing I'd love to see next is a persistent <strong>allowlist for safe commands</strong> (like lint/typecheck/format), so that I don't have to approve them every time.</p>

<p>Have fun!</p>

