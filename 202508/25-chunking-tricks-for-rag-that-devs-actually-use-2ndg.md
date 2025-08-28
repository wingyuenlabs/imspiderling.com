---
Title: 25 chunking tricks for RAG that devs actually use
Description: 
Author: <devtips/>
Date: 2025-08-28T21:53:56.000Z
Robots: noindex,nofollow
Template: index
---
<p><span></span></p>
<h2 id="a8a0">Not another theory-filled blog post just real chunking strategies that work (or blew up in testing).</h2>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AXhUpCjvm5FZhKzPblZrwlA.png" width="800" height="800"><p id="1bd0">Chunking is the unsung hero of RAG pipelines. Done right, it makes your LLM feel sharp and reliable. Done wrong, it spits out hallucinations like “The Eiffel Tower is a vegetable.”</p>
<p id="6aee">This guide skips the fluff. No recycled diagrams. No “just use LangChain lol.” Just <strong>25 real-world chunking strategies</strong>, tested in messy PDFs, gnarly codebases, and live projects.</p>
<p id="5a66">Some work like magic. Some failed hilariously. You’ll get both.</p>
<p id="d263">Let’s chunk smart.</p>
<h2 id="a815">Table of contents</h2>
<ol>

<li id="6fb6">What even is chunking in RAG?</li>

<li id="e4ac">Why bad chunking ruins everything</li>

<li id="2efc">Core chunking strategies (the classics)</li>

<li id="6f78">Semantic-aware chunking (AI knows best)</li>

<li id="0f9a">Code and technical docs chunking</li>

<li id="8efb">Multimodal chunking (images, tables, PDFs)</li>

<li id="0564">Application-based chunking (custom jobs)</li>

<li id="ef8f">Tools and libraries that help</li>

<li id="8884">Chunking experiments that worked (or hilariously failed)</li>

<li id="dfd5">Clean-up tips and sanity checks</li>

<li id="82f5">Conclusion + resources + outro</li>

</ol>
<span></span><span></span><span></span><h2 id="43b3">What even is “chunking” in RAG?</h2>
<p id="b89c">Chunking is how you prep documents for retrieval in a RAG system. Since LLMs can’t process entire files (too long, too chaotic), you split your data into smaller, meaningful pieces called chunks that are easy to index, embed, and search.</p>
<p id="3478">These chunks get converted into vector embeddings and stored in a vector database like <strong>Pinecone</strong>, <strong>Weaviate</strong>, or <strong>FAISS</strong>. When a user asks a question, the RAG pipeline uses semantic search to fetch the most relevant chunks, then passes those into the LLM as context.</p>
<p id="aec2">Think of it like Git diffs: instead of comparing whole files, you break them into meaningful diffs that are easier to track, understand, and manage. The same applies to chunking. You’re optimizing for <strong>relevance</strong> and <strong>retrievability</strong>, not just size.</p>
<h3 id="6e7c">When do you actually need chunking?</h3>
<p id="1202">You need chunking any time you:</p>
<ul>

<li id="e4e0">Feed long-form documents (PDFs, markdown, Notion exports, API docs) into RAG</li>

<li id="68a3">Want to avoid LLM context limits (OpenAI has ~16k–128k token windows, but that’s not unlimited)</li>

<li id="0049">Plan to <strong>embed and search</strong> content in a vector store</li>

<li id="f792">Work with semi-structured or messy data like scraped web pages, logs, or support chats</li>

</ul>
<p id="a208">Basically: if your LLM’s input context isn’t short, clean, and already scoped you need chunking. And the way you chunk it affects the final answer the LLM gives. Every time.</p>
<span></span><span></span><span></span><h2 id="0f72">Why bad chunking ruins everything</h2>
<p id="deb2">Chunking sounds simple until you realize bad chunks are why your RAG system keeps returning answers that read like a chatbot having a stroke.</p>
<p id="7b56">The problem? <strong>Loss of context.</strong> If your chunk cuts off mid-thought or splits a concept in half, the LLM loses the thread. Too much overlap and it becomes repetitive. Not enough? It becomes clueless. And if you’re blindly chunking by word count or tokens without considering the content yeah, you’re gonna get garbage out.</p>
<p id="bafc">Real talk: I once built a quick RAG prototype on company policy docs. I used basic 500-token chunks with zero overlap and no cleanup. When asked “What’s the parental leave policy?”, it confidently responded with “Refer to Section 3.1 and the attached ketchup bottle.” No idea where that came from.</p>
<p id="671f">That’s what happens when context boundaries are broken, semantically unrelated content is stuck together, and keywords are present without meaning.</p>
<p id="7416">Chunking isn’t just about <em>size</em> it’s about <em>sense</em>.</p>
<span></span><span></span><span></span><h2 id="0fa4">The Core chunking strategies (the classics)</h2>
<p id="bcaf">These are the OG methods most people start with. They’re simple, reliable, and work surprisingly well for many cases if you know when to use them.</p>
<h3 id="c5d2">1. Fixed-size chunking (tokens or words)</h3>
<ul>

