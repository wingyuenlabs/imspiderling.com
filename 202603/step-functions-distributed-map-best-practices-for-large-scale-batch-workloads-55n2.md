---
Title: Step Functions Distributed Map Best Practices for Large-Scale Batch Workloads
Description: 
Author: Renaldi
Date: 2026-03-01T22:00:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>When I need to process a very large dataset in parallel on AWS without standing up a whole batch platform, AWS Step Functions <strong>Distributed Map</strong> is one of the first tools I reach for.</p>

<p>It gives me a clean orchestration layer for large-scale batch jobs while still letting me control concurrency, retries, failure thresholds, and output handling in a way that is production-friendly.</p>

<p>In this post, I will walk through how I approach <strong>Distributed Map for large-scale batch workloads</strong>, with a focus on practical best practices I would actually use in a real system.</p>




<h2>
  
  
  When I choose Distributed Map (and when I do not)
</h2>

<p>I typically consider <strong>Distributed Map</strong> when one or more of these are true:</p>

<ul>
<li>My dataset is too large to comfortably pass around as state input</li>
<li>I would exceed normal inline <code>Map</code> limits or create a huge execution history</li>
<li>I need high concurrency across many items</li>
<li>My input already lives in S3 (CSV, JSON, object lists, etc.)</li>
<li>I want a serverless orchestration layer and clear visibility into the run</li>
</ul>

<p>I usually <strong>do not</strong> start with Distributed Map if:</p>

<ul>
<li>The batch is small and an inline <code>Map</code> or a single Lambda is enough</li>
<li>I need heavy distributed compute semantics (for example, Spark-style transformations)</li>
<li>My processing requires very long-running per-item tasks that are better suited to another service design</li>
</ul>

<p>The key is to treat Distributed Map as an orchestration primitive for <strong>massively parallel item or batch processing</strong>, not as a replacement for every data platform.</p>




<h2>
  
  
  Architecture at a glance
</h2>

<blockquote>
<p><strong>Figure 1.</strong> Reference architecture for large-scale batch processing with Step Functions Distributed Map (S3 input, batched child workflows, ResultWriter, and observability).</p>
</blockquote>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi9couhdi2uhu05pg02lw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi9couhdi2uhu05pg02lw.png" alt=" " width="800" height="488"></a></p>




<h2>
  
  
  Core best practices I use in production
</h2>

<h3>
  
  
  1) Start with a lower <code>MaxConcurrency</code> and scale up deliberately
</h3>

<p>This is the biggest mistake I see: people let the workflow fan out too aggressively before validating downstream capacity.</p>

<p>Distributed Map can run a very high number of child executions in parallel (AWS documents up to 10,000 parallel child workflow executions in distributed mode), which is great, but your <strong>downstream systems</strong> (Lambda concurrency, databases, APIs, third-party services) are usually the real bottleneck.</p>

<p>What I do instead:</p>

<ul>
<li>Start with a conservative <code>MaxConcurrency</code>
</li>
<li>Load test with representative input</li>
<li>Watch downstream latency, throttling, and error rates</li>
<li>Increase concurrency gradually</li>
</ul>

<p>I treat <code>MaxConcurrency</code> as a <strong>safety valve</strong>, not just a performance knob.</p>




<h3>
  
  
  2) Use <code>ItemBatcher</code> for throughput and cost efficiency
</h3>

<p>If each item is tiny and the per-item work is short, processing one item per child execution can create unnecessary overhead.</p>

<p>In those cases, I use <code>ItemBatcher</code> to group items into batches and process each batch in one child execution.</p>

<p>Why this helps:</p>

<ul>
<li>Fewer child executions</li>
<li>Fewer state transitions</li>
<li>Better Lambda efficiency (especially for small records)</li>
<li>Easier downstream write batching</li>
</ul>

<p>The tradeoff is that my worker logic must handle <code>Items[]</code> input and return a sensible batch-level result.</p>




<h3>
  
  
  3) Keep child outputs compact and use S3 for detailed results
</h3>

<p>A common anti-pattern is returning large payloads from every child execution. That creates pressure on Step Functions payload limits and makes debugging harder.</p>

