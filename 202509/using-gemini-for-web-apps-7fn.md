---
Title: Using Gemini for Web Apps
Description: 
Author: Lars Knudsen 🇩🇰
Date: 2025-09-23T21:45:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>For many years, I've been developing web applications using the web platform as is.</p>

<p>Recently, as many others, I've looked at what possibilities would be to use AI to help generate templates, components and even whole apps.</p>

<p>In this post, I've decided to let <a href="https://gemini.google.com/" rel="noopener noreferrer">Google's Gemini</a> generate everything for a small web demo, using web components and see how it goes.</p>

<h1>
  
  
  A simple data set
</h1>

<p>For data, I decided to ask Gemini to generate a JSON file containing a structured JSON of the top 100 classic novels:</p>

<blockquote>
<p>Prompt: "Create a structured JSON file, containing a list of the top 100 classic novels, including author, title, release year and a short synopsis"</p>
</blockquote>

<p>Within seconds, I received a JSON containing the following:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"classic_novels"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Homer"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"The Odyssey"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"release_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">-800</span><span class="p">,</span><span class="w">
      </span><span class="nl">"synopsis"</span><span class="p">:</span><span class="w"> </span><span class="s2">"The epic tale of Odysseus, king of Ithaca, and his ten-year journey home after the Trojan War, facing mythical creatures and wrathful gods."</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Mary Shelley"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Frankenstein"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"release_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">1818</span><span class="p">,</span><span class="w">
      </span><span class="nl">"synopsis"</span><span class="p">:</span><span class="w"> </span><span class="s2">"A young scientist, Victor Frankenstein, creates a sapient creature in an unorthodox scientific experiment, leading to tragic consequences for both."</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Jane Austen"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Pride and Prejudice"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"release_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">1813</span><span class="p">,</span><span class="w">
      </span><span class="nl">"synopsis"</span><span class="p">:</span><span class="w"> </span><span class="s2">"The story of Elizabeth Bennet, who navigates issues of manners, upbringing, morality, education, and marriage in the society of the landed gentry of early 19th-century England."</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"George Orwell"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1984"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"release_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">1949</span><span class="p">,</span><span class="w">
      </span><span class="nl">"synopsis"</span><span class="p">:</span><span class="w"> </span><span class="s2">"In a dystopian superstate ruled by the totalitarian Party, Winston Smith secretly defies the regime, an act that leads him to question everything he knows."</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"author"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Harper Lee"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"title"</span><span class="p">:</span><span class="w"> </span><span class="s2">"To Kill a Mockingbird"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"release_year"</span><span class="p">:</span><span class="w"> </span><span class="mi">1960</span><span class="p">,</span><span class="w">
      </span><span class="nl">"synopsis"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Narrated by a young girl named Scout Finch, this novel explores themes of racial injustice and the loss of innocence in the American South during the Great Depression."</span><span class="w">
    </span><span class="p">},</span><span class="w">
</span><span class="err">...</span><span class="w">
</span></code></pre>

</div>



<h1>
  
  
  Making it visible
</h1>

<p>Then Asking Gemini to generate a standalone HTML page, where it's possible to load the JSON and show it using only web standards, again within seconds, the output was presented:</p>

<blockquote>
<p>Prompt: "now make a standalone web page (HTML), where it's possible to load this file and display it in a table, where it's possible for the user to sort by author, title or release year. No external dependencies should be used and the items should be shown as a card, rendered like an old school library card - using web components but only using pure web standards"</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjizf23kr4oh0iumdyko1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjizf23kr4oh0iumdyko1.png" alt="Gemini HTML1" width="800" height="456"></a></p>

<p>The web view:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp1vtqsqeucie7epfhbfs.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp1vtqsqeucie7epfhbfs.png" alt=" " width="800" height="606"></a></p>

<p>Sorting works as expected - very cool :)</p>

<p>I then asked Gemini to modify the HTML to add a view transition on clicking a card to bring it into focus and again, within seconds the result was there:</p>

<blockquote>
<p>Prompt: "can you change the html, so when a card is clicked, it will make a view transition to fill the screen and when clicked again, it will go back in the list"</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/PeFyCgTvWUs">
</iframe>
</p>

<h1>
  
  
  Looking at the code
</h1>

