---
Title: AWS re:Invent 2025 - Accelerate product development lifecycle with a product digital twin (IND371)
Description: 
Author: Kazuya
Date: 2025-12-05T22:01:35.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 - Accelerate product development lifecycle with a product digital twin (IND371)</strong></p>

<blockquote>
<p>In this video, Steve Blackwell from AWS explains how manufacturers can accelerate product development from 5-7 years to 2 years using a product digital twin. He demonstrates how the Manufacturing Data Flywheel connects siloed data across design, manufacturing, and service phases through a Knowledge Graph built on Amazon Neptune. The Digital Engineering Framework enables manufacturers to create a product data fabric that links requirements, CAD models, PLM systems, MES, and MRO data. Using Amazon Bedrock and generative AI, engineers can query this knowledge graph in natural language to trace defects back to their origins and gain insights across the entire product lifecycle, transforming how companies develop and maintain products.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/YdV1Hj5YdB4">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcp555oszx1y0rmd1ujr8.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=30" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5iuhu1b3rked3z4i2a13.jpg" alt="Thumbnail 30" width="800" height="450"></a></p>

<h3>
  
  
  Breaking Down Data Silos: The Manufacturing Data Flywheel Challenge
</h3>

<p>Thank you and welcome to IND371, Accelerate Product Development Lifecycle with a Product Digital Twin. My name is Steve Blackwell. I lead our product engineering and services at the Centre of Excellence here at AWS, looking after our automotive and manufacturing customers. For the next 20 minutes, I want to talk about how we're seeing manufacturers start to look at their data and how they can actually use that data to  address product development. This is realized through what we define as a digital twin, or in some instances, what we call a product digital twin.</p>

<p>What we're seeing is that many manufacturers face the same industry challenges. They need to differentiate their products in a competitive market and increase their speed of innovation, bringing new features and capabilities to market as quickly as possible. A prime example is in the automotive industry, where traditional OEMs now compete with Chinese OEMs like BYD, who can develop new vehicles and features within a two-year period, whereas traditional OEMs had a development lifecycle of five to seven years to build a new platform and launch a vehicle.</p>

<p>We're seeing manufacturers reduce these new product introduction cycles from five to seven years down to two years by leveraging software and AI to accelerate development. Specifically, they're reducing the total cost of development. The faster they can develop a product, the lower the cost to develop it, which means they can generate more margin when the product goes to market. We're seeing this across all types of manufacturers, from consumer electronics manufacturers producing robot vacuum cleaners to aerospace manufacturers building aircraft engines and aircraft.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzliivwggqwi3l979cfft.jpg" alt="Thumbnail 150" width="800" height="450"></a></p>

<p>When we talk about product data, that data resides across different parts of the manufacturing value chain.  Engineers develop and design products using their applications and records. This work gets handed off to the manufacturing team, who work with suppliers to source raw materials and parts to manufacture or assemble the product. They use their own systems, such as manufacturing execution systems and supplier systems. Finally, once the product is manufactured and used by consumers, other systems maintain and service those products, including MRO systems and after-sales service systems.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=250" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzds9a7fp2f34vcos593w.jpg" alt="Thumbnail 250" width="800" height="450"></a></p>

<p>Traditionally, these three different areas of the value chain—as designed from the R&amp;D perspective, as manufactured or as built from the manufacturing operations perspective, and as serviced once it's in the field—have sat in data silos. What we're seeing now with manufacturers is that they can see value in taking all that data and connecting it as part of building a digital thread, specifically focused on the product,  so they can start harnessing and using that data.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=300" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcw0j5zwf3ifccg1kpojb.jpg" alt="Thumbnail 300" width="800" height="450"></a></p>

<p>A great example is that many manufacturers are now creating connected products with telemetry information flowing in from how those products are being used in the field. This includes characteristics of how owners use the products or events occurring on the product, such as motor failures or low battery charge. There's a huge amount of information coming through from connected products in the field. This information about how the product is actually being used is really valuable to the engineers designing the next iteration of that product. </p>

<p>The usage characteristics—how that product is actually being operated and how it's been maintained—can be used to feed into the design process. Rather than purely simulating scenarios, designers can use real-life data to feed into their analysis and design. This approach transforms how manufacturers develop products by grounding their work in actual operational data.</p>

<p>A similar example involves quality issues created by service events or product failures in the field. When a quality issue or service failure occurs, manufacturers need to determine whether it resulted from a manufacturing defect, a part that is out of specification from a supplier, or an engineering defect that is not meeting specification and quality standards. To understand exactly what the quality defect is and where it originated, manufacturers need to link this information together.</p>

