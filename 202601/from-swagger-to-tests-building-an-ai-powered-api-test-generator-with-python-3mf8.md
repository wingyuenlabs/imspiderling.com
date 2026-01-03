---
Title: From Swagger to Tests: Building an AI-Powered API Test Generator with Python
Description: 
Author: Alicia Marianne 🇧🇷 
Date: 2026-01-03T21:23:08.000Z
Robots: noindex,nofollow
Template: index
---
<p>Working as a QA with APIs can be… well, kind of a nightmare sometimes. APIs are always changing, endpoints get added, status codes get updated, and keeping your tests in sync feels like chasing a moving target.</p>

<p>If you only look at your task board, it’s easy to lose track of what actually changed and what still needs testing.</p>

<p>In the projects I worked on, we had Swagger available for the API. And I thought: <em>wait a minute… why not use AI and Swagger to save time generating tests?</em></p>

<p>And that’s how this little project started. In this post, I’ll walk you through how I did it, the challenges I faced, and some cool things you can do next.</p>

<h2>
  
  
  The Idea
</h2>

<p>The goal was simple: take the Swagger spec and extract all the useful info, like:</p>

<ul>
<li>HTTP methods</li>
<li>Expected status codes</li>
<li>Query parameters</li>
<li>Request bodies</li>
</ul>

<p>…and then generate <strong>both positive and negative test scenarios</strong> automatically.</p>

<p>For example, for a simple <code>GET /users/{id}</code> endpoint, I wanted the output to look like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>GET /users/<span class="o">{</span><span class="nb">id</span><span class="o">}</span>
✔ Scenario: Retrieve a user with a valid ID  
✔ Scenario: Validate 404 <span class="k">for </span>user not found  
✘ Scenario: Missing ID parameter  
✘ Scenario: Invalid format <span class="k">for </span>ID  
</code></pre>

</div>



<p>To make this work nicely, I used AI to create the scenarios based on the endpoint’s Swagger specification, following a template I defined.</p>

<h2>
  
  
  About the project
</h2>

<h3>
  
  
  Stack
</h3>

<ul>
<li>
<strong>Python</strong> – fast, easy to parse data, integrate stuff
</li>
<li>
<strong>Rich / Typer (CLI UX)</strong> – because a pretty CLI makes life better</li>
<li>
<strong>Gemini AI</strong> – super simple Python integration for AI prompts</li>
<li>
<strong>dotenv</strong> – to keep the AI keys safe</li>
</ul>

<h3>
  
  
  Project Structure
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>api-test-generator/
├── README.md                   # Documentation of the project
├── requirements.txt            # Dependências Python
├── main.py                     # Main function
│
├── output/                         # Folder with generated tests 
│   ├── get_Books.txt
│   ├── post_Books.txt                  
│
├── functions/                       # Main functions of the project 
│   ├── navigation.py                # CLI navigation
│   ├── read_swagger.py              # Read files and URL swaggers 
│   └── test_generator.py            # Generate tests and save them in the files
│
└── assets/                    # theme and example for the project 
    ├── swaggerexample.json
    └── theme.py
</code></pre>

</div>



<h3>
  
  
  How it works
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>┌──────────────────────────────┐
│          User / QA           │
│   (CLI Interaction - Rich)   │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│        CLI Interface         │
│   (Typer + Rich Menu)        │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│   Swagger/OpenAPI Loader     │
│  - URL, Manual, or Local JSON│
│  - Validation &amp; Parsing      │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│   API Specification Parser   │
│  - Endpoints                 │
│  - Methods                   │
│  - Parameters                │
│  - Responses / Status Codes  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│        Gemini AI API         │
│   (Test Case Generation)     │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│     Output Generator         │
│  - Text file export (.txt)   │
│  - Structured scenarios      │
└──────────────────────────────┘
</code></pre>

</div>



<p>So basically: user interacts with CLI → loads Swagger → parses specs → builds a prompt → sends to AI → AI returns tests → saves to file.</p>

<h2>
  
  
  Code Highlights
</h2>

<h3>
  
  
  The test generator
</h3>

<p>The core idea here was: extract <strong>as much info as possible</strong> from Swagger so the AI could generate meaningful tests.</p>

