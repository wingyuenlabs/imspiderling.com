---
Title: SkillGap: Building an AI-Powered Career Assistant
Description: 
Author: Jing
Date: 2026-03-13T22:09:56.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  SkillGap: Building an AI-Powered Career Assistant
</h2>

<p><em>Jing Ng &amp; Liuyi · Spring 2026</em></p>




<h2>
  
  
  Introduction: The Vision of SkillGap
</h2>

<h3>
  
  
  The Problem
</h3>

<p>Job descriptions are hard to read. They are long, full of buzzwords, and it is genuinely difficult to know if you qualify. Most people either talk themselves out of jobs they could get, or apply to roles they are not ready for.</p>

<p>There is no quick, honest way to compare your skills to what a job actually needs. That is the problem we wanted to solve.</p>

<h3>
  
  
  The Solution
</h3>

<p>SkillGap does the comparison for you. Paste a job description into the app. It checks it against your saved skill profile and gives you a match score, a breakdown of your skills into three groups (Have / Missing / Bonus), and a personalized AI learning roadmap.</p>

<p>You get a clear, actionable plan instead of a guess. As you learn and update your profile, your results improve too.</p>

<h3>
  
  
  A Two-Person, Two-Tier Approach
</h3>

<p>We divided the work so both of us could build at the same time without conflicts.</p>

<p><strong>Jing</strong> built the foundation: JWT authentication, skill profile CRUD, the keyword extraction engine, and the animated match ring.</p>

<p><strong>Liuyi</strong> built the smarter layer: Claude API integration for the learning roadmap, the TDD testing framework, framer-motion UI animations, Skeleton UI loading states, and the AI evaluation suite.</p>

<h3>
  
  
  Two Ways We Used AI
</h3>

<p>We used two AI tools in different roles throughout the project.</p>

<p><strong>Claude Web</strong> handled planning and artifacts. Jing used it to turn early ideas into a prioritized plan. Liuyi used it to convert a hand-drawn wireframe into a proper UI reference, and later to generate the evaluation results webpage from real test data.</p>

<p><strong>Antigravity</strong> handled coding inside the IDE. Both of us used it to write, refactor, and debug code across the frontend and backend.</p>

<p>The split was simple: Claude Web for planning, Antigravity for building.</p>




<h2>
  
  
  Part I: The Foundation: Building the Core
</h2>

<h3>
  
  
  JWT Authentication and User Accounts
</h3>

<p>Everything in SkillGap depends on knowing who the user is. We built JWT-based authentication first: signup, login, logout, and protected routes that reject unauthenticated requests at the API boundary.</p>

<p>Passwords are salted and hashed before they reach the database. JWT tokens are validated on every protected request. We built this first because every other feature depends on it.</p>

<p>On top of auth, we built full CRUD operations for the skill profile. Users can add, edit, and delete skills at any time. Changes persist immediately, so every new analysis uses the latest version of the profile.</p>

<h3>
  
  
  Data Modeling with SQLAlchemy
</h3>

<p>We designed the SQLAlchemy schema to support the full app from the start. The core tables cover users, skill profiles, and analysis history. The history table lets users look back at past analyses and track how their match scores improve over time.</p>

<p>We used Alembic for migrations. Any schema change is a migration file that can be reviewed, tested, and rolled back cleanly instead of a risky manual database edit.</p>

<h3>
  
  
  The Keyword Extraction Engine
</h3>

<p>This is the core algorithm of the product. The engine takes a job description and a user's skill profile, compares both against a curated list of tech skills, and produces three outputs: skills the user has that the job wants, skills the job wants that the user does not have, and bonus skills the user brings that the job did not ask for.</p>

<p>The matching uses keyword normalization so common variations map to the same skill. "React," "ReactJS," and "React.js" all resolve to the same entry. The vocabulary covers frontend, backend, infrastructure, and data tools.</p>

<p>The engine lives in its own module, separate from the auth and profile modules. That made it straightforward to unit test in isolation and easy to extend later.</p>

<h3>
  
  
  The Animated Match Ring
</h3>

