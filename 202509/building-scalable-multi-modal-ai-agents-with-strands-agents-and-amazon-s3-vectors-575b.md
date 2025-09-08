---
Title: Building Scalable Multi-Modal AI Agents with Strands Agents and Amazon S3 Vectors
Description: 
Author: Elizabeth Fuentes L
Date: 2025-09-08T22:07:53.000Z
Robots: noindex,nofollow
Template: index
---
<p>🇻🇪🇨🇱 <a href="https://dev.to/elizabethfuentes12">Dev.to</a> <a href="https://www.linkedin.com/in/lizfue/" rel="noopener noreferrer">Linkedin</a> <a href="https://github.com/elizabethfuentes12/" rel="noopener noreferrer">GitHub</a> <a href="https://twitter.com/elizabethfue12" rel="noopener noreferrer">Twitter</a> <a href="https://www.instagram.com/elifue.tech" rel="noopener noreferrer">Instagram</a> <a href="https://www.youtube.com/channel/UCr0Gnc-t30m4xyrvsQpNp2Q" rel="noopener noreferrer">Youtube</a><br>
<a href="https://linktr.ee/elizabethfuentesleone" rel="noopener noreferrer">Linktr</a></p>


<div class="ltag__user ltag__user__id__717518">
    <a href="/elizabethfuentes12" class="ltag__user__link profile-image-link">
      <div class="ltag__user__pic">
        <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F717518%2Fb550b165-b8b9-405d-acfb-e5dc846765b0.png" alt="elizabethfuentes12 image">
      </div>
    </a>
  <div class="ltag__user__content">
    <h2>
<a class="ltag__user__link" href="/elizabethfuentes12">Elizabeth Fuentes L</a>Follow
</h2>
    <div class="ltag__user__summary">
      <a class="ltag__user__link" href="/elizabethfuentes12">AWS Developer Advocate</a>
    </div>
  </div>
</div>


<p>GitHub repositorie: <a href="https://github.com/elizabethfuentes12/strands-agent-samples" rel="noopener noreferrer">Strands Agent Samples</a></p>

<h2>
  
  
  Part 3: Adding Persistent Memory with Amazon S3 Vector Store
</h2>

<p>Building sophisticated AI agents doesn't have to be complex. As we demonstrated in my <a href="https://dev.to/aws/multi-modal-content-processing-with-strands-agent-and-just-a-few-lines-of-code-4hn4">first blog post</a>, the <a href="https://strandsagents.com/latest/" rel="noopener noreferrer">Strands Agent</a> open source  framework makes it remarkably simple to create multi-modal AI agents with just a few lines of code. Whether you're processing images, documents, or videos, Strands maintains this simplicity while providing powerful capabilities.</p>

<p>This post continues my AI agent development series, building on our previous exploration of multi-modal AI agents with the <a href="https://dev.to/aws/multi-modal-content-processing-with-strands-agent-and-faiss-memory-39hg">Strands Agent framework using FAISS for local memory storage</a>. Now, we advance to enterprise-scale capabilities with <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-vectors.html" rel="noopener noreferrer"><strong>Amazon S3 Vectors</strong></a> –  the AWS object storage service with native vector support that transforms how you build scalable AI applications <strong>without sacrificing the simplicity that makes Strands Agent so accessible</strong>.</p>

<h2>
  
  
  From Local to Cloud: Production-Ready AI Memory
</h2>

<p>While FAISS works effectively for local development and prototyping, production AI agents require scalable capabilities. Amazon S3 Vectors provides these capabilities with cost advantages compared to traditional vector databases while maintaining subsecond query performance at scale.</p>

<h3>
  
  
  Core Amazon S3 Vectors Capabilities
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc552riv7o8uuseo2cvy8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fc552riv7o8uuseo2cvy8.png" alt="s3" width="800" height="189"></a></p>

<p>Amazon S3 Vectors offers cloud object storage with native vector support. This approach reduces the complexity of managing separate vector infrastructure while providing:</p>

