---
Title: AI fluff generation
Description: 
Author: sarabadu
Date: 2025-08-27T21:49:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've never been too picky on PR descriptions, but getting good context and details in a PR is a lovely experience. Now AI (Github copilot) gives you that experience with one click, right? RIGHT?</p>

<p>Well sometimes, maybe... but many times what you get is a detailed summary of what changed in each file and that is something I already have in a different tab, I see a lot of text but still gives me no value.</p>

<p>This happens mostly on the GH web interface while editing the PR description, and it is understandable in that instance the only context available is the files being modified, and commit messages, not much more. but the most valuable information inside a PR description is maybe the "Why" are we doing this.</p>

<h2>
  
  
  How the fluff looks like
</h2>

<p>If you ask an llm to generate text, It will by default add some fluff to it: unnecessary data, fancy wording, and unchecked facts.</p>

<p>This means you can easily end up with fluffy content everywhere.</p>

<ul>
<li>"add test" -&gt; ends up with 20 test you hardly read or follow</li>
<li>"write the commit message" -&gt; end in "improvements and refactors in file X"</li>
<li>"Writing social media post" -&gt; fancy words everywhere</li>
<li>"user story"... who knows</li>
</ul>

<p>The bigger issue with this fluff is getting the perception of value, but not real value. Your 20 test maybe test the same thing, or miss the actual business edge cases logic you need, but there are 20 of them and all of them are green!!! That LinkedIn post looks really professional you can remove some em dashes and nobody will notice is not you the one talking there.</p>

<h2>
  
  
  Taking the control back
</h2>

<p>After all this we need to filter the fluff to get the real value out and take the control back eventually. So why not try to work fluff free from the beginning.</p>

<p>I love to use llm on all this tasks, are really handy. So I look strategies to keep control.</p>

<p>While writing content like LinkedIn, PR descriptions, Documentation, and this post, I like to write it first and then ask for pointing out improvements or typos, but always being explicit on "Do not re-write it, just explain what to improve". This in my experience give a lot of value by correcting some things, and challenge my text but without losing my identity and the control of what I'm writing</p>

<p>While coding I like to use agents in this way:<br>
1- I start explain what Im trying to do, my objective, and some indications on how I'm thinking to do it<br>
2- I usually ask for "create a plan" and "do it step by step waiting for my input between steps"<br>
3- Then I challenge the plan or ask the llm agent to challenge my plan until I find a plan that makes sense for me<br>
4- Finally i execute the plan step by step</p>

<p>When exploring new usages or tools using llm agents, I always try to find this way to keep in control and fluff free. It looks slower at first but at the end is much better to get a controlled result rather that getting a lot of "content" but some personal debt</p>

<p>Many times I hear "It was (copilot|cursor|the AI)", I like to think on "It was me/us using (copilot|cursor|the AI)" and take the responsibility of my content</p>

<p>"But I like to vibe code", I understand I like it too, it's great in some situations like exploring ideas, trying out things, or work in hackathon mode, but eventually I need to get the control back.</p>

