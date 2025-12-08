---
Title: AWS re:Invent 2025 -JPMorganChase & Amazon Advanced Solutions Lab drive quantum R&D on Braket-CMP327
Description: 
Author: Kazuya
Date: 2025-12-08T21:05:34.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 -JPMorganChase &amp; Amazon Advanced Solutions Lab drive quantum R&amp;D on Braket-CMP327</strong></p>

<blockquote>
<p>In this video, Martin Schuetz from AWS and Romina Yalovetzky from JPMorgan Chase present their quantum R&amp;D collaboration on solving the Maximum Independent Set problem for portfolio optimization. They introduce qReduMIS, a hybrid quantum-classical algorithm that combines classical Pac-Man reduction techniques with quantum sampling on Amazon Braket. Testing on QuEra's Aquila device with over 200 qubits, qReduMIS achieved 89%+ success probability versus quantum annealing's exponential decay, demonstrating practical quantum advantage for financial optimization problems.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/qbbwbwzttTY">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn0pbokj7a37sa3sdbyns.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<h3>
  
  
  Introduction: AWS and JPMorgan Chase Collaboration on Quantum R&amp;D
</h3>

<p>My name is Martin Schuetz. I'm a Principal Research Scientist and the Global Lead for the Amazon Advanced Solutions Lab, and it is my great pleasure to be here today with Romina Yalovetzky, Vice President and Research Lead at JPMorgan Chase. Romina and I want to take you on a little journey today and show you how JPMorgan Chase and AWS have collaborated together on quantum R&amp;D in our larger effort to advance quantum research together.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=40" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhyhiekjp775ijeg9wiff.jpg" alt="Thumbnail 40" width="800" height="450"></a></p>

<p>Now if you've joined the session today, some of the questions you may have are: What is quantum computing? Short but profound. What does quantum R&amp;D look like today in 2025? And what are some of the use cases in industry that we are exploring together? These are some of the questions that Romina and I will try to answer with the following agenda.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=60" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6rvelph5tk4gtv6naxud.jpg" alt="Thumbnail 60" width="800" height="450"></a></p>

<p>We're going to kick us off with a broad overview of quantum computing and how we embrace this emerging technology at AWS. We're then going to double click on our collaboration with JPMorgan Chase. We're going to discuss some of the challenges we faced, some of the lessons learned, and then describe the solution that we came up with: a hybrid quantum-classical algorithm that we tested with more than 200 qubits on Amazon Braket. We will share with you some of the research results that we achieved together, and then we're going to conclude with a quick summary and an outlook for future research and next steps.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=110" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6g4teol6vl4x2j0mcuo8.jpg" alt="Thumbnail 110" width="800" height="450"></a></p>

<h3>
  
  
  Understanding Quantum Computing: A New Paradigm Beyond Classical Computation
</h3>

<p>Okay, so let's get started with the elephant in the room. What is quantum computing? And maybe let's first say what it is not. Quantum computers are not a classical computer that just runs faster, but really a new paradigm, a new frontier for computation that leverages the somewhat intricate and bizarre laws of quantum mechanics to perform computations in novel and improved ways. These quantum mechanical predictions might seem bizarre to us because we live in a macroscopic world, but if you consider the microscopic world of single electrons, single atoms, single photons, these quantum mechanical laws and predictions have been tested and verified again and again over the last few decades. This has boosted our confidence in this theory up to the point that we are now ready to build machines that exploit these fundamental quantum mechanical laws.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fiqaxkohj7h8ynwcfl7iv.jpg" alt="Thumbnail 170" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=190" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3kf9vfsa3rg0nn1pwp51.jpg" alt="Thumbnail 190" width="800" height="450"></a></p>

<p>And that originally dates back to the early 1980s, where Nobel Prize laureate Richard Feynman said, "Nature isn't classical, damn it, and if you want to make a simulation of nature, you better make it quantum mechanical. And by golly, it's a wonderful problem because it doesn't look so easy."  What Feynman recognized early on is that the cost of simulating a quantum system grows exponentially with its number of particles. So if you consider already a small quantum system with 50 to 100 particles, or qubits, and if you want to emulate the system on a classical computer, your memory requirements are going to balloon. They're going to go through the roof, and even for already these small systems with just 100 particles, you're going to talk about orders of magnitudes that are on the level of the number of particles in the universe.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=230" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7w0uiwa5p8gijljbd7jv.jpg" alt="Thumbnail 230" width="800" height="450"></a></p>

