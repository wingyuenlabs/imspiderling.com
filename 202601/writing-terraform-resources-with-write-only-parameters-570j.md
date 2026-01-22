---
Title: Writing Terraform Resources with Write-Only Parameters
Description: 
Author: drewmullen
Date: 2026-01-22T22:10:57.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Writing Terraform Resources with Write-Only Parameters
</h1>

<p>When building Terraform providers that handle sensitive data like passwords, API tokens, or secret keys, you'll eventually encounter the need for <strong>write-only parameters</strong>. These are values that should be sent to an API but never stored in Terraform state files.</p>

<p>In this post, I'll walk through implementing write-only parameters in a Terraform provider, the challenges involved, and how to balance automation with user control.</p>

<h2>
  
  
  What Are Write-Only Parameters?
</h2>

<p>Write-only parameters (introduced in Terraform 1.11) are resource arguments that:</p>

<ul>
<li>Are sent to the API during creation and updates</li>
<li><strong>Never appear in the Terraform state file</strong></li>
<li>Cannot be read back from the API (the API doesn't return them)</li>
</ul>

<p>This is crucial for security. Without write-only parameters, sensitive values like passwords would be stored in plaintext in your state files.</p>

<p>Here's an example from my fork of the <a href="https://github.com/drewmullen/terraform-provider-aap/blob/main/internal/provider/eda_credential_resource.go" rel="noopener noreferrer">Event Drive Ansible (EDA)</a> provider:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"example"</span> <span class="p">{</span>
  <span class="nx">name</span>               <span class="p">=</span> <span class="s2">"my-api-credential"</span>
  <span class="nx">credential_type_id</span> <span class="p">=</span> <span class="mi">1</span>

  <span class="c1"># Write-only: sent to API but NEVER stored in state</span>
  <span class="nx">inputs_wo</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span>
    <span class="nx">username</span>  <span class="p">=</span> <span class="s2">"service-account"</span>
    <span class="nx">api_token</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">api_token</span>
  <span class="p">})</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  The <code>_version</code> Argument
</h2>

<p>Since Terraform's statefile does not manage the value of the secret, Terraform requires way to know if the secret must be updated. The standard solution to detecting changes in write-only parameters is to add a companion <code>_version</code> field that users must manually increment:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"manual"</span> <span class="p">{</span>
  <span class="nx">name</span>      <span class="p">=</span> <span class="s2">"my-credential"</span>
  <span class="nx">inputs_wo</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span>
    <span class="nx">password</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">new_password</span>
  <span class="p">})</span>

  <span class="c1"># User must increment this to trigger an update</span>
  <span class="nx">inputs_wo_version</span> <span class="p">=</span> <span class="mi">2</span>  <span class="c1"># Changed from 1</span>
<span class="p">}</span>
</code></pre>

</div>



<p><strong>How it works:</strong></p>

<ol>
<li>User updates the secret in <code>inputs_wo</code>
</li>
<li>User manually increments <code>inputs_wo_version</code>
</li>
<li>Terraform detects the version change and triggers an update</li>
<li>The new secret is sent to the API</li>
</ol>

<p>This works, but it's <strong>not user-friendly</strong>. Users have to remember to:</p>

<ul>
<li>Increment the version every time they change secrets</li>
<li>Track version numbers manually</li>
<li>Update two fields instead of one</li>
</ul>

<h2>
  
  
  A Better Approach: Automatic Change Detection
</h2>

<p>Ideally, users shouldn't need to manage version numbers. They should just update the secret value and Terraform should detect the change automatically. But here's the problem:</p>

<p><strong>If <code>inputs_wo</code> is write-only and not in state, how do we detect when it changes?</strong></p>

<p>The answer: <strong>store a deterministic hash in private state</strong>.</p>

<h3>
  
  
  Using Private State and Hashing
</h3>

<p>Terraform's plugin framework provides <a href="https://developer.hashicorp.com/terraform/plugin/framework/resources/private-state" rel="noopener noreferrer">private state</a> - a place to store provider-managed data that's:</p>

