---
Title: Using Ollama and Tailscale to power an Android app with Gemini 3 27B
Description: 
Author: Thomas Ezan
Date: 2025-09-01T21:19:59.000Z
Robots: noindex,nofollow
Template: index
---
<p><em>The views and opinions expressed on this blog are my own and do not reflect those of my employer. Additionally, any solutions, APIs, or products mentioned are for informational and discussion purposes only and should not be considered an endorsement.</em></p>

<p>Running <a href="https://deepmind.google/models/gemma/" rel="noopener noreferrer">Gemma 3 27B</a> with <a href="https://ollama.com/library/gemma3" rel="noopener noreferrer">Ollama</a> on a powerful laptop or a gaming PC gives great performances. It’s very versatile, has good “world knowledge” and is good at text transformation (text summarization, rewrite, etc…). And Ollama also supports system prompts, so you can use it to give the model a persona and turn into a fun chatbot.</p>

<p>Today you can run 8 billion parameters models on a recent Android device (check out Gemma 3n E2B and Gemma 3n E4B through <a href="https://play.google.com/store/apps/details?id=com.google.ai.edge.gallery" rel="noopener noreferrer">Google AI Edge Gallery</a> 😉). But no device has enough memory yet to run 27 billions parameters models.</p>

<p>So as a workaround, let’s run the model on your machine at home and tunnel a connection from your phone to your home network.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffwpsqk1f9mc484lwxkzx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffwpsqk1f9mc484lwxkzx.png" alt="Tunneling Ollama server to an Android" width="800" height="600"></a><br>
<em>Product architecture</em></p>
<h2>
  
  
  Building blocks
</h2>

<p><strong>Ollama</strong><br>
Ollama is a local LLM runtime that makes running open-weight models dead simple. You download a model with one command and it runs locally even if your machine doesn’t have a GPU (it then defaults to CPI). Ollama also packs a web server and provides a <a href="https://github.com/ollama/ollama/blob/main/docs/api.md" rel="noopener noreferrer">REST API</a> for inference. </p>

<p>So no complex setup and no Python environments to manage. Recently, they even released a <a href="https://ollama.com/blog/new-app" rel="noopener noreferrer">desktop app</a> to make interacting with local models even easier.</p>

<p>In this blog post we are using Gemma 3 27B, but the setup will work with any <a href="https://ollama.com/search" rel="noopener noreferrer">model supported by Ollama</a> that your computer can run.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvizzsofe9ju8zefwgx9x.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvizzsofe9ju8zefwgx9x.png" alt="Llama dressed as the Android robot" width="800" height="800"></a><br>
<em>A llama dressed as Google’s Android robot (<a href="https://creativecommons.org/licenses/by/2.5/" rel="noopener noreferrer">cc</a>) <br>
(Image generated via Imagen 4)</em></p>

<p><strong>Tailscale</strong><br>
<a href="https://tailscale.com/" rel="noopener noreferrer">Tailscale</a> creates a secure mesh network between your devices. Install it on your laptop and phone, and they can talk to each other as if they're on the same local network – even when you're thousands miles apart. Tailscale is based on the open source protocol <a href="https://www.wireguard.com/" rel="noopener noreferrer">WireGuard</a> and it takes care of the configuration for you so you won’t have to worry about port forwarding, no firewall rules and router settings.</p>

<p><strong>Bespoke Ollama Android library</strong><br>
I implemented a bespoke library for Android designed to interact with an Ollama API server. I try to use generic Kotlin APIs as much as possible to potentially use this library in KMP. </p>

