---
Title: Building Guardrails for AI Coding Assistants: A PreToolUse Hook System for Claude Code
Description: 
Author: Mike Lane
Date: 2026-01-18T22:09:55.000Z
Robots: noindex,nofollow
Template: index
---
<p>AI coding assistants implement code quickly but follow their training defaults, not your project rules. They do not know your team bans <code>git reset --hard</code>, requires GPG-signed commits, or uses a different GitHub org than your username.</p>

<p>Claude Code's PreToolUse hook system intercepts tool calls before execution, blocks dangerous operations, and injects contextual guidance. This article walks through a production implementation consisting of four specialized hooks.</p>

<blockquote>
<p><strong>Version Note</strong>: The <code>additionalContext</code> feature used in this article requires <a href="https://github.com/anthropics/claude-code/releases/tag/v2.1.9" rel="noopener noreferrer">Claude Code v2.1.9</a> or later (released January 16, 2026). Earlier versions supported blocking hooks but not context injection. This capability originated from <a href="https://github.com/anthropics/claude-code/issues/15345" rel="noopener noreferrer">feature request #15345</a>.</p>
</blockquote>

<h2>
  
  
  The Hook Contract
</h2>

<p>A PreToolUse hook is an executable that receives JSON on stdin and communicates via exit codes and stdout/stderr:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Input (stdin)
</span><span class="p">{</span>
  <span class="sh">"</span><span class="s">tool_name</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Bash</span><span class="sh">"</span><span class="p">,</span>
  <span class="sh">"</span><span class="s">tool_input</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span><span class="sh">"</span><span class="s">command</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">git reset --hard HEAD~3</span><span class="sh">"</span><span class="p">}</span>
<span class="p">}</span>

<span class="c1"># Exit code 0: Allow (stdout parsed for additionalContext)
# Exit code 2: Block (stderr displayed to user)
</span></code></pre>

</div>



