---
Title: My API Testing & Postman Automation Journey with Gradific API
Description: 
Author: Loveline Chioma Ezenwafor
Date: 2025-10-29T22:09:04.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the past week, I worked on a detailed API Testing &amp; Presentation Task using the Gradific REST API documentation. This assignment pushed me deeper into advanced API testing concepts such as CRUD operations, nested endpoints, authorization flows, and Postman collections.</p>

<p>Here’s a breakdown of what I did ✅</p>

<p>✅ Task Overview</p>

<p>I was required to:<br>
    1.  Review the API documentation<br>
    2.  Select 4 resources (including Authentication)<br>
    3.  Perform full CRUD operations on each<br>
    4.  Test nested endpoints where applicable<br>
    5.  Create:<br>
    • 80 total test cases (20 per resource)<br>
    • A Postman collection for all requests<br>
    • Professional documentation files (.docx and .xlsx)</p>

<p>🔐 Authentication Challenges</p>

<p>Initially, every request returned:<br>
Cannot POST /api/docs</p>

<p>later:<br>
Cannot POST /api/auth/register</p>

<p>Eventually, I got the API responding, but then encountered:<br>
Invalid authentication token</p>

<p>While debugging, I discovered that the API requires a valid JWT for protected routes — meaning I needed to:</p>

<p>✔ Authenticate first<br>
✔ Retrieve a token<br>
✔ Add it to every request header</p>

<p>This taught me how token-based security impacts testing workflows and how to handle protected endpoints properly.</p>

<p>🛠 Tools &amp; Skills Used<br>
Tool / Concept<br>
Postman</p>

<p>How I Used It<br>
CRUD operations, environments, token automation</p>

<p>Tool / Concept<br>
HTTP Headers</p>

<p>How I Used It<br>
Added Authorization tokens</p>

<p>Tool / Concept<br>
Status Code Validation</p>

<p>How I Used It<br>
Confirmed expected responses</p>

<p>Tool / Concept<br>
API Documentation</p>

<p>How I Used It<br>
Navigation &amp; endpoint selection</p>

<p>Tool / Concept<br>
Test Documentation</p>

<p>How I Used It<br>
Excel + Word formatted test suites</p>

<p>📌 Deliverables I Produced</p>

<p>✅ Full Postman Collection<br>
✅ 80 detailed test scenarios<br>
✅ Functional + auth testing across 4 resources<br>
✅ Documentation files ready for review</p>

<p>This experience helped me:<br>
    • Understand backend dependencies<br>
    • Build confidence in API automation setup<br>
    • Strengthen debugging and problem-solving skills<br>
    • Work with REST APIs beyond simple GET/POST calls</p>

<p>🎯 Key Takeaways<br>
    • Not all API failures mean the API is “down” — authentication matters!<br>
    • Test cases should include both positive and negative scenarios.<br>
    • Clear documentation is vital for QA engineering roles.<br>
    • API testing is as much about logic as it is about tooling.</p>

<p>🙌 Wrapping Up</p>

<p>This project confirmed my passion for Quality Assurance Engineering and backend testing. I’m excited to build more with Postman, automation scripts, and CI pipelines as I advance in this field.</p>

<p>If you’re working on APIs:<br>
👉 Always start by validating authentication flow<br>
👉 Read the docs twice before assuming the endpoint is wrong 😅</p>

<p>Thanks for reading! Feel free to connect with me on my journey ❤️</p>

