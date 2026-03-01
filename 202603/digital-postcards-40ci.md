---
Title: Digital Postcards
Description: 
Author: vasilis-skourtis-dev
Date: 2026-03-01T21:45:09.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/weekend-2026-02-28">DEV Weekend Challenge: Community</a></em></p>

<h2>
  
  
  The Community
</h2>

<p>A couple, a family, a class, a team, or an organization can all be a community. The primary inspiration for this project, however, is the parent-child relationship. It allows parents to pass down messages to their offspring, preserving them within digital photos instead of crystals; much like how Jor-El prepared Kal-El (Superman) for his journey to Earth.</p>

<p>Ultimately, the largest community this application serves is humanity itself. Countless use cases can emerge across various groups, such as sending beautiful, encouraging messages to strangers. Like a digital "message in a bottle," these postcards can reach someone you've never met, bringing a smile to their face and making their day.</p>

<h2>
  
  
  What I Built
</h2>

<p><strong>Digital Postcards</strong> is a web app that lets people across various communities exchange beautiful digital postcards with hidden secret messages baked invisibly into the image via steganography. To non-community members, the image looks like a regular photo, keeping the true message hidden in plain sight.</p>

<p>Close your eyes for a moment. What do you remember from your childhood? When the world turns cold and difficult, what is the memory that keeps you warm, kind, and fiercely tolerant? If you search for the root of your true inner strength, you will rarely find a fleeting notification or a quick text message. Instead, you will find beautiful words from beautiful people—the quiet encouragement of those who loved and cared for you, preserved like ink on paper.</p>

<p>This project was built to capture and protect that exact feeling. It is an application designed to help you express your deepest compassion and support to your beloved ones, transforming fleeting thoughts into eternal digital keepsakes. By stripping away the noise of modern apps, we brought back the intentionality of a physical postcard: just seven carefully chosen lines, the nostalgic weight of handwriting, and a quiet space to say, "I am thinking of you."</p>

<p>But true vulnerability requires a safe harbor. Because the most meaningful words are often the most private, we wove the ancient art of steganography into the very pixels of the digital stamp. This allows you to encode a hidden message entirely invisible to the naked eye. To the outside world, it is simply a beautiful, vintage greeting. But to the one who receives it, it unlocks a secret whisper—ensuring that externals cannot view the message, and only your recipient learns exactly what you hold in your heart.</p>

<h3>
  
  
  Creating a Postcard
</h3>

<ol>
<li>Go to the <strong>Create</strong> page</li>
<li>Upload any cover photo — a landscape, a portrait, a memory</li>
<li>The card flips to the back — write your secret message and give it a title</li>
<li>Click the stamp (<strong>CREATE</strong>) — the message is encoded invisibly into the image using LSB steganography</li>
<li>A PNG is downloaded — it looks like your photo, but it carries your words inside</li>
<li>Share it anywhere: email, chat, social media</li>
</ol>

<h3>
  
  
  Reading a Postcard
</h3>

<ol>
<li>On the home page, an animated sealed envelope waits</li>
<li>Click the <strong>wax seal</strong> and upload the received image</li>
<li>The app decodes any hidden message from the pixel data</li>
<li>The postcard opens — the card flips automatically to reveal the message on the back</li>
</ol>




<p>The visuals lean fully into the physical postcard metaphor:</p>

<ul>
<li>An <strong>animated envelope</strong> with a wax seal on the home page</li>
<li>A <strong>flippable dual-sided card</strong> on the create and view pages</li>
<li>A <strong>stamp-style CREATE button</strong> on the back of the card</li>
</ul>

<h2>
  
  
  Demo
</h2>

<ul>
<li>
<strong>Live app:</strong> <a href="https://postcards.guardanangel.org" rel="noopener noreferrer">postcards.guardanangel.org</a>
</li>
</ul>

<h2>
  
  
  Code
</h2>

<p><a href="https://github.com/vasilis-skourtis-dev/devto-community-contests/tree/main/digital-postcards-mini-app" rel="noopener noreferrer">Repo</a></p>

<h2>
  
  
  How I Built It
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Technology</th>
</tr>
</thead>
<tbody>
<tr>
<td>Backend</td>
<td>Java 8, Spring Boot 2.7</td>
</tr>
<tr>
<td>Templating</td>
<td>Thymeleaf</td>
</tr>
<tr>
<td>Steganography</td>
<td>Pure Java — LSB on RGB pixel channels</td>
</tr>
<tr>
<td>Frontend</td>
<td>HTML, CSS (3D transforms, envelope animation), vanilla JS</td>
</tr>
<tr>
<td>Build</td>
<td>Maven</td>
</tr>
<tr>
<td>Runtime</td>
<td>Embedded Tomcat, port 9090</td>
</tr>
<tr>
<td>Database</td>
<td>None — fully stateless, all processing in-memory</td>
</tr>
</tbody>
</table></div>

<p>Demo Digital Postcard</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fluzdfjbiitlmfjt4rxiy.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fluzdfjbiitlmfjt4rxiy.png" alt=" " width="800" height="482"></a></p>

<p>Post Card View --&gt;<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhyq99wz8ldm7tyg9wqjh.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhyq99wz8ldm7tyg9wqjh.gif" alt=" " width="600" height="410"></a></p>

<p>Post Card Creation --&gt;<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl9hlriz4g2gdna8e3tgg.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl9hlriz4g2gdna8e3tgg.gif" alt=" " width="600" height="345"></a></p>

