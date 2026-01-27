---
Title: Javascript / JSON,
Description: 
Author: DevonLad
Date: 2026-01-27T22:06:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>I have a JSON file that I wish to sort by a certain field.</p>

<p>I have found the code I am using works fine for sorting strings but when passed a key that is a integer or array an error message is thrown.</p>

<p>I have created a very small version as an example, this can be found in this JSFiddle; [(<a href="https://jsfiddle.net/a4poster/54hjyrnt/)" rel="noopener noreferrer">https://jsfiddle.net/a4poster/54hjyrnt/)</a>]</p>

<p>For example on line 37 - <br>
<em>search_venueDB = approach2Fn(venueDB, "country");</em></p>

<p>If I change this to:<br>
_search_venueDB = approach2Fn(venueDB, "active");</p>

<p>or</p>

<p>search_venueDB = approach2Fn(venueDB, active);_</p>

<p>an error message is thrown.</p>

<p>I know for the JSON in this example I could use a simple for loop but the actual JSON file is a lot larger.</p>

<p>I am a relative newbie to Javascript, so please be lenient with me.  Is their is a better way then I am open to ideas.</p>

