---
Title: Anatomy of a RAG System Architecture
Description: 
Author: Mario García
Date: 2026-03-17T21:39:59.000Z
Robots: noindex,nofollow
Template: index
---
<p>For deploying a RAG System Architecture, consider that in a production environment requirements may vary when choosing a vector database, amount of data to be ingested, models used for creating embeddings, and architecture design when choosing a cloud platform. A RAG system can be built from scratch or implemented using solutions that already have the necessary components.</p>

<p>Following best practices is also critical when designing the system, to avoid common issues like hallucinations, or data exposure. Also consider that the model can be changed through the time, and using layer architecture may be helpful for future changes or updates.</p>

<h2>
  
  
  What is a RAG System?
</h2>

<p>Presenting false or inaccurate information when not knowing the response, using unvalidated sources, or giving outdated data are some of the challenges for LLMs. How to solve this, and improve the knowledge base? Retrieval-Augmented Generation Architecture (RAG) is the approach for solving this problem.</p>

<p>RAG uses some methods, and tools for acquiring knowledge from additional data sources. This information is then converted into a format that LLMs can understand, using different database technologies for saving this information, providing advanced search capabilities for data retrieval.</p>

<h2>
  
  
  RAG System Architecture Components
</h2>

<h3>
  
  
  Data Sources
</h3>

<p>RAG systems can have a variety of data sources from which to learn or acquire information. Those include but are not limited to:</p>

<ul>
<li>Documents in different formats (PDF, Word, TXT, Markdown)</li>
<li>Images</li>
<li>Audio or transcriptions</li>
<li>Datasets (CSV, JSON)</li>
<li>Application Programming Interfaces (APIs)</li>
<li>Relational databases (SQL)</li>
<li>NoSQL databases</li>
<li>Wikis or knowledge base</li>
<li>Web content</li>
<li>Internal files</li>
</ul>

<p>Data ingestion in a RAG system goes through a process where:</p>

<ul>
<li>Content is extracted from the data sources</li>
<li>Divided into chunks</li>
<li>Converted to embeddings</li>
<li>Stored in a vector database</li>
</ul>

<h3>
  
  
  Embedding Model
</h3>

<p>An embedding model is a machine learning model used for transforming data sources into vectors. Also known as embeddings, and are a series of float values representing the meaning of the data source. The model determines not only how embeddings are created but also the size of the vector, and is mainly provided by LLMs.</p>

<p>For apps built with Python use Gemini or OpenAI models through their official SDKs:</p>

<ul>
<li>OpenAI Python AI library → <a href="https://github.com/openai/openai-python" rel="noopener noreferrer">github.com/openai/openai-python</a>
</li>
<li>Google GenAI Python SDK → <a href="https://github.com/googleapis/python-genai" rel="noopener noreferrer">github.com/googleapis/python-genai</a>
</li>
</ul>

<p>Another option is to use <a href="https://github.com/langchain-ai/langchain" rel="noopener noreferrer">LangChain</a>, a framework for building agents, and LLM-powered applications. Requests to those models are sent through this framework.</p>

<p>LLMs can also be executed locally without sending data to the cloud, and is possible by using Open Source tools like <a href="https://ollama.com/" rel="noopener noreferrer">Ollama</a>. Models are downloaded from its own <a href="https://ollama.com/library" rel="noopener noreferrer">catalog</a>.</p>

<p>A similar library is <a href="https://sbert.net/" rel="noopener noreferrer">Sentence Transformers</a>. It has access to the <a href="https://huggingface.co/models" rel="noopener noreferrer">Hugging Face</a> catalog from which models are downloaded to use locally, generally via <a href="https://pytorch.org/" rel="noopener noreferrer">PyTorch</a>, without having to create an API key.</p>

<h3>
  
  
  Vector Database
</h3>

<p>In a vector database, information is stored in vectors, better known as embeddings, containing numerical values that represent the meaning of the stored data. Query results are determined by the values in the vectors, returning nearest vectors.</p>

<p>Having this text: <code>"Open source software is transforming the technology ecosystem.”</code>, when converting the value to an embedding, will look like this: <code>[-0.007894928, 0.0010742444, -0.03274113, -0.066677086, 0.004894953, 0.013967406, 0.0068471087, 0.009695383, 0.020278132, 0.019470839]</code>.</p>

