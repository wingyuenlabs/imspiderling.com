---
Title: AWS re:Invent 2025 - Customize & scale foundation models using Amazon SageMaker AI (AIM363)
Description: 
Author: Kazuya
Date: 2025-12-08T22:06:43.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>🦄 Making great presentations more accessible.</strong><br>
This project aims to enhances multilingual accessibility and discoverability while maintaining the integrity of original content. Detailed transcriptions and keyframes preserve the nuances and technical insights that make each session compelling.</p>

<h1>
  
  
  Overview
</h1>

<p>📖 <strong>AWS re:Invent 2025 - Customize &amp; scale foundation models using Amazon SageMaker AI (AIM363)</strong></p>

<blockquote>
<p>In this video, Sumedha Swamy from SageMaker AI demonstrates how to customize foundation models for production using SageMaker AI. The session addresses the gap between demos and production, where accuracy and economics are non-negotiable. Three customization techniques are explained: Supervised Fine-Tuning (SFT) for domain knowledge, Direct Preference Optimization (DPO) for judgment and style, and Reinforcement Fine-Tuning for complex reasoning. Giuseppe provides live demos showing DPO and RLAIF workflows using Llama 3.2 models, highlighting serverless infrastructure, integrated workflows in SageMaker Studio, and new features like serverless model evaluation and MLflow integration. The presentation emphasizes eliminating the "customization tax" through three pillars: choice, efficiency, and safety.</p>
</blockquote>

<p><iframe width="710" height="399" src="https://www.youtube.com/embed/_38nkfU9obE">
</iframe>
<br>
; This article is entirely auto-generated while preserving the original presentation content as much as possible. Please note that there may be typos or inaccuracies.</p>

<h1>
  
  
  Main Part
</h1>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=0" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx5yirk8bf6ks6q2gw99s.jpg" alt="Thumbnail 0" width="800" height="450"></a></p>

<h3>
  
  
  The Production Gap: When Foundation Model Demos Meet Reality
</h3>

<p>Good afternoon, everybody. Can you folks hear me on your headsets? Okay, good. Awesome. Now, quick show of hands, how many of you have experimented with foundation models in the past six months? Okay, now keep your hands up if you have actually gotten those experiments into production. That gap between what works in the demo and what works in production is what we're here to talk about.</p>

<p>Now teams everywhere are building impressive prototypes and demos with foundation models. The demos look great, stakeholders are excited, and budgets are approved. And then production requirements show up. Production doesn't care about your demos. Production has two non-negotiable demands. The first is accuracy that you can defend. Production means mistakes have real consequences: compliance violations, broken customer trust, and bad business decisions. The second non-negotiable requirement is economics that works. What costs just dollars in the demos can cost thousands of dollars in production. And when those inference costs hit your P&amp;L, your CFO is going to ask you pointed questions about ROI.</p>

<p>So teams get stuck. They try hitting accuracy targets with larger and more expensive foundation models, or they cut costs with smaller models and the accuracy drops below what the business can actually accept. Teams accept this false choice, but this is not a law of physics. It is a limitation of approach. There is a way to deliver both higher accuracy as well as dramatically lower costs simultaneously, and that is through model customization.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F22rui1num2yp8ykhrl0t.jpg" alt="Thumbnail 150" width="800" height="450"></a></p>

<h3>
  
  
  Model Customization as Competitive Advantage
</h3>

<p>I am Sumedha Swamy, and I lead product management for developer experiences in the SageMaker AI organization. And today I'll discuss how to customize and scale foundation models using SageMaker AI.  Following my talk, my colleague Giuseppe, who's standing there, will demonstrate these capabilities live through product demos. You're actually going to see advanced model customization techniques in action, so it's going to be an exciting 60 minutes.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=170" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyk0aypy5syu7ezk79o0.jpg" alt="Thumbnail 170" width="800" height="450"></a></p>

<p>So let's be clear, why customize models? Every business now has access to the same foundation models: GPT, Llama, and others. They're all available through APIs. Your competitors have access to the same models. They're solving the same problems and they're targeting the same customers. So where is your competitive advantage? It is not in the models that you can access, it is in what you can actually teach them.</p>

<p>Now foundation models are powerful generalists. They have broad knowledge but no depth in your specific domain. They don't know your industry terminology, your company's data, your business rules, or your brand voice. So in production you're forced to choose the largest and most expensive models just to approach acceptable accuracy, and they still don't fully understand your context. And in addition to that, you're paying premium inference costs to access those largest and most expensive foundation models.</p>

<p>Now when I tell you this, some of you may be thinking, well, I can use prompt engineering, I can use RAG to add context so that I can get higher accuracy, and you're right. For simple use cases that works great. But at production scale, which has strict accuracy as well as cost requirements, those approaches, when they're used alone, hit a ceiling.</p>

<h3>
  
  
  From Generalist to Specialist: The Power of Fine-Tuning
</h3>

<p>For example, take transaction categorization where you are categorizing millions of transactions daily for reporting and fraud detection. You may think that getting 92% accuracy on those models sounds great, but that also means that for 8% of the cases you're miscategorizing these transactions. That is compliance violations, incorrect fraud flags, and essentially you're losing your customers' trust.</p>

<p>That is because your model is reasoning like a generalist, not as a specialist in your domain. With model customization and fine-tuning, this is fundamentally different. What you're doing there is you're encoding your business's domain expertise directly into the model's weights. So the model learns to think like experts in your domain.</p>

<p>So here is an example of what changes when you customize a model. You take a smaller model, let's say an 8 billion parameter model rather than a 400 billion parameter model, and you can fine-tune it on your proprietary data, data such as your transaction patterns, customer interactions, or your domain-specific rules. That fine-tuned small model outperforms larger models on your specific task. Maybe not on all general tasks, but on the tasks that you care about, it can actually outperform the larger models because it has learned your domain patterns.</p>

