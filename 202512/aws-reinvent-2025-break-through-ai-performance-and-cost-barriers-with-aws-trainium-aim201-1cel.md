---
Title: AWS re:Invent 2025 - Break through AI performance and cost barriers with AWS Trainium (AIM201)
Description: 
Author: Kazuya
Date: 2025-12-08T22:06:13.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 - Break through AI performance and cost barriers with AWS Trainium (AIM201)</strong></p>

<blockquote>
<p>In this video, Colin Brace from AWS Annapurna Labs introduces Trainium3, AWS's latest machine learning accelerator designed for the agentic AI era. He discusses key 2026 trends including agentic AI workflows and domain-specific models, which drive demands for high-performance token generation and low latency. Trainium3 delivers 4.4x more compute and 3.9x more memory bandwidth than Trainium2, with innovations like high-performance data types and Neuron Switch achieving 5x tokens per megawatt. The presentation features customer testimonials from Poolside's Joe Rowell and Decart's Orian Leitersdorf, demonstrating real-world applications. AWS announces native PyTorch support, open-source Neuron Kernel Interface (NKI), and previews Trainium4, emphasizing their long-term commitment to ML silicon innovation.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/5SgZ3zRJFAY">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp965otlub4saohhwuehd.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<h3>
  
  
  Introduction: AI Trends Driving 2026 and the Requirements They Generate
</h3>

<p>Good morning. Thank you for being here today with us. On Tuesday, our CEO Matt Garman shared with you Trainium3, our latest generation server software and silicon. Today, I'd like to spend a bit of time with you to talk about the innovations that went into that work and why it matters to you. We'll look at some trends in the industry around generative AI, what's going to be driving those trends in 2026, and then see how the requirements they generate and how Trainium3 meets those requirements. We'll also hear from two customers, Poolside and Decart, who are writing fantastic applications on top of this technology already. My name is Colin Brace. I'm a VP at Annapurna Labs AWS. Let's get started.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=50" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F48rtqxiw1uxn31809m47.jpg" alt="Thumbnail 50" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=60" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmpsxbi6fjovp6kaey4y2.jpg" alt="Thumbnail 60" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=70" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvkzpmuktohuvrh8zhjip.jpg" alt="Thumbnail 70" width="800" height="450"></a></p>

<p>I'd like to start with the question of what will be driving AI in 2026 and share with you what we're hearing from customers.  The first trend I'd like to share is agentic AI. And this isn't just hype, we're seeing customers starting to adopt it already. Why is that?  With agentic AI, it gives you the opportunity to not only generate responses, but more importantly verify them, to make sure that whatever answers the LLM or generative AI is giving you matches your business requirements. And once that's been validated, you can turn that into action. We're hearing from customers the value of that, and we're seeing them start to deploy and develop these agents.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=90" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk43d5zh0lpyfy4v6l62a.jpg" alt="Thumbnail 90" width="800" height="450"></a></p>

<p>Gartner predicts that by the end of 2026, and that's only a year from now, 40% of applications will be using agents at that time. It's only 5% today, that's going to be very large growth. As an example, let's look at coding, where we saw that similar kind of growth. We started in the coding space with code completions in 2022 that moved to chat coding, which then moved to vibe coding. Now we're seeing agents and fleets of agents solving real world tasks through agentic AI in the coding space, and we see that same phenomenon happening in different areas as well.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj8rgxp8qty6ro9ew2uq1.jpg" alt="Thumbnail 150" width="800" height="450"></a></p>

<p>The next shift that we're seeing that customers are telling us is in the area of application or domain specific models. These models allow for much greater accuracy in the responses, and what's enabling this is the fact that training these models has become a lot cheaper.  You can start with the foundational model and then train based on information or data specific to your company. We're hearing specifics from in the legal case around LexisNexis has specific models, Bloomberg in the finance case, and in robotics companies where they have real world data that is shaping these models. These application or domain specific models are going to proliferate in addition to the agents as well.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=180" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbggr94zglph2eymv5ype.jpg" alt="Thumbnail 180" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2eqotjpccq0jk4gtk1cv.jpg" alt="Thumbnail 200" width="800" height="450"></a></p>

<p>The challenge that this brings is that there's going to be the need for an enormous amount of inference as well as training, which in total brings a lot of token generation demands. Looking at these two trends together, they drive a set of requirements that all customers will need to face to deliver on these demands.  The first is high performance token generation. As I mentioned before, the kind of token generation demands will be significantly higher than just generating a paragraph based on a chart. Not only are there long reasoning models to deal with long context models, reasoning flows, but in addition, there will be multitudes of agents running, demanding order of magnitude more token generations.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=220" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3fei92glanon06kzstim.jpg" alt="Thumbnail 220" width="800" height="450"></a></p>

<p>Low latency. If agents have any sort of interaction with computers and with humans, they'll need low latency to make sure that the workflow is not abandoned. Also, as agents compound and work with each other, latencies will slow down workflows overall. Low latency will continue to be an important requirement in the years ahead.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=240" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F386w50r3kfcpddql9xbt.jpg" alt="Thumbnail 240" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F42t1ljjsbpsdgrma7h52.jpg" alt="Thumbnail 260" width="800" height="450"></a></p>

<p>Easily accessible and cost effective accelerators. In order to achieve the first two, they need to be cost effective for your business needs as well as easily accessible, having capacity where you need it based on the geography requirements, the regulatory requirements, or other requirements like that.  Finally, you also need the scale to meet the demand, to have the capacity available as your workload scales up and also as it scales down. You need to be able to support small models, the application models, as well as large ones as well. And these are the requirements that drove Trainium3.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=280" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi550xhlb11b7b7aph62w.jpg" alt="Thumbnail 280" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftlc5ao447tgxz8axr565.jpg" alt="Thumbnail 300" width="800" height="450"></a></p>

<h3>
  
  
  A Decade of Silicon Innovation at AWS Annapurna Labs
