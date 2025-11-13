---
Title: Setting up CI/CD with GitHub Actions
Description: 
Author: Steven Hur
Date: 2025-11-13T21:09:56.000Z
Robots: noindex,nofollow
Template: index
---
<p>Welcome to my reflection on CI/CD experience, where the core objective was to move beyond local testing and integrate a CI pipeline into my project using GitHub Actions. This lab was an interesting experience understanding how to manage project complexity and a fundamental concept in collaborative software development.</p>

<p><strong>1. The <code>ci.yml</code> Blueprint</strong><br>
The first step was setting up the automation pipeline. Since my project is based on <code>Python</code> and uses <code>Pytest</code> for testing, I configured a workflow to automatically run tests whenever code was pushed or a <code>Pull Request (PR)</code> was opened.</p>

<p>-The GitHub Actions Workflow-<br>
The configuration was defined in the <code>.github/workflows/ci.yml</code> file, ensuring a consistent testing environment.</p>

<p>What does the YAML file do?<br>
This workflow automates four key steps:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>1. Checkout the code
2. Set up the required Python environment
3. Install project dependencies
4. Execute all unit tests using pytest
</code></pre>

</div>



<p>This process guarantees that no new changes will break the existing functionality before they are merged.</p>

<p><strong>2. Mastering the CI Cycle: Pass, Fail, Pass</strong><br>
The most important part of setting up CI was running the full test cycle. This proved that the CI system itself works as expected.</p>

<ul>
<li>Initial Pass (Success): I created a PR, and the CI successfully ran all existing tests, resulting in a green checkmark.</li>
<li>Intentional Fail (Failure): I then committed a change that caused a test to fail. The CI automatically re-ran and immediately reported a red X.</li>
<li>Final Pass (Recovery): After reverting the breaking change, the CI successfully ran again, showing a green checkmark.</li>
</ul>

<p>This cycle confirmed that the CI is functioning properly.</p>

<p><strong>3. The Cross-Project Collaboration Challenge</strong><br>
Afterwards, I had to find a partner and contribute a new test case to their repository. My partner's project was also Python-based, utilizing <code>Pytest</code> and a similar <code>src/tests</code> directory structure.</p>

<p>The experience of writing tests for external code highlighted the need for strong documentation and well-encapsulated functions. I chose to write tests for <code>ArgParser</code> class, focusing on its custom logic for loading <code>TOML</code> configuration files.</p>

<p>The main challenge I faced was an unexpected <code>ModuleNotFoundError</code> when trying to run tests locally, even though the file structure was clear. This was because <code>Python</code>'s default import path does not automatically include the sibling <code>src/</code> directory.</p>

<p>The solution required manually inserting the <code>src</code> folder's absolute path into the system's path configuration within the test file.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># The Fix for ModuleNotFoundError:
import sys
import os
src_dir = os.path.abspath(os.path.join(os.path.dirname(__file__), '..', 'src'))
sys.path.insert(0, src_dir)
from arg_parser import ArgParser
</code></pre>

</div>



<p>This experience was a reminder that while <code>CI</code> automates testing, basic project setup is essential for any developer joining the project.</p>

<p>Having successfully set up CI and used it in a real-world scenario, I now strongly believe that CI is inevitable.</p>

<p>Before this lab, running tests felt like an optional. However, proper testing removes human error and ensures that every code change, no matter how small, is immediately validated against the entire suite of existing tests.</p>

<p>CI is not just about testing. It's about reducing integration friction and providing immediate feedback which is key to maintaining a stable and scalable codebase</p>

