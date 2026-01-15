---
Title: Building a Neon-Soaked Last.fm Roast Bot for $1.75/Year
Description: 
Author: Paige Bailey
Date: 2026-01-15T22:04:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>We all have that one song in our listening history that we hope nobody sees. But instead of hiding my shame, I decided to automate it!</p>

<p>I recently built my own custom AI bot on <strong>Poe</strong> called <code>lastfm-roaster</code>. Its only job is to look at a person's Last.fm music taste (if you're curious. <a href="https://www.last.fm/user/profoundlypaige" rel="noopener noreferrer">here's mine</a> as an example) and absolutely destroy them.</p>

<p><strong>But there was a problem:</strong> The roasts were trapped in the chatbot interface. I wanted them delivered straight to my inbox every morning so I could easily forward the best (worst) burns to my friends for a laugh.</p>

<p>So, I built a pipeline.</p>

<p>Today, I’m going to show you how I connected my custom Poe bot to a Python script that pings the API, analyzes my listening history, and uses <strong>Regex</strong> to inject random neon colors into a beautiful HTML email.</p>

<p>Best of all? It runs entirely for free on GitHub Actions, and the API costs less than a cup of coffee <strong>for the entire year</strong>.</p>

<p>Onward!</p>

<h2>
  
  
  Building the Bot 🤖
</h2>

<p>I went to <strong>Poe</strong> and created a new bot using their <strong>ScriptBot</strong> feature. I gave it a specific system prompt roughly like this:</p>

<blockquote>
<p>"You are a pretentious music critic. Your job is to analyze Last.fm profiles and roast them mercilessly. Be sarcastic, use slang, and do not hold back."</p>
</blockquote>

<p>After iterating back and forth with the ScriptBot until I was satisfied with the results, I had the intelligence (<code>lastfm-roaster</code>), but I needed to get the output out of Poe and into my email.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr9hkgsx204zjeuwlnzz2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr9hkgsx204zjeuwlnzz2.png" alt=" " width="800" height="270"></a></p>

<h2>
  
  
  Keys and Secrets 🗝️
</h2>

<p>We are going to use GitHub Actions to run this, which means we need to keep our API keys safe. <strong>Never hardcode passwords in your script!</strong></p>

<h3>
  
  
  1. Get the Keys
</h3>

<ul>
<li>  <strong>Poe API Key:</strong> Go to <a href="https://poe.com/api_key" rel="noopener noreferrer">poe.com/api_key</a> and grab your key.</li>
<li>  <strong>Gmail App Password:</strong> To send email via Python, you can't use your normal password. Go to your Google Account &gt; Security &gt; 2-Step Verification &gt; <strong>App Passwords</strong>. Generate one and save that 16-character code.</li>
</ul>

<h3>
  
  
  2. Store them in GitHub
</h3>

<p>Create a new private repository on GitHub, then go to <strong>Settings &gt; Secrets and variables &gt; Actions</strong>.</p>

<p>Add these three secrets:</p>

<ul>
<li>  <code>POE_API_KEY</code>
</li>
<li>  <code>EMAIL_ADDRESS</code>
</li>
<li>  <code>EMAIL_PASSWORD</code>
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqfof1p1hxewda2gk55oc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqfof1p1hxewda2gk55oc.png" alt=" " width="800" height="212"></a></p>

<h2>
  
  
  Step 3: The Code 🐍
</h2>

<p>We need a few Python libraries to make this magic happen. Create a file named <code>requirements.txt</code> in your repo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>openai
markdown
</code></pre>

</div>



<p><em>(Yes, we use <code>openai</code>! Poe's API is now compatible with the OpenAI client, making it super easy to use. Gemini 2.5 Flash is referenced via the Poe API, so I don't have to worry about managing that key, as well.)</em></p>

<h3>
  
  
  The "Neon" Script (<code>lastfm_roast.py</code>)
</h3>

<p>Here is the cool part. I didn't want the email to look boring. I wanted the "insults" (the bold text) to pop in different colors.</p>

<p>We use Python's <code>re</code> (Regex) and <code>itertools</code> to find every <code>&lt;strong&gt;</code> tag my bot generates and cycle through a "Dracula" color palette to inject inline CSS styles.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">os</span>
<span class="kn">import</span> <span class="n">smtplib</span>
<span class="kn">import</span> <span class="n">markdown</span>
<span class="kn">import</span> <span class="n">re</span>
<span class="kn">import</span> <span class="n">itertools</span>
<span class="kn">from</span> <span class="n">email.message</span> <span class="kn">import</span> <span class="n">EmailMessage</span>
<span class="kn">from</span> <span class="n">openai</span> <span class="kn">import</span> <span class="n">OpenAI</span>