<p>And because it is smaller, inference of those models is actually going to cost you one or two orders of magnitude, that is 10x to 100x cheaper than doing inference with a larger model. And most importantly, you're not trading off performance for cost. Customization delivers both. In your organizations, your proprietary data is the one asset that your competitors cannot replicate. Model customization is how you take that data and turn that data into your competitive advantage.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=430" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmj39dlgxrrwjb0p4yneo.jpg" alt="Thumbnail 430" width="800" height="450"></a></p>

<h3>
  
  
  The Customization Tax: Why Teams Avoid Fine-Tuning
</h3>

<p>This all sounds great. So if customization is so great and it solves both these problems, why isn't everyone doing it already? Because there was a tax.  Here is what customization actually looks like. Let's say you want to fine-tune a model. It sounds very straightforward, but before you write even a single line of code, you're tweaking YAML configuration files. You're configuring your GPUs for distributed training. What instances should I use? How many of them? What is the fault tolerance strategy?</p>

<p>Then you're stitching together tools, one tool for data preparation, another tool for training, a third tool for evaluations. Nothing really talks to each other, and so you're writing glue code to tie all of these tools together. Finally, let's say you're running your experiment. Let's say you're getting promising results, but try to reproduce them. Let's say you got 94% accuracy. Which dataset gave you that 94% accuracy? What were the hyperparameters?</p>

<p>Without centralized tracking, you're piecing together this information through scattered notebooks and Slack messages. So let's say you have battled through all of that and then your model is actually working and you want to deploy that model. Now that's a completely different set of challenges. You need to build infrastructure from scratch, containerization, versioning, monitoring, rollback strategies. And the audit trail. Which data trained this model? What is the lineage?</p>

<p>You're reconstructing all of this information after the fact. And so it's no surprise that teams are spending 80% of their time on infrastructure and tooling and only 20% of their time on the actual AI problem that they want to solve. So teams make the rational decision. They stay with generic models and they skip customization. That's the customization tax.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjevx9kg01yxxbpjdknp6.jpg" alt="Thumbnail 560" width="800" height="450"></a></p>

<h3>
  
  
  Eliminating Friction: Three Principles for Serverless Model Customization
</h3>

<p>But that friction, it is solved. So here is how we solve the customization tax.  We rebuilt the entire customization experience around three specific principles. The first is serverless infrastructure. With this, you don't configure clusters, you don't plan capacity, you don't select instances. You simply submit a job and we provision the resources, we do the job of doing distributed training, and we scale automatically. You only pay for what you use, not what you actually reserve like you do with instances.</p>

<p>The second principle here is integrated workflow. One environment, SageMaker Studio, takes you from experiment to production.</p>

<p>We provide built-in data preparation tooling, built-in evaluation tooling, and built-in observability tooling. You don't have to switch tools, you don't have to write glue code to connect these tools, and you don't have to hunt through scattered notebooks and Slack messages to find what you did. Everything is tracked, versioned, and reproducible in one place.</p>

<p>The third principle here is production-ready techniques. Supervised fine-tuning, direct preference optimization, and reinforcement learning—these advanced model customization techniques are available to you across a broad set of models. Amazon Nova models, Llama, Qwen, and many, many more, both open weights as well as proprietary models, are available. These are the same methods that the top frontier labs are using. What we have done is we have packaged these methods for your production use cases so that with a simple configuration change you can make them work for you rather than writing these from scratch. You're going to see a demo of all of this very soon.</p>

<p>Focus on your domain problem—we handle infrastructure. That is the fundamental shift. The 80/20 split I spoke about earlier, we inverted it. Now you can spend 80% of your time on your AI problem and 20% on the infrastructure and tooling because that tax is gone. So now let me dive a little bit deeper into the customization techniques that you can actually use.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=720" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fepb0nvwga1uxb594zzt8.jpg" alt="Thumbnail 720" width="800" height="450"></a></p>

<h3>
  
  
  Three Production-Ready Techniques: SFT, DPO, and Reinforcement Fine-Tuning
</h3>

<p>SageMaker gives you three production-ready techniques to choose from. The first is supervised fine-tuning, or SFT. SFT teaches your model domain knowledge. You provide it with labeled examples, saying, for example, here is the transaction, here is the category, or here is the customer question, here is the answer. The model learns through pattern matching—it absorbs your terminology, your procedures, your institutional knowledge. This is how you move from a generic AI to AI that understands your domain.</p>

<p>The model absorbs your patterns and learns to recognize similar situations when new data is coming in. But it only learns explicitly what you teach it. It can't start developing judgment or reasoning beyond the data that you have actually given it.</p>

<p>The second customization technique is direct preference optimization, or DPO. DPO teaches a model judgment and style. Now, instead of labeled examples like you saw with supervised fine-tuning, with DPO you're showing the model pairs of responses, things like, hey, this response represents our brand voice, our risk tolerance, this response does not. So you give it these kinds of response pairs, and you're going to see an example in the demo. The model, using those response pairs, learns your quality standards, your decision-making criteria, and the subjective preferences that define how your company operates.</p>

<p>This is particularly powerful when the outcomes are hard to define programmatically, but once you see the outputs, it's easy to recognize which is a better response. This is when you would use something like DPO. For example, asking a question such as, hey, does this response sound professional? Does it align with values? DPO captures these nuanced preferences that cannot easily be reduced to simple rules. This is how you move from AI that knows your facts, which is what you did with SFT, to AI that actually sounds like you.</p>

<p>Now, the third technique is reinforcement fine-tuning. This is to teach your model complex reasoning. Reinforcement fine-tuning trains a model to achieve outcomes that are very difficult to define programmatically, but once you actually see the output, it is easy for an AI to evaluate and give feedback, or any other tool to be able to evaluate and give feedback. For example, training a model to sound helpful and empathetic is a very interesting challenge.</p>

<p>It's very hard to capture this in code. You can't write explicit code to define what is warmth in speech or what is understanding in speech, but you can give that response to an AI model and ask it for feedback on whether the response feels right. That's how Reinforcement Learning from AI Feedback works. The model generates output, you take that output, AI gives you feedback and a reward, and then the model learns to actually optimize for that reward. That is RLAIF, Reinforcement Learning from AI Feedback.</p>

