---
Title: Social media drama analizer / monitor
Description: 
Author: simon hryszko
Date: 2025-08-31T22:07:20.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>This is a submission for the <a href="https://dev.to/challenges/brightdata-n8n-2025-08-13">AI Agents Challenge powered by n8n and Bright Data</a></em></p>

<h2>
  
  
  What I Built
</h2>

<p>I built telegram bot what could help with alerting that your (or any) brand have troubles in social media (currently only on x)</p>

<h2>
  
  
  Demo
</h2>

<p>Feel free to chat with bot, but try not break it; it's not too stable unfortunetly: <a href="https://t.me/brand_crisis_checker_bot" rel="noopener noreferrer">https://t.me/brand_crisis_checker_bot</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo73k9r4t5melwt7qxuzk.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo73k9r4t5melwt7qxuzk.png" alt=" " width="586" height="517"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fif7rm3wsv6j1ea2okjyp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fif7rm3wsv6j1ea2okjyp.png" alt=" " width="612" height="532"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq0ni6dffg9lf5ebfb13b.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq0ni6dffg9lf5ebfb13b.png" alt=" " width="410" height="193"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa48o1mx1wl46qj6pnvg8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa48o1mx1wl46qj6pnvg8.png" alt=" " width="446" height="183"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcnt5aep818qjxmxwm2b9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcnt5aep818qjxmxwm2b9.png" alt=" " width="607" height="663"></a></p>

<h3>
  
  
  n8n Workflow
</h3>

<p>I splitted it into multiple workflows, so it might be hard to quickly import it into your instance. Feel free tho use following link to got all workflow: <a href="https://simonhryszko.app.n8n.cloud/webhook/6f0ee89f-9ee3-4ae7-9902-8f50e88c030d" rel="noopener noreferrer">https://simonhryszko.app.n8n.cloud/webhook/6f0ee89f-9ee3-4ae7-9902-8f50e88c030d</a></p>

<h2>
  
  
  Technical Implementation
</h2>

<p>n8n and brightdata of cource, mongodb (cloud) for some storage, no ai at all just to challange myself, telegram for 'easier' UI (at the end not good choice, due to poor n8n node integration ;/) and a lot of coffee ;P</p>

<h3>
  
  
  Bright Data Verified Node
</h3>

<p>Initialy wanted use multiple socialmedia and will add more like linkedin, instagram and so on. But for now use only get x posts</p>

<h2>
  
  
  Journey
</h2>

<p>I thought like companies pay fat money, for PR and recovery when someone post sth dump. That said if you can know it before your opponents, it's good. <br>
So my bot let you set your brands (profiles), set conditions (min likes / repost / followers, manage triggering words - I pretty proud of this idea) and then simple run check (could be done periodicly, but I'm scared I'll lose all my credits ;P)<br>
It's more PoC or MVP, TBH I was busy and did it durring last possible weekend, was fun and worth thought ;)</p>

<p>Great thanks to all my teammates: <a class="mentioned-user" href="https://dev.to/simonhryszko">@simonhryszko</a> super cool idea, <a class="mentioned-user" href="https://dev.to/simonhryszko">@simonhryszko</a> great job on n8n backend (especially with that limited time) and of cource thanks for <a class="mentioned-user" href="https://dev.to/simonhryszko">@simonhryszko</a> for writting this final post, making screenshots etc. Great job team;)</p>

<p>Thanks for this opportunity,<br>
Simon ;P</p>

