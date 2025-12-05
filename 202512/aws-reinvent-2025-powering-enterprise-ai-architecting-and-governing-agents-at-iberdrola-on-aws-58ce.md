---
Title: AWS re:Invent 2025 - Powering Enterprise AI: Architecting and Governing Agents at Iberdrola on AWS
Description: 
Author: Kazuya
Date: 2025-12-05T21:47:26.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 - Powering Enterprise AI: Architecting and Governing Agents at Iberdrola on AWS</strong></p>

<blockquote>
<p>In this video, Sergio Merchan, Unai Bermejo, and Joseph Santamaria from Iberdrola and AWS discuss how Iberdrola, Europe's largest energy company serving 100 million customers, is deploying agentic AI across its operations. They explain Iberdrola's multi-region Amazon Bedrock deployment strategy across Spain, UK, US, and Brazil, ensuring data sovereignty while enabling agent reusability. Three IT operations use cases are detailed: ServiceNow change request assistance, networking incident enrichment, and template selection chatbots, all built using Amazon Bedrock AgentCore runtime with LangGraph and Claude models. The architecture leverages MCP servers, PostgreSQL with vector capabilities, and a multi-account strategy for business isolation and scalability. Key benefits highlighted include framework-agnostic development, cost-effective serverless compute, auto-scaling, and zero cold starts. Future challenges around agent reusability and governance are discussed.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/hKlX4xwNoKY">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1617moqwcwrje33g44n0.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<h3>
  
  
  Introduction to Agentic AI and Its Transformative Impact on Enterprise Operations
</h3>

<p>Hi everyone. I'm Sergio Merchan, Global CIO for Iberdrola, and I will explain to you who we are and why we are leveraging the latest technology. My name is Unai Bermejo. I'm a Global AI Expert Engineer, and I will show you how we are architecting agents at Iberdrola today. I'm Joseph Santamaria, responsible for the technical teams within AWS that work with energy customers in our energy vertical. Today we'll talk a little bit about our agenda.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=30" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs3dfmhohsbgg1pcoo04x.jpg" alt="Thumbnail 30" width="800" height="450"></a></p>

<p>We'll start with an overview and a refresher on the AWS solutions for agentic and AI in general, with an emphasis on agentic AI. From there, Sergio will talk about why generative AI is important in energy, what Iberdrola is doing, and their strategy. Unai and I will dive into what they're actually doing, the use cases, and what they've learned. We'll discuss why they're doing it, how they're doing it, and what worked and what didn't work. This is a learning conference, so we want to make sure we share with the audience what you've learned. I think we'll have a great session.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=70" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fji7gxx92f1t6cb0qqflq.jpg" alt="Thumbnail 70" width="800" height="450"></a></p>

<p>Let's talk about what agentic AI is. We've been on our AI journey for decades, and we're now at the stage of the era of agentic AI, which is really the ability to create autonomous systems based on agents that allow you to automate and perform workflows in a way very similar to how humans would do it. Agents will plan activities. They have logic, and they'll be able to adapt to changing conditions. They'll be able to reason, and that is going to have the ability to really automate very complex systems and workflows that up to now have not really been accessible with more traditional AI.</p>

<p>This is going to impact every process, every customer interaction in your companies, every product, every service you offer is going to be impacted by agentic AI. This revolution and transformation will happen. It will first happen progressively, but it will also happen quickly. It will happen progressively because we have to earn trust that the agents are controlled and governed. Iberdrola will talk a little bit about how they're doing that.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=140" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs4wcvg81vzf19m4kigvz.jpg" alt="Thumbnail 140" width="800" height="450"></a></p>

<p>We're seeing companies start to take agents to automate simple tasks with a narrow focus, very specific interactions with humans, and quite a bit of human oversight going on. But now as companies' ambitions grow, and Iberdrola is a great example of that, they're moving towards the right of the chart where they start to take workflows and specific business problems that can be solved with agents, accelerated with agents, or where the cost can be lower with agents. They're starting to automate that, and eventually what we're going to get to is where you have systems and swarms of agents that either deterministically or through other agentic coordination get orchestrated to really solve very complex problems and end-to-end solutions.</p>