</h3>

<p>It was designed for the agentic workflow era. Before I talk about Trainium3 and tell you about the innovation that went into it, we need to start with the story of silicon innovation at AWS and Annapurna Labs, and that gives the foundation for how we built Trainium3.  AWS Annapurna Labs has been innovating at the silicon level for over 10 years. We started working together back in 2013 to work on our first silicon, which is the Nitro card.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=310" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feei7knnb8ldmv8rqbpso.jpg" alt="Thumbnail 310" width="800" height="450"></a></p>

<p>I'll share a bit about how that was created and how these were created because the story is all the same.  We started looking at our workloads in EC2 and saying, is there any way that we can make this cheaper and more performant for customers. We were able to analyze the network patterns, the network storage patterns, and that caused us to build custom silicon called Nitro that made it not only cheaper to run from a power drop perspective and cost perspective, but also more performant for customers. We're now in our seventh generation of that and have millions of chips in production. Every EC2 server has a Nitro card in it.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzn9tdr784k0ayuds0tmn.jpg" alt="Thumbnail 370" width="800" height="450"></a></p>

<p>Our next entrance was for a host CPU built on the ARM ISA. Again, we looked at common workloads across AWS and we innovated from scratch for a CPU that could meet these demands at a much lower cost. And now we're at our fifth generation of Graviton chip. A third product line, which we'll spend time on today, was Inferentia and Trainium. These are our machine learning chips. It started back in the era of 2016, late 2016. We started to see MXNet, ResNet models being run on AWS and we asked  ourselves again that same question. What can we build at the custom silicon level that would make these workloads run faster and cheaper?</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fca0bhtnl5zqgg2ftuhin.jpg" alt="Thumbnail 390" width="800" height="450"></a></p>

<p>And that resulted in 2019, our first chip Inferentia launched, designed for demanding workloads at the time, which were BERT models and some CNNs. Since then, we've had a number of iterations each time  achieving more and more performance to handle more demanding workloads. Let's talk about Trainium 2. Last year, we announced Trainium 2, and I wanted to tell you about some of the progress we've made with that in production. Today, we announced Trainium 3, but the success of Trainium 3 was built on the learnings of how we've built out Trainium 2 as well. So let's spend some time on that.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=420" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8x45ae4ial46rxwj5r8c.jpg" alt="Thumbnail 420" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=430" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbz6286m8o2lu7l3us450.jpg" alt="Thumbnail 430" width="800" height="450"></a></p>

<h3>
  
  
  Trainium2 Success: From Project Rainier to the World's Largest AI Cluster
</h3>

<p>Here are some of the numbers that we shipped Trainium 2 with, the specs.  It handled at the time, it could run on a single device, models from Llama, Qwen, and GPT-2 as well.  We innovated also at the server level, and this is a key point we'll return to over and over again. To really deliver the industry leading performance requires innovation at all levels: silicon, the interconnects, the servers, and the software. And with AWS Trainium 2, we had 64 Trainium chips all together with very fast interconnects in between them.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=480" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Favse1gnfapicymfyu9j7.jpg" alt="Thumbnail 480" width="800" height="450"></a></p>

<p>We innovated at two levels at the server. We had a special interconnect between the devices called NeuronLink, which allowed for extremely fast collective communications. We also had Elastic Fabric Adapter, which allowed for extremely large scale out, which I'll talk about in just a minute. So we have the chips, we composed those into servers, composed those into UltraServers. We also gave ourselves the goal to build extremely large data centers. We called that Project Rainier. The goal was to build  hundreds of thousands of Trainium 2 chips in production running the world's largest training cluster, and at the time it was five times larger compute than Anthropic's largest training cluster. That was the goal we set ourselves a year ago.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=500" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F76gxt42h4t95r04kntsi.jpg" alt="Thumbnail 500" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=530" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F396nwt83uyyso3hdmw59.jpg" alt="Thumbnail 530" width="800" height="450"></a></p>

<p>What did we do since then? Well, we deployed Trainium 2 at a record pace. And I emphasize this because part of  delivering for you the need to run your generative AI applications means getting the compute to you. That means understanding how to get the wafers from the manufacturer, how to assemble, how to build, how to get into our data center. And so, owning the end-to-end vertical integration of Trainium 2 allowed us to do that. And the results are here on the board. We now have more than three times Trainium 2 capacity in our data centers than the previous Gen AI instance across AWS. To achieve that, we ramped at a scale of four times.  That means inspecting everything that I mentioned before, from the supply chain to the assembly to the build, and we saw an incredible ramp. We're taking that learning into Trainium 3 as well.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=550" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgczxb3niqu8arm906dml.jpg" alt="Thumbnail 550" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=570" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwperc3gfzb2jczdpbmzp.jpg" alt="Thumbnail 570" width="800" height="450"></a></p>

<p>We continue to do more. Ultimately we were successful in Project Rainier. Within a year, we turned what was a cornfield in  Indiana into the world's largest AI compute cluster, with over 500,000 Trainium 2 chips deployed, all connected through EFA for a very large training cluster. In terms of performance, what does that mean? I want to share with you some numbers that we worked with in  concert with Anthropic, who runs their Claude models on Trainium 2. They run it across a variety of different workloads: 100 token input, 100,000 token input, video generation, and they see on average 1.4 times greater output tokens per second running on Trainium 2. It's phenomenal performance that has enabled applications like Claude to run as speedily as they do.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=600" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Famchgtji664veemstawe.jpg" alt="Thumbnail 600" width="800" height="450"></a></p>

<h3>
  
  
  Trainium3 Innovations: Hardware Breakthroughs for the Agentic Era
</h3>