<p>This is great when it's very difficult to say there's a clear right and wrong answer, but it's somewhat nuanced. However, in some cases there's a very clear right and wrong answer, like coding or math. In these cases, you use something like reinforcement learning with verifiable rewards, wherein you take a function that actually verifies whether the output is correct or not, gives that output feedback to your model, and the model will get the reward and try to optimize for that reward. All of these techniques are interesting in moving the complex reasoning part of the model forward.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=990" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ns1uezcfplka2l1rf80.jpg" alt="Thumbnail 990" width="800" height="450"></a></p>

<h3>
  
  
  SageMaker Studio: Unified Workflows with Serverless Evaluation and MLflow
</h3>

<p>Now, you can start simple and evolve as needed based on your use case because SageMaker supports all these three. So you can begin where it makes sense for your use case and scale up as your use case demands  more. So where do you get started with building with these techniques? What we realized was different teams work differently, so we built three experiences for model customization.</p>

<p>The first is in SageMaker Studio UI for visual workflows. You select a model, upload data, configure training, no code required. This is perfect for rapid experimentation. The second is AI agent guided experience. This one is in preview as of today. Here you describe your goal. You tell the agent something like, "Hey, I need to fine-tune a model for contract analysis." The agent then asks you a bunch of questions to learn more about your use case, and it actually builds a spec, a specification of what needs to be done in order to customize a model.</p>

<p>You can read that spec, you can edit that spec, and then the AI agent actually helps take that forward. For example, if it has to generate synthetic data in order to train your model, it goes ahead and generates the synthetic data. Then with your permission, it can actually execute the end-to-end pipeline as defined in that spec. So you can go from the spec to training to evaluation to a production model just by talking to that agent in natural language. It's like an AI expert that is helping you with your model customization journey.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1130" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fquw15b69uoboaetvspv3.jpg" alt="Thumbnail 1130" width="800" height="450"></a></p>

<p>Finally, the code-based SDK. This is super important for full programmatic control so you can integrate model customization directly into your MLOps pipelines. Now, here is the important thing. Here is what matters. These are not three separate products. These are three interfaces to the same underlying service, which is SageMaker AI. It's the same model registry, the same version control, the same audit trail, so your security teams will just see one compliance trail. So you have three different paths, one production model, one audit trail. </p>

<p>All of these, the three paths, the customization techniques, it all comes together in SageMaker Studio. Studio is the web-based IDE for your AI workflows. You get visual model customization forms and you get JupyterLab notebook code and VS Code IDE for your code-based development. Whatever matches how you work, we provide all of those development tools in SageMaker Studio, and you can seamlessly switch between them as well. You're going to see a demo of SageMaker Studio very, very soon.</p>

<p>But here is what actually matters. The entire workflow here is connected from experimentation to training to evaluation to deployment to monitoring. All of this is connected. It all happens in one environment. You can experiment in a notebook, you can train and evaluate with just a few clicks with the form. When the results meet your bar, you can click deploy, and the model goes and gets deployed live for your inference either on SageMaker or on Bedrock.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1210" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fto2mg7ntzyixuswlmbko.jpg" alt="Thumbnail 1210" width="800" height="450"></a></p>

<p>So I'm excited to share that we are adding two more new capabilities into SageMaker Studio to make your workflow even  more powerful. The first is serverless model evaluation. It is really important to be able to evaluate your models to understand how they are performing. With this feature, you can evaluate what actually matters, whether it is a customized model versus a base model, or when you're doing model customization, different checkpoints of your customization, or you may want to experiment with multiple fine-tuning techniques like SFT and DPO and evaluate them.</p>

<p>We've built several evaluation methods right into Studio. We provide standard benchmarks like, for example, Multitask Language Understanding, the MMLU that you see for many foundation model benchmarks. That's available in SageMaker Studio. Math problem-solving benchmarks are available already. There are a bunch of other standard benchmarks available. But sometimes you may want to use something like an LLM as a judge, wherein you enter a custom prompt to define your evaluation criteria. That's available as well. Or if you want to write your own custom evaluators to determine what constitutes good for your model for your use case, you can go ahead and bring your custom evaluators as well, and you're going to see a demo of that very soon. So the idea here is you know if your customization is actually working before you deploy your model to production. And most interestingly, all of this is serverless, so you don't worry about managing infrastructure or evaluation infrastructure. You just focus on actually what you're trying to evaluate in the model.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1310" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxbhthligw1k7hn4ltsv7.jpg" alt="Thumbnail 1310" width="800" height="450"></a></p>

<p>The second feature I want to talk about is serverless MLflow. Now every model customization training on SageMaker Studio automatically logs your metrics, your hyperparameters, your datasets, and your model artifacts. We've built experiment tracking right into Studio. So key metrics like training loss for your model are directly available in your Studio interface. If you want to dig into more details into metrics, with one click you can launch MLflow UI and you can start looking into more details into your model training metrics. You can view the complete lineage for any model, things like which data trained that model, which configuration did I use, and who ran that experiment. So you know exactly which experiments are running and you know how to reproduce them maybe a year later or six months later. Once again, all of this is serverless. You don't need to worry about infrastructure. We take care of it. And here, best of all, there is no additional charge for you to use this capability.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1380" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbfec6vph1dpfzinhha8q.jpg" alt="Thumbnail 1380" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyenozz5vsnmxkzzkjm4i.jpg" alt="Thumbnail 1390" width="800" height="450"></a></p>

<h3>
  
  
  Compute Options and Core Pillars: Choice, Efficiency, and Safety
</h3>

<p>Now let's talk a little bit about compute options because you have two options here. The first is  serverless model customization, and we've been talking about this a little bit in the past. In this case, you submit a job, we do the job of provisioning the GPU cluster, we take care of distributed training, model parallel, data parallel, we take care of all of those details for you. We train the model, and then once the model has been trained, we tear down the infrastructure. You only pay for the resources that you actually use for training. You're not paying for idle compute resources. This is perfect for rapid iteration and experimentation, and this is new. It is launched as of yesterday.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1440" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fddfxt4vqu81g5o3v1jzu.jpg" alt="Thumbnail 1440" width="800" height="450"></a></p>