<p>The reason I also mentioned this will happen quickly is because the price is enormous. We're seeing companies adopting agentic workflows where things that used to take years now take months, maybe at times weeks. Things that used to cost millions now cost thousands. We're seeing a 10x productivity gain. So the price to get this done is enormous, and that's why this is going to happen quickly because companies are going to be trying to drive and realize that value.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=220" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3x6cdeuz4fjtuw88keoz.jpg" alt="Thumbnail 220" width="800" height="450"></a></p>

<p>There's no surprise that Gartner anticipates that 33 percent of all applications will include agentic AI in the next two to three years by 2028, and that 15 percent of all the decisions that are made in a company will be automated through agents also with the same timeframe by 2028. Our aspiration and goal here at AWS is to make that possible and to help all of you build the most useful, well-managed, controlled, secure, governed, and cost-effective agents in the cloud.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=250" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fchgwrqxih27w7dql3vgq.jpg" alt="Thumbnail 250" width="800" height="450"></a></p>

<h3>
  
  
  AWS AI Stack: From Applications to Infrastructure for Building Intelligent Agents
</h3>

<p>To do that, we obviously have an AWS AI stack that is composed of three tiers. The top tier has our applications, and they have a broad range. We have applications like Kiro, a code assistant for developers that will help you accelerate your SDLC. We have AWS Transform that helps you modernize whether it's .NET to Java or modernize within Java. There are a bunch of transformations that will help you keep your tech stack modern and up to date.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu1krlohj1ehqugfk2go1.jpg" alt="Thumbnail 260" width="800" height="450"></a></p>

<p>We also have business solutions like Amazon Connect, which is a call center solution that helps you manage customer interactions in different channels from voice to chat and so forth, and embeds a number of agentic capabilities already.</p>

<p>For those of you who want to build agents but prefer to do that in a managed service, we offer Bedrock. Our core product within Bedrock for agentic building is Agent Core, which we released about a couple of months back and is now generally available. It has really great capabilities to manage memory of those agents, to have observability of what the agents are doing, and to manage how the agents are interacting with other tools through a gateway. It has a runtime that is framework independent, so you don't have to worry about what agentic framework you're using for development.</p>

<p>We're bringing a bunch of first-party tools, and you'll hear today how Iberdrola is using Agent Core and how each one of these components is being used to bring agentic solutions to fruition. At the bottom, we also have infrastructure. What we're seeing is that more and more customers are willing and eager to unlock the value of the data they sit on. They have decades of data—it could be great data, asset data, or subsurface data. Generic LLMs, while very useful, don't have the benefit of that information, so customers are starting to fine-tune and be more ambitious in terms of how they embed that data beyond RAG or a knowledge base into their workflows.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=420" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedage1phvkyvvrjp0pcw.jpg" alt="Thumbnail 420" width="800" height="450"></a></p>

<p>For that, we're seeing more and more customers starting to train, and we give you the infrastructure to do that. We announced yesterday Amazon Nova Forge, which gives you the ability to embed your own data through different pre-training, post-training, and training at different steps when you build a Nova conditional model. We give you the ability to train a small language model, a specialty language model on AWS in a cost-effective and efficient way through the different epochs.  With that, let's dive into a little bit of generative AI in the energy sector and what's going on at Iberdrola. I'm going to invite Sergio to come up.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=460" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsgvnu6sq3hee5zca5v4c.jpg" alt="Thumbnail 460" width="800" height="450"></a></p>

<h3>
  
  
  Iberdrola's Energy Transformation Challenge and AI-Driven Value Chain Excellence
</h3>

