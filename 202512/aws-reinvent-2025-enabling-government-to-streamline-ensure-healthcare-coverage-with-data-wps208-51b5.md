---
Title: AWS re:Invent 2025 - Enabling government to streamline & ensure healthcare coverage with data-WPS208
Description: 
Author: Kazuya
Date: 2025-12-05T21:38:53.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 - Enabling government to streamline &amp; ensure healthcare coverage with data-WPS208</strong></p>

<blockquote>
<p>In this video, Covered California presents two data-driven customer retention strategies using AWS services. First, they replaced credit-based identity verification with Socure's solution, reducing the 33% dropout rate to just 2% while detecting 5% fraudulent transactions. Second, they implemented a personalized retention campaign using risk scoring models built on Snowflake and R, creating thousands of AI-generated customized messages for five audience segments. This approach integrated with Amazon Connect and Salesforce to deliver personalized IVR messages and agent whispers. Results showed 5x increase in phone calls, 125% more website traffic, and 3-13x improvement in retention rates within two weeks of launch.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/QQHbbopPdRY">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcfnf13oebw613s9regjr.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<h3>
  
  
  Introduction: Covered California's Data-Driven Customer Retention Strategies
</h3>

<p>Hi everybody, welcome to WPS 208. This is Covered California with AWS, and today we're going to talk about the data-driven customer retention strategies that Covered California has put in place to help with customer retention. I'm Chris Carter, a senior sales specialist with AWS. I focus on Amazon Connect for public sector accounts in the West, and I'm joined today by Glenn and Kevin.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=30" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjsors8w41hh3un6psxi7.jpg" alt="Thumbnail 30" width="800" height="450"></a></p>

<p>Would you like to introduce yourself? Good afternoon. I'm Kevin Cornish, the Chief Information Officer at Covered California. I run our technology that supports our mission that we're going to talk about this afternoon. Hi everyone, I'm Glenn Oyoung, the Chief Marketing Officer at Covered California. You could probably tell because I'm the one that took the selfie. I want to remember this. How many times do you get to talk to people with a bunch of yellow headsets? Thank you for joining us.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=60" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2vplj79xw94ehunia2u5.jpg" alt="Thumbnail 60" width="800" height="450"></a></p>

<p>So real briefly, Glenn will do a quick introduction to Covered California. For those that aren't familiar with Covered California, you'll learn a little bit about who they are, some of the market forces and changes that have happened recently that are driving concern amongst the residents and constituents of the state of California in obtaining health insurance. We'll discuss some different initiatives that Covered California has taken to drive better retention and improve retention for their customers, and marketing outreach. Then we'll wrap it up with Q&amp;A. Q&amp;A is not going to be live today, so when we're done with the session, you can meet us up in the front or in the back, and we can take live Q&amp;A when we're done. So with that, I'm going to hand it over to Glenn.</p>

<h3>
  
  
  Covered California's Mission: Connecting People to Affordable, High-Quality Healthcare
</h3>

<p>Thanks so much, Chris. Hi again and thank you so much for taking time out of your day to come learn a little bit about Covered California, what we do, retention, and ultimately how our technology partners help us facilitate that. So before we get into it, if you're not familiar with Covered California, we are the state healthcare exchange for the state of California. Here on the slide you can see many of the plans that we connect people to, but we're not just connecting people to plans. We do much more than that. I just want to level set that many states do have healthcare exchanges, and for those that don't, they direct their folks to healthcare.gov.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=140" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj43jn6v0gt4uv4q63f9z.jpg" alt="Thumbnail 140" width="800" height="450"></a></p>

<p>What is our mission? Our mission is really fourfold. As I just alluded to, we do connect people to plans, and that's core to our mission. We want people to get covered. That's in our name. I pause like I just realized that, but I've known that. What does that really mean? It's our job and our mission to give people affordable choices. Those choices are not just affordable, but the plans and the coverage that we connect people to are high quality. We care very deeply about health equity, and what that means is closing the health equity gaps that do exist in our country and around the world. We have the great ability to work with these plans and work with the plans that we select and work with every day to try and make sure that we're closing gaps for the communities that we serve.</p>

<p>Speaking of service, across the board whether it's marketing, IT, or our service center, you name it, every part of Covered California is committed to the mission. It's truly and deeply ingrained in our hearts, and that shows up in exceptional service. I do want to pause and share a quick story. On the very first day of open enrollment, at about one a.m., Kevin and I got a text from one of our partners here, Brian, telling us about the very first enrollment in the state of California. Emojis were shared, and I say that to say that I know that everyone on the stage and the teams that we work with, including the partners that we choose to work with, we choose to work with people like Brian and his team because that mission is paramount to us.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=160" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdpwrghvmlsy5uefbqxye.jpg" alt="Thumbnail 160" width="800" height="450"></a></p>

