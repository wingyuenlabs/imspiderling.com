---
Title: Best Code Review Tools for Python in 2026 - Linters, SAST, and AI
Description: 
Author: Rahul Singh
Date: 2026-03-12T21:43:53.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Why Python needs specialized code review tools
</h2>

<p>Python is the most popular programming language in the world in 2026, powering everything from machine learning pipelines and data science notebooks to Django web applications and FastAPI microservices. But Python's flexibility is also its biggest liability when it comes to code quality. The features that make Python productive for individual developers - dynamic typing, duck typing, implicit conversions, mutable defaults - create entire categories of bugs that do not exist in statically typed languages.</p>

<p>Consider a simple example that ships to production more often than anyone would like to admit:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">process_users</span><span class="p">(</span><span class="n">users</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">dict</span><span class="p">],</span> <span class="n">active_only</span><span class="p">:</span> <span class="nb">bool</span> <span class="o">=</span> <span class="bp">True</span><span class="p">):</span>
    <span class="n">results</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">user</span> <span class="ow">in</span> <span class="n">users</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">active_only</span> <span class="ow">and</span> <span class="n">user</span><span class="p">[</span><span class="sh">"</span><span class="s">status</span><span class="sh">"</span><span class="p">]</span> <span class="o">==</span> <span class="sh">"</span><span class="s">active</span><span class="sh">"</span><span class="p">:</span>
            <span class="n">results</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span>
                <span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">:</span> <span class="n">user</span><span class="p">[</span><span class="sh">"</span><span class="s">name</span><span class="sh">"</span><span class="p">],</span>
                <span class="sh">"</span><span class="s">email</span><span class="sh">"</span><span class="p">:</span> <span class="n">user</span><span class="p">[</span><span class="sh">"</span><span class="s">email</span><span class="sh">"</span><span class="p">],</span>
                <span class="sh">"</span><span class="s">score</span><span class="sh">"</span><span class="p">:</span> <span class="n">user</span><span class="p">[</span><span class="sh">"</span><span class="s">points</span><span class="sh">"</span><span class="p">]</span> <span class="o">/</span> <span class="n">user</span><span class="p">[</span><span class="sh">"</span><span class="s">total_points</span><span class="sh">"</span><span class="p">]</span>  <span class="c1"># ZeroDivisionError
</span>            <span class="p">})</span>
    <span class="k">return</span> <span class="n">results</span>
</code></pre>

</div>



<p>A generic code review tool might flag the missing docstring or suggest a list comprehension. A Python-specific tool will catch the <code>ZeroDivisionError</code> when <code>total_points</code> is zero, the <code>KeyError</code> when any of those dictionary keys are missing, and the fact that <code>user["status"]</code> could be <code>None</code> in a dynamically typed context. That is the difference between a tool that happens to support Python and a tool that understands Python.</p>

<h3>
  
  
  Dynamic typing creates unique review challenges
</h3>

<p>In Java or Go, the compiler catches type mismatches before code even runs. In Python, a function that expects a <code>list[str]</code> will happily accept a <code>list[int]</code> at runtime - and may even work correctly in some cases while silently producing wrong results in others. Type annotations help, but they are not enforced by the interpreter. You need a separate type checker (mypy, pyright) to validate them, and those type checkers need to be integrated into your review workflow to provide value.</p>

<p>The problem goes deeper than basic type mismatches. Python's protocol-based type system means that objects with the right method signatures are treated as compatible, even if their semantics differ. A <code>StringIO</code> object and a file handle both support <code>.read()</code>, but substituting one for the other in a function that calls <code>.seek()</code> later will fail silently until it does not. Tools that understand Python's type system at a deep level catch these issues before they reach production.</p>

<h3>
  
  
  Web framework security requires Python-aware analysis
</h3>

<p>Django and Flask are two of the most deployed web frameworks in production. Both have extensive security features - Django's ORM prevents SQL injection by default, its template engine auto-escapes HTML, and its middleware handles CSRF protection. But these protections only work if developers use them correctly.</p>

<p>Here is a pattern that passes generic code review tools without a single warning:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Django view - looks fine to a generic tool
</span><span class="k">def</span> <span class="nf">search_users</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="n">query</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">GET</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">q</span><span class="sh">"</span><span class="p">,</span> <span class="sh">""</span><span class="p">)</span>
    <span class="c1"># SQL injection: raw query with string formatting
</span>    <span class="n">users</span> <span class="o">=</span> <span class="n">User</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">raw</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">SELECT * FROM users WHERE name LIKE </span><span class="sh">'</span><span class="s">%</span><span class="si">{</span><span class="n">query</span><span class="si">}</span><span class="s">%</span><span class="sh">'"</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">render</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="sh">"</span><span class="s">results.html</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">users</span><span class="sh">"</span><span class="p">:</span> <span class="n">users</span><span class="p">})</span>
</code></pre>

</div>



<p>A generic static analysis tool sees a function that reads a query parameter and returns a rendered template. A Python-aware security scanner like Bandit or Semgrep with Django rules immediately flags the raw SQL with untrusted user input as a critical SQL injection vulnerability. It knows that <code>request.GET.get()</code> is a taint source and <code>objects.raw()</code> is a dangerous sink, and it traces the data flow between them.</p>

<p>Flask introduces different challenges. Unlike Django, Flask does not enable CSRF protection by default, does not auto-escape Jinja2 templates in all contexts, and does not provide a built-in ORM that prevents SQL injection. Python-specific security tools know which Flask extensions are installed and whether they are configured correctly - something a language-agnostic tool cannot evaluate.</p>

<h3>
  
  
  The package ecosystem introduces supply chain risks
</h3>

<p>Python's pip ecosystem contains over 500,000 packages on PyPI, and Python projects tend to have deep dependency trees. A typical Django project might pull in 80-120 transitive dependencies through its <code>requirements.txt</code> or <code>pyproject.toml</code>. Each of those dependencies is a potential attack vector - and unlike npm, pip does not have built-in audit capabilities.</p>

<p>Tools like Snyk and Semgrep scan your dependency manifest for known vulnerabilities, but the Python-specific challenge goes further. Python packages can execute arbitrary code during installation through <code>setup.py</code>, and typosquatting attacks on PyPI have been documented extensively. A comprehensive Python code review setup needs to cover not just your code, but the code you are importing.</p>

<h2>
  
  
  Categories of Python code review tools
</h2>

<p>Before diving into specific tools, it helps to understand the different categories and how they complement each other. No single tool covers every dimension of Python code quality. The best setups layer tools from multiple categories.</p>

<h3>
  
  
  Linters and formatters
</h3>

<p>These tools enforce coding standards, catch common errors, and ensure consistent style. For Python, this category includes Ruff (the modern standard), Pylint (the deep analyzer), flake8 (the legacy standard), Black (opinionated formatter), and isort (import sorting). Ruff has increasingly consolidated this category by reimplementing the rules of multiple tools in a single, extremely fast binary.</p>

<h3>
  
  
  Type checkers
</h3>

<p>Python's type annotation system (PEP 484 and later) is purely advisory unless you run a type checker. Mypy (the original, from Dropbox) and Pyright (from Microsoft, powers VS Code's Pylance) are the two main options. They catch type mismatches, missing return types, incompatible overrides, and incorrect generic usage.</p>

<h3>
  
  
  Security scanners (SAST)
</h3>

<p>Static Application Security Testing tools analyze code for vulnerabilities without executing it. For Python, this includes Bandit (Python-specific), Semgrep (multi-language with strong Python rules), Snyk Code (cross-file dataflow analysis), and SonarQube (broad coverage). Django and Flask projects benefit enormously from security scanners that understand framework-specific vulnerability patterns.</p>

<h3>
  
  
  AI-powered code reviewers
</h3>

<p>These tools use large language models to understand code semantics and provide review feedback that goes beyond rule matching. Sourcery (Python-first), CodeRabbit (multi-language with strong Python support), DeepSource (AI-enhanced static analysis), and Qodo (test generation and review) fall in this category.</p>

<h3>
  
  
  Full code quality platforms
</h3>

<p>Platforms like SonarQube, Codacy, and DeepSource combine static analysis, security scanning, code coverage tracking, and sometimes AI review into a single tool. They provide dashboards, trend tracking, quality gates, and CI/CD integration across multiple dimensions.</p>

<h2>
  
  
  Quick comparison: Python code review tools at a glance
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Tool</th>
<th>Category</th>
<th>Python-Specific</th>
<th>Free Tier</th>
<th>Price</th>
<th>Best For</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ruff</td>
<td>Linter + formatter</td>
<td>Yes (Python-only)</td>
<td>OSS</td>
<td>Free</td>
<td>Fast linting, replacing flake8/isort/Black</td>
</tr>
<tr>
<td>Pylint</td>
<td>Deep linter</td>
<td>Yes (Python-only)</td>
<td>OSS</td>
<td>Free</td>
<td>Deep analysis, custom plugins</td>
</tr>
<tr>
<td>mypy</td>
<td>Type checker</td>
<td>Yes (Python-only)</td>
<td>OSS</td>
<td>Free</td>
<td>Type safety in CI/CD</td>
</tr>
<tr>
<td>Pyright</td>
<td>Type checker</td>
<td>Yes (Python-only)</td>
<td>OSS</td>
<td>Free</td>
<td>IDE type checking (Pylance)</td>
</tr>
<tr>
<td>Bandit</td>
<td>Security scanner</td>
<td>Yes (Python-only)</td>
<td>OSS</td>
<td>Free</td>
<td>Python-specific vulnerability detection</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/sourcery">Sourcery</a></td>
<td>AI code quality</td>
<td>Yes (Python-first)</td>
<td>OSS only</td>
<td>$29/seat/mo</td>
<td>Python refactoring suggestions</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/coderabbit">CodeRabbit</a></td>
<td>AI PR review</td>
<td>Strong support</td>
<td>Unlimited</td>
<td>$24/user/mo</td>
<td>Overall AI review quality</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/deepsource">DeepSource</a></td>
<td>Code quality + AI</td>
<td>Strong support</td>
<td>Individual</td>
<td>$30/user/mo</td>
<td>Low false positive analysis</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/sonarqube">SonarQube</a></td>
<td>Static analysis</td>
<td>500+ Python rules</td>
<td>Community</td>
<td>~$13/mo</td>
<td>Enterprise rule depth</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/semgrep">Semgrep</a></td>
<td>Security scanning</td>
<td>Django/Flask rules</td>
<td>10 devs</td>
<td>$35/dev/mo</td>
<td>Custom security rules</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/snyk-code">Snyk Code</a></td>
<td>Security SAST</td>
<td>Cross-file Python</td>
<td>Limited</td>
<td>$25/dev/mo</td>
<td>Full-stack security</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/codacy">Codacy</a></td>
<td>Code quality</td>
<td>Multi-tool Python</td>
<td>Yes</td>
<td>$15/user/mo</td>
<td>All-in-one for small teams</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/pr-agent">PR-Agent</a></td>
<td>AI PR review</td>
<td>Good support</td>
<td>Self-hosted</td>
<td>$19/user/mo</td>
<td>Self-hosted AI review</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/qodo">Qodo</a></td>
<td>AI review + tests</td>
<td>Good support</td>
<td>Yes</td>
<td>$19/seat/mo</td>
<td>Python test generation</td>
</tr>
<tr>
<td><a href="https://dev.to/tool/qlty">Qlty</a></td>
<td>Code health</td>
<td>Multi-tool Python</td>
<td>CLI free</td>
<td>$15/dev/mo</td>
<td>Polyglot codebases</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Python-specific linters: Ruff, Pylint, and flake8
</h2>

<p>The linter is the foundation of any Python code review setup. It runs fastest, catches the most common issues, and integrates into every stage of the development workflow - from pre-commit hooks to CI pipelines to IDE extensions. Choosing the right linter matters because it determines the baseline quality of every line of code your team writes.</p>

