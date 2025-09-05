---
Title: Why Can’t AI Say “I Don’t Know”? (Bite-size Article)
Description: 
Author: koshirok096
Date: 2025-09-05T22:00:37.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Introduction
</h1>

<p>When you ask cutting-edge AI chatbots like ChatGPT, Claude, or Gemini a question—no matter the topic—you usually get a confident answer back immediately.</p>

<p>It may seem reassuring at first, but have you ever found yourself wondering, “Is that really correct?”, or later discovered the answer was wrong after checking?</p>

<p>This phenomenon is called “AI hallucination.” It refers to outputs that look plausible but are in fact incorrect.<br>
So why does AI, which is supposed to help people, refrain from saying “I don’t know” when something is uncertain and instead respond as if it’s certain? (It’s not that AI can never say “I don’t know,” but structurally they tend to avoid it.)</p>

<p>Digging into this question reveals the mechanisms and design philosophy of AI.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fud47qj0lcqmujhmkl79r.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fud47qj0lcqmujhmkl79r.png" alt=" " width="800" height="533"></a></p>

<h1>
  
  
  LLMs Are “Probabilistic Prediction Devices”
</h1>

<p>To start, there are multiple reasons why AI can’t say “I don’t know” and ends up answering assertively, but the key premise is that large language models (LLMs) like ChatGPT are not knowledge bases. In essence, they learn from massive amounts of text and act as “devices that predict the next likely token.” The training objective is to produce plausible continuations, and whether the content is factually correct is often not part of the objective function. That’s why they can generate natural, fluent prose while also casually producing plausible mistakes. In many cases, that’s the essence of hallucination.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6l6px73m502jaauc3pi5.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6l6px73m502jaauc3pi5.png" alt=" " width="800" height="533"></a></p>

<h2>
  
  
  The “Knowledgeable Friend” Analogy
</h2>

<p>A useful way to think about this is that AI is closer to a “knowledgeable friend” than to an encyclopedia (knowledge base).</p>

<ul>
<li><p>Encyclopedia (knowledge base)<br>
→ Information is curated, and if something isn’t covered it returns “no entry.” Also, because the recorded information tends to be reliable, when your question matches what’s documented, it’s easy to treat it as trustworthy.</p></li>
<li><p>Knowledgeable friend (LLM)<br>
→ They marshal what they know and will still try to answer with “It’s probably like this” even when they don’t actually know.</p></li>
</ul>

<p>The friend is trying to help in good faith, but naturally misunderstandings and guesses get mixed in. AI behaves very similarly.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs96hqcxv7sg6jgwc0jd.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjs96hqcxv7sg6jgwc0jd.png" alt=" " width="800" height="533"></a></p>

<h1>
  
  
  Why Is “Assertion” the Default?
</h1>

<p>So why doesn’t AI conservatively say “I don’t know,” and instead gravitates toward assertive answers? There are several reasons:</p>

<ul>
<li><p>Data bias: The training corpus overwhelmingly contains declarative, assertive sentences.</p></li>
<li><p>Evaluation bias: With RLHF (human feedback), confident answers tend to be rated as “useful.”</p></li>
<li><p>KPI pressure: User satisfaction and conversation-continuation metrics often improve when the system “answers something for now.”</p></li>
<li><p>Lack of mechanisms: Truly withholding an answer requires confidence estimation and threshold design, but many implementations haven’t built that out.</p></li>
</ul>

<p>These are not the only reasons, but together they result in “assert even without 100% certainty” becoming the default behavior.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpjfogfr4sqq68k39pyl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjpjfogfr4sqq68k39pyl.png" alt=" " width="800" height="533"></a></p>

<h2>
  
  
  Google Maps vs. a Knowledgeable Friend
</h2>

<p>To make this less abstract, consider an example.</p>

<p>You want to go somewhere and need to look up the address. There are two ways to do that:</p>

<ul>
<li><p>Using Google Maps (a knowledge base)<br>
→ If the app has the information, it will give you accurate directions. If the address doesn’t exist, it will often return “no information” or only candidates, while acknowledging that mis-entries or outdated data can exist.</p></li>
<li><p>Asking a knowledgeable friend (an LLM)<br>
→ If your friend knows the place, they’ll answer confidently. But you may not be able to immediately verify whether that answer is correct. Also, because they want to help you, they might say “It’s probably this way” even if they don’t actually know.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmvm5bbwyqt1t8t0124wa.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmvm5bbwyqt1t8t0124wa.png" alt=" " width="800" height="533"></a></p>

<h2>
  
  
  How to Avoid Hallucinations
</h2>

<p>What users really want is a correct answer. If a wrong answer comes back and it affects something important in your work or life, that’s a problem. From that perspective, some argue AI should default to expressions like “I think so” or “I’m not certain, but…” when uncertainty is present.</p>

<p>In reality, many AI chatbots are still prone to assertiveness. Here are practical steps users can take:</p>

<ul>
<li><p>In your prompt, instruct the AI to fact-check.<br>
→ “Double-check whether that information is correct,” “Always provide sources,” and other directives that nudge verification.</p></li>
<li><p>Cross-check with a different LLM.<br>
→ Don’t rely on a single model; ask the same question to other services and compare answers (note: if they share similar training sources, multiple models can converge on the same mistake).</p></li>
<li><p>Always check sources and dates.<br>
→ Ask for citations like “as of 2024-06-01, from the official site,” and verify with your own eyes.</p></li>
</ul>

<p>These checks are a hassle, but if you want to proceed more safely, they’re worth adopting as default habits.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fikby311n19f7kyyh82iv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fikby311n19f7kyyh82iv.png" alt=" " width="800" height="533"></a></p>

<h1>
  
  
  Conclusion
</h1>

<p>Since AI emerged, I’ve used it daily, and I’ve often felt frustrated being led astray by hallucinations.<br>
Behind that frustration sits a fixed idea we developed from long familiarity with knowledge-base-style resources like Google—“If a tool clearly presents an answer, it must be correct.”</p>

<p>But LLMs are not knowledge bases; they are probabilistic prediction devices. Because of that structural property—and the broader social preference for assertiveness—AI tends to shift toward assertion rather than saying “I don’t know.”</p>

<p>That’s why it’s crucial not to mistake AI for an omniscient entity. Don’t blindly equate “assertion = correctness”; check sources and dates, and be ready to interpret some outputs as “this might be a guess.”</p>

<p>Treat AI not as an all-knowing answerer, but as a partner that operates under partial knowledge. Making that mental shift is the most important first step to using AI safely and effectively in practice.</p>

