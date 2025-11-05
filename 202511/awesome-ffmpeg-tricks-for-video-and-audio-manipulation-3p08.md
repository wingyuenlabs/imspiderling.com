---
Title: Awesome FFmpeg Tricks for Video and Audio Manipulation
Description: 
Author: Picoable
Date: 2025-11-05T22:08:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>FFmpeg is the swiss army knife of multimedia processing. It’s a free and open-source command-line tool that can handle virtually any video or audio format. Whether you're a developer, a content creator, or just someone who wants to have more control over their media files, FFmpeg is an essential tool to have in your arsenal. Here are some awesome tricks to get you started.</p>

<h3>
  
  
  Get Media File Information
</h3>

<p>Before you start manipulating a file, you can get detailed information about it, including its codecs, resolution, and duration.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ffmpeg <span class="nt">-i</span> input.mp4
</code></pre>

</div>



<h3>
  
  
  Convert Video and Audio Formats
</h3>

<p>One of the most basic and useful features of FFmpeg is format conversion.</p>

<p>To convert a video from <code>.mov</code> to <code>.mp4</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ffmpeg <span class="nt">-i</span> input.mov output.mp4
</code></pre>

</div>



<p>To extract the audio from a video and save it as an MP3:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ffmpeg <span class="nt">-i</span> input.mp4 <span class="nt">-vn</span> <span class="nt">-ar</span> 44100 <span class="nt">-ac</span> 2 <span class="nt">-ab</span> 192k <span class="nt">-f</span> mp3 output.mp3
</code></pre>

</div>



<h3>
  
  
  Trim and Cut Videos
</h3>

<p>Easily cut a video to a specific length by providing a start time and an end time.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ffmpeg <span class="nt">-i</span> input.mp4 <span class="nt">-ss</span> 00:00:10 <span class="nt">-to</span> 00:00:40 <span class="nt">-c</span> copy output.mp4
</code></pre>

</div>



<p>Using <code>-c copy</code> is a great trick because it avoids re-encoding the video, which is much faster and preserves the original quality.</p>

<h3>
  
  
  Create a GIF from a Video
</h3>

<p>Animated GIFs are a great way to share short video clips. FFmpeg makes creating them a breeze.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ffmpeg <span class="nt">-i</span> input.mp4 <span class="nt">-ss</span> 00:00:10 <span class="nt">-t</span> 5 <span class="nt">-vf</span> <span class="s2">"fps=10,scale=320:-1:flags=lanczos"</span> output.gif
</code></pre>

</div>



<p>This command creates a 5-second GIF starting at the 10-second mark, with a frame rate of 10 fps and a width of 320 pixels.</p>

<h3>
  
  
  Extract Images from a Video
</h3>

<p>Need to grab some frames from a video? FFmpeg can do that too.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>ffmpeg <span class="nt">-i</span> input.mp4 <span class="nt">-r</span> 1 <span class="nt">-f</span> image2 image-%3d.png
</code></pre>

</div>



<p>This command will extract one frame every second and save it as a PNG image with a numbered sequence (<code>image-001.png</code>, <code>image-002.png</code>, etc.).</p>

<h3>
  
  
  Conclusion
</h3>

<p>These are just a handful of the many tricks you can perform with FFmpeg. It’s a deep and powerful tool, and the best way to learn is to experiment. For more information, you can always consult the extensive official documentation. Happy encoding!</p>