<p>So let's try to unpack this. At a fundamental level, all our information, our classical information, is stored in binary bits that we typically label as 0 or 1. It doesn't matter if you take your cell phone, your laptop, or a supercomputer. Now if you take two of these classical bits, at any point in time, these two bits can be in one of four configurations: 00 or 01 or 10 or 11. However, if you take the quantum equivalent, two quantum mechanical bits, or in short qubits, they can be in 00 and 01 and 10 and 11 at the same time. That's perfectly fine by quantum physics because of the superposition principle.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=290" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcubkx4dsm1fatfg0j8vm.jpg" alt="Thumbnail 290" width="800" height="450"></a></p>

<p>More formally, how we write this down: we write, in general, a quantum mechanical wave function labeled here as psi, just because we physicists like Greek symbols. So if you start with a general quantum mechanical wave function, let's start with one single qubit. If you want to describe one single qubit on a classical computer, you're going to have to define two numbers.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=350" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff1uhsf0edw3zhj61x17y.jpg" alt="Thumbnail 350" width="800" height="450"></a></p>

<p>A and B. If you go one step further, if you want to define a general two-qubit state on a classical computer, you will have to track, define, and track 2 to the power of 2, which is 4 numbers: A, B, C, and D, because this wave function can form a superposition and can live in the space of these four computational states at the same time. If you take three qubits, it's 2 to the power of 3, which is 8 numbers. If you take 50 qubits, it's 10 million billion complex amplitudes that you have to track, and that nicely exemplifies this exponential wall that you will hit if you try to replicate or emulate a small quantum system with  just 50 qubits on a classical computer.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=360" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5ap0yxqfbzl1qi07gcdt.jpg" alt="Thumbnail 360" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=380" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxk3cljkwf9jgiwnajwdj.jpg" alt="Thumbnail 380" width="800" height="450"></a></p>

<p>So taking this one step further, we then as a community realized that quantum  computers can actually also solve some problems with exponentially fewer resources than classical computers. And that means that really what we are shooting for here is not just a, quote unquote, just 2x speed up as you might achieve with, say, your next generation GPU, but really  a fundamentally different 2 to the power of x speed up that will allow us to solve problems that we could not solve otherwise because they're intractable for classical computers.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=400" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7rwjqmhq2d4oses3lw19.jpg" alt="Thumbnail 400" width="800" height="450"></a></p>

<p>So where do we anticipate these kinds of speed ups? We've categorized and identified here  four major categories. On the left, there's physics and chemistry and material science, and these are intuitively relatively easy to understand because here, at the end of the day, you have to solve inherently quantum mechanical problems. Take, for example, drug discovery. You want to solve for the ground state of a large molecule. That's inherently a quantum mechanical problem, and you will have to solve Schrödinger's equation. There's arguably no better machine suited to complete this task than a quantum computer itself.</p>

<p>In material science, take our quest to build high-temperature superconductors. You will have to simulate and understand the behavior of many strongly correlated electrons. Again, that's inherently a quantum mechanical problem that you better solve with a quantum machine. Now, what is maybe even more striking is that later we realized that quantum computers could actually also solve classical problems efficiently. And one of the most famous examples is in cryptography, where then in the 90s, Peter Shor, with the Shor algorithm, showed us that you could actually crack classical encryption schemes such as RSA, the security of which rests on the hardness of factoring large numbers, something that cannot be done with classical computers, even supercomputers. But Shor showed us that this is an easy task for a quantum computer.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=530" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4prhkq0vn4rcy5t3j7i8.jpg" alt="Thumbnail 530" width="800" height="450"></a></p>

