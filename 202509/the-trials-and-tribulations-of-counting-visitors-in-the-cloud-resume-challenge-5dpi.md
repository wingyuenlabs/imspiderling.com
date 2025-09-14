---
Title: The Trials and Tribulations of Counting Visitors in the Cloud Resume Challenge
Description: 
Author: Sasha Lapommeray
Date: 2025-09-14T21:42:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>This summer, I wanted to challenge myself to go hands-on with AWS. I had been working with cloud services across multiple projects and wanted to gain a deeper understanding of the core services. After searching for a short while, I found the perfect project to start with—the <a href="https://cloudresumechallenge.dev/" rel="noopener noreferrer">Cloud Resume Challenge</a>. The challenge covered AWS services and concepts across various domains, including S3, CloudFront, Route 53, Lambda, DynamoDB, API Gateway, and SAM. Throughout the challenge, I gained a deeper understanding of how these services function. I could go deeper on what I learned about each of these services, but for now, I wanted to discuss what I found to be the hardest part of the challenge—the visitor counter.</p>

<p>For this step of the challenge, I needed to create a backend database, a frontend JavaScript program and a Lambda function capable of sending data between them. The project specifications required this setup to ensure logical separation between the frontend, backend, and database, which strengthens security. If all the code and database access were handled directly in the frontend, the site would be vulnerable to attacks such as data tampering, SQL injection, or exposure of API keys. To tackle this step, the very first thing I did was try to find a simple way to send and retrieve data from the frontend JavaScript to the Lambda to the backend database. This step was relatively straightforward.</p>

<p>After I was able to do this successfully, it was just a matter of sending the incrementing visitor count value and setting it up as an API in Amazon API Gateway. It was only a few days later that I realized the glaring flaw in the counter I had created. The counter was designed to increment every time the webpage was visited, even on reloads. This was obviously not the intent of the visitor counter and would not represent an accurate reflection of the true visitor count. I was struggling to find a reliable and universal way to make sure that the visitor counter would only go up for new and unique visitors. I considered many methods, but they all had drawbacks.</p>

<p>The first idea I had was to log the IP address of the user in the database, as well as the visitor counter value. Eventually, I decided against that idea for two reasons. First, the IP wouldn’t reliably prove that each user is really unique, since there are various cases where the same IP could be used by multiple users, or where the same user could use multiple IPs. Second, I felt that for a simple personal project like this, collecting personal user data, such as an IP address, would be overkill, unnecessary, and go against the principles of good data privacy. The next idea I had was looking into external tools, but again, I felt it was overkill for a simple personal project like this. I also felt that I should keep the scope of the project limited to AWS as much as possible.</p>

<p>The final idea I had was to use localStorage to set a parameter (visited=true) and increment the visitor counter only if the user had not visited the website before (i.e., if the parameter wasn’t present). This was still not a perfect solution since the user could delete the localStorage data and be counted again, but I think for a simple project like this, it was the most suitable solution.<br>
This was easily the hardest part of the challenge and had me racking my brain for solutions for days. I would like to look into this more and contrast various methods to reliably count unique visitors to a website. In working on this project, I found that this was a topic of interest for many researchers, so I would like to revisit and explore it more in the future.</p>

<p>This was only one part of an amazing challenge created by Forrest Brazeal. I highly recommend that anyone looking to learn more about cloud computing take the challenge. There are versions for GCP and Azure as well, so anyone can participate.</p>

<p>I am currently hosting my website at <a href="https://sashalapommeray.com/" rel="noopener noreferrer">sashalapommeray.com</a>, and I encourage anyone who’s interested to check it out. My source code is located on my <a href="https://github.com/sashalapommeray" rel="noopener noreferrer">personal GitHub</a>.</p>

<p>Cloud Resume website: <a href="https://sashalapommeray.com/" rel="noopener noreferrer">https://sashalapommeray.com/</a> <br>
Frontend code: <a href="https://github.com/sashalapommeray/cloud_resume_frontend" rel="noopener noreferrer">https://github.com/sashalapommeray/cloud_resume_frontend</a><br>
Backend code: <a href="https://github.com/sashalapommeray/cloud_resume_backend" rel="noopener noreferrer">https://github.com/sashalapommeray/cloud_resume_backend</a></p>

