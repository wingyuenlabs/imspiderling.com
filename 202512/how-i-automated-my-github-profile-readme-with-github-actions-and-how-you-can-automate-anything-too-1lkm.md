---
Title: How I Automated My GitHub Profile README With GitHub Actions (And How You Can Automate Anything Too)
Description: 
Author: BHARGAB KALITA
Date: 2025-12-01T22:01:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>At the end of last year, I started writing a few more articles on Dev.to. I’d been posting for a while, but I’m honestly too lazy to keep updating my GitHub Profile README every time. Opening the repo, editing the README, adding links… it wasn’t difficult, just boring. And boring tasks always make me wonder why I’m still doing them manually.</p>

<p>So I looked around, tried a few things, and with a bit of help from AI, I finally found a way to automate it. After some quick experiments, everything clicked. My profile started updating itself with my latest Dev.to articles. No manual edits. No drama. Just simple automation doing its job.</p>

<p>It was a small win, but it felt great. And that’s what pushed me to share how I did it and what else you can automate using GitHub Actions.</p>

<h2>
  
  
  <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxq6lk00fepxklwpn91xv.png" alt=" " width="800" height="445">
</h2>

<h2>
  
  
  Before You Start: The Special GitHub Profile Repository
</h2>

<p>This part is important.</p>

<p>GitHub only displays your README as your public profile if you create a repo <strong>exactly named after your username</strong>.</p>

<p>For example:</p>

<ul>
<li>Username → <code>yourusername</code>
</li>
<li>Profile repo → also <code>yourusername</code>
</li>
</ul>

<p>If you don’t have this repository yet, create a new <strong>public</strong> one with the same name as your username. Once it exists:</p>

<ul>
<li>the README becomes your profile README</li>
<li>you can add workflows in <code>.github/workflows</code>
</li>
<li>automation will run exactly like in any other repo</li>
</ul>

<p>This is the repo where I added my workflow last year, and it’s still running perfectly.</p>




<h2>
  
  
  How I Automated My <a href="https://github.com/the-wrong-guy/the-wrong-guy" rel="noopener noreferrer">Profile</a> <a href="https://github.com/the-wrong-guy/the-wrong-guy/blob/main/README.md" rel="noopener noreferrer">README </a>to Show My Latest Dev.to Articles
</h2>

<p>The goal was simple:<br>
<strong>Show the 3 latest posts from Dev.to in my README without updating anything manually.</strong></p>

<p>I changed my workflow so that it fetches only the latest 3 articles and rewrites the section in my README.</p>

<p>Here’s the complete GitHub Actions workflow.</p>


<h2>
  
  
  GitHub Actions Workflow (Shows Only 3 Latest Dev.to Articles)
</h2>

<p>Create:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>.github/workflows/update-readme.yml
</code></pre>

</div>



<p>Add:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">Update README with latest Dev.to Post</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">schedule</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s1">'</span><span class="s">0</span><span class="nv"> </span><span class="s">0</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*'</span>
  <span class="na">workflow_dispatch</span><span class="pi">:</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">update-readme</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Checkout Repository</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v3</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Fetch Latest Dev.to Posts (replace the username)</span>
        <span class="na">id</span><span class="pi">:</span> <span class="s">fetch_dev_posts</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">DEVTO_USERNAME="bhargab"</span>
          <span class="s">API_RESPONSE=$(curl -s "https://dev.to/api/articles?username=${DEVTO_USERNAME}")</span>

          <span class="s"># Get the titles, URLs, and publish dates of the latest 3 posts</span>
          <span class="s">TITLE_1=$(echo $API_RESPONSE | jq -r '.[0].title')</span>
          <span class="s">URL_1=$(echo $API_RESPONSE | jq -r '.[0].url')</span>
          <span class="s">DATE_1=$(echo $API_RESPONSE | jq -r '.[0].published_at' | cut -d'T' -f1)</span>

          <span class="s">TITLE_2=$(echo $API_RESPONSE | jq -r '.[1].title')</span>
          <span class="s">URL_2=$(echo $API_RESPONSE | jq -r '.[1].url')</span>
          <span class="s">DATE_2=$(echo $API_RESPONSE | jq -r '.[1].published_at' | cut -d'T' -f1)</span>

          <span class="s">TITLE_3=$(echo $API_RESPONSE | jq -r '.[2].title')</span>
          <span class="s">URL_3=$(echo $API_RESPONSE | jq -r '.[2].url')</span>
          <span class="s">DATE_3=$(echo $API_RESPONSE | jq -r '.[2].published_at' | cut -d'T' -f1)</span>

          <span class="s"># Pass these values to the GitHub environment for use in the next steps</span>
          <span class="s">echo "title_1=$TITLE_1" &gt;&gt; $GITHUB_ENV</span>
          <span class="s">echo "url_1=$URL_1" &gt;&gt; $GITHUB_ENV</span>
          <span class="s">echo "date_1=$DATE_1" &gt;&gt; $GITHUB_ENV</span>

          <span class="s">echo "title_2=$TITLE_2" &gt;&gt; $GITHUB_ENV</span>
          <span class="s">echo "url_2=$URL_2" &gt;&gt; $GITHUB_ENV</span>
          <span class="s">echo "date_2=$DATE_2" &gt;&gt; $GITHUB_ENV</span>

          <span class="s">echo "title_3=$TITLE_3" &gt;&gt; $GITHUB_ENV</span>
          <span class="s">echo "url_3=$URL_3" &gt;&gt; $GITHUB_ENV</span>
          <span class="s">echo "date_3=$DATE_3" &gt;&gt; $GITHUB_ENV</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Update README</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">README_FILE="./README.md"</span>
          <span class="s"># Build the new content for the latest 3 posts</span>
          <span class="s">NEW_CONTENT="- [${{ env.title_1 }}](${{ env.url_1 }}) - Published on ${{ env.date_1 }}\n\- [${{ env.title_2 }}](${{ env.url_2 }}) - Published on ${{ env.date_2 }}\n\- [${{ env.title_3 }}](${{ env.url_3 }}) - Published on ${{ env.date_3 }}"</span>

          <span class="s"># Replace the placeholder section using a different delimiter (|)</span>
          <span class="s">sed -i '/&lt;!-- LATEST_DEVTO_POST --&gt;/{n;N;N;s|.*|'"$NEW_CONTENT"'|}' $README_FILE</span>

      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Commit Changes</span>
        <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">git config --local user.name "github-actions[bot]"</span>
          <span class="s">git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"</span>
          <span class="s">git add README.md --force</span>
          <span class="s">git commit -m "Update README with latest 3 Dev.to posts" || echo "No changes to commit"</span>
          <span class="s">git push</span>