<p>With that as background, I now want to introduce Trainium 3. Trainium 3 was built with the learnings from Trainium 2, how to build it out,  but also looking ahead for the trends that I mentioned earlier around agentic AI.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmo5sct4sb0zebhlt96ta.jpg" alt="Thumbnail 610" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=620" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdxh0w5qqmnvkas6zk06v.jpg" alt="Thumbnail 620" width="800" height="450"></a></p>

<p>Across the board, when you look at the specs of the chip, it's more  compute, more memory, more memory bandwidth, all to address the bottlenecks within machine learning workloads. Of course that's the chip. We also looked at the  server. This is aggregating together Trainium chips into a single system view. We've innovated at all levels that connect them together. Overall, you'll see 4.4x more compute, 3.9x more memory bandwidth than Trainium 2 Ultra server. That means with the same server you can serve more customers and run larger models.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=660" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8rrd3rbpawj2mlgywk7a.jpg" alt="Thumbnail 660" width="800" height="450"></a></p>

<p>I want to talk about three innovations that went into the development of this particular Ultra server, including the silicon, and then I'll talk about some of the performance results that we're seeing concretely. The first innovation I'd like to talk about is our end-to-end hardware innovation. I spoke earlier about the three different products we have,  the Nitro card, the Graviton card, as well as the Trainium card. With Trainium 3 Ultra Server, we've been able to put these together, and that innovation allows us to control things like the power draw and control things like how we assemble them together and how we work with the supply chain to deliver these together.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=700" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqowd5izj4pfrjp6e8b1j.jpg" alt="Thumbnail 700" width="800" height="450"></a></p>

<p>That end-to-end hardware innovation allows for our speed of delivery of this capacity to the data centers. It also allows us innovations such as supporting both air-cooled and liquid-cooled deployments. Those are important to make sure we get the capacity to you as soon as we can, as soon as the technology is available in production.  The next innovation I'd like to talk to you about is high-performance data types. Why is this important? Well, there's been a very helpful innovation in the software space that's discovered that while we train at large data types, BF16 or larger, during inference, you can reduce the data size without sacrificing accuracy.</p>

<p>This is a great innovation and observation and learning in the software space, and taking advantage of that in the hardware space can mean at least 2x amount of compute and 2x amount of memory bandwidth improvement you can get by using lower data types. What we've done in smaller data types with Trainium 3 is make that precision and quantization and dequantization seamless with no performance impact. So you get at least 2x performance across both compute-bound and memory-bound workloads. And we have some results on the board here with GPT-2 XL where we've done our training on the 1.2 billion parameter model, and you see a 2x improvement.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=770" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmx81xkdqltvk5skhecay.jpg" alt="Thumbnail 770" width="800" height="450"></a></p>

<p>The next innovation I'd like to talk about is something called Neuron Switch. I mentioned before the technology Neuron Link, which is this connectivity that puts the chips together,  enabling extremely fast interconnects so that when you have a model that requires tensor parallelism, for example, you get extremely fast connectivity between them. With Neuron Link, there was a particular topology called the torus that we used to connect them. The Neuron Switch allows much more direct one-to-one communications between all of the devices. And we're seeing improvements in the all-gather primitive, for example, of up to 6x the reduction in latency, and for all-reduce, a latency improvement of up to 2x as well. All this translates to much faster training time and much faster inference as well.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=820" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foqjbn9kf7y8dxm4slp3x.jpg" alt="Thumbnail 820" width="800" height="450"></a></p>

<p>So now I want to share with you what this looks like in production as we run some end-to-end workloads and what this means for you. One of the things we're very proud about is the improvement we're seeing in what we call tokens per megawatt. It's  a bit of an unusual way of looking at it or a different way of looking at it. Most look at time to first token or output tokens per second. We look specifically at tokens per megawatt because that dictates the operational cost of running the servers and how fast we can build them based on the power draw that we can get from the available data center. Very happy to report that we see 5x the amount of tokens per power draw. That means for your applications you can deploy them as is and you'll get 5x the amount of tokens generated for that workload.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=860" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4wt75p72k9h2exxf4pgg.jpg" alt="Thumbnail 860" width="800" height="450"></a></p>

<p>That's on the power draw side. I'd like to take a different view now from more at the developer side, how you can change your  models to get different performance characteristics for what you're looking for. So there's two options I'll walk through here that you can see how Trainium 3 operates well on either side and gives you options between interactivity and overall server throughput. If we start here, we'll see Trainium 2 is on the bottom and Trainium 3 is on top, and we'll compare between the two. What we'll see here is for existing applications with certain interactivity requirements, you can leave that as is if you wish.</p>

<p>By using Trainium 3, you'll get 4.5x more users being served. So it's the same interactivity, same output tokens per second, actually a bit better, but 4.5x more users can be served at that same interactivity.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=920" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu6jtkegdtl53v57ulmoj.jpg" alt="Thumbnail 920" width="800" height="450"></a></p>

<p>Alternatively, if you're deploying newer applications that have longer context lengths or longer reasoning chains, you can slide the lever and serve the same number of users,  but at a much higher output tokens per second, over 6 times the output tokens per second for the same number of users. So Trainium3 gives you that flexibility between the aggregate amount of tokens that your server can serve compared to the number of interactive users you want based on the needs of your application. It gives you flexibility between those two.</p>

<h3>
  
  
  Poolside's Perspective: Inference Optimization and Partnership with AWS
</h3>

<p>So there we've talked about the history of silicon innovation at AWS here, how we got to Trainium3, and some of the results of Trainium3. I'd like to invite to the stage now Joe Rowell from poolside to talk about some of the fascinating applications they're writing using this technology. Joe, thank you. Well, thank you, Colin. It's always a pleasure to get to speak about all of the wonderful work that we've done together.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=980" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2esyqb48qlubv7xw8m1f.jpg" alt="Thumbnail 980" width="800" height="450"></a></p>