<p>I used <code>kotlinx.serialization</code> for serialization/deserialization. All API request and response models are defined as <code>@Serializable</code> data classes, which allows for easy and type-safe conversion between JSON and Kotlin objects.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="k">import</span> <span class="nn">kotlinx.serialization.Serializable</span>
<span class="k">import</span> <span class="nn">kotlinx.serialization.SerialName</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">GenerateRequest</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">model</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">prompt</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">stream</span><span class="p">:</span> <span class="nc">Boolean</span> <span class="p">=</span> <span class="k">false</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">format</span><span class="p">:</span> <span class="nc">String</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">options</span><span class="p">:</span> <span class="nc">Options</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">system</span><span class="p">:</span> <span class="nc">String</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">template</span><span class="p">:</span> <span class="nc">String</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">context</span><span class="p">:</span> <span class="nc">String</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">GenerateResponse</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">model</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">createdAt</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">response</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"done_reason"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">doneReason</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"total_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">totalDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"load_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">loadDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"prompt_eval_count"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">promptEvalCount</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"prompt_eval_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">promptEvalDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"eval_count"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">evalCount</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"eval_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">evalDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">context</span><span class="p">:</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">Int</span><span class="p">&gt;?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">ChatRequest</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">model</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">messages</span><span class="p">:</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">Message</span><span class="p">&gt;,</span>
    <span class="kd">val</span> <span class="py">stream</span><span class="p">:</span> <span class="nc">Boolean</span> <span class="p">=</span> <span class="k">false</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">format</span><span class="p">:</span> <span class="nc">String</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">options</span><span class="p">:</span> <span class="nc">Options</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">ChatResponse</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">model</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"created_at"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">createdAt</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">message</span><span class="p">:</span> <span class="nc">Message</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"done_reason"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">doneReason</span><span class="p">:</span> <span class="nc">String</span> <span class="p">=</span> <span class="s">""</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">done</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"total_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">totalDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"load_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">loadDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"prompt_eval_count"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">promptEvalCount</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"prompt_eval_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">promptEvalDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"eval_count"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">evalCount</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"eval_duration"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">evalDuration</span><span class="p">:</span> <span class="nc">Long</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">Message</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">role</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">content</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">images</span><span class="p">:</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">String</span><span class="p">&gt;?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">Model</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">name</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"modified_at"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">modifiedAt</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">size</span><span class="p">:</span> <span class="nc">Long</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">digest</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">details</span><span class="p">:</span> <span class="nc">ModelDetails</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">ModelDetails</span><span class="p">(</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"parent_model"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">parentModel</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">format</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">family</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="kd">val</span> <span class="py">families</span><span class="p">:</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">String</span><span class="p">&gt;?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"parameter_size"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">parameterSize</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"quantization_level"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">quantizationLevel</span><span class="p">:</span> <span class="nc">String</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">Options</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">numa</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_ctx"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numCtx</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_batch"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numBatch</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_gqa"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numGqa</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_gpu"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numGpu</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"main_gpu"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">mainGpu</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"low_vram"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">lowVram</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"f16_kv"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">f16Kv</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"logits_all"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">logitsAll</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"vocab_only"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">vocabOnly</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"use_mmap"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">useMmap</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"use_mlock"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">useMlock</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"embedding_only"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">embeddingOnly</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"rope_frequency_base"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">ropeFrequencyBase</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"rope_frequency_scale"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">ropeFrequencyScale</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_thread"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numThread</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_keep"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numKeep</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"seed"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">seed</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"num_predict"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">numPredict</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"top_k"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">topK</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"top_p"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">topP</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"tfs_z"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">tfsZ</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"typical_p"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">typicalP</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"repeat_last_n"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">repeatLastN</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"temperature"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">temperature</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"repeat_penalty"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">repeatPenalty</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"presence_penalty"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">presencePenalty</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"frequency_penalty"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">frequencyPenalty</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"mirostat"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">mirostat</span><span class="p">:</span> <span class="nc">Int</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"mirostat_tau"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">mirostatTau</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"mirostat_eta"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">mirostatEta</span><span class="p">:</span> <span class="nc">Float</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"penalize_newline"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">penalizeNewline</span><span class="p">:</span> <span class="nc">Boolean</span><span class="p">?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
    <span class="nd">@SerialName</span><span class="p">(</span><span class="s">"stop"</span><span class="p">)</span> <span class="kd">val</span> <span class="py">stop</span><span class="p">:</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">String</span><span class="p">&gt;?</span> <span class="p">=</span> <span class="k">null</span><span class="p">,</span>
<span class="p">)</span>

<span class="nd">@Serializable</span>
<span class="kd">data class</span> <span class="nc">ListModelsResponse</span><span class="p">(</span>
    <span class="kd">val</span> <span class="py">models</span><span class="p">:</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">Model</span><span class="p">&gt;</span>
<span class="p">)</span>
</code></pre>