<p>I hear we have an hour, so I'm going to tell you another story. It's going to get a little more personal, but I want you to think about what that means in the state of California. My parents are immigrants. They came to this country with nothing in pursuit of the American dream. My father was an engineer, and I think he'd be very proud that I get to stand in front of STEM people who are much smarter than me in the realm of STEM and talk about our mission. My father unfortunately passed away of cancer. He worked his way up to be a successful engineer. He could have picked any plan he wanted when he retired. He did not pick any of the plans I showed you. He picked a plan where most of the providers speak Mandarin. It's a smaller plan, and ultimately one of the great traumatic things in my life is battling with his providers and battling with his plan, and battling with anyone I could talk to to make the last few weeks of his life more comfortable.</p>

<p>So health equity is something I say a lot, but for me personally, that's what I think about when I think about health equity. I do think about my dad. I do think about myself. But I think about the people unlike my dad or myself who don't speak English, didn't go to college, didn't have a PhD, don't have a son that can even try to do that battle that is the complexity of our health care system.</p>

<p>I share that because oftentimes we talk about these things—complexity, affordability, equity—and for me it's very personal. I know that everyone at Covered California has some version of this story that propels us to work as hard as we do. So thank you for letting me share that with you because it's not about me or my dad, it's about Californians, it's about the people we serve. That's what makes us work super hard to try and do whatever it is that we can do to help our fellow Californians. I can tell you from talking to people in other states and state health care exchanges the feeling is very pervasive. That's what we mean by our mission.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6xktwpdovgsr3giv6d9.jpg" alt="Thumbnail 390" width="800" height="450"></a></p>

<h3>
  
  
  Serving One in Six Californians: Brand Platform and Market Challenges
</h3>

<p>We've been really good at what we do.  One in six Californians have or will have been a Covered California member in their lifetime, whether they ever thought they would be or not. And one in three, if you add in our partners at Medi-Cal, have been served by Medi-Cal or Covered California. That's huge. Just think about that. I will say that there are people in this convention right now who do not think they will ever need Covered California. But one in six of them might.</p>

<p>It's our job to make sure, especially in that type of environment, that they think I'll never need that so I don't really need to connect with that brand. It's my job in marketing and many of our peers to make sure that I hope that's true. I hope you always have employer-sponsored insurance, but if you don't, if you're laid off, I hope the first thing you think about is Covered California. That's the job.</p>

<p>We have been very successful in reducing our uninsured rate from 17% to 6%, meaning 94% of eligible Californians prior to this moment in time have found insurance either through Covered California and Medi-Cal or their employer. Last year I think we set a record for almost two million enrolled, which is very, very—we're proud of that. And at the same time, this year is a different year. So what do I mean by that? Our government was shut down and I'll get into that, but if you're tracking, then you know about the subsidies that are planning to expire.</p>

<p>There's a lot of federal policy changes and broader landscape changes in health care in general that have nothing to do with decisions that we've made but make it even more complex and more scary for Californians. And so our answer to that, starting from a year ago, is really doubling down on telling the story of who we are from upper funnel. I'm a marketer, so go with me, but upper funnel, right? And you guys know building awareness, building trust and working with partners like Kevin and his team and many of you in the front row to take that trust and awareness and turn that into action. That's where technology really comes into play.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=520" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuica8yw1m1ywfenvnfm7.jpg" alt="Thumbnail 520" width="800" height="450"></a></p>

<p>So I want to show you a bit before we get too deep into the technology aspect our new brand platform, which is for the love of Californians.  And this is our declaration, our first spot, which is called declaration because it is indeed a declaration of what our intent is here at Covered California. We love California, but we love Californians even more. Sixty countries in one state and the music of two hundred languages. At Covered California, we believe every single Californian deserves healthcare that works. So let us cut through the confusion. We're not a health insurance company. We're in it for you, because we are you, for the love of Californians. Covered California.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqqmio7z2mdif8sp44c1r.jpg" alt="Thumbnail 560" width="800" height="450"></a></p>

<p>In other places I do this, people clap, but I know we can't. So just, if you liked it, give me a sign, or you can clap. We'll break some rules. Thank you. I wasn't asking for applause. I just want to know if it was done. But again, joking aside, our brand platform, our creative campaign, our tagline, which you will see for many, many years, I hope for maybe even a decade, I hope, is for the love of Californians. That is going to permeate everything we do from an outreach perspective.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=600" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbnmcfn7alohtzda7m9lh.jpg" alt="Thumbnail 600" width="800" height="450"></a></p>

<p>And it's great because although it's new, it's not really new. We have always loved Californians and we're just saying it out loud so people understand who we are and why we do what we do. So as we think about that, we want to start showing you beyond the big  brand and the pivot.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9k9u90kqgr8e9y5u4ahe.jpg" alt="Thumbnail 610" width="800" height="450"></a></p>

