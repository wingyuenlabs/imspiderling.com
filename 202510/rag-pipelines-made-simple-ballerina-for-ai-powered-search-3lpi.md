---
Title: RAG Pipelines Made Simple: Ballerina for AI-Powered Search
Description: 
Author: Abhi
Date: 2025-10-28T21:53:26.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>Why building production-ready RAG systems shouldn't feel like assembling IKEA furniture in the dark</em></p>




<h2>
  
  
  The RAG Reality Check
</h2>

<p>Picture this: You've just convinced your team to build an AI-powered documentation search. The plan is simple—take your company's docs, throw them into a vector database, add some LLM magic, and boom: instant answers for your users.</p>

<p>Three weeks later, you're drowning in boilerplate. Your codebase looks like a Frankenstein monster stitched together from five different libraries. There's error handling scattered everywhere (or worse, nowhere). You're manually chunking documents, wrestling with embedding APIs, debugging vector similarity algorithms, and somehow your "simple" RAG pipeline has morphed into 2,000 lines of glue code.</p>

<p>Sound familiar?</p>

<p>This is where Ballerina walks in, takes a look at your mess, and says: <em>"Why are you working so hard?"</em></p>




<h2>
  
  
  What Makes RAG Systems Complex?
</h2>

<p>Before we dive into the solution, let's break down why RAG pipelines are deceptively complex:</p>

<h3>
  
  
  The RAG Dance (and Why It's Tricky)
</h3>

<p>A typical RAG workflow involves:</p>

<ol>
<li>
<strong>Document Loading</strong>: Read files from various sources (PDFs, markdown, databases)</li>
<li>
<strong>Chunking Strategy</strong>: Split documents intelligently without losing context</li>
<li>
<strong>Embedding Generation</strong>: Convert text chunks into vectors using an embedding model</li>
<li>
<strong>Vector Storage</strong>: Store embeddings efficiently with metadata</li>
<li>
<strong>Query Processing</strong>: When a user asks a question, embed their query</li>
<li>
<strong>Similarity Search</strong>: Find the most relevant document chunks</li>
<li>
<strong>Context Augmentation</strong>: Combine retrieved chunks with the user's query</li>
<li>
<strong>LLM Generation</strong>: Send the enriched prompt to an LLM for a final answer</li>
</ol>

<p>In traditional frameworks, each of these steps requires:</p>

<ul>
<li>Choosing and configuring multiple libraries</li>
<li>Writing integration code between incompatible APIs</li>
<li>Manual error handling at every boundary</li>
<li>Custom observability instrumentation</li>
<li>Performance optimization from scratch</li>
</ul>

<p>Most developers spend 70% of their time on plumbing and only 30% on actual AI logic.</p>




<h2>
  
  
  Enter Ballerina: Network-Native Meets AI-Native
</h2>

<p>Ballerina was designed with a radical idea: <em>What if distributed systems programming was actually... easy?</em></p>

<p>Now, with Ballerina's AI module, this philosophy extends to RAG pipelines. Instead of stitching together separate libraries for embeddings, vector stores, chunking, and LLMs, you get a unified, type-safe API that handles the complexity for you.</p>

<h3>
  
  
  The "Aha!" Moment
</h3>

<p>Let's compare building a RAG system in two approaches:</p>

<p><strong>Traditional Approach (Python + Multiple Libraries):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="n">langchain.document_loaders</span> <span class="kn">import</span> <span class="n">TextLoader</span>
<span class="kn">from</span> <span class="n">langchain.text_splitter</span> <span class="kn">import</span> <span class="n">RecursiveCharacterTextSplitter</span>
<span class="kn">from</span> <span class="n">langchain.embeddings</span> <span class="kn">import</span> <span class="n">OpenAIEmbeddings</span>
<span class="kn">from</span> <span class="n">langchain.vectorstores</span> <span class="kn">import</span> <span class="n">Chroma</span>
<span class="kn">from</span> <span class="n">langchain.llms</span> <span class="kn">import</span> <span class="n">OpenAI</span>
<span class="kn">import</span> <span class="n">os</span>

<span class="c1"># Configure everything separately
</span><span class="n">loader</span> <span class="o">=</span> <span class="nc">TextLoader</span><span class="p">(</span><span class="sh">"</span><span class="s">./leave_policy.md</span><span class="sh">"</span><span class="p">)</span>
<span class="n">documents</span> <span class="o">=</span> <span class="n">loader</span><span class="p">.</span><span class="nf">load</span><span class="p">()</span>

<span class="n">text_splitter</span> <span class="o">=</span> <span class="nc">RecursiveCharacterTextSplitter</span><span class="p">(</span>
    <span class="n">chunk_size</span><span class="o">=</span><span class="mi">1000</span><span class="p">,</span>
    <span class="n">chunk_overlap</span><span class="o">=</span><span class="mi">200</span>
<span class="p">)</span>
<span class="n">chunks</span> <span class="o">=</span> <span class="n">text_splitter</span><span class="p">.</span><span class="nf">split_documents</span><span class="p">(</span><span class="n">documents</span><span class="p">)</span>

<span class="n">embeddings</span> <span class="o">=</span> <span class="nc">OpenAIEmbeddings</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">OPENAI_KEY</span><span class="sh">"</span><span class="p">))</span>
<span class="n">vectorstore</span> <span class="o">=</span> <span class="n">Chroma</span><span class="p">.</span><span class="nf">from_documents</span><span class="p">(</span><span class="n">chunks</span><span class="p">,</span> <span class="n">embeddings</span><span class="p">)</span>

