---
Title: HTML Generation
Description: 
Author: Albin Manoj
Date: 2026-03-08T21:43:26.000Z
Robots: noindex,nofollow
Template: index
---
<p><code>\</code>`html</p>

<h4>
  
  
  DTMF Hand-Raise System
</h4>

<h1>
  
  
  wilio Conference Call
</h1>

<p>Participants</p>

<p>5–7 callers + 1 host</p>

<p>System Feasibility</p>

<p>✓ Still Fully Feasible</p>

<p>01</p>

<p>The Core TwiML Constraint corrected</p>

<p>02</p>

<p>Verified Working Patterns POC</p>

<p>03</p>

<p>Alternative 1: Media Streams + Goertzel</p>

<p>04</p>

<p>Alternative 2: Twilio Flex / TaskRouter</p>

<p>05</p>

<p>Alternative 3: Twilio Sync State</p>

<p>06</p>

<p>Alternative 4: Conference Hold + Gather</p>

<p>07</p>

<p>Alternative 5: Dual-Channel (Phone + Web)</p>

<p>08</p>

<p>Decision Matrix &amp; Recommendation</p>

<p>01</p>

<h2>
  
  
  The Core TwiML Constraint — What V1 Got Wrong
</h2>

<p>❌ Root Problem Twilio’s TwiML specification does NOT allow DTMF detection while a caller is inside a <code>&lt;Dial&gt;&lt;Conference&gt;</code>. Any keypad digits pressed by a participant are transmitted as audio tones to the conference room — no server-side webhook fires. There is no native event for this.</p>

<h3>
  
  
  Why the V1 Pattern Fails
</h3>

<p>The original document proposed nesting <code>&lt;Conference&gt;</code> inside <code>&lt;Gather&gt;</code>. This is structurally invalid. The Twilio TwiML schema enforces strict parent-child rules:</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>TwiML Verb</th>
<th>Valid Children</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>&lt;Gather&gt;</code></td>
<td>
<code>&lt;Say&gt;</code>, <code>&lt;Play&gt;</code>, <code>&lt;Pause&gt;</code>
</td>
<td>Collects DTMF or speech input</td>
</tr>
<tr>
<td><code>&lt;Dial&gt;</code></td>
<td>
<code>&lt;Conference&gt;</code>, <code>&lt;Number&gt;</code>, <code>&lt;Client&gt;</code>, <code>&lt;Queue&gt;</code>, <code>&lt;Sip&gt;</code>
</td>
<td>
<code>&lt;Conference&gt;</code> is a noun inside Dial</td>
</tr>
<tr>
<td><code>&lt;Conference&gt;</code></td>
<td>—</td>
<td>No children. Cannot be inside Gather.</td>
</tr>
</tbody>
</table></div>

<p><code></code>`</p>

<p><br>
  
          action="/dtmf-handler"<br>
          timeout="0" <br>
          numDigits="2"&gt;<br>
    &lt;!-- Conference CANNOT be a<br>
         child of Gather --&gt;<br>
    <br>
      MainRoom<br>
    <br>
  <br>
</p>

<p><br>
  Press *1 to raise hand.<br>
  
          action="/dtmf-handler"<br>
          timeout="3" <br>
          numDigits="2"&gt;<br>
    &lt;!-- Only Say/Play/Pause allowed --&gt;<br>
    Press now or hold.<br>
  <br>
  <br>
    <br>
      MainRoom<br>
    <br>
  <br>
</p>

<p>`<code></code></p>

<h3>
  
  
  SDK Error Encountered in Practice
</h3>

<p>The Twilio Node.js SDK correctly enforces this schema. Calling <code>.conference()</code> on a Gather or VoiceResponse object throws immediately:</p>

<p><code></code>`<br>
// These both throw TypeError at runtime<br>
twiml.conference('MainRoom', { muted: true });      // conference() doesn't exist on VoiceResponse<br>
gather.conference('MainRoom', { muted: true });     // conference() doesn't exist on Gather</p>

<p>// TypeError: twiml.conference is not a function<br>
// Correct: conference() only exists on Dial<br>
const dial = twiml.dial();<br>
dial.conference({ muted: true, statusCallback: '...', ... }, 'MainRoom');</p>

