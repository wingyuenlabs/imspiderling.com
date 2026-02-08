---
Title: I Built BreakFinder: Personalized Surf Forecasts + Full Garmin Ecosystem
Description: 
Author: Philipp
Date: 2026-02-08T21:00:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>I Built BreakFinder: Personalized Surf Forecasts + Full Garmin Integration<br>
I try to build something surfers actually needed. </p>

<p>Today I'm sharing the technical journey, what worked, and what I'd do differently.</p>

<h2>
  
  
  The Problem:
</h2>

<p>I'm a surfer. I check forecasts constantly. But here's the thing:</p>

<p><strong>A 6ft beach break ≠ a 6ft reef break</strong></p>

<p>Most forecast apps show you swell size and wind direction. Cool. But they <br>
don't tell you if the break matches YOUR skill level. <br>
Or if you're about to paddle into something dangerous.<br>
So I spent 2 months building BreakFinder to solve that.</p>




<h2>
  
  
  The Solution: Three Pillars
</h2>

<p><strong>Personalized Condition Scores</strong></p>

<p>Instead of generic forecasts, BreakFinder gives you a <strong>personalized condition <br>
score</strong> based on:</p>

<ul>
<li>Your preferred wave height range</li>
<li>Your preferred wind direction</li>
<li>Your preferred tide</li>
<li>Your preferred wave period</li>
</ul>

<p><strong>Example:</strong></p>

<ul>
<li>Forecast: "6ft swell, 15kt offshore wind"</li>
<li>My app: "🟢 Perfect for you today" (because I'm intermediate + like 4-6ft)</li>
<li>But for a beginner: "🔴 Too big for your level"</li>
</ul>

<p><strong>Full Garmin Integration (100+ Devices)</strong></p>

<p>This was the hardest part. I integrated with Garmin because I use a Garmin <br>
watch and wanted accurate session tracking.</p>

<p><strong>What's included:</strong></p>

<ul>
<li>5 Connect IQ apps (Forecast, Session Tracker, Surf Skate Tracker, Watch Face, 
Breath Training)</li>
<li>Wave counting (how many waves you caught today)</li>
<li>Post-session data sync to your account at BreakFinder</li>
</ul>

<p><strong>Why it was hard:</strong></p>

<p>Garmin's Connect IQ SDK is powerful but poorly documented. I had to:</p>

<ul>
<li>Support 100+ different watch models (each with different capabilities)</li>
<li>Handle offline functionality (watches have limited storage)</li>
<li>Optimize for battery life (can't drain your watch)</li>
<li>Build fallbacks when devices don't support certain features</li>
</ul>

<p>Worth it? <strong>Maybe.</strong> </p>

<p><strong>Skill-Level Spot Matching</strong></p>

<p>The database of 1500+ spots includes difficulty ratings. So when you're trip <br>
planning, you see:</p>

<ul>
<li>Beginner spots: Mellow beach breaks</li>
<li>Intermediate: Reefs with some risk</li>
<li>Advanced: Heavy breaks (Sunset, Pipe territory)</li>
</ul>




<p><strong>The Tech Stack</strong></p>

<p>Fullstack:</p>

<ul>
<li>Rails 7 (API + Monolith)</li>
<li>Hotwire (Turbo + Stimulus)</li>
<li>PostgreSQL for user data + spots</li>
<li>Redis for caching forecasts</li>
<li>Event-Driven-Architecture</li>
</ul>

<p>External:</p>

<ul>
<li>Garmin Connect IQ SDK (C++)</li>
<li>OpenMeteo for first 7 days forecast data</li>
<li>NOAA for 8–16 days forecast data (own blog worth documenting the pain)</li>
<li>Geocoder (Ruby Gem) </li>
<li>Leaflet for spot maps</li>
</ul>

<p>DevOps:</p>

<ul>
<li>Docker for dev environment</li>
<li>Hetzner + Kamal (loving it! So easy and cheap)</li>
<li>GitHub Actions for CI/CD</li>
</ul>




<h2>
  
  
  Current Metrics (2 Months In)
</h2>

<ul>
<li>
<strong>5,000 downloads</strong> (Garmin IQ Store)</li>
<li>
<strong>3.9★ rating</strong> (honest reviews, we support 100+ devices = some quirks and early version had some major issues like no wave counting at all)</li>
<li>1000+ registered surfers</li>
</ul>




<h2>
  
  
  What's Next
</h2>

<h3>
  
  
  Business Side (Launching Feb 2026)
</h3>

<p>Here's where it gets interesting. I'm building a <strong>marketplace for local <br>
guide operators</strong>.</p>

<p>The idea:</p>

<ul>
<li>Small surf shops, guides, crews can list themselves</li>
<li>Traveling surfers can find them directly</li>
<li>Fair pricing (€19-€299/month per business, NO algorithm trap)</li>
<li>No 30% commission like booking platforms</li>
</ul>

<p>This is where the real revenue lives. A solo surfer might pay €5/month. <br>
A local business? They'll pay €50-200/month to reach customers.</p>




<h2>
  
  
  For Other Solo Devs
</h2>

<p>If you're building something:</p>

<ol>
<li>
<strong>Pick a niche you understand deeply</strong> (I am a surf addict)</li>
<li>
<strong>Focus on one metric early</strong> (mine: Garmin support)</li>
<li>
<strong>Write about the technical journey</strong> (not just the marketing angle)</li>
</ol>




<h2>
  
  
  Links
</h2>

<ul>
<li><a href="https://apps.garmin.com/apps/6127de13-7f60-4360-9d45-7fa057de353f" rel="noopener noreferrer">Garmin IQ Store</a></li>
<li><a href="https://breakfinder.surf" rel="noopener noreferrer">Website</a></li>
</ul>




<h2>
  
  
  Questions?
</h2>

<p>I read every comment. Ask me about:</p>

<ul>
<li>Garmin APIs</li>
<li>Rails architecture</li>
<li>Forecasting algorithms</li>
<li>What I'd do differently</li>
<li>Building as a solo dev</li>
</ul>

<p>Let's talk! 🏄‍♂️</p>

