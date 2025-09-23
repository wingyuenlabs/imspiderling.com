---
Title: How I Built My First Web Tool as a Developer (and What I Learned Along the Way)
Description: 
Author: David Clarke
Date: 2025-09-23T21:55:03.000Z
Robots: noindex,nofollow
Template: index
---
<p>Like most beginner web developers, I spent a lot of time following tutorials and building “to-do apps” that never made it beyond my own computer. They were good practice, but I wanted to build something real that people could actually use.<br>
That’s how I ended up creating <a href="https://gamepadtester.co" rel="noopener noreferrer">gamepad tester</a> a small web tool that lets you check whether your game controller is working properly right in the browser.</p>
<h2>
  
  
  The Idea
</h2>

<p>The inspiration came from frustration. I plugged in my Xbox controller one day, and it just wasn’t behaving right in a game. Was it the controller, the USB cable, or the game itself?<br>
I thought: Wouldn’t it be great to have a quick online tool that tells me if all buttons and sticks are responding?<br>
Turns out the browser already has a neat feature for this: the HTML5 Gamepad API. That was enough to spark the project.</p>
<h2>
  
  
  Building the Tool
</h2>

<p>I didn’t overcomplicate it. Just HTML, CSS, and JavaScript.<br>
Here’s a simplified version of the core script that detects and displays button presses and stick movement:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
  &lt;meta charset="utf-8"&gt;
  &lt;title&gt;Gamepad Tester&lt;/title&gt;
  &lt;style&gt;
    body { font-family: sans-serif; }
    .button {
      display: inline-block;
      width: 40px; height: 40px;
      margin: 5px; text-align: center; line-height: 40px;
      border: 1px solid #333; border-radius: 8px;
    }
    .pressed { background: #4caf50; color: white; }
  &lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;h2&gt;Press any button on your controller to start&lt;/h2&gt;
  &lt;div id="buttons"&gt;&lt;/div&gt;
  &lt;script&gt;
    let controllers = {};

    function connectHandler(e) {
      addGamepad(e.gamepad);
    }

    function addGamepad(gamepad) {
      controllers[gamepad.index] = gamepad;
      const buttonsDiv = document.getElementById("buttons");
      buttonsDiv.innerHTML = "";
      gamepad.buttons.forEach((btn, i) =&gt; {
        const b = document.createElement("div");
        b.className = "button";
        b.innerText = i;
        buttonsDiv.appendChild(b);
      });
      requestAnimationFrame(updateStatus);
    }

    function updateStatus() {
      scanGamepads();
      for (let j in controllers) {
        const controller = controllers[j];
        const buttons = document.getElementById("buttons").children;
        controller.buttons.forEach((btn, i) =&gt; {
          if (btn.pressed) {
            buttons[i].classList.add("pressed");
          } else {
            buttons[i].classList.remove("pressed");
          }
        });
      }
      requestAnimationFrame(updateStatus);
    }

    function scanGamepads() {
      const gamepads = navigator.getGamepads();
      for (let i = 0; i &lt; gamepads.length; i++) {
        if (gamepads[i]) controllers[gamepads[i].index] = gamepads[i];
      }
    }

    window.addEventListener("gamepadconnected", connectHandler);
  &lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;

</code></pre>

</div>



<p>Open this in your browser, plug in a controller, and you will see the buttons light up when pressed. It’s a very stripped-down version of what I later turned into gamepadtest.com but the principle is the same.</p>

<h2>
  
  
  What I Learned
</h2>

<p>• Start small: You don’t need React, a backend, or a database to make something useful.<br>
• APIs are fun: Browsers can do more than you think. The Gamepad API was surprisingly powerful once I understood it.<br>
• Feedback is gold: Friends tested it and pointed out things I never thought of, like displaying joystick axes.<br>
• Shipping feels amazing: Even though it was a small project, publishing it online felt better than any tutorial app I had ever built.</p>

<p>Building my first web tool taught me that it’s not about making something huge or perfect. It’s about making something useful, no matter how small.<br>
If you want to try out the full version I built, check out <a href="https://gamepadtester.co" rel="noopener noreferrer">gamepad tester</a>. It might save you some frustration the next time your controller acts up.</p>

