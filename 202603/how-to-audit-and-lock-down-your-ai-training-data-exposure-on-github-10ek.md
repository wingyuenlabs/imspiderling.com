---
Title: How to Audit and Lock Down Your AI Training Data Exposure on GitHub
Description: 
Author: Alan West
Date: 2026-03-26T21:49:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>So GitHub just updated their Copilot data usage policy. The short version: interaction data from all user tiers — including free users — will be used to train and improve their models, and it's opt-in by default. If you're reading this with a knot in your stomach, you're not alone.</p>

<p>Let's skip the outrage cycle and focus on the actual problem: how do you figure out what data you're exposing, and how do you lock it down?</p>

<h2>
  
  
  The Real Problem: You Don't Know What's Being Sent
</h2>

<p>Here's what caught most people off guard. It's not just your code that's being collected — it's your <em>interaction data</em>. That means prompts, suggestions you accepted or rejected, and the context surrounding those interactions. If you've been using Copilot to help debug a production issue at 2am, that context went somewhere.</p>

<p>The frustrating part isn't that data collection exists. It's that the defaults changed silently, and most developers won't notice until someone posts about it on Reddit.</p>

<h2>
  
  
  Step 1: Check Your Current Settings
</h2>

<p>First, let's see where you actually stand. Head to your GitHub settings and check what's enabled:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://github.com/settings/copilot
</code></pre>

</div>



<p>You're looking for the section about data usage and telemetry. On individual accounts, you should see toggles for allowing GitHub to use your data for product improvement. On organization accounts, admins control this at the org level.</p>

<p>If you're managing a team, check the org-level settings too:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>https://github.com/organizations/YOUR_ORG/settings/copilot
</code></pre>

</div>



<h2>
  
  
  Step 2: Audit What You've Already Sent
</h2>

<p>Before you flip any switches, it helps to know what's already out there. You can request a data export from GitHub to see what they've collected:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Request your GitHub data export via the API</span>
curl <span class="nt">-X</span> POST <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$GITHUB_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  https://api.github.com/user/migrations <span class="se">\</span>
  <span class="nt">-d</span> <span class="s1">'{"repositories": [], "lock_repositories": false}'</span>
</code></pre>

</div>



<p>This won't show you Copilot interaction data specifically, but it's a good starting point for understanding your overall data footprint. For Copilot-specific data, you'll need to go through the <a href="https://support.github.com/request" rel="noopener noreferrer">GitHub privacy contact page</a> and submit a Subject Access Request.</p>

<h2>
  
  
  Step 3: Opt Out Properly
</h2>

<p>Here's the step-by-step to actually opt out. This matters because there are multiple settings and missing one means you're still sharing data.</p>

<p><strong>For individual users:</strong></p>