<p>A raw percentage is hard to read at a glance. We built an animated SVG ring that shows the match score visually: green for a strong match, yellow for partial, red for a significant gap. The ring draws progressively on page load so the result feels like it is being calculated rather than just appearing.</p>

<p>Below the ring, the three-column view shows the exact skills behind the score so users can see what drove the number.</p>

<h3>
  
  
  Claude Web for Early Product Definition
</h3>

<p>Before writing any code, Jing used Claude Web to work through the project design in plain language. The conversations helped pin down the core user flow, figure out which features to build first, and surface questions that the PRD did not answer yet. For example, what should the app do when a job description mentions a skill that is not in the curated list? What should a new user see before they have set up a profile?</p>

<p>Getting those decisions made before touching the code saved significant debugging time later.</p>




<h2>
  
  
  Part II: The Brain: AI and Engineering
</h2>

<h3>
  
  
  Claude API Integration for Learning Roadmaps
</h3>

<p>The extraction engine tells you what skills you are missing. The Claude API tells you what to do about it.</p>

<p>After the engine produces a list of missing skills, the FastAPI backend sends those skills and the original job description to the Claude API. Claude returns a structured JSON roadmap. Each step in the roadmap has a skill focus, recommended resources, estimated time, and a short explanation of why that skill appears at that point in the sequence.</p>

<p>The React frontend parses the JSON and renders it as a row of cards, one per topic, so the plan is easy to read and act on. It is not a generic list of links. It is a personalized, ordered plan built around what the specific job needs.</p>

<h3>
  
  
  TDD for the Engine, AI Evaluation for the Roadmap
</h3>

<p>AI features are harder to test than deterministic code because the output changes every time. We handled the two parts with different approaches.</p>

<p>For the extraction engine, we used strict TDD. Tests were written before any implementation code. Every module in <code>server/tests/</code> has a corresponding <code>test_&lt;module_name&gt;.py</code> file, and the CI pipeline blocks any merge that drops pytest coverage below 80%.</p>

<p>For the Claude roadmap, we built an AI Assessment Test Suite that scores each response on three dimensions: Relevance (do the resources match the missing skill?), Specificity (does it give concrete next steps rather than vague advice?), and Completeness (does it cover every missing skill?). These scores feed into the eval dashboard.</p>

<h3>
  
  
  Backend Security
</h3>

<p>Two security issues came up during development that were not in the original plan.</p>

<p>The first was client-side score spoofing. The match score was being calculated on the React frontend and sent to the FastAPI backend for storage, which meant a user could manipulate the value before it was saved. We moved the calculation entirely to the server so any client-provided score is ignored.</p>

<p>The second was unhandled SQLAlchemy exceptions. Database errors were returning 500 responses that included full stack traces, which exposes internal implementation details. We added a global error handler that logs the full detail server-side and returns a clean, safe error message to the client.</p>

<p>Both issues were caught in code review before they reached production.</p>

<h3>
  
  
  UI Polish with framer-motion and Skeleton UI
</h3>

<p>We did a focused UI pass to make the app feel responsive, especially during the Claude API call which takes a few seconds.</p>

<p>We used framer-motion to add staggered animations to the skill columns and roadmap cards. Results animate in one after another rather than all appearing at once, which makes the page feel like it is presenting information rather than dumping it.</p>

<p>While the Claude API call is in flight, the results panel shows a Skeleton UI placeholder that matches the shape of the real content. Users see the layout of what is coming instead of a blank screen, which makes the wait feel much shorter.</p>

<h3>
  
  
  Wireframe to Production: Liuyi's UI Workflow
</h3>

<p>Liuyi started with a hand-drawn sketch of the dashboard layout. That sketch went into Claude Web, which generated a detailed wireframe with component suggestions and layout reasoning. The wireframe became the design spec for the production implementation, which was then built with Antigravity inside the IDE.</p>

<p>The workflow was: sketch to Claude Web wireframe to production build. Having a concrete reference before writing any React code meant fewer design guesses and fewer rewrites.</p>




<h2>
  
  
  Part III: Quality and Evaluation
</h2>

<h3>
  
  
  CI/CD with GitHub Actions
</h3>