<h3>
  
  
  Three Obsessions: Mission, Consumer Experience, and Security
</h3>

<p>How does that show up in this moment where we have to focus on some other aspects that we typically  haven't had to focus on before? Thank you, Glenn. At Covered California, we're obsessed by three things. The first is our mission. Glenn talked a little bit about our mission. It's to make sure that every California resident has access to affordable, high quality healthcare. He talked about some numbers when Covered California came online about 12 years ago, more than 17% of our state's population had no access to insurance. That means when their kid got sick, they ran to the emergency room. That means they were using very expensive healthcare options in pretty inconvenient ways, and they were putting themselves and their families at risk for immense financial catastrophes.</p>

<p>Any health scare could take their family's car off the road. So number one is affinity for our mission. Number two, our second obsession is consumer experience. We're a state agency of the state of California. When I came to Covered California, I was intrigued by people saying that consumer experience is our North Star. It's something we're really obsessive about. I thought the government doesn't care about me. They care about me when they want my taxes. They care about me when I have to pay a traffic ticket, but otherwise they don't care about me too much. Covered California really cares about our consumers and our citizens, and that's the customer obsession we're going to talk about.</p>

<p>We'll talk a little bit about the consumer journey and how some of the technology we've deployed recently has really improved that consumer journey and the customer's experience. We'll talk about friction, which is the enemy of a consumer's experience, and what we do to reduce friction. The third thing we're obsessed with is information security and privacy. Glenn talked about one in six Californians having been through Covered California. We are also the front door. Our platform CalHERS is the front door for eligibility determination for the Medicaid system in California, which is called Medi-Cal. So every year about 17 million members of our population—that's close to half of the 39 million people in California—come through our system and either go to a Medi-Cal solution or come to Covered California.</p>

<p>That means over time we've collected personally identifiable information and other forms of health information on nearly 25 million Californians. So we're obsessed about information privacy and security. We want to safeguard those consumers' data. I'm going to talk about a challenge that we faced in the last couple of years that we recognized in that consumer journey. Remember, for the consumer journey, friction is a bad thing. We don't want it. We want to push people through a workflow that gets them to a conclusion where they're enrolled in a health plan as easily, smoothly, and quickly as possible.</p>

<p>We found that the way we were doing eligibility determination on this platform called CalHERS was giving anxiety to about 33% of our population. They were literally exiting the workflow at that point. We did some digging and started to determine that this eligibility determination process we used, which was relying on credit checking, was not something that our demographics liked. A large number of them don't have great credit scores or maybe even credit scores at all. Also, a large number of them just don't want to share that kind of information with the government or even have that conversation.</p>

<p>We found that a third of our consumers would exit the workflow. They would come to our solution, our self-serve platform. They would do an eligibility determination and determine that they were eligible for some form of subsidy which would reduce the cost of that insurance plan. They would begin to do what we call shopping and comparing health plan options between some of the health plans that Glenn showed before. And then when it came time for us to determine that you are really who you say you are so we know that eligibility determination is valid, we would go to this credit check process and they'd say I want out of here. I don't like this. I'm uncomfortable. I leave, and for the most part we never saw those consumers again.</p>

<h3>
  
  
  Case Study One: Eliminating Friction with Socure's Identity Verification Solution
</h3>

<p>A third of our population was leaving before we scored a goal. We were not able to satisfy our mission with them. So we began to look in the marketplace for a solution. We've been on the Amazon Cloud for about four years. We've had a great journey from an on-premises solution when I joined the company five years ago to a highly tuned microservices-based cloud native solution taking advantage of the cloud platform. We initially started to look at what other solutions are out there that would plug into this AWS ecosystem and how could we take advantage of someone we would trust because they're in our system.</p>

<p>Remember that third obsession with privacy and security that we really need to make sure this was very sensitive information and a sensitive workflow. Who could do that? So we found a solution from Socure.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=940" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F43glgvwt44rc0pf98kr8.jpg" alt="Thumbnail 940" width="800" height="450"></a></p>

<p>We began to prototype the concept of a credit-based risk score. It solved two problems for us. One was the exit ramp where 33% of our clients were leaving and never coming back. The other was the significant focus, both inside our industry and from the federal government, on fraud, waste, and abuse. We were looking for a solution that would address potential fraud, waste, and abuse issues that we may have been experiencing on our system and perhaps did not even know about. So we selected Socure and deployed it. </p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1fks83lwtqaitj2ydqtl.jpg" alt="Thumbnail 960" width="800" height="450"></a></p>

<p>Let me talk about some of the things this solution accomplished. We took a very different approach to validating consumers. The long story short is that we were able to achieve remarkable results.  We deployed it seamlessly in our system and worked with our solution, Socure, with no noticeable difference for consumers. We had very effective fraud deterrence components. Some of the statistics are striking. Right off the bat, we found that about 5% of the transactions coming into the system were fraudulent, and the system was able to identify those consumers and remove them from the workflow.</p>

