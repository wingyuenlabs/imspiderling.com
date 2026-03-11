---
Title: I had to build my own Symfony validation bundle because no existing one fits my requirements
Description: 
Author: David Evdoshchenko
Date: 2026-03-11T22:02:22.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Contents
</h2>

<ul>
<li>Long story short</li>
<li>The Problem</li>
<li>The Idea</li>
<li>The Solution</li>
<li>
Quick Examples

<ul>
<li>Example 1</li>
<li>Example 2</li>
</ul>


</li>

<li>What's the result?</li>

<li>The Full Story</li>

</ul>

<h2>
  
  
  Long story short
</h2>

<p>I created a bundle for request validation with JSON Schema because no existing "schema-first" validator fit my requirements. Now I just attach a JSON file to a route and get everything at once: validation, DTO mapping, and OpenAPI documentation from a single source of truth.</p>

<ul>
<li>Repo: <a href="https://github.com/outcomer/symfony-json-schema-validation" rel="noopener noreferrer">https://github.com/outcomer/symfony-json-schema-validation</a>
</li>
<li>Docs: <a href="https://outcomer.github.io/symfony-json-schema-validation/" rel="noopener noreferrer">https://outcomer.github.io/symfony-json-schema-validation/</a>
</li>
</ul>

<h2>
  
  
  The Problem
</h2>

<p>Most validation solutions that can generate API documentation from code (in the Symfony world I mostly mean FOSRestBundle and API Platform) assume that your business logic is:</p>

<ul>
<li>Well defined and relatively stable</li>
<li>Close to a classic CRUD model (or CRUD with small deviations)</li>
<li>Exposed via clean, REST-style endpoints that you fully control</li>
</ul>

<p>In other words, they assume your application defines the contract and the outside world adapts to it.</p>

<p>But in many real projects it is the opposite: the API contract is defined somewhere else (legacy frontend, external systems, partners), and you have to adapt to that contract. That is where problems start.</p>

<p>Here is a simplified example. The API expects this payload:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"company"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"user"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"John"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"email"</span><span class="p">:</span><span class="w"> </span><span class="s2">"john@example.com"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"company"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Acme"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>With the following rules:</p>

<ul>
<li>If <code>type = "company"</code> then <code>user.company.name</code> is required</li>
<li>If <code>type = "person"</code> then <code>user.company</code> must be absent</li>
</ul>

<p>Is this the most elegant API design? Probably not. But imagine a company with 2000 people and a frontend written years ago that sends exactly this structure. You cannot just redesign everything because you do not like the shape of the JSON.</p>

<p>Now, what does the "ideal" Symfony validation setup suggest here?<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="kd">class</span> <span class="nc">UserDto</span>
<span class="p">{</span>
    <span class="na">#[Assert\NotBlank]</span>
    <span class="k">public</span> <span class="kt">string</span> <span class="nv">$name</span><span class="p">;</span>

    <span class="na">#[Assert\Valid]</span>
    <span class="k">public</span> <span class="kt">?CompanyDto</span> <span class="nv">$company</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This does not express the conditional logic "company.name is required when type = company". To implement this you usually end up with one of the following:</p>

<ul>
<li>Custom constraint with its own validator</li>
<li>Manual validation logic in the controller or a service</li>
<li>Custom normalizer / denormalizer with additional checks</li>
</ul>

<p>Then another question appears: how do you forbid extra properties that are not defined in <code>UserDto</code>? For a long time you simply could not. In newer Symfony versions you can write something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="err">#</span><span class="p">[</span><span class="nf">MapRequestPayload</span><span class="p">(</span>
    <span class="n">serializationContext</span><span class="o">:</span> <span class="p">[</span><span class="s1">'allow_extra_attributes'</span> <span class="o">=&gt;</span> <span class="kc">false</span><span class="p">]</span>
<span class="p">)]</span>

<span class="err">#</span><span class="p">[</span><span class="nf">MapQueryString</span><span class="p">(</span>
    <span class="n">serializationContext</span><span class="o">:</span> <span class="p">[</span><span class="s1">'allow_extra_attributes'</span> <span class="o">=&gt;</span> <span class="kc">false</span><span class="p">]</span>
<span class="p">)]</span>
</code></pre>

</div>



<p>Whether this works for query parameters depends on the exact types and context. For headers this approach does not work at all.</p>