<p>My default pattern is:</p>

<ul>
<li>Child workflow returns a <strong>compact summary</strong> (counts, status, pointers)</li>
<li>Worker writes detailed outputs/errors to S3 (if needed)</li>
<li>Distributed Map uses <code>ResultWriter</code> to export consolidated execution results</li>
</ul>

<p>This keeps the orchestration layer lightweight while preserving traceability.</p>




<h3>
  
  
  4) Define explicit failure tolerance (<code>ToleratedFailure*</code>) for noisy datasets
</h3>

<p>Large batch workloads often contain some bad records. If I expect occasional malformed rows or downstream rejections, failing the entire Map Run on the first bad batch is usually too strict.</p>

<p>I set one of these based on business tolerance:</p>

<ul>
<li><code>ToleratedFailurePercentage</code></li>
<li><code>ToleratedFailureCount</code></li>
</ul>

<p>This lets me absorb known-noisy inputs while still failing fast when quality degrades beyond an acceptable threshold.</p>

<p>Important design point: these thresholds apply to <strong>failed/timed-out child workflow executions</strong>, so I make sure my child workflow raises failures intentionally when a batch truly cannot be processed.</p>




<h3>
  
  
  5) Put retries and error handling inside the child workflow
</h3>

<p>I prefer retries at the point of failure (inside the <code>ItemProcessor</code> states), not only at the parent level.</p>

<p>For example:</p>

<ul>
<li>Retry transient Lambda/service errors</li>
<li>Catch and classify non-retryable failures</li>
<li>Return a clear batch summary when possible</li>
<li>Fail the child execution when the batch should count toward failure tolerance</li>
</ul>

<p>This gives me better control over what counts as a retryable blip versus a real batch failure.</p>




<h3>
  
  
  6) Choose child execution type intentionally (<code>EXPRESS</code> vs <code>STANDARD</code>)
</h3>

<p>Distributed Map runs inside a <strong>Standard</strong> parent state machine, but each child workflow can be <code>EXPRESS</code> or <code>STANDARD</code>.</p>

<p>My practical rule of thumb:</p>

<ul>
<li>Use <strong>EXPRESS</strong> child workflows for high-throughput, shorter-running item/batch tasks</li>
<li>Use <strong>STANDARD</strong> child workflows when I need longer-running orchestration or richer execution semantics</li>
</ul>

<p>I decide this early because it affects cost, runtime behavior, and observability patterns.</p>




<h3>
  
  
  7) Design the worker to be idempotent
</h3>

<p>At scale, retries and reprocessing happen. I assume they <em>will</em> happen.</p>

<p>That means my batch worker should be idempotent, for example by:</p>

<ul>
<li>Using deterministic item IDs</li>
<li>Recording processed item keys/checkpoints</li>
<li>Using conditional writes (where supported)</li>
<li>Writing outputs with deterministic object keys (or versioned prefixes)</li>
</ul>

<p>This makes retries safe and reduces cleanup work.</p>




<h3>
  
  
  8) Treat quotas and throttling as part of the design, not an afterthought
</h3>

<p>With large-scale fan-out, service quotas and API throttling become part of normal engineering.</p>

<p>I usually validate:</p>

<ul>
<li>Step Functions quotas relevant to Distributed Map and Map Runs</li>
<li>
<code>StartExecution</code> / API throttling expectations</li>
<li>Lambda concurrency and reserved concurrency</li>
<li>Downstream service write capacity or request limits</li>
</ul>

<p>This saves a lot of pain later.</p>




<h2>
  
  
  Example state machine (Distributed Map with batching and ResultWriter)
</h2>

<p>Below is a <strong>JSONPath-based</strong> example that shows a production-style shape I like to start from.</p>

<p>What this example demonstrates:</p>

