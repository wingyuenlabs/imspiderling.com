---
Title: How to Look Up NCCI Workers Compensation Class Codes Programmatically
Description: 
Author: easysolutions906
Date: 2026-03-16T21:39:16.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  How to Look Up NCCI Workers Compensation Class Codes Programmatically
</h1>

<p>Every medical claim submitted to Medicare goes through the National Correct Coding Initiative (NCCI) edit system before payment is approved. CMS publishes tens of thousands of Procedure-to-Procedure (PTP) edits and Medically Unlikely Edits (MUE) that define which CPT/HCPCS code pairs can be billed together and how many units of a given code are reasonable per encounter. If your billing system does not check claims against these edits before submission, you are going to get denials.</p>

<p>This article explains what NCCI edits are, how PTP bundling and MUE limits work, and how to validate claims programmatically.</p>

<h2>
  
  
  What are NCCI edits
</h2>

<p>The NCCI was established by CMS in 1996 to promote correct coding and prevent improper payment for Medicare Part B claims. There are two types of edits:</p>

<p><strong>PTP (Procedure-to-Procedure) edits</strong> define pairs of CPT/HCPCS codes that should not normally be billed together on the same claim. One code in the pair is the "comprehensive" code (the broader procedure) and the other is the "component" code (a procedure that is considered part of the comprehensive one). For example, if a surgeon performs a total knee replacement, certain ancillary procedures performed during the same session are bundled into the primary code and should not be billed separately.</p>

<p><strong>MUE (Medically Unlikely Edits)</strong> set the maximum number of units that can be reported for a single CPT/HCPCS code per line, per day, or per encounter. An MUE of 1 for a bilateral procedure means you cannot bill 3 units of that code on a single claim line without triggering a denial.</p>

<p>CMS publishes updated NCCI edit files quarterly. Each quarterly release adds, modifies, or deletes edit pairs. A code pair that was billable together last quarter may not be this quarter.</p>

<h2>
  
  
  The modifier indicator
</h2>

<p>Not every PTP edit is an absolute block. Each edit has a modifier indicator that determines whether the bundling can be overridden:</p>

<ul>
<li>
<strong>Modifier Indicator 0</strong>: Not allowed. No modifier will unbundle this pair. The component code simply cannot be billed with the comprehensive code.</li>
<li>
<strong>Modifier Indicator 1</strong>: Modifier allowed. If the services are truly distinct (different anatomic site, different encounter, different practitioner), you can append modifier 25, 59, XE, XS, XP, or XU to the component code to unbundle the edit.</li>
<li>
<strong>Modifier Indicator 9</strong>: Not applicable.</li>
</ul>

<p>The -59 modifier and the newer X{ESPU} modifiers are the most commonly used unbundling modifiers. CMS introduced the X modifiers (XE, XS, XP, XU) to provide more specificity than the broad -59, and many payers now prefer them.</p>

<h2>
  
  
  Why this matters for billing teams
</h2>

<p>The financial impact of NCCI edits is significant:</p>

<ul>
<li>
<strong>Denied claims</strong> cost money to rework and resubmit. The average cost to rework a denied claim is $25-$50.</li>
<li>
<strong>Improper unbundling</strong> -- billing codes separately that should be bundled -- is one of the top reasons for Medicare audits. OIG recoveries for unbundling violations run in the millions annually.</li>
<li>
<strong>MUE violations</strong> are an automatic denial. If the MUE for a code is 2 and you bill 3 units, the excess units are denied with no appeal path.</li>
<li>
<strong>Quarterly updates</strong> mean your edit logic must stay current. A code pair that was clean in Q1 might have a new edit in Q2.</li>
</ul>

<h2>
  
  
  Validating a code pair
</h2>

<p>The MCP server <code>@easysolutions906/ncci-api</code> embeds the full CMS NCCI PTP edit table and MUE data, updated quarterly.</p>

<h3>
  
  
  Check if two codes can be billed together
</h3>

