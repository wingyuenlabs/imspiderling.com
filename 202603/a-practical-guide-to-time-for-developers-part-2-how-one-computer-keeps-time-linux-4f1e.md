---
Title: A Practical Guide to Time for Developers: Part 2 — How one computer keeps time (Linux)
Description: 
Author: Dmytro Huz
Date: 2026-03-05T21:28:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>Time on a computer <em>looks</em> simple: call now(), get a timestamp, move on.</p>

<p>In the <a href="https://www.dmytrohuz.com/p/a-practical-guide-to-time-for-developers" rel="noopener noreferrer">previous article</a>, we discussed the idea that time is just a single point on a timeline. The crucial part is defining <strong>which</strong> timeline that point belongs to.</p>

<p>For computers, this matters a lot. A system effectively works with two timelines: <strong>real time</strong> and <strong>boot time</strong>. You can convert between them, but they are different measuring systems and shouldn’t be used interchangeably—because each timeline serves a different purpose.</p>

<ul>
<li>
<strong>Real time</strong> answers: “What time is it in the real world right now?”</li>
<li>
<strong>Boot time</strong> answers: “How much time has passed since X (boot)?”</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbs7ld4isas2yihapw9o8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbs7ld4isas2yihapw9o8.png" alt="timeline" width="514" height="338"></a></p>

<p>The computer has has an ecosystem - a few components: hardware and software to track and calculate different times in both timelines. </p>

<p>This part explains that ecosystem using one diagram, top to bottom. The diagram is the spine; everything else is commentary that makes it click: where ticks come from, what the hardware pieces do, why there are multiple “system times”, what suspend breaks, where interrupts fit, and how epoch nanoseconds become “Tuesday 14:03 in Vienna”.</p>

<p><strong>Figure 1 — The whole pipeline</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvlmk629rsoscvaf559l0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvlmk629rsoscvaf559l0.png" alt="Pipeline" width="800" height="919"></a><br>
Four lanes:</p>

<ul>
<li>
<strong>RTC</strong>: survives power-off, keeps “wall time”</li>
<li>
<strong>CPU counter</strong> (often TSC): ticks while the CPU runs</li>
<li>
<strong>Kernel timekeeper</strong>: turns ticks into several clocks</li>
<li>
<strong>Userspace</strong>: calls clock_gettime() and formats time for humans</li>
</ul>

<p>Now: top → bottom.</p>




<h2>
  
  
  <strong>1) Boot &amp; Initialization Phase</strong>
</h2>

<h3>
  
  
  <strong>1.1 RTC: the “battery clock”</strong>
</h3>

<p>At the top-left sits the RTC. Think of it as the tiny clock that keeps time while the computer is asleep or powered off. It usually stores calendar-ish values (year/month/day/hour/min/sec). It’s not “nanoseconds since 1970” by nature — that’s something software creates later.</p>

<p>RTC exists so the system doesn’t boot into the void. Without it, everything starts at “some default” until NTP/PTP (or a human) sets the time.</p>

<h3>
  
  
  <strong>1.2 Turning RTC into “Unix time”</strong>
</h3>

<p>Next step: the kernel reads RTC and converts it into Unix epoch time (seconds + nanoseconds since 1970-01-01T00:00:00Z). That gives a sensible starting point for time-of-day.</p>

<p>This is still a <em>bootstrap</em> value. RTC isn’t a precision clock. It’s a “good enough to start” clock.</p>

<h3>
  
  
  <strong>1.3 The CPU counter: ticks while running</strong>
</h3>

<p>Now the machine is awake, so Linux wants something faster and more stable than “ask the RTC all the time.” Enter the CPU/platform counter — the <strong>clocksource</strong>. On modern x86, that’s often the <strong>TSC</strong>.</p>

<p>Important mindset shift:</p>

<blockquote>
<p>TSC is not “the time.”</p>

<p>TSC is “how many ticks happened since some arbitrary start.”</p>
</blockquote>

<p>It’s a counter. It’s only meaningful after conversion.</p>

<h3>
  
  
  <strong>1.4 Calibration: making ticks speak nanoseconds</strong>
</h3>

<p>Ticks are just ticks until the kernel knows the counter’s rate. That’s why the diagram shows calibration: “cycles per second”.</p>

<p>Linux maintains conversion parameters so it can cheaply do:</p>

<ul>
<li>delta ticks → delta nanoseconds</li>
</ul>

<p>That’s the key: Linux mostly cares about <strong>deltas</strong>.</p>