<p>Thank you, Joseph. Hi everyone. Now we are going to explain how we are enabling all this technology and all these capabilities to support our transformation. I think you mentioned the challenges that we're having in the energy sector. Basically, as you can imagine, all the growth in terms of energy is this electrification, and it's putting a lot of challenges to energy companies, including Iberdrola.  The challenge I was just mentioning is electrification and growth in terms of energy consumption. It's putting a lot of pressure on the energy sector, including Iberdrola. We want and need to create more sustainable, cleaner energy. To tackle these challenges, we need technology to transform the way we are operating today. We need to be excellent in every single piece of our value chain—excellent in our operations, excellent in the way we generate and create energy, excellent in the way we manage our customers, and excellent in the way we manage our corporate activities. That's why we need technology enabled by AI.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=510" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F66z7jb6e4u7tr03izyt3.jpg" alt="Thumbnail 510" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=550" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F97pmf6quuj6fkz0geq1e.jpg" alt="Thumbnail 550" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0ps68rp08wulhlxh1xqa.jpg" alt="Thumbnail 560" width="800" height="450"></a></p>

<p>Let me explain a little bit about who we are.  Iberdrola is the largest energy company in Europe and the second largest company worldwide by capitalization, with 120 billion euros of capitalization. We serve more than 100 million customers. We have an asset base of 160 billion euros and have invested 150 billion dollars over the last two decades. We are a very large company covering the full chain from generation, transmission, distribution, and retail. We have presence in many parts of the globe.  We have presence in Spain, the UK, the US, and Brazil.  We also have presence in other countries in Europe and outside Europe, like Japan and Australia. We have a strong global presence, and you can imagine the challenge we have in responding to this electrification, energy demand, and the need to create cleaner and sustainable energy worldwide.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=590" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftnh5ggzdo7estzyyeblo.jpg" alt="Thumbnail 590" width="800" height="450"></a></p>

<p>Our ambition is very clear.  Our ambition is to put AI at the center and enable AI capabilities to transform the way we are operating today and the way we are working across the lifecycle. This is not just an incremental approach.</p>

<p>We focus on specific use cases and specific processes that allow us to capture some incremental value, but we are moving toward a full transformation approach. We really take one process, imagine the process, and transform it by rethinking it. We imagine and complete the process using AI as the central enabler for transforming the process. For this, we base ourselves on a specific model that brings value or serves as the vehicle we put in place to capture the massive amount of value from the technology. This is not just incremental change; it is real, full transformation of processes across the value chain.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=650" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzdh328u4tka1nnv5v32n.jpg" alt="Thumbnail 650" width="800" height="450"></a></p>

<p>We are seeing the capabilities  of AI across the value chain. For example, we use AI to help us define where to position our wind turbines in order to optimize energy generation. Through operations, we also put in place agents or assistants to help our field workforce operate, perform inspections, and maintain the turbines. This helps them achieve quicker resolution, improving downtime and the way we create and generate energy. We have one of the main use cases here, but we also have use cases in retail and in the call center area, helping with assistance to our call agents so they can provide answers quicker and with higher quality.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=730" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4fv07ruxynffy8grzwtq.jpg" alt="Thumbnail 730" width="800" height="450"></a></p>

<p>The same agents are deployed across corporate areas such as legal, internal audit, administration, control, and financing. We have capabilities to deploy agents across the organization, and this is how we see we need to transform in order to be better prepared to respond to the future of energy. What are the models that we put in place?  One of the main key success factors is going to be the platforms and the technology, which I will explain in detail.</p>

<h3>
  
  
  Building Success: Competence Centers, Platforms, and Multi-Region Deployment Strategy
</h3>

<p>We are moving from traditional AI assistance to agentic AI, which is going to be the next wave. We are building our best-in-class platform and agentic framework leveraging AWS services. We also have another key success factor, which is our competence centers. We have been building competence centers across the group in a hub-and-spoke model. One of the things we are doing is standardizing our method to deliver solutions and products to our business areas.</p>

