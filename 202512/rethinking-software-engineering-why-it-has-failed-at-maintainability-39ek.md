---
Title: Rethinking Software Engineering: Why It Has Failed at Maintainability
Description: 
Author: Léo Antunes
Date: 2025-12-14T21:20:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>I have a strong opinion about the current state of software engineering for building maintainable applications: it has failed, and it's time to change. Not to throw everything away and start over, but to evolve. There are many good things in today's software engineering that we can preserve. However, before we can separate the wheat from the chaff, we must first understand why current approaches aren't working.</p>

<h2>
  
  
  The problem of Modern Software Engineering
</h2>

<p>Since the 2000s, several works have been highly influential in the field: Refactoring¹, Design Patterns², Domain-Driven Design³, Clean Code⁴, Clean Architecture⁵, SOLID Principles⁶, and architecture styles such as Hexagonal⁷ and Microservices⁸. This doesn't represent all of software engineering applied to maintainability, but it includes some of the most influential works in the field, giving us a good enough view of the current state. Despite this impressive intellectual foundation, something isn't working in practice.<br>
Some time ago, I was watching a talk entitled "Prioritizing Technical Debt as if Time &amp; Money Matters"⁹ by Adam Tornhill. He described a case where a company team he was consulting for complained about a specific service being hard to maintain. When he, as an external consultant, analyzed the codebase and compared it to other projects the team was working on, he couldn't identify any objective difference in code quality. However, the project the team complained about hadn't been developed by the team itself—it had been inherited after the original team was dismantled.<br>
This case reveals something crucial: the problem wasn't the codebase itself; rather, the codebase was expressing deeper socio-organizational problems. Building software professionally is fundamentally a social act. We construct software with other people, and our code inevitably reflects the dysfunction or health of those relationships, as Conway's Law¹⁰ has suggested since the sixties. This realization led me to reconsider how we've approached even our best-intentioned practices.</p>

<h2>
  
  
  The Gap Between Principles and Practice
</h2>

<p>Take Domain-Driven Design, for example. While it did great work introducing the concept of Ubiquitous Language¹¹—recognizing that a domain language emerges from social interactions, just like the language we speak—it stops short of addressing the practical challenges of implementation. We have Ubiquitous Language as a conceptual solution for communication issues between tech and business people, but we lack practical guidance on implementation: How can we convince business stakeholders to even participate in that construction? Does a regular individual contributor in an average-sized organization have the power to change how an entire team, or multiple teams, builds software? What are the limitations? What about the legacy non-ubiquitous language that has been written into the current codebase? What do we do with it? What are the risks? <br>
These aren't rhetorical complaints—they're genuine blockers that practicing engineers face daily. Yet our influential works rarely address them. The knowledge we have separates principles from their premises and from the real-world constraints that affect their application.<br>
The same problem appears in our most fundamental principles. If we take the clean code principle¹² of doing only "one thing" in a function, which is very similar to the Single Responsibility Principle¹³ from SOLID, we also hit challenges in applicability. What is a responsibility? What is a "thing"? What is a “reason to change”? I think not even the greatest philosophers could reach agreement on what that is. There's actually a dedicated field in philosophy called ontology¹⁴, which has been debating these questions for millennia. <br>
Let's take two different objects to illustrate why it's not really possible to achieve a "single" responsibility, thing or reason to change. Let's compare a desktop with a laptop from a maintenance perspective. When we have a broken keyboard, the desktop's physical properties are not affected by this change. We simply unplug the broken keyboard and replace it with a new one. The desktop has fewer responsibilities compared to a laptop, which is an object with more responsibilities as it has embedded keyboard, screen, etc. In the laptop, we need to open the entire device to fix the keyboard. This maintenance can be really challenging.<br>
But in the case of the desktop, is a keyboard a single thing? It's certainly a specialized responsibility compared to the laptop, but is it really single? For example, if the key "a" is broken, we may not need to replace the entire keyboard, but just this specific key, making each key in the keyboard a different thing. But we can push our lens further: is a key a single "thing"? Or is it composed of multiple smaller things? Maybe the problem with a key is just the switch, not the entire key. Now we know that the key is also composed of different things. We could continue zooming in until we reach the subatomic level. <br>
In software, our objects and modules are similar to this desktop/laptop example. We can zoom in software abstractions until we reach the bit level. Despite ontology, the Single Responsibility Principle itself doesn't answer a more fundamental question: what level of abstraction do we need, and which level of abstraction can we handle based on socio-organizational and other constraints of a specific business case?<br>
Notice that a similar issue occurs when we try to define how micro a Microservice should be. Remember, single is not the goal; micro is not the goal; rather, the goal is to increase or decrease the number of responsibilities, the size of the "thing" to achieve specific objectives. Reaching the single "thing" cannot be a principle or a goal in itself. Many experienced practitioners already know this, but theory lacks this formalization and can lead inexperienced engineers to mistakes.</p>