<h3>
  
  
  Ruff - The modern Python linter
</h3>

<p>Ruff has fundamentally changed the Python linting landscape since its initial release. Written in Rust, it is 10 to 100 times faster than Pylint and flake8, which means it can lint an entire large codebase in the time those tools take to lint a single file. But speed is only part of the story.</p>

<p>Ruff reimplements rules from over a dozen Python tools - flake8, isort, pycodestyle, pyflakes, pydocstyle, pyupgrade, autoflake, and more - in a single binary with a single configuration file. This means you can replace your entire linting toolchain with one tool:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># pyproject.toml - Ruff replaces flake8, isort, pyupgrade, and more</span>
<span class="nn">[tool.ruff]</span>
<span class="py">target-version</span> <span class="p">=</span> <span class="s">"py312"</span>
<span class="py">line-length</span> <span class="p">=</span> <span class="mi">120</span>

<span class="nn">[tool.ruff.lint]</span>
<span class="py">select</span> <span class="p">=</span> <span class="p">[</span>
    <span class="s">"E"</span><span class="p">,</span>    <span class="c"># pycodestyle errors</span>
    <span class="s">"W"</span><span class="p">,</span>    <span class="c"># pycodestyle warnings</span>
    <span class="s">"F"</span><span class="p">,</span>    <span class="c"># pyflakes</span>
    <span class="s">"I"</span><span class="p">,</span>    <span class="c"># isort</span>
    <span class="s">"N"</span><span class="p">,</span>    <span class="c"># pep8-naming</span>
    <span class="s">"UP"</span><span class="p">,</span>   <span class="c"># pyupgrade</span>
    <span class="s">"B"</span><span class="p">,</span>    <span class="c"># flake8-bugbear</span>
    <span class="s">"S"</span><span class="p">,</span>    <span class="c"># flake8-bandit (security)</span>
    <span class="s">"A"</span><span class="p">,</span>    <span class="c"># flake8-builtins</span>
    <span class="s">"C4"</span><span class="p">,</span>   <span class="c"># flake8-comprehensions</span>
    <span class="s">"DTZ"</span><span class="p">,</span>  <span class="c"># flake8-datetimez</span>
    <span class="s">"DJ"</span><span class="p">,</span>   <span class="c"># flake8-django</span>
    <span class="s">"PT"</span><span class="p">,</span>   <span class="c"># flake8-pytest-style</span>
    <span class="s">"SIM"</span><span class="p">,</span>  <span class="c"># flake8-simplify</span>
    <span class="s">"RUF"</span><span class="p">,</span>  <span class="c"># Ruff-specific rules</span>
<span class="p">]</span>

<span class="nn">[tool.ruff.lint.per-file-ignores]</span>
<span class="py">"tests/**/*.py"</span> <span class="p">=</span> <span class="p">[</span><span class="s">"S101"</span><span class="p">]</span>  <span class="c"># Allow assert in tests</span>
<span class="py">"migrations/**/*.py"</span> <span class="p">=</span> <span class="p">[</span><span class="s">"E501"</span><span class="p">]</span>  <span class="c"># Allow long lines in migrations</span>

<span class="nn">[tool.ruff.format]</span>
<span class="py">quote-style</span> <span class="p">=</span> <span class="s">"double"</span>
<span class="py">indent-style</span> <span class="p">=</span> <span class="s">"space"</span>
</code></pre>

</div>



<p>That single configuration replaces what previously required separate config files for flake8, isort, Black, pyupgrade, and several flake8 plugins. Ruff covers over 800 lint rules as of early 2026 and adds new ones regularly.</p>

<p><strong>Where Ruff excels for Python projects:</strong></p>

<p>Ruff's <code>flake8-django</code> ruleset (<code>DJ</code>) catches Django-specific issues like missing <code>__str__</code> methods on models, incorrect <code>Meta</code> class options, and deprecated Django patterns. The <code>flake8-bandit</code> ruleset (<code>S</code>) covers common security issues like hardcoded passwords, use of <code>eval()</code>, and insecure hash algorithms. The <code>flake8-pytest-style</code> rules (<code>PT</code>) enforce consistent test patterns.</p>

<p>Ruff also handles formatting through <code>ruff format</code>, which is a near-drop-in replacement for Black. Running <code>ruff check --fix &amp;&amp; ruff format</code> in your pre-commit hook gives you linting, auto-fixing, import sorting, and formatting in under a second for most projects.</p>

<p><strong>Where Ruff falls short:</strong></p>

<p>Ruff is a syntactic linter. It analyzes code as text and AST nodes, but it does not build a semantic model of your program. This means it cannot perform the kind of deep type inference that Pylint does, it cannot follow imports across modules to detect circular dependencies, and it cannot evaluate custom plugin logic. For most teams, this trade-off is worth it - Ruff covers 80%+ of common linting needs at dramatically better speed. But teams that need deep analysis should consider layering Pylint on top.</p>

<h3>
  
  
  Pylint - The deep Python analyzer
</h3>

<p>Pylint has been the most thorough Python linter for over two decades, and it still performs analysis that no other tool replicates. Unlike Ruff, Pylint builds a full abstract syntax tree and performs type inference, which lets it catch issues like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">class</span> <span class="nc">UserService</span><span class="p">:</span>
    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">db</span><span class="p">):</span>
        <span class="n">self</span><span class="p">.</span><span class="n">db</span> <span class="o">=</span> <span class="n">db</span>

    <span class="k">def</span> <span class="nf">get_user</span><span class="p">(</span><span class="n">self</span><span class="p">,</span> <span class="n">user_id</span><span class="p">:</span> <span class="nb">int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
        <span class="n">user</span> <span class="o">=</span> <span class="n">self</span><span class="p">.</span><span class="n">db</span><span class="p">.</span><span class="nf">query</span><span class="p">(</span><span class="n">User</span><span class="p">).</span><span class="nf">filter_by</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="n">user_id</span><span class="p">).</span><span class="nf">first</span><span class="p">()</span>
        <span class="k">return</span> <span class="n">user</span><span class="p">.</span><span class="nf">to_dict</span><span class="p">()</span>  <span class="c1"># Pylint: "Instance of 'None' has no 'to_dict' member"
</span></code></pre>

</div>



<p>Pylint understands that <code>.first()</code> on a SQLAlchemy query returns <code>Optional[User]</code>, and that calling <code>.to_dict()</code> on <code>None</code> will raise an <code>AttributeError</code>. Ruff cannot catch this because it does not perform cross-expression type inference.</p>

<p>Pylint's plugin system is another differentiator. The <code>pylint-django</code> plugin understands Django's ORM, model fields, and view patterns. It knows that a <code>CharField</code> defined in a model will be available as a string attribute on instances, which prevents false positives about undefined attributes. The <code>pylint-celery</code> plugin catches common mistakes in async task definitions. Custom Pylint plugins can encode your team's specific patterns and architectural constraints.</p>

<p><strong>The speed problem:</strong></p>

<p>Pylint's thoroughness comes at a cost. On a 50,000-line codebase, Pylint might take 30-60 seconds to complete a full analysis, while Ruff finishes in under a second. This makes Pylint impractical for pre-commit hooks or rapid IDE feedback on large projects. The common solution is to run Ruff locally and in pre-commit hooks for fast feedback, then run Pylint in CI for the deep analysis that catches what Ruff misses.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .pre-commit-config.yaml - Fast local, deep in CI</span>
<span class="na">repos</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">repo</span><span class="pi">:</span> <span class="s">https://github.com/astral-sh/ruff-pre-commit</span>
    <span class="na">rev</span><span class="pi">:</span> <span class="s">v0.9.0</span>
    <span class="na">hooks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">ruff</span>
        <span class="na">args</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">--fix</span><span class="pi">]</span>
      <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">ruff-format</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/lint.yml - Deep analysis in CI</span>
<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">lint</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v5</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">python-version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">3.12"</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pip install pylint pylint-django</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pylint --load-plugins pylint_django src/</span>
</code></pre>

</div>



<h3>
  
  
  flake8 - The legacy standard
</h3>

<p>flake8 is worth mentioning because it remains widely deployed, but for new projects in 2026, there is little reason to choose it over Ruff. Ruff implements all of flake8's core rules plus the rules from most popular flake8 plugins, runs dramatically faster, and provides autofix for many issues that flake8 can only report.</p>

<p>Teams already invested in flake8 with custom plugins that have not been ported to Ruff may want to keep flake8 running alongside Ruff. But for the vast majority of Python projects, Ruff is the strictly better choice in this category.</p>

<h2>
  
  
  Type checking tools: mypy and Pyright
</h2>

<p>Type checking is the second layer of Python code review. Linters catch style issues and common bugs; type checkers catch an entire class of errors that dynamic typing makes possible. If your codebase uses type annotations - and most professional Python codebases in 2026 do - a type checker is not optional.</p>

<h3>
  
  
  mypy - The standard for CI/CD type checking
</h3>

<p>mypy was the first mainstream Python type checker, developed at Dropbox by Jukka Lehtosalo and later contributed to by Guido van Rossum himself. It remains the most widely integrated type checker in CI/CD pipelines and has the broadest ecosystem of third-party type stubs.</p>

<p>A properly configured mypy setup catches a remarkable number of bugs:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># mypy catches all of these
</span><span class="kn">from</span> <span class="n">typing</span> <span class="kn">import</span> <span class="n">Optional</span>