<ul>
<li>S3-backed input via <code>ItemReader</code>
</li>
<li>
<code>ItemBatcher</code> to process multiple records per child execution</li>
<li>Controlled parallelism via <code>MaxConcurrency</code>
</li>
<li>Failure tolerance via <code>ToleratedFailurePercentage</code>
</li>
<li>
<code>ResultWriter</code> to S3</li>
<li>Retries and a clear failure path inside the child workflow
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"Comment"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Large-scale batch processing with Distributed Map"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"StartAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"RunDistributedBatch"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"States"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"RunDistributedBatch"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Map"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"Label"</span><span class="p">:</span><span class="w"> </span><span class="s2">"BatchIngestV1"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"ItemReader"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"ReaderConfig"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"InputType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"CSV"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"CSVHeaderLocation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"FIRST_ROW"</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::s3:getObject"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"Bucket.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.input.bucket"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"Key.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.input.key"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ItemSelector"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"jobId.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.jobId"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"record.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$$.Map.Item.Value"</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ItemBatcher"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"MaxItemsPerBatch"</span><span class="p">:</span><span class="w"> </span><span class="mi">100</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"MaxConcurrency"</span><span class="p">:</span><span class="w"> </span><span class="mi">300</span><span class="p">,</span><span class="w">
      </span><span class="nl">"ToleratedFailurePercentage"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
      </span><span class="nl">"ItemProcessor"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"ProcessorConfig"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"Mode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"DISTRIBUTED"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"ExecutionType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"EXPRESS"</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"StartAt"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ProcessBatch"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"States"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"ProcessBatch"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Task"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::lambda:invoke"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"OutputPath"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.Payload"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
              </span><span class="nl">"FunctionName"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:lambda:ap-southeast-2:123456789012:function:batch-worker"</span><span class="p">,</span><span class="w">
              </span><span class="nl">"Payload.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$"</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="nl">"Retry"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
              </span><span class="p">{</span><span class="w">
                </span><span class="nl">"ErrorEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
                  </span><span class="s2">"Lambda.ServiceException"</span><span class="p">,</span><span class="w">
                  </span><span class="s2">"Lambda.AWSLambdaException"</span><span class="p">,</span><span class="w">
                  </span><span class="s2">"Lambda.SdkClientException"</span><span class="p">,</span><span class="w">
                  </span><span class="s2">"States.TaskFailed"</span><span class="w">
                </span><span class="p">],</span><span class="w">
                </span><span class="nl">"IntervalSeconds"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
                </span><span class="nl">"BackoffRate"</span><span class="p">:</span><span class="w"> </span><span class="mf">2.0</span><span class="p">,</span><span class="w">
                </span><span class="nl">"MaxAttempts"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="w">
              </span><span class="p">}</span><span class="w">
            </span><span class="p">],</span><span class="w">
            </span><span class="nl">"Catch"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
              </span><span class="p">{</span><span class="w">
                </span><span class="nl">"ErrorEquals"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"States.ALL"</span><span class="p">],</span><span class="w">
                </span><span class="nl">"ResultPath"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.error"</span><span class="p">,</span><span class="w">
                </span><span class="nl">"Next"</span><span class="p">:</span><span class="w"> </span><span class="s2">"MarkBatchFailed"</span><span class="w">
              </span><span class="p">}</span><span class="w">
            </span><span class="p">],</span><span class="w">
            </span><span class="nl">"End"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
          </span><span class="p">},</span><span class="w">
          </span><span class="nl">"MarkBatchFailed"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"Type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Fail"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"Cause"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Batch worker failed after retries"</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"ResultWriter"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"WriterConfig"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"Transformation"</span><span class="p">:</span><span class="w"> </span><span class="s2">"COMPACT"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"OutputType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"JSONL"</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"Resource"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arn:aws:states:::s3:putObject"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"Parameters"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
          </span><span class="nl">"Bucket.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.resultWriter.bucket"</span><span class="p">,</span><span class="w">
          </span><span class="nl">"Prefix.$"</span><span class="p">:</span><span class="w"> </span><span class="s2">"$.resultWriter.prefix"</span><span class="w">
        </span><span class="p">}</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="nl">"End"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Notes on this definition (why I like this shape)
</h3>

<ul>
<li>I use <code>ItemReader</code> so the parent execution input stays small</li>
<li>I use <code>ItemBatcher</code> to reduce overhead for tiny records</li>
<li>I set <code>MaxConcurrency</code> explicitly so I do not accidentally overwhelm downstream systems</li>
<li>I use a <strong>small but non-zero</strong> failure tolerance only if the workload can tolerate some bad batches</li>
<li>I use <code>ResultWriter</code> so the Map Run does not need to return a giant in-memory result array</li>
</ul>