<p>The second compute option is for customers who want maximum control, and this is SageMaker HyperPod.  This is a persistent cluster. You can share it across training, inference, and your interactive machine learning workloads, and it automatically prioritizes your workloads so you can maximize the usage of these expensive GPU resources. The killer feature here is resiliency. Node failures are very common when you're running a large distributed training system. HyperPod automatically creates checkpoints and then recovers from checkpoints and does process-level restarts. So that means that when a fault occurs, you resume from the last checkpoint, and this is really cool because it saves up to 40% of time on a long-running job.</p>

<p>Now we tried to enhance this even more. So just yesterday we launched that HyperPod now supports checkpointless training, so that means that you're going to be able to recover from your faults even faster, in minutes rather than hours. So if you're running your distributed training job for days or months, this quickly adds up to huge amounts of savings.</p>

<p>So I presented these two compute options, which should you choose for your jobs? Now, if you're iterating quickly, training smaller models, you should simply use serverless training. But if you're running long training jobs for days or months on frontier scale models or you want granular control, you should be using HyperPod in those cases.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1560" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6anks7dlxw21rel9n2jo.jpg" alt="Thumbnail 1560" width="800" height="450"></a></p>

<p>HyperPod is also super flexible. I've shown here Amazon EKS as the control plane, but HyperPod supports orchestration with both Slurm as well as EKS. And if you want this kind of granular control, this is the best way to be able to train frontier scale foundation models.  So finally, let me bring this all together with the three pillars that comprise SageMaker AI model customization.</p>

<p>The first is choice. You're not locked into one model family or one technique. You can either use open weight models or you can use proprietary models. You can use supervised fine-tuning, direct preference optimization, reinforcement fine-tuning. You can work through the UI, you can work through an AI agent, you can work through an SDK. You can choose what fits your use case and your team, so we give you choice.</p>

<p>The second pillar here is efficiency. Serverless training eliminates infrastructure management, but if you want control, HyperPod gives you the control when you need it. The customization technique that used to take months before now takes days, and you will see Giuseppe demo it and you can see that it is actually that simple. So your teams move faster.</p>

<p>The third is safety. With enterprises, it is non-negotiable that safety is like a number one priority. So you get built-in governance, built-in lineage. You know what data was used, which model was used, which version of that model was used, who trained it. You get IAM-based access control. Everything operates securely in your VPC, and your data never leaves your control. Choice, efficiency, safety. That's how the customization tax is eliminated, and you turn foundation models to your competitive advantage.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1670" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1qiqik721804nlpi0mq6.jpg" alt="Thumbnail 1670" width="800" height="450"></a></p>

<h3>
  
  
  Live Demo: Direct Preference Optimization Through UI Workflow
</h3>

<p>Now Giuseppe is going to show you a demo that's going to bring all of this together. The problem that we're going to be solving in this demo is we want to customize a Llama 3.2 1 billion instruct model to sound more human-like in conversations. For that we're going to use a dataset that comprises of conversational questions and two types of responses. The first response is human-like responses, and the second response is formal responses.</p>

<p>Now Giuseppe is going to show you two demos. In the first demo, he's going to show you using direct preference optimization using a UI-based workflow. Remember again, DPO is where you teach a model judgment and style by showing it a pair of responses. In this case we're showing it human-like responses and formal responses, and we'll train the model to prefer the human-like responses. That's what we're going to be doing with DPO. And you will see how easy it is to configure this training without writing any code.</p>

<p>In the second demo, Giuseppe is going to demonstrate reinforcement learning with AI feedback, and this is going to be a code-based workflow. Again, remember, RLAIF is a technique wherein a model generates a response and gets feedback from an AI model as to how that model is evaluated. It gets a reward, and then your model that you're training actually works to maximize that reward. That's what you're doing with RLAIF. So it's the same model, same dataset, two different techniques, all of it training running on SageMaker AI. Giuseppe, take it away.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1790" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo9z0r4putiuezwa9so49.jpg" alt="Thumbnail 1790" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1800" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpvz7s90hbqcs6u7nt3zt.jpg" alt="Thumbnail 1800" width="800" height="450"></a></p>

<p>Thanks, thanks, Sumaira. Let's move on to the demo part of this session. Is the video readable? That's okay, great.  So we are here on SageMaker AI Studio and we are going to start the model customization workflow. Before we get started, let's take a look at the  dataset that we are going to use today.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1820" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fawxygm6fikno9gzsel9a.jpg" alt="Thumbnail 1820" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1830" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7nh7go6amdr830hjtoet.jpg" alt="Thumbnail 1830" width="800" height="450"></a></p>

<p>The Human-Like dataset comes with a prompt, a chosen response, and a rejected response since we want to use Direct Preference Optimization here. In order to use this dataset for model customization in  Amazon SageMaker AI, we can take a look at the supported formats for the datasets. There's this new section called Assets in SageMaker AI Studio, which gives you the opportunity to  collect all your datasets here and version all your datasets here.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1850" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsrnj1oggj7ekn1pf09t5.jpg" alt="Thumbnail 1850" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1870" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9myaxdxd6m17k87mzyb6.jpg" alt="Thumbnail 1870" width="800" height="450"></a></p>

<p>I can click on Upload Dataset, and in the Required Data Input Formats I can find the various input formats that are required to customize our model depending on the technique that we are utilizing. For Direct Preference Optimization,  we have a prompt, a chosen, and a rejected response to be provided. Indeed, it actually matches the format of the dataset that we have found on Hugging Face. However, since we also want to evaluate this model, we are going to do some data preparation on the dataset to prepare the evaluation dataset as well. </p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1890" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fip2f7jw2x8272vnep2xp.jpg" alt="Thumbnail 1890" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1900" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1brsbx58tgibtqn8z6k7.jpg" alt="Thumbnail 1900" width="800" height="450"></a></p>