<p>Validating codes 99213 (office visit) and 36415 (venipuncture):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"code1"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"code2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"canBillTogether"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"hasEdit"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"comprehensiveCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"componentCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"modifierIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
  </span><span class="nl">"modifierIndicatorDescription"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Modifier allowed — use modifier 25, 59, XE, XS, XP, or XU to unbundle"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"modifierApplied"</span><span class="p">:</span><span class="w"> </span><span class="kc">null</span><span class="p">,</span><span class="w">
  </span><span class="nl">"unbundledByModifier"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"rationale"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standards of medical/surgical practice"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="s2">"NCCI edit: 99213 bundles 36415. Apply modifier 25/59/XE/XS/XP/XU to 36415 if services are distinct."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The response tells you exactly what is happening: 99213 is the comprehensive code, 36415 is the component, and modifier indicator 1 means you can unbundle with the appropriate modifier. If the venipuncture was for a separately identifiable service, appending modifier 59 to 36415 resolves the edit.</p>

<h3>
  
  
  Applying a modifier to unbundle
</h3>

<p>If modifiers are included in the request, the API evaluates whether they resolve the edit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"code1"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"code2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"canBillTogether"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"hasEdit"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"comprehensiveCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"componentCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"modifierIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
  </span><span class="nl">"modifierApplied"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"59"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"unbundledByModifier"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"message"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Edit exists but unbundled by modifier 59 on 36415. These codes can be billed together."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This is the key workflow: check the pair, see if an edit exists, check the modifier indicator, and if modifier is allowed, apply the correct one.</p>

<h2>
  
  
  Validating a full claim
</h2>

<p>Real claims have more than two codes. The <code>ncci_validate_claim</code> tool checks every code pair combination on a claim and also verifies MUE limits.</p>

<p>Submitting a claim with codes 99213, 36415, 80053, and 85025:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"codesSubmitted"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"99213"</span><span class="p">,</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w"> </span><span class="s2">"80053"</span><span class="p">,</span><span class="w"> </span><span class="s2">"85025"</span><span class="p">],</span><span class="w">
  </span><span class="nl">"pairsChecked"</span><span class="p">:</span><span class="w"> </span><span class="mi">6</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editIssues"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
  </span><span class="nl">"mueIssues"</span><span class="p">:</span><span class="w"> </span><span class="mi">0</span><span class="p">,</span><span class="w">
  </span><span class="nl">"blocked"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
  </span><span class="nl">"unbundled"</span><span class="p">:</span><span class="w"> </span><span class="mi">0</span><span class="p">,</span><span class="w">
  </span><span class="nl">"clean"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
  </span><span class="nl">"issues"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"code1"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"code2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"canBillTogether"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
      </span><span class="nl">"comprehensiveCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"componentCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"modifierIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"code1"</span><span class="p">:</span><span class="w"> </span><span class="s2">"80053"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"code2"</span><span class="p">:</span><span class="w"> </span><span class="s2">"85025"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"canBillTogether"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
      </span><span class="nl">"comprehensiveCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"80053"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"componentCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"85025"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"modifierIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">0</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">],</span><span class="w">
  </span><span class="nl">"summary"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Claim has 2 blocked edit pair(s) and 0 MUE violation(s)."</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The first issue (99213/36415) can be resolved with a modifier. The second issue (80053/85025) has modifier indicator 0, meaning it cannot be unbundled -- one of those codes must be removed from the claim.</p>

<h2>
  
  
  Looking up MUE limits
</h2>

<p>The <code>ncci_mue</code> tool returns the maximum units allowed for a code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"code"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"found"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
  </span><span class="nl">"description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Office or other outpatient visit"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"practitionerMue"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
  </span><span class="nl">"facilityMue"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
  </span><span class="nl">"rationale"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Nature of Service/Procedure"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"adjudicationIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
  </span><span class="nl">"adjudicationDescription"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Per Day Edit — applies per beneficiary per day"</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>An MUE of 1 with adjudication indicator 2 means this code can only be billed once per beneficiary per day. Submitting 2 units on the same date of service will trigger an automatic denial.</p>

<p>The adjudication indicator matters:</p>