<p>The next significant finding was the passive flow of consumers through our solution. Today, we are operating with about 85% of our consumers passing through passively, meaning they do not have to do anything. We know enough about their biometrics and background information from Socure's approach that we are validating them without any additional action on their behalf. About 15% of our consumers do not pass that initial step, and we move them to another form of validation, which Chris will walk through in a moment. But essentially, in the end, 98% of consumers are getting through this critical step of identity validation.</p>

<p>We are eliminating the fraudsters, roughly the 5% of people attempting to do something improper. And if you recall the problem we had initially that brought us here, the 33% of our consumers leaving and never coming back without getting to an insurance plan, that is gone. We are passing 98% of them through. They are moving down the highway, going through the shop and compare step, finding one of those health plans that is affordable for their family, ensuring they are getting the maximum subsidy to reduce the cost of that plan, and signing up for health insurance. That is really important to us.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1100" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fduxzjz3pxph9sv0ea3sl.jpg" alt="Thumbnail 1100" width="800" height="450"></a></p>

<p>We remain concerned that 6% of Californians who probably have access to a solution through us are not taking advantage of it. This approach gets us back focused on consumer experience. How do we ensure that we get more Californians insured? How do we get to that basic human right, which is that I have a human right to be healthy? I have a human right to access this amazing healthcare system in the United States without being worried about catastrophic impact on my family's finances. This is really important to us, going back to our first obsession, which is our mission. I am going to give it to Chris. Chris is going to talk about that step where passive verification does not work, the 15% that have to go to the next step. </p>

<h3>
  
  
  Document Verification and AWS Services: Achieving 98% Validation Success
</h3>

<p>Thank you, Kevin. As Kevin mentioned, about 84% of the people going through the system will go through what is called passive verification. With passive verification, users will input personally identifiable information such as name, address, and other information. Through a process that Socure provides, they will do device analytics and a number of other things to process all of that information passively to make a determination of that person's identity. Again, 84% of the people will pass through that without even knowing they have gone through this process. It is great right off the bat.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fug0cto20l1wk5g69x6ao.jpg" alt="Thumbnail 1170" width="800" height="450"></a></p>

<p>But for the 16% that do not make it through that process, they go to what is called DocV, or document verification. Socure provides a mobile platform or experience where you can provide a government-issued ID and biometrics, in this case a picture.  Socure will use that to provide additional mechanisms to validate that the person is who they say they are. With the photo you see here, it is comparing that photo and liveliness, making sure your eyes are open or asking you to look to the left or look to the right. They are even checking things like whether your shoulder is up, because your shoulder would be up if you are holding a phone in front of you to take a picture. These are little things, but the system is checking liveliness and checking your document.</p>

<p>The process validates your document using AWS services, which I'll discuss on the next slide. It extracts all the information from those documents and compares it to the passive information that was collected previously. As Kevin mentioned, the vast majority of people between these two methods are now able to get through this process and not fall out of the system like they were before.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1240" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fszqedc6jbp1yyq05gara.jpg" alt="Thumbnail 1240" width="800" height="450"></a></p>

<p>This is what it looks like in practice.  On the right-hand side of the slide are the AWS services that Socure is using. It's an advanced machine learning model with a lot of proprietary technology behind the scenes to provide their service. Some of the AWS services forming the basis of that include Amazon Textract, which is used to pull information off documents, such as text from IDs or other scanned documents.</p>

<p>They're using Amazon Rekognition for photo recognition and liveliness detection. You can say things like turn your head to the right or turn your head to the left and open your mouth to detect that you're actually looking at a live person and not a picture of a person that might be on an ID. They're using services like SageMaker to help develop their ML models. It's their service that they're providing, but they're able to build quickly using SageMaker and adapt and continue to develop the service and make improvements.</p>

<p>They're using Amazon Pinpoint for messaging, including SMS and email for end-user messaging. Amazon Pinpoint is moving toward end-user messaging, and Socure is moving to that as well, basing their service on Amazon Pinpoint. All of this is co-located in the same region within AWS as the CalHEERS app itself, so it's very low latency and secure, allowing the two systems to interact together in a very efficient and secure manner.</p>

<h3>
  
  
  Case Study Two: Strategic Shift to Retention in Response to Federal Policy Changes
</h3>

<p>With that, we'll move on to case study number two. Thanks so much. We're going to now turn our attention to our second case study, and you've heard me mention retention. That's not necessarily something we've ever been really focused on that much at Covered California. It's always been a part, and I think if you were to step back and say each division, each department is doing something retention-related. But really, if you think about Covered California and state healthcare exchanges, we are a transitory experience for most of our members, meaning our average tenure is 14 months.</p>

