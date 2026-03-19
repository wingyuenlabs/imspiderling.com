---
Title: Starting GOKI2 ADV: Defining Backgrounds, Time Zones, and Screen Update Control Tags [Mod ADV Introduction]
Description: 
Author: HALO
Date: 2026-03-19T21:57:51.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Download
</h1>

<p>If you are interested, you can download the latest environment from the link in the Avalanches-Release2.x.y.y.zip in Assets from the release tag of the following site (Avalanches release version 2.x.y.y).</p>

<p><a href="https://github.com/halo1234/Avalanches/releases" rel="noopener noreferrer">https://github.com/halo1234/Avalanches/releases</a></p>

<h2>
  
  
  Background
</h2>

<p>The definition of a background is simple.<br>
You just need to specify the name and the storage name (file name) as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@make_stage name=white storage=white.png
@make_stage name=black storage=black.png
</code></pre>

</div>



<p>Additionally, you can also specify a time zone for the background.<br>
For time zones, specify the name and suffix as follows:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@make_time_zone name=morning suffix=''
@make_time_zone name=evening suffix=ev
@make_time_zone name=night suffix=ng
</code></pre>

</div>



<p>With this, you can now use five tags: "white", "black", "morning", "evening", and "night".<br>
When a time zone tag is specified, the background storage will load with the corresponding suffix added to the storage name.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@evening
@white

[halo] Loaded the white background for the evening.
</code></pre>

</div>



<p>In this case, the "white" tag will load <strong>whiteev.png</strong>.</p>

<h2>
  
  
  Timing of screen updates
</h2>

<p>ModADV updates (transitions) backgrounds and character sprites at the timing when text is processed.<br>
So, if you do the following, the screen will transition before displaying "あ".<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>@white

@halo pose1 expression1 middle show
[halo]abcdefg。
</code></pre>

</div>



<p>If you want the background and character sprites to transition separately,<br>
you can also use the [<strong>run_reserved_transition</strong>] tag to forcibly trigger a transition.</p>

<h2>
  
  
  Control tags
</h2>

<p>ModADV has several tags for control purposes (with <strong>run_reserved_transition</strong> being one of them).<br>
If you understand how they work, using them is fine.<br>
If you don't understand how they work, it might be difficult to use them properly.</p>