<p>So I want to take a moment to talk about who we are as poolside, because I think it really helps shape how we think about inference.  So poolside is a company that trains foundation models from scratch. If you think about an OpenAI or an Anthropic, that's essentially the work that we also do, from data collection, pre-training, fine-tuning, reinforcement learning, the whole thing. And I think something that is well known to people who work in the foundation model space but is not necessarily well known outside of it is that the big training run that you do in pre-training is not really where you spend most of your time from a compute perspective.</p>

<p>In practice, most of the place where you spend compute time is in two other places. It's in customer inference, once you've made the thing and you actually give it into the hands of customers, and it's also in inference that you use as part of your training workloads. So for example, when you're doing reinforcement learning or synthetic data generation, all of these things require inference to work successfully. Even evaluations, right, that big headline number that everyone has when they release a model comes from inference.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1050" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F492um870ww8sl9fq1jgt.jpg" alt="Thumbnail 1050" width="800" height="450"></a></p>

<p>And the reason why I mention this as a point is because at poolside, inference is our number one optimization target.  If I can save 1% by doing a dirty hack somewhere, I absolutely will do it because for us, that translates into huge cost and time savings. Inference is one of these things that has a really elastic demand. When you have it, you just want more of it. And alternatively, if you don't want more of it, maybe you're happy with what you can do, then you can do it cheaper or you can do it faster. Inference has a lot of really wonderful properties that mean that chasing optimizations really benefits you.</p>

<p>Now I want to be clear here. Obviously we do optimizations for training, right, we're a company that is pursuing AGI and so optimizing everything is very important. But especially from a customer perspective, I would say that inference is indeed where we spend most of our time optimizing. Our customers really value the ability to take our models and deploy them in a way that enables them to do more with less or to do the same amount faster or so on.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1110" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh73p46be1i7uhspph5pi.jpg" alt="Thumbnail 1110" width="800" height="450"></a></p>

<p>Now, inference is kind of a strange thing in that  there's a lot of performance curves that you can go along. I can only speak for ourselves, but what we see are two different forms of inference in practice. We see internal workloads and external workloads, and actually these have very different characteristics. I'll give you all an analogy. Let's imagine that you want to get from point A to point B, just yourself. What you care about is the latency of you getting between point A and point B. You don't care about anyone else. What you want is a sports car. You want to be able to get your work done as quickly as you possibly can. This is typically what we see from our customer requests. Really it's individual developers or large teams that aren't necessarily taking a broader view, they just want the agents that they're using or the tools that they're using to complete quickly.</p>

<p>By contrast, the inference work that we do internally is much more about throughput. So to go back to our earlier analogy, let's imagine you want to move as many things as you can from point A to point B. Chances are you're going to pick a truck or something like that, maybe a plane or a boat. Neither of these things are particularly good from a latency perspective, but if all you care about is doing as much as you can per unit time, then this is what you want to do. You want to build things for throughput.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx12j8xc11son13osthmw.jpg" alt="Thumbnail 1200" width="800" height="450"></a></p>

<p>And something I really want to point out here is that all of this adds up to the fact that modern inference workloads are demanding and are becoming increasingly more demanding. </p>

<p>If you look at, say, GPT-3 or any model from a few years ago, not only were the models not very capable, but the elaborate techniques that we now expect simply didn't exist. Chain of thought and other reasoning techniques hadn't been implemented in these products yet. Actually, one of the places where we really see this is in context windows. A few years ago, you might have had a 4K context window, and for asking ChatGPT how many R's there are in strawberry, that would have sufficed. But now with chain of thought techniques and modern agentic workflows, you're seeing context windows in the millions of tokens.</p>

<p>This really points to needing flexibility from your inference solution. If you bake something into your hardware today that has some assumption about how things are going to be, chances are by the time you deploy it, it's no longer going to be the case. The industry is moving so fast that the old assumptions simply do not hold true anymore. The second thing is that this really is a cost thing. The cheaper you can make your inference, the more you want to use it. Actually, inference is one of these things where once you have it and it's good and efficient, from a research perspective, you find more things to do with it. We have incredibly elaborate systems internally at poolside that are all based around our ability to do inference cheaply and easily.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1290" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl4kewrcce2wruaxd26b0.jpg" alt="Thumbnail 1290" width="800" height="450"></a></p>

<p>Now, one of the very important pieces of that has been the work that we have done with AWS and Annapurna Labs.  So to give you all some background, we've been interacting with Colin and the team for roughly two and a half years now, actually quite a long time. Along that way, we've done a lot of work on Trainium together. We have given various bits of feedback on the software and the hardware stack. Colin and the team have helped us with model design, fitting things to the hardware, and optimizations.</p>

<p>If I can give you one tip, it's this: if you want to deploy your models on Trainium, I highly recommend that you reach out to the AWS team or work with someone at AWS to help you get the most out of the platform that you can. There are so many wonderful things that you can do with Trainium that sometimes just require a little bit of help. Once you get that help, it unlocks a huge variety of things for you and for your business.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1340" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2878e37wkr5rp2td5cvk.jpg" alt="Thumbnail 1340" width="800" height="450"></a></p>

<p>I want to talk very briefly about  how we use inference technically, in particular in the context of Neuron. You can see here on the right-hand side, this is one of our systems overviews that we have internally. This is for a thing called the Model Factory. You can see that in the middle of the top row, inference is a very large piece of all of this. This diagram is actually somewhat inaccurate because in practice this graph could actually be an all-to-all where everything could talk to everything else. But for the sake of this, you can already see that inference is a very important piece of what happens here.</p>

<p>Now, as a company, we're a PyTorch company. That's what we use to write all of our AI code, and Neuron has solutions for this. It has a capable tracing compiler that walks through, produces an XLA graph, and then compiles down some optimized kernels. Actually, there are various tweaks that you can apply to that code to make your model more efficient on Trainium. Going forward, I'm sure Colin will speak more about this in lots of the sessions this week, but there will be fewer tweaks that are required to make things work well here. I do want to point out that actually every week that I look at the stack, there are fewer things that you have to change to make your model work nicely on the Trainium stack.</p>