<p>Prior to Covered California, I worked for one of those health plans. The average tenure ideally is as long as humanly possible. For us, we're meant to be here in a moment in time for most people where they need us, and then ideally in this country, the ideal path forward is employer-sponsored healthcare. You get it from your employer, or you start your own business or something like that. So we haven't really had to focus on that. That wasn't really core. But we do need to focus on that very much for all the reasons I'm about to get into, and that created another level of urgency for us across the organization to really think about what is our approach and strategy to retention, why does it matter, and how does technology and working together help us achieve that.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1450" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb605p67chbjwosqsj4z5.jpg" alt="Thumbnail 1450" width="800" height="450"></a></p>

<p>It matters because we don't want people to go uninsured. I mentioned a couple of headwinds, and I'm just going to lightly touch on it.  It's clear from our focus groups and from whether you talk about federal, state, or city—it doesn't matter. We're in the trust business. If you don't trust Covered California, if you read the news and it makes you want to close down your app, it's very hard for us to talk to you about healthcare or really anything. We see that in not only our research but very widely published research, and it's been that way before even this year.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1490" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0t30n3fa337bfe0f83fs.jpg" alt="Thumbnail 1490" width="800" height="450"></a></p>

<p>This healthcare system in general, because of the complexity, because of surprise bills, because of access problems even dating back to the pandemic, trust has really been a struggle for everyone in this space.  Specific to Covered California, we've typically enjoyed very high brand awareness, but when you double-click into beyond have you heard of Covered California, what do you know about Covered California, what do you feel about Covered California, the trend has been going down.</p>

<p>And this is even before what's happening now with our government's stalemate around federal tax subsidies. As a marketer, this is job security. No, I'm just kidding. As a marketer, this is the call to action, and this is the urgent need for us to do something to make sure that people take the time to take notice of who we are so that we then have the right to pull alongside them and tell them all the ways that we can support them.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F13l6xmvsp5mahm3ca60p.jpg" alt="Thumbnail 1540" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1570" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmai9w86vdvqjpmzr3wi7.jpg" alt="Thumbnail 1570" width="800" height="450"></a></p>

<p>I've mentioned this a couple of times, but without getting too technical,  there are tax subsidies, federal subsidies that are set to expire at the end of this year, and the end of this year is quickly approaching. Even today, depending on what day you open up your news apps, are they going to get extended? Are they not going to get extended? Are they extended for two years? Should we even have the ACA? This is the confusion that I'm talking about. The end result in human context is it creates a wave  of emotions that ultimately for consumers means that many of them will delay any kind of action. Many of them will say it's too confusing. I'm just going to wait this out.</p>

<p>The danger is that the 6% unemployment rate skyrockets. Our own internal projections say that potentially if these subsidies expire, something like 13% of our members will unfortunately make that tough choice to not renew. It wouldn't be because they don't know the value, and it wouldn't be because they haven't had access to high quality affordable plans. For many of them, it comes down to their health insurance bills doubling and having to make the choice between paying for food—literally, I'll say it again, paying for food, paying for gas to get to work, paying for rent, or this idea of healthcare insurance.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1650" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F13cvqp9p7nhfednlelld.jpg" alt="Thumbnail 1650" width="800" height="450"></a></p>

<p>Something that I believe after many years of marketing is that most humans are bad at managing risk. Look at my haircut as a great example. Don't look from the back. When you layer that on with this complexity, it goes to this moment in time where all of us at Covered California really have to meet this moment. I summarize the headwinds by saying that we did 21 focus groups across the state,  so this is not just an opinion. This is hearing from real people who really are struggling with this moment, and health care is just one part of their lives. I mentioned the other areas that we all have seen the effects of inflation and so on.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1690" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foln6yvdg4beq6nqnkpgz.jpg" alt="Thumbnail 1690" width="800" height="450"></a></p>

<p>The strategic shift to focus on retention was really paramount. Luckily for us as a team, we've been thinking about this for at least a year, and we've been working as a team before this month for a year to figure out what we can do and then stand up some things to lean in and help. What are those things? Ultimately, I would point your attention to the bottom. Through all the wonders of technology  and our partnerships and our wonderful team, many of whom are sitting right here, we are mining our own data to ultimately understand our customers and to profile the risk of leaving. This is a retention model, a risk profiling that then feeds into segmentation and then on the left creates more personalized messages, more nuanced messages.</p>

<p>I'm smiling because with my own marketing team, I think we all know I will say, well, what do we expect from Amazon, right? Not AWS but Amazon. What is that customer experience expectation that we have in for-profit brands in buying a car, in buying shoes, in buying an airplane ticket? Those are the same fluid expectations that transfer to us. It shouldn't be this hard. People do expect us to know them and to manifest that knowledge into personalized outreach, which we historically haven't done as much as we could have, but we are now.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fttgkepibtqxrd2loa4q5.jpg" alt="Thumbnail 1790" width="800" height="450"></a></p>