<ul>
<li>
<strong>1 (Claim Line Edit)</strong>: The limit applies per claim line. You cannot split the units across multiple lines to avoid the edit.</li>
<li>
<strong>2 (Per Day)</strong>: The limit applies per beneficiary per day, across all claim lines.</li>
<li>
<strong>3 (Per Encounter)</strong>: The limit applies per encounter, which is more restrictive than per day.</li>
</ul>

<h2>
  
  
  Browsing edits for a code
</h2>

<p>The <code>ncci_edits</code> tool returns all PTP edits associated with a specific code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"code"</span><span class="p">:</span><span class="w"> </span><span class="s2">"99213"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"editCount"</span><span class="p">:</span><span class="w"> </span><span class="mi">47</span><span class="p">,</span><span class="w">
  </span><span class="nl">"edits"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"pairedCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36415"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"comprehensive"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"modifierIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="nl">"rationale"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standards of medical/surgical practice"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"pairedCodeDescription"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Collection of venous blood by venipuncture"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="p">{</span><span class="w">
      </span><span class="nl">"pairedCode"</span><span class="p">:</span><span class="w"> </span><span class="s2">"36416"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"role"</span><span class="p">:</span><span class="w"> </span><span class="s2">"comprehensive"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"modifierIndicator"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="nl">"rationale"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standards of medical/surgical practice"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"pairedCodeDescription"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Collection of capillary blood specimen"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This is useful for building billing rules engines or populating a reference table in your practice management system.</p>

<h2>
  
  
  Searching edits by keyword
</h2>

<p>The <code>ncci_search</code> tool lets you search across the entire edit database by code, procedure name, or category:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"query"</span><span class="p">:</span><span class="w"> </span><span class="s2">"arthroscopy"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"ptpEdits"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"total"</span><span class="p">:</span><span class="w"> </span><span class="mi">312</span><span class="p">,</span><span class="w">
    </span><span class="nl">"results"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="err">...</span><span class="p">]</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="nl">"mue"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"total"</span><span class="p">:</span><span class="w"> </span><span class="mi">24</span><span class="p">,</span><span class="w">
    </span><span class="nl">"results"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="err">...</span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<h2>
  
  
  Setting up the MCP server
</h2>

<p>Add to your Claude Desktop or Cursor configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"mcpServers"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"ncci"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"npx"</span><span class="p">,</span><span class="w">
      </span><span class="nl">"args"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="s2">"-y"</span><span class="p">,</span><span class="w"> </span><span class="s2">"@easysolutions906/ncci-api"</span><span class="p">]</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>After restarting, you can ask natural language questions like:</p>

<ul>
<li>"Can I bill 99213 and 36415 together?"</li>
<li>"What is the MUE limit for 99213?"</li>
<li>"Show all NCCI edits for CPT 27447"</li>
<li>"Validate this claim: 99213, 36415, 80053, 85025"</li>
<li>"Will modifier 59 on 36415 unbundle the edit with 99213?"</li>
</ul>

<p>The server exposes five tools: <code>ncci_validate_pair</code>, <code>ncci_validate_claim</code>, <code>ncci_edits</code>, <code>ncci_mue</code>, and <code>ncci_search</code>.</p>

<h2>
  
  
  Practical integration example
</h2>