<ol>
<li>Go to <code>Settings → Copilot</code>
</li>
<li>Find the telemetry/data sharing section</li>
<li>Disable "Allow GitHub to use my data for product improvement" (or however they've worded it this week)</li>
<li>While you're there, review the code snippet collection settings too</li>
</ol>

<p><strong>For organization admins:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Check your org's current Copilot policy via the API</span>
curl <span class="nt">-s</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Authorization: Bearer </span><span class="nv">$GITHUB_TOKEN</span><span class="s2">"</span> <span class="se">\</span>
  <span class="nt">-H</span> <span class="s2">"Accept: application/vnd.github+json"</span> <span class="se">\</span>
  https://api.github.com/orgs/YOUR_ORG/copilot/billing <span class="se">\</span>
  | python3 <span class="nt">-m</span> json.tool
</code></pre>

</div>



<p>Org admins can enforce opt-out across the entire organization, which is the move if you're dealing with proprietary code.</p>

<h2>
  
  
  Step 4: Set Up a Local Safety Net
</h2>

<p>Opting out is one thing, but if you want real control, you need to think about what leaves your machine in the first place. Here's a <code>.gitconfig</code> approach to remind yourself:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Add a pre-commit hook that warns about sensitive patterns</span>
<span class="c"># Save this as .git/hooks/pre-commit and chmod +x it</span>

<span class="c">#!/bin/bash</span>
<span class="c"># Check for common sensitive patterns before commit</span>
<span class="nv">SENSITIVE_PATTERNS</span><span class="o">=(</span>
  <span class="s2">"API_KEY"</span>
  <span class="s2">"SECRET"</span>
  <span class="s2">"password"</span>
  <span class="s2">"private_key"</span>
  <span class="s2">"BEGIN RSA"</span>
<span class="o">)</span>

<span class="k">for </span>pattern <span class="k">in</span> <span class="s2">"</span><span class="k">${</span><span class="nv">SENSITIVE_PATTERNS</span><span class="p">[@]</span><span class="k">}</span><span class="s2">"</span><span class="p">;</span> <span class="k">do
  if </span>git diff <span class="nt">--cached</span> <span class="nt">--diff-filter</span><span class="o">=</span>ACM | <span class="nb">grep</span> <span class="nt">-qi</span> <span class="s2">"</span><span class="nv">$pattern</span><span class="s2">"</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"WARNING: Possible sensitive data detected: </span><span class="nv">$pattern</span><span class="s2">"</span>
    <span class="nb">echo</span> <span class="s2">"Review your staged changes before committing."</span>
    <span class="c"># Uncomment the next line to block the commit entirely</span>
    <span class="c"># exit 1</span>
  <span class="k">fi
done</span>
</code></pre>

</div>



<p>This doesn't stop Copilot from seeing your code in the editor — that ship sails the moment you type. But it adds a layer of awareness about what's in your repo.</p>

<h2>
  
  
  Step 5: Consider Your Editor-Level Options
</h2>

<p>If you're really serious about this, you can control things at the editor level. VS Code lets you configure which files Copilot can access:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="err">//</span><span class="w"> </span><span class="err">In</span><span class="w"> </span><span class="err">your</span><span class="w"> </span><span class="err">VS</span><span class="w"> </span><span class="err">Code</span><span class="w"> </span><span class="err">settings.json</span><span class="w">
</span><span class="p">{</span><span class="w">
  </span><span class="nl">"github.copilot.enable"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"*"</span><span class="p">:</span><span class="w"> </span><span class="kc">true</span><span class="p">,</span><span class="w">
    </span><span class="nl">"plaintext"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
    </span><span class="nl">"markdown"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="p">,</span><span class="w">
    </span><span class="nl">"yaml"</span><span class="p">:</span><span class="w"> </span><span class="kc">false</span><span class="w">  </span><span class="err">//</span><span class="w"> </span><span class="err">often</span><span class="w"> </span><span class="err">contains</span><span class="w"> </span><span class="err">config/secrets</span><span class="w">
  </span><span class="p">},</span><span class="w">
  </span><span class="err">//</span><span class="w"> </span><span class="err">Disable</span><span class="w"> </span><span class="err">Copilot</span><span class="w"> </span><span class="err">for</span><span class="w"> </span><span class="err">specific</span><span class="w"> </span><span class="err">file</span><span class="w"> </span><span class="err">patterns</span><span class="w">
  </span><span class="nl">"github.copilot.advanced"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"excludeFiles"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="s2">"**/.env*"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"**/secrets/**"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"**/config/production.*"</span><span class="w">
    </span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>This is actually my preferred approach. I keep Copilot on for the code I'm writing but exclude anything that might contain infrastructure secrets or sensitive configuration.</p>

<h2>
  
  
  The Bigger Picture: Building Better Habits
</h2>

<p>Honestly, this GitHub situation is a symptom of a broader problem. Every AI coding tool is hungry for training data, and the incentives will always push toward more collection, not less. Here's what I've started doing across all my projects:</p>

<ul>
<li>
<strong>Separate sensitive config from code.</strong> Use environment variables and external secret managers. If it never hits your editor, it can't be collected.</li>
<li>
<strong>Review your extensions periodically.</strong> Not just Copilot — any AI extension could be phoning home. Check what telemetry each one sends.</li>
<li>
<strong>Use <code>.gitignore</code> patterns aggressively.</strong> Even for files that shouldn't exist in the repo, having the ignore pattern is a safety net.</li>
<li>
<strong>Read the policy updates.</strong> Yeah, I know. Nobody does this. But a 5-minute skim every few months beats finding out on Reddit that your data practices changed.</li>
</ul>

<h2>
  
  
  Prevention: Automate the Boring Parts
</h2>

<p>Set up a quarterly reminder to audit your settings. Seriously, put it in your calendar. These policies change, defaults get reset during updates, and new features get opted in without fanfare.</p>

<p>For teams, consider writing this into your onboarding docs. New developer joins? Part of their setup checklist should include reviewing data sharing settings on every tool they install. It takes five minutes and saves everyone a headache later.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Quick audit script — run this periodically</span>
<span class="c">#!/bin/bash</span>
<span class="nb">echo</span> <span class="s2">"=== GitHub Copilot Settings Audit ==="</span>
<span class="nb">echo</span> <span class="s2">"Checking GitHub CLI auth status..."</span>
gh auth status 2&gt;&amp;1
<span class="nb">echo</span> <span class="s2">""</span>
<span class="nb">echo</span> <span class="s2">"Current Copilot settings:"</span>
gh api /user/copilot 2&gt;/dev/null <span class="o">||</span> <span class="nb">echo</span> <span class="s2">"Could not fetch Copilot settings (check your token scopes)"</span>
<span class="nb">echo</span> <span class="s2">""</span>
<span class="nb">echo</span> <span class="s2">"Remember to also check:"</span>
<span class="nb">echo</span> <span class="s2">"  - https://github.com/settings/copilot"</span>
<span class="nb">echo</span> <span class="s2">"  - Your org settings if applicable"</span>
<span class="nb">echo</span> <span class="s2">"  - VS Code extension telemetry settings"</span>
</code></pre>

</div>



<p>The goal isn't paranoia. It's informed consent. Use the tools if they make you productive — just know what you're trading for that productivity, and make sure it's a trade you're actually choosing to make.</p>