<p>Every pull request goes through a multi-stage GitHub Actions pipeline before it can merge: lint (ruff and ESLint), security scan, pytest, and build. The coverage check is a hard gate. If backend coverage drops below 80%, the pipeline fails and the PR is blocked.</p>

<p>No secrets live in the codebase. API keys, JWT secrets, and database connection strings are stored in GitHub Secrets and injected at runtime. The app deploys to Render automatically when a PR merges to main.</p>

<p>The pipeline enforces the quality standards that are easy to skip when they only live in a document. A rule wired into CI runs every single time.</p>

<h3>
  
  
  AI-as-a-Judge Evaluation
</h3>

<p>To evaluate the roadmaps Claude generates, we used a second AI call as the judge. After Claude produces a roadmap, a batch evaluation job sends that roadmap plus the original missing skills and job description to an LLM evaluator. The evaluator scores the response on Relevance, Specificity, and Completeness using a fixed rubric.</p>

<p>Using AI to evaluate AI output is the practical approach for testing non-deterministic results at scale. It does not replace human review, but it makes it possible to run evaluations across many test cases and get consistent, structured scores that can be tracked over time.</p>

<h3>
  
  
  The Evaluation Webpage Artifact
</h3>

<p>Liuyi ran the full evaluation suite on real test data and passed the results into Claude Web. Claude Web generated an evaluation webpage artifact: a clean, readable summary of all the scores with notes explaining what drove the high and low marks.</p>

<p>The page served two purposes. It gave us a concrete quality signal on the Claude API integration. And it produced shareable documentation that shows exactly how the AI output was measured and what the results were.</p>




<h2>
  
  
  Conclusion
</h2>

<h3>
  
  
  Efficiency Gains from an AI-First Workflow
</h3>

<p>Using Claude Web for planning and Antigravity for implementation made the whole development process significantly faster. The biggest gains came from two places: early planning conversations that cleared up ambiguity before any code was written, and in-IDE acceleration that handled boilerplate and caught errors during implementation rather than after.</p>

<h3>
  
  
  Why the Two AI Roles Needed to Stay Separate
</h3>

<p>Claude Web is a conversation. It works well for open-ended thinking, generating artifacts, and working through decisions that are not well-defined yet. Antigravity operates directly on your files. It works well for writing, refactoring, and debugging real code.</p>

<p>Mixing the two roles would have been slower. Copying code from a chat window into a file adds friction. Using a conversational tool for planning loses the iterative loop that surfaces better decisions. Keeping each tool in its role made both more effective.</p>

<h3>
  
  
  Future Roadmap
</h3>

<p>The most valuable technical upgrade from here would be replacing keyword extraction with sentence-transformer embeddings for semantic skill matching. The current engine only matches exact keywords. It cannot recognize that "component-based UI development" means React. Semantic matching would make the app useful for a much wider range of job descriptions, including ones written in plain English rather than keyword lists.</p>

<p>Other ideas include resume parsing so users can build their skill profile from an uploaded document, and a reverse analysis mode where an employer checks a candidate's profile against a job posting.</p>

<h3>
  
  
  Final Thoughts
</h3>

<p>SkillGap started as a class project and became something we are proud to show. The modular architecture meant both of us could work independently without stepping on each other's code. A shared <code>.antigravityrules</code> file meant we agreed on conventions before they became conflicts. And splitting AI responsibilities cleanly between planning and building gave us speed without losing control of the output quality.</p>

<p>The biggest thing we learned: AI tools make a good engineering process faster. They do not replace the need for one.</p>




<p><em>Tech Stack Skill Gap Analyzer · Jing Ng &amp; Liuyi · Spring 2026</em></p>





<div class="ltag-github-readme-tag">
  <div class="readme-overview">
    <h2>
      <img src="https://assets.dev.to/assets/github-logo-5a155e1f9a670af7944dd5e12375bc76ed542ea80224905ecaf878b9157cdefc.svg" alt="GitHub logo">
      <a href="https://github.com/MelanieLLY" rel="noopener noreferrer">
        MelanieLLY
      </a> / <a href="https://github.com/MelanieLLY/SkillGap" rel="noopener noreferrer">
        SkillGap
      </a>
    </h2>
    <h3>
      
    </h3>
  </div>
  <div class="ltag-github-body">
    