<p>We serve in so many countries and have too many businesses that we need to respond to. The first thing we put in place is to standardize and build something that is standard and common for everyone as much as we can. This not only accelerates but also creates speed and time to market. We build a center of competence that includes all the capabilities we need to deploy the solution. We do not just move from one area to another; we include architects, engineers, project managers, operations people, and security people. We put everything together into the center to create a one-stop shop, a factory concept where we deliver solutions end-to-end from discovery, engineering, architecture, operations, and security.</p>

<p>The second point is platforms, and this is also one of the key success factors. AWS is not just providing capability in terms of AI resources or services. We create a platform with all the different building blocks to embed this framework. When we deliver an AI solution, that solution is secure and observable so we can monitor it. We can ensure that we are delivering something that is secure, protected in terms of data, and responsible. We create all the framework we need to deliver something to our business areas that is creating security, putting observability, putting monitoring, putting auditing, putting logging, and putting everything in.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=830" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7258vuiqy7exyw1r6ce2.jpg" alt="Thumbnail 830" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=890" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzjitjkuiyyymbfoixoe3.jpg" alt="Thumbnail 890" width="800" height="450"></a></p>

<p>Platforms are again another success factor, and the last one is competence centers. We have a hub-and-spoke model where we deliver the same concept across the organization no matter where we are. We operate in the same way, leverage the same method, the same teams, the same capabilities, the same layers of services, and the same platforms across the group.  I think that is another area of value because at the end with this concept we can create solutions that are quite standard, flexible, modular, secure, and in a very time-to-market way. Being this kind of partner that we want to be with the business, we become  a technology leader, not just look at IT as standard business as usual activity.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F59c5a8axceht98xhpj92.jpg" alt="Thumbnail 930" width="800" height="450"></a></p>

<p>Now I will explain the way we are delivering agentic capabilities in the company. Thank you. Today I will walk you through the global deployment of agents with Amazon Bedrock and Corati Verdola, and we will see some use cases later. </p>

<p>This diagram represents the way we are scaling AI and IT initiatives across the entire Iberdrola group. If you turn your attention to the left side of the screen, you will see our global footprint. We are deploying Amazon Bedrock in our four core major markets: UK with Scottish Power, US with Avangrid, Brazil with Leonarchia, and Spain. With this regionally deployed platform, we ensure data sovereignty and ensure that the agents stay where they belong, complying with local regulations and standards while reducing networking costs.</p>

<p>The interesting part is on the right side. This is how we are structuring the platform. We have a multi-account strategy in which we have three environments: production, pre-production, and development. For each of those environments, we have a strict verticalization by business. We have an account for retail, for networks, for corporations, for renewables, and for IT. With these regions, companies, and environments, we ensure three things mainly.</p>

<p>First, we guarantee scalability in each region. This means that where there is a surge in smart meter data in Leonarchia, for example, or a new commercial campaign in Spain, we scale independently. Our agents are isolated, scaling independently with the powerful serverless approach of Amazon Bedrock. Second, we ensure business isolation by design. This means we are securing each business line and ensuring that we limit the visibility and sharing of agents.</p>

<p>Third, and probably the most important thing, we are ensuring reusability and discovery within the same business. This means that if the networks teams in Spain build a great agent, by design, we are allowing them to share it and reuse it because of the platform deployment strategy. The good news here is that we are not just building the platform; we are also the first adopters of it in IT.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1070" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd7yxz0i5gh4bmz4is452.jpg" alt="Thumbnail 1070" width="800" height="450"></a></p>

<h3>
  
  
  IT Operations Use Cases: Agent Architecture with Amazon Bedrock and LangGraph
</h3>

<p>Let's jump to the first three use cases of agents with Bedrock.  We chose IT operations as our first battleground. We have three use cases, simple use cases that are delivering value now in our operations. The first one is IT support change requests. We are working in ServiceNow and improving the way users work with changes. In a utility company like ours, making changes to systems is high stakes. You need to comply with a lot of compliance and standardization, and this often brings a cycle of rejection and delays because the petitioner is not fulfilling the drafts correctly.</p>