<h3>
  
  
  <strong>1.5 Boot finishes by aligning the timelines</strong>
</h3>

<p>At the end of boot, two things are true:</p>

<ul>
<li>there’s an “elapsed since boot” timeline (monotonic) starting at 0</li>
<li>there’s a “wall clock” timeline (realtime) aligned to the RTC-derived epoch time</li>
</ul>

<p>The clean relationship is:</p>

<blockquote>
<p><strong>CLOCK_REALTIME = CLOCK_MONOTONIC + wall_clock_offset</strong></p>
</blockquote>

<p>At boot, the kernel chooses the offset so realtime matches RTC.</p>

<p>This one relationship explains half of the weirdness people hit later.</p>




<h2>
  
  
  <strong>2) Runtime Phase (Continuous Tracking)</strong>
</h2>

<h3>
  
  
  <strong>2.1 Where ticks come from (without going into physics)</strong>
</h3>

<p>Under the hood, some oscillator ticks, hardware counts those ticks, and Linux reads the count. That’s it at the conceptual level.</p>

<p>The only thing worth memorizing here:</p>

<blockquote>
<p>The hardware gives ticks. The OS gives meaning.</p>
</blockquote>

<h3>
  
  
  <strong>2.2 The kernel’s “working memory”</strong>
</h3>

<p>In the middle of the diagram there’s a box of variables. That box is basically the kernel’s timekeeping brain:</p>

<ul>
<li>tsc_now, tsc_last — current and previous counter snapshot</li>
<li>mult, shift — ticks→ns conversion</li>
<li>mono — accumulated elapsed time since boot</li>
<li>suspend_ns — time spent asleep (so BOOTTIME can include it)</li>
<li>wall_off — the offset that turns monotonic into realtime</li>
</ul>

<p>With those, Linux can build the clock APIs that user space expects.</p>




<h2>
  
  
  <strong>3) Time requests: clock_gettime() makes everything happen</strong>
</h2>

<p>This part of the diagram is the “action scene”:</p>

<ul>
<li>userspace calls clock_gettime(CLOCK_...)</li>
<li>kernel reads the counter (RDTSC in the TSC world)</li>
<li>kernel updates its state (delta ticks → delta ns → accumulate)</li>
<li>kernel returns the requested clock</li>
</ul>

<p>A useful mental shortcut:</p>

<blockquote>
<p>The kernel doesn’t need a metronome to keep time moving.</p>

<p>It can compute “now” on demand by reading a running counter.</p>
</blockquote>

<p>(Internally there are periodic activities too, but this is the clean model.)</p>

<h3>
  
  
  <strong>The tiny update loop</strong>
</h3>

<p>The heartbeat is:</p>

<ul>
<li>delta_ticks = tsc_now - tsc_last</li>
<li>delta_ns = ticks_to_ns(delta_ticks)</li>
<li>mono += delta_ns</li>
</ul>

<p>Everything else is derived from mono.</p>




<h2>
  
  
  <strong>4) Kernel clocks: three timelines, three different promises</strong>
</h2>

<p>Now the diagram derives the clocks.</p>

<h3>
  
  
  <strong>4.1 CLOCK_MONOTONIC — for durations</strong>
</h3>

<p>The diagram says:</p>

<blockquote>
<p>CLOCK_MONOTONIC = mono</p>
</blockquote>

<p>That’s the “elapsed time” clock.</p>

<p>It’s the clock to use for anything that needs to be sane even if wall time changes:</p>

<ul>
<li>timeouts</li>
<li>retries</li>
<li>rate limiting</li>
<li>latency measurements</li>
<li>“sleep for X”</li>
</ul>

<p>It doesn’t go backwards, and it doesn’t jump when someone sets the wall clock.</p>

<h3>
  
  
  <strong>4.2 CLOCK_REALTIME — for human time</strong>
</h3>

<p>The diagram says:</p>

<blockquote>
<p>CLOCK_REALTIME = mono + wall_off</p>

<p><em>(setting time changes wall_off, not mono)</em></p>
</blockquote>

<p>This is <em>the</em> sentence.</p>

<p>Realtime is epoch-based wall time. It’s the one that becomes “2026-03-05 13:00:00”.</p>

<p>Because it must match the outside world, it’s adjustable (NTP/PTP/manual set), and that means it can jump. It’s great for timestamps, terrible for measuring elapsed time.</p>

<h3>
  
  
  <strong>4.3 CLOCK_BOOTTIME — monotonic that includes sleep</strong>