<p>`<code></code></p>
<h3>
  
  
  Additional SDK Issue: “unmute” is Not a Valid Event
</h3>

<p>⚠️ statusCallbackEvent Correction The statusCallbackEvent parameter does NOT include an “unmute” event. Valid values are: start, end, join, leave, mute, hold, modify, speaker, announcement. Twilio fires the participant-mute event for both mute and unmute actions. The Muted field in the webhook body (‘true’ or ‘false’) tells you which occurred.</p>

<p><code></code>`<br>
// Correct status callback handler<br>
app.post('/webhooks/conference', (req, res) =&gt; {<br>
  const { StatusCallbackEvent, CallSid, Muted } = req.body;</p>

<p>if (StatusCallbackEvent === 'participant-mute') {<br>
    const isMuted = Muted === 'true';   // "true" or "false" as strings<br>
    // isMuted = true  → participant was muted<br>
    // isMuted = false → participant was unmuted<br>
    updateParticipantState(CallSid, { muted: isMuted });<br>
    broadcastToAdmins({ type: isMuted ? 'participant_muted' : 'participant_unmuted', callSid: CallSid });<br>
  }</p>

<p>res.sendStatus(200);<br>
});</p>

<p>`<code></code></p>

<p>02</p>
<h2>
  
  
  Verified Working Patterns — The POC Implementation
</h2>

<p>The POC combines two approaches that work within Twilio’s actual TwiML constraints. Together they cover the hand-raise use case without any additional Twilio services.</p>
<h3>
  
  
  Pattern A — Used in POC
</h3>

<p><strong>Gather-Before-Conference</strong><br><br>
Participant gets a short <code>&lt;Gather&gt;</code> window before joining the conference. If they press *1 during this window, hand-raise is registered before entry.</p>

<p>✓ Zero audio interruption ⚡ One-time only</p>
<h3>
  
  
  Pattern B — Used in POC
</h3>

<p><strong>REST API Call Redirect</strong><br><br>
Host triggers mid-conference DTMF prompts via the dashboard. The backend calls <code>twilioClient.calls(callSid).update({ url: gatherUrl })</code>, temporarily pulling the participant out to collect a keypress, then returning them.</p>

<p>✓ Mid-call capable ⚡ Host-initiated</p>
<h3>
  
  
  Pattern A — Full Code: Gather-Before-Conference
</h3>

<p><code></code>`<br>
// Inbound call → welcome + DTMF window → then conference<br>
app.post('/voice/incoming', (req, res) =&gt; {<br>
  const twiml = new VoiceResponse();</p>

<p>twiml.say({ voice: 'Polly.Joanna' },<br>
    'Welcome. You are joining the conference muted. ' +<br>
    'Press star 1 to raise your hand before entering.');</p>

<p>// Gather window — participant can press *1 NOW<br>
  const gather = twiml.gather({<br>
    input: 'dtmf',<br>
    action: '/voice/pre-join-dtmf',<br>
    timeout: 4,         // 4 seconds to press a key<br>
    numDigits: 2,<br>
    finishOnKey: '',<br>
  });<br>
  gather.say({ voice: 'Polly.Joanna' }, 'Press star 1 now, or hold to join.');</p>

<p>// Fall-through: no key pressed → enter conference muted<br>
  const dial = twiml.dial();<br>
  dial.conference({<br>
    muted: true,<br>
    startConferenceOnEnter: false,<br>
    endConferenceOnExit: false,<br>
    statusCallback: <code>${BASE_URL}/webhooks/conference</code>,<br>
    statusCallbackEvent: ['start', 'end', 'join', 'leave', 'mute', 'hold'],<br>
    waitUrl: <code>${BASE_URL}/hold-music</code>,<br>
  }, 'MainRoom');</p>

<p>res.type('text/xml').send(twiml.toString());<br>
});</p>