<p>Now, taking this even one step further, we then opened the aperture towards other classical domains, in particular optimization and machine learning, where quantum computers hold the promise to solve problems such as resource allocation or portfolio optimization. And portfolio optimization will be actually the example use case that Romina will then discuss in the second part of this presentation. Okay, so this was kind of a broad overview into quantum  computing and some of its application areas.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2g9ojupgqr8ililyb17t.jpg" alt="Thumbnail 540" width="800" height="450"></a></p>

<h3>
  
  
  Quantum R&amp;D Today: AWS's Approach with Amazon Braket and the Advanced Solutions Lab
</h3>

<p>So what does quantum R&amp;D then look like today? So for  people in the field, scientists like Romina and me, the North Star is to lay the groundwork for future production workloads, typically through hybrid quantum-classical workflows. Now, to get there, we typically work backwards by first identifying target use cases that, on the one hand, are computationally hard and are interesting to your business. Once we have identified such a use case, we then research new quantum algorithms, and we benchmark them against the state-of-the-art classical algorithms. With that, we develop new IP and push the boundaries in our effort to ultimately lay the groundwork for the quantum future.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=590" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fshxpbsp9nne6qdr7gq4o.jpg" alt="Thumbnail 590" width="800" height="450"></a></p>

<p>Now, another twist to this is our effort in bringing this technology from a niche technology today towards broad adoption in the future. So how do we think we can go about this and achieve that?</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fryir97png5an87hxvri8.jpg" alt="Thumbnail 610" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=620" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0g7bfd8cmo1q4m48xkxn.jpg" alt="Thumbnail 620" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=630" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmhq4gtu4btkx5bfbnjmv.jpg" alt="Thumbnail 630" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=640" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzrh9n2rejn63cik14rld.jpg" alt="Thumbnail 640" width="800" height="450"></a></p>

<p>Well, we think we have to first research quantum devices.  We have to research and benchmark quantum algorithms. Once we then take quantum algorithms and quantum devices together, we can solve some first scientific  use cases, but ultimately again, the North Star is to solve industry-relevant problems with broad application  and in doing so in our journey as a community, we want to switch gears from studying the hammer towards using the hammer. </p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=650" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9d3wmwpxe05nyi9uxux2.jpg" alt="Thumbnail 650" width="800" height="450"></a></p>

<p>Now to help you on this journey towards using this hammer, we offer a suite of tools and offerings. One of them  is Amazon Braket, our quantum computing service on AWS, which provides one consistent experience no matter what your workload might look like. So this could be simple experimentations, this could be deep research, all the way to future larger scales in the future, all through one consistent experience where we provide access to different quantum hardware providers, third-party quantum computers, as well as our own quantum hardware in the fullness of time.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=690" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxuozyf6w5g2hgmmbwstx.jpg" alt="Thumbnail 690" width="800" height="450"></a></p>

<p>Now if you access Amazon Braket today, already today, you will  find a broad portfolio of quantum devices that features quantum computers based on trapped ions from IonQ and AQT, two superconducting devices from Rigetti and IQM, all of them accessible and programmable through the standard quantum circuit paradigm. But we also offer access to QuEra's Aquila device, which already today comes with 256 qubits, and this one is programmed through the paradigm of analog Hamiltonian simulation, which is geared towards solving optimization workloads.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=740" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpnb1agsgny9g6tud56il.jpg" alt="Thumbnail 740" width="800" height="450"></a></p>

<p>Another offering how we want to help you on your quantum journey is the Amazon Advanced Solutions Lab.  That's a team of scientists that I lead. We engage in deep custom R&amp;D engagements that are tailored towards your unique business challenges. We typically follow a two-pronged approach. On the one hand, we research and develop novel quantum algorithms, and at the same time we deliver business value today with state-of-the-art classical algorithms, often quantum-inspired or physics-inspired algorithms that adopt ideas and principles from quantum mechanics but can run already today on classical compute.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F200yawsuhxe265ydqm4i.jpg" alt="Thumbnail 790" width="800" height="450"></a></p>