<h3>
  
  
  Personalized Outreach at Scale: Risk Scoring, AI-Generated Messages, and Measurable Impact
</h3>

<p>How does that show up when you're talking to our agents? How prepared are our agents to also treat you in a way that's more tailored to your situation? This is really the unlock that this work with technology and with AWS and our partners have done for us. I'm going to hand it over to do the triple click into that. Thanks. All right, so how does that work? What does that look like, these personalized messages, and how do you develop those?  The team at Deloitte along with Covered California started working backwards from previous data from previous enrollees. They built what's considered a risk score, and a risk score is really just the modeled probability that a member</p>

<p>is going to renew or not. A number of factors go into that risk score, including things like a premium increase or the way that they earn income today—whether they have a large salary or are they a gig worker. Their age matters too—are they a young person, maybe very healthy, or are they older? Are they taking care of dependents or are they just taking care of themselves? There are a lot of factors that go into this, and when you look at past data, you can see how premium increases might have impacted certain audience segments and you can make predictions in the future about how these changes that Glenn outlined might impact the members in 2026.</p>

<p>Using R on Snowflake, the Deloitte team modeled at scale across hundreds of thousands, even millions of people to develop risk scores and created audience segments. One example of an audience segment is a young adult aged 26 to 35 who lives in an urban California city. They're a gig worker or an hourly worker, and their primary concern is going to be the cost of insurance. A $10 to $20 per month increase would really impact them, so they're primarily concerned about what that cost is.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1910" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft062boi65oijn6wo2om4.jpg" alt="Thumbnail 1910" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcq48u1lmhu5fhi8edlyj.jpg" alt="Thumbnail 1930" width="800" height="450"></a></p>

<p>Another audience segment might be an older couple, let's say aged 45 to 55, supporting dependents.  They might have chronic health issues in the family. They're obviously going to be concerned about price increases, but for them the most important thing is to have continued insurance. That's the most important thing. I think all of us here have realized that you don't really care about your health until it goes bad and you wake up one day with your back out, and then all of a sudden health matters. They're already experiencing that, so health insurance is very important to them.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=1990" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8ghzalstrwsfz2u5clf8.jpg" alt="Thumbnail 1990" width="800" height="450"></a></p>

<p>With this, the team developed these five audience segments initially based primarily on what the impact of premium increases will have on each of these different audience segments that were modeled. Based on these risk segments,  using AI, the team created a number of customized messages—and when I say a number, I mean thousands of customized messages, SMS messages, email messages, messages that are targeted to those different audience segments. Using AI, specifically large language models, we're able to run simulations of hundreds of thousands of simulations using these messages against these audience segments. We can look at things like click rates or open rates and model these in simulation rather than testing these on actual people.</p>

<p>The power of the cloud here is that you can run these simulations at massive scale in very short periods of time at extremely low cost. Ultimately, what you get out of it are the perfect message for each channel like SMS or email for each audience segment—the ones that person is going to open and click and that will resonate with them the best. To Glenn's point, you want that kind of Amazon experience. You want the company to know you. This is that more personalized experience. Then I'll turn it back over to Glenn for what that actually looked like in practice.</p>

<p>So what I'm going to show you now, we started with an ad. Now we're talking about communications, and this is when we started. I was looking at a real-time NPS score. We talked, I showed you the ad, and here we are talking about retention and member communications. On the left is information we sent out starting from July, actually July all the way through September to let people know. I would draw your attention to the chart on the bottom. This is personalized to each household.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fliuqf1ww6ryfbijkodok.jpg" alt="Thumbnail 2120" width="800" height="450"></a></p>

<p>What is this EPT thing, right? We know that people don't need to know those acronyms, but we wanted to help people understand that in this example, you're getting a total of $500 worth of  subsidy or assistance between these two different sources of help, and this is set to expire in December.</p>

<p>We started sending that out in July, pretty early on. In comparing notes with our colleagues across the nation, we were all trying to figure out what to do to get people's attention and to really start to plant that seed. I would draw your attention to the right, which is enabled by what Chris just described. This email goes longer and longer, but it's basically very tailored to each household and each segment. It shows what they need to see in terms of their choices of switching from different metal tiers—gold, platinum, bronze, and so on—or even looking at different plans.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0zjchhefffi331pooi5k.jpg" alt="Thumbnail 2200" width="800" height="450"></a></p>

<p>That sounds pretty standard. I don't expect you to do the wave for me, maybe outside I can induce you to do that. But I can tell you for Covered California and for our peers across the nation, this is big, right? It's us really leaning hard into what is the customer experience that today's modern consumer expects from us, but that frankly, large government organizations often find hard to do. It is hard to do for a lot of reasons, but because of all the technology and investment of time, we're able to do that. </p>

