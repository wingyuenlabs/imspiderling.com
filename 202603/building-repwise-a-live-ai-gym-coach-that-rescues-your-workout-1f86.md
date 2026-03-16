---
Title: Building RepWise: A Live AI Gym Coach That Rescues Your Workout
Description: 
Author: Sujnesh
Date: 2026-03-16T21:55:05.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Building RepWise: A Live AI Gym Coach That Rescues Your Workout
</h1>

<p><em>This article was created for the purposes of entering the <a href="https://geminiliveagentchallenge.devpost.com/" rel="noopener noreferrer">Gemini Live Agent Challenge</a>.</em></p>




<h2>
  
  
  The Problem
</h2>

<p>Most people don't quit workouts because they're lazy. They quit because something breaks.</p>

<p>The bench is taken. Energy crashes mid-session. A movement hurts. You don't recognize the exercise your plan says to do next. You only have 15 minutes left instead of 30.</p>

<p>These are real friction points that happen in every gym, every day. And no fitness app handles them. They generate a plan, hand it to you, and disappear.</p>

<p>I wanted to build something that stays with you — and adapts when things go wrong.</p>

<h2>
  
  
  What RepWise Does
</h2>

<p>RepWise is a beginner-friendly, friction-aware live gym companion that runs in the browser.</p>

<p>Here's the flow:</p>

<ol>
<li><p><strong>Quick Setup</strong> — You tell RepWise your level (beginner/intermediate), goal (build muscle, lose fat, quick session), available time, equipment, energy level, and context (crowded gym, low confidence, soreness).</p></li>
<li><p><strong>Mission Generation</strong> — Gemini generates a personalized workout plan with exercises, sets, reps, warmup, and coaching cues tailored to your inputs.</p></li>
<li><p><strong>Live Coaching</strong> — Your phone camera tracks your movement using MediaPipe pose detection. RepWise counts reps, tracks sets with rest intervals, detects form issues (leaning forward, insufficient depth), and provides real-time<br>
coaching cues from Gemini.</p></li>
<li><p><strong>Workout Rescue</strong> — This is the core differentiator. When friction hits, you tap Rescue and select a reason: Equipment Busy, Low Energy, Short on Time, Discomfort, Don't Know This Exercise, or Easier Please. RepWise instantly<br>
rewrites the remaining workout while preserving your original goal.</p></li>
<li><p><strong>Mission Complete</strong> — A grounded summary showing actual exercises performed, sets completed, form score, and a Gemini-generated coaching debrief with next-step recommendations.</p></li>
</ol>

<h2>
  
  
  Why Workout Rescue Matters
</h2>

<p>Rep counting and form detection are useful, but they're expected. Every fitness AI does some version of that.</p>

<p>Workout Rescue solves a different problem: <strong>keeping the session alive when real-world conditions change.</strong> It's the moment where the AI stops being a tracker and starts being a coach — one that makes a decision under changing<br>
  constraints, explains it, and keeps you moving.</p>

<p>That's the kind of agency the Gemini Live Agent Challenge is about.</p>

<h2>
  
  
  How Gemini Is Used
</h2>

<p>RepWise uses Gemini in three distinct ways:</p>

<ul>
<li><p><strong>Gemini Live API (via ADK bidi-streaming)</strong> — Real-time coaching during the workout session. Pose events and session context are streamed to Gemini, which responds with contextual coaching cues via WebSocket.</p></li>
<li><p><strong>Gemini 2.5 Flash (structured planning)</strong> — Workout generation, warmup creation, and exercise adaptation. The model returns structured JSON that the backend parses into exercise plans with sets, reps, and cues.</p></li>
<li><p><strong>Gemini 2.5 Flash (reasoning)</strong> — Post-workout debrief generation and workout rescue logic. When rescue is triggered, Gemini evaluates the remaining plan against the new constraint and produces a coherent adaptation.</p></li>
</ul>

<h2>
  
  
  How Google Cloud Is Used
</h2>

<ul>
<li><p><strong>Cloud Run</strong> — The FastAPI backend is containerized and deployed to Cloud Run. Handles WebSocket connections for live coaching, REST APIs for session management, and Gemini API calls.</p></li>
<li><p><strong>Cloud Firestore</strong> — Stores workout sessions with full rep event data, form scores, coaching summaries, and anonymous user profiles for continuity across sessions.</p></li>
<li><p><strong>Automated deployment</strong> — A single <code>deploy.sh</code> script enables, provisions, and deploys the entire stack.</p></li>
</ul>

<h2>
  
  
  Tech Stack
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Layer</th>
<th>Technology</th>
</tr>
</thead>
<tbody>
<tr>
<td>Frontend</td>
<td>JavaScript, HTML, CSS</td>
</tr>
<tr>
<td>Pose Detection</td>
<td>MediaPipe Tasks Vision (in-browser)</td>
</tr>
<tr>
<td>Backend</td>
<td>Python, FastAPI</td>
</tr>
<tr>
<td>AI Coaching</td>
<td>Gemini Live API + ADK</td>
</tr>
<tr>
<td>AI Planning</td>
<td>Gemini 2.5 Flash</td>
</tr>
<tr>
<td>Deployment</td>
<td>Google Cloud Run, Docker</td>
</tr>
<tr>
<td>Persistence</td>
<td>Google Cloud Firestore</td>
</tr>
<tr>
<td>Testing</td>
<td>Pytest (111 tests)</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  What I Learned
</h2>

<p><strong>MediaPipe in the browser is surprisingly capable.</strong> Running pose detection at 10fps with joint angle computation and rep counting — all client-side — keeps the architecture simple and latency low.</p>

<p><strong>Semantic events beat raw data.</strong> Sending Gemini a structured pose event like <code>{"exercise": "squat", "rep": 8, "form_issue": "leaning_forward"}</code> produces much better coaching than sending raw landmark coordinates.</p>

<p><strong>Fallbacks are non-negotiable for demos.</strong> Every Gemini call has a deterministic fallback. Workout generation, rescue, warmup, debrief — all have local logic that activates if the API is slow or unavailable. The demo never breaks.</p>

<p><strong>Beginner/intermediate branching changes everything.</strong> The same app feels meaningfully different for a beginner (3 simple exercises, reassuring cues, longer rest) versus an intermediate user (more exercises, tighter language,<br>
  performance framing). Small backend logic, big UX impact.</p>

<h2>
  
  
  Try It
</h2>

<ul>
<li>
<strong>Live</strong>: <a href="https://repwise-384586125133.us-central1.run.app" rel="noopener noreferrer">repwise-384586125133.us-central1.run.app</a>
</li>
<li>
<strong>Code</strong>: <a href="https://github.com/sujnesh/repwise" rel="noopener noreferrer">github.com/sujnesh/repwise</a>
</li>
</ul>




<p><em>Built for the <a href="https://geminiliveagentchallenge.devpost.com/" rel="noopener noreferrer">Gemini Live Agent Challenge</a> — March 2026.</em></p>