<p>Through that we develop IP and know-how and help you prepare for the quantum future. And I think the most important message and outcome is with that we  provide actionable outcomes without the hype, such that you can understand if and when quantum will impact your business and inform your business stakeholders with target use cases. And I think one beautiful example is the work that we did with JPMorgan Chase, and now Romina will show you and walk you through our collaboration.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=830" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq7f3tkggtov243listin.jpg" alt="Thumbnail 830" width="800" height="450"></a></p>

<h3>
  
  
  JPMorgan Chase's Perspective: Optimization Challenges in Finance and the Maximum Independent Set Problem
</h3>

<p>Thank you Martin. Hello everyone, good afternoon. My name is Romina Yalovetzky, and it's my pleasure being here today to  share with you all how at JPMorgan Chase we have been collaborating with AWS. So we are the Global Technology Applied Research team at JPMorgan Chase. We conduct applied research focused on the frontier technologies to transform scientific findings into business value.</p>

<p>To do this, we work on four areas. We assess the impact of novel technologies on the firm and for our customers. We perform research driven by use cases to enhance client experiences and optimize business processes. We offer competitive advantage via research-driven solutions, and we execute proof of concept initiatives to validate and demonstrate innovative solutions.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=900" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fz0loyoanjybtsvj2bloz.jpg" alt="Thumbnail 900" width="800" height="450"></a></p>

<p>Finance is rich in computationally hard problems in optimization.  Actually optimization is ubiquitous in our industry.</p>

<p>We can find this type of problem in different areas in the financial industry. Just a few of them are portfolio selection and optimization, pricing and hedging of options, risk management, and also asset and liability management. So as you can see, optimization is all over the place in finance.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh47lg5bzht1yw6nc09uf.jpg" alt="Thumbnail 930" width="800" height="450"></a></p>

<p>And just to give you an idea of one of these problems that can be a potential use case for quantum computing, this is a portfolio selection problem. So here we consider a universe of assets. These assets can be stocks, derivatives, different types of instruments. And we also consider the correlation between these assets, and the idea here is that we would like to find the largest set of uncorrelated assets.</p>

<p>One way of visualizing this is by putting each of these assets, let's say A, B, and C, and all of them in the graph, so you can see each node represents one of these assets. Then we have the edges between them which represent the correlation that we have. So for example, in this case, A is correlated with D, but A is not correlated, for example, with the asset E. And now we would like to find the largest set of uncorrelated assets.</p>

<p>In this case, you can see them colored in purple, so we will have A, C, and G. And you can see that these nodes are not connected with an edge. By doing this, we can find the set of uncorrelated assets, and this could be a very diversified strategy for then allocating capital. In principle, this problem we can have 2 to the 7, so 128 solutions if you would like to try out all possible combinations. So this is an NP-hard problem, and actually we have seen that this problem can become very hard.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1040" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvflcblnwusxg2z4gdjs1.jpg" alt="Thumbnail 1040" width="800" height="450"></a></p>

<p>So let me try to explain a little bit what we mean with hard and what is the challenge that we see in optimization in finance. So for example, here you can see the runtime in the Y-axis and in the X-axis the problem hardness. And what we see is that the runtime can scale exponentially with how hard the problem is.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbjq028x3tlkxrvpelgp5.jpg" alt="Thumbnail 1060" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1080" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnhji5yx8138taex8k1mv.jpg" alt="Thumbnail 1080" width="800" height="450"></a></p>

<p>One notion of hardness is the size, so let's say that we consider a universe with 100 assets. We will see that all the possible combinations that we can have is 10 to the 30. This is 10 followed by 30 zeros. Now increase a little bit more our universe,  and now let's say that we have 1,000 assets. Now we will have 10 to the 301 possible combinations if we would like to do the brute force and go over each possible combination. So now remember that this is 10 followed by 301 zeros.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1110" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fawuhhw1a07cys5ywjsv7.jpg" alt="Thumbnail 1110" width="800" height="450"></a></p>

<p>So we can see in this example how exponentially the runtime can increase. So actually we are very interested in that side where we identify problems  that are very hard for classical computers, as actually those type of problems that we can try to find some potential use cases for quantum.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs2ig0v0fi1pvhk45r9ke.jpg" alt="Thumbnail 1120" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx4jmdtminabkhzfpdv1p.jpg" alt="Thumbnail 1150" width="800" height="450"></a></p>