<div id="readme" class="md">
<div class="markdown-heading">
<h1 class="heading-element">SkillGap — Tech Stack Skill Gap Analyzer</h1>
</div>
<p>A web app that helps job seekers identify skill gaps from job descriptions
Paste a JD, see your match score, and get an AI-generated learning roadmap.</p>
<p><strong>Team 7:</strong> Jing Ng · Liuyi Yang</p>
<div class="markdown-heading">
<h2 class="heading-element">Tech Stack</h2>
</div>
<ul>
<li>
<strong>Frontend:</strong> React 18, TypeScript, Vite, Tailwind CSS, Zustand</li>
<li>
<strong>Backend:</strong> FastAPI, Python 3.11, SQLAlchemy, PostgreSQL</li>
<li>
<strong>AI:</strong> Claude API (claude-sonnet-4-20250514)</li>
</ul>
<div class="markdown-heading">
<h2 class="heading-element">Features</h2>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562615579-a8bc11e0-ef4b-4c96-b46f-4b56c599619a.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI2MTU1NzktYThiYzExZTAtZWY0Yi00Yzk2LWI0NmYtNGI1NmM1OTk2MTlhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQyNWY0MmI5MWYwMGI3YWVlNzg5NWY4NzFmOThlNDcyYzkzMGE4MjI2OWNkMDQzMGQ3N2Q2ZGYxZWI1YjQwNjImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.j3DKLSGXavDZlz-RH0pOYQXUI0eee-cxmMR_cBpmnQw"><img width="600" alt="Screenshot 2026-03-12 at 4 28 16 PM" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562615579-a8bc11e0-ef4b-4c96-b46f-4b56c599619a.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI2MTU1NzktYThiYzExZTAtZWY0Yi00Yzk2LWI0NmYtNGI1NmM1OTk2MTlhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQyNWY0MmI5MWYwMGI3YWVlNzg5NWY4NzFmOThlNDcyYzkzMGE4MjI2OWNkMDQzMGQ3N2Q2ZGYxZWI1YjQwNjImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.j3DKLSGXavDZlz-RH0pOYQXUI0eee-cxmMR_cBpmnQw"></a>
<div class="markdown-heading">
<h2 class="heading-element">Screenshots</h2>

