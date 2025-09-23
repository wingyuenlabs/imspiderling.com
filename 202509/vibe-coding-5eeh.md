---
Title: Vibe Coding!
Description: 
Author: coco
Date: 2025-09-23T21:27:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>I have been Vibe coding. Everybody is calling that shit. Yesterday, I was against the trend; now I jump on the wave. But why? What is it all about? When is it appropriate and when is not?</p>

<p>I have been using GitHub Copilot for over a year, especially since it was free. I tried Cursor, and Gemini is my best friend. However, this is not vibe coding. At least, this is not working well with AI. I was doing it all wrong. </p>

<blockquote>
<p>GitHub Copilot is not Vibe coding</p>
</blockquote>

<p>This quarter, the CEO sent me on a mission: “I’d like you to test every AI tool you can and make a manual for your team. I want the hole agency working with AI. We must be more efficient.”<br>
Therefore, I pushed myself to test them all for the last three months: GitHub Copilot, Cursor, Claude Code, Firebase Studio, Figma Make, and other tools. I looked up many courses, watched so many videos, and did some experiments. I didn’t want to just test these tools. Hello world was not enough. I wanted to really use them.<br>
That’s why, I made this plan:<br>
First, I made a list of the most important ones and found out what they are for.</p>

<ul>
<li>Gemini or ChatGPT is the new Google. It is useful to ask questions, investigate, or improve your ideas.</li>
<li>Cursor and GitHub Copilot are developer assistants.</li>
<li>Figma Make, Webflow builder, or Firebase studio are tools for prototyping.</li>
<li>And Claude code just makes the job for you.</li>
</ul>

<p>Then, I made another list with the issues or tasks the team had in the queue to do. Some were internal jobs, while others were for our clients. I tried to identify which ones could help me the most to test these tools. I picked up four for testing myself and left the others for the team.<br>
Tasks selected:</p>

<ul>
<li>To upgrade AWS-SDK V2 to V3 and write a new documentation of a 5-years-old API project. (We don’t need this, but I wanted to do it)</li>
<li>To fix one inconsistent bug of an internal tool developed with NodeJS and Handlebars.</li>
<li>A catalog products API demo for a client. As it was a JSON end API, we needed something more illustrative to show our client the functionalities and possibilities of the product. So, we must make an API Frontend and a retailer product page demo.</li>
<li>A landing page with a decision tree logic and a form.</li>
</ul>

<p>For the first and second task I chose Claude Code. This tool is awesome. You just need to install it, log in, and it does the work for you. Literally.<br>
It is a console tool. So, it can do everything. Since scanning all your project files, till executing console commands. It knows when Docker is running, your NodeJS version, etc. Of course, you could let it do whatever it wants, or you could keep an eye on the tool letting it ask you step by step:</p>

<blockquote>
<p>“I will run npm install, yes or no. I will add this package to the project, yes or no. Etc.”</p>
</blockquote>

<p>This tool is useful when you are working on an old project, and you need to fix some issues. First, I asked it to read the files and make me a new documentation of the project. After 5 minutes, it wrote me a new .md file with a very detailed documentation. Wonderful! It even mentioned a few features I didn’t remember at all.<br>
Then, I told Claude Code that I need to upgrade the AWS SDK version. So, it started again and after a minute it modified three files: </p>

<ul>
<li>It added the new package version to package.json, and installed it through a command in the console.</li>
<li>It modified two more files when this service was created and executed.</li>
<li>Then recommended me to test it and make a new .md file with the detailed modifications it did.</li>
</ul>

<p>It worked fine, of course and took me 30 minutes of work even tested the project online. :D</p>

<p>I don’t know how long it would have taken me to get this kind of work, but I am sure it should have been longer.<br>
The most importantly, this kind of task is not a creative one. You need to upgrade, only because AWS is warning you to do it. You never know exactly why, but you must do it. That’s the main reason to do it with AI: done quickly, efficiently, and focus on something else; more important, perhaps...</p>

<p>The second task was quite different. We had an internal tool with an inconsistent bug. Sometimes it worked but sometimes didn’t. We HAD TO fix it. This was not a warning.</p>

<p>I told Claude Code which was the issue and asked it for help to fix it. It took 1 hour. But finally, it was done. We had to work both this time. Claude Code modified a few files and I tested. Sometimes It fixed something and broke other thing. So, we were iterating changes till It was all fixed. I never had to write a single file. I was just reading the changes, testing the tool, and giving feedback to Claude to continue working.</p>