<p>Actually, this problem of portfolio selection can be framed as a more general problem which is known as a Maximum Independent Set. This is a problem that has been studied widely in the computer science community, and the idea here is that we are going to consider a graph of this type, and we can do this more generally instead of thinking about assets as each  node. We are just going to think about nodes and edges, and the idea here is to find the Maximum Independent Set. This is the set of nodes that are not correlating between each other and this is the maximum size that we can get.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1190" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0hro6iwfeo1rqhaujn5o.jpg" alt="Thumbnail 1190" width="800" height="450"></a></p>

<h3>
  
  
  The qReduMIS Algorithm: A Hybrid Quantum-Classical Solution with Pac-Man Reduction
</h3>

<p>This problem has applications in different domains such as map labeling, network design, vehicle routing, and as we commented before, in quantitative finance with the portfolio selection problem. So now let me comment a little bit on the situation in quantum computing  when we started our collaboration with AWS. So by the time there was an empirical demonstration of a quantum speed-up published in the journal Science by Ebadi et al.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1210" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq7mt9h7yzsubd0bted48.jpg" alt="Thumbnail 1210" width="800" height="450"></a></p>

<p>The idea in the paper is that the authors  showed experiments with up to 289 qubits for the Maximum Independent Set problem, which is a problem that we just discussed. In the paper, they show some quantum speed up in the form that you can see there in the plot. In terms of success rate, where larger is better, it means that the algorithm can run faster. We see that with quantum, we get that curve that is over the classical. So in purple, we have the performance with quantum, and in blue, we have classical. We can see this performance boost that we can get with a quantum computer. So this was a very good motivation for a collaboration between JPMorgan Chase and AWS at the time, and it was also very much motivated by this problem of the Maximum Independent Set. As we have seen, the portfolio selection problem can be framed in this way.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1280" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fanly4n5em6t0xphzi4o2.jpg" alt="Thumbnail 1280" width="800" height="450"></a></p>

<p>Through our collaboration, we have published four scholarly papers.  There you can find the links for the archive version of them, and we have also released the code that we developed for these works, which you can also find in the papers. Through our quantum R&amp;D journey, we have managed to develop a suite of tools for solving large and hard problems on today's quantum computers. Today, together with Martin, we would like to focus on the very last work that we have published.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1320" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmrw2p577mljazkoqkswn.jpg" alt="Thumbnail 1320" width="800" height="450"></a></p>

<p>So let's take a look into this work. Here we introduced a hybrid algorithm that uses both classical and quantum computers. We call this algorithm qReduMIS, Q for quantum, R for reduction, and MIS for the Maximum Independent Set problem. As you can see there, we are going to use both the classical and the quantum computer because at the end of the day, we want to get the best out of these two computational paradigms. You can see we have the circle there because this is a recursive algorithm where we are going to call both the classical and the quantum computer.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwrxnfou54wn9h2on5ykn.jpg" alt="Thumbnail 1370" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1380" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbd12daxqu5kpemhr5et5.jpg" alt="Thumbnail 1380" width="800" height="450"></a></p>

<p>So let's take a look at what they are  going to do on the classical computer. Here, the idea is to perform an exact reduction.  So the idea is that we are going to try to reduce the problem. We are going to start with a graph, as we discussed before, where each node can represent an asset, and we are going to reduce it. To do this, we take into account a basic property of the Maximum Independent Set, which is that any clique can host at most one selected node. In this case, the idea of a clique is a group of nodes that are connected between each other. So for example, here you can see that 10, 11, and 12 are connected between each other, so that's one example of a clique.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1430" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbbj7mq99w4jl9yx1x6q1.jpg" alt="Thumbnail 1430" width="800" height="450"></a></p>