<ul>
<li>
<strong>Massive scalability</strong>: Each vector index supports large-scale vector storage</li>
<li>
<strong>Managed service</strong>: Fully managed with automatic optimization</li>
<li>
<strong>Enterprise security</strong>: Native 
<a href="https://aws.amazon.com/es/iam/" rel="noopener noreferrer">AWS Identity and Access Management (AWS IAM)</a> integration with user isolation
</li>
<li>
<strong>Global availability</strong>: Multi-region support with built-in disaster recovery</li>
</ul>

<h2>
  
  
  Step 1: S3 Bucket Configuration
</h2>

<p>Before you begin, you need to <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-vectors-buckets-create.html" rel="noopener noreferrer">create an S3 bucket</a> that will serve as the backend for your vector memory.</p>

<h3>
  
  
  Important configuration points:
</h3>

<ul>
<li>Ensure the bucket is in the same region where you'll run your application</li>
<li>Enable versioning for additional security</li>
<li>Configure appropriate IAM access policies</li>
</ul>

<h3>
  
  
  Vector Buckets and Indexes
</h3>

<p>S3 Vectors introduces a bucket type specifically designed for vector data. Within each bucket, you can create multiple vector indexes, each capable of storing vectors with attached metadata for sophisticated filtering.</p>

<h3>
  
  
  Distance Metrics and Performance
</h3>

<p>Amazon S3 Vectors <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-vectors-create-index.html#:~:text=For%20Distance%20metric%2C%20choose%20one%20of%20the%20following%20options%3A" rel="noopener noreferrer">supports both Cosine and Euclidean distance calculations</a>. </p>

<h2>
  
  
  Enhanced Multi-Modal Content Processing Agent
</h2>

<p><a href="https://github.com/elizabethfuentes12/strands-agent-samples/blob/main/notebook/multi-understanding-with-s3-memory.ipynb" rel="noopener noreferrer">It's here in this notebook</a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv0duebu22167ktxvlgse.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv0duebu22167ktxvlgse.png" alt=" " width="800" height="535"></a></p>

<p>Our updated Strands Agents agents now supports S3 Vectors natively usign the <a href="https://github.com/elizabethfuentes12/strands-agent-samples/blob/main/notebook/s3_memory.py" rel="noopener noreferrer"><code>s3_memory</code></a> tool , providing:</p>

<h3>
  
  
  Core Memory Operations
</h3>

<ul>
<li>
<code>store()</code>: Persist agent conversations and learned insights</li>
<li>
<code>retrieve()</code>: Query similar experiences and context</li>
<li>
<code>list()</code>: Enumerate stored memories with metadata</li>
<li>
<code>auto_store_and_retrieve()</code>: Intelligent context management</li>
<li>
<code>auto_context()</code>: Dynamic conversation continuity</li>
</ul>

<h3>
  
  
  Multi-Modal Content Processing
</h3>

<p>Similar to our FAISS implementation, the new agent maintains full support for processing images, documents, and videos with persistent, cross-session memory.</p>

<h2>
  
  
  🛠️ Setting Up the Enhanced Agent
</h2>

<p>First, you need to set the variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Environment configuration for S3 Vectors
</span><span class="n">VECTOR_BUCKET_NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">your-vector-bucket</span><span class="sh">"</span>
<span class="n">VECTOR_INDEX_NAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">agent-memory-index</span><span class="sh">"</span>
<span class="n">USER_ID</span> <span class="o">=</span> <span class="sh">"</span><span class="s">unique-user-identifier</span><span class="sh">"</span>  <span class="c1"># For user isolation
</span></code></pre>

</div>


<p>Let's start by configuring our agent with S3 memory capabilities:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Model configuration
</span><span class="n">model</span> <span class="o">=</span> <span class="nc">BedrockModel</span><span class="p">(</span>
    <span class="n">model_id</span><span class="o">=</span><span class="sh">"</span><span class="s">us.anthropic.claude-3-5-sonnet-20241022-v2:0</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">region</span><span class="o">=</span><span class="sh">"</span><span class="s">us-east-1</span><span class="sh">"</span>
<span class="p">)</span>

<span class="c1"># System prompt for multi-modal processing with memory
</span><span class="n">MULTIMODAL_SYSTEM_PROMPT</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">You are an AI assistant with multi-modal processing capabilities and persistent memory.

Your capabilities:
- **Multi-Modal Analysis**: Process images, documents, videos, and text
- **Persistent Memory**: Remember preferences, previous analyses, and conversation history
- **Context Awareness**: Use memory to provide personalized and contextual responses
- **Continuous Learning**: Build understanding over time through memory accumulation