<h2>
  
  
  The Missing Foundation: Empirical Evidence
</h2>

<p>I recognize that software engineering is a recent and growing field compared to other fields of knowledge. I understand that we don't yet have many of the answers we need, and as professionals, we must make the best decisions we can with the guidance of our experience. However, I believe we could do more, and we could be more organized and meticulous about it. <br>
The lack of data about the effectiveness and challenges of software engineering for software maintainability principles and practices is an important part of the problem. We need more studies about clean code, SOLID, DDD, refactoring, and all other mainstream techniques and principles. I appreciate the idea of publishing new techniques that "worked for me." But we need to understand that they worked under certain premises and conditions. Until we have objective and empirical data about large-scale adoption, this is just a good hypothesis, nothing more. It works for them; it doesn't mean it will work for you. But we need to rationally understand why it would or wouldn't work for you.<br>
This lack of grounding in reality shows up in my own experience. In one decade working on enterprise web applications, from ERP and agriculture to logistics domain areas, I've never faced a situation where a delivery was delayed because I had to do a Shotgun Surgery¹⁵ change in the code. For sure, it's very annoying, slow, and fault-prone when we have to spread changes around multiple places to get things done, but it's not a blocker. Not at all. On the other hand, I have been stuck for days trying to contribute to a codebase (or part of a codebase) that nobody in the company knew how to work with because all the contributors were no longer with the company, forced to do a long, deep dive into the project internals to make a small contribution.<br>
I've seen projects delayed because a project leader tried DDD adoption despite the team's lack of readiness. In my opinion, the fault lies with the project leader—this is part of being a leader, being accountable—but at the same time, I feel bad blaming someone for trying a solution that claims to handle complex business problems.<br>
This brings me to a crucial realization: tech is not always the problem. In many situations, the problem is socio-organizational. I would argue that if you have a good team, most of the time the problem will be socio-organizational. At least, this is a good hypothesis about the problem.<br>
Historically, many techniques were developed through trial and error long before scientific understanding existed. Metallurgy and agriculture worked for millennia without deep scientific theory. But applying the scientific method to these techniques—asking why they work, testing variables systematically—transforms craft knowledge into scientific knowledge, which then enables better techniques. My goal with this essay is to provide a different view and a hypothesis on why software engineering is failing to provide the tools we need to create and maintain the software of our time. This essay alone is not going to solve the problem at all; at most, it will point in the correct direction for the industry and academia.<br>
To illustrate how ineffective 21st-century software engineering becomes when we ignore these socio-organizational aspects, I'll present a case study. Some of you might have faced something similar once or twice. Others, like me, have had to work with this type of project several times: the Legacy Inherited Repository.</p>

<h2>
  
  
  The Legacy Inherited Repository: A Case Study
</h2>