<p>Also, the idea is that we are going to reduce the problem  by removing what we call exposed corner nodes. Those are the ones that you see there in blue. For example, you can see that node 2 is connected with 3, and 3 is part of a clique with also 4, 7, 6, and 5. But you can see that node there looks like it's exposed because it's not connected with anything else. So that's one example of an exposed node. Also, the idea is that we are going to do this for the whole graph, so we're going to identify these exposed corner nodes.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1470" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Furc06l6u1v8jnd7lin81.jpg" alt="Thumbnail 1470" width="800" height="450"></a></p>

<p>Now we are going to play a sort of game that we refer to as Pac-Man.  So the idea is that we are going to start going over the graph, and we are going to start removing some of these nodes using the Pac-Man. For example, here we are going to start with node 1. As you can see here, node 1 is isolated, so trivially this node can be part of our solution because it's not connected with anything. In other words, it's not correlated with any other assets. So we are going to keep it as part of our solution, and we remove it.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1500" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqdr7qscps8wsj9s6lsw9.jpg" alt="Thumbnail 1500" width="800" height="450"></a></p>

<p>Now we are going to take a look at node 2. As we said, that's an exposed corner node.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1520" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F54cr6zjgglav7p1ln1k0.jpg" alt="Thumbnail 1520" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1530" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ff46f42rpdl7sigf6dor6.jpg" alt="Thumbnail 1530" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi17qnaen4c8tihs8yl0o.jpg" alt="Thumbnail 1540" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1550" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhdhya9quze6f88755qas.jpg" alt="Thumbnail 1550" width="800" height="450"></a></p>

<p>We are going to keep it in our solution, but we're also going to remove node 3 because remember that we don't want two nodes that have an edge together. So we remove it, we come here, we do the same with  nodes 8 and 6, and node 6 goes away. We do the same now with nodes 9 and 10. Here again we take node 11  and we also remove node 12. Now node 13, again this node is isolated, so we're going to keep it for our solution.  And now we reach here a clique, and the idea here is that we can select any of these three nodes. In this case, we're going to select node 4.  And you can see in this example that by playing with the Pac-Man reduction, we have managed to solve the problem. So you can see now all these nodes that are in blue, these are going to be part of our solution. You can see that they are not connected with an edge, and this is the largest set of uncorrelated assets. In this case, we managed to do it 100%, the reduction, so you can see it's a very powerful technique. But in the most general case, maybe if it's a more complex problem, it's not going to be 100%.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1590" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1avhhsoli386w96dvspi.jpg" alt="Thumbnail 1590" width="800" height="450"></a></p>

<p>We have performed many experiments with different networks, different graphs of this form, and you can see them there. These are some networks that are used in social sciences. And I would like to draw your attention to the very last one, the PubMed. This is a network where each node represents a publication, and the edges between them are the citations that these publications have between them. You can see that this network is quite large. It has around 19,000 nodes. We applied this reduction, this Pac-Man reduction, and we managed to get now a graph that we call a kernel with 16 nodes. And you can see that we managed to do this in less than a second, around 600 milliseconds. So you can see that this technique is very powerful.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1660" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmheqzt90huwzz0gt9r5r.jpg" alt="Thumbnail 1660" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1700" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7wnwu4hp6cpz79cdjcxy.jpg" alt="Thumbnail 1700" width="800" height="450"></a></p>

<p>So now let's see it in action for a  problem that is of our interest. We are going to have a graph like this. Remember that each of these nodes can represent an asset in the market, and the edge is the correlation. We take this graph, we put it inside a classical computer, and we are going to play with the Pac-Man, this reduction. The Pac-Man starts going over the graph, is going to take some of these nodes, and they are going to be put in the basket of selected nodes. And it's also going to remove some of them in blue, and they are going to go to the removed basket.  We do this all over the graph, and there is one point that the Pac-Man stops and it cannot get inside. So in this case, we managed to reduce the problem by 73%. But now there is a point where there are no more exposed corner nodes for the Pac-Man to continue going in, and this is what we call the kernel. In other words, this is the hardest part of the graph where actually now this Pac-Man cannot go in.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1740" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8e207wsqbioxgqu9eg29.jpg" alt="Thumbnail 1740" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1750" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmbcm28pz4esqom6zzsww.jpg" alt="Thumbnail 1750" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1770" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F94kp0k7arzi86jfjkp1i.jpg" alt="Thumbnail 1770" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2db9c6aivluivl4vfa5.jpg" alt="Thumbnail 1790" width="800" height="450"></a></p>