<p>Now, of course, I've said that the PyTorch compiler is very capable, but sometimes even with other platforms, there are low-level tweaks that you need to make to get the most performance out of it. If you think about the rise of DSLs for CUDA, for example, there are lots of small things that you need to do to really squeeze out those last drops of performance. Trainium is no different, and there's a wonderful domain-specific language called NKI that lets you do low-level hardware optimizations for the Trainium platform. Normally, I would say that we use this for the last 10% of performance.</p>

<p>One of the great joys of my life and for the other people that work on Trainium at poolside is taking PyTorch code that is quote unquote good enough, maybe 80% of the way there, and then gradually iterating over an NKI implementation to get something that is as fast as it can possibly be. Now, to be clear, obviously sometimes I still have to reach out to Colin and the team to get some insight that I've missed or some tweak that I didn't know about. But by and large, this is mostly work that you can do yourself once you have that knowledge in the first place.</p>

<p>This really brings me to perhaps the most sentimental point that I want to make, which is that the Neuron team at Annapurna Labs and AWS are really some of the best partners that you can ask for. I think it's a mistake to look at inference just from a compute accelerator perspective. There are so many things that you need to build and implement to get inference working at scale for customers.</p>

<p>Even if you just think about the ability to load checkpoints from somewhere, or the ability to automatically scale up your deployments, or the ability to do load balancing, these are all really systems problems. And it turns out that AWS has been building great systems for a long time. I've yet to find a problem where they didn't have a solution, but I'm sure if I ever find such a problem, a solution will not be far behind. AWS is very good at offering a full array of things that you need for inference.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc9enumifk8yhzgss4vdl.jpg" alt="Thumbnail 1540" width="800" height="450"></a></p>

<p>Lastly, I want to close by talking about the token economics of Trainium.  So Colin has already mentioned this in some way, but the economics of Trainium are incredibly compelling. The 5X number and all of the performance numbers that you saw in the keynote yesterday, I believe, we have seen them in practice. And in fact, every time we have taken a model and deployed it on a new generation of Trainium hardware, admittedly maybe after a few small tweaks, we've seen exactly the promised headline performance numbers and the performance per watt numbers. And this really just goes to show that all of these innovations that the people at AWS talk about every year really are true.</p>

<p>It's very easy to promise headline numbers that you will never actually see in reality. But in my experience, that is not something that you see from Trainium. Trainium almost always gives you exactly what the headline numbers promise. And the last thing I want to say is that you will have seen there are lots of announcements about Trainium that are happening this week. There are lots of sessions that are going on. I can only speak for us, but we're very excited about the advancements that are happening in Trainium.</p>

<p>Trainium3 looks like an amazing platform. All the new changes that are happening in the software stack are all things that we're very much looking forward to taking advantage of and looking forward to collaborating with you on. And with that, thank you all for listening. I'd like to hand back over to Colin.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1640" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9pch8y2uiybgonfebecp.jpg" alt="Thumbnail 1640" width="800" height="450"></a></p>

<h3>
  
  
  Developer Stack Evolution: Native PyTorch Support and Ecosystem Collaboration
</h3>

<p>Thank you, Joe, for sharing your experience. Fantastic work you're doing. You've been a great partner for many years. Thank you for that. Joe mentioned a few things around software, and that's what I'd like to spend the next few minutes on and talk about our developer stack. Of course, performance means nothing if it's hard to use. When we think about the customers that are interacting  and who need performance from Trainium, we think about three customer sets that we really want to understand and build solutions for and meet customers where they're at at three different levels.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1670" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffn9lgyz6hp6ydmahvfxe.jpg" alt="Thumbnail 1670" width="800" height="450"></a></p>

<p>Overall, our thought process is if you know JAX or if you know PyTorch, then you already know how to use Trainium. But let's look at these three together. We'll start first with machine learning developers. These are developers that need to productionize inference systems or training systems and get things up and running.  They don't necessarily have time to learn a new platform. They're going to rely on frameworks like vLLM or Hugging Face. And for these customers, that's what we do.</p>

<p>We have deep integrations with vLLM today. We integrate with the Transformers library from Hugging Face and frameworks like Ray and other things. So integration is key here, and that's what we've focused on for our machine learning developers. So with Trainium, right out of the box, you get integration with these. And of course, with the AWS services as well, EKS, Amazon ECS, we're deeply integrated to those services as well. So if you're familiar with these tools, with building solutions with other accelerators, you'll find that that just works on Trainium.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1720" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdv0i0xgo3mjyh31lwtv.jpg" alt="Thumbnail 1720" width="800" height="450"></a></p>

<p>The next set of customers are machine learning researchers. Now machine learning researchers are interesting. They are creating new models. They may be  trying new datasets and then tweaking the model for those datasets. What they need is really direct PyTorch access or framework level access so they can change it underneath, sometimes change it or create new models. I'm very excited today to mention what Joe referred to, which is that we're moving and we're building, we have something called PyTorch Native.</p>

<p>The idea is that if you know how to write on GPU, it will work automatically. That means inclusion for things like eager mode, torch.distributed capabilities, as well as torch.compile. And in fact, with the latest versions of PyTorch where the particular device has been abstracted, the exact same code will work across all of them. We've been partnering very closely with the PyTorch Foundation as well as the PyTorch maintainers and developers, and I have a small message to share with you from their perspective about the integration with Trainium.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1780" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0y1j7df595iqcufx3h4.jpg" alt="Thumbnail 1780" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxclnx5b6u6k2xwql560v.jpg" alt="Thumbnail 1790" width="800" height="450"></a></p>