<p>// Handle pre-join keypress<br>
app.post('/voice/pre-join-dtmf', async (req, res) =&gt; {<br>
  const { Digits, CallSid } = req.body;<br>
  const twiml = new VoiceResponse();</p>

<p>if (Digits === '*1') {<br>
    await markHandRaised(CallSid);   // store in state, push to dashboard WS<br>
    twiml.say({ voice: 'Polly.Joanna' }, 'Your hand has been raised. Joining now.');<br>
  }</p>

<p>// Either way, enter the conference<br>
  const dial = twiml.dial();<br>
  dial.conference({ muted: true, ... }, 'MainRoom');</p>

<p>res.type('text/xml').send(twiml.toString());<br>
});</p>

<p>`<code></code></p>
<h3>
  
  
  Pattern B — Full Code: REST Redirect for Mid-Call DTMF
</h3>

<p>The dashboard shows a “Prompt Hand Raise” button per participant. When clicked, the backend redirects that caller’s active call to a gather TwiML page, collects their response, then returns them to the conference.</p>

<p><code></code><code><br>
// Dashboard API: host triggers DTMF prompt for a specific participant<br>
app.post('/api/prompt-hand-raise/:callSid', authenticateHost, async (req, res) =&gt; {<br>
  const { callSid } = req.params;<br>
  try {<br>
    // Redirect their active call to a gather prompt<br>
    await twilioClient.calls(callSid).update({<br>
      url:</code>${BASE_URL}/voice/gather-hand-raise`,<br>
      method: 'POST',<br>
    });<br>
    res.json({ success: true });<br>
  } catch (err) {<br>
    res.status(500).json({ error: err.message });<br>
  }<br>
});</p>

<p>// The gather prompt TwiML page<br>
app.post('/voice/gather-hand-raise', (req, res) =&gt; {<br>
  const twiml = new VoiceResponse();</p>

<p>const gather = twiml.gather({<br>
    input: 'dtmf',<br>
    action: '/voice/mid-call-dtmf',<br>
    timeout: 6,<br>
    numDigits: 2,<br>
  });<br>
  gather.say({ voice: 'Polly.Joanna' },<br>
    'You have been prompted by the host. Press star 1 to raise your hand, ' +<br>
    'or star 2 to decline. Or stay silent to return.');</p>

<p>// Fall-through: no press → rejoin conference<br>
  const dial = twiml.dial();<br>
  dial.conference({ muted: true }, 'MainRoom');</p>

<p>res.type('text/xml').send(twiml.toString());<br>
});</p>

<p>// Process their response<br>
app.post('/voice/mid-call-dtmf', async (req, res) =&gt; {<br>
  const { Digits, CallSid } = req.body;<br>
  const twiml = new VoiceResponse();</p>

<p>if (Digits === '*1') {<br>
    await markHandRaised(CallSid);<br>
    twiml.say({ voice: 'Polly.Joanna' }, 'Hand raised. Returning you to the conference.');<br>
  } else {<br>
    twiml.say({ voice: 'Polly.Joanna' }, 'Returning you to the conference.');<br>
  }</p>

<p>const dial = twiml.dial();<br>
  dial.conference({ muted: true }, 'MainRoom');<br>
  res.type('text/xml').send(twiml.toString());<br>
});</p>

<p>`<code></code></p>

<p>ℹ️ Trade-off of Pattern B The participant is briefly disconnected from conference audio (~3–5 seconds) while the gather prompt plays. This is host-initiated — the participant cannot trigger it themselves from inside the conference. Pattern B is a usable POC solution, but the production path should upgrade to Media Streams (Section 03) for true participant-initiated hand-raises.</p>

<p>03</p>
<h2>
  
  
  Alternative 1 — Media Streams + Server-Side DTMF Detection
</h2>
<h3>
  
  
  Twilio Media Streams + Goertzel Algorithm
</h3>

<p>Raw audio piped to your WebSocket server — detect DTMF tones in real-time, participant never leaves the conference</p>

<p>Production Grade No Audio Interruption Participant-Initiated</p>

<p>Twilio Media Streams sends a raw audio stream (8kHz mulaw) from each participant’s call to a WebSocket endpoint on your server. You run a DTMF tone detector (Goertzel algorithm) on this audio stream. When a key is pressed, the tone is detected server-side without ever pulling the participant out of the conference.</p>
<h3>
  
  
  How it Works
</h3>

<p>Participant presses *1<br><br>
Phone keypad → Twilio streams audio 8kHz mulaw via WS → Your WS server Goertzel detector → Hand-raise event Dashboard notified</p>
<h3>
  
  
  TwiML — Enable Media Stream
</h3>

<p><code></code>`<br>
<br>
  <br>
    &lt;!-- Stream audio from this call to your WebSocket server --&gt;<br>
    
             track="inbound_track" /&gt;<br>
  <br>
  Joining the conference. Press star 1 any time to raise your hand.<br>
  <br>
    
                statusCallback="..."<br>
                statusCallbackEvent="start end join leave mute hold"&gt;<br>
      MainRoom<br>
    <br>
  <br>
