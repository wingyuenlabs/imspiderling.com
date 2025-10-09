---
Title: The Data Science Tech Stack you must master in 2025
Description: 
Author: SHH
Date: 2025-10-09T22:08:07.000Z
Robots: noindex,nofollow
Template: index
---
<p>It is the year 2025, everybody and their grandma have asked ChatGPT about the meaning of life. While we cannot be sure whether it generated a hallucinated answer, we do know that LLMs are developed using <a href="https://python.org" rel="noopener noreferrer">Python</a>. Data scientists today are expected to work with AI/ML models and therefore Python (see below), effectively settling the age-old "Python vs. R" debate.</p>

<h2>
  
  
  Package and environment manager
</h2>

<p>To keep your projects tidy and make your code reproducible on any machine, you need to keep track of the version numbers of the project's dependencies. Package and environment managers help you with that. There have been many package managers (conda, pip etc.) and perhaps even more virtual environment managers (virtualenv, pyenv, pipenv etc.). The general consensus nowadays is that you should just use <a href="https://github.com/astral-sh/uv" rel="noopener noreferrer">uv</a> as it combines both functions while being faster than the other solutions.</p>

<h2>
  
  
  Development environment
</h2>

<p>Jupyter notebooks are great to get started: easy to setup and run interactively (cell by cell). However, in the real world you will be expected to ship code to production in the form of scripts and apps, that is, not notebooks.</p>

<p>You could copy-paste code from a Jupyter notebook to a text editor, but there's a more convenient way: integrated development environments (IDE) like <a href="https://code.visualstudio.com" rel="noopener noreferrer">VS Code</a> and <a href="https://cursor.com" rel="noopener noreferrer">Cursor</a>. Not only do they combine file explorer, text editor and terminal in one application, but there are also many extensions which will make you life easier, e.g., code formatter, linter etc. Plus, you don't need to give up on Jupyter notebooks, you can create and run them inside of VS Code/Cursor. Lastly, they allow you to take advantage of AI features like tab/auto completions, making you even more productive.</p>

<h4>
  
  
  How to get started with VS Code and uv
</h4>

<ol>
<li>Download and install VS Code: <a href="https://code.visualstudio.com/Download" rel="noopener noreferrer">https://code.visualstudio.com/Download</a>
</li>
<li>Install <em>uv</em> by executing the following command in your VS Code terminal:

<ul>
<li>(Linux/MacOS) <code>curl -LsSf https://astral.sh/uv/install.sh | sh</code> </li>
<li>(Windows) <code>powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"</code>
</li>
</ul>
</li>
<li>Install <em>Python</em>: <code>uv python install</code>
</li>
<li>Install a package: <code>uv install pandas</code>
</li>
<li>Create a project: <code>uv init your-project-name</code>
</li>
<li>Add a dependency to your project lock file: <code>uv add pandas</code>
</li>
<li>Create a script: example.py <code>print('Hello world!')</code>
</li>
<li>Run a script: <code>uv run example.py</code>
</li>
</ol>

<h2>
  
  
  Skills
</h2>

<p>I have analyzed postings of <a href="https://mljobs.io/data-science-jobs" rel="noopener noreferrer">data science jobs</a> from AI frontier labs, like OpenAI, to identify those skills that are most likely <strong>future-proof</strong>.</p>

<h4>
  
  
  Programming languages
</h4>

<p>Python and SQL are listed as required qualifications in all listings. R was not mentioned even once...</p>

<h4>
  
  
  General skills
</h4>

<ul>
<li>design statistical experiments</li>
<li>conduct A/B tests</li>
<li>define and operationalize metrics</li>
<li>visualize results, dashboarding</li>
<li>communicate with stakeholders</li>
<li>prototyping</li>
<li>run simulations</li>
<li>version control (git)</li>
</ul>

<h4>
  
  
  Frameworks, modules and tools
</h4>

<p>A list of popular, but not necessarily required experiences:</p>

<ul>
<li>pandas, NumPy, scikit-learn, flask</li>
<li>seaborn/Matplotlib, Tableau/Power BI</li>
<li>GitHub</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>AI will certainly change how data scientists will work going forward. However, I believe that LLMs will not replace them. Instead, there will be a growing need for capable data scientists that are able to uncover the failure modes of today's AIs and design better systems.</p>