<p>You might ask: why be so strict? Why not just ignore extra parameters? Because in real life this often leads to subtle bugs. A typical scenario: you had a query parameter <code>offset</code> and later renamed it to <code>page</code> for consistency with other APIs. Some client code still sends <code>offset</code>. If you ignore unknown parameters, the request "works", but returns the wrong page. You then spend time debugging something that could have been caught immediately.</p>

<p>With strict validation the client would get a clear error about an unknown parameter, and the problem would be visible right away.</p>

<p>My personal view is that even though an API has no visual UI, it still has UX. Clients should receive clear, precise error messages, not a generic "Provided data is incorrect". Detailed validation errors are also in the interest of the backend team: fewer support tickets, less time spent guessing what went wrong on the client side.</p>

<h2>
  
  
  The Idea
</h2>

<p>The kind of validation I needed has actually existed for years, just not in the form of typical Symfony validators. I am talking about the JSON Schema standard:</p>

<p><a href="https://json-schema.org/specification" rel="noopener noreferrer">https://json-schema.org/specification</a></p>

<blockquote>
<p>JSON Schema is a declarative language for defining structure and constraints for JSON data</p>
</blockquote>

<p>It is designed exactly for problems like the ones above (and much more complex ones):</p>

<ul>
<li>Conditional rules based on other fields</li>
<li>Nested, deeply structured data</li>
<li>Strict control over allowed and forbidden properties</li>
</ul>

<p>So the idea was simple: instead of forcing my API contract into DTO classes and annotations, let Symfony validate incoming requests against a JSON Schema that fully describes the contract.</p>

<p>In other words, make Symfony request validation schema-first, with JSON Schema as the single source of truth.</p>

<h2>
  
  
  The Solution
</h2>

<p>The good news was that I did not need to implement JSON Schema myself. There was already a solid PHP implementation:</p>

<p><a href="https://opis.io/json-schema/" rel="noopener noreferrer">https://opis.io/json-schema/</a></p>

<p>The library takes a valid JSON Schema and any input data, validates the data against the schema and either:</p>

<ul>
<li>returns the data (when everything is valid), or</li>
<li>returns a structured list of validation errors.</li>
</ul>

<p>From there, the rest was mostly integration work:</p>

<ul>
<li>Make it convenient to plug this validation into a Symfony project</li>
<li>Wire it into the request lifecycle</li>
<li>Provide a way to map validated data into DTOs when needed</li>
<li>Integrate with Nelmio so that OpenAPI documentation is generated from the same schemas</li>
</ul>

<p>Below I will show a couple of short examples. In the "The Full Story" section I describe how I removed duplication between validation attributes and documentation, and how the bundle ended up solving both validation and API specification generation from a single source of truth: the JSON Schema files.</p>

<h2>
  
  
  Quick Examples
</h2>

<ul>
<li>Example 1</li>
<li>Example 2</li>
</ul>