<p>We now have a group of orchestrated agents that act as gatekeepers and help the user fulfill those drafts. This reduces delays and resource waste. The second use case is about enrichment of networking incidents. We want to reduce manual effort and improve time to market by boosting tagging, assignment, and resolution speed, having a swarm of agents that are specialized in specific things related to the request. Third, and probably the most simple one, is related to the first one. It's a chatbot-like agent that is connected to a corporate knowledge database that allows the user to pick the right template for the change in ServiceNow.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi7cdcvm366p6c0dbvy5r.jpg" alt="Thumbnail 1170" width="800" height="450"></a></p>

<p>We have more than 200 change templates, so it is not easy to pick the right one. Now we have this RAG architecture to help users do so. This is the architecture.  First, I would like to thank Accenture and our partners, necessary partners and areas in Iberdrola for achieving this milestone in record time. We are not only delivering value in these business processes, but we are also building the foundations for the future, preparing the platform for what is coming next.</p>

<p>We will review the architecture at a high level now, and then we will zoom in on the use cases and see how the agents are working under the hood. First, I would like to highlight that we have one agent called runtime for each of the use cases.</p>

<p>This means that we have decoupled the costs, the development life cycle, and the escalation by splitting the use cases into runtimes. Each of the runtimes has a Docker image in the ECR, so we can deploy with continuous integration pipelines and continuous deployment pipelines. These runtimes are running in Amazon Bedrock AgentCore agents that are consuming models in Bedrock through LLM. This LLM is a layer of code that allows us to balance requests between models in case of unavailability or quota cannibalization.</p>

<p>We have guard rails in place to ensure that the models generate what they are supposed to generate and nothing else. The second thing I would like to highlight is that we are deploying the MCP servers also in Amazon Bedrock AgentCore runtime. This means that the use cases that are using the database, the RDS PostgreSQL database, are consuming it through this MCP server. We are also using an Amazon Bedrock gateway for reaching ServiceNow and writing the results of the agents where they are needed. Our front end in this case is ServiceNow, so ServiceNow is calling AWS Cloud to perform the agentic workloads.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1320" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fm3ch5xhfya2x3317n168.jpg" alt="Thumbnail 1320" width="800" height="450"></a></p>

<p>We have also a micro gateway there to ensure authentication and routing the request to the particular runtime. This is the high-level perspective of the architecture. Now we will review the first use case and how the functional details and technical details work. We call it Assist for Change.  Making changes to systems is difficult, and this is why we have a strict process to make changes to systems at Iberdrola, as most companies do. In our case, each change request passes through six phases, from draft where the requester has to fill out a change model to ensure that the information is in place, to close when the change is done and the teams start working on it.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj57owbyn0u9bnese4mye.jpg" alt="Thumbnail 1390" width="800" height="450"></a></p>

<p>We are deploying agents at the draft phase. This means that we are speeding up the change management process by assisting requesters. These templates that have to be filled have several fields and several rules, so we have expert agents on each of the fields and each of the parts of the templates. We can first validate and also propose fixed proposals so we can speed up the process.  The requester does this change request draft and pushes the button to start change when the information is filled, and then a workflow of agents is triggered. We are building small agents. We have a first agent that is determining which fields and contents must be checked using LLMs and tools available.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1460" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjsd4jyacqh1afq70bpil.jpg" alt="Thumbnail 1460" width="800" height="450"></a></p>

<p>Once we know which rules have to be checked, the content validator takes this input and basically validates fields and content based on the rule structure output. In parallel, there is a third specialized agent called a model agent analyst that validates the change model selection based on the justification. This is the main point that the third use case is solving, checking if the change model that is being selected is appropriate to the change that the user wants to do. If all is in place, there is a fourth agent in LangGraph that is going to propose fixes in case there is something missing or automatically transition the phase. We are putting an intelligent layer in this ServiceNow front end powered by AWS. </p>

