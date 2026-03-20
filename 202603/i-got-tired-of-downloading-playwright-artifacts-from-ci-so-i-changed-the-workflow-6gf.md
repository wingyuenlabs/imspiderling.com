---
Title: I got tired of downloading Playwright artifacts from CI, so I changed the workflow
Description: 
Author: Adnan G
Date: 2026-03-20T21:20:22.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  I got tired of downloading Playwright artifacts from CI — so I changed the workflow
</h1>

<p>Debugging Playwright failures in CI has always felt more manual than it should be.</p>

<p>Not because the data isn’t there — it is.<br><br>
But because it’s scattered.</p>

<p>A typical failure for me looks like this:</p>

<ul>
<li>open CI job
</li>
<li>download artifacts
</li>
<li>open trace viewer locally
</li>
<li>check screenshots
</li>
<li>scroll logs
</li>
<li>try to line everything up
</li>
</ul>

<p>It works… but it’s slow. Especially when multiple tests fail at once.</p>




<h2>
  
  
  The real problem
</h2>

<p>The issue isn’t lack of data.</p>

<p>It’s that there’s no <strong>single place</strong> to understand what happened.</p>

<p>Everything lives in separate files:</p>

<ul>
<li>traces
</li>
<li>screenshots
</li>
<li>logs
</li>
<li>CI output
</li>
</ul>

<p>So debugging turns into stitching together context manually.</p>

<p>It gets worse with:</p>

<ul>
<li>parallel runs
</li>
<li>flaky tests
</li>
<li>multiple failures triggered by the same root cause
</li>
</ul>

<p>At that point you’re not debugging — you’re reconstructing events.</p>




<h2>
  
  
  What I tried instead
</h2>

<p>I wanted to answer one simple question faster:</p>

<blockquote>
<p>“What actually happened in this run?”</p>
</blockquote>

<p>So I changed the workflow.</p>

<p>Instead of downloading artifacts and inspecting things one by one,<br><br>
I pushed everything from a run into a single view.</p>

<p>That view shows:</p>

<ul>
<li>all failed tests across jobs
</li>
<li>traces, screenshots, logs in one place
</li>
<li>failures grouped if they look related
</li>
<li>a short summary of what likely happened
</li>
</ul>

<p>The goal wasn’t to add more data — it was to remove the jumping between tools.</p>




<h2>
  
  
  Example
</h2>

<p>Instead of this:</p>

<ul>
<li>open CI
</li>
<li>download artifacts
</li>
<li>open trace
</li>
<li>go back to logs
</li>
<li>repeat
</li>
</ul>

<p>You just open one link and see:</p>

<ul>
<li>which tests failed
</li>
<li>whether they failed for the same reason
</li>
<li>what the UI looked like at failure
</li>
<li>what the logs say
</li>
</ul>

<p>No downloading, no switching contexts.</p>




<h2>
  
  
  What improved
</h2>

<p>Two things stood out immediately.</p>

<h3>
  
  
  1. Faster triage
</h3>

<p>You can tell pretty quickly if:</p>

<ul>
<li>it’s one bug causing multiple failures
</li>
<li>or a bunch of unrelated issues
</li>
</ul>

<p>That alone saves a lot of time.</p>




<h3>
  
  
  2. Less noise from flakiness
</h3>

<p>Grouping similar failures makes it obvious when:</p>

<ul>
<li>multiple tests break for the same reason
</li>
<li>vs random flakes
</li>
</ul>

<p>Before that, everything just looked like chaos.</p>




<h2>
  
  
  What still isn’t great
</h2>

<p>This still feels like a workaround.</p>

<p>The ecosystem gives you all the pieces,<br><br>
but not a clean way to reason about failures at the run level.</p>

<p>I’m curious how others are handling this today.</p>

<ul>
<li>Do you rely mostly on trace viewer?
</li>
<li>Do you download artifacts every time?
</li>
<li>Any workflows that actually reduce debugging time?</li>
</ul>




<h2>
  
  
  If you’re curious
</h2>

<p>I open-sourced what I’ve been using here:</p>

<p>👉 <a href="https://github.com/adnangradascevic/playwright-reporter" rel="noopener noreferrer">https://github.com/adnangradascevic/playwright-reporter</a></p>

<p>Would love feedback — especially if you’re dealing with a lot of CI failures.</p>

