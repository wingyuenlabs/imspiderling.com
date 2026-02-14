---
Title: How We Taught Our App to Find Medical Lab Results in Gmail
Description: 
Author: Aleksei Kharitonov
Date: 2026-02-14T21:36:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>About a year ago, I received a task that seemed simple at first glance: implement a feature to search for PDF files containing medical lab results in Gmail.</p>

<p>The idea was straightforward - the user authorizes access to their email through our app, we scan the mailbox for PDFs with lab results and medical data, then display all found documents to the user. After that, the user decides which ones they want to share with us.</p>

<h2>
  
  
  The Challenge
</h2>

<p>Sounds simple enough, right? But there's a big catch - Google doesn't just give you access to a user's mailbox. Working with Gmail API requires going through a verification process and getting approval from the Google team.</p>

<p>I want to express my gratitude to the Google team for their support and for approving our request. But the hardest part wasn't the approval process itself - it was figuring out <strong>how</strong> to even approach it. The documentation didn't provide clear answers.</p>

<h2>
  
  
  Finding the Solution
</h2>

<p>An article from another developer who had faced a similar challenge helped me out. It turned out there's one key requirement for getting approved: <strong>all operations must be performed strictly on the client side</strong>.</p>

<p>Since we have a mobile app, all Gmail operations must happen directly on the user's device. Sending tokens, email IDs, or attachment IDs to the backend is strictly prohibited. Everything stays on the client side.</p>

<h2>
  
  
  The Result
</h2>

<p>The dialogue with the Google team lasted 6 months. And finally, we got the green light. The feature was shipped and warmly received by our users.</p>




<p><em>The moral of the story: sometimes the hardest part of a task isn't the technical solution itself, but figuring out how to get that solution approved by the platform.</em></p>