<ul>
<li>Stored in the state file (so it persists)</li>
<li>Not visible to users (doesn't appear in <code>terraform show</code>)</li>
<li>Perfect for storing hashes</li>
</ul>

<p>Here's the implementation approach:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="c">// Create operation</span>
<span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">EDACredentialResource</span><span class="p">)</span> <span class="n">Create</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">req</span> <span class="n">resource</span><span class="o">.</span><span class="n">CreateRequest</span><span class="p">,</span> <span class="n">resp</span> <span class="o">*</span><span class="n">resource</span><span class="o">.</span><span class="n">CreateResponse</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">data</span> <span class="n">EDACredentialResourceModel</span>

    <span class="n">req</span><span class="o">.</span><span class="n">Plan</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">data</span><span class="p">)</span>

    <span class="c">// Calculate SHA-256 hash of the inputs</span>
    <span class="n">inputsHash</span> <span class="o">:=</span> <span class="n">calculateInputsHash</span><span class="p">(</span><span class="n">data</span><span class="o">.</span><span class="n">InputsWO</span><span class="o">.</span><span class="n">ValueString</span><span class="p">())</span>

    <span class="c">// Store hash in private state (JSON-wrapped for validity)</span>
    <span class="n">hashJSON</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">`{"hash":"%s"}`</span><span class="p">,</span> <span class="n">inputsHash</span><span class="p">)</span>
    <span class="n">resp</span><span class="o">.</span><span class="n">Private</span><span class="o">.</span><span class="n">SetKey</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="s">"inputs_wo_hash"</span><span class="p">,</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="n">hashJSON</span><span class="p">))</span>

    <span class="c">// Set auto-managed version</span>
    <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span> <span class="o">=</span> <span class="n">tftypes</span><span class="o">.</span><span class="n">Int64Value</span><span class="p">(</span><span class="m">1</span><span class="p">)</span>

    <span class="c">// ... send to API and save state ...</span>
<span class="p">}</span>

<span class="k">func</span> <span class="n">calculateInputsHash</span><span class="p">(</span><span class="n">inputs</span> <span class="kt">string</span><span class="p">)</span> <span class="kt">string</span> <span class="p">{</span>
    <span class="n">h</span> <span class="o">:=</span> <span class="n">sha256</span><span class="o">.</span><span class="n">New</span><span class="p">()</span>
    <span class="n">h</span><span class="o">.</span><span class="n">Write</span><span class="p">([]</span><span class="kt">byte</span><span class="p">(</span><span class="n">inputs</span><span class="p">))</span>
    <span class="k">return</span> <span class="n">hex</span><span class="o">.</span><span class="n">EncodeToString</span><span class="p">(</span><span class="n">h</span><span class="o">.</span><span class="n">Sum</span><span class="p">(</span><span class="no">nil</span><span class="p">))</span>
<span class="p">}</span>
</code></pre>

</div>



<p>In the Update operation, we compare hashes:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">EDACredentialResource</span><span class="p">)</span> <span class="n">Update</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">req</span> <span class="n">resource</span><span class="o">.</span><span class="n">UpdateRequest</span><span class="p">,</span> <span class="n">resp</span> <span class="o">*</span><span class="n">resource</span><span class="o">.</span><span class="n">UpdateResponse</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">data</span><span class="p">,</span> <span class="n">state</span> <span class="n">EDACredentialResourceModel</span>

    <span class="n">req</span><span class="o">.</span><span class="n">Plan</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">data</span><span class="p">)</span>
    <span class="n">req</span><span class="o">.</span><span class="n">State</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">state</span><span class="p">)</span>

    <span class="c">// Get stored hash from private state</span>
    <span class="n">oldHashBytes</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">req</span><span class="o">.</span><span class="n">Private</span><span class="o">.</span><span class="n">GetKey</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="s">"inputs_wo_hash"</span><span class="p">)</span>

    <span class="k">var</span> <span class="n">hashWrapper</span> <span class="k">struct</span> <span class="p">{</span>
        <span class="n">Hash</span> <span class="kt">string</span> <span class="s">`json:"hash"`</span>
    <span class="p">}</span>
    <span class="n">json</span><span class="o">.</span><span class="n">Unmarshal</span><span class="p">(</span><span class="n">oldHashBytes</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">hashWrapper</span><span class="p">)</span>
    <span class="n">oldHash</span> <span class="o">:=</span> <span class="n">hashWrapper</span><span class="o">.</span><span class="n">Hash</span>

    <span class="c">// Calculate new hash</span>
    <span class="n">newHash</span> <span class="o">:=</span> <span class="n">calculateInputsHash</span><span class="p">(</span><span class="n">data</span><span class="o">.</span><span class="n">InputsWO</span><span class="o">.</span><span class="n">ValueString</span><span class="p">())</span>

    <span class="k">if</span> <span class="n">newHash</span> <span class="o">!=</span> <span class="n">oldHash</span> <span class="p">{</span>
        <span class="c">// Inputs changed! Auto-increment version</span>
        <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span> <span class="o">=</span> <span class="n">tftypes</span><span class="o">.</span><span class="n">Int64Value</span><span class="p">(</span><span class="n">state</span><span class="o">.</span><span class="n">InputsWOVersion</span><span class="o">.</span><span class="n">ValueInt64</span><span class="p">()</span> <span class="o">+</span> <span class="m">1</span><span class="p">)</span>

        <span class="c">// Update stored hash</span>
        <span class="n">hashJSON</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">`{"hash":"%s"}`</span><span class="p">,</span> <span class="n">newHash</span><span class="p">)</span>
        <span class="n">resp</span><span class="o">.</span><span class="n">Private</span><span class="o">.</span><span class="n">SetKey</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="s">"inputs_wo_hash"</span><span class="p">,</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="n">hashJSON</span><span class="p">))</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="c">// No change, keep version as-is</span>
        <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span> <span class="o">=</span> <span class="n">state</span><span class="o">.</span><span class="n">InputsWOVersion</span>
    <span class="p">}</span>

    <span class="c">// ... send to API and save state ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Now users can just update the secret and Terraform automatically detects it:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"auto"</span> <span class="p">{</span>
  <span class="nx">name</span>      <span class="p">=</span> <span class="s2">"my-credential"</span>
  <span class="nx">inputs_wo</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span>
    <span class="nx">password</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">new_password</span>  <span class="c1"># Just change this!</span>
  <span class="p">})</span>
  <span class="c1"># inputs_wo_version auto-increments (computed field)</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Best of both worlds