</div>
<div class="markdown-heading">
<h3 class="heading-element">Login page</h3>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562564320-e87a84cc-509c-4268-af6e-f27dbcf52289.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NjQzMjAtZTg3YTg0Y2MtNTA5Yy00MjY4LWFmNmUtZjI3ZGJjZjUyMjg5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWUzMzFiNzM2MmM0M2VlNWVkOGIxMzY4NWNiNjc3MDUxM2QzOTI3ZTFjZTQyZWU0ZDI0YWZkYjg4ZmQ1YjY1ZTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.QGrpmFNEgrmKmIomfJvoR_0ugvpp9a70xrcQPy574cg"><img width="600" alt="Login page" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562564320-e87a84cc-509c-4268-af6e-f27dbcf52289.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NjQzMjAtZTg3YTg0Y2MtNTA5Yy00MjY4LWFmNmUtZjI3ZGJjZjUyMjg5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWUzMzFiNzM2MmM0M2VlNWVkOGIxMzY4NWNiNjc3MDUxM2QzOTI3ZTFjZTQyZWU0ZDI0YWZkYjg4ZmQ1YjY1ZTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.QGrpmFNEgrmKmIomfJvoR_0ugvpp9a70xrcQPy574cg"></a>
<div class="markdown-heading">
<h3 class="heading-element">Sign up page</h3>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562564557-80c94f72-4cc7-4d7c-a633-ef2272248f9c.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NjQ1NTctODBjOTRmNzItNGNjNy00ZDdjLWE2MzMtZWYyMjcyMjQ4ZjljLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQxNmVjZmNjOGZlOTJiNGU5NjhjMDQ4MGY1Y2EwMmEwYTM4NDFhNjc2NGU2NjQyN2IzNmI1YzBmN2Q0NTAyYjgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.pO5sb3YkQeaqFOGeUElvH-W-0Q148UGRAXqa0JOkxrA"><img width="600" alt="Sign up page" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562564557-80c94f72-4cc7-4d7c-a633-ef2272248f9c.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NjQ1NTctODBjOTRmNzItNGNjNy00ZDdjLWE2MzMtZWYyMjcyMjQ4ZjljLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQxNmVjZmNjOGZlOTJiNGU5NjhjMDQ4MGY1Y2EwMmEwYTM4NDFhNjc2NGU2NjQyN2IzNmI1YzBmN2Q0NTAyYjgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.pO5sb3YkQeaqFOGeUElvH-W-0Q148UGRAXqa0JOkxrA"></a>
<div class="markdown-heading">
<h3 class="heading-element">Profile Setup Page</h3>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562570707-3a92c5ea-fade-4b8b-9baa-9a702f9cbda9.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NzA3MDctM2E5MmM1ZWEtZmFkZS00YjhiLTliYWEtOWE3MDJmOWNiZGE5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWRlZjZiZmM4MTNkY2Q3NzE5NjdmNmZkZjY2N2ZkMWQwYmNhZTFiMzM1YTFmNmVjYzc0NWU3YzQzNDExYmM1ZDMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.3RtZkeaw1DMcbxQBk9J6krTL1oy0mJUEF2drNfOZwxw"><img width="1000" alt="Screenshot 2026-03-12 at 2 44 13 PM" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562570707-3a92c5ea-fade-4b8b-9baa-9a702f9cbda9.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NzA3MDctM2E5MmM1ZWEtZmFkZS00YjhiLTliYWEtOWE3MDJmOWNiZGE5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWRlZjZiZmM4MTNkY2Q3NzE5NjdmNmZkZjY2N2ZkMWQwYmNhZTFiMzM1YTFmNmVjYzc0NWU3YzQzNDExYmM1ZDMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.3RtZkeaw1DMcbxQBk9J6krTL1oy0mJUEF2drNfOZwxw"></a>
<div class="markdown-heading">
<h3 class="heading-element">Main Dashboard - Skill Match Analysis</h3>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562607907-bedb4ad3-89da-4c37-98ef-d4654d7b6de7.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI2MDc5MDctYmVkYjRhZDMtODlkYS00YzM3LTk4ZWYtZDQ2NTRkN2I2ZGU3LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWFhZWM1YjUyMTQwNTQ1OTA2MWUyOTIxOWYwMTYwM2Y3N2IxZWI1ZTdmM2U4MDE2NmU5MGViOWZjMmNiYTcwYzAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.s04hHO6-siWEN_QJ1bTBVlZyp2vFsTT7uOUGiEHuTGc"><img width="1094" height="1245" alt="Screenshot 2026-03-12 at 4 07 02 PM" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562607907-bedb4ad3-89da-4c37-98ef-d4654d7b6de7.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI2MDc5MDctYmVkYjRhZDMtODlkYS00YzM3LTk4ZWYtZDQ2NTRkN2I2ZGU3LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWFhZWM1YjUyMTQwNTQ1OTA2MWUyOTIxOWYwMTYwM2Y3N2IxZWI1ZTdmM2U4MDE2NmU5MGViOWZjMmNiYTcwYzAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.s04hHO6-siWEN_QJ1bTBVlZyp2vFsTT7uOUGiEHuTGc"></a>
<div class="markdown-heading">
<h2 class="heading-element">System Architecture</h2>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562550422-d3def8ab-c815-4c69-aa8d-dd8b4037d7ce.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NTA0MjItZDNkZWY4YWItYzgxNS00YzY5LWFhOGQtZGQ4YjQwMzdkN2NlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTVkMmRmZjIxZWU2NTZmYjYxMmJmMzgxYTE3Njc2MWFiMTNkMTAwOWFjMmY1ZjU5ZjJkNDUzNmM1MmFkMDc0NTYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.w7g3P_dN62rHWJ1_Jm-R_pA2avfgQR3WULT2iEhJBcA"><img width="600" alt="System architecture" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562550422-d3def8ab-c815-4c69-aa8d-dd8b4037d7ce.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NTA0MjItZDNkZWY4YWItYzgxNS00YzY5LWFhOGQtZGQ4YjQwMzdkN2NlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTVkMmRmZjIxZWU2NTZmYjYxMmJmMzgxYTE3Njc2MWFiMTNkMTAwOWFjMmY1ZjU5ZjJkNDUzNmM1MmFkMDc0NTYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.w7g3P_dN62rHWJ1_Jm-R_pA2avfgQR3WULT2iEhJBcA"></a>
<div class="markdown-heading">
<h3 class="heading-element">Request flow when user submits a job description</h3>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562550775-b5f22eee-229e-4164-9734-40b3c3a0bd4f.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NTA3NzUtYjVmMjJlZWUtMjI5ZS00MTY0LTk3MzQtNDBiM2MzYTBiZDRmLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTFhOGUzOTY4N2MxYzNkYjE1MjhhMTNjNTcwMmI0NDQ0MzNkMjQ5MmU1MjE1MDQ3NWM5OTRjYzQ5YzEzYmQ0MzYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.3by54Y4FP1ftLt9eLrcTcWM84XtJJTYub1wmnGCaNEo"><img width="500" alt="Request flow" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562550775-b5f22eee-229e-4164-9734-40b3c3a0bd4f.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI1NTA3NzUtYjVmMjJlZWUtMjI5ZS00MTY0LTk3MzQtNDBiM2MzYTBiZDRmLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTFhOGUzOTY4N2MxYzNkYjE1MjhhMTNjNTcwMmI0NDQ0MzNkMjQ5MmU1MjE1MDQ3NWM5OTRjYzQ5YzEzYmQ0MzYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.3by54Y4FP1ftLt9eLrcTcWM84XtJJTYub1wmnGCaNEo"></a>
<p>The user pastes a job description, our server compares it against their saved skill profile, calculates a match score, and asks Claude AI to generate a learning plan for the gap.</p>
<div class="markdown-heading">
<h3 class="heading-element">Frontend + Backend + Database + Claude API</h3>