<p>When looking through the produced HTML, it's nice to see how comments explain the reasoning behind functionality that I didn't explicitly ask for, but makes sense in the context of - in this case - visualization of publishing year:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="c1">// Helper to format year (e.g., -800 -&gt; 800 BCE)</span>
<span class="nf">formatYear</span><span class="p">(</span><span class="nx">year</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="nx">year</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="s2">`</span><span class="p">${</span><span class="nb">Math</span><span class="p">.</span><span class="nf">abs</span><span class="p">(</span><span class="nx">year</span><span class="p">)}</span><span class="s2"> BCE`</span><span class="p">;</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="nx">year</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>It's also nice to see how Gemini has chosen to make a sort function to handle both text (title and author) and numbers (publishing year):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">function</span> <span class="nf">sortData</span><span class="p">(</span><span class="nx">key</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">novelsData</span> <span class="o">||</span> <span class="nx">novelsData</span><span class="p">.</span><span class="nx">length</span> <span class="o">===</span> <span class="mi">0</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

    <span class="nx">novelsData</span><span class="p">.</span><span class="nf">sort</span><span class="p">((</span><span class="nx">a</span><span class="p">,</span> <span class="nx">b</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
        <span class="k">if </span><span class="p">(</span><span class="k">typeof</span> <span class="nx">a</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">string</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
            <span class="c1">// Case-insensitive string comparison</span>
            <span class="k">return</span> <span class="nx">a</span><span class="p">[</span><span class="nx">key</span><span class="p">].</span><span class="nf">localeCompare</span><span class="p">(</span><span class="nx">b</span><span class="p">[</span><span class="nx">key</span><span class="p">]);</span>
        <span class="p">}</span> <span class="k">else</span> <span class="k">if </span><span class="p">(</span><span class="k">typeof</span> <span class="nx">a</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">number</span><span class="dl">'</span><span class="p">)</span> <span class="p">{</span>
            <span class="c1">// Numeric comparison</span>
            <span class="k">return</span> <span class="nx">a</span><span class="p">[</span><span class="nx">key</span><span class="p">]</span> <span class="o">-</span> <span class="nx">b</span><span class="p">[</span><span class="nx">key</span><span class="p">];</span>
        <span class="p">}</span>
        <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
    <span class="p">});</span>
<span class="p">}</span>
</code></pre>

</div>



<p>For browsers not supporting the View Transition API, there is even a fallback:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="nx">cardContainer</span><span class="p">.</span><span class="nf">addEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">click</span><span class="dl">'</span><span class="p">,</span> <span class="p">(</span><span class="nx">event</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">clickedCard</span> <span class="o">=</span> <span class="nx">event</span><span class="p">.</span><span class="nx">target</span><span class="p">.</span><span class="nf">closest</span><span class="p">(</span><span class="dl">'</span><span class="s1">library-card</span><span class="dl">'</span><span class="p">);</span>
    <span class="k">if </span><span class="p">(</span><span class="o">!</span><span class="nx">clickedCard</span><span class="p">)</span> <span class="k">return</span><span class="p">;</span>

    <span class="kd">const</span> <span class="nx">isFocused</span> <span class="o">=</span> <span class="nx">clickedCard</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nf">contains</span><span class="p">(</span><span class="dl">'</span><span class="s1">focused</span><span class="dl">'</span><span class="p">);</span>

    <span class="c1">// Use the View Transitions API</span>
    <span class="k">if </span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">startViewTransition</span><span class="p">)</span> <span class="p">{</span>
        <span class="nb">document</span><span class="p">.</span><span class="nf">startViewTransition</span><span class="p">(()</span> <span class="o">=&gt;</span> <span class="p">{</span>
            <span class="nf">toggleFocus</span><span class="p">(</span><span class="nx">clickedCard</span><span class="p">,</span> <span class="o">!</span><span class="nx">isFocused</span><span class="p">);</span>
        <span class="p">});</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="nf">toggleFocus</span><span class="p">(</span><span class="nx">clickedCard</span><span class="p">,</span> <span class="o">!</span><span class="nx">isFocused</span><span class="p">);</span>
    <span class="p">}</span>
<span class="p">});</span>
</code></pre>

</div>



<p>In general, the produced JavaScript, CSS and HTML gets the job done and I am impressed.</p>

<h1>
  
  
  Making a game
</h1>

<p>After seeing how well Gemini handled the task, I wanted to give it a slightly harder challenge: To create a small game with keyboard input, audio and all.</p>

<blockquote>
<p>Prompt: "Using pure web standards, create a small game in a standalone HTML file, where it's possible to control a car seen from above, using the arrow keys on the keyboard. Also include realistic generated sound effects using web audio. It should be possible to "drive" the car around on the screen, collecting coins randomly placed. There should be a score number accumulated in the top right corner"</p>
</blockquote>

<p>Within seconds, it delivered:</p>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/_MVXFKKE5-k">
</iframe>
</p>

<p><strong>NOTE: Sound was not captured in the video, but you can try out the game <a href="https://larsgk.github.io/geminiwebtest/car_game.html" rel="noopener noreferrer">here</a></strong></p>

<h1>
  
  
  Conclusion
</h1>

<p>It was quite fun and impressive to see how easy it was to create these fully functional web demos with a few simple requests to Gemini. Even if the output may not be exactly what was intended, the code is well structured and easy to read and modify. </p>

<p>The code produced can be found <a href="https://github.com/larsgk/geminiwebtest" rel="noopener noreferrer">here</a></p>

<p>Enjoy :) </p>