<p>We realized this through what we call the Manufacturing Data Flywheel. This approach enables manufacturers across the product digital thread to connect multiple different data sets together, interrogate that data, and gain insights to drive increased value in their products—how they develop them, how they manufacture them, and how they maintain and run them. These are examples of how product data is being used across the manufacturing value chain.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=400" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F37rarz0fxqn04roif0qy.jpg" alt="Thumbnail 400" width="800" height="450"></a></p>

<h3>
  
  
  Building the Product Digital Twin Through Knowledge Graph Technology
</h3>

<p>Now let's look at how we actually implement and start doing this.  If we look at the product life cycle of the data, I've simplified this down. We have the sales teams that conduct market analysis, define requirements, and perform sales forecasting. Then the engineers design the product based on the requirements that the sales and marketing teams provide. The product is then manufactured and maintained in a continuous circle of development as the product lives over multiple years.</p>

<p>On the outside, there are multiple systems of records that are used and interacted with as part of that whole life cycle. The sales team uses a CRM system to track sales forecasting and customers. As they take that market analysis, they create requirements they want to feed to the engineers to design against, so there is a requirement system. Most products today are not just hardware; they are hardware, software, electronics, and may have back-end cloud services that support them, so there are multiple streams of product development that come into play.</p>

<p>In the design phase, you may have CAD models, PLM systems, simulation data, and if you are doing software development, you have Application Lifecycle Management systems, electronics, and EDA applications. There is a huge amount of data. In manufacturing, we have the MES system that records how the product is being manufactured and assembled. When the product is out in the field, we have connected systems and MRO products that track maintenance and how it is being serviced. There is a huge amount of data that we need to bring together to build this product digital twin.</p>

<p>How do we do this? We do not do it by taking all those systems of record and bringing them in to create another system of record. We achieve this through what we know as the Knowledge Graph. The Knowledge Graph allows us to take those different data sets and create relationships between the different data sets and the entities or records across the whole life cycle.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpj6iqpdx4y5s7it5bvzu.jpg" alt="Thumbnail 560" width="800" height="450"></a></p>

<p>As an example, we have a requirement  that has been created by marketing that defines a particular feature of that actual product that is related to the product itself. When that product is then manufactured, it becomes part of the bill of materials, but specifically when it is actually manufactured, it becomes part of the MBOM, the Manufacturing Bill of Materials. When we manufacture it, there is a process plan that specifies how we take that bill of materials or the parts and materials and perform the operations to get there. That flows through the actual factory floor through routing. As part of that, there is a production order that specifies how many actual units to produce.</p>

<p>You can see the complexity of all these different types of data, but that different types of data is actually linked together. A product is going to have a part number. That part number is going to have multiple subpart numbers. When you do operation one, you're going to take part number 1 and assemble it to part number 2. So you can see there are actually relationships between even different types of data, and this is really where the knowledge graph comes in.</p>

<p>It allows us to create the relationship between the different types of data sets on common parts. So a defect at the end may not be directly associated with a requirement, but we can actually trace through the knowledge graph from what a defect is—it's a quality issue, it's done in a production order, it's associated with this part number of this product, which is associated with this requirement. So it's very powerful. Additionally, with that knowledge graph, you can actually enter and do queries against any part of the actual relationships that you've built out.</p>

<p>I could start and ask a question like, for this product, what are the part numbers, and it will list out the part numbers. I could do it from another end and ask, for this quality defect, what are the associated part numbers, and interrogate from there. So that relationship is really key for manufacturers to be able to use their data because it doesn't matter if it's the service department looking at it from a defect or a service quality issue, or an engineer looking for requirements or a part number—they're using the same data, they're just interrogating from different ends of the knowledge graph.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=710" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F16s0uwzhcr7xzoytzex0.jpg" alt="Thumbnail 710" width="800" height="450"></a></p>

<p>So I'm talking about how a knowledge graph is valuable, but when a manufacturer has actually built that knowledge graph and they've actually got that product digital twin, they can actually look at that product across its whole lifecycle. There's really a lot of value from that on top of actually just having the ability to access the data and understand the data in the context of the actual product. First of all, it allows manufacturers to start looking at new ways of actually doing systems development. Some people may have heard of model-based systems engineering. With model-based systems engineering, they have the ability to actually reduce that development cycle time, which is one of the benefits I spoke about at the beginning.</p>

