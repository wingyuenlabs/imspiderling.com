---
Title: A Token-Efficient Way to Send Time-Series Data into LLMs
Description: 
Author: Manas Mudbari
Date: 2025-12-31T21:14:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>If you’ve ever pushed time-series data (metrics, logs, network streams, sensor readings) into an LLM, you’ve probably noticed that even small datasets can get expensive and slow very quickly.</p>

<p>Not because the data is huge, but because of how it gets tokenized.</p>

<p>This post is about a representation experiment we’ve been running to reduce that overhead, what didn’t work, and what seems to help.</p>

<h2>
  
  
  The Problem We Ran Into
</h2>

<p>Time-series data is repetitive by nature:</p>

<ul>
<li>timestamps move forward predictably</li>
<li>values often change slowly</li>
<li>schema repeats on every row</li>
</ul>

<p>Humans immediately see the pattern but LLMs don’t.</p>

<p>Most LLM tokenizers are optimized for natural language, not numerical streams. Two numbers that look almost identical to us can tokenize very differently. Repeating structure (timestamps, keys, braces) quietly eats context and cost.</p>

<p>At small scale, it’s annoying, but at scale, it becomes an infrastructure problem.</p>

<h2>
  
  
  What We Tried (and What Didn’t Help Much)
</h2>

<p>Before building anything new, we tried existing formats:</p>

<ul>
<li>JSON (baseline)</li>
<li>CSV (more compact, but still verbose)</li>
<li>TOON (interesting idea, but still text that gets re-tokenized)</li>
</ul>

<p>In practice, TOON didn’t materially reduce token usage once everything was still passed as plain text into an LLM. The structure was different, but the tokenizer behavior didn’t improve much.</p>

<p>That was the key realization: compression alone isn’t the problem — tokenization is.</p>

<h2>
  
  
  Math was our intuition
</h2>

<p>If you’ve taken calculus or physics, this will feel familiar.</p>

<p>Think about motion:</p>

<ul>
<li>Position → where something is</li>
<li>Velocity → how position changes</li>
<li>Acceleration → how velocity changes</li>
</ul>

<p>Now map that to time-series data:</p>

<ul>
<li>raw values = position</li>
<li>differences between values = velocity (delta)</li>
<li>differences between deltas = acceleration (delta-of-delta)</li>
</ul>

<p>Most real-world time-series data has low acceleration. Values drift; timestamps tick forward regularly.</p>

<p>So instead of repeating full values and timestamps, we started experimenting with representing changes.</p>

<h2>
  
  
  TSLN: A Token-Aware Representation
</h2>

<p>That experiment turned into <strong>TSLN (Time-Series Lean Notation)</strong>.</p>

<p>At a high level, it’s a text-based serialization that:</p>

<ul>
<li>stores deltas instead of repeating full values</li>
<li>stores delta-of-delta for regular timestamps</li>
<li>declares schema once instead of repeating it</li>
<li>stays human-readable and streamable</li>
</ul>

<p>The key difference from “just compression” is that it’s designed to be tokenization-aware. Smaller, bounded numbers and less repeated syntax lead to far fewer tokens once the model sees the input.</p>

<p>In early benchmarks, the same datasets used up to ~80% fewer tokens compared to JSON. That directly translated into lower cost and better effective context windows when calling LLMs.</p>

<h2>
  
  
  Code and Next Steps
</h2>

<p>We’ve open-sourced <a href="https://github.com/turboline-ai/tsln-golang" rel="noopener noreferrer">Go</a> and <a href="https://github.com/turboline-ai/tsln-node" rel="noopener noreferrer">Node.js</a> implementations under the MIT license so it’s easy to experiment or drop into existing pipelines.</p>

<p>I’m currently expanding the benchmarks across more datasets, tokenizers, and workloads, and plan to publish a more formal preprint once that’s done.</p>

<p>If you work with:</p>

<ul>
<li>time-series data</li>
<li>LLM pipelines</li>
<li>serialization or streaming systems</li>
</ul>

<p>I’d genuinely love feedback — especially edge cases, comparisons we should run, or prior art I may have missed.</p>