<p>The schema<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
    </span><span class="nl">"$schema"</span><span class="p">:</span><span class="w"> </span><span class="s2">"https://json-schema.org/draft-07/schema#"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"object"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"properties"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="nl">"query"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"object"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"properties"</span><span class="p">:</span><span class="w"> </span><span class="p">{},</span><span class="w">
            </span><span class="nl">"additionalProperties"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"headers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"object"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"properties"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                </span><span class="nl">"authorization"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Bearer token for authentication"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"pattern"</span><span class="p">:</span><span class="w"> </span><span class="s2">"^Bearer .+"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"example"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ6..."</span><span class="w">
                </span><span class="p">},</span><span class="w">
                </span><span class="nl">"x-api-version"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"API version"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"enum"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"v1"</span><span class="p">,</span><span class="w"> </span><span class="s2">"v2"</span><span class="p">],</span><span class="w">
                    </span><span class="nl">"example"</span><span class="p">:</span><span class="w"> </span><span class="s2">"v1"</span><span class="w">
                </span><span class="p">},</span><span class="w">
                </span><span class="nl">"content-type"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Request content type"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"enum"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"application/json"</span><span class="p">],</span><span class="w">
                    </span><span class="nl">"example"</span><span class="p">:</span><span class="w"> </span><span class="s2">"application/json"</span><span class="w">
                </span><span class="p">}</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="nl">"additionalProperties"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="w">
        </span><span class="p">},</span><span class="w">
        </span><span class="nl">"body"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
            </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"object"</span><span class="p">,</span><span class="w">
            </span><span class="nl">"properties"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                </span><span class="nl">"name"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"minLength"</span><span class="p">:</span><span class="w"> </span><span class="mi">3</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"maxLength"</span><span class="p">:</span><span class="w"> </span><span class="mi">100</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"User's full name"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"example"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Jane Smith"</span><span class="w">
                </span><span class="p">},</span><span class="w">
                </span><span class="nl">"email"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"string"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"format"</span><span class="p">:</span><span class="w"> </span><span class="s2">"email"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"User's email address"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"example"</span><span class="p">:</span><span class="w"> </span><span class="s2">"john.doe@example.com"</span><span class="w">
                </span><span class="p">},</span><span class="w">
                </span><span class="nl">"age"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
                    </span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"integer"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"minimum"</span><span class="p">:</span><span class="w"> </span><span class="mi">21</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"maximum"</span><span class="p">:</span><span class="w"> </span><span class="mi">100</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"User's age (optional)"</span><span class="p">,</span><span class="w">
                    </span><span class="nl">"example"</span><span class="p">:</span><span class="w"> </span><span class="mi">30</span><span class="w">
                </span><span class="p">}</span><span class="w">
            </span><span class="p">},</span><span class="w">
            </span><span class="nl">"required"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"name"</span><span class="p">,</span><span class="w"> </span><span class="s2">"email"</span><span class="p">],</span><span class="w">
            </span><span class="nl">"additionalProperties"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">
        </span><span class="p">}</span><span class="w">
    </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h3>
  
  
  Example 1: validation using the built-in request object
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="err">#</span><span class="p">[</span><span class="nf">OA\Post</span><span class="p">(</span>
    <span class="n">operationId</span><span class="o">:</span> <span class="s1">'validateUser'</span><span class="p">,</span>
    <span class="n">summary</span><span class="o">:</span> <span class="s1">'Validate user'</span><span class="p">,</span>
