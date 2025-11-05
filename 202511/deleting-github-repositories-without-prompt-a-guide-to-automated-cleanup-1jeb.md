---
Title: Deleting GitHub Repositories Without Prompt: A Guide to Automated Cleanup
Description: 
Author: Picoable
Date: 2025-11-05T21:59:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Managing numerous GitHub repositories can sometimes be a chore, especially when you need to clean up a large number of old projects, test repositories, or deprecated forks. Manually deleting each repository through the web interface or even interactively via the GitHub CLI (<code>gh</code>) can be time-consuming and repetitive.</p>

<p>This guide will show you how to automate the process of deleting multiple GitHub repositories using a simple bash script combined with the powerful GitHub CLI, all without being prompted for confirmation for each deletion.</p>

<p><strong>⚠️ WARNING: This script performs destructive actions. Repositories deleted are gone forever. Always double-check your list of repositories before running any deletion script. It is highly recommended to test this process on dummy repositories first.</strong></p>

<h3>
  
  
  Prerequisites
</h3>

<p>Before you begin, ensure you have the following installed and configured:</p>

<ol>
<li> <strong>GitHub CLI (<code>gh</code>):</strong> This is the official command-line tool for GitHub. If you don't have it, install it from <a href="https://cli.github.com/" rel="noopener noreferrer">cli.github.com</a>.</li>
<li> <strong>Authenticated <code>gh</code> CLI:</strong> Make sure you are logged in to your GitHub account via the CLI. You can do this by running <code>gh auth login</code>.</li>
</ol>

<h3>
  
  
  Step 1: Generate a List of Repositories to Delete
</h3>

<p>The first step is to get a list of the repositories you intend to delete. We'll use <code>gh repo list</code> to fetch repository names and pipe the output to a file.</p>

<p>The <code>gh repo list</code> command is highly flexible. You can filter by owner, visibility (public, private, internal), and more. For this example, let's assume you want to delete all public repositories owned by your current user.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>gh repo list <span class="nt">--public</span> <span class="nt">--json</span> name <span class="nt">--jq</span> <span class="s1">'.[].name'</span> <span class="nt">-L</span> 9999 <span class="o">&gt;</span> deletrepo.txt
</code></pre>

</div>



<p>Let's break down this command:</p>

<ul>
<li>  <code>gh repo list</code>: Lists repositories.</li>
<li>  <code>--public</code>: Filters to only show public repositories. You could use <code>--private</code> or <code>--all</code> as needed.</li>
<li>  <code>--json name</code>: Outputs the repository data in JSON format, specifically extracting only the <code>name</code> field.</li>
<li>  <code>--jq '.[].name'</code>: Uses <code>jq</code> (a lightweight and flexible command-line JSON processor) to extract just the <code>name</code> value from each JSON object in the array. This ensures <code>deletrepo.txt</code> contains one repository name per line.</li>
<li>  <code>-L 9999</code>: Sets the limit to list up to 9999 repositories. Adjust this if you have more.</li>
<li>  <code>&gt; deletrepo.txt</code>: Redirects the output to a file named <code>deletrepo.txt</code>.</li>
</ul>

<p><strong>Crucial Step: Review <code>deletrepo.txt</code></strong></p>

<p>After running the above command, <strong>open <code>deletrepo.txt</code> in a text editor and carefully review its contents.</strong> Ensure that <em>only</em> the repositories you genuinely want to delete are listed. Remove any entries you wish to keep. This is your last chance to prevent accidental deletion!</p>

<p>Example <code>deletrepo.txt</code>:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>my-old-project
test-repo-1
another-unused-app
</code></pre>

</div>



<h3>
  
  
  Step 2: Create the Deletion Script (<code>delete_repos.sh</code>)
</h3>

<p>Now, we'll create a bash script that reads each repository name from <code>deletrepo.txt</code> and uses <code>gh repo delete</code> to remove it. The key to automating without prompts is the <code>--confirm</code> flag.</p>

<p>Create a file named <code>delete_repos.sh</code> and add the following content:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c">#!/bin/bash</span>

<span class="c"># This script reads a list of repository names from deletrepo.txt</span>
<span class="c"># and deletes each one using the GitHub CLI.</span>
<span class="c"># USE WITH EXTREME CAUTION! Repositories deleted are gone forever.</span>

<span class="nv">REPO_LIST_FILE</span><span class="o">=</span><span class="s2">"deletrepo.txt"</span>

