---
Title: How to Add a Photo to an Existing Page in a PDF Using macOS Preview
Description: 
Author: Romain Lespinasse
Date: 2026-02-20T21:56:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you have ever tried to paste an image into a PDF using the macOS Preview app, you probably ran into a highly frustrating issue. </p>

<p>Instead of dropping the image onto the page you are currently viewing, Preview insists on pasting the image as a completely new, blank page. </p>

<p>Preview doesn't have a standard "Insert Image" button. However, there is a hidden workaround that lets you paste a photo as a movable, resizable object directly onto an existing PDF page. </p>

<h3>
  
  
  The "Paste-Into-Itself" Method
</h3>

<ol>
<li><p><strong>Open the image:</strong> Double-click your photo to open it in the Preview app.</p></li>
<li><p><strong>Copy the image:</strong> Press <strong>Command + A</strong> (⌘A) to select the entire image, then press <strong>Command + C</strong> (⌘C) to copy it to your clipboard.</p></li>
<li><p><strong>Paste it into itself:</strong> Without leaving your image window, press <strong>Command + V</strong> (⌘V) to paste. </p></li>
</ol>

<blockquote>
<p><strong>How to know it worked:</strong> This creates a duplicate "object" layer directly on top of your original image. The new layer will have a bounding box with little blue dots on the corners.</p>
</blockquote>

<ol>
<li>
<strong>Select and copy the new object:</strong> This is the most crucial step! Make sure you explicitly click on that newly pasted image layer so that those blue corner dots are actively selected. </li>
</ol>

<blockquote>
<p>Once it is selected, press <strong>Command + C</strong> (⌘C) to copy this specific <em>object</em>.</p>
</blockquote>

<ol>
<li><p><strong>Open your PDF:</strong> Switch over to the PDF document you want to edit (also opened in Preview).</p></li>
<li><p><strong>Select the target page:</strong> Click directly in the middle of the actual PDF page where you want the image to go. </p></li>
</ol>

<blockquote>
<p><strong>Warning:</strong> Do not click the thumbnail in the sidebar, or Preview will default back to creating a new page.</p>
</blockquote>

<ol>
<li>
<strong>Paste and adjust:</strong> Press <strong>Command + V</strong> (⌘V). </li>
</ol>




<p>Your image should now drop successfully onto your PDF page! </p>

<p>From there, you can click and drag it into position and use the blue corner dots to resize it to fit perfectly. </p>

<p>When you are finished, just hit <strong>Command + S</strong> (⌘S) to save your newly updated PDF.</p>

<blockquote>
<p>💡 Note<br>
This post was originally published on <a href="https://www.romainlespinasse.dev/posts/photo-in-pdf-macos-preview/" rel="noopener noreferrer">my personal blog</a>.</p>
</blockquote>