<p>Hello, I'm Jana van Gruen, director of PyTorch Engineering at Meta.  I'm thrilled to talk to you about an important milestone in our collaboration with AWS. At the 2025 PyTorch conference, we shared our vision  for PyTorch as the open language of AI. A core part of that vision is simple. PyTorch should run everywhere. Researchers and developers should be able to use the same code, the same workflows across any hardware platform that they choose. That's why I'm so excited that AWS has delivered native PyTorch support on AWS Trainium.</p>

<p>That means that your existing PyTorch code just works. You get eager execution by default, the immediate intuitive developer experience that makes PyTorch, well, feel like PyTorch. Your distributed training APIs will work unchanged. Torch Compile will optimize your models automatically. And libraries like Torch Titan can run without modification, enabling production scale training.</p>

<p>We're also very happy to see Trainium supported in vLLM, the newest member of the Foundation for inference workloads. This completes the workflow and is especially exciting given where AI is heading. Joe will talk more about why this matters for the future of AI.</p>

<p>Thanks so much, Jana. I'm Joe Spizak. I'm the product director here at Meta for PyTorch, and I also sit on the PyTorch Foundation as a board member, a foundation that I helped found nearly three years ago. Just recently we brought in vLLM into the foundation, and as inference actually becomes really integral into this foundational shift we're seeing in AI, especially in pre-training, in post-training with reinforcement learning, we're now incorporating environments and real world scenarios and improving model capabilities and now kind of the agentic system level, the capabilities are improving.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1910" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4axsw4lpb75s7xs08jli.jpg" alt="Thumbnail 1910" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1920" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnobn3dmy8kyxbv6tf373.jpg" alt="Thumbnail 1920" width="800" height="450"></a></p>

<p>So one of the things that's changing really is how agents get developed in these agentic workflows. These systems are moving beyond just a prompt and response, kind of the traditional LLM system. We actually are moving towards these agents being able to take action, execute code, use tools, and interact with real environments and actually achieve real outcomes and real tasks. And so when we think about  some of these environments, this could be the system itself using code or executing code and being able to compile it and getting the right answer, mathematics all the way through to actually generating  kernels themselves for improvement of inference and training itself. So very kind of full circle virtuous cycle.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjv0p3e3spo9nmrj5spn3.jpg" alt="Thumbnail 1930" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1940" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fy260hf5w1vxltuxqi1cd.jpg" alt="Thumbnail 1940" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1950" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwsbnpaemjymhixzf1d79.jpg" alt="Thumbnail 1950" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj39nj1rpz7wuo6nadlw6.jpg" alt="Thumbnail 1960" width="800" height="450"></a></p>

<p>And that's why at the 2025 PyTorch conference we launched  actually a full stack all the way from how you author kernels at the lowest level at the metal and at the chip level  to how you orchestrate and run compute at the cluster scale level with a project called Monarch, as well as the RL layer with a project called Torchforge. And so  now we actually have a standard interface and a way for us to pull in the world of tasks, environments into our post-training and pre-training runs and improve our agents  at a level we've never been able to do before. And so this is really a new stack. This is the next generation stack that we see really driving the next wave of innovation.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1970" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F06y551v2l17osvp28geq.jpg" alt="Thumbnail 1970" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1980" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fubwjxtba2vl4y7x4kflw.jpg" alt="Thumbnail 1980" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=1990" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw4yqyheuio9qf4r7sohr.jpg" alt="Thumbnail 1990" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2000" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmfdv3p8e1m5fp4lny431.jpg" alt="Thumbnail 2000" width="800" height="450"></a></p>

<p>And so it's really exciting, I think, for us to partner with the AWS  team to bring these capabilities to Trainium. I think Trainium has had a long history of innovation and bringing alternative architectures of compute to developers, and now it's exciting to see native  PyTorch support. Joe's absolutely right. Agentic workflows represent where AI is heading, and having the right infrastructure in place  really matters. This is exactly the kind of ecosystem collaboration we envisioned when hardware integrates natively with PyTorch, everyone benefits. </p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2010" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgllboozjsdwszcv10q8i.jpg" alt="Thumbnail 2010" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2020" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq65grfc4x0sxuxvrgbsh.jpg" alt="Thumbnail 2020" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2030" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxwfm95v0cdi80jbfxjle.jpg" alt="Thumbnail 2030" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2040" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fofs2nm32tx9el11nlmu4.jpg" alt="Thumbnail 2040" width="800" height="450"></a></p>

<p>Researchers can freely experiment. Enterprises get real hardware choice without rewriting their pipelines, and the entire PyTorch community becomes stronger. We appreciate AWS's  continued commitment to the PyTorch infrastructure, just as AWS has contributed resources for PyTorch CI/CD on Graviton and GPUs.  They're now extending the support to include Trainium. The sustained investment ensures compatibility as PyTorch evolves. And PyTorch's strength has always been  its community and its ecosystem. With AWS Trainium now natively supported, that ecosystem just got more accessible to more developers. We're excited to see what the community builds,  and we look forward to deepening this partnership through the PyTorch Foundation and beyond.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foig3sr5k886ddzb29flg.jpg" alt="Thumbnail 2060" width="800" height="450"></a></p>

<h3>
  
  
  Empowering Performance Engineers: Neuron Kernel Interface and Open Source Commitment
</h3>

<p>We're very excited to see what machine learning researchers are going to build on Trainium with our new support in PyTorch.  The next class of developers I'd like to share with you that we hear from customers and we work with customers on are performance engineers. And certainly you heard some of the stories from Joe earlier about some of the work they've done to really optimize performance. To me, customers where they're at at this level, well, what performance engineers really need is direct access to the hardware.</p>

<p>And for that, we have the Neuron Kernel Interface, or what we call internally NKI, the Neuron Kernel Interface. This gives direct access to the ISA control so you can control things like execution, scheduling, memory allocation. We couple that as well with a profiler that enables you to get very detailed views into what executions are instruction, where you have bubbles in your compute to memory transfers as well.</p>