</code></pre>

</div>



<p>And in your README:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight markdown"><code><span class="c">&lt;!-- ARTICLES_START --&gt;</span>
<span class="c">&lt;!-- ARTICLES_END --&gt;</span>
</code></pre>

</div>



<p>GitHub Actions replaces everything between those two markers every time it runs.</p>

<p>Your profile stays updated. You don’t touch anything.</p>




<h2>
  
  
  What Else Can You Automate With GitHub Actions?
</h2>

<p>Once you get comfortable with this, you’ll realize GitHub Actions can automate pretty much anything you can script. Here are some actually useful examples:</p>




<h3>
  
  
  1. Automatically Add Your Latest YouTube Videos
</h3>

<p>If you upload tutorials or dev content, this keeps your profile fresh.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Fetch YouTube videos</span>
  <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">curl -s "https://www.googleapis.com/youtube/v3/search?key=$YT_API_KEY&amp;channelId=$CHANNEL_ID&amp;part=snippet&amp;order=date&amp;maxResults=5" &gt; videos.json</span>
</code></pre>

</div>






<h3>
  
  
  2. Auto-Update GitHub Stats or Badges
</h3>

<p>You can show live data:</p>

<ul>
<li>follower count</li>
<li>stars</li>
<li>views</li>
<li>repo metrics
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">curl -s https://api.github.com/users/YOUR_USER | jq '.followers'</span>
</code></pre>

</div>



<p>You can then embed that into badges or markdown.</p>




<h3>
  
  
  3. Use GitHub Actions as a Free Cron Server
</h3>

<p>Run scheduled tasks without having any server.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">on</span><span class="pi">:</span>
  <span class="na">schedule</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s2">"</span><span class="s">0</span><span class="nv"> </span><span class="s">*/6</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*"</span>  <span class="c1"># Every 6 hours</span>
</code></pre>

</div>



<p>Good for:</p>

<ul>
<li>cleaning old branches</li>
<li>updating analytics</li>
<li>regenerating documentation</li>
<li>syncing data</li>
</ul>




<h3>
  
  
  4. Create Your Own Personal Automation Bots
</h3>

<p>This part is way more fun than it sounds.</p>

<p>Examples:</p>

<ul>
<li>get a Telegram/Discord ping when someone stars your repo</li>
<li>generate daily GitHub activity summaries</li>
<li>sync posts from X/Twitter or LinkedIn</li>
<li>rebuild your portfolio site every night</li>
<li>generate OG images</li>
<li>track metrics for projects</li>
</ul>

<p>If you can script it, GitHub Actions can run it for free.</p>




<h2>
  
  
  A Simple Automation Template You Can Reuse
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">name</span><span class="pi">:</span> <span class="s">My Automation</span>

<span class="na">on</span><span class="pi">:</span>
  <span class="na">schedule</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="na">cron</span><span class="pi">:</span> <span class="s2">"</span><span class="s">0</span><span class="nv"> </span><span class="s">0</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*</span><span class="nv"> </span><span class="s">*"</span>
  <span class="na">workflow_dispatch</span><span class="pi">:</span>

<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">run-script</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">python3 script.py</span>
</code></pre>

</div>



<p>Drop your logic into <code>script.py</code> and you’re done.</p>




<p>If you’ve been meaning to automate your own workflows or make your profile more dynamic, this is a great place to start. Try it, break it, tweak it, and make it your own hand✌️.</p>

