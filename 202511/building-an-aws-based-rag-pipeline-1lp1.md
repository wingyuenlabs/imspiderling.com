---
Title: Building an AWS-Based RAG Pipeline
Description: 
Author: elbanic
Date: 2025-11-21T21:02:00.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Generational AI Blind Spot
</h2>

<p>We've got a fantastic new <strong>AI coding assistant</strong>, but when you ask it about your company's proprietary service architecture, it gives you a generic shrug (or worse, a confident hallucination). Why? Because our AI lives on the general internet, and our team's hard-won knowledge is locked behind firewalls and scattered across tools like <strong>Confluence, Slack, and internal documentation</strong>.</p>

<p>This is a pain point for everyone. As a service developer, we need that agent to understand the nuance of our codebase and team conventions. As a service operator, we can't afford to waste time retracing the steps of a solved production issue - we need the fastest, most reliable fix, instantly.</p>

<p>If you're interested in how to systematically manage context for AI coding assistants, check out my series on <a href="https://dev.to/elbanic/context-management-with-kiro-part-1-21hc">context management with Kiro</a> and <a href="https://dev.to/elbanic/context-bridging-the-ai-agents-blind-spot-38d6">Context - Bridging the AI Agent's Blind Spot</a>.</p>

<p><strong>User Stories</strong></p>

<ol>
<li>As a <strong>service developer</strong>, I want to provide context to my coding agent based on the knowledge my team possesses.</li>
<li>As a <strong>service operator</strong>, I want to find the right and shortest path to solving similar issues my team has previously encountered.</li>
</ol>

<h3>
  
  
  The Power of Retrieval Augmented Generation (RAG)
</h3>

<p>The bridge between general LLM intelligence and specific team expertise is <strong>Retrieval Augmented Generation (RAG)</strong>. This article details the construction of a robust, AWS-based RAG pipeline designed to eliminate those "blind spots." I'll walk you through the entire ETL process - extracting changes every six hours, cleaning sensitive info with Comprehend, and indexing for code structure (<strong>Neptune</strong>) and semantics (<strong>OpenSearch</strong>).</p>

<p>Our goal is simple: to create an AI agent that is not only intelligent but institutionally smart, giving software developers the right context to AI agents.</p>

<h2>
  
  
  Architecture the RAG Data Pipeline
</h2>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn4cejp4o87gy1ihx0gs4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn4cejp4o87gy1ihx0gs4.png" alt="RAG Pipeline Architecture" width="800" height="383"></a></p>

<p>Building a RAG system that delivers high-quality, up-to-date answers requires a disciplined and automated data pipeline. Our AWS-based RAG system is designed to periodically collect diverse internal materials - from wikis to Slack threads - and process, embed, and index them for the question answering engine.</p>

<p>The core of this system is an <strong>ETL (Extract, Transform, Load)</strong> workflow, triggered every six hours by <strong>AWS Step Functions</strong> or whenever the source data is updated.</p>

<ul>
<li>
<strong>Extract</strong>: Changes are incrementally synced from diverse data sources and saved to an <strong>S3 raw-data bucket</strong>.</li>
<li>
<strong>Transform/Process</strong>: Sensitive information (like PII) is detected and masked using <strong>Amazon Comprehend</strong>. Document formats are standardized and saved to an <strong>S3 processed-data bucket</strong>.</li>
<li>
<strong>Load/Index</strong>: Embeddings are generated, and the data is indexed in two places: <strong>Amazon OpenSearch Service</strong> for vector search and <strong>Amazon Neptune</strong> for code structure and relational information. If you'd like to see what these two databases can do, check out <a href="https://dev.to/elbanic/context-bridging-the-ai-agents-blind-spot-38d6">this article on context management</a>.</li>
</ul>

<h3>
  
  
  Data Sources and Incremental Ingestion
</h3>