<h2>
  
  
  Example batch worker (Python Lambda)
</h2>

<p>This is a simplified worker pattern I use for batched processing. The main ideas are:</p>

<ul>
<li>Accept <code>Items[]</code> from <code>ItemBatcher</code>
</li>
<li>Process each item safely</li>
<li>Return a compact summary</li>
<li>Fail the batch only when appropriate (so failure tolerance behaves meaningfully)
</li>
</ul>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="n">json</span>
<span class="kn">import</span> <span class="n">hashlib</span>
<span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Dict</span><span class="p">,</span> <span class="n">Any</span><span class="p">,</span> <span class="n">List</span>

<span class="k">def</span> <span class="nf">_stable_item_id</span><span class="p">(</span><span class="n">item</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">str</span><span class="p">:</span>
    <span class="n">raw</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">dumps</span><span class="p">(</span><span class="n">item</span><span class="p">,</span> <span class="n">sort_keys</span><span class="o">=</span><span class="bp">True</span><span class="p">,</span> <span class="n">separators</span><span class="o">=</span><span class="p">(</span><span class="sh">"</span><span class="s">,</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">:</span><span class="sh">"</span><span class="p">))</span>
    <span class="k">return</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">raw</span><span class="p">.</span><span class="nf">encode</span><span class="p">(</span><span class="sh">"</span><span class="s">utf-8</span><span class="sh">"</span><span class="p">)).</span><span class="nf">hexdigest</span><span class="p">()</span>

<span class="k">def</span> <span class="nf">handler</span><span class="p">(</span><span class="n">event</span><span class="p">:</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">],</span> <span class="n">context</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]:</span>
    <span class="c1"># When ItemBatcher is used, Step Functions passes batched input as {"Items": [...]}
</span>    <span class="n">items</span><span class="p">:</span> <span class="n">List</span><span class="p">[</span><span class="n">Dict</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="n">Any</span><span class="p">]]</span> <span class="o">=</span> <span class="n">event</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">Items</span><span class="sh">"</span><span class="p">,</span> <span class="p">[])</span>
    <span class="n">successes</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="n">failures</span> <span class="o">=</span> <span class="mi">0</span>
    <span class="n">failed_items</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">for</span> <span class="n">item_wrapper</span> <span class="ow">in</span> <span class="n">items</span><span class="p">:</span>
        <span class="c1"># In this example, ItemSelector wrapped the original row into {"jobId": ..., "record": ...}
</span>        <span class="n">record</span> <span class="o">=</span> <span class="n">item_wrapper</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">record</span><span class="sh">"</span><span class="p">,</span> <span class="p">{})</span>
        <span class="n">item_id</span> <span class="o">=</span> <span class="nf">_stable_item_id</span><span class="p">(</span><span class="n">record</span><span class="p">)</span>

        <span class="k">try</span><span class="p">:</span>
            <span class="c1"># Example validation (replace with real business logic)
</span>            <span class="k">if</span> <span class="ow">not</span> <span class="n">record</span><span class="p">:</span>
                <span class="k">raise</span> <span class="nc">ValueError</span><span class="p">(</span><span class="sh">"</span><span class="s">Empty record</span><span class="sh">"</span><span class="p">)</span>

            <span class="c1"># TODO: process the record (write to DynamoDB/S3/etc.)
</span>            <span class="c1"># Make downstream writes idempotent (conditional write / deterministic key)
</span>            <span class="n">successes</span> <span class="o">+=</span> <span class="mi">1</span>

        <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">exc</span><span class="p">:</span>
            <span class="n">failures</span> <span class="o">+=</span> <span class="mi">1</span>
            <span class="n">failed_items</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                <span class="sh">"</span><span class="s">itemId</span><span class="sh">"</span><span class="p">:</span> <span class="n">item_id</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">error</span><span class="sh">"</span><span class="p">:</span> <span class="nf">str</span><span class="p">(</span><span class="n">exc</span><span class="p">)</span>
            <span class="p">})</span>

    <span class="n">total</span> <span class="o">=</span> <span class="nf">len</span><span class="p">(</span><span class="n">items</span><span class="p">)</span>
    <span class="n">failure_ratio</span> <span class="o">=</span> <span class="p">(</span><span class="n">failures</span> <span class="o">/</span> <span class="n">total</span><span class="p">)</span> <span class="k">if</span> <span class="n">total</span> <span class="k">else</span> <span class="mf">0.0</span>

    <span class="c1"># Business decision:
</span>    <span class="c1"># fail the child execution only when the batch quality is unacceptable.
</span>    <span class="k">if</span> <span class="n">total</span> <span class="o">==</span> <span class="mi">0</span> <span class="ow">or</span> <span class="n">failure_ratio</span> <span class="o">&gt;</span> <span class="mf">0.20</span> <span class="ow">or</span> <span class="n">successes</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>
        <span class="k">raise</span> <span class="nc">RuntimeError</span><span class="p">(</span>
            <span class="sa">f</span><span class="sh">"</span><span class="s">Batch failed: successes=</span><span class="si">{</span><span class="n">successes</span><span class="si">}</span><span class="s">, failures=</span><span class="si">{</span><span class="n">failures</span><span class="si">}</span><span class="s">, total=</span><span class="si">{</span><span class="n">total</span><span class="si">}</span><span class="sh">"</span>
        <span class="p">)</span>

    <span class="k">return</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">PARTIAL_SUCCESS</span><span class="sh">"</span> <span class="k">if</span> <span class="n">failures</span> <span class="k">else</span> <span class="sh">"</span><span class="s">SUCCESS</span><span class="sh">"</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">totalItems</span><span class="sh">"</span><span class="p">:</span> <span class="n">total</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">successes</span><span class="sh">"</span><span class="p">:</span> <span class="n">successes</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">failures</span><span class="sh">"</span><span class="p">:</span> <span class="n">failures</span><span class="p">,</span>
        <span class="sh">"</span><span class="s">failedItemSample</span><span class="sh">"</span><span class="p">:</span> <span class="n">failed_items</span><span class="p">[:</span><span class="mi">10</span><span class="p">]</span>
    <span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Worker implementation tips I strongly recommend
</h3>

<ul>
<li>Keep the response small</li>
<li>Write detailed failure artifacts to S3 if you need full diagnostics</li>
<li>Use deterministic keys / conditional writes for idempotency</li>
<li>Separate retryable vs non-retryable exceptions in your code (if possible)</li>
<li>Emit metrics (success count, failure count, latency) from the worker</li>
</ul>




<h2>
  
  
  One retry gotcha I always call out (important)
</h2>

<p>I am very careful with adding a <code>Retry</code> block on the <strong>Distributed Map state itself</strong>.</p>

<p>Why? Because AWS documents that if you define retriers on the Distributed Map state, the retry policy applies to <strong>all child workflow executions started by that Map state</strong>, not just the one that failed. In practice, that can create a lot of duplicate work and surprise costs if you are not expecting it.</p>

<p>My default approach:</p>

<ul>
<li>Put retries inside the child workflow first (fine-grained retries)</li>
<li>Use Map-level <code>Retry</code> only when I intentionally want to rerun the whole Map state / Map Run behavior</li>
<li>Make sure idempotency is in place before enabling broader retries</li>
</ul>




<h2>
  
  
  Observability and operations checklist (what I watch)
</h2>

<p>When I productionize this pattern, I monitor the workflow at three levels:</p>

<h3>
  
  
  A) Parent state machine
</h3>

<ul>
<li>Execution failures/timeouts</li>
<li>End-to-end duration</li>
<li>Throttling indicators (if any)</li>
</ul>

<h3>
  
  
  B) Map Run / child workflow level
</h3>

<ul>
<li>Child execution success/failure trends</li>
<li>Failure rate relative to my tolerated threshold</li>
<li>Backlog behavior when concurrency is constrained</li>
</ul>

<h3>
  
  
  C) Worker and downstream systems
</h3>