<span class="p">)]</span>
<span class="na">#[Route('/user', name: '_example_validation_user', methods: ['POST'])]</span>
<span class="k">public</span> <span class="k">function</span> <span class="n">validateUser</span><span class="p">(</span><span class="err">#</span><span class="p">[</span><span class="nf">MapRequest</span><span class="p">(</span><span class="s1">'./user-create.json'</span><span class="p">)]</span> <span class="nc">ValidatedRequest</span> <span class="nv">$request</span><span class="p">)</span><span class="o">:</span> <span class="nc">JsonResponse</span>
<span class="p">{</span>
    <span class="nv">$payload</span> <span class="o">=</span> <span class="nv">$request</span><span class="o">-&gt;</span><span class="nf">getPayload</span><span class="p">();</span>
    <span class="nv">$body</span>    <span class="o">=</span> <span class="nv">$payload</span><span class="o">-&gt;</span><span class="nf">getBody</span><span class="p">();</span>

    <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">json</span><span class="p">([</span>
        <span class="s1">'success'</span> <span class="o">=&gt;</span> <span class="kc">true</span><span class="p">,</span>
        <span class="s1">'message'</span> <span class="o">=&gt;</span> <span class="s1">'User data is valid'</span><span class="p">,</span>
        <span class="s1">'data'</span>    <span class="o">=&gt;</span> <span class="p">[</span>
            <span class="s1">'name'</span>  <span class="o">=&gt;</span> <span class="nv">$body</span><span class="o">-&gt;</span><span class="n">name</span><span class="p">,</span>
            <span class="s1">'email'</span> <span class="o">=&gt;</span> <span class="nv">$body</span><span class="o">-&gt;</span><span class="n">email</span><span class="p">,</span>
            <span class="s1">'age'</span>   <span class="o">=&gt;</span> <span class="nv">$body</span><span class="o">-&gt;</span><span class="n">age</span> <span class="o">??</span> <span class="kc">null</span><span class="p">,</span>
        <span class="p">],</span>
        <span class="s1">'example'</span> <span class="o">=&gt;</span> <span class="s1">'This uses ValidatedRequest (standard way)'</span><span class="p">,</span>
    <span class="p">],</span> <span class="mi">200</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h3>
  
  
  Example 2: validation using a custom DTO (via ValidatedDtoInterface)
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight php"><code><span class="err">#</span><span class="p">[</span><span class="nf">OA\Post</span><span class="p">(</span>
    <span class="n">operationId</span><span class="o">:</span> <span class="s1">'createProfile'</span><span class="p">,</span>
    <span class="n">summary</span><span class="o">:</span> <span class="s1">'Create profile'</span><span class="p">,</span>
<span class="p">)]</span>
<span class="na">#[Route('/profile', name: '_example_validation_profile', methods: ['POST'])]</span>
<span class="k">public</span> <span class="k">function</span> <span class="n">createProfile</span><span class="p">(</span><span class="err">#</span><span class="p">[</span><span class="nf">MapRequest</span><span class="p">(</span><span class="s1">'./user-create.json'</span><span class="p">)]</span> <span class="nc">UserApiDtoRequest</span> <span class="nv">$profile</span><span class="p">)</span><span class="o">:</span> <span class="nc">JsonResponse</span>
<span class="p">{</span>
    <span class="k">return</span> <span class="nv">$this</span><span class="o">-&gt;</span><span class="nf">json</span><span class="p">([</span>
        <span class="s1">'success'</span> <span class="o">=&gt;</span> <span class="kc">true</span><span class="p">,</span>
        <span class="s1">'message'</span> <span class="o">=&gt;</span> <span class="s1">'Profile created successfully'</span><span class="p">,</span>
        <span class="s1">'profile'</span> <span class="o">=&gt;</span> <span class="p">[</span>
            <span class="s1">'name'</span>  <span class="o">=&gt;</span> <span class="nv">$profile</span><span class="o">-&gt;</span><span class="n">name</span><span class="p">,</span>
            <span class="s1">'email'</span> <span class="o">=&gt;</span> <span class="nv">$profile</span><span class="o">-&gt;</span><span class="n">email</span><span class="p">,</span>
            <span class="s1">'age'</span>   <span class="o">=&gt;</span> <span class="nv">$profile</span><span class="o">-&gt;</span><span class="n">age</span><span class="p">,</span>
        <span class="p">],</span>
        <span class="s1">'note'</span>    <span class="o">=&gt;</span> <span class="nb">sprintf</span><span class="p">(</span><span class="s1">'This demonstrates DTO auto-injection: MapRequestResolver calls %s::fromPayload() automatically'</span><span class="p">,</span> <span class="nc">UserApiDtoRequest</span><span class="o">::</span><span class="n">class</span><span class="p">),</span>
    <span class="p">],</span> <span class="mi">201</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  What's the result?
</h2>

<ul>
<li>Focused solution: instead of reinventing the wheel, the bundle fills a specific gap that existing Symfony tools do not cover well (schema-first request validation).</li>
<li>Less duplication: you no longer have to mirror the same rules in DTO constraints, controllers and OpenAPI annotations.</li>
<li>Automatic sync: Nelmio builds documentation from the same JSON Schema files that are used for validation, so your docs always match the real behavior.</li>
<li>Contract-centric design: the entire API contract lives in clean JSON files rather than being scattered across PHP attributes and PHP classes.</li>
</ul>

<p>If you were looking for a way to validate requests without creating a large number of redundant DTO classes and annotations, this bundle is designed for that use case:</p>

<ul>
<li>Repo: <a href="https://github.com/outcomer/symfony-json-schema-validation" rel="noopener noreferrer">https://github.com/outcomer/symfony-json-schema-validation</a>
</li>
<li>Docs: <a href="https://outcomer.github.io/symfony-json-schema-validation/" rel="noopener noreferrer">https://outcomer.github.io/symfony-json-schema-validation/</a>
</li>
</ul>

<h2>
  
  
  The Full Story
</h2>

<p>This article is the short, focused version of the story: what the bundle does and how to start using it. If you want the long version with all the scars and details, I wrote a separate, much bigger post.</p>

<p>In that post I go through:</p>

<ul>
<li>how the bundle was born in a very messy real project, not in a greenfield demo</li>
<li>why classic DTO + Assertions validation did not survive 500+ routes</li>
<li>how JSON Schema became the single contract language for backend, frontend and docs</li>
<li>how the bundle glues Symfony, Opis JSON Schema and Nelmio together</li>
</ul>

<p>You can read the full story here (starting from The Full Story section):</p>

<p><a href="https://outcomer.hashnode.dev/symfony-bundle-that-validates-anything-and-everything#the-full-story" rel="noopener noreferrer">https://outcomer.hashnode.dev/symfony-bundle-that-validates-anything-and-everything#the-full-story</a></p>