<p>A ~500k line-of-code repository/service, ten years old with thousands of relying customers, is assigned to your team to maintain and evolve. The previous team was dismantled, and the project was built by more than 150 different contributors over its lifetime. It was written in a modern language, but it uses older versions of core libraries and frameworks. The code has some incomplete modeling/data migrations and redesigns that were never concluded. The codebase has hard constraints everywhere: from unique items in a table to a max number of items per page in a endpoint, from a max number of committable rows in a transaction to cronjobs that clean up weird tables in the database. You have no idea why those specific constraints are there. Most of the stakeholders have a vague idea of how the system works. When they joined the company, it was already like this.<br>
There's one older stakeholder—let's call him Mike—who has been there since the beginning. He sometimes remembers why some features work the way they work, and sometimes he doesn't. Complicating matters, Mike is assigned to another project at the moment, so while you can chat with him, he's not part of your team. The team starts to ship the first features of the project, and the experience is terrible. It's frustrating. Deliveries start to be delayed. But the team has items in the backlog: a new feature needs to be delivered for a new customer. The deadline isn't short, but everyone is struggling to work with it. The parts of the project that the team has already worked with are full of functions and classes that the team is struggling to understand. Some of them are small (dozens of lines of code); others are big (hundreds of lines of code). One of the team members did a small assessment of other untouched parts of code that might be changed in the future, and the traits look the same.<br>
Why Traditional Approaches Fail This Scenario<br>
What would you do? Are you going to follow the existing literature? Would you split the biggest functions into "smaller, cohesive" ones like clean code suggests? Would you rename variables and functions to "meaningful names"? Is making it more aesthetic the solution for this case?<br>
What about a full remodeling of the codebase to adopt a Ubiquitous Language? It solves the problem of understanding and communicating about a domain. The project also has consistency issues (especially in the most important features) which very often interrupt the team with support tickets—should we refactor the application to encapsulate business transactions within repositories¹⁶ and aggregates¹⁷? Would you propagate eventual consistency between different parts of the project with domain events¹⁸?<br>
The honest answer? None of these approaches is going to solve the problem. A study¹⁹ (one of the few with valuable empirical insights that I could find) shows that the main challenges in DDD adoption include managing complex domain models, integration with legacy systems, communication gaps between domain experts and developers, and onboarding challenges and lack of DDD expertise. Suppose half of the team has no previous experience with DDD and the team has no strong support from domain experts. What is going to happen if the team decides that a major DDD refactor/redesign is needed to increase the project's maintainability and to fix the consistency issues?<br>
Also consider this other constraint: time is not infinite. Suppose you have 20% of the team's budget dedicated to making maintenance improvements over the next year. Do you really think that applying these software engineering principles will be effective in saving the project? Is it really possible to transform this repository into something "nice to contribute to" with a 20% tech debt budget (which, in my opinion, is very generous in this low market)? And remember, the remaining 80% will be used to add even more features (and complexity!) to the project.</p>

<h2>
  
  
  The Agile Blind Spot
</h2>

<p>Before discussing possible solutions, I want to briefly address some of the agile principles²⁰—because I think they suffer from the same applicability issues as the DDD and clean code principles and practices I've discussed. Let's take "Business people and developers must work together daily throughout the project," "The best architectures, requirements, and designs emerge from self-organizing teams," or "Individuals and interactions over processes and tools." These sound great as principles, or values, but they lack strategy in terms of adoption, which makes them empty ideals.<br><br>
The critical question is: How do we apply these principles in the first place? As far as I know, software engineers aren't in charge of deciding how an enterprise is run, nor do they have the power to establish or abolish processes within organizations. Sometimes software engineers can influence an organization, sometimes they can't. These principles assume a level of influence that most software engineers simply don't have. That's a significant blind spot. I won't go deep into the agile thing, and, to be honest, history makes it clear that it failed as well.</p>

<h2>
  
  
  The Shift Ahead
</h2>

