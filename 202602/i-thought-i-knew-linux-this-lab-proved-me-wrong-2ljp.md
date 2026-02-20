---
Title: I Thought I Knew Linux. This Lab Proved Me Wrong.
Description: 
Author: Yaa-K
Date: 2026-02-20T21:57:19.000Z
Robots: noindex,nofollow
Template: index
---
<p>I've been using Linux for a while. Commands like <code>useradd</code>, <code>chmod</code>, <code>grep</code> — I knew them. I could navigate the terminal without panicking. So when I started my first assignment in the ParoCyber DevSecOps Bootcamp, I figured it would be straightforward.</p>

<p>It wasn't. And I mean that in the best way possible.</p>




<h2>
  
  
  What the Assignment Was About
</h2>

<p>The lab had two scenarios. The first was a password state investigation — find users on a system with no passwords set, understand <em>where</em> Linux stores that information, and remediate it. The second was a full onboarding simulation: create users, assign them to department groups, configure a CI/CD service account, manage sudo access, and safely offboard a user.</p>

<p>On paper, it sounds like basic sysadmin work. In practice, it forced me to think like a security engineer — and that changed everything.</p>




<h2>
  
  
  The Moment That Reframed Everything
</h2>

<p>In Scenario 1, the task said: <em>"You are not told where Linux stores password state. Finding it is part of the task."</em></p>

<p>I knew about <code>/etc/passwd</code>. Everyone who has touched Linux knows <code>/etc/passwd</code>. But when I ran <code>cat /etc/passwd</code>, the password field just showed <code>x</code>. A placeholder. I had seen that a hundred times and never thought twice about it.</p>

<p>That <code>x</code> means the actual password data lives somewhere else — <code>/etc/shadow</code>. And when I inspected it, I saw something that stopped me:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>audit_test:!:...
</code></pre>

</div>



<p>That <code>!</code> in the second field. One character. That single character is the difference between a secured account and a vulnerability sitting open on your system. I had been using Linux without ever knowing that file existed, let alone what it meant.</p>

<p>Then I tested whether a normal user could read <code>/etc/shadow</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">cat</span> /etc/shadow
<span class="c"># Permission denied</span>
</code></pre>

</div>



<p>And that's when it clicked. The fact that <code>/etc/passwd</code> is world-readable but <code>/etc/shadow</code> is root-only isn't an accident — it's a deliberate security design. Two files, one visible to everyone, one locked down. I had been looking at half the picture this whole time.</p>




<h2>
  
  
  Scenario 2: Access Control Is a People Problem Too
</h2>

<p>The second scenario was about onboarding eight new staff members across four departments — dev, security, QA, and ops. Each team gets its own group. Each group gets access to what it needs and nothing more.</p>

<p>What struck me here wasn't the commands. It was the <em>reasoning</em> behind them.</p>

<p>When I created the <code>ci_runner</code> service account with a non-login shell:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>useradd <span class="nt">-r</span> <span class="nt">-s</span> /usr/sbin/nologin ci_runner
</code></pre>

</div>



<p>The trainer's question wasn't "did you run the command" — it was "do you know <em>why</em> this matters?" A service account with an interactive shell is an open door. Automated pipelines don't need to log in. Humans shouldn't be logging in as them either. That shell setting is a security decision, not a configuration detail.</p>

<p>Same with removing a user at the end. I used:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">sudo </span>userdel yaa
</code></pre>

</div>



<p>Not <code>userdel -r</code>. The <code>-r</code> flag would have deleted the home directory too. But in a real environment, that directory might contain evidence — scripts, logs, files that matter in an investigation. Deleting it immediately could mean destroying forensic data. That's not a Linux lesson. That's a security operations lesson.</p>




<h2>
  
  
  What I'm Taking Away
</h2>

<p>I came into this thinking DevSecOps was Linux plus some security tools on top. What I'm leaving with is a different understanding — security isn't a layer you add. It's a lens you apply to every decision, including the ones that look purely technical.</p>

<p>The commands I ran in this lab weren't new. The <em>questions</em> behind them were.</p>

<p>Why does this file have these permissions? Why does this account need this shell? What happens to this data when this user is gone? Those questions are what separate someone who <em>uses</em> Linux from someone who <em>secures</em> it.</p>

<p>This is assignment one. I'm already thinking differently. Looking forward to what's next.</p>

