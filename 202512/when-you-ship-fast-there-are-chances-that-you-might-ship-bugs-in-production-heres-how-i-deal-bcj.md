---
Title: When you ship fast, there are chances that you might ship bugs in production, Here's how I deal with that
Description: 
Author: Mark Kaave
Date: 2025-12-16T22:00:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey everyone , since we're all shipping startups in days and moving with speed, i noticed that a lot of times people ship and dont have a way to track errors or check if their users hit any bugs in production.</p>

<p>Most people add an option on their site where users can give feedback or report, but i've actually never used that. its just stressful to me honestly. funny thing is , i faced this challenge once , where i launched and users rage quit my saas because they hit errors.</p>

<p>i recently started working on something that solves this for me. although there are other error trackers out there like sentry and the rest , those are kind of an overkill for me , and the dashboards look too overwhelming for a simple project. so i built Bugmail(it literally looks like gmail) and its very easy to setup. bugmail notifies you (either through gmail or discord or slack) when something breaks in production and it shows you the event that caused the bug(breadcrumbs).</p>

<p>I initially built this for myself alone , but i realized there might be other devs out there who face this same issue of not having a way to track errors and the existing solutions out there are too much stress to setup or are bloated. so i put bugmail out there and its free to get started</p>

<p><a href="https:://www.bugmail.site" rel="noopener noreferrer">Bugmail</a></p>

