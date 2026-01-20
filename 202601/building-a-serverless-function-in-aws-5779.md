---
Title: Building a serverless function in AWS
Description: 
Author: andrewesley1211
Date: 2026-01-20T22:13:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>🧱 What I Built<br>
I created a simple Lambda function that returns a message like:<br>
Hello, Andre! Your serverless function is working.</p>

<p>The function runs instantly when someone hits the API URL. No servers. No maintenance. Just code.</p>

<p>🪜 Step-by-Step: Building the Serverless Function</p>

<ol>
<li>Open AWS Lambda</li>
<li>Log in to AWS</li>
<li>Search for Lambda</li>
<li>Click Create function</li>
<li>Create the Function</li>
<li>Choose Author from scratch</li>
<li>Name: myServerlessFunction</li>
<li>Runtime: Node.js 20.x</li>
<li>Leave everything else as default</li>
<li>Click Create function</li>
<li>Add the Code
Replace the default code with this:</li>
</ol>

<p>export const handler = async (event) =&gt; {<br>
  const name = event.queryStringParameters?.name || "World";</p>

<p>return {<br>
    statusCode: 200,<br>
    body: JSON.stringify({<br>
      message: <code>Hello, ${name}! Your serverless function is working.</code>,<br>
    }),<br>
  };<br>
};</p>

<p>Click Deploy.</p>

<ol>
<li>Add an API Gateway Trigger
This makes your function accessible through a URL.</li>
<li>Scroll to Function overview</li>
<li>Click Add trigger</li>
<li>Choose API Gateway</li>
<li>Select Create a new API</li>
<li>API type: HTTP API</li>
<li>Security: Open</li>
<li>Click Add
AWS will generate a public endpoint URL.</li>
<li>Test the Function
Open the URL in your browser:</li>
</ol>

<p>Got my result at below link<br>
<a href="https://yz179ik8xe.execute-api.us-east-2.amazonaws.com/default/myServerlessFunction" rel="noopener noreferrer">https://yz179ik8xe.execute-api.us-east-2.amazonaws.com/default/myServerlessFunction</a></p>

<p>🧠 What I Learned<br>
Building this project helped me understand:</p>

<ul>
<li>How AWS Lambda works</li>
<li>How serverless functions scale automatically</li>
<li>How API Gateway exposes functions through HTTP endpoints</li>
<li>How to pass query parameters into a Lambda function</li>
<li>How to deploy backend logic without provisioning servers
This is a great beginner project because it teaches the fundamentals of serverless architecture without overwhelming complexity.</li>
</ul>