</h3>

<p>The diagram says:</p>

<blockquote>
<p>CLOCK_BOOTTIME = mono + suspend_ns</p>
</blockquote>

<p>Suspend is where people get surprised: monotonic often pauses while the system sleeps. BOOTTIME exists for the “time since boot including sleep” definition.</p>




<h2>
  
  
  <strong>5) Power management: suspend/resume is where the split matters</strong>
</h2>

<p>During suspend:</p>

<ul>
<li>CPU isn’t running</li>
<li>counters may stop or aren’t sampled</li>
<li>mono doesn’t move (in the simple model)</li>
<li>real world keeps moving</li>
</ul>

<p>So the diagram does a clever but simple thing:</p>

<ul>
<li>store persistent time at suspend (often RTC)</li>
<li>store persistent time at resume</li>
<li>difference = sleep delta</li>
<li>add it to suspend_ns so BOOTTIME advances across sleep</li>
<li>keep wall clock aligned after resume (effectively by updating the offset)</li>
</ul>

<p>That’s why BOOTTIME exists and why wall time remains useful after sleep.</p>




<h2>
  
  
  <strong>6) From epoch nanoseconds to “Tuesday 14:03 in Vienna”</strong>
</h2>

<p>Kernel time is a number. Humans want a calendar.</p>

<p>On Linux, CLOCK_REALTIME is typically represented as <strong>nanoseconds since the Unix epoch</strong> (1970-01-01T00:00:00Z). It’s just an integer coordinate on a timeline. Converting it into “Tuesday 14:03 in Vienna” is a user-space job, and it happens in a few very specific steps.</p>

<h3>
  
  
  <strong>6.1 Step 1 — split nanoseconds into seconds + remainder</strong>
</h3>

<p>Most time libraries work in “seconds since epoch” plus a fractional part:</p>

<ul>
<li>sec = epoch_ns / 1_000_000_000</li>
<li>nsec = epoch_ns % 1_000_000_000</li>
</ul>

<p>That split is practical: seconds are large-scale time, nanoseconds are the sub-second detail.</p>

<h3>
  
  
  <strong>6.2 Step 2 — interpret seconds as UTC and create a UTC timestamp</strong>
</h3>

<p>At this point the number becomes “a moment” in UTC:</p>

<ul>
<li>utc_instant = epoch_seconds_to_utc(sec, nsec)</li>
</ul>

<h3>
  
  
  <strong>6.3 Step 3 — convert UTC to a named time zone using tzdata rules</strong>
</h3>

<p>Now comes the real-world complexity.</p>

<p>A numeric offset like +01:00 is not a time zone. It’s just “the offset right now.” Real zones (like Europe/Vienna) are a <strong>ruleset</strong>: they include historical changes and DST transitions.</p>

<p>So the conversion is:</p>

<ul>
<li>local_instant = convert_utc_to_zone(utc_instant, "Europe/Vienna", tzdata)</li>
</ul>

<p>That conversion does three things:</p>

<ol>
<li>finds the correct offset for that instant (+01:00 or +02:00, depending on DST and history)</li>
<li>applies that offset</li>
<li>produces local calendar fields (year/month/day/hour/min/sec) <em>plus</em> the offset</li>
</ol>

<p>This is why “time zones are formatting” is wrong: it’s not string styling, it’s rule evaluation.</p>

<h3>
  
  
  <strong>6.4 Ambiguous and missing local times (DST pain in one minute)</strong>
</h3>

<p>DST creates two special situations that break naive systems:</p>

<p><strong>Ambiguous local time (fall back)</strong></p>

<p>The clock repeats an hour. The same local time occurs twice.</p>

<ul>
<li>Example: 2026-10-25 02:30 in many European zones can mean two different instants.</li>
</ul>

<p><strong>Missing local time (spring forward)</strong></p>

<p>The clock jumps forward. Some local times never occur.</p>

<ul>
<li>Example: 02:30 on the spring-forward day might not exist at all.</li>
</ul>

<p>Notice what happens here: converting <em>from UTC → local</em> is always unambiguous (UTC instants are unique). The pain happens when converting <em>from local → UTC</em> without enough context.</p>

<p>That’s why systems that store “local wall time” without a zone ID eventually end up in a fight with reality.</p>

<h3>
  
  
  <strong>6.5 Step 4 — format for display or transport (ISO 8601 / RFC 3339)</strong>
</h3>

<p>After conversion, formatting is easy:</p>