<span class="n">llm</span> <span class="o">=</span> <span class="nc">OpenAI</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">OPENAI_KEY</span><span class="sh">"</span><span class="p">))</span>

<span class="c1"># Now try to make them work together...
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"</span><span class="s">How many leave days can I carry forward?</span><span class="sh">"</span>
<span class="n">docs</span> <span class="o">=</span> <span class="n">vectorstore</span><span class="p">.</span><span class="nf">similarity_search</span><span class="p">(</span><span class="n">query</span><span class="p">,</span> <span class="n">k</span><span class="o">=</span><span class="mi">10</span><span class="p">)</span>
<span class="n">context</span> <span class="o">=</span> <span class="sh">"</span><span class="se">\n</span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">([</span><span class="n">doc</span><span class="p">.</span><span class="n">page_content</span> <span class="k">for</span> <span class="n">doc</span> <span class="ow">in</span> <span class="n">docs</span><span class="p">])</span>
<span class="n">prompt</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">Context: </span><span class="si">{</span><span class="n">context</span><span class="si">}</span><span class="se">\n\n</span><span class="s">Question: </span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="sh">"</span>
<span class="n">answer</span> <span class="o">=</span> <span class="nf">llm</span><span class="p">(</span><span class="n">prompt</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Ballerina Approach:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import ballerina/ai;
import ballerina/io;

// Initialize the RAG system
final ai:VectorStore vectorStore = check new ai:InMemoryVectorStore();
final ai:EmbeddingProvider embeddingProvider = 
    check ai:getDefaultEmbeddingProvider();
final ai:KnowledgeBase knowledgeBase = 
    new ai:VectorKnowledgeBase(vectorStore, embeddingProvider);
final ai:ModelProvider modelProvider = check ai:getDefaultModelProvider();

public function main() returns error? {
    // Load and ingest documents
    ai:DataLoader loader = check new ai:TextDataLoader("./leave_policy.md");
    ai:Document|ai:Document[] documents = check loader.load();
    check knowledgeBase.ingest(documents);

    // Query and get answer
    string query = "How many leave days can I carry forward?";
    ai:QueryMatch[] matches = check knowledgeBase.retrieve(query, 10);
    ai:Chunk[] context = from ai:QueryMatch match in matches 
                         select match.chunk;

    ai:ChatUserMessage augmentedQuery = ai:augmentUserQuery(context, query);
    ai:ChatAssistantMessage answer = check modelProvider-&gt;chat(augmentedQuery);

    io:println("Answer: ", answer.content);
}
</code></pre>

</div>



<p>Notice what's <strong>different</strong>:</p>

<ul>
<li>No manual chunking configuration (handled intelligently by default)</li>
<li>No explicit embedding API calls (abstracted away)</li>
<li>No prompt engineering boilerplate (built-in augmentation)</li>
<li>No manual context concatenation (query expressions handle it)</li>
<li>Clean error handling with <code>check</code> keyword</li>
<li>Everything is type-safe</li>
</ul>




<h2>
  
  
  Building a Real RAG System: Employee Handbook Assistant
</h2>

<p>Let's build something practical: an AI assistant that answers questions about your company's employee handbook. We'll cover the complete journey from document ingestion to production deployment.</p>

<h3>
  
  
  The Setup: Configuration Made Easy
</h3>

<p>First, let's understand Ballerina's provider pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import ballerina/ai;
import ballerina/io;

// The VectorStore holds your embeddings
final ai:VectorStore vectorStore = check new ai:InMemoryVectorStore();

// EmbeddingProvider converts text to vectors
// Uses your configured provider (OpenAI, Cohere, etc.) via Ballerina VS Code command
final ai:EmbeddingProvider embeddingProvider = 
    check ai:getDefaultEmbeddingProvider();

// KnowledgeBase combines storage + embeddings
final ai:KnowledgeBase knowledgeBase = 
    new ai:VectorKnowledgeBase(vectorStore, embeddingProvider);

// ModelProvider handles LLM chat completions
final ai:ModelProvider modelProvider = check ai:getDefaultModelProvider();
</code></pre>

</div>



<p><strong>What's happening here?</strong></p>

<p>Ballerina uses a configuration-based approach. Instead of hardcoding API keys and model names, you configure providers through Ballerina's VS Code extension or configuration files. This means:</p>

<ul>
<li>No secrets in code</li>
<li>Easy switching between providers (OpenAI, Azure, local models)</li>
<li>Environment-specific configurations</li>
<li>Type-safe provider interfaces</li>
</ul>

<p>Note: This example uses the default embedding provider and model provider implementations. To generate the necessary configuration, open up the VS Code command palette (Ctrl + Shift + P or command + shift + P), and run the Configure default WSO2 Model Provider command to add your configuration to the Config.toml file.</p>

<h3>
  
  
  Step 1: Document Ingestion - Simpler Than You Think
</h3>

<p>The hardest part of building a RAG system is usually getting data in. Ballerina's <code>DataLoader</code> abstractions make this trivial:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>public function ingestEmployeeHandbook() returns error? {
    // Load a single document
    ai:DataLoader policyLoader = check new ai:TextDataLoader("./leave_policy.md");
    ai:Document|ai:Document[] policyDocs = check policyLoader.load();
    check knowledgeBase.ingest(policyDocs);
    io:println("✅ Leave policy ingested");

    // Load multiple documents from a directory
    ai:DataLoader benefitsLoader = check new ai:TextDataLoader("./benefits/");
    ai:Document|ai:Document[] benefitsDocs = check benefitsLoader.load();
    check knowledgeBase.ingest(benefitsDocs);
    io:println("✅ Benefits documentation ingested");

    // You can also load PDFs, Word docs, etc.
    ai:DataLoader handbookLoader = check new ai:PDFDataLoader("./handbook.pdf");
    ai:Document|ai:Document[] handbookDocs = check handbookLoader.load();
    check knowledgeBase.ingest(handbookDocs);
    io:println("✅ Employee handbook ingested");
}
</code></pre>