<p>Here’s the main function I wrote:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">test_generator</span><span class="p">(</span><span class="n">path</span><span class="p">,</span> <span class="n">method</span><span class="p">,</span> <span class="n">swagger_data</span><span class="p">):</span>  
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Generating tests for </span><span class="si">{</span><span class="n">method</span><span class="p">.</span><span class="nf">upper</span><span class="p">()</span><span class="si">}</span><span class="s"> </span><span class="si">{</span><span class="n">path</span><span class="si">}</span><span class="s">...</span><span class="sh">"</span><span class="p">)</span>  
    <span class="n">details</span> <span class="o">=</span> <span class="n">swagger_data</span><span class="p">[</span><span class="sh">"</span><span class="s">paths</span><span class="sh">"</span><span class="p">][</span><span class="n">path</span><span class="p">][</span><span class="n">method</span><span class="p">]</span>  
    <span class="n">request_body</span> <span class="o">=</span> <span class="sh">""</span>  
    <span class="n">parameters</span> <span class="o">=</span> <span class="sh">""</span>  
    <span class="c1"># Getting information about the endpoint  
</span>    <span class="k">if</span> <span class="sh">'</span><span class="s">tags</span><span class="sh">'</span> <span class="ow">not</span> <span class="ow">in</span> <span class="n">details</span><span class="p">:</span>  
        <span class="n">endpoint_name</span> <span class="o">=</span> <span class="n">path</span>  
    <span class="k">elif</span> <span class="nf">len</span><span class="p">(</span><span class="n">details</span><span class="p">[</span><span class="sh">'</span><span class="s">tags</span><span class="sh">'</span><span class="p">])</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  
        <span class="n">endpoint_name</span> <span class="o">=</span> <span class="n">path</span>  
    <span class="k">else</span><span class="p">:</span>  
        <span class="n">endpoint_name</span> <span class="o">=</span> <span class="n">details</span><span class="p">[</span><span class="sh">'</span><span class="s">tags</span><span class="sh">'</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>  
    <span class="k">if</span> <span class="sh">'</span><span class="s">requestBody</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">details</span><span class="p">:</span>  
        <span class="n">request_body</span> <span class="o">=</span> <span class="n">details</span><span class="p">[</span><span class="sh">'</span><span class="s">requestBody</span><span class="sh">'</span><span class="p">]</span>  
    <span class="k">if</span> <span class="sh">'</span><span class="s">parameters</span><span class="sh">'</span> <span class="ow">in</span> <span class="n">details</span><span class="p">:</span>  
        <span class="n">parameters</span> <span class="o">=</span> <span class="n">details</span><span class="p">[</span><span class="sh">'</span><span class="s">parameters</span><span class="sh">'</span><span class="p">]</span>  
    <span class="n">prompt</span> <span class="o">=</span> <span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Generate positive and negative tests for this endpoint:</span><span class="si">{</span><span class="n">path</span><span class="si">}</span><span class="s"> for the method </span><span class="si">{</span><span class="n">method</span><span class="p">.</span><span class="nf">upper</span><span class="p">()</span><span class="si">}</span><span class="sh">"</span>  
              <span class="sa">f</span><span class="sh">"</span><span class="s">considering the following specifications: </span><span class="sh">"</span>             
              <span class="sa">f</span><span class="sh">"</span><span class="s">Name of the endpoint: </span><span class="si">{</span><span class="n">endpoint_name</span><span class="si">}</span><span class="sh">"</span>  
              <span class="sa">f</span><span class="sh">"</span><span class="s">Request body: </span><span class="si">{</span><span class="n">request_body</span><span class="si">}</span><span class="sh">"</span>  
              <span class="sa">f</span><span class="sh">"</span><span class="s">Query Parameters: </span><span class="si">{</span><span class="n">parameters</span><span class="si">}</span><span class="s"> and return the tests following this template: </span><span class="si">{</span><span class="n">theme</span><span class="p">.</span><span class="n">PROMPT_TEMPLATE</span><span class="si">}</span><span class="sh">"</span><span class="p">)</span>  
    <span class="n">test_scenario</span> <span class="o">=</span> <span class="nf">ai_connection</span><span class="p">(</span><span class="n">prompt</span><span class="p">)</span>  
    <span class="nf">print</span><span class="p">(</span><span class="sa">f</span><span class="sh">"</span><span class="s">Exporting tests to file...</span><span class="sh">"</span><span class="p">)</span>  
    <span class="nf">export_to_file</span><span class="p">(</span><span class="n">test_scenario</span><span class="p">,</span> <span class="n">method</span><span class="p">,</span> <span class="n">endpoint_name</span><span class="p">)</span>
</code></pre>

</div>