<p>In order to do that, I've written some simple code in a managed notebook environment provided by Amazon SageMaker Studio to transform the dataset and generate the training and evaluation split. It's pretty simple code. We are just loading the dataset and then  observing the structure of the dataset that we have already seen on the Hugging Face website. Then you can easily see  how for a given prompt the chosen response sounds more human-like, more conversational, and the less preferred rejected response is more like an LLM-generated response which is more artificial, more robotic.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1920" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fq8t4bz2t51we3ew9cmhp.jpg" alt="Thumbnail 1920" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyoanbljk9zhcvut1ehi8.jpg" alt="Thumbnail 1930" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1950" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1ztk5slnjpsol2h0b507.jpg" alt="Thumbnail 1950" width="800" height="450"></a></p>

<p>In this dataset, we count the elements. It has around 10,000 elements for training.  We are doing a split of training and evaluation here with 10% of the dataset used for evaluation. Then I'm defining a couple of functions that we're going to use to transform the dataset.  There's no specific transformation for the training set, but just a simple transformation to obtain a query and response for the evaluation set. Then we are executing these transformations here in this part of the code, and finally displaying just an example, a random example from the validation dataset  to check that we have achieved the expected structure.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fylg33wzv42gjeicv251v.jpg" alt="Thumbnail 1960" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1970" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgcharc7hwxx95tpydgzn.jpg" alt="Thumbnail 1970" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1980" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzt12yi1o2mwuwz3we4tz.jpg" alt="Thumbnail 1980" width="800" height="450"></a></p>

<p>Finally we can save these datasets  to files or we can decide to upload the datasets to Amazon S3 depending on how we want to handle the customization workflow. Our data is ready. We can jump out of the notebook  environment and we can just use the functionality in the UI to upload the dataset. I have my training dataset here, then we can save the dataset and  this will just upload the dataset to Amazon S3 object storage and then store the dataset metadata in the registry.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=1990" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fogngk07invbdqs3x3x49.jpg" alt="Thumbnail 1990" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2000" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuwqaj33pys1zfxezjazd.jpg" alt="Thumbnail 2000" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2010" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo16lf1u3xyavwcfh7k15.jpg" alt="Thumbnail 2010" width="800" height="450"></a></p>

<p>If we create the dataset here, we should,  sorry, I forgot to give it a name. We can put just a random name for now. Then the dataset appears in  the catalog, and we can also take a look at some samples in this dataset. For example, we can see the first sample just to check the structure. That's "What's  your favorite type of cuisine?" and then the chosen response, "I'm a big fan of Italian cuisine." I promise that I didn't put any bias in this dataset.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2020" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvv01kwftooez6uxebers.jpg" alt="Thumbnail 2020" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2030" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7ghwfu9jv0axcr8htma.jpg" alt="Thumbnail 2030" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2040" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flcqdekwmllsaulvw401i.jpg" alt="Thumbnail 2040" width="800" height="450"></a></p>

<p>Let's move on to the model customization itself. We're going to customize a Llama 3.2 1 billion parameter  model, so we can search in this new Model Hub. We can search for our model, which is the 3.2 1B Instruct.  This is just a model detail page with all the information about the model, and then we can move to the Customize section, this new Customize with UI or Customize with Code section.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2050" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2wh3newjr9nc6vnlknk1.jpg" alt="Thumbnail 2050" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcoujulzog5xrkbrcehha.jpg" alt="Thumbnail 2060" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2070" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa17h0l831kl0w3y8ndvn.jpg" alt="Thumbnail 2070" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2080" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft9ns0zvrgky86xmdo7y2.jpg" alt="Thumbnail 2080" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2100" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp73ra7cem90xy8kze7nr.jpg" alt="Thumbnail 2100" width="800" height="450"></a></p>

<p>Let's go with the customization through the user interface. What we need to do here is give a name to our model, give the custom model a name.  Then we can choose the technique that we want to use, Direct Preference Optimization. For the dataset, we could have also uploaded the dataset here, but since I already have the dataset in the registry, then  I can select my dataset. Then we find a number of defaults already configured in the user interface like the output  location where we want our artifacts to be stored, as well as the hyperparameter configuration. All of that comes out of the box, preconfigured recipes for you to get started. You don't have to necessarily modify these parameters, but if you want, you can change the parameters according to the results of your experiments. Then you also have an Advanced Configuration  section where other hyperparameters can be tuned, as well as the configuration of the MLflow application that we want to use for experiment tracking.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2110" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fot7aslq89uh1ylhjvoz5.jpg" alt="Thumbnail 2110" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgr808gn8bzhzzae7k1qr.jpg" alt="Thumbnail 2120" width="800" height="450"></a></p>

<p>You can set that in this section. What we then need to do is just submit our job and accept, and  this will trigger a training job, which is the actual customization job.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2140" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwmfcwop39bw1m5u3lp1l.jpg" alt="Thumbnail 2140" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjca0fl5wfaqp51w35clc.jpg" alt="Thumbnail 2150" width="800" height="450"></a></p>

<p>This will take some time. We are training for three epochs. It's around one hour of training based on my experiments. What I'm going to do is show you an example that I have already pre-run for DPO.  We can take a look at the custom model screen, which is where you will land after the customization job gets executed.  Here you can see first of all the logs from the model training. In this screen we can also look at the performance that we have achieved, some performance metrics for our job.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2160" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fykk0ghzospyp0n1h3bat.jpg" alt="Thumbnail 2160" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2180" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fye4mfijldzk40ppwq9f1.jpg" alt="Thumbnail 2180" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2190" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwkmcoo14k6ow8jrth4us.jpg" alt="Thumbnail 2190" width="800" height="450"></a></p>

<p>One interesting metric is the margin of the rewards, which represents essentially the difference between the rewards given from our model to the chosen versus the rejected response. It means that in some way it can give us an indication on how the model is learning.  If we want to browse the full metrics, we can open MLflow directly from the user interface, and this will open the serverless MLflow interface  to dive deep into the various model metrics and how our training has been behaving.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F62l6xvg7i3c7bu2dsas3.jpg" alt="Thumbnail 2200" width="800" height="450"></a></p>