<p>What I would say is that I'm doing this one. Okay, here we go. So that was the outreach. Now we've reached the people. They might have interest, they might have questions, so they might call back in. When they reach the contact center, we wanted that personalized experience to continue. In a similar manner as for the outbound campaign, we're now looking at providing that customized experience for the customer as they're coming back in.</p>

<p>We set up custom messages. When a person calls in, they're going to be connected to a contact center—in this case, Amazon Connect for Covered California. When they connect to Amazon Connect using text to speech, we're doing a dynamic lookup in Salesforce to find that customer profile information and find that audience segment and that risk score that they're assigned to. Then we're dynamically going into DynamoDB to look up and find a customized message for that audience segment and that person. Using text to speech and Amazon Connect, we're able to play that message so it's customized to them.</p>

<p>At some point, they're going to get transferred to an agent. When they get transferred to an agent, the agent is also going to get played a customized whisper message letting them know who this customer is and their potential risk. Now that agent is going to provide personalized support for that particular caller. In addition, the Deloitte team developed a customer retention workspace for the enrollers so that they have a set of high-risk or at-risk customers that they can reach out to. They have the ability, once they reach out to them, to then score or disposition them—like, we reached out and they renewed, we reached out and they're switching plans, we reached out and they declined.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F37wb0iib4hpu41yz2lq2.jpg" alt="Thumbnail 2370" width="800" height="450"></a></p>

<p>So now we're gathering a lot of data, and that in turn fed into dashboards at the end so they now have much more information about how effective these campaigns are. You're receiving customized messages when you're calling in. You're getting customized messages, and the agents supporting you are supporting you with information. They know who you are and how to support you best. The best part, maybe the best slide in this whole slide deck, is the impact that this has created. </p>

<p>I tried to steal Chris's slide because if you spend enough time with IT guys, you want to become one. But from a results perspective, what it really ends up being about is what does it do, right? It's not personalization for the sake of it. It's not all this work just because we think it's right or we think it's going to pay off. What we're sharing here is on the top part. These are immediately following the campaign, literally the next day after the campaign launched. Again, this is multimodal, so you have email, you have SMS, you have other things going off. We had five times the phone calls that we normally would on that day. We had 125 percent more traffic to our website, and we had a 40 percent increase in logins. We can deduce that those people were looking at the plans that we sent that personalized information to.</p>

<p>That's just the first day after this was sent. When we look at the first two weeks after that campaign, it's basically a 3 to 13X improvement in terms of retention. Again, the goal is retention. For people who experienced the personalized, the aggregate effort of being more segmented and more personalized, we can see the payoff here, and we're very hopeful that in spite of the price and affordability challenges, this is the approach that we're going to take because we're not going to give up on any single Californian.</p>

<p>The hard math of it is that we can only do so much, and at some point there will be for some segments a very tough choice, which I've said multiple times. We don't pretend that any amount of personalization is going to get them over that tough choice because it becomes Maslow's hierarchy. It becomes prioritization for them in their life. But at the same time, who would we be if we give up on this? We're going to continue to do everything we know how to do and everything that you all in the audience can help us do to fight to keep people covered.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2490" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgfn7hzium37a324oo7yz.jpg" alt="Thumbnail 2490" width="800" height="450"></a></p>

<h3>
  
  
  Technical Implementation and Future Enhancements: Salesforce Integration and AI-Powered Agent Assistance
</h3>

<p>If we can't get them covered, we're going to fight to help them understand that we care about them. They should pick up the phone and we can walk through whatever options they want to explore with us. These results are not just a vanity set of numbers for us. They're proof that we're on the right path in terms of our approach.  I'm going to do this slide because I feel like I can't. I'm just kidding. So we'll talk. I've already briefly talked about how this works, but we'll walk through it again.</p>

<p>Based on the risk scores and the audience segments, an encrypted CSV file was developed and built by the Deloitte team and loaded into Salesforce. Now Salesforce has a mapping of the customers to their risk score and their audience segment. When a customer calls in to Amazon Connect, there is a lookup that's done through Lambda back to Salesforce to find out what that customer's risk score is and what their audience segment is. From that, there's a lookup to DynamoDB to get the specific message that's going to be played in Amazon Connect using the built-in text-to-speech function in English and Spanish.</p>

<p>That message is then played back to the customer while they're waiting in queue. At the same time, the whisper message for the agent is also being retrieved and will be ready when the customer is transferred to the agent. There are a lot of other services that were used to build this. I mentioned Snowflake earlier to build the risk scores and the audience segments. A lot of analytics tools were used, visualization tools like Tableau as shown here. Lots of things went behind the scenes to build the model, but on a daily basis and on a per-call basis, DynamoDB, Lambda, Salesforce, and Kinesis to stream out the analytics data to Varent today are all used on every single call, providing that personalized experience.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2640" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foxelci16jmbwy6e4e7ub.jpg" alt="Thumbnail 2640" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2650" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpv3f00r2la60giaxu40d.jpg" alt="Thumbnail 2650" width="800" height="450"></a></p>

