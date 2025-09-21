---
Title: Debugging Circuits and Code
Description: 
Author: Wale1202
Date: 2025-09-21T21:16:03.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Circuits and Code
</h2>

<p>The word “debugging” isn’t just a metaphor. A popular story from the 1940s is from Admiral Grace Hopper.  While she was working on a Mark II computer at Harvard University, her associates discovered a moth stuck in a relay that impeded operation and wrote in a logbook, "First actual case of a bug being found." Although probably a joke, conflating the two meanings of "bug" (biological and defect), the story indicates that the term was used in the computer field at that time. Today, whether you’re building hardware or writing software, the process of finding and fixing problems still follows the same mental blueprint.</p>

<h3>
  
  
  Hardware Debugging Tools
</h3>

<p>In electronics, when something breaks, you can’t see the electrons moving. That’s why certain tools exist to make the invisible visible, for example; </p>

<h4>
  
  
  Multimeter
</h4>

<p>This measures voltage, current, and continuity to confirm whether parts of a circuit are powered and connected. It’s the first stop for diagnosing dead components or breaks in a circuit.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F16ze2tywoj8oaeof6411.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F16ze2tywoj8oaeof6411.jpeg" alt="Multimeter" width="736" height="1308"></a></p>

<h4>
  
  
  Oscilloscope
</h4>

<p>This lets you view voltage changes over time. If a signal is missing or distorted, an oscilloscope can pinpoint the section of the circuit where things go wrong.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0xa5st7yym5nc8ivtxoa.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0xa5st7yym5nc8ivtxoa.jpeg" alt="Oscilloscope" width="601" height="563"></a></p>

<p>These tools don’t fix problems on their own, but they show you where to look.</p>

<h3>
  
  
  Software Debugging Tools
</h3>

<p>In code, the “current” isn’t electrons it’s the data you pass through. You need a different kind of visibility:</p>

<h4>
  
  
  IDE Debuggers
</h4>

<p>This allows you to pause execution, step through code line by line, and watch variables change.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvxs6mn55qp3fv162jnuo.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvxs6mn55qp3fv162jnuo.jpeg" alt="IDE Debuggers" width="735" height="552"></a></p>

<h4>
  
  
  Logging
</h4>

<p>This usually refers to recording events and the system state so you can diagnose and monitor systems.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvjzzqaii6rpkwl9x4zk0.webp" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvjzzqaii6rpkwl9x4zk0.webp" alt="Logging" width="800" height="450"></a></p>

<p>These tools work like the hardware tools; they reveal hidden processes so you can find the fault.</p>

<h3>
  
  
  A Debugging Mindset is Hypothesize → Test → Fix
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0pgit4pz9cue70z36442.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0pgit4pz9cue70z36442.jpg" alt="Hypothesize → Test → Fix" width="800" height="800"></a></p>

<p>Whether hardware or software, the same mental loop applies:</p>

<p><strong>Hypothesize</strong> - You ponder where the failure might occur based on evidence or error logs.</p>

<p><strong>Test</strong> - Measure a voltage, step through a function, or check logs.</p>

<p><strong>Fix</strong> - Cross off one possible cause at a time until you get to the real problem.</p>

<p>This works like the scientific method applied to engineering: build a theory, run an experiment, refine your theory, and then repeat over and over.</p>

<h3>
  
  
  Case Study
</h3>

<h4>
  
  
  1. Broken LED Circuit (Hardware)
</h4>

<p><strong>Hypothesize</strong><br>
The LED isn’t lighting up. You suspect no electricity is reaching the LED or that the polarity is wrong. Based on the symptom (voltage at the supply but none across the LED), you hypothesize the LED might be reversed.</p>

<p><strong>Test</strong><br>
You use a multimeter to confirm there’s voltage at the power supply and also measure across the LED terminals to see if any voltage drop exists. You find no voltage across the LED supporting the idea it’s installed backward.</p>

<p><strong>Fix</strong><br>
You reverse the LED leads so that the anode is connected to the positive side and the cathode to the negative side. After flipping it, you then test again and the LED lights up, confirming that its fixed.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fricoe9921ru72dbo6ltf.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fricoe9921ru72dbo6ltf.jpg" alt="Broken LED Circuit" width="800" height="800"></a></p>

<h4>
  
  
  2. Broken Login Function (Software)
</h4>

<p><strong>Hypothesize</strong>:<br>
Users can’t log in, and logs show the authentication function is never reached. You hypothesize a code error, which could possibly be a typo or bad conditional, which is blocking execution before authentication.</p>

<p><strong>Test</strong><br>
You use the debugger to step through the login code line by line and also check input validation and branching conditions. You then discover a small typo in the validation check that prevents reaching the authentication routine.</p>

<p><strong>Fix</strong><br>
You correct the typo and redeploy or rerun the login code. Test again with valid credentials; users can now log in successfully, confirming the issue is fixed.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx8hnyyc0w4ryl163o572.jpeg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx8hnyyc0w4ryl163o572.jpeg" alt="Broken Login" width="736" height="1309"></a></p>

<p>In my opinion, in both cases, the key wasn’t luck but methodically testing assumptions and following the trail.</p>

<p>My takeaway is debugging isn’t just a technical activity. It’s a structured way of thinking. Whether you’re diagnosing circuits, code, or even everyday problems like a misbehaving appliance, debugging is a universal reasoning skill and not limited to a single domain.</p>

