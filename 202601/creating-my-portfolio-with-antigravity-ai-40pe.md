---
Title: Creating my portfolio with Antigravity AI
Description: 
Author: Raffael Eloi
Date: 2026-01-15T21:25:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>I decided to write a detailed article describing my entire journey creating my portfolio with Antigravity.</p>

<p>The sequence:</p>

<ol>
<li>Idea</li>
<li>Approach / Strategy</li>
<li>AI Choice</li>
<li>Key concepts and choices for the frontend app</li>
<li>Experience / Journey</li>
<li>AI workflow</li>
<li>Result</li>
<li>Final thoughts</li>
</ol>

<h2>
  
  
  1. Idea
</h2>

<p>While I was searching for opinions about the <a href="https://www.wearedevelopers.com" rel="noopener noreferrer">WeAreDevelopers</a> event, which I would like to participate in one day, I found an interesting opinion on the dev.to website. After reading it, I came across a pop-up from the dev.to promoting a <a href="https://dev.to/devteam/join-the-new-year-new-you-portfolio-challenge-3000-in-prizes-feedback-from-google-ai-team-4e7g?bb=259942">Frontend challenge using AI</a>. <br>
I already wanted to create my portfolio and understand AI better, so it was a perfect opportunity to kill two birds with one stone.</p>
<h2>
  
  
  2. Approach
</h2>

<p>My idea was to use the <em>learn by doing</em> method. I didn't want to spend many hours learning about the AI, instead, I wanted to see how intuitive my chosen AI is and how far I could go without having to provide very technical prompts.</p>
<h2>
  
  
  3. AI Choice
</h2>

<p>In the challenge, we had three AI options (from Google), and I chose Antigravity. I'm going to be honest here: I chose it because I liked the name (you've probably done something similar at some point in your life, so don't judge me).</p>

<p>I watched the introduction video on the <a href="https://antigravity.google" rel="noopener noreferrer">Antigravity website</a>, and they presented the following method:</p>

<ul>
<li>Implementation plan</li>
<li>Pre-requisites</li>
<li>Proposed Architecture</li>
<li>Implementation steps</li>
<li>Verification plan</li>
</ul>
<h2>
  
  
  4. Key concepts and choices for the frontend app
</h2>

<p>Frontend: </p>

<ul>
<li>Next.js (I don't have much familiarity with it, so that's why I chose it)</li>
<li>TypeScript</li>
</ul>

<p>Key concepts:</p>

<ul>
<li>Easy to change</li>
<li>Separation of concerns</li>
<li>MVC Architecture</li>
<li>Pretty UI</li>
<li>Feature-based layout (<del>atomic design</del>)</li>
</ul>
<h2>
  
  
  5. Experience / Journey
</h2>

<p>I will highlight some moments in the process of creating the portfolio that made me reflect on certain AI approaches. </p>

<ul>
<li>Downgrading Node version</li>
</ul>

<p>In order to use the latest version of Next.js, the node version installed on my machine was not compatible. Instead of suggesting an upgrade, the AI decided to downgrade Next.js (I stopped it before that).<br>
This caught my attention: why didn’t the AI suggest upgrading Node.js instead?</p>

<ul>
<li>Accessing web pages</li>
</ul>

<p>During the process of creating my portfolio, I gave the AI some public links to my GitHub and LinkedIn profiles. The AI opened up a browser and fetched the information.<br>
That was pretty impressive, and it was interesting to watch the steps it took.</p>

<ul>
<li>All the entities in a single file</li>
</ul>

<p>The AI's first choice was to create all the entities inside a single file. Although this is a very basic issue, I often notice the AI choosing the most "cheap" and "compact" solutions. We need to pay attention to that. AI can read an entire codebase easily, we cannot.  </p>

<ul>
<li>Atomic design... literally</li>
</ul>

<p>This one is kind of funny. In the prerequisites I provided to the AI, I defined the Atomic Design, but it turns out the AI built the following structure:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm1i3s15nkl5xlqogkryo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm1i3s15nkl5xlqogkryo.png" alt=" " width="256" height="140"></a></p>

<p>Well, that was not my intention. I wanted to use Atomic Design, but not the literal folder names. I preferred a more organic separation instead of using concept names for each folder.</p>

<ul>
<li>Component delegation</li>
</ul>

<p>I don't know Next.js very well, so it was not clear to me how things were working. After some back and forth with the AI, we came up with the idea of component delegation, which I really enjoyed.</p>
<h2>
  
  
  6. AI workflow
</h2>

<p>I really liked Antigravity's approach. It consists of two or three steps, depending on the prompts you provide. These steps are represented as markdown files and require your approval.</p>

<ul>
<li>Proposal</li>
</ul>

<p>The AI provides a markdown file containing some ideas and options you can choose from, followed by the pros and cons of each choice.</p>

<ul>
<li>Implementation plan</li>
</ul>

<p>Before getting your hands dirty, the AI shows you the plan, the steps, and the goal for the action it is trying to implement. <br>
Having this step gave me a really good feeling about what the AI is thinking and how it plans to achieve the result. It is much better than letting the AI do everything and reviewing the output only at the end.</p>

<ul>
<li>Walk Through</li>
</ul>

<p>After you approve the implementation plan proposed by AI, you can see what has been implemented and the final result. <br>
Overall, this is a pretty good idea.</p>
<h2>
  
  
  7. Result
</h2>


<div class="ltag__cloud-run">
  <iframe height="600px" src="https://portifolio-updated-485096414492.europe-west1.run.app">
  </iframe>
</div>


<p>This is my application, and I'm going to break down the architecture and explain my choices.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6m55rvauagvbbnpdu87o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6m55rvauagvbbnpdu87o.png" alt=" " width="627" height="703"></a></p>

<ul>
<li>app: It contains the pages</li>
<li>components: related to the UI and the feature-based approach</li>
<li>models: entities representing the domain</li>
<li>services and data: I decided to mock a JSON file with the resume structure as follows:</li>
</ul>

<p>Resume</p>

<ul>
<li>Basic info</li>
<li>Experiences</li>
<li>Education</li>
<li>Extras</li>
</ul>

<p>For this section, I mocked the file with my data, but I kept it this way because, in the future, it can retrieve data from an API and generate the profile automatically.</p>

<h2>
  
  
  8. Final thoughts
</h2>

<p>In the end, it was a really fun experience. I was able to learn more about Antigravity AI, Next.js, and Google Cloud Run. I believe that the more we expose ourselves to challenges, the more we sharpen our minds and skills.</p>

<p>Antigravity surprised me. It was a good experience using it, especially with the implementation plan, which was a really cool idea. </p>

<p>I bought a domain, and it was pretty easy to link it to my portfolio using Google Cloud. Check it out: (<a href="https://raffaeleloi.dev" rel="noopener noreferrer">https://raffaeleloi.dev</a>)</p>

<p>Also, I was able to learn more about the turbo pack (<a href="https://nextjs.org/docs/app/api-reference/turbopack" rel="noopener noreferrer">a really fast compiler written in Rust</a>).</p>

<p>I didn't spend a lot of time on this challenge, but even so, I had fun and managed to build my portfolio, which has been on my to-do list for a long time.</p>

<p>Before I forget, I have been searching for a text editor that would help me avoid distractions, and I found this one <a href="https://zenpen.io" rel="noopener noreferrer">zenpen.io</a>. I used to write down my ideas, and I really recommend it. </p>

<p>My repo: <a href="https://github.com/Raffael-Eloi/portifolio-updated" rel="noopener noreferrer">https://github.com/Raffael-Eloi/portifolio-updated</a></p>

<p>Any comments or suggestions are welcome.</p>