<p>They can look at traceability because now they have the ability to track defects in the field right back to where that actual defect potentially occurred—whether it's an actual bad requirement based on how it's actually going to be operated, or if it's a manufacturing operations problem. It allows them to actually collaborate as well because a lot of manufacturers are working with multiple suppliers for raw materials and parts. So they can actually have traceability, but they also can actually collaborate. If a subsystem is done by a supplier, that can be integrated into the knowledge graph to actually have that information sharing and traceability and the ability to connect across. And then finally, you've got that digital continuity. You can actually truly connect all those different systems of record and actually get value from the data from those systems of record rather than just individually within those systems.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=810" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63lycnnikgqpje5vo5b5.jpg" alt="Thumbnail 810" width="800" height="450"></a></p>

<h3>
  
  
  Implementing the Digital Engineering Framework with AWS Architecture
</h3>

<p>So now I want to talk about how we actually enable this.  We've developed what we call the Digital Engineering Framework. This is a set of architectural best practices to enable manufacturers to basically implement this knowledge graph. This product digital twin is what we bring together, and this knowledge graph is what we're calling the product data fabric. Underneath there, we've got all our systems of record that I spoke about before, and through the Digital Engineering Framework, we can then help manufacturers build that knowledge graph, that product data fabric, that they can then interlock all their engineering and development tools across the whole of that product lifecycle. So it doesn't matter if they're developing the mechanics, the electronics, or the software—their engineering development tools can access that product digital thread through the product data fabric.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=880" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbsq4aw9w3oec4ku3ue7b.jpg" alt="Thumbnail 880" width="800" height="450"></a></p>

<p>So what does that product data fabric actually look like from an AWS architecture perspective? Simplifying it very much, first of all, we've got a couple of our systems of record out there.  What we want to do is take the data from those systems of record and basically bring it into our data store, which our knowledge graph can actually reference. The way that we can do that is using AWS Database Migration Service to take the data from the database and actually bring it into an S3 bucket, or you can use AWS DataSync to sync the data from on-premise if you're actually going to export it and obviously then transfer it up into your S3 bucket.</p>

<p>For some instances, such as running SAP, you could do a direct export from your SAP environment into that S3 bucket. From that S3 bucket, we then use Amazon Neptune to start building the relationships across all those different data sets so that you have your knowledge graph. That knowledge graph and product data fabric is basically the core of it, which is Amazon Neptune. That is the thing that allows us to realize that.</p>

<p>Anyone who has worked with Amazon Neptune or graph databases will know that they have a specific query language that allows you to access the knowledge graph and interpret it to get information back. This is not generally a common skill set, and this is really where the value of generative AI comes in. We can use generative AI with specific large language models to use natural language to interrogate our knowledge graph, take the results of our knowledge graph, and put it back into natural language.</p>

<p>This is where we use Amazon Bedrock and Lambda to do the actual queries against our knowledge graph. Rather than having to write a query in Spark or Gremlin, we can simply ask what part numbers are associated with this defect. Bedrock would then turn that natural language into the query, interrogate our Amazon Neptune knowledge graph, and bring the results back, turning it back into natural language such as part numbers 123456, as an example.</p>

<p>As part of that, we need to provide the ability for people to access and submit those natural language queries. So we need a web-based application and an API depending on whether you want to do it via the web interface or via a third-party application. That is associated on the right-hand side on block number 6. A key thing is that this is manufacturing and engineering data, which needs to be secure and protected. In most cases, this is the key intellectual property for manufacturers.</p>

<p>We need to make sure that we have the right access controls in place so that certain people only have access to the data they need through RBAC models. We need to secure by design, so areas 7 and 8 address that and allow us to audit and monitor it. On the complete right-hand side, we then have our engineering tools and development to interact with that. It can be done via a web interface or we can build native integration with whatever engineering system the customers are using to interact with that knowledge graph.</p>

<p><a href="https://www.youtube.com/watch?v=YdV1Hj5YdB4&amp;t=1090" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faclemmdy99q8kohh22hp.jpg" alt="Thumbnail 1090" width="800" height="450"></a></p>

<p>With that, I would like to say thank you, and that is a very  lightning talk, a brief overview of where we see a product digital twin coming in and how that manufacturing knowledge graph and product data fabric can really drive the value down. I appreciate it. If anyone has any questions, I will be around for a couple of minutes and I am happy to answer them.</p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

