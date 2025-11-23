---
Title: ZetaShare | Building Truly private file transfer with WebRTC
Description: 
Author: maste
Date: 2025-11-23T21:34:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>I built ZetaShare because I wanted a file transfer service that doesn’t spy on me — no tracking, no data collection. The project was inspired by ToffeeShare, but after seeing it abandoned and being unable to contact the developers, I decided to build my own alternative.</p>

<p>In case you're curious about how the system works under the hood, here’s a simplified explanation of the current architecture:</p>

<p>How ZetaShare works (simplified):</p>

<p>The sender uploads a file on the website. The server doesn’t store the file — After uploading, the sender receives a 6‑digit ID. This ID becomes the identifier for the transfer session.</p>

<p>The receiver opens the link with the ?id= parameter, for example: <a href="https://zetashare.com/?id=123456" rel="noopener noreferrer">https://zetashare.com/?id=123456</a></p>

<p>When the receiver joins the link, the server sends the receiver’s SDP offer to the sender. (This is part of the WebRTC signaling process.)</p>

<p>The sender responds by sending back their own SDP answer.</p>

<p>Both sides then exchange ICE candidates through the server.</p>

<p>After the WebRTC handshake is complete, a direct P2P connection is established between sender and receiver — and the file starts transferring directly between the two devices.</p>

<p>So the server is only used for:</p>

<p>exchanging SDP offer/answer</p>

<p>relaying ICE candidates …and nothing else. No file, no metadata, no tracking.</p>

<p>Once the WebRTC connection is established, all data flows peer‑to‑peer, fully outside the server.</p>

<p>What works now:</p>

<p>WebRTC P2P transfers — files go directly between users.</p>

<p>No metadata collection — unlike other services that send file metadata through signaling servers (like ToffeeShare), our server only handles SDP and ICE candidates.</p>

<p>No accounts required — just drag, drop, and share.</p>

<p>Unlimited file sizes.</p>

<p>Completely free — no ads at all.</p>

<p>What’s missing (in my view):</p>

<p>Only TLS (WebRTC’s built-in), no proper end-to-end encryption yet.</p>

<p>No password protection.</p>

<p>Links will have an expiration option soon.</p>

<p>Mobile file transfer speed needs improvement.</p>

<p>Coming next:</p>

<p>Password protection</p>

<p>File expiration option</p>

<p>Mobile performance improvements</p>

<p>Proper end-to-end encryption</p>

<p>Try it out: <a href="https://zetashare.com" rel="noopener noreferrer">https://zetashare.com</a></p>

<p>These are the features I could think of, but if you have better ideas, I’d love to hear them. I can’t promise to build everything, but I’ll definitely try.</p>

<p>I’m using AI to help me because my English isn’t perfect — sorry if it sounds a bit AI-generated!</p>

