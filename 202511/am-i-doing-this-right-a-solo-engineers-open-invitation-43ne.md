---
Title: Am I Doing This Right? A Solo Engineer's Open Invitation
Description: 
Author: JaredHarbison
Date: 2025-11-22T21:40:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>I'm a senior software engineer with almost six years of experience, mostly in Rails and React. For the past several years, I've been the only engineer at my company.<br>
That means I own everything—architecture, implementation, deploys, firefighting. It also means I don't have anyone to tell me when I'm doing something weird.</p>

<p>I've worked on teams before. I know the value of code review, of someone asking "why not just...?" before you're three layers deep in a bad abstraction. I miss that. So I'm trying to build it for myself, here.</p>

<p>This series is called "Am I Doing This Right?" and the premise is simple: I'll walk through something I've built or a decision I've made, explain my reasoning, and invite you to weigh in. Tell me how your team does it. Tell me I'm overcomplicating it. Tell me about the footgun I haven't noticed yet.</p>

<p>I'm not looking for validation. I'm looking for the perspective that comes from working with other engineers—something I don't get day-to-day.</p>

<p>First up: a GitHub Actions workflow that posts to Slack with recent commits and links to SOPs whenever we deploy. It's a small thing, but it touches CI/CD, team communication, and documentation habits, so there's plenty to pick apart.</p>

