---
Title: I Built a PDF Toolkit With 50+ Tools Using Open-Source Software - Here's How
Description: 
Author: Sanusi Hassan
Date: 2026-03-13T21:54:50.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnb3qdvlg1fcjcjx0pnbv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnb3qdvlg1fcjcjx0pnbv.png" alt="Screenshot of the PDFEquips interface showing various PDF conversion tools" width="800" height="364"></a></p>

<p>About two years ago I posted <a href="https://www.pdfequips.com/" rel="noopener noreferrer">PDFEquips</a> on Hacker News and it hit the front page. The comments were fascinating - most of the discussion wasn't about the tool itself, but about privacy, whether PDF processing belongs in the cloud, and why most online PDF tools feel like they're trying to scam you.</p>

<p>That feedback shaped everything I've built since. Here's where the project stands now and how I built it.</p>

<h2>
  
  
  Why I started
</h2>

<p>Every online PDF tool I tried mostly had the same problems: watermarks on free conversions, forced account creation for basic tasks, aggressive upsells, or sketchy ad-filled interfaces. The underlying tech to do most PDF operations isn't complicated - the problem was packaging.</p>

<p>So I started building <a href="https://www.pdfequips.com/" rel="noopener noreferrer">PDFEquips</a> as a clean, browser-based toolkit that does what you need without the friction.<br>
</p>
<div class="crayons-card c-embed">

  <br>
<strong>The Stack</strong>

<p>The frontend is Astro.js with React and TypeScript. I use Redux for state management and Zustand specifically for handling file blob state, which turned out to be a much cleaner pattern than trying to force file data through Redux.</p>

<p>The backend is split between Node.js/Express and Python Flask for the heavy PDF processing.</p>

<p>PostgreSQL handles user data and subscriptions. Everything runs behind nginx with PM2, gunicorn on a VPS.</p>


</div>


<h2>
  
  
  The hardest problem: document translation
</h2>

<p>Most PDF tools stop at conversion and merging. I wanted to add something I couldn't find anywhere else — translating entire documents while preserving the layout.</p>

<p>It works across PDF, Word, Excel, and PowerPoint files, and each format has its own set of edge cases.</p>

<p>Here's a quick demo of translating a PDF document<br>
  <iframe src="https://www.youtube.com/embed/iMvnE8U1oFk">
  </iframe>
</p>

<h2>
  
  
  Batch Web-to-PDF
</h2>

<p>A user asked if they could convert a list of web pages to PDF. Not one page - hundreds.</p>

<p>So, I built it. You can paste URLs one by one into a textarea, or upload a CSV/TXT file. Set your page size, margins, and orientation, then hit convert.</p>

<p>  <iframe src="https://www.youtube.com/embed/cHDPlVm8gms">
  </iframe>
</p>

<h2>
  
  
  Format conversions that actually work
</h2>

<p>One of the most requested features was converting PDFs to editable formats without destroying the layout. Here's PDF to Excel in action:</p>

<p>  <iframe src="https://www.youtube.com/embed/eO4p8vM1T1I">
  </iframe>
</p>

<p>And PDF to PowerPoint — preserving slides, images, and text positioning:</p>

<p>  <iframe src="https://www.youtube.com/embed/mYM5Z0JN77s">
  </iframe>
</p>

<p><a href="https://www.pdfequips.com" class="ltag_cta ltag_cta--branded" rel="noopener noreferrer">Try PDFEquips for free</a>
</p>

<p>I'm a solo developer building this from Chad. Happy to answer any technical questions.</p>