<h3>
  
  
  Quantum Computers as Probabilistic Sampling Machines: Experimental Results on Amazon Braket
</h3>

<p>And now the question is, okay, we have managed to identify  the hardest part of this problem. Now how can we solve it? For this, we are going to use the quantum  computer. Intuitively, the idea is that instead of using the whole graph and inputting everything into the quantum hardware, now we are going to focus on the hardest part of this problem. So let's see how we use a quantum  computer. The idea that we leverage in this work is that quantum computers are probabilistic sampling machines. So let me show you what that means. So the idea is that we are going to take our kernel that looks like that,  we're going to put it inside a quantum computer, and now the question is, what do we get out of it? Let's consider a graph that is smaller, so maybe it's easier to understand.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1820" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F79pg2cbr64wzhya0xrpn.jpg" alt="Thumbnail 1820" width="800" height="450"></a></p>

<p>To make this easier to understand, the idea is that we are going to consider here only those four nodes. Let's see what happens. We put it inside the quantum computer and let's see the output. The idea is that we are going to execute  the quantum program a couple of times, let's say 500 times. This is what we typically refer to as quantum shots. Every time that we run, we measure, and with that we construct the histogram that you see there.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1860" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fce5eij6mzdodrhvjix3t.jpg" alt="Thumbnail 1860" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1880" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbdw21cf8hexafd2cltgo.jpg" alt="Thumbnail 1880" width="800" height="450"></a></p>

<p>In the X axis we have an identification of the node, so you can see there 1, 2, 3, and 4. Each of them refers to a node. What we're going to do is report how many times we see these nodes appearing in our solutions. For example, here, let's say that node 1  appears only 10 times. So out of the 500 times that we measure, we only see that node 1 appears in only 10 of them. In contrast, node 3 appears 490 times.  So the idea here is that if we see node 3 appearing so much across our measurements, it seems like it has high likelihood for selection. The idea is that we are going to keep this node as part of our solution. In this idea that we want to identify assets that are uncorrelated, this could be a potential asset that we would like to have in our solution.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1910" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4lukdk1d36v4j2vroql0.jpg" alt="Thumbnail 1910" width="800" height="450"></a></p>

<p>So now let's put together all the different ingredients that we discussed. The idea is that we start with a graph. We have our baskets empty, the ones that we're going to put the nodes to select and the ones to remove. Here we may consider a market graph. Now we apply a reduction, this Pac-Man. We start putting some of the nodes in their respective baskets and we reach the kernel, that gray part that you see there.</p>

<p>Now we take the kernel, we put it into the quantum computer, and in this case we identified the pink node, the dark pink node, as one with high likelihood of being part of our solution. So we put it in the basket of selected and we remove it together with the neighbors in the light pink, because we don't want to keep nodes that are connected. Then we go back to a classical computer and we try again. Can the Pac-Man now solve the problem to optimality? And in this case, yes, it managed to do it. So in this case we managed to solve the problem.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=1980" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feosl9s030xlqbd0nv46z.jpg" alt="Thumbnail 1980" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=2000" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flr6b2oy1dplx61jzx2pe.jpg" alt="Thumbnail 2000" width="800" height="450"></a></p>

<p>And the idea is the following: classical reduction together with a quantum computer as a sampling machine is a powerful technique to solve the MIS problem, and one example of this problem is portfolio selection.  So now we would like to share with you some of the results. As you have seen, this is the algorithm and we have performed some experiments using real quantum hardware. Of course, here with our algorithm, we utilize both the classical and the quantum computer.</p>

<p>For our experiments we leverage the Aquila machine, which is available by QuEra and we access it through Amazon Braket. The idea here in this graph is that we are going to compare our algorithm, qReduMIS, which is in blue. We are going to compare it with quantum annealing. So quantum annealing is a standard algorithm that we can run on this type of devices. QuEra's Aquila device is based on Rydberg atoms. So with this type of architecture, we can run the quantum annealing algorithm.</p>

