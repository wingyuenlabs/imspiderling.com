---
Title: With Gemini CLI, I'm able to keep my pet projects alive and kicking
Description: 
Author: Paulo Henrique
Date: 2026-03-03T21:54:39.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/mlh/built-with-google-gemini-02-25-26">Built with Google Gemini: Writing Challenge</a></em></p>

<h2>
  
  
  What I Built with Google Gemini
</h2>

<p>It's 2023, and I'm trying to figure out how LLM APIs actually work. I don't like the idea of creating a "ChatGPT clone" that just talks to the OpenAI API. There are thousands out there. So I built a rough Next.js site to merge AI and SEO in a simple way. Out with the long prompts; the site would just do the work for you with minimal inputs.</p>

<p>It's 2024, and I'm loving being part of the DEV.to community, writing for fun, coding for fun. During a Hackathon, I decided to build a text-based RPG for a DEV.to challenge using AI, so every adventure and interaction would be different. I had a dream of using AI to create infinite ideas like the old "Choose Your Own Adventure" books and games from the 80s and 90s. I love a good pun, so "Choose Your Own AIdventure" was born. Unfortunately, given some time constraints, the site was rough, and I didn't win the challenge.</p>

<p>Both projects worked as MVPs. Both won their respective little battles. And, like most pet projects, they sat on a server and aged.</p>

<p>Code gets outdated. APIs change. You receive some alerts from GitHub or Vercel about outdated libraries or build variables. But you still need to work, or deal with adult things. And your cool idea just sits there, unchanged, cursed to never be what you <em>know</em> it could be.</p>

<p>Enter the Gemini CLI. I was already using it professionally, but one day I just thought: "What if I asked Gemini to evaluate my code and minimally update it to 2026 standards?"</p>

<p>Done.</p>

<p>Simple as that. I just had to approve some commands, and minutes later <em>Choose your Own AIdventure</em> was alive and kicking again.</p>

<p>This is where I decided to take those two legacy projects and use the Gemini CLI to not only do a minor update, but to refactor them exactly as I envisioned them the first time. Here is exactly how that went.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9rh8771go3u8yf2nls2q.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9rh8771go3u8yf2nls2q.jpg" alt="The old and the new" width="800" height="749"></a></p>

<h2>
  
  
  What I Built
</h2>

<p>I'm by nature a terminal person. This is where I spend most of my day. I access remote servers, I build automations, I execute shell scripts, and I test things locally before they ever see a browser. So, using the Gemini CLI to execute code was natural for me. I keep it running on the second monitor and provide inputs when needed. On my main monitor, I focus on other things.</p>

<p>The first project I refactored was <em><a href="https://choose.phalkmin.me/" rel="noopener noreferrer">Choose Your Own AIdventure</a></em>. As I mentioned, originally it was supposed to be like the old solo RPG books, where you read part of a text and choose what to do based on limited options. Good choices led you to victory. Bad choices led to a bad ending.</p>

<p>The biggest upgrade here was fixing the layout. I still wanted it to look like an old videogame, so I asked Gemini to analyze the codebase and create a mockup of a new front-end. After a few rounds of manual changes and conversations with the tool, I asked the CLI to apply the final version. Then, I started a brainstorming session with Gemini to discuss ideas and how to implement them by priority and complexity.</p>

<p>After that, I asked it to create a markdown file with all the ideas and an execution plan. In my spare time, I could just boot the CLI and ask it to execute the plan from item X to item Y. Gemini was able to understand everything, ask questions, and even test the build to ensure that everything was in place.</p>

<p>For the <em><a href="https://seo.phalkmin.me/" rel="noopener noreferrer">SEO Toolbox</a></em>, the Gemini CLI basically refactored the whole codebase, making it smaller, faster, and packed with new features. Originally, I only had two features: create some blog post ideas, and write a text about X. Now, it <em>feels</em> like a true toolbox, with half a dozen options, like Competitor Analysis, Meta Tag Generation, and more.</p>

<h2>
  
  
  What I Learned
</h2>

<p>Years ago, I studied to be a teacher. Going through the Brazilian Magistério program and studying Paulo Freire, you learn very quickly that you don't teach by just handing over the final answer or by assuming that everyone is on the same page as you. You need to know how to let the student learn on their own terms, and guide the interaction with the subject.</p>

<p>Prompting an AI that has almost infinite access to your files feels exactly like that.</p>

<p>I had to unlearn the way I prompted before. This meant giving clear instructions on when to act and when to just review everything, and also reviewing Gemini's decisions before approval. Giving a bad prompt meant that Gemini would overthink the solution or act on its own, modifying things I didn't ask for. Because I had limited time for these personal projects, I had to create almost perfect prompts and ensure my <code>GEMINI.md</code> rules were on point.</p>

<p>My skills evolved, too, as I learned that talking to the web version of a model is entirely different from talking to the agentic version.</p>

<p>Also, I usually have thousands of ideas in my head, so even minor things can get lost if I'm not paying attention. One night, after forgetting to ask Gemini to save the changes we had made in the <code>Changelog.txt</code> and update the <code>PLAN.md</code> file, I started a new session to ask if I could create a "macro" where just one phrase would make it execute a series of steps. It automatically added the "That's all, folks!" command to the <code>GEMINI.md</code> file.</p>

<p>Now, I just need to type that, and it does everything for me.</p>

<h2>
  
  
  Google Gemini Feedback
</h2>

<p>Building on top of a language model requires trial and error. And sometimes an error can be fatal.</p>

<p>The Gemini CLI is fast. Testing prompts and streaming responses without a frontend is a massive productivity boost. You can type your prompt and let the AI take care of it while you work on other things, or try to give some attention to your cats.</p>

<p>But Gemini has a distinct reasoning style. It tends to be overly helpful and act without authorization if it feels like it. So, you need to set strict boundaries.</p>

<p>Also, as the session gets longer and the codebase grows, Gemini can hallucinate. Nothing critical, but it might add code that isn't needed or enter a loop if for some reason the build won't finish.</p>

<p>I like to think of the Gemini CLI as a Junior Dev. He can make your life a lot easier. But he can also make your life a nightmare. That depends entirely on how you manage him.</p>

<h2>
  
  
  Looking Forward
</h2>

<p>Having pet projects can be a problem when you still need to work, deal with adult life, and feed two cats. This can be even more problematic for neurodivergent brains like mine, where that cool idea <em>never</em> leaves your mind.</p>

<p>Revisiting these projects in the terminal proved to me that legacy code can be salvaged if you have the right tools to pry it open. Ideas can leave your mind and turn into real applications, even when you don't have enough time or energy.</p>

<p>Right now, I am designing even more features for those two projects, and also using the CLI to work on my new portfolio page. I can think high-level while the CLI does the heavy lifting.</p>

<p>Which means that, someday, my pet projects can actually be turned into real products.</p>

<p>What about yours?</p>

