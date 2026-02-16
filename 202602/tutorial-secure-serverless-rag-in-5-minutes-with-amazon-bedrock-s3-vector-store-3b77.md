---
Title: Tutorial: Secure Serverless RAG in 5 Minutes with Amazon Bedrock + S3 Vector Store
Description: 
Author: Faye Ellis
Date: 2026-02-16T21:23:38.000Z
Robots: noindex,nofollow
Template: index
---
<p>While preparing for the <a href="https://aws.amazon.com/certification/certified-generative-ai-developer-professional/" rel="noopener noreferrer">AWS Certified Generative AI Developer - Professional Exam</a> I wanted to get some more hands-on experience with <strong>Guardrails</strong>, to get a better understanding of exactly what can be done with them.</p>

<p>Here's a five minute tutorial to quickly create an <strong>Amazon Bedrock Knowledge Base</strong> so you can easily explore <strong>Guardrails</strong> and close any knowledge gaps!</p>

<h2>
  
  
  1. Create an S3 bucket
</h2>

<p>Create an <strong>S3 bucket</strong> and upload the documents that you want to include in the <strong>Knowledge Base</strong>. I used a fake <a href="https://github.com/fayekins/genai-dev/blob/main/Ralphie_Bank_Corporate_Strategy.pdf" rel="noopener noreferrer">corporate strategy document</a> for a fictitious company, Ralphie Bank. Feel free to use this if you don't already have a suitable document to use! </p>

<h2>
  
  
  2. Create a Knowledge Base
</h2>

<p>In the <strong>Bedrock</strong> console, click <strong>Knowledge Bases</strong>, and <strong>Create Knowledge Base</strong>. Select <strong>S3 vector store</strong> as the type.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgio0korrtyn0ka1ykb7u.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgio0korrtyn0ka1ykb7u.png" alt="Create Knowledge Base. Select S3 vector store as the type." width="800" height="255"></a></p>

<p>Data source type is <strong>Amazon S3</strong>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxcf76wetwfcq4okhqcn0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxcf76wetwfcq4okhqcn0.png" alt="Data source type is Amazon S3" width="800" height="304"></a></p>

<p>Then click <strong>Next</strong> and <strong>Browse S3</strong> to select the bucket where your data is stored. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fya5juyftrz33yphi749f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fya5juyftrz33yphi749f.png" alt="browse S3 to select the bucket" width="800" height="231"></a></p>

<p>Before proceeding you have the opportunity to select a chunking strategy from a range of different strategies that are natively provided in <strong>Bedrock</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdhc4b8czwi1p31mku80z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fdhc4b8czwi1p31mku80z.png" alt="select a chunking strategy" width="800" height="487"></a></p>

<p>There's also the option to configure a <strong>Lambda</strong> function to perform custom chunking or data processing at this stage. </p>

<p>After that, click <strong>Next</strong>, and select an embeddings model, this is the model that will perform the vector embeddings that will be stored in the <strong>Knowledge Base</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffkqrcmxz0f1lagxyx454.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffkqrcmxz0f1lagxyx454.png" alt="Select an embeddings model" width="800" height="520"></a></p>

<p>Select the type of vector store you want to create. I used <strong>S3 Vectors</strong>, it is much cheaper than <strong>OpenSearch Serverless</strong> and fine for what I need. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ohsxjt5m917xuo0fm2e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ohsxjt5m917xuo0fm2e.png" alt="Select the type of vector store" width="800" height="238"></a></p>

<p>Then click <strong>Next</strong> and <strong>Create Knowledge Base</strong>.</p>

<h2>
  
  
  3. Sync Your Data Source
</h2>

<p>After the Knowledge Base is showing as available, you'll need to sync your data source to actually create populate the data source with the embeddings. Just select your <strong>Knowledge Base</strong> and click <strong>Sync</strong>. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fol113nvw8ns4v50mnot9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fol113nvw8ns4v50mnot9.png" alt="Sync the data source" width="800" height="317"></a></p>

<h2>
  
  
  4. Create a Guardrail
</h2>

<p>Click <strong>Guardrails</strong> in the <strong>Amazon Bedrock</strong> menu on the left, then <strong>Create Guardrail</strong>.</p>

<p>Name it <strong>PII-filter</strong> and click <strong>Next</strong>. This is where you can explore all the different capabilities available in Guardrails. Some of them I have not used before, so I wanted to review everything here and understand what can be configured. For instance there's a prompt attack filter to protect against attempts to override system instructions. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fooxvtrz89l5vuywtam8h.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fooxvtrz89l5vuywtam8h.png" alt="Prompt attack filter" width="800" height="266"></a></p>

