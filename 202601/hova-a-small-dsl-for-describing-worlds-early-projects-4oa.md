---
Title: Hova: a small DSL for describing worlds (early projects)
Description: 
Author: Gabriel Xavier
Date: 2026-01-21T21:13:33.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been working on a small project called Hova.</p>

<p><strong>Hova</strong> is a DSL focused on** describing game worlds, entities and universe structures** in a clean and declarative way.</p>

<p>It is not a game engine, and it doesn't execute logic or handle rendering.</p>

<p>Its only goal is to describe what exists in a universe — not how it run.</p>




<h3>
  
  
  Why I built it
</h3>

<p>When building games or tools, I often felt that world design ended up mixed with engine code, scripting logic or implementation details.</p>

<p>I wanted something that:</p>

<ul>
<li>stays engine-agnostic</li>
<li>focuses only on world structure and meaning</li>
<li>can be consumed by different tools or pipelines</li>
</ul>

<p>That's where Hova came from.</p>




<h3>
  
  
  How it works
</h3>

<p>You write Hova source code describing your world, and it gets converted into a <strong>neutral format</strong> (currently JSON).</p>

<p>This output can then be consumed by:</p>

<ul>
<li>game engines</li>
<li>custom tooling</li>
<li>build pipelines</li>
<li>converters</li>
</ul>

<p>Hova doesn't decide <em>how</em> the world is used — it only describes it.</p>

<p><strong>A small example</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>anvil Shapes 

    atomic 
        atom visual : "minimal"
        atom hideConfig : "on"
    end

    ore Square 
        spark sides : 4
    end

    ore Triangle
        spark sides : 3
    end

end
</code></pre>

</div>



<p><strong>Which converts to:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
    </span><span class="nl">"Shapes"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"atomic"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"emit"</span><span class="p">:</span><span class="w"> </span><span class="s2">"json"</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"Square"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"sides"</span><span class="p">:</span><span class="w"> </span><span class="mi">4</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"Triangle"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"sides"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="w">
        </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>






<h3>
  
  
  Current state
</h3>

<p>Hova is still an <strong>early project</strong>.</p>

<p>Documentation is a work in progress, so the <code>/examples</code> folder is currently the main reference to understand the language.</p>

<p>The language itself is evolving, and feedback at this stage is extremely valuable.</p>




<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi60xm0lu1uepi1pe8hnj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi60xm0lu1uepi1pe8hnj.png" alt=" " width="800" height="250"></a></p>

<h3>
  
  
  What I'm looking for
</h3>

<p>I'd love feedback on:</p>

<ul>
<li>the language design</li>
<li>syntax clarity</li>
<li>whether this kind of DSL could be useful in real game or tooling workflows</li>
</ul>

<p>If you're interested, here's the repository:<br>
👉 <a href="https://github.com/G4brielXavier/Hova" rel="noopener noreferrer">https://github.com/G4brielXavier/Hova</a></p>

<p>THANKS FOR READING.</p>