<li id="caee">Break text into equally sized blocks (e.g., 500 tokens or 300 words)</li>

<li id="7977">Easy to implement with tokenizer tools like <code><a href="https://github.com/openai/tiktoken" rel="noopener ugc nofollow noreferrer">tiktoken</a></code>
</li>

<li id="ca9e">Works well for uniform data like logs or chat transcripts</li>

</ul>
<blockquote><p id="4a9a"><strong>Downside: Splits can land mid-sentence or mid-paragraph, killing context.</strong></p></blockquote>
<h3 id="802c">2. Sentence-level chunking</h3>
<ul>

<li id="2549">Split based on sentence boundaries using punctuation or NLP libs like <code>spaCy</code>
</li>

<li id="1cff">Chunks make more grammatical sense, improving coherence</li>

</ul>
<blockquote><p id="a6a0"><strong>Downside: Chunks may be too short to be useful alone.</strong></p></blockquote>
<h3 id="e866">3. Paragraph-level chunking</h3>
<ul>

<li id="dfe3">Treat each paragraph as one chunk</li>

<li id="5e8c">Great for structured docs like wikis, blog posts, manuals</li>

</ul>
<p id="f4e1"><strong><em>Downside: Some paragraphs are massive; others too tiny. Inconsistent retrieval quality.</em></strong></p>
<h3 id="fdd8">4. Sliding window chunking</h3>
<ul>

<li id="c78e">Create overlapping chunks: chunk A (tokens 1–500), chunk B (tokens 400–900), etc.</li>

<li id="6c24">Improves continuity and recall at the cost of more chunks</li>

</ul>
<p id="d897"><strong><em>Downside: Increases storage and retrieval load</em></strong></p>
<h3 id="2a9e">5. Recursive chunking</h3>
<ul>

<li id="dfed">Try one method (like paragraph), then fallback to smaller units if the chunk is too big</li>

<li id="95e3">Supported by tools like <code>LangChain RecursiveCharacterTextSplitter</code>
</li>

</ul>
<p id="9687"><strong><em>Use case: When your data is uneven and you want graceful degradation</em></strong></p>
<p id="0fe7">These methods are simple, but still form the foundation for most chunking pipelines. Don’t dismiss them just because they’re old-school they’re predictable and fast.</p>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AdiJty5RFRvb7ZRS08ZC7aQ.png" width="800" height="800"><h2 id="7532">Semantic-aware chunking (AI knows best)</h2>
<p id="20cc">Classic chunking works fine… until your content stops being regular. Think messy blogs, technical docs, or transcripts. That’s where semantic chunking steps in splitting text based on <strong>meaning</strong>, not just size.</p>
<p id="6a17">Here are a few strategies devs actually use when they want their chunks to <em>make sense</em>:</p>
<h3 id="bfa3">6. Sentence similarity-based chunking</h3>
<ul>

<li id="efcb">Use cosine similarity between sentences to group semantically related ideas</li>

<li id="9d44">Great for narratives, tutorials, or anything with logical flow</li>

<li id="f11b">Often done using sentence transformers like <code>all-MiniLM</code>
</li>

</ul>
<blockquote><p id="f504"><strong><em>Ideal when you want each chunk to contain a full idea or sub-topic.</em></strong></p></blockquote>
<h3 id="9f04">7. Topic segmentation (TextTiling, Topical BERT)</h3>
<ul>

<li id="2bf3">Use NLP models to detect topic shifts and create boundaries</li>

<li id="1b8d">

