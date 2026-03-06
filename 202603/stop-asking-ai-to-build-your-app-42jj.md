---
Title: Stop asking AI to "build your app"
Description: 
Author: Alp Yalay
Date: 2026-03-06T21:24:15.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://github.com/KhazP/vibe-coding-prompt-template" rel="noopener noreferrer">Repo URL</a><br>
<a href="https://vibeworkflow.app" rel="noopener noreferrer">Website Version of the Repo</a></p>

<p>I think a lot of people are using AI coding tools in the worst possible order.</p>

<p>They start with code.</p>

<p>They open a chat, dump a half-formed idea into it, and ask for an MVP. The model happily gives them folders, components, a few routes, maybe even auth if it's feeling ambitious. For about twenty minutes, it looks like progress.</p>

<p>Then reality shows up.</p>

<p>The app has no real shape. The feature boundaries are fuzzy. The stack choices are accidental. Every follow-up prompt quietly changes the project a little more. By the third or fourth round, you're not building anymore. You're negotiating with a machine that keeps making assumptions on your behalf.</p>

<p>I've run into that enough times that I stopped doing it.</p>

<p>The workflow I use now is much less exciting, which is probably why it works better. Before I let an agent touch code, I make it go through the same sequence every time:</p>

<p>Research the idea<br>
Write the MVP requirements<br>
Make the technical plan<br>
Turn that into agent instructions<br>
Start building<br>
That's basically what this repo is for. It's a prompt template, but the useful part isn't the prompts by themselves. It's the order.</p>

<p>The first three steps force you to figure out what you're actually making. Not in the fake startup way where everything is "an AI-powered platform," but in plain terms. Who is this for? What does version one need to do? What can wait? What stack makes sense if you're optimizing for speed, cost, or maintainability?</p>

<p>That work matters more than people want it to.</p>

<p>A lot of the bad AI code I see is not bad because the model is uniquely dumb. It's bad because it inherited chaos. If the product definition is mushy, the implementation will be mushy too. The tool is just making the ambiguity visible.</p>

<p>Once the planning is written down, the coding part gets noticeably better. The agent has constraints. It knows what it's aiming at. It can break work into phases without drifting into random feature land. You still need to review what it does, obviously, but at least now you're reviewing a plan instead of reading tea leaves.</p>

<p>The other thing this workflow fixes is context decay.</p>

<p>Anybody who spends time with coding agents knows the feeling. A session starts strong, then gets weird. The model forgets earlier decisions. It reintroduces things you already removed. It starts answering the current prompt while quietly ignoring the project you were building an hour ago.</p>

<p>That gets much less painful when the important thinking lives in actual project docs instead of one huge chat thread. You can switch tools, compact context, start a fresh session, whatever. The project still has a memory outside the model's head.</p>

<p>That's probably my biggest takeaway after using this stuff for real projects: the artifact matters more than the conversation.</p>

<p>The chat feels productive because it's fast and fluent. But the docs, the constraints, the explicit instructions, those are the parts that keep the project from turning into mush.</p>

<p>I also don't think this workflow is only for experienced developers. If anything, beginners benefit more from it. When you're new, it's very easy to mistake generated volume for momentum. A repo full of files can feel like proof that something serious is happening. Sometimes it is. Sometimes it's just autocomplete with better marketing.</p>

<p>A simple structure helps. Research first. Scope second. Design third. Then hand the build to the agent in pieces.</p>

<p>Not one giant "build my startup" prompt. That's where people get themselves into trouble.</p>

<p>Anyway, that's the pitch.</p>

<p>If you're using AI to build side projects, try making the machine inherit your thinking instead of asking it to supply the thinking for you. It's less flashy, but the result is usually something you can still work with next week.</p>