<p>After that click <strong>Next</strong>, and here you can add denied topics. As my data contains information about corporate strategy and some financial data, I want my Guardrail to prevent the model giving investment advice. Click <strong>Add Denied Topic</strong>, name it <strong>Financial-Advice</strong> and then provide text describing the topic to block, here's what I added:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Financial advice, investment advice, stock purchase, investment fund
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foe5j662t7vgweeudcm5s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Foe5j662t7vgweeudcm5s.png" alt="Adding a denied topic" width="800" height="478"></a></p>

<p>Keeping <strong>Input</strong> and <strong>Output actions</strong> set to <strong>Block</strong>, click <strong>Confirm</strong>, then <strong>Next</strong>.</p>

<p>Word filters allow you to add profanity filters as well as custom words and phrases, then click <strong>Next</strong>. </p>

<p>After that you can specify how to handle PII.</p>

<p>Click <strong>Add New PII</strong>, then select the type of PII you want to identify. You then select how to deal with that type of PII, block, mask, or detect (no action). </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7tl0t6uqku958icenar.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fo7tl0t6uqku958icenar.png" alt="Select the type of PII" width="800" height="722"></a></p>

<p>I selected phone and email since I know that this type of PII exists in my data. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fesriyhkimd1lztcng7ak.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fesriyhkimd1lztcng7ak.png" alt="Blocking and masking different types of PII" width="800" height="274"></a></p>

<p>After that click <strong>Next</strong> in the next page you can optionally add a <strong>contextual grounding check</strong> and <strong>relevance check</strong>.</p>

<p>Contextual grounding lets you set a threshold for what is acceptable in terms of model confidence score for contextual grounding. The default setting is 0.7 out of 1, if the score is lower than the defined threshold, the model response will be blocked.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx88ky39nos4d7ijpl4dm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx88ky39nos4d7ijpl4dm.png" alt="Contextual grounding check" width="800" height="386"></a></p>

<p>Relevance checks work in a similar way, a relevance score is calculated and if the score is lower than the defined threshold, the model response will be blocked.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9chqc8irmwh43rua6344.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9chqc8irmwh43rua6344.png" alt="Relevance check" width="800" height="389"></a></p>

<p>Then click <strong>Next</strong>, <strong>Next</strong>, and <strong>Create Guardrail</strong>.</p>

<p>After that you are ready to test the functionality and see it in action. </p>

<h2>
  
  
  5. Testing the Knowledge Base Functionality
</h2>

<p>In the left hand menu, click <strong>Knowledge Bases</strong>, select your <strong>Knowledge Base</strong>, then click <strong>Test Knowledge Base</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjxd2gnslsgxxms5yhoma.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjxd2gnslsgxxms5yhoma.png" alt="Test the Knowledge Base" width="800" height="139"></a></p>

<p>Select the model you want to test with, a lite model is fine for this task. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs40ubkohr46pq7bybon9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs40ubkohr46pq7bybon9.png" alt=" " width="800" height="421"></a></p>

<p>Under <strong>Data Manipulation</strong>, open the <strong>Guardrails</strong> dropdown and then select your Guardrail.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2ntg4w422nc8ulyncj8.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe2ntg4w422nc8ulyncj8.png" alt=" " width="800" height="200"></a>  </p>

<p>There is also the opportunity to use a reranking model, like <strong>Cohere rerank</strong> to rescore the relevancy of the retrieved data and override the vector database's relevancy ranking. </p>

<p>After that I added a prompt to test out the configuration: <br>
<code>What are Ralphie Bank’s three strategic pillars?</code><br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa0cid54qhhbxr4duk1jx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fa0cid54qhhbxr4duk1jx.png" alt="Prompting the model" width="800" height="302"></a></p>

<h2>
  
  
  6. Testing the Guardrail
</h2>

<p>Next I tried some prompts that should trigger the Guardrail, here's what I tried: </p>

<p><code>Should I invest in Ralphie Bank shares this year?</code><br>
<code>What is the internal strategy hotline number?</code><br>
<code>What is the compliance email address?</code></p>

<p>If it's all worked, the <strong>Guardrail</strong> will block the request from being sent to the model, and <strong>Bedrock</strong> should not fulfill the request.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9qjaqxsbecbqe2uiunqr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9qjaqxsbecbqe2uiunqr.png" alt="Bedrock should not fulfill the request" width="800" height="420"></a></p>

<h2>
  
  
  7. Clean-up
</h2>

<p>Be sure to delete the <strong>Guardrail</strong>, <strong>Knowledge Base</strong>, and <strong>S3 bucket</strong> to avoid unnecessary charges!</p>

