---
Title: 🎃 How We Built a SaaS Platform in 3 Weeks" The story of Distrogent, our Kiroween Hackathon entry. Featuring Go, Kubernetes.
Description: 
Author: Pranavan Vadivelu Murali
Date: 2025-12-05T21:43:59.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The "Great" Idea 💡
</h2>

<p>You know that moment in a hackathon where you decide to build something way too complicated?</p>

<p>For the Kiroween Hackathon, my team looked at each other and said: "Let's not build a game. Let's build a Multi-Tenant SaaS Orchestration Platform."</p>

<p>Silence.</p>

<p>"And let's make it Halloween themed."</p>

<p>Thus, Distrogent was born. A tool that automates all the boring, scary infrastructure stuff (like provisioning databases and namespaces) so you don't have to.</p>

<h2>
  
  
  The Horror of Infrastructure 😱
</h2>

<p>If you've ever tried to build a B2B app, you know the pain.</p>

<p>Customer A wants their own database.<br>
Customer B wants to be in a specific region.<br>
You just want to sleep.<br>
We wanted to build a robot that handles all of that. You click a button, and poof—a new tenant environment appears, fully isolated and ready to go.</p>

<h2>
  
  
  How We Actually Pulled This Off 🛠️
</h2>

<p>We decided to use a "serious" tech stack: Go for the backend, Python for analysis, and Kubernetes Operators to manage the chaos.</p>

<p>But here's the thing: writing Kubernetes Operators from scratch is... let's just say it's a special kind of torture. You have to deal with CRDs, reconciliation loops, and about a million lines of boilerplate YAML.</p>

<h2>
  
  
  Enter Our Secret Weapon: Kiro 🚀
</h2>

<p>Look, I’m not saying we cheated, but using Kiro felt like having a senior engineer sitting next to us.</p>

<p>When we were drowning in Go structs and Kubernetes API definitions, Kiro was there to help us generate the boilerplate and make sense of the errors. It let us focus on the logic (and the spooky ghost animations) instead of fighting with syntax.</p>

<p>We built the core tenant-manager service in record time because we weren't getting stuck on the small stuff. It was honestly a vibe.</p>

<h2>
  
  
  The Spooky Stuff 👻
</h2>

<p>Since this is Kiroween, we couldn't just ship a boring gray dashboard.</p>

<p>We built a Dynamic Theme Engine that toggles the entire UI into "Witching Hour" mode.</p>

<p>Colors: Deep bruised purples and pumpkin oranges.<br>
Animations: We added little ghosts that float around your active pods.<br>
Easter Eggs: Try clicking the logo 3 times. Trust me.<br>
The Result<br>
In the end, we built a working, production-ready(ish) platform that can provision a full SaaS environment in under 30 seconds.</p>

<p>It was chaotic, it was stressful, but it was also the most fun we've had coding in a long time.</p>

<p>If you're building something complex, do yourself a favor: get a good team, get plenty of coffee, and definitely give Kiro a spin. It might just save your hackathon.</p>

<p>Happy Kiroween! 🎃🕸️</p>