</div>



<p>I used the Ktor client for handling HTTP requests and picked the <a href="https://api.ktor.io/ktor-client/ktor-client-cio/io.ktor.client.engine.cio/-c-i-o/index.html" rel="noopener noreferrer">CIO (Coroutine-based I/O) engine</a> for Ktor which is  a pure-Kotlin and lightweight solution. However, it’s not compatible with iOS, so I would have to add the Darwin engine for a KMP project supporting Android.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight kotlin"><code><span class="k">import</span> <span class="nn">io.ktor.client.HttpClient</span>
<span class="k">import</span> <span class="nn">io.ktor.client.call.body</span>
<span class="k">import</span> <span class="nn">io.ktor.client.engine.cio.CIO</span>
<span class="k">import</span> <span class="nn">io.ktor.client.plugins.contentnegotiation.ContentNegotiation</span>
<span class="k">import</span> <span class="nn">io.ktor.client.request.get</span>
<span class="k">import</span> <span class="nn">io.ktor.client.request.post</span>
<span class="k">import</span> <span class="nn">io.ktor.client.request.setBody</span>
<span class="k">import</span> <span class="nn">io.ktor.client.statement.bodyAsText</span>
<span class="k">import</span> <span class="nn">io.ktor.http.ContentType</span>
<span class="k">import</span> <span class="nn">io.ktor.http.contentType</span>
<span class="k">import</span> <span class="nn">io.ktor.serialization.kotlinx.json.json</span>
<span class="k">import</span> <span class="nn">kotlinx.coroutines.flow.Flow</span>
<span class="k">import</span> <span class="nn">kotlinx.coroutines.flow.flow</span>
<span class="k">import</span> <span class="nn">kotlinx.serialization.json.Json</span>