</div>
<a rel="noopener noreferrer" href="https://private-user-images.githubusercontent.com/36835097/562603800-01f3be1e-8035-4615-aaba-2f65f700c1cd.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI2MDM4MDAtMDFmM2JlMWUtODAzNS00NjE1LWFhYmEtMmY2NWY3MDBjMWNkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWViNGU0ZDcxYTVkZjMwY2MyNDlmNDUwZGY4MTkzNmJkMWY3MTkxYjg1YTA2MzJhZDMzYjRlOGJiMGY0NmUxNDUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.76iPTqXvH4jBp8qsnoXxqvSXypX39A6mFVjAmG7rF6o"><img width="599" height="386" alt="Screenshot 2026-03-12 at 3 56 49 PM" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fprivate-user-images.githubusercontent.com%2F36835097%2F562603800-01f3be1e-8035-4615-aaba-2f65f700c1cd.png%3Fjwt%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzM0NDAxOTMsIm5iZiI6MTc3MzQzOTg5MywicGF0aCI6Ii8zNjgzNTA5Ny81NjI2MDM4MDAtMDFmM2JlMWUtODAzNS00NjE1LWFhYmEtMmY2NWY3MDBjMWNkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNjAzMTMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjYwMzEzVDIyMTEzM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWViNGU0ZDcxYTVkZjMwY2MyNDlmNDUwZGY4MTkzNmJkMWY3MTkxYjg1YTA2MzJhZDMzYjRlOGJiMGY0NmUxNDUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.76iPTqXvH4jBp8qsnoXxqvSXypX39A6mFVjAmG7rF6o"></a>
<div class="markdown-heading">
<h2 class="heading-element">Project Structure</h2>

</div>
<div class="snippet-clipboard-content notranslate position-relative overflow-auto">
<pre class="notranslate"><code>SkillGap/
├── client/           # React frontend (Vite, Tailwind, Zustand)
├── server/           # FastAPI backend
│   ├── tests/        # pytest test suite
│   ├──</code></pre>…</div>
</div>
  </div>
  <div class="gh-btn-container"><a class="gh-btn" href="https://github.com/MelanieLLY/SkillGap" rel="noopener noreferrer">View on GitHub</a></div>
</div>