<span class="k">def</span> <span class="nf">calculate_discount</span><span class="p">(</span><span class="n">price</span><span class="p">:</span> <span class="nb">float</span><span class="p">,</span> <span class="n">discount</span><span class="p">:</span> <span class="n">Optional</span><span class="p">[</span><span class="nb">float</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">float</span><span class="p">:</span>
    <span class="k">return</span> <span class="n">price</span> <span class="o">*</span> <span class="p">(</span><span class="mi">1</span> <span class="o">-</span> <span class="n">discount</span><span class="p">)</span>  <span class="c1"># error: Unsupported operand types for * ("float" and "Optional[float]")
</span>
<span class="k">def</span> <span class="nf">process_items</span><span class="p">(</span><span class="n">items</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">]:</span>
    <span class="k">return</span> <span class="p">[</span><span class="n">item</span><span class="p">.</span><span class="n">upper</span> <span class="k">for</span> <span class="n">item</span> <span class="ow">in</span> <span class="n">items</span><span class="p">]</span>  <span class="c1"># error: "Callable[[], str]" has no attribute "__iter__"
</span>    <span class="c1"># (missing parentheses on .upper())
</span>
<span class="k">class</span> <span class="nc">Config</span><span class="p">:</span>
    <span class="n">debug</span><span class="p">:</span> <span class="nb">bool</span> <span class="o">=</span> <span class="bp">False</span>
    <span class="n">timeout</span><span class="p">:</span> <span class="nb">int</span> <span class="o">=</span> <span class="mi">30</span>

<span class="n">config</span> <span class="o">=</span> <span class="nc">Config</span><span class="p">()</span>
<span class="n">config</span><span class="p">.</span><span class="n">timeout</span> <span class="o">=</span> <span class="sh">"</span><span class="s">60</span><span class="sh">"</span>  <span class="c1"># error: Incompatible types in assignment (expression has type "str", variable has type "int")
</span></code></pre>

</div>



<p>mypy configuration for a production Python project typically looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># pyproject.toml</span>
<span class="nn">[tool.mypy]</span>
<span class="py">python_version</span> <span class="p">=</span> <span class="s">"3.12"</span>
<span class="py">strict</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">warn_return_any</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">warn_unused_configs</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">disallow_untyped_defs</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">disallow_incomplete_defs</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">check_untyped_defs</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">disallow_untyped_decorators</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">no_implicit_optional</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">warn_redundant_casts</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">warn_unused_ignores</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">warn_no_return</span> <span class="p">=</span> <span class="kc">true</span>

<span class="c"># Per-module overrides for third-party libraries without stubs</span>
<span class="nn">[[tool.mypy.overrides]]</span>
<span class="py">module</span> <span class="p">=</span> <span class="p">[</span><span class="s">"celery.*"</span><span class="p">,</span> <span class="s">"redis.*"</span><span class="p">]</span>
<span class="py">ignore_missing_imports</span> <span class="p">=</span> <span class="kc">true</span>

<span class="c"># Stricter settings for core business logic</span>
<span class="nn">[[tool.mypy.overrides]]</span>
<span class="py">module</span> <span class="p">=</span> <span class="p">[</span><span class="s">"app.core.*"</span><span class="p">,</span> <span class="s">"app.domain.*"</span><span class="p">]</span>
<span class="py">disallow_any_generics</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">disallow_any_unimported</span> <span class="p">=</span> <span class="kc">true</span>
</code></pre>

</div>



<p><strong>mypy with Django:</strong></p>

<p>Django's dynamic nature - model fields becoming instance attributes, queryset chaining, generic views - historically made mypy integration painful. The <code>django-stubs</code> package (maintained by the TypedDjango project) has solved most of these issues. It provides type stubs that make mypy understand Django models, views, forms, and the ORM:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># With django-stubs, mypy understands Django ORM types
</span><span class="kn">from</span> <span class="n">myapp.models</span> <span class="kn">import</span> <span class="n">User</span>

<span class="k">def</span> <span class="nf">get_active_users</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="nb">list</span><span class="p">[</span><span class="n">User</span><span class="p">]:</span>
    <span class="c1"># mypy knows .filter() returns a QuerySet[User]
</span>    <span class="c1"># mypy knows .values_list() changes the return type
</span>    <span class="n">users</span> <span class="o">=</span> <span class="n">User</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">filter</span><span class="p">(</span><span class="n">is_active</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">list</span><span class="p">(</span><span class="n">users</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Pyright - The faster alternative
</h3>

<p>Pyright, developed by Microsoft, is the type checker that powers VS Code's Pylance extension. It is significantly faster than mypy on large codebases - often 3 to 5 times faster - because it uses incremental analysis and is written in TypeScript with a focus on performance.</p>

<p>Pyright's strict mode is also stricter than mypy's strict mode. It catches patterns that mypy allows by default, such as implicit <code>Any</code> returns from untyped third-party libraries. This makes Pyright a good choice for teams that want maximum type safety, though it also means more initial configuration work to suppress false positives from libraries without complete type stubs.</p>

<p><strong>The practical recommendation:</strong></p>

<p>Many production Python teams use both tools. Pyright provides real-time feedback in VS Code through Pylance, giving developers immediate type error highlighting as they write code. mypy runs in CI/CD as the authoritative type check, because it has broader community adoption and more third-party stub packages. The two tools agree on the vast majority of checks, and where they disagree, the differences are usually edge cases around advanced generic types or protocol classes.</p>

<h2>
  
  
  Platform tools with Python-specific capabilities
</h2>

<p>The tools above are all open-source and Python-specific. The following platform tools provide broader functionality - AI review, security scanning, quality tracking - with strong Python support as part of a multi-language offering.</p>

<h3>
  
  
  Sourcery - Purpose-built AI for Python
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbxa6m3yu5a8qdi0k0sh4.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbxa6m3yu5a8qdi0k0sh4.png" alt="Sourcery screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/sourcery">Sourcery</a> was originally built exclusively for Python and has expanded to support JavaScript, TypeScript, and Go. But Python remains its strongest language by a significant margin, and it is the only AI code review tool that was designed from the ground up for Python's idioms and patterns.</p>

<p><strong>What makes Sourcery Python-specific:</strong></p>

<p>Sourcery understands Pythonic patterns at a deep level. It does not just flag issues - it suggests refactorings that transform correct-but-verbose code into idiomatic Python. Here are examples of the kinds of transformations Sourcery suggests:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Before: Sourcery flags this as non-Pythonic
</span><span class="n">filtered_users</span> <span class="o">=</span> <span class="p">[]</span>
<span class="k">for</span> <span class="n">user</span> <span class="ow">in</span> <span class="n">users</span><span class="p">:</span>
    <span class="k">if</span> <span class="n">user</span><span class="p">.</span><span class="n">is_active</span> <span class="ow">and</span> <span class="n">user</span><span class="p">.</span><span class="n">age</span> <span class="o">&gt;=</span> <span class="mi">18</span><span class="p">:</span>
        <span class="n">filtered_users</span><span class="p">.</span><span class="nf">append</span><span class="p">(</span><span class="n">user</span><span class="p">.</span><span class="n">name</span><span class="p">)</span>

<span class="c1"># After: Sourcery suggests list comprehension
</span><span class="n">filtered_users</span> <span class="o">=</span> <span class="p">[</span>
    <span class="n">user</span><span class="p">.</span><span class="n">name</span> <span class="k">for</span> <span class="n">user</span> <span class="ow">in</span> <span class="n">users</span>
    <span class="k">if</span> <span class="n">user</span><span class="p">.</span><span class="n">is_active</span> <span class="ow">and</span> <span class="n">user</span><span class="p">.</span><span class="n">age</span> <span class="o">&gt;=</span> <span class="mi">18</span>
<span class="p">]</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Before: Sourcery detects unnecessary else after return
</span><span class="k">def</span> <span class="nf">get_status</span><span class="p">(</span><span class="n">code</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">code</span> <span class="o">==</span> <span class="mi">200</span><span class="p">:</span>
        <span class="k">return</span> <span class="sh">"</span><span class="s">ok</span><span class="sh">"</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="k">if</span> <span class="n">code</span> <span class="o">==</span> <span class="mi">404</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">not found</span><span class="sh">"</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="k">return</span> <span class="sh">"</span><span class="s">error</span><span class="sh">"</span>

<span class="c1"># After: Sourcery suggests flat structure
</span><span class="k">def</span> <span class="nf">get_status</span><span class="p">(</span><span class="n">code</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">code</span> <span class="o">==</span> <span class="mi">200</span><span class="p">:</span>
        <span class="k">return</span> <span class="sh">"</span><span class="s">ok</span><span class="sh">"</span>
    <span class="k">if</span> <span class="n">code</span> <span class="o">==</span> <span class="mi">404</span><span class="p">:</span>
        <span class="k">return</span> <span class="sh">"</span><span class="s">not found</span><span class="sh">"</span>
    <span class="k">return</span> <span class="sh">"</span><span class="s">error</span><span class="sh">"</span>
</code></pre>

</div>



<p>These are not trivial style preferences. Pythonic code is genuinely easier to read, maintain, and review. Sourcery's suggestions consistently move code toward the patterns that experienced Python developers would write from scratch.</p>

<p><strong>Sourcery's code quality metrics:</strong></p>

<p>Beyond refactoring suggestions, Sourcery computes a quality score for every function in your codebase based on four dimensions: complexity (cyclomatic and cognitive), method length, working memory (number of variables a reader must track), and nesting depth. Functions scoring below thresholds are flagged in PR reviews with specific suggestions for improvement.</p>

<p><strong>Pricing and limitations:</strong></p>

<p>Sourcery's free tier is limited to open-source projects. The paid tier at $29/seat/month unlocks private repos and team features. The biggest limitation is language support - if your codebase is primarily Python, Sourcery is excellent. If you have significant TypeScript, Go, or Java code alongside Python, you may prefer a multi-language tool like CodeRabbit or DeepSource and lose the depth of Python-specific suggestions.</p>




<h3>
  
  
  CodeRabbit - Best overall AI review with strong Python support
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwmt6h59ydofxtvw1vwh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnwmt6h59ydofxtvw1vwh.png" alt="CodeRabbit screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/coderabbit">CodeRabbit</a> is the most widely installed AI code review tool on GitHub, and while it is not Python-specific, its Python support is among the best of any multi-language AI reviewer. CodeRabbit uses LLMs to understand code semantics, which means it can reason about Python code at a level that rule-based tools cannot reach.</p>

<p><strong>Python-specific capabilities:</strong></p>

<p>CodeRabbit's strength with Python code comes from its ability to understand context across an entire pull request. It catches issues like:</p>

<ul>
<li>
<strong>Missing error handling in async code</strong> - Flagging <code>await</code> calls without try/except in FastAPI endpoints where unhandled exceptions return 500 errors</li>
<li>
<strong>Django ORM N+1 queries</strong> - Detecting loops that trigger individual database queries when <code>select_related()</code> or <code>prefetch_related()</code> would be more efficient</li>
<li>
<strong>Type annotation inconsistencies</strong> - Noticing when function signatures declare <code>Optional[str]</code> but the implementation never handles <code>None</code>
</li>
<li>
<strong>Test coverage gaps</strong> - Pointing out that a new function with three code paths only has tests for two of them</li>
</ul>

<p>You can tune CodeRabbit's Python-specific behavior through its natural language configuration:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .coderabbit.yaml - Python-specific instructions</span>
<span class="na">reviews</span><span class="pi">:</span>
  <span class="na">instructions</span><span class="pi">:</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">For</span><span class="nv"> </span><span class="s">Python</span><span class="nv"> </span><span class="s">files,</span><span class="nv"> </span><span class="s">always</span><span class="nv"> </span><span class="s">check</span><span class="nv"> </span><span class="s">for</span><span class="nv"> </span><span class="s">proper</span><span class="nv"> </span><span class="s">exception</span><span class="nv"> </span><span class="s">handling</span><span class="nv"> </span><span class="s">in</span><span class="nv"> </span><span class="s">Django</span><span class="nv"> </span><span class="s">views"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">Flag</span><span class="nv"> </span><span class="s">any</span><span class="nv"> </span><span class="s">use</span><span class="nv"> </span><span class="s">of</span><span class="nv"> </span><span class="s">mutable</span><span class="nv"> </span><span class="s">default</span><span class="nv"> </span><span class="s">arguments</span><span class="nv"> </span><span class="s">in</span><span class="nv"> </span><span class="s">function</span><span class="nv"> </span><span class="s">signatures"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">Warn</span><span class="nv"> </span><span class="s">when</span><span class="nv"> </span><span class="s">Django</span><span class="nv"> </span><span class="s">model</span><span class="nv"> </span><span class="s">fields</span><span class="nv"> </span><span class="s">lack</span><span class="nv"> </span><span class="s">help_text</span><span class="nv"> </span><span class="s">or</span><span class="nv"> </span><span class="s">verbose_name"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">Check</span><span class="nv"> </span><span class="s">that</span><span class="nv"> </span><span class="s">all</span><span class="nv"> </span><span class="s">FastAPI</span><span class="nv"> </span><span class="s">endpoints</span><span class="nv"> </span><span class="s">have</span><span class="nv"> </span><span class="s">proper</span><span class="nv"> </span><span class="s">response_model</span><span class="nv"> </span><span class="s">types"</span>
    <span class="pi">-</span> <span class="s2">"</span><span class="s">Flag</span><span class="nv"> </span><span class="s">bare</span><span class="nv"> </span><span class="s">except</span><span class="nv"> </span><span class="s">clauses</span><span class="nv"> </span><span class="s">-</span><span class="nv"> </span><span class="s">always</span><span class="nv"> </span><span class="s">catch</span><span class="nv"> </span><span class="s">specific</span><span class="nv"> </span><span class="s">exception</span><span class="nv"> </span><span class="s">types"</span>
</code></pre>

</div>



<p><strong>The free tier advantage:</strong></p>

<p>CodeRabbit's unlimited free tier for both public and private repos makes it the easiest AI review tool to adopt. A Python team can install it on their GitHub organization in under five minutes and start getting AI reviews on every pull request immediately. The Pro tier at $24/user/month adds custom review profiles and advanced configuration, but the free tier is genuinely useful for most teams.</p>

<p><strong>Honest limitations for Python teams:</strong></p>

<p>CodeRabbit does not replace Ruff, mypy, or Bandit. It catches a different class of issues - logic errors, missing edge cases, architectural concerns - but it does not enforce linting rules deterministically and it does not perform formal type checking. The best Python workflow uses CodeRabbit alongside dedicated Python tools rather than as a replacement for them.</p>




<h3>
  
  
  DeepSource - Low false positives with Python autofix
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb5unb078gtfj88nul328.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb5unb078gtfj88nul328.png" alt="DeepSource screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/deepsource">DeepSource</a> combines static analysis with AI-powered review and has one of the strongest Python analyzers among the multi-language platforms. It specifically markets a sub-5% false positive rate, which is a meaningful claim for Python where dynamic typing often causes static analysis tools to generate noise.</p>

<p><strong>Python analysis depth:</strong></p>

<p>DeepSource's Python analyzer covers over 150 Python-specific issues across categories: anti-patterns, bug risks, performance, security, style, and type checking. Notable Python-specific checks include:</p>

<ul>
<li>
<strong>Mutable default arguments</strong> - The classic Python gotcha where <code>def func(items=[])</code> shares the list across calls</li>
<li>
<strong>Late binding closures</strong> - Detecting closures in loops that capture the variable rather than the value</li>
<li>
<strong>Incorrect use of <code>__all__</code></strong> - Catching exports that reference names not defined in the module</li>
<li>
<strong>Django-specific checks</strong> - Missing database indexes, unoptimized query patterns, insecure settings</li>
<li>
<strong>Pandas anti-patterns</strong> - Detecting iterative operations on DataFrames where vectorized operations exist</li>
</ul>

<p>DeepSource's Autofix feature is particularly strong for Python. When it detects an issue, it often generates a fix that can be applied directly from the pull request comment. For Python, these fixes are usually correct and Pythonic - they are not just mechanical transformations but reflect an understanding of Python idioms.</p>

<p><strong>Integration with Python tooling:</strong></p>

<p>DeepSource can run mypy, Ruff, and other Python tools as part of its analysis pipeline and surface their results in its unified dashboard. This means you can get Ruff linting, mypy type checking, and DeepSource's own analysis all reported in one place on your pull requests, with consistent formatting and a single configuration file:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># .deepsource.toml</span>
<span class="py">version</span> <span class="p">=</span> <span class="mi">1</span>

<span class="nn">[[analyzers]]</span>
<span class="py">name</span> <span class="p">=</span> <span class="s">"python"</span>
<span class="py">enabled</span> <span class="p">=</span> <span class="kc">true</span>

  <span class="nn">[analyzers.meta]</span>
  <span class="py">runtime_version</span> <span class="p">=</span> <span class="s">"3.x"</span>
  <span class="py">max_line_length</span> <span class="p">=</span> <span class="mi">120</span>
  <span class="py">type_checker</span> <span class="p">=</span> <span class="s">"mypy"</span>

<span class="nn">[[transformers]]</span>
<span class="py">name</span> <span class="p">=</span> <span class="s">"ruff"</span>
<span class="py">enabled</span> <span class="p">=</span> <span class="kc">true</span>
</code></pre>

</div>



<p><strong>Pricing:</strong></p>

<p>DeepSource offers a free tier for individual developers. The Team plan starts at $30/user/month. For Python teams that want a single platform covering linting, type checking, and AI review, DeepSource is one of the most integrated options available.</p>




<h3>
  
  
  SonarQube - Enterprise Python analysis with 500+ rules
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F540pvsw6efbieonjw1px.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F540pvsw6efbieonjw1px.png" alt="SonarQube screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/sonarqube">SonarQube</a> has over 500 rules specifically for Python, making it one of the deepest rule-based analyzers for the language. For enterprise teams that need compliance reporting, quality gate enforcement, and long-term trend tracking, SonarQube's Python support is comprehensive.</p>

<p><strong>Python-specific rule categories:</strong></p>

<p>SonarQube's Python rules cover bug detection, code smells, security vulnerabilities, and security hotspots. Some of the more valuable Python-specific rules include:</p>

<ul>
<li>
<strong>Cognitive complexity tracking</strong> - Measures how hard a function is to understand (not just how many branches it has) and flags functions that exceed configurable thresholds</li>
<li>
<strong>Django security rules</strong> - Detection of raw SQL queries, missing CSRF middleware, insecure session configuration, and improper use of <code>mark_safe()</code>
</li>
<li>
<strong>Flask security rules</strong> - Detection of debug mode in production, missing secure headers, and improper secret key handling</li>
<li>
<strong>Dead code detection</strong> - Identification of unreachable code paths, unused imports, and variables that are assigned but never read</li>
<li>
<strong>Exception handling analysis</strong> - Flagging overly broad exception handlers, empty except blocks, and re-raising patterns that lose the original traceback</li>
</ul>

<p><strong>SonarQube's type-aware analysis:</strong></p>

<p>Unlike simpler linters that only look at syntax, SonarQube performs cross-file analysis on Python code. It follows import chains to understand which modules and classes are available, and it uses this information to detect issues like accessing attributes that do not exist on imported objects or calling functions with the wrong number of arguments.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># SonarQube catches cross-file issues
# file: utils.py
</span><span class="k">def</span> <span class="nf">calculate_tax</span><span class="p">(</span><span class="n">amount</span><span class="p">:</span> <span class="nb">float</span><span class="p">,</span> <span class="n">rate</span><span class="p">:</span> <span class="nb">float</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="nb">float</span><span class="p">:</span>
    <span class="k">return</span> <span class="n">amount</span> <span class="o">*</span> <span class="n">rate</span>

<span class="c1"># file: views.py
</span><span class="kn">from</span> <span class="n">.utils</span> <span class="kn">import</span> <span class="n">calculate_tax</span>

<span class="k">def</span> <span class="nf">order_summary</span><span class="p">(</span><span class="n">request</span><span class="p">,</span> <span class="n">order_id</span><span class="p">):</span>
    <span class="n">order</span> <span class="o">=</span> <span class="n">Order</span><span class="p">.</span><span class="n">objects</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="nb">id</span><span class="o">=</span><span class="n">order_id</span><span class="p">)</span>
    <span class="c1"># SonarQube: calculate_tax expects 2 arguments, 3 given
</span>    <span class="n">tax</span> <span class="o">=</span> <span class="nf">calculate_tax</span><span class="p">(</span><span class="n">order</span><span class="p">.</span><span class="n">subtotal</span><span class="p">,</span> <span class="n">order</span><span class="p">.</span><span class="n">tax_rate</span><span class="p">,</span> <span class="n">order</span><span class="p">.</span><span class="n">state</span><span class="p">)</span>
</code></pre>

</div>



<p><strong>Quality gates for Python teams:</strong></p>

<p>SonarQube's quality gate feature is particularly useful for Python projects with growing teams. You can set thresholds for metrics like code coverage, duplicated lines, new bugs, and new security vulnerabilities. Pull requests that do not meet these thresholds are blocked from merging, which provides a consistent quality baseline regardless of who reviewed the code:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Quality Gate: Python Production
- New Coverage: &gt;= 80%
- New Duplicated Lines: &lt;= 3%
- New Bugs: 0
- New Vulnerabilities: 0
- New Security Hotspots Reviewed: 100%
- New Cognitive Complexity: &lt;= 15 per function
</code></pre>

</div>



<p><strong>Limitations for Python teams:</strong></p>

<p>SonarQube's self-hosting requirement is its biggest barrier. The free Community Build lacks branch analysis and PR decoration, which makes it impractical for teams using pull request workflows. The Developer Edition (required for PR integration) requires a paid license and a maintained server. SonarQube Cloud eliminates hosting overhead but uses LOC-based pricing that can become expensive for large Python monorepos. Teams that want SonarQube-level rule depth without the hosting burden should consider Codacy or DeepSource as alternatives.</p>




<h3>
  
  
  Semgrep - Custom security rules for Django and Flask
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqmrt83uxxdprc9jg2e5n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqmrt83uxxdprc9jg2e5n.png" alt="Semgrep screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/semgrep">Semgrep</a> is an open-source static analysis tool that excels at writing custom rules using a pattern-matching syntax that looks like the code it analyzes. For Python security scanning, Semgrep's approach is uniquely powerful because you can write rules that understand Python-specific and framework-specific patterns.</p>

<p><strong>Pre-built Python security rulesets:</strong></p>

<p>Semgrep maintains official rulesets for Python, Django, and Flask that cover hundreds of vulnerability patterns:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Running Semgrep with Python security rulesets</span>
<span class="c1"># semgrep --config p/python --config p/django --config p/flask</span>

<span class="c1"># Example rule: detecting SQL injection in Django</span>
<span class="na">rules</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">django-raw-sql-injection</span>
    <span class="na">patterns</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">pattern</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">$MODEL.objects.raw($QUERY)</span>
      <span class="pi">-</span> <span class="na">pattern-not</span><span class="pi">:</span> <span class="pi">|</span>
          <span class="s">$MODEL.objects.raw("...", [...])</span>
    <span class="na">message</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Potential</span><span class="nv"> </span><span class="s">SQL</span><span class="nv"> </span><span class="s">injection</span><span class="nv"> </span><span class="s">via</span><span class="nv"> </span><span class="s">Django</span><span class="nv"> </span><span class="s">raw</span><span class="nv"> </span><span class="s">query</span><span class="nv"> </span><span class="s">without</span><span class="nv"> </span><span class="s">parameterization"</span>
    <span class="na">languages</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">python</span><span class="pi">]</span>
    <span class="na">severity</span><span class="pi">:</span> <span class="s">ERROR</span>
    <span class="na">metadata</span><span class="pi">:</span>
      <span class="na">cwe</span><span class="pi">:</span> <span class="s2">"</span><span class="s">CWE-89:</span><span class="nv"> </span><span class="s">SQL</span><span class="nv"> </span><span class="s">Injection"</span>
      <span class="na">owasp</span><span class="pi">:</span> <span class="s2">"</span><span class="s">A03:2021</span><span class="nv"> </span><span class="s">Injection"</span>
</code></pre>

</div>



<p>The Django ruleset covers SQL injection through the ORM, XSS through <code>mark_safe()</code> and <code>|safe</code> template filters, CSRF bypass patterns, insecure file upload handling, and authentication weaknesses. The Flask ruleset covers similar categories plus Flask-specific concerns like debug mode in production, unsafe deserialization, and missing security headers.</p>

<p><strong>Writing custom Python rules:</strong></p>

<p>Semgrep's pattern syntax makes it straightforward to encode your team's specific Python security and quality requirements:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Custom rule: prevent environment variable access without defaults</span>
<span class="na">rules</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">require-env-default</span>
    <span class="na">pattern</span><span class="pi">:</span> <span class="s">os.environ[$KEY]</span>
    <span class="na">fix</span><span class="pi">:</span> <span class="s">os.environ.get($KEY, "")</span>
    <span class="na">message</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Use</span><span class="nv"> </span><span class="s">os.environ.get()</span><span class="nv"> </span><span class="s">with</span><span class="nv"> </span><span class="s">a</span><span class="nv"> </span><span class="s">default</span><span class="nv"> </span><span class="s">value</span><span class="nv"> </span><span class="s">instead</span><span class="nv"> </span><span class="s">of</span><span class="nv"> </span><span class="s">direct</span><span class="nv"> </span><span class="s">access"</span>
    <span class="na">languages</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">python</span><span class="pi">]</span>
    <span class="na">severity</span><span class="pi">:</span> <span class="s">WARNING</span>

  <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">no-pickle-loads</span>
    <span class="na">pattern</span><span class="pi">:</span> <span class="s">pickle.loads(...)</span>
    <span class="na">message</span><span class="pi">:</span> <span class="s2">"</span><span class="s">pickle.loads()</span><span class="nv"> </span><span class="s">is</span><span class="nv"> </span><span class="s">unsafe</span><span class="nv"> </span><span class="s">with</span><span class="nv"> </span><span class="s">untrusted</span><span class="nv"> </span><span class="s">data.</span><span class="nv"> </span><span class="s">Use</span><span class="nv"> </span><span class="s">json.loads()</span><span class="nv"> </span><span class="s">instead"</span>
    <span class="na">languages</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">python</span><span class="pi">]</span>
    <span class="na">severity</span><span class="pi">:</span> <span class="s">ERROR</span>

  <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">django-no-wildcard-import</span>
    <span class="na">pattern</span><span class="pi">:</span> <span class="s">from django.conf.settings import *</span>
    <span class="na">message</span><span class="pi">:</span> <span class="s2">"</span><span class="s">Wildcard</span><span class="nv"> </span><span class="s">imports</span><span class="nv"> </span><span class="s">from</span><span class="nv"> </span><span class="s">Django</span><span class="nv"> </span><span class="s">settings</span><span class="nv"> </span><span class="s">make</span><span class="nv"> </span><span class="s">dependencies</span><span class="nv"> </span><span class="s">unclear"</span>
    <span class="na">languages</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">python</span><span class="pi">]</span>
    <span class="na">severity</span><span class="pi">:</span> <span class="s">WARNING</span>
</code></pre>

</div>



<p><strong>Semgrep's Python dataflow analysis:</strong></p>

<p>Beyond pattern matching, Semgrep supports taint tracking for Python. It can follow data from sources (like <code>request.GET</code>, <code>request.POST</code>, or <code>request.FILES</code> in Django) through intermediate variables and function calls to sinks (like <code>cursor.execute()</code>, <code>subprocess.run()</code>, or <code>eval()</code>). This catches vulnerabilities that simple pattern matching misses - cases where untrusted input is stored in a variable, passed through several functions, and eventually reaches a dangerous operation.</p>

<p><strong>Pricing:</strong></p>

<p>Semgrep's OSS engine is free and can be run locally or in CI/CD. Semgrep Cloud (managed service with dashboard, policy management, and team features) is free for up to 10 contributors and $35/contributor/month after that. For most Python teams, the OSS engine running in GitHub Actions is sufficient for security scanning.</p>




<h3>
  
  
  Snyk Code - Cross-file Python security analysis
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmkjvaoxeyonhcutzft0e.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmkjvaoxeyonhcutzft0e.png" alt="Snyk Code screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/snyk-code">Snyk Code</a> is the SAST component of Snyk's security platform. Its primary differentiator for Python teams is cross-file dataflow analysis that traces data through imports, function calls, and class hierarchies across your entire codebase - not just individual files.</p>

<p><strong>Python-specific dataflow analysis:</strong></p>

<p>Snyk Code builds a semantic model of your Python application and uses it to detect vulnerabilities that span multiple files. Consider a common Django pattern:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># file: views.py
</span><span class="k">def</span> <span class="nf">create_report</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="n">data</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">POST</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">report_data</span><span class="sh">"</span><span class="p">)</span>
    <span class="n">report</span> <span class="o">=</span> <span class="n">ReportService</span><span class="p">.</span><span class="nf">generate</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>
    <span class="k">return</span> <span class="nc">JsonResponse</span><span class="p">(</span><span class="n">report</span><span class="p">)</span>

<span class="c1"># file: services.py
</span><span class="k">class</span> <span class="nc">ReportService</span><span class="p">:</span>
    <span class="nd">@staticmethod</span>
    <span class="k">def</span> <span class="nf">generate</span><span class="p">(</span><span class="n">data</span><span class="p">):</span>
        <span class="c1"># Several layers of indirection
</span>        <span class="n">parsed</span> <span class="o">=</span> <span class="n">ReportParser</span><span class="p">.</span><span class="nf">parse</span><span class="p">(</span><span class="n">data</span><span class="p">)</span>
        <span class="k">return</span> <span class="n">ReportBuilder</span><span class="p">.</span><span class="nf">build</span><span class="p">(</span><span class="n">parsed</span><span class="p">)</span>

<span class="c1"># file: builders.py
</span><span class="k">class</span> <span class="nc">ReportBuilder</span><span class="p">:</span>
    <span class="nd">@staticmethod</span>
    <span class="k">def</span> <span class="nf">build</span><span class="p">(</span><span class="n">parsed_data</span><span class="p">):</span>
        <span class="n">query</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">SELECT * FROM reports WHERE data = </span><span class="sh">'</span><span class="si">{</span><span class="n">parsed_data</span><span class="si">}</span><span class="sh">'"</span>
        <span class="c1"># SQL injection - but the taint source is 3 files away
</span>        <span class="k">return</span> <span class="n">db</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="n">query</span><span class="p">)</span>
</code></pre>

</div>



<p>A single-file scanner would not flag <code>builders.py</code> because it cannot see that <code>parsed_data</code> originates from <code>request.POST</code>. Snyk Code traces the data flow across all three files and flags the SQL injection with the full path from source to sink.</p>

<p><strong>Python dependency scanning:</strong></p>

<p>Snyk's broader platform also covers dependency vulnerabilities in <code>requirements.txt</code>, <code>Pipfile.lock</code>, <code>poetry.lock</code>, and <code>pyproject.toml</code>. This means a single Snyk installation covers both your code and your dependencies. For Python teams concerned about supply chain security, this combination is particularly valuable - you get SAST and SCA (Software Composition Analysis) from one tool.</p>

<p><strong>Integration with Python CI/CD:</strong></p>

<p>Snyk Code integrates with GitHub, GitLab, Bitbucket, and Azure DevOps for PR-level scanning. It also provides a CLI (<code>snyk code test</code>) that can run in any CI pipeline and a VS Code extension for real-time feedback during development. The Python-specific analysis typically completes in 1 to 3 minutes for medium-sized codebases.</p>

<p><strong>Pricing:</strong></p>

<p>Snyk offers a free tier with limited scans for a single user. The Team plan at $25/developer/month includes Snyk Code (SAST), Snyk Open Source (SCA), and basic container scanning. For Python teams that need both code security and dependency security, Snyk's bundled offering is cost-effective compared to running separate tools.</p>




<h3>
  
  
  Codacy - All-in-one Python quality for small teams
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpgxqycqyyo2u17sq1fkj.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpgxqycqyyo2u17sq1fkj.png" alt="Codacy screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/codacy">Codacy</a> is a code quality platform that aggregates results from multiple Python tools into a unified dashboard. Rather than building its own Python analyzer from scratch, Codacy runs established Python tools - Pylint, Bandit, Prospector, and others - and surfaces their results alongside its own analysis.</p>

<p><strong>Python tool aggregation:</strong></p>

<p>Codacy's approach means you get the combined coverage of multiple Python-specific tools without configuring each one individually. For a Python project, Codacy typically runs:</p>

<ul>
<li>
<strong>Pylint</strong> for code quality and style enforcement</li>
<li>
<strong>Bandit</strong> for security vulnerability detection</li>
<li>
<strong>Prospector</strong> for additional Python-specific checks</li>
<li>
<strong>Radon</strong> for cyclomatic complexity and maintainability index</li>
<li>Custom pattern matching for common Python anti-patterns</li>
</ul>

<p>The results are deduplicated, organized by severity, and presented in Codacy's dashboard alongside metrics like code coverage trends, duplication percentages, and technical debt estimates.</p>

<p><strong>Django and Flask support:</strong></p>

<p>Codacy understands Django and Flask project structures. It knows to apply different rule sets to <code>models.py</code>, <code>views.py</code>, <code>urls.py</code>, <code>settings.py</code>, and test files. Security rules are applied more aggressively to view functions and API endpoints, while complexity thresholds may be relaxed for Django migrations (which are auto-generated and inherently verbose).</p>

<p><strong>Pricing advantage:</strong></p>

<p>At $15/user/month, Codacy is the most affordable full-featured code quality platform for Python teams. The free tier supports limited repositories and is sufficient for small open-source projects. For teams that want a single platform covering linting, security, complexity, and coverage tracking without the operational overhead of SonarQube, Codacy is a strong value proposition.</p>

<p><strong>Limitations:</strong></p>

<p>Codacy's Python analysis depth is limited by the underlying tools it runs. It does not perform the cross-file dataflow analysis that Snyk Code or Semgrep offer, and its AI review capabilities are more basic than tools like CodeRabbit or Sourcery. For teams that need deep security scanning or sophisticated AI review, Codacy works best as a baseline quality platform supplemented by a specialized tool.</p>




<h3>
  
  
  PR-Agent (Qodo Merge) - Self-hosted AI review for Python
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6nlzgpum4n4p7odtnfo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6nlzgpum4n4p7odtnfo.png" alt="PR-Agent screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/pr-agent">PR-Agent</a> is an open-source AI code review tool created by Qodo (formerly CodiumAI) that you can self-host. For Python teams in regulated industries or organizations with strict data handling requirements, PR-Agent provides AI review capabilities without sending code to third-party servers.</p>

<p><strong>Python-specific review quality:</strong></p>

<p>PR-Agent uses LLMs (configurable - GPT-4, Claude, or self-hosted models) to review pull requests. For Python code, it provides:</p>

<ul>
<li>
<strong>Automatic PR descriptions</strong> - Generates a summary of changes with a focus on what the Python code does, not just what files changed</li>
<li>
<strong>Code suggestions</strong> - Python-specific refactoring suggestions including Pythonic patterns, error handling improvements, and performance optimizations</li>
<li>
<strong>Review comments</strong> - Inline comments flagging potential bugs, security issues, and maintainability concerns</li>
<li>
<strong>Test suggestions</strong> - Identifies untested code paths and suggests test cases</li>
</ul>

<p>The self-hosted option means you run PR-Agent on your own infrastructure, connecting it to your own LLM API keys. This keeps your Python source code within your network - a critical requirement for teams working with proprietary algorithms or sensitive data.</p>

<p><strong>Configuration for Python projects:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># .pr_agent.toml</span>
<span class="nn">[pr_description]</span>
<span class="py">enable_semantic_files_types</span> <span class="p">=</span> <span class="kc">true</span>

<span class="nn">[pr_reviewer]</span>
<span class="py">extra_instructions</span> <span class="p">=</span> <span class="s">"""
For Python files:
- Check for proper type annotations on all public functions
- Flag any use of mutable default arguments
- Verify Django views have proper permission decorators
- Check that FastAPI endpoints define response models
"""</span>
</code></pre>

</div>



<p><strong>Pricing:</strong></p>

<p>PR-Agent's open-source version is free to self-host. The managed version (Qodo Merge) costs $19/user/month and eliminates the need to maintain your own deployment. For Python teams that want AI review with full control over data handling, PR-Agent is the only serious open-source option.</p>




<h3>
  
  
  Qodo - AI test generation for Python
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fid5jvbatb7drwkk5ns2g.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fid5jvbatb7drwkk5ns2g.png" alt="Qodo screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/qodo">Qodo</a> (formerly CodiumAI) focuses on a specific dimension of code review that other tools largely ignore: test quality. For Python teams, where achieving meaningful test coverage requires understanding dynamic types, mock patterns, and framework-specific test utilities, Qodo's AI-generated tests are genuinely useful.</p>

<p><strong>Python test generation:</strong></p>

<p>Qodo analyzes your Python functions and generates pytest test cases that cover edge cases a human reviewer might miss:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="c1"># Given this function:
</span><span class="k">def</span> <span class="nf">parse_config</span><span class="p">(</span><span class="n">filepath</span><span class="p">:</span> <span class="nb">str</span><span class="p">,</span> <span class="n">required_keys</span><span class="p">:</span> <span class="nb">list</span><span class="p">[</span><span class="nb">str</span><span class="p">])</span> <span class="o">-&gt;</span> <span class="nb">dict</span><span class="p">:</span>
    <span class="k">with</span> <span class="nf">open</span><span class="p">(</span><span class="n">filepath</span><span class="p">)</span> <span class="k">as</span> <span class="n">f</span><span class="p">:</span>
        <span class="n">config</span> <span class="o">=</span> <span class="n">json</span><span class="p">.</span><span class="nf">load</span><span class="p">(</span><span class="n">f</span><span class="p">)</span>
    <span class="n">missing</span> <span class="o">=</span> <span class="p">[</span><span class="n">k</span> <span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="n">required_keys</span> <span class="k">if</span> <span class="n">k</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">config</span><span class="p">]</span>
    <span class="k">if</span> <span class="n">missing</span><span class="p">:</span>
        <span class="k">raise</span> <span class="nc">ValueError</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Missing required keys: </span><span class="si">{</span><span class="n">missing</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">config</span>

<span class="c1"># Qodo generates tests like:
</span><span class="k">def</span> <span class="nf">test_parse_config_valid</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Test with a valid config file containing all required keys.</span><span class="sh">"""</span>
    <span class="bp">...</span>

<span class="k">def</span> <span class="nf">test_parse_config_missing_keys</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Test that ValueError is raised when required keys are absent.</span><span class="sh">"""</span>
    <span class="bp">...</span>

<span class="k">def</span> <span class="nf">test_parse_config_file_not_found</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Test that FileNotFoundError is raised for nonexistent file.</span><span class="sh">"""</span>
    <span class="bp">...</span>

<span class="k">def</span> <span class="nf">test_parse_config_invalid_json</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Test that JSONDecodeError is raised for malformed JSON.</span><span class="sh">"""</span>
    <span class="bp">...</span>

<span class="k">def</span> <span class="nf">test_parse_config_empty_required_keys</span><span class="p">():</span>
    <span class="sh">"""</span><span class="s">Test with an empty required_keys list returns any valid config.</span><span class="sh">"""</span>
    <span class="bp">...</span>
</code></pre>

</div>



<p>For Django projects, Qodo understands Django's test client, factory patterns, and model fixtures. It generates tests that use <code>TestCase</code>, <code>RequestFactory</code>, and proper database setup/teardown patterns rather than generic unittest boilerplate.</p>

<p><strong>Integration with code review:</strong></p>

<p>Qodo integrates with GitHub and GitLab to provide test suggestions directly on pull requests. When a new Python function is added without tests, Qodo can automatically suggest test cases in the PR comments. This shifts the test coverage conversation from "did you write tests?" to "here are the tests you should add," which is a more productive review interaction.</p>

<p><strong>Pricing:</strong></p>

<p>Qodo offers a free tier with basic functionality. The Teams plan at $19/seat/month includes full test generation, PR review, and IDE integration. For Python teams where test coverage is a persistent challenge, Qodo addresses the root cause - making tests easy to write - rather than just measuring the symptom.</p>




<h3>
  
  
  Qlty - Unified code health with Python linter aggregation
</h3>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwyk79b4w4uv9dthh3end.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwyk79b4w4uv9dthh3end.png" alt="Qlty screenshot" width="800" height="500"></a></p>

<p><a href="https://dev.to/tool/qlty">Qlty</a> takes a different approach to Python code quality. Instead of building its own analyzer, Qlty provides a unified runtime that orchestrates existing Python tools and presents their results through a consistent interface. It supports over 40 languages, which makes it particularly useful for teams with polyglot codebases that include Python alongside JavaScript, Go, or Rust.</p>

<p><strong>Python tool orchestration:</strong></p>

<p>Qlty can run Ruff, Pylint, mypy, Bandit, and other Python tools as plugins, managing their installation, configuration, and output formatting. The advantage is a single <code>qlty.toml</code> configuration file and a single CLI command that runs your entire Python quality stack:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># qlty.toml</span>
<span class="nn">[plugins.ruff]</span>
<span class="py">enabled</span> <span class="p">=</span> <span class="kc">true</span>

<span class="nn">[plugins.mypy]</span>
<span class="py">enabled</span> <span class="p">=</span> <span class="kc">true</span>

<span class="nn">[plugins.bandit]</span>
<span class="py">enabled</span> <span class="p">=</span> <span class="kc">true</span>
</code></pre>

</div>



<p>Running <code>qlty check</code> executes all three tools in parallel and presents a unified output with consistent severity levels and formatting. This is particularly valuable in CI/CD where you want a single check that covers linting, type checking, and security scanning.</p>

<p><strong>Code health metrics:</strong></p>

<p>Qlty tracks code health metrics over time, including complexity trends, duplication percentages, and test coverage changes. For Python projects, this historical view helps teams identify quality trends - whether newly merged code is improving or degrading the overall codebase health.</p>

<p><strong>Pricing:</strong></p>

<p>Qlty's CLI is free for commercial use. The Cloud platform (dashboard, PR integration, team features) starts at $15/contributor/month. For Python teams already using multiple tools and looking for a unified way to run and report on them, Qlty reduces configuration overhead without replacing the underlying tools.</p>

<h2>
  
  
  Security scanning for Python: a deeper look
</h2>

<p>Security deserves its own section because Python's role in web applications (Django, Flask, FastAPI), data pipelines (pandas, Apache Airflow), and machine learning (PyTorch, TensorFlow) makes it a high-value target. Each domain has unique vulnerability patterns that require specialized detection.</p>

<h3>
  
  
  Bandit - Python-specific security analysis
</h3>

<p>Bandit is the standard open-source security scanner for Python. Developed by the OpenStack Security Project, it checks for common security issues in Python code using a set of plugins that each test for a specific vulnerability pattern.</p>

<p>Bandit catches issues like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code>
<span class="c1"># B603: subprocess call with shell=True
</span><span class="n">subprocess</span><span class="p">.</span><span class="nf">call</span><span class="p">(</span><span class="n">user_input</span><span class="p">,</span> <span class="n">shell</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span>

<span class="c1"># B301: pickle usage (deserialization of untrusted data)
</span><span class="n">data</span> <span class="o">=</span> <span class="n">pickle</span><span class="p">.</span><span class="nf">loads</span><span class="p">(</span><span class="n">untrusted_bytes</span><span class="p">)</span>

<span class="c1"># B303: use of insecure MD5 hash
</span><span class="n">password_hash</span> <span class="o">=</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">md5</span><span class="p">(</span><span class="n">password</span><span class="p">.</span><span class="nf">encode</span><span class="p">()).</span><span class="nf">hexdigest</span><span class="p">()</span>

<span class="c1"># B105: hardcoded password
</span><span class="n">DB_PASSWORD</span> <span class="o">=</span> <span class="sh">"</span><span class="s">super_secret_123</span><span class="sh">"</span>

<span class="c1"># B608: SQL injection via string formatting
</span><span class="n">query</span> <span class="o">=</span> <span class="sh">"</span><span class="s">SELECT * FROM users WHERE name = </span><span class="sh">'</span><span class="s">%s</span><span class="sh">'"</span> <span class="o">%</span> <span class="n">user_name</span>
<span class="n">cursor</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="n">query</span><span class="p">)</span>
</code></pre>

</div>



<p>Bandit integrates with CI/CD pipelines trivially:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># GitHub Actions</span>
<span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Security scan</span>
  <span class="na">run</span><span class="pi">:</span> <span class="pi">|</span>
    <span class="s">pip install bandit</span>
    <span class="s">bandit -r src/ -f json -o bandit-results.json</span>
</code></pre>

</div>



<p>The key limitation of Bandit is that it only analyzes single files. It cannot trace data flow across modules or understand framework-specific contexts. This is where tools like Semgrep and Snyk Code add value on top of Bandit's baseline coverage.</p>

<h3>
  
  
  Django-specific security concerns
</h3>

<p>Django applications face a specific set of vulnerabilities that require Python-aware analysis:</p>

<p><strong>Raw SQL and ORM bypass:</strong> Django's ORM prevents SQL injection by default, but developers sometimes bypass it with <code>raw()</code>, <code>extra()</code>, or <code>cursor.execute()</code> for complex queries. Every one of these bypass points needs to be checked for proper parameterization.</p>

<p><strong>Template injection:</strong> Django's template engine auto-escapes HTML output, but <code>mark_safe()</code>, the <code>|safe</code> filter, and <code>{% autoescape off %}</code> blocks disable this protection. A security scanner needs to verify that any content marked as safe does not include user input.</p>

<p><strong>CSRF bypass patterns:</strong> Django's CSRF middleware protects POST requests, but developers sometimes add <code>@csrf_exempt</code> decorators to views that handle API callbacks or webhook endpoints. These exemptions need review to ensure they are genuinely necessary and that alternative protections are in place.</p>

<p><strong>Insecure settings in production:</strong> <code>DEBUG = True</code>, <code>ALLOWED_HOSTS = ['*']</code>, weak <code>SECRET_KEY</code> values, and missing security middleware are common Django misconfigurations that are easy to scan for but costly if missed.</p>

<p>Semgrep's <code>p/django</code> ruleset covers all of these patterns and more. For teams that want maximum Django security coverage, running Semgrep with the Django ruleset alongside Bandit provides comprehensive protection:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Combined Django security scanning</span>
semgrep <span class="nt">--config</span> p/django <span class="nt">--config</span> p/python src/
bandit <span class="nt">-r</span> src/ <span class="nt">-ll</span>  <span class="c"># Only medium and high severity</span>
</code></pre>

</div>



<h3>
  
  
  Flask-specific security concerns
</h3>

<p>Flask's "batteries not included" philosophy means security features that Django provides by default must be explicitly added:</p>

<p><strong>No default CSRF protection:</strong> Flask applications need Flask-WTF or similar extensions for CSRF protection. A security scanner should verify that CSRF protection is installed and enabled on all state-changing endpoints.</p>

<p><strong>No default XSS protection in all contexts:</strong> Jinja2 auto-escapes HTML in templates, but not in JavaScript contexts, CSS contexts, or URL attributes. A common Flask vulnerability is inserting user input into an <code>onclick</code> handler or a <code>&lt;script&gt;</code> block where HTML escaping is insufficient.</p>

<p><strong>Debug mode exposure:</strong> Flask's debug mode includes an interactive debugger that allows arbitrary code execution. A security scanner should verify that <code>app.debug</code> is <code>False</code> and <code>app.run(debug=True)</code> is not present in production code.</p>

<p><strong>Secret key management:</strong> Flask's session signing depends on <code>app.secret_key</code>. Hardcoded or weak secret keys compromise session integrity.</p>

<h3>
  
  
  FastAPI security considerations
</h3>

<p>FastAPI has gained enormous adoption for Python APIs and introduces its own security patterns:</p>

<p><strong>Input validation through Pydantic:</strong> FastAPI uses Pydantic models for automatic request validation, which prevents many injection attacks. However, endpoints that accept raw <code>Request</code> objects or use <code>Body(embed=True)</code> bypass this validation. Security scanning should flag endpoints that do not use Pydantic models.</p>

<p><strong>Dependency injection security:</strong> FastAPI's dependency injection system is powerful but can be misused. Dependencies that access databases or external services should include proper error handling and timeout configuration.</p>

<p><strong>CORS misconfiguration:</strong> FastAPI's <code>CORSMiddleware</code> is commonly misconfigured with <code>allow_origins=["*"]</code> during development and left that way in production. Security scanners should flag overly permissive CORS settings.</p>

<h2>
  
  
  Building your Python code review stack
</h2>

<p>No single tool covers every dimension of Python code quality. The most effective Python code review setups layer tools from different categories, each handling what it does best. Here are recommended stacks for different team sizes and needs.</p>

<h3>
  
  
  Minimal stack (solo developer or small team)
</h3>

<p>For a solo developer or a team of 2 to 5 people working on a single Python project, this stack provides strong coverage with minimal configuration:</p>

<p><strong>Ruff</strong> for linting and formatting - Handles all syntactic issues, import sorting, and code formatting in a single tool. Configure it in <code>pyproject.toml</code> and add it to pre-commit hooks.</p>

<p><strong>mypy</strong> for type checking - Add type annotations incrementally and run mypy in CI to catch type errors before they merge.</p>

<p><strong>CodeRabbit</strong> for AI review - The free tier provides unlimited AI reviews on every pull request, catching logic errors and suggesting improvements that linters miss.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/python-quality.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Python Quality</span>
<span class="na">on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">pull_request</span><span class="pi">]</span>
<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">quality</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v5</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">python-version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">3.12"</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pip install ruff mypy</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Ruff lint</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">ruff check src/</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Ruff format check</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">ruff format --check src/</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">mypy type check</span>
        <span class="na">run</span><span class="pi">:</span> <span class="s">mypy src/</span>
      <span class="c1"># CodeRabbit runs automatically via GitHub App - no CI config needed</span>
</code></pre>

</div>



<p>Total cost: $0 (all tools are free at this scale).</p>

<h3>
  
  
  Standard stack (mid-size team, 5-20 developers)
</h3>

<p>For a team of 5 to 20 developers working on one or more Python services, add security scanning and deeper analysis:</p>

<p><strong>Ruff</strong> for linting and formatting (pre-commit and CI).</p>

<p><strong>mypy</strong> for type checking (CI).</p>

<p><strong>Pylint</strong> for deep analysis (CI only - too slow for pre-commit on large codebases).</p>

<p><strong>Bandit + Semgrep</strong> for security scanning. Run Bandit for Python-specific vulnerabilities and Semgrep with the Django or Flask ruleset for framework-specific issues.</p>

<p><strong>CodeRabbit or Sourcery</strong> for AI review. Choose CodeRabbit for multi-language teams or Sourcery for Python-heavy teams that want the deepest Python-specific refactoring suggestions.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .github/workflows/python-quality.yml</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Python Quality</span>
<span class="na">on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">pull_request</span><span class="pi">]</span>
<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">lint</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v5</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">python-version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">3.12"</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pip install ruff mypy pylint pylint-django bandit</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">ruff check src/</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">ruff format --check src/</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">mypy src/</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pylint --load-plugins pylint_django src/</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">bandit -r src/ -ll -f json -o bandit-report.json</span>

  <span class="na">security</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">returntocorp/semgrep-action@v1</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">config</span><span class="pi">:</span> <span class="pi">&gt;-</span>
            <span class="s">p/python</span>
            <span class="s">p/django</span>
            <span class="s">p/flask</span>
</code></pre>

</div>



<p>Total cost: $0-$29/seat/month depending on AI review tool choice.</p>

<h3>
  
  
  Enterprise stack (20+ developers, compliance requirements)
</h3>

<p>For larger teams with compliance needs, quality gate enforcement, and multiple Python services:</p>

<p><strong>Ruff</strong> for linting and formatting (pre-commit and CI).</p>

<p><strong>mypy</strong> for type checking (CI).</p>

<p><strong>SonarQube or DeepSource</strong> for centralized quality tracking with quality gates, trend analysis, and compliance reporting. SonarQube offers the deepest rule set; DeepSource offers lower false positives and better AI integration.</p>

<p><strong>Semgrep or Snyk Code</strong> for security. Semgrep for teams that want custom rule authoring and open-source flexibility; Snyk Code for teams that want cross-file dataflow analysis and combined SAST/SCA.</p>

<p><strong>CodeRabbit</strong> for AI review at scale. The enterprise tier provides SSO, self-hosted deployment options, and custom review profiles.</p>

<p><strong>Codacy or Qlty</strong> for unified quality metrics across all Python services, if the team needs a single dashboard covering multiple repositories.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># Enterprise CI pipeline - comprehensive Python analysis</span>
<span class="na">name</span><span class="pi">:</span> <span class="s">Python Quality Gate</span>
<span class="na">on</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">pull_request</span><span class="pi">]</span>
<span class="na">jobs</span><span class="pi">:</span>
  <span class="na">lint-and-type</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/setup-python@v5</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">python-version</span><span class="pi">:</span> <span class="s2">"</span><span class="s">3.12"</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">pip install ruff mypy django-stubs</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">ruff check src/ --output-format=github</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">ruff format --check src/</span>
      <span class="pi">-</span> <span class="na">run</span><span class="pi">:</span> <span class="s">mypy src/ --junit-xml=mypy-results.xml</span>

  <span class="na">sonarqube</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
        <span class="na">with</span><span class="pi">:</span>
          <span class="na">fetch-depth</span><span class="pi">:</span> <span class="m">0</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">SonarSource/sonarqube-scan-action@v5</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">SONAR_TOKEN</span><span class="pi">:</span> <span class="s">${{ secrets.SONAR_TOKEN }}</span>
          <span class="na">SONAR_HOST_URL</span><span class="pi">:</span> <span class="s">${{ secrets.SONAR_HOST_URL }}</span>

  <span class="na">security</span><span class="pi">:</span>
    <span class="na">runs-on</span><span class="pi">:</span> <span class="s">ubuntu-latest</span>
    <span class="na">steps</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">uses</span><span class="pi">:</span> <span class="s">actions/checkout@v4</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Snyk Code test</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">snyk/actions/code@master</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">SNYK_TOKEN</span><span class="pi">:</span> <span class="s">${{ secrets.SNYK_TOKEN }}</span>
      <span class="pi">-</span> <span class="na">name</span><span class="pi">:</span> <span class="s">Snyk dependency test</span>
        <span class="na">uses</span><span class="pi">:</span> <span class="s">snyk/actions/python@master</span>
        <span class="na">env</span><span class="pi">:</span>
          <span class="na">SNYK_TOKEN</span><span class="pi">:</span> <span class="s">${{ secrets.SNYK_TOKEN }}</span>
</code></pre>

</div>



<p>Total cost: Varies significantly based on tool selection and team size. Budget $20-$60/developer/month for the platform tools (SonarQube/DeepSource + Snyk/Semgrep + CodeRabbit).</p>

<h3>
  
  
  Data science and ML stack
</h3>

<p>Python data science and machine learning codebases have unique quality concerns - notebook code quality, pandas performance patterns, model serialization security, and experiment reproducibility. A tailored stack addresses these:</p>

<p><strong>Ruff</strong> for linting with the <code>NPY</code> (NumPy) and <code>PD</code> (pandas) rule sets enabled, which catch common pandas anti-patterns like using <code>.iterrows()</code> when vectorized operations exist.</p>

<p><strong>mypy</strong> with <code>numpy</code> and <code>pandas-stubs</code> for type checking data science code. Type annotations on DataFrame columns and NumPy array shapes catch shape mismatch errors before runtime.</p>

<p><strong>Bandit</strong> with extra emphasis on pickle and YAML deserialization rules - ML pipelines frequently load serialized models and config files, both of which are common attack vectors.</p>

<p><strong>DeepSource or CodeRabbit</strong> for AI review. Both are effective at reviewing data processing logic and flagging potential issues in transformation pipelines.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># pyproject.toml - Data science Ruff configuration</span>
<span class="nn">[tool.ruff.lint]</span>
<span class="py">select</span> <span class="p">=</span> <span class="p">[</span>
    <span class="s">"E"</span><span class="p">,</span> <span class="s">"W"</span><span class="p">,</span> <span class="s">"F"</span><span class="p">,</span> <span class="s">"I"</span><span class="p">,</span> <span class="s">"N"</span><span class="p">,</span> <span class="s">"UP"</span><span class="p">,</span> <span class="s">"B"</span><span class="p">,</span> <span class="s">"S"</span><span class="p">,</span>
    <span class="s">"NPY"</span><span class="p">,</span>   <span class="c"># NumPy-specific rules</span>
    <span class="s">"PD"</span><span class="p">,</span>    <span class="c"># pandas-vet rules</span>
    <span class="s">"PT"</span><span class="p">,</span>    <span class="c"># pytest style</span>
    <span class="s">"SIM"</span><span class="p">,</span>   <span class="c"># simplify</span>
    <span class="s">"RUF"</span><span class="p">,</span>   <span class="c"># Ruff-specific</span>
<span class="p">]</span>
</code></pre>

</div>



<h2>
  
  
  Integrating Python code review into your workflow
</h2>

<p>Having the right tools is only half the equation. How you integrate them into your workflow determines whether they actually improve code quality or just add friction.</p>

<h3>
  
  
  Pre-commit hooks for instant feedback
</h3>

<p>The fastest feedback loop is a pre-commit hook that runs before code even reaches a pull request. For Python, the ideal pre-commit configuration runs Ruff for linting and formatting, which completes in under a second for most changesets:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># .pre-commit-config.yaml</span>
<span class="na">repos</span><span class="pi">:</span>
  <span class="pi">-</span> <span class="na">repo</span><span class="pi">:</span> <span class="s">https://github.com/astral-sh/ruff-pre-commit</span>
    <span class="na">rev</span><span class="pi">:</span> <span class="s">v0.9.0</span>
    <span class="na">hooks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">ruff</span>
        <span class="na">args</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">--fix</span><span class="pi">]</span>
      <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">ruff-format</span>
  <span class="pi">-</span> <span class="na">repo</span><span class="pi">:</span> <span class="s">https://github.com/pre-commit/mirrors-mypy</span>
    <span class="na">rev</span><span class="pi">:</span> <span class="s">v1.14.0</span>
    <span class="na">hooks</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="na">id</span><span class="pi">:</span> <span class="s">mypy</span>
        <span class="na">additional_dependencies</span><span class="pi">:</span> <span class="pi">[</span><span class="nv">django-stubs</span><span class="pi">,</span> <span class="nv">types-requests</span><span class="pi">]</span>
</code></pre>

</div>



<p>The key principle is that pre-commit hooks should be fast. If a hook takes more than 5 seconds, developers will bypass it with <code>--no-verify</code>. Ruff and <code>ruff format</code> meet this bar easily. mypy in pre-commit can be slow on large codebases - consider running it only on changed files or moving it to CI only.</p>

<h3>
  
  
  CI/CD for comprehensive analysis
</h3>

<p>CI is where you run the slower, deeper analysis tools that would be too slow for pre-commit. Pylint, Bandit, Semgrep, and platform tools like SonarQube or DeepSource should all run in CI on every pull request.</p>

<p>Structure your CI pipeline so that fast checks run first and slow checks run in parallel:</p>

<ol>
<li>
<strong>Fast (&lt; 30 seconds):</strong> Ruff lint, Ruff format check</li>
<li>
<strong>Medium (30 seconds to 2 minutes):</strong> mypy, Bandit</li>
<li>
<strong>Slow (2+ minutes):</strong> Pylint, Semgrep, SonarQube scan, Snyk Code test</li>
<li>
<strong>Async (runs in background):</strong> CodeRabbit/Sourcery AI review (triggered by PR event, runs independently)</li>
</ol>

<p>This structure ensures developers get fast feedback on style and formatting issues within seconds, while deeper analysis runs in parallel without blocking the quick results.</p>

<h3>
  
  
  Configuring tools to work together without overlap
</h3>

<p>A common mistake is running multiple tools that cover the same rules, resulting in duplicate findings. Here is how to minimize overlap:</p>

<p><strong>Ruff replaces flake8, isort, pycodestyle, and pyflakes.</strong> If you adopt Ruff, remove these tools entirely.</p>

<p><strong>Ruff partially overlaps with Bandit.</strong> Ruff includes the <code>flake8-bandit</code> ruleset (S prefix), which covers the most common Bandit checks. If you run both, either disable the S rules in Ruff or accept the small amount of duplication.</p>

<p><strong>mypy and Pylint overlap on type-related checks.</strong> Pylint performs basic type inference, but mypy is more thorough. If you run both, configure Pylint to skip its type checking categories and focus on code quality checks that mypy does not cover.</p>

<p><strong>SonarQube/DeepSource/Codacy overlap with standalone linters.</strong> If you adopt a platform tool, you can often disable the standalone linters it wraps (since the platform runs them internally). Check the platform's documentation for which tools it includes.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight toml"><code><span class="c"># Pylint configuration when used alongside mypy and Ruff</span>
<span class="c"># pyproject.toml</span>
<span class="nn">[tool.pylint.messages_control]</span>
<span class="c"># Disable checks that Ruff handles better/faster</span>
<span class="py">disable</span> <span class="p">=</span> <span class="p">[</span>
    <span class="s">"C0114"</span><span class="p">,</span>  <span class="c"># missing-module-docstring (use Ruff's D rules)</span>
    <span class="s">"C0115"</span><span class="p">,</span>  <span class="c"># missing-class-docstring</span>
    <span class="s">"C0116"</span><span class="p">,</span>  <span class="c"># missing-function-docstring</span>
    <span class="s">"C0301"</span><span class="p">,</span>  <span class="c"># line-too-long (Ruff handles this)</span>
    <span class="s">"C0303"</span><span class="p">,</span>  <span class="c"># trailing-whitespace (Ruff handles this)</span>
    <span class="s">"W0611"</span><span class="p">,</span>  <span class="c"># unused-import (Ruff handles this)</span>
    <span class="s">"W0612"</span><span class="p">,</span>  <span class="c"># unused-variable (Ruff handles this)</span>
<span class="p">]</span>
</code></pre>

</div>



<h2>
  
  
  How we evaluated these tools
</h2>

<p>To provide accurate recommendations, we evaluated each tool against Python-specific criteria that matter in real production environments.</p>

<p><strong>Python version support:</strong> Does the tool support Python 3.12+ features including the newer type syntax (<code>type X = ...</code>), pattern matching (<code>match/case</code>), and exception groups? Tools that lag behind Python releases create friction for teams that want to use the latest language features.</p>

<p><strong>Framework awareness:</strong> Does the tool understand Django, Flask, and FastAPI patterns? A tool that flags Django's <code>Meta</code> class as unused or does not recognize Flask's <code>@app.route</code> decorator as a public API produces noise that erodes developer trust.</p>

<p><strong>Type annotation support:</strong> Does the tool leverage type annotations for deeper analysis? Python's type system has evolved rapidly (PEP 604 union syntax, PEP 612 ParamSpec, PEP 695 type alias syntax), and tools that understand modern annotations provide better results.</p>

<p><strong>False positive rate on Python code:</strong> Dynamic typing and Python's metaprogramming features (descriptors, metaclasses, <code>__getattr__</code>) cause many static analysis tools to produce false positives. We prioritized tools with demonstrably low false positive rates on real Python code.</p>

<p><strong>CI/CD integration:</strong> Can the tool run in GitHub Actions, GitLab CI, and other common CI systems? Does it provide structured output (JSON, SARIF) for integration with other tools? Can it be configured as a required check that blocks merging?</p>

<p><strong>Performance on Python codebases:</strong> How long does the tool take to analyze a 50,000-line Python project? Tools that take more than 5 minutes to complete frustrate developers and slow down the PR review cycle.</p>

<h2>
  
  
  Conclusion
</h2>

<p>Python's dynamic nature, rich framework ecosystem, and prevalence in security-sensitive domains - web applications, data pipelines, machine learning infrastructure - make it a language that benefits enormously from layered code review tooling. No single tool covers every dimension of Python code quality, but the right combination provides coverage that far exceeds what human reviewers can achieve alone.</p>

<p><strong>For linting</strong>, Ruff has become the clear standard. It is faster, covers more rules, and requires less configuration than any alternative. Teams still using flake8 should migrate - the effort is minimal and the speed improvement is dramatic.</p>

<p><strong>For type checking</strong>, mypy remains the standard for CI/CD, while Pyright through Pylance provides the best IDE experience. The two tools complement each other, and most Python teams benefit from running both.</p>

<p><strong>For security scanning</strong>, the choice depends on your depth requirements. Bandit provides a strong free baseline for Python-specific vulnerabilities. <a href="https://dev.to/tool/semgrep">Semgrep</a> adds framework-specific rules for Django and Flask plus custom rule authoring. <a href="https://dev.to/tool/snyk-code">Snyk Code</a> adds cross-file dataflow analysis that catches vulnerabilities spanning multiple modules.</p>

<p><strong>For AI-powered review</strong>, <a href="https://dev.to/tool/sourcery">Sourcery</a> provides the deepest Python-specific analysis with refactoring suggestions that teach Pythonic patterns. <a href="https://dev.to/tool/coderabbit">CodeRabbit</a> provides the best overall AI review quality with a generous free tier. <a href="https://dev.to/tool/deepsource">DeepSource</a> combines AI with static analysis for the lowest false positive rate.</p>

<p><strong>For full quality platforms</strong>, <a href="https://dev.to/tool/sonarqube">SonarQube</a> offers the deepest rule set and quality gate enforcement for enterprise teams. <a href="https://dev.to/tool/codacy">Codacy</a> offers the best value for small to mid-size teams. <a href="https://dev.to/tool/qlty">Qlty</a> offers the best unified experience for polyglot codebases that include Python alongside other languages.</p>

<p>Start with the minimal stack - Ruff, mypy, and a free AI reviewer - and add tools as your team and codebase grow. The goal is not to run every tool on this list. The goal is to build a code review workflow where Python-specific bugs, security vulnerabilities, and quality issues are caught automatically, freeing your human reviewers to focus on architecture, business logic, and the concerns that only humans can evaluate.</p>

<h2>
  
  
  Frequently Asked Questions
</h2>

<h3>
  
  
  What is the best Python linter in 2026?
</h3>

<p>Ruff has become the dominant Python linter - 10-100x faster than Pylint/flake8, covers 800+ rules from multiple tools, and handles formatting too. For teams starting fresh, Ruff is the clear choice. Pylint still offers deeper analysis for teams already invested in it.</p>

<h3>
  
  
  What is the best AI code review tool for Python?
</h3>

<p>Sourcery is purpose-built for Python and provides the most Python-specific suggestions. CodeRabbit offers the best overall AI review with strong Python support. DeepSource combines AI analysis with autofix specifically tuned for Python patterns.</p>

<h3>
  
  
  Is Pylint still worth using?
</h3>

<p>Pylint remains the deepest Python-specific linter with checks that Ruff cannot replicate, like complex type inference and custom plugin support. However, Ruff covers 80%+ of common linting needs at dramatically faster speeds. Many teams use Ruff for speed and add Pylint in CI for deep analysis.</p>

<h3>
  
  
  What tools detect Python security vulnerabilities?
</h3>

<p>Bandit (free, Python-specific), Semgrep (free OSS with Python rules), Snyk Code (cross-file dataflow analysis), and SonarQube (broad coverage with Python-specific rules). For Django/Flask-specific security, Semgrep has dedicated rulesets covering injection, CSRF, and authentication issues.</p>

<h3>
  
  
  Should I use mypy or pyright for Python type checking?
</h3>

<p>Mypy is the established standard with broad ecosystem support. Pyright (from Microsoft) is faster and used by Pylance in VS Code. For CI/CD, mypy is more commonly integrated. For IDE experience, Pyright through Pylance is superior. Many teams use both - Pyright in the IDE and mypy in CI.</p>

<h3>
  
  
  How do I set up automated code review for a Python project?
</h3>

<p>Start with Ruff for linting and formatting (replaces flake8, isort, black). Add mypy for type checking. Then add an AI review tool like CodeRabbit or Sourcery for logic-level analysis. Finally, add Semgrep or Bandit for security scanning. All can run in GitHub Actions or GitLab CI.</p>

<h3>
  
  
  Is Ruff a replacement for flake8 and Black?
</h3>

<p>Yes. Ruff reimplements the rules of flake8, isort, Black, pyupgrade, and over a dozen other Python tools in a single Rust-based binary. It covers 800+ lint rules and includes a Prettier-like formatter. For new Python projects in 2026, Ruff is the recommended default that replaces the entire legacy linting toolchain.</p>

<h3>
  
  
  How much do Python code review tools cost?
</h3>

<p>Many Python code review tools are completely free. Ruff, Pylint, mypy, Pyright, and Bandit are all open source. CodeRabbit offers unlimited free AI reviews on public and private repos. Paid tools like Sourcery ($29/seat/month), DeepSource ($30/user/month), and Semgrep ($35/dev/month for 10+ contributors) add deeper analysis and team features.</p>

<h3>
  
  
  What is the best code review tool for Django projects?
</h3>

<p>For Django specifically, combine Semgrep with the p/django ruleset for security scanning (SQL injection, CSRF, XSS), Pylint with pylint-django for deep code analysis, and CodeRabbit or Sourcery for AI-powered logic review. Ruff's flake8-django rules (DJ prefix) also catch Django-specific issues like missing model <strong>str</strong> methods.</p>

<h3>
  
  
  Can AI code review tools catch Python security vulnerabilities?
</h3>

<p>Yes, but they work best combined with dedicated security scanners. AI tools like CodeRabbit and Sourcery catch logic-level security issues such as missing input validation and improper error handling. Dedicated SAST tools like Bandit, Semgrep, and Snyk Code catch specific vulnerability patterns like SQL injection, hardcoded secrets, and insecure deserialization with near-zero false positives.</p>

<h3>
  
  
  What is the difference between Ruff and Pylint?
</h3>

<p>Ruff is a fast syntactic linter that analyzes code as text and AST nodes, covering 800+ rules at 10-100x the speed of Pylint. Pylint performs deeper semantic analysis including type inference and cross-expression evaluation. Many teams use both - Ruff in pre-commit hooks for instant feedback and Pylint in CI for the deep analysis that Ruff cannot replicate.</p>

<h3>
  
  
  How do I reduce false positives from Python static analysis tools?
</h3>

<p>Choose tools with low false positive rates like DeepSource (sub-5% false positive rate). Configure per-file ignores for generated code, migrations, and test files. Use baseline features to suppress existing issues and only enforce rules on new code. Disable rules that consistently produce unhelpful findings for your specific codebase.</p>

<h3>
  
  
  What Python code review tools work with FastAPI?
</h3>

<p>CodeRabbit can review FastAPI endpoints for missing response models, improper error handling, and async patterns. Semgrep has rules for FastAPI security issues including CORS misconfiguration and input validation bypass. Ruff catches general Python issues in FastAPI code, and mypy with Pydantic plugin validates type annotations used in FastAPI schemas.</p>

<h3>
  
  
  Do Python code review tools support GitHub Actions?
</h3>

<p>Yes, virtually all Python code review tools integrate with GitHub Actions. Ruff, mypy, Pylint, and Bandit run as standard CLI commands in workflow steps. CodeRabbit and Sourcery install as GitHub Apps that trigger automatically on pull requests. Semgrep and SonarQube provide official GitHub Actions. DeepSource and Codacy also offer native GitHub integration.</p>




<p><em>Originally published at <a href="https://aicodereview.cc/blog/best-code-review-tools-python/" rel="noopener noreferrer">aicodereview.cc</a></em></p>

