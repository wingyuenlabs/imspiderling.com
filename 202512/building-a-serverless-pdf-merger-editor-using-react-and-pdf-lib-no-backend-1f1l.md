---
Title: Building a Serverless PDF Merger & Editor using React and pdf-lib (No Backend!)
Description: 
Author: opzozi
Date: 2025-12-22T21:33:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>I merge PDFs about twice a week. Not enough to pay for an Adobe subscription, but enough to be annoyed by online tools.</p>

<p>You know the drill:</p>

<p>Google "merge pdf free".</p>

<p>Upload your private bank statement to a random server.</p>

<p>Get hit with a "Daily Limit Reached" popup just when you need to download the result.</p>

<p>I got tired of this. I realized that modern browsers and WebAssembly are fast enough to handle PDF manipulation client-side. We don't actually need a backend for this anymore.</p>

<p>So, I built Simple VaultPDF.</p>

<p>The Tech Stack<br>
I wanted to keep it lightweight and 100% offline.</p>

<p>Framework: React (Vite)</p>

<p>PDF Engine: pdf-lib (for merging/splitting) &amp; pdf.js (for rendering)</p>

<p>OCR: tesseract.js</p>

<p>State: Just React hooks, kept it simple.</p>

<p>The Code (How it works)<br>
The coolest part is that pdf-lib lets you manipulate binary data directly in the browser memory. Here is the core function that merges files without sending a single byte to a server:</p>

<p>JavaScript<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import { PDFDocument } from 'pdf-lib';

const mergeFiles = async (files) =&gt; {
  const mergedPdf = await PDFDocument.create();

  for (const file of files) {
    const fileBuffer = await file.arrayBuffer();

    const pdf = await PDFDocument.load(fileBuffer);

    const copiedPages = await mergedPdf.copyPages(pdf, pdf.getPageIndices());

    copiedPages.forEach((page) =&gt; mergedPdf.addPage(page));
  }

  const pdfBytes = await mergedPdf.save();
  return pdfBytes;
};
</code></pre>

</div>



<p>Why Local-First?<br>
Building it as a Chrome Extension instead of a website had huge benefits:</p>

<p>Zero Server Costs: Since I don't process the files, I don't pay for AWS/Cloud functions.</p>

<p>Privacy: Users don't have to trust me with their data, because the data never leaves their machine.</p>

<p>Speed: No upload/download latency.</p>

<p>The Result<br>
It's currently an MVP. It handles merging, reordering, and rotation perfectly. I also added basic OCR because why not.</p>

<p>I just published it to the Chrome Web Store. If you are tired of paywalls or just want to see how pdf-lib performs in a real extension, give it a try.</p>

<p>Link: <a href="https://www.google.com/search?q=https://chromewebstore.google.com/detail/simple-vaultpdf/nefkedjebfockbphoninolplkhgpakoh%3Fhl%3Den%26utm_source%3Ddevto" rel="noopener noreferrer">Simple VaultPDF on Chrome Web Store</a></p>

<p>Let me know if you find any bugs! I'm still optimizing the memory usage for large files.</p>

<p>Happy coding!</p>