<span class="c1"># Configs (Loaded safely from GitHub Secrets)
</span><span class="n">POE_API_KEY</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">POE_API_KEY</span><span class="sh">"</span><span class="p">)</span>
<span class="n">EMAIL_ADDRESS</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">EMAIL_ADDRESS</span><span class="sh">"</span><span class="p">)</span>
<span class="n">EMAIL_PASSWORD</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">EMAIL_PASSWORD</span><span class="sh">"</span><span class="p">)</span>
<span class="n">LASTFM_URL</span> <span class="o">=</span> <span class="sh">"</span><span class="s">https://www.last.fm/user/profoundlypaige</span><span class="sh">"</span>

<span class="c1"># --- NEON PALETTE ---
# A list of bright colors that look good on dark backgrounds
# (Pink, Cyan, Green, Orange, Purple, Yellow)
</span><span class="n">COLORS</span> <span class="o">=</span> <span class="p">[</span><span class="sh">"</span><span class="s">#FF79C6</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">#8BE9FD</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">#50FA7B</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">#FFB86C</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">#BD93F9</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">#F1FA8C</span><span class="sh">"</span><span class="p">]</span>

<span class="k">def</span> <span class="nf">get_roast</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Pings the Poe API to get the roast.</span><span class="sh">"""</span>
    <span class="n">client</span> <span class="o">=</span> <span class="nc">OpenAI</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">POE_API_KEY</span><span class="p">,</span> <span class="n">base_url</span><span class="o">=</span><span class="sh">"</span><span class="s">https://api.poe.com/v1</span><span class="sh">"</span><span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">🔥 Fetching roast from Poe...</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">chat</span><span class="p">.</span><span class="n">completions</span><span class="p">.</span><span class="nf">create</span><span class="p">(</span>
            <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">lastfm-roaster</span><span class="sh">"</span><span class="p">,</span>
            <span class="n">messages</span><span class="o">=</span><span class="p">[</span>
                <span class="p">{</span><span class="sh">"</span><span class="s">role</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">user</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="p">:</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Roast my music taste: </span><span class="si">{</span><span class="n">LASTFM_URL</span><span class="si">}</span><span class="sh">"</span><span class="p">}</span>
            <span class="p">]</span>
        <span class="p">)</span>
        <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">choices</span><span class="p">[</span><span class="mi">0</span><span class="p">].</span><span class="n">message</span><span class="p">.</span><span class="n">content</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="k">return</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Error fetching roast: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span>

<span class="k">def</span> <span class="nf">inject_colors</span><span class="p">(</span><span class="n">html_content</span><span class="p">):</span>
    <span class="sh">"""</span><span class="s">
    Finds every &lt;strong&gt; tag and injects a different color from the palette.
    </span><span class="sh">"""</span>
    <span class="n">color_cycle</span> <span class="o">=</span> <span class="n">itertools</span><span class="p">.</span><span class="nf">cycle</span><span class="p">(</span><span class="n">COLORS</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">replace_match</span><span class="p">(</span><span class="n">match</span><span class="p">):</span>
        <span class="n">next_color</span> <span class="o">=</span> <span class="nf">next</span><span class="p">(</span><span class="n">color_cycle</span><span class="p">)</span>
        <span class="c1"># Returns &lt;strong style="color: #CODE"&gt;
</span>        <span class="k">return</span> <span class="sa">f</span><span class="sh">'</span><span class="s">&lt;strong style=</span><span class="sh">"</span><span class="s">color: </span><span class="si">{</span><span class="n">next_color</span><span class="si">}</span><span class="sh">"</span><span class="s">&gt;</span><span class="sh">'</span>

    <span class="c1"># Regex to replace &lt;strong&gt; with the colored version
</span>    <span class="k">return</span> <span class="n">re</span><span class="p">.</span><span class="nf">sub</span><span class="p">(</span><span class="sa">r</span><span class="sh">'</span><span class="s">&lt;strong&gt;</span><span class="sh">'</span><span class="p">,</span> <span class="n">replace_match</span><span class="p">,</span> <span class="n">html_content</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">create_html_email</span><span class="p">(</span><span class="n">roast_text</span><span class="p">):</span>
    <span class="c1"># 1. Convert Markdown to basic HTML
</span>    <span class="n">raw_html</span> <span class="o">=</span> <span class="n">markdown</span><span class="p">.</span><span class="nf">markdown</span><span class="p">(</span><span class="n">roast_text</span><span class="p">)</span>

    <span class="c1"># 2. Inject the rotating neon colors into bold tags
</span>    <span class="n">colorful_html</span> <span class="o">=</span> <span class="nf">inject_colors</span><span class="p">(</span><span class="n">raw_html</span><span class="p">)</span>

    <span class="c1"># 3. Wrap in the styled container
</span>    <span class="n">html_template</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"""</span><span class="s">
    &lt;!DOCTYPE html&gt;
    &lt;html&gt;
    &lt;head&gt;
    &lt;style&gt;
        body {{ margin: 0; padding: 0; background-color: #121212; font-family: </span><span class="sh">'</span><span class="s">Helvetica Neue</span><span class="sh">'</span><span class="s">, Helvetica, Arial, sans-serif; }}

        .container {{ 
            max-width: 600px; 
            margin: 40px auto; 
            background-color: #1e1e1e; 
            border-radius: 16px; 
            overflow: hidden; 
            box-shadow: 0 10px 30px rgba(0,0,0,0.5); 
            border: 1px solid #333;
        }}

        .header {{ 
            background: linear-gradient(135deg, #2b2b2b 0%, #1a1a1a 100%); 
            padding: 30px; 
            text-align: center; 
            border-bottom: 2px solid #333;
        }}

        /* The title is now a gradient text effect */
        .header h1 {{ 
            margin: 0; font-size: 28px; letter-spacing: 2px; text-transform: uppercase; 
            background: -webkit-linear-gradient(#FF79C6, #8BE9FD);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }}

        .content {{ padding: 30px; color: #d1d5db; line-height: 1.7; font-size: 16px; }}

        h2 {{ 
            color: #ffffff; 
            border-left: 5px solid #BD93F9; /* Purple accent */
            padding-left: 15px;
            margin-top: 30px; 
            text-transform: uppercase;
            font-size: 18px;
            letter-spacing: 1px;
        }}

        ul {{ padding-left: 20px; }}
        li {{ margin-bottom: 10px; }}

        /* Styles for links */
        a {{ color: #8BE9FD; text-decoration: none; border-bottom: 1px dotted #8BE9FD; }}

        .footer {{ 
            background-color: #121212; 
            padding: 20px; 
            text-align: center; 
            font-size: 12px; 
            color: #555; 
        }}
    &lt;/style&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;div class=</span><span class="sh">"</span><span class="s">container</span><span class="sh">"</span><span class="s">&gt;
            &lt;div class=</span><span class="sh">"</span><span class="s">header</span><span class="sh">"</span><span class="s">&gt;
                &lt;h1&gt;🔥 The Daily Burn&lt;/h1&gt;
            &lt;/div&gt;
            &lt;div class=</span><span class="sh">"</span><span class="s">content</span><span class="sh">"</span><span class="s">&gt;
                </span><span class="si">{</span><span class="n">colorful_html</span><span class="si">}</span><span class="s">
            &lt;/div&gt;
            &lt;div class=</span><span class="sh">"</span><span class="s">footer</span><span class="sh">"</span><span class="s">&gt;
                Served fresh by Poe API, Gemini 2.5 Flash, &amp; GitHub Actions&lt;br&gt;
                &lt;a href=</span><span class="sh">"</span><span class="si">{</span><span class="n">LASTFM_URL</span><span class="si">}</span><span class="sh">"</span><span class="s"> style=</span><span class="sh">"</span><span class="s">color:#555; border:none;</span><span class="sh">"</span><span class="s">&gt;View your tragic Last.fm Profile&lt;/a&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    &lt;/body&gt;
    &lt;/html&gt;
    </span><span class="sh">"""</span>
    <span class="k">return</span> <span class="n">html_template</span>

<span class="k">def</span> <span class="nf">send_email</span><span class="p">(</span><span class="n">roast_text</span><span class="p">):</span>
    <span class="n">msg</span> <span class="o">=</span> <span class="nc">EmailMessage</span><span class="p">()</span>
    <span class="n">msg</span><span class="p">[</span><span class="sh">"</span><span class="s">Subject</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="sh">"</span><span class="s">Your Daily Last.fm Roast 🎸</span><span class="sh">"</span>
    <span class="n">msg</span><span class="p">[</span><span class="sh">"</span><span class="s">From</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">EMAIL_ADDRESS</span>
    <span class="n">msg</span><span class="p">[</span><span class="sh">"</span><span class="s">To</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">EMAIL_ADDRESS</span>
    <span class="n">msg</span><span class="p">.</span><span class="nf">set_content</span><span class="p">(</span><span class="n">roast_text</span><span class="p">)</span>
    <span class="n">msg</span><span class="p">.</span><span class="nf">add_alternative</span><span class="p">(</span><span class="nf">create_html_email</span><span class="p">(</span><span class="n">roast_text</span><span class="p">),</span> <span class="n">subtype</span><span class="o">=</span><span class="sh">'</span><span class="s">html</span><span class="sh">'</span><span class="p">)</span>

    <span class="k">try</span><span class="p">:</span>
        <span class="k">with</span> <span class="n">smtplib</span><span class="p">.</span><span class="nc">SMTP_SSL</span><span class="p">(</span><span class="sh">"</span><span class="s">smtp.gmail.com</span><span class="sh">"</span><span class="p">,</span> <span class="mi">465</span><span class="p">)</span> <span class="k">as</span> <span class="n">smtp</span><span class="p">:</span>
            <span class="n">smtp</span><span class="p">.</span><span class="nf">login</span><span class="p">(</span><span class="n">EMAIL_ADDRESS</span><span class="p">,</span> <span class="n">EMAIL_PASSWORD</span><span class="p">)</span>
            <span class="n">smtp</span><span class="p">.</span><span class="nf">send_message</span><span class="p">(</span><span class="n">msg</span><span class="p">)</span>
        <span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">✅ Email sent successfully!</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">❌ Failed to send email: </span><span class="si">{</span><span class="n">e</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">roast</span> <span class="o">=</span> <span class="nf">get_roast</span><span class="p">()</span>
    <span class="nf">send_email</span><span class="p">(</span><span class="n">roast</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  Automating with GitHub Actions 🤖
</h2>

<p>I don't want to run this manually -- I want to be roasted automatically. So we'll use a GitHub Actions workflow to run this script every day at 12:00 UTC.</p>

<p>To create <code>.github/workflows/daily_roast.yml</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Daily Lastfm Roast</span>
<span class="na">on</span><span class="pi">:</span>
  <span class="na">schedule</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0</span><span class="nv"> </span><span class="s">12</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*'</span> <span class="c1"># Noon UTC, every day</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">roast</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v2</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v2</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">python-version</span><span class="pi">:</span> <span class="s1">'</span><span class="s">3.9'</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pip install -r requirements.txt</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">python lastfm_roast.py</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">POE_API_KEY</span><span class="pi">:</span> <span class="s">${{ secrets.POE_API_KEY }}</span>
          <span class="na">EMAIL_ADDRESS</span><span class="pi">:</span> <span class="s">${{ secrets.EMAIL_ADDRESS }}</span>
          <span class="na">EMAIL_PASSWORD</span><span class="pi">:</span> <span class="s">${{ secrets.EMAIL_PASSWORD }}</span>
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3pjiwsb3zmr64usmtuzd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3pjiwsb3zmr64usmtuzd.png" alt=" " width="800" height="321"></a></p>

<h2>
  
  
  Is it worth it? 💸
</h2>

<p>This is my favorite part. Poe charges "Compute Points" to run the underlying model that powers the bot (I used Gemini 2.5 Flash for the backend of my bot). I checked my consumption after a few test runs to see what kind of bill I was racking up:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdotoj03npw05u8brink.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvdotoj03npw05u8brink.png" alt=" " width="800" height="214"></a></p>

<p>Here is the breakdown from my dashboard:</p>

<ul>
<li>  <strong>Model:</strong> Gemini 2.5 Flash</li>
<li>  <strong>Cost per Request:</strong> ~161 Compute Points</li>
<li>  <strong>Dollar Cost:</strong> ~$0.0048 per roast</li>
</ul>

<p>If I run this every single day for a year:<br>
<code>$0.0048 * 365 days = $1.75</code></p>

<p><strong>$1.75 per year.</strong></p>

<p>For less than two bucks, I can get a state-of-the-art LLM to analyze my listening trends and tell me my taste in indie pop is "derivative and sad" every single morning. That is high-value ROI. 📈</p>

<h2>
  
  
  The Result: Before vs After
</h2>

<p><strong>Before (Chat Interface):</strong><br>
Trapped in an app. Hard to share. Markdown text.</p>

<p><strong>After (The Neon Upgrade):</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0h7ijwpb7ulzrcqil4uv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0h7ijwpb7ulzrcqil4uv.png" alt=" " width="800" height="606"></a></p>

<p>Now, when I open my email, I get a sleek, dark-mode card. The band names—the targets of the insults—are highlighted in <strong>Pink</strong>, <strong>Cyan</strong>, and <strong>Green</strong>, making sure I don't miss exactly <em>who</em> I'm being mocked for listening to.</p>

<p>And because it's an email, I can instantly forward the roast to my friends so they, too, can laugh at my pain.</p>

<h2>
  
  
  Wrapping Up
</h2>

<p>This pattern (Custom Bot + API + HTML Generation + Actions) is my go-to for almost all my personal automation. It’s robust, free to host, and creates genuinely fun daily interactions.</p>

<p><strong>Repositories mentioned:</strong></p>

<ul>
<li>  <a href="https://poe.com/lastfm-roaster" rel="noopener noreferrer">lastfm-roaster on Poe</a>
</li>
<li>  <a href="https://developer.poe.com/" rel="noopener noreferrer">Poe API Documentation</a>
</li>
<li>  <a href="https://ai.google.dev" rel="noopener noreferrer">Gemini Developer Documentation</a>
</li>
</ul>

<p>Let me know in the comments if you try this out, or share the worst roast the AI gave you, and happy coding! ✨</p>

