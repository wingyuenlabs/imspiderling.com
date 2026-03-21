---
Title: What's Device Fingerprinting?
Description: 
Author: Shadai Scott
Date: 2026-03-21T21:46:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Imagine this. A user clears their cookies, switches to incognito mode or connects to a VPN and your webapp still recognizes them. That’s what device fingerprinting is. The official definition is that it’s a technique used to identify and track unique users by collecting specific data points from their browser, hardware, and operating system. This article explains how it works and how it fits into your security stack. </p>

<p><strong>The Problem with Cookies</strong></p>

<p>Cookies were the original device identifiers but there’s a downside to them. Some browsers block them by default, incognito mode bypass them entirely not to mention that some users delete them. As traditional methods lose effectiveness  device fingerprinting has surged as the alternative. It doesn’t store anything on the user’s device, only read what’s there. </p>

<p><strong>How a fingerprint is built</strong></p>

<p>Device-fingerprinting services create fingerprints based on a combination a number of data points. These include but not limited to: </p>

<ul>
<li>IP address</li>
<li>HTTP Request Headers</li>
<li>User agent string</li>
<li>Client time zone</li>
<li>Information about the client device: screen resolution, touch support, operating system and language</li>
<li>VPN</li>
<li>Battery Information</li>
<li>Mouse Movement</li>
<li>Scroll Velocity</li>
</ul>

<p><strong>Passive vs Active Fingerprinting</strong></p>

<p>Active Fingerprinting: It explicitly interrogates the browser for extra information. </p>

<p>Passive Fingerprinting:  The server simply gathers the information passed by the browser and the different HTTP connection layers.</p>

<p><strong>What makes a fingerprint persistent</strong></p>

<p>Device fingerprinting is persistent because it relies on signals that are inherent to the hardware and software configuration, not stored data. Fraudsters can attempt to manipulate their fingerprint through factory resets or OS changes but modern fingerprinting systems use machine learning similarity detection to recognize devices even as their fingerprints shift.</p>

<p><strong>How it fits into a security stack</strong></p>

<p>In a Zero Trust model, no device is implicitly trusted even after a successful login. Fingerprinting becomes a foundational signal for maintaining ongoing confidence in device integrity, with every request evaluated against past trusted sessions to detect anomalies. This should be a layer of your security and not a complete solution. </p>

