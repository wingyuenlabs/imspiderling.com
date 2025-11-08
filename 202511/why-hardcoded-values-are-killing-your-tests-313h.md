---
Title: Why Hardcoded Values Are Killing Your Tests
Description: 
Author: Ilya Ploskovitov
Date: 2025-11-08T22:03:51.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj6wkkuc892hw8837xmok.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj6wkkuc892hw8837xmok.png" alt=" " width="800" height="735"></a></p>

<p>Previously I talked about how I was tired of "flaky" tests and how I'm using AI to generate test steps, saving hours of my time.</p>

<p>I showed how I can create a 20-step E2E test in 6 minutes, just by describing it.</p>

<p>But there's a problem. Even an AI can create a "brittle" test.</p>

<p>Imagine you ask an AI: "Log in using '<a href="mailto:admin@test.com">admin@test.com</a>' and password '12345', and check for the text 'Welcome, John Doe'".</p>

<p>The AI will do it. And then your test will fail.</p>

<p>It will fail when you run it on <code>staging</code>, where the login is <code>staging_admin@test.com</code>. It will fail when 'John Doe' changes his name to 'Jane Doe'. It will fail because it is <strong>hardcoded</strong>.</p>

<p>This is the number one enemy of stable automation. And when I, as a solo developer, started building Debuggo, I knew I had to solve this problem first.</p>

<p><strong>AI is the "Magic," but Variables are the "Engineering"</strong><br>
The AI in Debuggo is great at "translating" English into steps. But real test stability isn't just about the steps; it's about the data those steps use.</p>

<p>From the very beginning, I designed Debuggo to work with two types of variables. This is the logic I wrote myself, on top of the AI core.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu8nproj17f9ilnx3x87z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu8nproj17f9ilnx3x87z.png" alt=" " width="800" height="800"></a></p>

<p><strong>1. Environment Variables</strong><br>
<strong>The Problem</strong>: You have different <code>URLs</code>, logins, or <code>API_KEYs</code> for your <code>dev</code>, <code>staging</code>, and <code>production</code> environments.</p>

<p><strong>The Solution</strong>: In Debuggo, you don't hardcode these values. You store them in your environment settings. Like this:</p>

<ul>
<li><p><code>%BASE_URL%</code> = <code>https://my-staging-site.com</code></p></li>
<li><p><code>%ADMIN_LOGIN%</code> = <code>staging_admin@test.com</code></p></li>
<li><p><code>%ADMIN_PASSWORD%</code> = <code>superS3cret!</code></p></li>
</ul>

<p>Now, here's the best part. I don't need to tell the AI: "Navigate to %BASE_URL%, then fill 'email' with %ADMIN_LOGIN%..."</p>

<p>I just write in plain English:</p>

<p><strong>"login to the website"</strong></p>

<p>And that's it.</p>

<p>The AI is smart enough to understand that "login" requires the <code>%BASE_URL%</code>, <code>%ADMIN_LOGIN%</code>, and <code>%ADMIN_PASSWORD%</code> variables. It will find the right elements on the page (even if the layout changes), plug in the values from your environment, and perform the login.</p>

<p><strong>The Result</strong>: The exact same test. Zero changes to the text. I can run it on any environment just by switching the <code>env</code> in Debuggo. That's the foundation.</p>

<p><strong>2. Temporary ("In-Test") Variables</strong><br>
This is my favorite part.</p>

<p><strong>The Problem</strong>: How do you test a flow where data is generated during the test? The classic example: "Create a user" -&gt; "Verify the user exists" -&gt; "Delete that specific user."</p>

<p>You don't know the <code>ID</code> or <code>email</code> of this user before the test. You can't hardcode it.</p>

<p><strong>The Solution</strong>: I gave the AI the ability to "remember" data from the page and use it in later steps.</p>

<p>Imagine a test case like this:</p>

<ol>
<li><p>"Navigate to '/users' and click 'Create New User'"</p></li>
<li><p>"Fill 'email' with '<a href="mailto:test.user.123@example.com">test.user.123@example.com</a>' and click 'Save'"</p></li>
<li><p><strong>"Find the new User ID from the table and save it as</strong> <code>%new_user_id%</code><strong>"</strong></p></li>
<li><p>"Navigate to /users/delete/%new_user_id%"</p></li>
<li><p>"Click 'Confirm Delete'"</p></li>
<li><p>"Verify that the user with ID <code>%new_user_id%</code> is no longer in the table"</p></li>
</ol>

<p>Step #3 and #6 are what make this a true E2E test. The AI isn't just doing actions; it's extracting dynamic data (<code>%new_user_id%</code>) and using it for test cleanup (Teardown).</p>

<p><strong>What's the takeaway?</strong><br>
For me, this is the difference between a "toy recorder" and a professional QA tool.</p>

<p>A simple recorder logs: <code>click("#btn-123")</code>. That's brittle.</p>

<p>Debuggo (with AI) understands: <code>click "Submit"</code>. That's better.</p>

<p>Debuggo (with AI + Variables) understands: <strong>"login to the website"</strong>... then... <strong>"save the new ID as</strong> <code>%user_id%"</code>... and then... "<strong>delete</strong> <code>%user_id%</code>".</p>

<p>That is a robust, maintainable automated test.</p>

<p>I'm building Debuggo not just to create tests quickly, but to create reliable tests that don't break every time the wind blows.</p>

<p>This approach is my biggest bet. And I'm very curious to see if it solves your "flaky" test pain just like it solves mine.</p>

<p>Try my approach. I'm actively looking for beta testers who understand this "hardcoded data" problem.</p>