<p>The hook output format for allowing with context injection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="p">{</span>
  <span class="sh">"</span><span class="s">hookSpecificOutput</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
    <span class="sh">"</span><span class="s">hookEventName</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">PreToolUse</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">permissionDecision</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">allow</span><span class="sh">"</span><span class="p">,</span>
    <span class="sh">"</span><span class="s">additionalContext</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Your reminder or context here</span><span class="sh">"</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Hook 1: Safety Guards
</h2>

<p>The first hook blocks destructive operations before they execute.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">BLOCKING_RULES</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="n">SafetyRule</span><span class="p">]</span> <span class="o">=</span> <span class="p">[</span>
    <span class="nc">SafetyRule</span><span class="p">(</span>
        <span class="n">pattern</span><span class="o">=</span><span class="sa">r</span><span class="sh">"</span><span class="s">git\s+reset\s+--hard</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">block</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">BLOCKED: Use `git stash` or `git rebase` instead. </span><span class="sh">"</span>
        <span class="sh">"</span><span class="s">`git reset --hard` destroys uncommitted work.</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
    <span class="nc">SafetyRule</span><span class="p">(</span>
        <span class="n">pattern</span><span class="o">=</span><span class="sa">r</span><span class="sh">"</span><span class="s">--no-gpg-sign</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">block</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">BLOCKED: GPG signing is mandatory. Fix GPG issues, don</span><span class="sh">'</span><span class="s">t bypass.</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
    <span class="nc">SafetyRule</span><span class="p">(</span>
        <span class="n">pattern</span><span class="o">=</span><span class="sa">r</span><span class="sh">"</span><span class="s">git\s+(?:add\s+)?(?:-A|--all|\.\s*$)</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">block</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">BLOCKED: Explicitly stage files to avoid committing unwanted files.</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
<span class="p">]</span>
</code></pre>

</div>



<p>When asked to commit changes, Claude often runs <code>git add -A</code> to stage everything. In repositories with build artifacts or <code>.env</code> files, this stages files that should never be committed. Forcing explicit staging ensures intentional commits.</p>

<p>Warning rules follow the same pattern but allow the operation while injecting context:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">WARNING_RULES</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="n">SafetyRule</span><span class="p">]</span> <span class="o">=</span> <span class="p">[</span>
    <span class="nc">SafetyRule</span><span class="p">(</span>
        <span class="n">pattern</span><span class="o">=</span><span class="sa">r</span><span class="sh">"</span><span class="s">git\s+push\s+--force(?!\s*-with-lease)</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">warn</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">WARNING: Use `--force-with-lease` for safer force push.</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
    <span class="nc">SafetyRule</span><span class="p">(</span>
        <span class="n">pattern</span><span class="o">=</span><span class="sa">r</span><span class="sh">"</span><span class="s">\.env</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">action</span><span class="o">=</span><span class="sh">"</span><span class="s">warn</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">WARNING: This file may contain secrets. Never commit .env files.</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
<span class="p">]</span>
</code></pre>

</div>



<p>The execution logic checks blocking rules first, then warnings:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">check_command</span><span class="p">(</span><span class="n">command</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">tuple</span><span class="p">[</span><span class="nb">str</span><span class="p">,</span> <span class="nb">str</span><span class="p">]:</span>
    <span class="k">for</span> <span class="n">rule</span> <span class="ow">in</span> <span class="n">BLOCKING_RULES</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">rule</span><span class="p">.</span><span class="nf">matches</span><span class="p">(</span><span class="n">command</span><span class="p">):</span>
            <span class="nf">return </span><span class="p">(</span><span class="sh">"</span><span class="s">block</span><span class="sh">"</span><span class="p">,</span> <span class="n">rule</span><span class="p">.</span><span class="n">message</span><span class="p">)</span>

    <span class="k">for</span> <span class="n">rule</span> <span class="ow">in</span> <span class="n">WARNING_RULES</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">rule</span><span class="p">.</span><span class="nf">matches</span><span class="p">(</span><span class="n">command</span><span class="p">):</span>
            <span class="nf">return </span><span class="p">(</span><span class="sh">"</span><span class="s">warn</span><span class="sh">"</span><span class="p">,</span> <span class="n">rule</span><span class="p">.</span><span class="n">message</span><span class="p">)</span>

    <span class="nf">return </span><span class="p">(</span><span class="sh">"</span><span class="s">allow</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>
</code></pre>

</div>



<h2>
  
  
  Hook 2: Context Injection for GitHub API Calls
</h2>

<p>Repository paths do not always match GitHub organization names. My local checkout lives at <code>~/dev/AcmeCorp/webapp</code>, but the GitHub owner is <code>AcmeCorp</code>, not my username <code>jdoe</code>. Without intervention, Claude defaults to the wrong owner for API calls.</p>

<p>The context injection hook injects repository metadata whenever Claude uses GitHub MCP tools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">GITHUB_BASE_CONTEXT</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">CONTEXT: This repo</span><span class="sh">'</span><span class="s">s GitHub owner is </span><span class="sh">'</span><span class="s">AcmeCorp</span><span class="sh">'</span><span class="s">, NOT </span><span class="sh">'</span><span class="s">jdoe</span><span class="sh">'</span><span class="s">.
Always use owner=</span><span class="sh">'</span><span class="s">AcmeCorp</span><span class="sh">'</span><span class="s"> in GitHub API calls.
Repository name: webapp
Default branch: main</span><span class="sh">"""</span>

<span class="n">PROJECT_BOARD_CONTEXT</span> <span class="o">=</span> <span class="sh">"""</span><span class="s">
Project board ID: PVT_xxxxxxxxxxxxxxxxxxxx
Epic Priority field ID: PVTF_xxxxxxxxxxxxxxxxxxxxxxx
Status field ID: PVTSSF_xxxxxxxxxxxxxxxxxxxxxxx

Note: gh project item-edit silently fails for Number fields. Use GraphQL mutations instead.</span><span class="sh">"""</span>
</code></pre>

</div>



<p>The hook matcher uses regex to capture all GitHub MCP tools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"matcher"</span><span class="p">:</span><span class="w"> </span><span class="s2">"mcp__github__.*"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w"> </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"python3 .../context-injection.py"</span><span class="p">}]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>Project-related tools receive additional context about board IDs.</p>

<h2>
  
  
  Hook 3: Workflow Rule Enforcement
</h2>

<p>Some workflow rules cannot be enforced by linters: TDD discipline, worktree usage, commit signing. This hook surfaces reminders at relevant moments without blocking operations:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">GPG_REMINDER</span> <span class="o">=</span> <span class="nc">WorkflowReminder</span><span class="p">(</span>
    <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">GPG REMINDER: Never use `--no-gpg-sign`. If GPG fails, check sandbox mode.</span><span class="sh">"</span>
<span class="p">)</span>

<span class="n">TDD_REMINDER</span> <span class="o">=</span> <span class="nc">WorkflowReminder</span><span class="p">(</span>
    <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">TDD REMINDER: Is there a failing test? Write the RED test first, </span><span class="sh">"</span>
    <span class="sh">"</span><span class="s">then write minimal code to make it GREEN.</span><span class="sh">"</span>
<span class="p">)</span>

<span class="n">WORKTREE_REMINDER</span> <span class="o">=</span> <span class="nc">WorkflowReminder</span><span class="p">(</span>
    <span class="n">message</span><span class="o">=</span><span class="sh">"</span><span class="s">WORKTREE REMINDER: You appear to be working directly on main. </span><span class="sh">"</span>
    <span class="sh">"</span><span class="s">Use `git worktree add .worktrees/issue-XX-description -b issue-XX-description`</span><span class="sh">"</span>
<span class="p">)</span>
</code></pre>

</div>



<p>Reminders are injected based on tool and file context:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">get_write_edit_reminders</span><span class="p">(</span><span class="n">file_path</span><span class="p">:</span> <span class="nb">str</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">]:</span>
    <span class="n">reminders</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="c1"># TDD reminder for Go production code
</span>    <span class="k">if</span> <span class="nf">is_go_production_code</span><span class="p">(</span><span class="n">file_path</span><span class="p">):</span>
        <span class="n">reminders</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">TDD_REMINDER</span><span class="p">.</span><span class="n">message</span><span class="p">)</span>

    <span class="c1"># Worktree reminder if editing production code outside worktree
</span>    <span class="k">if</span> <span class="ow">not</span> <span class="nf">is_in_worktree</span><span class="p">()</span> <span class="ow">and</span> <span class="nf">is_production_code_path</span><span class="p">(</span><span class="n">file_path</span><span class="p">):</span>
        <span class="n">reminders</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">WORKTREE_REMINDER</span><span class="p">.</span><span class="n">message</span><span class="p">)</span>

    <span class="k">return</span> <span class="n">reminders</span>
</code></pre>

</div>



<p>The worktree check uses an environment variable to detect the current directory:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">is_in_worktree</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="nb">bool</span><span class="p">:</span>
    <span class="n">cwd</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">PWD</span><span class="sh">"</span><span class="p">,</span> <span class="n">os</span><span class="p">.</span><span class="nf">getcwd</span><span class="p">())</span>
    <span class="k">return</span> <span class="sh">"</span><span class="s">.worktrees/</span><span class="sh">"</span> <span class="ow">in</span> <span class="n">cwd</span>
</code></pre>

</div>



<h2>
  
  
  Hook 4: Skill Suggestion Engine
</h2>

<p>Claude Code supports "skills" (reusable instruction sets for specific domains). The skill suggestion hook maps file patterns to relevant skills, prompting Claude to load domain-specific guidance before proceeding.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="n">SKILL_TRIGGERS</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="n">SkillTrigger</span><span class="p">]</span> <span class="o">=</span> <span class="p">[</span>
    <span class="c1"># Test files need test classification + TDD patterns
</span>    <span class="nc">SkillTrigger</span><span class="p">(</span>
        <span class="n">skills</span><span class="o">=</span><span class="p">(</span><span class="sh">"</span><span class="s">classifying-test-sizes</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">developing-with-tdd</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">path_contains</span><span class="o">=</span><span class="sh">"</span><span class="s">_test.go</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
    <span class="c1"># Temporal workflows need TDD + Temporal + Go monorepo skills
</span>    <span class="nc">SkillTrigger</span><span class="p">(</span>
        <span class="n">skills</span><span class="o">=</span><span class="p">(</span><span class="sh">"</span><span class="s">developing-with-tdd</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">developing-with-temporal</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">working-with-go-monorepo</span><span class="sh">"</span><span class="p">),</span>
        <span class="n">path_contains</span><span class="o">=</span><span class="sh">"</span><span class="s">apps/workers/</span><span class="sh">"</span><span class="p">,</span>
        <span class="n">extension</span><span class="o">=</span><span class="sh">"</span><span class="s">.go</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
    <span class="c1"># Database migrations
</span>    <span class="nc">SkillTrigger</span><span class="p">(</span>
        <span class="n">skills</span><span class="o">=</span><span class="p">(</span><span class="sh">"</span><span class="s">working-with-atlas-migrations</span><span class="sh">"</span><span class="p">,),</span>
        <span class="n">path_contains</span><span class="o">=</span><span class="sh">"</span><span class="s">db/migrations/</span><span class="sh">"</span><span class="p">,</span>
    <span class="p">),</span>
<span class="p">]</span>
</code></pre>

</div>



<p>The trigger list uses first-match-wins ordering. Specific patterns (test files, migrations) appear before generic patterns (any Go file in apps/). This ensures a <code>_test.go</code> file triggers test-specific skills rather than generic Go guidance.</p>

<p>The output reminds Claude to invoke the relevant skills:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">if</span> <span class="n">skills</span><span class="p">:</span>
    <span class="n">skill_list</span> <span class="o">=</span> <span class="sh">"</span><span class="s">, </span><span class="sh">"</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="sa">f</span><span class="sh">"'</span><span class="si">{</span><span class="n">s</span><span class="si">}</span><span class="sh">'"</span> <span class="k">for</span> <span class="n">s</span> <span class="ow">in</span> <span class="n">skills</span><span class="p">)</span>
    <span class="n">output</span> <span class="o">=</span> <span class="p">{</span>
        <span class="sh">"</span><span class="s">hookSpecificOutput</span><span class="sh">"</span><span class="p">:</span> <span class="p">{</span>
            <span class="sh">"</span><span class="s">additionalContext</span><span class="sh">"</span><span class="p">:</span> <span class="p">(</span>
                <span class="sa">f</span><span class="sh">"</span><span class="s">SKILL REMINDER: Before proceeding, invoke these skills: </span><span class="si">{</span><span class="n">skill_list</span><span class="si">}</span><span class="s">. </span><span class="sh">"</span>
                <span class="sa">f</span><span class="sh">"</span><span class="s">They contain patterns, gotchas, and best practices for this file type.</span><span class="sh">"</span>
            <span class="p">)</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre>

</div>



<h2>
  
  
  Hook Configuration
</h2>

<p>The settings.json file orchestrates which hooks run for which tools:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"PreToolUse"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"matcher"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Read|Write|Edit"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w"> </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"python3 .../suggest-skills.py"</span><span class="p">}]</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"matcher"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Bash"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w"> </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"python3 .../safety-guards.py"</span><span class="p">},</span><span class="w">
          </span><span class="p">{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w"> </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"python3 .../workflow-rules.py"</span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"matcher"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Write|Edit"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w"> </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"python3 .../workflow-rules.py"</span><span class="p">}]</span><span class="w">
      </span><span class="p">},</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"matcher"</span><span class="p">:</span><span class="w"> </span><span class="s2">"mcp__github__.*"</span><span class="p">,</span><span class="w">
        </span><span class="nl">"hooks"</span><span class="p">:</span><span class="w"> </span><span class="p">[{</span><span class="nl">"type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"command"</span><span class="p">,</span><span class="w"> </span><span class="nl">"command"</span><span class="p">:</span><span class="w"> </span><span class="s2">"python3 .../context-injection.py"</span><span class="p">}]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">]</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>The <code>matcher</code> field accepts regex patterns. Bash commands receive both safety guards and workflow rules. Write/Edit operations trigger workflow rules and skill suggestions. GitHub MCP tools receive context injection.</p>

<h2>
  
  
  Observations
</h2>

<p>In my workflow, the hooks have changed how Claude interacts with the repository:</p>

<ul>
<li>Claude no longer attempts <code>git reset --hard</code>. The safety hook blocks it and suggests alternatives.</li>
<li>GitHub API calls default to the correct owner. Before the context hook, I corrected the owner manually in roughly one of five API calls.</li>
<li>The TDD reminder appears on every Go file edit. I cannot quantify compliance, but the reminder makes skipping the red-test step feel deliberate rather than accidental.</li>
<li>Skill loading happens automatically when I edit workflow files, instead of requiring me to remember to invoke them.</li>
</ul>

<h2>
  
  
  Implementation Considerations
</h2>

<p>Hooks cannot modify tool inputs, only allow, block, or inject context. Complex validation requiring multiple file reads will slow down every tool call. Hooks see individual tool calls, not conversation context.</p>

<p>Error handling defaults to permissive: JSON parse failures result in exit code 0 (allow) rather than blocking operations. This prevents a malformed hook from breaking the development workflow entirely.</p>

<p>The complete implementation consists of four Python files totaling under 400 lines. Each hook has a single responsibility, making them straightforward to test and modify. Adding new concerns means adding new hooks, keeping each hook focused.</p>

