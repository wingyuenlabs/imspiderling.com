---
Title: I built a tool to track US Political Donations (and won an award for it)
Description: 
Author: Alvaro Sacasa
Date: 2025-12-29T21:11:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>The Problem with "Public" Data</p>

<p>We all know that government data is technically "public," but that doesn't mean it's accessible. If you've ever tried to pull large datasets from government portals, you know the pain: antiquated UIs, weird CSV exports, and complex pagination.</p>

<p>I wanted to solve this for US Campaign Finance data. Journalists and researchers need to see who is funding whom, but they often spend more time cleaning data than analyzing it.</p>

<p><strong>The Solution: Wrapping the OpenFEC API</strong></p>

<p>We built the FEC Campaign Finance Scraper, an Apify Actor that wraps the official OpenFEC API to make this data consumable at scale.</p>

<p>Instead of writing custom scripts for every query, developers and researchers can now just input their parameters (committee IDs, date ranges, etc.) and get back clean, structured JSON.</p>

<p>Key Technical Challenges we solved:</p>

<ul>
<li>Pagination Hell: Automating the deep pagination required to fetch thousands of contribution records.</li>
<li>Rate Limiting: Handling the FEC's API limits gracefully so the scraper doesn't get blocked mid-job.</li>
<li>Data Normalization: structuring the output so it's ready for visualization libraries or SQL databases immediately.</li>
</ul>

<p><strong>The Unexpected Win!</strong></p>

<p>We built this primarily to help the ecosystem, but this week we received some amazing validation. Apify selected our project as the winner of their "Social Good" Spotlight!</p>

<p>It was a great reminder that as developers, our side projects can have a real impact on transparency and democracy. It’s not always about building the next SaaS, sometimes it’s about making information free.</p>

<p>Try it out</p>

<p>If you are building in the Civic Tech or GovTech space, I’d love for you to stress-test it.</p>

<p>The Actor: <a href="https://apify.com/parseforge/fec-campaign-finance-contributions-scraper" rel="noopener noreferrer">https://apify.com/parseforge/fec-campaign-finance-contributions-scraper</a></p>

<p>The Goal: To help journalists track money in politics without needing a CS degree.</p>

<p>Question for the community: Have you worked with other government APIs? Which ones are the most painful to integrate? I’m looking for our next project!</p>