<ul>
<li>Lambda concurrency and duration</li>
<li>Error rate and retries</li>
<li>Database/API throttling</li>
<li>Write latency and saturation</li>
</ul>

<p>I also make sure the team has a runbook for:</p>

<ul>
<li>What to do when failure tolerance is exceeded</li>
<li>How to inspect Map Run details and failing child executions</li>
<li>When and how to redrive after fixing data or IAM issues</li>
</ul>




<h2>
  
  
  Common mistakes I try to avoid
</h2>

<h3>
  
  
  1) Leaving <code>MaxConcurrency</code> effectively unlimited
</h3>

<p>This can look great in testing and then melt a downstream dependency in production.</p>

<h3>
  
  
  2) Returning huge outputs from child workflows
</h3>

<p>This adds payload pressure and makes the state machine harder to operate.</p>

<h3>
  
  
  3) Using batch sizes without thinking about item size
</h3>

<p>Small records and large records behave very differently. Batch by both <strong>count</strong> and (if needed) <strong>input bytes</strong> based on your real data.</p>

<h3>
  
  
  4) Treating all failures the same
</h3>

<p>Malformed data, throttling, and transient AWS service errors should not all be handled identically.</p>

<h3>
  
  
  5) Skipping idempotency
</h3>

<p>At scale, retries are normal. Idempotency is what keeps retries safe.</p>




<h2>
  
  
  A practical rollout strategy I use
</h2>

<p>If I am introducing Distributed Map into an existing system, I usually do it in phases:</p>

<ol>
<li><strong>Prototype with a small sample dataset</strong></li>
<li><strong>Enable explicit <code>MaxConcurrency</code></strong></li>
<li><strong>Add <code>ItemBatcher</code></strong></li>
<li><strong>Add <code>ResultWriter</code> and compact outputs</strong></li>
<li><strong>Load test with realistic record sizes</strong></li>
<li><strong>Tune retries, batch size, and failure thresholds</strong></li>
<li><strong>Document redrive/runbook steps</strong></li>
</ol>

<p>This sequence helps me avoid premature complexity while still landing on a production-grade design.</p>




<h2>
  
  
  Final thoughts
</h2>

<p>Distributed Map is one of those features that becomes incredibly powerful once I stop thinking of it as “just a bigger map loop” and start treating it as a <strong>batch orchestration framework</strong> with explicit controls.</p>

<p>The best results usually come from combining:</p>

<ul>
<li><strong>S3-backed input</strong></li>
<li><strong>Deliberate concurrency limits</strong></li>
<li><strong>Batched child processing</strong></li>
<li><strong>Compact outputs + S3 result export</strong></li>
<li><strong>Clear failure semantics</strong></li>
<li><strong>Good observability and runbooks</strong></li>
</ul>

<p>That combination gives me something that scales well and is still operationally sane.</p>




<h2>
  
  
  References
</h2>

<ul>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/state-map-distributed.html" rel="noopener noreferrer">AWS Step Functions: Distributed Map (Developer Guide)</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/state-map.html" rel="noopener noreferrer">AWS Step Functions: Map state (inline vs distributed)</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/input-output-itemreader.html" rel="noopener noreferrer">AWS Step Functions: ItemReader (Map)</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/input-output-itembatcher.html" rel="noopener noreferrer">AWS Step Functions: ItemBatcher (Map)</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/input-output-resultwriter.html" rel="noopener noreferrer">AWS Step Functions: ResultWriter (Map)</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/concepts-examine-map-run.html" rel="noopener noreferrer">AWS Step Functions: Viewing a Distributed Map Run execution</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/procedure-cw-metrics.html" rel="noopener noreferrer">AWS Step Functions: Monitoring metrics with CloudWatch</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/redrive-map-run.html" rel="noopener noreferrer">AWS Step Functions: Redriving Map Runs</a></li>
<li><a href="https://docs.aws.amazon.com/step-functions/latest/dg/service-quotas.html" rel="noopener noreferrer">AWS Step Functions: Service quotas</a></li>
<li><a href="https://catalog.workshops.aws/large-scale-data-processing-with-step-functions/en-US" rel="noopener noreferrer">AWS Step Functions Workshop: Large-scale data processing</a></li>
</ul>