<span class="kd">class</span> <span class="nc">OllamaClient</span> <span class="p">{</span>

    <span class="k">private</span> <span class="k">lateinit</span> <span class="kd">var</span> <span class="py">host</span><span class="p">:</span> <span class="nc">String</span> <span class="c1">// server url and port number (usually 11434)</span>

    <span class="k">private</span> <span class="kd">val</span> <span class="py">client</span> <span class="p">=</span> <span class="nc">HttpClient</span><span class="p">(</span><span class="nc">CIO</span><span class="p">)</span> <span class="p">{</span>
        <span class="nf">install</span><span class="p">(</span><span class="nc">ContentNegotiation</span><span class="p">)</span> <span class="p">{</span>
            <span class="nf">json</span><span class="p">(</span><span class="nc">Json</span> <span class="p">{</span>
                <span class="n">ignoreUnknownKeys</span> <span class="p">=</span> <span class="k">true</span>
            <span class="p">})</span>
        <span class="p">}</span>
        <span class="nf">engine</span> <span class="p">{</span>
            <span class="n">requestTimeout</span> <span class="p">=</span> <span class="mi">60_000</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">fun</span> <span class="nf">setHost</span><span class="p">(</span><span class="n">ipAddress</span><span class="p">:</span> <span class="nc">String</span><span class="p">)</span> <span class="p">{</span>
        <span class="n">host</span> <span class="p">=</span> <span class="n">ipAddress</span>
    <span class="p">}</span>

    <span class="k">fun</span> <span class="nf">generate</span><span class="p">(</span><span class="n">request</span><span class="p">:</span> <span class="nc">GenerateRequest</span><span class="p">):</span> <span class="nc">Flow</span><span class="p">&lt;</span><span class="nc">GenerateResponse</span><span class="p">&gt;</span> <span class="p">=</span> <span class="nf">flow</span> <span class="p">{</span>
        <span class="kd">val</span> <span class="py">response</span> <span class="p">=</span> <span class="n">client</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="s">"$host/api/generate"</span><span class="p">)</span> <span class="p">{</span>
            <span class="nf">contentType</span><span class="p">(</span><span class="nc">ContentType</span><span class="p">.</span><span class="nc">Application</span><span class="p">.</span><span class="nc">Json</span><span class="p">)</span>
            <span class="nf">setBody</span><span class="p">(</span><span class="n">request</span><span class="p">)</span>
        <span class="p">}</span>

        <span class="n">response</span><span class="p">.</span><span class="nf">bodyAsText</span><span class="p">().</span><span class="nf">split</span><span class="p">(</span><span class="s">"\n"</span><span class="p">).</span><span class="nf">forEach</span> <span class="p">{</span> <span class="n">line</span> <span class="p">-&gt;</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">line</span><span class="p">.</span><span class="nf">isNotEmpty</span><span class="p">())</span> <span class="p">{</span>
                <span class="nf">emit</span><span class="p">(</span><span class="nc">Json</span><span class="p">.</span><span class="nf">decodeFromString</span><span class="p">(</span><span class="n">line</span><span class="p">))</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">fun</span> <span class="nf">chat</span><span class="p">(</span><span class="n">request</span><span class="p">:</span> <span class="nc">ChatRequest</span><span class="p">):</span> <span class="nc">Flow</span><span class="p">&lt;</span><span class="nc">ChatResponse</span><span class="p">&gt;</span> <span class="p">=</span> <span class="nf">flow</span> <span class="p">{</span>
        <span class="kd">val</span> <span class="py">response</span> <span class="p">=</span> <span class="n">client</span><span class="p">.</span><span class="nf">post</span><span class="p">(</span><span class="s">"$host/api/chat"</span><span class="p">)</span> <span class="p">{</span>
            <span class="nf">contentType</span><span class="p">(</span><span class="nc">ContentType</span><span class="p">.</span><span class="nc">Application</span><span class="p">.</span><span class="nc">Json</span><span class="p">)</span>
            <span class="nf">setBody</span><span class="p">(</span><span class="n">request</span><span class="p">)</span>
        <span class="p">}</span>

        <span class="n">response</span><span class="p">.</span><span class="nf">bodyAsText</span><span class="p">().</span><span class="nf">split</span><span class="p">(</span><span class="s">"\n"</span><span class="p">).</span><span class="nf">forEach</span> <span class="p">{</span> <span class="n">line</span> <span class="p">-&gt;</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">line</span><span class="p">.</span><span class="nf">isNotEmpty</span><span class="p">())</span> <span class="p">{</span>
                <span class="nf">emit</span><span class="p">(</span><span class="nc">Json</span><span class="p">.</span><span class="nf">decodeFromString</span><span class="p">(</span><span class="n">line</span><span class="p">))</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>

    <span class="k">suspend</span> <span class="k">fun</span> <span class="nf">list</span><span class="p">():</span> <span class="nc">List</span><span class="p">&lt;</span><span class="nc">Model</span><span class="p">&gt;</span> <span class="p">{</span>
        <span class="kd">val</span> <span class="py">response</span> <span class="p">=</span> <span class="n">client</span><span class="p">.</span><span class="k">get</span><span class="p">(</span><span class="s">"$host/api/tags"</span><span class="p">).</span><span class="n">body</span><span class="p">&lt;</span><span class="nc">ListModelsResponse</span><span class="p">&gt;()</span>
        <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">models</span>
    <span class="p">}</span>
<span class="na">
   [...]</span> 
   <span class="c1">// Omitting model update and manipulation functions</span>

<span class="p">}</span>
</code></pre>

</div>



<p>Finally, don’t forget to add the <code>INTERNET</code> permission to your manifest 😉 and enable <code>usesCleartextTraffic</code>. This setting will allow the application to easily connect to your local Ollama server over HTTP without requiring complex network security configurations. Usually HTTPS is recommended, but here since we are using Tailscale tunneling between your phone and the Ollama server on your laptop, the communication will still be <a href="https://tailscale.com/kb/1504/encryption" rel="noopener noreferrer">end-to-end encrypted</a>.</p>

<h2>
  
  
  Conclusion
</h2>

<p>After testing it extensively, I can confirm that this setup performs really well. The latency is almost imperceptible with a 5G or even LTE connection, which allows a solid real-time chat experience.</p>

<p>In conclusion, as long as you are connected, mobile private personal AI with a "mid-size" LLM is now possible!</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Few9qzxv1umds4isv14q8.gif" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Few9qzxv1umds4isv14q8.gif" alt="Ollama Android Demo" width="256" height="569"></a><br>
<em>Ollama on Android in Action!</em> </p>