<p>Which vector database to use? Depends on the requirements of the project, experience using vector databases, or how the RAG system is being implemented. If the system is being built from the ground up, the best option is the one that serves the needs of the project, in terms of requirements, and budget. Choose solutions like <a href="https://ragflow.io/" rel="noopener noreferrer">RAGFlow</a> that already provide all the necessary components to implement the system when time plays a key role.</p>

<p>A few options to consider when choosing a vector database:</p>

<ul>
<li>
<a href="https://github.com/pgvector/pgvector" rel="noopener noreferrer"><strong>pgvector</strong></a>: An extension for PostgreSQL that adds the vector datatype, and search capabilities found in vector databases. The extension can be installed manually by compiling the source code from the official repository, or deployed via Docker or Kubernetes, as well as on cloud platforms that support PostgreSQL like Amazon Web Services (AWS), Google Cloud Platform (GCP) or Supabase.</li>
<li>
<a href="https://www.pinecone.io/" rel="noopener noreferrer"><strong>Pinecone</strong></a>: A vector database designed for scalability, and to be used in production. Run as a managed service in AWS, GCP, and Azure. Every request to Pinecone is sent through an API that routes it to a control plane or a data plane, for managing projects, and indexes; reading, and writing data respectively.</li>
<li>
<a href="https://weaviate.io/" rel="noopener noreferrer"><strong>Weaviate</strong></a>: An Open Source vector database. Can be configured via the Weaviate Cloud, using a cloud provider like GCP, to deploy an instance of the database. It uses an API key for authentication, and sending requests. Docker image is available for running locally.</li>
<li>
<a href="https://github.com/elastic/elasticsearch" rel="noopener noreferrer"><strong>Elasticsearch</strong></a>: Distributed, and analytics search engine, scalable data store, vector database focused on speed, production-ready, and scalability support. RAGFlow component, used as its default vector database, while testing, and improving Infinity. Can be used in production environments through Elastic Cloud or configured manually in the cloud platform of choice. A Docker image is available to use it locally.</li>
</ul>

<h2>
  
  
  Challenges in RAG System Architecture Deployment
</h2>

<p>Ready to deploy the RAG system? Some challenges may be encountered when designing the architecture, and preparing the system to deploy in a production environment.</p>

<p>A few challenges to take into consideration:</p>

<ul>
<li>
<strong>Hallucinations</strong>: Occur when a model in a RAG system answers with unexpected information due to poor prompt engineering, passing irrelevant chunks, or weak context selection. To fix it, use templates to tell the model how to answer when no information is found, validate periodically the data in the knowledge base to check for outdated or wrong information.</li>
<li>
<strong>Data Exposure &amp; Prompt Injection</strong>: When adopting RAG systems, some security issues are expected to happen if the architecture has a poor design. These issues include exposing sensitive data or tricking the model to bypass the rules when receiving malicious prompts. A way to avoid it is by applying input/output sanitization, and setting up guardrails.</li>
</ul>

<h2>
  
  
  Best Practices for RAG Deployment
</h2>

<p>Data Exposure &amp; Prompt Injection: When adopting RAG systems, some security issues are expected to happen if the architecture has a poor design. These issues include exposing sensitive data or tricking the model to bypass the rules when receiving malicious prompts. A way to avoid it is by applying input/output sanitization, and setting up guardrails.</p>

<ul>
<li>
<strong>Decouple the Retrieval layer from the Generation layer</strong>: Following a layer approach when designing the system could help reduce hallucinations, maintain the information updated, and upgrades can be applied independently. The Retrieval layer is when data ingestion, embedding conversion, chunk division, and information storing happen, as well as query conversion into vectors for semantic search. While the Generation layer is where the retrieved information, and the original prompt are taken to produce a human-like response.</li>
<li>
<strong>Embedding model replaceability</strong>: To prevent vendor lock-in, and allow upgrades to better models, design the system to enable embedding model replacement without having to make critical changes or having to rewrite the implementation. How to get it? Create a wrapper function to change between models, use a framework like LangChain instead of using individual SDKs, create metadata with model ID, and version, separate ingestion from retrieval, create a dataset for benchmarking that is constantly evaluated to determine when to change the model, and normalize data before embedding.</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>Each component in a RAG system architecture is key for the system to work, and provides better answers by improving the knowledge base, but depending on the requirements, and budget of the project, embedding models, vector databases, and cloud platforms are chosen. Already familiar with PostgreSQL? Choose  pgvector, otherwise evaluate other solutions. Planning to build the system from scratch? Consider using Python for implementing the system, as there are frameworks, and libraries available for this language. And don’t forget to follow the recommendations.</p>

