---
Title: How to Build and Test iOS Apps on a Physical Phone: Adding External Testers (Part 4/5)
Description: 
Author: Cathy Lai
Date: 2026-03-02T22:08:11.000Z
Robots: noindex,nofollow
Template: index
---
<p>In <a href="https://dev.to/cathylai/how-to-build-and-test-ios-apps-on-a-physical-phone-testers-and-apple-testflight-part-35-eoo">Part 3</a>, we invited close friends and colleagues to test on their phones (internal testers). Now it's time to invite the wider audience (clients, investors, etc.)!</p>

<h2>
  
  
  External Testers
</h2>

<p>This is reasonably straightforward. Just go to the TestFlight section, find the Add Group button.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1xjz1938aa5pzt6ewv74.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1xjz1938aa5pzt6ewv74.jpg" alt=" " width="800" height="502"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsnqbih90nsnnxgoy8ly6.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsnqbih90nsnnxgoy8ly6.jpg" alt=" " width="800" height="502"></a></p>

<p>The scroll to the bottom of the page, find the list of Builds. Add the Group to a Build.</p>

<p>Notice that the Status will become "Waiting for Review".<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdazyqnlfq4j97w6aafsl.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdazyqnlfq4j97w6aafsl.jpg" alt=" " width="800" height="502"></a></p>

<p>For each new Version, Apple will need a Beta App Review which will usually take <strong>1-2 days</strong>. The Builds under a Version, however, won't need the review, provided no significant changes were made to the app's metadata or permissions. </p>

<p>If we make a new Build, we can enabled "automatic distribution" for emails to be sent out. Otherwise we will need to invite the (External Testers) Group every single time. We can set this up at the time we add a Build to our new Group. Refer to App Store Connect <a href="https://developer.apple.com/help/app-store-connect/test-a-beta-version/invite-external-testers" rel="noopener noreferrer">Invite External Testers section</a></p>
<h2>
  
  
  Process Flow of External Testers
</h2>


<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>[ 1. EAS SUBMIT ] -------&gt; (Run 'eas submit -p ios')
   |                     (Sends build to App Store Connect)
   v
[ 2. APPLE PROCESSING ] -&gt; (Wait 5-20 mins for Apple to "process")
   |
   v
[ 3. TESTFLIGHT TAB ] ----&gt; (Create "External Group")
   |
   v
[ 4. ADD TEST GROUP ] ---------&gt; (External Testers to EAS build)
   | 
   v
[ 5. (EACH VER) BETA REVIEW ] -------&gt; (Apple takes 24-48 hours to approve for each version)
   |
   v
[ 6. TESTFLIGHT App ] ------&gt; (Testers get notified via Email)
</code></pre>

</div>

<h2>
  
  
  Prefer to Watch a Video?
</h2>

<p>Follow me as I walk thru this config process, plus a demo when a user receives an email to use TestFlight - how to download, install, and use our beta app. <br>
<iframe width="710" height="399" src="https://www.youtube.com/embed/XpiziMGis4c">
</iframe>
</p>