<p>We have run the fine-tuning. The model is ready. The next thing that we might want to do is test this model very quickly, run some inferences, and see if we have really fine-tuned and adapted our model with respect to the task that we wanted to achieve. In order to do that, we can use the deploy functionality which is still available from the user interface.</p>

<p>You can decide to deploy to SageMaker or you can deploy to serverless inference in Bedrock. In the case of SageMaker, you can choose whether to use an existing endpoint that is already running, and this means that SageMaker gives you the ability to bring back models on an existing endpoint infrastructure so that you can optimize on the cost, or you can create a brand new endpoint. Let's create a new endpoint, for example, and just in a few clicks we give a name to the endpoint.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2250" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqqo12ssdqot1x3zmj6kc.jpg" alt="Thumbnail 2250" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2260" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzlxindyzy127bd049kwu.jpg" alt="Thumbnail 2260" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2270" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3t7206yh7p3xzz6ijjpu.jpg" alt="Thumbnail 2270" width="800" height="450"></a></p>

<p>The compute instance type is already pre-selected based on the model that we have fine-tuned, and we can just hit deploy.  I'm not going to run deployment for now, but we can go back to the model screen where I have already run a deployment and look at the details of the deployment.  We can see here that we have our model deployed to the endpoint. Since for the model training we use the LoRA techniques to optimize the fine-tuning process, the service automatically deploys on the endpoint both the base model as well as the adapters as separate components.</p>

<p>The reason is that in this way you have the ability to load multiple adapters on the same endpoint that would be merged at runtime with the base model to execute the inference. This is an additional optimization functionality that SageMaker provides. Let's see indeed if our model gave us some good results, and we can use this playground to do that. Here on the right-hand side you can choose which model you want to invoke, whether the base model or the fine-tuned one.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2330" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc0cfb8tizc8iyv86n70a.jpg" alt="Thumbnail 2330" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2360" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd0ahjrri74c01801ve64.jpg" alt="Thumbnail 2360" width="800" height="450"></a></p>

<p>Let's try to invoke the base model with a simple question. How are you today?  The base model is replying, I'm just a language model, I don't have emotions or feelings and so on, so the response that we would expect from the generic model. Now let's see if I just try to invoke the adapter and clear the conversation.  We can see that indeed we are having a way more informal response from our model, including emoticons, exclamation marks, and so on.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2370" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fw46pev92kgnnjtf4foky.jpg" alt="Thumbnail 2370" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2380" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxos3agc1nux8emx956q9.jpg" alt="Thumbnail 2380" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2390" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvknrasjp5mjrjhzh96tv.jpg" alt="Thumbnail 2390" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2400" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F80tmil0pos2qlyt8c4m2.jpg" alt="Thumbnail 2400" width="800" height="450"></a></p>

<p>That's not enough. We have deployed the model. We want to actually assess its performance using an evaluation dataset, and how can I do that?  We have a built-in capability from the user interface to evaluate our model, to set up an evaluation.  The way you can do that, you can use multiple techniques as mentioned. You can use standard well-known benchmarks for doing that.  You can implement a custom scorer which is essentially using code, your own function to evaluate the model.  Or you can use the LLM-as-a-judge technique where we are asking another bigger model to judge the response that is coming from our model with respect also to the ground truth that we are providing to the model from the evaluation set that we select as an input.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2420" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frwi8uc7tdr1a5np58feq.jpg" alt="Thumbnail 2420" width="800" height="450"></a></p>

<p>That's indeed what I have used as a technique to evaluate this model. There are multiple models that we can choose for  the evaluation. I've chosen Claude 3.5 Sonnet for this evaluation. On the metrics side, you have some built-in metrics, but as you can imagine, for evaluating the tone and style of a model, we might want to use a custom metric and instruct our judge model on how to interpret whether the responses from the fine-tuned model are actually humanlike responses.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2450" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftcmlrxy78wgdrk4v790x.jpg" alt="Thumbnail 2450" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2460" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsigu8fhzvp86fgnds790.jpg" alt="Thumbnail 2460" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2470" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fp8drtzoq3e3ub537x9d5.jpg" alt="Thumbnail 2470" width="800" height="450"></a></p>

<p>That's what indeed I have done, and you can use  the custom metric screen to do that. The custom metric screen supports both importing a metric that you have saved in a JSON file  for your consumption at any time, or you can use the custom screen where you can input the instructions that you want to provide to the judge model. You can also select a scale type  for rating the response that has been provided by your model, and this scale type could be numerical or categorical.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2490" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0qwysz2uu80902nenp62.jpg" alt="Thumbnail 2490" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2510" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4unf01qavg1ur1st561m.jpg" alt="Thumbnail 2510" width="800" height="450"></a></p>

<p>Let's take a look at the metric that I want to use for this evaluation. The metric that I have defined is called humanlike  metric, and as you can see, we have a few instructions. I hope this is readable, but we have a few instructions which instruct the large model on how to evaluate the outputs from the fine-tuned model with respect to the ground truth that we are providing from the evaluation dataset. Indeed, you can see here that we have some placeholders  for the prompt, the ground truth, and the prediction.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2520" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1fsxnadlynodiyrqk63l.jpg" alt="Thumbnail 2520" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2540" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpk8lui7kuo3vkj0gnms2.jpg" alt="Thumbnail 2540" width="800" height="450"></a></p>

<p>Let's see the results that I have achieved. The remaining steps here are just about  specifying the dataset that we want to use and submitting the evaluation job. Let's see the results that I have achieved on this model. I've run a few evaluations, but we can take a look at the evaluation results from this screen using LLM as a judge. As you can see, we have here all the  records that have been evaluated with respect to the base model. I have decided to compare side by side the base model results and the fine-tuned model results, and we can see here that, for example, for this prompt, the base model score was somewhat humanlike. For the humanlike metric, the custom model score is completely humanlike, and so forth for the various prompts.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2590" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fewwqb878zy6x3lllk53v.jpg" alt="Thumbnail 2590" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2600" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8y4qf2fyk8i22bjsirfu.jpg" alt="Thumbnail 2600" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2610" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqid650wrkgt2pczmg85f.jpg" alt="Thumbnail 2610" width="800" height="450"></a></p>