<p>We need to elevate software engineering from a craft to a science, or at least use the scientific method to evaluate the effectiveness of our techniques and principles for software maintainability.<br>
Considering socio-organizational aspects is a very good hypothesis for the moment and a good place to start. The Software Design X-Rays: Fix Technical Debt with Behavioral Code Analysis²¹ book by Adam Tornhill (the same person from the previous keynote I mentioned) is a good starting point as it provides an objective and measurable way of quantifying and prioritizing technical debt using git history as the data source for considering social aspects such as code ownership. Behavioral code analysis²² points in the correct direction for the software engineering field.<br>
We also need studies about 21st-century mainstream software engineering works, we need studies about behavioral code analysis, and we need studies about the effectiveness of considering socio-organizational aspects for software maintainability as well. Everything must be constantly and extensively evaluated, measured, and improved.<br>
As software engineers, we need to shift the way we study and apply new techniques and principles. We need to pore over all the existing studies and use this knowledge to base our decisions on whether to go in one direction or another. This will help us separate personal preferences and aesthetics from technique effectiveness. I don't know if I'm being idealistic and contradicting myself based on the agile critiques I made. But I do think we can improve the field. In the meantime, while we don't have more studies, sharing our experiences in a rational and meticulous way can help us spot the challenges and wins of adopting such techniques. Shifting our minds on how we evolve software engineering for software maintainability can influence the next generation of researchers to conduct studies in this field.<br>
This change, if it occurs, will take years, of course, but I think it's possible to go in this direction. As a software engineering enthusiast, this new view of the field brought me hope and motivated me to look into the challenges we currently have in a different way—in a way where we don't need to pay all the debt, but we can improve it where it matters.</p>

<h2>
  
  
  References
</h2>

<ol>
<li>Fowler, M. (1999). Refactoring: Improving the Design of Existing Code.</li>
<li>Gamma, E., Helm, R., Johnson, R., &amp; Vlissides, J. (1994). Design Patterns: Elements of Reusable Object-Oriented Software.</li>
<li>Evans, E. (2003). Domain-Driven Design: Tackling Complexity in the Heart of Software.</li>
<li>Martin, R. C. (2008). Clean Code: A Handbook of Agile Software Craftsmanship.</li>
<li>Martin, R. C. (2017). Clean Architecture: A Craftsman's Guide to Software Structure and Design.</li>
<li>Martin, R. C. (2002). Agile Software Development, Principles, Patterns, and Practices. Chapters 8-13.</li>
<li>Cockburn, A. (2005). Hexagonal Architecture. <a href="https://alistair.cockburn.us/hexagonal-architecture/" rel="noopener noreferrer">https://alistair.cockburn.us/hexagonal-architecture/</a>
</li>
<li>Richardson, C. Microservices.io - What are microservices? <a href="https://microservices.io" rel="noopener noreferrer">https://microservices.io</a>
</li>
<li>Tornhill, A. Prioritizing Technical Debt as if Time &amp; Money Matters. Conference talk.</li>
<li>Conway, M. E. (1968). How Do Committees Invent? Datamation, 14(4), 28-31.</li>
<li>Evans, E. (2003). Domain-Driven Design, Chapter 2: Ubiquitous Language.</li>
<li>Martin, R. C. (2008). Clean Code, Chapter 3: Functions, pp. 35-36.</li>
<li>Martin, R. C. (2002). Agile Software Development, Principles, Patterns, and Practices. Chapter 8: The Single Responsibility Principle.</li>
<li>Corazzon, R. Ontology: Theory and History. <a href="https://www.ontology.co" rel="noopener noreferrer">https://www.ontology.co</a>
</li>
<li>Fowler, M. (1999). Refactoring: Improving the Design of Existing Code. Chapter 3, pp. 66.</li>
<li>Evans, E. (2003). Domain-Driven Design, Chapter 6: Repositories.</li>
<li>Evans, E. (2003). Domain-Driven Design, Chapter 6: Aggregates.</li>
<li>Evans, E. (2003). Domain-Driven Design, Chapter 8: Domain Events.</li>
<li>Özkan, O., Babur, Ö., &amp; van den Brand, M. (2023). Domain-Driven Design in Software Development: A Systematic Literature Review on Implementation, Challenges, and Effectiveness. <a href="https://arxiv.org/abs/2310.01905" rel="noopener noreferrer">https://arxiv.org/abs/2310.01905</a>
</li>
<li>Beck, K., et al. (2001). Manifesto for Agile Software Development.</li>
<li>Tornhill, A. (2018). Software Design X-Rays: Fix Technical Debt with Behavioral Code Analysis.</li>
<li>Tornhill, A. (2018). Software Design X-Rays: Fix Technical Debt with Behavioral Code Analysis. Chapter 2.</li>
</ol>