<p>Here is a common pattern for a claims scrubber that checks a claim before submission:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">scrubClaim</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">codes</span><span class="p">,</span> <span class="nx">modifiers</span> <span class="o">=</span> <span class="p">{})</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://your-api-url.com/validate/claim</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
      <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
      <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NCCI_API_KEY</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="nx">codes</span><span class="p">,</span> <span class="nx">modifiers</span> <span class="p">}),</span>
  <span class="p">});</span>
  <span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

  <span class="k">if </span><span class="p">(</span><span class="nx">result</span><span class="p">.</span><span class="nx">clean</span><span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="na">pass</span><span class="p">:</span> <span class="kc">true</span><span class="p">,</span> <span class="na">message</span><span class="p">:</span> <span class="dl">'</span><span class="s1">Claim passes NCCI validation</span><span class="dl">'</span> <span class="p">};</span>
  <span class="p">}</span>

  <span class="kd">const</span> <span class="nx">actionItems</span> <span class="o">=</span> <span class="nx">result</span><span class="p">.</span><span class="nx">issues</span>
    <span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">issue</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">issue</span><span class="p">.</span><span class="nx">canBillTogether</span><span class="p">)</span>
    <span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">issue</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="k">if </span><span class="p">(</span><span class="nx">issue</span><span class="p">.</span><span class="nx">modifierIndicator</span> <span class="o">===</span> <span class="mi">1</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">return</span> <span class="s2">`Add modifier 59/XE/XS/XP/XU to </span><span class="p">${</span><span class="nx">issue</span><span class="p">.</span><span class="nx">componentCode</span><span class="p">}</span><span class="s2"> to unbundle from </span><span class="p">${</span><span class="nx">issue</span><span class="p">.</span><span class="nx">comprehensiveCode</span><span class="p">}</span><span class="s2">`</span><span class="p">;</span>
      <span class="p">}</span>
      <span class="k">return</span> <span class="s2">`Remove </span><span class="p">${</span><span class="nx">issue</span><span class="p">.</span><span class="nx">componentCode</span><span class="p">}</span><span class="s2"> — it cannot be billed with </span><span class="p">${</span><span class="nx">issue</span><span class="p">.</span><span class="nx">comprehensiveCode</span><span class="p">}</span><span class="s2"> (no modifier override)`</span><span class="p">;</span>
    <span class="p">});</span>

  <span class="kd">const</span> <span class="nx">mueItems</span> <span class="o">=</span> <span class="nx">result</span><span class="p">.</span><span class="nx">mue</span><span class="p">.</span><span class="nf">map</span><span class="p">((</span><span class="nx">m</span><span class="p">)</span> <span class="o">=&gt;</span>
    <span class="s2">`Reduce </span><span class="p">${</span><span class="nx">m</span><span class="p">.</span><span class="nx">code</span><span class="p">}</span><span class="s2"> from </span><span class="p">${</span><span class="nx">m</span><span class="p">.</span><span class="nx">unitsOnClaim</span><span class="p">}</span><span class="s2"> units to </span><span class="p">${</span><span class="nx">m</span><span class="p">.</span><span class="nx">mueLimit</span><span class="p">}</span><span class="s2"> (MUE limit)`</span>
  <span class="p">);</span>

  <span class="k">return</span> <span class="p">{</span>
    <span class="na">pass</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
    <span class="na">actionItems</span><span class="p">:</span> <span class="p">[...</span><span class="nx">actionItems</span><span class="p">,</span> <span class="p">...</span><span class="nx">mueItems</span><span class="p">],</span>
  <span class="p">};</span>
<span class="p">};</span>

<span class="c1">// Check a claim before submission</span>
<span class="kd">const</span> <span class="nx">result</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">scrubClaim</span><span class="p">(</span>
  <span class="p">[</span><span class="dl">'</span><span class="s1">99213</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">36415</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">80053</span><span class="dl">'</span><span class="p">],</span>
  <span class="p">{</span> <span class="dl">'</span><span class="s1">36415</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">59</span><span class="dl">'</span> <span class="p">}</span>
<span class="p">);</span>
<span class="c1">// { pass: true, message: 'Claim passes NCCI validation' }</span>
</code></pre>

</div>



<p>Run this before every claim submission. Flag blocked pairs for coder review, auto-suggest modifiers where indicator 1 allows it, and hard-stop any MUE violations.</p>

<h2>
  
  
  Batch validation
</h2>