<p>I will show you three diagrams like this. In the top part of the slide, we have the technical flow. ServiceNow is calling the micro gateway, the micro gateway is routing that request to the runtime, and the runtime has these LangGraph agents being run using Claude and using the RDS MCP server. If you turn your attention to the LangGraph agents, you see that we have a graph structure in which we are running the agents sequentially. First, the rule structure is triggered, the content validator takes that output and validates the contents.</p>

<p>At the same time, we are validating the aim of the model. If that is in place, the phase transition is done. In the bottom, we have the ETLs. We are putting ServiceNow templates and change models into an S3 bucket, and this S3 bucket is being used by a pod in Kubernetes to build the gold layer of the data that the agents are consuming through the MCP server.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft3w6t8ouujr8zt2dho4a.jpg" alt="Thumbnail 1560" width="800" height="450"></a></p>

<p>A good thing to know here is that we are using PostgreSQL with vector capabilities for the vectorization of the data. We also have some data models in place to ensure that we are versioning prompts, to ensure that we are versioning embeddings, and to ensure that we have all this lifecycle of the data in place. </p>

<p>The second use case has the same structure and the same strategy, but simpler. To help the user select the right model, we are putting a simple agent in the runtime. ServiceNow again calls the micro gateway in this case with different data, so the micro gateway can route the request to the appropriate runtime. This runtime is going to help the user in a chat-based interaction to pick the right model.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1600" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fr632o6nm6kooabdkytn0.jpg" alt="Thumbnail 1600" width="800" height="450"></a></p>

<p>We also have the same thing. We have the ETLs that are going in a regular way being loaded to the database. </p>

<p>The third use case is probably the most interesting one because it is the most complex one. We have the same strategy again with ServiceNow calling the micro gateway, the micro gateway calling the runtime. But in this case, the runtime has a swarm of agents. We have an orchestrator that is able to know which agents have to be triggered and when, depending on the information available in the incident.</p>

<p>So if the networking incident has to be tagged, the orchestrator will know it and will trigger the tag classifier. If we want to perform incident similarity, we will have the incident similarity agent in place, and so on. This is probably the most complex LangGraph architecture that we are deploying in these three use cases. Of course, we have again these ETLs from the S3 to the RDS to the golden layer and the PostgreSQL with vector capabilities to be able to consume the data.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1690" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh917urtt079b059mkry1.jpg" alt="Thumbnail 1690" width="800" height="450"></a></p>

<p>I didn't mention this, but we are using Claude models for the use cases. We are going with Claude 4.5 Sonnet, but we are flexible depending on the needs. So that is important too. </p>

<h3>
  
  
  Why Amazon Bedrock AgentCore Runtime and the Future of Agent Reusability
</h3>

<p>But why have we chosen Encore runtime and not other compute resources? First of all, it was really important for us that Encore is framework agnostic. We didn't have to upskill the teams to learn a new framework or to code in another framework. You just can put LangGraph code, expose it, and seamlessly deploy it with the MCP protocol. You just need to ensure that you have the FastMCP decorator of MCP tool and MCP run. You can also deploy the agent with HTTPS. We decided to do it with MCP, so if we want to reuse it in another use case, we have that in place, but you can do it with HTTPS as well.</p>

<p>Also, runtime is a really cost-effective compute resource. You are only being charged by actual CPU consumption, and we all know that agents pass more or less between 30 and 70 percent of the time calling LLMs and calling other tools. During this time they are waiting, we are not being charged. This is really good for economies of scale.</p>

<p>Thirdly, of course, Encore gives us session isolation so we can pause the session, make the agent stay full, and empower these human-in-the-loop loops and wait for agents and other agents or tools. In our case, we are not using that so much because the use cases are simple, but it is good to know that we have that available.</p>

<p>Also, it is a low-latency architecture. This means that there is no cold start. Maybe you come on Monday, the agent has been sleeping all the weekend, and you will have no cold start. You will have that ready in about 200 milliseconds. And also it auto scales.</p>