</div>



<p><strong>The Magic Behind the Scenes:</strong></p>

<p>When you call <code>knowledgeBase.ingest()</code>, Ballerina automatically:</p>

<ol>
<li>
<strong>Chunks your documents</strong> using intelligent text splitting (respects sentence boundaries, maintains context)</li>
<li>
<strong>Generates embeddings</strong> for each chunk using your configured embedding provider</li>
<li>
<strong>Stores vectors</strong> with metadata in the vector store</li>
<li>
<strong>Handles errors</strong> gracefully with proper propagation</li>
</ol>

<p>No manual chunking. No explicit embedding calls. Just load and ingest.</p>

<h3>
  
  
  Step 2: The Query Pipeline - Where RAG Shines
</h3>

<p>Now comes the fun part: answering questions with context from your documents.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>type RAGResponse record {|
    string answer;
    Source[] sources;
    float averageRelevance;
|};

type Source record {|
    string content;
    float relevance;
|};

function answerQuestion(string query, int topK = 5) returns RAGResponse|error {
    io:println("\n🔍 Query: ", query);

    // Step 1: Retrieve relevant chunks from knowledge base
    ai:QueryMatch[] queryMatches = check knowledgeBase.retrieve(query, topK);

    // Step 2: Extract chunks for context
    ai:Chunk[] context = from ai:QueryMatch queryMatch in queryMatches
                         select queryMatch.chunk;

    // Step 3: Augment user query with retrieved context
    ai:ChatUserMessage augmentedQuery = ai:augmentUserQuery(context, query);

    // Step 4: Get answer from LLM
    ai:ChatAssistantMessage assistantMessage = 
        check modelProvider-&gt;chat(augmentedQuery);

    // Step 5: Calculate average relevance for confidence scoring
    float totalRelevance = 0.0;
    foreach ai:QueryMatch match in queryMatches {
        totalRelevance += match.score;
    }
    float avgRelevance = queryMatches.length() &gt; 0 ? 
        totalRelevance / queryMatches.length() : 0.0;

    // Step 6: Build response with sources
    Source[] sources = from ai:QueryMatch match in queryMatches
                       select {
                           content: match.chunk.content,
                           relevance: match.score
                       };

    return {
        answer: assistantMessage.content,
        sources: sources,
        averageRelevance: avgRelevance
    };
}
</code></pre>

