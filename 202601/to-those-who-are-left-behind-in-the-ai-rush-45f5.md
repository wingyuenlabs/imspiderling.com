---
Title: To those who are left behind in the AI rush
Description: 
Author: Mariam Reba Alexander
Date: 2026-01-31T21:16:32.000Z
Robots: noindex,nofollow
Template: index
---
<p>Ever feel overwhelmed by AI posts on every social media and professional platform? And don't understand the AI jargon, buzzwords, and concepts being discussed? I am one of them, and here is a cheat sheet summarizing the key points to help you get up to speed.</p>

<h2>
  
  
  Core Concepts
</h2>

<ul>
<li>
<strong>NLP (Natural Language Processing)</strong>: The broad field of AI that teaches computers to understand, interpret, and generate human language.</li>
<li>
<strong>LLM (Large Language Model)</strong>: A type of AI trained on massive amounts of text to predict the next word in a sequence, allowing it to chat, write, and summarize.</li>
<li>
<strong>LMM (Large Multimodal Model)</strong>: An advanced version of an LLM that can understand more than just text, such as images, audio, and video.</li>
<li>
<strong>Foundational Model</strong>: A massive, "general purpose" AI model (like GPT-4) trained on huge data that serves as a base to be adapted for many different specific tasks.</li>
<li>
<strong>Multimodal</strong>: The ability of an AI to process and "connect" different types of data (modes) at once, like seeing an image and describing it in text. </li>
</ul>

<h2>
  
  
  Practical Techniques
</h2>

<ul>
<li>
<strong>Prompt Engineering</strong>: The art of crafting specific instructions or "prompts" to get the best possible answer from an AI model.</li>
<li>
<strong>Finetune</strong>: Taking a pre-trained model and giving it extra training on a smaller, specific dataset to make it an expert in a particular subject or style.</li>
<li>
<strong>RAG (Retrieval-Augmented Generation)</strong>: Connecting an AI to a trusted external database (like your company’s files) so it can look up real-time facts before answering, reducing "hallucinations".</li>
<li>
<strong>Embedding Model</strong>: A tool that converts text or images into a list of numbers (vectors) that represent their meaning, allowing computers to compare how "similar" two ideas are. </li>
</ul>

<h2>
  
  
  Advanced AI &amp; Engineering
</h2>

<ul>
<li>
<strong>AI Engineering</strong>: The practice of building complete applications using ready-made AI models (like foundation models). Instead of training a model from scratch, an AI engineer focuses on integrating these models into software, ensuring they are secure, cost-effective, and useful for the user.</li>
<li>
<strong>ML Engineering (Machine Learning Engineering)</strong>: A specialized role focused on creating and improving the models themselves. This involves training algorithms from scratch, cleaning data, and optimizing mathematical performance.</li>
<li>
<strong>Agentic AI</strong>: AI that doesn't just talk but can act. It can break a big goal into steps, use tools (like searching the web or booking a flight), and complete tasks autonomously.</li>
<li>
<strong>MLOps (Machine Learning Operations)</strong>: The "factory" side of AI; it’s a set of practices to ensure AI models are updated, monitored, and running reliably in the real world.</li>
</ul>

<h2>
  
  
  How the Roles Work Together
</h2>

<p>In a real-world project (like building a customer support agent), the roles collaborate in a "pipeline": </p>

<ul>
<li>
<strong>ML Engineer</strong>: Designs and trains the "brain." They might build a custom embedding model from scratch so the company’s specific technical jargon is understood perfectly.</li>
<li>
<strong>AI Engineer</strong>: Connects the brain to the world. They take a foundational model (like GPT-4), set up RAG so it can read private manuals, and use prompt engineering to ensure it stays polite.</li>
<li>
<strong>MLOps Engineer</strong>: Builds the "factory." they ensure that if 10,000 customers use the app at once, the servers don't crash, and they monitor the model to make sure it doesn't start giving wrong answers over time. </li>
</ul>