<p>If one of the agents or architectures are receiving more demand than expected, you don't have to worry about scaling that yourself or managing the infrastructure. The auto-scaling works like magic. It's really interesting for us that if we are running the agents in that runtime, we have so many managed services available for this kind of project. For example, memory allows agents to have soft memory so they can remember what is going on in the session, but also long-term memory so they can learn and improve over time. We also have a gateway for the tools, where we can put our tools into the gateway to enable discovery and to enable roles for access. It's really interesting, and we believe that it's difficult to find something similar to this as easily in some alternatives.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1870" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5mmgu6plby95tp7yxbz8.jpg" alt="Thumbnail 1870" width="800" height="450"></a></p>

<p>In the code, you just need to deploy the agents and ensure that you expose them with MCP or with HTTPS, and that's all. You can focus on the programming of the agents and forget about infrastructure.  Just to give you a code snippet of what is going on inside, this is for the Swarm Orchestrator I was talking about before—this orchestrator that enriches the incidents. We are using LangGraph, and LangGraph is all about graphs. We have a planner node that is going to decide which agents have to run, and we have a run_next node that, depending on that output, will run the agents in sequence. We could parallelize that too, but we didn't do that. Once the agents have been run, we have another node to assemble the results. It's as easy as that.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1880" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpp9cfagwto712amrtqsj.jpg" alt="Thumbnail 1880" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=1950" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnswh4ngszsurwq76w622.jpg" alt="Thumbnail 1950" width="800" height="450"></a></p>

<p>The difficult thing here is that you have to program what is inside the nodes, of course, and I don't have code snippets for that, but you could imagine that they are all calls to the LLMs and some business logic.  You can program in Strands, LangGraph, Google ADK, or whatever you want, put that on the runtime, and it works. What's next?  This is for me the most important part of the presentation because these use cases are delivering value to these business processes. We are preparing the platform so businesses can work with Amazon Bedrock AgentCore, which is our maturity tier and also our role as IT to transform.</p>

<p>We foresee some challenges of not reusing the agents or not thinking big when building an agent. If we build a great agent and we cannot reuse it, we have to program it again and develop it again to put that on another use case in another area. We of course face increased costs due to the need to develop existing capabilities, but we also duplicate business logic, which can lead to unpredictability and unexpected behaviors. We also have this risk of duplication of sources of truth. This happens also in microservices and in traditional software, but with agents especially because we have these capabilities of generalization that maybe traditional software don't. So we have an opportunity to fix these three things.</p>

<p><a href="https://www.youtube.com/watch?v=hKlX4xwNoKY&amp;t=2040" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fotfcolu0h0poz8lb5k9j.jpg" alt="Thumbnail 2040" width="800" height="450"></a></p>

<p>We believe that Amazon Bedrock AgentCore will be part of the solution or the solution to these problems, but it's also a cultural transformation of how we think about agents and transform the business processes. For me as a technical person,  there are three things I want to do to use an agent. First of all, we have the first scenario in which we have a good agent that is successful, and we want to reuse it across the organization, maybe in other business processes or maybe by other people. The first scenario is when the original owner retains the ownership and has to authorize new users or new areas to use it or to integrate it in other business processes. This part is easy, right? You just need to put some network in there, maybe some roles, and you are ready to go.</p>

<p>But there are two other use cases that for me are probably common or will be common when you have an agent. We normally want to introduce tweaks to it because we know it works. It has access to some tools, it has access to some data, and it works fine, but for my personal business case, I need something else, some extra detail. For that, we need to work with personas. The second scenario is easy when we have the owner keeping the ownership of the agent and we ask the owner to tweak or to improve that agent so it works for me too. That's about versioning and governing agents in a registry.</p>

<p>But this has the problem of making the owner work for you, and that's not easy sometimes. So we need also the possibility for these agents to be able to share the code, to serve the tools, to serve the infrastructure behind so we can ourselves, with our budget and with our development team, adapt it to the specific use case using the original agent as the base. These are the three things that we want to achieve. We have plenty of time now for questions and details if you want to ask something. I think that's pretty much it. Thank you.</p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