<p>In addition, we also have something called Neuron Kernel Library. This is a preset set of libraries that you can use yourself. It's code written on top of NKI that has all the common kernels necessary for transformers as well. You can take those building blocks or modify them for your own custom model itself.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2130" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5ukj2t682eipnjzgi1p5.jpg" alt="Thumbnail 2130" width="800" height="450"></a></p>

<p>We've been working with the academic community as well to give visibility into this lower level.  We've been hearing directly from researchers at the University of Illinois and researchers from Carnegie Mellon that they really enjoy that low-level access and are able to do the kind of experimentation and research that they haven't been able to do with other accelerators given that extreme low-level access.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2160" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F34xeg4eeltqc547hbzih.jpg" alt="Thumbnail 2160" width="800" height="450"></a></p>

<p>Finally, I want to mention how we're building this as well. We heard in the PyTorch presentation the importance of open source  communities. Very happy to announce today that this is also our approach for the Neuron Kernel stack all up. Today we have a number of components that are open source, and all the things I mentioned today from the PyTorch integration will be completely open source. The Neuron Kernel, both the Neuron Kernel Library itself as well as the interface and the compiler, will all be open source for you to look at, to make contributions, and to get visibility into how the software executes.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu2q64ogyd0gci4d7eu3y.jpg" alt="Thumbnail 2200" width="800" height="450"></a></p>

<p>This really gets rid of black boxes that are there today as you really understand your stack end to end. So that open source foundation will be very critical for us to work and collaborate within the  community and hear from you how we can make it better and give you visibility about how the software is running on the machines.</p>

<h3>
  
  
  Decart AI: Real-Time Video Generation with Frame-by-Frame Diffusion Models
</h3>

<p>Speaking of performance and great applications, I would like to invite now our next customer to talk to you about the fascinating applications they're building on Trainium. So I'd like to invite to the stage Orian Leitersdorf, the Chief Scientist for Decart AI. Thank you. Okay, all set. Thanks for having me on stage. Really exciting to be here, and I think the best way to explain what Decart AI is is to start with a demo because demos are fun.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9py0zs3toki7zqfdtype.jpg" alt="Thumbnail 2260" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2270" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fihmqtu2fqz5f2hxxsq4s.jpg" alt="Thumbnail 2270" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2280" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foctqtklja21p1d897t0z.jpg" alt="Thumbnail 2280" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2290" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fl2n89rolk3uanwff6pv6.jpg" alt="Thumbnail 2290" width="800" height="450"></a></p>

<p>So here we see a demo of Decart. You see me in real life, and that in real  time is transforming me into a lizard. I know it's a bit choppy. That's the Wi-Fi here. Usually it's much better FPS. What else could I do? What if I want to turn to anime  style, again happening in real time. What if instead, maybe, you know, it's a bit warm in Vegas sometimes. I want to go into a more cooler weather,  can transform the entire world, both the background, myself, everything else. Or maybe I just want to go to the jungle instead of staying in this conference  room. Finally, maybe, you know, because we're in a conference, I should be wearing something nicer. How do I give myself a tuxedo?</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F321vtf1ajibrl1dg31ca.jpg" alt="Thumbnail 2300" width="800" height="450"></a></p>

<p>Now what we just saw  is essentially what I've been working on at Decart, which is take video models which are usually offline. Usually it's, for example, text-to-video or image-to-video models. You give them a textual prompt, you wait two to three minutes, you get back a ten-second clip. What if instead of that, we optimize the video model to the point where we could run it at one-to-one speed, meaning generate a ten-second clip in ten seconds, and then the next step after that is not only generate a ten-second clip in ten seconds, but do it frame by frame, meaning it actually becomes interactive.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2360" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F734kdi1inf8c2t80kfsc.jpg" alt="Thumbnail 2360" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feoraapze3vddrpfwnd0t.jpg" alt="Thumbnail 2370" width="800" height="450"></a></p>

<p>Instead of just being offline where you give me a prompt and then three minutes later you get back a video, we can do cool stuff like this, condition it on the video input from the user, and then in real time it can generate a different video based off that input. And that's just one of many applications. So for example, we released this model a few weeks ago. We already got lots of traction on Twitch, people on Twitch turning themselves into different  skins while they're streaming. Lots of applications for gaming as well, where you can now essentially skin a game in a matter of seconds with any prompt  because this is a video diffusion model. It's not just trained for a few specific things. You can just make up any prompt that you want, turn the game into that in real time.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2380" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frclomgl0bm8lmp5bqodt.jpg" alt="Thumbnail 2380" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F464jly3o76yghc5zc8cl.jpg" alt="Thumbnail 2390" width="800" height="450"></a></p>

<p>Lots of applications as well for retail. You  could change specific objects, for example, in a movie, or you can do virtual try-ons where you take an image from Amazon and then you want to see how that looks like you're wearing it.  And at the end of the day, this also is very similar to world models, models that simulate the entire world based off of interaction from a keyboard, from a robot, and help us train next foundational models for robotics.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2410" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feww6c7wcp2fhzg2kw34u.jpg" alt="Thumbnail 2410" width="800" height="450"></a></p>

<p>Now these models that we developed at Decart essentially come down to  three fundamentally different capabilities. One is the fact that we're doing this on a frame-by-frame basis. We're generating one frame at a time instead of just generating a clip all at once. That gives us the interactivity we just saw. But on the opposite side, it means that we have a very strict latency constraint. Similar to large language models, we have a very strict latency constraint here, and it's even worse because when we run diffusion models, the diffusion models have to take a few iterations of the entire transformer per 50 milliseconds that you want to generate in the frame.</p>

<p>The next thing is that we took these models and ported them over to train on Trainium3. As we're going to see in a second, we're able to get to a staggering MFU of 80%. That's something very unprecedented for us, and it comes down to lots of fundamental design choices in the training architecture. We'll talk about them in a second that enable us to get very high utilization of the tensor engines.</p>

