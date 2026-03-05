---
Title: How to setup React Native / expo push notification according to different types of notification
Description: 
Author: Pierre
Date: 2026-03-05T21:28:23.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hi, I am actually desperate.<br>
I need to setup push notification on my IOS/Android app, I buold with React Native / Expo.</p>

<p>I could setup a simple push notification follwing <a href="https://docs.expo.dev/versions/latest/sdk/notifications/" rel="noopener noreferrer">this</a> and <a href="https://medium.com/@nnaemekaonyeji27/react-native-push-notifications-made-simple-ios-android-0d5995f202ab" rel="noopener noreferrer">this</a> but my big problem is the following:</p>

<p>I have several sensors into several field. A fermer have installed the App on his/her smartphone and they can monitor the fields.</p>

<p>The farmers need to be alarmed with a push notification, when a sensor measures a measure under a threshold value. A farmer can own several fields (let's say 6 fields)</p>

<p>I want that farmer can active or not a field to be monitor and to be alarmed for the active fields.</p>

<p>For example, </p>

<ul>
<li>if he do not active a field, he should never receive a push notification, excepted for the general notification (let's say the type "general"</li>
<li>if he active the field 2 and 4 and 5, he should reveive only push notification matching to the sensors on field 2,4 and 5.</li>
</ul>

<p>On my PHP scrypt (on my remote server) I can send push notification, but the script should send a push notification to the IOS/Addroid which are registered for the active field. (2,4 and 5 according to my above example)</p>

<p>I spent days and weeks to search how I could registered the App of a smartphone according to my needs.</p>

<ul>
<li>Step 1: the farmer open the App. The app is registred for push notification</li>
<li>Step 2: The farmer go the the setting view</li>
<li>Step 3: The farmer chose and select the fields he want to monitor</li>
</ul>

<p>How Expo notification can save the selected field as channel or categories?<br>
And how can my php script know on which devise it has to send the push notification</p>

<p>Other scenarios:<br>
Smartphone 1 saved the fields 1, 2 ,3<br>
Samrtphone 2 saved the field 2,4,6<br>
Smartphone 3 saved the field 3<br>
Smartphone 4 saved the field 4,5<br>
Smartphone 5 saved the field 6</p>

<p>On the morning, a station on field 6 run into alarm. How can I setup Expo to send a push notification to the Samrtphone 2 and 5 only</p>

<p>On the after noon, the stations on fields 2 and 4 run into alarm. How can I setup Expo to send a notification to the Smartphone 1,2 and 4 only?</p>

<p>I want to inform an update, how can I send a push notification to all Samrtphone</p>

<p>I kindly thank for your help, because I am lost :)</p>

<p>Cheers</p>