</div>



<p><strong>Breaking Down the Query Expression:</strong></p>

<p>One of Ballerina's superpowers is query expressions. Look at this beauty:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ai:Chunk[] context = from ai:QueryMatch queryMatch in queryMatches
                     select queryMatch.chunk;
</code></pre>

</div>



<p>This is not just syntactic sugar. It's a type-safe, functional way to transform data that:</p>

<ul>
<li>Reads like SQL</li>
<li>Compiles to efficient code</li>
<li>Maintains type safety throughout the pipeline</li>
<li>Makes data transformations explicit and readable</li>
</ul>

<p>Compare this to Python's list comprehension or JavaScript's map—it's clearer and more maintainable.</p>

<h3>
  
  
  Step 3: Understanding Query Augmentation
</h3>

<p>The <code>ai:augmentUserQuery()</code> function is doing heavy lifting behind the scenes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ai:ChatUserMessage augmentedQuery = ai:augmentUserQuery(context, query);
</code></pre>

</div>



<p>This function takes your retrieved chunks and combines them with the user's query using a well-designed prompt template. It's essentially doing:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>You are a helpful assistant. Answer the question based on the following context.

Context:
[Chunk 1 content]
[Chunk 2 content]
[Chunk 3 content]

Question: {user's query}

Answer:
</code></pre>

</div>



<p>But you don't have to worry about prompt engineering—Ballerina handles it with best practices baked in.</p>

<h3>
  
  
  Step 4: Making it Production-Ready with HTTP Service
</h3>

<p>Let's expose our RAG system as a REST API:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import ballerina/http;
import ballerina/ai;
import ballerina/io;

// Initialize RAG components (same as before)
final ai:VectorStore vectorStore = check new ai:InMemoryVectorStore();
final ai:EmbeddingProvider embeddingProvider = 
    check ai:getDefaultEmbeddingProvider();
final ai:KnowledgeBase knowledgeBase = 
    new ai:VectorKnowledgeBase(vectorStore, embeddingProvider);
final ai:ModelProvider modelProvider = check ai:getDefaultModelProvider();

type QueryRequest record {|
    string question;
    int topK = 5;
|};

type QueryResponse record {|
    string answer;
    Source[] sources;
    float confidence;
|};

type Source record {|
    string content;
    float relevance;
|};

service /api on new http:Listener(8080) {

    // Initialize knowledge base on service startup
    function init() returns error? {
        io:println("🚀 Starting Employee Handbook Assistant...");

        // Ingest documents
        ai:DataLoader loader = check new ai:TextDataLoader("./documents/");
        ai:Document|ai:Document[] documents = check loader.load();
        check knowledgeBase.ingest(documents);

        io:println("✅ Knowledge base initialized");
    }

    // Query endpoint
    resource function post query(@http:Payload QueryRequest request) 
            returns QueryResponse|http:InternalServerError {

        // Retrieve relevant chunks
        ai:QueryMatch[]|error queryMatches = 
            knowledgeBase.retrieve(request.question, request.topK);

        if queryMatches is error {
            return {
                body: {
                    message: "Failed to retrieve context",
                    error: queryMatches.message()
                }
            };
        }

        // Extract context chunks
        ai:Chunk[] context = from ai:QueryMatch match in queryMatches
                             select match.chunk;

        // Augment query and get answer
        ai:ChatUserMessage augmentedQuery = 
            ai:augmentUserQuery(context, request.question);
        ai:ChatAssistantMessage|error assistantMessage = 
            modelProvider-&gt;chat(augmentedQuery);

        if assistantMessage is error {
            return {
                body: {
                    message: "Failed to generate answer",
                    error: assistantMessage.message()
                }
            };
        }

        // Calculate confidence
        float totalRelevance = 0.0;
        foreach ai:QueryMatch match in queryMatches {
            totalRelevance += match.score;
        }
        float confidence = queryMatches.length() &gt; 0 ? 
            totalRelevance / queryMatches.length() : 0.0;

        // Build sources
        Source[] sources = from ai:QueryMatch match in queryMatches
                           select {
                               content: match.chunk.content,
                               relevance: match.score
                           };

        return {
            answer: assistantMessage.content,
            sources: sources,
            confidence: confidence
        };
    }

    // Health check endpoint
    resource function get health() returns json {
        return {
            status: "healthy",
            service: "Employee Handbook Assistant",
            timestamp: time:utcNow()
        };
    }

    // Add new documents endpoint
    resource function post documents(http:Request req) 
            returns http:Created|http:BadRequest|http:InternalServerError {

        string|http:ClientError filePath = req.getTextPayload();

        if filePath is http:ClientError {
            return {body: "Invalid file path"};
        }

        ai:DataLoader|error loader = new ai:TextDataLoader(filePath);
        if loader is error {
            return {body: "Failed to create data loader"};
        }

        ai:Document|ai:Document[]|error documents = loader.load();
        if documents is error {
            return {body: "Failed to load documents"};
        }

        error? result = knowledgeBase.ingest(documents);
        if result is error {
            return {body: "Failed to ingest documents"};
        }

        return {
            body: {message: "Documents ingested successfully"}
        };
    }
}
</code></pre>

</div>



<p><strong>What Makes This Special:</strong></p>

<ol>
<li>
<strong>Automatic JSON Binding</strong>: The <code>@http:Payload</code> annotation automatically deserializes JSON to Ballerina records</li>
<li>
<strong>Type-Safe Responses</strong>: Return types are checked at compile time</li>
<li>
<strong>Built-in Error Handling</strong>: Union types (<code>|error</code>) force you to handle failures</li>
<li>
<strong>Clean Service Definition</strong>: RESTful endpoints with clear resource functions</li>
</ol>

<h2>
  
  
  Learn More
</h2>

<ul>
<li><a href="https://ballerina.io/" rel="noopener noreferrer">Ballerina Official Documentation</a></li>
<li><a href="https://ballerina.io/use-cases/ai/" rel="noopener noreferrer">Ballerina AI</a></li>
</ul>

