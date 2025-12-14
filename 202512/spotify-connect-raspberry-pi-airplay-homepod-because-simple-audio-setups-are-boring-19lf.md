---
Title: Spotify Connect, Raspberry Pi, AirPlay & HomePod - because simple audio setups are boring
Description: 
Author: Piotr Ładoński
Date: 2025-12-14T21:39:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>I could write a whole long introduction of why I needed to stream Spotify from my Raspberry Pi to HomePod Mini speaker, how I arrived at the final idea, how I was hitting constant problems with other solutions... but if you're reading this, I guess you are way more interested in the actual setup 🙂 So let's just put it very briefly:</p>

<p>My problem: </p>

<ul>
<li>
<a href="https://www.ofzenandcomputing.com/airplay-vs-bluetooth-cy-complete-comparison-guide/#:~:text=Quick%20Answer%3A%20Bluetooth%20typically%20uses%2040%2D60%25%20less%20battery%20than%20AirPlay%20due%20to%20lower%20power%20requirements%20and%20simpler%20processing." rel="noopener noreferrer">AirPlay drains phone battery fast</a>, but it's the only option for playing Spotify on HomePod Mini. How can I AirPlay music to HomePod, but not from the phone... and still control it from the phone?</li>
</ul>

<p>Solution:</p>

<ul>
<li>Raspberry Pi &amp; Spotify Connect to the rescue! Make the Raspberry Pi the playing device and control it from the phone via Spotify Connect</li>
</ul>

<p>The tricky part... was the actual execution. It took me a significant amount of time, caused quite some frustration with solutions that didn't work, and, finally, I had to take bits from several of them to make things work. In the end, if you know what to do, it's not hard, but... you must know what to do 🙂 I'll do my best to explain the process as clearly as possible, so here we go!</p>

<h2>
  
  
  General idea for Raspberry setup
</h2>

<p>Basics: I'm using Raspberry Pi 5 and the newest Raspbian based on Debian 13 (Trixie).</p>

<p>There are two main things that RPi must do</p>

<ol>
<li>become a Spotify Connect device and</li>
<li>stream audio via AirPlay to the HomePod</li>
</ol>

<p>Luckily, it's nothing innovative, and there are already plenty of existing solutions for both things. However, there are actually so many that you can spend a lot of time trying to combine them and get nowhere. My silver bullets turned out to be</p>

<ul>
<li>
<a href="https://dtcooper.github.io/raspotify/" rel="noopener noreferrer">Raspotify</a> - turning RPi into a Spotify Connect device</li>
<li>
<a href="https://owntone.github.io/owntone-server/" rel="noopener noreferrer">Owntone</a> - AirPlay-ing to the speaker</li>
</ul>

<p>And, one secret ingredient that glues them together - Linux named pipes.</p>

<p>Oh, and a very important detail - Spotify Premium. Raspotify doesn't work with free accounts.</p>

<h2>
  
  
  Installing things
</h2>

<p>That's probably optional, but I'd start with a good old package upgrade:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt upgrade
</code></pre>

</div>



<p>Why <em>probably</em>? Because I've run it and haven't checked if things would work without the system-wide package upgrade 🙂</p>

<h3>
  
  
  Raspotify
</h3>

<p>That's super easy. As <a href="https://dtcooper.github.io/raspotify/" rel="noopener noreferrer">the docs</a> mention, just run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>apt-get <span class="nt">-y</span> <span class="nb">install </span>curl <span class="o">&amp;&amp;</span> curl <span class="nt">-sL</span> https://dtcooper.github.io/raspotify/install.sh | sh
</code></pre>

</div>



<p>and that's it. Raspotify will be installed &amp; started. You can verify it with<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>systemctl status raspotify
</code></pre>

</div>



<p>It should also already appear as a Spotify Connect device.<br>
Remember, your phone, Raspberry, and HomePod need to be on the same network!</p>
<h3>
  
  
  Owntone
</h3>

<p>You can follow the steps from the <a href="https://forums.raspberrypi.com/viewtopic.php?t=49928" rel="noopener noreferrer">forum post</a>, or just trust me that I copied everything correctly and run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>wget <span class="nt">-q</span> <span class="nt">-O</span> - https://raw.githubusercontent.com/owntone/owntone-apt/refs/heads/master/repo/rpi/owntone.gpg | <span class="nb">sudo </span>gpg <span class="nt">--dearmor</span> <span class="nt">--output</span> /usr/share/keyrings/owntone-archive-keyring.gpg <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>wget <span class="nt">-q</span> <span class="nt">-O</span> /etc/apt/sources.list.d/owntone.list https://raw.githubusercontent.com/owntone/owntone-apt/refs/heads/master/repo/rpi/owntone-trixie.list <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt <span class="nb">install </span>owntone <span class="nt">-y</span>
</code></pre>

