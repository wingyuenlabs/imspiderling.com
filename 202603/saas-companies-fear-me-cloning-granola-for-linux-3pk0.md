---
Title: SaaS Companies Fear Me: Cloning* Granola for Linux
Description: 
Author: Ryan Swift
Date: 2026-03-03T21:23:35.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/mlh/built-with-google-gemini-02-25-26">Built with Google Gemini: Writing Challenge</a></em></p>

<p>I've been spending a lot of time on Twitter recently because it seems to be one of the best places to get AI news. A really common refrain I see on Twitter is about how the "end" of SaaS (Software as a Service) companies is approaching now that vibe coding enables folks to just build their own customized SaaS tools. I don't know that I fully agree with that take, but I suppose I am the hypothetical person in those tweets. There was a SaaS tool I loved that I couldn't use on Linux, so I just built my own (albeit notably worse) version of it.</p>

<p>I, unfortunately, find myself in back-to-back meetings most days. I've played around with a few different note-taking tools. About half a year ago, I discovered <a href="https://www.granola.ai/" rel="noopener noreferrer">Granola</a>. I really really really liked it. It did transcriptions. It enhanced your existing notes. I could run it without inviting another note taker to the meeting. It had a nice little chat interface where I could ask questions about my meetings. 10/10 would (and still do) recommend.</p>

<p>A month or so after discovering Granola, I decided I wanted to switch back to using Linux as my primary OS though. Windows was an uphill battle with CLI coding tools like Gemini CLI, OpenCode, and Claude Code (at least in my experience at the time).</p>

<p>Unfortunately, Granola didn't support Linux and didn't have plans on adding support for it (I did ask).</p>

<p>So, I built <a href="https://github.com/thisisryanswift/quinoa" rel="noopener noreferrer">Quinoa</a>. A note-taking tool for me. </p>

<h2>
  
  
  What I Learned
</h2>

<p>I think it's <strong>very important</strong> to call out again this is a tool for me. While I do have the code up on GitHub, I regularly tell my coding agents that they're building for me. My systems. My configs. My preferences. I've had a few design choices where a coding agent raises a concern that might impact other users, but I have very intentionally been designing this tool for myself. This has been a big learning for me in this project. </p>

<p>Coding agents allow us to will software into existence in a way we couldn't a few years ago. It feels like it's okay to be a bit less intentional about certain things. It's okay to build stuff just for you and just for your friends. Not every project needs to be perfectly engineered. If you hit a bug, you can just ask an agent to resolve it. </p>

<p>I made Quinoa almost exclusively using Gemini CLI and OpenCode (using Google, Anthropic, OpenAI, and a bunch of other models). I'm sure some folks will cringe at this, but I try not to write any code myself for this project. That does absolutely lead to some mistakes, but it's been a great way to learn how to harness and direct coding agents. Most of my code review is fully agent-driven as well. </p>

<h2>
  
  
  What I Built with Google Gemini
</h2>

<p>Quinoa itself fully uses Google Gemini models under the hood. I deliberately used it as a project to experiment with new Gemini releases. </p>

<p>The Gemini API handles all of the following.</p>

<p><strong>Meeting transcription.</strong> Gemini handles identifying folks' names and labeling them for the transcript. <a href="https://ai.google.dev/gemini-api/docs/audio" rel="noopener noreferrer">Audio understanding docs are here</a>.</p>

<p><strong>Meeting summaries.</strong> Two- to three-sentence summaries of what happened and action item extractions.</p>

<p><strong>Notes enhancement.</strong> Takes my notes from the app and then enhances them with additional context and information from the audio recording and transcript.</p>

<p><strong>RAG notes search.</strong> This was a huge time saver for me. Google literally has RAG as a service. Allows me to search notes and cites where it's pulling information from. Drop-in API that did exactly what I needed to do. <a href="https://ai.google.dev/gemini-api/docs/file-search" rel="noopener noreferrer">File search docs are here</a>.</p>

<p>I also used <a href="https://ai.google.dev/gemini-api/docs/image-generation" rel="noopener noreferrer">Nano Banana</a> to make my project's icon too :)</p>

<h2>
  
  
  Demo
</h2>

<p>My note-taker is a local application, so I can't embed a demo for y'all. Here's a sample screenshot of the app though. I'm clearly not a designer. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5c9tlxwfg0stqa7mmfsm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5c9tlxwfg0stqa7mmfsm.png" alt="Showing today view, asking the assistant a question, and another transcript" width="800" height="649"></a></p>

<h2>
  
  
  Google Gemini Feedback
</h2>

<p>The biggest issue across all coding agents currently seems to be getting the right context, especially if you're building tools that use new libraries. Google Gemini struggles to implement itself sometimes. I heavily recommend using the <a href="https://github.com/google-gemini/gemini-skills" rel="noopener noreferrer">Gemini API Dev</a> skill if you haven't tried it out yet. </p>

<p><em>Just wanted to quickly mention that I'm not entering this contest to win. I am pretty sure I'm ineligible since I work for MLH, but wanted to share something I've been working on!</em></p>