<code>TextTiling</code> is classic, <code><a href="https://github.com/zelandiya/topical-segmentation" rel="noopener ugc nofollow noreferrer">Topical Segmenter</a></code> is newer</li>

<li id="c38b">Super useful for long-form content with multiple subtopics (e.g., research papers)</li>

</ul>
<h3 id="c8a4">8. Graph-based segmentation</h3>
<ul>

<li id="fc45">Build a semantic graph of sentences, find natural break points where topic flow changes</li>

<li id="a3a7">More complex but can give amazing results on academic or deeply structured content</li>

</ul>
<blockquote><p id="c8f8"><strong><em>Feels overkill for FAQs, goldmine for research-heavy domains.</em></strong></p></blockquote>
<h3 id="506d">9. Embedding-aware chunking</h3>
<ul>

<li id="b449">Use the <strong>embedding distance</strong> between chunks to decide where to split</li>

<li id="1ddf">Detects shifts in meaning at vector level</li>

<li id="930a">Can be paired with a threshold-based splitter or adaptive window</li>

</ul>
<p id="2eb4">Semantic chunking costs more compute, but pays off in quality. You’re not just throwing chunks into a vector DB you’re feeding the LLM <strong>coherent</strong>, <strong>complete</strong> thoughts.</p>
<span></span><span></span><span></span><h2 id="106a">Chunking for code and technical docs</h2>
<p id="31d3">Trying to chunk code like it’s prose? Congrats, you just created the worst autocomplete ever.</p>
<p id="0034">Code has a different structure. You can’t just slice by character count or paragraph breaks it’ll split <code>function()</code> declarations, orphan braces, and destroy context. Same with docs like Swagger specs, Markdown-heavy READMEs, or API manuals.</p>
<p id="ff6a">Here’s how to chunk code <strong>intelligently</strong>:</p>
<h3 id="2086">10. Function-based chunking</h3>
<ul>

<li id="f30a">Extract one function per chunk (<code>def</code>, <code>function</code>, etc.)</li>

<li id="381b">Keeps logic together and makes retrieval relevant</li>

<li id="19d5">Works great in Python, JavaScript, Go, etc.</li>

</ul>
<h3 id="0045">11. Class/module-based chunking</h3>
<ul>

<li id="0543">Group related functions under a class or file</li>

<li id="d0af">Useful for class-heavy languages like Java, TypeScript, or C++</li>

</ul>
<blockquote><p id="bd69"><strong><em>Watch out for massive files may still need fallback chunking.</em></strong></p></blockquote>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2A3fectsybLhleuGrvcmIVSw.jpeg" width="800" height="800"><h3 id="6d44">12. Comment-to-code chunking</h3>
<ul>

<li id="5b33">Chunk together comment blocks + the function or code they describe</li>

<li id="9a6a">Improves semantic search, since comments often contain user-facing terms</li>

</ul>
<h3 id="fb0d">13. Docstring-first chunking</h3>
<ul>

<li id="e66c">Prioritize docstrings and chunk surrounding logic with them</li>

<li id="d429">LLMs love docstrings it’s like cheat codes for retrieval</li>

</ul>
<h3 id="4747">Bonus: Markdown-aware splitting</h3>
<ul>