<h3>
  
  
  Connecting to Gemini AI
</h3>

<p>Connecting to the AI is simple: create a client, set the model, and pass the prompt:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight python"><code><span class="k">def</span> <span class="nf">ai_connection</span><span class="p">(</span><span class="n">prompt</span><span class="p">):</span>  
    <span class="nf">load_dotenv</span><span class="p">()</span>  
    <span class="n">api_key</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="nf">getenv</span><span class="p">(</span><span class="sh">"</span><span class="s">GOOGLE_API_KEY</span><span class="sh">"</span><span class="p">)</span>  
    <span class="n">client</span> <span class="o">=</span> <span class="n">genai</span><span class="p">.</span><span class="nc">Client</span><span class="p">(</span><span class="n">api_key</span><span class="o">=</span><span class="n">api_key</span><span class="p">)</span>  

    <span class="n">response</span> <span class="o">=</span> <span class="n">client</span><span class="p">.</span><span class="n">models</span><span class="p">.</span><span class="nf">generate_content</span><span class="p">(</span>  
        <span class="n">model</span><span class="o">=</span><span class="sh">"</span><span class="s">gemini-2.5-flash</span><span class="sh">"</span><span class="p">,</span>  
        <span class="n">contents</span><span class="o">=</span><span class="n">prompt</span>  
    <span class="p">)</span>  
    <span class="k">return</span> <span class="n">response</span><span class="p">.</span><span class="n">text</span>
</code></pre>

</div>



<p>And voilà. The AI returns something like:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>POST /api/v1/Books  
✔ Scenario: Successfully create a new book with all valid fields  
✔ Scenario: Successfully create a new book with only mandatory fields  
✔ Scenario: Successfully create a new book using <span class="s1">'text/json; v=1.0'</span> content <span class="nb">type</span>  

✘ Scenario: Fail to create book due to missing <span class="s1">'title'</span> field  
✘ Scenario: Fail to create book due to missing <span class="s1">'author'</span> field  
✘ Scenario: Fail to create book due to missing <span class="s1">'isbn'</span> field  
✘ Scenario: Fail to create book with an <span class="s1">'isbn'</span> that already exists <span class="o">(</span>conflict<span class="o">)</span>  
✘ Scenario: Fail to create book due to invalid <span class="s1">'isbn'</span> format <span class="o">(</span>e.g., too short, non-numeric where expected<span class="o">)</span>  
✘ Scenario: Fail to create book due to <span class="s1">'publication_year'</span> being a string instead of an integer  
✘ Scenario: Fail to create book due to empty request body  
✘ Scenario: Fail to create book due to malformed JSON <span class="k">in </span>request body  
✘ Scenario: Fail to create book with an empty <span class="s1">'title'</span> string  
✘ Scenario: Fail to create book with an empty <span class="s1">'author'</span> string  
</code></pre>

</div>



<h2>
  
  
  Challenges &amp; Lessons Learned
</h2>

<p>Honestly, the hardest part was <strong>cleaning up Swagger data</strong> and building prompts that make sense for the AI.<br>
Another challenge was designing a workflow that actually works in a CLI without feeling clunky.<br>
But in the end, it was super fun, and I learned a lot about AI-assisted testing.</p>

<h2>
  
  
  What’s Next
</h2>

<p>While building this, I started dreaming about all the things I could do next:</p>

<ul>
<li>Automatically generate <strong>Postman collections</strong> from these tests</li>
<li>Integrate with <strong>test management tools</strong> like Zephyr or Xray</li>
<li>Make it a <strong>service</strong> that monitors Swagger and updates tests whenever endpoints change
The possibilities are endless.</li>
</ul>

<h2>
  
  
  Conclusion
</h2>

<p>This project really showed me that <strong>AI + OpenAPI = massive time saver</strong>.</p>

<p>Instead of manually writing dozens of tests for every endpoint, I now have an automated system that generates <strong>both positive and negative scenarios in minutes</strong>.</p>

<p>Next steps? Think bigger: integrate it with CI/CD pipelines, plug it into test management tools, or even make it monitor APIs in real-time. Smarter, faster, and way less painful API testing—sounds like a win to me.</p>

<p>If you want to check out the full project, explore the code, or try it yourself, it’s all on my GitHub: <a href="https://github.com/m4rri4nne/api-test-generator" rel="noopener noreferrer">API Test Generator</a>. </p>

<p>Dive in, experiment, and see how much time you can save!</p>

