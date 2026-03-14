---
Title: Practical Workflow for AI Pixel Art: Gemini 2D Animation Experiment
Description: 
Author: Kenji | Unity Game Dev
Date: 2026-03-14T21:56:29.000Z
Robots: noindex,nofollow
Template: index
---
<p>For this experiment, I summarized the workflow of using Gemini's image generation model, "Nano Banana 2," to generate sprite sheets for a 2D action game and implement them into Unity, along with tips to stabilize the output. To jump straight to the conclusion: the results were far more practical than expected.</p>

<p><iframe class="tweet-embed" id="tweet-2032170858035257577-889" src="https://platform.twitter.com/embed/Tweet.html?id=2032170858035257577">
</iframe>

  // Detect dark theme
  var iframe = document.getElementById('tweet-2032170858035257577-889');
  if (document.body.className.includes('dark-theme')) {
    iframe.src = "https://platform.twitter.com/embed/Tweet.html?id=2032170858035257577&amp;theme=dark"
  }



 </p>



<p><strong>【The Prompt Used】</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>//{Prompt}
//2D side-scrolling pixel art style (dot-picture).
//Only requires the side view; no other directions.
//Create a female sword-wielding protagonist with long red hair.
//The style should be RPG-like, a fusion of medieval and modern fashion, making it cute.
//The character naturally has red hair, which changes to platinum blonde when holding a sword. 
//Include a visual sequence for the transition from red to platinum blonde while drawing the sword.
//Must include the following actions:
//1. Idle Action
//2. Attack Actions (Combo 1,2,3)
//3. Walking/Runing Action
//4. Death Action
//5. Action of eating a steamed bun
//Pure black background for all of the above actions.
//Each action must have its corresponding animation frames.

</code></pre>

</div>






<p><strong>【Basic Action Generation and the Strength of the "Infinite Gacha"】</strong></p>

<p>First, regarding basic motions like idling and walking. In this regard, Gemini handles the task perfectly.</p>

<p>The biggest advantage of using Gemini is that you can endlessly "pull the gacha" (regenerate) up to 100 times a day until you are satisfied. The consistency of the character design is also exceptionally high; by continuing the chat, you can successively derive different action modules based on the same character.</p>

<p>Furthermore, based on a generated image, you can give instructions like, "Keep this part unchanged, but fix only this specific area." This makes it possible to repeatedly make minor adjustments while maintaining the overall design and vibe of the character. When creating a series of motions for the same character over the long term, this feature is indispensable.</p>




<p><strong>【Challenges and Limitations of Complex Attack Combos】</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffdwbwfkxzmz2wwpjktnl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffdwbwfkxzmz2wwpjktnl.png" alt=" " width="800" height="570"></a></p>

<blockquote>
<p>While simple movements are perfect, once you specify "complex and continuous actions" like attack combos, it suddenly becomes difficult to maintain movement consistency.</p>
</blockquote>

<p>Countermeasures and Compromises:</p>

<p>Instead of trying to generate complex actions in a single shot, break down the flow and give instructions frame by frame. <br>
Subdivide the prompts into steps like "raise the sword" and "swing down," and relentlessly regenerate (loop) like a gacha game until you pull an output that is closest to your ideal.</p>

<p>While there is still significant room for improvement in the overall fluidity of the animation, looking at it frame by frame reveals pixel art of more than sufficient quality. Sometimes, it even renders effects that can be used directly as VFX.</p>




<p><strong>Background Transparency Workflow (Black Background + Photoshop)</strong></p>

<p>A clear issue at this point is that the AI alone cannot directly output a "PNG image with a transparent background."</p>

<p>Countermeasure:<br>
Forcefully instruct the prompt to "make the background completely black (or a solid color like green)." After that, bring the generated image into a tool like Photoshop and manually cut out the black background to apply transparency. It takes a bit of extra effort, but this is the most reliable method at present.</p>




<p><strong>Summary</strong></p>

<p>Pros:</p>

<p>Simple movements (idle, walk, etc.) can be generated perfectly while maintaining consistency.</p>

<p>By continuing the chat, you can mass-produce derivative motions of the same character.<br>
Partial modification instructions are possible while preserving the base image.</p>

<p>Cons (Challenges):</p>

<p>It is difficult to maintain continuity for complex actions (combos, etc.), requiring frame-by-frame instructions and trial-and-error (gacha).<br>
Since it cannot output with a transparent background, specifying a solid color background and manually cutting it out with an external tool is mandatory.</p>




<p>While still in the trial-and-error stage, it already has sufficient potential as a tool to dramatically speed up prototype production. I plan to continue verifying and testing this going forward!</p>




<ul>
<li> <strong>X (Twitter):</strong> <a href="https://twitter.com/kenjiDev9662" rel="noopener noreferrer">@kenjiDev9662</a> (I post daily devlogs!)</li>
<li> <strong>Portfolio:</strong> <a href="https://kenji-gamedev.vercel.app" rel="noopener noreferrer">KenjiDev Portoflio</a>
</li>
<li> <strong>GitHub:</strong> <a href="https://github.com/Kenji966" rel="noopener noreferrer">Github@Kenji966</a>
</li>
<li> <strong>BLUESKY:</strong> <a href="https://bsky.app/profile/kenjidev9662.bsky.social" rel="noopener noreferrer">@kenjidev9662</a> (I post daily devlogs!)</li>
<li>🇯🇵 <strong>Japanese Post (Zenn):</strong> <a href="https://zenn.dev/kenji966/articles/9f6f3e43e0f007" rel="noopener noreferrer">Zenn@kenji966</a>
</li>
</ul>

