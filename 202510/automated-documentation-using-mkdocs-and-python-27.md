---
Title: 🧩 Automated Documentation using MkDocs and Python
Description: 
Author: Alexandre Fernandes dos Santos
Date: 2025-10-27T21:39:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>Something I really enjoy is browsing through library documentation — honestly, good documentation is something I find beautiful to see and also difficult to maintain 😅.</p>

<p>Now imagine: every time a change is made to a behavior, having to go to the page that explains it, copy and paste the code, write what changed, run the build, check if everything is correct... wait, that's not how it should be! — I'm already tired just thinking about it 😤.</p>

<h2>
  
  
  🐛 Tool to be documented
</h2>

<p>I created a very simple tool that makes a call to the PokeAPI, fetches information about a Pokémon, and displays the result in the terminal.</p>

<p>The source code can be found here: <a href="https://github.com/XandeCoding/codigos-de-artigos/tree/main/python/documentacao_automatica" rel="noopener noreferrer">Automated Documentation</a></p>

<p>Talking a bit more about the project: I used Poetry to manage the virtual environment and keep the libraries isolated from the rest of the system (nobody wants to clutter their PC with libs, right?).<br>
But you can use any other tool, like pyenv or virtualenvwrapper — it doesn't matter.</p>

<p>The installed libraries are listed in pyproject.toml, so just install them and start playing.</p>
<h2>
  
  
  📝 How to document
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F54xr5ms28n2irn7x3eia.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F54xr5ms28n2irn7x3eia.png" alt="Pokémon API access code PokeAPI" width="800" height="711"></a></p>

<p>This is a small piece of the code I'll use as an example.<br>
You must have noticed those """ — they are Docstrings, used to document code. It's a very common convention for documenting modules, classes, functions... well, anything.<br>
If you're going to document something, I highly recommend using docstrings 😜.</p>

<p>These docstrings are what make the magic happen: we'll make them be automatically read and inserted into the corresponding documentation pages.<br>
With the docstrings ready and explaining what each function does, we can proceed.</p>
<h2>
  
  
  ⬆️ Installing MkDocs
</h2>

<p>To install using Poetry it's very simple — just use poetry add, or pip install if you're not using it.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb6lb7pla7ap4x8f01aiu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb6lb7pla7ap4x8f01aiu.png" alt="Code showing how to install mkdocs" width="800" height="391"></a></p>

<p><strong><em>Note:</em></strong> When installing mkdocstrings, replace [python] with the language you're using.<br>
Check the official documentation to verify support: Mkdocstrings Docs.</p>

<p>We installed MkDocs, a nice theme called mkdocs-material (a matter of taste, but I really like it 😄), and finally mkdocstrings, a plugin that scans the project and inserts the docstrings into the corresponding pages.</p>

<p>So now? What do we do with all this? 🤔 Don't worry, you don't need to send a letter to a TV show — it's much simpler than that 😂.<br>
Following the step-by-step below, your documentation will be ready in no time:</p>
<h3>
  
  
  Step 1️⃣
</h3>

<p>Open a terminal inside the project folder and use the command:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mkdocs new <span class="nb">.</span>
</code></pre>

</div>



<p>This will create the configuration files and the docs folder.</p>

<h3>
  
  
  Step 2️⃣
</h3>

<p>Now you should have a file called mkdocs.yml in the project root.<br>
In it, add the following configurations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code>    <span class="na">site_name</span><span class="pi">:</span> <span class="s">Automated Documentation</span> <span class="c1"># You can use your application's name</span>

    <span class="na">theme</span><span class="pi">:</span>
        <span class="na">name</span><span class="pi">:</span> <span class="s">material</span> <span class="c1"># Adds the nice theme</span>

        <span class="na">plugins</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">search</span> <span class="c1"># Plugin that enables search in the documentation</span>
        <span class="pi">-</span> <span class="na">mkdocstrings</span><span class="pi">:</span> <span class="c1"># This one makes the magic happen!</span>
            <span class="na">default_handler</span><span class="pi">:</span> <span class="s">python</span>
</code></pre>

</div>



<p><strong><em>Note:</em></strong> Yeah, the code looks a bit ugly, but this way you can just copy and paste — my laziness salutes yours 🙏</p>

<p>The mkdocstrings plugin is responsible for reading the docstrings and automatically adding them to the corresponding pages.</p>

<h3>
  
  
  Step 3️⃣
</h3>

<p>Run the commands:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>mkdocs build
mkdocs serve
</code></pre>

</div>



<p>Then, access the documentation through your browser at:<br>
👉 <a href="http://127.0.0.1:8000/" rel="noopener noreferrer">http://127.0.0.1:8000/</a></p>

<p>You'll see something similar to the image below:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2fa6vl45xhc8h9rkxmmm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2fa6vl45xhc8h9rkxmmm.png" alt="Documentation page index" width="800" height="438"></a></p>
<h3>
  
  
  Step 4️⃣
</h3>

<p>When you access it, you'll see an introduction explaining how to add pages — it's very simple!<br>
Just create a .md file inside the docs folder in the project root, and it will be automatically added to the documentation structure.</p>

<p>Your structure should look similar to this:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft2s6pnr4eigb1gbevq5c.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft2s6pnr4eigb1gbevq5c.png" alt="Folder structure" width="501" height="1050"></a></p>
<h3>
  
  
  Step 5️⃣
</h3>

<p>Now let's go to the pokemon page, used to document the pokemon module we saw at the beginning.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhqa6ko5t0c5laltywr7f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhqa6ko5t0c5laltywr7f.png" alt="Documentation markdown file" width="800" height="363"></a></p>

<p>In the page content, we have a brief introduction and then something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code>::: src.pokemon
</code></pre>

</div>



<p>But what is this?<br>
This command maps the module to be automatically documented — the path is the same one used in the import.<br>
Don't believe it works? Take a look at the result:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flh4ypym2c79vd456cu7d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flh4ypym2c79vd456cu7d.png" alt="Pokémon module documentation" width="800" height="438"></a></p>

<h3>
  
  
  Step 6️⃣
</h3>

<p>Enjoy! 😁</p>

<h2>
  
  
  🎉 The End
</h2>

<p>In just a few steps, we already have pretty nice documentation!<br>
If you add new modules or want to create more pages, just include the file inside the docs folder, and you're done.</p>

<p>I hope you enjoyed it!<br>
And if you have any questions (or found any errors — it happens to the best of us 😅), just leave a comment below!</p>