<li id="b351">For docs like <code>README.md</code>, chunk by headers (<code>##</code>) and subheaders (<code>###</code>)</li>

<li id="2b25">Keeps logical sections together and helps with clean context injection</li>

</ul>
<p id="351b">TL;DR: Treat code like code. If you split logic in half, you confuse the LLM and yourself.</p>
<span></span><span></span><span></span><h2 id="e729">Multimodal chunking (images, tables, PDFs)</h2>
<p id="007f">Not all data is neat, UTF-8 text. Sometimes you’re chunking scanned PDFs, HTML pages, tables, graphs, or slides that were exported by someone who hates future developers.</p>
<p id="d80a">Here’s how to avoid turning your RAG pipeline into a dumpster fire when handling mixed content:</p>
<h3 id="06a3">14. Table-aware chunking</h3>
<ul>

<li id="c0c4">Extract tables as <strong>complete units</strong>, not as rows or raw text</li>

<li id="cb02">Use <code>pandas</code>, <code>pdfplumber</code>, or <code>Camelot</code> to preserve structure</li>

<li id="062c">You want to retain the <strong>layout and headers</strong> for context</li>

</ul>
<h3 id="1aaf">15. OCR + layout-aware splitting</h3>
<ul>

<li id="c70c">Use OCR tools like <code><a href="https://github.com/tesseract-ocr/tesseract" rel="noopener ugc nofollow noreferrer">Tesseract</a></code> or <code>LayoutLM</code> for scanned docs</li>

<li id="371c">Preserve visual blocks (columns, headers, footers)</li>

<li id="46db">Especially useful for resumes, contracts, invoices</li>

</ul>
<blockquote><p id="bd36"><em>Pro tip: PDFs aren’t real documents they’re screenshots with trust issues.</em></p></blockquote>
<h3 id="a0b4">16. Image/document section segmentation</h3>
<ul>

<li id="7b70">Use vision encoders (e.g., <code>CLIP</code>, <code>Donut</code>) to segment visual content into logical regions</li>

<li id="a314">Chunk each region as a self-contained “semantic image chunk”</li>

<li id="817e">Works well for UI screenshots, infographics, slide decks</li>

</ul>
<h3 id="ef70">17. Hybrid chunking for HTML</h3>
<ul>

<li id="b425">Chunk by <code>&lt;h1&gt;</code>, <code>&lt;h2&gt;</code>, <code>&lt;p&gt;</code>, <code>&lt;table&gt;</code>, etc.</li>

<li id="b2d4">Retain DOM structure, skip nav/ads</li>

<li id="4041">Use tools like <code>BeautifulSoup</code>, <code>html2text</code>, or <code>trafilatura</code>
</li>

</ul>
<p id="dbe9">Chunking isn’t just about size here it’s about preserving layout, hierarchy, and meaning. If the model can’t “see” what’s grouped together, it won’t reason about it.</p>
<span></span><span></span><span></span><h3 id="5f0d">Application-based chunking (custom jobs)</h3>
<p id="bea4">Some data just refuses to behave. A generic chunker won’t cut it when you’re dealing with <strong>legal contracts</strong>, <strong>medical records</strong>, <strong>customer support chats</strong>, or anything with <strong>domain-specific structure</strong>. Here, you chunk with context in mind not just structure or size.</p>
<h3 id="0029">18. Legal documents → Section-aware chunking</h3>
<ul>

<li id="502c">Use heading detection (e.g., “Section 4.2 Liability”) to split at logical contract sections</li>

<li id="f303">Retain clause numbers and sub-clauses together</li>

<li id="2bf9">Don’t split mid-paragraph bad legal advice awaits</li>

</ul>
<h3 id="c190">19. Medical data → Entity + sentence-level</h3>
<ul>

<li id="ac6c">Combine sentence-level splits with key medical entities (e.g., drugs, symptoms, dosages)</li>

<li id="c099">Use clinical NLP models like <code>scispaCy</code>
</li>

<li id="b4e6">Crucial for preserving diagnostic relationships</li>

</ul>
<h3 id="1436">20. Customer support → Intent-based chunking</h3>
<ul>

<li id="8c8b">Chunk by resolved intents: each Q&amp;A pair or resolved case is one unit</li>

<li id="5247">Group similar problems with their resolutions</li>

<li id="eed5">Improves retrieval accuracy for chatbot or ticket-based RAG</li>

</ul>
<h3 id="60c2">21. Research papers → Abstract + section chunking</h3>
<ul>

<li id="24ca">Chunk by “Abstract,” “Methods,” “Results,” etc.</li>

<li id="45ee">Preserve citations as a unit with their explanation</li>

<li id="2d59">Tools like <code><a href="https://github.com/kermitt2/grobid" rel="noopener ugc nofollow noreferrer">Grobid</a></code> can help structure papers</li>

</ul>
<p id="e6e4">When you’re working in a specific domain, chunking <strong>must reflect how humans navigate the content</strong> not just what fits in 500 tokens.</p>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AXoUG92S5m6ttoNIsXdZKrQ.png" width="800" height="800"><h3 id="4e2d">Tools and libraries that help</h3>
<p id="b95b">Look, you <em>can</em> write your own chunker from scratch, regex and all. Or you can use one of these libraries that already figured it out, tested it, and wrapped it in a function with a name like <code>split_documents()</code>.</p>
<p id="0855">Here are some tools devs actually use:</p>
<h3 id="539c">22. LangChain</h3>
<ul>

<li id="7ca4">Offers <code>RecursiveCharacterTextSplitter</code>, <code>TokenTextSplitter</code>, and more</li>

<li id="1a33">Handles fallback strategies and overlapping chunks</li>

<li id="9915">Good starting point for structured and unstructured text</li>

<li id="ab30">Docs →</li>

</ul>
<h3 id="41fe">23. LlamaIndex (formerly GPT Index)</h3>
<ul>

<li id="4199">High-level abstractions for chunking + indexing</li>

<li id="06ef">Comes with <code>SentenceSplitter</code>, <code>SemanticSplitter</code>, and support for PDFs, HTML, Markdown, etc.</li>

<li id="328f">Especially nice when paired with retrieval eval tools</li>

<li id="b48a">Docs →</li>

</ul>
<h3 id="7978">24. Haystack</h3>
<ul>

<li id="5fda">Modular RAG framework with preprocessing pipelines</li>

<li id="9a4b">Use it with Hugging Face models or OpenAI</li>

<li id="5c6a">Strong PDF and OCR support for chunking weird docs</li>

<li id="fd58">Docs →</li>

</ul>
<h3 id="156b">25. OpenAI Tokenizer Viewer</h3>
<ul>

<li id="ae2c">Helps visualize how text will be tokenized before you chunk</li>

<li id="9379">Use this to debug token splits in fixed-size strategies</li>

<li id="edd8"><a href="https://platform.openai.com/tokenizer" rel="noopener ugc nofollow noreferrer"><strong>Web tool →</strong></a></li>

</ul>
<h3 id="db49">Honorable mentions</h3>
<ul>

<li id="9567">

<code>spaCy</code> – great for sentence parsing</li>

<li id="4c2b">

<code><a href="https://github.com/jsvine/pdfplumber" rel="noopener ugc nofollow noreferrer">pdfplumber</a></code> – table-aware PDF parsing</li>

<li id="9fdb">

<code><a href="https://github.com/adbar/trafilatura" rel="noopener ugc nofollow noreferrer">trafilatura</a></code> – web text + HTML cleaner with structural awareness</li>

</ul>
<p id="abd4">These tools don’t just save time they prevent you from burning hours on chunking edge cases that someone else already solved.</p>
<span></span><span></span><span></span><h2 id="4233">Chunking experiments that worked (or hilariously failed)</h2>
<p id="7403">Chunking isn’t just theory it’s trial, error, and screaming into your terminal.</p>
<p id="a44e">Here are a few real stories (yes, tested on actual projects) that show what can go right or spectacularly wrong.</p>
<h3 id="ba75">Worked: Recursive + overlap = clean dev wiki</h3>
<ul>

<li id="8255">Chunked a Notion wiki using <strong>recursive paragraph split</strong>, fallback to sentence, with 20% overlap</li>

<li id="3bb3">Retrieval relevance jumped ~30% with fewer hallucinations</li>

<li id="0f5b">Overlap helped with questions that landed between topics</li>

</ul>
<h3 id="e281">Failed: Fixed-token chunks on legal docs</h3>
<ul>

<li id="c998">Tried 500-token chunks with no structure awareness</li>

<li id="4c06">Retrieval kept surfacing “Definitions” section for everything</li>

<li id="6032">Outputs: legally impressive nonsense</li>

</ul>
<h3 id="345e">Worked: Markdown-aware chunking for tech blog archive</h3>
<ul>

<li id="5319">Split by headers (<code>##</code>, <code>###</code>) in Markdown</li>

<li id="916b">Each chunk became a clean, isolated sub-topic</li>

<li id="7a9a">Made fine-tuned RAG assistant way more accurate for tutorials</li>

</ul>
<h3 id="c1ad">Failed: Sentence-level chunking for source code</h3>
<ul>

<li id="20df">Chunked Python scripts sentence by sentence (why? idk, it was late)</li>

<li id="45cb">Retrieval returned “def”, “return”, and “if” as separate answers</li>

<li id="f764">Worse than useless it was confident <em>and</em> wrong</li>

</ul>
<p id="eed8">Some of these taught us more than any tutorial ever could. Chunking is like tuning hyperparameters you don’t really know what works until it breaks.</p>
<span></span><span></span><span></span><h3 id="dcb1">Clean-up tips and sanity checks</h3>
<p id="220f">You’ve chunked your docs. They’re in the vector DB. You run your first query…</p>
<p id="1191">…and the results make less sense than a regex written under duress.</p>
<p id="49cb">Before blaming the LLM, check your chunk hygiene. Here’s how:</p>
<h3 id="2409">1. Watch for cutoff chunks</h3>
<p id="d508">Chunks ending mid-sentence or mid-thought? That’s how you get hallucinations. Use sentence or semantic splitters if you’re seeing garbage completions.</p>
<h3 id="bb5e">2. Tune your overlap percentage</h3>
<p id="cbf9">Too little overlap = broken context. Too much = bloated retrieval with duplicate answers.<br> Start with <strong>10–20%</strong> overlap and adjust based on output quality.</p>
<h3 id="c3d9">3. Validate with actual retrieval tests</h3>
<p id="5200">Don’t just chunk and ship. Use a few test questions and verify:</p>
<ul>

<li id="4d47">Are the right chunks being retrieved?</li>

<li id="98d2">Do answers change if chunking config changes?</li>

</ul>
<p id="36ab"><strong>Use tools like:</strong></p>
<ul>

<li id="ca82">

<code>llama-index</code> retriever playground</li>

<li id="27ea">

<code><a href="https://github.com/explodinggradients/ragas" rel="noopener ugc nofollow noreferrer">RAGAS</a></code> for retrieval evals</li>

</ul>
<h3 id="da8d">4. Clean your input before you chunk</h3>
<p id="afbe">Remove:</p>
<ul>

<li id="82d9">Extra whitespace</li>

<li id="e361">Table of contents</li>

<li id="43b2">Footers and headers (especially in PDFs)</li>

<li id="67d2">Boilerplate sections repeated across docs</li>

</ul>
<p id="2363">Garbage in = garbage chunked = garbage out.</p>
<p id="40ef">These last-mile details are what separate a “cool prototype” from an actually useful RAG system.</p>
<span></span><span></span><span></span><h2 id="3c87">Conclusion: Chunk smarter, not harder</h2>
<p id="9314">If you made it this far, congrats you now know more about chunking than 95% of devs throwing PDFs at vector DBs and praying for answers.</p>
<p id="dd82"><strong>To recap:</strong></p>
<ul>

<li id="dc69">There’s no “one right” chunking method.</li>

<li id="8586">Classic strategies still work <strong>if</strong> you know when to use them.</li>

<li id="5a80">Semantic, code-aware, and layout-aware chunking can <strong>massively</strong> improve retrieval.</li>

<li id="0e69">Bad chunking quietly ruins everything, so test often and tune like a backend config.</li>

</ul>
<p id="77ff">You don’t need a PhD to chunk well you just need to stop slicing everything at 500 tokens like it’s 2022.</p>
<h2 id="a311">Helpful resources</h2>
<ul>

<li id="0f1c"><a href="https://python.langchain.com/docs/modules/data_connection/document_transformers/" rel="noopener ugc nofollow noreferrer">LangChain Text Splitters</a></li>

<li id="7d1e"><a href="https://docs.llamaindex.ai/en/stable/" rel="noopener ugc nofollow noreferrer">LlamaIndex Splitters</a></li>

<li id="7b97"><a href="https://docs.haystack.deepset.ai/docs/latest/" rel="noopener ugc nofollow noreferrer">Haystack Preprocessing Pipelines</a></li>

<li id="cf99"><a href="https://github.com/explodinggradients/ragas" rel="noopener ugc nofollow noreferrer">RAGAS RAG eval framework</a></li>

<li id="9b90"><a href="https://platform.openai.com/tokenizer" rel="noopener ugc nofollow noreferrer">OpenAI Tokenizer Viewer</a></li>

<li id="3fae"><a href="https://aclanthology.org/J97-1003.pdf" rel="noopener ugc nofollow noreferrer">TextTiling paper (1997)</a></li>

</ul>
<span></span><img alt="" src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A1155%2F1%2AWMCesV6B2egTIckiWZm7XQ.jpeg" width="800" height="800">