<p>The results that you get from the evaluation are still accessible at any time. While you see these results from the user interface, they are still accessible at any time from S3, where you see the links here for the base model evaluation results and the custom model evaluation results. Indeed, what I have done to have an overall summary of the evaluation results is I have downloaded these results,  those that are stored in the S3 bucket, to my notebook. Then I've mapped the various categories of the scores, not at all, somewhat, and completely,  to numbers so that I can compare the responses on the overall results. What I get with a simple function is that our fine-tuned  model has improved in 49% of the cases with respect to the base model, has equal performance for 41% of the cases, and in about 10% of the cases has degraded performance according to the judge model that we have used for this evaluation.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2630" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv4fdnagta1lrmv57xdbt.jpg" alt="Thumbnail 2630" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2640" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxh5xhleaq8ibbwvaqsub.jpg" alt="Thumbnail 2640" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2650" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fek88b0sp6t9js5qtot2t.jpg" alt="Thumbnail 2650" width="800" height="450"></a></p>

<p>Finally,  before we move to the next example, I wanted to show you that all the actions that we execute in the user interface are tracked, and we get the full lineage graph  of the operations that we have executed. We can check this from the UI, and as you can see, for the model that we are customizing,  we get all the links to the training job that was used for customization, as well as the deployments that we have executed, the various evaluation runs, and so forth. Finally, the last thing on this example is that you can always continue the customization starting from the fine-tuned model that we have obtained by either changing hyperparameters on the customization using the same technique or changing completely the technique. This concludes the DPO demo.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2690" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F93jaozrp6f94qie7vr3k.jpg" alt="Thumbnail 2690" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2700" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk5sr0e500rpy3fmda1hb.jpg" alt="Thumbnail 2700" width="800" height="450"></a></p>

<h3>
  
  
  Code-Based Workflow: Reinforcement Learning with AI Feedback
</h3>

<p>Let me move now to the code-based workflow. Whatever we have done in the user interface  can be achieved using the SageMaker Python SDK as a code-based workflow. Similar to the previous example, I have  prepared the dataset. I will skip these steps because they are almost the same steps.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2710" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9oj726axlnvtgz41qhoq.jpg" alt="Thumbnail 2710" width="800" height="450"></a></p>

<p>The data preparation steps are almost the same for reinforcement learning with AI feedback. The only thing to note here is that  the structure of the input format is slightly different because we want to provide the model that is going to give us the feedback on the generated responses from the fine-tuned model. We need to give this model the ground truth information. That's the expected structure that you can find also from the same user interface where the dataset formats are defined.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2740" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj6do57bv1q9symhyrh82.jpg" alt="Thumbnail 2740" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2750" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9xtrclb1qp08d9uv56rb.jpg" alt="Thumbnail 2750" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2780" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9o9vj7n45x0guhrjaswd.jpg" alt="Thumbnail 2780" width="800" height="450"></a></p>

<p>For evaluation, we are using the same format. I will not spend too much time on these data preparation stages, but  what I want to show you is that we are uploading the two datasets to S3. In this case, we are not using the user interface for the upload. We just upload via code to S3. Then what we are doing is importing from the SageMaker AI Registry module the DataSet class, and this helps us create the datasets. So the same operation that we have seen in the user interface can be done from the programmatic interface. We create the two datasets that get  added to the registry.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2810" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkasra53cgbcaruns0rig.jpg" alt="Thumbnail 2810" width="800" height="450"></a></p>

<p>The next important thing is defining the reward prompt. This is what the feedback model needs to look at to understand how to score the responses from the model that we are fine-tuning at each iteration of the reinforcement learning.  This is the prompt that I have used. You can see that it's a pretty sophisticated prompt to check if some response is human-like or not. What is important is that we need to return from this prompt a JSON that contains the score in a range that I have defined from zero to one. This is the score that has been given by the feedback model to the responses generated by the model that we are fine-tuning through reinforcement learning.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2840" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0ntb5g946nc1rzuwmkpq.jpg" alt="Thumbnail 2840" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2850" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9nf5q20e2qh6zkds0069.jpg" alt="Thumbnail 2850" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2860" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fu2mps4296qhjfi9bv8ip.jpg" alt="Thumbnail 2860" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2870" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fzjqhrsyegzyb1c38w3kz.jpg" alt="Thumbnail 2870" width="800" height="450"></a></p>

<p>Once we define the reward prompt,  just to show you where this ends up, it becomes available also in the evaluators screen in SageMaker AI Studio, so you can review the prompt at any time and  take a look at what the structure of the prompt is and the text also from the user interface. The next thing that we have to do is define the  name of the model, the same thing that we have seen in the UI. We will create the name for the custom model that we are fine-tuning.  The key step here is using the Trainer class of the SageMaker Python SDK, which configures the reinforcement learning with AI feedback process.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2910" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcsq4s67m1c2l4r9penw5.jpg" alt="Thumbnail 2910" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2920" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs3wpt3lkqsf4ydg5u54.jpg" alt="Thumbnail 2920" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2930" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk3fv17xetkg3kcr5bz09.jpg" alt="Thumbnail 2930" width="800" height="450"></a></p>