Memory Usage Guidelines:
- Check for relevant memories before responding
- Store important insights, preferences, and analysis results
- Reference previous conversations when relevant
- Maintain conversation continuity across sessions

When processing content:
1. First retrieve relevant memories for context
2. Analyze the new content thoroughly
3. Store key insights and findings
4. Provide comprehensive responses using both new analysis and memory context
</span><span class="sh">"""</span>

<span class="c1"># Create the multi-modal agent with S3 Vectors memory
</span><span class="n">multimodal_agent</span> <span class="o">=</span> <span class="n">Agent</span><span class="p">(</span>
    <span class="n">model</span><span class="o">=</span><span class="n">model</span><span class="p">,</span>
    <span class="n">tools</span><span class="o">=</span><span class="p">[</span>
        <span class="n">s3_vector_memory</span><span class="p">,</span>  <span class="c1"># Our S3 Vectors memory tool
</span>        <span class="n">image_reader</span><span class="p">,</span>      <span class="c1"># Image processing
</span>        <span class="n">file_read</span><span class="p">,</span>         <span class="c1"># Document processing  
</span>        <span class="n">video_reader</span><span class="p">,</span>      <span class="c1"># Video processing
</span>        <span class="n">use_llm</span>           <span class="c1"># Advanced reasoning
</span>    <span class="p">],</span>
    <span class="n">system_prompt</span><span class="o">=</span><span class="n">MULTIMODAL_SYSTEM_PROMPT</span>

</code></pre>

</div>

<h2>
  
  
  💾 Memory Operations in Action
</h2>
<h3>
  
  
  1. Storing Initial User Context
</h3>

<p>First, let's store some basic information about our user:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">response1</span> <span class="o">=</span> <span class="nf">multimodal_agent</span><span class="p">(</span>
    <span class="sa">f</span><span class="sh">"""</span><span class="s">Hello, I</span><span class="sh">'</span><span class="s">m Elizabeth Fuentes. You can call me Eli, I</span><span class="sh">'</span><span class="s">m a developer advocate at AWS, I like to work early in the morning, 
    I prefer Italian coffee, and I want to understand what</span><span class="sh">'</span><span class="s">s in images, videos, and documents to improve my day-to-day work. 
    I</span><span class="sh">'</span><span class="s">m also very interested in artificial intelligence and work in the financial sector.

    Please save this information about my preferences for future conversations.

    USER_ID: </span><span class="si">{</span><span class="n">USER_ID</span><span class="si">}</span><span class="sh">"""</span>
<span class="p">)</span>
</code></pre>

</div>


<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3dkc8hhz51gx3m1vdg98.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3dkc8hhz51gx3m1vdg98.png" alt=" " width="800" height="111"></a></p>
<h3>
  
  
  2. Image Analysis with Memory Storage
</h3>

<p>Now let's analyze an image and automatically store the results:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=== 📸 IMAGE ANALYSIS WITH MEMORY ===</span><span class="sh">"</span><span class="p">)</span>
<span class="n">image_result</span> <span class="o">=</span> <span class="nf">multimodal_agent</span><span class="p">(</span><span class="sa">f</span><span class="sh">"""</span><span class="s">
                                  Analyze the image data-sample/diagram.jpg in detail and describe everything you observe. 
                                  USER_ID: </span><span class="si">{</span><span class="n">user_id</span><span class="si">}</span><span class="sh">"""</span>
<span class="p">)</span>

</code></pre>

</div>


<p>The agent will:</p>

<ol>
<li>Process the image using <code>image_reader</code>
</li>
<li>Analyze the architectural diagram
Automatically store the analysis in memory using <code>s3_vector_memory</code>
</li>
<li>Provide a detailed description</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxnnkyel1yds0gysjyqpw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxnnkyel1yds0gysjyqpw.png" alt=" " width="800" height="191"></a></p>
<h3>
  
  
  3. Video Analysis with Memory
</h3>

<p>Let's process a video and store its content:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=== 🎬 VIDEO ANALYSIS WITH MEMORY ===</span><span class="sh">"</span><span class="p">)</span>
<span class="n">video_result</span> <span class="o">=</span> <span class="nf">multimodal_agent</span><span class="p">(</span>
    <span class="sh">"</span><span class="s">Analyze the video data-sample/moderation-video.mp4 and describe in detail </span><span class="sh">"</span>
    <span class="sh">"</span><span class="s">the actions and scenes you observe. Store this information in your memory.</span><span class="sh">"</span>
<span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">video_result</span><span class="p">)</span>
</code></pre>

</div>

<h3>
  
  
  4. Document Processing with Memory
</h3>

<p>Process and remember document content:<br>
</p>
<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="nf">print</span><span class="p">(</span><span class="sh">"</span><span class="s">=== 📄 DOCUMENT ANALYSIS WITH MEMORY ===</span><span class="sh">"</span><span class="p">)</span>
<span class="n">doc_result</span> <span class="o">=</span> <span class="nf">multimodal_agent</span><span class="p">(</span>
    <span class="sh">"</span><span class="s">Summarize as json the content of the document data-sample/Welcome-Strands-Agents-SDK.pdf </span><span class="sh">"</span>
    <span class="sh">"</span><span class="s">and store this information in your memory.</span><span class="sh">"</span>
<span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="n">doc_result</span><span class="p">)</span>
</code></pre>

</div>

<h2>
  
  
  🔍 Memory Retrieval and Management
</h2>
<h3>
  
  
  Retrieving Specific Memories
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Retrieve memories related to a specific query
</span><span class="n">memory_result</span> <span class="o">=</span> <span class="nf">s3_vector_memory</span><span class="p">(</span>
    <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">retrieve</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">query</span><span class="o">=</span><span class="sh">"</span><span class="s">preferences and interests</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">user_id</span><span class="o">=</span><span class="n">user_id</span>
<span class="p">)</span>
</code></pre>

</div>

<h3>
  
  
  Listing All Stored Memories
</h3>


<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># List all memories
</span><span class="n">memory_result</span> <span class="o">=</span> <span class="nf">s3_vector_memory</span><span class="p">(</span>
    <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">list</span><span class="sh">"</span><span class="p">,</span>
    <span class="n">user_id</span><span class="o">=</span><span class="n">user_id</span>
<span class="p">)</span>
<span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Total memories in system: </span><span class="si">{</span><span class="n">memory_result</span><span class="p">[</span><span class="sh">'</span><span class="s">total_found</span><span class="sh">'</span><span class="p">]</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
</code></pre>

</div>


<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcwzq3d2bgformjp2kae8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcwzq3d2bgformjp2kae8.png" alt=" " width="800" height="462"></a></p>
<h2>
  
  
  Production Use Cases
</h2>
<h3>
  
  
  AI Agent Memory Systems
</h3>

<p>Store conversation context, user preferences, and learned behaviors across multiple users with automatic scaling and enterprise-grade security.</p>
<h3>
  
  
  Retrieval-Augmented Generation (RAG)
</h3>

<p>Build cost-effective knowledge bases that grow with your business requirements without infrastructure management overhead.</p>
<h3>
  
  
  Semantic Search Applications
</h3>

<p>Process and search through large content datasets with optimized response times.</p>
<h3>
  
  
  Personalized Recommendations
</h3>

<p>Maintain user behavior patterns and preferences with built-in multi-region availability.</p>
<h2>
  
  
  Cost and Availability
</h2>

<p>Amazon S3 Vectors follows the AWS pay-per-use model with no upfront infrastructure costs. The service is currently available in preview across US East (N. Virginia), US East (Ohio), US West (Oregon), Europe (Frankfurt), and Asia Pacific (Sydney).</p>
<h2>
  
  
  🚀 Getting Started
</h2>

<ol>
<li>
<strong>Clone the repository</strong>:
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   git clone https://github.com/elizabethfuentes12/strands-agent-samples<span class="o">)</span>
   <span class="nb">cd </span>notebook
</code></pre>

</div>


<ol>
<li>
<strong>Install dependencies</strong>:
</li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   pip <span class="nb">install</span> <span class="nt">-r</span> requirements.txt
</code></pre>

</div>


<ol>
<li><p><strong>Configure AWS credentials</strong> for Bedrock access</p></li>
<li><p><strong>Try the notebook</strong>:<br>
</p></li>
</ol>
<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>   multi-understanding-with-s3-memory.ipynb
</code></pre>

</div>



<h2>
  
  
  The Strands Agent Advantage
</h2>

<p>What makes Strands Agent unique is its commitment to simplicity without compromising on power. Whether you're building a prototype with local FAISS memory or deploying a production system with Amazon S3 Vectors, the core development experience remains consistent and approachable.</p>

<p>This concludes our three-part series exploring the capabilities of the Strands Agent framework. Throughout this series, we've maintained the core principle that building powerful AI agents should remain simple and accessible, even as we scale to enterprise-level capabilities.</p>
<h2>
  
  
  What We've Covered in This Series:
</h2>

<p><a href="https://dev.to/aws/multi-modal-content-processing-with-strands-agent-and-just-a-few-lines-of-code-4hn4">Part 1: Multi-Modal Content Processing with Just a Few Lines of Code</a> - We showed how Strands Agent makes it effortless to build agents that understand images, documents, and videos.<br>
<a href="https://dev.to/aws/multi-modal-content-processing-with-strands-agent-and-faiss-memory-39hg">Part 2: Adding FAISS Memory for Local Development</a> - We enhanced our agents with persistent memory capabilities for local development and prototyping<br>
<a href="https://dev.tocurrent%20post">Part 3: Scaling with Amazon S3 Vectors</a> - We evolved to production-ready, enterprise-scale memory with cloud infrastructure</p>
<h2>
  
  
  Ready to create your own Strands agent? Here are some resources:
</h2>

<ul>
<li><a href="https://docs.aws.amazon.com/s3/latest/userguide/s3-vectors.html" rel="noopener noreferrer">Amazon S3 Vectors documentation</a></li>
<li><a href="https://github.com/aws-samples/strands-agent-framework" rel="noopener noreferrer">Strands Agent Framework</a></li>
<li><a href="https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html" rel="noopener noreferrer">Amazon Bedrock Knowledge Bases</a></li>
<li><a href="https://dev.to/aws/multi-modal-content-processing-with-strands-agent-and-faiss-memory-39hg">Previous post: Multi-Modal Content Processing with FAISS</a></li>
<li><a href="https://github.com/elizabethfuentes12/strands-agent-samples" rel="noopener noreferrer">Complete Code Examples</a></li>
<li><a href="https://github.com/aws-samples/sample-getting-started-with-strands-agents-course/?trk=4f1e9f0e-7b21-4369-8925-61f67341d27c&amp;sc_channel=el" rel="noopener noreferrer">Getting Started with Strands Agents</a></li>
</ul>



<p>Stay tuned for more Strands Agent implementations!</p>



<p>Gracias!</p>

<p>🇻🇪🇨🇱 <a href="https://dev.to/elizabethfuentes12">Dev.to</a> <a href="https://www.linkedin.com/in/lizfue/" rel="noopener noreferrer">Linkedin</a> <a href="https://github.com/elizabethfuentes12/" rel="noopener noreferrer">GitHub</a> <a href="https://twitter.com/elizabethfue12" rel="noopener noreferrer">Twitter</a> <a href="https://www.instagram.com/elifue.tech" rel="noopener noreferrer">Instagram</a> <a href="https://www.youtube.com/channel/UCr0Gnc-t30m4xyrvsQpNp2Q" rel="noopener noreferrer">Youtube</a><br>
<a href="https://linktr.ee/elizabethfuentesleone" rel="noopener noreferrer">Linktr</a></p>


<div class="ltag__user ltag__user__id__717518">
    <a href="/elizabethfuentes12" class="ltag__user__link profile-image-link">
      <div class="ltag__user__pic">
        <img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F717518%2Fb550b165-b8b9-405d-acfb-e5dc846765b0.png" alt="elizabethfuentes12 image">
      </div>
    </a>
  <div class="ltag__user__content">
    <h2>
<a class="ltag__user__link" href="/elizabethfuentes12">Elizabeth Fuentes L</a>Follow
</h2>
    <div class="ltag__user__summary">
      <a class="ltag__user__link" href="/elizabethfuentes12">AWS Developer Advocate</a>
    </div>
  </div>
</div>