<p>The last thing about our models is that one of the main challenges from a research perspective, which is what makes these models very difficult to train, is that once you start generating over long durations, the model starts to drift and explode. It happens because when you generate frame by frame, each frame has additional noise compared to the previous frames. Over time that begins to explode, and that's why most video models today can only do very short clips. Here we have innovation that lets us train a model that can generate infinite duration without exploding.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2500" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh3844nifite071xtzwyl.jpg" alt="Thumbnail 2500" width="800" height="450"></a></p>

<p>Okay, so before we dive into exactly how we got this to work on Trainium, a quick intro to what  image-to-image diffusion models are, then video-to-video diffusion models, and then how our models work. Image-to-image diffusion models essentially have a diffusion process in the middle. It begins with a frame that is completely noise, and through several iterations it cleans that noise to generate a new frame. What we do here is we take our input image, so for example, that would be my image from the camera here on stage, encode it using a convolutional neural network, take that through several iterations of diffusion steps where we're conditioning the diffusion on that input from the user, pass it through a decoder, get an output image, and essentially after training this pipeline with lots of different examples of edits, we can get a model that given any input image and any prompt can edit that image.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2550" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjzsorzsd1sweerhd2a88.jpg" alt="Thumbnail 2550" width="800" height="450"></a></p>

<p>How does video-to-video work? Well, our models don't do it only on a single image. They need to do it on an entire video and stay consistent over time. So the idea is essentially you have a KV cache between the different models where you share the KV input from the previous frames that the model generated with the current frame that's generating. That enables you to get a context window, which means that the model is consistent over time while also being able to edit the video in real time.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2580" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqswuu9sq2euz2q3y7s47.jpg" alt="Thumbnail 2580" width="800" height="450"></a></p>

<p>Now  the reason that these models are a very good fit for Trainium is very evident in the architecture of Trainium. What we see here in the architecture of Trainium is we have a very small number of NeuronCores, which means that the compute is very centralized. It's not spread out across lots of different streaming multiprocessors. It's centralized in a few small locations. We have, in addition to that, a very large centralized SRAM in each one of these NeuronCores, and that enables us essentially to avoid HBM.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpvz48mh306yqfxrwq31m.jpg" alt="Thumbnail 2610" width="800" height="450"></a></p>

<p>One of the biggest fundamental challenges we had previously was the high latency of going back and forth to HBM, hindering our ability to get very low latency models. By essentially keeping activations in SRAM the entire time, something that's not possible to do if the SRAM is very small and distributed, we're able to completely avoid HBM and keep it in the SRAM right next to the tensor engines for the entire model. The next thing is that we're able to get very highly parallel collectives. A very unique feature of Trainium is that the collective units are split from the main computational units, and that means you can run collectives in parallel to the main computation without it even hindering or affecting the performance of the computation itself. We exploit that here with tensor and context parallelism in order to overlap those collectives for KV cache and other operations without having any impact on the performance.</p>

<p>Next, to do something like this, we really need fine-grained control, and that was possible with NKI. NKI gave us the very low-level ISA, which means that we can understand the hardware. We can model what is the best way to map our model architecture to that hardware and then implement that directly with NKI as we mentioned. Lastly, I think one of the most helpful aspects of working with Trainium is the profiler. The profiler gives you a very low-level timeline of exactly what's running on each one of your engines at any point in time, and that means you can very easily optimize the performance to reach peak utilization.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2710" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frz41jjqaqmbzu8y4h1pr.jpg" alt="Thumbnail 2710" width="800" height="450"></a></p>

<p>So what does all this lead to?  Well, for our upcoming Lucy 2 model, we see that Trainium3, which we've already integrated with our model, is able to get to 2x higher performance than Trainium2, and Trainium2 is 2x higher performance than state-of-the-art GPUs. Essentially, this is an enabler for our workload because it means that we can now get to blazing high FPS that open up new applications such as gaming, VR, and everything that needs very low latency and high FPS for it to be reasonable. This is only possible with Trainium3.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2760" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flqfjka60agm2gdk21fko.jpg" alt="Thumbnail 2760" width="800" height="450"></a></p>

<h3>
  
  
  Closing: Community Innovation and the Road to Trainium4
</h3>

<p>Thank you, and I'll hand it back to Colin. Thanks very much, Orian. Fantastic demo, fantastic work, and great partner. Thank you for all your work.  So we've seen today that innovation is necessary across all stacks. We need not only chips and servers and UltraServers, but also the Neuron developer stack, along with partners like PyTorch, Poolside, Decart, and you as well.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ndujahk9fwofbwa3obd.jpg" alt="Thumbnail 2790" width="800" height="450"></a></p>

<p>I want to thank my two guests, Joe and Orian, for joining us today. It's going to require innovation across the community as well as the partners, and I invite you to join us.  I have a couple of QR codes, and I encourage you to take a picture of these. These are places where you can get started to learn more about the Neuron stack and to learn more about Trainium as well. This will give you the links to everything we've just announced today and various resources for you.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2810" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fin23grsihp5vec8owoke.jpg" alt="Thumbnail 2810" width="800" height="450"></a></p>

<p>I have one other aspect to mention too. Innovation never stops. We're also announcing today AWS Trainium4,  which is something that we're currently working on now. You've seen from the previous history of the silicon and the other products we have, Graviton and Nitro, that innovation requires a commitment from a company and from a set of people over time to innovate generation over generation. We're having that same commitment for our machine learning accelerator chips as well.</p>

<p><a href="https://www.youtube.com/watch?v=5SgZ3zRJFAY&amp;t=2830" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8e9mj16lji5gvwespeew.jpg" alt="Thumbnail 2830" width="800" height="450"></a></p>

<p>If you haven't already, I put the slide here so you can see the number of sessions that are here today. You can take a picture of the QR code to get them. All of these will be available online, and I encourage you to look at that. With that, I want to thank you very much for your time today. Thank you.</p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