<p>The idea here is that we are going to use our framework, the qReduMIS algorithm, and we are going to power it with quantum annealing. So here we compare the two results, the two algorithms. In the X axis we have the problem hardness. As we commented before, one notion of hardness could be the size of a problem.</p>

<p>Let's say in this example of portfolio selection, it could be how many assets we have in our universe, but also there are some other metrics that we can use to understand how hard a problem is. In the Y axis, we have the success probability. As we said before, the larger the better. This is a number between 0 and 1, so the closer to 1, the better. It means that the algorithm is going to run faster.</p>

<p>So here we can see in red quantum annealing. This is the standard quantum algorithm. We see that if the problem is not that hard at the very beginning of the plot, in some cases we get a quite good success probability. But as this problem becomes harder and harder, we start to see that exponential decay. It's such that at one point at the very end, you can see that the success probability is 0. That means that now we cannot get the optimal solution. We may be able to get a close to optimal solution, but actually the probability of measuring the optimal solution for a problem is zero.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=2230" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9zjfh90n4m9wzl8l5lqj.jpg" alt="Thumbnail 2230" width="800" height="450"></a></p>

<p>In contrast, you can see in blue the performance of qReduMIS. This performance is very much concentrated around one. You can see that for different problem hardness, it is very much concentrated there around one. At the very end of the graph, now that we are considering problems that are harder and harder, we see a small decay, but overall the performance of the algorithm is over 89%. So the main idea here is that we managed  to get a performance boost in comparison to quantum annealing.</p>

<p>As we said, we have run this on the real hardware, on the real quantum device through Amazon Braket. We have managed to go to a very large size, and also you can see that we can see different values of problem hardness. So yes, here we managed to overcome this suppression that, as we said, is very limiting because at some point we're not going to be able to get the optimal, while with qReduMIS in most of the cases we managed to get the optimal solution with some success probability.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=2300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffw6478axwclvw26h7rmq.jpg" alt="Thumbnail 2300" width="800" height="450"></a></p>

<h3>
  
  
  Summary and Future Outlook: Advancing Quantum Research Through Collaboration
</h3>

<p>With that, I would like to hand it over to Martin for the summary and outlook. Thank you, Romina, for this nice overview of our collaboration and some of the results that we  achieved together over the course of two years, culminating in four publications and this hybrid classical algorithm that we refer to as qReduMIS. So we tried to take you on a journey. This has been our quantum R&amp;D journey through this collaboration. We have developed several methods.</p>

<p>The key insight was that we want to exploit some of the state-of-the-art classical techniques where they are really the best at and exploit them as much as possible, such that eventually we can have the quantum computer focus on what is really the hard core or the hard kernel of the problem. For that, we developed this hybrid quantum-classical algorithm, which is a general-purpose solver for the maximum independent set problem, and we showed you how this can find applications, in particular in portfolio selection.</p>

<p>In our paper, we actually show how this can be part of a larger pipeline for portfolio optimization, where you then distribute your budget across these diversified selected assets. The results that we achieved, in particular our experiments with over 200 qubits, were all executed through the cloud on AWS on Amazon Braket. In the future, we're looking to test this algorithm on alternative hardware platforms. This algorithm, the concept per se, is actually hardware agnostic.</p>

<p>Here we focused on a device based on Rydberg atoms, but you could essentially plug and play and try your best, try your luck with, say, trapped ions or superconducting qubits, and then really compare them head to head for one single pane of glass. Another step in how we want to take this forward is obviously to look for other applications besides portfolio selection, because we think this is actually a pretty broad, powerful framework that combines the best of two worlds: the classical state-of-the-art reduction algorithms and the quantum samplers.</p>

<p><a href="https://www.youtube.com/watch?v=qbbwbwzttTY&amp;t=2460" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbsfc4eu3iw2g3ufam4rg.jpg" alt="Thumbnail 2460" width="800" height="450"></a></p>

<p>And with that, we'd like to thank you for listening. Thanks for coming, and a quick reminder: please complete the session survey  on the app. Thank you very much.</p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