</h2>

<p>While automatic detection is convenient, some users may not want a hash of their secrets stored in state - even if it's SHA-256 and in private state. The hash is still derived from the secret value.</p>

<p><strong>Solution: Support both modes</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">EDACredentialResource</span><span class="p">)</span> <span class="n">Create</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">req</span> <span class="n">resource</span><span class="o">.</span><span class="n">CreateRequest</span><span class="p">,</span> <span class="n">resp</span> <span class="o">*</span><span class="n">resource</span><span class="o">.</span><span class="n">CreateResponse</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">var</span> <span class="n">data</span> <span class="n">EDACredentialResourceModel</span>
    <span class="n">req</span><span class="o">.</span><span class="n">Plan</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">data</span><span class="p">)</span>

    <span class="k">var</span> <span class="n">versionToSet</span> <span class="n">tftypes</span><span class="o">.</span><span class="n">Int64</span>

    <span class="k">if</span> <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span><span class="o">.</span><span class="n">IsNull</span><span class="p">()</span> <span class="o">||</span> <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span><span class="o">.</span><span class="n">IsUnknown</span><span class="p">()</span> <span class="p">{</span>
        <span class="c">// Auto-managed mode: store hash</span>
        <span class="n">inputsHash</span> <span class="o">:=</span> <span class="n">calculateInputsHash</span><span class="p">(</span><span class="n">data</span><span class="o">.</span><span class="n">InputsWO</span><span class="o">.</span><span class="n">ValueString</span><span class="p">())</span>
        <span class="n">hashJSON</span> <span class="o">:=</span> <span class="n">fmt</span><span class="o">.</span><span class="n">Sprintf</span><span class="p">(</span><span class="s">`{"hash":"%s"}`</span><span class="p">,</span> <span class="n">inputsHash</span><span class="p">)</span>
        <span class="n">resp</span><span class="o">.</span><span class="n">Private</span><span class="o">.</span><span class="n">SetKey</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="s">"inputs_wo_hash"</span><span class="p">,</span> <span class="p">[]</span><span class="kt">byte</span><span class="p">(</span><span class="n">hashJSON</span><span class="p">))</span>
        <span class="n">versionToSet</span> <span class="o">=</span> <span class="n">tftypes</span><span class="o">.</span><span class="n">Int64Value</span><span class="p">(</span><span class="m">1</span><span class="p">)</span>
    <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
        <span class="c">// Manual mode: user provided version, don't store hash</span>
        <span class="n">versionToSet</span> <span class="o">=</span> <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span>
    <span class="p">}</span>

    <span class="c">// ... rest of create logic ...</span>
    <span class="n">data</span><span class="o">.</span><span class="n">InputsWOVersion</span> <span class="o">=</span> <span class="n">versionToSet</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Schema definition:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="s">"inputs_wo_version"</span><span class="o">:</span> <span class="n">schema</span><span class="o">.</span><span class="n">Int64Attribute</span><span class="p">{</span>
    <span class="n">Optional</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span>
    <span class="n">Computed</span><span class="o">:</span> <span class="no">true</span><span class="p">,</span>
    <span class="n">PlanModifiers</span><span class="o">:</span> <span class="p">[]</span><span class="n">planmodifier</span><span class="o">.</span><span class="n">Int64</span><span class="p">{</span>
        <span class="n">int64planmodifier</span><span class="o">.</span><span class="n">UseStateForUnknown</span><span class="p">(),</span>
    <span class="p">},</span>
    <span class="n">Description</span><span class="o">:</span> <span class="s">"Version number for managing credential updates. "</span> <span class="o">+</span>
        <span class="s">"If not set, the provider will automatically detect changes to inputs_wo "</span> <span class="o">+</span>
        <span class="s">"using a SHA-256 hash stored in private state. If set manually, you "</span> <span class="o">+</span>
        <span class="s">"control when the credential is updated by incrementing this value yourself."</span><span class="p">,</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Users can choose their preferred mode:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Auto-managed (default)</span>
<span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"auto"</span> <span class="p">{</span>
  <span class="nx">name</span>      <span class="p">=</span> <span class="s2">"auto-credential"</span>
  <span class="nx">inputs_wo</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span> <span class="nx">password</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">pwd</span> <span class="p">})</span>
  <span class="c1"># version auto-increments</span>
<span class="p">}</span>

<span class="c1"># Manual control</span>
<span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"manual"</span> <span class="p">{</span>
  <span class="nx">name</span>              <span class="p">=</span> <span class="s2">"manual-credential"</span>
  <span class="nx">inputs_wo</span>         <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span> <span class="nx">password</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">pwd</span> <span class="p">})</span>
  <span class="nx">inputs_wo_version</span> <span class="p">=</span> <span class="mi">1</span>  <span class="c1"># I control this</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Handling Mode Switching: Just Say No
</h2>

<p>You might think: "What if a user starts with auto-mode and then wants to switch to manual mode?"</p>

<p>Attempting to handle this creates a mess:</p>

<ul>
<li>
<strong>Auto → Manual</strong>: If a user suddenly sets <code>inputs_wo_version</code>, do we remove the hash? What if they're just trying to force an update?</li>
<li>
<strong>Manual → Auto</strong>: We'd need to create a hash, but we can't compare it to the old inputs (they're write-only and not in state), so we can't tell if inputs changed during the transition</li>
<li>The logic becomes complex and error-prone</li>
<li>Edge cases multiply</li>
</ul>

<p><strong>Better approach: Prevent mode switching</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight go"><code><span class="k">func</span> <span class="p">(</span><span class="n">r</span> <span class="o">*</span><span class="n">EDACredentialResource</span><span class="p">)</span> <span class="n">Update</span><span class="p">(</span><span class="n">ctx</span> <span class="n">context</span><span class="o">.</span><span class="n">Context</span><span class="p">,</span> <span class="n">req</span> <span class="n">resource</span><span class="o">.</span><span class="n">UpdateRequest</span><span class="p">,</span> <span class="n">resp</span> <span class="o">*</span><span class="n">resource</span><span class="o">.</span><span class="n">UpdateResponse</span><span class="p">)</span> <span class="p">{</span>
    <span class="c">// Determine current mode from private state</span>
    <span class="n">oldHashBytes</span><span class="p">,</span> <span class="n">_</span> <span class="o">:=</span> <span class="n">req</span><span class="o">.</span><span class="n">Private</span><span class="o">.</span><span class="n">GetKey</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="s">"inputs_wo_hash"</span><span class="p">)</span>
    <span class="n">wasAutoManaged</span> <span class="o">:=</span> <span class="n">oldHashBytes</span> <span class="o">!=</span> <span class="no">nil</span>

    <span class="c">// Determine desired mode from config</span>
    <span class="k">var</span> <span class="n">configModel</span> <span class="n">EDACredentialResourceModel</span>
    <span class="n">req</span><span class="o">.</span><span class="n">Config</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">configModel</span><span class="p">)</span>
    <span class="n">isNowManual</span> <span class="o">:=</span> <span class="o">!</span><span class="n">configModel</span><span class="o">.</span><span class="n">InputsWOVersion</span><span class="o">.</span><span class="n">IsNull</span><span class="p">()</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="n">configModel</span><span class="o">.</span><span class="n">InputsWOVersion</span><span class="o">.</span><span class="n">IsUnknown</span><span class="p">()</span>

    <span class="k">var</span> <span class="n">state</span> <span class="n">EDACredentialResourceModel</span>
    <span class="n">req</span><span class="o">.</span><span class="n">State</span><span class="o">.</span><span class="n">Get</span><span class="p">(</span><span class="n">ctx</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">state</span><span class="p">)</span>
    <span class="n">wasManual</span> <span class="o">:=</span> <span class="o">!</span><span class="n">wasAutoManaged</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="n">state</span><span class="o">.</span><span class="n">InputsWOVersion</span><span class="o">.</span><span class="n">IsNull</span><span class="p">()</span>

    <span class="c">// Prevent mode switching</span>
    <span class="k">if</span> <span class="n">wasAutoManaged</span> <span class="o">&amp;&amp;</span> <span class="n">isNowManual</span> <span class="p">{</span>
        <span class="n">resp</span><span class="o">.</span><span class="n">Diagnostics</span><span class="o">.</span><span class="n">AddError</span><span class="p">(</span>
            <span class="s">"Cannot switch from auto-managed to manual version management"</span><span class="p">,</span>
            <span class="s">"The inputs_wo_version field was previously auto-managed. Once auto-managed, "</span><span class="o">+</span>
            <span class="s">"it cannot be switched to manual mode. If you need to manually control the version, "</span><span class="o">+</span>
            <span class="s">"you must recreate the resource with inputs_wo_version set from the start."</span><span class="p">,</span>
        <span class="p">)</span>
        <span class="k">return</span>
    <span class="p">}</span>

    <span class="k">if</span> <span class="n">wasManual</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="n">isNowManual</span> <span class="p">{</span>
        <span class="n">resp</span><span class="o">.</span><span class="n">Diagnostics</span><span class="o">.</span><span class="n">AddError</span><span class="p">(</span>
            <span class="s">"Cannot switch from manual to auto-managed version management"</span><span class="p">,</span>
            <span class="s">"The inputs_wo_version field was previously manually managed. Once manually managed, "</span><span class="o">+</span>
            <span class="s">"it cannot be switched to auto mode. If you need auto-managed version control, "</span><span class="o">+</span>
            <span class="s">"you must recreate the resource without setting inputs_wo_version."</span><span class="p">,</span>
        <span class="p">)</span>
        <span class="k">return</span>
    <span class="p">}</span>

    <span class="c">// ... rest of update logic for the chosen mode ...</span>
<span class="p">}</span>
</code></pre>

</div>



<p>This gives users a clear error message:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Error: Cannot switch from auto-managed to manual version management

The inputs_wo_version field was previously auto-managed. Once auto-managed,
it cannot be switched to manual mode. If you need to manually control the
version, you must recreate the resource with inputs_wo_version set from the start.
</code></pre>

</div>



<h2>
  
  
  Key Takeaways
</h2>

<p>When implementing write-only parameters in Terraform providers:</p>

<ol>
<li>
<strong>Use private state for hashes</strong> - It's stored but not visible to users</li>
<li>
<strong>Support both auto and manual modes</strong> - Give users choice for their security preferences</li>
<li>
<strong>Prevent mode switching</strong> - The complexity isn't worth it; just error clearly</li>
<li>
<strong>Document clearly</strong> - Users need to understand the trade-offs</li>
</ol>

<p>The complete implementation provides a great user experience while respecting privacy concerns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight hcl"><code><span class="c1"># Most users: auto-managed, no version to track</span>
<span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"api"</span> <span class="p">{</span>
  <span class="nx">name</span>      <span class="p">=</span> <span class="s2">"api-credential"</span>
  <span class="nx">inputs_wo</span> <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span> <span class="nx">token</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">token</span> <span class="p">})</span>
<span class="p">}</span>

<span class="c1"># Privacy-conscious users: manual control, no hash stored</span>
<span class="nx">resource</span> <span class="s2">"aap_eda_credential"</span> <span class="s2">"secure"</span> <span class="p">{</span>
  <span class="nx">name</span>              <span class="p">=</span> <span class="s2">"secure-credential"</span>
  <span class="nx">inputs_wo</span>         <span class="p">=</span> <span class="nx">jsonencode</span><span class="p">({</span> <span class="nx">token</span> <span class="p">=</span> <span class="nx">var</span><span class="p">.</span><span class="nx">token</span> <span class="p">})</span>
  <span class="nx">inputs_wo_version</span> <span class="p">=</span> <span class="mi">1</span>
<span class="p">}</span>
</code></pre>

</div>



<p>Both approaches are valid. Both are clear. And switching between them requires a resource recreation, which is easy to communicate and reason about.</p>




<p><em>Have you implemented write-only parameters in your Terraform provider? What challenges did you face? Let me know in the comments!</em></p>