</p>

<p>`<code></code></p>

<h3>
  
  
  Server-Side DTMF Detector (Node.js)
</h3>

<p><code></code>`<br>
import WebSocket from 'ws';<br>
import { GoertzelDTMFDetector } from './dtmf-detector';  // or npm: node-dtmf</p>

<p>const mediaWss = new WebSocket.Server({ path: '/media-stream', server });</p>

<p>mediaWss.on('connection', (ws) =&gt; {<br>
  let callSid: string;<br>
  const detector = new GoertzelDTMFDetector({ sampleRate: 8000 });</p>

<p>ws.on('message', (raw: string) =&gt; {<br>
    const msg = JSON.parse(raw);</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>if (msg.event === 'start') {
  callSid = msg.start.callSid;   // map stream → call SID
}

if (msg.event === 'media') {
  // Decode base64 mulaw audio payload
  const audio = Buffer.from(msg.media.payload, 'base64');

  // Run Goertzel DTMF detector on this audio chunk
  const digit = detector.detect(audio);

  if (digit) {
    handleDTMFDigit(callSid, digit);
  }
}

if (msg.event === 'stop') {
  detector.reset();
}
</code></pre>

</div>

<p>});<br>
});</p>

<p>async function handleDTMFDigit(callSid: string, digit: string) {<br>
  // Debounce — same digit repeated quickly = one press<br>
  if (digit === '<em>' || digit === '1') {<br>
    // You'll receive '</em>' then '1' as separate detections<br>
    // Buffer them with a short debounce window<br>
    bufferDigit(callSid, digit, async (fullDigit) =&gt; {<br>
      if (fullDigit === '*1') {<br>
        await markHandRaised(callSid);     // update state<br>
        broadcastToAdmins({                // push to dashboard<br>
          type: 'hand_raised',<br>
          callSid,<br>
          callerId: getCallerInfo(callSid).callerId,<br>
        });<br>
      }<br>
    });<br>
  }<br>
}</p>

<p>`<code></code></p>

<h3>
  
  
  DTMF Detection Library Options
</h3>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Library</th>
<th>Language</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>node-dtmf</td>
<td>Node.js</td>
<td>Simple Goertzel implementation for mulaw audio</td>
</tr>
<tr>
<td>goertzel-js</td>
<td>Node.js</td>
<td>Low-level Goertzel filter, needs DTMF freq mapping</td>
</tr>
<tr>
<td>dtmf-decoder</td>
<td>Python</td>
<td>Good if your backend is Python/FastAPI</td>
</tr>
<tr>
<td>librosa + custom</td>
<td>Python</td>
<td>Overkill but very accurate</td>
</tr>
</tbody>
</table></div>

<h3>
  
  
  Advantages
</h3>

<ul>
<li>  Participant stays in conference — zero audio interruption</li>
<li>  True participant-initiated hand-raises at any time</li>
<li>  No additional Twilio cost — Media Streams included in Voice</li>
<li>  Works with any keypad digit combination you define</li>
</ul>

<h3>
  
  
  Challenges
</h3>

<ul>
<li>  Requires a WebSocket server to receive audio</li>
<li>  Need DTMF detection library + Goertzel implementation</li>
<li>  Higher server resource usage (audio processing per participant)</li>
<li>  Multi-digit debouncing logic needed (*1 = two signals)</li>
</ul>

<p>`<code>\</code></p>