<p>This is just the first phase. The data is going to continue to be analyzed and looked at. The team collectively is analyzing the data, and this can be continually improved on. This is really exciting stuff. So what's next?  As we're looking out into 2026 and some enhancements that can be made, one of the big items that stands out is a deeper integration with Salesforce.  This is really going to provide benefits not only for the customers of Covered California by providing personalized self-service when you call in. Lookups can be made back into Salesforce and that personalized self-service information can be provided right in the IVR, but also a lot of agent assistance.</p>

<p>Things like dynamically providing answers to questions to the agent live while the conversation is happening and a summary of contacts when it's over. Instead of spending 60 seconds or 2 minutes typing up a summary of what just happened on that contact, you can use AI to provide a summary of that contact and move on to the next one. That not only helps with that particular call, but the next call when that person calls again.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2720" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F06b8e4c0du38b1of7wxm.jpg" alt="Thumbnail 2720" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2730" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6jbhq2tbu3reugxs5ly.jpg" alt="Thumbnail 2730" width="800" height="450"></a></p>

<p>Now you can see that you talked to this person and discussed this topic. It helps all along the way. We just announced this week at re:Invent some of what that looks like here with Salesforce Contact Center with Amazon Connect.  An inbound call comes in through the Salesforce interface. An agent doesn't have to leave their screen, so they're not working from multiple screens. They can answer, and in this case it's a chat.  The customer's record is pulled up, so now the agent has a complete view of all that customer's information, including previous contact information and everything about them.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2750" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6fqxi8w6smxlw9gawivn.jpg" alt="Thumbnail 2750" width="800" height="450"></a></p>

<p>As they start chatting, this example is about someone who needs help installing a generator.  The customer says, "I need some help installing a generator." While they're talking, or in this case chatting, but this works with voice as well, Amazon Connect through Salesforce is providing answers to that customer's question or inquiry in real time. As an agent, I don't have to say, "Let me put you on hold and go look that information up." That information is being brought right to the agent live. In this case, because it's a chat, they can literally just copy it and paste it, helping that customer get their answer immediately.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flfy2xzpuknmle65a84pu.jpg" alt="Thumbnail 2790" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2810" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1zk2xlq012urcom44aru.jpg" alt="Thumbnail 2810" width="800" height="450"></a></p>

<p>We could provide sources as well,  so if you needed to dive deeper or look up more information, you could. And down here, though I cut it off on the screen, an agent could even type in and ask deeper questions about this particular topic.  All of this now helps answer the customer's question faster and ultimately provides a better customer experience for customers of California.</p>

<p>With that, I'll turn it over to Kevin. I think there are just a couple of thoughts to close with and to take away. One, we started this journey looking for partners, not vendors, but for human beings who had affinity for those three things I talked about earlier. Our obsessions, our mission: affordable, high-quality healthcare is a right for every resident of the state of California. The second one is about the consumer journey and trying to remove impedance or friction from that journey so that consumers can get from point A to point Z quickly, smoothly, with no issues, and feel confident that they have the right solution at the end of that journey. And obviously, information security and privacy.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2860" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsmtjkctrp4vbx17lts9z.jpg" alt="Thumbnail 2860" width="800" height="450"></a></p>

<p>With those three priorities, we have two fantastic partners: Deloitte as our system integrator  and Amazon as our AWS infrastructure provider. These people have the same affinity for our mission that we do. Glenn and I are employees of the state of California, and we're here because we wanted to do this mission work. We have two partners who are also here, and we treat them like they're our employees as well. We are all colleagues. I think the second thing is that we spent a lot of time today talking about data, AI, and machine learning. It's really the transition or translation of data into insight that AI and machine learning gives us, and that insight actually drives action.</p>

<p>The insight that drives action is something like, "30 percent of your people are leaving and never coming back. What are you going to do about it?" The insight that drives action is, "Let's create a custom message and a custom outreach campaign that will get every single human being to think I'm thinking about you in this transaction. I'm thinking about your family and what your needs are, and I'm giving you a custom plan. I'm explaining it in a language and in a method that you can understand and take action on too." So it's that transformation of data into insight that drives action, which is really at the heart of these two use cases that we provided.</p>

<p><a href="https://www.youtube.com/watch?v=QQHbbopPdRY&amp;t=2960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbrghgnhtf0n3nngppjff.jpg" alt="Thumbnail 2960" width="800" height="450"></a></p>

<p>Anything else to add, Glenn? We're just excited to keep this going forward. This is just the beginning, and we've done this collectively in a very short period of time, so we're excited to keep charging ahead. Yeah, great. Thank you for your time this afternoon. We enjoyed telling you our story, and we'll be at the back of the room if you have questions. Thank you. </p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