</div>



<p>After that, go to <a href="http://owntone.local:3689" rel="noopener noreferrer">http://owntone.local:3689</a> (or <a href="http://localhost:3689" rel="noopener noreferrer">http://localhost:3689</a>) and check if the web UI loads correctly.</p>

<h3>
  
  
  Pipes
</h3>

<p>Long story short, we'll create pipe files to enable communication between Raspotify and Owntune. Raspotify will be writing to them, OwnTune reading and playing that via AirPlay.<br>
Just run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo mkdir</span> /srv/music <span class="o">&amp;&amp;</span> <span class="nb">sudo mkfifo</span> /srv/music/raspotify-pipe <span class="o">&amp;&amp;</span> <span class="nb">sudo mkfifo</span> /srv/music/raspotify-pipe.metadata
</code></pre>

</div>



<p>to create two pipes. You can use different names without any issue, but if you want to put them in a different directory, you'll need to update <code>library.directories</code> option in Owntone configuration.<br>
Also, <code>.metadata</code> pipe won't be actually used, as Raspotify is not currently able to process metadata. We created it simply to have one less error from Owntone - it just needs the file to exist.</p>
<h2>
  
  
  Configuration
</h2>
<h3>
  
  
  Raspotify
</h3>

<p>All the Raspotify options are <a href="https://github.com/dtcooper/raspotify/wiki/Configuration" rel="noopener noreferrer">listed on the wiki</a>. We'll need to change only 2 of them.<br>
Let's open the configuration file with nano to edit it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>nano /etc/raspotify/conf
</code></pre>

</div>



<p>and set <code>LIBRESPOT_BACKEND</code> and <code>LIBRESPOT_DEVICE</code> as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>LIBRESPOT_BACKEND=pipe
LIBRESPOT_DEVICE=/var/run/raspotify-pipe
</code></pre>

</div>



<p>Remember to uncomment them!<br>
If you want, you can also change the name under which Raspberry will appear on Spotify Connect devices list. To do this, modify <code>LIBRESPOT_NAME</code>, for example<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>LIBRESPOT_NAME="Raspotify"
</code></pre>

</div>



<p>Save and close the file (<code>Ctrl + X</code>, <code>Y</code>, <code>Enter</code>).</p>

<h3>
  
  
  Owntone
</h3>

<p>In the bottom right of the web UI, you should see an arrow to expand the simplified output control panel. One of the devices there should be the AirPlay speaker (HomePod) that you want to use - click its icon to enable it.</p>

<p>You can also adjust the volume here. It's quite important - at the end, I had everything working correctly, but the slider here was too low, and I thought something was still broken 🙃 The "final volume" on the HomePod is a product of the volume you set in Owntone and the value picked for the Spotify Connect device on the phone. Keep that in mind when things are too quiet/loud.</p>

<p>To make sure all the config changes in both Raspotify and Owntune are applied, restart the services:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>systemctl restart raspotify <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>systemctl restart owntone
</code></pre>

</div>



<h2>
  
  
  Running everything
</h2>

<p>We're basically there. Now get your phone, open Spotify, and you should be able to select Raspberry as the Spotify Connect device 🎶<br>
<strong>Remember that Raspberry and your phone must be on the same Wi-Fi network!</strong><br>
Here are also some details about Spotify Connect on the Spotify support page if you're not familiar with the feature itself: <a href="https://support.spotify.com/article/spotify-connect/" rel="noopener noreferrer">https://support.spotify.com/article/spotify-connect/</a></p>
<h3>
  
  
  General troubleshooting
</h3>

<p>If things are not working, <code>journalctl</code> is your best friend in debugging. Simply open a terminal and run<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>journalctl <span class="nt">-u</span> owntone <span class="nt">-f</span>
</code></pre>

</div>



<p>(or <code>raspotify</code> instead of <code>owntone</code>)<br>
and you'll see some system logs from the services. Hopefully, you'll see some "error" lines that will help you fix the problem.</p>

<h2>
  
  
  Summary
</h2>

<p>As I've written at the beginning, if you know the steps, the setup should be straightforward. It's just about running two services and connecting them. Knowing the Raspberry/Linux ecosystem, the above steps will become outdated sooner than later, but I hope they will be useful for at least a moment 😄</p>

<p>If you hit any problems, write a comment. While my Linux-fu is not high, I'll do my best to help.</p>

<p>Happy streaming! 🎵 </p>




