---
Title: From branches to trunk (a journey)
Description: 
Author: Markus Decke
Date: 2026-01-20T22:01:29.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Preface
</h2>

<p>In environments where people are used to using some sort of Gitflow variant, the norm is having <code>main</code>, <code>develop</code>, and long-lived feature branches. There are often not enough good automated tests and use-case coverage to deliver with confidence, so branch isolation and friction-creating processes are used to ensure some quality. At least having a DEV stage where builds get deployed after they successfully created a container image is the usual case. This doesn't mean there is an automated release process or additional stage for testing prior to a release, also known as merging into <code>main</code>.</p>

<p>This means the DEV stage can be a moving target for testers, if the first stable stage is the INT stage, where Releases (2 weeks of work) are deployed for delivery towards production. Here, the testing works on a fixed set of features and stable servers, without the surprises of new features popping up when something is merged.</p>

<p>Since Releases happen every 2 weeks, people can be eager to merge instead of waiting another 2 weeks when they are nearly but not yet done, especially clients and project managers, who can get restless. So we get bugs that get caught on INT, and then we need to create hotfixes and test again. If there were merges in between, the fix needs to be cherry-picked, which means the hotfixed Release isn't the same set of features as the current <code>develop</code> branch. </p>

<h2>
  
  
  Current setup
</h2>

<p>Delivery Stages</p>

<ul>
<li>DEV (develop) - <code>develop</code> branch gets deployed after merge (build takes hours)</li>
<li>INT (Integration) - <code>main</code> branch gets deployed every 2 weeks after release build</li>
<li>...</li>
<li>PROD (Production) - gets deployed 2 weeks after INT</li>
</ul>

<p>This means the state on INT is not the same; if errors occur, we need to have a hotfix, and this means more friction. <br>
The often-used solution is to have a merge-freeze on <code>develop</code> until Release, to ensure everything on DEV gets tested. So <code>main</code> in the end is the Release branch and has no other function. You could also tag a commit in <code>develop</code> and release that without going through the motions of PRs from <code>develop</code> to <code>main</code>, which is done by hand; this is cumbersome and creates a lot of friction and therefore waste. </p>

<p>A new idea is to have a TST (testing) stage, so that a fixed set can be tested and released.</p>

<h2>
  
  
  New setup
</h2>

<p>Delivery Stages</p>

<ul>
<li>DEV (develop) - <code>develop</code> branch gets deployed after merge (build still needs hours)</li>
<li>TST (testing) - a fixed state of <code>develop</code> can be tested and then merged into main</li>
<li>INT (Integration) - <code>main</code> branch gets deployed every 2 weeks after release build</li>
<li>...</li>
<li>PROD (Production) - gets deployed 2 weeks after INT</li>
</ul>

<p><strong>Question</strong>: How do we get the tested commits into <code>main</code> after new merges into develop?<br>
<strong>Answer</strong>: More branches!</p>

<h2>
  
  
  Branches everywhere?
</h2>

<p>So now we have <code>develop</code> and <code>main</code> as branches with the new setup, we need to branch from develop with a certain set of commits to merge into <code>main</code>. <br>
One way could be to create a new branch from <code>develop</code> and merge into <code>main</code>, so that we still have PRs and manual actions and waste.<br>
Instead, we could tag a commit and merge the tag into <code>main</code>, which we can do by hand or we can automate.</p>

<h2>
  
  
  Moving towards trunk!
</h2>

<p><strong>Question</strong>: If we do this, do we really need <code>main</code> anymore? <br>
<strong>Answer</strong>: I think not.</p>

<p><strong>Question</strong>: Could we instead work and release only <code>develop</code>? <br>
<strong>Answer</strong>: Yes, let's call it <code>trunk</code> instead of <code>develop</code>.</p>

<p>We could tag a commit in <code>develop</code> and release the tagged commits, that way we don't need an extra branch and manual motions to get the commits into it. Less manual work and friction, less waste.<br>
Depending on your view, we could call it successfully moving towards trunk-based development.</p>

<p>What do you think? Let me know in the comments. </p>

<p>For me next stop in the journey is moving towards continuous delivery :)</p>