<ul>
<li>UTC canonical log style: 2026-03-05T13:03:12.123456789Z</li>
<li>Local display style (with offset): 2026-03-05T14:03:12.123456789+01:00</li>
</ul>

<p>The important thing is that formatted output should preserve:</p>

<ul>
<li>the offset (or Z)</li>
<li>and ideally the zone context when it matters</li>
</ul>

<h3>
  
  
  <strong>6.6 What should be stored vs what should be displayed</strong>
</h3>

<p>This is where many systems accidentally create “time debt.”</p>

<p><strong>Store (internally / in DB / across services):</strong></p>

<ul>
<li>an unambiguous instant:

<ul>
<li>epoch timestamp (integer + unit), or</li>
<li>UTC/RFC3339 timestamp with Z</li>
</ul>


</li>

</ul>

<p><strong>Display (UI / reports):</strong></p>

<ul>
<li>convert to the user’s zone at the edge using tzdata.</li>
</ul>

<p><strong>If civil meaning matters (schedules, payroll, appointments):</strong></p>

<ul>
<li>
<p>store the <em>rule</em>, not just the instant:</p>

<ul>
<li>“every day at 09:00 Europe/Vienna”</li>
<li>
<p>plus the zone ID</p>

<p>Because recurring human schedules live in civil time and DST rules matter.</p>
</li>
</ul>


</li>

</ul>

<h3>
  
  
  <strong>6.7 Tiny practical checklist (saves a lot of bugs)</strong>
</h3>

<ul>
<li>Use a <strong>zone ID</strong> (Europe/Vienna), not a fixed offset, for civil-time logic.</li>
<li>Keep timestamps in <strong>UTC-like canonical form</strong> internally.</li>
<li>Convert to local time only at the edges.</li>
<li>Treat “local naive timestamps” as incomplete data unless paired with a zone/ruleset.</li>
<li>When parsing timestamps, require either:

<ul>
<li>Z, or</li>
<li>an explicit offset, or</li>
<li>a zone ID (for civil-time workflows).</li>
</ul>


</li>

</ul>




<h2>
  
  
  <strong>7) Compact model (matches the diagram)</strong>
</h2>

<p><strong>Variables</strong></p>

<ul>
<li>tsc_last, mult, shift</li>
<li>mono (ns since boot)</li>
<li>suspend_ns (ns spent suspended)</li>
<li>wall_off (epoch ns − mono)</li>
</ul>

<p><strong>Update step (on each read)</strong></p>

<ul>
<li>compute delta ticks from the clocksource</li>
<li>convert delta ticks → delta ns</li>
<li>accumulate monotonic time: mono += delta_ns</li>
</ul>

<p><strong>Clock readouts</strong></p>

<ul>
<li>CLOCK_MONOTONIC = mono</li>
<li>CLOCK_BOOTTIME = mono + suspend_ns</li>
<li>CLOCK_REALTIME = mono + wall_off <em>(setting time changes wall_off, not mono)</em>
</li>
</ul>




<h2>
  
  
  <strong>8) Code: a small Linux-style emulator</strong>
</h2>

<p>I tried to create an clear and easy to understand code that would nicely show how everything works together.</p>

<p>This code mirrors the diagram and uses Linux-like APIs (clock_gettime(CLOCK_...), clock_settime(CLOCK_REALTIME, ...)) to show the interactions between RTC, TSC, and the kernel clocks.</p>

<p>You can find the result of my experiment here:</p>

<p><a href="https://github.com/DmytroHuzz/linux_clock_emulator/blob/main/linux_clock.py" rel="noopener noreferrer">https://github.com/DmytroHuzz/linux_clock_emulator/blob/main/linux_clock.py</a></p>




<h2>
  
  
  <strong>9) Developer cheat sheet</strong>
</h2>

<ul>
<li>Use <strong>CLOCK_MONOTONIC</strong> for: timeouts, retries, intervals, measuring latency, scheduling “sleep X”.</li>
<li>Use <strong>CLOCK_BOOTTIME</strong> for elapsed time that should include suspend.</li>
<li>Use <strong>CLOCK_REALTIME</strong> for logs, audits, UI timestamps, business meaning.</li>
<li>Never compute durations as realtime_end - realtime_start.</li>
<li>Time zone conversion is userspace logic (tzdata). Store UTC-like timestamps internally.</li>
</ul>

<h2>
  
  
  <strong>Next: Part 3</strong>
</h2>

<p>Part 3 leaves the single machine and goes to the network and we will see how to sync many machines.</p>

