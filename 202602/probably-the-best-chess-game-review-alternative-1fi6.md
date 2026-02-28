---
Title: Probably the best chess game review Alternative
Description: 
Author: True & Falls
Date: 2026-02-28T21:37:11.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F22i6x7bwowdql2vwksmh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F22i6x7bwowdql2vwksmh.png" alt="Game Review Button Picture" width="474" height="72"></a></p>

<p>Recently I decided to build chess.com game review alternative check demo - <a href="https://povchess.com" rel="noopener noreferrer">povchess.com</a>.</p>

<h2>
  
  
  What is Game Review ?
</h2>

<p>Chess engine analyze two different position before and after. And then compare <strong>"best found move"</strong> to actually <strong>"played move"</strong> by player and return evaluation difference as the number.</p>

<p>Engine return evaluation of the position as the number for example:<br>
+3 (means 3 centipawns advantage for white)<br>
-3 (means 3 centipawns advantage for black)</p>

<p>Using these numbers and difference between them allows to give classification for each of the moves.</p>

<h2>
  
  
  Classification
</h2>

<p><strong>Best Move</strong> - if the evaluation difference between the position before and after the move is 0 or very close to 0, it means the most accurate move was played.</p>

<p><strong>Blunder</strong> - if the evaluation difference is more than 200 centipawns (2.00 in engine terms), the move is considered a blunder.</p>

<p><strong>Brilliant</strong> - to be classified as brilliant, a move should fulfill a couple of conditions:</p>

<ol>
<li>It is the best move.</li>
<li>It is not obviously easy to find.</li>
</ol>

<p>(For example, it may involve sacrificing material such as a queen or a rook)</p>

<p><strong>Book move</strong> - a move is classified as a book move if it is found in a prepared database of known openings.</p>

<p>Basically, we analyze all positions and compare evaluations before and after each move.</p>

<h2>
  
  
  Why ?
</h2>

<p>First, to make it free for players who cannot pay for a premium subscription to review a game. Nowadays, such a tool can be created by anyone using open-source libraries and free tools, so it should be accessible to everyone.</p>

<p>Second, it's a space for creativity and improvement. Many ideas and tools can be built for chess that make analysis more useful and help players improve faster.</p>

<h2>
  
  
  So ?
</h2>

<p>There are few tasks I have:</p>

<ol>
<li>Better UI/UX</li>
<li>Most accurate classification</li>
<li>Increase speed of evaluation</li>
<li>Explanations</li>
<li>Stability</li>
</ol>

<p>I plan to continue work and improve my project until its gonna finally replace chess.com and lichess. If you have any thoughts or ideas please share it in the comments with me.</p>

