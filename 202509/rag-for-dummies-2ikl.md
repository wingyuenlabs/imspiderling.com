---
Title: RAG for Dummies
Description: 
Author: NgetichB
Date: 2025-09-13T21:04:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>Retrieval Augmented Generation(RAG) is a machine learning technique that enhances the capabilities of Large Language Models to provide more accurate and up-to date responses.</p>

<p>How RAG works:</p>

<p>(i)The user asks the Large Language Models (LLM) a question</p>

<p>(ii)Retrieval is the second step whereby the RAG system uses the question asked to search an external knowledge base for relevant information. The RAG system uses these three techniques; chunking, embedding and vector database. Chunking- the information in the knowledge base is broken into smaller pieces for efficient searching, the chunks are then converted into numerical representations that capture their meanings, finally the system searches a vector database to find the chunks that are more likely similar to the question asked.</p>

<p>(iii)Augmentation-The most relevant information from the retrieval process is then added to the original question to form an ‘augmented prompt’</p>

<p>(iv)The LLM receives the prompt and uses the original question and the retrieved context to generate a more comprehensive and accurate response</p>

<p>Models used in RAG</p>

<p>(i)Retrieval Models- these act as a detective that gathers relevant documents from the external knowledge base before the LLM generates an answer. The two types of retriever models are Sparse- examples BM25 and TF-IDF and Dense retrievers -eg Llamaindex &amp; Haystack.</p>

<p>(ii) Language Models (LLMs)- the generation component takes the users original prompt and the retrieved information and uses its learned knowledge to create a coherent, natural language response. The examples are- Transformer-based models( GPT-2, GPT-3, and BART (Bidirectional and Auto-Regressive Transformers) and Flan T5 used for the generation part</p>

<p>RAG is applied in Medical AI, chatbots, chat engines and legal assistance. It serves the purpose of bridging the gap between static information and dynamic knowledge hence reduces ambiguity and increases precision, transparency and accuracy.</p>

