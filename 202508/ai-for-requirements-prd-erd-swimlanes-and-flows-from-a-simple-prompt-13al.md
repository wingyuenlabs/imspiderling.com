---
Title: AI for Requirements: PRD, ERD, Swimlanes, and Flows from a Simple Prompt
Description: 
Author: Aaron K Saunders
Date: 2025-08-25T20:13:27.000Z
Robots: noindex,nofollow
Template: index
---
<p>In this video, I show my real workflow for turning a rough idea into clear, client-ready documentation—using AI. </p>

<p>I take a simple church visitor intake example (QR code → form → database → emails/postcards) and use T3Chat (GPT-5 Reasoning) to generate:</p>

<ul>
<li>A high-level <a href="https://www.geeksforgeeks.org/work-experiences/product-requirements-document-definition-importance-benefits-and-steps-with-example/" rel="noopener noreferrer">Product Requirements Document (PRD)</a>
</li>
<li>An <a href="https://www.geeksforgeeks.org/dbms/introduction-of-er-model/" rel="noopener noreferrer">Entity Relationship Diagram (ERD)</a> with Mermaid syntax</li>
<li>Database tables and fields you can drop into a spreadsheet</li>
<li>Flow and <a href="https://www.geeksforgeeks.org/system-design/swim-lanes-in-activity-diagram/" rel="noopener noreferrer">swimlane diagrams</a> you can paste into <a href="https://mermaid.live" rel="noopener noreferrer">Mermaid Live Editor</a>
This approach helps with client discovery, statements of work (SOW), estimates, and aligning technical and non-technical stakeholders—without needing super-detailed prompts.</li>
</ul>

<h2>
  
  
  The Video
</h2>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/O91I1EZlLGE">
</iframe>
</p>

<p><strong>What you’ll learn:</strong></p>

<ul>
<li>The exact prompt structure I use to get PRD, ERD, tables, and diagrams</li>
<li>How to paste Mermaid output into Mermaid Live Editor for instant visuals</li>
<li>How to validate requirements with teams/clients using generated docs</li>
<li>Tips for exporting tables to spreadsheets for easier reviews</li>
</ul>

<p>In the context of system design, a swimlane diagram is a powerful tool for visualizing the interactions and responsibilities between different components within a system element like microservices, APIs, databases and external system. </p>

<p><strong>Resources mentioned:</strong></p>

<ul>
<li><a href="https://gist.github.com/aaronksaunders/c5703bff774c00b45b11e2880abef4c5]" rel="noopener noreferrer">Full document output</a></li>
<li>Mermaid Live Editor: <a href="https://mermaid.live" rel="noopener noreferrer">https://mermaid.live</a>
</li>
<li>Alternative AI tools to try: ChatGPT, Gemini, Claude</li>
</ul>

<p>If you want the exact prompt I used, grab it in the link above. Drop your questions and share what AI tools you’re using for requirements and docs—I’ll feature some in a future video.</p>

<h3>
  
  
  The Prompt
</h3>

<blockquote>
<p>i need to create a solution for mu church group to capture information from guests and have a way for them to scan a QR code that launches a web app for them to enter information for us to add them to a system. the system needs the ability to export the list so i can then create welcome post cards to send to them. if they provided and email address that we could send an email if they opted in.<br>
create a product requirement document, create an ERD, create a table with field name, and appropriate description for the tables in the database, create a flow diagram to display application flow, create a swim diagram to show flow from technical perspective</p>
</blockquote>

<h3>
  
  
  The Diagram
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbkzjqfpk2kx45s6lakh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdbkzjqfpk2kx45s6lakh.png" alt=" " width="800" height="655"></a></p>

