---
Title: How We Built Transcript-Powered Video Editing in Go
Description: 
Author: Alex Neamtu
Date: 2026-02-20T21:35:39.000Z
Robots: noindex,nofollow
Template: index
---
<p>You record a five-minute product walkthrough. You say "um" eleven times. The first thirty seconds are you fumbling with screen share. The title reads "Recording 2/20/2026 3:45:12 PM."</p>

<p>Fixing any of this used to mean dragging tiny handles on a timeline, scrubbing back and forth to find the right millisecond. If you wanted to remove filler words, you'd have to find each one manually and trim them out one at a time.</p>

<p>We already had word-level transcripts from whisper.cpp. The data was sitting right there — timestamps, text, segment boundaries. We just weren't using it for editing.</p>

<p><a href="https://sendrec.eu" rel="noopener noreferrer">SendRec</a> v1.45.0 adds three features that change that: trim by transcript, filler word removal, and AI-generated title suggestions. All three build on the same transcript data.</p>

<h2>
  
  
  Trim by transcript
</h2>

<p>The existing trim tool works with draggable handles on a time bar. Fine for rough cuts, but hard to use when you need to cut at the exact moment a sentence starts.</p>

<p>The trim modal now shows the video's transcript segments below the timeline. Two mode buttons — "Set Start" and "Set End" — control what happens when you click a segment.</p>

<p>Click a segment in "Set Start" mode and the trim start jumps to that segment's timestamp. The mode automatically switches to "Set End." Click another segment and you've defined your range. Segments inside the range highlight green. Segments outside dim to 40% opacity.</p>

<p>The existing drag handles still work. The transcript panel is an addition, not a replacement. And since the trim endpoint hasn't changed, this was a frontend-only enhancement — no backend work needed.</p>

<p>If the video doesn't have a transcript yet (still processing, or transcription failed), the modal shows the original timeline-only UI.</p>

<h2>
  
  
  Filler word removal
</h2>

<p>This is the feature that saves the most time. Instead of finding and trimming each "um" individually, you preview all of them at once and remove them in a single operation.</p>

<p>Click "Remove fillers" in the Library overflow menu and a modal scans the transcript for filler-only segments. The detection uses a simple regex:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>/^\s*(?:um+|uh+|uhh+|hmm+|ah+|er+|you know|like|so|basically|actually|right|i mean)[.,!?\s]*$/i
</code></pre>

</div>



<p>The key word is "filler-only." A segment that says "um" gets flagged. A segment that says "um, so the next thing we need to discuss" does not. We only flag segments where the entire text is a filler word — no risk of cutting real content.</p>

<p>The modal shows every detected filler with a checkbox, its timestamp, and the text. All are checked by default. A summary line shows the count and total time you'd save: "Found 11 filler words (4.2s total)." Uncheck any you want to keep, then click Remove.</p>

<h3>
  
  
  The backend: one ffmpeg pass
</h3>

<p>The removal endpoint accepts a list of time ranges to cut:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">POST</span><span class="w"> </span><span class="err">/api/videos/</span><span class="p">{</span><span class="err">id</span><span class="p">}</span><span class="err">/remove-segments</span><span class="w">

</span><span class="p">{</span><span class="w">
  </span><span class="nl">"segments"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="mf">3.2</span><span class="p">,</span><span class="w"> </span><span class="nl">"end"</span><span class="p">:</span><span class="w"> </span><span class="mf">3.8</span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="mf">12.1</span><span class="p">,</span><span class="w"> </span><span class="nl">"end"</span><span class="p">:</span><span class="w"> </span><span class="mf">12.9</span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="nl">"start"</span><span class="p">:</span><span class="w"> </span><span class="mf">25.0</span><span class="p">,</span><span class="w"> </span><span class="nl">"end"</span><span class="p">:</span><span class="w"> </span><span class="mf">25.6</span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Validation rejects overlapping segments, unsorted segments, negative timestamps, segments beyond the video duration, more than 200 segments per request, and any cut that would leave less than one second of video.</p>

<p>The actual removal uses ffmpeg's <code>select</code> and <code>aselect</code> filters to keep only the frames NOT in the removed ranges:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ffmpeg -i input -filter_complex
  "[0:v]select='not(between(t,3.2,3.8)+between(t,12.1,12.9)+between(t,25.0,25.6))',setpts=N/FRAME_RATE/TB[v];
   [0:a]aselect='not(between(t,3.2,3.8)+between(t,12.1,12.9)+between(t,25.0,25.6))',asetpts=N/SR/TB[a]"
  -map "[v]" -map "[a]" output
</code></pre>

</div>



<p>This handles any number of segments in a single pass. The <code>setpts</code> and <code>asetpts</code> filters reset the timestamps so the output plays continuously without gaps.</p>

<p>After processing, the video gets a new thumbnail and goes back through transcription — the old transcript's timestamps no longer match.</p>

<h3>
  
  
  Why not word-level timestamps?
</h3>

<p>Whisper segments are phrase-level, not word-level. A segment might be "um, so basically" — three words, one timestamp range. We can't cut just the "um" and keep "so basically" because we don't know exactly when each word starts and ends within the segment.</p>

<p>This means some fillers get missed: the ones embedded in longer phrases. That's a deliberate tradeoff. We flag the segments we're certain about and leave the rest alone. No false positives.</p>

<h2>
  
  
  AI title suggestions
</h2>

<p>After transcription completes, the summary worker checks whether the video still has an auto-generated title — patterns like "Recording 2/20/2026 3:45:12 PM" or "Untitled Recording." If it does, it sends the first portion of the transcript to the AI client and asks for a concise title.</p>

<p>The prompt is simple:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Given this video transcript, generate a concise title (3-8 words)
that captures the main topic. Return ONLY the title text, no quotes,
no explanation. Write in the same language as the transcript.
</code></pre>

</div>



<p>The suggestion is stored in a <code>suggested_title</code> column. It doesn't overwrite anything — the original title stays until the user decides.</p>

<p>In the Library, videos with a suggested title show a small indicator below the title text. Click it to see the suggestion with Accept and Dismiss buttons. Accept updates the title. Dismiss clears the suggestion. Either way, it's a one-click decision.</p>

<p>The title generation piggybacks on the existing summary worker. No separate queue, no extra infrastructure. If the AI service is down, no suggestion is stored — the video just keeps its original title.</p>

<h2>
  
  
  What we didn't build
</h2>

<ul>
<li>Word-level transcript timestamps (whisper segments are phrase-level)</li>
<li>Real-time filler detection during recording</li>
<li>Automatic filler removal without user confirmation</li>
<li>Batch filler removal across multiple videos</li>
</ul>

<p>Each of these has tradeoffs that would add complexity without clear benefit yet. Word-level timestamps would improve filler detection but require a different whisper model configuration. Auto-removal without preview would be faster but risky — "like" and "so" are sometimes real words, not fillers.</p>

<h2>
  
  
  Try it
</h2>

<p>Transcript-powered editing is live at <a href="https://app.sendrec.eu" rel="noopener noreferrer">app.sendrec.eu</a> in v1.45.0. Self-hosters get all three features on upgrade — the migration runs automatically on startup. AI title suggestions require the <code>AI_ENABLED</code> environment variable and a compatible LLM endpoint.</p>

<p>If you're self-hosting SendRec, check out the <a href="https://github.com/sendrec/sendrec/blob/main/SELF-HOSTING.md" rel="noopener noreferrer">self-hosting guide</a>.</p>