<p>For high-volume billing operations, the REST API supports batch validation of multiple claims in a single request:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight javascript"><code><span class="kd">const</span> <span class="nx">batchScrub</span> <span class="o">=</span> <span class="k">async </span><span class="p">(</span><span class="nx">claims</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">res</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">fetch</span><span class="p">(</span><span class="dl">'</span><span class="s1">https://your-api-url.com/validate/batch</span><span class="dl">'</span><span class="p">,</span> <span class="p">{</span>
    <span class="na">method</span><span class="p">:</span> <span class="dl">'</span><span class="s1">POST</span><span class="dl">'</span><span class="p">,</span>
    <span class="na">headers</span><span class="p">:</span> <span class="p">{</span>
      <span class="dl">'</span><span class="s1">Content-Type</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">application/json</span><span class="dl">'</span><span class="p">,</span>
      <span class="dl">'</span><span class="s1">x-api-key</span><span class="dl">'</span><span class="p">:</span> <span class="nx">process</span><span class="p">.</span><span class="nx">env</span><span class="p">.</span><span class="nx">NCCI_API_KEY</span><span class="p">,</span>
    <span class="p">},</span>
    <span class="na">body</span><span class="p">:</span> <span class="nx">JSON</span><span class="p">.</span><span class="nf">stringify</span><span class="p">({</span> <span class="nx">claims</span> <span class="p">}),</span>
  <span class="p">});</span>

  <span class="kd">const</span> <span class="p">{</span> <span class="nx">results</span> <span class="p">}</span> <span class="o">=</span> <span class="k">await</span> <span class="nx">res</span><span class="p">.</span><span class="nf">json</span><span class="p">();</span>

  <span class="kd">const</span> <span class="nx">flagged</span> <span class="o">=</span> <span class="nx">results</span><span class="p">.</span><span class="nf">filter</span><span class="p">((</span><span class="nx">r</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="o">!</span><span class="nx">r</span><span class="p">.</span><span class="nx">clean</span><span class="p">);</span>
  <span class="k">return</span> <span class="p">{</span>
    <span class="na">total</span><span class="p">:</span> <span class="nx">results</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
    <span class="na">clean</span><span class="p">:</span> <span class="nx">results</span><span class="p">.</span><span class="nx">length</span> <span class="o">-</span> <span class="nx">flagged</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
    <span class="na">flagged</span><span class="p">:</span> <span class="nx">flagged</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
    <span class="na">details</span><span class="p">:</span> <span class="nx">flagged</span><span class="p">,</span>
  <span class="p">};</span>
<span class="p">};</span>

<span class="c1">// Scrub an entire day's claims</span>
<span class="kd">const</span> <span class="nx">dayResults</span> <span class="o">=</span> <span class="k">await</span> <span class="nf">batchScrub</span><span class="p">([</span>
  <span class="p">{</span> <span class="na">codes</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">99213</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">36415</span><span class="dl">'</span><span class="p">],</span> <span class="na">modifiers</span><span class="p">:</span> <span class="p">{</span> <span class="dl">'</span><span class="s1">36415</span><span class="dl">'</span><span class="p">:</span> <span class="dl">'</span><span class="s1">59</span><span class="dl">'</span> <span class="p">}</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">codes</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">27447</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">27486</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
  <span class="p">{</span> <span class="na">codes</span><span class="p">:</span> <span class="p">[</span><span class="dl">'</span><span class="s1">99214</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">80053</span><span class="dl">'</span><span class="p">,</span> <span class="dl">'</span><span class="s1">85025</span><span class="dl">'</span><span class="p">]</span> <span class="p">},</span>
<span class="p">]);</span>
</code></pre>

</div>



<h2>
  
  
  Data freshness
</h2>

<p>CMS publishes NCCI edit updates quarterly (January, April, July, October). Each update can add new edits, delete existing ones, or change modifier indicators. The <code>@easysolutions906/ncci-api</code> package includes a <code>data-info</code> endpoint that returns the CMS quarter, build date, and record counts so you can verify the data is current. The embedded data is rebuilt from CMS source files with each package update.</p>

<p>NCCI validation is one of those problems that every medical billing system needs to solve but few handle well. The edit tables are large, the quarterly updates are easy to miss, and the modifier logic has enough nuance that getting it wrong costs real money in denials and rework. Having the full CMS edit dataset available as both MCP tools and a REST API means you can integrate claims scrubbing wherever it is needed -- inside an AI coding assistant, a practice management system, or a standalone claims review workflow.</p>