<p>It took it 1 hour, so I thought it was too long. This time I wanted to compare wether the AI was efficient or not. Therefore, I made a pull request, but I didn’t approve yet. I started a new branch and tried to fix by myself, without AI. However, I couldn’t do it in 1 hour. It was a weird bug. So, I quit and approved the Claude Code job. The AI was ok and helped me with the bug. Now the tool is working, and everybody is using it. </p>

<p>The first two tasks were not vibe coding as I am listening to it. But, I thanked AI for helping with those tasks, because I really didn’t want to complete them. These are the typical kind of stuff that you always leave and nobody will never do. Now, in less than a morning, I completed them. Next!</p>

<p>The demo for a client was more happier task. There wasn’t a template, I had to develop from scratch. <br>
I had asked the PO to make a wireframe of what she wanted. So, I had this wireframe, an API swagger with all the endpoints documented, and a mood-board made by the creative director.<br>
The task was to develop a CRUD for an API had already developed. It was a laptop catalogue. Each product with its properties, images, and texts. I didn’t need to develop the functionalities to create and delete products. However, the Update has more than one functionality. For example: upload images, localizations, delete properties, etc. <br>
Besides, I needed to develop a retailer looked product page frontend. This fetches the data from the API. Therefore, anytime I update the product with the CRUD, the retailer page would show the differences. I had estimated 40 hours at least.</p>

<p>As I had read and watched many videos, I knew I had to start asking Claude to make a prompt for another AI. So, I told Claude what I needed to do and uploaded the wireframe and the mood-board. The LMS, wrote me a long and detailed markdown file. This is very useful because writing these instructions by myself it would take me so long and I would never do it as detailed as the AI. Between bots, they understand each other.<br>
Then I decided to test two similar AI: Firebase for the CRUD frontend and Bolt for the retailer product page. Both worked fine. I used the prompt Claude did for me, the wireframe and mood-board. </p>

<p>For the product page frontend I uploaded a screenshot for a real retailer product page and paste only the specific endpoint that fetch the product with all its properties and assets.<br>
Most of the work was already done. I need to add more CRUD features and localization. So I downloaded the code and run local. I continue working with GitHub Copilot. I started for free, but It didn’t last so I upgraded to pro. The tip here is to go step by step. Example: </p>

<ul>
<li>Add a button in this page to upload a new image. The button just opens a modal.</li>
<li>Add a form in the modal to upload an image. Drag and drop functionality.</li>
<li>Add a handle form function to get the image file and validate it.</li>
<li>Connect the function to the API and catch the response.</li>
<li>Close the modal and show the new image uploaded in the images’ carousel.</li>
</ul>

<p>As I experienced, if you just asked it all together, it wouldn’t work fine. It would get confused and would mix all up. GitHub copilot is a helper; it will not make all the job for you. If you need something like that, Claude code is better.<br>
This third task took me 20 hours. Half time I had estimated. It was ok and ready to production. However, I realized that you need to know what are you doing to make it right in the first time. My suspicious is that a SR developer would get much more out of the tool.</p>

<p>For the last task I had decided to do, I wanted to try something different. I need to develop a landing page with a decision tree logic and a form. The API to send the form was already done, I just need to fetch with POST method and that would be all. So I was only a Frontend job. The difference was that I had a brand guide to follow now. This landing page was a sub landing of the main site. So, I decided to use Figma Make. <br>
It was excellent. Now we are talking, this is fully vibe coding!</p>

<p>I only needed to tell Figma what I need, and I build a react project with this full landing developed. Then I paste the site URL and ask Figma to copy the design. I downloaded the code and asked Cursor to docker the project. We are using an AWS EKS, so I need the project into a Docker container. Finally, I replaced the text the account team had sent me and published.<br>
4 hours later, first version was running in production. The best is the account team (my client) could try it, browse it for a while, test the mobile version and then ask me for changes. This is better that made a Figma prototype. Because it is working, you could make no changes at all and publish. You could tried with your audience for better. The best is that was only 4 hours of work. More agile methodology than this is impossible.</p>

<p>I now understand what is Vibe coding about. It is very useful for prototyping. Having the MVP in the shortest time ever. This is very useful for business and we need to take advantage of it.</p>

