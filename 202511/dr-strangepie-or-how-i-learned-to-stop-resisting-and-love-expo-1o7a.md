---
Title: Dr. Strangepie or: How I Learned to Stop Resisting and Love Expo
Description: 
Author: Kevin Smith
Date: 2025-11-05T21:49:31.000Z
Robots: noindex,nofollow
Template: index
---
<p>In October 2024, I started a new role at Pie, a Chicago-based company  that is building a platform that makes it easier for people to find things to do and make memories with friends. I was excited to join Pie for a few reasons:</p>

<ul>
<li>
<strong>In-person culture</strong> After working remotely for 5 years, I was looking for a position that gave some face-to-face time</li>
<li>
<strong>Values alignment</strong> Pie's mission to cure loneliness was something that resonated with me, personally</li>
<li>
<strong>Stack alignment</strong> I had been working exclusively with React Native for years, and I was excited to continue that when building this product</li>
<li>
<strong>The people</strong> they're pretty cool, I guess</li>
</ul>

<p>I remember when I started, the only downside that I thought was going to be a point of frustration was that the existing React Native app was built with <a href="https://expo.dev" rel="noopener noreferrer">Expo</a>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcnl4fr1ys70dj7r8xswi.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcnl4fr1ys70dj7r8xswi.webp" alt="animation of a person slowly being pied in the face" width="245" height="275"></a></p>
This is how excited I was to work with Expo



<p><br><br>
After a year of working with Pie and Expo, however, I can confidently say that I have drank the Kool-Aid, and have come to enjoy the Expo ecosystem. I can see now why the React Native docs explicitly recommend using Expo when starting a new project. There are quite a few reasons that changed my tune:</p>

<h2>
  
  
  Vast library of native modules
</h2>

<p>As a social media and events app, there are a lot of features that the Pie mobile app requires to keep folks engaged. Have packages available to install in a cinch really helps us deliver speedy delights</p>

<h3>
  
  
  Media modules
</h3>

<ul>
<li>
<strong>expo-camera</strong>, <strong>expo-image-picker</strong>, <strong>expo-media-library</strong>
One of the first experiences users have when registering for Pie is selecting a profile picture. These packages let us give users the option to take a selfie real quick or upload their favorite shot. Users can also upload images for plans they create, as well as share photos in a plan chat.</li>
<li>
<strong>expo-audio</strong> We've started adding tones for feedback, like a little *ding* when someone accepts a friend request.</li>
<li>
<strong>expo-video</strong> Currently in discovery, we're exploring how we can include video in the app</li>
</ul>

<h3>
  
  
  Social modules
</h3>

<ul>
<li>
<strong>expo-contacts</strong> We like giving our users the ability to find friends who've already joined Pie, by letting them sync their contacts with the app.</li>
<li>
<strong>expo-clipboard</strong> Our highest converting share action is folks tapping to quickly copy the link to a plan to their clipboard</li>
<li>
<strong>expo-sms</strong> Users can directly share plans with their friends via SMS</li>
<li>
<strong>expo-sharing</strong> Makes it super simple for users to share plans or profiles in other apps</li>
</ul>

<h3>
  
  
  Plan discovery modules
</h3>

<ul>
<li>
<strong>expo-location</strong> We've started building out a maps feature (👀 <em>expo-maps</em> soon), and user's location lets us find plans that are close to them</li>
<li>
<strong>expo-calendar</strong> Users can easily add plans they've RSVPed to to their calendars</li>
</ul>

<h3>
  
  
  Miscellaneous Delight
</h3>

<ul>
<li>
<strong>expo-store-review</strong> Helps drive our app review metrics</li>
<li>
<strong>expo-haptics</strong> Adds tactile feedback when users do certain actions</li>
<li>
<strong>expo-datadog</strong> Very grateful for this Datadog wrapper library</li>
<li>
<strong>expo-insights</strong> Expo has been very helpful with providing insights on active users of the app.</li>
</ul>

<h2>
  
  
  Expo Services
</h2>

<p>I think one of the primary things that sold me on Expo are the Expo Application Services (EAS) offerings. They definitely simplify so many of our processes at Pie, I can't imagine working without them.</p>

<ul>
<li>
<strong>EAS Build</strong> By far my favorite service, filling a gaping hole left by the closing of App Center. It's a huge relief to have builds available for our team to test out <em>before</em> we submit them to the app stores.</li>
<li>
<strong>EAS Submit</strong> Makes it easy to submit to the app stores</li>
<li>
<strong>EAS Update</strong> We're currently ironing out the details on implementing over the air updates for our app, but we're looking forward to utilizing EAS Update when we get there</li>
<li>
<strong>EAS Workflows</strong> Our QA engineer is reviewing how we can use EAS Workflows to run our automated test suite</li>
</ul>

<p>With the wide range of modules designed to speed up development and the application services available, Expo really lets us ship new features at a rapid rate, while also ensuring our confidence in the stability of our app. Building an app that balances regular usage as well as inspiring people to put their phones down is challenging enough, and Expo helps lighten the load.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbci4fg6fd5vyhi22ybl9.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbci4fg6fd5vyhi22ybl9.gif" alt="A woman leans over a freshly baked pie, wafting its smell to her nose" width="480" height="270"></a></p>
How I feel about using Expo now that I've worked with it for over a year






<h4>
  
  
  <a href="https://getpie.app" rel="noopener noreferrer">Check out Pie!</a>
</h4>

<h4>
  
  
  Get the App
</h4>

<p><a href="https://apps.apple.com/us/app/pie-free-events-cool-people/id1509667820" rel="noopener noreferrer">Apple App Store</a> | <a href="https://play.google.com/store/apps/details?id=com.whippedCream.pumpkinPie" rel="noopener noreferrer">Google Play Store</a></p>

<h4>
  
  
  Stay up-to-date
</h4>

<p><a href="https://www.instagram.com/pie/" rel="noopener noreferrer">Instagram</a> | <a href="https://www.tiktok.com/@piesocial" rel="noopener noreferrer">TikTok</a></p>