<h2>
  
  
  Essential AI Engineering Tools
</h2>

<p>AI Engineers today rely on a specific stack to move fast without building everything from scratch: </p>

<ul>
<li>
<strong>Orchestration (Linking things together)</strong>: LangChain and LlamaIndex are the "glue" used to build complex workflows and Agentic AI.</li>
<li>
<strong>Vector Databases (For RAG)</strong>: Pinecone, Weaviate, and Chroma store the embeddings that allow an AI to "search" through your documents.</li>
<li>
<strong>App Builders &amp; UI</strong>: Streamlit and Gradio are used to quickly create chat interfaces or dashboards for the AI.</li>
<li>
<strong>Evaluation &amp; Observability</strong>: LangSmith and Arize Phoenix help engineers "see" into the AI's thoughts to debug why it might be hallucinating.</li>
<li>
<strong>Local Development</strong>: Ollama and LM Studio allow engineers to run powerful LLMs directly on their own laptops for private testing</li>
</ul>

<h2>
  
  
  Catching up on the tools and models in the industry
</h2>

<p>To help you catch up on the rapid evolution of AI, here is a breakdown of the leading tools and models currently dominating the industry.</p>

<ol>
<li>
<strong>Foundational Models</strong>:
These are general-purpose engines used as the basis for other applications. 

<ul>
<li>
<strong>Text &amp; General Reasoning</strong>: OpenAI's GPT-4o, Anthropic’s Claude 3.5 Sonnet, Google’s Gemini 1.5 Pro, and Meta’s Llama 3 are examples of text and general reasoning models. Llama 3 is a leader in open-source models.</li>
<li>
<strong>Image Generation</strong>: Stable Diffusion and Midjourney are examples of image generation models.</li>
<li>
<strong>Video Generation</strong>: Synthesia (for talking heads) and Google Veo (for cinematic clips) are used for video generation. </li>
</ul>
</li>
<li>
<strong>AI Applications &amp; Assistants</strong>:
These are ready-to-use products designed to improve productivity. 

<ul>
<li>
<strong>Coding Assistants</strong>: GitHub Copilot is a standard tool for real-time coding assistance.</li>
<li>
<strong>Meeting Automation</strong>: Fireflies.ai records, transcribes, and summarizes meetings automatically.</li>
<li>
<strong>Search &amp; Research</strong>: Perplexity AI is a conversational search engine that cites sources.</li>
<li>
<strong>Workflow Assistants</strong>: Lindy can execute multi-step business workflows across different apps. </li>
</ul>
</li>
<li>
<strong>RAG Tools</strong>:
These tools help AI interact with private data. 

<ul>
<li>
<strong>Frameworks</strong>: LlamaIndex specializes in indexing and retrieving data. LangChain is popular for building application logic.</li>
<li>
<strong>Vector Databases</strong>: Pinecone (managed cloud), Weaviate (open-source), and Chroma store the "mathematical meaning" of documents for search.</li>
<li>
<strong>Evaluation</strong>: LangSmith and Arize Phoenix help with debugging AI responses. </li>
</ul>
</li>
<li>
<strong>Agentic AI</strong>:
These frameworks enable AI to perform autonomous tasks. 

<ul>
<li>
<strong>Multi-Agent Teams</strong>: CrewAI and Microsoft’s AutoGen allow the creation of AI agent teams with specific roles.</li>
<li>
<strong>Complex Workflows</strong>: LangGraph provides control over how an agent performs a task, including human-approval steps.</li>
<li>
<strong>Autonomous Coding</strong>: Goose and Claude Code can edit files and run tests. </li>
</ul>
</li>
<li>
<strong>Other Important Engineering Tools</strong>:

<ul>
<li>
<strong>Local AI Runners</strong>: Ollama is a tool for running open-source models like Llama 3 on a personal computer.</li>
<li>
<strong>Model Optimization</strong>: DSPy optimizes prompts automatically.</li>
</ul>
</li>
</ol>

