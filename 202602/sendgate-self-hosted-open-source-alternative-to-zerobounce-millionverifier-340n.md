---
Title: SendGate — self-hosted open-source alternative to ZeroBounce / MillionVerifier
Description: 
Author: Ihor Filippov
Date: 2026-02-05T22:07:06.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hey everyone.</p>

<p>About a month ago I open-sourced Senlo. Right now the project has around 150 stars. Maybe not huge numbers, but for my first OSS experience it honestly feels great.</p>

<p>Lately I was spending most of my time working on an email validation package. But since i decided to focus <a href="https://senlo.io/" rel="noopener noreferrer">Senlo</a> more on transactional and product lifecycle emails, the need for this validation logic inside the main project kind of disappeared. so instead of letting that work just sit there, i decided to release it on github as a separate self-hosted service.</p>

<p>It’s basically a lightweight self-hosted email validation and risk analysis utility. You run it on your own server, no database or external accounts required.</p>

<p>It checks email syntax, looks up mx records, and inspects spf / dmarc configuration to understand how the domain is set up. It also tries to detect catch-all domains, disposable providers, and role-based addresses like info@ or support@ that usually behave differently in campaigns.</p>

<p>Instead of only returning valid / invalid, it gives a simple risk score based on multiple small heuristics — missing auth records, unusual mx setups, shared or suspicious infrastructure patterns, temporary-looking domains, and similar signals. Everything is processed in memory, no email lists are stored, and you can use it through a small rest api or run bulk csv checks locally. it’s more of a developer / internal utility than a full platform.</p>

<p>Here is the <a href="https://github.com/IgorFilippov3/sendgate" rel="noopener noreferrer">link to repo</a> and <a href="https://www.sendgate.io/" rel="noopener noreferrer">to demo</a> .</p>

<p>If you find it useful, a star or any feedback on github would mean a lot.</p>