<span class="c"># Check if the repository list file exists</span>
<span class="k">if</span> <span class="o">[</span> <span class="o">!</span> <span class="nt">-f</span> <span class="s2">"</span><span class="nv">$REPO_LIST_FILE</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
  </span><span class="nb">echo</span> <span class="s2">"Error: Repository list file '</span><span class="nv">$REPO_LIST_FILE</span><span class="s2">' not found."</span>
  <span class="nb">echo</span> <span class="s2">"Please create it using 'gh repo list --public --json name --jq '.[].name' -L 9999 &gt; </span><span class="nv">$REPO_LIST_FILE</span><span class="s2">' and review its contents."</span>
  <span class="nb">exit </span>1
<span class="k">fi

</span><span class="nb">echo</span> <span class="s2">"Reading repositories from </span><span class="nv">$REPO_LIST_FILE</span><span class="s2">..."</span>
<span class="nb">echo</span> <span class="s2">"Starting deletion process. This is irreversible!"</span>
<span class="nb">echo</span> <span class="s2">"-------------------------------------------------"</span>

<span class="c"># Read each line (repository name) from the file</span>
<span class="k">while </span><span class="nv">IFS</span><span class="o">=</span> <span class="nb">read</span> <span class="nt">-r</span> repo_name<span class="p">;</span> <span class="k">do</span>
  <span class="c"># Skip empty lines</span>
  <span class="k">if</span> <span class="o">[</span> <span class="nt">-z</span> <span class="s2">"</span><span class="nv">$repo_name</span><span class="s2">"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
    continue
  fi

  </span><span class="nb">echo</span> <span class="s2">"Attempting to delete repository: </span><span class="nv">$repo_name</span><span class="s2">"</span>
  <span class="c"># Use gh repo delete with --confirm to bypass the interactive prompt</span>
  gh repo delete <span class="s2">"</span><span class="nv">$repo_name</span><span class="s2">"</span> <span class="nt">--confirm</span>

  <span class="k">if</span> <span class="o">[</span> <span class="nv">$?</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">echo</span> <span class="s2">"✅ Successfully deleted </span><span class="nv">$repo_name</span><span class="s2">"</span>
  <span class="k">else
    </span><span class="nb">echo</span> <span class="s2">"❌ Failed to delete </span><span class="nv">$repo_name</span><span class="s2">. Check gh CLI output for details."</span>
  <span class="k">fi
  </span><span class="nb">echo</span> <span class="s2">"-------------------------------------------------"</span>
<span class="k">done</span> &lt; <span class="s2">"</span><span class="nv">$REPO_LIST_FILE</span><span class="s2">"</span>

<span class="nb">echo</span> <span class="s2">"Deletion process complete."</span>
<span class="nb">echo</span> <span class="s2">"It is recommended to run 'gh repo list' to verify remaining repositories."</span>
</code></pre>

</div>



<h3>
  
  
  Step 3: Make the Script Executable and Run It
</h3>

<ol>
<li>
<p><strong>Make the script executable:</strong><br>
</p>
<pre class="highlight shell"><code><span class="nb">chmod</span> +x delete_repos.sh
</code></pre>

</li>
<li>
<p><strong>Run the script:</strong><br>
</p>
<pre class="highlight shell"><code>./delete_repos.sh
</code></pre>

</li>
</ol>

<p>The script will iterate through <code>deletrepo.txt</code>, attempting to delete each repository. You will see output for each deletion attempt, indicating success or failure.</p>

<h3>
  
  
  Safety Measures and Best Practices
</h3>

<ul>
<li>  <strong>Always Review <code>deletrepo.txt</code>:</strong> This cannot be stressed enough. A mistake here can lead to irreversible data loss.</li>
<li>  <strong>Test on Dummy Repositories:</strong> Create a few throwaway repositories and test the entire process before using it on real data.</li>
<li>  <strong>Backup Important Data:</strong> If there's any chance a repository might contain valuable information, ensure you have a backup before deleting it.</li>
<li>  <strong>Use Specific Filters:</strong> When generating <code>deletrepo.txt</code>, be as specific as possible with <code>gh repo list</code> filters (e.g., <code>--owner YOUR_USERNAME</code>, <code>--topic old-projects</code>) to minimize the risk of including unintended repositories.</li>
<li>  <strong>Consider Archiving:</strong> If you're unsure about permanent deletion, GitHub allows archiving repositories, which makes them read-only but preserves their history.</li>
</ul>

<h3>
  
  
  Conclusion
</h3>

<p>Automating GitHub repository deletion can be a huge time-saver for developers and teams managing many projects. By combining the power of the GitHub CLI with a simple bash script, you can efficiently clean up your GitHub presence. Just remember to proceed with extreme caution and verify your target list thoroughly!</p>