<p>This class takes as input the base model, which is indeed set to Llama 3.2 1 billion instruct. It needs the information about the reward model, and in this case, as a reward model for the reinforcement learning, we are using GPT 120 billion. Then a few other parameters for the S3 location where we want the outputs to be stored, the input dataset that  we are providing for the fine-tuning, and so forth. After we create the class, we have  a property to access the hyperparameters and look at the default configuration of the hyperparameters, but we can also set the various hyperparameter values as needed. For example, in this case,  I wanted to just train for one epoch, and I have modified the configuration of the hyperparameters.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2940" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnkthlwng0urh1s0e4c0h.jpg" alt="Thumbnail 2940" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2950" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3xi2d4x2d4pv6z86tnp.jpg" alt="Thumbnail 2950" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2960" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fyy4lbym3xcstwdcawr8z.jpg" alt="Thumbnail 2960" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2970" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvt9ge13vguysbk9r9vt5.jpg" alt="Thumbnail 2970" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=2980" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faxujnis874prkmcdbwqh.jpg" alt="Thumbnail 2980" width="800" height="450"></a></p>

<p>Finally, the last step is running the training, so calling  the train method of the trainer to execute the training and trigger the training job.  The result of this operation is available already in my UI, and I can show you the model that I have trained. It's indeed  here. For this model, we are getting quite interesting results. As you can see, we have  the rewards which are increasing, which we expect, and also the policy entropy is decreasing, which means that the model during the reinforcement learning  is going towards more exploitation rather than exploration. So it means that it's kind of converging in the right direction.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3000" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk0ertcez5meuo1eu12qh.jpg" alt="Thumbnail 3000" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3010" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F961j8xgbk9oy3x4s5bh5.jpg" alt="Thumbnail 3010" width="800" height="450"></a></p>

<p>Same thing, we can monitor the metrics from MLflow. But again, now let's take a look at if we had good results with fine-tuning this model. I have also deployed this model to a SageMaker real-time endpoint. We can go  and open the playground and see what we get from this model, and we can ask the same question, hey, how are you today?" </p>

<p>And well, that's the expected answer also in this case from the base model.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3020" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flj6k0e89z9clq7mqqxth.jpg" alt="Thumbnail 3020" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3030" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh4bd0w5prdlreqrw1smg.jpg" alt="Thumbnail 3030" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3060" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh3wakfporv31mdir29zw.jpg" alt="Thumbnail 3060" width="800" height="450"></a></p>

<p>Now let's take a look at the adapter model  that we have fine-tuned. Let's clear the conversation. "Hey, how are you today?" We  see that again we have a more informal, more human-like response from the model. Although as you can see in this case, we don't see a lot of emoticons or exclamation marks, and the main reason here is that we are getting the feedback from another large language model and we didn't instruct this model to give more weight to this kind of style. So we didn't inform the model towards  achieving this objective.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3070" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fak1dcrhgenfo59397dww.jpg" alt="Thumbnail 3070" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3090" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdg24gfdn8w0n1vgjwrkm.jpg" alt="Thumbnail 3090" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3100" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foawzgpmmfaqb2t00jgoy.jpg" alt="Thumbnail 3100" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3110" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhyeh5r09p30g0ynwyqyl.jpg" alt="Thumbnail 3110" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3120" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn552p4emtiug6ggt59g2.jpg" alt="Thumbnail 3120" width="800" height="450"></a></p>

<p>Great. Similar to the previous example, we can see how the evaluation is triggered. The same evaluation with LLM-as-a-judge  can be triggered using SageMaker Python SDK, the same thing that we have seen in the UI, and we are doing exactly the same type of evaluation that I have run for the use case. In order to do that, the custom metrics are defined as a JSON  in this variable custom metric list for LLM-as-a-judge, and then I'm using this class LLM-as-a-judge evaluator  to execute the LLM-as-a-judge task. The model that we are using as an evaluator in this case is Anthropic Claude 3.5, so the same model  that we used in the previous case. For sure it's important to make sure that we are using a different model from the reward model that we use during the RLAIF. </p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3130" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fogxyu39cx7eo7c7rr070.jpg" alt="Thumbnail 3130" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3140" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkrx1i6w1setdka7o4o31.jpg" alt="Thumbnail 3140" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3150" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fuyec0x1y4wl534jootnu.jpg" alt="Thumbnail 3150" width="800" height="450"></a></p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3160" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3gq0tzuo8w558dzpktst.jpg" alt="Thumbnail 3160" width="800" height="450"></a></p>

<p>Great. That said, we can then run the evaluator evaluate method and this produces the evaluation that you see in this  screen, which has similar results to the previous evaluation. We have a prompt, the model response, the base score, custom score  that have been provided by the judge model, and now we can again compare the results, the overall results  using just a couple of lines of code. We can see that in 74% of the cases here  our model has improved and has equal performance in 24% of the cases, degraded way less than the other models. Well, that's kind of expected because we have used a large language model as a reward model during the reinforcement learning, and then we have used another large language model to evaluate the performance. But we didn't consider specific aspects, for example, including emoticons and so on. So the expectation would be that the evaluation run through Anthropic Claude in this case would have been higher.</p>

<p><a href="https://www.youtube.com/watch?v=_38nkfU9obE&amp;t=3200" rel="noopener noreferrer"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo8htzc721gg6rin6uasi.jpg" alt="Thumbnail 3200" width="800" height="450"></a></p>

<p>Great. That said,  this concludes the demo for today. Thanks, and we can now move back to the presentation. So the key that I would like you to note here is Giuseppe very seamlessly moved between the UI and the SDK. What you were actually able to do in the SDK, you can come into the UI. This is really what I meant by saying that these are not different products. This is the same service. Depending upon who you are as a user, you can choose which interface you want or maybe mix and match different interfaces.</p>

<p>I don't know how many of you have actually tried running reinforcement learning by yourself. These are all extremely complex model customization techniques, and you're able to actually with a few clicks or a few lines of code run that and actually add real business value. These are all production recipes. We have a large team of scientists who are actually building these recipes so that customers such as you can actually use these and run these in production. So our goal here is to make all of the customization techniques as accessible to you as possible, and I hope you saw a glimpse of what this experience looks like. Please feel free to explore more of this on SageMaker Studio, and we have lots more to tell you folks. So feel free to reach out to us and we're happy to talk to you about it. Thank you so much.</p>




<p>; This article is entirely auto-generated using Amazon Bedrock.</p>