<p>The ingestion stage focuses on efficiency by only processing what has changed. We pull from everywhere: internal wikis (Confluence, SharePoint), documents (Word/PDF), and collaboration tools (Slack/Teams message threads).</p>

<p>To handle large datasets efficiently, <strong>AWS Step Functions</strong> and <strong>Lambda functions/AWS Glue jobs</strong> are scheduled (via EventBridge) to incrementally sync <em>only</em> the content that has been changed or newly created since the last collection. This is critical for keeping the knowledge base fresh without wasting resources. We use APIs (like Confluence or Slack) to fetch content and store the results in an <strong>S3 raw-data bucket</strong>.</p>

<h3>
  
  
  Preprocessing and Transformation for Quality
</h3>

<p>The quality of your RAG answer is directly proportional to the quality of your source documents. This stage ensures a clean, consistent knowledge base.</p>

<ul>
<li>
<strong>Filtering and Cleansing</strong>: We use <strong>Amazon Comprehend's PII detection</strong> to automatically identify and mask sensitive items (names, emails) before they're embedded.</li>
<li>
<strong>Format Standardization</strong>: We convert various formats (Word, PDF) into plain text using tools like <strong>AWS Textract</strong>.</li>
<li>
<strong>Chunking</strong>: For optimal retrieval, the cleaned content is split into smaller, meaningful chunks (typically 200–500 tokens). Each chunk gets a unique ID and reference metadata, which is stored in the <strong>S3 processed-data bucket</strong>.</li>
</ul>

<h3>
  
  
  Dual Indexing: Vectors and Graphs
</h3>

<p>Effective RAG often requires more than just semantic similarity. By using two specialized databases, we can enrich the context with both meaning and structure.</p>

<ul>
<li>
<strong>Embedding Generation</strong>: We call models like <strong>Titan Text Embeddings models</strong> via <strong>Amazon Bedrock</strong> to generate the vector embedding for each document chunk.</li>
<li>
<strong>Vector DB (Semantic Search)</strong>: The generated embedding vectors are stored in the k-NN vector index of <strong>Amazon OpenSearch Service</strong>. This enables <strong>real-time vector search</strong> to find the most semantically similar document chunks relevant to the user's query.</li>
<li>
<strong>Graph DB (Structural Search)</strong>: Content with relationships - like source code, configuration files, or relational data - is stored in <strong>Amazon Neptune</strong>. This allows the RAG query to perform a <strong>knowledge graph query</strong> to retrieve associated entities, enriching the context with structural understanding.</li>
</ul>

<h3>
  
  
  Query Processing and Serving
</h3>

<p>This is where the user's question becomes a relevant, cited answer.</p>

<ol>
<li>
<strong>Query Reception (MCP server)</strong>: The query hits an <strong>AWS API Gateway</strong> endpoint, which is forwarded to a backend <strong>Lambda function</strong>.</li>
<li>
<strong>RAG Retrieval</strong>: The Lambda function calls the <strong>Bedrock RAG API (/retrieveAndGenerate)</strong>. It retrieves the top $k$ similar chunks from <strong>OpenSearch</strong> and simultaneously queries <strong>Neptune</strong> for additional structural context.</li>
<li>
<strong>Response Generation</strong>: All of this retrieved context (the document text chunks and key entity information) is passed in a single prompt to <strong>Claude Sonnet 4.5</strong> (Actually, you can use any LLM you want). The model synthesizes this evidence and the original question to formulate the final response. The response <strong>must</strong> include the source of information to ensure trustworthiness.</li>
</ol>

<h2>
  
  
  The Evolution of Context Engineering
</h2>

<p>We're still iterating to find the optimal way to develop a highly accurate agent. There will always be a gap when it comes to fully keeping up with human context and nuance. However, by defining tasks, breaking them into manageable units, and building a <strong>team-specific knowledge base</strong> that serves as a <strong>context engine</strong>, we can continuously evolve our agents to be more accurate, more reliable, and ultimately, more valuable to the developers and operators who rely on them.</p>

