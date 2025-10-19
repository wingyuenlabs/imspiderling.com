---
Title: Day 6 | 50 Projects in 50 Days
Description: 
Author: Augusta Igboama
Date: 2025-10-19T21:16:19.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Scroll Animation
</h2>

<p>This project was about adding simple animations that trigger as you scroll down the page. Each box slides in from the side when it reaches a certain point on the screen.</p>

<p>The idea is to check how far each element is from the top of the viewport and then add a class to trigger a CSS animation when it crosses the “trigger point.”</p>

<p>Here’s the logic in simple terms:</p>

<ol>
<li>Detect the user’s scroll position.</li>
<li>For each element, compare its position to the viewport height.</li>
<li>If it’s visible enough (past the trigger point), add a class to animate it in; if not, remove the class.</li>
</ol>

<p>In CSS, I used the <code>transform</code> property to shift the elements off-screen initially (for example, <code>translateX(400px)</code>), and then bring them back to their normal position (<code>translateX(0)</code>) when the active class is added.</p>

<p>Also, a fun little CSS discovery: you can use <code>even</code> as a value in <code>nth-of-type()</code> selectors (for example, <code>:nth-of-type(even)</code>), which makes alternating styles cleaner.</p>

<p>For a live demo of the project, click <a href="https://augusta08.github.io/50-projects-50-days/06-scroll-animation" rel="noopener noreferrer">here</a></p>

<p>To view the code on GitHub, click <a href="https://github.com/augusta08/50-projects-50-days/tree/main/06-scroll-animation" rel="noopener noreferrer">here</a></p>

<p>On to Day 7 ⚡</p>

